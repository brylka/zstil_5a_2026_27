# CORS i konfiguracja połączenia

**Technik programista | klasa 5 | materiał 06**

---

## O co chodzi

Masz działające API pod `localhost:8000` i działający frontend pod `localhost:5173`.
Wydawałoby się, że wystarczy jedno `fetch` i po sprawie.

Nie wystarczy. Przeglądarka to zablokuje.

Ten materiał jest krótki i prawie w całości backendowy, ale bez niego materiał 07 nie ma
prawa zadziałać. Przy okazji uporządkujemy konfigurację obu części projektu – wyprowadzimy
adresy i klucze do plików `.env`, żeby nie były wpisane na sztywno w kodzie.

---

## Krok 1 – zobacz błąd na własne oczy

Uruchom **oba** serwery, każdy w swoim terminalu:

```bash
# terminal 1
cd backend
venv\Scripts\activate
python manage.py runserver
```

```bash
# terminal 2
cd frontend
npm run dev
```

Wejdź na **http://localhost:5173**, otwórz narzędzia deweloperskie (F12), zakładka
**Console**, i wklej:

```js
fetch("http://localhost:8000/api/products/")
  .then((response) => response.json())
  .then((data) => console.log(data));
```

W konsoli pojawi się coś w tym stylu:

```
Access to fetch at 'http://localhost:8000/api/products/' from origin
'http://localhost:5173' has been blocked by CORS policy: No 'Access-Control-Allow-Origin'
header is present on the requested resource.
```

Teraz przejdź do zakładki **Network** i kliknij to żądanie. Zwróć uwagę na coś, co
wygląda absurdalnie: **serwer odpowiedział poprawnie, ze statusem 200**. Dane przyszły.
Przeglądarka po prostu odmówiła podania ich Twojemu skryptowi.

To jest sedno całej sprawy i najczęstsze nieporozumienie.

---

## Krok 2 – dlaczego przeglądarka to robi

### Same-Origin Policy

Przeglądarki od zawsze pilnują zasady: **skrypt ze strony A nie może swobodnie czytać
danych z serwera B**. Nazywa się to *Same-Origin Policy* i jest jednym z filarów
bezpieczeństwa internetu.

Wyobraź sobie, że jej nie ma. Otwierasz w jednej karcie bank, w drugiej losową stronę
ze śmiesznymi obrazkami. Skrypt na tej drugiej stronie wysyła `fetch` na adres banku –
przeglądarka dołącza Twoje ciasteczka sesyjne, bo jesteś zalogowany – i odczytuje stan
Twojego konta. Bez Same-Origin Policy byłoby to trywialne.

### Czym jest „origin"

Origin to **trzy rzeczy naraz**: protokół, host i port. Wystarczy, że jedna się różni,
i mamy inne pochodzenie.

| Adres | Ten sam origin co `http://localhost:5173`? |
|---|---|
| `http://localhost:5173/produkty` | tak – ścieżka nie ma znaczenia |
| `http://localhost:8000` | **nie** – inny port |
| `https://localhost:5173` | **nie** – inny protokół |
| `http://127.0.0.1:5173` | **nie** – inny host, mimo że ta sama maszyna |

Ostatni wiersz zaskakuje najczęściej. Dla przeglądarki `localhost` i `127.0.0.1` to dwa
różne pochodzenia, chociaż wskazują na ten sam komputer. Zapamiętaj to – zaraz wróci.

### Dlaczego `curl` i Thunder Client działają bez problemu

Bo **to nie serwer blokuje żądanie, tylko przeglądarka**. `curl` nie ma pojęcia
o Same-Origin Policy i nigdy nie miał. Nie chroni żadnego zalogowanego użytkownika,
bo nie ma kart, ciasteczek ani sesji.

Stąd bierze się klasyczna scena: „w Postmanie działa, w aplikacji nie działa". Zawsze
znaczy to jedno – CORS.

---

## Krok 3 – jak działa CORS

**CORS** (*Cross-Origin Resource Sharing*) to sposób, w jaki serwer może powiedzieć
przeglądarce: „spokojnie, ten konkretny origin ma prawo czytać moje odpowiedzi".

