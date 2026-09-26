# Zajęcia 2

## Programowanie obiektowe

Na poprzednich zajęciach poznaliśmy trzy paradygmaty programowania. Dziś skupimy się na **programowaniu obiektowym (OOP)** — sposobie myślenia o programie jako o zbiorze współpracujących obiektów, które reprezentują byty z naszego problemu.

Po dzisiejszych zajęciach będziesz w stanie:

- zdefiniować własną klasę i tworzyć z niej obiekty,
- ukryć dane wewnątrz obiektu (hermetyzacja),
- skorzystać z dziedziczenia, by uniknąć duplikowania kodu,
- napisać kod, który działa dla wielu typów (polimorfizm),
- wykorzystać metody specjalne (dunders), by Twoje obiekty zachowywały się jak natywne typy Pythona.


## Klasy i obiekty

**Klasa** to szablon — opisuje, jakie dane (atrybuty) i jakie zachowania (metody) ma mieć obiekt.
**Obiekt** to konkretna instancja klasy stworzona na bazie tego szablonu.

Analogia: klasa to przepis na ciasto, a obiekt to konkretne ciasto upieczone według tego przepisu. Z jednego przepisu (klasy) możesz upiec wiele ciast (obiektów), każde z innymi rodzynkami i polewą (atrybutami).

### Najprostsza klasa

```python
class Book:
    pass


book = Book()
print(type(book))   # <class '__main__.Book'>
```

Klasa `Book` nic jeszcze nie robi, ale pokazuje składnię — `class NazwaKlasy:` plus blok kodu. Konwencjonalnie nazwy klas piszemy w **PascalCase** (każde słowo z dużej litery, bez podkreśleń).

### Konstruktor `__init__` i `self`

Konstruktor to metoda uruchamiana automatycznie przy tworzeniu nowego obiektu. W Pythonie nazywa się `__init__`.

```python
class Book:

    def __init__(self, title, author, copies):
        self.title = title
        self.author = author
        self.copies = copies


book = Book("Pan Tadeusz", "Mickiewicz", 3)
print(book.title)    # Pan Tadeusz
print(book.copies)   # 3
```

`self` to referencja do **konkretnego obiektu**, na którym wywołujemy metodę. Jest pierwszym parametrem każdej metody — Python przekazuje go automatycznie:

```python
book.borrow()         # Python tłumaczy to na: Book.borrow(book)
```

Zapis `self.title = title` oznacza: zapisz wartość parametru `title` jako atrybut **tego konkretnego obiektu**.

### Metody

Metoda to funkcja zdefiniowana wewnątrz klasy:

```python
class Book:

    def __init__(self, title, author, copies):
        self.title = title
        self.author = author
        self.copies = copies

    def borrow(self):
        if self.copies > 0:
            self.copies -= 1
            print(f"Wypożyczono: {self.title}. Pozostało: {self.copies}")
        else:
            print(f"Brak dostępnych egzemplarzy: {self.title}")

    def return_copy(self):
        self.copies += 1


book = Book("Pan Tadeusz", "Mickiewicz", 2)
book.borrow()    # Wypożyczono: Pan Tadeusz. Pozostało: 1
book.borrow()    # Wypożyczono: Pan Tadeusz. Pozostało: 0
book.borrow()    # Brak dostępnych egzemplarzy: Pan Tadeusz
```

Zauważ — **dane i operacje na tych danych są w jednym miejscu**. Funkcja `borrow` operuje na atrybucie `self.copies` tego konkretnego obiektu. To jest sedno OOP: dane i ich logika żyją razem.

### Atrybuty klasy vs atrybuty obiektu

```python
class Book:
    library_name = "Biblioteka Główna"   # atrybut klasy (wspólny dla wszystkich)

    def __init__(self, title):
        self.title = title                # atrybut obiektu (każdy ma swój)


b1 = Book("Pan Tadeusz")
b2 = Book("Lalka")

print(b1.library_name)   # Biblioteka Główna
print(b2.library_name)   # Biblioteka Główna
print(b1.title)          # Pan Tadeusz
print(b2.title)          # Lalka
```

