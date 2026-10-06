# Zajęcia 2

## Wczytywanie danych i czyszczenie

Na poprzednich zajęciach dane były grzeczne: plik CSV z przecinkami, każda kolumna wypełniona,
liczby zapisane jak liczby. W prawdziwym życiu tak nie jest prawie nigdy. Plik
przychodzi z Excela z separatorem średnikiem, ceny mają dopisane „zł", połowa
miast jest wpisana małymi literami, a ktoś zamówił −3 sztuki regału.

Dziś zajmujemy się etapem, który w każdym projekcie zajmuje najwięcej czasu —
**czyszczeniem danych**. Nie jest efektowny, ale bez niego każdy wykres i każdy
model będzie liczony na śmieciach.

Po dzisiejszych zajęciach będziesz w stanie:

- wczytać dane z CSV z „polskimi" ustawieniami, z internetu, z Excela i z JSON-a,
- zdiagnozować, co jest nie tak ze zbiorem: braki, duplikaty, złe typy, literówki,
- ujednolicić tekst, zamienić „1 299,00 zł" na liczbę i tekst na datę,
- świadomie zdecydować, co zrobić z brakami i wartościami niemożliwymi,
- zapisać wyczyszczony zbiór do pliku.

!!! info "Gdzie jesteśmy"

    Na pierwszych zajęciach rozpisaliśmy workflow projektu. Dziś robimy punkt 2
    (zebranie danych) porządnie i wchodzimy w punkt 3 — czyszczenie. A zbiór, który
    wyczyścisz w zadaniu, za godzinę trafi do modelu na
    [Uczeniu głębokim](../uczenie_glebokie/zajecia_2.md).

---

## Plan na dziś

| Blok | Czas |
|---|---|
| Skąd biorą się dane — CSV, internet, Excel, JSON | 10 min |
| Diagnoza — co jest nie tak z tym zbiorem | 7 min |
| Duplikaty, tekst, liczby zapisane jako tekst, daty | 15 min |
| Braki danych i wartości niemożliwe | 10 min |
| Zapis wyniku i ściągawka | 3 min |
| **Zadanie 2** | **45 min** |

---

## Skąd biorą się dane

Na poprzednich zajęciach wystarczyło `pd.read_csv("plik.csv")`. Dziś zobaczysz,
że to samo polecenie potrafi dużo więcej — i że do innych formatów pandas ma
bliźniacze funkcje `read_...`.

### Dane, na których pracujemy

Wracamy do sklepu z pierwszych zajęć. Tym razem dostaliśmy eksport zamówień
z systemu sprzedażowego. Uruchom komórkę — utworzy plik `zamowienia.csv`:

```python
csv = """id;data;klient;miasto;produkt;cena;sztuki
1001;2026-09-01;Anna Kowalska;Warszawa;Laptop Dell;3 499,00 zł;1
1002;2026-09-01;Jan Nowak; kraków;Słuchawki JBL;299,00 zł;2
1003;2026-09-02;Piotr Wiśniewski;GDAŃSK;Czajnik Zelmer;149,00 zł;
1004;2026-09-02;Maria Lewandowska;Warszawa;Ekspres do kawy;899,00 zł;1
1002;2026-09-01;Jan Nowak; kraków;Słuchawki JBL;299,00 zł;2
1005;2026-09-03;Tomasz Zieliński;warszawa ;Monitor LG;1 099,00 zł;2
1006;2026-09-03;;Kraków;Lampka LED;89,00 zł;5
1007;2026-09-04;Katarzyna Wójcik;Gdańsk;Fotel biurowy;1 199,00 zł;1
1008;2026-09-05;Michał Kamiński;Kraków;Regał Ikea;399,00 zł;-3
1009;2026-09-05;Agnieszka Kaczmarek;Warszawa;Smartfon Xiaomi;1 299,00 zł;1
1010;2026-09-06;Paweł Mazur;Gdańsk;Mikrofalówka Amica;529,00 zł;
1007;2026-09-04;Katarzyna Wójcik;Gdańsk;Fotel biurowy;1 199,00 zł;1
"""

with open("zamowienia.csv", "w", encoding="utf-8") as f:
    f.write(csv)
```

Przyjrzyj się tym danym przez chwilę. Ile problemów widzisz gołym okiem? Za
chwilę znajdziemy je wszystkie — tyle że kodem, a nie wzrokiem, bo w prawdziwym
pliku będzie ich nie 12 wierszy, a 120 tysięcy.

