# Zajęcia 4

## Entity Framework Core: ORM, relacje, migracje, transakcje, seeding

Na poprzednich zajęciach zapisywaliście zamówienia do plików JSON i XML. Działało, ale wyobraźcie sobie: macie 100 000 zamówień i chcecie znaleźć te złożone przez klientów VIP w marcu, posortowane po kwocie malejąco. W JSON-ie musielibyście wczytać cały plik do pamięci, zdeserializować, przefiltrować LINQ-iem — a to trwa i zjada RAM. Baza danych zrobi to samo w milisekundach, bo ma indeksy, planer zapytań i dziesiątki lat optymalizacji pod spodem.

Dziś wprowadzamy **Entity Framework Core (EF Core)** — ORM, który pozwala operować na bazie danych **tymi samymi klasami C#**, które już macie. Zamiennik `OrderRepository` z Lab 3, ale zamiast `File.ReadAllTextAsync` piszecie zapytania LINQ — a EF Core tłumaczy je na SQL.

### Część 1: ORM — co to jest i dlaczego

#### Problem: dwa światy

Programujecie w C# — macie klasy, właściwości, kolekcje, dziedziczenie. Baza danych ma tabele, kolumny, klucze obce, SQL. Między tymi dwoma światami jest **przepaść**. Tradycyjnie budowalibyście most ręcznie:

```csharp
// Ręczny odczyt z bazy — ADO.NET
using var conn = new SqliteConnection("Data Source=orderflow.db");
conn.Open();
using var cmd = conn.CreateCommand();
cmd.CommandText = "SELECT Id, Customer, Total FROM Orders WHERE Total > @min";
cmd.Parameters.AddWithValue("@min", 100);

using var reader = cmd.ExecuteReader();
while (reader.Read())
{
    var order = new Order
    {
        Id = reader.GetInt32(0),
        Customer = reader.GetString(1),
        Total = reader.GetDecimal(2)
    };
    // ...użycie
}
```

To działa, ale jest **żmudne, powtarzalne i kruche** — zmieniacie nazwę kolumny w bazie i musicie ręcznie zaktualizować każdy string `"Customer"` w kodzie. Kompilator nic nie powie — dowiecie się o błędzie dopiero w runtime.

#### ORM — automatyczny tłumacz

**ORM (Object-Relational Mapping)** to warstwa, która mapuje klasy na tabele, właściwości na kolumny i kolekcje na relacje — automatycznie. Zamiast pisać SQL, piszecie LINQ, a ORM tłumaczy go na SQL za Was:

```csharp
// To samo zapytanie, ale przez EF Core
var drogie = await db.Orders
    .Where(o => o.Total > 100)
    .ToListAsync();
```

Pod spodem EF Core generuje:

```sql
SELECT "o"."Id", "o"."Customer", "o"."Total"
FROM "Orders" AS "o"
WHERE "o"."Total" > 100
```

Kluczowe — pisaliście **ten sam LINQ** co na zajęciach 1, kiedy filtrowaliście listy in-memory. Tyle że teraz zapytanie nie wykonuje się w pamięci, tylko jest tłumaczone na SQL i wysyłane do bazy danych. To jest właśnie „Language Integrated" w LINQ — ta sama składnia, różne źródła danych.

#### EF Core — historia w pigułce

Entity Framework istnieje od 2008 roku (jako część .NET Framework), ale tamta wersja jest dziś przestarzała. **Entity Framework Core** to przepisanie od zera, lekkie, modułowe, wieloplatformowe. W .NET 10 najnowsza wersja to **EF Core 10**. My będziemy pracować z bazą **SQLite** — jest plikowa (zero konfiguracji, zero serwera), idealna do nauki i małych projektów. EF Core wspiera też PostgreSQL, SQL Server, MySQL i inne — zmiana bazy to kwestia jednego NuGeta i connection stringa.

### Część 2: DbContext i konfiguracja

#### Instalacja

Dodajecie dwa pakiety NuGet do projektu:

```bash
dotnet add package Microsoft.EntityFrameworkCore.Sqlite
dotnet add package Microsoft.EntityFrameworkCore.Design
```

Pierwszy dostarcza providera SQLite, drugi — narzędzia do migracji. Instalujecie też CLI (jednorazowo, globalnie):

```bash
dotnet tool install --global dotnet-ef
```

Sprawdzenie instalacji:

```bash
dotnet ef --version
```

#### DbContext — brama do bazy danych

`DbContext` to centralna klasa EF Core. Reprezentuje sesję z bazą danych — przez nią odpytujecie, dodajecie, modyfikujecie i usuwacie dane. Definicja jest prosta:

```csharp
using Microsoft.EntityFrameworkCore;

public class OrderFlowContext : DbContext
{
    // Każdy DbSet<T> = jedna tabela w bazie
    public DbSet<Product> Products => Set<Product>();
    public DbSet<Customer> Customers => Set<Customer>();
    public DbSet<Order> Orders => Set<Order>();
    public DbSet<OrderItem> OrderItems => Set<OrderItem>();

    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        options.UseSqlite("Data Source=orderflow.db");
    }
}
```

