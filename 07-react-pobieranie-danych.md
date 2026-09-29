# React pobierający dane z API

**Technik programista | klasa 5 | materiał 07**

---

## Moment, do którego szliśmy

Cztery materiały temu zapowiedziałem, że kiedy podmienimy źródło danych, komponenty się
nie zmienią. Dziś to sprawdzimy.

Kasujesz `src/data.js`. W jego miejsce wchodzi `fetch` odpytujący Twoje własne API.
**`ProductCard.jsx` i `ProductDetails.jsx` zostają nietknięte** – nie otworzysz ich nawet
raz. Dostają te same obiekty, o tych samych polach, tylko z innego miejsca.

```
05:  data.js        ---> ProductCard  ---> ekran
07:  fetch z API    ---> ProductCard  ---> ten sam ekran
```

Dochodzi za to coś, czego przy pliku nie było: **czas i możliwość porażki**. Odczyt
z pliku jest natychmiastowy i zawsze się udaje. Żądanie sieciowe trwa i może się nie udać
na kilka sposobów. Większość tego materiału jest właśnie o tym.

### Zanim zaczniesz

Muszą działać **oba** serwery i musi być skonfigurowany CORS z materiału 06:

```bash
# terminal 1
cd backend && venv\Scripts\activate && python manage.py runserver

# terminal 2
cd frontend && npm run dev
```

Sprawdź też, czy `frontend/.env` zawiera `VITE_API_URL=http://localhost:8000/api`.

---

## Krok 1 – warstwa dostępu do API

Nie rozrzucamy wywołań `fetch` po komponentach. Wszystkie idą przez jeden plik – dzięki
temu adres, obsługa błędów i format odpowiedzi są w jednym miejscu.

Utwórz **`src/api.js`**:

```js
const API_URL = import.meta.env.VITE_API_URL;

export async function fetchProducts(category) {
  const url = new URL(`${API_URL}/products/`);

  if (category && category !== "all") {
    url.searchParams.set("category", category);
  }

  const response = await fetch(url);

  if (!response.ok) {
    throw new Error(`Serwer odpowiedział błędem ${response.status}`);
  }

  return response.json();
}
```

### Co tu się dzieje

**`import.meta.env.VITE_API_URL`** – adres z pliku `.env`, nie wpisany na sztywno.
Przy wdrożeniu zmienisz jedną zmienną, a nie dziesięć miejsc w kodzie.

**`new URL(...)` i `searchParams`** – zamiast sklejać adres z kawałków tekstu. Ta klasa
sama zadba o znak zapytania, znaki `&` między parametrami i zakoduje polskie znaki czy
spacje. Ręczne sklejanie działa do pierwszej spacji w wyszukiwanej frazie.

**`await fetch(url)`** – `fetch` zwraca obietnicę (*Promise*), czyli zapowiedź wyniku,
który pojawi się później. `await` czeka na spełnienie tej zapowiedzi.

**`response.json()`** – też trwa, bo treść odpowiedzi może jeszcze płynąć. Dlatego
również wymaga `await`, tu ukrytego w `return` z funkcji `async`.

### Najważniejsza linijka w całym pliku

```js
if (!response.ok) {
  throw new Error(...);
}
```

**`fetch` nie zgłasza błędu przy statusie 404 czy 500.** Dla niego „serwer odpowiedział,
że nie ma takiego zasobu" to udane żądanie – odpowiedź przecież dotarła. `catch` złapie
tylko sytuację, w której nie udało się w ogóle nawiązać połączenia.

Bez tego sprawdzenia Twój kod spróbuje odczytać JSON ze strony błędu i wywali się
w zupełnie innym miejscu, komunikatem, który nijak nie wskaże przyczyny. To jedna
z najczęstszych pułapek `fetch` i powód, dla którego wiele zespołów sięga po bibliotekę
`axios`, która robi to sprawdzenie sama.

`response.ok` jest prawdą dla statusów od 200 do 299.

---

## Krok 2 – `useEffect`, czyli kiedy w ogóle wywołać żądanie

