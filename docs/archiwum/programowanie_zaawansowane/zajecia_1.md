# Zajęcia 1

## Delegaty

### Czym jest delegat?

Delegat to typ przechowujący referencję do metody. Tak jak `int` trzyma liczbę, delegat trzyma wskaźnik na funkcję o określonej sygnaturze (typy parametrów + typ zwracany). Można go przekazać jako argument, zapisać w zmiennej i wywołać później.

Delegaty to fundament trzech mechanizmów C#:

- **Eventy** (`event`) — delegaty z ograniczonym dostępem (z zewnątrz tylko `+=` i `-=`)
- **LINQ** — metody jak `.Where()`, `.Select()`, `.ForEach()` przyjmują `Func` / `Predicate` / `Action`
- **Programowanie asynchroniczne** — callbacki i `Task`-based patterns

### Własny delegat — słowo kluczowe `delegate`

Definiujemy typ, a potem przypisujemy do niego metody, które pasują sygnaturą:

```csharp
// Metody lokalne
void WyslijEmail(string zamowienie)
{
    Console.WriteLine($"Email: Twoje zamówienie '{zamowienie}' przyjęte!");
}

void WyslijSMS(string zamowienie)
{
    Console.WriteLine($"SMS: Zamówienie '{zamowienie}' w realizacji.");
}

void ZapiszDoLoga(string zamowienie)
{
    Console.WriteLine($"Log: [{DateTime.Now:HH:mm:ss}] Nowe zamówienie: {zamowienie}");
}

var sklep = new SklepInternetowy();

sklep.Powiadom = WyslijEmail;       // przypisanie
sklep.Powiadom += WyslijSMS;        // += dodaje kolejną metodę (multicast)
sklep.Powiadom += ZapiszDoLoga;

sklep.ZlozZamowienie("Shure SM7B");

sklep.Powiadom -= WyslijSMS;        // -= odpina metodę
sklep.ZlozZamowienie("Kabel XLR");


// --- Deklaracje typów muszą być pod top-level statements ---

delegate void PowiadomOZamowieniu(string zamowienie);

class SklepInternetowy
{
    public PowiadomOZamowieniu? Powiadom;   // ? — pole może być null

    public void ZlozZamowienie(string produkt)
    {
        Console.WriteLine($"Zamówienie złożone: {produkt}");
        Powiadom?.Invoke(produkt);          // ?. — wywołaj tylko jeśli nie-null
    }
}
```

**Kluczowe elementy:**

- `delegate void PowiadomOZamowieniu(string)` — definiuje typ (sygnatura: `string` → `void`)
- `+=` / `-=` — delegaty mogą trzymać wiele metod naraz (multicast)
- `?.Invoke()` — bezpieczne wywołanie z zabezpieczeniem przed `null`
- `?` przy typie — nullable reference type, jawna zgoda na `null`

### Action, Func, Predicate — wbudowane delegaty

Zamiast definiować własne typy delegatów, w 99% przypadków wystarczą gotowce:

| Typ | Sygnatura | Zastosowanie |
|---|---|---|
| `Action<T>` | parametry → `void` | "zrób coś" — loguj, wyświetl, wyślij |
| `Func<T, TResult>` | parametry → wartość (ostatni typ = zwracany) | "oblicz/przekształć" — mapowanie, formatowanie |
| `Predicate<T>` | parametr → `bool` (= `Func<T, bool>`) | "czy spełnia warunek?" — filtrowanie, walidacja |

Warianty `Action` i `Func` obsługują od 0 do 16 parametrów:

```csharp
Action                        // () => void
Action<string>                // (string) => void
Action<string, int>           // (string, int) => void
Func<int>                     // () => int
Func<string, int>             // (string) => int
Func<string, string, bool>    // (string, string) => bool
Predicate<string>             // (string) => bool  ≡  Func<string, bool>
```

### Przykład — Action, Func, Predicate w praktyce

