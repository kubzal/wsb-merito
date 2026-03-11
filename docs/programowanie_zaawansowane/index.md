# Wprowadzenie

## Plan zajęć

- **Zajęcia 1 (_2026-03-19_):** Delegaty (Action, Func, Predicate), wyrażenia lambda, LINQ (składnia zapytań i metod, projekcje, joiny)
- **Zajęcia 2 (_2026-04-02_):** Zdarzenia niestandardowe (EventArgs), programowanie asynchroniczne (async/await), TPL, wielowątkowość, bezpieczeństwo wątków
- **Zajęcia 3 (_2026-04-16_):** Serializacja i deserializacja (JSON, XML), LINQ to XML, operacje na plikach i strumieniach (System.IO), FileSystemWatcher
- **Zajęcia 4 (_2026-04-30_):** Entity Framework Core: ORM, relacje, migracje, transakcje, seeding
- **Zajęcia 5 (_2026-05-14_):** Testowanie jednostkowe (xUnit), TDD, integracja z REST API (HttpClient)

## Zasady zaliczenia

- Ćwiczenia zaliczamy **projektem** realizowanym na laboratoriach — łącznie **100 punktów** (5 laboratoriów × max 20 pkt).
- Zadania wysyłamy na **GitHub** — na Moodle wrzucamy jedynie **link do pull requesta**.
- Za pracę na zajęciach (min. 1 commit w trakcie laboratorium) przysługuje **5 pkt z puli 20**. Bez commita z zajęć maksimum za dane zadanie to **15 pkt**.
- Zadanie można dokończyć w domu po zajęciach.
- Mogę zapytać o dowolny fragment kodu i jego znaczenie — na bieżących zajęciach (jeśli zadanie ukończone na miejscu) lub na kolejnych (jeśli dokończone w domu). **Nieumiejętność wyjaśnienia własnego kodu może skutkować obniżeniem punktacji.**

**Skala ocen:**

| Punkty | Ocena |
|---|---|
| 91–100% | 5,0 |
| 81–90% | 4,5 |
| 71–80% | 4,0 |
| 61–70% | 3,5 |
| 51–60% | 3,0 |
| ≤ 50% | 2,0 |

## Wymagania wstępne

- **Git** — init, commit, push, pull, repozytoria zdalne, praca na branchach, pull request.
- **Aplikacje konsolowe** — umiejętność tworzenia w dowolnym języku programowania.
- **Programowanie strukturalne** — zmienne, pętle, warunki, funkcje.
- **Programowanie obiektowe** — klasy, obiekty, dziedziczenie, hermetyzacja.

## Środowisko programistyczne
Zajęcia z Programowania zaawansowanego będą realizowane w języku C#. 
Jest mi obojętne, w jakim środowisku będą Państwo pisać swój kod — byle działało 😉. Poniżej kilka polecanych przeze mnie opcji.

1. VS Code + C# Dev Kit
2. JetBrains Rider
3. Visual Studio

W zależności od tego, czy bliższe jest Państwu środowisko JetBrains 
(bo np. programowaliście wcześniej w IntelliJ albo PyCharm), czy VS Code — sugerowałbym jedną z tych opcji. 
Dawno nie miałem natomiast do czynienia z Visual Studio, więc ciężko mi jest wypowiadać się o tym środowisku.

Jeżeli nie programowaliście Państwo zbyt wiele w swoim życiu i nie czujecie się zbyt pewnie, to polecam Ridera, 
który większość rzeczy robi za nas i jest darmowy do zastosowań niekomercyjnych. Innymi słowy — idealna niania na start.

Ja na zajęciach będę korzystał z VS Code. 

Należy pobrać wybrany przez siebie program oraz .NET 10 właściwy dla swojego systemu operacyjnego.

## Jak utworzyć nowy projekt w JetBrains Rider?
Proszę znaleźć i pobrać program JetBrains Rider oraz .NET 10. Po pobraniu i instalacji proszę postępować według poniższej instrukcji.

1. Włącz program _JetBrains Rider_.
2. Wybierz `New solution`.
3. W _Solution name_ wpisz nazwę projektu, np. `Zajecia1`.
4. Upewnij się, że wybrany język programowania w _Language_ to `C#` oraz _Target framework_ to `net10.0`.