Robi to **nagłówkiem odpowiedzi**:

```
Access-Control-Allow-Origin: http://localhost:5173
```

Przeglądarka widzi ten nagłówek, porównuje z pochodzeniem strony, zgadza się i podaje dane
skryptowi. Bez nagłówka – blokuje, mimo że odpowiedź fizycznie już dotarła.

```
React (5173)   ---                     GET /api/products/                   --->  Django (8000)
              <--- 200 + Access-Control-Allow-Origin: http://localhost:5173 ---
                        przeglądarka sprawdza nagłówek i przepuszcza dane
```

Kluczowe: **to serwer decyduje, kto może go czytać.** Nie da się obejść CORS-u po stronie
frontendu, bo cała konstrukcja polega właśnie na tym, że frontend nie ma tu nic do
gadania.

### Żądania wstępne (preflight)

Przy prostych żądaniach – GET, HEAD i POST bez nietypowych nagłówków – przeglądarka
wysyła zapytanie od razu i dopiero potem sprawdza nagłówek odpowiedzi.

Przy pozostałych – PUT, DELETE, POST z JSON-em, cokolwiek z własnym nagłówkiem
`Authorization` – wysyła najpierw **żądanie wstępne** metodą `OPTIONS`:

```
OPTIONS /api/products/
Origin: http://localhost:5173
Access-Control-Request-Method: POST
```

i dopiero po pozytywnej odpowiedzi wysyła to właściwe. To jest zabezpieczenie przed
wykonaniem operacji zmieniającej dane, zanim ktokolwiek sprawdzi uprawnienia.

Dziś robimy tylko GET, więc żądań wstępnych nie zobaczysz. Pojawią się przy logowaniu
JWT i przy formularzach – i wtedy przypomnisz sobie ten akapit, bo w zakładce Network
zaczną się pojawiać tajemnicze wpisy `OPTIONS` obok każdego zapytania.

---

## Krok 4 – django-cors-headers

Django nie obsługuje CORS-u samo z siebie. Standardem jest paczka `django-cors-headers`
(wspiera Django od 5.2 do 6.1 i Pythona od 3.10 do 3.15, więc nasz zestaw jest w sam raz).

```bash
cd backend
venv\Scripts\activate
pip install django-cors-headers
pip freeze > requirements.txt
```

W `core/settings.py` trzy zmiany.

**Pierwsza – rejestracja aplikacji:**

```python
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    "corsheaders",
    "shop",
]
```

**Druga – warstwa pośrednia:**

```python
MIDDLEWARE = [
    "corsheaders.middleware.CorsMiddleware",
    "django.middleware.security.SecurityMiddleware",
    "django.contrib.sessions.middleware.SessionMiddleware",
    "django.middleware.common.CommonMiddleware",
    ...
]
```

**Kolejność jest tu istotna, nie kosmetyczna.** *Middleware* to lista warstw, przez które
przechodzi każde żądanie w drodze do widoku i każda odpowiedź w drodze powrotnej – coś
w rodzaju kolejki kontroli. `CorsMiddleware` musi stać **możliwie wysoko, a na pewno przed
`CommonMiddleware`**, bo warstwy poniżej mogą wygenerować odpowiedź samodzielnie
(przekierowanie, `304 Not Modified`) i wtedy nagłówek CORS nigdy by się nie dokleił.

**Trzecia – lista dozwolonych pochodzeń.** Dopisz na końcu pliku:

```python
CORS_ALLOWED_ORIGINS = [
    "http://localhost:5173",
    "http://127.0.0.1:5173",
]
```

Oba wpisy, bo – jak ustaliliśmy – dla przeglądarki to dwa różne pochodzenia, a Vite
w komunikacie startowym podaje raz jeden, raz drugi.

> Uważaj na przecinki na końcu wpisów w `INSTALLED_APPS`. Brak przecinka po `"corsheaders"`
> skleja go z następnym tekstem i dostaniesz `ModuleNotFoundError` o nazwie modułu, który
> wygląda jak dwa sklejone – błąd bardzo mylący przy pierwszym spotkaniu.

Zrestartuj serwer i powtórz test z konsoli przeglądarki. Tym razem w konsoli pojawi się
tablica produktów.

