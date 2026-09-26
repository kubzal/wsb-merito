# Zajęcia 3

## Pliki, serializacja, LINQ to XML, FileSystemWatcher

Do tej pory OrderFlow trzymał wszystko w pamięci — po zamknięciu programu dane znikały. Dziś naprawiamy tę fundamentalną wadę: uczymy program zapisywać i wczytywać stan. Po drodze poznajemy strumienie (`System.IO`), dwa najpopularniejsze formaty wymiany danych (JSON i XML), wygodne API do budowania XML-a programatycznie (LINQ to XML) oraz mechanizm reagujący na zmiany w systemie plików (`FileSystemWatcher`).

### Część 1: Operacje na plikach i strumieniach (System.IO)

#### Od czego zaczynamy

Przestrzeń nazw `System.IO` to fundament — wszystko co dotyczy plików, katalogów i strumieni żyje właśnie tam. Na najwyższym poziomie mamy trzy statyczne klasy-pomocniki:

- **`File`** — pojedyncze pliki (`ReadAllText`, `WriteAllText`, `Exists`, `Delete`, `Copy`…).
- **`Directory`** — katalogi (`CreateDirectory`, `GetFiles`, `Exists`, `Delete`…).
- **`Path`** — składanie i analiza ścieżek (`Combine`, `GetFileName`, `GetExtension`…).

#### Najprostszy zapis i odczyt

Dla małych plików tekstowych wystarczy jedna linijka:

```csharp
// Zapis — nadpisuje plik, jeśli istnieje
File.WriteAllText("notatka.txt", "Cześć z C#!");

// Odczyt — cały plik do stringa
string tresc = File.ReadAllText("notatka.txt");
Console.WriteLine(tresc);

// Linia po linii
string[] linie = File.ReadAllLines("notatka.txt");
foreach (var l in linie)
    Console.WriteLine(l);

// Dopisanie (append) — nie nadpisuje, dokleja
File.AppendAllText("log.txt", $"[{DateTime.Now:HH:mm:ss}] start\n");
```

W Pythonie napisalibyście `open("notatka.txt").read()` — w C# `File.ReadAllText` jest bliskim odpowiednikiem, tylko że od razu zamyka plik po odczycie (nie musicie pamiętać o `close`).

> :warning: **Uwaga!** `ReadAllText` wczytuje **cały plik do pamięci**. Dla pliku 10 KB — super. Dla loga o rozmiarze 5 GB — zapłaczecie. Od tego są strumienie, o których za chwilę.

#### Ścieżki — `Path.Combine` zamiast hardkodowania

Sklejanie ścieżek stringami (`"folder" + "/" + "plik.txt"`) to pułapka: na Windowsie separatorem jest `\`, na Linuksie/macOS `/`. Do tego dochodzą podwójne ukośniki, spacje i inne atrakcje. `Path.Combine` robi to dobrze — używając separatora właściwego dla bieżącego OS:

```csharp
string sciezka = Path.Combine("dane", "zamowienia", "2026.json");
// Windows: "dane\zamowienia\2026.json"
// Linux:   "dane/zamowienia/2026.json"

// Przydatne przy katalogu bieżącym aplikacji
string appDir = AppContext.BaseDirectory;
string plik = Path.Combine(appDir, "config.json");
```

Kilka innych metod z `Path`, które warto znać:

```csharp
Path.GetFileName("C:/dane/raport.pdf");        // "raport.pdf"
Path.GetFileNameWithoutExtension("raport.pdf"); // "raport"
Path.GetExtension("raport.pdf");                // ".pdf"
Path.GetDirectoryName("C:/dane/raport.pdf");    // "C:/dane"
Path.ChangeExtension("raport.pdf", ".txt");     // "raport.txt"
```

#### Katalogi

```csharp
// Utwórz katalog (rekurencyjnie — tworzy też brakujące nadrzędne)
Directory.CreateDirectory("dane/zamowienia");

// Sprawdź istnienie
if (Directory.Exists("dane"))
    Console.WriteLine("Katalog istnieje");

// Listowanie
string[] pliki = Directory.GetFiles("dane");                    // tylko tu
string[] wszystkie = Directory.GetFiles("dane", "*.json",
    SearchOption.AllDirectories);                               // rekurencyjnie
string[] podkatalogi = Directory.GetDirectories("dane");

// Usuwanie
Directory.Delete("dane/temp", recursive: true);
```

Wzorzec `*.json` to glob pattern — identycznie jak w terminalu.

#### Strumienie — gdy plik jest duży albo binarny

`File.ReadAllText` jest wygodny, ale ładuje wszystko naraz. **Strumień (stream)** to abstrakcja pozwalająca czytać/zapisywać dane **porcjami**. Nie musicie trzymać całej zawartości w pamięci — pracujecie z przepływem bajtów.

W .NET hierarchia wygląda tak:

```
Stream                    ← klasa bazowa, czyste bajty
  ├── FileStream          ← bajty z/do pliku
  ├── MemoryStream        ← bajty w pamięci (przydatne w testach)
  ├── NetworkStream       ← bajty przez sieć
  └── ...