## Jak utworzyć nowy projekt w VS Code?
Proszę pobrać i zainstalować program VS Code oraz .NET 10. Następnie w Extensions należy wyszukać `C# Dev Kit` i zainstalować.

Opcjonalnie polecam również wcisnąć kombinację klawiszy `Ctrl/Cmd + Shift + P`, wpisać `code` i wybrać `Shell command: Install 'code' command in PATH`. Ta komenda pozwoli uruchamiać VS Code z terminala — raz, a dobrze.

Po tej operacji można na razie zamknąć VS Code i postępować według poniższej instrukcji.

1. Włącz terminal i sprawdź, czy masz zainstalowany dotnet w wersji 10.
```bash
dotnet --version
```
2. Stwórz nowy katalog w miejscu, gdzie będziesz chciał mieć swój projekt, i wejdź do niego.
```bash
mkdir zajecia1
cd zajecia1
```
3. Następnie polecam otworzyć stworzony katalog w VS Code i uruchomić zintegrowany terminal, np. poniższą komendą.
```bash
code .
```
4. Żeby stworzyć nowy projekt w bieżącym katalogu, wpisz poniższą komendę.
```bash
dotnet new console
```
5. Poprzednia komenda powinna stworzyć kilka plików, m.in. plik `Program.cs` z przykładowym kodem `Hello world`. Uruchom go poniższą komendą. Jeśli zobaczyłeś `Hello, World!` — gratulacje, Twoja przygoda z C# właśnie się zaczęła! 🎉
```bash
dotnet run
```

---

## _Hello world!_

Każda przygoda z programowaniem zaczyna się od _Hello world!_ — także nie inaczej będzie u nas!

W swoim ulubionym środowisku stwórz nowy projekt, otwórz plik _Program.cs_ i wpisz:

```cs
Console.WriteLine("Hello World!");
```

A następnie uruchom program (jeżeli używasz VS Code, wpisz w terminalu `dotnet run`; jeżeli używasz Ridera, kliknij przycisk Run w prawym górnym rogu — ale to ostatni raz, kiedy Ci o tym przypominam. Od teraz zakładam, że wiesz jak uruchomić program 😉).

Na ekranie w terminalu powinien się ukazać tekst:
```
Hello World!
```

Jest to tradycyjnie pierwszy program, jaki każdy programista tworzy poznając nowy język programowania. Taki rytuał inicjacyjny branży IT.

I co?! To tyle?! Możesz być zaskoczony, jeżeli programowałeś w starszej wersji C# albo w Javie. C# jest bardzo podobny do Javy. My korzystamy z C# 14, który jest dostarczany wraz z .NET 10. W starszych wersjach ten sam program był znacznie dłuższy i wyglądał mniej więcej tak:

```cs
using System;

namespace HelloWorld
{
  class Program
  {
    static void Main(string[] args)
    {
      Console.WriteLine("Hello World!");    
    }
  }
}
```

Rozłóżmy to sobie na czynniki pierwsze.

- Pierwsza linijka `using System;` mówi nam, że korzystamy z przestrzeni nazw (_namespace_) `System`. To trochę jak `import` w Pythonie — dzięki temu możemy pisać po prostu `Console.WriteLine(...)` zamiast `System.Console.WriteLine(...)`. W nowych wersjach C# (od C# 10) nie musimy tego pisać ręcznie, bo mechanizm _implicit usings_ robi to za nas automatycznie.
- Druga linijka definiuje namespace `HelloWorld`. Namespace to sposób na logiczne grupowanie kodu — coś jak szufladki, do których wrzucamy powiązane ze sobą klasy, żeby nam się nie pomieszały. Jeśli znasz Pythona, pomyśl o tym jak o module/paczce.
- Dalej mamy zdefiniowaną klasę `Program` oraz metodę statyczną `Main`, nazywaną też zwyczajowo _funkcją main_ (tak jak w C/C++). To punkt wejścia naszego programu — od niej zaczyna się wykonywanie kodu.

Dopiero wewnątrz metody `Main` mamy znaną już nam linijkę drukującą na ekranie _Hello World!_. Taka struktura wynika z tego, że — podobnie jak Java — C# jest językiem obiektowym i wszystko musi żyć w jakiejś klasie. Nawet najprostszy `Hello World` nie dostaje taryfy ulgowej.

