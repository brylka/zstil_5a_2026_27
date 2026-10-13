# Dokumentacja w kodzie

**Technik programista | klasa 5 | materiał 10**

---

## Zacznij od testu

Otwórz `backend/shop/views.py` – plik, który pisałeś przy materiale 03, czyli jakieś
sześć tygodni temu. Nie czytaj materiału. Odpowiedz z pamięci:

1. Dlaczego w `json_response` jest `safe=False`?
2. Dlaczego cena idzie przez `str(product.price)`, a nie wprost?
3. Co dokładnie robi `select_related("category")` i co się stanie, jak to usuniesz?
4. Dlaczego `created_at` ma `.isoformat()`?

Teraz to samo z `frontend/src/api.js`: dlaczego sprawdzasz `response.ok`, skoro masz
`catch`? I z `android/.../ProductAdapter.java`: czemu `findViewById` jest
w konstruktorze `ViewHolder`?

Jeśli choć raz się zawahałeś – i prawie na pewno tak było – to właśnie zobaczyłeś, po co
istnieje dokumentacja.

**Dokumentacja nie jest dla nauczyciela. Jest dla Ciebie za sześć tygodni.** Ty z września
wiedziałeś wszystko. Ty z października już nie. Ty ze stycznia, trzy dni przed egzaminem,
będziesz czytał ten kod jak cudzy.

---

## Dwa materiały, dwa rodzaje dokumentacji

| | Materiał | Dla kogo | Gdzie leży |
|---|---|---|---|
| **10** | dokumentacja **w kodzie** | dla programisty, który ten kod czyta i zmienia | w plikach źródłowych |
| **11** | dokumentacja **projektu** | dla osoby, która system uruchamia, używa lub ocenia | `README.md`, `docs/`, diagramy |

To są dwie różne umiejętności. Pisanie docstringa jest nawykiem programistycznym –
robi się to w trakcie kodowania, przy okazji. Pisanie dokumentacji projektu jest
pisaniem technicznym – wymaga spojrzenia na całość z zewnątrz i osobnego czasu.

Dziś pierwsza. Wracasz do wszystkich trzech części projektu i opisujesz je **od środka**.

---

## Część 1 – zasada, która decyduje o wszystkim

### Kod mówi JAK. Komentarz mówi DLACZEGO.

To jedno zdanie zastępuje połowę poradników o dokumentacji.

Kod już opisuje, co się dzieje – wystarczy go przeczytać. Czego kod **nie potrafi**
powiedzieć, to dlaczego został napisany właśnie tak, jakie rozwiązanie odrzucono i co się
zepsuje, gdy ktoś to „uprości".

Porównaj:

```python
# źle – komentarz powtarza kod
# zwiększ licznik o 1
counter += 1

# źle – opisuje oczywistość
# pętla po produktach
for product in products:
    ...

# dobrze – wyjaśnia powód
# select_related pobiera kategorie jednym JOIN-em; bez tego każdy produkt
# generuje osobne zapytanie (problem N+1) i przy 500 pozycjach mamy 501 zapytań
products = Product.objects.select_related("category")

# dobrze – ostrzega przed pozorną poprawą
# cena jako tekst, nie float – Decimal traci dokładność po konwersji,
# a przy sumowaniu faktur różnica staje się realna
"price": str(product.price),
```

Pierwsze dwa komentarze to **szum**. Zabierają miejsce, trzeba je czytać, nic nie wnoszą.
Dwa kolejne odpowiadają na pytanie, które naprawdę przyjdzie komuś do głowy – i oszczędzą
godzinę debugowania albo uratują przed cofnięciem Twojej dobrej decyzji.

### Najpierw nazwy, potem komentarze

Zanim napiszesz komentarz, sprawdź, czy nie da się go uniknąć lepszą nazwą.

```python
# komentarz zamiast nazwy
d = 0.23          # stawka VAT
t = p * (1 + d)   # cena brutto

# nazwa zamiast komentarza
VAT_RATE = 0.23
gross_price = net_price * (1 + VAT_RATE)
```

