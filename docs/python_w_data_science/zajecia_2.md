# Zajęcia 2

## Programowanie obiektowe — część 2

> **Poprzednie zajęcia:** nauczyliśmy się tworzyć klasy, definiować `__init__`, atrybuty i metody.
> Dziś idziemy dalej — poznamy cztery ważne koncepty, które sprawiają, że klasy stają się naprawdę potężnym narzędziem.

---

## 1. Dziedziczenie (Inheritance)

### O co chodzi?

Wyobraź sobie, że tworzysz grę i masz różne postacie: wojownika, maga i łucznika. Każda z nich ma imię, punkty życia i potrafi się przedstawić. Ale każda walczy inaczej.

Czy musimy pisać te same atrybuty (`name`, `hp`) i metody (`introduce()`) trzy razy? **Nie!**

Dziedziczenie pozwala stworzyć **klasę bazową** (rodzica) z tym, co wspólne, a potem klasy **potomne** (dzieci), które dziedziczą wszystko po rodzicu i dodają swoje unikalne cechy.

```
        Postać (rodzic)
       /       |       \
  Wojownik    Mag    Łucznik  (dzieci)
```

### Prosty przykład

Zacznijmy od czegoś prostego — zwierzęta:

```python
class Animal:
    """Klasa bazowa — opisuje dowolne zwierzę."""

    def __init__(self, name: str, sound: str):
        self.name = name
        self.sound = sound

    def speak(self):
        return f"{self.name} mówi: {self.sound}!"

    def __str__(self):
        return f"Zwierzę: {self.name}"
```

Teraz stwórzmy klasy potomne. Kluczowe elementy:
- W nawiasie po nazwie klasy podajemy **rodzica**: `class Dog(Animal)`
- `super().__init__(...)` wywołuje `__init__` rodzica — nie musimy powtarzać kodu

```python
class Dog(Animal):
    """Pies — dziedziczy po Animal, dodaje rasę."""

    def __init__(self, name: str, breed: str):
        # Wywołujemy __init__ rodzica (Animal),
        # żeby ustawił name i sound za nas:
        super().__init__(name, sound="Hau")
        # Dodajemy nowy atrybut, którego Animal nie ma:
        self.breed = breed

    def fetch(self):
        return f"{self.name} przynosi piłkę!"


class Cat(Animal):
    """Kot — dziedziczy po Animal, dodaje info czy lubi głaskanie."""

    def __init__(self, name: str, likes_petting: bool = True):
        super().__init__(name, sound="Miau")
        self.likes_petting = likes_petting

    def purr(self):
        if self.likes_petting:
            return f"{self.name} mruczy..."
        return f"{self.name} nie chce być głaskany!"
```

Zobaczmy, jak to działa:

```python
dog = Dog("Burek", breed="Labrador")
cat = Cat("Mruczek", likes_petting=True)

# Metody odziedziczone po rodzicu (Animal):
print(dog.speak())     # Burek mówi: Hau!
print(cat.speak())     # Mruczek mówi: Miau!

# Metody własne (tylko w klasie potomnej):
print(dog.fetch())     # Burek przynosi piłkę!
print(cat.purr())      # Mruczek mruczy...

# Pies NIE ma metody purr(), a kot NIE ma metody fetch()
```

### Co dokładnie robi `super()`?

`super()` to odwołanie do klasy rodzica. Używamy go najczęściej w `__init__`, żeby rodzic mógł ustawić swoje atrybuty:

```python
class Vehicle:
    def __init__(self, brand: str, year: int):
        self.brand = brand
        self.year = year

class Car(Vehicle):
    def __init__(self, brand: str, year: int, doors: int):
        super().__init__(brand, year)  # Vehicle ustawi brand i year
        self.doors = doors             # to jest nowe, tylko dla Car

car = Car("Toyota", 2020, 5)
print(car.brand)   # Toyota   — ustawione przez Vehicle.__init__
print(car.year)    # 2020     — ustawione przez Vehicle.__init__
print(car.doors)   # 5        — ustawione przez Car.__init__
```

