# Zajęcia 2

## Jak ocenić model — metryki i przeuczenie

Na poprzednich zajęciach wytrenowaliśmy pierwsze modele i ocenialiśmy je jedną liczbą:
trafnością (*accuracy*) dla klasyfikacji, MAE i R² dla regresji. Dziś zobaczysz,
że jedna liczba potrafi kłamać — model z trafnością 95% może być kompletnie
bezużyteczny.

Druga połowa zajęć to temat, który zasygnalizowaliśmy przy KNN z `k=1`:
**przeuczenie**. Tym razem nie tylko je zobaczysz, ale nauczysz się je mierzyć
i dobierać model tak, żeby go uniknąć.

Po dzisiejszych zajęciach będziesz w stanie:

- wyjaśnić, dlaczego sama trafność nie wystarcza, i porównać model z modelem
  bazowym,
- odczytać macierz pomyłek i nazwać jej cztery pola,
- policzyć i zinterpretować precision, recall i F1 — i wybrać, która jest ważna
  w danym problemie,
- ocenić model regresyjny przez MAE, RMSE i R² na tle modelu bazowego,
- rozpoznać niedouczenie i przeuczenie na wykresie (tabeli) trening kontra test.

!!! info "Godzinę temu na Data Science w Pythonie"

    Czyściliśmy dane klientów siłowni — duplikaty, braki, „tak"/„TAK"/„Tak".
    W dzisiejszym zadaniu ten sam zbiór trafia do modelu. Jeśli Twój plik
    `silownia_czyste.csv` przeszedł test, jest identyczny z wersją, z której
    tu skorzystamy.

---

## Plan na dziś

| Blok | Czas |
|---|---|
| Paradoks trafności i model bazowy | 8 min |
| Macierz pomyłek | 8 min |
| Precision, recall, F1 | 10 min |
| Metryki regresji | 6 min |
| Przeuczenie i niedouczenie | 13 min |
| **Zadanie 2** | **45 min** |

---

## Paradoks trafności

Wyobraź sobie test medyczny na rzadką chorobę. Na 100 badanych osób choruje 5.
Ktoś przynosi „model", który **każdemu** mówi: jesteś zdrowy.

```python
import numpy as np
from sklearn.metrics import accuracy_score

y_true = np.array([0] * 95 + [1] * 5)     # 95 zdrowych, 5 chorych
y_pred = np.zeros(100, dtype=int)         # model: wszyscy zdrowi

accuracy_score(y_true, y_pred)
```

```text
0.95
```

95% trafności. Brzmi świetnie — a model nie wykrył **ani jednego** chorego, czyli
nie zrobił jedynej rzeczy, do której był potrzebny.

To nie jest wydumany przykład. W prawdziwych problemach klasy prawie nigdy nie są
równe: oszustw kartą jest ułamek procenta, klientów odchodzących — kilkanaście
procent, awarii maszyn — jeszcze mniej. Wszędzie tam trafność jest liczbą,
którą najłatwiej podbić, nie robiąc nic.

### Model bazowy

Dlatego zanim pochwalisz się wynikiem, sprawdź, ile osiąga model, który
**nic nie umie**. W scikit-learn jest do tego `DummyClassifier` — model, który
zawsze odpowiada najczęstszą klasą.

Pracujemy na prawdziwym zbiorze: 569 guzów piersi, 30 cech wyliczonych ze zdjęć
mikroskopowych, zadanie — rozpoznać, czy guz jest złośliwy.

```python
import pandas as pd
from sklearn.datasets import load_breast_cancer

rak = load_breast_cancer()
X = pd.DataFrame(rak.data, columns=rak.feature_names)
y = 1 - rak.target       # odwracamy: 1 = złośliwy, 0 = łagodny

print(X.shape)
pd.Series(y).value_counts()
```

```text
(569, 30)
0    357
1    212
Name: count, dtype: int64
```

