# Wprowadzenie

!!! info "Semestr zimowy 2026/27"

    Strona przedmiotu **Systemy BIG DATA**, prowadzonego w semestrze zimowym
    roku akademickiego 2026/27.

    Zajęcia prowadzę w dwóch grupach: **15:10 – 16:40** i **16:50 – 18:20**.
    Program obu grup jest identyczny — przychodzicie na godzinę swojej grupy.

## Plan zajęć

- **Zajęcia 1 (_2026-10-03_ godz. 15:10 - 16:40 i 16:50 - 18:20 - 4h):** Czym jest Big Data i model 5V; kiedy jeden komputer przestaje wystarczać; formaty wierszowe i kolumnowe (CSV kontra Parquet); kompresja; partycjonowanie w stylu Hive; predicate pushdown; DuckDB
- **Zajęcia 2 (_2026-10-17_ godz. 15:10 - 16:40 i 16:50 - 18:20 - 4h):** Obliczenia rozproszone: od MapReduce do Sparka — architektura Sparka, lazy evaluation, DAG, shuffle, RDD i DataFrame
- **Zajęcia 3 (_2026-11-07_ godz. 15:10 - 16:40 i 16:50 - 18:20 - 4h):** Spark SQL i DataFrame w praktyce na Databricks — Unity Catalog, joiny, funkcje okienkowe
- **Zajęcia 4 (_2026-11-21_ godz. 15:10 - 16:40 i 16:50 - 18:20 - 4h):** Delta Lake — Data Lake, Warehouse i Lakehouse; transakcje ACID, time travel, MERGE, ewolucja schematu
- **Zajęcia 5 (_2026-12-05_ godz. 15:10 - 16:40 i 16:50 - 18:20 - 4h):** Architektura medalionowa (Bronze, Silver, Gold), przetwarzanie inkrementalne i jakość danych
- **Zajęcia 6 (_2026-12-19_ godz. 15:10 - 16:40 i 16:50 - 18:20 - 4h):** Przetwarzanie strumieniowe — Structured Streaming, okna czasowe, watermarki
- **Zajęcia 7 (_2027-01-09_ godz. 15:10 - 16:40 i 16:50 - 18:20 - 4h):** Hurtownia danych w chmurze — BigQuery, model kosztowy, partycjonowanie i klastrowanie
- **Zajęcia 8 (_2027-01-23_ godz. 15:10 - 16:40 i 16:50 - 18:20 - 4h):** Big Data w praktyce: od danych do modelu — cechy na dużych danych, MLflow, bezpieczeństwo danych; **test końcowy**

## Zasady zaliczenia

Do zdobycia jest **100 punktów**.

### Zadania na zajęciach — 50 pkt

- Na każdych zajęciach jedno **zadanie praktyczne** — łącznie **8 zadań**.
- Każde zadanie składa się z **dwóch części** i zajmuje około **45 minut**.
- Za każde zadanie maksymalnie **6 pkt** (8 × 6 = **48 pkt**).
- Dodatkowe **2 pkt** za regularność i aktywność — za wykonanie na zajęciach co
  najmniej **7 z 8** zadań.

Punktacja pojedynczego zadania:

| Punkty | Za co |
|---|---|
| **6 pkt** | obie części wykonane **na zajęciach** i pokazane prowadzącemu |
| **4 pkt** | jedna z dwóch części wykonana na zajęciach |
| **2 pkt** | zadanie dokończone po zajęciach i przesłane **w terminie** |
| **0 pkt** | brak rozwiązania albo przesłane po terminie |

Termin przesłania rozwiązania na Moodle to zawsze **dzień przed kolejnymi
zajęciami**.

!!! note "Każdy liczy na swoich danych"

    W większości zadań część danych zależy od Twojego **numeru albumu** (np.
    analizowany miesiąc albo kraj). Rozwiązania kolegów nie będą więc pasować do
    Twojego wariantu. Rozwiązania z cudzymi danymi nie zaliczam.

### Test końcowy — 50 pkt

- Test na platformie **Moodle**, pisany na ostatnich zajęciach (2027-01-23).
- Obejmuje materiał z całego semestru.

!!! danger "Weźcie to sobie do serca"

    Punkty zdobywa się przede wszystkim **na zajęciach** — to samo zadanie dokończone
    w domu jest warte 2 pkt zamiast 6. Osiem zadań to połowa oceny końcowej, więc
    odpuszczanie ich w trakcie semestru odbije się na ocenie, której sam test końcowy
    już nie uratuje. **Potem nie ma zmiłuj!**

### Skala ocen

| Punkty | Ocena |
|--------|-------|
| 91–100 | 5,0 |
| 81–90 | 4,5 |
| 71–80 | 4,0 |
| 61–70 | 3,5 |
| 51–60 | 3,0 |
| 50 i poniżej | 2,0 |

## Czego potrzebujesz

Wszystkie narzędzia są darmowe i działają w przeglądarce. Nie potrzebujesz karty
płatniczej.

- **Konto Google** — do [Google Colab](https://colab.research.google.com)
  (zajęcia 1–2) i BigQuery Sandbox (zajęcia 7).
- **Konto w [Databricks Free Edition](https://www.databricks.com/signup/free-edition)**
  — od zajęć 3. Zakładacie je w ramach zadania z zajęć 1.

!!! warning "Free Edition, nie Free Trial"

    Databricks oferuje też 14-dniowy *Free Trial*, który wymaga podpięcia konta
    w chmurze i po dwóch tygodniach wygasa. Nam potrzebna jest **Free Edition** —
    darmowa bezterminowo, bez karty płatniczej.
