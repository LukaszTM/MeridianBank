# Raport defektów — Meridian Bank 2.6.0

Defekty wykryte podczas analizy aplikacji i **potwierdzone wykonaniem** w przeglądarce
(Chromium, uruchomienie z pliku lokalnego, stan startowy po odświeżeniu strony).
Każdy defekt zawiera kroki odtworzenia, dowód i propozycję poprawki.

## Podsumowanie

| ID | Tytuł | Moduł | Waga | Prio | Status |
|---|---|---|---|---|---|
| DEF-001 | Przelew krajowy zawsze obciąża konto osobiste — możliwe saldo ujemne | Przelewy | **Krytyczna** | P1 | Nowy |
| DEF-002 | Przelew zaplanowany nie rezerwuje środków i nigdy nie jest realizowany | Przelewy | **Krytyczna** | P1 | Nowy |
| DEF-003 | Zduplikowane ID lokat — zerwanie usuwa dwie lokaty, zwraca jeden kapitał | Lokaty | **Krytyczna** | P1 | Nowy |
| DEF-004 | Brak limitu prób kodu SMS przy logowaniu | Logowanie | **Poważna** | P2 | Nowy |
| DEF-005 | Brak weryfikacji sumy kontrolnej numeru rachunku (IBAN/NRB) | Przelewy | **Poważna** | P2 | Nowy |
| DEF-006 | Zduplikowane ID zleceń stałych — akcje działają na niewłaściwym wierszu | Zlecenia stałe | **Poważna** | P2 | Nowy |
| DEF-007 | Ostrzeżenie o wygaśnięciu sesji pojawia się natychmiast przy limicie 1 min | Sesja | Drobna | P3 | Nowy |
| DEF-008 | Limity karty nie są zapisywane ani egzekwowane | Karty | Drobna | P3 | Nowy |
| DEF-009 | Ustawienia powiadomień nie są zapisywane w stanie aplikacji | Ustawienia | Drobna | P3 | Nowy |
| DEF-010 | Data zakończenia lokaty liczona jako miesiąc = 30 dni | Lokaty | Drobna | P3 | Nowy |
| DEF-011 | Niespójna tolerancja formatu kwoty (spacja akceptowana, kropka nie) | Formularze | Kosmetyczna | P4 | Nowy |
| DEF-012 | Modale bez semantyki dialogu, pułapki fokusu i obsługi klawisza Esc | Dostępność | Drobna | P3 | Nowy |
| DEF-013 | Odrzucona autoryzacja SMS nie kończy operacji — można ponawiać bez limitu | Autoryzacja | Drobna | P3 | Nowy |

**Rozkład:** 3 krytyczne · 3 poważne · 6 drobnych · 1 kosmetyczna.

> **Blokada wydania:** defekty DEF-001, DEF-002 i DEF-003 dotyczą poprawności operacji
> finansowych i uniemożliwiają spełnienie kryteriów wyjścia z [planu testów](02-plan-testow.md) (rozdz. 4.2).

---

## DEF-001 — Przelew krajowy zawsze obciąża konto osobiste

| | |
|---|---|
| **Moduł** | Przelewy → Przelew krajowy |
| **Waga / priorytet** | Krytyczna / P1 |
| **Wymaganie** | FR-TRF-13 |
| **Przypadek testowy** | TC-TRF-021 |
| **Środowisko** | Meridian Bank 2.6.0, Chromium, 1280×720, stan startowy |

**Warunki wstępne:** użytkownik zalogowany, salda startowe (osobiste 12 543,21 zł, oszczędnościowe 45 200,00 zł).

**Kroki odtworzenia:**
1. Przelewy → zakładka „Przelew krajowy".
2. W polu „Z rachunku" wybierz **Konto oszczędnościowe · 45 200,00 zł**.
3. Odbiorca: `Anna Nowak`, rachunek: `61109010140000071219812874`, kwota: `20000`, tytuł: `Test`.
4. „Dalej →" → „Zatwierdź i autoryzuj" → kod `123456`.
5. Przejdź na Pulpit i odczytaj salda.

**Rezultat oczekiwany:** konto oszczędnościowe pomniejszone o 20 000 zł (→ 25 200,00 zł),
konto osobiste bez zmian; w podsumowaniu i w potwierdzeniu wskazany rachunek oszczędnościowy.

**Rezultat rzeczywisty:** konto oszczędnościowe **bez zmian** (45 200,00 zł),
konto osobiste obciążone i **ujemne: −7 456,79 zł**. Operacja w historii przypisana do konta osobistego.

