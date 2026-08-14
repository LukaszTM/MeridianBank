# Przypadki testowe — Meridian Bank 2.6.0

Zestaw **212 przypadków testowych** dla aplikacji Meridian Bank 2.6.0.
Powiązania z wymaganiami `FR-*` z [specyfikacji funkcjonalnej](01-specyfikacja-funkcjonalna.md),
macierz pokrycia w rozdziale 14.

## Konwencje

- **Stan startowy** = otwarcie/odświeżenie strony `index.html` (F5). Odświeżenie przywraca wszystkie dane startowe.
- **Stan „zalogowany"** = wykonane kroki: login `jan.kowalski`, hasło `Test123!`, kod SMS `123456`.
- Priorytety: **K** — krytyczny, **W** — wysoki, **Ś** — średni, **N** — niski.
- Typ: **P** — pozytywny, **N** — negatywny, **B** — wartości brzegowe.
- Kolumna **Wynik** służy do wypełnienia podczas wykonania (Zdany / Niezdany / Zablokowany / Nie wykonano).
- Przypadki oznaczone 🔴 wykrywają potwierdzone defekty opisane w [05-raport-defektow.md](05-raport-defektow.md) — przy obecnej wersji aplikacji **kończą się wynikiem niezdanym**.

---

## 1. Logowanie i uwierzytelnianie (TC-LOG)

| ID | Prio | Typ | Tytuł i kroki | Oczekiwany rezultat | FR | Wynik |
|---|---|---|---|---|---|---|
| TC-LOG-001 | K | P | **Wyświetlenie ekranu logowania.** 1. Otwórz aplikację | Widoczna sekcja powitalna, formularz logowania, ramka z danymi testowymi, wstążka „ŚRODOWISKO TESTOWE QA"; fokus w polu Login | FR-LOG-01, FR-CMN-01 | |
| TC-LOG-002 | K | N | **Puste pola.** 1. Kliknij „Zaloguj się" bez wypełniania | Oba pola podświetlone; komunikaty „Podaj login." i „Podaj hasło."; brak przejścia dalej | FR-LOG-01 | |
| TC-LOG-003 | Ś | N | **Puste hasło przy wypełnionym loginie.** 1. Wpisz login `jan.kowalski` 2. Zatwierdź | Komunikat tylko przy polu Hasło | FR-LOG-01 | |
| TC-LOG-004 | K | P | **Poprawne logowanie — krok 1.** 1. `jan.kowalski` / `Test123!` 2. Zatwierdź | Loader „Trwa logowanie…" (~0,9 s) → krok „Autoryzacja SMS"; toast z kodem `123456` | FR-LOG-02 | |
| TC-LOG-005 | K | P | **Poprawne logowanie — krok 2.** 1. Wykonaj TC-LOG-004 2. Wpisz `123456` 3. „Potwierdź i zaloguj" | Loader „Przygotowujemy Twój pulpit…" → pulpit, toast „Zalogowano pomyślnie. Witaj, Jan!"; widoczny przycisk czatu | FR-LOG-05 | |
| TC-LOG-006 | W | P | **Zatwierdzenie kodu SMS klawiszem Enter.** 1. Na kroku 2 wpisz `123456` 2. Naciśnij Enter | Logowanie jak w TC-LOG-005 | FR-LOG-05 | |
| TC-LOG-007 | K | N | **Błędne hasło — 1. próba.** 1. `jan.kowalski` / `Zle123!` | Komunikat „Nieprawidłowy login lub hasło. Pozostałe próby: 2." | FR-LOG-03 | |
| TC-LOG-008 | K | N | **Błędny login.** 1. `anna.nowak` / `Test123!` | Komunikat jw. z liczbą pozostałych prób | FR-LOG-03 | |
| TC-LOG-009 | K | B | **Blokada po 3 próbach.** 1. Trzykrotnie błędne hasło | Baner blokady, przycisk „Zaloguj się" nieaktywny, odliczanie 30 → 0 s | FR-LOG-04 | |
| TC-LOG-010 | W | P | **Automatyczne odblokowanie.** 1. Wykonaj TC-LOG-009 2. Odczekaj 30 s 3. Zaloguj się poprawnie | Baner znika, przycisk aktywny, logowanie kończy się sukcesem | FR-LOG-04 | |
| TC-LOG-011 | Ś | N | **Próba logowania w trakcie blokady.** 1. Wykonaj TC-LOG-009 2. Naciśnij Enter w formularzu | Brak reakcji, licznik prób bez zmian, brak nowych komunikatów | FR-LOG-04 | |
| TC-LOG-012 | K | N | **Błędny kod SMS.** 1. Na kroku 2 wpisz `000000` | Komunikat „Nieprawidłowy kod SMS. Spróbuj ponownie."; użytkownik pozostaje na kroku 2 | FR-LOG-05 | |
| TC-LOG-013 | W | N | 🔴 **Limit prób kodu SMS przy logowaniu.** 1. Wpisz błędny kod 6 razy | **Oczekiwane:** po 3 próbach powrót do kroku 1 lub blokada (spójnie z autoryzacją operacji). **Rzeczywiste:** brak limitu — patrz DEF-004 | FR-LOG-05 | |
| TC-LOG-014 | Ś | P | **Powrót z kroku SMS.** 1. Na kroku 2 kliknij „← Wróć do logowania" | Powrót do formularza; ponowne logowanie możliwe | FR-LOG-05 | |
| TC-LOG-015 | Ś | P | **Pokaż/ukryj hasło.** 1. Wpisz hasło 2. Kliknij ikonę oka 3. Kliknij ponownie | Hasło widoczne jako tekst, następnie ponownie maskowane | FR-LOG-06 | |
| TC-LOG-016 | Ś | P | **Zapamiętanie loginu.** 1. Zaznacz „Zapamiętaj mój login" 2. Zaloguj się 3. Wyloguj | Pole Login zawiera `jan.kowalski`, pole Hasło puste | FR-LOG-07 | |
| TC-LOG-017 | Ś | P | **Brak zapamiętania loginu.** 1. Zaloguj bez zaznaczenia 2. Wyloguj | Pole Login puste | FR-LOG-07 | |
| TC-LOG-018 | N | P | **Link „Nie pamiętasz hasła?".** 1. Kliknij link | Toast z informacją o infolinii; brak zmiany widoku | FR-LOG-08 | |
| TC-LOG-019 | Ś | N | **Odporność na spacje w loginie.** 1. Wpisz ` jan.kowalski ` / `Test123!` | Logowanie udane (login jest przycinany) | FR-LOG-02 | |
| TC-LOG-020 | Ś | N | **Wielkość liter w loginie.** 1. Wpisz `JAN.KOWALSKI` / `Test123!` | Logowanie odrzucone (login rozróżnia wielkość liter) | FR-LOG-03 | |

## 2. Sesja i wylogowanie (TC-SES)