Atrybuty klasy są wspólne dla wszystkich instancji — używaj ich dla rzeczy, które się nie zmieniają między obiektami.


## Cztery filary OOP

Tradycyjnie wyróżnia się cztery filary programowania obiektowego:

1. **Hermetyzacja** (encapsulation) — ukrywanie wewnętrznego stanu obiektu
2. **Dziedziczenie** (inheritance) — tworzenie nowych klas na bazie istniejących
3. **Polimorfizm** (polymorphism) — różne klasy reagują na to samo wywołanie
4. **Abstrakcja** (abstraction) — pokazywanie tylko tego, co istotne

Omówmy je po kolei.



## Hermetyzacja (encapsulation)

Pomysł: obiekt powinien sam dbać o swój stan. Świat zewnętrzny **nie powinien** móc dowolnie modyfikować wewnętrznych danych — może z nimi rozmawiać tylko przez **publiczne metody**.

Po co? Po to, żeby logika walidacji i niezmienników była **w jednym miejscu** — w klasie.

### Przykład — czemu hermetyzacja jest ważna

Bez hermetyzacji:

```python
class BankAccount:

    def __init__(self, balance):
        self.balance = balance


account = BankAccount(1000)
account.balance = -50000   # nikt mnie nie powstrzyma!
```

Tu nic nie chroni naszego konta — każdy z zewnątrz może ustawić ujemne saldo.

Z hermetyzacją:

```python
class BankAccount:

    def __init__(self, balance):
        self._balance = balance

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("Kwota musi być dodatnia")
        self._balance += amount

    def withdraw(self, amount):
        if amount <= 0:
            raise ValueError("Kwota musi być dodatnia")
        if amount > self._balance:
            raise ValueError("Brak środków")
        self._balance -= amount

    def get_balance(self):
        return self._balance
```

Teraz klient klasy nie operuje bezpośrednio na `_balance` — używa metod, które pilnują reguł.

### Konwencje w Pythonie

Python **nie ma** prawdziwych modyfikatorów `private` jak Java czy C# — opiera się na **konwencjach**:

| Zapis | Znaczenie |
|---|---|
| `self.name` | Publiczny — używaj swobodnie |
| `self._name` | Chroniony (konwencja) — używaj tylko wewnątrz klasy lub klas pochodnych |
| `self.__name` | "Prywatny" — Python stosuje *name mangling* (zmienia nazwę na `_NazwaKlasy__name`) |

```python
class Example:

    def __init__(self):
        self.public = "wszyscy mogą"
        self._protected = "raczej nie ruszaj"
        self.__private = "name mangling"


e = Example()
print(e.public)             # OK
print(e._protected)         # technicznie działa, ale to konwencja "nie ruszaj"
print(e.__private)          # AttributeError!
print(e._Example__private)  # tak można, ale po co...
```

W praktyce w Pythonie **wystarcza pojedyncze podkreślenie**. To umowa między programistami: „to jest szczegół implementacyjny, nie polegaj na tym".

### Properties — getter i setter w Pythonie

Zamiast pisać `get_balance()` i `set_balance()`, w Pythonie używamy **dekoratora `@property`**:

```python
class BankAccount:

    def __init__(self, balance):
        self._balance = balance

    @property
    def balance(self):
        return self._balance

    @balance.setter
    def balance(self, value):
        if value < 0:
            raise ValueError("Saldo nie może być ujemne")
        self._balance = value


account = BankAccount(1000)
print(account.balance)    # 1000 — wygląda jak atrybut, ale jest metodą
account.balance = 500     # uruchamia setter z walidacją
account.balance = -100    # ValueError!
```

Dzięki temu API klasy wygląda jak praca na atrybutach, a pod spodem działa cała walidacja. To bardzo „pythoniczne" — Python promuje proste interfejsy.


## Dziedziczenie (inheritance)

Dziedziczenie pozwala stworzyć nową klasę, która **przejmuje wszystko** z klasy bazowej, a potem może coś dodać lub zmienić.