### CSV z niespodzianką

Spróbujmy wczytać plik tak jak na poprzednich zajęciach:

```python
import pandas as pd

pd.read_csv("zamowienia.csv").shape
```

```text
(12, 1)
```

Jedna kolumna? pandas domyślnie dzieli wiersze po **przecinku**, a ten plik ma
separator **średnik**. Cały wiersz wylądował więc w jednej kolumnie — i to
w dodatku pocięty w złym miejscu, bo przecinek w „3 499,00 zł" pandas też uznał
za separator.

To klasyka: polski Excel zapisuje CSV ze średnikami, bo przecinek jest u nas
separatorem dziesiętnym. Rozwiązanie to jeden parametr:

```python
df = pd.read_csv("zamowienia.csv", sep=";")
df
```

```text
      id        data               klient     miasto             produkt         cena  sztuki
0   1001  2026-09-01        Anna Kowalska   Warszawa         Laptop Dell  3 499,00 zł     1.0
1   1002  2026-09-01            Jan Nowak     kraków       Słuchawki JBL    299,00 zł     2.0
2   1003  2026-09-02     Piotr Wiśniewski     GDAŃSK      Czajnik Zelmer    149,00 zł     NaN
3   1004  2026-09-02    Maria Lewandowska   Warszawa     Ekspres do kawy    899,00 zł     1.0
4   1002  2026-09-01            Jan Nowak     kraków       Słuchawki JBL    299,00 zł     2.0
5   1005  2026-09-03     Tomasz Zieliński  warszawa           Monitor LG  1 099,00 zł     2.0
6   1006  2026-09-03                  NaN     Kraków          Lampka LED     89,00 zł     5.0
7   1007  2026-09-04     Katarzyna Wójcik     Gdańsk       Fotel biurowy  1 199,00 zł     1.0
8   1008  2026-09-05      Michał Kamiński     Kraków          Regał Ikea    399,00 zł    -3.0
9   1009  2026-09-05  Agnieszka Kaczmarek   Warszawa     Smartfon Xiaomi  1 299,00 zł     1.0
10  1010  2026-09-06          Paweł Mazur     Gdańsk  Mikrofalówka Amica    529,00 zł     NaN
11  1007  2026-09-04     Katarzyna Wójcik     Gdańsk       Fotel biurowy  1 199,00 zł     1.0
```

Teraz wygląda jak tabela. **Zasada:** po każdym wczytaniu sprawdź `df.shape`
albo `df.head()`. Jeśli wyszła jedna kolumna — to prawie zawsze separator.

Parametry `read_csv`, które ratują życie najczęściej:

| Parametr | Kiedy | Przykład |
|---|---|---|
| `sep` | separator inny niż przecinek | `sep=";"`, `sep="\t"` |
| `decimal` | liczby zapisane z przecinkiem: `12,5` | `decimal=","` |
| `thousands` | separator tysięcy: `1 299` | `thousands=" "` |
| `encoding` | krzaczki zamiast polskich liter | `encoding="cp1250"` |
| `skiprows` | nad tabelą są wiersze z opisem | `skiprows=3` |
| `nrows` | chcesz tylko podejrzeć ogromny plik | `nrows=1000` |

!!! tip "Krzaczki zamiast „ąę"?"

    Jeśli zamiast „Gdańsk" widzisz „GdaÅ„sk" albo dostajesz `UnicodeDecodeError`,
    plik nie jest zapisany w UTF-8. Starsze polskie systemy i Excel na Windowsie
    często używają kodowania `cp1250` — spróbuj `encoding="cp1250"`.

### Dane prosto z internetu

`read_csv` przyjmuje nie tylko nazwę pliku, ale też **adres URL**. Nie trzeba
niczego pobierać ręcznie:

```python
URL = "https://kubzal.github.io/wsb-merito/semestr_zimowy_2026_27/data_science/dane/sprzedaz_2025.csv"

sprzedaz = pd.read_csv(URL)
sprzedaz.head(3)
```

```text
         data    region    kategoria             produkt  sztuki    cena  koszt
0  2025-01-01    Wschód  Elektronika       Tablet Lenovo       6   899.0  690.0
1  2025-01-01    Północ          AGD  Mikrofalówka Amica       5   529.0  385.0
2  2025-01-02  Południe        Meble       Fotel biurowy       4  1199.0  760.0
```