Na to nakładamy "tekstowe" opakowania:
  StreamReader            ← czyta tekst ze strumienia (dekoduje bajty na znaki)
  StreamWriter            ← zapisuje tekst do strumienia
```

Typowy zapis linia po linii przez `StreamWriter`:

```csharp
using (var writer = new StreamWriter("log.txt"))
{
    writer.WriteLine("Start aplikacji");
    writer.WriteLine($"Czas: {DateTime.Now}");
    writer.WriteLine("Koniec");
}  // ← tu automatycznie zamyka plik
```

Odczyt linia po linii:

```csharp
using (var reader = new StreamReader("log.txt"))
{
    string? linia;
    while ((linia = reader.ReadLine()) != null)
    {
        Console.WriteLine(linia);
    }
}
```

#### `using` — czemu to jest ważne

Pliki to **zasób zewnętrzny** (system operacyjny trzyma dla was uchwyt). Jeśli go nie zwolnicie, zaczynają się problemy: nie możecie pliku usunąć, inne procesy nie mogą go otworzyć, aplikacja przecieka uchwytami.

Słowo kluczowe `using` gwarantuje, że metoda `Dispose()` zostanie wywołana — **nawet jeśli w środku poleci wyjątek**. To odpowiednik `with` z Pythona:

```python
# Python:
with open("log.txt") as f:
    f.write("hello")
# plik automatycznie zamknięty
```

```csharp
// C# — klasyczna forma z blokiem
using (var writer = new StreamWriter("log.txt"))
{
    writer.WriteLine("hello");
}  // Dispose() wywołane tutaj

// C# 8+ — using declaration (krócej)
using var writer = new StreamWriter("log.txt");
writer.WriteLine("hello");
// Dispose() wywołane na końcu BIEŻĄCEGO BLOKU (np. metody)
```

**Zasada:** wszystko co implementuje `IDisposable` (pliki, połączenia do bazy, `HttpClient`) zawsze przez `using`. Bez wyjątków.

#### Wersje asynchroniczne

Operacje na plikach to klasyczne I/O-bound — czekacie na dysk. Przypomnijcie sobie z zajęć 2: **wątek nie powinien stać i patrzeć w ścianę**. Dlatego prawie wszystkie metody mają warianty `...Async`:

```csharp
// Synchroniczne — blokuje wątek
string tresc = File.ReadAllText("plik.txt");

// Asynchroniczne — wątek jest wolny podczas czytania z dysku
string tresc = await File.ReadAllTextAsync("plik.txt");

// Strumieniowo + async
using var writer = new StreamWriter("log.txt");
await writer.WriteLineAsync("async wpis");
```

W aplikacjach konsolowych różnica jest kosmetyczna. W UI i serwerach — kluczowa. Na labie używamy wariantów `Async`, żeby wejść w dobre nawyki.

#### Wyjątki, które warto znać

```
IOException                  ← bazowy — coś poszło nie tak z I/O
├── FileNotFoundException    ← plik nie istnieje
├── DirectoryNotFoundException
└── PathTooLongException

UnauthorizedAccessException  ← brak uprawnień
```

Dobrą praktyką jest **sprawdzić istnienie przed odczytem** i opakować operacje w try/catch:

```csharp
string sciezka = "dane/orders.json";

if (!File.Exists(sciezka))
{
    Console.WriteLine("Brak pliku — używam pustej listy.");
    return new List<Order>();
}

try
{
    string json = await File.ReadAllTextAsync(sciezka);
    return JsonSerializer.Deserialize<List<Order>>(json) ?? new();
}
catch (IOException ex)
{
    Console.WriteLine($"Nie udało się odczytać pliku: {ex.Message}");
    throw;
}
```

### Część 2: Serializacja i deserializacja

#### Problem: jak wysłać obiekt przez sieć albo zapisać do pliku

Macie w pamięci obiekt `Order`. Chcecie go:

- zapisać do pliku, żeby jutro wczytać,
- wysłać przez HTTP do innego serwisu,
- zapisać w kolejce komunikatów,
- zapamiętać w cache (np. Redis).

Każdy z tych kanałów akceptuje **tekst** (albo bajty) — a nie obiekty C#. Potrzebujecie mechanizmu, który przetłumaczy obiekt na tekst i z powrotem.

```
Obiekt w pamięci  ──serializacja──▶  Tekst / bajty
Obiekt w pamięci  ◀──deserializacja── Tekst / bajty
```

Dwa najpopularniejsze formaty to **JSON** (webowy standard, krótki i czytelny) i **XML** (starszy, bardziej rozbudowany, wciąż powszechny w enterprise i dokumentach konfiguracyjnych).

#### JSON — `System.Text.Json`

W .NET historycznie królował `Newtonsoft.Json` (aka `Json.NET`). Od .NET Core 3.0 Microsoft dostarcza wbudowany `System.Text.Json` — szybszy, oparty na `Span<T>`, od razu dostępny bez dodawania paczek. To on jest dziś rekomendowany.

```csharp
using System.Text.Json;

