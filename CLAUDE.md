# Angličtina hravě

Interaktivní webová aplikace pro učení angličtiny (inspirace Duolingem). Uživatel zadá svou úroveň (A1–B2) ručně nebo úvodním testem, pak prochází krátké lekce. UI je česky, učí se angličtina pro česky mluvící.

## Struktura
- `index.html` – **celá aplikace v jednom souboru** (HTML + CSS + JS). Název musí zůstat `index.html` (GitHub Pages). Žádný build, žádné závislosti, žádné externí požadavky.
- `CLAUDE.md` – tento soubor.

Spuštění: otevřít `index.html` v prohlížeči (nebo `python -m http.server`). Nasazení: GitHub Pages z kořene větve.

## Architektura (uvnitř `index.html`)
Skript má sekce v tomto pořadí:
1. **Obsah** – `RAW` (úrovně → lekce → slovíčka `w` a věty `s`) a `TEST` (úvodní test, 2 otázky na úroveň).
   Formát položky: `"anglicky|alternativa=česky"`. První anglická varianta je hlavní (zobrazuje se), ostatní se přijímají při psaní. U sloves: `"to read|read=číst"`.
2. **Pomocné funkce** – `norm` (normalizace odpovědi: malá písmena, interpunkce, rozbalení zkratek jako I'm → I am), `lev`/`judge` (přesná shoda / překlep s Levenshteinem ≤ 1 u slov od 5 znaků / špatně).
3. **Stav** – `S` uložen v `localStorage` pod klíčem `anglictina-hrave.v1` (`level`, `xp`, `streak`, `lastDay`, `done{unitId:hvězdy}`, `audio`). Při změně struktury `S` zvyš verzi klíče nebo doplň výchozí hodnoty v `defaults()`.
4. **Zvuk** – Web Speech API (`speechSynthesis`), s detekcí podpory (`TTS`). Bez podpory se poslechová cvičení nahradí skládáním vět.
5. **Generátor cvičení** – `buildLesson` vyrobí 13 cvičení: `choose` (výběr), `type` (psaní), `build` (skládání věty), `listen` (poslech), `match` (spojování dvojic). Špatně zodpovězená cvičení se zařadí na konec fronty (kromě `match`).
6. **Zobrazení** – funkce `*HTML()` vracejí řetězce, `render()` přepíše `#app`. Pohledy: `welcome`, `home`, `lesson`, `fail`, `result`, `test`. Modál nastavení je jen nad `home`.
7. **Události** – jedna delegace kliknutí přes `data-act` → objekt `H`. Enter spouští tlačítko `#primary`.

## Pravidla hry
- Lekce: 5 životů (srdíček), chyba = −1. Při 0 se lekce nezdaří. Hvězdy: 3 = bez chyby, 2 = ≤ 2 chyby, jinak 1. XP: 10 (+5 bez chyby).
- Série dní se počítá lokálním datem (`dayKey`).
- Odemykání: první lekce každé úrovně je vždy otevřená, další po dokončení předchozí; úrovně pod zvolenou jsou odemčené celé.

## Konvence
- Vanilla JS, žádné frameworky ani CDN. Vše uživatelské přes `esc()` (ochrana před vložením HTML).
- Barvy a témata přes CSS proměnné v `:root` (včetně tmavého režimu).
- Texty UI česky; identifikátory v kódu anglicky.
- Mobile-first, max šířka 640 px.

## Jak přidat obsah
- Nová lekce: přidej objekt `{t:'Název', i:'emoji', w:[…≥8 slovíček…], s:[…≥4 věty…]}` do `units` dané úrovně v `RAW`.
- Pravidla pro kvalitu dat (dodržuj, jinak vznikají chyby v cvičeních):
  - české překlady slovíček v rámci jedné úrovně musí být **unikátní** (jinak by se v možnostech objevily dva správné překlady);
  - věty piš tak, aby měly jednoznačný překlad; další přijatelné znění přidej přes `|`;
  - zkratky (I'm, don't…) fungují automaticky díky `norm`, nepiš je jako alternativy.
- Nová úroveň (např. C1): přidej do `RAW` a rozšiř `TEST` a pole `scores` ve `starttest` (aktuálně pevně 4 úrovně).

## Kontrola před commitem
1. Otevři stránku, projdi welcome → lekci → výsledek; zkus i test úrovně.
2. Zkontroluj: chybná odpověď ubírá život, opakuje se na konci, 0 životů ukáže obrazovku `fail`.
3. Konzole bez chyb; vyzkoušej mobilní šířku a tmavý režim.

## Nápady do budoucna
Gramatická cvičení (doplňování), slovíčka k opakování podle chyb (spaced repetition), denní cíl, další úrovně (C1), více lekcí, PWA (offline), export/import postupu.