---

## Krok 5 – sprawdź nagłówek

Warto zobaczyć, że to naprawdę tylko jeden nagłówek. W terminalu:

```bash
curl -i -H "Origin: http://localhost:5173" http://localhost:8000/api/products/
```

W odpowiedzi znajdź:

```
access-control-allow-origin: http://localhost:5173
```

A teraz to samo z pochodzeniem, którego nie ma na liście:

```bash
curl -i -H "Origin: http://zlosliwa-strona.pl" http://localhost:8000/api/products/
```

Dane przyszły, ale nagłówka **nie ma**. Serwer nie odmówił odpowiedzi – po prostu nie
wystawił przepustki. Przeglądarka na tej podstawie zablokowałaby odczyt.

To dobre ćwiczenie, bo pokazuje granice CORS-u: **to nie jest mechanizm autoryzacji**.
CORS chroni użytkownika przed cudzymi stronami, a nie Twoje dane przed odczytem. Kto
chce, weźmie je `curl`-em. Danych chroni się logowaniem i uprawnieniami – to temat na
później.

### Dlaczego nie `CORS_ALLOW_ALL_ORIGINS = True`

W internecie ta linijka jest wszędzie, bo natychmiast rozwiązuje problem. Znaczy jednak:
„każda strona na świecie może czytać moje odpowiedzi".

Przy publicznym katalogu produktów bez logowania to jeszcze nic strasznego. W momencie,
w którym dojdzie logowanie i dane użytkowników, staje się to dziurą. A nawyk zostaje –
i przenosi się na projekt, w którym już strasznie boli.

W tym projekcie wypisujemy origin jawnie. Zawsze.

---

## Krok 6 – konfiguracja w `.env` (backend)

Adresy, klucze i tryb debugowania różnią się między Twoim komputerem a serwerem
produkcyjnym. Trzymanie ich w `settings.py` znaczy, że przy każdym wdrożeniu trzeba
edytować kod. Przenosimy je do pliku, którego nie ma w repozytorium.

```bash
pip install python-dotenv
pip freeze > requirements.txt
```

**`backend/.env`** – prawdziwe wartości:

```
SECRET_KEY=tu-wklej-klucz-ze-swojego-settings.py
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1
CORS_ALLOWED_ORIGINS=http://localhost:5173,http://127.0.0.1:5173
```

**`backend/.env.example`** – wzór, **ten trafia do repozytorium**:

```
SECRET_KEY=
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1
CORS_ALLOWED_ORIGINS=http://localhost:5173,http://127.0.0.1:5173
```

Teraz `core/settings.py`. Na górze pliku:

```python
import os
from pathlib import Path

from dotenv import load_dotenv

BASE_DIR = Path(__file__).resolve().parent.parent

load_dotenv(BASE_DIR / ".env")


def env_list(name, default=""):
    """Zamienia zmienną środowiskową 'a,b,c' na listę ['a', 'b', 'c']."""
    value = os.environ.get(name, default)
    return [item.strip() for item in value.split(",") if item.strip()]
```

I podmiana czterech ustawień:

```python
SECRET_KEY = os.environ["SECRET_KEY"]
DEBUG = os.environ.get("DEBUG", "False") == "True"
ALLOWED_HOSTS = env_list("ALLOWED_HOSTS")
CORS_ALLOWED_ORIGINS = env_list("CORS_ALLOWED_ORIGINS")
```

### Trzy szczegóły, które nie są przypadkowe

**`os.environ["SECRET_KEY"]` w nawiasach kwadratowych, nie `.get()`.** Brak klucza ma
wywalić aplikację przy starcie z czytelnym błędem. Gdyby zadziałało `.get()` z wartością
domyślną, aplikacja wystartowałaby z pustym albo przykładowym kluczem i nikt by tego nie
zauważył – aż do momentu, gdy okaże się, że sesje wszystkich użytkowników dają się
podrobić. **Awaria przy starcie jest lepsza niż cicha dziura.**

