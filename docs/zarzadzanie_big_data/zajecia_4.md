# Zajęcia 4

## Wizualizacja danych i budowanie interaktywnego dashboardu

Na poprzednich zajęciach robiliśmy mnóstwo wykresów — histogramy, boxploty, scattery, heatmapy. Wszystkie miały jednego odbiorcę: **nas samych**. To była wizualizacja **eksploracyjna** — szybka, brzydka, jednorazowa. Wykres odpalany w notebooku, żeby coś zrozumieć, i zaraz potem zapominany.

Dziś zmieniamy odbiorcę. Wizualizacja **prezentacyjna** to wykres, który pokazujesz innym — szefowi, klientowi, czytelnikowi raportu. Musi być **czytelny**, **estetyczny** i często **interaktywny** — żeby odbiorca mógł sam pofiltrować, najechać, przybliżyć. A na końcu cały ten zestaw wykresów lądujemy w **dashboardzie** — jednej stronie, na której wszystko widać naraz.

Po dzisiejszych zajęciach będziesz w stanie:

- odróżnić wizualizację eksploracyjną od prezentacyjnej i wiedzieć, kiedy stosować którą,
- stosować podstawowe zasady dobrej wizualizacji (kolor, skala, hierarchia informacji),
- tworzyć interaktywne wykresy w `plotly` (express i graph objects),
- komponować kilka wykresów w jednym obrazku (`subplots`),
- pokazywać dane geograficzne na mapie,
- zbudować w pełni działający dashboard w Streamlit — z widgetami, layoutem i cachowaniem.

---

## Dorzucamy pakiety do projektu

Dziś dochodzą dwa kluczowe pakiety: `plotly` (interaktywne wykresy) i `streamlit` (dashboardy).

```bash
uv add plotly streamlit
```

Standardowy zestaw importów na dziś:

```python
import pandas as pd
import numpy as np
import plotly.express as px
import plotly.graph_objects as go
from plotly.subplots import make_subplots
```

`plotly.express` (skrót `px`) to wysokopoziomowe API — jedna linijka, gotowy wykres. `plotly.graph_objects` (skrót `go`) to niskopoziomowe API, gdy potrzebujesz pełnej kontroli. W praktyce 80% czasu zostajesz przy `express`.

---

## Eksploracja vs prezentacja — to nie ten sam wykres

Wykres eksploracyjny i prezentacyjny mają zupełnie inne cele:

| | Eksploracyjny | Prezentacyjny |
|---|---|---|
| Odbiorca | Ja sam | Inni |
| Czas tworzenia | 5 sekund | 5–30 minut |
| Styl | Domyślny | Dopracowany |
| Tytuł, etykiety | Często brak | Obowiązkowe |
| Interakcja | Niepotrzebna | Często warta |
| Cykl życia | Wyrzucany po użyciu | Trafia do raportu / dashboardu |

Na zajęciach 3 robiliśmy histogram `sns.histplot(df["Age"])` w jednej linijce — bo nas interesowało tylko „jak wygląda rozkład". Dziś, jeśli ten histogram ma trafić do raportu, musi mieć **tytuł**, **opisane osie**, **sensowną liczbę binów**, **ładny kolor** i być w **rozmiarze pasującym do prezentacji**.

### Zasady dobrej wizualizacji — w pigułce

1. **Jeden wykres = jedna myśl.** Jeśli próbujesz pokazać pięć rzeczy naraz, zrób pięć wykresów.
2. **Mniej znaczy więcej.** Każdy dodatkowy element (linia siatki, ramka, legenda) powinien być uzasadniony. Reszta to chartjunk.
3. **Kolor ma znaczenie.** Używaj go celowo: do kodowania kategorii, sekwencji, dywergencji. Nie do dekoracji.
4. **Skala osi.** Słupki **zawsze** zaczynaj od 0 (inaczej wprowadzasz w błąd). Skale logarytmiczne — gdy dane różnią się o rzędy wielkości.
5. **Hierarchia.** Najważniejsza informacja powinna być najbardziej widoczna. Reszta — w tle.
6. **Wykresy kołowe i 3D — ostrożnie.** Pie chart jest OK dla 2–3 kategorii. Powyżej tego, słupkowy zawsze wygrywa. 3D najczęściej psuje, nie pomaga.

