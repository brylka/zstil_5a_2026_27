# Android w Javie – wstęp

**Technik programista | klasa 5 | materiał 08**

---

## Co zbudujemy

Aplikację mobilną wyświetlającą listę produktów: przewijana lista kart, wyszukiwarka
i osobny ekran szczegółów po dotknięciu pozycji.

**Dane będą wpisane na sztywno**, w klasie `ProductData`. Backendu nie uruchamiamy.

### Trzeci raz ten sam schemat

Rozpoznajesz go już na pewno:

```
03: API na liście w pamięci  ---> 04: API na bazie
05: React na pliku           ---> 07: React na API
08: Android na klasie        ---> 09: Android na API
```

Za każdym razem najpierw budujemy działający szkielet na atrapie danych, potem
podmieniamy źródło. Za każdym razem ta podmiana okazuje się mniejsza, niż się wydawało.

Ale tym razem dochodzi coś jeszcze ważniejszego. W materiale 09 Twój backend obsłuży
**drugiego klienta** – zupełnie inną technologię, inny język, inne urządzenie – i nie
zmieni się w nim **ani jedna linia kodu**. Endpoint `/api/products/` jest ten sam.
Odpowiedź JSON jest ta sama.

To jest ostateczny dowód na to, po co w ogóle rozdzielaliśmy backend od frontendu.
Gdyby Django generowało gotowe strony HTML, aplikacja na telefon wymagałaby napisania
całego serwera od nowa.

### Dlaczego Java, a nie Kotlin

Bo tak stanowi podstawa programowa INF.04 i bo mieliście Javę w czwartej klasie.

Warto jednak wiedzieć, jak jest naprawdę: **Google od 2019 roku rekomenduje Kotlin
jako język pierwszego wyboru** dla Androida, nowe biblioteki i dokumentacja powstają
przede wszystkim z myślą o nim, a Jetpack Compose – nowoczesny sposób budowania
interfejsów – w praktyce wymaga Kotlina.

Java na Androidzie działa, jest w pełni wspierana i miliony aplikacji w sklepie są
w niej napisane. Ale jeśli po szkole pójdziesz w mobilki, Kotlina nauczysz się w tydzień –
i będziesz musiał. Traktuj ten materiał jako naukę **mechanizmów Androida**: aktywności,
układów, adapterów i cyklu życia. One są identyczne w obu językach.

### Wersje

| | Ustawienie |
|---|---|
| Android Studio | najnowsza stabilna wersja |
| Język | Java |
| minSdk | 26 (Android 8.0) |
| targetSdk / compileSdk | 36 (Android 16) |

