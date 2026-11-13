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

**6. Reset listy przez `role="list"`**
`@layer reset` ma już regułę, która resetuje listy, ale tylko te z atrybutem `role="list"`:
```css
/* https://www.scottohara.me/blog/2019/01/12/lists-and-safari.html */
[role='list'] {
  list-style: none;
  margin: 0;
  padding: 0;
}
```
Nie dodawałem osobnego resetu dla `ul`. Wystarczy dopisać `role="list"` do `<ul>` i reset zadziała. **Dlaczego tak:** Safari z VoiceOverem przestaje traktować `<ul>` jako listę, kiedy ma `list-style: none`. `role="list"` przywraca semantykę listy, a przy okazji jest „włącznikiem” resetu. Listy bez tego atrybutu zachowują domyślne style.

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

- [x] Pille w kartach: `<ul class="pills" role="list">` + `<li class="pill">` (zob. wpis „Karty grzybów”; `.pills` dostało też `flex-wrap: wrap`)
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

---

## Karty grzybów: ten sam `.card`, inna zawartość i kontekst sekcji

**Data:** 2026-10-27
**Projekt:** FungiFinders (`mushroom-guide.html`, sekcja „Get to know your mushrooms”)
**Status:** struktura i style gotowe, teraz wstawiam treść z Figmy

### Problem

Na drugiej stronie są karty grzybów. Na pierwszy rzut oka wyglądają jak zupełnie inny komponent niż karty z pierwszej strony („Outdoor Exploration” itd.): mają większy, biały tytuł, pille i stopkę z notatkami. Pytanie: nowy komponent, modyfikator `.card`, czy coś jeszcze innego?

### Rozbiór: z czego składają się karty

| Część          | Pierwsza strona | Druga strona       |
| -------------- | --------------- | ------------------ |
| obrazek        | ✅              | ❌                 |
| tytuł          | ✅              | ✅ (inny wygląd)   |
| pille          | ❌              | ✅                 |
| opis           | ✅              | ✅ (taki sam)      |
| stopka z notes | ❌              | ✅                 |

Wspólne pudełko: tło, `border-radius`, padding, pionowy układ z `gap`.

Wrażenie „są na tyle różne, że to nie może być ta sama karta” nie jest argumentem. Dopiero wypisanie części pokazało konkrety. Tytuł nie jest „w innym miejscu”, po prostu nad nim nie ma obrazka.

### Decyzje i dlaczego

**1. `.card` to pudełko, a nie przepis na zawartość**
Sprawdziłem, co `.card` naprawdę ustawia: układ, `gap`, padding, tło, zaokrąglenie i zaokrąglenie obrazka, *jeśli* jakiś jest. Nic nie wymaga obrazka ani nie zabrania pilli czy stopki. Dlatego obie karty to `.card`. Różna zawartość leży w tym samym pudełku i nie potrzebuję modyfikatora od „layoutu i contentu”.

**2. Nazwa modyfikatora: ani według wyglądu, ani według treści**
- `card__title--white` odpada z tego samego powodu co `.bg-red`: przy zmianie koloru klasa kłamie i trzeba ją podmieniać w każdej karcie.
- `card--mushroom` (według treści) kłamie, gdy w takiej karcie pojawi się coś innego niż grzyb.
- „Karta, która coś opisuje” niczego nie odróżnia, bo karty z pierwszej strony też coś opisują.

Wniosek: kiedy nie da się znaleźć nazwy wariantu, warto sprawdzić, czy to w ogóle *jest* wariant. Tutaj nie był.

**3. Kontekst ustawia zmienne, a komponent ich używa**
`.card__title` już wcześniej czytał zmienne z wartościami domyślnymi:
```css
.card__title {
  font-size: var(--card-title-font-size, var(--font-size-heading-sm));
  color: var(--card-title-color, var(--text-brand));
}
```
Zmienne CSS są dziedziczone, więc wystarczy ustawić je na sekcji. Wszystkie tytuły kart w środku je dostaną, a kod `.card__title` się nie zmienia.

Dlaczego to lepsze niż `.card__title--mushroom-types`:
- karta nic nie wie o sekcjach, tylko wystawia „pokrętła”, a kontekst je przekręca,
- nowa sekcja z innym kolorem tytułów to zero zmian w komponencie karty, a nie kolejny modyfikator,
- „wiedza o miejscu” leży w jedynym miejscu, które i tak jest jednorazowe, czyli w sekcji.

**4. `class="section mushroom-types"`, a nie `section--mushroom-types`**
`.section--compact` zmienia *samą sekcję* (padding). `.mushroom-types` zmienia *karty w środku*. To inna rola, więc modyfikator `.section` nie pasuje. To osobna klasa, która tylko stoi na tym samym elemencie.

Nie ustawiam zmiennych przez atrybut `style` w HTML, bo HTML to struktura, a CSS to wygląd.

**5. Nowa warstwa `@layer pages`**
`.mushroom-types` nie pasuje do żadnej istniejącej warstwy: to nie layout, nie komponent wielokrotnego użytku (w przeciwieństwie do `.hero`, który jest na dwóch stronach) i nie utility.

Odrzucone nazwy:
- `custom-page-components`: „components” przeczy wnioskowi, że to nie jest komponent, a „custom” nic nie mówi, bo cały CSS jest „custom”.
- `page-modifiers`: „modyfikator” ma już w projekcie konkretne znaczenie (klasy z `--`), a tu ich nie ma.

`pages` mówi tylko, *do czego* należą style. Kolejność:
```
reset → base → layout → components → pages → utilities
```
- **za `components`**, bo style strony czasem muszą nadpisać komponent,
- **przed `utilities`**, bo utility to świadoma decyzja na konkretnym elemencie i ma wygrywać zawsze.

**6. Token znaczeniowy `--font-size-heading-xs`**
Tytuł w tych kartach potrzebuje `--fs-600`, czyli mniej niż najmniejszy dotychczasowy nagłówek (`--font-size-heading-sm` = `--fs-700`). Najpierw użyłem `--font-size-lg`, bo ma tę samą wartość, ale korzystają z niego linki w mobilnej nawigacji i „Happy Foraging!” w stopce strony. Gdyby ktoś je powiększył, tytuły kart urosłyby razem z nimi, bez powodu.

