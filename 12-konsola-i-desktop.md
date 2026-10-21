# Aplikacja konsolowa i desktopowa

**Technik programista | klasa 5 | materiał 12**

---

## Dlaczego przerywamy projekt

Przez jedenaście materiałów budowaliśmy jedną rzecz: backend i dwóch klientów. Teraz
na dwa tygodnie wychodzimy z tego projektu, żeby zrobić coś, czego w nim nie ma wcale.

Powód jest prosty i konkretny.

**Pierwsze zadanie na części praktycznej INF.04 to aplikacja konsolowa z programowaniem
obiektowym.** Nie strona. Nie aplikacja mobilna. Klasa, konstruktor, gettery i settery,
pole statyczne, metoda modyfikująca. Treść zadania brzmi zwykle tak:

> *Korzystając z mechanizmów programowania obiektowego, zaprojektuj część logiki systemu
> wirtualnej wypożyczalni filmów. Zaimplementuj aplikację zawierającą klasę o nazwie
> `Film` z metodami `get`, `set` oraz `inkrementuj`.*

Jeśli przez cały rok robisz tylko projekt webowy, w styczniu otworzysz arkusz i zobaczysz
zadanie, którego nigdy nie ćwiczyłeś. Modele Django są klasami – ale Ty ich nie pisałeś
jako klas. Django napisało je za Ciebie.

Druga rzecz: podstawa programowa wymaga też **aplikacji desktopowych** – okien, menu,
okien dialogowych, kontrolek i obsługi zdarzeń myszy i klawiatury. Tego w projekcie
nie ma i nie będzie.

Oba tematy robimy **raz, porządnie i do końca**. Nie wrócimy do nich.

### Co zbudujemy

Dwie aplikacje, które **dzielą ten sam model**:

```
         model.py          <- klasy, logika, zero interfejsu
        /        \
  console.py    desktop.py
  (menu tekstowe)  (okno tkinter)
```

I to jest trzeci raz, kiedy spotykasz tę samą ideę. W materiale 07 jeden backend obsłużył
React. W 09 ten sam backend obsłużył Androida. Teraz jeden plik z modelem obsłuży konsolę
i okno. **Logika oddzielona od interfejsu** – za każdym razem w innej skali, zawsze ten
sam zysk.

### Czym będziemy pisać

| | Wybór | Dlaczego |
|---|---|---|
| Język | Python 3.14 | znasz go z Django; jest na liście języków egzaminacyjnych |
| Interfejs graficzny | tkinter | w bibliotece standardowej – nic nie instalujesz |

Na końcu materiału jest porównanie z C++ i przegląd innych bibliotek graficznych –
żebyś wiedział, co istnieje, ale uczymy się **jednego** rozwiązania do końca.

---

# Część 1 – aplikacja konsolowa

## Krok 1 – katalog

Nowy katalog **poza** projektem fullstackowym, ale w tym samym repozytorium:

```
sklep-fullstack/
├── backend/
├── frontend/
├── android/
└── desktop/          <-- tutaj
    ├── model.py
    ├── console.py
    └── desktop.py
```

```bash
mkdir desktop
cd desktop
```

Żadnego środowiska wirtualnego, żadnych zależności. tkinter jest częścią Pythona.

---

## Krok 2 – klasa, czyli to, o co pyta egzamin

**`desktop/model.py`**:

```python
"""Model danych katalogu produktów.

Ten moduł nie wie nic o interfejsie użytkownika. Korzystają z niego
zarówno console.py, jak i desktop.py.
"""


class Product:
    """Produkt w katalogu.

    Attributes:
        counter: pole statyczne; liczba utworzonych produktów. Służy
            jednocześnie do nadawania kolejnych identyfikatorów.
    """

    counter = 0

    def __init__(self, name, price, stock=0, category="inne"):
        """Tworzy produkt i nadaje mu kolejny identyfikator."""
        Product.counter += 1
        self._id = Product.counter
        self._name = name
        self._price = price
        self._stock = stock
        self._category = category

    # --- dostęp do pól ---

    @property
    def id(self):
        return self._id

    @property
    def name(self):
        return self._name

    @name.setter
    def name(self, value):
        if not value.strip():
            raise ValueError("Nazwa nie może być pusta.")
        self._name = value.strip()

    @property
    def price(self):
        return self._price

    @price.setter
    def price(self, value):
        if value < 0:
            raise ValueError("Cena nie może być ujemna.")
        self._price = value

    @property
    def stock(self):
        return self._stock

    @property
    def category(self):
        return self._category

    # --- metody ---

    def increase_stock(self, amount=1):
        """Zwiększa stan magazynowy o podaną liczbę sztuk."""
        if amount < 1:
            raise ValueError("Liczba sztuk musi być dodatnia.")
        self._stock += amount
        return self._stock

    def decrease_stock(self, amount=1):
        """Zmniejsza stan magazynowy, nie pozwalając zejść poniżej zera."""
        if amount < 1:
            raise ValueError("Liczba sztuk musi być dodatnia.")
        if amount > self._stock:
            raise ValueError(f"Na stanie jest tylko {self._stock} szt.")
        self._stock -= amount
        return self._stock

    def is_available(self):
        """Zwraca True, jeśli produkt jest dostępny."""
        return self._stock > 0

    def __str__(self):
        return f"[{self._id}] {self._name} – {self._price:.2f} zł ({self._stock} szt.)"
```

### Co tu jest z egzaminu, punkt po punkcie

**Enkapsulacja.** Pola mają podkreślenie na początku: `_name`, `_price`. W Pythonie to
**umowa, nie blokada** – nikt Ci nie zabroni napisać `product._name = ""`. Podkreślenie
znaczy „to jest wewnętrzne, nie ruszaj z zewnątrz". W C++, Javie i C# istnieje do tego
słowo `private`, które naprawdę blokuje.

