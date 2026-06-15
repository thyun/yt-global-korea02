# Copilot Instructions — 외국인의 한국 생활 YouTube 에이전트

## 프로젝트 개요

이 프로젝트는 **"외국인의 한국 생활"** 주제의 YouTube 동영상을 자동으로 기획·제작하는 **멀티스테이지 AI 에이전트 파이프라인**입니다.

- **레퍼런스 채널/영상**: https://www.youtube.com/watch?v=H2KRGy6O0_0
- **타겟 시청자**: 30대 이상 한국인
- **콘텐츠 방향**: 외국인이 경험하는 한국 문화·생활·사회 이슈를 한국인 시청자 시각에서 흥미롭게 재구성
- **제작 방식**: 기존 YouTube 원본 소스 영상을 선정한 뒤, 편집·재구성하여 새 영상을 제작 (원본 촬영 없음)

---

## 에이전트 파이프라인 구조

각 단계는 독립된 에이전트 역할을 가지며, 이전 단계의 산출물을 입력으로 받습니다.

각 에피소드의 모든 산출물은 `episodes/{YYYYMMDD_episode-slug}/` 하위에 단계별로 저장됩니다.

```
Stage 1: Trend Research Agent    → episodes/{slug}/01_trend/
Stage 2: Topic Planning Agent    → episodes/{slug}/02_topic/
Stage 3: Script Writing Agent    → episodes/{slug}/03_script/
Stage 4: Visual Planning Agent   → episodes/{slug}/04_visual/
Stage 5: Asset Collection Agent  → episodes/{slug}/05_assets/
Stage 6: Video Assembly Agent    → episodes/{slug}/06_video/
Stage 7: Publishing Agent        → episodes/{slug}/07_publish/
```

### 단계별 입출력 정의

| 단계 | 에이전트 역할 | 입력 | 출력 경로 (`episodes/{slug}/`) |
|------|-------------|------|------|
| 01_trend | 트렌드 리서처 | 검색 키워드 + 웹/YouTube 탐색 | `01_trend/trends.json` (트렌드 테마 + **소스 영상 후보 목록**) — **검색 시작 전 키워드를 사용자에게 제안하고 확인 받은 후 진행** |
| 02_topic | 소스 선정 & 기획자 | `01_trend/trends.json` | `02_topic/topic.md` + **`02_topic/transcript.json`** (소스 영상 트랜스크립트) |
| 03_script | 스크립트 작가 | `02_topic/topic.md` + `02_topic/transcript.json` | `03_script/script.md` (편집 대본) + `03_script/narration.md` (보이스오버 녹음용) |
| 04_visual | 영상 기획자 | `03_script/script.md` + `02_topic/transcript.json` | `04_visual/storyboard.md` (컷 구성 + **실제 타임코드 매핑**) |
| 05_assets | 에셋 수집자 | `04_visual/storyboard.md` | `05_assets/` (소스 영상 다운로드 + 자막·BGM 등 추가 에셋) |
| 06_video | 영상 편집자 | `05_assets/` + `04_visual/storyboard.md` | `06_video/output.mp4` |
| 07_publish | 퍼블리셔 | `06_video/output.mp4` + `02_topic/topic.md` | `07_publish/metadata.json` |

### 01단계 검색 키워드 확인 규칙 (필수)

1단계(트렌드 리서치)를 시작할 때 **반드시 아래 순서를 지킨다**:

1. 프로젝트 방향(`콘텐츠 방향`, 이전 에피소드 주제 등)을 바탕으로 검색 키워드 후보 **3~5개**를 사용자에게 먼저 제안한다.
2. 사용자가 키워드를 **확인(승인 또는 수정)** 한 이후에만 실제 웹 검색을 수행한다.
3. 검색은 사용자가 별도로 기간을 지정하지 않는 한 **최근 2년 이내** 영상을 기준으로 탐색한다.
4. 확인된 최종 키워드를 `trends.json`의 `search_keywords` 필드에 기록한다.

---

### 트랜스크립트 다운로드 규칙 (02단계 필수)

- 소스 영상 선정 후 **반드시 `transcript.json`을 `02_topic/` 에 저장**
- 다운로드 도구: `youtube-transcript-api` Python 패키지 사용
  ```bash
  python3 -c "
  from youtube_transcript_api import YouTubeTranscriptApi
  import json
  video_id = 'VIDEO_ID'
  transcript = YouTubeTranscriptApi.get_transcript(video_id, languages=['en','ko'])
  print(json.dumps(transcript, ensure_ascii=False, indent=2))
  " > 02_topic/transcript.json
  ```
- 영어 자막 우선(`languages=['en','ko']`), 없으면 자동 생성 자막 시도
- `transcript.json` 형식: `[{"text": "...", "start": 0.0, "duration": 2.5}, ...]`
- **03_script, 04_visual 작성 시 transcript.json의 실제 발언 내용과 타임코드를 기준으로 소스 클립 구간 결정**

### 소스 영상 선정 기준 (01~02단계 핵심)

