# OpenMontage 전수조사 분석 & 활용/수익화 정리 (한국어)

> 이 문서는 OpenMontage 저장소를 폴더 단위로 전수조사한 결과와,
> 설치·사용법, 성격 규명(플러그인/스킬/MCP), API 토큰 정책, AI 에이전트 구축 활용도,
> React/PHP 구현 가능성, 유튜브 강의화 가능성, 수익화 아이디어를 하나로 정리한 자료입니다.

## 🔗 GitHub 주소

| 구분 | 주소 |
|---|---|
| **원본(업스트림) 저장소** | https://github.com/calesthio/OpenMontage |
| **이 저장소(포크)** | https://github.com/bmshin94/OpenMontage |
| 공식 웹사이트 | https://openmontage.video |
| 공식 유튜브 | https://www.youtube.com/@OpenMontage |
| 커뮤니티(Discussions) | https://github.com/calesthio/OpenMontage/discussions |
| 스폰서 | https://github.com/sponsors/calesthio |
| 라이선스 | GNU AGPLv3 (`LICENSE`) |

- 작성일: 2026-10-07
- 분석 기준 커밋: `81cbf31` (Merge pull request #1 from bmshin94/feat/claude-guide)
- 작업 브랜치: `claude/awesome-newton-6pfe96`

---

## 1. 한 줄 정의

> **AI 코딩 에이전트(Claude Code, Cursor, Copilot, Windsurf, Codex)를
> "영상 제작 스튜디오"로 바꿔주는 오픈소스 에이전틱 영상 제작 프레임워크.**

영상 편집 **앱이 아니다.** AI 에이전트가 읽고 따라 하는 **지시서(YAML + Markdown) + 도구(Python) 모음집**이다.
따라서 **에이전트 없이는 동작하지 않는다.**

---

## 2. 핵심 설계 철학 — "지능은 코드가 아니라 지시서에 있다"

`PROJECT_CONTEXT.md`에 명시된 실행 모델:

```
Agent reads pipeline manifest (YAML) → reads stage director skill (MD)
→ uses tools (Python BaseTool) → self-reviews (meta skill)
→ checkpoints (Python utility) → presents to human for approval
```

| 구성요소 | 역할 |
|---|---|
| **Python** | 도구(tool) + 저장(persistence) **만** |
| **YAML / Markdown** | 오케스트레이션, 창작 판단, 리뷰, 단계 전환 **전부** |
| **AI 에이전트** | 실제 "총감독" — 지시서를 읽고 판단하고 실행 |

> 문서 원문: **"No Python orchestrator, no Python reviewer, no Python handlers."**

---

## 3. 저장소 전수조사 (실측치)

| 폴더/파일 | 규모 | 역할 |
|---|---|---|
| `AGENT_GUIDE.md` | 48KB | 에이전트 계약서. Rule Zero 등 행동 규칙 전체 |
| `PROJECT_CONTEXT.md` | 8.4KB | 아키텍처 단일 진실 소스(Single Source of Truth) |
| `README.md` | 44KB | 쇼케이스 8편 + 설치 + 프로바이더 전체 |
| `README_zh-CN.md` | 39KB | 중국어 번역본 |
| `tools/` | **Python 167개** | 실제 도구. 카테고리별 패키지 분리 |
| `skills/` | **MD 157개** | Layer 2 — "OpenMontage가 도구를 쓰는 방식" |
| `.agents/skills/` | **폴더 90개 / 파일 994개** | Layer 3 — "기술 자체가 어떻게 동작하는지" |
| `pipeline_defs/` | **YAML 13개** | 파이프라인 선언서(단계/도구/승인 게이트) |
| `schemas/` | JSON 스키마 32개 | 산출물(artifact)·파이프라인·스타일 검증 |
| `styles/` | 플레이북 5개 | 비주얼 언어(타이포/모션/컬러) 정의 |
| `lib/` | Python 19개 | 체크포인트, 설정, CLIP 임베딩, 코퍼스, 스코어링 |
| `remotion-composer/` | React/TSX | Remotion 렌더링 엔진 + 씬 컴포넌트 18개 |
| `backlot/` | FastAPI + JS | "리빙 스토리보드" 실시간 제작 대시보드 |
| `ink-theater/` | JS + BVH 모캡 | 손그림 잉크 애니메이션 엔진 |
| `tests/` | Python 120개 | 계약 테스트, QA, 골든 시나리오, eval 하네스 |
| `docs/PROVIDERS.md` | 1,400줄+ | 프로바이더별 셋업 가이드 |
| `.claude/` `.cursor/` `.codex/` | 플랫폼 설정 | 멀티 에이전트 지원 |

### 3.1 도구 카테고리 (`tools/` 167개)

```
analysis/     (15) 전사, 장면감지, 얼굴추적, 영상이해, 비주얼QA, 다운로더
audio/        (22) TTS 11종 + 음악 8종 + 믹서/인핸스
video/        (40) 생성 20종 + 스티치/컴포즈/리프레임/무음컷/캡션번
graphics/     (22) 이미지생성 15종 + 3D(Three.js/Blender/Atlas/fal) + 다이어그램
avatar/        (4) 토킹헤드, 립싱크, Kling 아바타
enhancement/   (6) 업스케일, 얼굴복원, 배경제거, 컬러그레이딩
capture/       (3) 화면녹화
character/     (1) 캐릭터 애니메이션(리그/포즈/타임라인)
subtitle/      (1) 자막 생성
publishers/    (1) 내보내기 번들
```

**프로바이더 커버리지 60종 이상**: FLUX, Veo, Kling, Sora, Runway, Seedance, MiniMax, Hunyuan,
WAN, LTX-2, CogVideo, Grok, Jimeng, Higgsfield, HeyGen, ElevenLabs, Suno, Azure Speech,
Google Imagen/Chirp3/Lyria, OpenAI, Piper, Pexels/Pixabay/Unsplash, Atlas Cloud, DashScope,
fish.audio, Doubao, ComfyUI 등.

### 3.2 파이프라인 13종

| 파이프라인 | 산출물 |
|---|---|
| `animated-explainer` | AI 생성 설명영상(리서치→나레이션→비주얼→음악) |
| `animation` | 모션그래픽, 키네틱 타이포 |
| `cinematic` | 트레일러/티저/무드 편집 |
| `documentary-montage` | 무료 아카이브 **실촬영 footage** CLIP 검색 → 몽타주 |
| `talking-head` | 말하는 사람 중심 편집 |
| `screen-demo` | 소프트웨어 데모/튜토리얼 |
| `clip-factory` | 롱폼 1개 → 숏폼 다수 배치 추출 |
| `podcast-repurpose` | 팟캐스트 → 영상 하이라이트 |
| `avatar-spokesperson` | 아바타 발표자 영상 |
| `localization-dub` | 자막/더빙/번역 |
| `character-animation` | SVG 리그 캐릭터 애니메이션 |
| `hybrid` | 실촬영 + AI 보조 비주얼 |
| `framework-smoke` | 테스트 하네스 |

공통 흐름: `research → proposal → script → scene_plan → assets → edit → compose → publish`
각 단계마다 **디렉터 스킬**(`skills/pipelines/<pipeline>/<stage>-director.md`)을 반드시 먼저 읽는다.

### 3.3 거버넌스 (AGENT_GUIDE.md 강제 규칙)

1. **Rule Zero** — 모든 영상 제작은 파이프라인 경유. 즉석 Python 스크립트 금지.
2. **Decision Communication Contract** — 유료 호출 전 "툴명/프로바이더/모델/이유/샘플여부" 공개.
3. **Decision Log는 append-only** — 결정 변경 시 덮어쓰기 금지, 같은 `(category, subject)`로 신규 추가.
4. **두 런타임 모두 제시 (HARD RULE)** — Remotion·HyperFrames 둘 다 설치 시 **반드시 둘 다 제시 후 승인**.
   몰래 default 선택은 "CRITICAL reviewer finding".
5. **Templated vs Atelier** — 히어로 작업은 atelier(처음부터 손으로 저작) 기본 권장.
6. **Budget Controls** — `config.yaml`: `total_usd: 10.00`, `single_action_approval_usd: 0.50`.
7. **Quality Gates** — ffprobe 검증, 프레임 샘플링, 오디오 레벨 분석,
   `lib/delivery_promise.py` + `lib/slideshow_risk.py`로 "슬라이드쇼형 결과물" 차단.

---

## 4. 언제 쓰나 / 언제 안 맞나

### 적합
- "뉴럴넷 학습 원리 60초 설명영상 만들어줘" (리서치→완성본 자동)
- 2시간 팟캐스트 → 숏폼 12개 자동 추출
- 영상 1개 → 10개 언어 더빙/자막
- 레퍼런스 유튜브/릴스/틱톡 URL 제시 → "이런 느낌으로 내 주제로" (1급 워크플로)
- 브랜드 티저/제품 필름 (atelier 모드)
- **API 키 0개**로 무료 오픈 아카이브 실촬영 다큐 몽타주

### 부적합
- 프레임 단위 수동 미세조정 (→ 프리미어/다빈치)
- 클릭 몇 번으로 끝나는 GUI 기대 (CLI + 에이전트 대화형)
- 실시간 라이브 편집

---

## 5. 설치 및 사용법

### 5.1 사전 준비물

| 항목 | 설치 |
|---|---|
| Python 3.10+ | https://www.python.org/downloads/ |
| FFmpeg | `brew install ffmpeg` / `sudo apt install ffmpeg` |
| Node.js 18+ | https://nodejs.org/ (Remotion 렌더링용) |
| AI 코딩 에이전트 | Claude Code / Cursor / Copilot / Windsurf / Codex |

### 5.2 설치

```bash
git clone https://github.com/bmshin94/OpenMontage.git
cd OpenMontage
make setup
```

`make setup`이 수행하는 일:
1. `.venv` 가상환경 생성
2. `pip install -r requirements.txt`
3. `cd remotion-composer && npm install`
4. `pip install piper-tts` (무료 오프라인 TTS)
5. `cp .env.example .env`

**make 없을 때 (Windows PowerShell)**
```powershell
py -3 -m venv .venv; .\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
cd remotion-composer; npm install; cd ..
python -m pip install piper-tts; Copy-Item .env.example .env
```
`npm install`이 `ERR_INVALID_ARG_TYPE`로 실패하면 `npx --yes npm install` 사용.

### 5.3 점검 명령 (Makefile)

```bash
make preflight            # 사용 가능한 도구 점검
make test                 # 테스트 실행
make test-contracts       # 계약 테스트
make hyperframes-doctor   # HyperFrames 런타임 진단
make demo-list / make demo
make lint
```

내 환경의 실제 capability 확인:
```bash
python -c "from tools.tool_registry import registry; import json; registry.discover(); print(json.dumps(registry.support_envelope(), indent=2))"
python -c "from tools.tool_registry import registry; import json; registry.discover(); print(json.dumps(registry.provider_menu(), indent=2))"
```

### 5.4 사용법 — 자연어로 지시

```
"60초 애니메이션 설명영상 만들어줘, 주제는 뉴럴넷 학습 원리"
"이 유튜브 쇼츠 같은 느낌으로, 주제는 양자컴퓨팅으로 바꿔서"
"비 내리는 도시 생활 75초 다큐 몽타주. 실footage만, 나레이션 없이, 음악만"
"이 2시간 팟캐스트에서 숏폼 12개 뽑아줘"
```

### 5.5 결과물 위치

```
projects/<프로젝트명>/
├── artifacts/     # 각 단계 JSON (script.json, scene_plan.json ...)
├── assets/        # images/ video/ audio/ music/ subtitles.srt
└── renders/
    └── final.mp4  ← 최종 완성본
```

### 5.6 Backlot 대시보드

```bash
python -m backlot open <project-id>   # 특정 프로젝트 보드
python -m backlot open                # 전체 라이브러리
python -m backlot serve --port 4750   # 포그라운드 서버
```

---

## 6. 플러그인? 스킬? MCP? — 성격 규명

| 구분 | 해당 | 근거 |
|---|:---:|---|
| **MCP 서버** | ❌ 아니다 | MCP 서버 구현·의존성·stdio/SSE 핸들러 전무. 에이전트가 Python을 직접 실행 |
| **Claude 플러그인** | ❌ 아니다 | `.claude-plugin/`·`plugin.json` 없음. 마켓플레이스 설치 불가 |
| **스킬** | 🟡 부분적으로 그렇다 | `.claude/skills/`에 SKILL.md 형식 스킬 다수 실재(flux-best-practices, create-video, ai-video-gen, remotion, video-download, threejs-* 등) → 레포를 열면 자동 인식 |
| **클론해서 쓰는 프로젝트 레포** | ✅ **정답** | `git clone` → `make setup` → 에이전트로 열기 |

### 6.1 3계층 지식 아키텍처

```
Layer 1: tools/tool_registry.py   → "어떤 도구가 있나" (런타임 능력/상태/비용)
Layer 2: skills/ (157개 MD)        → "OpenMontage는 이걸 어떻게 쓰나" (프로젝트 관례)
Layer 3: .agents/skills/ (994개)   → "그 기술이 어떻게 동작하나" (API 규칙)
```
각 도구의 `agent_skills[]` 필드가 Layer 1 → Layer 3를 연결한다.

### 6.2 멀티 에이전트 지원

| 플랫폼 | 설정 파일 |
|---|---|
| Claude Code | `CLAUDE.md` + `.claude/skills/` |
| Cursor | `CURSOR.md` + `.cursor/rules/` |
| GitHub Copilot | `COPILOT.md` + `.github/copilot-instructions.md` |
| Codex | `CODEX.md` + `.codex/` |
| Windsurf | `.windsurfrules` |
| 공통 | `AGENTS.md` → `AGENT_GUIDE.md` |

> MCP 서버로의 전환은 가능하다. `tools/base_tool.py`의 `BaseTool`이
> `name`/`version`/`capability`/`supports`/`execute() → ToolResult` 계약을 이미 갖추고 있어
> 어댑터 한 겹으로 래핑할 수 있다.

---

## 7. API 토큰 정책 — 필수가 아니다

`.env.example`의 **37개 키 전부 optional**이다.

### 7.1 API 키 0개로 가능한 것

| 기능 | 무료 도구 |
|---|---|
| 나레이션 | **Piper TTS** (오프라인, 무료) |
| 실촬영 footage | **Archive.org + NASA + Wikimedia Commons** |
| 합성 (React) | **Remotion** (스프링 애니메이션, 카드/차트, 단어단위 캡션) |
| 합성 (HTML/GSAP) | **HyperFrames** (키네틱 타이포, SVG 캐릭터 리깅) |
| 후반작업 | **FFmpeg** (인코딩, 자막 번인, 믹싱, 컬러그레이딩) |
| 자막 | 내장 (단어단위 타이밍) |
| 전사(STT) | faster-whisper 로컬 (`tools/analysis/transcriber.py`) |
| 음악 | Pixabay / Freesound / music_library |

무료 경로 3가지: ① 이미지 기반 영상 ② 실footage 다큐 몽타주 ③ 로컬 캐릭터 애니메이션

### 7.2 키 추가 시 열리는 기능 (선택)

| 키 | 기능 | 가성비 |
|---|---|:---:|
| `FAL_KEY` | FLUX 이미지 + Veo/Kling/MiniMax 영상 + Recraft | ★★★★★ |
| `ATLASCLOUD_API_KEY` | Seedream/Nano Banana/GPT Image + Kling/Seedance/Hailuo | ★★★★★ |
| `PEXELS_API_KEY` `PIXABAY_API_KEY` `UNSPLASH_ACCESS_KEY` | 무료 스톡 (키 발급 자체가 무료) | ★★★★★ |
| `GOOGLE_API_KEY` | Imagen + Chirp3 TTS(700+ 보이스) + Lyria + Veo | ★★★★ |
| `ELEVENLABS_API_KEY` | 프리미엄 TTS + AI 음악 + 효과음 | ★★★★ |
| `OPENAI_API_KEY` | OpenAI TTS + GPT Image 2 + Sora 2 | ★★★ |
| `SUNO_API_KEY` | 완전한 노래/연주곡 | ★★★ |
| `KLING_API_KEY` | Kling 공식 직통(영상/이미지/TTS/아바타/립싱크) | ★★★ |
| `HEYGEN_API_KEY` | 아바타 + VEO/Sora/Runway/Kling 게이트웨이 | ★★★ |
| `AZURE_SPEECH_KEY` | Azure STT + TTS (단일 리소스) | ★★★ |

### 7.3 권장 단계

```bash
# 1단계: 완전 무료 (키 0개)
make setup

# 2단계: 무료 키만 발급
PEXELS_API_KEY= / PIXABAY_API_KEY= / UNSPLASH_ACCESS_KEY=

# 3단계: 품질 확보 ($10~20 충전)
FAL_KEY=  (또는 ATLASCLOUD_API_KEY=)
GOOGLE_API_KEY=
```

### 7.4 예산 안전장치 (`config.yaml`)

```yaml
budget:
  mode: warn                       # observe | warn | cap  ← 처음엔 cap 권장
  total_usd: 10.00
  reserve_pct: 0.10
  single_action_approval_usd: 0.50
  require_approval_for_new_paid_tool: true
```

> ⚠️ LLM 토큰 비용(에이전트 대화비)은 이 예산에 포함되지 않는다. 별도 계산 필요.

---

## 8. AI 에이전트 구축에 도움이 되는가 — 매우 그렇다

영상과 무관한 도메인(법률/의료/금융/커머스) 에이전트를 만들 때도 그대로 적용 가능한 설계 패턴:

| # | 패턴 | 참고 파일 |
|:---:|---|---|
| 1 | **3계층 지식 분리** (레지스트리 / 프로젝트 관례 / 기술 지식팩) → 필요할 때만 로드해 컨텍스트 절약 | `PROJECT_CONTEXT.md` |
| 2 | **셀렉터 + 프로바이더 패턴** (라우터 1개 + 구현체 N개) | `tools/audio/tts_selector.py`, `tools/video/video_selector.py` |
| 3 | **7차원 스코어링 선택기** (task fit·quality·control·reliability·cost·latency·continuity) | `lib/scoring.py` |
| 4 | **Append-only 결정 로그** — 과거 결정 수정은 결함(defect)으로 취급 | `schemas/artifacts/decision_log.schema.json` |
| 5 | **스키마 검증된 산출물** — 단계 간 계약 명확화, LLM의 임의 구조 변경 차단 | `schemas/artifacts/` (22개) |
| 6 | **체크포인트 + 재개** — 장시간 워크플로 중단 복구 | `lib/checkpoint.py` |
| 7 | **품질 게이트 + 자기검수(최대 2라운드)** — 쓰레기 결과물의 사용자 노출 차단 | `skills/meta/reviewer.md`, `lib/delivery_promise.py`, `lib/slideshow_risk.py` |

### 8.1 "에이전트 계약서" 작성 테크닉 (`AGENT_GUIDE.md` 48KB)

| 테크닉 | 실제 문구 |
|---|---|
| 강제 라우팅 | "Read AGENT_GUIDE.md before responding to ANY user message" |
| Rule Zero | "Every video production request MUST go through the pipeline. No exceptions." |
| 명시적 금지 목록 | "Do NOT: Write ad-hoc Python scripts to call tools directly..." |
| HARD RULE 표시 | "silently picking a default is forbidden" |
| 위반 심각도 표기 | "...is a CRITICAL reviewer finding" |
| 에스컬레이션 템플릿 | 시도 → 실패 → 원인분류 → 선택지 → 추천 (5단) |
| 일방적 대체 금지 | "No Unilateral Substitutions" |

### 8.2 Backlot = 에이전트 관측성(Observability) 모범 사례

> `backlot/README.md` 원문: **"No agent involvement."**
> `watchfiles` 워처가 `projects/` 변화를 감지 → SSE 푸시 → 브라우저가 상태 재조회.

에이전트에게 "보고하라"고 시키지 않는다. 에이전트는 파일만 쓰고, 보드가 파일을 읽어 스스로 채워진다.
에이전트에게 로깅 책임을 주면 누락·왜곡이 발생하므로, 이 방식이 더 견고하다.

### 8.3 상황별 참조 맵

| 목표 | 참조 |
|---|---|
| 에이전트 설계 학습 | `AGENT_GUIDE.md`, `PROJECT_CONTEXT.md`, `skills/meta/*` |
| 도구 레이어 설계 | `tools/base_tool.py`, `tools/tool_registry.py` |
| 멀티 프로바이더 추상화 | `tools/audio/tts_selector.py`, `lib/scoring.py` |
| 워크플로 상태머신 | `lib/checkpoint.py`, `lib/pipeline_loader.py`, `pipeline_defs/*.yaml` |
| 산출물 계약 설계 | `schemas/artifacts/` |
| 품질 게이트 | `skills/meta/reviewer.md`, `lib/delivery_promise.py`, `lib/slideshow_risk.py` |
| 관측성 대시보드 | `backlot/` (FastAPI + watchfiles + SSE) |
| 평가(eval) 하네스 | `tests/eval/` (golden_scenarios, replay_harness, golden_outputs) |

---

## 9. React / PHP로 만들 수 있는가

### 9.1 React — 이미 핵심에 포함됨 ✅

`remotion-composer/`가 React다. 실측 구성:

```
remotion-composer/src/
├── Root.tsx, Explainer.tsx, CinematicRenderer.tsx
├── TitledVideo.tsx, TalkingHead.tsx, CollageBurst.tsx, LyricOverlay.tsx
├── components/ (18개)
│   ├── HeroTitle, TextCard, StatCard, StatReveal, ProgressBar
│   ├── CalloutBox, ComparisonCard, CaptionOverlay, SectionTitle, EndTag
│   ├── ProductReveal, ScreenshotScene, TerminalScene, AnimeScene
│   ├── ParticleOverlay, ProviderChip
│   └── charts/
└── lib/resolveAsset.ts
```

**React 컴포넌트가 곧 영상 씬**이므로, React를 다룰 수 있으면 즉시 새 씬 타입을 추가할 수 있다.
참고: `remotion-composer/SCENE_TYPES.md`, `skills/core/remotion.md`, `.agents/skills/remotion-best-practices/`

### 9.2 React로 웹 UI 붙이기 — 현실적 ✅

```
backlot/server.py      # FastAPI + SSE  ← 그대로 사용
backlot/state.py       # 디스크 → 보드 상태
backlot/ui/board.js    # 바닐라 JS      ← React/Next.js로 교체 가능
```

### 9.3 PHP로 전체 재작성 — 비추천 ⚠️

| 이유 | 설명 |
|---|---|
| 생태계 미스매치 | torch/numpy/Pillow/whisper/CLIP 임베딩이 Python 중심 |
| Remotion이 Node 전용 | React 기반, PHP로 대체 불가 |
| 재작성 규모 | Python 167개 + MD 1,151개 → 수개월 |
| 롱러닝 작업 부적합 | 렌더가 수 분~수십 분, PHP-FPM 요청 모델과 불일치 |

### 9.4 PHP 권장 구조 — "프론트는 PHP, 엔진은 Python"

```
[PHP/Laravel 웹앱]
  ├─ 인증, 결제, 주문 관리, 대시보드
  └─ 작업 큐에 영상 요청 등록 (DB/Redis)
            ↓
[Python 워커 = OpenMontage 원본 그대로]
  ├─ 큐 소비 → 파이프라인 실행 → final.mp4
  └─ 상태를 DB/파일에 기록
            ↓
[PHP가 결과 조회 → 사용자 제공]
```

장점: OpenMontage를 포크/수정하지 않아 업스트림 업데이트를 그대로 수용, 워커 수평 확장 용이.
⚠️ 단, 외부에 서비스로 제공하면 AGPLv3 §13 검토가 필요하다(§11 참조).

### 9.5 난이도 요약

| 작업 | 난이도 |
|---|:---:|
| React 씬 컴포넌트 추가 | ★ 쉬움 |
| Backlot UI를 React로 교체 | ★★ 보통 |
| PHP 웹앱 + Python 워커 연동 | ★★★ 중상 |
| 새 도구(tool) 추가 (`BaseTool` 상속, 6단계) | ★★ 보통 |
| 새 파이프라인 추가 (YAML + 디렉터 스킬 7개) | ★★★ 중상 |
| PHP 전체 재작성 | ★★★★★ 비현실 |

---

## 10. 유튜브 강의 영상 제작 가능성 — 가능하고 타이밍이 좋다

### 근거
- 소재 과잉: 파이프라인 13종 / 도구 167개 / 프로바이더 60종 + `PROMPT_GALLERY.md` 예제
- 썸네일용 숫자: "60초 픽사풍 애니 **$1.33**", "API 키 **0개**", "프롬프트 한 줄로 3D 월드"
- 메타 플렉스: **OpenMontage로 OpenMontage 강의를 제작** (`screen-demo` 파이프라인)
- 경쟁 공백: 공식 채널 외 **한국어 콘텐츠 사실상 없음**

### 추천 커리큘럼 (12편)

| # | 제목 | 길이 | 난이도 |
|:---:|---|:---:|:---:|
| 0 | "AI가 혼자 영상 만들어줬습니다" (충격 데모) | 3분 | 훅 |
| 1 | 설치 완벽 가이드 (Windows/Mac) | 12분 | ★ |
| 2 | API 키 0개로 첫 영상 만들기 | 15분 | ★ |
| 3 | 13개 파이프라인 전부 소개 + 선택 기준 | 18분 | ★★ |
| 4 | $1.33로 픽사풍 애니 만들기 (실비용 공개) | 20분 | ★★ |
| 5 | 레퍼런스 영상 기반 제작 | 15분 | ★★ |
| 6 | 팟캐스트 2시간 → 숏폼 12개 | 15분 | ★★ |
| 7 | 내 영상 10개 언어 더빙 | 15분 | ★★ |
| 8 | Backlot 대시보드 + 비용 통제 | 12분 | ★★ |
| 9 | React로 나만의 씬 컴포넌트 만들기 | 25분 | ★★★ |
| 10 | 새 도구 추가하기 (BaseTool 상속) | 25분 | ★★★ |
| 11 | AI 에이전트 설계 패턴 7개 (개발자용) | 30분 | ★★★★ |

### 포맷 전략

| 포맷 | 용도 |
|---|---|
| 숏츠 30~60초 | "$1.33", "키 0개", "3D 월드 한 프롬프트" → 유입 |
| 롱폼 12~25분 | 커리큘럼 → 수익 + 체류시간 |
| 라이브 빌드 | 시청자 주제로 실시간 제작 → 참여도 |
| 비교 영상 | OpenMontage vs Runway vs Pika → 검색 유입 |

### 제작 시 주의사항
1. **AGPLv3 명시** (특히 "사업 활용" 편)
2. **원작자 크레딧**: https://github.com/calesthio/OpenMontage 설명란 표기
3. **비용은 실측치만** 공개 (추정치 금지)
4. **AI 생성물 고지** (유튜브 합성 콘텐츠 표시 정책)
5. **스폰서 링크 주의** — README의 Bloome/Atlas Cloud 링크에는 `ref` 제휴 파라미터가 포함되어 있다. 그대로 쓰면 원작자 수익이므로, 본인 제휴는 별도 체결 필요
6. **버전 변화 빠름** — 영상에 날짜/커밋 해시 표기 권장

### 현실적 기대치

| 지표 | 예상 |
|---|---|
| 초기 10편 제작 | 2~3개월 (주 1편) |
| 유튜브 광고 단독 수익 | 초기엔 소액 |
| 실질 수익원 | 강의 판매 / 제작 대행 유입 / 컨설팅 |

→ 유튜브는 수익원보다 **신뢰 자산 + 리드 생성기**로 보는 것이 정확하다.

---

## 11. 수익화 아이디어

### 11.0 선행 전제 — AGPLv3 이해 (⚠️ 필수)

| 상황 | 소스 공개 의무 |
|---|:---:|
| 내 PC에서 혼자 사용 | ❌ 없음 |
| **만든 영상을 판매/상업 이용** | ❌ **없음 (영상은 내 저작물)** |
| 사내에서만 사용 | ❌ 없음 (배포 아님) |
| 코드를 수정해 배포 | ✅ AGPL로 공개 |
| **수정판을 네트워크 서비스(SaaS)로 제공** | ✅ **AGPL §13 — 이용자에게 소스 제공** |

> **핵심**: AGPL은 **코드**에 걸린다. **결과물(영상)에는 제약이 없다.**
> 따라서 가장 안전한 수익화는 **"결과물과 전문성을 파는 것"**이고,
> 리스크가 큰 것은 **"코드를 SaaS로 파는 것"**이다.
> ⚠️ 실제 사업 전 변호사 검토 필수. 본 문서는 법률 자문이 아니다.

### 11.1 Tier 1 — 라이선스 리스크 없음, 즉시 시작 가능

#### ① 숏폼 제작 대행 (최우선 추천)

| 항목 | 내용 |
|---|---|
| 상품 | 월 정액으로 숏폼 N개 납품 |
| 원가 | 영상당 $0.15~$1.50 + LLM 토큰비 |
| 가격 | 영상당 5~15만 원 |
| 마진 | 매우 높음 (원가가 매출의 1~2%) |
| 타깃 | 중소기업, 1인 쇼핑몰, 병원/학원, 부동산, 식당 |
| AGPL | ✅ 없음 |

```
라이트  : 월 30만원 → 숏폼 4개
스탠다드: 월 80만원 → 숏폼 12개 + 썸네일
프리미엄: 월 200만원 → 숏폼 30개 + 롱폼 2개 + 리포트
```
차별화: `clip-factory`로 **롱폼 1개 → 숏폼 12개 자동 추출** → "기존 영상 재활용" 상품 가격 경쟁력 압도.

#### ② 다국어 더빙/현지화 서비스

| 항목 | 내용 |
|---|---|
| 상품 | 영상 1개 → 10개 언어 더빙 + 자막 |
| 근거 | `localization-dub` + Google TTS 700+ 보이스 + ElevenLabs |
| 기존 시세 | 전문 더빙 분당 수만~수십만 원 |
| 원가 | TTS 호출비 수준 |
| 타깃 | 유튜버, 온라인 강의, 게임사, K-뷰티/푸드 브랜드 |
| AGPL | ✅ 없음 |

#### ③ 유튜브 + 온라인 강의

```
유튜브(무료, 리드) → 유료 강의 → 1:1 컨설팅 / 기업 교육
```

| 상품 | 가격대 |
|---|---|
| 유튜브 광고 | 월 수십만 원 |
| 온라인 강의 | 15~25만원 × N명 |
| 노션 템플릿/프롬프트팩 | 2~5만원 |
| **기업 출강** | 회당 100~300만원 |
| 멤버십/디스코드 | 월 2~5만원 |

#### ④ 니치 콘텐츠 채널 양산

| 니치 | 파이프라인 |
|---|---|
| 과학/역사 설명 | `animated-explainer` |
| 명상/힐링 | `documentary-montage` |
| 제품 리뷰 | `hybrid` |
| 영어 학습 | `avatar-spokesperson` |
| 3D 쇼케이스 | `animation` |

```
채널 1개 = 주 3편 = 월 12편
월 원가 = 12 × $1 ≈ $12 (+LLM 토큰비)
채널 5개 운영 → 월 원가 10만원 이하
```

#### ⑤ 기업 내부 영상 자동화 컨설팅

| 항목 | 내용 |
|---|---|
| 상품 | 사내 셋업 + 커스터마이즈 + 교육 |
| 근거 | **사내 사용은 AGPL "배포"가 아님** → 공개 의무 없음 |
| 가격 | 구축 500~2,000만원 + 유지보수 월 50~200만원 |
| 타깃 | 교육기업, 보험/금융, 제조(매뉴얼), HR(온보딩) |
| 셀링포인트 | "데이터가 외부로 나가지 않음" (Piper/Remotion/FFmpeg 로컬) |

### 11.2 Tier 2 — 가능하지만 라이선스 검토 필수

#### ⑥ SaaS 래핑

```
[React 프론트] → [PHP/Laravel 또는 FastAPI] → [Python 워커 = OpenMontage]
```

```
무료     : 월 3개 (워터마크)
Starter  : 월 29,000원 → 20개
Pro      : 월 99,000원 → 100개 + 워터마크 제거
Business : 월 299,000원 → 무제한 + API
```

| 케이스 | 의무 |
|---|:---:|
| 수정 없이 별개 프로세스로 호출, 내 코드는 독립 | 🟡 회색지대 — 법률 검토 |
| OpenMontage 코드를 수정해 서비스 | 🔴 내 서비스 소스 AGPL 공개 |
| UI/비즈니스 로직이 한 덩어리(결합 저작물) | 🔴 공개 의무 |

완화 전략:
- **A. 프로세스 완전 분리** — 원본 그대로, 별도 컨테이너, CLI/큐로만 호출
- **B. 오픈소스로 공개** — 호스팅 편의성으로 수익 (GitLab/Sentry 모델)
- **C. 서비스 중심** — 소프트웨어가 아니라 제작 서비스를 판매 (= Tier 1)
- **D. 상업 라이선스 협의** — 원작자에게 듀얼 라이선스 문의 (가장 깔끔)

권장: **C로 현금흐름 확보 → 규모 확대 시 D 협의**

#### ⑦ MCP 서버로 래핑해 배포

| 항목 | 내용 |
|---|---|
| 기술 난이도 | ★★ (`BaseTool` 계약이 명확해 어댑터만 필요) |
| 수익 모델 | 무료 OSS로 명성 → 호스팅 유료 / 스폰서십 / 컨설팅 유입 |
| AGPL | 🔴 파생물이면 공개 — 애초에 OSS 공개라면 문제 없음 |
| 매력 | MCP 생태계 급성장 중, 선점 효과 |

#### ⑧ 프리미엄 템플릿/플레이북 판매

현재 `styles/`에 플레이북 5개뿐(anime-ghibli, clean-professional, flat-motion-graphics,
minimalist-diagram, premium-minimalist). 스키마(`schemas/styles/playbook.schema.json`)가 공개되어
누구나 제작 가능 → 시장 공백.

| 상품 | 가격 |
|---|---|
| 업종별 플레이북 팩(병원/법률/뷰티/부동산/교육) | 5~15만원 |
| Remotion 씬 컴포넌트 팩(React) | 10~30만원 |
| 브랜드 커스텀 플레이북 제작 | 50~200만원 |

⚠️ 플레이북 YAML의 파생물 여부는 회색지대 → **별도 저작물로 설계** 권장.

### 11.3 Tier 3 — 장기/간접

| # | 아이디어 |
|:---:|---|
| ⑨ | 오픈소스 기여 → GitHub 포트폴리오 → AI 엔지니어 커리어 |
| ⑩ | 기술 블로그 + 유료 뉴스레터("AI 에이전트 설계 패턴") |
| ⑪ | 책/전자책 — OpenMontage를 케이스 스터디로 |
| ⑫ | 한국 유저 커뮤니티 운영 → 멤버십/구인 중개 |
| ⑬ | 스폰서십 중개 — 제작량 확보 후 Atlas Cloud/fal.ai 등 제휴 제안 |

### 11.4 실행 로드맵

```
[0~1개월] 무료로 익히기
  make setup → 키 0개로 영상 5개 → 원가/시간/품질 실측

[1~3개월] 유튜브 + 포트폴리오
  "$1.33 영상" 숏츠 → 롱폼 강의 → 니치 채널 1개 테스트

[2~4개월] 제작 대행 시작 (첫 현금흐름)
  지인/중소기업 2~3곳 저가 레퍼런스 → 포트폴리오 → 정가 전환

[4~8개월] 확장
  강의 판매 + 다국어 더빙 상품 + 기업 컨설팅 / 채널 3~5개

[8개월+] 스케일
  SaaS 검토 (변호사 + 원작자 라이선스 협의)
```

### 11.5 우선순위 매트릭스

| 아이디어 | 수익성 | 난이도 | AGPL 리스크 | 추천 순위 |
|---|:---:|:---:|:---:|:---:|
| 숏폼 제작 대행 | ★★★★ | ★★ | 없음 | **1** |
| 다국어 더빙 | ★★★★ | ★★ | 없음 | **2** |
| 유튜브 + 강의 | ★★★ | ★★ | 없음 | **3** |
| 기업 컨설팅 | ★★★★★ | ★★★★ | 없음 | 4 |
| 니치 채널 양산 | ★★★ | ★★★ | 없음 | 5 |
| 템플릿 판매 | ★★ | ★★ | 회색 | 6 |
| MCP 서버 | ★ | ★★ | OSS면 없음 | 7 |
| SaaS | ★★★★★ | ★★★★★ | 높음 | 8 ⚠️ |

---

## 12. 한계와 주의사항 (솔직한 평가)

- **라이선스**: AGPLv3 강력 카피레프트. SaaS 제공 시 소스 공개 의무 발생 가능
- **설치 장벽**: Python 3.10+ / Node.js 18+ / FFmpeg 필수
- **로컬 영상생성은 GPU 필요** (`requirements-gpu.txt`: torch/torchaudio/torchvision)
- **품질이 에이전트 실력에 좌우됨** — 지시서를 읽지 않는 모델은 결과물 품질이 크게 떨어진다 (가이드에도 명시)
- **LLM 토큰 비용 별도** — "영상 $1.33"에 에이전트 대화 비용은 포함되지 않음
- **활발한 개발 중** — API/구조 변화가 빠르므로 문서·강의는 버전 명시 필요
- **README 쇼케이스 비용은 원작자 실측치** — 본인 환경/프로바이더에 따라 달라질 수 있음

---

## 13. 참고 문서 맵

| 알고 싶은 것 | 읽을 파일 |
|---|---|
| 에이전트 행동 규칙 전체 | `AGENT_GUIDE.md` |
| 아키텍처 개요 | `PROJECT_CONTEXT.md`, `docs/ARCHITECTURE.md` |
| 스킬 전체 색인 | `skills/INDEX.md` |
| 프로바이더 셋업 | `docs/PROVIDERS.md` |
| 예제 프롬프트 | `PROMPT_GALLERY.md` |
| Remotion 씬 타입 | `remotion-composer/SCENE_TYPES.md` |
| Backlot 동작 원리 | `backlot/README.md` |
| 기여 방법 | `CONTRIBUTING.md`, `docs/PR_REVIEW_GUIDE.md` |
| 환경변수 전체 | `.env.example` (37개, 전부 optional) |
| 전역 설정 | `config.yaml` |

---

**문서 끝.** 분석 기준 커밋 `81cbf31` / 작성일 2026-10-07
