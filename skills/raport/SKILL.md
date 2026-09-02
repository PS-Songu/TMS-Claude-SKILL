---
name: raport
description: Raport wykonanej roboty z firmowego TMS — co komu wyszło z rąk w danym okresie, z możliwością zapisania tego do dziennika pracy w dokumentach TMS. Użyj, gdy użytkownik pyta „co dziś zrobione", „podsumuj mi dzień", „podsumuj tydzień", „co zrobiłem od poniedziałku", „co Wojtek zrobił wczoraj", „raport z sierpnia", „co wyszło mi z rąk w tym tygodniu", albo prosi „zapisz to do dziennika", „dopisz dzisiejszy dzień do dziennika pracy".
---

# Raport wykonanej roboty

Podsumowujesz robotę, która w podanym okresie wyszła komuś z rąk. Wszystko bierzesz
z TMS — raport ma odbijać stan systemu, a nie go upiększać.

## Ustawienia

Te same co przy zadaniach: `~/.claude/tms.json`, a w przykładach niżej `$KLUCZ` to
`apiKey`, `$BASE` to `baseUrl`. Brak pliku albo brak klucza → powiedz, że skill nie
jest skonfigurowany, i odeślij do `/tms:ustawienia`.

## Co się liczy

Pozycją raportu jest zadanie, w którym wskazana osoba jest **wykonawcą** i które
w danym okresie **wyszło jej z rąk** — czyli ma `doneAt` w zakresie. Nic więcej nie
liczysz; wybór robi serwer, Ty podajesz zakres i osobę.

Kotwicą jest moment wyjścia z rąk, nie moment cudzej decyzji. **Zatwierdzenie przez
kogoś innego dwa dni później nie tworzy drugiej pozycji ani nie przesuwa zadania na
inny dzień** — robota oddana w piątek zostaje w piątku, choćby recenzent kliknął
w poniedziałek. Zadanie z `reworked: true` wracało już do poprawy; powiedz o tym w raporcie,
bo bez tego dzień wygląda na obfitszy, niż był.

Poza raportem zostają **zadania założone, zaczęte i stojące w toku**. To decyzja, nie
przeoczenie: raport pokazuje wyniki, nie ruch. Gdy człowiek dziwi się, że dzień
spędzony na jednej dużej rzeczy jest pusty — powiedz właśnie to, zamiast dosypywać
mu zadań będących w trakcie.

## Jak pytasz

Jedno zapytanie na cały okres:

```bash
curl -s -H "Authorization: Bearer $KLUCZ" --get \
  --data-urlencode "doneFrom=2026-08-31T00:00:00+02:00" \
  --data-urlencode "doneTo=2026-08-31T23:59:59+02:00" \
  --data-urlencode "personId=7" \
  --data-urlencode "limit=300" \
  "$BASE/api/v1/integrations/tasks"
```

Wraca `{"data":{"tasks":[…]}}`. Poza tym, co znasz z zadań (`id` — numer zadania,
`title`, `status`, `statusLabel`, `projectName`; `taskId` to nazwa parametru
zapytania, nie pola w odpowiedzi), każda pozycja niesie trzy pola raportowe:
`doneAt` — moment wyjścia roboty z rąk **tej osoby**, `materialsExcerpt` — początek
jej materiałów do weryfikacji gołym tekstem albo `null`, oraz `reworked`.

### Okres

**Granice doby liczysz sam, w czasie warszawskim, i wysyłasz pełne znaczniki ISO
z przesunięciem.** Sama data (`2026-08-31`) zostanie odrzucona — serwer nie zgaduje,
w jakiej strefie siedzi człowiek. Latem przesunięcie to `+02:00`, zimą `+01:00`.
Nie wysyłaj `Z`: robota oddana wieczorem wpadłaby wtedy do jutra.

Domyślnie „dziś". Poza tym rozumiesz to, co ludzie mówią: „wczoraj", „od
poniedziałku" (od poniedziałku bieżącego tygodnia do teraz), „od 25 sierpnia",
„sierpień" (cały miesiąc), „ostatnie trzy dni". Przy okresie dłuższym niż dzień
powiedz w pierwszym zdaniu, jaki zakres wziąłeś — „od poniedziałku" bywa nie tym
poniedziałkiem, o którym myślał pytający.

### Osoba

