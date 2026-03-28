# Projekt końcowy

**Forma:** indywidualny | **Punktacja:** 52 pkt | **Termin:** *2026-06-06*

---

## Temat

Student **samodzielnie wybiera temat** – aplikacja musi być nietrywalna i możliwa do zademonstrowania w przeglądarce. Temat wymaga zatwierdzenia przez prowadzącego.

| Pomysł | Opis |
|---|---|
| Wyszukiwarka filmów / książek | Integracja z publicznym API (OMDB, OpenLibrary) |
| Aplikacja pogodowa | Pobieranie danych z API, reaktywny UI |
| Quiz / fiszki | Pytania, zliczanie punktów, statystyki postępów |
| Planer zadań | Kanban / Todo z filtrowaniem i priorytetami |
| Kalkulator branżowy | Kalkulator kredytowy, przelicznik walut, makro/mikro składniki |
| Blog / portfolio | Statyczna strona z treścią, nawigacją i filtrowaniem |

> 💡 Unikaj prostych kalkulatorów i list Todo bez dodatkowej logiki – są zbyt trywialne.

---

## Wymagania

**Framework** – Vue 3 *(preferowany)* lub React, min. 3 komponenty, Composition API / hooki. 

Projekt bootstrapujesz przez **Vite** – narzędzie budujące projekt (odpowiednik kompilatora), które obsługuje pliki `.vue` i uruchamia lokalny serwer deweloperski. Vite nie jest frameworkiem – to fundament pod Vue lub React (`npm create vue@latest` / `npm create vite@latest`).

- **Funkcjonalność** – reaktywny stan, renderowanie warunkowe i listowe, obsługa zdarzeń, komunikacja między komponentami (props/emit).
- **SEO** – meta tagi (`title`, `description`, Open Graph), semantyczny HTML.
- **Testy** – min. 3 testy jednostkowe (Vitest / Jest) pokrywające logikę aplikacji, działają przez `npm run test`.
- **Repozytorium** – publiczne repo na GitHub/GitLab, sensowna historia commitów, `README.md` z instrukcją uruchomienia.
- **Demo** – aplikacja publicznie dostępna (Vercel / Netlify / GitHub Pages).

---

## Plan projektu

Krótki dokument (MD lub PDF) zawierający. 

Plan należy przygotować na następne zajęcia (2026-05-09).

1. Temat i nazwa aplikacji
2. Opis funkcjonalności (3–6 punktów)
3. Grupa docelowa
4. Stos technologiczny (framework, biblioteki, API)
5. Wstępna lista komponentów
6. Makieta / szkic UI (odręczny lub Figma / Excalidraw)
7. Plan testów – co będzie testowane
8. Link do założonego repozytorium

---

## Punktacja

| Kryterium | Pkt |
|---|---|
| Plan projektu | 5 |
| Framework (komponenty, reaktywność, komunikacja) | 14 |
| Jakość kodu i struktura | 8 |
| Testy | 8 |
| Demo | 5 |
| SEO | 4 |
| Repozytorium Git | 4 |
| Raport | 4 |
| **Łącznie** | **52** |

> ⚠️ Plagiat lub nieopisany cudzy kod = **0 pkt**.