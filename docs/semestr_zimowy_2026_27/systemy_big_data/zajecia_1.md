# Zajęcia 1

## Kiedy jeden komputer przestaje wystarczać

Zanim uruchomimy pierwszy klaster, warto zrozumieć, **co właściwie robi dane
„dużymi”**. Zaskakująco często okazuje się, że problemem nie jest liczba
wierszy, tylko sposób, w jaki je przechowujemy i czytamy. Dziś zobaczysz, że ten
sam zbiór danych może zajmować 300 MB albo 50 MB, a to samo zapytanie może
przeczytać cały rok danych albo tylko jeden miesiąc. Zależy to wyłącznie od
formatu pliku i od tego, jak ułożymy pliki na dysku.

Po dzisiejszych zajęciach będziesz w stanie:

- wyjaśnić, czym jest Big Data i opisać problem za pomocą modelu 5V,
- powiedzieć, kiedy wystarczy jedna maszyna, a kiedy trzeba skalować wszerz,
- porównać formaty wierszowe (CSV) i kolumnowe (Parquet) i uzasadnić, dlaczego
  analityka lubi te drugie,
- opisać, jak zbudowany jest plik Parquet i do czego służą jego statystyki,
- zapisać dane jako Parquet partycjonowany w stylu Hive,
- odczytać z planu zapytania (`EXPLAIN`), ile plików i kolumn naprawdę zostało
  przeczytanych.

---

## Plan na dziś

| Blok | Czas |
|---|---|
| O przedmiocie i narzędziach | 5 min |
| Czym jest Big Data — model 5V | 7 min |
| Kiedy jedna maszyna przestaje wystarczać | 5 min |
| Formaty plików: wierszowe kontra kolumnowe, Parquet od środka | 10 min |
| Partycjonowanie i predicate pushdown | 5 min |
| Demo: NYC Taxi w pandas i DuckDB | 10 min |
| **Zadanie 1** | **45 min** |

---

## O przedmiocie

Przez osiem spotkań przejdziemy drogę od pojedynczego pliku na laptopie do
pipeline'u danych w chmurze:

| Zajęcia | Temat | Gdzie pracujemy |
|---|---|---|
| 1 | Kiedy jeden komputer przestaje wystarczać — formaty, partycjonowanie | Colab |
| 2 | Obliczenia rozproszone: od MapReduce do Sparka | Colab (PySpark) |
| 3 | Spark SQL i DataFrame w praktyce | Databricks |
| 4 | Delta Lake — data lake z transakcjami | Databricks |
| 5 | Architektura medalionowa i jakość danych | Databricks |
| 6 | Przetwarzanie strumieniowe | Databricks |
| 7 | Hurtownia danych w chmurze | BigQuery |
| 8 | Big Data w praktyce: od danych do modelu | Databricks |

Wszystkie narzędzia są **darmowe** i nie wymagają karty płatniczej:

- **Google Colab** — notatniki Pythona w przeglądarce. Dziś i za tydzień.
- **Databricks Free Edition** — Spark, Delta Lake i pipeline'y danych. Główne
  narzędzie od zajęć 3. Konto zakładacie **dziś**, w ramach zadania. Rejestracja
  i potwierdzanie maila potrafią chwilę potrwać, a na zajęciach 3 nie chcę na to
  tracić czasu.
- **BigQuery Sandbox** — hurtownia danych Google. Na zajęciach 7, wystarczy konto
  Google.