Domyślnie właściciel klucza — jego numer to `me.userId` ze słownika, więc `personId`
podajesz zawsze, także przy raporcie o sobie. Kogoś innego rozpoznajesz po tym samym
słowniku:

```bash
curl -s -H "Authorization: Bearer $KLUCZ" "$BASE/api/v1/integrations/dictionary"
```

`personId` to numer osoby z `users`. **Pułapka:** słownik zwraca listę osób tylko
temu, kto ma prawo przypisywania zadań innym (`canAssignToOthers`). Bez tego prawa
widać w nim samego siebie — i wtedy pytania o cudzą robotę nie da się obsłużyć.
Powiedz to wprost i **nie zgaduj numeru**:

```
Nie mam jak rozpoznać, kto to Wojtek — Twój klucz nie widzi listy osób w TMS.
Mogę podsumować tylko Twoją robotę.
```

Imienia spoza słownika nie dopasowuj na siłę do podobnego. Dwa trafienia → pokaż oba
i zapytaj, o kogo chodzi.

### Ucięta lista

`limit` to sufit, nie obietnica. **Gdy wróci dokładnie tyle pozycji, ile wynosi
`limit`, lista mogła zostać ucięta** — napisz o tym pod raportem i zaproponuj węższy
okres. Milczenie zostawia człowieka z raportem, który wygląda na komplet.

## Jak wygląda raport

Raport dzieli się **na projekty**: nazwa projektu, myślnik, a po nim to, co w nim
wyszło z rąk. Jedna linijka na projekt, nie na zadanie.

```
OMS - oferta dostaje w tle dane z Amazona, oferty wyłączone z FBA da się przywrócić
POCZTA - wysyłanie wiadomości ze szkicu nie działało, naprawione
TMS - podstawowe narzędzia diagnostyczne wprowadzone na produkcję
```

Projekt bierzesz z `projectName`. Zadanie bez projektu też dostaje swoją linijkę,
nazwaną tym, czego dotyczy — „POCZTA", „MAGAZYN" — a nie wrzucone do „pozostałych".
Kolejność linijek: najpierw projekt, w którym wyszło najwięcej.

Gdy w jednym projekcie wyszły dwie wyraźnie różne rzeczy, **rozbij go na dwie
linijki** z własnymi nazwami — „TMS" i „SKILL TMS" czytają się lepiej niż jedna
linijka o wszystkim naraz. Dzielisz tematem, nie pulą; trzy linijki z jednego
projektu to już siekanie.

**Bez numerów zadań, bez linków, bez godzin.** Nazwę zadania wplatasz tylko wtedy,
gdy sama coś znaczy; poza tym piszesz, co się zmieniło. Kto chce zajrzeć do
konkretnego zadania, poprosi — wtedy podasz numer i link.

### Konkret zamiast etykiety

**Nazwa projektu to nie jest opis roboty.** „OMS — poprawki", „fixy", „prace nad
wtyczką" nie mówią nic i codziennie wyglądają tak samo. Każda linijka ma nazwać,
**co dokładnie** się zmieniło:

```
źle:     OMS - fixy i poprawki
dobrze:  OMS - naprawa wygasającej sesji w panelu, poprawiony skrypt sprzątający po deployach

źle:     TMS - prace nad wtyczką Claude'a
dobrze:  TMS - wtyczka nazywa recenzenta po imieniu i umie go wskazać
```

Miarą jest to, czy po tygodniu da się odróżnić ten dzień od poprzedniego. Nie da się
— linijka jest za ogólna i wracasz po szczegół do `materialsExcerpt`.

**Dwa zdania na projekt wystarczą, ale muszą być szczegółowe.** Krótko nie znaczy
ogólnie: skracasz, wyrzucając wątki poboczne, nie wypłukując treści z tych, które
zostają.

Rzeczy z jednego wątku łącz: osiem wydań wtyczki to nie osiem pozycji, tylko jedna
mówiąca, co wtyczka przez ten dzień dostała — po nazwie każdej z tych rzeczy. Treść
bierzesz z `materialsExcerpt` — to jedyne miejsce, gdzie stoi, co naprawdę zrobione.
Zadanie bez materiałów wchodzi do raportu samą nazwą; niczego nie dopowiadasz.