// Nasz obiekt
var order = new Order
{
    Id = 42,
    Customer = "Jan Kowalski",
    Items = new List<string> { "Mleko", "Chleb", "Kawa" },
    Total = 29.99m
};

// SERIALIZACJA — obiekt → JSON
string json = JsonSerializer.Serialize(order);
Console.WriteLine(json);
// {"Id":42,"Customer":"Jan Kowalski","Items":["Mleko","Chleb","Kawa"],"Total":29.99}

// DESERIALIZACJA — JSON → obiekt
Order? wczytany = JsonSerializer.Deserialize<Order>(json);
Console.WriteLine(wczytany?.Customer);  // Jan Kowalski
```

Zwróćcie uwagę na `?` przy zwracanym typie — `Deserialize` może zwrócić `null` (np. gdy JSON brzmi `"null"`). Kompilator wymusza na Was jawną obsługę tego przypadku.

#### Opcje serializacji

Surowy JSON jest zwarty, ale nieczytelny. Do kontroli formatowania służy `JsonSerializerOptions`:

```csharp
var opcje = new JsonSerializerOptions
{
    WriteIndented = true,                           // wcięcia dla czytelności
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase, // PascalCase → camelCase
    DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull, // pomiń nulle
    Encoder = System.Text.Encodings.Web.JavaScriptEncoder
        .UnsafeRelaxedJsonEscaping                  // nie escape'uj polskich znaków
};

string ladny = JsonSerializer.Serialize(order, opcje);
Console.WriteLine(ladny);
```

Wynik:

```json
{
  "id": 42,
  "customer": "Jan Kowalski",
  "items": [
    "Mleko",
    "Chleb",
    "Kawa"
  ],
  "total": 29.99
}
```

#### Atrybuty — kontrola per-property

Czasem nazwa właściwości w C# nie pasuje do tego, co ma być w JSON-ie. Albo chcecie jakieś pole pominąć. Do tego służą atrybuty:

```csharp
using System.Text.Json.Serialization;

public class Order
{
    public int Id { get; set; }

    [JsonPropertyName("customer_name")]
    public string Customer { get; set; } = "";

    [JsonIgnore]
    public string HaszWewnetrzny { get; set; } = "";  // nie trafi do JSON-a

    [JsonPropertyName("total_amount")]
    public decimal Total { get; set; }
}
```

Po takiej dekoracji JSON wygląda tak:

```json
{
  "Id": 42,
  "customer_name": "Jan Kowalski",
  "total_amount": 29.99
}
```

#### Zapis i odczyt plików JSON (strumieniowo, async)

Dla dużych kolekcji lepiej nie budować wielkiego stringa w pamięci — lepiej pisać od razu do strumienia:

```csharp
// Zapis
await using (var stream = File.Create("orders.json"))
{
    await JsonSerializer.SerializeAsync(stream, orders, opcje);
}

// Odczyt
await using (var stream = File.OpenRead("orders.json"))
{
    var orders = await JsonSerializer.DeserializeAsync<List<Order>>(stream);
}
```

`await using` to połączenie `using` z `IAsyncDisposable` — zasób jest zwalniany asynchronicznie (przydaje się przy strumieniach sieciowych, flushowaniu buforów itp.).

#### XML — `XmlSerializer`

XML jest starszy, bardziej gadatliwy, ale wciąż masz z nim do czynienia w konfiguracjach (SOAP, Office, wiele systemów enterprise). W .NET klasyczne API to `XmlSerializer` z przestrzeni nazw `System.Xml.Serialization`:

```csharp
using System.Xml.Serialization;

// XmlSerializer wymaga bezparametrowego konstruktora w klasie!
public class Order
{
    public int Id { get; set; }
    public string Customer { get; set; } = "";
    public decimal Total { get; set; }
}

// SERIALIZACJA
var order = new Order { Id = 42, Customer = "Jan Kowalski", Total = 29.99m };
var serializer = new XmlSerializer(typeof(Order));

using (var writer = new StreamWriter("order.xml"))
{
    serializer.Serialize(writer, order);
}
```

Wynik (`order.xml`):

```xml
<?xml version="1.0" encoding="utf-8"?>
<Order xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <Id>42</Id>
  <Customer>Jan Kowalski</Customer>
  <Total>29.99</Total>
