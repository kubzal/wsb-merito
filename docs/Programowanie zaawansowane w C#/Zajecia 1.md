# Zajęcia 1
Drodzy Państwo! Nasza przygoda z C# właśnie się rozpoczyna! Jeżeli Państwo nie mają nic przeciwko, ze względów czysto praktycznych (i redukujących ilość zbędnego tekstu) w dalszej części naszych materiałów będę sobie pozwalał na bardziej bezpośrednią formę oraz zwracanie się w liczbie pojedynczej. 
Zakładam, że się Państwo zgodzili 😉

Przed wyruszeniem w dalszą drogę upewnij się, że masz przygotowane środowisko programistyczne (szczegóły we [wprowadzeniu](README.md)).

Jeżeli wszystko gotowe, to możemy ruszać!

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

