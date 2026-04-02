# Zajęcia 2

## Zdarzenia, asynchroniczność, wielowątkowość

### Część 1: Zdarzenia niestandardowe (EventArgs)

#### Od czego zaczynamy

Na poprzednich zajęciach poznaliście delegaty — zmienne, które przechowują metody. Dziś idziemy krok dalej: co jeśli chcemy, żeby **wiele miejsc w programie** mogło zareagować na jedno zdarzenie?

#### Problem: sklep i powiadomienia

Wyobraźcie sobie, że macie klasę `Sklep`. Po złożeniu zamówienia chcecie:

- wysłać email do klienta,
- powiadomić magazyn,
- zaktualizować statystyki.

Moglibyście przekazać callback:

```csharp
void ZlozZamowienie(string produkt, Action<string> callback)
{
    Console.WriteLine($"Zapisuję: {produkt}");
    callback($"Zamówienie na {produkt}");
}
```

Ale to obsługuje **jednego** odbiorcę. Żeby obsłużyć wielu, musielibyście ręcznie trzymać listę callbacków. C# ma na to wbudowany mechanizm — **zdarzenie**.

#### Zdarzenie = lista callbacków wbudowana w język

```csharp
public class Sklep
{
    public event Action<string>? NoweZamowienie;

    public void ZlozZamowienie(string produkt)
    {
        Console.WriteLine($"Zapisuję: {produkt}");
        NoweZamowienie?.Invoke($"Zamówienie na {produkt}");
    }
}
```

Teraz różne moduły mogą się „wpiąć" za pomocą `+=`:

```csharp
var sklep = new Sklep();

sklep.NoweZamowienie += msg => Console.WriteLine($"  EMAIL: {msg}");
sklep.NoweZamowienie += msg => Console.WriteLine($"  MAGAZYN: {msg}");
sklep.NoweZamowienie += msg => Console.WriteLine($"  STATYSTYKI: {msg}");

sklep.ZlozZamowienie("Laptop");
```

Wynik:

```
Zapisuję: Laptop
  EMAIL: Zamówienie na Laptop
  MAGAZYN: Zamówienie na Laptop
  STATYSTYKI: Zamówienie na Laptop
```

Sklep **nie wie** ilu ma subskrybentów i co oni robią. Po prostu woła `Invoke` i kto się wpiął, ten dostaje wiadomość. To jest wzorzec **publish/subscribe** — ten sam, który w architekturze mikroserwisów realizuje np. Kafka, tylko tutaj działa w ramach jednego programu.

#### Problem: jeden string to za mało

W powyższym przykładzie przekazujemy tylko tekst. Ale magazyn potrzebuje wiedzieć **ile sztuk**, faktura potrzebuje **cenę**, email potrzebuje **adres klienta**. Moglibyśmy dodawać parametry:

```csharp
public event Action<string, int, decimal, string>? NoweZamowienie;
//                  produkt  ilość cena    email   ← bałagan
```

Ale za miesiąc dojdzie kolejne pole i kolejne — nie wiadomo co jest co. Naturalne rozwiązanie: **pakujemy dane do klasy**.

#### EventArgs — konwencja .NET na dane zdarzenia

```csharp
// Klasa z danymi zdarzenia — dziedziczy po EventArgs (konwencja)
public class ZamowienieEventArgs : EventArgs
{
    public string Produkt { get; }
    public int Ilosc { get; }
    public decimal Cena { get; }

    public ZamowienieEventArgs(string produkt, int ilosc, decimal cena)
    {
        Produkt = produkt;
        Ilosc = ilosc;
        Cena = cena;
    }
}
```

Dziedziczenie po `EventArgs` nie dodaje żadnej magii — to **konwencja**, dzięki której każdy programista C# widząc `XxxEventArgs` od razu wie, że to dane zdarzenia.

#### EventHandler&lt;T&gt; — standardowy delegat dla zdarzeń

