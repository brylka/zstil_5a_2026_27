# Git i GitHub – wstęp

**Technik programista | klasa 5 | rok projektu fullstack**

---

## Czym ten rok różni się od poprzedniego

W czwartej klasie repozytorium było magazynem: wrzucasz projekt, jest dowód, że pracowałeś. W piątej repozytorium staje się **narzędziem pracy**, bo po raz pierwszy budujesz coś, co ma **dwie strony**:

```
   FRONTEND                               BACKEND
   React + Vite                           Django + DRF
   localhost:5173   ---- HTTP/JSON ---->  localhost:8000
                    <-------------------
```

Dwie strony to dwa zestawy plików, dwa terminale, dwa `.gitignore`, dwie listy zależności i dwa miejsca, w których można coś zepsuć. Do tego dochodzi rzecz, której w czwartej klasie prawie nie było: **zmiana w backendzie potrafi zepsuć frontend**. Zmieniasz nazwę pola w serializerze – i formularz w Reakcie przestaje działać, chociaż nikt go nie dotykał.

Git jest tym, co pozwala nad tym zapanować: pokazuje, co dokładnie się zmieniło, pozwala cofnąć zmianę i pozwala pracować nad nową funkcją bez rozwalania działającej wersji.

> Wszystko, co robisz na zajęciach, ląduje na Twoim GitHubie. To jednocześnie ocena z testowania i dokumentowania, portfolio na rozmowę o pracę i Twoje własne archiwum na czas, gdy w styczniu będziesz sobie przypominał, jak się robi CRUD.

---

## Szybka powtórka (jeśli to masz – przeskocz)

### Instalacja i konfiguracja (raz na komputer)

```bash
git --version                                   # sprawdzenie, czy jest

git config --global user.name "Imię Nazwisko"
git config --global user.email "twoj@email.com" # TEN SAM e-mail co na GitHubie
git config --global init.defaultBranch main
```

Jeśli Gita nie ma: https://git-scm.com/download/win – instalator, same domyślne opcje.

### Model: cztery miejsca

```
 katalog roboczy --git add--> poczekalnia --git commit--> historia lokalna
   (edytujesz)                 (staging)                             |
                                                            git push | git pull
                                                                     ▼
                                                                   GitHub
```

### Codzienna pętla

```bash
git status                      # co się zmieniło
git add .                       # do poczekalni
git commit -m "Co zrobiłem"     # zapis w historii
git push                        # wysyłka na GitHub
```

Te cztery komendy to 90% pracy. Reszta tego dokumentu to pozostałe 10%, które w projekcie fullstack robi różnicę.

---

## Jak zorganizować repozytorium projektu fullstack

Masz dwa sensowne warianty. W tej klasie pracujemy w **monorepo**, ale musisz wiedzieć, że drugi istnieje, bo w firmach spotkasz oba.

| | **Monorepo** (jedno repo) | **Dwa repozytoria** |
|---|---|---|
| Struktura | `backend/` i `frontend/` obok siebie | `sklep-api` + `sklep-web` |
| Zmiana API + frontendu | jeden commit, jeden PR | dwa commity w dwóch miejscach |
| Historia | widać, że zmiana pola i poprawka formularza to jedna rzecz | trzeba się domyślać |
| Wdrożenie | trzeba wskazać podkatalog | prostsze, każde osobno |
| Kiedy stosować | mały zespół, jeden projekt | osobne zespoły, osobne cykle wydawnicze |

### Struktura, którą zakładamy

```
sklep-fullstack/
├── .git/
├── .gitignore              ← jeden, wspólny, obsługuje oba stacki
├── .github/
│   └── workflows/
│       └── ci.yml          ← automatyczne testy przy każdym PR
├── README.md               ← wizytówka projektu (osobny dokument)
├── docs/
│   ├── api.md              ← lista endpointów
│   └── ERD.png             ← diagram bazy
├── backend/
│   ├── manage.py
│   ├── requirements.txt
│   ├── .env.example        ← wzór, BEZ prawdziwych haseł
│   ├── core/
│   └── sklep/
└── frontend/
    ├── package.json
    ├── .env.example
    ├── index.html
    └── src/
```

Utworzenie tego od zera:

