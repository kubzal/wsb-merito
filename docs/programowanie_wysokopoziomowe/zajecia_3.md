# Zajęcia 3

## Programowanie funkcyjne

Tu pojawi się notatka.

---

## Praca projektowa

### Część 3 — Programowanie funkcyjne (20 pkt)

**Temat:** Biblioteka — rozszerzenie z wykorzystaniem programowania funkcyjnego

Rozszerz aplikację z Części 2 o nowe funkcjonalności, implementując je w **stylu funkcyjnym**: wyrażenia lambda, funkcje `map`, `filter`, `sorted` z kluczem, list/dict comprehensions, a także funkcje wyższego rzędu (funkcje przyjmujące lub zwracające inne funkcje).

**Nowe funkcjonalności:**

1. **Wyszukiwanie i filtrowanie katalogu** — czytelnik i bibliotekarz mogą filtrować książki po frazie w tytule lub autorze oraz wyświetlić tylko książki aktualnie dostępne (sztuki > 0). Filtrowanie zaimplementuj za pomocą `filter` + `lambda` lub comprehension.
2. **Sortowanie katalogu** — możliwość wyświetlenia książek posortowanych wg tytułu, autora lub liczby dostępnych sztuk. Użyj `sorted` z parametrem `key` jako lambda.
3. **Rezerwacja niedostępnego tytułu** — jeśli książka ma 0 dostępnych sztuk, czytelnik może ją zarezerwować. Przy obsłudze próśb o przedłużenie bibliotekarz widzi informację, czy na daną książkę istnieje rezerwacja.
4. **Statystyki (bibliotekarz)** — nowa opcja w menu bibliotekarza:
   - najpopularniejsza książka (największa różnica między łączną liczbą sztuk a dostępnymi),
   - liczba aktywnych wypożyczeń ogółem,
   - lista czytelników posortowana malejąco wg liczby wypożyczonych książek.
   
   Statystyki zaimplementuj przy użyciu `map`/`filter`/comprehension — bez pętli `for` z ręczną akumulacją.

5. **Funkcja wyższego rzędu** — napisz co najmniej jedną funkcję, która przyjmuje inną funkcję jako argument (np. uniwersalna funkcja do wyświetlania przefiltrowanej kolekcji, przyjmująca predykat filtrowania).

**Wymagania techniczne:**

- W nowych funkcjonalnościach nie stosuj klasycznych pętli `for`/`while` do filtrowania, przekształcania ani sortowania — używaj narzędzi programowania funkcyjnego.
- Co najmniej 3 użycia `lambda`.
- Co najmniej 2 użycia comprehension (list lub dict).
- Co najmniej 1 funkcja wyższego rzędu (przyjmująca funkcję jako argument).

**Punktacja (20 pkt):**

| Element | Punkty |
|---|---|
| Filtrowanie katalogu (`filter`/`lambda`/comprehension) | 3 |
| Sortowanie katalogu (`sorted` + `key` jako lambda) | 2 |
| Rezerwacja niedostępnego tytułu | 3 |
| Info o rezerwacji przy obsłudze próśb o przedłużenie | 2 |
| Statystyki bibliotekarza w stylu funkcyjnym | 4 |
| Funkcja wyższego rzędu | 3 |
| Spełnienie wymagań technicznych (min. 3× lambda, 2× comprehension, brak pętli w nowej logice) | 3 |

