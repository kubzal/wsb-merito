# Zajęcia 1

## Od Excela do Pythona — ta sama analiza, dwa narzędzia

Zaczynamy od rzeczy, którą większość z Was już zna: arkusza kalkulacyjnego.
Weźmiemy prawdziwie wyglądający zbiór danych sprzedażowych, odpowiemy na nim na
kilka pytań biznesowych — a potem zrobimy **dokładnie to samo** w Pythonie, na
tym samym pliku, żeby zobaczyć, że to nie jest inny świat, tylko inne narzędzie
do tej samej roboty.

Po dzisiejszych zajęciach będziesz w stanie:

- powiedzieć, czym zajmuje się Data Science i jak wygląda projekt analityczny,
- policzyć w Excelu przychód, koszt i marżę oraz podsumować dane formułami
  warunkowymi,
- zbudować tabelę przestawną i wykres, i **wyczytać z nich, co się stało w firmie**,
- zrobić to samo w Pythonie: wczytać plik, przefiltrować, pogrupować, narysować,
- ocenić, kiedy Excel wystarczy, a kiedy zaczyna przeszkadzać.

---

## Plan na dziś

| Blok | Czas |
|---|---|
| Czym jest Data Science | 15 min |
| Excel: dane, sortowanie, filtrowanie | 15 min |
| Excel: formuły i kolumny obliczeniowe | 20 min |
| Excel: tabela przestawna i wykres | 20 min |
| Przerwa | 10 min |
| To samo w Pythonie — Colab i pandas | 35 min |
| Excel kontra Python — podsumowanie | 5 min |
| **Zadanie 1** | **60 min** |

---

## Czym jest Data Science

Data Science to wyciąganie z danych odpowiedzi na pytania, na które nie da się
odpowiedzieć „na oko". Nie chodzi o to, żeby mieć dużo danych — chodzi o to, żeby
z nich coś **wynikało**.

Typowe pytania, które w firmie trafiają do analityka:

- Dlaczego sprzedaż spadła w ostatnim kwartale i czy to problem, czy sezonowość?
- Którzy klienci najprawdopodobniej odejdą do konkurencji w przyszłym miesiącu?
- Czy warto dać rabat 10%, czy zjemy nim całą marżę?
- Komu możemy bezpiecznie udzielić kredytu?

Zwróć uwagę, że w każdym z nich **najpierw jest pytanie, potem dane**. Odwrotna
kolejność — „mamy dane, zobaczmy co z nich wyjdzie" — kończy się zwykle ładnym
wykresem, z którego nic nie wynika.

### Kto się tym zajmuje

Nazwy stanowisk bywają mylące, ale podział jest mniej więcej taki:

| Rola | Czym się zajmuje | Typowe narzędzia |
|---|---|---|
| **Data Analyst** | opisuje, co się wydarzyło; raporty, dashboardy | Excel, SQL, Power BI |
| **Data Scientist** | prognozuje, co się wydarzy; modele, eksperymenty | Python, R, SQL |
| **Data Engineer** | buduje rurociągi, którymi płyną dane | SQL, Spark, chmura |

Na tym przedmiocie stoimy głównie w butach **analityka**, a na ostatnich zajęciach
wejdziemy jedną nogą w buty **data scientista**.

### Jak wygląda projekt analityczny

Prawie każdy projekt przechodzi przez te same etapy:

1. **Pytanie** — co konkretnie chcemy wiedzieć i jaką decyzję to zmieni.
2. **Zebranie danych** — pliki, baza, system sprzedażowy.
3. **Czyszczenie** — braki, duplikaty, literówki, złe formaty. Zwykle najdłuższy etap.
4. **Analiza** — statystyki, grupowania, wykresy.
5. **Modelowanie** — jeśli pytanie dotyczy przyszłości.
6. **Komunikacja** — wniosek i rekomendacja. Analiza, której nikt nie zrozumiał,
   nie istnieje.

!!! info "Gdzie jesteśmy dzisiaj"

    Dziś robimy punkty 1, 2, 4 i 6 — i to dwa razy: raz w Excelu, raz w Pythonie.
    Czyszczenie danych (3) to temat zajęć 3, a modelowanie (5) — zajęć 4.

!!! quote "Zasada, o której warto pamiętać przez cały semestr"

    Liczba bez interpretacji jest bezużyteczna. „Przychód wyniósł 3 835 357 zł" to
    nie jest wniosek. Wnioskiem jest dopiero: „przychód spadł o 10% i wiemy,
    który region za to odpowiada".

---

## Dane, na których pracujemy

Pracujemy na danych sprzedażowych fikcyjnej sieci sklepów RTV/AGD za **cały
2025 rok**. Każdy wiersz to jedna transakcja: co, gdzie, kiedy i za ile.

Pobierz plik w wygodnym dla siebie formacie — **to ten sam zbiór danych**:

- :material-microsoft-excel: [**sprzedaz_2025.xlsx**](dane/sprzedaz_2025.xlsx) — do Excela
- :material-file-delimited: [**sprzedaz_2025.csv**](dane/sprzedaz_2025.csv) — do Pythona

| Kolumna | Co oznacza |
|---|---|
| `data` | data transakcji (2025-01-01 … 2025-12-28) |
| `region` | region sprzedaży: Północ, Południe, Wschód, Zachód |
| `kategoria` | Elektronika, AGD, Meble |
| `produkt` | nazwa produktu |
| `sztuki` | ile sztuk sprzedano |
| `cena` | cena sprzedaży jednej sztuki (zł) |
| `koszt` | koszt zakupu jednej sztuki (zł) |

Zbiór ma **576 wierszy danych** — czyli w Excelu wiersze od 2 do 577, bo wiersz 1
to nagłówki.

