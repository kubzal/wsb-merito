# Zajęcia 1

## Powtórzenie podstaw Pythona i wstęp do klas

Pierwsze zajęcia zaczynamy od krótkiego przypomnienia podstaw Pythona — typów danych, list, słowników, pętli i funkcji — a następnie przechodzimy do **klas**, które są punktem wyjścia do całego dalszego kursu.

Po dzisiejszych zajęciach będziesz w stanie:

- swobodnie operować na podstawowych typach i strukturach danych Pythona,
- iterować po kolekcjach na trzy różne sposoby (`for`, `for` + `range`, `while`),
- rozróżnić słownik od seta i wiedzieć, kiedy używać którego,
- zdefiniować własną klasę z konstruktorem i metodami,
- użyć metody specjalnej `__str__`, żeby Twój obiekt ładnie się drukował.

---

## Typy podstawowe

Cztery typy, których używa się najczęściej:

```python
liczby_calkowite = 100         # int
liczby_rzeczywiste = 3.14      # float
tekstowy = "Tekst tekst"       # str
logiczny = True                # bool
```

Typ zmiennej sprawdzimy funkcją `type()`:

```python
zmienna_a = 1
print(zmienna_a, type(zmienna_a))   # 1 <class 'int'>
```

### Typowanie dynamiczne

Python jest typowany **dynamicznie** — zmienna nie ma stałego typu. Tej samej zmiennej można w kolejnej linii przypisać wartość zupełnie innego typu:

```python
zmienna_a = 1
print(zmienna_a, type(zmienna_a))   # 1 <class 'int'>

zmienna_a = 1.5
print(zmienna_a, type(zmienna_a))   # 1.5 <class 'float'>
```

To wygodne, ale w większych projektach bywa źródłem błędów — dlatego później w kursie poznamy **type hints** i **dataclasses**, które przywracają trochę dyscypliny.

---

## Listy

Lista to **uporządkowana kolekcja** elementów. W Pythonie nie wymusza jednego typu — może mieszać wszystko ze wszystkim, łącznie z innymi listami:

```python
lista_a = [1, 2, 3]
lista_b = [1, 2.5, "tekst", -1, True, ["inny tekst", True]]
```

Pustą listę tworzymy tak:

```python
lista_a = []          # zapis "literałowy"
lista_b = list()      # konstruktor — to samo
```

Dostęp do elementu po **indeksie** (od zera):

```python
print(lista_b[1])     # 2.5
```

### Iteracja po liście

Trzy najczęstsze sposoby:

**1. `for` przez element**

```python
for element in lista_b:
    print(element, type(element))
```

Najczystszy zapis — używamy, gdy potrzebujemy tylko wartości.

**2. `for` z `range()` przez indeks**

```python
for i in range(0, len(lista_b)):
    print(lista_b[i], type(lista_b[i]))
```

Przydatne, gdy potrzebujemy znać indeks (np. żeby modyfikować element listy w miejscu).

**3. `while`**

```python
i = 0
while i < len(lista_b):
    print(lista_b[i], type(lista_b[i]))
    i += 1   # to samo co i = i + 1
```

Używamy, gdy nie wiemy z góry, ile razy pętla się wykona — czekamy na warunek.

---

## Słowniki (i set)

**Słownik (dict)** to kolekcja par **klucz–wartość**:

```python
slownik_a = {"imie": "Jakub", "nazwisko": "Zalewski", "miasto": "Warszawa"}
print(slownik_a["imie"])   # Jakub
```

Pusty słownik tworzymy przez `{}` lub `dict()`.

### Pułapka — `{}` to nie zawsze słownik

Nawiasy klamrowe **bez par klucz–wartość** to **set** (zbiór), a nie słownik:

```python
to_jest_set = {1, 2, 3, 3}
print(to_jest_set)         # {1, 2, 3}  — duplikaty znikają
print(type(to_jest_set))   # <class 'set'>

pusty_set = set()           # pusty set
pusty_dict = {}              # pusty słownik
```

