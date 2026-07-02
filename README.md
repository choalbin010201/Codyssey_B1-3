# Mission03. No-Code 자동화 도구 활용 과제

## 과제 개요

본 과제는 노코드 자동화 도구를 활용하여 실제로 동작하는 자동화 워크플로우를 설계하고 구현하는 것을 목표로 한다.

총 2개의 프로젝트를 수행했다.

1. **프로젝트 1: 자동화 도구 비교 구현**
   - 동일한 자동화 워크플로우를 Make와 Zapier로 각각 구현하고 비교 분석했다.

2. **프로젝트 2: 자유 주제 자동화 설계 및 구현**
   - 홍보 포스터 제작 문의 자동 분류 및 알림 시스템을 Make로 구현했다.

---

# 프로젝트 1. 자동화 도구 비교 구현

## 1. 프로젝트 개요

프로젝트 1에서는 동일한 자동화 워크플로우를 서로 다른 노코드 자동화 도구인 **Make**와 **Zapier**로 각각 구현하고, 두 도구의 특징과 장단점을 비교했다.

구현한 자동화 주제는 **문의 접수 자동 분류 및 알림 시스템**이다.

Google Form으로 문의가 접수되면 Google Sheets에 응답이 저장되고, 자동화 도구가 새 응답을 감지한다. 이후 문의 우선순위에 따라 긴급 문의와 일반 문의로 분기하고, 각 분기별로 Google Sheets의 별도 탭에 기록한 뒤 Gmail로 알림을 전송한다.

---

## 2. 자동화 워크플로우

```text
Google Form 문의 제출
↓
Google Sheets 응답 저장
↓
자동화 도구가 새 응답 감지
↓
문의 우선순위 기준 조건 분기
├─ 긴급 문의
│  ├─ 긴급문의 시트에 행 추가
│  └─ Gmail 긴급 알림 전송
└─ 일반 문의
   ├─ 일반문의 시트에 행 추가
   └─ Gmail 일반 알림 전송
```

## 3. Google Form 입력 항목

| 항목 | 설명 |
|---|---|
| 성함 | 문의자 이름 |
| 연락처 또는 이메일 | 문의자의 연락처 정보 |
| 문의 우선순위 | 긴급 / 일반 |
| 문의 상세 내용 | 문의 내용 |
| 희망 처리 방식 | 이메일 답변 / 전화 상담 |
| 테스트 구분 | Make 테스트 / Zapier 테스트 |

조건 분기의 기준은 **문의 우선순위** 필드이다.

---

## 4. Google Sheets 구성

| 시트 탭 이름 | 용도 |
|---|---|
| Form_Responses | Google Form 원본 응답 저장 |
| 긴급문의 | 긴급 문의 자동 기록 |
| 일반문의 | 일반 문의 자동 기록 |

`긴급문의`와 `일반문의` 탭의 헤더는 다음과 같이 구성했다.

```text
타임스탬프 | 성함 | 연락처 또는 이메일 | 문의 우선순위 | 문의 상세 내용 | 희망 처리 방식 | 테스트 구분 | 처리상태
```

---

## 5. Make 구현

### Make 워크플로우 구조

```text
Google Sheets - Watch New Rows
↓
Router
├─ 긴급 문의 필터
│  ├─ Google Sheets - Add a Row → 긴급문의
│  └─ Gmail - Send an Email → 긴급 알림
└─ 일반 문의 필터
   ├─ Google Sheets - Add a Row → 일반문의
   └─ Gmail - Send an Email → 일반 알림
```

### Make 구성 요소

| 구분 | 내용 |
|---|---|
| Trigger | Google Sheets - Watch New Rows |
| Router | 긴급 문의 / 일반 문의 분기 |
| Filter 1 | 문의 우선순위 = 긴급 |
| Filter 2 | 문의 우선순위 = 일반 |
| Action 1 | Google Sheets - Add a Row |
| Action 2 | Gmail - Send an Email |

### Make 실행 결과

| 테스트 | 결과 |
|---|---|
| 긴급 문의 테스트 | 긴급문의 탭에 기록되고 Gmail 긴급 알림 도착 |
| 일반 문의 테스트 | 일반문의 탭에 기록되고 Gmail 일반 알림 도착 |

---

## 6. Zapier 구현

Zapier에서는 동일한 자동화 흐름을 긴급용 Zap과 일반용 Zap으로 나누어 구현했다.

### 긴급 문의 Zap

