# Handoff: kontynuacja nauki CSS (projekt FungiFinders)

> Wklej całość jako pierwszą wiadomość w nowej rozmowie. Nowa sesja nie ma dostępu do moich plików, dlatego aktualny kod jest wklejony poniżej.

---

## 1. Jak masz ze mną pracować (najważniejsze)

- **Pisz po polsku.** Kod, nazwy klas i treść HTML zostają po angielsku.
- **Zawsze tryb sokratejski.** Prowadź mnie pytaniami, jeden temat na raz. Nie pisz za mnie HTML-a ani CSS-a i nie dawaj gotowych rozwiązań.
- **Przy przeglądzie mojego kodu:** najpierw powiedz, co jest dobre, a potem każdy problem zamień w pytanie wskazujące konkretne miejsce. Nie podawaj gotowej poprawki.
- **Kiedy proszę „wytłumacz” albo „podaj przykład”, albo mówię, że nie ma sensu zgadywać:** wyjaśnij wprost i krótko, a na końcu zadaj pytanie kontrolne. Nie każ mi zgadywać nazw, których nie znam (np. nazwy elementu HTML albo nazwy warstwy).
- **Nie mieszaj tematów w jednym punkcie.** Wcześniej mnie to zgubiło. Jeden punkt to jedno pytanie.
- **Pilnuj zasad, które sam ustaliłem** (sekcja 3). Jeśli proponuję coś sprzecznego z nimi, pokaż mi to moimi słowami.
- **Prowadzę dziennik deliberate practice** (`notes/pro-css-patterns.md`). Kiedy poproszę o wpis, dostosuj go do tej struktury: tytuł, data/projekt/status, problem, tabela „co wspólne / co różne” (jeśli pasuje), numerowane decyzje z uzasadnieniem (razem z odrzuconymi opcjami i pomyłkami), poznane techniki z kodem, aktualny kod, otwarte punkty, pytania kontrolne. Wpisy po polsku.

---

## 2. Projekt

- Kurs „Pro CSS” (Kevin Powell, „website from scratch”), strona **FungiFinders**, projekt w Figmie.
- Pliki: `index.html` (strona 1), `mushroom-guide.html` (strona 2), jeden `style.css`, `assets/`, `notes/`.
- `style.css` ma warstwy w kolejności: `reset → base → layout → components → pages → utilities` (kolejność wynika z kolejności w pliku, nie ma deklaracji `@layer ...;` na górze).
- Konwencja nazw: BEM (`.card`, `.card__title`, `.pill--select`).
- Tokeny: surowe (`--clr-*`, `--fs-*`, `--fw-*`) i znaczeniowe (`--text-*`, `--background-*`, `--font-size-*`, `--font-weight-*`).
- Fonty: Fira Sans (`--ff-body`, ładowane 400 i 500) i Outfit (`--ff-heading`, zakres 100..900).
- Breakpoint: `760px`.

---

## 3. Zasady, które już ustaliłem (z uzasadnieniem)

1. **Element HTML wybieram po tym, co robi, a nie jak wygląda.** Filtr to `<select>`, tag to `<li>` w `<ul>`, a link to `<a>`, choć wygląda jak przycisk.
2. **Klasa to wygląd, element to znaczenie.** `.card` stoi na `<div>` (strona 1) i na `<article>` (strona 2).
3. **Nazwy modyfikatorów według znaczenia, nie według wyglądu ani miejsca.** `.pill--poisonous`, a nie `.bg-red`. `.pill--select`, a nie `.pill--filters`. Test: czy nazwa zostanie prawdziwa po zmianie koloru albo przeniesieniu elementu?
4. **Nazwa według treści jest OK tylko wtedy, gdy wygląd ma sygnalizować tę treść.** `.pill--spring` jest OK, bo kolor ma pokazywać porę roku. `card--mushroom` nie jest OK, bo karta to uniwersalne pudełko.
5. **Nazwy elementów BEM według roli, nie treści:** `card__footer`, a nie `card__notes`.
6. **`.card` to pudełko, nie przepis na zawartość.** Nie wie, co ma w środku. Żadnego `grid-template-rows` zależnego od liczby dzieci.
7. **Kontekst ustawia zmienne, a komponent je czyta** (`var(--x, domyślna)`). Komponent nie wie o sekcjach.
8. **Nie tworzę zmiennych ani plików „na zapas”.** Zmienna jest narzędziem na konkretny problem. Przykłady: `--pill-background` odrzucone, Fira Sans 700 usunięte z linku.
9. **Tokeny znaczeniowe, nie surowe, w komponentach i stronach.** Dwa tokeny mogą wskazywać tę samą wartość, jeśli znaczą co innego (`--font-size-heading-xs` i `--font-size-lg` → `--fs-600`).
10. **`sm`/`md`/`lg` mówią o kolejności, a nie o odstępie.**
11. **Warstwa `pages`** jest na style związane z jedną stroną lub sekcją, które nie są komponentem wielokrotnego użytku.
12. **Zmian potrzebnych w jednym miejscu nie robię w bazie wspólnej klasy.**
13. **Bez atrybutu `style` w HTML.** HTML to struktura, CSS to wygląd.
14. **Błędy w Figmie** (niespójności) implementuję w wersji spójnej i potwierdzam z projektantem.

