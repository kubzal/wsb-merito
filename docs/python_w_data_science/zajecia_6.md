# Zajęcia 6

## Uczenie nienadzorowane — klasteryzacja danych bez etykiet

> **Poprzednie zajęcia:** zbudowaliśmy regresję (przewidywanie liczby) i 6 klasyfikatorów (przewidywanie kategorii). We wszystkich tych przypadkach mieliśmy **etykiety** — dane mówiły nam, jaka jest poprawna odpowiedź (`y`). To było **uczenie nadzorowane**.
> Dziś robimy ostatni krok i wchodzimy do drugiego świata: **uczenia nienadzorowanego**. Tu **nie ma `y`**. Dostajemy tylko cechy `X` i sami szukamy w nich struktury — naturalnych grup, które same się wyłaniają z danych. Algorytm nikt nie mówi „to jest grupa A, to grupa B" — on musi to odkryć sam.

> **Uwaga:** to są zajęcia w skróconym formacie (2h). Mniej teorii, jedno zadanie, ale przechodzimy przez trzy najważniejsze algorytmy klasteryzacji.

Po dzisiejszych zajęciach będziesz w stanie:

- wyjaśnić różnicę między uczeniem **nadzorowanym** a **nienadzorowanym**,
- wytrenować i zinterpretować trzy algorytmy klastrowania: **K-Means**, **klasteryzację hierarchiczną** i **DBSCAN**,
- dobrać liczbę klastrów metodą **łokcia** i współczynnikiem **silhouette**,
- rozumieć, czym DBSCAN różni się od K-Means (kształty klastrów, szum, brak `k`),
- wiedzieć, **kiedy którego algorytmu użyć** — i czym każdy z nich płaci.

> **Środowisko:** Google Colab. Wszystkie biblioteki (`numpy`, `pandas`, `matplotlib`, `scikit-learn`) są tam już zainstalowane.

---

## 1. Nadzorowane vs nienadzorowane — co się zmienia?

|  | Uczenie nadzorowane (zajęcia 4–5) | Uczenie nienadzorowane (dziś) |
|---|---|---|
| **Dane wejściowe** | cechy `X` **+ etykiety `y`** | **tylko cechy `X`** |
| **Cel** | nauczyć się mapowania `X → y` | znaleźć **ukrytą strukturę** w `X` |
| **Przykład** | „czy ten guz jest złośliwy?" | „na ile naturalnych grup dzielą się ci klienci?" |
| **Ewaluacja** | proste — porównujemy z prawdą (`y_test`) | trudne — nie ma „prawidłowej odpowiedzi" |
| **Typowe zadania** | regresja, klasyfikacja | **klasteryzacja**, redukcja wymiarów, wykrywanie anomalii |