```bash
mkdir sklep-fullstack && cd sklep-fullstack
git init
mkdir backend frontend docs
# ... tworzysz pliki ...
git add .
git commit -m "chore: struktura projektu (backend + frontend)"
gh repo create sklep-fullstack --public --source=. --push
```

Ostatnia linia wymaga narzędzia `gh` (GitHub CLI). Bez niego: załóż repo klikając na github.com, potem `git remote add origin URL` i `git push -u origin main`.

---

## `.gitignore` – teraz obsługuje dwa światy

Jeden plik w katalogu głównym. Ścieżki są względne do miejsca, w którym leży.

```gitignore
# --- Python / Django ---
__pycache__/
*.py[cod]
venv/
.venv/
db.sqlite3
media/
staticfiles/

# --- Node / React ---
node_modules/
dist/
build/
.vite/
npm-debug.log*

# --- sekrety (OBOWIĄZKOWO) ---
.env
.env.local
*.pem

# --- edytory i system ---
.idea/
.vscode/
.DS_Store
Thumbs.db
```

Sprawdzenie, czy działa – zanim zrobisz pierwszy commit:

```bash
git status --short          # nie powinno być tu node_modules ani venv
git check-ignore -v .env    # pokaże, która reguła ignoruje ten plik
```

> **Najczęstszy błąd piątej klasy:** `node_modules/` w repozytorium. To 200 MB plików, których nikt nie potrzebuje – odtwarza się jednym `npm install`. Jeśli już to wypchnąłeś: `git rm -r --cached node_modules` → commit → push.

---

## Sekrety: `.env` i `.env.example`

W projekcie fullstack pojawiają się rzeczy, których **nigdy** nie commitujesz: `SECRET_KEY` Django, hasło do bazy, klucz do bramki płatności, token API pogodowego.

**`backend/.env`** – realne wartości, ignorowany przez Gita:
```
SECRET_KEY=django-insecure-prawdziwy-klucz-abc123
DEBUG=True
DATABASE_URL=postgres://user:haslo@localhost:5432/sklep
```

**`backend/.env.example`** – wzór, **jest** w repozytorium:
```
SECRET_KEY=
DEBUG=True
DATABASE_URL=
```

**`frontend/.env`**:
```
VITE_API_URL=http://localhost:8000/api
```

Dzięki `.env.example` ktoś, kto sklonuje Twoje repo (ja, kolega, przyszły pracodawca), wie **jakie** zmienne musi ustawić, nie znając ich wartości. To jest część dokumentacji projektu.

> Jeśli wypchniesz hasło na GitHuba – **zmień to hasło**. Usunięcie pliku kolejnym commitem nic nie daje, stara wersja zostaje w historii i każdy może ją odczytać. Boty skanujące GitHuba w poszukiwaniu kluczy działają w minutach, nie dniach.

---

## Gałęzie – po co i kiedy

Do tej pory pewnie pracowałeś tylko na `main`. W projekcie, który ma działać, `main` powinien być **zawsze uruchamialny**. Nowe rzeczy robisz na gałęzi.

```bash
git switch -c feature/koszyk      # utwórz gałąź i przejdź na nią
# ... pracujesz, commitujesz ...
git push -u origin feature/koszyk # wyślij gałąź na GitHuba
```

Powrót i sprzątanie:

```bash
git switch main                   # wróć na główną
git branch                        # lista gałęzi lokalnych
git branch -d feature/koszyk      # usuń po scaleniu
```

### Nazewnictwo gałęzi

| Prefiks | Kiedy | Przykład |
|---|---|---|
| `feature/` | nowa funkcja | `feature/logowanie-jwt` |
| `fix/` | naprawa błędu | `fix/pusty-koszyk-500` |
| `docs/` | tylko dokumentacja | `docs/opis-api` |
| `refactor/` | porządki bez zmiany działania | `refactor/serializery` |

### Dlaczego to ma sens akurat przy fullstacku

Funkcja „koszyk" to zwykle: nowy model, nowa migracja, nowy serializer, nowy endpoint, nowy komponent w Reakcie i nowe wywołanie `fetch`. Sześć plików w dwóch katalogach. Na osobnej gałęzi możesz to zostawić w połowie na trzy dni, a `main` dalej się uruchamia i dalej można go pokazać.