### Nadpisywanie metod (override)

Klasa potomna może **nadpisać** metodę rodzica — czyli zdefiniować ją od nowa:

```python
class Vehicle:
    def __init__(self, brand: str):
        self.brand = brand

    def describe(self):
        return f"Pojazd marki {self.brand}"

class Motorcycle(Vehicle):
    def describe(self):
        # Nadpisujemy metodę — zwracamy coś innego
        return f"Motocykl marki {self.brand}"

class Truck(Vehicle):
    def __init__(self, brand: str, capacity_tons: float):
        super().__init__(brand)
        self.capacity_tons = capacity_tons

    def describe(self):
        # Nadpisujemy I rozszerzamy — najpierw wywołujemy rodzica
        base = super().describe()
        return f"{base}, ładowność: {self.capacity_tons}t"

print(Vehicle("Ford").describe())           # Pojazd marki Ford
print(Motorcycle("Yamaha").describe())      # Motocykl marki Yamaha
print(Truck("MAN", 18.0).describe())        # Pojazd marki MAN, ładowność: 18.0t
```

### 🏋️ Zadanie 1 — Dziedziczenie (4 pkt)

Stwórz system opisujący produkty w sklepie internetowym:

1. **(1.5 pkt)** Klasa bazowa `Product`:
   - `__init__(self, name: str, price: float)` — ustawia nazwę i cenę
   - `display(self)` — zwraca string `"Produkt: {name}, cena: {price} zł"`
   - `apply_discount(self, percent: float)` — obniża cenę o podany procent

2. **(1.5 pkt)** Klasa `Book(Product)`:
   - dodatkowy atrybut `author: str`
   - nadpisz `display()` — zwraca `"Książka: {name} (autor: {author}), cena: {price} zł"`

3. **(1 pkt)** Klasa `Electronics(Product)`:
   - dodatkowy atrybut `warranty_months: int`
   - nadpisz `display()` — dodaj info o gwarancji

```python
# Oczekiwane zachowanie:
book = Book("Pan Tadeusz", 29.99, author="Adam Mickiewicz")
print(book.display())   # Książka: Pan Tadeusz (autor: Adam Mickiewicz), cena: 29.99 zł
book.apply_discount(10)
print(book.display())   # Książka: Pan Tadeusz (autor: Adam Mickiewicz), cena: 26.99 zł

laptop = Electronics("Laptop", 3999.0, warranty_months=24)
print(laptop.display()) # Elektronika: Laptop, cena: 3999.0 zł, gwarancja: 24 mies.
```

---

## 2. Polimorfizm (Polymorphism)

### O co chodzi?

Polimorfizm to po grecku "wiele form". W programowaniu oznacza, że **ta sama operacja działa różnie** w zależności od obiektu.

Już to widzieliśmy! Funkcja `len()` działa na stringach, listach, słownikach — za każdym razem robi co innego, ale interfejs jest ten sam:

```python
print(len("Ala"))          # 3 — liczy znaki
print(len([1, 2, 3, 4]))  # 4 — liczy elementy
print(len({"a": 1}))      # 1 — liczy klucze
```

To jest polimorfizm! Jedna nazwa (`len`), wiele zachowań.

### Polimorfizm z naszymi klasami

Wróćmy do przykładu z dziedziczeniem. Mamy klasy `Dog` i `Cat`, obie mają metodę `speak()`. Możemy napisać kod, który **nie wie** z jakim zwierzęciem ma do czynienia, a i tak działa:

```python
def present_animal(animal):
    """Ta funkcja nie wie, czy dostaje psa czy kota.
    Wie tylko, że obiekt ma metodę speak()."""
    print(f"Oto {animal.name}!")
    print(animal.speak())
    print("---")

# Działa z psem:
present_animal(Dog("Burek", "Labrador"))
# Oto Burek!
# Burek mówi: Hau!

# Działa z kotem:
present_animal(Cat("Mruczek"))
# Oto Mruczek!
# Mruczek mówi: Miau!

# Działa z KAŻDYM obiektem, który ma name i speak()!
```

