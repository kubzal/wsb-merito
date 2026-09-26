# Zajęcia 1

## Czym jest Machine Learning i pierwszy model

Nazwa przedmiotu brzmi groźnie, ale zaczynamy spokojnie. Zanim dojdziemy do sieci
neuronowych, musimy zrozumieć, na czym w ogóle polega uczenie maszynowe — bo sieć
neuronowa to tylko jeden z wielu modeli, a cała reszta procesu wygląda tak samo.

Cel na dziś jest konkretny: **wychodzisz z zajęć mając za sobą `model.fit()`
i `model.predict()`**, nawet jeśli nie rozumiesz jeszcze matematyki, która za tym
stoi. Zrozumienie przyjdzie w kolejnych tygodniach.

Po dzisiejszych zajęciach będziesz w stanie:

- wyjaśnić, czym uczenie maszynowe różni się od zwykłego programowania,
- rozróżnić uczenie nadzorowane od nienadzorowanego,
- wskazać w tabeli z danymi **cechy** i **zmienną celu**,
- powiedzieć, czy dany problem to klasyfikacja, czy regresja,
- wytrenować i ocenić model w scikit-learn — klasyfikacyjny i regresyjny.

!!! info "Godzinę temu na Data Science w Pythonie"

    Poznaliśmy pandas i `DataFrame`. Dziś z niego korzystamy — dane, które
    wczytujesz i filtrujesz w pandas, to dokładnie te same dane, które podajesz
    modelowi. To nie przypadek, że te przedmioty są obok siebie.

---

## Plan na dziś

| Blok | Czas |
|---|---|
| Czym jest Machine Learning | 8 min |
| Rodzaje uczenia, cechy i zmienna celu, klasyfikacja vs regresja | 12 min |
| Proces uczenia i pierwszy model — Iris | 15 min |
| Regresja w dziesięciu linijkach + po co dzielimy dane | 10 min |
| **Zadanie 1** | **45 min** |

---

## Czym jest Machine Learning

Wyobraź sobie, że masz napisać program rozpoznający spam. W klasycznym
programowaniu siadasz i wypisujesz reguły:

```python
if "wygrałeś milion" in tresc.lower():
    return "spam"
if nadawca.endswith(".ru") and zawiera_link:
    return "spam"
# ...i jeszcze dwieście takich reguł
```

Problem w tym, że spamerzy zmieniają taktykę co tydzień, reguły zaczynają się
wykluczać, a Ty spędzasz resztę życia na ich łataniu.

Machine Learning odwraca kierunek. Zamiast pisać reguły, **pokazujesz komputerowi
przykłady** — dziesięć tysięcy maili oznaczonych jako spam albo nie-spam — a on
sam znajduje reguły, które je rozróżniają.

| | Klasyczne programowanie | Machine Learning |
|---|---|---|
| Na wejściu | dane + reguły | dane + **odpowiedzi** |
| Na wyjściu | odpowiedzi | **reguły** (model) |
| Kto pisze reguły | programista | algorytm |

To jedyna naprawdę istotna różnica. Cała reszta to szczegóły techniczne.

!!! note "Kiedy ML, a kiedy zwykły `if`"

    Jeśli regułę da się łatwo zapisać — pisz `if`. Nikt nie trenuje modelu do
    sprawdzania, czy PESEL ma 11 cyfr. ML ma sens, gdy reguła istnieje, ale jest
    zbyt skomplikowana albo zbyt zmienna, żeby ją wypisać ręcznie.

---

## Uczenie nadzorowane i nienadzorowane

### Uczenie nadzorowane (supervised learning)

Mamy dane **razem z poprawnymi odpowiedziami**. Ktoś wcześniej oznaczył każdy mail
jako spam lub nie-spam, każde zdjęcie jako kot lub pies, każde mieszkanie ceną,
za którą się sprzedało.

Model uczy się na tych przykładach i ma przewidywać odpowiedź dla nowych,
niewidzianych danych. To 90% tego, co robi się w praktyce, i cały dzisiejszy dzień.