Komponent to funkcja, która się wykonuje przy każdym renderowaniu. Gdybyś wywołał `fetch`
wprost w jej ciele, stałoby się coś takiego:

```
render -> fetch -> przyjdą dane -> setProducts -> render -> fetch -> ...
```

Nieskończona pętla. Potrzebujesz sposobu, żeby powiedzieć: „zrób to **raz**, po
wyrenderowaniu, a nie przy każdym".

Tym sposobem jest `useEffect`:

```jsx
useEffect(() => {
  // kod, który ma się wykonać
}, [zależności]);
```

**Tablica zależności decyduje o wszystkim:**

| Zapis | Kiedy się wykona |
|---|---|
| `[]` | raz, po pierwszym wyświetleniu komponentu |
| `[category]` | po pierwszym i za każdym razem, gdy `category` się zmieni |
| brak tablicy | po **każdym** renderowaniu – prawie zawsze błąd |

Nazwa „efekt" bierze się stąd, że chodzi o **efekt uboczny** – coś poza zwykłym
wyliczeniem tego, co ma być na ekranie. Pobranie danych, ustawienie stopera, zapis do
pamięci przeglądarki.

---

## Krok 3 – trzy stany zamiast jednego

Przy pliku `data.js` dane po prostu były. Teraz w każdej chwili aplikacja jest w jednym
z trzech położeń:

| Stan | Co widzi użytkownik |
|---|---|
| trwa pobieranie | informacja o ładowaniu |
| błąd | komunikat, co poszło nie tak |
| dane gotowe | lista produktów |

Do tego czwarty przypadek, o którym się zapomina: **dane przyszły, ale jest ich zero**.
To nie jest błąd – to pusta lista i osobny komunikat.

Pominięcie stanu ładowania jest najczęstszym błędem początkujących. Przy szybkim
lokalnym serwerze nie widać różnicy, więc problem ujawnia się dopiero u użytkownika
z wolnym łączem, który przez dwie sekundy patrzy na pustą stronę i wychodzi.

---

## Krok 4 – nowy `App.jsx`

```jsx
import { useEffect, useState } from "react";

import { fetchProducts } from "./api";
import ProductCard from "./components/ProductCard";
import ProductDetails from "./components/ProductDetails";

function App() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  const [query, setQuery] = useState("");
  const [selected, setSelected] = useState(null);

  useEffect(() => {
    async function load() {
      setLoading(true);
      setError(null);

      try {
        const data = await fetchProducts();
        setProducts(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }

    load();
  }, []);

  const visible = products.filter((product) =>
    product.name.toLowerCase().includes(query.toLowerCase())
  );

  if (selected) {
    return <ProductDetails product={selected} onBack={() => setSelected(null)} />;
  }

  return (
    <main>
      <h1>Katalog produktów</h1>

      <input
        type="text"
        placeholder="Szukaj produktu..."
        value={query}
        onChange={(event) => setQuery(event.target.value)}
      />

      {loading && <p>Ładowanie produktów...</p>}

      {error && (
        <p className="error">
          Nie udało się pobrać danych: {error}
        </p>
      )}

      {!loading && !error && visible.length === 0 && (
        <p>Nic nie pasuje do wyszukiwania.</p>
      )}

      {!loading && !error && (
        <div className="grid">
          {visible.map((product) => (
            <ProductCard key={product.id} product={product} onSelect={setSelected} />
          ))}
        </div>
      )}
    </main>
  );
}

export default App;
```

Uruchom. Lista wygląda tak samo jak w materiale 05 – tylko dane płyną teraz z Django.

### Dlaczego funkcja wewnątrz, a nie `useEffect(async () => ...)`

Bo funkcja `async` zawsze zwraca obietnicę, a React oczekuje od `useEffect` czegoś
zupełnie innego: funkcji czyszczącej albo niczego. Dlatego deklarujemy funkcję `load`
w środku i od razu ją wywołujemy. Wygląda to trochę dziwnie i jest to standardowy
sposób – zobaczysz go w każdym projekcie.

### `finally`

