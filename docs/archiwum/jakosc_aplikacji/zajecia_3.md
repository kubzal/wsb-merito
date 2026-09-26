# Zajęcia 3

## Powtórka z zajęć 1 i 2

Do tej pory patrzyliśmy na jakość „od środka":

- **Zajęcia 1** — testy manualne, piramida testów, pisanie testów jednostkowych w `pytest`.
- **Zajęcia 2** — jakość samego kodu (PEP 8, linter, formatter, typy) i automatyzacja (pre-commit, CI/CD, coverage).

To wszystko sprawdza, czy kod **działa zgodnie ze specyfikacją** i czy jest **dobrze napisany**. Ale można mieć projekt z zielonym CI, 100% coverage i kodem czystym jak łza — który dla użytkownika jest koszmarem. Bo specyfikacja była zła, albo formularz da się wysłać tylko myszką, albo „Zapisz" wygląda jak „Anuluj".

Dziś patrzymy na jakość z **dwóch perspektyw, których nie widać w testach jednostkowych**:

- **perspektywa użytkownika** — czy aplikacja jest użyteczna, dostępna i szybka *dla człowieka*,
- **perspektywa procesu** — czy *sposób*, w jaki pracujemy, w ogóle prowadzi do jakości (kryteria akceptacji, Definition of Done, cykl życia buga, metryki).

Hasło na dziś: **„it works on my machine" to nie jest jakość.**

---

## Część 1 — Jakość z perspektywy użytkownika

### UX vs UI — to nie to samo

Dwa pojęcia, które notorycznie się myli:

- **UI (User Interface)** — *jak to wygląda*. Kolory, przyciski, czcionki, layout. Warstwa wizualna.
- **UX (User Experience)** — *jak się tego używa*. Czy użytkownik dociera do celu szybko i bez frustracji. Cały proces, nie tylko obrazek.

> Analogia: UI to wygląd pilota do telewizora. UX to czy faktycznie da się nim ściszyć dźwięk bez czytania instrukcji. Można mieć piękny pilot z 80 przyciskami i fatalny UX.

Aplikacja może mieć śliczny UI i tragiczny UX (ładny, ale nie wiadomo, gdzie kliknąć) — albo brzydki UI i niezły UX (proste i działa).

### Użyteczność (usability) — 10 heurystyk Nielsena

**Heurystyki Nielsena** to 10 ogólnych zasad projektowania użytecznych interfejsów. Nie są to sztywne reguły, tylko „dobre nawyki", po których ocenia się interfejs (tzw. **ewaluacja heurystyczna** — tester ocenia ekran pod kątem każdej zasady).

