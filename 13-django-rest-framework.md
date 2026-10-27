# Django REST Framework

**Technik programista | klasa 5 | materiał 13**

---

## Policz linie

Otwórz `backend/shop/views.py`. Masz tam:

- funkcję pomocniczą `json_response` z trzema argumentami,
- funkcję `product_to_dict` wypisującą siedem pól ręcznie,
- cztery widoki z pętlami, warunkami i obsługą 404.

Razem około siedemdziesięciu linii. Dla **czterech endpointów, tylko do czytania**.

Dziś zastąpimy to piętnastoma liniami. Ale nie o oszczędność chodzi – chodzi o coś,
co jest warte więcej:

> **Zrozumiesz te piętnaście linii tylko dlatego, że wcześniej napisałeś siedemdziesiąt.**

Gdybyśmy zaczęli od Django REST Framework w materiale 03, dostałbyś działające API
i zero pojęcia, skąd się bierze. Serializer byłby magią. Teraz wiesz dokładnie, co robi,
bo robiłeś to ręcznie: zamieniał obiekt na słownik, pilnował, żeby cena nie była liczbą
zmiennoprzecinkową, spłaszczał kategorię do nazwy.

To jest sens całego porządku tych materiałów. Każde narzędzie, które poznajesz, zastępuje
pracę, którą wcześniej wykonałeś własnymi rękami.

### Co dochodzi

| | Przed | Po |
|---|---|---|
| Serializacja | `product_to_dict` ręcznie | `ModelSerializer` z modelu |
| Widoki | cztery funkcje | jedna klasa `ModelViewSet` |
| Adresy | cztery wpisy `path` | jeden `router.register` |
| Metody | tylko GET | GET, POST, PUT, PATCH, DELETE |
| Dokumentacja API | plik `docs/api.md` | przeglądalna strona, generowana sama |
| Walidacja | brak | z definicji pól modelu |

### Wersje

| | Wersja |
|---|---|
| Django | 6.1.1 |
| Django REST Framework | 3.18.1 |

DRF 3.18 to pierwsze wydanie z oficjalnym wsparciem Django 6.1.

---

## Krok 1 – instalacja

```bash
cd backend
venv\Scripts\activate
pip install djangorestframework
pip freeze > requirements.txt
```

W `core/settings.py`, w `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    "rest_framework",
    "corsheaders",
    "shop",
]
```

Na razie bez żadnej dodatkowej konfiguracji. DRF ma rozsądne wartości domyślne
i dojdziemy do nich wtedy, gdy będą potrzebne.

---

## Krok 2 – serializer, czyli serce DRF

**Serializer tłumaczy w dwie strony**:

```
obiekt Pythona  --- serializacja --->  JSON       (odczyt, GET)
obiekt Pythona  <-- deserializacja ---  JSON      (zapis, POST)
                         +
                    walidacja
```

Twoja funkcja `product_to_dict` robiła tylko połowę tej pracy – pierwszą strzałkę.
Drugiej nie było, bo nie przyjmowaliśmy danych.

Nowy plik **`shop/serializers.py`**:

```python
"""Serializery API sklepu – tłumaczenie między modelami a formatem JSON."""

from rest_framework import serializers

from .models import Category, Product


class CategorySerializer(serializers.ModelSerializer):
    """Kategoria produktów."""

    class Meta:
        model = Category
        fields = ["id", "name"]


class ProductSerializer(serializers.ModelSerializer):
    """Produkt w katalogu.

    Kategoria jest reprezentowana nazwą, nie identyfikatorem – dzięki temu
    format odpowiedzi jest taki sam jak przed wprowadzeniem DRF i klienci
    (React, Android) nie wymagają żadnej zmiany.
    """

    category = serializers.SlugRelatedField(
        slug_field="name",
        queryset=Category.objects.all(),
    )

    class Meta:
        model = Product
        fields = [
            "id",
            "name",
            "description",
            "price",
            "stock",
            "category",
            "created_at",
        ]
        read_only_fields = ["id", "created_at"]
```

### Co tu się właściwie stało

`ModelSerializer` **czyta model i sam buduje pola**. Nie wypisujesz typów ani walidacji –
bierze je z definicji modelu. `max_length=200` przy nazwie staje się regułą walidacji.
`PositiveIntegerField` przy stanie magazynowym sprawia, że wartość ujemna zostanie
odrzucona.

