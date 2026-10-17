# Zajęcia 1

## Skąd się biorą dane — pozyskiwanie i wczytywanie

Każda analiza zaczyna się od tego samego pytania: **skąd wziąć dane?** Na
kursach zwykle dostaje się gotowy, grzeczny plik CSV. W pracy dane przychodzą
z eksportu starego systemu (z polskimi znakami w egzotycznym kodowaniu),
z serwisu internetowego przez API, z firmowej bazy danych — i prawie nigdy
z jednego miejsca naraz.

Dziś zajmujemy się pierwszym krokiem drogi, którą będziemy szli cały semestr:
**pozyskanie → czyszczenie → analiza → wizualizacja**. Nauczysz się wyciągać dane
z różnych źródeł i sklejać je w jedną tabelę, na której da się już coś policzyć.

Po dzisiejszych zajęciach będziesz w stanie:

- wczytać plik CSV, który „nie chce się wczytać" — złe kodowanie, opis nad
  tabelą, średniki, przecinek dziesiętny,
- wczytać zagnieżdżony JSON i spłaszczyć go do tabeli,
- pobrać dane z publicznego REST API (Narodowy Bank Polski) biblioteką `requests`,
- zadać zapytanie SQL do bazy danych prosto z Pythona,
- połączyć dane z kilku źródeł — także wtedy, gdy daty nie pasują do siebie
  jeden do jednego.

---

## Plan na dziś

| Blok | Czas |
|---|---|
| Organizacja i zasady zaliczenia | 10 min |
| Skąd biorą się dane | 5 min |
| Pliki: CSV z niespodzianką i zagnieżdżony JSON | 20 min |
| API — dane prosto z serwisu | 15 min |
| Bazy danych — SQL z Pythona | 10 min |
| Łączenie źródeł — `merge` i `merge_asof` | 10 min |
| Przerwa | 10 min |
| **Zadanie 1** | **55 min** |

---

## Skąd biorą się dane

Najczęstsze źródła danych, z którymi spotkasz się w pracy:

| Źródło | Przykład | Czym wczytujemy |
|---|---|---|
| **pliki** | eksport z systemu sprzedażowego, raport z Excela | `pd.read_csv`, `pd.read_excel` |
| **API** | kursy walut z NBP, pogoda, dane z aplikacji | `requests` + `pd.json_normalize` |
| **bazy danych** | firmowa baza zamówień, CRM | `pd.read_sql_query` |
| **strony WWW** | tabela na Wikipedii, cennik konkurencji | `pd.read_html`, scraping |