| # | Heurystyka | O co chodzi |
|---|------------|-------------|
| 1 | Widoczność statusu systemu | Użytkownik zawsze wie, co się dzieje (spinner, „Zapisano", pasek postępu) |
| 2 | Zgodność z rzeczywistością | Język i ikony znane użytkownikowi, nie żargon programisty |
| 3 | Kontrola i wolność użytkownika | Łatwe „cofnij", wyjście awaryjne, anulowanie operacji |
| 4 | Spójność i standardy | Te same rzeczy wyglądają i działają tak samo w całej aplikacji |
| 5 | Zapobieganie błędom | Lepiej nie dopuścić do błędu, niż ładnie go pokazać |
| 6 | Rozpoznawanie zamiast przypominania | Opcje widoczne, użytkownik nie musi nic pamiętać |
| 7 | Elastyczność i wydajność | Skróty dla zaawansowanych, prostota dla nowych |
| 8 | Estetyka i minimalizm | Nie zaśmiecaj ekranu nieistotnymi informacjami |
| 9 | Pomoc w rozpoznaniu i naprawie błędów | Komunikat błędu mówi *co* i *jak naprawić* (nie „Error 0x8004") |
| 10 | Pomoc i dokumentacja | Dostępna, gdy potrzebna, łatwa do przeszukania |

> Przykład naruszenia #9: komunikat „Wystąpił błąd" to porażka. Dobry komunikat: „Hasło musi mieć min. 8 znaków i jedną cyfrę".

### Testy użyteczności (usability testing)

To **obserwacja prawdziwych użytkowników** wykonujących realne zadania w aplikacji. Nie pytamy „czy podoba ci się ten przycisk" — dajemy zadanie („zamów produkt i opłać") i **patrzymy, gdzie się gubią**.

Kluczowe zasady:

- Dajemy **zadanie**, nie instrukcję krok po kroku.
- **Nie podpowiadamy** — milczymy, nawet gdy boli.
- Prosimy o **myślenie na głos** (think-aloud) — „teraz szukam koszyka... gdzie to jest...".
- Wystarczy **5 użytkowników**, żeby wyłapać większość problemów (zasada Nielsena — pierwszych 5 testerów znajduje ~85% problemów).

### Dostępność (accessibility, a11y)

**Dostępność** to projektowanie tak, żeby z aplikacji mogły korzystać **osoby z niepełnosprawnościami** — niewidome (czytniki ekranu), słabowidzące, niesłyszące, z ograniczeniami ruchowymi (tylko klawiatura).

> Skrót `a11y` = „accessibility" (a + 11 liter + y). Tak samo jak `i18n` = internationalization.

Standardem są **WCAG** (Web Content Accessibility Guidelines). Cztery filary — zasada **POUR**:

- **Perceivable** (postrzegalny) — treść da się odebrać (tekst alternatywny do obrazków, napisy do wideo),
- **Operable** (obsługiwalny) — da się obsłużyć z klawiatury, bez myszy,
- **Understandable** (zrozumiały) — przewidywalny, czytelny język,
- **Robust** (solidny) — działa z różnymi technologiami wspomagającymi.

Najczęstsze grzechy dostępności (i jak je sprawdzić):

| Problem | Jak sprawdzić |
|---------|---------------|
| Brak `alt` przy obrazkach | Czytnik ekranu nie wie, co jest na obrazku |
| Za niski kontrast tekstu | WCAG wymaga kontrastu min. **4.5:1** dla zwykłego tekstu |
| Nie da się nawigować klawiszem Tab | Spróbuj przejść cały formularz bez myszy |
| Brak etykiet (`label`) przy polach formularza | Czytnik czyta „pole tekstowe", nie wie czego dotyczy |
| Informacja przekazana tylko kolorem | Daltonista nie odróżni „czerwone = błąd" |

### Lighthouse — automatyczny audyt strony

**Lighthouse** to wbudowane w Chrome narzędzie (zakładka **Lighthouse** w DevTools, F12), które automatycznie audytuje stronę w kilku kategoriach i wystawia ocenę **0–100**:

- **Performance** — jak szybko strona się ładuje i staje się interaktywna,
- **Accessibility** — automatyczne sprawdzenie części reguł dostępności,
- **Best Practices** — bezpieczeństwo, poprawne API przeglądarki,
- **SEO** — czy strona jest przyjazna wyszukiwarkom.

> **Uwaga:** Lighthouse łapie tylko część problemów dostępności (np. kontrast, brak `alt`). Ocena 100/100 w Accessibility **nie znaczy**, że strona jest w pełni dostępna — tak jak 100% coverage nie znaczy „brak bugów". Audyt automatyczny + ręczny test (klawiatura, czytnik) = dopiero komplet.

Można też uruchomić z linii poleceń:

```bash
npm install -g lighthouse
lighthouse https://example.com --view
```

### Core Web Vitals — mierzalne wrażenie z ładowania

Google zdefiniował zestaw metryk opisujących **realne odczucie szybkości** (Lighthouse je raportuje):

- **LCP** (Largest Contentful Paint) — kiedy załaduje się największy element. Dobrze: **< 2.5 s**.
- **CLS** (Cumulative Layout Shift) — jak bardzo „skacze" layout podczas ładowania. Dobrze: **< 0.1**.
- **INP** (Interaction to Next Paint) — jak szybko strona reaguje na kliknięcie. Dobrze: **< 200 ms**.

> CLS znacie z życia: chcesz kliknąć przycisk, w ostatniej chwili doładowuje się baner i klikasz reklamę. To wysoki CLS.

---

## Część 2 — Jakość z perspektywy procesu

Tu nie chodzi o to, *co* testujemy, tylko *jak zorganizowana jest praca*, żeby jakość w ogóle powstawała.

### Shift-left testing — testuj jak najwcześniej

**Shift-left** to przesunięcie testowania „w lewo" na osi czasu projektu — czyli **jak najbliżej początku**, a nie dopiero na końcu przed wdrożeniem.

Dlaczego? **Koszt naprawy błędu rośnie wykładniczo** im później go znajdziemy:

```
Wymagania → Projekt → Kod → Test → Produkcja
   1×         3×       10×    30×      100×     ← orientacyjny koszt naprawy
```

Bug w wymaganiach poprawiamy zdaniem w dokumencie. Ten sam bug znaleziony na produkcji = hotfix, przeprosiny i nieprzespana noc. To ta sama logika co piramida testów i CI z poprzednich zajęć — **wykrywaj wcześnie**.

### Kryteria akceptacji i Definition of Done

Dwa pojęcia, które łatwo pomylić:

- **Kryteria akceptacji (Acceptance Criteria)** — warunki, które musi spełnić **konkretne zadanie/funkcja**, żeby uznać je za zrobione. Specyficzne dla zadania.
- **Definition of Done (DoD)** — wspólna lista warunków dla **każdego** zadania w zespole. Uniwersalna.

Przykładowa **Definition of Done** zespołu:

- [ ] kod przechodzi `ruff check` i `ruff format --check` (z zajęć 2!),
- [ ] napisane testy jednostkowe, CI zielone,
- [ ] code review zaakceptowane przez min. 1 osobę,
- [ ] funkcja przetestowana manualnie na środowisku testowym,
- [ ] dokumentacja zaktualizowana.

> DoD to taki „pre-commit hook dla całego zadania" — dopóki nie spełnisz wszystkich punktów, zadanie nie jest *Done*, tylko *prawie done* (czyli niezrobione).

### Gherkin — kryteria akceptacji w formacie Given-When-Then

**Gherkin** to ustrukturyzowany sposób zapisu scenariuszy, czytelny zarówno dla biznesu, jak i programisty. Schemat **Given-When-Then**:

- **Given** (zakładając) — stan początkowy,
- **When** (kiedy) — akcja użytkownika,
- **Then** (wtedy) — oczekiwany rezultat.

```gherkin
Funkcja: Logowanie do aplikacji

  Scenariusz: Poprawne logowanie
    Zakładając, że jestem na stronie logowania
    Oraz mam konto o adresie "jan@example.com"
    Kiedy wpiszę poprawny email i hasło
    I kliknę "Zaloguj"
    Wtedy zobaczę pulpit użytkownika

  Scenariusz: Błędne hasło
    Zakładając, że jestem na stronie logowania
    Kiedy wpiszę poprawny email i błędne hasło
    I kliknę "Zaloguj"
    Wtedy zobaczę komunikat "Nieprawidłowy email lub hasło"
    Oraz pozostanę na stronie logowania
```

Ten sam zapis to fundament **BDD (Behavior-Driven Development)** — podejścia, w którym scenariusze pisze się *przed* kodem, wspólnie z biznesem. W Pythonie realizuje to np. biblioteka `behave`, a scenariusze Gherkin stają się wykonywalnymi testami.

### Cykl życia buga (bug lifecycle)

Błąd nie żyje w próżni — przechodzi przez statusy. Typowy cykl:

```
New → Assigned → In Progress → Fixed → Retest → Closed
                                  │
                                  └──(błąd wraca)──► Reopened
```

- **New** — zgłoszony, jeszcze nieprzejrzany.
- **Assigned** — przypisany do programisty.
- **In Progress / Fixed** — naprawiany / naprawiony.
- **Retest** — tester sprawdza poprawkę.
- **Closed** — potwierdzone, że działa.
- **Reopened** — poprawka nie zadziałała, bug wraca do gry.

Inne częste statusy: **Rejected** (to nie błąd / działa zgodnie z założeniem), **Duplicate** (już zgłoszony), **Won't Fix** (świadomie nie naprawiamy).

### Severity vs Priority — to dwie różne osie

Najczęściej mylone pojęcia w QA:

- **Severity (waga / dotkliwość)** — jak bardzo bug szkodzi *technicznie* (czy wywala aplikację?).
- **Priority (priorytet)** — jak szybko trzeba to naprawić *z perspektywy biznesu*.

Nie zawsze idą w parze:

| Sytuacja | Severity | Priority |
|----------|----------|----------|
| Aplikacja wywala się przy starcie | Wysoka | Wysoki |
| Literówka w nazwie firmy na stronie głównej | Niska | **Wysoki** (wstyd!) |
| Crash w funkcji, której nikt nie używa | **Wysoka** | Niski |
| Drobne przesunięcie ikony o 2px | Niska | Niski |

> Klasyk: literówka w logo prezesa — technicznie błahostka (severity niska), ale naprawiasz to *natychmiast* (priority wysoki).

### Rodzaje testów na poziomie procesu

- **Smoke test** — szybki test „czy w ogóle się pali" — czy najważniejsze funkcje (login, główny ekran) działają. Robi się po każdym deployu, zanim w ogóle warto testować dalej.
- **Sanity test** — wąskie sprawdzenie konkretnej poprawki, czy zadziałała.
- **Regression test** — sprawdzenie, czy nowa zmiana **nie zepsuła** tego, co wcześniej działało. To główny kandydat do automatyzacji (nikt nie chce klikać tego samego ręcznie co tydzień).

### Metryki jakości

To, czego nie mierzysz, trudno poprawić. Kilka praktycznych metryk:

- **Defect density** — liczba bugów na 1000 linii kodu (lub na moduł). Pokazuje, gdzie jest najgoręcej.
- **Defect escape rate** — ile bugów „uciekło" na produkcję zamiast zostać złapanych w testach. Im mniej, tym lepszy proces QA.
- **MTTR** (Mean Time To Repair) — średni czas od zgłoszenia do naprawy.
- **Lead time** — czas od pomysłu do wdrożenia.
- **Test coverage** — znane z zajęć 2 (ale to metryka *kodu*, nie jakości całego produktu).

> Ostrożnie z metrykami: jeśli rozliczasz testerów z „liczby znalezionych bugów", nagle wszyscy zaczną zgłaszać literówki. Metryka, która staje się celem, przestaje być dobrą metryką (prawo Goodharta).

### Retrospektywa — jakość procesu, nie tylko produktu

Po każdym sprincie zespół robi **retrospektywę** — krótkie spotkanie, na którym odpowiada na trzy pytania:

1. Co poszło dobrze?
2. Co poszło źle?
3. Co zmieniamy na następny raz?

To „testowanie jednostkowe procesu" — szukamy bugów nie w kodzie, tylko w *sposobie pracy*. Dobry zespół z każdej retro wynosi 1–2 konkretne zmiany, a nie listę pretensji.

---

## Podsumowanie najważniejszych pojęć i narzędzi

| Pojęcie / Narzędzie | Perspektywa | Po co |
|---------------------|-------------|-------|
| Heurystyki Nielsena | Użytkownik | Szybka ocena użyteczności interfejsu |
| WCAG / a11y | Użytkownik | Dostępność dla osób z niepełnosprawnościami |
| Lighthouse | Użytkownik | Automatyczny audyt: performance, a11y, SEO, best practices |
| Core Web Vitals (LCP, CLS, INP) | Użytkownik | Mierzalne odczucie szybkości strony |
| Shift-left | Proces | Testuj wcześnie — taniej naprawiać |
| Definition of Done | Proces | Wspólna definicja „skończone" dla zespołu |
| Gherkin / BDD | Proces | Czytelne kryteria akceptacji i testy |
| Cykl życia buga | Proces | Wspólny język statusów błędu |
| Severity vs Priority | Proces | Rozdzielenie „jak groźne" od „jak pilne" |
| Smoke / regression | Proces | Co i kiedy testować po zmianie |
| Metryki jakości | Proces | Mierzyć, żeby poprawiać |

---

## Zadanie praktyczne (20 pkt)

Za każdą z części do uzyskania jest 10 pkt jeżeli będzie wykonana w trakcie zajęć lub 8 pkt jeżeli będzie dokończona w domu.

### Część 1 — Audyt UX i dostępności (10 pkt)

Twoim zadaniem jest przeprowadzenie **audytu jakości z perspektywy użytkownika** dla wybranej strony internetowej.

**Wybór strony:** możesz użyć dowolnej **publicznej** strony (np. strona uczelni, sklep internetowy, serwis informacyjny) **lub** własnego projektu z poprzednich zajęć. Unikaj stron, które wymagają logowania.

#### Krok 1 — Audyt Lighthouse

1. Otwórz stronę w Chrome, naciśnij **F12** i przejdź do zakładki **Lighthouse**.
2. Zaznacz wszystkie kategorie (Performance, Accessibility, Best Practices, SEO), tryb **Navigation**, urządzenie **Desktop** lub **Mobile** (wybierz jedno i się go trzymaj).
3. Kliknij **Analyze page load** i poczekaj na raport.
4. Zrób **screenshot** czterech ocen (0–100) oraz sekcji **Accessibility** z listą problemów.

#### Krok 2 — Interpretacja wyników

W raporcie opisz:

- Cztery oceny liczbowe i krótki komentarz, która kategoria wypadła najgorzej.
- **Minimum 3 konkretne problemy** zgłoszone przez Lighthouse (np. „za niski kontrast", „obrazki bez `alt`", „LCP = 4.2 s").
- Dla każdego problemu — **proponowaną poprawkę**.

#### Krok 3 — Ręczny test dostępności (czego Lighthouse nie złapie)

Wykonaj **2 testy ręczne** i opisz wynik:

1. **Nawigacja klawiaturą** — przejdź przez stronę używając tylko klawisza **Tab** (i Enter). Czy da się dotrzeć do wszystkich linków i przycisków? Czy widać, który element jest aktualnie zaznaczony (focus)?
2. **Test kontrastu / koloru** — znajdź miejsce, gdzie informacja jest przekazana **tylko kolorem**, albo tekst o niskim kontraście. Jeśli nie ma — napisz, że strona zdała.

#### Krok 4 — Ewaluacja heurystyczna

Oceń stronę według **minimum 5 heurystyk Nielsena** (z notatki). Wypełnij tabelę:

| # | Heurystyka | Ocena (✅ / ⚠️ / ❌) | Komentarz / przykład ze strony |
|---|------------|--------------------|--------------------------------|
| 1 | | | |
| 2 | | | |
| 3 | | | |
| 4 | | | |
| 5 | | | |

#### Co oddajesz?

Raport w formacie **PDF** zawierający:

- nazwę i URL badanej strony,
- screenshoty z Lighthouse (oceny + sekcja Accessibility),
- interpretację wyników (min. 3 problemy + poprawki),
- wyniki 2 testów ręcznych dostępności,
- tabelę ewaluacji heurystycznej (min. 5 heurystyk),
- **podsumowanie** — 3–5 zdań: czy strona jest jakościowa z perspektywy użytkownika i co poprawiłbyś w pierwszej kolejności.

---

### Część 2 — Kryteria akceptacji i test E2E (10 pkt)

W tej części łączysz **perspektywę procesu** (kryteria akceptacji w Gherkin) z **automatyzacją testu użytkownika** (E2E w Playwright). Wracamy do szczytu piramidy testów z zajęć 1 — ale tym razem piszemy te testy sami.

Testować będziemy publiczną stronę-piaskownicę **The Internet** (Heroku) — przygotowaną specjalnie do nauki testów E2E:

- **Strona logowania:** `https://the-internet.herokuapp.com/login`
- Poprawne dane: login `tomsmith`, hasło `SuperSecretPassword!`
- Po poprawnym logowaniu pojawia się komunikat o sukcesie i strona `/secure`.
- Po błędnym logowaniu pojawia się komunikat o błędzie.

> Jeśli wolisz, możesz przetestować własną aplikację z poprzednich zajęć — wystarczy, że ma formularz z walidacją.

#### Krok 1 — Napisz kryteria akceptacji w Gherkin

Dla funkcji logowania napisz **minimum 3 scenariusze** w formacie Given-When-Then (Zakładając–Kiedy–Wtedy). Muszą obejmować:

- 1 scenariusz **pozytywny** (poprawne logowanie),
- minimum 2 scenariusze **negatywne** (błędne hasło, błędny login / puste pola).

Zapisz je w pliku `kryteria_akceptacji.feature` (czysty Gherkin, jak w notatce).

#### Krok 2 — Zainstaluj Playwright

```bash
pip install pytest-playwright
playwright install chromium
```

#### Krok 3 — Zaimplementuj testy E2E

W pliku `test_login_e2e.py` napisz **testy automatyzujące Twoje scenariusze z Kroku 1**. Szkielet na start:

```python
import re
from playwright.sync_api import Page, expect

LOGIN_URL = "https://the-internet.herokuapp.com/login"


def test_poprawne_logowanie(page: Page):
    # Arrange
    page.goto(LOGIN_URL)
    # Act
    page.fill("#username", "tomsmith")
    page.fill("#password", "SuperSecretPassword!")
    page.click("button[type='submit']")
    # Assert
    expect(page.locator("#flash")).to_contain_text("You logged into a secure area")
    expect(page).to_have_url(re.compile(r"/secure"))


def test_bledne_haslo(page: Page):
    page.goto(LOGIN_URL)
    page.fill("#username", "tomsmith")
    page.fill("#password", "zle_haslo")
    page.click("button[type='submit']")
    expect(page.locator("#flash")).to_contain_text("Your password is invalid!")


def test_bledny_login(page: Page):
    page.goto(LOGIN_URL)
    page.fill("#username", "nieistnieje")
    page.fill("#password", "SuperSecretPassword!")
    page.click("button[type='submit']")
    expect(page.locator("#flash")).to_contain_text("Your username is invalid!")
```

Wymagania:

- **Minimum 3 testy**, po jednym na każdy scenariusz z Kroku 1.
- Każdy test zgodny ze strukturą **AAA** (Arrange–Act–Assert) — z zajęć 1.
- Dodaj **przynajmniej jeden test parametryzowany** (`@pytest.mark.parametrize`) — np. kilka wariantów błędnych danych logowania w jednym teście.

#### Krok 4 — Uruchom testy

```bash
# Tryb headless (bez okna przeglądarki)
pytest -v

# Tryb headed — zobaczysz, jak Playwright klika za Ciebie
pytest -v --headed
```

Uruchom przynajmniej raz w trybie `--headed`, żeby zobaczyć test E2E w akcji. Zrób screenshot z wynikiem `pytest -v` (wszystkie testy zielone).

#### Co oddajesz?

Spakowany folder `zadanie_3.zip` zawierający:

- plik `kryteria_akceptacji.feature` (min. 3 scenariusze Gherkin),
- plik `test_login_e2e.py` (min. 3 testy + 1 parametryzowany),
- screenshot z wynikiem `pytest -v` (wszystkie testy zielone),
- krótki plik `README.md` (3–5 zdań): które kryterium akceptacji odpowiada któremu testowi.

---

## Podsumowanie zajęć 3

| Temat | Kluczowy wniosek |
|-------|-------------------|
| UX vs UI | Ładny wygląd to nie to samo co dobre doświadczenie |
| Heurystyki Nielsena | 10 zasad do szybkiej oceny użyteczności |
| Dostępność (a11y / WCAG) | Aplikacja musi działać też bez myszy i dla czytnika ekranu |
| Lighthouse | Automatyczny audyt — ale nie zastępuje testu ręcznego |
| Shift-left | Im wcześniej znajdziesz błąd, tym taniej go naprawisz |
| Definition of Done | Wspólna definicja „skończone" dla całego zespołu |
| Gherkin / BDD | Kryteria akceptacji czytelne dla biznesu i wykonywalne jako testy |
| Severity vs Priority | „Jak groźne" to nie to samo co „jak pilne" |
| Cykl życia buga | Błąd ma statusy — od New do Closed (czasem Reopened) |
| E2E (Playwright) | Szczyt piramidy testów — symulacja prawdziwego użytkownika |