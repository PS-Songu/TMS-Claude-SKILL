# Skill do Claude — zadania w TMS

## Czym to jest

Dodatek do Claude'a, który po skończonej robocie sam zakłada zadanie w TMS.
Zamiast przepisywać ręcznie co zostało zrobione i co jeszcze trzeba dorobić,
Claude sam składa nazwę, opis, projekt, wykonawcę i priorytet, pokazuje to do
zatwierdzenia i po potwierdzeniu wysyła prosto na produkcję.

Dodatek jest firmowy i wieloosobowy — każdy pracownik ma go u siebie, z własnym
kluczem i własnymi regułami. Zadania zakładają się pod nazwiskiem tej osoby,
która z Claude'a korzysta, a nie pod wspólnym kontem.

## Jak to działa

1. Kończysz robotę z Claude'em.
2. Claude daje krótkie podsumowanie tego, co zrobił.
3. Pod spodem pokazuje gotowe zadanie:

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

4. Odpowiadasz `tak`, `popraw` (wtedy mówisz co zmienić) albo `anuluj`.
5. Po `tak` zadanie ląduje w TMS, a Ty dostajesz jego numer i link — żeby od razu
   zobaczyć jak wyszło i ewentualnie poprawić na miejscu.

Zadania założone tą drogą mają w TMS własną ikonkę źródła, tak jak te z Discorda —
od razu widać, że przyszły z rozmowy z Claude'em.

Claude nie zgaduje projektów i osób z pamięci — listę pobiera z TMS na bieżąco,
żeby nie rozjechała się z tym, co faktycznie jest w systemie.

## Zmiana statusu

Poza zakładaniem zadań Claude umie też ruszać te, które już są. Mówisz zwykłym
językiem — „zaczynam 1721", „biorę się za to o filtrach", „oznacz jako zrobione" —
a on znajduje zadanie, pokazuje co znalazł i zmienia status.

Rozpoczęcie pracy dzieje się od razu, bo to odwracalne. Zamknięcie zadania albo
oddanie do weryfikacji zawsze czeka na Twoje „tak".

Przy „zrobione" Claude sam sprawdza, czy zadanie da się zamknąć od ręki. Jeśli
zlecił je ktoś inny albo czeka na recenzenta, zaproponuje oddanie do weryfikacji
zamiast zamknięcia — i powie dlaczego.

Wszystko dzieje się **na Twoich uprawnieniach**: Claude może dokładnie tyle, co
Ty w przeglądarce. Nie ruszy zadania, którego nie widzisz, i nie zamknie takiego,
którego sam nie mógłbyś zamknąć. Zatwierdzania cudzej roboty i odsyłania jej do
poprawy wtyczka nie robi w ogóle — to decyzje recenzenta, podejmowane w TMS.



## Formatowanie opisów

Opis zadania i materiały do weryfikacji trafiają do TMS **sformatowane**, tak jak
gdybyś pisał je ręcznie w edytorze: nagłówki sekcji, listy, pogrubienia, cytaty,
a tam gdzie to ma sens — listy z checkboxami do odhaczania w trakcie roboty.

Claude nie formatuje na siłę. Krótkie zadanie zostaje zwykłym akapitem; sekcje
i listy pojawiają się dopiero wtedy, gdy jest co porządkować — kilka wątków,
ustalenia ze spotkania, rzeczy do zrobienia i te świadomie zostawione na potem.


## Poprawa opisu

Zadanie, które już jest w TMS, można kazać Claude'owi posprzątać — „popraw opis 1766",
„sformatuj to zadanie". Przyda się przy starszych zadaniach, założonych zanim wtyczka
umiała formatować.

Zasada jest jedna: **zmienia się struktura, nie słowa**. Claude rozbije ścianę tekstu na
sekcje i listy, ale nie skróci ani nie przepisze treści, chyba że wyraźnie o to poprosisz.
Pokaże, co się zmieni, zanim wyśle.

Opis zadania to domena osoby, która je zleciła — poprawisz więc zadania własne, a cudze
tylko wtedy, gdy jesteś managerem albo liderem. Jeśli jesteś w zadaniu wykonawcą, Twoim
miejscem są materiały do weryfikacji, nie opis.

## Materiały do weryfikacji

Po skończonej robocie Claude nie zostawia zadania pustego. Gdy zatwierdzisz
propozycję, jednym ciągiem: zakłada zadanie, ustawia je na „W trakcie" i wpisuje
materiały do weryfikacji — co zostało zrobione, a pod tym kroki dla osoby, która
ma to sprawdzić.

Potem się zatrzymuje i pyta, czy oddać. Proponuje właściwe wyjście: „Zakończone",
gdy zadanie jest Twoje i nikt go nie weryfikuje, „Do weryfikacji", gdy czeka na
kogoś. Możesz też powiedzieć „jeszcze nie" i zostawić je w trakcie.