Zamiast `Action<T>` używamy `EventHandler<T>`. Różnica? `EventHandler` zawsze ma dwa parametry:

```csharp
// EventHandler<T> to w praktyce:
// delegate void EventHandler<T>(object? sender, T args);
//                                ↑ KTO wysłał   ↑ DANE
```

Ten `sender` to bonus — subskrybent może sprawdzić, **kto** wywołał zdarzenie (przydatne gdy nasłuchujesz zdarzeń z wielu źródeł).

##### Demo

```csharp
// ═══════════════ DANE ZDARZENIA ═══════════════
public class ZamowienieEventArgs : EventArgs
{
    public string Produkt { get; }
    public int Ilosc { get; }
    public decimal Cena { get; }
    public DateTime Kiedy { get; } = DateTime.Now;

    public ZamowienieEventArgs(string produkt, int ilosc, decimal cena)
    {
        Produkt = produkt;
        Ilosc = ilosc;
        Cena = cena;
    }
}

// ═══════════════ NADAWCA (publisher) ═══════════════
public class Sklep
{
    public event EventHandler<ZamowienieEventArgs>? NoweZamowienie;

    public void ZlozZamowienie(string produkt, int ilosc, decimal cena)
    {
        Console.WriteLine($"[Sklep] Przyjmuję zamówienie: {ilosc}x {produkt}");
        NoweZamowienie?.Invoke(this, new ZamowienieEventArgs(produkt, ilosc, cena));
    }
}

// ═══════════════ SUBSKRYBENCI ═══════════════
public class Magazyn
{
    public void Obsluz(object? sender, ZamowienieEventArgs e)
    {
        Console.WriteLine($"  [Magazyn] Pakuję {e.Ilosc}x {e.Produkt}");
    }
}

public class Ksiegowosc
{
    public void Obsluz(object? sender, ZamowienieEventArgs e)
    {
        decimal kwota = e.Ilosc * e.Cena;
        Console.WriteLine($"  [Księgowość] Faktura na {kwota:C}");
    }
}

// ═══════════════ UŻYCIE ═══════════════
var sklep = new Sklep();
var magazyn = new Magazyn();
var ksiegowosc = new Ksiegowosc();

// Subskrypcja — każdy moduł sam się rejestruje
sklep.NoweZamowienie += magazyn.Obsluz;
sklep.NoweZamowienie += ksiegowosc.Obsluz;

// Można też lambdą (np. logowanie)
sklep.NoweZamowienie += (s, e) =>
    Console.WriteLine($"  [Log] {e.Kiedy:HH:mm:ss} — {e.Produkt}");

sklep.ZlozZamowienie("Klawiatura", 3, 149.99m);
Console.WriteLine();
sklep.ZlozZamowienie("Monitor", 1, 1299.00m);
```

Wynik:

```
[Sklep] Przyjmuję zamówienie: 3x Klawiatura
  [Magazyn] Pakuję 3x Klawiatura
  [Księgowość] Faktura na 449,97 zł
  [Log] 14:32:07 — Klawiatura

[Sklep] Przyjmuję zamówienie: 1x Monitor
  [Magazyn] Pakuję 1x Monitor
  [Księgowość] Faktura na 1 299,00 zł
  [Log] 14:32:07 — Monitor
```

#### Podsumowanie — mapa pojęć

```
delegat            → typ zmiennej przechowującej metodę
Action<T>          → gotowy delegat: przyjmuje T, zwraca void
EventHandler<T>    → gotowy delegat: przyjmuje (object? sender, T args)
event              → lista delegatów z += / -=
EventArgs          → bazowa klasa na dane zdarzenia (konwencja)
XxxEventArgs       → Twoja klasa z konkretnymi danymi
?.Invoke(...)      → bezpieczne wywołanie (null gdy brak subskrybentów)
```

### Część 2: Programowanie asynchroniczne (async/await)