To jest ta sama idea, co z migracjami: **model jest jedynym źródłem prawdy**, a reszta
się z niego wyprowadza.

Sprawdź w konsoli:

```bash
python manage.py shell
```

```python
from shop.models import Product
from shop.serializers import ProductSerializer

product = Product.objects.first()
serializer = ProductSerializer(product)

print(serializer.data)
```

Dostajesz słownik – dokładnie to, co zwracał `product_to_dict`. Zobacz też, jak serializer
opisuje sam siebie:

```python
print(ProductSerializer())
```

Wypisze listę pól z typami i regułami. Tego `product_to_dict` nie potrafiło: **serializer
wie, czym jest każde pole**, a funkcja tylko przepisywała wartości.

---

## Krok 3 – kontrakt, którego DRF prawie nie zepsuł

To najważniejszy fragment tego materiału. Przeczytaj go uważnie.

W materiale 04 ustaliliśmy format odpowiedzi i od tamtej pory React i Android na nim
stoją. Zmiana choćby jednej nazwy pola psuje oba klienty. Sprawdźmy więc, co DRF robi
z trzema szczegółami, o które wtedy walczyliśmy.

### Cena jako tekst – zgodne

```json
"price": "349.00"
```

DRF serializuje `DecimalField` **do tekstu**, nie do liczby zmiennoprzecinkowej.
Z tego samego powodu, dla którego my robiliśmy `str(product.price)`: konwersja na `float`
psuje dokładność.

Miło, prawda? Twórcy DRF podjęli tę samą decyzję, co Ty w materiale 04 – bo problem jest
ten sam.

### Data w ISO 8601 – zgodne

```json
"created_at": "2026-09-01T09:12:44.812000+00:00"
```

DRF formatuje `DateTimeField` zgodnie z ISO 8601, czyli tak jak nasze `.isoformat()`.

### Kategoria – **tu by się zepsuło**

Gdybyś nie dopisał tej jednej linii:

```python
category = serializers.SlugRelatedField(slug_field="name", queryset=Category.objects.all())
```

domyślny `ModelSerializer` zwróciłby dla klucza obcego **identyfikator**:

```json
"category": 3
```

A React wyświetla `{product.category}`, więc pokazałby „3". Android wpisałby „3"
w `TextView`. Nic by nie wybuchło, nic nie rzuciłoby błędem – po prostu zamiast nazwy
kategorii byłaby liczba.

**To jest najbardziej podstępny rodzaj awarii** i spotkałeś go już dwa razy: w materiale 07
jako `undefined` w Reakcie i w materiale 09 jako puste pole przy niezgodności nazw dla
Gsona. Za każdym razem mechanizm jest ten sam – zmienił się kontrakt, a nikt nie dostał
komunikatu o błędzie.

Dlatego `SlugRelatedField`, a nie domyślne zachowanie. Dwie zalety tego wyboru:

- **działa w obie strony** – przy zapisie w materiale 14 będziesz mógł przysłać
  `"category": "audio"` i DRF znajdzie właściwy wiersz;
- **zachowuje format** – klienci nic nie zauważą.

Jest też `StringRelatedField`, krótszy w zapisie, ale **tylko do odczytu** – nie
przyjąłby POST-a. Dlatego od razu wybieramy wersję, która nie będzie wymagała przepisania
za tydzień.

### Sprawdź polskie znaki

Po uruchomieniu API zajrzyj do odpowiedzi i upewnij się, że widzisz `"Słuchawki"`,
a nie `"Słuchawki"`. DRF domyślnie **nie ucieka** znaków poza ASCII – więc
`ensure_ascii=False`, o które walczyliśmy w materiale 03, dostajesz tutaj za darmo.

### `fields` wypisane, nie `__all__`

Mogłeś napisać `fields = "__all__"` i oszczędzić siedem linii. **Nie rób tego.**

Powód jest konkretny: za miesiąc dodasz do modelu pole `koszt_zakupu` albo `marza`
i ono **samo pojawi się w publicznym API**, bo `__all__` oznacza „wszystko, co jest
w modelu teraz i w przyszłości". Jawna lista jest świadomą decyzją o tym, co wychodzi
na świat.

To nie jest teoretyczne zmartwienie – tak wyciekają dane.

---

## Krok 4 – widoki: drabinka od funkcji do klasy