Druga wersja nie potrzebuje komentarza i **nie może się z nim rozjechać**. To ważne,
bo komentarze mają brzydką właściwość: zostają, gdy kod się zmienia.

### Komentarz, który kłamie, jest gorszy niż jego brak

```python
# zwraca produkty tańsze niż 100 zł
def get_cheap_products():
    return Product.objects.filter(price__lt=200)
```

Ktoś zmienił progi i nie poprawił komentarza. Następna osoba przeczyta komentarz,
uwierzy mu i podejmie na tej podstawie decyzję. Brak komentarza kazałby jej przeczytać
kod – i poznać prawdę.

**Zasada: komentarz zmienia się w tym samym commicie co kod, który opisuje.** Tak samo
jak README, o którym mówiliśmy przy materiale 01.

---

## Część 2 – Python i Django: docstringi

### Czym docstring różni się od komentarza

Komentarz po `#` jest dla człowieka czytającego plik. **Docstring jest obiektem
w programie.**

```python
def json_response(data, status=200):
    """Zwraca odpowiedź JSON obsługującą listy i polskie znaki."""
    ...
```

Spróbuj w konsoli:

```bash
python manage.py shell
```

```python
from shop.views import json_response

print(json_response.__doc__)
help(json_response)
```

Docstring **żyje w czasie działania programu**. Dlatego z docstringów da się
automatycznie wygenerować dokumentację, podpowiedź w edytorze i – co zobaczysz przy
Django REST Framework – opis endpointu w przeglądarce. Z komentarza po `#` nie da się nic.

To jest praktyczna różnica, nie kwestia gustu: **docstring to dokumentacja, komentarz to
notatka**.

### Konwencja (PEP 257)

```python
def product_to_dict(product):
    """Zamienia obiekt modelu Product na słownik gotowy do serializacji.

    Cena wychodzi jako tekst, bo Decimal nie jest typem JSON-a, a konwersja
    na float psułaby dokładność. Kategoria jest spłaszczona do nazwy, żeby
    zachować format odpowiedzi z materiału 03.

    Args:
        product: obiekt Product, najlepiej pobrany z select_related("category"),
            bo funkcja sięga do product.category.name.

    Returns:
        Słownik z kluczami: id, name, description, price, stock, category,
        created_at.
    """
```

Reguły, które warto zapamiętać:

- Trzy podwójne cudzysłowy, **zawsze**, nawet dla jednej linii.
- Pierwsza linia to **jedno zdanie** podsumowania, zaczynające się wielką literą
  i kończące kropką.
- Po pierwszej linii pusta linia, potem szczegóły.
- Opis mówi, **co funkcja robi**, nie co robi programista: „Zamienia obiekt na
  słownik", nie „Zamień obiekt na słownik".
- Docstring jest **pierwszą rzeczą** w funkcji, klasie lub module – przed jakimkolwiek
  kodem.

### Trzy poziomy

```python
"""Widoki API sklepu.

Wszystkie widoki zwracają JSON i obsługują wyłącznie metodę GET.
Zapis odbywa się na razie przez panel administratora.
"""

import sys
...


class Product(models.Model):
    """Produkt w katalogu sklepu.

    Cena jest typu Decimal, nie Float – patrz dokumentacja pola price.
    Usunięcie kategorii z produktami jest blokowane (on_delete=PROTECT).
    """
```

Docstring modułu stoi **na samej górze pliku**, przed importami. To miejsce, w którym
odpowiadasz na pytanie „co tu w ogóle jest?" – pierwszą rzecz, jaką chce wiedzieć ktoś
otwierający nieznany plik.

### Adnotacje typów

Python nie wymaga podawania typów, ale pozwala je zapisać – i to też jest dokumentacja,
tylko sprawdzalna przez narzędzia.

