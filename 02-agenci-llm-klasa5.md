# Agenci LLM w projekcie fullstack

**Technik programista | klasa 5 | Claude Code · Codex CLI · Gemini CLI · Copilot**

---

## Zacznijmy od rzeczy najważniejszej

W styczniu zdajesz **INF.04**. Na egzaminie nie ma internetu, nie ma czatu, nie ma agenta. Jest edytor, dokumentacja lokalna i cztery godziny.

To nie znaczy, że przez cały rok nie wolno używać AI. To znaczy, że **sposób** używania AI musi Cię do tego egzaminu przybliżać, a nie oddalać. Uczeń, który przez pół roku pisał „zrób mi CRUD" i wklejał wynik, w styczniu siada przed pustym plikiem i nie wie, od czego zacząć. Uczeń, który tego samego agenta pytał „dlaczego to działa" i „co się stanie, jak zmienię tę linię", siada i pisze.

Cały ten dokument sprowadza się do jednego zdania: **agent ma być korepetytorem, który przy okazji pisze, a nie pisarzem, który przy okazji tłumaczy.**

---

## Czym agent różni się od czatu

Czat: kopiujesz kod do okna, dostajesz odpowiedź, wklejasz z powrotem. Przy projekcie z dwoma katalogami, trzydziestoma plikami i migracjami – nie do utrzymania. Agent nie widzi tylko tego, co wkleiłeś. Agent widzi **cały projekt**.

Uruchamiasz go w katalogu projektu w terminalu. On:

- czyta pliki, także te, o których nie wspomniałeś,
- edytuje je i pokazuje diff do zatwierdzenia,
- uruchamia komendy (`python manage.py test`, `npm run build`, `git diff`),
- czyta błędy z konsoli i sam się poprawia,
- pyta o zgodę przed poważniejszymi zmianami.

Przy projekcie fullstack to jest różnica jakościowa. Prośba *„dodaj pole `stan_magazynowy` do modelu Produkt i uwzględnij je wszędzie, gdzie trzeba"* to w praktyce: model, migracja, serializer, widok, komponent w Reakcie, testy. Agent widzi te wszystkie miejsca naraz. Czat widzi jeden wklejony plik.

---

## Trzy narzędzia (stan: wrzesień 2026)

| Narzędzie | Firma | Konto | Koszt |
|---|---|---|---|
| **Claude Code** | Anthropic | Claude Pro / Max / Team / Enterprise albo Claude Console | płatny |
| **Codex CLI** | OpenAI | ChatGPT Plus / Pro / Edu albo klucz API | płatny |
| **Gemini CLI** | Google | zwykłe konto Google | **darmowy limit** |
| **GitHub Copilot** | GitHub | Student Developer Pack | darmowy po weryfikacji (plan Copilot Student) |

**Jeśli nie masz abonamentu – zaczynasz od Gemini CLI.** Jest darmowy na zwykłym koncie Google, a wszystko, czego się przy nim nauczysz, przenosi się 1:1 na pozostałe. Interfejs, sposób promptowania i pułapki są takie same.