Dodałem więc `--font-size-heading-xs: var(--fs-600)`.
- `--fs-600` mówi **ile** (stopień skali).
- `--font-size-heading-xs` i `--font-size-lg` mówią **do czego**.

Dwa tokeny wskazujące na tę samą wartość to nie powielanie. Te elementy dziś *przypadkiem* mają ten sam rozmiar. Wspólny token dostają tylko rzeczy, które mają się zmieniać razem.

**7. `card__footer`, a nie `card__notes`**
Nazwa elementu opisuje jego **rolę** w karcie, a nie treść. „notes” przestaje być prawdą, gdy na dole innej karty pojawi się np. cena z przyciskiem. „footer” pasuje do każdej karty, tak jak `__title`.

**8. `<article>` dla kart grzybów, `<div>` dla kart z pierwszej strony**
Test: *czy ta treść ma sens, kiedy wyjmę ją ze strony i wyślę komuś osobno?*
- „Chanterelle”: tak, to pełny opis grzyba → `<article>` (jak artykuł w gazecie: tytuł, treść, stopka).
- „Culinary Delights”: nie, bez strony nie wiadomo, czego dotyczy. To punkt z listy zalet → `<div>`.

`<section>` to *część większej całości*, a `<article>` to *samodzielna treść*.

Ta sama klasa `.card` stoi na `<div>` i na `<article>`. Klasa odpowiada za wygląd, element za znaczenie i jedno nie zależy od drugiego.

**9. `<footer>` wewnątrz `<article>`**
`<footer>` nie musi być stopką strony. Wewnątrz `<article>` jest stopką tego artykułu.

**10. Karty równej wysokości, pille bez rozciągania, stopka na dole**
Objaw: kiedy jedna karta ma więcej pilli, w pozostałych pille robią się wyższe, a stopka „puchnie”.

Łańcuch przyczyn:
1. `.equal-columns` ma `align-items: stretch`, więc wszystkie karty w rzędzie dostają wysokość najwyższej. **To chcę zachować.**
2. `.card` był gridem. Dodatkowa wysokość rozkładała się **po równo między wszystkie wiersze** (tytuł, pille, opis, stopka). Dlatego przerwy wyglądały na „równe”: wszystkie urosły o tyle samo. Punkt odniesienia to najwyższa karta, bo ona nie jest rozciągana.
3. `.pills` to flex z domyślnym `align-items: stretch`, więc pille rosły razem ze swoim wierszem.

Po drodze: `align-items: start` na `.card` zmniejszyło elementy, ale wiersze nadal rosły. Potrzebne było `align-content: start`.

Ostatecznie stopka ma być przyklejona do dołu. Rozważyłem dwa podejścia:
- `grid-template-rows: auto auto 1fr auto` odpada, bo `.card` musiałby wiedzieć, ile ma dzieci (pierwsza strona: 3, grzyby: 4).
- **flex w kolumnie i `margin-top: auto` na stopce**. Wiedza „stoję na dole” należy do stopki, więc każda przyszła karta ze stopką zachowa się tak samo.

**11. `<strong>`, a nie `<b>` dla „Important notes:”**
- `<strong>` oznacza, że coś jest ważne, poważne albo pilne (ostrzeżenie).
- `<b>` zwraca uwagę, ale nie dodaje ważności (słowa kluczowe, nazwa produktu).
- `<em>` zmienia akcent w zdaniu.
- `<i>` oznacza tekst w innym tonie, np. nazwę łacińską: `<i lang="la">Cantharellus cibarius</i>`. `lang` sprawia, że czytnik ekranu przeczyta ją z łacińską wymową.

### Poznane techniki

**Łączenie klas w selektorze**
| Selektor | Co łapie |
|---|---|
| `.a.b` | jeden element z **obiema** klasami, (0,2,0) |
| `.a .b` | `.b` **wewnątrz** `.a` |
| `.a, .b` | element z **którąkolwiek** klasą |

**Warstwy wygrywają ze specyficznością**
Kiedy reguły są w różnych warstwach, decyduje **kolejność warstw**, a nie specyficzność ani miejsce w pliku. Nawet `#id` w `components` przegra z jedną klasą w `pages`. Specyficzność i kolejność w pliku liczą się tylko w obrębie tej samej warstwy.

Kolejność warstw bez deklaracji wynika z kolejności ich pierwszego pojawienia się w pliku. Można ją zapisać wprost na górze:
```css
@layer reset, base, layout, components, pages, utilities;
```

**Dziedziczenie a reguła bezpośrednia**
`.mushroom-types { color: white; }` na `<section>` **nie** zmieni koloru tytułu, bo `.card__title` ustawia `color` bezpośrednio na `<h3>`. Wartość odziedziczona zawsze przegrywa z regułą, która trafia w element, nawet z wcześniejszej warstwy. Zmienne działają, bo `.card__title` sam *czyta* odziedziczoną zmienną.

**`align-items` w gridzie i we flexie**
| | Co wyrównuje | Oś |
|---|---|---|
| flex | dzieci w *linii flexa* | poprzeczna |
| grid | dzieci w *ich komórce* | blokowa |

`stretch` w obu przypadkach znaczy „wypełnij dostępną wysokość”.

**`align-items` a `align-content`**
- `align-items` ustawia, gdzie stoi **element w swoim wierszu**.
- `align-content` ustawia, jak **same wiersze** dzielą przestrzeń kontenera.

**`margin-top: auto` we flexie**
Automatyczny margines zjada całą wolną przestrzeń po swojej stronie i dopycha element na koniec osi.

**`flex-wrap: wrap`**
Bez niego flex upycha wszystko w jednym rzędzie (elementy się ściskają albo wychodzą poza kontener). `gap` działa też między zawiniętymi rzędami.