`DbSet<T>` to kolekcja encji — odpowiednik tabeli. Możecie na nim wywoływać LINQ (`Where`, `Select`, `GroupBy`...) — EF Core przetłumaczy to na SQL. `"Data Source=orderflow.db"` oznacza, że baza danych to plik `orderflow.db` w katalogu roboczym.

Oto wzorzec użycia — znów widzicie `using`, bo `DbContext` implementuje `IDisposable`:

```csharp
using (var db = new OrderFlowContext())
{
    var orders = await db.Orders
        .Where(o => o.Status == Status.Completed)
        .ToListAsync();
        
    foreach (var o in orders)
        Console.WriteLine($"#{o.Id} — {o.TotalAmount:C}");
}
```

#### Modele — konwencje EF Core

EF Core odkrywa strukturę bazy na podstawie **konwencji**. Wasze istniejące klasy z Lab 1 prawdopodobnie wymagają minimalnych zmian:

```csharp
public class Product
{
    public int Id { get; set; }              // konwencja: "Id" = klucz główny
    public string Name { get; set; } = "";
    public decimal Price { get; set; }
    public string Category { get; set; } = "";

    // Nawigacja — kolekcja „odwrotna"
    public List<OrderItem> OrderItems { get; set; } = new();
}

public class Customer
{
    public int Id { get; set; }
    public string FullName { get; set; } = "";
    public string City { get; set; } = "";
    public bool IsVip { get; set; }

    // Nawigacja — jeden klient ma wiele zamówień
    public List<Order> Orders { get; set; } = new();
}

public class Order
{
    public int Id { get; set; }
    public DateTime OrderDate { get; set; }
    public Status Status { get; set; }

    // Klucz obcy
    public int CustomerId { get; set; }
    // Nawigacja — referencja do powiązanego obiektu
    public Customer Customer { get; set; } = null!;

    public List<OrderItem> Items { get; set; } = new();

    // Właściwość obliczana — EF Core jej NIE mapuje na kolumnę
    // (bo nie ma settera publicznego i nie da się jej zapisać w bazie)
    public decimal TotalAmount => Items.Sum(i => i.TotalPrice);
}

public class OrderItem
{
    public int Id { get; set; }
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
    public decimal TotalPrice => Quantity * UnitPrice;

    public int OrderId { get; set; }
    public Order Order { get; set; } = null!;

    public int ProductId { get; set; }
    public Product Product { get; set; } = null!;
}
```

**Konwencje klucza głównego:** Właściwość o nazwie `Id` lub `<NazwaKlasy>Id` (np. `ProductId`) jest automatycznie rozpoznawana jako klucz główny (primary key). Dla `int` EF Core generuje wartości automatycznie (autoincrement).

**Nawigacje:** `public Customer Customer { get; set; }` to **właściwość nawigacyjna** — EF Core rozumie, że `CustomerId` to klucz obcy wskazujący na tabelę `Customers`. Dzięki temu możecie pisać `order.Customer.FullName` bez ręcznego joina. Z kolei `public List<Order> Orders` w klasie `Customer` to nawigacja „odwrotna" — lista zamówień danego klienta.

> :warning: **`= null!`** przy nawigacjach referencyjnych — to konwencja mówi kompilatorowi „wiem, że teraz to jest null, ale EF Core wypełni to zanim użyję". Bez tego dostaniecie warning o nullable reference types.

#### Fluent API — gdy konwencje nie wystarczają

Czasem chcecie więcej kontroli niż daje konwencja. Na przykład: wymusić unikalność, ustawić długość kolumny, wykluczyć właściwość obliczaną z mapowania. Do tego służy **Fluent API** — nadpisujecie metodę `OnModelCreating` w `DbContext`:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Customer>(entity =>
    {
        entity.Property(c => c.FullName)
            .HasMaxLength(200)
            .IsRequired();

        entity.HasIndex(c => c.FullName);
    });

    modelBuilder.Entity<Order>(entity =>
    {
        // Relacja jawnie: Order ma jednego Customera, Customer ma wiele Orders
        entity.HasOne(o => o.Customer)
            .WithMany(c => c.Orders)
            .HasForeignKey(o => o.CustomerId)
            .OnDelete(DeleteBehavior.Restrict);   // nie kasuj klienta gdy ma zamówienia

        // Wykluczenie właściwości obliczanej
        entity.Ignore(o => o.TotalAmount);
    });

    modelBuilder.Entity<OrderItem>(entity =>
    {
        entity.Ignore(oi => oi.TotalPrice);

        entity.Property(oi => oi.UnitPrice)
            .HasPrecision(18, 2);       // decimal(18,2) w bazie
    });
}
```

Alternatywnie — atrybuty (Data Annotations), ale Fluent API daje więcej możliwości i trzyma konfigurację w jednym miejscu:

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

public class Customer
{
    public int Id { get; set; }

    [Required]
    [MaxLength(200)]
    public string FullName { get; set; } = "";

    [NotMapped]   // nie mapuj na kolumnę
    public string Inicjaly => FullName[..2];
}
```