#### Problem: kucharz gapiący się na piekarnik

Wyobraźcie sobie pizzerię z jednym kucharzem. Wkłada Margheritę do pieca i **stoi i gapi się przez 10 minut**. Kolejka rośnie. A mógłby w tym czasie przygotować następną pizzę.

To jest kod synchroniczny — program czeka na operację, nie robiąc nic:

```csharp
string PobierzDane()
{
    Thread.Sleep(3000);  // wątek STOI przez 3 sekundy
    return "dane z serwera";
}

var wynik = PobierzDane();  // cały program zamrożony na 3s
```

#### Gdzie to naprawdę boli?

- **Aplikacja okienkowa (WPF/WinForms)** — klikasz przycisk „Pobierz dane". Przez 3 sekundy okno jest zamrożone, nie da się go nawet przeciągnąć. Windows wyświetla komunikat „Nie odpowiada".
- **Serwer webowy (ASP.NET)** — przychodzi request, handler czeka na bazę danych. Wątek jest zajęty *czekaniem na nic*. Przy 1000 równoczesnych requestów serwer pada, bo wątki się skończyły — a każdy z nich po prostu stał bezczynnie.

Problem nie jest w tym, że operacja trwa długo. Problem w tym, że **wątek nie robi nic użytecznego podczas czekania**.

#### async/await = „daj mi znać jak będzie gotowe"

```csharp
async Task<string> PobierzDaneAsync()
{
    await Task.Delay(3000);  // wątek jest WOLNY przez te 3 sekundy
    return "dane z serwera";
}
```

Przez te 3 sekundy wątek może obsługiwać inne requesty, odświeżać UI, robić cokolwiek innego. Gdy `Task.Delay` się skończy, program wraca tam, gdzie było `await`, i kontynuuje.

#### Słowniczek

```
async       →  oznacza: "ta metoda może używać await wewnątrz"
await       →  oznacza: "tu czekam, ale NIE blokuję wątku"
Task        →  obietnica: "kiedyś skończę" (jak Future w Pythonie)
Task<T>     →  obietnica: "kiedyś zwrócę wynik typu T"
```

#### Demo 1: różnica sync vs async

```csharp
using System.Diagnostics;

// Symulacja pobierania danych (np. HTTP request do API)
async Task<string> PobierzAsync(string url)
{
    Console.WriteLine($"  Zaczynam pobierać: {url}");
    await Task.Delay(2000);  // symulacja 2s opóźnienia sieciowego
    Console.WriteLine($"  Gotowe: {url}");
    return $"dane z {url}";
}

var sw = Stopwatch.StartNew();

// ═══ SEKWENCYJNIE ═══
Console.WriteLine("=== Sekwencyjnie ===");
sw.Restart();
var a = await PobierzAsync("api/users");
var b = await PobierzAsync("api/orders");
var c = await PobierzAsync("api/products");
Console.WriteLine($"Czas: {sw.ElapsedMilliseconds}ms\n");   // ~6000ms

// ═══ RÓWNOLEGLE ═══
Console.WriteLine("=== Równolegle (Task.WhenAll) ===");
sw.Restart();
var t1 = PobierzAsync("api/users");      // startuje TERAZ
var t2 = PobierzAsync("api/orders");      // startuje TERAZ
var t3 = PobierzAsync("api/products");    // startuje TERAZ
var wyniki = await Task.WhenAll(t1, t2, t3);  // czekamy na WSZYSTKIE
Console.WriteLine($"Czas: {sw.ElapsedMilliseconds}ms");     // ~2000ms
```

Kluczowa obserwacja: w wersji sekwencyjnej **każdy `await` czeka na zakończenie** przed startem następnego. W wersji równoległej **najpierw uruchamiamy wszystkie taski**, a potem czekamy na wyniki. Trzy operacje po 2s trwają łącznie ~2s, nie ~6s.