Co więc się stało?! Czy C# przestał być obiektowy?! W żadnym razie! Od wersji C# 9 nie ma konieczności definiowania metody `Main`, ponieważ C# sam opakowuje nasz kod w tę metodę za kulisami. Jest to tzw. _top-level statements_ — takie ułatwienie, żebyśmy mogli szybciej przejść do meritum zamiast pisać kilkanaście linijek ceremoniału.

## Szybka powtórka

Ponieważ ten przedmiot nazywa się _Programowanie zaawansowane_, to zakładam, że musiałeś już kiedyś coś kodzić — nawet jeśli nie w C#, to w innym języku. Wiesz co to ify, pętle, funkcje, klasy itd. To nie jest podręcznik do C# i bez problemu znajdziesz mnóstwo materiałów, jeżeli potrzebujesz omówienia podstaw od zera. Ale żeby nie zostawić Cię bez niczego — poniżej mała ściąga najważniejszych zagadnień na przykładach.


### Zmienne

C# jest językiem **silnie typowanym** — każda zmienna musi mieć określony typ. Oto najczęściej spotykane typy danych:

```cs
// liczba całkowita
int wiek = 25;

// liczba zmiennoprzecinkowa (64-bit)
double pi = 3.14;

// liczba zmiennoprzecinkowa (32-bit), wymaga suffiksu 'f'
float temperatura = 36.6f; 

// typ dla precyzyjnych obliczeń (np. pieniądze), suffiks 'm'
decimal cena = 19.99m; 

// wartość logiczna
bool czyZaliczyl = true;

// pojedynczy znak (apostrof!)
char ocena = 'A';

// tekst (cudzysłów!)
string imie = "Jan";
```

Możesz też użyć słowa kluczowego `var` — kompilator sam wydedukuje typ na podstawie przypisanej wartości:

```cs
var miasto = "Warszawa"; // kompilator wie, że to string
var liczba = 42; // kompilator wie, że to int
```

> :warning: **Uwaga!**
    `var` nie oznacza, że zmienna nie ma typu (to nie Python!). Typ jest ustalany w momencie kompilacji i nie może się zmienić.

#### Rzutowanie typów

Czasem trzeba zamienić jeden typ na inny. C# rozróżnia dwa rodzaje konwersji:

**Niejawna (implicit)** — dzieje się automatycznie, gdy nie tracimy danych (np. `int` → `double`):

```cs
int a = 10;
double b = a; // OK — int "mieści się" w double
```

**Jawna (explicit)** — wymaga ręcznego rzutowania, bo możemy stracić dane:

```cs
double x = 9.78;
int y = (int)x; // y = 9, część dziesiętna zostaje obcięta
```

Konwersja ze `string` na typ liczbowy (i odwrotnie):

```cs
string tekst = "123";
int liczba = int.Parse(tekst); // string → int
int liczba2 = Convert.ToInt32(tekst); // alternatywna metoda

string zPowrotem = liczba.ToString();  // int → string
```

#### Przepełnienie i `checked`

Co się stanie, gdy przekroczysz maksymalną wartość danego typu? Domyślnie C# po cichu „zawija" wartość (tzw. _overflow_) — co może prowadzić do trudnych do znalezienia bugów:

```cs
int max = int.MaxValue; // 2 147 483 647
int wynik = max + 1; // -2 147 483 648 😱 (overflow bez błędu!)
```

Jeśli chcesz, żeby program rzucił wyjątek zamiast cicho się psuć, użyj `checked`:

```cs
checked
{
    int max = int.MaxValue;
    int wynik = max + 1; // System.OverflowException!
}
```

### Stałe

Jeśli masz wartość, która nigdy się nie zmieni — użyj `const`. Próba przypisania nowej wartości zakończy się błędem kompilacji:

```cs
const double Pi = 3.14159;
const string Uczelnia = "WSB Merito";

// Pi = 3.14; // błąd kompilacji!
```

### `if`, `else`, `else if`

Klasyczna instrukcja warunkowa — działa tak samo jak w większości języków, z tą różnicą, że warunek **musi** być typu `bool` (żadnego `if (1)` jak w C/C++):

