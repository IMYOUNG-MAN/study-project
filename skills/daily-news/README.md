# 📰 daily-news

> RSS 피드에서 오늘의 주요 세계 소식을 모아 **영어 원문 + 한국어 번역 + 요약**으로 정리하고, 날짜별 HTML 브리핑 파일로 저장하는 Claude Code 스킬.

---

## 설치

Claude Code 개인 스킬 폴더에 `daily-news` 디렉터리를 넣으면 끝. 재시작 없이 바로 인식됨.

```bash
# 이 저장소 기준
cp -r skills/daily-news ~/.claude/skills/
```

폴더 구성:

```
~/.claude/skills/daily-news/
├── SKILL.md      # 스킬 본체 (지침 + HTML 템플릿)
├── feeds.txt     # RSS 피드 목록 (여기만 고치면 소스 추가/삭제)
└── README.md
```

설치 확인: Claude Code에서 `/daily-news` 자동완성에 뜨면 성공.

---

## 사용법 (트리거 예시)

슬래시 커맨드로 직접 실행:

```
/daily-news
```

또는 자연어로 (스킬 설명에 매칭돼 자동 발동):

- `오늘 뉴스 정리해줘`
- `세계 소식 요약해줘`
- `daily news briefing`
- `오늘의 주요 소식 브리핑`

옵션도 자연어로 얹을 수 있음:

- `오늘 뉴스 정리하되 기술 뉴스만`
- `오늘 뉴스 20건으로 정리해줘`

---

## 동작

1. 오늘 날짜 확인
2. `feeds.txt`의 RSS 피드 병렬 수집
3. 같은 사건 중복 제거
4. 카테고리별(정치·국제 / 경제·시장 / 기술·과학 / 국내) 상위 10건+ 선별
5. 각 건: 영어 원문 헤드라인 + 한국어 번역 + 2~3줄 한국어 요약 + 출처 링크
6. `YYYY-MM-DD.html` 파일로 저장

**저장 위치(고정):** `C:\Users\dladu\OneDrive\바탕 화면\daily new\YYYY-MM-DD.html`
(다른 경로로 바꾸려면 `SKILL.md`의 `Output dir (FIXED)` 한 줄 수정.)

---

## 실행 결과 예시

더블클릭하면 브라우저에서 열리는 카드형 HTML. 다크모드 자동 대응, "원문 →" 링크 클릭 가능.

![실행 결과 예시](example.png)

> 위 스크린샷은 `2026-07-24` 브리핑 (12개 피드 · 22건).

---

## 소스 커스터마이징

`feeds.txt`에 한 줄씩 추가/삭제. 형식:

```
CATEGORY | Source Name | URL
```

예:

```
정치·국제 | BBC World | http://feeds.bbci.co.uk/news/world/rss.xml
국내      | 경향신문 정치 | https://www.khan.co.kr/rss/rssdata/politic_news.xml
```

**작동 확인된 피드:** BBC(World/Business/Technology), NPR World, CBS World, Al Jazeera, CNBC, 한국경제(정치/경제), 경향신문(정치/경제/전체)

**WebFetch가 차단하는 피드(추가 불가):** NYT, The Guardian, AP, Ars Technica, 한겨레, 연합뉴스, 동아, 매일경제, JTBC, 노컷뉴스, 한국일보, 오마이뉴스
(`feeds.txt` 상단 주석에도 목록 있음.)

---

## 참고

- 팩트는 피드 내용만 사용 — 헤드라인은 원문 그대로, 요약만 자체 작성.
- 매일 자동 실행은 미지원(클라우드 스케줄은 별도 설정 필요). 현재는 직접 `/daily-news` 실행 방식.