Po co? Żeby nie pisać tego samego kodu wielokrotnie. Jeśli `Reader` i `Librarian` mają wspólne pole `login`, niech to pole będzie w jednej klasie `User`.

### Składnia

```python
class User:

    def __init__(self, login, password):
        self.login = login
        self.password = password

    def authenticate(self, password):
        return self.password == password


class Reader(User):
    pass    # Reader ma już login, password i authenticate — z dziedziczenia


class Librarian(User):
    pass


reader = Reader("anna", "tajne")
print(reader.login)                    # anna
print(reader.authenticate("tajne"))    # True
```

Zapis `class Reader(User):` oznacza „Reader dziedziczy po User". `Reader` to **klasa pochodna**, `User` to **klasa bazowa**.

### Rozszerzanie klasy pochodnej i `super()`

Klasa pochodna może dodać własne atrybuty i metody:

```python
class Reader(User):

    def __init__(self, login, password):
        super().__init__(login, password)   # wywołanie konstruktora rodzica
        self.borrowed_books = []
        self.extension_requests = []

    def borrow(self, book):
        self.borrowed_books.append(book)


class Librarian(User):

    def __init__(self, login, password, employee_id):
        super().__init__(login, password)
        self.employee_id = employee_id
```

`super()` daje Ci dostęp do klasy bazowej. `super().__init__(login, password)` mówi: „uruchom konstruktor `User`, żeby ustawił `login` i `password`, a potem ja dorobię swoją część".

Bez `super().__init__` musiałbyś przepisać `self.login = login; self.password = password` ręcznie — duplikacja, której właśnie chcemy uniknąć.

### Nadpisywanie metod (method overriding)

Klasa pochodna może **nadpisać** metodę z klasy bazowej:

```python
class User:

    def __init__(self, login):
        self.login = login

    def menu(self):
        print("Menu domyślne")


class Reader(User):

    def menu(self):    # nadpisujemy
        print("Menu czytelnika: 1. Przeglądaj 2. Wypożycz 3. Moje wypożyczenia")


class Librarian(User):

    def menu(self):
        print("Menu bibliotekarza: 1. Lista wypożyczeń 2. Prośby o przedłużenie")


users = [Reader("anna"), Librarian("kowalski")]

for u in users:
    u.menu()    # każdy obiekt zachowa się inaczej
```

To prowadzi nas wprost do polimorfizmu.

### Dziedziczenie wielokrotne — krótko

Python pozwala dziedziczyć z wielu klas naraz (`class C(A, B):`), ale w praktyce rzadko jest to potrzebne i potrafi narobić bałaganu z kolejnością wywoływania metod (tzw. **MRO — Method Resolution Order**). Na zajęciach trzymaj się jednego rodzica.


## Polimorfizm (polymorphism)

Polimorfizm — z greki „wiele kształtów". W praktyce: ten sam interfejs, różne implementacje.

### Przykład — różne typy książek

```python
class Book:

    def __init__(self, title):
        self.title = title

    def description(self):
        return f"Książka: {self.title}"


class Audiobook(Book):

    def __init__(self, title, length_minutes):
        super().__init__(title)
        self.length_minutes = length_minutes

    def description(self):
        return f"Audiobook: {self.title} ({self.length_minutes} min)"


class Ebook(Book):

    def __init__(self, title, file_format):
        super().__init__(title)
        self.file_format = file_format

    def description(self):
        return f"E-book: {self.title} [{self.file_format}]"


items = [
    Book("Pan Tadeusz"),
    Audiobook("Lalka", 1200),
    Ebook("Ferdydurke", "PDF"),
]

for item in items:
    print(item.description())
```

Wynik:

```
Książka: Pan Tadeusz
Audiobook: Lalka (1200 min)
E-book: Ferdydurke [PDF]
```

W pętli wywołujemy `item.description()` — bez sprawdzania typu! Każdy obiekt wie, jak ma się zachować. Możesz dodać `Magazine`, `Comic`, cokolwiek — pętla pozostanie taka sama.