**Pole statyczne.** `counter = 0` stoi **w ciele klasy, poza metodami**. Należy do klasy,
nie do obiektu – wszystkie produkty dzielą jedną wartość. Dlatego zwiększa się je przez
`Product.counter += 1`, z nazwą klasy, a nie `self.counter += 1` (to ostatnie utworzyłoby
osobne pole w obiekcie i licznik przestałby działać).

Sprawdź w konsoli:

```python
from model import Product

a = Product("Klawiatura", 349.00)
b = Product("Mysz", 129.99)

print(a.id, b.id)          # 1 2
print(Product.counter)     # 2
```

**Gettery i settery.** Dekorator `@property` sprawia, że metoda wygląda jak pole:

```python
print(product.name)        # wywołuje getter
product.name = "Nowa"      # wywołuje setter – i sprawdza, czy nie jest pusta
```

To jest pythonowy sposób i w prawdziwym kodzie używa się właśnie jego. Zysk jest
praktyczny: **setter może odrzucić bzdurę**. `product.price = -5` rzuci wyjątkiem, a nie
zepsuje danych po cichu.

Zwróć uwagę, że `id`, `stock` i `category` mają tylko getter. Celowo: identyfikatora
nie wolno zmieniać, a stan magazynowy zmienia się **metodami**, które pilnują reguł.
To nie przeoczenie – to projekt klasy.

> **Uwaga egzaminacyjna.** Jeśli zadanie mówi dosłownie „zaimplementuj metody `get`
> i `set`", napisz je dosłownie:
>
> ```python
> def get_name(self):
>     return self._name
>
> def set_name(self, value):
>     self._name = value
> ```
>
> Oceniający sprawdza według listy wymagań. Metoda o nazwie `get_name` jest na liście,
> `@property` może nie być. **Rób to, co napisane w treści zadania** – to ta sama zasada
> co z nazwami plików w materiale 11.
>
> Dlatego zadania są tak formułowane: w C++, Javie i C# jawne gettery są normą, a nie
> wyborem stylistycznym.

**Metoda „inkrementuj".** `increase_stock` to dokładnie to, o co pytają zadania. Zwróć
uwagę, że ma wartość domyślną (`amount=1`), sprawdza argument i zwraca nowy stan.

---

## Krok 3 – konstruktory: jeden czy kilka?

Zadania egzaminacyjne często mówią „klasa z **konstruktorami**", w liczbie mnogiej.
W C++, Javie i C# pisze się wtedy kilka konstruktorów o różnych zestawach parametrów –
to **przeciążanie**.

**Python przeciążania nie ma.** Jedna klasa, jedna metoda `__init__`. Ten sam efekt
osiąga się dwoma sposobami.

### Wartości domyślne

```python
def __init__(self, name, price, stock=0, category="inne"):
```

Jeden konstruktor obsługuje cztery sposoby wywołania:

```python
Product("Klawiatura", 349.00)
Product("Mysz", 129.99, 40)
Product("Monitor", 1199.00, 5, "monitory")
Product(name="Słuchawki", price=289.00, category="audio")
```

### Konstruktor alternatywny

Gdy obiekt ma powstawać z innych danych, dodaje się metodę klasy:

```python
    @classmethod
    def from_dict(cls, data):
        """Tworzy produkt ze słownika – używane przy czytaniu z pliku."""
        product = cls(
            data["name"],
            data["price"],
            data.get("stock", 0),
            data.get("category", "inne"),
        )
        product._id = data["id"]
        Product.counter = max(Product.counter, data["id"])
        return product
```

`@classmethod` dostaje jako pierwszy argument **klasę** (`cls`), nie obiekt. Dzięki temu
może wywołać `cls(...)` i zwrócić gotowy produkt.

To właśnie odpowiednik drugiego konstruktora. Zapamiętaj tę różnicę między językami –
na egzaminie wybierasz język i musisz wiedzieć, jak w nim zrealizować to, czego żąda
treść zadania.

---

## Krok 4 – druga klasa: katalog

Jedna klasa to za mało. Produkty trzeba gdzieś trzymać i zarządzać nimi – to zadanie
osobnej klasy.

Dopisz w `model.py`:

```python
class Catalog:
    """Zbiór produktów z operacjami wyszukiwania i modyfikacji."""

    def __init__(self):
        self._products = []

    def add(self, product):
        """Dodaje produkt do katalogu."""
        self._products.append(product)
        return product

    def remove(self, product_id):
        """Usuwa produkt o podanym identyfikatorze.

        Returns:
            True, jeśli usunięto; False, jeśli nie znaleziono.
        """
        product = self.find(product_id)
        if product is None:
            return False
        self._products.remove(product)
        return True

    def find(self, product_id):
        """Zwraca produkt o podanym identyfikatorze albo None."""
        for product in self._products:
            if product.id == product_id:
                return product
        return None

    def search(self, phrase):
        """Zwraca produkty, których nazwa zawiera podaną frazę."""
        needle = phrase.lower()
        return [p for p in self._products if needle in p.name.lower()]

    def by_category(self, category):
        """Zwraca produkty z podanej kategorii."""
        return [p for p in self._products if p.category == category]

    def categories(self):
        """Zwraca posortowaną listę kategorii występujących w katalogu."""
        return sorted({p.category for p in self._products})

    def all(self):
        """Zwraca wszystkie produkty."""
        return list(self._products)

    def total_value(self):
        """Zwraca łączną wartość magazynu."""
        return sum(p.price * p.stock for p in self._products)

    def __len__(self):
        return len(self._products)
```