!!! tip "Trzy pytania, na które dziś odpowiemy"

    1. Ile zarobiliśmy w 2025 roku i na czym zarabiamy najlepiej?
    2. **Zarząd twierdzi, że sprzedaż spadła w ostatnim kwartale. Czy to prawda?**
    3. Jeśli tak — co za to odpowiada?

---

## Część 1: Excel

### Pierwsze spojrzenie na dane

Otwórz `sprzedaz_2025.xlsx`. Zanim cokolwiek policzysz, zorientuj się, co masz
w ręku — to nawyk, który zostaje na całe życie zawodowe.

| Chcę… | Jak |
|---|---|
| zobaczyć, gdzie kończą się dane | `Ctrl` + `End` → powinno rzucić do `G577` |
| policzyć wiersze | zaznacz kolumnę `A`, spójrz na pasek stanu na dole |
| wrócić na górę | `Ctrl` + `Home` |

Nagłówki są już zamrożone (*Widok → Zablokuj okienka*), więc przy przewijaniu
w dół nie stracisz z oczu nazw kolumn.

### Sortowanie

Sortowanie odpowiada na pytania typu „co jest największe / najmniejsze".

Kliknij dowolną komórkę w danych, potem *Dane → Sortuj*. Posortuj po kolumnie
`sztuki` malejąco — na górze wylądują największe transakcje.

!!! warning "Najczęstszy błąd początkującego"

    Nigdy nie zaznaczaj **jednej kolumny** przed sortowaniem. Excel posortuje
    wtedy tylko ją, a reszta wierszy zostanie na miejscu — dane się rozjadą
    i każdy wiersz będzie opisywał coś innego niż przed chwilą. Zaznacz
    **jedną komórkę** wewnątrz danych i pozwól Excelowi wykryć zakres.

### Filtrowanie

Filtr odpowiada na pytanie „pokaż mi tylko te wiersze, które…".

Zaznacz komórkę w danych i wciśnij `Ctrl` + `Shift` + `L`. W nagłówkach pojawią
się strzałki. Spróbuj:

- pokazać tylko region **Południe**,
- pokazać tylko kategorię **Elektronika**,
- oba filtry naraz (działają jak „i", nie „lub").

Zwróć uwagę na numery wierszy z lewej — robią się niebieskie i „przeskakują".
To znaczy, że część wierszy jest **ukryta, a nie usunięta**.

!!! note "Sortowanie kontra filtrowanie"

    **Sortowanie** trwale zmienia kolejność wierszy w pliku.
    **Filtrowanie** tylko ukrywa wiersze — wyłączasz filtr i wszystko wraca.
    Dlatego filtr jest bezpieczniejszy do „rozglądania się" po danych.

### Kolumny obliczeniowe — przychód, koszt, zysk

W danych nie ma kolumny z przychodem. I bardzo dobrze — systemy sprzedażowe
zapisują fakty (ile sztuk, po jakiej cenie), a nie wyliczenia. Policzenie
wyliczeń to **Twoja** robota.

Zanim zaczniesz klikać, ustalmy pojęcia — to ekonomia, nie Excel:

| Pojęcie | Wzór | Co mówi |
|---|---|---|
| **przychód** | `sztuki × cena` | ile pieniędzy wpłynęło |
| **koszt całkowity** | `sztuki × koszt` | ile nas to kosztowało |
| **zysk** (marża kwotowa) | `przychód − koszt całkowity` | ile zostało w kieszeni |
| **marża %** | `zysk ÷ przychód` | ile groszy z każdej złotówki zostaje |

Przychód to **nie** jest zarobek. To rozróżnienie wraca dziś jeszcze dwa razy,
w tym w zadaniu.

W komórkach `H1`, `I1`, `J1` wpisz nagłówki `przychod`, `koszt_calkowity`, `zysk`,
a pod nimi formuły:

```text
H2:  =E2*F2
I2:  =E2*G2
J2:  =H2-I2
```

Teraz skopiuj je w dół do wiersza 577. Najszybciej: zaznacz `H2:J2`, złap mały
kwadracik w prawym dolnym rogu zaznaczenia i **kliknij go dwukrotnie** — Excel
sam wypełni w dół do końca danych.

!!! tip "Skąd Excel wie, że ma zmienić E2 na E3?"

    Bo `E2` to **adres względny** — przy kopiowaniu w dół przesuwa się razem
    z formułą. Gdybyś chciał, żeby adres się nie ruszał (np. wskazywał stałą
    stawkę VAT w jednej komórce), piszesz `$E$2`. Znak `$` znaczy „zablokuj".
    Skrót: zaznacz adres w formule i wciśnij `F4`.

### Formuły podsumowujące

Teraz policzmy podsumowania. Wejdź na wolne miejsce z boku, np. `L2` w dół,
i wpisuj kolejno:

```text
=SUMA(H2:H577)                                  3 835 357 zł  - przychód roczny
=SUMA(J2:J577)                                  1 051 409 zł  - zysk roczny
=SUMA(J2:J577)/SUMA(H2:H577)                    27,4%         - marża
=ŚREDNIA(E2:E577)                               5,79          - średnia sztuk
=MAX(H2:H577)                                   31 491 zł     - największa transakcja
=MIN(H2:H577)                                   89 zł         - najmniejsza
```

Czyli z każdej złotówki przychodu zostaje nam **27 groszy**. To pierwsza liczba,
która naprawdę coś mówi.

#### Formuły warunkowe

Tu zaczyna się prawdziwa analiza — liczymy nie wszystko, tylko to, co spełnia
warunek.