**`DEBUG = ... == "True"`** – bo wszystko, co przychodzi ze zmiennych środowiskowych, jest
**tekstem**. Gdybyś napisał `DEBUG = os.environ.get("DEBUG")`, to przy wartości `False`
dostałbyś napis `"False"`, który w Pythonie jest prawdą. Klasyczna pułapka, kosztowna,
bo na produkcji `DEBUG = True` wyświetla obcym ludziom pełne ślady stosu razem
z fragmentami konfiguracji.

**`env_list`** – bo `"".split(",")` zwraca `[""]`, czyli listę z pustym tekstem, a nie
pustą listę. Ta jedna funkcja załatwia problem raz na zawsze.

Sprawdź, czy `.env` na pewno jest ignorowany, i zrestartuj serwer:

```bash
git check-ignore -v backend/.env      # ma coś wypisać
git status --short                    # NIE może tu być .env
```

---

## Krok 7 – konfiguracja w `.env` (frontend)

Adres API też nie powinien być wpisany na sztywno w kodzie – na produkcji będzie zupełnie
inny.

**`frontend/.env`**:

```
VITE_API_URL=http://localhost:8000/api
```

**`frontend/.env.example`** – identyczny, trafia do repozytorium.

Odczyt w kodzie:

```js
const API_URL = import.meta.env.VITE_API_URL;
```

Sprawdź teraz, czy działa – w `src/main.jsx` dopisz tymczasowo:

```js
console.log("API:", import.meta.env.VITE_API_URL);
```

W konsoli przeglądarki ma się pojawić Twój adres. Jeśli widzisz `undefined`, sprawdź dwie
rzeczy poniżej. Po sprawdzeniu tę linię skasuj – w materiale 07 wykorzystamy tę zmienną
naprawdę.

### Prefiks `VITE_` jest obowiązkowy

Vite udostępnia przeglądarce **wyłącznie** zmienne zaczynające się od `VITE_`. Nazwa
`API_URL` bez prefiksu po prostu nie dotrze i dostaniesz `undefined`.

### Restart po każdej zmianie `.env`

Vite czyta ten plik raz, przy starcie. Zapisanie `.env` przy działającym `npm run dev`
nic nie da – trzeba zatrzymać serwer i uruchomić ponownie.

### To nie jest miejsce na sekrety

Rzecz absolutnie kluczowa i notorycznie mylona: **wszystko, co ma prefiks `VITE_`, ląduje
w plikach wysłanych do przeglądarki**. Po `npm run build` znajdziesz tę wartość otwartym
tekstem w `dist/assets/`.

`.env` frontendu służy do **konfiguracji**, nie do ukrywania. Adres API – tak. Klucz do
bramki płatniczej, hasło do bazy, token administratora – **nigdy**. Cokolwiek trafia do
frontendu, trafia do użytkownika. Nie ma od tego wyjątków, bo kod działa na jego
komputerze.

Dwa różne pliki `.env` w jednym projekcie mają więc zupełnie różny status:
`backend/.env` naprawdę chroni sekrety, `frontend/.env` tylko oddziela konfigurację od
kodu. Oba są w `.gitignore`, ale z innego powodu.

---

## Krok 8 – alternatywa: proxy w Vite (dla wiedzy)

Jest inny sposób na obejście problemu w czasie pracy nad projektem: kazać Vite udawać,
że API leży pod tym samym adresem co frontend.

W `frontend/vite.config.js`:

```js
export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      "/api": "http://localhost:8000",
    },
  },
});
```

Wtedy w kodzie piszesz `fetch("/api/products/")` bez pełnego adresu. Vite przechwytuje
takie żądania i przekazuje je do Django. Dla przeglądarki wszystko dzieje się w obrębie
`localhost:5173`, więc **CORS w ogóle nie wchodzi w grę** – to jest to samo pochodzenie.

Wygodne i często spotykane. Ma jednak dwie wady, przez które **nie używamy tego
rozwiązania jako podstawowego**:

- Działa tylko przy `npm run dev`. Po zbudowaniu wersji produkcyjnej proxy znika i CORS
  wraca – więc problem jest odłożony, nie rozwiązany.
- Ukrywa mechanizm. Zrobiłbyś projekt, nie dowiadując się, czym jest origin.

Warto o tym wiedzieć, bo spotkasz to w cudzych projektach i w połowie poradników.

---