To jest siła polimorfizmu: **kod, który operuje na zbiorach obiektów, nie musi znać konkretnych typów**.

### Duck typing

Python idzie krok dalej — **nie wymaga dziedziczenia**, by polimorfizm zadziałał. Jeśli obiekt ma metodę o właściwej nazwie, można jej użyć:

```python
class Cat:
    def speak(self):
        return "miau"


class Robot:
    def speak(self):
        return "BIP BOP"


def talk(thing):
    print(thing.speak())


talk(Cat())     # miau
talk(Robot())   # BIP BOP
```

`Robot` nie dziedziczy po `Cat`, ale ma metodę `speak()` — i to wystarczy. To się nazywa **duck typing**: „jeśli coś chodzi jak kaczka i kwacze jak kaczka — to pewnie kaczka".

W praktyce — w naszym projekcie polimorfizm pojawi się w metodzie `menu()`, która będzie inna dla `Reader` i `Librarian`.



## Abstrakcja

Czwarty filar — abstrakcja — to idea pokazywania użytkownikowi klasy **tylko tego, co istotne**, a chowania reszty.

Przykład: wystarczy że wiesz, że samochód ma kierownicę i pedały — nie musisz znać szczegółów wtrysku paliwa.

W kodzie oznacza to: prosty, jasny **publiczny interfejs**, a skomplikowana implementacja schowana wewnątrz.

W zaawansowanym OOP istnieją tzw. **klasy abstrakcyjne** (w Pythonie moduł `abc`) — szablony, których nie da się instancjować, a tylko dziedziczyć. Wymuszają one zaimplementowanie pewnych metod w klasach pochodnych:

```python
from abc import ABC, abstractmethod


class User(ABC):

    def __init__(self, login):
        self.login = login

    @abstractmethod
    def menu(self):
        pass    # podklasy MUSZĄ to zaimplementować


# u = User("test")   # TypeError! Nie można utworzyć instancji klasy abstrakcyjnej
```

Na naszych zajęciach nie będziemy ich wymagać, ale warto wiedzieć, że istnieją — w projekcie wystarczy zwykła klasa bazowa.


## Dunders — metody specjalne

**Dunders** to metody otoczone podwójnym podkreśleniem, np. `__init__`, `__str__`. Nazwa to skrót od **d**ouble **under**score.

Pozwalają one Twoim obiektom zachowywać się jak natywne typy Pythona — można je drukować, porównywać, dodawać, używać w pętlach itp.

### `__str__` i `__repr__`

Domyślnie `print(obj)` wypisuje coś brzydkiego:

```python
class Book:
    def __init__(self, title, author):
        self.title = title
        self.author = author


book = Book("Pan Tadeusz", "Mickiewicz")
print(book)   # <__main__.Book object at 0x000001A2B3C4D5E0>
```

Definiujemy `__str__`:

```python
class Book:
    def __init__(self, title, author):
        self.title = title
        self.author = author

    def __str__(self):
        return f"{self.title} — {self.author}"


book = Book("Pan Tadeusz", "Mickiewicz")
print(book)        # Pan Tadeusz — Mickiewicz
print(str(book))   # Pan Tadeusz — Mickiewicz
```

`__str__` zwraca **przyjazny tekst dla użytkownika**.
`__repr__` zwraca **techniczny tekst dla programisty** (najlepiej taki, który da się skopiować do kodu):

```python
class Book:
    def __init__(self, title, author):
        self.title = title
        self.author = author

    def __repr__(self):
        return f"Book(title={self.title!r}, author={self.author!r})"


book = Book("Pan Tadeusz", "Mickiewicz")
print(repr(book))   # Book(title='Pan Tadeusz', author='Mickiewicz')
```

W konsoli interaktywnej i przy drukowaniu listy obiektów Python używa `__repr__`:

```python
books = [Book("Pan Tadeusz", "Mickiewicz"), Book("Lalka", "Prus")]
print(books)
# [Book(title='Pan Tadeusz', author='Mickiewicz'), Book(title='Lalka', author='Prus')]
```

