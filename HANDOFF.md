# HANDOFF — Augmenting English meanings for the Star Gauge characters

This is a light handoff/continuation note for the **"add deeper English definitions to every character after 心 (xīn)"** task. If a session runs out of tokens, point the next session (or yourself) here.

## What we're doing

The vault `Xuanji Vault/Characters/` holds one Markdown file per unique character of Su Hui's **Star Gauge (璇璣圖)**, a 4th-century-CE (Former Qin) palindrome poem. Each file has:

- YAML frontmatter (readings, glyph data, grid positions, `english_meanings:` list, …)
- a body section **`## English meanings`** (the curated, human-facing sense list)
- Shuowen / Guangyun / Kangxi text + translations, reading-path companions, etc.

**Goal of this task:** for **every character *after* 心 in Unicode-codepoint order**, look the character up (at minimum on **ninchanese**, then go deeper — we use **en.wiktionary** because it labels senses as *obsolete/literary/dialectal/slang/Buddhist*), and **append** any well-attested senses that are **missing** from the file.

### Rules (agreed with the user)
1. **Only addition.** Never delete or reword existing bullets. Never duplicate an existing sense.
2. **Depth: comprehensive** — add every period-appropriate classical + literary sense, including grammatical-particle and rarer senses.
3. **Anachronism filter.** The text is ~4th c. CE. **Drop** senses that are modern-only, internet slang, or purely dialectal (e.g. Southern Min / Cantonese-only "to crack", "comfortable", "crowded"). **Keep** classical/literary/obsolete senses (obsolete in Wiktionary = it *was* used anciently → keep). Buddhist senses are borderline-OK (Buddhism was present in China by the 4th c.) — include them but tag `(Buddhist)`.
4. **Mark pronunciations.** When a sense belongs to a different reading, prefix it, e.g. `(also read zī) …`, `(as conjunction) …`, `(literary) …`.
5. **Sync both places.** After editing the body list, the frontmatter `english_meanings:` must be regenerated to match the body (body = source of truth).
6. **Do NOT commit** — leave everything as uncommitted working-tree changes for the user to review.

## Ordering

Work order = **Unicode codepoint order** (this is what the user's earlier hand-commits followed: 周 → 姬 → 心). `心` is U+5FC3. Everything with a higher codepoint is in scope: **317 characters total**.

## Files / tooling (under `.omc/state/`)

- `.omc/state/xin-onward-worklist.tsv` — all 317 target chars (codepoint order) with URL-encodings, `char<TAB>%XX%XX`.
- `.omc/state/xin-onward-progress.txt` — header + one line per **completed** char. **This is the source of truth for "where we are."**
- `/tmp/add_meanings.py` — the working script (regenerate if /tmp was cleared; see "Scripts" below).

### Current progress
- **Done: see the line count of `xin-onward-progress.txt` minus 4 header lines** (that file is authoritative; the number below may lag).
- As of last update: **224 / 317**. Last completed char: **蘭**. Next up in the worklist: `虎 處 虞 虧 …` (verify by reading the worklist after 蘭).

### Side-task already completed
The user also asked to **retroactively sync frontmatter** on the characters they'd already hand-refined (≤ 心). Done: all 143 affected files now have `english_meanings:` matching their body list. No action needed there.

## The workflow (per batch of ~6 chars)

1. **Resume point:** list chars not yet in progress file:
   ```bash
   cd "<repo root>"
   comm -23 <(cut -f1 .omc/state/xin-onward-worklist.tsv) \
            <(tail -n +5 .omc/state/xin-onward-progress.txt | sort) \
     | head -6   # next batch (note: comm needs sorted input; simplest is just read worklist in order and skip done ones)
   ```
   Simpler in practice: open `xin-onward-worklist.tsv`, find the last char in `xin-onward-progress.txt`, take the next 6.

2. **Dump current meanings** (cheap, low context) to avoid duplicates:
   ```bash
   for c in 想 愁 愆 意 …; do echo "=== $c ==="; \
     awk '/^## English meanings/{f=1;next} f&&/^## /{exit} f&&/^- /{print}' \
     "Xuanji Vault/Characters/$c.md"; done
   ```

3. **Research** each char with WebFetch:
   - ninchanese: `https://app.ninchanese.com/word/<URL-ENCODED-CHAR>` (encoding is column 2 of the worklist)
   - wiktionary: `https://en.wiktionary.org/wiki/<CHAR>` — ask it to group senses by pinyin and flag obsolete/literary/dialectal/slang/Buddhist.

4. **Decide additions** (apply the rules above), then **apply** via the script (it appends non-duplicate body bullets AND regenerates frontmatter in one pass):
   ```bash
   python3 /tmp/add_meanings.py <<'JSON'
   {
   "想.md": ["sense 1", "sense 2"],
   "愁.md": ["…"]
   }
   JSON
   ```
   It prints `('OK added=N', path)` per file.

5. **Record progress:**
   ```bash
   printf '想\n愁\n愆\n意\n' >> .omc/state/xin-onward-progress.txt
   ```

> Why a script and not the Edit tool: editing files via the Edit tool forces a prior Read and then the post-edit re-display dumps the whole file into context, which burns tokens fast across 300+ files. The script touches files out-of-band and stays cheap.

## Scripts

If `/tmp/add_meanings.py` is gone, recreate it. It: (a) finds the body `## English meanings` block, (b) appends `- <sense>` lines for senses not already present (case-insensitive dedupe), (c) regenerates the frontmatter `english_meanings:` block (between `english_meanings:` and the next top-level key, normally `scripts:`) from the updated body. Input is JSON on stdin: `{"<file>.md": ["sense", …], …}`, paths relative to `Xuanji Vault/Characters/`.

(There is also a body→frontmatter-only sync version used for the retroactive pass; the combined script supersedes it.)

## Sanity checks before finishing
- Spot-check a few files: body and frontmatter lists match; no `## ` heading got swallowed; no duplicate bullets.
- `git status` should show ~one changed file per processed character (plus this HANDOFF.md). **Leave uncommitted.**
