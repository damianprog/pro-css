# Pro CSS Patterns

Dziennik deliberate practice: czego się nauczyłem, jakie problemy rozwiązałem, jakie decyzje podjąłem i dlaczego.

---

## Pille: wspólny komponent dla selectów-filtrów i tagów w kartach

**Data:** 2026-10-26
**Projekt:** FungiFinders (`mushroom-guide.html`)
**Status:** selecty-filtry gotowe, następne są pille w kartach i modyfikatory kolorów

### Problem

W projekcie są dwa elementy, które wyglądają prawie tak samo:

- **filtry** nad listą grzybów („Season: All”, „Type: All”). To listy rozwijane ze strzałką.
- **pille** w każdej karcie („edible”, „summer”). To tylko etykiety.

Mają stać obok siebie i dzielić jeden zestaw stylów, bez powielania kodu.

### Co jest wspólne, a co się różni

| Właściwość    | Filtr (select)              | Pilla (tag)  |
| ------------- | --------------------------- | ------------ |
| tło           | ta sama zieleń              | ta sama zieleń |
| kolor tekstu  | biały                       | biały        |
| border-radius | takie samo                  | takie samo   |
| padding       | większy, miejsce na strzałkę | mniejszy    |
| font-size     | 16px                        | 14px         |

Trzy wspólne właściwości i dwie różne → **jedna klasa bazowa plus wariant**, a nie dwa osobne komponenty.

### Decyzje i dlaczego

**1. Element wybieram po tym, co robi, a nie jak wygląda**
- Filtr otwiera listę opcji, a użytkownik wybiera jedną → `<select>`. Nie wymyślam koła na nowo: natywny select daje za darmo obsługę klawiatury, systemowe pickery na telefonie i dostępność. `<button>` oznaczałby budowanie tego wszystkiego samemu.
- Pille nic nie robią po kliknięciu → **to nie są przyciski**. To grupa tagów opisujących grzyba → lista: `<ul>` + `<li>`.
- Dwa elementy też mogą tworzyć listę. To, czy coś jest listą, zależy od *tego, czym jest treść*, a nie od *liczby* elementów. Czytnik ekranu ogłosi „lista, 2 elementy”, więc użytkownik wie, ile jest tagów i gdzie się kończą.

**2. Styluję klasą na elemencie, a nie przez `.pills li`**
Mój pierwszy pomysł to `.pills li { ... }`. Odpada z dwóch powodów:
- **Specyficzność:** `.pills li` = (0,1,1), a modyfikator `.pill--poisonous` = (0,1,0). Modyfikator *przegrywa*, więc warianty kolorów nie działają.
- **Ponowne użycie:** `<select>` nie jest `<li>`, więc nie przejmie tych stylów.

Klasa `.pill` na każdym elemencie rozwiązuje oba problemy. Baza i modyfikatory mają tę samą specyficzność, więc **decyduje kolejność w kodzie**: modyfikatory piszę *pod* klasą bazową.

**3. Modyfikatory nazywam po znaczeniu, nie po wyglądzie: `.pill--poisonous`, a nie `.bg-red`**
Rozważałem klasy utility z kolorem. Odrzuciłem je, bo:
- jeśli projekt zmieni kolor trujących z czerwonego na pomarańczowy, `.bg-red` albo kłamie, albo trzeba go podmienić w każdej karcie. Przy `.pill--poisonous` edytuję jedną regułę.
- zmiana tła może wymagać też zmiany koloru tekstu (kontrast), więc utility się mnożą (`bg-yellow text-dark`).
- Utility mają sens dla rzeczy powtarzanych w *wielu różnych* komponentach. Kolor „trujący” występuje tylko w pillach.

**4. `.pill--select` zamiast `.pill--lg`**
Najpierw myślałem o modyfikatorze rozmiaru `.pill--lg`. Ale padding selecta jest specyficzny *przez strzałkę*: potrzebuje dodatkowego miejsca z prawej. Wszystko, co go odróżnia, dotyczy tylko selectów: `appearance: none`, strzałka w tle, padding na strzałkę i większy font. Dlatego `.pill--select` opisuje **inny rodzaj pilli** (pillę ze strzałką rozwijania), a nie miejsce, w którym jest użyta.

**5. Jedna klasa układu `.pills` dla obu grup**
Oba wrappery potrzebują `display: flex` + `gap`. Selecty też mają klasę `.pill`, więc rząd filtrów to także „grupa pilli” i nazwa `.pills` pasuje do obu. `gap` daję na rodzicu, a dzieci nie mają marginesów.

**6. Reset listy i `role="list"`**
Reset `ul` (bez kropek, paddingu i marginesów) trafia do `@layer reset`, bo w tym projekcie nigdy nie potrzebuję domyślnych stylów listy. **Pułapka:** Safari z VoiceOverem przestaje traktować `<ul>` jako listę, kiedy ma `list-style: none`. Rozwiązanie: dodać `role="list"` do `<ul>`.