| Pytanie | Formuła (PL) | Wynik |
|---|---|---|
| Ile było transakcji w Elektronice? | `=LICZ.JEŻELI(C2:C577;"Elektronika")` | 192 |
| Jaki przychód dała Elektronika? | `=SUMA.JEŻELI(C2:C577;"Elektronika";H2:H577)` | 1 841 124 zł |
| A Elektronika na Południu? | `=SUMA.WARUNKÓW(H2:H577;B2:B577;"Południe";C2:C577;"Elektronika")` | 389 635 zł |

Zwróć uwagę na kolejność argumentów — to klasyczna pułapka:

- `SUMA.JEŻELI(gdzie_szukam; czego; co_sumuję)` — zakres do zsumowania **na końcu**,
- `SUMA.WARUNKÓW(co_sumuję; gdzie_szukam_1; czego_1; gdzie_szukam_2; czego_2)` —
  zakres do zsumowania **na początku**.

!!! note "Polskie i angielskie nazwy funkcji"

    Jeśli masz Excela po angielsku, nazwy są inne, a argumenty rozdzielasz
    przecinkiem zamiast średnika.

    | Polski | Angielski |
    |---|---|
    | `SUMA` | `SUM` |
    | `ŚREDNIA` | `AVERAGE` |
    | `MEDIANA` | `MEDIAN` |
    | `LICZ.JEŻELI` | `COUNTIF` |
    | `LICZ.WARUNKI` | `COUNTIFS` |
    | `SUMA.JEŻELI` | `SUMIF` |
    | `SUMA.WARUNKÓW` | `SUMIFS` |
    | `MIESIĄC` | `MONTH` |
    | `ZAOKR.GÓRA` | `ROUNDUP` |

### Kolumny pomocnicze — miesiąc i kwartał

Zarząd myśli w kwartałach, a w danych mamy pojedyncze daty. Dodajmy więc dwie
kolumny pomocnicze. W `K1` i `L1` wpisz nagłówki `miesiac` i `kwartal`, a pod nimi:

```text
K2:  =MIESIĄC(A2)
L2:  ="Q"&ZAOKR.GÓRA(MIESIĄC(A2)/3;1)
```

Po angielsku: `=MONTH(A2)` oraz `="Q"&ROUNDUP(MONTH(A2)/3,0)`.

Skopiuj w dół. Znak `&` skleja tekst — dzięki niemu zamiast `1` dostajesz `Q1`.

---

### Tabela przestawna

Formuły warunkowe są dobre, gdy masz jedno konkretne pytanie. Ale gdy chcesz
zobaczyć **wszystkie przekroje naraz**, pisanie kilkunastu `SUMA.WARUNKÓW` jest
bez sensu. Od tego jest tabela przestawna — narzędzie, które w kilka sekund
podsumowuje dane w dowolnym układzie.

Zaznacz komórkę w danych, potem *Wstawianie → Tabela przestawna → Nowy arkusz*.
Po prawej pojawi się panel z listą pól. Przeciągnij:

| Pole | Gdzie |
|---|---|
| `region` | **Wiersze** |
| `kwartal` | **Kolumny** |
| `przychod` | **Wartości** |

Jeśli w polu Wartości pojawi się *Liczba z przychod* zamiast *Suma z przychod* —
kliknij to pole, *Ustawienia pola wartości* → **Suma**. Excel wybiera sumowanie
tylko wtedy, gdy cała kolumna jest liczbowa.

Dostaniesz to (zaokrąglone do pełnych złotych):

| region | Q1 | Q2 | Q3 | Q4 | **Razem** |
|---|---|---|---|---|---|
| Południe | 260 185 | 231 971 | 241 359 | **59 990** | 793 505 |
| Północ | 257 314 | 245 920 | 245 255 | 280 120 | 1 028 609 |
| Wschód | 236 251 | 219 327 | 241 749 | 245 165 | 942 492 |
| Zachód | 255 587 | 269 072 | 251 809 | 294 283 | 1 070 751 |
| **Razem** | 1 009 337 | 966 290 | **980 172** | **879 558** | 3 835 357 |

!!! tip "Formatowanie, żeby dało się to czytać"

    Zaznacz obszar wartości i nadaj format liczbowy z separatorem tysięcy i bez
    miejsc po przecinku (`Ctrl` + `1` → *Liczbowe*). Siedmiocyfrowe liczby bez
    spacji są nieczytelne, a nieczytelna tabela to tabela, z której nikt nie
    wyciągnie wniosku.

### Case: czy sprzedaż naprawdę spadła?

Wróćmy do pytania zarządu.

**Krok 1 — czy spadek w ogóle jest?** Patrzymy na wiersz Razem: Q3 to 980 172 zł,
Q4 to 879 558 zł. Czyli spadek o **10,3%**. Tak, zarząd ma rację.

**Krok 2 — kogo to dotyczy?** I tu robi się ciekawie. Porównaj Q4 z Q3 w każdym
regionie:

| region | Q3 | Q4 | zmiana |
|---|---|---|---|
| Południe | 241 359 | 59 990 | **−75,1%** |
| Północ | 245 255 | 280 120 | +14,2% |
| Wschód | 241 749 | 245 165 | +1,4% |
| Zachód | 251 809 | 294 283 | +16,9% |

Trzy regiony na cztery **urosły** — co ma sens, bo Q4 to okres przedświąteczny.
Cały spadek pochodzi z jednego regionu, w którym sprzedaż praktycznie się
zawaliła.

**Krok 3 — kiedy to się stało?** Dodaj do tabeli przestawnej `miesiac` zamiast
`kwartal` w kolumnach. Zobaczysz, że Południe przez dziewięć miesięcy kręciło się
wokół 80 tys. zł miesięcznie, a od października spadło do 25, 20 i 15 tys. zł.
To nie jest powolne osuwanie się — to **ścięcie z miesiąca na miesiąc**.

