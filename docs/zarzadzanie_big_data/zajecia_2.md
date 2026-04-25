# Zajęcia 2

## Czyszczenie, transformacja i przygotowanie danych

Na poprzednich zajęciach pozyskaliśmy dane z różnych źródeł — plików CSV, JSON, REST API, baz SQL. Padło stwierdzenie, że analityk spędza **60–80% czasu** na pozyskiwaniu i przygotowywaniu danych. Dziś zajmiemy się tym drugim — przygotowaniem.

Surowe dane prawie nigdy nie nadają się od razu do analizy. Brakuje wartości, są duplikaty, daty są tekstem, kategorie mają literówki, kolumny nazywają się jak popadnie. Naszym celem jest doprowadzenie zbioru do takiego stanu, w którym jest **czysty**, **spójny** i **gotowy do analizy lub modelowania**.

Po dzisiejszych zajęciach będziesz w stanie:

- ocenić jakość zbioru danych w kilku liniach kodu,
- obsłużyć brakujące wartości i duplikaty,
- konwertować typy danych (w tym daty),
- standaryzować i czyścić kolumny tekstowe,
- tworzyć pochodne kolumny i transformować dane,
- filtrować, grupować, agregować i łączyć zbiory danych,
- pisać czytelne pipeline'y w stylu method chaining.


## Pierwsze spojrzenie na dane

Zanim cokolwiek wyczyścisz — **zrozum, z czym masz do czynienia**. To zawsze pierwszy krok.

```python
import pandas as pd

url = "https://raw.githubusercontent.com/datasciencedojo/datasets/master/titanic.csv"
df = pd.read_csv(url)
```

### Podstawowe metody eksploracji

```python
df.shape           # (liczba wierszy, liczba kolumn)
df.head(5)         # pierwsze 5 wierszy
df.tail(5)         # ostatnie 5 wierszy
df.sample(5)       # losowe 5 wierszy — dobry sanity check
df.columns         # nazwy kolumn
df.dtypes          # typ każdej kolumny
```

### `info()` — podsumowanie struktury

```python
df.info()
```

Pokazuje liczbę wierszy, kolumny, ich typy i liczbę wartości nie-`null` w każdej kolumnie. To jeden z najważniejszych „pierwszych ruchów" — od razu widać:

- ile danych w ogóle masz,
- gdzie są braki,
- czy typy są sensowne (np. czy data nie jest stringiem).

### `describe()` — statystyki opisowe

```python
df.describe()              # kolumny numeryczne
df.describe(include="O")   # kolumny tekstowe (object)
df.describe(include="all") # wszystko
```

Dla kolumn numerycznych zobaczysz `count`, `mean`, `std`, `min`, `25%`, `50%`, `75%`, `max`. Dla tekstowych — liczbę unikalnych wartości i najczęstszą.

### `value_counts()` i `nunique()`

```python
df["Embarked"].value_counts()             # liczność każdej kategorii
df["Embarked"].value_counts(dropna=False) # uwzględnia NaN
df["Embarked"].nunique()                  # liczba unikalnych wartości
```

To kluczowe przy kolumnach kategorycznych — szybko widzisz literówki, dziwne wartości i niespójności (np. `"warszawa"`, `"Warszawa"`, `"WARSZAWA"` jako trzy różne kategorie).

!!! tip "Checklist „pierwszego spojrzenia"
    Zawsze, gdy dostajesz nowy zbiór:

    1. `df.shape` — czy wielkość się zgadza z oczekiwaniem?
    2. `df.head()` — jak wyglądają dane?
    3. `df.info()` — jakie typy, gdzie braki?
    4. `df.describe()` — czy zakresy mają sens?
    5. `df.duplicated().sum()` — ile duplikatów?
    6. `df.isnull().sum()` — gdzie i ile braków?

## Brakujące wartości (NaN)

W pandas brakujące wartości są reprezentowane jako `NaN` (Not a Number). Dotyczy to też tekstu — pusta komórka w CSV staje się `NaN`, a nie pustym stringiem.

### Wykrywanie braków

