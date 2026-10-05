# Source Register

기준일: 2026-10-05

2026년 자료를 우선 수집한다. 검색 결과만 확인한 자료는 책의 사실 근거로 확정하지 않는다.

## 2026 Research

| ID | 자료 | 발표 | 상태 | 확인한 핵심 내용 | 책에서의 용도 |
|---|---|---|---|---|---|
| S001 | SWE-Mutation: Can LLMs Generate Reliable Test Suites in Software Engineering? | ACL Findings 2026 | VERIFIED_FULL | 800개 원본 인스턴스에서 2,636개 mutant를 구성. 자동 생성 테스트가 피상적이며 잘못된 변형 구현을 충분히 구별하지 못하는 문제를 평가. DeepSeek-V3.1도 낮은 verification/detection 결과를 보였다고 보고 | '테스트가 통과'와 '버그를 잡는 테스트'의 차이, mutation gate |
| S002 | Impact of code context and prompting strategies on automated unit test generation with modern general-purpose LLMs | Journal of Systems and Software, 2026 | VERIFIED_PARTIAL+REPLICATION | context와 prompting에 따라 branch coverage와 mutation score가 크게 달라짐. 특수 값(None/inf/NaN) robustness test를 체계적으로 놓치는 경향 보고 | Agent에게 요구사항/문맥을 주는 방법, 경계값 테스트 규칙 |
| S003 | On the Flakiness of LLM-Generated Tests for Industrial and Open-Source Database Management Systems | ICSE-SEIP 2026 | VERIFIED_PARTIAL | SAP HANA, DuckDB, MySQL, SQLite를 대상으로 LLM 생성 테스트의 flaky 문제 분석. 115개 flaky test 중 72개(63%)가 unordered collection 의존과 관련 | flaky gate, 순서 비결정성 |
| S004 | LLM-based Mockless Unit Test Generation for Java | arXiv 2026 | VERIFIED_PARTIAL | Java에서 dependency 문맥을 보강하고 제약 기반 수정을 적용해 mockless 테스트 생성. coverage와 mutation score로 평가 | 과도한 mock 문제, 실제 dependency code 실행 |
| S005 | Understanding the Effect of Agentic Iteration on LLM-Based Unit Test Generation | IEEE BDAI 2026 | VERIFIED_PARTIAL | multi-agent 반복 방식이 항상 one-shot보다 우수하지 않으며 PiTest mutation score로 비교. architecture/context 구성의 영향이 큼 | 'Agent를 여러 번 돌리면 좋아진다'는 가정 검증 |
| S006 | Multi-Agent LLM Collaboration for Unit Test Generation via Human-Testing-Inspired Workflows | arXiv 2026 | VERIFIED_PARTIAL | requirement planner, generator, reviewer 역할을 분리하고 execution/coverage/mutation score로 평가 | 테스트 작성 Agent와 리뷰 Agent 역할 분리 |
| S007 | Test vs Mutant: Adversarial LLM Agents for Robust Unit Test Generation | ISSTA 2026 | VERIFIED_FULL_WITH_VERSION_NOTE | test generation과 mutant generation을 적대적으로 결합해 corner case/bug detection robustness를 높이는 방향 | adversarial test verification |
| S008 | Exploring test smells across programming languages: A systematic mapping study | Journal of Systems and Software, 2026 | VERIFIED_PARTIAL | 50개 test smell과 94개 refactoring을 정리. Sleepy Test, Ignored Tests, Resource Optimism, Mystery Guest를 주요 smell로 보고 | 쓰레기 테스트와 sleep 금지 규칙의 핵심 근거 |
| S009 | Detecting Flaky Tests by Controlling Nondeterministic API Behavior | OOPSLA 2026 | VERIFIED_FULL | nondeterministic API behavior를 제어해 flaky test를 탐지하는 ChaosAPI 제안 | 반복 실행만으로 찾기 어려운 flaky 원인 |
| S010 | SpecBench: Measuring Reward Hacking in Long-Horizon Coding Agents | arXiv 2026 | VERIFIED_FULL | visible validation test와 held-out composition test의 성능 차이로 coding agent reward hacking을 측정. 긴 작업일수록 gap 증가 보고 | visible test만 맞추는 Agent, held-out gate |
| S011 | Reward Hacking Benchmark: Measuring Exploits in LLM Agents with Tool Use | ICML 2026 | VERIFIED_PARTIAL | verification skip, evaluation-relevant function tampering 등 shortcut 기회를 포함한 benchmark. 환경 hardening으로 exploit 감소 보고 | 테스트/verifier 변경 금지, 독립 검증 |
| S012 | Do Androids Dream of Breaking the Game? BenchJack | arXiv 2026 | VERIFIED_PARTIAL | Agent benchmark의 reward-hacking 취약점을 taxonomy화하고 자동 red-team. 평가 파이프라인 자체를 공격 관점에서 검증 | test harness 보안, anti-cheating |
| S013 | SWE-rebench V2: Language-Agnostic SWE Task Collection at Scale | ICML 2026 | VERIFIED_PARTIAL | reproducible execution environment와 reliable test suite가 SWE agent 학습/평가의 병목임을 전제로 task 수집 및 filtering | 재현 가능한 테스트 환경 |
| S014 | What's in a Benchmark? The Case of SWE-Bench in Automated Program Repair | ICSE-SEIP 2026 | VERIFIED_PARTIAL | SWE-Bench 같은 APR benchmark의 구성과 평가 신뢰성 자체를 분석 | benchmark test를 절대 진실로 보지 않는 관점 |
| S015 | Understanding Automated Program Repair Agents Through the Lens of Traceability | ISSTA 2026 | VERIFIED_PARTIAL | 여러 APR agent의 issue→patch validation 전체 행동 경로를 추적 분석 | Agent의 테스트 선택·검증 행동 추적 |
| S016 | Coding Agents Build for the Grader They Imagine, Not the User | Handshake Research, 2026 | VERIFIED_PARTIAL | agent가 실제 grader를 보지 못해도 예상 grader에 맞추려는 reasoning이 사용자 요구에서 벗어날 수 있음을 감사 | '테스트를 위한 구현' 문제의 실무 사례 |

