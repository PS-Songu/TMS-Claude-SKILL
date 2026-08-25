---
name: tms-zadania
description: Zakładanie zadań w firmowym TMS. Użyj po skończonej robocie, żeby zaproponować zadanie do TMS, oraz kiedy użytkownik prosi „załóż zadanie", „wrzuć to do TMS", „dodaj task", „zrób z tego zadanie".
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

## Propozycja

Po skończonej robocie: najpierw dwa–trzy zdania podsumowania prozą, potem blok:

```
Zadanie do TMS

Nazwa:      Poprawić rozpoznawanie kodów EAN przy przyjęciu
Opis:       Skaner gubi ostatnią cyfrę przy kodach 13-znakowych.
            Zmiana w module przyjęć, dotyczy wszystkich magazynierów.
            Do sprawdzenia też przy wydaniu.

Projekt:    TMS
Pula:       Magazyn / błędy
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
  -d '{"title":"...","description":"...","priority":"high","projectName":"TMS","poolName":"Magazyn / błędy","assigneeName":"Jan Kowalski"}' \
  "$BASE/api/v1/integrations/inbound/claude"
```

Pola: `title`, `description`, `priority` (`today` | `high` | `medium` | `low` |
`do_not_touch`), `dueDate` (`YYYY-MM-DD`), `projectName`, `poolName`,
`assigneeName`. Nazwy podawaj dokładnie tak, jak przyszły ze słownika.

Odpowiedź `{"data":{"taskId":123,"created":true}}`. Zgłoś numer i link
`$BASE/tasks/123` — jednym zdaniem, żeby dało się od razu zajrzeć i poprawić
na miejscu.

Kiedy coś nie wyjdzie:
- `401` — klucz odwołany albo zły. Powiedz to i odeślij do Ustawień w TMS.
- `403` — właściciel klucza nie jest członkiem tego projektu. Powiedz który
  projekt i zaproponuj inny ze słownika.
- `400` z informacją o niejednoznacznej nazwie — pokaż pasujące pozycje ze
  słownika i zapytaj, o którą chodzi.

Nie ponawiaj po błędzie w kółko i nie obchodź go innym polem. Powiedz, co się
stało, i zapytaj.

## Czego nie robisz

Nie zmieniasz i nie zamykasz istniejących zadań — poprawki robi się w TMS.
Nie zakładasz kilku zadań naraz bez osobnego potwierdzenia każdego.
Nie wysyłasz do TMS treści, których nie było w bloku, który człowiek zatwierdził.
