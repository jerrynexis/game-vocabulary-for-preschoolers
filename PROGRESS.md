# SLOVÍČKA — Progress

## Aktuální stav
Větev: main | Fáze: Příprava modulární struktury

## Plán migrace

### Fáze 1 — CSS
- [ ] Extrahovat CSS → css/style.css
- [ ] Ověřit vizuální shodu

### Fáze 2 — JavaScript
- [ ] js/app.js — boot, router, showScreen()
- [ ] js/audio.js — TTS, Web Audio API
- [ ] js/progression.js — levely, hvězdičky, unlock
- [ ] js/game.js — herní logika, výběr slov
- [ ] js/ui.js — renderování obrazovek
- [ ] js/profiles.js — profily, nastavení, storage
- [ ] js/favorites.js — oblíbené sady
- [ ] js/admin.js — admin panel, report log

### Fáze 3 — Data
- [ ] data/sady/*.json (50 souborů)
- [ ] data/manifest.json

### Fáze 4 — Obrázky
- [ ] img/nazev-sady/*.svg (1185 souborů)

### Fáze 5 — Optimalizace
- [ ] Lazy loading obrázků
- [ ] Ověřit na GitHub Pages

## Záznamy ze sessions
### Session 1 — [datum]
- Připravena struktura repozitáře
- Vytvořeny CLAUDE.md a PROGRESS.md

## Rozhodnutí
- JS: klasické skripty (ne ES moduly)
- Zdroj pravdy: single-file/index.html