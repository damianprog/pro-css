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