Najważniejsza zmiana mentalna: **nie ma `train_test_split`, nie ma `y_test`, nie ma accuracy**. Skoro nikt nie powiedział nam, jaki jest poprawny podział, to nie mamy z czym porównać wyniku. Jakość klasteryzacji oceniamy **pośrednio** — albo metrykami wewnętrznymi (jak zwarte i odseparowane są grupy), albo zdrowym rozsądkiem („czy te grupy mają sens biznesowy?").

### Po co to komu? Realne zastosowania

- **Segmentacja klientów** — podziel użytkowników na grupy o podobnym zachowaniu (bez z góry zdefiniowanych segmentów).
- **Wykrywanie anomalii** — punkty, które nie pasują do żadnej grupy, to potencjalne fraudy / awarie.
- **Kompresja i eksploracja** — zanim zbudujesz model nadzorowany, klasteryzacja pokaże Ci, jak w ogóle wyglądają Twoje dane.

---

## 2. Dane do przykładów

Żeby **zobaczyć** klastry na oczy, pracujemy w 2D na danych syntetycznych — `make_blobs` generuje gotowe „kleksy", a `make_moons` dwa zachodzące na siebie półksiężyce (to drugie świetnie pokaże różnicę między algorytmami).

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.datasets import make_blobs

# 4 naturalne grupy punktów w 2D
X, y_prawda = make_blobs(
    n_samples=300, centers=4, cluster_std=0.70, random_state=42
)

plt.figure(figsize=(7, 6))
plt.scatter(X[:, 0], X[:, 1], s=30, alpha=0.7)
plt.title("Dane bez etykiet — ile tu widzisz grup?")
plt.xlabel("cecha 1"); plt.ylabel("cecha 2")
plt.show()
```

`make_blobs` zwraca też prawdziwe etykiety (`y_prawda`), ale **udajemy, że ich nie mamy** — używamy ich tylko na końcu, żeby sprawdzić, czy algorytm „zgadł". W prawdziwym zadaniu nienadzorowanym ich po prostu nie ma.

> **Skalowanie — tak jak na zajęciach 5.** Wszystkie trzy dzisiejsze algorytmy liczą **odległości** między punktami, więc cechy o różnych skalach trzeba **standaryzować** (`StandardScaler`). `make_blobs` daje już ładne dane, ale na prawdziwym zbiorze skalowanie jest **obowiązkowe** — inaczej cecha o większych liczbach zdominuje odległość (dokładnie ten sam problem, co przy KNN).

---

## 3. Algorytm 1 — K-Means

Najpopularniejszy algorytm klastrowania. Pomysł jest prosty: **„podziel dane na `k` grup tak, żeby punkty w każdej grupie były jak najbliżej swojego środka"**.

### Jak działa — pętla w 4 krokach

```
1. Rozrzuć losowo k środków (centroidów).
2. Przypisz każdy punkt do NAJBLIŻSZEGO środka.
3. Przesuń każdy środek do ŚREDNIEJ jego punktów.
4. Powtarzaj 2–3, aż środki przestaną się ruszać.
```

```
   krok 1: losowe środki      krok 4: środki ustabilizowane
        ●    ✦                       ●●●    ✦
      ●   ●       ▲                 ●●●●    
   ✦    ●     ▲  ▲              ✦ otoczony swoimi punktami
        ●        ▲                       ▲▲▲ ✦
   (✦ = centroid)                        ▲▲▲
```

Kluczowy hiperparametr to **`k` — liczbę klastrów podajesz z góry**. To zarazem największa wada K-Means: musisz wiedzieć (albo zgadnąć), ile grup szukasz.

```python
from sklearn.cluster import KMeans

kmeans = KMeans(n_clusters=4, random_state=42, n_init=10)
etykiety = kmeans.fit_predict(X)   # przypisanie każdego punktu do klastra (0,1,2,3)

print(kmeans.cluster_centers_)     # współrzędne 4 środków
```

```python
plt.scatter(X[:, 0], X[:, 1], c=etykiety, cmap="viridis", s=30, alpha=0.7)
plt.scatter(kmeans.cluster_centers_[:, 0], kmeans.cluster_centers_[:, 1],
            c="red", marker="X", s=250, edgecolor="black", label="centroidy")
plt.legend(); plt.title("K-Means, k=4"); plt.show()
```

> **`n_init=10`** — K-Means startuje od losowych środków i może utknąć w kiepskim podziale. `n_init=10` uruchamia algorytm 10 razy z różnych startów i wybiera najlepszy wynik. Zostaw to ustawienie.

**Zalety:** szybki, prosty, skaluje się na duże zbiory. **Wady:** musisz podać `k`, zakłada klastry **okrągłe i podobnej wielkości**, źle radzi sobie z dziwnymi kształtami (zaraz zobaczymy), wrażliwy na outliery.

---

## 4. Ile klastrów? Metoda łokcia i silhouette

Skoro K-Means wymaga `k`, a w prawdziwym zadaniu nie znamy prawdy — jak je dobrać? Dwa narzędzia.

### Metoda łokcia (elbow method)

Dla każdego `k` liczymy **inercję** (`inertia_`) — sumę kwadratów odległości punktów od ich środków. Im więcej klastrów, tym inercja mniejsza (przy `k` = liczbie punktów spada do zera). Szukamy **„łokcia"** — punktu, w którym dokładanie kolejnych klastrów przestaje istotnie poprawiać wynik.

```python
inercje = []
zakres_k = range(1, 10)
for k in zakres_k:
    km = KMeans(n_clusters=k, random_state=42, n_init=10).fit(X)
    inercje.append(km.inertia_)

plt.plot(zakres_k, inercje, marker="o")
plt.xlabel("liczba klastrów k"); plt.ylabel("inercja")
plt.title("Metoda łokcia"); plt.show()
```

Na wykresie szukasz miejsca, gdzie krzywa „łamie się" jak łokieć — dla naszych danych będzie to `k=4`.

### Współczynnik silhouette

Metryka od **−1 do 1**, która mówi, jak dobrze każdy punkt pasuje do swojego klastra w porównaniu z sąsiednim. Bierze pod uwagę dwie rzeczy: jak blisko ma do **swojej** grupy i jak daleko do **najbliższej obcej**.

- **bliski 1** → punkt jest głęboko w swoim klastrze (świetnie),
- **bliski 0** → punkt leży na granicy dwóch klastrów,
- **ujemny** → punkt prawdopodobnie wpadł do złego klastra.

```python
from sklearn.metrics import silhouette_score

for k in range(2, 8):
    et = KMeans(n_clusters=k, random_state=42, n_init=10).fit_predict(X)
    print(f"k={k}:  silhouette = {silhouette_score(X, et):.3f}")
```

Wybierasz `k` o **najwyższym** silhouette. W przeciwieństwie do łokcia (który czyta się „na oko") silhouette daje **konkretną liczbę** — dlatego często wygrywa jako kryterium.

> **Łokieć czy silhouette?** Łokieć jest szybki i intuicyjny, ale bywa niejednoznaczny (czasem nie ma wyraźnego załamania). Silhouette jest bardziej obiektywny. W praktyce patrzy się na **oba** — i na koniec na zdrowy rozsądek.

---

## 5. Algorytm 2 — Klasteryzacja hierarchiczna

Zamiast od razu dzielić dane na `k` grup, **buduj hierarchię**: zacznij od tego, że każdy punkt to osobny klaster, a potem **łącz po kolei najbliższe** klastry, aż zostanie jeden. To podejście **aglomeracyjne** (od dołu do góry).

```
punkty:    a   b   c   d   e
           │   │   │   │   │
           └─┬─┘   │   └─┬─┘     ← łączymy najbliższe pary
             │     │     │
             └──┬──┘     │       ← potem najbliższe grupy
                │        │
                └────┬───┘
                     │           ← aż wszystko w jednym
```

Efektem jest **dendrogram** — drzewo pokazujące, w jakiej kolejności i na jakim „dystansie" łączyły się grupy. Tnąc dendrogram poziomą linią na wybranej wysokości, dostajesz dowolną liczbę klastrów.

```python
from scipy.cluster.hierarchy import dendrogram, linkage

Z = linkage(X, method="ward")   # ward minimalizuje wariancję wewnątrz klastrów

plt.figure(figsize=(12, 5))
dendrogram(Z, truncate_mode="lastp", p=20)
plt.title("Dendrogram"); plt.xlabel("punkty"); plt.ylabel("dystans")
plt.show()
```

Sam podział na klastry robi `AgglomerativeClustering`:

```python
from sklearn.cluster import AgglomerativeClustering

hc = AgglomerativeClustering(n_clusters=4, linkage="ward")
etykiety_hc = hc.fit_predict(X)
```

> **`linkage`** — sposób mierzenia odległości między **grupami** punktów: `ward` (minimalizuje wariancję, domyślny wybór), `complete` (najdalsze punkty), `average` (średnia), `single` (najbliższe punkty). `ward` jest najbezpieczniejszym startem.

**Zalety:** **nie musisz podawać `k` z góry** (decydujesz, oglądając dendrogram), daje czytelną hierarchię, działa z różnymi metrykami odległości. **Wady:** wolny na dużych zbiorach (porównuje wszystko ze wszystkim — `O(n²)` i gorzej), wynik zależy od wyboru `linkage`.

---

## 6. Algorytm 3 — DBSCAN

K-Means i hierarchiczna mają wspólny problem: zakładają, że klastry są w miarę **okrągłe**, i **każdy** punkt wpychają do jakiejś grupy. DBSCAN podchodzi zupełnie inaczej: **klaster to gęsty obszar punktów otoczony pustką**.

### Intuicja — gęstość zamiast odległości od środka

DBSCAN ma dwa parametry:

- **`eps`** — promień, w którym szukamy sąsiadów (jak „blisko" znaczy blisko),
- **`min_samples`** — ile punktów musi być w tym promieniu, żeby uznać obszar za gęsty.

Punkt, który ma wokół siebie (w promieniu `eps`) co najmniej `min_samples` sąsiadów, to **rdzeń** klastra. Algorytm „rozlewa się" od rdzeni, dołączając gęsto połączone punkty. Punkty, które do żadnego gęstego obszaru nie pasują, dostają etykietę **`-1` — to szum (outliery)**.

```
   ●●●●           ▲▲▲▲
   ●●●●●    ✱     ▲▲▲▲▲        ✱ = szum (etykieta -1)
   ●●●●           ▲▲▲▲
  klaster 0       klaster 1
```

To jest **dwie supermoce DBSCAN naraz**: sam wykrywa **outliery** i radzi sobie z **dowolnymi kształtami** klastrów.

```python
from sklearn.cluster import DBSCAN

db = DBSCAN(eps=0.5, min_samples=5)
etykiety_db = db.fit_predict(X)

print("Znalezione klastry:", set(etykiety_db))   # np. {0, 1, 2, 3, -1}
n_szum = list(etykiety_db).count(-1)
print(f"Punktów uznanych za szum: {n_szum}")
```

> **`-1` to nie błąd!** Etykieta `-1` oznacza świadomą decyzję algorytmu: „ten punkt nie należy do żadnej grupy". Przy rysowaniu warto pokolorować szum osobno (np. na czarno), żeby było widać, co DBSCAN odrzucił.

### Kiedy DBSCAN wygrywa — test półksiężyców

To jest moment, w którym DBSCAN miażdży K-Means. Dwa zachodzące półksiężyce — K-Means przetnie je linią na pół (bo szuka okrągłych grup), a DBSCAN poprawnie wykryje dwa kształty:

```python
from sklearn.datasets import make_moons

X_moons, _ = make_moons(n_samples=300, noise=0.06, random_state=42)

km_lab = KMeans(n_clusters=2, random_state=42, n_init=10).fit_predict(X_moons)
db_lab = DBSCAN(eps=0.2, min_samples=5).fit_predict(X_moons)

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(13, 5))
ax1.scatter(X_moons[:, 0], X_moons[:, 1], c=km_lab, cmap="viridis", s=20)
ax1.set_title("K-Means — tnie półksiężyce na pół ❌")
ax2.scatter(X_moons[:, 0], X_moons[:, 1], c=db_lab, cmap="viridis", s=20)
ax2.set_title("DBSCAN — łapie kształty ✅")
plt.show()
```

**Zalety:** nie musisz podawać `k`, wykrywa **dowolne kształty**, automatycznie znajduje **outliery**, odporny na szum. **Wady:** czuły na dobór `eps` i `min_samples` (zła wartość = jeden wielki klaster albo same szumy), słabo radzi sobie z klastrami o **różnej gęstości**, gorszy przy wielu cechach.

---

## 7. Porównanie — kiedy czego użyć?

| Cecha | K-Means | Hierarchiczna | DBSCAN |
|---|---|---|---|
| **Trzeba podać `k`?** | tak | nie (czytasz z dendrogramu) | nie |
| **Kształty klastrów** | tylko okrągłe | dość elastyczne | **dowolne** |
| **Wykrywa outliery?** | nie | nie | **tak** (etykieta `-1`) |
| **Szybkość** | bardzo szybki | wolny (`O(n²)`) | średni |
| **Duże zbiory** | ⭐⭐⭐ tak | ⭐ słabo | ⭐⭐ ok |
| **Klastry różnej gęstości** | ok | ok | słabo |
| **Główny parametr** | `k` | `linkage`, próg cięcia | `eps`, `min_samples` |

**Reguła kciuka:**

- duży zbiór, spodziewasz się okrągłych grup, znasz mniej więcej `k` → **K-Means**,
- chcesz zobaczyć **hierarchię** podobieństw albo nie wiesz, ile grup → **hierarchiczna**,
- dziwne kształty, dużo szumu / outlierów, nie chcesz podawać `k` → **DBSCAN**.

Tak jak na zajęciach 5 z klasyfikacją — **nie ma jednego najlepszego algorytmu**. Zaczynaj od K-Means jako baseline (jest najszybszy i najprostszy), a po dziwne kształty i outliery sięgaj po DBSCAN.

---

## Łączymy wszystko — trzy algorytmy obok siebie

Spięjmy całość: trenujemy wszystkie trzy na tych samych danych i porównujemy wizualnie + metryką silhouette.

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.datasets import make_blobs
from sklearn.cluster import KMeans, AgglomerativeClustering, DBSCAN
from sklearn.metrics import silhouette_score

X, _ = make_blobs(n_samples=300, centers=4, cluster_std=0.70, random_state=42)

modele = {
    "K-Means":        KMeans(n_clusters=4, random_state=42, n_init=10),
    "Hierarchiczna":  AgglomerativeClustering(n_clusters=4, linkage="ward"),
    "DBSCAN":         DBSCAN(eps=0.6, min_samples=5),
}

fig, axes = plt.subplots(1, 3, figsize=(16, 5))
for ax, (nazwa, model) in zip(axes, modele.items()):
    et = model.fit_predict(X)
    # silhouette wymaga min. 2 klastrów i nie liczy się dla samego szumu
    if len(set(et)) > 1:
        sil = silhouette_score(X, et)
        tytul = f"{nazwa}  (silhouette={sil:.3f})"
    else:
        tytul = nazwa
    ax.scatter(X[:, 0], X[:, 1], c=et, cmap="viridis", s=25)
    ax.set_title(tytul)
plt.tight_layout(); plt.show()
```

