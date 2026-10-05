# 2026 Test Smell and ChaosAPI Verification

기준일: 2026-10-05

## S008 Exploring test smells across programming languages

상태: VERIFIED_PARTIAL

확인 범위:
- Journal of Systems and Software 공식 초록/메타데이터
- 공개 preprint 메타데이터
- ResearchGate/저자 공개 정보

검증한 사실:
- systematic mapping study
- literature cutoff: 2025-08
- papers reviewed: 117
- test smells identified: 50
- refactorings identified: 94
- Java가 test smell 연구에서 가장 많이 다뤄진 언어 중 하나
- 자주 언급되거나 중요도가 높게 평가된 smell에 Sleepy Test, Ignored Tests, Resource Optimism, Mystery Guest 등이 포함됨

책에서 사용할 주장:
1. fixed sleep은 단순 스타일 문제가 아니라 기존 test-smell 문헌에서 명시적으로 다뤄지는 유지보수/신뢰성 문제다.
2. test smell은 언어와 framework 전반에 존재하며 자동 탐지와 refactoring 연구가 축적되어 있다.
3. Agent-generated test도 smell 검사의 대상이어야 한다.

주의:
- 최종 출판본 전체 본문을 현재 조사 환경에서 안정적으로 열람하지 못했다.
- 50 smells, 94 refactorings 등은 초록/공개 메타데이터 수준에서 확인한 값이다.
- 각 smell의 criticality 순위와 언어별 세부 표는 원문 전체 확인 전까지 본문에서 과도하게 일반화하지 않는다.

## S009 Detecting Flaky Tests by Controlling Nondeterministic API Behavior

상태: VERIFIED_FULL

원문:
- Proc. ACM Program. Lang. 10, OOPSLA1, Article 157
- April 2026
- 27 pages
- DOI 10.1145/3798265
- 저자 공개 PDF 전체 확인

핵심 설계:
- Java Standard Library의 nondeterministic API를 runtime instrumentation으로 감싼다.
- API specification을 위반하지 않는 범위에서 input/output을 perturb한다.
- perturbation 이후 기존 pass test가 fail하면 flaky 후보로 본다.
- source code를 직접 바꾸지 않고 dynamic bytecode instrumentation을 사용한다.

구현 범위:
- 11 perturbation strategies
- 17 Java Standard Library APIs
- strategy groups include random, time, date/sleep, locale, network, concurrency 등

known flaky dataset:
- 11 projects
- 87 known flaky tests
- prior dataset은 tests를 10,000회 rerun해 flaky failure를 수집한 자료
- ChaosAPI가 61 known flaky tests를 탐지
- 전체 534 reported tests 중 507을 manual inspection으로 true flaky 확인
- false positive: 27

정확도:
- precision 94.9%
- recall 70.1%

rerun 비교:
- 같은 filtered known dataset에서 simple rerun baseline은 28 flaky tests 탐지
- ChaosAPI single-strategy mode는 name-matched 74개, manual-confirmed known true positives 61개를 탐지
- 추가 24개 popular OSS 프로젝트에서 482 potential flaky tests를 탐지
- sampled subset에서 300개를 실제 flaky로 확인
- 같은 시간 동안 simple rerun baseline은 12개만 탐지

추가 결과:
- 논문은 총 746개의 previously unknown flaky tests를 manual confirmation했다고 보고한다.
- combined mode보다 single-strategy mode가 더 많은 flaky를 찾은 경우가 있었고, strategy 간 상호작용이 perturbation 효과를 상쇄할 수 있다고 분석한다.

대표 원인 예:
- Random API boundary
- System.currentTimeMillis/System.nanoTime 가정
- locale-sensitive behavior
- network delay
- concurrency
- unordered behavior

중요한 구현 원칙:
- fixed seed로 명시적으로 deterministic하게 만든 Random instance는 perturb하지 않는다.
- Time perturbation도 specification을 지키도록 시간이 뒤로 가지 않게 한다.
- 목표는 arbitrary chaos가 아니라 API contract 안에서 가능한 극단 동작을 노출하는 것이다.

책에서 사용할 주장:
1. flaky detection을 동일 테스트 N회 재실행에만 의존하면 많은 결함을 놓칠 수 있다.
2. nondeterminism source를 직접 흔드는 방식은 rare failure를 더 빠르게 노출할 수 있다.
3. Agent가 만든 테스트에는 time/random/locale/concurrency/network 같은 환경 축을 별도 검증할 가치가 있다.
4. fixed sleep으로 시간을 소비하기보다 virtualized/perturbed time으로 timing assumption을 드러낼 수 있다.

책의 Java 실무 적용 후보:
- PR에서는 lightweight rerun/order checks
- Main/Nightly에서는 nondeterminism perturbation
- time/random/locale/order-sensitive 코드가 변경됐을 때 targeted perturbation
- flaky 후보가 나오면 자동 retry-green 처리 대신 triage로 전달

주의:
- ChaosAPI가 모든 flaky 원인을 검출하는 것은 아니다.
- 논문의 recall도 70.1%로 100%가 아니다.
- perturbation 설정이 과도하면 false positive가 생길 수 있다고 논문이 직접 설명한다.
