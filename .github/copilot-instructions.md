# Copilot Instructions — 외국인의 한국 생활 YouTube 에이전트

## 프로젝트 개요

이 프로젝트는 **"외국인의 한국 생활"** 주제의 YouTube 동영상을 자동으로 기획·제작하는 **멀티스테이지 AI 에이전트 파이프라인**입니다.

- **레퍼런스 채널/영상**: https://www.youtube.com/watch?v=H2KRGy6O0_0
- **타겟 시청자**: 30대 이상 한국인
- **콘텐츠 방향**: 외국인이 경험하는 한국 문화·생활·사회 이슈를 한국인 시청자 시각에서 흥미롭게 재구성

---

## 에이전트 파이프라인 구조

각 단계는 독립된 에이전트 역할을 가지며, 이전 단계의 산출물을 입력으로 받습니다.

```
Stage 1: Trend Research Agent    → /pipeline/01_trend/
Stage 2: Topic Planning Agent    → /pipeline/02_topic/
Stage 3: Script Writing Agent    → /pipeline/03_script/
Stage 4: Visual Planning Agent   → /pipeline/04_visual/
Stage 5: Asset Collection Agent  → /pipeline/05_assets/
Stage 6: Video Assembly Agent    → /pipeline/06_video/
Stage 7: Publishing Agent        → /pipeline/07_publish/
```

### 단계별 입출력 정의

| 단계 | 에이전트 역할 | 입력 | 출력 |
|------|-------------|------|------|
| 01_trend | 트렌드 리서처 | YouTube API / 검색 키워드 | `trends.json` (트렌드 키워드, 인기 영상 목록) |
| 02_topic | 주제 기획자 | `trends.json` | `topic.md` (주제, 제목 후보, 썸네일 컨셉) |
| 03_script | 스크립트 작가 | `topic.md` | `script.md` (인트로/본문/아웃트로 구성) |
| 04_visual | 영상 기획자 | `script.md` | `storyboard.md` (장면 구성, 자막 계획) |
| 05_assets | 에셋 수집자 | `storyboard.md` | `/assets/` (이미지, 영상 클립, 음악) |
| 06_video | 영상 편집자 | `/assets/` + `storyboard.md` | `output.mp4` |
| 07_publish | 퍼블리셔 | `output.mp4` + `topic.md` | YouTube 업로드 메타데이터 |

---

## 디렉토리 구조

```
yt-global-korea02/
├── pipeline/
│   ├── 01_trend/         # 트렌드 리서치 결과
│   ├── 02_topic/         # 주제 기획서
│   ├── 03_script/        # 스크립트
│   ├── 04_visual/        # 스토리보드
│   ├── 05_assets/        # 수집된 에셋
│   ├── 06_video/         # 완성 영상
│   └── 07_publish/       # 배포 메타데이터
├── tools/                # 각 단계를 자동화하는 Python 스크립트
│   ├── trend_search.py   # YouTube API 트렌드 수집
│   ├── topic_planner.py  # 주제 기획 자동화
│   └── ...
├── config/
│   └── settings.yaml     # API 키 경로, 언어 설정 등
└── README.md
```

---

## 핵심 컨벤션

### 언어 및 톤
- 스크립트·기획 산출물은 **한국어**로 작성
- 시청자 대상이 30대 이상 한국인이므로, 어조는 **친근하되 신뢰감 있는 정보 전달형**
- 자극적·클릭베이트 성격은 지양하고, 외국인 관점의 진정성 있는 스토리 중심

### 프로그램 작성 규칙
- YouTube 검색·크롤링 등 외부 데이터 수집은 `tools/` 디렉토리에 **Python 스크립트**로 작성
- API 키 등 민감 정보는 `.env` 파일에서 로드하고 절대 소스코드에 하드코딩 금지
- 각 스크립트는 단독 실행 가능하도록 `if __name__ == "__main__":` 진입점 포함

### 산출물 관리
- 각 파이프라인 단계의 결과물은 해당 단계 디렉토리에 저장
- 날짜 기반 버전 관리: `YYYYMMDD_topic.md` 형식 권장
- 영상 에셋은 라이선스 정보를 함께 기록 (`05_assets/license_log.md`)

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