### Uczenie nienadzorowane (unsupervised learning)

Mamy dane **bez odpowiedzi**. Nikt nie powiedział, co jest czym — model ma sam
znaleźć w nich strukturę.

Typowy przykład: masz dane o 50 tysiącach klientów i chcesz wiedzieć, czy dzielą
się na jakieś naturalne grupy. Nie wiesz z góry, ile ich jest ani czym się różnią
— algorytm grupowania (klasteryzacji) ma to wykryć sam.

| | Nadzorowane | Nienadzorowane |
|---|---|---|
| Dane | z odpowiedziami | bez odpowiedzi |
| Pytanie | „jaka będzie odpowiedź dla nowego przypadku?" | „jaka jest struktura tych danych?" |
| Przykłady | klasyfikacja spamu, prognoza ceny | grupowanie klientów, wykrywanie anomalii |

---

## Cechy i zmienna celu

To najważniejsze pojęcia dnia, więc zatrzymajmy się na chwilę.

Weź tabelę z mieszkaniami:

| metraż | pokoje | piętro | **cena** |
|---|---|---|---|
| 52 | 2 | 3 | **420 000** |
| 78 | 3 | 1 | **610 000** |
| 35 | 1 | 7 | **295 000** |

- **Cechy** (ang. *features*) to kolumny, na podstawie których przewidujemy:
  metraż, pokoje, piętro. Oznaczamy je zbiorczo jako **`X`**.
- **Zmienna celu** (ang. *target*) to kolumna, którą chcemy przewidzieć: cena.
  Oznaczamy ją jako **`y`**.

Konwencja `X` (wielka litera) i `y` (mała) jest uniwersalna — zobaczysz ją
w każdej książce, każdym tutorialu i w dokumentacji scikit-learn. Wielka litera,
bo `X` to tabela (wiele kolumn), mała — bo `y` to jedna kolumna.

!!! warning "Częsty błąd na starcie"

    Zmienna celu **nie może** zostać w `X`. Jeśli zostawisz cenę wśród cech i każesz
    modelowi przewidywać cenę, dostaniesz 100% trafności i zero wartości — model
    po prostu przepisuje odpowiedź. To się nazywa **wyciek danych** (*data leakage*)
    i jest jednym z najczęstszych błędów początkujących.

---

## Klasyfikacja i regresja

To podział uczenia nadzorowanego ze względu na to, **co przewidujemy**:

| | Klasyfikacja | Regresja |
|---|---|---|
| Przewidujemy | kategorię | liczbę |
| `y` zawiera | etykiety: `spam` / `nie-spam` | wartości: `420000`, `610000` |
| Przykłady | czy klient zrezygnuje, gatunek kwiatu, cyfra na zdjęciu | cena mieszkania, temperatura jutro, sprzedaż w przyszłym miesiącu |
| Typowa metryka | trafność (accuracy) | średni błąd (MAE) |

Szybki test: **czy odpowiedź da się sensownie uśrednić?** Średnia z cen mieszkań
ma sens → regresja. Średnia ze „spam" i „nie-spam" nie ma sensu → klasyfikacja.

!!! note "Uwaga na liczby, które są kategoriami"

    Ocena w skali 1–5 wygląda jak liczba, ale zwykle traktujemy ją jako kategorię.
    Kod pocztowy to też liczba, a uśrednianie kodów pocztowych nie ma najmniejszego
    sensu. Patrz na znaczenie, nie na typ danych.

---

## Jak wygląda proces uczenia

Zawsze te same pięć kroków — niezależnie od tego, czy model to drzewo decyzyjne,
czy sieć neuronowa z miliardem parametrów:

1. **Przygotuj dane** — rozdziel `X` (cechy) i `y` (cel).
2. **Podziel** na zbiór treningowy i testowy.
3. **Trenuj** — `model.fit(X_train, y_train)`.
4. **Przewiduj** — `model.predict(X_test)`.
5. **Oceń** — porównaj przewidywania z prawdziwymi odpowiedziami.

