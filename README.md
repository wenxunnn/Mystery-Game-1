# The Ashbourne Inquiry — a 30-minute research-group murder mystery

A frontend-only detective game for your research group. No backend, no database — just `index.html`, `style.css`, and `script.js`. Works locally and on GitHub Pages, and is laid out to work comfortably on a phone screen as well as a laptop.

**Before game day:** print `cooling-rate-reference-card.html` (open it in a browser and use the "Print this card" button, or your browser's normal print/save-as-PDF). One clue genuinely requires it — there's no way to solve it from the screen alone.

## 1. Run it locally

1. Download all four files (`index.html`, `style.css`, `script.js`, `cooling-rate-reference-card.html`) into the same folder.
2. Double-click `index.html`, or right-click → "Open with" → your browser.
3. That's it. No installation, no server.
4. Separately, open `cooling-rate-reference-card.html` and print it (or save as PDF) before game day — see the note above.

If your browser blocks local fonts/scripts in rare setups, you can also run a tiny local server (optional): in the folder, run `python3 -m http.server`, then visit `http://localhost:8000`.

## 2. Publish on GitHub Pages

1. Create a new GitHub repository (e.g. `ashbourne-inquiry`).
2. Upload all four files to the root of the repo.
3. Go to **Settings → Pages**.
4. Under "Build and deployment," set **Source: Deploy from a branch**, branch: `main`, folder: `/ (root)`.
5. Save. GitHub gives you a URL like `https://yourusername.github.io/ashbourne-inquiry/` within a minute or two.
6. Open that link on game day — works on phones, tablets, and laptops. The reference card page works the same way if you'd rather print it from the published URL than from a local copy.

## 3. How the game is organized

Open `script.js` and look at the top: it's split into two clearly labeled parts.

- **Part 1, CONFIGURATION** — this is the only part you'll normally touch:
  - `CARDS` — the list of case-file cards on the dashboard (title, subtitle, which screen they open).
  - `UNLOCKS` — which clue unlocks which card (e.g. solving `clue1` unlocks `autopsy`).
  - `HINTS` — an array of hints per clue, vaguest first. Tapping the hint button once reveals the first hint; tapping again reveals the second, and so on. The button disables itself once all hints for that clue are shown. This is intentional: a team that taps Hint once still has to do most of the thinking themselves.
  - `CHECKERS` — the function that decides if a typed answer is correct.
  - `MESSAGES` — the feedback text shown after submitting an answer.
- **Part 2, ENGINE** — state saving, rendering, navigation, and the hint-reveal logic. You shouldn't need to edit this.

The actual report text (the letter, autopsy table, police report, ledger) lives directly in `index.html`, since it's easier to edit prose and tables in HTML than inside JavaScript strings. `cooling-rate-reference-card.html` is a separate, self-contained printable page — it isn't linked from the main site on purpose, so players can't shortcut the physical prop by clicking around.

## 4. How to change an answer

Find the relevant entry in `CHECKERS` in `script.js`. For example, to change the Job ID answer for clue 4:

```js
clue4: function (raw) {
  const n = raw.toLowerCase().replace(/[\s-]/g, "");
  return n.includes("2289");   // <-- change this to your new Job ID
},
```

Each checker normalizes the player's text first (lowercase, strips spaces/punctuation/titles), so you don't need to anticipate every possible typo — just check what the normalized text should contain.

## 5. How to change clue text, reports, or hints

- **Report content** (the letter, autopsy table, suspect cards, etc.): open `index.html` and edit the text directly inside the matching `<section id="screen-...">` block. Each block is labeled with an HTML comment.
- **Hint text**: edit the matching array inside `HINTS` in `script.js`. Each clue has a list of strings, shown one at a time, vaguest first — add or remove entries as you like, or change the wording, but keep the vague-to-specific order.
- **Puzzle prompt text**: edit the `<label>` or `<blockquote>` text inside the relevant screen in `index.html`.
- **The cooling-rate table**: edit the values directly inside `cooling-rate-reference-card.html`. If you change the rate for whichever month you use in the story, also recompute the target time of death and update `CHECKERS.clue2`'s comparison value in `script.js` (it currently checks for 4:00 PM / 16:00).

## 6. How to add or remove a clue/page

To add a new evidence page between two existing ones:

1. Copy an existing `<section class="screen hidden" id="screen-...">` block in `index.html`, give it a new `id`, and write your content. Include an `.answer-box` div with a unique `data-clue` value if it has a puzzle.
2. Add a new entry to `CARDS` in `script.js` pointing at your new screen id.
3. Add a line to `UNLOCKS` saying which clue unlocks your new card, and update the clue that should unlock it next so the chain stays connected.
4. Add a checker function to `CHECKERS` and a hint to `HINTS` if your new page has a puzzle.

To remove a page, just delete its `CARDS` entry, its `UNLOCKS` line, and its `<section>` block — then make sure the clue before it points its unlock at whatever comes after it.

## 7. Testing tips

- The **"Reset case (testing)"** link at the bottom of the dashboard clears all progress (it wipes the browser's `localStorage` for this page) so you can replay from the start.
- Progress is saved per browser/device. If you want every team to start fresh on separate laptops, that happens automatically — there's nothing shared between devices.
- Open the browser console (F12) if something doesn't look right; any JavaScript errors will show there.

## 8. Physical props

- **Required:** `cooling-rate-reference-card.html`, printed ahead of time. The autopsy report deliberately withholds the cooling rate — players need the printed card to look up the correct rate for the month listed in the report, then do the math themselves. There's no digital fallback, so don't forget to print it.
- A printed Roman numeral cheat sheet (you mentioned you already have this).
- A printed copy of the suspect list, used as physical cards the team can lay out and annotate during discussion.
- An "evidence folder" or envelope labeled **CASE FILE #4471 — CONFIDENTIAL** to hold any printouts and set the mood at the table.
- A magnifying glass or red string/pushpins on a small board if you want a literal "case board" centerpiece.
- A simple kitchen timer for the optional 30-minute countdown.

## 9. Game flow summary

| Step | Page | What players do | Unlocks |
|---|---|---|---|
| 1 | Letter from Margaret Hale | Read the background | — |
| 2 | The Study Safe | Solve the Roman numeral puzzle | Autopsy Report |
| 3 | Autopsy Report | Calculate time of death from body temperature | Police Report |
| 4 | Police Report | Match the time of death to the security log and contradicted alibi | Company Ledger |
| 5 | Company Ledger | Spot the falsified accounting entry | Final Verdict |
| 6 | Final Verdict | Name the murderer | Ending |

Total play time is roughly 25–35 minutes for a group of 3–5 working together.