To najwygodniejszy sposób pracy w Colabie — notatnik zadziała u każdego, kto go
otworzy, bez wgrywania plików.

### Excel

Excel ma własną funkcję. Działa i z plikiem lokalnym, i z adresem URL:

```python
URL_XLSX = "https://kubzal.github.io/wsb-merito/semestr_zimowy_2026_27/data_science/dane/sprzedaz_2025.xlsx"

sprzedaz_xlsx = pd.read_excel(URL_XLSX, sheet_name="dane")
sprzedaz_xlsx.head(3)
```

```text
        data    region    kategoria             produkt  sztuki  cena  koszt
0 2025-01-01    Wschód  Elektronika       Tablet Lenovo       6   899    690
1 2025-01-01    Północ          AGD  Mikrofalówka Amica       5   529    385
2 2025-01-02  Południe        Meble       Fotel biurowy       4  1199    760
```

- `sheet_name` — nazwa albo numer arkusza (`0` to pierwszy). Bez tego parametru
  pandas weźmie pierwszy arkusz.
- Jakie arkusze są w pliku? `pd.ExcelFile(URL_XLSX).sheet_names`.

Zwróć uwagę, że z Excela daty przyszły od razu jako daty — Excel przechowuje typy,
CSV to tylko tekst. Do daty z CSV wrócimy za chwilę.

### JSON

JSON to format, w którym dane zwracają **API** — serwisy internetowe, aplikacje,
systemy firmowe. Wygląda jak lista pythonowych słowników:

```python
import json

klienci = [
    {"id": 1, "imie": "Anna", "miasto": "Warszawa", "punkty": 120, "newsletter": True},
    {"id": 2, "imie": "Jan", "miasto": "Kraków", "punkty": 45, "newsletter": False},
    {"id": 3, "imie": "Piotr", "miasto": "Gdańsk", "punkty": 300, "newsletter": True},
]

with open("klienci.json", "w", encoding="utf-8") as f:
    json.dump(klienci, f, ensure_ascii=False, indent=2)

pd.read_json("klienci.json")
```

```text
   id   imie    miasto  punkty  newsletter
0   1   Anna  Warszawa     120        True
1   2    Jan    Kraków      45       False
2   3  Piotr    Gdańsk     300        True
```

Każdy słownik to wiersz, każdy klucz to kolumna. Jeśli JSON jest zagnieżdżony
(słownik w słowniku), przydaje się `pd.json_normalize(...)`, które „spłaszcza"
go do tabeli — ale to już na wypadek, gdy na takie dane traficie.

!!! note "Rodzina `read_...`"

    `read_csv`, `read_excel`, `read_json`, `read_parquet`, `read_sql`,
    `read_html`… Wszystkie zwracają ten sam `DataFrame`. Jak raz nauczysz się
    pracować z tabelą, źródło przestaje mieć znaczenie.

---

## Diagnoza — co jest nie tak

Zanim cokolwiek naprawisz, musisz wiedzieć, **co** jest zepsute. Do czterech
poleceń z poprzednich zajęć (`shape`, `head`, `info`, `describe`) dokładamy trzy
kolejne.

### Typy i braki — `info()`

```python
df.info()
```

```text
<class 'pandas.DataFrame'>
RangeIndex: 12 entries, 0 to 11
Data columns (total 7 columns):
 #   Column   Non-Null Count  Dtype
---  ------   --------------  -----
 0   id       12 non-null     int64
 1   data     12 non-null     str
 2   klient   11 non-null     str
 3   miasto   12 non-null     str
 4   produkt  12 non-null     str
 5   cena     12 non-null     str
 6   sztuki   10 non-null     float64
dtypes: float64(1), int64(1), str(5)
memory usage: 804.0 bytes
```

Trzy sygnały ostrzegawcze:

1. `cena` ma typ `str` — to tekst, nie liczba. Nie policzysz z niej średniej.
2. `data` też jest tekstem. Nie przefiltrujesz po zakresie dat.
3. `sztuki` to `float64`, choć sztuki są całkowite. To znak, że w kolumnie są
   **braki** — `NaN` (*not a number*) jest liczbą zmiennoprzecinkową, więc cała
   kolumna musiała się do niej dostosować. Potwierdza to `10 non-null` przy
   12 wierszach.

### Ile braków — `isna().sum()`

```python
df.isna().sum()
```

```text
id         0
data       0
klient     1
miasto     0
produkt    0
cena       0
sztuki     2
dtype: int64
```

