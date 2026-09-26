# Praca projektowa 

**Aplikacja analityczna w Streamlit**

## O projekcie

Praca zaliczeniowa to **indywidualna aplikacja analityczna w Streamlit**, która spina cały materiał semestru w jeden praktyczny projekt. Wybierasz interesujący cię zbiór danych, czyścisz go, analizujesz, wizualizujesz — i opakowujesz w działający dashboard, do którego mogę wejść z dowolnej przeglądarki i kliknąć kilka filtrów.

Projekt sprawdza, czy umiesz przeprowadzić **pełny przepływ pracy z danymi** — od pozyskania, przez czyszczenie i analizę, po komunikację wyników. Czyli wszystko, czego uczyliśmy się od zajęć 1 do 4.

---

## Terminy i forma oddania

- **Deadline:** koniec pierwszego terminu sesji — **12 lipca 2026** (do północy)
- **Co oddajesz** (na moodle):
    1. **Link do publicznego repozytorium na GitHubie** z kodem aplikacji
    2. **Działający link** do wdrożonej aplikacji (np. `https://twoja-app.streamlit.app`)
- **Forma:** linki na moodle. Bez ZIP-ów, bez Google Drive, bez „aplikacja u mnie lokalnie działa".

!!! warning "Działa pod linkiem = oddane"
    Jeśli w dniu sprawdzania (po deadline'ie) link do aplikacji nie działa albo wyrzuca błąd na starcie — projekt nie zostaje uznany za oddany. Sprawdź wdrożenie **kilka dni przed deadlinem**, nie w nocy 11 lipca.

---

## Wymagania techniczne

Aplikacja **musi** spełniać poniższe minima:

1. **Prawdziwe źródło danych.** Plik CSV/JSON, REST API, baza SQL, scrape, cokolwiek — byle dane istniały w realnym świecie. Datasety generowane skryptem typu `np.random.normal()` nie wystarczą (chyba że to dodatek do prawdziwych danych — np. symulacja predykcji).
2. **Krok czyszczenia i przygotowania.** Nawet jeśli dane są względnie czyste, pokaż, że umiesz to robić — obsługa braków, konwersje typów, transformacje, kolumny pochodne.
3. **Minimum 5 różnych typów wykresów.** Słupkowy, liniowy, scatter, histogram/boxplot, mapa, heatmapa, sunburst, treemap — wybór jest twój. Liczy się **różnorodność**, nie ilość. Pięć słupkowych z różnymi danymi to nadal jeden typ wykresu.
4. **Minimum 3 widgety filtrujące** — slider, selectbox, multiselect, date_input, text_input. Aplikacja musi **reagować** na zmianę filtrów — przeliczać wykresy, KPI, tabele.
5. **Przemyślany layout.** Sidebar z filtrami, kolumny z KPI, ewentualnie zakładki. Nie wszystko jeden pod drugim w pionie na 3 ekrany scrolla.
6. **Publiczny deployment.** Działający link, dostępny bez logowania.
7. **README.md w repo** — krótki opis: co robi aplikacja, skąd są dane, jak ją uruchomić lokalnie, link do działającej wersji.

---

## Sugerowane tematy i źródła danych

Jeśli nie masz pomysłu — wybierz coś z poniższej listy. Wszystkie źródła są darmowe i nie wymagają płatnego konta (część wymaga rejestracji po klucz API).

### 🌤️ Pogoda i klimat
- **Open-Meteo** ([open-meteo.com](https://open-meteo.com/)) — bez klucza, prognozy i dane historyczne dla dowolnych współrzędnych
- **IMGW** — historyczne dane meteo dla polskich stacji
- Pomysły: temperatura w polskich miastach na mapie, anomalie pogodowe, sezonowość, prognoza vs realizacja

### 🌫️ Jakość powietrza
- **Airly API** (klucz darmowy)
- **GIOŚ** — dane otwarte z państwowego monitoringu
- Pomysły: mapa stacji, PM2.5/PM10 w czasie, ranking najgorszych miast, korelacja z pogodą

### 💱 Kursy walut i kryptowaluty
- **NBP API** ([api.nbp.pl](https://api.nbp.pl/)) — robiliśmy na zajęciach 1
- **CoinGecko API** — historyczne ceny krypto bez klucza
- Pomysły: zmienność walut, korelacje BTC vs altcoiny, wpływ wydarzeń na kursy

### ⚽ Sport
- **Football-data.org** — wyniki lig europejskich (klucz darmowy)
- Datasety z Kaggle (NBA, NFL, F1)
- **api-sports.io** (limit darmowy)
- Pomysły: tabela ligowa na żywo, statystyki zawodników, ranking ELO

### 🎬 Filmy i seriale
- **TMDB API** ([themoviedb.org](https://www.themoviedb.org/)) — klucz darmowy, świetnie udokumentowane
- **IMDb non-commercial datasets** — gotowe TSV-ki
- Pomysły: analiza gatunków, oceny w czasie, ranking reżyserów, koszty vs przychody

### 🎵 Muzyka
- **Spotify Charts** ([charts.spotify.com](https://charts.spotify.com/)) — eksport CSV
- **Last.fm API** — historia odsłuchań i metadane
- **MusicBrainz** — otwarta baza muzyczna
- Pomysły: top hity wg krajów, popularność gatunków, długość utworów w czasie

### 🚇 Transport
- **ZTM Warszawa** — API z rozkładami i pozycjami pojazdów
- **GTFS** — otwarte rozkłady jazdy z różnych miast
- **api-pkp.pl** — dane kolejowe
- Pomysły: mapa linii, punktualność, pokrycie transportem dzielnic

### 🏠 Nieruchomości
- Datasety o cenach mieszkań z Kaggle / GUS
- Możesz też zrobić własny scrape (z poszanowaniem `robots.txt` i rate-limitów)
- Pomysły: mapa cen za m², trendy w dzielnicach, kalkulator zdolności

### 📊 Dane gospodarcze i demograficzne
- **GUS — Bank Danych Lokalnych** ([bdl.stat.gov.pl](https://bdl.stat.gov.pl/)) — masa wskaźników dla Polski
- **World Bank Open Data** — globalne wskaźniki ekonomiczne
- **Eurostat** — dane europejskie
- Pomysły: bezrobocie regionalne, demografia, migracje, PKB

### 🎮 Gaming
- **Steam Web API** — statystyki gier (klucz darmowy)
- **RAWG API** — baza gier
- Datasety o sprzedaży konsolowych gier
- Pomysły: ranking gier, gracze online w czasie, ceny vs oceny

### 💼 Twoje hobby lub praca
Najlepszy projekt to ten, który **naprawdę cię interesuje**. Jeśli grasz na giełdzie — analiza twojego portfela. Jeśli biegasz albo jeździsz na rowerze — twoje dane ze Stravy. Jeśli pracujesz w firmie, gdzie masz dostęp do otwartych danych — bierzesz je (po anonimizacji, jeśli to potrzebne).

!!! tip "Jak wybrać dobry temat"
    Dobry zbiór do dashboardu ma **co najmniej trzy wymiary**: czas (do liniowych), kategorie (do słupkowych) i liczby (do scatter / histogram). Bonus: geografia (lat/lon do mapy). Sprawdź szybko, czy twój zbiór ma to wszystko, zanim zaczniesz coś budować.

---

## Deployment — Streamlit Community Cloud

Najprostsza darmowa opcja: **[share.streamlit.io](https://share.streamlit.io/)**. Limit 1 GB RAM, kod musi być publiczny na GitHubie.

### Krok po kroku

1. **Wrzuć kod na publiczne repo GitHub.** W repo muszą być co najmniej:
    - `app.py` (główny plik aplikacji)
    - `requirements.txt` (lista pakietów — patrz niżej)
    - `README.md` (opis projektu)
2. **Stwórz `requirements.txt`.** Jeśli używasz `uv`, wygeneruj go z `pyproject.toml`:
    ```bash
    uv pip compile pyproject.toml -o requirements.txt
    ```
    Albo wypisz ręcznie:
    ```
    streamlit
    pandas
    plotly
    requests
    numpy
    ```
3. **Wejdź na [share.streamlit.io](https://share.streamlit.io/)** i zaloguj się przez GitHuba.
4. **"New app"** → wybierz repo, branch (`main`), główny plik (`app.py`).
5. **Deploy.** Po 2–3 minutach masz publiczny URL typu `https://twoj-app-xyz.streamlit.app`.

### Częste wpadki przy deployu

- Brak `requirements.txt` → aplikacja nie wstanie
- Hardcoded ścieżki absolutne (`C:/Users/Janek/...`) → działa lokalnie, nie działa na serwerze
- Klucze API w kodzie zamiast w `st.secrets` → bezpieczeństwo + git history
- Plik z danymi > 100 MB → GitHub odmówi pusha, użyj zewnętrznego źródła
- `time.sleep()` lub nieskończona pętla → aplikacja się wiesza

!!! tip "Sekrety w Streamlit"
    Klucze API i hasła nie idą do kodu. Trzymasz je w `secrets.toml` (lokalnie) lub w panelu Streamlit Cloud, a w kodzie odczytujesz przez `st.secrets["nazwa_klucza"]`.

### Alternatywy

Jeśli z jakiegoś powodu Streamlit Community Cloud ci nie pasuje:
- **Hugging Face Spaces** — darmowy, prosty deployment dla Streamlita
- **Render** / **Railway** — bardziej elastyczne, darmowy tier z ograniczeniami
- Własny serwer + Docker — gdy chcesz pełnej kontroli

---

## Kryteria oceny — 30 pkt

| Element | Punkty |
|---|---|
| Pozyskanie danych z prawdziwego źródła | **3 pkt** |
| Czyszczenie i przygotowanie danych (widać myśl, nie tylko `dropna()`) | **4 pkt** |
| Analiza / EDA / wykrycie wzorców w danych | **4 pkt** |
| Różnorodność i jakość wizualizacji (min 5 typów) | **6 pkt** |
| Działające filtry i widgety (min 3) — aplikacja reaguje na zmiany | **4 pkt** |
| Layout, estetyka, UX (sidebar, kolumny, zakładki) | **4 pkt** |
| Działający publiczny deployment | **3 pkt** |
| README z opisem projektu | **2 pkt** |
| **TOTAL** | **30 pkt** |

### Co zwiększa szanse na maksa punktów

- **Spójna narracja** — aplikacja prowadzi użytkownika przez historię, a nie pokazuje przypadkowy zlepek wykresów
- **Cachowanie** — duże operacje (wczytywanie, requesty API) opakowane w `@st.cache_data`
- **Kod podzielony na funkcje** — `app.py` jako orkiestrator, logika w osobnych modułach
- **Komentarze biznesowe** — pod wykresem opisz krótko, co z niego wynika
- **Dbałość o detale** — sensowne tytuły, formatowanie liczb (`1 234 567 zł` zamiast `1234567.0`), poprawna polska typografia

### Czego nie akceptuję

- Aplikacja działa lokalnie, ale link nie odpala albo wyrzuca błąd
- „Aplikacja" to jeden wykres na pełnej stronie
- Dane są całkowicie losowe / wygenerowane bez przyczyny
- Wykresy bez tytułów, etykiet osi, z domyślnymi nazwami kolumn typu `Unnamed: 0`
- Kod skopiowany 1:1 z tutoriala / przykładów z zajęć bez żadnej własnej myśli
- Brak README albo README typu „mój projekt"

---

## Harmonogram pracy — sugestia

Projekt zrobiony w jeden weekend wygląda jak zrobiony w jeden weekend. Polecam rozłożyć go na 3–4 sesje:

| Sesja | Czas | Co robisz |
|---|---|---|
| 1 | ~2h | Wybór tematu, eksploracja źródła danych, pobranie pierwszej próbki |
| 2 | ~3h | Czyszczenie, EDA w notebooku, prototypy wykresów |
| 3 | ~3h | Składanie aplikacji w Streamlit, layout, widgety |
| 4 | ~2h | Deployment, README, testy, kosmetyka |

Razem ~10h. Nie jest dużo, ale potrafi rozjechać się na trzy dni, jeśli zaczynasz wieczorem 11 lipca.

---

## Konsultacje i pytania

Jeśli utykasz, masz wątpliwości co do tematu albo coś nie chce zadziałać — pisz na maila albo łapaj mnie na zajęciach. Lepiej dopytać tydzień przed deadlinem niż w nocy.

Powodzenia! 🚀
