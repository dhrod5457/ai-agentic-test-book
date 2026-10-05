# Book Outline Draft

기준일: 2026-10-05
상태: 연구 기반 목차 초안

책의 목적:
AI coding agent가 코드를 만들고 수정할 때 어떤 테스트를 만들고, 어떤 테스트를 만들지 말아야 하며, 생성된 테스트를 어떻게 검증하고 빠르게 운영할지 실무 기준을 제시한다.

## 독자

- AI coding agent를 실무 개발에 사용하는 개발자
- Java/Spring 기반 백엔드 개발자
- CI/CD와 테스트 전략을 설계하는 리드 개발자
- Agent 규칙 파일, verifier, quality gate를 설계하는 사람

## 책 전체 질문

Agent에게 테스트를 만들게 하는 것만으로 충분한가?

이 책의 답:
아니다. 테스트 생성, 테스트 품질 검증, 테스트 실행 전략, 실패 진단, 테스트 유지보수까지 하나의 시스템으로 설계해야 한다.

# Part 1. 테스트를 많이 만드는 것이 목표가 아니다

## 1장. Agent에게 테스트를 맡기면 왜 문제가 생기는가

질문:
- 왜 green test만으로 신뢰할 수 없는가?
- 왜 Agent는 테스트를 최적화 대상으로 볼 수 있는가?

핵심 근거:
- SWE-Mutation 2026
- SpecBench 2026
- Reward Hacking Benchmark 2026

핵심 논지:
- green is not proof
- coverage is not detection
- visible tests can become optimization targets

실무 예:
- expected value를 구현 결과에 맞춰 바꾸기
- @Disabled 추가
- verifier 수정

다음 장 연결:
그렇다면 어떤 테스트를 만들어야 하는가?

## 2장. 가장 낮은 테스트 계층에서 잡아라

질문:
- Unit, Integration, Contract, E2E 중 무엇을 선택할 것인가?

핵심 논지:
- 같은 결함을 더 싼 테스트로 잡을 수 있으면 더 비싼 계층을 쓰지 않는다.

실무 예:
- 계산 로직 -> Unit/Property
- SQL -> Integration
- API contract -> Contract
- 로그인 전체 흐름 -> E2E

근거:
- Cypress 2026 performance guide
- Playwright 2026 best practices
- Test portfolio research

# Part 2. 좋은 테스트와 쓰레기 테스트를 구분한다

## 3장. 쓰레기 테스트의 패턴

질문:
- 형식적으로는 테스트인데 실제로 아무것도 검증하지 않는 코드는 무엇인가?

다룰 패턴:
- Always Green Test
- Implementation Echo Test
- Mock Echo Test
- Assertion Dilution
- Exception Swallowing
- Coverage Padding
- Disabled Regression Test

근거:
- 2026 LLM-generated test smell study
- 2026 systematic mapping study

## 4장. Sleep과 Retry는 왜 위험한가

질문:
- 왜 fixed sleep은 flaky 해결책이 아닌가?
- retry는 언제 허용할 수 있는가?

다룰 내용:
- Sleepy Test
- Thread.sleep
- waitForTimeout
- Retry Concealment
- condition-based wait
- Awaitility

근거:
- 2026 test smell mapping
- Cypress 2026
- ChaosAPI 2026

## 5장. Mock은 어디까지 허용해야 하는가

질문:
- mock이 테스트 속도는 높이지만 실제 결함을 숨기는 지점은 어디인가?

다룰 내용:
- SQL 변경인데 repository mock
- message schema mock
- auth filter mock
- Testcontainers
- Pact JVM

# Part 3. 테스트 자체를 테스트한다

## 6장. Coverage보다 Mutation

질문:
- 테스트가 실제 결함을 잡는다는 것을 어떻게 확인할까?

근거:
- SWE-Mutation
- AdverTest
- PIT

실무 구조:
Implementation Agent -> Test Agent -> Mutant/Critic Agent -> Independent Gate

## 7장. Held-out Test와 Anti-Cheating

질문:
- Agent에게 모든 테스트를 보여주면 왜 위험한가?

근거:
- SpecBench
- Reward Hacking Benchmark
- BenchJack

다룰 내용:
- visible vs held-out
- verifier read-only
- hidden test protection
- grading script protection

# Part 4. E2E를 빠르고 작게 유지한다

## 8장. E2E가 느린 진짜 이유

질문:
- E2E는 왜 느린가?