Fluent API vs Data Annotations — co wybrać? W prostych przypadkach atrybuty są OK. W większych projektach Fluent API jest czytelniejsze, bo cała konfiguracja jest w jednym pliku zamiast rozsiana po 20 klasach modeli. Oba podejścia można mieszać — Fluent API ma priorytet nad atrybutami.

### Część 3: Relacje

#### Trzy typy relacji

Relacje w bazach danych (i EF Core) sprowadzają się do trzech wzorców:

**One-to-Many (1:N)** — najczęstszy. Jeden klient ma wiele zamówień. Jedno zamówienie ma jednego klienta:

```csharp
// Strona "jeden" (Customer):
public List<Order> Orders { get; set; } = new();

// Strona "wiele" (Order):
public int CustomerId { get; set; }          // FK
public Customer Customer { get; set; } = null!; // nawigacja
```

EF Core rozpoznaje to automatycznie: widzi `CustomerId` + `Customer` w `Order` i `List<Order>` w `Customer` — i wie, że to 1:N.

**One-to-One (1:1)** — rzadszy. Na przykład zamówienie ma dokładnie jeden adres dostawy:

```csharp
public class ShippingAddress
{
    public int Id { get; set; }
    public string Street { get; set; } = "";
    public string City { get; set; } = "";

    public int OrderId { get; set; }         // FK + unikalne
    public Order Order { get; set; } = null!;
}

// W Order:
public ShippingAddress? ShippingAddress { get; set; }
```

**Many-to-Many (N:M)** — produkt może być w wielu zamówieniach, zamówienie ma wiele produktów. U nas ta relacja jest już modelowana **przez encję pośrednią `OrderItem`** (z dodatkowymi danymi jak `Quantity` i `UnitPrice`):

```
Order ──1:N── OrderItem ──N:1── Product
```

To jest świadomy wybór — tabela pośrednia z danymi to częstszy wzorzec w rzeczywistości niż czyste M:N. EF Core od wersji 5 wspiera też czyste M:N (bez klasy pośredniej), ale gdy potrzebujecie dodatkowych kolumn w tabeli łączącej (ilość, cena, data) — klasyczna encja pośrednia jest lepsza.

#### Eager loading, lazy loading, explicit loading

Gdy pobieracie zamówienie z bazy, powiązany `Customer` **nie jest ładowany automatycznie** — to byłoby kosztowne (wyobraźcie sobie pobranie 1000 zamówień i dla każdego osobne zapytanie po klienta). Macie trzy strategie:

**Eager loading** — `Include()` — powiedzcie EF Core z góry, czego potrzebujecie:

```csharp
var orders = await db.Orders
    .Include(o => o.Customer)              // załaduj klienta
    .Include(o => o.Items)                 // załaduj pozycje
        .ThenInclude(i => i.Product)       // ...i produkt w każdej pozycji
    .Where(o => o.Status == Status.Completed)
    .ToListAsync();

// Teraz order.Customer.FullName działa bez dodatkowego zapytania
```

EF Core wygeneruje SQL z JOIN-ami — jedno zapytanie, wszystkie dane. To **rekomendowany** sposób w większości przypadków.

**Lazy loading** — powiązane obiekty są ładowane automatycznie przy pierwszym dostępie do właściwości nawigacyjnej. Wygodne, ale **niebezpieczne** — generuje tzw. **N+1 problem** (pobranie 100 zamówień = 1 zapytanie po zamówienia + 100 osobnych zapytań po klientów). Nie będziemy tego używać.

**Explicit loading** — ładujecie powiązane dane ręcznie, na żądanie:

```csharp
var order = await db.Orders.FindAsync(42);
await db.Entry(order!).Reference(o => o.Customer).LoadAsync();
await db.Entry(order!).Collection(o => o.Items).LoadAsync();
```

Przydaje się, gdy nie wiedzieliście z góry, że będziecie potrzebować relacji.

#### Projekcje — `Select` zamiast `Include`

Często nie potrzebujecie **całego** obiektu z relacjami — wystarczą wybrane pola. W takim przypadku `Select` jest szybszy niż `Include`, bo EF Core pobiera z bazy tylko to, o co prosicie:

```csharp
var raport = await db.Orders
    .Where(o => o.Status == Status.Completed)
    .Select(o => new
    {
        o.Id,
        Klient = o.Customer.FullName,   // EF Core sam zrobi JOIN
        Kwota = o.Items.Sum(i => i.Quantity * i.UnitPrice),
        LiczbaPozycji = o.Items.Count
    })
    .OrderByDescending(x => x.Kwota)
    .ToListAsync();
```

Zauważcie — `o.Customer.FullName` w `Select` powoduje automatyczny JOIN, bez `Include`. EF Core jest na tyle sprytny, żeby wygenerować optymalny SQL. To jest jedna z największych zalet LINQ w kontekście baz danych.

### Część 4: Migracje