```cs
int ocena = 4;

if (ocena == 5)
{
    Console.WriteLine("Brawo!");
}
else if (ocena >= 3)
{
    Console.WriteLine("Zaliczone.");
}
else
{
    Console.WriteLine("Poprawka 😬");
}
```

### Pętla `while`

Wykonuje się dopóki warunek jest prawdziwy. Uwaga na pętle nieskończone — to nie feature, to bug.

```cs
int i = 0;

while (i < 5)
{
    Console.WriteLine($"i = {i}");
    i++;
}
```

### Pętla `do while`

Jak `while`, ale gwarantuje co najmniej jedno wykonanie — bo warunek sprawdzany jest **po** wykonaniu bloku:

```cs
int liczba;

do
{
    Console.Write("Podaj liczbę większą od 0: ");
    liczba = int.Parse(Console.ReadLine()!);
} while (liczba <= 0);

Console.WriteLine($"Podałeś: {liczba}");
```

### Pętla `for`

Klasyk. Idealna, gdy wiesz z góry ile razy chcesz coś powtórzyć:

```cs
for (int i = 0; i < 5; i++)
{
    Console.WriteLine($"Krok {i}");
}
```

### Pętla `foreach`

Najwygodniejszy sposób na iterowanie po kolekcjach. Nie musisz się martwić o indeksy:

```cs
string[] owoce = { "jabłko", "banan", "kiwi" };

foreach (string owoc in owoce)
{
    Console.WriteLine(owoc);
}
```

### Listy i tablice

**Tablica (`array`)** — ma stały rozmiar ustalany przy tworzeniu. Szybka, ale sztywna jak regulamin BHP:

```cs
int[] oceny = new int[3]; // tablica 3 elementów (domyślnie 0)
oceny[0] = 5;
oceny[1] = 4;
oceny[2] = 3;

string[] dni = { "Pon", "Wt", "Śr" }; // inicjalizacja od razu
Console.WriteLine(dni.Length); // 3
```

**Lista (`List<T>`)** — dynamiczna kolekcja, która rośnie i maleje w miarę potrzeb. To co pewnie znasz z Pythona jako zwykłą listę:

```cs
List<string> zakupy = new List<string>();
zakupy.Add("mleko");
zakupy.Add("chleb");
zakupy.Add("kawa");
zakupy.Remove("mleko");

Console.WriteLine(zakupy.Count); // 2
Console.WriteLine(zakupy[0]); // chleb
```

`T` w `List<T>` to tzw. _typ generyczny_ — wstawiasz tam typ, który chcesz przechowywać:

```cs
List<int> liczby = new List<int> { 1, 2, 3, 4, 5 };
List<double> temperatury = new List<double> { 36.6, 37.2, 38.1 };
```

| Cecha | Tablica (`T[]`) | Lista (`List<T>`) |
|---|---|---|
| Rozmiar | Stały | Dynamiczny |
| Wydajność | Szybsza (bezpośredni dostęp do pamięci) | Minimalnie wolniejsza |
| Elastyczność | Niska | Wysoka (`Add`, `Remove`, `Insert`...) |
| Kiedy używać? | Znasz rozmiar z góry | Rozmiar może się zmieniać |

Here's a dictionary section in the same style:

### Słowniki

Słownik (`Dictionary<TKey, TValue>`) — kolekcja par klucz-wartość. Jeśli znasz Pythona, to jest dokładnie to samo co `dict`. Szukasz po kluczu, dostajesz wartość — szybko i bez zbędnego przeszukiwania:

```csharp
Dictionary<string, int> wiek = new Dictionary<string, int>();
wiek["Ala"] = 25;
wiek["Bob"] = 30;
wiek["Celina"] = 22;

Console.WriteLine(wiek["Bob"]); // 30
Console.WriteLine(wiek.Count);  // 3
```

Można też zainicjalizować od razu — składnia wygląda jak JSON po trzech kawach:

```csharp
Dictionary<string, string> stolice = new Dictionary<string, string>
{
    { "Polska", "Warszawa" },
    { "Niemcy", "Berlin" },
    { "Francja", "Paryż" }
};

// lub nowsza składnia (C# 6+), bliższa temu co znasz z Pythona:
Dictionary<string, string> stolice2 = new()
{
    ["Polska"] = "Warszawa",
    ["Niemcy"] = "Berlin",
    ["Francja"] = "Paryż"
};
```

