# Zajęcia 2

## Powtórka z zajęć 1

Na poprzednich zajęciach poznaliśmy testy manualne, piramidę testów oraz pisanie testów jednostkowych w `pytest`. Wiemy już, że:

- testy automatyczne są szybsze i bardziej powtarzalne niż manualne,
- najwięcej powinno być testów jednostkowych (podstawa piramidy),
- dobry test ma strukturę **Arrange–Act–Assert**.

Dziś idziemy o krok dalej — zajmiemy się **jakością samego kodu** (zanim w ogóle dojdzie do testowania) oraz **automatyzacją**, czyli sprawianiem, żeby maszyny robiły za nas nudną pracę.

---

## Część 1 — Jakość kodu

### Dlaczego jakość kodu ma znaczenie?

Kod pisze się raz, a czyta się go **dziesiątki razy**. Kod niskiej jakości to:

- więcej czasu na wprowadzanie zmian,
- więcej bugów,
- frustracja w zespole,
- wyższy koszt utrzymania projektu (technical debt).

Jakość kodu zaczyna się od **spójnej konwencji** i **automatycznej weryfikacji**. Nikt nie powinien w 2026 roku spierać się na code review o liczbę spacji albo kolejność importów — od tego są narzędzia.

### PEP 8 — standard stylu Pythona