Reguła kciuka: jeśli definiujesz tylko jedno — definiuj `__repr__`. Wtedy `print` na liście obiektów będzie miał sensowny output.

### `__eq__` — porównywanie

Domyślnie `==` porównuje obiekty po identyczności (czy to ten sam obiekt w pamięci):

```python
b1 = Book("Pan Tadeusz", "Mickiewicz")
b2 = Book("Pan Tadeusz", "Mickiewicz")
print(b1 == b2)   # False — to dwa różne obiekty
```

Możemy nadpisać:

```python
class Book:
    def __init__(self, title, author):
        self.title = title
        self.author = author

    def __eq__(self, other):
        if not isinstance(other, Book):
            return False
        return self.title == other.title and self.author == other.author


b1 = Book("Pan Tadeusz", "Mickiewicz")
b2 = Book("Pan Tadeusz", "Mickiewicz")
print(b1 == b2)   # True
```

### `__len__`, `__contains__`, `__iter__`

Te metody pozwalają obiektowi zachowywać się jak kolekcja:

```python
class Library:

    def __init__(self):
        self.books = []

    def add(self, book):
        self.books.append(book)

    def __len__(self):
        return len(self.books)

    def __contains__(self, title):
        return any(b.title == title for b in self.books)

    def __iter__(self):
        return iter(self.books)


lib = Library()
lib.add(Book("Pan Tadeusz", "Mickiewicz"))
lib.add(Book("Lalka", "Prus"))

print(len(lib))                        # 2
print("Lalka" in lib)                  # True
for book in lib:
    print(book)                        # iteracja działa
```

Tak naprawdę napisaliśmy własną kolekcję, która zachowuje się jak natywna lista — bez dziedziczenia po `list`.

### Inne przydatne dunders

| Dunder | Co robi |
|---|---|
| `__init__` | Konstruktor |
| `__str__` | `str(obj)`, `print(obj)` |
| `__repr__` | `repr(obj)`, podgląd w konsoli |
| `__eq__` | `obj1 == obj2` |
| `__lt__`, `__gt__` | `<`, `>` (porządek, sortowanie) |
| `__len__` | `len(obj)` |
| `__contains__` | `x in obj` |
| `__iter__` | `for x in obj` |
| `__getitem__` | `obj[key]` |
| `__add__` | `obj1 + obj2` |
| `__call__` | `obj()` — obiekt staje się wywoływalny |

Dunders to **„hooki" do języka Pythona** — pozwalają Twoim klasom integrować się z natywnymi mechanizmami.


## Łączymy wszystko — minimalny przykład biblioteki

Pokazuje wszystkie filary i kilka dunderów w działaniu:

```python
class Book:

    def __init__(self, title, author, copies):
        self.title = title
        self.author = author
        self._copies = copies          # hermetyzacja

    @property
    def available(self):
        return self._copies > 0

    def borrow(self):
        if not self.available:
            raise ValueError(f"Brak egzemplarzy: {self.title}")
        self._copies -= 1

    def return_copy(self):
        self._copies += 1

    def __str__(self):
        return f"{self.title} — {self.author} (dostępne: {self._copies})"


class User:

    def __init__(self, login, password):
        self.login = login
        self._password = password

    def authenticate(self, password):
        return self._password == password

    def menu(self):
        raise NotImplementedError("Klasy pochodne muszą zaimplementować menu()")


class Reader(User):

    def __init__(self, login, password):
        super().__init__(login, password)
        self.borrowed = []

    def menu(self):
        print(f"Menu czytelnika ({self.login}):")
        print("  1. Przeglądaj katalog")
        print("  2. Wypożycz")
        print("  3. Moje wypożyczenia")


class Librarian(User):

    def menu(self):
        print(f"Menu bibliotekarza ({self.login}):")
        print("  1. Lista wszystkich wypożyczeń")
        print("  2. Prośby o przedłużenie")


# polimorfizm w akcji
users = [
    Reader("anna", "tajne"),
    Librarian("kowalski", "admin"),
]

for u in users:
    u.menu()      # każdy reaguje po swojemu
    print()
```

