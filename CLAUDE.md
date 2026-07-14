# SLOVÍČKA — Claude Code Kontext

## Co je tento projekt
Česká dětská výuková webová aplikace pro děti 3–6 let.
Dítě slyší slovo česky (TTS) a vybere odpovídající obrázek z nabídky karet.
Primárně pro tablet a desktop.

## Aktuální stav repozitáře

### Větev `single-file`
- Kompletní produkční verze jako jediný `index.html` (1,1 MB)
- 50 sad slov, 1185 slov, 1185 SVG obrázků (base64 inline)
- Běží na GitHub Pages jako veřejná aplikace

### Větev `main` (tato větev — práce v průběhu)
- Cíl: modulární verze rozdělená do více souborů
- Zdrojem pravdy je `single-file/index.html`

## Cílová struktura `main`
```
slovicka/
├── index.html              ← minimální shell
├── css/
│   └── style.css
├── js/
│   ├── app.js              ← boot, router, screen navigation
│   ├── audio.js            ← TTS (cs-CZ) + Web Audio API efekty
│   ├── progression.js      ← level 0–4, hvězdičky, unlock sad
│   ├── game.js             ← herní logika, výběr slov, distractors
│   ├── ui.js               ← renderování všech 10 obrazovek
│   ├── profiles.js         ← profily hráčů, nastavení, localStorage
│   ├── favorites.js        ← oblíbené sady (3 sloty)
│   └── admin.js            ← admin panel (heslo 8888), report log, revize
├── data/
│   ├── manifest.json       ← seznam všech 51 sad
│   └── sady/
│       └── *.json          ← 50 JSON souborů (1 per sada)
└── img/
    └── nazev-sady/
        └── *.svg           ← SVG obrázky jako samostatné soubory
```

## Architektura aplikace

### Obrazovky (10)
0. Splash — logo + načítání
1. Výběr profilu — avatary (barva + iniciály), prázdný stav
2. Editace profilu — jméno, barva avataru, smazání
3. Hlavní menu — sady po 6 kategoriích, progress, odemykání
4. Výběr sad — checkboxy, oblíbené záložky (3 sloty)
4b. Pre-game splash — přehled vybraných sad
5. Herní obrazovka — slovo + TTS + mřížka karet s obrázky
6. Výsledky kola — hvězdičky, naučená/procvičit slova
7. Achievement modal — při splnění sady
8. Nastavení — 8 parametrů per profil
9. Admin panel — přístup heslem 8888

### Klíčové datové struktury

**Profil hráče** (localStorage `slovicka_v1`):
```js
{
  id, name, color,
  settings: { cards, questions, scoreDisplay, sfx, autoRead, readSpeed, pregameDuration, wrongDelay },
  progression: { [setId]: { [wordId]: { seen, correct, wrong, streak, level, lastSeenDate } } },
  lastStars: { [setId]: 0–3 },        // výsledek posledního kola
  unlockedSets: ['set-id', ...],       // trvale odemčené sady
  achievements: { [setId]: true },     // splněné sady (level≥3 všech slov)
  favorites: [['setId',...], null, null] // 3 oblíbené kombinace
}
```

**Slovo v sadě**:
```js
{ id: 'krava', word: 'KRÁVA', speakText: 'kráva', image: 'img/zvirata-farma/krava.svg', difficulty: 1 }
```

**JSON sada** (`data/sady/zvirata-farma.json`):
```json
{
  "id": "zvirata-farma",
  "nazev": "Zvířata na statku",
  "kategorie": "ziva-priroda",
  "ikona": "🐄",
  "slova": [{ "id": "krava", "slovo": "kráva", "obtiznost": 1 }, ...]
}
```

### Progression systém (Hybrid E — Leitner)
- Level 0–4 per slovo per hráč
- Váhové losování: level 0→váha 20, 1→15, 2→8, 3→3, 4→1
- Max 2–3 nová slova za kolo (level 0, nikdy neviděná)
- Fáze sezení: Seznámení → Procvičování → Opakování chyb
- Časové blédnutí: level -1 po 7 dnech bez hraní
- Anti-repeat: žádné stejné slovo dvakrát za sebou

