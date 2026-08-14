# Dane testowe i selektory — Meridian Bank 2.6.0

Dokument pomocniczy do wykonywania testów manualnych i pisania testów automatycznych.

---

## 1. Konto i kody

| Element | Wartość | Uwagi |
|---|---|---|
| Login | `jan.kowalski` | Rozróżnia wielkość liter; spacje na brzegach są przycinane |
| Hasło | `Test123!` | Zmienialne w Ustawieniach; obowiązuje do odświeżenia strony |
| Kod SMS | `123456` | Ten sam dla logowania i wszystkich autoryzacji operacji |
| Blokada logowania | 3 błędne próby → 30 s | Automatyczne odblokowanie |
| Limit prób SMS (operacje) | 3 | Po przekroczeniu operacja anulowana |
| Limit prób SMS (logowanie) | brak | Znany defekt DEF-004 |

## 2. Rachunki i salda startowe

| ID | Nazwa | Waluta | Saldo | Numer rachunku (bez spacji) |
|---|---|---|---|---|
| `ror` | Konto osobiste | PLN | 12 543,21 | 61109010140000071219812874 |
| `sav` | Konto oszczędnościowe | PLN | 45 200,00 | 27114020040000300201355387 |
| `eur` | Konto walutowe | EUR | 3 250,50 | 83124069570111000011314260 |

## 3. Dane wejściowe do testów walidacji

### 3.1 Numer rachunku odbiorcy (NRB)

| Wartość | Oczekiwany rezultat |
|---|---|
| `61109010140000071219812874` | Przyjęty (26 cyfr) |
| `PL61 1090 1014 0000 0712 1981 2874` | Przyjęty (usuwane spacje i prefiks `PL`) |
| `6110901014000007121981287` (25 cyfr) | Odrzucony |
| `611090101400000712198128745` (27 cyfr) | Odrzucony |
| `6110901014000007121981287X` | Odrzucony (litera) |
| pusty | Odrzucony |
| `00000000000000000000000000` | **Przyjęty** — brak weryfikacji sumy kontrolnej (DEF-005) |

### 3.2 Kwoty (funkcja `parseAmount`)

| Wartość | Wynik | Uwagi |
|---|---|---|
| `150` / `150,50` / `150.50` | Przyjęta | Przecinek i kropka równoważne |
| `1 000,50` | Przyjęta (1 000,50) | Spacje usuwane |
| `1.000,50` | Odrzucona | Kropka jako separator tysięcy nieobsługiwana (DEF-011) |
| `0` | Odrzucona | Musi być > 0 |
| `0,01` | Przyjęta | Wartość brzegowa |
| `-50` | Odrzucona | |
| `10,999` | Odrzucona | Maks. 2 miejsca po przecinku |
| `abc`, pusta | Odrzucona | |
| = saldo rachunku | Przyjęta | Wartość brzegowa |
| saldo + 0,01 | Odrzucona | „Niewystarczające środki na rachunku." |

### 3.3 Hasła (zmiana hasła)

| Wartość | Wynik | Powód |
|---|---|---|
| `NoweHaslo1!` | Poprawne | ≥8 znaków, wielka litera, cyfra, znak specjalny |
| `haslo123!` | Odrzucone | Brak wielkiej litery |
| `Haslo!` | Odrzucone | Za krótkie (<8) |
| `Haslo1234` | Odrzucone | Brak znaku specjalnego |
| `HasloABC!` | Odrzucone | Brak cyfry |

Wskaźnik siły: 25% za każde spełnione kryterium (długość ≥ 8, wielka litera, cyfra, znak specjalny).

### 3.4 E-mail i telefon (profil, formularz kontaktowy)

| Wartość | E-mail | Telefon |
|---|---|---|
| `jan.kowalski@example.com` | Poprawny | — |
| `jan.kowalski` | Odrzucony | — |
| `jan@example` | Odrzucony (brak domeny najwyższego poziomu) | — |
| `jan @example.com` | Odrzucony (spacja) | — |
| `+48 601 234 789` | — | Poprawny (12 cyfr) |
| `12345678` | — | Odrzucony (8 cyfr) |
| `1234567890123456` | — | Odrzucony (16 cyfr) |

