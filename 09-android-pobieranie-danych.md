# Android pobierający dane z API

**Technik programista | klasa 5 | materiał 09**

---

## Puenta całego kursu

Dziś Twój backend obsłuży **drugiego klienta**. Inny język, inne urządzenie, inna
technologia – i po stronie Django **nie zmieni się ani jedna linia kodu**.

Ten sam endpoint. Ten sam JSON. Zero pracy na serwerze.

```
                       ┌─────────────────┐
                       │   Django + API  │
                       │   /api/products/│
                       └────────┬────────┘
                    ┌───────────┴───────────┐
                    │                       │
            ┌───────┴───────┐       ┌───────┴───────┐
            │  React (web)  │       │Android (Java) │
            │   materiał 07 │       │   materiał 09 │
            └───────────────┘       └───────────────┘
```

Zatrzymaj się nad tym na chwilę, bo to jest odpowiedź na pytanie, które ciągnie się od
materiału 03: **po co było rozdzielać backend od frontendu?**

Gdyby Django generowało gotowe strony HTML – tak jak robi to klasyczna aplikacja
z szablonami – aplikacja na telefon wymagałaby zbudowania całego serwera od nowa. Albo
wciskania stron internetowych do widoku przeglądarki wewnątrz aplikacji, co wygląda
i działa źle.

Zamiast tego Twój backend wystawia **dane**, a każdy klient rysuje je po swojemu. Trzeci
i czwarty dołączą tak samo: aplikacja na iOS, panel dla pracowników, integracja
z hurtownią.

### Co się zmieni w aplikacji

Kasujesz `ProductData`. Wchodzi Retrofit odpytujący `/api/products/`.

**`ProductAdapter` i wszystkie układy XML zostają nietknięte.** Dokładnie tak jak
`ProductCard` w materiale 07.

Dochodzą za to trzy rzeczy, których przy danych w klasie nie było:
**uprawnienie, wątek i czas**.

### Wersje

| Biblioteka | Wersja |
|---|---|
| Retrofit | 3.0.0 |
| Converter Gson | 3.0.0 |

---

## Krok 1 – uprawnienie do internetu

W **`app/src/main/AndroidManifest.xml`**, przed znacznikiem `<application>`:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

Bez tego każde żądanie skończy się wyjątkiem o braku uprawnień.

To uprawnienie należy do tych, o które **nie trzeba pytać użytkownika** – system
przyznaje je przy instalacji. Inaczej jest z dostępem do lokalizacji, kamery czy
kontaktów: tam trzeba wyświetlić okno z pytaniem i obsłużyć odmowę. Android dzieli
uprawnienia na „zwykłe" i „groźne" właśnie według tego, czy mogą naruszyć prywatność.

---

## Krok 2 – biblioteki

W **`app/build.gradle.kts`**, w bloku `dependencies`:

```kotlin
implementation("com.squareup.retrofit2:retrofit:3.0.0")
implementation("com.squareup.retrofit2:converter-gson:3.0.0")
```

Kliknij **Sync Now** na pasku, który się pojawi.

### Co robi każda z nich

**Retrofit** zamienia opis endpointów – zwykły interfejs Javy – w działający klient HTTP.
Ty deklarujesz „pod tym adresem jest lista produktów", a Retrofit generuje kod, który
wykonuje żądanie, obsługuje wątki i zwraca wynik.

**Gson** zamienia JSON na obiekty Javy i odwrotnie. To on dopasowuje klucz `"name"`
z odpowiedzi serwera do pola `name` w klasie `Product`. Właśnie dlatego w materiale 08
nazwy pól musiały być identyczne z kluczami JSON-a – teraz to się opłaca.

**Converter Gson** to klej między nimi.

Sam Retrofit stoi na bibliotece **OkHttp**, która wykonuje faktyczne żądania. Nie
dopisujesz jej – wchodzi automatycznie jako zależność zależności, dokładnie tak jak
`asgiref` przy Django w materiale 03.

---

## Krok 3 – adres, czyli pierwsza niespodzianka

Twój serwer działa na `http://localhost:8000`. Wpisanie tego adresu w aplikacji
**nie zadziała**, i to z dwóch niezależnych powodów.

### Powód pierwszy: `localhost` telefonu to telefon

Dla emulatora i dla prawdziwego telefonu `localhost` oznacza **jego samego**, a nie
Twój komputer. Aplikacja szukałaby serwera wewnątrz urządzenia, na którym działa.