---

## 4. Aktualny kod (stan na 2026-10-27)

### Tokeny (`@layer base`, `:root`, wybór)
```css
--clr-green-400: hsl(143, 19%, 49%);
--clr-green-500: hsl(143, 38%, 37%);
--clr-orange-500: hsl(28, 43%, 28%);
--clr-red-500: hsl(359, 34%, 24%);
--clr-teal-500: hsl(186, 42%, 25%);

--fs-300: 0.875rem; --fs-400: 1rem; --fs-500: 1.125rem; --fs-600: 1.25rem;
--fs-700: 1.5rem;   --fs-800: 2rem; --fs-900: 3.75rem;  --fs-1000: 3.75rem;
/* @media (width > 760px): fs-500..fs-1000 większe */

--fw-400: 400; --fw-500: 500; --fw-700: 700;

--text-high-contrast: var(--clr-white);
--background-accent-light: var(--clr-green-400);
--background-accent-main: var(--clr-green-500);
--background-extra-light: var(--clr-brown-500);
--background-light: var(--clr-brown-600);

--font-size-heading-xs: var(--fs-600);
--font-size-heading-sm: var(--fs-700);
--font-size-heading-regular: var(--fs-800);
--font-size-sm: var(--fs-300);
--font-size-regular: var(--fs-400);
--font-size-md: var(--fs-500);
--font-size-lg: var(--fs-600);

--font-weight-regular: var(--fw-400);
--font-weight-md: var(--fw-500);
--font-weight-lg: var(--fw-700);
```

```css
/* @layer base */
b, th, summary, strong {
  font-weight: var(--font-weight-md);
}
```

### Layout (`@layer layout`): tu jest bieżący temat
```css
.equal-columns {
  --equal-columns-gap: 1rem;

  display: grid;
  gap: var(--equal-columns-gap, 1rem);
  align-items: var(--equal-columns-vertical-alignment, stretch);

  @media (width > 760px) {
    grid-auto-flow: column;
    grid-auto-columns: 1fr;
  }
}

.equal-columns--gap-lg {
  --equal-columns-gap: 2rem;
}

.equal-columns--vertical-alignment {
  --equal-columns-vertical-alignment: center;
}
```

Gdzie jest używane `.equal-columns`:
| Miejsce | Ile dzieci |
|---|---|
| `index.html`, karty „Outdoor Exploration” itd. | 3 |
| `index.html`, dwie sekcje tekst + obrazek (z `--gap-lg` i `--vertical-alignment`) | 2 |
| `mushroom-guide.html`, sekcja tekst + obrazek | 2 |
| `mushroom-guide.html`, karty grzybów (`.mushroom-types`) | docelowo 12 |

### Komponenty (`@layer components`)
```css
.card {
  display: flex;
  flex-direction: column;
  gap: var(--card-gap, 1rem);
  padding: 1rem;
  background-color: var(--background-light);
  border-radius: var(--border-radius-3);

  img { border-radius: var(--border-radius-2); }
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

### Strony (`@layer pages`)
```css
.mushroom-types {
  --card-title-font-size: var(--font-size-heading-xs);
  --card-title-color: var(--text-high-contrast);
}
```

### HTML sekcji grzybów (`mushroom-guide.html`, skrócone)
```html
<section class="section mushroom-types">
  <div class="wrapper flow">
    <h2 class="section-title">Get to know your mushrooms</h2>
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
    <div class="equal-columns">
      <article class="card">
        <h3 class="card__title">Chanterelle</h3>
        <ul role="list" class="pills">
          <li class="pill pill--edible">edible</li>
          <li class="pill pill--summer">summer</li>
        </ul>
        <p>Golden-yellow, funnel-shaped mushroom with false gills</p>
        <footer class="card__footer">
          <strong>Important notes:</strong>
          Has toxic look-alikes - learn proper identification
        </footer>
      </article>
      <!-- ... na razie 5 kart, docelowo 12 -->
    </div>
  </div>
