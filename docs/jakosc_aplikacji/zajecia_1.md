# Zajęcia 1

## Czym jest jakość oprogramowania?

Jakość oprogramowania to stopień, w jakim produkt spełnia wymagania użytkowników i jest wolny od defektów. W praktyce oznacza to, że aplikacja:

- działa zgodnie ze specyfikacją,
- jest niezawodna i wydajna,
- jest łatwa w utrzymaniu i rozwijaniu.

Testowanie to jeden z głównych sposobów zapewniania jakości — ale nie jedyny. Na kolejnych zajęciach zajmiemy się też jakością kodu, automatyzacją CI/CD oraz perspektywą użytkownika (UX).

## Testy manualne

### Czym są?

Testy manualne polegają na ręcznym sprawdzaniu działania aplikacji przez testera — bez użycia narzędzi automatyzujących. Tester wciela się w rolę użytkownika i wykonuje scenariusze testowe krok po kroku.

### Kiedy stosujemy testy manualne?

- **Testy eksploracyjne** — swobodne „klikanie" po aplikacji w poszukiwaniu błędów.
- **Testy akceptacyjne** — sprawdzanie, czy aplikacja spełnia wymagania biznesowe.
- **Testy użyteczności (UX)** — ocena, czy interfejs jest intuicyjny.
- **Jednorazowe weryfikacje** — szybkie sprawdzenie poprawki lub nowej funkcji.

### Wady testów manualnych

- Są **czasochłonne** i **powtarzalne**.
- Wyniki zależą od uwagi testera — łatwo coś przeoczyć.
- Nie skalują się — im większa aplikacja, tym więcej pracy.

Dlatego w profesjonalnym wytwarzaniu oprogramowania **automatyzujemy testy** wszędzie, gdzie to możliwe.

## Narzędzia używane w testach manualnych

### Postman — testowanie API

Postman to aplikacja desktopowa (i webowa) do wysyłania zapytań HTTP i analizowania odpowiedzi serwera. Pozwala testować backend **bez konieczności posiadania frontendu**.

Główne możliwości:

- Wysyłanie requestów GET, POST, PUT, DELETE.
- Podgląd odpowiedzi — status code, body (JSON/XML), nagłówki, czas odpowiedzi.
- Tworzenie kolekcji zapytań i organizowanie ich w foldery.
- Ustawianie zmiennych środowiskowych (np. `{{base_url}}`).
- Automatyczne testy w zakładce „Tests" (JavaScript).