```python
def product_to_dict(product: Product) -> dict:
    ...


def env_list(name: str, default: str = "") -> list[str]:
    """Zamienia zmienną środowiskową 'a,b,c' na listę ['a', 'b', 'c']."""
    ...
```

Python **nie sprawdza** tych adnotacji w czasie działania – przekażesz tam cokolwiek
i nic się nie stanie. Robią one trzy inne rzeczy: dokumentują, włączają podpowiedzi
w edytorze i dają się zweryfikować narzędziem w rodzaju `mypy`.

`list[str]` to nowszy zapis, dostępny od Pythona 3.9. W starszych materiałach zobaczysz
`List[str]` z importem z modułu `typing` – to przeżytek, nie kopiuj go.

### Django: `help_text`

W Django jest trzeci rodzaj dokumentacji, o którym łatwo nie wiedzieć. W materiale 04
nadaliśmy polom etykiety (`verbose_name`). Teraz dochodzi opis:

```python
class Product(models.Model):
    price = models.DecimalField(
        "cena",
        max_digits=8,
        decimal_places=2,
        help_text="Cena brutto w złotych. Maksymalnie 999999.99.",
    )
    stock = models.PositiveIntegerField(
        "stan magazynowy",
        default=0,
        help_text="Liczba sztuk dostępnych. Zero oznacza brak w magazynie.",
    )
```

Wejdź teraz do panelu administratora i spójrz na formularz produktu – opisy pojawiły się
pod polami, jako podpowiedzi dla osoby wprowadzającej dane.

To jest dokumentacja, która **dociera do użytkownika**, nie tylko do programisty. A przy
Django REST Framework trafi dodatkowo do automatycznie generowanego opisu API. Jeden wpis,
trzy zastosowania – dlatego warto.

---

## Część 3 – JavaScript i React: JSDoc

### Składnia

JSDoc to komentarz zaczynający się od **dwóch gwiazdek**:

```js
/**
 * Pobiera listę produktów z API.
 *
 * @param {string} [category] - nazwa kategorii; pominięcie lub "all" zwraca wszystko
 * @returns {Promise<Product[]>} lista produktów
 * @throws {Error} gdy serwer odpowie statusem poza zakresem 2xx
 */
export async function fetchProducts(category) {
  ...
}
```

Zwykły `/* */` to komentarz blokowy. **`/** */` z dwiema gwiazdkami to JSDoc** – i tę
różnicę rozpoznają edytory. Nazwa pochodzi wprost od Javadoc, który poznasz w następnej
części; JSDoc jest jego młodszym naśladowcą, stąd identyczne znaczniki `@param`
i `@returns`.

| Znacznik | Znaczenie |
|---|---|
| `@param {typ} nazwa - opis` | parametr; nazwa w `[nawiasach]` oznacza opcjonalny |
| `@returns {typ} opis` | wartość zwracana |
| `@throws {typ} opis` | kiedy funkcja rzuca wyjątkiem |
| `@typedef` | definicja własnego typu |
| `@example` | przykład użycia |

### Praktyczny zysk: podpowiedzi

To nie jest sztuka dla sztuki. Zdefiniuj raz kształt produktu – najlepiej w `src/api.js`,
na górze pliku:

```js
/**
 * Produkt zwracany przez API.
 *
 * @typedef {Object} Product
 * @property {number} id
 * @property {string} name
 * @property {string} description
 * @property {string} price - tekst, nie liczba; serwer zwraca Decimal jako string
 * @property {number} stock
 * @property {string} category - nazwa kategorii, nie obiekt
 * @property {string} created_at - data w formacie ISO 8601
 */
```

Teraz w komponencie:

```jsx
/**
 * Karta pojedynczego produktu na liście.
 *
 * @param {Object} props
 * @param {import('../api').Product} props.product - produkt do wyświetlenia
 * @param {(product: import('../api').Product) => void} props.onSelect
 *        wywoływane po kliknięciu karty, z klikniętym produktem
 */
function ProductCard({ product, onSelect }) {
  ...
}
```

