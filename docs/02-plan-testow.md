# Plan testów — Meridian Bank 2.6.0

| | |
|---|---|
| **Dokument** | Plan testów aplikacji Meridian Bank (bankowość internetowa — środowisko szkoleniowe QA) |
| **Wersja planu** | 1.0 |
| **Data** | 2026-08-14 |
| **Przedmiot testów** | `index.html`, wersja aplikacji 2.6.0 |
| **Autor** | Zespół QA |
| **Status** | Do zatwierdzenia |
| **Dokumenty powiązane** | [Specyfikacja funkcjonalna](01-specyfikacja-funkcjonalna.md) · [Przypadki testowe](03-przypadki-testowe.md) · [Dane testowe i selektory](04-dane-testowe-i-selektory.md) · [Raport defektów](05-raport-defektow.md) · [Szablony i checklisty](06-szablony-i-checklisty.md) |

---

## 1. Wstęp

### 1.1 Cel dokumentu
Plan definiuje zakres, podejście, zasoby i harmonogram testów aplikacji Meridian Bank 2.6.0.
Jest podstawą do:
- uzgodnienia z interesariuszami, **co** i **w jakim stopniu** zostanie przetestowane,
- oszacowania pracochłonności i przydzielenia zadań,
- oceny gotowości produktu do wydania (kryteria wyjścia).

### 1.2 Charakterystyka przedmiotu testów
Meridian Bank to jednoplikowa aplikacja webowa (SPA) symulująca bankowość internetową.
Nie posiada backendu — wszystkie operacje wykonują się w przeglądarce, a dane żyją wyłącznie
w pamięci (odświeżenie strony przywraca stan startowy). Aplikacja obejmuje 11 modułów
funkcjonalnych: logowanie z 2FA, pulpit, przelewy, historię operacji, karty, BLIK, lokaty,
kredyty, kantor walutowy, wiadomości, ustawienia i pomoc oraz asystenta AI.

### 1.3 Podstawa testów
1. Specyfikacja funkcjonalna ([01-specyfikacja-funkcjonalna.md](01-specyfikacja-funkcjonalna.md)) — wymagania `FR-*` odtworzone z implementacji.
2. Interfejs użytkownika i teksty w aplikacji.
3. Ogólne reguły domeny bankowej (poprawność księgowania, spójność sald, autoryzacja operacji).
4. Dobre praktyki użyteczności i dostępności (WCAG 2.1 AA — poziom podstawowy).

---

## 2. Zakres testów

### 2.1 W zakresie

| Obszar | Moduł / funkcje | Priorytet |
|---|---|---|
| A. Uwierzytelnianie i sesja | logowanie, 2FA SMS, blokada konta, wylogowanie, timeout sesji | **Krytyczny** |
| B. Przelewy | krajowy (3 kroki), własny, zaplanowane, zlecenia stałe, autoryzacja SMS, potwierdzenie | **Krytyczny** |
| C. Rachunki i historia | salda, kafelki, filtrowanie, sortowanie, paginacja, szczegóły operacji, eksport CSV | **Wysoki** |
| D. Produkty | lokaty (kalkulator, otwarcie, zerwanie), kredyty (kalkulator, wniosek), kantor (przelicznik, wymiana) | **Wysoki** |
| E. Karty i BLIK | podgląd numeru, blokada/odblokowanie, limity, generowanie i wygasanie kodu BLIK | **Wysoki** |
| F. Ustawienia i profil | dane osobowe, zmiana hasła, powiadomienia, motyw, czas sesji | **Średni** |
| G. Komunikacja | wiadomości, powiadomienia, formularz kontaktowy, FAQ, czat Meri | **Średni** |
| H. Testy niefunkcjonalne | użyteczność, responsywność (RWD), kompatybilność przeglądarek, dostępność podstawowa, wydajność renderowania, bezpieczeństwo warstwy klienckiej (XSS w polach tekstowych, maskowanie danych) | **Średni** |

### 2.2 Poza zakresem

