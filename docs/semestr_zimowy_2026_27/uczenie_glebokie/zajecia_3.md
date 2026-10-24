# Zajęcia 3

## Skalowanie cech, walidacja krzyżowa i porównanie modeli

Mamy dwa długi z poprzednich zajęć. Pierwszy: na winach z zadania 1 KNN przegrał
z drzewem decyzyjnym 72% do 94% i obiecaliśmy wyjaśnić, dlaczego. Drugi:
głębokość drzewa wybieraliśmy, patrząc na zbiór testowy — a to trochę jak
ściąganie na egzaminie.

Dziś spłacamy oba. Odpowiedź na pierwszy to **skalowanie cech**, na drugi —
**walidacja krzyżowa**. Na koniec złożymy wszystko w jeden uczciwy przepis na
porównanie kilku modeli i wybranie najlepszego. Po drodze pojawi się nowy model —
regresja logistyczna — który na następnych zajęciach okaże się pierwszym
krokiem do sieci neuronowych.

Po dzisiejszych zajęciach będziesz w stanie:

- wyjaśnić, dlaczego KNN potrzebuje cech w podobnej skali, a drzewo nie,
- przeskalować cechy `StandardScaler` bez wycieku danych ze zbioru testowego,
- połączyć skalowanie i model w jeden `Pipeline`,
- ocenić model walidacją krzyżową i odczytać średnią i odchylenie wyników,
- uczciwie porównać kilka modeli i wybrać najlepszy, nie zaglądając do testu.

!!! tip "Przykłady z zajęć w Colabie"

    Cały materiał z tej strony jest też w gotowym notatniku — z kodem do
    uruchomienia. Otwórz go i uruchamiaj komórki po kolei, czytając materiał. Żeby zachować
    swoje zmiany, zapisz kopię: **Plik → Zapisz kopię na Dysku**.

    [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/kubzal/wsb-merito/blob/main/notebooks/semestr_zimowy_2026_27/uczenie_glebokie/zajecia_3.ipynb)

!!! info "Godzinę temu na Data Science w Pythonie"

    Oglądaliśmy dane kredytowe MeritoBanku — rozkłady dochodów, kwot kredytów
    i to, czym różnią się klienci, którzy przestali spłacać. W dzisiejszym
    zadaniu ten sam zbiór trafia do modeli. Zwróć uwagę na skalę kolumn: dochód
    i kwota liczone są w tysiącach złotych, a liczba opóźnień to 0, 1, 2… To
    za chwilę będzie miało ogromne znaczenie.

---

## Plan na dziś

| Blok | Czas |
|---|---|
| Zagadka win — dlaczego KNN przegrał | 8 min |
| Skalowanie cech | 8 min |
| `Pipeline` — skalowanie bez wycieku | 6 min |
| Walidacja krzyżowa | 12 min |
| Porównanie modeli i regresja logistyczna | 11 min |
| **Zadanie 3** | **45 min** |

---

## Zagadka win

Przypomnijmy wynik z zadania 1:

```text
KNN k=5        accuracy = 0.722
DecisionTree   accuracy = 0.944
```

Żeby zrozumieć, co poszło nie tak, popatrzmy na same dane. Wczytujemy wina
i dzielimy dokładnie tak samo jak wtedy:

```python
import numpy as np
import pandas as pd
from sklearn.datasets import load_wine
from sklearn.model_selection import train_test_split

wine = load_wine()
X = pd.DataFrame(wine.data, columns=wine.feature_names)
y = wine.target

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

kolumny = ["alcohol", "malic_acid", "color_intensity", "magnesium", "proline"]
X_train[kolumny].describe().loc[["mean", "std", "min", "max"]].round(2)
```

```text
      alcohol  malic_acid  color_intensity  magnesium  proline
mean    12.98        2.37             5.06     100.44   734.89
std      0.82        1.14             2.33      14.65   302.32
min     11.03        0.89             1.74      70.00   278.00
max     14.83        5.80            13.00     162.00  1547.00
```

Widzisz problem? Zawartość alkoholu waha się między 11 a 15, a `proline`
(zawartość jednego z aminokwasów) — między 278 a 1547. Różnica skal to dwa rzędy
wielkości.

### Jak KNN mierzy podobieństwo