| Gdzie uruchamiasz | Adres backendu |
|---|---|
| Emulator Android Studio | `http://10.0.2.2:8000/api/` |
| Prawdziwy telefon w tej samej sieci Wi-Fi | `http://192.168.x.x:8000/api/` |

`10.0.2.2` to specjalny adres, pod którym emulator widzi komputer-gospodarza. Wspominałem
o nim w materiale 06.

Dla prawdziwego telefonu sprawdź adres komputera przez `ipconfig` i uruchom serwer tak,
żeby nasłuchiwał na wszystkich interfejsach:

```bash
python manage.py runserver 0.0.0.0:8000
```

Do `backend/.env` dopisz ten adres w `ALLOWED_HOSTS`:

```
ALLOWED_HOSTS=localhost,127.0.0.1,10.0.2.2,192.168.1.15
```

Bez tego Django odpowie `DisallowedHost`.

### Powód drugi: Android blokuje zwykły `http`

Od Androida 9 (API 28) system **domyślnie odrzuca połączenia nieszyfrowane**. Zobaczysz
wyjątek `CLEARTEXT communication to 10.0.2.2 not permitted by network security policy`.

To jest **mobilny odpowiednik CORS-u z materiału 06**: platforma blokuje coś, co wygląda
na poprawne, w imię bezpieczeństwa użytkownika. I znów rozwiązaniem nie jest wyłączenie
zabezpieczenia, a jego świadome zawężenie.

Utwórz **`res/xml/network_security_config.xml`** (katalog `xml` trzeba dodać):

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config cleartextTrafficPermitted="true">
        <domain includeSubdomains="false">10.0.2.2</domain>
        <domain includeSubdomains="false">192.168.1.15</domain>
    </domain-config>
</network-security-config>
```

I wskaż ten plik w `AndroidManifest.xml`, w znaczniku `<application>`:

```xml
<application
    android:networkSecurityConfig="@xml/network_security_config"
    ... >
```

Zwróć uwagę, co tu robimy: **wypisujemy konkretne adresy**, dla których wolno użyć
nieszyfrowanego połączenia. Kuszące `android:usesCleartextTraffic="true"` w znaczniku
`<application>` załatwiłoby sprawę jedną linią, ale otwarłoby całą aplikację na
połączenia bez szyfrowania – z każdym serwerem na świecie.

To dokładnie ta sama decyzja co `CORS_ALLOWED_ORIGINS` kontra `CORS_ALLOW_ALL_ORIGINS`.
Wersja wygodna kontra wersja bezpieczna. W projekcie produkcyjnym cały ten plik zniknie,
bo API będzie pod `https`.

---

## Krok 4 – opis endpointów

**`ApiService.java`**:

```java
package com.example.sklep;

import java.util.List;

import retrofit2.Call;
import retrofit2.http.GET;
import retrofit2.http.Path;
import retrofit2.http.Query;

public interface ApiService {

    @GET("products/")
    Call<List<Product>> getProducts(@Query("category") String category);

    @GET("products/{id}/")
    Call<Product> getProduct(@Path("id") int id);
}
```

To **interfejs** – sama deklaracja, bez ani jednej linii wykonywalnego kodu. Implementację
wygeneruje Retrofit.

| Element | Znaczenie |
|---|---|
| `@GET("products/")` | metoda i ścieżka dopisywana do adresu podstawowego |
| `Call<List<Product>>` | zapowiedź wyniku: lista produktów |
| `@Query("category")` | parametr w adresie: `?category=audio` |
| `@Path("id")` | wstawienie wartości w miejsce `{id}` w ścieżce |

**`Call<T>` to mobilny odpowiednik obietnicy z JavaScriptu.** Jeszcze nie wynik – tylko
zapowiedź, że wynik się pojawi. Sam obiekt `Call` nic nie wysyła; żądanie startuje
dopiero po wywołaniu `enqueue`.

Gdy do `getProducts` przekażesz `null`, Retrofit **pominie parametr** i wyśle czysty
adres `products/`. Wygodne: jedna metoda obsługuje listę pełną i filtrowaną.

Zwróć uwagę na ukośniki na końcu ścieżek. Django domyślnie wymaga adresów zakończonych
`/` – bez niego dostaniesz przekierowanie albo błąd 404.

---

## Krok 5 – klient

**`ApiClient.java`**:

```java
package com.example.sklep;

