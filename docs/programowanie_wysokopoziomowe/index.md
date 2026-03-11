# Wprowadzenie

## Plan zajęć

- **Zajęcia 1 (_2026-03-22_):** Wprowadzenie, środowisko pracy, paradygmaty programowania, programowanie strukturalne
- **Zajęcia 2 (_2026-04-26_):** Programowanie obiektowe
- **Zajęcia 3 (_2026-05-17_):** Programowanie obiektowe c.d., programowanie funkcyjne
- **Zajęcia 4 (_2026-06-28_):** Programowanie funkcyjne c.d., oddanie projektów i zadań, quiz

## Zasady zaliczenia

- Quiz zaliczeniowy na platformie Moodle na ostatnich zajęciach, obejmujący treści omawiane na ćwiczeniach — maks. **30 pkt**.
- Praca projektowa — aplikacja biblioteczna wykonywana etapami na kolejnych zajęciach (3 części po 20 pkt) — maks. **60 pkt**.
- Zadanie praktyczne (nieobowiązkowe) — samodzielne zaprojektowanie własnej aplikacji w języku obiektowym, umożliwiające zdobycie dodatkowych punktów w celu podniesienia oceny — maks. **10 pkt**.

Suma punktów: **100**.

- Zarówno praca projektowa, jak i zadanie praktyczne muszą być opublikowane na GitHubie — na Moodle należy przesłać wyłącznie link do pull requesta.
- Mogę zapytać o dowolny fragment kodu i jego znaczenie — na bieżących zajęciach (jeśli zadanie ukończone na miejscu) lub na kolejnych (jeśli dokończone w domu). Nieumiejętność wyjaśnienia własnego kodu może skutkować obniżeniem punktacji.

**Skala ocen:**

| Punkty | Ocena |
|||
| 91–100% | 5,0 |
| 81–90% | 4,5 |
| 71–80% | 4,0 |
| 61–70% | 3,5 |
| 51–60% | 3,0 |
| ≤ 50% | 2,0 |

## Środowisko programistyczne

### Edytor / IDE

Polecam korzystać z **VS Code** lub **PyCharm Community Edition** — oba są darmowe i świetnie wspierają Pythona. VS Code wymaga doinstalowania rozszerzenia „Python" od Microsoftu. PyCharm działa out-of-the-box.

### Wersja Pythona

Zalecam **Python 3.12**. Upewnij się, że masz go zainstalowanego — w terminalu wpisz `python --version` (na Macu może być `python3 --version`).



## uv — menedżer środowisk i zależności

`uv` to nowoczesne narzędzie, które zastępuje `pip`, `venv` i `pip-tools` jednym szybkim poleceniem. Projekt powinien posiadać własne, wyseparowane środowisko wirtualne, plik `pyproject.toml` (definicja projektu i zależności) oraz `uv.lock` (zablokowane wersje paczek).

### Instalacja uv

**Windows (PowerShell):**
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

**macOS / Linux:**
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Po instalacji zrestartuj terminal i sprawdź: `uv --version`.

### Tworzenie nowego projektu

```bash
uv init moja-biblioteka
cd moja-biblioteka
```

To polecenie stworzy folder z gotowym `pyproject.toml` i plikiem `hello.py`. Plik `pyproject.toml` to serce projektu — znajdziesz w nim nazwę, wersję Pythona i listę zależności.

### Przypinanie wersji Pythona

```bash
uv python pin 3.12
```

To utworzy plik `.python-version`, dzięki któremu `uv` zawsze użyje właściwej wersji.

### Tworzenie i aktywacja środowiska wirtualnego

```bash
uv venv
```

Aktywacja:

**Windows (PowerShell):**
```powershell
.venv\Scripts\activate
```

**Windows (cmd):**
```cmd
.venv\Scripts\activate.bat
```

**macOS / Linux:**
```bash
source .venv/bin/activate
```

Po aktywacji zobaczysz `(.venv)` na początku wiersza poleceń.

### Dodawanie zależności

```bash
uv add requests
```

To polecenie doda paczkę do `pyproject.toml`, zainstaluje ją w środowisku i zaktualizuje plik `uv.lock`. Nie edytuj `uv.lock` ręcznie — jest generowany automatycznie.

### Uruchamianie skryptu

```bash
uv run main.py
```

Polecenie `uv run` automatycznie korzysta z właściwego środowiska, nawet bez ręcznej aktywacji.

### Odtwarzanie środowiska (np. po sklonowaniu repo)

```bash
uv sync
```

To zainstaluje dokładnie te wersje paczek, które są zapisane w `uv.lock`.

## Podstawy Pythona

### Zmienne i typy danych

W Pythonie nie deklarujesz typu — interpreter sam go rozpoznaje na podstawie przypisanej wartości.