KNN szuka najbliższych sąsiadów, a „najbliższy" znaczy: o najmniejszej
**odległości**. Odległość to pierwiastek z sumy kwadratów różnic na wszystkich
cechach — dokładnie tak jak odległość dwóch punktów na mapie, tylko w 13
wymiarach.

Policzmy ją dla dwóch pierwszych win ze zbioru treningowego:

```python
a, b = X_train.iloc[0], X_train.iloc[1]

print((a - b).abs().sort_values(ascending=False).head(5).round(2))
print("odległość:", round(np.sqrt(((a - b) ** 2).sum()), 1))
```

```text
proline            145.00
color_intensity      8.00
malic_acid           3.83
magnesium            2.00
alcohol              1.81
dtype: float64
odległość: 145.3
```

Odległość wynosi 145.3, a sama różnica w `proline` — 145. **Pozostałe dwanaście
cech razem dorzuciło 0.3.** Intensywność koloru różni się o 8 w skali do 13, to
ogromna różnica — a dla KNN jest niewidoczna, bo ginie przy liczbach rzędu setek.

KNN „na winach" w praktyce patrzy na jedną kolumnę z trzynastu. Nic dziwnego, że
przegrał.

!!! note "Analogia"

    Wyobraź sobie, że porównujesz ludzi po wzroście zapisanym w milimetrach
    i wadze zapisanej w tonach. Dwie osoby różniące się o 5 cm wzrostu
    (50 mm) i 40 kg wagi (0.04 t) wyjdą „prawie identyczne pod względem wagi".
    Liczby zależą od jednostek, a jednostki wybrał przypadek, nie sens.

---

## Skalowanie cech

Rozwiązanie: sprowadzić wszystkie cechy do **wspólnej skali**. Najpopularniejszy
sposób to **standaryzacja** — od każdej wartości odejmujemy średnią kolumny
i dzielimy przez jej odchylenie standardowe:

```text
wartość_przeskalowana = (wartość − średnia) / odchylenie
```

Po takiej operacji każda kolumna ma średnią 0 i odchylenie 1. Wartość `1.5`
znaczy „półtora odchylenia powyżej średniej" — i znaczy to samo dla alkoholu,
co dla `proline`.

W scikit-learn robi to `StandardScaler`:

```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)   # uczy się średnich i odchyleń na treningu
X_test_s = scaler.transform(X_test)         # stosuje TE SAME liczby do testu

pd.DataFrame(X_train_s, columns=X.columns)[kolumny].describe().loc[["mean", "std"]].round(2)
```

```text
      alcohol  malic_acid  color_intensity  magnesium  proline
mean     -0.0         0.0             -0.0       -0.0     -0.0
std       1.0         1.0              1.0        1.0      1.0
```

`-0.0` to po prostu zero — bardzo mała liczba ujemna zaokrąglona do dwóch miejsc.
Teraz KNN na przeskalowanych danych:

```python
from sklearn.neighbors import KNeighborsClassifier
from sklearn.metrics import accuracy_score

knn = KNeighborsClassifier(n_neighbors=5)
knn.fit(X_train_s, y_train)
round(accuracy_score(y_test, knn.predict(X_test_s)), 3)
```

```text
0.944
```

Z 72% na 94% — jedną operacją, bez zmiany modelu. Tyle samo co drzewo.

!!! warning "`fit` tylko na treningu"

    Zwróć uwagę: `fit_transform` na treningu, ale na teście **tylko**
    `transform`. Zbiór testowy udaje dane z przyszłości — w momencie budowania
    modelu nie znamy ich średniej. Gdyby scaler policzył średnią także z testu,
    informacja z testu wyciekłaby do modelu. To ten sam **wyciek danych**, przed
    którym ostrzegaliśmy na pierwszych zajęciach, tylko w sprytniejszym
    przebraniu.

### Które modele potrzebują skalowania

| Model | Potrzebuje skalowania? | Dlaczego |
|---|---|---|
| KNN | **tak** | liczy odległości |
| regresja logistyczna | **tak** | porównuje wagi cech, trenuje się szybciej i stabilniej |
| **sieci neuronowe** | **tak, zawsze** | bez tego trening jest wolny albo w ogóle się nie udaje |
| drzewo decyzyjne | nie | zadaje pytania typu „czy `proline` > 755?" — próg działa w każdej skali |