To jest ogromna siła — możemy pisać **kod generyczny**, który operuje na różnych typach obiektów.

### Polimorfizm w praktyce — lista różnych obiektów

```python
class Circle:
    def __init__(self, radius: float):
        self.radius = radius

    def area(self):
        return 3.14159 * self.radius ** 2

    def describe(self):
        return f"Koło o promieniu {self.radius}"

class Rectangle:
    def __init__(self, width: float, height: float):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

    def describe(self):
        return f"Prostokąt {self.width}x{self.height}"

class Triangle:
    def __init__(self, base: float, height: float):
        self.base = base
        self.height = height

    def area(self):
        return 0.5 * self.base * self.height

    def describe(self):
        return f"Trójkąt o podstawie {self.base} i wysokości {self.height}"


# Tworzymy listę RÓŻNYCH figur:
shapes = [Circle(5), Rectangle(3, 4), Triangle(6, 3)]

# Iterujemy — nie obchodzi nas, jaki to typ!
# Każdy obiekt "wie", jak obliczyć swoje pole:
for shape in shapes:
    print(f"{shape.describe()} → pole = {shape.area()}")

# Koło o promieniu 5 → pole = 78.53975
# Prostokąt 3x4 → pole = 12
# Trójkąt o podstawie 6 i wysokości 3 → pole = 9.0
```

### Duck typing — "jeśli kwacze jak kaczka..."

Python nie sprawdza typu obiektu — sprawdza, czy **ma odpowiednią metodę**. To tzw. duck typing:

> *"Jeśli chodzi jak kaczka i kwacze jak kaczka — to jest kaczka."*

```python
class Printer:
    """Drukarka — nie dziedziczy po żadnej figurze!"""
    def describe(self):
        return "Drukarka HP LaserJet"

    def area(self):
        return 0.3 * 0.4  # wymiary w metrach

# Drukarka NIE jest figurą geometryczną, ale ma metody
# describe() i area() — więc zadziała z tym samym kodem!
printer = Printer()
print(f"{printer.describe()} → pole = {printer.area()}")
# Drukarka HP LaserJet → pole = 0.12
```

### Klasy abstrakcyjne — wymuszanie interfejsu

Czasem chcemy **zmusić** programistę do zaimplementowania konkretnej metody. Służy do tego moduł `abc` (Abstract Base Class):

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    """Klasa abstrakcyjna — nie można stworzyć jej instancji!"""

    @abstractmethod
    def area(self):
        """Każda podklasa MUSI zaimplementować tę metodę."""
        pass

    @abstractmethod
    def describe(self):
        pass

# shape = Shape()  # TypeError! Nie można stworzyć Shape() bezpośrednio.

class Square(Shape):
    def __init__(self, side: float):
        self.side = side

    def area(self):            # MUSIMY to zdefiniować
        return self.side ** 2

    def describe(self):        # MUSIMY to zdefiniować
        return f"Kwadrat o boku {self.side}"

sq = Square(4)
print(sq.area())       # 16
print(sq.describe())   # Kwadrat o boku 4
```

Jeśli zapomnimy zaimplementować `area()` lub `describe()` w klasie potomnej, Python wyrzuci błąd. To świetny sposób na pilnowanie, żeby każda klasa "trzymała się kontraktu".

### 🏋️ Zadanie 2 — Polimorfizm (4 pkt)

Stwórz system powiadomień:

1. **(1 pkt)** Klasa abstrakcyjna `Notification(ABC)` z metodami:
   - `send(self, message: str) -> str` — abstrakcyjna
   - `__str__` — zwraca typ powiadomienia (np. `"EmailNotification"`)

2. **(1 pkt)** Klasa `EmailNotification` — `send()` zwraca `"Wysyłam email: {message}"`

3. **(1 pkt)** Klasa `SMSNotification` — `send()` zwraca `"Wysyłam SMS: {message}"`

4. **(1 pkt)** Napisz funkcję `send_all(notifications: list, message: str)`, która iteruje po liście i wywołuje `send()` na każdym obiekcie. Przetestuj:

```python
channels = [EmailNotification(), SMSNotification()]
send_all(channels, "Twoje zamówienie zostało wysłane!")