Set to zbiór unikalnych wartości — przyda się, gdy chcemy zdeduplikować dane lub szybko sprawdzić przynależność (`x in zbior`).

### Zagnieżdżone struktury danych

Słowniki i listy łączą się dowolnie, więc można nimi zamodelować bardziej złożone dane:

```python
ksiazka_1 = {"tytul": "Harry Potter i kamień filozoficzny", "autor": "J.K. Rowling"}
ksiazka_2 = {"tytul": "Władca Pierścieni", "autor": "J.R.R. Tolkien"}

biblioteka = {
    "ksiazki": [ksiazka_1, ksiazka_2],
    "czytelnicy": ["Jan Kowalski", "Elżbieta Nowak"]
}

for ksiazka in biblioteka["ksiazki"]:
    print(f"Tytuł: '{ksiazka['tytul']}', Autor: {ksiazka['autor']}")
```

To działa, ale ma wady — klucze trzeba pisać jako stringi (łatwo o literówkę), IDE nie podpowiada pól, brak walidacji, brak metod operujących na danych. Za chwilę zobaczymy, że **lepszym narzędziem do tego są klasy**.

---

## Funkcje

Funkcja to nazwany fragment kodu, który można wywoływać wielokrotnie:

```python
def suma(a, b):
    return a + b

print(suma(1, 2))   # 3
```

Słowo kluczowe `def`, nazwa, parametry w nawiasach, dwukropek, blok kodu. `return` zwraca wartość — bez niego funkcja zwraca `None`.

---

## Klasy — wstęp do OOP

Wracamy do biblioteki. Zamiast modelować książkę słownikiem, zdefiniujemy **klasę** — szablon opisujący, "co to znaczy być książką":

```python
class Ksiazka:

    def __init__(self, tytul, autor):
        self.tytul = tytul
        self.autor = autor
```

Co tu się dzieje:

- `class Ksiazka:` — definicja klasy. Konwencjonalnie nazwy klas piszemy w **PascalCase** (każde słowo z dużej litery).
- `__init__` — **konstruktor**, metoda uruchamiana automatycznie przy tworzeniu nowego obiektu.
- `self` — referencja do **konkretnego obiektu**, na którym wywołujemy metodę. Pierwszy parametr każdej metody — Python przekazuje go automatycznie.
- `self.tytul = tytul` — zapisanie wartości parametru jako atrybut tego konkretnego obiektu.

Tworzenie obiektu (instancji klasy) wygląda jak wywołanie funkcji o nazwie klasy:

```python
ksiazka_1 = Ksiazka(tytul="Harry Potter i komnata tajemnic", autor="J.K. Rowling")

print(ksiazka_1.tytul)    # Harry Potter i komnata tajemnic
print(ksiazka_1.autor)    # J.K. Rowling
print(type(ksiazka_1))    # <class '__main__.Ksiazka'>
```

### Metody

Funkcje zdefiniowane wewnątrz klasy nazywamy **metodami**. Pierwszy parametr to zawsze `self`:

```python
class Ksiazka:

    def __init__(self, tytul, autor):
        self.tytul = tytul
        self.autor = autor

    def pokaz_tytul_i_autora(self):
        print("Tytuł:", self.tytul, "Autor:", self.autor)


ksiazka_1 = Ksiazka("Solaris", "Stanisław Lem")
ksiazka_1.pokaz_tytul_i_autora()
# Tytuł: Solaris Autor: Stanisław Lem
```

Wywołanie `ksiazka_1.pokaz_tytul_i_autora()` Python tłumaczy pod spodem na `Ksiazka.pokaz_tytul_i_autora(ksiazka_1)`. Stąd `self` jako pierwszy parametr — to **ten obiekt**, na którym wywołaliśmy metodę.

### Metoda specjalna `__str__`

`__str__` to **metoda specjalna** (tzw. *dunder*, od *double underscore*), która mówi Pythonowi, jak obiekt ma się "drukować" przez `print()`:

```python
class Ksiazka:

    def __init__(self, tytul, autor):
        self.tytul = tytul
        self.autor = autor

    def __str__(self):
        return f"Tytuł: {self.tytul}, Autor: {self.autor}"


ksiazka_1 = Ksiazka("Solaris", "Stanisław Lem")
print(ksiazka_1)
# Tytuł: Solaris, Autor: Stanisław Lem
```

Bez `__str__` `print(ksiazka_1)` wypisałby coś w stylu `<__main__.Ksiazka object at 0x7f...>` — adres w pamięci. Z `__str__` mamy czytelną reprezentację obiektu.

### Walidacja w konstruktorze

Konstruktor to dobre miejsce, żeby od razu sprawdzić, czy dane wejściowe mają sens. Jeśli nie — rzucamy `ValueError`:

```python
class Ksiazka:

    def __init__(self, tytul, autor, rok):
        if not tytul or not tytul.strip():
            raise ValueError("Tytuł nie może być pusty.")
        if not isinstance(rok, int):
            raise ValueError("Rok musi być liczbą całkowitą.")

        self.tytul = tytul.strip()
        self.autor = autor
        self.rok = rok
```

Dzięki temu **nie da się stworzyć obiektu w niepoprawnym stanie**. Błąd wyłapujemy w momencie konstrukcji, a nie pół godziny później przy pierwszym użyciu, kiedy ciężko już zrozumieć, skąd wziął się pusty tytuł.

---

## Łączymy wszystko — System Biblioteki

Mając klasę `Ksiazka`, zbudujmy drugą klasę — `Biblioteka` — która **przechowuje listę książek** i operuje na nich. Pokazuje to, jak klasy ze sobą współpracują.

```python
class Ksiazka:

    def __init__(self, tytul, autor, rok):
        if not tytul or not tytul.strip():
            raise ValueError("Tytuł nie może być pusty.")
        self.tytul = tytul.strip()
        self.autor = autor
        self.rok = rok
        self.dostepna = True

    def wypozycz(self):
        if not self.dostepna:
            return f'"{self.tytul}" jest już wypożyczona.'
        self.dostepna = False
        return f'Wypożyczono: "{self.tytul}".'

    def zwroc(self):
        if self.dostepna:
            return f'"{self.tytul}" nie jest wypożyczona.'
        self.dostepna = True
        return f'Zwrócono: "{self.tytul}".'

    def __str__(self):
        status = "dostępna" if self.dostepna else "wypożyczona"
        return f"{self.tytul} - {self.autor} ({self.rok}) [{status}]"


class Biblioteka:

    def __init__(self, nazwa="Biblioteka"):
        self.nazwa = nazwa
        self.ksiazki = []

    def dodaj(self, ksiazka):
        self.ksiazki.append(ksiazka)

    def szukaj(self, fraza):
        fraza = fraza.lower()
        return [k for k in self.ksiazki
                if fraza in k.tytul.lower() or fraza in k.autor.lower()]

    def dostepne(self):
        return [k for k in self.ksiazki if k.dostepna]

    def __str__(self):
        if not self.ksiazki:
            return f"{self.nazwa}: brak książek."
        lines = [f"{self.nazwa} ({len(self.ksiazki)} książek):"]
        for i, k in enumerate(self.ksiazki, start=1):
            lines.append(f"  {i}. {k}")
        return "\n".join(lines)
```

Użycie:

```python
lib = Biblioteka("Biblioteka Uczelniana")
lib.dodaj(Ksiazka("Solaris", "Stanisław Lem", 1961))
lib.dodaj(Ksiazka("Cyberiada", "Stanisław Lem", 1965))
lib.dodaj(Ksiazka("Lalka", "Bolesław Prus", 1890))

print(lib)

# Wypożyczenie
print(lib.ksiazki[0].wypozycz())

# Wyszukiwanie po autorze
for k in lib.szukaj("lem"):
    print(k)
```

