---
name: zadanie
description: Zadania i projekty w firmowym TMS — zakładanie zadań, materiały do weryfikacji, poprawa opisu, zmiana statusu i zakładanie projektów. Użyj po skończonej robocie, żeby zaproponować zadanie do TMS i opisać, co zrobione, oraz kiedy użytkownik prosi „załóż zadanie", „wrzuć to do TMS", „dodaj task", „zrób z tego zadanie", „dopisz materiały", „popraw opis zadania", „sformatuj to zadanie", „załóż projekt", „odhacz punkt", „co zostało w zadaniu", „co zostało w projekcie", „co wisi w puli", „na czym stanęliśmy", „co następne", „czemu to stoi", „co blokuje to zadanie", „załóż zadanie z tego PR-a", „co jest na tym zdjęciu w zadaniu", „przeczytaj załącznik", „obejrzyj zrzut z zadania", a także gdy mówi „zaczynam to", „biorę się za to", „oznacz jako zrobione", „oddaj do weryfikacji" albo „zmień status zadania".
---

# Zadania w TMS

Po skończonej robocie proponujesz zadanie i po potwierdzeniu zakładasz je w TMS.
Zadanie idzie na produkcję pod nazwiskiem właściciela klucza.

## Ustawienia

Wszystko siedzi w `~/.claude/tms.json` — poza tym repo, klucz nigdy do gita:

```json
{
  "baseUrl": "https://tms.example.pl",
  "apiKey": "tms_...",
  "propose": true,
  "rules": "Domyślny projekt: TMS. Zadania dla siebie chyba że mówię inaczej.",
  "style": "Krótko, bez ozdobników. Opis maksymalnie trzy zdania.",
  "projectByFolder": { "C:\\Users\\jan\\Desktop\\OMS": "OMS 🚚" }
}
```

Czytaj przez `cat ~/.claude/tms.json`. Plik bywa zapisany ze znacznikiem BOM na
początku — pomiń go, to nie błąd. Brak pliku albo brak `apiKey` → powiedz, że skill
nie jest skonfigurowany, i odeślij do `/tms:ustawienia`. **Nie pytaj o klucz
w rozmowie i nigdy go nie wypisuj.**

`propose: false` → nie proponuj z własnej inicjatywy, zakładaj tylko na wyraźną
prośbę. `rules` to prywatne ustalenia tej osoby — trzymaj się ich, chyba że
w rozmowie padło co innego. `style` dotyczy brzmienia: długości opisu, tonu, tego
czy używać wyliczeń. Reguły mówią CO wpisać, styl JAK to napisać.

`projectByFolder` to mapa katalog na dysku → projekt w TMS. Gdy rozmowa toczy się
w takim katalogu albo gdziekolwiek pod nim, podstaw ten projekt do propozycji
zamiast pytać. Pasuje kilka ścieżek → wygrywa najdłuższa (najbardziej szczegółowa).
Projekt powiedziany wprost w rozmowie ma pierwszeństwo, a nazwy spoza słownika
nie wpisujesz nawet z mapy — powiedz wtedy, że tego projektu nie ma w Twoim
zasięgu. Katalog spoza mapy: ustalasz projekt jak dotąd.

Ustawienia pokazuje i objaśnia `/tms:ustawienia`. Plik ma komentarze `//` — pomiń
je przy czytaniu i NIE kasuj ich, gdyby przyszło Ci coś w nim zmieniać.

W przykładach niżej `$KLUCZ` to `apiKey`, a `$BASE` to `baseUrl`. Podstaw je sam
przy wywołaniu.

## Numer zadania ZAWSZE z linkiem

**Ilekroć wymieniasz zadanie, dawaj link** — założone, ruszone, oddane, zablokowane,
znalezione przy wyszukiwaniu, wymienione mimochodem w podsumowaniu roboty. Bez wyjątku
i bez czekania, aż ktoś poprosi.

Adres składasz z `baseUrl`: `$BASE/tasks/<numer>`.

```
Zadanie #1814 czeka na Wojtka jako „Do weryfikacji"
https://tms.example.pl/tasks/1814
```

Sam numer zmusza człowieka do szukania zadania po numerku — a to jest dokładnie ta
robota, której wtyczka ma go oszczędzić. Przy liście kilku zadań link dawaj przy każdym
(w wierszu albo pod nim), nie tylko przy pierwszym.

Wyjątek jest jeden: gdy w tej samej odpowiedzi ten sam link już padł — nie powtarzaj go
przy każdej wzmiance.

## Słownik — ZAWSZE PIERWSZY

**Zanim cokolwiek zaproponujesz i zanim cokolwiek wyślesz, pobierz słownik.**
Raz na rozmowę, ale przed pierwszą propozycją — bez wyjątków.

```bash
curl -s -H "Authorization: Bearer $KLUCZ" "$BASE/api/v1/integrations/dictionary"
```

Zwraca `projects`, `pools` (z `projectId`), `users`, `priorities` oraz `me`
z `canAssignToOthers` i `canCreateProject`. Wszystko przycięte do uprawnień
właściciela klucza — **czego tam nie ma, tego on nie może**.

### Sprawdzenie uprawnień przed działaniem

Człowiek prosi o zadanie w konkretnym projekcie, a tego projektu **nie ma na
liście**? Powiedz mu to od razu, zanim ułożysz propozycję:

```
W TMS nie masz dostępu do projektu WMS — nie założysz tam zadania.

Możesz: poprosić kogoś o dodanie Cię do projektu, wybrać inny
(TMS ✅, OMS 🚚, …) albo założyć zadanie bez projektu.
```

Nie próbuj „na wszelki wypadek" — odmowa z serwera po pokazaniu gotowej
propozycji wygląda, jakby coś się zepsuło, a to zwykły brak uprawnień.

To samo dotyczy reszty:
- `canAssignToOthers` na `false` → wykonawcą jest wyłącznie właściciel klucza,
  nawet jeśli rozmowa sugeruje kogoś innego. Powiedz o tym, nie zgaduj.
- `canCreateProject` na `false` → nie proponuj zakładania projektu.
- Puli nie ma na liście dla tego projektu → nie wpisuj jej.