#### Problem: schemat bazy musi ewoluować razem z kodem

Dodajecie pole `Email` do klasy `Customer`. Plik bazy `orderflow.db` nadal ma starą strukturę bez tej kolumny. Ktoś klonuje repo — u niego bazy nie ma w ogóle. Jak zsynchronizować?

**Migracje** to mechanizm EF Core, który:

1. Porównuje bieżący model (klasy C#) z poprzednim stanem.
2. Generuje **plik z kodem** opisujący różnicę (dodaj kolumnę, utwórz tabelę, zmień typ...).
3. Aplikuje te zmiany do bazy danych.

To jak Git, ale dla schematu bazy — każda migracja to „commit", a ich sekwencja daje aktualny schemat.

#### Workflow

**Krok 1: Stwórzcie pierwszą migrację** po zdefiniowaniu modeli i `DbContext`:

```bash
dotnet ef migrations add InitialCreate
```

Powstanie folder `Migrations/` z plikiem `XXXXXX_InitialCreate.cs`:

```csharp
public partial class InitialCreate : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "Customers",
            columns: table => new
            {
                Id = table.Column<int>(nullable: false)
                    .Annotation("Sqlite:Autoincrement", true),
                FullName = table.Column<string>(maxLength: 200, nullable: false),
                City = table.Column<string>(nullable: false),
                IsVip = table.Column<bool>(nullable: false)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_Customers", x => x.Id);
            });
        
        // ... dalsze tabele i relacje
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropTable("Customers");
        // ... cofnięcie zmian
    }
}
```

Metoda `Up` — co zrobić (stwórz tabele). Metoda `Down` — jak cofnąć (usuń tabele). Zawsze generowane jest obie kierunki — możecie migrować „w przód" i „w tył".

**Krok 2: Zaaplikujcie migrację** do bazy:

```bash
dotnet ef database update
```

Baza `orderflow.db` zostanie utworzona (jeśli nie istniała) ze wszystkimi tabelami. EF Core trzyma w bazie tabelę `__EFMigrationsHistory`, żeby wiedzieć, które migracje już zostały zastosowane.

**Krok 3: Zmieniajcie model i dodawajcie kolejne migracje.** Na przykład dodajecie pole `Email` do `Customer`:

```csharp
public class Customer
{
    // ... istniejące pola ...
    public string? Email { get; set; }   // nowe pole (nullable, więc bezpieczne)
}
```

```bash
dotnet ef migrations add AddCustomerEmail
dotnet ef database update
```

EF Core porówna model z ostatnią migracją i wygeneruje:

```csharp
migrationBuilder.AddColumn<string>(
    name: "Email",
    table: "Customers",
    nullable: true);
```

#### Migracje w kodzie — `EnsureCreated` vs migracje

Dla prototypów i labów możecie też zastosować schemat programatycznie:

```csharp
using (var db = new OrderFlowContext())
{
    // Opcja A: migracje (rekomendowane)
    await db.Database.MigrateAsync();

    // Opcja B: stwórz bazę jeśli nie istnieje, bez migracji
    // (wygodne do nauki, ale NIE współgra z migracjami!)
    // await db.Database.EnsureCreatedAsync();
}
```

`MigrateAsync()` aplikuje wszystkie oczekujące migracje. `EnsureCreatedAsync()` tworzy bazę od razu z aktualnym schematem, ale **nie zna migracji** — nie można go mieszać z `dotnet ef`. Na labie użyjcie migracji od początku — dobra praktyka, zerowy dodatkowy wysiłek.

#### Przydatne komendy

```bash
# Lista migracji i ich status
dotnet ef migrations list

# Cofnięcie ostatniej migracji (jeśli jeszcze nie zaaplikowana)
dotnet ef migrations remove

# Cofnięcie bazy do konkretnej migracji
dotnet ef database update NazwaMigracji

# Cofnięcie wszystkich migracji (pusta baza)
dotnet ef database update 0

# Wygenerowanie skryptu SQL zamiast bezpośredniej aktualizacji
dotnet ef migrations script
```

### Część 5: Seeding — dane początkowe

#### Problem: pusta baza po migracji

Po `dotnet ef database update` macie schemat, ale zero danych. Na Lab 1 mieliście `SampleData` z listami in-memory. Teraz te dane powinny trafiać do bazy.

EF Core ma wbudowany mechanizm seedowania — dane definiujecie w `OnModelCreating`, a EF Core generuje migrację, która je wstawia:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // ... konfiguracja relacji ...

    modelBuilder.Entity<Customer>().HasData(
        new Customer { Id = 1, FullName = "Jan Kowalski", City = "Warszawa", IsVip = true },
        new Customer { Id = 2, FullName = "Anna Nowak", City = "Kraków", IsVip = false },
        new Customer { Id = 3, FullName = "Piotr Wiśniewski", City = "Warszawa", IsVip = false },
        new Customer { Id = 4, FullName = "Kasia Zielińska", City = "Gdańsk", IsVip = true }
    );

    modelBuilder.Entity<Product>().HasData(
        new Product { Id = 1, Name = "Shure SM7B", Price = 1899.00m, Category = "Mikrofony" },
        new Product { Id = 2, Name = "Kabel XLR", Price = 49.99m, Category = "Akcesoria" },
        new Product { Id = 3, Name = "Interface Focusrite", Price = 899.00m, Category = "Interfejsy" },
        new Product { Id = 4, Name = "Pop filtr", Price = 29.99m, Category = "Akcesoria" },
        new Product { Id = 5, Name = "Statyw mikrofonowy", Price = 149.99m, Category = "Akcesoria" }
    );
}
```

> :warning: **Przy `HasData` musicie podać wartości `Id` jawnie** — EF Core musi wiedzieć, które rekordy już istnieją (żeby nie dublować przy ponownym uruchomieniu). To jest jedyne miejsce, gdzie ręcznie ustawiacie klucz główny.

Po dodaniu danych uruchamiacie:

```bash
dotnet ef migrations add SeedInitialData
dotnet ef database update
```

Migracja będzie zawierać `INSERT`-y. Przy kolejnych uruchomieniach dane nie będą zdublowane, bo EF Core zna ich `Id`.

#### Seeding warunkowy — dla danych dynamicznych

`HasData` jest sztywne — dla danych, które mogą się zmieniać lub zależą od logiki, lepsze jest ręczne seedowanie w kodzie:

```csharp
public static class DatabaseSeeder
{
    public static async Task SeedAsync(OrderFlowContext db)
    {
        if (await db.Customers.AnyAsync())
            return;   // baza już ma dane — nie seeduj ponownie

        var customers = new List<Customer>
        {
            new() { FullName = "Jan Kowalski", City = "Warszawa", IsVip = true },
            new() { FullName = "Anna Nowak", City = "Kraków", IsVip = false }
        };

        db.Customers.AddRange(customers);
        await db.SaveChangesAsync();

        // Teraz customers mają przypisane Id (autoincrement)
        var orders = new List<Order>
        {
            new()
            {
                CustomerId = customers[0].Id,
                OrderDate = DateTime.Now.AddDays(-5),
                Status = Status.Completed,
                Items = new List<OrderItem>
                {
                    new() { ProductId = 1, Quantity = 1, UnitPrice = 1899.00m }
                }
            }
        };

        db.Orders.AddRange(orders);
        await db.SaveChangesAsync();
    }
}

