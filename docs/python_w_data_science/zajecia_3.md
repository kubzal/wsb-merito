# Zajęcia 3

## Podstawy NLP i biblioteka spaCy

Na wykładzie pokazaliśmy, jak za pomocą **HuggingFace `pipeline`** w kilku linijkach zrobić klasyfikację sentymentu, NER, question answering czy generowanie tekstu. To jest "wysoki poziom" — bierzemy gotowy model i wrzucamy do niego tekst.

Dziś schodzimy o piętro niżej. Zobaczymy, **co tak naprawdę dzieje się z tekstem**, zanim model cokolwiek zaklasyfikuje, i poznamy **spaCy** — bibliotekę, która do dziś jest standardem produkcyjnym do "klasycznego" NLP.

Po dzisiejszych zajęciach będziesz w stanie:

- wymienić podstawowe etapy przetwarzania tekstu (tokenizacja, lematyzacja, POS, NER),
- użyć spaCy do analizy lingwistycznej tekstu w języku angielskim i polskim,
- zwizualizować zależności składniowe i encje nazwane (`displacy`),
- policzyć podobieństwo dwóch dokumentów na bazie wektorów,
- świadomie wybrać między spaCy a transformerem — wiedząc, co dostajesz i czym płacisz.

---

## Co to NLP?

**Natural Language Processing** to dziedzina zajmująca się przetwarzaniem języka naturalnego przez komputer. Typowe zadania:

| Zadanie | Przykład |
|---|---|
| Klasyfikacja tekstu | sentyment recenzji, kategoria newsa, spam/nie-spam |
| Named Entity Recognition (NER) | wyciągnięcie z tekstu osób, firm, miejsc, dat |
| Tagowanie części mowy (POS) | który token to rzeczownik, czasownik, przymiotnik |
| Lematyzacja / stemming | sprowadzenie słowa do formy podstawowej (`biegałem` → `biec`) |
| Question Answering | znalezienie odpowiedzi na pytanie w tekście |
| Tłumaczenie maszynowe | EN → PL |
| Generowanie tekstu | dokończ zdanie, napisz streszczenie |
| Podobieństwo semantyczne | jak bardzo dwa zdania znaczą to samo |

Pierwsze cztery to "klasyczny" NLP — i to są zadania, w których spaCy świeci. Reszta to świat transformerów, z którymi pracowaliśmy na wykładzie.

---

## Tekst nie istnieje dla komputera

Komputer nie umie "czytać". Zanim dotrzemy do jakiegokolwiek modelu, surowy tekst musi przejść przez kilka etapów:

```
"Apple is looking at buying U.K. startup for $1 billion."
        │
        ▼
1.  Tokenizacja          → ["Apple", "is", "looking", "at", "buying", "U.K.", "startup", "for", "$", "1", "billion", "."]
2.  Lematyzacja          → ["apple", "be", "look", "at", "buy", "u.k.", "startup", "for", "$", "1", "billion", "."]
3.  POS tagging          → [PROPN, AUX, VERB, ADP, VERB, PROPN, NOUN, ADP, SYM, NUM, NUM, PUNCT]
4.  Dependency parsing   → who does what to whom
5.  NER                  → "Apple" → ORG, "U.K." → GPE, "$1 billion" → MONEY
```

To wszystko spaCy zrobi za nas w **jednym wywołaniu** — i to jest sedno tego, co nazywamy *language pipeline*.

---

## spaCy — instalacja i pierwszy doc

```bash
pip install spacy
python -m spacy download en_core_web_sm
python -m spacy download pl_core_news_sm   # polski też mamy
```

```python
import spacy

nlp = spacy.load("en_core_web_sm")

doc = nlp("Apple is looking at buying U.K. startup for $1 billion.")

print(type(doc))         # <class 'spacy.tokens.doc.Doc'>
print(len(doc))          # 12 — liczba tokenów
print(doc[0])            # Apple
print(doc[0:3])          # Apple is looking — slice działa jak na liście
```

