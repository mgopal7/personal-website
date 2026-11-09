# Lippy

Live lip-combo try-on. Open the camera and Lippy picks 3 liner + lipstick + gloss combos for your lips from independent beauty brands. You see them on your face in real time, turn each layer on and off, swap shades, and lock in pieces to check out at each brand's own Shopify store.

Live at `/lippy/app/` on my site, with a write-up at `/lippy/`. The whole app is `app/index.html`; face tracking lives in `app/mp/`.

## How it works, start to finish

1. **Track the face.** MediaPipe Face Mesh finds 468 points on your face in every video frame, on your phone (nothing is uploaded). Twenty points trace the outer lip line and twenty trace the inner one, so color stays inside your lips and off your teeth.
2. **Read your colors.** For about a second (24 frames), Lippy measures your natural lip color, your skin, and a white reference. Details below.
3. **Pick 3 combos.** Each combo has its own rules. Every shade in the catalog gets a score and the best one wins.
4. **Paint the lips every frame.** Lipstick, liner and gloss are blended in a color space built to match human vision (CIELAB), so your own lip texture, lines and shadows show through.
5. **Lock in and check out.** Only the layers you have turned on go into your collection, grouped by brand, with one checkout per Shopify store.

## Keeping color on the lips

Three things keep the lipstick exactly on your lip line:

1. **Refined lip points.** Face Mesh runs with `refineLandmarks` on, which adds a second model that re-fits the lips (and eyes) more precisely than the base mesh.
2. **Steady, not laggy.** Lip points are smoothed between frames, but the smoothing switches off as soon as your lips move, so color sticks to still lips without trailing behind when you talk.
3. **Edge snapping.** After the first read, Lippy knows your lip color and skin color. Near the edge of the lips it checks each pixel: if it looks like lip, color goes on; if it looks like skin, color stays off. The core of the lips is always painted.

## Reading your colors

| What | Where it comes from | Why |
|---|---|---|
| Natural lip color | Pixels inside the traced lip line | It's your actual lips, not a guess from skin |
| Skin | Small patches on both cheeks | Cheeks are flat and evenly lit; around the mouth is redder and shadowed |
| White reference | The whites of your eyes, plus teeth if you smile | These are close to white, so they reveal the light's tint and the camera's brightness |

- **Ignore shine and shadow.** For lips and skin, the brightest 20% and darkest 20% of pixels are dropped before taking the middle value.
- **Use many frames.** Each value is the median across about 24 frames, so one bad frame (a blink, motion blur) can't skew the read.
- **Correct the lighting.** If the white reference is clearly brighter than your skin and nearly colorless, Lippy trusts it. It removes the light's color tint, and it rescales brightness as if that reference were a standard white, which undoes a camera that over- or under-exposed your face. If the reference can't be trusted (eyes too small, blurry, or squinting, no teeth showing), Lippy skips the correction instead of guessing.
- **Products are drawn in your light too.** Shade colors get the same tint and brightness as your camera, so a nude doesn't look grey in warm light.

### If you're already wearing lip color

Natural lips sit a little darker and pinker than the skin around them, with soft color. Lippy flags lip product when your lips are very saturated, lighter than your skin, an unusual hue, or much darker than your skin. When that happens, it estimates your natural lip color from your skin and repaints your lips to that natural base before trying products. If your lips are just shiny (gloss or balm), it keeps your color and tones down the shine.

## Skin depth: ITA°

Skin depth uses the **Individual Typology Angle (ITA)**, the standard dermatology measure of how light or deep skin is:

`ITA = arctan((L* − 50) / b*) × 180 / π`

| ITA | Band | Bold lipstick depth range (L*) |
|---|---|---|
| > 55° | very light | 32–50 |
| 41–55° | light | 30–48 |
| 28–41° | medium | 27–45 |
| 10–28° | tan | 23–41 |
| −30–10° | brown | 18–36 |
| < −30° | deep | 12–31 |

**Undertone** comes from the hue angle of your skin: warm (more yellow), cool (more pink) or neutral.

## Scoring shades

Each look has a target color and rules. Every shade gets a score, lowest wins:

```
score = color distance to the target (ΔE2000)
      + 2.5 × how far its depth falls outside the allowed range
      + 40  if it's lighter than your skin (reads chalky)
      + undertone penalty (warm skin away from peach and brown, cool skin away from rose and mauve)
      + look-specific checks (below)
```