### Dlaczego dwie klasy, a nie jedna

`Product` wie wszystko o **jednym produkcie**. `Catalog` wie, jak zarządzać **zbiorem**.
To jest podział odpowiedzialności – każda klasa ma jedno zadanie.

Zauważ, że dokładnie tak działa Django: `Product` to model, a `Product.objects` to
menedżer z metodami `filter`, `get`, `all`. Napisałeś właśnie ręcznie to, co Django daje
za darmo – i teraz wiesz, co tam jest pod spodem.

`__len__` to **metoda specjalna**. Dzięki niej działa `len(catalog)` – Python wywołuje ją
za Ciebie. `__str__` z klasy `Product` jest z tej samej rodziny; `print(product)` szuka
właśnie jej.

---

## Krok 5 – zapis do pliku

Dane mają przetrwać zamknięcie programu. Bez bazy – zwykły plik JSON.

W `model.py` dopisz do klasy `Product`:

```python
    def to_dict(self):
        """Zamienia produkt na słownik gotowy do zapisu w JSON."""
        return {
            "id": self._id,
            "name": self._name,
            "price": self._price,
            "stock": self._stock,
            "category": self._category,
        }
```

Rozpoznajesz? To ta sama funkcja co `product_to_dict` z materiału 04 – tylko tam
serializowaliśmy do odpowiedzi HTTP, a tu do pliku. **Ten sam problem, inne ujście.**

I funkcje na końcu modułu:

```python
import json
from pathlib import Path

DATA_FILE = Path(__file__).with_name("products.json")


def save_catalog(catalog, path=DATA_FILE):
    """Zapisuje katalog do pliku JSON."""
    data = [product.to_dict() for product in catalog.all()]
    with open(path, "w", encoding="utf-8") as file:
        json.dump(data, file, ensure_ascii=False, indent=2)


def load_catalog(path=DATA_FILE):
    """Czyta katalog z pliku JSON. Zwraca pusty katalog, gdy pliku nie ma."""
    catalog = Catalog()

    if not Path(path).exists():
        return catalog

    with open(path, encoding="utf-8") as file:
        data = json.load(file)

    for item in data:
        catalog.add(Product.from_dict(item))

    return catalog
```

Trzy rzeczy warte uwagi:

**`encoding="utf-8"` przy otwieraniu pliku.** Bez tego Windows użyje swojego domyślnego
kodowania i polskie znaki się rozsypią. To ta sama pułapka co z przekierowaniem
`dumpdata` w materiale 04.

**`ensure_ascii=False`** – żeby w pliku było `"Słuchawki"`, a nie `"Słuchawki"`.
Dokładnie ten sam argument, który dodawaliśmy w materiale 03.

**`Path(__file__).with_name(...)`** – plik powstanie obok skryptu, a nie w katalogu,
z którego uruchomiono program. Bez tego dane „gubią się" przy uruchomieniu z innego
miejsca.

---

## Krok 6 – menu konsolowe

**`desktop/console.py`**:

```python
"""Konsolowy interfejs katalogu produktów."""

from model import Product, load_catalog, save_catalog

catalog = load_catalog()


def read_int(prompt, minimum=None):
    """Czyta liczbę całkowitą od użytkownika, aż poda poprawną."""
    while True:
        value = input(prompt).strip()
        try:
            number = int(value)
        except ValueError:
            print("  To nie jest liczba całkowita.")
            continue

        if minimum is not None and number < minimum:
            print(f"  Liczba musi być nie mniejsza niż {minimum}.")
            continue

        return number


def read_float(prompt, minimum=None):
    """Czyta liczbę rzeczywistą od użytkownika, aż poda poprawną."""
    while True:
        value = input(prompt).strip().replace(",", ".")
        try:
            number = float(value)
        except ValueError:
            print("  To nie jest liczba.")
            continue

        if minimum is not None and number < minimum:
            print(f"  Liczba musi być nie mniejsza niż {minimum}.")
            continue

        return number


def show_all():
    if len(catalog) == 0:
        print("  Katalog jest pusty.")
        return

    for product in catalog.all():
        mark = " " if product.is_available() else "!"
        print(f" {mark} {product}")


def add_product():
    name = input("  Nazwa: ").strip()
    if not name:
        print("  Nazwa nie może być pusta.")
        return

    price = read_float("  Cena: ", minimum=0)
    stock = read_int("  Stan magazynowy: ", minimum=0)
    category = input("  Kategoria: ").strip() or "inne"

    product = catalog.add(Product(name, price, stock, category))
    save_catalog(catalog)
    print(f"  Dodano: {product}")


def delete_product():
    product_id = read_int("  Identyfikator do usunięcia: ", minimum=1)

    if catalog.remove(product_id):
        save_catalog(catalog)
        print("  Usunięto.")
    else:
        print("  Nie ma produktu o takim identyfikatorze.")


def restock():
    product_id = read_int("  Identyfikator produktu: ", minimum=1)
    product = catalog.find(product_id)

    if product is None:
        print("  Nie ma produktu o takim identyfikatorze.")
        return

    amount = read_int("  Ile sztuk dodać: ", minimum=1)

    try:
        new_stock = product.increase_stock(amount)
    except ValueError as error:
        print(f"  Błąd: {error}")
        return

    save_catalog(catalog)
    print(f"  Nowy stan: {new_stock} szt.")


def search():
    phrase = input("  Szukana fraza: ").strip()
    results = catalog.search(phrase)

    if not results:
        print("  Nic nie znaleziono.")
        return

    for product in results:
        print(f"    {product}")


def statistics():
    print(f"  Liczba produktów:   {len(catalog)}")
    print(f"  Utworzono łącznie:  {Product.counter}")
    print(f"  Wartość magazynu:   {catalog.total_value():.2f} zł")
    print(f"  Kategorie:          {', '.join(catalog.categories()) or 'brak'}")

    unavailable = [p for p in catalog.all() if not p.is_available()]
    print(f"  Niedostępne:        {len(unavailable)}")


MENU = """
=== KATALOG PRODUKTÓW ===
 1. Pokaż wszystkie
 2. Dodaj produkt
 3. Usuń produkt
 4. Przyjmij dostawę
 5. Szukaj
 6. Statystyki
 0. Wyjście
"""

ACTIONS = {
    "1": show_all,
    "2": add_product,
    "3": delete_product,
    "4": restock,
    "5": search,
    "6": statistics,
}


def main():
    while True:
        print(MENU)
        choice = input("Wybór: ").strip()

        if choice == "0":
            print("Do widzenia.")
            break

        action = ACTIONS.get(choice)
        if action is None:
            print("  Nie ma takiej opcji.")
            continue

        action()


if __name__ == "__main__":
    main()
```