Drzewo radziło sobie z winami od początku, bo skala dla niego nie istnieje. Ale
ostatni wiersz przed drzewem jest najważniejszy dla tego przedmiotu: **od
zajęć 4 skalowanie będzie obowiązkowym krokiem przed każdą siecią neuronową.**

!!! note "Inne sposoby skalowania"

    `MinMaxScaler` przekształca każdą kolumnę do zakresu 0–1. Przydaje się np.
    przy obrazkach (piksele 0–255 → 0–1). Dla danych tabelarycznych domyślnie
    wybieraj `StandardScaler`.

---

## `Pipeline` — skalowanie bez wycieku

Pilnowanie, gdzie `fit_transform`, a gdzie samo `transform`, jest męczące i łatwo
o pomyłkę. scikit-learn ma na to **potok** (*pipeline*) — łączy kroki
przetwarzania i model w jeden obiekt:

```python
from sklearn.pipeline import make_pipeline

model = make_pipeline(StandardScaler(), KNeighborsClassifier(n_neighbors=5))
model.fit(X_train, y_train)
round(accuracy_score(y_test, model.predict(X_test)), 3)
```

```text
0.944
```

Ten sam wynik, ale:

- `model.fit(X_train, ...)` najpierw uczy scaler na `X_train`, potem trenuje
  KNN na przeskalowanych danych,
- `model.predict(X_test)` przeskalowuje test **liczbami z treningu** i dopiero
  potem przewiduje.

Podajesz surowe dane, a pipeline sam pilnuje kolejności. Dla reszty kodu
zachowuje się jak zwykły model — ma `fit` i `predict`. Za chwilę zobaczysz, że
przy walidacji krzyżowej to nie wygoda, tylko konieczność.

---

## Walidacja krzyżowa

### Problem 1: wynik na teście to loteria

Wytrenujmy to samo drzewo na winach pięć razy. Jedyna różnica to `random_state`
w podziale — czyli które wina trafiają do testu:

```python
from sklearn.tree import DecisionTreeClassifier

for rs in [0, 1, 2, 3, 4]:
    Xa, Xb, ya, yb = train_test_split(X, y, test_size=0.2, random_state=rs)
    drzewo = DecisionTreeClassifier(random_state=42).fit(Xa, ya)
    print(rs, round(accuracy_score(yb, drzewo.predict(Xb)), 3))
```

```text
0 0.972
1 0.861
2 0.944
3 0.833
4 0.889
```

Ten sam model, te same dane, te same ustawienia — a wynik od 83% do 97%. Zbiór
testowy ma 36 win, więc jedno wino to prawie 3 punkty procentowe. Na jednej
liczbie z jednego podziału nie da się uczciwie porównać dwóch modeli.

### Problem 2: haczyk z zajęć 2

Głębokość drzewa wybieraliśmy, patrząc na wynik **na teście**. Skoro test
posłużył do wyboru, przestał być niezależną oceną.

### Rozwiązanie: k-krotna walidacja krzyżowa

Zbiór testowy **odkładamy i nie dotykamy**. Zbiór treningowy dzielimy na
5 równych części (*foldów*) i trenujemy model pięć razy — za każdym razem
inna część służy do oceny, a pozostałe cztery do treningu:

```text
           część 1   część 2   część 3   część 4   część 5
runda 1:  [ OCENA ] [trening] [trening] [trening] [trening]
runda 2:  [trening] [ OCENA ] [trening] [trening] [trening]
runda 3:  [trening] [trening] [ OCENA ] [trening] [trening]
runda 4:  [trening] [trening] [trening] [ OCENA ] [trening]
runda 5:  [trening] [trening] [trening] [trening] [ OCENA ]
```

Każdy przykład treningowy raz trafia do oceny. Dostajemy pięć wyników zamiast
jednego — ich **średnia** to ocena modelu, a **odchylenie** mówi, jak bardzo
wynik zależy od przypadku.

```python
from sklearn.model_selection import cross_val_score

wyniki = cross_val_score(DecisionTreeClassifier(random_state=42), X_train, y_train, cv=5)

print(wyniki.round(3))
print(f"średnio {wyniki.mean():.3f} ± {wyniki.std():.3f}")
```

```text
[0.931 0.931 0.893 0.929 0.893]
średnio 0.915 ± 0.018
```

`cross_val_score` sam dzieli dane, trenuje pięć modeli i zwraca pięć wyników.
Zwróć uwagę, że podajemy mu **`X_train`**, a nie całe `X` — test dalej leży
nietknięty.

