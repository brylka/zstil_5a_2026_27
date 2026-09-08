# Materiały – klasa 5 TP

**Technik programista | rok szkolny 2026/2027 | projekt fullstack**

Repozytorium z materiałami do zajęć. Kolejne pliki dochodzą **z zajęć na zajęcia** –
zaglądaj tu regularnie, `git pull` przed każdą lekcją.

---

## Co robimy w tym roku

Jeden projekt przez cały rok, budowany warstwa po warstwie. Zamiast osobnych bloków
„backend", „frontend", „testy" – jedna aplikacja, która z tygodnia na tydzień zyskuje
kolejny element. Punkt ciężkości: **łączenie backendu z frontendem**.

```
Django ---> REST API ---> React ---> Android ---> logowanie (JWT) ---> testy ---> wdrożenie
```

Kluczowy moment jest w środku: ten sam backend obsługuje **dwa różne klienty** – webowy
i mobilny. Nie piszemy API drugi raz na potrzeby telefonu, tylko podpinamy Androida pod
te same endpointy, z których korzysta React. Wtedy widać, po co API w ogóle istnieje.

Każdy wybiera własną domenę na początku roku (sklep, serwis rezerwacyjny, portal
ogłoszeniowy, coś swojego) i ciągnie ją do końca.

---

## Materiały

| Nr | Dokument | Temat |
|---|---|---|
| 01 | [Git i GitHub – wstęp](01-git-github-klasa5.md) | repozytorium fullstack, gałęzie, PR-y, konflikty, sekrety, CI |
| 02 | [Agenci LLM w projekcie fullstack](02-agenci-llm-klasa5.md) | Claude Code, Codex CLI, Gemini CLI, `AGENTS.md`, praca z agentem |

---

## Pozostałe technologie z podstawy INF.04

Podstawa w jednostce **INF.04.7 (programowanie aplikacji zaawansowanych webowych)**
wymienia wprost: **ASP.NET Core, Django, Angular, React.js, Node.js** oraz bibliotekę
**jQuery**. Nasz projekt roczny stoi na Django i Reakcie, ale pozostałych nie pomijamy –
robimy je jako lekcje porównawcze pod koniec roku, na zasadzie „ten sam CRUD, inna
składnia".

| Technologia | Jak ją traktujemy |
|---|---|
| **Node.js** | dostajecie ją po drodze przy Reakcie – `npm`, Vite, `package.json`. Osobno: Express i endpoint napisany po stronie serwera w JS |
| **Angular** | jedna–dwie lekcje porównawcze z Reactem: komponenty, TypeScript, `ng serve` na porcie 4200 |
| **ASP.NET Core** | przegląd: kontrolery, routing, jak wygląda ten sam endpoint w C# |
| **jQuery** | krótko i historycznie: manipulacja DOM przed Reactem, `$.ajax` jako przodek `fetch` |

Chodzi o rozpoznanie i zrozumienie różnic, nie o biegłość. Ale uwaga na egzaminie:
w zadaniu praktycznym część webowa bywa formułowana jako **„z zastosowaniem frameworka
Angular lub biblioteki React"** – więc Angulara trzeba przynajmniej umieć uruchomić
i przeczytać.

---

## Zasady

Pełne wersje w dokumentach 01 i 02, tutaj skrót:

1. **Jedno repozytorium na cały rok** – monorepo `backend/` + `frontend/`.
2. **Na `main` nie pushujesz.** Gałąź -> PR -> mój komentarz -> merge.
3. `.env` nigdy nie trafia do repozytorium, `.env.example` zawsze.
4. `README.md` Twojego projektu jest aktualny – zmienia się w tym samym PR co kod.
5. Na koniec zajęć: commit + push. Brak pusha = brak dowodu, że pracowałeś.
6. **Możesz używać LLM.** Ale na kolejnych zajęciach dostajesz pytania o kod.
   Wiesz, co robi każda linia – świetnie. Nie wiesz – do poprawy.
7. Co jakiś czas jedno zadanie piszesz bez AI, z oznaczeniem `[bez AI]` w commicie.
   W styczniu na INF.04 agenta nie będzie.

---

## Zanim zaczniemy

Sprawdź, czy masz:

```bash
git --version         # jeśli nie ma: https://git-scm.com/download/win
python --version      # 3.12+
node --version        # 20 LTS+
```

Konto na GitHubie z **rozsądną nazwą użytkownika** – to zobaczy pracodawca.
Warto od razu złożyć wniosek o [GitHub Student Developer Pack](https://education.github.com/pack)
(zaświadczenie ze szkoły do wzięcia w sekretariacie).