Działa też osobno — „dopisz materiały do 1654" uzupełni dowolne zadanie, w którym
jesteś wykonawcą, także takie, które ktoś zlecił Ci wcześniej. Przy zadaniach
wieloosobowych Claude pisze wyłącznie do Twojej części.

## Podobne zadania

Zanim Claude pokaże propozycję nowego zadania, sprawdza w TMS, czy tego samego
już ktoś nie zgłosił. Jeśli znajdzie coś podobnego — również zamkniętego albo
cudzego — pokaże je i zapyta, czy to ta sama sprawa. Decyzja zawsze należy do
Ciebie; czasem podobne zadanie to celowo osobny temat.

## Instalacja

### 1. Klucz

W TMS: Ustawienia → Klucze → „Wydaj klucz". Klucz pokazuje się raz — skopiuj go
od razu. Zgubiony odwołujesz i wydajesz nowy, w tym samym miejscu.

### 2. Wtyczka

W rozmowie z Claude'em, dwie komendy:

```
/plugin marketplace add jf-investing/TMS-Claude-SKILL
/plugin install tms-zadania@jf-tms
```

Repozytorium jest publiczne, więc instalacja nie wymaga żadnych uprawnień ani
logowania do GitHuba.

Po instalacji wtyczka jest widoczna w „Manage Plugins" — stamtąd idą też
aktualizacje, nie trzeba nic kopiować ręcznie.

### 3. Automatyczne aktualizacje (warto)

Marketplace'y spoza Anthropic mają aktualizacje domyślnie **wyłączone**, więc bez
tego kroku nowe wersje wtyczki trzeba za każdym razem ściągać ręcznie przyciskiem
odświeżania.

W rozmowie z Claude'em napisz `/plugin`, przejdź na zakładkę **Marketplaces**,
wejdź w pozycję **jf-tms** (Enter na niej, nie ikonka odświeżania obok) i wybierz
**Enable auto-update**.


**W okienku „Manage Plugins" w edytorze tej opcji nie ma** — pozycje marketplace'ów
są tam zwykłym tekstem, bez wejścia w szczegóły. Wtedy włącz to w pliku ustawień.
Otwórz `%USERPROFILE%\.claude\settings.json` i przy wpisie `jf-tms` dopisz jedną
linię:

```json
"jf-tms": {
  "source": {
    "source": "git",
    "url": "https://github.com/jf-investing/TMS-Claude-SKILL.git"
  },
  "autoUpdate": true
}
```

Zadziała od następnego uruchomienia Claude'a.

Od tej pory po każdym starcie rozmowy Claude sam sprawdza w tle, czy jest nowsza
wersja. Gdy coś pobierze, dostaniesz powiadomienie z prośbą o `/reload-plugins` —
albo nowa wersja wczyta się przy kolejnym uruchomieniu. Bieżąca rozmowa zawsze
pracuje na tym, co miała w chwili startu, więc nic nie zmieni Ci się w trakcie.

### 4. Ustawienia

Dwa miejsca: **klucz w pliku**, **reszta w okienku wtyczki**.

**Klucz.** W PowerShellu:

```powershell
notepad "$env:USERPROFILE\.claude\tms.json"
```

Notatnik zapyta, czy utworzyć plik — tak. Wklej i uzupełnij:

```json
{
  "apiKey": "WKLEJ-SWOJ-KLUCZ"
}
```

**Klucz nigdy nie trafia do repo ani do okienka.** Leży osobno i tylko u Ciebie —
jest Twoją tożsamością, zadania zakładają się pod Twoim nazwiskiem. W okienku go
nie ma celowo: tamte wartości są wstawiane w treść instrukcji, czyli przewijałyby
się przez każdą rozmowę.

**Reszta.** Napisz `/plugin`, wejdź w **tms-zadania** i uzupełnij cztery pola:

| Pole | Po co |
|---|---|
| Adres TMS | Z paska przeglądarki, bez ukośnika na końcu |
| Reguły doboru pól | Domyślny projekt, kogo ustawiać wykonawcą, czego nie proponować |
| Styl pisania | Jak mają brzmieć zadania — długość opisu, ton, czy używać wyliczeń |
| Proponuj zadania sam | Wyłącz, jeśli Claude ma zakładać zadania tylko na wyraźną prośbę |

Reguły mówią **co** wpisać, styl **jak** to napisać. Oba można zostawić puste.

Masz starszy `tms.json` z adresem i regułami w środku? Nic nie musisz zmieniać —
wtyczka czyta go dalej, gdy okienko jest puste.

### 5. Sprawdzenie

Nowa rozmowa, polecenie: „załóż w TMS zadanie na próbę". Powinien pokazać blok
do zatwierdzenia, a po `tak` — numer i link.

## Więcej

Szczegóły ustaleń: [docs/2026-08-19-spec.md](docs/2026-08-19-spec.md)