**ΔE2000 (CIEDE2000)** is the industry standard for how different two colors look to a person. It handles reds and pinks much better than straight-line color distance. Lippy's implementation matches the published reference values (Sharma et al., 2005).

### The 3 looks

| Look | Target | Rules |
|---|---|---|
| **everyday** (your lips, but better) | Your natural lip color, a touch deeper and richer | Depth between 12 steps deeper than your lips and 3 steps lighter, and never lighter than your skin. Saturation stays close to your own lips, so no jump to a bright berry. |
| **90s brown** (the TikTok classic) | A nude about 14 steps deeper than your skin | Only nudes and browns, and it must sit 7–26 steps deeper than your skin. Liner is a brown 12–26 steps deeper than the lipstick. |
| **night out** (the statement) | A bold shade for your undertone: orange-red if warm, blue-red or berry if cool | Depth must fall in your skin band's bold range (table above). |

- **Everyday and 90s are picked together.** Lippy chooses the pair with the best combined score, using two different shades when possible. If the catalog has nothing else that fits your skin, they can share a lipstick, and everyday then uses clear gloss so the two still look different.
- **Liner:** the liner closest in color to the lipstick, 5–20 steps deeper, so the edge is defined, not drawn on.
- **Gloss:** a sheer tint in the lipstick's family, or clear. Milky glosses lighter than your skin are penalized, since they go chalky on deeper skin.

"Steps" are units of lightness (L*) on a 0–100 scale.

## Catalog

2,049 shades from 35 brands sold at Sephora and Ulta: 251 liners, 1,646 lipsticks and 152 glosses. Open the shade picker to filter by brand or search by shade or product name. Matches are listed closest-first.

### Where the shades come from

- **Makeup API dataset** (public copies on GitHub). Brand, product, shade name, hex color, price and product link, collected around 2017–2018.
- **Ovilia/lipstick dataset** (GitHub). Hex colors for lipstick lines from Dior, Chanel, YSL and Givenchy, with series names translated to English.
- **31 checked 2026 shades** from Kylie Cosmetics, Rare Beauty, Tower 28 and Kosas, with prices taken from each brand's site on Oct 29, 2026.
- **2026 gloss lines** (`data/gloss_2026.json`). The datasets had almost no gloss, so I added 12 current gloss and lip oil lines: Dior Addict Lip Maximizer, Too Faced Lip Injection, NYX Butter Gloss, Charlotte Tilbury Collagen Lip Bath, Fenty Gloss Bomb, Laneige Lip Glowy Balm, Clarins Lip Comfort Oil, Summer Fridays Lip Butter Balm, Rhode Peptide Lip Tint, Rare Beauty Stay Vulnerable, e.l.f. Glow Reviver and Maybelline Lifter Gloss. Shade names come from brand and retailer pages (Nov 2026). Their colors are estimated from the shade names and descriptions, and the prices are US list prices.

`data/build_library.py` combines these sources. It keeps only brands sold at Sephora or Ulta and drops hex values that aren't plausible lip colors (for example near-white or neon). It guesses finish and coverage from the product name, removes duplicates, and points links to the brand's own site instead of resellers. Prices in currencies other than USD, GBP or CAD are dropped.

### Shopify tagging

A brand is tagged **shopify store** when its official site runs on Shopify. Those pieces say "check out at" the brand's store, and the rest say "shop at" the brand's site. Tagged so far: ColourPop, Kosas, Kylie Cosmetics, Rare Beauty, Rhode, Tower 28. ColourPop was detected from its product images, which are served from Shopify's CDN.