```csharp
var sklep = new SklepInternetowy();

sklep.DodajProdukt("Shure SM7B", 1899.00m);
sklep.DodajProdukt("Kabel XLR", 49.99m);
sklep.DodajProdukt("Interface Focusrite", 899.00m);
sklep.DodajProdukt("Pop filtr", 29.99m);
sklep.DodajProdukt("Statyw mikrofonowy", 149.99m);

// ACTION — wykonaj operację na każdym produkcie (void)
sklep.DlaKazdegoProduktu(p =>
    Console.WriteLine($"  📦 {p.Nazwa} — {p.Cena:C}"));

// PREDICATE — filtruj według warunku (bool)
var drogie = sklep.Filtruj(p => p.Cena > 100);
foreach (var p in drogie)
    Console.WriteLine($"  💰 {p.Nazwa}");

// FUNC — przekształć produkt w string (zwraca wartość)
var etykiety = sklep.Transformuj(p => $"{p.Nazwa.ToUpper()} [{p.Cena:F2} PLN]");
foreach (var e in etykiety)
    Console.WriteLine($"  🏷️ {e}");


// --- Typy ---

record Produkt(string Nazwa, decimal Cena);

class SklepInternetowy
{
    private readonly List<Produkt> _produkty = [];

    public void DodajProdukt(string nazwa, decimal cena)
        => _produkty.Add(new Produkt(nazwa, cena));

    public void DlaKazdegoProduktu(Action<Produkt> akcja)
    {
        foreach (var p in _produkty)
            akcja(p);
    }

    public List<Produkt> Filtruj(Predicate<Produkt> warunek)
    {
        List<Produkt> wynik = [];
        foreach (var p in _produkty)
            if (warunek(p))
                wynik.Add(p);
        return wynik;
    }

    public List<string> Transformuj(Func<Produkt, string> transformacja)
    {
        List<string> wynik = [];
        foreach (var p in _produkty)
            wynik.Add(transformacja(p));
        return wynik;
    }
}
```

### Wyrażenia lambda — skrócony zapis

Lambdy to anonimowe metody przypisywane do delegatów:

```csharp
Action<string> powitanie = (imie) => Console.WriteLine($"Cześć {imie}!");
Func<int, int, int> dodaj = (a, b) => a + b;
Predicate<int> czyParzysta = n => n % 2 == 0;
```

## Wyrażenia Lambda

### Czym jest lambda?

Lambda to anonimowa funkcja — metoda bez nazwy, którą zapisujecie w jednej linii i przypisujecie do delegata. Zamiast definiować osobną metodę i przekazywać jej nazwę, piszecie ciało funkcji w miejscu użycia. Składnia to `(parametry) => wyrażenie` — operator `=>` czytamy jako "goes to" lub po prostu "strzałka".

Lambdy nie istnieją same z siebie — zawsze "lądują" w jakimś delegacie (`Action`, `Func`, `Predicate` lub własnym). Kompilator sam dopasowuje typy parametrów na podstawie kontekstu, dlatego zwykle nie trzeba ich podawać jawnie:

```csharp
// Pełna forma — typy podane jawnie
Func<int, int, int> dodaj = (int a, int b) => a + b;

// Skrócona — kompilator wywnioskuje typy z Func<int, int, int>
Func<int, int, int> dodaj2 = (a, b) => a + b;

// Jeden parametr — można pominąć nawiasy
Predicate<string> niepuste = s => s.Length > 0;

// Brak parametrów — puste nawiasy obowiązkowe
Action info = () => Console.WriteLine("Gotowe!");
```

### Lambda wyrażeniowa vs. lambda blokowa

Gdy ciało funkcji to jedno wyrażenie, wystarczy prawa strona strzałki — wartość jest automatycznie zwracana (nie piszemy `return`):

```csharp
Func<double, double> kwadrat = x => x * x;
```

Gdy potrzebujecie kilku instrukcji, dodajecie klamry `{}` i wtedy musicie użyć jawnego `return` (jeśli delegat zwraca wartość):

```csharp
Func<string, string> formatuj = nazwa =>
{
    var trimmed = nazwa.Trim();
    var upper = trimmed.ToUpper();
    return $"[{upper}]";
};
```

Zasada kciuka — jeśli lambda blokowa rozrasta się powyżej 3–4 linii, lepiej wyciągnąć ją do nazwanej metody. Lambda ma służyć zwięzłości, a nie zaciemnianiu kodu.

