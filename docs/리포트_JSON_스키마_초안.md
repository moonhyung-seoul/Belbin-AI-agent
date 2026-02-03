# 리포트 JSON 스키마 (개발용 초안, MVP-1)

> 목적: 리포트 템플릿 구조를 개발에서 바로 쓰기 위한 데이터 스키마 정의
> 포맷: JSON 스타일(필드/타입/설명)

## 변경 요약(읽기용)
- 팀 리포트 강점: 2~3개
- 팀 리포트 리스크: 2~3개
- 팀 리포트 시나리오: 4~5개
- 액션 안건에 병목 상황(bottleneckMoment) 필드 추가
- 액션 안건에 필요한 행동/팀역할 포함
- 개인 최종점수 = 자가 1/3 + 관찰 2/3 (개인보고서 6p 기준)
- 팀 리포트에 개인-팀 비교(Top1~3 vs 팀평균) 섹션 추가
- 개인 점수가 팀 평균 대비 35% 이상 또는 200% 이상인 인원/팀역할만 표시
- 팀 평균 2종 계산(개인 평균 vs OCR 평균) -> 차이 감지 시 FT 선택
- FT 미응답 시 리포트 생성 보류
- 팀 평균 0일 때: 부재 역할 표시, 개인 점수>0이면 희소 역할 보유 플래그(높음 판단 제외)
- 팀 공동 액션 안건은 관찰자 응답 개괄의 부정 키워드 Top3를 고려해 제시

---

## 공통 메타 (모든 리포트)
```json
{
  "meta": {
    "projectId": "string",
    "projectName": "string",
    "teamName": "string",
    "generatedAt": "ISO-8601 datetime",
    "version": "string",
    "inputs": [
      {"type": "teamReportPdf", "name": "string", "ref": "string"},
      {"type": "individualReportPdf", "name": "string", "ref": "string"},
      {"type": "teamContext", "name": "string", "ref": "string"},
      {"type": "interviewNote", "name": "string", "ref": "string"}
    ],
    "reviewer": "string",
    "approvedAt": "ISO-8601 datetime"
  }
}
```

> 표시 규칙
- FT 전용 표시: generatedAt, version, reviewer, approvedAt, teamMissionSummary

---

## 근거(출처) 표기 공통 모델
```json
{
  "source": {
    "label": "string", 
    "docType": "teamReport|individualReport|teamContext|interview",
    "page": "string",
    "field": "string"
  },
  "needConfirm": false
}
```

---

# 1) 팀 리포트 (Team pack)
```json
{
  "teamReport": {
    "teamMissionSummary": {"text": "string", "source": {}, "ftOnly": true},
    "teamPersonaOneLine": {"text": "string", "source": {}},
    "observerNegativeTop3": [
      {"keyword": "string", "source": {}}
    ],
    "strengths": [
      {"title": "string", "detail": "string", "source": {}}
    ],
    "risks": [
      {"title": "string", "detail": "string", "source": {}}
    ],
    "roleBalance": {
      "missingRoles": ["string"],
      "overrepresentedRoles": ["string"],
      "summary": "string",
      "source": {}
    },
    "personTeamComparison": {
      "basis": {
        "individualFinalScoreMethod": "self_1_3 + observer_2_3",
        "scoreSource": "individualReportPage6",
        "topN": 3
      },
      "teamAverages": {
        "calcFromIndividuals": [{"role": "RI", "score": 0}],
        "ocrFromTeamReport": [{"role": "RI", "score": 0, "source": {}}],
        "diffDetected": true,
        "ftSelectionRequired": true,
        "ftSelected": "calc|ocr|pending",
        "blockedUntilSelection": true,
        "internalNoteToFT": "string"
      },
      "comparisons": [
        {
          "person": {"name": "string"},
          "topRoles": ["string"],
          "items": [
            {
              "role": "string",
              "personFinalScore": "number",
              "teamAvgScore": "number",
              "highlighted": true,
              "thresholdType": "ratio135|ratio200|none",
              "teamAvgZero": true,
              "scarceRoleFlag": true,
              "noteToFT": "string"
            }
          ],
          "source": {}
        }
      ],
      "display": { "showOnlyAboveThreshold": true }
    },
    "contextInterpretation": [
      {"statement": "string", "source": {}}
    ],
    "scenarios": [
      {
        "type": "strength_moment|risk_moment",
        "title": "string",
        "scene": "string",
        "source": {}
      }
    ],
    "actionLongList": [
      {
        "category": "decision|conflict|stress|role_gap|workflow",
        "bottleneckMoment": "string",
        "why": "string",
        "what": "string",
        "how": "string",
        "expectedImpact": "string",
        "neededBehavior": "string",
        "neededRoles": ["string"],
        "consideredNegativeKeywords": ["string"],
        "source": {}
      }
    ]
  }
}
```