Uruchom:

```bash
python console.py
```

### Trzy rzeczy warte zapamiętania

**Funkcje czytające dane wejściowe.** `read_int` i `read_float` pytają w pętli, **aż
użytkownik poda coś poprawnego**. Program nigdy się nie wywali na `int("abc")`. To jest
walidacja danych wejściowych – temat, który wróci w materiale 14 w wersji serwerowej,
ale zasada jest identyczna: **nie wierz temu, co wpisuje użytkownik**.

`replace(",", ".")` w `read_float` to drobiazg z życia: Polacy piszą cenę z przecinkiem,
a Python oczekuje kropki.

**Słownik zamiast drabinki `if`.** `ACTIONS` odwzorowuje wybór na funkcję. Alternatywa –
siedem `elif` – działa tak samo, ale rośnie. Dodanie opcji to tu jedna linia w słowniku
i jedna w tekście menu.

W Pythonie funkcja jest zwykłą wartością: można ją trzymać w słowniku i wywołać później.
Spotkałeś to już w Reakcie, gdzie przekazywałeś `onSelect` jako props.

**`if __name__ == "__main__":`** – kod pod tym warunkiem wykona się tylko wtedy, gdy plik
jest **uruchamiany**, a nie **importowany**. Dzięki temu `desktop.py` może zaimportować
coś z `console.py`, nie wyświetlając menu. Bez tego warunku import uruchamiałby program.

---

# Część 2 – aplikacja desktopowa

## Krok 7 – dlaczego tkinter

| Biblioteka | Zaleta | Wada |
|---|---|---|
| **tkinter** | w bibliotece standardowej, nic nie instalujesz | wygląd sprzed lat |
| CustomTkinter | nowoczesny wygląd, nadbudowa na tkinter | dodatkowa instalacja |
| PySide6 / PyQt6 | potężne, profesjonalne, to jest **Qt** z podstawy programowej | duże, licencja PyQt bywa kłopotliwa |
| Kivy | ekrany dotykowe, działa na telefonach | nietypowe, mało podobne do reszty |
| wxPython | wygląd systemowy | mniejsza społeczność |

Wybieramy **tkinter** z jednego powodu, który przy pracy w szkolnej pracowni jest
rozstrzygający: **jest już zainstalowany**. Zero `pip install`, zero problemów
z uprawnieniami, działa na każdym komputerze z Pythonem.

Warto wiedzieć, co wymienia podstawa programowa dla aplikacji desktopowych: **WPF i Qt**.
WPF to świat C# i opisu interfejsu w XAML. Qt to C++ i Python przez PySide6. Umiejętności
przenoszą się – okno, kontrolki, zdarzenia i dialogi są wszędzie te same, zmienia się
składnia.

### Nasza lista kontrolna

Podstawa wymaga, żebyś umiał zaprogramować: **okna, menu, okna dialogowe, kontrolki
oraz obsługę zdarzeń myszy i klawiatury**.

Nasza aplikacja zrobi wszystkie pięć. Przy każdym kroku zaznaczę który.

---

## Krok 8 – okno i kontrolki

**`desktop/desktop.py`**:

```python
"""Okienkowy interfejs katalogu produktów (tkinter)."""

import tkinter as tk
from tkinter import ttk, messagebox, simpledialog

from model import Product, load_catalog, save_catalog

catalog = load_catalog()

root = tk.Tk()                                  # OKNO
root.title("Katalog produktów")
root.geometry("760x480")
root.minsize(600, 400)
```

`tk.Tk()` tworzy okno główne. Jest jedno w całej aplikacji.

**Formularz dodawania – kontrolki:**

```python
form = ttk.Frame(root, padding=10)
form.pack(fill="x")

name_var = tk.StringVar()
price_var = tk.StringVar()
stock_var = tk.StringVar(value="0")
category_var = tk.StringVar(value="inne")

ttk.Label(form, text="Nazwa:").grid(row=0, column=0, sticky="w")
name_entry = ttk.Entry(form, textvariable=name_var, width=30)
name_entry.grid(row=0, column=1, padx=5)

ttk.Label(form, text="Cena:").grid(row=0, column=2, sticky="w")
ttk.Entry(form, textvariable=price_var, width=10).grid(row=0, column=3, padx=5)

ttk.Label(form, text="Stan:").grid(row=0, column=4, sticky="w")
ttk.Spinbox(form, from_=0, to=9999, textvariable=stock_var, width=6).grid(
    row=0, column=5, padx=5
)

ttk.Label(form, text="Kategoria:").grid(row=0, column=6, sticky="w")
category_box = ttk.Combobox(form, textvariable=category_var, width=14)
category_box.grid(row=0, column=7, padx=5)

ttk.Button(form, text="Dodaj", command=lambda: add_product()).grid(
    row=0, column=8, padx=5
)
```