Napisz w tym komponencie `product.` i zobacz, co się stanie: edytor podpowie nazwy pól.
Pomyl się w nazwie – podkreśli ją. Bez TypeScriptu, bez żadnej dodatkowej biblioteki,
z samego komentarza.

To jest moment, w którym dokumentacja przestaje być obowiązkiem i zaczyna się opłacać
tego samego dnia.

### PropTypes – nie używaj

Szukając w internecie sposobu na opisanie propsów, trafisz na `PropTypes`. **W React 19
zostały usunięte** – React je po cichu ignoruje, więc kod wygląda, jakby działał,
a nie sprawdza niczego. Z komponentów funkcyjnych usunięto też `defaultProps`; wartości
domyślne podaje się teraz zwyczajnie w destrukturyzacji:

```jsx
function Heading({ text = "Katalog produktów" }) {
  return <h1>{text}</h1>;
}
```

Oficjalną drogą do sprawdzania typów w Reakcie jest dziś TypeScript. My go w tym
projekcie nie używamy, więc dokumentujemy JSDoc-iem – który daje podpowiedzi w edytorze,
choć nie zatrzyma błędu przy budowaniu.

To dobry przykład na to, że **wiek znalezionego poradnika ma znaczenie**. Sprawdzaj daty.

### Co dokumentować w swoim froncie

- **`api.js`** – każdą funkcję: co pobiera, co zwraca, kiedy rzuca wyjątkiem.
  Plus `@typedef` dla kształtu danych.
- **komponenty** – propsy i jedno zdanie o tym, co komponent wyświetla.
- **`format.js`** – że przyjmuje tekst, bo cena przychodzi jako tekst.
- **efekty** – krótki komentarz przy `useEffect` z nieoczywistą tablicą zależności,
  a koniecznie przy funkcji czyszczącej ze zmienną `ignore`. Ten kod wygląda na
  nadmiarowy i ktoś go kiedyś usunie, jeśli nie napiszesz, przed czym chroni.

---

## Część 4 – Java i Android: Javadoc

### Składnia

Identyczna jak JSDoc, bo to JSDoc jest kopią Javadoc:

```java
/**
 * Adapter listy produktów dla RecyclerView.
 *
 * <p>Adapter trzyma odniesienie do przekazanej listy, a nie jej kopię –
 * filtrowanie musi więc modyfikować tę samą listę (clear + addAll),
 * a nie podstawiać nową.
 *
 * @see ProductViewHolder
 */
public class ProductAdapter extends RecyclerView.Adapter<ProductAdapter.ProductViewHolder> {
```

```java
/**
 * Wypełnia istniejący wiersz danymi produktu.
 *
 * <p>Wykonuje się przy każdym przewinięciu listy, dla każdego wiersza
 * wchodzącego na ekran – dlatego nie wolno tu wywoływać findViewById
 * ani tworzyć nowych obiektów.
 *
 * @param holder wiersz do wypełnienia, z gotowymi odniesieniami do widoków
 * @param position pozycja w liście danych
 */
@Override
public void onBindViewHolder(@NonNull ProductViewHolder holder, int position) {
```

| Znacznik | Znaczenie |
|---|---|
| `@param nazwa opis` | parametr – bez nawiasów klamrowych, typ jest w sygnaturze |
| `@return opis` | wartość zwracana |
| `@throws Typ opis` | zgłaszany wyjątek |
| `@see Klasa` | odsyłacz do powiązanej klasy |
| `{@link Klasa#metoda}` | odsyłacz wewnątrz zdania |
| `@deprecated` | element przestarzały, nie używać |

Dwie różnice wobec JSDoc: typów **nie podaje się w nawiasach klamrowych**, bo Java i tak
je zna z sygnatury, a akapity rozdziela się znacznikiem `<p>`, bo Javadoc generuje HTML
i przyjmuje znaczniki HTML wprost w treści.

