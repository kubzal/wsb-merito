# Zajęcia 3

## Testowanie aplikacji Vue.js

> **Testy** to nie luksus, tylko polisa ubezpieczeniowa Twojej aplikacji.
> Lepiej spędzić godzinę pisząc testy, niż trzy godziny szukając buga w produkcji.

---

## Materiały

- Vitest (test runner zintegrowany z Vite)
    - [https://vitest.dev/](https://vitest.dev/)
- Vue Test Utils (oficjalna biblioteka do testowania komponentów Vue)
    - [https://test-utils.vuejs.org/](https://test-utils.vuejs.org/)
- Testing Library for Vue (alternatywa, nastawiona na user-centric testing)
    - [https://testing-library.com/docs/vue-testing-library/intro/](https://testing-library.com/docs/vue-testing-library/intro/)
- Vue 3 – oficjalny przewodnik testowania
    - [https://vuejs.org/guide/scaling-up/testing.html](https://vuejs.org/guide/scaling-up/testing.html)
- Istanbul / V8 coverage
    - [https://vitest.dev/guide/coverage.html](https://vitest.dev/guide/coverage.html)
- Playwright (dla zainteresowanych testami E2E)
    - [https://playwright.dev/](https://playwright.dev/)

---

## Spis treści

1. [Po co testować?](#po-co-testowac)
2. [Piramida testów](#piramida-testow)
3. [Vitest – instalacja i pierwsze uruchomienie](#vitest-instalacja)
4. [Anatomia testu jednostkowego](#anatomia-testu)
5. [Vue Test Utils – mount i wrapper](#vue-test-utils)
6. [Testowanie propsów](#testowanie-propsow)
7. [Testowanie zdarzeń użytkownika](#testowanie-zdarzen)
8. [Testowanie emit](#testowanie-emit)
9. [Testowanie `computed` i logiki pochodnej](#testowanie-computed)
10. [Testowanie kodu asynchronicznego](#testowanie-async)
11. [Mockowanie zależności](#mockowanie)
12. [Pokrycie kodu (coverage)](#coverage)
13. [Raport z testów – co i jak](#raport)
14. [Omówienie planów projektu końcowego](#plan-projektu)
15. [Podsumowanie](#podsumowanie)

---

## 1. Po co testować? {#po-co-testowac}

Testy automatyczne dają cztery rzeczy, których nie da Ci nic innego:

- **Pewność przy refaktorze** – zmieniasz kod i od razu wiesz, czy coś popsułeś
- **Dokumentację działania** – test pokazuje, jak komponent ma się zachowywać
- **Szybkie wykrywanie regresji** – CI sprawdza to, na co człowiek by zapomniał
- **Lepszy design kodu** – kod trudny do testowania to zwykle kod źle zaprojektowany

> 💡 Zasada: jeśli boisz się dotknąć fragmentu kodu, brakuje mu testów.

### Co testujemy w aplikacjach frontendowych?

| Warstwa            | Co sprawdzamy                                              |
|--------------------|------------------------------------------------------------|
| Logika pomocnicza  | Funkcje czyste, walidacja, formatowanie, computed          |
| Komponenty         | Renderowanie, props, emit, reakcja na zdarzenia            |
| Integracja         | Współpraca kilku komponentów, store, router                |
| End-to-end         | Cała aplikacja w przeglądarce, z perspektywy użytkownika   |

---

## 2. Piramida testów {#piramida-testow}

Klasyczne podejście – im wyżej, tym wolniej i drożej:

```
        /\
       /E2\        ← mało, wolne, drogie (Playwright, Cypress)
      /----\
     / Int. \      ← średnio (Vue Test Utils + store/router)
    /--------\
   /  Unit    \    ← dużo, szybkie, tanie (Vitest)
  /____________\
```

### Co z tego płynie?

- Najwięcej testów piszemy na poziomie **jednostkowym** – są tanie, szybkie i precyzyjne
- **Integracyjne** dorzucamy do kluczowych przepływów (np. rejestracja, koszyk)
- **E2E** zostawiamy dla najważniejszych ścieżek użytkownika – bo są wolne i kruche

> ⚠️ Nie odwracaj piramidy. 200 testów E2E i 0 unitów = test pipeline biegnący 40 minut i CI w ciągłym ogniu.

---

## 3. Vitest – instalacja i pierwsze uruchomienie {#vitest-instalacja}

### Jeśli tworzyłeś projekt przez `npm create vue@latest`

Kreator pyta o Vitest – jeśli zaznaczyłeś, masz już wszystko. Jeśli nie:

```bash
npm install -D vitest @vue/test-utils jsdom
```

### Konfiguracja `vite.config.js`

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  test: {
    globals: true,         // describe/it/expect dostępne globalnie
    environment: 'jsdom',  // symulacja przeglądarki w Node.js
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html'],
      include: ['src/**/*.{js,vue}'],
      exclude: ['src/main.js', 'src/router/**', 'src/**/*.spec.js']
    }
  }
})
```

### Skrypty w `package.json`

```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage",
    "test:ui": "vitest --ui"
  }
}
```

### Uruchomienie

```bash
npm run test           # tryb watch – idealny w trakcie pisania
npm run test:run       # jednorazowo (CI)
npm run test:coverage  # z raportem pokrycia
```

> 💡 `vitest --ui` otwiera ładny dashboard w przeglądarce. Świetne do prezentacji i debugowania.

---

## 4. Anatomia testu jednostkowego {#anatomia-testu}

### Wzorzec AAA (Arrange – Act – Assert)

Każdy dobrze napisany test ma trzy fazy:

1. **Arrange** – przygotuj dane wejściowe i kontekst
2. **Act** – wykonaj testowaną akcję
3. **Assert** – sprawdź wynik

### Pierwszy test – funkcja pomocnicza

```js
// src/utils/bmi.js
export function obliczBMI(masa, wzrostCm) {
  if (!masa || !wzrostCm || masa <= 0 || wzrostCm <= 0) return null
  const wzrostM = wzrostCm / 100
  return +(masa / (wzrostM ** 2)).toFixed(2)
}

export function kategoriaBMI(bmi) {
  if (bmi === null) return null
  if (bmi < 18.5) return 'Niedowaga'
  if (bmi < 25) return 'Waga prawidłowa'
  if (bmi < 30) return 'Nadwaga'
  return 'Otyłość'
}
```

```js
// src/utils/bmi.spec.js
import { describe, it, expect } from 'vitest'
import { obliczBMI, kategoriaBMI } from './bmi'

describe('obliczBMI', () => {
  it('zwraca poprawny BMI dla typowych wartości', () => {
    // Arrange
    const masa = 70
    const wzrost = 175

    // Act
    const wynik = obliczBMI(masa, wzrost)

    // Assert
    expect(wynik).toBe(22.86)
  })

  it('zwraca null dla pustych danych', () => {
    expect(obliczBMI(0, 175)).toBe(null)
    expect(obliczBMI(70, 0)).toBe(null)
    expect(obliczBMI(null, null)).toBe(null)
  })

  it('zwraca null dla wartości ujemnych', () => {
    expect(obliczBMI(-70, 175)).toBe(null)
  })
})

describe('kategoriaBMI', () => {
  it.each([
    [17, 'Niedowaga'],
    [22, 'Waga prawidłowa'],
    [27, 'Nadwaga'],
    [35, 'Otyłość']
  ])('dla BMI %i zwraca "%s"', (bmi, oczekiwana) => {
    expect(kategoriaBMI(bmi)).toBe(oczekiwana)
  })

  it('zwraca null dla null', () => {
    expect(kategoriaBMI(null)).toBe(null)
  })
})
```

### Najważniejsze matchery Vitest

| Matcher                      | Sprawdza                              |
|------------------------------|---------------------------------------|
| `.toBe(x)`                   | Równość referencyjna (===)            |
| `.toEqual(x)`                | Równość strukturalna (głęboka)        |
| `.toBeNull()` / `.toBeTruthy()` | Wartości specjalne                 |
| `.toContain(x)`              | Tablica/string zawiera element        |
| `.toHaveLength(n)`           | Długość kolekcji                      |
| `.toThrow(...)`              | Funkcja rzuca wyjątkiem               |
| `.toBeCloseTo(n, precyzja)`  | Liczby zmiennoprzecinkowe (BMI ❤️)    |

> 💡 `it.each(...)` to najwygodniejszy sposób na testy parametryzowane. Mniej copy-paste = mniej bugów.

---

## 5. Vue Test Utils – mount i wrapper {#vue-test-utils}

`@vue/test-utils` montuje komponent w pamięci i zwraca **wrapper** – obiekt z metodami do interakcji z komponentem.

### Pierwszy test komponentu

```vue
<!-- src/components/Powitanie.vue -->
<script setup>
defineProps({
  imie: { type: String, default: 'Świecie' }
})
</script>

<template>
  <h1 class="powitanie">Cześć, {{ imie }}!</h1>
</template>
```

```js
// src/components/Powitanie.spec.js
import { describe, it, expect } from 'vitest'
import { mount } from '@vue/test-utils'
import Powitanie from './Powitanie.vue'

describe('Powitanie.vue', () => {
  it('renderuje domyślne powitanie', () => {
    const wrapper = mount(Powitanie)
    expect(wrapper.text()).toContain('Cześć, Świecie!')
  })

  it('używa propsa "imie"', () => {
    const wrapper = mount(Powitanie, {
      props: { imie: 'Kuba' }
    })
    expect(wrapper.text()).toContain('Cześć, Kuba!')
  })

  it('renderuje element h1 z odpowiednią klasą', () => {
    const wrapper = mount(Powitanie)
    const h1 = wrapper.find('h1.powitanie')
    expect(h1.exists()).toBe(true)
  })
})
```

### `mount` vs `shallowMount`

| Metoda          | Co robi                                                  | Kiedy używać                            |
|-----------------|----------------------------------------------------------|-----------------------------------------|
| `mount`         | Renderuje komponent z **wszystkimi dziećmi**             | Test integracyjny, mały komponent       |
| `shallowMount`  | Renderuje komponent, ale **dzieci jako stuby**           | Izolacja, gdy dzieci są skomplikowane   |

```js
import { mount, shallowMount } from '@vue/test-utils'
```

### Najczęściej używane metody wrappera

| Metoda                         | Do czego                                |
|--------------------------------|------------------------------------------|
| `wrapper.text()`               | Cały tekst komponentu                    |
| `wrapper.html()`               | Wygenerowany HTML (przydatne w debugu)   |
| `wrapper.find('selektor')`     | Pierwszy element pasujący do selektora   |
| `wrapper.findAll('selektor')`  | Wszystkie pasujące elementy              |
| `wrapper.exists()`             | Czy element istnieje                     |
| `wrapper.classes()`            | Tablica klas CSS                         |
| `wrapper.attributes('atr')`    | Wartość atrybutu                         |
| `wrapper.element`              | Surowy DOM element                       |

---

## 6. Testowanie propsów {#testowanie-propsow}

```vue
<!-- src/components/StatusBadge.vue -->
<script setup>
const props = defineProps({
  status: {
    type: String,
    required: true,
    validator: (v) => ['ok', 'warning', 'error'].includes(v)
  }
})
</script>

<template>
  <span :class="['badge', `badge--${status}`]">{{ status.toUpperCase() }}</span>
</template>
```

```js
// src/components/StatusBadge.spec.js
import { describe, it, expect } from 'vitest'
import { mount } from '@vue/test-utils'
import StatusBadge from './StatusBadge.vue'

describe('StatusBadge.vue', () => {
  it('renderuje status na wielkie litery', () => {
    const wrapper = mount(StatusBadge, { props: { status: 'ok' } })
    expect(wrapper.text()).toBe('OK')
  })

  it.each(['ok', 'warning', 'error'])(
    'dodaje klasę badge--%s dla statusu %s',
    (status) => {
      const wrapper = mount(StatusBadge, { props: { status } })
      expect(wrapper.classes()).toContain(`badge--${status}`)
    }
  )
})
```

---

## 7. Testowanie zdarzeń użytkownika {#testowanie-zdarzen}

Wrapper udostępnia metody `trigger()` (zdarzenia DOM) i `setValue()` (formularze).

```vue
<!-- src/components/Licznik.vue -->
<script setup>
import { ref } from 'vue'

const wartosc = ref(0)

function zwieksz() { wartosc.value++ }
function zmniejsz() { wartosc.value-- }
function reset() { wartosc.value = 0 }
</script>

<template>
  <div>
    <p data-testid="wartosc">{{ wartosc }}</p>
    <button @click="zwieksz">+</button>
    <button @click="zmniejsz">-</button>
    <button @click="reset">Reset</button>
  </div>
</template>
```

```js
import { describe, it, expect } from 'vitest'
import { mount } from '@vue/test-utils'
import Licznik from './Licznik.vue'

describe('Licznik.vue', () => {
  it('zaczyna od 0', () => {
    const wrapper = mount(Licznik)
    expect(wrapper.get('[data-testid="wartosc"]').text()).toBe('0')
  })

  it('zwiększa wartość po kliknięciu +', async () => {
    const wrapper = mount(Licznik)
    const przyciskPlus = wrapper.findAll('button')[0]

    await przyciskPlus.trigger('click')
    await przyciskPlus.trigger('click')

    expect(wrapper.get('[data-testid="wartosc"]').text()).toBe('2')
  })

  it('resetuje wartość', async () => {
    const wrapper = mount(Licznik)
    const [plus, , reset] = wrapper.findAll('button')

    await plus.trigger('click')
    await plus.trigger('click')
    await reset.trigger('click')

    expect(wrapper.get('[data-testid="wartosc"]').text()).toBe('0')
  })
})
```

> 💡 **Zawsze używaj `await`** przy `trigger`, `setValue` i innych asynchronicznych operacjach. Bez tego DOM nie zdąży się zaktualizować i test sprawdzi **stary** stan.

### Atrybut `data-testid`

Selektory CSS są kruche – zmiana klasy łamie testy. **Lepiej używać `data-testid`**:

```html
<button data-testid="dodaj-do-koszyka">Dodaj</button>
```

```js
wrapper.get('[data-testid="dodaj-do-koszyka"]').trigger('click')
```

To konwencja niezależna od wyglądu i klas CSS. Stylista może przesuwać guziki – test nadal działa.

---

## 8. Testowanie emit {#testowanie-emit}

```vue
<!-- src/components/Subskrypcja.vue -->
<script setup>
import { ref } from 'vue'

const emit = defineEmits(['zapisz'])
const email = ref('')

function zapisz() {
  if (email.value.includes('@')) {
    emit('zapisz', email.value)
    email.value = ''
  }
}
</script>

<template>
  <form @submit.prevent="zapisz">
    <input v-model="email" data-testid="email" type="email" />
    <button data-testid="zapisz">Zapisz</button>
  </form>
</template>
```

```js
import { describe, it, expect } from 'vitest'
import { mount } from '@vue/test-utils'
import Subskrypcja from './Subskrypcja.vue'

describe('Subskrypcja.vue', () => {
  it('emituje "zapisz" z poprawnym adresem email', async () => {
    const wrapper = mount(Subskrypcja)

    await wrapper.get('[data-testid="email"]').setValue('kuba@example.com')
    await wrapper.get('form').trigger('submit.prevent')

    const emisje = wrapper.emitted('zapisz')
    expect(emisje).toHaveLength(1)
    expect(emisje[0]).toEqual(['kuba@example.com'])
  })

  it('NIE emituje przy nieprawidłowym emailu', async () => {
    const wrapper = mount(Subskrypcja)

    await wrapper.get('[data-testid="email"]').setValue('to-nie-email')
    await wrapper.get('form').trigger('submit.prevent')

    expect(wrapper.emitted('zapisz')).toBeUndefined()
  })

  it('czyści pole po pomyślnym zapisie', async () => {
    const wrapper = mount(Subskrypcja)
    const input = wrapper.get('[data-testid="email"]')

    await input.setValue('kuba@example.com')
    await wrapper.get('form').trigger('submit.prevent')

    expect(input.element.value).toBe('')
  })
})
```

`wrapper.emitted()` zwraca obiekt typu `{ nazwa_zdarzenia: [[arg1, arg2], [arg1]] }` – każde wywołanie to osobna tablica argumentów.

---

## 9. Testowanie `computed` i logiki pochodnej {#testowanie-computed}

`computed` testujemy **przez efekt** – nie sięgamy do zmiennej, tylko sprawdzamy, co się wyrenderowało.

```vue
<!-- src/components/KalkulatorBMI.vue (uproszczona wersja z zajęć 2) -->
<script setup>
import { ref, computed } from 'vue'

const masa = ref('')
const wzrost = ref('')

const bmi = computed(() => {
  const m = parseFloat(masa.value)
  const w = parseFloat(wzrost.value) / 100
  if (!m || !w || m <= 0 || w <= 0) return null
  return +(m / (w * w)).toFixed(2)
})

const kategoria = computed(() => {
  if (bmi.value === null) return null
  if (bmi.value < 18.5) return 'Niedowaga'
  if (bmi.value < 25) return 'Waga prawidłowa'
  if (bmi.value < 30) return 'Nadwaga'
  return 'Otyłość'
})
</script>

<template>
  <div>
    <input data-testid="masa" v-model="masa" />
    <input data-testid="wzrost" v-model="wzrost" />
    <p v-if="bmi !== null" data-testid="bmi">BMI: {{ bmi }}</p>
    <p v-if="kategoria" data-testid="kategoria">{{ kategoria }}</p>
  </div>
</template>
```

```js
import { describe, it, expect } from 'vitest'
import { mount } from '@vue/test-utils'
import KalkulatorBMI from './KalkulatorBMI.vue'

describe('KalkulatorBMI.vue', () => {
  async function ustawDane(wrapper, masa, wzrost) {
    await wrapper.get('[data-testid="masa"]').setValue(masa)
    await wrapper.get('[data-testid="wzrost"]').setValue(wzrost)
  }

  it('oblicza BMI dla poprawnych danych', async () => {
    const wrapper = mount(KalkulatorBMI)
    await ustawDane(wrapper, '70', '175')
    expect(wrapper.get('[data-testid="bmi"]').text()).toContain('22.86')
  })

  it('nie wyświetla BMI dla pustych pól', () => {
    const wrapper = mount(KalkulatorBMI)
    expect(wrapper.find('[data-testid="bmi"]').exists()).toBe(false)
  })

  it.each([
    ['50', '180', 'Niedowaga'],
    ['70', '175', 'Waga prawidłowa'],
    ['90', '175', 'Nadwaga'],
    ['120', '170', 'Otyłość']
  ])('dla %s kg i %s cm pokazuje "%s"', async (masa, wzrost, oczekiwana) => {
    const wrapper = mount(KalkulatorBMI)
    await ustawDane(wrapper, masa, wzrost)
    expect(wrapper.get('[data-testid="kategoria"]').text()).toBe(oczekiwana)
  })
})
```

---

## 10. Testowanie kodu asynchronicznego {#testowanie-async}

### Funkcje asynchroniczne i zapytania HTTP

```js
// src/api/pogoda.js
export async function pobierzPogode(miasto) {
  const res = await fetch(`https://api.example.com/weather?city=${miasto}`)
  if (!res.ok) throw new Error('Błąd API')
  return res.json()
}
```

```js
// src/api/pogoda.spec.js
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { pobierzPogode } from './pogoda'

describe('pobierzPogode', () => {
  beforeEach(() => {
    vi.restoreAllMocks()
  })

  it('zwraca dane pogodowe', async () => {
    vi.stubGlobal('fetch', vi.fn(() =>
      Promise.resolve({
        ok: true,
        json: () => Promise.resolve({ temp: 21, miasto: 'Warszawa' })
      })
    ))

    const wynik = await pobierzPogode('Warszawa')

    expect(wynik).toEqual({ temp: 21, miasto: 'Warszawa' })
    expect(fetch).toHaveBeenCalledWith(
      expect.stringContaining('city=Warszawa')
    )
  })

  it('rzuca błędem gdy API zwraca status błędu', async () => {
    vi.stubGlobal('fetch', vi.fn(() =>
      Promise.resolve({ ok: false })
    ))

    await expect(pobierzPogode('Mordor')).rejects.toThrow('Błąd API')
  })
})
```

### `flushPromises` – gdy aktualizacja DOM-u nie nadąża

```js
import { flushPromises } from '@vue/test-utils'

it('renderuje dane po załadowaniu', async () => {
  const wrapper = mount(KomponentZeFetchem)
  await flushPromises()  // czeka aż wszystkie promise się rozwiążą
  expect(wrapper.text()).toContain('Warszawa: 21°C')
})
```

---

## 11. Mockowanie zależności {#mockowanie}

`vi.mock()` zastępuje cały moduł na czas testów. Przydatne, gdy nie chcesz w testach prawdziwego API, czasu lub localStorage.

```js
// src/components/ZegarApp.vue używa formatujCzas() z utils/data.js
import { vi } from 'vitest'

vi.mock('@/utils/data', () => ({
  formatujCzas: vi.fn(() => '12:34:56'),
  formatujDate: vi.fn(() => '2026-05-09')
}))
```

### Trzy najczęstsze przypadki

| Co mockujemy           | Jak                                                       |
|------------------------|-----------------------------------------------------------|
| Funkcję                | `vi.fn(() => 'wartość zwrotna')`                          |
| Cały moduł             | `vi.mock('ścieżka', () => ({ ... }))`                     |
| Globalne (fetch, Date) | `vi.stubGlobal('fetch', vi.fn(...))`                      |

> ⚠️ Mockowanie to ostrość brzytwy. Im więcej mockujesz, tym mniej testujesz. Mockuj tylko to, co naprawdę musisz odciąć (sieć, czas, losowość, system plików).

---

## 12. Pokrycie kodu (coverage) {#coverage}

Coverage mówi Ci, **ile linii kodu** zostało wykonanych w trakcie testów. Nie mówi, czy są **dobre testy** – ale mówi, czego **w ogóle nie testujesz**.

### Uruchomienie

```bash
npm run test:coverage
```

W terminalu pojawi się tabelka, a w katalogu `coverage/` powstanie raport HTML (`coverage/index.html`).

### Cztery metryki

| Metryka      | Co mierzy                                              |
|--------------|--------------------------------------------------------|
| Statements   | % wykonanych instrukcji                                |
| Branches     | % wykonanych gałęzi (if/else, switch, ternary)         |
| Functions    | % wywołanych funkcji                                   |
| Lines        | % wykonanych linii (najbardziej intuicyjna)            |
| **Pomijaj**  | `node_modules`, `main.js`, pliki konfiguracyjne        |

### Realistyczne progi

- **70%** – minimum dla projektu studenckiego, wymagane w zadaniu 3
- **80–90%** – dobry wynik dla aplikacji produkcyjnej
- **100%** – syndrom obsesji. Bardzo rzadko ma sens.

> 💡 100% pokrycia to **nie** gwarancja braku bugów. Można pokryć każdą linię i nie sprawdzić ani jednego asercji. Coverage to wskaźnik pomocniczy, nie cel sam w sobie.

### Próg w konfiguracji (opcjonalnie)

```js
// vite.config.js
test: {
  coverage: {
    thresholds: {
      lines: 70,
      branches: 60,
      functions: 70,
      statements: 70
    }
  }
}
```

Wtedy `npm run test:coverage` rzuci błędem, jeśli pokrycie spadnie poniżej progu – idealne do CI/CD.

---

## 13. Raport z testów – co i jak {#raport}

Raport z testów to dokument, który **potrafi przeczytać osoba nietechniczna** (np. Twój klient, PO, manager). Powinien odpowiedzieć na cztery pytania:

1. **Co testowałem?**
2. **Jak testowałem?**
3. **Co znalazłem?**
4. **Co z tego wynika?**

### Standardowa struktura

```markdown
# Raport z testów – [nazwa aplikacji]

**Autor:** Imię Nazwisko
**Data:** YYYY-MM-DD
**Wersja aplikacji:** v0.1.0 (commit hash)

## 1. Wstęp
Krótki opis testowanej aplikacji – co robi, dla kogo, jaka jest jej wartość.

## 2. Zakres testów
- Co zostało objęte testami (komponenty, funkcje, scenariusze)
- Co świadomie zostało pominięte i dlaczego

## 3. Środowisko testowe
- Wersja Node.js
- Vitest, Vue Test Utils (wersje z package.json)
- System operacyjny / przeglądarka (jeśli dotyczy)

## 4. Strategia testowania
Krótkie uzasadnienie – dlaczego testujemy te rzeczy, a nie inne.
Np.: "Skupiłem się na logice obliczania BMI i renderowaniu kategorii,
bo to serce aplikacji. Pominąłem testy stylów."

## 5. Lista przypadków testowych
| ID    | Opis                                       | Oczekiwany wynik                | Status |
|-------|--------------------------------------------|---------------------------------|--------|
| TC-01 | Obliczenie BMI dla 70 kg / 175 cm          | BMI = 22.86, kat. "Prawidłowa"  | ✅     |
| TC-02 | Pole masy puste                            | Brak wyniku, brak błędu         | ✅     |
| TC-03 | Wzrost ujemny                              | Brak wyniku                     | ✅     |
| ...   | ...                                        | ...                             | ...    |

## 6. Wyniki ilościowe
- Liczba testów: **12**
- Przeszło: **12** (100%)
- Nie przeszło: **0**
- Pominiętych: **0**
- Czas wykonania: **1.2s**

## 7. Pokrycie kodu
| Metryka     | Wynik   |
|-------------|---------|
| Statements  | 87.5%   |
| Branches    | 75.0%   |
| Functions   | 90.0%   |
| Lines       | 87.5%   |

(Załącz screenshot raportu HTML lub konsoli)

## 8. Zaobserwowane problemy
| ID    | Problem                                    | Severity | Status     |
|-------|--------------------------------------------|----------|------------|
| BUG-1 | Wpisanie litery powoduje wyświetlenie NaN  | medium   | naprawione |
| BUG-2 | Brak walidacji bardzo dużych wartości      | low      | otwarte    |

## 9. Wnioski i rekomendacje
- Co działa stabilnie
- Czego brakuje (np. brak testów E2E)
- Plan poprawek przed wdrożeniem

## 10. Załączniki
- Link do repozytorium
- Plik `coverage/index.html` (lub screenshot)
```

> 💡 Raport ma być **konkretny i krótki**. 2–4 strony A4 to dobry przedział. Każdy nagłówek powinien zarobić na swoje miejsce.

### Skala severity

| Severity  | Znaczenie                                                    |
|-----------|--------------------------------------------------------------|
| critical  | Aplikacja nie działa, blokuje główny przepływ                |
| high      | Główna funkcjonalność uszkodzona, ale jest obejście          |
| medium    | Funkcjonalność poboczna nie działa lub działa nieprawidłowo  |
| low       | Drobiazg, kosmetyka, edge case                               |

---

## 14. Omówienie planów projektu końcowego {#plan-projektu}

Druga część zajęć to **omówienie Waszych planów projektu** (zgodnie z `projekt.md`, plan miał być przygotowany na dziś). Każdy z Was prezentuje:

1. **Temat i nazwa aplikacji** (1 zdanie)
2. **Funkcjonalność** (3–6 punktów)
3. **Stos technologiczny** (Vue/React, biblioteki, API)
4. **Wstępna lista komponentów**
5. **Co planujesz testować**

Idziemy szybko – każdy ma **3–5 minut**. Wspólnie szukamy:

- ❓ czy zakres jest realny do skończenia w terminie
- ❓ czy temat nie jest **zbyt trywialny** (todo lista bez logiki = nie)
- ❓ czy testy mają sens dla wybranej aplikacji

> 💡 Jeśli temat wymaga korekty – lepiej zmienić go teraz niż na tydzień przed terminem.

---

## 15. Podsumowanie {#podsumowanie}

### Czego się nauczyłeś

| Temat                       | Słowa kluczowe                                          |
|-----------------------------|---------------------------------------------------------|
| Po co testy                 | Refaktor, regresja, dokumentacja                        |
| Vitest                      | `describe`, `it`, `expect`, `it.each`                   |
| Vue Test Utils              | `mount`, `wrapper`, `find`, `data-testid`               |
| Interakcje                  | `trigger('click')`, `setValue`, `await`                 |
| Komunikacja                 | `wrapper.emitted()`                                     |
| Async                       | `flushPromises`, `vi.stubGlobal`                        |
| Mockowanie                  | `vi.fn()`, `vi.mock()`                                  |
| Coverage                    | `--coverage`, statements/branches/functions/lines       |
| Raport                      | Zakres, wyniki, severity, wnioski                       |

### Dalsze kroki

1. **Vue Testing Library** – alternatywa nastawiona na user-centric testy
2. **Playwright / Cypress** – testy end-to-end w prawdziwej przeglądarce
3. **Testy migawkowe (snapshot)** – `expect(wrapper.html()).toMatchSnapshot()`
4. **CI/CD** – uruchamianie testów na GitHub Actions przy każdym pushu
5. **Mutation testing (Stryker)** – test, który testuje Twoje testy

> 🎯 **Najważniejsze do zapamiętania**: nie testuj implementacji, testuj **zachowanie**. „Czy po kliknięciu pojawia się komunikat?", a nie „czy zmienna `x` zmieniła wartość".

---

## Zadanie 3

### Testy + Raport z testów

Weź **jedną z aplikacji** napisanych w ramach Zadania 2 (Kalkulator BMI **lub** Aplikacja Quizowa) i obejmij ją testami jednostkowymi. Następnie przygotuj **raport z testów** zgodnie ze strukturą omówioną na zajęciach.

> 💡 Polecam Quiz – ma więcej logiki do przetestowania (computed dla wyniku, emit między komponentami, walidacja). Ale BMI też nada się dobrze, jeśli rozbijesz go na funkcje pomocnicze.

### Wymagania techniczne

| Wymaganie                                                                                          | Punkty |
|----------------------------------------------------------------------------------------------------|--------|
| Minimum **8 testów jednostkowych** pokrywających: logikę (computed/funkcje), props, emit, zdarzenia | 2 pkt  |
| Wszystkie testy **przechodzą** (`npm run test:run`), pokrycie **≥ 70% lines**                       | 2 pkt  |
| **Raport z testów** w formacie PDF lub Markdown z wymaganymi sekcjami (1–10)                       | 2 pkt  |
| **Razem (merytoryczne)**                                                                            | **6 pkt** |

### Co musi zawierać raport (minimum)

1. Wstęp (krótki opis aplikacji)
2. Zakres testów
3. Środowisko (Node, Vitest, Vue Test Utils z wersjami)
4. Strategia testowania
5. Lista przypadków testowych (tabela)
6. Wyniki ilościowe
7. Pokrycie kodu (z wartościami metryk + screenshot)
8. Zaobserwowane problemy (jeśli były)
9. Wnioski
10. Link do repozytorium

### Pozostałe 6 punktów

| Wymaganie                                                                                                                                              | Punkty |
|--------------------------------------------------------------------------------------------------------------------------------------------------------|--------|
| Realizacja na zajęciach (dopuszczalny work in progress, byle nie blank page)                                                                           | 2 pkt  |
| Kod (testy + raport) na GitHubie                                                                                                                       | 2 pkt  |
| Działające demo online (dowolny serwer, np. GitHub Pages, Render, GCP/AWS/Azure free tier, MIKR.US od 35 zł/rok, lub prezentacja lokalna na zajęciach) | 2 pkt  |
| **Razem (organizacyjne)**                                                                                                                              | **6 pkt** |

### Wskazówki

- Jeśli komponent ma za dużo logiki – **wydziel ją do osobnego pliku** (np. `utils/bmi.js`) i testuj tam. Funkcje czyste są banalnie proste do testowania.
- **Nie testuj wszystkiego po kolei** – wybierz najważniejsze przepływy. 8 dobrych testów > 30 słabych.
- W raporcie **bądź szczery**: jeśli czegoś nie testowałeś, napisz dlaczego. To plus, nie minus.
- Screenshot pokrycia robisz z **raportu HTML** (`coverage/index.html`) – wygląda dużo lepiej niż konsola.
- Raport jako Markdown w repo (`docs/raport-testow.md`) jest w pełni akceptowalny – nie musisz wszystkiego konwertować do PDF.
- Testy nazywaj **opisowo**: nie `test('test1')` tylko `it('zwraca null dla pustych danych')`. Test ma czytać się jak dokumentacja.

### Punktacja końcowa

| Kategoria        | Punkty     |
|------------------|:----------:|
| Merytoryczne     | 6 pkt      |
| Organizacyjne    | 6 pkt      |
| **Łącznie**      | **12 pkt** |

> ⚠️ Testy, które się nie uruchamiają, lub plagiat raportu = **0 punktów**.

> 🎯 **Bonus tematyczny**: jeśli dorzucisz **1 test E2E w Playwright** sprawdzający główny przepływ aplikacji – mogę dorzucić +2 pkt do oceny końcowej z przedmiotu (poza pulą 100 pkt). Ale tylko jeśli reszta zadania jest na 6/6 merytorycznie.

---

*Powodzenia z testami! Pamiętajcie – kod bez testów to nadzieja, nie inżynieria. 🧪*