**DevTools: plakietki `grid` / `flex`**
W panelu Elements plakietka obok elementu włącza nakładkę z liniami siatki lub flexa. Dzięki niej widać, który wiersz rośnie i dlaczego.

### Aktualny kod

```html
<section class="section mushroom-types">
  <div class="wrapper flow">
    <h2 class="section-title">Get to know your mushrooms</h2>
    <!-- filtry: .pills z selectami -->
    <div class="equal-columns">
      <article class="card">
        <h3 class="card__title">Chanterelle</h3>
        <ul role="list" class="pills">
          <li class="pill">edible</li>
          <li class="pill">summer</li>
        </ul>
        <p>Golden-yellow, funnel-shaped mushroom with false gills</p>
        <footer class="card__footer">
          Important notes: Has toxic look-alikes - learn proper
          identification
        </footer>
      </article>
    </div>
  </div>
</section>
```

```css
/* @layer base, :root */
--font-size-heading-xs: var(--fs-600);

/* @layer components */
.card {
  display: flex;
  flex-direction: column;
  gap: var(--card-gap, 1rem);
  padding: 1rem;
  background-color: var(--background-light);
  border-radius: var(--border-radius-3);

  img {
    border-radius: var(--border-radius-2);
  }
}

.card__title {
  font-size: var(--card-title-font-size, var(--font-size-heading-sm));
  color: var(--card-title-color, var(--text-brand));
}

.card__footer {
  font-size: var(--font-size-sm);
  padding: 12px;
  background-color: var(--background-extra-light);
  border-radius: var(--border-radius-2);
  margin-top: auto;
}

.pills {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
  align-items: start;
}

/* @layer pages */
.mushroom-types {
  --card-title-font-size: var(--font-size-heading-xs);
  --card-title-color: var(--text-high-contrast);
}
```

### Otwarte punkty

- [ ] `align-items: start` na `.pills`: usunąć (nic teraz nie robi) czy zostawić jako zabezpieczenie? Jeśli zostaje, to z komentarzem, po co.
- [ ] Usunąć `align-content: start` z `.card`, jeśli jeszcze gdzieś zostało (we flexie w kolumnie bez wrap nie działa).
- [ ] Sprawdzić obrazki w kartach na `index.html` po zmianie `.card` na flex.
- [ ] Zadeklarować kolejność warstw na górze `style.css`.
- [ ] `<strong>` w stopce i ewentualnie `<i lang="la">` z nazwą łacińską.
- [ ] Sprawdzić `gap` między zawiniętymi rzędami pilli i zachowanie rzędu filtrów przy 320px.

### Pytania kontrolne

1. Po czym poznać, czy dwie karty to różne komponenty, czy ten sam komponent z inną zawartością?
2. Dlaczego `.card__title--mushroom-types` jest gorsze niż ustawienie `--card-title-color` na sekcji?
3. Dlaczego `.mushroom-types` nie jest modyfikatorem `.section`?
4. Reguła w `@layer components` ma selektor z `#id`, a reguła w `@layer pages` jedną klasę. Która wygra i dlaczego?
5. Dlaczego `.mushroom-types { color: white; }` nie zmieni koloru `.card__title`?
6. Po co osobny token `--font-size-heading-xs`, skoro `--font-size-lg` ma tę samą wartość?
7. `card__footer` czy `card__notes`, i dlaczego?
8. Jaki test rozstrzyga między `<article>` a `<div>`?
9. Czym różni się `align-items` od `align-content`?
10. Dlaczego `margin-top: auto` na stopce jest lepsze niż `grid-template-rows: auto auto 1fr auto` na karcie?
11. `<strong>`, `<b>`, `<em>` i `<i>`: kiedy który?

---

## Zawijająca się siatka kart: `.equal-columns-wrap` z `auto-fill` + `minmax`

**Data:** 2026-10-30 – 2026-11-08
**Projekt:** FungiFinders (`mushroom-guide.html`, sekcja „Get to know your mushrooms”)
**Status:** gotowe. Do zrobienia zostało potwierdzenie progów w DevTools.

### Problem

`.equal-columns` na desktopie ma `grid-auto-flow: column` i `grid-auto-columns: 1fr`, więc wszystkie dzieci lądują w jednym rzędzie. Przy 12 kartach grzybów to nie działa. Projekt (`design/`) pokazuje 4 karty w rzędzie przy 1440px i 1 kolumnę przy 425px. Tabletu w projekcie nie ma.

### Co jest wspólne, a co się różni

| | `.equal-columns` | `.equal-columns-wrap` |
|---|---|---|
| liczba dzieci | 2–3 | dowolna (12) |
| kolumny | tyle, ile dzieci, zawsze w jednym rzędzie | tyle, ile się zmieści |
| szerokość kolumn | równa | równa |
| zawijanie | ❌ | ✅ |
| media query | tak (`760px`) | nie |

### Decyzje i dlaczego

**1. Nie zmieniam bazy `.equal-columns`**
Pozostałe użycia (2 i 3 dzieci) by się zepsuły. Przy `repeat(4, 1fr)` sekcja z 2 dziećmi zajęłaby pół szerokości.

**2. Osobna klasa, a nie modyfikator `.equal-columns--...`**
Najpierw myślałem o modyfikatorze. Ale `.equal-columns` znaczy „każde dziecko dostaje równą kolumnę w dostępnym miejscu”, a tutaj kolumn jest mniej niż dzieci. To inny układ, a nie wariant tego samego. Wspólne jest tylko to, że kolumny są równe, zarówno w jednym rzędzie, jak i w kilku.

**3. Sztywne „4 kolumny” odrzucone**
Pierwsza wersja: zmienna `--equal-columns-wrap-template` ustawiana w `.mushroom-types` na `repeat(4, 1fr)` w media query. Problemy:
- między 425px a 1440px projekt nic nie mówi, a przeskok z 1 na 4 kolumny przy 760px dałby bardzo wąskie karty,
- każda inna sekcja z innymi kartami musiałaby przepisywać wszystkie media queries.

Wybrałem stopniowy wzrost liczby kolumn, kiedy jest miejsce.

