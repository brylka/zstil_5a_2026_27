# Django krok po kroku – pierwsze API

**Technik programista | klasa 5 | materiał 03**

---

## Co zbudujemy

Backend sklepu, który na razie **nie ma bazy danych**. Produkty siedzą w zwykłej liście
słowników w pliku `.py`. Aplikacja odpowiada JSON-em na kilka adresów:

| Adres | Zwraca |
|---|---|
| `/api/health/` | czy serwer żyje |
| `/api/info/` | wersje, źródło danych, czas działania |
| `/api/products/` | listę produktów |
| `/api/products/1/` | jeden produkt |
| `/api/products/?category=peripherals` | listę przefiltrowaną |

Tylko metoda **GET**. Żadnego dodawania, edycji, usuwania – to przyjdzie później.

### Wersje, na których pracujemy

| | Wersja | Uwaga |
|---|---|---|
| Python | **3.14** | Django 6.1 wymaga minimum 3.12 |
| Django | **6.1.1** | seria wydana w sierpniu 2026 |

Jeśli na Twoim komputerze jest coś nowszego, to dobrze – trzymamy się najnowszych
stabilnych wydań. Ważne tylko, żeby **wpisać rzeczywiste numery do `requirements.txt`**
i do `README.md` projektu, a nie przepisać tych z materiału.

### Dlaczego zaczynamy od listy, a nie od bazy

Bo baza danych to osobny problem i osobny zestaw błędów. Jeśli zaczniemy od wszystkiego
naraz, przy pierwszym błędzie nie będziesz wiedział, czy zawiódł adres URL, widok, model,
migracja czy sam SQLite.

Na razie dane po prostu **są**. Skupiamy się na jednej rzeczy: jak żądanie z przeglądarki
zamienia się w odpowiedź JSON. Kiedy to będzie oczywiste, podmienimy listę na model
i SQLite – i będzie to zmiana **jednego pliku**, bo reszta zostanie bez zmian.

To nie jest sztuczka dydaktyczna, tylko normalny sposób pracy: najpierw działający szkielet
na atrapie danych, potem prawdziwe źródło.

---

## Krok 0 – sprawdzenie środowiska

```bash
python --version
```

Django 6.1 wymaga **Pythona 3.12 lub nowszego** i oficjalnie wspiera 3.12, 3.13 i 3.14.
Instalujemy **3.14** – najnowszą stabilną serię (3.15 jest na wrzesień 2026 dopiero
w wersji kandydującej i Django jej jeszcze nie obsługuje).

Jeśli Windows odpowiada „Python nie jest rozpoznawalny", spróbuj `py --version`.
Jeśli nadal nic – instalacja z https://www.python.org/downloads/ i **zaznaczone**
„Add Python to PATH" podczas instalacji.

Masz starszego Pythona, na przykład 3.11? Django 6 na nim **nie zadziała** – trzeba
zainstalować nowszego. Stara wersja może zostać w systemie, nie przeszkadza.

---

## Krok 1 – katalog projektu i środowisko wirtualne

```bash
mkdir sklep-fullstack
cd sklep-fullstack
mkdir backend
cd backend

python -m venv venv
venv\Scripts\activate            # Windows
# source venv/bin/activate       # Linux / macOS
```

Po aktywacji na początku wiersza pojawia się `(venv)`. Jeśli go nie ma – środowisko
nie jest aktywne i wszystko, co zainstalujesz, wyląduje w systemie zamiast w projekcie.

**Czym jest środowisko wirtualne.** Osobnym, prywatnym kompletem bibliotek dla tego jednego
projektu. Bez niego wszystkie projekty na komputerze dzielą jedną wersję Django – i w dniu,
w którym jeden potrzebuje Django 5, a drugi Django 6, masz problem nie do rozwiązania.
Katalog `venv/` **nigdy nie trafia do repozytorium**.

> PowerShell potrafi zablokować aktywację komunikatem o „execution policy". Wtedy raz:
> `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` i potwierdź.

---

## Krok 2 – instalacja Django

```bash
python -m pip install --upgrade pip
pip install django
django-admin --version
```

Powinno wypisać **6.1.1** albo nowszą wersję z serii 6.1. `pip install django` zawsze
bierze najnowszą stabilną – nie trzeba podawać numeru.

Teraz zapis listy zależności:

```bash
pip freeze > requirements.txt
```

Plik będzie wyglądał mniej więcej tak:

```
asgiref==3.10.0
Django==6.1.1
sqlparse==0.5.3
```

Django ciągnie za sobą dwie paczki: `asgiref` (obsługa kodu asynchronicznego) i `sqlparse`
(formatowanie zapytań SQL). Nie instalowałeś ich świadomie – to **zależności zależności**.
Właśnie dlatego `requirements.txt` generuje się komendą, a nie pisze ręcznie z pamięci.

Zwróć uwagę na `==` przy wersjach. To **przypięcie wersji**: kolega, który jutro wykona
`pip install -r requirements.txt`, dostanie dokładnie Django 6.1.1, a nie „cokolwiek jest
najnowsze". Bez tego projekt, który dziś działa, za pół roku potrafi się nie uruchomić.

**Powtarzaj `pip freeze > requirements.txt` po każdej nowej instalacji** – to najczęściej
zapominany krok w całym projekcie.

---

## Krok 3 – utworzenie projektu

```bash
django-admin startproject core .
```

Kropka na końcu jest istotna. Bez niej Django utworzy katalog w katalogu i skończysz
ze ścieżką `backend/core/core/settings.py`, w której łatwo się pogubić.

Powstało:

```
backend/
├── manage.py           # narzędzie do wydawania komend
└── core/
    ├── settings.py     # ustawienia całej aplikacji
    ├── urls.py         # główna mapa adresów
    ├── wsgi.py         # uruchamianie na serwerze produkcyjnym
    ├── asgi.py         # to samo, ale asynchronicznie
    └── __init__.py     # informuje Pythona, że to pakiet
```

W tym materiale dotkniemy tylko `settings.py` i `urls.py`. Pozostałe pliki zaczną mieć
znaczenie dopiero przy wdrożeniu.

---

## Krok 4 – pierwsze uruchomienie

```bash
python manage.py runserver
```

Wejdź na **http://localhost:8000**. Powinna pojawić się strona powitalna z rakietą.

W terminalu zobaczysz ostrzeżenie o niezastosowanych migracjach. **Zignoruj je.**
Django zakłada, że użyjesz bazy danych; my na razie nie używamy. Ostrzeżenie zniknie
w kolejnym materiale.

Serwer zatrzymujesz `Ctrl+C`. Podczas pracy zostawiasz go uruchomionego – sam wykrywa
zmiany w plikach i przeładowuje się.

---

## Krok 5 – aplikacja wewnątrz projektu

W nowym terminalu (albo po zatrzymaniu serwera):

```bash
python manage.py startapp shop
```

**Projekt kontra aplikacja.** Projekt (`core`) to konfiguracja całości: ustawienia,
główna mapa adresów, uruchamianie. Aplikacja (`shop`) to jeden obszar funkcjonalny:
modele, widoki, adresy dotyczące jednej rzeczy. Duży serwis ma jeden projekt i kilka
aplikacji – `shop`, `accounts`, `payments`. Nasz ma jedną i to w zupełności wystarczy.

Zarejestruj aplikację w `core/settings.py`, dopisując ją na końcu listy:

```python
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    "shop",                       # <-- nasza aplikacja
]
```

Dla samych widoków i adresów ten wpis nie jest jeszcze konieczny, ale stanie się
niezbędny, gdy dojdą modele. Dopisujemy od razu, żeby później nie szukać przyczyny
błędu, który wygląda na zupełnie niezwiązany.

---

## Krok 6 – dane

Nowy plik **`shop/data.py`**:

```python
PRODUCTS = [
    {
        "id": 1,
        "name": "Klawiatura mechaniczna K80",
        "price": 349.00,
        "category": "peripherals",
        "stock": 12,
    },
    {
        "id": 2,
        "name": "Mysz bezprzewodowa Swift",
        "price": 129.99,
        "category": "peripherals",
        "stock": 40,
    },
    {
        "id": 3,
        "name": "Monitor 27 cali QHD",
        "price": 1199.00,
        "category": "monitors",
        "stock": 5,
    },
    {
        "id": 4,
        "name": "Słuchawki nauszne Cliff",
        "price": 289.00,
        "category": "audio",
        "stock": 0,
    },
    {
        "id": 5,
        "name": "Podkładka pod mysz XL",
        "price": 59.00,
        "category": "peripherals",
        "stock": 87,
    },
]
```