```python
df.isnull()             # DataFrame z True/False
df.isnull().sum()       # liczba braków w każdej kolumnie
df.isnull().sum().sum() # łączna liczba braków
df.isnull().mean() * 100  # procent braków w każdej kolumnie
```

Procentowy widok pomaga zdecydować, co dalej:

```python
braki = df.isnull().mean() * 100
braki = braki[braki > 0].sort_values(ascending=False)
print(braki)
```

### Usuwanie braków — `dropna()`

```python
df.dropna()                          # usuń każdy wiersz z choć jednym NaN
df.dropna(subset=["Age"])            # usuń wiersze z NaN tylko w kolumnie Age
df.dropna(thresh=10)                 # zostaw wiersze z minimum 10 nie-NaN
df.dropna(axis=1)                    # usuń kolumny zawierające NaN
df.dropna(axis=1, thresh=len(df)*0.7)  # usuń kolumny z >30% braków
```

!!! warning "Uwaga"
    `dropna()` **zwraca** nowy DataFrame, nie modyfikuje oryginału. Jeśli chcesz zmodyfikować w miejscu — przypisz do zmiennej:
    ```python
    df = df.dropna(subset=["Age"])
    ```

### Wypełnianie braków — `fillna()`

```python
df["Age"].fillna(df["Age"].mean())          # średnia
df["Age"].fillna(df["Age"].median())        # mediana (odporniejsza na outliery)
df["Embarked"].fillna(df["Embarked"].mode()[0])  # moda dla kategorii
df["Cabin"].fillna("Unknown")               # stała wartość
df["Age"].fillna(method="ffill")            # wartość z poprzedniego wiersza (forward fill)
df["Age"].fillna(method="bfill")            # z następnego (backward fill)
```

### Wypełnianie warunkowe (group-aware)

Czasem najlepiej imputować wartością charakterystyczną dla grupy — np. medianą wieku w klasie biletu:

```python
df["Age"] = df.groupby("Pclass")["Age"].transform(
    lambda x: x.fillna(x.median())
)
```

To bardzo silny wzorzec — wartości brakujące w `Age` są wypełniane medianą wieku **w danej klasie**, a nie globalną medianą. Dużo bliżej rzeczywistości.

### Kiedy usuwać, a kiedy wypełniać?

| Sytuacja | Co zrobić |
|---|---|
| Mało braków (< 5%) i dane losowe | Można usunąć wiersze |
| Dużo braków (> 50%) w kolumnie | Rozważ usunięcie kolumny |
| Braki niosą informację (np. brak emaila = anonimowy klient) | Wypełnij wartością wskaźnikową (np. `"unknown"`) |
| Braki w kluczowej kolumnie | Imputacja (średnia/mediana/moda lub model) |
| Dane czasowe (szeregi) | `ffill`/`bfill` często ma sens |

!!! note "Złota zasada"
    Nie ma jednej dobrej odpowiedzi. Wybór zależy od **kontekstu biznesowego**. Zawsze udokumentuj swoją decyzję — co zrobiłeś z brakami i dlaczego.

## Duplikaty

```python
df.duplicated()              # Series True/False — czy wiersz jest duplikatem
df.duplicated().sum()        # liczba duplikatów

df.drop_duplicates()                          # usuń identyczne wiersze
df.drop_duplicates(subset=["email"])          # duplikaty po jednej kolumnie
df.drop_duplicates(subset=["email"], keep="last")  # zostaw ostatnie wystąpienie
df.drop_duplicates(subset=["email"], keep=False)   # usuń WSZYSTKIE duplikaty (też "oryginał")
```

!!! tip "Subtelna pułapka"
    Domyślnie `drop_duplicates` patrzy na **wszystkie kolumny**. Dwa zamówienia tego samego klienta tego samego produktu w różnych dniach **nie są** duplikatami — różnią się datą. Zawsze świadomie wybieraj `subset`.

## Typy danych i konwersje

Surowe dane często mają złe typy. Klasyczne grzechy:

- daty wczytane jako string (`object`),
- liczby wczytane jako string (bo zawierały spację albo `"-"` jako brak),
- kolumna z `True/False` jako `object` zamiast `bool`,
- kategorie tekstowe trzymane jako `object` zamiast `category` (marnują pamięć).