**Wpływ:** obciążenie niewłaściwego rachunku, obejście kontroli dostępnych środków
(walidacja sprawdza saldo rachunku wybranego, a księgowanie następuje na innym), saldo ujemne
mimo braku limitu debetowego. Błędne są także saldo w kolumnie „Saldo po operacji" i eksport CSV.

**Przyczyna (analiza kodu):** funkcja `executeTransfer()` wywołuje `addTxn(...)`, a `addTxn()`
operuje na sztywno na rachunku `acc('ror')` — pomija `pendingTransfer.from`.

**Propozycja poprawki:** `addTxn` powinno przyjmować identyfikator rachunku
(np. `addTxn(accountId, desc, cat, amount)`) i modyfikować saldo wskazanego rachunku;
dodatkowo należy dodać zabezpieczenie przed zejściem salda poniżej zera przy księgowaniu.

---

## DEF-002 — Przelew zaplanowany nie rezerwuje środków i nigdy nie jest realizowany

| | |
|---|---|
| **Moduł** | Przelewy → Zaplanowane |
| **Waga / priorytet** | Krytyczna / P1 |
| **Wymaganie** | FR-TRF-14, FR-SCH-01 |
| **Przypadek testowy** | TC-TRF-020 |

**Kroki odtworzenia:**
1. Przelewy → Przelew krajowy, odbiorca `Anna Nowak`, rachunek `61109010140000071219812874`,
   kwota `9999`, data: **+5 dni**, tytuł `Zaplanowany`.
2. Zrealizuj proces z autoryzacją SMS.
3. Sprawdź saldo konta osobistego na pulpicie.
4. Powtórz kroki 1–2 jeszcze dwa razy (łącznie 29 997 zł zaplanowanych przelewów).

**Rezultat oczekiwany:** środki zarezerwowane lub co najmniej kontrola dostępnego salda
z uwzględnieniem zaplanowanych zleceń; komunikat, gdy suma przekracza saldo.

**Rezultat rzeczywisty:** saldo pozostaje 12 543,21 zł po każdym zleceniu; można zaplanować
przelewy na kwotę wielokrotnie przekraczającą saldo. Aplikacja nie realizuje ich w żadnym momencie
(brak mechanizmu wykonania w dacie realizacji) — zlecenie pozostaje na liście bezterminowo.

**Wpływ:** deklarowana funkcja „realizacja o godz. 06:00" nie działa; użytkownik może zaplanować
przelewy bez pokrycia. Podobna luka dotyczy zleceń stałych (nie generują żadnych operacji).

**Propozycja poprawki:** wprowadzić kwotę dostępną (saldo − suma zaplanowanych) i walidować ją
przy planowaniu, a przy wejściu do aplikacji realizować przelewy z datą ≤ dzisiaj
(z obsługą braku środków).

---

## DEF-003 — Zduplikowane ID lokat: zerwanie usuwa dwie lokaty i zwraca jeden kapitał

| | |
|---|---|
| **Moduł** | Lokaty |
| **Waga / priorytet** | Krytyczna / P1 |
| **Wymaganie** | FR-DEP-08 |
| **Przypadek testowy** | TC-DEP-009 |

**Kroki odtworzenia:**
1. Lokaty → kwota `2000` → „Otwórz lokatę" → potwierdź. *(Na liście: DEP-2044, DEP-2045.)*
2. Zerwij lokatę `DEP-2044` → potwierdź.
3. Kwota `3000` → „Otwórz lokatę" → potwierdź.
4. Sprawdź identyfikatory na liście lokat.
5. Zerwij widoczną lokatę `DEP-2045` → potwierdź.
6. Sprawdź listę lokat i saldo konta osobistego.

**Rezultat oczekiwany:** każda lokata ma unikalny identyfikator; zerwanie dotyczy jednej lokaty
i zwraca dokładnie jej kapitał.

**Rezultat rzeczywisty:** po kroku 4 **obie lokaty mają identyfikator `DEP-2045`**;
po kroku 5 **znikają obie**, a na konto wraca kapitał tylko jednej z nich (2 000 zł) —
**3 000 zł ulega utracie** (potwierdzone: saldo 19 543,21 zł zamiast 22 543,21 zł).

**Przyczyna (analiza kodu):** identyfikator generowany jako `'DEP-' + (2044 + state.deposits.length)`
— po usunięciu pozycji długość tablicy maleje i wartość się powtarza; usuwanie realizowane przez
`filter(x => x.id !== d.id)` kasuje wszystkie pozycje o tym samym ID.

