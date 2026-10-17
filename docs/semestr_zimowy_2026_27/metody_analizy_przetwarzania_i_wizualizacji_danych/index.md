# Wprowadzenie

!!! info "Semestr zimowy 2026/27"

    Strona przedmiotu **Metody analizy, przetwarzania i wizualizacji danych**, prowadzonego w semestrze zimowym
    roku akademickiego 2026/27.

    Przez cały semestr przechodzimy jedną drogę — tę, którą przechodzi każdy
    analityk danych: **pozyskanie danych → czyszczenie i przetwarzanie → analiza →
    wizualizacja**. Na każdych zajęciach robimy jeden krok i przez cały czas pracujemy
    na danych tego samego sklepu internetowego, więc pod koniec semestru zobaczysz
    całą drogę od surowego eksportu do wykresu dla zarządu.

## Plan zajęć

- **Zajęcia 1 (_2026-10-18_ godz. 10:30 - 12:55 - 3h):** Skąd się biorą dane — pliki CSV z pułapkami, JSON, REST API (NBP), bazy danych i SQL z Pythona; łączenie danych z kilku źródeł
- **Zajęcia 2 (_2026-11-22_ godz. 10:30 - 12:55 - 3h):** Czyszczenie i przetwarzanie danych — braki, duplikaty, niespójne kategorie, typy danych; nowe kolumny, przedziały, tabele przestawne, format szeroki i długi
- **Zajęcia 3 (_2026-12-20_ godz. 10:30 - 12:55 - 3h):** Eksploracyjna analiza danych (EDA) — rozkłady, wartości odstające, grupowanie, korelacja; od liczb do wniosków biznesowych
- **Zajęcia 4 (_2027-01-24_ godz. 09:40 - 12:05 - 3h):** Wizualizacja danych — dobór wykresu do pytania, zasady czytelnych wykresów, jak wykresem kłamać; interaktywne wykresy w Plotly; **test końcowy**

## Zasady zaliczenia

Do zdobycia jest **100 punktów**.

### Zadania na zajęciach — 50 pkt

- Na każdych zajęciach jedno **zadanie praktyczne** — łącznie **4 zadania**.
- Każde zadanie składa się z **dwóch części** i zajmuje około **godziny**.
- Za każde zadanie maksymalnie **12 pkt** (4 × 12 = **48 pkt**).
- Dodatkowe **2 pkt** za regularność — za wykonanie na zajęciach co najmniej
  **3 z 4** zadań.

Punktacja pojedynczego zadania:

| Punkty | Za co |
|---|---|
| **12 pkt** | obie części wykonane **na zajęciach** i pokazane prowadzącemu |
| **8 pkt** | jedna z dwóch części wykonana **na zajęciach** i pokazana prowadzącemu |
| **4 pkt** | zadanie dokończone po zajęciach i przesłane **w terminie** |
| **0 pkt** | brak rozwiązania albo przesłane po terminie |

!!! note "Pokazane na zajęciach kontra dosłane później"

    Punkty za część pokazaną na zajęciach nie przepadają. Jeśli na zajęciach
    zrobiłeś jedną część (8 pkt), a całość dosłałeś później (4 pkt), liczy się
    **wariant korzystniejszy**, czyli 8 pkt. Dosyłanie ma sens przede wszystkim
    wtedy, gdy na zajęciach nie zdążyłeś pokazać nic.

Terminy przesłania rozwiązań na Moodle:

| Zadanie | Z zajęć | Termin |
|---|---|---|
| Zadanie 1 | 2026-10-18 | **2026-11-21** |
| Zadanie 2 | 2026-11-22 | **2026-12-19** |
| Zadanie 3 | 2026-12-20 | **2027-01-23** |
| Zadanie 4 | 2027-01-24 | **2027-01-31** |

!!! note "Każdy liczy na swoich danych"

    W zadaniach część danych zależy od Twojego **numeru albumu** (np. analizowana
    waluta albo miesiąc). Rozwiązania kolegów nie będą więc pasować do Twojego
    wariantu. Rozwiązania z cudzymi danymi nie zaliczam.

### Test końcowy — 50 pkt

- Test na platformie **Moodle**, pisany na ostatnich zajęciach (2027-01-24).
- Obejmuje materiał z całego semestru.
- Sprawdza rozumienie, a nie pamięć do składni — pytania są w stylu „co zrobi ten
  kod", „który wykres tu pasuje", „co jest nie tak z tym wnioskiem".

!!! danger "Weźcie to sobie do serca"

    Punkty zdobywa się przede wszystkim **na zajęciach** — to samo zadanie dokończone
    w domu jest warte 4 pkt zamiast 12. Mamy tylko **cztery spotkania**, w dodatku
    co miesiąc, więc jedno odpuszczone zadanie to od razu 12 punktów, czyli cała
    ocena w dół.

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

- **Konto Google** — do pracy w [Google Colab](https://colab.research.google.com),
  gdzie piszemy Pythona. Nie trzeba niczego instalować, wystarczy przeglądarka.
- **Laptop** na każde zajęcia — zadania robimy na miejscu.

Podstawy Pythona się przydadzą, ale wszystko, czego potrzebujesz z biblioteki
pandas, jest w notatkach do zajęć.
