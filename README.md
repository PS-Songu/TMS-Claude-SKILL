# Skill do Claude — zakładanie zadań w TMS

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

4. Odpowiadasz `tak`, `popraw` (wtedy mówisz co zmienić) albo `anuluj`.
5. Po `tak` zadanie ląduje w TMS, a Ty dostajesz jego numer i link — żeby od razu
   zobaczyć jak wyszło i ewentualnie poprawić na miejscu.

Zadania założone tą drogą mają w TMS własną ikonkę źródła, tak jak te z Discorda —
od razu widać, że przyszły z rozmowy z Claude'em.

Claude nie zgaduje projektów i osób z pamięci — listę pobiera z TMS na bieżąco,
żeby nie rozjechała się z tym, co faktycznie jest w systemie.

## Instalacja

### 1. Klucz

W TMS: Ustawienia → Klucze → „Wydaj klucz". Klucz pokazuje się raz — skopiuj go
od razu. Zgubiony odwołujesz i wydajesz nowy, w tym samym miejscu.

### 2. Wtyczka

W rozmowie z Claude'em, dwie komendy:

```
/plugin marketplace add PS-Songu/TMS-Claude-SKILL
/plugin install tms-zadania@jf-tms
```

Repo jest prywatne, więc GitHub musi Cię znać — trzeba mieć dostęp do
`PS-Songu/TMS-Claude-SKILL` i zalogowane `gh` albo skonfigurowany klucz SSH.
Po instalacji wtyczka jest widoczna w „Manage Plugins", a aktualizacje idą
stamtąd — nie trzeba nic kopiować ręcznie.

### 3. Ustawienia

Klucz i reguły trzymasz u siebie, poza wtyczką. W PowerShellu:

```powershell
notepad "$env:USERPROFILE\.claude\tms.json"
```

Notatnik zapyta, czy utworzyć plik — tak. Wklej i uzupełnij:

```json
{
  "baseUrl": "https://adres-twojego-tms",
  "apiKey": "WKLEJ-SWOJ-KLUCZ",
  "propose": true,
  "rules": ""
}
```

Adres TMS-a weź z paska przeglądarki, bez ukośnika na końcu. `rules` to Twoje
prywatne reguły, prozą: domyślny projekt, kogo zwykle ustawiać wykonawcą, czego
nie proponować. Można zostawić puste. `propose` na `false`, jeśli Claude ma
zakładać zadania wyłącznie na wyraźną prośbę, a nie proponować sam z siebie.

**Klucz nigdy nie trafia do repo.** `tms.json` leży osobno i tylko u Ciebie —
jest tożsamością, zadania zakładają się pod Twoim nazwiskiem.

### 4. Sprawdzenie

Nowa rozmowa, polecenie: „załóż w TMS zadanie na próbę". Powinien pokazać blok
do zatwierdzenia, a po `tak` — numer i link.

## Więcej

Szczegóły ustaleń: [docs/2026-08-19-spec.md](docs/2026-08-19-spec.md)
