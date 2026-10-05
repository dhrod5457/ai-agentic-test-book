# E2E Test Performance Research

기준일: 2026-10-05

이 문서는 AI coding agent가 E2E 테스트를 무분별하게 늘려 CI를 느리게 만드는 문제와, E2E 테스트의 속도를 개선하는 방법을 연구한다.

핵심 질문은 단순하다.

> 이 검증이 정말 E2E여야 하는가?

E2E는 실제 브라우저, 실제 서버, 실제 라우팅, 실제 네트워크와 여러 시스템 경계를 통과하기 때문에 단위·API·컴포넌트 테스트보다 느리다. 2026년 Cypress 공식 성능 가이드는 E2E 테스트가 일반적으로 가장 느린 테스트 계층이며, 잘못된 테스트 타입 선택이 가장 큰 성능 문제 중 하나라고 설명한다.

## 1. E2E가 느린 주요 이유

### 1.1 전체 스택을 실제로 거친다

브라우저가 뜨고, 페이지를 탐색하고, 서버가 응답하고, 데이터베이스와 외부 서비스까지 거친다.

하나의 assertion을 확인하기 위해 다음 비용이 누적될 수 있다.

```text
browser startup
  + browser context/page creation
  + navigation
  + authentication
  + frontend rendering
  + API
  + backend
  + DB
  + queue/cache
  + network
  + UI interaction
  + assertion
```

이 때문에 UI로 검증할 이유가 없는 업무 규칙을 E2E로 작성하면 비싼 경로를 반복해서 실행하게 된다.

### 1.2 매 테스트마다 로그인한다

UI 로그인은 다음 작업을 반복한다.

- 로그인 페이지 로드
- credential 입력
- 로그인 API
- redirect
- 세션/쿠키 생성
- 초기 화면 API 호출

Playwright 2026 문서는 인증 상태를 저장하고 재사용하면 매 테스트의 로그인 단계를 제거할 수 있다고 권장한다.

Cypress 2026 성능 문서도 authentication caching을 높은 우선순위 최적화로 둔다.

### 1.3 UI를 통해 테스트 데이터를 준비한다

예를 들어 수강신청 테스트에 학생, 과목, 학기, 수강 가능 상태를 만들기 위해 UI를 전부 클릭하면 준비 작업 자체가 테스트 본문보다 오래 걸릴 수 있다.

좋은 원칙:

> 검증 대상만 UI로 실행하고, 테스트 준비는 가능한 한 API·DB fixture·factory로 빠르게 만든다.

Cypress 공식 가이드도 programmatic state setup을 UI navigation보다 우선하는 고효율 최적화로 제시한다.

### 1.4 실제 네트워크 요청을 너무 많이 한다

E2E라는 이유로 분석 도구, 이미지 CDN, 알림, 외부 API까지 모두 실제 호출하면 대기 시간이 누적된다.

검증 대상이 아닌 외부 호출은 다음 방식으로 줄일 수 있다.

- stub
- fake server
- contract test로 분리
- third-party request 차단

단, 실제 시스템 연결 자체가 테스트 목적이라면 stub으로 숨기지 않는다.

### 1.5 fixed sleep

대표적인 성능 악화 패턴이다.

```text
click
sleep 3s
assert
```

실제 이벤트가 200ms에 끝나도 3초를 기다린다.

100개의 테스트가 각각 3초 sleep을 가지면 단순 계산으로 300초의 고정 지연이 추가된다.

또한 느린 CI에서는 3초가 충분하지 않아 flaky 문제도 해결되지 않는다.

따라서 다음 원칙을 연구 기준으로 둔다.

> 시간을 기다리지 말고 조건을 기다린다.

- DOM condition
- network response
- message arrival
- DB state
- async future completion

Cypress 공식 문서도 arbitrary wait 제거와 명시적 route wait를 권장한다.

### 1.6 불필요한 브라우저 조합

모든 PR에서 Chromium, Firefox, WebKit 전체 스위트를 모두 실행하면 실행량이 3배 가까이 증가할 수 있다.

대안:

- PR: 대표 브라우저 + critical path
- main/nightly: 전체 browser matrix

Playwright 공식 가이드는 필요한 브라우저만 설치하도록 권장하며 설치 시간과 디스크 사용량을 줄일 수 있다고 설명한다.

### 1.7 테스트 파일이 너무 크다

