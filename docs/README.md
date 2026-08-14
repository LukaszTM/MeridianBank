# Dokumentacja testowa — Meridian Bank 2.6.0

Komplet dokumentacji QA dla aplikacji szkoleniowej **Meridian Bank** (bankowość internetowa,
środowisko testowe QA). Dokumentacja powstała na podstawie analizy kodu źródłowego `index.html`
oraz weryfikacji zachowania aplikacji w przeglądarce.

## Spis dokumentów

| Dokument | Zawartość | Dla kogo |
|---|---|---|
| [01 — Specyfikacja funkcjonalna](01-specyfikacja-funkcjonalna.md) | Opis wszystkich funkcji i reguł biznesowych aplikacji, 143 wymagania `FR-*`, dane startowe, znane ograniczenia | QA, deweloperzy, właściciel produktu |
| [02 — Plan testów](02-plan-testow.md) | Zakres, podejście, kryteria wejścia i wyjścia, środowisko, harmonogram, ryzyka, metryki, zarządzanie defektami | Kierownik testów, interesariusze |
| [03 — Przypadki testowe](03-przypadki-testowe.md) | 212 przypadków testowych z krokami i rezultatami oczekiwanymi + macierz pokrycia wymagań | Inżynierowie QA |
| [04 — Dane testowe i selektory](04-dane-testowe-i-selektory.md) | Konta, dane brzegowe do walidacji, wartości referencyjne obliczeń, katalog `data-testid`, czasy oczekiwania | QA manualne i automatyzujące |
| [05 — Raport defektów](05-raport-defektow.md) | 13 defektów potwierdzonych wykonaniem, z krokami odtworzenia, analizą przyczyny i propozycją poprawki | Deweloperzy, właściciel produktu |
| [06 — Szablony i checklisty](06-szablony-i-checklisty.md) | Checklisty smoke i regresji, szablony zgłoszenia defektu i raportów, karty testów eksploracyjnych | Cały zespół |

## Jak zacząć

1. **Uruchom aplikację** — zalecany serwer statyczny (protokół `file://` blokuje API schowka):
   ```bash
   python3 -m http.server 8000
   # następnie otwórz http://localhost:8000
   ```
2. **Zaloguj się:** `jan.kowalski` / `Test123!`, kod SMS `123456`.
3. **Wykonaj testy dymne** — checklista w [dokumencie 06](06-szablony-i-checklisty.md), rozdz. 1.
4. **Prowadź testy właściwe** wg [przypadków testowych](03-przypadki-testowe.md), zaczynając od
   przypadków o priorytecie krytycznym (przelewy, autoryzacja, spójność sald).
5. **Zgłaszaj defekty** wg szablonu z [dokumentu 06](06-szablony-i-checklisty.md), rozdz. 3.

> **Uwaga:** aplikacja nie przechowuje danych — odświeżenie strony (F5) przywraca stan startowy.
> Każdy przypadek testowy zakłada rozpoczęcie od świeżego stanu, chyba że zapisano inaczej.

## Stan jakości na dzień 2026-08-14

Analiza i weryfikacja wykazały **13 defektów**, w tym **3 krytyczne** dotyczące poprawności
operacji finansowych:

| ID | Defekt | Skutek |
|---|---|---|
| DEF-001 | Przelew krajowy zawsze obciąża konto osobiste | Obciążenie niewłaściwego rachunku, saldo ujemne |
| DEF-002 | Przelew zaplanowany nie rezerwuje środków i nie jest realizowany | Zlecenia bez pokrycia, funkcja nie działa |
| DEF-003 | Zduplikowane identyfikatory lokat | Zerwanie jednej lokaty usuwa dwie i zwraca jeden kapitał |

Zgodnie z kryteriami wyjścia z [planu testów](02-plan-testow.md) (rozdz. 4.2) wersja 2.6.0
**nie kwalifikuje się do wydania** do czasu naprawy defektów krytycznych i wykonania retestów.