</Order>
```

Deserializacja:

```csharp
using (var reader = new StreamReader("order.xml"))
{
    var wczytany = (Order?)serializer.Deserialize(reader);
    Console.WriteLine(wczytany?.Customer);
}
```

> :warning: **Dwa typowe błędy na start:**
>
> 1. Brak **bezparametrowego konstruktora** — `XmlSerializer` rzuci wyjątek na etapie tworzenia. JSON jest tolerancyjniejszy.
> 2. Próba serializacji **typu anonimowego** albo `Dictionary<TKey, TValue>` — nie działa out-of-the-box w XML. JSON radzi sobie bez problemu.

#### Atrybuty XML

Analogicznie do JSON-a, możecie kontrolować wygląd XML-a atrybutami:

```csharp
using System.Xml.Serialization;

[XmlRoot("order")]
public class Order
{
    [XmlAttribute("id")]           // <order id="42">
    public int Id { get; set; }

    [XmlElement("customer")]       // <customer>...</customer>
    public string Customer { get; set; } = "";

    [XmlIgnore]
    public string HaszWewnetrzny { get; set; } = "";

    [XmlArray("items")]            // <items>
    [XmlArrayItem("item")]         //   <item>...</item>
    public List<string> Items { get; set; } = new();
}
```

Daje to bardziej „ludzki" XML:

```xml
<order id="42">
  <customer>Jan Kowalski</customer>
  <items>
    <item>Mleko</item>
    <item>Chleb</item>
  </items>
</order>
```

Zwróćcie uwagę na różnicę **element vs atrybut**:

- **Element** — `<customer>Jan</customer>` — może mieć dzieci, długą zawartość, strukturę.
- **Atrybut** — `<order id="42">` — prosta wartość, klucz-wartość, bez struktury.

Konwencja: dane "identyfikujące" i metadane zwykle jako atrybuty, treść jako elementy.

#### JSON vs XML — kiedy co?

| Cecha | JSON | XML |
|---|---|---|
| Zwięzłość | Zwarty | Gadatliwy |
| Czytelność | Bardzo dobra | Dobra (przy dobrym formatowaniu) |
| Wsparcie w webie | Standard (REST, JS) | Rzadziej (SOAP, starsze API) |
| Schema/walidacja | JSON Schema (osobny standard) | XSD (wbudowane w ekosystem) |
| Atrybuty + zagnieżdżenia | Tylko zagnieżdżenia | Atrybuty + elementy |
| Komentarze | ❌ (standard nie pozwala) | ✅ (`<!-- komentarz -->`) |
| Typ danych | Prymitywy + string + null + tablica + obiekt | Wszystko to string (typy opcjonalnie przez XSD) |
| Typowe użycie | Web API, konfiguracja nowoczesnych aplikacji, NoSQL | Konfiguracja Windows/.NET, SOAP, Office, kontrakty legacy |

W 2026 roku w nowych projektach webowych wybieracie JSON. Z XML-em spotykacie się, gdy: (a) integrujecie się z czymś starszym, (b) pracujecie z dokumentami Office/OpenDocument, (c) obsługujecie konfigurację typu `app.config`, (d) czytacie pliki MusicXML, SVG, RSS itd.

### Część 3: LINQ to XML

#### Dlaczego jeszcze jedno API do XML?

`XmlSerializer` jest wygodny, gdy **macie klasę i chcecie ją wymapować 1:1** na XML. Ale czasem:

- XML nie pasuje dokładnie do Waszej klasy (np. chcecie tylko kilka pól z cudzego schematu),
- budujecie XML **dynamicznie**, nie z istniejącego obiektu,
- chcecie **przeszukiwać** XML jak kolekcję — `zamówienia powyżej 1000 zł`, `unikalne miasta klientów`.

Wtedy wygodniejsze jest `System.Xml.Linq` — czyli LINQ to XML. To model obiektowy reprezentujący XML jako drzewo obiektów, po którym można chodzić LINQ-iem.

#### Główne klasy

```
XDocument    ← cały dokument XML (może mieć deklarację <?xml ... ?>)
XElement     ← pojedynczy element <tag>...</tag>  ← używacie najczęściej
XAttribute   ← atrybut wewnątrz elementu, np. id="42"
```

Każdy `XElement` ma `Name`, `Value`, dzieci (`Elements()`), atrybuty (`Attributes()`) — pełne drzewo dostępne z każdego miejsca.

#### Budowanie XML programatycznie

Oto typowy "wow moment" LINQ to XML. Zagnieżdżając konstruktory `XElement`, tworzycie drzewo w jednym wyrażeniu:

```csharp
using System.Xml.Linq;

var doc = new XDocument(
    new XDeclaration("1.0", "utf-8", "yes"),
    new XElement("orders",
        new XElement("order",
            new XAttribute("id", 1),
            new XElement("customer", "Jan Kowalski"),
            new XElement("total", 29.99)
        ),
        new XElement("order",
            new XAttribute("id", 2),
            new XElement("customer", "Anna Nowak"),
            new XElement("total", 150.00)
        )
    )
);