Krok 2 wymaga wyjaśnienia, i zrobimy to zaraz po pierwszym przykładzie.

---

## Pierwszy model — rozpoznawanie irysów

Klasyczny zbiór na start: 150 kwiatów irysa z trzech gatunków. Dla każdego zmierzono
cztery wymiary płatków i działek kielicha. Zadanie: na podstawie wymiarów rozpoznać
gatunek. Czyli **klasyfikacja**.

### Krok 1 — dane

```python
import pandas as pd
from sklearn.datasets import load_iris

iris = load_iris()

X = pd.DataFrame(iris.data, columns=iris.feature_names)   # cechy
y = iris.target                                           # zmienna celu

X.head()
```

```text
   sepal length (cm)  sepal width (cm)  petal length (cm)  petal width (cm)
0                5.1               3.5                1.4               0.2
1                4.9               3.0                1.4               0.2
2                4.7               3.2                1.3               0.2
3                4.6               3.1                1.5               0.2
4                5.0               3.6                1.4               0.2
```

Zbiór jest wbudowany w scikit-learn, więc nic nie pobieramy — działa też bez
internetu.

```python
print(X.shape)               # (150, 4) — 150 kwiatów, 4 cechy
print(iris.target_names)     # ['setosa' 'versicolor' 'virginica']
print(y[:10])                # [0 0 0 0 0 0 0 0 0 0]
```

Zwróć uwagę: `y` to liczby 0, 1, 2, a nie nazwy. Modele operują na liczbach —
`iris.target_names` mówi, który numer oznacza który gatunek.

### Krok 2 — podział na trening i test

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

print(X_train.shape, X_test.shape)    # (120, 4) (30, 4)
```

- `test_size=0.2` — 20% danych odkładamy na test, 80% zostaje na trening.
- `random_state=42` — podział jest losowy, a ta liczba go „zamraża". Dzięki temu
  przy każdym uruchomieniu dostaniesz ten sam podział i te same wyniki. Bez tego
  za każdym razem miałbyś inne liczby i nie wiedziałbyś, czy zmiana wyniku to
  efekt Twojej poprawki, czy losu. Wartość 42 jest umowna — może być dowolna.

### Krok 3 — trenowanie

```python
from sklearn.neighbors import KNeighborsClassifier

model = KNeighborsClassifier(n_neighbors=5)
model.fit(X_train, y_train)
```

To wszystko. Dwie linijki i model jest wytrenowany.

Algorytm **k najbliższych sąsiadów** działa rozbrajająco prosto: żeby zaklasyfikować
nowy kwiat, znajduje 5 najbardziej podobnych kwiatów ze zbioru treningowego
i sprawdza, który gatunek występuje wśród nich najczęściej. Tyle.

### Krok 4 — przewidywanie

```python
y_pred = model.predict(X_test)

print(y_pred[:10])     # [1 0 2 1 1 0 1 2 1 1]  — co model przewidział
print(y_test[:10])     # [1 0 2 1 1 0 1 2 1 1]  — jak było naprawdę
```

### Krok 5 — ocena

Porównywanie takich list ręcznie byłoby męczące. Od tego jest metryka:

```python
from sklearn.metrics import accuracy_score

print(accuracy_score(y_test, y_pred))     # 1.0
```

**Accuracy** (trafność) to po prostu odsetek poprawnych odpowiedzi. `1.0` oznacza
100% — model trafił wszystkie 30 kwiatów ze zbioru testowego.

!!! note "Nie przyzwyczajaj się do stu procent"

    Irysy to zbiór wyjątkowo grzeczny — gatunki różnią się na tyle wyraźnie, że
    niemal każdy model sobie z nimi poradzi. Na prawdziwych danych 100% oznacza
    zwykle, że gdzieś popełniłeś błąd. W dzisiejszym zadaniu zobaczysz bardziej
    realistyczne liczby.

### Przewidywanie dla nowego kwiatu

Po to trenowaliśmy model — żeby odpowiadał na nowe przypadki:

```python
nowy = pd.DataFrame([[5.1, 3.5, 1.4, 0.2]], columns=iris.feature_names)