Wykonuje się **zawsze** – po sukcesie i po błędzie. Gdyby `setLoading(false)` znalazło
się tylko w `try`, po nieudanym żądaniu napis „Ładowanie" zostałby na ekranie
na zawsze, obok komunikatu o błędzie.

### `useState([])`, nie `useState(null)`

Początkowa wartość to **pusta tablica**, bo zaraz wywołujemy na niej `.filter()`.
Na `null` dostałbyś `Cannot read properties of null (reading 'filter')` jeszcze zanim
dane przyjdą. Stan początkowy zawsze ma być tego samego typu co wartość docelowa.

---

## Krok 5 – podwójne żądanie, czyli obiecany `StrictMode`

Otwórz zakładkę **Network**. Zobaczysz **dwa** identyczne żądania do `/api/products/`.

Nie zepsułeś niczego – uprzedzałem o tym w materiale 05. `StrictMode` w trybie
deweloperskim celowo montuje komponent, odmontowuje i montuje ponownie, żeby wykryć
efekty, które nie sprzątają po sobie. W wersji produkcyjnej (`npm run build`) dzieje się
to raz.

Chcesz się przekonać? Zajrzyj do `src/main.jsx` i zakomentuj na chwilę `<StrictMode>` –
żądanie będzie jedno. Potem odkomentuj, bo ten tryb działa na Twoją korzyść.

---

## Krok 6 – co, gdy backend nie działa

Zatrzymaj Django (`Ctrl+C` w pierwszym terminalu) i odśwież stronę.

Zamiast pustego ekranu dostajesz komunikat o błędzie – bo obsłużyłeś ten przypadek.
W konsoli przeglądarki zobaczysz `TypeError: Failed to fetch`. Tak wygląda **jedyna**
sytuacja, w której `fetch` naprawdę zgłasza wyjątek: nie udało się nawiązać połączenia.

Uruchom serwer z powrotem i odśwież.

Sprawdź jeszcze wolne łącze. W zakładce Network jest lista rozwijana z przepustowością –
wybierz „Slow 3G" i odśwież. Teraz naprawdę widać, po co jest stan ładowania.

Warto też dodać przycisk ponowienia próby, żeby użytkownik nie musiał odświeżać całej
strony. Wystarczy wyciągnąć `load` poza `useEffect`, do osobnej funkcji, i wywołać ją
z `onClick`. Zostawiam to jako zadanie.

---

## Krok 7 – filtrowanie po stronie serwera

Wyszukiwarka filtruje lokalnie, w przeglądarce. Ale kategorie mamy przecież obsłużone
w API – `/api/products/?category=peripherals` działa od materiału 03 i dotąd nikt z tego
nie korzystał.

Dopisz w `src/api.js`:

```js
export async function fetchInfo() {
  const response = await fetch(`${API_URL}/info/`);

  if (!response.ok) {
    throw new Error(`Serwer odpowiedział błędem ${response.status}`);
  }

  return response.json();
}
```

Lista kategorii pochodzi teraz z serwera – endpoint `/api/info/` zwraca ją w polu
`data_source.categories`. W materiale 03 wyglądał na ciekawostkę, a właśnie okazał się
przydatny.

W `App.jsx` dochodzi stan kategorii i **drugi efekt**:

```jsx
const [categories, setCategories] = useState([]);
const [category, setCategory] = useState("all");

useEffect(() => {
  fetchInfo()
    .then((info) => setCategories(info.data_source.categories))
    .catch(() => setCategories([]));
}, []);
```

A efekt pobierający produkty zyskuje zależność:

```jsx
useEffect(() => {
  async function load() {
    setLoading(true);
    setError(null);

    try {
      const data = await fetchProducts(category);
      setProducts(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }

  load();
}, [category]);
```

Lista rozwijana w JSX:

```jsx
<select value={category} onChange={(event) => setCategory(event.target.value)}>
  <option value="all">wszystkie kategorie</option>
  {categories.map((name) => (
    <option key={name} value={name}>{name}</option>
  ))}
</select>
```

