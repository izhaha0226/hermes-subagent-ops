# Hermes Subagent Ops

Hermes 서브에이전트, 분신술, Prime 중심 운영 모델을 정리한 공용 레포입니다.

이 레포는 단순히 "에이전트를 여러 개 띄우는 방법"을 설명하는 문서 모음이 아닙니다.
핵심은 아래 3가지를 하나의 운영 모델로 묶는 것입니다.

1. root agent(Prime)가 최종 판단권을 가진다
2. 서브에이전트/분신은 실행과 증거 수집에 집중한다
3. 24/7 자율실행은 반복이 아니라 학습 루프로 운영한다

즉, 이 레포는
- Hermes 서브에이전트를 어떻게 설계할지
- 분신을 어떻게 물리적으로 운영할지
- 완료 판단과 검증을 어떻게 통제할지
- 24시간 일하는 에이전트를 어떻게 진짜 운영 가능한 시스템으로 만들지
를 정리한 운영 레포입니다.

## 왜 이 레포가 필요한가

대부분의 "멀티 에이전트" 구조는 두 가지 문제를 가집니다.

첫째, 말뿐인 분신이 많습니다.
이름만 frontend agent, backend agent, qa agent 인데 실제로는 같은 작업 공간을 번갈아 쓰거나, 상태 추적도 안 되고, 누가 무엇을 했는지 불명확합니다.

둘째, 완료 판단이 흐립니다.
각 분신이 자기 할 일을 끝냈다고 "완료"를 주장해버리면, 최종 사용자 입장에서는 무엇이 진짜 검증된 결과인지 알 수 없습니다.

이 레포는 이 문제를 해결하기 위해 만들어졌습니다.

핵심 철학은 간단합니다.

- Prime만 완료를 선언한다
- 분신은 실행만 한다
- 물리 분신은 실제 worktree, branch, runtime 상태를 가져야 한다
- 자율실행은 바뀐 내용만 기록하지 말고 결과까지 학습해야 한다

## 이 레포가 다루는 범위

이 레포는 특정 프로젝트 전용이 아닙니다.
Ads, SNS, RFP, 백엔드 서비스, 프론트엔드 앱, 자동화 도구 등 어떤 코드베이스에도 적용 가능한 공용 운영 패턴을 다룹니다.

적용 대상 예시:
- 하나의 제품 레포를 프론트/백엔드/QA/배포 축으로 병렬 운영할 때
- 디버깅과 구현을 분리하고 싶을 때
- root agent 한 명이 모든 보고와 판단을 맡고, 분신은 내부 실행축으로만 쓰고 싶을 때
- 24/7 자율실행 크론/루프를 신뢰 가능한 방식으로 운영하고 싶을 때
- "에이전트가 일은 하는데 운영 체계가 없다"는 문제를 해결하고 싶을 때

## 핵심 개념

### 1. Prime
Prime는 root agent입니다.

Prime의 책임:
- 작업 해석
- 분신 라우팅
- 증거 검토
- 최종 검증
- 완료/보류/실패 판단
- 사용자 보고

Prime는 직접 구현할 수도 있지만, 중요한 점은 역할입니다.
Prime는 항상 최종 판단권자입니다.

### 2. Clone / Subagent / 분신
분신은 Prime의 내부 실행축입니다.

분신의 책임:
- 재현
- 구현
- 테스트
- QA
- 배포 보조
- 증거 수집

분신은 절대 스스로 완료를 선언하지 않습니다.
분신은 최대 "Prime 검토대기"까지만 올릴 수 있습니다.

### 3. Logical clone vs Physical clone
이 구분이 매우 중요합니다.

Logical clone:
- 역할 개념
- 예: debug clone, qa clone, deploy clone

Physical clone:
- 실제 git worktree
- 실제 branch
- 실제 파일 시스템 경로
- 실제 runtime/process 상태

운영에서는 physical clone이 있어야 합니다.
대시보드에 보이는데 실제 경로나 브랜치가 없으면, 그건 운영 시스템이 아니라 연출입니다.