`isna()` zamienia tabelę na `True`/`False` (czy brak?), a `sum()` zlicza `True`
w każdej kolumnie — dokładnie ten sam trik co z maską w numpy.

### Duplikaty — `duplicated()`

```python
print(df.duplicated().sum())
```

```text
2
```

Dwa wiersze to dokładne kopie wcześniejszych. Które? `keep=False` oznacza
„pokaż wszystkie egzemplarze, łącznie z oryginałem":

```python
df[df.duplicated(keep=False)]
```

```text
      id        data            klient   miasto        produkt         cena  sztuki
1   1002  2026-09-01         Jan Nowak   kraków  Słuchawki JBL    299,00 zł     2.0
4   1002  2026-09-01         Jan Nowak   kraków  Słuchawki JBL    299,00 zł     2.0
7   1007  2026-09-04  Katarzyna Wójcik   Gdańsk  Fotel biurowy  1 199,00 zł     1.0
11  1007  2026-09-04  Katarzyna Wójcik   Gdańsk  Fotel biurowy  1 199,00 zł     1.0
```

To samo zamówienie wyeksportowało się dwa razy — typowy błąd przy łączeniu
eksportów z kilku dni.

### Warianty tekstu — `value_counts()` i `unique()`

```python
df["miasto"].value_counts()
```

```text
miasto
Warszawa     3
Gdańsk       3
 kraków      2
Kraków       2
GDAŃSK       1
warszawa     1
Name: count, dtype: int64
```

Dla człowieka to trzy miasta. Dla pandas — sześć różnych wartości. `" kraków"`
ze spacją na początku i `"warszawa "` ze spacją na końcu to dla komputera zupełnie
inne napisy niż `"Kraków"` i `"Warszawa"`.

!!! warning "Spacje są niewidzialne"

    W `value_counts()` spacji na końcu nie widać. Jeśli dwie wartości wyglądają
    identycznie, a liczą się osobno — użyj `df["miasto"].unique()`. Pokazuje
    wartości w apostrofach, więc `'warszawa '` od razu się zdradza.

Podsumowanie diagnozy: duplikaty, braki w dwóch kolumnach, liczby i daty jako
tekst, niespójne miasta. Do tego jeszcze −3 sztuki, ale to wyłapiemy za chwilę.
Naprawiamy po kolei.

---

## Czyszczenie

### Duplikaty

```python
df = df.drop_duplicates()
df.shape
```

```text
(10, 7)
```

Z 12 wierszy zostało 10. Zwróć uwagę na `df = ...` — większość metod pandas
**nie zmienia** tabeli w miejscu, tylko zwraca nową. Jeśli nie przypiszesz wyniku,
duplikaty zostaną.

!!! note "Duplikat to nie zawsze błąd"

    Dwa identyczne wiersze w tabeli zamówień to prawie na pewno błąd eksportu.
    Ale dwa identyczne wiersze w tabeli pomiarów temperatury mogą być po prostu
    dwoma dniami z tą samą pogodą. Zanim usuniesz, zastanów się, czy w tych
    danych duplikat **może** być prawdziwy. Tu pomaga kolumna `id` — dwa
    zamówienia o tym samym numerze to na pewno kopia.

### Tekst — `.str`

Na kolumnie tekstowej po `.str` masz dostęp do wszystkich metod napisów z Pythona,
wykonywanych na każdym wierszu naraz:

```python
df["miasto"].str.strip()           # usuwa spacje z początku i końca
df["miasto"].str.lower()           # małe litery
df["miasto"].str.title()           # Każde Słowo Wielką Literą
```

Metody można łączyć w łańcuch — każda działa na wyniku poprzedniej:

```python
df["miasto"] = df["miasto"].str.strip().str.title()
df["miasto"].value_counts()
```

```text
miasto
Warszawa    4
Kraków      3
Gdańsk      3
Name: count, dtype: int64
```

Sześć wariantów zamieniło się w trzy miasta.

### Liczby zapisane jako tekst

Kolumna `cena` wygląda tak:

```python
df["cena"].head(3)
```

```text
0    3 499,00 zł
1      299,00 zł
2      149,00 zł
Name: cena, dtype: str
```

Żeby zamienić to na liczbę, trzeba usunąć wszystko, czego Python nie rozumie,
i dopiero wtedy zmienić typ przez `astype(float)`:

```python
df["cena"] = (
    df["cena"]
    .str.replace(" zł", "")     # "3 499,00 zł" → "3 499,00"
    .str.replace(" ", "")       # "3 499,00"    → "3499,00"
    .str.replace(",", ".")      # "3499,00"     → "3499.00"
    .astype(float)              # "3499.00"     → 3499.0
)
df["cena"].head(3)
```

```text
0    3499.0
1     299.0
2     149.0
Name: cena, dtype: float64
```

Nawias wokół całości pozwala rozpisać łańcuch na kilka linijek — dzięki temu
przy każdym kroku zmieści się komentarz.

!!! tip "Kolejność ma znaczenie"

    Najpierw usuń `" zł"` (ze spacją), dopiero potem pozostałe spacje. W odwrotnej
    kolejności zostałoby `"zł"` przyklejone do liczby. Jeśli `astype(float)`
    rzuca błędem `could not convert string to float: '...'` — w apostrofach
    zobaczysz dokładnie ten napis, którego nie udało się przerobić.

### Daty

Tekst `"2026-09-01"` zamieniamy na prawdziwą datę funkcją `pd.to_datetime`:

```python
df["data"] = pd.to_datetime(df["data"])
```

Od teraz pandas wie, że to daty. Można je porównywać i wyciągać z nich części
przez `.dt`:

```python
df[df["data"] >= "2026-09-05"][["id", "data", "produkt"]]
```

```text
      id       data             produkt
8   1008 2026-09-05          Regał Ikea
9   1009 2026-09-05     Smartfon Xiaomi
10  1010 2026-09-06  Mikrofalówka Amica
```

```python
df["data"].dt.day        # dzień miesiąca
df["data"].dt.month      # miesiąc
df["data"].dt.dayofweek  # dzień tygodnia: 0 = poniedziałek, 6 = niedziela
```

Gdyby daty były zapisane po polsku, np. `"01.09.2026"`, trzeba podpowiedzieć
format: `pd.to_datetime(df["data"], format="%d.%m.%Y")`.

---

## Braki danych i wartości niemożliwe

### Wartości niemożliwe

Brak to puste pole. Ale gorsze od braku jest pole wypełnione **bzdurą** — bo nie
widać jej w `isna()`. Szukamy ich filtrem, z wiedzą o tym, co dane oznaczają:

```python
df[df["sztuki"] < 0]
```

```text
     id       data           klient  miasto     produkt   cena  sztuki
8  1008 2026-09-05  Michał Kamiński  Kraków  Regał Ikea  399.0    -3.0
```

Nie da się zamówić −3 regałów. Najpewniej ktoś wpisał minus przez pomyłkę albo
system zapisał zwrot jako zamówienie. Nie wiemy, jaka jest prawdziwa wartość —
więc uczciwie jest potraktować ją jak **brak**:

```python
import numpy as np

df.loc[df["sztuki"] < 0, "sztuki"] = np.nan
```

`df.loc[warunek, "kolumna"] = wartość` czytamy: „w wierszach spełniających
warunek, w tej kolumnie, wpisz tę wartość". To podstawowy sposób na zmianę
wybranych komórek tabeli.

Teraz kolumna `sztuki` ma trzy braki: dwa oryginalne i jeden, który sami
oznaczyliśmy.

!!! tip "Skąd wiadomo, co jest niemożliwe?"

    Z wiedzy o dziedzinie, nie z kodu. Pandas nie wie, że wiek 230 lat albo
    temperatura −90 °C w Warszawie to bzdura. `describe()` pomaga je znaleźć —
    patrz na `min` i `max` każdej kolumny i zadaj sobie pytanie, czy to
    w ogóle możliwe.

### Usunąć czy uzupełnić?

Z brakami można zrobić dwie rzeczy.

**Usunąć wiersze** — `dropna()`:

```python
df.dropna()                       # usuń wiersz, jeśli brakuje czegokolwiek
df.dropna(subset=["klient"])      # usuń tylko, jeśli brakuje klienta
```

Prosto, ale kosztownie: `df.dropna()` na naszej tabeli wyrzuciłoby 4 z 10
zamówień — prawie połowę danych — z powodu jednej pustej komórki w każdym z nich.

**Uzupełnić** — `fillna()`:

```python
df["sztuki"] = df["sztuki"].fillna(df["sztuki"].median()).astype(int)
df["klient"] = df["klient"].fillna("nieznany")
```

- Liczby najczęściej uzupełniamy **medianą** — z pierwszych zajęć pamiętasz, że
  średnią łatwo rozciągnąć jedną wartością odstającą, a medianę dużo trudniej.
  Tu mediana to 1 sztuka.