Bez względu na źródło cel jest zawsze ten sam: dostać **DataFrame** — tabelę
pandas, z którą dalej pracuje się już identycznie. Wszystkie przykłady
uruchamiamy w [Google Colab](https://colab.research.google.com), więc niczego nie
musisz instalować.

!!! info "Dane, na których pracujemy w demo"

    W przykładach analizujemy miejską wypożyczalnię rowerów: przejazdy (plik CSV),
    stacje (JSON) i — dla odmiany — ceny złota z NBP. W zadaniu te same techniki
    zastosujesz do danych sklepu internetowego.

---

## Pliki: CSV z niespodzianką

Uruchom komórkę — utworzy plik `przejazdy.csv` taki, jaki dostałbyś z systemu
wypożyczalni:

```python
tekst = """Raport wypożyczeń — Rower Miejski
Okres: 2026-09-01 – 2026-09-03
id;data;stacja_id;czas_min;oplata
1;2026-09-01;S01;12;0,00
2;2026-09-01;S02;47;4,50
3;2026-09-01;S01;23;0,00
4;2026-09-02;S03;65;8,00
5;2026-09-02;S02;8;0,00
6;2026-09-02;S04;31;2,00
7;2026-09-03;S01;94;12,50
8;2026-09-03;S03;19;0,00
"""

with open("przejazdy.csv", "w", encoding="cp1250") as f:
    f.write(tekst)
```

Zwróć uwagę na `encoding="cp1250"` — tak zapisują pliki starsze polskie systemy
i Excel na Windowsie. Spróbujmy go wczytać „normalnie":

```python
import pandas as pd

pd.read_csv("przejazdy.csv")
```

```text
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xbf in position 11: invalid start byte
```

Pierwsza niespodzianka. pandas domyślnie zakłada kodowanie **UTF-8**, a w pliku
litera „ż" z „wypożyczeń" jest zapisana jako bajt `0xbf`, który w UTF-8 nic nie
znaczy. Podpowiadamy kodowanie:

```python
pd.read_csv("przejazdy.csv", encoding="cp1250")
```

```text
ParserError: Error tokenizing data. C error: Expected 1 fields in line 4, saw 2
```

Druga niespodzianka. pandas uznał pierwszy wiersz („Raport wypożyczeń…") za
nagłówek z jedną kolumną, a w wierszu 4 znalazł nagle dwie — bo przecinek
w `0,00` wziął za separator. Dwa pierwsze wiersze to opis, nie dane. Pomijamy je
parametrem `skiprows`, a do tego podajemy separator i przecinek dziesiętny:

```python
przejazdy = pd.read_csv(
    "przejazdy.csv",
    encoding="cp1250",   # polskie znaki ze starego systemu
    skiprows=2,          # dwa wiersze opisu nad tabelą
    sep=";",             # kolumny rozdzielone średnikiem
    decimal=",",         # liczby z przecinkiem: 4,50
)
przejazdy
```

```text
   id        data stacja_id  czas_min  oplata
0   1  2026-09-01       S01        12     0.0
1   2  2026-09-01       S02        47     4.5
2   3  2026-09-01       S01        23     0.0
3   4  2026-09-02       S03        65     8.0
4   5  2026-09-02       S02         8     0.0
5   6  2026-09-02       S04        31     2.0
6   7  2026-09-03       S01        94    12.5
7   8  2026-09-03       S03        19     0.0
```

Teraz wygląda jak tabela. Ale „wygląda" to za mało — sprawdzamy typy:

```python
przejazdy.dtypes
```

```text
id             int64
data             str
stacja_id        str
czas_min     int64
oplata       float64
dtype: object
```

`oplata` to liczba (`float64`), więc da się ją sumować. Gdyby nie `decimal=","`,
byłaby tekstem `"4,50"` — i to bez żadnego błędu, po cichu. Data na razie jest
tekstem; tym zajmiemy się za chwilę.

!!! tip "Jak czytać błędy przy wczytywaniu"

    | Objaw | Przyczyna | Lekarstwo |
    |---|---|---|
    | `UnicodeDecodeError` albo „GdaÅ„sk" zamiast „Gdańsk" | kodowanie inne niż UTF-8 | `encoding="cp1250"` |
    | `Expected 1 fields in line …` | coś nad tabelą albo zły separator | `skiprows=…`, `sep=";"` |
    | wszystko w jednej kolumnie | zły separator | `sep=";"` |
    | liczby jako tekst `"4,50"` | przecinek dziesiętny | `decimal=","` |

    Zanim zaczniesz zgadywać — otwórz plik w zwykłym edytorze tekstu albo podejrzyj
    pierwsze linijki. Dwie sekundy patrzenia oszczędzają kwadrans prób.

### Zagnieżdżony JSON

JSON to format, w którym dane zwracają serwisy internetowe. Często jest
**zagnieżdżony** — w środku rekordu siedzi kolejny słownik. Tak wyglądają
stacje naszej wypożyczalni:

```python
import json

stacje_json = [
    {"stacja_id": "S01", "nazwa": "Metro Centrum",
     "lokalizacja": {"dzielnica": "Śródmieście", "stojaki": 30}},
    {"stacja_id": "S02", "nazwa": "Plac Wilsona",
     "lokalizacja": {"dzielnica": "Żoliborz", "stojaki": 20}},
    {"stacja_id": "S03", "nazwa": "Politechnika",
     "lokalizacja": {"dzielnica": "Mokotów", "stojaki": 25}},
]

with open("stacje.json", "w", encoding="utf-8") as f:
    json.dump(stacje_json, f, ensure_ascii=False, indent=2)
```

`pd.read_json` wczyta plik, ale zagnieżdżony słownik wrzuci w całości do jednej
kolumny:

```python
pd.read_json("stacje.json")
```

```text
  stacja_id          nazwa                                  lokalizacja
0       S01  Metro Centrum  {'dzielnica': 'Śródmieście', 'stojaki': 30}
1       S02   Plac Wilsona     {'dzielnica': 'Żoliborz', 'stojaki': 20}
2       S03   Politechnika      {'dzielnica': 'Mokotów', 'stojaki': 25}
```

Z kolumny pełnej słowników niewiele policzysz. Do spłaszczania służy
`pd.json_normalize` — dostaje listę słowników i każde zagnieżdżone pole zamienia
na osobną kolumnę:

```python
with open("stacje.json", encoding="utf-8") as f:
    dane = json.load(f)          # plik JSON → lista słowników Pythona

stacje = pd.json_normalize(dane)
stacje
```

```text
  stacja_id          nazwa lokalizacja.dzielnica  lokalizacja.stojaki
0       S01  Metro Centrum           Śródmieście                   30
1       S02   Plac Wilsona              Żoliborz                   20
2       S03   Politechnika               Mokotów                   25
```

Nazwy nowych kolumn powstają z całej „ścieżki" do pola: `lokalizacja.dzielnica`.
Kropka w nazwie nie przeszkadza — w nawiasach kwadratowych działa normalnie:
`stacje["lokalizacja.dzielnica"]`.

!!! note "Rodzina `read_...`"

    Do innych formatów pandas ma bliźniacze funkcje: `read_excel` (z parametrem
    `sheet_name`), `read_parquet`, `read_html`, `read_sql_query`… Wszystkie
    zwracają ten sam `DataFrame`. Jak raz nauczysz się pracować z tabelą,
    źródło przestaje mieć znaczenie.

---

## API — dane prosto z serwisu

**API** (*Application Programming Interface*) to „drzwi dla programów" do
serwisu internetowego. Zamiast klikać na stronie i ręcznie pobierać plik,
wysyłasz zapytanie pod konkretny adres i dostajesz dane — zwykle w JSON-ie.
Zaletą jest to, że kod działa zawsze tak samo: jutro uruchamiasz go jeszcze raz
i masz świeże dane.

Ćwiczymy na **API Narodowego Banku Polskiego**
([api.nbp.pl](https://api.nbp.pl)): darmowe, bez rejestracji i bez klucza. NBP
udostępnia kursy walut i ceny złota. Adres zapytania składa się z kawałków:

```text
https://api.nbp.pl/api/cenyzlota/2025-12-19/2026-01-09/?format=json
└──────── serwis ───────┘└ co ──┘└─ od ────┘└─ do ────┘└─ format ─┘
```

Zapytanie wysyłamy biblioteką `requests` (w Colabie jest od razu):

```python
import requests

URL = "https://api.nbp.pl/api/cenyzlota/2025-12-19/2026-01-09/?format=json"
odp = requests.get(URL)
odp.status_code
```

```text
200
```

**Kod odpowiedzi** mówi, czy się udało: `200` — OK, `404` — nie ma takich
danych, `400` — błędne zapytanie (np. zła data). Zawsze sprawdzaj go przed dalszą
pracą. Same dane wyciągamy metodą `.json()` — dostajemy zwykłą listę słowników:

```python
odp.json()[:3]
```

```text
[{'data': '2025-12-19', 'cena': 498.88}, {'data': '2025-12-22', 'cena': 500.23}, {'data': '2025-12-23', 'cena': 509.15}]
```

A z listy słowników robimy tabelę — dokładnie tak jak z pliku JSON:

```python
zloto = pd.DataFrame(odp.json())
zloto
```

```text
          data    cena
0   2025-12-19  498.88
1   2025-12-22  500.23
2   2025-12-23  509.15
3   2025-12-29  516.35
4   2025-12-30  516.11
5   2025-12-31  505.12
6   2026-01-02  500.84
7   2026-01-05  507.01
8   2026-01-07  513.89
9   2026-01-08  517.03
10  2026-01-09  514.00
```

Cena to złotówki za gram złota. Przyjrzyj się datom: brakuje weekendów, ale
też 24–26 grudnia, 1 i 6 stycznia. **NBP publikuje notowania tylko w dni robocze.**
Zapamiętaj to — za chwilę będzie z tego problem.

!!! tip "Kursy walut z NBP"

    Kursy średnie walut (tabela A) mają adres w tym samym stylu:

    ```text
    https://api.nbp.pl/api/exchangerates/rates/a/eur/2025-03-01/2025-03-31/?format=json
    ```

    Tu odpowiedź jest zagnieżdżona: kursy siedzą w kluczu `"rates"`, a każdy
    kurs ma pola `effectiveDate` (data) i `mid` (kurs średni). Dwie rzeczy, na
    które trzeba uważać: jedno zapytanie obejmuje maksymalnie **93 dni**, a zakres
    dat bez żadnego notowania (np. sam weekend) kończy się kodem `404` i komunikatem
    „Brak danych".

---

## Bazy danych — SQL z Pythona

W firmach większość danych mieszka w **bazach danych** (PostgreSQL, MySQL,
SQL Server, Oracle…). Pyta się je językiem **SQL**, a pandas potrafi wysłać
zapytanie i od razu zwrócić wynik jako DataFrame.

Ćwiczymy na **SQLite** — to baza w jednym pliku, wbudowana w Pythona. Nie ma
serwera, nie ma haseł, a SQL jest taki sam jak w „dużych" bazach. Najpierw
zapiszemy do niej nasze dwie tabele (w prawdziwej firmie baza już by istniała):

```python
import sqlite3

con = sqlite3.connect("rowery.db")      # połączenie; plik powstanie, jeśli go nie ma

przejazdy.to_sql("przejazdy", con, index=False, if_exists="replace")
stacje.to_sql("stacje", con, index=False, if_exists="replace")
```

A teraz zapytanie. SQL czyta się prawie jak zdanie po angielsku: *wybierz stację,
liczbę przejazdów, średni czas i sumę opłat z tabeli przejazdy, pogrupuj po
stacji, posortuj malejąco po liczbie*:

```python
zapytanie = """
SELECT stacja_id,
       COUNT(*)                AS liczba,
       ROUND(AVG(czas_min), 1) AS sredni_czas,
       SUM(oplata)             AS oplaty
FROM przejazdy
GROUP BY stacja_id
ORDER BY liczba DESC
"""

pd.read_sql_query(zapytanie, con)
```

```text
  stacja_id  liczba  sredni_czas  oplaty
0       S01       3         43.0    12.5
1       S03       2         42.0     8.0
2       S02       2         27.5     4.5
3       S04       1         31.0     2.0
```

To samo w pandas zrobiłbyś przez `groupby`. Po co więc SQL? Bo przy dużej bazie
**liczy baza**, a do Pythona przychodzi tylko gotowy wynik — cztery wiersze
zamiast czterech milionów.

!!! note "Najważniejsze słowa SQL"

    | SQL | Co robi | Odpowiednik w pandas |
    |---|---|---|
    | `SELECT kol1, kol2` | wybiera kolumny | `df[["kol1", "kol2"]]` |
    | `FROM tabela` | z której tabeli | — |
    | `WHERE oplata > 0` | filtruje wiersze | `df[df["oplata"] > 0]` |
    | `GROUP BY kol` | grupuje | `df.groupby("kol")` |
    | `ORDER BY kol DESC` | sortuje malejąco | `df.sort_values("kol", ascending=False)` |

    Z prawdziwą bazą firmową łączysz się tak samo, tylko zamiast `sqlite3.connect`
    używasz biblioteki SQLAlchemy z adresem serwera i hasłem.

---

## Łączenie źródeł

Mamy przejazdy z CSV i stacje z JSON-a. Chcemy wiedzieć, w których
**dzielnicach** jeździ się najwięcej — a dzielnica jest w drugiej tabeli. Łączymy
je po wspólnej kolumnie `stacja_id` metodą `merge`:

```python
razem = przejazdy.merge(stacje, on="stacja_id", how="left")
razem[["id", "stacja_id", "czas_min", "nazwa", "lokalizacja.dzielnica"]]
```

```text
   id stacja_id  czas_min          nazwa lokalizacja.dzielnica
0   1       S01        12  Metro Centrum           Śródmieście
1   2       S02        47   Plac Wilsona              Żoliborz
2   3       S01        23  Metro Centrum           Śródmieście
3   4       S03        65   Politechnika               Mokotów
4   5       S02         8   Plac Wilsona              Żoliborz
5   6       S04        31            NaN                   NaN
6   7       S01        94  Metro Centrum           Śródmieście
7   8       S03        19   Politechnika               Mokotów
```

Przejazd nr 6 zaczął się na stacji `S04`, której nie ma w pliku stacji — dostał
więc `NaN`. Parametr `how` decyduje, co z takimi wierszami zrobić:

- `how="left"` — zostaw **wszystkie** wiersze lewej tabeli (przejazdów), braki
  uzupełnij `NaN`; to najczęstszy wybór,
- `how="inner"` (domyślnie) — zostaw tylko wiersze, które mają parę w obu
  tabelach; przejazd nr 6 by **zniknął**, i to bez żadnego ostrzeżenia.

!!! warning "Po każdym `merge` policz wiersze"

    Sprawdź `len()` przed i po łączeniu. Mniej wierszy — coś się nie dopasowało.
    Więcej — w drugiej tabeli klucz się powtarza i wiersze się zduplikowały.
    Jedno i drugie po cichu psuje wszystkie późniejsze sumy.

### Gdy daty nie pasują: `merge_asof`

Teraz trudniejszy przypadek. Klient kupował złoto w różne dni, także w weekendy
i święta, a chcemy wycenić każdy zakup po cenie NBP. Zwykły `merge` po dacie
zawodzi:

```python
zloto["data"] = pd.to_datetime(zloto["data"])

zakupy = pd.DataFrame({
    "data": pd.to_datetime(["2025-12-22", "2025-12-24", "2025-12-26", "2025-12-28",
                            "2026-01-01", "2026-01-06", "2026-01-08"]),
    "gramy": [10, 5, 20, 2, 50, 15, 1],
})

zakupy.merge(zloto, on="data", how="left")
```

```text
        data  gramy    cena
0 2025-12-22     10  500.23
1 2025-12-24      5     NaN
2 2025-12-26     20     NaN
3 2025-12-28      2     NaN
4 2026-01-01     50     NaN
5 2026-01-06     15     NaN
6 2026-01-08      1  517.03
```

Pięć z siedmiu zakupów bez ceny — bo tego dnia NBP nic nie opublikował. Chcemy
reguły: **weź ostatnią cenę opublikowaną w dniu zakupu albo wcześniej**. Robi to
dokładnie `pd.merge_asof`:

```python
wycena = pd.merge_asof(zakupy, zloto, on="data")
wycena["wartosc"] = wycena["gramy"] * wycena["cena"]
wycena
```

```text
        data  gramy    cena   wartosc
0 2025-12-22     10  500.23   5002.30
1 2025-12-24      5  509.15   2545.75
2 2025-12-26     20  509.15  10183.00
3 2025-12-28      2  509.15   1018.30
4 2026-01-01     50  505.12  25256.00
5 2026-01-06     15  507.01   7605.15
6 2026-01-08      1  517.03    517.03
```

Zakupy z 24, 26 i 28 grudnia dostały cenę z 23 grudnia, a zakup z 1 stycznia —
cenę z 31 grudnia. Dwa warunki, bez których `merge_asof` nie zadziała:

1. kolumna, po której łączysz, musi mieć **ten sam typ** w obu tabelach — tu
   obie są datami (`pd.to_datetime`), a nie tekstem,
2. obie tabele muszą być **posortowane** po tej kolumnie — w razie czego
   `.sort_values("data")`.

!!! warning "Pierwszy dzień zakresu"

    `merge_asof` szuka ceny *wcześniej*. Jeśli pierwszy zakup przypada na dzień
    przed pierwszym notowaniem w pobranej tabeli, nie ma czego wziąć — wyjdzie
    `NaN`. Dlatego notowania pobiera się z kilkudniowym zapasem **przed** początkiem
    okresu, który analizujesz.

---

## Ściągawka

| Chcę… | Kod |
|---|---|
| CSV z polskiego systemu | `pd.read_csv("plik.csv", encoding="cp1250", sep=";", decimal=",")` |
| pominąć wiersze nad tabelą | `pd.read_csv(..., skiprows=2)` |
| dane z internetu | `pd.read_csv("https://...")` |
| sprawdzić typy kolumn | `df.dtypes` |
| plik JSON → lista słowników | `json.load(open("plik.json", encoding="utf-8"))` |
| JSON z internetu → lista słowników | `requests.get(URL).json()` |
| spłaszczyć zagnieżdżony JSON | `pd.json_normalize(lista)` |
| zapytanie do API | `odp = requests.get(URL)` |
| czy się udało | `odp.status_code` (200 = OK) |
| połączyć się z bazą SQLite | `con = sqlite3.connect("baza.db")` |
| zapisać tabelę do bazy | `df.to_sql("nazwa", con, index=False, if_exists="replace")` |
| zapytanie SQL → DataFrame | `pd.read_sql_query("SELECT ...", con)` |
| zmienić nazwy kolumn | `df.rename(columns={"stara": "nowa"})` |
| tekst na datę | `pd.to_datetime(df["kol"])` |
| połączyć tabele po kluczu | `a.merge(b, on="klucz", how="left")` |
| połączyć „ostatnią wartością wcześniej" | `pd.merge_asof(a, b, on="data")` |
| czy wartość występuje w innej kolumnie | `a["data"].isin(b["data"])` |

---

## Zadanie 1 — Sprzedaż zagraniczna (12 pkt)

**Czas:** ok. 55 minut
**Środowisko:** Google Colab
**Oddanie:** notatnik `.ipynb` na Moodle

Sklep internetowy „Gadżet24" sprzedaje w Polsce i za granicą. Klienci zagraniczni
płacą w swojej walucie, więc w systemie sprzedażowym wszystkie ceny są w różnych
walutach, a zarząd chce znać przychód **w złotówkach**. Masz do dyspozycji trzy
źródła: eksport zamówień ze starego systemu (CSV), bazę klientów (JSON) i kursy
walut z API NBP. Twoim zadaniem jest skleić je w jedną tabelę.

Dane obejmują cały **rok 2025**:

- :material-file-delimited: [**zamowienia_2025.csv**](dane/zamowienia_2025.csv) —
  4000 zamówień
- :material-code-json: [**klienci.json**](dane/klienci.json) — 400 klientów

| Kolumna | Co oznacza |
|---|---|
| `zamowienie_id` | numer zamówienia |
| `data` | data złożenia zamówienia |
| `klient_id` | identyfikator klienta (ten sam co w `klienci.json`) |
| `produkt`, `kategoria` | co kupiono |
| `sztuki` | ile sztuk |
| `cena` | cena jednej sztuki **w walucie klienta** |
| `waluta` | PLN, EUR, USD, GBP, CHF albo CZK |

Rób części po kolei i **pokaż mi wynik każdej z nich, gdy będzie gotowa** —
punktacja zależy od tego, ile zdążysz na zajęciach.

### Część 1 — Dwa pliki, jedna tabela (ok. 25 min)

Adresy plików:

```python
URL_ZAMOWIENIA = "https://kubzal.github.io/wsb-merito/semestr_zimowy_2026_27/metody_analizy_przetwarzania_i_wizualizacji_danych/dane/zamowienia_2025.csv"
URL_KLIENCI = "https://kubzal.github.io/wsb-merito/semestr_zimowy_2026_27/metody_analizy_przetwarzania_i_wizualizacji_danych/dane/klienci.json"
```

1. Wczytaj zamówienia do DataFrame'u `zamowienia`. Plik pochodzi ze starego
   systemu — zanim zaczniesz zgadywać parametry, otwórz go w przeglądarce
   i zobacz, jak wygląda. Masz dostać **4000 wierszy i 8 kolumn**, a kolumna
   `cena` ma być liczbą.
2. Wczytaj klientów do DataFrame'u `klienci` tak, żeby miasto i kraj były
   osobnymi kolumnami (**400 wierszy, 6 kolumn**).
3. Połącz obie tabele w jedną o nazwie `df` (każde zamówienie ma dostać dane
   swojego klienta) i policz dla każdego kraju: **liczbę zamówień** i **liczbę
   różnych klientów**. W komórce tekstowej odpowiedz jednym zdaniem: dlaczego
   na tej tabeli nie można jeszcze policzyć łącznego przychodu sklepu?

Komórka sprawdzająca — skopiuj i uruchom:

```python
print(f"zamówienia:  {zamowienia.shape}   (powinno być (4000, 8))")
print(f"cena:        {zamowienia['cena'].dtype}   (powinno być float64)")
print(f"sztuk:       {zamowienia['sztuki'].sum()}   (powinno być 6678)")
print(f"klienci:     {klienci.shape}   (powinno być (400, 6))")
print(f"po złączeniu: {df.shape}   (powinno być (4000, 13))")
print(f"zamówień z Polski: {(df['adres.kraj'] == 'Polska').sum()}   (powinno być 2012)")
```

### Część 2 — Przeliczenie na złotówki (ok. 30 min)

Każdy liczy na **swojej walucie i swoim miesiącu** 2025 roku. Wpisz swój numer
albumu i uruchom komórkę:

```python
NR_ALBUMU = 123456   # ← wpisz swój numer albumu

WALUTY = ["EUR", "USD", "GBP", "CHF", "CZK"]
WALUTA = WALUTY[NR_ALBUMU % 5]
MIESIAC = NR_ALBUMU % 12 + 1

# kursy pobieramy z tygodniowym zapasem przed początkiem miesiąca
OD = (pd.Timestamp(2025, MIESIAC, 1) - pd.Timedelta(days=7)).strftime("%Y-%m-%d")
DO = (pd.Timestamp(2025, MIESIAC, 1) + pd.offsets.MonthEnd()).strftime("%Y-%m-%d")

print(WALUTA, MIESIAC, OD, DO)
```

```text
USD 1 2024-12-25 2025-01-31
```

1. Pobierz z API NBP kursy średnie swojej waluty od `OD` do `DO` (adres jest
   w ramce „Kursy walut z NBP" w materiale). Zbuduj z nich DataFrame `kursy`
   z dwiema kolumnami: `data` (typ daty) i `kurs`.
2. Wybierz z `df` zamówienia w Twojej walucie złożone w Twoim miesiącu
   i dołącz do każdego kurs NBP — z dnia zamówienia, a jeśli tego dnia nie było
   notowania (weekend, święto), z ostatniego wcześniejszego dnia. Wynik zapisz
   jako `wynik`.
3. Dodaj kolumnę `wartosc_pln` (sztuki × cena × kurs). W komórce tekstowej podaj:
   **łączny przychód w PLN** z Twojego wariantu oraz **ile zamówień** przypadło
   na dzień, w którym NBP nie opublikował kursu.

Komórka sprawdzająca:

```python
print(f"wariant:      {WALUTA}, miesiąc {MIESIAC}")
print(f"notowań:      {len(kursy)}, od {kursy['data'].min():%Y-%m-%d}")
print(f"zamówień:     {len(wynik)}")
print(f"braki kursu:  {wynik['kurs'].isna().sum()}   (powinno być 0)")
print(f"kurs:         {wynik['kurs'].min():.4f} – {wynik['kurs'].max():.4f}")
print(f"przychód PLN: {wynik['wartosc_pln'].sum():,.2f}")
```

### Wskazówki

- Plik zamówień nie chce się wczytać? Wróć do ramki „Jak czytać błędy przy
  wczytywaniu" i poprawiaj **po jednym** parametrze naraz.
- `pd.json_normalize` potrzebuje listy słowników, a nie adresu. Plik JSON
  z internetu zamienisz na listę przez `requests.get(URL_KLIENCI).json()`.
- Część 1, punkt 3: grupowanie po kraju i dwie różne funkcje naraz —
  `df.groupby("adres.kraj").agg(zamowienia=("zamowienie_id", "count"), klienci=("klient_id", "nunique"))`.
- Część 2, punkt 1: kursy są w kluczu `"rates"` odpowiedzi. Kolumny
  `effectiveDate` i `mid` przemianuj przez `rename(columns={...})`. Jeśli
  dostajesz kod `404` albo `400`, wypisz adres (`print(url)`) i sprawdź go
  w przeglądarce.
- Część 2, punkt 2: data w `df` jest tekstem — przed filtrowaniem po miesiącu
  (`.dt.month`) i przed `merge_asof` zamień ją na datę. Pamiętaj
  o sortowaniu po dacie.
- `wynik["kurs"]` ma braki? Sprawdź, czy pobrałeś kursy od `OD`, a nie od
  pierwszego dnia miesiąca — to dokładnie problem z ramki „Pierwszy dzień zakresu".
- Zamówienia z dni bez notowania to te, których daty **nie ma** wśród dat
  w `kursy` — `~wynik["data"].isin(kursy["data"])`.
- Wynik się nie zgadza? **Runtime → Restart and run all** — bardzo często winna
  jest komórka uruchomiona dwa razy albo wcale.

### Punktacja (12 pkt)

| Punkty | Za co |
|---|---|
| **12 pkt** | obie części zrobione **na zajęciach** i pokazane prowadzącemu |
| **8 pkt** | jedna z dwóch części zrobiona **na zajęciach** i pokazana prowadzącemu |
| **4 pkt** | zadanie dokończone po zajęciach i wysłane na Moodle **w terminie** |
| **0 pkt** | brak rozwiązania albo wysłane po terminie |

Termin przesłania na Moodle: **2026-11-21** (dzień przed kolejnymi zajęciami).

!!! danger "Serio, róbcie to na zajęciach"

    Ta sama godzina pracy jest warta 12 pkt tu i teraz albo 4 pkt w domu. Mamy
    tylko cztery zadania w całym semestrze, więc każde z nich to 12% oceny
    końcowej. Jestem na sali po to, żeby Wam pomagać — korzystajcie z tego,
    zamiast walczyć z tym samym problemem wieczorem w domu.
