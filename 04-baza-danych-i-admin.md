# Baza danych i panel administratora

**Technik programista | klasa 5 | materiał 04**

---

## Co się zmieni, a co zostanie

Zmieniamy **źródło danych**: zamiast listy w pliku `data.py` będzie tabela w bazie SQLite.
Zyskujemy trwałość, relacje, wyszukiwanie i – co dziś najważniejsze – **panel
administratora**, w którym sam dodasz produkty bez dotykania kodu.

A teraz rzecz, o którą w tym materiale naprawdę chodzi:

> **Adresy i format odpowiedzi nie zmienią się ani o przecinek.**

`/api/products/` dalej zwróci listę słowników z tymi samymi kluczami. Gdyby ktoś napisał
już frontend na wczorajszym API, dziś działałby bez jednej poprawki. To jest sens
oddzielania **kontraktu** (co API obiecuje) od **implementacji** (skąd bierze dane).

```
materiał 03:   /api/products/ ---> lista w pamięci   ---> JSON
materiał 04:   /api/products/ ---> SQLite przez ORM  ---> ten sam JSON
```

Na końcu porównasz odpowiedzi z obu wersji i powinny być identyczne poza kolejnością
i kilkoma nowymi polami.

### Wersje

| | Wersja |
|---|---|
| Python | 3.14 |
| Django | 6.1.1 |
| Baza | SQLite (wbudowana, nic nie instalujesz) |

---

## Krok 1 – ustawienia regionalne

W `core/settings.py` znajdź i zmień:

```python
LANGUAGE_CODE = "pl"
TIME_ZONE = "Europe/Warsaw"
USE_TZ = True
```

Pierwsza linia przetłumaczy panel administratora na polski – to nie kosmetyka, tylko
realna oszczędność czasu przy dodawaniu danych. Druga ustawia strefę czasową na naszą.

`USE_TZ = True` zostawiamy włączone. Oznacza, że Django **przechowuje daty w UTC**,
a przelicza je na naszą strefę dopiero przy wyświetlaniu. Brzmi jak komplikacja, ale to
jedyny sposób, żeby aplikacja nie zwariowała przy zmianie czasu i przy użytkownikach
z różnych krajów.

---

## Krok 2 – modele

**Model** to klasa Pythona opisująca tabelę w bazie. Jedna klasa – jedna tabela, jeden
atrybut – jedna kolumna. Nie piszesz SQL-a; Django wygeneruje go za Ciebie.

**`shop/models.py`**:

```python
from django.db import models


class Category(models.Model):
    name = models.CharField("nazwa", max_length=50, unique=True)

    class Meta:
        verbose_name = "kategoria"
        verbose_name_plural = "kategorie"
        ordering = ["name"]

    def __str__(self):
        return self.name


class Product(models.Model):
    name = models.CharField("nazwa", max_length=200)
    description = models.TextField("opis", blank=True)
    price = models.DecimalField("cena", max_digits=8, decimal_places=2)
    stock = models.PositiveIntegerField("stan magazynowy", default=0)
    category = models.ForeignKey(
        Category,
        on_delete=models.PROTECT,
        related_name="products",
        verbose_name="kategoria",
    )
    created_at = models.DateTimeField("data dodania", auto_now_add=True)

    class Meta:
        verbose_name = "produkt"
        verbose_name_plural = "produkty"
        ordering = ["name"]

    def __str__(self):
        return self.name
```

### Rozbiór na czynniki pierwsze

**Typy pól to nie ozdoba.** Każdy przekłada się na inny typ kolumny w bazie i na inną
walidację.

| Pole | Typ | Dlaczego akurat ten |
|---|---|---|
| `name` | `CharField` | krótki tekst, wymaga `max_length` |
| `description` | `TextField` | tekst dowolnej długości, bez limitu |
| `price` | `DecimalField` | **pieniądze nigdy nie w `FloatField`** |
| `stock` | `PositiveIntegerField` | baza nie pozwoli zapisać liczby ujemnej |
| `category` | `ForeignKey` | relacja do innej tabeli |
| `created_at` | `DateTimeField(auto_now_add=True)` | ustawia się sam przy tworzeniu |

