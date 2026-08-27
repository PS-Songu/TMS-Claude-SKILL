---
name: zadanie
description: Zadania i projekty w firmowym TMS — zakładanie zadań, materiały do weryfikacji, poprawa opisu, zmiana statusu i zakładanie projektów. Użyj po skończonej robocie, żeby zaproponować zadanie do TMS i opisać, co zrobione, oraz kiedy użytkownik prosi „załóż zadanie", „wrzuć to do TMS", „dodaj task", „zrób z tego zadanie", „dopisz materiały", „popraw opis zadania", „sformatuj to zadanie", „załóż projekt", „odhacz punkt", „co zostało w zadaniu", „co zostało w projekcie", „co wisi w puli", „na czym stanęliśmy", „co następne", „czemu to stoi", „co blokuje to zadanie", „załóż zadanie z tego PR-a", a także gdy mówi „zaczynam to", „biorę się za to", „oznacz jako zrobione", „oddaj do weryfikacji" albo „zmień status zadania".
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
  "style": "Krótko, bez ozdobników. Opis maksymalnie trzy zdania."
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

Ustawienia pokazuje i objaśnia `/tms:ustawienia`. Plik ma komentarze `//` — pomiń
je przy czytaniu i NIE kasuj ich, gdyby przyszło Ci coś w nim zmieniać.

W przykładach niżej `$KLUCZ` to `apiKey`, a `$BASE` to `baseUrl`. Podstaw je sam
przy wywołaniu.

W przykładach niżej `$KLUCZ` to klucz z pliku, a `$BASE` — adres TMS z okienka
(albo z pliku, gdy okienko puste). Podstaw je sam przy wywołaniu.

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

Gdy człowiek pyta „co następne", pokaż same czekające i zaproponuj jedno — nie
wyliczaj wszystkiego. Pusty wynik znaczy tyle, że w JEGO zasięgu widoczności nic
tam nie ma; nie dopowiadaj, czy zadania nie ma, czy tylko go nie widzi.

**Stan czytasz z TMS, nie z pliku w repozytorium.** Tablica w `BOARD.md` czy innym
pliku bywa nieaktualna i nie jest drugim źródłem prawdy — jeśli rozjeżdża się z TMS,
powiedz to, ale nie synchronizuj jej sam.

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
— w trakcie, u Wojtka". Wpis z `done: true` to blokada już domknięta; punkt odhaczy
się sam, nie ma tam nic do zrobienia.

Blokad nie zakładasz i nie zdejmujesz — to się robi w TMS.

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
  odwracalne. Powiedz jednym zdaniem, które zadanie ruszyłeś.
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

Tu się zatrzymujesz i pytasz. Podpowiedz właściwe wyjście, patrząc na
`canSelfComplete` zadania:

```
Zadanie 1721 założone, materiały wpisane.
https://tms.example.pl/tasks/1721

Oddać? [do weryfikacji / jeszcze nie]
```

Gdy `canSelfComplete` jest `true` — zamiast „do weryfikacji" proponuj
„zakończone", bo zadanie jest własne i nikt go nie sprawdza. `jeszcze nie` →
zostaje `in_progress`, temat zamknięty.

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
Nie zmieniasz nazwy, terminu, projektu ani wykonawcy istniejącego zadania — to się
robi w TMS. Opis poprawiasz wyłącznie na wyraźną prośbę (patrz „Poprawa opisu").
Nie zakładasz kilku zadań naraz bez osobnego potwierdzenia każdego.
Nie przeglądasz repozytorium ani kodu na potrzeby stanu zadań — TMS mówi, co
zrobione i co czeka; jeśli trzeba zajrzeć w kod, to osobna robota, nie ta wtyczka.
Nie wysyłasz do TMS treści, których nie było w bloku, który człowiek zatwierdził.