!!! note "Dlaczego `1 - rak.target`?"

    W oryginalnym zbiorze `0` oznacza guz złośliwy, a `1` łagodny
    (`rak.target_names` → `['malignant' 'benign']`). Metryki, które za chwilę
    poznamy, domyślnie patrzą na klasę `1` jako na „tę, której szukamy". Szukamy
    nowotworów, więc odwracamy etykiety. To częsta, porządkowa operacja — klasa
    `1` to zawsze **to, co chcemy wykryć**.

Podział na trening i test — z jednym nowym parametrem:

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

print(y_train.mean().round(3), y_test.mean().round(3))
```

```text
0.374 0.368
```

`stratify=y` pilnuje, żeby **odsetek klas** był taki sam w zbiorze treningowym
i testowym — tu w obu ok. 37% guzów złośliwych. Bez tego losowanie mogłoby
wrzucić do testu np. same łagodne przypadki. Przy niezrównoważonych klasach
używaj `stratify` zawsze.

Teraz model bazowy i prawdziwy model obok siebie:

```python
from sklearn.dummy import DummyClassifier
from sklearn.tree import DecisionTreeClassifier

dummy = DummyClassifier(strategy="most_frequent")
dummy.fit(X_train, y_train)
print("bazowy:", round(accuracy_score(y_test, dummy.predict(X_test)), 3))

model = DecisionTreeClassifier(max_depth=3, random_state=42)
model.fit(X_train, y_train)
y_pred = model.predict(X_test)
print("drzewo:", round(accuracy_score(y_test, y_pred), 3))
```

```text
bazowy: 0.632
drzewo: 0.904
```

Model bazowy ma 63% za darmo — tyle, ile wynosi udział najczęstszej klasy.
**Dopiero to, co ponad tę liczbę, jest zasługą modelu.** Drzewo ma 90%, więc
czegoś się nauczyło. Ale czy 90% to dobry wynik dla testu na raka? Żeby to
ocenić, musimy zobaczyć, **gdzie** model się myli.

---

## Macierz pomyłek

Każda odpowiedź klasyfikatora binarnego trafia do jednego z czterech pól:

| | model: łagodny (0) | model: złośliwy (1) |
|---|---|---|
| **naprawdę łagodny (0)** | TN — prawdziwie negatywny | FP — fałszywy alarm |
| **naprawdę złośliwy (1)** | FN — przeoczenie | TP — prawdziwie pozytywny |

- **TP** (*true positive*) — model mówi „złośliwy" i ma rację.
- **TN** (*true negative*) — model mówi „łagodny" i ma rację.
- **FP** (*false positive*) — fałszywy alarm: zdrowa osoba niepotrzebnie
  idzie na biopsję.
- **FN** (*false negative*) — przeoczenie: chora osoba wraca do domu
  z informacją, że wszystko w porządku.

```python
from sklearn.metrics import confusion_matrix

confusion_matrix(y_test, y_pred)
```

```text
array([[70,  2],
       [ 9, 33]])
```

Wiersze to prawda, kolumny to odpowiedź modelu — w tej samej kolejności co
w tabeli wyżej:

- **70** łagodnych guzów model poprawnie uznał za łagodne (TN),
- **2** łagodne uznał za złośliwe — fałszywy alarm (FP),
- **9** złośliwych uznał za łagodne — przeoczenie (FN),
- **33** złośliwe wykrył poprawnie (TP).

Trafność to po prostu `(70 + 33) / 114 = 0.904`. Ale macierz mówi coś, czego
trafność nie powie: na 42 nowotwory model **przeoczył 9**. Dziewięć osób
z nowotworem usłyszałoby, że są zdrowe. Teraz 90% nie brzmi już tak dobrze.

!!! tip "Jak zapamiętać, gdzie co jest"

    Na przekątnej (lewy górny → prawy dolny) są trafienia. Wszystko poza
    przekątną to pomyłki. Dobry model ma duże liczby na przekątnej i małe poza nią.

---

## Precision, recall, F1

Z czterech pól macierzy liczy się trzy metryki, które patrzą na problem z różnych
stron.

### Precision — czy alarmy są prawdziwe?

Z wszystkich przypadków, które model **oznaczył** jako złośliwe — ile naprawdę
było złośliwych?

```text
precision = TP / (TP + FP) = 33 / (33 + 2) = 0.943
```

Kiedy model mówi „złośliwy", ma rację w 94% przypadków.

### Recall — ile przypadków wyłapaliśmy?

Z wszystkich **naprawdę** złośliwych guzów — ile model znalazł?

```text
recall = TP / (TP + FN) = 33 / (33 + 9) = 0.786
```

Model wykrył 79% nowotworów. Co piąty mu umknął.

### F1 — jedna liczba, która łączy obie

```text
F1 = 2 · precision · recall / (precision + recall)
```

F1 to średnia (harmoniczna) z precision i recall. Jest wysokie tylko wtedy, gdy
**obie** są wysokie — model, który ma precision 1.0 i recall 0.1, dostanie F1
ok. 0.18, a nie 0.55.

W kodzie:

```python
from sklearn.metrics import precision_score, recall_score, f1_score

