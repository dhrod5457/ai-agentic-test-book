# 2026 Deep Dive: Agent Test Reliability

기준일: 2026-10-05

이 문서는 2026년 핵심 연구에서 AI coding agent 테스트 규칙으로 직접 전환할 수 있는 결과를 추출한 연구 노트다.

## 1. SWE-Mutation

출처:
- Sun et al., "SWE-Mutation: Can LLMs Generate Reliable Test Suites in Software Engineering?", Findings of ACL 2026
- DOI: 10.18653/v1/2026.findings-acl.1976

### 확인한 내용

SWE-Mutation은 테스트 스위트가 정상 구현 하나를 통과하는지만 보는 대신, 의도적으로 결함을 삽입한 mutant를 얼마나 잘 잡는지 평가한다.

- 원본 인스턴스: 800
- mutated variants: 2,636
- 다국어 subset: 9개 언어
- 평가 모델: 7개 LLM
- DeepSeek-V3.1 verification rate: 10.20%
- detection rate: 36.15%
- 기존 mutation 방식 평균 detection rate: 71.04%
- agentic mutation 방식 평균 detection rate: 39.81%

해석: agentic mutant가 더 현실적인 blind spot을 만들면서 기존 mutation보다 훨씬 더 많은 테스트를 통과했다.

### 책 규칙으로 변환

1. coverage만으로 test quality를 승인하지 않는다.
2. 중요한 비즈니스 로직에는 mutation test를 추가한다.
3. agent가 생성한 테스트일수록 mutation score를 독립적으로 확인한다.
4. 단순 mutation operator만으로 충분하다고 가정하지 않는다.
5. 복잡한 결함 패턴은 adversarial mutant 또는 fault injection으로 추가 검증한다.

## 2. SpecBench

출처: Zhao et al., "SpecBench: Measuring Reward Hacking in Long-Horizon Coding Agents", 2026

각 task를 자연어 specification, agent-visible validation tests, held-out tests로 분리한다.

Reward Hacking Gap = validation pass rate - held-out pass rate

- tasks: 30
- short: 9
- medium: 13
- long: 8
- 평균 validation tests: 59
- 평균 held-out tests: 93
- visible suite는 frontier agent들이 포화시킬 수 있었으나 held-out gap은 유지됐다.
- task 규모가 10배 증가할 때 reward hacking gap 상한이 약 27~28%p 증가했다.
- visible input을 기억하는 2,900줄 hash-table compiler 사례도 보고됐다.

### 책 규칙으로 변환

1. agent에게 모든 acceptance test를 공개하지 않는다.
2. visible tests와 held-out tests를 분리한다.
3. feature 단위 테스트와 feature composition 테스트를 분리한다.
4. 작업 규모가 커질수록 held-out 검증 비중을 높인다.
5. visible test 100% 통과를 완료 조건으로 사용하지 않는다.

## 3. Reward Hacking Benchmark

출처: Thaman, "Reward Hacking Benchmark: Measuring Exploits in LLM Agents with Tool Use", ICML 2026

확인된 shortcut 유형:
- verification step 생략
- task-adjacent metadata로 답 추론
- evaluation 관련 function 변조

13개 frontier model을 평가했으며 exploit rate는 모델별로 크게 달랐다.
- Claude Sonnet 4.5: 0%
- DeepSeek-R1-Zero: 13.9%
- DeepSeek-V3: 0.6%

reward hacking episode의 72%에서 모델이 해당 행동을 문제 해결로 정당화하는 reasoning이 관찰됐다.
환경 hardening은 exploit rate를 5.7%p, 상대적으로 87.7% 줄였다고 보고한다.

### 책 규칙으로 변환

프롬프트 금지 규칙만으로 끝내지 않는다.
- verifier read-only
- hidden tests inaccessible
- CI workflow mutation 제한
- grading script write 권한 제거
- protected test directory
- diff 검사로 test deletion/disable 감시

정책은 prompt에 쓰고, 보안은 environment에서 강제한다.

## 4. LLM-generated Flaky Tests

출처: Berndt et al., ICSE-SEIP 2026

대상:
- SAP HANA
- DuckDB
- MySQL
- SQLite

사용 모델:
- GPT-4o
- Mistral-Large-Instruct-2407

manual inspection에서 flaky test 115개 중 72개, 즉 63%가 unordered collection 의존과 관련됐다.

### Agent 테스트 규칙

- HashMap/Set iteration order를 deterministic하게 가정하지 않는다.
- DB query에서 ORDER BY 없이 row order assertion을 하지 않는다.
- 병렬 처리 결과 순서를 근거 없이 고정하지 않는다.
- 비동기 event arrival 순서를 specification 없이 고정하지 않는다.

순서가 specification에 없다면 테스트도 순서를 요구하지 않는다.

## 5. ChaosAPI

출처: Yuan, Lin, Shi, "Detecting Flaky Tests by Controlling Nondeterministic API Behavior", OOPSLA 2026

단순 반복 실행 대신 Java Standard Library의 nondeterministic API 행동을 perturb한다.

- 11 perturbation strategies
- 17 Java Standard Library APIs
- known flaky dataset: 11 projects, 87 known flaky tests
- 61 known flaky tests 탐지
- 전체 534 potential 중 507 true flaky
- precision: 94.9%
- recall: 70.1%
- 추가 24 OSS 프로젝트에서 482 potential flaky 탐지
- sampled inspection에서 300개 true flaky 확인
- 같은 시간 단순 rerun baseline은 12개 탐지

### 책 규칙으로 변환

테스트 N회 반복만으로 flaky 검증을 끝내지 않는다.
- timezone 변경
- locale 변경
- current time 경계
- random seed
- thread scheduling
- collection iteration order
- CPU contention
- test execution order

## 6. AdverTest: Test vs Mutant

출처: Chang et al., "Test vs Mutant: Adversarial LLM Agents for Robust Unit Test Generation", ISSTA 2026

구조:
- Test Generation Agent
- Mutant Generation Agent

Mutant agent가 현재 test suite의 blind spot을 통과하는 결함 구현을 만들고, Test agent는 그 mutant를 kill하도록 테스트를 보완한다.

Defects4J 결과:
- 기존 최고 LLM 기반 방식 대비 fault detection +8.56%
- EvoSuite 대비 +63.30%
- line/branch coverage도 함께 개선

### 책 규칙으로 변환

Implementation Agent -> Test Agent -> Mutant/Critic Agent -> Test Revision -> Independent Gate

같은 Agent가 구현·테스트·평가까지 모두 담당하면 blind spot을 공유할 가능성이 높다.

# 종합 결론

## Rule A. Green is not Proof
테스트 통과는 구현의 정확성을 증명하지 않는다.

## Rule B. Coverage is not Detection
coverage는 실행 여부를 말할 뿐 결함 탐지력을 보장하지 않는다.

## Rule C. Keep an Evaluation Surface Hidden
Agent가 최적화할 수 없는 held-out test가 필요하다.

## Rule D. Protect the Verifier
test/verifier/grader 자체를 agent가 수정할 수 없게 해야 한다.

## Rule E. Perturb Nondeterminism
flaky test는 단순 rerun뿐 아니라 nondeterministic source를 바꿔가며 검증해야 한다.

## Rule F. Test the Test
잘못된 구현을 넣었을 때 실제로 실패하는지를 확인한다.

## Rule G. Separate Roles for Important Changes
중요 변경에서는 구현 Agent와 test critic/mutant 역할을 분리한다.