- `astype(int)` — kiedy braków już nie ma, kolumna może wrócić do liczb
  całkowitych.
- Tekst uzupełniamy wartością typu `"nieznany"` albo najczęstszą wartością
  w kolumnie.

```python
df.isna().sum()
```

```text
id         0
data       0
klient     0
miasto     0
produkt    0
cena       0
sztuki     0
dtype: int64
```

Czysto.

Kiedy co wybrać?

| Sytuacja | Co zrobić |
|---|---|
| braków jest mało, a wiersz bez tej wartości jest bezużyteczny | `dropna(subset=[...])` |
| braków jest sporo, a kolumna liczbowa | `fillna(mediana)` |
| kolumna tekstowa | `fillna("nieznany")` albo najczęstsza wartość |
| brakuje tego, co chcemy **przewidywać** (zmiennej celu) | **zawsze** usuwamy wiersz |

Ostatni wiersz jest ważny na Uczeniu głębokim: jeśli nie wiemy, czy klient
zrezygnował, to nie możemy na nim uczyć modelu. Wpisanie tam mediany albo
najczęstszej wartości to zmyślanie odpowiedzi, z których model ma się uczyć.

!!! warning "Uzupełnianie to też zgadywanie"

    Każda uzupełniona wartość to Twoje założenie, a nie fakt. Przy kilku procentach
    braków to zwykle w porządku. Gdy brakuje połowy kolumny — lepiej się zastanowić,
    czy ta kolumna w ogóle się do czegoś nadaje.

---

## Zapis wyniku

Na koniec liczymy wartość zamówienia — co do niedawna było niemożliwe, bo cena
była tekstem — i zapisujemy czysty zbiór:

```python
df["wartosc"] = df["cena"] * df["sztuki"]
df
```

```text
      id       data               klient    miasto             produkt    cena  sztuki  wartosc
0   1001 2026-09-01        Anna Kowalska  Warszawa         Laptop Dell  3499.0       1   3499.0
1   1002 2026-09-01            Jan Nowak    Kraków       Słuchawki JBL   299.0       2    598.0
2   1003 2026-09-02     Piotr Wiśniewski    Gdańsk      Czajnik Zelmer   149.0       1    149.0
3   1004 2026-09-02    Maria Lewandowska  Warszawa     Ekspres do kawy   899.0       1    899.0
5   1005 2026-09-03     Tomasz Zieliński  Warszawa          Monitor LG  1099.0       2   2198.0
6   1006 2026-09-03             nieznany    Kraków          Lampka LED    89.0       5    445.0
7   1007 2026-09-04     Katarzyna Wójcik    Gdańsk       Fotel biurowy  1199.0       1   1199.0
8   1008 2026-09-05      Michał Kamiński    Kraków          Regał Ikea   399.0       1    399.0
9   1009 2026-09-05  Agnieszka Kaczmarek  Warszawa     Smartfon Xiaomi  1299.0       1   1299.0
10  1010 2026-09-06          Paweł Mazur    Gdańsk  Mikrofalówka Amica   529.0       1    529.0
```

```python
df.to_csv("zamowienia_czyste.csv", index=False)
```

`index=False` sprawia, że indeks (0, 1, 2, 3, 5, …) nie trafi do pliku jako
dodatkowa, bezsensowna kolumna. Ustawiaj go praktycznie zawsze.

Gotowy plik znajdziesz w Colabie w panelu **Files** (ikona folderu po lewej) —
prawy przycisk → *Download*.

!!! danger "Nigdy nie nadpisuj oryginału"

    Zapisuj wynik do **nowego** pliku (`zamowienia_czyste.csv`), a surowe dane
    zostaw w spokoju. Jeśli w czyszczeniu był błąd — a prędzej czy później będzie —
    poprawiasz kod i uruchamiasz go jeszcze raz od surowych danych. Dlatego
    czyszczenie robimy w notatniku, a nie ręcznie w Excelu.

### Ściągawka