### Podgląd i generowanie

W Android Studio ustaw kursor na nazwie klasy lub metody i wciśnij **Ctrl+Q**
(na macOS F1) – pokaże się Javadoc. Zrób to na `enqueue` z Retrofitu albo na
`notifyDataSetChanged` i zobacz, jak wygląda dokumentacja biblioteki, z której
korzystasz.

Pełną dokumentację HTML generuje się przez **Tools → Generate JavaDoc**. Powstanie
katalog ze stronami wyglądającymi jak oficjalna dokumentacja Javy – bo ona jest
generowana dokładnie tym samym narzędziem.

Katalog z wygenerowaną dokumentacją **nie trafia do repozytorium**, tak jak `dist/`
i `build/`. To wynik przetworzenia kodu, odtwarzalny jednym kliknięciem.

### Co dokumentować w swojej mobilce

- **`Product`** – że nazwy pól muszą zgadzać się z kluczami JSON-a, bo Gson dopasowuje
  je po nazwie i przy niezgodności zostawia pole puste **bez żadnego błędu**. To jest
  dokładnie ten rodzaj wiedzy, która musi być zapisana przy kodzie, bo z samego kodu
  nie wynika.
- **`ApiClient`** – dlaczego singleton i dlaczego `baseUrl` musi kończyć się ukośnikiem.
- **`ApiService`** – przy każdej metodzie: jaki endpoint, co zwraca.
- **`ProductAdapter`** – trzy metody i różnica między nimi, rola `ViewHolder`.
- **`MainActivity`** – dlaczego dwie listy (`allProducts` i `visible`) i dlaczego po
  pobraniu wołamy `filter`, a nie `addAll`.

---

## Część 5 – trzy języki, jedna idea

| | Python | JavaScript | Java |
|---|---|---|---|
| Nazwa | docstring | JSDoc | Javadoc |
| Składnia | `"""..."""` | `/** ... */` | `/** ... */` |
| Parametr | `Args:` | `@param {typ} n` | `@param n` |
| Zwraca | `Returns:` | `@returns {typ}` | `@return` |
| Dostępny w czasie działania | **tak**, `__doc__` | nie | nie |
| Podgląd w edytorze | tak | tak | tak (Ctrl+Q) |
| Generator HTML | `pdoc`, Sphinx | `jsdoc` | `javadoc` |
| Standard | PEP 257 | – | część JDK |

Zauważ, co z tego wynika: **nauczyłeś się jednej rzeczy trzy razy**. Składnia się różni,
intencja jest identyczna – opisać, czego z kodu nie widać, w miejscu, w którym ten kod
leży.

Tak samo było z listą (`map`, `RecyclerView`), z formatowaniem ceny (`Intl.NumberFormat`,
`NumberFormat`) i z obsługą błędu żądania (`response.ok`, `isSuccessful()`). Każda
następna technologia kosztuje Cię coraz mniej, bo wzorce się powtarzają. To jest cała
tajemnica ludzi, którzy „znają dziesięć języków".

---

## Część 6 – czego nie dokumentować

Dokumentacji można mieć za dużo. Każda linia komentarza to linia do czytania
i do utrzymywania.

**Nie dokumentuj oczywistości:**

```java
// źle
/**
 * Zwraca nazwę.
 * @return nazwa
 */
public String getName() {
    return name;
}
```

Ten docstring nie wnosi niczego, czego nie ma w nazwie metody. Gettery i settery zostaw
w spokoju – chyba że dzieje się w nich coś nieoczywistego.

**Nie zostawiaj zakomentowanego kodu:**

```python
# źle
# products = Product.objects.all()
# products = Product.objects.filter(stock__gt=0)
products = Product.objects.select_related("category")
```

Usuń. **Git pamięta każdą wersję** – po to go mamy od materiału 01. Zakomentowany kod
tylko zaśmieca i każe się zastanawiać, czy jest ważny.

