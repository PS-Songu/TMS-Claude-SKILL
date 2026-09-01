---
name: zadanie
description: Zadania i projekty w firmowym TMS — zakładanie zadań, materiały do weryfikacji, poprawa opisu, zmiana statusu i zakładanie projektów. Użyj po skończonej robocie, żeby zaproponować zadanie do TMS i opisać, co zrobione, oraz kiedy użytkownik prosi „załóż zadanie", „wrzuć to do TMS", „załóż zadanie do puli", „niech ktoś to weźmie", „dodaj task", „zrób z tego zadanie", „dopisz materiały", „popraw opis zadania", „sformatuj to zadanie", „załóż projekt", „odhacz punkt", „co zostało w zadaniu", „co zostało w projekcie", „co wisi w puli", „rozdziel niczyje zadania", „kto co bierze", „na czym stanęliśmy", „co następne", „czemu to stoi", „co blokuje to zadanie", „załóż zadanie z tego PR-a", „co jest na tym zdjęciu w zadaniu", „przeczytaj załącznik", „obejrzyj zrzut z zadania", „co pisali w uwagach", „czemu wróciło do poprawy", a także gdy mówi „zaczynam to", „biorę się za to", „oznacz jako zrobione", „oddaj do weryfikacji", „to stoi", „odstawiam to", „czekam na kogoś z tym", „wracam do tego" albo „zmień status zadania", „popraw nazwę zadania", „zmień tytuł", „podnieś priorytet", „to już nie jest pilne".
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

## Treść ZAWSZE z pliku

**Cokolwiek napisanego po polsku wysyłasz przez plik, nigdy wpisane wprost
w komendę.** Tytuł, opis, materiały, komentarz, punkt checklisty, nazwa projektu —
wszystko, co czyta potem człowiek.

```bash
cat > /tmp/tresc.json << 'JSON'
{"title":"Poprawić eksport zamówień","description":"<p>Ogonki i „cudzysłowy” dochodzą całe.</p>"}
JSON
curl -s -w '\n%{http_code}' -X POST \
  -H "Authorization: Bearer $KLUCZ" -H "Content-Type: application/json" \
  --data-binary @/tmp/tresc.json "$BASE/api/v1/integrations/inbound/claude"
```

**Why:** ogonki i cudzysłowy wpisane prosto w komendę potrafią dojść jako krzaki —
zależnie od powłoki i kodowania. Najgorsze jest to, że **wygląda to na sukces**:
serwer odpowiada „przyjąłem", numer zadania wraca, a rozsypany tekst wychodzi
dopiero wtedy, gdy ktoś na to spojrzy w TMS. Zdarzyło się naprawdę — tytuł zapisał
się jako `obs?uga statusu ?Czeka?`.

Wprost w komendzie (`-d '{...}'`) zostają **wyłącznie wartości techniczne**: statusy,
daty, numery, `true`/`false`. Tam nie ma czego zepsuć.

Heredoc musi być cytowany (`<< 'JSON'`), inaczej powłoka zje ukośniki i `$`.
Po wysłaniu **sprawdź w odpowiedzi, jak zapisał się tekst** — jeśli widzisz krzaki
zamiast ogonków, popraw od razu, zamiast meldować gotowe.

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
  --data-binary @/tmp/projekt.json \
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

