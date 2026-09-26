# Zajęcia 3

## Eksploracyjna analiza danych (EDA) i detekcja outlierów

Na poprzednich zajęciach przeszliśmy przez pozyskiwanie danych i ich czyszczenie. Mamy więc komplet narzędzi, żeby z surowego pliku CSV / odpowiedzi API / tabeli SQL zrobić **czysty DataFrame** — bez braków, z poprawnymi typami, z ujednoliconym tekstem.

Co dalej? Zanim zaczniemy modelować, raportować albo wyciągać wnioski biznesowe, musimy **zrozumieć dane**. Tym zajmuje się EDA — Exploratory Data Analysis. Jest to etap, na którym zadajesz zbiorowi danych pytania i sprawdzasz, czego można się po nim spodziewać:

- jakie są rozkłady poszczególnych zmiennych?
- czy są wartości skrajne (outliery)?
- jak zmienne zachowują się względem siebie?
- czy są nieoczywiste zależności, wzorce, anomalie?

Po dzisiejszych zajęciach będziesz w stanie:

- liczyć i interpretować statystyki opisowe (centralne tendencje, miary rozproszenia, percentyle),
- robić sensowne wykresy jednowymiarowe (histogram, boxplot, KDE) i interpretować, co z nich wynika,
- badać zależności między zmiennymi (scatter, korelacja, heatmapa, pairplot),
- wykrywać outliery trzema metodami (IQR, Z-score, modified Z-score / MAD),
- podjąć świadomą decyzję, co z outlierami zrobić.

---

## Dorzucamy pakiety do projektu