### 3.5 Pozostałe pola

| Pole | Reguła |
|---|---|
| Nazwa odbiorcy | min. 3 znaki (po przycięciu spacji) |
| Tytuł przelewu | wymagany, maks. 70 znaków |
| Data przelewu | ≥ dzisiaj (atrybut `min`) |
| Kwota lokaty | 1 000 – 500 000 zł |
| Dochód (wniosek kredytowy) | ≥ 1 000 zł |
| Wiadomość (kontakt) | min. 10 znaków |
| Dzień miesiąca (zlecenie stałe) | 1–28 |

## 4. Wartości referencyjne do weryfikacji obliczeń

### 4.1 Lokaty — `odsetki brutto = kwota × stopa/100 × miesiące/12`, `netto = brutto × 0,81`

| Kwota | Okres | Stopa | Brutto | Netto | Kapitał + odsetki |
|---|---|---|---|---|---|
| 10 000 | 3 mies. | 4,50% | 112,50 | 91,13 | 10 091,13 |
| 10 000 | 6 mies. | 4,80% | 240,00 | 194,40 | 10 194,40 |
| 10 000 | 12 mies. | 5,00% | 500,00 | 405,00 | 10 405,00 |
| 10 000 | 24 mies. | 4,60% | 920,00 | 745,20 | 10 745,20 |
| 1 000 | 6 mies. | 4,80% | 24,00 | 19,44 | 1 019,44 |
| 500 000 | 12 mies. | 5,00% | 25 000,00 | 20 250,00 | 520 250,00 |

### 4.2 Kredyt — rata annuitetowa, `r = 0,0999/12`

| Kwota | Okres | Rata (≈) | Suma spłat (≈) |
|---|---|---|---|
| 25 000 | 36 mies. | 806,56 | 29 036,24 |
| 1 000 | 6 mies. | 171,56 | 1 029,34 |
| 100 000 | 60 mies. | 2 124,21 | 127 452,75 |
| 200 000 | 120 mies. | 2 641,91 | 317 028,88 |

*Wartości wyliczone wzorem annuitetowym; dopuszczalna różnica prezentacji ±0,01 zł na racie.*

### 4.3 Kantor — kupno waluty źródłowej → PLN → sprzedaż waluty docelowej

| Kwota | Para | Wynik | Obliczenie |
|---|---|---|---|
| 100 | PLN → EUR | 22,83 € | 100 / 4,38 |
| 100 | EUR → PLN | 424,00 zł | 100 × 4,24 |
| 438 | PLN → EUR | 100,00 € | 438 / 4,38 |
| 100 | USD → GBP | 76,02 £ | 100 × 3,90 / 5,13 |
| 100 | PLN → CHF | 21,79 CHF | 100 / 4,59 |
| 100 | CHF → PLN | 445,00 zł | 100 × 4,45 |

Wymiana wykonalna wyłącznie dla pary **PLN ⇄ EUR**; pozostałe pary działają jako przelicznik.

## 5. Katalog selektorów `data-testid`

Wszystkie kluczowe elementy mają atrybut `data-testid` — jest to **rekomendowany i jedyny
stabilny sposób** adresowania elementów w testach automatycznych.

### 5.1 Logowanie i sesja
`login-view`, `login-username`, `login-password`, `login-password-toggle`, `login-remember`,
`login-submit`, `login-username-error`, `login-password-error`, `login-error`, `login-lockout`,
`login-info-banner`, `login-forgot-link`, `login-test-credentials`, `login-sms-input`,
`login-sms-submit`, `login-sms-back`, `login-sms-error`, `modal-session`, `session-countdown`,
`session-extend`, `session-logout-now`, `session-timeout`