| Chcę… | Kod |
|---|---|
| CSV ze średnikami | `pd.read_csv("plik.csv", sep=";")` |
| CSV z przecinkiem dziesiętnym | `pd.read_csv("plik.csv", sep=";", decimal=",")` |
| polskie znaki w starym pliku | `pd.read_csv("plik.csv", encoding="cp1250")` |
| dane z internetu | `pd.read_csv("https://...")` |
| Excel | `pd.read_excel("plik.xlsx", sheet_name="Arkusz1")` |
| JSON | `pd.read_json("plik.json")` |
| ile braków w kolumnach | `df.isna().sum()` |
| ile duplikatów | `df.duplicated().sum()` |
| usunąć duplikaty | `df = df.drop_duplicates()` |
| zobaczyć warianty tekstu | `df["kol"].unique()` |
| ujednolicić tekst | `df["kol"].str.strip().str.title()` |
| zamienić fragment tekstu | `df["kol"].str.replace(" zł", "")` |
| tekst na liczbę | `df["kol"].astype(float)` |
| tekst na datę | `pd.to_datetime(df["kol"])` |
| zamienić wartości na podstawie warunku | `df.loc[df["kol"] < 0, "kol"] = np.nan` |
| zamienić wartości według słownika | `df["kol"].map({"tak": 1, "nie": 0})` |
| usunąć wiersze z brakiem w kolumnie | `df.dropna(subset=["kol"])` |
| uzupełnić braki medianą | `df["kol"].fillna(df["kol"].median())` |
| sprawdzić, czy wartość w przedziale | `df["kol"].between(16, 90)` |
| zapisać do CSV | `df.to_csv("plik.csv", index=False)` |

---

## Zadanie 2 — Klienci siłowni (6 pkt)

**Czas:** maks. 45 minut
**Środowisko:** Google Colab
**Oddanie:** notatnik `.ipynb` na Moodle

Sieć siłowni „FitMerito" wyeksportowała ze swojego systemu dane o klientach.
Zarząd chce wiedzieć, którzy klienci rezygnują z karnetu — ale zanim ktokolwiek
zbuduje na tym model, ktoś musi te dane doprowadzić do porządku. Ten ktoś to Ty.

Kolumny w pliku:

| Kolumna | Znaczenie |
|---|---|
| `klient_id` | identyfikator klienta |
| `wiek` | wiek klienta w latach |
| `miasto` | miasto, w którym jest klub |
| `typ_karnetu` | Basic, Standard albo Premium |
| `oplata_mies` | miesięczna opłata za karnet |
| `staz_mies` | od ilu miesięcy klient ma karnet |
| `wizyty_mies` | średnia liczba wizyt w miesiącu |
| `zgloszenia` | ile razy klient zgłaszał reklamację lub skargę |
| `zrezygnowal` | czy klient zrezygnował z karnetu (`tak` / `nie`) |

Zadanie ma dwie części. Rób je po kolei i **pokaż mi wynik każdej części, gdy
będzie gotowa** — punktacja zależy od tego, ile zrobisz na zajęciach.

### Część 1 — diagnoza (ok. 20 min)

Dane są pod adresem:

```python
URL = "https://kubzal.github.io/wsb-merito/semestr_zimowy_2026_27/data_science_w_pythonie/dane/silownia_2026.csv"
```

1. Wczytaj plik do DataFrame'u o nazwie `df`. Upewnij się, że dostałeś
   **9 kolumn**, a nie jedną.
2. Sprawdź rozmiar zbioru i wyświetl pierwsze 5 wierszy.
3. Wyświetl typy kolumn. Która kolumna powinna być liczbą, a jest tekstem?
4. Policz, ile braków jest w każdej kolumnie.
5. Policz, ile jest zduplikowanych wierszy.
6. Sprawdź, jakie warianty zapisu mają kolumny `typ_karnetu`, `miasto`
   i `zrezygnowal`.
7. Wyświetl statystyki opisowe (`describe()`). Czy w którejś kolumnie liczbowej
   widzisz wartości, które **nie mogą** być prawdziwe?
8. W komórce tekstowej (Markdown) wypisz listę wszystkich problemów, które
   znalazłeś — każdy w jednym punkcie, z liczbą wierszy, których dotyczy (tam,
   gdzie da się ją policzyć).

### Część 2 — czyszczenie (ok. 25 min)

Wykonaj kroki po kolei, każdy w osobnej komórce. Po każdym kroku sprawdź, czy
zadziałał — `shape`, `unique()`, `isna().sum()` — zanim przejdziesz dalej:

1. Usuń zduplikowane wiersze.
2. Usuń wiersze, w których brakuje wartości `zrezygnowal` — to jest zmienna,
   którą za godzinę będziemy przewidywać.