DRF daje kilka poziomów gotowości. Warto je zobaczyć po kolei, bo wtedy `ModelViewSet`
przestaje być magią.

### Poziom 1: `@api_view` – najbliżej tego, co miałeś

```python
from rest_framework.decorators import api_view
from rest_framework.response import Response


@api_view(["GET"])
def product_list(request):
    products = Product.objects.select_related("category")
    serializer = ProductSerializer(products, many=True)
    return Response(serializer.data)
```

Zwróć uwagę na trzy rzeczy:

**`Response` zamiast `JsonResponse`.** Przyjmuje zwykłe struktury Pythona i sam decyduje
o formacie wyjściowym. Twoja funkcja pomocnicza `json_response` przestaje być potrzebna.

**`many=True`** – bo serializujemy wiele obiektów, nie jeden. Bez tego DRF rzuci błędem.

**Dekorator wymienia dozwolone metody.** Żądanie POST pod ten adres dostanie automatycznie
odpowiedź `405 Method Not Allowed`. Wcześniej musiałbyś to sprawdzać sam.

### Poziom 2: widoki generyczne

```python
from rest_framework import generics


class ProductList(generics.ListAPIView):
    queryset = Product.objects.select_related("category")
    serializer_class = ProductSerializer
```

Dwie linie zamiast pięciu. Nie piszesz **jak** pobrać i zserializować – mówisz tylko
**co**. Klasa bazowa zna resztę.

> **To jest wzorzec Metoda szablonowa** (*Template Method*), jeden z tych, które wymienia
> podstawa programowa INF.04. Klasa bazowa definiuje szkielet algorytmu – pobierz dane,
> zserializuj, zwróć odpowiedź – a Ty podstawiasz tylko brakujące elementy przez
> `queryset` i `serializer_class`.
>
> Omawialiśmy wzorce na sztucznych przykładach. To jest prawdziwy, w bibliotece, z której
> korzystasz.

Rodzina widoków generycznych jest większa: `RetrieveAPIView` (jeden obiekt),
`CreateAPIView` (zapis), `ListCreateAPIView`, `RetrieveUpdateDestroyAPIView`.

### Poziom 3: `ModelViewSet` – wszystko naraz

```python
class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.select_related("category")
    serializer_class = ProductSerializer
```

Te trzy linie dają **sześć operacji**: listę, pojedynczy obiekt, utworzenie, pełną
aktualizację, częściową aktualizację i usunięcie.

Im wyżej na drabince, tym mniej kodu i tym mniej kontroli. Dlatego warto znać wszystkie
trzy poziomy: gdy standardowe zachowanie nie wystarcza, zjeżdżasz niżej – do konkretnej
metody albo do `@api_view`.

---

## Krok 5 – nowe `views.py`

```python
"""Widoki API sklepu.

Produkty i kategorie obsługiwane są przez ViewSety DRF.
Endpointy techniczne (health, info) to zwykłe funkcje – nie dotyczą modelu,
więc ViewSet byłby tu nadużyciem.
"""

import sys
import time

import django
import rest_framework

from django.conf import settings
from django.db import connection

from rest_framework import status, viewsets
from rest_framework.decorators import api_view
from rest_framework.response import Response

from .models import Category, Product
from .serializers import CategorySerializer, ProductSerializer

APP_VERSION = "0.4.0"
START_TIME = time.time()


class CategoryViewSet(viewsets.ModelViewSet):
    """Kategorie produktów."""

    queryset = Category.objects.all()
    serializer_class = CategorySerializer


class ProductViewSet(viewsets.ModelViewSet):
    """Produkty w katalogu.

    Obsługuje filtrowanie parametrem `category`, na przykład:
    /api/products/?category=audio
    """

    serializer_class = ProductSerializer

    def get_queryset(self):
        queryset = Product.objects.select_related("category")

        category = self.request.query_params.get("category")
        if category and category != "all":
            queryset = queryset.filter(category__name=category)

        return queryset


@api_view(["GET"])
def health(request):
    """Zwraca stan usługi i połączenia z bazą danych."""
    try:
        connection.ensure_connection()
    except Exception:
        return Response(
            {"status": "error", "database": False},
            status=status.HTTP_503_SERVICE_UNAVAILABLE,
        )

    return Response({"status": "ok", "database": True})


@api_view(["GET"])
def info(request):
    """Zwraca wersje, informacje o źródle danych i czas działania."""
    return Response({
        "application": "Shop API",
        "app_version": APP_VERSION,
        "python_version": sys.version.split()[0],
        "django_version": django.get_version(),
        "drf_version": rest_framework.VERSION,
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
```