### 5.2 Powłoka i nawigacja
`sidebar`, `main-nav`, `burger-menu`, `page-title`, `theme-toggle`, `env-ribbon`,
`global-loader`, `toast-container`, `toast`, `notifications-button`, `notifications-dot`,
`notifications-dropdown`, `notifications-mark-read`, `user-menu-button`, `user-dropdown`,
`user-menu-settings`, `user-menu-messages`, `user-menu-logout`,
`nav-dashboard`, `nav-transfers`, `nav-history`, `nav-cards`, `nav-blik`, `nav-deposits`,
`nav-loans`, `nav-fx`, `nav-messages`, `nav-settings`, `nav-help`, `messages-badge`

### 5.3 Strony
`page-dashboard`, `page-transfers`, `page-history`, `page-cards`, `page-blik`, `page-deposits`,
`page-loans`, `page-fx`, `page-messages`, `page-settings`, `page-help`
*(aktywna strona ma dodatkowo klasę `.active`)*

### 5.4 Pulpit
`accounts-grid`, `account-tile-ror` / `-sav` / `-eur`, `balance-ror` / `-sav` / `-eur`,
`ai-insight`, `quick-transfer`, `quick-blik`, `quick-fx`, `quick-deposit`,
`recent-transactions`, `recent-see-all`, `rates-widget`, `spending-chart`

### 5.5 Przelewy
`transfer-tabs`, `tab-domestic`, `tab-own`, `tab-scheduled`, `tab-standing`,
`transfer-from`, `transfer-saved-recipient`, `transfer-recipient`, `transfer-account`,
`transfer-amount`, `transfer-date`, `transfer-title`, `transfer-save-recipient`,
`transfer-submit`, `transfer-clear`, `transfer-recipient-error`, `transfer-account-error`,
`transfer-amount-error`, `transfer-date-error`, `transfer-title-error`,
`transfer-summary`, `summary-recipient`, `summary-amount`, `transfer-confirm`, `transfer-edit`,
`transfer-success`, `transfer-reference`, `transfer-download`, `transfer-new`,
`own-from`, `own-to`, `own-amount`, `own-amount-error`, `own-submit`,
`scheduled-list`, `scheduled-cancel-<ID>`,
`standing-list`, `standing-new`, `standing-recipient`, `standing-title`, `standing-amount`,
`standing-day`, `standing-save`, `standing-cancel`

### 5.6 Historia
`history-search`, `history-type`, `history-date-from`, `history-date-to`, `history-per-page`,
`history-reset`, `history-count`, `history-export`, `history-sort-date`, `history-sort-amount`,
`history-table-body`, `h-row-<ID transakcji>`, `history-page-info`, `history-pagination`,
`modal-transaction`, `tx-detail-desc`, `tx-detail-amount`, `tx-detail-close`

### 5.7 Karty i BLIK
`cards-grid`, `card-deb`, `card-cre`, `card-number-deb`, `card-number-cre`,
`card-status-deb`, `card-status-cre`, `card-shownum-deb`, `card-shownum-cre`,
`card-block-deb`, `card-block-cre`, `card-limit-online`, `card-limit-atm`,
`card-contactless`, `card-save-limits`,
`blik-generate`, `blik-code`, `blik-countdown`, `blik-copy`, `blik-cancel`, `blik-regenerate`

### 5.8 Produkty
`deposit-amount`, `deposit-amount-error`, `deposit-period`, `deposit-profit`, `deposit-net`,
`deposit-open`, `deposits-list`, `deposit-row-<ID>`, `deposit-close-<ID>`,
`loan-amount`, `loan-months`, `loan-installment`, `loan-apply`, `modal-loan`, `loan-income`,
`loan-income-error`, `loan-employment`, `loan-consent-rodo`, `loan-consent-bik`,
`loan-consent-error`, `loan-submit`, `loan-cancel`,
`fx-amount`, `fx-from`, `fx-to`, `fx-result`, `fx-execute`, `fx-rates`

