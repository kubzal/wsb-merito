# Zajęcia 2

## Programowanie obiektowe

Tu pojawi się notatka.

- Dziedziczenie
- Polimorfizm
- Hermetyzacja
- Dunders

---

## Praca projektowa

### Część 2 — Programowanie obiektowe (20 pkt)

**Temat:** Biblioteka — refaktoryzacja do OOP i rola bibliotekarza

Przepisz aplikację z Części 1 na wersję obiektową i rozszerz ją o rolę **bibliotekarza**.

**Wymagania dotyczące klas:**

1. **`Book`** — tytuł, autor, łączna liczba sztuk, liczba dostępnych sztuk.
2. **`User`** (klasa bazowa) — login, hasło, rola. Klasy pochodne: **`Reader`** (posiada listę wypożyczonych książek i listę próśb o przedłużenie) oraz **`Librarian`**.
3. **`Library`** — przechowuje kolekcje książek i użytkowników; zawiera metody realizujące logikę biznesową (wyszukiwanie, wypożyczanie, itp.).

**Nowe funkcjonalności (bibliotekarz):**

4. Po zalogowaniu menu jest różne w zależności od roli.
5. **Bibliotekarz — lista wypożyczeń** — wyświetla wszystkie aktualnie wypożyczone książki wraz z loginami użytkowników, którzy je wypożyczyli.
6. **Czytelnik — prośba o przedłużenie** — czytelnik może wysłać prośbę o przedłużenie wybranej wypożyczonej książki (prośba trafia do kolejki).
7. **Bibliotekarz — obsługa próśb** — bibliotekarz widzi listę próśb o przedłużenie i może każdą zaakceptować lub odrzucić.

**Wymagania techniczne:**

- Zastosowanie dziedziczenia (`Reader`, `Librarian` dziedziczą po `User`).
- Zastosowanie hermetyzacji — pola prywatne/chronione tam, gdzie to sensowne, dostęp przez metody lub properties.
- Metoda `__str__` w co najmniej jednej klasie.
- Dane początkowe tworzone jako instancje klas.

**Punktacja (20 pkt):**

| Element | Punkty |
|---|---|
| Klasa `Book` z atrybutami i metodami | 2 |
| Klasa `User` z dziedziczeniem (`Reader`, `Librarian`) | 4 |
| Klasa `Library` z logiką biznesową w metodach | 3 |
| Hermetyzacja i użycie `__str__` | 2 |
| Menu zależne od roli (czytelnik / bibliotekarz) | 2 |
| Widok wypożyczeń dla bibliotekarza (książka + użytkownik) | 3 |
| Prośba o przedłużenie (czytelnik) i jej obsługa (bibliotekarz) | 4 |