```python
imie = "Anna"              # str — tekst
wiek = 23                  # int — liczba całkowita
srednia = 4.75             # float — liczba zmiennoprzecinkowa
czy_student = True         # bool — wartość logiczna (True / False)
```

Typ zmiennej możesz sprawdzić za pomocą `type()`:

```python
print(type(imie))      # <class 'str'>
print(type(wiek))      # <class 'int'>
```

Przydatne operacje na stringach:

```python
powitanie = "Cześć, " + imie + "!"           # konkatenacja
powitanie = f"Cześć, {imie}! Masz {wiek} lat."  # f-string (zalecane)
print(len(imie))                               # długość stringa → 4
```

### Pobieranie wartości od użytkownika (input)

Funkcja `input()` zawsze zwraca **string** — jeśli potrzebujesz liczby, musisz jawnie skonwertować.

```python
imie = input("Podaj imię: ")
wiek = int(input("Podaj wiek: "))         # konwersja na int
wzrost = float(input("Podaj wzrost: "))   # konwersja na float
```

Uwaga na błędy — jeśli użytkownik wpisze tekst zamiast liczby, `int()` rzuci wyjątek. Na razie nie musisz tego obsługiwać, ale warto o tym pamiętać.



### Instrukcje warunkowe (if / elif / else)

```python
wiek = int(input("Podaj wiek: "))

if wiek < 18:
    print("Jesteś niepełnoletni.")
elif wiek < 65:
    print("Jesteś dorosły.")
else:
    print("Jesteś na emeryturze.")
```

Bloki kodu wyznaczamy **wcięciami** (4 spacje) — nie ma klamer `{}` jak w C# czy Javie.

Operatory porównania: `==`, `!=`, `<`, `>`, `<=`, `>=`.
Operatory logiczne: `and`, `or`, `not`.

```python
wiek = 20
ma_legitymacje = True

if wiek >= 18 and ma_legitymacje:
    print("Możesz wejść.")
```

### Pętle

**Pętla `for`** — iteracja po zakresie lub kolekcji:

```python
# Zakres liczb: 0, 1, 2, 3, 4
for i in range(5):
    print(i)

# Iteracja po liście
owoce = ["jabłko", "banan", "gruszka"]
for owoc in owoce:
    print(owoc)
```

**Pętla `while`** — powtarza się dopóki warunek jest spełniony:

```python
licznik = 0
while licznik < 3:
    print(f"Próba nr {licznik + 1}")
    licznik += 1
```

Typowy wzorzec — pętla menu:

```python
while True:
    wybor = input("Wybierz opcję (1-3) lub 'q' aby wyjść: ")
    if wybor == "q":
        break           # przerywa pętlę
    elif wybor == "1":
        print("Wybrałeś opcję 1")
    elif wybor == "2":
        print("Wybrałeś opcję 2")
    else:
        print("Nieznana opcja, spróbuj ponownie.")
        continue        # wraca na początek pętli
```



### Listy

Lista to uporządkowana kolekcja elementów. Elementy mogą być różnych typów, ale w praktyce trzymamy elementy jednego typu.

```python
oceny = [5, 4, 3, 5, 4]

print(oceny[0])         # pierwszy element → 5
print(oceny[-1])        # ostatni element → 4
print(len(oceny))       # liczba elementów → 5

oceny.append(2)         # dodanie na koniec
oceny.remove(3)         # usunięcie pierwszego wystąpienia wartości 3
```

Iteracja po liście z indeksem:

```python
for i, ocena in enumerate(oceny):
    print(f"Pozycja {i}: ocena {ocena}")
```

Sprawdzanie czy element istnieje:

```python
if 5 in oceny:
    print("Jest piątka!")
```

### Słowniki

Słownik przechowuje pary **klucz: wartość**. Klucze muszą być unikalne.

```python
student = {
    "imie": "Anna",
    "wiek": 23,
    "kierunek": "Informatyka"
}

print(student["imie"])              # dostęp po kluczu → "Anna"
student["rok"] = 2                  # dodanie nowej pary
student["wiek"] = 24                # nadpisanie wartości
```

Bezpieczny dostęp — `get()` zwraca `None` (lub podaną domyślną wartość) zamiast rzucać błąd:

```python
print(student.get("email"))                 # None (klucz nie istnieje)
print(student.get("email", "brak"))         # "brak"
```

Iteracja po słowniku:

```python
for klucz, wartosc in student.items():
    print(f"{klucz}: {wartosc}")
```

Lista słowników — bardzo przydatny wzorzec, którego użyjecie w projekcie:

```python
ksiazki = [
    {"tytul": "Pan Tadeusz", "autor": "Mickiewicz", "sztuk": 3},
    {"tytul": "Lalka", "autor": "Prus", "sztuk": 1},
    {"tytul": "Ferdydurke", "autor": "Gombrowicz", "sztuk": 0},
]

for ksiazka in ksiazki:
    print(f"{ksiazka['tytul']} — {ksiazka['autor']} (dostępne: {ksiazka['sztuk']})")
```

### Funkcje

Funkcja to wydzielony, nazwany blok kodu, który można wielokrotnie wywoływać.

```python
def powitaj(imie):
    print(f"Cześć, {imie}!")

powitaj("Anna")       # Cześć, Anna!
powitaj("Tomek")      # Cześć, Tomek!
```

Funkcja z wartością zwracaną:

```python
def dodaj(a, b):
    return a + b

wynik = dodaj(3, 5)
print(wynik)           # 8
```

Parametry domyślne:

```python
def powitaj(imie, godzina=12):
    if godzina < 12:
        print(f"Dzień dobry, {imie}!")
    else:
        print(f"Dobry wieczór, {imie}!")

powitaj("Anna")            # użyje godzina=12
powitaj("Anna", godzina=9) # użyje godzina=9
```

Funkcja może zwracać kilka wartości jako krotkę:

```python
def statystyki(oceny):
    return min(oceny), max(oceny), sum(oceny) / len(oceny)

najnizsza, najwyzsza, srednia = statystyki([3, 5, 4, 2, 5])
print(f"Min: {najnizsza}, Max: {najwyzsza}, Średnia: {srednia:.2f}")
```

### Łączymy wszystko — mini przykład

Prosty program, który pokazuje większość powyższych elementów razem:

```python
def wyswietl_studentow(studenci):
    for i, s in enumerate(studenci, start=1):
        print(f"  {i}. {s['imie']} (wiek: {s['wiek']})")

def dodaj_studenta(studenci):
    imie = input("Imię: ")
    wiek = int(input("Wiek: "))
    studenci.append({"imie": imie, "wiek": wiek})
    print(f"Dodano {imie}.")

def main():
    studenci = [
        {"imie": "Anna", "wiek": 23},
        {"imie": "Tomek", "wiek": 21},
    ]

    while True:
        print("\n MENU ")
        print("1. Wyświetl studentów")
        print("2. Dodaj studenta")
        print("q. Wyjdź")

        wybor = input("Twój wybór: ")

        if wybor == "1":
            wyswietl_studentow(studenci)
        elif wybor == "2":
            dodaj_studenta(studenci)
        elif wybor == "q":
            print("Do widzenia!")
            break
        else:
            print("Nieznana opcja.")

main()
```

Ten wzorzec — słowniki jako dane, funkcje jako operacje, pętla `while True` jako menu — jest dokładnie tym, czego użyjecie w Części 1 projektu.

## Git

### Konfiguracja (jednorazowo)

```bash
git config --global user.name "Imię Nazwisko"
git config --global user.email "twoj@email.com"
```

### Podstawowy workflow

```bash
git init                        # inicjalizacja repo (tylko raz)
git add .                       # dodanie wszystkich zmian do staging area
git commit -m "opis zmian"      # zapisanie snapshotu
```

### Praca ze zdalnym repozytorium (GitHub)

```bash
git remote add origin https://github.com/user/repo.git   # powiązanie z GitHubem (raz)
git push -u origin main                                    # wypchnięcie na serwer
git pull                                                   # pobranie zmian z serwera
```

### Branche i Pull Requesty

Na zajęciach oddajecie zadania jako **Pull Request**, więc workflow wygląda tak:

```bash
git checkout -b czesc-1          # nowy branch na część zadania
# ... piszesz kod, commitujesz ...
git add .
git commit -m "Część 1 - programowanie strukturalne"
git push -u origin czesc-1      # wypchnięcie brancha na GitHub
```

Następnie na GitHubie tworzysz **Pull Request** z brancha `czesc-1` do `main` i link do tego PR-a wklejasz na Moodle.

### .gitignore

Stwórz plik `.gitignore` w głównym folderze projektu, żeby nie commitować śmieci:

```
.venv/
__pycache__/
*.pyc
.idea/
.vscode/
*.db
```

Plik `*.db` jest na liście, bo w Części 2 pojawi się baza SQLite, która nie powinna trafiać do repozytorium (każdy generuje ją lokalnie przy uruchomieniu).

### Najczęstsze polecenia — ściągawka

| Polecenie | Co robi |
|||
| `git status` | Pokazuje co się zmieniło |
| `git log --oneline` | Historia commitów w skrócie |
| `git diff` | Podgląd zmian przed commitem |
| `git checkout main` | Przełączenie na branch main |
| `git merge czesc-1` | Wciągnięcie brancha do main |