[PEP 8](https://peps.python.org/pep-0008/) to oficjalny przewodnik stylu dla Pythona. Najważniejsze zasady:

- **wcięcia:** 4 spacje (nigdy taby),
- **długość linii:** max 79 znaków (w praktyce zespoły używają 88-100),
- **nazewnictwo:**
  - `snake_case` dla funkcji i zmiennych: `calculate_total`, `user_name`,
  - `PascalCase` dla klas: `UserAccount`, `OrderManager`,
  - `UPPER_CASE` dla stałych: `MAX_RETRIES`, `API_URL`,
- **importy:** każdy w osobnej linii, na górze pliku, pogrupowane (standard / third-party / local),
- **dwie puste linie** między definicjami funkcji top-level, **jedna** między metodami w klasie.

### Linter — `flake8` / `ruff`

**Linter** to narzędzie, które analizuje kod statycznie (bez uruchamiania) i wskazuje:

- naruszenia stylu (PEP 8),
- niewykorzystane importy/zmienne,
- potencjalne bugi (np. porównanie z `None` przez `==` zamiast `is`).

#### `ruff` — nowoczesny linter (rekomendowany)

`ruff` jest napisany w Ruście, więc działa **niesamowicie szybko** i zastępuje kilka starszych narzędzi (flake8, isort, pyupgrade) jednym.

```bash
pip install ruff

# Sprawdzenie kodu
ruff check .

# Automatyczna naprawa tego, co da się naprawić
ruff check . --fix
```

Przykładowy output:

```
app/calculator.py:5:1: F401 [*] `os` imported but unused
app/calculator.py:12:80: E501 Line too long (95 > 79)
app/calculator.py:18:1: E302 Expected 2 blank lines, found 1
Found 3 errors.
[*] 1 fixable with the `--fix` option.
```

### Formatter — `ruff format`

**Formatter** to narzędzie, które **automatycznie przepisuje** kod do spójnego stylu. W przeciwieństwie do lintera nie tylko wskazuje błędy — sam je naprawia.

Dobra wiadomość: `ruff` to nie tylko linter — zawiera też wbudowany formatter, który jest zaprojektowany jako **drop-in replacement dla `black`** (ponad 99.9% kompatybilności na kodzie formatowanym wcześniej Blackiem). Dzięki temu nie potrzebujemy dwóch oddzielnych narzędzi — jedno `ruff` załatwia i lintowanie, i formatowanie.

> **Kontekst historyczny:** Przez lata standardem był `black` — tzw. *opinionated formatter* z filozofią „nie kłóćmy się o styl, niech narzędzie zdecyduje". `ruff format` przejął jego styl i dodał ogromne zwiększenie wydajności (Rust zamiast Pythona). W nowych projektach nie ma już powodu, żeby instalować Blacka osobno.

```bash
# Sformatuj wszystkie pliki .py w bieżącym katalogu
ruff format .

# Tylko sprawdź, nie zmieniaj (przydatne w CI)
ruff format --check .

# Pokaż, co by zostało zmienione
ruff format --diff .
```

#### Przykład działania `ruff format`

**Przed:**

```python
def add(a,b):
    return a+b

result=add( 1,2 )
```

**Po `ruff format .`:**

```python
def add(a, b):
    return a + b


result = add(1, 2)
```

#### Typowy workflow z ruff

```bash
# 1. Naprawa błędów stylu i prostych bugów
ruff check . --fix

# 2. Sformatowanie kodu
ruff format .
```

To dwa polecenia zamiast trzech (kiedyś: `flake8` + `isort` + `black`).

### Type hinting i `mypy`

Python jest językiem dynamicznie typowanym, ale od wersji 3.5 wspiera **adnotacje typów** (type hints). To jak instrukcja obsługi dla innych programistów (i dla nas samych za pół roku).

```python
def add(a: int, b: int) -> int:
    return a + b


def get_user_name(user_id: int) -> str | None:
    # ...
    return None
```

`mypy` to narzędzie, które **statycznie sprawdza zgodność typów**:

```bash
pip install mypy
mypy app/
```

Jeśli ktoś wywoła `add("foo", "bar")` — `mypy` wyłapie to **przed** uruchomieniem programu.

### Code review

Nawet najlepsze narzędzia nie zastąpią **drugiej pary oczu**. Code review to proces, w którym inny programista czyta nasze zmiany przed mergem do głównej gałęzi.

Dobre code review:

- skupia się na **logice** i **architekturze**, a nie na stylu (od tego jest linter),
- jest **konkretne** — „ten warunek może rzucić `KeyError` jeśli klucz nie istnieje" zamiast „nie podoba mi się to",
- jest **uprzejme** — opinia o kodzie, nie o autorze.

---

## Część 2 — Automatyzacja

### Pre-commit hooks

Pre-commit hook to skrypt, który uruchamia się **automatycznie przed każdym commitem**. Jeśli skrypt zwróci błąd — commit zostaje zablokowany.

Idea: nie wpuszczamy do repo kodu, który nie przechodzi lintera/formattera/testów.

#### Instalacja

```bash
pip install pre-commit
```

#### Konfiguracja — plik `.pre-commit-config.yaml`

```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.7.0
    hooks:
      # Linter — wyłapuje błędy i naprawia co się da
      - id: ruff
        args: [--fix]
      # Formatter — zastępuje blacka
      - id: ruff-format

  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v5.0.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
```

#### Aktywacja

```bash
pre-commit install
```

Od tej chwili każdy `git commit` najpierw uruchomi te narzędzia. Jeśli któreś coś zmieni lub zgłosi błąd — commit się nie wykona.

### CI/CD — Continuous Integration / Continuous Deployment

**CI/CD** to praktyka, w której każda zmiana w kodzie jest automatycznie:

1. **Budowana** (build),
2. **Testowana** (testy jednostkowe, integracyjne),
3. **Sprawdzana** (linter, formatter, type checker),
4. (opcjonalnie) **Wdrażana** (deployment).

Cel: wykryć problemy **jak najwcześniej** i nie pozwolić, żeby zepsuty kod trafił na produkcję.

#### Popularne platformy CI/CD

- **GitHub Actions** — wbudowane w GitHub, darmowe dla publicznych repo,
- **GitLab CI/CD** — wbudowane w GitLab,
- **Jenkins** — klasyk, self-hosted, bardzo elastyczny,
- **CircleCI**, **Travis CI** — chmurowe alternatywy.

### GitHub Actions — praktyczny przykład

GitHub Actions konfigurujemy w pliku `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest ruff mypy

      - name: Run linter
        run: ruff check .

      - name: Check formatting
        run: ruff format --check .

      - name: Run type checker
        run: mypy app/

      - name: Run tests
        run: pytest -v
```

#### Co się dzieje?

Po każdym `git push` lub otwarciu PR-a:

1. GitHub uruchamia świeżą maszynę z Ubuntu,
2. Pobiera kod,
3. Instaluje Pythona i zależności,
4. Uruchamia lintera, formatter, type checker i testy,
5. Pokazuje zielony ✅ lub czerwony ❌ przy commicie.

Jeśli którykolwiek krok zawiedzie — cały job jest oznaczony jako failed i nie można zmergować PR-a (jeśli ustawiono *branch protection rules*).

### Coverage — pokrycie kodu testami

**Coverage** to procent linii kodu, które są wykonywane podczas testów. Mierzymy go narzędziem `pytest-cov`:

```bash
pip install pytest-cov

# Raport w konsoli
pytest --cov=app

# Raport HTML (otwórz htmlcov/index.html)
pytest --cov=app --cov-report=html
```

Przykładowy output:

```
---------- coverage: platform linux, python 3.12 ----------
Name                  Stmts   Miss  Cover
-----------------------------------------
app/__init__.py           0      0   100%
app/calculator.py        12      2    83%
-----------------------------------------
TOTAL                    12      2    83%
```

#### Uwaga o coverage

100% pokrycia **nie oznacza, że nie ma bugów**. Można mieć 100% coverage i bezużyteczne testy (np. bez asercji). Coverage to wskaźnik **pomocniczy**, nie cel sam w sobie. Sensowny próg dla większości projektów to 70–85%.

---

## Podsumowanie najważniejszych narzędzi

| Narzędzie | Co robi | Kiedy uruchomić |
|-----------|---------|-----------------|
| `ruff check` | Wyłapuje błędy stylu i potencjalne bugi | Pre-commit, CI |
| `ruff format` | Formatuje kod do jednolitego stylu (zastępuje `black`) | Pre-commit, lokalnie przed commitem |
| `mypy` | Sprawdza zgodność typów | Pre-commit, CI |
| `pytest` | Uruchamia testy jednostkowe | Lokalnie + CI |
| `pytest-cov` | Mierzy pokrycie kodu testami | CI (z progiem minimum) |
| `pre-commit` | Wymusza powyższe przed każdym commitem | Lokalnie (po `pre-commit install`) |
| GitHub Actions | Uruchamia całość w chmurze przy każdym push/PR | Automatycznie po push |

---

## Zadanie praktyczne (20 pkt)

Za każdą z części do uzyskania jest 10 pkt jeżeli będzie wykonana w trakcie zajęć lub 8 pkt jeżeli będzie dokończona w domu.

### Część 1 — Testy jednostkowe modułu zarządzania kontami (10 pkt)

Twoim zadaniem jest napisanie modułu `BankAccount` reprezentującego konto bankowe oraz **kompletu testów jednostkowych** sprawdzających jego działanie.

#### Krok 1 — Utwórz strukturę projektu

```
zadanie_2/
├── app/
│   ├── __init__.py
│   └── bank_account.py
└── tests/
    ├── __init__.py
    └── test_bank_account.py
```

#### Krok 2 — Zaimplementuj klasę `BankAccount`

W pliku `app/bank_account.py` utwórz klasę zgodnie z poniższą specyfikacją:

```python
class BankAccount:
    """Reprezentuje proste konto bankowe."""

    def __init__(self, owner: str, balance: float = 0.0) -> None:
        # Zapisz właściciela i saldo początkowe.
        # Saldo początkowe nie może być ujemne — w przeciwnym razie
        # rzuć ValueError z komunikatem "Saldo początkowe nie może być ujemne".
        ...

    def deposit(self, amount: float) -> None:
        # Wpłaca pieniądze na konto.
        # Kwota musi być dodatnia — w przeciwnym razie
        # rzuć ValueError z komunikatem "Kwota wpłaty musi być dodatnia".
        ...

    def withdraw(self, amount: float) -> None:
        # Wypłaca pieniądze z konta.
        # Kwota musi być dodatnia — ValueError "Kwota wypłaty musi być dodatnia".
        # Kwota nie może przekraczać salda — ValueError "Brak wystarczających środków".
        ...

    def transfer(self, other: "BankAccount", amount: float) -> None:
        # Przelewa kwotę z self na konto other.
        # Wykorzystaj withdraw() i deposit().
        ...

    def __repr__(self) -> str:
        return f"BankAccount(owner={self.owner!r}, balance={self.balance})"
```

#### Krok 3 — Napisz testy w `tests/test_bank_account.py`

Twoje testy muszą pokrywać poniższe przypadki. **Każdy przypadek = osobna funkcja testowa.**

| # | Test | Co sprawdza |
|---|------|-------------|
| 1 | `test_create_account_with_default_balance` | Konto utworzone bez podania salda ma saldo 0.0 |
| 2 | `test_create_account_with_initial_balance` | Konto utworzone z saldem 100 ma saldo 100 |
| 3 | `test_create_account_with_negative_balance_raises` | Tworzenie konta z saldem -10 rzuca `ValueError` |
| 4 | `test_deposit_increases_balance` | Po wpłacie 50 na konto z saldem 100, saldo wynosi 150 |
| 5 | `test_deposit_negative_amount_raises` | Wpłata -10 rzuca `ValueError` |
| 6 | `test_deposit_zero_raises` | Wpłata 0 rzuca `ValueError` |
| 7 | `test_withdraw_decreases_balance` | Po wypłacie 30 z konta z saldem 100, saldo wynosi 70 |
| 8 | `test_withdraw_more_than_balance_raises` | Wypłata 200 z konta z saldem 100 rzuca `ValueError` |
| 9 | `test_withdraw_negative_amount_raises` | Wypłata -10 rzuca `ValueError` |
| 10 | `test_transfer_moves_money_between_accounts` | Po przelewie 50 z konta A (100) na konto B (0): A=50, B=50 |

**Wskazówki:**

- Wykorzystaj `pytest.raises(ValueError, match="..."):` do testowania wyjątków.
- Dodaj **przynajmniej jeden test parametryzowany** za pomocą `@pytest.mark.parametrize` (np. dla różnych kwot wpłat).
- Dodaj **przynajmniej jedną fixture** (np. `@pytest.fixture` zwracającą gotowe konto z saldem 100, używane w kilku testach).
- Każdy test zgodny z zasadą **AAA (Arrange-Act-Assert)**.

#### Krok 4 — Uruchom testy i sprawdź coverage

```bash
pip install pytest pytest-cov
pytest -v --cov=app --cov-report=term-missing
```

Cel: **100% coverage** dla pliku `bank_account.py`. Jeśli coverage jest niższe — `--cov-report=term-missing` pokaże, które linie nie są przetestowane. Dopisz brakujące testy.

#### Co oddajesz?

Spakowany folder `zadanie_2.zip` zawierający:

- pliki `bank_account.py` i `test_bank_account.py`,
- screenshot z wynikiem `pytest -v` (wszystkie testy zielone),
- screenshot z raportem coverage.

---

### Część 2 — Konfiguracja jakości kodu i CI (10 pkt)

Do projektu z Części 1 dodaj **automatyzację jakości kodu**.

#### Krok 1 — Skonfiguruj `ruff` (linter + formatter)

1. Zainstaluj: `pip install ruff`.
2. Uruchom `ruff check . --fix` — naprawi błędy stylu i wyłapie potencjalne bugi.
3. Uruchom `ruff format .` — sformatuje kod do spójnego stylu.
4. Upewnij się, że `ruff check .` oraz `ruff format --check .` kończą się bez błędów.

#### Krok 2 — Dodaj type hints i sprawdź `mypy`

1. Upewnij się, że **wszystkie metody** klasy `BankAccount` mają adnotacje typów (parametry + zwracana wartość).
2. Zainstaluj `mypy`: `pip install mypy`.
3. Uruchom `mypy app/` — output musi być `Success: no issues found`.

#### Krok 3 — Skonfiguruj pre-commit

1. Zainstaluj: `pip install pre-commit`.
2. Utwórz plik `.pre-commit-config.yaml` (skopiuj z notatki).
3. Uruchom `pre-commit install`.
4. Wykonaj testowy commit i pokaż, że hooki się uruchomiły.

#### Krok 4 — Utwórz workflow GitHub Actions

1. Wrzuć projekt na GitHub (możesz utworzyć repozytorium prywatne).
2. Dodaj plik `.github/workflows/ci.yml` (możesz oprzeć się na przykładzie z notatki).
3. Workflow musi uruchamiać przynajmniej:
   - `ruff check .`
   - `ruff format --check .`
   - `pytest -v`
4. Wykonaj `git push` i sprawdź, że workflow się uruchomił i zakończył sukcesem (zielony ✅).

#### Co oddajesz?

- **Link do publicznego repozytorium GitHub** (lub prywatnego z dodanym dostępem dla prowadzącego),
- screenshot z zakładki *Actions* w GitHubie pokazujący zielony build,
- screenshot z lokalnego uruchomienia `pre-commit run --all-files` zakończonego sukcesem.

---

## Podsumowanie zajęć 2

| Temat | Kluczowy wniosek |
|-------|-------------------|
| PEP 8 | Spójna konwencja stylu — fundament czytelności |
| Linter (`ruff check`) | Wyłapuje błędy zanim uruchomisz kod |
| Formatter (`ruff format`) | Niech narzędzie decyduje o stylu, nie ludzie — zastępuje Blacka |
| Type hints + `mypy` | Statyczne typowanie wyłapuje całe klasy bugów |
| Pre-commit | Brudny kod nie ma prawa trafić do repo |
| CI/CD | Każda zmiana automatycznie testowana i sprawdzana |
| Coverage | Mierzy, ile kodu pokrywają testy — ale nie ich jakość |