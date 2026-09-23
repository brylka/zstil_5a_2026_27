# React – wstęp

**Technik programista | klasa 5 | materiał 05**

---

## Co zbudujemy

Stronę wyświetlającą listę produktów: karty z nazwą, ceną i dostępnością, wyszukiwarkę,
filtr kategorii i widok szczegółów po kliknięciu. Wszystko działa w przeglądarce, bez
przeładowania strony.

**Backendu w tym materiale nie uruchamiamy.** Dane będą wpisane na sztywno w pliku
`data.js`.

### Dlaczego znowu dane na sztywno

Bo to dokładnie ta sama sztuczka, którą zrobiliśmy w materiale 03 – tylko z drugiej strony.

```
03:  API na liście w pamięci   ---> 04:  to samo API, ale na bazie
05:  React na liście w pliku   ---> 07:  ten sam React, ale na API
```

W materiale 03 uczyłeś się API bez bazy danych, żeby jeden problem nie mieszał się
z drugim. Teraz uczysz się Reacta bez sieci, asynchroniczności i CORS-u – z tego samego
powodu. Kiedy w materiale 07 podmienisz źródło danych, **komponenty nie zmienią się ani
o linijkę**, bo dostaną dokładnie te same obiekty.

Jest w tym też lekcja o pracy zespołowej: frontend można napisać, zanim backend powstanie.
Wystarczy uzgodniony kontrakt. W firmach robi się to codziennie – i dlatego dane, które za
chwilę wkleisz do `data.js`, skopiujesz **z własnego API**, a nie wymyślisz od nowa.

### Wersje

| | Wersja | Uwaga |
|---|---|---|
| Node.js | 24 LTS | `node --version` |
| React | 19 | najnowszy major, obecnie 19.2.x |
| Vite | 8 | od wersji 8 pod spodem działa Rolldown |

---

## Krok 0 – skąd wziąć dane

Uruchom **na chwilę** backend z materiału 04 i wejdź na `http://localhost:8000/api/products/`.
Skopiuj całą odpowiedź do schowka. Zaraz się przyda.

Potem możesz go zatrzymać – do końca tego materiału nie będzie potrzebny.

---

## Krok 1 – utworzenie projektu

Wróć do katalogu głównego repozytorium (tego, w którym leży `backend/`):

```bash
cd sklep-fullstack
npm create vite@latest frontend -- --template react
cd frontend
npm install
npm run dev
```

Otwórz **http://localhost:5173** – zobaczysz stronę startową Vite z licznikiem.

### Co się właśnie stało

`npm create vite@latest` pobiera i uruchamia generator projektu. Argument `--template react`
mówi, że chcemy React w czystym JavaScripcie (jest jeszcze wariant `react-ts`
z TypeScriptem – w tym projekcie go nie używamy). Podwójny myślnik `--` przed `--template`
oddziela argumenty dla `npm` od argumentów dla generatora; bez niego npm zjadłby je
dla siebie.

`npm install` czyta `package.json` i pobiera wszystkie zależności do `node_modules/`.
To potrwa i utworzy kilkadziesiąt tysięcy plików – dlatego `node_modules/` jest
w `.gitignore` od materiału 01 i **nigdy** nie trafia do repozytorium. Odtwarza się
jedną komendą.

**Vite** to serwer deweloperski i narzędzie budujące. Podczas pracy podaje pliki
przeglądarce niemal natychmiast i podmienia je w locie – zapisujesz plik, strona
aktualizuje się sama, bez odświeżania i bez utraty stanu. Przy budowaniu wersji
produkcyjnej pakuje wszystko w kilka zoptymalizowanych plików.

---

## Krok 2 – co jest w projekcie