Nie zgaduj projektów, pul ani ludzi z pamięci, z rozmowy ani z tego repo. Jedynym
źródłem prawdy jest słownik pobrany TERAZ.

## Zakładanie projektu

Gdy zadanie nie pasuje do żadnego istniejącego projektu, a `canCreateProject`
w słowniku jest `true`, możesz zaproponować nowy. **Tylko na wyraźną prośbę albo
gdy człowiek sam powie, że projektu brakuje** — nie zakładaj projektu przy okazji
zakładania zadania.

```bash
curl -s -w '\n%{http_code}' -X POST \
  -H "Authorization: Bearer $KLUCZ" -H "Content-Type: application/json" \
  -d '{"name":"Nazwa projektu","description":"<p>Po co ten projekt.</p>"}' \
  "$BASE/api/v1/integrations/projects"
```

Pola: `name` (min. 3 znaki, musi być unikalna), `description` (HTML jak w opisach
zadań), `status` — `planned` (domyślny), `active`, `on_hold`, `done`.

Zanim wyślesz, pokaż do zatwierdzenia:

```
Nowy projekt w TMS

Nazwa: Automatyzacja raportów magazynowych
Opis:  Zbiera w jednym miejscu robotę wokół raportów z WMS.
       Na razie dwa zadania, ale będzie ich więcej.

Zakładam? [tak / popraw / anuluj]
```

Właścicielem zostaje właściciel klucza. Odpowiedź niesie `id` i `name` — powiedz,
że projekt powstał, i **dopiero teraz** zakładaj w nim zadanie. Świeży projekt nie
ma pul, więc pola „Pula" nie wypełniaj.

Odmowy:
- `403 forbidden` — właściciel klucza nie może zakładać projektów. Sprawdź to
  wcześniej w słowniku, żeby nie dopytywać po fakcie.
- `409 name_taken` — projekt o tej nazwie już istnieje. Pokaż go i zapytaj, czy
  o niego chodziło.
- `400` — nazwa krótsza niż trzy znaki.

## Duplikaty

Zanim pokażesz propozycję, sprawdź, czy tego samego już ktoś nie zgłosił:

```bash
curl -s -H "Authorization: Bearer $KLUCZ" \
  --get --data-urlencode "query=filtr terminow" --data-urlencode "limit=5" \
  "$BASE/api/v1/integrations/tasks"
```

W `query` daj dwa–trzy nośne słowa z nazwy, którą właśnie ułożyłeś — nie całe
zdanie, bo szuka po dosłownym fragmencie tytułu i opisu. Wynik obejmuje też
zadania zamknięte i cudze, byle mieściły się w zakresie widoczności właściciela
klucza.

Sam oceń, czy trafienie to naprawdę ta sama sprawa, czy tylko zbieżne słowa.
Gdy to samo — zamiast zwykłego bloku pokaż ostrzeżenie i czekaj:

```
Uwaga: w TMS jest już podobne zadanie

#1654  Poprawić filtr terminów na liście zadań
       TMS · W trakcie · Jan Kowalski

To samo? [pokaż mi je / zakładam mimo to / anuluj]
```

Przy zamkniętym trafieniu powiedz to wprost — „to samo zrobiono trzy tygodnie
temu" bywa ważniejsze niż otwarty bliźniak. Nigdy nie blokuj założenia: ostatnie
słowo ma człowiek, czasem podobne zadanie to celowo osobna sprawa.

## Formatowanie

`description` zadania i materiały do weryfikacji to **tekst formatowany** — ten sam
edytor, w którym ludzie piszą ręcznie w TMS. Wysyłaj HTML, nie goły tekst z `\n`:
łamania linii bez znaczników zlewają się w jedną ścianę.

Dozwolone znaczniki: `<p>` `<br>` `<strong>` `<em>` `<u>` `<s>` `<mark>` `<code>`
`<pre>` `<h1>`–`<h4>` `<ul>` `<ol>` `<li>` `<a href>` `<blockquote>` `<hr>`
`<table>` z `<tr>/<th>/<td>`. Reszta jest wycinana przy wyświetlaniu.

Lista do odhaczania — dokładnie w tym kształcie, inaczej TMS nie pokaże checkboxów:

```html
<ul data-type="taskList">
  <li data-checked="false" data-type="taskItem"><label><input type="checkbox"><span></span></label><div><p>Rozdzielić pakowanie zamówień powyżej pięciu pozycji</p></div></li>
  <li data-checked="true" data-type="taskItem"><label><input type="checkbox" checked="checked"><span></span></label><div><p>Zrobione już wcześniej</p></div></li>
</ul>
```

### Ile formatowania

Miarą jest to, czy jest co porządkować — nie to, czy da się użyć znacznika.

- Krótkie zadanie (dwa–trzy zdania) → same `<p>`. Nagłówek nad trzema zdaniami to
  hałas, nie struktura.
- Kilka wątków, ustalenia ze spotkania, wyliczenia → `<h3>` na sekcje i `<ul>` na
  punkty. Typowy podział: co ustalono, co do zrobienia, czego świadomie nie robimy.
- Punkty, które ktoś będzie odhaczał w trakcie roboty → lista zadań z checkboxami
  zamiast `<ul>`. Wszystkie zaczynają się odznaczone (`data-checked="false"`).
- Cytat z ustaleń albo warunek od kogoś z zewnątrz → `<blockquote>`.
- Nazwy plików, komendy, identyfikatory → `<code>`.

Nie używaj tabel do rzeczy, które są listą. Nie pogrubiaj całych zdań — `<strong>`
jest od wyróżnienia paru słów. Nie wstawiaj `<hr>` między akapitami tej samej myśli.

To samo dotyczy materiałów do weryfikacji: „Co zrobione" i „Jak sprawdzić" to
naturalne `<h3>`, a kroki weryfikacji — `<ol>`, bo mają kolejność.

## Propozycja

Po skończonej robocie: najpierw dwa–trzy zdania podsumowania prozą, potem blok:

```
Zadanie do TMS

Nazwa:      Poprawić filtr terminów na liście zadań
Opis:       Filtr „ten tydzień" gubi zadania z soboty i niedzieli.
            Zmiana w widoku listy, dotyczy wszystkich użytkowników.
            Do sprawdzenia też w widoku puli.

Projekt:    TMS
Pula:       Zadania / błędy
Wykonawca:  Jan Kowalski
Priorytet:  wysoki
Termin:     —

Założyć? [tak / popraw / anuluj]
```

