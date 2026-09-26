# Zajęcia 3

## Programowanie obiektowe c.d.

Na poprzednich zajęciach poznaliśmy podstawy OOP — klasy, obiekty, cztery filary (hermetyzacja, dziedziczenie, polimorfizm, abstrakcja) oraz najczęściej używane dunders. Dziś zbieramy resztę — rzeczy, które w realnym kodzie produkcyjnym widzisz na co dzień, a które niekoniecznie pojawiają się w pierwszym kontakcie z klasami.

Po dzisiejszej części OOP będziesz w stanie:

- pisać klasy "danych" bez powtarzalnego boilerplate'u (`dataclass`),
- walidować dane wchodzące do klasy przy pomocy Pydantica,
- rozróżniać metody zwykłe, klasowe i statyczne i wiedzieć kiedy której użyć,
- wybierać między dziedziczeniem a kompozycją,
- używać klas abstrakcyjnych (`ABC`), by wymusić kontrakt na klasach pochodnych,
- pisać własne context managery (`with` block).


## Dataclasses — mniej boilerplate'u

Gdy piszesz klasę, która głównie **przechowuje dane**, większość kodu w niej to konstruktor i `__repr__`. Każdy atrybut przepisujesz dwa razy — raz w sygnaturze `__init__`, raz w `self.x = x`. Po piątym takim "tępym" konstruktorze człowiek się buntuje.

Klasyczny przykład:

```python
class Book:

    def __init__(self, title, author, copies):
        self.title = title
        self.author = author
        self.copies = copies

    def __repr__(self):
        return f"Book(title={self.title!r}, author={self.author!r}, copies={self.copies})"

    def __eq__(self, other):
        if not isinstance(other, Book):
            return False
        return (self.title, self.author, self.copies) == (other.title, other.author, other.copies)
```

To samo z `dataclass`:

```python
from dataclasses import dataclass


@dataclass
class Book:
    title: str
    author: str
    copies: int
```

Tyle. Dekorator `@dataclass` automatycznie generuje za Ciebie `__init__`, `__repr__` i `__eq__`. Atrybuty definiujesz **na poziomie klasy** z **type hintami** — bez `self.`.

Użycie wygląda dokładnie tak samo:

```python
book = Book("Pan Tadeusz", "Mickiewicz", 3)
print(book)            # Book(title='Pan Tadeusz', author='Mickiewicz', copies=3)
print(book.title)      # Pan Tadeusz

b2 = Book("Pan Tadeusz", "Mickiewicz", 3)
print(book == b2)      # True — dataclass porównuje po wartościach atrybutów
```

### Wartości domyślne i pola mutowalne

```python
from dataclasses import dataclass, field


@dataclass
class Reader:
    login: str
    password: str
    borrowed_books: list = field(default_factory=list)
```

Dlaczego `field(default_factory=list)` zamiast `borrowed_books: list = []`? Bo wartości domyślne w Pythonie tworzą się **raz** — wszystkie instancje dzieliłyby tę samą listę. To klasyczna pułapka. `field(default_factory=list)` mówi: "przy każdym tworzeniu instancji uruchom `list()` od nowa".

### Inne przydatne opcje

```python
@dataclass(frozen=True)        # obiekt staje się niemutowalny
@dataclass(order=True)         # generuje też __lt__, __le__, __gt__, __ge__
@dataclass(slots=True)         # mniejsze zużycie pamięci (Python 3.10+)
```

`frozen=True` warto znać — robi z dataclass odpowiednik tuple z nazwanymi polami. Próba `book.title = "..."` rzuci wtedy wyjątek.

### Kiedy dataclass, kiedy zwykła klasa?

| Sytuacja | Wybierz |
|---|---|
| Klasa głównie trzyma dane, niewiele logiki | `dataclass` |
| Klasa ma bogatą logikę (walidacja w setterach, properties, metody biznesowe) | zwykła klasa |
| Chcesz immutable obiekt z porównywaniem po wartości | `@dataclass(frozen=True)` |

Dataclass nie wyklucza dodawania metod — możesz mieszać:

```python
@dataclass
class Book:
    title: str
    author: str
    copies: int

    def borrow(self):
        if self.copies <= 0:
            raise ValueError(f"Brak egzemplarzy: {self.title}")
        self.copies -= 1
```


## Pydantic — walidacja danych

`dataclass` jest świetny, gdy **ufasz danym** wchodzącym do klasy — np. tworzysz obiekty z wartości wpisanych na sztywno w kodzie. Ale w realnym świecie często danym nie ufasz: przychodzą z formularza, z API, z pliku JSON, od użytkownika. Wtedy potrzebujesz **walidacji**.