!!! tip "Dobry test"
    Pokaż wykres komuś, kto nie zna kontekstu. Jeśli musisz dopowiadać po słowie „o co tu chodzi", to wykres nie zrobił swojej roboty.

---

## Plotly — interaktywne wykresy

Matplotlib i seaborn rysują statyczne obrazki (PNG). Plotly rysuje **HTML-owe wykresy z JavaScriptem pod spodem** — można w nie najechać, przybliżyć, schować serię, zapisać do PNG.

### Plotly Express — wykres w jednej linijce

```python
import plotly.express as px

# Wbudowany przykładowy zbiór
df = px.data.gapminder()

fig = px.scatter(
    df.query("year == 2007"),
    x="gdpPercap",
    y="lifeExp",
    size="pop",
    color="continent",
    hover_name="country",
    log_x=True,
    size_max=60,
    title="PKB na osobę vs oczekiwana długość życia (2007)"
)
fig.show()
```

W jednej funkcji `px.scatter` siedzi: oś X, oś Y, kodowanie rozmiarem (`size`), kodowanie kolorem (`color`), tekst po najechaniu (`hover_name`), skala logarytmiczna i tytuł. To esencja Plotly Express.

### Podstawowe typy wykresów

```python
# Słupkowy
px.bar(df, x="kategoria", y="wartosc", color="grupa")

# Liniowy (świetny do szeregów czasowych)
px.line(df, x="data", y="cena", color="produkt")

# Punktowy (scatter)
px.scatter(df, x="x", y="y", color="kategoria")

# Histogram
px.histogram(df, x="cena", nbins=30, color="kategoria")

# Boxplot / violin
px.box(df, x="kategoria", y="cena")
px.violin(df, x="kategoria", y="cena", box=True)

# Heatmapa
px.imshow(macierz_korelacji, text_auto=True, color_continuous_scale="RdBu_r")

# Sunburst (hierarchiczny pie chart)
px.sunburst(df, path=["region", "kraj"], values="populacja")
```

### Dostrajanie wykresu — `update_layout` i `update_traces`

Plotly Express zwraca obiekt `Figure`, który możesz modyfikować dalej:

```python
fig = px.bar(df, x="miasto", y="liczba_zamowien")

fig.update_layout(
    title="Zamówienia wg miasta — styczeń 2025",
    xaxis_title="Miasto",
    yaxis_title="Liczba zamówień",
    showlegend=False,
    template="plotly_white",          # czyste białe tło
    width=900, height=500,
)

fig.update_traces(
    marker_color="steelblue",
    hovertemplate="<b>%{x}</b><br>Zamówień: %{y}<extra></extra>"
)

fig.show()
```

!!! note "Templaty"
    Plotly ma kilka gotowych stylów: `plotly`, `plotly_white`, `plotly_dark`, `simple_white`, `seaborn`, `ggplot2`. Ustawiasz przez `template=` w `update_layout` albo globalnie:
    ```python
    import plotly.io as pio
    pio.templates.default = "plotly_white"
    ```

### Plotly Graph Objects — gdy Express nie wystarcza

Jeśli chcesz na jednym wykresie połączyć rzeczy, których Express nie składa naturalnie (np. wykres słupkowy + linia ze średnią), używasz `graph_objects`:

```python
import plotly.graph_objects as go

fig = go.Figure()

# Słupki
fig.add_trace(go.Bar(
    x=df["miesiac"],
    y=df["sprzedaz"],
    name="Sprzedaż",
    marker_color="steelblue"
))

# Linia ze średnią kroczącą
fig.add_trace(go.Scatter(
    x=df["miesiac"],
    y=df["sprzedaz"].rolling(3).mean(),
    name="Średnia krocząca (3 mies.)",
    mode="lines+markers",
    line=dict(color="crimson", width=3)
))

fig.update_layout(
    title="Sprzedaż miesięczna",
    xaxis_title="Miesiąc",
    yaxis_title="Sprzedaż [PLN]",
    template="plotly_white"
)
fig.show()
```

