# Kviz: Voditelj brodice kategorije B

Interaktivni kviz za pripremu hrvatskog ispita "Voditelj brodice kategorije B". Jedna `index.html` datoteka, bez backenda, bez baze, radi i offline (osim Google fontova).

## Sadržaj

- **5 predmeta ispita**, ukupno **335 pitanja** s potpunim odgovorima
- **Vježba sricanja** s međunarodnom tablicom i 50 jadranskih pojmova
- **9 SVG dijagrama** (kompas, ruža vjetrova, navigacijska svjetla, dnevni znakovi, pravila sudara...)
- Skrivene natuknice, više-djelna pitanja, mobilna prilagodba

## Kako otvoriti lokalno

Samo dvostruko klikni na `index.html`. Otvori se u pregledniku, sve radi.

## Kako objaviti na Vercel

Najlakše preko CLI alata (treba Node.js):

```bash
npm i -g vercel
vercel --prod
```

Slijedi upute u terminalu, dobiješ javni link tipa `tvoj-projekt.vercel.app`.

Alternativno: stavi mapu na GitHub i poveži repozitorij s Vercel projektom preko vercel.com.

## Rad na projektu s Claude Code

```bash
# u root mapi projekta
claude
```

Claude Code će automatski pročitati `CLAUDE.md` i dobiti puni kontekst projekta (strukturu, pravila, validacijske testove).

## Tehnička pravila

- Sve u jednoj `index.html` datoteci (CSS + JS + SVG zajedno)
- Vanilla JS, bez vanjskih biblioteka
- Bez `localStorage` / `sessionStorage` (kviz mora raditi u svim okruženjima)
- Natuknice ne smiju sadržavati riječi iz odgovora

Detaljne smjernice i validacijski testovi su u [CLAUDE.md](CLAUDE.md).