## 운영 원칙

### Prime only completion
가장 중요한 규칙입니다.

- 완료 판단은 항상 root agent(Prime)이 한다
- 어떤 분신도 스스로 완료를 확정하지 않는다
- 분신은 증거와 결과만 Prime에게 보고한다
- 사용자에게 노출되는 최종 상태는 Prime 선언만 인정한다

이 원칙이 없으면 운영이 무너집니다.

### Evidence first
Prime는 아래 증거를 직접 확인하고 판단해야 합니다.

- 변경 파일
- 테스트 결과
- 빌드 결과
- 배포 결과
- 실서비스 검증 결과
- 남은 리스크

"아마 될 것 같다"는 완료 근거가 아닙니다.

### Real clones, not theater
분신은 실제여야 합니다.

예를 들어 frontend clone이 있다면 최소한 아래가 있어야 합니다.
- worktree path
- branch
- HEAD commit
- dirty/clean state
- 필요 시 연결된 dev server/process

### Learning loop over raw activity
24/7 자율실행에서 중요한 것은 많이 실행하는 것이 아니라, 무엇을 배웠는지 남기는 것입니다.

권장 루프:
1. 상태 확인
2. 이번 조각 선택
3. 실행
4. 검증
5. 배포선 반영
6. 실서비스 확인
7. 전후 변화 기록
8. 다음 액션 결정

## 추천 분신 구조

기본적으로 아래 5분신 구조를 권장합니다.

### frontend clone
역할:
- UI
- 페이지
- 컴포넌트
- 프론트 빌드 검증

### backend clone
역할:
- API
- DB
- 스케줄러
- 비즈니스 로직

### debug clone
역할:
- 재현
- 로그 분석
- 루트코즈 조사

### qa clone
역할:
- 회귀 확인
- 핵심 흐름 점검
- 완료 전 검증

### deploy clone
역할:
- main 반영
- healthcheck
- 실서비스 확인

권장 브랜치명:
- worktree/frontend-clone
- worktree/backend-clone
- worktree/debug-clone
- worktree/qa-clone
- worktree/deploy-clone

## 빠른 시작

### 1. worktree 디렉터리 ignore
```bash
cd <repo-root>
echo '.worktrees/' >> .gitignore
git add .gitignore
git commit -m "chore: ignore local worktrees directory"
```

### 2. worktree 생성
```bash
cd <repo-root>
mkdir -p .worktrees
git worktree add .worktrees/frontend -b worktree/frontend-clone
git worktree add .worktrees/backend -b worktree/backend-clone
git worktree add .worktrees/debug -b worktree/debug-clone
git worktree add .worktrees/qa -b worktree/qa-clone
git worktree add .worktrees/deploy -b worktree/deploy-clone
```

### 3. 생성 확인
```bash
git worktree list
git -C .worktrees/frontend branch --show-current
git -C .worktrees/backend branch --show-current
```

### 4. 작업 시작 전 상태 확인
```bash
git -C <repo-root> worktree list
git -C <repo-root> status --short --branch
git -C <repo-root>/.worktrees/frontend status --short --branch
git -C <repo-root>/.worktrees/backend status --short --branch
git -C <repo-root>/.worktrees/debug status --short --branch
git -C <repo-root>/.worktrees/qa status --short --branch
git -C <repo-root>/.worktrees/deploy status --short --branch
```

## Prime 승인 게이트

모든 분신 작업은 아래 게이트를 통과해야만 완료로 인정됩니다.

### 게이트 1. 분신 실행 완료
- 분신은 자기 역할 범위의 실행만 끝냅니다
- 이 단계에서는 `완료`라는 표현을 쓰지 않습니다
- 허용 표현 예시:
  - 수정 완료, Prime 검증 대기
  - 재현 완료, Prime 판단 대기
  - 배포 수행 완료, Prime 실서비스 확인 대기

