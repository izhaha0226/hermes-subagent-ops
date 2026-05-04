# HAME 멀티에이전트 운영 체크포인트 — 2026-05-04

> 작성시각: 2026-05-04 19:15 KST  
> 목적: HAME(Heri AimTop Memory Engine) Phase 1~멀티에이전트 확장, 영상 제작 스킬 탐색 결과를 레포/위키에 보존한다.

## 1. 기준

- Hermes core/provider는 수정하지 않는다.
- HAME은 아직 공식 MemoryProvider가 아니라 **로컬 SQLite + CLI 기억창고**다.
- 속도 보호를 위해 매 턴 자동 주입하지 않는다.
- 필요한 프로젝트 작업 시작/재개/혼선 상황에서만 `context`/`recall`로 3~8개 정도 조회한다.
- secret/token/password/API key/credential/connection string은 원문 저장하지 않는다.
- 에이전트 공동 DB/shared write는 기본 금지한다.
- 먼저 **에이전트별 독립 HAME DB**를 사용하고, 헤리는 총괄 consolidation만 담당한다.

## 2. Phase 1 — 헤리 HAME MVP 완료

생성 위치:

- CLI: `/Users/yosiki/.hermes/aim-memory/aim_memory.py`
- DB: `/Users/yosiki/.hermes/aim-memory/aim_memory.db`
- wrapper: `/Users/yosiki/.hermes/aim-memory/hame`
- tests: `/Users/yosiki/.hermes/aim-memory/test_aim_memory.py`
- README: `/Users/yosiki/.hermes/aim-memory/README.md`

구현 기능:

- `remember`
- `recall`
- `context`
- `update`
- `archive`
- `export`
- `import`
- `consolidate`

검증:

- stdlib unittest 기준 `Ran 3 tests ... OK`
- pytest 미설치 이슈는 unittest로 전환해 해결
- secret scan 통과

## 3. Phase 2 — MEMORY/USER 이관 및 성능 정책

이관 결과:

- 기존 `~/.hermes/memories/MEMORY.md`, `USER.md` 원본은 보존
- HAME에는 복사/이관 후보만 저장
- 후보 파일: `/Users/yosiki/.hermes/aim-memory/migration_candidates_2026-05-04.json`

현재 헤리 HAME 카운트:

- ads: 12
- chief: 5
- global: 6
- hermes: 1
- homepage: 2
- ops: 4
- rfp: 2
- sns: 1

성능 결정:

- HAME을 상시 provider처럼 매 턴 붙이면 토큰 증가로 느려질 수 있다.
- SQLite 조회 자체는 빠르다.
- 실제 속도 저하는 조회 결과를 LLM 프롬프트에 과도하게 주입할 때 발생한다.
- 따라서 HAME은 “항상 켜는 기억”이 아니라 “필요할 때 꺼내 쓰는 기억창고”로 운영한다.

## 4. Phase 3 — 멀티에이전트 독립 HAME 확장

설치 완료:

- 헤리: `/Users/yosiki/.hermes/aim-memory`
- 로케: `/Users/yosiki/.hermes-ads/aim-memory`
- 데브: `/Users/yosiki/.hermes-dev/aim-memory`
- 디자인: `/Users/yosiki/.hermes-design/aim-memory`
- 메가: `/Users/yosiki/.hermes-mega/aim-memory`

각 에이전트 구성:

- `aim_memory.py`: OK
- `hame` wrapper: OK
- `aim_memory.db`: OK
- namespace seed memory: OK

검증 카운트:

- 헤리: 33개
- 로케: 1개
- 데브: 1개
- 디자인: 1개
- 메가: 1개

에이전트별 namespace:

- roke → ads
- dev → rfp
- design → homepage
- mega → global
- heri → ops/global 총괄

중요 원칙:

- 메가는 `.hermes-design`을 쓰지 않고 `.hermes-mega`를 사용한다.
- 각 에이전트는 자기 DB에만 기록한다.
- 헤리는 매일 새벽 총괄 consolidation만 확인한다.
- 공동 DB는 권한/역할경계/오염 방지 설계 전까지 만들지 않는다.

## 5. 자동 consolidation

헤리 단일 HAME:

- script: `/Users/yosiki/.hermes/scripts/hame-consolidate.sh`
- cron: `HAME daily consolidation`
- job id: `e1288e199fdd`
- schedule: 매일 04:20 KST

멀티에이전트 HAME:

- script: `/Users/yosiki/.hermes/scripts/hame-multi-agent-consolidate.sh`
- 실행 검증: `RUN_OK`
- cron: `HAME multi-agent consolidation`
- job id: `3d365d597f1f`
- schedule: 매일 04:35 KST
- 출력: local only

## 6. secret scan 결과

전체 DB secret scan:

- `openai_sk`: false
- `secret_assignment`: false

대상:

- 헤리
- 로케
- 데브
- 디자인
- 메가

## 7. 관련 스킬

생성/갱신된 HAME 스킬:

- `/Users/yosiki/.hermes/skills/devops/hame-memory/SKILL.md`
- references:
  - `references/session-2026-05-04-phase2.md`
  - `references/multi-agent-isolated-hame.md`

핵심 정책:

- HAME은 매 턴 자동 주입하지 않는다.
- 필요한 프로젝트에서만 `context`/`recall` 조회한다.
- 여러 에이전트로 확장할 때는 독립 DB 우선이다.

## 8. 영상 제작 스킬 탐색 결과

확인된 영상 제작 관련 스킬:

- `video-creation`: 전체 영상 제작 오케스트레이터
- `video-planning`: 제목, 훅, 비트 구성, 나레이션, 씬, 자막, 썸네일, 업로드 메타
- `video-editing`: 원본 인벤토리, 컷 전략, 편집, 렌더링
- `video-review`: 렌더 QA, 자막/오디오/컷/연속성 검수
- `video-frames`: ffmpeg 기반 프레임/짧은 클립 추출

Claude에 전달한 확장 방향:

- 숏폼/Reels/TikTok 전용 스킬
- 유튜브 롱폼 전용 스킬
- 광고영상/VSL 전용 스킬
- 인터뷰/강의 요약 영상 스킬
- 원본 영상 분석 스킬
- 자막/나레이션/후킹 문구 스킬
- 썸네일/타이틀 패키징 스킬
- 최종 QA/출판 체크리스트 스킬

## 9. 다음 액션

1. HAME을 provider로 상시 연결하지 말고 필요 시 조회 방식 유지
2. 각 에이전트가 실제 작업 후 자기 HAME에 handoff/decision만 기록하도록 유도
3. 헤리는 매일 multi-agent consolidation 로그만 확인
4. 공동 DB는 read-only curated bridge 설계 후 별도 승인
5. Claude가 만든 영상 제작 스킬 결과를 받으면 SkillVault 규격으로 저장/검증

## Evidence

- local HAME DB paths verified
- wrapper/CLI/DB existence verified
- consolidation script run result: `RUN_OK`
- cron job ids: `e1288e199fdd`, `3d365d597f1f`
- secret scan: no detected OpenAI key or secret assignment patterns