[Pydantic](https://docs.pydantic.dev/) to biblioteka, która rozwiązuje ten problem. Definiujesz klasę z type hintami, a Pydantic sam sprawdza i konwertuje typy w momencie tworzenia obiektu. Jest sercem m.in. **FastAPI**, więc jeśli kiedyś będziesz pisać API w Pythonie albo cokolwiek z ML/AI, prawie na pewno trafisz na Pydantic.

### Instalacja

```bash
uv add pydantic
```

### Najprostszy model

```python
from pydantic import BaseModel


class Book(BaseModel):
    title: str
    author: str
    copies: int


book = Book(title="Pan Tadeusz", author="Mickiewicz", copies=3)
print(book)         # title='Pan Tadeusz' author='Mickiewicz' copies=3
print(book.title)   # Pan Tadeusz
```

Wygląda jak `dataclass`, prawda? Sygnatura tworzenia obiektu jest podobna, atrybuty mają type hinty. Różnica zaczyna się, gdy dane są "niegrzeczne".

### Walidacja typów

```python
# To zadziała — string da się skonwertować na int (coercion)
book = Book(title="Pan Tadeusz", author="Mickiewicz", copies="3")
print(type(book.copies))   # <class 'int'> — Pydantic skonwertował!

# A to już nie:
book = Book(title="Pan Tadeusz", author="Mickiewicz", copies="abc")
# ValidationError: 1 validation error for Book
#   copies
#     Input should be a valid integer, ...
```

W `dataclass` ten drugi przypadek **przeszedłby bez słowa** — atrybut byłby stringiem, a dopiero przy `book.copies + 1` wybuchłoby pół godziny później w innym miejscu kodu. Pydantic łapie problem **na granicy** — w momencie wejścia danych do systemu.

### Wartości domyślne i pola opcjonalne

```python
from pydantic import BaseModel


class Book(BaseModel):
    title: str
    author: str
    copies: int = 1                # wartość domyślna
    isbn: str | None = None        # pole opcjonalne


book = Book(title="Lalka", author="Prus")
print(book.copies)   # 1
print(book.isbn)     # None
```

### Ograniczenia przez `Field`

Najczęstsze warunki (długość stringa, zakres liczby) można zapisać deklaratywnie przez `Field`, bez pisania osobnego walidatora:

```python
from pydantic import BaseModel, Field


class Book(BaseModel):
    title: str = Field(min_length=1, max_length=200)
    author: str = Field(min_length=1)
    copies: int = Field(ge=0)        # ge = greater or equal, czyli >= 0
    year: int = Field(ge=1450, le=2100)


Book(title="", author="X", copies=1, year=2020)
# ValidationError: title — String should have at least 1 character

Book(title="OK", author="X", copies=-5, year=2020)
# ValidationError: copies — Input should be greater than or equal to 0
```

### Walidatory własne

Gdy logika sprawdzania nie mieści się w prostych ograniczeniach, używasz `@field_validator`:

```python
from pydantic import BaseModel, field_validator


class Book(BaseModel):
    title: str
    author: str
    copies: int

    @field_validator("title")
    @classmethod
    def title_nonempty(cls, v):
        if not v.strip():
            raise ValueError("Tytuł nie może być pusty")
        return v.strip()       # walidator może też przekształcić wartość

    @field_validator("author")
    @classmethod
    def author_capitalized(cls, v):
        return v.strip().title()


book = Book(title="  pan tadeusz  ", author="adam mickiewicz", copies=3)
print(book.title)    # pan tadeusz
print(book.author)   # Adam Mickiewicz
```

Walidator może **przekształcić** wartość, nie tylko ją sprawdzić. Dane wchodzące mogą być "brudne", a wychodzące — znormalizowane. To bardzo cenne w projektach, gdzie input idzie z wielu źródeł.

### Serializacja — JSON in, JSON out

To jeden z głównych powodów, dla których Pydantic wygrywa w API i ML:

```python
book = Book(title="Pan Tadeusz", author="Mickiewicz", copies=3)

# obiekt → dict
print(book.model_dump())
# {'title': 'Pan Tadeusz', 'author': 'Mickiewicz', 'copies': 3}

# obiekt → JSON string
print(book.model_dump_json())
# {"title":"Pan Tadeusz","author":"Mickiewicz","copies":3}

# dict → obiekt (z walidacją!)
data = {"title": "Lalka", "author": "Prus", "copies": 2}
book = Book.model_validate(data)

# JSON string → obiekt (z walidacją!)
raw = '{"title":"Ferdydurke","author":"Gombrowicz","copies":1}'
book = Book.model_validate_json(raw)
```

Wczytanie payloadu z API, walidacja, konwersja typów — to wszystko jedna linia.

### Modele zagnieżdżone

```python
class Author(BaseModel):
    name: str
    birth_year: int


class Book(BaseModel):
    title: str
    author: Author            # zagnieżdżony model
    copies: int


data = {
    "title": "Pan Tadeusz",
    "author": {"name": "Mickiewicz", "birth_year": 1798},
    "copies": 3,
}

book = Book.model_validate(data)
print(book.author.name)         # Mickiewicz
print(book.author.birth_year)   # 1798
```

Pydantic sam rozpozna, że trzeba zbudować `Author` z zagnieżdżonego dicta. Walidacja działa rekurencyjnie — błąd w `author.birth_year` zostanie zgłoszony równie ładnie, co błąd w `copies`.

### Pydantic vs dataclass — kiedy co?

| Sytuacja | Wybierz |
|---|---|
| Dane już znane, ufam im (kod wewnętrzny, testy) | `dataclass` |
| Dane z zewnątrz (API, formularz, plik JSON, CSV) | `pydantic` |
| Liczy się surowa szybkość tworzenia obiektów | `dataclass` |
| Potrzebuję walidacji + konwersji typów + serializacji | `pydantic` |
| Brak zależności zewnętrznych (standardowa biblioteka) | `dataclass` |
| Piszę API (np. FastAPI) lub model ML | `pydantic` |

W projekcie biblioteki **nie musisz** używać Pydantica — dane są pod Twoją kontrolą, więc `dataclass` lub zwykła klasa w zupełności wystarczą. Ale warto wiedzieć, że Pydantic istnieje — w realnym kodzie webowym i ML/AI to dziś standard.


## Metody klasowe i statyczne

Do tej pory pisaliśmy tylko **metody instancyjne** — takie z `self` jako pierwszym parametrem. Ale w klasie mogą być też metody, które **nie potrzebują konkretnego obiektu**.

### `@staticmethod` — funkcja w przestrzeni nazw klasy

Metoda statyczna to po prostu funkcja, która "mieszka" w klasie, ale nie używa ani obiektu (`self`), ani klasy (`cls`). Logicznie jest powiązana z klasą, ale nie potrzebuje stanu.

```python
class Book:

    def __init__(self, title, isbn):
        self.title = title
        self.isbn = isbn

    @staticmethod
    def is_valid_isbn(isbn):
        return len(isbn.replace("-", "")) in (10, 13)


print(Book.is_valid_isbn("978-83-7469-001-2"))    # True
print(Book.is_valid_isbn("123"))                   # False
```

Możesz ją wywołać bez tworzenia obiektu: `Book.is_valid_isbn(...)`. Często używane do funkcji pomocniczych, walidatorów, konwerterów.

### `@classmethod` — operuje na klasie, nie na obiekcie

Metoda klasowa dostaje jako pierwszy parametr **klasę** (konwencjonalnie `cls`), nie konkretny obiekt. Najpopularniejszy use case: **alternatywne konstruktory**.

```python
@dataclass
class Book:
    title: str
    author: str
    copies: int

    @classmethod
    def from_csv_row(cls, row):
        title, author, copies = row.split(";")
        return cls(title.strip(), author.strip(), int(copies))


book = Book.from_csv_row("Pan Tadeusz; Mickiewicz; 3")
print(book)   # Book(title='Pan Tadeusz', author='Mickiewicz', copies=3)
```

Po co `cls(...)` zamiast `Book(...)`? Bo jeśli ktoś odziedziczy po `Book`, `cls.from_csv_row(...)` zwróci instancję **klasy pochodnej**, a nie zawsze `Book`. Czyli to bezpieczniejsza wersja.

### Porównanie

| Rodzaj | Pierwszy parametr | Dostęp do obiektu? | Dostęp do klasy? | Typowy use case |
|---|---|---|---|---|
| Instancyjna | `self` | tak | tak (przez `type(self)`) | normalne operacje na obiekcie |
| `@classmethod` | `cls` | nie | tak | alternatywny konstruktor, fabryki |
| `@staticmethod` | brak | nie | nie | funkcja pomocnicza powiązana z klasą |


## Kompozycja vs dziedziczenie

Dziedziczenie jest fajne, ale **łatwo go nadużyć**. Klasyczna pułapka: ktoś widzi wspólny atrybut, dziedziczy "dla oszczędności", a po roku okazuje się, że ma hierarchię na 5 poziomów i nie wiadomo, co dziedziczy po czym.

W realnym kodzie istnieje zasada: **prefer composition over inheritance** — kompozycja zamiast dziedziczenia.

### Dziedziczenie — relacja "jest"

```python
class Book:
    pass


class Audiobook(Book):   # Audiobook JEST książką — OK
    pass
```

### Kompozycja — relacja "ma"

```python
class Address:
    def __init__(self, street, city):
        self.street = street
        self.city = city


class Library:

    def __init__(self, name, address):
        self.name = name
        self.address = address    # Library MA adres — kompozycja


lib = Library("Biblioteka Białołęcka", Address("Modlińska 6", "Warszawa"))
print(lib.address.city)   # Warszawa
```

### Przykład — kiedy kompozycja wygrywa

Załóżmy, że `Library` musi mieć logger, walidator, magazyn książek i system rezerwacji. Pokusa: zrobić wielokrotne dziedziczenie. Lepiej:

```python
class Logger:
    def log(self, msg):
        print(f"[LOG] {msg}")


class BookStorage:
    def __init__(self):
        self._books = []

    def add(self, book):
        self._books.append(book)

    def all(self):
        return list(self._books)


class Library:

    def __init__(self, name):
        self.name = name
        self.logger = Logger()         # ma loggera
        self.storage = BookStorage()   # ma magazyn

    def add_book(self, book):
        self.storage.add(book)
        self.logger.log(f"Dodano: {book}")
```

Plusy:
- **Każda klasa robi jedną rzecz** — łatwiej testować i zmieniać.
- **Możesz wymienić komponent** — np. podmienić `Logger` na `FileLogger` bez zmiany `Library`.
- **Nie wpadasz w pułapki MRO** (kolejność dziedziczenia przy wielu rodzicach).

### Reguła kciuka

> Pytaj: czy `X` **jest** typu `Y` (dziedziczenie), czy `X` **ma** `Y` (kompozycja)?

- `Reader` **jest** typu `User` → dziedziczenie ✅
- `Library` **ma** magazyn i loggera → kompozycja ✅
- `Library` **jest** typu `User`? → nigdy ❌


## Klasy abstrakcyjne — rozwinięcie

Na zajęciach 2 wspomnieliśmy o module `abc`. Dziś pokazuję, do czego się go realnie używa.

**Klasa abstrakcyjna** to taka, której **nie można zinstancjować**. Służy wyłącznie jako szablon dla klas pochodnych — wymusza, że muszą one zaimplementować pewne metody. To kontrakt.

```python
from abc import ABC, abstractmethod


class User(ABC):

    def __init__(self, login, password):
        self.login = login
        self._password = password

    def authenticate(self, password):
        return self._password == password

    @abstractmethod
    def menu(self):
        ...    # podklasy MUSZĄ to nadpisać

    @abstractmethod
    def role_name(self):
        ...


class Reader(User):
    def menu(self):
        print("1. Przeglądaj  2. Wypożycz  3. Moje wypożyczenia")

    def role_name(self):
        return "czytelnik"


class Librarian(User):
    def menu(self):
        print("1. Lista wypożyczeń  2. Prośby o przedłużenie")

    def role_name(self):
        return "bibliotekarz"


# u = User("anna", "tajne")
# → TypeError: Can't instantiate abstract class User with abstract methods menu, role_name

reader = Reader("anna", "tajne")
reader.menu()
print(reader.role_name())
```

Co się dzieje, gdy zapomnisz zaimplementować abstrakcyjną metodę?

```python
class BadUser(User):
    def menu(self):
        print("menu")
    # zapomnieliśmy o role_name


# BadUser("test", "test")
# → TypeError: Can't instantiate abstract class BadUser with abstract method role_name
```

Python pilnuje kontraktu w momencie **tworzenia instancji**. Klasa, która nie nadpisała wszystkich metod abstrakcyjnych, sama też jest abstrakcyjna.

### Po co to w praktyce?

W naszej bibliotece klasa `User` ma sens jako abstrakt — sama jako taka nie istnieje, istnieją tylko czytelnik i bibliotekarz, każdy z własnym menu. ABC dokumentuje to wprost: "tu musi być coś, w klasie pochodnej dopisz co".

Na zajęciach nie wymagamy ABC w projekcie, ale jeśli chcesz — śmiało użyj. Wygląda profesjonalnie i nie kosztuje wiele linii.


## Context managery — własny `with`

Konstrukcję `with` znasz z czytania plików:

```python
with open("data.txt") as f:
    content = f.read()
# po wyjściu z bloku plik jest zamknięty AUTOMATYCZNIE
```

To jest **context manager**. Jego sednem są dwie metody specjalne: `__enter__` (uruchamia się przy wejściu do bloku) i `__exit__` (uruchamia się przy wyjściu — nawet jeśli rzucono wyjątek).

### Własny context manager

```python
class Timer:

    def __enter__(self):
        import time
        self.start = time.perf_counter()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        import time
        elapsed = time.perf_counter() - self.start
        print(f"Czas: {elapsed:.3f}s")
        return False    # False → ewentualny wyjątek leci dalej; True → zostaje przełknięty


with Timer():
    sum(range(10_000_000))
# Czas: 0.142s
```

### Po co to znać?

Context managery zapewniają, że **kod sprzątający się wykona** — niezależnie od tego, czy w bloku poleciał wyjątek, czy nie. Klasyczne use case'y:

- otwieranie/zamykanie plików,
- otwieranie/zamykanie połączenia z bazą danych,
- akwizycja/zwolnienie blokady (lock),
- pomiar czasu,
- tymczasowa zmiana konfiguracji.

Jest też prostszy sposób — przez dekorator `@contextmanager` z modułu `contextlib` — ale `__enter__`/`__exit__` warto zobaczyć raz, żeby wiedzieć jak to działa pod spodem.

---


## Programowanie funkcyjne

Na zajęciach 1 zarysowaliśmy programowanie funkcyjne — funkcje jako podstawowy budulec, brak zmiany stanu, transformacje danych. Dziś rozbijemy to na czynniki pierwsze i pokażę narzędzia, których realnie używasz w Pythonie codziennie — często bez świadomości, że to "funkcyjne podejście".

Po dzisiejszej części będziesz w stanie:

- pisać krótkie, anonimowe funkcje (`lambda`),
- transformować kolekcje przez `map`, `filter`, `reduce`,
- używać list, dict i set comprehensions zamiast pętli,
- sortować po dowolnym kryterium z `sorted(key=...)`,
- pisać funkcje wyższego rzędu (przyjmujące inne funkcje),
- rozumieć closures i czystość funkcji,
- używać `functools` (`partial`, `reduce`, `lru_cache`).


## Funkcje to obywatele pierwszej klasy

W Pythonie **funkcja jest zwykłą wartością** — możesz ją przypisać do zmiennej, włożyć do listy, przekazać jako argument, zwrócić z innej funkcji. Brzmi banalnie, ale to fundament całego paradygmatu funkcyjnego.

```python
def square(x):
    return x * x


# Funkcja jako wartość:
f = square
print(f(5))           # 25

# Funkcja w liście:
operations = [square, abs, str]
for op in operations:
    print(op(-3))     # 9, 3, "-3"

# Funkcja jako argument:
def apply(fn, value):
    return fn(value)

print(apply(square, 4))   # 16
```

W językach mocno proceduralnych (C, starsze Pascal) to się nie udaje — funkcja nie jest wartością. W Pythonie jest. Stąd cały bogaty świat narzędzi, które omówimy poniżej.


## Lambda — funkcja anonimowa

`lambda` to **wyrażenie**, które tworzy małą, jednolinijkową funkcję bez nazwy.

Składnia:

```
lambda parametry: wyrażenie
```

Te dwa zapisy są równoważne:

```python
def square(x):
    return x * x

square = lambda x: x * x
```

Po co? Bo często chcesz przekazać krótką funkcję jako argument do innej funkcji — i tworzenie pełnoprawnego `def` dla jednej linii to przesada.

```python
# zamiast tego:
def is_available(book):
    return book["copies"] > 0

available = list(filter(is_available, books))

# można to:
available = list(filter(lambda b: b["copies"] > 0, books))
```

### Ograniczenia lambdy

- **Tylko jedno wyrażenie** — żadnych `if`/`while`/`for` blokowych, żadnych przypisań.
- **Brak nazwy** (no name) — w stack trace zobaczysz `<lambda>`, nie sensowną nazwę.
- **Nieczytelna, gdy zbyt skomplikowana** — gdy ciało robi się dłuższe niż linia, lepiej napisać normalny `def`.

Reguła kciuka: lambda jest dobra dla krótkich predykatów (`lambda x: x > 0`) i prostych transformacji (`lambda x: x.upper()`). Wszystko ponad to → zwykła funkcja.


## `map` — przekształcanie elementów

`map(function, iterable)` przepuszcza każdy element przez funkcję i zwraca **iterator** wyników.

```python
numbers = [1, 2, 3, 4, 5]
squares = list(map(lambda x: x * x, numbers))
print(squares)   # [1, 4, 9, 16, 25]
```

Uwaga — `map` zwraca **iterator** (leniwy), nie listę. Jeśli chcesz listę, trzeba opakować w `list()`.

W praktyce w Pythonie częściej używa się comprehensions niż `map` (czytelniej), ale `map` ma sens, gdy:

```python
# masz już gotową funkcję
def normalize(text):
    return text.strip().lower()

names = ["  Anna ", "TOMEK", "kasia"]
print(list(map(normalize, names)))   # ['anna', 'tomek', 'kasia']
```


## `filter` — wybieranie elementów

`filter(predicate, iterable)` zostawia tylko te elementy, dla których `predicate(element)` zwróci `True`.

```python
numbers = [1, 2, 3, 4, 5, 6]
even = list(filter(lambda x: x % 2 == 0, numbers))
print(even)   # [2, 4, 6]
```

Realny przykład:

```python
books = [
    {"title": "Pan Tadeusz", "copies": 3},
    {"title": "Lalka",       "copies": 0},
    {"title": "Ferdydurke",  "copies": 1},
]

available = list(filter(lambda b: b["copies"] > 0, books))
```


## `reduce` — agregacja do pojedynczej wartości

`reduce` (z `functools`) przepuszcza kolekcję przez funkcję, która łączy element po elemencie w jeden wynik.

```python
from functools import reduce

numbers = [1, 2, 3, 4, 5]
total = reduce(lambda acc, x: acc + x, numbers)
print(total)   # 15
```

Mechanika krok po kroku dla `[1, 2, 3, 4, 5]`:
```
((((1 + 2) + 3) + 4) + 5)  →  15
```

Drugi argument można dać jako **wartość początkową**:

```python
reduce(lambda acc, x: acc + x, numbers, 100)   # 115
```

W Pythonie do typowych agregacji są szybsze wbudowane funkcje — `sum`, `min`, `max`, `any`, `all` — więc `reduce` widzisz rzadziej. Ale do nietypowych operacji bywa nie do zastąpienia:

```python
# konkatenacja stringów z separatorem
words = ["Python", "jest", "świetny"]
sentence = reduce(lambda a, b: a + " " + b, words)
print(sentence)   # Python jest świetny

# znajdowanie najdłuższego stringa
longest = reduce(lambda a, b: a if len(a) >= len(b) else b, words)
print(longest)    # świetny
```


## Comprehensions — pythoniczny sposób

Comprehensions to składnia Pythona, która łączy w jednej linii to, co `map` i `filter` robią osobno. Dla wielu Pythonowców to **domyślne narzędzie** do transformacji kolekcji.

### List comprehension

```python
# zamiast:
squares = list(map(lambda x: x * x, range(10)))

# pisze się:
squares = [x * x for x in range(10)]
```

Z filtrowaniem:

```python
# zamiast:
even_squares = list(map(lambda x: x * x, filter(lambda x: x % 2 == 0, range(10))))

# pisze się:
even_squares = [x * x for x in range(10) if x % 2 == 0]
```

Czytelniejsze, prawda? Realny przykład:

```python
books = [
    {"title": "Pan Tadeusz", "author": "Mickiewicz", "copies": 3},
    {"title": "Lalka",       "author": "Prus",       "copies": 0},
    {"title": "Ferdydurke",  "author": "Gombrowicz", "copies": 1},
]

# tytuły dostępnych książek
available_titles = [b["title"] for b in books if b["copies"] > 0]
print(available_titles)   # ['Pan Tadeusz', 'Ferdydurke']
```

### Dict comprehension

```python
books = ["Pan Tadeusz", "Lalka", "Ferdydurke"]
indexed = {i: title for i, title in enumerate(books)}
print(indexed)   # {0: 'Pan Tadeusz', 1: 'Lalka', 2: 'Ferdydurke'}

# tytuł → liczba sztuk (filtracja po drodze)
stock = {b["title"]: b["copies"] for b in books_list if b["copies"] > 0}
```

### Set comprehension

```python
authors = {b["author"] for b in books}    # unikalne autorzy
```

### Generator expression — leniwa wersja

Wystarczy zamienić nawiasy kwadratowe na zwykłe — wynik to **iterator** liczony na żądanie, nie cała lista w pamięci:

```python
total_copies = sum(b["copies"] for b in books)
print(total_copies)
```

To bywa istotne przy dużych zbiorach — nie tworzymy listy 10 milionów elementów, tylko zliczamy je w locie.

### Kiedy comprehension, kiedy `map`/`filter`?

Reguła kciuka: jeśli logika mieści się w jednej linii **bez dziwactw**, comprehension jest czytelniejszy. Jeśli operacja na elemencie to wywołanie gotowej, nazwanej funkcji — `map` też jest OK. Unikaj zagnieżdżonych, kilkupoziomowych comprehensions — nie warto, lepsza zwykła pętla.


## `sorted` z parametrem `key`

`sorted(iterable, key=function, reverse=False)` sortuje, używając wyniku `key(element)` jako podstawy porównania.

```python
books = [
    {"title": "Pan Tadeusz", "copies": 3},
    {"title": "Lalka",       "copies": 0},
    {"title": "Ferdydurke",  "copies": 1},
]

# alfabetycznie po tytule
by_title = sorted(books, key=lambda b: b["title"])

# malejąco po liczbie sztuk
by_copies = sorted(books, key=lambda b: b["copies"], reverse=True)
```

### Sortowanie po wielu polach

Wystarczy zwrócić **krotkę** z `key` — Python porównuje krotki leksykograficznie:

```python
# najpierw rosnąco po autorze, potem malejąco po liczbie sztuk
sorted(books, key=lambda b: (b["author"], -b["copies"]))
```

Minus przed liczbą sztuk odwraca kolejność tego konkretnego pola, gdy reszta ma rosnąć.

### `sorted` vs `list.sort`

- `sorted(lst)` — zwraca **nową listę**, oryginalna zostaje.
- `lst.sort()` — sortuje **w miejscu**, zwraca `None`.

W stylu funkcyjnym używamy `sorted`, bo nie zmienia oryginału (immutability).


## Funkcje wyższego rzędu

**Higher-order function** to funkcja, która:
- przyjmuje inną funkcję jako argument, lub
- zwraca inną funkcję jako wynik.

`map`, `filter`, `sorted`, `reduce` — wszystkie te wbudowane to funkcje wyższego rzędu. Ale i Ty możesz takie pisać.

### Przyjmowanie funkcji jako argumentu

```python
def show_filtered(items, predicate, label):
    print(f"--- {label} ---")
    for item in filter(predicate, items):
        print(item)


books = [
    {"title": "Pan Tadeusz", "copies": 3},
    {"title": "Lalka",       "copies": 0},
    {"title": "Ferdydurke",  "copies": 1},
]

show_filtered(books, lambda b: b["copies"] > 0, "Dostępne")
show_filtered(books, lambda b: b["copies"] == 0, "Wyczerpane")
```

`show_filtered` nie wie, czego konkretnie szukamy — to wybiera dzwoniący, przekazując predykat. Klasyczny wzorzec, używany w wielu bibliotekach.

### Zwracanie funkcji jako wyniku

```python
def make_multiplier(factor):
    def multiply(x):
        return x * factor
    return multiply


double = make_multiplier(2)
triple = make_multiplier(3)

print(double(10))   # 20
print(triple(10))   # 30
```

`make_multiplier` to fabryka funkcji. Każde wywołanie tworzy nową funkcję zapamiętującą swój `factor`. To prowadzi nas wprost do closures.


## Closures — funkcje, które pamiętają

**Closure** (domknięcie) to funkcja, która "pamięta" zmienne ze swojego otoczenia — nawet gdy zewnętrzna funkcja już dawno się skończyła.

```python
def make_counter():
    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment


counter = make_counter()
print(counter())   # 1
print(counter())   # 2
print(counter())   # 3
```

`increment` korzysta ze zmiennej `count` zdefiniowanej w `make_counter`. Mimo że `make_counter` skończyła się po pierwszym wywołaniu, `count` żyje dalej — bo jest "domknięta" w środku `increment`.

Słowo `nonlocal` jest tu kluczowe — bez niego Python potraktowałby `count` jako nową, lokalną zmienną wewnątrz `increment` i `count += 1` by się wysypał. `nonlocal` mówi: "korzystaj z `count` z otaczającej funkcji".

### Praktyczne zastosowanie

```python
def make_filter(min_copies):
    return lambda book: book["copies"] >= min_copies


has_at_least_one = make_filter(1)
has_at_least_three = make_filter(3)

print(list(filter(has_at_least_one, books)))
print(list(filter(has_at_least_three, books)))
```

Closures są ciche, ale wszechobecne — m.in. dlatego, że dekoratory (poniżej) to closures.


## Dekoratory — krótko

Dekorator to **funkcja, która opakowuje inną funkcję** — dodaje jej zachowanie, ale jej nie modyfikuje. Na poziomie składni:

```python
@my_decorator
def function():
    ...
```

…jest skrótem dla:

```python
function = my_decorator(function)
```

Czyli `my_decorator` musi być funkcją, która **przyjmuje funkcję** i **zwraca funkcję** (czyli higher-order function w obie strony).

### Najprostszy przykład

```python
def log_calls(fn):
    def wrapper(*args, **kwargs):
        print(f"Wywołuję {fn.__name__}({args}, {kwargs})")
        result = fn(*args, **kwargs)
        print(f"  → {result}")
        return result
    return wrapper


@log_calls
def add(a, b):
    return a + b


add(2, 3)
# Wywołuję add((2, 3), {})
#   → 5
```

Dekoratory już znasz z OOP: `@property`, `@staticmethod`, `@classmethod`, `@dataclass`. To wszystko dekoratory zaimplementowane w Pythonie.

Nie wymagamy ich w projekcie — pokazuję, byś rozpoznał składnię.


## Czystość funkcji i niezmienność

W idealnym świecie funkcyjnym **funkcje są czyste** (pure):

- ten sam wynik dla tych samych argumentów,
- **brak efektów ubocznych** (nie modyfikują nic na zewnątrz — listy, zmiennej globalnej, pliku, bazy).

Czysta funkcja:

```python
def add(a, b):
    return a + b
```

Brudna:

```python
total = 0

def add_to_total(x):
    global total
    total += x       # efekt uboczny — zmienia stan globalny
    return total
```

Drugi przykład trudniej zrozumieć (musisz znać stan `total`), trudniej testować (zależy od tego, kiedy ją wywołasz) i trudniej zrównoleglić.

### Niezmienność (immutability)

W stylu funkcyjnym **nie modyfikujemy danych w miejscu** — zwracamy nową kopię. W Pythonie nie jest to wymuszone, ale można pisać w tym duchu.

```python
# stylowo (mutuje listę):
books.append(new_book)

# funkcyjnie (zwraca nową listę):
new_books = books + [new_book]
```

Plusy podejścia funkcyjnego:
- Łatwiej śledzić, co się zmienia.
- Brak side-effectów = łatwiejsze testy.
- Bezpieczeństwo przy wielowątkowości.

Minusy: czasem droższe pamięciowo, czasem mniej naturalne.

W praktyce w Pythonie pisze się **mieszany styl** — modeluje obiekty (OOP), ale operacje na kolekcjach robi się funkcyjnie. I to jest OK.


## `functools` — pakowane narzędzia funkcyjne

Moduł `functools` zawiera kilka rzeczy, które warto znać.

### `reduce` — już omówione wyżej

```python
from functools import reduce
reduce(lambda acc, x: acc + x, [1, 2, 3])    # 6
```

### `partial` — utrwalanie argumentów

`partial(fn, *args)` tworzy nową funkcję z **już ustawionymi** częściowo argumentami.

```python
from functools import partial

def power(base, exponent):
    return base ** exponent


square = partial(power, exponent=2)
cube = partial(power, exponent=3)

print(square(5))   # 25
print(cube(5))     # 125
```

Coś podobnego osiągniesz lambdą (`lambda x: power(x, 2)`), ale `partial` jest czytelniejszy, gdy argumentów jest więcej.

### `lru_cache` — memoizacja

Dekorator, który **zapamiętuje wyniki** wywołań funkcji — kolejne wywołania z tymi samymi argumentami zwracają wynik bez liczenia.

```python
from functools import lru_cache


@lru_cache(maxsize=None)
def fib(n):
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)


print(fib(100))    # liczy w mgnieniu oka — bez cache to by trwało wieczność
```

Pamiętaj: `lru_cache` zadziała poprawnie tylko dla **czystych funkcji** — bez memoizacji efektów ubocznych. Stąd związek z funkcyjnym podejściem.


## Łączymy paradygmaty — mini przykład biblioteki

W realnym kodzie mieszasz OOP i FP — modelujesz świat obiektami, ale operacje na kolekcjach piszesz w stylu funkcyjnym.

```python
from dataclasses import dataclass, field


@dataclass
class Book:
    title: str
    author: str
    copies: int


@dataclass
class Reader:
    login: str
    borrowed: list = field(default_factory=list)


@dataclass
class Library:
    books: list = field(default_factory=list)
    readers: list = field(default_factory=list)

    def search(self, predicate):
        """Funkcja wyższego rzędu — kryterium podajesz Ty."""
        return list(filter(predicate, self.books))

    def sorted_by(self, key):
        """Też higher-order — sposób sortowania wybiera dzwoniący."""
        return sorted(self.books, key=key)

    def stats(self):
        return {
            "total_books":   sum(b.copies for b in self.books),
            "unique_titles": len({b.title for b in self.books}),
            "authors":       sorted({b.author for b in self.books}),
            "most_active":   sorted(
                self.readers,
                key=lambda r: len(r.borrowed),
                reverse=True,
            )[:3],
        }


lib = Library(
    books=[
        Book("Pan Tadeusz", "Mickiewicz", 3),
        Book("Lalka",       "Prus",       0),
        Book("Ferdydurke",  "Gombrowicz", 1),
    ],
    readers=[Reader("anna"), Reader("tomek")],
)

# Wyszukiwanie - predykat jako argument
available = lib.search(lambda b: b.copies > 0)

# Sortowanie - key jako argument
by_author = lib.sorted_by(lambda b: b.author)

# Filtrowanie po frazie - comprehension
matching = [b for b in lib.books if "tad" in b.title.lower()]

# Mapowanie - kropka tytułów
titles = list(map(lambda b: b.title, available))
```

Co tu się dzieje:

- **OOP** — modelujemy świat obiektami (`Book`, `Reader`, `Library`), dane i metody razem.
- **FP** — operacje wewnątrz tych metod robimy przez `filter`, `map`, `sorted`, comprehensions, generator expressions.
- **Higher-order** — metody `search` i `sorted_by` przyjmują funkcje jako argumenty.

To **ten sam wzorzec**, którego użyjesz w Części 3 projektu.


## FP w praktyce — kilka rad

1. **Comprehension > map/filter w prostych przypadkach** — czytelniej i pythoniczniej. Ale `map(funkcja_nazwana, iterable)` też jest OK.
2. **Lambda krótko** — jednolinijkowy predykat, jednolinijkowa transformacja. Dłużej → zrób `def`.
3. **Sortowanie wielokryterialne** — zwracaj krotkę w `key`, minus przed polem odwraca kierunek.
4. **`sorted` zamiast `list.sort`** — gdy chcesz zachować oryginał.
5. **Czystość się opłaca** — funkcje bez efektów ubocznych są łatwiejsze do testowania i debugowania.
6. **Mieszaj paradygmaty świadomie** — Python jest hybrydowy. Klasy do modelowania bytów, funkcyjne narzędzia do operacji na danych — i już masz styl większości produkcyjnego kodu.

---

## Praca projektowa

### Część 3 — Programowanie funkcyjne (20 pkt)

**Temat:** Biblioteka — rozszerzenie z wykorzystaniem programowania funkcyjnego

Rozszerz aplikację z Części 2 o nowe funkcjonalności, implementując je w **stylu funkcyjnym**: wyrażenia lambda, funkcje `map`, `filter`, `sorted` z kluczem, list/dict comprehensions, a także funkcje wyższego rzędu (funkcje przyjmujące lub zwracające inne funkcje).

**Nowe funkcjonalności:**

1. **Wyszukiwanie i filtrowanie katalogu** — czytelnik i bibliotekarz mogą filtrować książki po frazie w tytule lub autorze oraz wyświetlić tylko książki aktualnie dostępne (sztuki > 0). Filtrowanie zaimplementuj za pomocą `filter` + `lambda` lub comprehension.
2. **Sortowanie katalogu** — możliwość wyświetlenia książek posortowanych wg tytułu, autora lub liczby dostępnych sztuk. Użyj `sorted` z parametrem `key` jako lambda.
3. **Rezerwacja niedostępnego tytułu** — jeśli książka ma 0 dostępnych sztuk, czytelnik może ją zarezerwować. Przy obsłudze próśb o przedłużenie bibliotekarz widzi informację, czy na daną książkę istnieje rezerwacja.
4. **Statystyki (bibliotekarz)** — nowa opcja w menu bibliotekarza:
   - najpopularniejsza książka (największa różnica między łączną liczbą sztuk a dostępnymi),
   - liczba aktywnych wypożyczeń ogółem,
   - lista czytelników posortowana malejąco wg liczby wypożyczonych książek.
   
   Statystyki zaimplementuj przy użyciu `map`/`filter`/comprehension — bez pętli `for` z ręczną akumulacją.

5. **Funkcja wyższego rzędu** — napisz co najmniej jedną funkcję, która przyjmuje inną funkcję jako argument (np. uniwersalna funkcja do wyświetlania przefiltrowanej kolekcji, przyjmująca predykat filtrowania).

**Wymagania techniczne:**

- W nowych funkcjonalnościach nie stosuj klasycznych pętli `for`/`while` do filtrowania, przekształcania ani sortowania — używaj narzędzi programowania funkcyjnego.
- Co najmniej 3 użycia `lambda`.
- Co najmniej 2 użycia comprehension (list lub dict).
- Co najmniej 1 funkcja wyższego rzędu (przyjmująca funkcję jako argument).

**Punktacja (20 pkt):**

| Element | Punkty |
|---|---|
| Filtrowanie katalogu (`filter`/`lambda`/comprehension) | 3 |
| Sortowanie katalogu (`sorted` + `key` jako lambda) | 2 |
| Rezerwacja niedostępnego tytułu | 3 |
| Info o rezerwacji przy obsłudze próśb o przedłużenie | 2 |
| Statystyki bibliotekarza w stylu funkcyjnym | 4 |
| Funkcja wyższego rzędu | 3 |
| Spełnienie wymagań technicznych (min. 3× lambda, 2× comprehension, brak pętli w nowej logice) | 3 |
