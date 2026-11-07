# Zajęcia 1

## Dane jako zasób firmy

Poniedziałek, 9:00, zebranie zarządu FreshBoxa. Prezes pyta, ilu firma ma
klientów. Dyrektor obsługi klienta mówi: „43 tysiące”. Dyrektor sprzedaży:
„38 tysięcy”. Szef produktu: „21 tysięcy”. Każdy ma rację — i każdy liczy co
innego. Zebranie kończy się po godzinie bez odpowiedzi, a prezes prosi dział
danych, żeby „to wreszcie uporządkował”. Tym działem danych jesteś Ty.

Na tym przedmiocie nie uczymy się analizować danych — uczymy się **zarządzać
nimi jak zasobem firmy**. Zaczynamy od podstaw: co to znaczy, że dane są
zasobem, kto za nie odpowiada i jak sprawdzić, co w ogóle mamy.

Po dzisiejszych zajęciach będziesz w stanie:

- wyjaśnić, czym jest Big Data z perspektywy firmy, a nie technologii — jako
  źródło wartości i źródło ryzyka,
- opisać cykl życia danych i wskazać decyzje zarządcze na każdym jego etapie,
- rozróżnić role w obszarze danych: właściciela, stewarda, inżyniera,
  analityka, inspektora ochrony danych,
- ocenić dojrzałość organizacji w zarządzaniu danymi,
- przeprowadzić inwentaryzację zasobów danych,
- wyjaśnić, po co firmie słownik biznesowy i jedno źródło prawdy, i napisać
  definicję wskaźnika.

!!! tip "Przykłady z zajęć w Colabie"

    Demo z dzisiejszych zajęć jest też w gotowym notatniku. Otwórz go
    i uruchamiaj komórki po kolei, czytając materiał. Żeby zachować swoje zmiany,
    zapisz kopię: **Plik → Zapisz kopię na Dysku**.

    [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/kubzal/wsb-merito/blob/main/notebooks/semestr_zimowy_2026_27/zarzadzanie_big_data/zajecia_1.ipynb)

---

## Plan na dziś

| Blok | Czas |
|---|---|
| O przedmiocie i firmie FreshBox | 10 min |
| Big Data z perspektywy firmy | 15 min |
| Cykl życia danych | 5 min |
| Kto jest kim w świecie danych; dojrzałość danych | 10 min |
| Inwentaryzacja danych, słownik biznesowy, jedno źródło prawdy | 5 min |
| Demo: FreshBox w Colabie i DuckDB | 15 min |
| Przerwa | 10 min |
| **Zadanie 1** | **65 min** |

---

## O przedmiocie

Przez cały semestr pracujesz w dziale danych **FreshBoxa** — fikcyjnej firmy,
która dowozi zakupy spożywcze w ośmiu polskich miastach. FreshBox ma sklep
internetowy, aplikację mobilną, infolinię i kurierów. Ma też wszystkie problemy
z danymi, jakie ma prawdziwa firma tej wielkości: kilka systemów, które nie
wiedzą o sobie nawzajem, Excela „którego nie wolno ruszać” i zarząd, który
chciałby podejmować decyzje na podstawie liczb, ale nie wie, którym liczbom
wierzyć.

Na każdych zajęciach zarząd zadaje jedno pytanie:

| Zajęcia | Pytanie zarządu | Temat |
|---|---|---|
| 1 | „Ile właściwie mamy klientów?” | dane jako zasób, inwentaryzacja, definicje |
| 2 | „Ile nas to kosztuje?” | architektura i ekonomia Big Data |
| 3 | „Czy możemy ufać tym liczbom?” | jakość danych |
| 4 | „Kto za to odpowiada i kto może to oglądać?” | data governance |
| 5 | „Czy wolno nam to trzymać?” | RODO i prywatność |
| 6 | „Co dalej?” | strategia danych + test |