병렬화는 전체 테스트 수보다 분할 단위의 균형이 중요하다.

예:

```text
spec A: 30초
spec B: 40초
spec C: 50초
spec D: 12분
```

4개 worker가 있어도 마지막 12분짜리 spec 때문에 전체 시간이 12분 이상 걸린다.

Cypress는 파일 단위 load balancing을 사용하므로 긴 spec 하나가 병렬화 병목이 될 수 있다고 설명한다.

Playwright도 file-level sharding에서는 테스트 파일 크기가 불균형하면 shard 간 실행 시간이 크게 차이 날 수 있다고 문서화한다.

## 2. 가장 효과가 큰 최적화 순서

2026년 Cypress 공식 성능 가이드와 Playwright 공식 문서를 종합하면 다음 순서가 실무적으로 타당하다.

### Priority 1. 테스트 계층을 다시 나눈다

가장 먼저 해야 할 일이다.

다음은 E2E가 아니어도 된다.

- validation rule
- 계산식
- formatter
- business rule
- API response schema
- 단일 component state
- DB repository 조건
- 단일 service method

E2E로 남길 후보:

- 로그인부터 실제 업무 완료까지의 critical journey
- 여러 서비스가 연결되는 핵심 흐름
- 실제 routing/auth/session 확인
- 브라우저와 backend가 함께 맞물려야만 검증 가능한 흐름

즉 Agent 규칙은 다음과 같이 두는 것이 적절하다.

> 새 기능이라고 해서 E2E를 자동 추가하지 않는다. 가장 낮고 빠른 테스트 계층에서 요구사항을 검증하고, 전체 시스템 연결이 필요한 핵심 흐름만 E2E로 승격한다.

### Priority 2. UI setup을 제거한다

나쁜 예:

```text
UI로 관리자 로그인
→ 사용자 생성
→ 과정 생성
→ 과목 생성
→ 수강 등록
→ 테스트 시작
```

좋은 예:

```text
API/fixture로 상태 준비
→ browser는 검증 대상 화면부터 시작
```

### Priority 3. 인증 상태를 재사용한다

Playwright는 저장된 authenticated storage state를 재사용할 수 있다.

공유 상태를 변경하지 않는 테스트는 하나의 인증 상태를 재사용할 수 있다.

공유 서버 상태를 변경하는 병렬 테스트는 worker별 독립 계정을 사용하는 방식이 권장된다.

### Priority 4. fixed sleep 제거

고정 sleep을 다음으로 변환한다.

- locator 상태 기다리기
- API response 기다리기
- polling condition
- Awaitility
- event/latch
- virtual clock

### Priority 5. 병렬화

Playwright는 worker process 기반 병렬화를 지원하고, 여러 CI machine에서는 sharding을 지원한다.

Cypress도 여러 CI machine에 spec을 분산하며 역사적 실행 시간을 바탕으로 load balancing할 수 있다.

단순히 worker 수를 늘리는 것만으로 해결되지 않는다.

CPU, memory, DB connection, test account, shared data가 병렬 실행을 감당해야 한다.

### Priority 6. shard 균형

Playwright는 `fullyParallel`을 사용하면 individual test 수준에서 shard 균형을 더 잘 맞출 수 있다.

file-level shard를 사용한다면 spec 크기를 비슷하게 유지한다.

### Priority 7. CI 자원 확인

브라우저, 애플리케이션 서버, 테스트 runner, DB/container가 같은 runner에서 CPU와 memory를 경쟁할 수 있다.

Cypress 2026 공식 문서는 resource starvation이 단순한 '느림'뿐 아니라 flaky test처럼 보일 수 있다고 설명한다.

따라서 테스트 최적화 전에 다음을 측정한다.

- CPU
- memory
- disk IO
- container startup
- DB connection
- network latency

## 3. E2E 성능 예산 후보

Cypress 2026 성능 가이드의 공개 기준을 참고해 책에서는 예산 개념을 도입하는 방안을 검토한다.

예시:

| 단위 | 조사 기준 |
|---|---|
| individual E2E | 3~10초는 일반적, 10~30초는 조사 필요 |
| single spec | 3~5분부터 병목 가능성 조사 |
| 전체 suite | 규모에 따라 병렬화 전제 |
| fixed sleep | 기본 금지 |
| login UI setup | 매 테스트 반복 금지 후보 |