Treść idzie z pliku (patrz „Treść ZAWSZE z pliku") — tak samo w każdym przykładzie
niżej, gdzie widzisz `--data-binary`:

```bash
cat > /tmp/zadanie.json << 'JSON'
{"title":"Poprawić eksport zamówień","description":"<p>Co i dlaczego.</p>","priority":"high","projectName":"TMS ✅","poolName":"Zadania / błędy","assigneeName":"Jan Kowalski"}
JSON
curl -s -w '\n%{http_code}' -X POST \
  -H "Authorization: Bearer $KLUCZ" -H "Content-Type: application/json" \
  --data-binary @/tmp/zadanie.json \
  "$BASE/api/v1/integrations/inbound/claude"
```

Pola: `title`, `description`, `priority` (`today` | `high` | `medium` | `low` |
`do_not_touch`), `dueDate` (`YYYY-MM-DD`), `projectName`, `poolName`,
`assigneeName`. Nazwy podawaj dokładnie tak, jak przyszły ze słownika.

Odpowiedź `{"data":{"taskId":123,"created":true}}`. Zgłoś numer i link
`$BASE/tasks/123` — jednym zdaniem, żeby dało się od razu zajrzeć i poprawić
na miejscu.

### Zadanie do puli, czyli niczyje

„Załóż zadanie w OMS, ktoś to weźmie", „wrzuć do puli" — **pominięcie
`assigneeName` NIE robi zadania niczyim**. TMS podpisuje wtedy właściciela klucza,
tak samo jakby poprosił o zadanie dla siebie. Zadanie wygląda na wzięte, choć nikt
go nie wziął, a Ty meldujesz sukces.

Niczyje robi się drugim ruchem — oddaniem do puli, jak przyciskiem „Oddaj"
w oknie zadania:

```bash
curl -s -w '\n%{http_code}' -X POST \
  -H "Authorization: Bearer $KLUCZ" -H "Content-Type: application/json" \
  -d '{"unassign":true}' \
  "$BASE/api/v1/integrations/tasks/123/fields"
```

Zrób oba kroki po kolei i powiedz o tym jednym zdaniem — dla człowieka to jedna
czynność („założone i leży w puli, bez wykonawcy"). Gdy oddanie odbije się odmową,
powiedz wprost, że zadanie **zostało podpisane na Ciebie**; milczenie zostawiłoby
je przypisane wbrew temu, o co prosił.

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
  --data-binary @/tmp/opis.json \
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

## Nazwa, priorytet, termin i wykonawca

„Przesuń 1827 na piątek", „zleć to Wojtkowi", „oddaję to z powrotem do puli",
„popraw nazwę na …" — jedno wejście, bo w TMS to jedno okno edycji:

```bash
curl -s -w '\n%{http_code}' -X POST \
  -H "Authorization: Bearer $KLUCZ" -H "Content-Type: application/json" \
  -d '{"dueDate":"2026-09-04"}' \
  "$BASE/api/v1/integrations/tasks/1827/fields"
```

Do wyboru, po jednym na raz albo razem:
- `title` — nowa nazwa zadania (3–200 znaków),
- `priority` — `today` | `high` | `medium` | `low` | `do_not_touch`,
- `dueDate` — `"2026-09-04"` ustawia termin, `null` go zdejmuje,
- `assigneeUserId` — numer osoby ZE SŁOWNIKA (nie imię),
- `unassign: true` — oddanie zadania do puli, czyli zdjęcie siebie.

**Priorytet mów po ludzku, wysyłaj po technicznemu.** „Podnieś 1827 na pilne" to
`today`, „to już nie jest pilne" — zwykle `medium`, ale przy takim luźnym zdaniu
upewnij się, na co ma zejść, zamiast wybierać za człowieka. Nazwy do rozmowy masz
w słowniku (`priorities`); `do_not_touch` znaczy „nie ruszać", nie „najniższy".

**Priorytet jest do zapisu, nie do odczytu.** Żaden odczyt zadań go nie zwraca —
ani wyszukiwanie, ani `taskId`, ani widoki. Ustawiony przed chwilą też nie wróci.
Jedyny sygnał pilności z API to `dueDate`.

Nazwa to treść pisana dla ludzi, więc idzie z pliku (patrz „Treść ZAWSZE z pliku"):

```bash
cat > /tmp/nazwa.json << 'JSON'
{"title":"Poprawić eksport zamówień do BaseLinkera"}
JSON
curl -s -w '\n%{http_code}' -X POST \
  -H "Authorization: Bearer $KLUCZ" -H "Content-Type: application/json" \
  --data-binary @/tmp/nazwa.json "$BASE/api/v1/integrations/tasks/1827/fields"
```

Sam termin czy wykonawca (bez nazwy) mogą iść wprost — to wartości techniczne.

**Nową nazwę pokaż, zanim wyślesz** — obok starej, bo zmiana tytułu jest widoczna
dla wszystkich i nie zostawia po sobie śladu, czym była wcześniej:

```
#1827  było: Poprawić eksport
       ma być: Poprawić eksport zamówień do BaseLinkera

Zmieniam? [tak / popraw / anuluj]
```

Sam z siebie nazw nie poprawiaj — nawet gdy widzisz literówkę albo tytuł niezgodny
z tym, co ostatecznie weszło. Powiedz o tym i zaproponuj.

**Datę zawsze pokaż jako konkretny dzień, zanim wyślesz.** „Piątek" bywa nie tym
piątkiem, o którym myślisz — dzień tygodnia z datą rozwiewa to od razu: „termin na
piątek 4 września?".

**Przepisanie zadania komuś innemu potwierdź.** Tamta osoba zobaczy je u siebie
i dostanie powiadomienie. Oddanie do puli i własny termin — bez ceregieli, to
odwracalne.

Odmowy:
- `403 forbidden` — nazwę, priorytet, termin i wykonawcę zmienia ten, kto zlecił (albo
  lider czy manager). Jesteś tylko wykonawcą cudzego zadania? Możesz je oddać do puli,
  ale nie przestawić terminu ani nazwy — powiedz to i zaproponuj komentarz z prośbą.
- `403 assign_to_others_denied` — właściciel klucza nie ma prawa zlecać innym
  (`canAssignToOthers` w słowniku na `false`). Sprawdź to ZANIM zaproponujesz
  przepisanie, nie po odmowie.
- `409 project_done` — projekt zakończony, zadanie tylko do odczytu.
- `400` — sprzeczne wejście (naraz wykonawca i oddanie do puli), puste, nazwa
  krótsza niż trzy znaki albo priorytet spoza listy.

Projektu ani puli tędy nie zmieniasz — przeniesienie zdejmuje zadanie z jednej
tablicy i wiesza na drugiej, więc robi się to w TMS, patrząc na obie.

## Co do mnie przyszło

„Co mam do zrobienia", „co czeka na moją ocenę", „co oddałem", „co komu zleciłem" —
to pytania o WŁASNE sprawy, nie o tablicę projektu. Do tego służy `view`:

```bash
curl -s -H "Authorization: Bearer $KLUCZ" "$BASE/api/v1/integrations/tasks?view=mine_active&limit=100"
```

Widoki i pytania, na które odpowiadają:
- `mine_active` — „co mam do zrobienia", moje niezakończone.
- `mine_done` — „co zrobiłem". Pytanie o konkretny okres („co dziś zrobione",
  „podsumuj mi tydzień") obsługuje skill `raport`, nie ten widok.
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
curl -s -H "Authorization: Bearer $KLUCZ"   "$BASE/api/v1/integrations/tasks?projectId=7&status=not_started,in_progress,waiting&limit=100"
```

Do wyboru `projectId`, `poolId` (sama pula wystarczy — należy do jednego projektu),
`status` (kilka po przecinku) i `limit` (do 100). Bez `status` wracają wszystkie,
także zakończone — to właśnie odpowiedź na „co zrobione, a co czeka".

W tym widoku są też **zadania bez wykonawcy** — czyli to, co dopiero czeka na
podjęcie. Przy zwykłym wyszukiwaniu po słowach ich nie ma.

Każde zadanie niesie `status`, `statusLabel`, `assignees`, `poolName` i `dueDate`.
**Priorytetu tu nie ma** (patrz „Nazwa, priorytet, termin i wykonawca") — nie
układaj listy „od najpilniejszych" i nie mów, że coś jest pilne, skoro tego nie widzisz.
Odpowiadaj stanem tablicy, nie surową listą:

```
Projekt OMS — 14 zadań

Zrobione       8
W trakcie      2   #61 śledzenie przesyłek (Piotr), #58 raport dzienny (Wojtek)
Odstawione     1   #57 integracja z kurierem (Piotr) — status „Czeka"
Nierozpoczęte  3   w tym 2 bez wykonawcy
```

Status `waiting` („Czeka") to zadanie **odstawione** — ktoś je już miał i wstrzymał.
To co innego niż `not_started`, które po prostu nie ruszyło. Nie zlepiaj obu
w jedną kupkę „czeka" i nie podsuwaj odstawionych, gdy ktoś pyta „co następne" —
one stoją z powodu, którego lista nie pokazuje.

Przy zestawieniu takim jak wyżej linki wypisz pod spodem, po jednym na zadanie —
w tabelce rozwaliłyby układ, ale zniknąć nie mogą.

Gdy człowiek pyta „co następne", pokaż same czekające i zaproponuj jedno — nie
wyliczaj wszystkiego. Kolejność bierzesz z terminu i blokad, bo priorytetu
nie dostajesz; przy wyborze powiedz, czym się kierowałeś, i dodaj, że priorytet
widzi tylko człowiek w TMS. Pusty wynik znaczy tyle, że w JEGO zasięgu widoczności nic
tam nie ma; nie dopowiadaj, czy zadania nie ma, czy tylko go nie widzi.

**Stan czytasz z TMS, nie z pliku w repozytorium.** Tablica w `BOARD.md` czy innym
pliku bywa nieaktualna i nie jest drugim źródłem prawdy — jeśli rozjeżdża się z TMS,
powiedz to, ale nie synchronizuj jej sam.

## Rozdanie niczyich zadań

„Rozdziel niczyje z OMS między mnie, Wojtka i Daniela", „kto co bierze z tej puli" —
gdy nad projektem siedzi kilka osób, a część zadań nie ma wykonawcy.

**Osoby bierzesz z rozmowy, nie z projektu.** Nie zgaduj, kto akurat pracuje nad
tematem — padły trzy imiona, dzielisz między te trzy. Nikogo nie dokładasz od siebie.
Imiona zamień na numery ze słownika; kogoś, kogo tam nie ma, zgłoś zamiast pomijać
po cichu.

Zacznij od jednego zapytania — z blokadami, bo bez nich podzieliłbyś na ślepo:

```bash
curl -s -H "Authorization: Bearer $KLUCZ" \
  "$BASE/api/v1/integrations/tasks?projectId=7&status=not_started&withBlockers=1&limit=100"
```

`withBlockers=1` dokłada przy każdym zadaniu `blockedBy` — numery zadań, na które
ono czeka. **Niczyje poznajesz po pustym `assignees`.**

Jak dzielić, po kolei:

1. **Łańcuch blokad trzymaj w całości.** Zadanie i to, na co czeka, idą do JEDNEJ
   osoby. Rozbite między dwie znaczy, że ktoś siedzi bezczynnie, czekając na kolegę
   przy własnej robocie. Łańcuch bywa dłuższy niż para — idź po `blockedBy`, dopóki
   prowadzi dalej. Gdy bloker ma już wykonawcę, resztę łańcucha dawaj właśnie jemu.
2. **Potem trzymaj razem jedną pulę.** Zadania z tej samej puli do tej samej osoby —
   mniej przeskakiwania między tematami. To reguła słabsza od blokad: gdy się kłócą,
   wygrywa łańcuch.
3. **Resztę rozłóż równo co do liczby.** Nie patrzysz, kto ile ma już na głowie —
   tego nie liczysz i nie udawaj, że wiesz.

**Pokaż CAŁY podział i czekaj na „tak".** Jedno pytanie o wszystko, nie osobne
o każde zadanie:

```
Niczyje w OMS: 7 zadań, 3 osoby

Piotr    #1812 Eksport do BaseLinkera
         #1815 Testy eksportu          ← czeka na #1812
Wojtek   #1820, #1821  (oba z puli „11 — Obsługa klienta")
Daniel   #1808, #1809, #1810  (pula „0 - fixy")

#1815 idzie z #1812, bo bez niego nie ruszy.

Rozdzielam? [tak / popraw / anuluj]
```

Przy każdej grupie dopisz **powód jednym zdaniem** tam, gdzie nie jest oczywisty —
łańcuch blokad zawsze, wspólna pula gdy to ona zdecydowała. Linki do zadań pod spodem,
jak przy zestawieniach.

Po „tak" przypisujesz po kolei (`assigneeUserId`, patrz „Nazwa, priorytet, termin i wykonawca")
i meldujesz jednym zdaniem, co komu przypadło. Odmowa na którymś zadaniu nie
przerywa reszty — dokończ i powiedz na końcu, co nie weszło i dlaczego.

Czego NIE robisz:
- nie rozdajesz bez potwierdzenia — każde przypisanie to powiadomienie u żywej osoby,
- nie ruszasz zadań, które KTOŚ już ma; rozdajesz wyłącznie niczyje,
- nie zgadujesz obciążenia („Wojtek ma dużo roboty") — nie masz tego skąd wiedzieć,
- przy `canAssignToOthers` na `false` mówisz to od razu, ZANIM ułożysz podział;
  wtedy jedyne, co możesz, to wziąć zadania na siebie.

## Treść zadania: opis, zdjęcia, załączniki

Wyszukiwanie daje tytuł, status i wykonawców — nie treść. Po opis sięgasz osobno:

```bash
curl -s -H "Authorization: Bearer $KLUCZ" "$BASE/api/v1/integrations/tasks/1721/description"
```

Wraca `{taskId, title, html, imagesInDescription, images}`. `html` to opis tak, jak
trzyma go edytor, a `images` to zdjęcia w nim wklejone — każde z `name` i `path`.

Zdjęcie wklejone do edytora **nie jest załącznikiem zadania**: nie ma go na liście
załączników i nie szukaj go tam. Pobierasz je ścieżką z `path` (doklej `$BASE`):

```bash
curl -s -H "Authorization: Bearer $KLUCZ" -o /tmp/zrzut.png \
  "$BASE/api/v1/integrations/tasks/1503/images/a7e1a5c1-....png"
```

Potem otwierasz plik jak każdy inny — patrz „Zawartość" niżej. Adres z samego `html`
(`/api/v1/editor-images/…`) tędy nie zadziała, bo chce sesji przeglądarki; bierz `path`.

Co doczepione:

```bash
curl -s -H "Authorization: Bearer $KLUCZ" "$BASE/api/v1/integrations/tasks/1721/attachments"
```

Lista `{id, fileName, mimeType, sizeBytes, kind, uploadedByName, uploadedAt, readable}`.
`kind` to `task` (kontekst zadania: brief, instrukcja) albo `verification` (dowód
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
`author`, `createdAt`, `html`, `replyToAuthor`, `images`. Gdy `total` jest większe niż
to, co dostałeś, powiedz o tym — „ostatnie 30 z 54" — zamiast udawać, że widziałeś całość.

`images` to zdjęcia wklejone do komentarza, pobierane tak samo jak te z opisu. Zrzut
w komentarzu zwykle NIESIE sedno („o, tu się sypie") — obejrzyj go, zanim streścisz wątek.

Streszczaj, nie przepisuj. Człowiek pyta „co tam ustalili", nie „przeczytaj mi wątek".

### Dopisanie komentarza

```bash
curl -s -w '\n%{http_code}' -X POST \
  -H "Authorization: Bearer $KLUCZ" -H "Content-Type: application/json" \
  --data-binary @/tmp/komentarz.json \
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
`waiting` (Czeka), `to_verify` (Do weryfikacji), `completed` (Zakończone),
`rework_needed` (Do poprawy).

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
- „to stoi", „czekam na Wojtka", „odstawiam to na potem" → `waiting`. Bez pytania
  o zgodę, to odwracalne. Powiedz, że zadanie odstawione i **na co czeka** —
  sam status tego nie niesie, a bez tego nikt nie wie, kiedy je wznowić.
- „wracam do tego", „już mogę robić" → **osobne wejście, bez podawania statusu**:

  ```bash
  curl -s -w '\n%{http_code}' -X POST \
    -H "Authorization: Bearer $KLUCZ" \
    "$BASE/api/v1/integrations/tasks/1721/resume"
  ```

  TMS sam wie, dokąd wrócić: zadanie odstawione w toku wraca w tok, odstawione
  przed rozpoczęciem — do nierozpoczętego. **Nie ustawiaj statusu ręcznie** i nie
  pytaj, na co wrócić — zgadywanie kończy się tym, że system twierdzi, że ktoś
  zaczął robotę, której nikt nie tknął. Powiedz potem, w jakim stanie zadanie
  wylądowało. `409 not_waiting` znaczy, że zadanie wcale nie stoi — sprawdź stan,
  zanim powiesz, że coś poszło nie tak.
- Zadanie w stanie `not_started` trzeba najpierw wystartować — TMS nie pozwoli
  oddać nierozpoczętego. Zrób oba kroki po kolei i wspomnij o tym jednym zdaniem,
  bo dla człowieka to jedna czynność.

Czekanie bywa też **postawione przez TMS, nie przez człowieka**: gdy każdy
nieodhaczony punkt checklisty ma blokadę, zadanie samo idzie na „Czeka" i samo
wraca, gdy blokada zniknie. Zadanie na „Czeka" z zablokowanymi punktami sprawdź
przez blokady (patrz wyżej) i powiedz, na co czeka — zamiast proponować, żeby
je ręcznie wznowić, bo automat odstawi je z powrotem.

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
  --data-binary @/tmp/punkty.json \
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
  --data-binary @/tmp/materialy.json \
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

### Odczyt materiałów i uwag do poprawy

To samo wejście czytane, nie pisane — po nie sięgasz, gdy masz **sprawdzić czyjąś
robotę** albo **poprawić własną po uwagach**:

```bash
curl -s -H "Authorization: Bearer $KLUCZ" "$BASE/api/v1/integrations/tasks/1721/materials"
```

Wraca `sections` (przy zadaniu wieloosobowym jedna na wykonawcę, z `author`), a w każdej
`html`, `revisions` (kolejne tury poprawek, od najstarszej) i `reworkComment` — uwagi
weryfikatora do TEJ osoby. Osobno `reworkReason`: uwagi przy odesłaniu całego zadania
do poprawy. Wszędzie `images`, bo materiały pisze się tym samym edytorem co opis.

Gdy zadanie wróciło **Do poprawy**, uwagi są pierwszą rzeczą do przeczytania — razem
ze zrzutami. Zwykle to na nich widać, o co chodzi, a sam tekst mówi „patrz obrazek".

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
Nazwę, termin i wykonawcę zmieniasz na prośbę (patrz „Nazwa, priorytet, termin i wykonawca"), nie z własnej
inicjatywy. Opis poprawiasz wyłącznie na wyraźną prośbę (patrz „Poprawa opisu").
Nie zakładasz kilku zadań naraz bez osobnego potwierdzenia każdego.
Nie zamykasz zgłoszeń błędów z własnej inicjatywy — proponujesz, decyduje człowiek.
Nie przeglądasz repozytorium ani kodu na potrzeby stanu zadań — TMS mówi, co
zrobione i co czeka; jeśli trzeba zajrzeć w kod, to osobna robota, nie ta wtyczka.
Nie wysyłasz do TMS treści, których nie było w bloku, który człowiek zatwierdził.