# Oczekiwany wynik:
# Wysyłam email: Twoje zamówienie zostało wysłane!
# Wysyłam SMS: Twoje zamówienie zostało wysłane!
```

---

## 3. Hermetyzacja (Encapsulation)

### O co chodzi?

Hermetyzacja to **ukrywanie wnętrza obiektu** przed światem zewnętrznym. Obiekt sam decyduje, co pokazuje na zewnątrz, a co trzyma "pod maską".

Analogia z życia: **bankomat**. Wkładasz kartę, wpisujesz PIN, dostajesz pieniądze. Nie wiesz (i nie musisz wiedzieć), jak bankomat komunikuje się z bankiem, jak liczy banknoty, jak weryfikuje PIN. Widzisz tylko **interfejs** (ekran, klawiatura), a **implementacja** jest ukryta.

W programowaniu hermetyzacja oznacza:
- Niektóre atrybuty i metody są **prywatne** — nie powinno się ich używać z zewnątrz.
- Dostęp do danych odbywa się przez **kontrolowane metody**, które mogą walidować dane.

### Konwencje nazewnictwa w Pythonie

Python nie ma "twardego" systemu prywatności jak Java czy C#. Zamiast tego stosujemy **konwencje**:

```python
class BankAccount:

    def __init__(self, owner: str, balance: float):
        self.owner = owner           # publiczny — każdy może odczytać
        self._transactions = []      # chroniony — "proszę, nie ruszaj tego z zewnątrz"
        self.__balance = balance     # prywatny — Python utrudnia dostęp (name mangling)
```

| Zapis | Znaczenie | Przykład |
|-------|-----------|---------|
| `self.name` | **Publiczny** — używaj swobodnie | `self.owner` |
| `self._name` | **Chroniony** — konwencja "nie ruszaj" | `self._transactions` |
| `self.__name` | **Prywatny** — Python zmienia nazwę na `_ClassName__name` | `self.__balance` |

**Ważne:** podwójny podkreślnik (`__`) nie czyni atrybutu naprawdę prywatnym — to bardziej "proszę, naprawdę nie ruszaj". Python zmienia nazwę atrybutu (tzw. *name mangling*), ale technicznie wciąż da się do niego dotrzeć. Chodzi o **intencję i dyscyplinę**, nie o blokadę.

### Przykład z kontrolowanym dostępem

```python
class BankAccount:

    def __init__(self, owner: str, initial_balance: float = 0):
        self.owner = owner
        self.__balance = initial_balance    # prywatny!
        self._history = []                  # chroniony

    def deposit(self, amount: float):
        """Kontrolowane wpłacanie — z walidacją."""
        if amount <= 0:
            raise ValueError("Kwota musi być dodatnia!")
        self.__balance += amount
        self._history.append(f"+{amount}")

    def withdraw(self, amount: float):
        """Kontrolowane wypłacanie — z walidacją."""
        if amount <= 0:
            raise ValueError("Kwota musi być dodatnia!")
        if amount > self.__balance:
            raise ValueError("Brak środków na koncie!")
        self.__balance -= amount
        self._history.append(f"-{amount}")

    def get_balance(self):
        """Bezpieczne odczytanie salda — bez możliwości modyfikacji."""
        return self.__balance

    def get_history(self):
        """Zwracamy KOPIĘ historii — oryginał jest bezpieczny."""
        return self._history.copy()

account = BankAccount("Anna", 1000)
account.deposit(500)
account.withdraw(200)
print(account.get_balance())     # 1300
print(account.get_history())     # ['+500', '-200']

