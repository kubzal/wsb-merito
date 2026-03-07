# Wprowadzenie

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