Uwaga na pułapkę — odwołanie się do nieistniejącego klucza rzuci `KeyNotFoundException` (w Pythonie byłby `KeyError`). Bezpieczniej użyć `TryGetValue`:

```csharp
if (stolice.TryGetValue("Japonia", out string? wynik))
{
    Console.WriteLine(wynik);
}
else
{
    Console.WriteLine("Nie ma takiego klucza!");
}
```

Przydatne operacje:

```csharp
stolice.ContainsKey("Polska");        // true
stolice.ContainsValue("Berlin");      // true
stolice.Remove("Francja");            // usuwa parę

// iteracja — jak w Pythonie dict.items()
foreach (KeyValuePair<string, string> para in stolice)
{
    Console.WriteLine($"{para.Key} → {para.Value}");
}

// lub krócej z var:
foreach (var (kraj, miasto) in stolice)
{
    Console.WriteLine($"{kraj} → {miasto}");
}
```

| Cecha | `Dictionary<TKey, TValue>` | `List<T>` |
|---|---|---|
| Dostęp po | Kluczu (dowolny typ) | Indeksie (int) |
| Szukanie elementu | Bardzo szybkie (O(1)) | Wolne (O(n)) |
| Kolejność | Nie gwarantowana* | Zachowana |
| Kiedy używać? | Mapowanie klucz → wartość | Uporządkowana kolekcja |

\* W praktyce `Dictionary` w C# zachowuje kolejność wstawiania (jak Python 3.7+), ale oficjalnie tego nie gwarantuje. Jeśli kolejność jest kluczowa — jest `SortedDictionary<TKey, TValue>`.


### Funkcje

A w zasadzie **metody statyczne** — bo w C# każda funkcja żyje wewnątrz klasy. Na razie, w kontekście _top-level statements_, wystarczy nam słowo `static`:

```cs
static int Dodaj(int a, int b)
{
    return a + b;
}

static void Przywitaj(string imie)
{
    Console.WriteLine($"Cześć, {imie}!");
}

// Użycie:
int suma = Dodaj(3, 5); // 8
Przywitaj("Anna"); // Cześć, Anna!
```

Zwróć uwagę na:

- **Typ zwracany** — przed nazwą metody (`int`, `void` jeśli nic nie zwraca).
- **Parametry** — każdy musi mieć określony typ.
- **`return`** — zwraca wartość i kończy działanie metody.


### Klasy

Klasa to szablon opisujący obiekty — definiuje jakie dane przechowują (pola/właściwości) i co potrafią robić (metody). Dobrą praktyką jest ustawianie wartości przez **konstruktor**, a nie przez publiczne settery — dzięki temu nie da się stworzyć obiektu w "niepełnym" stanie (np. studenta bez imienia):

```cs
class Student
{
    public string Imie { get; private set; }
    public int NumerIndeksu { get; private set; }

    public Student(string imie, int numerIndeksu)
    {
        Imie = imie;
        NumerIndeksu = numerIndeksu;
    }

    public void PrzedstawSie()
    {
        Console.WriteLine($"Jestem {Imie}, numer indeksu: {NumerIndeksu}");
    }
}

// Użycie:
Student s = new Student("Jan", 12345);
s.PrzedstawSie();  // Jestem Jan, numer indeksu: 12345

// s.Imie = "Adam";  // ❌ błąd kompilacji — setter jest prywatny!
```

Zwróć uwagę na `private set` — właściwość można odczytać z zewnątrz (`s.Imie`), ale zmienić ją może tylko sama klasa (np. w konstruktorze). To taki kompromis między pełną enkapsulacją a wygodą.


#### Dziedziczenie

Klasa może dziedziczyć po innej klasie, przejmując jej pola i metody. Dzięki temu unikamy powtarzania kodu — bo lenistwo to cnota programisty. Klasa pochodna wywołuje konstruktor klasy bazowej za pomocą słowa kluczowego `base`:

```cs
class Osoba
{
    public string Imie { get; private set; }

    public Osoba(string imie)
    {
        Imie = imie;
    }

    public void PrzedstawSie()
    {
        Console.WriteLine($"Jestem {Imie}.");
    }
}

class Wykladowca : Osoba
{
    public string Przedmiot { get; private set; }

    public Wykladowca(string imie, string przedmiot) : base(imie)
    {
        Przedmiot = przedmiot;
    }

    public void ProwadzZajecia()
    {
        Console.WriteLine($"{Imie} prowadzi: {Przedmiot}");
    }
}

// Użycie:
Wykladowca w = new Wykladowca("Jakub", "Programowanie zaawansowane");
w.PrzedstawSie(); // Jestem Jakub.
w.ProwadzZajecia(); // Jakub prowadzi: Programowanie zaawansowane
```

`: base(imie)` po konstruktorze `Wykladowca` oznacza: „zanim zrobisz cokolwiek, najpierw wywołaj konstruktor klasy `Osoba` i przekaż mu `imie`".


#### Polimorfizm

Polimorfizm pozwala na nadpisywanie zachowania metod z klasy bazowej w klasach dziedziczących. Klasa bazowa oznacza metodę jako `virtual`, a klasa pochodna nadpisuje ją za pomocą `override`:

```cs
class Zwierze
{
    public string Imie { get; private set; }

    public Zwierze(string imie)
    {
        Imie = imie;
    }

    public virtual void DajGlos()
    {
        Console.WriteLine($"{Imie}: ...");
    }
}

class Pies : Zwierze
{
    public Pies(string imie) : base(imie) { }

    public override void DajGlos()
    {
        Console.WriteLine($"{Imie}: Hau hau!");
    }
}

class Kot : Zwierze
{
    public Kot(string imie) : base(imie) { }

    public override void DajGlos()
    {
        Console.WriteLine($"{Imie}: Miau!");
    }
}

// Magia polimorfizmu:
List<Zwierze> zwierzeta = new List<Zwierze>
{
    new Pies("Burek"),
    new Kot("Mruczek"),
    new Pies("Reksio")
};

foreach (Zwierze z in zwierzeta)
{
    z.DajGlos();
}
// Burek: Hau hau!
// Mruczek: Miau!
// Reksio: Hau hau!
```

Kluczowe jest to, że wywołujemy tę samą metodę `DajGlos()` na typie `Zwierze`, ale dzięki polimorfizmowi każdy obiekt wykonuje **swoją** wersję metody.

#### Klasy abstrakcyjne

W powyższym przykładzie jest pewien problem — nic nie stoi na przeszkodzie, żeby ktoś napisał `new Zwierze("Tajemniczy")`. Ale co to za zwierzę? Jaki głos wydaje? `"..."` to raczej słaba odpowiedź.

Jeśli klasa bazowa ma służyć wyłącznie jako szablon i nie powinna być tworzona bezpośrednio, możemy oznaczyć ją jako **abstrakcyjną** (`abstract`). Metody oznaczone jako `abstract` nie mają ciała — klasy dziedziczące **muszą** je zaimplementować, inaczej kod się nie skompiluje:

```cs
abstract class Zwierze
{
    public string Imie { get; private set; }

    public Zwierze(string imie)
    {
        Imie = imie;
    }

    public abstract void DajGlos();  // brak ciała — tylko deklaracja
}

class Pies : Zwierze
{
    public Pies(string imie) : base(imie) { }

    public override void DajGlos()
    {
        Console.WriteLine($"{Imie}: Hau hau!");
    }
}

class Kot : Zwierze
{
    public Kot(string imie) : base(imie) { }

    public override void DajGlos()
    {
        Console.WriteLine($"{Imie}: Miau!");
    }
}

// Zwierze z = new Zwierze("Tajemniczy"); 
// ❌ błąd kompilacji! Klasa abstrakcyjna.

// Ale lista typu Zwierze nadal działa:
List<Zwierze> zwierzeta = new List<Zwierze>
{
    new Pies("Burek"),
    new Kot("Mruczek")
};

foreach (Zwierze z in zwierzeta)
{
    z.DajGlos();
}
// Burek: Hau hau!
// Mruczek: Miau!
```

