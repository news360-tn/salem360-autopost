# salem360-autopost - project context

Automatic Salem-district news reels for the Instagram page **சேலம் 360** (`@salem360.official`, placeholder; hashtag `#Salem360`;
`post_to` value in the Sheet: `Salem 360`). A separate, fully independent copy of the Namakkal project `news360-autopost`:
own GitHub repository, own Google account / Sheet / Apps Script, own Instagram token, own Cloudinary account and own free API keys.
Namakkal 360 / Rasipuram 360 are never touched. Same file structure and function names as Namakkal, so Namakkal fixes can be
ported by diff. Everything runs on GitHub Actions; the Google Sheet (through the Apps Script bridge) is the working store.

**Status: live** (the `LIVE` file is present and posting runs on GitHub Actions; checked 19 Nov 2026). News is collected by `pick.yml`.
Note: this file was first copied from Coimbatore 360 - where a line below still names Coimbatore places, sources or schedules,
the code and workflow files of this repo are what actually runs.

## How it works (end to end)

```
every 2 h    research_free.yml  collect news (newspaper sites incl. Dinamalar Coimbatore page, Google/Bing News, Coimbatore searches)
                                 -> open + check each article -> only Coimbatore-district news of today/yesterday
                                 -> queue by priority (research_free/archive/queue.json)
every 20 min write_stories.yml  write each queued news by priority (free LLM chain only) -> ONE AI image of the main scene
                                 (Cloudflare FLUX schnell -> NVIDIA FLUX -> paid FAL only if both fail) -> story saved in the Sheet
                                 -> starts posting at once
every 5 min  post.yml           checks (Coimbatore only, not stale, no duplicate, daily cap 50; posts at once) -> reel (reel/reel4.py,
                                 Valluvar voice) -> Instagram (one page, no collab)
daily        insights.yml       Instagram insights -> learned topic weights (writer priority)
```

## Rules the owner decided (keep them)

| Area | Rule |
|---|---|
| News scope | Only Coimbatore-district news, published today or yesterday (Coimbatore city, Pollachi, Mettupalayam, Sulur, Annur, Valparai, Kinathukadavu, Madukkarai, Perur, Thondamuthur, Anaimalai, Karamadai ...). Tiruppur, Nilgiris, Erode, Palakkad and every other district are not ours, even when close. "Avinashi Road" is a Coimbatore city road: a headline that also names a Coimbatore place is ours |
| One page | Only சேலம் 360. No second page, no collaborator, no Rasipuram routing |
| Priority | owner 27 Nov: by this page's Instagram views per topic (research_free/learned.json); urgent = an upcoming power cut (first) or a topic whose reels often pass 10,000 views (1 in 5) |
| Never miss news | Everything eligible is queued and written; duplicates and other-district news are skipped; nothing eligible is dropped |
| Daily cap | owner 27 Nov: 30 news a day per page on Instagram, Facebook and YouTube each (IST day; rasipalan / gold rate extra); after 30 only urgent news |
| Posting time | owner 27 Nov: 06:00-22:00 IST every reel posts as soon as it is ready; 22:00-06:00 only urgent news. News is collected every 2 hours |
| Errors and blocks | owner 27 Nov: after an error 5 tries 10 minutes apart, then 1 try 1 hour later, then Hold - never retried at once; a temporary block waits the time the platform states, else 2 hours. No manual work for the owner (config.json owner_emails false) |
| Format | Reel only (no carousel) |
| Voice | **Valluvar only** (free edge-tts `ta-IN-ValluvarNeural`). Google voices are never used, even if a Google key exists |
| Follow line | Spoken: மேலும் செய்திகளுக்கு கோயம்புத்தூர் த்ரீ சிக்ஸ்டி பக்கத்தை பின்தொடருங்கள். Caption: கோயம்புத்தூர் செய்திகளைத் தவறவிடாமல் இருக்க Save செய்து Follow செய்யுங்கள் 📌 |
| Images | ONE new photo-realistic AI image per reel (main scene of THIS news, no text/logos); the reel uses different camera moves / crops of it. Free first: Cloudflare FLUX schnell -> NVIDIA FLUX; paid FAL `flux/schnell` -> `flux/dev` only as the last fallback. Picture check (free Gemini) as in Namakkal |
| Writing | Free LLM chain only; Claude backup OFF (`CLAUDE_MAX_CALLS=0`, no `ANTHROPIC_API_KEY`) |
| Reel language | Formal news Tamil, no slang; everyday Tamil Nadu words in voice, subtitles and cards (பஸ், டூவீலர், போலீசார், கலெக்டர் ...); dates always with the month |
| Website | Off for now (later). `poster/website.py` does nothing without `WEBSITE_INGEST_KEY`; workflows do not pass it |
| Separation | Own accounts and keys, so Namakkal's free API limits are never used; bridge e-mails tagged `SALEM360` |
| Privacy | Personal project; bot commits as `news360-bot`; no office account, office email or Claude lines in commits |

## Main files