### Domknięcie (closure) — lambda "przechwytuje" zmienne

Lambda ma dostęp do zmiennych z otaczającego zakresu (scope). To nazywamy domknięciem — lambda "zamyka" nad zmienną i korzysta z niej w momencie wywołania, nie w momencie definicji:

```csharp
decimal minKwota = 100;
Predicate<Order> drogie = o => o.TotalAmount > minKwota;

minKwota = 500; // zmiana PRZED wywołaniem lambdy
// drogie teraz filtruje > 500, nie > 100!
```

To potężny mechanizm, ale wymaga świadomości — lambda nie kopiuje wartości zmiennej, tylko trzyma referencję do niej. Klasyczna pułapka to domknięcie nad zmienną pętli:

```csharp
var akcje = new List<Action>();
for (int i = 0; i < 3; i++)
    akcje.Add(() => Console.WriteLine(i));

foreach (var a in akcje) a();
// Wypisze: 3, 3, 3 — bo wszystkie lambdy widzą TĘ SAMĄ zmienną i,
// która po pętli ma wartość 3

// Fix — lokalna kopia:
for (int i = 0; i < 3; i++)
{
    int kopia = i;
    akcje.Add(() => Console.WriteLine(kopia));
}
```

### Gdzie lambdy pojawiają się w praktyce?

Lambdy to "klej" między waszym kodem a bibliotekami. Prawie wszystkie metody kolekcji i LINQ przyjmują delegaty, a wygodnie jest je przekazywać właśnie jako lambdy:

```csharp
var produkty = new List<string> { "Shure SM7B", "Kabel XLR", "Pop filtr" };

// List<T>.Find — przyjmuje Predicate<T>
var wynik = produkty.Find(p => p.Contains("XLR"));

// List<T>.ForEach — przyjmuje Action<T>
produkty.ForEach(p => Console.WriteLine(p.ToUpper()));

// List<T>.ConvertAll — przyjmuje Func<T, TResult> (pod spodem Converter<T,TResult>)
var dlugosci = produkty.ConvertAll(p => p.Length);
```

Lambdy to pomost między delegatami (które poznaliście na początku zajęć) a LINQ (który zaraz przed wami). Delegat definiuje *kontrakt* — jaką funkcję można przekazać. Lambda to *najwygodniejszy sposób* na spełnienie tego kontraktu w miejscu wywołania.

---

## LINQ — Language Integrated Query

### Czym jest LINQ?

LINQ (Language Integrated Query) to zestaw metod rozszerzających i składnia zapytań wbudowana w C#, która pozwala odpytywać kolekcje (listy, tablice, słowniki, a później też bazy danych i XML) w jednolity, deklaratywny sposób. Zamiast pisać pętle `for`/`foreach` z warunkami, mówicie *co* chcecie dostać, a nie *jak* to iterować.

LINQ działa na wszystkim, co implementuje `IEnumerable<T>` — czyli praktycznie na każdej kolekcji w .NET. Metody LINQ zwracają nowe sekwencje (nie modyfikują oryginału) i są leniwie ewaluowane — wykonują się dopiero, gdy ktoś poprosi o wyniki (np. `foreach`, `ToList()`, `Count()`).

### Dwie składnie — ten sam silnik

C# oferuje dwa sposoby pisania zapytań LINQ:

**Składnia metod (method syntax)** — łańcuch wywołań metod rozszerzających z lambda jako argumentami:

```csharp
var drogie = produkty
    .Where(p => p.Cena > 100)
    .OrderByDescending(p => p.Cena)
    .Select(p => new { p.Nazwa, p.Cena });
```

**Składnia zapytań (query syntax)** — SQL-podobna składnia z `from`, `where`, `select`:

```csharp
var drogie = from p in produkty
             where p.Cena > 100
             orderby p.Cena descending
             select new { p.Nazwa, p.Cena };
```

Obie formy kompilują się do tego samego kodu — kompilator zamienia query syntax na wywołania metod. Składnia metod jest bardziej elastyczna (ma więcej operatorów), a składnia zapytań bywa czytelniejsza przy joinach i grupowaniach. W praktyce często mieszacie obie — to jest jak najbardziej idiomatyczne.

