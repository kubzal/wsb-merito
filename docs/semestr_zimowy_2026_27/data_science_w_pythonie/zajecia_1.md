# Zajęcia 1

## Czym jest Data Science i pierwsze kroki w pandas

Zaczynamy od zera: czym właściwie zajmuje się Data Science, jak wygląda praca nad
projektem analitycznym i w czym będziemy pracować. Potem szybkie przypomnienie
Pythona i dwie biblioteki, które będą Ci towarzyszyć przez cały semestr —
**numpy** i **pandas**.

Po dzisiejszych zajęciach będziesz w stanie:

- opowiedzieć, z jakich etapów składa się projekt Data Science,
- poruszać się po notatniku Jupyter / Google Colab i unikać jego typowej pułapki,
- policzyć statystyki na tablicy `numpy` bez pisania pętli,
- wczytać plik CSV do `DataFrame` i obejrzeć, co w nim jest,
- wybrać kolumny, przefiltrować wiersze i posortować dane w pandas.

---

## Plan na dziś

| Blok | Czas |
|---|---|
| Czym jest Data Science i workflow projektu | 10 min |
| Jupyter Notebook — jak w tym pracować | 5 min |
| Python w pigułce — tylko to, co dziś potrzebne | 5 min |
| numpy | 10 min |
| pandas | 15 min |
| **Zadanie 1** | **45 min** |

---

## Czym jest Data Science

Data Science to wyciąganie z danych odpowiedzi na pytania, na które nie da się
odpowiedzieć „na oko". Nie chodzi o to, żeby mieć dużo danych — chodzi o to, żeby
z nich coś **wynikało**.

Typowe pytania, na które odpowiada analityk danych:

- Którzy klienci najprawdopodobniej zrezygnują z abonamentu w przyszłym miesiącu?
- Czy spadek sprzedaży w marcu to sezonowość, czy realny problem?
- Ile zamówień obsłużymy w Black Friday i ilu kurierów trzeba zamówić?

Zwróć uwagę, że w każdym z nich najpierw jest **pytanie biznesowe**, a dopiero
potem dane. Odwrotna kolejność — „mamy dane, zobaczmy co z nich wyjdzie" — kończy
się zwykle ładnym wykresem, z którego nic nie wynika.

### Workflow projektu

Prawie każdy projekt analityczny przechodzi przez te same etapy:

1. **Pytanie** — co konkretnie chcemy wiedzieć i po co.
2. **Zebranie danych** — pliki, baza, API, scraping.
3. **Czyszczenie** — braki, duplikaty, literówki, złe typy. Zwykle najdłuższy etap.
4. **Eksploracja (EDA)** — oglądamy dane, liczymy statystyki, rysujemy wykresy.
5. **Modelowanie** — jeśli pytanie tego wymaga (o tym na Uczeniu głębokim).
6. **Komunikacja** — raport, dashboard, prezentacja. Analiza, której nikt nie
   zrozumiał, nie istnieje.

!!! info "Gdzie jesteśmy"

    Dziś dotykamy punktów 2 i 4. Punkt 3 — czyszczenie danych — to temat kolejnych
    zajęć, a punkt 5 robimy zaraz po tych zajęciach, na Uczeniu głębokim.

---

## Jupyter Notebook

Analizę danych pisze się inaczej niż zwykły program. Nie piszemy skryptu od góry do
dołu i nie uruchamiamy go raz — pracujemy **krokami**, oglądając wynik każdego
kroku. Do tego służy notatnik.

Notatnik składa się z **komórek**:

- **komórki z kodem** — uruchamiasz je i pod spodem pojawia się wynik,
- **komórki tekstowe (Markdown)** — opis, wnioski, nagłówki.