### Trzy decyzje warte omówienia

**`get_queryset` zamiast `queryset`.** Pole `queryset` jest wyliczane **raz**, przy
starcie serwera. Metoda `get_queryset` wykonuje się **przy każdym żądaniu** – i tylko
dlatego ma dostęp do `self.request`, czyli do parametrów w adresie. Filtrowanie wymaga
metody.

Zauważ też, że `select_related("category")` zostało. DRF nie rozwiązuje problemu N+1 –
serializer sięga do `product.category.name` dokładnie tak samo jak nasz kod
z materiału 04, więc bez `select_related` zapytań znów byłoby 501 zamiast jednego.

**`status.HTTP_503_SERVICE_UNAVAILABLE` zamiast `503`.** Działa identycznie, czyta się
o wiele lepiej. Pełna lista stałych jest w `rest_framework.status`.

**Health i info jako funkcje.** Nie dotyczą żadnego modelu, nie mają listy ani pojedynczego
obiektu – `ViewSet` nie ma tu czego obsługiwać. Dobre narzędzie użyte nie tam, gdzie trzeba,
tworzy więcej pracy niż oszczędza.

Numer wersji skacze na `0.4.0` – zmiana mniejsza, bo doszły nowe możliwości, a stare
działają dalej. Zgodnie z materiałem 01.

---

## Krok 6 – router

**`shop/urls.py`** w całości:

```python
"""Adresy API sklepu.

Endpointy produktów i kategorii generuje router DRF na podstawie ViewSetów.
"""

from django.urls import include, path
from rest_framework.routers import DefaultRouter

from . import views

router = DefaultRouter()
router.register("products", views.ProductViewSet, basename="product")
router.register("categories", views.CategoryViewSet, basename="category")

urlpatterns = [
    path("health/", views.health),
    path("info/", views.info),
    path("", include(router.urls)),
]
```

Jeden `register` tworzy komplet adresów:

| Metoda | Adres | Operacja |
|---|---|---|
| GET | `/api/products/` | lista |
| POST | `/api/products/` | utworzenie |
| GET | `/api/products/<id>/` | pojedynczy |
| PUT | `/api/products/<id>/` | pełna aktualizacja |
| PATCH | `/api/products/<id>/` | częściowa aktualizacja |
| DELETE | `/api/products/<id>/` | usunięcie |

Plus adres `/api/` z listą dostępnych zasobów.

**`basename` jest wymagany**, gdy `ViewSet` nie ma pola `queryset`, a tylko metodę
`get_queryset` – router nie potrafi wtedy zgadnąć nazwy. Komunikat błędu bez tego jest
zaskakująco nieczytelny, więc podajemy od razu.

Zwróć uwagę, że `health/` i `info/` stoją **przed** `include(router.urls)`. Router dokłada
między innymi wzorzec pasujący do pustej ścieżki, więc kolejność ma znaczenie – tak jak
omawialiśmy przy `urlpatterns` w materiale 03.

---

## Krok 7 – nagroda: przeglądalne API

Uruchom serwer i wejdź **przeglądarką** na `http://localhost:8000/api/`.

To nie jest surowy JSON. To **strona**, która:

- wypisuje dostępne zasoby jako odsyłacze,
- przy każdym endpoincie pokazuje dozwolone metody,
- wyświetla docstring Twojej klasy jako opis,
- ma formularz do wysłania POST-a,
- przy pojedynczym produkcie daje przyciski PUT, PATCH i DELETE.

Przejdź na `http://localhost:8000/api/products/` i spójrz na dół strony. Jest tam
formularz. Wypełnij go i kliknij POST.

**Produkt został dodany.** Bez linii kodu po Twojej stronie, bez formularza w Reakcie,
bez Postmana.

### Dlaczego to jest ważne

W materiale 10 pisałem, że docstringi „zaczną pracować same". Właśnie to się stało:
opis klasy `ProductViewSet`, który napisałeś przy dokumentacji kodu, jest teraz
dokumentacją Twojego API, widoczną w przeglądarce.