### Najważniejsze operatory — filtrowanie, projekcja, sortowanie

**Where** — filtruje elementy po warunku. Przyjmuje `Func<T, bool>` (w praktyce to ten sam kontrakt co `Predicate<T>`, ale LINQ używa `Func`):

```csharp
var elektronika = produkty.Where(p => p.Category == "Electronics");

// query syntax:
var elektronika2 = from p in produkty
                   where p.Category == "Electronics"
                   select p;
```

**Select** — projekcja, czyli transformacja każdego elementu na nowy kształt. Przyjmuje `Func<T, TResult>`:

```csharp
var nazwy = produkty.Select(p => p.Nazwa);
var summary = produkty.Select(p => new { p.Nazwa, CenaBrutto = p.Cena * 1.23m });

// query syntax:
var summary2 = from p in produkty
               select new { p.Nazwa, CenaBrutto = p.Cena * 1.23m };
```

**OrderBy / OrderByDescending / ThenBy** — sortowanie. `ThenBy` dodaje drugorzędne kryterium:

```csharp
var posortowane = produkty
    .OrderBy(p => p.Category)
    .ThenByDescending(p => p.Cena);

// query syntax:
var posortowane2 = from p in produkty
                   orderby p.Category, p.Cena descending
                   select p;
```

### Grupowanie — GroupBy

`GroupBy` dzieli kolekcję na grupy po kluczu. Wynikiem jest kolekcja `IGrouping<TKey, TElement>` — każda grupa ma `.Key` i jest sama w sobie `IEnumerable<T>`:

```csharp
var grupyKategorie = produkty
    .GroupBy(p => p.Category);

foreach (var grupa in grupyKategorie)
{
    Console.WriteLine($"Kategoria: {grupa.Key}");
    foreach (var p in grupa)
        Console.WriteLine($"  - {p.Nazwa}: {p.Cena:C}");
}

// query syntax:
var grupy2 = from p in produkty
             group p by p.Category into g
             select new { Kategoria = g.Key, Ile = g.Count(), Suma = g.Sum(p => p.Cena) };
```

Zwróćcie uwagę na `into g` w query syntax — po `group by` tworzy się nowa zmienna zakresowa (`g`), na której możecie dalej operować. W method syntax tę rolę pełni po prostu kolejne `.Select()`.

### Joiny — łączenie kolekcji

Gdy macie dane w osobnych kolekcjach (np. zamówienia i klienci z osobnych list), potrzebujecie joina — łączenia po wspólnym kluczu.

**Join (inner join)** — zwraca tylko pary, które mają dopasowanie w obu kolekcjach:

```csharp
var zamowieniaZKlientami = zamowienia
    .Join(klienci,
        z => z.CustomerId,       // klucz z lewej
        k => k.Id,               // klucz z prawej
        (z, k) => new            // co zrobić z parą
        {
            z.Id,
            Klient = k.FullName,
            z.TotalAmount
        });

// query syntax — tu join jest znacznie czytelniejszy:
var zamowienia2 = from z in zamowienia
                  join k in klienci on z.CustomerId equals k.Id
                  select new { z.Id, Klient = k.FullName, z.TotalAmount };
```

**GroupJoin (left join pattern)** — każdemu elementowi z lewej przypisuje kolekcję dopasowań z prawej (może być pusta — stąd "left join"):

```csharp
var klienciZZamowieniami = klienci
    .GroupJoin(zamowienia,
        k => k.Id,
        z => z.CustomerId,
        (k, ordery) => new
        {
            Klient = k.FullName,
            LiczbaZamowien = ordery.Count(),
            Suma = ordery.Sum(z => z.TotalAmount)
        });

// query syntax:
var klienci2 = from k in klienci
               join z in zamowienia on k.Id equals z.CustomerId into ordery
               select new { Klient = k.FullName, Ile = ordery.Count() };
```

To właśnie `into` po `join` tworzy `GroupJoin` — kluczowy pattern, który w query syntax jest dużo bardziej naturalny niż w method syntax.

