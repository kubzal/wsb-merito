# Zajęcia 1

## Paradygmaty programowania

W programowaniu wyróżnia się różne **paradygmaty**, czyli sposoby myślenia o strukturze programu. Trzy bardzo popularne to:

- **programowanie strukturalne**
- **programowanie obiektowe (OOP)**
- **programowanie funkcyjne**

Najłatwiej zrozumieć je przez **porównanie sposobu organizowania kodu**.



## Programowanie strukturalne

To najstarszy z tych trzech stylów. Program jest zorganizowany jako **ciąg instrukcji i funkcji**, które wykonują się krok po kroku.

Podstawowe elementy: zmienne, instrukcje (`if`, `for`, `while`), funkcje — brak obiektów.

Program wygląda jak **przepis kulinarny**: wykonuj instrukcje po kolei.

### Przykład

```python
def calculate_area(width, height):
    return width * height


w = 5
h = 3

area = calculate_area(w, h)
print(area)
```

### Typowe języki

C, Pascal, w dużej części Python.



## Programowanie obiektowe (OOP)

W OOP program organizuje się wokół **obiektów**, które reprezentują rzeczy ze świata rzeczywistego.

Obiekt:
- ma **dane** (atrybuty)
- ma **zachowanie** (metody)

Zamiast funkcji operujących na danych, mamy **obiekty, które coś robią**.

### Przykład

```python
class Dog:

    def __init__(self, name):
        self.name = name

    def bark(self):
        print(self.name + " says woof!")


dog = Dog("Torvi")
dog.bark()
```

Tutaj: `Dog` → klasa, `dog` → obiekt, `name` → atrybut, `bark()` → metoda.

### Cechy OOP

- **encapsulation** – ukrywanie danych
- **inheritance** – dziedziczenie
- **polymorphism** – różne zachowanie tej samej metody

### Typowe języki

Java, C#, C++, Python, Ruby.



## Programowanie funkcyjne

Program składa się głównie z **funkcji**, które:
- nie zmieniają stanu
- zawsze dla tych samych danych dają ten sam wynik

Najważniejsza zasada — **funkcje są "czyste" (pure functions)**:

```
wynik = funkcja(dane)
```

i nic poza tym.

### Przykład — `map`

```python
def add(a, b):
    return a + b


numbers = [1, 2, 3, 4]

result = list(map(lambda x: x * 2, numbers))

print(result)
```

### Przykład — `filter`

```python
numbers = [1, 2, 3, 4, 5, 6]

even = list(filter(lambda x: x % 2 == 0, numbers))

print(even)
```

### Kluczowe idee

- **immutability** (dane się nie zmieniają)
- **functions as first-class objects**
- **map / filter / reduce**

### Typowe języki

Haskell, Lisp, Clojure, Scala, częściowo Python i JavaScript.



## Porównanie paradygmatów

| Paradygmat   | Myślenie o programie               | Pytanie, które zadajesz                              |
| ------------ | ---------------------------------- | ---------------------------------------------------- |
| strukturalny | lista instrukcji                   | Jakie kroki mam wykonać?                             |
| obiektowy    | współpracujące obiekty             | Jakie obiekty istnieją i co potrafią robić?          |
| funkcyjny    | transformacje danych przez funkcje | Jak przepływają dane i jakimi funkcjami je przekształcam? |

### Analogia — restauracja

**Strukturalne** — szef kuchni ma listę kroków:

```
1. pokrój cebulę
2. usmaż cebulę
3. dodaj mięso
4. gotuj
```

**Obiektowe** — masz obiekty: `Chef`, `Oven`, `Pan`, `Dish`, każdy coś robi:

```
chef.cook(dish)
oven.heat()
```

**Funkcyjne** — masz transformacje danych:

```
ingredients → chop() → fry() → mix() → dish
```

### Kiedy który styl ma sens?

**Strukturalne** — proste skrypty, małe programy, nauka podstaw, zadania typu „wejście → przetworzenie → wynik". Najprostsze na start, ale przy dużym projekcie może się zrobić bałagan.

**Obiektowe** — większe aplikacje, systemy z wieloma bytami (User, Order, Product, Invoice), aplikacje webowe, kod rozwijany długo.

**Funkcyjne** — praca na danych, pipeline'y, analiza danych, logika biznesowa, którą chcesz mieć czystą i łatwą do testowania.



## Przykład: Aplikacja do liczenia wartości zamówienia

- mamy produkty (nazwa, cena, liczba sztuk)
- chcemy policzyć łączną wartość zamówienia
- chcemy wypisać podsumowanie

### Podejście strukturalne

Wszystko opiera się na danych w prostych strukturach, funkcjach i instrukcjach krok po kroku.

```python
products = [
    {"name": "Laptop", "price": 3000, "quantity": 1},
    {"name": "Mouse", "price": 100, "quantity": 2},
    {"name": "Keyboard", "price": 200, "quantity": 1},
]


def calculate_item_total(product):
    return product["price"] * product["quantity"]


def calculate_order_total(products):
    total = 0
    for product in products:
        total += calculate_item_total(product)
    return total


def print_order_summary(products):
    print("ORDER SUMMARY")
    for product in products:
        item_total = calculate_item_total(product)
        print(f'{product["name"]}: {product["quantity"]} x {product["price"]} = {item_total}')
    print(f"TOTAL: {calculate_order_total(products)}")


print_order_summary(products)
```

