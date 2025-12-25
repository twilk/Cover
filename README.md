PRD - Generator coverów eventów z Google Sheets + Google Slides + Apps Script

1) Cel produktu

Zautomatyzować tworzenie powtarzalnych coverów wydarzeń (np. milongi) na bazie jednego spójnego szablonu graficznego w Google Slides, sterowanego danymi i assetami z Google Sheets oraz Google Drive.

System ma:

generować grafiki w kilku formatach (np. IG post 1080x1350, story 1080x1920, FB 1200x628, A2 PDF)

podmieniać teksty i obrazy w szablonie (DJ, data, adres, logo serii, tło, zdjęcia z milongi, overlaye)

zapisywać output w Drive i wpisywać linki do Sheets

mieć prosty workflow statusów (TODO -> RENDER -> DONE / ERROR)

mieć walidację danych i czytelne logi błędów


2) Zakres

In scope

Konfiguracja arkusza Google Sheets jako "data source"

Konfiguracja folderów na Google Drive: ASSETS, OUTPUT, LOGS (opcjonalnie)

Szablon Google Slides:

placeholdery tekstowe

placeholdery obrazów (kształty/ramki)

warianty formatów jako osobne slajdy lub osobne prezentacje


Apps Script:

menu w arkuszu (Generate selected / Generate all / Validate / Settings)

render 1 wiersz = 1 zestaw plików wyjściowych

eksport do PNG/JPG i PDF

nazewnictwo plików + linki zwrotne do Sheets

logowanie do osobnego arkusza LOGS i/lub Logger

retry i obsługa błędów



Out of scope (na start)

Automatyczne kadrowanie twarzy DJa "jak człowiek" (da się częściowo, ale to osobny moduł)

Zaawansowana obróbka zdjęć (retusz, AI upscaling, itp.)

Integracja z social scheduling (Meta Business Suite, itp.)

Wersjonowanie szablonu jak w CI/CD (można dodać później)


3) Użytkownicy i role

Operator (marketing/organizator)

wypełnia wiersze w Sheets

uruchamia generowanie lub masowe generowanie


Administrator (techniczny)

utrzymuje szablon Slides i mapowanie placeholderów

ustawia foldery i uprawnienia

debug i rozwój



4) Główne scenariusze (User Stories)

1. Jako operator chcę dodać wiersz z danymi eventu i wygenerować cover w 2-4 formatach jednym kliknięciem.


2. Jako operator chcę widzieć status i linki do wygenerowanych plików w tym samym wierszu.


3. Jako operator chcę dostać zrozumiały błąd (co jest nie tak i w której kolumnie).


4. Jako admin chcę mieć jeden szablon, który utrzymuje spójność designu dla serii (NEW WAVE, POPOŁUDNIOWA, itd.).


5. Jako admin chcę łatwo dodać nowy format (np. banner na stronę) bez przebudowy całego systemu.



5) Architektura rozwiązania

Google Sheets:

arkusz EVENTS - dane wejściowe + statusy + wyniki

arkusz SETTINGS - konfiguracja (ID szablonu, foldery, formaty, mapowanie)

arkusz LOGS - historia renderów i błędów


Google Drive:

folder ASSETS - stałe elementy: loga, tła, overlaye, tekstury

folder OUTPUT - wygenerowane grafiki

opcjonalnie folder OUTPUT per seria lub per miesiąc


Google Slides:

plik Template Slides (master)

slajdy jako formaty lub warianty:

np. Slide 1: IG post 1080x1350

Slide 2: IG story 1080x1920

Slide 3: FB cover 1200x628

Slide 4: A2 (PDF)



Apps Script:

skrypt spięty z arkuszem (container-bound)

korzysta z SlidesApp, DriveApp, SpreadsheetApp, UrlFetchApp (opcjonalnie)



6) Dane wejściowe - specyfikacja Google Sheets

6.1 Arkusz EVENTS - kolumny wymagane

Minimalny zestaw:

row_id (unikalny identyfikator, np. auto: EV-000123)

status (TODO / RENDER / DONE / ERROR)

event_series (np. NEW WAVE, POPOŁUDNIOWA)

event_title (np. "New Wave Milonga")

date (YYYY-MM-DD)

time_from (HH:MM)

time_to (HH:MM)

venue_name

address_line

city

dj_name


Assety (wskazania do Drive):

dj_photo_file_id

background_file_id

logo_file_id

overlay_file_id (opcjonalnie)

milonga_photo_1_file_id (opcjonalnie)

milonga_photo_2_file_id (opcjonalnie)


Wyniki (wypełniane automatycznie):

output_links (np. sklejone linki albo osobne kolumny per format)

output_folder_link

last_render_at

error_message


6.2 Arkusz EVENTS - kolumny opcjonalne (bardzo przydatne)

language (PL/EN) - jeśli kiedyś będziesz robić wersje językowe