### Sprawdzanie typów

```python
df.dtypes
```

### Konwersja przez `astype()`

```python
df["wiek"] = df["wiek"].astype(int)
df["cena"] = df["cena"].astype(float)
df["miasto"] = df["miasto"].astype("category")  # oszczędza pamięć przy małej liczbie unikalnych wartości
```

`astype` rzuci wyjątek, jeśli konwersja jest niemożliwa (np. `"abc"` na `int`).

### Konwersja liczb z `to_numeric()`

```python
df["cena"] = pd.to_numeric(df["cena"], errors="coerce")
```

Parametr `errors="coerce"` zamienia niekonwertowalne wartości na `NaN` zamiast rzucać wyjątek — bardzo wygodne przy brudnych danych. Niekonwertowalne komórki obsługujesz potem standardowymi narzędziami od `NaN`.

### Konwersja dat

```python
df["data"] = pd.to_datetime(df["data"])

# Z konkretnym formatem (szybsze i bezpieczniejsze)
df["data"] = pd.to_datetime(df["data"], format="%Y-%m-%d")

# Wiele formatów w jednej kolumnie? Pozwól pandas zgadywać
df["data"] = pd.to_datetime(df["data"], errors="coerce")
```

Po konwersji masz dostęp do akcesora `.dt`:

```python
df["data"].dt.year
df["data"].dt.month
df["data"].dt.day_name()        # "Monday", "Tuesday", ...
df["data"].dt.quarter
df["data"].dt.dayofweek         # 0 = poniedziałek
df["data"].dt.is_month_end
```

To zamienia datę w bogate źródło cech (feature engineering) — z jednej kolumny robisz dziesięć.

## Czyszczenie tekstu — akcesor `.str`

Tekst w pandas ma swój zestaw metod dostępnych przez `.str`:

```python
df["miasto"].str.lower()          # małe litery
df["miasto"].str.upper()          # wielkie litery
df["miasto"].str.title()          # Pierwsza Litera Każdego Słowa
df["miasto"].str.strip()          # usuwa białe znaki z początku/końca
df["miasto"].str.replace("ó", "o")  # zamiana znaków
df["email"].str.contains("@")     # czy zawiera (zwraca bool Series)
df["email"].str.startswith("admin")
df["telefon"].str.len()           # długość
df["adres"].str.split(",")        # zwraca Series list
```

### Klasyczny pipeline czyszczenia tekstu

```python
df["miasto"] = (
    df["miasto"]
    .str.strip()           # usuń białe znaki
    .str.lower()           # ujednolić wielkość liter
    .str.replace(r"\s+", " ", regex=True)  # wielokrotne spacje → jedna
)
```

To rozwiązuje 80% problemów z tekstem.

### Regex — kiedy potrzebujesz większej mocy

```python
# Wyciągnięcie kodu pocztowego z adresu
df["kod_pocztowy"] = df["adres"].str.extract(r"(\d{2}-\d{3})")

# Walidacja emaila
df["email_ok"] = df["email"].str.match(r"^[\w\.-]+@[\w\.-]+\.\w+$")

# Usunięcie wszystkich nie-cyfr
df["telefon"] = df["telefon"].str.replace(r"\D", "", regex=True)
```

!!! note "Regex w skrócie"
    - `\d` — cyfra, `\D` — wszystko poza cyfrą
    - `\w` — litera/cyfra/`_`, `\W` — wszystko inne
    - `\s` — biały znak, `\S` — niebiały znak
    - `+` — jeden lub więcej, `*` — zero lub więcej, `?` — zero lub jeden
    - `^` — początek stringa, `$` — koniec
    - `(...)` — grupa do wyciągnięcia

## Operacje na kolumnach

### Zmiana nazw

```python
df = df.rename(columns={"Age": "wiek", "Sex": "plec"})
df.columns = df.columns.str.lower()  # wszystkie nazwy małymi literami
df.columns = df.columns.str.replace(" ", "_")  # spacje na podkreślenia
```

### Usuwanie kolumn

```python
df = df.drop(columns=["Cabin", "Ticket"])
df = df.drop(columns="Name")
```

### Tworzenie nowych kolumn