**Propozycja poprawki:** globalny licznik rosnący (nigdy nie malejący) lub identyfikator
generowany niezależnie od długości tablicy; usuwanie po referencji lub po indeksie.

---

## DEF-004 — Brak limitu prób kodu SMS przy logowaniu

| | |
|---|---|
| **Moduł** | Logowanie (krok 2) |
| **Waga / priorytet** | Poważna / P2 |
| **Wymaganie** | FR-LOG-05 |
| **Przypadek testowy** | TC-LOG-013 |

**Kroki odtworzenia:**
1. Zaloguj się poprawnie loginem i hasłem (krok 1).
2. Wpisz błędny kod SMS i zatwierdź — powtórz **6 razy**.
3. Wpisz poprawny kod `123456`.

**Rezultat oczekiwany:** po 3 błędnych próbach powrót do kroku 1 lub blokada — spójnie
z autoryzacją operacji w aplikacji (3 próby) i z blokadą po 3 błędnych hasłach.

**Rezultat rzeczywisty:** liczba prób jest nieograniczona; po 6 błędach poprawny kod nadal
loguje użytkownika do aplikacji.

**Wpływ:** drugi składnik uwierzytelniania można łamać metodą siłową bez ograniczeń — istotna
luka bezpieczeństwa i niespójność z pozostałymi mechanizmami aplikacji.

**Propozycja poprawki:** licznik prób analogiczny do `smsAttempts` w modalu autoryzacji;
po 3 błędach powrót do kroku 1 z komunikatem i unieważnieniem kodu.

---

## DEF-005 — Brak weryfikacji sumy kontrolnej numeru rachunku

| | |
|---|---|
| **Moduł** | Przelewy → Przelew krajowy |
| **Waga / priorytet** | Poważna / P2 |
| **Wymaganie** | FR-TRF-04 |
| **Przypadek testowy** | TC-TRF-009 |

**Kroki odtworzenia:**
1. Przelewy → Przelew krajowy.
2. Numer rachunku odbiorcy: `00000000000000000000000000` (26 zer).
3. Uzupełnij pozostałe pola i kliknij „Dalej →".

**Rezultat oczekiwany:** komunikat o niepoprawnym numerze rachunku — numer nie spełnia
sumy kontrolnej NRB/IBAN (modulo 97).

**Rezultat rzeczywisty:** numer przyjęty, przelew możliwy do wykonania.

**Wpływ:** przelew na nieistniejący rachunek; w realnym systemie oznacza to odrzucenie
przelewu przez izbę rozliczeniową lub przelew na niewłaściwy rachunek przy literówce
(walidacja sumy kontrolnej wykrywa większość pomyłek w cyfrach).

**Propozycja poprawki:** dodać walidację modulo 97 dla numeru z prefiksem kraju
(`PL` + 26 cyfr → przestawienie i sprawdzenie reszty z dzielenia = 1).

---

## DEF-006 — Zduplikowane ID zleceń stałych

| | |
|---|---|
| **Moduł** | Przelewy → Zlecenia stałe |
| **Waga / priorytet** | Poważna / P2 |
| **Wymaganie** | FR-STO-05 |
| **Przypadek testowy** | TC-STO-007 |

**Kroki odtworzenia:**
1. Zlecenia stałe → „＋ Nowe zlecenie" → `Zlecenie A`, tytuł `Abonament`, kwota `100` → utwórz.
2. Usuń pierwotne zlecenie `ST-1001` (Wspólnota…) → potwierdź.
3. Dodaj kolejne zlecenie `Zlecenie B` (dane jak w kroku 1).
4. Kliknij „Wstrzymaj" lub „Usuń" przy pierwszym wierszu.

**Rezultat oczekiwany:** unikalne identyfikatory; akcja dotyczy wyłącznie wybranego wiersza.

**Rezultat rzeczywisty:** oba zlecenia mają identyfikator `ST-1002`; akcja „Wstrzymaj" zmienia
status pierwszego znalezionego zlecenia (może to być inny wiersz niż kliknięty), a „Usuń"
kasuje oba wiersze jednocześnie.

**Przyczyna (analiza kodu):** `'ST-' + (1000 + state.standing.length + 1)` — ten sam mechanizm
co w DEF-003.

**Propozycja poprawki:** jak w DEF-003 — licznik globalny, usuwanie i modyfikacja po unikalnym kluczu.

---