# Nie możemy "oszukać" — te linijki nie zadziałają poprawnie:
# account.__balance = 999999     # To NIE zmieni prawdziwego salda!
# account.__balance               # AttributeError!
```

### `@property` — elegancki sposób na gettery i settery

Zamiast pisać `get_balance()` i `set_balance()`, Python oferuje dekorator `@property`. Dzięki niemu atrybut **wygląda jak zwykły atrybut**, ale pod spodem jest metoda z walidacją:

```python
class Student:

    def __init__(self, name: str, grade: float):
        self.name = name
        self._grade = grade

    @property
    def grade(self):
        """Getter — wywoływany gdy piszemy: student.grade"""
        return self._grade

    @grade.setter
    def grade(self, new_grade: float):
        """Setter — wywoływany gdy piszemy: student.grade = 4.5"""
        if not (2.0 <= new_grade <= 5.0):
            raise ValueError(f"Ocena musi być między 2.0 a 5.0, podano: {new_grade}")
        self._grade = new_grade

    @property
    def is_passing(self):
        """Getter bez settera — atrybut tylko do odczytu."""
        return self._grade >= 3.0


student = Student("Jan", 4.0)

# Używamy jak zwykłego atrybutu, ale pod spodem działa metoda:
print(student.grade)        # 4.0  (wywołuje getter)
student.grade = 4.5         # OK   (wywołuje setter z walidacją)
print(student.grade)        # 4.5
print(student.is_passing)   # True (getter bez settera — read-only)

# student.grade = 6.0       # ValueError! Walidacja działa!
# student.is_passing = False # AttributeError! Nie ma settera!
```

### 🏋️ Zadanie 3 — Hermetyzacja (4 pkt)

Stwórz klasę `Playlist` do zarządzania playlistą muzyczną:

1. **(1 pkt)** Prywatny atrybut `__songs: list` (lista piosenek — stringów) i publiczny `name: str`.
2. **(1 pkt)** Metody `add_song(title: str)` i `remove_song(title: str)`:
   - `add_song` — dodaje piosenkę, ale rzuca `ValueError` jeśli już istnieje na liście
   - `remove_song` — usuwa piosenkę, rzuca `ValueError` jeśli jej nie ma
3. **(1 pkt)** `@property song_count` — zwraca liczbę piosenek (bez settera — read-only).
4. **(1 pkt)** Metoda `get_songs()` — zwraca **kopię** listy piosenek (żeby ktoś z zewnątrz nie mógł zmodyfikować oryginału).

```python
playlist = Playlist("Moje ulubione")
playlist.add_song("Bohemian Rhapsody")
playlist.add_song("Hotel California")
playlist.add_song("Smells Like Teen Spirit")

print(playlist.song_count)     # 3
print(playlist.get_songs())    # ['Bohemian Rhapsody', 'Hotel California', 'Smells Like Teen Spirit']

playlist.remove_song("Hotel California")
print(playlist.song_count)     # 2

# playlist.add_song("Bohemian Rhapsody")  # ValueError: Piosenka już istnieje!
# playlist.song_count = 10                # AttributeError! Read-only!
```

---

## 4. Dataclasses — klasy do przechowywania danych

### Problem

Często tworzymy klasy, które głównie **przechowują dane** — np. informacje o użytkowniku, konfigurację, wynik. Za każdym razem musimy pisać ten sam powtarzalny kod:

```python
# Bez dataclass — dużo pisania:
class PersonOld:
    def __init__(self, name: str, age: int, city: str):
        self.name = name
        self.age = age
        self.city = city

    def __repr__(self):
        return f"PersonOld(name='{self.name}', age={self.age}, city='{self.city}')"

    def __eq__(self, other):
        return (self.name == other.name and
                self.age == other.age and
                self.city == other.city)