Najprostszy sposób — z istniejących kolumn:

```python
df["family_size"] = df["SibSp"] + df["Parch"] + 1
df["is_adult"] = df["Age"] >= 18
df["fare_per_person"] = df["Fare"] / df["family_size"]
```

### `apply()` — gdy potrzebujesz funkcji

```python
def kategoria_wieku(wiek):
    if wiek < 18:
        return "dziecko"
    elif wiek < 65:
        return "dorosły"
    else:
        return "senior"

df["grupa_wiekowa"] = df["Age"].apply(kategoria_wieku)
```

Z lambdą, gdy logika jest prosta:

```python
df["fare_log"] = df["Fare"].apply(lambda x: np.log1p(x))
```

!!! warning "Wydajność `apply`"
    `apply` jest wygodny, ale wolny — uruchamia funkcję Pythona dla każdego wiersza. Jeśli da się tę samą operację zapisać przez wektoryzację (np. `df["a"] + df["b"]`), zrób to. Wektoryzacja na dużych zbiorach bywa nawet 100× szybsza.

### `map()` — podmiana wartości po słowniku

```python
mapowanie = {"male": "M", "female": "K"}
df["plec"] = df["Sex"].map(mapowanie)
```

Idealne przy ujednolicaniu kategorii.

### `pd.cut()` — ręczne przedziały

```python
df["grupa_wiekowa"] = pd.cut(
    df["Age"],
    bins=[0, 12, 18, 35, 60, 120],
    labels=["dziecko", "nastolatek", "młody dorosły", "dorosły", "senior"]
)
```

`pd.cut` dzieli zmienną ciągłą na zdefiniowane przedziały. Świetne do zamiany wieku, ceny, dochodu itp. na kategorie.


## Filtracja i selekcja danych

### Boolean indexing — najczęściej używana metoda

```python
df[df["Age"] > 30]                                # wiek > 30
df[(df["Age"] > 30) & (df["Sex"] == "female")]    # AND — uwaga na nawiasy!
df[(df["Age"] > 60) | (df["Pclass"] == 1)]        # OR
df[~(df["Age"] > 30)]                             # NOT (wiek <= 30 lub NaN)
```

!!! warning "Klasyczny błąd"
    W pandas używamy `&`, `|`, `~` — **nie** `and`, `or`, `not`. I zawsze nawiasujemy każdy warunek:
    ```python
    df[df["a"] > 5 & df["b"] < 3]    # ❌ — nie zadziała
    df[(df["a"] > 5) & (df["b"] < 3)]  # ✅
    ```

### `isin()` — wartość z listy

```python
df[df["Embarked"].isin(["S", "C"])]
df[~df["miasto"].isin(["Warszawa", "Kraków"])]  # zaprzeczenie
```

### `between()` — zakres

```python
df[df["Age"].between(20, 30)]              # 20 <= Age <= 30
df[df["Age"].between(20, 30, inclusive="left")]  # 20 <= Age < 30
```

### `query()` — czytelniejsza składnia

```python
df.query("Age > 30 and Sex == 'female'")
df.query("Pclass in [1, 2]")
df.query("Fare > @prog")     # `@` referuje do zmiennej Pythona
```

`query` jest często czytelniejszy niż boolean indexing przy bardziej złożonych warunkach.

### `loc` i `iloc`

```python
df.loc[0:5, ["Age", "Sex"]]      # po etykietach (0:5 włącznie!)
df.iloc[0:5, [3, 4]]              # po pozycjach (0:5 wyłącznie)
df.loc[df["Age"] > 30, "Fare"]    # boolean + selekcja kolumny
```

## Grupowanie i agregacja

`groupby` to jeden z najpotężniejszych mechanizmów pandas. Schemat: **split — apply — combine**: dziel zbiór na grupy, zastosuj operację na każdej, sklej wyniki.

```python
# Średnia cena biletu per klasa
df.groupby("Pclass")["Fare"].mean()

# Wiele agregacji naraz
df.groupby("Pclass")["Fare"].agg(["mean", "median", "min", "max", "count"])

# Różne agregacje dla różnych kolumn
df.groupby("Pclass").agg(
    sredni_wiek=("Age", "mean"),
    sredni_bilet=("Fare", "mean"),
    liczba_pasazerow=("PassengerId", "count")
)
```