| Path | What |
|---|---|
| `research_free/collect.py`, `allnews.py`, `sources.py` | collection, Coimbatore district checks, queue, writer, image |
| `research_free/rules.md`, `examples.json`, `names.py` | writing rules and spoken-word fixes |
| `poster/post.py` | posting checks, prime time, daily cap, Instagram publishing |
| `poster/freshness.py` | stale / duplicate / power-cut timing rules |
| `poster/config.json` | voice (Valluvar), prime time, design switches |
| `reel/reel4.py` | reel renderer (one image, camera moves, voice, opening, dates) |
| `apps_script/Code.gs` | Google Sheet bridge for the Salem 360 account (set the `..._HERE` values; redeploy after changes) |
| `tests/test_regressions.py` | regression tests (run before every push) |

## Sheet bridge (apps_script/Code.gs)

- Page key `salem`, label `சேலம் 360`, `post_to` `Salem 360`, token property `IG_TOKEN_SALEM` (refreshed weekly).
- Columns: same 31 as Namakkal; column 25 `spare_link` (unused), column 26 `coimbatore_link` (carousel link, unused while reel only),
  27 `reel_link`.
- GET actions: `queue`, `posted_rows`, `token`, `voice`, `recent`, `ping`, `autoPending`.
  POST actions: `posted`, `error`, `addStories`, `archiveNews`, `reoffer`, `setImages`, `repost`.
- E-mail subjects: `SALEM360 STORY`, `SALEM360 REEL`, `[SALEM360 IMAGES NEEDED]` ...

## Secrets (GitHub -> Settings -> Secrets -> Actions; values never in the repo)

BRIDGE_URL, BRIDGE_KEY, GEMINI_API_KEY, GROQ_API_KEY, NVIDIA_API_KEY, CLOUDFLARE_API_TOKEN, CLOUDFLARE_ACCOUNT_ID,
CLOUDINARY_CLOUD, CLOUDINARY_PRESET; optional MISTRAL_API_KEY, OPENROUTER_API_KEY, GH_MODELS_TOKEN, FAL_KEY (paid image fallback).
Not used: ANTHROPIC_API_KEY, GOOGLE_TTS_API_KEY, WEBSITE_INGEST_KEY. Local copies live only in `.env` (git-ignored).

## Run and check

```
python3 -m unittest discover -s tests        # regression tests (also run by tests.yml on every code push)
python3 tools/repost.py RN-158               # (Actions: repost) post a reel again after it was removed from Instagram
```

## Recent changes
<!-- recent-changes:start (updated automatically by .github/workflows/context.yml - do not edit by hand) -->
| Date | Commit | Change |
|---|---|---|
| 2026-11-28 | `0b99ea9` | Writer hook (first 3 s): place first + strong event word + one hard detail; நாளை/இன்று first; schemes open with the amount / last date; caption first line 7-12 words (data: 2,686 reels) |
| 2026-11-28 | `e78509f` | stories shared / Instagram pause |
| 2026-11-28 | `f6fe6ae` | pick: written and posted |
| 2026-11-28 | `6f57a96` | facebook RN-353 |
| 2026-11-28 | `3fec50e` | posted RN-353 |
| 2026-11-28 | `47ce9da` | stories shared / Instagram pause |
| 2026-11-28 | `4583c9e` | stories shared / Instagram pause |
| 2026-11-28 | `af24bbf` | facebook RN-352 |
| 2026-11-28 | `85313ed` | posted RN-352 |
| 2026-11-28 | `745c0eb` | stories shared / Instagram pause |
| 2026-11-28 | `e990dc0` | facebook RN-351 |
| 2026-11-28 | `9ec8900` | posted RN-351 |
| 2026-11-28 | `0be0e28` | pick: picked news saved |
| 2026-11-28 | `09341c9` | stories shared / Instagram pause |
| 2026-11-28 | `0123b29` | pick: picked news saved |
| 2026-11-28 | `b73aba4` | pick: list sent |
| 2026-11-28 | `1e02505` | stories shared / Instagram pause |
| 2026-11-28 | `5801979` | stories shared / Instagram pause |
| 2026-11-28 | `db927d8` | stories shared / Instagram pause |
| 2026-11-28 | `81cb8fd` | stories shared / Instagram pause |
| 2026-11-28 | `f2c745d` | stories shared / Instagram pause |
| 2026-11-28 | `117aeea` | stories shared / Instagram pause |
| 2026-11-28 | `0156857` | stories shared / Instagram pause |
| 2026-11-28 | `bd00797` | stories shared / Instagram pause |
| 2026-11-28 | `754faa0` | stories shared / Instagram pause |
| 2026-11-28 | `7e639b1` | stories shared / Instagram pause |
| 2026-11-28 | `9d95995` | stories shared / Instagram pause |
| 2026-11-28 | `2051ef6` | stories shared / Instagram pause |
| 2026-11-28 | `181ad5d` | stories shared / Instagram pause |
| 2026-11-28 | `697b9aa` | pick: written and posted |
<!-- recent-changes:end -->