### 게이트 2. Prime 증거 검토
Prime는 반드시 직접 확인합니다.
- 변경 파일
- 테스트/빌드 결과
- 배포 결과
- 실서비스 확인 결과
- 남은 리스크

### 게이트 3. Prime 완료 선언
Prime만 아래 상태 중 하나를 선언할 수 있습니다.
- 완료
- 조건부 완료
- 보류
- 실패

## 완료 상태 머신

권장 상태는 아래와 같습니다.

- 대기
- 실행중
- Prime 검토대기
- 완료
- 조건부 완료
- 보류
- 실패

상태 전환 규칙:
- 분신은 `Prime 검토대기`까지만 올릴 수 있음
- `완료/조건부 완료/보류/실패`는 Prime만 선언 가능
- 사용자 보고 상태도 Prime 선언만 인정

## 분신 보고 포맷

분신은 Prime에게 아래 포맷으로 보고하는 것을 권장합니다.

1. 맡은 역할
2. 수행 내용
3. 증거
4. 미검증 영역
5. Prime에게 필요한 다음 판단

예시:
- 역할: debug clone
- 수행 내용: 400 에러 재현, 원인 후보 2개 제거, 루트코즈 1개 확정
- 증거: 로그, 재현 경로, failing payload
- 미검증 영역: 프로덕션 재현 여부 미확인
- Prime 판단 필요: 수정 진행 여부

## Prime 최종 보고 포맷

Prime는 사용자에게 아래 순서로 보고하는 것을 권장합니다.

1. 문제
2. 원인
3. 이번에 바꾼 것
4. 검증 결과
5. 남은 리스크
6. 다음 액션

## 24/7 자율실행 운영 방법

24시간 일하는 에이전트를 만들 때 가장 흔한 착각은 "계속 돌고 있으면 자율적이다"입니다.
아닙니다.

진짜 자율시스템은 아래를 가져야 합니다.
- 현재 상태 인식
- 우선순위 선택
- 안전한 실행
- 검증
- 전후 비교
- 학습 축적

권장 루프:
1. 상태 확인
2. 이번 작업 조각 선택
3. 해당 분신에서 수정 또는 실행
4. QA 또는 최소 검증 수행
5. deploy/main 반영
6. 실서비스 health/live flow 확인
7. 전후 성과 또는 시스템 변화 기록
8. 다음 액션 결정

## 자주 터지는 gotcha

- worktree는 clean한데 실제 dev 서버는 다른 클론에서 떠 있음
- QA/deploy clone의 기준선이 뒤쳐져 cherry-pick 또는 raw patch가 충돌함
- 역할명만 믿고 작업했는데 실제 최신 구현은 다른 분신에 있음
- 기능 브랜치 성공을 production 성공으로 착각함
- 자율루프가 변경 로그는 남기지만 결과 로그는 안 남김

## 레포 구성

- `SKILL.md`
  - 공용 분신술 운영 스킬 본문

- `docs/prime-gate.md`
  - Prime 승인 게이트와 완료 상태 머신

- `docs/24x7-loop.md`
  - 24시간 자율실행 운영 루프

## 누구를 위한 레포인가

이 레포는 아래에 해당하는 사람에게 특히 유용합니다.

- Hermes를 단일 front door agent로 두고 싶은 사람
- 여러 서브에이전트를 쓰되 완료 판단은 하나의 root agent가 하길 원하는 사람
- 말뿐인 멀티에이전트가 아니라 실제 운영 가능한 clone 시스템을 만들고 싶은 사람
- 24/7 자율실행을 제품/운영/개발 전반에 적용하고 싶은 사람

## 한 줄 요약

Hermes Subagent Ops는 root agent(Prime) 중심으로 서브에이전트와 worktree 분신들을 정직하게 운영하기 위한 공용 운영 레포입니다.
에이전트를 많이 만드는 방법이 아니라, 실행, 검증, 배포, 학습을 한 시스템으로 묶는 방법을 다룹니다.