## DEF-007 — Ostrzeżenie o sesji pojawia się natychmiast przy limicie 1 minuty

| | |
|---|---|
| **Moduł** | Sesja |
| **Waga / priorytet** | Drobna / P3 |
| **Wymaganie** | FR-SES-02 |
| **Przypadek testowy** | TC-SES-009 |

**Kroki odtworzenia:**
1. Ustawienia → „Automatyczne wylogowanie" = **1 minuta**.
2. Nie wykonuj żadnych akcji.

**Rezultat oczekiwany:** ostrzeżenie na krótko przed końcem sesji (np. 15–30 s przy limicie 1 min).

**Rezultat rzeczywisty:** modal „Twoja sesja wkrótce wygaśnie" pojawia się po ok. **1 sekundzie**
z odliczaniem od 60 s — praktycznie natychmiast po zmianie ustawienia.

**Przyczyna:** próg ostrzeżenia jest liczony jako `limit − 60 s`, co dla limitu 60 s daje 0 s.
Dodatkowo aktywność użytkownika nie resetuje licznika, gdy ostrzeżenie jest już wyświetlone —
jedynym wyjściem jest przycisk „Przedłuż sesję".

**Propozycja poprawki:** próg ostrzeżenia jako `min(60, limit / 2)` lub ukrycie opcji 1 minuty.

---

## DEF-008 — Limity karty nie są zapisywane ani egzekwowane

| | |
|---|---|
| **Moduł** | Karty |
| **Waga / priorytet** | Drobna / P3 |
| **Wymaganie** | FR-CRD-07 |
| **Przypadek testowy** | TC-CRD-010 |

**Kroki odtworzenia:**
1. Karty → ustaw „Limit płatności internetowych" na `0 zł`.
2. „Zapisz limity" → autoryzuj kodem `123456`.
3. Sprawdź obiekt `state` w konsoli przeglądarki; wykonaj dowolną operację obciążającą rachunek.

**Rezultat oczekiwany:** limit zapisany w stanie aplikacji i uwzględniany przy operacjach.

**Rezultat rzeczywisty:** po autoryzacji wyświetlany jest wyłącznie komunikat „Limity karty
zostały zapisane."; wartość nie trafia do stanu aplikacji i nie ma wpływu na żadną operację.
Po odświeżeniu strony suwaki wracają do 3 000 / 2 000 zł.

**Wpływ:** funkcja pozoruje działanie — użytkownik jest informowany o zapisie, który nie następuje.

**Propozycja poprawki:** zapis limitów w `state.cards` (per karta) i weryfikacja przy operacjach
kartowych lub wyraźne oznaczenie funkcji jako demonstracyjnej.

---

## DEF-009 — Ustawienia powiadomień nie są zapisywane w stanie aplikacji

| | |
|---|---|
| **Moduł** | Ustawienia |
| **Waga / priorytet** | Drobna / P3 |
| **Wymaganie** | FR-SET-09 |
| **Przypadek testowy** | TC-SET-013 |

**Kroki odtworzenia:** Ustawienia → przełącz dowolny przełącznik powiadomień → sprawdź `state` w konsoli.

**Rezultat oczekiwany:** wartość zapisana w stanie aplikacji.
**Rezultat rzeczywisty:** wyświetlany jest wyłącznie toast „Ustawienia powiadomień zostały
zapisane."; stan przechowuje jedynie wartość w polu formularza.

**Propozycja poprawki:** zapis w `state.settings.notifications`.

---

## DEF-010 — Data zakończenia lokaty liczona jako miesiąc = 30 dni

| | |
|---|---|
| **Moduł** | Lokaty |
| **Waga / priorytet** | Drobna / P3 |
| **Wymaganie** | FR-DEP-06 |
| **Przypadek testowy** | TC-DEP-010 |

**Kroki odtworzenia:** otwórz lokatę na 6 miesięcy i odczytaj datę zakończenia.

**Rezultat oczekiwany:** data zakończenia = data otwarcia + 6 miesięcy kalendarzowych
(np. 14.08.2026 → 14.02.2027).
**Rezultat rzeczywisty:** data = dzisiaj + 180 dni (np. 10.02.2027) — różnica do 4 dni;
dla lokaty 24-miesięcznej różnica sięga ~24 dni, co przekłada się na inny okres naliczania odsetek
niż deklarowany.

**Propozycja poprawki:** `setMonth(getMonth() + m)` zamiast dodawania `m × 30 × 86400000` ms.

---

## DEF-011 — Niespójna tolerancja formatu kwoty