---

## Pull Request – i dlaczego u nas nie pushujesz na `main`

**Zasada w tej klasie: na `main` nic nie trafia bezpośrednio.** Otwierasz Pull Request, ja go czytam i komentuję, Ty odpowiadasz albo poprawiasz, dopiero potem merge.

Pełny cykl:

```bash
git switch main
git pull                              # 1. zacznij od aktualnego stanu
git switch -c feature/koszyk          # 2. gałąź
# ... praca, commity ...
git push -u origin feature/koszyk     # 3. wypchnij
gh pr create --fill                   # 4. otwórz PR (albo klikając na GitHubie)
# ... review, poprawki, kolejne commity + push ...
gh pr merge --squash --delete-branch   # 5. po akceptacji
```

### Co ma być w opisie PR

To nie jest formalność – to jest Twoja dokumentacja procesu i realna umiejętność zawodowa.

```markdown
## Co robi
Dodaje koszyk: model, endpointy i widok w Reakcie.

## Zmiany w API
- `GET /api/koszyk/` – zawartość koszyka zalogowanego użytkownika
- `POST /api/koszyk/pozycje/` – dodanie produktu (body: `produkt_id`, `ilosc`)

## Jak sprawdzić
1. `cd backend && python manage.py migrate && python manage.py runserver`
2. `cd frontend && npm run dev`
3. Zaloguj się, kliknij „Dodaj do koszyka", odśwież stronę – pozycja zostaje.

## Czego nie zrobiłem
Brak walidacji ilości > stan magazynowy – osobne zgłoszenie #14.

Closes #7
```

Wpisanie `Closes #7` automatycznie zamknie zgłoszenie nr 7 po scaleniu PR-a.

### Na co patrzę w review

- Czy `main` po scaleniu dalej się uruchamia.
- Czy zmiana w API ma odpowiednik po stronie frontendu (albo świadomą adnotację, że nie ma).
- Czy w diffie nie ma `.env`, `node_modules`, zakomentowanego kodu i `console.log` z debugowania.
- Czy commitów jest kilka i każdy mówi, co robi – nie jeden na 600 linii.
- **Czy umiesz opowiedzieć, co robi kod.** To jest główne kryterium i ono się nie zmienia od czwartej klasy.

---

## Konflikty scalania

Konflikt powstaje, gdy dwie gałęzie zmieniły **te same linie tego samego pliku**. Git nie zgaduje – pyta Ciebie.

```bash
git switch feature/koszyk
git merge main
# CONFLICT (content): Merge conflict in frontend/src/App.jsx
```

W pliku zobaczysz:

```jsx
<<<<<<< HEAD
  <Route path="/koszyk" element={<Koszyk />} />
=======
  <Route path="/zamowienia" element={<Zamowienia />} />
>>>>>>> main
```

Między `<<<<<<<` a `=======` jest Twoja wersja, między `=======` a `>>>>>>>` – ta z `main`. Rozwiązanie: **usuwasz znaczniki i zostawiasz kod, który ma być**. Bardzo często odpowiedzią jest „obie linie", bo to po prostu dwie różne trasy:

```jsx
  <Route path="/koszyk" element={<Koszyk />} />
  <Route path="/zamowienia" element={<Zamowienia />} />
```

Potem:

```bash
git add frontend/src/App.jsx
git commit                      # domyślny komunikat o scaleniu jest OK
```

Ucieczka, jeśli się pogubisz: `git merge --abort` cofa wszystko do stanu sprzed scalania. Nic nie ginie.

**Pliki, które konfliktują najczęściej:** `App.jsx` (trasy), `urls.py` (adresy), `models.py` (nowe modele na końcu pliku), `requirements.txt` i `package.json` (nowe zależności). Prosta profilaktyka: rób `git pull` na `main` codziennie i wciągaj `main` do swojej gałęzi, zamiast czekać dwa tygodnie.

---

## GitHub jako system zgłoszeń i zarządzania

Podstawa programowa wymaga znajomości systemów śledzenia błędów i narzędzi do zarządzania projektem. Nie potrzebujesz Jiry – GitHub ma to wbudowane i tego używamy.

**Issues** – jedno zgłoszenie = jedna rzecz do zrobienia albo jeden błąd.