```
frontend/
├── index.html          ← jedyna strona HTML w całej aplikacji
├── package.json        ← zależności i skrypty (odpowiednik requirements.txt)
├── vite.config.js      ← konfiguracja Vite
├── eslint.config.js    ← reguły sprawdzania kodu
├── node_modules/       ← pobrane biblioteki (poza repozytorium)
├── public/             ← pliki kopiowane bez zmian (favicon itp.)
└── src/
    ├── main.jsx        ← punkt wejścia, montuje aplikację w HTML-u
    ├── App.jsx         ← główny komponent
    ├── App.css
    ├── index.css
    └── assets/
```

Otwórz `index.html`. Zwróć uwagę, że **nie ma tam prawie nic**:

```html
<div id="root"></div>
<script type="module" src="/src/main.jsx"></script>
```

Cała strona jest generowana przez JavaScript i wstawiana do tego jednego pustego `<div>`.
Tak działa **SPA** – Single Page Application. Serwer wysyła jeden pusty szkielet, a resztą
zajmuje się przeglądarka. To dlatego nawigacja w takich aplikacjach jest natychmiastowa:
nie ma kolejnych żądań o całe strony.

`src/main.jsx`:

```jsx
createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>,
)
```

Ten plik znajduje `<div id="root">` i wstawia do niego komponent `App`. Zapamiętaj
`StrictMode` – w trybie deweloperskim celowo renderuje komponenty **dwa razy**, żeby
wykryć błędy. W materiale 07 spowoduje to, że zobaczysz podwójne zapytania do API
i pomyślisz, że coś zepsułeś. Nie zepsujesz.

---

## Krok 3 – pierwszy własny komponent

Wyczyść `src/App.css` i `src/index.css` (na razie zostaw puste), a `src/App.jsx` zastąp
w całości:

```jsx
function App() {
  const title = "Katalog produktów";

  return (
    <main>
      <h1>{title}</h1>
      <p>Tu za chwilę pojawi się lista.</p>
    </main>
  );
}

export default App;
```

Zapisz i spójrz na przeglądarkę – zmiana jest widoczna od razu, bez odświeżania.

**Komponent to funkcja zwracająca opis interfejsu.** Tyle. Nazwa **musi** zaczynać się
wielką literą, inaczej React potraktuje ją jak zwykły znacznik HTML i nic nie wyświetli.

---

## Krok 4 – JSX, czyli HTML w JavaScripcie

To, co komponent zwraca, wygląda jak HTML, ale jest **JSX-em** – składnią, którą Vite
zamienia na wywołania funkcji JavaScriptu. Stąd różnice, które trzeba znać:

| HTML | JSX | Dlaczego |
|---|---|---|
| `class="card"` | `className="card"` | `class` jest słowem kluczowym JavaScriptu |
| `for="pole"` | `htmlFor="pole"` | to samo, `for` to pętla |
| `onclick="..."` | `onClick={...}` | zdarzenia pisane wielbłądzią notacją |
| `<br>` | `<br />` | każdy znacznik musi być domknięty |
| `<!-- komentarz -->` | `{/* komentarz */}` | komentarz to wyrażenie JS |

Dwie reguły dodatkowe:

**Jeden element nadrzędny.** Komponent nie może zwrócić dwóch elementów obok siebie.
Gdy nie chcesz dokładać niepotrzebnego `<div>`, użyj pustego znacznika – to
*fragment*:

```jsx
return (
  <>
    <h1>Tytuł</h1>
    <p>Treść</p>
  </>
);
```

**Klamry osadzają JavaScript.** W `{ }` możesz wstawić dowolne **wyrażenie**: zmienną,
działanie, wywołanie funkcji. Nie możesz wstawić instrukcji sterujących – `if` ani `for`
tam nie zadziałają.

```jsx
<p>Cena netto: {price} zł, brutto: {price * 1.23} zł</p>
<p>Dodano: {new Date().getFullYear()}</p>
```

---

## Krok 5 – dane

Utwórz **`src/data.js`** i wklej odpowiedź skopiowaną z własnego API. Dopisz tylko
`export const PRODUCTS =` na początku i średnik na końcu:

```js
export const PRODUCTS = [
  {
    id: 1,
    name: "Klawiatura mechaniczna K80",
    description: "Przełączniki brązowe, układ TKL.",
    price: "349.00",
    stock: 12,
    category: "peripherals",
    created_at: "2026-09-01T09:12:44.812000+00:00",
  },
  {
    id: 2,
    name: "Mysz bezprzewodowa Swift",
    description: "Sensor 16000 DPI, łączność 2.4 GHz.",
    price: "129.99",
    stock: 40,
    category: "peripherals",
    created_at: "2026-09-01T09:14:02.401000+00:00",
  },
  {
    id: 3,
    name: "Słuchawki nauszne Cliff",
    description: "Redukcja szumów, 30 h pracy.",
    price: "289.00",
    stock: 0,
    category: "audio",
    created_at: "2026-09-01T09:15:37.128000+00:00",
  },
];
```

**Zwróć uwagę na `price`.** To tekst w cudzysłowie, nie liczba – tak wysyła go Twój
backend, bo cena jest typem `Decimal`. Frontend musi się z tym liczyć. Za chwilę to
obsłużymy.

**`export`** udostępnia zmienną innym plikom. Bez tego słowa `data.js` byłby wyspą.

---

## Krok 6 – formatowanie ceny

Utwórz **`src/format.js`**:

```js
export function formatPrice(price) {
  return new Intl.NumberFormat("pl-PL", {
    style: "currency",
    currency: "PLN",
  }).format(Number(price));
}
```

`Number("349.00")` zamienia tekst na liczbę. `Intl.NumberFormat` to wbudowany
w przeglądarkę mechanizm formatowania – bez żadnej biblioteki dostajesz `349,00 zł`
ze spacją, przecinkiem i walutą, zgodnie z polskimi zasadami. Zmiana `"pl-PL"` na
`"en-US"` i `"USD"` da `$349.00`.

Warto o tym wiedzieć, bo ludzie piszą takie formatowanie ręcznie i mylą się przy
zaokrąglaniu i separatorach tysięcy.

---

## Krok 7 – komponent karty i propsy

Utwórz katalog `src/components/` i w nim **`ProductCard.jsx`**:

```jsx
import { formatPrice } from "../format";

function ProductCard({ product }) {
  return (
    <article className="card">
      <h2>{product.name}</h2>
      <p className="category">{product.category}</p>
      <p className="price">{formatPrice(product.price)}</p>
    </article>
  );
}

export default ProductCard;
```

**Propsy to dane przekazywane komponentowi od rodzica** – odpowiednik argumentów funkcji.
Rodzic napisze `<ProductCard product={...} />`, a dziecko odbierze to jako `props.product`.
Zapis `{ product }` w nawiasach klamrowych to **destrukturyzacja**: wyciąga jedno pole
z obiektu propsów, żeby nie pisać `props.` przy każdym użyciu.

Reguła bez wyjątków: **propsy są tylko do odczytu**. Komponent nigdy nie modyfikuje
tego, co dostał. Dane płyną w jedną stronę, z góry na dół, i dzięki temu zawsze wiadomo,
kto co zmienił.

Użyj karty w `App.jsx`:

```jsx
import { PRODUCTS } from "./data";
import ProductCard from "./components/ProductCard";

function App() {
  return (
    <main>
      <h1>Katalog produktów</h1>
      <ProductCard product={PRODUCTS[0]} />
    </main>
  );
}

export default App;
```

---

## Krok 8 – lista

Jedna karta to za mało. Zamiast pisać trzy razy to samo, generujemy elementy z tablicy:

```jsx
<div className="grid">
  {PRODUCTS.map((product) => (
    <ProductCard key={product.id} product={product} />
  ))}
</div>
```

`map` przekształca tablicę obiektów w tablicę elementów JSX, a React wyświetla je po
kolei. To najczęściej używany wzorzec w całym Reakcie – zobaczysz go w każdym projekcie.