doc.Save("orders.xml");
Console.WriteLine(doc);
```

Wynik:

```xml
<?xml version="1.0" encoding="utf-8" standalone="yes"?>
<orders>
  <order id="1">
    <customer>Jan Kowalski</customer>
    <total>29.99</total>
  </order>
  <order id="2">
    <customer>Anna Nowak</customer>
    <total>150</total>
  </order>
</orders>
```

Z istniejącej kolekcji możecie wygenerować XML przez `Select`:

```csharp
var orders = new List<(int Id, string Customer, decimal Total)>
{
    (1, "Jan Kowalski", 29.99m),
    (2, "Anna Nowak", 150.00m),
    (3, "Piotr Wiśniewski", 499.50m)
};

var doc = new XDocument(
    new XElement("orders",
        from o in orders
        select new XElement("order",
            new XAttribute("id", o.Id),
            new XElement("customer", o.Customer),
            new XElement("total", o.Total))
    )
);
```

Zauważcie, że to jest ta sama składnia zapytań LINQ, którą znacie z zajęć 1 — tylko efektem jest drzewo XML, a nie lista.

#### Wczytywanie i przeszukiwanie

```csharp
// Wczytanie pliku
XDocument doc = XDocument.Load("orders.xml");

// Wszystkie zamówienia (Descendants szuka rekurencyjnie)
IEnumerable<XElement> wszystkie = doc.Descendants("order");

// Zapytanie LINQ
var drogie = from o in doc.Descendants("order")
             where (decimal)o.Element("total")! > 100
             select new
             {
                 Id = (int)o.Attribute("id")!,
                 Klient = o.Element("customer")?.Value,
                 Kwota = (decimal)o.Element("total")!
             };

foreach (var o in drogie)
    Console.WriteLine($"#{o.Id} — {o.Klient} — {o.Kwota:C}");
```

Zwróćcie uwagę na **rzutowanie** — `(decimal)o.Element("total")`. `XElement` ma zdefiniowane **jawne konwersje** do typów prymitywnych. Wykonuje za Was parsowanie, uwzględnia `null`, i jest idiomatyczne. Alternatywnie:

```csharp
decimal kwota = decimal.Parse(o.Element("total")!.Value);  // ręcznie
decimal kwota2 = (decimal)o.Element("total")!;             // LINQ to XML way
```

#### Modyfikacja drzewa

```csharp
XDocument doc = XDocument.Load("orders.xml");

// Dodanie nowego zamówienia
doc.Root!.Add(new XElement("order",
    new XAttribute("id", 3),
    new XElement("customer", "Nowy Klient"),
    new XElement("total", 99.99)));

// Zmiana wartości
foreach (var o in doc.Descendants("order"))
{
    var total = o.Element("total")!;
    decimal kwota = (decimal)total;
    total.Value = (kwota * 1.23m).ToString();  // +VAT
}

// Usunięcie elementu
doc.Descendants("order")
   .Where(o => (decimal)o.Element("total")! < 50)
   .Remove();

doc.Save("orders_updated.xml");
```

#### XmlSerializer vs LINQ to XML — kiedy co?

| Scenariusz | Wybór |
|---|---|
| Mam klasę i chcę ją zapisać/wczytać 1:1 | `XmlSerializer` |
| Buduję XML dynamicznie (różne struktury) | LINQ to XML |
| Czytam cudzy XML i biorę tylko niektóre pola | LINQ to XML |
| Przeszukuję, filtruję, agreguję XML | LINQ to XML |
| Muszę mieć dokładną kontrolę nad strukturą | LINQ to XML |
| Szybkie prototypowanie DTO | `XmlSerializer` |

Zasada kciuka: **`XmlSerializer` dla obiektów, LINQ to XML dla drzew**.

### Część 4: FileSystemWatcher

#### Problem: reagowanie na zmiany w katalogu

Wyobraźcie sobie: klient wrzuca nowy plik CSV do folderu `inbox`. Chcecie go automatycznie wczytać i zaimportować do systemu. Tradycyjnie musielibyście co sekundę odpytywać katalog (polling) — mało eleganckie i obciąża dysk.

System operacyjny (Windows, Linux, macOS) ma **wbudowany mechanizm powiadomień** o zmianach w systemie plików. `FileSystemWatcher` to opakowanie tego API w .NET.

#### Podstawowe użycie

```csharp
using System.IO;

var watcher = new FileSystemWatcher
{
    Path = "inbox",
    Filter = "*.json",                        // reaguj tylko na pliki JSON
    NotifyFilter = NotifyFilters.FileName
                 | NotifyFilters.LastWrite    // na jakie zmiany reagujemy
};

