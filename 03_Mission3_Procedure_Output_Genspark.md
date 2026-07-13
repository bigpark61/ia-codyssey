 이 과제는 사실상 **“노코드 자동화 도구를 직접 써 보고, 같은 워크플로우를 비교한 뒤, 하나를 골라 실전 자동화를 설계·구현하라”**는 미션입니다.  
제출 성공률을 높이려면 범위를 넓히기보다 **무료 플랜으로 재현 가능한 단일 시나리오를 정확하게 2개 도구로 반복 구현**하는 쪽이 가장 안전합니다.

아래처럼 진행하면 과제 요구사항을 거의 그대로 만족시킬 수 있습니다.

---

# 과제 해석 요약

핵심은 3가지입니다.

1. **같은 자동화 흐름을 서로 다른 2개 이상 도구로 구현**
2. **Trigger / Action / 조건 분기**를 실제로 동작하게 만들기
3. **도구별 차이와 적합한 사용 상황을 근거 있게 설명**

즉, “자동화가 된다”보다 **왜 그렇게 설계했는지**가 중요합니다.

---

# 추천 구현 조합

## 프로젝트 1: [Make](https://www.make.com/en/pricing) vs [Zapier](https://zapier.com/pricing)

가장 무난한 조합은 **[Make](https://www.make.com/en/pricing)** 와 **[Zapier](https://zapier.com/pricing)** 입니다.

이유는 단순합니다.  
둘 다 대표적인 노코드 자동화 도구이고, 과제에서 요구하는 **Trigger + Action + 분기 구조**를 안정적으로 구현할 수 있습니다. Zapier는 무료 플랜에서 **100 tasks/month**, 무제한 Zap/Forms/Tables, 2-step Zap을 제공하고, 유료 시작 플랜은 Professional **$19.99/month**, Team **$69/month**입니다. [Source](https://zapier.com/pricing)

Make는 Router로 분기를 구성할 수 있고, Zapier는 Paths로 조건 분기를 구성할 수 있어 “같은 논리의 워크플로우를 두 도구로 비교”하기에 적합합니다. Make Router는 데이터를 여러 경로로 분기해 각 경로를 조건별로 다르게 처리하며, fallback route도 지원합니다. [Source](https://help.make.com/router) Zapier Paths는 “If A면 X, If B면 Y” 형태의 조건 분기 로직을 제공해 동일한 목적을 달성합니다. [Source](https://help.zapier.com/hc/en-us/articles/8496288555917-Add-branching-logic-to-Zap-workflows-with-Paths)

## 선택적 3번째 도구: [n8n](https://n8n.io/pricing/)

여유가 있으면 **[n8n](https://n8n.io/pricing/)** 을 비교군으로 추가할 수 있습니다.  
n8n의 **If node**는 비교 조건으로 워크플로우를 분기하고, 여러 조건은 AND/OR로 결합할 수 있습니다. 2개보다 많은 분기가 필요하면 Switch node 사용이 권장됩니다. [Source](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.if/)  
또한 hosted starter 수준에서 **1 shared project, 5 concurrent executions, unlimited users, 2,300 AI credits/month** 같은 사용 한도가 명시되어 있어 “기술팀 친화형 자동화 도구”라는 비교 포인트를 만들기 좋습니다. [Source](https://n8n.io/pricing/)

---

# 가장 안전한 공통 워크플로우

## 권장 시나리오

**Google Form 제출 → 조건 분기(점수 기준) → Google Sheets 기록 + Slack/Discord 알림**

이 시나리오가 좋은 이유는 다음과 같습니다.

- Trigger가 명확함: 새 응답 제출
- Action 2개 이상 구현 쉬움: 시트 기록 + 메시지 전송
- 조건 분기 검증 쉬움: 합격/불합격 두 경로를 각각 테스트 가능
- 캡처할 화면이 분명함: 폼 응답, 자동화 캔버스, 시트 결과, 메신저 결과

---

# 프로젝트 1 구현안

## [Make](https://www.make.com/en/pricing) vs [Zapier](https://zapier.com/pricing)

## 공통 워크플로우 정의

**입력**: Google Form 응답  
예시 필드: 이름, 이메일, 점수

**분기 규칙**

- 점수 **80 이상** → 합격 시트 기록 + 합격 알림
- 점수 **80 미만** → 불합격 시트 기록 + 불합격 알림

**요구사항 대응**

- Trigger 1개: Form 응답
- Action 2개 이상: Sheets 기록, Slack/Discord 메시지
- 조건 분기 1개 이상: 점수 기준 분기
- 분기 실행 증빙: 80 이상 / 미만 테스트 데이터 각각 1건 이상

---

## 1) [Make](https://www.make.com/en/pricing) 구현

### 구성

- **Trigger**: Google Forms/Google Sheets 응답 감지
- **Router**: 점수 80 이상 / 미만 분기
- **Action A1**: Google Sheets - Add a Row (합격 시트)
- **Action A2**: Slack/Discord - Send Message (합격 안내)
- **Action B1**: Google Sheets - Add a Row (불합격 시트)
- **Action B2**: Slack/Discord - Send Message (불합격 안내)

### 기술 설명 문장

Make의 Router는 하나의 입력 데이터를 여러 경로로 나누고, 각 경로에 개별 필터를 걸어 서로 다른 처리를 수행하도록 설계된다. 또한 fallback route를 둘 수 있어 어떤 조건에도 맞지 않는 데이터의 마지막 처리 경로를 만들 수 있다. [Source](https://help.make.com/router)

### 캡처해야 할 화면

- 시나리오 전체 캔버스
- Router 필터 설정 화면
- 합격 경로 실행 로그
- 불합격 경로 실행 로그
- 결과가 기록된 Google Sheets
- Slack/Discord 알림 화면

---

## 2) [Zapier](https://zapier.com/pricing) 구현

### 구성

- **Trigger**: New Form Response / New Spreadsheet Row
- **Paths**:
    - Path A: 점수 80 이상
    - Path B: 점수 80 미만
- **Action A1**: Google Sheets - Create Spreadsheet Row
- **Action A2**: Slack/Discord - Send Channel Message
- **Action B1**: Google Sheets - Create Spreadsheet Row
- **Action B2**: Slack/Discord - Send Channel Message

### 기술 설명 문장

Zapier의 Paths는 하나의 Zap 안에서 규칙 기반 분기를 구현하는 기능으로, 트리거 이후 데이터 상태에 따라 서로 다른 액션 체인을 실행할 수 있다. 즉 “If A then X / If B then Y” 방식의 조건 분기다. [Source](https://help.zapier.com/hc/en-us/articles/8496288555917-Add-branching-logic-to-Zap-workflows-with-Paths)

### 캡처해야 할 화면

- Zap 전체 단계 화면
- Paths 조건 설정 화면
- Task History 실행 결과
- 시트 적재 결과
- Slack/Discord 전송 결과

---

# 프로젝트 1 비교 분석 보고서 초안

## 비교 대상

- [Make](https://www.make.com/en/pricing)
- [Zapier](https://zapier.com/pricing)
- (선택) [n8n](https://n8n.io/pricing/)

## 비교 항목 1. 분기 구조 표현 방식

Make는 **시각적 캔버스 + Router 분기**가 강점이고, 전체 흐름이 눈에 잘 들어옵니다. [Source](https://help.make.com/router)  
Zapier는 **리스트형 단계 + Paths 분기**라서 초보자는 이해하기 쉬우나, 복잡해질수록 전체 흐름을 한눈에 보기 어렵습니다. [Source](https://help.zapier.com/hc/en-us/articles/8496288555917-Add-branching-logic-to-Zap-workflows-with-Paths)

## 비교 항목 2. 무료 플랜 접근성

Zapier는 무료 플랜에서 **100 tasks/month**, 무제한 Zap/Forms/Tables, 2-step Zap을 제공합니다. [Source](https://zapier.com/pricing)  
n8n hosted 안내 페이지는 Starter 기준 **1 shared project, 5 concurrent executions, unlimited users, 2,300 AI credits/month** 등 사용량 중심 정보를 보여 줍니다. [Source](https://n8n.io/pricing/)  
Make는 과거 학습용으로 널리 쓰이는 무료/저가 진입이 강점이며, 실제 교육 환경에서 비교 구현에 많이 활용됩니다.

## 비교 항목 3. 조건 분기 유연성

Make Router는 fallback route와 route ordering이 가능해 다중 흐름 설계에 좋습니다. [Source](https://help.make.com/router)  
Zapier Paths는 실무에서 자주 쓰는 “승인/반려”, “존재/미존재”, “고객 유형별 처리” 같은 분기에 적합합니다. [Source](https://help.zapier.com/hc/en-us/articles/8496288555917-Add-branching-logic-to-Zap-workflows-with-Paths)  
n8n은 If node AND/OR, 그리고 다중 분기용 Switch node까지 지원해 로직 확장성이 좋습니다. [Source](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.if/)

## 비교 항목 4. 기술 친화성

n8n은 self-hosted 지향성과 조건 노드 구조 때문에 기술팀에게 유리합니다. If node는 데이터 타입별 비교 연산을 매우 세밀하게 줄 수 있습니다. [Source](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.if/)  
Zapier는 빠른 구축과 쉬운 진입이 장점입니다.  
Make는 시각적 설계와 중간 난이도 자동화에 강합니다.

## 비교 항목 5. 실행 로그 확인 방식

Make는 실행 로그와 라우터 경로별 처리 확인이 직관적인 편입니다. [Source](https://help.make.com/router)  
Zapier는 Task History 중심으로 단계별 성공/실패를 확인합니다. [Source](https://zapier.com/pricing)

## 비교 항목 6. 적합한 사용 상황

초보자, 빠른 MVP, 단순 사무 자동화에는 Zapier가 적합합니다. [Source](https://zapier.com/pricing)  
시각적 워크플로우 설계와 분기 흐름 설명이 중요한 교육/과제 환경에는 Make가 매우 적합합니다. [Source](https://help.make.com/router)  
기술팀, 세밀한 조건 처리, self-hosted/확장성 관점에서는 n8n이 강점이 있습니다. [Source](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.if/)

---

# 프로젝트 2 추천 주제

## 자유 주제 자동화 설계 및 구현

가장 무난한 자유 주제는 아래 둘 중 하나입니다.

## 안 1. 팀 회의 신청 자동 분류

**폼 제출 → 긴급도 분기 → 담당 시트 기록 + 팀 채널 알림**

- Trigger: Google Form 제출
- Filter/Router: 긴급도 = 높음 / 보통
- Action 1: Google Sheets 기록
- Action 2: Slack/Discord 알림
- 장점: 프로젝트 1과 유사해서 구현 재사용 가능

## 안 2. 이메일 문의 자동 정리

**이메일 수신 → 제목 키워드 분기 → 문의 유형별 시트 적재 + 알림**

- Trigger: Gmail 새 메일
- 조건 분기: “환불”, “버그”, “일반 문의”
- Action 1: Sheets 적재
- Action 2: 채널 알림
- 보너스 과제와 연결 쉬움: 실패 시 알림, AI 요약 추가 가능

---

# 프로젝트 2 제출용 문서 예시

## 자동화할 반복 업무

매일 여러 채널로 들어오는 문의를 수작업으로 정리하고 담당자에게 전달하는 업무

## 선정 도구

**[Make](https://www.make.com/en/pricing)**

## 선정 이유

Make는 시각적 캔버스 기반이라 조건 분기와 처리 경로를 한눈에 설명하기 쉽고, Router를 통해 여러 조건별 흐름을 깔끔하게 분리할 수 있다. 교육 목적상 설계 의도를 보여 주기 좋고, 실행 로그도 비교적 직관적으로 확인할 수 있다. [Source](https://help.make.com/router)

## 워크플로우 설명

1. Gmail에서 새 문의 메일을 감지한다.
2. 제목 또는 본문 키워드를 읽는다.
3. Router로 문의 유형을 분기한다.
4. 각 유형별 시트에 기록한다.
5. 팀 채널에 문의 요약을 전송한다.
6. 필요 시 fallback route로 “기타 문의”를 별도 분류한다. [Source](https://help.make.com/router)

---

# 보너스 과제까지 챙기려면

## 보너스 1: AI 연동

- Action 추가: “문의 내용을 3줄로 요약”
- 제출 문서에는
    - 왜 AI를 넣었는지
    - 사람이 하던 요약을 어떻게 대체했는지
    - 오탐 가능성은 없는지  
        를 같이 쓰면 좋습니다.

## 보너스 2: 실패 알림

- 시트 기록 실패 시 → Slack/Discord “저장 실패” 알림
- 또는 fallback sheet에 임시 적재
- 이러면 과제의 “재시도/대체 경로” 설명이 쉬워집니다.

---

# 실제 제출 체크리스트

- [ ] 프로젝트 1에서 **같은 워크플로우**를 2개 도구로 구현
- [ ] Trigger 1개 이상
- [ ] Action 2개 이상
- [ ] 조건 분기 1개 이상
- [ ] 분기 **양쪽 경로가 실제 실행된 결과 캡처**
- [ ] 비교 항목 5개 이상
- [ ] 장단점 + 적합한 상황 서술
- [ ] 프로젝트 2에서 반복 업무 정의
- [ ] 도구 선정 이유 작성
- [ ] 흐름 설명 포함
- [ ] 민감정보 마스킹

---

# 제일 추천하는 최종 제출 전략

**프로젝트 1:**  
[Make](https://www.make.com/en/pricing) vs [Zapier](https://zapier.com/pricing)  
“Google Form 제출 → 점수 분기 → Sheets 기록 + Slack/Discord 알림”

**프로젝트 2:**  
[Make](https://www.make.com/en/pricing)  
“이메일/폼 문의 → 문의 유형 분기 → 시트 적재 + 팀 채널 알림”

이 조합이면 과제 요구사항, 비교 분석, 실동작 증빙, 보너스 확장까지 모두 자연스럽게 연결됩니다.

원하시면 다음 단계로 바로 이어서  
**1) 제출용 비교 분석 보고서 Markdown 초안** 또는  
**2) Make / Zapier 각각의 단계별 구현 가이드**  
형태로 정리해드릴게요.