**`ttk` czy `tk`?** `tkinter` ma dwa zestawy kontrolek. Starsze (`tk.Button`) wyglądają
archaicznie. Nowsze (`ttk.Button`) dostosowują się do wyglądu systemu. **Używaj `ttk`,
gdy istnieje odpowiednik** – wyjątkiem jest okno główne, menu i `tk.StringVar`.

**`StringVar` to pomost.** Zmienna powiązana z kontrolką: `name_var.get()` odczytuje
zawartość pola, `name_var.set("x")` ją zmienia. Nie dotykasz kontrolki, tylko zmiennej.

To jest ten sam mechanizm co pole kontrolowane w Reakcie z materiału 05: jedno źródło
prawdy o zawartości pola, poza samym widokiem.

**`grid` czy `pack`?** `pack` układa elementy jeden pod drugim albo obok siebie – dobre
dla dużych obszarów. `grid` ustawia je w siatce wiersz-kolumna – dobre dla formularzy.
**Nie wolno ich mieszać w jednym pojemniku**; dlatego formularz jest w osobnym `Frame`.

---

## Krok 9 – lista produktów

```python
columns = ("id", "name", "price", "stock", "category")

tree = ttk.Treeview(root, columns=columns, show="headings", selectmode="browse")

tree.heading("id", text="ID")
tree.heading("name", text="Nazwa")
tree.heading("price", text="Cena")
tree.heading("stock", text="Stan")
tree.heading("category", text="Kategoria")

tree.column("id", width=50, anchor="center")
tree.column("name", width=280)
tree.column("price", width=90, anchor="e")
tree.column("stock", width=70, anchor="center")
tree.column("category", width=140)

scrollbar = ttk.Scrollbar(root, orient="vertical", command=tree.yview)
tree.configure(yscrollcommand=scrollbar.set)

tree.pack(side="left", fill="both", expand=True, padx=(10, 0), pady=10)
scrollbar.pack(side="left", fill="y", pady=10)

status = tk.StringVar()
ttk.Label(root, textvariable=status, anchor="w", padding=5).pack(
    side="bottom", fill="x"
)
```

**`Treeview` to tabela.** Mimo nazwy sugerującej drzewo, z `show="headings"` zachowuje
się jak lista z kolumnami. To odpowiednik `RecyclerView` z Androida i `map` z Reacta –
trzeci sposób na to samo.

Odświeżanie listy:

```python
def refresh():
    """Przerysowuje tabelę i pasek stanu na podstawie katalogu."""
    tree.delete(*tree.get_children())

    for product in catalog.all():
        tree.insert(
            "",
            "end",
            iid=str(product.id),
            values=(
                product.id,
                product.name,
                f"{product.price:.2f} zł",
                product.stock,
                product.category,
            ),
            tags=("unavailable",) if not product.is_available() else (),
        )

    tree.tag_configure("unavailable", foreground="#b00020")

    category_box["values"] = catalog.categories()
    status.set(
        f"Produktów: {len(catalog)}   |   "
        f"Wartość magazynu: {catalog.total_value():.2f} zł"
    )
```

Dwie sztuczki warte uwagi.

**`iid=str(product.id)`** – każdemu wierszowi nadajemy identyfikator równy identyfikatorowi
produktu. Dzięki temu po zaznaczeniu wiersza od razu wiemy, o który produkt chodzi,
bez szukania po nazwie.

**`tags` i `tag_configure`** – produkty niedostępne są czerwone. To odpowiednik
renderowania warunkowego z materiału 05 i `if (product.getStock() == 0)` z materiału 08.
**Czwarty raz ten sam wzorzec.**

---

## Krok 10 – operacje i okna dialogowe

```python
def selected_product():
    """Zwraca zaznaczony produkt albo None."""
    selection = tree.selection()
    if not selection:
        return None
    return catalog.find(int(selection[0]))


def add_product():
    name = name_var.get().strip()

    if not name:
        messagebox.showwarning("Brak nazwy", "Podaj nazwę produktu.")
        name_entry.focus_set()
        return

    try:
        price = float(price_var.get().replace(",", "."))
        stock = int(stock_var.get())
    except ValueError:
        messagebox.showerror("Błędne dane", "Cena i stan muszą być liczbami.")
        return

    try:
        product = Product(name, price, stock, category_var.get().strip() or "inne")
    except ValueError as error:
        messagebox.showerror("Błędne dane", str(error))
        return

    catalog.add(product)
    save_catalog(catalog)
    refresh()

    name_var.set("")
    price_var.set("")
    stock_var.set("0")
    name_entry.focus_set()


def delete_selected(event=None):
    product = selected_product()

    if product is None:
        messagebox.showinfo("Nic nie zaznaczono", "Wybierz produkt z listy.")
        return

    if not messagebox.askyesno("Potwierdzenie", f"Usunąć „{product.name}"?"):
        return

    catalog.remove(product.id)
    save_catalog(catalog)
    refresh()


def restock_selected(event=None):
    product = selected_product()

    if product is None:
        messagebox.showinfo("Nic nie zaznaczono", "Wybierz produkt z listy.")
        return

    amount = simpledialog.askinteger(
        "Przyjęcie dostawy",
        f"Ile sztuk dodać do „{product.name}"?",
        minvalue=1,
        maxvalue=1000,
        parent=root,
    )

    if amount is None:
        return

    product.increase_stock(amount)
    save_catalog(catalog)
    refresh()
```

**To są okna dialogowe** – trzeci punkt z listy kontrolnej.

