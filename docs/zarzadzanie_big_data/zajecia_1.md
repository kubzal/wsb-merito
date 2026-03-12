# Zajęcia 1

## Pozyskiwanie i wczytywanie danych z różnych źródeł

## 🛠️ Przygotowanie środowiska pracy

### Instalacja `uv` — nowoczesny menedżer pakietów Pythona

[`uv`](https://docs.astral.sh/uv/) to nowy, bardzo szybki menedżer pakietów i środowisk wirtualnych dla Pythona.
Zastępuje `pip`, `venv`, a nawet `pyenv` — wszystko w jednym narzędziu.

#### Windows (PowerShell)

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Po instalacji zamknij i otwórz ponownie terminal. Sprawdź, czy działa:

```powershell
uv --version
```

#### macOS / Linux

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Po instalacji zrestartuj terminal i sprawdź:

```bash
uv --version
```

---

### Tworzenie środowiska wirtualnego i instalacja pakietów

Środowisko wirtualne izoluje pakiety danego projektu od reszty systemu.
Dzięki temu unikamy konfliktów wersji między projektami.

```bash
# Tworzenie nowego projektu z wirtualnym środowiskiem
uv init zajecia-big-data
cd zajecia-big-data

# Instalacja potrzebnych pakietów
uv add jupyter jupyterlab pandas requests matplotlib
```

!!! tip "Jak to działa?"
    `uv init` tworzy folder projektu z plikiem `pyproject.toml` (lista zależności)
    i automatycznie zakłada środowisko wirtualne w folderze `.venv`.
    Polecenie `uv add` instaluje pakiet **i** dopisuje go do `pyproject.toml`.

---

### Uruchamianie Jupyter Lab / Notebook

```bash
# Jupyter Lab (polecane — nowszy interfejs)
uv run jupyter lab

# Jupyter Notebook (klasyczny interfejs)
uv run jupyter notebook
```

Po uruchomieniu w przeglądarce otworzy się interfejs Jupytera. Stwórz nowy notebook
(`New` → `Python 3`) i możesz zaczynać pracę.

!!! note "Prefiks `uv run`"
    Dodajemy `uv run` przed każdym poleceniem, żeby uruchomić je wewnątrz
    wirtualnego środowiska — bez konieczności ręcznej aktywacji.

---

### Alternatywne sposoby pracy

**Anaconda** — jeśli ktoś preferuje bardziej „klikalne" podejście:

1. Pobierz Anacondę ze strony [anaconda.com/download](https://www.anaconda.com/download)
2. Po instalacji uruchom **Anaconda Navigator**
3. Kliknij **Launch** przy Jupyter Notebook lub Jupyter Lab
4. Pakiety doinstalowujesz przez `conda install pandas requests` w terminalu Anaconda Prompt

**Google Colab** — jako rozwiązanie awaryjne (jeśli nic innego nie działa):

1. Wejdź na [colab.research.google.com](https://colab.research.google.com/)
2. Stwórz nowy notebook
3. Pakiety `pandas` i `requests` są już preinstalowane

!!! warning "Uwaga"
    Google Colab **nie jest** zalecanym rozwiązaniem docelowym.
    Zależy od dostępu do internetu, ma ograniczenia zasobów i nie uczy pracy
    z lokalnym środowiskiem — a to jest umiejętność potrzebna w pracy zawodowej.

---

## Skąd bierzemy dane w realnym świecie?

Dane, z którymi pracujemy w analizie, pochodzą z bardzo różnych źródeł i przybierają
różne formaty. Oto najczęściej spotykane:

| Format | Opis | Przykład |
|--------|------|---------|
| **CSV** | Dane tabelaryczne rozdzielone przecinkiem (lub średnikiem) | Eksport z Excela, raporty finansowe |
| **JSON** | Lekki format tekstowy, popularny w API | Odpowiedzi z REST API, konfiguracje |
| **TXT** | Zwykły tekst, różne separatory | Logi systemowe, dane legacy |
| **Bazy SQL** | Ustrukturyzowane dane relacyjne | Systemy ERP, CRM, hurtownie danych |
| **API** | Dane udostępniane przez serwisy internetowe | Pogoda, kursy walut, social media |
| **Excel (.xlsx)** | Arkusze kalkulacyjne | Raporty biznesowe |

W praktyce analityk danych spędza **60–80% czasu** na pozyskiwaniu i przygotowywaniu
danych, a dopiero resztę na właściwej analizie. Dlatego umiejętność sprawnego wczytywania
danych z różnych źródeł jest absolutnie kluczowa.

---

## Wczytywanie danych z plików CSV i JSON

### Import biblioteki

```python
import pandas as pd
```

### Wczytywanie pliku CSV

```python
# Podstawowe wczytanie
df = pd.read_csv("dane.csv")
df.head()
```

```python
# CSV z polskimi znakami i średnikiem jako separatorem (eksport z polskiego Excela)
df = pd.read_csv(
    "dane_pl.csv",
    sep=";",              # separator kolumn
    encoding="utf-8",     # kodowanie znaków (alternatywa: "cp1250" dla starszych plików)
    decimal=","           # przecinek jako separator dziesiętny
)
df.head()
```

#### Przydatne parametry `read_csv`

```python
# Plik bez nagłówków
df = pd.read_csv("dane.csv", header=None, names=["kolumna_a", "kolumna_b", "kolumna_c"])

# Pominięcie pierwszych wierszy (np. komentarze w pliku)
df = pd.read_csv("dane.csv", skiprows=3)

# Wczytanie tylko wybranych kolumn
df = pd.read_csv("dane.csv", usecols=["imie", "wiek", "miasto"])

# Automatyczne parsowanie dat
df = pd.read_csv("dane.csv", parse_dates=["data_zamowienia"])
```

#### Wczytywanie CSV z URL

Nie musisz ściągać pliku na dysk — `read_csv` akceptuje też adresy URL:

```python
url = "https://raw.githubusercontent.com/datasciencedojo/datasets/master/titanic.csv"
df = pd.read_csv(url)
df.head()
```

```python
# Podgląd podstawowych informacji
print(f"Rozmiar: {df.shape[0]} wierszy × {df.shape[1]} kolumn")
print(f"\nTypy danych:\n{df.dtypes}")
print(f"\nBraki danych:\n{df.isnull().sum()}")
```

---

### Wczytywanie pliku JSON

```python
# Prosty plik JSON (lista obiektów)
df = pd.read_json("dane.json")
df.head()
```

```python
# JSON z zagnieżdżoną strukturą — użyj json_normalize
import json

with open("dane_zagniezdzone.json", "r", encoding="utf-8") as f:
    data = json.load(f)

# Spłaszczenie zagnieżdżonej struktury do tabeli
df = pd.json_normalize(data)
df.head()
```

#### Przykład: zagnieżdżony JSON

```python
# Załóżmy, że mamy taki JSON:
dane_json = [
    {
        "imie": "Anna",
        "wiek": 28,
        "adres": {
            "miasto": "Warszawa",
            "kod": "00-001"
        }
    },
    {
        "imie": "Jan",
        "wiek": 35,
        "adres": {
            "miasto": "Kraków",
            "kod": "30-001"
        }
    }
]

df = pd.json_normalize(dane_json)
print(df)
```

Wynik:

```
  imie  wiek adres.miasto adres.kod
0 Anna    28     Warszawa    00-001
1  Jan    35       Kraków    30-001
```

Zwróć uwagę na spłaszczone nazwy kolumn (`adres.miasto`, `adres.kod`).

---

## Pobieranie danych z publicznego REST API

### Czym jest API?

**API** (Application Programming Interface) to sposób, w jaki aplikacje komunikują
się ze sobą. **REST API** to najpopularniejszy styl API w internecie — wysyłamy
zapytanie HTTP pod określony adres (URL) i dostajemy odpowiedź (najczęściej w formacie JSON).

### Biblioteka `requests`

```python
import requests

# Wysłanie zapytania GET
response = requests.get("https://jsonplaceholder.typicode.com/users")

# Sprawdzenie statusu odpowiedzi
print(f"Status: {response.status_code}")  # 200 = OK
```

### Przykład 1: JSONPlaceholder (dane testowe)

```python
import requests
import pandas as pd

# Pobranie listy użytkowników
response = requests.get("https://jsonplaceholder.typicode.com/users")
users = response.json()  # zamiana odpowiedzi na listę słowników Pythona

# Konwersja do DataFrame
df_users = pd.json_normalize(users)
print(df_users[["id", "name", "email", "address.city"]].head())
```

### Przykład 2: API Narodowego Banku Polskiego (kursy walut)

To API nie wymaga klucza — idealne do ćwiczeń.

```python
import requests
import pandas as pd

# Pobranie tabeli kursów walut (tabela A)
url = "https://api.nbp.pl/api/exchangerates/tables/A?format=json"
response = requests.get(url)
data = response.json()

# Dane są w strukturze: lista z jednym elementem → klucz "rates"
rates = data[0]["rates"]
df_kursy = pd.DataFrame(rates)

print(f"Data tabeli: {data[0]['effectiveDate']}")
print(f"Liczba walut: {len(df_kursy)}")
df_kursy.head(10)
```

```python
# Pobranie historycznych kursów EUR z ostatnich 30 dni
url = "https://api.nbp.pl/api/exchangerates/rates/A/EUR/last/30/?format=json"
response = requests.get(url)
data = response.json()

df_eur = pd.DataFrame(data["rates"])
print(df_eur.head())
print(f"\nŚredni kurs EUR: {df_eur['mid'].mean():.4f}")
print(f"Min: {df_eur['mid'].min():.4f}, Max: {df_eur['mid'].max():.4f}")
```

### Przykład 3: API z parametrami (posty użytkownika)

```python
# Pobranie postów konkretnego użytkownika
url = "https://jsonplaceholder.typicode.com/posts"
params = {"userId": 1}  # parametry zapytania

response = requests.get(url, params=params)
posts = response.json()

df_posts = pd.DataFrame(posts)
print(f"Liczba postów użytkownika 1: {len(df_posts)}")
df_posts.head()
```

### Obsługa błędów

W pracy z API zawsze mogą wystąpić problemy (brak internetu, błędny URL, limit zapytań).
Dobra praktyka to obsługa wyjątków:

```python
import requests

url = "https://api.nbp.pl/api/exchangerates/rates/A/EUR/last/30/?format=json"

try:
    response = requests.get(url, timeout=10)  # timeout = max czas oczekiwania
    response.raise_for_status()  # wyrzuci wyjątek jeśli status != 200
    data = response.json()
    print(f"Pobrano {len(data['rates'])} rekordów")
except requests.exceptions.ConnectionError:
    print("Błąd połączenia — sprawdź internet")
except requests.exceptions.Timeout:
    print("Przekroczono czas oczekiwania")
except requests.exceptions.HTTPError as e:
    print(f"Błąd HTTP: {e}")
```

---

## Podstawy SQL w Pythonie

### Czym jest SQLite?

**SQLite** to lekka baza danych przechowywana w jednym pliku. Nie wymaga instalacji
serwera — jest wbudowana w Pythona (moduł `sqlite3`). Idealna do nauki SQL
i małych projektów.

### Tworzenie bazy danych i tabeli

```python
import sqlite3

# Połączenie z bazą (jeśli plik nie istnieje, zostanie utworzony)
conn = sqlite3.connect("sklep.db")
cursor = conn.cursor()

# Tworzenie tabeli
cursor.execute("""
    CREATE TABLE IF NOT EXISTS produkty (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        nazwa TEXT NOT NULL,
        kategoria TEXT,
        cena REAL,
        stan_magazynowy INTEGER
    )
""")

conn.commit()
print("Tabela 'produkty' utworzona!")
```

### Wstawianie danych

```python
# Wstawienie pojedynczego rekordu
cursor.execute("""
    INSERT INTO produkty (nazwa, kategoria, cena, stan_magazynowy)
    VALUES (?, ?, ?, ?)
""", ("Laptop Dell", "Elektronika", 3499.99, 15))

# Wstawienie wielu rekordów naraz
dane = [
    ("Mysz Logitech", "Elektronika", 129.99, 200),
    ("Kawa ziarnista 1kg", "Spożywcze", 49.90, 500),
    ("Monitor 27 cali", "Elektronika", 1299.00, 30),
    ("Klawiatura mechaniczna", "Elektronika", 349.00, 75),
    ("Herbata Earl Grey", "Spożywcze", 12.50, 300),
    ("Słuchawki bezprzewodowe", "Elektronika", 199.99, 120),
    ("Woda mineralna 6-pack", "Spożywcze", 8.99, 1000),
    ("Pendrive 64GB", "Elektronika", 39.99, 250),
    ("Czekolada gorzka", "Spożywcze", 7.50, 400),
]

cursor.executemany("""
    INSERT INTO produkty (nazwa, kategoria, cena, stan_magazynowy)
    VALUES (?, ?, ?, ?)
""", dane)

conn.commit()
print(f"Wstawiono {len(dane) + 1} rekordów")
```

### Zapytania SQL z poziomu Pythona

```python
# Proste zapytanie
cursor.execute("SELECT * FROM produkty")
wyniki = cursor.fetchall()

for row in wyniki:
    print(row)
```

### Pandas + SQL — najwygodniejsza kombinacja

```python
import pandas as pd

# Wczytanie wyników zapytania SQL prosto do DataFrame
df = pd.read_sql_query("SELECT * FROM produkty", conn)
df.head()
```

```python
# Zapytanie z filtrowaniem
df_elektronika = pd.read_sql_query("""
    SELECT nazwa, cena, stan_magazynowy
    FROM produkty
    WHERE kategoria = 'Elektronika'
    ORDER BY cena DESC
""", conn)

print(df_elektronika)
```

```python
# Agregacje w SQL
df_stats = pd.read_sql_query("""
    SELECT
        kategoria,
        COUNT(*) as liczba_produktow,
        ROUND(AVG(cena), 2) as srednia_cena,
        SUM(stan_magazynowy) as laczny_stan
    FROM produkty
    GROUP BY kategoria
""", conn)

print(df_stats)
```

### Zapis DataFrame do bazy SQL

Możesz też zapisać dowolny DataFrame jako tabelę w bazie:

```python
# Stwórzmy przykładowy DataFrame
df_zamowienia = pd.DataFrame({
    "produkt_id": [1, 3, 2, 5, 1],
    "klient": ["Kowalski", "Nowak", "Wiśniewska", "Kowalski", "Zielińska"],
    "ilosc": [1, 2, 5, 1, 1],
    "data_zamowienia": ["2025-01-15", "2025-01-16", "2025-01-16", "2025-01-17", "2025-01-18"]
})

# Zapis do bazy
df_zamowienia.to_sql("zamowienia", conn, if_exists="replace", index=False)
print("Tabela 'zamowienia' zapisana w bazie!")
```

```python
# Zapytanie łączące dwie tabele (JOIN)
df_raport = pd.read_sql_query("""
    SELECT
        z.klient,
        p.nazwa AS produkt,
        p.cena,
        z.ilosc,
        ROUND(p.cena * z.ilosc, 2) AS wartosc,
        z.data_zamowienia
    FROM zamowienia z
    JOIN produkty p ON z.produkt_id = p.id
    ORDER BY z.data_zamowienia
""", conn)

print(df_raport)
```

```python
# Pamiętaj o zamknięciu połączenia na koniec pracy
conn.close()
```

---

## Ćwiczenie łączone (wspólne)

Połączmy wszystko, czego się dziś nauczyliśmy, w jeden pipeline:
**API → DataFrame → SQLite → analiza**.

### Krok 1: Pobierz kursy walut z API NBP

```python
import requests
import pandas as pd
import sqlite3

# Pobranie kursów walut z ostatnich 90 dni dla kilku walut
waluty = ["EUR", "USD", "GBP", "CHF"]
frames = []

for waluta in waluty:
    url = f"https://api.nbp.pl/api/exchangerates/rates/A/{waluta}/last/90/?format=json"
    response = requests.get(url)
    response.raise_for_status()
    data = response.json()

    df_temp = pd.DataFrame(data["rates"])
    df_temp["waluta"] = data["code"]
    df_temp["nazwa"] = data["currency"]
    frames.append(df_temp)

df_kursy = pd.concat(frames, ignore_index=True)
print(f"Pobrano {len(df_kursy)} rekordów")
df_kursy.head()
```

### Krok 2: Zapisz dane do bazy SQLite

```python
conn = sqlite3.connect("kursy_walut.db")

df_kursy.to_sql("kursy", conn, if_exists="replace", index=False)
print("Dane zapisane w bazie 'kursy_walut.db'")
```

### Krok 3: Wczytaj dane z bazy i wykonaj analizę

```python
# Średni kurs każdej waluty
df_srednie = pd.read_sql_query("""
    SELECT
        waluta,
        nazwa,
        COUNT(*) AS liczba_obserwacji,
        ROUND(MIN(mid), 4) AS kurs_min,
        ROUND(AVG(mid), 4) AS kurs_sredni,
        ROUND(MAX(mid), 4) AS kurs_max
    FROM kursy
    GROUP BY waluta
    ORDER BY kurs_sredni DESC
""", conn)

print(df_srednie)
```

```python
# Dzienny kurs EUR i USD — porównanie
df_porownanie = pd.read_sql_query("""
    SELECT
        effectiveDate AS data,
        MAX(CASE WHEN waluta = 'EUR' THEN mid END) AS EUR,
        MAX(CASE WHEN waluta = 'USD' THEN mid END) AS USD
    FROM kursy
    WHERE waluta IN ('EUR', 'USD')
    GROUP BY effectiveDate
    ORDER BY effectiveDate
""", conn)

df_porownanie.head()
```

```python
# Szybka wizualizacja
import matplotlib.pyplot as plt

df_porownanie["data"] = pd.to_datetime(df_porownanie["data"])

plt.figure(figsize=(12, 5))
plt.plot(df_porownanie["data"], df_porownanie["EUR"], label="EUR")
plt.plot(df_porownanie["data"], df_porownanie["USD"], label="USD")
plt.title("Kursy EUR i USD w ostatnich 90 dniach")
plt.xlabel("Data")
plt.ylabel("Kurs (PLN)")
plt.legend()
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
```

```python
conn.close()
```

---

## Zadanie

### Analiza danych o krajach świata

Twoim zadaniem jest zbudowanie pełnego pipeline'u:
**API → przetworzenie → zapis do bazy → analiza SQL → wizualizacja**.

#### Źródło danych

Użyj publicznego API **REST Countries**: `https://restcountries.com/`

To API nie wymaga klucza i zwraca dane o wszystkich krajach świata w formacie JSON.

#### Polecenia

**Część 1 — Pozyskanie danych (API → DataFrame)**

1. Pobierz dane o wszystkich krajach z API REST Countries.
2. Z odpowiedzi JSON wyciągnij i umieść w DataFrame następujące kolumny:
    - `nazwa` — nazwa potoczna kraju (pole `name.common`)
    - `stolica` — stolica (pole `capital`, uwaga: to jest lista!)
    - `region` — region świata (pole `region`)
    - `subregion` — subregion (pole `subregion`)
    - `populacja` — liczba ludności (pole `population`)
    - `powierzchnia` — powierzchnia w km² (pole `area`)
    - `waluta` — kod głównej waluty (pole `currencies`, uwaga: zagnieżdżona struktura!)
3. Wyświetl `head()`, `shape` i `dtypes` swojego DataFrame.

**Część 2 — Zapis do bazy SQLite**

4. Zapisz DataFrame jako tabelę `kraje` w bazie `kraje_swiata.db`.

**Część 3 — Analiza SQL (użyj `pd.read_sql_query`)**

5. Napisz zapytania SQL, które odpowiedzą na pytania:
    - Jaka jest łączna populacja świata?
    - Które 10 krajów ma największą populację?
    - Ile krajów jest w każdym regionie i jaka jest ich średnia populacja?
    - Które kraje mają powierzchnię większą niż Polska (~312 679 km²)?
    - Który kraj ma najwyższą gęstość zaludnienia (populacja / powierzchnia)?

**Część 4 — Wizualizacja**

6. Stwórz wykres słupkowy (bar chart) pokazujący łączną populację każdego regionu.

#### Podpowiedzi

- Do wyciągania danych z zagnieżdżonego JSON użyj pętli lub list comprehension:

```python
# Przykład wyciągania stolic (capital to lista, bierzemy pierwszy element)
stolice = [kraj.get("capital", [None])[0] for kraj in data]
```

- Pamiętaj o obsłudze `None` — nie każdy kraj ma wypełnione wszystkie pola.
- Waluta wymaga trochę gimnastyki — klucz w `currencies` to kod waluty (np. `"PLN"`),
  który jest dynamiczny. Możesz go wyciągnąć tak:

```python
def get_currency(currencies_dict):
    """Wyciąga kod pierwszej waluty z zagnieżdżonego słownika."""
    if currencies_dict:
        return list(currencies_dict.keys())[0]
    return None
```

#### Kryteria zaliczenia

- Dane pobrane z API i poprawnie sparsowane do DataFrame — **5 pkt**
- Zapis do SQLite i poprawne zapytania SQL — **6 pkt**
- Wykres — **1 pkt**
- Czystość kodu (nazwy zmiennych, komentarze) — **3 pkt**
- Wysłanie zadania w trakcie zajęć — **2 pkt** 

**TOTAL: 17 pkt**

Powodzenia! 🚀