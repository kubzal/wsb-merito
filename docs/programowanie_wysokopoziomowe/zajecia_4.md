# Zajęcia 4

## Programowanie funkcyjne c.d. — i podsumowanie semestru

To ostatnie zajęcia. Plan jest taki:

1. **Domykamy temat funkcyjny** — dorzucam narzędzia, których jeszcze nie było, a które realnie ułatwiają życie: `itertools`, `operator`, generatory z `yield`, kompozycja funkcji.
2. **Ciekawostki** — skąd się wzięła `lambda`, gdzie programowanie funkcyjne spotkasz w prawdziwym kodzie z danymi.
3. **Podsumowanie semestru** — trzy paradygmaty w jednym miejscu, jako spójna mapa.
4. **Quiz i oddanie projektów** — co powtórzyć i jak oddać.

Po dzisiejszych zajęciach będziesz w stanie:

- używać `itertools` do leniwego przetwarzania kolekcji,
- zastąpić proste lambdy gotowcami z modułu `operator`,
- pisać własne generatory (`yield`) i rozumieć, czemu są leniwe,
- składać funkcje w pipeline,
- spiąć w głowie wszystkie trzy paradygmaty z całego semestru.

Dziś **nie ma zadania projektowego** — Część 3 z poprzednich zajęć była ostatnią.

---

## itertools — fabryka iteratorów

`itertools` to moduł ze standardowej biblioteki pełen **leniwych** narzędzi do pracy na kolekcjach. Wszystkie zwracają iteratory (liczą się na żądanie), więc świetnie współgrają z funkcyjnym stylem z poprzednich zajęć.

### `chain` — sklejanie kolekcji

```python
from itertools import chain

fiction = ["Lalka", "Ferdydurke"]
poetry = ["Pan Tadeusz"]

for title in chain(fiction, poetry):
    print(title)
# Lalka, Ferdydurke, Pan Tadeusz
```

Zamiast `fiction + poetry` (które tworzy nową listę w pamięci) `chain` przechodzi po obu po kolei, nic nie kopiując.

### `islice` — wycinanie z iteratora

Zwykłe `lista[2:5]` nie zadziała na iteratorze (generator nie ma indeksów). Od tego jest `islice`:

```python
from itertools import islice, count

# count(start, step) — nieskończony licznik
evens = (x for x in count(0, 2))      # 0, 2, 4, 6, ... w nieskończoność

print(list(islice(evens, 5)))         # [0, 2, 4, 6, 8]
```

`count`, `cycle` (zapętla kolekcję) i `repeat` (powtarza wartość) to **nieskończone** iteratory — bez `islice` albo `break` zawiesisz program. To cena leniwości: można reprezentować nieskończoność, byle jej nie zmaterializować.

### `takewhile` / `dropwhile` — bierz/porzucaj dopóki

```python
from itertools import takewhile, dropwhile

nums = [1, 2, 3, 10, 1, 2]

print(list(takewhile(lambda x: x < 5, nums)))   # [1, 2, 3]  — bierze do pierwszego fałszu
print(list(dropwhile(lambda x: x < 5, nums)))   # [10, 1, 2] — pomija do pierwszego fałszu
```

Uwaga — to nie to samo co `filter`. `filter` sprawdza **każdy** element; `takewhile` zatrzymuje się na pierwszym, który nie spełnia warunku, i kończy.

### `accumulate` — sumy kroczące

```python
from itertools import accumulate

daily_borrows = [3, 1, 4, 1, 5]
print(list(accumulate(daily_borrows)))   # [3, 4, 8, 9, 14] — narastająco
```

