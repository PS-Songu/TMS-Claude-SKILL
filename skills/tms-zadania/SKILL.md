---
name: tms-zadania
description: Zadania w firmowym TMS — zakładanie, materiały do weryfikacji i zmiana statusu. Użyj po skończonej robocie, żeby zaproponować zadanie do TMS i opisać, co zrobione, oraz kiedy użytkownik prosi „załóż zadanie", „wrzuć to do TMS", „dodaj task", „zrób z tego zadanie", „dopisz materiały", a także gdy mówi „zaczynam to", „biorę się za to", „oznacz jako zrobione", „oddaj do weryfikacji" albo „zmień status zadania".
---

# Zadania w TMS

Po skończonej robocie proponujesz zadanie i po potwierdzeniu zakładasz je w TMS.
Zadanie idzie na produkcję pod nazwiskiem właściciela klucza.

## Ustawienia

Wszystko siedzi w `~/.claude/tms.json` (poza tym repo, klucz nigdy do gita):

```json
{
  "baseUrl": "https://tms.example.pl",
  "apiKey": "tms_...",
  "propose": true,
  "rules": "Domyślny projekt: TMS. Zadania dla siebie chyba że mówię inaczej."
}
```

Czytaj go przez `cat ~/.claude/tms.json`. Nie ma pliku → powiedz, że skill nie
jest skonfigurowany, i odeślij do README repo. Nie pytaj o klucz w rozmowie.

`propose: false` → nie proponuj sam z siebie, zakładaj tylko na wyraźną prośbę.
`rules` to prywatne reguły tej osoby — trzymaj się ich, chyba że w rozmowie padło
co innego.

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
Nie edytujesz treści istniejących zadań — nazwę, opis i termin poprawia się w TMS.
Nie zakładasz kilku zadań naraz bez osobnego potwierdzenia każdego.
Nie wysyłasz do TMS treści, których nie było w bloku, który człowiek zatwierdził.