## Krok 9 – przy okazji: dostęp z innego urządzenia

Przyda się przy Androidzie, ale skonfigurujmy to od razu.

Domyślnie `python manage.py runserver` nasłuchuje tylko na `127.0.0.1`, czyli jest
niewidoczny dla innych urządzeń. Żeby udostępnić serwer w sieci lokalnej:

```bash
python manage.py runserver 0.0.0.0:8000
```

i w `.env` dopisać do `ALLOWED_HOSTS` adres IP swojego komputera (sprawdzisz go przez
`ipconfig`), na przykład:

```
ALLOWED_HOSTS=localhost,127.0.0.1,192.168.1.15
```

Vite udostępnia się analogicznie:

```bash
npm run dev -- --host
```

Emulator Androida ma tu własną osobliwość: jego `localhost` to on sam, a komputer, na
którym działa, widoczny jest pod adresem `10.0.2.2`. Wrócimy do tego w materiale
o aplikacji mobilnej.

---

## Krok 10 – dokumentacja i zapis

W `README.md` uzupełnij sekcję o zmiennych środowiskowych – tabela z nazwą, opisem
i przykładową wartością, osobno dla backendu i frontendu. Dopisz też krok „skopiuj
`.env.example` do `.env` i uzupełnij" w instrukcji uruchamiania. Bez tego kroku projekt
po sklonowaniu **nie wystartuje**, bo zabraknie `SECRET_KEY`.

```bash
git status --short          # .env NIE może się tu pojawić, .env.example TAK
git add .
git commit -m "feat(backend): obsługa CORS dla frontendu"
```

Podział na commity:

```
feat(backend): django-cors-headers i lista dozwolonych origin
refactor(backend): konfiguracja przeniesiona do .env
feat(frontend): adres API w zmiennej środowiskowej
docs(readme): opis zmiennych środowiskowych
```

---

## Sprawdź, czy rozumiesz

1. Czy błąd CORS oznacza, że serwer odrzucił żądanie? Co dokładnie się stało?
2. Dlaczego `http://localhost:8000` i `http://127.0.0.1:8000` to dwa różne pochodzenia?
3. Dlaczego w Thunder Client wszystko działa, a w przeglądarce nie?
4. Który nagłówek decyduje o przepuszczeniu odpowiedzi i kto go ustawia?
5. Kiedy przeglądarka wysyła żądanie wstępne `OPTIONS`, a kiedy nie?
6. Dlaczego `CorsMiddleware` musi stać przed `CommonMiddleware`?
7. Dlaczego `SECRET_KEY` czytamy przez `os.environ[...]`, a nie `os.environ.get(...)`?
8. Dlaczego do `frontend/.env` nie wolno wpisać hasła, skoro plik jest w `.gitignore`?

---

## Częste błędy

| Objaw | Przyczyna |
|---|---|
| dalej `No 'Access-Control-Allow-Origin' header` | serwer nie został zrestartowany |
| działa na `localhost`, nie działa na `127.0.0.1` | brak drugiego wpisu w `CORS_ALLOWED_ORIGINS` |
| `ModuleNotFoundError: No module named 'corsheadersshop'` | brak przecinka po `"corsheaders"` w `INSTALLED_APPS` |
| `KeyError: 'SECRET_KEY'` | brak pliku `.env` albo brak wpisu – skopiuj `.env.example` |
| `DisallowedHost at /` | adres, z którego wchodzisz, nie jest w `ALLOWED_HOSTS` |
| `import.meta.env.VITE_API_URL` to `undefined` | brak prefiksu `VITE_` albo nie zrestartowano Vite |
| origin z ukośnikiem na końcu nie działa | `CORS_ALLOWED_ORIGINS` nie przyjmuje `/` na końcu |
| nagłówek pojawia się dwa razy | CORS ustawiony i w Django, i w serwerze pośredniczącym |

---

## Czego jeszcze nie ma

- **Uwierzytelniania.** CORS pozwala czytać, ale nikogo nie sprawdza. API jest wciąż
  całkowicie publiczne.
- **HTTPS.** Lokalnie wszystko chodzi po `http`. Na produkcji origin będzie zaczynał się
  od `https` i lista dozwolonych pochodzeń będzie inna.