**7. Dostępne etykiety selectów**
Tekst opcji („Season: All”) zmienia się po wyborze, więc nie może pełnić roli etykiety. Owijam select w `<label>` (powiązanie niejawne, bez `for` i `id`) ze spanem `.visually-hidden`.
- Nie używam `display: none`, bo czytniki ekranu pomijają taki element.
- Nie piszę „Dropdown list of seasons”, bo czytnik sam ogłasza rolę („lista rozwijana”) i to się powtarza. **Dobra ukryta etykieta brzmi tak, jak brzmiałaby widoczna:** „Seasons” (albo „Filter by season”).
- `.visually-hidden` ma `position: absolute`, więc span nie staje się elementem flexa i nie dodaje dodatkowego `gap`.

### Poznane techniki

**Usuwanie natywnego wyglądu selecta**
```css
appearance: none; /* usuwa wygląd systemowy i strzałkę przeglądarki */
```
Wpływa tylko na zamknięty select. Rozwijana lista opcji zachowuje wygląd przeglądarki. (Na później: `appearance: base-select` pozwala ostylować też ją, ale wsparcie przeglądarek nie jest jeszcze pełne.)

**Pozycjonowanie obrazka w tle względem krawędzi**
```css
background-repeat: no-repeat;
background-position: right 6px center; /* 6px od prawej krawędzi, w pionie pośrodku */
```
Składnia wielowartościowa: krawędź + odległość. `center` nie przyjmuje odległości, dlatego są tu 3 wartości. `padding-right` musi być na tyle duży, żeby długi tekst nie wchodził na strzałkę.

**`background-color` zamiast skrótu `background`**
`.pill` ustawia `background-color`, a `.pill--select` dokłada `background-image`. Skrót `background` w modyfikatorze skasowałby kolor z klasy bazowej.

**Ścieżki URL w CSS**
- `/assets/x.svg` to ścieżka **od korzenia**, liczona od korzenia domeny. Psuje się przy otwarciu z dysku (`file:///assets/...`) i w podkatalogu (GitHub Pages: `user.github.io/pro-css/` → `user.github.io/assets/`).
- `assets/x.svg` lub `./assets/x.svg` to ścieżka **względna, liczona od pliku CSS**, a nie od HTML-a, który go wczytuje.
- Jeśli CSS leży w `css/`, a assets w korzeniu: `../assets/x.svg`.

**Nadpisywanie paddingu i fontu w modyfikatorze**
`.pill` używa `padding-block` i `padding-inline` + `--font-size-sm` (14px). `.pill--select` stoi niżej i używa skrótu `padding` + `--font-size-regular` (16px). Specyficzność jest równa, więc wygrywa późniejsza reguła. Używam tokenów fontu (rem) zamiast wpisanych na sztywno px.

**Fokus**
`:hover` i `:focus-visible` zmieniają tło na jaśniejszą zieleń, tak samo jak w `.button`. Reset nie usuwa `outline`, więc domyślna obwódka fokusu przeglądarki zostaje widoczna przy nawigacji Tabem.

### Aktualny kod

```html
<div class="pills">
  <label>
    <span class="visually-hidden">Seasons</span>
    <select class="pill pill--select">
      <option value="">Season: All</option>
    </select>
  </label>
  <label>
    <span class="visually-hidden">Mushroom types</span>
    <select class="pill pill--select">
      <option value="">Type: All</option>
    </select>
  </label>
</div>
```

```css
.pills {
  display: flex;
  gap: 0.5rem;
}

.pill {
  padding-block: 4px;
  padding-inline: 7px;
  background-color: var(--background-accent-main);
  color: var(--clr-white);
  border-radius: 4px;
  font-size: var(--font-size-sm);
}

.pill--select {
  appearance: none;
  border: none;
  background-image: url(./assets/down-arrow.svg);
  background-repeat: no-repeat;
  background-position: right 6px center;
  padding: 8px 30px 8px 16px;
  font-size: var(--font-size-regular);
}

.pill--select:hover,
.pill--select:focus-visible {
  background-color: var(--background-accent-light);
}
```

### Otwarte punkty

- [ ] Pille w kartach: `<ul class="pills" role="list">` + `<li class="pill">`
- [ ] Decyzja: czy „edible” to domyślna `.pill`, czy dostaje własny modyfikator?
- [ ] `.pill--poisonous` (pod `.pill`): tylko `background-color`, czy też kolor tekstu?
- [ ] Kontrast białego tekstu na `--clr-green-400` (hover i fokus). Sprawdzić w DevTools względem WCAG AA (4.5:1 dla zwykłego tekstu). Dotyczy też `.button`.
- [ ] `cursor: pointer` na `.pill--select`? (`.button` go ma)

### Pytania kontrolne

1. Dlaczego `.pills li` psuje modyfikatory kolorów? Policz obie specyficzności.
2. Dlaczego grupa dwóch tagów to nadal `<ul>`?
3. Dlaczego `.pill--poisonous`, a nie `.bg-red`?
4. Co zmienia `appearance: none`, a czego nie zmienia?
5. Plik CSS w `css/` wskazuje na `assets/arrow.svg` w korzeniu projektu. Jaka jest ścieżka?
6. Dlaczego etykieta `.visually-hidden` wewnątrz kontenera flex nie dodaje odstępu?
7. Po co `role="list"` na `<ul>` z `list-style: none`?