### Spłaszczanie — SelectMany

Gdy każdy element kolekcji sam zawiera kolekcję (np. zamówienie ma listę pozycji), `SelectMany` "spłaszcza" zagnieżdżoną strukturę do jednej płaskiej sekwencji:

```csharp
// Chcemy listę WSZYSTKICH pozycji ze WSZYSTKICH zamówień
var wszystkiePozycje = zamowienia
    .SelectMany(z => z.Items, 
        (zamowienie, pozycja) => new 
        { 
            zamowienie.Id, 
            pozycja.ProductId, 
            pozycja.TotalPrice 
        });

// query syntax — tu SelectMany pojawia się niejawnie przez podwójne "from":
var pozycje2 = from z in zamowienia
               from item in z.Items
               select new { z.Id, item.ProductId, item.TotalPrice };
```

Podwójne `from` w query syntax to jeden z najczęstszych powodów, dla których ludzie sięgają po tę składnię — jest po prostu czytelniejsze od jawnego `SelectMany`.

### Agregacje i operatory elementów

LINQ oferuje zestaw metod kończących łańcuch — te metody **wymuszają natychmiastowe wykonanie** (nie są leniwe):

```csharp
var suma = zamowienia.Sum(z => z.TotalAmount);
var srednia = zamowienia.Average(z => z.TotalAmount);
var max = zamowienia.Max(z => z.TotalAmount);
var ile = zamowienia.Count(z => z.Status == Status.Completed);
var jest = zamowienia.Any(z => z.TotalAmount > 10000);
var wszystkie = zamowienia.All(z => z.Items.Count > 0);
```

Operatory elementów pobierają pojedynczy obiekt:

```csharp
var pierwszy = zamowienia.First(z => z.Status == Status.New);
var alboNull = zamowienia.FirstOrDefault(z => z.Status == Status.New);
var najdrozsze = zamowienia.MaxBy(z => z.TotalAmount);  // .NET 6+
```

Różnica między `First` a `FirstOrDefault` — `First` rzuca wyjątek, gdy nie ma wyniku, `FirstOrDefault` zwraca `null` (dla typów referencyjnych) lub wartość domyślną.

### Leniwa ewaluacja — LINQ nie robi nic, dopóki nie musi

To kluczowa koncepcja. Większość operatorów LINQ (`Where`, `Select`, `OrderBy`, `GroupBy`) tworzy tylko **opis zapytania** — nie iteruje jeszcze po kolekcji. Wykonanie następuje dopiero przy materializacji:

```csharp
var zapytanie = produkty.Where(p => p.Cena > 100); // ← jeszcze NIC się nie wykonało

var lista = zapytanie.ToList();     // ← TUTAJ iteruje i filtruje
foreach (var p in zapytanie) { }    // ← albo TUTAJ
var ile = zapytanie.Count();        // ← albo TUTAJ
```

Ma to dwie konsekwencje. Po pierwsze, jeśli potrzebujecie wyniku wielokrotnie, zmaterializujcie go raz przez `ToList()` — inaczej zapytanie wykona się od nowa za każdym razem. Po drugie, możecie budować zapytanie krok po kroku i dołączać kolejne warunki dynamicznie (to właśnie wykorzystuje `WhereIf` z zadania bonusowego).

### Słowo kluczowe `let` w query syntax

`let` pozwala zdefiniować zmienną pomocniczą wewnątrz zapytania — przydatne, gdy obliczona wartość jest używana w kilku miejscach:

```csharp
var raport = from z in zamowienia
             let kwota = z.Items.Sum(i => i.TotalPrice)
             let czyDuze = kwota > 1000
             where czyDuze
             orderby kwota descending
             select new { z.Id, kwota, Rozmiar = czyDuze ? "Duże" : "Małe" };
```

W method syntax nie ma bezpośredniego odpowiednika `let` — trzeba użyć dodatkowego `Select`, który dorzuca obliczoną wartość jako pole anonimowego typu, a potem kontynuować łańcuch. To jeden z przypadków, gdzie query syntax jest czytelniejsza.

### Zip i Aggregate — rzadziej używane, ale warto znać