| Cecha | `virtual` | `abstract` |
|---|---|---|
| Klasa bazowa ma ciało metody? | Tak (domyślna implementacja) | Nie (tylko deklaracja) |
| Nadpisanie w klasie pochodnej | Opcjonalne | Obowiązkowe |
| Można utworzyć instancję klasy bazowej? | Tak | Nie |
| Kiedy używać? | Gdy domyślne zachowanie ma sens | Gdy klasa bazowa to tylko szablon |

Innymi słowy — `virtual` mówi: „mam swoje zachowanie, ale możesz je zmienić". `abstract` mówi: „sam nie wiem co robić, Ty mi powiedz".

### Interfejsy

Klasa abstrakcyjna mówi: „jestem niedokończonym szablonem, dokończ mnie". Interfejs idzie o krok dalej — to **czysty kontrakt**. Nie mówi czym obiekt *jest*, tylko co *potrafi*. Różnica jak między „jestem ptakiem" a „umiem latać" — bo latać może też samolot, Superman i rzucony laptop.

Interfejs definiujemy słowem kluczowym `interface`. Konwencja w C# nakazuje zaczynać nazwę od wielkiej litery **I** (tak, to nie błąd — to tradycja):

```cs
interface ILatajacy
{
    void Lec();
}

interface IPlywajacy
{
    void Plyn();
}
```

Interfejs nie ma konstruktora, nie ma pól, nie przechowuje stanu — to lista obietnic. Klasa, która implementuje interfejs, **musi** dostarczyć wszystkie zadeklarowane metody:

```cs
class Kaczka : Zwierze, ILatajacy, IPlywajacy
{
    public Kaczka(string imie) : base(imie) { }

    public override void DajGlos()
    {
        Console.WriteLine($"{Imie}: Kwa kwa!");
    }

    public void Lec()
    {
        Console.WriteLine($"{Imie} leci!");
    }

    public void Plyn()
    {
        Console.WriteLine($"{Imie} płynie!");
    }
}

// Użycie:
Kaczka k = new Kaczka("Donald");
k.DajGlos(); // Donald: Kwa kwa!
k.Lec();     // Donald leci!
k.Plyn();    // Donald płynie!
```

Zwróć uwagę — `Kaczka` dziedziczy po klasie `Zwierze` **i jednocześnie** implementuje dwa interfejsy. To kluczowa różnica: dziedziczyć możesz po jednej klasie, ale implementować interfejsów — ile chcesz.


#### Po co interfejsy, skoro jest dziedziczenie?

Wyobraź sobie, że do systemu dochodzi klasa `Samolot`. Samolot lata, ale raczej nie jest zwierzęciem. Bez interfejsów masz problem — nie wpakujesz `Samolot` do hierarchii `Zwierze` tylko dlatego, że lata. Z interfejsem — nie musisz:

```cs
class Samolot : ILatajacy
{
    public string Model { get; private set; }

    public Samolot(string model)
    {
        Model = model;
    }

    public void Lec()
    {
        Console.WriteLine($"{Model} startuje z pasa!");
    }
}

// Teraz możesz traktować wszystko co lata jednakowo:
List<ILatajacy> latajace = new List<ILatajacy>
{
    new Kaczka("Donald"),
    new Samolot("Boeing 737")
};

foreach (ILatajacy obiekt in latajace)
{
    obiekt.Lec();
}
// Donald leci!
// Boeing 737 startuje z pasa!
```

To jest właśnie polimorfizm przez interfejsy — obchodzi nas **co obiekt potrafi**, a nie czym jest. Kaczka i Boeing nie mają ze sobą nic wspólnego poza tym, że oba implementują `ILatajacy`.

W Pythonie osiągnąłbyś to samo przez duck typing (jeśli coś chodzi jak kaczka i kwacze jak kaczka...). Tyle że Python sprawdzi to dopiero w czasie działania programu, a C# — już przy kompilacji.


#### Interfejsy jako typy parametrów

Interfejsy naprawdę błyszczą, gdy używasz ich jako typów w parametrach metod. Dzięki temu metoda akceptuje **cokolwiek, co spełnia kontrakt** — niezależnie od konkretnej klasy:

```cs
void WykonajLot(ILatajacy obiekt)
{
    Console.WriteLine("Przygotowanie do lotu...");
    obiekt.Lec();
    Console.WriteLine("Lądowanie.");
}

WykonajLot(new Kaczka("Donald"));   // działa
WykonajLot(new Samolot("Cessna"));  // też działa
```