> **Instalacja:** [postman.com/downloads](https://www.postman.com/downloads/) — dostępny na Windows, macOS, Linux.
>
> **Alternatywa przeglądarkowa:** Można też używać rozszerzenia **Thunder Client** w VS Code lub narzędzia **curl** w terminalu.

#### curl
`curl` to narzędzie konsolowe do wysyłania zapytań HTTP (i nie tylko) prosto z terminala. W kontekście testowania API działa tak:

**Podstawowe użycie — zapytanie GET:**

```bash
curl https://api.nbp.pl/api/exchangerates/rates/a/usd/
```

To wyświetli odpowiedź JSON bezpośrednio w terminalu.

**Przydatne flagi:**

```bash
# -i  wyświetl nagłówki odpowiedzi (status code, Content-Type itd.)
curl -i https://api.nbp.pl/api/exchangerates/rates/a/usd/

# -v  tryb verbose — pokazuje cały przebieg komunikacji (request + response)
curl -v https://api.nbp.pl/api/exchangerates/rates/a/usd/

# -s  tryb cichy — bez paska postępu
curl -s https://api.nbp.pl/api/exchangerates/rates/a/usd/

# -o plik.json  zapisz odpowiedź do pliku
curl -s https://api.nbp.pl/api/exchangerates/rates/a/usd/ -o kurs_usd.json
```

**Ładne formatowanie JSON** — curl sam z siebie wyrzuca JSON w jednej linii, więc warto przepuścić przez `jq`:

```bash
curl -s https://api.nbp.pl/api/exchangerates/rates/a/usd/ | jq
```

**Inne metody HTTP:**

```bash
# POST z JSON body
curl -X POST https://example.com/api/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Jan", "email": "jan@example.com"}'

# PUT
curl -X PUT https://example.com/api/users/1 \
  -H "Content-Type: application/json" \
  -d '{"name": "Jan Nowak"}'

# DELETE
curl -X DELETE https://example.com/api/users/1
```

W skrócie — curl to taki Postman, tylko bez GUI. Dla szybkich testów typu "czy ten endpoint w ogóle odpowiada" jest nawet wygodniejszy, bo nie trzeba nic otwierać.

### Postman

Polecam ten [tutorial na YouTube](https://www.youtube.com/watch?v=MFxk5BZulVU).

### Chrome DevTools (F12) — testowanie aplikacji webowych

Wbudowane narzędzia deweloperskie przeglądarki Chrome (i innych przeglądarek opartych na Chromium). Kluczowe zakładki dla testera:

- **Elements** — podgląd i edycja HTML/CSS na żywo.
- **Console** — wyświetla błędy JavaScript, pozwala wykonywać komendy.
- **Network** — podgląd wszystkich zapytań HTTP, statusy, czasy odpowiedzi, payload.
- **Application** — cookies, localStorage, sessionStorage.

### Narzędzia do raportowania błędów

Znaleziony bug trzeba **udokumentować** tak, żeby deweloper mógł go odtworzyć. Używa się do tego:

- **JIRA** — najpopularniejsze narzędzie w firmach IT.
- **GitHub Issues / GitLab Issues** — wbudowane w repozytoria kodu.
- **Trello** — proste tablice kanban.
- **Arkusz kalkulacyjny** — na początek wystarczy Google Sheets / Excel.

### Dobry raport z buga zawiera

1. **Tytuł** — krótki i konkretny (np. „Błąd 500 przy pobieraniu kursu USD").
2. **Kroki do reprodukcji** — co trzeba zrobić, żeby bug się pojawił.
3. **Oczekiwany rezultat** — co powinno się stać.
4. **Faktyczny rezultat** — co się faktycznie stało.
5. **Środowisko** — przeglądarka, system, wersja API.
6. **Priorytet / Severity** — jak bardzo to wpływa na działanie systemu.

## Piramida testów

Piramida testów to model, który pokazuje **proporcje** między różnymi rodzajami testów w projekcie.

```
        /\
       /  \
      / E2E \          ← mało, wolne, kosztowne
     /--------\
    /Integracyjne\     ← średnio
   /--------------\
  / Jednostkowe     \  ← dużo, szybkie, tanie
 /___________________\
```

### Testy jednostkowe (unit tests)

- Testują **pojedynczą funkcję lub metodę** w izolacji.
- Są **najszybsze** i **najtańsze** w utrzymaniu.
- Powinny stanowić **fundament** zestawu testów.

### Testy integracyjne (integration tests)

- Sprawdzają **współpracę między modułami** (np. serwis + baza danych).
- Są wolniejsze, bo wymagają uruchomienia zależności.

### Testy E2E (end-to-end)

- Symulują **prawdziwego użytkownika** (np. Selenium, Playwright).
- Testują cały przepływ — od UI przez backend po bazę danych.
- Są **najwolniejsze** i **najdroższe** w utrzymaniu.

### Dlaczego piramida, a nie odwrócony trójkąt?

Projekty, w których większość testów to testy E2E (tzw. „ice cream cone anti-pattern"), cierpią na:

- bardzo długi czas wykonania testów,
- niestabilne testy (flaky tests),
- trudność w lokalizowaniu błędów.

Zasada: **im bliżej kodu, tym więcej testów**.

## Testy jednostkowe w pytest

### Instalacja

```bash
pip install pytest
```

### Struktura projektu

```
projekt/
├── app/
│   ├── __init__.py
│   └── calculator.py
└── tests/
    ├── __init__.py
    └── test_calculator.py
```

### Kod aplikacji — `app/calculator.py`

```python
def add(a: float, b: float) -> float:
    return a + b


def subtract(a: float, b: float) -> float:
    return a - b


def multiply(a: float, b: float) -> float:
    return a * b


def divide(a: float, b: float) -> float:
    if b == 0:
        raise ValueError("Nie można dzielić przez zero!")
    return a / b
```

### Testy — `tests/test_calculator.py`

```python
import pytest
from app.calculator import add, subtract, multiply, divide


# --- Proste testy ---

def test_add():
    assert add(2, 3) == 5


def test_subtract():
    assert subtract(10, 4) == 6


def test_multiply():
    assert multiply(3, 7) == 21


def test_divide():
    assert divide(10, 2) == 5.0


# --- Testowanie wyjątków ---

def test_divide_by_zero():
    with pytest.raises(ValueError, match="Nie można dzielić przez zero"):
        divide(10, 0)


# --- Parametryzacja testów ---

@pytest.mark.parametrize("a, b, expected", [
    (1, 1, 2),
    (-1, 1, 0),
    (0, 0, 0),
    (100, 200, 300),
    (-5, -3, -8),
])
def test_add_parametrized(a, b, expected):
    assert add(a, b) == expected
```

### Uruchamianie testów

```bash
# Wszystkie testy
pytest

# Z szczegółowym outputem
pytest -v

# Konkretny plik
pytest tests/test_calculator.py

# Konkretny test
pytest tests/test_calculator.py::test_add

# Zatrzymaj się na pierwszym błędzie
pytest -x
```

### Przykładowy output

```
$ pytest -v
========================= test session starts =========================
tests/test_calculator.py::test_add PASSED
tests/test_calculator.py::test_subtract PASSED
tests/test_calculator.py::test_multiply PASSED
tests/test_calculator.py::test_divide PASSED
tests/test_calculator.py::test_divide_by_zero PASSED
tests/test_calculator.py::test_add_parametrized[1-1-2] PASSED
tests/test_calculator.py::test_add_parametrized[-1-1-0] PASSED
tests/test_calculator.py::test_add_parametrized[0-0-0] PASSED
tests/test_calculator.py::test_add_parametrized[100-200-300] PASSED
tests/test_calculator.py::test_add_parametrized[-5--3--8] PASSED
========================= 10 passed in 0.03s ==========================
```

---

## Dobre praktyki pisania testów

1. **Nazewnictwo** — nazwa testu powinna mówić, co testuje: `test_divide_by_zero_raises_error`, nie `test_1`.
2. **Arrange–Act–Assert (AAA)** — każdy test ma 3 fazy:
   - *Arrange* — przygotuj dane wejściowe,
   - *Act* — wywołaj testowaną funkcję,
   - *Assert* — sprawdź wynik.
3. **Jeden test = jedna rzecz** — nie sprawdzaj 5 zachowań w jednym teście.
4. **Testy muszą być niezależne** — kolejność uruchamiania nie może wpływać na wynik.
5. **Testuj przypadki brzegowe** — puste listy, zera, wartości ujemne, `None`.

## Fixture w pytest

Fixture to mechanizm przygotowywania danych lub zasobów współdzielonych między testami.

```python
import pytest


@pytest.fixture
def sample_list():
    """Przygotowuje przykładową listę do testów."""
    return [3, 1, 4, 1, 5, 9, 2, 6]


def test_sort(sample_list):
    assert sorted(sample_list) == [1, 1, 2, 3, 4, 5, 6, 9]


def test_length(sample_list):
    assert len(sample_list) == 8


def test_max(sample_list):
    assert max(sample_list) == 9
```

Fixture przekazujemy jako **argument funkcji testowej** — pytest automatycznie je wstrzykuje.

## Zadanie praktyczne (20 pkt)

Za każdą z części do uzyskania jest 10 pkt jeżeli będzie wykonana w trakcie zajęć lub 8 pkt jeżeli będzie dokończona w domu.

### Część 1 — Wykonanie scenariusza testowego: API NBP (10 pkt)

API Narodowego Banku Polskiego udostępnia kursy walut i ceny złota.

- **Dokumentacja:** [https://api.nbp.pl/en.html](https://api.nbp.pl/en.html)
- **Bazowy URL:** `https://api.nbp.pl`
- **Format odpowiedzi:** JSON (domyślny) lub XML (`?format=xml`)
- **Ograniczenie:** pojedyncze zapytanie o zakres dat nie może przekraczać 93 dni

#### Przydatne endpointy

| Endpoint | Opis |
|----------|------|
| `/api/exchangerates/tables/a/` | Aktualna tabela kursów średnich (tabela A) |
| `/api/exchangerates/rates/a/{kod_waluty}/` | Aktualny kurs średni danej waluty |
| `/api/exchangerates/rates/a/{kod_waluty}/last/{n}/` | Ostatnie N notowań danej waluty |
| `/api/exchangerates/rates/a/{kod_waluty}/{data_od}/{data_do}/` | Kursy z zakresu dat (max 93 dni) |
| `/api/cenyzlota/` | Aktualna cena złota (PLN za 1g) |

#### Scenariusz testowy do wykonania

Otwórz Postman (lub Thunder Client / curl) i wykonaj poniższe testy. Dla każdego kroku zapisz: status code, czy odpowiedź jest zgodna z oczekiwaniem, i ewentualne uwagi. 

Do kadego testu dołącz te screenshot z jego wykonania.

Raport zapisz w formacie PDF.

| # | Przypadek testowy | Metoda | URL | Oczekiwany status | Oczekiwany rezultat |
|---|-------------------|--------|-----|-------------------|---------------------|
| 1 | Pobranie aktualnej tabeli kursów A | GET | `https://api.nbp.pl/api/exchangerates/tables/a/` | 200 OK | JSON z listą walut, każda ma pola `currency`, `code`, `mid` |
| 2 | Pobranie kursu USD | GET | `https://api.nbp.pl/api/exchangerates/rates/a/usd/` | 200 OK | JSON z obiektem zawierającym `code: "USD"` i pole `mid` z kursem |
| 3 | Pobranie kursu nieistniejącej waluty | GET | `https://api.nbp.pl/api/exchangerates/rates/a/xyz/` | 404 Not Found | Brak danych — API poprawnie odrzuca nieznany kod waluty |
| 4 | Pobranie ostatnich 10 notowań EUR | GET | `https://api.nbp.pl/api/exchangerates/rates/a/eur/last/10/` | 200 OK | JSON z tablicą `rates` zawierającą dokładnie 10 elementów |
| 5 | Pobranie kursów z przyszłej daty | GET | `https://api.nbp.pl/api/exchangerates/rates/a/usd/2099-01-01/` | 404 Not Found | Brak danych dla przyszłej daty |
| 6 | Zakres dat dłuższy niż 93 dni | GET | `https://api.nbp.pl/api/exchangerates/rates/a/usd/2025-01-01/2025-12-31/` | 400 Bad Request | Błąd — API limituje zakres do 93 dni |
| 7 | Pobranie aktualnej ceny złota | GET | `https://api.nbp.pl/api/cenyzlota/` | 200 OK | JSON z polem `cena` (cena w PLN za 1g złota) |
| 8 | Żądanie w formacie XML | GET | `https://api.nbp.pl/api/exchangerates/rates/a/usd/?format=xml` | 200 OK | Odpowiedź w formacie XML zamiast JSON |
| 9 | Sprawdzenie nagłówka Content-Type | GET | `https://api.nbp.pl/api/exchangerates/rates/a/usd/?format=json` | 200 OK | Nagłówek `Content-Type` zawiera `application/json` |
| 10 | Nieprawidłowa ścieżka endpointu | GET | `https://api.nbp.pl/api/exchangerates/nieistniejacy/` | 404 Not Found | API poprawnie zwraca błąd dla złej ścieżki |

#### Szablon raportu z testów (Część 1)

Wypełnij poniższą tabelę na podstawie wyników:

| # | Status code | Wynik (PASS/FAIL) | Uwagi |
|---|-------------|-------------------|-------|
| 1 | | | |
| 2 | | | |
| 3 | | | |
| 4 | | | |
| 5 | | | |
| 6 | | | |
| 7 | | | |
| 8 | | | |
| 9 | | | |
| 10 | | | |

---

### Część 2 — Stworzenie i wykonanie własnego scenariusza: Open-Meteo API (10 pkt)

Open-Meteo to darmowe API pogodowe — nie wymaga klucza API ani rejestracji.

- **Dokumentacja:** [https://open-meteo.com/en/docs](https://open-meteo.com/en/docs)
- **Bazowy URL:** `https://api.open-meteo.com`
- **Format odpowiedzi:** JSON

#### Przydatne endpointy i parametry

| Parametr | Opis | Przykładowe wartości |
|----------|------|----------------------|
| `latitude` | Szerokość geograficzna | `52.23` (Warszawa), `51.51` (Londyn) |
| `longitude` | Długość geograficzna | `21.01` (Warszawa), `-0.13` (Londyn) |
| `hourly` | Dane godzinowe | `temperature_2m`, `precipitation`, `wind_speed_10m` |
| `daily` | Dane dzienne | `temperature_2m_max`, `temperature_2m_min`, `precipitation_sum` |
| `current` | Dane bieżące | `temperature_2m`, `wind_speed_10m`, `weather_code` |
| `timezone` | Strefa czasowa | `Europe/Warsaw`, `auto` |
| `forecast_days` | Liczba dni prognozy | `1` do `16` |

#### Przykładowe zapytania

```
# Prognoza godzinowa temperatury dla Warszawy
https://api.open-meteo.com/v1/forecast?latitude=52.23&longitude=21.01&hourly=temperature_2m

# Aktualna pogoda + dane dzienne
https://api.open-meteo.com/v1/forecast?latitude=52.23&longitude=21.01&current=temperature_2m,wind_speed_10m&daily=temperature_2m_max,temperature_2m_min&timezone=Europe/Warsaw

# Prognoza na 1 dzień z opadami
https://api.open-meteo.com/v1/forecast?latitude=52.23&longitude=21.01&daily=precipitation_sum&forecast_days=1&timezone=auto
```

#### Zadanie

1. **Napisz własny scenariusz testowy** składający się z **minimum 8 przypadków testowych** dla API Open-Meteo. Użyj tabeli analogicznej do scenariusza NBP (kolumny: #, Przypadek testowy, Metoda, URL, Oczekiwany status, Oczekiwany rezultat).

   Scenariusz musi zawierać:
   - Minimum **5 testów pozytywnych** (poprawne zapytania, różne parametry).
   - Minimum **3 testy negatywne** (błędne dane, brakujące parametry, nieprawidłowe wartości).

   Inspiracje na przypadki testowe:

   - Pobranie prognozy godzinowej temperatury dla Warszawy.
   - Pobranie prognozy dla innego miasta (np. Londyn, Nowy Jork).
   - Zapytanie bez wymaganych parametrów (`latitude` lub `longitude`).
   - Zapytanie z nieprawidłowymi współrzędnymi (np. `latitude=999`).
   - Pobranie danych dziennych (max/min temperatura, suma opadów).
   - Sprawdzenie, czy `current` zwraca aktualną temperaturę i wiatr.
   - Zmiana `timezone` i sprawdzenie wpływu na format dat w odpowiedzi.
   - Porównanie `forecast_days=1` vs `forecast_days=16` (ilość danych).
   - Zapytanie z wieloma zmiennymi `hourly` jednocześnie.
   - Zapytanie z pustym parametrem `hourly=`.

2. **Wykonaj scenariusz** w Postmanie i wypełnij raport z wynikami (analogiczny do szablonu z Części 1).

3. **Oddaj raport** zawierający:
   - Tabelę scenariusza testowego (co testujesz i jakie URL).
   - Tabelę z wynikami testów (status code, PASS/FAIL, uwagi).
   - **Podsumowanie** — ile testów PASS, ile FAIL, jakie wnioski.

---

## Podsumowanie zajęć 1

| Temat | Kluczowy wniosek |
|-------|-------------------|
| Testy manualne | Przydatne, ale nie skalują się — automatyzuj co się da |
| Narzędzia | Postman do API, DevTools do UI, JIRA/GitHub do raportowania bugów |
| Piramida testów | Dużo unit testów, mniej integracyjnych, najmniej E2E |
| pytest | Prosty framework — `assert` + funkcje `test_*` wystarczą na start |
| Parametryzacja | `@pytest.mark.parametrize` pozwala testować wiele przypadków jednym testem |
| Fixture | `@pytest.fixture` do współdzielenia danych między testami |
