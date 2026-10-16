# Updating everything when the menu changes

Standing rule from the owner: **any change to the menu or its items updates everything**: the crockery checklist, the plating cards and this site.

The menu source of truth is `food.html` and `drinks.html` in the Dream repo (`~/Documents/Code/Dream`). A Claude Code hook in that project runs `src/check-menu.mjs` after edits and raises a reminder when the menu data no longer matches `src/menu.hash` (the hash of the menu this site was last built from).

## Checklist

1. **See what changed.** `git -C ~/Documents/Code/Dream diff` (or compare dish names). List new, renamed, removed and re-priced items.
2. **Crockery.** Edit the arrays in `src/crockery-source.html`:
   - `ITEMS`: add, remove or re-quantify vessels (id, group, photo id, name, spec, use, qty, unit).
   - `FOOD` / `DRINKS`: the dish-to-vessel map, one row per menu line (`[name, sub, vesselId, alongside, why]`). Every menu line needs a row.
   - If a new vessel is needed, generate its image (see Images).
3. **Plating.** Edit `src/plating_data.py`: one entry per food item or drink (`add(...)` / `dadd(...)`), with the image prompt, build steps, garnish, side, serving temperature and note. Remove entries for dropped items.
4. **Images** (free, via Google Flow; never use a paid generator without asking first):
   - Sign in is saved as the `brandcanvastc` gflow profile (`gflow auth status`). Flow project id `5657dbf4-d0a4-414b-9fda-48d501728c40`.
   - Crockery photos: `gflow image t2i "<prompt>" --model nano2 --aspect 4:3 -n 1 --project <id> -o <file>.png`, then resize to 800x600 JPEG into `img/crockery/<photoId>.jpg`. Style: catalogue studio shot on plain warm sand-beige paper, soft daylight from the left, no text.
   - Plating photos: `gflow image i2i "<prompt>" --ref img/crockery/<vessel>.jpg ... -o <file>.png`, resize to 900x675 JPEG into `img/plating/<id>.jpg`. See `src/plating-image-jobs.tsv` for the prompt style (warm oak table for food, dark bar top for drinks).
   - Regenerate an image only when the dish or vessel changed.
   - **Veg / non-veg options:** every option the menu offers has its own plating photo. `VARS` in `src/plating_data.py` lists the options per dish (label, veg/non-veg mark, image id); the plating cards show them as tabs and the photo menus as one photo per option. To add an option or a photo-less item, add its prompt to `src/plating-variant-jobs.py` and run `src/gen-variants.sh <id>` (free Flow images, same project, writes `img/plating/<id>.jpg`; it skips ids that exist, so delete a file to redo it).
5. **Rebuild the site.** `node src/build.mjs` (writes `index.html`, reports missing images). Check it in a browser at desktop and phone width.
6. **Crockery checklist artifact (claude.ai).** Publish `src/crockery-source.html` to https://claude.ai/artifact/MkGM2HfkUPEgK7cZSjz5zv with its photos at `g/<photoId>.jpg` (copy `img/crockery/*.jpg` into a temp folder `g/`). It keeps the owner's ticks stored in the browser under key `dd-crockery-ticks-v2`, so keep item ids stable.
7. **Photo menus.** `python3 src/build-menus.py` rebuilds `food-menu.html` and `drinks-menu.html` (the guest menus with a plated photo beside each dish, text from the live Dream menu, photos from `img/plating`). A new dish needs a plating photo and, if its name differs from its plating card, an entry in `GROUP`/`LINE_IMGS` in the script. Print them to A3 PDF: `chrome --headless=new --no-pdf-header-footer --print-to-pdf=food-menu.pdf food-menu.html` (same for drinks).
8. **Push.** Commit and push this repo to `main`; GitHub Pages rebuilds at https://singhgenics.github.io/Menu/ . Verify the live page loads with no broken images.
8a. **Brand.** The pages carry the dimo brand (Fredoka wordmark + four enamel signs). `src/brand-snippets.json` holds the baked wordmark and signs for `build-menus.py`. To re-apply the brand to Menu sources, Dream menus or a pub model, use the idempotent scripts in `~/Documents/Code/Dream/Dimo/src/` (`apply_live_menu_site.py`, `apply_live_menus.py`, `apply_live_model.py`). Brand edits never touch menu data, so the change detector stays quiet.
9. **Record it.** `node src/check-menu.mjs --update` so the hook goes quiet.
10. Tell the owner what changed on each surface.

## Furnishings (the fourth view)

The furnishings schedule follows the **Simplex room** (`~/Documents/Code/Dream/Simplex`: 4.9 × 8.8 m, 18 covers inside + 8 in the garden), not the menu. When the Simplex layout or the 3D model changes (tables, bar, lights, AC), update it:

1. Edit `src/furnishings_data.py`: one tuple per line in each `grp(...)` (code, name, size, material, qty, unit, low and high unit price in INR, phase 1 or 2, where, note, image id). Prices are planning ranges, not quotes. Codes (S1, T1, B1, L1, C1, ...) are what the two plans use.
2. Edit `src/furnishing_plans.py` if a piece moves: it draws `img/furnishings/furniture-plan.svg` and `ceiling-plan.svg` from the same x/z coordinates as the Simplex model (x from the right wall -2.45 to the left wall +2.45, z from the shopfront 0 to the back wall 8.8) and prints the fan clearance checks.
3. Images (free, Google Flow, same project as the rest): prompts are in `src/furnishing-jobs.py`; run `src/gen-furnishings.sh [id...]` (runs one at a time, because the gflow profile allows one process; it skips ids that exist, so delete a `img/furnishings/<id>.jpg` to redo it). Room shots use screenshots of the Simplex 3D model in `src/furnishing-refs/` as the layout reference.
4. `node src/build.mjs` regenerates `furnishings.json`, both plans and `index.html`; it reports missing images.
5. Commit and push as usual.

## Files

- `index.html`: generated, do not edit by hand.
- `src/template.html`: page design and scripts.
- `src/build.mjs`, `src/menu-data.mjs`: build and menu reader.
- `src/crockery-source.html`: crockery data (also the source of the claude.ai checklist artifact).
- `src/plating_data.py`: plating specs and image prompts.
- `src/check-menu.mjs`, `src/menu.hash`: change detector used by the hook.
- `src/plating-variant-jobs.py`, `src/gen-variants.sh`: prompts and runner for the option photos and the drinks that had none.
- `src/build-menus.py`, `src/menus.css`: build the two photo menus; `food-menu.html`, `drinks-menu.html` (+ `.pdf`) are generated, do not edit by hand.