To samo podejście znajdziesz w bibliotece standardowej .NET. Np. metoda `Sort()` działa na wszystkim, co implementuje `IComparable<T>` — nie obchodzi jej, czy sortujesz studentów, liczby czy pizze.


#### Domyślna implementacja (C# 8+)

Od C# 8 interfejsy mogą zawierać domyślne implementacje metod. To trochę rozmywa granicę między interfejsem a klasą abstrakcyjną, ale bywa przydatne, gdy chcesz dodać nową metodę do istniejącego interfejsu bez psucia wszystkich klas, które go implementują:

```cs
interface ILatajacy
{
    void Lec();

    void Laduj()  // domyślna implementacja
    {
        Console.WriteLine("Lądowanie standardowe.");
    }
}

// Kaczka nie musi implementować Laduj() — dostaje wersję domyślną
// Ale Samolot może ją nadpisać:
class Samolot : ILatajacy
{
    public string Model { get; private set; }

    public Samolot(string model) { Model = model; }

    public void Lec() => Console.WriteLine($"{Model} startuje!");

    public void Laduj() => Console.WriteLine($"{Model} ląduje na pasie 3.");
}
```

Używaj tego z umiarem — jeśli interfejs zaczyna mieć więcej kodu niż kontraktu, prawdopodobnie powinien być klasą abstrakcyjną.

| Cecha | Klasa abstrakcyjna | Interfejs |
|---|---|---|
| Ile można dziedziczyć/implementować? | Jedną | Wiele |
| Może mieć pola (stan)? | Tak | Nie |
| Może mieć konstruktor? | Tak | Nie |
| Może mieć gotowy kod? | Tak | Od C# 8 (domyślne metody) |
| Modyfikatory dostępu metod? | Dowolne | Domyślnie `public` |
| Kiedy używać? | Wspólny przodek ze wspólną logiką | Wspólna umiejętność niezwiązanych klas |

Zasada kciuka: **klasa abstrakcyjna = „jest czymś"** (Kaczka *jest* Zwierzęciem), **interfejs = „potrafi coś"** (Kaczka *potrafi* latać i pływać). Jeśli nie wiesz co wybrać — zacznij od interfejsu. Zawsze możesz go później wzbogacić o klasę abstrakcyjną, w drugą stronę jest trudniej.

### Pobieranie wartości od użytkownika w konsoli

Już wiesz jak wypisać coś na ekranie — a jak pobrać coś od użytkownika? Do tego służy `Console.ReadLine()`, które zwraca zawsze `string`:

```cs
Console.Write("Jak masz na imię? ");
string imie = Console.ReadLine()!;
Console.WriteLine($"Cześć, {imie}!");

Console.Write("Ile masz lat? ");
int wiek = int.Parse(Console.ReadLine()!);
Console.WriteLine($"Za 10 lat będziesz mieć {wiek + 10} lat.");
```

A po co to ten `!` po `ReadLine()`? `Console.ReadLine()` technicznie może zwrócić `null`, więc kompilator ostrzega nas o tym. 
Operator `!` (null-forgiving operator) mówi kompilatorowi: "spokojnie, wiem co robię, tu nie będzie nulla". W kontekście prostych programów konsolowych jest to bezpieczne założenie.

## Powtórka z Git

Na zajęciach pracujemy z Gitem w następującym schemacie: tworzymy repozytorium (`git init`), dodajemy pliki do śledzenia (`git add`), zapisujemy zmiany (`git commit`), wypychamy je na zdalne repozytorium (`git push`). 

Dla każdego laboratorium tworzymy osobny **branch** (`git checkout -b lab1`), na którym commitujemy postępy. Po zakończeniu zadania otwieramy **pull request** do głównej gałęzi — to właśnie link do niego wrzucamy na Moodle. 

Jeśli pracujecie na kilku urządzeniach, pamiętajcie o `git pull` przed rozpoczęciem pracy, żeby pobrać najnowsze zmiany. 

Warto commitować często i z sensownymi opisami — nie tylko ze względu na punkty za pracę na zajęciach, ale też dlatego, że historia commitów pomaga wrócić do działającej wersji, gdy coś się zepsuje.