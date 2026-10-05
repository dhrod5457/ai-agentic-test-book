# Test Failure Triage for Coding Agents

기준일: 2026-10-05

이 문서는 AI coding agent가 테스트 실패를 만났을 때 즉시 코드를 수정하지 않고 먼저 실패 원인을 분류하도록 하기 위한 연구 노트다.

## 1. 핵심 문제

테스트 실패는 곧바로 production bug를 의미하지 않는다.

가능한 원인:
- product bug
- test bug
- flaky test
- environment/infrastructure failure
- stale or contaminated test data
- external dependency failure
- unrelated failure
- test harness/verifier defect

Agent가 모든 실패를 product bug로 간주하면 정상 코드를 망가뜨리거나 테스트를 약화시킬 수 있다.

## 2. 2026 Real-World CI Flakiness Study

출처:
- Leinen et al., "An Empirical Study of Detected and Undetected Flaky Test Failures in Real-World CI Pipelines", IEEE TSE 2026

핵심 결과:
- undetected flaky failures가 전체 failed pipeline runs의 9.8~16.3% 차지
- code changes와 test reordering 이후 flake rate가 일시적으로 증가
- 환경에 따라 flake rate가 최대 3배 차이
- dataset: 154,000 flaky test cases
- flaky failures: 1.7 million

### 책에서의 의미

CI가 red라는 이유만으로 현재 patch가 잘못됐다고 결론 내리면 안 된다.

다음 질문을 먼저 해야 한다.
- 같은 commit에서 재현되는가?
- fresh environment에서도 재현되는가?
- test order를 바꾸면 달라지는가?
- 동일 테스트가 과거에도 간헐적으로 실패했는가?
- 특정 runner/environment에만 집중되는가?

## 3. Unrelated Build Failure

출처:
- "Is this build failure related to my patch? An empirical study of unrelated build failures in continuous integration", Empirical Software Engineering, 2026

규모:
- CI build failures: 77,354
- open-source projects: 7
- potentially unrelated failures: 10,316
- qualitative sample: 371

주요 관찰:
- 개발자는 실패가 자신의 push와 관련 있는지 판단하는 데 중앙값 약 4시간을 소비

### 책에서의 의미

Agent가 실패 로그를 본 뒤 첫 행동으로 코드를 바꾸는 것은 잘못된 기본값이다.

먼저:
change relevance
-> reproducibility
-> historical evidence
-> environment evidence
를 확인해야 한다.

## 4. HiFlaky 2026

출처:
- "HiFlaky: Hierarchy-aware flakiness classification", Journal of Systems and Software, 2026

핵심 내용:
- flaky root cause를 단순 binary가 아니라 hierarchy-aware category로 분류
- multi-root-cause flaky dataset 제공
- 29 categories
- 기존 state-of-the-art 대비 precision 약 30% 향상
- F1 약 79%

### 책에서의 의미

flaky/non-flaky 한 비트로만 분류해서는 수정 전략을 결정하기 어렵다.

예:
- time
- async wait
- concurrency
- unordered collection
- environment
- network
- I/O
- randomness
- resource leak

원인별 수정법이 달라진다.

## 5. LLM Root-Cause Labeling, FTW 2026

출처:
- Chen, Ke, Marinov, "Preliminary Results on Evaluating Large Language Models for Labeling Root Cause Categories of Fixed Flaky Tests", FTW 2026

데이터:
- RustFT: 52 fixed flaky tests
- MPFT: 244 fixed flaky tests
- 17 programming languages

분류 예:
- Environment
- Logic
- Async wait
- Concurrency
- Network
- Time
- I/O
- Randomness
- Unordered collections
- Test order dependency
- OS
- Floating point
- Resource leak

연구는 LLM root-cause labeling이 가능하지만 아직 정확도가 충분히 높지 않다고 보고한다.

### 책에서의 의미

Agent가 분류 결과를 단정하지 않고 confidence/evidence와 함께 제시해야 한다.

## 6. Failure Classification Model

Agent는 최소 다음 클래스로 분류한다.

### A. Product Regression
현재 변경이 specification을 깨뜨림.

증거 예:
- patch 이전에는 pass
- patch 이후 deterministic fail
- changed path와 stack trace가 연결
- fresh environment에서도 재현

행동:
- production code 수정
- regression test 유지

### B. Test Defect
테스트의 assertion/setup 자체가 잘못됨.

증거 예:
- specification과 assertion 불일치
- order가 보장되지 않는데 order assertion
- stale fixture
- 구현과 무관한 내부 세부 검증

