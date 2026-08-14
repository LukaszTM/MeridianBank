# Szablony i checklisty — Meridian Bank 2.6.0

Dokumenty robocze do codziennej pracy zespołu QA: checklisty wykonawcze oraz szablony
zgłoszeń i raportów.

---

## 1. Checklista testów dymnych (smoke) — 15–20 minut

Wykonywana po każdej nowej wersji, **przed** rozpoczęciem właściwych testów.
Niepowodzenie dowolnej pozycji = niespełnione kryterium wejścia (patrz [plan testów](02-plan-testow.md), rozdz. 4.1).

| # | Sprawdzenie | Oczekiwany rezultat | Wynik |
|---|---|---|---|
| 1 | Aplikacja uruchamia się | Widoczny ekran logowania, brak błędów w konsoli | ☐ |
| 2 | Logowanie `jan.kowalski` / `Test123!` + kod `123456` | Pulpit z powitaniem i toastem | ☐ |
| 3 | Kafelki rachunków | Trzy rachunki z saldami 12 543,21 zł / 45 200,00 zł / 3 250,50 € | ☐ |
| 4 | Nawigacja po 11 modułach | Każda strona otwiera się z właściwym tytułem i treścią | ☐ |
| 5 | Przelew krajowy na dziś (10 zł) | Ekran sukcesu z numerem referencyjnym | ☐ |
| 6 | Saldo po przelewie | Konto osobiste pomniejszone o 10 zł | ☐ |
| 7 | Historia operacji | Nowa operacja na szczycie listy, licznik zaktualizowany | ☐ |
| 8 | Eksport CSV | Plik pobrany, dane zgodne z widokiem | ☐ |
| 9 | Generowanie kodu BLIK | Kod 6-cyfrowy, licznik od 120 s | ☐ |
| 10 | Kalkulator lokaty (10 000 / 6 mies.) | Odsetki brutto 240,00 zł, netto 194,40 zł | ☐ |
| 11 | Kantor — przeliczenie 100 PLN → EUR | Wynik 22,83 € | ☐ |
| 12 | Blokada i odblokowanie karty | Zmiana statusu w obie strony | ☐ |
| 13 | Tryb ciemny | Interfejs czytelny, przełączniki zsynchronizowane | ☐ |
| 14 | Wylogowanie | Ekran logowania z banerem potwierdzającym | ☐ |
| 15 | Konsola przeglądarki | Brak błędów poziomu `error` | ☐ |

**Wynik testów dymnych:** ☐ Pozytywny ☐ Negatywny — uzasadnienie: …

---

## 2. Checklista regresji — 60–90 minut

Wykonywana po każdej poprawce defektu oraz przed zamknięciem cyklu testowego.

### Uwierzytelnianie i sesja
- ☐ Walidacja pustych pól logowania
- ☐ Blokada po 3 błędnych próbach + automatyczne odblokowanie po 30 s
- ☐ Błędny i poprawny kod SMS
- ☐ Wylogowanie z potwierdzeniem
- ☐ Ostrzeżenie o sesji i jej przedłużenie

### Przelewy
- ☐ Pełny przelew krajowy z autoryzacją (saldo, historia, numer referencyjny)
- ☐ Walidacje: nazwa < 3 znaki, NRB ≠ 26 cyfr, kwota 0 / > saldo, data z przeszłości, pusty tytuł
- ☐ Przelew z konta oszczędnościowego — **obciążany właściwy rachunek** (DEF-001)
- ☐ Przelew zaplanowany — pozycja na liście i kontrola środków (DEF-002)
- ☐ Przelew własny w obie strony
- ☐ Zlecenie stałe: utworzenie, wstrzymanie, usunięcie (DEF-006)
- ☐ Anulowanie autoryzacji SMS i 3 błędne kody

### Rachunki i historia
- ☐ Filtry (fraza, typ, daty), sortowanie, paginacja, czyszczenie filtrów
- ☐ Szczegóły operacji
- ☐ Eksport CSV zgodny z filtrami
- ☐ Zgodność sald: pulpit ↔ historia ↔ listy wyboru rachunków

### Produkty
- ☐ Kalkulator i otwarcie lokaty; zerwanie lokaty (DEF-003)
- ☐ Kalkulator kredytu i wniosek (walidacja dochodu i zgody RODO)
- ☐ Kantor: przeliczenia i wymiana PLN ⇄ EUR, kontrola środków

### Karty i BLIK
- ☐ Pokazanie/ukrycie numeru, blokada/odblokowanie
- ☐ Limity z autoryzacją SMS (DEF-008)
- ☐ Generowanie, wygaśnięcie i anulowanie kodu BLIK

### Ustawienia i komunikacja
- ☐ Edycja profilu z walidacją e-maila i telefonu
- ☐ Zmiana hasła + logowanie nowym hasłem
- ☐ Wiadomości (odczyt, badge), powiadomienia
- ☐ FAQ i formularz kontaktowy
- ☐ Czat Meri — saldo, przelew, kursy, temat nierozpoznany

### Niefunkcjonalne
- ☐ Widok 390 px i 768 px
- ☐ Tryb ciemny we wszystkich modułach
- ☐ Nawigacja klawiaturą w formularzach
- ☐ Konsola bez błędów

---

## 3. Szablon zgłoszenia defektu

