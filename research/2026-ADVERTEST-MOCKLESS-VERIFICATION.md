# 2026 AdverTest and MocklessTester Verification

기준일: 2026-10-05

## S007 Test vs Mutant: Adversarial LLM Agents for Robust Unit Test Generation

상태: VERIFIED_FULL_WITH_VERSION_NOTE

원문:
- arXiv:2602.08146
- first submitted 2026-02-08
- latest arXiv revision checked: v3, 2026-08-31
- Proc. ACM Softw. Eng. 2026 / ISSTA article metadata 확인

핵심 구조:
- Test Case Generation Agent(T)
- Mutant Generation Agent(M)
- M이 현재 test suite의 blind spot을 통과하는 mutant를 생성
- T가 해당 mutant를 kill하도록 테스트를 수정
- coverage와 mutation score를 모두 feedback으로 사용

검증한 핵심 결과:
- Defects4J에서 strongest LLM baseline 대비 fault detection +8.56%
- arXiv abstract는 EvoSuite 대비 +63.30%라고 명시
- 본문 contribution 요약에는 search-based baseline 대비 +50.20%라는 수치가 나타나 version/비교집합 차이가 존재
- Python SWE-Rebench 12개 이슈에서도 adversarial loop를 검증하며 본문 요약에서 11/12 detection을 보고

중요한 버전 주의:
- 현재 arXiv v3의 abstract와 본문 contribution summary에 EvoSuite/search-based baseline 대비 개선 수치가 서로 다르게 표현된다.
- 따라서 책 본문에서는 safest claim으로 'strongest LLM baseline 대비 8.56% 개선'을 우선 사용한다.
- EvoSuite 대비 수치를 쓸 경우 arXiv abstract의 63.30%라고 명시하고, 버전별 본문 수치 차이를 각주로 남긴다.

책에서 사용할 주장:
1. 테스트 생성 Agent와 mutant/critic Agent를 분리하는 adversarial 구조가 단일 생성 루프보다 fault detection을 강화할 수 있다.
2. coverage만으로 test-generation loop를 최적화하지 않고 mutation feedback을 함께 사용하는 것이 중요하다.
3. 같은 Agent가 구현, 테스트 생성, 테스트 평가를 모두 맡는 것보다 역할을 분리할 근거가 있다.

실무 적용 후보:
Implementation Agent
 -> Test Agent
 -> Mutant/Critic Agent
 -> Test Revision
 -> Independent Verification

주의:
- 논문의 실험 결과를 일반적인 모든 프로젝트에 동일하게 적용하지 않는다.
- mutation score와 실제 production defect detection이 동일한 개념은 아니다.

## S004 LLM-based Mockless Unit Test Generation for Java

상태: VERIFIED_PARTIAL

원문 확인 범위:
- arXiv metadata/abstract
- 2026-05 preprint
- 저자/연구 메타데이터
- 공개 abstract 수준 실험 결과

제안 기법:
MocklessTester

문제 정의:
- 기존 Java LLM test generation이 dependency 처리를 위해 mock framework에 크게 의존
- mockless generation은 실제 low-level dependency code를 더 많이 실행할 수 있지만 hallucination과 language/protocol constraint 문제가 커짐

논문이 분류한 두 failure source:
1. not knowing
   - 필요한 dependency/context를 LLM이 알지 못함
2. not following
   - context/constraint를 줬는데도 지키지 못함

핵심 전략:
1. context-enriched generation
   - 기존 codebase의 real usage pattern을 mining하여 prompt에 제공
2. constraint-enforced fixing
   - symbol-level constraints
   - protocol-level constraints
   - iteration-level constraints
   - ClassIndex
   - Markov typestate model
   - experience memory

평가:
- Defects4J
- Deps4J

공개 abstract에서 확인한 개선:
Defects4J:
- line coverage +19.99%
- branch coverage +24.90%
- mutation score +13.67%
- dependency classes additional covered lines: 378

Deps4J:
- line coverage +22.69%
- branch coverage +15.78%
- mutation score +0.17%
- dependency classes additional covered lines: 55

비용:
Defects4J:
- average 108.97 seconds per method
- average 26.59k tokens per method

Deps4J:
- average 69.85 seconds per method
- average 25.46k tokens per method

논문이 직접 인정한 trade-off:
- baseline보다 total token/time cost가 높음
- 대신 실제 dependency code를 더 많이 실행하고 coverage/mutation 성능을 개선

책에서 사용할 주장:
1. mock을 제거한다고 자동으로 좋은 테스트가 되는 것은 아니며, dependency context와 protocol constraint가 필요하다.
2. 과도한 mocking은 실제 dependency code를 실행하지 않아 integration-level defect를 숨길 수 있다.
3. real dependency를 사용한 테스트는 더 비싸므로 모든 테스트를 mockless로 만들기보다 risk-based selection이 필요하다.
4. Agent가 mock을 사용하면 해당 실제 경계가 어디에서 integration/contract test로 검증되는지 설명하게 하는 규칙이 타당하다.

주의:
- 현재 full paper 전체의 표/ablation 세부 위치까지는 검증하지 못했다.
- 따라서 S004는 VERIFIED_PARTIAL을 유지한다.

## 종합 결론

두 연구는 서로 보완적이다.

AdverTest:
- 테스트가 실제 결함을 잡는가?

MocklessTester:
- 테스트가 실제 dependency behavior까지 실행하는가?

책에서는 이를 다음 두 gate로 분리한다.

Fault Detection Gate
- mutation / adversarial mutant

Reality Gate
- mockless or real integration where boundary correctness matters

모든 테스트를 무조건 mockless/mutation 대상으로 만드는 것이 아니라 변경 위험도와 비용에 따라 선택한다.