!!! tip "Dlaczego `Pipeline` jest tu konieczny"

    W każdej z pięciu rund część oceniana udaje dane z przyszłości, więc scaler
    musi być uczony **tylko na czterech częściach treningowych tej rundy**. Gdy
    podasz do `cross_val_score` pipeline, dzieje się to automatycznie. Gdybyś
    przeskalował `X_train` raz, przed walidacją — mały wyciek wróciłby
    tylnymi drzwiami.

Domyślnie `cross_val_score` liczy accuracy (dla klasyfikacji) albo R² (dla
regresji). Inną metrykę wybierasz parametrem `scoring`:

| Chcę | `scoring=` |
|---|---|
| accuracy | `"accuracy"` |
| F1 | `"f1"` |
| recall | `"recall"` |
| MAE | `"neg_mean_absolute_error"` |

Dla klasyfikatorów `cross_val_score` sam pilnuje proporcji klas w każdej części
— tak jak `stratify=y` przy zwykłym podziale.

### Uczciwy dobór głębokości

Wracamy do mieszkań z zajęć 2 i tym razem wybieramy głębokość drzewa
walidacją krzyżową — **bez zaglądania do testu**. Ten sam kod generujący dane:

```python
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

Xm = mieszkania[["metraz", "pokoje", "pietro"]]
ym = mieszkania["cena"]

Xm_train, Xm_test, ym_train, ym_test = train_test_split(
    Xm, ym, test_size=0.2, random_state=42
)
```

```python
from sklearn.tree import DecisionTreeRegressor

print(f"{'głębokość':>9}  {'MAE (CV)':>8}  {'odch.':>6}")

for glebokosc in [1, 2, 3, 4, 5, 6, 8, 10, None]:
    drzewo = DecisionTreeRegressor(max_depth=glebokosc, random_state=42)
    mae = -cross_val_score(drzewo, Xm_train, ym_train, cv=5,
                           scoring="neg_mean_absolute_error")
    print(f"{str(glebokosc):>9}  {mae.mean():8.0f}  {mae.std():6.0f}")
```

```text
głębokość  MAE (CV)   odch.
        1     78870    7032
        2     48548    6734
        3     34363    6429
        4     31718    3667
        5     33073    3170
        6     35281    4386
        8     36605    2680
       10     35081    3064
     None     34730    3339
```

!!! note "Skąd minus przed `cross_val_score`?"

    scikit-learn zakłada, że wynik zawsze jest „im więcej, tym lepiej". Dla MAE
    jest odwrotnie, więc zwraca je **z minusem** (stąd `neg_` w nazwie).
    Dopisujemy minus z przodu, żeby wrócić do zwykłych złotówek.

Walidacja krzyżowa też wskazuje głębokość 4 — tym razem jednak wybór jest
uczciwy, bo test nie brał w nim udziału. Dopiero teraz, **raz**, trenujemy
wybrany model na całym zbiorze treningowym i sprawdzamy go na teście:

```python
from sklearn.metrics import mean_absolute_error

najlepsze = DecisionTreeRegressor(max_depth=4, random_state=42)
najlepsze.fit(Xm_train, ym_train)
round(mean_absolute_error(ym_test, najlepsze.predict(Xm_test)))
```

```text
26891
```

Na teście wyszło 26 891 zł, czyli lepiej niż 31 718 z walidacji. Który wynik
jest „prawdziwy"? Walidacja jest średnią z pięciu ocen, test — jedną oceną na 40
mieszkaniach. Tym razem test trafił się trochę łatwiejszy. Rozsądne oczekiwanie
dla nowych mieszkań to **ok. 30 tysięcy** pomyłki, a nie 27.

---

## Porównanie modeli

### Nowy zawodnik: regresja logistyczna

Mimo nazwy to model do **klasyfikacji**. Działa w trzech krokach:

1. Liczy **ważoną sumę cech** — dokładnie jak regresja liniowa z zajęć 1:
   `w₁·cecha₁ + w₂·cecha₂ + … + b`.
2. Przepuszcza wynik przez funkcję, która „ściska" dowolną liczbę do przedziału
   0–1. Wychodzi **prawdopodobieństwo** klasy 1.
3. Jeśli prawdopodobieństwo przekracza 0.5 — przewiduje klasę 1.