**Nie pisz `TODO` bez kontekstu.** Jeśli już, to tak, żeby dało się z tego coś zrobić:

```python
# TODO: dodać paginację – przy ponad 1000 produktów ten endpoint zwraca
#       wszystko naraz i zabija przeglądarkę (zgłoszenie #23)
```

Konwencja, którą spotkasz wszędzie i którą rozpoznają edytory:

| Znacznik | Znaczenie |
|---|---|
| `TODO` | brakująca funkcja, do zrobienia |
| `FIXME` | działa źle, wymaga naprawy |
| `HACK` | obejście; wiem, że brzydkie, działa z takiego powodu |
| `NOTE` | ostrzeżenie dla czytającego |

W Android Studio i PyCharmie jest zakładka **TODO**, która zbiera je wszystkie z całego
projektu. Przed oddaniem pracy warto tam zajrzeć.

---

## Część 7 – praca z agentem, czyli uwaga

Agent LLM napisze Ci docstringi do całego projektu w minutę. To kuszące i w połowie
przypadków się sprawdzi. Ale musisz wiedzieć, na czym polega druga połowa.

**Agent widzi kod, nie widzi powodów.** Potrafi bezbłędnie opisać, że funkcja przyjmuje
obiekt i zwraca słownik – bo to widać. Nie potrafi wiedzieć, że cena jest tekstem,
bo tak zdecydowałeś w materiale 04 po rozmowie o dokładności `Decimal`. Napisze coś
prawdopodobnie brzmiącego, na przykład „cena w formacie tekstowym dla wygody" – i to
będzie wyglądało sensownie, a będzie nieprawdą.

To jest najgorszy możliwy rodzaj dokumentacji: **wiarygodna i fałszywa**.

Dlatego zasada na ten materiał: agent może pisać docstringi, ale **każdy sprawdzasz
i poprawiasz**, a te wyjaśniające „dlaczego" piszesz sam. Jeśli docstring zawiera wiedzę,
której agent nie mógł mieć, to znaczy, że ją wymyślił.

Dobry sposób pracy: najpierw sam dopisz komentarze „dlaczego" w najtrudniejszych
miejscach, potem poproś agenta o uzupełnienie opisów parametrów i typów. Odwrotna
kolejność daje ładny, bezużyteczny plik.

---

## Część 8 – przejście przez własny projekt

Teraz robota. Po kolei, trzy części.

### Backend

| Plik | Co opisać |
|---|---|
| `shop/models.py` | docstring modułu; docstring każdego modelu; `help_text` przy polach nieoczywistych; komentarz przy `on_delete=PROTECT` |
| `shop/views.py` | docstring modułu (tylko GET, zapis przez panel); docstring każdego widoku; komentarz „dlaczego" przy `safe=False`, `str(price)`, `select_related` |
| `shop/urls.py` | docstring modułu z listą endpointów |
| `core/settings.py` | komentarze przy `env_list`, przy `os.environ[...]` zamiast `.get()`, przy kolejności `CorsMiddleware` |

### Frontend

| Plik | Co opisać |
|---|---|
| `src/api.js` | `@typedef` kształtu danych; JSDoc każdej funkcji z `@throws`; komentarz przy `response.ok` |
| `src/components/*.jsx` | JSDoc propsów każdego komponentu |
| `src/format.js` | że przyjmuje tekst i dlaczego |
| `src/App.jsx` | komentarz przy funkcji czyszczącej ze zmienną `ignore` – koniecznie |

### Mobilka

| Plik | Co opisać |
|---|---|
| `Product.java` | Javadoc klasy z ostrzeżeniem o zgodności nazw z JSON-em |
| `ApiService.java` | Javadoc każdej metody: endpoint i wynik |
| `ApiClient.java` | dlaczego singleton, dlaczego ukośnik na końcu adresu |
| `ProductAdapter.java` | Javadoc klasy i trzech metod; rola `ViewHolder` |
| `MainActivity.java` | dlaczego dwie listy, dlaczego `filter` po pobraniu |