Zmień kategorię i obserwuj zakładkę Network – przy każdej zmianie leci nowe żądanie
z parametrem w adresie. `[category]` w tablicy zależności znaczy dokładnie: „powtórz ten
efekt, gdy ta wartość się zmieni".

### Filtrować u siebie czy na serwerze?

| | W przeglądarce | Na serwerze |
|---|---|---|
| Szybkość reakcji | natychmiast, bez sieci | opóźnienie każdego żądania |
| Ilość przesyłanych danych | wszystko naraz, raz | tylko to, co potrzebne |
| Sensowne przy | małych zbiorach (do kilkuset) | dużych zbiorach, stronicowaniu |

Dlatego w naszej aplikacji wyszukiwarka tekstowa działa lokalnie – reaguje na każdą
literę i nie ma sensu zasypywać serwera żądaniami – a filtr kategorii po stronie serwera,
bo zmienia się rzadko i potrafi odciąć większość danych. Przy dziesięciu tysiącach
produktów oba przeniosłyby się na serwer, razem ze stronicowaniem.

---

## Krok 8 – wyścig żądań i funkcja czyszcząca

Filtrowanie na serwerze wprowadza problem, którego wcześniej nie było.

Przełączasz kategorię A, potem szybko B. Lecą dwa żądania. Nic nie gwarantuje, że wrócą
w tej samej kolejności – jeśli odpowiedź dla A dotrze **po** odpowiedzi dla B, na ekranie
wylądują produkty z A, mimo że w liście rozwijanej widnieje B. Interfejs kłamie.

Nazywa się to **wyścigiem** (*race condition*) i jest to prawdziwy błąd produkcyjny,
trudny do odtworzenia, bo pojawia się tylko przy określonych opóźnieniach.

Rozwiązanie: `useEffect` może zwrócić **funkcję czyszczącą**, którą React wywoła przed
kolejnym uruchomieniem efektu i przy usuwaniu komponentu.

```jsx
useEffect(() => {
  let ignore = false;

  async function load() {
    setLoading(true);
    setError(null);

    try {
      const data = await fetchProducts(category);
      if (!ignore) setProducts(data);
    } catch (err) {
      if (!ignore) setError(err.message);
    } finally {
      if (!ignore) setLoading(false);
    }
  }

  load();

  return () => {
    ignore = true;
  };
}, [category]);
```

Żądanie nadal poleci do końca – po prostu **jego wynik zostanie zignorowany**, bo
dotyczy nieaktualnego już wyboru. Każde uruchomienie efektu ma własną zmienną `ignore`
i własną funkcję czyszczącą; React woła ją, zanim uruchomi efekt ponownie.

To właśnie takie niesprzątające efekty wykrywa `StrictMode` podwójnym montowaniem.
Teraz widać, że nie robi tego złośliwie.

---

## Krok 9 – szczegóły prosto z serwera

Na razie widok szczegółów dostaje obiekt z listy. Działa, bo nasza lista zwraca komplet
pól. W prawdziwych API zwykle tak nie jest: lista podaje skrót, a pełne dane – opinie,
zdjęcia, parametry – dopiero endpoint szczegółów.

**Uwaga: to już nie jest podmiana źródła, tylko nowa funkcja.** Dlatego dopiero teraz
`ProductDetails.jsx` się zmieni – wcześniej ani razu.

W `src/api.js`:

```js
export async function fetchProduct(id) {
  const response = await fetch(`${API_URL}/products/${id}/`);

  if (response.status === 404) {
    throw new Error("Nie ma takiego produktu");
  }

  if (!response.ok) {
    throw new Error(`Serwer odpowiedział błędem ${response.status}`);
  }

  return response.json();
}
```

Zwróć uwagę, że **404 obsługujemy osobno**, przed ogólnym sprawdzeniem. Użytkownik
powinien zobaczyć „nie ma takiego produktu", a nie „błąd 404" – kod statusu nic mu
nie mówi. Twój endpoint zwraca ten status od materiału 03 i wreszcie ktoś z niego
korzysta.

Komponent pobierający własne dane:

```jsx
import { useEffect, useState } from "react";

import { fetchProduct } from "../api";
import { formatPrice } from "../format";

function ProductDetails({ productId, onBack }) {
  const [product, setProduct] = useState(null);
  const [error, setError] = useState(null);

  useEffect(() => {
    let ignore = false;

    fetchProduct(productId)
      .then((data) => {
        if (!ignore) setProduct(data);
      })
      .catch((err) => {
        if (!ignore) setError(err.message);
      });

    return () => {
      ignore = true;
    };
  }, [productId]);

  if (error) {
    return (
      <article className="details">
        <button onClick={onBack}>&larr; Wróć do listy</button>
        <p className="error">{error}</p>
      </article>
    );
  }

  if (!product) {
    return <p>Ładowanie...</p>;
  }

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

W `App.jsx` przekazujesz teraz identyfikator zamiast całego obiektu:

```jsx
const [selectedId, setSelectedId] = useState(null);

if (selectedId) {
  return <ProductDetails productId={selectedId} onBack={() => setSelectedId(null)} />;
}
```

i w karcie:

```jsx
<ProductCard key={product.id} product={product} onSelect={() => setSelectedId(product.id)} />
```

Dwie rzeczy warte zauważenia. Po pierwsze, **komponent potomny też może mieć własny
`useEffect`** – pobieranie danych nie musi być w komponencie nadrzędnym. Po drugie,
przekazywanie samego identyfikatora zamiast całego obiektu to przygotowanie pod adresy
typu `/produkty/7`, które dojdą razem z routingiem.

---

## Krok 10 – sprzątanie i zapis

```bash
git rm frontend/src/data.js
git status --short
git add .
git commit -m "feat(frontend): dane pobierane z API zamiast z pliku"
```

Podział na commity:

```
feat(frontend): warstwa dostępu do API w src/api.js
feat(frontend): lista produktów pobierana przez fetch
feat(frontend): obsługa stanów ładowania i błędu
feat(frontend): filtr kategorii realizowany po stronie serwera
fix(frontend): ignorowanie nieaktualnych odpowiedzi przy zmianie filtra
feat(frontend): szczegóły produktu z osobnego endpointu
chore(frontend): usunięcie danych statycznych
```

Zaktualizuj `README.md`: uruchomienie frontendu wymaga teraz **działającego backendu**.
To nowa zależność i ktoś, kto o niej nie wie, zobaczy tylko komunikat o błędzie.

---

## Sprawdź, czy rozumiesz

1. Dlaczego nie można wywołać `fetch` wprost w ciele komponentu?
2. Co oznacza pusta tablica `[]` jako drugi argument `useEffect`, a co `[category]`?
3. Dlaczego `fetch` nie zgłasza wyjątku przy statusie 500 i co z tego wynika dla kodu?
4. Dlaczego `useEffect(async () => ...)` jest błędem?
5. Po co `setLoading(false)` znajduje się w `finally`, a nie w `try`?
6. Na czym polega wyścig żądań i jak go rozwiązuje zmienna `ignore`?
7. Dlaczego wyszukiwarka filtruje lokalnie, a kategoria na serwerze?
8. Dlaczego stan początkowy to `useState([])`, a nie `useState(null)`?

---

## Częste błędy

| Objaw | Przyczyna |
|---|---|
| nieskończona pętla żądań | brak tablicy zależności w `useEffect` |
| `Cannot read properties of null (reading 'map')` | stan początkowy `null` zamiast `[]` |
| `TypeError: Failed to fetch` | backend nie działa albo zły adres w `.env` |
| błąd CORS | patrz materiał 06; sprawdź, czy serwer był restartowany |
| dwa żądania zamiast jednego | `StrictMode` w trybie deweloperskim – tak ma być |
| `Unexpected token '<' ... is not valid JSON` | serwer zwrócił HTML (stronę błędu), a kod próbuje czytać JSON |
| „Ładowanie" nie znika po błędzie | `setLoading(false)` poza `finally` |
| `import.meta.env.VITE_API_URL` to `undefined` | brak restartu Vite po zmianie `.env` |
| lista nie odświeża się po zmianie filtra | brak zmiennej w tablicy zależności |
| pokazują się dane poprzedniej kategorii | wyścig żądań – brak funkcji czyszczącej |

---

## Czego jeszcze nie ma

- **Tylko odczyt.** Nadal wyłącznie GET. Dodawanie i edycja przez panel administratora.
- **Brak stronicowania.** Jedno żądanie pobiera wszystko.
- **Brak pamięci podręcznej.** Każde przełączenie kategorii to nowe żądanie, nawet jeśli
  te dane już raz przyszły. W prawdziwych projektach używa się do tego bibliotek takich
  jak TanStack Query.
- **Brak adresów.** Szczegóły produktu wciąż nie mają własnego URL-a.
- **Brak logowania.** API jest publiczne.

---

## Co dalej

Backend obsłużył pierwszego klienta. W kolejnym materiale dostanie **drugiego** –
aplikację na Androida w Javie. Ten sam endpoint, ten sam JSON, zero zmian po stronie
Django. Znowu zaczniemy od danych wpisanych na sztywno, a potem podmienimy je na
prawdziwe żądania.

---

# Zadania

**Zadanie 1 – warstwa API**
Utwórz `src/api.js` z funkcją pobierającą listę z Twojego API. Adres wyłącznie
z `import.meta.env`. Obowiązkowe sprawdzenie `response.ok`.

**Zadanie 2 – podmiana źródła**
Usuń `data.js`. Lista ma się wyświetlać z serwera. Udowodnij w `docs/react.md`, że
komponent karty nie wymagał zmian – wklej jego kod przed i po.

**Zadanie 3 – trzy stany**
Obsłuż ładowanie, błąd i pustą listę. Każdy z osobnym komunikatem dla użytkownika.
Napisanym po polsku, zrozumiale – „Error 500" to nie jest komunikat dla człowieka.

**Zadanie 4 – dowód na stany**
Zrzuty ekranu trzech sytuacji: ładowanie przy przepustowości „Slow 3G", błąd przy
wyłączonym backendzie, pusta lista przy filtrze bez wyników. Do `docs/react.md`.

**Zadanie 5 – przycisk ponowienia**
Przy błędzie pokaż przycisk „Spróbuj ponownie", który powtarza żądanie bez odświeżania
strony. Podpowiedź: wydziel funkcję ładującą poza `useEffect`.

**Zadanie 6 – filtr na serwerze**
Kategorie pobierane z Twojego endpointu informacyjnego, filtrowanie przez parametr
w adresie. W zakładce Network pokaż, że przy zmianie filtra leci nowe żądanie.

**Zadanie 7 – wyścig**
Zaimplementuj funkcję czyszczącą ze zmienną `ignore`. W `docs/react.md` opisz własnymi
słowami scenariusz, w którym bez niej interfejs pokazałby nieprawdę.

**Zadanie 8 – szczegóły z serwera**
Widok szczegółów pobiera dane osobnym żądaniem po identyfikatorze. Obsłuż 404 osobnym,
zrozumiałym komunikatem. Sprawdź, wpisując w kodzie nieistniejące `id`.

**Zadanie 9 – dokumentacja**
W `README.md` zaznacz, że frontend wymaga działającego backendu, i wypisz kolejność
uruchamiania. Dopisz sekcję o zmiennej `VITE_API_URL`.

**Zadanie 10 – budowanie**
`npm run build` bez błędów i ostrzeżeń. Sprawdź `npm run preview` przy działającym
backendzie.

---

**Zadanie dodatkowe (dla chętnych)**
Napisz własny *hook* `useFetch(url)`, który zwraca `{ data, loading, error }`, i przepisz
na niego oba miejsca pobierające dane. Reguły: nazwa musi zaczynać się od `use`, a hook
to zwykła funkcja, która sama korzysta z `useState` i `useEffect`. W `docs/react.md`
napisz, ile linii kodu zniknęło z `App.jsx` i co byś stracił, gdyby każdy komponent
pobierał dane po swojemu.