Szybki, prosty i zaskakująco często trudny do pobicia. Zapamiętaj te trzy kroki:
**na następnych zajęciach okaże się, że to dokładnie jeden sztuczny neuron.**
Jak każdy model liczący ważoną sumę cech, lubi skalowanie — dlatego używamy go
w pipeline.

### Wszyscy na start

Porównujemy pięć modeli na winach — każdy walidacją krzyżową na tym samym
zbiorze treningowym:

```python
from sklearn.dummy import DummyClassifier
from sklearn.linear_model import LogisticRegression

modele = {
    "bazowy": DummyClassifier(strategy="most_frequent"),
    "regresja logistyczna": make_pipeline(StandardScaler(), LogisticRegression()),
    "KNN k=5": make_pipeline(StandardScaler(), KNeighborsClassifier(n_neighbors=5)),
    "drzewo": DecisionTreeClassifier(max_depth=4, random_state=42),
    "KNN bez skalowania": KNeighborsClassifier(n_neighbors=5),
}

for nazwa, model in modele.items():
    wyniki = cross_val_score(model, X_train, y_train, cv=5)
    print(f"{nazwa:22} {wyniki.mean():.3f} ± {wyniki.std():.3f}")
```

```text
bazowy                 0.401 ± 0.010
regresja logistyczna   0.979 ± 0.017
KNN k=5                0.958 ± 0.026
drzewo                 0.922 ± 0.015
KNN bez skalowania     0.662 ± 0.036
```

Jak to czytać:

- **Bazowy** — punkt odniesienia. Wszystko poniżej 0.401 to porażka.
- **KNN bez skalowania kontra ze skalowaniem** — 0.662 kontra 0.958. Zagadka
  win rozwiązana, tym razem na pięciu podziałach, a nie jednym.
- **Regresja logistyczna kontra KNN** — 0.979 i 0.958. Różnica (0.021) jest
  podobna do odchyleń (0.017 i 0.026), więc to **praktycznie remis**. Na innym
  losowaniu foldów kolejność mogłaby się odwrócić.
- **Drzewo** — 0.922, wyraźnie słabsze od dwóch pierwszych: różnica jest
  większa niż odchylenia.

!!! tip "Reguła kciuka"

    Jeśli różnica między dwoma modelami jest **mniejsza niż odchylenie** ich
    wyników, traktuj to jako remis. Przy remisie wybierz model prostszy,
    szybszy albo stabilniejszy (z mniejszym odchyleniem).

Wybieramy regresję logistyczną. Ostatni krok — trening na całym zbiorze
treningowym i **jednorazowa** ocena na teście:

```python
najlepszy = make_pipeline(StandardScaler(), LogisticRegression())
najlepszy.fit(X_train, y_train)
round(accuracy_score(y_test, najlepszy.predict(X_test)), 3)
```

```text
1.0
```

Wszystkie 36 win rozpoznane poprawnie — zgodnie z oczekiwaniem z walidacji
(ok. 98%).

### Uczciwy przepis — na resztę semestru

1. **Odłóż test** — `train_test_split(..., stratify=y)`. Od teraz go nie dotykasz.
2. **Na treningu** porównaj kandydatów walidacją krzyżową — różne modele, różne
   ustawienia. Skalowanie zawsze w pipeline.
3. **Wybierz** najlepszego według średniej, patrząc też na odchylenie.
4. **Wytrenuj** go na całym zbiorze treningowym.
5. **Raz** oceń na teście i ten wynik raportuj. Jeśli po teście wracasz
   do poprawiania modelu, test znów przestaje być uczciwy.

---

## Ściągawka

| Chcę… | Kod |
|---|---|
| przeskalować cechy | `scaler = StandardScaler()`, `scaler.fit_transform(X_train)`, `scaler.transform(X_test)` |
| skalowanie + model w jednym | `make_pipeline(StandardScaler(), KNeighborsClassifier(n_neighbors=5))` |
| walidacja krzyżowa | `cross_val_score(model, X_train, y_train, cv=5)` |
| walidacja z inną metryką | `cross_val_score(..., scoring="f1")` |
| walidacja z MAE | `-cross_val_score(..., scoring="neg_mean_absolute_error")` |
| średnia i odchylenie wyników | `wyniki.mean()`, `wyniki.std()` |
| regresja logistyczna | `make_pipeline(StandardScaler(), LogisticRegression())` |
| porównać kilka modeli | słownik `{"nazwa": model, ...}` i pętla `for nazwa, model in modele.items():` |
| sprawdzić skale cech | `X_train.describe().loc[["mean", "std", "min", "max"]]` |