- constraints
  - strengths: 2~3 items
  - risks: 2~3 items
  - scenarios: 4~5 items
  - actionLongList: 3 items (서로 다른 이슈 권장)
  - actionLongList는 관찰자 응답 개괄의 부정 키워드 Top3를 고려

- 비교 표시 규칙
  - teamAvgScore 대비 personFinalScore가 1.35 이상 또는 2.0 이상인 경우만 표시
  - teamAvgScore = 0이면 비교 계산 안 함, teamAvgZero = true
  - teamAvgScore = 0이고 personFinalScore > 0이면 scarceRoleFlag = true

---

# 2) 개인 리포트 (팀원용)
```json
{
  "memberReport": {
    "person": {"name": "string", "role": "string", "position": "string"},
    "selfObserver": {
      "summary": "string",
      "profileType": "string",
      "source": {}
    },
    "topRoles": ["string"],
    "bottomRoles": ["string"],
    "keywords": ["string"],
    "contribution": [
      {"scene": "string", "behavior": "string", "teamBalanceLink": "string", "source": {}}
    ],
    "riskSignals": [
      {"signal": "string", "linkToTopRole": "string", "teamBalanceLink": "string", "source": {}}
    ],
    "actionSuggestions": {
      "strengthen": [
        {"purpose": "string", "action": "string", "frequency": "string", "dod": "string", "partner": "string", "impact": "string"}
      ],
      "manageWeakness": [
        {"purpose": "string", "action": "string", "frequency": "string", "dod": "string", "partner": "string", "impact": "string", "helpRequest": "string"}
      ]
    }
  }
}
```

- constraints
  - actionSuggestions.strengthen: 2~3 items
  - actionSuggestions.manageWeakness: 2~3 items

---

# 3) 리더 리포트 (리더용)
```json
{
  "leaderReport": {
    "leader": {"name": "string", "role": "string", "position": "string"},
    "selfObserver": {"summary": "string", "source": {}},
    "contribution": [
      {"impact": "string", "source": {}}
    ],
    "riskBlindspots": [
      {"signal": "string", "linkToTopRole": "string", "teamBalanceLink": "string", "source": {}}
    ],
    "systemGuidance": [
      {"guidance": "string", "source": {}}
    ],
    "coachingPrep": {
      "enabled": true,
      "issue": "string",
      "evidence": "string",
      "hypothesis": "string",
      "questions": ["string"],
      "experiment2w": "string",
      "checkin": ["string"]
    }
  }
}
```

---

# 4) FT-only 리포트
```json
{
  "ftOnlyReport": {
    "facilitationCautions": [
      {"risk": "string", "response": "string", "source": {}}
    ],
    "participantCards": [
      {
        "person": {"name": "string", "role": "string", "position": "string"},
        "profileType": "string",
        "gapRoles": ["string"],
        "topRoles": [{"role": "string", "score": "number"}],
        "bottomRole": {"role": "string", "score": "number"},
        "negativeBarOver30": ["string"],
        "negativeKeywords": ["string"],
        "dropPoint": "string"
      }
    ]
  }
}
```

---

# 편집/승인 상태 모델
```json
{
  "status": {
    "draft": true,
    "editedByFT": true,
    "approved": true,
    "approvedAt": "ISO-8601 datetime",
    "pendingTeamAvgSelection": true
  }
}
```
