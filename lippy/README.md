# Lippy

Live lip-combo try-on. Open the camera, get 3 combos picked for your lips, turn liner, lipstick and gloss on and off, swap shades, and lock in pieces to check out at each brand's own Shopify store.

## Put it online (about 2 minutes)

The live camera only works on a real https website, so host this folder:

**Netlify Drop (no setup)**
1. Unzip this folder.
2. Go to https://app.netlify.com/drop
3. Drag the whole `lippy-site` folder onto the page.
4. Open the link it gives you on your phone and tap "try it on live".

**Vercel (if you have an account)**
1. In a terminal, `cd lippy-site`
2. Run `npx vercel` and follow the prompts.

## What's inside

- `index.html`: the whole app (layout, styles, try-on engine, combos, collection).
- `mp/`: MediaPipe Face Mesh, which finds 468 points on your face every frame, including the exact lip outline. It runs on the phone; nothing is uploaded. The two data files end in `.wasm` on purpose so any static host serves them.

## How the 3 combos are picked

From the first second of video, Lippy reads your natural lip color (inside the traced lip outline) and your skin (cheek points), then:

- **everyday**: the lipstick closest to your natural lip color, one step richer, with a liner one step deeper.
- **90s brown**: a deep brown liner and a nude matched to your skin depth.
- **night out**: the bold shade closest to your undertone and skin depth.

If you're already wearing lip color, Lippy notices (too saturated, lighter than your skin, an unusual hue, or very shiny), estimates your natural lip color from your skin, and paints on that natural base so products show true.

Shade colors are estimated from each brand's shade description. Next step: pull real swatch colors and variant IDs from Shopify's Global Catalog so "check out" goes straight to each brand's cart.