### `key` nie jest opcjonalne

Brak `key` daje ostrzeżenie w konsoli, a przy dodawaniu i usuwaniu elementów – realne
błędy: zaznaczenia i wpisany tekst przeskakują do niewłaściwych wierszy.

React używa klucza, żeby wiedzieć, **który element listy jest którym** między kolejnymi
renderowaniami. Bez niego może tylko zgadywać po pozycji.

Kluczem ma być coś **trwałego i unikalnego** – u nas `product.id`. Kuszące
`key={index}` jest gorsze niż nic: przy usunięciu pierwszego elementu wszystkie pozostałe
dostaną nowe numery i React uzna, że zmieniła się ich zawartość.

---

## Krok 9 – renderowanie warunkowe

Pokażmy, że produktu nie ma na stanie. W `ProductCard.jsx`:

```jsx
{product.stock === 0 ? (
  <p className="out">Niedostępny</p>
) : (
  <p className="in">Na stanie: {product.stock} szt.</p>
)}
```

Ponieważ w JSX nie wolno użyć `if`, stosuje się **operator warunkowy** `warunek ? A : B`.
Nazywa się go też **trójargumentowym** albo **ternarnym**, bo przyjmuje trzy argumenty:
warunek oraz wartości dla prawdy i dla fałszu. To jedyny taki operator w JavaScripcie
(i w większości innych języków), dlatego samo określenie „operator trójargumentowy"
wystarcza, żeby wiedzieć, o który chodzi.

Gdy druga gałąź nie jest potrzebna, wystarczy `&&`:

```jsx
{product.stock < 5 && product.stock > 0 && <p className="warning">Ostatnie sztuki!</p>}
```

> **Pułapka.** `{product.stock && <p>...</p>}` przy `stock` równym `0` wyświetli na
> stronie samo `0`, bo zero nie jest wartością logiczną, tylko liczbą i JSX ją wypisze.
> Dlatego porównuj jawnie: `product.stock > 0 && ...`.

---

## Krok 10 – stan i interaktywność

Do tej pory strona była statyczna. Teraz wyszukiwarka.

W `App.jsx`:

```jsx
import { useState } from "react";
import { PRODUCTS } from "./data";
import ProductCard from "./components/ProductCard";

function App() {
  const [query, setQuery] = useState("");

  const visible = PRODUCTS.filter((product) =>
    product.name.toLowerCase().includes(query.toLowerCase())
  );

  return (
    <main>
      <h1>Katalog produktów</h1>

      <input
        type="text"
        placeholder="Szukaj produktu..."
        value={query}
        onChange={(event) => setQuery(event.target.value)}
      />

      <div className="grid">
        {visible.map((product) => (
          <ProductCard key={product.id} product={product} />
        ))}
      </div>
    </main>
  );
}

export default App;
```

Wpisz coś w pole – lista filtruje się przy każdym znaku.

### Jak to właściwie działa

`useState("")` zwraca parę: **aktualną wartość** i **funkcję do jej zmiany**. Zapis
`const [query, setQuery]` to destrukturyzacja tablicy; nazwy wybierasz sam, ale
konwencja `x` / `setX` jest powszechna.

Mechanizm jest taki:

```
wpisanie znaku -> onChange -> setQuery(nowa wartość) -> React renderuje App
ponownie -> filter liczy się od nowa -> lista na ekranie się zmienia
```

**Nie manipulujesz DOM-em.** Nie ma tu `document.getElementById` ani `innerHTML`.
Opisujesz, jak strona ma wyglądać **dla danego stanu**, a React sam wylicza, co
w rzeczywistym drzewie HTML trzeba zmienić. To jest cała różnica między Reactem
a jQuery, o którym wspominaliśmy przy omawianiu podstawy programowej.

**Dlaczego nie zwykła zmienna.** Gdybyś napisał `let query = ""` i zmienił jej wartość,
React by o tym nie wiedział i nic by się nie przerysowało. Zmiana stanu przez funkcję
`set...` jest sygnałem: „przelicz to jeszcze raz".