!!! success "Wniosek, który idzie do zarządu"

    Sprzedaż spadła o 10% kwartał do kwartału, ale to nie jest problem rynkowy —
    trzy z czterech regionów urosły. Cały spadek to region Południe, gdzie
    sprzedaż runęła o 75% i stało się to nagle w październiku. Szukamy przyczyny
    lokalnej: zamknięty sklep, odejście zespołu handlowego, nowa konkurencja,
    awaria systemu sprzedażowego.

    Zwróć uwagę, że dane **nie mówią, co się stało**. Mówią, gdzie i kiedy szukać.
    To już ogromna różnica wobec „sprzedaż spadła, zróbmy promocję".

### Wykres

Liczby przekonują, ale wykres przekonuje szybciej. Na tabeli przestawnej
z układem `region` w wierszach i `miesiac` w kolumnach: *Wstawianie → Wykres
liniowy*.

Zobaczysz cztery linie — trzy falują sobie mniej więcej w przedziale
55–105 tys. zł przez cały rok, a czwarta (Południe) trzyma ten sam poziom do września i **spada
pionowo w dół** w październiku. Tego jednego obrazka nie trzeba tłumaczyć nikomu.

!!! warning "Wykres ma odpowiadać na pytanie"

    Wykres kołowy z dwunastoma kawałkami, wykres słupkowy z pięćdziesięcioma
    słupkami albo trójwymiarowy cokolwiek — to nie są wizualizacje, to dekoracje.
    Zanim wstawisz wykres, odpowiedz sobie: **jakie zdanie ma z niego wynikać?**
    Tutaj zdanie brzmi „Południe odpadło w październiku" — i dlatego rysujemy
    zmianę w czasie, czyli wykres liniowy.

### Gdzie Excel zaczyna boleć

Excel jest świetny i nie zamierzam Was do niego zniechęcać. Ale zanim przejdziemy
do Pythona, zauważ kilka rzeczy z ostatniej godziny:

- klikanie w tabelę przestawną **nie zostawia śladu** — za tydzień nie odtworzysz,
  co dokładnie zrobiłeś,
- gdybyś dostał nowy plik za 2026 rok, całą tę pracę robisz **od zera**,
- nikt nie jest w stanie sprawdzić Twojej analizy inaczej niż klikając to samo,
- przy 500 tys. wierszy zamiast 576 Excel zaczyna się dławić (limit arkusza to
  ok. 1 048 576 wierszy).

Python rozwiązuje wszystkie cztery, bo analiza jest **zapisana jako tekst**.
Uruchamiasz ją ponownie jednym kliknięciem, na dowolnym pliku, a każdy może
przeczytać, co dokładnie policzyłeś.

---

## Przerwa — 10 minut

---

## Część 2: to samo w Pythonie

Teraz przechodzimy na drugą stronę. **Nie uczymy się programowania** — nie będzie
pętli, funkcji ani klas. Będziemy robić te same rzeczy co przed chwilą, tylko
zapisywać je zdaniami zamiast klikać.

### Google Colab