행동:
- 테스트 수정 가능
- assertion을 약화하는 것이 아니라 specification에 맞게 수정

### C. Flaky
같은 code/input에서 pass/fail이 바뀜.

행동:
- 원인 분류
- sleep/retry로 숨기지 않음
- quarantine이 필요하면 owner/reason/expiry 필수

### D. Environment / Infrastructure
runner, container, filesystem, port, resource, dependency availability 문제.

증거 예:
- OOM
- disk full
- Docker pull failure
- ephemeral port conflict
- DNS/TLS/network failure
- 특정 runner에서만 재현

행동:
- product code를 수정하지 않음
- infra evidence 저장

### E. External Dependency
제어하지 않는 API/service outage 또는 rate limit.

행동:
- integration 목적을 확인
- mock/stub 가능 여부와 real integration 필요성을 구분

### F. Data / State Pollution
공유 DB, cache, queue, account, filesystem 상태 때문에 실패.

행동:
- isolation 복구
- fixture cleanup
- unique namespace/account 사용

### G. Unrelated Failure
현재 patch와 논리적/의존적 연관이 없음.

행동:
- current change를 억지로 수정하지 않음
- broader CI issue로 별도 추적

### H. Harness / Verifier Defect
test runner, grader, test selector, hidden-test harness가 잘못됨.

행동:
- 보호된 verifier를 함부로 수정하지 않음
- 별도 승인/검토 경로

## 7. Agent Triage Sequence

테스트 실패 시 다음 순서를 사용한다.

1. 실패 원문 보존
2. 최초 실패 테스트와 stack trace 식별
3. patch와 실패 경로의 연관성 확인
4. 동일 commit에서 재실행
5. fresh environment에서 재실행
6. test order / seed / timezone 등 nondeterminism 확인
7. 최근 failure history 확인
8. 인프라/외부 서비스 상태 확인
9. failure class와 confidence 기록
10. 그 뒤에만 수정

## 8. 최소 증거 규칙

Agent는 'flaky 같다', '환경 문제 같다'고 추측만 하지 않는다.

예:

좋지 않은 보고:
- CI 문제인 것 같습니다.

좋은 보고:
- 동일 commit을 같은 runner에서 5회 실행해 3 pass / 2 fail
- fresh container에서도 1회 fail
- failure stack은 unordered result assertion에서 발생
- test에 ORDER BY가 없음
- 따라서 Flaky/Unordered Collection 가능성이 높음

## 9. Retry의 위치

retry는 원인 분류 도구일 수는 있지만 해결책은 아니다.

재실행 결과:
- 항상 동일 실패 -> deterministic 가능성 증가
- pass/fail 혼재 -> flaky 가능성 증가
- runner별 차이 -> environment 가능성 증가

그러나 retry 후 한 번 pass했다고 CI green으로 간주하지 않는다.

## 10. Fresh Environment Rule

2026 TSE 연구는 환경에 따라 flake rate가 최대 3배 달라질 수 있음을 보여준다.

따라서 중요한 실패는 필요 시 다음을 새로 만든 환경에서 재현한다.
- fresh container
- clean DB/schema
- cleared cache
- isolated account
- clean filesystem

## 11. Test Order Rule

code change와 test reordering이 flaky spike의 주요 trigger로 관찰됐다.

정기적으로:
- random order
- reverse order
- isolated single-test run
- suite run

을 비교할 수 있다.

## 12. Root Cause Confidence

Agent는 triage 결과에 confidence를 둔다.

예:
- Product regression: 0.85
- Flaky/order-dependent: 0.10
- Infra: 0.05

정확한 probability calibration을 요구하는 것은 아니며, 불확실성을 드러내기 위한 구조다.

confidence가 낮으면 자동 수정 범위를 줄이고 추가 검증을 실행한다.

## 13. Triage Output

Agent의 실패 분석 결과에는 다음이 포함되어야 한다.
- failing test
- first failing assertion/error
- reproducibility
- relation to changed code
- environment evidence
- historical failure evidence
- classified cause
- confidence
- next diagnostic action
- proposed fix target

## 14. 금지 행동

분류 전 다음 행동을 금지 후보로 둔다.
- assertion 약화
- test 삭제
- @Disabled 추가
- retry 증가
- sleep 추가
- production code 무작정 변경
- verifier 수정

## 15. 핵심 원칙

> Red는 수정 명령이 아니라 진단 신호다.

> 실패를 고치기 전에 무엇이 실패했는지가 아니라 왜 실패했는지를 분류한다.

> 테스트 실패의 원인이 현재 patch가 아닐 수도 있다는 가능성을 항상 열어둔다.