**Pole kontrolowane.** `value={query}` plus `onChange` znaczy, że jedynym źródłem prawdy
o zawartości pola jest stan Reacta, a nie DOM. Jeśli podasz `value` bez `onChange`, pole
będzie niedziałające – to najczęstsze zdziwienie początkujących.

---

## Krok 11 – drugi stan: filtr kategorii

Komponent może mieć wiele niezależnych stanów:

```jsx
const [query, setQuery] = useState("");
const [category, setCategory] = useState("all");

const categories = ["all", ...new Set(PRODUCTS.map((p) => p.category))];

const visible = PRODUCTS.filter((product) => {
  const matchesQuery = product.name.toLowerCase().includes(query.toLowerCase());
  const matchesCategory = category === "all" || product.category === category;
  return matchesQuery && matchesCategory;
});
```

I lista rozwijana w JSX:

```jsx
<select value={category} onChange={(event) => setCategory(event.target.value)}>
  {categories.map((name) => (
    <option key={name} value={name}>
      {name === "all" ? "wszystkie kategorie" : name}
    </option>
  ))}
</select>

{visible.length === 0 && <p>Nic nie pasuje do wyszukiwania.</p>}
```

`new Set(...)` usuwa duplikaty, a `...` (operator rozwinięcia) zamienia zbiór z powrotem
na tablicę i dokłada `"all"` na początek. Lista kategorii wylicza się z danych – dodasz
nową kategorię w `data.js` i pojawi się sama.

Zwróć uwagę, że **kategorie i filtrowanie liczą się przy każdym renderowaniu**, na
podstawie stanu. Nie trzymasz ich w osobnych zmiennych i nie musisz pamiętać
o aktualizowaniu. To podstawowa idea Reacta: z minimalnego stanu wyliczasz całą resztę.

---

## Krok 12 – szczegóły produktu

Bez żadnej biblioteki do nawigacji – wystarczy trzeci stan.

**`src/components/ProductDetails.jsx`**:

```jsx
import { formatPrice } from "../format";

function ProductDetails({ product, onBack }) {
  return (
    <article className="details">
      <button onClick={onBack}>&larr; Wróć do listy</button>

      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <p className="price">{formatPrice(product.price)}</p>
      <p>Kategoria: {product.category}</p>
      <p>Na stanie: {product.stock} szt.</p>
    </article>
  );
}

export default ProductDetails;
```

W `App.jsx` dochodzi stan i wcześniejszy powrót z funkcji:

```jsx
const [selected, setSelected] = useState(null);

if (selected) {
  return <ProductDetails product={selected} onBack={() => setSelected(null)} />;
}
```

A karta dostaje możliwość kliknięcia. W `ProductCard.jsx` dopisz drugi props:

```jsx
function ProductCard({ product, onSelect }) {
  return (
    <article className="card" onClick={() => onSelect(product)}>
```

i przekaż go z `App.jsx`:

```jsx
<ProductCard key={product.id} product={product} onSelect={setSelected} />
```

### Funkcja przekazana jako props

`onSelect` to **funkcja**, którą rodzic daje dziecku. Dziecko nie wie, co ona robi –
po prostu ją wywołuje, gdy ktoś kliknie. Dzięki temu `ProductCard` nadaje się do użycia
wszędzie: raz może otwierać szczegóły, innym razem dodawać do koszyka.

Tak dane wracają **do góry**: propsy płyną w dół, zdarzenia w górę.

Zwróć też uwagę na `onClick={() => onSelect(product)}`. Strzałka jest konieczna.
Gdybyś napisał `onClick={onSelect(product)}`, funkcja wykonałaby się **od razu przy
renderowaniu**, a nie po kliknięciu – i aplikacja wpadłaby w pętlę.

---

## Krok 13 – trochę stylu

`src/index.css` – celowo minimalnie, bez żadnego frameworka:

```css
body {
  font-family: system-ui, sans-serif;
  margin: 0;
  padding: 2rem;
  background: #f5f5f5;
  color: #1a1a1a;
}

.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 1rem;
  margin-top: 1.5rem;
}

.card {
  background: #fff;
  border-radius: 8px;
  padding: 1rem;
  cursor: pointer;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.card:hover {
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.15);
}

.card h2 {
  font-size: 1rem;
  margin: 0 0 0.5rem;
}

.category {
  color: #777;
  font-size: 0.85rem;
  margin: 0;
}

.price {
  font-weight: bold;
  font-size: 1.1rem;
}

.out {
  color: #b00020;
}

.in {
  color: #1b7a34;
}
```

Upewnij się, że `src/main.jsx` importuje ten plik (`import './index.css'`). Zwykły CSS
w osobnym pliku wystarczy na potrzeby tego projektu – Tailwind i biblioteki komponentów to temat
na później i nie są potrzebne do niczego, co robimy.

---

## Krok 14 – budowanie wersji produkcyjnej

```bash
npm run build
npm run preview
```

`build` tworzy katalog `dist/` – kilka plików HTML, CSS i JS, zminifikowanych
i gotowych do wrzucenia na dowolny serwer. `preview` uruchamia lokalny serwer, który
podaje właśnie te pliki, żebyś sprawdził wersję produkcyjną przed wdrożeniem.

Zajrzyj do `dist/assets/`. Cały Twój kod razem z Reactem zmieścił się w jednym pliku
`.js` o nazwie z losowym fragmentem – ten fragment zmienia się przy każdej zmianie treści
i dzięki temu przeglądarki nie podają użytkownikom starej wersji z pamięci podręcznej.

`dist/` **nie trafia do repozytorium** – jest w `.gitignore` od materiału 01, razem
z `node_modules/`. Wynik budowania zawsze da się odtworzyć z kodu źródłowego.

---

## Krok 15 – zapis

```bash
git status --short          # NIE może tu być node_modules/ ani dist/
git add .
git commit -m "feat(frontend): lista produktów w Reakcie na danych statycznych"
```

Podział na commity:

```
chore(frontend): projekt React + Vite
feat(frontend): komponent karty produktu
feat(frontend): lista produktów z tablicy
feat(frontend): wyszukiwarka i filtr kategorii
feat(frontend): widok szczegółów produktu
style(frontend): podstawowe style siatki i kart
```

---

## Sprawdź, czy rozumiesz

1. Dlaczego nazwa komponentu musi zaczynać się wielką literą?
2. Czym różni się props od stanu? Które z nich komponent może zmieniać?
3. Do czego React używa `key` i dlaczego indeks tablicy jest złym kluczem?
4. Dlaczego `let query = ""` nie zadziała zamiast `useState("")`?
5. Co się stanie, gdy napiszesz `onClick={onSelect(product)}` bez strzałki?
6. Dlaczego `{product.stock && <p>Dostępny</p>}` przy zerze wypisuje na stronie `0`?
7. Dlaczego cena przychodzi jako tekst i gdzie zamieniamy ją na liczbę?

---

## Częste błędy

| Objaw | Przyczyna |
|---|---|
| `Objects are not valid as a React child` | próbujesz wyświetlić cały obiekt zamiast jego pola |
| `Each child in a list should have a unique "key"` | brak `key` przy `map` |
| pole tekstowe nie reaguje na pisanie | `value` bez `onChange` |
| `Cannot read properties of undefined` | literówka w nazwie propsa albo pola |
| komponent się nie wyświetla | nazwa małą literą albo brak `export default` |
| `Adjacent JSX elements must be wrapped` | dwa elementy obok siebie, brakuje fragmentu `<>` |
| strona pusta, biały ekran | błąd w konsoli przeglądarki – **zawsze otwieraj F12** |
| zmiany nie są widoczne | serwer padł po błędzie składni; sprawdź terminal |