Obiekt `nlp` to **pipeline** — kiedy wywołujesz `nlp(tekst)`, wewnętrznie wykonują się po kolei: tokenizer → tagger → parser → NER → ... — i wynikiem jest gotowy `Doc` z wszystkimi adnotacjami.

### Tokenizacja

Tokenizacja w spaCy nie jest zwykłym `text.split()`. Biblioteka rozumie kontrakcje, kropki w skrótach, znaki interpunkcyjne, waluty:

```python
doc = nlp("Don't look at U.K. — it costs $1.5M!")

for token in doc:
    print(f"{token.text:<10} is_punct={token.is_punct}  is_alpha={token.is_alpha}")
```

```
Don        is_punct=False  is_alpha=True
't         is_punct=False  is_alpha=True   # kontrakcja rozbita na "Do" + "n't"
look       is_punct=False  is_alpha=True
at         is_punct=False  is_alpha=True
U.K.       is_punct=False  is_alpha=False  # skrót zachowany w całości
—          is_punct=True   is_alpha=False
...
```

### Lematyzacja

Lematyzacja sprowadza słowo do **formy słownikowej**. Inaczej niż prymitywny stemming (który po prostu obcina końcówki), lematyzacja zna gramatykę:

```python
doc = nlp("She was running and the children were better than yesterday.")

for token in doc:
    print(f"{token.text:<10} -> {token.lemma_}")
```

```
She        -> she
was        -> be
running    -> run
and        -> and
the        -> the
children   -> child         # nieregularna liczba mnoga
were       -> be
better     -> well          # stopień wyższy → forma podstawowa
than       -> than
yesterday  -> yesterday
.          -> .
```

To kluczowe, kiedy liczymy częstotliwości słów albo budujemy wyszukiwarkę — chcesz, żeby `bieganie`, `biegałem`, `biegnij` trafiły do tego samego kubełka `biec`.

### POS tagging i części mowy

```python
for token in doc:
    print(f"{token.text:<10} {token.pos_:<8} {token.tag_}")
```

`pos_` to **uniwersalna** część mowy (`NOUN`, `VERB`, `ADJ`, ...), `tag_` to bardziej szczegółowy tag specyficzny dla języka.

Typowe użycie — wyciągnięcie z tekstu wszystkich rzeczowników:

```python
nouns = [token.lemma_ for token in doc if token.pos_ == "NOUN"]
```

### Named Entity Recognition

Encje nazwane w spaCy dostajesz przez `doc.ents`:

```python
doc = nlp(
    "Apple is looking at buying U.K. startup for $1 billion. "
    "The deal is expected to close in 2026."
)

for ent in doc.ents:
    print(f"{ent.text:<15} {ent.label_}")
```

```
Apple           ORG
U.K.            GPE
$1 billion      MONEY
2026            DATE
```

Etykiety w `en_core_web_sm`: `PERSON`, `ORG`, `GPE` (kraje, miasta), `LOC` (geografia), `DATE`, `TIME`, `MONEY`, `PERCENT`, `PRODUCT` i kilka innych.

### Wizualizacja w `displacy`

Jeden z najfajniejszych elementów spaCy — wbudowana wizualizacja w Jupyterze:

```python
from spacy import displacy

doc = nlp("Marie Curie was born in Warsaw and worked at the Sorbonne in Paris.")

displacy.render(doc, style="ent", jupyter=True)         # encje w tekście
displacy.render(doc, style="dep", jupyter=True)         # drzewo zależności
```

`style="ent"` pokoloruje encje w tekście (jak na wykładzie, ale bez ręcznego budowania HTML-a). `style="dep"` narysuje strzałki zależności składniowych — co od czego zależy w zdaniu.

### Wektory i podobieństwo

Modele `_md` (medium) i `_lg` (large) zawierają **wektory słów**. Każdy token ma reprezentację numeryczną w wielowymiarowej przestrzeni, a podobieństwo dwóch dokumentów to po prostu kosinus między ich uśrednionymi wektorami:

```python
nlp = spacy.load("en_core_web_md")   # uwaga — większy model

doc1 = nlp("I like cats and dogs.")
doc2 = nlp("Pets are wonderful companions.")
doc3 = nlp("The stock market crashed yesterday.")

print(doc1.similarity(doc2))   # ~0.85 — bliskie tematycznie
print(doc1.similarity(doc3))   # ~0.45 — luźno
```

To proste, deterministyczne i szybkie. Dla SOTA podobieństwa semantycznego użyłbyś sentence-transformerów (BERT-based) — ale dla 80% zastosowań produkcyjnych spaCy wystarcza.

---

## spaCy vs transformery — kiedy co?

|  | spaCy | HuggingFace transformers |
|---|---|---|
| **Szybkość** | tysiące dokumentów na sekundę na CPU | dziesiątki na CPU, setki na GPU |
| **Pamięć** | model ~50 MB | model 400 MB – kilka GB |
| **Jakość NER** | bardzo dobra (klasyczna) | SOTA, lepsza w trudnych przypadkach |
| **Konfigurowalność pipeline'u** | bardzo wysoka (custom componenty) | trzeba samemu spinać |
| **Klasyfikacja zero-shot, QA, generacja** | brak / słaba | natywne |
| **Deterministyczność** | tak | tylko z `temperature=0` |
| **Prod readiness** | wysoka — typowy wybór do prod | rośnie, ale wymaga ostrożności |

W praktyce te biblioteki **się nie wykluczają**. Jest pakiet `spacy-transformers`, który pozwala wpiąć model BERT-owy jako komponent pipeline'u spaCy — masz wtedy szybką tokenizację i POS ze spaCy oraz kontekstowy embedding z transformera.

**Reguła kciuka:**

- klasyczne zadania (tokenizacja, POS, lematyzacja, NER, składnia) → **spaCy**,
- nowoczesne zadania (zero-shot, QA, generacja, kontekstowe podobieństwo) → **transformery**.

---

## Łączymy wszystko — mini analiza nagłówków

Pokażmy spaCy w działaniu na tych samych nagłówkach, na których na wykładzie pracował zero-shot classifier:

```python
import spacy
from collections import Counter

nlp = spacy.load("en_core_web_sm")

NAGLOWKI = [
    "Government announces new austerity measures amid rising debt.",
    "Scientists discover potential cure for Alzheimer's disease.",
    "Stock markets plunge following central bank interest rate decision.",
    "Climate summit ends without binding agreement on emissions.",
    "Apple unveils new iPhone in Cupertino.",
]

# Przetwarzamy wszystko jednym przebiegiem (nlp.pipe jest szybsze niż pętla po nlp())
docs = list(nlp.pipe(NAGLOWKI))

# 1) Encje per nagłówek
for naglowek, doc in zip(NAGLOWKI, docs):
    encje = [(e.text, e.label_) for e in doc.ents]
    print(f"{naglowek}\n  {encje}\n")

# 2) Najczęstsze rzeczowniki w całym korpusie (po lematyzacji)
rzeczowniki = [
    token.lemma_.lower()
    for doc in docs
    for token in doc
    if token.pos_ == "NOUN"
]
print(Counter(rzeczowniki).most_common(5))

# 3) Najbardziej "rzeczownikowy" nagłówek — heurystyka gęstości tematu
for naglowek, doc in zip(NAGLOWKI, docs):
    n_nouns = sum(1 for t in doc if t.pos_ == "NOUN")
    print(f"{n_nouns}/{len(doc):<3}  {naglowek}")
```

W kilkudziesięciu linijkach mamy **ekstrakcję encji**, **agregację tematów** i **prostą metrykę gęstości informacyjnej** — bez ani jednego wywołania API i bez GPU.