**Zip** — łączy dwie kolekcje element po elemencie (jak zamek błyskawiczny):

```csharp
var nazwy = new[] { "Shure SM7B", "Kabel XLR", "Pop filtr" };
var ceny = new[] { 1899.00m, 49.99m, 29.99m };

var produkty = nazwy.Zip(ceny, (n, c) => new { Nazwa = n, Cena = c });
// Wynik: { Shure SM7B, 1899 }, { Kabel XLR, 49.99 }, { Pop filtr, 29.99 }
```

**Aggregate** — najbardziej ogólna redukcja kolekcji do pojedynczej wartości (jak `reduce` w JavaScript/Pythonie):

```csharp
var koszyk = new[] { 100m, 200m, 300m };

// Suma z 5% rabatem kumulowanym na każdą kolejną pozycję
var total = koszyk.Aggregate(
    (dotychczas, kolejna) => dotychczas + kolejna * 0.95m);
// 100 + 200*0.95 + 300*0.95 = 100 + 190 + 285 = 575

// Z wartością początkową (seed):
var zSeedem = koszyk.Aggregate(0m, 
    (akumulator, pozycja) => akumulator + pozycja * 1.23m);
```

`Aggregate` to "nóż szwajcarski" — `Sum`, `Min`, `Max`, `Count` to tak naprawdę jego specjalizowane wersje. W codziennym kodzie używacie tych specjalizacji, ale `Aggregate` przydaje się, gdy potrzebujecie niestandardowej logiki redukcji.

### Method syntax vs. query syntax — kiedy co?

Nie ma jednej "poprawnej" składni. Ogólne wytyczne:

Składnia metod sprawdza się lepiej, gdy łańcuch jest prosty (filtr → sortuj → weź), gdy używacie operatorów bez odpowiednika w query syntax (`Distinct`, `Take`, `Skip`, `Zip`, `Aggregate`, `Any`, `All`) i gdy korzystacie z metod zwracających pojedyncze wartości (`First`, `Count`, `Sum`).

Składnia zapytań jest czytelniejsza przy joinach (jedno `join ... on ... equals` vs. rozbudowany `Join()` z czterema lambdami), przy grupowaniach z dalszym przetwarzaniem (dzięki `into`), przy wielu zmiennych zakresowych (podwójne `from` zamiast `SelectMany`) i przy obliczeniach pośrednich (`let`).

Mieszanie obu składni w jednym zapytaniu jest normalne — otaczacie fragment query syntax nawiasem i dokładacie `.Count()` czy `.ToList()`:

```csharp
var wynik = (from z in zamowienia
             join k in klienci on z.CustomerId equals k.Id
             where k.IsVip
             select z.TotalAmount).Average();
```

---

## Laboratorium 1 — Model domenowy i operacje na kolekcjach

**Projekt:** OrderFlow — System przetwarzania zamówień

**Tematy:** Delegaty, wyrażenia lambda, Action/Func/Predicate, LINQ (składnia zapytań i metod)

### Kontekst projektu

Przez 5 zajęć budujecie system **OrderFlow** — aplikację przetwarzającą zamówienia klientów. Zaczynamy od modelu domenowego i operacji na kolekcjach in-memory, które w kolejnych tygodniach rozbudujecie o zdarzenia, asynchroniczność, pliki, bazę danych i testy.

Pracujcie w solucji konsolowej .NET 8+. Zalecana struktura:

```
OrderFlow/
├── OrderFlow.sln
└── OrderFlow.Console/
    ├── Models/
    ├── Services/
    ├── Data/
    └── Program.cs
```

---

### Zadanie 1 — Model domenowy i dane testowe (3 pkt)

Utwórzcie klasy: **Product**, **Customer**, **Order**, **OrderItem** z sensownymi właściwościami (cena, ilość, kategoria, status itp.). Order powinien mieć właściwość obliczaną `TotalAmount`, a OrderItem — `TotalPrice`.

Status zamówienia zróbcie jako enum (`New`, `Validated`, `Processing`, `Completed`, `Cancelled`).