| Wyłączenie | Uzasadnienie |
|---|---|
| Testy API / integracyjne / bazy danych | Brak backendu i warstwy integracji |
| Testy wydajnościowe pod obciążeniem (load, stress) | Brak serwera; aplikacja jednoużytkownikowa |
| Testy penetracyjne, audyt kryptografii | Brak transmisji danych i mechanizmów bezpieczeństwa serwerowego |
| Zgodność z przepisami (PSD2, RODO, rekomendacje KNF) | Podmiot fikcyjny, aplikacja szkoleniowa |
| Testy migracji i kopii zapasowych | Brak trwałości danych |
| Aplikacje mobilne natywne | Nie istnieją — testowany wyłącznie widok RWD |
| Lokalizacja i tłumaczenia | Tylko wersja polska |

---

## 3. Podejście do testów

### 3.1 Poziomy i typy testów

| Poziom / typ | Opis | Udział |
|---|---|---|
| Testy systemowe funkcjonalne | Weryfikacja wymagań `FR-*` na kompletnej aplikacji | ~65% |
| Testy walidacji danych wejściowych | Formularze: pola wymagane, formaty, wartości brzegowe, komunikaty błędów | ~15% |
| Testy przepływów end-to-end | Scenariusze biznesowe (E2E) łączące wiele modułów | ~10% |
| Testy niefunkcjonalne | RWD, kompatybilność, dostępność, użyteczność, wydajność UI | ~5% |
| Testy eksploracyjne | Sesje z kartami testowymi (charter-based), 60–90 min | ~5% |
| Testy regresji | Po każdej poprawce — zestaw regresyjny z [06-szablony-i-checklisty.md](06-szablony-i-checklisty.md) | wg potrzeb |
| Retesty | Weryfikacja poprawionych defektów | wg potrzeb |

### 3.2 Techniki projektowania testów

| Technika | Zastosowanie |
|---|---|
| **Klasy równoważności** | Kwoty przelewu, kwota lokaty (<1 000 / 1 000–500 000 / >500 000), długość nazwy odbiorcy, długość hasła |
| **Analiza wartości brzegowych** | 26 cyfr NRB (25/26/27), kwota = saldo / saldo + 0,01, kwota 0,00 i 0,01, lokata 999,99 / 1 000 / 500 000 / 500 000,01, limity suwaków (min/max), data = dzisiaj / wczoraj |
| **Tablica decyzyjna** | Zmiana hasła (obecne × siła nowego × zgodność powtórzenia), wniosek kredytowy (dochód × zgoda RODO), wymiana walut (para × kwota × saldo) |
| **Przejścia stanów** | Logowanie (0/1/2/3 błędne próby → blokada → odblokowanie), BLIK (pusty → aktywny → wygasły), przelew (formularz → podsumowanie → sukces), karta (aktywna ⇄ zablokowana), sesja (aktywna → ostrzeżenie → wygaśnięcie) |
| **Testowanie oparte na przypadkach użycia** | Scenariusze E2E (rozdział 3.3) |
| **Zgadywanie błędów** | Wielokrotne kliknięcia, operacje w trakcie loadera, powroty w procesie, kopiuj-wklej danych, znaki specjalne, bardzo długie ciągi |
| **Testowanie eksploracyjne** | Karty: „przepływ pieniądza", „stan po wielu operacjach", „zachowanie interfejsu przy wąskim ekranie" |

### 3.3 Scenariusze end-to-end (przekrojowe)

| ID | Scenariusz |
|---|---|
| E2E-01 | Logowanie → przelew krajowy na dziś → weryfikacja salda na pulpicie → odnalezienie operacji w historii → eksport CSV → wylogowanie |
| E2E-02 | Logowanie → otwarcie lokaty → weryfikacja obciążenia konta → zerwanie lokaty → weryfikacja zwrotu kapitału |
| E2E-03 | Logowanie → wymiana PLN → EUR → weryfikacja obu sald → przelew własny → zgodność historii |
| E2E-04 | Zmiana hasła (z autoryzacją SMS) → wylogowanie → logowanie starym hasłem (odrzucone) → logowanie nowym hasłem |
| E2E-05 | Przelew z datą przyszłą → pozycja na liście „Zaplanowane" → anulowanie → weryfikacja braku wpływu na saldo |
| E2E-06 | Ustawienie sesji na 1 minutę → bezczynność → ostrzeżenie → wygaśnięcie sesji → ponowne logowanie |