### Subploty — kilka wykresów obok siebie

```python
from plotly.subplots import make_subplots

fig = make_subplots(
    rows=2, cols=2,
    subplot_titles=("Rozkład cen", "Sprzedaż w czasie",
                    "Sprzedaż wg kategorii", "Cena vs ilość")
)

fig.add_trace(go.Histogram(x=df["cena"]), row=1, col=1)
fig.add_trace(go.Scatter(x=df["data"], y=df["sprzedaz"], mode="lines"), row=1, col=2)
fig.add_trace(go.Bar(x=df["kategoria"], y=df["liczba"]), row=2, col=1)
fig.add_trace(go.Scatter(x=df["cena"], y=df["ilosc"], mode="markers"), row=2, col=2)

fig.update_layout(height=700, showlegend=False, title_text="Mini-dashboard")
fig.show()
```

### Mapy — `scatter_mapbox` i `choropleth`

Plotly potrafi rysować na mapach bez żadnych dodatkowych pakietów geograficznych. Najprostszy wariant — `scatter_mapbox`, czyli „scatter, ale w miejsce X/Y wstawiasz lat/lon":

```python
df_miasta = pd.DataFrame({
    "miasto": ["Warszawa", "Kraków", "Wrocław", "Poznań", "Gdańsk"],
    "lat": [52.2297, 50.0647, 51.1079, 52.4064, 54.3520],
    "lon": [21.0122, 19.9450, 17.0385, 16.9252, 18.6466],
    "populacja": [1_860_281, 779_115, 674_312, 538_633, 486_022],
})

fig = px.scatter_mapbox(
    df_miasta,
    lat="lat", lon="lon",
    size="populacja",
    hover_name="miasto",
    zoom=5, center={"lat": 52, "lon": 19},
    mapbox_style="open-street-map",
    height=500
)
fig.show()
```

`mapbox_style="open-street-map"` nie wymaga klucza API — dostajesz mapę OSM za darmo.

---

## Streamlit — od wykresów do dashboardu

Plotly daje pojedyncze wykresy. Streamlit pozwala je **zorganizować** — z widgetami do filtrowania, podziałem na zakładki, sidebarem z parametrami. Wszystko w czystym Pythonie, bez HTML i JavaScriptu.

### Czym jest Streamlit?

Streamlit to framework do szybkiego budowania aplikacji webowych w Pythonie. Piszesz zwykły skrypt — Streamlit zamienia go w działającą stronę. Filozofia: **każda interakcja użytkownika = ponowne uruchomienie skryptu od góry do dołu**. Brzmi nieoptymalnie, ale w praktyce działa świetnie (z cachowaniem, o którym za chwilę).

### Pierwsza aplikacja

Stwórz plik `app.py`:

```python
import streamlit as st
import pandas as pd

st.title("Moja pierwsza aplikacja")
st.write("Cześć, świecie!")

df = pd.DataFrame({"x": [1, 2, 3], "y": [10, 20, 30]})
st.dataframe(df)
```

Uruchom:

```bash
uv run streamlit run app.py
```

W przeglądarce otworzy się `http://localhost:8501` z gotową aplikacją. Edytujesz plik, zapisujesz — strona sama proponuje przeładowanie.

### Wyświetlanie tekstu i danych

```python
st.title("Tytuł strony")
st.header("Sekcja")
st.subheader("Podsekcja")
st.markdown("To jest **pogrubiony** tekst z [linkiem](https://streamlit.io).")
st.write("Cokolwiek — DataFrame, dict, lista, liczba, tekst.")
st.code("print('hello')", language="python")
st.dataframe(df)              # interaktywna tabela (sortowanie, filtrowanie)
st.table(df.head())           # statyczna tabela
st.metric("Sprzedaż", "1.2M zł", "+15%")   # KPI box (z deltą)
st.json({"klucz": "wartość"}) # ładnie sformatowany JSON
```