Osobno warto złożyć wniosek o **GitHub Student Developer Pack** (https://education.github.com/pack). Wymagania: ukończone 13 lat, nauka w szkole kończącej się dyplomem oraz **osobiste** konto na GitHubie (konto organizacji nie kwalifikuje się). Weryfikacja idzie po szkolnym adresie e-mail albo po skanie dokumentu z datą ważności – legitymacji lub zaświadczenia; przyjdźcie po nie do sekretariatu. Rozmyte zdjęcia i dokumenty bez daty to najczęstszy powód odrzucenia wniosku.

Po weryfikacji dostajesz plan **Copilot Student**: nieograniczone podpowiedzi w edytorze plus przydział kredytów AI na czat i tryb agentowy, przy czym model dobierany jest automatycznie – nie wybierasz go ręcznie. W pakiecie jest też GitHub Pro, licencja JetBrains (PyCharm, IntelliJ) i Codespaces. Oferty partnerów bywają ograniczone wiekiem: pełne Azure dla studentów wymaga 18 lat, dla młodszych jest osobna, węższa wersja „Microsoft Azure (for ages 13–17)".

> Zawartość pakietu i zasady zmieniają się w czasie – między kwietniem a czerwcem 2026 GitHub miał zawieszone nowe aktywacje Copilota. Sprawdzajcie aktualny stan na stronie pakietu, zanim na nim coś zaplanujecie.

### Wymagania wstępne

1. **Git for Windows** – Claude Code używa Git Bash jako powłoki. Bez niego działa na PowerShellu, ale gorzej.
2. **Node.js LTS** (https://nodejs.org) – potrzebny do Codexa i Gemini, a i tak masz go od Reacta.
   ```powershell
   node --version
   npm --version
   ```
3. **Terminal** – PowerShell. Prompt zaczyna się od `PS C:\`. Jeśli nie ma `PS` – jesteś w CMD i część komend nie zadziała.

---

## Claude Code

Dokumentacja: https://code.claude.com/docs/en/quickstart

```powershell
irm https://claude.ai/install.ps1 | iex      # instalacja
claude --version                             # sprawdzenie
```

Jeśli terminal nie widzi komendy `claude` – zamknij go i otwórz nowy. Jeśli dalej nie widzi, instalator wypisał ścieżkę (zwykle `C:\Users\NAZWA\.local\bin`), którą trzeba dodać do PATH. Komenda `claude doctor` diagnozuje problemy z instalacją i logowaniem.

```powershell
cd sciezka\do\projektu
claude
```

Pierwsze uruchomienie otwiera przeglądarkę – logujesz się kontem Claude (wymagany plan Pro, Max, Team lub Enterprise; darmowy plan Claude.ai nie obejmuje Claude Code). Zmiana konta później: `/login`.

Przydatne w sesji:

| Komenda | Co robi |
|---|---|
| `/init` | analizuje projekt i generuje `CLAUDE.md` |
| `/help` | lista komend |
| `/clear` | czyści kontekst rozmowy |
| `Shift+Tab` | przełącza tryb uprawnień (pyta / nie pyta o każdą zmianę) |
| `claude -c` | (z terminala) wznów ostatnią rozmowę w tym katalogu |

---

## Codex CLI (OpenAI)

Dokumentacja: https://github.com/openai/codex

```powershell
npm install -g @openai/codex
codex --version
cd sciezka\do\projektu
codex
```

Logowanie: *Sign in with ChatGPT* (Plus, Pro, Edu lub wyżej) albo klucz API – przy kluczu płacisz za zużycie, więc uważaj. Konfiguracja siedzi w `~/.codex/config.toml`. Plik z instrukcjami dla projektu to **`AGENTS.md`**.

---

## Gemini CLI (Google)

Dokumentacja: https://github.com/google-gemini/gemini-cli

```powershell
npm install -g @google/gemini-cli
gemini --version
cd sciezka\do\projektu
gemini
```

Logowanie przez *Login with Google* – zwykłe konto od Gmaila. Darmowy limit (rzędu 60 zapytań na minutę i ok. 1000 dziennie) na zajęcia wystarcza z zapasem. Alternatywnie klucz z https://aistudio.google.com/apikey w zmiennej `GEMINI_API_KEY`. Plik projektowy: **`GEMINI.md`**.

---

## Porównanie

| | Claude Code | Codex CLI | Gemini CLI |
|---|---|---|---|
| Uruchomienie | `claude` | `codex` | `gemini` |
| Plik projektowy | `CLAUDE.md` | `AGENTS.md` | `GEMINI.md` |
| Instalacja | `irm https://claude.ai/install.ps1 \| iex` | `npm i -g @openai/codex` | `npm i -g @google/gemini-cli` |
| Logowanie | konto Claude | konto ChatGPT | konto Google |
| Darmowe? | nie | nie | tak (z limitem) |

Wszystkie trzy czytają coraz częściej wspólny `AGENTS.md` – jeśli chcesz mieć jeden plik zamiast trzech, zacznij od niego.

---

## Plik projektowy – najważniejsze pół godziny w tym roku

Agent czyta ten plik na starcie każdej sesji. Bez niego za każdym razem od nowa zgaduje, jaka jest struktura projektu i jakie masz zasady. Z nim – wie.

W projekcie fullstack ten plik jest szczególnie ważny, bo agent musi wiedzieć, że **istnieją dwie strony i że zmiana po jednej pociąga zmianę po drugiej**.

**`AGENTS.md`** w katalogu głównym repozytorium:

```markdown
# Projekt: sklep-fullstack

Aplikacja e-commerce. Backend Django + DRF, frontend React + Vite.
Projekt szkolny – technik programista, klasa 5, przygotowanie do INF.04.

## Struktura
- `backend/` – Django 5 + Django REST Framework, baza SQLite
- `frontend/` – React 18 + Vite, wywołania przez `fetch`
- Backend na porcie 8000, frontend na 5173

## Zasady techniczne
- API zawsze pod prefiksem `/api/`, odpowiedzi w formacie JSON
- Nazwy pól w API: snake_case (tak jak w modelach Django)
- Autoryzacja: JWT (SimpleJWT), token w nagłówku `Authorization: Bearer <token>`
- Nowe zależności backendu dopisuj do `requirements.txt`
- Przed każdym commitem: `python manage.py test`

## Zasady dydaktyczne – WAŻNE
- Zanim napiszesz kod, wyjaśnij mi w 3–4 zdaniach, co zamierzasz zrobić i dlaczego
- Komentuj po polsku, nietrywialne fragmenty obowiązkowo
- Nie wprowadzaj bibliotek, których nie ma w podstawie INF.04, bez wyraźnej prośby
- Rób małe kroki. Jedna funkcja na raz, nie cały moduł
- Po zmianie w API zawsze przypomnij, co trzeba poprawić po stronie frontendu
- Nie commituj za mnie i nie pushuj

## Czego nie ruszać
- `.env` – nigdy nie czytaj i nie modyfikuj
- Istniejących migracji w `backend/*/migrations/`
```

Sekcja „zasady dydaktyczne" to nie ozdobnik. To dosłownie instrukcja, dzięki której agent zachowuje się jak korepetytor, a nie jak automat do kodu. Bez niej domyślnie dostaniesz gotowe 200 linii bez słowa wyjaśnienia.

W Claude Code możesz wygenerować pierwszą wersję komendą `/init` – ale **przeczytaj ją i popraw** przed commitem. Wygenerowana wersja opisuje projekt, nie Twoje zasady nauki.

---

## Gdzie agent naprawdę pomaga przy łączeniu backendu z frontendem

To jest część, która odróżnia ten rok od poprzedniego. Sytuacje, w których agent oszczędza godziny:

### 1. Błędy CORS

Klasyk. Frontend na 5173, backend na 8000, w konsoli przeglądarki czerwień:

```
Access to fetch at 'http://localhost:8000/api/produkty/' from origin
'http://localhost:5173' has been blocked by CORS policy
```

Zamiast wklejać to w wyszukiwarkę: *„mam ten błąd, wyjaśnij mi najpierw, czym jest CORS i dlaczego przeglądarka to blokuje, a potem pokaż, co dodać w settings.py"*. Kolejność ma znaczenie – najpierw zrozumienie mechanizmu, potem cztery linijki konfiguracji `django-cors-headers`.

### 2. Niezgodność kontraktu API

Backend zwraca `{"nazwa_produktu": ...}`, frontend czyta `produkt.nazwa`. Nic nie wybucha – po prostu wyświetla się `undefined`. To jeden z najbardziej frustrujących błędów w fullstacku, bo nie ma komunikatu.

*„Porównaj, jakie pola zwraca ProduktSerializer, z tym, czego używa komponent ListaProduktow. Wypisz różnice, nic nie zmieniaj."*

Agent czyta oba pliki naraz. Ty przez dziesięć minut przeklikiwałbyś się między katalogami.

### 3. Uwierzytelnianie tokenem

*„Wyjaśnij mi przepływ JWT: co się dzieje od kliknięcia »Zaloguj« do wyświetlenia danych chronionych. Wypisz kolejno, po stronie frontendu i backendu. Narysuj to jako listę kroków."*

Potem dopiero implementacja. Uwierzytelnianie to jest temat, w którym „działa, ale nie wiem czemu" kończy się dziurą bezpieczeństwa.

### 4. Odczytanie cudzego błędu

Wklej pełny traceback Django albo błąd z konsoli przeglądarki i poproś: *„wyjaśnij, co ten błąd znaczy, wskaż linię, która go powoduje, i powiedz, jaka jest przyczyna – nie naprawiaj jeszcze"*. Umiejętność czytania stack trace'a jest w pracy warta więcej niż znajomość dowolnego frameworka.

### 5. Testy

*„Napisz testy dla endpointu POST /api/koszyk/pozycje/: przypadek poprawny, brak autoryzacji, nieistniejący produkt, ilość ujemna."*

Wypisanie przypadków testowych jest łatwiejsze niż ich implementacja i to Ty powinieneś je wymyślić. Implementację może napisać agent – ale listę przypadków wymyślasz sam, bo to jest właśnie myślenie o testowaniu.

### 6. Dokumentacja

*„Na podstawie plików urls.py i serializers.py wygeneruj docs/api.md: tabela endpointów z metodą, ścieżką, wymaganą autoryzacją, formatem żądania i odpowiedzi."*

Dokumentacja API to Twoja ocena z dokumentowania aplikacji. Agent ją wygeneruje w minutę – ale musi być **prawdziwa**, więc sprawdzasz każdy endpoint w Postmanie albo Thunder Client.

---

## Siedem zasad, które robią różnicę

### 1. Najpierw pytanie, potem polecenie

Źle: *„zrób logowanie JWT"*
Dobrze: *„jak w DRF działa uwierzytelnianie JWT? jakie pliki trzeba dotknąć i co robi każdy z nich?"* → czytasz → *„OK, zróbmy to"*

Ten jeden nawyk jest wart więcej niż cała reszta dokumentu.

### 2. Czytaj diff, zanim zatwierdzisz

Agent pokazuje, co zmieni. To nie jest formalność do przeklikania. Jeśli w diffie jest linia, której nie rozumiesz – pytaj **teraz**: *„co robi ta linia?"*, *„po co ten import?"*, *„czy bez tego zadziała?"*. Za tydzień nie będziesz pamiętał, że tam w ogóle coś jest.

### 3. Małe kroki, jeden commit na krok

Nie: *„zbuduj sklep"*.
Tak: *„model Produkt"* → commit → *„serializer i endpoint listy"* → commit → *„komponent listy w Reakcie"* → commit.

Historia commitów jest widoczna i mówi mi, jak pracowałeś. Jeden commit na 600 linii to sygnał alarmowy, nie osiągnięcie.

### 4. Każ sobie tłumaczyć na głos

Po większej zmianie: *„wyjaśnij, co zrobiłeś, tak żebym mógł to opowiedzieć nauczycielowi bez zaglądania w kod"*. To dosłownie symulacja tego, co Cię czeka na następnych zajęciach.

### 5. Raz w tygodniu napisz coś bez agenta

Wybierz jeden endpoint albo jeden komponent i napisz go od zera, sam, z dokumentacją Django/Reacta otwartą obok. Wolno. Z błędami. To jest trening dokładnie tej sytuacji, która czeka Cię w styczniu. Bez tego treningu wszystko inne jest bez znaczenia.

### 6. Nie dawaj mu sekretów

Nie wklejaj kluczy API, haseł do bazy ani tokenów. Trzymaj je w `.env`, a `.env` w `.gitignore`. W pliku `AGENTS.md` wpisz wprost, żeby tego pliku nie czytał.

### 7. Sprawdzaj, czy nie kłamie

Agent potrafi wymyślić metodę, której w bibliotece nie ma, albo wersję pakietu, która nie istnieje. Nazywa się to konfabulacją i zdarza się każdemu modelowi. Jeśli coś wygląda podejrzanie prosto – sprawdź w dokumentacji. **Uruchomienie kodu jest jedynym rozstrzygającym dowodem.**

---

## Co agent robi dobrze, a co źle

| Robi dobrze | Wypada słabo |
|---|---|
| tłumaczy błędy i stack trace | decyduje o architekturze projektu |
| pisze powtarzalny kod (CRUD, serializery) | rozumie, czego naprawdę chcesz z jednego zdania |
| generuje testy z podanych przypadków | wymyśla, co warto przetestować |
| przepisuje między technologiami | trzyma spójność przez wiele sesji |
| generuje dokumentację z kodu | wie, czego jeszcze nie wie |
| znajduje literówki i drobne błędy | odróżnia „działa" od „działa dobrze" |

Zauważ prawidłowość: prawa kolumna to same rzeczy, za które w firmie płaci się najwięcej. Twoja wartość na rynku jest w prawej kolumnie, nie w lewej.

---

## Zasady w tej klasie

1. Możesz używać agenta do wszystkiego, co robimy na zajęciach.
2. Na kolejnych zajęciach dostajesz pytania o kod. Wiesz, co robi każda linia – świetnie. Nie wiesz – do poprawy.
3. W repozytorium ma być `AGENTS.md` (lub `CLAUDE.md` / `GEMINI.md`) z Twoimi zasadami – nie samą wygenerowaną wersją.
4. W opisie PR zaznaczasz, co powstało z pomocą agenta. Nie po to, żeby obniżyć ocenę – po to, żeby wiedzieć, o co pytać.
5. Commit „dodałem wszystko" na 600 linii jest odrzucany bez czytania.
6. Raz w tygodniu jedno zadanie piszesz bez agenta. Zaznaczasz to w commicie: `feat(api): endpoint zamówień [bez AI]`.
7. Nie wklejasz do agenta treści zadań egzaminacyjnych CKE w trakcie próbnych – to jest trening warunków egzaminu, a nie zadanie do rozwiązania.

---

## Zadania

**Zadanie 1 – instalacja i pierwszy kontakt**
Zainstaluj wybranego agenta i uruchom go w repozytorium projektu. Poproś: *„opisz strukturę tego projektu, powiedz, co jest po stronie backendu, a co po stronie frontendu, i jak się komunikują"*. Wklej odpowiedź do `docs/agent-pierwszy-kontakt.md`. Oceń: co zgadło poprawnie, a gdzie się pomyliło?

**Zadanie 2 – plik projektowy**
Napisz własny `AGENTS.md` według wzoru wyżej. Ma mieć minimum 5 zasad technicznych i 5 dydaktycznych. Commit + push.

**Zadanie 3 – najpierw zrozumienie**
Wybierz jedną rzecz, której nie umiesz (CORS, JWT, serializery zagnieżdżone, `useEffect` z zależnościami). **Nie pozwól agentowi napisać kodu.** Zadaj minimum 5 pytań pogłębiających. Zapisz przebieg rozmowy w `docs/nauka-<temat>.md` i dopisz na końcu własne podsumowanie w 5 zdaniach – swoimi słowami, bez kopiowania.

**Zadanie 4 – porównanie kontraktu**
Zepsuj celowo nazwę jednego pola w serializerze. Uruchom frontend, zobacz `undefined`. Poproś agenta o porównanie serializera z komponentem – **bez naprawiania**. Sam napraw. Opisz w `docs/kontrakt-api.md`, dlaczego ten błąd nie dał żadnego komunikatu.

**Zadanie 5 – testy z Twoimi przypadkami**
Sam wypisz 6 przypadków testowych dla wybranego endpointu (poprawny, bez autoryzacji, złe dane, brakujące pole, nieistniejący zasób, granica). Dopiero potem: *„zaimplementuj te testy"*. Uruchom. Co najmniej jeden ma początkowo nie przechodzić – napraw kod, nie test.

**Zadanie 6 – bez agenta**
Napisz od zera, bez AI, z otwartą dokumentacją: model + serializer + endpoint + komponent, który go wyświetla. Zmierz czas. Commit z dopiskiem `[bez AI]`. W `docs/bez-ai.md` zapisz, co było najtrudniejsze i ile zajęło.

**Zadanie 7 – dokumentacja**
Wygeneruj `docs/api.md` z listą wszystkich endpointów. Potem **sprawdź każdy** w Thunder Client lub Postmanie i popraw to, co agent zmyślił. Zapisz, ile pozycji wymagało poprawki.