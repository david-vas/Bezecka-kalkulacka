# CLAUDE.md — Běžecká kalkulačka

Kontext o repu a uživateli pro Claude Code. Drž tohle v hlavě při každém promptu.

---

## 🏗️ Co to je

**Běžecká kalkulačka #TýmDejvid** — samostatná PWA pro běžce: tempo, cílový čas,
mezičasy, závodní predikce, tréninkové zóny, triatlon, kadence, velikost bot,
odpočet do závodu. Prodává se za **99 Kč** na davidvas.cz.

**Živá adresa:** `https://david-vas.github.io/Bezecka-kalkulacka/` (GitHub Pages, větev `main`)
⚠️ URL je **case-sensitive** — velké `B` a `k`. `bezecka-kalkulacka` nefunguje.

**Prodejní stránka:** `davidvas.cz/sluzby/bezecka-kalkulacka#objednat`
(zdroj je v jiném repu: `tymdejvid-app` → `web/sluzby-bezecka-kalkulacka.html`)

```
/                    ← root, žádné podsložky
├── index.html       ← CELÁ appka, 3 700 řádků (React uvnitř <script type="text/babel">)
├── landing.html     ← prodejní/představovací stránka appky
├── navod.html       ← návod „přidej si to na plochu telefonu"
├── manifest.json    ← PWA manifest (start_url: /Bezecka-kalkulacka/)
├── sw.js            ← service worker (offline režim)
└── icon.png
```

## ⚙️ Stack — vanilla, bez buildu

- **React 18 + ReactDOM + Babel standalone z CDN** (cdnjs), `html2canvas` na sdílení obrázku
- **JSX se kompiluje až v prohlížeči** (`<script type="text/babel">`, `index.html:309`)
- **Žádný npm, žádný bundler, žádný `package.json`** — a nezavádět je
- Fonty: Barlow + Barlow Condensed (Google Fonts)
- **Neděl z toho víc souborů** bez Davidova souhlasu — jedno-souborová appka je
  schválně, kvůli GitHub Pages a jednoduchosti.

## 🚀 Nasazení = `git push` na `main`

Není žádný deploy skript ani preview kanál. **Push na `main` = do minuty je to živé.**
Zvlášť opatrně: tohle je placený produkt, kterým lidem přijde v mailu odkaz.

**Po každé změně souborů:**
1. Když **přidáváš/přejmenováváš soubor**, přidej ho do `APP_SHELL` v `sw.js`
   **a bumpni `VERSION`** (`tymdejvid-v1` → `v2`). Bez toho lidem visí stará
   verze z cache a vypadá to, že oprava nefunguje.
2. Ověř v prohlížeči, ne jen v kódu — Babel chyby se projeví až za běhu
   (bílá stránka + chyba v konzoli, nic se nezkompiluje dopředu).

## 🧮 Záložky (TABS, `index.html:3505`)

| Zdarma | Pro (zamčené 🔒) |
|---|---|
| 👤 Profil · 🎯 Cílový čas · 🔄 Přepočet | ⏱ Mezičasy · 🏆 Závody · 🏊 Triatlon · 📊 Zóny TP · ⛰️ Terén · 👟 Kadence · 👞 Velikost · ⏳ Odpočet |

Seznam zamčených je `PRO_TABS` (`index.html:3429`).

## 🔐 Licenční zámek — ŽIJE VE DVOU KOPIÍCH

Kód se **počítá z e-mailu kupujícího** (deterministicky, žádná databáze).
Stejný algoritmus musí být na obou stranách:

| Kde | Co dělá |
|---|---|
| `index.html` → `calcLicenseCode()` + `CALC_SECRET` (ř. 3430) | ověří kód, který člověk zadá |
| repo `tymdejvid-app` → `functions/index.js:33621` `_calcLicenseCode()` | vygeneruje kód do e-mailu po zaplacení |

🚨 **`CALC_SECRET` nikdy neměň.** Změna = všem dosavadním zákazníkům přestane
fungovat jejich kód. Když opravdu musíš, změň to **v obou souborech naráz**
a rozešli nové kódy (v appce `emaily.html` je na to šablona `kalkulacka_access`).

⚠️ **Je to vědomě MĚKKÝ zámek.** Appka je veřejná na GitHub Pages, tajný klíč
je v ní vidět. Zastaví běžného člověka, ne experta — a u produktu za 99 Kč to
tak stačí. **Nedávej za tenhle zámek nic, co by opravdu nemělo uniknout.**
(Platí zlaté pravidlo z hlavního repu: skutečný zámek je jen na serveru.)

## 💾 Data — všechno v prohlížeči, žádný server

Appka **nemá backend ani přihlášení**. Vše je v `localStorage`:

| Klíč | Co drží |
|---|---|
| `tymdejvid_profile` | Davidův/uživatelův profil (OR na 5/10/21/42 km, výška, noha, terén) |
| `tymdejvid_athletes` + `tymdejvid_active_athlete` | uložení svěřenci a kdo je zrovna vybraný |
| `tymdejvid_countdowns` | odpočty do závodů a událostí |
| `tymdejvid_calc_license` | odemykací kód |
| `tymdejvid_url_athlete` | dočasný běžec načtený z odkazu (viz níž) |

**Důsledek:** vyčištění prohlížeče = ztráta dat. Když přidáváš cokoliv, co si
uživatel naklikal, ulož to sem — a **vždy v `try/catch`** (v anonymním okně
`localStorage` hází chybu).