Narzędzia są darmowe i działają w przeglądarce: **Google Colab** z bazą
**DuckDB** (dziś i na większości zajęć), **BigQuery Sandbox** (zajęcia 2)
i **Databricks Free Edition** (zajęcia 4). Zasady zaliczenia znajdziesz na
[stronie przedmiotu](index.md#zasady-zaliczenia). W skrócie: 50 pkt za sześć
zadań na zajęciach i 50 pkt za test końcowy.

!!! info "Dlaczego DuckDB"

    DuckDB to baza danych, która działa wewnątrz notatnika i czyta pliki CSV,
    Parquet, JSON prosto z dysku — bez serwera, bez instalowania czegokolwiek
    i bez ładowania danych. Piszesz zwykły SQL. Kilka milionów wierszy liczy
    w ułamku sekundy.

---

## Big Data z perspektywy firmy

Na zajęciach z technologii Big Data opisuje się zwykle modelem **5V**. Z punktu
widzenia zarządu każda z tych cech to jednocześnie szansa i koszt:

| V | Co oznacza | Szansa dla firmy | Koszt i ryzyko |
|---|---|---|---|
| **Volume** (ilość) | dużo danych | więcej szczegółów o klientach i procesach | przechowywanie, przetwarzanie, kopie zapasowe — rachunek rośnie z każdym miesiącem |
| **Velocity** (szybkość) | dane napływają ciągle | reakcja w czasie rzeczywistym: wykrywanie oszustw, dynamiczne ceny | systemy działające 24/7, dyżury, bardziej złożona architektura |
| **Variety** (różnorodność) | tabele, logi, teksty, obrazy | pełniejszy obraz klienta | każdy format to inne narzędzia i inne kompetencje |
| **Veracity** (wiarygodność) | dane bywają błędne i sprzeczne | — | złe decyzje podjęte na złych danych |
| **Value** (wartość) | po co to wszystko | lepsze decyzje, nowe produkty | wartość nie pojawia się sama — trzeba za nią zapłacić |

Ostatni wiersz jest najważniejszy. **Dane same w sobie nie mają wartości.**
Wartość powstaje dopiero wtedy, gdy ktoś na ich podstawie podejmie lepszą
decyzję. Do tego czasu dane są wyłącznie kosztem: trzeba je przechowywać,
chronić, utrzymywać i pilnować, żeby nie wyciekły.

### Dane jako aktywo

Mówi się, że „dane to nowa ropa”. To porównanie jest prawdziwe tylko
w połowie. Jak ropa, dane trzeba wydobyć i przetworzyć, zanim będą do czegoś
przydatne. W odróżnieniu od ropy:

- **nie zużywają się** — ten sam zbiór może jednocześnie służyć finansom,
  marketingowi i logistyce,
- **starzeją się** — adres klienta sprzed pięciu lat jest wart mniej niż
  wczorajszy, a czasem jest wręcz szkodliwy,
- **nie ma ich w bilansie** — firma dokładnie wie, ile ma samochodów
  dostawczych, ale zwykle nie wie, ile ma tabel z danymi klientów i kto z nich
  korzysta,
- **mogą stać się zobowiązaniem** — dane osobowe, których firma nie potrzebuje,
  nie przynoszą zysku, a w razie wycieku przynoszą karę.

### Dark data — dane, których nikt nie używa

Duża część danych zbieranych przez firmy jest zapisywana „na wszelki wypadek”
i nigdy więcej nie jest czytana: stare logi, kopie kopii, eksporty do Excela
sprzed trzech lat, tabele po projektach, które już się skończyły. Nazywa się to
**dark data**. Szacunki różnią się między źródłami, ale zgodnie mówią o **ponad
połowie** wszystkich danych w typowej organizacji.

Dark data kosztuje podwójnie: płacisz za jej przechowywanie i ponosisz ryzyko,
bo nikt nie wie, co w niej jest. Dlatego pierwszym krokiem zarządzania danymi
nie jest kupno nowej platformy, tylko **inwentaryzacja**: sprawdzenie, co
właściwie mamy.

---

## Cykl życia danych

Każdy zbiór danych przechodzi przez te same etapy. Na każdym z nich ktoś
podejmuje decyzję — świadomie albo nie.

```text
  POZYSKANIE ──► PRZECHOWYWANIE ──► PRZETWARZANIE ──► UDOSTĘPNIANIE ──► ARCHIWIZACJA ──► USUNIĘCIE
  skąd i po co?   gdzie i za ile?   kto i jak liczy?  kto może czytać?  jak długo?        kiedy i czy
                                                                                          naprawdę?
```

| Etap | Pytanie zarządcze | Przykład z FreshBoxa |
|---|---|---|
| Pozyskanie | Po co zbieramy te dane? Czy mamy na to zgodę? | aplikacja zapisuje każde kliknięcie — czy ktoś tego potrzebuje? |
| Przechowywanie | Gdzie leżą, ile to kosztuje, kto ma dostęp? | CRM, system sprzedażowy, logi aplikacji, Excel na dysku Ani |
| Przetwarzanie | Kto liczy wskaźniki i według jakiej definicji? | trzy działy liczą „klientów” na trzy sposoby |
| Udostępnianie | Kto może zobaczyć dane i w jakiej postaci? | czy marketing powinien widzieć numery telefonów? |
| Archiwizacja | Jak długo trzymamy dane i po co? | logi aplikacji: 3 miesiące, zamówienia: rok, klienci: na zawsze? |
| Usunięcie | Kiedy usuwamy i czy na pewno ze wszystkich miejsc? | konto „usunięte” w CRM nadal ma pełne dane osobowe |

Najczęściej zaniedbywany jest ostatni etap. Firmy chętnie zbierają dane, ale
rzadko je usuwają — a od 2018 roku RODO wymaga, żeby dane osobowe były
przechowywane **nie dłużej, niż to konieczne**. Wrócimy do tego na zajęciach 5.

---

## Kto jest kim w świecie danych

Dane bez właściciela to dane niczyje: wszyscy z nich korzystają, nikt nie
odpowiada za ich jakość. Dlatego zarządzanie danymi zaczyna się od ról.

| Rola | Za co odpowiada | Kto to zwykle jest |
|---|---|---|
| **CDO** (*Chief Data Officer*) | strategia danych całej firmy, budżet, priorytety | członek zarządu lub dyrektor podlegający zarządowi |
| **Właściciel danych** (*data owner*) | decyduje, kto ma dostęp, akceptuje definicje, odpowiada za jakość | kierownik **biznesowy** — np. dyrektor sprzedaży dla danych o zamówieniach |
| **Steward danych** (*data steward*) | pilnuje jakości i definicji na co dzień, odpowiada na pytania „co oznacza ta kolumna?” | doświadczony pracownik działu, który zna dane od podszewki |
| **Inżynier danych** | systemy i pipeline'y: żeby dane dotarły na czas i w całości | dział IT / danych |
| **Analityk / data scientist** | odpowiada na pytania biznesowe na podstawie danych | dział danych albo działy biznesowe |
| **Inspektor ochrony danych** (IOD) | zgodność z RODO, kontakt z UODO | wymagany prawem w wielu firmach |
| **Użytkownik biznesowy** | korzysta z raportów i podejmuje decyzje | wszyscy pozostali |

Najważniejsza i najczęściej źle rozumiana rola to **właściciel danych**.
Właścicielem danych o zamówieniach **nie jest** dział IT, choć to on utrzymuje
system sprzedażowy. IT jest **opiekunem technicznym** (*custodian*): dba, żeby
system działał. Właścicielem jest biznes, bo to biznes wie, co dane znaczą, i to
biznes ponosi skutki ich złej jakości.

!!! warning "„Dane należą do IT” — najczęstszy błąd"

    Jeśli w firmie na pytanie „kto odpowiada za dane klientów?” wszyscy
    wskazują dział IT, to w praktyce nie odpowiada nikt. IT nie wie, czy
    klient „zablokowany” powinien być liczony jako aktywny. Wie to dział
    obsługi klienta.

### Dojrzałość danych w organizacji

Firmy różnią się tym, jak bardzo dane są u nich uporządkowane. Istnieje wiele
modeli dojrzałości (m.in. DAMA, Gartner, CMMI) — różnią się szczegółami, ale
mają podobny szkielet:

| Poziom | Nazwa | Jak to wygląda |
|---|---|---|
| 1 | **Chaos** | dane w Excelach i skrzynkach mailowych, każdy liczy po swojemu, „raport” to praca ręczna |
| 2 | **Raportowanie** | jest hurtownia i raporty, ale definicje różnią się między działami |
| 3 | **Zarządzanie** | są właściciele danych, słownik biznesowy, wspólne definicje kluczowych wskaźników |
| 4 | **Decyzje oparte na danych** | jakość jest mierzona, dostęp kontrolowany, decyzje zarządu opierają się na wspólnych liczbach |
| 5 | **Dane jako produkt** | zespoły udostępniają dane innym jak produkty: z opisem, gwarancją jakości i właścicielem |

FreshBox jest dziś gdzieś między poziomem 1 a 2. Na koniec semestru
zaproponujemy, jak dojść do poziomu 3–4.

---

## Inwentaryzacja danych

Nie da się zarządzać czymś, o czym się nie wie. **Inwentaryzacja danych** to
spis wszystkich zbiorów danych w firmie z podstawowymi informacjami o każdym:

| Atrybut | Pytanie | Po co |
|---|---|---|
| nazwa i opis | co to jest? | żeby nowa osoba wiedziała, czego szukać |
| system źródłowy | skąd pochodzą? | żeby wiedzieć, kogo pytać |
| właściciel | kto odpowiada? | żeby było kogo pytać |
| format, wolumen | jak duże, w jakiej postaci? | koszty, wybór narzędzi |
| zakres dat, częstotliwość zmian | od kiedy, jak często aktualizowane? | czy można na tym polegać |
| dane osobowe | czy są, jakie? | RODO, bezpieczeństwo |
| konsumenci | kto z tego korzysta? | co się zepsuje, jeśli zbiór zniknie |

!!! info "Inwentaryzacja a RODO"

    RODO wymaga od większości firm prowadzenia **rejestru czynności
    przetwarzania** danych osobowych — w praktyce to inwentaryzacja ograniczona do
    danych osobowych. Firma, która ma inwentaryzację danych, ma też połowę tego
    rejestru za darmo.

---

## Słownik biznesowy i jedno źródło prawdy

Wróćmy do zebrania zarządu. Trzy liczby klientów wzięły się stąd, że każdy
dział liczy w **swoim** systemie i według **swojej** definicji:

- obsługa klienta liczy **konta w CRM**, które nie są usunięte,
- sprzedaż liczy klientów, którzy **coś kupili** w ostatnim czasie,
- produkt liczy klientów, którzy **logowali się do aplikacji**.

Żadna z tych liczb nie jest błędna. Błędem jest to, że wszystkie nazywają się
„klienci”. Lekarstwem jest **słownik biznesowy** (*business glossary*): spis
pojęć, z których korzysta firma, z jedną, zatwierdzoną definicją każdego.

Przykładowy wpis:

| Pole | Wartość |
|---|---|
| **Pojęcie** | Aktywny klient |
| **Definicja** | klient, który w ciągu ostatnich 90 dni złożył co najmniej jedno nieanulowane zamówienie |
| **Źródło danych** | system sprzedażowy, tabela zamówień |
| **Sposób liczenia** | liczba różnych `id_klienta` w zamówieniach o statusie innym niż anulowane, z ostatnich 90 dni |
| **Właściciel** | dyrektor sprzedaży |
| **Częstotliwość** | codziennie, stan na koniec dnia |
| **Uwagi** | nie obejmuje zamówień gości (bez konta) |

Słownik prowadzi do drugiej zasady: **jedno źródło prawdy** (*single source of
truth*). Dla każdego kluczowego wskaźnika istnieje jedno miejsce, z którego się
go liczy, i wszystkie raporty biorą go stamtąd. Jeśli finanse i sprzedaż
podają różne przychody, to któryś z raportów nie korzysta ze źródła prawdy.

!!! tip "Wskaźnik bez definicji to opinia"

    Zanim zaczniesz dyskutować o liczbie, zapytaj: „jak to jest policzone?”.
    W połowie przypadków okaże się, że rozmówcy liczą różne rzeczy.

---

## Demo: FreshBox w Colabie i DuckDB

Robimy dziś pierwszy krok inwentaryzacji: sprawdzamy, jakie dane FreshBox ma
w ogóle do dyspozycji. Dział IT przysłał nam pięć plików:

| Plik | Skąd | Co zawiera |
|---|---|---|
| `klienci.csv` | CRM (obsługa klienta) | dane klientów i status konta |
| `zamowienia.parquet` | system sprzedażowy | zamówienia z ostatnich 12 miesięcy |
| `zdarzenia_aplikacji.jsonl.gz` | aplikacja mobilna | logi: logowania, wyświetlenia produktów, koszyk |
| `reklamacje.xlsx` | Excel obsługi klienta | reklamacje wpisywane ręcznie |
| `miasta.csv` | słownik | miasta, w których działa FreshBox |

„Dziś” w FreshBoxie to **31 października 2026** — na ten dzień wygenerowano dane.

### Pobranie danych

W Colabie najpierw aktualizujemy DuckDB:

```python
!pip install -q -U duckdb
```

Pobieramy pliki do folderu `freshbox` — to nasz mały „data lake”:

```python
import os
import urllib.request

import duckdb
import pandas as pd

URL = "https://kubzal.github.io/wsb-merito/semestr_zimowy_2026_27/zarzadzanie_big_data/dane/"
PLIKI = ["klienci.csv", "zamowienia.parquet", "zdarzenia_aplikacji.jsonl.gz",
         "reklamacje.xlsx", "miasta.csv"]

os.makedirs("freshbox", exist_ok=True)
for plik in PLIKI:
    urllib.request.urlretrieve(URL + plik, f"freshbox/{plik}")

for plik in PLIKI:
    print(f"{plik:30} {os.path.getsize('freshbox/' + plik) / 1e6:6.1f} MB")
```

```text
klienci.csv                       6.1 MB
zamowienia.parquet                5.9 MB
zdarzenia_aplikacji.jsonl.gz     10.0 MB
reklamacje.xlsx                   0.1 MB
miasta.csv                        0.0 MB
```

Zauważ, że zamówienia są w Parquecie — formacie kolumnowym, skompresowanym.
Tabela zamówień ma ponad 400 tysięcy wierszy, a zajmuje mniej niż CSV
z 50 tysiącami klientów. Ile to oznacza w pieniądzach, policzymy na zajęciach 2.

### Pierwsze spojrzenie: `DESCRIBE`

DuckDB czyta plik bezpośrednio — wystarczy podać jego nazwę w `FROM`. Na
początek: jakie kolumny ma tabela klientów?

```python
duckdb.sql("DESCRIBE SELECT * FROM 'freshbox/klienci.csv'").project("column_name, column_type")
```

```text
┌──────────────────┬─────────────┐
│   column_name    │ column_type │
├──────────────────┼─────────────┤
│ id_klienta       │ BIGINT      │
│ imie             │ VARCHAR     │
│ nazwisko         │ VARCHAR     │
│ email            │ VARCHAR     │
│ telefon          │ BIGINT      │
│ data_urodzenia   │ DATE        │
│ plec             │ VARCHAR     │
│ miasto           │ VARCHAR     │
│ kod_pocztowy     │ VARCHAR     │
│ data_rejestracji │ TIMESTAMP   │
│ zgoda_marketing  │ VARCHAR     │
│ status_konta     │ VARCHAR     │
│ data_usuniecia   │ DATE        │
└──────────────────┴─────────────┘
```

Już na tym etapie widać dwie rzeczy ważne dla zarządzania:

- to są **dane osobowe** — imię, nazwisko, e-mail, telefon, data urodzenia,
- jest kolumna `data_usuniecia`. Skoro klient został „usunięty”, to dlaczego
  jego dane nadal tu są?

DuckDB sam zgadł typy kolumn. Telefon uznał za liczbę (`BIGINT`) — dla numeru
telefonu to zły pomysł, bo na liczbach można liczyć średnią, a na telefonach nie.

### Statystyki jednym poleceniem: `SUMMARIZE`

```python
duckdb.sql("""
SELECT column_name, column_type, min, max, approx_unique, null_percentage
FROM (SUMMARIZE SELECT * FROM 'freshbox/zamowienia.parquet')
""")
```

```text
┌────────────────┬──────────────┬─────────────────────┬─────────────────────┬───────────────┬─────────────────┐
│  column_name   │ column_type  │         min         │         max         │ approx_unique │ null_percentage │
├────────────────┼──────────────┼─────────────────────┼─────────────────────┼───────────────┼─────────────────┤
│ id_zamowienia  │ BIGINT       │ 5000001             │ 5407189             │        434321 │            0.00 │
│ ts_zamowienia  │ TIMESTAMP_NS │ 2025-11-01 00:01:52 │ 2026-10-31 23:58:26 │        409992 │            0.00 │
│ id_klienta     │ BIGINT       │ 100001              │ 150000              │         36519 │            2.91 │
│ miasto         │ VARCHAR      │ Gdańsk              │ Łódź                │             8 │            0.00 │
│ kanal          │ VARCHAR      │ aplikacja           │ www                 │             3 │            0.00 │
│ wartosc_brutto │ DOUBLE       │ 12.92               │ 2551.18             │         51832 │            0.00 │
│ koszt_dostawy  │ DOUBLE       │ 0.0                 │ 9.99                │             2 │            0.00 │
│ status         │ VARCHAR      │ ANUL                │ zwrot               │             6 │            0.00 │
│ id_kuriera     │ INTEGER      │ 7001                │ 14025               │           196 │            0.00 │
└────────────────┴──────────────┴─────────────────────┴─────────────────────┴───────────────┴─────────────────┘
```

`SUMMARIZE` to szybka inwentaryzacja jednej tabeli: zakres dat (od 1 listopada
2025 do 31 października 2026 — dokładnie rok), liczba różnych wartości
i odsetek braków. Prawie 3% zamówień nie ma `id_klienta`. Kim są ci klienci?
`approx_unique` to liczba **przybliżona** — DuckDB szacuje ją, żeby było szybko.

Kolumna `status` ma 6 różnych wartości. Zobaczmy jakie:

```python
duckdb.sql("""
SELECT status, COUNT(*) AS liczba
FROM 'freshbox/zamowienia.parquet'
GROUP BY status
ORDER BY liczba DESC
""")
```

```text
┌──────────────┬────────┐
│    status    │ liczba │
├──────────────┼────────┤
│ dostarczone  │ 375852 │
│ cancelled    │  10384 │
│ anulowane    │   9518 │
│ zwrot        │   7733 │
│ ANUL         │   2502 │
│ w_realizacji │   1200 │
└──────────────┴────────┘
```

Anulowane zamówienie zapisano na **trzy sposoby**: `anulowane`, `cancelled`
i `ANUL` — bo zamówienia spływają z trzech kanałów (www, aplikacja, infolinia),
a każdy budował inny zespół. Ktoś, kto policzy przychód z wykluczeniem tylko
statusu `anulowane`, zawyży wynik i nawet się nie zorientuje.

### Logi aplikacji: JSON

Aplikacja zapisuje zdarzenia w formacie **JSON lines** — jeden obiekt JSON na
linię — spakowanym gzipem. DuckDB czyta go tak samo jak CSV:

```python
duckdb.sql("SELECT * FROM 'freshbox/zdarzenia_aplikacji.jsonl.gz' LIMIT 3")
```

```text
┌─────────────────────┬─────────┬────────────┬─────────────┬───┬──────────────────┬──────────┬─────────────┬─────────┐
│         ts          │   typ   │ id_klienta │    sesja    │ … │ wersja_aplikacji │  miasto  │ id_produktu │  email  │
├─────────────────────┼─────────┼────────────┼─────────────┼───┼──────────────────┼──────────┼─────────────┼─────────┤
│ 2026-08-01 00:00:15 │ login   │     147167 │ sa98cb3fb07 │ … │ 4.0.3            │ Warszawa │        NULL │ NULL    │
│ 2026-08-01 00:00:50 │ login   │     145080 │ s8144800215 │ … │ 4.0.3            │ Katowice │        NULL │ NULL    │
│ 2026-08-01 00:01:08 │ login   │     106377 │ s4dc6ad009f │ … │ 4.0.3            │ Wrocław  │        NULL │ NULL    │
└─────────────────────┴─────────┴────────────┴─────────────┴───┴──────────────────┴──────────┴─────────────┴─────────┘
  3 rows                                                                                         9 columns (8 shown)
```

Ostatnia kolumna to `email`. W logach aplikacji, czyli w danych, które czyta
zespół produktu, analitycy i każdy, kto ma dostęp do logów. W pierwszych wierszach
jest pusta — jak często nie jest i skąd się tam wzięła, sprawdzimy na zajęciach 5.

```python
duckdb.sql("""
SELECT typ, COUNT(*) AS liczba, MIN(ts) AS od, MAX(ts) AS do
FROM 'freshbox/zdarzenia_aplikacji.jsonl.gz'
GROUP BY typ
ORDER BY liczba DESC
""")
```

```text
┌───────────────────────┬────────┬─────────────────────┬─────────────────────┐
│          typ          │ liczba │         od          │         do          │
├───────────────────────┼────────┼─────────────────────┼─────────────────────┤
│ wyswietlenie_produktu │ 484785 │ 2026-08-01 00:01:17 │ 2026-10-31 23:59:52 │
│ login                 │ 200633 │ 2026-08-01 00:00:15 │ 2026-10-31 23:59:32 │
│ dodanie_do_koszyka    │  69669 │ 2026-08-01 00:04:09 │ 2026-10-31 23:59:16 │
└───────────────────────┴────────┴─────────────────────┴─────────────────────┘
```

Logi zaczynają się 1 sierpnia, czyli obejmują **tylko trzy miesiące**.
Starsze zespół aplikacji kasuje, bo „za dużo miejsca zajmowały”. To też jest
decyzja zarządcza (retencja), tylko że nikt jej świadomie nie podjął.

### Excel, którego „nie wolno ruszać”

Reklamacje obsługa klienta prowadzi w Excelu. Nie ma ich w żadnym systemie —
to klasyczne **shadow IT**: dane, o których dział IT nie wie i za które nikt
formalnie nie odpowiada. Excela czytamy przez pandas:

```python
pd.read_excel("freshbox/reklamacje.xlsx").head(3)
```

```text
  REKLAMACJE 2026 — plik prowadzi Ania K. (NIE RUSZAĆ, NIE KASOWAĆ!!!)  ... Unnamed: 6
0                                                NaN                    ...        NaN
1                                    Data zgłoszenia                    ...    Przyjął
2                                2026-01-04 00:00:00                    ...         PW
```

Nad tabelą jest tytuł i pusty wiersz, więc nagłówek zaczyna się w trzecim
wierszu. Pomijamy dwa pierwsze:

```python
reklamacje = pd.read_excel("freshbox/reklamacje.xlsx", skiprows=2)
reklamacje.head(3)
```

DuckDB potrafi pytać także o DataFrame z pandas — wystarczy podać nazwę
zmiennej w `FROM`:

```python
duckdb.sql("""
SELECT COUNT(*) AS reklamacje, COUNT(DISTINCT Miasto) AS rozne_miasta
FROM reklamacje
""")
```

```text
┌────────────┬──────────────┐
│ reklamacje │ rozne_miasta │
├────────────┼──────────────┤
│       2600 │           28 │
└────────────┴──────────────┘
```

FreshBox działa w 8 miastach, a w kolumnie `Miasto` jest 28 różnych wartości.
Jak zapisano Warszawę?

```python
duckdb.sql("""
SELECT '"' || Miasto || '"' AS miasto, COUNT(*) AS liczba
FROM reklamacje
WHERE Miasto ILIKE 'w%' AND Miasto NOT ILIKE 'wro%'
GROUP BY Miasto
ORDER BY liczba DESC
""")
```

```text
┌─────────────┬────────┐
│   miasto    │ liczba │
├─────────────┼────────┤
│ "Warszawa"  │    157 │
│ "Warszawa " │    137 │
│ "Wwa"       │    136 │
│ "WAW"       │    128 │
│ "warszawa"  │    125 │
└─────────────┴────────┘
```

Pięć wariantów, w tym jeden ze spacją na końcu — niewidoczną, dopóki nie
otoczymy nazwy cudzysłowami. Ręcznie prowadzony plik nie ma słownika ani
walidacji, więc każdy wpisuje po swojemu.

### Ilu właściwie mamy klientów?

Teraz pytanie prezesa — na razie dla całej firmy i bez wchodzenia w szczegóły
definicji:

```python
duckdb.sql("""
SELECT 'CRM — wszystkie konta' AS zrodlo, COUNT(*) AS klienci
FROM 'freshbox/klienci.csv'
UNION ALL
SELECT 'CRM — konta aktywne', COUNT(*)
FROM 'freshbox/klienci.csv' WHERE status_konta = 'aktywne'
UNION ALL
SELECT 'Sprzedaż — zamówili w ciągu roku', COUNT(DISTINCT id_klienta)
FROM 'freshbox/zamowienia.parquet'
UNION ALL
SELECT 'Aplikacja — logowali się w 3 mies.', COUNT(DISTINCT id_klienta)
FROM 'freshbox/zdarzenia_aplikacji.jsonl.gz' WHERE typ = 'login'
""")
```

```text
┌────────────────────────────────────┬─────────┐
│               zrodlo               │ klienci │
├────────────────────────────────────┼─────────┤
│ CRM — wszystkie konta              │   49494 │
│ CRM — konta aktywne                │   42936 │
│ Sprzedaż — zamówili w ciągu roku   │   38174 │
│ Aplikacja — logowali się w 3 mies. │   21522 │
└────────────────────────────────────┴─────────┘
```

Cztery liczby, od 21 do 49 tysięcy. `COUNT(DISTINCT id_klienta)` liczy
**różnych** klientów i pomija puste `id_klienta`.

Różnice nie wynikają tylko z definicji. Sprawdźmy, czy wszyscy klienci
z systemu sprzedażowego są w CRM. `ANTI JOIN` zwraca wiersze z lewej tabeli,
które **nie mają** pary w prawej:

```python
duckdb.sql("""
SELECT COUNT(DISTINCT z.id_klienta) AS klienci_spoza_crm, COUNT(*) AS ich_zamowienia
FROM 'freshbox/zamowienia.parquet' z
ANTI JOIN 'freshbox/klienci.csv' k ON z.id_klienta = k.id_klienta
WHERE z.id_klienta IS NOT NULL
""")
```

```text
┌───────────────────┬────────────────┐
│ klienci_spoza_crm │ ich_zamowienia │
├───────────────────┼────────────────┤
│               369 │           3643 │
└───────────────────┴────────────────┘
```

369 klientów złożyło w tym roku ponad 3,6 tysiąca zamówień, a w CRM ich nie ma.
Dział IT wyjaśnia, że „coś się zgubiło przy migracji CRM w 2024 roku”. Do tego
dochodzą zamówienia bez żadnego klienta:

```python
duckdb.sql("""
SELECT kanal, COUNT(*) AS zamowienia_gosci
FROM 'freshbox/zamowienia.parquet'
WHERE id_klienta IS NULL
GROUP BY kanal
""")
```

To zakupy **gości** — bez zakładania konta, tylko przez stronę www. Czy gość to
klient? Dla działu sprzedaży tak (zapłacił), dla CRM nie (nie ma konta). Słownik
biznesowy musi to rozstrzygnąć.

I ostatnia ciekawostka: kto zamawiał w październiku?

```python
duckdb.sql("""
SELECT k.status_konta, COUNT(DISTINCT z.id_klienta) AS klienci
FROM 'freshbox/zamowienia.parquet' z
JOIN 'freshbox/klienci.csv' k ON z.id_klienta = k.id_klienta
WHERE z.ts_zamowienia >= DATE '2026-10-01'
GROUP BY k.status_konta
ORDER BY klienci DESC
""")
```

```text
┌──────────────┬─────────┐
│ status_konta │ klienci │
├──────────────┼─────────┤
│ aktywne      │   17729 │
│ zablokowane  │     533 │
│ usuniete     │     288 │
└──────────────┴─────────┘
```

Ponad 500 klientów, których CRM uważa za **zablokowanych**, normalnie zamawia.
Okazuje się, że blokada w CRM (np. za nieopłacone zamówienie) nie jest
przekazywana do systemu sprzedażowego. Systemy nie wiedzą o sobie nawzajem —
a to już problem procesu, a nie danych.

---

## Ściągawka

| Chcę… | Kod |
|---|---|
| zapytać o plik CSV / Parquet / JSON | `duckdb.sql("SELECT * FROM 'freshbox/plik.csv'")` |
| zapytać o DataFrame z pandas | `duckdb.sql("SELECT * FROM nazwa_zmiennej")` |
| kolumny i ich typy | `duckdb.sql("DESCRIBE SELECT * FROM 'plik'")` |
| statystyki wszystkich kolumn | `duckdb.sql("SUMMARIZE SELECT * FROM 'plik'")` |
| liczba wierszy i kolumn | `duckdb.sql("SELECT * FROM 'plik'").shape` |
| nazwy kolumn | `duckdb.sql("SELECT * FROM 'plik'").columns` |
| jedna liczba jako zmienna Pythona | `duckdb.sql("SELECT COUNT(*) FROM 'plik'").fetchall()[0][0]` |
| wynik jako DataFrame | `duckdb.sql("...").df()` |
| najmniejsza i największa data | `SELECT MIN(kol), MAX(kol) FROM 'plik'` |
| liczba różnych klientów (bez pustych) | `COUNT(DISTINCT id_klienta)` |
| zakres dat | `WHERE ts >= TIMESTAMP '2026-08-02'` |
| warunek „jedna z wartości” | `WHERE status IN ('a', 'b', 'c')` / `NOT IN (...)` |
| wiersze bez pary w drugiej tabeli | `FROM a ANTI JOIN b ON a.id = b.id` |
| zmienna Pythona w zapytaniu | `duckdb.sql(f"... WHERE miasto = '{MIASTO}'")` |
| rozmiar pliku | `os.path.getsize("freshbox/plik") / 1e6` (MB) |
| Excel z tytułem nad tabelą | `pd.read_excel("plik.xlsx", skiprows=2)` |
| tabela z listy słowników | `pd.DataFrame([{"a": 1, "b": 2}, {"a": 3, "b": 4}])` |

Zbiory na liczbach w Pythonie — przydadzą się w części 2 zadania:

| Chcę… | Kod |
|---|---|
| zbiór `id_klienta` z wyniku zapytania | `zbior = {r[0] for r in duckdb.sql("SELECT DISTINCT id_klienta ...").fetchall()}` |
| elementy w `a`, których nie ma w `b` | `a - b` |
| elementy wspólne | `a & b` |
| liczność zbioru | `len(a)` |

---

## Zadanie 1 — Inwentaryzacja danych FreshBoxa (9 pkt)

**Czas:** ok. 65 minut
**Środowisko:** Google Colab (DuckDB)
**Oddanie:** notatnik `.ipynb` na Moodle (*Plik → Pobierz → Pobierz plik .ipynb*)

Zadanie ma dwie części. Rób je po kolei i **pokaż mi wynik każdej części, gdy
będzie gotowa** — punktacja zależy od tego, ile zrobisz na zajęciach.

### Twoje miasto

Prezes zlecił inwentaryzację całej firmy, a kierownik Twojego miasta chce
wiedzieć, ilu ma klientów. Miasto wybiera numer albumu. Pierwsze komórki
notatnika:

```python
!pip install -q -U duckdb
```

```python
import os
import urllib.request

import duckdb
import pandas as pd

NR_ALBUMU = 123456                 # ← wpisz swój numer albumu

MIASTA = ["Warszawa", "Kraków", "Wrocław", "Poznań", "Gdańsk", "Łódź", "Katowice", "Lublin"]
MIASTO = MIASTA[NR_ALBUMU % 8]
print(f"Moje miasto: {MIASTO}")

URL = "https://kubzal.github.io/wsb-merito/semestr_zimowy_2026_27/zarzadzanie_big_data/dane/"
PLIKI = ["klienci.csv", "zamowienia.parquet", "zdarzenia_aplikacji.jsonl.gz",
         "reklamacje.xlsx", "miasta.csv"]
os.makedirs("freshbox", exist_ok=True)
for plik in PLIKI:
    urllib.request.urlretrieve(URL + plik, f"freshbox/{plik}")

DZIS = "2026-10-31"
OD_SPRZEDAZ = "2026-08-02"         # ostatnie 90 dni
OD_APLIKACJA = "2026-10-02"        # ostatnie 30 dni
```

Rozwiązania z cudzym miastem nie zaliczam.

### Część 1 — spis zasobów (ok. 30 min)

1. Dla **każdego z pięciu plików** ustal: format, liczbę wierszy, liczbę kolumn,
   rozmiar w MB oraz zakres dat (najwcześniejsza i najpóźniejsza data w głównej
   kolumnie z datą). Zbierz wyniki w jednym DataFrame `inwentaryzacja` — jeden
   wiersz na plik.
2. Dodaj do niego trzy kolumny uzupełnione **ręcznie**, na podstawie tego, co
   widzisz w danych: `wlasciciel` (który dział biznesowy powinien odpowiadać za
   zbiór), `dane_osobowe` (jakie kolumny zawierają dane osobowe, albo „brak”),
   `aktualizacja` (jak często zbiór się zmienia — zgadnij rozsądnie).
3. W komórce Markdown wskaż **dwa najbardziej ryzykowne** zbiory i w 2–4
   zdaniach uzasadnij dlaczego.

### Część 2 — ilu klientów ma Twoje miasto (ok. 35 min)

Trzy działy liczą „aktywnych klientów” w swoich systemach:

| Dział | Definicja | Źródło |
|---|---|---|
| Obsługa klienta | konto ze statusem `aktywne`, miasto w CRM = Twoje miasto | `klienci.csv` |
| Sprzedaż | co najmniej jedno **nieanulowane** zamówienie od `OD_SPRZEDAZ` do końca danych, dostarczane do Twojego miasta | `zamowienia.parquet` |
| Produkt | co najmniej jedno zdarzenie `login` od `OD_APLIKACJA` do końca danych, miasto w aplikacji = Twoje miasto | `zdarzenia_aplikacji.jsonl.gz` |

1. Policz liczbę aktywnych klientów w swoim mieście według każdej z trzech
   definicji (licz różne `id_klienta`, pomijając puste).
2. Wyjaśnij liczbami, skąd biorą się różnice. Policz:

    - ilu klientów aktywnych według **sprzedaży** nie jest aktywnych według
      **CRM** — i rozbij ich na grupy: klienta nie ma w CRM, ma w CRM status inny
      niż `aktywne`, jest w CRM aktywny, ale zapisany w innym mieście,
    - ilu klientów aktywnych według **aplikacji** nie jest aktywnych według
      **sprzedaży**.

3. W komórce Markdown zaproponuj **jeden wpis do słownika biznesowego** dla
   pojęcia „aktywny klient” (pojęcie, definicja, źródło, sposób liczenia,
   właściciel, częstotliwość, uwagi — jak w przykładzie w notatkach). Może to
   być jedna z trzech definicji albo Twoja własna. W 2–3 zdaniach uzasadnij,
   dlaczego ta, a nie inne.

### Wskazówki

- Ściągawka jest kilka sekcji wyżej — wszystkie potrzebne polecenia tam są.
- Część 1, punkt 1: dla Excela liczbę wierszy i kolumn weź z DataFrame
  (`reklamacje.shape`), a nie z DuckDB. Główne kolumny z datą to:
  `data_rejestracji`, `ts_zamowienia`, `ts`, `Data zgłoszenia`, `data_startu`.
- Część 1, punkt 1: nie musisz pisać pętli — pięć osobnych zapytań i ręcznie
  zbudowana lista słowników to w zupełności dobre rozwiązanie.
- Część 2, punkt 1: zanim policzysz sprzedaż, przypomnij sobie z demo, ile jest
  wariantów statusu „anulowane”.
- Część 2, punkt 2: najprościej pobrać trzy **zbiory** `id_klienta` do Pythona
  i odejmować je od siebie (ściągawka na dole). Do rozbicia na grupy przyda się
  `klienci.csv` z kolumnami `status_konta` i `miasto`.
- Część 2, punkt 2: sumy w rozbiciu muszą się zgadzać z liczbą klientów, których
  rozbijasz. Sprawdź to w kodzie.

!!! tip "Na co patrzę przy ocenie"

    W części 1 — na kompletność spisu i sensowność ręcznych kolumn (nie ma
    jednej dobrej odpowiedzi, ale „właściciel: IT” to zła odpowiedź). W części
    2 — na zgodność trzech liczb i rozbicia z moimi oraz na to, czy wpis do
    słownika jest na tyle precyzyjny, że dwie osoby policzą z niego tę samą
    liczbę.

### Punktacja (9 pkt)

| Punkty | Za co |
|---|---|
| **9 pkt** | obie części zrobione **na zajęciach** i pokazane prowadzącemu |
| **6 pkt** | jedna z dwóch części zrobiona na zajęciach |
| **3 pkt** | zadanie dokończone po zajęciach i wysłane na Moodle **w terminie** |
| **0 pkt** | brak rozwiązania albo wysłane po terminie |

Termin przesłania na Moodle: **2026-11-21** (dzień przed kolejnymi zajęciami).

!!! danger "Serio, róbcie to na zajęciach"

    Te same 65 minut pracy jest warte 9 pkt tu i teraz albo 3 pkt w domu. Przy
    sześciu zadaniach w semestrze to różnica kilkunastu punktów, czyli całej
    oceny.
