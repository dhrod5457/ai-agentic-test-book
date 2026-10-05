# Chapter Evidence Map

기준일: 2026-10-05

| 장 | 핵심 근거 | 상태 | 주요 주장 |
|---|---|---|---|
| 1 | SWE-Mutation, SpecBench, Reward Hacking Benchmark | VERIFIED_PARTIAL | green/visible tests만으로 신뢰 불가 |
| 2 | Cypress/Playwright 2026, portfolio research | VERIFIED_PARTIAL | 가장 낮은 테스트 계층 선택 |
| 3 | LLM-generated test smell 2026, JSS mapping | VERIFIED_PARTIAL | Agent도 test smell 생성 |
| 4 | JSS mapping, ChaosAPI, Cypress | VERIFIED_PARTIAL | sleep/retry는 flaky 은폐 가능 |
| 5 | Testcontainers, Pact, mockless UT paper | VERIFIED_PARTIAL | 실제 경계는 mock으로 숨기지 않음 |
| 6 | SWE-Mutation, AdverTest, PIT | VERIFIED_PARTIAL | coverage보다 fault detection |
| 7 | SpecBench, Reward Hacking, BenchJack | VERIFIED_PARTIAL | held-out/verifier protection |
| 8 | Cypress/Playwright 2026 | VERIFIED_PARTIAL | E2E 비용 원인 |
| 9 | Playwright sharding, ICSE parallelization | VERIFIED_PARTIAL | E2E 축소와 병렬화 |
| 10 | Google RTS 2026, ChaCo | VERIFIED_PARTIAL | impact-based selection |
| 11 | Google RTS, ChaCo, MSR AI PR coverage | VERIFIED_PARTIAL | PR Fast Gate |
| 12 | TSE CI flakiness, unrelated build failure | VERIFIED_PARTIAL | failure triage 선행 |
| 13 | HiFlaky, FTW LLM labeling, ChaosAPI | VERIFIED_PARTIAL | flaky root-cause 분류 |
| 14 | TOSEM/JSS 2026 | VERIFIED_PARTIAL | test code technical debt |
| 15 | TOSEM/JSS + 실무 패턴 | VERIFIED_PARTIAL | duplication/fixture debt |
| 16 | 앞 장 전체 종합 | SYNTHESIS | Agent instruction 설계 |
| 17 | Java testing OSS | VERIFIED_PARTIAL | Java/Spring implementation stack |
| 18 | 전체 연구 종합 | SYNTHESIS | final gate architecture |

## 원문 추가 확인 필요

- 각 논문의 exact metric/table/page 위치
- AutoCover acceptance/review workflow 세부
- TestDecision 수치 원문 위치
- Google RTS ranking feature 상세
- ChaCo patch coverage 정의
- test-code SATD category 상세

본문 집필 전에 위 항목은 VERIFIED_FULL 수준으로 끌어올린다.