### Widgety wejściowe

```python
nazwa = st.text_input("Twoje imię", value="Anna")
wiek = st.slider("Wiek", min_value=0, max_value=100, value=25)
przedzial = st.slider("Przedział cen", 0, 1000, (100, 500))  # zakres
kategoria = st.selectbox("Kategoria", ["A", "B", "C"])
tagi = st.multiselect("Tagi", ["x", "y", "z"], default=["x"])
data = st.date_input("Wybierz datę")
zaznaczone = st.checkbox("Zgadzam się")
plik = st.file_uploader("Wgraj CSV", type=["csv"])
button = st.button("Kliknij mnie")
```

Każdy widget zwraca aktualną wartość. Możesz jej używać dalej w skrypcie:

```python
wiek = st.slider("Wiek", 0, 100, 25)
st.write(f"Za 10 lat będziesz miał {wiek + 10} lat.")
```

### Wyświetlanie wykresów

```python
# Wbudowane (szybkie, ale ograniczone)
st.line_chart(df["sprzedaz"])
st.bar_chart(df.set_index("miesiac")["sprzedaz"])

# Plotly (polecane)
fig = px.bar(df, x="miasto", y="liczba")
st.plotly_chart(fig, use_container_width=True)

# Matplotlib też działa
fig, ax = plt.subplots()
ax.hist(df["wiek"])
st.pyplot(fig)
```

`use_container_width=True` rozciąga wykres na pełną szerokość — prawie zawsze tego chcesz.

### Layout — sidebar, kolumny, zakładki

```python
# Sidebar — typowo na filtry i kontrolki
with st.sidebar:
    st.title("Filtry")
    region = st.selectbox("Region", ["Mazowsze", "Małopolska", "Śląsk"])
    rok = st.slider("Rok", 2020, 2025, 2024)

# Kolumny — KPI obok siebie
col1, col2, col3 = st.columns(3)
col1.metric("Sprzedaż", "1.2M zł", "+15%")
col2.metric("Klienci", "847", "+23")
col3.metric("Konwersja", "3.2%", "-0.4%")

# Zakładki — kilka widoków w jednym
tab1, tab2, tab3 = st.tabs(["📊 Wykresy", "📋 Dane", "ℹ️ Info"])

with tab1:
    st.plotly_chart(fig)

with tab2:
    st.dataframe(df)

with tab3:
    st.markdown("Dashboard sprzedaży — wersja 1.0")
```

### Cachowanie — `@st.cache_data`

Streamlit re-uruchamia cały skrypt przy każdej interakcji z widgetem. Dla małych zbiorów to nie problem. Dla dużych pliku CSV / zapytania SQL / requestu do API — masakra. Stąd dekorator:

```python
@st.cache_data
def wczytaj_dane(sciezka):
    return pd.read_csv(sciezka)

df = wczytaj_dane("duzy_plik.csv")  # tylko pierwszy raz wykonuje się read_csv
```

Wynik jest cachowany na podstawie argumentów funkcji. Zmienisz argument → funkcja wykonuje się od nowa. Ten sam argument → wynik z cache'a.

!!! tip "Kiedy cachować"
    Cachuj wszystko, co jest **deterministyczne i kosztowne**: wczytywanie plików, zapytania SQL, pobrania z API, ciężkie transformacje. Nie cachuj rzeczy losowych ani wykresów (te są tanie do narysowania).

### Działający mini-dashboard

Sklejmy wszystko w jedną aplikację:

```python
# app.py
import streamlit as st
import pandas as pd
import plotly.express as px

st.set_page_config(page_title="Dashboard zamówień", layout="wide")

@st.cache_data
def wczytaj_dane():
    return pd.read_csv("zamowienia_clean.csv", parse_dates=["data_zamowienia"])

df = wczytaj_dane()

st.title("📦 Dashboard zamówień")

# Filtry w sidebarze
with st.sidebar:
    st.header("Filtry")
    kategorie = st.multiselect(
        "Kategoria",
        options=df["kategoria"].unique(),
        default=df["kategoria"].unique()
    )
    zakres_dat = st.date_input(
        "Zakres dat",
        value=(df["data_zamowienia"].min(), df["data_zamowienia"].max())
    )

# Filtrowanie danych
df_f = df[
    (df["kategoria"].isin(kategorie)) &
    (df["data_zamowienia"].between(*pd.to_datetime(zakres_dat)))
]

# KPI w kolumnach
c1, c2, c3 = st.columns(3)
c1.metric("Liczba zamówień", f"{len(df_f):,}")
c2.metric("Łączna wartość", f"{df_f['wartosc_zamowienia'].sum():,.0f} zł")
c3.metric("Średnie zamówienie", f"{df_f['wartosc_zamowienia'].mean():,.0f} zł")

# Wykresy
col_a, col_b = st.columns(2)
with col_a:
    fig1 = px.bar(
        df_f.groupby("kategoria")["wartosc_zamowienia"].sum().reset_index(),
        x="kategoria", y="wartosc_zamowienia",
        title="Wartość zamówień wg kategorii"
    )
    st.plotly_chart(fig1, use_container_width=True)

with col_b:
    df_dzienne = df_f.groupby(df_f["data_zamowienia"].dt.date)["wartosc_zamowienia"].sum().reset_index()
    fig2 = px.line(
        df_dzienne,
        x="data_zamowienia", y="wartosc_zamowienia",
        title="Sprzedaż dzienna"
    )
    st.plotly_chart(fig2, use_container_width=True)

# Surowe dane na dole
with st.expander("📋 Pokaż surowe dane"):
    st.dataframe(df_f)
```

To w sumie ~60 linii kodu — i mamy w pełni działający dashboard z filtrami, KPI i dwoma wykresami. Klasyczny Power BI / Tableau zajmuje na to dzień klikania.

!!! note "Streamlit vs Power BI / Tableau"
    Streamlit nie zastąpi w pełni Power BI w korporacji — narzędzia BI mają lepszą integrację z hurtowniami, role i uprawnienia, harmonogramy odświeżeń. Ale jako narzędzie **dla analityka, który zna Pythona**, Streamlit jest bezkonkurencyjny — pełna kontrola, brak ograniczeń wizualnych, działa z każdą biblioteką ML.

### Wieloplikowe aplikacje (multipage apps)

Gdy aplikacja rośnie, dzielisz ją na strony. Struktura folderów:

```
projekt/
├── app.py              # strona główna
└── pages/
    ├── 1_📊_Sprzedaz.py
    ├── 2_👥_Klienci.py
    └── 3_📦_Produkty.py
```

Streamlit automatycznie zbuduje menu nawigacji z plików w folderze `pages/`. Każdy plik to osobny skrypt — uruchamiany niezależnie.

### Deployment — gdzie postawić aplikację?