### 3.4 Kryteria pokrycia
- **100%** wymagań `FR-*` pokryte co najmniej jednym przypadkiem testowym (macierz pokrycia w [03-przypadki-testowe.md](03-przypadki-testowe.md), rozdz. 14).
- **100%** przypadków o priorytecie krytycznym i wysokim wykonanych przed decyzją o wydaniu.
- **100%** komunikatów walidacyjnych zweryfikowanych co do treści.
- Każdy moduł objęty co najmniej jednym scenariuszem negatywnym.

---

## 4. Kryteria wejścia, wyjścia, zawieszenia

### 4.1 Kryteria wejścia (rozpoczęcia testów)
1. Wersja aplikacji dostarczona i uruchamialna w przeglądarce.
2. Plan testów i przypadki testowe przygotowane oraz przejrzane.
3. Dostępne dane testowe i konta (rozdz. 6).
4. Środowisko testowe skonfigurowane (przeglądarki, rozdzielczości).
5. Testy dymne (smoke, 12 pozycji) zakończone wynikiem pozytywnym.

### 4.2 Kryteria wyjścia (zakończenia testów)
1. Wykonano ≥ 95% zaplanowanych przypadków testowych, w tym **100%** o priorytecie krytycznym i wysokim.
2. Brak otwartych defektów o priorytecie **Blokujący** i **Krytyczny**.
3. Defekty **Poważne**: maks. 2 otwarte, każdy z zaakceptowanym obejściem i decyzją właściciela produktu.
4. Wszystkie defekty zgłoszone, sklasyfikowane i mające status końcowy (naprawiony / odroczony / odrzucony).
5. Testy regresji po ostatniej poprawce zakończone wynikiem pozytywnym.
6. Raport końcowy z testów przygotowany i zaakceptowany.

### 4.3 Kryteria zawieszenia i wznowienia
**Zawieszenie**, gdy:
- aplikacja nie uruchamia się lub logowanie jest niemożliwe (blokada > 30% przypadków),
- defekt blokujący uniemożliwia testowanie kluczowego modułu,
- dostarczono nową wersję w trakcie cyklu testowego bez uzgodnienia.

**Wznowienie**, gdy: przyczyna zawieszenia została usunięta, dostarczono poprawioną wersję,
a testy dymne zakończyły się wynikiem pozytywnym.

---

## 5. Środowisko testowe

### 5.1 Konfiguracja

| Element | Wymaganie |
|---|---|
| Uruchomienie | Otwarcie `index.html` w przeglądarce (`file://`) lub serwer statyczny (`python3 -m http.server`) |
| Zalecane | **Serwer statyczny (`http://localhost`)** — protokół `file://` blokuje API schowka (wpływa na FR-BLK-05) |
| Sieć | Wymagany dostęp do `fonts.googleapis.com` (kroje pisma); przy braku sieci aplikacja działa na krojach zastępczych — warto przetestować oba warianty |
| Czyszczenie stanu | Odświeżenie strony (F5) — przywraca dane startowe; nie są wykorzystywane `localStorage` ani cookies |

### 5.2 Matryca przeglądarek i urządzeń

| Przeglądarka | Wersja | Priorytet | Zakres |
|---|---|---|---|
| Chrome (Windows/macOS) | najnowsza | **Podstawowa** | Pełny zestaw przypadków |
| Firefox | najnowsza | Wysoki | Zestaw regresyjny + moduły krytyczne |
| Edge | najnowsza | Średni | Zestaw regresyjny |
| Safari (macOS/iOS) | najnowsza | Średni | Zestaw regresyjny + RWD |
| Chrome Android | najnowsza | Wysoki | RWD, menu burger, formularze dotykowe |

