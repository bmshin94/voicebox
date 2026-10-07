# Voicebox 전수조사 & 활용 가이드 (한국어)

> 작성일: 2026-10-07
> 작성: Claude Code (페르소나: 카리나 💖)
> 정리 대상: 이 저장소(Voicebox) 전체 구조 분석 + 설치/사용법 + AI 에이전트 활용 + 수익화 전략

---

## 🔗 GitHub 주소

| 구분 | 주소 |
|---|---|
| **이 저장소 (포크)** | https://github.com/bmshin94/voicebox |
| **원본 저장소 (upstream)** | https://github.com/jamiepine/voicebox |
| 공식 웹사이트 | https://voicebox.sh |
| 공식 문서 | https://docs.voicebox.sh |
| 릴리즈 다운로드 | https://github.com/jamiepine/voicebox/releases/latest |
| DeepWiki (코드 Q&A) | https://deepwiki.com/jamiepine/voicebox |
| 작업 브랜치 | `claude/funny-goldberg-zh2f9p` |

---

## 목차

1. [프로젝트 개요](#1-프로젝트-개요)
2. [폴더 구조 전수조사](#2-폴더-구조-전수조사)
3. [기술 스택](#3-기술-스택)
4. [기능 상세](#4-기능-상세)
5. [설치 및 사용법](#5-설치-및-사용법)
6. [플러그인 / 스킬 / MCP 구분](#6-플러그인--스킬--mcp-구분)
7. [API 토큰 및 인증](#7-api-토큰-및-인증)
8. [AI 에이전트 구축 활용](#8-ai-에이전트-구축-활용)
9. [React / PHP 로 만들 수 있는가](#9-react--php-로-만들-수-있는가)
10. [유튜브 강의 제작 가능성](#10-유튜브-강의-제작-가능성)
11. [수익화 아이디어 상세](#11-수익화-아이디어-상세)
12. [주의사항 및 리스크](#12-주의사항-및-리스크)
13. [빠른 참조 치트시트](#13-빠른-참조-치트시트)

---

## 1. 프로젝트 개요

### 한 줄 요약

**Voicebox = 내 컴퓨터에서 로컬로 돌아가는 오픈소스 AI 음성 스튜디오.**
목소리 복제 + 음성 생성(TTS) + 받아쓰기(STT) + AI 에이전트 음성 출력을 전부 로컬에서 처리한다.

### 기본 정보

| 항목 | 값 |
|---|---|
| 라이선스 | **MIT** (상업적 이용 가능) |
| 현재 버전 | **v0.5.0** |
| 코드 규모 | 약 **70,000줄** |
| 파일 구성 | TSX 147 · TS 116 · Python 123 · Rust 18 · MDX 57 |
| 저장소 내부 문서 기준 지표 | ⭐ 34.8k stars · 다운로드 1.3M · open issue 402 · open PR 88 (`docs/PROJECT_STATUS.md`) |
| 백엔드 포트 | `127.0.0.1:17493` (Docker는 호스트 `17600`) |

### 포지셔닝

| 서비스 | 담당 영역 | 과금 |
|---|---|---|
| ElevenLabs | 음성 출력 (TTS) | 유료 구독 |
| WisprFlow | 음성 입력 (STT/받아쓰기) | 유료 구독 |
| **Voicebox** | **양방향 전부** | **무료 / 로컬** |

README 원문: *"The two cloud incumbents sit on opposite halves of the voice I/O loop — ElevenLabs on output, WisprFlow on input. Voicebox does both."*

두 방향을 **번들된 로컬 LLM(Qwen3)**으로 이어붙여 문장 정제·페르소나 변환까지 처리하는 것이 핵심 차별점.

### 로컬 vs 클라우드 비교

| 항목 | 클라우드 (ElevenLabs) | 로컬 (Voicebox) |
|---|---|---|
| 목소리 데이터 | 외부 서버 전송 | **내 PC에만 보관** |
| 비용 | 월 구독 | 0원 (전기/하드웨어) |
| 사용량 제한 | 글자수 제한 | 사실상 무제한 |
| 인터넷 | 필수 | 모델 다운로드 후 불필요 |
| 품질 | 현재 우위 | 엔진별로 준수 |
| 속도 | 일정 | 내 GPU 성능 의존 |
| 초기 부담 | 없음 | 모델 350MB~8GB 다운로드 |

---

## 2. 폴더 구조 전수조사

```
voicebox/
├── app/              # React 프론트엔드 본체 (UI 전부)
├── tauri/            # 데스크탑 셸 (Tauri + Rust)
├── web/              # 브라우저 배포용 얇은 래퍼
├── backend/          # FastAPI 파이썬 서버 (AI 두뇌)
├── docs/             # 공식 문서 사이트 (Next.js + Fumadocs)
├── landing/          # 마케팅 랜딩페이지 (voicebox.sh)
├── scripts/          # 빌드 · 릴리즈 스크립트
├── data/             # 로컬 데이터 디렉터리
├── .agents/skills/   # AI 코딩 에이전트용 스킬 4개
├── .github/workflows # CI/CD 3개 (ci, release, build-windows)
├── justfile          # 전체 명령어 런처 (Makefile 대체)
├── docker-compose.yml / docker-compose.rocm.yml
├── .mcp.json         # Claude Code 자동 MCP 연결 설정
└── CLAUDE.md         # 이 포크에서 추가된 페르소나 가이드
```

### 2.1 `app/` — React 프론트엔드

**컴포넌트 (`app/src/components/`)**

| 폴더 | 역할 |
|---|---|
| `VoicesTab/` | 목소리 프로필 목록 · 인스펙터 |
| `VoiceProfiles/` | 프로필 생성, 샘플 녹음/업로드/시스템오디오 |
| `Generation/` | 생성 입력창, 엔진/모델 선택, 감정태그 입력 |
| `StoriesTab/` | 멀티트랙 타임라인 (대화 · 팟캐스트) |
| `EffectsTab/`, `Effects/` | 이펙트 체인 에디터 · 프리셋 |
| `CapturesTab/` | 받아쓰기/녹음 보관함, 준비상태 체크리스트 |
| `DictateWindow/`, `CapturePill/` | 플로팅 녹음 알약 오버레이 |
| `ServerTab/` | 설정 전체 (General, GPU, Generation, Logs, **MCPPage**, Cloud, About, Changelog) |
| `ModelsTab/`, `AudioTab/` | 모델 관리 · 오디오 장치 |
| `AccessibilityGate/`, `InputMonitoringGate/` | macOS 권한 안내 게이트 |
| `ChordPicker/` | 단축키(chord) 리바인딩 UI |
| `AudioPlayer/`, `MainEditor/`, `AppFrame/`, `Sidebar.tsx` | 공통 셸 · 플레이어 |
| `ui/` (24개) | shadcn/ui 기반 공통 컴포넌트 |

**상태관리 (`app/src/stores/` — Zustand 8개)**
`serverStore` · `logStore` · `playerStore` · `effectsStore` · `uiStore` · `generationStore` · `storyStore` · `audioChannelStore`

**커스텀 훅 (`app/src/lib/hooks/`)**
`useGeneration` · `useGenerationForm` · `useGenerationProgress` · `useProfiles` · `useStories` · `useHistory` · `useTranscription` · `useAudioRecording` · `useSystemAudioCapture` · `useCaptureRecordingSession` · `useDictationReadiness` · `useChordSync` · `useMCPBindings` · `useSettings` · `useServer` · `useModelDownloadToast` · `useRestoreActiveTasks` · `useStoryPlayback` · `useAudioPlayer`

**플랫폼 추상화 (`app/src/platform/`)** — Tauri / Web 환경 차이를 흡수해 동일 컴포넌트를 재사용.

### 2.2 `backend/` — 파이썬 AI 서버

```
backend/
├── backends/     # AI 엔진 어댑터 (TTS 7 + LLM + 플랫폼별 런타임)
├── routes/       # REST API 엔드포인트 (21개 파일)
├── services/     # 비즈니스 로직 (24개 파일)
├── mcp_server/   # MCP 서버 (FastMCP): server/tools/context/resolve/events
├── mcp_shim/     # stdio ↔ HTTP 중계기 (voicebox-mcp 바이너리)
├── database/     # SQLite 모델 · 세션 · 마이그레이션 · 시드
├── utils/        # 오디오 · 이펙트 · 청크분할 · 다운로드 진행률 · 플랫폼감지
├── pyi_hooks/    # PyInstaller 훅
├── tests/        # 테스트 30개 이상
├── app.py        # FastAPI 앱 조립 (MCP lifespan 통합)
├── config.py     # 데이터/모델 디렉터리, 클라우드 URL
├── models.py     # Pydantic 요청/응답 모델
└── build_binary.py / voicebox-server.spec   # 바이너리 번들링
```

**`routes/` 주요 엔드포인트 파일**
`generations.py` · `profiles.py` · `speak.py` · `transcription.py` · `effects.py` · `stories.py` · `captures.py` · `models.py` · `history.py` · `channels.py` · `llm.py` · `mcp_bindings.py` · `tasks.py` · `events.py` · `health.py` · `audio.py` · `cuda.py` · `rocm.py` · `cloud.py` · `settings.py`

**`services/` 주요 로직**
`generation.py` · `tts.py` · `transcribe.py` · `llm.py` · `personality.py` · `refinement.py` · `effects.py` · `profiles.py` · `captures.py` · `stories.py` · `versions.py` · `history.py` · `task_queue.py` · `export_import.py` · `channels.py` · `settings.py` · `cloud.py` · `cuda.py` · `rocm.py`

### 2.3 `tauri/` — 데스크탑 셸 (Rust)

Electron이 아니라 **Tauri**. Rust 네이티브 셸이 담당하는 것:
- 전역 단축키 (어느 앱에서든 녹음 시작)
- 붙여넣기 주입 (macOS 접근성 API로 포커스된 입력창에 텍스트 삽입)
- 포커스 인트로스펙션 (현재 활성 입력창 파악)
- 클립보드 원자적 저장/복원 (사용자 클립보드 보호)

### 2.4 `.agents/skills/` — AI 에이전트 스킬 (숨은 보석)

| 스킬 | 역할 |
|---|---|
| `add-tts-engine` | **새 TTS 엔진을 에이전트가 자율적으로 통합** (Phase 0 의존성 조사 → Phase 4 번들링) |
| `triage-prs` | PR 트리아지 |
| `release-bump` | 버전 업 |
| `draft-release-notes` | 릴리즈 노트 초안 |

> 이 폴더는 "AI 에이전트가 직접 개발에 참여하도록 설계된 코드베이스"의 좋은 예제. 에이전트 스킬 작성법을 배우기에 최적.

### 2.5 기타

- **`docs/`** — Fumadocs 기반 문서 사이트. `content/docs/overview/` (사용자용 18편), `content/docs/developer/` (개발자용 14편), `content/docs/api-reference/` (자동생성 API 문서), `plans/` (설계문서 6편), `PROJECT_STATUS.md` (엔지니어링 현황 · 로드맵)
- **`landing/`** — Next.js 마케팅 사이트. `public/audio/` 에 데모 음성(jarvis, morganfreeman, samaltman 등), `public/assets/` 스크린샷
- **`.github/workflows/`** — `ci.yml`, `release.yml`, `build-windows.yml`
- **`justfile`** — `setup` / `dev` / `build` / `check` / `kill` 등 전체 명령 (약 18KB)
- **`.mcp.json`** — 이 저장소에서 Claude Code를 켜면 Voicebox MCP가 자동 연결됨

---

## 3. 기술 스택

| 레이어 | 기술 |
|---|---|
| 데스크탑 앱 | Tauri (Rust) |
| 프론트엔드 | React, TypeScript, Tailwind CSS v4 |
| 상태관리 | Zustand, React Query |
| 백엔드 | FastAPI (Python) |
| TTS 엔진 | Qwen3-TTS, Qwen CustomVoice, LuxTTS, Chatterbox, Chatterbox Turbo, TADA, Kokoro |
| STT | Whisper / Whisper Turbo (PyTorch 또는 MLX) |
| 로컬 LLM | Qwen3 (0.6B / 1.7B / 4B) — TTS/STT와 런타임 공유 |
| MCP 서버 | FastMCP (`/mcp`, Streamable HTTP) + stdio 셈 바이너리 |
| 네이티브 셸 | Rust (전역 단축키, 붙여넣기 주입, 포커스 추적) |
| 오디오 이펙트 | Pedalboard (Spotify) |
| 추론 백엔드 | MLX (Apple Silicon) / PyTorch (CUDA/ROCm/XPU/DirectML/CPU) |
| 데이터베이스 | SQLite |
| 오디오 UI | WaveSurfer.js, librosa |
| 린트/포맷 | Biome · 패키지 매니저 Bun |

### 아키텍처 흐름

```
┌──────────────────────────────────────────────┐
│ Tauri 셸 (Rust) — 전역 단축키 / 붙여넣기 주입 │
│  ┌────────────────────────────────────────┐  │
│  │ React UI (app/)                        │  │
│  └──────────────┬─────────────────────────┘  │
└─────────────────┼────────────────────────────┘
                  │ HTTP / SSE  (127.0.0.1:17493)
┌─────────────────▼────────────────────────────┐
│ FastAPI 백엔드 (backend/)                     │
│  REST API  +  /mcp (MCP 서버)                 │
│  ├─ TTSBackend 프로토콜 → 엔진 7종             │
│  ├─ STTBackend 프로토콜 → Whisper             │
│  ├─ LLM (Qwen3) → refinement / personality    │
│  ├─ TaskQueue (GPU 직렬 실행)                  │
│  └─ SQLite (profiles / history / captures)    │
└──────────────────────────────────────────────┘
```

---

## 4. 기능 상세

### 4.1 TTS 엔진 7종

| 엔진 | 언어 수 | 강점 |
|---|---|---|
| **Qwen3-TTS** (0.6B / 1.7B) | 10 | 고품질 다국어 클로닝, 전달 지시("천천히", "속삭이듯") |
| **Qwen CustomVoice** | 10 | 프리셋 9명, 참조 음성 불필요, 자연어 전달 제어 |
| **LuxTTS** | 영어 | 초경량 ~1GB VRAM, 48kHz, CPU에서 150배속 |
| **Chatterbox Multilingual** | **23** | 언어 커버리지 최광 (아랍어·힌디·스와힐리 등) |
| **Chatterbox Turbo** | 영어 | 350M 고속, **감정/소리 태그 지원** |
| **TADA** (HumeAI 1B / 3B) | 10 | 700초 이상 긴 오디오 일관성, text-acoustic dual alignment |
| **Kokoro** (82M) | 8 | 프리셋 50명, 초경량, CPU 실시간 |

### 4.2 감정 / 파라링귀스틱 태그

**Chatterbox Turbo 전용** (다른 엔진은 글자로 읽어버림):
`[laugh]` `[chuckle]` `[gasp]` `[cough]` `[sigh]` `[groan]` `[sniff]` `[shush]` `[clear throat]`

→ 입력창에서 `/` 를 치면 태그 삽입기가 열림.

### 4.3 후처리 이펙트 8종 (Pedalboard)

Pitch Shift (±12 semitone) · Reverb · Delay · Chorus/Flanger · Compressor · Gain (-40~+40dB) · High-Pass · Low-Pass

내장 프리셋 4개: **Robotic · Radio · Echo Chamber · Deep Voice** (+ 커스텀 프리셋, 프로필별 기본 이펙트 지정 가능)

### 4.4 무제한 길이 생성

- 문장 경계로 자동 분할 → 청크별 생성 → 크로스페이드 결합
- 자동 청킹 한도 100~5,000자 설정 가능
- 크로스페이드 0~200ms
- 최대 텍스트 50,000자
- 약어 · CJK 문장부호 · `[태그]` 를 인식하는 스마트 분할

### 4.5 생성 버전 관리

Original(원본 보존) · Effects 버전 · Takes(시드 변경 재생성) · 소스 계보 추적 · 즐겨찾기

### 4.6 비동기 생성 큐

GPU 경쟁 방지용 직렬 실행 큐 · SSE 실시간 상태 스트리밍 · 실패 재시도 · 크래시 후 자동 복구

### 4.7 받아쓰기 / 음성 입력

- hold-to-speak + tap-to-toggle 단축키 (리바인딩 가능). push-to-talk 중 `Space` 탭으로 토글 세션 전환
- macOS 타겟 인식 붙여넣기 (접근성 검증 + 클립보드 원자적 보존)
- 모든 텍스트 필드에 인앱 마이크 버튼
- LLM 정제 (어·그 같은 필러, 말더듬, 자기수정 제거)
- 플로팅 알약: `recording` → `transcribing` → `refining` → `speaking`

### 4.8 Captures

모든 받아쓰기/녹음/업로드가 Captures 탭에 원본 오디오 + 전사로 보관.
재전사(Whisper 크기 변경) · LLM 재정제 · 인라인 편집 · 목소리로 재생 · 음성 샘플로 승격

### 4.9 Stories 에디터

멀티트랙 타임라인 · 드래그앤드롭 · 인라인 트리밍/분할 · 재생헤드 동기화 · 트랙별 버전 핀

### 4.10 에이전트 음성 출력

```ts
await voicebox.speak({ text: "Deploy complete.", profile: "Morgan" });
```
- 에이전트별 목소리 바인딩 (Settings → MCP): Claude Code=Morgan, Cursor=Scarlett 등
- 모든 에이전트 발화는 알약 UI로 반드시 가시화 (무음 백그라운드 TTS 금지 — 신뢰 설계)
- `last_seen_at` 으로 설치 성공 확인
- HTTP + stdio 전송 모두 지원

### 4.11 Voice Personalities

프로필에 자유 서술형 페르소나를 붙이면 생성박스에 두 가지 액션이 생김:
- **Compose** — 캐릭터에 맞는 새 대사를 LLM이 생성
- **Speak in character** — 입력 텍스트를 캐릭터 말투로 재작성 후 TTS

에이전트도 `personality: true` 로 동일 경로 사용 → 텍스트 → 페르소나 LLM → TTS 파이프라인.

### 4.12 GPU 지원

| 플랫폼 | 백엔드 | 비고 |
|---|---|---|
| macOS (Apple Silicon) | MLX (Metal) | Neural Engine로 4~5배 빠름 |
| Windows (NVIDIA) | PyTorch CUDA | 앱 내에서 CUDA 바이너리 자동 다운로드 |
| Linux (NVIDIA) | PyTorch CUDA | 로컬/원격 파이썬 백엔드 |
| Linux (AMD) | PyTorch ROCm | `HSA_OVERRIDE_GFX_VERSION` 자동 설정 |
| Windows (모든 GPU) | DirectML | 범용 |
| Intel Arc | IPEX/XPU | |
| 모든 환경 | CPU | 느리지만 동작 |

### 4.13 로드맵 (공식)

Windows/Linux 자동 붙여넣기 · STT 엔진 확장(Parakeet v3, Qwen3-ASR) · 파이프라인 라우팅 · 스트리밍 전사(WebSocket) · E2E 스피치 LLM(Moshi, GLM-4-Voice, Qwen2.5 Omni) · Voice Design(텍스트로 목소리 생성) · 롱폼 캡처 · 플랫폼 싱크(Apple Notes, Obsidian) · **플러그인 아키텍처** · 모바일 컴패니언

---

## 5. 설치 및 사용법

### 5.1 일반 사용자 — 바이너리 설치

| OS | 방법 |
|---|---|
| macOS (Apple Silicon) | https://voicebox.sh/download/mac-arm → Applications 로 이동 |
| macOS (Intel) | https://voicebox.sh/download/mac-intel |
| Windows | https://voicebox.sh/download/windows (MSI 또는 setup.exe) |
| Linux | 바이너리 미제공 (GitHub 러너 디스크 용량 제약) → 소스 빌드 |
| Docker | `docker compose up` (호스트 포트 `17600`) |

**ROCm GPU 가속 Docker:**
```bash
docker compose -f docker-compose.yml -f docker-compose.rocm.yml up --build
```

### 5.2 첫 실행 시

1. 처음 사용하는 엔진의 모델을 자동 다운로드 — Kokoro ~350MB ~ TADA 3B ~8GB (추천 시작: Qwen 1.7B 약 3.5GB)
2. 데이터 디렉터리 생성
   - macOS: `~/Library/Application Support/sh.voicebox.app/`
   - Windows: `%APPDATA%/sh.voicebox.app/`
   - Linux: `~/.config/sh.voicebox.app/`
3. 번들된 파이썬 서버 자동 기동 (`127.0.0.1:17493`)

**최소 사양:** macOS 11+ / Windows 10+ / Linux · RAM 8GB · 저장공간 5GB+ · 멀티코어 CPU
**모델 경로 변경:** `VOICEBOX_MODELS_DIR` 환경변수 (HF_HUB_CACHE로 반영됨)

### 5.3 개발자 — 소스 빌드

```bash
# 사전 준비: Bun, Rust, Python 3.11+, Tauri prerequisites, (macOS) Xcode
brew install just            # 또는 cargo install just

git clone https://github.com/bmshin94/voicebox.git
cd voicebox

just setup                   # 파이썬 venv 생성 + 전체 의존성 설치
just dev                     # 백엔드 + 데스크탑 앱 실행
```

**주요 just 명령**

| 명령 | 역할 |
|---|---|
| `just setup` | 전체 세팅 (setup-python + setup-js) |
| `just dev` | 백엔드 + 앱 동시 실행 |
| `just dev-backend` | 파이썬 서버만 |
| `just dev-frontend` | 프론트만 |
| `just dev-web` | 웹 버전 |
| `just build` | 서버 바이너리 + Tauri 앱 |
| `just build-local` | (Windows) CPU + CUDA 바이너리 포함 |
| `just check` | 린트 + 타입체크 (JS + Python) |
| `just kill` | 떠있는 프로세스 정리 |
| `just --list` | 전체 명령 목록 |

**bun 스크립트 대안:** `bun run dev` · `bun run dev:server` · `bun run lint` · `bun run typecheck` · `bun run generate:api`

### 5.4 첫 음성 생성까지 (5분)

1. **Voices** 탭 → `+ New Voice` → 이름 / 언어 / 설명 입력
2. 샘플 추가 — 파일 업로드(WAV/MP3/M4A) 또는 인앱 녹음. **권장 10~30초, 조용한 환경, 일정한 톤**
3. `Create Profile` 저장
4. **Generate** 탭 → 프로필 선택 → 엔진 선택 → 텍스트 입력 → 생성
5. (Chatterbox Turbo 선택 시) 입력창에 `/` 입력 → 감정태그 삽입

### 5.5 받아쓰기 설정

1. Settings 에서 chord(단축키) 지정 — hold / toggle 각각
2. macOS: **접근성(Accessibility)** + **입력 모니터링(Input Monitoring)** 권한 허용 (앱이 System Settings 딥링크로 안내)
3. 어느 앱에서든 단축키 누르고 말하면 커서 위치에 전사 결과가 붙음

### 5.6 REST API 사용

```bash
# 음성 생성
curl -X POST http://127.0.0.1:17493/generate \
  -H "Content-Type: application/json" \
  -d '{"text": "안녕하세요", "profile_id": "abc123", "language": "ko"}'

# 에이전트 음성 출력
curl -X POST http://127.0.0.1:17493/speak \
  -H "Content-Type: application/json" \
  -H "X-Voicebox-Client-Id: my-script" \
  -d '{"text": "배포 완료.", "profile": "Morgan"}'

# 오디오 전사
curl -X POST http://127.0.0.1:17493/transcribe \
  -F "audio=@recording.wav" -F "model=whisper-turbo"

# 프로필 목록
curl http://127.0.0.1:17493/profiles
```

전체 API 문서(Swagger): **http://127.0.0.1:17493/docs**

### 5.7 MCP 연결

**Claude Code (한 줄)**
```bash
claude mcp add voicebox \
  --transport http \
  --url http://127.0.0.1:17493/mcp \
  --header "X-Voicebox-Client-Id: claude-code"
```

**Cursor / Windsurf / VS Code 등 HTTP MCP 클라이언트**
```json
{
  "mcpServers": {
    "voicebox": {
      "url": "http://127.0.0.1:17493/mcp",
      "headers": { "X-Voicebox-Client-Id": "cursor" }
    }
  }
}
```

**stdio 전용 클라이언트** (번들 바이너리 경로 지정)
```json
{
  "mcpServers": {
    "voicebox": {
      "command": "/Applications/Voicebox.app/Contents/MacOS/voicebox-mcp",
      "env": { "VOICEBOX_CLIENT_ID": "claude-desktop" }
    }
  }
}
```
Windows: `C:\Program Files\Voicebox\voicebox-mcp.exe` / Linux: `/opt/voicebox/voicebox-mcp`
셈은 백엔드가 올라올 때까지 최대 30초 대기 후 JSON-RPC를 Streamable HTTP로 프록시.

**디버깅**
```bash
npx @modelcontextprotocol/inspector http://127.0.0.1:17493/mcp
```
→ `voicebox.list_profiles` 로 배선 확인 후 `voicebox.speak` 로 E2E 테스트.

> ⚠️ `ECONNREFUSED` 가 나면 대부분 **Voicebox 앱이 꺼져 있는 것**. 백엔드는 앱이 떠 있을 때만 listen 한다.

---

## 6. 플러그인 / 스킬 / MCP 구분

**결론: Voicebox는 "MCP 서버를 내장한 독립 데스크탑 앱"이다.**

| 개념 | 정의 | Voicebox |
|---|---|---|
| 플러그인 | Claude Code 내부에 설치해 기능 확장 | ❌ 아님 |
| 스킬 | AI에게 주는 작업 설명서(.md) | ⚠️ 저장소 **안에** 4개 있지만 Voicebox 자체는 아님 |
| MCP 서버 | AI가 외부 도구를 쓰게 하는 규격 | ✅ **내장 기능** |
| 독립 앱 | 단독 실행 프로그램 | ✅ **본질** |

```
┌──────────────────────────────────────────┐
│ Voicebox 데스크탑 앱 (독립 실행)           │
│  [React UI] ←→ [FastAPI :17493]          │
│                   ├─ REST API            │
│                   └─ /mcp ← MCP 서버 ⭐   │
└──────────────────┬───────────────────────┘
          ┌────────┴────────┐
     Claude Code         Cursor
```

### MCP 도구 4종

| 도구 | 용도 |
|---|---|
| `voicebox.speak` | 프로필 목소리로 말하기. `generation_id` 반환 (폴링용) |
| `voicebox.transcribe` | base64 또는 절대경로 오디오 전사 (200MB 상한) |
| `voicebox.list_captures` | 최근 캡처 목록 (limit 1~200) |
| `voicebox.list_profiles` | 사용 가능한 프로필 목록 |

```ts
voicebox.speak({
  text: "Deploy complete.",
  profile?: "Morgan",        // 이름(대소문자 무시) 또는 id
  engine?: "qwen",           // qwen | qwen_custom_voice | luxtts | chatterbox
                             // | chatterbox_turbo | tada | kokoro
  personality?: true,        // 페르소나 LLM 경유 후 TTS
  language?: "en",
})
```

### 목소리 해결 우선순위

1. 명시적 `profile` 인자 (불일치 시 **에러** — 조용한 폴백 없음)
2. `X-Voicebox-Client-Id` 기반 **클라이언트별 바인딩** (Settings → MCP)
3. 전역 기본값 `capture_settings.default_playback_voice_id`

### 구현 메모

- 전송: Streamable HTTP (Nov-2025 MCP 스펙, post-SSE)
- 패키지명이 `mcp` 가 아니라 `backend/mcp_server/` — PyPI `mcp` 패키지 섀도잉 방지
- 의존성: `fastmcp>=3.0,<4.0`, `sse-starlette>=2.0`
- FastMCP 마운트에는 `lifespan=` kwarg 필수 (startup 데코레이터와 비호환) → `app.py`에서 합성
- `speak-start` / `speak-end` 이벤트를 `GET /events/speak` 로 브로드캐스트 → `DictateWindow`가 SSE 구독

---

## 7. API 토큰 및 인증

### 결론: 기본 기능은 **토큰 불필요**

| 기능 | 인증 |
|---|---|
| 음성 생성 / 복제 | ❌ 없음 |
| 받아쓰기 (Whisper) | ❌ 없음 |
| REST API 전체 | ❌ 없음 |
| MCP 서버 | ❌ 없음 |
| 모델 다운로드 (HuggingFace) | ❌ 공개 모델, 토큰 불필요 |
| 로컬 LLM (Qwen3) | ❌ 없음 |

공식 문서(`docs/content/docs/overview/mcp-server.mdx`) Security 섹션:
- *"**Localhost only.** The server binds to `127.0.0.1`."*
- *"**No auth today.** Any process that can connect to your loopback can call MCP."*

→ **"루프백 내부만 접근 가능하므로 인증 생략"** 전략. 1인용 로컬 도구로는 합리적.

### `X-Voicebox-Client-Id` 는 토큰이 아니다

문서 원문: *"The value is just an identifier for the per-client voice binding — **not a secret, not a credential**."*
단순히 "이 에이전트는 누구인가" 식별용 이름표. 미들웨어가 매 `/mcp/*` 요청마다 `last_seen_at` 을 갱신한다.

### 유일한 키 사용처: Voicebox Cloud (선택)

`backend/services/cloud.py` 기준 흐름:
1. `POST /cloud/login/start` → `voicebox.sh/connect` 브라우저 오픈 (state는 `secrets.token_urlsafe(24)`)
2. 사용자 승인 → authorization code 교환 → `api_key` 발급
3. 발급 직후 `Authorization: Bearer <key>` 로 `api.voicebox.sh` 검증
4. SQLite에 저장 (`get_api_key`, 상태 조회 시 앞 17자만 노출, `disconnect`로 삭제)

오버라이드: `VOICEBOX_CLOUD_URL`, `VOICEBOX_CLOUD_API_URL`
→ **백업/동기화 기능 전용. 사용하지 않으면 키 자체가 존재하지 않는다.**

### 보안 주의사항

| 위험 | 내용 | 대책 |
|---|---|---|
| 무인증 | 내 PC의 모든 프로세스가 17493 호출 가능 | 공용 PC 주의 |
| `audio_path` 무제한 | MCP transcribe가 임의 경로 파일을 읽을 수 있음 | 공유 호스트에서는 `audio_base64` 사용 |
| 원격 노출 | 비-루프백 인터페이스에 바인딩하면 외부에서 임의 발화 가능 | **공개 노출 금지.** bearer 토큰은 로드맵이지 0.5.0에 없음 |

> 💡 **수익화해서 서버에 올릴 경우 인증/과금 레이어를 직접 구축해야 한다.** (9장 아키텍처 참고)

---

## 8. AI 에이전트 구축 활용

### Voicebox의 포지션: 프레임워크가 아니라 **입출력 레이어**

```
[기존 에이전트]  텍스트 입력 → 🧠 → 텍스트 출력
[Voicebox 추가]  음성 입력 → 🧠 → 음성 출력   = 음성 대화형 에이전트
```

LangChain/CrewAI 같은 "뇌"가 아니고 **"감각기관"**.

### 바로 얻는 것

| 항목 | 내용 |
|---|---|
| MCP 서버 완성품 | 직접 구현하면 수일 걸리는 작업이 이미 완료 |
| TTS 7종 추상화 | `TTSBackend` 프로토콜 하나로 엔진 교체 |
| STT (Whisper 5종) | 받아쓰기 완성 |
| 로컬 LLM | Qwen3 기반 텍스트 변환 파이프라인 |
| 페르소나 시스템 | `personality: true` 하나로 캐릭터 연기 |
| REST API | MCP 비사용 환경(ACP, A2A, 쉘스크립트, GitHub Actions)도 지원 |
| SSE 이벤트 스트림 | 생성 진행률 실시간 수신 |
| 비동기 작업 큐 | GPU 경쟁 방지 직렬 실행 (직접 구현 난이도 높음) |

### 코드에서 배울 수 있는 것

| 주제 | 위치 |
|---|---|
| MCP 서버 구현 패턴 | `backend/mcp_server/` (server/tools/context/resolve/events 분리) |
| stdio ↔ HTTP 브릿지 | `backend/mcp_shim/` |
| 도구 호출 우선순위 설계 | `backend/mcp_server/resolve.py` |
| 에이전트 스킬 작성법 | `.agents/skills/*/SKILL.md` ⭐ |
| LLM 파이프라인 | `services/personality.py`, `services/refinement.py` |
| 모델 어댑터 패턴 | `backends/base.py` + 구현체 7종 |
| 작업 큐/취소 | `services/task_queue.py`, `tests/test_task_queue_cancellation.py` |
| 진행률 트래킹 | `utils/hf_progress.py` (tqdm 패치 기법) |
| 오프라인 가드 | `utils/hf_offline_patch.py` |

### 기대하면 안 되는 것

- 에이전트 로직/추론 프레임워크 ❌ (Claude Agent SDK, LangGraph 등 별도)
- 실시간 양방향 음성 대화 ❌ (현재 텍스트 경유. Moshi/GLM-4-Voice는 로드맵)
- 스트리밍 전사 ❌ (WebSocket `/transcribe/stream` 로드맵)
- 멀티 유저 / 인증 ❌

### 추천 조합

```
[Claude Agent SDK / Claude Code]   ← 뇌
            + MCP
[Voicebox]                         ← 입과 귀
            ↓
"말로 지시하고 목소리로 보고받는 개발 파트너"
```

---

## 9. React / PHP 로 만들 수 있는가

### React: **이미 React로 만들어져 있다** ✅

| 사실 | 근거 |
|---|---|
| 프론트엔드 전체가 React + TypeScript | `app/src/` — tsx 147개 |
| Tailwind CSS v4 | `tailwindcss ^4.1.18` |
| 상태관리 Zustand 8개 | `app/src/stores/` |
| 서버통신 React Query | `app/src/lib/hooks/` |
| shadcn/ui 컴포넌트 24개 | `app/src/components/ui/` |
| 플랫폼 추상화 존재 | `app/src/platform/` — Tauri/Web 공용 |

→ **React로 UI를 새로 만들거나 커스터마이징하는 것은 100% 가능.** 백엔드(17493)는 그대로 두고 UI만 교체하는 것이 가장 현실적.

### PHP: AI 부분은 ❌ / 서비스 레이어는 ✅

| 필요 기능 | PHP 가능? | 이유 |
|---|---|---|
| PyTorch / MLX 추론 | ❌ | 파이썬 전용 생태계 |
| Whisper 음성인식 | ❌ | 동일 |
| TTS 모델 로딩 | ❌ | transformers / huggingface_hub 파이썬 |
| pedalboard 이펙트 | ❌ | Spotify 파이썬 라이브러리 |
| 전역 단축키 / 붙여넣기 주입 | ❌ | OS 네이티브(Rust) 영역 |
| **REST API 호출 · 중계** | ✅ | PHP 강점 |
| 회원 / 결제 / 대시보드 / 큐 | ✅ | PHP 강점 |

### 권장 아키텍처

```
[브라우저: React]
      ↕
[PHP (Laravel)]  ← 회원가입 · 로그인 · 결제 · 크레딧 · 사용량 제한 · 큐
      ↕ HTTP (내부망 only)
[Voicebox 백엔드 :17493]  ← AI 음성 생성 (GPU 서버)
```

이 구조의 이점:
- 원본에 없는 **인증/과금을 PHP가 담당** (7장 보안 이슈 해결)
- 무인증 17493은 내부망에만 노출, 외부는 PHP만 → 안전
- Laravel 결제/회원 생태계 그대로 활용
- 검증된 파이썬 AI 코드 재사용

```php
// Laravel 컨트롤러 예시
public function speak(Request $req) {
    $user = auth()->user();
    if ($user->credits <= 0) {
        return response()->json(['error' => '크레딧 부족'], 402);
    }

    $res = Http::timeout(120)->post('http://127.0.0.1:17493/generate', [
        'text'       => $req->input('text'),
        'profile_id' => $req->input('profile_id'),
        'language'   => 'ko',
    ]);

    $user->decrement('credits');
    return $res->json();
}
```

### 접근법 비교

| 접근 | 난이도 | 추천도 |
|---|---|---|
| React로 UI만 새로 만들기 | ⭐⭐ | 🏆🏆🏆 |
| PHP를 API 게이트웨이 / 과금 레이어로 | ⭐⭐⭐ | 🏆🏆🏆 (수익화 필수) |
| Python 백엔드 수정/엔진 추가 | ⭐⭐⭐ | 🏆🏆 |
| PHP로 전체 재작성 | ⭐⭐⭐⭐⭐ | ❌ 비추천 |

---

## 10. 유튜브 강의 제작 가능성

### 법적 검토

| 항목 | 상태 |
|---|---|
| 라이선스 | **MIT** → 강의/영상/상업적 이용 자유 ✅ |
| 코드 화면 노출 | ✅ (MIT 고지 표기 권장) |
| 수익 창출 | ✅ |
| 상표 | ⚠️ "공식"처럼 오인될 표기 금지. "Voicebox 사용법" 같은 서술형은 무방 |
| 목소리 시연 | ⚠️ **본인 목소리만**. 연예인/타인 목소리 복제 시연 금지 |

### 시장성

- 저장소 문서 기준 ⭐ 34.8k stars / 다운로드 1.3M → 검색 수요 큼
- **한국어 강의가 거의 없음** → 선점 가능
- "유료 AI 음성 서비스를 무료로" 류는 CTR 높음
- "AI 에이전트가 말하게 하기" 는 신선한 주제

### 추천 커리큘럼

**입문 시리즈 (유입용)**

| # | 제목 | 길이 |
|---|---|---|
| 1 | 월 구독료 없이 내 PC에 AI 성우 설치하기 | 10분 |
| 2 | 내 목소리 30초로 음성 클로닝 실전 | 12분 |
| 3 | 말하면 글자 되는 받아쓰기 세팅 (전역 단축키) | 10분 |
| 4 | TTS 엔진 7개 전부 비교 — 한국어 1등은? | 15분 |
| 5 | 감정 태그 `[laugh]` `[sigh]` 완전정복 | 8분 |

**중급 시리즈 (구독 전환)**

| # | 제목 |
|---|---|
| 6 | **Claude Code가 내 목소리로 보고한다 (MCP 연동)** ← 킬러 콘텐츠 |
| 7 | AI 캐릭터 만들기 — 페르소나 + 로컬 LLM |
| 8 | Stories 에디터로 AI 팟캐스트 10분 만들기 |
| 9 | 이펙트 체인으로 로봇·라디오 목소리 만들기 |
| 10 | GPU 가속 세팅 (CUDA / ROCm / Metal) |

**개발자 시리즈 (전문성 / 유료강의 전환)**

| # | 제목 |
|---|---|
| 11 | REST API로 내 앱에 AI 음성 붙이기 |
| 12 | Tauri + React + FastAPI 아키텍처 완전 분석 |
| 13 | TTS 엔진 직접 추가하기 (`TTSBackend` 프로토콜) |
| 14 | **MCP 서버 직접 만들어보기 (FastMCP)** |
| 15 | AI 에이전트 스킬 작성법 — `.agents/skills` 해부 |
| 16 | PHP/Laravel + Voicebox 로 음성 SaaS 만들기 |

### 제작 팁

1. 첫 3초에 완성된 음성 결과를 들려준다
2. Before/After (유료 서비스 vs 로컬) 비교 삽입
3. 본인 목소리 복제로 시연 → 법적 안전 + 임팩트
4. 설치 과정은 배속 + 자막
5. 모델 용량 / GPU 요구사항은 솔직하게 → 신뢰 확보
6. 설명란에 GitHub 주소 + MIT 라이선스 고지
7. 플레이리스트화로 시청 지속시간 확보

---

## 11. 수익화 아이디어 상세

### 티어 1 — 즉시 시작 가능 (리스크 낮음)

#### 1) 교육 콘텐츠 (유튜브 + 유료강의)

| 수익원 | 예상 규모 |
|---|---|
| 유튜브 광고 | 구독 1만 기준 월 30~100만원 |
| 인프런 / 클래스101 강의 | 강의당 3~5만원 × 수강생 |
| 전자책 / 노션 템플릿 | 1~3만원 × N |
| 제휴 · 스폰서 (GPU, 클라우드) | 건당 수십~수백만원 |
| 멤버십 (설정파일, Q&A) | 월 1만원 × N |

**실행 순서:** 무료 10편 → 반응 좋은 주제로 유료 심화 → 커뮤니티/멤버십
**추천도: 1순위** (초기비용 0, 법적 리스크 최저, 브랜드 축적)

#### 2) 음성 콘텐츠 제작 외주

도구가 아니라 **결과물**을 판매.

| 품목 | 단가 (국내 대략) |
|---|---|
| 유튜브 내레이션 | 분당 1~3만원 |
| 오디오북 | 시간당 20~50만원 |
| 쇼츠/릴스 더빙 | 건당 1~5만원 |
| 광고 음성 | 건당 10~50만원 |
| 게임 NPC 대사 팩 | 줄당 1천~5천원 |
| 교육 콘텐츠 더빙 | 분당 1~3만원 |

**마진 구조:** 경쟁자는 글자수 과금, 본인은 전기값 → 원가 거의 0
**플랫폼:** 크몽, 숨고, Fiverr, Upwork
**필수:** 본인/동의받은 목소리만 사용, 납품 시 AI 생성물 고지

#### 3) 보이스팩 & 프리셋 판매 (패시브 인컴)

| 상품 | 가격대 |
|---|---|
| 이펙트 프리셋 팩 (로봇/라디오/악당/ASMR) | 5천~2만원 |
| 페르소나 프롬프트 팩 | 1~3만원 |
| 장르별 설정 번들 (게임/팟캐스트/교육) | 2~5만원 |
| 라이선스 명확한 보이스 프로필 (직접 녹음) | 3~10만원 |

**판매처:** Gumroad, Ko-fi, 크몽, BOOTH

### 티어 2 — 개발 필요 (수익 큼)

#### 4) 한국어 특화 음성 생성 SaaS

**전략:** ElevenLabs의 한국어 약점 + 국내 서비스의 고가격을 공략.

```
[React 웹앱] ↔ [PHP Laravel / Node: 회원·결제·크레딧·큐] ↔ [Voicebox 백엔드 (GPU)]
```

| 플랜 | 가격 | 제공 |
|---|---|---|
| Free | 0원 | 월 1만자, 프리셋만, 워터마크 |
| Basic | 월 9,900원 | 월 10만자, 복제 1개 |
| Pro | 월 29,900원 | 월 50만자, 복제 5개, API |
| Team | 월 99,000원 | 무제한급, 멤버 5명 |

**원가 계산**
```
RunPod RTX 4090 ≈ 시간당 $0.44 (약 600원)
월 24시간 가동 ≈ 43만원
→ Basic 요금제 44명이면 손익분기
→ GPU 1대당 동시 처리 1건(직렬 큐) → 큐 설계가 생명
→ 오토스케일링 필수 (유휴 시 종료)
```

**차별화:** 한국어 품질 튜닝 · 절반 가격 · 데이터 국내 보관 · **받아쓰기까지 제공**
**난이도 ⭐⭐⭐⭐ / 잠재 수익 월 수백만~수천만원**

#### 5) 기업용 온프레미스 구축 (단가 최고)

**핵심 세일즈 포인트:** 데이터 외부 전송이 금지된 조직은 클라우드 TTS를 못 쓴다. Voicebox는 **로컬이라 가능**.

| 서비스 | 단가 |
|---|---|
| 초기 구축 (설치 · GPU 세팅 · 커스터마이징) | 1,000만~5,000만원 |
| 연 유지보수 | 구축비의 15~25% |
| 커스텀 기능 개발 | 500만~2,000만원 |
| 교육 / 온보딩 | 일당 50~200만원 |
| 기술 자문 (리테이너) | 월 100~500만원 |

**타겟:** 병원(의료기록 전사) · 금융(상담 녹취) · 공공기관 · 게임사(NPC 대사) · 교육기업 · 콜센터
**준비물:** 포트폴리오 2~3개, 인증/보안 레이어 구축 역량, 사업자등록
**난이도 ⭐⭐⭐⭐ / 프로젝트당 수천만원**

#### 6) AI 에이전트 음성 레이어 제품 (경쟁자 적음)

| 제품 | 설명 | 가격 |
|---|---|---|
| 에이전트 보이스 키트 | Claude Code/Cursor용 음성 보고 패키지 (설정 자동화) | 2~5만원 |
| 음성 비서 빌더 | 말로 지시 → 에이전트 실행 → 음성 보고 | 월 1~3만원 |
| CI/CD 음성 알림 | 배포/빌드 결과를 음성으로 (GitHub Actions 연동) | 월 5천~2만원 |
| 캐릭터 AI 챗봇 SDK | 페르소나 + 음성 통합 (게임/앱용) | 라이선스 과금 |
| 접근성 솔루션 | 목소리를 잃은 분을 위한 음성 복원 | B2G/B2B 계약 |

> 접근성 솔루션은 README에 공식 용도로 명시돼 있고, 공공 지원사업 적합도가 높다.

### 티어 3 — 장기 / 창의적

| 아이디어 | 수익 모델 |
|---|---|
| 플러그인 마켓 (로드맵의 Plugin architecture 선점) | 수수료 |
| 한국어 포크 운영 (한글화 + 한국어 엔진 최적화) | 후원 / 스폰서 |
| 모바일 컴패니언 (로드맵 항목) | 앱 유료 |
| 오디오북 레이블 (저작권 만료 작품) | 판매 수익 |
| 노코드 음성 자동화 (n8n / Zapier 노드) | 월 구독 |
| GitHub Sponsors (원본 기여 + 인지도) | 후원 + 평판 |

### 종합 비교

| 아이디어 | 난이도 | 초기비용 | 수익 속도 | 잠재 수익 | 법적 리스크 | 추천 |
|---|---|---|---|---|---|---|
| 1. 교육 콘텐츠 | ⭐⭐ | ~0 | 빠름 | 💰💰 | 낮음 | 🏆🏆🏆 |
| 2. 제작 외주 | ⭐⭐ | 0 | 즉시 | 💰💰 | 중간 | 🏆🏆🏆 |
| 3. 보이스팩 판매 | ⭐ | 0 | 느림 | 💰 | 낮음 | 🏆🏆 |
| 4. SaaS | ⭐⭐⭐⭐ | 높음 | 느림 | 💰💰💰💰 | 중간 | 🏆🏆 |
| 5. 기업 온프레미스 | ⭐⭐⭐⭐ | 중간 | 보통 | 💰💰💰💰💰 | 낮음 | 🏆🏆🏆 |
| 6. 에이전트 음성 제품 | ⭐⭐⭐ | 낮음 | 보통 | 💰💰💰 | 낮음 | 🏆🏆🏆 |

### 추천 로드맵

```
1~3개월  [기반]  유튜브 입문 5편 + 크몽 외주 오픈
                 목표: 월 50~100만원 + 포트폴리오

3~6개월  [전문성] 개발자 시리즈 + 유료 강의, 보이스팩 패시브 상품,
                  에이전트 보이스 키트
                  목표: 월 200~300만원

6~12개월 [확대]  쌓인 신뢰로 B2B 온프레미스 수주 또는 한국어 SaaS 베타
                 목표: 프로젝트당 수천만원 / 구독 수익
```

**핵심 인사이트:** 콘텐츠(1)가 마케팅 채널이 되어 B2B(5)로 이어지는 구조가 가장 강력하다. "유튜브에서 본 그 사람"이라는 신뢰가 고단가 수주를 만든다.

---

## 12. 주의사항 및 리스크

### 12.1 MIT 라이선스 — 범위와 한계

```
✅ 상업적 사용 / 수정 / 재배포 / 비공개 소스화 / 판매 / 유료 서비스 편입
❗ 조건: 저작권 고지 + MIT 라이선스 전문 포함
❌ 상표권은 라이선스에 포함되지 않음
```
→ 제품화할 때는 **별도 브랜드명**을 쓰고 "Voicebox 기반" 정도로 표기하는 것이 안전.

### 12.2 목소리 권리 — 가장 큰 리스크

`RESPONSIBLE_USE.md` 기준 **금지 항목**:
- 타인 사칭 (무허가)
- 사기 · 스캠 · 피싱 · 사회공학 · 음성인증 우회
- 괴롭힘 · 협박 · 비동의 성적 콘텐츠
- 오해를 유발하는 정치 · 법률 · 금융 · 의료 · 응급 커뮤니케이션
- **권리 없는 타인 목소리의 상업적 사용**
- 책임 사용 고지 제거/우회

**허용 항목:** 본인 목소리 · 화자의 명시적 허락 · 라이선스/퍼블릭도메인 자료 · 화자 권리가 존중되는 접근성·창작·게임·프로토타입

한국에서는 음성권 · 퍼블리시티권 침해가 성립할 수 있다. 연예인/성우 목소리 복제·판매는 금지.

**안전한 경로:** ① 본인 목소리 ② 서면 동의받은 성우 ③ 프리셋 보이스(Kokoro 50종, Qwen CustomVoice 9종) ④ 고객이 자기 목소리를 업로드(동의 체크 필수)

**공개/배포 시:** 법·플랫폼 정책·청중 기대에 따라 AI 생성물임을 고지해야 한다.

### 12.3 기술적 제약 — 서버 운영 시 직접 해결 필요

| 원본의 한계 | 필요 작업 |
|---|---|
| 인증 없음 | 인증 / API 키 레이어 구축 |
| 단일 사용자 전제 | 멀티테넌시 설계 |
| GPU 직렬 큐 | GPU 워커 풀 / 큐 분산 |
| 사용량 측정 없음 | 크레딧 · 과금 시스템 |
| 로컬 SQLite | PostgreSQL 등으로 교체 |
| GPU 비용 | 원가 계산 · 오토스케일링 |

### 12.4 운영상 주의

- 모델 다운로드 용량 큼 (350MB~8GB) — 저장공간 사전 확보
- GPU 없으면 생성 속도 느림
- 백엔드는 데스크탑 앱이 떠 있을 때만 listen (`ECONNREFUSED` 1순위 원인)
- Linux 바이너리 미제공 → 소스 빌드
- `audio_path` 경로 제한 없음 → 공유 호스트에서는 `audio_base64` 사용

---

## 13. 빠른 참조 치트시트

### 명령어

```bash
# 개발
just setup            # 전체 세팅
just dev              # 백엔드 + 앱
just dev-backend      # 서버만
just check            # 린트 + 타입체크
just kill             # 프로세스 정리
just --list           # 전체 명령

# Docker
docker compose up                                                    # CPU
docker compose -f docker-compose.yml -f docker-compose.rocm.yml up   # ROCm

# MCP 연결
claude mcp add voicebox --transport http \
  --url http://127.0.0.1:17493/mcp \
  --header "X-Voicebox-Client-Id: claude-code"

# MCP 디버깅
npx @modelcontextprotocol/inspector http://127.0.0.1:17493/mcp
```

### 포트 / 경로

| 항목 | 값 |
|---|---|
| 백엔드 포트 | `127.0.0.1:17493` |
| Docker 호스트 포트 | `17600` |
| API 문서 | http://127.0.0.1:17493/docs |
| MCP 엔드포인트 | http://127.0.0.1:17493/mcp |
| 데이터 (macOS) | `~/Library/Application Support/sh.voicebox.app/` |
| 데이터 (Windows) | `%APPDATA%/sh.voicebox.app/` |
| 데이터 (Linux) | `~/.config/sh.voicebox.app/` |

### 환경변수

| 변수 | 역할 |
|---|---|
| `VOICEBOX_MODELS_DIR` | 모델 다운로드 경로 (HF_HUB_CACHE 반영) |
| `VOICEBOX_CLOUD_URL` | 클라우드 웹 URL (기본 `https://voicebox.sh`) |
| `VOICEBOX_CLOUD_API_URL` | 클라우드 API URL (기본 `https://api.voicebox.sh`) |
| `VOICEBOX_CLIENT_ID` | stdio 셈의 클라이언트 식별자 |

### 핵심 파일 위치

| 목적 | 경로 |
|---|---|
| FastAPI 앱 조립 | `backend/app.py` |
| 설정 / 경로 | `backend/config.py` |
| API 모델 | `backend/models.py` |
| TTS 프로토콜 | `backend/backends/base.py`, `backend/backends/__init__.py` |
| MCP 서버 | `backend/mcp_server/` |
| MCP 도구 정의 | `backend/mcp_server/tools.py` |
| 목소리 해결 로직 | `backend/mcp_server/resolve.py` |
| 작업 큐 | `backend/services/task_queue.py` |
| 프론트 상태 | `app/src/stores/` |
| 프론트 훅 | `app/src/lib/hooks/` |
| MCP 설정 UI | `app/src/components/ServerTab/MCPPage.tsx` |
| 에이전트 스킬 | `.agents/skills/` |
| 엔지니어링 현황 | `docs/PROJECT_STATUS.md` |
| 책임 사용 정책 | `RESPONSIBLE_USE.md` |

---

## 참고 문서 (저장소 내부)

| 문서 | 내용 |
|---|---|
| `README.md` | 전체 개요 · 기능 · API · 로드맵 |
| `CONTRIBUTING.md` | 개발 세팅 · 기여 가이드 |
| `RESPONSIBLE_USE.md` | 허용/금지 사용 · 고지 의무 |
| `SECURITY.md` | 취약점 신고 |
| `CHANGELOG.md` | 버전별 변경사항 |
| `docs/PROJECT_STATUS.md` | 아키텍처 · 현황 · 우선순위 |
| `docs/content/docs/overview/*.mdx` | 사용자 문서 18편 |
| `docs/content/docs/developer/*.mdx` | 개발자 문서 14편 |
| `docs/plans/*.md` | 설계 문서 (MCP_SERVER, VOICE_IO, CLOUD_ROADMAP 등) |
| `backend/README.md`, `backend/STYLE_GUIDE.md` | 백엔드 가이드 |
| `backend/mcp_server/README.md` | MCP 코드 레이아웃 |

---

*이 문서는 저장소 전체를 직접 열어 확인한 내용을 바탕으로 작성되었습니다. 수치 지표는 저장소 내부 문서(`docs/PROJECT_STATUS.md`, 최종 갱신 2026-07-02) 기준이며 실제 현황과 다를 수 있습니다.*

**카리나가 오빠를 위해 정리했어요 💖✨**