Zasady składania:
- Nazwa: czasownik + rzecz, do 200 znaków, bez numerów zadań i żargonu z kodu.
- Opis: prozą, po ludzku — co jest do zrobienia i czego dotyczy. Bez nazw
  plików, funkcji i komend; to ma zrozumieć osoba, która nie siedziała w kodzie.
  W bloku pokazujesz go zwykłym tekstem, ale do TMS wysyłasz jako HTML — patrz
  „Formatowanie" wyżej. W bloku nie pokazujesz znaczników.
- Priorytet: `na dziś` tylko gdy termin jest dzisiaj, `wysoki` gdy blokuje
  kogoś lub psuje robotę na produkcji, inaczej `średni`.
- Projekt: z mapy `projectByFolder`, gdy katalog rozmowy do niej pasuje (patrz
  „Ustawienia"). Nie pytaj wtedy o projekt — pokaż go w bloku, człowiek poprawi
  przez „popraw", jeśli tym razem chodzi o co innego.
- Pola, których nie da się ustalić — myślnik. Nie wymyślaj puli ani terminu.
- Bez strzałek i uzasadnień przy polach. Dlaczego akurat ta osoba czy priorytet —
  tłumacz dopiero na pytanie.

Czekasz na odpowiedź. `popraw` → nanieś zmianę i pokaż blok jeszcze raz.
`anuluj` → nic nie zakładasz, temat zamknięty. Nie zakładaj bez wyraźnego „tak".

## Założenie

```bash
curl -s -w '\n%{http_code}' -X POST \
  -H "Authorization: Bearer $KLUCZ" -H "Content-Type: application/json" \
  -d '{"title":"...","description":"...","priority":"high","projectName":"TMS","poolName":"Zadania / błędy","assigneeName":"Jan Kowalski"}' \
  "$BASE/api/v1/integrations/inbound/claude"
```

Pola: `title`, `description`, `priority` (`today` | `high` | `medium` | `low` |
`do_not_touch`), `dueDate` (`YYYY-MM-DD`), `projectName`, `poolName`,
`assigneeName`. Nazwy podawaj dokładnie tak, jak przyszły ze słownika.

Odpowiedź `{"data":{"taskId":123,"created":true}}`. Zgłoś numer i link
`$BASE/tasks/123` — jednym zdaniem, żeby dało się od razu zajrzeć i poprawić
na miejscu.

Gdy zadanie powstało z roboty właśnie skończonej w tej rozmowie, nie kończysz na
numerze — przechodzisz od razu do „Materiałów do weryfikacji" niżej.

Gdy zadanie rozkłada się na kilka wyraźnych kroków, dopisz je jako punkty
checklisty (patrz „Podzadania") zamiast wyliczać w opisie — postęp widać wtedy
na liście zadań.

Kiedy coś nie wyjdzie:
- `401` — klucz odwołany albo zły. Powiedz to i odeślij do Ustawień w TMS.
- `403` — właściciel klucza nie jest członkiem tego projektu. Powiedz który
  projekt i zaproponuj inny ze słownika.
- `400` z informacją o niejednoznacznej nazwie — pokaż pasujące pozycje ze
  słownika i zapytaj, o którą chodzi.

Nie ponawiaj po błędzie w kółko i nie obchodź go innym polem. Powiedz, co się
stało, i zapytaj.

## Poprawa opisu

Opis istniejącego zadania poprawiasz **tylko na wyraźną prośbę** („popraw opis
1766", „sformatuj to zadanie") — nigdy z własnej inicjatywy, bo to cudza treść.

```bash
curl -s -w '\n%{http_code}' -X POST \
  -H "Authorization: Bearer $KLUCZ" -H "Content-Type: application/json" \
  -d '{"html":"<h3>...</h3><p>...</p>"}' \
  "$BASE/api/v1/integrations/tasks/1766/description"
```

Zapis nadpisuje cały opis, więc **najpierw go przeczytaj** — zadanie wraca
z wyszukiwania bez treści opisu, więc poproś człowieka o wklejenie jej albo
odczytaj z rozmowy, jeśli to Ty ją pisałeś. Nie zgaduj, co tam było.

Przy samym formatowaniu (bez zmiany treści) trzymaj się reguły: **zmienia się
struktura, nie słowa**. Nagłówki, listy, pogrubienia — tak. Skracanie,
przestawianie zdań, poprawianie stylu — nie, chyba że człowiek o to poprosi.
Przed wysłaniem pokaż, co się zmieni.

Odmowy:
- `403 forbidden` — opis to domena zlecającego. Właściciel klucza może poprawiać
  opisy zadań, które sam założył (albo dowolne, gdy jest managerem lub liderem
  z uprawnieniem). Wykonawca cudzego zadania — nie; jego domeną są materiały do
  weryfikacji. Powiedz to wprost i zaproponuj materiały.
- `409 project_done` — projekt zakończony, zadanie tylko do odczytu.
- `404` — zadania nie ma albo jest poza jego zasięgiem.

## Termin i wykonawca

„Przesuń 1827 na piątek", „zleć to Wojtkowi", „oddaję to z powrotem do puli" —
jedno wejście, bo w TMS to jedno okno edycji:

```bash
curl -s -w '\n%{http_code}' -X POST \
  -H "Authorization: Bearer $KLUCZ" -H "Content-Type: application/json" \
  -d '{"dueDate":"2026-09-04"}' \
  "$BASE/api/v1/integrations/tasks/1827/fields"
```

Do wyboru, po jednym na raz albo razem:
- `dueDate` — `"2026-09-04"` ustawia termin, `null` go zdejmuje,
- `assigneeUserId` — numer osoby ZE SŁOWNIKA (nie imię),
- `unassign: true` — oddanie zadania do puli, czyli zdjęcie siebie.

**Datę zawsze pokaż jako konkretny dzień, zanim wyślesz.** „Piątek" bywa nie tym
piątkiem, o którym myślisz — dzień tygodnia z datą rozwiewa to od razu: „termin na
piątek 4 września?".

**Przepisanie zadania komuś innemu potwierdź.** Tamta osoba zobaczy je u siebie
i dostanie powiadomienie. Oddanie do puli i własny termin — bez ceregieli, to
odwracalne.

Odmowy:
- `403 forbidden` — termin i wykonawcę zmienia ten, kto zadanie zlecił (albo lider
  czy manager). Jesteś tylko wykonawcą cudzego zadania? Możesz je oddać do puli,
  ale nie przestawić terminu — powiedz to i zaproponuj komentarz z prośbą.
- `403 assign_to_others_denied` — właściciel klucza nie ma prawa zlecać innym
  (`canAssignToOthers` w słowniku na `false`). Sprawdź to ZANIM zaproponujesz
  przepisanie, nie po odmowie.
- `409 project_done` — projekt zakończony, zadanie tylko do odczytu.
- `400` — sprzeczne wejście (naraz wykonawca i oddanie do puli) albo puste.

Nazwy, projektu ani puli tędy nie zmieniasz — to się robi w TMS, patrząc na tablicę.

## Co do mnie przyszło

„Co mam do zrobienia", „co czeka na moją ocenę", „co oddałem", „co komu zleciłem" —
to pytania o WŁASNE sprawy, nie o tablicę projektu. Do tego służy `view`:

```bash
curl -s -H "Authorization: Bearer $KLUCZ" "$BASE/api/v1/integrations/tasks?view=mine_active&limit=100"
```

Widoki i pytania, na które odpowiadają:
- `mine_active` — „co mam do zrobienia", moje niezakończone.
- `mine_done` — „co zrobiłem".
- `to_verify` — „co czeka na MOJĄ ocenę", cudza robota oddana do sprawdzenia.
- `awaiting` — „co oddałem i wisi" u kogoś do zatwierdzenia.
- `delegated` — „co zleciłem innym".

Zadanie z `unopened` na `true` jest świeże: zlecone właścicielowi klucza i jeszcze
przez niego nieotwierane w TMS. To najbliższe temu, co człowiek nazywa „nowe u mnie",
więc wypisz takie osobno albo oznacz — ale nie rób z tego alarmu.

```
Masz 5 zadań w toku, 2 jeszcze nieotwierane:

Nowe    #1841 Poprawić eksport faktur (zlecił Wojtek)
        #1839 Zdjęcia do karty produktu (zlecił Kuba)
W toku  #1833, #1827, #1791
```

Nie ma tu widoku puli — „pula" w TMS znaczy co innego niż w rozmowie. Zadania
czekające w puli projektu czytasz przez `poolId` (patrz niżej).

## Stan projektu i puli

Pytanie „co zostało w OMS", „co wisi w puli Fixy", „na czym stanęliśmy" to pytanie
o tablicę, nie o jedno zadanie. Numery projektów i pul masz w słowniku — użyj ich:

```bash
curl -s -H "Authorization: Bearer $KLUCZ"   "$BASE/api/v1/integrations/tasks?projectId=7&status=not_started,in_progress&limit=100"
```

Do wyboru `projectId`, `poolId` (sama pula wystarczy — należy do jednego projektu),
`status` (kilka po przecinku) i `limit` (do 100). Bez `status` wracają wszystkie,
także zakończone — to właśnie odpowiedź na „co zrobione, a co czeka".

W tym widoku są też **zadania bez wykonawcy** — czyli to, co dopiero czeka na
podjęcie. Przy zwykłym wyszukiwaniu po słowach ich nie ma.

Każde zadanie niesie `status`, `statusLabel`, `assignees`, `poolName` i `dueDate`.
Odpowiadaj stanem tablicy, nie surową listą:

```
Projekt OMS — 14 zadań

Zrobione     8
W trakcie    2   #61 śledzenie przesyłek (Piotr), #58 raport dzienny (Wojtek)
Czeka        4   w tym 2 bez wykonawcy
```

Przy zestawieniu takim jak wyżej linki wypisz pod spodem, po jednym na zadanie —
w tabelce rozwaliłyby układ, ale zniknąć nie mogą.

Gdy człowiek pyta „co następne", pokaż same czekające i zaproponuj jedno — nie
wyliczaj wszystkiego. Pusty wynik znaczy tyle, że w JEGO zasięgu widoczności nic
tam nie ma; nie dopowiadaj, czy zadania nie ma, czy tylko go nie widzi.

**Stan czytasz z TMS, nie z pliku w repozytorium.** Tablica w `BOARD.md` czy innym
pliku bywa nieaktualna i nie jest drugim źródłem prawdy — jeśli rozjeżdża się z TMS,
powiedz to, ale nie synchronizuj jej sam.

## Treść zadania: opis, zdjęcia, załączniki

Wyszukiwanie daje tytuł, status i wykonawców — nie treść. Po opis sięgasz osobno:

```bash
curl -s -H "Authorization: Bearer $KLUCZ" "$BASE/api/v1/integrations/tasks/1721/description"
```

Wraca `{taskId, title, html, imagesInDescription}`. `html` to opis tak, jak trzyma go
edytor. `imagesInDescription` mówi, ile zdjęć siedzi w środku — **samych obrazów w tym
polu nie ma**, bo w TMS zdjęcie wklejone do opisu jest zwykłym załącznikiem zadania.

Co doczepione:

```bash
curl -s -H "Authorization: Bearer $KLUCZ" "$BASE/api/v1/integrations/tasks/1721/attachments"
```

Lista `{id, fileName, mimeType, sizeBytes, kind, uploadedByName, uploadedAt, readable}`.
`kind` to `task` (załącznik zadania, w tym obrazy z opisu) albo `verification` (dowód
pracy) — zawęzisz przez `?kind=task`. `readable: false` znaczy, że plik przekracza limit
25 MB; powiedz to zamiast próbować go ściągać.

Zawartość — **zapisz do pliku i otwórz go**:

```bash
curl -s -H "Authorization: Bearer $KLUCZ" -o /tmp/zrzut.png \
  "$BASE/api/v1/integrations/task-attachments/45"
```

Potem czytasz plik zwykłym narzędziem do odczytu. Obrazy widzisz naprawdę — co jest
zaznaczone strzałką, jaki komunikat błędu jest na zrzucie, co pokazuje wykres. Tak samo
PDF-y, pliki tekstowe, CSV, logi i kod.

Kiedy po to sięgasz:
- **zadanie mówi „jak na zrzucie"** — obejrzyj zrzut, zamiast pytać, co na nim jest,
- **człowiek prosi wprost** („co jest na tym zdjęciu w 1721", „przeczytaj załącznik"),
- **bierzesz się za zadanie**, a opis ma obrazy albo pliki — zajrzyj, zanim zaczniesz.

Nie pobieraj wszystkiego hurtem „na wszelki wypadek". Bierz to, co potrzebne do roboty,
o którą chodzi — każdy plik to koszt i czas.

Czego nie otworzysz wprost: **Worda i Excela**. Powiedz to po ludzku („to plik Excela,
nie odczytam go stąd") i zapytaj, czy człowiek nie woli wkleić tego, co istotne.

Odmowy:
- `404` — zadania albo pliku nie ma, lub są poza zasięgiem właściciela klucza. Numer
  załącznika sprawdzamy przez zadanie, do którego należy, więc cudzy plik też da `404`.
- `413 file_too_large` — plik ponad 25 MB. Nie próbuj obejść.
- `400 invalid_kind` — dozwolone są tylko `task` i `verification`.

Plików do TMS **nie wysyłasz** — dodaje się je w przeglądarce.

### Wysłanie pliku

Plik Z DYSKU — log, zrzut z testu, wygenerowany raport — dokładasz do zadania:

```bash
curl -s -w '\n%{http_code}' -X POST \
  -H "Authorization: Bearer $KLUCZ" \
  -F "file=@/sciezka/do/log.txt" -F "kind=verification" \
  "$BASE/api/v1/integrations/tasks/1721/attachments"
```

`kind=verification` (domyślne) to materiały do weryfikacji — dowód skończonej
roboty; wgrywa je wykonawca. `kind=task` to kontekst zadania, brief czy instrukcja —
wgrywa go ten, kto zadanie zlecił. Nie odwracaj tego: dowód pracy w załącznikach
zadania wygląda, jakby ktoś dołożył wymagania.

**Powiedz, co wysyłasz i dokąd, zanim wyślesz.** Plik zostaje przy zadaniu na stałe
i widzą go wszyscy — to nie jest ruch odwracalny po cichu.

**Czego nie wyślesz:** zrzutu wklejonego do okna rozmowy. Dla Ciebie to obraz
w kontekście, nie plik na dysku — nie ma czego przekazać dalej. Powiedz to wprost
i poproś o zapisanie go na dysku albo o ścieżkę; nie udawaj, że się nie udało
z innego powodu, i nie próbuj odtwarzać obrazka.

Odmowy:
- `403 forbidden` — nie ta sekcja dla tej osoby (patrz podział wyżej).
- `413` — plik albo cała sekcja przekracza limit. Serwer podaje, ile zajęte
  i ile zostało — powtórz to człowiekowi zamiast samego „za duży".
- `409 project_done` — projekt zakończony, zadanie tylko do odczytu.

## Komentarze

Komentarze są w TMS główną rozmową o zadaniu. Opis mówi, co było do zrobienia na
starcie; co ustalono po drodze — mówią komentarze. Czytaj je zawsze, gdy masz
zrozumieć stan sprawy, a nie tylko zobaczyć tytuł.

```bash
curl -s -H "Authorization: Bearer $KLUCZ" "$BASE/api/v1/integrations/tasks/1721/comments"
```

Wracają `total` i ostatnie wpisy (domyślnie 30, `limit` do 100), od najstarszego:
`author`, `createdAt`, `html`, `replyToAuthor`. Gdy `total` jest większe niż to, co
dostałeś, powiedz o tym — „ostatnie 30 z 54" — zamiast udawać, że widziałeś całość.

Streszczaj, nie przepisuj. Człowiek pyta „co tam ustalili", nie „przeczytaj mi wątek".

### Dopisanie komentarza

```bash
curl -s -w '\n%{http_code}' -X POST \
  -H "Authorization: Bearer $KLUCZ" -H "Content-Type: application/json" \
  -d '{"html":"<p>Poprawione, zostaje jeszcze eksport.</p>"}' \
  "$BASE/api/v1/integrations/tasks/1721/comments"
```

Treść formatujesz tak jak opis (patrz „Formatowanie"). Odpowiedź na czyjś wpis —
dołóż `replyToId` z odczytu.

**Pokaż treść i poczekaj na „tak".** Komentarz widzą wszyscy przy zadaniu i idzie
z niego powiadomienie — to nie jest ruch odwracalny po cichu, jak start zadania.

Komentujesz pod każdym zadaniem, które właściciel klucza widzi, także cudzym —
to jego prawo w TMS. Nie mylą się za to trzy rzeczy, każda ma swoje miejsce:
- **opis** — co jest do zrobienia; domena zlecającego,
- **materiały do weryfikacji** — co zrobiono i jak to sprawdzić; domena wykonawcy,
- **komentarz** — rozmowa, pytanie, ustalenie; każdego, kto widzi zadanie.

Gdy człowiek mówi „dopisz, że…", zwykle chodzi o komentarz. Gdy mówi „opisz, co
zrobiłeś" przy własnym zadaniu — o materiały. W razie wątpliwości zapytaj, zamiast
wpisywać ustalenie z rozmowy do opisu cudzego zadania.

Komentarzy nie poprawiasz i nie kasujesz — także własnych. Porządki w wątku robi
się w TMS, gdzie widać kontekst.

Odmowy: `404` — zadania nie ma albo jest poza zasięgiem właściciela klucza.

## Zgłoszenia błędów

Część zadań powstaje ze **zgłoszeń** — ktoś zgłosił błąd, ktoś zrobił z tego zadanie.
Zgłoszenie żyje wtedy własnym życiem: ma swój status i swojego autora, który czeka na
odpowiedź. Zamknięcie zadania samo z siebie NIC z nim nie robi.

```bash
curl -s -H "Authorization: Bearer $KLUCZ" "$BASE/api/v1/integrations/tasks/1721/bug-reports"
```

Pusta lista = zadanie nie powstało ze zgłoszenia; nie drąż. Lista bywa dłuższa niż
jednoelementowa — ten sam błąd zgłasza czasem kilka osób i wszystkie te zgłoszenia
wskazują jedno zadanie. Każde niesie `status`, `statusLabel`, autora i `canManage`.

### Kiedy proponować zmianę statusu

- **Start zadania**, a zgłoszenie ma `nowe` → zaproponuj „w trakcie". Autor zgłoszenia
  widzi wtedy, że ktoś się tym zajął.
- **Zamknięcie albo oddanie do weryfikacji** → zaproponuj „rozwiązane".

```bash
curl -s -w '\n%{http_code}' -X POST \
  -H "Authorization: Bearer $KLUCZ" -H "Content-Type: application/json" \
  -d '{"status":"rozwiazane"}' \
  "$BASE/api/v1/integrations/bug-reports/12"
```

**Zawsze pytaj, nigdy sam.** Domknięcie zgłoszenia wysyła powiadomienie do osoby,
która je zgłosiła — inaczej niż start zadania, to nie jest ruch po cichu. Przy kilku
zgłoszeniach naraz zapytaj o wszystkie jednym pytaniem, nie po kolei.

`canManage` na `false` → powiedz, że statusami zgłoszeń zarządza ten, kto je
rozpatruje, i **nie próbuj**. Zadanie idzie swoim torem, zgłoszenie zostaje.

Uzasadnienia nie dopisuj z własnej inicjatywy. Puste zostawia notatkę, którą
rozpatrujący wpisał wcześniej w module; wysłane — nadpisuje ją.

### Odrzucenie i duplikat

Na wyraźną prośbę: `{"status":"odrzucone","resolutionNote":"..."}` — tu uzasadnienie
jest na miejscu, bo autor zgłoszenia dowie się, dlaczego. Duplikat wymaga wskazania
zgłoszenia głównego: `{"status":"duplikat","duplicateOfId":8}`. Numer znajdź, zamiast
zgadywać:

```bash
curl -s -H "Authorization: Bearer $KLUCZ" "$BASE/api/v1/integrations/bug-reports?status=nowe"
```

Sam z siebie nie proponuj ani odrzucenia, ani duplikatu — obie decyzje wymagają
porównania z resztą zgłoszeń, a Ty widzisz tylko wycinek.

### Powiązanie ze zgłoszeniem

Zadanie da się przypiąć do zgłoszenia (i odpiąć):

```bash
curl -s -w '\n%{http_code}' -X POST \
  -H "Authorization: Bearer $KLUCZ" -H "Content-Type: application/json" \
  -d '{"taskId":1841}' \
  "$BASE/api/v1/integrations/bug-reports/12"
```

`{"taskId":null}` odpina. Przydaje się, gdy zadanie powstało z rozmowy, a dopiero
potem okazało się, że dotyczy zgłoszonego błędu. Zgłoszenia bez zadania znajdziesz
przez `?unlinked=1`.

Pokaż jedno i drugie — zgłoszenie i zadanie — i zapytaj, zanim zwiążesz. Powiązanie
stempluje zadanie jako pochodzące ze zgłoszenia, więc widać je potem w TMS.

Status i powiązanie to dwie osobne decyzje: nie załatwiaj obu jednym pytaniem.

## Blokady

Zadanie potrafi czekać na inne — przez punkt checklisty z przypiętym cudzym
zadaniem. Checklista mówi tylko `blocked: true`; czym jest blokada, mówi to:

```bash
curl -s -H "Authorization: Bearer $KLUCZ" "$BASE/api/v1/integrations/tasks/1721/blockers"
```

Wracają obie strony: `waitingOn` (na co czeka to zadanie — `subtaskText`, `taskId`,
`title`, `statusLabel`, `assignees`, `done`) i `blocking` (kogo samo wstrzymuje).

Sięgasz po to, gdy:
- punkt checklisty ma `blocked: true` i trzeba powiedzieć, na co czeka,
- oddanie odbiło się o `409 subtasks_pending`, a punkt jest zablokowany — wtedy
  nie ma czego odhaczać, jest na co czekać,
- człowiek pyta „czemu to stoi" albo „co odblokuje 1721".

Powiedz po ludzku, kto jest po drugiej stronie: „czeka na #1699 *Klucze integracyjne*
— w trakcie, u Wojtka", i dołóż link do tamtego zadania — to na nie ktoś będzie chciał
zajrzeć. Wpis z `done: true` to blokada już domknięta; punkt odhaczy
się sam, nie ma tam nic do zrobienia.

### Założenie blokady

„To czeka na 1827", „zablokuj ten punkt, dopóki Wojtek nie skończy" — blokada wisi
na PUNKCIE checklisty i wskazuje zadanie, na które ten punkt czeka. Nie ma punktu?
Najpierw go dopisz (patrz „Podzadania"), potem przypnij.

```bash
curl -s -w '\n%{http_code}' -X PUT \
  -H "Authorization: Bearer $KLUCZ" -H "Content-Type: application/json" \
  -d '{"blockerTaskId":1827}' \
  "$BASE/api/v1/integrations/tasks/1721/subtasks/45/blocker"
```

Zdjęcie — `DELETE` tej samej ścieżki, bez treści. Punkt wraca wtedy nieodhaczony,
nawet jeśli bloker był zamknięty: roboty nikt za niego nie zrobił.

Zanim przypniesz, ustal KTÓRE zadanie blokuje — po numerze albo wyszukaniem, jak przy
zmianie statusu. Pokaż oba zadania i zapytaj; przypięcie wysyła powiadomienie do
wykonawcy blokera, więc to nie jest ruch po cichu.

Punktu z blokadą nie odhaczysz ręcznie i nie próbuj — odhaczy się sam, gdy bloker
zostanie zamknięty. Nowego zadania-blokera wtyczka nie zakłada jednym ruchem: gdy
blokera jeszcze nie ma, załóż zadanie normalnie, z potwierdzeniem, i dopiero je przypnij.

Odmowy:
- `403 forbidden` — blokady stawia twórca zadania albo jego wykonawca. Przy samych
  punktach checklisty reguła jest szersza, więc „mogę dopisać punkt, ale nie mogę go
  zablokować" to normalna sytuacja, nie błąd.
- `409 cycle` — zadania zablokowałyby się nawzajem, choćby przez łańcuch pośredników.
  Powiedz, że tak się nie da, i pokaż, co już na co czeka.
- `404 blocker_not_found` — zadania-blokera nie ma albo właściciel klucza go nie widzi.
- `409 no_blocker` przy zdejmowaniu — na tym punkcie nic nie wisi.


## Zmiana statusu

Statusy w TMS: `not_started` (Nierozpoczęty), `in_progress` (W trakcie),
`to_verify` (Do weryfikacji), `completed` (Zakończone), `rework_needed`
(Do poprawy).

Najpierw ustal, o które zadanie chodzi. Numer podany wprost → pobierz je po
`taskId`. Opis słowny („to o filtrach") → poszukaj przez `query` jak przy
duplikatach. Kilka trafień — pokaż listę i zapytaj. Zero trafień — powiedz to,
nie zgaduj.

```bash
curl -s -H "Authorization: Bearer $KLUCZ" "$BASE/api/v1/integrations/tasks?taskId=1721"
```

Zadanie wraca z `status`, `canSelfComplete` i `canVerify`. Zmiana:

```bash
curl -s -w '\n%{http_code}' -X POST \
  -H "Authorization: Bearer $KLUCZ" -H "Content-Type: application/json" \
  -d '{"status":"in_progress"}' \
  "$BASE/api/v1/integrations/tasks/1721/status"
```

Co z czym:
- „zaczynam", „biorę się za to" → `in_progress`. Bez pytania o zgodę — to
  odwracalne. Powiedz jednym zdaniem, które zadanie ruszyłeś, z linkiem.
  Zadanie z puli (bez wykonawcy) start bierze na właściciela klucza — tak jak
  „Weź" w oknie zadania. Powiedz to jednym zdaniem: wzięte i w toku. Zadania,
  które ma już wykonawcę, start nie przejmuje.
- „zrobione", „skończone" → gdy `canSelfComplete` jest `true`, proponuj
  `completed`. Gdy `false`, zadanie czeka na czyjąś weryfikację — proponuj
  `to_verify` i powiedz dlaczego. **Czekaj na „tak"**, dopiero potem wysyłaj.
- Zadanie w stanie `not_started` trzeba najpierw wystartować — TMS nie pozwoli
  oddać nierozpoczętego. Zrób oba kroki po kolei i wspomnij o tym jednym zdaniem,
  bo dla człowieka to jedna czynność.

Odmowy:
- `403 forbidden_transition` — uprawnienia nie dają tego przejścia. Najczęściej:
  zadanie zlecił ktoś inny, więc zamyka je on, nie wykonawca. Powiedz to po
  ludzku i zaproponuj oddanie do weryfikacji.
- `404` — zadania nie ma albo właściciel klucza go nie widzi. Nie drąż, o które
  chodziło; poproś o numer.
- `409 subtasks_pending` — zostały nieodhaczone punkty checklisty. Pokaż je
  (patrz „Podzadania") i zapytaj, czy odhaczyć. `409 not_published` — zadanie
  jeszcze nieodblokowane w serii. Powiedz, co blokuje.

Nie obchodź odmowy innym statusem i nie ponawiaj jej w kółko.

## Podzadania

Checklista wewnątrz zadania: krótkie punkty do odhaczenia. To nie są osobne
zadania — nie mają wykonawcy, statusu ani terminu, więc nie znajdziesz ich
wyszukiwaniem. Zawsze idziesz przez numer zadania.

Podgląd:

```bash
curl -s -H "Authorization: Bearer $KLUCZ" "$BASE/api/v1/integrations/tasks/1721/subtasks"
```

Wraca lista `{id, text, description, done, blocked}`.

Dopisanie punktów — całą listą naraz, nie po jednym:

```bash
curl -s -w '\n%{http_code}' -X POST \
  -H "Authorization: Bearer $KLUCZ" -H "Content-Type: application/json" \
  -d '{"items":[{"text":"Pierwszy punkt"},{"text":"Drugi punkt"}]}' \
  "$BASE/api/v1/integrations/tasks/1721/subtasks"
```

Odhaczenie (`done: false` odznacza z powrotem):

```bash
curl -s -w '\n%{http_code}' -X PATCH \
  -H "Authorization: Bearer $KLUCZ" -H "Content-Type: application/json" \
  -d '{"done":true}' \
  "$BASE/api/v1/integrations/tasks/1721/subtasks/45"
```

Kiedy z tego korzystasz:
- **Zadanie z kilku wyraźnych kroków** — przy zakładaniu wpisz je jako punkty
  checklisty zamiast wyliczenia w opisie. Wtedy widać postęp na liście zadań.
  Opis zostaje na „po co i dlaczego", punkty niosą „co po kolei".
- **Odmowa `subtasks_pending`** — pobierz listę, pokaż nieodhaczone i zapytaj,
  czy odhaczyć. Nie odhaczaj sam pod pretekstem oddania zadania.
- **Człowiek mówi wprost** („odhacz drugi punkt", „co jeszcze zostało w 1766").

Punkt z `blocked: true` ma przypięte cudze zadanie — odhaczy się sam, gdy tamto
zostanie zamknięte. Ręcznie się nie da (`409 subtask_blocked`); powiedz, na co
czeka, zamiast próbować.

Odmowy:
- `404` — zadania nie ma, jest poza zasięgiem właściciela klucza, albo punkt nie
  należy do tego zadania.
- `403 forbidden` — zadanie widoczne, ale checklisty w nim nie ruszysz. Punkty
  dopisuje i odhacza twórca zadania, wykonawca, właściciel lub manager projektu.
- `409 project_done` / `409 task_archived` — zadanie tylko do odczytu.
- `409 limit_exceeded` — limit punktów na zadanie. Zaproponuj rozbicie na osobne
  zadania.

## Materiały do weryfikacji

Po założeniu zadania z właśnie skończonej roboty prowadzisz ciąg dalej **bez
pytania**: ustawiasz `in_progress`, zapisujesz materiały, i dopiero przed
oddaniem się zatrzymujesz.

```bash
curl -s -w '\n%{http_code}' -X POST \
  -H "Authorization: Bearer $KLUCZ" -H "Content-Type: application/json" \
  -d '{"html":"<p><strong>Co zrobione</strong></p><p>...</p>"}' \
  "$BASE/api/v1/integrations/tasks/1721/materials"
```

Treść to prosty HTML: `<p>`, `<strong>`, `<ul><li>`. Bez nagłówków, tabel
i stylów. Dwie części, w tej kolejności:

1. **Co zrobione** — dwa–cztery zdania prozą: co się zmieniło i dlaczego.
2. **Jak sprawdzić** — lista kroków dla osoby weryfikującej: gdzie zajrzeć, co
   powinna zobaczyć, czego świadomie nie ruszaliśmy. Konkretnie — „wejdź w
   Ustawienia → Klucze i wydaj klucz, powinien pokazać się raz", nie „sprawdź
   czy działa".

Nazwy plików, funkcji i komend tylko wtedy, gdy weryfikujący bez nich nie ruszy.
Zwykle nie ruszy bez nich w zadaniach czysto technicznych — wtedy podaj, ale
resztę pisz po ludzku.

Po zapisie pokaż krótko, co wpisałeś, i link do zadania.

**Zanim zapytasz o oddanie, odczytaj zadanie** — `canSelfComplete` przychodzi
wyłącznie z odczytu, samo założenie zwraca goły numer:

```bash
curl -s -H "Authorization: Bearer $KLUCZ" "$BASE/api/v1/integrations/tasks?taskId=1721"
```

Bez tego kroku nie wiesz, czy zadanie ma w ogóle kogoś, kto je sprawdzi. Zadanie
własne bez recenzenta oddane „do weryfikacji" ląduje na liście recenzji u tego,
kto je zlecił — czyli u autora roboty. Kierownik projektu tej listy nie widzi,
więc zadanie utyka tam, gdzie nikt go nie szuka.

Osobno, na prośbę („dopisz materiały do 1654"): znajdź zadanie jak przy zmianie
statusu i zapisz. Działa na każdym zadaniu, w którym właściciel klucza jest
wykonawcą — także zleconym mu wcześniej przez kogoś innego.

Odmowy:
- `409 task_not_started` — zadanie jeszcze nierozpoczęte, TMS nie pokazuje wtedy
  sekcji materiałów. Ustaw `in_progress` i powtórz.
- `409 project_done` — projekt zakończony, zadanie tylko do odczytu.
- `403 forbidden` / `403 not_assignee` — właściciel klucza nie jest wykonawcą
  tego zadania. Przy zadaniu wieloosobowym pisze się wyłącznie do własnej części.
- `404` — zadania nie ma albo jest poza jego zasięgiem.

### Oddanie po materiałach

Tu się zatrzymujesz i pytasz. Wyjście podpowiada `canSelfComplete` odczytany
przed chwilą — nie zgaduj go i nie pytaj z góry o weryfikację.

`true` (zadanie własne, bez cudzego recenzenta) — nikt tego nie sprawdza, więc
jedyne sensowne wyjście to zamknięcie:

```
Zadanie 1721 założone, materiały wpisane.
https://tms.example.pl/tasks/1721

Zamknąć? [zakończone / jeszcze nie]
```

`false` — czeka na kogoś i dopiero wtedy:

```
Oddać? [do weryfikacji / jeszcze nie]
```

`jeszcze nie` → zostaje `in_progress`, temat zamknięty.

Po oddaniu potwierdź jednym zdaniem — **z linkiem**, bo to ostatnia rzecz, jaką
człowiek widzi z całej roboty, i często jedyna, do której wraca:

```
Zadanie #1814 czeka na Wojtka jako „Do weryfikacji"
https://tms.example.pl/tasks/1814
```

## Zadanie z pull requesta

„Załóż zadanie z tego PR-a" — treść bierzesz z GitHuba, nie z pamięci:

```bash
gh pr view 55 --json number,title,body,url,state,headRefName,files
```

Z tego składasz zwykłą propozycję (blok jak w „Propozycji", potwierdzenie jak zawsze):

- **nazwa** — tytuł PR-a, oczyszczony z prefiksów typu `feat:` i numeru gałęzi,
- **opis** — po co ta zmiana, dwa–trzy zdania z opisu PR-a własnymi słowami; nie
  wklejaj całego `body`, zwłaszcza szablonu z checkboxami,
- **materiały** — link do PR-a i co sprawdzić; dopisujesz je dopiero po założeniu,
  tak samo jak przy robocie z rozmowy.

Numer PR-a i link do niego dawaj **zawsze**, w opisie albo w materiałach — bez tego
zadanie nie prowadzi z powrotem do kodu.

Gdy PR jest już scalony, zadanie i tak ma sens (ślad w changelogu), ale powiedz to
w propozycji: człowiek może chcieć od razu `completed` zamiast `in_progress`.

Bez numeru PR-a nie zgaduj — zapytaj o numer albo o link. Nie przeglądaj repozytorium
w poszukiwaniu „tego właściwego" PR-a.

## Czego nie robisz

Nie zatwierdzasz cudzej roboty i nie odsyłasz jej do poprawy — to decyzja
recenzenta, podejmowana po obejrzeniu zadania w TMS, nie w czacie.
Nie zmieniasz nazwy, projektu ani puli istniejącego zadania — to się robi w TMS.
Termin i wykonawcę zmieniasz na prośbę (patrz „Termin i wykonawca"), nie z własnej
inicjatywy. Opis poprawiasz wyłącznie na wyraźną prośbę (patrz „Poprawa opisu").
Nie zakładasz kilku zadań naraz bez osobnego potwierdzenia każdego.
Nie zamykasz zgłoszeń błędów z własnej inicjatywy — proponujesz, decyduje człowiek.
Nie przeglądasz repozytorium ani kodu na potrzeby stanu zadań — TMS mówi, co
zrobione i co czeka; jeśli trzeba zajrzeć w kod, to osobna robota, nie ta wtyczka.
Nie wysyłasz do TMS treści, których nie było w bloku, który człowiek zatwierdził.