#### Demo 2: częsty błąd — blokowanie async kodu

```csharp
// ═══ ŹLE — .Result blokuje wątek ═══
// W aplikacji z UI lub w ASP.NET może to spowodować DEADLOCK
string dane = PobierzDaneAsync().Result;   // NIGDY tak nie róbcie!

// ═══ DOBRZE — await nie blokuje ═══
string dane = await PobierzDaneAsync();
```

Zasada: **jeśli metoda zwraca Task, używaj await. Nigdy .Result, nigdy .Wait().**

#### Demo 3: async w praktyce — pobieranie z HTTP

```csharp
using var http = new HttpClient();

// Pobieramy 3 strony równolegle
var urls = new[]
{
    "https://httpbin.org/delay/1",
    "https://httpbin.org/delay/1",
    "https://httpbin.org/delay/1"
};

var sw = Stopwatch.StartNew();

// Startujemy wszystkie requesty na raz
var taski = urls.Select(url => http.GetStringAsync(url)).ToArray();

// Czekamy na wszystkie
var odpowiedzi = await Task.WhenAll(taski);

Console.WriteLine($"Pobrano {odpowiedzi.Length} stron w {sw.ElapsedMilliseconds}ms");
// ~1000ms, nie ~3000ms
```



### Część 3: TPL (Task Parallel Library) i wielowątkowość

#### Dwa rodzaje „czekania"

Zanim pójdziemy dalej, ważne rozróżnienie:

```
I/O-bound  →  program CZEKA na coś zewnętrznego (sieć, dysk, baza)
               Rozwiązanie: async/await — wątek jest wolny podczas czekania

CPU-bound  →  program LICZY coś ciężkiego (obróbka obrazu, hash, raport)
               Rozwiązanie: TPL — dzielimy pracę na wiele rdzeni procesora
```

#### Analogia: koperty

Macie 1000 kopert do zaklejenia. 

- **Sekwencyjnie** — robicie to sami, jedna po drugiej.
- **Parallel.ForEach** — macie 8 osób i każda dostaje porcję. System sam dzieli pracę.

#### Demo: sekwencyjnie vs równolegle

```csharp
using System.Diagnostics;

void ZmniejszObrazek(int id)
{
    // Symulacja ciężkiej operacji CPU (np. resize zdjęcia)
    Thread.Sleep(100);
}

var obrazki = Enumerable.Range(1, 40).ToList();
var sw = Stopwatch.StartNew();

// ═══ SEKWENCYJNIE ═══
sw.Restart();
foreach (var id in obrazki)
{
    ZmniejszObrazek(id);
}
Console.WriteLine($"Sekwencyjnie:  {sw.ElapsedMilliseconds}ms");  // ~4000ms

// ═══ RÓWNOLEGLE ═══
sw.Restart();
Parallel.ForEach(obrazki, id =>
{
    ZmniejszObrazek(id);
});
Console.WriteLine($"Parallel:      {sw.ElapsedMilliseconds}ms");  // ~500-800ms
```

`Parallel.ForEach` sam decyduje ile wątków użyć (zwykle = liczba rdzeni). Możecie to kontrolować:

```csharp
var opcje = new ParallelOptions { MaxDegreeOfParallelism = 4 };

Parallel.ForEach(obrazki, opcje, id =>
{
    ZmniejszObrazek(id);
    Console.WriteLine($"  Obrazek {id} → wątek {Thread.CurrentThread.ManagedThreadId}");
});
```

#### Thread — najniższy poziom (rzadko potrzebny)

`Thread` to ręczne tworzenie wątku. Daje pełną kontrolę, ale wymaga więcej kodu:

```csharp
void DlugaOperacja(string nazwa)
{
    for (int i = 0; i < 5; i++)
    {
        Console.WriteLine($"[{nazwa}] krok {i} (wątek {Thread.CurrentThread.ManagedThreadId})");
        Thread.Sleep(300);
    }
}

var w1 = new Thread(() => DlugaOperacja("A"));
var w2 = new Thread(() => DlugaOperacja("B"));

w1.Start();
w2.Start();

w1.Join();  // czekaj aż A skończy
w2.Join();  // czekaj aż B skończy

Console.WriteLine("Oba gotowe.");
```

Kolejność wypisywania jest **nieprzewidywalna** — raz A będzie przed B, raz na odwrót. To jest natura wielowątkowości.

W praktyce **rzadko tworzycie `Thread` ręcznie** — `Task.Run` i `Parallel.ForEach` robią to za was lepiej i bezpieczniej.

#### Kiedy co użyć — podsumowanie

```
Czekasz na sieć / plik / bazę danych    →  async/await
Ciężkie obliczenia do rozłożenia na rdzenie  →  Parallel.For / Parallel.ForEach  
Potrzebujesz pełnej kontroli nad wątkiem     →  Thread (rzadko)
Odpalasz jednorazowe zadanie w tle           →  Task.Run
```



### Część 4: Bezpieczeństwo wątków (Thread Safety)

#### Problem: dwóch kasjerów, jedno konto

Dwóch kasjerów sprawdza stan konta klienta. Obaj widzą 1000 zł. Obaj wypłacają 800 zł. Klient ma -600 zł — bo nikt nie zablokował dostępu na czas operacji.

To jest **race condition** — wyścig między wątkami.

#### Demo: problem na żywo

```csharp
int licznik = 0;

var watki = new List<Thread>();

for (int i = 0; i < 10; i++)
{
    var t = new Thread(() =>
    {
        for (int j = 0; j < 10_000; j++)
            licznik++;
    });
    watki.Add(t);
    t.Start();
}

foreach (var t in watki) t.Join();

Console.WriteLine($"Oczekiwano: 100 000");
Console.WriteLine($"Otrzymano:  {licznik}");
// Prawie na pewno MNIEJ niż 100 000!
```

Uruchomcie to kilka razy — za każdym razem **inny wynik**. To jest race condition.

#### Dlaczego `licznik++` nie jest bezpieczne?

`licznik++` wygląda na jedną operację, ale procesor wykonuje ją w trzech krokach:

```
1. Odczytaj wartość licznika    → np. 42
2. Dodaj 1                     → 43
3. Zapisz nową wartość          → 43

Jeśli dwa wątki wykonają krok 1 jednocześnie:
  Wątek A czyta 42
  Wątek B czyta 42
  Wątek A zapisuje 43
  Wątek B zapisuje 43   ← nadpisał wynik A!

Zamiast 44 mamy 43. Straciliśmy jedną inkrementację.
```

#### Rozwiązanie 1: lock — kłódka

```csharp
int licznik = 0;
object zamek = new object();  // obiekt pełniący rolę kłódki

var watki = new List<Thread>();

for (int i = 0; i < 10; i++)
{
    var t = new Thread(() =>
    {
        for (int j = 0; j < 10_000; j++)
        {
            lock (zamek)  // tylko jeden wątek na raz wchodzi do środka
            {
                licznik++;
            }
        }
    });
    watki.Add(t);
    t.Start();
}

foreach (var t in watki) t.Join();

Console.WriteLine($"Wynik: {licznik}");  // ZAWSZE 100 000
```

`lock` działa jak toaleta jednoosobowa — kto wejdzie, zamyka drzwi. Reszta czeka w kolejce.

#### Rozwiązanie 2: Interlocked — atomowa operacja

Dla prostych operacji liczbowych nie trzeba lock-a — `Interlocked` wykonuje operację w jednym niepodzielnym kroku:

```csharp
int licznik = 0;

var watki = new List<Thread>();

for (int i = 0; i < 10; i++)
{
    var t = new Thread(() =>
    {
        for (int j = 0; j < 10_000; j++)
        {
            Interlocked.Increment(ref licznik);  // atomowa inkrementacja
        }
    });
    watki.Add(t);
    t.Start();
}

foreach (var t in watki) t.Join();

Console.WriteLine($"Wynik: {licznik}");  // ZAWSZE 100 000
```

`Interlocked` jest szybszy od `lock`, ale obsługuje tylko proste operacje (increment, decrement, exchange).

#### Rozwiązanie 3: kolekcje thread-safe

Standardowe kolekcje (List, Dictionary) **nie są** bezpieczne dla wielu wątków. .NET oferuje wersje thread-safe w `System.Collections.Concurrent`:

```csharp
// ═══ NIEBEZPIECZNE ═══
var dict = new Dictionary<string, int>();

// ═══ BEZPIECZNE ═══
var safeDict = new ConcurrentDictionary<string, int>();

Parallel.For(0, 10_000, _ =>
{
    safeDict.AddOrUpdate("klucz", 1, (key, stara) => stara + 1);
});

Console.WriteLine($"Wynik: {safeDict["klucz"]}");  // ZAWSZE 10 000
```

#### Przegląd mechanizmów

```
lock                    →  sekcja krytyczna — prosty, uniwersalny
Interlocked             →  atomowe operacje na liczbach
ConcurrentDictionary    →  thread-safe słownik
ConcurrentQueue<T>      →  thread-safe kolejka (producent-konsument)
SemaphoreSlim           →  ogranicza liczbę jednoczesnych wątków
```

#### Zasada: jeśli dwa wątki dotykają tych samych danych — synchronizuj.



### Podsumowanie: jak to wszystko łączy się razem

```
Zdarzenia (EventArgs)
  → Komunikacja między obiektami: "coś się stało, oto szczegóły"
  → Wzorzec publish/subscribe wewnątrz aplikacji

async/await
  → Nie blokuj wątku czekając na I/O (sieć, plik, baza danych)
  → Kluczowe dla UI i serwerów webowych

TPL (Parallel.For / ForEach)
  → Rozdziel ciężkie obliczenia (CPU-bound) na wiele rdzeni

Thread
  → Niskopoziomowa kontrola wątku (rzadko potrzebna bezpośrednio)

Thread Safety (lock, Interlocked, Concurrent*)
  → Chroń współdzielone dane przed race condition
```



#### Laboratorium 2 — Zdarzenia i asynchroniczność w OrderFlow

#### Projekt: OrderFlow — kontynuacja

**Tematy:** Zdarzenia niestandardowe (EventArgs), async/await, Task.WhenAll, przetwarzanie równoległe, thread safety

Pracujecie w tej samej solucji co na Lab 1. Możecie dodać nowe klasy i foldery.



#### Zadanie 1 — Zdarzenia w procesie zamówienia (5 pkt)

Rozbudujcie `OrderProcessor` (lub utwórzcie nową klasę `OrderPipeline`) o system zdarzeń informujących o przetwarzaniu zamówienia.

**Wymagania:**

1. Stwórzcie `OrderStatusChangedEventArgs : EventArgs` z właściwościami: `Order`, `OldStatus`, `NewStatus`, `Timestamp`.
2. Stwórzcie `OrderValidationEventArgs : EventArgs` z właściwościami: `Order`, `IsValid`, `Errors` (lista stringów).
3. W klasie pipeline'u zdefiniujcie zdarzenia:
    - `StatusChanged` (EventHandler&lt;OrderStatusChangedEventArgs&gt;)
    - `ValidationCompleted` (EventHandler&lt;OrderValidationEventArgs&gt;)
4. Metoda `ProcessOrder` powinna zmieniać statusy zamówienia (`New → Validated → Processing → Completed`) i odpalać zdarzenie `StatusChanged` przy każdej zmianie.
5. Zarejestrujcie co najmniej **3 subskrybentów** reagujących na zdarzenia (np. logger konsolowy, symulacja powiadomienia email, aktualizacja statystyk).