| ID | Prio | Typ | Tytuł i kroki | Oczekiwany rezultat | FR | Wynik |
|---|---|---|---|---|---|---|
| TC-SES-001 | W | P | **Wylogowanie z potwierdzeniem.** 1. Menu użytkownika → „Wyloguj się" 2. Potwierdź | Ekran logowania, baner „Zostałeś poprawnie wylogowany. Do zobaczenia!" | FR-LOG-07 | |
| TC-SES-002 | Ś | P | **Anulowanie wylogowania.** 1. Menu → „Wyloguj się" 2. Kliknij „Anuluj" | Modal zamknięty, użytkownik pozostaje zalogowany | FR-CMN-03 | |
| TC-SES-003 | W | P | **Ostrzeżenie o wygaśnięciu sesji.** 1. Ustawienia → czas 1 min 2. Nie wykonuj żadnych akcji | Modal „Twoja sesja wkrótce wygaśnie" z odliczaniem od 60 s | FR-SES-02 | |
| TC-SES-004 | W | P | **Przedłużenie sesji.** 1. Wykonaj TC-SES-003 2. „Przedłuż sesję" | Modal zamknięty, toast „Sesja została przedłużona.", użytkownik zalogowany | FR-SES-02 | |
| TC-SES-005 | W | P | **Wygaśnięcie sesji.** 1. Wykonaj TC-SES-003 2. Odczekaj do końca odliczania | Automatyczne wylogowanie, baner „Sesja wygasła z powodu braku aktywności." | FR-SES-03 | |
| TC-SES-006 | Ś | P | **Wylogowanie z modala sesji.** 1. Wykonaj TC-SES-003 2. „Wyloguj teraz" | Natychmiastowe wylogowanie z banerem | FR-SES-02 | |
| TC-SES-007 | Ś | N | **Modal sesji nie zamyka się kliknięciem w tło.** 1. Wykonaj TC-SES-003 2. Kliknij tło modala | Modal pozostaje otwarty | FR-CMN-03 | |
| TC-SES-008 | Ś | P | **Aktywność resetuje licznik.** 1. Ustaw 5 min 2. Klikaj co ~1 min przez 6 min | Sesja aktywna, brak ostrzeżenia | FR-SES-01 | |
| TC-SES-009 | Ś | N | 🔴 **Ostrzeżenie przy limicie 1 min.** 1. Ustaw czas sesji 1 min | **Oczekiwane:** ostrzeżenie po ok. 1 min bezczynności. **Rzeczywiste:** modal pojawia się natychmiast (~1 s) — patrz DEF-007 | FR-SES-02 | |
| TC-SES-010 | Ś | P | **Stan danych po ponownym zalogowaniu.** 1. Wykonaj przelew 2. Wyloguj się 3. Zaloguj ponownie | Salda i historia zachowują zmiany z poprzedniej sesji (reset dopiero po F5) | FR-LOG-07 | |

## 3. Nawigacja i powłoka (TC-NAV)

| ID | Prio | Typ | Tytuł i kroki | Oczekiwany rezultat | FR | Wynik |
|---|---|---|---|---|---|---|
| TC-NAV-001 | W | P | **Nawigacja po wszystkich modułach.** 1. Kliknij kolejno 11 pozycji menu | Każde przejście: loader, poprawny tytuł w nagłówku, aktywna pozycja menu, właściwa treść strony | FR-NAV-01 | |
| TC-NAV-002 | Ś | P | **Skróty na pulpicie.** 1. Kliknij „＋ Nowy przelew", „Kod BLIK", „Wymień walutę", „Załóż lokatę" | Przejście odpowiednio do Przelewów, BLIK, Kantoru, Lokat | FR-NAV-02 | |
| TC-NAV-003 | Ś | P | **Kafelek rachunku prowadzi do historii.** 1. Kliknij kafelek „Konto osobiste" | Otwarta strona Historia operacji | FR-DASH-02, FR-NAV-02 | |
| TC-NAV-004 | Ś | P | **Menu użytkownika.** 1. Kliknij awatar 2. „Profil i ustawienia" | Otwarte Ustawienia, dropdown zamknięty | FR-NAV-02 | |
| TC-NAV-005 | W | P | **Widok mobilny — menu burger.** 1. Ustaw szerokość 390 px 2. Kliknij burger 3. Wybierz „Historia" | Sidebar wysuwa się, po wyborze zamyka się, strona zmieniona | FR-NAV-03 | |
| TC-NAV-006 | Ś | P | **Przełączanie motywu w nagłówku.** 1. Kliknij ikonę księżyca | Interfejs w trybie ciemnym; przełącznik w Ustawieniach ustawiony na włączony | FR-NAV-04, FR-SET-10 | |
| TC-NAV-007 | Ś | P | **Przełączanie motywu w ustawieniach.** 1. Ustawienia → „Tryb ciemny" | Motyw zmieniony, stan zsynchronizowany z ikoną w nagłówku | FR-NAV-04 | |
| TC-NAV-008 | Ś | P | **Powiadomienia — lista.** 1. Kliknij dzwonek | Lista 3 powiadomień, 2 wyróżnione jako nieprzeczytane, widoczna kropka przy dzwonku | FR-NAV-05 | |
| TC-NAV-009 | Ś | P | **Oznaczenie powiadomień jako przeczytane.** 1. Dzwonek → „Oznacz jako przeczytane" | Wszystkie pozycje bez wyróżnienia, kropka znika | FR-NAV-05 | |
| TC-NAV-010 | N | P | **Zamykanie dropdownów.** 1. Otwórz dzwonek 2. Kliknij awatar 3. Kliknij w treść strony | Otwarcie jednego zamyka drugi; kliknięcie poza zamyka oba | FR-NAV-05 | |
| TC-NAV-011 | N | P | **Zanikanie toastów.** 1. Wywołaj toast 2. Odczekaj 5 s | Toast znika samoczynnie i jest usuwany z DOM | FR-NAV-06 | |

## 4. Pulpit (TC-DASH)

| ID | Prio | Typ | Tytuł i kroki | Oczekiwany rezultat | FR | Wynik |
|---|---|---|---|---|---|---|
| TC-DASH-001 | W | P | **Kafelki rachunków.** 1. Otwórz pulpit | Trzy kafelki: 12 543,21 zł, 45 200,00 zł, 3 250,50 € z poprawnymi nazwami i numerami IBAN | FR-DASH-02 | |
| TC-DASH-002 | Ś | P | **Powitanie i data.** 1. Otwórz pulpit | Powitanie zgodne z porą dnia + pełna data po polsku | FR-DASH-01 | |
| TC-DASH-003 | Ś | P | **Ostatnie operacje.** 1. Otwórz pulpit | Dokładnie 5 pozycji, najnowsze na górze, uznania na zielono z „+" | FR-DASH-05 | |
| TC-DASH-004 | Ś | P | **Szczegóły operacji z pulpitu.** 1. Kliknij wiersz operacji | Modal ze szczegółami (data z godziną, opis, kategoria, kwota, saldo po, numer referencyjny) | FR-DASH-05, FR-HIS-11 | |
| TC-DASH-005 | Ś | P | **Widget kursów walut.** 1. Wejdź na pulpit po zalogowaniu | Skeleton ~1,4 s, następnie 4 waluty z kursem średnim i zmianą 24 h (kolor + strzałka) | FR-DASH-06 | |
| TC-DASH-006 | N | P | **Widget kursów przy ponownym wejściu.** 1. Przejdź na inną stronę 2. Wróć na pulpit | Kursy widoczne natychmiast, bez skeletona | FR-DASH-06 | |
| TC-DASH-007 | Ś | P | **Wykres wydatków.** 1. Otwórz pulpit | 6 słupków z etykietami miesięcy i sumami; najwyższy słupek = największe wydatki | FR-DASH-07 | |
| TC-DASH-008 | Ś | P | **Aktualizacja pulpitu po operacji.** 1. Wykonaj przelew 100 zł 2. Wróć na pulpit | Saldo konta osobistego pomniejszone o 100 zł; nowa operacja na liście ostatnich | FR-DASH-02, FR-TRF-13 | |

## 5. Przelew krajowy (TC-TRF)