- **저작권**: Creative Commons(CC) 라이선스 또는 명시적 재사용 허가 영상 우선
- **조회수**: 1만 뷰 이상 (검증된 주제 관심도)
- **영상 품질**: 720p 이상, 영어 또는 한국어 자막 포함
- **내용 적합성**: 외국인 화자, 한국 생활 주제, 타겟 시청자(30대 이상 한국인) 공감 가능
- `trends.json` 의 `source_candidates` 배열에 URL·채널명·조회수·라이선스·사용 가능 구간을 반드시 기록

---

## 디렉토리 구조

```
yt-global-korea02/
├── episodes/
│   └── {YYYYMMDD_episode-slug}/   # 에피소드별 독립 디렉토리
│       ├── 01_trend/              # trends.json
│       ├── 02_topic/              # topic.md
│       ├── 03_script/             # script.md
│       ├── 04_visual/             # storyboard.md
│       ├── 05_assets/             # 이미지·클립·음악, license_log.md
│       ├── 06_video/              # output.mp4
│       └── 07_publish/            # metadata.json
├── tools/                         # 각 단계를 자동화하는 Python 스크립트
│   ├── trend_search.py            # YouTube API 트렌드 수집
│   ├── topic_planner.py           # 주제 기획 자동화
│   └── ...
├── config/
│   └── settings.yaml              # API 키 경로, 언어 설정 등
└── README.md
```

---

## 핵심 컨벤션

### 언어 및 톤
- 스크립트·기획 산출물은 **한국어**로 작성
- 시청자 대상이 30대 이상 한국인이므로, 어조는 **친근하되 신뢰감 있는 정보 전달형**
- 자극적·클릭베이트 성격은 지양하고, 외국인 관점의 진정성 있는 스토리 중심

### 나레이션 작성 규칙

나레이션 스타일 레퍼런스: **`templates/narration.md`** — 반드시 이 파일을 먼저 읽고 스타일을 맞출 것.

핵심 원칙:
- **끊김 없는 흐름**: 구간 헤더 없이 하나의 연속된 산문으로 작성. 단락 구분은 자연스러운 호흡 단위로만 사용
- **스토리텔링 구조**: 트렌드/사건 훅 → 특정 인물 도입 → 인물을 따라가며 사건 전개 → 감정적 마무리 → CTA
- **호기심 연결 어미**: 문단 말미에 `~는데요`, `~인데요`, `~고 맙니다` 등으로 다음 문장이 궁금하게 끝맺기
- **전환 접속어**: `그런데`, `하지만`, `결국`, `그렇게`, `그리고 드디어` 등으로 흐름을 이어 독자를 붙잡기
- **인물 대사 자연 삽입**: 외국인 발언은 따옴표로 나레이션 안에 녹여 쓰고 별도 태그 없이 처리
- **군더더기 제거**: 조사·어미 중복, 불필요한 수식어 제거. 한 문장에 하나의 정보만
- `narration.md`는 **녹음 텍스트만** 수록 — `[나레이션]`, `[BGM]` 등 편집 태그 일체 포함 금지

### 프로그램 작성 규칙
- YouTube 검색·크롤링 등 외부 데이터 수집은 `tools/` 디렉토리에 **Python 스크립트**로 작성
- API 키 등 민감 정보는 `.env` 파일에서 로드하고 절대 소스코드에 하드코딩 금지
- 각 스크립트는 단독 실행 가능하도록 `if __name__ == "__main__":` 진입점 포함

### 산출물 관리
- 모든 단계의 산출물은 **`episodes/{YYYYMMDD_episode-slug}/`** 하위 해당 단계 디렉토리에 저장
- 에피소드 슬러그 예시: `20260615_foreigner-subway-experience`
- 영상 에셋은 라이선스 정보를 함께 기록 (`05_assets/license_log.md`)

### 영상 길이 기준
- **목표 길이: 8분 이상** (권장 사항이며 엄격히 지킬 필요 없음)
- `02_topic/topic.md` 영상 구성 개요 작성 시 구간 합계를 8분 기준으로 설계

### Copilot 에이전트 역할
Copilot이 직접 수행하는 단계:
- **02 (주제 기획)**, **03 (스크립트 작성)**, **04 (스토리보드)** — LLM이 직접 콘텐츠 생성

프로그램으로 자동화해야 하는 단계:
- **01 (트렌드 리서치)** — YouTube Data API v3 또는 웹 검색 필요
- **05 (에셋 수집)** — 저작권 무료 소스(Pixabay, Pexels, Free Music Archive) API 사용
- **07 (퍼블리시)** — YouTube Data API 업로드

---

## 참고 외부 서비스 및 API

| 서비스 | 용도 | 비고 |
|--------|------|------|
| YouTube Data API v3 | 트렌드 검색, 영상 업로드 | Google Cloud Console에서 키 발급 |
| Pixabay / Pexels API | 무료 이미지·영상 에셋 | 상업적 사용 가능 라이선스 확인 필요 |
| Free Music Archive | 배경음악 | CC 라이선스 확인 |
| OpenAI / Copilot API | 스크립트 생성, 번역 | 필요 시 활용 |