// W Program.cs:
using var db = new OrderFlowContext();
await db.Database.MigrateAsync();
await DatabaseSeeder.SeedAsync(db);
```

Ta metoda jest elastyczniejsza — możecie generować dane losowo, ładować z pliku JSON (hej, Lab 3!), warunkowo sprawdzać stan bazy.

### Część 6: Operacje CRUD

#### Create — dodawanie encji

```csharp
using var db = new OrderFlowContext();

// Pojedynczy obiekt
var newCustomer = new Customer
{
    FullName = "Nowy Klient",
    City = "Łódź",
    IsVip = false
};
db.Customers.Add(newCustomer);
await db.SaveChangesAsync();
// newCustomer.Id jest teraz ustawione (np. 5)

// Wiele obiektów naraz
var products = new List<Product>
{
    new() { Name = "Mikrofon USB", Price = 299.00m, Category = "Mikrofony" },
    new() { Name = "Słuchawki", Price = 199.00m, Category = "Audio" }
};
db.Products.AddRange(products);
await db.SaveChangesAsync();
```

Kluczowy moment: `Add` / `AddRange` jeszcze nic nie zapisuje do bazy — tylko oznacza obiekty jako „do wstawienia". Dopiero `SaveChangesAsync()` wykonuje faktyczny `INSERT`. To pozwala zgrupować wiele zmian w jedną operację.

#### Read — zapytania

Znany Wam już LINQ — ale teraz zamiast listy in-memory odpytujecie bazę:

```csharp
// Wszystkie zamówienia — UWAGA: pobiera WSZYSTKO, unikajcie na dużych tabelach
var all = await db.Orders.ToListAsync();

// Filtrowanie
var completedOrders = await db.Orders
    .Where(o => o.Status == Status.Completed)
    .ToListAsync();

// Szukanie po Id
var order = await db.Orders.FindAsync(42);

// Pierwszy pasujący (lub null)
var latest = await db.Orders
    .OrderByDescending(o => o.OrderDate)
    .FirstOrDefaultAsync();

// Projekcja — pobiera tylko potrzebne kolumny
var summary = await db.Orders
    .Select(o => new { o.Id, o.Status, Klient = o.Customer.FullName })
    .ToListAsync();

// Grupowanie — ile zamówień per status
var stats = await db.Orders
    .GroupBy(o => o.Status)
    .Select(g => new { Status = g.Key, Count = g.Count() })
    .ToListAsync();
```

**Ważne:** Dopóki nie wywołacie `ToListAsync()`, `FirstOrDefaultAsync()`, `CountAsync()` itp. — zapytanie **nie jest wysyłane** do bazy. To ta sama leniwa ewaluacja co w LINQ z zajęć 1. Możecie budować zapytanie krok po kroku:

```csharp
IQueryable<Order> query = db.Orders;