Najprościej zacząć od **Google Colab** ([colab.research.google.com](https://colab.research.google.com)) —
nie trzeba nic instalować, wszystko działa w przeglądarce i ma już zainstalowane
numpy i pandas. Na tych zajęciach pracujemy właśnie w Colabie.

Przydatne skróty:

| Skrót | Działanie |
|---|---|
| `Shift` + `Enter` | uruchom komórkę i przejdź do następnej |
| `Ctrl` + `Enter` | uruchom komórkę i zostań w niej |
| `Esc`, potem `A` / `B` | dodaj komórkę nad / pod |
| `Esc`, potem `M` | zamień komórkę na tekstową |
| `Esc`, potem `D` `D` | usuń komórkę |

### Pułapka: kolejność wykonania

To jest najczęstsze źródło frustracji na pierwszych zajęciach, więc przeczytaj
uważnie.

Notatnik pamięta stan **w kolejności, w jakiej uruchamiałeś komórki**, a nie
w kolejności, w jakiej są ułożone na ekranie. Jeśli uruchomisz komórkę,
potem ją zmienisz i uruchomisz coś niżej — zmienna może mieć wartość, której się
nie spodziewasz.

```python
# komórka 1
x = 10

# komórka 2
print(x * 2)      # 20

# wracasz do komórki 1, zmieniasz na x = 99, ale JEJ NIE URUCHAMIASZ
# komórka 2 nadal wypisze 20, bo w pamięci x to wciąż 10
```

Numer w nawiasie kwadratowym obok komórki (`[1]`, `[2]`, …) mówi, w jakiej
kolejności komórki faktycznie się wykonały. Jeśli coś zachowuje się dziwnie:
**Runtime → Restart and run all**. To rozwiązuje 90% takich problemów.

---

## Python w pigułce

Tylko te elementy, których dziś użyjemy. Jeśli coś jest nowe — nie panikuj,
wrócimy do tego w praktyce.

```python
# zmienne
nazwa = "Laptop"          # tekst (str)
cena = 3499.0             # liczba zmiennoprzecinkowa (float)
sztuki = 4                # liczba całkowita (int)

# f-string — wstawianie wartości do tekstu
print(f"{nazwa} kosztuje {cena:.2f} zł, mamy {sztuki} szt.")
# Laptop kosztuje 3499.00 zł, mamy 4 szt.

# lista — uporządkowana kolekcja
ceny = [3499.0, 1299.0, 299.0]
print(len(ceny), ceny[0], ceny[-1])     # 3 3499.0 299.0

# słownik — pary klucz: wartość
produkt = {"nazwa": "Laptop", "cena": 3499.0}
print(produkt["cena"])                   # 3499.0

# pętla
for c in ceny:
    print(c * 1.23)                      # cena brutto

# funkcja
def brutto(netto, vat=0.23):
    return netto * (1 + vat)

print(brutto(100))                       # 123.0
```

Zapamiętaj `f"..."` i słowniki — będą wracać przez cały semestr.

---

## numpy

`numpy` to biblioteka do liczenia na tablicach liczb. Jej podstawowy typ to
**ndarray** — tablica, która wygląda jak lista, ale zachowuje się zupełnie inaczej.

```python
import numpy as np

oceny = np.array([3.0, 4.5, 5.0, 3.5, 4.0, 2.0, 5.0, 4.5])
print(oceny)
```

```text
[3.  4.5 5.  3.5 4.  2.  5.  4.5]
```

### Wektoryzacja — liczenie bez pętli

Na zwykłej liście, żeby podwoić każdy element, musisz napisać pętlę. Na tablicy
numpy piszesz to, co masz na myśli:

```python
print(oceny * 2)
```

```text
[ 6.  9. 10.  7.  8.  4. 10.  9.]
```

Operacja wykonała się **na każdym elemencie naraz**. To nie jest tylko krótszy
zapis — to też dużo szybsze, bo pętla dzieje się w skompilowanym kodzie C, a nie
w Pythonie. Przy milionie wierszy różnica jest odczuwalna.

### Statystyki

```python
print(oceny.mean())    # średnia
print(oceny.max())     # największa
print(oceny.min())     # najmniejsza
print(oceny.std())     # odchylenie standardowe
print(oceny.sum())     # suma
print(len(oceny))      # ile elementów
```

```text
3.9375
5.0
2.0
0.982264602843857
31.5
8
```

Do ładnego wyświetlania przyda się `round()`:

```python
print(round(oceny.mean(), 2))     # 3.94
```

### Filtrowanie przez maskę

To pojęcie wróci za chwilę w pandas, więc warto je teraz zrozumieć.

Porównanie tablicy z liczbą nie daje `True`/`False` — daje **tablicę** wartości
logicznych, po jednej na element:

```python
print(oceny >= 4.0)
```

```text
[False  True  True False  True False  True  True]
```

Taką tablicę (nazywamy ją **maską**) można włożyć w nawias kwadratowy i dostać
tylko te elementy, dla których było `True`:

```python
print(oceny[oceny >= 4.0])
```

```text
[4.5 5.  4.  5.  4.5]
```

A skoro `True` liczy się jak 1, to zliczanie jest banalne:

```python
print((oceny >= 4.0).sum())     # 5 — tyle jest ocen co najmniej 4.0
```

---

## pandas

`numpy` radzi sobie z tablicami liczb. Ale prawdziwe dane to tabela: kolumny mają
nazwy i różne typy — tekst, liczby, daty. Do tego służy **pandas**, a jego
głównym typem jest **DataFrame** — w praktyce arkusz kalkulacyjny w Pythonie.

### Dane, na których pracujemy

Uruchom tę komórkę — utworzy plik `sklep.csv`, z którego za chwilę będziemy
czytać:

```python
csv = """produkt,kategoria,cena,sztuki,miasto
Laptop Dell,Elektronika,3499.00,4,Warszawa
Smartfon Xiaomi,Elektronika,1299.00,11,Kraków
Słuchawki JBL,Elektronika,299.00,23,Warszawa
Ekspres do kawy,AGD,899.00,7,Gdańsk
Odkurzacz Bosch,AGD,749.00,3,Kraków
Czajnik Zelmer,AGD,149.00,31,Warszawa
Fotel biurowy,Meble,1199.00,5,Gdańsk
Biurko dębowe,Meble,1599.00,2,Warszawa
Lampka LED,Meble,89.00,44,Kraków
Monitor LG,Elektronika,1099.00,9,Gdańsk
Mikrofalówka Amica,AGD,529.00,6,Warszawa
Regał Ikea,Meble,399.00,14,Kraków
"""

with open("sklep.csv", "w", encoding="utf-8") as f:
    f.write(csv)
```

### Wczytanie pliku

```python
import pandas as pd

df = pd.read_csv("sklep.csv")
df.head()
```

```text
           produkt    kategoria    cena  sztuki    miasto
0      Laptop Dell  Elektronika  3499.0       4  Warszawa
1  Smartfon Xiaomi  Elektronika  1299.0      11    Kraków
2    Słuchawki JBL  Elektronika   299.0      23  Warszawa
3  Ekspres do kawy          AGD   899.0       7    Gdańsk
4  Odkurzacz Bosch          AGD   749.0       3    Kraków
```

`df` to standardowa nazwa dla DataFrame'u — zobaczysz ją w każdym tutorialu.
Liczby z lewej (0, 1, 2, …) to **indeks**, czyli etykiety wierszy.

!!! tip "head() bez print()"

    W notatniku ostatnia linijka komórki wyświetla się sama, i to ładnie
    sformatowana jako tabela. `df.head()` da czytelniejszy wynik niż
    `print(df.head())`.

### Pierwsze spojrzenie na dane

Cztery polecenia, od których zaczynasz **zawsze**, gdy dostajesz nowy zbiór:

```python
df.shape        # (12, 5) — 12 wierszy, 5 kolumn
df.head()       # pierwsze 5 wierszy (df.head(10) — pierwsze 10)
df.info()       # nazwy kolumn, typy, ile wartości niepustych
df.describe()   # statystyki kolumn liczbowych
```

`df.info()`:

```text
<class 'pandas.DataFrame'>
RangeIndex: 12 entries, 0 to 11
Data columns (total 5 columns):
 #   Column     Non-Null Count  Dtype
---  ------     --------------  -----
 0   produkt    12 non-null     str
 1   kategoria  12 non-null     str
 2   cena       12 non-null     float64
 3   sztuki     12 non-null     int64
 4   miasto     12 non-null     str
dtypes: float64(1), int64(1), str(3)
memory usage: 612.0 bytes
```

Kolumna `Non-Null Count` to pierwsze miejsce, gdzie zobaczysz **braki danych** —
jeśli któraś kolumna ma mniej niepustych wartości niż wierszy, masz problem do
rozwiązania. Tutaj wszędzie jest 12, więc komplet.

!!! note "Starsza wersja pandas"

    W pandas 2.x kolumny tekstowe mają typ `object`, a nie `str`. To ta sama
    rzecz — tylko inaczej nazwana.

`df.describe()`:

```text
              cena     sztuki
count    12.000000  12.000000
mean    984.000000  13.250000
std     926.444818  12.990381
min      89.000000   2.000000
25%     374.000000   4.750000
50%     824.000000   8.000000
75%    1224.000000  16.250000
max    3499.000000  44.000000
```

Warto zwrócić uwagę na `mean` (984 zł) kontra `50%`, czyli medianę (824 zł).
Średnia jest wyższa od mediany, bo ciągnie ją w górę jeden drogi laptop. To
pierwszy sygnał, że w danych jest wartość odstająca.

### Wybieranie kolumn

Jedna kolumna — pojedynczy nawias kwadratowy. Wynik to **Series**, czyli jedna
kolumna z indeksem:

```python
df["cena"].head(3)
```

```text
0    3499.0
1    1299.0
2     299.0
Name: cena, dtype: float64
```

Kilka kolumn — **podwójny** nawias (bo w środku przekazujesz listę nazw). Wynik to
znowu DataFrame:

```python
df[["produkt", "cena"]].head(3)
```

```text
           produkt    cena
0      Laptop Dell  3499.0
1  Smartfon Xiaomi  1299.0
2    Słuchawki JBL   299.0
```

!!! warning "Jeden nawias czy dwa?"

    `df["cena"]` → Series (jedna kolumna).
    `df[["cena"]]` → DataFrame z jedną kolumną.
    Na razie zapamiętaj: **lista nazw w środku = podwójny nawias**. To najczęstszy
    błąd pierwszego dnia.

### Filtrowanie wierszy

Dokładnie ta sama maska co w numpy — tylko na kolumnie:

```python
df["cena"] > 1000        # Series z True/False
df[df["cena"] > 1000]    # tylko wiersze, gdzie było True
```

```text
           produkt    kategoria    cena  sztuki    miasto
0      Laptop Dell  Elektronika  3499.0       4  Warszawa
1  Smartfon Xiaomi  Elektronika  1299.0      11    Kraków
6    Fotel biurowy        Meble  1199.0       5    Gdańsk
7    Biurko dębowe        Meble  1599.0       2  Warszawa
9       Monitor LG  Elektronika  1099.0       9    Gdańsk
```

Zwróć uwagę na indeks — zostały oryginalne numery wierszy (0, 1, 6, 7, 9), więc
od razu widać, że to wycinek.

Kilka warunków naraz — każdy w **osobnych nawiasach**, łączone przez `&` (i) oraz
`|` (lub):

```python
df[(df["kategoria"] == "AGD") & (df["sztuki"] > 5)]
```

```text
               produkt kategoria   cena  sztuki    miasto
3      Ekspres do kawy       AGD  899.0       7    Gdańsk
5       Czajnik Zelmer       AGD  149.0      31  Warszawa
10  Mikrofalówka Amica       AGD  529.0       6  Warszawa
```

!!! warning "`&` zamiast `and`"

    W pandas piszemy `&` i `|`, nie `and` i `or`. Nawiasy wokół każdego warunku są
    **obowiązkowe** — bez nich Python policzy to w złej kolejności i dostaniesz
    błąd. To druga najczęstsza pomyłka pierwszego dnia.

### Sortowanie

```python
df.sort_values("cena", ascending=False).head(3)
```

```text
           produkt    kategoria    cena  sztuki    miasto
0      Laptop Dell  Elektronika  3499.0       4  Warszawa
7    Biurko dębowe        Meble  1599.0       2  Warszawa
1  Smartfon Xiaomi  Elektronika  1299.0      11    Kraków
```

`ascending=False` to sortowanie malejąco. Domyślnie (`True`) — rosnąco.

### Nowa kolumna

Nową kolumnę tworzysz, przypisując do nieistniejącej nazwy. Obliczenia idą
wierszami, bez żadnej pętli:

```python
df["wartosc"] = df["cena"] * df["sztuki"]
df[["produkt", "cena", "sztuki", "wartosc"]].head(3)
```

```text
           produkt    cena  sztuki  wartosc
0      Laptop Dell  3499.0       4  13996.0
1  Smartfon Xiaomi  1299.0      11  14289.0
2    Słuchawki JBL   299.0      23   6877.0
```

### Podstawowe statystyki

Na kolumnie działają te same metody co na tablicy numpy:

```python
df["cena"].mean()       # 984.0
df["cena"].median()     # 824.0
df["sztuki"].sum()      # 159
```

Dla kolumny tekstowej najprzydatniejsze jest `value_counts()` — zlicza, ile razy
występuje każda wartość:

```python
df["kategoria"].value_counts()
```

```text
kategoria
Elektronika    4
AGD            4
Meble          4
Name: count, dtype: int64
```

### Ściągawka

| Chcę… | Kod |
|---|---|
| wczytać CSV | `pd.read_csv("plik.csv")` |
| zobaczyć kilka pierwszych wierszy | `df.head()` |
| sprawdzić rozmiar | `df.shape` |
| sprawdzić typy i braki | `df.info()` |
| statystyki kolumn liczbowych | `df.describe()` |
| jedną kolumnę | `df["cena"]` |
| kilka kolumn | `df[["produkt", "cena"]]` |
| przefiltrować wiersze | `df[df["cena"] > 1000]` |
| dwa warunki | `df[(df["a"] > 1) & (df["b"] == "x")]` |
| posortować | `df.sort_values("cena", ascending=False)` |
| dodać kolumnę | `df["nowa"] = df["a"] * df["b"]` |
| policzyć średnią | `df["cena"].mean()` |
| zliczyć wartości | `df["kategoria"].value_counts()` |

---

## Zadanie 1 — Pogoda i filmy (6 pkt)

**Czas:** maks. 45 minut
**Środowisko:** Google Colab
**Oddanie:** notatnik `.ipynb` na Moodle

Zadanie ma dwie części. Rób je po kolei i **pokaż mi wynik każdej części, gdy
będzie gotowa** — punktacja zależy od tego, ile zrobisz na zajęciach.

### Część 1 — numpy (ok. 20 min)

Skopiuj do notatnika tablicę z temperaturami z ostatnich 14 dni:

```python
import numpy as np

temperatury = np.array([12.4, 14.1, 9.8, 15.6, 18.2, 17.9, 11.3,
                        8.7, 13.5, 16.8, 19.4, 20.1, 14.7, 10.2])
```

Policz i wypisz (każde w osobnej komórce, z opisem przez f-string):

1. Średnią temperaturę, zaokrągloną do 2 miejsc po przecinku.
2. Temperaturę najwyższą i najniższą.
3. Odchylenie standardowe, zaokrąglone do 2 miejsc.
4. **Ile dni** było cieplejszych niż średnia.
5. Tablicę zawierającą **tylko** dni cieplejsze niż 15 stopni.
6. Tę samą tablicę temperatur przeliczoną na stopnie Fahrenheita
   (wzór: `F = C * 9/5 + 32`) — **bez użycia pętli**.

Przykład oczekiwanego wyjścia dla punktu 1:

```text
Średnia temperatura: 14.48 °C
```

### Część 2 — pandas (ok. 25 min)

Uruchom tę komórkę, żeby utworzyć plik z danymi:

```python
csv = """tytul,gatunek,rok,ocena,widzowie_mln
Incepcja,Sci-Fi,2010,8.8,62.5
Skazani na Shawshank,Dramat,1994,9.3,28.3
Mroczny Rycerz,Akcja,2008,9.0,71.2
Pulp Fiction,Kryminal,1994,8.9,21.4
Ojciec chrzestny,Dramat,1972,9.2,19.8
Matrix,Sci-Fi,1999,8.7,45.6
Forrest Gump,Dramat,1994,8.8,54.1
Interstellar,Sci-Fi,2014,8.7,58.9
Gladiator,Akcja,2000,8.5,49.3
Siedem,Kryminal,1995,8.6,24.7
Wyspa tajemnic,Kryminal,2010,8.2,38.4
Diuna,Sci-Fi,2021,8.0,41.2
Joker,Dramat,2019,8.4,66.8
Mad Max Furia,Akcja,2015,8.1,37.5
"""

with open("filmy.csv", "w", encoding="utf-8") as f:
    f.write(csv)
```

Następnie:

1. Wczytaj plik do DataFrame'u i wyświetl pierwsze 5 wierszy.
2. Sprawdź rozmiar zbioru oraz typy kolumn.
3. Wyświetl **tylko tytuły i oceny** filmów z oceną wyższą niż 8.7.
4. Wyświetl filmy Sci-Fi wydane **po roku 2000** (dwa warunki naraz).
5. Wyświetl **3 filmy z największą liczbą widzów**, posortowane malejąco.
6. Dodaj kolumnę `wiek` — ile lat ma film w 2026 roku — i wyświetl tytuł, rok
   i wiek dla pierwszych 5 filmów.
7. Policz średnią ocenę wszystkich filmów oraz medianę liczby widzów.
8. Sprawdź, ile filmów przypada na każdy gatunek.
9. Na koniec dodaj **komórkę tekstową (Markdown)** i napisz w dwóch zdaniach, co
   ciekawego zauważyłeś w tych danych.

### Wskazówki

- Utknąłeś? Ściągawka jest kilka sekcji wyżej — korzystaj z niej bez skrupułów.
- Filtr z dwoma warunkami: pamiętaj o `&` i o nawiasach wokół każdego warunku.
- Punkt 5 to złożenie dwóch rzeczy, które już znasz: `sort_values(...)` i `.head(3)`.
- Jeśli coś zachowuje się nielogicznie — **Runtime → Restart and run all**.
- Nazywaj zmienne sensownie. `df` dla tabeli jest w porządku, `x1`, `x2`, `x3` już nie.

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