### Test końcowy

Odłóż projekt na jeden dzień. Następnego otwórz **tylko pliki źródłowe**, bez żadnego
materiału z zajęć, i spróbuj odpowiedzieć na te same cztery pytania z początku tego
dokumentu.

Jeśli odpowiedzi są w kodzie – zrobiłeś to dobrze. Jeśli musisz zaglądać do materiałów –
czegoś brakuje i wiesz już dokładnie czego.

---

## Sprawdź, czy rozumiesz

1. Co mówi kod, a co ma mówić komentarz?
2. Dlaczego komentarz, który się rozjechał z kodem, jest gorszy od braku komentarza?
3. Czym docstring różni się od komentarza po `#` – i co z tej różnicy wynika praktycznie?
4. Dlaczego adnotacje typów w Pythonie są dokumentacją, a nie zabezpieczeniem?
5. Co daje `@typedef` w JSDoc, czego nie daje zwykły opis słowny?
6. Dlaczego nie używamy `PropTypes` w React 19?
7. Po co w Javadoc stosuje się `<p>`, a w JSDoc nie?
8. Dlaczego nie dokumentujemy getterów?
9. Dlaczego zakomentowany kod należy usunąć, a nie zostawić „na wszelki wypadek"?
10. Czego agent LLM nie może wiedzieć o Twoim kodzie i co z tego wynika dla docstringów?

---

## Częste błędy

| Objaw | Przyczyna |
|---|---|
| docstring jest ignorowany przez `help()` | nie jest pierwszym elementem w funkcji lub użyto `#` |
| edytor nie podpowiada pól po JSDoc | komentarz ma jedną gwiazdkę: `/*` zamiast `/**` |
| `@typedef` nie działa w innym pliku | brak `import('./api').Product` w odwołaniu |
| `PropTypes` nie zgłasza błędów | React 19 je ignoruje – to nie awaria, to usunięta funkcja |
| Javadoc nie pokazuje akapitów | brak `<p>` między akapitami |
| `Generate JavaDoc` kończy się błędami | brak `@param` dla któregoś parametru albo literówka w nazwie |
| komentarzy jest więcej niż kodu | opisujesz „jak" zamiast „dlaczego" |
| docstringi brzmią ładnie, ale mylą | napisał je agent i nikt ich nie sprawdził |

---

## Czego jeszcze nie ma

- **Dokumentacji projektu.** README, opis API, diagramy, instrukcja użytkownika –
  materiał 11.
- **Automatycznej dokumentacji API.** Django REST Framework potrafi wygenerować
  przeglądalny opis endpointów z docstringów i serializerów. Dojdzie razem z DRF –
  i wtedy docstringi, które dziś napiszesz, zaczną pracować same.
- **Testów.** Test jest szczególną formą dokumentacji: opisuje, jak kod **ma** się
  zachowywać, i w odróżnieniu od komentarza **nie może skłamać**, bo przestaje
  przechodzić. To osobny materiał.

---

## Co dalej

**Materiał 11:** dokumentacja projektu. README jako wizytówka, pełny opis API, diagram
bazy danych, diagram architektury, instrukcja uruchomienia i instrukcja użytkownika.
Czyli to wszystko, co pozwala obcej osobie wziąć Twoje repozytorium i z niego skorzystać.

---

# Zadania

Pracujesz na swoim projekcie – wszystkich trzech częściach.

**Zadanie 1 – test wyjściowy**
Przed jakąkolwiek pracą: otwórz swoje trzy najstarsze pliki i zapisz w `docs/dokumentacja.md`
**trzy fragmenty własnego kodu, których działania nie potrafisz wyjaśnić z pamięci**.
Dosłownie je wklej. Do nich wrócisz w zadaniu 10.