Wyróżnij to, co czytający musi wiedzieć: co czeka na weryfikację, co wróciło do
poprawy (`reworked`), co jest cudzą decyzją, a nie jego robotą. Reszta ma prawo
zniknąć w zbiorczym zdaniu. Nazwy stanów czytasz ze `status` — `completed`
(zatwierdzone), `to_verify` (czeka na weryfikację), `rework_needed` (wróciło do
poprawy); cokolwiek innego nazywasz `statusLabel` tak, jak przyszło.

Piszesz dla zwykłego człowieka, nie dla programisty. **Nazwy plików, funkcji
i bibliotek zostawiasz w zadaniu** — w raporcie stoi skutek, który widać z zewnątrz.
„Kursy walut pokazywały wczorajszy odczyt, teraz są świeże" zamiast „godzinny cache
w route.ts oddawał przeterminowaną wartość" — ale też **nie** zamiast „poprawka
w WMS". To, czego robota dotyczyła, nazywasz wprost; wycinasz żargon, nie treść.
Materiały bywają pisane technicznie, bo pisał je wykonawca dla siebie; Twoim
zadaniem jest je przetłumaczyć, nie przepisać ani nie wypłukać. Robota, której
skutku nie da się opisać po ludzku, wchodzi do raportu samą nazwą.

Zaczynasz od rzeczy, nie od ramki. „Dzień poszedł prawie w całości na…", „W tym
tygodniu skupiłeś się na…" — takie otwarcia nic nie niosą i brzmią jak wypracowanie.
Dnia nie oceniasz i nie dorabiasz mu narracji: piszesz, co doszło i co się zmieniło.

Długość liczysz projektami, nie zdaniami: **dzień to linijka na każdy projekt,
w którym coś wyszło**, tydzień to samo z grubszym wyliczeniem, miesiąc — akapit na
projekt. Liczby zadań nie podajesz; nikt jej nie potrzebuje, a raport przez nią
wygląda jak rozliczenie.

```
OMS - domknięte atrybuty amazonowe oferty, adres zdjęcia produktu czeka na weryfikację
TMS - wtyczka umie zmienić nazwę i priorytet zadania, wrócić z „Czeka",
      rozdzielić niczyje zadania i zrobić raport z zapisem do dziennika
WMS - kursy walut się odświeżają, poprawione flagi przy wersji angielskiej i mapa
```

Przy okresie dłuższym niż dzień powiedz w pierwszym zdaniu, jaki zakres wziąłeś —
„od poniedziałku" bywa nie tym poniedziałkiem, o którym myślał pytający. Dzień czytasz
z `doneAt` po warszawsku, tak samo jak liczyłeś granice okresu — inaczej wieczorna
robota wyląduje w dniu następnym. Dni dziel tylko wtedy, gdy to coś zmienia;
zwykle lepiej czyta się okres opisany jako całość.

### Kilka osób

Pytanie o kilka osób to osobne zapytanie na osobę — `personId` przyjmuje jedną.
Każda dostaje własny nagłówek z imieniem, a pod nim swoje linijki projektów. Nie
mieszaj ludzi w jednej linijce, nawet gdy robili to samo zadanie.

```
Wojtek:
OMS - naprawa wygasającej sesji w panelu, poprawiony skrypt sprzątający po deployach,
      przeglądy PR-ów

Piotr:
OMS - oferta dostaje w tle dane z Amazona, oferty wyłączone z FBA da się przywrócić
POCZTA - wysyłanie wiadomości ze szkicu nie działało, naprawione
TMS - wtyczka nazywa recenzenta po imieniu i umie go wskazać
```

Pusty okres kwituj wprost: „W piątek nic nie wyszło Ci z rąk" — bez dorabiania
tłumaczeń, dlaczego tak. Przy kilku osobach człowiek bez domkniętej roboty też
dostaje swoją linijkę; pominięcie go wygląda jak przeoczenie.

Przy raporcie o **cudzej robocie** dołóż zdanie o niepełnym wglądzie — lista jest
przycięta do tego, co widzi właściciel klucza: zadania własne, zlecone przez siebie
i te z projektów, których jest członkiem:

```
To tylko zadania widoczne z Twojego konta — własne, zlecone przez Ciebie
i te z projektów, w których jesteś. Wojtek mógł zrobić więcej.
```

## Dziennik

