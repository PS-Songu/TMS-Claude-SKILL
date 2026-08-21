## Więcej

Szczegóły ustaleń: [docs/2026-08-19-spec.md](docs/2026-08-19-spec.md)
## Instalacja

### 1. Klucz

W TMS: Ustawienia → Klucze → „Wydaj klucz". Klucz pokazuje się raz — skopiuj go
od razu. Zgubiony odwołujesz i wydajesz nowy, w tym samym miejscu.

### 2. Skill

W PowerShellu, na swoim komputerze:

```powershell
git clone https://github.com/PS-Songu/TMS-Claude-SKILL.git "$env:USERPROFILE\Desktop\TMS-Claude-SKILL"
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills\tms-zadania" | Out-Null
Copy-Item "$env:USERPROFILE\Desktop\TMS-Claude-SKILL\skill\SKILL.md" "$env:USERPROFILE\.claude\skills\tms-zadania\"
```

### 3. Ustawienia

```powershell
Copy-Item "$env:USERPROFILE\Desktop\TMS-Claude-SKILL\skill\tms.example.json" "$env:USERPROFILE\.claude\tms.json"
notepad "$env:USERPROFILE\.claude\tms.json"
```

Wpisz swój klucz i adres TMS-a — ten sam, który masz w pasku przeglądarki, bez
ukośnika na końcu. `rules` to Twoje prywatne reguły, prozą: domyślny projekt,
kogo zwykle ustawiać wykonawcą, czego nie proponować. Można zostawić puste.
`propose` na `false`, jeśli Claude ma zakładać zadania wyłącznie na wyraźną
prośbę, a nie proponować sam z siebie.

**Klucz nigdy nie trafia do tego repo.** `tms.json` leży poza repozytorium i tylko
u Ciebie — jest tożsamością, zadania zakładają się pod Twoim nazwiskiem.

### 4. Sprawdzenie

Nowa rozmowa z Claude'em, polecenie: „załóż w TMS zadanie na próbę". Powinien
pokazać blok do zatwierdzenia, a po `tak` — numer i link.

## Więcej

Szczegóły ustaleń: [docs/2026-08-19-spec.md](docs/2026-08-19-spec.md)
