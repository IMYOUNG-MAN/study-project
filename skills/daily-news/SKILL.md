---
name: daily-news
description: Collect and summarize today's major world news from RSS feeds. Fetches configured feeds, dedupes, categorizes (World/Politics, Economy, Tech, Korea, Science/etc.), picks the top 10+ stories, and writes a dated Korean markdown briefing with English original headline + Korean translation + Korean summary. Use when the user asks for a news briefing, daily news summary, "오늘 뉴스 정리", "세계 소식 요약", or similar.
---

# Daily News Briefing

Build a dated news briefing from RSS feeds. Output = one Korean markdown file per day.

## Inputs

- `feeds.txt` (next to this file) — RSS feed list. Format per line: `CATEGORY | Source Name | URL`. Lines starting `#` are comments.
- Optional user args: category filter, item count, output dir. Defaults below.

## Defaults

- Output dir (FIXED): `C:\Users\dladu\OneDrive\바탕 화면\daily new\`. Filename: `YYYY-MM-DD.html` (use today's date). Always write here regardless of the current working directory. Create the dir if missing.
- Output format: a single self-contained styled HTML file (open by double-click in browser). Use the HTML template below.
- Item count: 10+ total across categories. Skip a category if no fresh items.
- Freshness: prefer items with pubDate within last ~36h. If feed lacks dates, keep top items as-is.

## Steps

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

## Notes

- Never invent facts or dates. Only summarize what the feed content actually says. If unsure of a detail, omit it.
- Keep headlines verbatim; only the summary is your own wording.
- If the user wants a different language mix, count, or categories, honor that over defaults.
- To add/remove sources, edit `feeds.txt` — no code change needed.
