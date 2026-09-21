# wiiiv App Templates

Pre-built interactive HTML apps for [wiiiv](https://wiiiv.ai).

## Usage

Templates are served via GitHub Pages:

```
https://wiiiv-io.github.io/wiiiv-templates/{template-id}.html
```

Template index:
```
https://wiiiv-io.github.io/wiiiv-templates/index.json
```

## Templates

| ID | Category |
|----|----------|
| tetris | game |
| snake | game |
| 2048 | game |
| minesweeper | game |
| breakout | game |
| flappy-bird | game |
| gomoku | game |
| galaga | game |
| memory | game |
| freecell | game |
| calculator | tool |
| pomodoro | tool |
| paint | tool |
| typing-practice | tool |
| typing-test | tool |
| analog-clock | tool |
| color-palette | tool |
| piano | music |
| drum-machine | music |
| mandelbrot | visualization |
| sorting-viz | visualization |
| physics | visualization |
| conway | visualization |

---

*wiiiv inc.*

## Adding a template (no server/desktop build needed)

1. Create `{id}.html` — **self-contained** (no external scripts/CSS), dark theme (`#0d0d1a` bg, `#7c5cfc` accent), `const LANG = 'ko';` + a `T` dictionary with `ko`/`en` (see `memory.html`). The server rewrites `LANG` to the session language.
2. Add an entry to `index.json`. Put **every spelling you can think of** in `keywords` (Korean spacing/typo variants + English), e.g. `"프리셀", "프리 셀", "프리쉘", "프레셀", "freecell", "free cell"`. Matching is lowercase `contains()` against the registered strings only — an unregistered spelling is simply not recognised. Avoid short generic words (`비트`, `시계`) that appear inside unrelated requests.
3. Add a row to the table above and `git push origin main`. GitHub Pages serves it in ~1–2 min.
4. Optionally copy the two files into `wiiiv/wiiiv-backend/wiiiv-server/src/main/resources/wiiivapp/` (CDN-outage fallback; picked up by the next regular build — never trigger a build for it).

How it is served (wiiiv build 4691+): `ConversationalGovernor.chat()` checks the catalog **before any LLM call** (`[WIIIVAPP-INTERCEPT]`): keyword match + creation intent (만들어줘/보여줘/play/create…) and no modification intent (속도/색상/faster…) → immediate `wiiiv://template/{id}`; the frontend fetches `{id}.html` from this CDN. Modification requests ("테트리스 속도 빠르게") still go through the LLM sandbox edit path.

Why it "doesn't show up" — always one of these, never a build problem:
- Servers refresh `index.json` every **10 minutes** → wait, then check `GET /api/v2/wiiivapp/templates`.
- The spelling the user typed is not in `keywords` → add it.
- A fix to the HTML isn't visible → browser cached it (`Cache-Control: max-age=600`); wait 10 min or reload with DevTools "Disable cache".

Verify without the UI (desktop backend, dev auto-login):
```bash
B=http://127.0.0.1:8555/api/v2; TOK=$(curl -s $B/auth/auto-login | jq -r .data.accessToken)
SID=$(curl -s -X POST $B/sessions -H "Authorization: Bearer $TOK" -H "Content-Type: application/json" -d '{}' | jq -r .data.sessionId)
curl -s -X POST $B/sessions/$SID/chat -H "Authorization: Bearer $TOK" -H "Content-Type: application/json" -d '{"message":"프리셀 만들어줘"}' | grep '^data: {"action'
# expect: "action":"REPLY" with ```wiiivapp / wiiiv://template/freecell
```
Full write-up: `wiiiv/docs/wiiivapp-template-architecture.md` §0.