| Wywołanie | Zastosowanie |
|---|---|
| `messagebox.showinfo` | informacja |
| `messagebox.showwarning` | ostrzeżenie |
| `messagebox.showerror` | błąd |
| `messagebox.askyesno` | pytanie, zwraca `True` / `False` |
| `simpledialog.askinteger` | pytanie o liczbę, zwraca `None` po anulowaniu |
| `simpledialog.askstring` | pytanie o tekst |
| `filedialog.askopenfilename` | wybór pliku |

**Sprawdzaj `None` po dialogu.** Użytkownik mógł kliknąć „Anuluj". Pominięcie tego
sprawdzenia to najczęstszy błąd przy pracy z dialogami.

Zauważ, że **walidacja jest w dwóch miejscach**: interfejs sprawdza, czy da się zamienić
tekst na liczbę, a setter w klasie `Product` sprawdza, czy cena nie jest ujemna. To nie
jest dublowanie – interfejs chroni przed bzdurą w formacie, model chroni przed bzdurą
w sensie. Model musi się obronić, bo korzysta z niego też konsola.

---

## Krok 11 – menu

```python
menubar = tk.Menu(root)

file_menu = tk.Menu(menubar, tearoff=0)
file_menu.add_command(label="Odśwież", command=refresh, accelerator="F5")
file_menu.add_separator()
file_menu.add_command(label="Zakończ", command=root.quit, accelerator="Ctrl+Q")
menubar.add_cascade(label="Plik", menu=file_menu)

product_menu = tk.Menu(menubar, tearoff=0)
product_menu.add_command(label="Przyjmij dostawę...", command=restock_selected)
product_menu.add_command(label="Usuń zaznaczony", command=delete_selected)
menubar.add_cascade(label="Produkt", menu=product_menu)

help_menu = tk.Menu(menubar, tearoff=0)
help_menu.add_command(
    label="O programie",
    command=lambda: messagebox.showinfo(
        "O programie",
        "Katalog produktów\nTechnik programista, klasa 5\nPython + tkinter",
    ),
)
menubar.add_cascade(label="Pomoc", menu=help_menu)

root.config(menu=menubar)
```

**To jest menu** – drugi punkt z listy.

`tearoff=0` wyłącza możliwość „oderwania" menu od okna – dziwactwo z lat
dziewięćdziesiątych, które domyślnie jest włączone i wygląda na błąd.

`accelerator` tylko **wyświetla** skrót w menu. Samo go nie włącza – to trzeba zrobić
osobno, w następnym kroku. O tym zapomina każdy przy pierwszym podejściu.

Menu kontekstowe pod prawym przyciskiem myszy:

```python
context_menu = tk.Menu(root, tearoff=0)
context_menu.add_command(label="Przyjmij dostawę...", command=restock_selected)
context_menu.add_command(label="Usuń", command=delete_selected)


def show_context_menu(event):
    row = tree.identify_row(event.y)
    if row:
        tree.selection_set(row)
        context_menu.tk_popup(event.x_root, event.y_root)
```

---

## Krok 12 – zdarzenia myszy i klawiatury

```python
# --- MYSZ ---
tree.bind("<Double-1>", restock_selected)      # podwójne kliknięcie
tree.bind("<Button-3>", show_context_menu)     # prawy przycisk

# --- KLAWIATURA ---
root.bind("<F5>", lambda event: refresh())
root.bind("<Control-q>", lambda event: root.quit())
tree.bind("<Delete>", delete_selected)
name_entry.bind("<Return>", lambda event: add_product())
```

**To są zdarzenia myszy i klawiatury** – czwarty i piąty punkt z listy. Lista kontrolna
z podstawy programowej odhaczona w całości.

| Zapis | Zdarzenie |
|---|---|
| `<Button-1>` | lewy przycisk myszy |
| `<Button-3>` | prawy przycisk |
| `<Double-1>` | podwójne kliknięcie lewym |
| `<Return>` | Enter |
| `<Delete>` | Delete |
| `<Escape>` | Esc |
| `<Control-s>` | Ctrl+S |
| `<KeyPress>` | dowolny klawisz |
| `<Motion>` | ruch myszy |

**Funkcja obsługująca zdarzenie dostaje argument** z informacjami o nim – pozycją myszy,
kodem klawisza. Dlatego w `delete_selected` jest `event=None`: ta sama funkcja działa
wywołana z menu (bez argumentu) i ze zdarzenia (z argumentem).

Gdzie `lambda event: ...` – bo funkcja nie przyjmuje argumentu, a `bind` i tak go
przekaże.

Na koniec pliku:

```python
refresh()
name_entry.focus_set()

root.mainloop()
```

**`mainloop` to pętla zdarzeń.** Program zatrzymuje się na tej linii i czeka: na
kliknięcie, na klawisz, na zamknięcie okna. Każde zdarzenie wywołuje odpowiednią funkcję,
po czym pętla czeka dalej.

Kod **po** `mainloop()` wykona się dopiero po zamknięciu okna. To zupełnie inny model
działania niż w aplikacji konsolowej, która sama prowadzi użytkownika przez kolejne
kroki. Tutaj **użytkownik decyduje, co się stanie** – program tylko reaguje.

Ten sam model miała aplikacja na Androida i strona w Reakcie. Jeśli czujesz, że jest
w tym coś znajomego, to dobrze.

---

## Krok 13 – uruchom obie i zobacz puentę

```bash
python console.py
```

Dodaj dwa produkty, wyjdź.

```bash
python desktop.py
```

**Są tam.**

Dwie zupełnie różne aplikacje – jedna tekstowa, jedna okienkowa – pracują na tych samych
danych i tej samej logice. `model.py` nie wie o istnieniu żadnej z nich. Reguła „stan
magazynowy nie może zejść poniżej zera" jest zapisana **raz** i obowiązuje w obu.