Zwróć uwagę na dwie rzeczy, które nie są przypadkowe:

- **Każdy słownik ma te same klucze.** To jest kontrakt Twojego API. Jeśli jeden produkt
  będzie miał pole `name`, a drugi `title`, frontend się wywróci. Baza danych wymusi tę
  spójność sama, ale dziś pilnujesz jej Ty.
- **Pole `id` jest unikalne.** Po nim będziemy wyszukiwać pojedynczy produkt. W bazie
  danych to samo pole pojawi się automatycznie jako klucz główny.

Jeden produkt ma celowo `stock` równy zero, a ceny są różne – przyda się przy
filtrowaniu i statystykach.

### Dlaczego wszystko jest po angielsku

Zauważ, że **nazwy zmiennych, funkcji, kluczy i adresów są po angielsku**, a po polsku
zostały tylko komentarze i same wartości (`"Słuchawki nauszne Cliff"`). Tak się pisze
kod zawodowo i są ku temu konkretne powody:

- Twój kod miesza się z kodem Django, który jest angielski. `product.stock` czyta się
  płynnie, `produkt.stan_magazynowy` obok `models.CharField` już nie.
- Polskie znaki w identyfikatorach to proszenie się o kłopoty – w adresach URL, nazwach
  plików i konsolach o dziwnym kodowaniu potrafią się rozsypać.
- Nazwy kluczy JSON stają się częścią API. Za miesiąc podepnie się pod nie React,
  a potem Android – i wszędzie będzie `product.price`.
- Praktycznie każdy pracodawca, dokumentacja i przykład w internecie zakłada angielski.

**Reguła na cały rok:** identyfikatory po angielsku, komentarze i teksty dla użytkownika
po polsku. Byle konsekwentnie – najgorsze jest mieszanie w obrębie jednego pliku.

---

## Krok 7 – pierwszy widok

**Widok** w Django to funkcja, która przyjmuje żądanie i zwraca odpowiedź. Tyle.

**`shop/views.py`** – wykasuj to, co wygenerowało Django, i wpisz:

```python
from django.http import JsonResponse


def health(request):
    return JsonResponse({"status": "ok"})
```

`JsonResponse` bierze słownik Pythona, zamienia go na tekst JSON i dokleja nagłówek
`Content-Type: application/json`, po którym przeglądarka i frontend poznają, że to dane,
a nie strona HTML.

### Po co endpoint `/health/`

To standard w każdej poważnej aplikacji. Odpowiada na jedno pytanie: **czy ta usługa żyje?**
Odpytuje go monitoring co kilkadziesiąt sekund, load balancer przed skierowaniem na serwer
ruchu, a system wdrożeniowy zaraz po starcie nowej wersji. Kosztuje pięć linii, a jest
pierwszą rzeczą, którą sprawdzasz, gdy „nic nie działa".

To także najprostszy możliwy sposób sprawdzenia, czy Twoja konfiguracja adresów jest
poprawna – bo nie zależy od żadnych danych.

---

## Krok 8 – adresy

Django nie zgadnie samo, że funkcja `health` ma odpowiadać na `/api/health/`. Trzeba to
wpisać.

Nowy plik **`shop/urls.py`**:

```python
from django.urls import path
from . import views

urlpatterns = [
    path("health/", views.health),
]
```

Następnie **`core/urls.py`** – podłączenie adresów aplikacji do projektu:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/", include("shop.urls")),
]
```

**Jak Django dopasowuje adres.** Przychodzi żądanie `/api/health/`. Django przegląda
`core/urls.py` **od góry do dołu**. `admin/` nie pasuje. `api/` pasuje do początku – więc
Django odcina `api/`, a resztę (`health/`) przekazuje do `shop/urls.py`. Tam `health/`
pasuje w całości, więc wywoływana jest funkcja `views.health`.

```
/api/health/  ->  core/urls.py  ->  odcina "api/"  ->  shop/urls.py  ->  views.health()
```

Ten podział na dwa pliki wydaje się nadmiarowy przy jednym adresie, ale dzięki niemu
aplikacja jest przenośna: żeby całe API wystawić pod `/v2/` zamiast `/api/`, zmieniasz
jeden znak w jednym miejscu.

Uruchom serwer i wejdź na **http://localhost:8000/api/health/**

```json
{"status": "ok"}
```

Pierwszy własny endpoint działa.

---

## Krok 9 – lista produktów

Dopisz w **`shop/views.py`**:

```python
from django.http import JsonResponse
from .data import PRODUCTS


