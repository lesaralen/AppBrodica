# Kviz: Voditelj brodice kategorije B

## O projektu

Interaktivni kviz za pripremu ispita "Voditelj brodice kategorije B" prema hrvatskom programu (Lučka kapetanija / HRB). Sve je jedna samostalna `index.html` datoteka bez backenda, baze ili build koraka. Hostana je na Vercelu kao statična stranica.

## Struktura datoteka

- `index.html` - cijeli kviz (HTML + CSS + JS + SVG dijagrami u jednoj datoteci, ~110 KB)
- `vercel.json` - konfiguracija za Vercel (clean URLs)
- `CLAUDE.md` - ova datoteka, kontekst za Claude Code
- `README.md` - opis projekta i upute za deploy
- `.gitignore` - standardni Node ignore

## Sadržaj kviza

Pet predmeta + jedna vježba sricanja, ukupno 335 pitanja i 50 pojmova za sricanje.

| Cjelina | Naslov | Pitanja |
|---------|--------|---------|
| 1 | Pomorska plovidba (navigacija) | 82 |
| 2 | Motoristika i zaštitne mjere | 41 |
| 3 | Pomorstvo, propisi i meteorologija | 64 |
| 4 | Manevriranje, sigurnost i prva pomoć | 79 |
| 5 | Pomorska radiotelefonska služba | 69 |
| 6 | Sricanje (vježba) | 50 pojmova |

## Tehnička arhitektura

### Glavni JS objekti (svi u jednoj datoteci, unutar `<script>` taga na dnu)

- `SVG` - objekt s 9 SVG dijagrama (kompas, ruža vjetrova, sektori svjetala, dnevni znakovi, sudar, kardinalni sustav, karta, taktovi motora, područja plovidbe)
- `SECTIONS` - polje od 5 cjelina, svaka s `questions: [...]`
- `SOLS` - mapa potpunih točnih odgovora po `[sectionIndex][questionIndex]` (potpune rečenice za prikaz)
- `SPELL_ALPHA` - međunarodna tablica sricanja A-Z
- `SPELL_NUM` - brojevi 0-9 s `{full, simple}` oblicima (IMO složenice + jednostavni engleski)
- `SPELL_TERMS` - 50 jadranskih pojmova s tipom (otok, luka, uvala, svjetionik...)

### Struktura pitanja

```js
{
  q: "Tekst pitanja?",
  a: ["varijanta1", "varijanta2"],   // za jednostavna pitanja
  all: true,                          // za višedjelna pitanja
  req: [["dio1var1","dio1var2"], ["dio2var1"]],  // sve grupe moraju biti pokrivene
  hint: "Skrivena natuknica koja NE smije sadržavati riječi iz odgovora",
  textarea: true,                     // veliko polje za unos
  diagram: "kompas",                  // ključ u SVG objektu
  cap: "Opis dijagrama"
}
```

Pitanja s `all: true` zahtijevaju da odgovor pokrije SVE grupe iz `req` (a ne samo jednu varijantu).

### Logika provjere odgovora (`matchGroup`)

Tekst odgovora se normalizira: lowercase, dijakritika → latinica (č→c, š→s itd), interpunkcija → razmak. Zatim se traži da odgovor sadrži barem jednu varijantu iz grupe. Za grupe s `all: true`, sve grupe moraju proći.

### Glavne funkcije

- `buildMenu()` - gradi početni izbornik s 6 kartica
- `openSection(si)` - renderira jednu cjelinu (predmete 1-5)
- `openSpelling()` - renderira vježbu sricanja (cjelina 6, drugačiji UI)
- `checkAnswer(gid)` - provjerava upisani odgovor
- `showAnswer(gid)` - prikazuje točan odgovor iz `SOLS`
- `toggleHint(gid)` - pokazuje/skriva natuknicu
- `spellWord(word)` - vraća sricanje pojma (npr. "SPLIT" → "Sierra - Papa - Lima - India - Tango")
- `normalize(s)` / `matchGroup(nv, variants)` - normalizacija i poklapanje
- `goMenu()` / `resetSection()` - navigacija
- `updateBar()` / `showFinal()` - traka napretka i rezultat

### CSS pristup

CSS varijable za boje (`--navy`, `--sea`, `--gold`, `--coral` itd) na vrhu. Tri responzivna prijeloma:
- `@media(max-width:768px)` - tablet
- `@media(max-width:560px)` - mobitel (input 16px da spriječi iOS zoom, gumbi puna širina, dijagrami min-height 200px)
- `@media(max-width:380px)` - mali telefoni
- `@media(hover:none)` - touch uređaji bez hover efekata

## Stanje aplikacije

- **State u memoriji, NIKAKAV browser storage**: `localStorage` i `sessionStorage` se NE SMIJU koristiti (kviz se izvodi i u okruženjima gdje ti API-ji nisu dostupni).
- `progress[si] = {gid: 'correct'|'wrong'|'shown'}` - praćenje odgovora po cjelini.
- `currentSec` - aktivna cjelina (broj 0-4 za predmete, string `"spell"` za vježbu sricanja).

## Pravila pri uređivanju