Zasady zaliczenia znajdziesz na [stronie przedmiotu](index.md#zasady-zaliczenia).
W skrócie: 50 pkt za zadania na zajęciach i 50 pkt za test końcowy.

---

## Czym jest Big Data

Nie ma jednej liczby, od której dane stają się „duże”. Najuczciwsza definicja
brzmi tak:

> Dane są duże wtedy, gdy narzędzia, które masz, przestają sobie z nimi radzić.

Dla Excela granicą jest 1 048 576 wierszy. Dla pandas na laptopie z 16 GB RAM —
kilka gigabajtów. Dla bazy danych na jednym serwerze — kilka terabajtów. Big Data
to zbiór technik na moment, w którym przekraczasz taką granicę.

### Skala

| Jednostka | Ile to jest | Przykład |
|---|---|---|
| 1 MB | 10⁶ B | kilka tysięcy wierszy w CSV |
| 1 GB | 10⁹ B | miesiąc kursów taksówek w NYC w CSV (~0,3 GB) |
| 1 TB | 10¹² B | dzienne logi dużego serwisu internetowego |
| 1 PB | 10¹⁵ B | hurtownia danych dużego banku czy operatora |
| 1 EB | 10¹⁸ B | dane przechowywane przez największych dostawców chmury |

### Model 5V

Najczęściej używany opis tego, co sprawia, że dane są trudne:

| V | Co oznacza | Przykład |
|---|---|---|
| **Volume** — wolumen | po prostu dużo danych | 40 mln kursów taksówek rocznie |
| **Velocity** — szybkość | dane napływają nieustannie i trzeba reagować na bieżąco | kliknięcia w sklepie internetowym, transakcje kartą |
| **Variety** — różnorodność | różne formaty i struktury | tabele, JSON z API, logi, obrazy, tekst |
| **Veracity** — wiarygodność | dane bywają brudne, niekompletne i sprzeczne | kurs taksówki z datą z 2002 roku w danych za 2024 |
| **Value** — wartość | z danych ma coś wynikać, inaczej to tylko koszt | rekomendacje, wykrywanie oszustw |

Przykład z kolumny *Veracity* nie jest wymyślony. Takie kursy naprawdę są
w danych, na których dziś pracujemy. Sprawdzisz to w zadaniu.

!!! info "Volume to nie wszystko"

    Dwa gigabajty danych napływające co minutę (Velocity) albo pochodzące z
    piętnastu systemów o różnych formatach (Variety) to problem Big Data, choć
    zmieszczą się na pendrivie. Na tym przedmiocie zajmiemy się wszystkimi pięcioma
    V, ale dziś skupiamy się na pierwszym.

---

## Kiedy jedna maszyna przestaje wystarczać

Komputer ma trzy zasoby, które mogą się skończyć:

- **pamięć RAM** — pandas wczytuje całą tabelę do pamięci, i to zwykle w formie
  kilka razy większej niż plik na dysku,
- **dysk** — zarówno jego pojemność, jak i szybkość odczytu,
- **procesor** — przy złożonych obliczeniach.

Najczęściej wąskim gardłem jest **szybkość odczytu**. Rzędy wielkości:

| Źródło | Przepustowość | Odczyt 1 TB |
|---|---|---|
| pamięć RAM | ~20 GB/s | ~1 min |
| dysk SSD NVMe | ~3 GB/s | ~6 min |
| dysk HDD | ~150 MB/s | ~2 h |
| sieć 1 Gb/s | ~120 MB/s | ~2,5 h |

Jeśli dane leżą na stu maszynach i każda czyta swoją setną część, ten sam terabajt
z dysków HDD przeczytasz w nieco ponad minutę zamiast w dwie godziny. Na tym
pomyśle opiera się całe przetwarzanie rozproszone.

### Skalowanie w górę i wszerz

| | Skalowanie w górę (*scale up*) | Skalowanie wszerz (*scale out*) |
|---|---|---|
| Na czym polega | większa maszyna: więcej RAM, rdzeni, szybszy dysk | więcej maszyn, każda robi część pracy |
| Zaleta | prosto — kod się nie zmienia | praktycznie bez górnej granicy |
| Wada | jest sufit i ceny rosną szybciej niż moc | koordynacja, sieć, awarie pojedynczych węzłów |
| Przykład | DuckDB, Polars, pandas na mocnym serwerze | Spark, BigQuery, Snowflake |

!!! tip "Zanim sięgniesz po klaster"

    Dzisiejszy laptop ma 16–32 GB RAM i dysk czytający kilka GB/s. Silniki
    jednomaszynowe, takie jak **DuckDB** czy **Polars**, przetwarzają na nim
    dziesiątki, a nawet setki gigabajtów, jeśli dane są w dobrym formacie. Klaster
    ma sens, gdy dane naprawdę przekraczają możliwości jednej maszyny albo gdy
    potrzebujesz odporności na awarie i pracy wielu zespołów na tych samych danych.
    Spark poznamy za tydzień. Dziś zobaczymy, jak daleko można zajść bez niego.

---

## Formaty plików

Ten sam zbiór danych możesz zapisać na wiele sposobów. To, który wybierzesz,
decyduje o rozmiarze pliku, czasie odczytu i o tym, czy w ogóle da się czytać
tylko jego część.

### Wierszowo czy kolumnowo

Weźmy małą tabelę:

| id | miasto | kwota |
|---|---|---|
| 1 | Warszawa | 25.0 |
| 2 | Kraków | 18.5 |
| 3 | Warszawa | 31.2 |

**Format wierszowy** (CSV, JSON, Avro, tabele w PostgreSQL) zapisuje dane wiersz po
wierszu:

```text
1,Warszawa,25.0 | 2,Kraków,18.5 | 3,Warszawa,31.2
```

**Format kolumnowy** (Parquet, ORC) zapisuje kolumna po kolumnie:

```text
1,2,3 | Warszawa,Kraków,Warszawa | 25.0,18.5,31.2
```

Różnica wygląda niewinnie, ale ma ogromne konsekwencje. Typowe zapytanie
analityczne, np. „średnia kwota per miasto”, potrzebuje **dwóch kolumn z
dziewiętnastu**:

- w CSV trzeba przeczytać i sparsować **cały plik**, bo kolumny są przemieszane
  w każdym wierszu,
- w Parquet czytamy **tylko te dwie kolumny**, a resztę pomijamy, nawet jej nie
  dotykając.

Druga konsekwencja: w jednej kolumnie leżą obok siebie wartości tego samego typu
i często powtarzające się (`Warszawa, Warszawa, Warszawa…`). Takie dane **świetnie
się kompresują**.

| | Wierszowy (CSV, Avro) | Kolumnowy (Parquet, ORC) |
|---|---|---|
| Dobry do | zapisywania pojedynczych rekordów, systemów transakcyjnych (OLTP) | analityki, agregacji po wielu wierszach (OLAP) |
| Odczyt kilku kolumn | trzeba przeczytać wszystko | czyta się tylko potrzebne |
| Kompresja | słaba | bardzo dobra |
| Typy danych | CSV — brak, wszystko jest tekstem | zapisane w pliku |
| Czytelny dla człowieka | CSV — tak | nie |

!!! warning "CSV gubi typy"

    W CSV nie ma informacji, że `2024-01-01 00:57:55` to data, a `00123` to kod,
    a nie liczba 123. Każde narzędzie zgaduje typy na nowo i każde może zgadnąć
    inaczej. Parquet przechowuje schemat w pliku, więc data pozostaje datą.

### Parquet od środka

Plik Parquet nie jest jednym wielkim blokiem. Ma strukturę, dzięki której da się
czytać jego fragmenty:

```text
yellow_tripdata_2024-01.parquet
├── Row group 0  (wiersze 0 – 1 048 575)
│   ├── kolumna tpep_pickup_datetime  ← skompresowane strony (pages)
│   ├── kolumna trip_distance
│   ├── ...
│   └── kolumna total_amount
├── Row group 1  (wiersze 1 048 576 – 2 097 151)
│   └── ...
├── Row group 2  (pozostałe wiersze)
│   └── ...
└── Stopka (footer)
    ├── schemat: nazwy i typy kolumn
    ├── położenie każdej kolumny w każdym row group
    └── statystyki: min, max, liczba NULL-i dla kolumn w row groupach
```

- **Row group** to poziomy kawałek tabeli, zwykle od kilkuset tysięcy do miliona
  wierszy.
- **Column chunk** to jedna kolumna w obrębie row group. To najmniejsza jednostka,
  którą czytamy z dysku.
- **Stopka** jest czytana jako pierwsza. Silnik zagląda do niej, zanim przeczyta
  jakiekolwiek dane, i na jej podstawie decyduje, czego **nie** musi czytać.

Jeśli zapytanie dotyczy `total_amount > 500`, a statystyki row group mówią
`max(total_amount) = 312`, to cały row group zostaje pominięty bez czytania.

### Kompresja i kodowanie

Parquet zmniejsza dane w dwóch krokach:

1. **Kodowanie** wykorzystuje strukturę danych:
    - *dictionary encoding*: zamiast `Warszawa, Kraków, Warszawa` zapisuje słownik
      `{0: Warszawa, 1: Kraków}` i ciąg `0, 1, 0`,
    - *run-length encoding (RLE)*: zamiast `1, 1, 1, 1, 1` zapisuje „pięć razy 1”.
2. **Kompresja** to ogólny algorytm nakładany na wynik kodowania.

Jak to wygląda dla jednego miesiąca kursów taksówek (2,96 mln wierszy, 19 kolumn):

| Format | Rozmiar |
|---|---|
| CSV | 299 MB |
| CSV + gzip | 56 MB |
| Parquet bez kompresji | 86 MB |
| Parquet + snappy | 61 MB |
| Parquet + zstd | 45 MB |

Zwróć uwagę na dwie rzeczy. Po pierwsze, Parquet **bez żadnej kompresji** jest
3,5 razy mniejszy od CSV. To zasługa samego kodowania i binarnego zapisu liczb.
Po drugie, CSV spakowany gzipem jest prawie tak mały jak Parquet, ale żeby
przeczytać z niego jedną kolumnę, trzeba rozpakować i sparsować cały plik.

| Kodek | Charakterystyka | Kiedy |
|---|---|---|
| **snappy** | szybki, umiarkowana kompresja | domyślny w Sparku, dane często czytane |
| **zstd** | dobra kompresja przy dobrej szybkości | rozsądny wybór domyślny |
| **gzip** | mocna kompresja, wolny | archiwizacja, przesyłanie przez sieć |

---

## Partycjonowanie i pushdown

### Partycjonowanie w stylu Hive

Gdy danych jest dużo, nie trzymamy ich w jednym pliku. Dzielimy je na katalogi
według wartości wybranej kolumny:

```text
taxi_2024/
├── miesiac=1/
│   └── data_0.parquet
├── miesiac=2/
│   └── data_0.parquet
├── ...
└── miesiac=12/
    └── data_0.parquet
```

Taki układ katalogów `kolumna=wartość` nazywa się **partycjonowaniem w stylu Hive**
(od narzędzia, które go spopularyzowało). Rozumieją go Spark, DuckDB, BigQuery,
Athena, pandas i praktycznie każde inne narzędzie.

Kolumny `miesiac` **nie ma w środku plików**. Jej wartość wynika z nazwy katalogu.
Dzięki temu zapytanie `WHERE miesiac = 3` może pominąć jedenaście katalogów na
podstawie samych nazw, bez otwierania choćby jednego pliku. To jest **partition
pruning**.

Jak wybrać kolumnę partycjonującą:

- **filtrujesz po niej często** — zwykle data, rzadziej region lub kraj,
- **ma umiarkowaną liczbę wartości** — miesiące albo dni tak, identyfikator
  klienta nie,
- **dzieli dane w miarę równo** — jedna partycja z 90% danych niewiele daje.

!!! warning "Problem małych plików"

    Partycjonowanie po kolumnie o wielu wartościach (np. po identyfikatorze
    strefy, minucie albo kliencie) daje tysiące katalogów z malutkimi plikami.
    Otwarcie pliku i przeczytanie jego stopki kosztuje tyle samo, niezależnie od
    tego, czy plik ma 1 KB, czy 100 MB. Tysiąc plików po 50 KB czyta się
    **wolniej** niż jeden plik 50 MB. Rozsądny rozmiar pliku to dziesiątki do
    setek megabajtów.

### Predicate pushdown i projection pushdown

Oba pojęcia oznaczają to samo: **przepchnąć część pracy jak najbliżej danych**,
żeby czytać jak najmniej.

- **Projection pushdown** — czytamy tylko kolumny potrzebne w zapytaniu
  (`SELECT a, b` → kolumny `a` i `b`, reszta zostaje na dysku).
- **Predicate pushdown** — warunek `WHERE` jest sprawdzany już w trakcie czytania
  pliku, a nie po wczytaniu wszystkiego. Silnik pomija row groupy, których
  statystyki wykluczają dopasowanie.

Razem z partycjonowaniem daje to trzy poziomy pomijania danych:

| Poziom | Mechanizm | Na podstawie czego |
|---|---|---|
| katalogi i pliki | partition pruning | nazwy katalogów (`miesiac=3`) |
| row groupy | predicate pushdown | statystyki min/max w stopce |
| kolumny | projection pushdown | lista kolumn w `SELECT` |

Najlepsze zapytanie to takie, które **nie czyta** prawie niczego.

---

## Demo: NYC Taxi w pandas i DuckDB

Komisja taksówek Nowego Jorku (NYC TLC) publikuje dane o każdym kursie żółtych
taksówek: kiedy się zaczął i skończył, skąd i dokąd, ile kosztował i jak za niego
zapłacono. Jeden miesiąc to około 3 mln wierszy, cały rok 2024 to **41 mln**.
Dane są publiczne i udostępniane jako pliki Parquet, po jednym na miesiąc.

Pracujemy w [Google Colab](https://colab.research.google.com). Pandas jest tam
od razu, a DuckDB aktualizujemy do najnowszej wersji:

```python
!pip install -q -U duckdb
```

### DuckDB w jednym akapicie

**DuckDB** to analityczna baza danych, która działa jako biblioteka Pythona. Nie
ma serwera, nie trzeba niczego konfigurować. Piszesz SQL, a w miejscu tabeli
podajesz ścieżkę do pliku. Dane są przetwarzane kolumnowo, na wszystkich rdzeniach
procesora i bez wczytywania wszystkiego do pamięci. Można ją traktować jak
„SQLite do analityki”.

```python
import duckdb

duckdb.sql("SELECT 42 AS odpowiedz")
```

### Pobranie danych

```python
import os
import urllib.request

def pobierz(rok, miesiac, katalog="dane"):
    """Pobiera jeden miesiąc kursów żółtych taksówek NYC (jeśli jeszcze go nie ma)."""
    os.makedirs(katalog, exist_ok=True)
    nazwa = f"yellow_tripdata_{rok}-{miesiac:02d}.parquet"
    sciezka = f"{katalog}/{nazwa}"
    if not os.path.exists(sciezka):
        url = f"https://d37ci6vzurychx.cloudfront.net/trip-data/{nazwa}"
        urllib.request.urlretrieve(url, sciezka)
    return sciezka

plik = pobierz(2024, 1)
print(f"{plik}: {os.path.getsize(plik) / 1e6:.1f} MB")
```

```text
dane/yellow_tripdata_2024-01.parquet: 50.0 MB
```

### Wczytanie w pandas

```python
import pandas as pd

%time df = pd.read_parquet(plik)
print(df.shape)
print(f"Pamięć: {df.memory_usage(deep=True).sum() / 1e6:.0f} MB")
```

```text
(2964624, 19)
Pamięć: 418 MB
```

Plik ma 50 MB, a w pamięci zajmuje **418 MB** — ponad osiem razy więcej. Na
dysku dane są skompresowane, w pamięci pandas trzyma je w pełnej postaci, po 8
bajtów na każdą liczbę. Zapamiętaj tę proporcję: rozmiar pliku nie mówi, czy
dane zmieszczą się w RAM.

!!! tip "`%time` i `%%time`"

    `%time` przed linijką mierzy czas jej wykonania. `%%time` w pierwszej linijce
    komórki mierzy czas całej komórki. Interesuje nas wiersz `Wall time` — tyle
    faktycznie czekałeś.

    Czasy w tym dokumencie pomijam, bo na każdym komputerze będą inne. W Colabie
    wyjdą zwykle kilka razy dłuższe niż na moim laptopie. Liczą się
    **proporcje**, nie wartości bezwzględne.

```python
df[["tpep_pickup_datetime", "trip_distance", "PULocationID",
    "payment_type", "total_amount"]].head()
```

```text
  tpep_pickup_datetime  trip_distance  PULocationID  payment_type  total_amount
0  2024-01-01 00:57:55           1.72           186             2         22.70
1  2024-01-01 00:03:00           1.80           140             1         18.75
2  2024-01-01 00:17:06           4.70           236             1         31.30
3  2024-01-01 00:36:38           1.40            79             1         17.00
4  2024-01-01 00:46:51           0.80           211             1         16.10
```

### Projection pushdown w praktyce

Teraz wczytujemy tylko dwie kolumny z dziewiętnastu:

```python
%time df2 = pd.read_parquet(plik, columns=["payment_type", "total_amount"])
print(f"Pamięć: {df2.memory_usage(deep=True).sum() / 1e6:.0f} MB")
```

```text
Pamięć: 47 MB
```

Dziewięć razy mniej pamięci, a czas odczytu spada o rząd wielkości. Parquet
pozwolił pominąć 17 kolumn bez czytania ich z dysku.

### To samo w DuckDB

```python
duckdb.sql(f"""
    SELECT payment_type,
           COUNT(*)                   AS kursy,
           ROUND(AVG(total_amount), 2) AS srednia_kwota
    FROM '{plik}'
    GROUP BY payment_type
    ORDER BY kursy DESC
""")
```

```text
┌──────────────┬─────────┬───────────────┐
│ payment_type │  kursy  │ srednia_kwota │
│    int64     │  int64  │    double     │
├──────────────┼─────────┼───────────────┤
│            1 │ 2319046 │         28.26 │
│            2 │  439191 │         22.88 │
│            0 │  140162 │         25.81 │
│            4 │   46628 │          1.77 │
│            3 │   19597 │          8.76 │
└──────────────┴─────────┴───────────────┘
```

Dwie rzeczy są tu ważne:

- DuckDB **nie wczytał tabeli do pamięci Pythona**. Przeczytał z pliku dwie
  potrzebne kolumny, policzył wynik i oddał pięć wierszy.
- Dla porównania w pandas trzeba najpierw wczytać plik (`read_parquet`), a dopiero
  potem liczyć (`groupby`). Policz oba kroki razem, a nie samo `groupby`, bo
  inaczej porównanie jest nieuczciwe.

Wynik w pandas, dla porównania:

```python
df.groupby("payment_type")["total_amount"].agg(["count", "mean"]).round(2)
```

`payment_type` to kod sposobu płatności: 1 — karta, 2 — gotówka, 0 — taryfa
elastyczna (*flex fare*), 3 — bez opłaty, 4 — spór. Średnia kwota 1,77 $ przy
sporach to nie błąd: prawie połowa spornych kursów ma kwotę ujemną (zwrot).

### CSV dla porównania

Zapiszmy ten sam miesiąc jako CSV. DuckDB robi to w sekundę, `df.to_csv()`
w pandas kilka razy dłużej:

```python
duckdb.sql(f"COPY (SELECT * FROM '{plik}') TO 'dane/taxi_2024-01.csv' (HEADER)")
print(f"CSV: {os.path.getsize('dane/taxi_2024-01.csv') / 1e6:.1f} MB")
```

```text
CSV: 299.3 MB
```

```python
%time df_csv = pd.read_csv("dane/taxi_2024-01.csv", low_memory=False)
print(f"Pamięć: {df_csv.memory_usage(deep=True).sum() / 1e6:.0f} MB")
print(df["tpep_pickup_datetime"].dtype, "kontra", df_csv["tpep_pickup_datetime"].dtype)
```

```text
Pamięć: 566 MB
datetime64[us] kontra str
```

CSV jest sześć razy większy, czyta się wolniej, zajmuje w pamięci więcej, a na
dodatek **daty stały się tekstem**, bo CSV nie przechowuje typów. Żeby z nich
skorzystać, trzeba je jeszcze raz sparsować.

### Zaglądamy do planu zapytania

`EXPLAIN` pokazuje, **jak** silnik zamierza wykonać zapytanie. Wynik jest tekstem
w drugiej kolumnie, więc wypisujemy go przez `print`:

```python
def plan(zapytanie):
    print(duckdb.sql("EXPLAIN " + zapytanie).fetchall()[0][1])

plan(f"""
    SELECT payment_type, AVG(total_amount)
    FROM '{plik}'
    WHERE trip_distance > 10
    GROUP BY payment_type
""")
```

```text
┌───────────────────────────┐
│       HASH_GROUP_BY       │
│    ────────────────────   │
│         Groups: #0        │
│    Aggregates: avg(#1)    │
└─────────────┬─────────────┘
┌─────────────┴─────────────┐
│         PROJECTION        │
│    ────────────────────   │
│        payment_type       │
│        total_amount       │
└─────────────┬─────────────┘
┌─────────────┴─────────────┐
│        PARQUET_SCAN       │
│    ────────────────────   │
│        Projections:       │
│        payment_type       │
│        total_amount       │
│                           │
│          Filters:         │
│     trip_distance>10.0    │
└───────────────────────────┘
```

Plan czytamy **od dołu**. Na samym dole jest odczyt pliku (`PARQUET_SCAN`), a w nim
widać oba rodzaje pushdownu:

- `Projections` — czytane są tylko potrzebne kolumny,
- `Filters` — warunek `WHERE` został „wepchnięty” do odczytu pliku.

Po dodaniu `ANALYZE` (`EXPLAIN ANALYZE`) zapytanie zostanie faktycznie wykonane,
a plan pokaże rzeczywistą liczbę wierszy, czas każdego kroku i **liczbę
przeczytanych plików**. To przyda się w zadaniu.

---

## Ściągawka

| Chcę… | Kod |
|---|---|
| zmierzyć czas linijki / komórki | `%time ...` / `%%time` |
| rozmiar pliku w MB | `os.path.getsize(plik) / 1e6` |
| pamięć DataFrame'u w MB | `df.memory_usage(deep=True).sum() / 1e6` |
| wczytać Parquet w pandas | `pd.read_parquet(plik)` |
| tylko wybrane kolumny | `pd.read_parquet(plik, columns=["a", "b"])` |
| zapytanie SQL na pliku | `duckdb.sql("SELECT ... FROM 'plik.parquet'")` |
| zapytanie na wielu plikach | `FROM 'dane/*.parquet'` |
| wynik DuckDB jako DataFrame | `duckdb.sql("...").df()` |
| zapisać wynik jako CSV | `COPY (SELECT ...) TO 'plik.csv' (HEADER)` |
| zapisać jako Parquet partycjonowany | `COPY (SELECT ...) TO 'katalog' (FORMAT parquet, PARTITION_BY (kol))` |
| czytać dane partycjonowane | `FROM read_parquet('katalog/*/*.parquet', hive_partitioning = true)` |
| rok / miesiąc / godzina z daty | `year(kol)`, `month(kol)`, `hour(kol)` |
| plan zapytania | `EXPLAIN SELECT ...` |
| plan z wykonaniem i liczbą plików | `EXPLAIN ANALYZE SELECT ...` |
| pamięć RAM maszyny w GB | `psutil.virtual_memory().total / 1e9` |
| zawartość katalogu | `!ls -R katalog` / `!du -sh katalog/*` |

---

## Zadanie 1 — Taksówki w Nowym Jorku (6 pkt)

**Czas:** maks. 45 minut
**Środowisko:** Google Colab i Databricks
**Oddanie:** notatnik `.ipynb` (część 1) oraz zrzut ekranu z Databricks (część 2)
na Moodle

Zadanie ma dwie części. Rób je po kolei i **pokaż mi wynik każdej części, gdy
będzie gotowa** — punktacja zależy od tego, ile zrobisz na zajęciach.

### Część 1 — od jednego miesiąca do całego roku (ok. 30 min)

Każdy analizuje inny miesiąc 2024 roku. Wylicz go ze swojego numeru albumu.
Pierwsze komórki notatnika:

```python
!pip install -q -U duckdb
```

```python
NR_ALBUMU = 123456                 # ← wpisz swój numer albumu
MIESIAC = NR_ALBUMU % 12 + 1       # Twój miesiąc: 1–12
print(f"Mój miesiąc: {MIESIAC}")
```

Wszędzie, gdzie zadanie mówi o **Twoim miesiącu**, chodzi o ten wynik. Rozwiązania
z cudzym miesiącem nie zaliczam. Skopiuj też do notatnika funkcje `pobierz()`
i `plan()` z demo.

1. **Jeden miesiąc w pandas.** Pobierz swój miesiąc i wczytaj go w pandas dwa
   razy: najpierw cały plik, potem tylko kolumny `payment_type` i `total_amount`.
   Dla obu wariantów zmierz czas i pamięć. Wypisz też rozmiar pliku na dysku.
2. **Parquet kontra CSV.** Zapisz swój miesiąc jako CSV i porównaj rozmiar
   z Parquetem. W DuckDB policz liczbę kursów i średnią kwotę `total_amount`
   w podziale na `payment_type` — raz na pliku Parquet, raz na CSV. Porównaj czasy.
3. **Cały rok, partycjonowany.** Pobierz wszystkie 12 miesięcy 2024 roku
   i zapisz je jako **Parquet partycjonowany po miesiącu** (katalog `taxi_2024/`,
   kolumna `miesiac` wyliczona z `tpep_pickup_datetime`). Zapisuj tylko kursy
   z 2024 roku — w danych są kursy z błędnymi latami. Sprawdź, ile ich jest.
4. **Ile plików czyta zapytanie.** Dla swojego miesiąca policz liczbę kursów
   i średnią kwotę `total_amount` przez `EXPLAIN ANALYZE` na dwa sposoby:

    - (a) na danych partycjonowanych, z warunkiem `WHERE miesiac = ...`,
    - (b) na oryginalnych plikach (`'dane/*.parquet'`), z warunkiem na zakres dat
      `tpep_pickup_datetime`.

    Wyniki liczbowe muszą być identyczne. Znajdź w obu planach, **ile plików
    zostało przeczytanych**.

5. **Wnioski.** Dodaj komórkę tekstową (Markdown) i w 3–4 zdaniach odpowiedz:
   skąd biorą się różnice z punktów 1 i 2 oraz dlaczego w punkcie 4 wariant (a)
   czyta jeden plik, a wariant (b) wszystkie.

### Część 2 — konto w Databricks (ok. 15 min)

Od zajęć 3 pracujemy w Databricks, więc konto zakładamy już dziś.

1. Wejdź na
   [databricks.com/signup/free-edition](https://www.databricks.com/signup/free-edition)
   i zarejestruj się (najprościej kontem Google albo mailem uczelnianym).
2. Potwierdź adres e-mail i zaloguj się do swojego workspace'u.
3. Zrób **zrzut ekranu** strony głównej workspace'u, na którym widać Twój adres
   e-mail (kliknij ikonę użytkownika w prawym górnym rogu).
4. Wgraj zrzut na Moodle razem z notatnikiem.

!!! warning "Free Edition, nie Free Trial"

    Jeśli formularz każe Ci wybrać dostawcę chmury (AWS, Azure, GCP) albo podać
    kartę płatniczą, to trafiłeś na 14-dniowy *Free Trial*. Wróć i wybierz
    **Free Edition**.

!!! tip "Mail z potwierdzeniem"

    Mail z kodem czasem przychodzi z opóźnieniem albo wpada do spamu. Możesz
    zacząć rejestrację na początku zadania, a w czasie oczekiwania pracować nad
    częścią 1.

### Wskazówki

- Ściągawka jest kilka sekcji wyżej — wszystkie potrzebne polecenia tam są.
- Plan z wykonaniem wypiszesz funkcją `plan()` z demo, dopisując `ANALYZE` na
  początku zapytania: `plan("ANALYZE SELECT ...")`.
- Kolumnę do partycjonowania dodajesz w `SELECT`:
  `SELECT *, month(tpep_pickup_datetime) AS miesiac FROM ...`.
- Jeśli zapis do istniejącego katalogu kończy się błędem, dopisz w opcjach
  `COPY` jeszcze `OVERWRITE_OR_IGNORE`.
- **Nie wczytuj całego roku w pandas.** Colab ma około 12 GB RAM, a rok danych
  zajmuje w pandas około 6 GB, plus kopie robione przy wczytywaniu. Runtime się
  zrestartuje i stracisz wszystkie zmienne. Cały rok obsługujemy w DuckDB.

### Punktacja (6 pkt)

| Punkty | Za co |
|---|---|
| **6 pkt** | obie części zrobione **na zajęciach** i pokazane prowadzącemu |
| **4 pkt** | jedna z dwóch części zrobiona na zajęciach |
| **2 pkt** | zadanie dokończone po zajęciach i wysłane na Moodle **w terminie** |
| **0 pkt** | brak rozwiązania albo wysłane po terminie |

Termin przesłania na Moodle: **2026-10-16** (dzień przed kolejnymi zajęciami).

!!! danger "Serio, róbcie to na zajęciach"

    Te same 45 minut pracy jest warte 6 pkt tu i teraz albo 2 pkt w domu. Przez
    cały semestr robi to różnicę 32 punktów, czyli całej oceny w górę lub w dół.