To jest **dokumentacja, która nie może się rozjechać z kodem**, bo jest z niego
generowana. Twój `docs/api.md` z materiału 11 trzeba aktualizować ręcznie i kiedyś
się rozjedzie. Ta strona nie.

### Czego nie zrobiłeś, a co masz

Spróbuj w przeglądalnym API wysłać POST z pustą nazwą. Albo z ceną `abc`. Albo
z kategorią, która nie istnieje.

Dostaniesz `400 Bad Request` i czytelny opis błędu per pole:

```json
{
  "name": ["To pole nie może być puste."],
  "price": ["Wymagana poprawna liczba."],
  "category": ["Obiekt o podanej wartości name=kosmos nie istnieje."]
}
```

Tej walidacji nie napisałeś. Wzięła się z definicji pól modelu – `max_length`,
`PositiveIntegerField`, `ForeignKey`. **Dlatego warto było poświęcić czas na dobre
typy w materiale 04.**

W materiale 14 dopiszemy do tego reguły własne, których z modelu nie da się wyczytać.

---

## Krok 8 – sprawdzenie, czy klienci nic nie zauważyli

Teraz najważniejszy test tego materiału.

**Nie dotykaj Reacta. Nie dotykaj Androida.** Uruchom je i sprawdź, czy działają.

```bash
# terminal 2
cd frontend && npm run dev
```

Lista produktów ma wyglądać dokładnie tak samo. Filtrowanie po kategorii ma działać.
Szczegóły produktu też. To samo w emulatorze.

Jeśli wszystko działa – **przepisałeś całą warstwę API i nie zepsułeś żadnego klienta**.
Kontrakt został dotrzymany.

Jeśli React pokazuje liczby w miejscu kategorii, zajrzyj do kroku 3: brakuje
`SlugRelatedField`.

---

## Krok 9 – stronicowanie i dlaczego go nie włączamy

DRF potrafi dzielić wyniki na strony. Wystarczy wpisać w `settings.py`:

```python
REST_FRAMEWORK = {
    "DEFAULT_PAGINATION_CLASS": "rest_framework.pagination.PageNumberPagination",
    "PAGE_SIZE": 10,
}
```

**Nie wpisuj tego teraz.** Spójrz najpierw, co to robi z odpowiedzią:

```json
{
  "count": 12,
  "next": "http://localhost:8000/api/products/?page=2",
  "previous": null,
  "results": [ ... ]
}
```

Lista przestała być tablicą. Jest teraz **obiektem**, a produkty siedzą w polu `results`.

Co się stanie u klientów:

- React wywoła `products.filter(...)` na obiekcie i wywali się na `filter is not
  a function`;
- Android oczekuje `List<Product>`, a dostanie obiekt – Gson zwróci `null` albo rzuci
  wyjątkiem.

**Oba klienty przestaną działać.** I to nie jest błąd DRF – to jest **zmiana główna
kontraktu**, dokładnie taka, o jakiej mówiliśmy przy wersjonowaniu semantycznym
w materiale 01. Zmiana kształtu odpowiedzi psuje zgodność, więc wymagałaby wersji `1.0.0`
i poprawek we wszystkich klientach.

Dlatego zostawiamy to jako **świadomą, zapisaną decyzję**: stronicowanie włączymy wtedy,
gdy będziemy gotowi poprawić Reacta i Androida. Przy dwunastu produktach nie ma po co.

Dopisz to do `docs/api.md` jako znaną decyzję projektową. Dokumentowanie rzeczy, których
się **nie** zrobiło i dlaczego, jest oznaką porządnej inżynierii – a nie przyznaniem się
do zaniedbania.

---

## Krok 10 – co zniknęło, co zostało

Usuń z `views.py` to, co stało się zbędne:

```python
# do usunięcia
def json_response(data, status=200): ...
def product_to_dict(product): ...
```

Nie usuwaj ich jednak z pamięci. Porównaj:

```python
# materiał 04 – ręcznie
def product_to_dict(product):
    return {
        "id": product.id,
        "name": product.name,
        "description": product.description,
        "price": str(product.price),
        "stock": product.stock,
        "category": product.category.name,
        "created_at": product.created_at.isoformat(),
    }

# materiał 13 – z modelu
class ProductSerializer(serializers.ModelSerializer):
    category = serializers.SlugRelatedField(
        slug_field="name", queryset=Category.objects.all()
    )

    class Meta:
        model = Product
        fields = ["id", "name", "description", "price", "stock", "category", "created_at"]
```