Najnowszy Android to obecnie **17 (API 37, „Cinnamon Bun")**, ale Google Play wymaga
`targetSdk` co najmniej **36** dla nowych aplikacji i aktualizacji od 31 sierpnia 2026 –
i to jest rozsądna wartość na zajęcia.

`minSdk 26` oznacza, że aplikacja zainstaluje się na urządzeniach z Androidem 8 i nowszym,
czyli na praktycznie wszystkich będących dziś w użyciu. Im niższa ta wartość, tym więcej
urządzeń obsłużysz – i tym więcej starych ograniczeń musisz omijać w kodzie.

---

## Krok 0 – skąd wziąć dane

Tak jak w materiale 05: uruchom na chwilę backend, wejdź na
`http://localhost:8000/api/products/` i skopiuj odpowiedź. Będzie potrzebna w kroku 5.

---

## Krok 1 – gdzie umieścić projekt

Aplikacja mobilna jest **trzecią częścią tego samego projektu**, więc trafia do tego
samego repozytorium:

```
sklep-fullstack/
├── backend/
├── frontend/
└── android/        <-- tutaj
```

W Android Studio: **New Project → Empty Views Activity** i dalej:

| Pole | Wartość |
|---|---|
| Name | `Sklep` |
| Package name | `com.example.sklep` |
| Save location | `...\sklep-fullstack\android` |
| Language | **Java** |
| Minimum SDK | API 26 |
| Build configuration language | Kotlin DSL (domyślne) |

> Uwaga na szablon. **Empty Views Activity** to klasyczne układy w XML – tego chcemy.
> Sam „Empty Activity" to szablon dla Jetpack Compose, dostępny tylko dla Kotlina.
> Nazwy są mylące i łatwo kliknąć nie to, co trzeba.

Pierwsze otwarcie projektu potrwa – Gradle pobiera zależności. Poczekaj, aż pasek na dole
przestanie się ruszać.

---

## Krok 2 – co jest w projekcie

```
android/
├── app/
│   ├── build.gradle.kts          ← zależności i ustawienia modułu
│   └── src/main/
│       ├── AndroidManifest.xml   ← spis ekranów i uprawnień
│       ├── java/com/example/sklep/
│       │   └── MainActivity.java
│       └── res/
│           ├── layout/
│           │   └── activity_main.xml
│           ├── values/
│           │   ├── strings.xml
│           │   └── themes.xml
│           └── mipmap/           ← ikony aplikacji
├── build.gradle.kts              ← ustawienia całego projektu
├── settings.gradle.kts
├── gradle/libs.versions.toml     ← katalog wersji bibliotek
└── local.properties              ← ścieżka do SDK, NIE do repozytorium
```

Trzy rzeczy, które warto zrozumieć od razu.

**`res/` to zasoby, a nie kod.** Układy ekranów, teksty, kolory i ikony leżą w osobnych
plikach XML. Android generuje z nich klasę `R`, przez którą sięgasz do nich w kodzie:
`R.layout.activity_main`, `R.id.productList`. Jeśli `R` nagle podkreśla się na czerwono,
to zwykle znaczy, że któryś plik XML ma błąd składni – napraw XML, a `R` się odbuduje.

**`AndroidManifest.xml` to spis treści aplikacji.** Każdy ekran musi tam być zgłoszony,
inaczej system odmówi jego uruchomienia. Tam też deklaruje się uprawnienia – w materiale
09 dopiszemy dostęp do internetu.

**Gradle to system budowania**, odpowiednik `requirements.txt` i `package.json` razem
wziętych. Zależności dopisuje się w `app/build.gradle.kts`, a wersje bibliotek trzyma
w `gradle/libs.versions.toml`. Po każdej zmianie Android Studio wyświetla pasek
z przyciskiem **Sync Now** – trzeba go kliknąć.

---

## Krok 3 – na czym to uruchomić

**Emulator:** Device Manager (ikona telefonu po prawej) → Create Virtual Device →
dowolny Pixel → obraz systemu z API 36. Emulator wymaga włączonej wirtualizacji
w BIOS-ie; jeśli działa wyjątkowo wolno, to najczęściej właśnie dlatego.

**Prawdziwy telefon** jest szybszy i wygodniejszy. W telefonie: Ustawienia →
Informacje o telefonie → siedmiokrotne dotknięcie „Numer kompilacji" → wraca się do
ustawień, wchodzi w Opcje programisty i włącza **Debugowanie USB**. Po podłączeniu
kablem telefon zapyta o zgodę na debugowanie z tego komputera.

Kliknij zielony trójkąt (Run). Powinno pojawić się „Hello World".

---

## Krok 4 – model danych

Nowy plik **`Product.java`** (prawy klawisz na pakiecie → New → Java Class):

```java
package com.example.sklep;

public class Product {

    private final int id;
    private final String name;
    private final String description;
    private final String price;
    private final int stock;
    private final String category;

    public Product(int id, String name, String description,
                   String price, int stock, String category) {
        this.id = id;
        this.name = name;
        this.description = description;
        this.price = price;
        this.stock = stock;
        this.category = category;
    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getDescription() {
        return description;
    }

    public String getPrice() {
        return price;
    }

    public int getStock() {
        return stock;
    }

    public String getCategory() {
        return category;
    }
}
```

### Dwie decyzje, które nie są przypadkowe

**Nazwy pól są identyczne jak klucze w JSON-ie.** `name`, `price`, `stock`, `category` –
dokładnie tak, jak zwraca Twoje API. W materiale 09 biblioteka Gson wypełni te pola
automatycznie właśnie na podstawie zgodności nazw. Gdybyś nazwał pole `nazwa`, trzeba by
to potem ręcznie mapować.

**`price` jest typu `String`, nie `double`.** Bo tak przychodzi z serwera – pamiętasz
`DecimalField` i `str(product.price)` z materiału 04. Model ma odwzorowywać to, co
naprawdę przychodzi, a nie to, co wygodniejsze. Konwersją zajmiemy się przy wyświetlaniu.

**`final` przy polach** oznacza, że po utworzeniu obiektu nie da się ich zmienić. Obiekt
niemodyfikowalny jest bezpieczniejszy – nikt go przypadkiem nie popsuje w innym miejscu
programu, a przy wielu wątkach (a z siecią zawsze są wątki) to ma realne znaczenie.

---

## Krok 5 – dane

**`ProductData.java`** – wklej tu rekordy skopiowane z własnego API:

```java
package com.example.sklep;

import java.util.Arrays;
import java.util.List;

public final class ProductData {

    private ProductData() {
        // klasa tylko na dane, nie tworzymy jej obiektów
    }

    public static final List<Product> PRODUCTS = Arrays.asList(
            new Product(1, "Klawiatura mechaniczna K80",
                    "Przełączniki brązowe, układ TKL.", "349.00", 12, "peripherals"),
            new Product(2, "Mysz bezprzewodowa Swift",
                    "Sensor 16000 DPI, łączność 2.4 GHz.", "129.99", 40, "peripherals"),
            new Product(3, "Słuchawki nauszne Cliff",
                    "Redukcja szumów, 30 h pracy.", "289.00", 0, "audio")
    );

    public static Product findById(int id) {
        for (Product product : PRODUCTS) {
            if (product.getId() == id) {
                return product;
            }
        }
        return null;
    }
}
```

Metoda `findById` przyda się na ekranie szczegółów. W materiale 09 zniknie – zastąpi ją
zapytanie do serwera.

---

## Krok 6 – formatowanie ceny

**`Format.java`**:

```java
package com.example.sklep;

import java.text.NumberFormat;
import java.util.Locale;

public final class Format {

    private Format() {
    }

    public static String price(String value) {
        NumberFormat formatter =
                NumberFormat.getCurrencyInstance(Locale.forLanguageTag("pl-PL"));
        return formatter.format(Double.parseDouble(value));
    }
}
```

`Double.parseDouble` zamienia tekst `"349.00"` na liczbę, a `NumberFormat` formatuje ją
zgodnie z polskimi zasadami: `349,00 zł`, z przecinkiem i spacją przed walutą.

To jest dokładny odpowiednik `Intl.NumberFormat` z materiału 05. Ta sama potrzeba, inne
narzędzie – warto zauważyć, że umiejętności przenoszą się między technologiami znacznie
lepiej niż konkretne nazwy funkcji.

---

## Krok 7 – wygląd pojedynczej pozycji

Prawy klawisz na `res/layout` → New → Layout Resource File → nazwa `item_product`,
element główny `LinearLayout`.

**`res/layout/item_product.xml`**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="16dp"
    android:background="?android:attr/selectableItemBackground">

    <TextView
        android:id="@+id/productName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="16sp"
        android:textStyle="bold"
        tools:text="Klawiatura mechaniczna K80" />

    <TextView
        android:id="@+id/productCategory"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="13sp"
        tools:text="peripherals" />

    <TextView
        android:id="@+id/productPrice"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="18sp"
        android:textStyle="bold"
        tools:text="349,00 zł" />

    <TextView
        android:id="@+id/productStock"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="13sp"
        tools:text="Na stanie: 12 szt." />

</LinearLayout>
```

Aby działało `tools:text`, w znaczniku `LinearLayout` musi być jeszcze:

```
xmlns:tools="http://schemas.android.com/tools"
```

### Warto wiedzieć

**`dp` i `sp`.** `dp` to jednostka niezależna od gęstości ekranu – 16dp wygląda tak samo
na tanim telefonie i na tablecie. `sp` działa podobnie, ale dodatkowo skaluje się
z ustawieniem rozmiaru czcionki w systemie, dlatego **teksty zawsze podaje się w `sp`**.
To kwestia dostępności: ktoś słabo widzący ma powiększoną czcionkę i Twoja aplikacja
ma to uszanować.

**`tools:text`** widać wyłącznie w podglądzie w Android Studio. Nie trafia do działającej
aplikacji. Służy do tego, żeby projektując układ, nie patrzeć na puste prostokąty.

**`?android:attr/selectableItemBackground`** daje efekt podświetlenia przy dotknięciu.
Drobiazg, ale bez niego lista sprawia wrażenie zepsutej.

---

## Krok 8 – ekran główny

**`res/layout/activity_main.xml`** – zastąp zawartość:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <EditText
        android:id="@+id/searchField"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Szukaj produktu..."
        android:inputType="text"
        android:autofillHints="" />

    <TextView
        android:id="@+id/emptyMessage"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="16dp"
        android:text="Nic nie pasuje do wyszukiwania."
        android:visibility="gone" />

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/productList"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</LinearLayout>
```

Jeśli Android Studio nie rozpoznaje `RecyclerView`, dopisz zależność w
`app/build.gradle.kts` i kliknij **Sync Now**:

```kotlin
implementation("androidx.recyclerview:recyclerview:1.4.0")
```

(Zwykle nie trzeba – wciąga ją biblioteka Material, którą szablon dodaje sam.)

---

## Krok 9 – adapter

To jest centralny element całego materiału. **`ProductAdapter.java`**:

```java
package com.example.sklep;

import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;

import java.util.List;

public class ProductAdapter extends RecyclerView.Adapter<ProductAdapter.ProductViewHolder> {

    public interface OnProductClickListener {
        void onProductClick(Product product);
    }

    private final List<Product> products;
    private final OnProductClickListener listener;

    public ProductAdapter(List<Product> products, OnProductClickListener listener) {
        this.products = products;
        this.listener = listener;
    }

    @NonNull
    @Override
    public ProductViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext())
                .inflate(R.layout.item_product, parent, false);
        return new ProductViewHolder(view);
    }

    @Override
    public void onBindViewHolder(@NonNull ProductViewHolder holder, int position) {
        holder.bind(products.get(position), listener);
    }

    @Override
    public int getItemCount() {
        return products.size();
    }

    static class ProductViewHolder extends RecyclerView.ViewHolder {

        private final TextView nameView;
        private final TextView categoryView;
        private final TextView priceView;
        private final TextView stockView;

        ProductViewHolder(@NonNull View itemView) {
            super(itemView);
            nameView = itemView.findViewById(R.id.productName);
            categoryView = itemView.findViewById(R.id.productCategory);
            priceView = itemView.findViewById(R.id.productPrice);
            stockView = itemView.findViewById(R.id.productStock);
        }

        void bind(Product product, OnProductClickListener listener) {
            nameView.setText(product.getName());
            categoryView.setText(product.getCategory());
            priceView.setText(Format.price(product.getPrice()));

            if (product.getStock() == 0) {
                stockView.setText("Niedostępny");
            } else {
                stockView.setText("Na stanie: " + product.getStock() + " szt.");
            }

            itemView.setOnClickListener(view -> listener.onProductClick(product));
        }
    }
}
```

### Po co ta cała konstrukcja

Lista w telefonie może mieć tysiące pozycji, a na ekranie mieści się sześć.
`RecyclerView` tworzy **tylko tyle widoków, ile widać**, plus kilka zapasowych – i przy
przewijaniu **używa ich ponownie**, wypełniając innymi danymi. Stąd nazwa: *recycler*,
czyli coś, co odzyskuje i przetwarza.

Trzy metody odpowiadają trzem pytaniom:

| Metoda | Pytanie |
|---|---|
| `onCreateViewHolder` | jak zbudować **pusty** wiersz? (rzadko, kilkanaście razy) |
| `onBindViewHolder` | jak wstawić dane do **istniejącego** wiersza? (przy każdym przewinięciu) |
| `getItemCount` | ile jest pozycji? |

**`ViewHolder` istnieje po to, żeby nie wywoływać `findViewById` w kółko.** To operacja
kosztowna – przeszukuje drzewo widoków. Gdyby wywoływać ją przy każdym przewinięciu, dla
każdego pola, lista zaczęłaby się szarpać. `ViewHolder` znajduje pola **raz**, przy
tworzeniu wiersza, i trzyma do nich odniesienia.

**`inflate`** to zamiana pliku XML na prawdziwe obiekty widoków w pamięci. Argument
`false` oznacza „nie dołączaj jeszcze do rodzica" – zrobi to `RecyclerView` sam. Podanie
tam `true` to klasyczny błąd, który kończy się wyjątkiem o tym, że widok ma już rodzica.

**Własny interfejs `OnProductClickListener`** to dokładny odpowiednik przekazania funkcji
przez props w Reakcie (`onSelect` z materiału 05). Adapter nie wie, co ma się stać po
dotknięciu – tylko zgłasza zdarzenie temu, kto go utworzył. Java nie ma funkcji jako
wartości, więc rolę tę pełni jednometodowy interfejs. Dzięki temu można go przekazać
jako wyrażenie lambda.

---

## Krok 10 – złożenie całości

**`MainActivity.java`**:

```java
package com.example.sklep;

import android.content.Intent;
import android.os.Bundle;
import android.text.Editable;
import android.text.TextWatcher;
import android.view.View;
import android.widget.EditText;
import android.widget.TextView;

import androidx.appcompat.app.AppCompatActivity;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import java.util.ArrayList;
import java.util.List;
import java.util.Locale;

public class MainActivity extends AppCompatActivity {

    private final List<Product> visible = new ArrayList<>(ProductData.PRODUCTS);
    private ProductAdapter adapter;
    private TextView emptyMessage;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        emptyMessage = findViewById(R.id.emptyMessage);

        RecyclerView productList = findViewById(R.id.productList);
        productList.setLayoutManager(new LinearLayoutManager(this));

        adapter = new ProductAdapter(visible, this::openDetails);
        productList.setAdapter(adapter);

        EditText searchField = findViewById(R.id.searchField);
        searchField.addTextChangedListener(new TextWatcher() {
            @Override
            public void beforeTextChanged(CharSequence s, int start, int count, int after) {
            }

            @Override
            public void onTextChanged(CharSequence s, int start, int before, int count) {
                filter(s.toString());
            }

            @Override
            public void afterTextChanged(Editable s) {
            }
        });
    }

    private void filter(String query) {
        String needle = query.toLowerCase(Locale.ROOT);

        visible.clear();
        for (Product product : ProductData.PRODUCTS) {
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

Jeśli szablon wygenerował w `onCreate` linie z `enableEdgeToEdge()` i obsługą marginesów
systemowych – zostaw je, odpowiadają za to, żeby treść nie chowała się pod paskiem stanu.

### Trzy rzeczy do zauważenia

**`LinearLayoutManager`** decyduje o układzie listy. Zamiast niego można wstawić
`GridLayoutManager(this, 2)` i dostać siatkę dwukolumnową – jedna linijka, zupełnie inny
wygląd.

**`adapter.notifyDataSetChanged()`** to powiadomienie: „dane się zmieniły, przerysuj".
W Reakcie robił to za Ciebie `useState`. Tutaj musisz powiedzieć o tym wprost – i o tym
zapomnieć jest bardzo łatwo. Objaw: filtrujesz, a lista stoi.

**`visible.clear()` zamiast `visible = new ArrayList<>(...)`.** Adapter trzyma
odniesienie do **tej konkretnej** listy. Podstawienie nowej sprawiłoby, że adapter dalej
patrzyłby na starą i nic by się nie zmieniło. Trzeba modyfikować tę samą listę.

---

## Krok 11 – ekran szczegółów

New → Activity → Empty Views Activity → nazwa `ProductDetailActivity`. Android Studio
utworzy klasę, układ **i wpis w manifeście** – dlatego lepiej robić to kreatorem niż
ręcznie.

**`res/layout/activity_product_detail.xml`**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:id="@+id/detailName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="22sp"
        android:textStyle="bold" />

    <TextView
        android:id="@+id/detailDescription"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:paddingTop="8dp"
        android:textSize="15sp" />

    <TextView
        android:id="@+id/detailPrice"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:paddingTop="16dp"
        android:textSize="20sp"
        android:textStyle="bold" />

    <TextView
        android:id="@+id/detailStock"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:paddingTop="8dp"
        android:textSize="15sp" />

</LinearLayout>
```

**`ProductDetailActivity.java`**:

```java
package com.example.sklep;

import android.os.Bundle;
import android.widget.TextView;
import android.widget.Toast;

import androidx.appcompat.app.AppCompatActivity;

public class ProductDetailActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_product_detail);

        int productId = getIntent().getIntExtra("productId", -1);
        Product product = ProductData.findById(productId);

        if (product == null) {
            Toast.makeText(this, "Nie ma takiego produktu", Toast.LENGTH_SHORT).show();
            finish();
            return;
        }

        setTitle(product.getName());

        ((TextView) findViewById(R.id.detailName)).setText(product.getName());
        ((TextView) findViewById(R.id.detailDescription)).setText(product.getDescription());
        ((TextView) findViewById(R.id.detailPrice)).setText(Format.price(product.getPrice()));
        ((TextView) findViewById(R.id.detailStock))
                .setText("Na stanie: " + product.getStock() + " szt.");
    }
}
```

### Intencje

**`Intent` to komunikat: „uruchom ten ekran"**, a `putExtra` dokłada do niego dane.
Ekran docelowy odbiera je przez `getIntent()`.

Przekazujemy **identyfikator, a nie cały obiekt** – dokładnie tak jak w materiale 07.
Powód jest tu nawet mocniejszy: żeby przekazać cały obiekt przez intencję, musiałby
implementować `Parcelable`, co znaczy kilkadziesiąt linii dodatkowego kodu. A w materiale
09 i tak zamienimy `findById` na zapytanie do serwera po tym identyfikatorze.

**Przycisku „wstecz" nie trzeba programować** – Android obsługuje go sam, zdejmując ekran
ze stosu. `finish()` w kodzie robi to samo ręcznie.

---

## Krok 12 – Git

Android Studio tworzy własny `.gitignore` w katalogu projektu. Sprawdź, czy zawiera co
najmniej:

```gitignore
*.iml
.gradle/
local.properties
.idea/
build/
captures/
*.apk
```

**`local.properties` nie może trafić do repozytorium** – zawiera ścieżkę do SDK
z Twojego komputera, u kolegi zupełnie inną.

Katalogi `build/` też nie – to wynik kompilacji, odtwarzalny w każdej chwili. Potrafią
zajmować setki megabajtów.

```bash
cd sklep-fullstack
git status --short          # NIE może tu być build/ ani local.properties
git add .
git commit -m "feat(android): lista produktów na danych statycznych"
```

Podział na commity:

```
chore(android): projekt Android Studio w module android/
feat(android): model Product i dane statyczne
feat(android): układ pozycji listy
feat(android): adapter RecyclerView
feat(android): wyszukiwarka
feat(android): ekran szczegółów produktu
```

---

## Sprawdź, czy rozumiesz

1. Dlaczego pola klasy `Product` mają angielskie nazwy identyczne z kluczami JSON-a?
2. Dlaczego `price` jest typu `String`, a nie `double`?
3. Co robi `RecyclerView` inaczej niż zwykłe wyświetlenie wszystkich pozycji naraz?
4. Która metoda adaptera wykonuje się przy każdym przewinięciu, a która rzadko?
5. Po co istnieje `ViewHolder`?
6. Do czego służy interfejs `OnProductClickListener` i co jest jego odpowiednikiem
   w Reakcie?
7. Dlaczego po zmianie danych trzeba wywołać `notifyDataSetChanged()`?
8. Dlaczego przekazujemy przez `Intent` identyfikator, a nie cały obiekt?
9. Jaka jest różnica między `dp` a `sp` i dlaczego teksty podaje się w `sp`?

---

## Częste błędy

| Objaw | Przyczyna |
|---|---|
| `R` podkreślone na czerwono, „cannot resolve symbol R" | błąd składni w którymś pliku XML |
| lista pusta, aplikacja nie wywala się | brak `setLayoutManager` albo `setAdapter` |
| `NullPointerException` przy `setText` | `findViewById` z identyfikatorem z innego układu |
| filtrowanie nie działa, lista stoi | brak `notifyDataSetChanged()` |
| filtrowanie nie działa mimo powiadomienia | podstawiono nową listę zamiast zmienić istniejącą |
| `IllegalStateException: already has a parent` | `inflate(..., parent, true)` zamiast `false` |
| `ActivityNotFoundException` | brak ekranu w `AndroidManifest.xml` |
| po obrocie ekranu znika wpisany tekst | aktywność jest tworzona od nowa – to temat cyklu życia |
| emulator działa bardzo wolno | wyłączona wirtualizacja w BIOS-ie |
| Gradle nie kończy synchronizacji | brak internetu albo pierwsze pobieranie zależności |

---

## Czego jeszcze nie ma

- **Sieci.** Dane są w klasie, aplikacja działa w trybie samolotowym. To zmieni się
  w materiale 09.
- **Uprawnienia do internetu.** Trzeba je będzie zadeklarować w manifeście.
- **Wątków.** Żądanie sieciowe nie może iść na głównym wątku – Android to zablokuje.
  To jest mobilny odpowiednik problemu, który w Reakcie rozwiązywał `useEffect`.
- **Obsługi obrotu ekranu.** Po obrocie aktywność tworzy się od nowa i stan przepada.
- **Zapisu danych.** Tylko odczyt, tak jak wszędzie do tej pory.

---

## Co dalej

**Materiał 09:** Retrofit i `/api/products/`. Kasujesz `ProductData`, dodajesz
uprawnienie do internetu, opisujesz endpointy interfejsem i pobierasz dane w tle.
`ProductAdapter` i układy **zostają bez zmian** – już wiesz, na jakiej zasadzie.

Zmierzysz się też z czymś, czego React nie wymagał: emulator nie widzi `localhost`
Twojego komputera pod tą nazwą. Adres, którego użyjesz, to `10.0.2.2` – wspominałem
o tym w materiale 06.

---

# Zadania

Pracujesz na swojej domenie z poprzednich materiałów.

**Zadanie 1 – projekt**
Utwórz projekt Android Studio w katalogu `android/` swojego repozytorium. Java, Empty
Views Activity, minSdk 26. Uruchom na emulatorze lub telefonie, zrób zrzut ekranu
z „Hello World". Commit – sprawdź, że `build/` i `local.properties` nie weszły.

**Zadanie 2 – model**
Klasa modelu z polami **o nazwach identycznych z kluczami w Twoim API**. Pola prywatne
i `final`, konstruktor, gettery. W `docs/android.md` wklej obok siebie odpowiedź JSON
z endpointu i kod klasy – nazwy muszą się zgadzać co do znaku.

**Zadanie 3 – dane**
Klasa z danymi statycznymi, minimum 8 rekordów **skopiowanych z własnego API**, oraz
metoda wyszukująca po identyfikatorze.

**Zadanie 4 – układ pozycji**
Plik układu pojedynczego wiersza, minimum 4 pola, w tym jedno sformatowane. Teksty
w `sp`, odstępy w `dp`, podgląd wypełniony przez `tools:text`.

**Zadanie 5 – adapter**
Adapter z klasą `ViewHolder`. W `docs/android.md` wyjaśnij własnymi słowami, dlaczego
`findViewById` jest w konstruktorze `ViewHolder`, a nie w `onBindViewHolder`.

**Zadanie 6 – lista działa**
Lista wyświetla wszystkie rekordy i daje się płynnie przewijać. Zrzut ekranu.

**Zadanie 7 – wyszukiwarka**
Filtrowanie w trakcie pisania, bez rozróżniania wielkości liter, z komunikatem przy
braku wyników.

**Zadanie 8 – szczegóły**
Dotknięcie pozycji otwiera osobny ekran z pełnymi danymi. Identyfikator przekazany przez
`Intent`. Tytuł ekranu ustawiony na nazwę rekordu.

**Zadanie 9 – warunek wizualny**
Jedno pole ma wyglądać inaczej w zależności od wartości – na przykład „Niedostępny"
zamiast liczby przy zerowym stanie. Odpowiednik renderowania warunkowego z materiału 05.

**Zadanie 10 – porównanie technologii**
W `docs/android.md` tabela porównująca rozwiązania tego samego problemu w Reakcie
i w Androidzie: wyświetlenie listy, reakcja na kliknięcie, przejście do szczegółów,
formatowanie liczby, odświeżenie widoku po zmianie danych. Po jednym zdaniu na komórkę.
To zadanie jest ważniejsze, niż wygląda – pokazuje, że uczysz się wzorców, a nie nazw.

---

**Zadanie dodatkowe (dla chętnych)**
Zamień `notifyDataSetChanged()` na `DiffUtil`. Ta klasa porównuje starą i nową listę
i powiadamia adapter **tylko o tym, co faktycznie się zmieniło** – dzięki czemu
pojawianie się i znikanie pozycji jest animowane, a nie skokowe. W `docs/android.md`
napisz, dlaczego przerysowanie całej listy przy każdej literze jest marnotrawstwem
i jak `DiffUtil` to zmienia.