numer = model.predict(nowy)[0]
print(iris.target_names[numer])     # setosa
```

!!! tip "Dlaczego DataFrame, a nie zwykła lista"

    Model trenowaliśmy na DataFrame z nazwami kolumn, więc oczekuje ich też przy
    przewidywaniu. Podanie zwykłej listy `[[5.1, 3.5, 1.4, 0.2]]` zadziała, ale
    wypisze ostrzeżenie o brakujących nazwach kolumn. Łatwiej trzymać się jednego
    formatu.

---

## Po co w ogóle dzielimy dane

Wróćmy do kroku 2, bo to najważniejsza idea dnia.

Ocenianie modelu na tych samych danych, na których się uczył, to jak ocenianie
studenta na podstawie zadań, których odpowiedzi dostał wcześniej do domu. Wysoki
wynik nic nie mówi o tym, czy student cokolwiek rozumie.

Zobaczmy to na przykładzie. Ustawmy `n_neighbors=1` — model patrzy tylko na
jednego najbliższego sąsiada:

```python
model_1 = KNeighborsClassifier(n_neighbors=1)
model_1.fit(X_train, y_train)

print(accuracy_score(y_train, model_1.predict(X_train)))    # 1.0
```

100% na zbiorze treningowym — i ta liczba jest **kompletnie bezwartościowa**.
Najbliższym sąsiadem każdego punktu treningowego jest ten sam punkt, więc model
zawsze trafi. Nie nauczył się niczego, po prostu zapamiętał dane.

To zjawisko nazywa się **przeuczeniem** (*overfitting*): model świetnie radzi
sobie z danymi, które widział, i gubi się na nowych. Właśnie dlatego odkładamy
zbiór testowy — to jedyna liczba, której można ufać.

!!! tip "Zobaczysz to w zadaniu"

    Na irysach test też wyjdzie bardzo wysoko, bo zbiór jest zbyt łatwy, żeby
    pokazać problem. W dzisiejszym zadaniu pracujesz na trudniejszych danych —
    tam różnica między wynikiem treningowym a testowym będzie już dobrze widoczna.

**Zasada na resztę semestru:** wynik na zbiorze treningowym to ciekawostka. Wynik
na zbiorze testowym to ocena modelu.

---

## Regresja w dziesięciu linijkach

Klasyfikacja przewiduje kategorię. Teraz to samo, tylko przewidujemy liczbę —
i przekonasz się, że kod wygląda niemal identycznie.

Wygenerujmy sztuczne dane o mieszkaniach:

```python
import numpy as np
import pandas as pd

rng = np.random.default_rng(42)
n = 200

