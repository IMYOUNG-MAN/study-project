# study-project

Claude Code 스킬을 만들어 두는 저장소. 플러그인 마켓플레이스로도 동작함.

## 설치

Claude Code에서:

```
/plugin marketplace add IMYOUNG-MAN/study-project
```

그 다음 원하는 플러그인 설치:

```
/plugin install daily-news@study-project
```

업데이트는 `/plugin marketplace update study-project`.

## 수록 플러그인

| 이름 | 설명 |
|---|---|
| [`daily-news`](skills/daily-news/) | 뉴스를 모아 **영어 원문 + 한국어 번역 + 요약** HTML 브리핑으로 저장. DAILY 모드는 RSS로 오늘의 세계 소식 전반, TOPIC 모드는 특정 주제를 RSS + 웹 검색으로 파고듦. |

각 플러그인 상세 사용법은 위 링크의 README 참고.

## 구조

```
study-project/
├── .claude-plugin/
│   ├── marketplace.json   # 마켓플레이스 카탈로그
│   └── plugin.json        # 플러그인 매니페스트
└── skills/
    └── daily-news/        # 스킬 본체
```