```bash
gh issue create --title "Koszyk gubi zawartość po odświeżeniu" \
                --body "Kroki: dodaj produkt, F5, koszyk pusty. Oczekiwane: pozycje zostają."
gh issue list
```

Dobre zgłoszenie błędu ma: **kroki odtworzenia**, **oczekiwany rezultat**, **rzeczywisty rezultat**. Trzy zdania. To dokładnie ten format, którego używa się w pracy.

**Labels** – `bug`, `backend`, `frontend`, `dokumentacja`, `pilne`.
**Milestones** – grupują zgłoszenia w etapy: „API gotowe", „Frontend MVP", „Wersja na obronę".
**Projects** – tablica kanban: *Do zrobienia → W trakcie → Do sprawdzenia → Gotowe*. To jest odpowiednik Trello, tylko połączony z kodem.

---

## Wersje i wydania

Kiedy projekt osiąga stan, który działa w całości, oznaczasz go tagiem:

```bash
git tag -a v1.0.0 -m "Pierwsza działająca wersja: API + frontend + logowanie"
git push origin v1.0.0
```

Numeracja semantyczna `GŁÓWNA.MNIEJSZA.POPRAWKA`:
- `1.0.0 → 1.0.1` – poprawka błędu,
- `1.0.1 → 1.1.0` – nowa funkcja, stare działa dalej,
- `1.1.0 → 2.0.0` – zmiana, która psuje zgodność (np. zmieniona nazwa pola w API).

Ta ostatnia sytuacja jest przy fullstacku bardzo konkretna: jeśli zmienisz `nazwa` na `tytul` w serializerze, każdy klient tego API przestaje działać. Dlatego to jest zmiana „główna".

---

## Automatyczne testy przy każdym PR (GitHub Actions)

Plik `.github/workflows/ci.yml`. Przy każdym pushu GitHub uruchomi testy backendu i zbuduje frontend. Jeśli coś nie przejdzie, przy PR pojawi się czerwony krzyżyk.

```yaml
name: CI

on: [push, pull_request]

jobs:
  backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.14'
      - name: Instalacja zależności
        run: pip install -r backend/requirements.txt
      - name: Testy
        run: cd backend && python manage.py test

  frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '24'
      - name: Instalacja zależności
        run: cd frontend && npm ci
      - name: Build
        run: cd frontend && npm run build
```

To jest ten sam mechanizm, który w firmach nazywa się CI. Jego wartość dydaktyczna: **twoje testy przestają być czymś, co uruchamiasz, gdy pamiętasz**.

---

## Komunikaty commitów – konwencja

Stosujemy prosty schemat `typ(zakres): opis`:

```
feat(api): endpoint GET /api/produkty
fix(frontend): koszyk nie gubi pozycji po odświeżeniu
docs(readme): instrukcja uruchomienia obu części
test(api): testy serializera Zamowienie
refactor(models): wydzielenie klasy bazowej Produkt
chore(deps): aktualizacja django do 6.1
```

Typy: `feat`, `fix`, `docs`, `test`, `refactor`, `chore`, `style`.
Zakres: `api`, `frontend`, `models`, `auth`, `db` – cokolwiek mówi, gdzie to jest.

**Zakres jest opcjonalny.** Podajesz go, gdy zmiana dotyczy jednego wycinka projektu.
Pomijasz, gdy obejmuje całość – `chore: struktura projektu (backend + frontend)` nie ma
sensownego zakresu, bo tworzy wszystko naraz. Nie wymyślaj zakresu na siłę, ale też nie
pomijaj go tam, gdzie coś mówi.

| Dobrze | Źle |
|---|---|
| `feat(auth): logowanie JWT + odświeżanie tokenu` | `zmiany` |
| `fix(api): 500 przy pustym koszyku` | `poprawki` |
| `docs: opis endpointów w docs/api.md` | `update` |
| `test(cart): 4 testy dodawania pozycji` | `dziala juz` |

Zasada bez wyjątków: **jeden commit = jedna rzecz**. Commit „dodałem wszystko" na 600 linii jest nieoceniany, nieodwracalny i mówi mi, że nie czytałeś tego, co wygenerował agent.

---

## Ściąga komend