import retrofit2.Retrofit;
import retrofit2.converter.gson.GsonConverterFactory;

public final class ApiClient {

    private static final String BASE_URL = "http://10.0.2.2:8000/api/";

    private static ApiService service;

    private ApiClient() {
    }

    public static ApiService getService() {
        if (service == null) {
            Retrofit retrofit = new Retrofit.Builder()
                    .baseUrl(BASE_URL)
                    .addConverterFactory(GsonConverterFactory.create())
                    .build();

            service = retrofit.create(ApiService.class);
        }
        return service;
    }
}
```

### Trzy szczegóły

**Ukośnik na końcu `baseUrl` jest obowiązkowy.** Retrofit rzuci wyjątkiem, jeśli go
zabraknie – i słusznie, bo inaczej sklejanie adresów dawałoby dziwne wyniki.

**Jeden obiekt na całą aplikację.** Tworzenie `Retrofit` jest kosztowne, a pod spodem
siedzi pula połączeń, którą warto współdzielić. Sprawdzenie `if (service == null)`
sprawia, że powstaje raz. Ten wzorzec nazywa się **singletonem** i jest jednym z tych
z podstawy programowej INF.04, o których mówiliśmy przy programowaniu obiektowym.

**`retrofit.create(ApiService.class)`** to moment, w którym Retrofit generuje
implementację Twojego interfejsu. Nigdy nie piszesz klasy, która go implementuje.

Adres jest tu wpisany na sztywno, co w prawdziwym projekcie byłoby błędem – tak samo jak
byłoby nim wpisanie adresu API w kodzie Reacta. Odpowiednikiem `.env` jest w Androidzie
pole w `build.gradle.kts` (`buildConfigField`) i osobne warianty kompilacji. Zostawiam to
jako zadanie dodatkowe.

---

## Krok 6 – wątek, czyli druga niespodzianka

Android ma **jeden wątek odpowiedzialny za interfejs** – nazywany głównym albo wątkiem
UI. Rysuje on ekran i obsługuje dotknięcia. Jeśli go zablokujesz, aplikacja zamiera:
nic nie reaguje, a po kilku sekundach system pokazuje okno „aplikacja nie odpowiada".

Żądanie sieciowe trwa. Może trwać sekundę, może dziesięć. Dlatego **Android po prostu
zabrania** wykonywania go na głównym wątku – próba kończy się wyjątkiem
`NetworkOnMainThreadException`.

Retrofit rozwiązuje to metodą `enqueue`:

```java
ApiClient.getService().getProducts(null).enqueue(new Callback<List<Product>>() {

    @Override
    public void onResponse(Call<List<Product>> call, Response<List<Product>> response) {
        // wykonuje się na głównym wątku – można bezpiecznie zmieniać widoki
    }

    @Override
    public void onFailure(Call<List<Product>> call, Throwable t) {
        // nie udało się połączyć
    }
});
```

**`enqueue` wysyła żądanie w tle i wraca natychmiast.** Kod pod nim wykonuje się dalej,
nie czekając. Gdy odpowiedź przyjdzie, Retrofit wywołuje `onResponse` – i robi to
**już na głównym wątku**, więc wolno w nim od razu ustawiać teksty i widoczność widoków.
To duża wygoda; bez Retrofitu trzeba by samemu przeskakiwać między wątkami.

### `onResponse` nie znaczy „sukces"

Dokładnie ten sam problem, co z `fetch` w materiale 07:

```java
if (!response.isSuccessful()) {
    // serwer odpowiedział, ale statusem 404, 500...
}
```

`onResponse` wykonuje się **za każdym razem, gdy serwer odpowiedział** – także wtedy, gdy
odpowiedział błędem. `onFailure` dotyczy wyłącznie sytuacji, w której nie udało się
w ogóle połączyć: brak sieci, wyłączony serwer, zły adres.

| Sytuacja | Metoda | `isSuccessful()` |
|---|---|---|
| status 200, dane | `onResponse` | `true` |
| status 404 lub 500 | `onResponse` | `false` |
| brak sieci, serwer nie działa | `onFailure` | – |

`response.body()` przy nieudanym statusie bywa `null`. Sprawdzanie `isSuccessful()`
przed odczytem to nie ostrożność na wyrost, tylko warunek działania programu.

---

## Krok 7 – widoki na trzy stany

Tak jak w Reakcie potrzebne są trzy sytuacje: ładowanie, błąd, dane. Plus czwarta –
dane przyszły, ale są puste.

W **`res/layout/activity_main.xml`** dodaj pasek postępu i komunikat błędu, nad listą:

```xml
<ProgressBar
    android:id="@+id/loading"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="center_horizontal"
    android:padding="24dp"
    android:visibility="gone" />