Ostatnia składnia (`named aggregation`) jest najczyściejsza — od razu masz przyjazne nazwy kolumn wynikowych.

### Grupowanie po wielu kolumnach

```python
df.groupby(["Pclass", "Sex"])["Survived"].mean()
```

### `pivot_table` — alternatywa dla grupowania

```python
df.pivot_table(
    index="Pclass",
    columns="Sex",
    values="Survived",
    aggfunc="mean"
)
```

Wynik to czytelna macierz: klasa × płeć → średnia przeżywalność. Świetne do raportów.

## Łączenie zbiorów danych

### `pd.concat` — sklejanie po wierszach lub kolumnach

```python
# Sklejanie wierszy (np. dane z różnych miesięcy)
pd.concat([df_styczen, df_luty, df_marzec], ignore_index=True)

# Sklejanie kolumn (uważaj na wyrównanie indeksu!)
pd.concat([df_dane, df_dodatkowe_kolumny], axis=1)
```

### `pd.merge` — łączenie po kluczu (jak SQL JOIN)

```python
# Inner join
pd.merge(zamowienia, klienci, on="klient_id")

# Left join
pd.merge(zamowienia, klienci, on="klient_id", how="left")

# Outer join
pd.merge(zamowienia, klienci, on="klient_id", how="outer")

# Klucze o różnych nazwach
pd.merge(
    zamowienia, klienci,
    left_on="id_klienta", right_on="klient_id",
    how="left"
)
```

| Typ JOIN | Co zwraca |
|---|---|
| `inner` (domyślnie) | Tylko pasujące rekordy z obu tabel |
| `left` | Wszystkie z lewej + pasujące z prawej |
| `right` | Wszystkie z prawej + pasujące z lewej |
| `outer` | Wszystkie z obu, brakujące jako NaN |

!!! tip "Sanity check po merge"
    Po każdym merge sprawdzaj `len(wynik)` — jeśli niespodziewanie urósł, masz duplikaty po stronie klucza. Klasyczny błąd — łączenie 1:N traktowane jak 1:1.


## Method chaining — pisanie czytelnych pipeline'ów

Zamiast tworzyć dziesięć zmiennych pośrednich, pandas pozwala łączyć operacje w jeden łańcuch:

```python
wynik = (
    df
    .dropna(subset=["Age", "Fare"])
    .assign(
        family_size=lambda d: d["SibSp"] + d["Parch"] + 1,
        fare_per_person=lambda d: d["Fare"] / d["family_size"]
    )
    .query("Age >= 18")
    .groupby("Pclass")
    .agg(
        sredni_wiek=("Age", "mean"),
        sredni_bilet=("fare_per_person", "mean"),
        n=("PassengerId", "count")
    )
    .sort_values("sredni_bilet", ascending=False)
)
```

Czyta się to jak instrukcję: weź `df` → usuń NaN-y → dodaj kolumny → przefiltruj → pogrupuj → policz → posortuj. Mniej zmiennych, łatwiej śledzić logikę.

`assign` to chain-friendly sposób tworzenia nowych kolumn. `lambda d:` w środku referuje do DataFrame'u w tym kroku łańcucha — możesz odwoływać się do kolumn dodanych chwilę wcześniej.

!!! note "Kiedy chain, kiedy nie"
    Method chaining jest piękny, gdy operacje są **sekwencyjne i krótkie**. Jeśli któryś krok wymaga skomplikowanej logiki — wyjmij go do osobnej funkcji i użyj `.pipe(funkcja)` w łańcuchu. Nie zmuszaj się też do chainowania wszystkiego — czasem trzy zmienne pośrednie są czytelniejsze niż jeden łańcuch na 50 linii.

---

## Ćwiczenie łączone — kompletny pipeline na Titanicu

Zróbmy razem pełen przepływ od surowych danych do gotowej tabeli analitycznej.

### Krok 1 — Wczytanie i wstępna ocena

