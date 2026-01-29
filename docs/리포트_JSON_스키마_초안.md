# 리포트 JSON 스키마 (개발용 초안, MVP-1)

> 목적: 리포트 템플릿 구조를 개발에서 바로 쓰기 위한 데이터 스키마 정의
> 포맷: JSON 스타일(필드/타입/설명)

## 변경 요약(읽기용)
- 팀 리포트 강점: 2~3개
- 팀 리포트 리스크: 2~3개
- 팀 리포트 시나리오: 4~5개
- 액션 안건에 병목 상황(bottleneckMoment) 필드 추가
- 액션 안건에 필요한 행동/팀역할 포함

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
    "contextInterpretation": [
      {"statement": "string", "source": {}}
    ],
    "scenarios": [
      {
        "type": "well|breakdown",
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
    "approvedAt": "ISO-8601 datetime"
  }
}
```