```

To jest nudne i łatwo o literówkę. Rozwiązanie? **Dataclasses!**

### Podstawy

Moduł `dataclasses` jest wbudowany w Pythona (od wersji 3.7). Wystarczy dodać dekorator `@dataclass`, a Python **automatycznie wygeneruje** `__init__`, `__repr__` i `__eq__`:

```python
from dataclasses import dataclass

@dataclass
class Person:
    name: str
    age: int
    city: str

# To wszystko! Python wygenerował za nas __init__, __repr__ i __eq__.

p1 = Person("Anna", 25, "Warszawa")
p2 = Person("Anna", 25, "Warszawa")
p3 = Person("Jan", 30, "Kraków")

print(p1)           # Person(name='Anna', age=25, city='Warszawa')
print(p1 == p2)     # True  — porównuje po wartościach, nie po id!
print(p1 == p3)     # False
```

### Wartości domyślne

Tak jak w zwykłych funkcjach — atrybuty z domyślną wartością muszą być **na końcu**:

```python
@dataclass
class Config:
    name: str                     # wymagany — bez wartości domyślnej
    debug: bool = False           # opcjonalny — domyślnie False
    max_retries: int = 3          # opcjonalny — domyślnie 3
    timeout: float = 30.0         # opcjonalny — domyślnie 30 sekund

# Można podać tylko to, co chcemy zmienić:
cfg = Config("moja-aplikacja")
print(cfg)  # Config(name='moja-aplikacja', debug=False, max_retries=3, timeout=30.0)

cfg2 = Config("test", debug=True, timeout=5.0)
print(cfg2) # Config(name='test', debug=True, max_retries=3, timeout=5.0)
```

### Uwaga na listy i słowniki jako wartości domyślne!

W Pythonie jest pułapka: jeśli użyjemy mutowalnego obiektu (listy, słownika) jako wartości domyślnej, **wszystkie instancje będą współdzielić ten sam obiekt!** Dataclasses rozwiązują to przez `field(default_factory=...)`:

```python
from dataclasses import dataclass, field

@dataclass
class ShoppingList:
    owner: str
    items: list = field(default_factory=list)    # każda instancja dostaje NOWĄ listę
    notes: dict = field(default_factory=dict)    # każda instancja dostaje NOWY słownik

s1 = ShoppingList("Anna")
s2 = ShoppingList("Jan")

s1.items.append("mleko")
print(s1.items)  # ['mleko']
print(s2.items)  # []  — OK! Każdy ma swoją listę
```

### `__post_init__` — walidacja po utworzeniu

Jeśli chcemy sprawdzić poprawność danych po stworzeniu obiektu:

```python
@dataclass
class Rectangle:
    width: float
    height: float

    def __post_init__(self):
        """Wywoływana automatycznie PO __init__."""
        if self.width <= 0 or self.height <= 0:
            raise ValueError("Wymiary muszą być dodatnie!")

    @property
    def area(self):
        return self.width * self.height

r = Rectangle(5, 3)
print(r.area)           # 15

# Rectangle(-1, 3)      # ValueError: Wymiary muszą być dodatnie!
```

### `frozen=True` — niemutowalny obiekt

Czasem chcemy, żeby obiektu **nie dało się zmienić** po utworzeniu:

```python
@dataclass(frozen=True)
class Color:
    r: int
    g: int
    b: int

red = Color(255, 0, 0)
print(red)          # Color(r=255, g=0, b=0)