```python
import pandas as pd

url = "https://raw.githubusercontent.com/datasciencedojo/datasets/master/titanic.csv"
df = pd.read_csv(url)

print(f"Rozmiar: {df.shape}")
df.info()
print("\nBraki:")
print(df.isnull().sum())
```

### Krok 2 — Czyszczenie

```python
# Ujednolicamy nazwy kolumn
df.columns = df.columns.str.lower()

# Usuwamy kolumny bezużyteczne dla analizy
df = df.drop(columns=["ticket", "cabin"])

# Wypełniamy braki
df["age"] = df.groupby("pclass")["age"].transform(
    lambda x: x.fillna(x.median())
)
df["embarked"] = df["embarked"].fillna(df["embarked"].mode()[0])

# Sprawdzamy
print(df.isnull().sum())
```

### Krok 3 — Transformacje

```python
df = df.assign(
    family_size=lambda d: d["sibsp"] + d["parch"] + 1,
    is_alone=lambda d: d["family_size"] == 1,
    age_group=lambda d: pd.cut(
        d["age"],
        bins=[0, 12, 18, 35, 60, 120],
        labels=["dziecko", "nastolatek", "młody dorosły", "dorosły", "senior"]
    ),
    fare_per_person=lambda d: d["fare"] / d["family_size"]
)

df["sex"] = df["sex"].map({"male": "M", "female": "K"})

df.head()
```

### Krok 4 — Analiza

```python
# Przeżywalność wg klasy i płci
df.pivot_table(
    index="pclass",
    columns="sex",
    values="survived",
    aggfunc="mean"
).round(3)
```

```python
# Przeżywalność wg grupy wiekowej
(
    df
    .groupby("age_group", observed=True)
    .agg(
        liczba=("survived", "count"),
        przezylo=("survived", "sum"),
        przezywalnosc=("survived", "mean")
    )
    .round(3)
)
```

```python
# Czy podróżni samotni mieli inną przeżywalność?
df.groupby("is_alone")["survived"].mean()
```

To w skrócie cały dzisiejszy materiał w jednym przepływie: ocena → czyszczenie → transformacje → analiza.

---

## Zadanie

### Czyszczenie i analiza zamówień e-commerce

Twoim zadaniem jest doprowadzenie brudnego zbioru danych o zamówieniach do stanu analitycznego, a następnie odpowiedzenie na pytania biznesowe.

#### Generowanie danych

Zacznij od wygenerowania pliku z brudnymi danymi — wystarczy **raz uruchomić** poniższy skrypt:

```python
import pandas as pd
import numpy as np
from datetime import datetime, timedelta

np.random.seed(42)

n = 500
klienci = ["Anna Kowalska", "  Jan Nowak", "Anna Kowalska", "PIOTR WIŚNIEWSKI",
           "katarzyna lewandowska", "Tomasz Zieliński ", "Marta Wójcik",
           "anna kowalska ", "Krzysztof Kamiński", " Magdalena Dąbrowska"]
produkty = ["Laptop", "Mysz", "Klawiatura", "Monitor", "laptop", "MYSZ",
            "Słuchawki", "Pendrive", "monitor", "Webcam"]
kategorie = ["Elektronika", "elektronika", "ELEKTRONIKA", "Akcesoria",
             "akcesoria", "Akcesoria "]
miasta = ["Warszawa", "Kraków", "warszawa", "Gdańsk", "WROCŁAW",
          "Poznań", "Łódź ", " Warszawa", "kraków"]

start_date = datetime(2025, 1, 1)
daty_iso = [(start_date + timedelta(days=int(d))).strftime("%Y-%m-%d")
            for d in np.random.randint(0, 300, n // 2)]
daty_pl = [(start_date + timedelta(days=int(d))).strftime("%d.%m.%Y")
           for d in np.random.randint(0, 300, n // 2)]
daty = daty_iso + daty_pl
np.random.shuffle(daty)

df = pd.DataFrame({
    "order_id": range(1001, 1001 + n),
    "klient": np.random.choice(klienci, n),
    "produkt": np.random.choice(produkty, n),
    "kategoria": np.random.choice(kategorie, n),
    "miasto": np.random.choice(miasta, n),
    "ilosc": np.random.choice([1, 2, 3, 5, -1, 0], n, p=[0.5, 0.2, 0.15, 0.1, 0.025, 0.025]),
    "cena_jednostkowa": np.random.choice(
        ["199.99", "299,99", "1 499.00", "89.50", "2999", "399.00 zł", None, "abc"],
        n
    ),
    "data_zamowienia": daty,
    "email": np.random.choice(
        ["anna@gmail.com", "JAN@WP.PL", "piotr.w@onet", "marta@gmail.com",
         "tomasz@interia.pl", None, "krzysztof.k@gmail.com", "brak"],
        n
    )
})

# Wprowadzamy braki i duplikaty
for col in ["miasto", "kategoria", "data_zamowienia"]:
    df.loc[df.sample(frac=0.05, random_state=1).index, col] = np.nan

df = pd.concat([df, df.sample(20, random_state=2)], ignore_index=True)

df.to_csv("zamowienia_messy.csv", index=False)
print(f"Wygenerowano plik 'zamowienia_messy.csv' — {len(df)} wierszy")
```