```text
Google Sheets - New or Updated Spreadsheet Row
↓
Filter by Zapier
- 문의 우선순위 = 긴급
↓
Google Sheets - Create Spreadsheet Row → 긴급문의
↓
Gmail - Send Email → 긴급 알림
```

### 일반 문의 Zap

```text
Google Sheets - New or Updated Spreadsheet Row
↓
Filter by Zapier
- 문의 우선순위 = 일반
↓
Google Sheets - Create Spreadsheet Row → 일반문의
↓
Gmail - Send Email → 일반 알림
```

### Zapier 실행 결과

| 테스트 | 결과 |
|---|---|
| 긴급 문의 테스트 | 긴급문의 탭에 기록되고 Gmail 긴급 알림 도착 |
| 일반 문의 테스트 | 일반문의 탭에 기록되고 Gmail 일반 알림 도착 |

---

## 7. Make와 Zapier 비교 분석

| 비교 항목 | Make | Zapier |
|---|---|---|
| UI/UX | 노드 기반 시각적 화면으로 전체 흐름을 한눈에 보기 쉬움 | 단계형 리스트 구조로 순서대로 설정하기 쉬움 |
| 설정 난이도 | Router와 Filter 개념을 이해해야 해서 초반 진입 장벽이 있음 | 각 단계가 안내형으로 구성되어 초보자가 따라가기 쉬움 |
| 조건 분기 방식 | Router를 통해 하나의 시나리오 안에서 긴급/일반 분기 구현 가능 | 무료 플랜에서는 Paths 사용이 제한될 수 있어 Zap을 2개로 나누어 구현 |
| 실행 로그 확인 | 각 모듈별 데이터 흐름과 분기 통과 여부를 시각적으로 확인 가능 | Test run과 Zap History를 통해 단계별 성공 여부 확인 가능 |
| 무료 플랜 활용성 | 비교적 복잡한 분기 구조도 무료 범위에서 구현하기 쉬움 | 단순 자동화에는 적합하지만 복잡한 분기에는 제한이 있을 수 있음 |
| 확장성 | 복잡한 분기와 여러 경로가 있는 자동화에 적합 | 단순하고 반복적인 Trigger → Action 자동화에 적합 |

---

## 8. 프로젝트 1 결론

이번 비교 구현에서는 **Make가 조건 분기 구조를 더 명확하게 표현했다**. Make는 Router를 사용해 하나의 시나리오 안에서 긴급 문의와 일반 문의를 분리할 수 있었고, 실행 결과에서도 각 분기별 통과 여부를 한눈에 확인할 수 있었다.

반면 Zapier는 단계별 설정 방식이 직관적이어서 초보자가 따라가기 쉬웠다. 하지만 동일한 분기 구조를 하나의 Zap 안에서 구현하기에는 제한이 있어 긴급용 Zap과 일반용 Zap을 따로 구성했다.

따라서 복잡한 분기와 여러 경로가 필요한 자동화에는 Make가 더 적합하고, 단순한 Trigger → Action 구조의 업무에는 Zapier가 더 적합하다고 판단했다.

---

## 9. 프로젝트 1 스크린샷

### Google Form 및 Google Sheets

![Google Form 질문 구성](project01_tool_comparison/screenshots/google_form_questions.png)

![Google Sheets 응답 화면](project01_tool_comparison/screenshots/google_sheet_responses.png)

### Make 구현

![Make 전체 워크플로우](project01_tool_comparison/screenshots/make_workflow_full.png)

![Make 긴급 분기 실행 결과](project01_tool_comparison/screenshots/make_result_urgent.png)

![Make 일반 분기 실행 결과](project01_tool_comparison/screenshots/make_result_normal.png)

![Make 긴급 Gmail 알림](project01_tool_comparison/screenshots/make_gmail_urgent.png)

![Make 일반 Gmail 알림](project01_tool_comparison/screenshots/make_gmail_normal.png)

### Zapier 구현



![Zapier 긴급 Gmail 알림](project01_tool_comparison/screenshots/zapier_gmail_urgent.png)



![Zapier 일반 Gmail 알림](project01_tool_comparison/screenshots/zapier_gmail_normal.png)

---

# 프로젝트 2. 자유 주제 자동화 설계 및 구현

## 1. 프로젝트 개요

프로젝트 2에서는 자유 주제로 반복 업무를 선정하고, 노코드 자동화 도구를 활용하여 실제 자동화 워크플로우를 설계하고 구현했다.

