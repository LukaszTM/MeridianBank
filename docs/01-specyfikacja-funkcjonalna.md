# Specyfikacja funkcjonalna — Meridian Bank 2.6.0

Dokument opisuje **rzeczywiste, zaimplementowane zachowanie** aplikacji szkoleniowej
Meridian Bank (plik `index.html`). Powstał na podstawie analizy kodu źródłowego i
weryfikacji w przeglądarce, więc stanowi *bazę odniesienia (baseline)* dla testów —
tam, gdzie zachowanie jest wątpliwe lub niezgodne z regułami bankowymi, oznaczono to
jako **[!]** i opisano w dokumencie [05-raport-defektow.md](05-raport-defektow.md).

Identyfikatory wymagań (`FR-xxx-nn`) są używane w macierzy pokrycia w dokumencie
[03-przypadki-testowe.md](03-przypadki-testowe.md).

---

## 1. Charakterystyka produktu

| Cecha | Wartość |
|---|---|
| Nazwa | Meridian Bank — bankowość internetowa (środowisko testowe QA) |
| Wersja | 2.6.0 |
| Rodzaj | Jednoplikowa aplikacja SPA (HTML + CSS + vanilla JS), bez frameworków |
| Artefakt | `index.html` (~2 090 linii, ~128 kB) |
| Backend | **Brak.** Wszystkie operacje są symulowane po stronie przeglądarki |
| Trwałość danych | **Brak.** Stan żyje w obiekcie `state` w pamięci; odświeżenie strony (F5) przywraca dane startowe |
| Język interfejsu | polski (`<html lang="pl">`) |
| Motywy | jasny / ciemny (`data-theme` na elemencie `<html>`) |
| Waluty | PLN, EUR, USD, GBP, CHF (formatowanie `Intl.NumberFormat('pl-PL')`) |
| Wsparcie testów | atrybuty `data-testid` na wszystkich istotnych elementach (katalog: [04-dane-testowe-i-selektory.md](04-dane-testowe-i-selektory.md)) |

### 1.1 Architektura logiczna

```
index.html
├── <style>            — style, motywy, komponenty (linie ~10–294)
├── <body>
│   ├── #login-view    — ekran logowania (krok 1: login/hasło, krok 2: kod SMS)
│   ├── #app           — aplikacja po zalogowaniu
│   │   ├── #sidebar   — nawigacja (11 pozycji)
│   │   ├── #topbar    — tytuł, motyw, powiadomienia, menu użytkownika
│   │   └── .content   — 11 sekcji <section class="page">
│   ├── modale         — potwierdzenie, SMS, sesja, szczegóły operacji, kredyt, zlecenie stałe
│   └── czat AI „Meri" — przycisk FAB + panel
└── <script>           — logika (linie ~971–2089)
```

Kluczowe obiekty w kodzie:

- `state` — cały stan aplikacji (użytkownik, rachunki, operacje, karty, lokaty, zlecenia, ustawienia).
- `H` — stan modułu Historia (filtry, sortowanie, paginacja).
- `RATES` — statyczna tabela kursów walut.
- `DEP_RATES` — oprocentowanie lokat wg okresu.
- `SMS_CODE = '123456'` — stały kod autoryzacyjny.

---

## 2. Dane startowe (po każdym przeładowaniu strony)

### 2.1 Użytkownik

| Pole | Wartość |
|---|---|
| Login | `jan.kowalski` |
| Hasło | `Test123!` (zmienialne w Ustawieniach — obowiązuje do końca sesji przeglądarki) |
| Kod SMS | `123456` (stały, dla wszystkich autoryzacji) |
| Imię i nazwisko | Jan Kowalski |
| E-mail | jan.kowalski@example.com |
| Telefon | +48 601 234 789 |

### 2.2 Rachunki

| ID | Nazwa | Waluta | Saldo startowe | IBAN |
|---|---|---|---|---|
| `ror` | Konto osobiste | PLN | 12 543,21 | PL61 1090 1014 0000 0712 1981 2874 |
| `sav` | Konto oszczędnościowe | PLN | 45 200,00 | PL27 1140 2004 0000 3002 0135 5387 |
| `eur` | Konto walutowe | EUR | 3 250,50 | PL83 1240 6957 0111 0000 1131 4260 |