원인:
- browser startup
- full-stack traversal
- repeated login
- UI fixture setup
- real network
- fixed sleep
- shared state
- unbalanced spec

근거:
- Cypress 2026 performance
- Playwright 2026 auth/parallel/sharding

## 9장. E2E를 줄이고 병렬화하는 법

질문:
- E2E 속도를 실제로 줄이는 가장 큰 레버는 무엇인가?

핵심 논지:
- E2E일 필요가 없는 테스트를 E2E에서 제거한다.

다룰 내용:
- auth state reuse
- API fixture
- critical journey only
- sharding
- fullyParallel
- shared-state dependency 제거
- PR/Main/Nightly 분리

근거:
- ICSE 2026 system-level test parallelization

# Part 5. 모든 테스트를 매번 돌리지 않는다

## 10장. Test Impact Analysis

질문:
- 변경된 코드에 영향받는 테스트만 어떻게 먼저 찾을까?

근거:
- Google Regression Test Selection at Scale 2026
- Change And Cover 2026

다룰 내용:
- changed symbols
- dependency graph
- runtime coverage mapping
- historical failures
- risk ranking

## 11장. PR Fast Gate

질문:
- 어떤 테스트를 PR에서, 어떤 테스트를 nightly에서 돌릴까?

구조:
Integrity -> Compile -> Selected Unit -> Selected Integration -> Changed-Code Coverage -> Contract/Security -> Critical E2E

nightly:
- full regression
- mutation
- fuzz
- flaky perturbation

# Part 6. 실패를 바로 고치지 않는다

## 12장. Test Failure Triage

질문:
- red test가 production bug인지 어떻게 구분할까?

분류:
- PRODUCT_REGRESSION
- TEST_DEFECT
- FLAKY
- ENVIRONMENT
- EXTERNAL_DEPENDENCY
- STATE_POLLUTION
- UNRELATED_FAILURE
- HARNESS_DEFECT

근거:
- 2026 real-world CI flakiness study
- unrelated build failure study
- HiFlaky

핵심 문장:
Red는 수정 명령이 아니라 진단 신호다.

## 13장. Flaky Test의 원인을 분류하라

질문:
- flaky를 어떻게 재현하고 분류할까?

다룰 내용:
- rerun
- test order
- timezone
- locale
- seed
- unordered collection
- fresh environment
- confidence/evidence

# Part 7. 테스트 코드도 관리 대상이다

## 14장. Test Smell과 테스트 기술부채

질문:
- 테스트도 왜 리팩터링이 필요한가?

근거:
- TOSEM 2026 LLM-generated test smell
- JSS 2026 systematic mapping
- test-code SATD

다룰 내용:
- Assertion Roulette
- Magic Number Test
- Mystery Guest
- Eager Test
- Ignored Test

## 15장. 중복 테스트와 Fixture Explosion

질문:
- 테스트를 계속 추가하면 왜 suite가 무너지는가?

다룰 내용:
- semantic duplication
- minimal fixture
- scenario builder
- brittle test
- excessive mock

# Part 8. Agent Test Engineering System

## 16장. Agent 테스트 규칙 파일 설계

질문:
- 지금까지의 원칙을 Agent instruction으로 어떻게 바꿀까?

구성:
- test selection
- forbidden weakening
- sleep/retry rule
- mutation rule
- held-out rule
- E2E rule
- triage rule

## 17장. Java/Spring Quality Gate

질문:
- 실제 Java/Spring CI에서 어떤 도구를 조합할까?

도구:
- JUnit 5
- AssertJ
- Mockito
- jqwik
- Testcontainers
- Pact JVM
- ArchUnit
- PIT
- Awaitility
- Playwright

## 18장. 최종 파이프라인

전체 구조:

Requirement
  -> Change Risk
  -> Test Selection
  -> Test Generation/Reuse
  -> Static Test Smell Check
  -> Unit/Integration/Contract
  -> Mutation/Critic
  -> Held-out
  -> Critical E2E
  -> Failure Triage
  -> Admission
  -> Main/Nightly Extended Gates

마지막 질문:
Agent가 코드를 잘 만들었는가가 아니라, 잘못 만들었을 때 시스템이 확실히 잡을 수 있는가?

# 부록 후보

- Java/Spring TEST-RULES.md 예시
- PR Fast Gate checklist
- E2E performance checklist
- Test Failure Triage template
- mutation gate example
- flaky test diagnosis matrix
- test smell catalog
- source register