Make 에서 Google Form 제출 → 조건 분기(점수 기준) → Google Sheets 기록 + Slack/Discord 알림을 자동화 하려고해

Make 에서 실제 구현하는 절차와 방법에 대해서 자세히 설명해줘

1. Make 에서 실제 구현하는 절차
2. Make 에서 모듈별 클릭순서 등 상세 설명
3. 작성용 구현절차 정리

이 자동화는 **[Make](https://www.make.com/en/pricing)** 에서 가장 무난하게 구현할 수 있는 대표 패턴입니다.

가장 먼저 한 가지부터 정리하면, **Google Form 자체를 바로 트리거로 잡는 방식**보다 **Google Form 응답이 저장되는 Google Sheets를 트리거로 감시하는 방식**이 과제용·실습용으로 가장 안정적입니다. Google Forms는 응답을 연결된 Google Sheet에 저장할 수 있고, Make 커뮤니티에서도 이 패턴으로 많이 구현합니다. 다만 이 방식은 “폼 제출 즉시”가 아니라 **시트의 새 행을 주기적으로 감지**하는 구조입니다. 또한 Make 가격 안내 기준으로 스케줄 간격은 플랜에 따라 달라질 수 있으며, 무료 플랜은 더 느릴 수 있습니다. [Source](https://support.google.com/docs/answer/2917686?hl=en) [Source](https://community.make.com/t/google-sheets-watch-rows-trigger/55507) [Source](https://www.make.com/en/pricing)

아래 설명은 **가장 실전적이고 과제 제출에도 적합한 구현 방식**으로 정리해드릴게요.

---

# [Make](https://www.make.com/en/pricing)에서 구현할 전체 구조

구조는 이렇게 잡으면 됩니다.

**Google Form 제출 → 연결된 Google Sheets에 응답 1행 추가 → Make가 새 행 감지 → Router로 점수 분기 → 결과용 Google Sheets 기록 → Slack 또는 Discord 알림 전송**

여기서 핵심은 **응답 원본 시트**와 **결과 기록 시트**를 분리하는 것입니다.  
같은 시트를 트리거와 기록 대상으로 동시에 쓰면, 기록 액션이 다시 트리거를 불러서 **무한 반복** 또는 중복 처리 위험이 생길 수 있습니다. 그래서 보통은:

- **응답 원본 시트**: Google Form이 자동 기록하는 시트
- **결과 기록 시트**: Make가 처리 결과를 적는 별도 시트

이렇게 나눕니다.

---

# 1. 사전 준비

## 1-1. [Google Forms](https://forms.google.com/) 만들기

먼저 Google Form을 하나 만듭니다.  
예를 들어 아래 필드를 넣으면 좋습니다.

- 이름
- 이메일
- 점수

여기서 **점수는 숫자형으로 관리할 수 있게** 만드는 것이 중요합니다.  
단답형으로 만들고, 응답 규칙에서 숫자만 입력되도록 제한하면 뒤에서 조건 분기가 훨씬 깔끔해집니다.

## 1-2. 응답을 [Google Sheets](https://sheets.google.com/)에 연결하기

Form 상단의 **Responses** 탭에서 응답 저장용 스프레드시트를 연결합니다. Google 공식 도움말에도 Form 응답을 연결된 Google Sheet에 저장할 수 있다고 안내되어 있습니다. [Source](https://support.google.com/docs/answer/2917686?hl=en)

이때 시트 이름 예시는 이렇게 두면 편합니다.

- `Form_Responses` : 폼 응답 원본
- `Processed_Results` : Make가 결과를 기록할 시트

`Processed_Results` 시트에는 미리 헤더를 만들어 두세요. 예를 들면:

- Timestamp
- Name
- Email
- Score
- Result
- Notification_Channel
- Note

이렇게 만들어 두면 나중에 Add a Row 매핑이 쉬워집니다.

## 1-3. Slack 또는 Discord 준비

알림을 보낼 도구를 하나 정합니다.

- **Slack**을 쓸 경우: Make에서 사용하는 Slack bot이 **목표 채널에 반드시 들어가 있어야** 합니다. 그렇지 않으면 `channel not found` 오류가 날 수 있습니다. [Source](https://apps.make.com/slack-modules)
- **Discord**를 쓸 경우: Make의 Discord 모듈은 채널, 스레드, DM으로 메시지를 보낼 수 있습니다. [Source](https://apps.make.com/discord)

과제 목적이라면 Slack이든 Discord든 아무거나 괜찮지만, 이미 쓰고 있는 쪽으로 가는 게 빠릅니다.

---

# 2. [Make](https://www.make.com/en/pricing)에서 새 시나리오 만들기

Make에 로그인한 뒤 **Create a new scenario**를 누릅니다.

이제 빈 캔버스가 열리면 첫 모듈을 추가해야 합니다.

---

# 3. 트리거 만들기: Google Sheets에서 새 응답 감지

## 3-1. 첫 모듈 선택

첫 모듈은 보통 다음처럼 갑니다.

**Google Sheets → Watch New Rows**

이 모듈의 의미는, 응답 시트에 **새로운 행이 생기면 그 행 데이터를 읽어오는 것**입니다.

Google Form 응답은 결국 연결된 Google Sheets에 행 단위로 쌓이므로, 실무적으로는 이 패턴이 가장 간단합니다. Make 커뮤니티에서도 Google Form 응답이 시트에 추가된 행을 Watch Rows로 감지하는 구성을 자주 씁니다. [Source](https://community.make.com/t/google-sheets-watch-rows-trigger/55507)

## 3-2. 연결 설정

이 모듈 안에서 다음을 설정합니다.

- Google 계정 연결
- Spreadsheet: Form 응답이 저장되는 스프레드시트 선택
- Sheet: `Form_Responses` 선택
- 어디서부터 읽을지 선택

보통 첫 설정 시에는 테스트용으로 최근 행을 읽게 하거나, 특정 위치부터 시작하게 설정합니다.

### 팁

테스트를 쉽게 하려면 기존 응답이 너무 많지 않은 상태에서 시작하는 게 좋습니다.  
처음엔 테스트용 응답 1~2개만 넣고 확인하는 편이 가장 편합니다.

---

# 4. 트리거 데이터가 제대로 들어오는지 먼저 확인

이 단계에서 바로 다음 모듈을 붙이지 말고, 먼저 **Run once**를 눌러서 테스트 데이터를 받는지 봅니다.

그 후 Google Form에 직접 응답을 1건 제출합니다.  
예를 들어:

- 이름: 홍길동
- 이메일: hong@test.com
- 점수: 85

그러면 일정 시간이 지난 뒤 Make가 해당 새 행을 읽어와야 합니다.

여기서 확인할 것은:

- 이름 필드가 제대로 들어오는지
- 이메일 필드가 제대로 들어오는지
- 점수 필드가 문자열인지 숫자인지
- 헤더명이 어떤 이름으로 잡히는지

이걸 먼저 확인해야 나중에 필터와 메시지 매핑에서 헷갈리지 않습니다.

---

# 5. 조건 분기 만들기: [Router](https://help.make.com/router) 추가

이제 트리거 모듈 다음에 **Router**를 추가합니다.

Make 문서 기준으로 Router는 **하나의 흐름을 여러 경로로 나누고**, 각 경로에서 서로 다른 처리를 하게 하는 기능입니다. 또한 각 경로에 **필터 조건**을 붙여 어떤 데이터가 어느 경로로 갈지 결정할 수 있습니다. [Source](https://help.make.com/router)

## 5-1. 왜 Router를 쓰는가

이번 과제에서는 점수에 따라 결과가 달라져야 하니까, 논리가 대략 이렇게 됩니다.

- 점수 ≥ 80 → 합격 처리
- 점수 < 80 → 불합격 처리

이런 **조건별 분기**를 Make에서는 Router + Filter 조합으로 구현합니다.

## 5-2. 라우트 2개 만들기

Router를 추가한 뒤 라우트를 2개 만듭니다.

- Route A: 합격
- Route B: 불합격

그리고 각 라우트 첫 연결선에 **Filter**를 겁니다.

### Route A 필터

- 조건: `Score >= 80`

### Route B 필터

- 조건: `Score < 80`

여기서 중요한 건 **점수 필드 타입**입니다.  
만약 Google Form 점수가 문자열처럼 들어오면 숫자 비교가 제대로 안 될 수 있습니다. 그런 경우엔 필터에서 숫자 변환을 써서 비교하거나, 애초에 폼 입력을 숫자형으로 강제하는 쪽이 안전합니다.

## 5-3. Router의 처리 순서 이해

Make Router 문서에서는 **routes are processed sequentially, not in parallel**라고 설명합니다. 즉, 라우트들은 동시에 실행되는 개념이 아니라 **순서대로 처리**됩니다. [Source](https://help.make.com/router)

이번 과제에서는 사실 2개 조건이 서로 배타적이라 큰 문제는 없지만, 나중에 조건이 겹치는 경우에는 **라우트 순서**를 신경 써야 합니다.

---

# 6. 합격 경로 만들기

이제 Route A, 즉 **점수 80 이상** 경로에 실제 액션을 붙입니다.

## 6-1. 결과 시트 기록 모듈 추가

합격 경로 첫 번째 액션으로 다음 모듈을 추가합니다.

**Google Sheets → Add a Row**

이 모듈에서 설정할 내용은:

- Spreadsheet: 결과 기록용 스프레드시트
- Sheet: `Processed_Results`

그리고 각 컬럼에 값을 매핑합니다.

예시:

- Timestamp → Form 응답의 제출 시간
- Name → 이름
- Email → 이메일
- Score → 점수
- Result → `"PASS"`
- Notification_Channel → `"Slack"` 또는 `"Discord"`
- Note → `"점수 80 이상"`

이렇게 하면 원본 응답과 별개로, 자동화 처리 결과가 정리된 별도 테이블이 생깁니다.

## 6-2. Slack 알림 추가

합격 경로 두 번째 액션으로:

**Slack → Send a Message**

를 붙입니다.

예시 메시지 템플릿은 이렇게 만들 수 있습니다.

> [합격 알림]  
> 이름: {{Name}}  
> 이메일: {{Email}}  
> 점수: {{Score}}  
> 결과: PASS

Slack(bot) 연결을 쓴다면, Make 문서상 **봇이 해당 채널 멤버여야** 정상 발송됩니다. 그렇지 않으면 `channel not found` 오류가 날 수 있습니다. [Source](https://apps.make.com/slack-modules)

### Slack 대신 Discord를 쓸 경우

대신

**Discord → Send a message**

를 사용하면 됩니다. Make의 Discord 모듈은 채널, 스레드, DM 대상 메시지 전송을 지원합니다. [Source](https://apps.make.com/discord)

예시 메시지는 비슷하게:

> ✅ 합격 처리 완료  
> 이름: {{Name}}  
> 이메일: {{Email}}  
> 점수: {{Score}}

처럼 쓰면 됩니다.

---

# 7. 불합격 경로 만들기

이제 Route B, 즉 **점수 80 미만** 경로를 만듭니다.

구조는 합격 경로와 거의 같습니다.

## 7-1. 결과 시트 기록

다시:

**Google Sheets → Add a Row**

를 추가하고, 같은 `Processed_Results` 시트에 기록하되 `Result` 값만 다르게 넣습니다.

예시:

- Result → `"FAIL"`
- Note → `"점수 80 미만"`

## 7-2. Slack/Discord 알림

알림 모듈을 붙이고 메시지를 이렇게 다르게 구성하면 됩니다.

> [불합격 알림]  
> 이름: {{Name}}  
> 이메일: {{Email}}  
> 점수: {{Score}}  
> 결과: FAIL

이렇게 하면 **같은 Trigger**에서 들어온 데이터가 **조건에 따라 서로 다른 결과 시트 기록과 메시지 전송**으로 나뉘게 됩니다.

---

# 8. 실제 테스트 방법

이 단계가 과제 제출에서 정말 중요합니다.  
그냥 시나리오만 만들어 두는 게 아니라 **분기 양쪽이 모두 실행된 증거**를 남겨야 합니다.

## 8-1. 테스트 데이터 2개 준비

폼에 아래처럼 2건을 제출하세요.

### 테스트 1

- 이름: 김합격
- 이메일: pass@test.com
- 점수: 90

### 테스트 2

- 이름: 이불합격
- 이메일: fail@test.com
- 점수: 65

## 8-2. 확인할 것

첫 번째 응답은:

- `Processed_Results`에 PASS로 기록
- Slack/Discord에 합격 메시지 전송

두 번째 응답은:

- `Processed_Results`에 FAIL로 기록
- Slack/Discord에 불합격 메시지 전송

이 두 결과가 모두 나오면, 과제 요구사항 중 **조건 분기 실동작 검증**이 완료됩니다.

---

# 9. 시나리오 스케줄 활성화

모듈 테스트가 끝나면 시나리오를 **ON**으로 켭니다.

여기서 알아둘 점은, 이 방식은 보통 **실시간 webhook 방식이 아니라 polling 방식**이기 때문에 즉시 반응하지 않을 수 있다는 것입니다. Google Form 응답을 시트에 저장한 뒤, Make가 새 행을 주기적으로 확인해서 처리하는 구조이기 때문입니다. Make 커뮤니티에서도 “폼 제출 즉시 실행”이 필요하면 Google Form 앱이나 Apps Script로 webhook을 보내는 대안을 언급합니다. [Source](https://community.make.com/t/google-sheets-watch-rows-trigger/55507)

과제용이라면 오히려 지금 설명한 **Google Sheets 감시 방식**이 훨씬 구현이 쉽고 안정적입니다.

---

# 10. 가장 추천하는 실제 설정 예시

구성 예시를 한 번에 보면 아래와 같습니다.

## 시나리오 흐름

1. **Google Sheets / Watch New Rows**
    
    - 대상: `Form_Responses`
2. **Router**
    
3. **Route A 필터**
    
    - `Score >= 80`
4. **Google Sheets / Add a Row**
    
    - 대상: `Processed_Results`
    - Result = PASS
5. **Slack / Send a Message**
    
    - 또는 Discord / Send a message
6. **Route B 필터**
    
    - `Score < 80`
7. **Google Sheets / Add a Row**
    
    - 대상: `Processed_Results`
    - Result = FAIL
8. **Slack / Send a Message**
    
    - 또는 Discord / Send a message

이 구조는 과제 요구사항인 **Trigger 1개 이상, Action 2개 이상, 조건 분기 1개 이상**을 모두 충족합니다.

---

# 11. 실수하기 쉬운 포인트

## 점수가 숫자가 아니라 문자열로 들어오는 문제

가장 흔한 문제입니다.  
예를 들어 `"90"`이 문자열로 들어오면 필터 비교가 기대와 다르게 동작할 수 있습니다.  
그래서 폼 입력 자체를 숫자로 제한하거나, 필터 단계에서 숫자 변환을 의식해야 합니다.

## 결과 시트를 원본 응답 시트와 같은 곳에 기록하는 문제

이건 되도록 피하세요.  
같은 시트를 트리거와 기록 대상으로 동시에 쓰면 새 행 추가가 다시 트리거가 되어 루프가 생길 수 있습니다.

## Slack 채널 오류

Slack bot이 채널에 들어가 있지 않으면 메시지가 안 갑니다. Make Slack 모듈 문서에서도 이 점을 명확히 안내합니다. [Source](https://apps.make.com/slack-modules)

## 무료 플랜에서 즉시 실행 기대

Google Sheets 감시 방식은 즉시 실행이 아닐 수 있습니다.  
과제에서는 “자동화가 정상 동작함”이 중요하므로, 이 부분은 보고서에 **polling 기반이라 약간의 지연이 있을 수 있다**고 적어두면 오히려 이해도가 높아 보입니다. [Source](https://community.make.com/t/google-sheets-watch-rows-trigger/55507) [Source](https://www.make.com/en/pricing)

---

# 12. 보고서에 그대로 써도 되는 기술 설명

아래 문장은 과제 보고서에 거의 그대로 써도 됩니다.

**설계 이유**  
Google Form 응답은 연결된 Google Sheets에 자동 저장되므로, Make에서는 Google Sheets의 새 행을 감시하는 방식으로 안정적으로 트리거를 구성할 수 있다. [Source](https://support.google.com/docs/answer/2917686?hl=en) [Source](https://community.make.com/t/google-sheets-watch-rows-trigger/55507)

**조건 분기 방식**  
Make의 Router는 하나의 입력 흐름을 여러 경로로 분기하고, 각 경로에 필터 조건을 적용해 서로 다른 처리 로직을 실행할 수 있다. 라우트는 순차적으로 처리되며 필요하면 fallback route도 구성할 수 있다. [Source](https://help.make.com/router)

**알림 처리 방식**  
점수 기준을 만족하는지 여부에 따라 결과를 별도 Google Sheets에 기록하고, 이후 Slack 또는 Discord 메시지 전송 모듈을 통해 담당자에게 자동 알림을 보낸다. Slack 사용 시에는 봇이 대상 채널 멤버여야 하며, Discord 모듈은 채널·스레드·DM 전송을 지원한다. [Source](https://apps.make.com/slack-modules) [Source](https://apps.make.com/discord)

---

# 13. 제가 추천하는 최종 구현안

과제 제출용으로는 이렇게 하세요.

- 트리거: **Google Sheets / Watch New Rows**
- 분기: **Router**
- 합격 경로: **Google Sheets / Add a Row + Slack/Discord Send Message**
- 불합격 경로: **Google Sheets / Add a Row + Slack/Discord Send Message**
- 결과 기록 시트는 반드시 별도 사용

이 방식이 가장 재현성이 높고, 캡처도 잘 나오고, 설명도 쉽습니다.

원하시면 다음 답변에서 바로 이어서  
**“Make에서 모듈별 클릭 순서까지 포함한 초상세 가이드”** 또는  
**“과제 보고서 제출용으로 붙여넣을 수 있는 단계별 설명문”**  
형태로 정리해드릴게요.


----------------------------------
```
Make에서 모듈별 클릭 순서까지 포함한 초상세 구현 가이드를 작성해줘
```
 이번에는 정말 **손으로 그대로 따라 만들 수 있게**, **[Make](https://www.make.com/en/pricing)** 화면에서 **어디를 누르고, 어떤 모듈을 넣고, 어떤 값을 매핑해야 하는지**까지 최대한 촘촘하게 설명드릴게요.  
다만 먼저 한 줄로 정리하면, 이번 구현은 **Google Form 응답을 [Google Sheets](https://sheets.google.com/)에 저장한 뒤, Make에서 `Google Sheets > Watch New Rows`로 새 행을 감지하고, `Router`로 점수 기준 분기 후, `Google Sheets > Add a Row`와 `Slack/Discord` 메시지 모듈로 처리하는 방식**입니다. Google Form 응답은 연결된 시트로 저장할 수 있고, Make의 Watch New Rows는 새 행이 추가될 때 트리거되며, Router는 흐름을 여러 경로로 분기해 각 경로를 다른 조건으로 처리할 수 있습니다. [Source](https://support.google.com/docs/answer/2917686?hl=en) [Source](https://apps.make.com/google-sheets-modules) [Source](https://help.make.com/router)

---

# 0. 최종 완성 모습

최종 시나리오는 아래처럼 생기게 됩니다.

**Google Sheets: Watch New Rows → Router → 합격 경로(PASS) → Google Sheets: Add a Row → Slack/Discord: Send a Message**  
**Google Sheets: Watch New Rows → Router → 불합격 경로(FAIL) → Google Sheets: Add a Row → Slack/Discord: Send a Message**

여기서 중요한 점은, **폼 응답이 쌓이는 원본 시트**와 **자동화 결과를 적는 결과 시트**를 분리하는 것입니다. `Add a Row`는 시트 하단에 새 행을 추가하는 모듈이므로, 같은 시트를 트리거와 결과 기록 대상으로 동시에 쓰면 중복 실행이나 루프를 만들 수 있습니다. 또한 Watch New Rows는 시트에 빈 행이 있으면 그 아래의 이후 행을 처리하지 않을 수 있으니, 원본 시트와 결과 시트 모두 중간 빈 줄 없이 관리하는 것이 안전합니다. [Source](https://apps.make.com/google-sheets-modules)

---

# 1. [Google Forms](https://forms.google.com/) 준비

## 1-1. 폼 만들기

먼저 브라우저에서 [Google Forms](https://forms.google.com/)를 엽니다.  
빈 폼으로 새 폼을 만들고 제목을 예를 들어 **“점수 판정 테스트”** 정도로 넣습니다.

그다음 질문 3개를 만듭니다.

첫 번째는 `이름`, 두 번째는 `이메일`, 세 번째는 `점수`입니다.  
특히 `점수`는 나중에 Make에서 조건 분기를 해야 하므로 **숫자만 입력되게 설정**하는 것이 좋습니다. 단답형으로 만들고 응답 검증에서 숫자 조건을 걸어두면 뒤에서 필터 설정이 훨씬 안정적입니다.

## 1-2. 폼 응답을 [Google Sheets](https://sheets.google.com/)에 연결

폼 상단의 **Responses** 탭을 클릭합니다.  
그다음 초록색 스프레드시트 아이콘, 또는 **“Link to Sheets / Select destination for responses”**에 해당하는 버튼을 눌러 응답 저장용 시트를 만듭니다. Google 공식 도움말에 따르면 Form 응답은 Form 내부 요약에서 볼 수도 있고, 연결된 Google Sheet에 저장할 수도 있습니다. [Source](https://support.google.com/docs/answer/2917686?hl=en)

이때 새 시트를 만들면 보통 자동으로 응답 전용 스프레드시트가 생성됩니다.  
여기서는 이 파일을 **원본 응답 시트**로 쓰면 됩니다.

---

# 2. [Google Sheets](https://sheets.google.com/) 결과 시트 준비

이제 응답 시트와는 별도로 **결과 기록용 시트**를 하나 더 준비하세요.  
방법은 두 가지입니다. 같은 스프레드시트 안에 새 시트를 하나 추가해도 되고, 아예 별도 스프레드시트를 만들어도 됩니다. 과제 제출용으로는 **같은 파일 안에 다른 탭**으로 두는 게 가장 보기 쉽습니다.

추천 구조는 이렇습니다.

- 탭 1: `Form Responses 1` 또는 원본 응답 탭
- 탭 2: `Processed_Results`

`Processed_Results` 탭의 1행에는 아래 헤더를 미리 입력해 두세요.

`SubmittedAt | Name | Email | Score | Result | NotificationChannel | Note`

이렇게 헤더를 만들어 두면 Make의 `Add a Row` 모듈에서 컬럼별 매핑이 편해집니다. `Add a Row`는 새 행을 아래쪽에 추가하는 액션 모듈입니다. [Source](https://apps.make.com/google-sheets-modules)

---

# 3. [Make](https://www.make.com/en/pricing)에서 새 시나리오 만들기

이제 Make에 로그인합니다.  
로그인 후 대시보드에서 **Create a new scenario**를 클릭합니다. Make 도움말의 입문 가이드도 새 시나리오를 만들어 모듈을 연결하는 방식으로 설명합니다. [Source](https://help.make.com/create-your-first-scenario)

빈 캔버스가 열리면 가운데 큰 **+ 버튼** 또는 앱 검색 창이 보일 겁니다.  
여기서 첫 모듈을 추가합니다.

---

# 4. 첫 모듈 추가: [Google Sheets](https://apps.make.com/google-sheets) `Watch New Rows`

## 4-1. 모듈 선택 클릭 순서

캔버스 가운데 **+** 클릭  
→ 검색창에 **Google Sheets** 입력  
→ **Google Sheets** 선택  
→ 모듈 목록에서 **Watch New Rows** 선택

이 모듈은 새 행이 추가될 때 트리거되는 역할을 하며, Make 문서에는 “새 행이 추가되면 트리거된다”고 설명되어 있습니다. [Source](https://apps.make.com/google-sheets-modules)

## 4-2. Google Sheets 연결 만들기

처음 연결이면 모듈 창 안에 **Create a connection** 버튼이 보입니다.  
그 버튼을 클릭합니다. Make의 Google Sheets 연결 문서에서도 “Google Sheets 모듈을 추가한 뒤 `Create a connection`을 클릭하고, `Sign in with Google`로 인증한다”고 안내합니다. [Source](https://apps.make.com/google-sheets)

클릭 순서는 보통 다음과 같습니다.

**Create a connection**  
→ 연결 이름 입력(예: `My Google Sheets`)  
→ **Sign in with Google**  
→ 사용할 Google 계정 선택  
→ 권한 승인  
→ Make 화면으로 돌아오기

연결이 완료되면 이제 어느 시트를 볼지 선택할 수 있습니다. [Source](https://apps.make.com/google-sheets)

## 4-3. Watch New Rows 설정

이제 모듈 설정창에서 다음 항목을 채웁니다.

- **Spreadsheet**: Google Form 응답이 쌓이는 스프레드시트 선택
- **Sheet**: 보통 `Form Responses 1`
- 필요하다면 헤더가 있는 표 범위를 지정

여기서 매우 중요한 점이 하나 있습니다.  
원본 응답 시트 중간에 **완전히 빈 행이 있으면**, Make는 그 아래 행들을 더 이상 처리하지 않을 수 있습니다. 따라서 원본 응답 시트는 **중간 빈 줄 없이 연속된 데이터 구조**로 두는 것이 좋습니다. [Source](https://apps.make.com/google-sheets-modules)

---

# 5. 첫 테스트: `Run once`로 응답 잡히는지 확인

이제 아직 다른 모듈을 붙이기 전에, 트리거가 제대로 데이터를 읽는지 확인해야 합니다.

클릭 순서는 이렇습니다.

시나리오 하단 또는 빌더 하단의 **Run once** 클릭  
→ 이제 Make가 대기 상태로 들어감  
→ 브라우저 다른 탭에서 Google Form 열기  
→ 테스트 응답 1건 제출

예를 들어 이렇게 제출합니다.

- 이름: 홍길동
- 이메일: hong@test.com
- 점수: 85

Make의 스케줄 도움말에서도 수동 테스트용으로 `Run once`를 사용할 수 있다고 안내합니다. [Source](https://help.make.com/schedule-a-scenario)

응답을 제출하고 잠시 기다리면 `Watch New Rows` 모듈 위에 숫자 배지나 실행 흔적이 생깁니다.  
그 모듈을 클릭해서 **Output bundle**을 열어보면, 각 컬럼 값이 어떻게 들어오는지 볼 수 있습니다.

여기서 꼭 확인할 것은 세 가지입니다.

첫째, **점수 컬럼 이름이 무엇으로 잡히는지**입니다.  
둘째, **점수가 숫자처럼 보이는지**입니다.  
셋째, **제출 시간 컬럼이 어떤 이름으로 들어오는지**입니다.

이 이름을 그대로 나중에 필터와 메시지 매핑에 써야 합니다.

---

# 6. [Router](https://help.make.com/router) 추가

이제 조건 분기를 만들 차례입니다.  
Router는 Make에서 데이터를 여러 경로로 분기하고, 각 경로를 다른 조건으로 처리하는 기능입니다. 문서상 Router는 여러 체인으로 흐름을 나누고, 각 route에 필터를 둘 수 있으며, route는 순차적으로 처리됩니다. [Source](https://help.make.com/router)

## 6-1. Router 넣는 클릭 순서

`Watch New Rows` 모듈 오른쪽의 **+** 클릭  
→ 검색창에 **Flow Control** 또는 **Router** 입력  
→ **Router** 선택

그러면 한 개의 모듈에서 여러 갈래로 뻗어 나가는 분기점이 생깁니다.

## 6-2. 분기 개수 만들기

이번 과제는 합격/불합격 2개 분기면 충분하니, Router에서 **2개 경로**를 만들면 됩니다.  
일반적으로 Router를 추가한 뒤 오른쪽에 생기는 라우트에 계속 모듈을 붙이는 식으로 2갈래를 구성하면 됩니다.

---

# 7. 합격 경로 필터 만들기

이제 첫 번째 경로를 **합격 경로**로 설정합니다.

## 7-1. 합격 경로에 액션 모듈 하나 먼저 붙이기

Router의 첫 번째 가지 끝에 있는 **+** 버튼 클릭  
→ **Google Sheets** 검색  
→ **Add a Row** 선택

`Add a Row`는 새 행을 표 맨 아래에 추가하는 모듈입니다. 옵션으로 `Insert rows`와 `Overwrite`가 언급되는데, 이번 경우는 새 기록을 누적해야 하므로 일반적으로 새 행 추가 방식으로 쓰면 됩니다. [Source](https://apps.make.com/google-sheets-modules)

## 7-2. 필터 설정 위치

이제 `Router`와 `Add a Row` 사이의 선 위를 보면 작은 아이콘이 보이거나, 선을 클릭/우클릭하면 **Set up a filter** 같은 메뉴가 나옵니다.  
그걸 클릭합니다.

## 7-3. 필터 이름과 조건 입력

필터 이름은 예를 들어 `PASS`로 둡니다.

조건은 이렇게 만듭니다.

- 왼쪽 값: 점수 필드
- 연산자: **Greater than or equal to**
- 오른쪽 값: `80`

즉, **Score >= 80** 입니다.

만약 점수 필드가 문자열처럼 들어온다면, 숫자로 비교되도록 설정을 다시 확인해야 합니다. 그래서 앞단 Google Form에서 숫자 입력을 강제하는 것이 가장 편합니다.

---

# 8. 합격 경로 1번째 액션: [Google Sheets](https://apps.make.com/google-sheets-modules) `Add a Row`

이제 방금 추가한 `Add a Row` 모듈 내용을 채웁니다.

## 8-1. 시트 선택 클릭 순서

`Add a Row` 모듈 클릭  
→ Google 계정 연결 선택(아까 만든 연결 재사용)  
→ **Spreadsheet**에서 결과 기록용 스프레드시트 선택  
→ **Sheet**에서 `Processed_Results` 선택

## 8-2. 컬럼 매핑

이제 아래처럼 값을 하나씩 넣습니다.

- `SubmittedAt` → 원본 시트의 제출 시각
- `Name` → 이름
- `Email` → 이메일
- `Score` → 점수
- `Result` → 직접 텍스트로 `PASS`
- `NotificationChannel` → `Slack` 또는 `Discord`
- `Note` → `점수 80 이상`

여기서 중요한 건, `Name`, `Email`, `Score`는 **왼쪽 패널에서 트리거 모듈이 읽어온 값**을 클릭해서 매핑하는 것입니다.  
반대로 `PASS`, `Slack`, `점수 80 이상` 같은 값은 **직접 입력**해도 됩니다.

---

# 9. 합격 경로 2번째 액션: [Slack](https://apps.make.com/slack-modules) 메시지 보내기

이제 합격자 알림을 보내겠습니다.

## 9-1. Slack 모듈 추가 클릭 순서

합격 경로의 `Add a Row` 모듈 오른쪽 **+** 클릭  
→ 검색창에 **Slack** 입력  
→ **Slack** 선택  
→ **Send a Message** 선택

## 9-2. Slack 연결

처음 연결이면 Slack 워크스페이스 인증이 필요합니다.  
인증을 완료한 뒤 메시지를 보낼 채널을 고릅니다.

여기서 꼭 기억할 점은, Make Slack 문서상 **Slack(bot) connection을 사용할 경우 bot이 대상 채널 멤버여야 한다**는 것입니다. 채널에 봇이 없으면 `channel not found` 오류가 날 수 있습니다. [Source](https://apps.make.com/slack-modules)

## 9-3. 메시지 내용 작성

메시지 본문에는 이렇게 넣으면 됩니다.

`[합격 알림] 이름: {{Name}} 이메일: {{Email}} 점수: {{Score}} 결과: PASS}`

실제 입력할 때는 `{{Name}}`처럼 직접 타이핑하는 게 아니라, 오른쪽/왼쪽 매핑 패널에서 해당 필드를 클릭해 삽입하면 됩니다.

과제 제출용으로는 메시지를 조금 더 보기 좋게 이런 식으로 써도 좋습니다.

`✅ 합격 처리 완료 이름: [이름값] 이메일: [이메일값] 점수: [점수값] 결과: PASS`

---

# 10. 불합격 경로 만들기

이제 Router의 두 번째 가지를 **불합격 경로**로 구성합니다.

## 10-1. 두 번째 가지에 `Add a Row` 추가

Router 두 번째 경로의 **+** 클릭  
→ **Google Sheets**  
→ **Add a Row**

## 10-2. 필터 설정

`Router`와 두 번째 `Add a Row` 사이 선 클릭  
→ **Set up a filter**

필터 이름은 `FAIL`로 둡니다.

조건은:

- 왼쪽 값: 점수 필드
- 연산자: **Less than**
- 오른쪽 값: `80`

즉, **Score < 80** 입니다.

이렇게 하면 첫 번째 route는 80 이상만, 두 번째 route는 80 미만만 통과하게 됩니다.

## 10-3. 결과 시트 매핑

불합격용 `Add a Row`는 합격 쪽과 거의 같고, 아래 값만 다르게 합니다.

- `Result` → `FAIL`
- `NotificationChannel` → `Slack` 또는 `Discord`
- `Note` → `점수 80 미만`

나머지 `SubmittedAt`, `Name`, `Email`, `Score`는 동일하게 매핑하면 됩니다.

---

# 11. 불합격 경로 2번째 액션: [Slack](https://apps.make.com/slack-modules) 메시지 보내기

불합격 경로의 `Add a Row` 오른쪽 **+** 클릭  
→ **Slack**  
→ **Send a Message**

채널은 합격과 같은 곳을 써도 되고, 별도 채널을 써도 됩니다.

메시지는 이렇게 넣으면 됩니다.

`❌ 불합격 처리 완료 이름: [이름값] 이메일: [이메일값] 점수: [점수값] 결과: FAIL`

이로써 **각 경로마다 Action 2개**가 생깁니다.  
즉, 조건을 만족하면 **(1) 결과 시트 기록 + (2) Slack 알림**이 실행되는 구조입니다.

---

# 12. [Discord](https://apps.make.com/discord)로 바꾸고 싶을 때

Slack 대신 Discord를 쓰고 싶다면 구조는 똑같습니다.  
차이는 메시지 모듈만 바꾸면 됩니다. Make의 Discord 문서에 따르면 `Send a message` 모듈은 채널, 스레드, DM 대상으로 메시지를 보낼 수 있습니다. [Source](https://apps.make.com/discord)

클릭 순서는 다음처럼 바꾸면 됩니다.

`Add a Row` 오른쪽 **+**  
→ **Discord** 검색  
→ **Send a message** 선택  
→ 연결  
→ 전송 대상 방식 선택(채널 / 스레드 / DM)  
→ 채널 ID 또는 대상 지정  
→ 메시지 입력

과제용으로는 보통 **채널 메시지**가 가장 쉽습니다.  
Slack을 이미 쓰고 있지 않다면 Discord가 더 간단할 수도 있습니다.

---

# 13. 테스트 2번으로 분기 양쪽 검증하기

이제 시나리오가 완성됐으니 **양쪽 분기 모두 실행되는지** 확인해야 합니다.  
과제에서 이 부분이 중요합니다.

## 13-1. 첫 번째 테스트: 합격 케이스

시나리오 하단 **Run once** 클릭  
→ Google Form 제출

예시 응답:

- 이름: 김합격
- 이메일: pass@test.com
- 점수: 90

예상 결과는 이렇습니다.

원본 응답 시트에 새 행 1개가 생기고, Make가 그 행을 읽은 뒤 PASS 경로로 들어가 `Processed_Results`에 PASS를 기록하고 Slack/Discord에 합격 메시지를 보냅니다.

## 13-2. 두 번째 테스트: 불합격 케이스

다시 **Run once** 클릭  
→ Google Form에 새 응답 제출

예시 응답:

- 이름: 이불합격
- 이메일: fail@test.com
- 점수: 65

이번에는 FAIL 경로가 실행되어야 합니다.  
즉, 결과 시트에는 FAIL이 기록되고, 메시지도 불합격용 텍스트로 와야 합니다.

이 두 번의 테스트 결과를 캡처하면, **조건 분기 양방향 검증 증빙**이 됩니다.

---

# 14. [Make](https://help.make.com/schedule-a-scenario) 스케줄 켜기

테스트가 끝났으면 수동 실행 상태가 아니라 실제 시나리오로 켜야 합니다.  
Make의 스케줄 설정은 시나리오 빌더 상단 툴바의 시간 문구를 눌러 열 수 있으며, 기본값은 **Every 15 minutes**입니다. 문서에도 기본적으로 시나리오는 15분마다 실행되며, 툴바의 **Every 15 minutes**를 클릭해 설정을 바꿀 수 있다고 나와 있습니다. [Source](https://help.make.com/schedule-a-scenario)

클릭 순서는 보통 이렇습니다.

상단 툴바의 **Every 15 minutes** 클릭  
→ 원하는 주기 선택  
→ 저장  
→ 시나리오 **ON**

주의할 점은 이번 구조가 **Google Sheets의 새 행을 감시하는 polling 방식**이라, 완전한 즉시 실행이 아닐 수 있다는 것입니다. Make 커뮤니티에서도 Google Form 응답이 시트에 들어온 뒤 Watch Rows가 이를 감지하는 방식은 간격 기반이라고 설명되고, 더 즉시성이 필요하면 Google Form 앱이나 Apps Script 웹훅 방식을 고려하라고 언급합니다. [Source](https://community.make.com/t/google-sheets-watch-rows-trigger/55507)

---

# 15. 과제 제출용 캡처 순서

보고서용으로는 아래 순서대로 캡처하면 가장 깔끔합니다.

첫 번째 캡처는 Google Form 질문 화면입니다.  
두 번째는 Form 응답이 연결된 Google Sheets 원본 시트입니다.  
세 번째는 Make 시나리오 전체 캔버스입니다.  
네 번째는 Router 양쪽 필터 설정 화면입니다. Make Router는 여러 체인으로 분기하고 조건 필터를 걸 수 있는 구조입니다. [Source](https://help.make.com/router)  
다섯 번째는 PASS 실행 결과입니다.  
여섯 번째는 FAIL 실행 결과입니다.  
일곱 번째는 `Processed_Results` 시트의 누적 결과입니다.  
여덟 번째는 Slack 또는 Discord에 도착한 알림 메시지입니다. Slack의 경우 봇이 채널 구성원이어야 한다는 점도 함께 언급하면 좋습니다. [Source](https://apps.make.com/slack-modules)

---

# 16. 자주 막히는 오류와 해결

## [Google Sheets](https://apps.make.com/google-sheets-modules)에서 새 응답이 안 잡히는 경우

가장 먼저 원본 응답 시트에 **중간 빈 행이 있는지** 보세요. Make 문서상 시트에 빈 행이 있으면 그 뒤의 행을 처리하지 않을 수 있습니다. [Source](https://apps.make.com/google-sheets-modules)

둘째로, `Watch New Rows`가 **정확한 탭**을 보고 있는지 확인하세요.  
Google Form 응답은 보통 `Form Responses 1` 같은 탭에 들어가는데, 결과 시트 탭을 잘못 보고 있으면 당연히 트리거가 안 됩니다.

## [Slack](https://apps.make.com/slack-modules) 메시지가 안 가는 경우

가장 흔한 원인은 **Slack bot이 채널에 초대되지 않은 경우**입니다. 문서에 따르면 이때 `channel not found` 오류가 날 수 있습니다. [Source](https://apps.make.com/slack-modules)

## 점수 조건이 이상하게 분기되는 경우

대부분 점수 값 형식 때문입니다.  
Google Form에서 점수를 숫자형으로 강제하지 않으면 문자열 취급될 수 있어 비교가 꼬일 수 있습니다. 이때는 폼에서 숫자 입력만 허용하도록 바꾸고 다시 테스트하는 게 가장 빠릅니다.

## 너무 늦게 실행되는 경우

`Watch New Rows`는 즉시 웹훅형이 아니라 주기 감시형일 수 있습니다. 시나리오 스케줄 설정을 확인하고, 테스트 중에는 `Run once`로 검증하는 것이 좋습니다. 기본 스케줄은 15분 간격이며, 플랜에 따라 최소 간격이 달라질 수 있습니다. [Source](https://help.make.com/schedule-a-scenario) [Source](https://www.make.com/en/pricing)

---

# 17. 그대로 따라 쓰는 최종 클릭 경로 요약

아주 짧게 압축하면 클릭 순서는 이렇습니다.

**Google Form 생성**  
→ **Responses**  
→ **Google Sheets 연결** [Source](https://support.google.com/docs/answer/2917686?hl=en)

**Make 로그인**  
→ **Create a new scenario**  
→ **Google Sheets**  
→ **Watch New Rows**  
→ **Create a connection**  
→ **Sign in with Google**  
→ 원본 응답 시트 선택 [Source](https://apps.make.com/google-sheets) [Source](https://apps.make.com/google-sheets-modules)

**Run once**  
→ 폼 테스트 제출  
→ 트리거 출력 확인 [Source](https://help.make.com/schedule-a-scenario)

**Watch New Rows 오른쪽 +**  
→ **Router** [Source](https://help.make.com/router)

**Route 1 +**  
→ **Google Sheets > Add a Row**  
→ 필터 `Score >= 80`  
→ 결과 시트에 PASS 기록 [Source](https://apps.make.com/google-sheets-modules)

**Route 1 오른쪽 +**  
→ **Slack > Send a Message** 또는 **Discord > Send a message**  
→ 합격 메시지 작성 [Source](https://apps.make.com/slack-modules) [Source](https://apps.make.com/discord)

**Route 2 +**  
→ **Google Sheets > Add a Row**  
→ 필터 `Score < 80`  
→ 결과 시트에 FAIL 기록 [Source](https://apps.make.com/google-sheets-modules)

**Route 2 오른쪽 +**  
→ **Slack > Send a Message** 또는 **Discord > Send a message**  
→ 불합격 메시지 작성 [Source](https://apps.make.com/slack-modules) [Source](https://apps.make.com/discord)

**상단 Every 15 minutes 클릭**  
→ 스케줄 설정  
→ 시나리오 ON [Source](https://help.make.com/schedule-a-scenario)

---

#보고서 작성용 구현절차 정리

아래는 **보고서에 그대로 붙여넣을 수 있도록 문장형으로 정리한 구현 절차 설명**입니다.  
과제 제출 문체에 맞게 **설계 목적 → 구현 절차 → 테스트 → 결과 확인 → 유의사항** 흐름으로 구성했습니다.

---

# [Make](https://www.make.com/en/pricing)를 활용한 Google Form 응답 자동화 구현 절차

본 프로젝트에서는 [Google Forms](https://forms.google.com/)로 수집된 응답 데이터를 기반으로, 점수 조건에 따라 결과를 분기하고, 이를 [Google Sheets](https://sheets.google.com/)에 기록한 뒤 [Slack](https://apps.make.com/slack-modules) 또는 [Discord](https://apps.make.com/discord)로 알림을 전송하는 자동화 워크플로우를 [Make](https://www.make.com/en/pricing)에서 구현하였다. 전체 흐름은 “Google Form 제출 → Google Sheets에 응답 저장 → Make에서 새 행 감지 → Router를 이용한 조건 분기 → 결과 시트 기록 → 메신저 알림 전송” 구조로 설계하였다. Google Forms는 응답을 연결된 Google Sheets에 저장할 수 있으며, Make의 Google Sheets 모듈은 시트에 새 행이 추가될 때 이를 감지하여 후속 자동화를 실행할 수 있다. [Source](https://support.google.com/docs/answer/2917686?hl=en) [Source](https://apps.make.com/google-sheets-modules)

자동화를 구현하기에 앞서 먼저 Google Form을 생성하고, 응답 항목으로 이름, 이메일, 점수를 구성하였다. 이때 점수 항목은 이후 조건 분기에 사용되므로 숫자 데이터로 입력되도록 설정하였다. 이후 Google Form의 Responses 메뉴에서 응답 저장용 Google Sheets를 연결하여, 사용자가 폼을 제출할 때마다 새로운 응답이 한 행씩 자동으로 기록되도록 구성하였다. 이 과정은 Google Forms의 기본 응답 저장 기능을 활용한 것으로, 폼 응답 데이터를 구조화된 스프레드시트 형태로 자동 누적할 수 있다는 점에서 후속 자동화의 트리거로 적합하다. [Source](https://support.google.com/docs/answer/2917686?hl=en)

다음으로 자동화 결과를 저장할 별도의 시트를 준비하였다. 원본 응답 시트와 결과 기록 시트를 분리한 이유는, 동일한 시트를 트리거와 기록 대상으로 동시에 사용할 경우 새로 기록된 결과 행이 다시 트리거 조건에 포함되어 중복 실행 또는 반복 처리 문제가 발생할 수 있기 때문이다. 따라서 원본 응답은 Google Form이 연결된 응답 시트에 그대로 저장되도록 두고, 자동화 처리 결과는 별도의 `Processed_Results` 시트에 기록하도록 설계하였다. 이 결과 시트에는 제출 시각, 이름, 이메일, 점수, 판정 결과, 알림 채널, 비고 등의 컬럼을 미리 생성하여 후속 모듈에서 쉽게 매핑할 수 있도록 하였다. [Source](https://apps.make.com/google-sheets-modules)

이후 Make에 로그인한 뒤 새 시나리오를 생성하였다. 시나리오의 첫 번째 모듈로는 Google Sheets 앱의 `Watch New Rows` 모듈을 추가하였다. 이 모듈은 지정한 시트에 새로운 행이 추가될 때 이를 감지하는 트리거 역할을 수행한다. 설정 과정에서는 먼저 Google 계정을 Make에 연결한 뒤, Google Form 응답이 저장되는 스프레드시트와 해당 시트를 선택하였다. Make의 Google Sheets 연결 방식은 모듈 추가 후 `Create a connection`을 통해 Google 계정을 인증하고, 필요한 권한을 승인하는 형태로 구성된다. 또한 Watch New Rows 모듈은 시트에 완전히 빈 행이 존재할 경우 그 아래의 행들을 처리하지 않을 수 있으므로, 응답 시트는 중간 빈 줄 없이 연속적으로 관리하는 것이 중요하다. [Source](https://apps.make.com/google-sheets) [Source](https://apps.make.com/google-sheets-modules)

트리거 설정 후에는 시나리오 하단의 `Run once` 기능을 사용하여 테스트를 수행하였다. 이 기능은 시나리오를 즉시 1회 실행 대기 상태로 만들며, 사용자가 Google Form에 실제 응답을 제출하면 해당 응답이 Google Sheets에 기록되고, Make가 이를 읽어와 데이터 구조를 확인할 수 있도록 해준다. 이 단계에서는 특히 이름, 이메일, 점수, 제출 시각 등의 필드가 어떤 이름으로 출력되는지 확인하였으며, 이후 분기 조건과 메시지 매핑에 활용할 데이터가 정확히 추출되는지를 검증하였다. Make는 기본적으로 시나리오 스케줄을 통해 주기적으로 동작하지만, 테스트 단계에서는 Run once 기능을 사용하여 응답 처리 흐름을 수동으로 즉시 검증할 수 있다. [Source](https://help.make.com/schedule-a-scenario)

트리거 데이터 확인이 완료된 뒤에는 `Router` 모듈을 추가하여 조건 분기 구조를 구성하였다. Make의 Router는 하나의 입력 흐름을 여러 경로로 분기한 뒤, 각 경로에 서로 다른 필터 조건을 적용할 수 있도록 하는 모듈이다. 본 프로젝트에서는 점수를 기준으로 두 개의 경로를 구성하였다. 첫 번째 경로는 점수가 80점 이상인 경우를 처리하는 합격 경로로 설정하였고, 두 번째 경로는 점수가 80점 미만인 경우를 처리하는 불합격 경로로 설정하였다. 각 경로의 연결선에는 필터를 적용하여, 합격 경로에는 `Score >= 80`, 불합격 경로에는 `Score < 80` 조건이 충족될 때만 후속 모듈이 실행되도록 구성하였다. Router는 여러 경로를 통해 데이터를 다르게 처리할 수 있게 하며, 라우트는 순차적으로 처리된다는 점에서 조건 기반 워크플로우를 설명하기에 적합한 구조이다. [Source](https://help.make.com/router)

합격 경로에는 먼저 Google Sheets의 `Add a Row` 모듈을 추가하여 결과 기록 시트에 처리 결과를 저장하도록 하였다. 이 모듈은 시트 하단에 새 행을 추가하는 역할을 하며, 원본 응답 시트에서 읽어온 제출 시각, 이름, 이메일, 점수 값을 결과 시트의 대응 컬럼에 매핑하였다. 또한 판정 결과 컬럼에는 `PASS`, 알림 채널 컬럼에는 `Slack` 또는 `Discord`, 비고 컬럼에는 `점수 80 이상`과 같은 고정 문구를 직접 입력하여 저장하도록 설정하였다. Make의 `Add a Row` 모듈은 새 행을 추가하는 방식으로 결과를 누적 기록할 수 있어, 자동화 실행 이력을 남기고 후속 검토를 쉽게 할 수 있다는 장점이 있다. [Source](https://apps.make.com/google-sheets-modules)

합격 경로의 두 번째 액션으로는 Slack의 `Send a Message` 모듈을 추가하여 판정 결과를 메신저 채널로 전송하도록 설정하였다. 메시지 본문에는 이름, 이메일, 점수, 최종 결과를 포함하여 담당자가 즉시 내용을 파악할 수 있도록 구성하였다. 예를 들어 “합격 처리 완료, 이름: ○○○, 이메일: ○○○, 점수: ○○점, 결과: PASS”와 같은 형식으로 메시지를 작성하였다. Slack을 사용할 경우에는 Make에서 연결한 Slack bot이 대상 채널의 멤버여야 정상적으로 메시지를 전송할 수 있으며, 채널에 참여하지 않은 상태에서는 `channel not found` 오류가 발생할 수 있다. 따라서 메시지 전송 전에 봇이 해당 채널에 초대되어 있는지 반드시 확인하였다. [Source](https://apps.make.com/slack-modules)

불합격 경로 역시 동일한 방식으로 구성하였다. 우선 Google Sheets의 `Add a Row` 모듈을 추가하여 결과 시트에 불합격 데이터를 기록하였고, 이때 판정 결과는 `FAIL`, 비고는 `점수 80 미만`으로 입력하였다. 이후 Slack 또는 Discord의 메시지 전송 모듈을 통해 불합격 알림이 자동 발송되도록 설정하였다. 이 메시지 역시 이름, 이메일, 점수, 결과를 포함하여 구성하였으며, 합격 경로와 동일한 구조를 사용하되 메시지 문구와 결과값만 다르게 설정하였다. 이와 같이 같은 트리거 데이터를 Router를 통해 두 개의 경로로 분기하고, 각 경로에서 서로 다른 기록과 알림을 수행하도록 함으로써 조건 분기 자동화 구조를 명확히 구현할 수 있었다. [Source](https://apps.make.com/google-sheets-modules) [Source](https://apps.make.com/discord) [Source](https://apps.make.com/slack-modules)

구현 완료 후에는 실제 테스트 데이터를 사용하여 분기 동작을 검증하였다. 첫 번째 테스트에서는 점수를 90점으로 입력한 응답을 제출하여 합격 경로가 정상 실행되는지 확인하였다. 그 결과 원본 응답 시트에 새 행이 추가된 뒤, Make가 해당 행을 감지하여 결과 시트에 `PASS`로 기록하고 Slack 또는 Discord 채널에 합격 알림을 전송하는 것을 확인하였다. 두 번째 테스트에서는 점수를 65점으로 입력하여 불합격 경로를 검증하였으며, 이번에는 결과 시트에 `FAIL`이 기록되고 불합격 알림 메시지가 전송되는 것을 확인하였다. 이를 통해 동일한 입력 구조가 점수 조건에 따라 서로 다른 처리 흐름으로 분기된다는 점을 실증적으로 확인하였다. [Source](https://help.make.com/router) [Source](https://apps.make.com/google-sheets-modules)

마지막으로 시나리오를 실제 운영 가능한 상태로 전환하기 위해 Make의 스케줄 설정을 확인하였다. Make의 시나리오는 기본적으로 15분 간격으로 실행되도록 설정되며, 시나리오 빌더 상단의 스케줄 설정 메뉴에서 실행 주기와 방식을 변경할 수 있다. 본 프로젝트에서 사용한 `Watch New Rows` 방식은 Google Form이 직접 Make를 즉시 호출하는 구조가 아니라, Google Sheets에 새 응답 행이 생성된 뒤 Make가 이를 감지하는 방식이므로 즉시 실행보다는 주기적 확인에 가까운 구조이다. 따라서 보고서에서는 이 자동화가 “폼 제출 직후 곧바로”가 아니라 “시트 업데이트 감지 주기에 따라” 실행될 수 있음을 함께 기술하였다. 이는 구현 한계를 숨기기보다 시스템 동작 원리를 정확히 이해하고 있음을 보여주는 설명이기도 하다. [Source](https://help.make.com/schedule-a-scenario) [Source](https://community.make.com/t/google-sheets-watch-rows-trigger/55507)

종합적으로 본 구현은 Google Form 응답 수집, Google Sheets 기반 데이터 저장, Make의 Watch New Rows 트리거, Router를 활용한 조건 분기, Add a Row를 통한 결과 기록, 그리고 Slack 또는 Discord를 통한 자동 알림 전송까지 하나의 흐름으로 연결한 사례이다. 이를 통해 단순한 데이터 수집을 넘어, 입력값에 따라 자동으로 판정하고 결과를 기록하며 담당자에게 즉시 전달하는 실용적인 노코드 자동화 워크플로우를 구현할 수 있었다. 또한 Trigger, Action, 조건 분기 요소를 모두 포함하고 있어 과제의 필수 요구사항을 충족하는 사례로 활용할 수 있다. [Source](https://apps.make.com/google-sheets-modules) [Source](https://help.make.com/router) [Source](https://apps.make.com/slack-modules)

---


![[Pasted image 20260713094607.png]]


![[Pasted image 20260713094646.png]]

![[Pasted image 20260713094707.png]]

![[Pasted image 20260713111047.png]]