Jeden przebieg, trzy podejścia, jedna metryka do porównania. Dokładnie tak wygląda pierwszy etap eksploracji nienadzorowanej — sprawdzasz kilka algorytmów i patrzysz, który układ klastrów ma sens.

---

## Praca projektowa

### Zadanie 6 — Segmentacja klientów centrum handlowego (10 pkt)

> **Uwaga:** przykłady liczyliśmy na danych syntetycznych (`make_blobs`). **Zadanie robisz na prawdziwym zbiorze — Mall Customers** (dane klientów centrum handlowego: wiek, dochód roczny, „spending score"). Celem jest **podzielić klientów na segmenty** — bez żadnych etykiet, bo nikt ich nie nadał. To klasyczne zadanie segmentacji marketingowej.

Zbiór wczytasz bezpośrednio z URL-a (to publiczny, popularny dataset):

```python
import pandas as pd

url = "https://raw.githubusercontent.com/SteffiPeTaffy/machineLearningAZ/master/Machine%20Learning%20A-Z%20Template%20Folder/Part%204%20-%20Clustering/Section%2025%20-%20Hierarchical%20Clustering/Mall_Customers.csv"
df = pd.read_csv(url)
df.head()
```

Kolumny: `CustomerID`, `Genre` (płeć), `Age`, `Annual Income (k$)`, `Spending Score (1-100)`.

> Jeśli URL nie zadziała — pobierz „Mall_Customers.csv" z Kaggle (Mall Customer Segmentation Data) i wgraj do Colaba przez `files.upload()`.

---

#### 🏋️ Etap 1 — Przygotowanie danych (2 pkt)

1. **(0.5 pkt)** Wyświetl `shape`, `head()` i `info()`. W komentarzu napisz, ilu jest klientów i czy są braki danych.

2. **(1 pkt)** Wybierz do klastrowania **dwie cechy numeryczne**: `Annual Income (k$)` oraz `Spending Score (1-100)` (zostawiamy 2 cechy, żeby dało się **narysować** wynik na płaszczyźnie). Zapisz je jako `X`. **Przeskaluj** je `StandardScaler` (przypomnienie z zajęć 5 — algorytmy liczą odległości!).

3. **(0.5 pkt)** Narysuj wykres rozrzutu surowych danych (`income` vs `spending`). W komentarzu napisz, ile grup wstępnie widzisz „na oko".

---

#### 🏋️ Etap 2 — K-Means + dobór liczby klastrów (4 pkt)

1. **(1.5 pkt)** Wykonaj **metodę łokcia** dla `k` od 1 do 10 (zbierz `inertia_` i narysuj wykres). W komentarzu wskaż, gdzie jest „łokieć".

2. **(1 pkt)** Policz **silhouette score** dla `k` od 2 do 8. Wskaż `k` o najwyższej wartości. Czy zgadza się z łokciem?

3. **(1 pkt)** Wytrenuj `KMeans` z wybranym `k` i narysuj klastry różnymi kolorami **wraz z centroidami** (czerwone `X`).

4. **(0.5 pkt)** W komentarzu **opisz segmenty po ludzku** — np. „klaster 0 = wysoki dochód + niskie wydatki = klienci oszczędni". To jest sedno segmentacji: liczby muszą zamienić się w wnioski biznesowe.

---

#### 🏋️ Etap 3 — Hierarchiczna i DBSCAN (3 pkt)

1. **(1 pkt)** Narysuj **dendrogram** (`scipy.cluster.hierarchy.linkage` + `dendrogram`, `method="ward"`). W komentarzu napisz, ile klastrów sugeruje cięcie dendrogramu.

2. **(1 pkt)** Wytrenuj `AgglomerativeClustering` z tą samą liczbą klastrów co w K-Means i narysuj wynik. Porównaj w komentarzu z podziałem z K-Means — czy segmenty wyszły podobne?

3. **(1 pkt)** Wytrenuj `DBSCAN` (dobierz `eps` i `min_samples` — zacznij od `eps=0.5`, `min_samples=5` na przeskalowanych danych i **poeksperymentuj**). Narysuj wynik z **szumem na osobnym kolorze**. W komentarzu: ilu klientów DBSCAN uznał za **outliery** (etykieta `-1`) i czy to ma sens (np. nietypowy profil wydatków)?

---

#### 🏋️ Etap 4 — Wnioski (1 pkt)

1. **(1 pkt)** W komentarzu (3–5 zdań) podsumuj: **który algorytm dał najsensowniejszy podział** dla tego zbioru i dlaczego? Który segment klientów jest najciekawszy z punktu widzenia marketingu (np. „wysoki dochód, wysokie wydatki — VIP-y, warci programu lojalnościowego")?

---

## Podsumowanie punktacji

| Etap | Temat | Punkty |
|------|-------|--------|
| 1 | Przygotowanie danych (wybór cech, skalowanie) | 2 |
| 2 | K-Means + łokieć + silhouette + opis segmentów | 4 |
| 3 | Hierarchiczna (dendrogram) + DBSCAN (szum) | 3 |
| 4 | Wnioski i porównanie algorytmów | 1 |
| **Suma** | | **10** |

> To krótsze zajęcia — maksymalnie **10 pkt** zdobywasz, prezentując rozwiązanie prowadzącemu na zajęciach. Zadanie wysyłasz na Moodle najpóźniej dzień przed... no właśnie, to ostatnie zajęcia — termin oddania ustalimy wspólnie. 🎓

---

## To już koniec kursu — co dalej?

Przeszliśmy razem całą drogę: od podstaw Pythona i klas (zajęcia 1–2), przez NLP (3), regresję (4) i klasyfikację (5), aż po uczenie nienadzorowane (dziś). Masz teraz solidny fundament Data Science w Pythonie.

Naturalne kierunki dalej:

- **Strojenie hiperparametrów** — `GridSearchCV`, `RandomizedSearchCV` (jak automatycznie znaleźć najlepsze `k`, `max_depth` itd.),
- **Pipeline'y w scikit-learn** — `Pipeline` spinający skalowanie + model w jeden obiekt (koniec z ręcznym `fit_transform`),
- **Redukcja wymiarów** — **PCA**, **t-SNE**, **UMAP** (drugi wielki dział uczenia nienadzorowanego — wizualizacja danych wielowymiarowych),
- **Sieci neuronowe** — `PyTorch` / `TensorFlow`, gdy dane przestają być tabelaryczne (obrazy, dźwięk, tekst).

---

## Materiały dodatkowe

- [scikit-learn — Clustering (przegląd wszystkich algorytmów)](https://scikit-learn.org/stable/modules/clustering.html)
- [scikit-learn — `KMeans`](https://scikit-learn.org/stable/modules/generated/sklearn.cluster.KMeans.html)
- [scikit-learn — `DBSCAN`](https://scikit-learn.org/stable/modules/generated/sklearn.cluster.DBSCAN.html)
- [scikit-learn — `AgglomerativeClustering`](https://scikit-learn.org/stable/modules/generated/sklearn.cluster.AgglomerativeClustering.html)
- [scikit-learn — silhouette analysis](https://scikit-learn.org/stable/auto_examples/cluster/plot_kmeans_silhouette_analysis.html)
- [scipy — dendrogramy (`linkage`)](https://docs.scipy.org/doc/scipy/reference/generated/scipy.cluster.hierarchy.linkage.html)
- [Porównanie algorytmów klastrowania — galeria scikit-learn](https://scikit-learn.org/stable/auto_examples/cluster/plot_cluster_comparison.html)