**Rozdzielczości:** 1920×1080, 1440×900, 1280×800, 1024×768, 768×1024 (tablet), 390×844 (telefon).
Punkty łamania w kodzie: **900 px** (siatki dwukolumnowe → jedna kolumna, ukrycie sidebara) i **560 px**.

### 5.3 Narzędzia

| Cel | Narzędzie |
|---|---|
| Zarządzanie testami | Arkusz / TestRail / Xray (ID przypadków wg [03-przypadki-testowe.md](03-przypadki-testowe.md)) |
| Zgłaszanie defektów | Jira (szablon w [06-szablony-i-checklisty.md](06-szablony-i-checklisty.md)) |
| Debug i inspekcja | Narzędzia deweloperskie przeglądarki (konsola, Network, Elements, Application) |
| RWD | Tryb urządzenia w DevTools + urządzenia fizyczne |
| Dostępność | axe DevTools, nawigacja klawiaturą, kontrast |
| Automatyzacja (opcjonalnie) | Playwright / Cypress + selektory `data-testid` (katalog: [04-dane-testowe-i-selektory.md](04-dane-testowe-i-selektory.md)) |
| Dowody | Zrzuty ekranu, nagrania, eksport konsoli |

---

## 6. Dane testowe

Pełny wykaz: [04-dane-testowe-i-selektory.md](04-dane-testowe-i-selektory.md). Podstawowe:

| Element | Wartość |
|---|---|
| Login / hasło | `jan.kowalski` / `Test123!` |
| Kod SMS (każda autoryzacja) | `123456` |
| Rachunek odbiorcy (poprawny, 26 cyfr) | `61109010140000071219812874` |
| Salda startowe | 12 543,21 PLN · 45 200,00 PLN · 3 250,50 EUR |

**Zasada:** każdy przypadek testowy zakłada stan startowy uzyskany przez **odświeżenie strony**,
chyba że w warunkach wstępnych zapisano inaczej. Testy zmieniające salda (przelewy, lokaty,
wymiana walut) muszą być wykonywane na świeżym stanie lub uwzględniać wynik poprzednich operacji.

---

## 7. Organizacja i harmonogram

### 7.1 Role i odpowiedzialności

| Rola | Odpowiedzialność |
|---|---|
| Kierownik testów | Plan testów, harmonogram, raportowanie, decyzja o kryteriach wyjścia |
| Inżynier QA (2 os.) | Projektowanie i wykonywanie przypadków, zgłaszanie defektów, retesty |
| Automatyk QA (opcjonalnie) | Automatyzacja zestawu dymnego i regresyjnego |
| Właściciel produktu | Priorytetyzacja defektów, akceptacja odstępstw |
| Deweloper | Naprawa defektów, wsparcie w analizie |

### 7.2 Harmonogram (cykl 5 dni roboczych)

| Dzień | Zakres | Produkt pracy |
|---|---|---|
| 1 | Przygotowanie środowiska, testy dymne, moduły A (logowanie/sesja) i B (przelewy) | Wyniki smoke + wyniki modułów A, B |
| 2 | Moduły C (rachunki, historia) i E (karty, BLIK) | Wyniki modułów C, E |
| 3 | Moduł D (lokaty, kredyty, kantor), scenariusze E2E | Wyniki modułu D + E2E |
| 4 | Moduły F, G, testy niefunkcjonalne (RWD, kompatybilność, dostępność) | Wyniki F, G, NFR |
| 5 | Testy eksploracyjne, retesty, regresja, raport końcowy | Raport z testów |

### 7.3 Szacowana pracochłonność

| Aktywność | Osobodni |
|---|---|
| Przygotowanie (plan, przypadki, środowisko) | 2,0 |
| Wykonanie testów funkcjonalnych | 4,0 |
| Testy niefunkcjonalne i eksploracyjne | 1,0 |
| Retesty i regresja | 1,5 |
| Raportowanie i zamknięcie | 0,5 |
| **Razem** | **9,0** |

