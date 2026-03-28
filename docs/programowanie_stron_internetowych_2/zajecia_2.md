# Zajęcia 2

## Vue.js

> **Vue.js** to progresywny framework JavaScript do budowania interfejsów użytkownika.
> Jest lekki, intuicyjny i świetny jako pierwszy framework frontendowy.

---

## Spis treści

1. [Czym jest Vue.js?](#czym-jest-vuejs)
2. [Instalacja i konfiguracja](#instalacja-i-konfiguracja)
3. [Struktura projektu](#struktura-projektu)
4. [Komponenty i Single File Components (SFC)](#komponenty-i-single-file-components)
5. [Template syntax – szablony](#template-syntax)
6. [Reaktywność – `ref` i `reactive`](#reaktywnosc)
7. [Dyrektywy](#dyrektywy)
8. [Obsługa zdarzeń](#obsługa-zdarzen)
9. [Computed properties i watchers](#computed-i-watchers)
10. [Cykl życia komponentu](#cykl-zycia)
11. [Props i komunikacja między komponentami](#props)
12. [Emit – komunikacja w górę](#emit)
13. [Composables (logika wielokrotnego użytku)](#composables)
14. [Vue Router – podstawy](#vue-router)
15. [Pinia – zarządzanie stanem](#pinia)
16. [Podsumowanie i co dalej](#podsumowanie)

---

## 1. Czym jest Vue.js? {#czym-jest-vuejs}

Vue.js (wersja 3) to framework oparty na komponentach, który:

- **Deklaratywnie** renderuje HTML na podstawie stanu aplikacji
- Używa **Composition API** (nowoczesne podejście) lub Options API
- Jest **reaktywny** – UI automatycznie aktualizuje się przy zmianie danych
- Łatwo integruje się z istniejącymi projektami

### Vue vs React vs Angular

| Cecha            | Vue 3         | React         | Angular       |
|------------------|---------------|---------------|---------------|
| Krzywa uczenia   | ✅ Łagodna    | ⚠️ Średnia    | ❌ Stroma     |
| Rozmiar bundla   | ✅ Mały       | ✅ Mały       | ❌ Duży       |
| Two-way binding  | ✅ Wbudowany  | ⚠️ Ręczny     | ✅ Wbudowany  |
| Oficjalny router | ✅ Vue Router | ⚠️ 3rd party  | ✅ Wbudowany  |

---

## 2. Instalacja i konfiguracja {#instalacja-i-konfiguracja}

### Wymagania

- Node.js >= 18
- npm, yarn lub **pnpm** (zalecany)

### Tworzenie projektu przez Vite (zalecane)

```bash
npm create vue@latest moj-projekt
cd moj-projekt
npm install
npm run dev
```

Kreator zapyta o opcje – dla początkujących wybierz:
- ✅ Vue Router
- ✅ Pinia
- ❌ TypeScript (opcjonalnie, na start można pominąć)

### Alternatywnie – CDN (do szybkiego testowania)

```html
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
<div id="app">{{ message }}</div>
<script>
  const { createApp, ref } = Vue
  createApp({
    setup() {
      const message = ref('Cześć, Vue!')
      return { message }
    }
  }).mount('#app')
</script>
```

---

## 3. Struktura projektu {#struktura-projektu}

Po `create vue` dostaniesz:

```
moj-projekt/
├── public/             # Pliki statyczne
├── src/
│   ├── assets/         # Obrazki, fonty, style globalne
│   ├── components/     # Komponenty wielokrotnego użytku
│   ├── views/          # Strony (używane przez router)
│   ├── stores/         # Stan aplikacji (Pinia)
│   ├── router/
│   │   └── index.js    # Definicja tras
│   ├── App.vue         # Komponent główny
│   └── main.js         # Punkt wejścia aplikacji
├── index.html
└── vite.config.js
```

---

## 4. Komponenty i Single File Components {#komponenty-i-single-file-components}

Każdy plik `.vue` to **Self-Contained Component** – zawiera logikę, szablon i style w jednym miejscu.

```vue
<!-- src/components/Powitanie.vue -->
<template>
  <div class="powitanie">
    <h1>Cześć, {{ imie }}!</h1>
    <p>Licznik: {{ licznik }}</p>
    <button @click="zwieksz">Kliknij mnie</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// Props (dane z zewnątrz)
const props = defineProps({
  imie: {
    type: String,
    default: 'Świecie'
  }
})

// Stan lokalny
const licznik = ref(0)

function zwieksz() {
  licznik.value++
}
</script>

<style scoped>
/* scoped = style działają tylko w tym komponencie */
.powitanie {
  padding: 1rem;
  border: 1px solid #ccc;
}
</style>
```

> 💡 `<script setup>` to skrócona składnia Composition API – zalecana w Vue 3.

---

## 5. Template Syntax – szablony {#template-syntax}

### Interpolacja tekstu

```html
<p>{{ wiadomosc }}</p>
<p>{{ liczba * 2 }}</p>
<p>{{ osoba.imie.toUpperCase() }}</p>
```

### Bindowanie atrybutów (`v-bind` lub `:`)

```html
<!-- Pełna składnia -->
<img v-bind:src="urlObrazka" />

<!-- Skrócona (zalecana) -->
<img :src="urlObrazka" :alt="opisObrazka" />
<button :disabled="czyNieaktywny">Wyślij</button>
```

### Surowy HTML (`v-html`)

```html
<!-- Uwaga: używaj tylko z zaufanymi danymi! -->
<div v-html="htmlContent"></div>
```

---

## 6. Reaktywność – `ref` i `reactive` {#reaktywnosc}

### `ref` – dla wartości prostych (i nie tylko)

```js
import { ref } from 'vue'

const licznik = ref(0)
const imie = ref('Kuba')
const czyZalogowany = ref(false)

// W skrypcie odwołuj się przez .value
console.log(licznik.value)  // 0
licznik.value++
console.log(licznik.value)  // 1

// W szablonie .value jest zbędne – Vue robi to automatycznie
// {{ licznik }} → "1"
```

### `reactive` – dla obiektów

```js
import { reactive } from 'vue'

const uzytkownik = reactive({
  imie: 'Kuba',
  wiek: 30,
  adres: {
    miasto: 'Warszawa'
  }
})

// Modyfikacja – bez .value
uzytkownik.wiek = 31
uzytkownik.adres.miasto = 'Kraków'
```

### Kiedy używać czego?

| `ref`                          | `reactive`                      |
|--------------------------------|---------------------------------|
| Prymitywy (string, number...)  | Złożone obiekty / tablice       |
| Gdy potrzebujesz reassign      | Gdy modyfikujesz właściwości    |
| Domyślny wybór (bezpieczniejszy) | Wygodniejsza składnia dla obiektów |

> 💡 Dla uproszczenia – używaj `ref` do wszystkiego, aż poczujesz potrzebę `reactive`.

---

## 7. Dyrektywy {#dyrektywy}

### `v-if`, `v-else-if`, `v-else` – renderowanie warunkowe

```html
<div v-if="wynik > 90">Ocena: 5</div>
<div v-else-if="wynik > 75">Ocena: 4</div>
<div v-else-if="wynik > 60">Ocena: 3</div>
<div v-else>Ocena: 2</div>
```

### `v-show` – ukrywanie przez CSS

```html
<!-- Element istnieje w DOM, ale display: none gdy false -->
<p v-show="czyWidoczny">Ten tekst można ukryć</p>
```

> Użyj `v-show` gdy element często zmienia widoczność, `v-if` gdy warunek rzadko się zmienia.

### `v-for` – pętle

```html
<ul>
  <li v-for="produkt in produkty" :key="produkt.id">
    {{ produkt.nazwa }} – {{ produkt.cena }} zł
  </li>
</ul>

<!-- Z indeksem -->
<div v-for="(element, index) in lista" :key="index">
  {{ index + 1 }}. {{ element }}
</div>
```

> ⚠️ Zawsze dodawaj `:key` – Vue potrzebuje go do efektywnego re-renderowania.

### `v-model` – two-way binding

```html
<input v-model="szukana" placeholder="Wpisz szukaną frazę..." />
<p>Szukasz: {{ szukana }}</p>

<!-- Checkbox -->
<input type="checkbox" v-model="zgodaRodo" />

<!-- Select -->
<select v-model="wybraneMiasto">
  <option>Warszawa</option>
  <option>Kraków</option>
  <option>Gdańsk</option>
</select>
```

```js
const szukana = ref('')
const zgodaRodo = ref(false)
const wybraneMiasto = ref('Warszawa')
```

---

## 8. Obsługa zdarzeń {#obsługa-zdarzen}

### `v-on` / `@`

```html
<!-- Pełna i skrócona składnia -->
<button v-on:click="kliknij">Kliknij</button>
<button @click="kliknij">Kliknij</button>

<!-- Inline -->
<button @click="licznik++">+1</button>

<!-- Z argumentem -->
<button @click="usun(produkt.id)">Usuń</button>

<!-- Z obiektem zdarzenia -->
<input @keyup.enter="zatwierdz" />
<form @submit.prevent="wyslij">...</form>
```

### Modyfikatory zdarzeń

```html
@click.stop      <!-- event.stopPropagation() -->
@click.prevent   <!-- event.preventDefault() -->
@click.once      <!-- wywołaj tylko raz -->
@keyup.enter     <!-- tylko klawisz Enter -->
@keyup.esc       <!-- tylko klawisz Escape -->
```

---

## 9. Computed Properties i Watchers {#computed-i-watchers}

### `computed` – wartości pochodne (z cache'owaniem)

```js
import { ref, computed } from 'vue'

const produkty = ref([
  { nazwa: 'Kawa', cena: 15, kategoria: 'napoje' },
  { nazwa: 'Herbata', cena: 8, kategoria: 'napoje' },
  { nazwa: 'Ciastko', cena: 6, kategoria: 'przekąski' },
])

const filtr = ref('')

// Przelicza się tylko gdy produkty lub filtr się zmienią
const przefiltrowane = computed(() =>
  produkty.value.filter(p =>
    p.nazwa.toLowerCase().includes(filtr.value.toLowerCase())
  )
)

const sumaCen = computed(() =>
  produkty.value.reduce((sum, p) => sum + p.cena, 0)
)
```

### `watch` – reagowanie na zmiany

```js
import { ref, watch, watchEffect } from 'vue'

const szukana = ref('')

// Reaguje na zmianę konkretnej wartości
watch(szukana, (nowaWartosc, staraWartosc) => {
  console.log(`Zmieniono z "${staraWartosc}" na "${nowaWartosc}"`)
  pobierzWyniki(nowaWartosc)
})

// watchEffect – uruchamia się od razu i śledzi zależności automatycznie
watchEffect(() => {
  console.log(`Aktualna fraza: ${szukana.value}`)
})
```

---

## 10. Cykl życia komponentu {#cykl-zycia}

```
Tworzenie → Montowanie → Aktualizacje → Odmontowanie
```

```js
import { onMounted, onUpdated, onUnmounted, onBeforeMount } from 'vue'

// Najczęściej używane:
onMounted(() => {
  // Komponent jest w DOM – tutaj pobierasz dane z API
  console.log('Komponent zamontowany')
  pobierzDane()
})

onUpdated(() => {
  // Wywołane po każdej aktualizacji DOM
})

onUnmounted(() => {
  // Sprzątanie: zatrzymaj timery, odsubskrybuj eventy
  clearInterval(timer)
})
```

### Przykład – pobieranie danych

```js
import { ref, onMounted } from 'vue'

const posty = ref([])
const ladowanie = ref(true)
const blad = ref(null)

onMounted(async () => {
  try {
    const res = await fetch('https://jsonplaceholder.typicode.com/posts?_limit=5')
    posty.value = await res.json()
  } catch (err) {
    blad.value = 'Nie udało się pobrać danych'
  } finally {
    ladowanie.value = false
  }
})
```

---

## 11. Props – komunikacja z rodzica do dziecka {#props}

```vue
<!-- Rodzic: App.vue -->
<template>
  <KartaUzytkownika
    :imie="uzytkownik.imie"
    :wiek="uzytkownik.wiek"
    :aktywny="true"
  />
</template>
```

```vue
<!-- Dziecko: KartaUzytkownika.vue -->
<script setup>
const props = defineProps({
  imie: {
    type: String,
    required: true
  },
  wiek: {
    type: Number,
    default: 0
  },
  aktywny: {
    type: Boolean,
    default: false
  }
})
</script>

<template>
  <div :class="{ aktywny: props.aktywny }">
    <h2>{{ props.imie }}</h2>
    <p>Wiek: {{ props.wiek }}</p>
  </div>
</template>
```

> ⚠️ Props są **readonly** – dziecko nie powinno ich modyfikować!

---

## 12. Emit – komunikacja z dziecka do rodzica {#emit}

```vue
<!-- Dziecko: FormularzKomentarza.vue -->
<script setup>
import { ref } from 'vue'

const emit = defineEmits(['dodajKomentarz', 'anuluj'])
const tresc = ref('')

function wyslij() {
  if (tresc.value.trim()) {
    emit('dodajKomentarz', { tresc: tresc.value, data: new Date() })
    tresc.value = ''
  }
}
</script>

<template>
  <div>
    <textarea v-model="tresc" placeholder="Twój komentarz..."></textarea>
    <button @click="wyslij">Wyślij</button>
    <button @click="emit('anuluj')">Anuluj</button>
  </div>
</template>
```

```vue
<!-- Rodzic -->
<FormularzKomentarza
  @dodaj-komentarz="handleNowyKomentarz"
  @anuluj="pokazFormularz = false"
/>
```

---

## 13. Composables – wielokrotna logika {#composables}

Composable to funkcja, która opakowuje logikę reaktywną do ponownego użycia.

```js
// src/composables/useLicznik.js
import { ref, computed } from 'vue'

export function useLicznik(poczatkowa = 0) {
  const wartosc = ref(poczatkowa)

  const podwojena = computed(() => wartosc.value * 2)

  function zwieksz(o = 1) { wartosc.value += o }
  function zmniejsz(o = 1) { wartosc.value -= o }
  function reset() { wartosc.value = poczatkowa }

  return { wartosc, podwojena, zwieksz, zmniejsz, reset }
}
```

```vue
<!-- Użycie w komponencie -->
<script setup>
import { useLicznik } from '@/composables/useLicznik'

const { wartosc, podwojena, zwieksz, zmniejsz, reset } = useLicznik(10)
</script>

<template>
  <p>Wartość: {{ wartosc }} (podwojona: {{ podwojena }})</p>
  <button @click="zwieksz()">+1</button>
  <button @click="zmniejsz()">-1</button>
  <button @click="reset()">Reset</button>
</template>
```

---

## 14. Vue Router – podstawy {#vue-router}

```js
// src/router/index.js
import { createRouter, createWebHistory } from 'vue-router'
import Strona from '@/views/Strona.vue'
import OFirmie from '@/views/OFirmie.vue'
import Produkt from '@/views/Produkt.vue'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    { path: '/', component: Strona },
    { path: '/o-firmie', component: OFirmie },
    { path: '/produkt/:id', component: Produkt },  // parametr dynamiczny
  ]
})

export default router
```

```vue
<!-- Nawigacja w szablonie -->
<nav>
  <RouterLink to="/">Strona główna</RouterLink>
  <RouterLink to="/o-firmie">O firmie</RouterLink>
  <RouterLink :to="`/produkt/${produkt.id}`">{{ produkt.nazwa }}</RouterLink>
</nav>

<!-- Tutaj renderują się strony -->
<RouterView />
```

```js
// Programowa nawigacja w skrypcie
import { useRouter, useRoute } from 'vue-router'

const router = useRouter()
const route = useRoute()

// Odczyt parametru URL: /produkt/42
console.log(route.params.id)  // "42"

// Nawigacja
router.push('/o-firmie')
router.push({ path: `/produkt/${id}` })
router.back()
```

---

## 15. Pinia – zarządzanie stanem {#pinia}

Pinia to oficjalny store Vue 3 – zastąpił Vuex.

```js
// src/stores/koszyk.js
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useKoszykStore = defineStore('koszyk', () => {
  // Stan
  const produkty = ref([])

  // Computed (jak getters)
  const liczbaProduktow = computed(() => produkty.value.length)
  const sumaKoszyka = computed(() =>
    produkty.value.reduce((sum, p) => sum + p.cena * p.ilosc, 0)
  )

  // Akcje
  function dodaj(produkt) {
    const istniejacy = produkty.value.find(p => p.id === produkt.id)
    if (istniejacy) {
      istniejacy.ilosc++
    } else {
      produkty.value.push({ ...produkt, ilosc: 1 })
    }
  }

  function usun(id) {
    produkty.value = produkty.value.filter(p => p.id !== id)
  }

  function wyczysc() {
    produkty.value = []
  }

  return { produkty, liczbaProduktow, sumaKoszyka, dodaj, usun, wyczysc }
})
```

```vue
<!-- Użycie w komponencie -->
<script setup>
import { useKoszykStore } from '@/stores/koszyk'

const koszyk = useKoszykStore()
</script>

<template>
  <p>Produktów w koszyku: {{ koszyk.liczbaProduktow }}</p>
  <p>Suma: {{ koszyk.sumaKoszyka }} zł</p>
  <button @click="koszyk.dodaj(produkt)">Dodaj do koszyka</button>
</template>
```

---

## 16. Podsumowanie i co dalej {#podsumowanie}

### Czego się nauczyłeś

| Temat                  | Słowa kluczowe                              |
|------------------------|---------------------------------------------|
| Komponenty             | `.vue`, `<script setup>`, SFC               |
| Reaktywność            | `ref()`, `reactive()`                       |
| Szablony               | `{{ }}`, `:bind`, `@event`                  |
| Dyrektywy              | `v-if`, `v-for`, `v-model`, `v-show`        |
| Logika pochodna        | `computed()`, `watch()`                     |
| Cykl życia             | `onMounted()`, `onUnmounted()`              |
| Komunikacja            | `props`, `emit`                             |
| Wielokrotna logika     | Composables                                 |
| Routing                | Vue Router, `RouterLink`, `RouterView`      |
| Stan globalny          | Pinia, `defineStore`                        |

### Dalsze kroki

1. **Oficjalna dokumentacja**: [vuejs.org/guide](https://vuejs.org/guide/introduction.html) – najlepsza w branży
2. **TypeScript z Vue 3** – silne typowanie, bardzo polecane
3. **Nuxt.js** – meta-framework Vue z SSR, file-based routing
4. **VueUse** – biblioteka gotowych composables (useFetch, useStorage, useGeolocation...)
5. **Vitest + Vue Test Utils** – testowanie komponentów
6. **Vite** – dogłębne poznanie bundlera


---

> 📚 Notatka opracowana dla poziomu: **Początkujący → Średniozaawansowany**
> Vue 3 + Composition API + `<script setup>` | Vite | Pinia | Vue Router

## Zadanie 2

Vue.js: Aplikacje Reaktywne

## Zasady ogólne

- Projekt tworzysz przy użyciu **Vite + Vue 3** (`npm create vue@latest`)
- Używasz wyłącznie **Composition API** ze składnią `<script setup>`
- Każdy komponent powinien znajdować się w osobnym pliku `.vue`
- Kod powinien być **czytelny i opatrzony komentarzami** tam, gdzie to konieczne
- Niedozwolone jest używanie gotowych bibliotek komponentów (Vuetify, PrimeVue itp.)
- Stylowanie: dowolne (czysty CSS, SCSS, Tailwind – do wyboru)
- Oddajesz **repozytorium Git** (GitHub / GitLab) lub spakowany folder projektu

---

## Część 1 – Kalkulator BMI (4 pkt)

### Opis zadania

Stwórz aplikację w Vue 3, która oblicza wskaźnik BMI (Body Mass Index) na podstawie danych wprowadzonych przez użytkownika.

**Wzór:**
```
BMI = masa (kg) / (wzrost (m))²
```

### Wymagania funkcjonalne

Aplikacja musi:

1. Posiadać **dwa pola tekstowe** (`<input>`) do wprowadzenia:
   - masy ciała w kilogramach
   - wzrostu w centymetrach
2. **Na bieżąco** (bez przycisku) obliczać i wyświetlać wynik BMI z dokładnością do **2 miejsc po przecinku**
3. Wyświetlać **kategorię wagową** odpowiadającą wyniku:

| BMI              | Kategoria          |
|------------------|--------------------|
| poniżej 18.5     | Niedowaga          |
| 18.5 – 24.99     | Waga prawidłowa    |
| 25.0 – 29.99     | Nadwaga            |
| 30.0 i powyżej   | Otyłość            |

4. Wyświetlać kategorię w **innym kolorze** dla każdego przedziału
5. Gdy pola są puste lub dane są niepoprawne – **nie wyświetlać wyniku** (żadnych błędów `NaN`)

### Wymagania techniczne

| Wymaganie                                  | Punkty |
|--------------------------------------------|--------|
| Poprawne użycie `v-model` na obu polach    | 1 pkt  |
| Obliczenie BMI przez `computed()`          | 1 pkt  |
| Warunkowe wyświetlanie kategorii (`v-if`)  | 1 pkt  |
| Kolorowanie kategorii + obsługa błędnych danych | 1 pkt |
| **Razem**                                  | **4 pkt** |

### Wskazówki

- Wartości z `<input>` są zawsze **stringiem** – użyj `parseFloat()` do konwersji
- Sprawdź czy wzrost i waga są **liczbami większymi od 0** przed obliczeniem
- Pamiętaj o konwersji centymetrów na metry przed podstawieniem do wzoru
- Kategorie najwygodniej zwrócić z osobnej funkcji pomocniczej lub drugiego `computed`

---

## Część 2 – Aplikacja Quizowa (8 pkt)

### Opis zadania

Stwórz **wielokomponentową** aplikację quizową w Vue 3. Quiz powinien zawierać minimum **5 pytań** jednokrotnego wyboru (4 odpowiedzi do każdego pytania). Pytania mogą dotyczyć dowolnego tematu.

### Wymagania funkcjonalne

Aplikacja musi składać się z **trzech ekranów**:

#### Ekran 1 – Start
- Wyświetla tytuł quizu i ewentualny opis
- Zawiera przycisk **„Rozpocznij quiz"** uruchamiający rozgrywkę

#### Ekran 2 – Pytanie
- Wyświetla **numer pytania** (np. „Pytanie 3 / 5") i **pasek postępu**
- Wyświetla treść aktualnego pytania
- Pokazuje **4 przyciski** z odpowiedziami do wyboru
- Po wybraniu odpowiedzi:
  - Podświetla wybraną odpowiedź na **zielono** (poprawna) lub **czerwono** (błędna)
  - Podświetla **poprawną odpowiedź** jeśli wybrano błędną
  - Blokuje możliwość zmiany odpowiedzi
  - Aktywuje przycisk **„Następne pytanie"**
- Przejście do następnego pytania czyści zaznaczenie i pokazuje kolejne

#### Ekran 3 – Wynik
- Wyświetla **liczbę poprawnych odpowiedzi** (np. „4 / 5")
- Wyświetla **procentowy wynik** i słowną ocenę:

| Wynik        | Komunikat                        |
|--------------|----------------------------------|
| 100%         | Doskonale! Bezbłędny wynik! 🏆   |
| 80–99%       | Bardzo dobrze! 🎉                |
| 60–79%       | Nieźle, ale jest pole do poprawy |
| poniżej 60%  | Warto powtórzyć materiał 📚      |

- Zawiera przycisk **„Zagraj ponownie"** resetujący quiz do stanu początkowego

### Wymagania techniczne i architektura

Aplikacja powinna być podzielona na **co najmniej 3 komponenty**:

```
App.vue
├── EkranStart.vue       # Ekran powitalny
├── EkranPytanie.vue     # Pojedyncze pytanie + odpowiedzi
└── EkranWynik.vue       # Podsumowanie
```

Dane pytań definiujesz jako tablicę obiektów w `App.vue` lub osobnym pliku `questions.js`:

```js
// Przykładowa struktura danych
const pytania = [
  {
    id: 1,
    tresc: 'Jakie rozszerzenie mają pliki komponentów Vue?',
    odpowiedzi: ['.jsx', '.vue', '.component', '.html'],
    poprawna: 1  // indeks poprawnej odpowiedzi (0-based)
  },
  // ...
]
```

### Kryteria oceniania

| Wymaganie                                                         | Punkty |
|-------------------------------------------------------------------|--------|
| Trzy ekrany działają poprawnie, nawigacja między nimi             | 2 pkt  |
| Poprawna struktura komponentów, props i emit                      | 2 pkt  |
| Podświetlanie odpowiedzi (zielony/czerwony) + blokada po wyborze  | 2 pkt  |
| Ekran wyników z oceną słowną + reset quizu                        | 1 pkt  |
| Pasek postępu + licznik pytań                                     | 1 pkt  |
| **Razem**                                                         | **8 pkt** |

### Wskazówki

- Stan aktualnego ekranu trzymaj w `App.vue` jako `ref` (np. `'start' | 'quiz' | 'wynik'`)
- Do przełączania ekranów użyj `v-if` / `v-else-if` na komponentach
- Wynik (liczba poprawnych odpowiedzi) trzymaj w `App.vue` i przekazuj do `EkranWynik` przez `props`
- Komunikacja dziecko → rodzic (wybór odpowiedzi, przejście dalej) przez `emit`
- Pamiętaj o **resetowaniu indeksu pytania i wyniku** przy ponownym starcie

---

## Punktacja końcowa

| Część                  | Max. punktów |
|------------------------|--------------|
| Część 1 – Kalkulator BMI | 4 pkt      |
| Część 2 – Quiz           | 8 pkt      |
| **Łącznie**            | **12 pkt**   |



> ⚠️ Projekty, które się nie uruchamiają lub są plagiatem, otrzymują **0 punktów**.

---

*Powodzenia! 🚀*