Co tu się dzieje?

- **Hermetyzacja** — `_copies`, `_password` chronione; `available` jako `@property`.
- **Dziedziczenie** — `Reader` i `Librarian` rozszerzają `User`, używają `super().__init__`.
- **Polimorfizm** — `u.menu()` w pętli zachowuje się inaczej dla każdego typu.
- **Dunder** — `__str__` w `Book` daje czytelny print.

To w mniejszej skali jest dokładnie to, co napiszesz w Części 2 projektu.


## OOP w praktyce — kilka rad

1. **Klasa to rzeczownik, metoda to czasownik** — `Book.borrow()`, nie `book_borrower(book)`.
2. **Nie wszystko musi być klasą** — w Pythonie funkcja jest pełnoprawnym obywatelem. Klasa ma sens, gdy masz **dane + zachowanie razem**.
3. **Używaj hermetyzacji oszczędnie** — w Pythonie pojedyncze `_` zwykle wystarczy. Nie kopiuj bezmyślnie wzorców z Javy.
4. **Każda klasa powinna mieć jeden cel** — jeśli klasa robi za dużo, podziel ją.
5. **Dziedziczenie ≠ kopiowanie** — dziedzicz tylko gdy istnieje prawdziwa relacja „jest typu". `Reader` to `User` — OK. `Library` to `User` — bez sensu.

---

## Praca projektowa

### Część 2 — Programowanie obiektowe (20 pkt)

**Temat:** Biblioteka — refaktoryzacja do OOP i rola bibliotekarza

Przepisz aplikację z Części 1 na wersję obiektową i rozszerz ją o rolę **bibliotekarza**.

**Wymagania dotyczące klas:**

1. **`Book`** — tytuł, autor, łączna liczba sztuk, liczba dostępnych sztuk.
2. **`User`** (klasa bazowa) — login, hasło, rola. Klasy pochodne: **`Reader`** (posiada listę wypożyczonych książek i listę próśb o przedłużenie) oraz **`Librarian`**.
3. **`Library`** — przechowuje kolekcje książek i użytkowników; zawiera metody realizujące logikę biznesową (wyszukiwanie, wypożyczanie, itp.).

**Nowe funkcjonalności (bibliotekarz):**

4. Po zalogowaniu menu jest różne w zależności od roli.
5. **Bibliotekarz — lista wypożyczeń** — wyświetla wszystkie aktualnie wypożyczone książki wraz z loginami użytkowników, którzy je wypożyczyli.
6. **Czytelnik — prośba o przedłużenie** — czytelnik może wysłać prośbę o przedłużenie wybranej wypożyczonej książki (prośba trafia do kolejki).
7. **Bibliotekarz — obsługa próśb** — bibliotekarz widzi listę próśb o przedłużenie i może każdą zaakceptować lub odrzucić.

**Wymagania techniczne:**

- Zastosowanie dziedziczenia (`Reader`, `Librarian` dziedziczą po `User`).
- Zastosowanie hermetyzacji — pola prywatne/chronione tam, gdzie to sensowne, dostęp przez metody lub properties.
- Metoda `__str__` w co najmniej jednej klasie.
- Dane początkowe tworzone jako instancje klas.

**Punktacja (20 pkt):**

| Element | Punkty |
|---|---|
| Klasa `Book` z atrybutami i metodami | 2 |
| Klasa `User` z dziedziczeniem (`Reader`, `Librarian`) | 4 |
| Klasa `Library` z logiką biznesową w metodach | 3 |
| Hermetyzacja i użycie `__str__` | 2 |
| Menu zależne od roli (czytelnik / bibliotekarz) | 2 |
| Widok wypożyczeń dla bibliotekarza (książka + użytkownik) | 3 |
| Prośba o przedłużenie (czytelnik) i jej obsługa (bibliotekarz) | 4 |