Dziś dochodzą dwa nowe pakiety: `seaborn` (ładniejsze i bardziej „statystyczne" wykresy nad matplotlibem) i `scipy` (do Z-score i innych funkcji statystycznych).

```bash
uv add seaborn scipy
```

Standardowy zestaw importów na dziś:

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from scipy import stats

sns.set_theme(style="whitegrid")  # ładniejsze domyślne wykresy
```

---

## Czym jest EDA?

Termin **EDA** spopularyzował statystyk **John Tukey** w latach 70. (ten sam, od którego pochodzi boxplot). Jego główna myśl: zanim założysz cokolwiek o danych — popatrz na nie. **Wykres często mówi więcej niż średnia.**

Dobry przykład — kwartet Anscombe'a. Cztery zbiory danych, które mają **identyczne** średnie, wariancje i korelację, a wyglądają zupełnie inaczej:

```python
df_anscombe = sns.load_dataset("anscombe")
sns.relplot(data=df_anscombe, x="x", y="y", col="dataset", col_wrap=2,
            kind="scatter", height=3)
plt.show()
```

Spróbuj zgadnąć, czy regresja liniowa ma sens dla każdego z nich — popatrzysz na statystyki i powiesz „tak", popatrzysz na wykres i powiesz „tylko dla jednego".

!!! note "Złota zasada EDA"
    Liczby kłamią chętniej niż wykresy. Zawsze rób oba — statystyki **i** wizualizację.

---

## Statystyki opisowe — głębiej niż `describe()`

`df.describe()` znamy z poprzednich zajęć, ale warto rozumieć, **co konkretnie zwraca** i czego nie powie.

### Miary centralnej tendencji

```python
df = pd.read_csv("https://raw.githubusercontent.com/datasciencedojo/datasets/master/titanic.csv")

df["Fare"].mean()    # średnia arytmetyczna
df["Fare"].median()  # mediana — środek po posortowaniu
df["Fare"].mode()    # moda — najczęściej występująca wartość
```

| Miara | Kiedy używać | Wada |
|---|---|---|
| Średnia (`mean`) | Rozkład symetryczny, bez outlierów | Bardzo wrażliwa na wartości skrajne |
| Mediana (`median`) | Rozkład skośny lub z outlierami | Ignoruje całkowicie wielkość ogonów |
| Moda (`mode`) | Dane kategoryczne, dyskretne | Dla ciągłych często bezużyteczna |

!!! tip "Test domowy"
    Zarobki w firmie: 4 osoby zarabiają po 4000 zł, jedna 500 000 zł (CEO). Średnia: **103 200 zł**. Mediana: **4 000 zł**. Która liczba lepiej opisuje „typową pensję"? Mediana — i dlatego w raportach o płacach zawsze podaje się medianę, nie średnią.

### Miary rozproszenia

```python
df["Fare"].std()      # odchylenie standardowe
df["Fare"].var()      # wariancja (std do kwadratu)
df["Fare"].min(), df["Fare"].max()
df["Fare"].max() - df["Fare"].min()  # range / rozstęp
```

Sama średnia nic ci nie powie bez informacji o rozrzucie. Dwa zbiory mogą mieć tę samą średnią, ale jeden zmienia się o ±1, a drugi o ±100.

### Kwartyle, percentyle, IQR

```python
df["Fare"].quantile(0.25)    # Q1 — pierwszy kwartyl (25 percentyl)
df["Fare"].quantile(0.50)    # Q2 — mediana
df["Fare"].quantile(0.75)    # Q3 — trzeci kwartyl
df["Fare"].quantile([0.1, 0.5, 0.9])  # wiele naraz

# IQR — Interquartile Range, kluczowa miara w detekcji outlierów
iqr = df["Fare"].quantile(0.75) - df["Fare"].quantile(0.25)
print(f"IQR: {iqr:.2f}")
```

**IQR (Interquartile Range)** to rozstęp między Q1 a Q3 — szerokość „środkowych 50%" obserwacji. Odporny na outliery, bo wyrzuca skrajne 25% z każdej strony.

### Skewness i kurtosis — kształt rozkładu

```python
df["Fare"].skew()      # skośność
df["Fare"].kurt()      # kurtoza (excess kurtosis)
```

- **Skewness ≈ 0** → rozkład symetryczny (np. rozkład normalny)
- **Skewness > 0** → prawostronnie skośny (długi ogon w prawo — typowo: ceny, dochody, czas oczekiwania)
- **Skewness < 0** → lewostronnie skośny (długi ogon w lewo)
- **Kurtoza > 0** → ostry pik, ciężkie ogony
- **Kurtoza < 0** → spłaszczony rozkład

!!! tip "Praktyczna intuicja"
    Jeśli widzisz w `describe()`, że **średnia jest dużo wyższa niż mediana** — masz prawostronną skośność i prawdopodobnie outliery w górnym ogonie. Mediana < średnia to klasyczny sygnał, że warto zerknąć na rozkład.

---

## Analiza pojedynczej zmiennej (univariate)

Pierwsze, co robisz po `describe()`, to rysujesz **rozkład każdej ważnej zmiennej z osobna**. To pokazuje rzeczy, których statystyki nie wyłapią — bimodalność, dziury w danych, sztuczne wartości graniczne.

### Zmienne numeryczne

#### Histogram

Klasyka — pokazuje, jak rozkłada się zmienna na przedziałach (binach).

```python
plt.figure(figsize=(10, 4))
sns.histplot(df["Age"], bins=30, kde=False)
plt.title("Rozkład wieku pasażerów Titanica")
plt.xlabel("Wiek")
plt.ylabel("Liczba pasażerów")
plt.show()
```

Liczba binów ma znaczenie — za mało zgubi szczegóły, za dużo zrobi szum. Domyślne `bins="auto"` jest często OK, ale czasem trzeba ustawić ręcznie.

#### Wykres gęstości (KDE)

KDE (Kernel Density Estimation) to „wygładzony histogram" — pokazuje rozkład bez konieczności wybierania binów.

```python
sns.histplot(df["Age"], bins=30, kde=True)  # histogram + KDE razem
plt.show()

# Sam KDE
sns.kdeplot(df["Age"], fill=True)
plt.show()
```

KDE jest super, ale uważaj — wygładzanie może „wymyślać" wartości tam, gdzie ich nie było (np. ujemny wiek). Histogram jest zawsze blisko surowych danych.

#### Boxplot — diament EDA

```python
plt.figure(figsize=(8, 3))
sns.boxplot(x=df["Fare"])
plt.title("Boxplot ceny biletu")
plt.show()
```

Boxplot pokazuje na jednym wykresie:

- **medianę** (linia w środku pudełka),
- **Q1 i Q3** (krawędzie pudełka),
- **wąsy** (zazwyczaj 1.5 × IQR od krawędzi pudełka),
- **outliery** (kropki poza wąsami).

!!! note "Jak czytać boxplot"
    - Symetryczne pudełko + krótkie wąsy → rozkład symetryczny, mało outlierów
    - Pudełko ścieśnione przy jednym końcu + długi wąs po drugiej stronie → skośność
    - Dużo kropek za wąsami → masz outliery, których nie zobaczysz w `describe()`

#### Violinplot — boxplot + KDE w jednym

```python
plt.figure(figsize=(8, 3))
sns.violinplot(x=df["Fare"])
plt.show()
```

Łączy informację o medianie/kwartylach z kształtem rozkładu. Świetny, gdy chcesz zobaczyć **bimodalność** — czego boxplot nie pokaże.

### Zmienne kategoryczne

#### Countplot — liczność kategorii

```python
sns.countplot(data=df, x="Pclass")
plt.title("Liczba pasażerów wg klasy biletu")
plt.show()
```

To samo co `df["Pclass"].value_counts()`, tylko wykresem.

#### Barplot ze średnią (lub inną agregacją)

```python
sns.barplot(data=df, x="Pclass", y="Survived")  # domyślnie średnia
plt.title("Średnia przeżywalność wg klasy")
plt.show()
```

Pionowa kreska na słupku to przedział ufności — pokazuje niepewność estymacji.

---

## Analiza zależności między zmiennymi

Pojedyncze zmienne to dopiero początek. Najciekawsze odkrycia robi się na **parach** zmiennych — czy `wiek` wpływa na `przeżywalność`, czy `cena biletu` koreluje z `klasą` itd.

### Numeryczna vs numeryczna — scatter plot

```python
tips = sns.load_dataset("tips")

sns.scatterplot(data=tips, x="total_bill", y="tip")
plt.title("Napiwek vs całkowity rachunek")
plt.show()
```

Z dodatkową informacją kategoryczną (kolor, kształt):

```python
sns.scatterplot(data=tips, x="total_bill", y="tip", hue="time", style="smoker")
plt.show()
```

#### Z linią regresji

```python
sns.regplot(data=tips, x="total_bill", y="tip")
plt.show()
```

`regplot` dorzuca dopasowaną linię prostą + przedział ufności. Szybka odpowiedź na pytanie „czy w ogóle jest tu zależność liniowa?".

### Korelacja — siła zależności liniowej

```python
df_num = df[["Age", "Fare", "Pclass", "SibSp", "Parch", "Survived"]]
df_num.corr()                  # korelacja Pearsona (domyślnie)
df_num.corr(method="spearman") # korelacja Spearmana (rangowa, odporna na outliery)
```

| Wartość | Interpretacja |
|---|---|
| 1.0 | Idealna dodatnia zależność liniowa |
| 0.7–0.9 | Silna dodatnia |
| 0.4–0.7 | Umiarkowana dodatnia |
| 0.1–0.4 | Słaba dodatnia |
| ~0 | Brak zależności liniowej |
| -0.1 do -0.4 | Słaba ujemna |
| ... | (analogicznie ujemne) |
| -1.0 | Idealna ujemna zależność liniowa |

!!! warning "Pułapka korelacji"
    Korelacja Pearsona wykrywa **tylko zależności liniowe**. Jeśli zmienne są związane parabolicznie (`y = x²`), Pearson da prawie 0, choć zależność jest oczywista. Zawsze patrz na **scatter plot**, nie tylko na liczbę.

    Klasyczne hasło: *correlation is not causation* — silna korelacja nie znaczy, że jedna zmienna wpływa na drugą. Może oba zjawiska zależą od trzeciej zmiennej, której nie widzisz.

#### Heatmapa macierzy korelacji

```python
plt.figure(figsize=(8, 6))
sns.heatmap(df_num.corr(), annot=True, cmap="coolwarm", center=0, fmt=".2f")
plt.title("Macierz korelacji")
plt.show()
```

Najczytelniejszy sposób, żeby od razu zobaczyć, które pary zmiennych są ze sobą najsilniej związane. `annot=True` dorzuca wartości w komórkach, `center=0` ustawia neutralny kolor dla zera.

### Kategoryczna vs numeryczna

#### Boxplot grupowy

```python
sns.boxplot(data=df, x="Pclass", y="Age")
plt.title("Rozkład wieku w każdej klasie biletu")
plt.show()
```

Od razu widać, że pasażerowie w 1. klasie byli średnio starsi.

#### Violinplot grupowy

```python
sns.violinplot(data=df, x="Pclass", y="Age", hue="Sex", split=True)
plt.show()
```

`split=True` pokazuje rozkład K i M po dwóch stronach jednego „skrzypka" — kompaktowe porównanie.

#### Stripplot / swarmplot

```python
sns.swarmplot(data=tips, x="day", y="total_bill")
plt.show()
```

Każdy punkt to jedna obserwacja. Świetne dla małych zbiorów, bo nie zlewa się jak boxplot.

### Kategoryczna vs kategoryczna

#### Crosstab

```python
pd.crosstab(df["Pclass"], df["Survived"])             # liczności
pd.crosstab(df["Pclass"], df["Survived"], normalize="index")  # proporcje w wierszach
```

#### Heatmapa crosstaba

```python
ct = pd.crosstab(df["Pclass"], df["Survived"], normalize="index")
sns.heatmap(ct, annot=True, cmap="YlOrRd", fmt=".2f")
plt.show()
```

### Pairplot — wszystko naraz

```python
sns.pairplot(df[["Age", "Fare", "Pclass", "Survived"]], hue="Survived")
plt.show()
```

Pairplot rysuje **wszystkie pary zmiennych** w jednej siatce — na diagonali rozkłady jednowymiarowe, poza nią scatter ploty. Świetne narzędzie do szybkiego przeglądu, ale wolne i nieczytelne przy >6 zmiennych.

---

## Detekcja outlierów

**Outlier** (wartość odstająca) to obserwacja, która znacząco różni się od pozostałych. Może być:

- **prawdziwa, ale rzadka** — najbogatszy klient, sportowiec o niebywałej formie, wyjątkowy dzień świąteczny w sprzedaży,
- **błędem** — wiek 200 lat, ujemna cena, literówka przy wpisywaniu,
- **artefaktem zbioru** — np. wartość `999` jako kod „brak danych".

!!! warning "Outlier ≠ błąd"
    Outlier to obiektywny fakt statystyczny — coś, co odstaje. Czy to **błąd**, czy **interesujący przypadek** — to już decyzja kontekstowa. **Najpierw wykryj, potem zdecyduj.**

### Metoda 1: IQR (najpopularniejsza)

Wartość jest outlierem, jeśli leży poza zakresem `[Q1 - 1.5 × IQR, Q3 + 1.5 × IQR]`. To dokładnie to, co pokazuje boxplot.

```python
def outliers_iqr(series, mnoznik=1.5):
    Q1 = series.quantile(0.25)
    Q3 = series.quantile(0.75)
    IQR = Q3 - Q1
    dolna = Q1 - mnoznik * IQR
    gorna = Q3 + mnoznik * IQR
    return (series < dolna) | (series > gorna)

maska = outliers_iqr(df["Fare"])
print(f"Liczba outlierów: {maska.sum()} z {len(df)} ({maska.mean()*100:.1f}%)")
df[maska].head()
```

Możesz też pobawić się mnożnikiem:

- **1.5** — standard, „mild outliers"
- **3.0** — bardziej konserwatywne, „extreme outliers"

**Zalety:** prosta, odporna na same outliery (bo Q1/Q3 ich nie uwzględniają), nie zakłada normalności.

**Wady:** dla rozkładów silnie skośnych może oznaczać zbyt dużo wartości jako outliery (np. dla cen biletów na Titanicu).

### Metoda 2: Z-score

Z-score mierzy, ile odchyleń standardowych obserwacja jest od średniej.

```python
z = (df["Fare"] - df["Fare"].mean()) / df["Fare"].std()
# albo z scipy:
z = stats.zscore(df["Fare"], nan_policy="omit")

maska = np.abs(z) > 3
print(f"Liczba outlierów (|z| > 3): {maska.sum()}")
df[maska].head()
```

Typowy próg to **|z| > 3** (mniej niż 0.3% obserwacji w rozkładzie normalnym).

**Zalety:** intuicyjna interpretacja, łatwa do liczenia.

**Wady:** **wrażliwa na same outliery!** Średnia i odchylenie standardowe są wciągane przez wartości skrajne, przez co realne outliery mają mniejsze z-score niż powinny (efekt „masking"). Zakłada też rozkład w miarę normalny.

### Metoda 3: Modified Z-score (oparta na MAD)

Wariant Z-score odporny na outliery — używa **mediany** i **MAD (Median Absolute Deviation)** zamiast średniej i odchylenia.

```python
def modified_zscore(series):
    mediana = series.median()
    mad = (series - mediana).abs().median()
    return 0.6745 * (series - mediana) / mad

z_mod = modified_zscore(df["Fare"])
maska = np.abs(z_mod) > 3.5
print(f"Liczba outlierów (|mod z| > 3.5): {maska.sum()}")
```

Próg **3.5** to standardowa rekomendacja (Iglewicz & Hoaglin, 1993). Stała `0.6745` to czynnik skalujący, żeby przy rozkładzie normalnym wynik był porównywalny z klasycznym Z-score.

**Zalety:** odporna na outliery, dobra dla rozkładów skośnych.

**Wady:** mniej znana, trochę mniej intuicyjna w interpretacji.

### Porównanie metod — która kiedy

| Metoda | Najlepsza dla | Słabość |
|---|---|---|
| IQR (1.5×) | Większości przypadków, szybki sanity check | Może być za agresywna dla skośnych |
| Z-score | Rozkładów ~normalnych, łatwa interpretacja | Wciągana przez ekstremalne outliery |
| Modified Z (MAD) | Skośnych i z dużymi outlierami | Mniej znana |
| Boxplot wizualny | Pierwsze spojrzenie | Subiektywny dla bordeline cases |

!!! tip "W praktyce"
    Najczęściej zaczynam od boxplota + IQR. Jeśli wynik jest dziwny (np. zbyt dużo outlierów), uciekam się do modified Z-score. Z-score używam, gdy mam pewność, że rozkład jest blisko normalnego.

### Outliery wielowymiarowe — krótko

Czasem obserwacja **w żadnej pojedynczej zmiennej nie jest outlierem**, ale **kombinacja** wartości jest nietypowa — np. człowiek mierzący 1.50 m i ważący 120 kg. Każda wartość z osobna jest w normie, ale razem to anomalia.

Do takich przypadków używa się metod modelowych:

- **Isolation Forest** (`sklearn.ensemble.IsolationForest`) — drzewa decyzyjne izolujące anomalie,
- **Local Outlier Factor** (`sklearn.neighbors.LocalOutlierFactor`) — porównuje gęstość lokalną,
- **DBSCAN** — clustering, który wprost identyfikuje punkty „nieprzypisane".

Nie będziemy ich dziś używać, ale dobrze wiedzieć, że istnieją.

---

## Co zrobić z outlierami?

Tu nie ma jednej odpowiedzi — to **decyzja biznesowa**, nie techniczna. Podstawowe opcje:

| Decyzja | Kiedy stosować |
|---|---|
| **Zostaw jak jest** | Outlier jest prawdziwą obserwacją i niesie informację (np. VIP klient, czarny łabędź w sprzedaży) |
| **Usuń** | Jasny błąd danych (wiek 200, ujemna cena) lub gdy psuje model i ich obecność jest nieistotna |
| **Zwinsoryzuj (capping)** | Chcesz ograniczyć wpływ, ale nie usuwać obserwacji — np. ucinasz wszystko > 99 percentyla |
| **Zlogarytmuj zmienną** | Rozkład jest silnie skośny i log spłaszcza ogony (typowe dla cen, dochodów) |
| **Modeluj osobno** | Outliery są ciekawe same w sobie (detekcja fraudów, oszustw) |

### Winsoryzacja (capping)

```python
gorna_granica = df["Fare"].quantile(0.99)
dolna_granica = df["Fare"].quantile(0.01)

df["Fare_capped"] = df["Fare"].clip(lower=dolna_granica, upper=gorna_granica)
```

`.clip()` ustala wartości spoza granic na granice — zachowujesz wiersz, ale „odgryzasz" mu skrajną wartość.

### Transformacja logarytmiczna

```python
df["Fare_log"] = np.log1p(df["Fare"])  # log1p = log(1+x), działa też dla zer
```

Po transformacji rozkład jest zwykle bliższy normalnemu, a outliery „wracają do stada".

```python
fig, axes = plt.subplots(1, 2, figsize=(12, 4))
sns.histplot(df["Fare"], ax=axes[0], bins=30)
axes[0].set_title("Przed: surowe Fare")
sns.histplot(df["Fare_log"], ax=axes[1], bins=30)
axes[1].set_title("Po: log(1 + Fare)")
plt.tight_layout()
plt.show()
```

!!! note "Dokumentuj decyzję"
    Jeśli usuwasz lub modyfikujesz dane — **zapisz, co i dlaczego**. Inny analityk (albo ty za pół roku) musi móc odtworzyć twój tok rozumowania. Komentarz w notebooku jest złotem.

---

## Ćwiczenie łączone — pełna EDA na Titanicu

Zróbmy razem pełen przepływ EDA na znanym już zbiorze.

### Krok 1 — wczytanie i wstępna ocena

```python
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt

sns.set_theme(style="whitegrid")

url = "https://raw.githubusercontent.com/datasciencedojo/datasets/master/titanic.csv"
df = pd.read_csv(url)

df.columns = df.columns.str.lower()
df["age"] = df.groupby("pclass")["age"].transform(lambda x: x.fillna(x.median()))
df["embarked"] = df["embarked"].fillna(df["embarked"].mode()[0])

print(df.shape)
df.describe()
```

### Krok 2 — rozkłady kluczowych zmiennych

```python
fig, axes = plt.subplots(1, 3, figsize=(15, 4))
sns.histplot(df["age"], bins=30, kde=True, ax=axes[0]); axes[0].set_title("Wiek")
sns.histplot(df["fare"], bins=30, kde=True, ax=axes[1]); axes[1].set_title("Cena biletu")
sns.countplot(data=df, x="pclass", ax=axes[2]); axes[2].set_title("Klasa biletu")
plt.tight_layout()
plt.show()

print(f"Skewness wieku: {df['age'].skew():.2f}")
print(f"Skewness fare:  {df['fare'].skew():.2f}")
```

Od razu widać, że `fare` jest mocno prawostronnie skośny (skewness ~5) — typowy kandydat na log lub winsoryzację.

### Krok 3 — zależności

```python
plt.figure(figsize=(8, 5))
sns.heatmap(
    df[["age", "fare", "pclass", "sibsp", "parch", "survived"]].corr(),
    annot=True, cmap="coolwarm", center=0, fmt=".2f"
)
plt.title("Macierz korelacji")
plt.show()
```

```python
# Przeżywalność wg klasy i płci — pivot + heatmapa
ct = df.pivot_table(index="pclass", columns="sex", values="survived", aggfunc="mean")
sns.heatmap(ct, annot=True, cmap="YlGn", fmt=".2f")
plt.title("Średnia przeżywalność: klasa × płeć")
plt.show()
```

### Krok 4 — detekcja outlierów w cenie biletu

```python
# Boxplot wizualny
plt.figure(figsize=(10, 3))
sns.boxplot(x=df["fare"])
plt.title("Boxplot cen biletów")
plt.show()

# IQR
def outliers_iqr(s, k=1.5):
    Q1, Q3 = s.quantile(0.25), s.quantile(0.75)
    IQR = Q3 - Q1
    return (s < Q1 - k * IQR) | (s > Q3 + k * IQR)

m_iqr = outliers_iqr(df["fare"])
print(f"IQR (1.5×):       {m_iqr.sum()} outlierów ({m_iqr.mean()*100:.1f}%)")

# Z-score
from scipy import stats
z = stats.zscore(df["fare"])
m_z = np.abs(z) > 3
print(f"Z-score (|z|>3):  {m_z.sum()} outlierów")

# Modified Z-score
def mod_z(s):
    med = s.median()
    mad = (s - med).abs().median()
    return 0.6745 * (s - med) / mad

m_mod = np.abs(mod_z(df["fare"])) > 3.5
print(f"Mod Z (>3.5):     {m_mod.sum()} outlierów")
```

### Krok 5 — co z nimi zrobimy?

```python
# Zobaczmy outliery z bliska
df.loc[m_iqr, ["name", "pclass", "fare", "survived"]].sort_values("fare", ascending=False).head(10)
```

Top „outliery" to pasażerowie 1. klasy z biletami za ponad 200 GBP — to nie są błędy, to prawdziwi VIP-owie. Decyzja: nie usuwamy, ale na potrzeby modelu wykonamy log-transformację.

```python
df["fare_log"] = np.log1p(df["fare"])

fig, axes = plt.subplots(1, 2, figsize=(12, 4))
sns.histplot(df["fare"], ax=axes[0], bins=30); axes[0].set_title("fare")
sns.histplot(df["fare_log"], ax=axes[1], bins=30); axes[1].set_title("log(1 + fare)")
plt.tight_layout()
plt.show()

print(f"Skewness przed: {df['fare'].skew():.2f}")
print(f"Skewness po:    {df['fare_log'].skew():.2f}")
```

Skośność spadła z ~5 do ~0.4 — rozkład jest teraz prawie symetryczny, outliery są w normie.

To pełen przepływ EDA: dane → rozkłady → zależności → outliery → decyzja.

---

## Zadanie

### Analiza rynku mieszkań w Warszawie

Twoim zadaniem jest przeprowadzenie pełnej eksploracyjnej analizy danych o ofertach sprzedaży mieszkań w Warszawie — od wstępnej oceny zbioru, przez wizualizacje, po detekcję i obsługę outlierów.

#### Generowanie danych

Wygeneruj plik z danymi — **uruchom poniższy skrypt raz**:

```python
import pandas as pd
import numpy as np

np.random.seed(42)

n = 2000
dzielnice = ["Mokotów", "Wola", "Śródmieście", "Praga-Południe", "Ursynów",
             "Bemowo", "Białołęka", "Targówek", "Bielany", "Ochota", "Wilanów"]
multiplikator_dzielnicy = {
    "Mokotów": 1.15, "Wola": 1.10, "Śródmieście": 1.40, "Praga-Południe": 0.90,
    "Ursynów": 1.00, "Bemowo": 0.95, "Białołęka": 0.85, "Targówek": 0.88,
    "Bielany": 0.95, "Ochota": 1.05, "Wilanów": 1.20
}

dzielnica = np.random.choice(dzielnice, n)
metraz = np.clip(np.random.normal(55, 22, n), 18, 180)
pokoje = np.clip(np.round(metraz / 18 + np.random.normal(0, 0.5, n)), 1, 6).astype(int)
pietro = np.random.randint(0, 12, n)
rok_budowy = np.random.choice(
    list(range(1950, 2025)),
    n,
    p=np.linspace(0.5, 2, 75) / np.linspace(0.5, 2, 75).sum()
)
ma_balkon = np.random.choice([True, False], n, p=[0.75, 0.25])
ma_miejsce_parkingowe = np.random.choice([True, False], n, p=[0.45, 0.55])
odleglosc_od_centrum = np.clip(np.random.gamma(2.5, 2.5, n), 0.5, 25)

# Cena za m² zależna od wielu czynników + szum
cena_za_m2 = (
    14000
    * np.array([multiplikator_dzielnicy[d] for d in dzielnica])
    * (1 + 0.005 * (rok_budowy - 1980))
    * (1 - 0.015 * odleglosc_od_centrum)
    * (1 + 0.05 * ma_balkon)
    * (1 + 0.08 * ma_miejsce_parkingowe)
    + np.random.normal(0, 1500, n)
)
cena = (cena_za_m2 * metraz).round(0)

df = pd.DataFrame({
    "id_oferty": range(10001, 10001 + n),
    "dzielnica": dzielnica,
    "metraz_m2": metraz.round(1),
    "liczba_pokoi": pokoje,
    "pietro": pietro,
    "rok_budowy": rok_budowy,
    "ma_balkon": ma_balkon,
    "ma_miejsce_parkingowe": ma_miejsce_parkingowe,
    "odleglosc_od_centrum_km": odleglosc_od_centrum.round(2),
    "cena_pln": cena
})

# Wstrzykujemy outliery i błędy
outlier_idx = np.random.choice(df.index, 30, replace=False)
df.loc[outlier_idx[:10], "cena_pln"] *= np.random.uniform(5, 12, 10)   # absurdalnie drogie (penthousy / błędy)
df.loc[outlier_idx[10:20], "cena_pln"] *= np.random.uniform(0.05, 0.2, 10)  # absurdalnie tanie (błędy / oszustwa)
df.loc[outlier_idx[20:25], "metraz_m2"] = np.random.uniform(300, 600, 5)   # gigantyczne metraże
df.loc[outlier_idx[25:30], "rok_budowy"] = np.random.choice([1800, 1850, 2050, 2099], 5)  # bzdurne daty

df.to_csv("mieszkania_warszawa.csv", index=False)
print(f"Wygenerowano plik 'mieszkania_warszawa.csv' — {len(df)} ofert")
```

#### Polecenia

**Część 1 — Wstępna eksploracja (2 pkt)**

1. Wczytaj plik `mieszkania_warszawa.csv` do DataFrame.
2. Wykonaj wstępną ocenę zbioru: `shape`, `info`, `describe`, `isnull().sum()`. W komentarzu lub komórce markdown opisz, **co już teraz widać podejrzanego** w statystykach opisowych (np. nielogiczne min/max).

**Część 2 — Statystyki opisowe (3 pkt)**

3. Dla kolumny `cena_pln` policz: średnią, medianę, odchylenie standardowe, skewness i kurtosis. Skomentuj — co skewness mówi o kształcie rozkładu?
4. Dla kolumny `metraz_m2` policz Q1, Q3 i IQR.
5. Pokaż, ile jest unikalnych dzielnic i ile ofert przypada na każdą (`value_counts`).

**Część 3 — Analiza pojedynczych zmiennych (3 pkt)**

6. Narysuj histogram + KDE dla `cena_pln` i `metraz_m2`. Czy widzisz skośność?
7. Narysuj boxplot dla `cena_pln` — w komentarzu opisz, co widać.
8. Narysuj countplot pokazujący liczność ofert w każdej dzielnicy (posortuj malejąco).

**Część 4 — Analiza zależności (3 pkt)**

9. Narysuj heatmapę macierzy korelacji dla wszystkich zmiennych numerycznych. Która zmienna najsilniej koreluje z ceną?
10. Narysuj scatter plot `metraz_m2` vs `cena_pln` (dodatkowo użyj `hue="dzielnica"` lub innego sensownego kolorowania).
11. Narysuj boxplot ceny za m² (`cena_pln / metraz_m2`) w podziale na dzielnice. Która dzielnica ma najwyższe ceny za m²?

**Część 5 — Detekcja outlierów (4 pkt)**

12. Wykryj outliery w kolumnie `cena_pln` **trzema metodami**: IQR (1.5×), Z-score (|z| > 3), Modified Z-score (|z_mod| > 3.5). Wypisz, ile outlierów znalazła każda metoda.
13. Wykryj outliery w kolumnie `metraz_m2` metodą IQR. Pokaż top 5 największych wartości.
14. Znajdź wiersze z bzdurnymi wartościami w `rok_budowy` (lata przed 1900 lub po 2026).

**Część 6 — Decyzja i czyszczenie (2 pkt)**

15. Usuń wiersze z nielogicznym `rok_budowy`.
16. Dla `cena_pln` — zastosuj winsoryzację (cap na 1 i 99 percentylu). Dodaj nową kolumnę `cena_pln_capped`.
17. Dodaj kolumnę `cena_pln_log` = `log1p(cena_pln)`. Porównaj skewness przed i po — narysuj dwa histogramy obok siebie.

**Część 7 — Wnioski (1 pkt)**

18. W komórce markdown wypisz **3–5 najważniejszych wniosków** z całej analizy. Czego dowiedziałeś się o rynku mieszkań w Warszawie z tego zbioru?

#### Podpowiedzi

- Dla heatmapy korelacji rzutuj `ma_balkon` i `ma_miejsce_parkingowe` na `int` — pandas nie liczy korelacji dla bool:

```python
df_corr = df.copy()
df_corr["ma_balkon"] = df_corr["ma_balkon"].astype(int)
df_corr["ma_miejsce_parkingowe"] = df_corr["ma_miejsce_parkingowe"].astype(int)
df_corr.select_dtypes(include="number").corr()
```

- Cena za m² to świetne narzędzie do porównywania ofert — kawalerka 30 m² za 500 000 zł i 80-metrowe mieszkanie za 1 200 000 zł mają inne ceny **całkowite**, ale podobne **za m²**.

- `df["cena_pln_per_m2"] = df["cena_pln"] / df["metraz_m2"]` przyda ci się w kilku miejscach.

#### Kryteria zaliczenia

- Wstępna eksploracja — **2 pkt**
- Statystyki opisowe (3 zadania) — **3 pkt**
- Analiza pojedynczych zmiennych (3 wykresy) — **3 pkt**
- Analiza zależności (3 zadania) — **3 pkt**
- Detekcja outlierów (3 metody + 2 dodatkowe) — **4 pkt**
- Decyzja i czyszczenie — **2 pkt**
- Wnioski w markdown — **1 pkt**
- Wysłanie zadania w trakcie zajęć — **2 pkt**

**TOTAL: 20 pkt**

Powodzenia! 📊