**4. Minimalna sensowna szerokość karty: 270px**
270 nie wynika z tego, jak wygląda karta grzyba poniżej tej szerokości, tylko z tego, kiedy cały układ zaczyna wyglądać źle. Założenia: gap 16px i padding wrappera 2 × 16px na wszystkich szerokościach.

Progi:
```
4 kolumny: 4 × 270 + 3 × 16 + 2 × 16 = 1160px
3 kolumny: 3 × 270 + 2 × 16 + 2 × 16 = 874px
```
Poniżej progu karta musiałaby mieć mniej niż 270px, a minimum na to nie pozwala, więc liczba kolumn spada.

**5. Media queries zastąpione przez `repeat(auto-fill, minmax(270px, 1fr))`**
Progi same wynikają z minimum, więc nie trzeba ich wpisywać. Dzięki temu martwy blok `@media` z `--equal-columns-wrap-template` w `.mushroom-types` mogłem usunąć (zasada: zmienna to narzędzie na konkretny problem).

**6. 270px jako wartość domyślna w klasie layoutu, nie w `.mushroom-types`**
270 to ogólna minimalna szerokość karty w tym projekcie, a nie cecha grzybów. Sekcja z innymi kartami może nadpisać `--column-min-width` w swoim kontekście, bez zmian w klasie.

**7. Pomyłki po drodze**
- `minmax(--column-min-width, 1fr)`: zabrakło `var()`. Sama nazwa po prawej stronie dwukropka to dla przeglądarki zwykłe słowo, więc cała deklaracja jest nieprawidłowa (w DevTools przekreślona).
- Domyślne `--column-min-width: 1fr`: `fr` to ułamek *wolnego* miejsca, więc nie da się z niego policzyć, ile kolumn się zmieści. Jako minimum w `minmax()` jest niedozwolone.

**8. Dlaczego nigdy nie ma 5 kolumn**
5 kolumn potrzebuje `5 × 270 + 4 × 16 = 1414px`, a `.wrapper` ma maksymalnie 1130px (nawet wariant 1330px nie wystarcza). Liczbę kolumn ogranicza więc nie tylko `minmax`, ale też **szerokość kontenera**.

### Poznane techniki

**`grid-auto-flow`**
- `row` (domyślne): gdy skończą się kolumny, grid przechodzi do następnego rzędu.
- `column`: gdy skończą się rzędy, grid przechodzi do następnej kolumny. Przy jednym rzędzie wszystko ląduje obok siebie, i dlatego `.equal-columns` nie zawija.

**`auto-fill` ze stałą szerokością a z `minmax`**
```css
grid-template-columns: repeat(auto-fill, 270px);              /* kolumny przybywają, ale po prawej zostaje puste miejsce */
grid-template-columns: repeat(auto-fill, minmax(270px, 1fr)); /* kolumny przybywają i rozciągają się na całą szerokość */
```

**Ustawienie a odczytanie zmiennej**
```css
--column-min-width: 270px;            /* ustawienie: nazwa przed dwukropkiem */
minmax(var(--column-min-width), 1fr)  /* odczytanie: zawsze przez var() */
```

**`fr` nie może być minimum w `minmax()`**
`fr` dzieli to, co *zostanie*. Minimum musi być konkretną długością (px, rem…).

### Aktualny kod

```css
/* @layer layout */
.equal-columns-wrap {
  --column-min-width: 270px;

  display: grid;
  gap: 1rem;
  grid-template-columns: repeat(
    auto-fill,
    minmax(var(--column-min-width), 1fr)
  );
}

/* @layer pages */
.mushroom-types {
  --card-title-font-size: var(--font-size-heading-xs);
  --card-title-color: var(--text-high-contrast);
}
```

```html
<div class="equal-columns-wrap">
  <article class="card">…</article>
  <!-- … -->
</div>
```

### Otwarte punkty

- [ ] Potwierdzić progi ~1160px i ~874px w DevTools (plakietka `grid`).
- [ ] `gap: 1rem` jest wpisany na sztywno. Zmienna dopiero wtedy, gdy jakaś sekcja będzie potrzebowała innego.
- [ ] Rozważyć `auto-fit` zamiast `auto-fill`: czym się różnią, kiedy kart jest mniej niż kolumn?
- [ ] Przemianować eksporty w `design/` (`Guide - Desktop-1.png` to w rzeczywistości mobile).

### Pytania kontrolne

1. Dlaczego `.equal-columns` z `grid-auto-flow: column` nie zawija dzieci?
2. Dlaczego zawijająca się siatka to osobna klasa, a nie modyfikator `.equal-columns`?
3. Skąd wzięło się 270px i dlaczego nie mieszka w `.mushroom-types`?
4. Policz próg dla 3 kolumn (gap 16px, padding wrappera 2 × 16px).
5. Co jest źle w `minmax(--column-min-width, 1fr)`?
6. Dlaczego `1fr` nie może być minimum w `minmax()`?
7. Czym różni się `repeat(auto-fill, 270px)` od `repeat(auto-fill, minmax(270px, 1fr))`?
8. Dlaczego na monitorze 1920px nadal są 4 kolumny, a nie 5?

---

## Kolory pilli, grubości fontu i błędy w Figmie

**Data:** 2026-10-28 – 2026-10-29 (wpis dopisany 2026-11-08)
**Projekt:** FungiFinders (`mushroom-guide.html`, karty grzybów)
**Status:** modyfikatory gotowe, `.pill--fall` jeszcze nieużyte w HTML

### Problem

Pille w kartach mają różne kolory zależnie od treści: edible i summer są zielone, toxic czerwona, spring turkusowa, fall pomarańczowa. Do tego „Important notes:” w stopce karty jest pogrubione, a strona ładuje Fira Sans tylko w grubościach 400 i 500.

### Decyzje i dlaczego

**1. Nazwa modyfikatora według treści: `.pill--spring`, a nie `.pill--blue`**
`.pill--blue` kłamie po zmianie koloru i trzeba by go podmieniać w każdej pilli „spring”. Przy `.pill--spring` edytuję jedną regułę.