선정한 주제는 **홍보 포스터 제작 문의 자동 분류 및 알림 시스템**이다.

소상공인이나 가게 운영자가 Google Form을 통해 홍보 포스터 제작 문의를 제출하면, Google Sheets에 응답이 저장되고 Make가 이를 감지한다. 이후 문의의 긴급 여부에 따라 긴급 제작 문의와 일반 제작 문의로 분기하고, 각각 별도 시트에 기록한 뒤 Gmail로 알림을 전송한다.

---

## 2. 자동화 대상 업무

홍보 포스터 제작 문의가 들어오면 제작자가 직접 문의 내용을 확인하고, 긴급 문의인지 일반 문의인지 판단한 뒤, 별도 문서나 시트에 정리하고 메일로 확인해야 한다.

본 자동화의 목적은 포스터 제작 문의 접수 이후의 반복적인 분류, 기록, 알림 업무를 자동화하여 문의 확인 시간을 줄이고, 긴급 문의를 빠르게 파악할 수 있도록 하는 것이다.

---

## 3. 사용 도구

| 구분 | 사용 도구 | 역할 |
|---|---|---|
| 입력 도구 | Google Form | 포스터 제작 문의 접수 |
| 데이터 저장 | Google Sheets | 문의 응답 및 분류 결과 저장 |
| 자동화 도구 | Make | Trigger, Router, Action 자동화 |
| 알림 도구 | Gmail | 긴급/일반 문의 알림 전송 |

---

## 4. Google Form 설계

Google Form 제목은 **홍보 포스터 제작 문의 폼**으로 설정했다.

| 항목 | 설명 |
|---|---|
| 의뢰자 이름 | 포스터 제작을 요청한 사람의 이름 |
| 연락처 또는 이메일 | 문의자 연락처 |
| 가게 종류 | 카페, 식당, 미용실, 학원, 기타 등 |
| 포스터 제작 목적 | 이벤트 홍보, 신메뉴 홍보, 신규 고객 유입 등 |
| 희망 납기 | 3일 이내, 1주일 이내, 상관없음 |
| 긴급 여부 | 긴급 / 일반 |
| 요청 내용 | 구체적인 제작 요청 내용 |

조건 분기의 기준은 **긴급 여부** 항목이다.

---

## 5. Google Sheets 설계

| 시트 탭 이름 | 용도 |
|---|---|
| Form_Responses | Google Form 원본 응답 저장 |
| 긴급제작문의 | 긴급 제작 문의 자동 기록 |
| 일반제작문의 | 일반 제작 문의 자동 기록 |

`긴급제작문의`와 `일반제작문의` 탭의 헤더는 다음과 같이 구성했다.

```text
타임스탬프 | 의뢰자 이름 | 연락처 또는 이메일 | 가게 종류 | 포스터 제작 목적 | 희망 납기 | 긴급 여부 | 요청 내용 | 처리상태
```

---

## 6. Make 자동화 워크플로우

```text
Google Sheets - Watch New Rows
↓
Router
├─ 긴급 제작 문의 필터
│  ├─ Google Sheets - Add a Row → 긴급제작문의
│  └─ Gmail - Send an Email → 긴급 알림
└─ 일반 제작 문의 필터
   ├─ Google Sheets - Add a Row → 일반제작문의
   └─ Gmail - Send an Email → 일반 알림
```

---

## 7. Trigger, Router, Action 구성

| 구분 | 구현 내용 |
|---|---|
| Trigger | Google Sheets - Watch New Rows |
| Router | 긴급 제작 문의 / 일반 제작 문의 분기 |
| 긴급 필터 | 긴급 여부 = 긴급 |
| 일반 필터 | 긴급 여부 = 일반 |
| Action 1 | Google Sheets - Add a Row |
| Action 2 | Gmail - Send an Email |

---

## 8. 긴급 제작 문의 경로

긴급 제작 문의는 `긴급제작문의` 시트에 자동으로 기록된다.

| 열 | 입력값 |
|---|---|
| 타임스탬프 | 폼 응답의 타임스탬프 |
| 의뢰자 이름 | 폼 응답의 의뢰자 이름 |
| 연락처 또는 이메일 | 폼 응답의 연락처 또는 이메일 |
| 가게 종류 | 폼 응답의 가게 종류 |
| 포스터 제작 목적 | 폼 응답의 포스터 제작 목적 |
| 희망 납기 | 폼 응답의 희망 납기 |
| 긴급 여부 | 폼 응답의 긴급 여부 |
| 요청 내용 | 폼 응답의 요청 내용 |
| 처리상태 | 긴급 접수 |