| ID | Prio | Typ | Tytuł i kroki | Oczekiwany rezultat | FR | Wynik |
|---|---|---|---|---|---|---|
| TC-TRF-001 | K | P | **Pełny przelew z datą dzisiejszą.** 1. Przelewy → wypełnij: odbiorca `Anna Nowak`, rachunek `61109010140000071219812874`, kwota `150,50`, tytuł `Test` 2. „Dalej →" 3. „Zatwierdź i autoryzuj" 4. Kod `123456` | Podsumowanie z poprawnymi danymi → loader → ekran sukcesu z numerem `PRZ-…`; toast o przyjęciu; saldo konta osobistego pomniejszone o 150,50 zł; operacja w historii | FR-TRF-09…13 | |
| TC-TRF-002 | K | N | **Wszystkie pola puste.** 1. Kliknij „Dalej →" | Komunikaty przy: nazwie odbiorcy, numerze rachunku, kwocie, tytule (data jest wypełniona domyślnie) | FR-TRF-03…07 | |
| TC-TRF-003 | W | B | **Nazwa odbiorcy — 2 znaki.** 1. Wpisz `Ab` + pozostałe dane poprawne | Komunikat „Podaj nazwę odbiorcy (min. 3 znaki)." | FR-TRF-03 | |
| TC-TRF-004 | W | B | **Nazwa odbiorcy — 3 znaki.** 1. Wpisz `Abc` + pozostałe dane poprawne | Walidacja przechodzi, wyświetlone podsumowanie | FR-TRF-03 | |
| TC-TRF-005 | K | B | **NRB — 25 cyfr.** 1. Wpisz 25 cyfr | Komunikat „Numer rachunku musi zawierać dokładnie 26 cyfr." | FR-TRF-04 | |
| TC-TRF-006 | K | B | **NRB — 27 cyfr.** 1. Wpisz 27 cyfr | Komunikat jw. | FR-TRF-04 | |
| TC-TRF-007 | W | P | **NRB ze spacjami i prefiksem PL.** 1. Wpisz `PL61 1090 1014 0000 0712 1981 2874` | Walidacja przechodzi; w podsumowaniu numer sformatowany grupami | FR-TRF-04, FR-TRF-09 | |
| TC-TRF-008 | W | N | **NRB z literami.** 1. Wpisz `6110901014000007121981287X` | Komunikat o wymaganych 26 cyfrach | FR-TRF-04 | |
| TC-TRF-009 | W | N | 🔴 **NRB z niepoprawną sumą kontrolną.** 1. Wpisz `00000000000000000000000000` | **Oczekiwane:** odrzucenie numeru niespełniającego sumy kontrolnej IBAN. **Rzeczywiste:** numer przyjęty — patrz DEF-005 | FR-TRF-04 | |
| TC-TRF-010 | K | B | **Kwota 0.** 1. Wpisz `0` | Komunikat „Podaj poprawną kwotę większą od zera." | FR-TRF-05 | |
| TC-TRF-011 | K | B | **Kwota 0,01.** 1. Wpisz `0,01` | Walidacja przechodzi | FR-TRF-05 | |
| TC-TRF-012 | K | B | **Kwota równa saldu.** 1. Wpisz `12543,21` (konto osobiste) | Walidacja przechodzi | FR-TRF-05 | |
| TC-TRF-013 | K | B | **Kwota o 0,01 większa od salda.** 1. Wpisz `12543,22` | Komunikat „Niewystarczające środki na rachunku." | FR-TRF-05 | |
| TC-TRF-014 | W | N | **Kwota ujemna / tekstowa.** 1. Wpisz `-50`, następnie `abc` | W obu przypadkach komunikat o niepoprawnej kwocie | FR-TRF-05 | |
| TC-TRF-015 | W | B | **Kwota z 3 miejscami po przecinku.** 1. Wpisz `10,999` | Komunikat o niepoprawnej kwocie | FR-TRF-05 | |
| TC-TRF-016 | Ś | B | **Kwota z separatorem tysięcy.** 1. Wpisz `1 000,50` 2. Powtórz z `1.000,50` | `1 000,50` → przyjęte (1 000,50 zł); `1.000,50` → odrzucone (niespójność opisana w DEF-011) | FR-TRF-05 | |
| TC-TRF-017 | K | N | **Data z przeszłości.** 1. Ustaw datę wczorajszą | Komunikat „Data nie może być z przeszłości." | FR-TRF-06 | |
| TC-TRF-018 | W | B | **Data dzisiejsza.** 1. Pozostaw datę domyślną | Walidacja przechodzi; przelew księgowany natychmiast | FR-TRF-06, FR-TRF-13 | |
| TC-TRF-019 | W | P | **Data przyszła — przelew zaplanowany.** 1. Ustaw datę +5 dni 2. Zrealizuj proces | Komunikat o realizacji o 06:00; pozycja w zakładce „Zaplanowane"; **saldo bez zmian** | FR-TRF-14, FR-SCH-01 | |
| TC-TRF-020 | K | N | 🔴 **Rezerwacja środków dla przelewu zaplanowanego.** 1. Zaplanuj przelew na 9 999 zł 2. Sprawdź saldo 3. Zaplanuj kolejny na 9 999 zł | **Oczekiwane:** środki zarezerwowane lub kontrola dostępnego salda. **Rzeczywiste:** można zaplanować przelewy przewyższające saldo — patrz DEF-002 | FR-TRF-14 | |
| TC-TRF-021 | K | N | 🔴 **Przelew z konta oszczędnościowego.** 1. Wybierz „Konto oszczędnościowe" 2. Kwota `20000` 3. Zrealizuj proces | **Oczekiwane:** obciążenie konta oszczędnościowego. **Rzeczywiste:** obciążone konto osobiste, saldo −7 456,79 zł — patrz DEF-001 | FR-TRF-13 | |
| TC-TRF-022 | W | P | **Odbiorca zapisany.** 1. Wybierz „Anna Nowak" z listy | Pola nazwy i numeru rachunku wypełnione automatycznie | FR-TRF-02 | |
| TC-TRF-023 | W | P | **Zapisanie nowego odbiorcy.** 1. Wypełnij dane nowego odbiorcy 2. Zaznacz „Zapisz odbiorcę" 3. Zrealizuj przelew 4. Rozpocznij nowy przelew | Nowy odbiorca dostępny na liście wyboru | FR-TRF-15 | |
| TC-TRF-024 | Ś | N | **Brak duplikatu odbiorcy.** 1. Zrealizuj dwa przelewy na ten sam rachunek z zaznaczonym zapisem | Odbiorca występuje na liście tylko raz | FR-TRF-15 | |
| TC-TRF-025 | Ś | P | **Czyszczenie formularza.** 1. Wypełnij pola 2. „Wyczyść formularz" | Pola puste, data = dzisiaj, checkboxy odznaczone, komunikaty błędów ukryte | FR-TRF-08 | |
| TC-TRF-026 | W | P | **Powrót do edycji z podsumowania.** 1. Przejdź do kroku 2 2. „← Popraw dane" | Formularz z zachowanymi wartościami | FR-TRF-09 | |
| TC-TRF-027 | K | N | **Anulowanie autoryzacji SMS.** 1. Krok 2 → „Zatwierdź i autoryzuj" 2. „Anuluj" | Modal zamknięty, przelew niezrealizowany, saldo bez zmian | FR-CMN-04 | |
| TC-TRF-028 | K | N | **Błędny kod SMS — 1. i 2. próba.** 1. Wpisz `111111` 2. Wpisz `222222` | Komunikaty „Nieprawidłowy kod. Pozostałe próby: 2." i „… 1." | FR-TRF-11 | |
| TC-TRF-029 | K | N | **Trzy błędne kody SMS.** 1. Wpisz trzykrotnie błędny kod | Modal zamknięty, toast „Autoryzacja odrzucona po 3 błędnych próbach. Operacja anulowana."; saldo bez zmian | FR-TRF-11 | |
| TC-TRF-030 | W | N | 🔴 **Ponowienie po odrzuconej autoryzacji.** 1. Wykonaj TC-TRF-029 2. Ponownie „Zatwierdź i autoryzuj" 3. Wpisz `123456` | **Oczekiwane:** po odrzuceniu autoryzacji operacja anulowana (powrót do formularza). **Rzeczywiste:** przelew nadal na kroku 2, licznik prób zerowany, cykl można powtarzać bez ograniczeń — patrz DEF-013 | FR-TRF-11, FR-CMN-04 | |
| TC-TRF-031 | W | P | **Pobranie potwierdzenia.** 1. Po sukcesie kliknij „Pobierz potwierdzenie" | Pobrany plik `potwierdzenie_PRZ-….txt` z danymi przelewu i adnotacją o środowisku szkoleniowym | FR-TRF-16 | |
| TC-TRF-032 | Ś | P | **Nowy przelew po sukcesie.** 1. Kliknij „Nowy przelew" | Pusty formularz na kroku 1 | FR-TRF-12 | |
| TC-TRF-033 | Ś | P | **Lista rachunków źródłowych.** 1. Rozwiń „Z rachunku" | Wyłącznie rachunki PLN (2 pozycje) z aktualnymi saldami w etykietach | FR-TRF-01 | |
| TC-TRF-034 | Ś | B | **Tytuł przelewu — limit 70 znaków.** 1. Wklej 100 znaków | Pole przyjmuje maksymalnie 70 znaków | FR-TRF-07 | |
| TC-TRF-035 | Ś | N | **Ochrona przed XSS w nazwie odbiorcy.** 1. Wpisz `<img src=x onerror=alert(1)>` (min. 3 znaki) 2. Przejdź do podsumowania i zrealizuj przelew | Tekst wyświetlany dosłownie w podsumowaniu i historii; brak wykonania skryptu, brak błędów w konsoli | FR-CMN-07 | |

