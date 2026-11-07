# Wprowadzenie

!!! info "Semestr zimowy 2026/27"

    Strona przedmiotu **Zarządzanie Big Data**, prowadzonego w semestrze zimowym
    roku akademickiego 2026/27.

    To nie jest kurs programowania ani analizy danych. Uczymy się **zarządzać
    danymi jako zasobem firmy**: wiedzieć, co się ma, ile to kosztuje, czy można
    temu ufać, kto za to odpowiada i czy w ogóle wolno to trzymać. Przez cały
    semestr pracujemy w dziale danych fikcyjnej firmy **FreshBox**, a na każdych
    zajęciach zarząd zadaje nam jedno pytanie.

## Plan zajęć

- **Zajęcia 1 (_2026-11-08_ godz. 08:00 - 10:25 - 3h):** Dane jako zasób firmy — Big Data z perspektywy organizacji, cykl życia danych, role, dojrzałość danych; inwentaryzacja danych, słownik biznesowy i jedno źródło prawdy („Ile właściwie mamy klientów?”)
- **Zajęcia 2 (_2026-11-22_ godz. 08:00 - 10:25 - 3h):** Architektura i ekonomia Big Data — hurtownia, data lake i lakehouse; chmura kontra on-premise, TCO, model kosztowy chmury, FinOps; BigQuery Sandbox („Ile nas to kosztuje?”)
- **Zajęcia 3 (_2026-12-06_ godz. 08:00 - 10:25 - 3h):** Jakość danych — wymiary jakości, reguły jakości jako kod, scorecard, kontrakt danych, koszt złej jakości („Czy możemy ufać tym liczbom?”)
- **Zajęcia 4 (_2026-12-20_ godz. 08:00 - 10:25 - 3h):** Data governance — role i odpowiedzialność, katalog danych i metadane, klasyfikacja danych, uprawnienia i maskowanie; Databricks Unity Catalog („Kto za to odpowiada i kto może to oglądać?”)
- **Zajęcia 5 (_2027-01-10_ godz. 09:40 - 11:10 - 2h):** RODO, prywatność i cykl życia danych — pseudonimizacja kontra anonimizacja, k-anonimowość, retencja, prawo do usunięcia danych („Czy wolno nam to trzymać?”)
- **Zajęcia 6 (_2027-01-24_ godz. 08:00 - 09:30 - 2h):** Strategia danych, data mesh i dane jako produkt — podsumowanie semestru; **test końcowy**

## Zasady zaliczenia

Do zdobycia jest **100 punktów**.

### Zadania na zajęciach — 50 pkt

- Na każdych zajęciach jedno **zadanie praktyczne** — łącznie **6 zadań**.
- Każde zadanie składa się z **dwóch części** i zajmuje mniej więcej **połowę
  zajęć**: ok. 65 minut na zajęciach trzygodzinnych, ok. 35–45 minut na
  dwugodzinnych.
- Zadania 1–4 są warte po **9 pkt**, zadania 5–6 po **7 pkt**
  (4 × 9 + 2 × 7 = **50 pkt**).

Punktacja pojedynczego zadania:

| Za co | Zadania 1–4 | Zadania 5–6 |
|---|---|---|
| obie części wykonane **na zajęciach** i pokazane prowadzącemu | **9 pkt** | **7 pkt** |
| jedna z dwóch części wykonana **na zajęciach** i pokazana prowadzącemu | **6 pkt** | **5 pkt** |
| zadanie dokończone po zajęciach i przesłane **w terminie** | **3 pkt** | **2 pkt** |
| brak rozwiązania albo przesłane po terminie | **0 pkt** | **0 pkt** |

!!! note "Pokazane na zajęciach kontra dosłane później"

    Punkty za część pokazaną na zajęciach nie przepadają. Jeśli na zajęciach
    zrobiłeś jedną część (6 pkt), a całość dosłałeś później (3 pkt), liczy się
    **wariant korzystniejszy**, czyli 6 pkt. Dosyłanie ma sens przede wszystkim
    wtedy, gdy na zajęciach nie zdążyłeś pokazać nic.

Terminy przesłania rozwiązań na Moodle:

| Zadanie | Z zajęć | Termin |
|---|---|---|
| Zadanie 1 | 2026-11-08 | **2026-11-21** |
| Zadanie 2 | 2026-11-22 | **2026-12-05** |
| Zadanie 3 | 2026-12-06 | **2026-12-19** |
| Zadanie 4 | 2026-12-20 | **2027-01-09** |
| Zadanie 5 | 2027-01-10 | **2027-01-23** |
| Zadanie 6 | 2027-01-24 | **2027-01-31** |

!!! note "Każdy liczy na swoich danych"

    W zadaniach część danych zależy od Twojego **numeru albumu** — najczęściej
    jest to miasto, w którym działa FreshBox. Rozwiązania kolegów nie będą więc
    pasować do Twojego wariantu. Rozwiązań z cudzymi danymi nie zaliczam.

### Test końcowy — 50 pkt

- Test na platformie **Moodle**, pisany na ostatnich zajęciach (2027-01-24).
- Obejmuje materiał z całego semestru.
- Sprawdza rozumienie i umiejętność podjęcia decyzji, a nie pamięć do składni.
  Część pytań to krótkie kazusy w stylu „FreshBox chce zrobić X — co jest tu
  problemem?”.

!!! danger "Weźcie to sobie do serca"

    Punkty zdobywa się przede wszystkim **na zajęciach** — to samo zadanie
    dokończone w domu jest warte 3 pkt zamiast 9. Mamy tylko **sześć spotkań**,
    więc każde odpuszczone zadanie to od razu kilka punktów mniej, a test
    końcowy sam oceny nie uratuje.

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
  (większość zajęć) i BigQuery Sandbox (zajęcia 2).
- **Konto w [Databricks Free Edition](https://www.databricks.com/signup/free-edition)**
  — na zajęcia 4. Zakładamy je wspólnie pod koniec zajęć 3.
- **Laptop.** Telefon nie wystarczy — będziemy pisać zapytania SQL.

Znajomość podstaw SQL bardzo się przyda. Wszystkie potrzebne polecenia są
w ściągawkach w notatkach do zajęć.
