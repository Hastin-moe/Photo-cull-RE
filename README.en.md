# photo-cull

[简体中文](README.md) · [繁體中文](README.zh-TW.md) · **English** · [日本語](README.ja.md) · [한국어](README.ko.md)

photo-cull contains two programs that live in **one window** — switch between them at the top right (⌘1/⌘2) and both keep their state. On macOS it can be packaged as a standalone **照片工具.app** that needs no Terminal (see “Mac app” below):

- **Photo Cull** (double-click `启动照片工具.command`): technical culling on your own computer — focus, blur, exposure, faces and closed eyes, plus grouping of bursts and repeated shots of the same scene; the best frame of each burst is chosen by “highlight weights”. Those photos can then get an aesthetic review from **Claude, ChatGPT, Grok, Gemini, DeepSeek**, or any OpenAI-compatible vision model (Qwen, Kimi, OpenRouter, a local Ollama, …). Scores use a **10-point scale**: local technical scores go up to 6, and only AI review gives more than 6. Scores are converted to **star ratings and color labels** and written to XMP, which Lightroom Classic and Capture One both read, and you get an interactive web report.
- **Film Lab** (a separately installed third module, [photo-cull-film](https://github.com/Hastin-moe/photo-cull-film), GPLv3, Chinese interface for now): spectral film simulation with [spektrafilm](https://github.com/andreavolpato/spektrafilm). Once installed, the one window gets a third tab.
- **Photo Edit** (basic): global adjustments with curves, a histogram and waveform scopes; apply `.cube` LUTs and Lightroom/Photoshop XMP presets; export JPEG or 16-bit TIFF. Supports Leica DNG, Panasonic DNG, every Sony ARW and Fujifilm RAF.

> The launcher files keep their Chinese names: `启动照片工具` = start Photo Tools, `tools/修复环境` = repair environment, `打包Mac应用` = build Mac app.

**Interface languages**: 简体中文, 繁體中文, English, 日本語 and 한국어 (the five Mac system languages; Traditional Chinese added 11-21) — switch at the top right of either program (both stay in sync). The report, the run log, the run report, AI comments and **every folder and file name** the program creates (organized output, quick picks, report folder) follow the chosen language. On first launch the language is picked from your browser.

**RAW+JPEG pairing**: a RAW (DNG, ARW, RAF…) and a JPEG with the same name in the same folder are treated as one photo. Scores are written to both; deleting removes both.

**Safety**
- Image data is never modified.
  - RAW ratings go into a sidecar `same-name.xmp`; the RAW file is untouched.
  - JPEG ratings go into the embedded XMP segment (metadata only). After writing, the image data — including any multi-picture previews from the camera — is compared byte for byte, and the write is abandoned on any mismatch.
- Existing XMP keeps everything except rating and label (develop settings, keywords…). Before the first change to a sidecar written by another app, a `.xmp.cull-bak` backup is made.
- “Delete” only moves files to the **Trash / Recycle Bin**, so you can restore them.
- Organizing **copies** by default and leaves the original folder alone; files are moved only if you choose “Move”.

## Install (once)

macOS only (Windows support is paused as of 2026-11-17). You need Python 3.9 or later ([python.org](https://www.python.org/downloads/)).

Double-click **`启动照片工具.command`** (start Photo Tools). The first run creates the environment and installs the dependencies (about 1–5 minutes, needs the internet); after that it opens straight away. It starts with Let’s start; once you pick a photo folder, culling, editing (and Film Lab, if installed) share one window.

### Repairing the environment

If the environment breaks, double-click **`tools/修复环境.command`** (repair environment). It:

1. creates a private environment `.venv` inside the tool folder without touching other Python installs;
2. installs the dependencies in `requirements.txt`, trying PyPI first and switching to the Tsinghua mirror if PyPI can’t be reached within 4 seconds;
3. checks that each component actually works (numpy, Pillow, OpenCV, rawpy/LibRaw, ExifRead and cryptography are required; HEIC, Trash and AI review are optional), then checks the built-in LUTs and the settings folder, and lists the results.

When `requirements.txt` changes, the launcher installs what’s missing automatically. Options in Terminal:

| Option | What it does |
|---|---|
| `--check` | check only, install nothing |
| `--recreate` | delete the old environment and reinstall (when it’s broken or Python changed) |
| `--mirror tuna`/`aliyun`/`ustc`/`pypi`/URL | choose the pip index (default `auto`) |
| `--wheels FOLDER` | install offline from a local folder |
| `--download FOLDER --target mac-arm/mac-intel` | on a computer with internet, download the packages first (with `--target-python`), then install them offline with `--wheels` |

To install by hand, open Terminal:

```bash
cd ~/Desktop/照片筛选工具
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
```

## Mac app (recommended)

Double-click **`打包Mac应用.command`** (build Mac app) in the tool folder to create `dist/mac/照片工具.app`:
- it bundles Python and every dependency, so it also runs on other Macs (Apple silicon, macOS 11 or later);
- a native window with no Terminal and no browser; Photo Cull and Photo Edit switch inside the same window;
- the first build downloads the packaging tools (about 3–8 minutes) into a separate `.venv-build`, leaving your `.venv` alone;
- after building it runs a self-test (dependencies, built-in LUTs, both services) and creates `dist/照片工具-<version>-macOS-app.zip` to share.

Drag `照片工具.app` into Applications and open it from Launchpad or the Dock; close the window to quit. Logs are in `~/.photo-cull/logs/app.log`, and the build log in `dist/mac/build.log`.
If you download an app someone else built, right-click → Open the first time (it is not notarized by Apple).

**Development edition and release package**: an app built in this folder (the development edition) **needs no activation code** — it’s for the author’s own use. To give the app to others, double-click `tools/导出发布版.command` (make release package): it exports a slimmed copy to a temporary folder, writes in the hash of the shared activation code (the code itself is never written to any file), builds the app, and puts the zip, the install guide and the release record into the neighbouring `photo-cull-main-release`, with a copy in `Grey-test/<version>/`. That app asks for the activation code on first launch. `Grey-test` holds each test version’s package and test documents only; the slimmed test source is exported with `tools/导出灰度版.command` to `build/grey-export` and stays local; the private repo Photo-cull-RE holds only descriptions and development logs. Code is changed only in the development folder.

**Activation code**: only the release edition checks it. The author issues codes with `tools/激活码生成器.command` (activation code generator): its first run creates a key pair — the private key stays only in `~/.photo-cull-keys/` on the author’s Mac (back it up!), the public key goes into `photo_cull/activation_key.py` and ships with the app. A code can have an expiry date and can be tied to one computer’s machine ID (shown on the activation page). The build script refuses to build until the key exists.

## Using the interface

Double-click **`启动照片工具.command`**: your browser opens the Photo Tools window. Switch with “Photo Cull | Photo Edit” at the top right. Double-clicking the launcher again just opens the existing window instead of starting another copy.

Photo Cull:

1. Click “Choose…” and pick a photo folder. If the folder was culled before, “Load the last results for this folder” lets you continue reviewing.
2. Choose “Local only” or “Local + AI review”. For AI review:
   - pick a provider: Claude, ChatGPT, Grok, Gemini, DeepSeek or Custom;
   - pick a model from the list (each is marked recommended / cheaper / strongest), or choose “Other” and type one;
   - enter that provider’s API key; for Custom, also enter the endpoint URL.
3. Turn on “Dry run” the first time to see how well it judges.
   Turn on “Organize output” to sort and rename after rating (see “Organizing output” below).
4. Click “Start”. Before any AI call, a dialog shows how many photos will be sent and the estimated usage; nothing is sent until you confirm.
5. The report appears in the window. In it you can:
   - click ★ to give 1–10 points (7–10 is the AI range, but you can set it by hand) or ✕ to reject; changes are converted to stars and written to the RAW and the JPEG immediately;
   - tick the box on a photo and click “Delete selected”, or “Delete all rejects”; paired files and their XMP go to the Trash together;
   - click “Organize by current scores” after reviewing to sort into the output folder using your edited scores;
   - click “Edit” on a photo to switch to Photo Edit with that photo open.

Everything runs locally on 127.0.0.1 and is not reachable from other computers. Close the terminal window to quit.
If macOS says the app “can’t be opened because it is from an unidentified developer”: right-click the launcher → Open → Open.

**Portrait field**: when shooting people (Type of shoot “People” or “Candid” in the full cull, or genre “Portrait” in the quick pick), you can also pick a field — cosplay, commercial, outdoor, event — up to two (with two, the AI balances both standards, e.g. outdoor cosplay). The AI scores and comments by that field’s standards and knows what such shoots can’t control (no relighting at events, no clearing a convention floor), so it won’t mark down for those or suggest them. Affects AI review only; on the command line `--portrait-style cosplay,outdoor`.

**AI comments**: the comment names the biggest strength and the one problem that hurts most, both pointing at specific places in the frame; issues list first what can be fixed in post, then what could change next time under the same conditions, never things the conditions rule out; problems caused by the conditions (low-light noise, mixed venue light) are explained with a fix in post.

## Models for AI review

| Provider | Models (first is the default) | Get an API key at | Environment variable |
|---|---|---|---|
| Claude | `claude-sonnet-5-5` (recommended), `claude-haiku-5-5` (cheaper), `claude-opus-5-5` (stronger), `claude-fable-5-1` (strongest, most expensive) | console.anthropic.com | `ANTHROPIC_API_KEY` |
| ChatGPT | `gpt-6.1-sol` (recommended), `gpt-6-luna` (cheaper), `gpt-6-astra` (strongest), `gpt-5.5` (previous) | platform.openai.com | `OPENAI_API_KEY` |
| Grok | `grok-4.7` | console.x.ai | `XAI_API_KEY` |
| Gemini | `gemini-3.8-flash` (recommended), `gemini-3.6-flash`, `gemini-3.5-flash-lite` (cheaper), `gemini-4-argon` (strongest) | aistudio.google.com | `GEMINI_API_KEY` (or `GOOGLE_API_KEY`) |
| DeepSeek | `deepseek-flash` | platform.deepseek.com | `DEEPSEEK_API_KEY` |
| Custom | your choice | your provider | `PHOTO_CULL_CUSTOM_API_KEY` |

- The list was checked in October 2026 and only includes models that accept images. When a provider releases a new model, choose “Other (type a name)”.
- **DeepSeek**: only `deepseek-flash` accepts images right now (`deepseek-v4-pro` is text-only), so it is the only option.
- **Custom (OpenAI-compatible)**: enter an endpoint and a model name to use any service that supports image input through Chat Completions, for example:
  - OpenRouter `https://openrouter.ai/api/v1`
  - Qwen (international) `https://dashscope-intl.aliyuncs.com/compatible-mode/v1` (e.g. `qwen-vl-max`)
  - Kimi `https://api.moonshot.ai/v1`
  - a local Ollama `http://127.0.0.1:11434/v1` or LM Studio `http://127.0.0.1:1234/v1` (no key needed)

  Check each provider’s docs for exact URLs and model names. DeepSeek and Custom use plain HTTP and need no extra SDK.
- API keys are pay-as-you-go and separate from chat subscriptions (Claude Pro, ChatGPT Plus…).
- Photos already reviewed aren’t charged again: the cache is keyed by provider + model + language + image. Switching the interface language reviews again once so the comments are in the new language.
- If the first 3 calls in a row fail (wrong key or model name, no credit…), AI review stops automatically.

## Command line

```bash
# Local culling only, free
.venv/bin/python cull.py "/Volumes/SSD/2026-10-19 street"

# Dry run first: report only, no ratings written
.venv/bin/python cull.py "photo folder" --dry-run

# Claude review
export ANTHROPIC_API_KEY="…"
.venv/bin/python cull.py "photo folder" --ai

# ChatGPT / Grok / Gemini / DeepSeek
.venv/bin/python cull.py "photo folder" --ai --provider openai     # OPENAI_API_KEY
.venv/bin/python cull.py "photo folder" --ai --provider grok       # XAI_API_KEY
.venv/bin/python cull.py "photo folder" --ai --provider gemini     # GEMINI_API_KEY
.venv/bin/python cull.py "photo folder" --ai --provider deepseek   # DEEPSEEK_API_KEY

# Any OpenAI-compatible endpoint (e.g. OpenRouter)
export PHOTO_CULL_CUSTOM_API_KEY="…"
.venv/bin/python cull.py "photo folder" --ai --provider custom --model <model> --base-url https://openrouter.ai/api/v1

# Report, log and AI comments in English (zh-CN / en / ja / ko)
.venv/bin/python cull.py "photo folder" --lang en

# After rating, move rejects (with paired files) to the Trash
.venv/bin/python cull.py "photo folder" --trash-rejects

# Organize: Kept (Picks / Burst moments / Usable) / Rejected × JPEG/RAW, renamed by score (copies by default; add --move to move)
.venv/bin/python cull.py "photo folder" --organize "output folder" --by-result --by-category --by-format --rename

# Change the pick line and the highlight weights
.venv/bin/python cull.py "photo folder" --ai --pick-ai 7.5 --weights sharp=0.5,eyes=0.3
```

The report is written to `photo folder/photo-cull-report/report.html`, next to `results.csv` for Numbers or Excel. A report opened directly from disk is read-only; use the program to change scores or delete.

## Scoring (10-point scale)

- The **local technical score** goes up to **6**: focus (compared within the batch) 60%, exposure 30%, faces and open eyes 10%, mapped to 1.0–6.0. “Possibly soft” photos get at most 4; badly out of focus, over- or underexposed photos are rejected.
- **AI review** gives an overall score from 1.0 to 10.0; **only AI gives more than 6**. For reviewed photos the AI score is used.
- Scores are **kept to 0.1 internally**; the interface, report and file names show only the whole number.
- In XMP the score is **converted to 1–5 stars: score ÷ 2, rounded down** (7.9 → 3★, 8.0 → 4★). Rejects are written as “reject” (Rating −1).

| Score | XMP stars | Label |
|---|---|---|
| 10 | 5★ | Green |
| 8–9.9 | 4★ | Green |
| 6–7.9 | 3★ | Yellow |
| 4–5.9 | 2★ | none |
| 2–3.9 | 1★ | Red |
| 0–1.9 | unrated | none |
| Rejected | Reject (−1) | Red |

**Categories**: every kept photo falls into one of three (rejects and unreadable files are counted separately):

| Category | How |
|---|---|
| Pick | A single photo at or above the pick line: ≥ 7 when AI-reviewed, ≥ 5 with a local score only (both lines can be changed under “Advanced”, or with `--pick-ai` / `--pick-local`); a **burst’s representative** (the frame sent to the AI) that reaches the AI pick line is a pick too (“Count burst representatives as picks”, on by default; `--no-pick-bursts` turns it off) |
| Burst moment | Each burst (including repeated shots of a scene) **shortlists 3–5 frames** by the highlight weights (3 for 4–19 frames, 4 for 20–59, 5 for 60 or more; bursts of up to 3 frames are shortlisted whole), spread out in time so a 120 fps burst doesn’t yield near-identical frames. The best of the shortlist is the burst’s **representative** and goes to the AI review (the rest of the shortlist too if you tick “Also review the other shortlisted burst frames”); other shortlisted frames score at most 5.9 |
| Usable | Every other kept photo, including burst frames that were not shortlisted (at most 4.9, i.e. 2★) |

**Highlight weights** (defaults; change them under “Advanced” or with `--weights`): within a burst, one frame is chosen by the weighted sum below. Each factor is compared within the burst; the weights don’t need to add up to 1.

| Factor | Default | What it looks at |
|---|---|---|
| Subject sharpness | 0.35 | sharpest frame in the burst |
| Eyes open | 0.25 | share of detected faces with eyes found |
| Smile | 0.10 | share of detected faces that smile |
| Exposure | 0.15 | brightness in range, no clipping |
| Subject prominence | 0.15 | how much of the frame the largest face fills, compared with the other frames |

When every frame of a burst has an AI score (“Also review the other burst frames”), the choice is made again with AI 60% + highlight score 40%.

**Where ratings are written**

| File | Written to |
|---|---|
| RAW (DNG/ARW…) | The same-name `.xmp` sidecar, shared by a RAW+JPEG pair |
| JPEG | Embedded XMP (turn off “Also embed XMP in JPEGs” or use `--no-jpeg-embed` to skip) |
| Standalone HEIC/HIF/PNG | Not by default; turn on “Write XMP sidecars for standalone JPEG/HEIC too” for Capture One |

**Lightroom Classic**
- Newly imported photos: ratings are read on import.
- Photos already in the catalog:
  1. select them and press ⌘S (Save Metadata to File) so the catalog’s latest changes are in the files;
  2. run photo-cull;
  3. select them again and choose Metadata → Read Metadata from Files.
- If RAW+JPEG were imported as one photo (Lightroom’s default), Lightroom shows only the RAW’s rating. To see both, turn on “Treat JPEG files next to raw files as separate photos” in Preferences → General before importing.

**Capture One**
- In Preferences → Image → Metadata, set XMP sidecar auto sync to “Full sync” or “Load”. Then open the folder, or run “Reload metadata from XMP” on the images (menu names vary by version).

**Deleting and catalogs**
- Photos already in Lightroom or Capture One show as missing after being deleted here; remove them in that app too.
- Deleted the wrong one? Open the Trash and drag the files back.

## Organizing output

After rating, photos can be organized into an output folder (`photo folder/Organized` if left empty). Layout (since 2026-11-17):

- **One parent folder per cull**, named `camera_time_field`, e.g. `ILCE-1M2_261117-2039_Portrait-Cosplay+Outdoor`. The field is the portrait field(s) when shooting people with fields chosen, otherwise the type of shoot (General, Action…); with several cameras in one batch, the most-used one is named followed by the camera count.
- The parent folder holds **only Picks and Usable**.
- **Bursts**: each burst gets **2–3 frames in Usable**, one subfolder per burst (e.g. `Usable/Burst012_103045`): the 2 best others when the representative made Picks, otherwise the representative plus the 2 best. **The rest of the burst is moved** into a new `Bursts_camera_time_field` folder inside the original photo folder, one subfolder per burst — **moved, not copied, in both copy and move mode.**
- **Rejects are not organized** and stay where they are. In move mode, a photo you later reject in the report is moved back to the photo folder under its original name; in copy mode its copy is deleted.
- Copy / move apply to Picks and Usable: copying leaves those originals alone (an APFS clone, no extra space); moving puts them in the output folder.
- On the command line, `--by-result` / `--by-category` still give the old Kept / Rejected layout.

The other two options can still be switched separately:
- **Then split into JPEG / RAW**: HEIC/PNG go with JPEG.
- **Rename**: prefix the name with `score_camera_` — a two-digit score, so sorting by name sorts by score; camera from EXIF, e.g.
  `08_ILCE-1M2_DSC01234.ARW`, `06_LEICA-M11-D_L1003467.DNG`, `X_X-T5_DSCF0001.RAF`.
  Rejects get `X_`; spaces and special characters in the camera name become `-`.
  **Only files in the output folder are renamed**; originals keep their names.
- **AI-reviewed photos get a unified name** (when renaming): `category code + two-digit score _ ID _ original name _R stars`, e.g.
  `P09_AI261104-2315-001_DSC01234_R4.ARW`. Codes: `P` pick, `M` burst moment, `U` usable, `X` rejected (score `00`). The `_R0`–`_R5` at the end is the 5-star rating (same as XMP: score ÷ 2 rounded down), `_RX` for rejects; the XMP sidecar is renamed to match.
  The ID is `AI<batch>-<number>`: the batch is when this run started (YYMMDD-HHMM), numbers run from the highest score down, and a RAW/JPEG pair shares one ID.
  To count them, split on `_` or use `^([PMUX])(\d{2})_AI(\d{6}-\d{4})-(\d{3,})_(.+)_R([0-5X])(\.[^.]+)?$`.
- **AI review notes**: each AI review run writes `AI-review_<batch>.md` (overview table and per-photo comments) and a `.csv` into the report folder, plus a copy in the output folder when organized. They are rewritten after you change a score, delete or reorganize.
- **Light table**: after a run, switch the result area from “Report” to “Light table”. It shows only AI-reviewed photos by ID, with a large view, the comment, breakdown scores and a filmstrip or grid.
  Keys: `←`/`→` move, `1`–`9` and `0` (= 10) score, `X` reject, `F` full screen, `G` single/grid.

Folder names follow the interface language at the time you organize (中文 `保留`/`精选`/`连拍瞬间`/`可用`/`淘汰`, English `Kept`/`Picks`/`Burst moments`/`Usable`/`Rejected`, and likewise for 日本語 and 한국어) and are saved with the organize record, so later updates from the report reuse the same names. When a photo changes category, folders left empty (such as a burst moment folder) are removed.

With everything on:

```
Organized/
├── Kept/
│   ├── Picks/
│   │   ├── JPEG/  08_ILCE-1M2_DSC01234.JPG
│   │   └── RAW/   08_ILCE-1M2_DSC01234.ARW  08_ILCE-1M2_DSC01234.xmp
│   ├── Burst moments/
│   │   └── Burst012_103045/
│   │       ├── JPEG/  07_ILCE-1M2_DSC01240.JPG
│   │       └── RAW/   07_ILCE-1M2_DSC01240.ARW
│   └── Usable/
│       ├── JPEG/
│       └── RAW/
└── Rejected/
    ├── JPEG/
    └── RAW/
```

**Copy or move**
- **Copy** (default): the original folder stays as it is and the output holds copies (twice the space). Later score changes in the report move the copy to the matching folder and name; deleting a photo also sends its copy to the Trash. Organizing again never makes a second copy.
- **Move**: photos leave the original folder. Score changes and deletions in the report then apply to the files in their new place. Photos already in Lightroom / Capture One will show as missing and need their folder relocated there.

**Details**
- Paired RAW and JPEG travel together; a RAW’s `.xmp` sidecar follows it and is renamed with it, so Lightroom / Capture One see the ratings when importing the output folder.
- Name clashes get `-2`, `-3`…; both files of a pair share the same suffix.
- In dry-run mode the plan is listed but nothing is copied or moved.
- Unreadable and deleted photos are skipped.
- The output folder may live inside the photo folder: with “Include subfolders” on, the next cull skips it.

## Common options

| Option | Effect |
|---|---|
| `-r` | Include subfolders |
| `--dry-run` | Report only, no ratings written |
| `--no-xmp` | Report only |
| `--no-jpeg-embed` | Leave JPEG files alone; write RAW sidecars only |
| `--ai` | AI review for the chosen photos (singles and each burst moment); gives scores above 6 |
| `--provider claude / openai / grok / gemini / deepseek / custom` | Which provider |
| `--model` | Model name |
| `--base-url` | Endpoint for `--provider custom` |
| `--lang zh-CN / en / ja / ko` | Language of the report, log and AI comments (default: the interface setting or system language) |
| `--ai-alts` | Also review the burst frames that were not chosen (costs more) |
| `--trash-rejects` | Move rejects (with paired files) to the Trash |
| `--organize [folder]` | Organize after rating; default `photo folder/Organized`; relative paths start at the photo folder |
| `--by-result` / `--by-category` / `--by-format` | Split into Kept/Rejected, Picks/Burst moments/Usable and JPEG/RAW folders |
| `--rename` | Rename to `score_camera_original` |
| `--move` | Move instead of copy |
| `--yes` | Don’t ask before calling the API or deleting |
| `--burst-gap 15` | Burst: shots at most this many seconds apart that look alike form a group |
| `--scene-gap 60` | Same scene: shots at most this many seconds apart that look very alike also group (for slower shooting, e.g. rangefinders) |
| `--similarity 0.6` | Scene similarity threshold, 0–1; higher is stricter |
| `--blur-warn 0.58` / `--blur-reject 0.75` | Blur above the first is flagged “possibly soft”; above the second, rejected |
| `--pick-ai 7` / `--pick-local 5` | Pick lines (10-point scale) for AI-reviewed / local-only photos |
| `--weights sharp=0.35,eyes=0.25,…` | Highlight weights (`sharp`, `eyes`, `smile`, `exposure`, `subject`); omitted ones keep their defaults |

## Notes and limits

- **Which file is analyzed**: with a paired JPEG, the JPEG (full size, in-camera colors); otherwise the RAW’s embedded preview. TIFF-based RAW (DNG, ARW) and Fujifilm RAF are read directly; formats like CR3 need rawpy.
- **Pairing**: same folder, same name, different extension (case-insensitive), e.g. `L1003467.DNG` and `L1003467.JPG`. Files in different folders are not paired.
- **Focus check**: uses a re-blur metric — blur the image again and measure how much the edges change — so it doesn’t depend on how much detail a scene has, and only the sharpest region counts, so portraits with blurred backgrounds aren’t misjudged. Default thresholds were calibrated on Leica M11-D files; other cameras may need tuning.
- **Faces, closed eyes and smiles**: OpenCV’s classic detectors — fast but rough, so they only add a hint and help choose burst moments; they never reject a photo.
- **AI review**:
  - only the chosen photos (singles, and the frame chosen from each burst) are uploaded, downsized to 1024-pixel JPEGs;
  - results are cached in `ai_cache.json`;
  - models differ in taste: treat scores as suggestions and make the final call yourself.

## Photo Edit (basic)

Switch to Photo Edit at the top right, or click “Edit” on a photo in the cull report to open that photo directly. The “Cull results” button opens the folder Photo Cull processed most recently.

**Adjustments**
- Basic: temperature, tint, exposure, contrast, highlights, shadows, whites, blacks, clarity, vibrance, saturation, black & white. Double-click a slider name to reset it.
- Curves: an RGB curve and separate red/green/blue curves. Click to add a point, drag to adjust, double-click to delete.
- **Preview**: resolution follows the pixels the image actually occupies on screen (Retina included) and re-renders when the window grows. Click the preview for 1:1 screen pixels (scrollable); click again or press Esc to fit. Hide the side panels with ◧ ◨ in the title bar or the `[` `]` keys; the choice is remembered. The file list has small and large thumbnails.
- Scopes: RGB histogram (with clipped highlight/shadow percentages) and a luma waveform or RGB parade.
- “Hold to compare” shows the photo before your adjustments.
- **Auto**: pick a metering style in the toolbar menu, then click “Auto”; you can fine-tune afterwards. The styles are rules of thumb based on each maker’s metering approach and look — **not copies of in-camera algorithms**. Metering runs on the editor’s real rendering, so RAW base exposure, JPEG brightness, current white balance and any chosen lens vignetting all count.

  Each style sets **exposure** first, then the **overall tone** (the Contrast / Blacks / Whites sliders):

  | Style | Metering | Tone |
  |---|---|---|
  | Evaluative (whole frame) | 6×4 zones, brightest and darkest 15% dropped, averaged to 18% gray | Neutral: moderate black and white points, contrast unchanged |
  | Leica classic center-weighted (default) | M-body style center weighting, about 1/4 EV below 18% gray; dark scenes only partly lifted | Deep: darkest blacks, base contrast +12, never lowers contrast |
  | Leica multi-field +0.3 EV | Leica multi-field metering plus +0.3 EV | Leica look, a little airier (base contrast +8) |
  | Fujifilm center-weighted | Wider center than Leica, 18% gray; stricter highlight protection | Soft: slightly lifted blacks, base contrast −5, gentle transitions |

  Tone rules: flat, gray images get a bit more contrast, already-punchy ones get less or slightly less; contrast is judged only on the part with content (pure black and white excluded), so low-key images on black aren’t misread. Blacks only go deeper (pure black never turns gray) and whites only brighter; low-key images aren’t forced to white, high-key ones aren’t forced to black. Every style pulls exposure back a little when too much clips. Your last style is remembered.

  HDR dynamic metering is not offered yet; it will arrive together with an HDR display mode.
- **Lens vignetting**: under Basic, pick a lens and strength to simulate the optical falloff of classic Leica M lenses before in-camera correction (circular, gradual, darkest in the corners):

  | Code | Lens | Light (≈f/5.6) | Medium (≈f/2.8) | Strong (wide open) |
  |---|---|---|---|---|
  | 11874 | Summilux-M 35mm f/1.4 ASPH. (1994, pre-FLE) | −0.6 EV | −1.2 EV | −2.2 EV (f/1.4) |
  | 11699 | APO-Summicron-M 35mm f/2 ASPH. | −0.8 EV | −1.3 EV | −1.9 EV (f/2) |
  | 11141 | APO-Summicron-M 50mm f/2 ASPH. | −0.5 EV | −0.9 EV | −1.4 EV (f/2) |
  | 11672 | Summicron-M 28mm f/2 ASPH. (2016, 2nd gen) | −1.0 EV | −1.6 EV | −2.3 EV (f/2) |

  Values are how many stops darker the corners are than the center. Leica publishes falloff only as charts, so these are estimates from each lens’s design (focal length, aperture, size); wide and fast lenses start falling off closer to the center. M bodies usually correct vignetting from the lens code; this simulates the uncorrected optics. It applies on export too; Basic → Reset clears it.
- Edits are saved automatically to `photo folder/photo-cull-report/edits/`; originals are never modified. “Copy settings / Paste settings” moves adjustments between photos.

**Formats**
- RAW is decoded by LibRaw (rawpy): Leica DNG (M11, Q3, SL…), Panasonic DNG/RW2, every Sony ARW (uncompressed, compressed, lossless), Fujifilm RAF (X-Trans and Bayer, including compressed RAF). DNG BaselineExposure is applied.
- JPG, HEIC, TIFF and PNG can be edited too.
- If a RAW can’t be decoded (e.g. a very new camera LibRaw doesn’t know yet), the embedded preview is used instead with a notice; quality and latitude are limited then.

**Style (LUT)**
- **Built-in LUTs**: `luts/` contains 39 LUTs: 20 Fujifilm GFX film simulations (named “camera + film simulation (sRGB)”, e.g. `GFX50S PROVIA (sRGB)`) and 19 Leica Looks (named “Leica Look + filter (sRGB)”, e.g. `Leica Look CHR (sRGB)`). **All use an sRGB working space**: apply them to images already in sRGB, not to Log or linear footage. They were collected from public sources; brand names only describe the style. They work on any computer you copy the folder to (Windows too), because edits store the built-in LUT’s name rather than a path on this machine.
- **Your own LUTs**: under Style (LUT), click “Add LUT…” and choose one or more `.cube` files. They are checked, then stored in `~/.photo-cull/luts` (kept across upgrades) and listed under “My LUTs”; hover an entry and click × to remove it.
- Search and click a LUT to apply it; the amount is adjustable. To use another library, click “Choose…” and pick a folder; “Built-in” switches back. An external library lists all its LUTs and presets.
- **Input space**: always treated as sRGB, matching the in-camera look of Sony “ST” / Leica “STD”.
  - JPEG: JPEGs shot in ST/STD match what the LUTs were designed for.
  - RAW: the editor’s standard rendering approximates that look before the LUT.
- Camera Log conversion (S-Log3 and others) is disabled for now (kept in code: `ENABLE_LOG` in `photo_cull/edit/logspace.py`).
- The editor can also decode Adobe color tables (RGBTable / LookTable) in XMP presets.

**Export**
- JPEG (adjustable quality, with sRGB profile and capture data) or 16-bit TIFF; full size, or long edge 6000 / 4000 / 2048.
- Exports go to a `photo-cull-export` subfolder next to the photo; the original is never overwritten.
- Export decodes at full resolution and renders in strips, so even 60-megapixel files don’t fill memory. Preview and export share one pipeline, and each strip is processed on several threads.

**Notes**
- Highlights and Shadows are global curves, not Lightroom-style local tone mapping, so large moves look different.
- Previews render in the background: a low-resolution image while dragging, full quality on release. Rendering uses all CPU cores; large JPEGs are decoded at reduced size for previews, and thumbnails use the small previews embedded in files (Sony JPEG and ARW both have them), so folders open quickly.

## For developers

The code separates front end and back end: the Python package `photo_cull/` handles photos and serves an API; the interface is HTML/CSS/JS in `web/`, and every piece of text lives in the `locales/` language packs. You can change the interface without touching Python, and the wording without touching code.

```
photo_cull/
  imaging/   reading images and EXIF (embedded RAW previews, HEIF, Fujifilm RAF), shared by cull and edit
  cull/      Photo Cull: analysis → grouping → scoring → ai/ review → xmp → organize → report; pipeline runs the stages, cli is the command line
  edit/      Photo Edit: color, lut3d / refs / styles (LUTs), presets, render, auto, session, export
  server/    local web server: settings, jobs, apps (registry of the two programs), pages, actions (report actions), handler (API), runtime (one window)
  i18n.py · paths.py · version.py · activation.py
cull.py · gui.py · desktop.py      entry points (command line, local web UI, Mac app)
tests/                              tests: python3 tests/run.py
```

- [Changelog](CHANGELOG.md) (Chinese)
- [Work log](docs/工作记录.md) and [1.7.1 structure before the refactor](docs/历史资料.md) (Chinese)

## License

Copyright © 2026 Hastin-moe. All rights reserved. From Pre-A on, Photo Tools is no longer open source and the source lives in a private repository; you may not copy, distribute or modify it without permission.

- Versions up to 1.9-beta that were published earlier remain under the MIT License they were released with.
- How it works (principles and flowcharts) and the plugin protocol are published in a separate repository, **photo-cull-public**.
- Film Lab (photo-cull-film) is a separate program, open source under GPLv3, connected through the plugin protocol.
- The LUTs in `luts/` were collected from public sources; see `luts/说明.txt`.