def health(request):
    return JsonResponse({"status": "ok"})


def product_list(request):
    return JsonResponse(PRODUCTS, safe=False)
```

W **`shop/urls.py`**:

```python
urlpatterns = [
    path("health/", views.health),
    path("products/", views.product_list),
]
```

### Skąd `safe=False`

Bez tego argumentu `JsonResponse` odmawia zamiany listy na JSON i rzuca błędem. To nie
jest kaprys – to zabezpieczenie przed starą podatnością przeglądarek, w której JSON
zaczynający się od `[` dawał się wykraść z innej strony. Nowoczesne przeglądarki już na to
nie pozwalają, ale Django domyślnie przyjmuje słownik i każe świadomie potwierdzić, że
naprawdę chcesz zwrócić tablicę.

Zapamiętaj po prostu: **słownik zwracasz normalnie, listę z `safe=False`**.

Wejdź na **http://localhost:8000/api/products/**

---

## Krok 10 – polskie znaki

W odpowiedzi zobaczysz coś takiego:

```json
{"name": "S\u0142uchawki nauszne Cliff"}
```

To jest **poprawny JSON** – każdy klient go rozkoduje i wyświetli „Słuchawki". Ale
nieczytelny podczas pracy, a Ty będziesz na to patrzeć przez cały rok.

Zamiast dopisywać ten sam argument przy każdej odpowiedzi, dodaj w `views.py` jedną
własną funkcję i używaj wyłącznie jej:

```python
from django.http import JsonResponse
from .data import PRODUCTS


def json_response(data, status=200):
    """Nasza wersja JsonResponse: obsługuje listy i nie ucieka polskich znaków."""
    return JsonResponse(
        data,
        safe=False,
        status=status,
        json_dumps_params={"ensure_ascii": False},
    )


def health(request):
    return json_response({"status": "ok"})


def product_list(request):
    return json_response(PRODUCTS)
```

To jest pierwszy moment w tym projekcie, w którym stosujesz zasadę **DRY** (*Don't Repeat
Yourself*): zamiast powtarzać trzy argumenty w dziesięciu miejscach, zapisujesz je raz.
Gdy za miesiąc będziesz chciał zmienić coś we wszystkich odpowiedziach naraz, poprawisz
jedną funkcję.

---

## Krok 11 – pojedynczy produkt

```python
def product_detail(request, product_id):
    for product in PRODUCTS:
        if product["id"] == product_id:
            return json_response(product)

    return json_response(
        {"error": f"Product {product_id} not found"},
        status=404,
    )
```

W `shop/urls.py`:

```python
path("products/<int:product_id>/", views.product_detail),
```

### Co się tu dzieje

`<int:product_id>` to **konwerter**. Mówi Django: w tym miejscu adresu spodziewaj się
liczby całkowitej, zamień ją na `int` i przekaż do funkcji pod nazwą `product_id`.
Nazwa w nawiasach musi być identyczna jak nazwa parametru funkcji.

Skutek uboczny, który warto docenić: adres `/api/products/abc/` **w ogóle nie dotrze**
do Twojej funkcji, bo `abc` nie jest liczbą. Django odrzuci go wcześniej. Jedna linijka
konfiguracji zastąpiła walidację, którą inaczej trzeba by pisać ręcznie.

### Status 404 i dlaczego jest ważny

Gdyby brakujący produkt zwracał `200 OK` z pustym słownikiem, frontend nie miałby jak
odróżnić „produkt nie istnieje" od „produkt istnieje, tylko nie ma żadnych danych".
Kod statusu jest częścią odpowiedzi – niesie informację, której nie ma w samej treści.

Sprawdź oba przypadki:
- http://localhost:8000/api/products/3/
- http://localhost:8000/api/products/999/

W drugim przypadku otwórz narzędzia deweloperskie (F12), zakładka **Network** – zobaczysz
tam czerwone `404`. Sama treść odpowiedzi wygląda podobnie, różnica jest w statusie.

---

## Krok 12 – filtrowanie

```python
def product_list(request):
    category = request.GET.get("category")

    if category:
        products = [p for p in PRODUCTS if p["category"] == category]
    else:
        products = PRODUCTS

    return json_response(products)
```

`request.GET` to słownik parametrów z adresu, czyli tego, co po znaku `?`. Metoda `.get()`
zwraca `None`, gdy parametru nie ma – dlatego nie trzeba tu żadnego `try`.

Sprawdź:
- http://localhost:8000/api/products/
- http://localhost:8000/api/products/?category=peripherals
- http://localhost:8000/api/products/?category=toys (pusta lista, ale status 200)

Ostatni przypadek jest ciekawy dydaktycznie: **pusta lista to nie jest błąd**. Zapytanie
było poprawne, odpowiedź brzmi „nic takiego nie mam". Status zostaje 200. Odróżnianie
„zapytałeś źle" (400), „nie ma takiego zasobu" (404) i „jest zasób, ale pusty" (200)
to jedna z tych rzeczy, które odróżniają API napisane dobrze od napisanego byle jak.

---

## Krok 13 – endpoint informacyjny

Ten endpoint mówi, **co dokładnie jest teraz uruchomione**. W pracy to pierwsza rzecz,
którą sprawdzasz, gdy „u mnie działa, a na serwerze nie" – zwykle okazuje się, że na
serwerze chodzi wersja sprzed trzech dni.

Na górze `views.py`:

```python
import sys
import time

import django

from django.http import JsonResponse
from .data import PRODUCTS

APP_VERSION = "0.1.0"
START_TIME = time.time()
```

I sam widok:

```python
def info(request):
    return json_response({
        "application": "Shop API",
        "app_version": APP_VERSION,
        "python_version": sys.version.split()[0],
        "django_version": django.get_version(),
        "data_source": {
            "type": "in-memory list",
            "record_count": len(PRODUCTS),
            "categories": sorted({p["category"] for p in PRODUCTS}),
        },
        "uptime_seconds": round(time.time() - START_TIME, 1),
    })
```

W `shop/urls.py`:

```python
path("info/", views.info),
```

### Trzy rzeczy warte uwagi

**`START_TIME` liczy się raz.** Kod na poziomie pliku wykonuje się przy pierwszym imporcie
modułu, czyli przy starcie serwera – nie przy każdym żądaniu. Dlatego różnica dat daje czas
działania aplikacji. Odśwież stronę kilka razy i patrz, jak liczba rośnie.

**`sorted({...})`** – to nawiasy klamrowe, więc powstaje zbiór, który sam usuwa duplikaty;
`sorted` zamienia go na posortowaną listę. JSON nie zna typu „zbiór", więc bez `sorted`
dostałbyś błąd. Krótki zapis, ale robi dwie rzeczy naraz – warto go rozumieć, a nie tylko
przepisać.

**`"type": "in-memory list"`** – to pole jest tu po coś. W następnym materiale zmieni się
na `"sqlite"` i będzie widać w jednym miejscu, że przesiadka na bazę faktycznie nastąpiła.
Endpoint informacyjny nie jest ozdobnikiem, tylko narzędziem diagnostycznym.

---

## Krok 14 – komplet plików

Tak wygląda `shop/views.py` po wszystkich krokach:

```python
import sys
import time

import django

from django.http import JsonResponse
from .data import PRODUCTS

APP_VERSION = "0.1.0"
START_TIME = time.time()


def json_response(data, status=200):
    """Nasza wersja JsonResponse: obsługuje listy i nie ucieka polskich znaków."""
    return JsonResponse(
        data,
        safe=False,
        status=status,
        json_dumps_params={"ensure_ascii": False},
    )


def health(request):
    return json_response({"status": "ok"})


def info(request):
    return json_response({
        "application": "Shop API",
        "app_version": APP_VERSION,
        "python_version": sys.version.split()[0],
        "django_version": django.get_version(),
        "data_source": {
            "type": "in-memory list",
            "record_count": len(PRODUCTS),
            "categories": sorted({p["category"] for p in PRODUCTS}),
        },
        "uptime_seconds": round(time.time() - START_TIME, 1),
    })


def product_list(request):
    category = request.GET.get("category")

    if category:
        products = [p for p in PRODUCTS if p["category"] == category]
    else:
        products = PRODUCTS

    return json_response(products)


def product_detail(request, product_id):
    for product in PRODUCTS:
        if product["id"] == product_id:
            return json_response(product)

    return json_response(
        {"error": f"Product {product_id} not found"},
        status=404,
    )
```

I `shop/urls.py`:

```python
from django.urls import path
from . import views

urlpatterns = [
    path("health/", views.health),
    path("info/", views.info),
    path("products/", views.product_list),
    path("products/<int:product_id>/", views.product_detail),
]
```

Cała aplikacja to **dwa pliki z kodem i jeden z danymi**. Mniej się nie da, a wszystko,
co potrzebne, jest.

---

## Krok 15 – testowanie inaczej niż przez przeglądarkę

Przeglądarka wysyła wyłącznie żądania GET, więc na razie wystarcza. Ale za dwa materiały
będziesz potrzebować POST-a i wtedy jest bezużyteczna. Zainstaluj **Thunder Client**
(rozszerzenie do VS Code) albo używaj `curl` z terminala:

```bash
curl http://localhost:8000/api/products/
curl -i http://localhost:8000/api/products/999/     # -i pokazuje nagłówki i status
```

W Thunder Client utwórz kolekcję `Sklep API` i zapisz w niej wszystkie cztery adresy.
Będziesz ją rozbudowywać przez cały rok, a przy okazji powstaje z tego dokumentacja
do oddania.

---

## Krok 16 – zapis w repozytorium

W katalogu głównym `sklep-fullstack/` plik `.gitignore` (patrz materiał 01):

```gitignore
__pycache__/
*.py[cod]
venv/
db.sqlite3
.env
.vscode/
.idea/
```

Sprawdź i zapisz:

```bash
git status --short          # NIE może tu być venv/ ani __pycache__/
git add .
git commit -m "feat(api): endpointy GET na danych z listy"
git push
```

Historia ma pokazywać kroki, nie jeden skok. Sensowny podział na commity:

```
chore(backend): szkielet projektu Django
feat(api): endpoint /api/health/
feat(api): lista produktów z danych statycznych
feat(api): szczegóły produktu i obsługa 404
feat(api): filtrowanie po kategorii
feat(api): endpoint /api/info/ z wersjami i stanem danych
```

---

## Sprawdź, czy rozumiesz

Odpowiedz sobie bez zaglądania wyżej. Jeśli któreś pytanie sprawia kłopot, wróć do
odpowiedniego kroku – następny materiał założy, że to jest jasne.

1. Co dokładnie robi `include("shop.urls")` w pliku `core/urls.py`?
2. Dlaczego lista wymaga `safe=False`, a słownik nie?
3. Skąd funkcja `product_detail` bierze wartość parametru `product_id`?
4. Dlaczego pusta lista wyników zwraca 200, a nie 404?
5. Dlaczego `START_TIME` nie zmienia się przy każdym odświeżeniu strony?
6. Co się stanie, gdy zamienisz kolejność dwóch wpisów w `urlpatterns`? A co, gdyby oba
   pasowały do tego samego adresu?

---

## Częste błędy

| Objaw | Przyczyna |
|---|---|
| `TypeError: In order to allow non-dict objects to be serialized` | zwracasz listę bez `safe=False` |
| `Page not found (404)` na własnym adresie | brak ukośnika na końcu, literówka w `urls.py` albo brak `include` |
| `ModuleNotFoundError: No module named 'django'` | nieaktywne środowisko wirtualne – brak `(venv)` w wierszu |
| `Object of type set is not JSON serializable` | próbujesz zwrócić zbiór; opakuj w `sorted()` lub `list()` |
| zmiany w kodzie nie działają | serwer nie wystartował po błędzie – sprawdź terminal |
| `NameError: name 'PRODUCTS' is not defined` | brak importu `from .data import PRODUCTS` |

---

## Co dalej

W materiale 04 podmienimy `data.py` na **model Django i bazę SQLite**. Zmienią się tylko
wnętrza funkcji – adresy i format odpowiedzi zostaną identyczne. To będzie namacalny dowód,
że kontrakt API jest niezależny od tego, skąd biorą się dane. Potem dojdzie Django REST
Framework, który większość dzisiejszego kodu napisze za nas – ale dopiero wtedy będziesz
wiedział, **co** on właściwie robi.

---

# Zadania

Sklep jest zajęty jako przykład – **wybierasz własną domenę** i prowadzisz ją przez cały
rok. Zgłoś wybór na liście w klasie; domeny się nie powtarzają, kto pierwszy, ten lepszy.

**Propozycje:** biblioteka · wypożyczalnia sprzętu narciarskiego · kino (repertuar
i seanse) · siłownia (karnety i zajęcia) · warsztat samochodowy (zlecenia) · pizzeria
(menu i zamówienia) · przychodnia (lekarze i terminy) · szkoła nauki jazdy (kursanci
i jazdy) · katalog gier planszowych · giełda ogłoszeń · schronisko dla zwierząt ·
serwis rowerowy · planer treningów · katalog roślin doniczkowych · wypożyczalnia
instrumentów. Możesz zaproponować własną – musi mieć **listę obiektów z kilkoma polami**
i sensowne kategorie.

---

**Zadanie 1 – projekt**
Postaw środowisko wirtualne, zainstaluj Django, utwórz projekt i aplikację. Wygeneruj
`requirements.txt`. Doprowadź do działającej strony powitalnej. Commit.

**Zadanie 2 – health**
Wystaw `/api/health/` zwracające `{"status": "ok"}`. To ma zadziałać, zanim napiszesz
cokolwiek innego – jeśli działa, znaczy że cała ścieżka od adresu do widoku jest poprawna.

**Zadanie 3 – dane**
Plik `data.py` z **minimum 8 rekordami**. Każdy rekord ma mieć co najmniej 5 pól, w tym:
unikalne `id`, pole tekstowe, pole liczbowe i pole kategoryzujące. Wszystkie rekordy
z identycznym zestawem kluczy. Co najmniej jeden rekord z wartością brzegową (zero,
pusty tekst, wartość skrajna) – przyda się przy testach.

**Zadanie 4 – lista**
Endpoint zwracający wszystkie rekordy. Nazwy zasobów, pól i funkcji **po angielsku**,
w liczbie mnogiej dla list: `/api/books/`, nie `/api/ksiazka/`. Nazwa aplikacji Django
też po angielsku (`books`, `gym`, `cinema`).

**Zadanie 5 – szczegóły**
Endpoint `/api/<zasoby>/<int:id>/`. Dla nieistniejącego identyfikatora: status **404**
i czytelny komunikat błędu w JSON-ie. Sprawdź w Thunder Client, że status naprawdę
wynosi 404, a nie 200.

**Zadanie 6 – filtrowanie**
Filtrowanie po co najmniej jednym polu przez parametr w adresie. Ambitniej: dwa parametry
działające jednocześnie (np. `?category=fantasy&available=true`).

**Zadanie 7 – info**
Endpoint `/api/info/` z: nazwą aplikacji, jej wersją, wersją Pythona, wersją Django,
typem źródła danych, liczbą rekordów, listą dostępnych kategorii i czasem działania.

**Zadanie 8 – statystyki**
Endpoint `/api/statystyki/` liczący coś sensownego dla Twojej domeny: liczbę rekordów
w każdej kategorii, wartość średnią, minimum i maksimum pola liczbowego. Wszystko
policzone w Pythonie, żadnych wartości wpisanych na sztywno.

**Zadanie 9 – dokumentacja**
W `README.md` swojego repozytorium tabela wszystkich endpointów: metoda, adres, opis,
przykładowa odpowiedź. Plus instrukcja uruchomienia backendu krok po kroku – tak, żeby
zadziałała u kogoś, kto Twojego projektu nigdy nie widział.

**Zadanie 10 – historia pracy**
Cała powyższa praca ma być rozbita na **minimum 6 commitów** z sensownymi komunikatami
według konwencji z materiału 01. Jeden commit „zrobione" nie zostanie przyjęty.

---

**Zadanie dodatkowe (dla chętnych)**
Nieistniejący adres, na przykład `/api/cokolwiek/`, zwraca teraz stronę HTML z debugowania
Django – a przecież to jest API i klient spodziewa się JSON-a. Znajdź w dokumentacji, czym
jest `handler404`, i spraw, żeby dla nieznanego adresu też wracał JSON. Uwaga: przy
`DEBUG = True` Django pokazuje własną stronę błędu, więc do testu trzeba ustawić
`DEBUG = False` i uzupełnić `ALLOWED_HOSTS`. Opisz w `docs/handler404.md`, co zrobiłeś
i dlaczego to nie działało od razu.