## 🔗 Sdílení a vkládání

- **Odkaz s tempy:** `?name=Petr&pmar=4:15&phalf=4:05&p10k=3:50&p5k=3:40&tab=zones`
  (`buildAthShareUrl`, ř. 356) — David tak pošle svěřenci kalkulačku už nastavenou
  na jeho tempa. **URL parametry přebijí všechno ostatní**, i uloženého běžce.
- **Embed:** iframe 390×700 na davidvas.cz (`EmbedModal`, ř. 2548).
- **Obrázek na Instagram:** `html2canvas` (`ShareModal`, ř. 2473).

Když měníš pole profilu, projdi i tyhle tři cesty — jinak se sdílený odkaz rozejde.

## 🪤 Pasti (reálné, ne teoretické)

1. **Stará kopie kalkulačky žije v hlavní appce.** `tymdejvid-app/app/kalkulacka.html`
   je zaseklá verze se **třemi** záložkami (Přepočet, Závody, Mezičasy) a běží na
   `tymdejvid-app.web.app/kalkulacka.html`, odkazuje se na ni admin. **Když opravíš
   výpočet tady, tam zůstane starý.** Než začneš, zeptej se Davida, jestli má
   ta kopie zůstat, nebo z ní udělat jen odkaz sem.
2. **Odpočet je naprogramovaný dvakrát** — jednou uvnitř `ProfilTab` (ř. ~2118)
   a jednou jako samostatný `CountdownTab` (ř. 2802). Obě píšou do stejného klíče
   `tymdejvid_countdowns`. **Měníš-li chování odpočtu, oprav obě.**
3. **`index.html` má 3 700 řádků v jednom souboru.** Než něco přidáš, `grep`
   si název funkce — hodně věcí už existuje (`fmtPace`, `toHMS`, `riegel`,
   `paceToSec`). Nepiš je znovu.
4. **`toHMS` existuje dvakrát** (globálně ř. 321 a lokálně v `ProfilTab` ř. 2158)
   — pozor, kterou zrovna voláš.

## 📐 Vzorce (ať je nepřepisuješ z hlavy)

- **Riegel** (predikce z jednoho výkonu): `t2 = t1 × (d2/d1)^1.06` — ř. 329
- **Karvonen** (tepové zóny z klidového a maximálního tepu) — ř. 419
- **GAP** (přepočet tempa podle sklonu) — ř. 426
- **Yasso 800** (odhad maratonu z 10× 800 m) — ř. 436
- **Běžící pás ↔ silnice** podle sklonu — ř. 452
- **Korekce na teplo a vlhkost** — ř. 466

Když měníš číslo ve vzorci, **napiš k tomu do commitu proč** a ověř na Davidových
vlastních číslech (maraton 2:23:03, půlmaraton 1:07:11) — musí vyjít realisticky.

---

## 🎨 Vzhled

- **Brand barva = zelená.** Tmavý režim `#4cff00`, světlý `#1e7d00` (`--accent`).
- **Appka má tmavý I světlý režim** (`body.light`). 🚨 **Každou novou barvu textu
  přidej i do `body.light` bloku** (`index.html:47+`) — jinak bude ve světlém
  režimu bílý text na bílém pozadí. To je nejčastější chyba v tomhle repu.
- **Mobil je hlavní zařízení** — je to PWA na plochu telefonu. Testuj na **375 px
  i 320 px**, nic nesmí ujíždět do strany.
- Brand jméno: `#tymdejvid` v textu, `#TYMDEJVID` jen v logu/nadpisu.

## 🧑‍🏫 Komunikace s Davidem (KRITICKÉ)

**David není programátor.** Platí to samé co v hlavním repu `tymdejvid-app`:

1. **Krátce.** Odpověď na běžnou otázku = pár vět. Ne tabulky, ne tři nadpisy.
2. **Vždy krátký příklad.** „Zadáš cíl 40:00 na desítku → ukáže ti 4:00/km."
3. **Česky, tykáním, bez omáček**, emoji OK (✅ 🔴 ⚠️ 🚀 🏃).
4. **Po změně napiš 2–4 odrážky, co se prakticky stalo** — žádné názvy funkcí,
   žádné technické termíny.
5. **Ukazuj screenshot**, když měníš vzhled — David si to z popisu nepředstaví.
6. **Nikdy nepushuj bez jeho „ano"** — push je tady rovnou nasazení na produkci.

**Auto-přepočet tempa (povinné):** kdekoliv je čas + vzdálenost, vždy vedle ukaž
`min:sec/km`. To je celý smysl téhle appky.

**Skloňuj jména do 5. pádu:** „Ahoj Davide", ne „Ahoj David".

## ✅ Než něco označíš za hotové

- [ ] Otevřel jsem appku v prohlížeči a proklikal změněnou záložku
- [ ] Zkusil jsem **tmavý i světlý** režim
- [ ] Zkusil jsem **320 px** šířku, nic neujíždí do strany
- [ ] Když jsem sahal na soubory → bumpnutá `VERSION` v `sw.js`
- [ ] Když jsem sahal na licenci → změna i v `tymdejvid-app/functions/index.js`
- [ ] Konzole prohlížeče je čistá (Babel chyby se projeví až tady)

---

*Založeno srpen 2026 podle reálného auditu kódu.*