W klasie **SampleData** przygotujcie statyczne listy z przykładowymi danymi — minimum 5 produktów (różne kategorie), 4 klientów (w tym min. 1 VIP) i 6 zamówień. Dane powinny być na tyle zróżnicowane, żeby zapytania z dalszych zadań dawały ciekawe wyniki.

---

### Zadanie 2 — Delegaty i walidacja zamówień (4 pkt)

Zbudujcie klasę `OrderValidator` z pipeline'em reguł walidacyjnych.

Wymagania:

- Zdefiniujcie **własny delegat** `ValidationRule` (z parametrem `out string errorMessage`).
- Zaimplementujcie minimum **3 reguły jako named methods** (np. zamówienie musi mieć pozycje, kwota nie przekracza limitu, ilości > 0).
- Dodajcie drugi mechanizm walidacji oparty na **`Func<Order, bool>`** z minimum **2 regułami jako lambdy** (np. data nie z przyszłości, status nie jest `Cancelled`).
- Metoda `ValidateAll` powinna łączyć wyniki obu mechanizmów.

Pokażcie w konsoli walidację zamówienia poprawnego i jednego łamiącego kilka reguł.

---

### Zadanie 3 — Action, Func, Predicate (4 pkt)

Zbudujcie klasę `OrderProcessor` operującą na liście zamówień. Zaimplementujcie metody wykorzystujące generyczne delegaty:

- **`Predicate<Order>`** — filtrowanie zamówień (min. 3 różne predykaty jako lambdy).
- **`Action<Order>`** — wykonanie akcji na zamówieniach (min. 2 zastosowania, np. wypisanie, zmiana statusu).
- **`Func<Order, T>`** — generyczna projekcja zamówień na dowolny typ (pokażcie użycie z typem anonimowym).
- **Agregacja** — metoda przyjmująca `Func<IEnumerable<Order>, decimal>` wywołana z min. 3 agregatorami (suma, średnia, max).

Na koniec zbudujcie **łańcuch**: filtruj → sortuj → weź top N → wypisz, używając `Predicate`, `Func` i `Action` w jednym flow.

---

### Zadanie 4 — LINQ (4 pkt)

Zaimplementujcie zapytania LINQ w **obu składniach** (method syntax i query syntax). Minimum 6 zapytań, w tym obowiązkowo:

- Join zamówień z klientami (np. grupowanie zamówień po mieście klienta).
- Spłaszczenie z `SelectMany` (np. Order → OrderItems → Product).
- `GroupBy` z agregacją (np. top klientów wg kwoty, średnia wartość per kategoria).
- Jedno zapytanie z `GroupJoin` (left join pattern).
- Jedno zapytanie łączące obie składnie (mixed syntax), np. raport per klient z ulubioną kategorią.

Nad każdym zapytaniem dodajcie **krótki komentarz** opisujący co robi i dlaczego wybraliście daną składnię. Wyniki wypisujcie do konsoli.

---

### Punktacja

| Zadanie | Punkty |
|---|---|
| Wysłanie zadania na GitHub z commitem z zajęć | **5 pkt** |
| 1. Model domenowy + dane testowe | 3 pkt |
| 2. Delegaty i walidacja | 4 pkt |
| 3. Action, Func, Predicate | 4 pkt |
| 4. LINQ (method + query syntax) | 4 pkt |
| **Razem** | **20 pkt** |

> **Uwaga o GitHub:** Aby otrzymać 5 pkt za wysłanie, musicie mieć **co najmniej 1 commit z czasów zajęć** (nie musi być ukończony kod — liczy się systematyczna praca). Zadanie można dokończyć po zajęciach, ale finalna wersja też musi być na repozytorium.

### Wskazówki

- Zacznijcie od zadania 1 i dobrych danych testowych — reszta zadań z nich korzysta.
- Zwróćcie uwagę na różnicę między `delegate` a `Func` — oba działają, ale mają inne zastosowania.
- W LINQ pamiętajcie, że `join` w query syntax to odpowiednik `Join`/`GroupJoin` w method syntax, a `let` nie ma bezpośredniego odpowiednika.
- Kod z dzisiejszych zajęć będzie **rozbudowywany** na kolejnych — zadbajcie o czytelną strukturę.