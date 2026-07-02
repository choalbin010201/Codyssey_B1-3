# 프로젝트 1. 자동화 도구 비교 구현 보고서

## 1. 프로젝트 개요

본 프로젝트는 동일한 자동화 워크플로우를 서로 다른 노코드 자동화 도구인 **Make**와 **Zapier**로 각각 구현하고, 도구별 특징과 장단점을 비교하기 위해 수행했다.

구현한 자동화 주제는 **문의 접수 자동 분류 및 알림 시스템**이다. Google Form으로 문의가 접수되면 Google Sheets에 응답이 저장되고, 자동화 도구가 새 응답을 감지한 뒤 문의 우선순위에 따라 긴급 문의와 일반 문의로 분기한다. 이후 각 분기별로 Google Sheets의 별도 탭에 기록하고 Gmail로 알림을 전송한다.

---

## 2. 자동화 워크플로우 설명

### 자동화 목적

반복적으로 들어오는 문의를 사람이 직접 확인하고 분류한 뒤, 별도 시트에 기록하고 담당자에게 알림을 보내는 작업을 자동화하는 것이 목적이다.

### 전체 흐름

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

### Trigger

- Google Sheets에 새 응답 행이 추가됨

### 조건 분기

- 문의 우선순위가 `긴급`인 경우
- 문의 우선순위가 `일반`인 경우

### Action

각 분기마다 다음 두 가지 Action을 수행했다.

1. Google Sheets에 분기별 행 추가
2. Gmail 알림 메일 전송

---

## 3. 사용한 입력 데이터

Google Form은 다음 항목으로 구성했다.

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

Google Form 응답은 Google Sheets의 `Form_Responses` 탭에 자동 저장되도록 설정했다. 자동화 결과를 분리하기 위해 다음과 같은 탭을 추가로 구성했다.

| 시트 탭 이름 | 용도 |
|---|---|
| Form_Responses | Google Form 원본 응답 저장 |
| 긴급문의 | 긴급 문의 자동 기록 |
| 일반문의 | 일반 문의 자동 기록 |

`긴급문의`와 `일반문의` 탭에는 다음과 같은 헤더를 동일하게 구성했다.

```text
타임스탬프 | 성함 | 연락처 또는 이메일 | 문의 우선순위 | 문의 상세 내용 | 희망 처리 방식 | 테스트 구분 | 처리상태
```

---

## 5. Make 구현 내용

### 사용 도구

- Make
- Google Sheets
- Gmail

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

### Trigger

- **Google Sheets - Watch New Rows**
- `Form_Responses` 탭에 새 행이 추가되면 자동화가 시작된다.

### 조건 분기

Make에서는 **Router**를 사용해 두 개의 경로를 구성했다.

| 분기 | 조건 |
|---|---|
| 긴급 문의 필터 | 문의 우선순위 = 긴급 |
| 일반 문의 필터 | 문의 우선순위 = 일반 |

### Action 1: Google Sheets 행 추가

긴급 문의는 `긴급문의` 탭에 기록하고, 일반 문의는 `일반문의` 탭에 기록했다.

### Action 2: Gmail 알림 전송

각 분기별로 Gmail 알림을 전송했다.

- 긴급 문의: `[Make 긴급 문의 접수]` 형식의 이메일 전송
- 일반 문의: `[Make 일반 문의 접수]` 형식의 이메일 전송

### Make 실행 결과

긴급 문의 테스트와 일반 문의 테스트를 각각 1회 이상 실행했다.

| 테스트 | 결과 |
|---|---|
| 긴급 문의 테스트 | 긴급문의 탭에 기록되고 Gmail 긴급 알림 도착 |
| 일반 문의 테스트 | 일반문의 탭에 기록되고 Gmail 일반 알림 도착 |

Make 실행 화면에서는 Router의 각 분기별 통과 여부가 숫자로 표시되어, 긴급 경로와 일반 경로가 각각 정상 실행되었음을 확인할 수 있었다.

---

## 6. Zapier 구현 내용

### 사용 도구

- Zapier
- Google Sheets
- Gmail

### Zapier 구현 방식

Zapier에서는 Make의 Router와 동일한 분기 구조를 구현하기 위해 긴급 문의용 Zap과 일반 문의용 Zap을 각각 따로 만들었다.

Make에서는 하나의 시나리오 안에서 Router를 통해 긴급/일반 분기를 구성했지만, Zapier에서는 무료 플랜 기준으로 복잡한 분기 기능 사용에 제한이 있을 수 있어 두 개의 Zap으로 분리했다.