### Odemykání sad
- První sada každé kategorie: vždy odemčena
- Další sada: odemkne se při prvním dosažení ★★★ (≥90%) na předchozí
- Odemčení je permanentní → uloženo v `profile.unlockedSets`
- `grantUnlock(profile, setId)` zapíše a uloží

### Hvězdičky (výsledek posledního kola)
- ★ ≥ 30% správně
- ★★ ≥ 60% správně  
- ★★★ ≥ 90% správně
- Uloženo jako `profile.lastStars[setId]` (0–3, null = nikdy nehráno)

### Kategorie sad (6)
```js
'ziva-priroda'    // 12 sad — Živá příroda
'jidlo'           // 8 sad  — Jídlo a pití
'clovek-domacnost'// 9 sad  — Člověk a domácnost
'svet-kolem-nas'  // 13 sad — Svět kolem nás
'abstraktni'      // 6 sad  — Abstraktní pojmy
'fantazie'        // 2 sady — Fantazie a pohádky
```

### localStorage klíče
- `slovicka_v1` — profily a veškerá progression data
- `slovicka_report_log` — log nahlášených slov
- `slovicka_review_v1` — stav rozpracované revize slov (admin)

### TTS (Web Speech API)
- Jazyk: `cs-CZ`, priorita: Google → Microsoft → fallback
- `speakText` pole obsahuje slovo s diakritikou pro správnou výslovnost
- Rychlost: slow=0.65, normal=0.82, fast=1.1

## Konvence a rozhodnutí

### Co NESDÍLET s uživatelem
- Admin panel (heslo 8888) — vývojářský nástroj
- Report log a revize slov

### CSS proměnné (hlavní paleta)
```css
--cream: #F0EDE6;  --green-dark: #3D6B52;  --green-mid: #5A8C6E;
--gold: #E8A83A;   --red: #D95C4A;         --text-dark: #2C3A30;
```

### Hover pouze na desktop
```css
@media (hover: hover) and (pointer: fine) { ... }
```
iOS Safari zachovává hover po tapnutí — proto je hover zabalený v media query.

### Ghost karty pro sudé gridy
3 karty → 1 ghost (grid 2×2), 5 → 1 ghost (3×2), 7 → 2 ghost (3×3), 8 → 1 ghost

### Výchozí nastavení nového hráče
cards: 6, questions: 10, scoreDisplay: 'end', sfx: true,
autoRead: true, readSpeed: 'normal', pregameDuration: 'medium', wrongDelay: 'medium'

## Migrace: single-file → modulární

### Postup (doporučené pořadí)
1. Extrahovat CSS → `css/style.css`
2. Extrahovat JS sekce → jednotlivé soubory v `js/`
3. Extrahovat JSON data sad → `data/sady/*.json`
4. Extrahovat SVG z base64 → `img/nazev-sady/*.svg`
5. Upravit `image:` pole v JSON z base64 na relativní cestu
6. Přidat lazy loading obrázků
7. Ověřit funkčnost na GitHub Pages

### Priorita při konfliktech
Vždy preferovat funkčnost z `single-file/index.html` jako zdrojovou pravdu.
Pokud si nejsi jistý chováním, přečti příslušnou funkci tam.

## Časté dotazy pro Claude Code

**Kde je definice herní obrazovky?**
Hledej `function renderQuestion()` a `function handleCardClick()` v single-file.

**Jak funguje výběr slov?**
Viz `function buildRoundWordList()` a `function weightedSample()`.

**Jak se renderuje hlavní menu?**
Viz `function renderMenu()` — prochází kategorie, volá `isSetUnlocked()`.

**Kde jsou uloženy barvy kategorií?**
V objektu `CATEGORIES` v sekci DATA.
