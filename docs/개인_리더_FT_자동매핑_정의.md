# 개인/리더/FT 리포트 자동 매핑 정의 (MVP-1)

목적: 파싱 결과(CSV/JSON)와 Team Context를 **개인/리더/FT 리포트 템플릿**에 자동으로 채우는 규칙을 정의한다.

---

## 1) 입력(파싱 결과)
- `person_role_scores.csv` : 개인별 역할 점수(자가/관찰/최종)
- `person_top3.csv` : 개인별 Top1~3 역할
- `team_average.csv` : 역할별 팀 평균(finalScore)
- `observer_keyword_map` : 관찰자 키워드 분류표(부정/긍정)
- `individual_report_sections` : 개인 보고서에서 추출된 텍스트 요약
- `leader_interview_notes` : 리더 사전 인터뷰 요약(옵션)
- `team_context` : 팀 미션/업무/KPI/흐름/이슈

---

## 2) 출력(템플릿 필드)
리포트 JSON 스키마 기준:
- `memberReport`
- `leaderReport`
- `ftOnlyReport`

---

## 3) 개인 리포트 매핑

### A. Self vs Observer 요약
| 입력 | 변환 | 출력 |
|---|---|---|
| `individual_report_sections.selfObserverSummary` | 그대로 | `memberReport.selfObserver.summary` |
| `individual_report_sections.profileType` | 그대로 | `memberReport.selfObserver.profileType` |

### B. Top/Bottom 역할
| 입력 | 변환 | 출력 |
|---|---|---|
| `person_top3.csv` | Top1~3 | `memberReport.topRoles` |
| `person_role_scores.csv` | 최하위 1개 | `memberReport.bottomRoles` |

### C. 키워드/기여/리스크
| 입력 | 변환 | 출력 |
|---|---|---|
| `individual_report_sections.keywords` | 요약 | `memberReport.keywords` |
| `individual_report_sections.contributionScenes` | 팀 밸런스 연결 | `memberReport.contribution[]` |
| `observer_keyword_map` | Top 역할의 허용 가능한 약점 연결 | `memberReport.riskSignals[]` |

### D. 행동 제안(2~3개씩)
- 강점 강화: Top 역할 기반 행동
- 허용 가능한 약점 관리: Top 역할 약점 관리(도움 요청 포함)
- **8~9위 역할 직접 보완 제안 금지**

---

## 4) 리더 리포트 매핑

### A. 리더 Self vs Observer
| 입력 | 변환 | 출력 |
|---|---|---|
| `individual_report_sections.selfObserverSummary` | 리더 기준 | `leaderReport.selfObserver.summary` |

### B. 리더 기여/리스크
| 입력 | 변환 | 출력 |
|---|---|---|
| `individual_report_sections.leaderContribution` | 팀 운영 영향 중심 | `leaderReport.contribution[]` |
| `observer_keyword_map` | Top 역할 약점 연결 | `leaderReport.riskBlindspots[]` |

### C. 시스템으로 보완하는 리더 행동 가이드
| 입력 | 변환 | 출력 |
|---|---|---|
| `team_context` | 구조/흐름/의사결정 기반 | `leaderReport.systemGuidance[]` |

### D. 코칭 사전자료(옵션)
| 입력 | 변환 | 출력 |
|---|---|---|
| `leader_interview_notes` | 이슈→근거→가설→질문→2주 실험→체크인 | `leaderReport.coachingPrep` |

---

## 5) FT-only 리포트 매핑

### A. 퍼실리테이션 주의사항
| 입력 | 변환 | 출력 |
|---|---|---|
| `team_context` + `team_average.csv` | 팀 특성 반영 | `ftOnlyReport.facilitationCautions[]` |

### B. 참석자별 1-pager
| 입력 | 변환 | 출력 |
|---|---|---|
| `person_top3.csv` | Top3 역할 | `participantCards.topRoles` |
| `person_role_scores.csv` | Bottom1 역할 | `participantCards.bottomRole` |
| `observer_keyword_map` | 부정 bar 30% 이상 역할 | `participantCards.negativeBarOver30` |
| `individual_report_sections.negativeKeywords` | 부정 키워드 요약 | `participantCards.negativeKeywords` |
| `individual_report_sections.dropPoint` | Drop point | `participantCards.dropPoint` |

---

## 6) FT 개입 지점
- 프로파일 유형 자동 분류 결과가 애매할 때
- 리더 인터뷰 데이터 누락 시 코칭 사전자료 비활성화
- 부정 키워드 Top3 동률 발생 시 FT 최종 선택