Dopisz teraz metodę do klasy `Product` i zauważ, że obie aplikacje mogą jej od razu użyć.
Zmień regułę w setterze i obie ją respektują.

To jest ta sama lekcja, którą dostałeś w materiale 09, gdy backend obsłużył drugiego
klienta bez żadnej zmiany. Tam granica była po HTTP, tu przez import. **Idea jest
identyczna: logika osobno, interfejs osobno.**

Gdybyś upchnął logikę w `console.py`, aplikacja okienkowa wymagałaby przepisania
wszystkiego od zera. Dokładnie tak jak aplikacja mobilna, gdyby Django generowało HTML.

---

## Krok 14 – to samo w C++

Dla porównania – ta sama klasa w C++. Nie piszemy w tym materiale aplikacji w C++;
chodzi o to, żebyś zobaczył różnicę, bo na egzaminie możesz wybrać ten język.

```cpp
#include <iostream>
#include <string>

class Product {
private:
    int id;
    std::string name;
    double price;
    int stock;

public:
    static int counter;                        // deklaracja pola statycznego

    // konstruktor z wartościami domyślnymi
    Product(std::string name, double price, int stock = 0)
        : name(name), price(price), stock(stock) {
        id = ++counter;
    }

    // konstruktor kopiujący – w Pythonie nie ma odpowiednika
    Product(const Product& other)
        : name(other.name), price(other.price), stock(other.stock) {
        id = ++counter;
    }

    int getId() const { return id; }
    std::string getName() const { return name; }
    double getPrice() const { return price; }
    int getStock() const { return stock; }

    void setName(std::string value) {
        if (value.empty()) {
            throw std::invalid_argument("Nazwa nie moze byc pusta.");
        }
        name = value;
    }

    int increaseStock(int amount = 1) {
        stock += amount;
        return stock;
    }

    bool isAvailable() const { return stock > 0; }
};

int Product::counter = 0;                      // definicja poza klasą – OBOWIĄZKOWA
```

### Co jest inaczej

| | Python | C++ |
|---|---|---|
| Ukrywanie pól | podkreślenie, umowa | `private:`, naprawdę blokuje |
| Typy | niewymagane | obowiązkowe |
| Gettery | `@property` | jawne metody `getX()` |
| Pole statyczne | `counter = 0` w klasie | deklaracja w klasie **i** definicja poza nią |
| Kilka konstruktorów | nie da się, są wartości domyślne | przeciążanie |
| Koniec instrukcji | nowa linia | średnik |

Trzy rzeczy, na których ludzie się przewracają przy przejściu na C++:

**`int Product::counter = 0;` poza klasą jest obowiązkowe.** W klasie jest tylko
deklaracja („takie pole istnieje"), definicja musi być osobno. Bez tej linii program
się nie zlinkuje, a komunikat błędu nie będzie pomocny.

**`const` na końcu metody** znaczy „ta metoda nie zmienia obiektu". Kompilator tego
pilnuje. Python nie ma odpowiednika.

**Lista inicjalizacyjna** – `: name(name), price(price)` po dwukropku. Można to napisać
wewnątrz nawiasów klamrowych jak w Pythonie, ale ten zapis jest w C++ normą.

Jeżeli na egzaminie wybierzesz C#, zobaczysz rozwiązanie pośrednie: `private` jak w C++,
ale gettery i settery zapisywane jako właściwości, bliżej pythonowego `@property`.

---

## Sprawdź, czy rozumiesz

1. Czym pole statyczne różni się od pola obiektu i dlaczego zwiększamy je przez nazwę
   klasy?
2. Co daje `@property` ponad zwykłe pole publiczne?
3. Dlaczego `id` ma tylko getter, a `stock` zmienia się metodami?
4. Jak w Pythonie zrealizować „kilka konstruktorów", których język nie obsługuje?
5. Dlaczego `Product` i `Catalog` to dwie klasy, a nie jedna?
6. Czym `if __name__ == "__main__":` chroni importowany moduł?
7. Dlaczego walidacja jest i w interfejsie, i w setterze – czy to nie dublowanie?
8. Co robi `mainloop()` i dlaczego kod po nim się nie wykonuje?
9. Dlaczego po `simpledialog.askinteger` trzeba sprawdzić `None`?
10. Co trzeba by zmienić w `model.py`, żeby dodać trzeci interfejs – na przykład stronę?

---

## Częste błędy

| Objaw | Przyczyna |
|---|---|
| licznik `counter` nie rośnie | `self.counter += 1` zamiast `Product.counter += 1` |
| `AttributeError: can't set attribute` | brak settera przy `@property` |
| polskie znaki w pliku JSON jako `ł` | brak `ensure_ascii=False` |
| polskie znaki rozsypane po zapisie | brak `encoding="utf-8"` przy `open` |
| plik JSON powstaje w dziwnym miejscu | ścieżka względna zamiast `Path(__file__).with_name(...)` |
| `ValueError: invalid literal for int()` | brak walidacji wejścia; użyj `read_int` |
| okno pokazuje się i od razu znika | brak `root.mainloop()` |
| kontrolki nie są widoczne | brak `pack()` albo `grid()` |
| `cannot use geometry manager grid` | mieszanie `grid` i `pack` w jednym pojemniku |
| skrót z `accelerator` nie działa | `accelerator` tylko wyświetla; trzeba dodać `bind` |
| `TypeError: takes 0 positional arguments but 1 was given` | funkcja zdarzenia bez parametru `event` |
| menu ma u góry dziwną kreskę | brak `tearoff=0` |
| C++: `undefined reference to Product::counter` | brak definicji pola statycznego poza klasą |

---

## Zamknięcie tematu

Tu kończymy z konsolą i oknami – nie wracamy do nich. Co z tego zostaje:

**Na egzamin.** Umiesz napisać klasę z enkapsulacją, polem statycznym, konstruktorem,
getterami, setterami i metodą modyfikującą – czyli całe zadanie pierwsze. Umiesz czytać
dane od użytkownika, nie wywalając programu. Wiesz, że treść zadania decyduje
o nazewnictwie.

**Do projektu.** Wracasz w materiale 13 do Django REST Framework. Z jedną nową wiedzą:
`Product.objects.filter(...)` nie jest magią. To metoda klasy menedżera, bardzo podobnej
do Twojego `Catalog`. Napisałeś ją ręcznie, więc wiesz, co tam się dzieje.

**Na zawsze.** Logika osobno, interfejs osobno. Trzy razy ta sama lekcja w trzech skalach:
jeden backend i dwa klienty po HTTP, jeden model i dwa interfejsy przez import.

---

# Zadania

Pracujesz na swojej domenie z poprzednich materiałów. W katalogu `desktop/`.

**Zadanie 1 – klasa główna**
Klasa odpowiadająca Twojej domenie. Obowiązkowo: pola prywatne (z podkreśleniem), pole
statyczne liczące utworzone obiekty, konstruktor z co najmniej jedną wartością domyślną,
minimum trzy właściwości z getterem, co najmniej dwie z setterem **sprawdzającym
poprawność**, metoda zwiększająca wartość liczbową, metoda `__str__`.

**Zadanie 2 – wersja egzaminacyjna getterów**
Dopisz do klasy **dodatkowo** jawne metody `get_X()` i `set_X()` dla jednego pola.
W `docs/notatki/oop.md` wyjaśnij, czym różnią się od `@property` i w której sytuacji
napisałbyś które.

**Zadanie 3 – konstruktor alternatywny**
Metoda `from_dict` jako `@classmethod`. W `docs/notatki/oop.md` opisz, jak ten sam efekt
osiąga się w C++ albo C#.

**Zadanie 4 – klasa zbioru**
Druga klasa zarządzająca kolekcją: dodawanie, usuwanie, szukanie po identyfikatorze,
szukanie po frazie, lista kategorii, co najmniej jedna metoda statystyczna. Plus `__len__`.

**Zadanie 5 – dziedziczenie**
Klasa potomna Twojej klasy głównej, z co najmniej jednym dodatkowym polem i **nadpisaną**
metodą `__str__`. Na przykład produkt na wagę, film wypożyczony długoterminowo, zabieg
z dojazdem. Użyj `super().__init__(...)`. Dziedziczenie jest w podstawie programowej
i pojawia się w zadaniach.

**Zadanie 6 – trwałość**
Zapis i czytanie z pliku JSON, z `ensure_ascii=False` i `encoding="utf-8"`. Dane muszą
przetrwać zamknięcie programu. Sprawdź, otwierając plik w edytorze – polskie znaki mają
być czytelne.

**Zadanie 7 – aplikacja konsolowa**
Menu z minimum sześcioma opcjami, w tym dodawanie, usuwanie, szukanie i statystyki.
Obowiązkowo funkcje czytające liczby **w pętli, aż do poprawnej wartości**. Program nie
może się wywalić na żadnym wejściu – przetestuj, wpisując litery tam, gdzie oczekiwana
jest liczba.

**Zadanie 8 – okno i kontrolki**
Aplikacja okienkowa korzystająca z **tego samego** `model.py`. Minimum: formularz
z trzema różnymi rodzajami kontrolek (`Entry`, `Spinbox`, `Combobox`), tabela `Treeview`
z kolumnami, pasek stanu, pasek przewijania.

**Zadanie 9 – menu i dialogi**
Pasek menu z minimum dwoma rozwijanymi menu i separatorem. Menu kontekstowe pod prawym
przyciskiem myszy. Minimum trzy różne okna dialogowe, w tym jedno pytające o potwierdzenie
i jedno o wartość. Obsłuż anulowanie.

**Zadanie 10 – zdarzenia**
Minimum dwa zdarzenia myszy i trzy klawiatury, w tym Enter zatwierdzający formularz
i Delete usuwający zaznaczenie. Skróty z `accelerator` w menu muszą **naprawdę działać**.

**Zadanie 11 – wyróżnienie wizualne**
Wiersze tabeli wyróżnione kolorem według stanu danych (`tags` i `tag_configure`).
W `docs/notatki/oop.md` wypisz, jak to samo robiłeś w Reakcie i w Androidzie.

**Zadanie 12 – dowód separacji**
Dopisz do modelu **nową metodę** i użyj jej w **obu** aplikacjach. Zrzuty ekranu obu.
W `docs/notatki/oop.md` odpowiedz: ile plików musiałeś zmienić i co by było, gdyby logika
siedziała w `console.py`?

**Zadanie 13 – dokumentacja i historia**
Docstringi w obu plikach według materiału 10. W README sekcja o uruchamianiu części
konsolowej i okienkowej. Minimum pięć commitów.

---

**Zadanie dodatkowe (dla chętnych)**
Napisz tę samą klasę w C++ albo C# – tylko klasę z prostym menu konsolowym, bez okien.
W `docs/notatki/oop.md` zestaw tabelą: enkapsulacja, pole statyczne, konstruktory,
gettery i settery, obsługa błędów. Po jednym zdaniu na komórkę.

Potem odpowiedz na pytanie, które warto sobie zadać przed egzaminem: **w którym języku
napisałbyś zadanie pierwsze i dlaczego?** Uzasadnij czymś konkretnym – ile kodu trzeba
napisać, co kompilator wychwyci za Ciebie, czego nie pamiętasz ze składni.