```
TYTUŁ:        [Moduł] Zwięzły opis obserwowanego problemu

ID:           DEF-XXX
Moduł:        Przelewy / Historia / Lokaty / …
Waga:         Blokująca | Krytyczna | Poważna | Drobna | Kosmetyczna
Priorytet:    P1 | P2 | P3 | P4
Wersja:       Meridian Bank 2.6.0
Wymaganie:    FR-XXX-nn
Przypadek:    TC-XXX-nnn
Zgłaszający:  Imię i nazwisko, data

ŚRODOWISKO
  Przeglądarka:  Chrome 1xx (Windows 11)
  Rozdzielczość: 1920×1080
  Uruchomienie:  http://localhost:8000 | file://

WARUNKI WSTĘPNE
  1. Aplikacja w stanie startowym (po odświeżeniu strony)
  2. Użytkownik zalogowany jako jan.kowalski

KROKI ODTWORZENIA
  1. …
  2. …
  3. …

REZULTAT OCZEKIWANY
  …

REZULTAT RZECZYWISTY
  …

CZĘSTOTLIWOŚĆ:  Zawsze (3/3) | Sporadycznie (1/5)
WPŁYW NA UŻYTKOWNIKA / BIZNES
  …

ZAŁĄCZNIKI
  screenshot_DEF-XXX.png, nagranie.mp4, zrzut konsoli
```

---

## 4. Szablon raportu z wykonania testów (dzienny)

```
RAPORT DZIENNY — dzień N cyklu, data

ZAKRES DNIA
  Moduły: …

STATYSTYKA
  Zaplanowane:   xx
  Wykonane:      xx (xx%)
  Zdane:         xx
  Niezdane:      xx
  Zablokowane:   xx

NOWE DEFEKTY
  DEF-XXX  [Waga]  Tytuł
  …

BLOKADY I RYZYKA
  …

PLAN NA KOLEJNY DZIEŃ
  …
```

---

## 5. Szablon raportu końcowego z testów

```
RAPORT KOŃCOWY Z TESTÓW — Meridian Bank 2.6.0
Okres testów:   …
Zespół:         …

1. ZAKRES
   Przetestowane moduły, wyłączenia, odstępstwa od planu testów.

2. WYNIKI WYKONANIA
   Przypadki:  zaplanowane / wykonane / zdane / niezdane / zablokowane
   Pokrycie wymagań: xx%
   Wskaźnik zdawalności: xx%

3. DEFEKTY
   Razem: xx  |  Krytyczne: x  Poważne: x  Drobne: x  Kosmetyczne: x
   Otwarte na dzień zamknięcia: …
   Najistotniejsze defekty i ich wpływ: …

4. OCENA KRYTERIÓW WYJŚCIA
   Kryterium                                   Spełnione   Komentarz
   ≥95% przypadków wykonanych                  TAK/NIE     …
   Brak otwartych defektów krytycznych         TAK/NIE     …
   Regresja po ostatniej poprawce zdana        TAK/NIE     …

5. OCENA RYZYKA
   Obszary o najwyższym ryzyku rezydualnym: …

6. REKOMENDACJA
   ☐ Zalecam wydanie
   ☐ Zalecam wydanie warunkowo (z listą zastrzeżeń)
   ☐ Nie zalecam wydania — uzasadnienie: …

Podpis kierownika testów: …
```

---

## 6. Karty testów eksploracyjnych (sesje 60 min)

| Karta | Cel (charter) | Obszar ryzyka |
|---|---|---|
| EXP-01 | Prześledź „przepływ pieniądza": wykonaj serię operacji (przelew, przelew własny, lokata, wymiana walut) i po każdej porównaj saldo na pulpicie, w historii i na listach wyboru rachunków | RP-1 — niespójność sald |
| EXP-02 | Spróbuj wykonać operację bez pełnej autoryzacji: anuluj modal SMS w różnych momentach, odśwież stronę w trakcie procesu, klikaj wielokrotnie przyciski zatwierdzenia | RP-2 — obejście autoryzacji |
| EXP-03 | Testuj listy w seriach: dodaj i usuń wiele lokat, zleceń stałych i przelewów zaplanowanych, sprawdzając identyfikatory i skutki akcji | RP-5 — duplikaty ID |
| EXP-04 | Wprowadzaj nietypowe dane: bardzo długie ciągi, znaki specjalne, emoji, HTML, kopiowanie kwot z arkusza | RP-3 — walidacja |
| EXP-05 | Przejdź całą aplikację na ekranie 390 px, wyłącznie palcem/klawiaturą, w trybie ciemnym | RP-6 — użyteczność, dostępność |
| EXP-06 | Testuj zależności czasowe: wygaśnięcie BLIK-a, ostrzeżenie o sesji, blokada logowania, praca w wielu kartach przeglądarki | RP-2, RP-7 |

**Zapis z sesji:** czas trwania, obszar, zastosowane dane, znalezione problemy, pytania otwarte,
pomysły na nowe przypadki testowe.

---

## 7. Definicja gotowości i ukończenia

**Definition of Ready (przypadek gotowy do wykonania)**
- ☐ Ma jednoznaczne warunki wstępne i dane testowe
- ☐ Ma mierzalny rezultat oczekiwany (konkretne wartości, nie „działa poprawnie")
- ☐ Jest powiązany z wymaganiem `FR-*`
- ☐ Ma nadany priorytet

**Definition of Done (moduł przetestowany)**
- ☐ Wykonano wszystkie przypadki o priorytecie K i W
- ☐ Wszystkie odchylenia zgłoszone jako defekty z kompletem informacji
- ☐ Wyniki zapisane w narzędziu do zarządzania testami
- ☐ Defekty krytyczne przekazane deweloperom tego samego dnia