<TextView
    android:id="@+id/errorMessage"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="16dp"
    android:textColor="#B00020"
    android:visibility="gone" />

<Button
    android:id="@+id/retryButton"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="center_horizontal"
    android:text="Spróbuj ponownie"
    android:visibility="gone" />
```

`android:visibility="gone"` znaczy „niewidoczny i nie zajmuje miejsca". Jest jeszcze
`invisible` – niewidoczny, ale miejsce zostaje puste. Do stanów używa się `gone`.

---

## Krok 8 – nowa `MainActivity`

```java
package com.example.sklep;

import android.content.Intent;
import android.os.Bundle;
import android.text.Editable;
import android.text.TextWatcher;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.ProgressBar;
import android.widget.TextView;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import java.util.ArrayList;
import java.util.List;
import java.util.Locale;

import retrofit2.Call;
import retrofit2.Callback;
import retrofit2.Response;

public class MainActivity extends AppCompatActivity {

    private final List<Product> allProducts = new ArrayList<>();
    private final List<Product> visible = new ArrayList<>();

    private ProductAdapter adapter;
    private ProgressBar loading;
    private TextView errorMessage;
    private TextView emptyMessage;
    private Button retryButton;
    private EditText searchField;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        loading = findViewById(R.id.loading);
        errorMessage = findViewById(R.id.errorMessage);
        emptyMessage = findViewById(R.id.emptyMessage);
        retryButton = findViewById(R.id.retryButton);
        searchField = findViewById(R.id.searchField);

        RecyclerView productList = findViewById(R.id.productList);
        productList.setLayoutManager(new LinearLayoutManager(this));

        adapter = new ProductAdapter(visible, this::openDetails);
        productList.setAdapter(adapter);

        searchField.addTextChangedListener(new TextWatcher() {
            @Override
            public void beforeTextChanged(CharSequence s, int st, int c, int a) {
            }

            @Override
            public void onTextChanged(CharSequence s, int st, int b, int c) {
                filter(s.toString());
            }

            @Override
            public void afterTextChanged(Editable s) {
            }
        });

        retryButton.setOnClickListener(view -> loadProducts());

        loadProducts();
    }

    private void loadProducts() {
        loading.setVisibility(View.VISIBLE);
        errorMessage.setVisibility(View.GONE);
        retryButton.setVisibility(View.GONE);
        emptyMessage.setVisibility(View.GONE);

        ApiClient.getService().getProducts(null).enqueue(new Callback<List<Product>>() {

            @Override
            public void onResponse(@NonNull Call<List<Product>> call,
                                   @NonNull Response<List<Product>> response) {
                loading.setVisibility(View.GONE);

                if (!response.isSuccessful() || response.body() == null) {
                    showError("Serwer odpowiedział błędem " + response.code());
                    return;
                }

                allProducts.clear();
                allProducts.addAll(response.body());
                filter(searchField.getText().toString());
            }

            @Override
            public void onFailure(@NonNull Call<List<Product>> call, @NonNull Throwable t) {
                loading.setVisibility(View.GONE);
                showError("Nie udało się połączyć z serwerem");
            }
        });
    }

    private void showError(String text) {
        errorMessage.setText(text);
        errorMessage.setVisibility(View.VISIBLE);
        retryButton.setVisibility(View.VISIBLE);
    }

    private void filter(String query) {
        String needle = query.toLowerCase(Locale.ROOT);

        visible.clear();
        for (Product product : allProducts) {
            if (product.getName().toLowerCase(Locale.ROOT).contains(needle)) {
                visible.add(product);
            }
        }

        adapter.notifyDataSetChanged();
        emptyMessage.setVisibility(visible.isEmpty() ? View.VISIBLE : View.GONE);
    }

    private void openDetails(Product product) {
        Intent intent = new Intent(this, ProductDetailActivity.class);
        intent.putExtra("productId", product.getId());
        startActivity(intent);
    }
}
```

Uruchom przy działającym backendzie. Lista wygląda tak samo jak w materiale 08 – tylko
dane płyną teraz z Django.

### Dwie listy, nie jedna

`allProducts` trzyma wszystko, co przyszło z serwera. `visible` zawiera to, co widać po
odfiltrowaniu. Bez tego rozdziału wpisanie litery zniszczyłoby pobrane dane i trzeba by
odpytywać serwer przy każdym znaku.

Dokładnie tak samo działało to w Reakcie: `products` ze stanu i wyliczane z nich
`visible`.

### `filter` zamiast bezpośredniego przypisania

Po pobraniu danych wołamy `filter(...)` z aktualną treścią pola wyszukiwania, a nie
wprost `visible.addAll(...)`. Powód: użytkownik mógł zacząć pisać, **zanim** odpowiedź
dotarła. Gdybyśmy pokazali wszystko, jego wyszukiwanie zostałoby zignorowane.

---

## Krok 9 – szczegóły z serwera

**`ProductDetailActivity.java`**:

```java
package com.example.sklep;