1. **Sve u jednoj datoteci.** Ne uvoditi build sustave (Vite, Webpack), bundlere, preprocessore. Sve ostaje u `index.html`.
2. **Bez vanjskih JS biblioteka.** Vanilla JS. Jedini vanjski resurs su Google fontovi preko `@import` u CSS-u.
3. **Bez browser storage API-ja.** Ne `localStorage`, ne `sessionStorage`, ne `IndexedDB`, ne cookies.
4. **Natuknice (hint) NE smiju sadržavati riječi iz odgovora.** Provjeri prije izmjene: naprovjeri prolazi li ovaj test (Node):
   ```js
   // Pseudokod testa: za svaki q, normaliziraj hint, normaliziraj sve varijante iz q.a (ili q.req.flat()),
   // i provjeri da NIJEDNA varijanta duljine >=4 nije podstring natuknice.
   ```
5. **Pitanja s više obaveznih dijelova koriste `all: true` + `req: [[group1...], [group2...]]`.** Ne stavljaj sve varijante u `a:` ako ih treba sve pokriti, jer bi tada jedna sama prošla kao točno.
6. **Hrvatski jezik s pravom dijakritikom** (č, ć, š, ž, đ) u tekstovima.
7. **Cijeli kviz radi i bez interneta** (osim Google fontova). Ako se ti fontovi ne učitaju, fallback je Georgia/serif.

## Validacija (pokreni nakon izmjena)

```bash
# 1. JS sintaksa
python3 -c "import re; t=open('index.html').read(); m=re.search(r'<script>(.*)</script>', t, re.S); open('/tmp/qjs.js','w').write(m.group(1))"
node --check /tmp/qjs.js

# 2. Broj pitanja po cjelini (treba biti 82, 41, 64, 79, 69)
node -e '
const fs=require("fs");
let t=fs.readFileSync("index.html","utf8");
let s=t.indexOf("const SECTIONS=[");
let e=t.indexOf("/* ---------- POTPUNI");
let SECTIONS=eval("("+t.slice(s+15,e).trim().replace(/;\s*$/,"")+")");
SECTIONS.forEach(x=>console.log(x.num,x.title,"->",x.questions.length));
'

# 3. Provjera curenja natuknica
node -e '
const fs=require("fs");
let t=fs.readFileSync("index.html","utf8");
let s=t.indexOf("const SECTIONS=[");
let e=t.indexOf("/* ---------- POTPUNI");
let SECTIONS=eval("("+t.slice(s+15,e).trim().replace(/;\s*$/,"")+")");
function norm(str){return str.toLowerCase().replace(/[\u010d\u0107]/g,"c").replace(/\u0161/g,"s").replace(/\u017e/g,"z").replace(/\u0111/g,"dj");}
let leaks=0;
SECTIONS.forEach((sec,si)=>sec.questions.forEach((q,qi)=>{
  if(!q.hint)return;
  let hn=norm(q.hint);
  let answers=q.all?q.req.flat():q.a;
  answers.forEach(a=>{ if(norm(a).length>=4 && hn.includes(norm(a))){ leaks++; console.log("LEAK S"+(si+1)+"P"+(qi+1)+": "+a); } });
}));
console.log("Leaks:",leaks);
'

# 4. Test sricanja
node -e '
const fs=require("fs");
let t=fs.readFileSync("index.html","utf8");
let js=t.slice(t.indexOf("// Medunarodna tablica"), t.indexOf("function openSpelling"));
eval(js);
[["SPLIT","Sierra - Papa - Lima - India - Tango"],["BRAČ","Bravo - Romeo - Alfa - Charlie"]].forEach(([w,e])=>{
  console.log(w,"->",spellWord(w),"|",spellWord(w)===e?"OK":"FAIL");
});
'
```

## Deploy

Promijenjen kod se objavi na Vercel jednom od ovih načina:

```bash
# Ako je projekt povezan s Gitom:
git add . && git commit -m "opis izmjene" && git push

# Ili direktno preko Vercel CLI:
vercel --prod
```

## Što je već riješeno

- 335 pitanja kroz 5 predmeta, svako s potpunim cjelovitim odgovorom u `SOLS`
- 56 višedjelnih pitanja koja zahtijevaju sve dijelove
- 9 SVG dijagrama umetnutih u pripadajuća pitanja
- Skrivene natuknice (gumb Pomoć) bez curenja odgovora
- Vježba sricanja s međunarodnom tablicom (slova + dijakritika + brojevi) i 50 jadranskih pojmova
- Mobilna prilagodba (tablet/mobitel/mali mobitel breakpoints, iOS zoom fix, touch hover removal)
- Spremno za Vercel (postoji `index.html` i `vercel.json`)

## Što bi se još moglo raditi (ideje)

- Ispitna simulacija: nasumičnih 30 pitanja s vremenskim ograničenjem
- Spremanje rezultata po cjelini (ali BEZ localStorage - eventualno preko URL state ili kao izvoz/uvoz JSON-a)
- Više pojmova za sricanje (trenutno 50, lako proširivo u `SPELL_TERMS`)
- Detaljnije slike navigacijskih svjetala (trenutno su shematski u tekstu pitanja)
- PWA manifest da se može instalirati na telefon kao aplikacija
- Dark mode toggle