**2. Pozorna sprzeczność: `.pill--spring` jest OK, a `card--mushroom` nie**
Nazwa według treści jest w porządku tylko wtedy, gdy **wygląd ma sygnalizować tę treść**. Kolor pilli ma pokazywać porę roku. Projekt nie wskazuje natomiast, żeby wygląd karty sygnalizował „teraz opisujemy grzyba”. Karta to uniwersalne pudełko.

**3. Osobne modyfikatory nawet dla pilli, które dziś mają kolor bazowy**
`.pill--edible` i `.pill--summer` mają dziś ten sam kolor co `.pill`. Mają jednak własne klasy, bo kiedyś mogą dostać różne kolory, i wtedy nie trzeba będzie zmieniać HTML-a.

**4. Bezpośrednie nadpisanie `background-color`, bez zmiennej `--pill-background`**
Rozważałem `background-color: var(--pill-background, var(--background-accent-main))` w `.pill`. Odrzuciłem to, bo zmienna dotyczyłaby jednej właściwości, więc w modyfikatorze to i tak jedna linia. Zmienna „na zapas” wprowadzałaby w błąd, bo sugeruje problem, którego nie ma.

> Zasada: zmienna to narzędzie na konkretny problem, a nie domyślny sposób pisania CSS.

**5. Edible i summer na tokenie akcentu, z komentarzem**
Edible i summer używają `--background-accent-main`, bo w projekcie mają kolor akcentu. Ryzyko: zmiana akcentu zmieni też te pille i mogą wtedy np. zlać się z inną porą roku. Komentarz w kodzie mówi **dlaczego** i **na co uważać**, a nie **co** robi kod.

**6. Pogrubienie w Fira Sans domyślnie przez 500**
`strong` w `@layer base` dostaje `--font-weight-md` (500). Fira Sans 700 usunąłem z linku do Google Fonts, bo nic jej nie używało.

**7. Błędy w Figmie implementuję w wersji spójnej**
- „fall” w karcie Matsutake ma kolor „toxic”,
- w karcie King Bolete pogrubiona jest cała notatka, a nie tylko „Important notes:”.

Implementuję wersję spójną (fall pomarańczowa, pogrubiony tylko nagłówek notatki) i potwierdzam to z projektantem.

### Poznane techniki

**Algorytm dopasowania grubości fontu**
Gdy żądanej grubości nie ma:
- prośba o 400 lub 500, której brakuje: przeglądarka sprawdza najpierw drugą z tych wartości, potem lżejsze, potem cięższe (brak 500 → 400),
- prośba o więcej niż 500: najpierw cięższe, potem lżejsze (brak 700 → szuka w górę, potem w dół).

Jeśli prosimy o pogrubienie (600 lub więcej), a dopasowana grubość jest dużo lżejsza (np. 400), przeglądarka może zrobić **faux bold**, czyli sztucznie pogrubić glify. Wygląda to gorzej niż prawdziwa grubość.

**Tokeny surowe i znaczeniowe dla grubości**
```css
--fw-400: 400; --fw-500: 500; --fw-700: 700;  /* ile */
--font-weight-regular: var(--fw-400);         /* do czego */
--font-weight-md: var(--fw-500);
--font-weight-lg: var(--fw-700);
```

**Font zmienny**
Outfit ładuję jako zakres `wght@100..900`: jeden plik obsługuje każdą grubość. Dla Fira Sans wypisuję konkretne grubości (`wght@400;500`).

### Aktualny kod

```css
/* @layer base */
strong {
  font-weight: var(--font-weight-md);
}

/* @layer components */
/* background-color of pills edible and summer
should be coherent with background-accent-main
to represent accent.
Remember that changing accent might make these pills confusing.
*/
.pill--edible { background-color: var(--background-accent-main); }
.pill--summer { background-color: var(--background-accent-main); }
.pill--toxic  { background-color: var(--clr-red-500); }
.pill--spring { background-color: var(--clr-teal-500); }
.pill--fall   { background-color: var(--clr-orange-500); }
```

### Otwarte punkty

- [ ] Dokończyć 12 kart z projektu i użyć `.pill--fall` w HTML.
- [ ] Potwierdzić z projektantem oba błędy w Figmie.
- [ ] Znane ograniczenie (nie naprawiać): `strong` ma stałe 500, więc wewnątrz nagłówka (Outfit, 700) byłby *cieńszy* od otoczenia. W projekcie nie ma takiego przypadku.
- [ ] `align-items: start` na `.pills`: usunąć czy zostawić z komentarzem?
- [ ] Kontrast białego tekstu na `--clr-green-400` (WCAG AA 4.5:1).
- [ ] `cursor: pointer` na `.pill--select`?
- [ ] Deklaracja kolejności warstw na górze `style.css`.
- [ ] Ścieżki obrazków w HTML od korzenia (`/assets/...`) → względne.

### Pytania kontrolne

1. Dlaczego `.pill--spring` jest OK, skoro `card--mushroom` nie jest?
2. Po co `.pill--edible`, skoro ma ten sam kolor co `.pill`?
3. Dlaczego nie wprowadziłem `--pill-background`?
4. Co powinien mówić dobry komentarz w kodzie?
5. Strona ładuje Fira Sans 400 i 500. Którą grubość zobaczysz przy `font-weight: 700`, a którą przy `600`?
6. Co to jest faux bold i kiedy się pojawia?
7. Co robisz, gdy projekt w Figmie jest niespójny?

---

## Sekcja FAQ: Bento grid, obrazki wypełniające obszar oraz `align-items` a `align-content`

**Data:** 2026-11-08 – 2026-11-10
**Projekt:** FungiFinders (`mushroom-guide.html`, sekcja „Frequently Asked Questions”)
**Status:** gotowe (commit `54fdadd`)

### Problem

FAQ to 4 karty w układzie Bento: na desktopie 3 kolumny, karta 3 zajmuje dwa rzędy, a karta 4 dwie kolumny. Każda karta ma obrazek, ale w każdej stoi on inaczej:
- karta 2: obrazek **pod** tekstem,
- karta 3: obrazek ma wypełnić całą wolną wysokość karty, która jest wysoka na dwa rzędy,
- karta 4: obrazek **obok** tytułu i tekstu, na całą wysokość karty.

