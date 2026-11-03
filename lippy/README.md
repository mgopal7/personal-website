# Lippy

Live lip-combo try-on. Open the camera and Lippy picks 3 liner + lipstick + gloss combos for your lips from independent beauty brands. You see them on your face in real time, turn each layer on and off, swap shades, and lock in pieces to check out at each brand's own Shopify store.

Live at `/lippy/` on my site. The whole app is `index.html`; face tracking lives in `mp/`.

## How it works, start to finish

1. **Track the face.** MediaPipe Face Mesh finds 468 points on your face in every video frame, on your phone (nothing is uploaded). Twenty points trace the outer lip line and twenty trace the inner one, so color stays inside your lips and off your teeth.
2. **Read your colors.** For about a second (24 frames), Lippy measures your natural lip color, your skin, and a white reference. Details below.
3. **Pick 3 combos.** Each combo has its own rules. Every shade in the catalog gets a score and the best one wins.
4. **Paint the lips every frame.** Lipstick, liner and gloss are blended in a color space built to match human vision (CIELAB), so your own lip texture, lines and shadows show through.
5. **Lock in and check out.** Only the layers you have turned on go into your collection, grouped by brand, with one checkout per Shopify store.

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

31 real shades from 4 brands that run on Shopify: Kylie Cosmetics liners, Rare Beauty Kind Words lipsticks, Tower 28 ShineOn Lip Jelly, and Kosas Wet Lip Oil Gloss. Prices are from each brand's site, Oct 29, 2026.

## Known limits and next steps

- **Shade colors are estimates** from each brand's own shade description, not measured swatches. Next: pull swatch images and variant IDs from Shopify's Global Catalog and measure each shade's color from the swatch.
- **Deep-skin range is thin.** The catalog has few deep nudes, so on deep skin everyday and 90s can share a lipstick. Adding more brands fixes this.
- **Camera exposure.** Phones auto-brighten faces. The white-reference correction helps, but only when it finds clean eye whites or teeth. A smile helps.
- **Rules are hand-tuned.** Next: thumbs up or down on each combo, then tune the weights from that feedback for each skin band.
- **Checkout** links go to product pages until variant IDs are in. Then they become Shopify cart permalinks, one per brand.

## Put it online

The live camera needs https. Any static host works; this folder deploys as-is on Vercel or Netlify. The two data files in `mp/` end in `.wasm` on purpose so every host serves them.