theme (np. SILVER, NEON, CLASSIC) - do wyboru tła/overlayu

format_set (np. BASIC, FULL) - generuj tylko wybrane formaty

publish_date (planowanie)

notes (dla operatora)


6.3 Arkusz SETTINGS - konfiguracja

template_presentation_id (ID pliku Slides)

assets_folder_id

output_folder_id

logs_enabled (TRUE/FALSE)

default_formats (np. IG_POST, IG_STORY)

filename_pattern

przykład: "{date}{series}{dj}_{format}"


timezone (np. Europe/Warsaw)

max_batch (np. 50 na raz, żeby nie udusić limitów)


6.4 Arkusz SETTINGS - mapowanie placeholderów

Mapowanie musi być jawne, żeby nie było "a skrypt sobie zgadnie".

Przykładowa tabela mapowania:

placeholder_key - typ - slides_object_name - source_column - required

EVENT_TITLE - text - ph_event_title - event_title - TRUE

DJ_NAME - text - ph_dj_name - dj_name - TRUE

DATE - text - ph_date - date - TRUE

TIME - text - ph_time - time_from/time_to - TRUE

ADDRESS - text - ph_address - address_line - TRUE

LOGO - image - ph_logo - logo_file_id - TRUE

DJ_PHOTO - image - ph_dj_photo - dj_photo_file_id - TRUE

BACKGROUND - image - ph_bg - background_file_id - TRUE

OVERLAY - image - ph_overlay - overlay_file_id - FALSE


Wymóg: każdy placeholder w Slides ma ustawioną nazwę obiektu (Alt text/Description) zgodnie z "slides_object_name", żeby Apps Script mógł go znaleźć niezawodnie.

7) Specyfikacja szablonu Google Slides

7.1 Zasady szablonu

Każdy format to osobny slajd (najprościej)

Warstwy:

tło (BG) jako obraz lub kształt z wypełnieniem

overlaye/efekty (opcjonalnie)

zdjęcie DJa w ramce lub masce

teksty na wierzchu

logo serii


Placeholdery tekstowe muszą mieć unikalne "Description" (np. ph_dj_name)

Placeholdery obrazów najlepiej robić jako kształt-prostokąt, który skrypt podmieni obrazem (wtedy łatwo zachować rozmiar i pozycję)


7.2 Formatowanie daty i czasu

date: wejście YYYY-MM-DD

wyświetlanie np. "12 stycznia 2026" albo "12.01.2026"

time: "18:00-23:00" Konwersja formatów w Apps Script ma być deterministyczna.


7.3 Warianty serii (NEW WAVE vs POPOŁUDNIOWA)

Opcje:

Jeden szablon, a logo/tło/overlay podmieniane assetami

Albo osobne slajdy wariantowe per seria W tym PRD: rekomendacja - jeden template, assety sterowane kolumnami.


8) Funkcjonalności Apps Script

8.1 Menu w Google Sheets

Po otwarciu arkusza:

Covers - Validate row(s)

Covers - Generate selected rows

Covers - Generate all TODO

Covers - Retry ERROR

Covers - Settings (otwórz arkusz SETTINGS)

Covers - Help (krótka instrukcja w sidebarze)


8.2 Walidacja danych

Walidacja przed renderem:

status musi być TODO lub RENDER (dla batch)

wymagane pola tekstowe niepuste

file_id dla wymaganych obrazów niepuste

sprawdzenie czy plik w Drive istnieje i jest dostępny

sprawdzenie czy slajdy mają wszystkie wymagane placeholdery W przypadku błędu:

status -> ERROR

error_message -> precyzyjny komunikat: "Missing dj_photo_file_id" albo "Cannot access Drive file: <id>"


8.3 Generowanie

Dla każdego wiersza:

1. Ustaw status na RENDER i zapisz timestamp startu


2. Skopiuj template prezentacji do folderu roboczego (opcjonalnie) albo pracuj na kopii w pamięci


3. Dla każdego slajdu/formatu:

wstaw tło, overlay, DJ photo, logo

podmień teksty



4. Eksportuj slajd do PNG lub JPG


5. (Opcjonalnie) eksport całej prezentacji do PDF dla A2


6. Zapisz pliki w OUTPUT:

folder główny: OUTPUT

podfolder per event: "{date}{series}{dj}" (czytelność)



7. Wpisz linki do Sheets:

output_folder_link

output_links per format



8. Status -> DONE, last_render_at -> now



8.4 Eksport formatów - spec

PNG dla social (najlepsza jakość)

JPG opcjonalnie (mniejszy rozmiar)

PDF dla druku (A2) Ważne:

Slides eksportuje slajdy w rozmiarze ustawionym w Page setup, więc jeśli chcesz różne proporcje, robisz różne slajdy z różnymi page setup nie da się w jednej prezentacji. Dlatego:

rekomendacja: osobne prezentacje per format albo kompromis:

jedna prezentacja per format_set (np. Social 1080x1350, Story 1080x1920, Print A2)


alternatywa: jedna prezentacja, ale wszystkie slajdy w jednym rozmiarze (nie polecam)



Ten PRD przyjmuje podejście praktyczne:

1 template prezentacja per format (lub per grupa formatów o tym samym page setup)

SETTINGS trzyma listę template IDs per format


Przykład:

template_id_IG_POST

template_id_IG_STORY

template_id_PRINT_A2


8.5 Nazewnictwo plików

Wzorzec:

"{date}{series}{dj}_{format}.png" Normalizacja:

usuń znaki specjalne

spacje -> "_"

polskie znaki -> bez ogonków (opcjonalnie) Przykład:

"2026-01-12_NEW-WAVE_DJ-Agnieszka-Kozlowska_IG-POST.png"


8.6 Logowanie i audyt

Arkusz LOGS:

timestamp

row_id

action (VALIDATE/RENDER/EXPORT)

format

duration_ms

result (OK/ERROR)

message


Dodatkowo Logger.log do debugowania.

8.7 Retry i batch

Retry ERROR: generuje ponownie tylko te wiersze, które mają ERROR, o ile operator poprawił dane

Generate all TODO: limit max_batch, reszta zostaje TODO

Jeśli skrypt natrafi na limit czasu:

zapisuje postęp

zostawia niedokończone jako TODO (albo PARTIAL, jeśli chcesz dodatkowy status)



9) Uprawnienia i bezpieczeństwo

Skrypt działa na uprawnieniach użytkownika uruchamiającego (standard w Apps Script) Wymogi:

operatorzy muszą mieć dostęp do:

template Slides

folderów ASSETS i OUTPUT


pliki output nie muszą być publiczne, ale muszą być dostępne dla zespołu

jeśli DJ photo przychodzi jako URL zamiast file_id:

preferowane: zawsze file_id z Drive

URL tylko jeśli publiczny i stabilny (mniej bezpieczne, bardziej kapryśne)



10) Wydajność i limity

Ryzyka:

limity czasu Apps Script (ok. kilka minut na wykonanie)

limity operacji na Drive/Slides Mitigacje:

max_batch

cache dostępności plików

minimalizacja kopiowania prezentacji

eksport tylko tego co trzeba (np. nie generuj print jeśli format_set = BASIC)


11) Edge cases

Brak zdjęcia DJa -> opcjonalnie wstaw "placeholder silhouette" z ASSETS

Za długie nazwisko DJa -> fallback:

zmniejsz czcionkę w danym placeholderze (można w Apps Script)

albo skróć (np. "A. Kozlowska") jeśli przekracza limit znaków


Niezgodny format daty -> walidacja i ERROR

Brak uprawnień do pliku -> ERROR z jasnym komunikatem

Puste pole overlay_file_id -> pomiń overlay (nie wywalaj renderu)


12) UX i operacyjność

Operator ma widzieć:

status

ostatni render

link do folderu output

linki do poszczególnych formatów


Operator nie musi dotykać Slides - tylko wypełnia dane


13) Kryteria akceptacji (Acceptance Criteria)

System uznajemy za gotowy, gdy:

1. Dla poprawnego wiersza generuje minimum 2 formaty i zapisuje je w OUTPUT


2. Wypełnia output_folder_link i output_links w Sheets


3. Dla błędnego wiersza ustawia ERROR i wpisuje konkretny powód


4. Batch generuje co najmniej 20 wierszy bez ręcznej interwencji (zależnie od limitów)


5. Szablon zachowuje spójny layout dla wszystkich wygenerowanych coverów (brak losowych przesunięć)


6. Retry działa po poprawieniu danych



14) Plan wdrożenia (MVP -> V1)

MVP (1-2 dni pracy technicznej):

1 format (IG post)

podmiana tekstów + 3 obrazów (BG, DJ, logo)

export PNG

statusy TODO/RENDER/DONE/ERROR

podstawowe logi


V1 (wersja "już serio produkcja"):

3-4 formaty (osobne template IDs)

overlaye i dodatkowe zdjęcia

walidacja kompletna

retry, batch, logi w arkuszu

normalizacja nazw plików

podfoldery per event

prosty "Settings"


15) Definicja "Done"

Operator generuje cover jednym kliknięciem

System nie wymaga ręcznego przeklejania assetów

Błędy są czytelne i naprawialne bez wsparcia dev-a

Output jest zawsze w tym samym miejscu, z tym samym nazewnictwem



---

Minimalna checklista przygotowania (żeby nie było "czemu nie działa")

1. Foldery Drive: ASSETS + OUTPUT


2. Template Slides przygotowany z opisanymi placeholderami (ph_*)


3. Arkusz EVENTS z kolumnami


4. Arkusz SETTINGS z template_id i folderami


5. Apps Script podpięty do arkusza, menu działa