Zwróć uwagę na podział odpowiedzialności:

- `Ksiazka` wie wszystko o **pojedynczej książce** — tytuł, autor, rok, status, jak się ją wypożycza.
- `Biblioteka` wie wszystko o **kolekcji książek** — jak je dodawać, wyszukiwać, jakie są dostępne.

To podstawowa zasada OOP: **każda klasa odpowiada za swój kawałek logiki**. `Biblioteka` nie zagląda w pola `Ksiazka` i nie ustawia ich ręcznie — woła `wypozycz()` i ufa, że książka sama wie, co zrobić ze swoim stanem.

---

## Praca projektowa

### Zadanie 1 — Mini Sklep Internetowy (10 pkt)

**Czas:** ok. 60 minut
**Środowisko:** Google Colab

Zaimplementuj dwie klasy tworzące uproszczony system sklepu: `Product` (produkt) i `Cart` (koszyk). Klasy mają ze sobą współpracować — koszyk przechowuje produkty i operuje na ich danych.

**Klasa `Product`:**

- konstruktor: `name` (str), `price` (float), `quantity` (int, domyślnie 0)
- walidacja: nazwa nie może być pusta, cena ani ilość nie mogą być ujemne — w razie błędu `raise ValueError`
- metody:
  - `is_available()` — `True` jeśli `quantity > 0`
  - `sell(amount=1)` — zmniejsza stan; obsługuje brak stanu i `amount <= 0`
  - `restock(amount)` — zwiększa stan; obsługuje `amount <= 0`
  - `__str__` — np. `Laptop - 3499.99 zł [5 szt.]` lub `Słuchawki - 199.99 zł [BRAK]` przy `quantity == 0`

**Klasa `Cart`:**

- konstruktor: bez argumentów; tworzy pustą listę `items` ze słownikami `{"product": Product, "quantity": int}`
- metody:
  - `add(product, quantity=1)` — dodaje produkt; jeśli już jest w koszyku, zwiększa ilość; sprawdza stan magazynowy
  - `remove(product_name)` — usuwa po nazwie (case-insensitive)
  - `total()` — łączna kwota koszyka (`float`)
  - `checkout()` — finalizuje zakup: na każdym produkcie woła `sell()`, wyświetla podsumowanie i czyści koszyk
  - `__str__` — zawartość koszyka z sumą

**Plan testów (kolejne komórki w notebooku):**

1. Utwórz min. **4 produkty** (jeden z `quantity=0`).
2. Wyświetl wszystkie produkty (`print`).
3. Sprawdź dostępność metodą `is_available()`.
4. Uzupełnij stan niedostępnego produktu metodą `restock()`.
5. Stwórz koszyk i dodaj kilka produktów w różnych ilościach.
6. Wyświetl koszyk.
7. Usuń jeden produkt z koszyka i wyświetl ponownie.
8. Wywołaj `checkout()`.
9. Sprawdź, że stany magazynowe spadły poprawnie.
10. Sprawdź, że koszyk jest pusty po finalizacji.

**Wskazówki:**

- `self` to referencja do konkretnej instancji — każdy produkt i każdy koszyk mają swój własny stan.
- `__str__` wywołuje się automatycznie przez `print()`.
- Formatowanie kwot: `f"{cena:.2f}"` (dwa miejsca po przecinku).
- Porównania nazw bez wielkości liter: `.lower()`.

**Punktacja (10 pkt):**

| Element | Punkty |
|---|---|
| Klasa `Product` z atrybutami i walidacją w konstruktorze | 2 |
| Metody `is_available`, `sell`, `restock` (w tym obsługa błędnych argumentów) | 2 |
| Metody `__str__` w `Product` i `Cart` (czytelne formatowanie) | 1 |
| Klasa `Cart` z metodami `add` / `remove` / `total` | 2 |
| `checkout()` — finalizacja zakupu i czyszczenie koszyka | 2 |
| Realizacja planu testów w notebooku (1–10) | 1 |