### 5.9 Ustawienia, komunikacja, modale wspólne
`profile-edit`, `profile-email`, `profile-phone`, `profile-email-error`, `profile-phone-error`,
`profile-save`, `password-current`, `password-new`, `password-repeat`, `password-strength-bar`,
`password-current-error`, `password-new-error`, `password-repeat-error`, `password-save`,
`notify-email`, `notify-sms`, `notify-push`, `notify-marketing`, `settings-dark-mode`,
`messages-list`, `message-M-1` / `M-2` / `M-3`, `message-detail`, `message-back`,
`faq`, `faq-1`…`faq-4`, `contact-name`, `contact-email`, `contact-topic`, `contact-message`,
`contact-submit`,
`chat-fab`, `chat-panel`, `chat-body`, `chat-input`, `chat-send`, `chat-close`,
`modal-confirm`, `modal-confirm-ok`, `modal-confirm-cancel`,
`modal-sms`, `sms-input`, `sms-confirm`, `sms-cancel`, `sms-error`, `modal-standing`

### 5.10 Elementy bez `data-testid` (adresowanie po atrybutach danych)

| Element | Selektor |
|---|---|
| Wiersz zaplanowanego przelewu — przycisk anulowania | `[data-cancel="<ID>"]` |
| Zlecenie stałe — wstrzymaj/wznów | `[data-st-toggle="<ID>"]` |
| Zlecenie stałe — usuń | `[data-st-del="<ID>"]` |
| Lokata — zerwij | `[data-dep-close="<ID>"]` |
| Wiersz operacji (pulpit/historia) | `[data-tx="<ID>"]` |
| Przycisk paginacji | `.pag-btn[data-pg="1\|prev\|next"]` |
| Powiadomienie na liście | `[data-notif="N-1"]` |

## 6. Stany interfejsu przydatne w automatyzacji

| Stan | Sposób weryfikacji |
|---|---|
| Trwa operacja (loader) | `#loader.on` widoczny → czekaj na jego zniknięcie |
| Modal otwarty | element modala ma klasę `.open` |
| Aktywna strona | `[data-testid=page-…].active` |
| Aktywna zakładka przelewów | `.ttab.active` |
| Pole z błędem | kontener ma klasę `.invalid`, komunikat `.error-msg.show` |
| Karta zablokowana | `.bankcard.blocked`, badge `card-status-<id>` = „Zablokowana" |
| Nieprzeczytana wiadomość/powiadomienie | `.dd-item.unread` |
| Kod BLIK aktywny | `#blik-active` bez klasy `hidden` |

## 7. Czasy oczekiwania w aplikacji

| Zdarzenie | Czas |
|---|---|
| Logowanie (krok 1) | 900 ms |
| Logowanie (krok 2 → pulpit) | 900 ms |
| Przejście między stronami | 450 ms |
| Realizacja przelewu | 1 100 ms |
| Analiza wniosku kredytowego | 1 500 ms |
| Skeleton kursów walut (pierwsze wejście) | 1 400 ms |
| Toast z kodem SMS | 300–350 ms po otwarciu ekranu/modala |
| Odpowiedź czatu Meri | 750 ms |
| Ważność kodu BLIK | 120 s |
| Blokada logowania | 30 s |
| Ostrzeżenie o sesji | 60 s przed wygaśnięciem |
| Czas życia toasta | ~4,2 s (usunięcie po 4,6 s) |

## 8. Szablon środowiska testowego (do zgłoszeń defektów)

```
Aplikacja:     Meridian Bank 2.6.0 (index.html)
Uruchomienie:  http://localhost:8000 (python3 -m http.server) | file://
Przeglądarka:  Chrome 1xx / Firefox 1xx / Safari 1x / Edge 1xx
System:        Windows 11 / macOS 15 / Android 15
Rozdzielczość: 1920×1080 (lub 390×844 — mobile)
Stan startowy: po odświeżeniu strony (F5)
```