// Podłączenie zdarzeń — pamiętacie z zajęć 2?
watcher.Created += (sender, e) =>
    Console.WriteLine($"Nowy plik: {e.FullPath}");

watcher.Changed += (sender, e) =>
    Console.WriteLine($"Zmiana: {e.FullPath}");

watcher.Deleted += (sender, e) =>
    Console.WriteLine($"Usunięto: {e.FullPath}");

watcher.Renamed += (sender, e) =>
    Console.WriteLine($"Zmieniono nazwę: {e.OldFullPath} → {e.FullPath}");

watcher.EnableRaisingEvents = true;           // ← to włącza obserwację

Console.WriteLine("Obserwuję folder 'inbox'. Wciśnij Enter, aby zakończyć.");
Console.ReadLine();
```

Zwróćcie uwagę — to **dokładnie ten sam wzorzec**, co zdarzenia z zajęć 2. `Created`, `Changed`, `Deleted`, `Renamed` to eventy, a Wy podpinacie się przez `+=`. Różnica: zamiast Wy sami emitujecie zdarzenie, tu to system operacyjny dostarcza informację.

#### `NotifyFilter` — na co konkretnie reagujecie

Bez wskazania `NotifyFilter` `Changed` odpali się tylko przy zmianach rozmiaru lub atrybutu. Lista filtrów do wyboru (można łączyć `|`):

```
NotifyFilters.FileName        ← utworzenie/usunięcie/zmiana nazwy pliku
NotifyFilters.DirectoryName   ← jak wyżej, ale dla katalogów
NotifyFilters.Attributes      ← np. readonly, hidden
NotifyFilters.Size            ← zmiana rozmiaru
NotifyFilters.LastWrite       ← zmiana treści (edycja pliku)
NotifyFilters.LastAccess      ← odczyt pliku (uwaga: wyłączone na wielu systemach)
NotifyFilters.CreationTime    ← zmiana daty utworzenia
NotifyFilters.Security        ← zmiany ACL
```

#### Dwie pułapki, o których musicie wiedzieć

**Pułapka 1: wiele zdarzeń na jeden plik**

Edytor tekstu zapisując plik może wygenerować 3-4 zdarzenia `Changed` pod rząd (bo pisze nagłówek, potem treść, potem flushuje). Nie powinniście tego debugować — powinniście to **zaakceptować** i zaprojektować obsługę idempotentnie (albo debounce'ować — poczekać np. 500 ms od ostatniej zmiany, dopiero wtedy reagować).

**Pułapka 2: plik może być jeszcze zajęty**

Gdy `Created` się odpala, system mówi "ktoś zaczął tworzyć plik" — ale zapis może jeszcze trwać. Jeśli od razu spróbujecie otworzyć plik do odczytu, dostaniecie `IOException: file is being used by another process`. Typowe rozwiązanie:

```csharp
async Task<string?> BezpiecznyOdczytAsync(string sciezka, int proby = 5)
{
    for (int i = 0; i < proby; i++)
    {
        try
        {
            return await File.ReadAllTextAsync(sciezka);
        }
        catch (IOException)
        {
            await Task.Delay(200);  // poczekaj i spróbuj ponownie
        }
    }
    return null;
}
```

#### `IncludeSubdirectories`

Domyślnie watcher pilnuje tylko wskazanego katalogu. Żeby objąć również podkatalogi:

```csharp
watcher.IncludeSubdirectories = true;
```

Uwaga: przy dużych drzewach katalogów (np. cały dysk C:) generuje to lawinę zdarzeń. Używajcie z rozsądkiem.

#### `InternalBufferSize`

`FileSystemWatcher` trzyma wewnętrzny bufor na zdarzenia z systemu operacyjnego. Przy intensywnych zmianach (kopiowanie 10 000 plików naraz) bufor może się przepełnić i **niektóre zdarzenia po prostu zginą** — odpali się wtedy zdarzenie `Error`. Dla aplikacji produkcyjnych zwiększa się bufor (max 64 KB):

```csharp
watcher.InternalBufferSize = 64 * 1024;
watcher.Error += (s, e) =>
    Console.WriteLine($"Watcher error: {e.GetException().Message}");
```

### Podsumowanie

```
System.IO (File, Directory, Path)
  → Podstawowe operacje — czytaj/pisz małe pliki, sklejaj ścieżki

Strumienie (FileStream, StreamReader/Writer)
  → Duże pliki, czytanie/pisanie porcjami
  → Zawsze przez using!

JSON (System.Text.Json)
  → Standard dla web API i nowoczesnej konfiguracji
  → Serialize / Deserialize + JsonSerializerOptions + atrybuty

XML (XmlSerializer)
  → Klasa ↔ XML 1:1, wymaga bezparametrowego konstruktora
  → Atrybuty XmlElement, XmlAttribute, XmlArray, XmlIgnore