| Komenda | Co robi |
|---|---|
| `git status` | co jest zmienione |
| `git diff` | które linie się zmieniły (przed `add`) |
| `git diff --staged` | co jest w poczekalni |
| `git log --oneline --graph --all` | historia z gałęziami, graficznie |
| `git switch -c nazwa` | nowa gałąź + przejście |
| `git switch nazwa` | przejście na istniejącą |
| `git merge nazwa` | wciągnij gałąź do bieżącej |
| `git pull` | pobierz zmiany ze zdalnego |
| `git push -u origin nazwa` | wypchnij nową gałąź |
| `git restore plik` | cofnij niezapisane zmiany w pliku |
| `git restore --staged plik` | wyjmij z poczekalni |
| `git reset --soft HEAD~1` | cofnij ostatni commit, zmiany zostaw |
| `git stash` / `git stash pop` | schowaj zmiany na chwilę / przywróć |
| `git show <hash>` | co dokładnie zmienił dany commit |
| `git blame plik` | kto i kiedy napisał każdą linię |
| `gh pr create --fill` | otwórz Pull Request |
| `gh pr status` | stan Twoich PR-ów |
| `gh issue create` | nowe zgłoszenie |

---

## Zasady w tej klasie

1. **Jedno repozytorium na cały rok** – monorepo `backend/` + `frontend/`. Nie zakładasz nowego przy każdej technologii.
2. **Na `main` nie pushujesz.** Gałąź → PR → review → merge.
3. **Każdy PR ma opis** według wzoru wyżej i jest podpięty pod zgłoszenie.
4. `README.md` jest aktualny **zawsze**. Jeśli zmienił się sposób uruchomienia, README zmienia się w tym samym PR.
5. `.env` nigdy nie trafia do repozytorium, `.env.example` zawsze.
6. Na koniec zajęć: commit + push. Brak pusha = brak dowodu, że pracowałeś.
7. **Możesz używać LLM.** Ale na kolejnych zajęciach dostajesz pytania o ten kod. Wiesz, co robi każda linia – świetnie, nieważne kto ją napisał. Nie wiesz – do poprawy. Bez zmian od czwartej klasy.

---

## Zadania

**Zadanie 1 – struktura**
Załóż repozytorium `<twoja-domena>-fullstack` z katalogami `backend/` i `frontend/`, wspólnym `.gitignore`, `README.md` i dwoma plikami `.env.example`. Push na GitHuba. W `git status --short` po `npm install` i utworzeniu `venv` nie może być ani `node_modules`, ani `venv`.

**Zadanie 2 – gałąź i PR**
Na gałęzi `feature/pierwszy-endpoint` dodaj dowolny endpoint zwracający JSON. Otwórz PR z pełnym opisem. Nie scalaj – czekasz na mój komentarz.

**Zadanie 3 – konflikt na zamówienie**
W parze: obaj tworzycie gałąź z `main` i obaj dopisujecie trasę w `App.jsx`. Jeden scala pierwszy. Drugi ma konflikt – rozwiąż go i opisz w `docs/konflikt.md`, co dokładnie zrobiłeś i dlaczego.

**Zadanie 4 – zgłoszenia**
Załóż 5 zgłoszeń opisujących funkcje Twojego projektu. Nadaj etykiety `backend` / `frontend`. Utwórz milestone „API gotowe" i przypisz do niego te, które go dotyczą. Ustaw tablicę Projects z czterema kolumnami.

**Zadanie 5 – CI**
Dodaj `.github/workflows/ci.yml`. Doprowadź do zielonej „fajki" przy PR. Potem **celowo** zepsuj jeden test, wypchnij i zobacz czerwony krzyżyk. Napraw.

**Zadanie 6 – archeologia**
W dowolnym publicznym repozytorium z Django lub Reactem znajdź commit naprawiający błąd. `git show <hash>`. Opisz w `docs/analiza.md`: co było zepsute, jak naprawili, ile linii zmienili.

**Zadanie 7 – cofanie**
Zrób commit z celowym błędem. Cofnij go trzema sposobami i opisz różnicę: `git reset --soft HEAD~1`, `git reset --hard HEAD~1`, `git revert HEAD`. Który z nich jest bezpieczny na gałęzi, którą ktoś już pobrał?