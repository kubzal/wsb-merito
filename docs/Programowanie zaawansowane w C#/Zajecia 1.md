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