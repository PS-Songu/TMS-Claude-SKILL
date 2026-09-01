# Praca w tym repozytorium

## Wersja — cztery miejsca, jednym ruchem

Numer wersji siedzi w czterech miejscach. **Podbijasz wszystkie naraz, w jednym
commicie:**

1. `.claude-plugin/plugin.json`
2. `.claude-plugin/marketplace.json`
3. `skills/ustawienia/SKILL.md` — sekcja „Wersja" i przykładowy blok pod nią
4. wydanie na GitHubie: `gh release create vX.Y.Z --target main`

Punkt 3 to numer, który wtyczka melduje człowiekowi. Rozjazd z resztą znaczy, że
`/tms:ustawienia` kłamie o tym, co jest wczytane — a to jedyne miejsce, z którego
da się to sprawdzić.

Zanim wypchniesz podbicie, sprawdź, że nic nie zostało:

```bash
grep -rn "0\.31\.2" --include=*.md --include=*.json .
```

## Numeru wydanego nie nadpisujesz

**Cache wtyczek jest kluczowany numerem wersji.** Katalog `~/.claude/plugins/cache/
jf-tms/tms/<wersja>/` raz pobrany nie odświeża się już nigdy — odświeżanie źródła
widzi ten numer i uznaje, że ma swoje.

Wniosek: **cokolwiek dopchniesz pod numerem, który komuś już się pobrał, do tej
osoby nie dotrze.** Poprawka do wydanej wersji to zawsze nowy numer, nawet gdy
zmieniasz jedną linijkę. Zdarzyło się naprawdę: 0.31.2 poszło z poprawką numeru
wysłaną po fakcie i u nikogo się nie pojawiło — trzeba było wydać 0.31.3.

## Zadanie w TMS po scaleniu

Każde wydanie dostaje zadanie w projekcie „TMS ✅", pula „🤖 SKILL-Claude", nazwane
`Wtyczka Claude'a X.Y.Z — <co się zmieniło>`. Opis prozą, bez nazw plików; link do
PR-a w opisie albo w materiałach.

Jedno wydanie = jedno zadanie. Gdy dwa PR-y wyjdą pod tym samym numerem, podbij
drugi, zamiast wpisywać oba do jednego zadania.

## Czego nie robisz

Nie zmieniasz zachowania wtyczki przy okazji podbijania wersji — wydanie porządkowe
zostaje porządkowe.