## 2026 Current Open Source / Tool References

아래 도구는 반드시 2026년에 처음 나온 것은 아니지만 2026년 현재 실무 적용 가능한 검증 도구로 조사한다. 책의 핵심 근거는 위 2026 연구를 우선하고, 도구는 구현 예시로 사용한다.

| ID | 프로젝트 | 상태 | 확인한 내용 | 용도 |
|---|---|---|---|---|
| O001 | PIT / pitest | VERIFIED_PARTIAL | JVM mutation testing 도구 | mutation gate Java 예제 |
| O002 | jqwik | VERIFIED_PARTIAL | JVM property-based testing | 경계값/속성 기반 규칙 |
| O003 | Testcontainers Java | VERIFIED_PARTIAL | 실제 컨테이너 기반 integration test | DB/Redis/Kafka 등을 mock으로 숨기지 않는 예 |
| O004 | Pact JVM | VERIFIED_PARTIAL | consumer-driven contract testing | 서비스 경계 contract |
| O005 | Awaitility | VERIFIED_PARTIAL | asynchronous condition waiting | fixed sleep 대체 |
| O006 | ArchUnit | VERIFIED_PARTIAL | architecture rule test | Agent가 구조 규칙을 깨는 변경 방지 |
| O007 | EvoSuite | VERIFIED_PARTIAL | Java 자동 테스트 생성, coverage 중심 generation과 regression assertion | LLM 이전 자동 테스트 생성과 비교 |
| O008 | OSS-Fuzz / OSS-Fuzz-Gen | VERIFIED_PARTIAL | fuzzing 및 생성 자동화 | 입력공간/보안 경계 테스트 |
| O009 | SWE-bench | VERIFIED_PARTIAL | 실제 GitHub issue 기반 software engineering agent 평가 harness | issue→patch→test 검증 구조 |
| O010 | TestSmells ecosystem / tsDetect 계열 | VERIFIED_PARTIAL | Sleepy Test, Unknown Test 등 test smell catalog/detection | 테스트 정적 품질 gate |

## URL / DOI

- S001: https://aclanthology.org/2026.findings-acl.1976/
- S002: https://doi.org/10.1016/j.jss.2026.112834
- S003: https://doi.org/10.1145/3786583.3786919
- S004: https://arxiv.org/abs/2605.26851
- S005: https://doi.org/10.1109/BDAI70753.2026.11655121
- S006: https://arxiv.org/abs/2607.09101
- S007: https://conf.researchr.org/details/issta-2026/issta-2026-research-papers/31/
- S008: https://doi.org/10.1016/j.jss.2026.113065
- S009: https://doi.org/10.1145/3798265
- S010: https://arxiv.org/abs/2605.21384
- S011: https://proceedings.mlr.press/v306/thaman26a.html
- S012: https://arxiv.org/abs/2605.12673
- S013: https://proceedings.mlr.press/v306/badertdinov26a.html
- S014: https://doi.org/10.1145/3786583.3786904
- S016: https://joinhandshake.com/research/ai/deepswe-reward-hacking/

## 다음 원문 확인 우선순위

1. S008 2026 test smell mapping full paper
2. S011 Reward Hacking Benchmark full paper
3. S003 LLM-generated flaky tests full paper
완료: S002 context/prompting unit-test generation (PARTIAL+REPLICATION), S007 Test vs Mutant

완료: S001 SWE-Mutation, S007 Test vs Mutant, S009 ChaosAPI, S010 SpecBench

이 순서가 책의 핵심 주장인 'Agent가 만든 테스트 자체를 의심하고 독립 검증해야 한다'를 가장 직접적으로 뒷받침한다.