---

## Czego jeszcze nie ma

- **Danych z serwera.** Cała aplikacja działa na pliku. To zmieni się w materiale 07.
- **Nawigacji po adresach.** Szczegóły produktu nie mają własnego adresu, więc nie da się
  ich udostępnić linkiem ani cofnąć przyciskiem przeglądarki. React Router dojdzie razem
  z logowaniem.
- **Wysyłania danych.** Tylko odczyt. Formularze i metoda POST po Androidzie.
- **Testów.** Sprawdzasz klikaniem.

---

## Co dalej

**Materiał 06:** CORS i konfiguracja połączenia – kilka zmian po stronie Django, dzięki
którym przeglądarka w ogóle pozwoli Twojemu frontendowi rozmawiać z API.

**Materiał 07:** ten sam React, ale dane pobierane z `/api/products/`. Skasujesz
`data.js`, a `ProductCard` i `ProductDetails` zostaną **bez jednej zmiany**.

---

# Zadania

Pracujesz na swojej domenie z materiałów 03 i 04.

**Zadanie 1 – projekt**
Utwórz `frontend/` przez Vite w tym samym repozytorium co backend. Sprawdź, że
`git status` nie pokazuje `node_modules/`. Commit.

**Zadanie 2 – prawdziwe dane**
Uruchom swoje API, skopiuj odpowiedź z endpointu listy i wklej ją do `src/data.js`.
Minimum 8 rekordów. **Nie wymyślaj danych ręcznie** – mają pochodzić z Twojej bazy,
razem z polami dokładnie tak nazwanymi jak w API.

**Zadanie 3 – komponent karty**
Osobny plik w `src/components/`. Karta pokazuje minimum 4 pola, w tym jedno
sformatowane (cena, data, jednostka).

**Zadanie 4 – lista**
Renderowanie przez `map` z poprawnym `key`. W `docs/react.md` odpowiedz jednym akapitem,
co się stanie, gdy zamiast `id` użyjesz indeksu, i dlaczego.

**Zadanie 5 – warunki**
Dwa różne warunki wizualne: jeden z operatorem `? :`, drugi z `&&`. Na przykład status
dostępności i ostrzeżenie o kończącym się zapasie.

**Zadanie 6 – wyszukiwarka**
Filtrowanie po polu tekstowym, bez rozróżniania wielkości liter. Komunikat, gdy nic nie
pasuje.

**Zadanie 7 – filtr**
Lista rozwijana z kategoriami **wyliczonymi z danych**, nie wpisanymi ręcznie. Ma działać
jednocześnie z wyszukiwarką.

**Zadanie 8 – szczegóły**
Kliknięcie w kartę otwiera widok szczegółów z przyciskiem powrotu. Zrealizowane stanem,
bez biblioteki routingu.

**Zadanie 9 – sortowanie**
Trzeci stan: sortowanie po nazwie i po polu liczbowym, rosnąco i malejąco.
Podpowiedź: `[...tablica].sort(...)` – kopia jest istotna, bo `sort` zmienia oryginał,
a danych z propsów nie wolno modyfikować.

**Zadanie 10 – budowanie i dokumentacja**
`npm run build` ma przejść bez błędów. W `README.md` dopisz sekcję o uruchamianiu
frontendu: `npm install`, `npm run dev`, adres i wzmiankę, że dane są tymczasowo
statyczne.

---

**Zadanie dodatkowe (dla chętnych)**
Wydziel z `App.jsx` osobny komponent `Filters` przyjmujący propsy `query`, `category`,
`categories`, `onQueryChange` i `onCategoryChange`. `App` ma dalej przechowywać stan –
`Filters` tylko go wyświetla i zgłasza zmiany. W `docs/react.md` napisz, dlaczego stan
został w rodzicu, a nie przeniósł się do nowego komponentu. Ten wzorzec nazywa się
*lifting state up* i wrócimy do niego przy koszyku.