# red.r = 100       # FrozenInstanceError! Nie można zmieniać!
```

### 🏋️ Zadanie 4 — Dataclasses (4 pkt)

1. **(2 pkt)** Stwórz `@dataclass` o nazwie `Movie` z polami:
   - `title: str`
   - `director: str`
   - `year: int`
   - `genres: list[str]` (domyślnie pusta lista — użyj `field(default_factory=list)`)
   - `rating: float` (domyślnie `0.0`)
   - W `__post_init__` sprawdź, że `year >= 1888` (pierwszy film w historii!) i `0.0 <= rating <= 10.0`.

2. **(2 pkt)** Stwórz `@dataclass(frozen=True)` o nazwie `Address` z polami: `street: str`, `city: str`, `zip_code: str`. Następnie stwórz `@dataclass` o nazwie `Contact` z polami `name: str`, `email: str`, `address: Address`. Dodaj `@property full_info`, który zwraca string `"{name} ({email}), {city}"`.

```python
# Test Movie:
movie = Movie("Incepcja", "Christopher Nolan", 2010, ["Sci-Fi", "Thriller"], 8.8)
print(movie)
# Movie(title='Incepcja', director='Christopher Nolan', year=2010, ...)

# Movie("Test", "Test", 1500, rating=11.0)  # ValueError!

# Test Contact:
addr = Address("Marszałkowska 1", "Warszawa", "00-001")
contact = Contact("Anna Kowalska", "anna@email.com", addr)
print(contact.full_info)   # Anna Kowalska (anna@email.com), Warszawa
```

---

## 5. Pydantic — walidacja danych na poważnie

### Problem, który Pydantic rozwiązuje

`Dataclasses` są świetne, ale mają jedną słabość — **nie sprawdzają typów w runtime**:

```python
from dataclasses import dataclass

@dataclass
class User:
    name: str
    age: int

# To nie wyrzuci błędu — mimo że age powinien być int!
user = User(name="Anna", age="dwadzieścia pięć")
print(user.age)   # "dwadzieścia pięć" — string, nie int!
```

**Pydantic** rozwiązuje ten problem — aktywnie **sprawdza i konwertuje typy**:

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int

# Pydantic próbuje skonwertować "25" na int — i udaje mu się:
user = User(name="Anna", age="25")
print(user.age)        # 25 (int, nie string!)
print(type(user.age))  # <class 'int'>

# Ale "dwadzieścia pięć" nie da się skonwertować:
# User(name="Anna", age="dwadzieścia pięć")  # ValidationError!
```

> **Instalacja:** `pip install pydantic`

### Podstawy: `BaseModel`

Zamiast `@dataclass` dziedziczymy po `BaseModel`:

```python
from pydantic import BaseModel
from typing import Optional

class Product(BaseModel):
    name: str
    price: float
    quantity: int = 0
    description: Optional[str] = None   # może być None

p = Product(name="Kawa", price="12.99", quantity="5")
print(p)
# name='Kawa' price=12.99 quantity=5 description=None

# Pydantic automatycznie skonwertował stringi "12.99" i "5" na float i int!
```

### `Field` — dodatkowe ograniczenia

```python
from pydantic import BaseModel, Field

class Product(BaseModel):
    name: str = Field(min_length=1, max_length=100)          # długość stringa
    price: float = Field(gt=0, description="Cena w złotych")  # gt = greater than
    quantity: int = Field(ge=0, le=10000, default=0)           # ge = greater or equal

# Product(name="", price=10.0)       # ValidationError! name za krótkie
# Product(name="Kawa", price=-5.0)   # ValidationError! price musi być > 0
# Product(name="Kawa", price=10.0, quantity=-1)  # ValidationError! quantity >= 0
```

### Własne walidatory

Gdy potrzebujemy bardziej zaawansowanej logiki:

```python
from pydantic import BaseModel, Field, field_validator

class UserRegistration(BaseModel):
    username: str = Field(min_length=3, max_length=20)
    email: str
    age: int = Field(ge=13, le=120)
    password: str = Field(min_length=8)

    @field_validator("email")
    @classmethod
    def validate_email(cls, v: str) -> str:
        if "@" not in v or "." not in v:
            raise ValueError("Niepoprawny format emaila!")
        return v.lower()    # normalizacja — zawsze małe litery

    @field_validator("username")
    @classmethod
    def validate_username(cls, v: str) -> str:
        if not v.isalnum():
            raise ValueError("Username może zawierać tylko litery i cyfry!")
        return v

# OK:
user = UserRegistration(
    username="jankowalski",
    email="Jan@Email.COM",
    age=25,
    password="bezpieczne123"
)
print(user.email)   # jan@email.com — znormalizowany!

# Błędy:
# UserRegistration(username="ab", email="bad", age=10, password="short")
# → ValidationError z LISTĄ wszystkich błędów naraz!
```