> **`nlp.pipe(teksty)`** — kiedy przetwarzasz wiele dokumentów, zawsze używaj `nlp.pipe()` zamiast pętli po `nlp(tekst)`. Pod spodem batchuje i potrafi być nawet 5–10× szybsze.

---

## Praca projektowa

### Zadanie 3 — News Analyzer (20 pkt)

Zbuduj klasę `NewsAnalyzer`, która opakowuje pipeline spaCy i pozwala analizować kolekcję krótkich tekstów (nagłówki, fragmenty newsów, recenzje — co wolisz). Zadanie łączy **OOP z poprzednich zajęć** z **nową wiedzą o NLP**.

**Dane wejściowe:** lista co najmniej 10 tekstów w języku angielskim. Możesz użyć nagłówków z wykładu, ulubionych cytatów, fragmentów Wikipedii — cokolwiek, byle realne i różnorodne (z encjami, liczbami, datami).

**Klasa `NewsAnalyzer`:**

- konstruktor przyjmuje listę tekstów oraz nazwę modelu spaCy (domyślnie `"en_core_web_sm"`)
- konstruktor ładuje model, przetwarza wszystkie teksty przez `nlp.pipe()` i zapisuje listę `Doc`-ów jako atrybut
- walidacja: jeśli lista tekstów jest pusta → `raise ValueError`

**Wymagane metody:**

| Metoda | Opis |
|---|---|
| `top_entities(label, n=5)` | zwraca `n` najczęstszych encji o danej etykiecie (np. `"ORG"`, `"PERSON"`) wraz z liczebnościami |
| `most_common_lemmas(pos, n=10)` | zwraca `n` najczęstszych lematów filtrowanych po części mowy (`"NOUN"`, `"VERB"`, `"ADJ"`) — z pominięciem stopwords |
| `find_similar(query, n=3)` | bierze nowy tekst (string) i zwraca `n` najbardziej podobnych dokumentów z korpusu (wymaga modelu `_md` lub `_lg`) |
| `entity_summary()` | drukuje rozkład etykiet encji w całym korpusie (ile `PERSON`, ile `ORG`, ...) |
| `visualize(idx)` | renderuje `displacy` (`style="ent"`) dla dokumentu o indeksie `idx` |
| `__str__` | zwraca podsumowanie typu `NewsAnalyzer: 12 dokumentów, 47 encji, 318 tokenów` |

**Plan testów (w notebooku):**

1. Utwórz analizator z listą min. 10 tekstów.
2. Wyświetl `__str__` i `entity_summary()`.
3. Pokaż `top_entities("PERSON")` i `top_entities("ORG")`.
4. Pokaż `most_common_lemmas("NOUN")` i `most_common_lemmas("VERB")`.
5. Wywołaj `visualize(0)` na pierwszym dokumencie.
6. (Z modelem `_md`) Zadaj zapytanie `find_similar("technology and finance", n=3)` i wyświetl wyniki.

**Wskazówki:**

- `nlp.pipe(teksty)` zamiast pętli po `nlp(t)` — szybciej.
- Stopwords: `token.is_stop` zwraca `True` dla zaimków, spójników itp.
- Liczenie częstotliwości: `collections.Counter`.
- Podobieństwo: `doc1.similarity(doc2)` (wymaga modelu z wektorami).
- Walidacja w metodach: jeśli `pos` nie jest jednym z dopuszczalnych → komunikat błędu.

**Punktacja (20 pkt):**

| Element | Punkty |
|---|---|
| Konstruktor z walidacją i `nlp.pipe()` | 2 |
| `top_entities()` | 3 |
| `most_common_lemmas()` z filtrowaniem stopwords | 3 |
| `find_similar()` z poprawnym sortowaniem po podobieństwie | 4 |
| `entity_summary()` | 2 |
| `visualize()` z użyciem `displacy` | 2 |
| `__str__` z czytelnym podsumowaniem | 1 |
| Realizacja planu testów (1–6) | 3 |