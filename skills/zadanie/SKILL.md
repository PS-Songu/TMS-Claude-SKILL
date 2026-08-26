---
name: zadanie
description: Zadania w firmowym TMS — zakładanie, materiały do weryfikacji, poprawa opisu i zmiana statusu. Użyj po skończonej robocie, żeby zaproponować zadanie do TMS i opisać, co zrobione, oraz kiedy użytkownik prosi „załóż zadanie", „wrzuć to do TMS", „dodaj task", „zrób z tego zadanie", „dopisz materiały", „popraw opis zadania", „sformatuj to zadanie", a także gdy mówi „zaczynam to", „biorę się za to", „oznacz jako zrobione", „oddaj do weryfikacji" albo „zmień status zadania".
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

## Słownik

Zanim zaproponujesz zadanie, raz na rozmowę pobierz co wolno wpisać:

```bash
curl -s -H "Authorization: Bearer $KLUCZ" "$BASE/api/v1/integrations/dictionary"
```

Zwraca `projects`, `pools` (z `projectId`), `users`, `priorities` oraz `me`
z `canAssignToOthers`. Lista jest przycięta do uprawnień właściciela klucza —
czego tam nie ma, tego nie proponuj. Gdy `canAssignToOthers` jest `false`,
wykonawcą jest zawsze on sam, nawet jeśli rozmowa sugeruje kogoś innego.

Nie zgaduj projektów, pul ani ludzi z pamięci ani z tego repo.

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
- `409 subtasks_pending` — zostały nieskończone podzadania. `409 not_published` —
  zadanie jeszcze nieodblokowane w serii. Powiedz, co blokuje.

Nie obchodź odmowy innym statusem i nie ponawiaj jej w kółko.

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

## Czego nie robisz

Nie zatwierdzasz cudzej roboty i nie odsyłasz jej do poprawy — to decyzja
recenzenta, podejmowana po obejrzeniu zadania w TMS, nie w czacie.
Nie zmieniasz nazwy, terminu, projektu ani wykonawcy istniejącego zadania — to się
robi w TMS. Opis poprawiasz wyłącznie na wyraźną prośbę (patrz „Poprawa opisu").
Nie zakładasz kilku zadań naraz bez osobnego potwierdzenia każdego.
Nie wysyłasz do TMS treści, których nie było w bloku, który człowiek zatwierdził.