**Pokażcie w konsoli** przetworzenie 2-3 zamówień z widocznymi reakcjami subskrybentów.



#### Zadanie 2 — Asynchroniczne pobieranie danych (5 pkt)

Zasymulujcie scenariusz, w którym przetwarzanie zamówienia wymaga danych z zewnętrznych serwisów.

**Wymagania:**

1. Stwórzcie klasę `ExternalServiceSimulator` z metodami async:
    - `CheckInventoryAsync(Product product)` — symulacja sprawdzenia stanu magazynowego (delay 500-1500ms, losowy).
    - `ValidatePaymentAsync(Order order)` — symulacja walidacji płatności (delay 1000-2000ms, losowy).
    - `CalculateShippingAsync(Order order)` — symulacja wyliczenia kosztów wysyłki (delay 300-800ms, losowy).
2. Napiszcie metodę `ProcessOrderAsync(Order order)`, która:
    - Wywołuje **wszystkie trzy** serwisy równolegle (`Task.WhenAll`).
    - Mierzy i wypisuje czas wykonania (użyjcie `Stopwatch`).
3. Napiszcie metodę `ProcessMultipleOrdersAsync(List<Order> orders)`, która:
    - Przetwarza wiele zamówień równolegle.
    - Ogranicza równoległość do max 3 jednoczesnych zamówień (`SemaphoreSlim`).
    - Wypisuje postęp: "Przetworzono 3/6 zamówień".
4. Pokażcie w konsoli **porównanie czasu**: przetwarzanie sekwencyjne vs równoległe.



#### Zadanie 3 — Thread safety w statystykach (5 pkt)

Zbudujcie klasę `OrderStatistics`, która zbiera statystyki w trakcie równoległego przetwarzania zamówień.

**Wymagania:**

1. Klasa przechowuje:
    - `TotalProcessed` (int) — liczba przetworzonych zamówień.
    - `TotalRevenue` (decimal) — łączna kwota.
    - `OrdersPerStatus` — ile zamówień w każdym statusie (słownik).
    - `ProcessingErrors` — lista błędów (jeśli zamówienie nie przeszło walidacji).
2. Pokażcie **problem bez synchronizacji**: odpalcie `Parallel.ForEach` na zamówieniach, aktualizując statystyki bez żadnych zabezpieczeń. Uruchomcie kilka razy — wyniki powinny się różnić.
3. Naprawcie problem używając:
    - `lock` — dla aktualizacji revenue i listy błędów.
    - `Interlocked.Increment` — dla licznika `TotalProcessed`.
    - `ConcurrentDictionary` — dla `OrdersPerStatus`.
4. Pokażcie, że po naprawie wyniki są **zawsze identyczne**.



#### Punktacja

| Zadanie | Punkty |
|||
| Wysłanie na GitHub z commitem z zajęć | 5 pkt |
| 1. Zdarzenia w procesie zamówienia | 5 pkt |
| 2. Asynchroniczne pobieranie danych | 5 pkt |
| 3. Thread safety w statystykach | 5 pkt |
| **Razem** | **20 pkt** |



#### Wskazówki

- Zacznijcie od zadania 1 — zdarzenia to fundament, a przykład z wykładu możecie dostosować do OrderFlow.
- W zadaniu 2 użyjcie `Random` z opóźnieniami, żeby symulacja wyglądała realistycznie. Pamiętajcie: `Random` nie jest thread-safe — twórzcie osobną instancję per wątek albo używajcie `Random.Shared` (.NET 6+).
- W zadaniu 3 celowo pokażcie najpierw **wersję z bugiem** (bez synchronizacji), a potem naprawioną. To najlepsza demonstracja, dlaczego thread safety ma znaczenie.
- Możecie korzystać z modeli i danych z Lab 1.