#### Polecenia

**Część 1 — Eksploracja i identyfikacja problemów (3 pkt)**

1. Wczytaj plik `zamowienia_messy.csv` do DataFrame.
2. Wykonaj wstępną eksplorację (`shape`, `info`, `describe`, `isnull().sum()`, `value_counts()` dla kolumn kategorycznych) i wypisz, jakie problemy z jakością danych widzisz. Wymień **co najmniej 5 różnych problemów** w komentarzu lub komórce markdown.

**Część 2 — Czyszczenie (6 pkt)**

3. Usuń duplikaty wierszy.
4. Standaryzuj kolumny tekstowe `klient`, `produkt`, `kategoria`, `miasto` — usuń białe znaki, ujednolić wielkość liter (zaproponuj sensowną konwencję, np. `title case` dla imion i miast, lowercase dla kategorii).
5. Zamień kolumnę `data_zamowienia` na typ `datetime` — pamiętaj, że są tam dwa różne formaty.
6. Zamień kolumnę `cena_jednostkowa` na typ `float`. Niemożliwe do skonwertowania wartości potraktuj jako braki.
7. Obsłuż braki:
    - wiersze z `NaN` w `cena_jednostkowa` lub `data_zamowienia` — usuń (kluczowe dla analizy),
    - braki w `miasto` i `kategoria` — wypełnij wartością `"unknown"`,
    - braki w `email` — wypełnij `"brak_emaila"`.
8. Usuń wiersze, w których `ilosc <= 0` (zamówienia o zerowej lub ujemnej liczbie sztuk to błędne dane).

**Część 3 — Transformacje (3 pkt)**

9. Dodaj kolumnę `wartosc_zamowienia = ilosc * cena_jednostkowa`.
10. Dodaj kolumny pochodne z daty: `rok`, `miesiac`, `nazwa_dnia` (`dt.day_name()`).
11. Dodaj kolumnę `email_poprawny` (bool) — `True`, jeśli email pasuje do wzorca `coś@coś.coś`.

**Część 4 — Analiza SQL-style (3 pkt)**

Odpowiedz na poniższe pytania, używając `groupby` lub `pivot_table`:

12. Łączna wartość zamówień w każdym miesiącu.
13. Top 5 klientów pod względem łącznej wartości zamówień.
14. Średnia wartość zamówienia w każdej kategorii.

**Część 5 — Wizualizacja (1 pkt)**

15. Wykres słupkowy łącznej wartości zamówień w każdym miesiącu.

**Część 6 — Zapis (1 pkt)**

16. Zapisz oczyszczony DataFrame do pliku `zamowienia_clean.csv`.

#### Kryteria zaliczenia

- Eksploracja i identyfikacja problemów — **3 pkt**
- Czyszczenie danych (wszystkie kroki) — **6 pkt**
- Transformacje (nowe kolumny) — **3 pkt**
- Analiza (3 pytania) — **3 pkt**
- Wykres — **1 pkt**
- Zapis pliku wynikowego — **1 pkt**
- Wysłanie zadania w trakcie zajęć — **1 pkt**

**TOTAL: 18 pkt**