LINQ to XML (XDocument, XElement)
  → Drzewo obiektów — budowa programatyczna i zapytania LINQ
  → Świetne do dynamicznego XML i przeszukiwania

FileSystemWatcher
  → Reaguj na zmiany w katalogu (Created/Changed/Deleted/Renamed)
  → Ustawcie NotifyFilter; uwaga na duplikaty zdarzeń i zajęte pliki
```

---

## Laboratorium 3 — Persystencja i monitoring plików w OrderFlow

**Projekt:** OrderFlow — kontynuacja

**Tematy:** System.IO, serializacja JSON i XML, LINQ to XML, FileSystemWatcher

Pracujecie w tej samej solucji co na Lab 1 i Lab 2. W zadaniu 3 wykorzystacie **zdarzenia z Lab 2** — spajamy kolejne warstwy w jedną aplikację.

Sugerowana rozbudowa struktury:

```
OrderFlow.Console/
├── Models/
├── Services/
├── Persistence/          ← nowość: OrderRepository, XmlReportBuilder
├── Watchers/             ← nowość: InboxWatcher
├── data/                 ← katalog na pliki JSON/XML (dodać do .gitignore!)
└── Program.cs
```

> :warning: Dodajcie do `.gitignore`: `data/`, `inbox/`, `*.tmp` — repozytorium ma zawierać kod, nie dane testowe.

---

### Zadanie 1 — Repozytorium: zapis i odczyt zamówień w JSON i XML (5 pkt)

Zbudujcie klasę `OrderRepository`, która potrafi zapisać i wczytać kolekcję zamówień w dwóch formatach.

**Wymagania:**

- Metody asynchroniczne:
    - `Task SaveToJsonAsync(IEnumerable<Order> orders, string path)`
    - `Task<List<Order>> LoadFromJsonAsync(string path)`
    - `Task SaveToXmlAsync(IEnumerable<Order> orders, string path)`
    - `Task<List<Order>> LoadFromXmlAsync(string path)`
- Każda metoda zapisu tworzy katalog docelowy, jeśli nie istnieje (`Directory.CreateDirectory`).
- JSON ma być **sformatowany z wcięciami**, z `camelCase` i bez ucieczki polskich znaków.
- Użyjcie atrybutów:
    - w JSON: minimum jedno `[JsonPropertyName("...")]` i jedno `[JsonIgnore]`,
    - w XML: minimum jedno `[XmlAttribute]` i jedno `[XmlElement]` z customową nazwą oraz `[XmlIgnore]`.
- Wszystkie metody używają strumieni (`FileStream`/`StreamReader`) i `await using`.
- Obsłużcie przypadek **braku pliku** — `LoadFrom...Async` zwraca pustą listę zamiast rzucać wyjątkiem.

**Pokażcie w konsoli round-trip:** zapiszcie dane z `SampleData` do `data/orders.json` i `data/orders.xml`, wyczyśćcie pamięć, wczytajcie z powrotem, porównajcie liczbę i kwoty (powinny się zgadzać).

---

### Zadanie 2 — Raport XML budowany przez LINQ to XML (5 pkt)

Zbudujcie klasę `XmlReportBuilder` generującą raport zagregowany z kolekcji zamówień.

**Wymagania:**

1. Metoda `XDocument BuildReport(IEnumerable<Order> orders)` zwraca drzewo XML o strukturze:

    ```xml
    <report generated="2026-04-16T14:30:00">
      <summary totalOrders="6" totalRevenue="12345.67" />
      <byStatus>
        <status name="Completed" count="3" revenue="9000.00" />
        <status name="New" count="2" revenue="2345.67" />
        ...
      </byStatus>
      <byCustomer>
        <customer id="1" name="Jan Kowalski" isVip="true">
          <orderCount>3</orderCount>
          <totalSpent>4500.00</totalSpent>
          <orders>
            <orderRef id="101" total="1500.00" />
            <orderRef id="102" total="2000.00" />
            ...
          </orders>
        </customer>
        ...
      </byCustomer>
    </report>
    ```

2. Struktura ma być **budowana w całości przez LINQ to XML** (`XDocument`, `XElement`, `XAttribute`) — bez `XmlSerializer`, bez sklejania stringów.

3. Użyjcie LINQ (z zajęć 1) do obliczenia agregacji: `GroupBy` po statusie, `GroupBy` po kliencie, sumy kwot.

4. Metoda `Task SaveReportAsync(XDocument report, string path)` zapisuje raport do pliku.

5. Dodatkowo napiszcie metodę `Task<IEnumerable<int>> FindHighValueOrderIdsAsync(string reportPath, decimal threshold)`, która **wczytuje raport z pliku** i za pomocą zapytania LINQ to XML zwraca `Id` zamówień o kwocie powyżej progu. Nie odczytujcie zamówień z pamięci — czytajcie z pliku XML.

**Pokażcie w konsoli:** wygenerowanie raportu do `data/report.xml`, a potem wywołanie `FindHighValueOrderIdsAsync("data/report.xml", 1000m)` i wypisanie wyników.

---

### Zadanie 3 — Automatyczny import z folderu `inbox` (5 pkt)

Zbudujcie klasę `InboxWatcher`, która obserwuje katalog `inbox/` i automatycznie importuje zamówienia z pojawiających się tam plików JSON.

**Wymagania:**

1. Klasa przyjmuje w konstruktorze ścieżkę do obserwowanego katalogu i **referencję do `OrderPipeline`** (tego z Lab 2).

2. Obserwujcie tylko pliki `*.json` i reagujcie na zdarzenie `Created`.

3. Po wykryciu nowego pliku:
    - Odczekajcie 200-500 ms albo implementujcie **retry** na `IOException` (plik może być jeszcze zajęty).
    - Zdeserializujcie plik do `List<Order>` (pojedynczy plik może zawierać jedno lub wiele zamówień).
    - Dodajcie każde zamówienie do pipeline'u i uruchomcie `ProcessOrderAsync` (z Lab 2).
    - Przenieście przetworzony plik do podkatalogu `inbox/processed/`, a w razie błędu do `inbox/failed/` (wraz z plikiem `.error.txt` zawierającym opis błędu).

4. Wykorzystajcie **zdarzenia z Lab 2** — `StatusChanged` powinno się odpalać dla zamówień zaimportowanych z pliku, bo przechodzą przez ten sam pipeline.

5. Zadbajcie o thread safety — `Created` może odpalić się dla kilku plików równocześnie. Użyjcie `SemaphoreSlim` (znanego z Lab 2) do ograniczenia równoczesnego przetwarzania do np. 2 plików naraz.

6. Klasa implementuje `IDisposable` — w `Dispose()` wyłącza watcher i zwalnia zasoby.

**Pokażcie w konsoli scenariusz demo:**

- Uruchomcie program z aktywnym watcherem.
- W trakcie działania ręcznie utwórzcie w `inbox/` plik JSON z 2-3 zamówieniami (możecie użyć `OrderRepository.SaveToJsonAsync` z zadania 1 — albo wkleić ręcznie przygotowany JSON).
- W konsoli powinny pojawić się: wykrycie pliku, import, zdarzenia `StatusChanged` z pipeline'u, potwierdzenie przeniesienia do `processed/`.

> **Wskazówka:** Najprościej zademonstrować to tak, że `Program.cs` uruchamia watcher, a potem w pętli co 3 sekundy sam wrzuca testowy plik do `inbox/`. Nie musicie nic robić ręcznie w Eksploratorze.

---

### Punktacja

| Zadanie | Punkty |
|---|---|
| Wysłanie na GitHub z commitem z zajęć | **5 pkt** |
| 1. Repozytorium JSON + XML (OrderRepository) | 5 pkt |
| 2. Raport XML z LINQ to XML (XmlReportBuilder) | 5 pkt |
| 3. FileSystemWatcher (InboxWatcher) | 5 pkt |
| **Razem** | **20 pkt** |

> **Uwaga o GitHub:** Jak zawsze — aby otrzymać 5 pkt za wysłanie, musicie mieć **co najmniej 1 commit z czasów zajęć**. Zadanie można dokończyć po zajęciach, ale finalna wersja też musi być na repozytorium.

### Wskazówki

- Zadanie 1 zróbcie najpierw i dokładnie — pozostałe z niego korzystają (repozytorium używacie w zadaniu 3, a dane testowe z zadania 1 są podstawą raportu z zadania 2).
- W zadaniu 2 **nie serializujcie obiektów** — zadanie polega właśnie na tym, żeby ręcznie zbudować drzewo XML przez `XElement`/`XAttribute`. Pokazuje to kiedy LINQ to XML jest lepszy od `XmlSerializer`.
- W zadaniu 3 pamiętajcie, że `FileSystemWatcher` może odpalić `Created` **wielokrotnie** dla tego samego pliku. Dobrą praktyką jest trzymanie `HashSet<string>` już przetwarzanych plików albo użycie `SemaphoreSlim` per plik.
- Jeżeli zadanie 3 sprawia Wam trudność — zacznijcie od najprostszej wersji (wykryj plik → wypisz nazwę → skopiuj do `processed/`). Dopiero gdy to działa, dokładajcie pipeline i obsługę błędów.
- Logujcie wszystko do konsoli na bieżąco — łatwiej zrozumieć, co się dzieje, gdy widzicie kolejność zdarzeń.
- Kolejne zajęcia (Lab 4) wprowadzają Entity Framework Core — zastąpimy pliki JSON/XML bazą danych. Ale mechanizm importu z `inbox/` zostanie, więc dobrze go już teraz napiszcie czytelnie.