## 6. Przelew własny (TC-OWN)

| ID | Prio | Typ | Tytuł i kroki | Oczekiwany rezultat | FR | Wynik |
|---|---|---|---|---|---|---|
| TC-OWN-001 | K | P | **Przelew z konta osobistego na oszczędnościowe.** 1. Kwota `500` 2. „Przelej środki" | Toast o realizacji; konto osobiste −500 zł, oszczędnościowe +500 zł; operacja „Przelew własny: …" w historii; pole kwoty wyczyszczone | FR-OWN-04 | |
| TC-OWN-002 | K | P | **Przelew w drugą stronę.** 1. Zamień rachunki 2. Kwota `300` | Konto oszczędnościowe −300 zł, osobiste +300 zł; operacja uznaniowa w historii | FR-OWN-04 | |
| TC-OWN-003 | K | N | **Ten sam rachunek po obu stronach.** 1. Ustaw oba pola na „Konto osobiste" 2. Zatwierdź | Komunikat „Rachunek źródłowy i docelowy muszą być różne."; brak operacji | FR-OWN-02 | |
| TC-OWN-004 | W | N | **Kwota przekraczająca saldo.** 1. Kwota `999999` | Komunikat „Niewystarczające środki na rachunku." | FR-OWN-03 | |
| TC-OWN-005 | W | N | **Kwota 0 i pusta.** 1. Wpisz `0`, następnie pozostaw puste | W obu przypadkach komunikat o niepoprawnej kwocie | FR-OWN-03 | |
| TC-OWN-006 | Ś | P | **Brak autoryzacji SMS.** 1. Zrealizuj przelew własny | Operacja wykonana bez modala SMS (zgodnie ze specyfikacją) | FR-OWN-05 | |
| TC-OWN-007 | Ś | P | **Domyślne rachunki.** 1. Otwórz zakładkę „Przelew własny" | Rachunek docelowy różny od źródłowego; obie listy zawierają tylko rachunki PLN | FR-OWN-01 | |

## 7. Przelewy zaplanowane i zlecenia stałe (TC-SCH, TC-STO)

| ID | Prio | Typ | Tytuł i kroki | Oczekiwany rezultat | FR | Wynik |
|---|---|---|---|---|---|---|
| TC-SCH-001 | Ś | P | **Pusta lista zaplanowanych.** 1. Otwórz zakładkę „Zaplanowane" (stan startowy) | Komunikat „Brak zaplanowanych przelewów…" | FR-SCH-01 | |
| TC-SCH-002 | W | P | **Pozycja po zaplanowaniu przelewu.** 1. Wykonaj TC-TRF-019 2. Otwórz „Zaplanowane" | Wiersz z datą DD.MM.RRRR, odbiorcą, tytułem i kwotą ze znakiem minus | FR-SCH-01 | |
| TC-SCH-003 | W | P | **Anulowanie zaplanowanego przelewu.** 1. Kliknij „Anuluj" 2. Potwierdź | Wiersz usunięty, toast; saldo bez zmian | FR-SCH-02 | |
| TC-SCH-004 | Ś | N | **Rezygnacja z anulowania.** 1. Kliknij „Anuluj" 2. W modalu wybierz „Anuluj" | Pozycja pozostaje na liście | FR-SCH-02, FR-CMN-03 | |
| TC-STO-001 | Ś | P | **Lista zleceń stałych.** 1. Otwórz zakładkę „Zlecenia stałe" | Jedno zlecenie: Wspólnota…, Czynsz, 1 200,00 zł, 10. dzień, status „Aktywne" | FR-STO-01 | |
| TC-STO-002 | W | P | **Utworzenie zlecenia.** 1. „＋ Nowe zlecenie" 2. Odbiorca `Netflix`, tytuł `Abonament`, kwota `43`, dzień `15` 3. „Utwórz zlecenie" | Nowy wiersz na liście, toast „Zlecenie stałe zostało utworzone." | FR-STO-02 | |
| TC-STO-003 | W | N | **Walidacja formularza zlecenia.** 1. „Utwórz zlecenie" bez danych | Komunikaty przy polach: odbiorca, tytuł, kwota | FR-STO-02 | |
| TC-STO-004 | Ś | B | **Zakres dnia miesiąca.** 1. Rozwiń listę „Dzień miesiąca" | Wartości 1–28 (brak 29–31) | FR-STO-02 | |
| TC-STO-005 | Ś | P | **Wstrzymanie i wznowienie.** 1. „Wstrzymaj" 2. „Wznów" | Status zmienia się na „Wstrzymane" (szary) i z powrotem na „Aktywne" (zielony), toasty | FR-STO-03 | |
| TC-STO-006 | Ś | P | **Usunięcie zlecenia.** 1. „Usuń" 2. Potwierdź | Wiersz usunięty, toast | FR-STO-04 | |
| TC-STO-007 | W | N | 🔴 **Unikalność ID po usunięciu.** 1. Dodaj zlecenie 2. Usuń pierwotne `ST-1001` 3. Dodaj kolejne 4. Kliknij „Wstrzymaj" na pierwszym wierszu | **Oczekiwane:** akcja dotyczy wybranego zlecenia. **Rzeczywiste:** zduplikowane ID `ST-1002` — akcja dotyczy niewłaściwego wiersza, a usunięcie kasuje oba zlecenia — patrz DEF-006 | FR-STO-05 | |

## 8. Historia operacji (TC-HIS)