Dwie wersje tej samej rzeczy. Druga jest krótsza, obsługuje zapis i waliduje dane –
ale **czytasz ją ze zrozumieniem tylko dlatego, że napisałeś pierwszą**.

### Co DRF zrobił za Ciebie

Serializację i deserializację, walidację z definicji pól, kody statusu, obsługę
nieistniejącego zasobu, odrzucanie niedozwolonych metod, generowanie adresów,
przeglądalną dokumentację.

### Czego DRF **nie** zrobił

- **Nie zoptymalizował zapytań.** `select_related` jest nadal Twoje. Problem N+1 istnieje.
- **Nie zabezpieczył API.** Wszystko jest publiczne – każdy może teraz wysłać DELETE.
  Tym zajmiemy się przy logowaniu.
- **Nie wymyślił reguł biznesowych.** „Cena nie może przekraczać dziesięciokrotności
  ceny zakupu" to wiedza, której w modelu nie ma. Materiał 14.
- **Nie zadbał o kontrakt.** Kategoria wyszłaby jako liczba, gdybyś nie dopisał
  jednej linii. Narzędzie nie wie, co obiecałeś klientom.

To ostatnie jest najważniejsze. **Framework zdejmuje z Ciebie pisanie, nie myślenie.**

---

## Sprawdź, czy rozumiesz

1. Co serializer robi w dwie strony, czego `product_to_dict` nie robiło wcale?
2. Skąd `ModelSerializer` bierze reguły walidacji?
3. Dlaczego `category` wymagała osobnej linii, a `price` i `created_at` nie?
4. Czym `SlugRelatedField` różni się od `StringRelatedField` i dlaczego wybraliśmy
   pierwsze?
5. Dlaczego `fields = "__all__"` jest ryzykowne?
6. Kiedy użyjesz `get_queryset`, a kiedy wystarczy pole `queryset`?
7. Które sześć operacji daje jeden `router.register` i dlaczego potrzebny był `basename`?
8. Dlaczego `select_related` nie zniknęło razem z ręczną serializacją?
9. Co stronicowanie robi z kształtem odpowiedzi i którzy klienci na tym ucierpią?
10. Dlaczego `health` i `info` zostały funkcjami, a nie `ViewSet`-ami?

---

## Częste błędy

| Objaw | Przyczyna |
|---|---|
| `category` w odpowiedzi to liczba | brak `SlugRelatedField` w serializerze |
| `basename argument not specified` | `ViewSet` bez pola `queryset`; dopisz `basename` |
| `Object of type ... is not JSON serializable` | zwracasz obiekt modelu zamiast `serializer.data` |
| `AttributeError: 'QuerySet' object has no attribute 'pk'` | brak `many=True` przy wielu obiektach |
| przeglądalne API się nie pokazuje | wchodzisz narzędziem innym niż przeglądarka albo brak `rest_framework` w `INSTALLED_APPS` |
| nowe pole modelu samo pojawiło się w API | `fields = "__all__"` |
| React pokazuje pustą listę po włączeniu stronicowania | kształt odpowiedzi się zmienił – patrz krok 9 |
| 500 zamiast 400 przy błędnych danych | ominięcie serializera i ręczne tworzenie obiektu |
| wolne odpowiedzi przy wielu produktach | brak `select_related` – problem N+1 |
| `405 Method Not Allowed` | metoda nie jest na liście w `@api_view` |

---

## Czego jeszcze nie ma

- **Walidacji własnej.** Reguły, których nie da się wyczytać z modelu. Materiał 14.
- **Formularzy u klientów.** Zapis działa tylko z przeglądalnego API i z panelu
  administratora. Materiały 16 i 17.
- **Uwierzytelniania.** API przyjmuje DELETE od każdego. To teraz poważniejszy problem
  niż był, bo wcześniej dało się tylko czytać.
- **Testów.** Materiał 15.
- **Stronicowania.** Świadomie odłożone.

---

## Co dalej

**Materiał 14:** zapis na poważnie. POST, PUT, PATCH i DELETE z walidacją własną,
czytelnymi komunikatami błędów i regułami, których w modelu nie ma. Plus odpowiedź
na pytanie, które powinno Cię teraz niepokoić: **dlaczego ktokolwiek może usunąć
dowolny produkt?**