---

## 8. Zarządzanie defektami

### 8.1 Klasyfikacja — waga (severity)

| Waga | Definicja | Przykład z tej aplikacji |
|---|---|---|
| **Blokująca** | Uniemożliwia korzystanie z aplikacji lub testowanie modułu | Nie działa logowanie |
| **Krytyczna** | Błędne skutki finansowe, utrata danych, obejście autoryzacji | Obciążenie niewłaściwego rachunku (DEF-001), utrata środków przy zerwaniu lokaty (DEF-003) |
| **Poważna** | Kluczowa funkcja działa niepoprawnie, brak sensownego obejścia | Brak walidacji sumy kontrolnej NRB (DEF-005) |
| **Drobna** | Niedogodność z obejściem, błąd niekrytycznej funkcji | Limity karty niezapisywane w stanie (DEF-008) |
| **Kosmetyczna** | Wygląd, literówka, drobna niespójność | Niespójny komunikat, przesunięcie elementu |

### 8.2 Priorytet naprawy

| Priorytet | Oczekiwany czas naprawy |
|---|---|
| P1 — natychmiastowy | ≤ 1 dzień roboczy (blokuje wydanie) |
| P2 — wysoki | ≤ 3 dni robocze (w bieżącym wydaniu) |
| P3 — normalny | Kolejne wydanie |
| P4 — niski | Backlog, do decyzji właściciela produktu |

### 8.3 Cykl życia defektu
`Nowy → Przypisany → W naprawie → Do retestu → Zamknięty`
Ścieżki alternatywne: `Odrzucony` (brak defektu / działa zgodnie z projektem), `Odroczony`,
`Ponownie otwarty` (retest negatywny).

### 8.4 Wymagana zawartość zgłoszenia
Tytuł, moduł, waga, priorytet, środowisko (przeglądarka + rozdzielczość), warunki wstępne,
kroki odtworzenia, rezultat oczekiwany i rzeczywisty, dowody (zrzut/nagranie), powiązany
przypadek testowy i wymaganie `FR-*`. Szablon: [06-szablony-i-checklisty.md](06-szablony-i-checklisty.md).

---

## 9. Ryzyka

### 9.1 Ryzyka produktowe (co może zawieść w aplikacji)

| ID | Ryzyko | Praw. | Wpływ | Poziom | Przeciwdziałanie |
|---|---|---|---|---|---|
| RP-1 | Błędne księgowanie / niespójność sald między modułami | Wysokie | Krytyczny | **Bardzo wysoki** | Priorytetowe testy przepływu pieniądza; po każdej operacji weryfikacja salda na pulpicie, w historii i w listach wyboru |
| RP-2 | Obejście autoryzacji SMS lub blokady konta | Średnie | Krytyczny | **Wysoki** | Testy przejść stanów dla logowania i modala SMS, próby anulowania w trakcie procesu |
| RP-3 | Błędy walidacji danych (NRB, kwoty, daty) prowadzące do przyjęcia niepoprawnej operacji | Wysokie | Poważny | **Wysoki** | Wartości brzegowe i klasy równoważności dla wszystkich formularzy |
| RP-4 | Niepoprawne wyliczenia (lokata, kredyt, kurs) | Średnie | Poważny | Średni | Weryfikacja wyników wobec obliczeń referencyjnych (arkusz kalkulacyjny) |
| RP-5 | Utrata danych po operacjach na listach (duplikaty ID) | Wysokie | Poważny | **Wysoki** | Testy dodawania/usuwania w seriach dla lokat, zleceń stałych, zaplanowanych |
| RP-6 | Problemy z układem strony na urządzeniach mobilnych | Średnie | Drobny | Niski | Testy RWD w punktach łamania 900 px i 560 px |
| RP-7 | Niedostępność krojów pisma z sieci zewnętrznej | Niskie | Kosmetyczny | Niski | Test w trybie offline |