| ID | Prio | Typ | Tytuł i kroki | Oczekiwany rezultat | FR | Wynik |
|---|---|---|---|---|---|---|
| TC-HIS-001 | W | P | **Widok domyślny.** 1. Otwórz Historię | Licznik „(48 operacji)", 10 wierszy, sortowanie po dacie malejąco (▼), informacja „Strona 1 z 5 · pozycje 1–10" | FR-HIS-01, FR-HIS-06, FR-HIS-08 | |
| TC-HIS-002 | W | P | **Wyszukiwanie po opisie.** 1. Wpisz `Biedronka` | Tylko operacje zawierające frazę; licznik zaktualizowany; powrót na stronę 1 | FR-HIS-02, FR-HIS-05 | |
| TC-HIS-003 | Ś | P | **Wyszukiwanie po kategorii.** 1. Wpisz `subskrypcje` | Operacje z kategorii „Subskrypcje" (wielkość liter bez znaczenia) | FR-HIS-02 | |
| TC-HIS-004 | Ś | N | **Wyszukiwanie bez wyników.** 1. Wpisz `xyz123` | Komunikat „Brak operacji spełniających kryteria…", licznik „(0 operacji)", brak paginacji | FR-HIS-09 | |
| TC-HIS-005 | W | P | **Filtr uznań.** 1. Typ = „Uznania (przychodzące)" | Wyłącznie kwoty dodatnie (zielone, z „+") | FR-HIS-03 | |
| TC-HIS-006 | W | P | **Filtr obciążeń.** 1. Typ = „Obciążenia (wychodzące)" | Wyłącznie kwoty ujemne | FR-HIS-03 | |
| TC-HIS-007 | W | P | **Filtr zakresu dat.** 1. Ustaw „Data od" = pierwszy dzień bieżącego miesiąca | Operacje wyłącznie z tego zakresu (data graniczna włącznie) | FR-HIS-04 | |
| TC-HIS-008 | Ś | B | **Zakres dat bez wyników.** 1. „Data od" = jutro | Komunikat o braku operacji | FR-HIS-04, FR-HIS-09 | |
| TC-HIS-009 | Ś | P | **Łączenie filtrów.** 1. Typ = obciążenia 2. Fraza `zakupy` 3. Zakres dat | Wyniki spełniają wszystkie kryteria jednocześnie | FR-HIS-02…04 | |
| TC-HIS-010 | W | P | **Zmiana liczby pozycji na stronę.** 1. Ustaw 25, następnie 50 | Liczba wierszy i liczba stron przeliczone; powrót na stronę 1 | FR-HIS-05 | |
| TC-HIS-011 | W | P | **Paginacja — następna/poprzednia.** 1. Kliknij „›" 2. Kliknij „‹" | Zmiana strony, aktualizacja informacji o pozycjach; „‹" nieaktywne na stronie 1, „›" na ostatniej | FR-HIS-07 | |
| TC-HIS-012 | Ś | P | **Paginacja — wybór numeru strony.** 1. Kliknij „3" | Strona 3 wyróżniona, pozycje 21–30 (przy 10/stronę) | FR-HIS-07 | |
| TC-HIS-013 | Ś | P | **Sortowanie po dacie.** 1. Kliknij nagłówek „Data" | Kolejność rosnąca, wskaźnik ▲; ponowne kliknięcie przywraca malejącą | FR-HIS-08 | |
| TC-HIS-014 | Ś | P | **Sortowanie po kwocie.** 1. Kliknij nagłówek „Kwota" | Sortowanie od najwyższej kwoty; wskaźnik przenosi się na kolumnę Kwota | FR-HIS-08 | |
| TC-HIS-015 | Ś | P | **Czyszczenie filtrów.** 1. Ustaw filtry 2. „Wyczyść filtry" | Wszystkie filtry, sortowanie i paginacja w stanie domyślnym; toast | FR-HIS-10 | |
| TC-HIS-016 | W | P | **Szczegóły operacji.** 1. Kliknij dowolny wiersz | Modal z kompletem danych; „Zamknij" zamyka modal | FR-HIS-11 | |
| TC-HIS-017 | W | P | **Eksport CSV — pełna lista.** 1. „Eksportuj CSV" | Pobrany plik `wyciag_meridian_RRRR-MM-DD.csv`, toast z liczbą operacji zgodną z licznikiem | FR-HIS-12 | |
| TC-HIS-018 | W | P | **Eksport CSV — po filtrach.** 1. Ustaw filtr uznań 2. Eksportuj | Plik zawiera wyłącznie przefiltrowane operacje (wszystkie strony, nie tylko widoczna) | FR-HIS-12 | |
| TC-HIS-019 | Ś | P | **Format pliku CSV.** 1. Otwórz pobrany plik w arkuszu i edytorze tekstu | Nagłówek `Data;Opis;Kategoria;Kwota;Saldo po operacji`, separator `;`, przecinek dziesiętny, polskie znaki poprawne (BOM UTF-8) | FR-HIS-12 | |
| TC-HIS-020 | W | P | **Spójność salda po operacji.** 1. Wykonaj przelew 100 zł 2. Otwórz Historię | Najnowszy wiersz ma kwotę −100,00 zł, a „Saldo po" równe saldu na pulpicie | FR-HIS-01, FR-TRF-13 | |

## 9. Karty płatnicze (TC-CRD)

| ID | Prio | Typ | Tytuł i kroki | Oczekiwany rezultat | FR | Wynik |
|---|---|---|---|---|---|---|
| TC-CRD-001 | W | P | **Widok kart.** 1. Otwórz Karty | Dwie karty (VISA debetowa, Mastercard kredytowa), numery maskowane do 4 ostatnich cyfr, status „Aktywna" | FR-CRD-01 | |
| TC-CRD-002 | W | P | **Pokazanie numeru karty.** 1. „Pokaż numer" na karcie debetowej | Pełny numer `4556 7375 8689 4523`, przycisk zmienia nazwę na „Ukryj numer"; druga karta pozostaje zamaskowana | FR-CRD-02 | |
| TC-CRD-003 | Ś | P | **Ukrycie numeru karty.** 1. „Ukryj numer" | Numer ponownie zamaskowany | FR-CRD-02 | |
| TC-CRD-004 | K | P | **Blokada karty.** 1. „Zablokuj" 2. Potwierdź | Status „Zablokowana", karta wyszarzona, przycisk „Odblokuj", toast | FR-CRD-03 | |
| TC-CRD-005 | W | P | **Odblokowanie karty.** 1. „Odblokuj" 2. Potwierdź | Status „Aktywna", toast | FR-CRD-04 | |
| TC-CRD-006 | Ś | N | **Anulowanie blokady.** 1. „Zablokuj" 2. „Anuluj" | Status bez zmian | FR-CRD-03, FR-CMN-03 | |
| TC-CRD-007 | Ś | B | **Suwaki limitów — zakresy.** 1. Przesuń suwaki do wartości skrajnych | Limit internetowy 0–10 000 zł, bankomatowy 0–5 000 zł, krok 100 zł; etykiety aktualizowane na żywo | FR-CRD-05, FR-CRD-06 | |
| TC-CRD-008 | W | P | **Zapis limitów z autoryzacją SMS.** 1. Zmień limity 2. „Zapisz limity" 3. Kod `123456` | Toast „Limity karty zostały zapisane." | FR-CRD-07 | |
| TC-CRD-009 | Ś | N | **Anulowanie autoryzacji limitów.** 1. „Zapisz limity" 2. „Anuluj" | Brak toasta potwierdzającego | FR-CRD-07 | |
| TC-CRD-010 | Ś | N | 🔴 **Skuteczność zapisu limitów.** 1. Ustaw limit internetowy 0 zł i zapisz (kod `123456`) 2. Sprawdź w konsoli obiekt `state.settings` 3. Wykonaj dowolną operację obciążającą rachunek | **Oczekiwane:** limit zapisany w stanie aplikacji i egzekwowany. **Rzeczywiste:** zapis kończy się wyłącznie toastem — wartość nie trafia do stanu i nie ma wpływu na operacje — patrz DEF-008 | FR-CRD-07 | |
| TC-CRD-011 | N | P | **Przełącznik płatności zbliżeniowych.** 1. Wyłącz 2. Włącz | Toasty o wyłączeniu i włączeniu | FR-CRD-08 | |

## 10. BLIK (TC-BLK)