### Decyzje i dlaczego

**1. Spany tylko w układzie 3-kolumnowym (mobile-first)**
Na mobile karty stoją jedna pod drugą, więc `span` nie ma sensu. Wszystkie spany i wyjątki są w `@media (width > 760px)`.

**2. Komentarz: siatka jest zaprojektowana na dokładnie 4 karty**
Spany zależą od *pozycji* (`:nth-child`), a nie od treści. Przy innej liczbie kart siatka zostawi dziury albo dodatkowe rzędy. Komentarz mówi o tym ograniczeniu, a nie opisuje, co robi kod.

**3. Karta 2: `order: 1` na obrazku**
`.card` to flex w kolumnie, więc `order` przesuwa obrazek na koniec bez zmiany HTML-a. W HTML-u obrazek zostaje na początku, tak jak w pozostałych kartach. Kolejność czytania przez czytnik ekranu się nie zmienia, a obrazek nie jest treścią, którą trzeba czytać w określonej kolejności.

**4. Karta 3: `flex-grow: 1` + `object-fit: cover`**
Karta ma `grid-row: span 2`, więc jest wyższa niż jej treść. W karcie (flex w kolumnie) zostaje wolne miejsce na osi głównej.
- `margin-top: auto` zjada wolne miejsce *obok* elementu i go **przesuwa**.
- `flex-grow: 1` oddaje to miejsce *samemu elementowi*, który **rośnie**. Tego właśnie potrzebuję.

Obrazek urósł w pionie, ale ma tę samą szerokość, więc bez `object-fit` byłby zniekształcony.

**5. Karta 4: wewnętrzny grid `1fr 2fr` i `align-self: stretch` na obrazku**
Obrazek ma `grid-row: 1/3` (oba wiersze), a `h3` i `p` mają po jednym wierszu w drugiej kolumnie.

Przewidywanie było takie: obrazek wyższy niż tekst → dodatkowa wysokość dzieli się po równo między wiersze `auto` → tytuł odjeżdża od tekstu. DevTools pokazało coś innego: obrazek był **niższy** niż tekst i wisiał u góry z pustym miejscem pod spodem. **Najpierw liczby z DevTools, potem wnioski.**

Mimo to obrazek się nie rozciągał, chociaż `h3` i `p` się rozciągają. Przyczyna jest w technice nr 2 poniżej (`normal` ≠ `stretch`).

**6. Reguły obrazków tylko w blokach kart 3 i 4, a nie w `.card img`**
Tylko te dwie karty mają obrazek w obszarze, którego wysokość wyznacza coś innego. W kartach 1–2 i na `index.html` obrazki zachowują naturalne proporcje i nie ma czego wypełniać. Reguła należy do jedynego miejsca, które jej potrzebuje.

**7. Kadr bez `object-position`**
Po `cover` grzyb jest dobrze widoczny na każdej szerokości, więc domyślny środek wystarcza.

### Poznane techniki

**1. `align-content` a `align-items`: dwie warstwy, każda z własnym wolnym miejscem**
| | Działa na | Pytanie | Nic nie robi, gdy… |
|---|---|---|---|
| `align-content` | **wiersze** (tory) | jak wiersze dzielą miejsce w kontenerze? | nie ma wolnego miejsca w **kontenerze** (np. `height: auto`) |
| `align-items` | **elementy** | gdzie element stoi w swoim wierszu? | nie ma wolnego miejsca w **wierszu** (element jest najwyższy w wierszu) |

Kolejność: najpierw wiersze dostają wysokość (z treści i z `align-content: stretch`), potem `align-items` ustawia w nich elementy. Przy `stretch` element rozciąga się do **wysokości swojego wiersza**, a nie do najwyższego sąsiada czy kontenera.

Przykład z kart grzybów: `align-items: start` naprawiło pille, ale odstępy dalej rosły, bo wiersze nadal zabierały wolne miejsce karty. To była robota dla `align-content: start`.

Element rozpięty na kilka wierszy (`grid-row: 1/3`), który jest wyższy niż ich suma, **żąda** dodatkowej wysokości. Grid dzieli ją po równo między wiersze `auto`, przez które ten element przechodzi. To nie jest wolne miejsce w kontenerze, więc `align-content` tu nie pomoże. Pomogłoby wskazanie wiersza, który ma przejąć nadmiar, np. `grid-template-rows: auto 1fr`.

Demo: https://claude.ai/artifact/GJV43aiJVxRR37BPMgyKc3

**2. Domyślne `align-items` to `normal`, a nie `stretch`**
W gridzie `normal` działa jak:
- `stretch` dla zwykłych elementów (`h3`, `p`, `div`),
- `start` dla elementów z **własnymi proporcjami** (`img`, `video`, cokolwiek z `aspect-ratio`), bo rozciąganie by je zniekształciło.

Rozciągnięcie obrazka to świadoma decyzja: `align-self: stretch` (albo `height: 100%`).

**3. `object-fit`**
- `cover`: wypełnia cały prostokąt, zachowuje proporcje i przycina nadmiar.
- `contain`: mieści całe zdjęcie i zostawia puste paski.
- `object-position`: wybiera, która część zdjęcia zostaje po przycięciu (domyślnie środek). Składnia jak w `background-position`.

**4. Element z `position: absolute` nie wpływa na rozmiar wiersza ani na `gap`**
Błąd w demo: etykieta „wiersz 2” była wyższa niż pudełko z pillami, więc to ona wyznaczała wysokość wiersza i `stretch` miał co wypełniać. Po `position: absolute` etykieta wypadła z przepływu. To ten sam mechanizm co przy `.visually-hidden` we flexie.

**5. `> :nth-child(2) > img` a zagnieżdżone `img`**
`> img` łapie tylko bezpośrednie dziecko, a samo `img` łapie każdy obrazek w środku. Wszystkie trzy karty zapisane są teraz tak samo (zagnieżdżone `img`).