if (statusFilter != null)
    query = query.Where(o => o.Status == statusFilter);

if (minAmount > 0)
    query = query.Where(o => o.Items.Sum(i => i.Quantity * i.UnitPrice) > minAmount);

var results = await query.ToListAsync();  // ← dopiero teraz SQL
```

Zwróćcie uwagę na typ `IQueryable<T>` — to nie jest `IEnumerable<T>`! `IQueryable` buduje drzewo wyrażeń, które EF Core tłumaczy na SQL. `IEnumerable` iteruje w pamięci. Jeśli zrobicie `ToList()` za wcześnie, reszta filtrów wykona się w pamięci zamiast w bazie — to częsty błąd wydajnościowy.

#### Update — modyfikacja

```csharp
var customer = await db.Customers.FindAsync(1);
if (customer != null)
{
    customer.City = "Wrocław";
    customer.IsVip = true;
    await db.SaveChangesAsync();   // UPDATE Customers SET City='Wrocław', IsVip=1 WHERE Id=1
}
```

EF Core **śledzi zmiany** (change tracking) — wie, że zmieniliście `City` i `IsVip`, więc generuje `UPDATE` tylko na te kolumny. Nie musicie mówić „co się zmieniło" — EF Core sam porównuje stan obiektu z oryginałem.

#### Delete — usuwanie

```csharp
var product = await db.Products.FindAsync(4);
if (product != null)
{
    db.Products.Remove(product);
    await db.SaveChangesAsync();   // DELETE FROM Products WHERE Id=4
}
```

Przy relacjach z `OnDelete(DeleteBehavior.Cascade)` usunięcie klienta usunie też jego zamówienia. Z `Restrict` — dostaniecie wyjątek, jeśli klient ma zamówienia. Wybór zależy od logiki biznesowej — kasowanie kaskadowe jest wygodne, ale ryzykowne.

### Część 7: Transakcje

#### Problem: spójność danych

Przetwarzacie zamówienie: zmieniacie status na `Completed`, zmniejszacie stan magazynowy, twórzycie rekord płatności. Co jeśli po zmianie statusu program padnie — a stan magazynowy nie został zaktualizowany? Macie niespójne dane.

**Transakcja** to mechanizm „wszystko albo nic" — albo wszystkie operacje się udają, albo żadna zmiana nie trafia do bazy.

#### SaveChanges = jedna transakcja

Domyślnie `SaveChangesAsync()` opakowuje **wszystkie oczekujące zmiany** w jedną transakcję. Jeśli cokolwiek się nie uda — nic nie jest zapisane:

```csharp
using var db = new OrderFlowContext();

var order = await db.Orders.FindAsync(1);
order!.Status = Status.Completed;

var newLog = new OrderLog { OrderId = 1, Message = "Completed", Timestamp = DateTime.Now };
db.OrderLogs.Add(newLog);

// Obie zmiany w jednej transakcji — albo obie, albo żadna
await db.SaveChangesAsync();
```

To wystarcza w większości przypadków — grupujecie zmiany i wołacie `SaveChanges` raz.

#### Jawna transakcja — `BeginTransaction`

Gdy musicie kontrolować transakcję sami (np. obejmuje wiele `SaveChanges` albo łączy się z inną operacją):

```csharp
using var db = new OrderFlowContext();
using var transaction = await db.Database.BeginTransactionAsync();

try
{
    // Krok 1: zmiana statusu zamówienia
    var order = await db.Orders
        .Include(o => o.Items)
        .FirstAsync(o => o.Id == orderId);
    order.Status = Status.Processing;
    await db.SaveChangesAsync();

    // Krok 2: aktualizacja stanów magazynowych
    foreach (var item in order.Items)
    {
        var product = await db.Products.FindAsync(item.ProductId);
        if (product!.Stock < item.Quantity)
            throw new InvalidOperationException(
                $"Brak towaru: {product.Name}");

        product.Stock -= item.Quantity;
    }
    await db.SaveChangesAsync();

    // Krok 3: zmiana statusu na Completed
    order.Status = Status.Completed;
    await db.SaveChangesAsync();

    // Wszystko OK — zatwierdzamy
    await transaction.CommitAsync();
}
catch (Exception)
{
    // Coś poszło nie tak — wycofujemy WSZYSTKIE zmiany
    await transaction.RollbackAsync();
    throw;
}
```

Jeśli wyjątek poleci w kroku 2 (brak towaru), `RollbackAsync()` cofnie zmianę statusu z kroku 1. Baza pozostaje spójna.

#### ExecutionStrategy — retry przy tymczasowych błędach

Bazy danych potrafią zwracać tymczasowe błędy (timeout, deadlock). EF Core pozwala skonfigurować automatyczne ponawianie:

```csharp
options.UseSqlite("Data Source=orderflow.db",
    sqliteOptions => sqliteOptions.CommandTimeout(30));