Myślenie: **„weź dane i je przetwórz"** — mam listę danych i funkcje, które coś z nimi robią.

### Podejście obiektowe

Dane i zachowanie łączymy w obiekty: `Product` i `Order`.

```python
class Product:
    def __init__(self, name, price, quantity):
        self.name = name
        self.price = price
        self.quantity = quantity

    def total_price(self):
        return self.price * self.quantity


class Order:
    def __init__(self):
        self.products = []

    def add_product(self, product):
        self.products.append(product)

    def total_amount(self):
        return sum(product.total_price() for product in self.products)

    def print_summary(self):
        print("ORDER SUMMARY")
        for product in self.products:
            print(f"{product.name}: {product.quantity} x {product.price} = {product.total_price()}")
        print(f"TOTAL: {self.total_amount()}")


order = Order()
order.add_product(Product("Laptop", 3000, 1))
order.add_product(Product("Mouse", 100, 2))
order.add_product(Product("Keyboard", 200, 1))

order.print_summary()
```

Myślenie: **„stwórz rzeczy, które umieją coś robić"** — produkt to obiekt, zamówienie to obiekt, każdy ma swoje dane i metody.

### Podejście funkcyjne

Skupiamy się na funkcjach i przekształcaniu danych — bez zmiany stanu.

```python
products = [
    {"name": "Laptop", "price": 3000, "quantity": 1},
    {"name": "Mouse", "price": 100, "quantity": 2},
    {"name": "Keyboard", "price": 200, "quantity": 1},
]


def item_total(product):
    return product["price"] * product["quantity"]


def order_total(products):
    return sum(map(item_total, products))


def order_lines(products):
    return list(
        map(
            lambda p: f'{p["name"]}: {p["quantity"]} x {p["price"]} = {item_total(p)}',
            products
        )
    )


def print_summary(products):
    print("ORDER SUMMARY")
    for line in order_lines(products):
        print(line)
    print(f"TOTAL: {order_total(products)}")


print_summary(products)
```

Myślenie: **„przekształcaj dane przez funkcje"** — dane wpływają do funkcji, funkcje zwracają wynik.



## Python w praktyce — mix paradygmatów

W praktyce większość języków jest **mieszana**. Python to doskonały przykład — w jednym pliku możesz mieć wszystkie trzy style:

- modele i klasy → obiektowo
- operacje na danych → funkcyjnie
- prosty flow programu → strukturalnie

Jako Pythonowiec często już używasz elementów funkcyjnych, nawet jeśli o tym nie myślisz: `map`, `filter`, `sorted(key=...)`, list comprehensions, funkcje przekazywane jako argumenty.

Normalny kod produkcyjny to zazwyczaj **mix** — i to jest w porządku.



## Praca projektowa

### Część 1 — Programowanie strukturalne (20 pkt)

**Temat:** Biblioteka — wersja podstawowa

Napisz konsolową aplikację do obsługi biblioteki wykorzystując **wyłącznie programowanie strukturalne** (funkcje, pętle, instrukcje warunkowe, kolekcje — bez klas).

Dane przechowuj w strukturach takich jak listy, słowniki i listy słowników. Zdefiniuj na sztywno w kodzie co najmniej 5 książek (tytuł, autor, liczba sztuk) oraz 3 użytkowników z hasłami — wszyscy o roli „czytelnik".

**Wymagania funkcjonalne:**

1. **Logowanie** — użytkownik podaje login i hasło; po 3 nieudanych próbach program się kończy.
2. **Menu główne** z opcjami do wyboru (pętla aż do wybrania „Wyloguj").
3. **Przeglądanie katalogu** — wyświetlenie wszystkich książek z autorem i liczbą dostępnych sztuk.
4. **Wypożyczenie książki** — użytkownik podaje tytuł; jeśli są dostępne sztuki, liczba się zmniejsza, a książka trafia na listę wypożyczeń tego użytkownika. Jeśli brak sztuk — komunikat.
5. **Moje wypożyczenia** — wyświetlenie listy książek aktualnie wypożyczonych przez zalogowanego użytkownika.

**Wymagania techniczne:**

- Każda operacja (logowanie, wyświetlanie katalogu, wypożyczanie itd.) musi być wyodrębniona jako osobna funkcja.
- Żadnych klas ani importów zewnętrznych bibliotek.
- Kod powinien być czytelny — sensowne nazwy zmiennych i funkcji, komentarze przy kluczowych fragmentach.

**Punktacja (20 pkt):**

| Element | Punkty |
|---|---|
| Poprawna struktura danych (książki, użytkownicy, wypożyczenia) | 3 |
| Logowanie z walidacją i limitem prób | 3 |
| Menu w pętli z obsługą błędnych wyborów | 2 |
| Przeglądanie katalogu z liczbą dostępnych sztuk | 3 |
| Wypożyczanie z aktualizacją stanu i obsługą braku sztuk | 4 |
| Wyświetlanie wypożyczeń zalogowanego użytkownika | 3 |
| Podział na funkcje i czytelność kodu | 2 |