---

## Zadanie 3 — Komu bank powinien odmówić? (6 pkt)

**Czas:** maks. 45 minut
**Środowisko:** Google Colab
**Oddanie:** notatnik `.ipynb` na Moodle

Godzinę temu poznałeś dane kredytowe MeritoBanku. Teraz bank chce modelu, który
wskaże wnioski kredytowe wysokiego ryzyka. Zadanie: przygotować cechy, porównać
kilka modeli uczciwie i zarekomendować jeden.

Rób części po kolei i **pokaż mi wynik każdej części, gdy będzie gotowa**.

```python
import pandas as pd

URL = "https://kubzal.github.io/wsb-merito/semestr_zimowy_2026_27/data_science_w_pythonie/dane/kredyty_2026.csv"
df = pd.read_csv(URL)

cechy = ["wiek", "dochod_mies", "staz_pracy_lat", "kwota_kredytu",
         "okres_mies", "inne_kredyty", "opoznienia_12m"]
X = df[cechy]
y = df["niesplacony"]
```

Kolumnę `cel_kredytu` na razie pomijamy — jest tekstowa, a zamianę kategorii
na liczby poznamy później.

### Część 1 — skalowanie (ok. 20 min)

1. Podziel dane na treningowe i testowe — 20% na test, `random_state=42`,
   z zachowaniem proporcji klas. Wyświetl średnią i odchylenie każdej cechy
   w zbiorze treningowym.
2. Wytrenuj `KNeighborsClassifier(n_neighbors=5)` **bez skalowania** oraz ten
   sam model **w pipeline ze `StandardScaler`**. Dla obu policz na teście
   accuracy i F1. Policz też accuracy modelu bazowego
   `DummyClassifier(strategy="most_frequent")`.
3. W komórce tekstowej wyjaśnij: która cecha decydowała o odległościach w KNN bez
   skalowania i dlaczego jego accuracy wyszła **niższa niż modelu, który nic
   nie umie**?

### Część 2 — uczciwe porównanie (ok. 25 min)

1. Porównaj walidacją krzyżową (5 części, **F1**, tylko na zbiorze
   treningowym) cztery modele: bazowy, regresję logistyczną ze skalowaniem, KNN
   `k=5` ze skalowaniem i `DecisionTreeClassifier(max_depth=4, random_state=42)`.
   Zbierz wyniki w DataFrame — kolumny: `model`, `f1_srednia`, `f1_odchylenie`.
2. Wybierz model, który zarekomendujesz bankowi. W komórce tekstowej uzasadnij
   wybór: czy różnica między dwoma najlepszymi modelami jest większa niż
   odchylenie ich wyników?
3. Wytrenuj wybrany model na całym zbiorze treningowym i **raz** policz F1 na
   teście. Czy wynik zgadza się z tym, czego spodziewałeś się po walidacji?

### Wskazówki

- Średnia i odchylenie cech: `X_train.describe().loc[["mean", "std"]]` — jak
  w zagadce win.
- „Z zachowaniem proporcji klas" to `stratify=y`.
- Model bazowy ma accuracy równą odsetkowi klientów, którzy spłacają. Jeśli KNN
  wypadł gorzej, to znaczy, że jego wskazania są gorsze niż mówienie „wszyscy
  spłacą".
- W części 2 wyniki najwygodniej zbierać jak na zajęciach 2 — w listę
  słowników, a na końcu `pd.DataFrame(wyniki).round(3)`:

    ```python
    wyniki = []
    for nazwa, model in modele.items():
        f1 = cross_val_score(model, X_train, y_train, cv=5, scoring="f1")
        wyniki.append({"model": nazwa, "f1_srednia": f1.mean(), "f1_odchylenie": f1.std()})
    ```

- Model bazowy w walidacji wypisze ostrzeżenie i dostanie F1 równe 0 — znasz to
  z zajęć 2. Tak ma być.
- Nie ma jednej poprawnej odpowiedzi w punkcie 2 części 2 — oceniam
  uzasadnienie, nie wybór.

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