긴급 문의가 접수되면 Gmail로 긴급 알림 메일이 전송된다.

```text
[긴급 포스터 제작 문의] 의뢰자 이름
```

---

## 9. 일반 제작 문의 경로

일반 제작 문의는 `일반제작문의` 시트에 자동으로 기록된다.

| 열 | 입력값 |
|---|---|
| 타임스탬프 | 폼 응답의 타임스탬프 |
| 의뢰자 이름 | 폼 응답의 의뢰자 이름 |
| 연락처 또는 이메일 | 폼 응답의 연락처 또는 이메일 |
| 가게 종류 | 폼 응답의 가게 종류 |
| 포스터 제작 목적 | 폼 응답의 포스터 제작 목적 |
| 희망 납기 | 폼 응답의 희망 납기 |
| 긴급 여부 | 폼 응답의 긴급 여부 |
| 요청 내용 | 폼 응답의 요청 내용 |
| 처리상태 | 일반 접수 |

일반 문의가 접수되면 Gmail로 일반 알림 메일이 전송된다.

```text
[일반 포스터 제작 문의] 의뢰자 이름
```

---

## 10. 테스트 결과

| 테스트 | 예상 결과 | 실제 결과 |
|---|---|---|
| 긴급 문의 테스트 | 긴급제작문의 시트에 기록 및 Gmail 긴급 알림 전송 | 정상 실행 |
| 일반 문의 테스트 | 일반제작문의 시트에 기록 및 Gmail 일반 알림 전송 | 정상 실행 |

---

## 11. 과제 요구사항 충족 여부

| 요구사항 | 구현 내용 | 충족 여부 |
|---|---|---|
| 자동화할 반복 업무 선정 | 홍보 포스터 제작 문의 분류 및 알림 | 충족 |
| Trigger 1개 이상 | Google Sheets - Watch New Rows | 충족 |
| Action 2개 이상 | Google Sheets Add a Row, Gmail Send an Email | 충족 |
| Filter 또는 Router 1개 이상 | Make Router 및 긴급/일반 필터 | 충족 |
| 각 분기 실행 확인 | 긴급 문의와 일반 문의 각각 테스트 | 충족 |
| 실제 동작 확인 | 시트 기록 및 Gmail 수신 확인 | 충족 |

---

## 12. 프로젝트 2 스크린샷

![프로젝트 2 Google Form](project02_custom_automation/screenshots/project02_form.png)

![프로젝트 2 Google Sheets](project02_custom_automation/screenshots/project02_spreadsheet.png)

![프로젝트 2 Make 워크플로우](project02_custom_automation/screenshots/project02_make_workflow.png)

![프로젝트 2 긴급 Gmail 알림](project02_custom_automation/screenshots/project02_gmail_urgent.png)

![프로젝트 2 일반 Gmail 알림](project02_custom_automation/screenshots/project02_gmail_normal.png)

---

## 13. 프로젝트 2 결론

프로젝트 2에서는 자유 주제로 **홍보 포스터 제작 문의 자동 분류 및 알림 시스템**을 설계하고 Make로 구현했다. Google Form, Google Sheets, Make, Gmail을 연동하여 문의 접수부터 분류, 기록, 알림까지의 반복 업무를 자동화했다.

이번 구현을 통해 Trigger, Router, Filter, Action이 실제 업무 자동화에서 어떻게 연결되는지 확인할 수 있었다. 특히 긴급 문의와 일반 문의를 자동으로 분리함으로써 업무 처리 우선순위를 빠르게 판단할 수 있었고, 단순 반복 업무를 줄이는 자동화의 효과를 확인할 수 있었다.

---

# 전체 결론

Mission03에서는 노코드 자동화 도구를 활용해 실제로 작동하는 자동화 워크플로우를 구현했다.

프로젝트 1에서는 Make와 Zapier를 비교하며 같은 자동화 흐름을 서로 다른 도구로 구현했고, 프로젝트 2에서는 자유 주제로 홍보 포스터 제작 문의 자동화 시스템을 설계했다.

두 프로젝트 모두 Trigger, Action, Filter/Router 조건을 포함했으며, 긴급 분기와 일반 분기를 각각 실행하여 실제 동작을 확인했다.
