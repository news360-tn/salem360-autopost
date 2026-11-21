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
| Priority | power cut (TNEB) > crime > accident > civic > the rest |
| Never miss news | Everything eligible is queued and written; duplicates and other-district news are skipped; nothing eligible is dropped |
| Daily cap | 50 reels a day (Instagram's limit) (`DAILY_CAP` in post.yml, also in the collector / writer); urgent news first |
| Posting time | No fixed times (owner 29 Oct): every reel posts the moment it is made (`poster/config.json` `prime_time.enabled: false`) |
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
| 2026-11-21 | `f764b84` | Keep 20 of Instagram's 50 daily posts for news: stories stop at 30 (news was waiting at 48/50) |
| 2026-11-21 | `2ea4b55` | Stop posting bursts: 10 min between news posts; an Instagram block pauses the page 6 h (news waits) instead of retrying every run |
| 2026-11-21 | `2a7775a` | rasipalan posted |
| 2026-11-20 | `7d4cc80` | pick: written and posted |
| 2026-11-20 | `963c259` | pick: picked news saved |
| 2026-11-20 | `61b2d14` | pick: list sent |
| 2026-11-20 | `82beacc` | powercut: notices written |
| 2026-11-20 | `97cb900` | pick: written and posted |
| 2026-11-20 | `a9b9aea` | pick: picked news saved |
| 2026-11-20 | `3c16314` | pick: picked news saved |
| 2026-11-20 | `4176cc1` | pick: list sent |
| 2026-11-20 | `9d1699d` | Facebook first comment = the same source comment as Instagram (link into the text if Facebook refuses the comment) |
| 2026-11-20 | `5e707b2` | pick: written and posted |
| 2026-11-20 | `4f4473b` | pick: picked news saved |
| 2026-11-20 | `6debcf3` | pick: list sent |
| 2026-11-20 | `3222ab2` | Facebook: up to 18 news a day per page (45 min apart), article link in the first comment, 3 hashtags |
| 2026-11-20 | `3a039e0` | Daily rasipalan as a reel in the 360 gold-reel pattern (music only); carousel only if the reel fails |
| 2026-11-20 | `774bcd1` | Reels without a picture: a designed background by news type, never the page logo |
| 2026-11-20 | `a85f216` | stories shared |
| 2026-11-20 | `ac8eaaf` | stories shared |
| 2026-11-20 | `a8305ca` | Every news reel goes to the story again (stories_urgent_only false) |
| 2026-11-20 | `a6e231f` | powercut: notices written |
| 2026-11-20 | `217c08c` | powercut: notices written |
| 2026-11-20 | `45f66ce` | pick: written and posted |
| 2026-11-20 | `e459791` | pick: picked news saved |
| 2026-11-20 | `ba0fbea` | pick: picked news saved |
| 2026-11-20 | `5ff7740` | pick: list sent |
| 2026-11-20 | `335339c` | Promotion ads (owner-approved only, bottom of the reel during the news) and twice-a-day YouTube details e-mail |
| 2026-11-20 | `454c27a` | pick: written and posted |
| 2026-11-20 | `5dce39c` | Fix: reels again made with reel4 (config reel_design back to v4) |
<!-- recent-changes:end -->