### Zagnieżdżone modele i serializacja

Modele Pydantic mogą zawierać inne modele — i wszystko jest walidowane:

```python
from pydantic import BaseModel, Field
from typing import Optional

class Address(BaseModel):
    street: str
    city: str
    zip_code: str = Field(pattern=r"^\d{2}-\d{3}$")  # regex: format "00-000"

class Person(BaseModel):
    name: str
    age: int = Field(ge=0)
    address: Optional[Address] = None

# Pydantic parsuje zagnieżdżone dicty automatycznie:
data = {
    "name": "Anna",
    "age": 25,
    "address": {
        "street": "Marszałkowska 1",
        "city": "Warszawa",
        "zip_code": "00-001"
    }
}

person = Person(**data)
print(person.address.city)           # Warszawa

# Konwersja na dict i JSON:
print(person.model_dump())              # → dict
print(person.model_dump_json(indent=2)) # → ładny JSON string
```

### Dataclasses vs Pydantic — kiedy co?

| | `dataclasses` | `pydantic` |
|---|---|---|
| **Sprawdza typy?** | Nie | Tak |
| **Konwertuje typy?** | Nie | Tak (`"25"` → `25`) |
| **Eksport do JSON?** | Ręcznie | `model_dump_json()` |
| **Instalacja** | Wbudowany | `pip install pydantic` |
| **Kiedy używać?** | Proste struktury wewnętrzne | Dane z zewnątrz, formularze, API |

### 🏋️ Zadanie 5 — Pydantic (4 pkt)

Stwórz system rejestracji na wydarzenie:

1. **(1.5 pkt)** Model `Participant(BaseModel)`:
   - `first_name: str` — min. 2 znaki
   - `last_name: str` — min. 2 znaki
   - `email: str` — walidator sprawdzający, czy zawiera `@` i `.`
   - `age: int` — między 16 a 99

2. **(1.5 pkt)** Model `Event(BaseModel)`:
   - `name: str` — min. 3 znaki
   - `max_participants: int` — między 1 a 1000
   - `participants: list[Participant]` (domyślnie pusta lista)
   - Walidator: jeśli `len(participants) > max_participants`, rzuć błąd

3. **(1 pkt)** Przetestuj — stwórz `Event` z kilkoma uczestnikami, wydrukuj `model_dump_json(indent=2)`. Pokaż co się stanie przy niepoprawnych danych (zły email, za młody uczestnik, za wielu uczestników).

```python
event = Event(
    name="Warsztaty Python",
    max_participants=2,
    participants=[
        Participant(first_name="Anna", last_name="Kowalska",
                   email="anna@example.com", age=25),
        Participant(first_name="Jan", last_name="Nowak",
                   email="jan@example.com", age=30),
    ]
)
print(event.model_dump_json(indent=2))
```

---

## Podsumowanie punktacji

| Zadanie | Temat | Punkty |
|---------|-------|--------|
| 1 | Dziedziczenie | 4 |
| 2 | Polimorfizm | 4 |
| 3 | Hermetyzacja | 4 |
| 4 | Dataclasses | 4 |
| 5 | Pydantic | 4 |
| **Suma** | | **20** |

---

## Materiały dodatkowe

- [Python docs — Klasy (tutorial)](https://docs.python.org/3/tutorial/classes.html)
- [Python docs — `dataclasses`](https://docs.python.org/3/library/dataclasses.html)
- [Python docs — `abc`](https://docs.python.org/3/library/abc.html)
- [Pydantic docs (v2)](https://docs.pydantic.dev/latest/)
- [Real Python — OOP in Python](https://realpython.com/python3-object-oriented-programming/)