```

Dla SQLite to mniej istotne, ale w SQL Server / PostgreSQL strategia retry jest standardem produkcyjnym.

### Część 8: Logowanie wygenerowanego SQL

Żeby zobaczyć, co EF Core naprawdę wysyła do bazy, włączcie logowanie:

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder options)
{
    options
        .UseSqlite("Data Source=orderflow.db")
        .LogTo(Console.WriteLine, LogLevel.Information)       // SQL na konsolę
        .EnableSensitiveDataLogging();                        // pokaż wartości parametrów
}
```

Teraz przy każdym zapytaniu zobaczycie w konsoli wygenerowany SQL. To **nieocenione** do debugowania — gdy LINQ daje nieoczekiwane wyniki, patrzenie na SQL natychmiast wyjaśnia co się dzieje.

> :warning: `EnableSensitiveDataLogging` pokazuje dane (np. wartości parametrów WHERE). Na produkcji to wyłączcie — ale na labie chcecie to widzieć.

### Podsumowanie

```
DbContext / DbSet<T>
  → Brama do bazy — definicja tabel, konfiguracja, zapytania

Konwencje + Fluent API + Data Annotations
  → Mapowanie klas na tabele — od „zero konfiguracji" do pełnej kontroli

Relacje (1:N, 1:1, N:M)
  → Nawigacje + klucze obce, Include() do ładowania powiązanych danych

Migracje
  → Ewolucja schematu bazy — dotnet ef migrations add/update
  → Traktuj jak Git dla struktury bazy danych

Seeding
  → HasData (w OnModelCreating) lub ręczne seedowanie
  → Zapewnia dane startowe po migracji

CRUD
  → Add → SaveChanges (INSERT), zmień property → SaveChanges (UPDATE)
  → Remove → SaveChanges (DELETE), LINQ → ToListAsync (SELECT)
  → IQueryable ≠ IEnumerable — pilnujcie, gdzie kończy się SQL a zaczyna C#

Transakcje
  → SaveChanges = domyślna transakcja
  → BeginTransaction + Commit/Rollback dla złożonych scenariuszy

Logowanie SQL
  → .LogTo(Console.WriteLine) — zobaczcie co naprawdę robi EF Core
```

---

## Laboratorium 4 — Baza danych w OrderFlow

**Projekt:** OrderFlow — kontynuacja

**Tematy:** EF Core, DbContext, relacje, migracje, transakcje, seeding, CRUD, zapytania LINQ na bazie

Pracujecie w tej samej solucji co na Lab 1–3. Główna zmiana: zamiast plików JSON/XML z Lab 3, dane trafiają teraz do bazy danych SQLite. Model domenowy z Lab 1 pozostaje — tylko dodajecie nawigacje i konfigurację EF Core.

```
OrderFlow.Console/
├── Models/                 ← rozbudowa: nawigacje, ewentualnie nowe pola
├── Services/
├── Persistence/
│   ├── OrderFlowContext.cs ← nowość: DbContext
│   └── DatabaseSeeder.cs   ← nowość: seedowanie
├── Migrations/             ← wygenerowane przez dotnet ef
├── Watchers/
└── Program.cs
```

Przed rozpoczęciem zainstalujcie paczki:

```bash
dotnet add package Microsoft.EntityFrameworkCore.Sqlite
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet tool install --global dotnet-ef    # jeśli jeszcze nie macie
```

---

### Zadanie 1 — DbContext, relacje i migracje (5 pkt)

Skonfigurujcie EF Core dla istniejącego modelu OrderFlow.

**Wymagania:**

1. Stwórzcie klasę `OrderFlowContext : DbContext` z `DbSet`-ami dla: `Product`, `Customer`, `Order`, `OrderItem`.
2. Skonfigurujcie relacje za pomocą **Fluent API** w `OnModelCreating`:
    - `Customer` → `Order` (1:N) z `DeleteBehavior.Restrict`.
    - `Order` → `OrderItem` (1:N) z `DeleteBehavior.Cascade`.
    - `OrderItem` → `Product` (N:1).
    - Wykluczcie z mapowania właściwości obliczane (`TotalAmount`, `TotalPrice`) za pomocą `Ignore`.
    - Ustawcie `HasPrecision(18, 2)` na kolumnach `decimal` (`UnitPrice`, `Price`).
    - Dodajcie **indeks** na `Customer.FullName` i na `Order.Status`.
3. Wygenerujcie migrację `InitialCreate` i zaaplikujcie ją.
4. Następnie dodajcie **nowe pole** `Email` (string, nullable) do klasy `Customer` i pole `Notes` (string, nullable) do klasy `Order`. Wygenerujcie drugą migrację `AddEmailAndNotes`.
5. Włączcie **logowanie SQL** na konsolę (`.LogTo(Console.WriteLine)`).

**Pokażcie w konsoli:** że `dotnet ef migrations list` wyświetla obie migracje, a `dotnet ef database update` tworzy bazę.

---

### Zadanie 2 — Seeding i CRUD (5 pkt)

