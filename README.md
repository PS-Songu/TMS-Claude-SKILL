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


## Uprawnienia

Claude pyta TMS o Twoje uprawnienia, **zanim** cokolwiek zaproponuje. Jeśli poprosisz
o zadanie w projekcie, do którego nie masz dostępu, powie to od razu — zamiast układać
propozycję, wysyłać ją i pokazywać Ci odmowę serwera.

Widzisz dokładnie tyle, co w przeglądarce: projekty, w których jesteś, ludzi, których
możesz ustawić wykonawcą, i akcje, na które pozwala Twoja rola.

## Zakładanie projektów

Gdy zadanie nie pasuje do niczego, co już jest, możesz kazać Claude'owi założyć nowy
projekt — „załóż projekt Automatyzacja raportów". Pokaże nazwę i opis do zatwierdzenia,
a po Twoim „tak" utworzy go i dopiero w nim założy zadanie.

Właścicielem zostajesz Ty. Jeśli Twoja rola nie pozwala zakładać projektów, Claude
powie to od razu, zamiast próbować.

## Podzadania

Claude widzi checklistę w zadaniu i potrafi ją prowadzić. Przy zakładaniu zadania,
które rozkłada się na kilka wyraźnych kroków, wpisze je jako punkty do odhaczenia
zamiast wyliczenia w opisie — dzięki temu postęp widać na liście zadań.

Potem wystarczy powiedzieć „odhacz drugi punkt" albo zapytać „co jeszcze zostało
w 1766". Gdy przy oddawaniu zadania okaże się, że któryś punkt wisi nieodhaczony,
Claude pokaże które i zapyta — sam ich nie odhacza, żeby oddać zadanie.

Punkt, do którego przypięto inne zadanie, zostaje nietknięty — odhacza się sam,
gdy tamto zadanie zostanie zamknięte.

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
/plugin install tms@jf-tms
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

Wszystko siedzi w jednym pliku: `%USERPROFILE%\.claude\tms.json` (na Macu i Linuksie
`~/.claude/tms.json`). Nie ma go jeszcze? Napisz w rozmowie:

```
/tms:ustawienia
```

Claude utworzy plik z opisami wszystkich pól i przeprowadzi przez uzupełnienie.
Ta sama komenda pokazuje później, co masz ustawione i gdzie plik leży.

Plik wygląda tak — każde pole ma nad sobą wyjaśnienie i przykłady:

```json
{
  // Adres firmowego TMS, bez ukośnika na końcu.
  "baseUrl": "https://tms.firma.pl",

  // Klucz osobisty z TMS: Ustawienia → Klucze → „Wydaj klucz".
  "apiKey": "tms_...",

  // true — Claude sam proponuje zadanie po skończonej robocie
  "propose": true,

  // CO wpisywać: domyślny projekt, kogo ustawiać wykonawcą, czego nie proponować
  "rules": "Domyślny projekt: WMS. Zadania dla siebie chyba że mówię inaczej.",

  // JAK to ma brzmieć: długość opisu, ton, czy używać wyliczeń
  "style": "Krótko, bez ozdobników. Opis maksymalnie trzy zdania."
}
```

`rules` mówi **co** wpisać, `style` **jak** to napisać. Oba można zostawić puste.

**Klucz nigdy nie trafia do repo.** Leży tylko u Ciebie i jest Twoją tożsamością —
zadania zakładają się pod Twoim nazwiskiem. Wtyczka nie wypisuje go w rozmowie,
pokazuje najwyżej cztery ostatnie znaki.

### 5. Sprawdzenie

Nowa rozmowa, polecenie `/tms:ustawienia` — powinno pokazać komplet ustawień
bez ostrzeżeń. Potem „załóż w TMS zadanie na próbę": powinien pokazać blok
do zatwierdzenia, a po `tak` — numer i link.

## Więcej

Szczegóły ustaleń: [docs/2026-08-19-spec.md](docs/2026-08-19-spec.md)