### Zap 1. 긴급 문의 자동화

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

#### Trigger

- Google Sheets의 `Form_Responses` 탭에서 새 행 또는 업데이트된 행 감지

#### Filter

- 문의 우선순위가 `긴급`인 경우에만 다음 단계로 진행

#### Action 1

- Google Sheets의 `긴급문의` 탭에 행 추가

#### Action 2

- Gmail로 긴급 문의 접수 알림 전송

### Zap 2. 일반 문의 자동화

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

#### Trigger

- Google Sheets의 `Form_Responses` 탭에서 새 행 또는 업데이트된 행 감지

#### Filter

- 문의 우선순위가 `일반`인 경우에만 다음 단계로 진행

#### Action 1

- Google Sheets의 `일반문의` 탭에 행 추가

#### Action 2

- Gmail로 일반 문의 접수 알림 전송

### Zapier 실행 결과

긴급 문의와 일반 문의를 각각 Google Form으로 제출하여 테스트했다.

| 테스트 | 결과 |
|---|---|
| 긴급 문의 테스트 | 긴급문의 탭에 기록되고 Gmail 긴급 알림 도착 |
| 일반 문의 테스트 | 일반문의 탭에 기록되고 Gmail 일반 알림 도착 |

Zapier에서도 각 Zap의 Test run 결과와 Gmail 수신 결과를 통해 자동화가 정상적으로 실행되었음을 확인했다.

---

## 7. Make와 Zapier 비교 분석

| 비교 항목 | Make | Zapier |
|---|---|---|
| UI/UX | 노드 기반 시각적 화면으로 전체 흐름을 한눈에 보기 쉬움 | 단계형 리스트 구조로 순서대로 설정하기 쉬움 |
| 설정 난이도 | Router와 Filter 개념을 이해해야 해서 초반 진입 장벽이 있음 | 각 단계가 안내형으로 구성되어 초보자가 따라가기 쉬움 |
| 조건 분기 방식 | Router를 통해 하나의 시나리오 안에서 긴급/일반 분기 구현 가능 | 무료 플랜에서는 Paths 사용이 제한될 수 있어 Zap을 2개로 나누어 구현 |
| 실행 로그 확인 | 각 모듈별 데이터 흐름과 분기 통과 여부를 시각적으로 확인 가능 | Test run과 Zap History를 통해 단계별 성공 여부 확인 가능 |
| 무료 플랜 활용성 | 비교적 복잡한 분기 구조도 무료 범위에서 구현하기 쉬움 | 단순 자동화에는 적합하지만 복잡한 분기에는 제한이 있을 수 있음 |
| Google Sheets 연동 | 시트와 탭 선택, 필드 매핑이 비교적 명확함 | 샘플 데이터를 보며 필드 매핑할 수 있어 직관적임 |
| 확장성 | Router, Iterator 등으로 복잡한 워크플로우 확장에 유리함 | 단순하고 반복적인 Trigger → Action 자동화에 적합함 |
| 오류 확인 | 모듈별 입력/출력 데이터를 상세히 확인 가능 | 실행 결과는 보기 쉽지만 복잡한 데이터 흐름 추적은 Make보다 제한적임 |

---

## 8. 도구별 장단점

### Make 장점

- Router를 사용해 하나의 시나리오 안에서 여러 조건 분기를 시각적으로 표현할 수 있다.
- 각 모듈의 실행 결과와 데이터 흐름을 상세하게 확인할 수 있다.
- 복잡한 워크플로우를 구성할 때 구조를 한눈에 파악하기 쉽다.
- 분기별 실행 여부가 화면에 숫자로 표시되어 테스트 결과를 확인하기 좋다.

### Make 단점

- 처음 사용하는 사용자는 Router, Filter, 모듈 연결 방식에 익숙해지는 데 시간이 필요하다.
- 설정 화면이 세부적이기 때문에 단순 작업에는 다소 복잡하게 느껴질 수 있다.

### Zapier 장점

- 단계별 설정 방식이 직관적이라 초보자가 접근하기 쉽다.
- Trigger, Filter, Action을 순서대로 설정하면 되어 작업 흐름이 단순하다.
- 필드 매핑 시 샘플 데이터가 함께 보여 이해하기 쉽다.
- Gmail, Google Sheets 같은 주요 서비스 연동이 간단하다.

### Zapier 단점

