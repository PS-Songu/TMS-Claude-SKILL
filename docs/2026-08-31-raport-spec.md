# Raport wykonanej roboty — komenda `/tms:raport`

Data ustaleń: 2026-08-31. Wydanie wtyczki: **0.30.0**.

## Po co to

Po dniu pracy trzeba powiedzieć, co z rąk wyszło — sobie do dziennika, komuś do
rozliczenia. Dziś oznacza to klikanie po TMS i przepisywanie tytułów. Wtyczka ma
to złożyć sama: przeczytać zadania, wybrać te domknięte w podanym okresie i podać
gotowe podsumowanie, z możliwością zapisania go do dziennika w module dokumentów.

Źródłem jest wyłącznie TMS. Raport nie streszcza rozmowy i nie dopisuje niczego
z pamięci modelu — ma odbijać stan systemu, a nie go upiększać.

## Jak to działa dla użytkownika

Komenda `/tms:raport`, ale też zwykłe zdanie: „co dziś zrobione", „podsumuj mi
tydzień", „co Wojtek zrobił wczoraj". Wyzwalacze wpisane tak, żeby nie trzeba
było pamiętać nazwy komendy.

**Osoba.** Domyślnie właściciel klucza. Można wskazać kogoś innego imieniem —
rozpoznanie po liście osób ze słownika integracji. Uwaga: słownik zwraca listę
osób tylko temu, kto ma prawo przypisywania zadań innym; bez tego prawa widać
w nim samego siebie. Wtedy zamiast zgadywać, wtyczka mówi wprost, że nie ma jak
rozpoznać osoby.

**Okres.** Domyślnie dziś. Rozumiane po ludzku: „wczoraj", „od poniedziałku",
„od 25 sierpnia", „sierpień", „ostatnie trzy dni". Doba liczona według czasu
warszawskiego, nie UTC — inaczej robota oddana wieczorem wpadałaby do jutra.

## Co wchodzi do raportu

Pozycją jest zadanie, w którym wskazana osoba jest **wykonawcą** i które w danym
okresie **wyszło jej z rąk**:

- zostało oddane do weryfikacji (`submittedForReviewAt` w zakresie), albo
- zostało domknięte bez weryfikacji — zadania własne, które przez zatwierdzanie
  nie przechodzą (`completedAt` / `verifiedAt` w zakresie).

Kotwicą jest moment wyjścia z rąk. Cudze zatwierdzenie dwa dni później **nie**
tworzy drugiej pozycji ani nie przesuwa zadania na inny dzień. Zadanie oddane
ponownie po poprawkach liczy się od ostatniego oddania (`lastSubmittedAt`),
z adnotacją, że to poprawka.

Stan doklejany jest jako znacznik: *zatwierdzone*, *czeka na weryfikację*,
*wróciło do poprawy*.

Świadomie poza raportem: zadania założone, zaczęte i stojące w toku. Decyzja
z 2026-08-31 — raport pokazuje wyniki, nie ruch. Gdyby okazało się, że dzień
spędzony na jednej dużej rzeczy wygląda na pusty, rozszerzenie dopisze się
później.

## Jak wygląda

Grupowanie po dniach, od najnowszego, w dniu po projekcie. Pozycja niesie numer
zadania, tytuł, projekt, godzinę, znacznik stanu i **jedno zdanie streszczone
z materiałów do weryfikacji**. Bez materiałów zostaje sama linijka — nic się nie
dopisuje od siebie.

Na końcu liczba zadań. Przy pytaniu o cudzą robotę dochodzi zdanie o tym, że
widok obejmuje tylko zadania, do których pytający ma wgląd (własne, zlecone przez
siebie oraz wszystko z projektów, których jest członkiem; z uprawnieniem podglądu
wszystkich zadań — całość). Raport ma powiedzieć, że jest niepełny, zamiast
udawać komplet.

## Dziennik w JF Docs

Zapis **wyłącznie na prośbę** — domyślnie raport zostaje w rozmowie.

Dokument nazywa się „Dziennik pracy — Imię Nazwisko" i należy do osoby, która
raport zleciła (także wtedy, gdy dotyczy kogoś innego). Każdy dzień to sekcja
z nagłówkiem-datą, najnowsze na górze.

**Idempotencja.** Ponowny raport za dzień, który w dzienniku już jest, nadpisuje
tamtą sekcję; nagłówek z datą jest kotwicą. Bez tego dwie prośby o „dzisiaj"
zostawiłyby dwie sprzeczne sekcje pod tą samą datą.

**Limit treści.** Dokument w TMS ma sufit 500 KB. Przy zbliżaniu się do niego
wtyczka zakłada „Dziennik pracy — Imię Nazwisko (część 2)" i pisze dalej tam,
zamiast wywalić się na zapisie.

## Zmiany w TMS — dwa osobne PR-y

**PR 1 — zakresy dat w wyszukiwaniu zadań dla integracji.** `TaskListFilters` ma
dziś tylko `dueDateFrom` / `dueDateTo`, czyli termin wykonania, a nie datę
zdarzenia. Do dorobienia filtr po dacie oddania i domknięcia oraz podniesienie
limitu wyników — dziś zwraca się najwyżej dwadzieścia, a tydzień roboty się w tym
nie mieści. Widoczność bez zmian: liczy się sama, w `listTasks`.

**PR 2 — dokumenty dla klucza osobistego.** Dziś `/api/v1/documents` wymaga
sesji z przeglądarki. Potrzebne wejście integracyjne: znajdź po tytule, załóż,
dopisz treść. Zero własnych reguł uprawnień — te same funkcje, których używa
interfejs (prawo zakładania dokumentów; edycja własnych albo jako admin).

## Czego tu nie ma

- Raportu zespołowego „co się działo w projekcie" — na razie tylko robota
  wskazanej osoby.
- Analizy repozytorium, PR-ów i commitów. Przegląd kodu agent robi sam; wtyczka
  jest oknem na TMS.
- Wysyłki raportu gdziekolwiek poza dokument w TMS.

## Kolejność

Najpierw ten raport, potem cztery rzeczy z backlogu, przesądzone 2026-08-31:
komentarz do istniejącego zadania, załączniki z dysku, numer PR-a w materiałach
oraz relacje blokad (które zadanie blokuje które). Przy okazji odczytu zadania
wtyczka ma widzieć pełen obraz: tytuł, opis, komentarze, załączniki, zdjęcia
w opisie oraz podzadania wraz z ich opisami, zdjęciami i blokadami.
