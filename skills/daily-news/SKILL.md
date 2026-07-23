---
name: daily-news
description: Collect and summarize news into a dated Korean HTML briefing (English original headline + Korean translation + Korean summary). Two modes — DAILY (default) sweeps configured RSS feeds for today's top world stories across categories; TOPIC mode digs into one subject the user names (person, company, country, event) using RSS + web search over a recent window. Use when the user asks for a news briefing, daily news summary, "오늘 뉴스 정리", "세계 소식 요약", or asks to gather/summarize recent news about a specific topic, e.g. "트럼프 관련 최근 소식 정리해줘", "엔비디아 뉴스 모아줘", "이번 주 중동 상황 요약".
---

# Daily News Briefing

Build a dated news briefing. Output = one self-contained Korean HTML file.

## Inputs

- `feeds.txt` (next to this file) — RSS feed list. Format per line: `CATEGORY | Source Name | URL`. Lines starting `#` are comments.
- Optional user args: topic, category filter, item count, time window, output dir. Defaults below.

## Modes

Pick the mode from the request, then follow that mode's step list.

| | **DAILY** (default) | **TOPIC** |
|---|---|---|
| Trigger | No subject named — "오늘 뉴스 정리", `/daily-news` | A subject named — "트럼프 관련 최근 소식", "엔비디아 뉴스", "이번 주 우크라이나 상황" |
| Sources | `feeds.txt` RSS only | `feeds.txt` RSS **+ WebSearch** (+ WebFetch on promising articles) |
| Window | last ~36h | last ~7 days (override if user says "오늘"/"이번 주"/"한 달") |
| Sections | Fixed 4 categories | Sub-themes derived from what was actually found |
| Count | 10+ | As many as genuinely relevant (aim 6~15). Never pad. |
| Filename | `YYYY-MM-DD.html` | `YYYY-MM-DD-{topic}.html` |
| Extra | — | `핵심 요약` TL;DR block at top |

Ambiguous? Subject named wins — go TOPIC. If the user wants both ("오늘 뉴스 + 트럼프 따로"), write two files.

## Defaults

- Output dir (FIXED): `C:\Users\dladu\OneDrive\바탕 화면\daily new\`. Always write here regardless of the current working directory. Create the dir if missing.
- Filename: DAILY → `YYYY-MM-DD.html`. TOPIC → `YYYY-MM-DD-{topic}.html` (topic slug from the user's words, e.g. `2026-07-24-트럼프.html`; strip `\/:*?"<>|` and spaces→`-`). If the file exists, append `-2`, `-3`, … rather than overwriting.
- Output format: a single self-contained styled HTML file (open by double-click in browser). Use the HTML template below.
- Item count: DAILY 10+ across categories, skip empty categories. TOPIC as many as relevant.
- Freshness: DAILY ~36h. TOPIC ~7 days. If feed lacks dates, keep top items as-is.

## Steps — DAILY mode

1. **Read config.** Read `feeds.txt`. Group feeds by CATEGORY.
2. **Fetch feeds in parallel.** Use WebFetch on each URL. Prompt each fetch to return the top 5 items as `title | pubDate | link | one-line gist`. Some feeds fail (blocked/moved) — skip failures silently, note them at the end under "수집 실패".
3. **Dedupe.** Same story across sources → keep one, richest. Match on topic, not exact title.
4. **Rank & select.** Pick 10+ most significant stories. Bias toward: major geopolitical events, market-moving economy, big tech/science, major Korea news. Balance across categories — no single category dominates.
5. **Translate + summarize.** For each selected story write:
   - English original headline (verbatim from source; if source is Korean, put Korean headline here and skip the translation line).
   - Korean translation of the headline.
   - 2~3 line Korean summary (what happened, why it matters). No filler.
   - Source name + link.
6. **Write file.** Fill the HTML template below (one `<article class="card">` per story, grouped under each category `<section>`; drop empty categories). Write to the FIXED output dir `C:\Users\dladu\OneDrive\바탕 화면\daily new\YYYY-MM-DD.html` (create the dir if missing). Escape `&`, `<`, `>` in any text. Confirm path to user, show a short console preview (top 3 headlines).

## Steps — TOPIC mode

The user named a subject. Everything is about that subject; unrelated headlines are noise, drop them.

1. **Pin the topic.** Restate it in one line and settle the window before fetching (default last 7 days; "오늘"→24h, "이번 주"→7d, "최근 한 달"→30d). Resolve vague references to a concrete search term (e.g. "대통령" → who, which country).
2. **Build query set.** Write 3~6 search queries covering different angles, mixing Korean and English, e.g. for 트럼프: `Trump latest news`, `Trump tariff`, `Trump 발언`, `트럼프 한국`. Include the window in the query when it helps (`this week`).
3. **Fetch in parallel.**
   - WebSearch each query.
   - WebFetch each `feeds.txt` feed, but this time ask the fetch to return only items matching the topic (return "none" if the feed has none).
   - WebFetch 3~8 of the most substantive article URLs the search surfaced, for real detail beyond the snippet.
   Failures → skip silently, list under "수집 실패".