Zapis do dokumentu w TMS robisz **wyłącznie na prośbę** („zapisz to do dziennika") —
nigdy z własnej inicjatywy, choćby raport wyszedł ładny.

Dokument nazywa się `Dziennik pracy — Imię Nazwisko` i **należy do właściciela klucza,
także gdy raport dotyczy kogoś innego** — piszesz do swojego dziennika, nie do
cudzego.

**Imienia i nazwiska do tytułu nie ma w `me`** — jest tam sam `userId` (plus
`canAssignToOthers` i `canCreateProject`). Nazwisko znajdujesz w liście `users`,
dopasowując po `me.userId`, i przepisujesz dokładnie tak, jak przyszło. Gdy
dopasowania nie ma — bo klucz bez prawa przypisywania widzi w słowniku samego siebie
w okrojonej postaci — **zapytaj człowieka, jak ma brzmieć tytuł**. Imienia z rozmowy
nie bierzesz: dziennik założy się wtedy pod innym tytułem, a przy następnym raporcie
powstanie drugi dokument zamiast nadpisania tego, który już jest.

Kolejność jest zawsze ta sama: **szukaj po tytule → brak, to załóż → pobierz treść →
złóż nową → zapisz.** Pominięcie odczytu kasuje wszystko, co w dzienniku było.

```bash
curl -s -H "Authorization: Bearer $KLUCZ" --get \
  --data-urlencode "title=Dziennik pracy — Jan Kowalski" \
  "$BASE/api/v1/integrations/documents"
```

Wraca `{"data":{"document":…}}`, a `null` znaczy, że dziennika jeszcze nie ma.

**Gdy dziennik ma już części, piszesz do najwyższej.** Szukaj od góry: `(część 3)`,
`(część 2)`, na końcu tytuł bez dopisku — i bierz pierwszy, który się znajdzie. Sam
tytuł bez dopisku zawsze istnieje, więc pytany jako pierwszy trafiałby w część
najstarszą i najpełniejszą, czyli dokładnie w tę, od której uciekliśmy. Nową część
zakładasz dopiero wtedy, gdy najwyższa jest pełna (patrz „Limit treści").

Dziennika nie ma w ogóle? Wtedy go zakładasz. Tytuł jest tekstem pisanym dla ludzi,
więc idzie **z pliku**, nie wpisany wprost w komendę (ogonki potrafią dojść jako
krzaki, a odpowiedź i tak wygląda na sukces):

```bash
cat > /tmp/dziennik.json << 'JSON'
{"title":"Dziennik pracy — Jan Kowalski"}
JSON
curl -s -w '\n%{http_code}' -X POST \
  -H "Authorization: Bearer $KLUCZ" -H "Content-Type: application/json" \
  --data-binary @/tmp/dziennik.json "$BASE/api/v1/integrations/documents"
```

Odpowiedź niesie `id` i `slug`. Treść czytasz po numerze:

```bash
curl -s -H "Authorization: Bearer $KLUCZ" "$BASE/api/v1/integrations/documents/12"
```

W `{"data":{"document":{…}}}` interesuje Cię `content` — cały dokument jako HTML.

### Kształt sekcji

Sekcja dnia to nagłówek z datą, a pod nim **ten sam tekst co w raporcie** — linijka
na projekt, bez numerów zadań i linków. Każdy projekt to własne `<p>` z nazwą
w `<strong>`; list punktowanych nie zakładasz.

**W dzienniku data jest zawsze zapisana jako `<h2>RRRR-MM-DD</h2>`** — niezależnie od
tego, że raport dla człowieka mówi „Poniedziałek 31 sierpnia". Data w tym formacie
sortuje się sama i daje się dopasować bez zgadywania.

```html
<h2>2026-08-31</h2>
<p><strong>OMS</strong> — domknięte atrybuty amazonowe oferty, adres zdjęcia produktu czeka na weryfikację.</p>
<p><strong>TMS</strong> — wtyczka umie zmienić nazwę i priorytet zadania, wrócić z „Czeka" i zrobić raport z zapisem do dziennika.</p>
<p><strong>WMS</strong> — kursy walut się odświeżają, poprawione flagi przy wersji angielskiej i mapa.</p>
```

Gdy raport obejmował kilka osób, każda dostaje w sekcji własny blok poprzedzony
`<p><strong>Imię</strong></p>` — tak jak w rozmowie.

**Raport wielodniowy rozbijasz na dni: każdy dzień okresu dostaje własną sekcję**,
osobno dopasowywaną i osobno nadpisywaną. Zapis tygodnia to pięć sekcji, a nie jedna
pod datą początku okresu. Dzień bez zadań pomijasz — pustej sekcji nie zakładasz.

### Kolejność sekcji

**Sekcje stoją datami malejąco, a nową wstawiasz we właściwe miejsce** — nie zawsze na
górę. Gdy w dzienniku są 31 i 25 sierpnia, a dopisujesz 28, ląduje ono **między nimi**.
Na górę trafia tylko dzień świeższy od wszystkiego, co już jest. Raport uzupełniający
za dzień sprzed tygodnia to normalna prośba, nie wyjątek.

### Nadpisywanie

**Jeśli w treści jest już `<h2>` z tą datą, podmieniasz wszystko od tego nagłówka do
następnego `<h2>` (albo do końca dokumentu).** Nie dokładasz drugiej sekcji pod tą
samą datą — dwa wpisy z jednego dnia to dziennik, który kłamie, a człowiek dowie się
o tym miesiąc później, czytając własną notatkę.

Dopasowujesz **po samej dacie w treści nagłówka**, nie po dosłownym `<h2>2026-08-31</h2>`:
nagłówek bywa zapisany z atrybutami, a dziennik mógł powstać wcześniej ręcznie w TMS.
Gdy w dokumencie stoją nagłówki dni w innym formacie (np. „31 sierpnia 2026"),
**powiedz o tym człowiekowi i zapytaj**, zamiast po cichu doklejać obok drugą sekcję
w swoim formacie — inaczej dziennik dostanie dwa wpisy o jednym dniu.

To nie jest wyjątek, tylko normalny bieg rzeczy: ktoś prosi o zapis po południu,
a potem jeszcze raz wieczorem. Za drugim razem dzień ma być **jeden**, ten pełniejszy.
Powiedz o tym jednym zdaniem: „dzisiejsza sekcja nadpisana, pozostałe dni bez zmian".

Zapis nadpisuje **cały** dokument, więc wysyłasz starą treść ze sklejoną nową sekcją.
Też z pliku — to polski tekst:

```bash
cat > /tmp/tresc.json << 'JSON'
{"content":"<h2>2026-08-31</h2><ul><li>…</li></ul><h2>2026-08-28</h2><ul><li>…</li></ul>"}
JSON
curl -s -w '\n%{http_code}' -X PATCH \
  -H "Authorization: Bearer $KLUCZ" -H "Content-Type: application/json" \
  --data-binary @/tmp/tresc.json "$BASE/api/v1/integrations/documents/12"
```

Po zapisie powiedz, co doszło, i podaj link do dokumentu (`$BASE/dokumenty/<slug>`).

### Limit treści

Dokument w TMS ma sufit **500 KB**. Progiem, na którym przechodzisz dalej, jest
**450 KB treści** — zmierz ją po złożeniu, przed wysłaniem. Powyżej progu nie dopisuj
na siłę: załóż kolejną część (`Dziennik pracy — Imię Nazwisko (część 2)`, potem
`(część 3)` i tak dalej), zapisz dzień tam i powiedz człowiekowi, że dziennik przeszedł
na następną część.

Odbita `400` przy zapisie znaczy to samo — treść przekroczyła limit w walidacji — ale
to ścieżka awaryjna, nie sposób na poznanie limitu. Załóż wtedy kolejną część zamiast
ponawiać zapis w kółko.

Odmowy:
- `403` — właściciel klucza nie może zakładać dokumentów albo nie jest właścicielem
  tego, do którego piszesz. Powiedz to i nie próbuj obejść innym dokumentem.
- `404` — dokumentu nie ma albo jest poza jego zasięgiem.

## Granice

**Źródłem jest wyłącznie TMS.** Raport nie streszcza rozmowy, nie dolicza roboty,
o której wiesz z tej sesji, i nie dopisuje niczego z pamięci — nawet gdy wiesz, że
człowiek zrobił coś jeszcze. Robota bez zadania w TMS w raporcie nie istnieje; możesz
najwyżej zaproponować założenie zadania (patrz skill `zadanie`).

Nierozpoznane imię kwitujesz pytaniem, nie wyborem najbliższego pasującego. To nie
jest błąd wart tłumaczenia się — po prostu brakuje Ci danych, żeby odpowiedzieć.