### 9.2 Ryzyka projektowe (co może zawieść w procesie)

| ID | Ryzyko | Przeciwdziałanie |
|---|---|---|
| RJ-1 | Brak dokumentacji wymagań (odtworzone z kodu) | Uzgodnienie specyfikacji funkcjonalnej z właścicielem produktu przed startem; zachowania wątpliwe oznaczone **[!]** |
| RJ-2 | Reset danych przy odświeżeniu utrudnia odtwarzanie defektów | Kroki odtworzenia zawsze od stanu startowego; dokumentowanie pełnej sekwencji |
| RJ-3 | Zależności czasowe (loadery, 120 s BLIK, timeout sesji) wydłużają testy | Testy czasowe zgrupowane; w automatyzacji stabilne oczekiwania na stan, nie na czas |
| RJ-4 | Ograniczone zasoby (2 testerów, 5 dni) | Priorytetyzacja oparta na ryzyku, moduły krytyczne w pierwszej kolejności |

---

## 10. Metryki i raportowanie

### 10.1 Metryki

| Metryka | Definicja | Cel |
|---|---|---|
| Postęp wykonania | wykonane / zaplanowane przypadki | ≥ 95% |
| Wskaźnik zdawalności | zdane / wykonane | ≥ 90% |
| Gęstość defektów | defekty / moduł | monitorowana |
| Defekty otwarte wg wagi | liczba wg klasyfikacji | 0 blokujących i krytycznych |
| Skuteczność retestów | retesty zdane / wykonane | ≥ 95% |
| Pokrycie wymagań | wymagania z ≥ 1 przypadkiem | 100% |

### 10.2 Raportowanie
- **Codziennie:** krótki status (wykonane / zdane / niezdane / zablokowane, nowe defekty).
- **Na koniec cyklu:** raport końcowy z testów (szablon w [06-szablony-i-checklisty.md](06-szablony-i-checklisty.md)) zawierający ocenę ryzyka i rekomendację wydania.

---

## 11. Produkty pracy (deliverables)

| Produkt | Odpowiedzialny | Termin |
|---|---|---|
| Plan testów (ten dokument) | Kierownik testów | Przed startem |
| Specyfikacja funkcjonalna (baseline) | QA | Przed startem |
| Przypadki testowe + macierz pokrycia | QA | Przed startem |
| Zestaw danych testowych i selektorów | QA | Przed startem |
| Wyniki wykonania przypadków | QA | Codziennie |
| Zgłoszenia defektów | QA | Na bieżąco |
| Raport końcowy z testów | Kierownik testów | Dzień 5 |

---

## 12. Podejście do automatyzacji (rekomendacja)

Aplikacja jest przygotowana do automatyzacji: wszystkie istotne elementy mają atrybuty
`data-testid` (katalog w [04-dane-testowe-i-selektory.md](04-dane-testowe-i-selektory.md)).

**Rekomendowany zakres pierwszego etapu** (największy zwrot z nakładu):
1. Zestaw dymny (12 przypadków) — uruchamiany po każdej zmianie.
2. Ścieżka przelewu krajowego z autoryzacją SMS (TC-TRF-001 … 015).
3. Walidacje formularza przelewu (parametryzowane danymi).
4. Filtry i paginacja historii operacji.
5. Weryfikacja spójności sald po operacjach finansowych (najwyższe ryzyko RP-1).

**Zasady:** selektory wyłącznie po `data-testid`; brak sztywnych opóźnień — oczekiwanie na
stan elementu (loader `#loader.on` znika, modal ma klasę `.open`); każdy test startuje od
przeładowania strony, co gwarantuje niezależność (stan aplikacji nie jest utrwalany).

---

## 13. Zatwierdzenie

| Rola | Imię i nazwisko | Data | Podpis |
|---|---|---|---|
| Kierownik testów | | | |
| Właściciel produktu | | | |
| Lider zespołu deweloperskiego | | | |