4. **Filter hard.** Keep only items where the topic is the actual subject, not a passing mention. Drop opinion columns, listicles, and anything outside the window. If a date is unclear, verify it or drop it.
5. **Dedupe & group.** Merge same-event coverage. Then group into 2~5 sub-themes that emerge from what was found — not the fixed 4 categories. Name them plainly (e.g. `관세·통상`, `사법 리스크`, `한반도 관련`, `발언·정치 동향`). Order sub-themes by importance.
6. **Write TL;DR.** 3~5 bullets in Korean: what moved on this topic in the window, the through-line, and anything imminent (scheduled ruling, vote, deadline). Only what the sources support.
7. **Translate + summarize** each story — same format as DAILY step 5.
8. **Write file.** Same HTML template, with these swaps: `<h1>` → `🔎 {topic} 최근 소식`; `.meta` → `{window} · 검색 {Q}건 + 피드 {N}개 · 선정 {M}건`; insert the `핵심 요약` block (template below) right after `</header>`; `<h2>` = sub-theme names (drop the fixed emoji set, use `📌` or a fitting emoji). Save as `YYYY-MM-DD-{topic}.html`. Confirm path, preview top 3.
9. **Nothing found?** Say so plainly and stop. Do not fill the gap with background knowledge or older articles — a topic can simply be quiet in the window.

## Output template (HTML)

Write the whole file as this self-contained HTML. Replace `{...}` placeholders. Repeat the `<section>` block per category (emoji: 🏛️ 정치·국제 / 💰 경제·시장 / 💻 기술·과학 / 🇰🇷 국내), and one `<article class="card">` per story. Item numbering is continuous across sections. If a story's source is Korean, put the Korean headline in `<h3>` and omit the `.trans` line.

```html
<!doctype html>
<html lang="ko">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>오늘의 세계 소식 — {DATE}</title>
<style>
  :root{--bg:#f5f6f8;--card:#fff;--txt:#1a1a1a;--sub:#5b6470;--line:#e4e7eb;--accent:#2f6fed;--chip:#eef2ff;--chiptx:#2f6fed}
  @media (prefers-color-scheme:dark){:root{--bg:#0f1216;--card:#1a1f27;--txt:#e8eaed;--sub:#9aa4b2;--line:#2a3039;--accent:#6ea0ff;--chip:#1e2a45;--chiptx:#9dbcff}}
  *{box-sizing:border-box}
  body{margin:0;background:var(--bg);color:var(--txt);font-family:"Pretendard","Malgun Gothic","Apple SD Gothic Neo",system-ui,sans-serif;line-height:1.6;-webkit-font-smoothing:antialiased}
  .wrap{max-width:760px;margin:0 auto;padding:32px 20px 64px}
  header h1{font-size:1.7rem;margin:0 0 6px;letter-spacing:-.02em}
  .meta{color:var(--sub);font-size:.9rem;margin-bottom:28px}
  h2{font-size:1.15rem;margin:34px 0 14px;padding-bottom:8px;border-bottom:2px solid var(--line);letter-spacing:-.01em}
  .card{background:var(--card);border:1px solid var(--line);border-radius:14px;padding:18px 20px;margin:12px 0;box-shadow:0 1px 2px rgba(0,0,0,.04)}
  .card h3{margin:0 0 4px;font-size:1.05rem;letter-spacing:-.01em}
  .card .num{color:var(--accent);font-weight:700;margin-right:6px}
  .trans{color:var(--txt);font-weight:600;margin:0 0 8px;font-size:.98rem}
  .card p.sum{margin:8px 0 12px;color:var(--txt)}
  .src{display:flex;align-items:center;gap:8px;font-size:.85rem;color:var(--sub);flex-wrap:wrap}
  .chip{background:var(--chip);color:var(--chiptx);padding:2px 10px;border-radius:999px;font-weight:600}
  .src a{color:var(--accent);text-decoration:none}
  .src a:hover{text-decoration:underline}
  footer{margin-top:36px;color:var(--sub);font-size:.82rem;border-top:1px solid var(--line);padding-top:16px}
  .tldr{background:var(--chip);border:1px solid var(--line);border-left:4px solid var(--accent);border-radius:12px;padding:16px 20px;margin:0 0 8px}
  .tldr h2{margin:0 0 8px;font-size:1rem;border:0;padding:0;color:var(--chiptx)}
  .tldr ul{margin:0;padding-left:20px}
  .tldr li{margin:4px 0}
</style>
</head>
<body>
<div class="wrap">
  <header>
    <h1>🌏 오늘의 세계 소식</h1>
    <div class="meta">{DATE} · 수집 소스 {N}개 피드 · 선정 {M}건</div>
  </header>

  <section>
    <h2>🏛️ 정치·국제</h2>
    <article class="card">
      <h3><span class="num">1.</span>{English original headline}</h3>
      <p class="trans">{한국어 번역 제목}</p>
      <p class="sum">{2~3줄 한국어 요약.}</p>
      <div class="src"><span class="chip">{Source}</span><a href="{link}" target="_blank" rel="noopener">원문 →</a></div>
    </article>
    <!-- more .card ... -->
  </section>

  <!-- repeat <section> for 💰 경제·시장 / 💻 기술·과학 / 🇰🇷 국내 -->

  <footer>수집 실패/제외: {failed feed names, or "없음"}</footer>
</div>
</body>
</html>
```

### TOPIC mode extra block

Insert directly after `</header>`, before the first `<section>`:

```html
  <div class="tldr">
    <h2>핵심 요약</h2>
    <ul>
      <li>{한 줄 요점}</li>
      <li>{한 줄 요점}</li>
      <li>{한 줄 요점}</li>
    </ul>
  </div>
```

## Notes

- Never invent facts or dates. Only summarize what the source content actually says. If unsure of a detail, omit it.
- Keep headlines verbatim; only the summary is your own wording.
- TOPIC mode: never answer from your own background knowledge of the subject — everything in the file must trace to a fetched source with a link. Model training data is stale by definition; the whole point is the fresh window.
- If the user wants a different language mix, count, window, or categories, honor that over defaults.
- To add/remove RSS sources, edit `feeds.txt` — no code change needed. (WebSearch in TOPIC mode is not limited to `feeds.txt`.)
