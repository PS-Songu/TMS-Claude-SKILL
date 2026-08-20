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

Projekt:    WMS
Pula:       Magazyn / błędy
Wykonawca:  Wojtek Kulus
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

Do dopisania, kiedy będzie po stronie TMS ekran do wygenerowania klucza.
Teraz opis byłby wymyślony.

Jedno jest pewne już dziś: **klucz nigdy nie trafia do tego repo.** Leży u każdego
lokalnie, w pliku poza repozytorium.

## Więcej

Szczegóły ustaleń: [docs/2026-08-19-spec.md](docs/2026-08-19-spec.md)