</section>
```

---

## 5. Bieżący temat: zawijanie kart grzybów po 4 w rzędzie na desktopie

**Problem:** `.equal-columns` na desktopie ma `grid-auto-flow: column` i `grid-auto-columns: 1fr`, więc wszystkie dzieci lądują w jednym rzędzie. Przy 12 kartach grzybów to nie działa. Chcę maksymalnie 4 w rzędzie, a reszta ma przechodzić do kolejnych rzędów.

**Co już ustaliłem:**
- Nie zmieniam bazy `.equal-columns`, bo pozostałe użycia (2 i 3 dzieci) by się zepsuły. Np. przy `repeat(4, 1fr)` sekcja z 2 dziećmi zajęłaby pół szerokości.
- „Po 4 w rzędzie” to nie jest cecha grzybów, tylko ogólne zachowanie układu. Dlatego to **modyfikator `.equal-columns--...` w `@layer layout`**, a nie reguła w `@layer pages`.
- Sama nazwa `equal-columns` nie wyklucza zawijania (mówi tylko „równe kolumny”).

**Otwarte pytania, na których skończyliśmy (prowadź mnie przez nie po kolei):**
1. **Nazwa modyfikatora:** jakie słowo opisuje różnicę względem bazy (zawijanie po określonej liczbie)? Według znaczenia, nie miejsca.
2. **Gdzie zapisać „4”:** w nazwie (`--max-4`) czy w zmiennej z wartością domyślną, wzorem `--equal-columns-gap` i `--equal-columns-vertical-alignment`? Jak to się ma do mojej zasady „bez zmiennych na zapas”?
3. **Szerokość ok. 800px** (tuż powyżej breakpointu 760px): czy 4 karty obok siebie są tam czytelne? Czy w Figmie jest projekt na tablet? Od tego zależy, czy wystarczy sztywne „4 kolumny”, czy potrzebne jest coś, co samo dobiera liczbę kolumn do szerokości (np. `repeat(auto-fit, minmax(...))`). **Nie podawaj mi tej techniki od razu, naprowadź.**
4. Po implementacji: sprawdzić, czy `grid-auto-flow: column` z bazy nie przeszkadza modyfikatorowi, i czy pozostałe użycia `.equal-columns` wyglądają bez zmian.

---

## 6. Pozostałe otwarte punkty (na później)

- [ ] Dokończyć treść kart z Figmy: są 4 unikalne karty + zduplikowana „Death Cap”, docelowo 12. Brak jeszcze `.pill--fall` w HTML.
- [ ] Niespójności w Figmie do potwierdzenia z projektantem: (a) „fall” w karcie Matsutake ma kolor „toxic”, (b) w karcie King Bolete pogrubiona jest cała notatka, a nie tylko „Important notes:”. Implementuję wersję spójną.
- [ ] `align-items: start` na `.pills`: usunąć (dziś nic nie robi) czy zostawić z komentarzem jako zabezpieczenie?
- [ ] Kontrast białego tekstu na `--clr-green-400` (hover/fokus selectów i `.button`): sprawdzić w DevTools względem WCAG AA 4.5:1. Odłożone.
- [ ] `cursor: pointer` na `.pill--select`? (`.button` go ma)
- [ ] Zadeklarować kolejność warstw na górze `style.css`: `@layer reset, base, layout, components, pages, utilities;`
- [ ] W HTML obrazki mają ścieżki od korzenia (`src="/assets/..."`). Przy otwarciu z dysku albo w podkatalogu (GitHub Pages) się zepsują. Tę samą lekcję przerobiłem już dla `url()` w CSS.
- [ ] Znane ograniczenie (zapisać w dzienniku, nie naprawiać): `strong` ma stałe 500, więc wewnątrz nagłówka (700, Outfit) byłby *cieńszy*. W projekcie nie ma takiego przypadku.
- [ ] **Wpis do dziennika** (jeszcze niezrobiony) o: modyfikatorach kolorów pilli (nazwa według znaczenia vs `card--mushroom`, zmienna vs bezpośrednie nadpisanie, edible/summer na tokenie akcentu i ryzyko kolizji z `fall`, dobry komentarz mówi „dlaczego” i „na co uważać”), grubościach fontu (`<strong>`/`<b>`/`<em>`/`<i>`, algorytm dopasowania grubości: brak 500 → 400, brak 700 → szuka w górę, potem w dół; faux bold; tokeny `--fw-*` i `--font-weight-*`; usunięcie Fira Sans 700), błędach w Figmie, oraz o module zawijania `.equal-columns`, kiedy go skończę.

---

## 7. Gdzie skończyliśmy

Ostatnie pytanie, które mi zadano: nazwa modyfikatora i gdzie zapisać „4” (sekcja 5, pytania 1 i 2), plus pytanie o szerokość ~800px. Zacznij od tego.
