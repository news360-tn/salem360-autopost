# Salem 360 – Instagram news page (NOT LIVE)

Copy of the Coimbatore 360 code, started on 12 Nov 2026. This repo is **not live yet**:
- no `LIVE` file (scheduled posting is switched off by the code),
- no secrets set, and all workflows disabled.

## Already changed for Salem 360
- page id `salem`, hashtag `#Salem360`, label `சேலம் 360`, Instagram handle in `poster/comments.py`.
- post stories, first comments and source comments switched off in `poster/config.json`.

## Still to change before going live (Coimbatore-specific)
- Districts and place names: `poster/post.py` `DISTRICTS`, `research_free/allnews.py`, `research_free/names.py`.
- News sources: `research_free/sources.json` (Salem district pages of Dinamalar / Dinamani / Daily Thanthi).
- Power-cut sources and queries: `research_free/powercut.py` (`QUERIES`, `SITES`, `DISTRICT_URL`).
- Hashtags and search line: `poster/config.json` (`reach`, `district_tag`, `broad_tag`, `search_suffix`).
- Logo and footer: `kit/` (currently the Coimbatore logo).
- Learned insights: start empty (no `learned.json` yet).
- Apps Script bridge, Google Sheet and Instagram/Facebook tokens: new per page.