### Aktualny kod

```css
/* @layer pages */
.faq-cards {
  /* … */
  @media (width > 760px) {
    grid-template-columns: repeat(3, 1fr);

    > :nth-child(2) {
      img { order: 1; }
    }

    > :nth-child(3) {
      grid-row: span 2;

      img {
        flex-grow: 1;
        object-fit: cover;
      }
    }

    > :nth-child(4) {
      grid-column: span 2;
      display: grid;
      grid-template-columns: 1fr 2fr;
      grid-template-rows: repeat(2, auto);

      img {
        grid-row: 1/3;
        grid-column: 1/2;
        align-self: stretch;
        object-fit: cover;
      }
      h3 { grid-row: 1/2; grid-column: 2/3; }
      p  { grid-row: 2/3; grid-column: 2/3; }
    }
  }
}
```

### Otwarte punkty

- [ ] `align-items: start` na `.pills`: w kartach grzybów `.card` jest teraz flexem w kolumnie. Czy w wierszu pilli jest jeszcze wolne miejsce do wypełniania? Jeśli nie, reguła nic nie robi.
- [ ] Mobile: porównać sekcję FAQ z projektem (`Guide - Desktop-1.png` to mobile).
- [ ] Karta 3 przy ~761px: czy wąska, wysoka karta nie przycina zdjęcia za mocno?

### Pytania kontrolne

1. `align-content` czy `align-items`: która właściwość działa na wiersze, a która na elementy?
2. Kiedy `align-content` nic nie robi? A kiedy `align-items`?
3. Przy `align-items: stretch`: do jakiej wysokości rozciąga się element?
4. Dlaczego w karcie grzyba `align-items: start` nie usunęło rosnących odstępów?
5. Obrazek zajmuje dwa wiersze `auto` i jest o 150px wyższy niż ich treść. Gdzie trafiają te piksele i dlaczego `align-content: start` tego nie zmieni?
6. Dlaczego `img` w gridzie nie rozciąga się domyślnie, skoro `h3` się rozciąga?
7. `margin-top: auto` czy `flex-grow: 1`: które przesuwa, a które powiększa?
8. `object-fit: cover` a `contain`?
9. Dlaczego reguły obrazków są w blokach kart 3 i 4, a nie w `.card img`?

---

## FAQ zgodnie z Figmą: ramka na obrazek, `aspect-ratio` i zoom ponad `cover`

**Data:** 2026-11-11 – 2026-11-13
**Projekt:** FungiFinders (`mushroom-guide.html`, sekcja „Frequently Asked Questions”)
**Status:** ramki gotowe (commit `3a72eca`), zoom karty 1 gotowy, ale jeszcze niezacommitowany. Karty 3 i 4 czekają.

### Problem

Porównanie FAQ z Figmą wykazało trzy różnice:
- tytuły kart są większe (`--fs-700`, a nie `--fs-600`),
- odstęp między kartami jest inny na mobile i na desktopie,
- zdjęcia w kartach 1, 3 i 4 są w Figmie **powiększone ponad `cover`** i przesunięte. Kadr pokazuje mniejszy fragment zdjęcia niż samo `object-fit: cover`.

Dane karty 1 z Figmy:
| | kadr (ramka) | warstwa zdjęcia | pozycja |
|---|---|---|---|
| mobile | 361 × 240 (3:2) | 437 × 328 (4:3) | X = 0% (76px nadmiaru w prawo), Y = 50% (po 44px góra i dół) |
| desktop | 395 × 240 (~1.64) | 437 szer. | X = 0% |

### Decyzje i dlaczego

**1. Tytuł FAQ: `--font-size-heading-sm` zamiast `-xs`**
Zgodność z Figmą: tytuł ma `--fs-700`. Zmieniłem tylko wartość custom property na `.faq-cards`. Komponent `.card` pozostał bez zmian.

**2. `gap`: 1.5rem na mobile, 1rem na desktopie**
Figma na desktopie jest niespójna (14, 17 i 24px). Uznałem to za błąd projektu i wybrałem 1rem. Na mobile Figma jest spójna i ma 24px. Baza to mobile, a desktop nadpisuje wartość w `@media (width > 760px)`.

**3. Kolejność kart na mobile: 1, 2, 4, 3**
`> :nth-child(3) { order: 1 }` w bazie, a w `@media` z powrotem `order: 0`. Zmienia się tylko kolejność wizualna, HTML zostaje taki sam.

**4. Ramka `.card__img-frame` wokół każdego `img`**
Powiększone zdjęcie wystaje poza swój prostokąt, więc musi go przyciąć coś *na zewnątrz*: rodzic z `overflow: hidden`. Ramka przejęła też `border-radius`, bo przy powiększonym zdjęciu zaokrąglenie musi być na tym, co przycina.

Karta 2 w Figmie nie ma zoomu, ale też dostała ramkę. Każda karta ma wtedy tę samą strukturę HTML, a reguły kart nie muszą sprawdzać, czy ramka istnieje.

**5. Podział reguł: ogólnie w komponencie, wyjątki w sekcji**
- `@layer components`, `.card__img-frame`: `overflow: hidden`, `border-radius` oraz `img { width: 100%; height: 100%; object-fit: cover }`. Obowiązuje każdą ramkę.
- `@layer pages`, `.faq-cards`: `aspect-ratio`, zoom i układ kart 2/3/4.

**6. Reguły układu przeniesione z `img` na ramkę**
`order` (karta 2), `flex-grow` (karta 3) oraz `grid-row/column` + `align-self` (karta 4) działają na **dziecko** kontenera flex/grid. Dzieckiem karty jest teraz ramka, a nie `img`. `img` wypełnia ramkę przez `width/height: 100%`.

Pytanie, które się po drodze pojawiło: dlaczego `align-self: stretch` nie rozciągał ramki w pionie w karcie 2? `.card` to flex **w kolumnie**, więc `align-self` działa na osi poprzecznej, czyli w **poziomie**. Wzrost w pionie (oś główna) daje `flex-grow: 1`.