Wejdź na [colab.research.google.com](https://colab.research.google.com) i załóż
nowy notatnik (*Plik → Nowy notatnik*). Nie trzeba nic instalować — wszystko
działa w przeglądarce, a pandas jest już na miejscu.

Notatnik składa się z **komórek**:

- **komórki z kodem** — uruchamiasz je i pod spodem pojawia się wynik,
- **komórki tekstowe (Markdown)** — opis, wnioski, nagłówki.

To ważna różnica wobec zwykłego programowania: nie piszesz całego programu
i nie uruchamiasz go raz. Pracujesz **małymi krokami**, oglądając wynik każdego
z nich — dokładnie tak, jak przed chwilą w Excelu.

| Skrót | Działanie |
|---|---|
| `Shift` + `Enter` | uruchom komórkę i przejdź do następnej |
| `Ctrl` + `Enter` | uruchom komórkę i zostań w niej |
| `Esc`, potem `A` / `B` | dodaj komórkę nad / pod |
| `Esc`, potem `M` | zamień komórkę na tekstową |

!!! danger "Pułapka, która zje Wam pół zajęć, jeśli jej nie zrozumiecie"

    Notatnik pamięta stan **w kolejności, w jakiej uruchamiałeś komórki**, a nie
    w kolejności, w jakiej leżą na ekranie. Jeśli zmienisz komórkę wyżej i jej
    **nie uruchomisz**, komórki niżej nadal widzą starą wartość.

    Gdy coś zachowuje się nielogicznie: *Środowisko wykonawcze → Uruchom ponownie
    i wykonaj wszystko*. To rozwiązuje zdecydowaną większość takich problemów.

### Wczytanie tego samego pliku

W Excelu plik otwierałeś dwuklikiem. W Pythonie robi to jedna linijka — i co
ważne, możesz wczytać plik **prosto z internetu**, bez pobierania:

```python
import pandas as pd

URL = "https://kubzal.github.io/wsb-merito/semestr_zimowy_2026_27/data_science/dane/sprzedaz_2025.csv"
df = pd.read_csv(URL)

df.head()
```

```text
         data    region    kategoria             produkt  sztuki    cena  koszt
0  2025-01-01    Wschód  Elektronika       Tablet Lenovo       6   899.0  690.0
1  2025-01-01    Północ          AGD  Mikrofalówka Amica       5   529.0  385.0
2  2025-01-02  Południe        Meble       Fotel biurowy       4  1199.0  760.0
3  2025-01-02    Zachód          AGD     Ekspres do kawy       6   899.0  640.0
4  2025-01-02    Zachód  Elektronika     Smartfon Xiaomi       6  1299.0  980.0
```

Trzy rzeczy, które właśnie się wydarzyły:

- `import pandas as pd` — wczytujemy bibliotekę **pandas** i nadajemy jej krótką
  nazwę `pd`. Pandas to „Excel w Pythonie". Ta linijka jest na początku
  praktycznie każdej analizy na świecie.
- `pd.read_csv(...)` — wczytuje plik do **DataFrame'u**, czyli tabeli. `df` to
  standardowa nazwa, zobaczysz ją w każdym poradniku.
- Liczby z lewej (0, 1, 2, …) to **indeks** — numery wierszy. Python liczy od zera,
  więc pierwszy wiersz ma numer 0, a nie 1.

!!! tip "Bez print()"

    W notatniku ostatnia linijka komórki wyświetla się sama, i to ładnie
    sformatowana jako tabela. `df.head()` da czytelniejszy wynik niż
    `print(df.head())`.

### Pierwsze spojrzenie

Pamiętasz `Ctrl` + `End` w Excelu? To samo, tylko konkretniej:

```python
df.shape
```

```text
(576, 7)
```

576 wierszy, 7 kolumn. Teraz typy danych:

```python
df.info()
```

```text
<class 'pandas.DataFrame'>
RangeIndex: 576 entries, 0 to 575
Data columns (total 7 columns):
 #   Column     Non-Null Count  Dtype
---  ------     --------------  -----
 0   data       576 non-null    str
 1   region     576 non-null    str
 2   kategoria  576 non-null    str
 3   produkt    576 non-null    str
 4   sztuki     576 non-null    int64
 5   cena       576 non-null    float64
 6   koszt      576 non-null    float64
dtypes: float64(2), int64(1), str(4)
memory usage: 31.6 KB
```

Dwie rzeczy warte zauważenia:

- Kolumna `Non-Null Count` mówi, ile jest **niepustych** wartości. Wszędzie 576,
  czyli komplet — nie ma braków danych. To pierwsze miejsce, gdzie się je wykrywa.
- Kolumna `data` ma typ `str`, czyli **tekst**. Python nie wie, że to data. Zaraz
  to naprawimy.

```python
df.describe()
```

```text
           sztuki         cena        koszt
count  576.000000   576.000000   576.000000
mean     5.786458  1138.322917   819.086806
std      1.570785   882.498071   685.069537
min      1.000000    89.000000    41.000000
25%      5.000000   399.000000   230.000000
50%      6.000000   899.000000   690.000000
75%      7.000000  1599.000000  1020.000000
max      9.000000  3499.000000  2750.000000
```

Jedna komenda zamiast sześciu formuł z Excela. `describe()` pomija kolumny
tekstowe i dla pozostałych podaje liczebność, średnią, odchylenie standardowe,
minimum, maksimum i kwartyle. Na razie patrz na `mean`, `min` i `max` — resztą
zajmiemy się na zajęciach 2.

Kolumna `data` się tu nie pojawiła, bo jest tekstem. Gdy za chwilę wczytamy ją
jako prawdziwą datę, `describe()` pokaże także ją — z najwcześniejszą i najpóźniejszą
datą zamiast minimum i maksimum.

!!! note "Jeśli u Ciebie zamiast `str` jest `object`"

    To ta sama rzecz, tylko starsza nazwa (pandas 2.x). Nic nie musisz zmieniać.

Odpowiednik `LICZ.JEŻELI` dla całej kolumny naraz:

```python
df["region"].value_counts()
```

```text
region
Wschód      144
Północ      144
Południe    144
Zachód      144
Name: count, dtype: int64
```

### Kolumna obliczeniowa

W Excelu: wpisz `=E2*F2` i przeciągnij w dół 576 razy. W Pythonie:

```python
df["przychod"] = df["sztuki"] * df["cena"]

df[["data", "region", "produkt", "sztuki", "cena", "przychod"]].head()
```

```text
         data    region             produkt  sztuki    cena  przychod
0  2025-01-01    Wschód       Tablet Lenovo       6   899.0    5394.0
1  2025-01-01    Północ  Mikrofalówka Amica       5   529.0    2645.0
2  2025-01-02  Południe       Fotel biurowy       4  1199.0    4796.0
3  2025-01-02    Zachód     Ekspres do kawy       6   899.0    5394.0
4  2025-01-02    Zachód     Smartfon Xiaomi       6  1299.0    7794.0
```

Nie ma przeciągania i nie ma pętli. Mnożenie **całej kolumny przez całą kolumnę**
wykonuje się wiersz po wierszu automatycznie. To się nazywa *wektoryzacja* i jest
jednym z głównych powodów, dla których pandas jest szybki.

Dorzućmy pozostałe:

```python
df["koszt_calkowity"] = df["sztuki"] * df["koszt"]
df["zysk"] = df["przychod"] - df["koszt_calkowity"]
```

!!! warning "Jeden nawias czy dwa?"

    `df["cena"]` → **jedna kolumna**.
    `df[["produkt", "cena"]]` → **kilka kolumn** — w środku jest lista nazw,
    stąd podwójny nawias. To najczęstsza pomyłka pierwszego dnia.

### Podsumowania

```python
df["przychod"].sum()        # 3835357.0
df["zysk"].sum()            # 1051409.0
round(df["sztuki"].mean(), 2)   # 5.79
df["przychod"].max()        # 31491.0
```

Te same liczby co w Excelu — bo to te same dane i ta sama arytmetyka.

### Filtrowanie

W Excelu klikałeś strzałkę w nagłówku. W Pythonie piszesz warunek:

```python
df[df["kategoria"] == "Elektronika"].shape
```

```text
(192, 10)
```

192 wiersze — dokładnie tyle, ile pokazało `LICZ.JEŻELI`.

Jak to działa? Porównanie kolumny z wartością daje kolumnę wartości
`True`/`False` (nazywamy ją **maską**), a wstawienie maski w nawias kwadratowy
zostawia tylko wiersze z `True`.

Dwa warunki naraz — odpowiednik dwóch filtrów w Excelu:

```python
maska = (df["region"] == "Południe") & (df["kategoria"] == "Elektronika")

df[maska][["data", "produkt", "sztuki", "przychod"]].head()
```

```text
          data          produkt  sztuki  przychod
27  2025-01-14    Tablet Lenovo       6    5394.0
38  2025-01-21      Laptop Dell       7   24493.0
40  2025-01-22  Smartfon Xiaomi       6    7794.0
48  2025-02-01    Tablet Lenovo       8    7192.0
54  2025-02-02  Smartfon Xiaomi       7    9093.0
```

Zwróć uwagę na indeks — zostały oryginalne numery wierszy (27, 38, 40…), więc
od razu widać, że to wycinek. A odpowiednik `SUMA.WARUNKÓW`:

```python
df[maska]["przychod"].sum()     # 389635.0
```

Ta sama liczba co w Excelu.

!!! warning "`&` zamiast `and`"

    W pandas piszemy `&` (i) oraz `|` (lub), nie `and`/`or`. Nawiasy wokół
    każdego warunku są **obowiązkowe** — bez nich Python policzy to w złej
    kolejności i dostaniesz błąd.

### Sortowanie

```python
df.sort_values("przychod", ascending=False)[["data", "region", "produkt", "sztuki", "przychod"]].head(5)
```

```text
           data    region      produkt  sztuki  przychod
559  2025-12-20    Zachód  Laptop Dell       9   31491.0
494  2025-11-09    Zachód  Laptop Dell       9   31491.0
529  2025-12-01    Północ  Laptop Dell       9   31491.0
380  2025-08-27  Południe  Laptop Dell       8   27992.0
444  2025-10-09    Północ  Laptop Dell       8   27992.0
```

`ascending=False` to sortowanie malejąco. Pięć największych transakcji roku to
same laptopy — najdroższy produkt w ofercie.

### groupby — serce analizy danych

`groupby` odpowiada dokładnie na to samo pytanie co tabela przestawna: **podziel
dane na grupy i policz coś w każdej**.

```python
df.groupby("region")["przychod"].sum().sort_values(ascending=False)
```

```text
region
Zachód      1070751.0
Północ      1028609.0
Wschód       942492.0
Południe     793505.0
Name: przychod, dtype: float64
```

Czytaj to jak zdanie: *pogrupuj po regionie, weź kolumnę przychód, zsumuj,
posortuj malejąco*. Jedna linijka zamiast całego panelu z przeciąganiem pól.

Można policzyć kilka statystyk naraz:

```python
df.groupby("kategoria")["przychod"].agg(["sum", "mean", "count"]).round(2)
```

```text
                   sum     mean  count
kategoria
AGD           968211.0  4939.85    196
Elektronika  1841124.0  9589.19    192
Meble        1026022.0  5457.56    188
```

I tu od razu ciekawostka biznesowa — policzmy marżę w każdej kategorii:

```python
g = df.groupby("kategoria")[["przychod", "zysk"]].sum()
g["marza_%"] = (g["zysk"] / g["przychod"] * 100).round(1)
g
```

```text
              przychod      zysk  marza_%
kategoria
AGD           968211.0  253925.0     26.2
Elektronika  1841124.0  435199.0     23.6
Meble        1026022.0  362285.0     35.3
```

**Elektronika daje prawie dwa razy większy przychód niż Meble, ale ma najgorszą
marżę.** Gdyby patrzeć tylko na przychód, wnioski byłyby odwrotne. To jest ta
różnica między „ile wpłynęło" a „ile zarobiliśmy", o której mówiliśmy na początku.

### Daty i tabela przestawna

Pamiętasz, że kolumna `data` była tekstem? Żeby wyciągnąć z niej miesiąc, trzeba
powiedzieć Pythonowi, że to data:

```python
df = pd.read_csv(URL, parse_dates=["data"])
df["przychod"] = df["sztuki"] * df["cena"]

df["kwartal"] = "Q" + df["data"].dt.quarter.astype(str)
df["miesiac"] = df["data"].dt.month
```

`parse_dates=["data"]` mówi: potraktuj tę kolumnę jako datę. Potem `.dt` daje
dostęp do części daty — `.dt.month`, `.dt.quarter`, `.dt.year`, `.dt.day_name()`.

A teraz tabela przestawna. Nazywa się tak samo jak w Excelu:

```python
df.pivot_table(index="region", columns="kwartal", values="przychod", aggfunc="sum")
```

```text
kwartal         Q1        Q2        Q3        Q4
region
Południe  260185.0  231971.0  241359.0   59990.0
Północ    257314.0  245920.0  245255.0  280120.0
Wschód    236251.0  219327.0  241749.0  245165.0
Zachód    255587.0  269072.0  251809.0  294283.0
```

Porównaj to z tabelą, którą klikałeś pół godziny temu. **Identyczne liczby.**
Cztery argumenty odpowiadają czterem polom z panelu Excela:

| Argument | Pole w Excelu |
|---|---|
| `index=` | Wiersze |
| `columns=` | Kolumny |
| `values=` | Wartości |
| `aggfunc=` | Suma / Średnia / Licznik |

### Wykres

```python
df.pivot_table(index="miesiac", columns="region", values="przychod", aggfunc="sum").plot(
    figsize=(10, 5),
    title="Przychód wg regionu i miesiąca (2025)",
)
```

Jedna linijka i masz ten sam wykres, co po kilku kliknięciach w Excelu: cztery
linie, z których jedna spada pionowo w październiku.

Zwróć uwagę, że w `index` dałem tym razem `miesiac`, a w `columns` — `region`.
Zamiana miejscami decyduje o tym, co jest na osi poziomej, a co jest linią.

!!! success "Tu jest cała pointa dzisiejszych zajęć"

    Ta analiza — od wczytania pliku po wykres — to jakieś **15 linijek kodu**.
    Możesz je jutro uruchomić na danych za 2026 rok, zmieniając jeden adres.
    Możesz wysłać je koledze, który zobaczy dokładnie, co policzyłeś. Możesz
    wrócić do nich za rok i zrozumieć własną pracę.

    Żadnej z tych trzech rzeczy nie da się zrobić z tabelą przestawną, którą się
    wyklikało.

---

## Excel kontra Python — ściągawka

Trzymaj to pod ręką, robiąc zadanie. Zakresy typu `H2:H577` dotyczą naszego pliku.

| Chcę… | Excel | Python (pandas) |
|---|---|---|
| otworzyć plik | dwuklik | `df = pd.read_csv(URL)` |
| zobaczyć początek | patrzę na ekran | `df.head()` |
| ile wierszy i kolumn | `Ctrl` + `End` | `df.shape` |
| typy kolumn i braki | oglądam | `df.info()` |
| podstawowe statystyki | `SUMA`, `ŚREDNIA`, `MIN`, `MAX` | `df.describe()` |
| nowa kolumna | `=E2*F2` + przeciągnięcie | `df["p"] = df["a"] * df["b"]` |
| suma kolumny | `=SUMA(H2:H577)` | `df["p"].sum()` |
| średnia | `=ŚREDNIA(E2:E577)` | `df["sztuki"].mean()` |
| mediana | `=MEDIANA(E2:E577)` | `df["sztuki"].median()` |
| zliczanie z warunkiem | `=LICZ.JEŻELI(C:C;"AGD")` | `(df["kategoria"] == "AGD").sum()` |
| suma z warunkiem | `=SUMA.JEŻELI(C:C;"AGD";H:H)` | `df[df["kategoria"] == "AGD"]["p"].sum()` |
| filtr | `Ctrl` + `Shift` + `L` | `df[df["cena"] > 1000]` |
| filtr z dwoma warunkami | dwa filtry naraz | `df[(df["a"] > 1) & (df["b"] == "x")]` |
| sortowanie | *Dane → Sortuj* | `df.sort_values("cena", ascending=False)` |
| ile razy występuje wartość | tabela przestawna | `df["region"].value_counts()` |
| podsumowanie wg grup | tabela przestawna | `df.groupby("region")["p"].sum()` |
| tabela przestawna | przeciąganie pól | `df.pivot_table(index=…, columns=…, values=…, aggfunc="sum")` |
| wykres | *Wstawianie → Wykres* | `.plot()` |

### To którego w końcu używać?

Obu. Naprawdę.

| Sytuacja | Narzędzie |
|---|---|
| szybkie zerknięcie w dane, 200 wierszy | **Excel** |
| pokazanie czegoś osobie nietechnicznej | **Excel** |
| ten sam raport co miesiąc | **Python** |
| ponad 100 tys. wierszy | **Python** |
| analiza, którą ktoś musi zweryfikować | **Python** |
| łączenie danych z kilku źródeł | **Python** |
| prognozowanie, modele | **Python** |

Analityk, który zna tylko Excela, w pewnym momencie uderza w sufit. Analityk,
który zna tylko Pythona, marnuje pół godziny na coś, co w arkuszu zająłby minutę.

---

## Zadanie 1 — Sieć kawiarni (12 pkt)

**Czas:** ok. 60 minut (30 min Excel + 30 min Python)
**Oddanie:** plik `.xlsx` (część 1) oraz notatnik `.ipynb` (część 2) na Moodle

Prowadzisz analizę dla sieci pięciu kawiarni. Właściciel jest zadowolony, bo
lokal w Gdańsku sprzedaje najwięcej ze wszystkich. Twoim zadaniem jest sprawdzić,
czy „sprzedaje najwięcej" znaczy to samo co „zarabia najwięcej".

Dane obejmują pierwsze **półrocze 2026 roku**, 180 wierszy:

- :material-microsoft-excel: [**kawiarnie_2026.xlsx**](dane/kawiarnie_2026.xlsx) — do części 1
- :material-file-delimited: [**kawiarnie_2026.csv**](dane/kawiarnie_2026.csv) — do części 2

| Kolumna | Co oznacza |
|---|---|
| `data` | data sprzedaży (2026-01-01 … 2026-06-27) |
| `miasto` | Warszawa, Kraków, Gdańsk, Wrocław |
| `lokal` | Nowy Świat, Mokotów, Rynek Główny, Długi Targ, Świdnicka |
| `kategoria` | Kawa, Herbata, Ciasta, Kanapki |
| `produkt` | nazwa pozycji z menu |
| `sztuki` | ile sztuk sprzedano |
| `cena` | cena sprzedaży jednej sztuki (zł) |
| `koszt` | koszt przygotowania jednej sztuki (zł) |

Rób części po kolei i **pokaż mi wynik każdej z nich, gdy będzie gotowa** —
punktacja zależy od tego, ile zdążysz na zajęciach.

### Część 1 — Excel (ok. 30 min)

Otwórz `kawiarnie_2026.xlsx`. Dane zajmują wiersze **2–181**, kolumny **A–H**.

1. Dodaj trzy kolumny obliczeniowe (nagłówki w `I1`, `J1`, `K1`) i skopiuj
   formuły w dół do wiersza 181:

    - `przychod` = `sztuki × cena`
    - `koszt_calkowity` = `sztuki × koszt`
    - `zysk` = `przychod − koszt_calkowity`

2. Obok danych policz dla całej sieci: **łączny przychód**, **łączny zysk**
   i **marżę procentową** (zysk ÷ przychód). Marżę sformatuj jako procent.

3. Używając formuł warunkowych, policz:

    - ile było transakcji w kategorii **Ciasta** (`LICZ.JEŻELI`),
    - jaki przychód dała kategoria **Kawa** (`SUMA.JEŻELI`),
    - jaki przychód dały **Ciasta w lokalu Długi Targ** (`SUMA.WARUNKÓW`).

4. Zbuduj **tabelę przestawną**: `lokal` w wierszach, a w wartościach **suma
   przychodu** oraz **suma zysku**. Posortuj ją malejąco po przychodzie.

5. Dodaj do tej tabeli kolumnę z **marżą procentową** każdego lokalu
   (zysk ÷ przychód — policz ją formułą obok tabeli).

6. Zbuduj **drugą tabelę przestawną**: `kategoria` w wierszach, przychód i zysk
   w wartościach. Policz marżę też tutaj.

7. Wstaw **wykres słupkowy** porównujący przychód i zysk w poszczególnych
   lokalach.

8. W pustej komórce arkusza napisz w 2–3 zdaniach odpowiedź na pytanie
   właściciela: **czy lokal z największym przychodem jest lokalem, który
   najwięcej zarabia?** Jeśli nie — wyjaśnij, co to powoduje. Wskazówka: spójrz
   na tabelę z kategoriami i zastanów się, czym różni się to, co sprzedaje
   każdy lokal.

### Część 2 — Python (ok. 30 min)

Nowy notatnik w Colabie. Zaczynamy od wczytania tych samych danych:

```python
import pandas as pd

URL = "https://kubzal.github.io/wsb-merito/semestr_zimowy_2026_27/data_science/dane/kawiarnie_2026.csv"
df = pd.read_csv(URL, parse_dates=["data"])

df.head()
```

Następnie wykonaj poniższe punkty, **każdy w osobnej komórce**:

1. Sprawdź rozmiar zbioru (`shape`) oraz typy kolumn i braki danych (`info`).
2. Wyświetl podstawowe statystyki kolumn liczbowych (`describe`).
3. Dodaj kolumny `przychod`, `koszt_calkowity` i `zysk` — tak jak w części 1.
4. Policz łączny przychód i łączny zysk całej sieci. Sprawdź, czy zgadzają się
   z wynikiem z Excela (powinny!).
5. Wyświetl **przychód i zysk w podziale na lokale**, posortowane malejąco po
   przychodzie. Użyj `groupby`.
6. Dodaj do tego wyniku kolumnę `marza_%` i zaokrąglij ją do jednego miejsca
   po przecinku.
7. Zrób to samo w podziale na **kategorie** — przychód, zysk i marża.
8. Wyświetl **5 transakcji o największym zysku** (tylko kolumny `data`, `lokal`,
   `produkt`, `sztuki`, `zysk`).
9. Wyświetl transakcje z kategorii **Kawa** w lokalu **Mokotów**, w których
   sprzedano więcej niż 100 sztuk (filtr z trzema warunkami).
10. Zbuduj tabelę przestawną: `lokal` w wierszach, `miesiac` w kolumnach,
    suma przychodu w wartościach. Kolumnę `miesiac` musisz najpierw dodać
    (`df["data"].dt.month`).
11. Narysuj wykres z tej tabeli przestawnej (`.plot()`).
12. Na koniec dodaj **komórkę tekstową (Markdown)** i napisz w 3–4 zdaniach:
    który lokal ma największy przychód, który ma największy zysk, dlaczego to
    nie jest ten sam lokal i co doradziłbyś właścicielowi.

### Wskazówki

- Ściągawka Excel ↔ Python jest kilka sekcji wyżej. Korzystaj z niej bez skrupułów
  — nikt nie pamięta składni na pamięć, ja też nie.
- Punkt 6 części 2: policz to tak, jak liczyliśmy marżę kategorii na zajęciach —
  najpierw `groupby` do zmiennej, potem dodanie kolumny.
- Punkt 9: trzy warunki łączysz tak samo jak dwa, każdy w swoim nawiasie:
  `(A) & (B) & (C)`.
- Jeśli w Excelu tabela przestawna pokazuje *Liczba z przychod* zamiast sumy —
  kliknij pole wartości i zmień na **Suma**.
- Jeśli w Colabie coś zachowuje się nielogicznie — *Środowisko wykonawcze →
  Uruchom ponownie i wykonaj wszystko*.
- Nazywaj rzeczy sensownie. `df` dla tabeli jest w porządku, `x1`, `x2`, `x3`
  już nie.

!!! tip "Na co patrzę przy ocenie"

    Nie na to, czy kod jest elegancki. Na to, czy **liczby się zgadzają** i czy
    wniosek na końcu wynika z tego, co policzyłeś. Punkt 12 to nie jest
    formalność — to właściwie całe zadanie. Reszta to tylko droga do niego.

### Punktacja (12 pkt)

| Punkty | Za co |
|---|---|
| **12 pkt** | obie części zrobione **na zajęciach** i pokazane prowadzącemu |
| **8 pkt** | jedna z dwóch części zrobiona **na zajęciach** i pokazana prowadzącemu |
| **4 pkt** | zadanie dokończone po zajęciach i wysłane na Moodle **w terminie** |
| **0 pkt** | brak rozwiązania albo wysłane po terminie |

Termin przesłania na Moodle: **2026-10-24** (dzień przed kolejnymi zajęciami).

!!! danger "Serio, róbcie to na zajęciach"

    Ta sama godzina pracy jest warta 12 pkt tu i teraz albo 4 pkt w domu. Mamy
    tylko cztery zadania w całym semestrze, więc każde z nich to 12% oceny
    końcowej. Jestem na sali po to, żeby Wam pomagać — korzystajcie z tego,
    zamiast walczyć z tym samym problemem wieczorem w domu.