print("precision:", round(precision_score(y_test, y_pred), 3))
print("recall:   ", round(recall_score(y_test, y_pred), 3))
print("F1:       ", round(f1_score(y_test, y_pred), 3))
```

```text
precision: 0.943
recall:    0.786
F1:        0.857
```

Albo wszystko naraz, dla obu klas:

```python
from sklearn.metrics import classification_report

print(classification_report(y_test, y_pred, target_names=["łagodny", "złośliwy"]))
```

```text
              precision    recall  f1-score   support

     łagodny       0.89      0.97      0.93        72
    złośliwy       0.94      0.79      0.86        42

    accuracy                           0.90       114
   macro avg       0.91      0.88      0.89       114
weighted avg       0.91      0.90      0.90       114
```

Interesuje nas wiersz klasy, której szukamy — `złośliwy`. `support` to liczba
prawdziwych przypadków danej klasy w zbiorze testowym.

!!! warning "Model bazowy w tych metrykach"

    `DummyClassifier`, który zawsze mówi „łagodny", ma recall **0** — nie wykrył
    niczego. Precision nie da się nawet policzyć, bo model ani razu nie powiedział
    „złośliwy" (scikit-learn wypisze wtedy ostrzeżenie i zwróci 0). To dlatego
    przy niezrównoważonych klasach patrzymy na recall i F1, a nie na trafność.

### Która metryka jest ważniejsza?

Nie ma jednej odpowiedzi — zależy, **która pomyłka jest droższa**:

| Problem | Droższa pomyłka | Patrzymy na |
|---|---|---|
| badanie przesiewowe na raka | przeoczenie chorego (FN) | **recall** |
| wykrywanie oszustw kartą | przepuszczenie oszustwa (FN) | **recall** |
| filtr spamu | ważny mail w spamie (FP) | **precision** |
| wysyłka drogiego prezentu do „najlepszych klientów" | prezent dla przypadkowej osoby (FP) | **precision** |
| nie wiadomo / obie pomyłki podobnie kosztowne | — | **F1** |

Zawsze, zanim wybierzesz metrykę, zadaj pytanie biznesowe: **co się stanie, gdy
model się pomyli w jedną stronę, a co — w drugą?** To nie jest pytanie
techniczne i model na nie nie odpowie.

---

## Metryki regresji

W regresji nie ma macierzy pomyłek — każda odpowiedź jest „trochę" chybiona.
Wracamy do mieszkań z pierwszych zajęć — ten sam kod generujący dane:

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
```

Tym razem porównujemy regresję liniową z modelem bazowym. `DummyRegressor`
zawsze przewiduje **średnią** ceny ze zbioru treningowego.

```python
from sklearn.dummy import DummyRegressor
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_absolute_error, root_mean_squared_error, r2_score

X = mieszkania[["metraz", "pokoje", "pietro"]]
y = mieszkania["cena"]

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

for nazwa, model in [("Dummy (średnia)", DummyRegressor()),
                     ("LinearRegression", LinearRegression())]:
    model.fit(X_train, y_train)
    y_pred = model.predict(X_test)
    print(f"{nazwa:18} MAE = {mean_absolute_error(y_test, y_pred):7.0f}   "
          f"RMSE = {root_mean_squared_error(y_test, y_pred):7.0f}   "
          f"R2 = {r2_score(y_test, y_pred):6.3f}")
```