---

# Zadania

Pracujesz na swoim projekcie i swojej domenie.

**Zadanie 1 – instalacja**
DRF zainstalowany i dopisany do `INSTALLED_APPS`. `requirements.txt` zaktualizowany.

**Zadanie 2 – serializery**
Serializery dla obu Twoich modeli, z **jawną listą pól** i `read_only_fields`.
W `docs/notatki/drf.md` wyjaśnij jednym akapitem, dlaczego nie użyłeś `__all__`.

**Zadanie 3 – kontrakt**
Zanim podmienisz widoki: zapisz w `docs/notatki/drf.md` odpowiedź Twojego API z materiału
04. Po podmianie zapisz nową. **Porównaj je pole po polu** i wypisz każdą różnicę.
Różnic ma nie być – jeśli są, popraw serializer, nie klienta.

**Zadanie 4 – trzy poziomy widoków**
Zaimplementuj listę swoich obiektów **trzy razy**: jako `@api_view`, jako widok
generyczny i jako `ModelViewSet`. Zostaw w kodzie tylko ostatnią wersję, ale wszystkie
trzy wklej do `docs/notatki/drf.md` i policz linie każdej.

**Zadanie 5 – ViewSet z filtrowaniem**
`ModelViewSet` dla modelu głównego, z `get_queryset` obsługującym filtrowanie po tym
samym parametrze co w materiale 03. Zachowaj `select_related`.

**Zadanie 6 – router**
Endpointy produktów i kategorii generowane routerem. Endpointy techniczne zostają
funkcjami. W `docs/notatki/drf.md` wypisz tabelę wszystkich adresów, które wygenerował
router.

**Zadanie 7 – test klientów**
Uruchom frontend i aplikację mobilną **bez żadnych zmian w ich kodzie**. Zrzuty ekranu
obu, działających na nowym backendzie. To jest dowód dotrzymania kontraktu.

**Zadanie 8 – przeglądalne API**
Zrzut ekranu strony `/api/products/` z widocznym formularzem. Dodaj produkt przez ten
formularz i sprawdź, że pojawił się w Reakcie po odświeżeniu.

**Zadanie 9 – walidacja z modelu**
Wyślij przez przeglądalne API trzy błędne żądania POST: puste wymagane pole, zła wartość
liczbowa, nieistniejąca kategoria. Wklej odpowiedzi do `docs/notatki/drf.md` i przy
każdej napisz, **z której części definicji modelu** wzięła się ta reguła.

**Zadanie 10 – stronicowanie jako eksperyment**
Włącz stronicowanie, uruchom Reacta i **zobacz, jak się psuje**. Zrzut ekranu błędu
w konsoli przeglądarki. Potem wyłącz je i dopisz do `docs/api.md` notkę o tej decyzji
wraz z uzasadnieniem. W `docs/notatki/drf.md` odpowiedz: jaki numer wersji wymagałaby
ta zmiana i dlaczego?

**Zadanie 11 – info i dokumentacja**
`/api/info/` zwraca wersję DRF i podniesiony numer wersji aplikacji. `CHANGELOG.md`
uzupełniony. `docs/api.md` zaktualizowany o nowe metody.

**Zadanie 12 – porządki i historia**
Usuń `json_response` i funkcję serializującą ręcznie. Minimum pięć commitów z sensownymi
komunikatami, w tym jeden `refactor`.

---

**Zadanie dodatkowe (dla chętnych)**
Zainstaluj `drf-spectacular` i wygeneruj dokumentację API w standardzie **OpenAPI**:

```bash
pip install drf-spectacular
```

Po konfiguracji dostaniesz pod `/api/schema/swagger-ui/` interaktywną dokumentację –
listę endpointów, schematy danych i możliwość wysyłania żądań z przeglądarki. To jest
narzędzie, którym dokumentuje się API w prawdziwych firmach.

W `docs/notatki/drf.md` porównaj je z plikiem `docs/api.md`, który napisałeś ręcznie
w materiale 11. Odpowiedz na dwa pytania: **co OpenAPI wie lepiej od Ciebie** i **czego
nigdy się nie domyśli** – a potem zdecyduj, czy po tym porównaniu `docs/api.md` jest
jeszcze potrzebny, czy wystarczy w nim zostawić to drugie.