| Brand | Liners | Lipsticks | Glosses | Shopify |
|---|---|---|---|---|
| NYX Professional Makeup | 163 | 475 | 40 |  |
| Dior | 0 | 208 | 19 |  |
| Clinique | 28 | 181 | 8 |  |
| Smashbox | 15 | 145 | 0 |  |
| YSL Beauty | 0 | 95 | 13 |  |
| Revlon | 0 | 95 | 0 |  |
| ColourPop | 34 | 52 | 0 | yes |
| Maybelline | 4 | 59 | 10 |  |
| Chanel | 0 | 59 | 0 |  |
| Milani | 0 | 46 | 0 |  |
| Givenchy | 0 | 45 | 0 |  |
| L'Oréal Paris | 0 | 44 | 0 |  |
| Fenty Beauty | 0 | 14 | 8 |  |
| IMAN Cosmetics | 0 | 20 | 0 |  |
| e.l.f. Cosmetics | 0 | 13 | 6 |  |
| Burt's Bees | 0 | 18 | 0 |  |
| CARGO | 0 | 16 | 0 |  |
| Rare Beauty | 0 | 11 | 4 | yes |
| Almay | 0 | 12 | 0 |  |
| COVERGIRL | 0 | 11 | 0 |  |
| Benefit Cosmetics | 0 | 10 | 0 |  |
| Tower 28 | 0 | 0 | 8 | yes |
| Too Faced | 0 | 0 | 8 |  |
| Kylie Cosmetics | 7 | 0 | 0 | yes |
| Kosas | 0 | 0 | 5 | yes |
| Charlotte Tilbury | 0 | 0 | 5 |  |
| Laneige | 0 | 0 | 5 |  |
| Summer Fridays | 0 | 0 | 5 |  |
| Glossier | 0 | 5 | 0 |  |
| Mineral Fusion | 0 | 5 | 0 |  |
| Clarins | 0 | 0 | 4 |  |
| Rhode | 0 | 0 | 4 | yes |
| butter LONDON | 0 | 3 | 0 |  |
| wet n wild | 0 | 3 | 0 |  |
| Pacifica | 0 | 1 | 0 |  |

## Vanity

Every look you lock in is saved to your vanity with the pieces kept together: a small picture of your lips wearing it (taken at the moment you lock it in), the products on a little shelf, and the price of what you still need.

Open a look to get it:

- each piece has a **buy** link to the brand's page, and an **I have it** toggle that takes it off your list
- **get the look** groups what's left by store, so you check out once per brand ("check out at" for Shopify stores, "shop at" otherwise), with Sephora and Ulta search links as a backup
- **try it on again** puts the look back on your face

Below the shelf, a **shopping list for every look** combines all your looks, without duplicates or pieces you already have. Looks are saved in your browser (localStorage), so nothing is sent anywhere.

## Known limits and next steps

- **Older dataset shades.** Most dataset shades are from 2017–2018, so some may be discontinued, and hex values come from the dataset rather than measured swatches.
- **Missing brands.** MAC, NARS, Charlotte Tilbury, Huda Beauty, Too Faced, Urban Decay, Tarte, Anastasia, Pat McGrath, Makeup by Mario, Rhode and Summer Fridays aren't in the public data yet.
- **Shade colors are estimates**, not measured swatches. Next: pull swatch images and variant IDs from Shopify's Global Catalog and measure each shade's color from the swatch.
- **Deep-skin range is thin.** Few deep nudes in the data, so on deep skin everyday and 90s can share a lipstick. Adding the missing brands would fix this.
- **Camera exposure.** Phones auto-brighten faces. The white-reference correction helps, but only when it finds clean eye whites or teeth. A smile helps.
- **Rules are hand-tuned.** Next: thumbs up or down on each combo, then tune the weights from that feedback for each skin band.
- **Checkout** links go to product pages until variant IDs are in. Then they become Shopify cart permalinks, one per brand.

## Put it online

The live camera needs https. Any static host works; this folder deploys as-is on Vercel or Netlify. The two data files in `mp/` end in `.wasm` on purpose so every host serves them.


## Matching and color accuracy (Nov 2026 update)

- **Liners match the lipstick.** Every combo's liner is picked from the lipstick's own color family (within about 10° of hue), a little deeper and similar in saturation. The 90s brown liner leans brown but stays close to the lipstick's family.
- **Swapping re-matches the liner.** When you swap the lipstick, the liner is re-picked to go with it. You can turn this off under the shade picker.
- **Glosses match too.** A tinted gloss has to sit in the lipstick's color family; clear gloss always works.
- **Checked shades first.** About 80% of shade colors come from a 2017–18 public dataset of web swatches. Those are labeled "approx. color" in the picker, and recommendations favor shades checked on brand and retailer pages in 2026.
- **Finish-aware rendering.** Mattes and satins cover more of your natural lip, sheers and glossy lipsticks let more show through. Intensity starts at 100%.