Domyślnie sumuje, ale możesz podać własną funkcję (np. `max`, by mieć „rekord narastająco").

### `groupby` — grupowanie (z pułapką!)

```python
from itertools import groupby

books = [
    {"title": "Pan Tadeusz", "author": "Mickiewicz"},
    {"title": "Dziady",      "author": "Mickiewicz"},
    {"title": "Lalka",       "author": "Prus"},
]

# PUŁAPKA: groupby grupuje tylko SĄSIEDNIE elementy → najpierw posortuj po kluczu!
books_sorted = sorted(books, key=lambda b: b["author"])

for author, group in groupby(books_sorted, key=lambda b: b["author"]):
    titles = [b["title"] for b in group]
    print(f"{author}: {titles}")
# Mickiewicz: ['Pan Tadeusz', 'Dziady']
# Prus: ['Lalka']
```

Najczęstszy błąd początkujących: `groupby` bez wcześniejszego `sorted`. Wtedy ten sam autor rozbity w kilku miejscach listy da kilka osobnych grup. To inaczej niż `GROUP BY` w SQL — tutaj kolejność ma znaczenie.

### `combinations` / `product` — kombinatoryka za darmo

```python
from itertools import combinations

readers = ["anna", "tomek", "kasia"]

for pair in combinations(readers, 2):
    print(pair)
# ('anna', 'tomek'), ('anna', 'kasia'), ('tomek', 'kasia')
```

`product` daje iloczyn kartezjański (każdy z każdym), `permutations` — wszystkie uporządkowania. Przydaje się, gdy chcesz wygenerować wszystkie pary/zestawienia bez pisania zagnieżdżonych pętli.

---

## operator — gotowe lambdy

Wiele lambd, które piszesz do `sorted`, `map` czy `max`, to po prostu „weź pole X" albo „zawołaj metodę Y". Moduł `operator` ma to gotowe — czytelniej i odrobinę szybciej.

```python
from operator import itemgetter, attrgetter, methodcaller

books = [
    {"title": "Pan Tadeusz", "author": "Mickiewicz", "copies": 3},
    {"title": "Lalka",       "author": "Prus",       "copies": 0},
]

# było:
by_copies = sorted(books, key=lambda b: b["copies"])
# można:
by_copies = sorted(books, key=itemgetter("copies"))

# wielokryterialne — itemgetter przyjmuje kilka kluczy i zwraca krotkę
by_author_then_title = sorted(books, key=itemgetter("author", "title"))
```

Trzy najczęstsze:

| Z `operator` | Odpowiednik lambdą | Do czego |
|---|---|---|
| `itemgetter("copies")` | `lambda b: b["copies"]` | dostęp po kluczu/indeksie (`dict`, `list`, `tuple`) |
| `attrgetter("author")` | `lambda b: b.author` | dostęp do atrybutu obiektu |
| `methodcaller("strip")` | `lambda s: s.strip()` | wywołanie metody na elemencie |

```python
from operator import methodcaller

names = ["  Anna ", "TOMEK", " kasia "]
clean = list(map(methodcaller("strip"), names))
print(clean)   # ['Anna', 'TOMEK', 'kasia']
```

Reguła kciuka: jeśli lambda to dokładnie „weź pole" albo „zawołaj metodę bez argumentów" — `operator` jest czytelniejszy. Coś bardziej złożonego → zostań przy lambdzie.

---

## Generatory i `yield` — leniwość na poważnie

Na zajęciach 3 widzieliśmy **generator expressions** (`(x for x in ...)`). Teraz ich starszy brat — **funkcja generatora** ze słowem `yield`.

Zwykła funkcja zwraca jedną wartość i kończy się. Generator **oddaje wartości po jednej** (`yield`), zatrzymuje się i czeka, aż ktoś poprosi o następną.

```python
def available_books(books):
    for b in books:
        if b["copies"] > 0:
            yield b           # oddaj jedną książkę i zaczekaj


books = [
    {"title": "Pan Tadeusz", "copies": 3},
    {"title": "Lalka",       "copies": 0},
    {"title": "Ferdydurke",  "copies": 1},
]

for book in available_books(books):
    print(book["title"])      # Pan Tadeusz, Ferdydurke
```

Kluczowa różnica: ta funkcja **nie buduje listy**. Generuje kolejne elementy dopiero, gdy pętla o nie poprosi.

### Po co to?

**Pamięć.** Klasyczny przykład — czytanie wielkiego pliku linia po linii:

```python
def read_lines(path):
    with open(path, encoding="utf-8") as f:
        for line in f:
            yield line.strip()
```

Plik może mieć 50 GB — generator i tak trzyma w pamięci tylko jedną linię naraz. Lista z 50 GB danych zabiłaby program.

### Generator pipeline — leniwy łańcuch

```python
numbers = range(1, 1_000_000)
squared = (n * n for n in numbers)          # nic się jeszcze nie policzyło
evens   = (n for n in squared if n % 2 == 0)  # nadal nic
total   = sum(evens)                          # DOPIERO TERAZ liczymy — element po elemencie
```

Żaden krok nie tworzy listy miliona elementów. Dane „przepływają" przez pipeline pojedynczo aż do `sum`. To dokładnie ten leniwy, funkcyjny duch — i, jak zobaczymy w ciekawostkach, tak właśnie działa Spark na danych liczonych w terabajtach.

---

## Wbudowane funkcje wyższego rzędu z `key`

Nie tylko `sorted` przyjmuje `key`. Robią to też `max` i `min`, a `any`/`all` świetnie łączą się z generatorami:

```python
from operator import itemgetter

books = [
    {"title": "Pan Tadeusz", "copies": 3},
    {"title": "Lalka",       "copies": 0},
    {"title": "Ferdydurke",  "copies": 1},
]

most_stocked  = max(books, key=itemgetter("copies"))     # {'title': 'Pan Tadeusz', ...}
least_stocked = min(books, key=itemgetter("copies"))     # {'title': 'Lalka', ...}

any_available = any(b["copies"] > 0 for b in books)      # True — wystarczy jeden
all_available = all(b["copies"] > 0 for b in books)      # False — Lalka ma 0
```

`any`/`all` są **leniwe** — `any` przestaje szukać przy pierwszym `True`, `all` przy pierwszym `False`. Dla dużych kolekcji to oszczędność.

---

## Kompozycja funkcji — pipeline z funkcji

W „czystym" funkcyjnym świecie program to często **złożenie funkcji**: wynik jednej wpada do drugiej. Python nie ma wbudowanego operatora kompozycji, ale można go zrobić w jednej linii (znów `reduce` z zajęć 3):

```python
from functools import reduce

def compose(*funcs):
    return reduce(lambda f, g: lambda x: f(g(x)), funcs)


clean_title = compose(str.strip, str.lower)
print(clean_title("  Pan Tadeusz  "))   # 'pan tadeusz'
```

`compose(str.strip, str.lower)` znaczy „najpierw `lower`, potem `strip`". To esencja stylu funkcyjnego: budujesz duże transformacje, składając małe, czyste funkcje jak klocki.

---

## Ciekawostki

### Skąd nazwa „lambda"?

Słowo `lambda` nie wzięło się znikąd. To grecka litera **λ** z **rachunku lambda** — formalnego systemu opisu obliczeń, który w latach 30. XX wieku stworzył matematyk **Alonzo Church** (nawiasem mówiąc, promotor Alana Turinga). Rachunek lambda i maszyna Turinga okazały się równoważne — to dwa różne sposoby opisania tego samego pojęcia „obliczalności".

Do programowania `lambda` trafiła przez język **Lisp** (1958, John McCarthy) i od tej pory wędruje przez kolejne języki. Gdy piszesz `lambda x: x * 2`, używasz notacji sprzed prawie stu lat.

### Guido chciał wyrzucić `map`, `filter` i `reduce`

Twórca Pythona, **Guido van Rossum**, nigdy nie przepadał za funkcyjnym stylem — uważał, że **comprehensions są czytelniejsze**. Przy przejściu na Pythona 3 `reduce` wyleciało z funkcji wbudowanych do modułu `functools` (dlatego importujesz je osobno). `map` i `filter` przetrwały, ale dodatkowo zmieniły się tak, że zwracają leniwe iteratory zamiast list. Stąd to nieustanne `list(map(...))` — pamiątka po tej decyzji.

### `match` — pattern matching (Python 3.10+)

Stosunkowo nowy dodatek, inspirowany językami funkcyjnymi. To dużo potężniejszy `switch` — potrafi „rozpakować" strukturę danych:

```python
def handle(command):
    match command.split():
        case ["wypozycz", title]:
            return f"Wypożyczam: {title}"
        case ["zwroc", title]:
            return f"Zwracam: {title}"
        case ["wyloguj"]:
            return "Wylogowano"
        case _:
            return "Nieznana komenda"


print(handle("wypozycz Lalka"))   # Wypożyczam: Lalka
print(handle("wyloguj"))          # Wylogowano
```

### Walrus `:=` (Python 3.8+)

Operator „morsa" (bo `:=` wygląda jak oczy i kły morsa) pozwala **przypisać i użyć wartości w jednym wyrażeniu**:

```python
books = [{"title": "Lalka", "copies": 0}]

if (n := sum(b["copies"] for b in books)) == 0:
    print(f"Brak dostępnych egzemplarzy (łącznie: {n})")
```

Liczymy sumę raz, od razu zapisujemy do `n` i używamy. Bez walrusa trzeba by ją liczyć dwa razy albo dodać linię wcześniej.

### Gdzie to wszystko prowadzi — FP w świecie danych

To nie jest akademicka ciekawostka. Funkcyjne podejście to **codzienny chleb pracy z danymi**:

```python
# pandas — funkcyjny duch
df["title_upper"] = df["title"].map(str.upper)   # map na kolumnie
available = df[df["copies"] > 0]                  # filtrowanie warunkiem
```

W **PySpark** (przetwarzanie danych na klastrach, terabajty) idea leniwych generatorów wchodzi na skalę przemysłową: transformacje `filter`, `map`, `select` są **leniwe** — Spark niczego nie liczy, dopóki nie poprosisz o wynik (tzw. *action*, np. zapis albo `count`). Buduje sobie plan, optymalizuje go i dopiero wtedy odpala. To ten sam mechanizm, co generator pipeline z dzisiejszych zajęć, tylko rozłożony na setki maszyn.

Krótko: `map`, `filter`, leniwość i czyste funkcje, które dziś ćwiczymy na liście trzech książek, w realnym data engineeringu rządzą przetwarzaniem miliardów rekordów.

---

## Podsumowanie semestru — trzy paradygmaty w jednym miejscu

Cały semestr to była jedna historia opowiedziana trzy razy: ta sama biblioteka, trzy sposoby myślenia.

| | Strukturalne | Obiektowe (OOP) | Funkcyjne (FP) |
|---|---|---|---|
| **Jednostka** | funkcja + dane osobno | obiekt (dane + zachowanie razem) | czysta funkcja, transformacja |
| **Pytanie** | jakie kroki wykonać? | jakie obiekty istnieją i co robią? | jak przepływają dane przez funkcje? |
| **Narzędzia Pythona** | `def`, `if`, `for`, listy, słowniki | `class`, dziedziczenie, `@property`, dunders, `dataclass` | `lambda`, `map`/`filter`, comprehensions, `sorted(key=...)`, `functools`, `itertools` |
| **Stan** | zmienne globalne/lokalne | stan wewnątrz obiektu | unikamy zmiany stanu (immutability) |
| **Sprawdza się** | proste skrypty, „wejście → wynik" | duże aplikacje z wieloma bytami | przetwarzanie i analiza danych |

### Ten sam problem, trzy podejścia

Przez trzy części projektu przepisywałeś **dokładnie tę samą bibliotekę**:

- **Część 1 (strukturalnie)** — słowniki jako dane, funkcje jako operacje, `while True` jako menu.
- **Część 2 (OOP)** — `Book`, `User`, `Reader`, `Librarian`, `Library`; dane i logika razem, dziedziczenie i polimorfizm.
- **Część 3 (funkcyjnie)** — wyszukiwanie, sortowanie i statystyki przez `filter`, `map`, `sorted`, comprehensions i funkcje wyższego rzędu.

To była celowa lekcja: **paradygmat to nie cecha języka, tylko sposób myślenia**. Python pozwala mieszać wszystkie trzy.

### Sześć rzeczy do zapamiętania na lata

1. **Paradygmat to sposób myślenia, nie sztywna reguła** — w prawdziwym kodzie mieszasz wszystkie trzy świadomie.
2. **OOP do modelowania świata** — gdy masz byty z danymi i zachowaniem (User, Order, Product), klasy są naturalne.
3. **FP do operacji na danych** — filtrowanie, mapowanie, agregacje czytają się lepiej funkcyjnie niż jako ręczne pętle.
4. **Hermetyzacja i czyste funkcje to ten sam cel** — ograniczyć, ile kodu może coś zepsuć.
5. **Czytelność > sprytność** — comprehension zamiast zagnieżdżonego `map`/`filter`, krótka lambda zamiast funkcji-potwora.
6. **Standardowa biblioteka jest ogromna** — `functools`, `itertools`, `operator`, `dataclasses` rozwiązują problemy, zanim sięgniesz po pętlę.

---

## Przygotowanie do quizu

Quiz odbędzie się dziś na Moodle, obejmuje materiał z ćwiczeń — maks. **30 pkt**. Poniżej ściąga, co warto przejrzeć.

### Co powtórzyć — checklist

**Zajęcia 1 — podstawy i paradygmaty**

- czym różnią się trzy paradygmaty (tabela wyżej),
- podstawy: zmienne, `if`/`elif`/`else`, `for`/`while`, listy, słowniki, lista słowników,
- funkcje: parametry domyślne, zwracanie krotki, `return` vs `print`.

**Zajęcia 2 — OOP podstawy**

- `class`, `__init__`, `self`, atrybut klasy vs obiektu,
- cztery filary: hermetyzacja, dziedziczenie, polimorfizm, abstrakcja,
- `super()`, nadpisywanie metod, duck typing,
- dunders: `__str__` vs `__repr__`, `__eq__`, `__len__`, `__iter__`.

**Zajęcia 3 — OOP zaawansowane + FP**

- `@dataclass` (i pułapka `field(default_factory=list)`), `frozen=True`,
- `@staticmethod` vs `@classmethod` (alternatywny konstruktor),
- kompozycja vs dziedziczenie („jest" vs „ma"),
- `lambda`, `map`, `filter`, `reduce`, comprehensions, `sorted(key=...)`,
- funkcje wyższego rzędu, closures, czysta funkcja vs efekt uboczny.

**Zajęcia 4 — domknięcie**

- `itertools` (zwłaszcza pułapka `groupby` z sortowaniem),
- `operator` (`itemgetter`, `attrgetter`),
- generatory i `yield` — czemu są leniwe.

### Pytania kontrolne (do samodzielnego sprawdzenia — to NIE są pytania z quizu)

1. Czym różni się `__str__` od `__repr__` i które warto zdefiniować, jeśli masz zdefiniować tylko jedno?
2. Dlaczego `borrowed_books: list = []` w `dataclass` to pułapka, a `field(default_factory=list)` nie?
3. Co zwraca `map(...)` w Pythonie 3 — listę czy iterator?
4. Kiedy `Reader` powinien **dziedziczyć** po `User`, a kiedy `Library` powinna **mieć** magazyn (kompozycja)?
5. Dlaczego `groupby` z `itertools` wymaga wcześniejszego `sorted`?
6. Czym różni się czysta funkcja od funkcji z efektem ubocznym i czemu ta pierwsza jest łatwiejsza do testowania?

Jeśli na każde z tych pytań umiesz odpowiedzieć jednym–dwoma zdaniami i pokazać przykład — jesteś gotowy.

---

## Oddanie projektów i zadań

Drobiazgi techniczne, na które warto rzucić okiem przed oddaniem:

- czy `.gitignore` zawiera `.venv/`, `__pycache__/`, `*.db` (baza nie powinna trafić do repo),
- czy projekt odpala się z czystego klona (`uv sync`, potem `uv run`),
- czy PR jest z brancha do `main`, a link działa publicznie.

To wszystko z mojej strony. Dzięki za cały semestr i powodzenia na quizie.