Najprostszy darmowy wariant: **Streamlit Community Cloud** ([share.streamlit.io](https://share.streamlit.io/)). Wrzucasz kod na GitHuba, klikasz „Deploy", masz aplikację pod publicznym URL w 2 minuty. Limit: 1 GB RAM, publiczny kod.

Inne opcje: Heroku, Render, Railway, własny serwer + Docker, Hugging Face Spaces.

---

## Ćwiczenie łączone — mini-dashboard pogody

Zróbmy razem coś od zera — dashboard prezentujący temperaturę w polskich miastach. Pobierzemy dane z darmowego API Open-Meteo, narysujemy parę wykresów w plotly i opakujemy w Streamlit.

### Krok 1 — pobranie danych z API

```python
import requests
import pandas as pd

miasta = {
    "Warszawa": (52.23, 21.01),
    "Kraków": (50.06, 19.94),
    "Wrocław": (51.11, 17.04),
    "Gdańsk": (54.35, 18.65),
    "Poznań": (52.41, 16.93),
}

frames = []
for miasto, (lat, lon) in miasta.items():
    url = (
        f"https://api.open-meteo.com/v1/forecast"
        f"?latitude={lat}&longitude={lon}"
        f"&hourly=temperature_2m"
        f"&past_days=14&forecast_days=1"
    )
    response = requests.get(url)
    data = response.json()

    df_temp = pd.DataFrame({
        "data": pd.to_datetime(data["hourly"]["time"]),
        "temperatura": data["hourly"]["temperature_2m"],
        "miasto": miasto,
        "lat": lat,
        "lon": lon,
    })
    frames.append(df_temp)

df = pd.concat(frames, ignore_index=True)
df.to_csv("pogoda.csv", index=False)
print(f"Pobrano {len(df)} obserwacji dla {df['miasto'].nunique()} miast")
```

### Krok 2 — proste wykresy plotly

```python
import plotly.express as px

# Linia — temperatura w czasie dla każdego miasta
fig1 = px.line(
    df, x="data", y="temperatura", color="miasto",
    title="Temperatura w ostatnich 14 dniach",
    labels={"data": "Data", "temperatura": "Temperatura [°C]"}
)
fig1.show()

# Boxplot — rozkład temperatur per miasto
fig2 = px.box(
    df, x="miasto", y="temperatura",
    title="Rozkład temperatur wg miasta"
)
fig2.show()
```

### Krok 3 — mapa średnich temperatur

```python
df_srednie = df.groupby(["miasto", "lat", "lon"], as_index=False)["temperatura"].mean()

fig3 = px.scatter_mapbox(
    df_srednie,
    lat="lat", lon="lon",
    size=df_srednie["temperatura"].abs() + 1,    # rozmiar nie może być ujemny
    color="temperatura",
    color_continuous_scale="RdBu_r",
    hover_name="miasto",
    zoom=5, center={"lat": 52, "lon": 19},
    mapbox_style="open-street-map",
    height=500,
    title="Średnia temperatura w polskich miastach"
)
fig3.show()
```

### Krok 4 — opakowanie w Streamlit

```python
# pogoda_app.py
import streamlit as st
import pandas as pd
import plotly.express as px

st.set_page_config(page_title="Pogoda PL", layout="wide")

@st.cache_data
def wczytaj():
    return pd.read_csv("pogoda.csv", parse_dates=["data"])

df = wczytaj()

st.title("🌡️ Pogoda w polskich miastach")

wybrane = st.sidebar.multiselect(
    "Miasta",
    options=df["miasto"].unique(),
    default=list(df["miasto"].unique())
)

df_f = df[df["miasto"].isin(wybrane)]

c1, c2, c3 = st.columns(3)
c1.metric("Średnia", f"{df_f['temperatura'].mean():.1f}°C")
c2.metric("Maksimum", f"{df_f['temperatura'].max():.1f}°C")
c3.metric("Minimum", f"{df_f['temperatura'].min():.1f}°C")

tab1, tab2 = st.tabs(["📈 Wykres", "🗺️ Mapa"])

with tab1:
    fig = px.line(df_f, x="data", y="temperatura", color="miasto")
    st.plotly_chart(fig, use_container_width=True)

with tab2:
    df_srednie = df_f.groupby(["miasto", "lat", "lon"], as_index=False)["temperatura"].mean()
    fig = px.scatter_mapbox(
        df_srednie, lat="lat", lon="lon",
        size=df_srednie["temperatura"].abs() + 1,
        color="temperatura", color_continuous_scale="RdBu_r",
        hover_name="miasto",
        zoom=5, mapbox_style="open-street-map", height=500
    )
    st.plotly_chart(fig, use_container_width=True)
```

Uruchom: `uv run streamlit run pogoda_app.py`. Gotowy dashboard.

To przepływ, który będzie ci towarzyszył w pracy projektowej: **dane → DataFrame → wykresy plotly → Streamlit z layoutem i filtrami**.

---

## Zadanie

### Wizualizacja rynku koncertów muzycznych w Polsce

Twoim zadaniem jest przygotowanie zestawu **interaktywnych wykresów w plotly** dla zbioru danych o koncertach w Polsce. To trening przed pracą projektową — wszystkie wykresy z tego zadania będziesz mógł później wrzucić do Streamlita.

**Zadanie nie wymaga Streamlita.** Wystarczy Jupyter Notebook i plotly.

#### Generowanie danych

Uruchom poniższy skrypt **raz**, żeby wygenerować plik z danymi:

```python
import pandas as pd
import numpy as np
from datetime import datetime, timedelta

np.random.seed(42)

n = 1200

miasta = {
    "Warszawa":   (52.2297, 21.0122, 1.00),
    "Kraków":     (50.0647, 19.9450, 0.75),
    "Wrocław":    (51.1079, 17.0385, 0.65),
    "Poznań":     (52.4064, 16.9252, 0.55),
    "Gdańsk":     (54.3520, 18.6466, 0.55),
    "Łódź":       (51.7592, 19.4560, 0.50),
    "Katowice":   (50.2649, 19.0238, 0.45),
    "Lublin":     (51.2465, 22.5684, 0.30),
    "Białystok":  (53.1325, 23.1688, 0.25),
    "Szczecin":   (53.4285, 14.5528, 0.35),
}

gatunki = ["rock", "pop", "hip-hop", "electronic", "jazz",
           "classical", "folk", "metal", "indie", "reggae"]

typy_obiektow = ["klub", "arena", "stadion", "festiwal", "teatr", "amfiteatr"]
kapacjety = {
    "klub": (200, 1500), "arena": (3000, 15000), "stadion": (20000, 70000),
    "festiwal": (10000, 80000), "teatr": (400, 2000), "amfiteatr": (1500, 8000),
}
cena_bazowa = {
    "rock": 150, "pop": 200, "hip-hop": 180, "electronic": 160, "jazz": 130,
    "classical": 110, "folk": 90, "metal": 140, "indie": 100, "reggae": 110,
}
cena_mnoznik = {
    "klub": 0.7, "arena": 1.3, "stadion": 1.8,
    "festiwal": 1.5, "teatr": 1.2, "amfiteatr": 1.0,
}

start_date = datetime(2024, 1, 1)
daty = [start_date + timedelta(days=int(d)) for d in np.random.randint(0, 730, n)]

wagi = np.array([miasta[m][2] for m in miasta])
miasto = np.random.choice(list(miasta.keys()), n, p=wagi / wagi.sum())
gatunek = np.random.choice(gatunki, n)
typ_obiektu = np.random.choice(typy_obiektow, n,
                                p=[0.40, 0.15, 0.05, 0.10, 0.15, 0.15])

pojemnosc = np.array([np.random.randint(*kapacjety[t]) for t in typ_obiektu])
wypelnienie = np.clip(np.random.beta(5, 2, n), 0.15, 1.0)
sprzedane = (pojemnosc * wypelnienie).astype(int)

cena = np.array([cena_bazowa[g] * cena_mnoznik[t] for g, t in zip(gatunek, typ_obiektu)])
cena = np.round(cena * np.random.uniform(0.7, 1.4, n), -1)
przychod = (cena * sprzedane).astype(int)

df = pd.DataFrame({
    "event_id": range(50001, 50001 + n),
    "data": daty,
    "miasto": miasto,
    "latitude": [miasta[m][0] for m in miasto],
    "longitude": [miasta[m][1] for m in miasto],
    "gatunek": gatunek,
    "typ_obiektu": typ_obiektu,
    "pojemnosc": pojemnosc,
    "bilety_sprzedane": sprzedane,
    "cena_biletu_pln": cena,
    "przychod_pln": przychod,
})

df.to_csv("koncerty_polska.csv", index=False)
print(f"Wygenerowano plik 'koncerty_polska.csv' — {len(df)} koncertów")
```

#### Polecenia

**Część 1 — Wczytanie i wstępna eksploracja (1 pkt)**

1. Wczytaj plik `koncerty_polska.csv` (pamiętaj o `parse_dates=["data"]`).
2. Wyświetl `shape`, `head()`, `dtypes`. Sprawdź, ile jest unikalnych miast i gatunków.

**Część 2 — Wykres słupkowy (2 pkt)**

3. Stwórz interaktywny wykres słupkowy w plotly pokazujący **łączny przychód w każdym mieście**. Posortuj słupki malejąco. Dodaj tytuł i sensowne etykiety osi. Wykres ma być czytelny — żadnych domyślnych „untitled" w tytułach.

**Część 3 — Wykres liniowy / szereg czasowy (2 pkt)**

4. Zagreguj dane do poziomu miesiąca i narysuj wykres liniowy pokazujący **łączną liczbę koncertów w każdym miesiącu**. Podpowiedź: `df["data"].dt.to_period("M").astype(str)` daje czytelne etykiety typu `"2024-03"`.
5. W drugim wykresie liniowym pokaż **miesięczną liczbę koncertów z podziałem na typ obiektu** (każdy typ jako osobna linia).

**Część 4 — Histogram i boxplot (2 pkt)**

6. Narysuj histogram cen biletów. Spróbuj różnych wartości `nbins` (20, 50, 100) i wybierz tę, która najlepiej pokazuje rozkład.
7. Narysuj boxplot przychodu z podziałem na typ obiektu (`x="typ_obiektu"`, `y="przychod_pln"`). Skomentuj w markdownie, który typ obiektu generuje najwyższe przychody.

**Część 5 — Scatter plot z kodowaniem koloru (2 pkt)**

8. Dodaj do DataFrame'u kolumnę `wypelnienie = bilety_sprzedane / pojemnosc`.
9. Narysuj scatter plot: oś X — `cena_biletu_pln`, oś Y — `wypelnienie`, **kolor** wg `gatunek`, **rozmiar** wg `pojemnosc`, w `hover` pokaż `miasto` i `typ_obiektu`. Skomentuj w markdownie: czy widać zależność między ceną biletu a wypełnieniem sali?

**Część 6 — Mapa (2 pkt)**

10. Zagreguj dane do poziomu miasta: średnia cena biletu, liczba koncertów, łączny przychód, wraz z `latitude` i `longitude`.
11. Narysuj `px.scatter_mapbox` polskich miast — **rozmiar** punktu wg liczby koncertów, **kolor** wg średniej ceny biletu. Użyj `mapbox_style="open-street-map"` (bez klucza API). W `hover` pokaż nazwę miasta i wszystkie zagregowane wartości.

**Część 7 — Kompozycja: subploty (2 pkt)**

12. Stwórz figurę 2×2 z `make_subplots`, w której umieścisz **cztery różne wykresy** podsumowujące zbiór (do wyboru, np.: słupki przychodu wg miasta, słupki liczby koncertów wg gatunku, histogram cen, boxplot wypełnienia wg typu obiektu). Każdy subplot ma mieć tytuł. Cała figura — wspólny nagłówek.

**Część 8 — Wnioski (1 pkt)**

13. W komórce markdown napisz **3–5 wniosków** z całej analizy. Czego dowiedziałeś się o rynku koncertów w Polsce? Które miasto ma najwięcej imprez? W jakim typie obiektu są najwyższe ceny? Czy widać sezonowość?

#### Kryteria zaliczenia

- Wczytanie i eksploracja — **1 pkt**
- Wykres słupkowy — **2 pkt**
- Wykresy liniowe (dwa) — **2 pkt**
- Histogram + boxplot — **2 pkt**
- Scatter plot — **2 pkt**
- Mapa — **2 pkt**
- Subploty 2×2 — **2 pkt**
- Wnioski w markdown — **1 pkt**
- Wysłanie zadania w trakcie zajęć — **1 pkt**

**TOTAL: 15 pkt**

!!! tip "Bonus do projektu zaliczeniowego"
    Jeśli zachowasz kod z tego zadania w czystej formie (każdy wykres jako osobna funkcja `def plot_xxx(df) -> go.Figure`), będzie ci go bardzo łatwo wrzucić do Streamlita w pracy projektowej. To dobry trening na koniec semestru.

Powodzenia! 🎸📊