절대적인 표준으로 선언하지 않고 프로젝트별 baseline을 먼저 수집하도록 한다.

## 4. AI Agent 전용 E2E 규칙 후보

### 금지 후보

- 단위 테스트로 가능한 로직을 E2E로 작성
- 테스트마다 UI 로그인
- 테스트마다 UI로 fixture 생성
- `Thread.sleep`, `page.waitForTimeout`, 고정 `wait(ms)`
- flaky 해결을 위한 retry 증가
- 모든 PR에서 모든 브라우저 full suite
- 하나의 거대한 serial E2E scenario
- 독립 테스트 사이에 공유 상태 의존
- 외부 third-party 호출을 이유 없이 실제 호출
- 실패 분석을 어렵게 만드는 과도한 setup

### 필수 검토

Agent가 E2E를 추가할 때 다음 질문에 답하도록 한다.

1. 이 테스트를 API/component/integration test로 내릴 수 없는가?
2. E2E가 검증하는 시스템 경계가 무엇인가?
3. UI로 준비하는 데이터 중 API/fixture로 바꿀 수 있는 것은 무엇인가?
4. fixed sleep이 존재하는가?
5. 병렬 실행 가능한가?
6. 테스트가 다른 테스트의 상태를 공유하는가?
7. 인증을 재사용할 수 있는가?
8. 해당 flow가 실제 critical user journey인가?
9. PR마다 실행해야 하는가, nightly로 내려도 되는가?
10. 실패 시 원인을 한 테스트 안에서 추적할 수 있는가?

## 5. 테스트 실행 계층 후보

```text
PR Fast Gate
  ├─ unit
  ├─ property
  ├─ API/integration
  ├─ contract
  └─ critical E2E smoke

Main Gate
  ├─ PR tests
  └─ broader E2E

Nightly
  ├─ full E2E
  ├─ cross-browser
  ├─ mutation
  ├─ fuzz
  └─ long-running integration
```

모든 테스트를 모든 commit에 동일하게 실행하는 것이 좋은 테스트 전략은 아니다.

피드백 속도와 탐지 범위를 분리해야 한다.

## 6. 중요한 역설

E2E를 빠르게 만들겠다고 지나치게 mock하면 E2E의 의미가 사라진다.

따라서 목표는

> 모든 E2E를 fake로 바꾸는 것

이 아니라

> 정말 전체 연결을 확인해야 하는 소수의 시나리오만 E2E로 남기고, 나머지를 더 낮은 계층으로 이동하는 것

이다.

이 원칙은 Agent가 테스트 개수를 무작정 늘리는 것을 막는 중요한 품질 기준이 될 수 있다.

## 7. 2026 현행 공식 자료

- Cypress, Optimizing test performance, 2026-09-20 업데이트
  - wrong test type, repeated login, real network, CI setup, resource constraint를 주요 병목으로 분류
  - API/component로 테스트를 내리는 것을 가장 영향이 큰 개선 중 하나로 제시
  - auth caching, programmatic state setup, parallelization, arbitrary wait 제거 권장
- Cypress, Best Practices / Parallelization, 2026 현행
  - arbitrary wait 대신 명시적 network wait
  - spec file 기반 병렬화와 load balancing
- Playwright, Authentication, 2026 현행
  - signed-in state 재사용
  - state-changing parallel test에는 worker별 account 권장
- Playwright, Parallelism / Sharding, 2026 현행
  - worker 기반 병렬 실행
  - 여러 machine에 shard 분산
  - fullyParallel 사용 시 더 세밀한 shard balancing
- Playwright, Isolation, 2026 현행
  - 각 테스트에 독립 BrowserContext를 사용해 상태 누수와 cascading failure를 방지
- Playwright, Best Practices, 2026 현행
  - 필요한 browser만 CI에서 설치
  - parallelism과 sharding 활용

## 8. 책에서 강조할 문장 후보

> 느린 E2E를 빠르게 만드는 가장 좋은 방법은 E2E runner의 옵션을 만지는 것이 아니라, E2E일 필요가 없는 테스트를 E2E에서 제거하는 것이다.

> Agent는 테스트를 많이 만드는 것이 아니라, 가장 싼 계층에서 결함을 잡도록 테스트를 배치해야 한다.

> `sleep`은 기다림이 아니라 낭비일 수 있다. 비동기 테스트는 시간이 아니라 상태를 기다려야 한다.