### 2.3 Pozostałe dane startowe

- **Odbiorcy zapisani (3):** Anna Nowak, Wspólnota Mieszkaniowa „Słoneczna 5", Biuro Rachunkowe LibraTax Sp. z o.o.
- **Karty (2):** debetowa VISA `4556 7375 8689 4523` (ważna do 08/29), kredytowa Mastercard `5387 1043 2276 8817` (11/28) — obie aktywne.
- **Lokaty (1):** `DEP-2044`, 10 000 zł, 4,80%, koniec = dzisiaj + 92 dni.
- **Zlecenia stałe (1):** `ST-1001`, Wspólnota Mieszkaniowa „Słoneczna 5", Czynsz, 1 200 zł, 10. dzień miesiąca, aktywne.
- **Wiadomości (3):** `M-3` (nieprzeczytana), `M-2`, `M-1`.
- **Powiadomienia (3):** `N-1`, `N-2` (nieprzeczytane), `N-3`.
- **Historia operacji:** generowana deterministycznie (`mulberry32(20260801)`) — 46 transakcji z ostatnich ~75 dni + do 3 wypłat wynagrodzenia 8 500 zł (28. dzień miesiąca). Salda („Saldo po") liczone wstecz od salda konta osobistego.
- **Zaplanowane przelewy:** lista pusta.

### 2.4 Tabela kursów (statyczna)

| Waluta | Kupno | Sprzedaż | Zmiana 24 h |
|---|---|---|---|
| EUR | 4,24 | 4,38 | +0,12% |
| USD | 3,90 | 4,04 | −0,34% |
| GBP | 4,97 | 5,13 | +0,05% |
| CHF | 4,45 | 4,59 | +0,21% |

---

## 3. Logowanie i sesja

### FR-LOG-01 — Walidacja pól logowania
Puste pole „Login" lub „Hasło" blokuje wysłanie formularza; pole podświetla się na czerwono
(`.field.invalid`) i pokazuje komunikat „Podaj login." / „Podaj hasło.". Walidacja odbywa się
przed wysłaniem (formularz ma `novalidate` — walidacja HTML5 nie działa).

### FR-LOG-02 — Poprawne logowanie (krok 1)
Dla `jan.kowalski` + aktualne hasło: pokazuje się loader „Trwa logowanie…" na **900 ms**,
następnie krok 2 (kod SMS) i toast „SMS z banku: Twój kod autoryzacyjny to 123456."

### FR-LOG-03 — Błędne dane logowania
Licznik `loginFails` rośnie; komunikat: „Nieprawidłowy login lub hasło. Pozostałe próby: N."
(N = 2, następnie 1).

### FR-LOG-04 — Blokada konta
Po **3.** błędnej próbie: baner blokady, przycisk „Zaloguj się" `disabled`, licznik 30 → 0 s.
Po upływie 30 s blokada znika, przycisk wraca, licznik prób jest zerowany.

### FR-LOG-05 — Autoryzacja SMS przy logowaniu
Kod `123456` → loader „Przygotowujemy Twój pulpit…" (900 ms) → pulpit + toast „Zalogowano
pomyślnie. Witaj, Jan!". Kod błędny → komunikat „Nieprawidłowy kod SMS. Spróbuj ponownie."
**[!]** Brak limitu prób na tym etapie (w przeciwieństwie do autoryzacji operacji w aplikacji).
Przycisk „← Wróć do logowania" wraca do kroku 1 (bez czyszczenia loginu).

### FR-LOG-06 — Pokaż/ukryj hasło
Przycisk z ikoną oka przełącza `type` pola między `password` a `text`.

### FR-LOG-07 — Zapamiętaj login
Checkbox „Zapamiętaj mój login" — przy wylogowaniu **pole loginu nie jest czyszczone**
(gdy odznaczony — jest czyszczone). Nie ma zapisu do `localStorage` — po odświeżeniu strony pole jest puste.

### FR-LOG-08 — Nie pamiętasz hasła
Link wyświetla wyłącznie toast informacyjny (symulacja) — brak procesu odzyskiwania.

### FR-SES-01 — Automatyczne wylogowanie
Timer sprawdza bezczynność co 1 s. Limit: 1 / 5 / 10 minut (domyślnie 5, Ustawienia).
Aktywność (`click`, `keydown`, `touchstart`) resetuje licznik bezczynności — **ale nie wtedy,
gdy pokazane jest ostrzeżenie**.

### FR-SES-02 — Ostrzeżenie o wygaśnięciu sesji
Na 60 s przed końcem pojawia się modal „Twoja sesja wkrótce wygaśnie" z odliczaniem.
Przyciski: „Przedłuż sesję" (reset licznika + toast) oraz „Wyloguj teraz".
Modal sesji jest jedynym, którego **nie zamyka kliknięcie w tło**.
**[!]** Dla limitu 1 minuty próg ostrzeżenia wynosi 0 s, więc modal pojawia się natychmiast.

### FR-SES-03 — Wygaśnięcie sesji
Po przekroczeniu limitu: wylogowanie i baner „Sesja wygasła z powodu braku aktywności.
Zaloguj się ponownie."

### FR-SES-04 — Wylogowanie ręczne
Menu użytkownika → „Wyloguj się" → modal potwierdzenia → ekran logowania, zamknięcie
wszystkich modali i czatu, baner „Zostałeś poprawnie wylogowany. Do zobaczenia!".
Stan danych (salda, operacje) **nie jest resetowany** — pozostaje z sesji.

---

## 4. Nawigacja i powłoka aplikacji

### FR-NAV-01 — Menu boczne
11 pozycji: Pulpit, Przelewy, Historia, Karty, BLIK, Lokaty, Kredyty, Kantor, Wiadomości,
Ustawienia, Pomoc. Kliknięcie: loader **450 ms**, zmiana tytułu w nagłówku, aktywny stan
przycisku, przewinięcie do góry. Wejście na stronę odświeża jej dane.

### FR-NAV-02 — Skróty `data-goto`
Dowolny element z atrybutem `data-goto` (kafelki rachunków, przyciski szybkiego dostępu,
linki w treści, pozycje menu użytkownika) nawiguje do wskazanej strony.

### FR-NAV-03 — Widok mobilny
Przy szerokości ≤ 900 px sidebar jest ukryty; otwiera go przycisk „burger". Nawigacja
zamyka sidebar.

### FR-NAV-04 — Motyw jasny/ciemny
Przełącznik w nagłówku i w Ustawieniach → wspólny stan (`state.settings.dark`), oba kontrolki
są synchronizowane. Motyw nie jest zapamiętywany po odświeżeniu.

### FR-NAV-05 — Powiadomienia
Dzwonek otwiera listę 3 powiadomień; kropka `#bell-dot` widoczna, gdy istnieje nieprzeczytane.
Kliknięcie pozycji oznacza ją jako przeczytaną; „Oznacz jako przeczytane" oznacza wszystkie.
Otwarcie jednego dropdownu zamyka drugi; kliknięcie poza obszarem zamyka oba.

### FR-NAV-06 — Toasty
Komunikaty pojawiają się w prawym górnym rogu, zanikają po ~4,2 s i są usuwane po ~4,6 s.
Wiele toastów układa się jeden pod drugim.

---

## 5. Pulpit

| ID | Wymaganie |
|---|---|
| FR-DASH-01 | Powitanie zależne od godziny: <5 „Dobrej nocy", <12 i <18 „Dzień dobry", inaczej „Dobry wieczór" + data w formacie długim (pl-PL) |
| FR-DASH-02 | Trzy kafelki rachunków z nazwą, saldem w walucie rachunku i numerem IBAN; kliknięcie przenosi do Historii |
| FR-DASH-03 | Baner „Meri analizuje…" z linkiem do Historii |
| FR-DASH-04 | Cztery skróty: Nowy przelew, Kod BLIK, Wymień walutę, Załóż lokatę |
| FR-DASH-05 | „Ostatnie operacje" — 5 najnowszych pozycji; kliknięcie wiersza otwiera modal szczegółów |
| FR-DASH-06 | Widget „Kursy walut" — skeleton przez **1,4 s** przy pierwszym wyświetleniu w sesji, potem kurs średni ((kupno+sprzedaż)/2) i zmiana 24 h ze strzałką i kolorem |
| FR-DASH-07 | Wykres „Wydatki — ostatnie 6 miesięcy": suma obciążeń (kwoty ujemne) w podziale na miesiące, słupki skalowane do maksimum |

---

## 6. Przelewy

Moduł ma 4 zakładki: **Przelew krajowy**, **Przelew własny**, **Zaplanowane**, **Zlecenia stałe**.

### 6.1 Przelew krajowy — proces 3-krokowy

| ID | Wymaganie |
|---|---|
| FR-TRF-01 | Lista „Z rachunku" zawiera **tylko rachunki w PLN** (konto osobiste, oszczędnościowe) z aktualnym saldem w etykiecie |
| FR-TRF-02 | Wybór odbiorcy zapisanego automatycznie wypełnia nazwę i numer rachunku |
| FR-TRF-03 | Walidacja nazwy odbiorcy: min. **3 znaki** po przycięciu spacji |
| FR-TRF-04 | Walidacja numeru rachunku: dokładnie **26 cyfr** po usunięciu spacji i opcjonalnego prefiksu `PL`. **[!]** Brak weryfikacji sumy kontrolnej IBAN/NRB |
| FR-TRF-05 | Walidacja kwoty: format `cyfry[,\|.cyfry(1–2)]`, spacje ignorowane; kwota > 0; kwota ≤ saldo rachunku źródłowego (komunikat „Niewystarczające środki na rachunku.") |
| FR-TRF-06 | Walidacja daty: wymagana, nie może być z przeszłości (`min` = dzisiaj); domyślnie dzisiaj |
| FR-TRF-07 | Walidacja tytułu: wymagany, maks. 70 znaków (`maxlength`) |
| FR-TRF-08 | „Wyczyść formularz" czyści pola, checkboxy, ustawia datę na dzisiaj i kasuje komunikaty błędów |
| FR-TRF-09 | Krok 2 — podsumowanie: rachunek źródłowy, odbiorca, numer rachunku sformatowany grupami, kwota, data (DD.MM.RRRR), tytuł, prowizja 0,00 zł; „← Popraw dane" wraca do kroku 1 z zachowaniem wartości |
| FR-TRF-10 | „Zatwierdź i autoryzuj" otwiera modal SMS; poprawny kod → loader „Realizujemy przelew…" (1 100 ms) → krok 3 |
| FR-TRF-11 | Modal SMS: **3 błędne próby** → zamknięcie modala i toast „Autoryzacja odrzucona po 3 błędnych próbach. Operacja anulowana." (formularz pozostaje na kroku 2) |
| FR-TRF-12 | Krok 3 — sukces: numer referencyjny `PRZ-` + 9 ostatnich cyfr znacznika czasu, komunikat o obciążeniu lub o dacie realizacji, przyciski „Pobierz potwierdzenie" i „Nowy przelew" |
| FR-TRF-13 | Data dzisiejsza → natychmiastowe księgowanie: nowa operacja „Przelew do: <odbiorca> — <tytuł>", kategoria „Przelew", kwota ujemna. **[!]** Środki są zawsze pobierane z **konta osobistego**, niezależnie od wybranego rachunku źródłowego |
| FR-TRF-14 | Data przyszła → wpis na liście „Zaplanowane", komunikat o realizacji o godz. 06:00. **[!]** Środki nie są rezerwowane, a przelew nigdy nie zostaje zrealizowany (brak mechanizmu wykonania) |
| FR-TRF-15 | „Zapisz odbiorcę" dodaje odbiorcę do listy (bez duplikatów po numerze rachunku) i odświeża listę wyboru |
| FR-TRF-16 | „Pobierz potwierdzenie" generuje plik `potwierdzenie_<ref>.txt` z danymi przelewu i adnotacją o środowisku szkoleniowym |

### 6.2 Przelew własny

| ID | Wymaganie |
|---|---|
| FR-OWN-01 | Obie listy zawierają tylko rachunki PLN; domyślnie rachunek docelowy ≠ źródłowy |
| FR-OWN-02 | Ten sam rachunek po obu stronach → błąd „Rachunek źródłowy i docelowy muszą być różne." |
| FR-OWN-03 | Walidacja kwoty jak w przelewie krajowym (format, > 0, ≤ saldo) |
| FR-OWN-04 | Realizacja natychmiastowa: salda obu rachunków aktualizowane, operacja „Przelew własny: A → B" w historii (kategoria „Przelew własny"), toast, wyczyszczenie pola kwoty |
| FR-OWN-05 | Brak autoryzacji SMS dla przelewu własnego |

### 6.3 Zaplanowane

| ID | Wymaganie |
|---|---|
| FR-SCH-01 | Tabela: data (DD.MM.RRRR), odbiorca, tytuł, kwota; pusty stan z instrukcją |
| FR-SCH-02 | „Anuluj" → modal potwierdzenia → usunięcie pozycji + toast |

### 6.4 Zlecenia stałe

| ID | Wymaganie |
|---|---|
| FR-STO-01 | Tabela: odbiorca, tytuł, kwota, dzień miesiąca, status (Aktywne/Wstrzymane), akcje |
| FR-STO-02 | Nowe zlecenie: odbiorca i tytuł wymagane, kwota > 0, dzień miesiąca 1–28 (lista) |
| FR-STO-03 | „Wstrzymaj"/„Wznów" przełącza status + toast |
| FR-STO-04 | „Usuń" → modal potwierdzenia → usunięcie + toast |
| FR-STO-05 | **[!]** ID nowego zlecenia = `ST-` + (1000 + liczba zleceń + 1) — po usunięciu zlecenia kolejne dostaje **zduplikowany identyfikator** |
| FR-STO-06 | Zlecenia stałe nie generują operacji w historii (brak realizacji cyklicznej) |

---

## 7. Historia operacji

| ID | Wymaganie |
|---|---|
| FR-HIS-01 | Tabela: data, opis, kategoria (badge), kwota (kolor: dodatnie zielone z „+"), saldo po operacji |
| FR-HIS-02 | Wyszukiwarka filtruje po **opisie i kategorii**, bez uwzględniania wielkości liter, reaguje na każde wpisanie znaku |
| FR-HIS-03 | Filtr typu: Wszystkie / Uznania (kwota > 0) / Obciążenia (kwota < 0) |
| FR-HIS-04 | Filtry dat „od"/„do" — porównanie po dacie ISO, włącznie z datami granicznymi |
| FR-HIS-05 | Liczba pozycji na stronę: 10 / 25 / 50; zmiana filtra lub liczby pozycji wraca na stronę 1 |
| FR-HIS-06 | Licznik „(N operacji)" oraz informacja „Strona X z Y · pozycje A–B" |
| FR-HIS-07 | Paginacja: ‹ / › (wyłączone na skrajnych stronach), numery stron; przy > 7 stronach numery środkowe zwijane do „…" |
| FR-HIS-08 | Sortowanie po kolumnie Data i Kwota — kliknięcie przełącza kierunek, wskaźnik ▼/▲ przy aktywnej kolumnie (domyślnie: data malejąco) |
| FR-HIS-09 | Brak wyników → komunikat „Brak operacji spełniających kryteria. Zmień filtry, aby zobaczyć wyniki." |
| FR-HIS-10 | „Wyczyść filtry" przywraca stan domyślny wszystkich filtrów, sortowania i paginacji + toast |
| FR-HIS-11 | Kliknięcie wiersza otwiera modal ze szczegółami: data i godzina, opis, kategoria, kwota, saldo po operacji, numer referencyjny |
| FR-HIS-12 | „Eksportuj CSV" pobiera plik `wyciag_meridian_RRRR-MM-DD.csv`: separator `;`, BOM UTF-8, końce linii CRLF, przecinek dziesiętny, opis w cudzysłowach; eksportowane są **operacje po filtrach** (wszystkie strony), toast z liczbą operacji |

---

## 8. Karty płatnicze

| ID | Wymaganie |
|---|---|
| FR-CRD-01 | Dwie karty w formie graficznej: marka, numer maskowany `•••• •••• •••• 4523`, posiadacz, data ważności, status (Aktywna/Zablokowana) |
| FR-CRD-02 | „Pokaż numer"/„Ukryj numer" przełącza widoczność pełnego numeru (niezależnie dla każdej karty) |
| FR-CRD-03 | „Zablokuj" → modal ostrzegawczy → status „Zablokowana", karta wyszarzona, przycisk zmienia się w „Odblokuj" + toast |
| FR-CRD-04 | „Odblokuj" → modal → status „Aktywna" + toast |
| FR-CRD-05 | Suwak limitu płatności internetowych: 0–10 000 zł, krok 100; etykieta aktualizowana na żywo |
| FR-CRD-06 | Suwak limitu wypłat z bankomatów: 0–5 000 zł, krok 100 |
| FR-CRD-07 | „Zapisz limity" wymaga autoryzacji SMS; po poprawnym kodzie toast „Limity karty zostały zapisane." **[!]** Limity nie są zapisywane w stanie aplikacji (tylko toast) |
| FR-CRD-08 | Przełącznik płatności zbliżeniowych — toast o włączeniu/wyłączeniu |
| FR-CRD-09 | Limity i przełącznik dotyczą wyłącznie karty debetowej (brak rozróżnienia kart) |

---

## 9. BLIK

| ID | Wymaganie |
|---|---|
| FR-BLK-01 | Stan początkowy: ikona + przycisk „Generuj kod BLIK" |
| FR-BLK-02 | Generowanie: losowy kod 6-cyfrowy (zakres 100000–999999) wyświetlany w formacie `XXX XXX` |
| FR-BLK-03 | Odliczanie 120 s (co 1 s) + pasek postępu zmniejszający się proporcjonalnie |
| FR-BLK-04 | Po upływie czasu: stan „Kod BLIK wygasł." + przycisk „Generuj nowy kod" |
| FR-BLK-05 | „Kopiuj kod" kopiuje kod do schowka + toast; przy braku dostępu do schowka (np. protokół `file://`) toast pokazuje kod |
| FR-BLK-06 | „Anuluj" zatrzymuje licznik i wraca do stanu początkowego |
| FR-BLK-07 | Ponowne generowanie resetuje licznik do 120 s i tworzy nowy kod |
| FR-BLK-08 | Kod BLIK nie jest powiązany z żadną transakcją (brak płatności kodem) |

---

## 10. Lokaty

| ID | Wymaganie |
|---|---|
| FR-DEP-01 | Kalkulator: kwota (domyślnie 10 000) i okres 3 / 6 / 12 / 24 mies. z oprocentowaniem 4,50 / 4,80 / 5,00 / 4,60% |
| FR-DEP-02 | Walidacja kwoty: 1 000 ≤ kwota ≤ 500 000; poza zakresem — komunikat i wyniki „—" |
| FR-DEP-03 | Odsetki brutto = kwota × (stopa/100) × (miesiące/12); netto = brutto × 0,81 (podatek 19%); suma = kapitał + odsetki netto. Przeliczanie na żywo |
| FR-DEP-04 | „Otwórz lokatę" wymaga wystarczających środków na koncie osobistym (inaczej toast błędu) |
| FR-DEP-05 | Potwierdzenie w modalu → obciążenie konta osobistego operacją „Założenie lokaty N mies." (kategoria „Lokata") + nowa pozycja na liście lokat + toast |
| FR-DEP-06 | Data zakończenia lokaty = dzisiaj + (miesiące × 30 dni) — **[!]** uproszczenie: miesiąc = 30 dni |
| FR-DEP-07 | „Zerwij" → modal z informacją o utracie odsetek → zwrot kapitału na konto osobiste (operacja „Zwrot środków z lokaty …", kategoria „Lokata") + toast |
| FR-DEP-08 | **[!]** ID nowej lokaty = `DEP-` + (2044 + liczba lokat) — po zerwaniu lokaty kolejna dostaje **zduplikowany identyfikator**, a zerwanie jednej z takich lokat usuwa obie |
| FR-DEP-09 | Pusta lista lokat → komunikat „Nie masz aktywnych lokat…" |

---

## 11. Kredyty

| ID | Wymaganie |
|---|---|
| FR-LOA-01 | Suwak kwoty: 1 000–200 000 zł, krok 1 000 (domyślnie 25 000) |
| FR-LOA-02 | Suwak okresu: 6–120 miesięcy, krok 6 (domyślnie 36) |
| FR-LOA-03 | Rata annuitetowa dla stopy 9,99% rocznie: `rata = P·r / (1 − (1+r)^−n)`, gdzie `r = 0,0999/12`; całkowita kwota = rata × liczba rat. Przeliczanie na żywo |
| FR-LOA-04 | „Złóż wniosek" otwiera modal z podsumowaniem parametrów (kwota, okres, rata) |
| FR-LOA-05 | Walidacja wniosku: dochód netto ≥ 1 000 zł (format kwoty), zgoda RODO **wymagana**, zgoda BIK opcjonalna |
| FR-LOA-06 | Poprawny wniosek → loader „Analizujemy Twój wniosek…" (1 500 ms) → toast z numerem `WNK-` + 6 cyfr |
| FR-LOA-07 | Wniosek nie tworzy żadnego rekordu w aplikacji (symulacja) |

---

## 12. Kantor walutowy

| ID | Wymaganie |
|---|---|
| FR-FXR-01 | Tabela kursów: kupno, sprzedaż, zmiana 24 h (kolor i strzałka) dla EUR, USD, GBP, CHF |
| FR-FXR-02 | Przelicznik: kwota + para walut (PLN, EUR, USD, GBP, CHF po obu stronach), przeliczanie na żywo |
| FR-FXR-03 | Algorytm: kwota → PLN po kursie **kupna** waluty źródłowej, następnie PLN → waluta docelowa po kursie **sprzedaży**; dla PLN kurs = 1 |
| FR-FXR-04 | Ta sama waluta po obu stronach → wynik „—", komunikat „Wybierz dwie różne waluty.", przycisk wymiany nieaktywny |
| FR-FXR-05 | Niepoprawna kwota → wynik „—", komunikat „Podaj poprawną kwotę.", przycisk nieaktywny |
| FR-FXR-06 | Wymiana wykonalna **wyłącznie dla pary PLN ⇄ EUR**; dla pozostałych par przycisk nieaktywny i komunikat wyjaśniający |
| FR-FXR-07 | Kontrola środków: PLN → EUR sprawdza saldo konta osobistego, EUR → PLN saldo konta walutowego (toast błędu przy braku środków) |
| FR-FXR-08 | Potwierdzenie w modalu → aktualizacja obu sald, operacja „Wymiana walut PLN → EUR" / „EUR → PLN" (kategoria „Wymiana walut") + toast |
| FR-FXR-09 | Informacja o zastosowanych kursach pod wynikiem przeliczenia |

---

## 13. Wiadomości

| ID | Wymaganie |
|---|---|
| FR-MSG-01 | Lista 3 wiadomości; nieprzeczytane wyróżnione kropką i pogrubieniem |
| FR-MSG-02 | Badge przy pozycji „Wiadomości" w menu pokazuje liczbę nieprzeczytanych; znika przy 0 |
| FR-MSG-03 | Kliknięcie wiadomości otwiera widok szczegółów (tytuł, nadawca, data, treść) i oznacza ją jako przeczytaną |
| FR-MSG-04 | „← Wróć do listy" wraca do skrzynki |
| FR-MSG-05 | Brak możliwości tworzenia, usuwania i odpowiadania na wiadomości |

---

## 14. Ustawienia

| ID | Wymaganie |
|---|---|
| FR-SET-01 | Dane osobowe: imię i nazwisko tylko do odczytu; e-mail i telefon edytowalne po kliknięciu „Edytuj" |
| FR-SET-02 | „Anuluj" (ten sam przycisk) przywraca wartości z modelu i wyłącza edycję |
| FR-SET-03 | Walidacja e-mail: wzorzec `tekst@tekst.tekst` (bez spacji) |
| FR-SET-04 | Walidacja telefonu: 9–15 cyfr (znaki inne niż cyfry są ignorowane przy liczeniu) |
| FR-SET-05 | „Zapisz zmiany" zapisuje dane w modelu, blokuje pola i pokazuje toast |
| FR-SET-06 | Zmiana hasła: obecne hasło musi być zgodne z aktualnym; nowe hasło min. 8 znaków, wielka litera, cyfra, znak specjalny; powtórzenie musi być identyczne i niepuste |
| FR-SET-07 | Wskaźnik siły hasła: 4 kryteria × 25% szerokości; kolory: ≤1 czerwony, 2–3 żółty, 4 zielony |
| FR-SET-08 | Poprawna zmiana hasła wymaga autoryzacji SMS; po niej pola są czyszczone, wskaźnik resetowany, toast informuje o zmianie. **Nowe hasło obowiązuje przy kolejnym logowaniu** |
| FR-SET-09 | Cztery przełączniki powiadomień (e-mail, SMS, push, marketing) — każda zmiana pokazuje toast. **[!]** Wartości nie są zapisywane w stanie aplikacji |
| FR-SET-10 | Przełącznik trybu ciemnego zsynchronizowany z przełącznikiem w nagłówku |
| FR-SET-11 | Czas automatycznego wylogowania: 1 / 5 / 10 minut; zmiana resetuje licznik bezczynności + toast |

---

## 15. Pomoc i kontakt

| ID | Wymaganie |
|---|---|
| FR-HLP-01 | FAQ — 4 pozycje typu akordeon; otwarcie jednej zamyka pozostałe; ponowne kliknięcie zamyka pozycję |
| FR-HLP-02 | Formularz kontaktowy: imię i nazwisko (wymagane), e-mail (wzorzec), temat (4 opcje), wiadomość (min. 10 znaków) |
| FR-HLP-03 | Poprawne wysłanie: formularz jest czyszczony, toast z numerem zgłoszenia `ZGL-` + 6 cyfr |
| FR-HLP-04 | Zgłoszenie nie trafia nigdzie (symulacja) |

---

## 16. Asystent AI „Meri"

| ID | Wymaganie |
|---|---|
| FR-CHT-01 | Przycisk FAB widoczny tylko po zalogowaniu; otwiera i zamyka panel czatu |
| FR-CHT-02 | Pierwsze otwarcie wyświetla wiadomość powitalną (tylko raz w sesji) |
| FR-CHT-03 | Wysyłka przyciskiem lub klawiszem Enter; puste wiadomości są ignorowane |
| FR-CHT-04 | Wskaźnik „Meri pisze…" zamieniany na odpowiedź po 750 ms |
| FR-CHT-05 | Rozpoznawane tematy (słowa kluczowe): saldo/stan konta/ile mam, przelew, blik, kurs/euro/walut/dolar, lokat, kontakt/infolini/telefon, powitania, hasło. Odpowiedź o saldzie zawiera **aktualne** salda trzech rachunków |
| FR-CHT-06 | Brak dopasowania → odpowiedź domyślna z listą obsługiwanych tematów |
| FR-CHT-07 | Historia czatu nie jest zapisywana (znika po wylogowaniu/odświeżeniu) |

---

## 17. Elementy wspólne

| ID | Wymaganie |
|---|---|
| FR-CMN-01 | Wstążka „ŚRODOWISKO TESTOWE QA" widoczna zawsze (bez możliwości kliknięcia) |
| FR-CMN-02 | Loader pełnoekranowy z tekstem kontekstowym blokuje interakcję na czas operacji |
| FR-CMN-03 | Modale zamykane przyciskiem „Anuluj" lub kliknięciem w tło — **z wyjątkiem modala sesji** |
| FR-CMN-04 | Autoryzacja SMS (operacje w aplikacji): kod `123456`, 3 próby, komunikat „Nieprawidłowy kod. Pozostałe próby: N.", zatwierdzanie klawiszem Enter; toast z kodem po 350 ms |
| FR-CMN-05 | Operacje wymagające SMS: przelew krajowy, zapis limitów karty, zmiana hasła |
| FR-CMN-06 | Kwoty formatowane wg pl-PL: spacja jako separator tysięcy, przecinek dziesiętny, sufiks waluty |
| FR-CMN-07 | Dane wejściowe są escapowane przed wstawieniem do HTML (funkcja `esc`) — ochrona przed XSS w polach tekstowych |
| FR-CMN-08 | Aplikacja obsługuje `prefers-reduced-motion` (wyłączenie animacji) i widoczny focus (`:focus-visible`) |

---

## 18. Ograniczenia znane i zamierzone

Te zachowania **nie są defektami** — wynikają z charakteru aplikacji szkoleniowej:

1. Brak backendu i trwałości — F5 przywraca dane startowe.
2. Kod SMS jest stały (`123456`) i podawany w toaście oraz na ekranie logowania.
3. Kursy walut są statyczne (brak integracji z NBP mimo etykiety „NBP + marża").
4. Wnioski kredytowe, zgłoszenia kontaktowe i limity kart nie tworzą rekordów.
5. Wymiana walut działa wyłącznie dla pary PLN ⇄ EUR.
6. Zlecenia stałe i przelewy zaplanowane nie są realizowane w czasie.
7. Aplikacja nie ma logowania zdarzeń ani mechanizmu obsługi błędów sieciowych.