**Dlaczego cena nie może być `FloatField`.** Liczby zmiennoprzecinkowe są przybliżone.
Wpisz w konsoli Pythona `0.1 + 0.2` – dostaniesz `0.30000000000000004`. Przy jednej cenie
nikt nie zauważy, przy sumowaniu tysiąca pozycji na fakturze różnica staje się realna
i prawnie kłopotliwa. `DecimalField` przechowuje wartość dokładnie. `max_digits=8`
i `decimal_places=2` znaczą: maksymalnie 8 cyfr, z czego 2 po przecinku, czyli do
999 999,99.

> Skąd się bierze ta czwórka na siedemnastym miejscu po przecinku i dlaczego nie jest to
> błąd Pythona, tylko konsekwencja sposobu zapisu liczb w pamięci – przeczytaj:
> [Zrozumienie niedokładności liczb zmiennoprzecinkowych](https://brylka.net/zrozumienie-niedokladnosci-liczb-zmiennoprzecinkowych).
> To ta sama rzecz, która pojawia się na egzaminie w pytaniach o typy danych.

**Pierwszy argument w cudzysłowie** (`"nazwa"`, `"cena"`) to etykieta wyświetlana
w panelu administratora. Nazwa pola w kodzie zostaje angielska, etykieta dla człowieka
jest polska – dokładnie ta reguła, którą przyjęliśmy w materiale 03.

**`ForeignKey` to relacja jeden-do-wielu.** Jedna kategoria ma wiele produktów, jeden
produkt należy do jednej kategorii. W bazie powstanie kolumna `category_id`
przechowująca identyfikator wiersza z tabeli kategorii.

- `on_delete=models.PROTECT` – **nie pozwól** usunąć kategorii, w której są produkty.
  Alternatywa `CASCADE` skasowałaby razem z kategorią wszystkie jej produkty, a to przy
  jednym nieuważnym kliknięciu w panelu oznacza utratę danych. Trzecia opcja, `SET_NULL`,
  zostawiłaby produkt bez kategorii.
- `related_name="products"` – nazwa, pod którą z kategorii sięgniesz do jej produktów:
  `category.products.all()`.

**`__str__`** decyduje, jak obiekt wygląda jako tekst. Bez tej metody panel administratora
wyświetli listę pozycji `Product object (1)`, `Product object (2)` i nie da się w tym
pracować. Dwie linie kodu, ogromna różnica.

**`Meta.ordering`** ustala domyślną kolejność. Bez niego baza może zwracać wiersze
w dowolnym porządku i lista produktów potrafi się przestawiać między odświeżeniami.

---

## Krok 3 – migracje

Model to opis w Pythonie. Baza o nim jeszcze nie wie. **Migracja** to wygenerowana
instrukcja zmiany struktury bazy.

```bash
python manage.py makemigrations
```

```
Migrations for 'shop':
  shop/migrations/0001_initial.py
    + Create model Category
    + Create model Product
```

Zajrzyj do wygenerowanego pliku – to zwykły Python, całkiem czytelny. Zobacz też, jaki
SQL z niego powstanie:

```bash
python manage.py sqlmigrate shop 0001
```

```sql
CREATE TABLE "shop_category" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT,
                              "name" varchar(50) NOT NULL UNIQUE);
...
```

**Zatrzymaj się na chwilę przy tym wyniku.** To jest ten sam `CREATE TABLE`, który pisałeś
ręcznie na lekcjach o bazach danych. Django nie robi żadnej magii – generuje SQL, który
umiesz przeczytać. Zwróć uwagę, że kolumna `id` powstała sama: Django dodaje klucz główny
do każdego modelu, jeśli sam go nie zdefiniujesz.

Teraz wykonanie migracji:

```bash
python manage.py migrate
```

Zniknie też ostrzeżenie o niezastosowanych migracjach, które towarzyszyło Ci od
materiału 03. Powstał plik **`db.sqlite3`** – cała baza w jednym pliku.

### Co do repozytorium, a co nie

| Plik | Do Gita? | Dlaczego |
|---|---|---|
| `shop/migrations/0001_initial.py` | **TAK** | to część kodu; bez niego kolega nie odtworzy struktury bazy |
| `db.sqlite3` | **NIE** | plik binarny, konfliktuje przy każdym zapisie, zawiera lokalne dane |

To jest jedna z rzeczy, którą najłatwiej zrobić odwrotnie. Migracje są w `.gitignore`
tylko u ludzi, którzy potem nie wiedzą, czemu nic nie działa po sklonowaniu projektu.
`db.sqlite3` jest już w naszym `.gitignore` z materiału 03 – sprawdź, czy na pewno:

```bash
git check-ignore -v backend/db.sqlite3
```

---

## Krok 4 – konto administratora

```bash
python manage.py createsuperuser
```

Podaj nazwę, e-mail (może być pusty) i hasło. Hasła nie widać podczas wpisywania – to
normalne, nie zepsute.

Uruchom serwer i wejdź na **http://localhost:8000/admin/**

Zobaczysz panel po polsku, a w nim sekcje „Uwierzytelnianie i autoryzacja". Twoich modeli
jeszcze nie ma, bo Django nie pokazuje ich, dopóki ich nie zarejestrujesz.

> Konto administratora zapisuje się w `db.sqlite3`, którego nie ma w repozytorium.
> Każdy tworzy własne, u siebie. Dopisz o tym zdanie w `README.md`.

---

## Krok 5 – rejestracja modeli w panelu

**`shop/admin.py`**:

```python
from django.contrib import admin
from .models import Category, Product


@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = ["name"]


@admin.register(Product)
class ProductAdmin(admin.ModelAdmin):
    list_display = ["name", "category", "price", "stock", "created_at"]
    list_filter = ["category"]
    search_fields = ["name", "description"]
    list_editable = ["price", "stock"]
```

Odśwież panel – pojawiły się „kategorie" i „produkty".

Same trzy linie z `list_display`, `list_filter` i `search_fields` zamieniają surową listę
w narzędzie, którym da się pracować:

- **`list_display`** – kolumny widoczne na liście. Bez tego zobaczysz wyłącznie to,
  co zwraca `__str__`.
- **`list_filter`** – panel filtrów z prawej strony.
- **`search_fields`** – pole wyszukiwania u góry; przeszukuje wskazane kolumny.
- **`list_editable`** – te pola edytujesz wprost na liście, bez wchodzenia w produkt.
  Wygodne przy poprawianiu stanów magazynowych. Pole z `list_editable` nie może być
  pierwsze w `list_display`, bo pierwsza kolumna jest linkiem do edycji.

Panel administratora dostajesz w Django **za darmo**, w komplecie z logowaniem,
uprawnieniami i historią zmian każdego obiektu. Napisanie czegoś takiego samodzielnie
to tygodnie pracy. To jeden z głównych powodów, dla których ten framework wygrał.

---

## Krok 6 – wprowadzenie danych

Teraz Ty. W panelu:

1. Dodaj **3 kategorie**.
2. Dodaj **minimum 8 produktów** rozłożonych po tych kategoriach.
3. Co najmniej jeden produkt ma mieć `stock` równy 0.
4. Ceny mają się różnić – przydadzą się do statystyk.

Zajmie to trzy minuty i jest to dokładnie ta czynność, którą w prawdziwej firmie wykonuje
osoba nietechniczna. Twoja aplikacja właśnie dostała **interfejs do zarządzania treścią**,
a Ty nie napisałeś do niego ani jednej linii.

Spróbuj też usunąć kategorię, w której są produkty. Django zablokuje operację –
to działa `on_delete=models.PROTECT`, które ustawiłeś w modelu.

---

## Krok 7 – ORM w konsoli

Zanim przepiszemy widoki, pobaw się zapytaniami:

```bash
python manage.py shell
```

```python
from shop.models import Category, Product

Product.objects.all()                          # wszystkie
Product.objects.count()                        # ile ich jest
Product.objects.first()                        # pierwszy wg Meta.ordering

Product.objects.filter(stock=0)                # niedostępne
Product.objects.filter(price__lt=200)          # tańsze niż 200
Product.objects.filter(name__icontains="mysz") # nazwa zawiera, bez rozróżniania wielkości liter
Product.objects.filter(category__name="peripherals")   # po polu z powiązanej tabeli

Product.objects.exclude(stock=0)               # wszystkie oprócz
Product.objects.order_by("-price")             # malejąco po cenie

p = Product.objects.get(id=1)                  # dokładnie jeden
p.category.name                                # przejście po relacji
p.category.products.count()                    # i w drugą stronę, przez related_name

print(Product.objects.filter(stock=0).query)   # zobacz wygenerowany SQL
```

Wyjście: `exit()`.

**Dwa podwójne podkreślenia warte zapamiętania.** `price__lt=200` to *lookup* – sposób
porównania (`lt`, `gte`, `icontains`, `in`, `isnull`). `category__name` to przejście
przez relację. Ten sam zapis z podwójnym podkreśleniem robi dwie różne rzeczy zależnie
od kontekstu i to jest jedna z rzeczy, które na początku mylą wszystkich.

**`filter()` kontra `get()`.** `filter()` zwraca zbiór wyników, choćby pusty. `get()`
zwraca jeden obiekt albo rzuca wyjątkiem: `DoesNotExist`, gdy nie ma nic,
`MultipleObjectsReturned`, gdy jest więcej niż jeden. Za chwilę wykorzystamy to do 404.

---

## Krok 8 – widoki na bazie danych

Tu następuje podmiana. **`shop/views.py`**:

```python
import sys
import time

import django

from django.conf import settings
from django.db import connection
from django.http import JsonResponse

from .models import Category, Product

APP_VERSION = "0.2.0"
START_TIME = time.time()


def json_response(data, status=200):
    """Nasza wersja JsonResponse: obsługuje listy i nie ucieka polskich znaków."""
    return JsonResponse(
        data,
        safe=False,
        status=status,
        json_dumps_params={"ensure_ascii": False},
    )


def product_to_dict(product):
    """Zamienia obiekt modelu na słownik gotowy do wysłania jako JSON."""
    return {
        "id": product.id,
        "name": product.name,
        "description": product.description,
        "price": str(product.price),
        "stock": product.stock,
        "category": product.category.name,
        "created_at": product.created_at.isoformat(),
    }
```

Numer wersji podniosłem do `0.2.0` – zmieniło się źródło danych, więc to nowa wersja
aplikacji. Tak działa wersjonowanie semantyczne z materiału 01.

### Funkcja `product_to_dict` jest tu najważniejsza

`JsonResponse` nie potrafi zamienić obiektu modelu na JSON. Ktoś musi wypisać, które pola
mają się znaleźć w odpowiedzi i w jakiej postaci. Ta funkcja to **serializacja** wykonana
ręcznie – i to jest dokładnie ta robota, którą w materiale 05 przejmie Django REST
Framework. Warto zrobić ją raz samodzielnie, żeby wiedzieć, co tam się dzieje.

Trzy pola wymagają konwersji, bo JSON nie zna ich typów:

**`str(product.price)`** – `Decimal` nie jest typem JSON-a. Zamieniamy na tekst, nie na
`float`, bo to by zniweczyło cały sens używania `DecimalField`. W odpowiedzi zobaczysz
`"price": "349.00"` w cudzysłowach. Tak samo domyślnie robi Django REST Framework
i tak samo działa większość API finansowych. Frontend zamieni to na liczbę u siebie.

**`product.created_at.isoformat()`** – data w formacie ISO 8601, czyli
`2026-09-09T14:32:10.123456+00:00`. Uniwersalny standard, który rozumie JavaScript
(`new Date(...)`) i każdy inny język. Widoczne `+00:00` to UTC – pamiętasz `USE_TZ`.

**`product.category.name`** – zamiast całego obiektu kategorii wstawiamy jej nazwę.
Dzięki temu odpowiedź wygląda **identycznie jak w materiale 03**, mimo że pod spodem
zmieniła się cała struktura danych. Kontrakt dotrzymany.

---

## Krok 9 – pozostałe widoki

Dopisz w tym samym pliku:

```python
def health(request):
    try:
        connection.ensure_connection()
    except Exception:
        return json_response({"status": "error", "database": False}, status=503)

    return json_response({"status": "ok", "database": True})


def info(request):
    return json_response({
        "application": "Shop API",
        "app_version": APP_VERSION,
        "python_version": sys.version.split()[0],
        "django_version": django.get_version(),
        "data_source": {
            "type": "sqlite",
            "engine": settings.DATABASES["default"]["ENGINE"].split(".")[-1],
            "record_count": Product.objects.count(),
            "categories": list(
                Category.objects.order_by("name").values_list("name", flat=True)
            ),
        },
        "uptime_seconds": round(time.time() - START_TIME, 1),
    })


def product_list(request):
    products = Product.objects.select_related("category")

    category = request.GET.get("category")
    if category:
        products = products.filter(category__name=category)

    return json_response([product_to_dict(p) for p in products])


def product_detail(request, product_id):
    try:
        product = Product.objects.select_related("category").get(id=product_id)
    except Product.DoesNotExist:
        return json_response({"error": f"Product {product_id} not found"}, status=404)

    return json_response(product_to_dict(product))
```

`shop/urls.py` **zostaje bez zmian**. To jest ta obiecana ciągłość: zmieniliśmy wnętrza
wszystkich funkcji, a mapa adresów pasuje dalej.

### Health, który wreszcie coś sprawdza

W materiale 03 `/api/health/` zawsze zwracał `ok`, bo nie miał czego sprawdzać. Teraz
próbuje nawiązać połączenie z bazą i przy niepowodzeniu zwraca **503 Service Unavailable**
– „usługa chwilowo niedostępna". To poprawny kod dla sytuacji „aplikacja stoi, ale nie
może działać, bo brakuje jej zależności". Monitoring reaguje na status HTTP, nie na treść,
więc samo `{"status": "error"}` z kodem 200 byłoby bezużyteczne.

### `select_related` i problem N+1

`product_to_dict` sięga do `product.category.name`. Bez `select_related` Django wykonałby
**osobne zapytanie do bazy dla każdego produktu**: jedno po listę i po jednym na kategorię
każdego z nich. Przy 8 produktach to 9 zapytań, przy 500 produktach – 501. Nazywa się to
**problemem N+1** i jest najczęstszą przyczyną wolnych aplikacji webowych.

`select_related("category")` mówi: pobierz produkty razem z kategoriami **jednym**
zapytaniem, używając SQL-owego `JOIN`. Jedna metoda, całą różnicę widać dopiero na
prawdziwych danych.

Sprawdź to sam – w konsoli:

```python
from django.db import connection, reset_queries
from django.test.utils import override_settings
from shop.models import Product

# uruchom shell z DEBUG=True (domyślnie jest)
reset_queries()
for p in Product.objects.all():
    p.category.name
print(len(connection.queries))          # tyle zapytań poszło do bazy

reset_queries()
for p in Product.objects.select_related("category"):
    p.category.name
print(len(connection.queries))          # a teraz tyle
```

Różnica między tymi dwiema liczbami jest całą lekcją.

---

## Krok 10 – sprawdzenie kontraktu

Uruchom serwer i porównaj z tym, co pamiętasz z materiału 03:

- http://localhost:8000/api/health/
- http://localhost:8000/api/info/ – `"type"` mówi teraz `sqlite`
- http://localhost:8000/api/products/
- http://localhost:8000/api/products/1/
- http://localhost:8000/api/products/999/ – nadal 404
- http://localhost:8000/api/products/?category=peripherals

Klucze te same, doszły `description` i `created_at`, `price` jest teraz tekstem.
Adresy, statusy i struktura – bez zmian.

---

## Krok 11 – dane startowe (fixtures)

Problem: `db.sqlite3` nie trafia do repozytorium, więc kolega po sklonowaniu projektu
dostanie pustą bazę. Rozwiązanie to **fixtures** – dane wyeksportowane do pliku JSON,
który już do repozytorium trafia.

```bash
mkdir shop\fixtures
python manage.py dumpdata shop --indent 2 -o shop/fixtures/initial_data.json
```

> Użyj przełącznika `-o`, a nie przekierowania `>`. PowerShell zapisuje przekierowany
> tekst w kodowaniu, które rozjeżdża polskie znaki – i potem plik się nie wczytuje.

Wczytanie u siebie albo u kolegi:

```bash
python manage.py loaddata initial_data
```

Od teraz pełna instrukcja uruchomienia projektu z czystego klona brzmi:

```bash
pip install -r requirements.txt
python manage.py migrate            # tworzy strukturę bazy
python manage.py loaddata initial_data   # wypełnia danymi
python manage.py createsuperuser    # własne konto do panelu
python manage.py runserver
```

**Te cztery komendy w tej kolejności mają się znaleźć w `README.md`.** To jest teraz
najważniejszy fragment Twojej dokumentacji.

---

## Krok 12 – sprzątanie i zapis

Plik `shop/data.py` nie jest już do niczego potrzebny:

```bash
git rm shop/data.py
```

Sprawdź, czy do repozytorium nie wchodzi baza:

```bash
git status --short      # NIE może tu być db.sqlite3
```

Podział na commity:

```
feat(models): modele Category i Product
chore(db): migracja initial
feat(admin): rejestracja i konfiguracja modeli w panelu
refactor(api): widoki korzystają z ORM zamiast listy w pamięci
feat(api): health sprawdza połączenie z bazą
chore(data): dane startowe jako fixture
docs(readme): instrukcja uruchomienia z migracją i fixtures
```

---

## Sprawdź, czy rozumiesz

1. Czym różni się model od migracji i po co istnieją osobno?
2. Dlaczego `shop/migrations/` trafia do repozytorium, a `db.sqlite3` nie?
3. Co się stanie przy próbie usunięcia kategorii z produktami i która linia kodu o tym
   decyduje?
4. Dlaczego cena jest wysyłana jako `"349.00"`, a nie `349.0`?
5. Co robi `select_related("category")` i ile zapytań oszczędza przy 100 produktach?
6. Kiedy użyjesz `filter()`, a kiedy `get()`?
7. Dlaczego adresy w `urls.py` nie wymagały żadnej zmiany?

---

## Częste błędy

| Objaw | Przyczyna |
|---|---|
| `no such table: shop_product` | brak `python manage.py migrate` |
| `You are trying to add a non-nullable field without a default` | dodałeś pole do modelu z danymi; podaj wartość domyślną albo `null=True` |
| `Object of type Decimal is not JSON serializable` | zapomniany `str()` przy cenie |
| `Object of type datetime is not JSON serializable` | zapomniany `.isoformat()` przy dacie |
| modeli nie ma w panelu | brak rejestracji w `admin.py` |
| lista w panelu pokazuje `Product object (1)` | brak metody `__str__` w modelu |
| `RelatedObjectDoesNotExist` | produkt bez kategorii; `ForeignKey` jest wymagane |
| po sklonowaniu baza jest pusta | brak `loaddata` – i brak wzmianki o tym w README |

---

## Czego jeszcze nie ma

Świadome luki, żeby nic Cię później nie zaskoczyło:

- **Tylko GET.** Dodawanie i edycja idą na razie przez panel administratora. Metody POST,
  PUT i DELETE dojdą po Reakcie, razem z walidacją danych wejściowych.
- **Brak CORS.** Przy pierwszej próbie odpytania tego API z Reacta przeglądarka
  je zablokuje. To nie będzie awaria, tylko następny temat.
- **Brak paginacji.** Przy 10 produktach nieistotne, przy 10 000 endpoint zwróci wszystko
  naraz i zabije przeglądarkę.
- **Brak testów.** Sprawdzasz ręcznie w przeglądarce. Wystarczy na teraz, przestanie
  wystarczać przy pierwszej większej zmianie.
- **`SECRET_KEY` i `DEBUG` siedzą w `settings.py`.** Przed wdrożeniem trafią do `.env`.
- **SQLite.** Świetny do nauki i do jednego użytkownika. Przy wdrożeniu zwykle zamienia
  się go na PostgreSQL – a dzięki ORM będzie to zmiana kilku linii w ustawieniach.

---

## Co dalej

Materiał 05: **React konsumujący to API**. Backend zostaje dokładnie taki, jaki jest.
Napiszesz frontend, który odpyta `/api/products/`, wyświetli listę i szczegóły produktu,
i po drodze natkniesz się na CORS.

---

# Zadania

Pracujesz dalej na swojej domenie z materiału 03. Sklep jest zajęty jako przykład.

**Zadanie 1 – modele**
Dwa modele powiązane relacją `ForeignKey`, odpowiadające Twojej domenie (książka –
autor, seans – film, zlecenie – klient). Każdy z sensownym `__str__`, `Meta.ordering`
i polskimi etykietami pól. Model główny ma mieć minimum 5 pól, w tym jedno liczbowe
i jedno datowe.

**Zadanie 2 – uzasadnienie typów**
W `docs/baza.md` tabela wszystkich pól: nazwa, typ Django, typ kolumny w SQL i **jedno
zdanie uzasadnienia**, dlaczego akurat ten typ. Wynik `python manage.py sqlmigrate`
wklej jako dowód.

**Zadanie 3 – migracje**
Wygeneruj i wykonaj migracje. Obejrzyj SQL. Migracje zacommituj, bazy nie.

**Zadanie 4 – panel**
Zarejestruj oba modele. Skonfiguruj `list_display`, `list_filter` i `search_fields`
tak, żeby dało się realnie pracować. Wprowadź **minimum 12 rekordów** przez panel.

**Zadanie 5 – przepisanie widoków**
Wszystkie endpointy z materiału 03 mają działać na bazie, zwracając **ten sam format**.
Zapisz odpowiedzi sprzed i po zmianie i porównaj je w `docs/kontrakt.md`.

**Zadanie 6 – ORM**
W `docs/orm.md` zapisz 8 różnych zapytań do swojej bazy z konsoli, wraz z wynikami.
Mają wystąpić: `filter`, `exclude`, `get`, `count`, `order_by`, lookup z `__` oraz
przejście przez relację w obie strony.

**Zadanie 7 – statystyki na agregacji**
Przepisz endpoint `/api/statistics/` z materiału 03 tak, żeby liczyła go **baza**,
a nie Python:

```python
from django.db.models import Avg, Min, Max, Count
```

Minimum: liczba rekordów w każdej kategorii, średnia, minimum i maksimum pola liczbowego.
Żadnych pętli po wynikach.

**Zadanie 8 – health i info**
`/api/health/` sprawdza połączenie z bazą i zwraca 503 przy awarii. `/api/info/` mówi
`sqlite` i podaje aktualną liczbę rekordów oraz listę kategorii pobraną z bazy.

**Zadanie 9 – fixtures i README**
Wyeksportuj dane do `fixtures/initial_data.json` i zacommituj. Zaktualizuj `README.md`
o pełną instrukcję uruchomienia z czystego klona. Sprawdź ją: sklonuj własne repozytorium
do nowego katalogu i uruchom projekt **wyłącznie** według README.

**Zadanie 10 – historia**
Minimum 6 commitów według konwencji.

---

**Zadanie dodatkowe (dla chętnych)**
Dodaj do modelu głównego pole `slug` (`SlugField(unique=True)`) i drugi endpoint
szczegółów, dostępny pod adresem tekstowym zamiast liczbowego – na przykład
`/api/products/mysz-bezprzewodowa-swift/`. Podpowiedzi: konwerter `<slug:...>` w `path()`
oraz `prepopulated_fields` w klasie admina. W `docs/slug.md` napisz, dlaczego adresy ze
slugiem są lepsze od tych z identyfikatorem – i podaj jeden przypadek, w którym są gorsze.