**Zadanie 2 – backend: docstringi**
Docstring modułu w każdym pliku aplikacji. Docstring każdej funkcji i klasy, według
PEP 257: jedno zdanie podsumowania, potem szczegóły, sekcje `Args` i `Returns` tam, gdzie
są parametry.

**Zadanie 3 – backend: „dlaczego"**
Minimum **pięć** komentarzy wyjaśniających decyzję, nie działanie. Każdy musi odpowiadać
na pytanie, które naprawdę ktoś zadałby. W `docs/dokumentacja.md` wypisz te pięć miejsc
i w jednym zdaniu uzasadnij, czemu wybrałeś właśnie je.

**Zadanie 4 – backend: adnotacje typów i `help_text`**
Adnotacje typów we wszystkich własnych funkcjach pomocniczych. `help_text` przy minimum
trzech polach modelu. Zrzut ekranu formularza w panelu administratora z widocznymi
podpowiedziami – do `docs/dokumentacja.md`.

**Zadanie 5 – frontend: `@typedef` i podpowiedzi**
Zdefiniuj `@typedef` dla kształtu danych zwracanych przez Twoje API. Użyj go w JSDoc
propsów komponentów. **Udowodnij, że działa**: zrzut ekranu z podpowiedzią edytora po
wpisaniu kropki za nazwą obiektu.

**Zadanie 6 – frontend: funkcje i komponenty**
JSDoc każdej funkcji w warstwie dostępu do API, z `@throws`. JSDoc propsów każdego
komponentu. Obowiązkowo komentarz wyjaśniający funkcję czyszczącą ze zmienną `ignore` –
napisany tak, żeby następna osoba jej nie usunęła.

**Zadanie 7 – mobilka: Javadoc**
Javadoc klas i metod publicznych w Twoich klasach. Obowiązkowo: ostrzeżenie o zgodności
nazw pól z kluczami JSON-a oraz opis różnicy między `onCreateViewHolder`
i `onBindViewHolder`.

**Zadanie 8 – mobilka: wygenerowana dokumentacja**
Uruchom **Tools → Generate JavaDoc**, otwórz `index.html` i zrób zrzut ekranu strony
swojej klasy modelu. Upewnij się, że katalog wyniku **nie wszedł do repozytorium** –
dopisz go do `.gitignore`, jeśli trzeba.

**Zadanie 9 – porządki**
Przejdź przez cały projekt i usuń: zakomentowany kod, komentarze powtarzające kod,
`console.log` i `print` z debugowania. Każde `TODO` bez kontekstu albo uzupełnij, albo
usuń. W `docs/dokumentacja.md` podaj, ile linii zniknęło (`git diff --stat` pomoże).

**Zadanie 10 – test skuteczności**
Dzień po skończeniu pracy wróć do trzech fragmentów z zadania 1. Odpowiedz na pytanie
o każdy z nich, patrząc **wyłącznie w kod**. Zapisz odpowiedzi i uczciwie oceń: czy
dokumentacja wystarczyła? Jeśli nie – popraw ją i zapisz, co dodałeś.

To zadanie jest sprawdzianem całego materiału. Dokumentacja, która nie przechodzi tego
testu, jest ozdobą.

**Zadanie 11 – historia**
Minimum 5 commitów z typem `docs(...)`, osobno dla każdej części projektu.

---

**Zadanie dodatkowe (dla chętnych)**
Wygeneruj dokumentację backendu narzędziem `pdoc`:

```bash
pip install pdoc
pdoc --output-dir docs/api-python shop
```

Otwórz wynik w przeglądarce i porównaj z tym, co wygenerował Javadoc. W
`docs/dokumentacja.md` odpowiedz: co widać w wygenerowanej dokumentacji, czego nie widać
w kodzie, i **odwrotnie** – jakiej wiedzy o Twoim projekcie żaden generator nie wyciągnie
z plików źródłowych. Odpowiedź na to drugie pytanie to w zasadzie spis treści
materiału 11.