```text
Dummy (średnia)    MAE =  147500   RMSE =  175000   R2 = -0.001
LinearRegression   MAE =   22545   RMSE =   27561   R2 =  0.975
```

- **MAE** — średnia pomyłka w złotówkach. Model bazowy myli się średnio
  o 147 500 zł, regresja liniowa o 22 545 zł.
- **RMSE** (*root mean squared error*) — też w złotówkach, ale **mocniej karze
  duże pomyłki**, bo błędy są podnoszone do kwadratu przed uśrednieniem. RMSE
  jest zawsze co najmniej równe MAE; jeśli jest od niego dużo większe, model od
  czasu do czasu myli się bardzo mocno.
- **R²** — teraz widać, skąd się bierze jego skala: model bazowy ma R² ≈ 0.
  R² mówi więc, **o ile model jest lepszy od zgadywania średniej**. 1.0 to ideał,
  0 to poziom modelu bazowego, a wartości ujemne oznaczają model gorszy od
  przewidywania średniej.

!!! tip "MAE czy RMSE?"

    Jeśli pomyłka o 200 tys. zł jest dla Ciebie „tylko" dwa razy gorsza od pomyłki
    o 100 tys. — MAE. Jeśli jest dużo gorsza (np. bank, który na jednej złej
    wycenie traci fortunę) — RMSE. W raportach zwykle podaje się MAE, bo łatwo je
    wytłumaczyć: „mylimy się średnio o 22 tysiące".

---

## Przeuczenie i niedouczenie

Na pierwszych zajęciach KNN z `k=1` miał 100% na treningu i dużo mniej na
teście. Teraz zrobimy z tego eksperyment.

Drzewo decyzyjne ma parametr `max_depth` — ile pytań „tak/nie" może zadać,
zanim wyda odpowiedź. Płytkie drzewo (1–2 pytania) jest bardzo proste. Drzewo
bez limitu może zadawać pytania tak długo, aż każdy przykład treningowy trafi do
osobnego liścia — czyli zapamięta dane.

Trenujemy drzewa o rosnącej głębokości na tych samych mieszkaniach i dla każdego
liczymy błąd na treningu i na teście:

```python
from sklearn.tree import DecisionTreeRegressor

print(f"{'głębokość':>9}  {'MAE trening':>11}  {'MAE test':>8}")

for glebokosc in [1, 2, 3, 4, 5, 6, 8, 10, None]:
    drzewo = DecisionTreeRegressor(max_depth=glebokosc, random_state=42)
    drzewo.fit(X_train, y_train)
    mae_train = mean_absolute_error(y_train, drzewo.predict(X_train))
    mae_test = mean_absolute_error(y_test, drzewo.predict(X_test))
    print(f"{str(glebokosc):>9}  {mae_train:11.0f}  {mae_test:8.0f}")
```

```text
głębokość  MAE trening  MAE test
        1        75882     85429
        2        41513     43389
        3        27143     31901
        4        22631     26891
        5        19143     27659
        6        12994     29362
        8         4409     39194
       10         1633     38250
     None         1327     38650
```

`None` oznacza brak limitu głębokości. Ta tabela to najważniejszy obraz dnia —
przeczytaj ją kolumnami:

- **MAE na treningu spada cały czas.** Im głębsze drzewo, tym lepiej pasuje do
  danych, które widziało — aż do niemal zera.
- **MAE na teście najpierw spada, potem rośnie.** Najlepsze jest przy głębokości
  4 (26 891 zł). Od głębokości 6 model jest coraz gorszy na nowych danych, choć
  na treningu coraz lepszy.

Z tego wynikają trzy strefy:

| Strefa | Głębokość | Trening | Test | Co się dzieje |
|---|---|---|---|---|
| **niedouczenie** (*underfitting*) | 1–2 | słabo | słabo | model zbyt prosty, nie łapie zależności |
| **w sam raz** | 3–5 | dobrze | dobrze | model uczy się reguły |
| **przeuczenie** (*overfitting*) | 6+ | świetnie | coraz gorzej | model zapamiętuje szum |

Sygnał rozpoznawczy przeuczenia: **duża i rosnąca różnica** między wynikiem na
treningu a wynikiem na teście. Drzewo bez limitu myli się na treningu o 1 327 zł,
a na teście o 38 650 zł — prawie trzydzieści razy więcej.

!!! note "Skąd się bierze szum"

    Pamiętasz `rng.normal(0, 30000, n)` w kodzie generującym mieszkania? Każda
    cena ma doklejony losowy szum — tak jak w prawdziwym życiu dwa identyczne
    mieszkania sprzedają się za różne kwoty. Głębokie drzewo próbuje odtworzyć
    również ten szum, a szumu z definicji nie da się przewidzieć. Dlatego na
    nowych danych przegrywa.

!!! warning "Haczyk, do którego wrócimy"

    Wybraliśmy głębokość 4, bo była najlepsza **na zbiorze testowym**. Ale w ten
    sposób zbiór testowy przestał być „niewidziany" — użyliśmy go do podjęcia
    decyzji. Jego wynik jest teraz odrobinę zbyt optymistyczny. Uczciwe
    rozwiązanie tego problemu — **walidację krzyżową** — poznamy na kolejnych
    zajęciach. Tam też wyjaśnimy zagadkę słabego KNN na winach z zadania 1.

---

## Ściągawka

| Chcę… | Kod |
|---|---|
| podział z zachowaniem proporcji klas | `train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)` |
| model bazowy — klasyfikacja | `DummyClassifier(strategy="most_frequent")` |
| model bazowy — regresja | `DummyRegressor()` |
| macierz pomyłek | `confusion_matrix(y_test, y_pred)` |
| precision / recall / F1 | `precision_score(...)`, `recall_score(...)`, `f1_score(...)` |
| wszystkie metryki naraz | `print(classification_report(y_test, y_pred))` |
| RMSE | `root_mean_squared_error(y_test, y_pred)` |
| drzewo o ograniczonej głębokości | `DecisionTreeClassifier(max_depth=4, random_state=42)` |
| sprawdzić przeuczenie | porównać metrykę na `X_train` i na `X_test` |
| odsetek klas | `y.value_counts(normalize=True)` |

---

## Zadanie 2 — Kto odejdzie z siłowni? (6 pkt)

**Czas:** maks. 45 minut
**Środowisko:** Google Colab
**Oddanie:** notatnik `.ipynb` na Moodle

Siłownia „FitMerito" chce przewidywać, którzy klienci zrezygnują z karnetu, żeby
zadzwonić do nich z ofertą, zanim odejdą. Dane wyczyściłeś godzinę temu na Data
Science w Pythonie — teraz budujesz na nich model.

Rób części po kolei i **pokaż mi wynik każdej części, gdy będzie gotowa**.

### Część 1 — model kontra zgadywanie (ok. 20 min)

Wczytaj dane. Żeby wszyscy mieli identyczny zbiór (i porównywalne wyniki),
korzystamy z wersji wzorcowej — jest taka sama jak Twój plik, jeśli przeszedł
test na DS w Pythonie.

```python
import pandas as pd

URL = "https://kubzal.github.io/wsb-merito/semestr_zimowy_2026_27/uczenie_glebokie/dane/silownia_czyste.csv"
df = pd.read_csv(URL)

cechy = ["wiek", "oplata_mies", "staz_mies", "wizyty_mies", "zgloszenia"]
X = df[cechy]
y = df["zrezygnowal"]
```

Kolumny tekstowe (`miasto`, `typ_karnetu`) na razie pomijamy — modele potrzebują
liczb, a zamianę kategorii na liczby poznamy później. Typ karnetu i tak jest
w dużej mierze zawarty w opłacie.

Do zrobienia:

1. Sprawdź, ilu jest klientów i **jaki odsetek** zrezygnował.
2. Podziel dane na treningowe i testowe — 20% na test, `random_state=42`,
   **z zachowaniem proporcji klas**. Sprawdź odsetek rezygnacji w obu częściach.
3. Wytrenuj model bazowy `DummyClassifier(strategy="most_frequent")` i policz na
   zbiorze testowym jego **accuracy** i **recall**.
4. Wytrenuj `DecisionTreeClassifier(max_depth=4, random_state=42)` i policz na
   zbiorze testowym: accuracy, macierz pomyłek, precision, recall i F1.
5. W komórce tekstowej opisz macierz pomyłek **słowami, w języku siłowni**:
   ilu odchodzących klientów model wyłapał, ilu przeoczył, do ilu osób
   zadzwonilibyśmy niepotrzebnie.
6. W komórce tekstowej odpowiedz: siłownia dzwoni do każdego klienta wskazanego
   przez model i proponuje mu 20% rabatu. **Co jest dla siłowni gorsze —
   przeoczony klient czy telefon (i rabat) dla kogoś, kto i tak by został?**
   Która metryka jest więc ważniejsza: precision czy recall?

### Część 2 — szukamy właściwej głębokości (ok. 25 min)

1. Wytrenuj drzewa `DecisionTreeClassifier` o głębokościach **od 1 do 15** oraz
   bez limitu (`max_depth=None`), wszystkie z `random_state=42`. Dla każdego
   policz: **accuracy na treningu**, **accuracy na teście** i **F1 na teście**.
2. Zbierz wyniki w jeden DataFrame — jeden wiersz na głębokość — i wyświetl go.
   Przykładowy początek:

    ```text
       glebokosc  acc_train  acc_test  f1_test
    0        1.0      0.765     0.764    0.000
    1        2.0        ...       ...      ...
    ```

3. Wskaż głębokość z **najwyższym F1 na teście**. Jaką ma accuracy na teście?
4. Porównaj drzewo bez limitu głębokości z modelem bazowym z części 1. Ile ma
   accuracy na treningu, a ile na teście? W komórce tekstowej napisz, co to
   oznacza.
5. W komórce tekstowej wskaż na podstawie swojej tabeli, które głębokości to
   **niedouczenie**, a które **przeuczenie**. Uzasadnij jednym zdaniem dla każdej
   strefy.
6. Dlaczego głębokość 1 ma F1 równe 0, choć jej accuracy jest niemal identyczna
   z modelem bazowym? Odpowiedz jednym zdaniem.

### Wskazówki

- „Z zachowaniem proporcji klas" to jeden parametr `train_test_split` —
  sprawdź w ściągawce.
- Odsetek rezygnacji: `y.mean()` (bo `y` to zera i jedynki) albo
  `y.value_counts(normalize=True)`.
- Przy modelu bazowym `precision_score` wypisze ostrzeżenie — to normalne,
  wyjaśnienie jest w materiale.
- W części 2 wyniki najwygodniej zbierać w listę słowników, a na końcu zrobić
  z niej tabelę:

    ```python
    wyniki = []
    for glebokosc in list(range(1, 16)) + [None]:
        # ... trenowanie i liczenie metryk ...
        wyniki.append({"glebokosc": glebokosc, "acc_train": ..., "acc_test": ..., "f1_test": ...})

    tabela = pd.DataFrame(wyniki).round(3)
    tabela
    ```

- Wiersz z najwyższym F1: `tabela.sort_values("f1_test", ascending=False).head(1)`.
- Głębokości w tabeli wyświetlą się jako `1.0`, `2.0`, …, a brak limitu jako
  `NaN` — to dlatego, że w kolumnie jest `None`. Nie przejmuj się tym.
- Wyniki pojedynczych głębokości mogą „skakać" o kilka punktów procentowych —
  to efekt niewielkiego zbioru testowego. Patrz na ogólny trend, nie na każdy
  wiersz z osobna.
- Utknąłeś? Część 2 to dokładnie pętla z sekcji „Przeuczenie i niedouczenie",
  tylko z klasyfikatorem zamiast regresora i z innymi metrykami.

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
