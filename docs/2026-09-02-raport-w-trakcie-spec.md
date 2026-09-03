# Sekcja „W trakcie" w raporcie — komenda `/tms:raport`

Data ustaleń: 2026-09-02. Wydanie wtyczki: **0.33.0**.

## Po co to

Raport z 0.30.0 pokazuje wyłącznie robotę domkniętą — to, co w okresie wyszło
komuś z rąk. Decyzję o pominięciu zadań w toku zapisano wtedy świadomie
(spec z 2026-08-31, „raport pokazuje wyniki, nie ruch"), z adnotacją, że gdyby
dzień spędzony na jednej dużej rzeczy wyglądał na pusty, rozszerzenie dopisze się
później. To jest to rozszerzenie.

Powód jest prosty: dzień, w którym nic nie zostało oddane, nie jest dniem pustym.
Raport, który pokazuje samo zero, każe człowiekowi tłumaczyć się z roboty, która
po prostu jeszcze trwa.

## Co się zmienia

Raport dostaje drugą część, oddzieloną poziomą kreską. Nad kreską wszystko zostaje
bez zmian. Pod kreską stoją zadania, przy których wskazana osoba **siedzi teraz** —
status `in_progress` i nic poza tym.

Odstawione („Czeka") i nierozpoczęte zostają poza raportem, tak jak dotąd. Zadanie
odstawione stoi z powodu, którego lista nie pokazuje, a nierozpoczęte to pytanie
„co mam do zrobienia" — na nie odpowiada skill `zadanie` przez `view=mine_active`.

## Skąd dane

```
GET /api/v1/integrations/tasks?view=mine_active&status=in_progress&limit=100
```

Dwa ograniczenia API, zmierzone 2026-09-02:

**`personId` działa wyłącznie razem z `doneFrom`.** Serwer odrzuca każdą inną
kombinację komunikatem „personId działa tylko w raporcie". Zadań w toku innej osoby
nie da się więc pobrać — `view=mine_active` z definicji dotyczy właściciela klucza.
Przy raporcie o cudzej robocie sekcji nie ma, a jej brak jest wyjaśniony zdaniem.

**Nie ma pola „od kiedy w trakcie".** Zadanie niesie `createdAt` i `dueDate`;
momentu wzięcia się za nie nie niesie żadne pole. Wiek zadania liczony z `createdAt`
jest jedynym dostępnym przybliżeniem i trzeba go traktować jak przybliżenie —
zadanie założone w lipcu, a wzięte wczoraj, wygląda w nim na wiszące od lipca.

## Jak wygląda

```
OMS - oferta dostaje w tle dane z Amazona, oferty wyłączone z FBA da się przywrócić
WMS - kursy walut się odświeżają, poprawione flagi przy wersji angielskiej
Zadania bez projektu - podstrona z kodami UFI, naprawiona wysyłka ze szkiców w poczcie

───────────────

W trakcie
WMS — miniaturki karteczek olejkowych na sklep niemiecki
Bez projektu — wtyczka pod pliki feedowe na czeskim sklepie
```

**Ziarno jest inne po obu stronach kreski i to jest celowe.** Nad kreską linijka
przypada na projekt, bo streszcza się tam skutki — osiem wydań wtyczki zlepia się
w jedno zdanie o tym, co wtyczka dostała. Pod kreską linijka przypada na zadanie,
bo „nad czym siedzę" musi dać się wskazać palcem; nazwa projektu idzie z przodu.

Numerów zadań i linków nie ma po żadnej stronie — kto chce zajrzeć, poprosi.

Sekcja jest przy każdym raporcie, niezależnie od okresu. Przy pytaniu o zamknięty
okres („sierpień") pokazuje stan na dziś, nie na koniec tamtego okresu; to jest
świadomy koszt tego, że stan „w trakcie" nie ma daty.

Pustej sekcji nie ma — jest jedna linijka „Nic nie masz w trakcie". Cisza w tym
miejscu wygląda jak przeoczenie wtyczki, a to co innego niż brak otwartej roboty.

## Zadania bez projektu

Przy okazji zmienia się reguła z 0.30.0, która kazała zadaniu bez projektu dostać
własną linijkę nazwaną tematem („POCZTA", „MAGAZYN"). Powód zmiany: w dzienniku
taka linijka jest nie do odróżnienia od prawdziwego projektu z TMS, a zadanie
nazwane „OMS | Pakowanie paczek przez GITÓW" sklejało się z projektem OMS, do
którego nie należy.

Od 0.33.0 wszystkie takie zadania zbierają się w jedną sekcję **„Zadania bez
projektu", zawsze ostatnią** — w raporcie i w dzienniku tak samo, po obu stronach
kreski. Zasada opisywania się nie zmienia: w środku dalej stoi konkret, co się
zmieniło, a nie sama etykieta.

Skala na danych z sierpnia 2026: 149 domkniętych zadań, z tego 8 bez projektu.

## Zadania, które wiszą

Zadanie starsze niż **7 dni** (licząc `createdAt`) nie wchodzi do sekcji po cichu.
Trafia do pytania pod raportem, po nazwie i z datą założenia:

```
W trakcie
WMS — miniaturki karteczek olejkowych na sklep niemiecki

Dwa zadania w OMS wiszą od 31 lipca — równoległy odczyt Allegro i pobieranie
zgłoszeń z poczty. Dopisać je do „W trakcie", czy to martwe wątki?
```

Pytanie idzie **po** raporcie, nie przed. Raport ma być gotowy od razu, a nie po
odpowiedzi na pytanie o rzeczy poboczne.

## Dziennik

**Sekcja „W trakcie" do dziennika nie trafia.** Do dokumentu idzie dalej wyłącznie
część nad kreską.

Dziennik jest zapisem dnia, czytanym miesiącami później. Stan chwili wpisany pod
datą 31 sierpnia skłamie przy pierwszym czytaniu w listopadzie — zadanie dawno
domknięte albo dawno porzucone będzie tam stało jako trwające.

## Czego ta zmiana nie robi

Nie rusza doboru zadań nad kreską, liczenia granic doby, obsługi wielu osób ani
zapisu do dziennika. Nie wprowadza pamięci między raportami: odpowiedź na pytanie
o wiszące zadania dotyczy tego jednego raportu i nie jest nigdzie zapisywana.