| | |
|---|---|
| **Moduł** | Formularze (przelewy, lokaty, kantor, zlecenia stałe) |
| **Waga / priorytet** | Kosmetyczna / P4 |
| **Wymaganie** | FR-TRF-05 |
| **Przypadek testowy** | TC-TRF-016 |

**Rezultat rzeczywisty:** kwota `1 000,50` (ze spacją) jest akceptowana, a `1.000,50`
(z kropką jako separatorem tysięcy — format kopiowany m.in. z arkuszy kalkulacyjnych) odrzucana
z komunikatem „Podaj poprawną kwotę." Aplikacja jednocześnie traktuje kropkę jako separator dziesiętny.

**Wpływ:** mylące dla użytkownika kopiującego kwoty z innych systemów.

**Propozycja poprawki:** normalizacja wejścia (usunięcie separatorów tysięcy) albo maska pola
z formatowaniem w locie i jednoznaczny komunikat o dozwolonym formacie.

---

## DEF-012 — Modale bez semantyki dialogu, pułapki fokusu i obsługi klawisza Esc

| | |
|---|---|
| **Moduł** | Elementy wspólne / dostępność |
| **Waga / priorytet** | Drobna / P3 |
| **Wymaganie** | FR-CMN-03, FR-CMN-08 |
| **Przypadek testowy** | TC-NFR-005 |

**Rezultat rzeczywisty:** żaden z modali (potwierdzenie, SMS, sesja, szczegóły operacji, wniosek
kredytowy, zlecenie stałe) nie ma atrybutów `role="dialog"` ani `aria-modal="true"`, fokus nie jest
przenoszony do modala ani w nim zatrzymywany (klawiszem Tab można przejść do elementów pod spodem),
a klawisz Esc nie zamyka okna. Osoba korzystająca z czytnika ekranu nie otrzymuje informacji
o otwarciu okna dialogowego.

**Propozycja poprawki:** dodać semantykę ARIA, przeniesienie i pułapkę fokusu, obsługę Esc
(z wyjątkiem modala sesji, który celowo wymaga decyzji użytkownika) oraz powrót fokusu
do elementu wywołującego po zamknięciu.

---

## DEF-013 — Odrzucona autoryzacja SMS nie kończy operacji

| | |
|---|---|
| **Moduł** | Autoryzacja operacji |
| **Waga / priorytet** | Drobna / P3 |
| **Wymaganie** | FR-TRF-11, FR-CMN-04 |
| **Przypadek testowy** | TC-TRF-030 |

**Kroki odtworzenia:**
1. Przygotuj przelew i przejdź do kroku 2.
2. „Zatwierdź i autoryzuj" → wpisz trzykrotnie błędny kod (operacja zostaje anulowana komunikatem).
3. Ponownie kliknij „Zatwierdź i autoryzuj".

**Rezultat oczekiwany:** po odrzuceniu autoryzacji operacja jest anulowana — powrót do formularza
lub blokada kolejnych prób dla tej operacji.

**Rezultat rzeczywisty:** przelew pozostaje na kroku 2, licznik prób jest zerowany i cykl
„3 próby" można powtarzać w nieskończoność.

**Propozycja poprawki:** po odrzuceniu autoryzacji powrót do kroku 1 (formularz) lub globalny
licznik odrzuceń w ramach sesji.

---

## Obserwacje niebędące defektami (do potwierdzenia z właścicielem produktu)

| Obserwacja | Uzasadnienie |
|---|---|
| Dane logowania i kod SMS są widoczne na ekranie logowania i w toastach | Zamierzone w środowisku szkoleniowym; musi zostać usunięte przed jakimkolwiek wdrożeniem produkcyjnym |
| Brak trwałości danych (F5 przywraca stan startowy) | Zamierzone — brak backendu |
| Wymiana walut wyłącznie dla pary PLN ⇄ EUR | Udokumentowane w interfejsie |
| Kursy walut są statyczne mimo etykiety „NBP + marża" | Uproszczenie symulacji; etykieta może wprowadzać w błąd |
| Wnioski kredytowe i zgłoszenia kontaktowe nie tworzą rekordów | Zamierzona symulacja |
| Kod BLIK nigdy nie zaczyna się od cyfry 0 (zakres 100000–999999) | Odstępstwo od realnego BLIK-a; bez wpływu funkcjonalnego |
| Przelew własny nie wymaga autoryzacji SMS | Zgodne z praktyką rynkową dla przelewów między własnymi rachunkami |