3. Ujednolić kolumny `miasto` i `typ_karnetu`: bez spacji na początku i końcu,
   pierwsza litera wielka (`Warszawa`, `Premium`).
4. Zamień `oplata_mies` na liczbę (`"129,00 zł"` → `129.0`).
5. Zamień `zrezygnowal` na liczby: `1` dla „tak", `0` dla „nie". Uważaj na
   różne wielkości liter.
6. Wiek spoza przedziału **16–90 lat** potraktuj jak brak danych (zamień na
   `np.nan`), a następnie uzupełnij **wszystkie** braki w kolumnie `wiek`
   medianą i zamień kolumnę na liczby całkowite.
7. Zapisz wynik do pliku `silownia_czyste.csv` (bez indeksu).

Na koniec skopiuj i uruchom komórkę sprawdzającą:

```python
spr = pd.read_csv("silownia_czyste.csv")

print(f"wiersze:       {len(spr)}   (powinno być 1484)")
print(f"kolumny:       {spr.shape[1]}   (powinno być 9)")
print(f"duplikaty:     {spr.duplicated().sum()}   (powinno być 0)")
print(f"braki:         {spr.isna().sum().sum()}   (powinno być 0)")
print(f"typy karnetów: {sorted(spr['typ_karnetu'].unique().tolist())}")
print(f"miasta:        {sorted(spr['miasto'].unique().tolist())}")
print(f"opłata:        {spr['oplata_mies'].dtype}   (powinno być float64)")
print(f"zrezygnował:   {sorted(spr['zrezygnowal'].unique().tolist())}   (powinno być [0, 1])")
print(f"wiek:          {spr['wiek'].min()}–{spr['wiek'].max()}   (powinno być 16–69)")
print(f"% rezygnacji:  {spr['zrezygnowal'].mean() * 100:.1f}   (powinno być 23.5)")
```

Wszystkie wartości powinny zgadzać się z tymi w nawiasach. Typy karnetów
to dokładnie trzy wartości, miasta — pięć: Gdańsk, Kraków, Poznań, Warszawa,
Wrocław.

Na koniec dodaj komórkę tekstową i w dwóch–trzech zdaniach odpowiedz: **ile
wierszy straciłeś w trakcie czyszczenia i czy to dużo?** Który krok kosztował
najwięcej?

### Wskazówki

- Jedna kolumna po wczytaniu? Wróć do sekcji „CSV z niespodzianką".
- W `value_counts()` dla `typ_karnetu` zobaczysz dwa razy „Basic". To nie błąd
  pandas — w jednym z nich jest spacja. Sprawdź `unique()`.
- Punkt 5: najpierw ujednolić wielkość liter (`.str.lower()`), potem `.map(...)`.
  Jeśli po `map` pojawiają się `NaN`, to któryś wariant nie trafił do słownika.
- Punkt 6 to dokładnie ten sam wzorzec co −3 sztuki w materiale: `df.loc[...]`
  z warunkiem, potem `fillna`. Warunek „poza przedziałem" to zaprzeczenie
  `between` — w pandas zaprzeczenie zapisujemy znakiem `~`:
  `~df["wiek"].between(16, 90)`.
- Wynik testu się nie zgadza? Najpierw **Runtime → Restart and run all** —
  bardzo często winna jest komórka uruchomiona dwa razy albo wcale. Potem
  sprawdź, który wiersz testu odstaje, i wróć do odpowiadającego mu kroku.
- Za mało wierszy? Najczęstsza przyczyna to **usunięcie** wierszy z niemożliwym
  wiekiem zamiast zamiany wieku na brak.
- Plik `silownia_czyste.csv` pobierz z Colaba (panel **Files**) — przyda się na
  Uczeniu głębokim.

### Punktacja (6 pkt)

| Punkty | Za co |
|---|---|
| **6 pkt** | obie części zrobione **na zajęciach** i pokazane prowadzącemu |
| **4 pkt** | jedna z dwóch części zrobiona na zajęciach |
| **2 pkt** | zadanie dokończone po zajęciach i wysłane na Moodle **w terminie** |
| **0 pkt** | brak rozwiązania albo wysłane po terminie |

Termin przesłania na Moodle: **dzień przed kolejnymi zajęciami**.

!!! danger "Serio, róbcie to na zajęciach"

    Te same 45 minut pracy jest warte 6 pkt tu i teraz albo 2 pkt w domu. Przez
    cały semestr robi to różnicę 32 punktów, czyli całej oceny w górę lub w dół.