import android.os.Bundle;
import android.widget.TextView;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;

import retrofit2.Call;
import retrofit2.Callback;
import retrofit2.Response;

public class ProductDetailActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_product_detail);

        int productId = getIntent().getIntExtra("productId", -1);

        ApiClient.getService().getProduct(productId).enqueue(new Callback<Product>() {

            @Override
            public void onResponse(@NonNull Call<Product> call,
                                   @NonNull Response<Product> response) {
                if (response.code() == 404) {
                    Toast.makeText(ProductDetailActivity.this,
                            "Nie ma takiego produktu", Toast.LENGTH_SHORT).show();
                    finish();
                    return;
                }

                if (!response.isSuccessful() || response.body() == null) {
                    Toast.makeText(ProductDetailActivity.this,
                            "Błąd " + response.code(), Toast.LENGTH_SHORT).show();
                    finish();
                    return;
                }

                show(response.body());
            }

            @Override
            public void onFailure(@NonNull Call<Product> call, @NonNull Throwable t) {
                Toast.makeText(ProductDetailActivity.this,
                        "Brak połączenia", Toast.LENGTH_SHORT).show();
                finish();
            }
        });
    }

    private void show(Product product) {
        setTitle(product.getName());

        ((TextView) findViewById(R.id.detailName)).setText(product.getName());
        ((TextView) findViewById(R.id.detailDescription)).setText(product.getDescription());
        ((TextView) findViewById(R.id.detailPrice)).setText(Format.price(product.getPrice()));
        ((TextView) findViewById(R.id.detailStock))
                .setText("Na stanie: " + product.getStock() + " szt.");
    }
}
```

**404 obsłużone osobno**, przed sprawdzeniem ogólnym – tak samo jak w materiale 07.
Użytkownik ma zobaczyć „nie ma takiego produktu", a nie numer statusu.

Zwróć uwagę, że ten ekran **pobiera dane sam**, a nie dostaje ich od listy. Dostaje
wyłącznie identyfikator. Dzięki temu można go otworzyć z powiadomienia albo z odsyłacza –
lista nie musi w ogóle istnieć.

---

## Krok 10 – dlaczego nazwy pól były takie ważne

Uruchom aplikację i sprawdź, czy wszystkie pola się wypełniły.

Jeśli któreś jest puste albo zerowe, prawie zawsze znaczy to jedno: **nazwa pola w klasie
nie zgadza się z kluczem w JSON-ie**. Gson dopasowuje je po nazwie, a gdy nie znajdzie
odpowiednika, zostawia pole niewypełnione – bez żadnego błędu i bez ostrzeżenia. To jest
podstępne, bo aplikacja działa, tylko część danych nie dociera.

Gdyby nazwy musiały się różnić – bo API zwraca `product_name`, a Ty chcesz mieć pole
`name` – używa się adnotacji:

```java
@SerializedName("product_name")
private final String name;
```

Nam nie jest potrzebna, bo w materiale 08 nazwy dobraliśmy pod JSON-a z rozmysłem.

### Pole `created_at`

Odpowiedź serwera zawiera datę, której model nie ma. **Gson po prostu ją zignoruje** –
nadmiarowe klucze nie przeszkadzają. To ważna właściwość: serwer może dodać nowe pole,
a stara wersja aplikacji będzie dalej działać.

Zasada nazywa się **tolerancyjnym czytaniem** i jest podstawą tego, że aplikacje mobilne
nie psują się przy każdej aktualizacji API. Odwrotnie już tak dobrze nie jest: gdy serwer
**usunie** albo **przemianuje** pole, stare aplikacje przestają pokazywać dane. Dlatego
w materiale 01 zmiana nazwy pola w API była zmianą „główną" w wersjonowaniu
semantycznym.

---

## Krok 11 – sprzątanie i zapis

```bash
git rm android/app/src/main/java/com/example/sklep/ProductData.java
```

Usuń też metodę `findById`, jeśli została gdzieś wywołana.

```bash
git status --short          # NIE może tu być build/ ani local.properties
git add .
git commit -m "feat(android): dane pobierane z API zamiast z klasy"
```

Podział na commity:

```
feat(android): uprawnienie INTERNET i biblioteki Retrofit
feat(android): konfiguracja bezpieczeństwa sieci dla adresów lokalnych
feat(android): interfejs ApiService i klient Retrofit
feat(android): lista produktów pobierana z API
feat(android): obsługa stanów ładowania, błędu i ponowienia
feat(android): szczegóły produktu z osobnego endpointu
chore(android): usunięcie danych statycznych
```

W `README.md` dopisz, że aplikacja mobilna wymaga **działającego backendu**, i podaj
oba adresy – dla emulatora i dla telefonu w sieci lokalnej. Bez tej informacji nikt tego
nie uruchomi.

---

## Sprawdź, czy rozumiesz

1. Ile linii kodu trzeba było zmienić w Django, żeby obsłużyć aplikację mobilną?
2. Dlaczego adres `localhost` nie działa w emulatorze i co wpisujemy zamiast niego?
3. Co blokuje połączenia `http` od Androida 9 i dlaczego nie wyłączamy tego globalnie?
4. Czym jest `ApiService`, jeśli nie ma w nim ani jednej linii kodu wykonywalnego?
5. Dlaczego żądania sieciowego nie wolno wykonać na głównym wątku?
6. Kiedy wykonuje się `onResponse`, a kiedy `onFailure`?
7. Dlaczego trzeba sprawdzać `isSuccessful()`, skoro jesteśmy w `onResponse`?
8. Po co dwie listy: `allProducts` i `visible`?
9. Co zrobi Gson z polem `created_at`, którego nie ma w klasie `Product`?
10. Co się stanie, jeśli nazwa pola w klasie nie będzie zgodna z kluczem JSON-a?

---

## Częste błędy

| Objaw | Przyczyna |
|---|---|
| `SecurityException: Permission denied` | brak `INTERNET` w manifeście |
| `CLEARTEXT communication not permitted` | brak konfiguracji bezpieczeństwa sieci |
| `Failed to connect to /10.0.2.2:8000` | backend nie działa albo zły port |
| `DisallowedHost` w logach Django | brak adresu w `ALLOWED_HOSTS` |
| `IllegalArgumentException: baseUrl must end in /` | brak ukośnika na końcu adresu podstawowego |
| `NetworkOnMainThreadException` | użyto `execute()` zamiast `enqueue()` |
| część pól puste, reszta wypełniona | niezgodność nazw pól z kluczami JSON |
| `NullPointerException` na `response.body()` | brak sprawdzenia `isSuccessful()` |
| lista pusta, brak błędu | serwer zwrócił pustą tablicę – sprawdź bazę i `loaddata` |
| telefon nie widzi serwera | `runserver` bez `0.0.0.0` albo inna sieć Wi-Fi |
| 404 na poprawnym adresie | brak ukośnika na końcu ścieżki w `@GET` |

### Gdzie patrzeć, gdy nie działa

W Android Studio otwórz **Logcat** i filtruj po nazwie pakietu. Tam trafiają wszystkie
wyjątki – to odpowiednik konsoli przeglądarki z materiału 07.

Drugie miejsce to **terminal z Django**. Każde żądanie zostawia tam wpis ze statusem.
Jeśli nie ma żadnego wpisu, żądanie w ogóle nie dotarło – problem jest po stronie adresu
albo sieci, nie kodu serwera.

---

## Czego jeszcze nie ma

- **Tylko odczyt.** Dalej wyłącznie GET, w obu klientach.
- **Obsługi obrotu ekranu.** Po obrocie aktywność powstaje od nowa i dane pobierają się
  ponownie. Rozwiązuje to `ViewModel`.
- **Filtrowania po kategorii.** Endpoint to obsługuje, `ApiService` też – brakuje tylko
  elementu interfejsu. Zadanie.
- **Pamięci podręcznej i pracy bez sieci.** Przy braku połączenia aplikacja nie pokazuje
  nic.
- **Adresu w konfiguracji.** Wpisany na sztywno w kodzie.
- **Logowania.** API jest publiczne dla wszystkich klientów.

---

## Co dalej

Zamyka się etap „tylko odczyt". Backend obsługuje dwóch klientów, oba wyłącznie czytają.

Dalej wchodzi **zapis**: Django REST Framework, który weźmie na siebie ręczną
serializację z materiału 04, a potem metody POST, PUT i DELETE wraz z walidacją danych
wejściowych i logowaniem. Wtedy też zobaczysz żądania wstępne `OPTIONS`, o których
pisałem w materiale 06.

---

# Zadania

**Zadanie 1 – połączenie**
Uprawnienie, biblioteki, konfiguracja bezpieczeństwa sieci. W `docs/android.md` zapisz,
jakiego adresu użyłeś i dlaczego nie `localhost`.

**Zadanie 2 – opis endpointów**
Interfejs `ApiService` z metodami: lista, lista filtrowana parametrem, szczegóły po
identyfikatorze. Ścieżki zgodne z Twoim API.

**Zadanie 3 – klient**
Klasa klienta w postaci singletonu. W `docs/android.md` wyjaśnij jednym akapitem,
dlaczego obiekt Retrofit tworzymy raz, a nie przy każdym żądaniu.

**Zadanie 4 – podmiana źródła**
Usuń klasę z danymi statycznymi. Lista ma się wyświetlać z serwera. **Udowodnij, że
adapter nie wymagał zmian** – wklej jego kod przed i po do `docs/android.md`.

**Zadanie 5 – trzy stany**
Pasek postępu, komunikat błędu z przyciskiem ponowienia, komunikat przy pustych wynikach.
Każdy po polsku i zrozumiale dla użytkownika.

**Zadanie 6 – dowody**
Trzy zrzuty ekranu: ładowanie, błąd przy wyłączonym backendzie, poprawna lista.
Do `docs/android.md`.

**Zadanie 7 – szczegóły**
Ekran szczegółów pobiera dane osobnym żądaniem po identyfikatorze. 404 obsłużone
osobnym komunikatem – sprawdź, podając w kodzie nieistniejący identyfikator.

**Zadanie 8 – filtr kategorii**
Dodaj element `Spinner` z kategoriami i filtruj **po stronie serwera**, przez parametr
w adresie. W terminalu Django pokaż wpisy z parametrem.

**Zadanie 9 – test niezgodności**
Zmień celowo nazwę jednego pola w klasie modelu i uruchom aplikację. Opisz
w `docs/android.md`, co się stało, dlaczego nie było żadnego błędu i jak naprawić to
adnotacją `@SerializedName`. Potem przywróć poprawną nazwę.

**Zadanie 10 – podsumowanie architektury**
W `docs/architektura.md` narysuj (choćby znakami tekstu) swój system: backend, klient
webowy, klient mobilny, endpointy. Odpowiedz na dwa pytania: ile razy musiałeś zmienić
kod backendu, dodając drugiego klienta, i co trzeba byłoby zrobić, gdyby Django
generowało strony HTML zamiast JSON-a.

To zadanie jest najważniejsze w całym materiale. Reszta to technika – to jest zrozumienie,
po co ta technika istnieje.

---

**Zadanie dodatkowe (dla chętnych)**
Wyprowadź adres API z kodu do konfiguracji budowania. W `app/build.gradle.kts` użyj
`buildConfigField` i odczytaj wartość przez `BuildConfig`. Ambitniej: dwa warianty
kompilacji (`debug` z adresem lokalnym, `release` z produkcyjnym), żeby nie podmieniać
adresu ręcznie przed każdym wydaniem. W `docs/android.md` porównaj to rozwiązanie
z plikiem `.env` z materiału 06 – co jest podobne, a co inne, i dlaczego w aplikacji
mobilnej nie da się „ukryć" adresu ani klucza.