- 복잡한 조건 분기 구조를 하나의 Zap에서 구현하기에는 제한이 있을 수 있다.
- 이번 구현에서는 긴급용 Zap과 일반용 Zap을 따로 만들어야 했다.
- 여러 Zap으로 나누면 관리해야 할 자동화 수가 늘어난다.
- Make보다 전체 흐름을 시각적으로 한눈에 파악하기는 어렵다.

---

## 9. 어떤 상황에서 적합한가

### Make가 적합한 상황

Make는 조건 분기, 여러 경로, 복잡한 데이터 처리 흐름이 필요한 자동화에 적합하다. 이번 과제처럼 긴급 문의와 일반 문의를 하나의 흐름 안에서 나누고, 각 경로마다 서로 다른 Action을 실행해야 하는 경우 Make의 Router 구조가 유용하다.

### Zapier가 적합한 상황

Zapier는 단순하고 명확한 자동화에 적합하다. 예를 들어 “새 응답이 들어오면 메일 보내기”, “새 행이 생기면 다른 시트에 기록하기”처럼 Trigger와 Action이 직선형으로 연결되는 업무에는 Zapier가 빠르고 직관적이다.

---

## 10. 최종 의견

이번 워크플로우에서는 **Make가 조건 분기 구조를 더 명확하게 표현했다**. Make는 Router를 사용해 하나의 시나리오 안에서 긴급 문의와 일반 문의를 분리할 수 있었고, 실행 결과에서도 각 분기별 통과 여부를 한눈에 확인할 수 있었다.

반면 Zapier는 단계별 설정 방식이 직관적이어서 초보자가 따라가기 쉬웠다. 하지만 동일한 분기 구조를 하나의 Zap 안에서 구현하기에는 제한이 있어 긴급용 Zap과 일반용 Zap을 따로 구성했다.

따라서 **복잡한 분기와 여러 경로가 필요한 자동화에는 Make가 더 적합하고, 단순한 Trigger → Action 구조의 업무에는 Zapier가 더 적합하다**고 판단했다.

---

## 11. 제출용 스크린샷 목록

아래 파일들은 GitHub의 `mission03/project01_tool_comparison/screenshots/` 폴더에 저장한다.

### 공통 화면

| 파일명 | 설명 |
|---|---|
| google_form_questions.png | Google Form 질문 구성 화면 |
| google_sheet_responses.png | Google Sheets 원본 응답 화면 |

### Make 구현 화면

| 파일명 | 설명 |
|---|---|
| make_workflow_full.png | Make 전체 워크플로우 화면 |
| make_result_urgent.png | Make 긴급 분기 실행 결과 |
| make_result_normal.png | Make 일반 분기 실행 결과 |
| make_gmail_urgent.png | Make 긴급 문의 Gmail 알림 결과 |
| make_gmail_normal.png | Make 일반 문의 Gmail 알림 결과 |

### Zapier 구현 화면

| 파일명 | 설명 |
|---|---|
| zapier_workflow_urgent.png | Zapier 긴급 문의 Zap 구성 화면 |
| zapier_result_urgent.png | Zapier 긴급 문의 실행 결과 |
| zapier_gmail_urgent.png | Zapier 긴급 문의 Gmail 알림 결과 |
| zapier_workflow_normal.png | Zapier 일반 문의 Zap 구성 화면 |
| zapier_result_normal.png | Zapier 일반 문의 실행 결과 |
| zapier_gmail_normal.png | Zapier 일반 문의 Gmail 알림 결과 |

---

## 12. 보안 및 개인정보 처리

제출용 스크린샷과 문서에는 API Key, 토큰, 비밀번호 등 민감정보가 포함되지 않도록 확인했다. 이메일 주소나 계정 정보가 노출되는 경우 일부를 마스킹하거나 테스트용 이메일을 사용했다.

---

## 13. 결론

프로젝트 1에서는 동일한 문의 접수 자동화 워크플로우를 Make와 Zapier로 각각 구현했다. 두 도구 모두 Google Sheets를 Trigger로 사용하고, 문의 우선순위에 따라 조건 분기를 수행한 뒤, Google Sheets 기록과 Gmail 알림이라는 두 가지 Action을 실행했다.

Make는 복잡한 분기 구조를 하나의 화면에서 관리하기 좋았고, Zapier는 단계별 설정이 직관적이었다. 이번 실습을 통해 Trigger, Action, Filter/Router의 개념을 실제 워크플로우 안에서 확인할 수 있었으며, 자동화 도구별 특징과 적합한 사용 상황을 비교할 수 있었다.