**7. Zoom: mobile w bazie, desktop nadpisuje tylko różnice**
Rozważałem dwie drogi: (A) baza = mobile, a `@media (width > 760px)` nadpisuje; (B) osobne `@media` dla mobile i desktopu. Wybrałem A, bo cała `.faq-cards` jest już zbudowana mobile-first.

Na desktopie zdjęcie też jest przyklejone do lewej, więc `object-position` i `transform-origin` przechodzą z bazy. Nadpisuję tylko `aspect-ratio` i `scale`.

To zastępuje decyzję nr 7 z poprzedniego wpisu („kadr bez `object-position`”). Wtedy nie porównywałem kadru z Figmą.

**8. `aspect-ratio: 395 / 240` zamiast `1.64`**
Wymiary z Figmy w ułamku pokazują, skąd wzięła się liczba. `1.64` byłoby magiczną liczbą.

Na początku martwiłem się, że szerokość ramki zależy od szerokości karty, więc nie znam rozmiarów. To nie ma znaczenia: `aspect-ratio` to proporcja, a `scale` to mnożnik. Przy każdej szerokości karty ramka ma te same proporcje, więc `cover` + `scale` tną zdjęcie tak samo.

### Poznane techniki

**1. `cover` = najmniejsze powiększenie bez pustych miejsc**
`cover` sprawdza dwa sposoby dopasowania i wybiera ten, który zakrywa całą ramkę:
- ramka **szersza** proporcjonalnie niż zdjęcie → dopasowuje **szerokość**, a zdjęcie wystaje w pionie,
- ramka **węższa** proporcjonalnie → dopasowuje **wysokość**, a zdjęcie wystaje na boki.

Przykład: ramka 395 × 240, zdjęcie 4:3. Po szerokości wychodzi 395 × 296 i ramka jest zakryta ✅. Po wysokości wychodzi 321 × 240 i z prawej zostaje dziura ❌.

**2. Zoom ponad `cover`: `scale` = warstwa / kadr na osi, którą dopasowuje `cover`**
U nas ramka (1.5 i ~1.64) jest szersza niż zdjęcie (1.33), więc `cover` dopasowuje szerokość. Szerokość zdjęcia po `cover` = szerokość ramki:
- mobile: 437 / 361 ≈ **1.21**
- desktop: 437 / 395 ≈ **1.11** (najpierw zaokrągliłem do 1.10, a to za mało dokładnie)

Przy ramce węższej niż zdjęcie trzeba by dzielić **wysokości**.

**3. `transform-origin` = te same procenty co `object-position`**
`scale` powiększa wokół punktu, który stoi w miejscu. Domyślnie jest to środek (`50% 50%`). Przy domyślnym punkcie zdjęcie urosłoby we wszystkie strony, a lewa krawędź, przypięta przez `object-position: 0% 50%`, wyjechałaby poza ramkę.

`transform-origin: 0% 50%` zostawia w miejscu punkt na lewej krawędzi, w połowie wysokości. Cały nadmiar w poziomie idzie w prawo, a w pionie dzieli się po równo. Przykład: zdjęcie przyklejone do prawej i do góry → `100% 0%` w **obu** właściwościach.

Demo: https://claude.ai/artifact/VXQqefLc9ikUhJMZpkjQh6

### Aktualny kod

```css
/* @layer components */
.card__img-frame {
  overflow: hidden;
  border-radius: var(--border-radius-2);

  img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
}

/* @layer pages */
.faq-cards {
  --card-title-font-size: var(--font-size-heading-sm);
  --card-title-color: var(--text-high-contrast);

  display: grid;
  grid-template-columns: 1fr;
  gap: 1.5rem;

  > :first-child {
    .card__img-frame {
      aspect-ratio: 3/2;

      img {
        object-position: 0% 50%;
        scale: 1.21;
        transform-origin: 0% 50%;
      }
    }
  }

  > :nth-child(3) { order: 1; }

  @media (width > 760px) {
    grid-template-columns: repeat(3, 1fr);
    gap: 1rem;

    > :first-child {
      .card__img-frame {
        aspect-ratio: 395 / 240;
        img { scale: 1.11; }
      }
    }

    > :nth-child(2) {
      .card__img-frame { order: 1; flex-grow: 1; }
    }

    > :nth-child(3) {
      order: 0;
      grid-row: span 2;
      .card__img-frame { flex-grow: 1; }
    }

    > :nth-child(4) {
      grid-column: span 2;
      display: grid;
      grid-template-columns: 1fr 2fr;
      grid-template-rows: repeat(2, auto);

      .card__img-frame { grid-row: 1/3; grid-column: 1/2; align-self: stretch; }
      h3 { grid-row: 1/2; grid-column: 2/3; }
      p  { grid-row: 2/3; grid-column: 2/3; }
    }
  }
}
```

### Otwarte punkty

- [ ] Zoom i kadr kart 3 i 4 według Figmy (karta 2 nie ma zoomu).
- [ ] Punkty z poprzedniego wpisu: `align-items: start` na `.pills` oraz karta 3 przy ~761px.

### Pytania kontrolne

1. Po co ramka z `overflow: hidden`, skoro `img` ma już `object-fit: cover`?
2. Dlaczego `border-radius` jest na ramce, a nie na `img`?
3. Dlaczego `order`/`flex-grow` przeszły z `img` na ramkę?
4. Karta jest flexem w kolumnie. Dlaczego `align-self: stretch` nie rozciąga ramki w pionie?
5. Ramka 200 × 400, zdjęcie 4:3. Co dopasuje `cover`, szerokość czy wysokość?
6. Jak policzyć `scale` ponad `cover` z wymiarów kadru i warstwy w Figmie?
7. Co się stanie z lewą krawędzią zdjęcia przy `object-position: 0% 50%` i `scale: 1.2`, ale bez `transform-origin`?
8. Zdjęcie przyklejone do prawej i do góry: jakie wartości w `object-position` i `transform-origin`?
9. Dlaczego szerokość karty nie wpływa na kadr, skoro ramka ma zmienną szerokość?