Wypełnijcie bazę danymi i pokażcie podstawowe operacje.

**Wymagania:**

1. Stwórzcie klasę `DatabaseSeeder` z metodą `Task SeedAsync(OrderFlowContext db)`, która:
    - Sprawdza, czy baza jest pusta (`AnyAsync`).
    - Wstawia dane z `SampleData` (Lab 1) — minimum 5 produktów, 4 klientów (w tym VIP), 6 zamówień z pozycjami.
    - Używa `AddRange` i `SaveChangesAsync`.
2. W `Program.cs` na starcie wywołajcie `MigrateAsync()` + `SeedAsync()`.
3. Pokażcie w konsoli **operacje CRUD**:
    - **Create:** Dodajcie nowe zamówienie z 2 pozycjami dla istniejącego klienta.
    - **Read:** Pobierzcie zamówienia z `Include` na `Customer` i `Items.Product` — wypiszcie szczegóły.
    - **Update:** Zmieńcie status zamówienia z `New` na `Processing` i zaktualizujcie `Notes`.
    - **Delete:** Usuńcie zamówienie o statusie `Cancelled` (lub pokażcie, że próba usunięcia klienta z zamówieniami rzuca wyjątek dzięki `Restrict`).
4. Po każdej operacji CRUD wypiszcie wynik w konsoli (np. „Dodano zamówienie #7 z 2 pozycjami").

---

### Zadanie 3 — Zapytania i transakcje (5 pkt)

Napiszcie zaawansowane zapytania LINQ na `DbContext` i pokażcie transakcje.

**Wymagania:**

1. Zaimplementujcie minimum **5 zapytań** LINQ (na `IQueryable`, nie `IEnumerable`!) — pokażcie w konsoli wynik każdego:
    - Zamówienia klientów VIP z kwotą powyżej zadanego progu (wymaga `Include` lub `Select` z nawigacją).
    - Ranking klientów wg łącznej wartości zamówień (`GroupBy` + `Sum` + `OrderByDescending`).
    - Średnia wartość zamówienia per miasto klienta (join przez nawigację).
    - Produkty, które **nigdy nie zostały zamówione** (anti-join: `Where(p => !p.OrderItems.Any())`).
    - Dynamicznie budowane zapytanie — użytkownik podaje opcjonalny filtr statusu i minimalną kwotę, zapytanie buduje się warunkowo (`IQueryable` + `if`).

2. Zaimplementujcie **transakcję** procesowania zamówienia:
    - Metoda `Task ProcessOrderAsync(OrderFlowContext db, int orderId)`.
    - W transakcji: zmień status `New → Processing`, sprawdź dostępność towaru (dodajcie pole `Stock` do `Product`, jeśli go nie macie — nowa migracja!), zmniejsz stany magazynowe, zmień status na `Completed`.
    - Jeśli jakiegoś produktu brak — `Rollback` i rzuć wyjątek.
    - Pokażcie w konsoli scenariusz sukcesu i scenariusz niepowodzenia (celowo ustawcie niski `Stock`).

---

### Punktacja

| Zadanie | Punkty |
|---|---|
| Wysłanie na GitHub z commitem z zajęć | **5 pkt** |
| 1. DbContext, relacje, migracje | 5 pkt |
| 2. Seeding i CRUD | 5 pkt |
| 3. Zapytania i transakcje | 5 pkt |
| **Razem** | **20 pkt** |

> **Uwaga o GitHub:** Jak zawsze — co najmniej 1 commit z czasów zajęć. Folder `Migrations/` **commitujcie** (to kod, nie dane). Plik `orderflow.db` dodajcie do `.gitignore`.

### Wskazówki

- Zacznijcie od zadania 1 — bez `DbContext` i migracji nie ruszycie z miejsca. Sprawdźcie, że `dotnet ef database update` przechodzi bez błędów, zanim pójdziecie dalej.
- W zadaniu 2 seedowanie zamówień z pozycjami wymaga kolejności: najpierw `SaveChanges` dla klientów i produktów (żeby dostały `Id`), potem dopiero twórzcie zamówienia z `CustomerId` i `OrderItem` z `ProductId`.
- W zadaniu 3 włączcie logowanie SQL — zobaczcie, jakie zapytania generuje EF Core. Jeśli zapytanie wygląda dziwnie (np. pobiera 1000 rekordów zamiast 5), sprawdźcie czy nie wywołaliście `ToList()` za wcześnie.
- Pamiętajcie o `using` przy tworzeniu `DbContext` — tak jak przy `StreamWriter` z Lab 3.
- Jeżeli dostajecie błędy migracji („model has changed"), sprawdźcie czy `Ignore()` jest ustawione dla właściwości obliczanych (`TotalAmount`, `TotalPrice`). EF Core próbuje mapować każdą publiczną właściwość z getterem i setterem.
- Na kolejnych zajęciach (Lab 5) piszemy testy jednostkowe — EF Core z SQLite świetnie się do tego nadaje (in-memory provider lub `Data Source=:memory:`). Dobrze napisany `DbContext` ułatwi testowanie.