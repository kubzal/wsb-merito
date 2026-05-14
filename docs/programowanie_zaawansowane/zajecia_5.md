# Zajęcia 5

## Testowanie jednostkowe (xUnit), TDD, integracja z REST API (HttpClient)

Przez cztery zajęcia budowaliście OrderFlow: model, zdarzenia, pliki, bazę. Działa? Pewnie tak — bo każdy z Was odpalił `dotnet run` i sprawdził ręcznie, czy w konsoli pojawia się to, czego się spodziewa. Tyle że za tydzień zmienicie logikę walidacji i… zapomnicie sprawdzić wszystkie ścieżki. Za miesiąc dodacie nowe pole i okaże się, że stara funkcja agregująca zwraca śmieci.

To jest klasyczny problem **regresji** — coś, co działało, przestaje działać po zmianie w innym miejscu kodu. Lekarstwo? **Testy automatyczne**. Piszecie raz, uruchamiacie tysiąc razy — komputer sprawdza za Was, czy nic się nie popsuło.

Drugi temat dnia: aplikacje rzadko żyją w izolacji. Trzeba pogadać z zewnętrznym API — banku, kurierem, dostawcą. W .NET służy do tego `HttpClient`, a do testowania kodu używającego sieci — **mockowanie**. Pokażemy oba podejścia na realnym scenariuszu: pobieranie kursu walut z NBP, żeby zamówienia z OrderFlow można było wyceniać w euro i dolarach.

### Część 1: Po co testy jednostkowe?

#### Problem: ręczne testowanie nie skaluje się

Wyobraźcie sobie scenariusz: piszecie `OrderValidator` (Lab 1), uruchamiacie `dotnet run`, widzicie, że walidacja działa. Super.

Tydzień później dochodzi nowa reguła: zamówienie ze statusem `Cancelled` nie może być modyfikowane. Dodajecie kod, odpalacie program — wygląda OK. Tyle że nie sprawdziliście, czy stare reguły dalej działają. A jedna z nich (np. limit kwoty) przestała przepuszczać poprawne zamówienia, bo niechcący zmieniliście warunek z `>` na `>=`.

Bez testów dowiecie się o tym dopiero wtedy, gdy zadzwoni klient z pretensjami. Z testami — dowiecie się **przy zapisie pliku**.

#### Test jednostkowy = mały, izolowany, szybki

**Jednostka** to najczęściej pojedyncza metoda lub klasa. Test jednostkowy:

- Sprawdza **jedną konkretną rzecz** (np. „walidator odrzuca zamówienie bez pozycji").
- Nie używa bazy danych, sieci, dysku, czasu rzeczywistego — wszystko, co jest „zewnętrzne", trzeba **zamockować** albo zabudować w kontrolowany sposób.
- Powinien wykonywać się w **milisekundach** — żebyście mogli odpalać tysiące testów w kilka sekund i nie tracić rytmu pisania kodu.

To odróżnia testy jednostkowe od **integracyjnych** (które testują kilka modułów razem, czasem z prawdziwą bazą) i **end-to-end** (gdzie odpalacie całą aplikację i klikacie po niej jak użytkownik).

#### Po co to wszystko?

- **Pewność refactoringu.** Macie zielony pasek? Możecie przepisać metodę pod spodem na trzy różne sposoby i sprawdzić, czy dalej działa. Bez testów refaktor to gra w kości.
- **Dokumentacja zachowania.** Nazwy testów typu `Validate_OrderWithoutItems_ReturnsError` mówią więcej niż komentarz nad metodą — bo nie da się skłamać, działający test nie kłamie.
- **Wymuszone myślenie o API.** Pisząc test od razu widzicie, czy klasa jest wygodna w użyciu, czy ma dziwną konstrukcję. Trudny test = pewnie zły kod.
- **Regresje.** Coś przestało działać po zmianie? Test krzyczy natychmiast, a nie po tygodniu.

### Część 2: xUnit — biblioteka testowa

W świecie .NET są trzy główne biblioteki testowe: **MSTest** (od Microsoftu, stara), **NUnit** (klasyk, port z JUnit), **xUnit** (nowsza, czystsza filozofia, autorstwa Brada Wilsona i Jima Newkirka — autora NUnita). My używamy xUnit — to obecnie de facto standard w .NET, używany m.in. przez sam zespół .NET.

#### Tworzenie projektu testowego

Testy żyją w **osobnym projekcie** w solucji. Konwencja: `OrderFlow.Tests`. Tworzymy go w katalogu solucji:

```bash
dotnet new xunit -o OrderFlow.Tests
dotnet sln add OrderFlow.Tests/OrderFlow.Tests.csproj
dotnet add OrderFlow.Tests reference OrderFlow.Console
```

Pierwsza komenda tworzy projekt z szablonu xUnit (od razu z paczkami: `xunit`, `xunit.runner.visualstudio`, `Microsoft.NET.Test.Sdk`). Druga dodaje projekt do solucji. Trzecia daje testowanej dostęp do kodu produkcyjnego — bez tego nie zobaczycie klas z `OrderFlow.Console`.

Uruchamianie testów z linii poleceń:

```bash
dotnet test                    # uruchom wszystkie testy w solucji
dotnet test --filter "Validate" # tylko testy z "Validate" w nazwie
```

W VS Code/Rider/Visual Studio macie dedykowany Test Explorer — kliknięcie odpala pojedynczy test, grupę albo całość.

#### Pierwszy test — `[Fact]` i `Assert`

```csharp
using Xunit;
using OrderFlow.Console.Services;

public class CalculatorTests
{
    [Fact]
    public void Add_TwoPositiveNumbers_ReturnsSum()
    {
        // Arrange — przygotuj dane wejściowe
        var calc = new Calculator();
        
        // Act — wykonaj testowaną operację
        int result = calc.Add(2, 3);
        
        // Assert — sprawdź wynik
        Assert.Equal(5, result);
    }
}
```

Trzy elementy do zapamiętania:

- **`[Fact]`** — atrybut oznaczający metodę jako test. xUnit znajdzie go w runtime i odpali.
- **Wzorzec AAA (Arrange-Act-Assert)** — trzy fazy każdego testu: przygotuj, wykonaj, sprawdź. Często rozdzielacie je pustą linijką albo komentarzem. To nie ozdobnik — czytelność testu jest **kluczowa**, bo test który się zepsuje musicie zrozumieć szybko, najczęściej rok później.
- **`Assert`** — statyczna klasa z metodami sprawdzającymi: `Assert.Equal`, `Assert.True`, `Assert.NotNull`, `Assert.Throws<T>` i kilkanaście innych.

#### Konwencja nazewnictwa testów

Nie ma jednej obowiązującej, ale powszechnie używana to **`MetodaTestowana_Scenariusz_OczekiwanyWynik`**:

```csharp
Validate_OrderWithoutItems_ReturnsError
Add_TwoNegativeNumbers_ReturnsNegativeSum
GetCustomer_NonExistentId_ThrowsNotFoundException
```

Patrząc na nazwę powinniście **bez czytania ciała** wiedzieć, co test sprawdza. Jeśli musicie zaglądać do środka — nazwa jest za słaba.

#### Najczęstsze asercje

```csharp
Assert.Equal(expected, actual);         // równość (porównuje wartości)
Assert.NotEqual(unexpected, actual);
Assert.True(condition);                  // warunek prawdziwy
Assert.False(condition);
Assert.Null(value);
Assert.NotNull(value);
Assert.Empty(collection);                // pusta kolekcja/string
Assert.NotEmpty(collection);
Assert.Contains(item, collection);       // kolekcja zawiera element
Assert.DoesNotContain(item, collection);
Assert.Single(collection);               // dokładnie 1 element
Assert.Equal(3, collection.Count());     // konkretna liczba elementów

// Wyjątki
var ex = Assert.Throws<ArgumentException>(() => calc.Divide(1, 0));
Assert.Equal("Cannot divide by zero", ex.Message);

// Float / decimal — z tolerancją (ostrożnie z floatem!)
Assert.Equal(0.1m + 0.2m, 0.3m, precision: 2);
```

> :warning: **Uwaga na kolejność argumentów w `Equal`!** Konwencja w xUnit (i większości frameworków testowych) to `Assert.Equal(expected, actual)` — najpierw oczekiwane, potem faktyczne. Pomylenie nie zmieni wyniku (test nadal działa), ale komunikat o błędzie będzie odwrócony — „Expected 5, but was 7" zamiast „Expected 7, but was 5". Można się szybko zgubić podczas debugowania.

#### `[Theory]` i `[InlineData]` — jeden test, wiele przypadków

Często chcemy sprawdzić tę samą logikę dla kilku zestawów danych. Zamiast pisać pięć prawie identycznych `[Fact]`, używamy `[Theory]`:

```csharp
[Theory]
[InlineData(1, 1, 2)]
[InlineData(2, 3, 5)]
[InlineData(-1, 1, 0)]
[InlineData(0, 0, 0)]
[InlineData(int.MaxValue, 0, int.MaxValue)]
public void Add_VariousInputs_ReturnsExpectedSum(int a, int b, int expected)
{
    var calc = new Calculator();
    
    int result = calc.Add(a, b);
    
    Assert.Equal(expected, result);
}
```

Test wykona się **pięć razy** — raz dla każdej linijki `InlineData`. Każdy zestaw widać osobno w Test Explorerze, więc gdy jeden padnie, od razu wiecie który.

`[InlineData]` przyjmuje tylko **stałe kompilacji** (literały, primitives). Gdy potrzebujecie złożonych obiektów, użyjcie `[MemberData]` albo `[ClassData]`:

```csharp
public static IEnumerable<object[]> OrdersWithExpectedTotals =>
    new List<object[]>
    {
        new object[] { new Order { /* ... */ }, 100m },
        new object[] { new Order { /* ... */ }, 250m },
    };

[Theory]
[MemberData(nameof(OrdersWithExpectedTotals))]
public void TotalAmount_VariousOrders_CalculatesCorrectly(Order order, decimal expected)
{
    Assert.Equal(expected, order.TotalAmount);
}
```

#### Setup i Teardown — konstruktor i `IDisposable`

W innych frameworkach (NUnit, JUnit) są atrybuty `[SetUp]` i `[TearDown]`. W xUnit filozofia jest inna: klasa testowa jest **tworzona na nowo dla każdego testu**. Więc:

```csharp
public class OrderValidatorTests : IDisposable
{
    private readonly OrderValidator _validator;
    
    public OrderValidatorTests()        // SETUP — wykonywany przed każdym testem
    {
        _validator = new OrderValidator();
    }
    
    public void Dispose()                // TEARDOWN — po każdym teście
    {
        // np. usunięcie plików tymczasowych
    }
    
    [Fact]
    public void Validate_EmptyOrder_ReturnsError() { /* ... */ }
    
    [Fact]
    public void Validate_ValidOrder_ReturnsSuccess() { /* ... */ }
}
```

Każdy test dostaje **świeżą instancję** klasy. Dzięki temu testy są od siebie odseparowane — żaden nie może wpłynąć na drugi przez modyfikację pola w klasie testowej.

Gdy setup jest **drogi** (np. tworzenie bazy danych in-memory) i nie chcecie powtarzać go dla każdego testu, używacie `IClassFixture<T>` — wtedy instancja jest jedna na całą klasę. Ale uwaga: to obciąża Was odpowiedzialnością za to, żeby testy się nie nakładały na siebie.

### Część 3: TDD — Test-Driven Development

#### Pomysł: pisz test, zanim napiszesz kod

TDD odwraca naturalny porządek. Normalnie piszecie funkcję, a potem może test. W TDD:

1. **Red** — piszesz test, **jeszcze nie ma kodu**. Test się nie kompiluje albo pada.
2. **Green** — piszesz **najprostszy możliwy** kod, który sprawia, że test przechodzi. Nawet jeśli to brzydko: zaharkoduj wartość, zwróć stałą — byle przeszło.
3. **Refactor** — masz zielony test, możesz teraz spokojnie poprawić kod. Test pilnuje, żebyś nic nie zepsuł.

Powtarzasz cykl, dodajesz kolejny test, kolejny przypadek, kolejną regułę. Stopniowo kod się rozrasta, a testy przez cały czas chronią Was przed regresją.

#### Przykład: kalkulator rabatów

Załóżmy, że mamy zaimplementować `DiscountCalculator` z regułami:

- Klient VIP dostaje 10% rabatu.
- Zamówienia powyżej 1000 zł dostają dodatkowe 5%.
- Maksymalny rabat to 25%.

**Krok 1 — Red.** Piszemy pierwszy test:

```csharp
[Fact]
public void Calculate_StandardCustomer_NoDiscount()
{
    var calculator = new DiscountCalculator();
    var order = new Order { TotalAmount = 100m };
    var customer = new Customer { IsVip = false };
    
    decimal discount = calculator.Calculate(order, customer);
    
    Assert.Equal(0m, discount);
}
```

Klasa `DiscountCalculator` jeszcze nie istnieje. Kod się nie kompiluje. **Red.**

**Krok 2 — Green.** Najprostsza implementacja:

```csharp
public class DiscountCalculator
{
    public decimal Calculate(Order order, Customer customer) => 0m;
}
```

Test przechodzi. Tak, zwracamy zero. Tak, to brzydkie. Ale działa i spełnia wymaganie z testu. **Green.**

**Krok 3 — kolejny test (Red).**

```csharp
[Fact]
public void Calculate_VipCustomer_Returns10PercentDiscount()
{
    var calculator = new DiscountCalculator();
    var order = new Order { TotalAmount = 100m };
    var customer = new Customer { IsVip = true };
    
    decimal discount = calculator.Calculate(order, customer);
    
    Assert.Equal(10m, discount);
}
```

Test pada — wciąż zwracamy 0. **Red.**

**Krok 4 — Green.**

```csharp
public decimal Calculate(Order order, Customer customer)
{
    if (customer.IsVip)
        return order.TotalAmount * 0.10m;
    return 0m;
}
```

Oba testy przechodzą. **Green.**

**Krok 5 — kolejny przypadek (kwota > 1000 zł):**

```csharp
[Fact]
public void Calculate_OrderOver1000_AddsExtra5Percent()
{
    var calculator = new DiscountCalculator();
    var order = new Order { TotalAmount = 1200m };
    var customer = new Customer { IsVip = true };
    
    decimal discount = calculator.Calculate(order, customer);
    
    // 10% VIP + 5% za kwotę = 15% z 1200 = 180
    Assert.Equal(180m, discount);
}
```

…i dalej w pętli. Każdy test wymusza dopisanie nowej gałęzi logiki. **Refaktoryzujecie** dopiero, gdy widzicie, że kod się duplikuje albo nazwy są niejasne — np. wyciągacie stałe `VipDiscount = 0.10m`, rozbijacie metodę na mniejsze. Testy gwarantują, że refaktor niczego nie psuje.

#### Po co tak komplikować?

Pierwszy raz wygląda to absurdalnie — piszę test do nieistniejącego kodu? Ale efekty są realne:

- **Projektujecie z perspektywy użytkownika klasy.** Najpierw piszecie, jak chcecie ją *wywoływać*, a dopiero potem implementujecie. Wymusza to dobre API.
- **Masz pokrycie testami od razu.** Nie ma sytuacji „dopiszę testy później" (czyli nigdy).
- **Trudno przeoczyć przypadki brzegowe.** Każda gałąź `if` rodzi się z testu — co znaczy, że każdą można sprawdzić.

TDD nie jest religią. Nie do każdego problemu pasuje (eksperymenty, prototypy, ad-hoc skrypty — strata czasu). Ale w logice biznesowej z regułami i przypadkami brzegowymi — wchodzi jak w masło.

### Część 4: Mockowanie zależności

#### Problem: klasa testowana ma zależności

Wyobraźcie sobie `OrderProcessor`, który w środku wywołuje bazę danych:

```csharp
public class OrderProcessor
{
    private readonly OrderFlowContext _db;
    
    public OrderProcessor(OrderFlowContext db)
    {
        _db = db;
    }
    
    public async Task<bool> ProcessAsync(int orderId)
    {
        var order = await _db.Orders.FindAsync(orderId);
        if (order == null) return false;
        order.Status = Status.Processing;
        await _db.SaveChangesAsync();
        return true;
    }
}
```

Jak to przetestować? Możecie utworzyć prawdziwy `OrderFlowContext` z SQLite, ale to:

- Wolniejsze (otwarcie pliku, migracja, query, zamknięcie).
- Trudniejsze do izolacji (musicie usuwać plik między testami albo używać `:memory:`).
- Testuje *integrację* z EF Core, a nie *logikę* `OrderProcessor`-a.

Lepsze rozwiązanie: **wstrzyknąć interfejs** i dać atrapę (mock) zamiast prawdziwego obiektu.

#### Interfejsy = oszczędność na testach

```csharp
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(int id);
    Task UpdateAsync(Order order);
}

public class OrderProcessor
{
    private readonly IOrderRepository _repo;
    
    public OrderProcessor(IOrderRepository repo)  // wstrzyknięta zależność
    {
        _repo = repo;
    }
    
    public async Task<bool> ProcessAsync(int orderId)
    {
        var order = await _repo.GetByIdAsync(orderId);
        if (order == null) return false;
        order.Status = Status.Processing;
        await _repo.UpdateAsync(order);
        return true;
    }
}
```

W teście nie podajemy prawdziwego repozytorium — podajemy atrapę. Można ją napisać ręcznie (in-memory implementation), albo wygenerować bibliotekę mockującą (Moq, NSubstitute, FakeItEasy).

#### Moq — najpopularniejsza biblioteka

Instalacja:

```bash
dotnet add OrderFlow.Tests package Moq
```

Użycie:

```csharp
[Fact]
public async Task ProcessAsync_ExistingOrder_ChangesStatusAndSaves()
{
    // Arrange — przygotuj mock
    var order = new Order { Id = 1, Status = Status.New };
    
    var repoMock = new Mock<IOrderRepository>();
    repoMock.Setup(r => r.GetByIdAsync(1)).ReturnsAsync(order);
    repoMock.Setup(r => r.UpdateAsync(It.IsAny<Order>())).Returns(Task.CompletedTask);
    
    var processor = new OrderProcessor(repoMock.Object);
    
    // Act
    var result = await processor.ProcessAsync(1);
    
    // Assert — wynik metody
    Assert.True(result);
    Assert.Equal(Status.Processing, order.Status);
    
    // Assert — czy mock dostał oczekiwane wywołania
    repoMock.Verify(r => r.GetByIdAsync(1), Times.Once);
    repoMock.Verify(r => r.UpdateAsync(It.Is<Order>(o => o.Status == Status.Processing)), Times.Once);
}
```

Co tu się dzieje?

- `new Mock<IOrderRepository>()` tworzy atrapę interfejsu.
- `Setup(...)` mówi: „gdy ktoś wywoła tę metodę z tym argumentem, zwróć to".
- `It.IsAny<Order>()` to matcher — „każdy argument typu `Order`".
- `It.Is<Order>(o => o.Status == Status.Processing)` to bardziej precyzyjny matcher — „obiekt spełniający warunek".
- `Verify(...)` sprawdza, że metoda została wywołana z oczekiwanymi parametrami i odpowiednią liczbę razy.
- `repoMock.Object` to wygenerowany obiekt, który podajemy do testowanej klasy zamiast prawdziwego repozytorium.

#### Mock vs Stub — czym się różnią?

Słowo „mock" jest często używane zamiennie ze „stub" i „fake", ale są drobne różnice (warto je znać, bo ktoś Was o to zapyta na rekrutacji):

- **Stub** — atrapa zwracająca przygotowane dane, nie sprawdzacie *jak* była wywołana, tylko *co zwróciła*.
- **Mock** — atrapa, na której **weryfikujecie interakcje** (czy metoda była wywołana, ile razy, z jakimi argumentami).
- **Fake** — uproszczona, ale **działająca** implementacja (np. in-memory repository).

W Moq robicie wszystko jednym narzędziem — `Setup` to stub, `Verify` to mock. Praktyka:

- Sprawdzajcie **wynik** testowanej metody, gdy się da. To najczytelniejsza forma asercji.
- Używajcie `Verify` tylko gdy zachowanie się nie objawia w zwracanej wartości (np. „czy wysłał email", „czy zapisał do bazy"). Nadużywanie `Verify` prowadzi do testów sprawdzających **implementację**, a nie **kontrakt** — i sypią się przy każdym refaktorze.

### Część 5: HttpClient — integracja z REST API

#### Wprowadzenie

`HttpClient` to standardowa klasa w .NET do wysyłania zapytań HTTP. Jest częścią `System.Net.Http`. Wygląda prosto:

```csharp
using var client = new HttpClient();
HttpResponseMessage response = await client.GetAsync("https://api.nbp.pl/api/exchangerates/rates/A/USD/?format=json");
response.EnsureSuccessStatusCode();
string json = await response.Content.ReadAsStringAsync();
Console.WriteLine(json);
```

Co tu mamy:

- `GetAsync` — wykonuje request GET, zwraca `Task<HttpResponseMessage>`.
- `EnsureSuccessStatusCode()` — rzuca `HttpRequestException`, jeśli status to coś z zakresu 400-599.
- `ReadAsStringAsync()` — pobiera treść odpowiedzi jako string.

Oprócz GET są oczywiście `PostAsync`, `PutAsync`, `DeleteAsync`, a bardziej elastycznie — `SendAsync` z ręcznie zbudowanym `HttpRequestMessage`.

#### Deserializacja JSON wprost z odpowiedzi

Zamiast czytać string i parsować ręcznie, .NET 6+ ma dedykowane rozszerzenie:

```csharp
using System.Net.Http.Json;

NbpRateResponse? response = await client.GetFromJsonAsync<NbpRateResponse>(
    "https://api.nbp.pl/api/exchangerates/rates/A/USD/?format=json");
```

`GetFromJsonAsync<T>` w jednym kroku robi GET, sprawdza status, deserializuje JSON do typu `T`. Wewnętrznie używa `System.Text.Json`, którego znacie już z Lab 3.

Klasa docelowa musi mieć kształt pasujący do odpowiedzi (z konwencją `camelCase` → `PascalCase` lub atrybutami `[JsonPropertyName]`):

```csharp
public class NbpRateResponse
{
    [JsonPropertyName("table")]
    public string Table { get; set; } = "";
    
    [JsonPropertyName("currency")]
    public string Currency { get; set; } = "";
    
    [JsonPropertyName("code")]
    public string Code { get; set; } = "";
    
    [JsonPropertyName("rates")]
    public List<NbpRate> Rates { get; set; } = new();
}

public class NbpRate
{
    [JsonPropertyName("no")]
    public string No { get; set; } = "";
    
    [JsonPropertyName("effectiveDate")]
    public DateOnly EffectiveDate { get; set; }
    
    [JsonPropertyName("mid")]
    public decimal Mid { get; set; }
}
```

#### `HttpClient` i czas życia — pułapka

Naturalnym odruchem jest pisanie `using var client = new HttpClient()` w każdej metodzie. **To zła praktyka.** `HttpClient` pod spodem trzyma pulę połączeń TCP, a tworzenie i niszczenie go w pętli powoduje wyczerpanie portów (problem znany jako *socket exhaustion*).

Dwa poprawne podejścia:

**Wariant 1 — jedna instancja na cały program (singleton):**

```csharp
public class CurrencyService
{
    private static readonly HttpClient _client = new();
    
    public async Task<decimal> GetRateAsync(string code) { /* ... */ }
}
```

Działa, ale ma wadę: ta instancja nie wie nic o zmianach DNS — jeśli serwer zmieni IP, klient może utknąć ze starą wartością do końca życia procesu.

**Wariant 2 — `IHttpClientFactory` (zalecane na produkcji):**

```csharp
// Rejestracja (w aplikacjach z DI — ASP.NET, .NET Generic Host)
services.AddHttpClient<CurrencyService>(client =>
{
    client.BaseAddress = new Uri("https://api.nbp.pl/");
});

// Użycie
public class CurrencyService
{
    private readonly HttpClient _client;
    
    public CurrencyService(HttpClient client)
    {
        _client = client;  // factory wstrzykuje skonfigurowanego clienta
    }
}
```

Fabryka zarządza czasem życia klientów, obsługuje DNS, pozwala konfigurować politykę retry. W aplikacjach konsolowych bez DI singleton jest OK — w „prawdziwych" aplikacjach używajcie fabryki.

#### Obsługa błędów

`HttpClient` rzuca `HttpRequestException`, gdy:

- Serwer zwróci kod 4xx/5xx (przy użyciu `EnsureSuccessStatusCode` lub `GetFromJsonAsync`).
- Nie da się nawiązać połączenia (brak sieci, DNS, timeout).

Dodatkowo `TaskCanceledException` (a właściwie `OperationCanceledException`) leci, gdy:

- Minął timeout (domyślnie 100 sekund — najczęściej warto skrócić).
- Wywołanie zostało anulowane przez `CancellationToken`.

```csharp
try
{
    var response = await client.GetFromJsonAsync<NbpRateResponse>(url);
    return response;
}
catch (HttpRequestException ex) when (ex.StatusCode == HttpStatusCode.NotFound)
{
    // Waluta nie istnieje — zwracamy null/wyjątek domenowy
    return null;
}
catch (HttpRequestException ex)
{
    // Inny błąd HTTP — logujemy i ewentualnie rzucamy własny wyjątek
    throw new CurrencyServiceException($"Błąd pobierania kursu: {ex.Message}", ex);
}
catch (TaskCanceledException)
{
    throw new CurrencyServiceException("Timeout przy pobieraniu kursu");
}
```

Konwersja błędów technicznych (HTTP) na błędy domenowe (`CurrencyServiceException`) to dobra praktyka — kod wyżej nie musi wiedzieć, że pod spodem leci HTTP.

### Część 6: Testowanie kodu używającego HttpClient

#### Problem: HttpClient nie ma interfejsu

Patrząc na `CurrencyService`, naturalnym pomysłem byłoby wstrzyknąć `IHttpClient` i go zamockować. Niestety — `HttpClient` jest **klasą**, nie interfejsem. Co więcej, jest **zapieczętowany** w wielu metodach (nie da się ich łatwo zaślepić).

W .NET stosuje się trick: nie mockujemy `HttpClient`, tylko `HttpMessageHandler` — klasę pod spodem, która wykonuje faktyczny request. `HttpClient` przyjmuje handler w konstruktorze i deleguje do niego pracę.

#### Własny handler testowy

```csharp
public class TestHttpMessageHandler : HttpMessageHandler
{
    private readonly Func<HttpRequestMessage, HttpResponseMessage> _responder;
    
    public TestHttpMessageHandler(Func<HttpRequestMessage, HttpResponseMessage> responder)
    {
        _responder = responder;
    }
    
    protected override Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request, CancellationToken cancellationToken)
    {
        return Task.FromResult(_responder(request));
    }
}
```

Tworzymy handler, który zamiast wysyłać request przez sieć, wywołuje przekazaną funkcję i zwraca to, co ona ustali. W teście robimy tak:

```csharp
[Fact]
public async Task GetRateAsync_ValidCurrency_ReturnsRate()
{
    // Arrange — przygotowujemy odpowiedź, jaką "udaje" serwer
    var jsonResponse = """
        {
          "table": "A",
          "currency": "dolar amerykański",
          "code": "USD",
          "rates": [
            { "no": "086/A/NBP/2026", "effectiveDate": "2026-05-06", "mid": 3.9512 }
          ]
        }
        """;
    
    var handler = new TestHttpMessageHandler(request =>
        new HttpResponseMessage(HttpStatusCode.OK)
        {
            Content = new StringContent(jsonResponse, Encoding.UTF8, "application/json")
        });
    
    var httpClient = new HttpClient(handler);
    var service = new CurrencyService(httpClient);
    
    // Act
    decimal rate = await service.GetRateAsync("USD");
    
    // Assert
    Assert.Equal(3.9512m, rate);
}
```

Test jest:

- **Szybki** — żadna sieć nie jest używana.
- **Deterministyczny** — odpowiedź jest pod naszą kontrolą.
- **Pełny** — możemy symulować dowolny scenariusz: 200 OK, 404, 500, timeout.

#### Test scenariusza błędu

```csharp
[Fact]
public async Task GetRateAsync_NotFound_ReturnsNull()
{
    var handler = new TestHttpMessageHandler(_ =>
        new HttpResponseMessage(HttpStatusCode.NotFound)
        {
            Content = new StringContent("404 NotFound - Not Found - Brak danych")
        });
    
    var service = new CurrencyService(new HttpClient(handler));
    
    decimal? rate = await service.GetRateAsync("XYZ");
    
    Assert.Null(rate);
}
```

Możemy też weryfikować, że request poszedł pod właściwy URL:

```csharp
[Fact]
public async Task GetRateAsync_BuildsCorrectUrl()
{
    HttpRequestMessage? capturedRequest = null;
    
    var handler = new TestHttpMessageHandler(request =>
    {
        capturedRequest = request;
        return new HttpResponseMessage(HttpStatusCode.OK)
        {
            Content = new StringContent("""{"rates":[{"mid":4.0}]}""")
        };
    });
    
    var service = new CurrencyService(new HttpClient(handler));
    
    await service.GetRateAsync("EUR");
    
    Assert.NotNull(capturedRequest);
    Assert.Contains("/EUR/", capturedRequest!.RequestUri!.AbsolutePath);
    Assert.Equal(HttpMethod.Get, capturedRequest.Method);
}
```

#### Moq do mockowania `HttpMessageHandler`

Jeśli wolicie Moq zamiast własnego handlera (oba podejścia są poprawne):

```csharp
var handlerMock = new Mock<HttpMessageHandler>();
handlerMock
    .Protected()
    .Setup<Task<HttpResponseMessage>>(
        "SendAsync",
        ItExpr.IsAny<HttpRequestMessage>(),
        ItExpr.IsAny<CancellationToken>())
    .ReturnsAsync(new HttpResponseMessage(HttpStatusCode.OK)
    {
        Content = new StringContent("""{"rates":[{"mid":4.0}]}""")
    });

var httpClient = new HttpClient(handlerMock.Object);
```

`SendAsync` jest `protected`, więc używamy `Protected()` z Moq, żeby się do niej dostać. Działa, ale wielu programistów uważa, że własna klasa `TestHttpMessageHandler` jest czytelniejsza — bo wystarczy raz ją napisać i potem używać wszędzie.

### Część 7: Testowanie kodu używającego EF Core

Wasz Lab 4 używa SQLite. Do testów warto użyć providera **SQLite in-memory** albo **EF Core In-Memory** — bazy, która istnieje tylko w pamięci, jest tworzona od nowa dla każdego testu i nie wymaga pliku.

```bash
dotnet add OrderFlow.Tests package Microsoft.EntityFrameworkCore.InMemory
```

```csharp
public class OrderRepositoryTests
{
    private OrderFlowContext CreateContext()
    {
        var options = new DbContextOptionsBuilder<OrderFlowContext>()
            .UseInMemoryDatabase(databaseName: Guid.NewGuid().ToString())
            .Options;
        return new OrderFlowContext(options);
    }
    
    [Fact]
    public async Task AddOrder_PersistsToDatabase()
    {
        // Arrange
        using var db = CreateContext();
        var order = new Order { OrderDate = DateTime.Now, Status = Status.New };
        
        // Act
        db.Orders.Add(order);
        await db.SaveChangesAsync();
        
        // Assert
        var fromDb = await db.Orders.FindAsync(order.Id);
        Assert.NotNull(fromDb);
        Assert.Equal(Status.New, fromDb!.Status);
    }
}
```

Każdy test dostaje **świeżą bazę** dzięki `Guid.NewGuid()` w nazwie — testy się nie nakładają.

> :warning: Provider In-Memory **nie obsługuje wszystkiego, co prawdziwa baza** — np. transakcji, niektórych ograniczeń, raw SQL. Dla testów typowej logiki CRUD wystarczy. Dla rzeczy specyficznych dla SQL użyjcie `:memory:` w SQLite (relacyjna baza w RAM, zachowuje się jak prawdziwa).

Aby `DbContext` przyjmował opcje z zewnątrz, musi mieć drugi konstruktor:

```csharp
public class OrderFlowContext : DbContext
{
    public OrderFlowContext() { }                                       // dla migracji
    public OrderFlowContext(DbContextOptions<OrderFlowContext> options) // dla testów
        : base(options) { }
    
    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        if (!options.IsConfigured)
            options.UseSqlite("Data Source=orderflow.db");
    }
}
```

### Podsumowanie

```
xUnit (Fact, Theory, Assert)
  → Standard testów jednostkowych w .NET
  → AAA pattern: Arrange-Act-Assert
  → [Theory] + [InlineData] dla wielu przypadków
  → Konstruktor = setup, IDisposable = teardown

TDD (Red-Green-Refactor)
  → Najpierw test, potem najprostsza implementacja, potem refactor
  → Wymusza dobre API i pełne pokrycie
  → Stosować z głową — nie do wszystkiego

Mockowanie (Moq, NSubstitute, FakeItEasy)
  → Wstrzykujcie interfejsy, nie klasy konkretne
  → Stub = przygotowane dane, Mock = weryfikacja interakcji
  → Verify używać oszczędnie — sprawdza implementację, nie kontrakt

HttpClient
  → Standardowy klient HTTP w .NET
  → GetFromJsonAsync<T> + System.Text.Json
  → Nie twórzcie w pętli — singleton albo IHttpClientFactory
  → Obsługa: HttpRequestException, TaskCanceledException

Testowanie HttpClient
  → Mockujemy HttpMessageHandler, nie HttpClient
  → Własny TestHttpMessageHandler — proste i czytelne
  → Można też Moq z Protected().Setup

Testowanie EF Core
  → InMemory provider dla prostych testów
  → SQLite :memory: dla testów wymagających prawdziwego SQL
  → Świeża baza per test (Guid w nazwie)
```

---

## Laboratorium 5 — Testy i integracja z REST API w OrderFlow

**Projekt:** OrderFlow — kontynuacja (ostatni lab!)

**Tematy:** xUnit, TDD, mockowanie (Moq), HttpClient, REST API (NBP), testowanie kodu sieciowego

Pracujecie w tej samej solucji co na Lab 1–4. Główna zmiana strukturalna: dochodzi **drugi projekt** w solucji — `OrderFlow.Tests`. Logika produkcyjna pozostaje w `OrderFlow.Console`, dochodzi nowa klasa `CurrencyService`.

```
OrderFlow/
├── OrderFlow.sln
├── OrderFlow.Console/
│   ├── Models/
│   ├── Services/
│   │   ├── CurrencyService.cs    ← nowość
│   │   └── DiscountCalculator.cs ← nowość (TDD)
│   ├── Persistence/
│   └── Program.cs
└── OrderFlow.Tests/               ← nowy projekt
    ├── OrderValidatorTests.cs
    ├── DiscountCalculatorTests.cs
    ├── CurrencyServiceTests.cs
    └── OrderRepositoryTests.cs
```

Przed rozpoczęciem dodajcie projekt testowy:

```bash
cd OrderFlow
dotnet new xunit -o OrderFlow.Tests
dotnet sln add OrderFlow.Tests/OrderFlow.Tests.csproj
dotnet add OrderFlow.Tests reference OrderFlow.Console
dotnet add OrderFlow.Tests package Moq
dotnet add OrderFlow.Tests package Microsoft.EntityFrameworkCore.InMemory
```

---

### Zadanie 1 — Testy istniejącego kodu z OrderFlow (5 pkt)

Pokryjcie testami wybrane fragmenty kodu z poprzednich labów. Celem jest oswojenie się z xUnit, asercjami i wzorcem AAA na **kodzie, który już macie**.

**Wymagania:**

1. Napiszcie minimum **6 testów** dla `OrderValidator` z Lab 1:
    - Testy dla **named methods** (zamówienie musi mieć pozycje, kwota nie przekracza limitu, ilości > 0) — każda reguła osobno.
    - Testy dla **lambd Func&lt;Order, bool&gt;** (data nie z przyszłości, status nie jest `Cancelled`).
    - Test dla `ValidateAll` łączącego oba mechanizmy — z zamówieniem łamiącym kilka reguł naraz.
2. Co najmniej **jeden test** napiszcie jako `[Theory]` z `[InlineData]` — np. różne kombinacje statusów i oczekiwanego wyniku walidacji.
3. Napiszcie **co najmniej 2 testy** dla `OrderProcessor` z Lab 1 (metody korzystające z `Predicate`, `Func`, agregacja).
4. Wszystkie testy stosują wzorzec **AAA** z rozdzieleniem (komentarze lub puste linie) i czytelne nazwy (`Metoda_Scenariusz_Wynik`).
5. Pokażcie, że **wszystkie testy przechodzą** (`dotnet test` zwraca zielone wyniki w konsoli).

**Wskazówka:** Jeśli `OrderValidator` ma niewygodne API do testowania (np. wymaga publicznych metod, które są dziś prywatne), to **zrefaktoryzujcie kod produkcyjny** — to normalne, że pisanie testów wymusza zmiany w architekturze. Tylko zróbcie to po napisaniu pierwszego testu, nie przed.

---

### Zadanie 2 — DiscountCalculator metodą TDD (5 pkt)

Zbudujcie **od zera** serwis `DiscountCalculator` obliczający rabaty dla zamówień, **stosując cykl Red-Green-Refactor**. Klasa nie ma jeszcze istnieć w `OrderFlow.Console` — piszecie ją wyłącznie po napisaniu odpowiednich testów.

**Reguły rabatu (do zaimplementowania krok po kroku):**

- Klient standardowy, mała kwota → 0% rabatu.
- Klient VIP → 10% rabatu.
- Zamówienie powyżej 1000 zł → dodatkowe 5%.
- Klient VIP z zamówieniem powyżej 5000 zł → dodatkowe 5% (czyli 20% łącznie).
- Maksymalny rabat to **25%** — niezależnie od liczby spełnionych reguł.
- Rabat zawsze zwracany jako **kwota w PLN**, nie procent.

**Wymagania:**

1. Pracujcie w cyklu **Red-Green-Refactor**:
    - Najpierw napiszcie test dla pierwszej reguły, upewnijcie się, że się nie kompiluje (Red).
    - Napiszcie **minimalną implementację** (Green).
    - Dopiero potem przejdźcie do kolejnej reguły.
2. Po każdej regule **zacommitujcie** zmiany — chcemy zobaczyć w historii Gita, że stosowaliście TDD. Sugerowane komunikaty: `Red: VIP discount test`, `Green: VIP discount implementation`, `Refactor: extract VipDiscountRate constant`.
3. Po wszystkich regułach zróbcie krok **Refactor**: wyciągnijcie stałe (`VipDiscountRate`, `HighValueThreshold`), rozbijcie metodę na mniejsze prywatne metody pomocnicze, zadbajcie o czytelność. Testy mają cały czas przechodzić.
4. Pokażcie **łańcuch commitów w README** lub na osobnym ekranie zrzutu — chcemy zobaczyć Red → Green → Refactor w historii.
5. Co najmniej **8 testów** pokrywających różne kombinacje reguł (włącznie z testem dla limitu 25%).

**Wskazówka:** Pokusa jest taka, żeby napisać od razu całą logikę z głowy. **Nie róbcie tego** — celem zadania jest doświadczenie iteracyjnego procesu. Ma być wolniej i drobnymi krokami.

---

### Zadanie 3 — CurrencyService z NBP API + testy z mockiem HttpClient (5 pkt)

Stwórzcie `CurrencyService` — serwis pobierający aktualne kursy walut z **publicznego API Narodowego Banku Polskiego** (`https://api.nbp.pl`). Następnie pokryjcie go testami, mockując `HttpMessageHandler`.

**O NBP API:** Endpoint dla bieżącego kursu waluty:
```
https://api.nbp.pl/api/exchangerates/rates/A/USD/?format=json
```
Zwraca JSON z polem `rates[0].mid` (kurs średni). Działa bez klucza, bez logowania, bez limitu (w rozsądnym zakresie). Dla nieistniejącej waluty zwraca **404 Not Found**.

**Wymagania:**

1. Stwórzcie interfejs `ICurrencyService`:
    ```csharp
    public interface ICurrencyService
    {
        Task<decimal?> GetRateAsync(string currencyCode);
        Task<decimal> ConvertAsync(decimal amount, string fromCurrency, string toCurrency);
    }
    ```
2. Implementacja `CurrencyService`:
    - Konstruktor przyjmuje `HttpClient` (testowalność!).
    - `GetRateAsync` zwraca kurs średni do PLN lub `null` jeśli waluta nie istnieje (HTTP 404).
    - `ConvertAsync` używa `GetRateAsync` do przeliczenia kwoty z jednej waluty na drugą (np. USD → EUR przez PLN jako wspólny mianownik).
    - Specjalny przypadek: `GetRateAsync("PLN")` zwraca `1.0m` bez wywołania API.
    - Dla błędów innych niż 404 rzucajcie własny wyjątek `CurrencyServiceException`.
3. Napiszcie **klasę `TestHttpMessageHandler`** w projekcie testowym (do reużycia w testach).
4. Napiszcie minimum **6 testów** dla `CurrencyService`, mockując `HttpMessageHandler`:
    - Happy path — poprawna waluta, kurs zwrócony.
    - Specjalny przypadek `PLN` — zwraca 1, **nie wywołuje** API (zweryfikujcie to przez sprawdzenie, że handler nie był wywołany).
    - 404 dla nieistniejącej waluty — zwraca `null`.
    - 500 (Internal Server Error) — rzuca `CurrencyServiceException`.
    - Test dla `ConvertAsync` z dwiema różnymi walutami niezerowymi.
    - Test weryfikujący, że request idzie pod właściwy URL (`/api/exchangerates/rates/A/USD/...`).
5. **Integracja z OrderFlow:** dodajcie metodę `ConvertOrderTotalAsync(Order order, string targetCurrency)` w nowej klasie `OrderCurrencyConverter`, która korzysta z `ICurrencyService` (wstrzykniętego przez konstruktor). Napiszcie minimum **2 testy** dla tej klasy, mockując `ICurrencyService` przez **Moq** (nie przez HttpMessageHandler!). To pokazuje różnicę: HttpMessageHandler mockuje się dla klas używających HTTP, Moq dla zwykłych zależności.
6. W `Program.cs` pokażcie **przykładowe użycie**: wczytajcie kilka zamówień z bazy (Lab 4) i wypiszcie ich `TotalAmount` w USD i EUR.

**Bonus (+1 pkt, max 5 pkt za zadanie):** Dodajcie **prosty cache** w `CurrencyService` — kurs pobrany raz w danej sesji nie jest pobierany ponownie. Napiszcie test weryfikujący, że dla dwóch wywołań `GetRateAsync("USD")` w tym samym serwisie API jest pytane **tylko raz**.

---

### Punktacja

| Zadanie | Punkty |
|---|---|
| Wysłanie na GitHub z commitem z zajęć | **5 pkt** |
| 1. Testy istniejącego kodu (OrderValidator, OrderProcessor) | 5 pkt |
| 2. DiscountCalculator metodą TDD | 5 pkt |
| 3. CurrencyService + REST API + mockowanie | 5 pkt |
| **Razem** | **20 pkt** |

> **Uwaga o GitHub:** Jak na poprzednich zajęciach — aby otrzymać 5 pkt za wysłanie, musicie mieć **co najmniej 1 commit z czasów zajęć**. W zadaniu 2 łańcuch commitów typu Red-Green-Refactor jest wymagany — to dowód stosowania TDD, a nie tylko końcowego wyniku.

> **Uwaga o `dotnet test`:** Komisja zaliczeniowa (czyli ja) odpali Wam `dotnet test` na sklonowanym repo. Jeśli testy nie przechodzą — punkty za zadanie 1 lub 2 lecą. Sprawdźcie u siebie przed pull requestem.

### Wskazówki

- Zacznijcie od zadania 1 — jest najprostsze i pomoże Wam się oswoić z xUnit. Bez tego ciężko będzie zabrać się za TDD.
- W zadaniu 2 **opierajcie się pokusie napisania całej logiki naraz**. Klasyczny błąd: piszemy DiscountCalculator z głową pełną reguł, a potem dopisujemy testy. To nie jest TDD — to jest TAD (test-after-development). Punkty będą za widoczny w historii Gita proces, nie tylko za końcowy stan.
- W zadaniu 3 sprawdźcie ręcznie raz endpoint NBP w przeglądarce albo curl-em, żeby zobaczyć kształt JSON:
    ```bash
    curl "https://api.nbp.pl/api/exchangerates/rates/A/USD/?format=json"
    ```
- Pamiętajcie, że NBP API zwraca daty jako `"effectiveDate": "2026-05-13"` — `System.Text.Json` poradzi sobie z `DateOnly` od .NET 7. Dla `DateTime` też zadziała.
- W testach `CurrencyService` **nie wołajcie prawdziwego NBP API** — to nie test jednostkowy, tylko integracyjny. Cała pointa zadania 3 to mockowanie warstwy sieciowej. Jeśli `dotnet test` wymaga internetu — straciliście sens ćwiczenia.
- Jeśli macie ochotę napisać też **testy integracyjne** (z prawdziwym API), zróbcie je w osobnym pliku z atrybutem `[Trait("Category", "Integration")]` i pomijajcie standardowo w CI. Ale to bonus, nie wymaganie.
- `Moq` + `Protected()` jest trochę nieintuicyjny — własny `TestHttpMessageHandler` to zwykle czytelniejsza droga. Wybierzcie podejście, które Wam wygodniej, i bądźcie konsekwentni.
- Jak zwykle: **commitujcie często**. W zadaniu 2 to wręcz wymagane, a w 1 i 3 — po prostu dobry nawyk. Commit po każdym przechodzącym teście to dobra granica.

---

## Na zakończenie

To były ostatnie zajęcia z Programowania zaawansowanego. Przez pięć labów zbudowaliście **OrderFlow** — od pustego `Program.cs` do aplikacji z modelem domenowym, zdarzeniami, asynchronicznością, persystencją w plikach i bazie danych, a teraz testami i integracją z zewnętrznym API.

To, co napisaliście, jest **w 80% gotowym szkieletem prawdziwej aplikacji backendowej**. Brakuje warstwy webowej (ASP.NET Core) i deployu, ale to już kolejny rozdział — i niezły temat na pracę dyplomową, jeśli ktoś z Was szuka pomysłu.

Dzięki za zajęcia. Powodzenia na egzaminach i w dalszej drodze z C#.