metraz = rng.integers(25, 120, n)
pokoje = np.clip(metraz // 25 + rng.integers(0, 2, n), 1, 5)
pietro = rng.integers(0, 11, n)
cena = 6000 * metraz + 15000 * pokoje - 2000 * pietro + rng.normal(0, 30000, n) + 100000

mieszkania = pd.DataFrame({
    "metraz": metraz,
    "pokoje": pokoje,
    "pietro": pietro,
    "cena": cena.round(-3).astype(int),
})

mieszkania.head()
```

```text
   metraz  pokoje  pietro    cena
0      33       1      10  284000
1      98       4       8  729000
2      87       3       3  653000
3      66       3      10  526000
4      66       2       5  560000
```

Dane są sztuczne i wygenerowane ze wzoru: każdy metr kwadratowy wart jest 6000 zł,
każdy pokój dokłada 15 000 zł, każde piętro odejmuje 2000 zł, plus losowy szum.
Zobaczymy, czy model to odkryje.

Te same pięć kroków co poprzednio:

```python
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_absolute_error, r2_score

X = mieszkania[["metraz", "pokoje", "pietro"]]      # cechy
y = mieszkania["cena"]                              # cel

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

model = LinearRegression()
model.fit(X_train, y_train)

y_pred = model.predict(X_test)

print("MAE:", round(mean_absolute_error(y_test, y_pred)))
print("R2 :", round(r2_score(y_test, y_pred), 3))
```

```text
MAE: 22545
R2 : 0.975
```

Zmieniły się dokładnie trzy rzeczy: inna klasa modelu, inne metryki i `y` jest
liczbą zamiast kategorii. Szkielet jest ten sam — i taki zostanie do końca
semestru, również gdy zaczniemy używać sieci neuronowych.

Jak czytać te dwie metryki:

- **MAE** (*mean absolute error*) — średnia pomyłka wyrażona w jednostkach celu.
  Tutaj: model myli się średnio o **22 545 zł** przy cenie mieszkania. Im mniej,
  tym lepiej.
- **R²** — jaka część zmienności ceny jest wyjaśniona przez model. `1.0` to ideał,
  `0.0` to model nie lepszy od zgadywania średniej. Tutaj `0.975`, czyli bardzo
  dobrze — ale to sztuczne dane, na prawdziwych bywa znacznie gorzej.

Najciekawsze jest to, czego model się nauczył:

```python
pd.Series(model.coef_.round(0), index=X.columns)
```

```text
metraz     5980.0
pokoje    15754.0
pietro    -1965.0
dtype: float64
```

Model odtworzył wzór, którym wygenerowaliśmy dane — 6000 za metr, 15 000 za pokój,
−2000 za piętro — mimo że nikt mu go nie podał. **To właśnie znaczy, że algorytm
znalazł reguły sam.**

---

## Ściągawka

| Chcę… | Kod |
|---|---|
| wczytać wbudowany zbiór | `from sklearn.datasets import load_iris` |
| podzielić dane | `train_test_split(X, y, test_size=0.2, random_state=42)` |
| model klasyfikacyjny | `KNeighborsClassifier(n_neighbors=5)` |
| inny klasyfikator | `DecisionTreeClassifier(random_state=42)` |
| model regresyjny | `LinearRegression()` |
| wytrenować | `model.fit(X_train, y_train)` |
| przewidzieć | `model.predict(X_test)` |
| ocenić klasyfikację | `accuracy_score(y_test, y_pred)` |
| ocenić regresję | `mean_absolute_error(...)`, `r2_score(...)` |

---

## Zadanie 1 — Wina i mieszkania (6 pkt)

**Czas:** maks. 45 minut
**Środowisko:** Google Colab
**Oddanie:** notatnik `.ipynb` na Moodle

Dwie części — jedna klasyfikacja, jedna regresja. Rób po kolei i **pokaż mi wynik
każdej części, gdy będzie gotowa**.

### Część 1 — klasyfikacja win (ok. 20 min)

Zbiór `load_wine` zawiera wyniki analizy chemicznej 178 win pochodzących od trzech
różnych producentów. Zadanie: rozpoznać producenta na podstawie składu chemicznego.

```python
import pandas as pd
from sklearn.datasets import load_wine

wine = load_wine()
X = pd.DataFrame(wine.data, columns=wine.feature_names)
y = wine.target
```

Do zrobienia:

1. Sprawdź rozmiar `X` oraz nazwy klas (`wine.target_names`). Wyświetl pierwsze
   5 wierszy.
2. Podziel dane na treningowe i testowe — 20% na test, `random_state=42`.
3. Wytrenuj **trzy** modele i policz accuracy każdego na zbiorze testowym:
    - `KNeighborsClassifier(n_neighbors=1)`
    - `KNeighborsClassifier(n_neighbors=5)`
    - `DecisionTreeClassifier(random_state=42)`
4. Wypisz wyniki w czytelnej formie, na przykład tak:

    ```text
    KNN k=1        accuracy = 0.778
    KNN k=5        accuracy = 0.722
    DecisionTree   accuracy = 0.944
    ```

5. Dla modelu `KNN k=1` policz accuracy **także na zbiorze treningowym**
   i porównaj z testowym. W komórce tekstowej napisz jednym zdaniem, co to
   porównanie mówi o modelu.
6. W komórce tekstowej odpowiedz: **który model wypadł najlepiej?**

### Część 2 — regresja cen mieszkań (ok. 25 min)

Uruchom tę komórkę, żeby wygenerować dane:

```python
import numpy as np
import pandas as pd

rng = np.random.default_rng(7)
n = 300

metraz = rng.integers(20, 150, n)
pokoje = np.minimum(rng.integers(1, 6, n), np.maximum(metraz // 18, 1))
pietro = rng.integers(0, 12, n)
wiek_budynku = rng.integers(0, 60, n)

cena = (7500 * metraz + 12000 * pokoje - 3000 * pietro
        - 1500 * wiek_budynku + rng.normal(0, 25000, n) + 120000)

mieszkania = pd.DataFrame({
    "metraz": metraz,
    "pokoje": pokoje,
    "pietro": pietro,
    "wiek_budynku": wiek_budynku,
    "cena": cena.round(-3).astype(int),
})

mieszkania.head()
```

Do zrobienia:

1. Sprawdź rozmiar zbioru i wyświetl statystyki opisowe (`describe()` — znasz je
   z poprzednich zajęć).
2. Rozdziel dane na `X` (cztery cechy) i `y` (cena). **Uważaj, żeby cena nie
   została w `X`.**
3. Podziel na trening i test — 20% na test, `random_state=42`.
4. Wytrenuj `LinearRegression`.
5. Policz na zbiorze testowym **MAE** oraz **R²** i wypisz je z opisem.
6. Wyświetl współczynniki modelu (`model.coef_`) razem z nazwami cech i porównaj
   je z liczbami we wzorze generującym dane (7500, 12000, −3000, −1500).
   W komórce tekstowej napisz, **które cechy model odtworzył najdokładniej,
   a która wypadła najgorzej**.
7. Przewidź cenę mieszkania: **65 m², 3 pokoje, 2. piętro, budynek 15-letni**.
   Wynik wypisz zaokrąglony, z opisem przez f-string.

### Wskazówki

- Oba zadania to ten sam szkielet pięciu kroków z materiału — podmieniasz tylko
  dane i klasę modelu.
- Trzy modele w części 1 najwygodniej porównać pętlą po liście par
  `(nazwa, model)`, ale trzy osobne komórki też są w porządku.
- Przy przewidywaniu dla jednego mieszkania podaj DataFrame z tymi samymi nazwami
  kolumn, na których model był trenowany — inaczej dostaniesz ostrzeżenie.
- Współczynniki z nazwami: `pd.Series(model.coef_.round(0), index=X.columns)`.
- Jeśli w części 1 KNN wypadnie wyraźnie gorzej od drzewa — tak ma być. Powód
  poznamy na kolejnych zajęciach.
- W punkcie 6 części 2 nie wszystkie współczynniki trafią równie blisko. Zastanów
  się, która cecha najsłabiej wpływa na cenę — i czy to przypadek, że akurat ona
  została oszacowana najgorzej.

### Punktacja (6 pkt)

| Punkty | Za co |
|---|---|
| **6 pkt** | obie części zrobione **na zajęciach** i pokazane prowadzącemu |
| **4 pkt** | jedna z dwóch części zrobiona na zajęciach |
| **2 pkt** | zadanie dokończone po zajęciach i wysłane na Moodle **w terminie** |
| **0 pkt** | brak rozwiązania albo wysłane po terminie |

Termin przesłania na Moodle: **dzień przed kolejnymi zajęciami**.

!!! danger "Serio, róbcie to na zajęciach"

    Te same 45 minut pracy jest warte 6 pkt tu i teraz albo 2 pkt w domu. Przez
    cały semestr robi to różnicę 32 punktów, czyli całej oceny w górę lub w dół.