- **`CORS_ALLOW_CREDENTIALS`.** Potrzebne, gdyby uwierzytelnianie działało na
  ciasteczkach. Przy tokenach JWT w nagłówku – nie.

---

## Co dalej

**Materiał 07:** React pobierający dane z `/api/products/`. Kasujesz `data.js`, dodajesz
`useEffect` i `fetch`, obsługujesz stany ładowania i błędu. Komponenty `ProductCard`
i `ProductDetails` zostają bez zmian.

---

# Zadania

**Zadanie 1 – diagnoza**
Zanim cokolwiek zmienisz: wywołaj `fetch` na własne API z konsoli przeglądarki na stronie
swojego frontendu. Zrób zrzut ekranu konsoli **i** zakładki Network. W `docs/cors.md`
opisz, jaki status zwrócił serwer i dlaczego dane mimo to nie dotarły do skryptu.

**Zadanie 2 – konfiguracja**
Zainstaluj i skonfiguruj `django-cors-headers`. Origin wypisz jawnie, oba warianty
(`localhost` i `127.0.0.1`). Zaktualizuj `requirements.txt`.

**Zadanie 3 – dowód**
Wykonaj `curl` z nagłówkiem `Origin` dozwolonym i niedozwolonym. Obie odpowiedzi wklej
do `docs/cors.md` i zaznacz, czym się różnią. Jednym zdaniem: dlaczego CORS nie jest
zabezpieczeniem danych przed odczytem?

**Zadanie 4 – żądanie wstępne**
Wywołaj żądanie wstępne ręcznie:

```bash
curl -i -X OPTIONS -H "Origin: http://localhost:5173" \
     -H "Access-Control-Request-Method: POST" \
     http://localhost:8000/api/products/
```

Opisz w `docs/cors.md`, jakie nagłówki wróciły i co przeglądarka z nich wyczyta.

**Zadanie 5 – zmienne backendu**
Przenieś `SECRET_KEY`, `DEBUG`, `ALLOWED_HOSTS` i `CORS_ALLOWED_ORIGINS` do `.env`.
Utwórz `.env.example`. Sprawdź, że `.env` nie wchodzi do repozytorium, a `.env.example`
wchodzi.

**Zadanie 6 – test odporności**
Zmień tymczasowo nazwę pliku `.env` i uruchom serwer. Powinien odmówić startu
z komunikatem o `SECRET_KEY`. W `docs/cors.md` napisz, dlaczego to zachowanie jest
lepsze niż start z kluczem domyślnym.

**Zadanie 7 – zmienne frontendu**
Utwórz `frontend/.env` z adresem API i `.env.example`. Sprawdź odczyt przez
`console.log`. Zapisz w `docs/cors.md`, co się stanie, jeśli usuniesz prefiks `VITE_`.

**Zadanie 8 – dowód, że to nie sekret**
Wpisz do `frontend/.env` zmienną `VITE_TAJNE=abc123`, użyj jej gdziekolwiek w kodzie,
uruchom `npm run build` i **znajdź tę wartość** w plikach w katalogu `dist/`
(podpowiedź: `findstr /s "abc123" dist\*`). Wklej wynik do `docs/cors.md`. Potem usuń
tę zmienną.

**Zadanie 9 – dokumentacja**
Tabele zmiennych środowiskowych w `README.md`, osobno dla obu części. Instrukcja
uruchomienia ma zawierać kopiowanie `.env.example`.

**Zadanie 10 – test z czystego klona**
Sklonuj własne repozytorium do nowego katalogu i uruchom całość **wyłącznie** według
README. Każde miejsce, w którym musiałeś się domyślić, popraw w dokumentacji.

---

**Zadanie dodatkowe (dla chętnych)**
Skonfiguruj proxy w `vite.config.js` i przełącz kod na adresy względne (`/api/products/`).
Sprawdź w zakładce Network, na jaki adres faktycznie idzie żądanie i jakie nagłówki mu
towarzyszą. W `docs/cors.md` odpowiedz: dlaczego przy proxy nie ma żadnego nagłówka
`Access-Control-Allow-Origin` i co się stanie z tym rozwiązaniem po `npm run build`?