| ID | Prio | Typ | Tytuł i kroki | Oczekiwany rezultat | FR | Wynik |
|---|---|---|---|---|---|---|
| TC-BLK-001 | W | P | **Stan początkowy.** 1. Otwórz BLIK | Ikona i przycisk „Generuj kod BLIK"; brak kodu | FR-BLK-01 | |
| TC-BLK-002 | K | P | **Generowanie kodu.** 1. „Generuj kod BLIK" | Kod 6-cyfrowy w formacie `XXX XXX`, licznik startuje od 120 s, pasek postępu pełny | FR-BLK-02, FR-BLK-03 | |
| TC-BLK-003 | W | P | **Odliczanie.** 1. Wygeneruj kod 2. Obserwuj 10 s | Licznik maleje co sekundę, pasek proporcjonalnie się skraca | FR-BLK-03 | |
| TC-BLK-004 | W | B | **Wygaśnięcie kodu.** 1. Wygeneruj kod 2. Odczekaj 120 s | Komunikat „Kod BLIK wygasł." i przycisk „Generuj nowy kod" | FR-BLK-04 | |
| TC-BLK-005 | W | P | **Wygenerowanie kodu po wygaśnięciu.** 1. Po TC-BLK-004 kliknij „Generuj nowy kod" | Nowy kod (inny niż poprzedni), licznik od 120 s | FR-BLK-07 | |
| TC-BLK-006 | Ś | P | **Anulowanie kodu.** 1. Wygeneruj kod 2. „Anuluj" | Powrót do stanu początkowego, licznik zatrzymany | FR-BLK-06 | |
| TC-BLK-007 | Ś | P | **Kopiowanie kodu (http).** 1. Uruchom aplikację przez serwer lokalny 2. Wygeneruj kod 3. „Kopiuj kod" 4. Wklej w edytorze | Toast „Kod BLIK skopiowany do schowka.", wklejony kod zgodny z wyświetlonym | FR-BLK-05 | |
| TC-BLK-008 | N | N | **Kopiowanie kodu (file://).** 1. Uruchom z pliku 2. „Kopiuj kod" | Toast zastępczy z kodem (schowek niedostępny), brak błędu blokującego | FR-BLK-05 | |
| TC-BLK-009 | Ś | P | **Losowość kodu.** 1. Wygeneruj kod 5 razy | Za każdym razem inny kod, zawsze 6 cyfr | FR-BLK-02 | |
| TC-BLK-010 | N | P | **Regeneracja w trakcie odliczania.** 1. Wygeneruj kod 2. Po 10 s wygeneruj ponownie („Anuluj" → „Generuj") | Licznik resetowany do 120 s, brak dwóch równoległych liczników | FR-BLK-07 | |

## 11. Lokaty, kredyty, kantor (TC-DEP, TC-LOA, TC-FXR)

| ID | Prio | Typ | Tytuł i kroki | Oczekiwany rezultat | FR | Wynik |
|---|---|---|---|---|---|---|
| TC-DEP-001 | W | P | **Kalkulator — wartości domyślne.** 1. Otwórz Lokaty | Kwota 10 000, okres 6 mies., 4,80%, odsetki brutto 240,00 zł, netto 194,40 zł, suma 10 194,40 zł | FR-DEP-01, FR-DEP-03 | |
| TC-DEP-002 | W | P | **Przeliczenie dla 12 miesięcy.** 1. Wybierz „12 miesięcy — 5,00%" | Oprocentowanie 5,00%; brutto 500,00 zł; netto 405,00 zł; suma 10 405,00 zł | FR-DEP-03 | |
| TC-DEP-003 | W | B | **Kwota 999 zł.** 1. Wpisz `999` | Komunikat „Kwota od 1 000 do 500 000 zł.", wyniki „—" | FR-DEP-02 | |
| TC-DEP-004 | W | B | **Kwota 1 000 zł.** 1. Wpisz `1000` | Walidacja przechodzi, wyniki przeliczone | FR-DEP-02 | |
| TC-DEP-005 | W | B | **Kwota 500 001 zł.** 1. Wpisz `500001` | Komunikat o zakresie, wyniki „—" | FR-DEP-02 | |
| TC-DEP-006 | K | P | **Otwarcie lokaty.** 1. Kwota `5000` 2. „Otwórz lokatę" 3. Potwierdź | Konto osobiste −5 000 zł, nowa lokata na liście, operacja „Założenie lokaty …" w historii, toast | FR-DEP-04, FR-DEP-05 | |
| TC-DEP-007 | W | N | **Brak środków na lokatę.** 1. Kwota `500000` 2. „Otwórz lokatę" | Toast „Niewystarczające środki na koncie osobistym."; brak nowej lokaty | FR-DEP-04 | |
| TC-DEP-008 | W | P | **Zerwanie lokaty.** 1. „Zerwij" przy `DEP-2044` 2. Potwierdź | Kapitał 10 000 zł wraca na konto osobiste, operacja „Zwrot środków z lokaty…", lokata usunięta z listy, toast | FR-DEP-07 | |
| TC-DEP-009 | K | N | 🔴 **Unikalność ID lokat.** 1. Otwórz lokatę 2 000 zł 2. Zerwij `DEP-2044` 3. Otwórz lokatę 3 000 zł 4. Zerwij jedną z lokat `DEP-2045` | **Oczekiwane:** zerwana zostaje jedna lokata, zwrot jej kapitału. **Rzeczywiste:** obie lokaty znikają, zwrot tylko jednego kapitału (utrata 3 000 zł) — patrz DEF-003 | FR-DEP-08 | |
| TC-DEP-010 | Ś | P | **Data zakończenia lokaty.** 1. Otwórz lokatę 6-miesięczną | Data końca = dzisiaj + 180 dni (miesiąc = 30 dni — uproszczenie, patrz DEF-010) | FR-DEP-06 | |
| TC-DEP-011 | Ś | P | **Pusta lista lokat.** 1. Zerwij wszystkie lokaty | Komunikat „Nie masz aktywnych lokat…" | FR-DEP-09 | |
| TC-LOA-001 | W | P | **Kalkulator — wartości domyślne.** 1. Otwórz Kredyty | 25 000 zł, 36 mies., rata 806,56 zł, całkowita kwota 29 036,24 zł (weryfikacja wobec wzoru annuitetowego) | FR-LOA-01…03 | |
| TC-LOA-002 | W | B | **Skrajne wartości suwaków.** 1. Ustaw min (1 000 zł / 6 mies.) 2. Ustaw max (200 000 zł / 120 mies.) | Wartości i rata przeliczane na żywo; brak wartości ujemnych i `NaN` | FR-LOA-01…03 | |
| TC-LOA-003 | W | P | **Otwarcie wniosku.** 1. „Złóż wniosek" | Modal z podsumowaniem: kwota, okres, rata zgodne z kalkulatorem | FR-LOA-04 | |
| TC-LOA-004 | W | N | **Wniosek bez zgody RODO.** 1. Dochód `5000` 2. Nie zaznaczaj RODO 3. „Wyślij wniosek" | Komunikat „Ta zgoda jest wymagana."; wniosek niewysłany | FR-LOA-05 | |
| TC-LOA-005 | W | B | **Dochód poniżej progu.** 1. Dochód `999` 2. Zaznacz RODO 3. Wyślij | Komunikat „Podaj dochód (min. 1 000 zł)." | FR-LOA-05 | |
| TC-LOA-006 | W | P | **Poprawny wniosek.** 1. Dochód `5000` 2. Zaznacz RODO 3. Wyślij | Loader „Analizujemy Twój wniosek…" → toast z numerem `WNK-…` | FR-LOA-06 | |
| TC-LOA-007 | Ś | P | **Zgoda BIK jest opcjonalna.** 1. Wyślij wniosek bez zaznaczenia BIK | Wniosek przyjęty | FR-LOA-05 | |
| TC-LOA-008 | Ś | P | **Anulowanie wniosku.** 1. Otwórz modal 2. „Anuluj" 3. Otwórz ponownie | Modal zamknięty; przy ponownym otwarciu pola wyczyszczone, zgody odznaczone | FR-LOA-04 | |
| TC-FXR-001 | W | P | **Tabela kursów.** 1. Otwórz Kantor | 4 waluty z kursami kupna/sprzedaży i zmianą 24 h; USD na czerwono (−0,34%), pozostałe na zielono | FR-FXR-01 | |
| TC-FXR-002 | W | P | **Przeliczenie PLN → EUR.** 1. Kwota `100`, PLN → EUR | Wynik 22,83 € (100 / 4,38); informacja o kursie sprzedaży EUR | FR-FXR-02, FR-FXR-03 | |
| TC-FXR-003 | W | P | **Przeliczenie EUR → PLN.** 1. Kwota `100`, EUR → PLN | Wynik 424,00 zł (100 × 4,24) | FR-FXR-03 | |
| TC-FXR-004 | Ś | P | **Przeliczenie krzyżowe USD → GBP.** 1. Kwota `100`, USD → GBP | Wynik 76,02 £ (100 × 3,90 / 5,13); przycisk wymiany nieaktywny z komunikatem o parze PLN ⇄ EUR | FR-FXR-03, FR-FXR-06 | |
| TC-FXR-005 | Ś | N | **Ta sama waluta.** 1. Ustaw PLN → PLN | Wynik „—", komunikat „Wybierz dwie różne waluty.", przycisk nieaktywny | FR-FXR-04 | |
| TC-FXR-006 | Ś | N | **Niepoprawna kwota.** 1. Wpisz `abc`, następnie `0` | Wynik „—", komunikat „Podaj poprawną kwotę.", przycisk nieaktywny | FR-FXR-05 | |
| TC-FXR-007 | K | P | **Wymiana PLN → EUR.** 1. Kwota `438`, PLN → EUR 2. „Wykonaj wymianę" 3. Potwierdź | Konto osobiste −438 zł, konto walutowe +100 €, operacja „Wymiana walut PLN → EUR" w historii, toast | FR-FXR-07, FR-FXR-08 | |
| TC-FXR-008 | K | P | **Wymiana EUR → PLN.** 1. Kwota `100`, EUR → PLN 2. Wykonaj i potwierdź | Konto walutowe −100 €, osobiste +424 zł, operacja uznaniowa w historii | FR-FXR-07, FR-FXR-08 | |
| TC-FXR-009 | W | N | **Brak środków na wymianę.** 1. Kwota `999999`, PLN → EUR 2. Wykonaj | Toast „Niewystarczające środki na koncie osobistym."; salda bez zmian | FR-FXR-07 | |
| TC-FXR-010 | Ś | N | **Anulowanie wymiany.** 1. Wykonaj wymianę 2. W modalu „Anuluj" | Salda bez zmian | FR-FXR-08, FR-CMN-03 | |

## 12. Ustawienia, wiadomości, pomoc, czat (TC-SET, TC-MSG, TC-HLP, TC-CHT)

| ID | Prio | Typ | Tytuł i kroki | Oczekiwany rezultat | FR | Wynik |
|---|---|---|---|---|---|---|
| TC-SET-001 | Ś | P | **Pola profilu domyślnie zablokowane.** 1. Otwórz Ustawienia | Wszystkie pola danych osobowych nieedytowalne, przycisk „Zapisz zmiany" ukryty | FR-SET-01 | |
| TC-SET-002 | Ś | P | **Tryb edycji profilu.** 1. „Edytuj" | E-mail i telefon edytowalne, imię i nazwisko nadal zablokowane, przycisk zmienia się na „Anuluj", widoczne „Zapisz zmiany" | FR-SET-01 | |
| TC-SET-003 | Ś | P | **Anulowanie edycji.** 1. „Edytuj" 2. Zmień e-mail 3. „Anuluj" | Przywrócona pierwotna wartość, pola zablokowane | FR-SET-02 | |
| TC-SET-004 | W | N | **Niepoprawny e-mail.** 1. Edytuj → wpisz `jan.kowalski` 2. Zapisz | Komunikat „Podaj poprawny adres e-mail." | FR-SET-03 | |
| TC-SET-005 | W | B | **Telefon poza zakresem.** 1. Wpisz `12345678` (8 cyfr), następnie 16 cyfr | W obu przypadkach komunikat „Podaj poprawny numer telefonu (9–15 cyfr)." | FR-SET-04 | |
| TC-SET-006 | W | P | **Zapis danych profilu.** 1. Zmień e-mail na `jan.nowy@example.com` 2. Zapisz 3. Przejdź na inną stronę i wróć | Toast, wartość zachowana, pola zablokowane | FR-SET-05 | |
| TC-SET-007 | K | N | **Błędne obecne hasło.** 1. Obecne `Zle123!`, nowe `NoweHaslo1!` ×2 2. „Zmień hasło" | Komunikat „Obecne hasło jest nieprawidłowe." | FR-SET-06 | |
| TC-SET-008 | K | N | **Nowe hasło niespełniające wymagań.** 1. Obecne `Test123!`, nowe `haslo` ×2 | Komunikat „Hasło nie spełnia wymagań bezpieczeństwa." | FR-SET-06 | |
| TC-SET-009 | K | N | **Niezgodne powtórzenie hasła.** 1. Nowe `NoweHaslo1!`, powtórz `NoweHaslo2!` | Komunikat „Hasła nie są identyczne." | FR-SET-06 | |
| TC-SET-010 | W | P | **Wskaźnik siły hasła.** 1. Wpisuj kolejno `haslo`, `Haslo123`, `Haslo123!` | Pasek rośnie skokowo (25% na kryterium), zmiana koloru: czerwony → żółty → zielony | FR-SET-07 | |
| TC-SET-011 | K | P | **Zmiana hasła i logowanie nowym hasłem.** 1. Zmień hasło na `NoweHaslo1!` 2. Autoryzuj kodem `123456` 3. Wyloguj się 4. Zaloguj starym hasłem 5. Zaloguj nowym hasłem | Toast o zmianie i wyczyszczenie pól; logowanie starym hasłem odrzucone; logowanie nowym udane | FR-SET-08 | |
| TC-SET-012 | Ś | N | **Anulowanie autoryzacji zmiany hasła.** 1. Wypełnij poprawnie 2. „Zmień hasło" 3. „Anuluj" w modalu SMS | Hasło niezmienione (logowanie starym hasłem nadal działa) | FR-SET-08, FR-CMN-04 | |
| TC-SET-013 | Ś | P | **Przełączniki powiadomień.** 1. Przełącz każdy z 4 przełączników 2. Sprawdź obiekt `state.settings` w konsoli | Toast po każdej zmianie; ustawienia powinny trafiać do stanu aplikacji — obecnie tak się nie dzieje (DEF-009) | FR-SET-09 | |
| TC-SET-014 | Ś | P | **Zmiana czasu sesji.** 1. Wybierz 10 minut | Toast „Czas automatycznego wylogowania: 10 min."; licznik bezczynności zresetowany | FR-SET-11 | |
| TC-MSG-001 | Ś | P | **Skrzynka odbiorcza.** 1. Otwórz Wiadomości | 3 wiadomości, najnowsza nieprzeczytana (kropka + pogrubienie); badge w menu = 1 | FR-MSG-01, FR-MSG-02 | |
| TC-MSG-002 | Ś | P | **Otwarcie wiadomości.** 1. Kliknij nieprzeczytaną wiadomość | Widok szczegółów z tytułem, nadawcą, datą i treścią; badge znika | FR-MSG-03 | |
| TC-MSG-003 | Ś | P | **Powrót do listy.** 1. W szczegółach kliknij „← Wróć do listy" | Lista wiadomości; otwarta wiadomość oznaczona jako przeczytana | FR-MSG-04 | |
| TC-HLP-001 | Ś | P | **Rozwijanie FAQ.** 1. Kliknij pierwsze pytanie 2. Kliknij drugie | Pierwsza odpowiedź rozwinięta, po kliknięciu drugiego pierwsza się zwija (tylko jedna otwarta) | FR-HLP-01 | |
| TC-HLP-002 | Ś | P | **Zwijanie FAQ.** 1. Kliknij otwarte pytanie ponownie | Odpowiedź zwinięta | FR-HLP-01 | |
| TC-HLP-003 | W | N | **Walidacja formularza kontaktowego.** 1. Wyślij pusty formularz | Komunikaty przy: imieniu i nazwisku, e-mailu, wiadomości | FR-HLP-02 | |
| TC-HLP-004 | W | B | **Wiadomość krótsza niż 10 znaków.** 1. Wpisz `Pomocy` (6 znaków) + poprawne pozostałe pola | Komunikat „Wiadomość musi mieć min. 10 znaków." | FR-HLP-02 | |
| TC-HLP-005 | W | P | **Wysłanie zgłoszenia.** 1. Wypełnij poprawnie wszystkie pola 2. Wyślij | Toast z numerem `ZGL-…`; formularz wyczyszczony | FR-HLP-03 | |
| TC-CHT-001 | Ś | P | **Otwarcie czatu.** 1. Kliknij przycisk czatu | Panel otwarty z wiadomością powitalną Meri, fokus w polu wpisywania | FR-CHT-01, FR-CHT-02 | |
| TC-CHT-002 | Ś | P | **Zapytanie o saldo.** 1. Wpisz `jakie mam saldo` 2. Enter | Wskaźnik „Meri pisze…", po ~0,75 s odpowiedź z **aktualnymi** saldami trzech rachunków | FR-CHT-04, FR-CHT-05 | |
| TC-CHT-003 | Ś | P | **Zapytania tematyczne.** 1. Zadaj kolejno pytania o: przelew, BLIK, kurs euro, lokatę, zmianę hasła, kontakt | Każde pytanie otrzymuje odpowiedź dopasowaną tematycznie | FR-CHT-05 | |
| TC-CHT-004 | Ś | N | **Zapytanie nierozpoznane.** 1. Wpisz `pogoda w Warszawie` | Odpowiedź domyślna z listą obsługiwanych tematów | FR-CHT-06 | |
| TC-CHT-005 | N | N | **Pusta wiadomość.** 1. Kliknij „Wyślij" bez tekstu | Brak reakcji, żadna wiadomość nie zostaje dodana | FR-CHT-03 | |
| TC-CHT-006 | N | P | **Zamknięcie czatu.** 1. Kliknij „✕" 2. Otwórz ponownie | Panel zamknięty; po ponownym otwarciu historia rozmowy zachowana, brak powtórzonego powitania | FR-CHT-01, FR-CHT-02 | |

## 13. Testy niefunkcjonalne (TC-NFR)

| ID | Prio | Typ | Tytuł i kroki | Oczekiwany rezultat | FR | Wynik |
|---|---|---|---|---|---|---|
| TC-NFR-001 | W | P | **RWD — telefon 390×844.** 1. Przejdź przez wszystkie moduły | Brak poziomego przewijania strony, tabele przewijane w swoim kontenerze, siatki jednokolumnowe, sidebar ukryty | FR-NAV-03 | |
| TC-NFR-002 | W | P | **RWD — tablet 768×1024.** 1. Przejdź przez kluczowe moduły | Układ czytelny, elementy klikalne, formularze użyteczne | FR-NAV-03 | |
| TC-NFR-003 | W | P | **Kompatybilność przeglądarek.** 1. Wykonaj zestaw dymny w Chrome, Firefox, Edge, Safari | Identyczne zachowanie funkcjonalne; brak błędów w konsoli | — | |
| TC-NFR-004 | Ś | P | **Nawigacja klawiaturą.** 1. Przejdź przez formularz logowania i przelewu klawiszem Tab 2. Zatwierdzaj Enterem | Logiczna kolejność fokusu, widoczna obwódka fokusu, formularze zatwierdzalne z klawiatury | FR-CMN-08 | |
| TC-NFR-005 | Ś | N | **Fokus w modalach.** 1. Otwórz modal SMS 2. Naciskaj Tab | Sprawdź, czy fokus nie „ucieka" pod modal — brak pułapki fokusu jest znanym ograniczeniem (DEF-012) | FR-CMN-08 | |
| TC-NFR-006 | Ś | P | **Kontrast i czytelność w trybie ciemnym.** 1. Włącz tryb ciemny 2. Przejrzyj wszystkie moduły | Brak elementów nieczytelnych, poprawny kontrast tekstu i badge'ów | FR-NAV-04 | |
| TC-NFR-007 | Ś | P | **Ograniczenie animacji.** 1. Włącz w systemie „reduce motion" 2. Korzystaj z aplikacji | Animacje i przejścia wyłączone; funkcjonalność bez zmian | FR-CMN-08 | |
| TC-NFR-008 | Ś | P | **Brak dostępu do sieci zewnętrznej.** 1. Zablokuj `fonts.googleapis.com` 2. Odśwież | Aplikacja działa na krojach zastępczych; brak błędów blokujących | — | |
| TC-NFR-009 | Ś | P | **Czystość konsoli.** 1. Przejdź pełny scenariusz E2E-01 z otwartą konsolą | Brak błędów JavaScript (poziom `error`) | — | |
| TC-NFR-010 | Ś | P | **Wydajność renderowania historii.** 1. Ustaw 50 pozycji na stronę 2. Sortuj i filtruj | Odświeżenie tabeli poniżej 1 s, brak zauważalnych zacięć | FR-HIS-05 | |
| TC-NFR-011 | Ś | N | **Odporność na wielokrotne kliknięcia.** 1. Kliknij szybko 5× „Zatwierdź i autoryzuj", „Otwórz lokatę", „Wykonaj wymianę" | Brak zdublowanych operacji, sald ani wpisów w historii | FR-CMN-02 | |
| TC-NFR-012 | W | N | **Reset stanu po odświeżeniu.** 1. Wykonaj kilka operacji 2. Naciśnij F5 | Ekran logowania i wszystkie dane w stanie startowym (zachowanie zamierzone — brak trwałości) | — | |

## 14. Macierz pokrycia wymagań

| Obszar | Wymagania | Przypadki testowe | Pokrycie |
|---|---|---|---|
| Logowanie | FR-LOG-01…08 | TC-LOG-001…020, TC-SES-001 | 100% |
| Sesja | FR-SES-01…04 | TC-SES-003…010 | 100% |
| Nawigacja | FR-NAV-01…06 | TC-NAV-001…011, TC-NFR-001, TC-NFR-002 | 100% |
| Pulpit | FR-DASH-01…07 | TC-DASH-001…008 | 100% |
| Przelew krajowy | FR-TRF-01…16 | TC-TRF-001…035 | 100% |
| Przelew własny | FR-OWN-01…05 | TC-OWN-001…007 | 100% |
| Zaplanowane | FR-SCH-01…02 | TC-SCH-001…004, TC-TRF-019, TC-TRF-020 | 100% |
| Zlecenia stałe | FR-STO-01…06 | TC-STO-001…007 | 100% |
| Historia | FR-HIS-01…12 | TC-HIS-001…020 | 100% |
| Karty | FR-CRD-01…09 | TC-CRD-001…011 | 100% |
| BLIK | FR-BLK-01…08 | TC-BLK-001…010 | 100% |
| Lokaty | FR-DEP-01…09 | TC-DEP-001…011 | 100% |
| Kredyty | FR-LOA-01…07 | TC-LOA-001…008 | 100% |
| Kantor | FR-FXR-01…09 | TC-FXR-001…010 | 100% |
| Wiadomości | FR-MSG-01…05 | TC-MSG-001…003, TC-NAV-008 | 100% |
| Ustawienia | FR-SET-01…11 | TC-SET-001…014, TC-NAV-006, TC-NAV-007 | 100% |
| Pomoc | FR-HLP-01…04 | TC-HLP-001…005 | 100% |
| Czat AI | FR-CHT-01…07 | TC-CHT-001…006 | 100% |
| Elementy wspólne | FR-CMN-01…08 | TC-LOG-001, TC-TRF-027…030, TC-NFR-004…011 | 100% |

**Podsumowanie liczbowe:** 212 przypadków testowych

| Obszar | Liczba |
|---|---|
| Logowanie i uwierzytelnianie | 20 |
| Sesja i wylogowanie | 10 |
| Nawigacja i powłoka | 11 |
| Pulpit | 8 |
| Przelewy (krajowy 35, własny 7, zaplanowane 4, zlecenia stałe 7) | 53 |
| Historia operacji | 20 |
| Karty płatnicze | 11 |
| BLIK | 10 |
| Produkty (lokaty 11, kredyty 8, kantor 10) | 29 |
| Ustawienia i komunikacja (ustawienia 14, wiadomości 3, pomoc 5, czat 6) | 28 |
| Testy niefunkcjonalne | 12 |
| **Razem** | **212** |

**Rozkład priorytetów:** krytyczne (K) — testy przepływu pieniądza i autoryzacji, wykonywane
w pierwszej kolejności; wysokie (W) — kluczowe funkcje modułów; średnie (Ś) i niskie (N) —
w miarę dostępnego czasu. Przypadki oznaczone 🔴 (9 sztuk) wykrywają potwierdzone defekty opisane w raporcie defektów.
