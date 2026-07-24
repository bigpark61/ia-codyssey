Zapier 에서 Google Form 제출 → 조건 분기(점수 기준) → Google Sheets 기록 + Slack/Discord 알림을 자동화 하려고해
Zaiper 에서 실제 구현하는 절차와 방법에 대해서 자세히 설명해줘

1. Zapier 에서 실제 구현하는 절차
2. Zapier에서 모듈별 클릭순서 등 상세 설명
3. 작성용 구현절차 정리


 **[Zapier](https://zapier.com/pricing)** 에서  
**Google Form 제출 → 점수 기준 조건 분기 → Google Sheets 기록 → Slack/Discord 알림**  
을 실제로 만드는 절차를 **실습 가이드 방식으로 자세히** 설명드릴게요.

먼저 핵심부터 말하면, 이 자동화는 **[Google Forms](https://help.zapier.com/hc/en-us/articles/8495997064845-How-to-get-started-with-Google-Forms-on-Zapier)의 `New Form Response (Instant)` 트리거**로 시작하고, 그 다음 **[Paths by Zapier](https://help.zapier.com/hc/en-us/articles/8496288555917-Add-branching-logic-to-Zap-workflows-with-Paths)** 로 점수에 따라 PASS/FAIL 경로를 나눈 뒤, 각 경로에서 **[Google Sheets](https://help.zapier.com/hc/en-us/articles/8495994922509-How-to-get-started-with-Google-Sheets-on-Zapier) `Create Spreadsheet Row`** 와 **[Slack](https://help.zapier.com/hc/en-us/articles/8495993391629-How-to-get-started-with-Slack-on-Zapier) 또는 [Discord](https://help.zapier.com/hc/en-us/articles/8496048271501-How-to-get-started-with-Discord-on-Zapier) 메시지 전송 액션**을 붙이면 됩니다. Google Forms는 Zapier에서 **New Form Response (Instant)** 트리거를 지원하고, Paths는 조건별로 서로 다른 액션을 실행하는 분기 기능입니다. [Source](https://help.zapier.com/hc/en-us/articles/8495997064845-How-to-get-started-with-Google-Forms-on-Zapier) [Source](https://help.zapier.com/hc/en-us/articles/8496288555917-Add-branching-logic-to-Zap-workflows-with-Paths)

다만 시작 전에 꼭 알아야 할 점이 있습니다. **이 구조에서 가장 중요한 기능은 Paths**인데, Zapier의 무료 플랜은 **100 tasks/month, two-step Zap workflows**까지만 포함합니다. 즉, 이번처럼 **트리거 + 조건 분기 + 시트 기록 + 메신저 알림** 구조를 하나의 Zap 안에서 만들려면 보통 **Professional 이상 플랜**으로 작업하는 것이 안전합니다. 또한 Zapier 도움말에서도 Filters와 Paths는 유료 플랜 기준으로 안내됩니다. [Source](https://zapier.com/pricing) [Source](https://help.zapier.com/hc/en-us/articles/8496276332557-Add-conditions-to-Zap-workflows-with-filters) [Source](https://help.zapier.com/hc/en-us/articles/8496288555917-Add-branching-logic-to-Zap-workflows-with-Paths)

---

# 1. 구현 전에 준비할 것

먼저 [Google Forms](https://forms.google.com/)에서 폼을 하나 만듭니다.  
예를 들어 질문은 `이름`, `이메일`, `점수` 3개면 충분합니다. 여기서 가장 중요한 것은 `점수` 항목을 **숫자로 받도록 설계**하는 것입니다. Zapier에서 조건 분기를 만들 때 숫자 비교를 해야 하므로, 가능하면 Google Form 단답형에 숫자 검증을 걸어 두는 편이 좋습니다.

그 다음 이 폼의 응답이 [Google Sheets](https://sheets.google.com/)에 저장되도록 연결해 두는 것을 권장합니다. Zapier의 Google Forms 문서에는 `New Form Response (Instant)` 트리거가 있지만, 동시에 **Google Form responses must be saved to a Google Sheet** 라는 제한사항도 안내하고 있습니다. 즉, 실제 운영에서는 **폼 응답이 시트에 저장되도록 연결된 상태**로 두는 것이 안정적입니다. [Source](https://help.zapier.com/hc/en-us/articles/8495997064845-How-to-get-started-with-Google-Forms-on-Zapier)

그리고 결과를 저장할 **별도의 Google Sheets 시트**도 준비합니다.  
추천 구조는 다음과 같습니다.

- 원본 응답 시트: Google Form이 자동으로 채우는 시트
- 결과 시트: Zapier가 PASS/FAIL 결과를 적는 시트

결과 시트에는 미리 아래 같은 헤더를 만들어 두세요.

`SubmittedAt | Name | Email | Score | Result | NotificationChannel | Note`

이렇게 분리해 두면 나중에 기록 구조가 깔끔하고, 원본 응답과 자동화 처리 결과를 쉽게 비교할 수 있습니다. Zapier의 Google Sheets 액션인 `Create Spreadsheet Row`는 지정한 시트에 새 행을 추가하는 데 적합합니다. [Source](https://help.zapier.com/hc/en-us/articles/8495994922509-How-to-get-started-with-Google-Sheets-on-Zapier)

---

# 2. [Zapier](https://zapier.com/pricing)에서 계정 연결하기

Zapier에 로그인한 뒤, 미리 앱 연결을 해 두면 작업이 편합니다.  
Zapier 도움말 기준으로 앱 연결은 보통 **Apps 페이지 → + Add connection → 앱 선택 → 로그인 및 권한 승인** 방식으로 진행됩니다. 이 구조는 Google Forms, Google Sheets, Slack, Discord 모두 동일합니다. [Source](https://help.zapier.com/hc/en-us/articles/8495997064845-How-to-get-started-with-Google-Forms-on-Zapier) [Source](https://help.zapier.com/hc/en-us/articles/8495994922509-How-to-get-started-with-Google-Sheets-on-Zapier) [Source](https://help.zapier.com/hc/en-us/articles/8495993391629-How-to-get-started-with-Slack-on-Zapier) [Source](https://help.zapier.com/hc/en-us/articles/8496048271501-How-to-get-started-with-Discord-on-Zapier)

연결할 앱은 최소한 다음 3개입니다.

- Google Forms
- Google Sheets
- Slack 또는 Discord

Slack을 쓸 경우, 워크스페이스 앱 설치 권한이 제한된 환경이라면 관리자 권한이 필요할 수 있습니다. 또한 무료 Slack 워크스페이스는 앱 슬롯 제한이 있을 수 있습니다. [Source](https://help.zapier.com/hc/en-us/articles/8495993391629-How-to-get-started-with-Slack-on-Zapier)

Discord를 쓸 경우에는 서버 소유자 또는 매니저 권한, 그리고 **Manage Webhooks 권한**이 필요합니다. [Source](https://help.zapier.com/hc/en-us/articles/8496048271501-How-to-get-started-with-Discord-on-Zapier)

---

# 3. [Zapier](https://zapier.com/pricing)에서 새 Zap 만들기

Zapier 대시보드에서 **Create** 또는 **Create Zap** 버튼을 눌러 새 Zap을 만듭니다.  
이후 첫 단계는 항상 Trigger 설정입니다.

---

# 4. 트리거 설정: [Google Forms](https://help.zapier.com/hc/en-us/articles/8495997064845-How-to-get-started-with-Google-Forms-on-Zapier) `New Form Response (Instant)`

이번 자동화에서 가장 자연스러운 트리거는 **Google Forms → New Form Response (Instant)** 입니다. Zapier 문서에도 Google Forms는 `New Form Response (Instant)` 와 `New or Updated Form Response (Instant)` 트리거를 지원한다고 나와 있습니다. [Source](https://help.zapier.com/hc/en-us/articles/8495997064845-How-to-get-started-with-Google-Forms-on-Zapier)

실제 클릭 흐름은 보통 아래와 같습니다.

**Trigger** 단계에서 앱으로 **Google Forms** 선택  
→ Trigger event에서 **New Form Response** 선택  
→ 연결해 둔 Google 계정 선택  
→ 대상 폼 선택  
→ **Test trigger** 실행

여기서 Test trigger를 누르면 Zapier가 최근 제출된 응답 하나를 샘플 데이터로 불러옵니다.  
이 단계에서 꼭 확인해야 할 것은 아래 3가지입니다.

첫째, 이름/이메일/점수 필드가 제대로 들어오는지.  
둘째, 점수 값이 숫자로 해석 가능한 형태인지.  
셋째, 제출 시각이나 응답 ID 같은 부가 필드가 필요한 경우 함께 들어오는지.

테스트용 응답이 없다면, 먼저 Google Form에 본인이 직접 응답을 1건 제출한 뒤 다시 Test trigger를 누르면 됩니다.

---

# 5. 조건 분기 추가: [Paths by Zapier](https://help.zapier.com/hc/en-us/articles/8496288555917-Add-branching-logic-to-Zap-workflows-with-Paths)

트리거가 정상적으로 동작하면, 다음 단계로 **Paths**를 추가합니다.  
Zapier의 Paths는 “If A then X / If B then Y” 구조로 분기하는 기능입니다. 점수 80 이상이면 PASS 경로로, 80 미만이면 FAIL 경로로 보내려면 Paths가 가장 적합합니다. [Source](https://help.zapier.com/hc/en-us/articles/8496288555917-Add-branching-logic-to-Zap-workflows-with-Paths)

또한 Zapier는 Paths를 사용할 때 경로들이 **동시에 실행되는 것이 아니라 순차적으로, 왼쪽에서 오른쪽 순서로 평가 및 실행**된다고 설명합니다. 즉, 경로 순서도 어느 정도 의미가 있습니다. [Source](https://help.zapier.com/hc/en-us/articles/8496288555917-Add-branching-logic-to-Zap-workflows-with-Paths)

실제 클릭 흐름은 다음과 같습니다.

트리거 아래 **+** 클릭  
→ 앱 검색창에 **Paths** 입력  
→ **Paths by Zapier** 선택  
→ 기본 2개 경로 생성

보통 Path A와 Path B가 기본으로 만들어집니다.  
여기서 다음처럼 역할을 정하면 됩니다.

- Path A: 합격 경로
- Path B: 불합격 경로

---

# 6. Path A 설정: 점수 80 이상이면 합격 처리

이제 첫 번째 경로를 설정합니다.  
Path A 안에서 **Path rules** 또는 규칙 설정 화면으로 들어간 뒤, **Custom rules**를 사용합니다.

규칙은 이렇게 잡습니다.

- Field: 점수
- Condition: **(Number) Greater than or equal to**
- Value: `80`

즉, `Score >= 80` 입니다.

Zapier의 조건 로직 문서에서는 Filter와 Paths 모두 **field / rule / value** 구조로 조건을 만든다고 설명합니다. 숫자 비교를 쓸 때는 숫자 규칙을 선택하는 것이 중요합니다. [Source](https://help.zapier.com/hc/en-us/articles/34372501750285-Use-conditional-logic-to-filter-and-split-your-Zap-workflows) [Source](https://help.zapier.com/hc/en-us/articles/8496276332557-Add-conditions-to-Zap-workflows-with-filters)

설정 후에는 **Test path**를 눌러 샘플 데이터가 실제로 이 경로를 통과하는지 확인합니다.  
점수 85 같은 샘플을 불러왔다면 PASS 경로가 통과해야 정상입니다.

---

# 7. Path A 안에서 [Google Sheets](https://help.zapier.com/hc/en-us/articles/8495994922509-How-to-get-started-with-Google-Sheets-on-Zapier) 기록하기

이제 PASS 경로 안에 첫 액션으로 **Google Sheets → Create Spreadsheet Row** 를 추가합니다. Zapier의 Google Sheets 앱은 `Create Spreadsheet Row` 액션을 지원합니다. [Source](https://help.zapier.com/hc/en-us/articles/8495994922509-How-to-get-started-with-Google-Sheets-on-Zapier)

실제 흐름은 다음과 같습니다.

Path A 내부에서 **+** 클릭  
→ **Google Sheets** 선택  
→ Action event로 **Create Spreadsheet Row** 선택  
→ Google Sheets 계정 선택  
→ 결과 기록용 스프레드시트 선택  
→ 결과 시트 탭 선택

이후 컬럼별 매핑을 합니다.

- `SubmittedAt` → Google Forms 트리거의 제출 시각
- `Name` → 폼의 이름 응답
- `Email` → 폼의 이메일 응답
- `Score` → 폼의 점수 응답
- `Result` → 직접 `PASS` 입력
- `NotificationChannel` → `Slack` 또는 `Discord`
- `Note` → 직접 `점수 80 이상` 입력

이렇게 하면 합격자 데이터가 결과 시트에 누적됩니다.

---

# 8. Path A 안에서 [Slack](https://help.zapier.com/hc/en-us/articles/8495993391629-How-to-get-started-with-Slack-on-Zapier) 또는 [Discord](https://help.zapier.com/hc/en-us/articles/8496048271501-How-to-get-started-with-Discord-on-Zapier) 알림 보내기

다음으로 PASS 경로 안에 두 번째 액션을 추가합니다.

Slack을 쓸 경우에는 **Slack → Send Channel Message** 를 선택하면 됩니다. Zapier의 Slack 앱은 `Send Channel Message` 액션을 지원하며, 채널 메시지 또는 스레드 답글 전송도 가능합니다. [Source](https://help.zapier.com/hc/en-us/articles/8495993391629-How-to-get-started-with-Slack-on-Zapier)

메시지 예시는 다음처럼 구성하면 됩니다.

`✅ 합격 처리 완료 이름: [이름] 이메일: [이메일] 점수: [점수] 결과: PASS`

여기서 주의할 점이 있습니다. Slack에서 `channel_not_found` 오류가 날 수 있는데, Zapier 도움말에 따르면 그 원인은 보통 **연결된 Slack 계정이 해당 채널 멤버가 아니거나**, 잘못된 워크스페이스 연결을 사용했거나, 채널 값을 잘못 넣었기 때문입니다. [Source](https://help.zapier.com/hc/en-us/articles/31661561373837-Slack-error-channel-not-found)

Discord를 쓰고 싶다면 **Discord → Send Channel Message** 를 선택하면 됩니다. Discord 앱은 채널 메시지 액션을 지원하며, 사전 요구사항으로 서버 관리 권한과 웹훅 권한이 필요합니다. [Source](https://help.zapier.com/hc/en-us/articles/8496048271501-How-to-get-started-with-Discord-on-Zapier)

Discord 메시지 예시는 이렇게 둘 수 있습니다.

`✅ 합격 처리 완료 이름: [이름] 이메일: [이메일] 점수: [점수] 결과: PASS`

---

# 9. Path B 설정: 점수 80 미만이면 불합격 처리

이제 두 번째 경로를 설정합니다.  
Path B에서는 다음 규칙을 둡니다.

- Field: 점수
- Condition: **(Number) Less than**
- Value: `80`

즉, `Score < 80` 입니다.

규칙 저장 후 Test path를 실행해 샘플 데이터가 이 조건에서 통과하는지 확인합니다.  
만약 현재 샘플이 90점이면 이 경로는 통과하지 않는 것이 정상입니다. 이런 경우에는 테스트용 응답을 새로 제출해서 65점 같은 샘플을 불러오는 것이 좋습니다.

---

# 10. Path B 안에서 [Google Sheets](https://help.zapier.com/hc/en-us/articles/8495994922509-How-to-get-started-with-Google-Sheets-on-Zapier) 기록하기

불합격 경로 안에서도 똑같이 **Create Spreadsheet Row** 액션을 추가합니다.

매핑 구조는 합격 경로와 거의 같고, 아래만 다르게 하면 됩니다.

- `Result` → `FAIL`
- `NotificationChannel` → `Slack` 또는 `Discord`
- `Note` → `점수 80 미만`

이렇게 하면 불합격자도 결과 시트에 별도로 기록됩니다.

---

# 11. Path B 안에서 [Slack](https://help.zapier.com/hc/en-us/articles/8495993391629-How-to-get-started-with-Slack-on-Zapier) 또는 [Discord](https://help.zapier.com/hc/en-us/articles/8496048271501-How-to-get-started-with-Discord-on-Zapier) 알림 보내기

이제 Path B의 마지막 액션으로 메시지 전송 단계를 붙입니다.

Slack을 쓸 경우 `Send Channel Message` 를 선택하고, 메시지는 예를 들어 다음처럼 만듭니다.

`❌ 불합격 처리 완료 이름: [이름] 이메일: [이메일] 점수: [점수] 결과: FAIL`

Discord도 동일하게 `Send Channel Message` 를 쓰면 됩니다. Discord 쪽은 메시지 길이 제한과 권한 요건이 있다는 점만 기억하면 됩니다. [Source](https://help.zapier.com/hc/en-us/articles/8496048271501-How-to-get-started-with-Discord-on-Zapier)

---

# 12. 최종 테스트 방법

이제 Zap 전체 구조가 완성되었습니다.  
반드시 **양쪽 경로를 따로 테스트**해야 합니다.

먼저 Google Form에 테스트 응답 1건을 제출합니다.

- 이름: 김합격
- 이메일: pass@test.com
- 점수: 90

이 경우에는 Path A가 실행되어야 합니다.  
즉, 결과 시트에 PASS가 기록되고 Slack 또는 Discord에 합격 메시지가 와야 합니다.

그 다음 두 번째 응답을 제출합니다.

- 이름: 이불합격
- 이메일: fail@test.com
- 점수: 65

이번에는 Path B가 실행되어야 합니다.  
즉, 결과 시트에 FAIL이 기록되고, 불합격 메시지가 와야 합니다.

이 두 테스트가 모두 정상 동작하면, 과제 요구사항인 **트리거 + 조건 분기 + 2개 이상 액션**을 모두 충족했다고 볼 수 있습니다.

---

# 13. [Zapier](https://zapier.com/pricing)에서 켜기 전에 알아둘 점

이번 구조는 Google Forms 트리거 자체는 **Instant** 로 제공되기 때문에, Make에서 Google Sheets를 polling하는 방식보다 더 자연스럽게 “폼 제출 → Zap 실행” 흐름을 만들 수 있습니다. Google Forms 앱 문서에는 `New Form Response (Instant)` 트리거가 명시되어 있습니다. [Source](https://help.zapier.com/hc/en-us/articles/8495997064845-How-to-get-started-with-Google-Forms-on-Zapier)

반면, Slack과 Discord는 각각 연결 권한 이슈가 자주 발생합니다.  
Slack은 채널 접근권과 채널 멤버십 문제가 흔하고, Discord는 서버 권한 및 웹훅 권한이 중요합니다. [Source](https://help.zapier.com/hc/en-us/articles/31661561373837-Slack-error-channel-not-found) [Source](https://help.zapier.com/hc/en-us/articles/8496048271501-How-to-get-started-with-Discord-on-Zapier)

또 하나 중요한 점은, **조건 분기가 꼭 필요하다면 Filter보다 Paths가 적합**하다는 것입니다. Zapier 문서에 따르면 Filter는 “조건을 만족하면 계속, 아니면 중단”하는 **선형 흐름**에 적합하고, Paths는 “조건에 따라 서로 다른 후속 액션을 수행하는 **분기형 흐름**”에 적합합니다. 이번 과제처럼 PASS와 FAIL 두 방향 모두에서 액션이 필요하다면 Paths가 더 알맞습니다. [Source](https://help.zapier.com/hc/en-us/articles/34372501750285-Use-conditional-logic-to-filter-and-split-your-Zap-workflows)

---

# 14. 가장 추천하는 최종 구성

이번 과제 목적이라면 아래 구성이 가장 깔끔합니다.

**Trigger**는 [Google Forms](https://help.zapier.com/hc/en-us/articles/8495997064845-How-to-get-started-with-Google-Forms-on-Zapier)의 `New Form Response (Instant)`  
**Branching**은 [Paths by Zapier](https://help.zapier.com/hc/en-us/articles/8496288555917-Add-branching-logic-to-Zap-workflows-with-Paths)  
**Record**는 [Google Sheets](https://help.zapier.com/hc/en-us/articles/8495994922509-How-to-get-started-with-Google-Sheets-on-Zapier)의 `Create Spreadsheet Row`  
**Notify**는 [Slack](https://help.zapier.com/hc/en-us/articles/8495993391629-How-to-get-started-with-Slack-on-Zapier)의 `Send Channel Message` 또는 [Discord](https://help.zapier.com/hc/en-us/articles/8496048271501-How-to-get-started-with-Discord-on-Zapier)의 `Send Channel Message`

이 구조가 가장 설명하기 쉽고, 실제 캡처도 잘 나오고, 보고서 작성에도 유리합니다.

원하시면 다음 답변에서 바로 이어서  
**“Zapier에서 모듈별 클릭 순서까지 포함한 초상세 가이드”** 또는  
**“보고서에 그대로 붙여넣을 수 있는 Zapier 구현 절차 문장형 설명”**  
으로 정리해드릴게요.

---
이번에는 **[Zapier](https://zapier.com/pricing)** 화면에서 실제로 따라 할 수 있도록,  
**어디를 클릭하고 → 어떤 앱을 선택하고 → 어떤 조건을 넣고 → 어떤 값을 매핑하는지**까지 포함한 **초상세 구현 가이드**로 정리해드릴게요.

이번 목표는 다음 자동화입니다.

**Google Form 제출 → 점수 기준 조건 분기 → Google Sheets 기록 → Slack/Discord 알림**

먼저 아주 중요한 전제부터 말씀드리면, 이 구조의 핵심인 **Paths**는 분기형 워크플로우를 만들 때 쓰는 기능입니다. Zapier 도움말에 따르면 Paths는 “If A면 X, If B면 Y” 방식의 조건 분기이며, 경로는 왼쪽에서 오른쪽으로 **순차 실행**됩니다. 또한 무료 플랜은 **100 tasks/month, two-step Zap workflows** 중심이므로, 이번처럼 분기와 여러 액션이 포함된 Zap은 보통 **Professional 이상 플랜**에서 구현하는 것이 현실적입니다. [Source](https://help.zapier.com/hc/en-us/articles/8496288555917-Add-branching-logic-to-Zap-workflows-with-Paths) [Source](https://zapier.com/pricing)

---

# 0. 완성 구조를 먼저 이해하기

최종적으로 Zap은 아래처럼 구성됩니다.

1. **Trigger**: Google Forms - New Form Response
2. **Paths**: 점수 기준 분기
    - **Path A**: 점수 80 이상
    - **Path B**: 점수 80 미만
3. **Path A 액션**
    - Google Sheets - Create Spreadsheet Row
    - Slack/Discord - Send Channel Message
4. **Path B 액션**
    - Google Sheets - Create Spreadsheet Row
    - Slack/Discord - Send Channel Message

즉, **하나의 폼 응답**이 들어오면 점수에 따라 다른 경로로 가고, 각 경로에서 **결과 시트 기록 + 메신저 알림**을 수행합니다. Zapier 문서상 이런 다중 결과 처리는 Filter보다 Paths가 더 적합합니다. Filter는 “통과/중단”형 선형 흐름에, Paths는 “여러 후속 경로”가 필요한 분기형 흐름에 적합합니다. [Source](https://help.zapier.com/hc/en-us/articles/34372501750285-Use-conditional-logic-to-filter-and-split-your-Zap-workflows)

---

# 1. 사전 준비

## 1-1. Google Form 만들기

먼저 [Google Forms](https://forms.google.com/)에서 폼을 하나 만듭니다.

질문은 예를 들어 아래처럼 구성하면 됩니다.

- 이름
- 이메일
- 점수

여기서 **점수**는 반드시 숫자형으로 받는 것이 좋습니다.  
단답형으로 만들고 응답 검증에서 숫자만 허용하면 나중에 Zapier에서 조건 비교가 훨씬 안정적입니다.

## 1-2. Form 응답을 Google Sheets에 연결

Google Forms의 **Responses** 탭에서 응답을 [Google Sheets](https://sheets.google.com/)로 연결합니다. Zapier의 Google Forms 도움말에도, 실제 활용을 위해서는 **Google Form responses must be saved to a Google Sheet** 라고 안내되어 있습니다. [Source](https://help.zapier.com/hc/en-us/articles/8495997064845-How-to-get-started-with-Google-Forms-on-Zapier)

추천 구조는 이렇게 두는 것입니다.

- 원본 응답 시트: Google Form이 자동 기록
- 결과 기록 시트: Zapier가 PASS/FAIL을 기록

## 1-3. 결과 기록용 시트 만들기

같은 스프레드시트 안에 새 탭을 하나 추가해서 `Processed_Results`라고 이름 붙입니다.

1행에는 다음 헤더를 미리 만듭니다.

- SubmittedAt
- Name
- Email
- Score
- Result
- NotificationChannel
- Note

이렇게 해 두면 Zapier의 `Create Spreadsheet Row`에서 컬럼 매핑이 쉬워집니다. [Source](https://help.zapier.com/hc/en-us/articles/8495994922509-How-to-get-started-with-Google-Sheets-on-Zapier)

## 1-4. Slack 또는 Discord 준비

알림을 어디로 보낼지 정합니다.

- Slack 사용 시: 워크스페이스 연결 필요, 채널 접근 가능해야 함
- Discord 사용 시: 서버 권한과 **Manage Webhooks** 권한 필요

Zapier 도움말 기준으로 Slack은 채널 메시지 전송 액션을 제공하고, Discord도 `Send Channel Message` 액션을 제공합니다. [Source](https://help.zapier.com/hc/en-us/articles/8495993391629-How-to-get-started-with-Slack-on-Zapier) [Source](https://help.zapier.com/hc/en-us/articles/8496048271501-How-to-get-started-with-Discord-on-Zapier)

---

# 2. Zapier에서 앱 연결하기

실제로 Zap을 만들기 전에 앱 연결을 먼저 해두면 편합니다.

## 2-1. Google Forms 연결

Zapier에서  
**Apps**  
→ **+ Add connection**  
→ 검색창에 **Google Forms** 입력  
→ **Google Forms** 선택  
→ **Add connection**  
→ Google 계정 로그인  
→ 권한 승인

Google Forms 앱은 Zapier에서 `New Form Response (Instant)`와 `New or Updated Form Response (Instant)` 트리거를 지원합니다. [Source](https://help.zapier.com/hc/en-us/articles/8495997064845-How-to-get-started-with-Google-Forms-on-Zapier)

## 2-2. Google Sheets 연결

같은 방식으로

**Apps**  
→ **+ Add connection**  
→ **Google Sheets** 검색  
→ **Add connection**  
→ Google 계정 로그인  
→ 권한 승인

Google Sheets는 `New Spreadsheet Row`, `New or Updated Spreadsheet Row`, `Create Spreadsheet Row` 등 다양한 트리거/액션을 지원합니다. [Source](https://help.zapier.com/hc/en-us/articles/8495994922509-How-to-get-started-with-Google-Sheets-on-Zapier)

## 2-3. Slack 연결

Slack 사용 시:

**Apps**  
→ **+ Add connection**  
→ **Slack** 검색  
→ **Add connection**  
→ Slack 로그인  
→ 워크스페이스 선택  
→ 권한 승인

Slack은 `Send Channel Message` 액션을 지원합니다. [Source](https://help.zapier.com/hc/en-us/articles/8495993391629-How-to-get-started-with-Slack-on-Zapier)

## 2-4. Discord 연결

Discord 사용 시:

**Apps**  
→ **+ Add connection**  
→ **Discord** 검색  
→ **Add connection**  
→ Discord 로그인  
→ 권한 승인

Discord는 `Send Channel Message` 액션을 지원하고, 서버 관리 권한 및 웹훅 권한이 필요합니다. [Source](https://help.zapier.com/hc/en-us/articles/8496048271501-How-to-get-started-with-Discord-on-Zapier)

---

# 3. 새 Zap 만들기

이제 실제 자동화를 만듭니다.

Zapier 대시보드에서  
**Create** 또는 **Create Zap** 클릭

그러면 Zap 편집기가 열립니다.

---

# 4. 1단계: Trigger 설정

## 4-1. 앱 선택

Trigger 영역에서  
**Trigger** 클릭  
→ 검색창에 **Google Forms** 입력  
→ **Google Forms** 선택

## 4-2. Trigger event 선택

**Trigger event** 드롭다운에서  
→ **New Form Response** 선택

Zapier 문서상 `New Form Response`는 **Instant** 트리거입니다. 즉, 새 폼 응답이 들어오면 Zap 실행 시작점이 됩니다. [Source](https://help.zapier.com/hc/en-us/articles/8495997064845-How-to-get-started-with-Google-Forms-on-Zapier)

## 4-3. 계정 선택

**Account** 단계에서  
→ 아까 연결한 Google Forms 계정 선택  
→ **Continue**

## 4-4. 폼 선택

**Form** 선택 드롭다운에서  
→ 방금 만든 Google Form 선택  
→ **Continue**

## 4-5. 테스트

**Test trigger** 클릭

Zapier가 최근 제출 응답을 샘플로 불러옵니다.  
이때 응답이 없다면 Google Form에 테스트 응답을 먼저 1건 제출하세요.

예시 테스트 응답:

- 이름: 홍길동
- 이메일: hong@test.com
- 점수: 85

## 4-6. 여기서 확인할 것

샘플 데이터에서 다음을 확인합니다.

- 이름 필드가 제대로 보이는지
- 이메일 필드가 제대로 보이는지
- 점수 필드가 있는지
- 제출 시간 필드가 있는지

이 필드들이 이후 모든 분기와 매핑의 기준이 됩니다.

---

# 5. 2단계: Paths 추가

이제 점수 기준 분기를 만듭니다.

## 5-1. Paths 모듈 추가

트리거 단계 아래의 **+** 클릭  
→ 검색창에 **Paths** 입력  
→ **Paths by Zapier** 선택

그러면 기본적으로 여러 분기용 path 그룹이 생성됩니다.

Zapier 문서 기준으로 Paths 설정의 고수준 흐름은  
**Paths step 추가 → path rule 작성 → 테스트 → 각 path에 액션 추가** 순서입니다. [Source](https://help.zapier.com/hc/en-us/articles/8496288555917-Add-branching-logic-to-Zap-workflows-with-Paths)

## 5-2. 경로 이름 정하기

기본으로 보통 Path A, Path B가 생깁니다.  
이름을 아래처럼 이해하고 작업하면 됩니다.

- Path A = PASS 경로
- Path B = FAIL 경로

---

# 6. Path A 설정: 점수 80 이상

## 6-1. Path A 열기

**Path A** 클릭  
→ **Path rules** 또는 규칙 설정 화면 열기

## 6-2. 규칙 유형 선택

**Custom rules** 선택

Zapier는 Path 규칙으로 Custom rules, Always run, Fallback 등을 지원합니다. 이번 경우는 점수 조건이 필요하므로 **Custom rules**를 선택합니다. [Source](https://help.zapier.com/hc/en-us/articles/34372501750285-Use-conditional-logic-to-filter-and-split-your-Zap-workflows)

## 6-3. 조건 입력

규칙을 다음처럼 설정합니다.

- 첫 번째 박스: 점수 필드 선택
- 두 번째 박스: **(Number) Greater than or equal to**
- 세 번째 박스: `80`

즉, **점수 ≥ 80** 입니다.

## 6-4. 테스트

**Test path** 클릭

현재 샘플 점수가 85라면 이 경로는 통과해야 합니다.  
통과하면 PASS 경로 규칙이 정상입니다.

---

# 7. Path A 안에서 Google Sheets 기록

이제 PASS 경로 안에서 결과를 시트에 적습니다.

## 7-1. 액션 추가

Path A 내부에서 **+** 클릭  
→ **Google Sheets** 검색  
→ **Google Sheets** 선택

## 7-2. Action event 선택

**Action event** 드롭다운에서  
→ **Create Spreadsheet Row** 선택  
→ **Continue**

Zapier의 Google Sheets 앱은 `Create Spreadsheet Row` 액션을 지원합니다. [Source](https://help.zapier.com/hc/en-us/articles/8495994922509-How-to-get-started-with-Google-Sheets-on-Zapier)

## 7-3. 계정 선택

아까 연결한 Google Sheets 계정 선택  
→ **Continue**

## 7-4. 시트 선택

설정 화면에서:

- **Drive**: 해당 Google Drive 선택
- **Spreadsheet**: 결과 기록용 스프레드시트 선택
- **Worksheet**: `Processed_Results` 선택

## 7-5. 컬럼 매핑

이제 각 열에 값을 넣습니다.

- **SubmittedAt** → Google Forms 트리거의 제출 시각 필드 선택
- **Name** → 이름 응답 필드 선택
- **Email** → 이메일 응답 필드 선택
- **Score** → 점수 응답 필드 선택
- **Result** → 직접 `PASS` 입력
- **NotificationChannel** → 직접 `Slack` 또는 `Discord` 입력
- **Note** → 직접 `점수 80 이상` 입력

## 7-6. 테스트

**Test step** 클릭

성공하면 `Processed_Results` 시트에 새 행이 하나 추가됩니다.

---

# 8. Path A 안에서 Slack 알림 보내기

Slack을 쓴다면 다음처럼 합니다.

## 8-1. 액션 추가

Google Sheets 단계 아래 **+** 클릭  
→ **Slack** 검색  
→ **Slack** 선택

## 8-2. Action event 선택

**Action event**에서  
→ **Send Channel Message** 선택  
→ **Continue**

Slack 앱 문서 기준으로 `Send Channel Message`는 채널에 새 메시지를 게시하거나 스레드 답글을 보낼 수 있습니다. [Source](https://help.zapier.com/hc/en-us/articles/8495993391629-How-to-get-started-with-Slack-on-Zapier)

## 8-3. 계정 선택

Slack 계정 선택  
→ **Continue**

## 8-4. 채널 선택

**Channel** 드롭다운에서  
→ 메시지를 보낼 채널 선택

## 8-5. 메시지 본문 작성

메시지 박스에 다음처럼 작성합니다.

`✅ 합격 처리 완료 이름: [이름 필드] 이메일: [이메일 필드] 점수: [점수 필드] 결과: PASS`

여기서 `[이름 필드]` 같은 부분은 직접 타이핑이 아니라, 오른쪽 데이터 패널에서 필드를 클릭해 넣습니다.

## 8-6. 테스트

**Test step** 클릭

성공하면 Slack 채널에 메시지가 도착합니다.

## 8-7. Slack 오류 주의

만약 `channel_not_found` 오류가 뜨면, Zapier 도움말 기준으로 보통 다음 중 하나입니다.

- 연결된 Slack 계정이 해당 채널 멤버가 아님
- 잘못된 워크스페이스 계정을 선택함
- 채널 값을 잘못 넣음
- 채널명이 바뀜

즉, 가장 먼저 **연결 계정이 그 채널에 접근 가능한지** 확인하면 됩니다. [Source](https://help.zapier.com/hc/en-us/articles/31661561373837-Slack-error-channel-not-found)

---

# 9. Path A 안에서 Discord 알림 보내기

Slack 대신 Discord를 원하면 아래처럼 바꾸면 됩니다.

## 9-1. 액션 추가

Google Sheets 단계 아래 **+** 클릭  
→ **Discord** 검색  
→ **Discord** 선택

## 9-2. Action event 선택

**Action event**에서  
→ **Send Channel Message** 선택  
→ **Continue**

Zapier 문서상 Discord는 `Send Channel Message` 액션을 제공합니다. [Source](https://help.zapier.com/hc/en-us/articles/8496048271501-How-to-get-started-with-Discord-on-Zapier)

## 9-3. 계정 선택

Discord 계정 선택  
→ **Continue**

## 9-4. 서버/채널 선택

설정 화면에서

- 서버 선택
- 텍스트 채널 선택

## 9-5. 메시지 작성

예시:

`✅ 합격 처리 완료 이름: [이름 필드] 이메일: [이메일 필드] 점수: [점수 필드] 결과: PASS`

## 9-6. 테스트

**Test step** 클릭

성공하면 Discord 채널에 메시지가 올라옵니다.

Discord 쪽은 사전 권한으로 **Manage Webhooks**가 필요합니다. [Source](https://help.zapier.com/hc/en-us/articles/8496048271501-How-to-get-started-with-Discord-on-Zapier)

---

# 10. Path B 설정: 점수 80 미만

이제 FAIL 경로를 만듭니다.

## 10-1. Path B 열기

**Path B** 클릭  
→ **Path rules** 열기

## 10-2. 규칙 설정

**Custom rules** 선택 후 다음처럼 설정합니다.

- 첫 번째 박스: 점수 필드
- 두 번째 박스: **(Number) Less than**
- 세 번째 박스: `80`

즉, **점수 < 80** 입니다.

## 10-3. 테스트

테스트 샘플이 85점이면 이 경로는 통과하지 않는 것이 정상입니다.  
이 경우엔 65점짜리 새 응답을 만들어 샘플을 바꾸거나, 나중에 실제 실행에서 검증하면 됩니다.

---

# 11. Path B 안에서 Google Sheets 기록

PASS 경로와 동일하게 Google Sheets 액션을 추가합니다.

## 11-1. 액션 추가

Path B 내부에서 **+** 클릭  
→ **Google Sheets**  
→ **Create Spreadsheet Row**

## 11-2. 시트 선택

- **Spreadsheet**: 결과 기록용 시트
- **Worksheet**: `Processed_Results`

## 11-3. 컬럼 매핑

- **SubmittedAt** → 제출 시각
- **Name** → 이름
- **Email** → 이메일
- **Score** → 점수
- **Result** → 직접 `FAIL`
- **NotificationChannel** → `Slack` 또는 `Discord`
- **Note** → `점수 80 미만`

## 11-4. 테스트

**Test step** 클릭

성공하면 결과 시트에 FAIL 행이 추가됩니다.

---

# 12. Path B 안에서 Slack 또는 Discord 알림 보내기

## Slack 사용 시

Path B의 Google Sheets 단계 아래 **+**  
→ **Slack**  
→ **Send Channel Message**

메시지 예시:

`❌ 불합격 처리 완료 이름: [이름 필드] 이메일: [이메일 필드] 점수: [점수 필드] 결과: FAIL`

## Discord 사용 시

Path B의 Google Sheets 단계 아래 **+**  
→ **Discord**  
→ **Send Channel Message**

메시지 예시:

`❌ 불합격 처리 완료 이름: [이름 필드] 이메일: [이메일 필드] 점수: [점수 필드] 결과: FAIL`

---

# 13. 전체 테스트 순서

이제 Zap을 전체적으로 검증합니다.

## 테스트 1: 합격 케이스

Google Form에 다음 응답 제출:

- 이름: 김합격
- 이메일: pass@test.com
- 점수: 90

예상 결과:

- Path A 실행
- `Processed_Results`에 PASS 기록
- Slack/Discord에 합격 메시지 발송

## 테스트 2: 불합격 케이스

Google Form에 다음 응답 제출:

- 이름: 이불합격
- 이메일: fail@test.com
- 점수: 65

예상 결과:

- Path B 실행
- `Processed_Results`에 FAIL 기록
- Slack/Discord에 불합격 메시지 발송

이 두 테스트가 모두 성공하면 과제 요건인 **트리거 + 조건 분기 + 2개 이상 액션**을 충족합니다.

---

# 14. Zap 켜기

모든 테스트가 끝나면 Zap 편집기 상단에서 **Publish** 또는 **Turn on Zap** 을 눌러 활성화합니다.

이후부터는 Google Form에 새 응답이 들어올 때마다 Zap이 실행됩니다. Google Forms 트리거는 `New Form Response (Instant)`이므로, Make에서 Google Sheets를 주기 감시하는 방식보다 더 직접적인 트리거 흐름을 만들 수 있습니다. [Source](https://help.zapier.com/hc/en-us/articles/8495997064845-How-to-get-started-with-Google-Forms-on-Zapier)

---

# 15. 자주 막히는 문제

## 점수 비교가 이상한 경우

폼의 점수 값이 숫자가 아니라 텍스트처럼 들어오면 조건 비교가 꼬일 수 있습니다.  
Google Form에서 숫자 입력만 허용하는 방식이 가장 안전합니다.

## Slack 메시지가 안 가는 경우

`channel_not_found`가 가장 흔합니다.  
연결 계정, 채널 멤버십, 채널 선택값을 확인하세요. [Source](https://help.zapier.com/hc/en-us/articles/31661561373837-Slack-error-channel-not-found)

## Discord 메시지가 안 가는 경우

서버 권한이나 **Manage Webhooks** 권한 부족일 가능성이 큽니다. [Source](https://help.zapier.com/hc/en-us/articles/8496048271501-How-to-get-started-with-Discord-on-Zapier)

## 무료 플랜에서 안 되는 경우

Paths와 여러 단계가 필요한 구조라면 무료 플랜 한계에 걸릴 수 있습니다.  
Zapier 가격 페이지 기준으로 무료 플랜은 **two-step Zap workflows** 중심입니다. [Source](https://zapier.com/pricing)

---

# 16. 초압축 클릭 순서 요약

아주 짧게 요약하면 아래 순서입니다.

**Zapier → Create Zap**  
→ **Trigger: Google Forms**  
→ **New Form Response**  
→ 폼 선택  
→ 테스트 [Source](https://help.zapier.com/hc/en-us/articles/8495997064845-How-to-get-started-with-Google-Forms-on-Zapier)

→ **+ Add step**  
→ **Paths by Zapier**  
→ Path A: `Score >= 80`  
→ Path B: `Score < 80` [Source](https://help.zapier.com/hc/en-us/articles/8496288555917-Add-branching-logic-to-Zap-workflows-with-Paths)

→ **Path A 내부**

- **Google Sheets → Create Spreadsheet Row**
- **Slack/Discord → Send Channel Message** [Source](https://help.zapier.com/hc/en-us/articles/8495994922509-How-to-get-started-with-Google-Sheets-on-Zapier) [Source](https://help.zapier.com/hc/en-us/articles/8495993391629-How-to-get-started-with-Slack-on-Zapier) [Source](https://help.zapier.com/hc/en-us/articles/8496048271501-How-to-get-started-with-Discord-on-Zapier)

→ **Path B 내부**

- **Google Sheets → Create Spreadsheet Row**
- **Slack/Discord → Send Channel Message**

→ 테스트 2회  
→ **Publish / Turn on Zap**

---

원하시면 다음 답변에서 바로 이어서  
**“보고서에 그대로 붙여넣을 수 있는 Zapier 구현 절차 문장형 설명”**  
형태로 정리해드릴게요.

---
 아래는 **보고서에 그대로 붙여넣을 수 있는 Zapier 구현 절차 문장형 설명**입니다.  
과제 보고서 문체에 맞춰 **설계 목적 → 구현 절차 → 분기 설정 → 테스트 및 결과 → 유의사항** 흐름으로 정리했습니다.

---

# [Zapier](https://zapier.com/pricing)를 활용한 Google Form 응답 자동화 구현 절차

본 프로젝트에서는 [Google Forms](https://forms.google.com/)를 통해 수집된 응답 데이터를 기반으로, 점수 조건에 따라 처리 경로를 분기하고, 이를 [Google Sheets](https://sheets.google.com/)에 기록한 뒤 [Slack](https://help.zapier.com/hc/en-us/articles/8495993391629-How-to-get-started-with-Slack-on-Zapier) 또는 [Discord](https://help.zapier.com/hc/en-us/articles/8496048271501-How-to-get-started-with-Discord-on-Zapier)로 알림을 전송하는 자동화 워크플로우를 [Zapier](https://zapier.com/pricing)에서 구현하였다. 전체 구조는 “Google Form 제출 → Zapier 트리거 실행 → 점수 조건에 따른 경로 분기 → Google Sheets 결과 기록 → 메신저 알림 전송”의 흐름으로 설계하였다. Zapier의 Google Forms 앱은 `New Form Response (Instant)` 트리거를 지원하며, Paths 기능을 통해 하나의 입력 데이터를 조건별로 서로 다른 액션 경로로 분기할 수 있다. [Source](https://help.zapier.com/hc/en-us/articles/8495997064845-How-to-get-started-with-Google-Forms-on-Zapier) [Source](https://help.zapier.com/hc/en-us/articles/8496288555917-Add-branching-logic-to-Zap-workflows-with-Paths)

자동화를 구현하기에 앞서 먼저 Google Form을 생성하고, 입력 항목을 이름, 이메일, 점수로 구성하였다. 이때 점수 항목은 이후 조건 분기에 사용되므로 숫자값으로 입력되도록 설정하였다. 또한 Google Form 응답은 별도의 Google Sheets 스프레드시트에 자동 저장되도록 연결하였다. Zapier의 Google Forms 안내 문서에서도 실제 활용을 위해 폼 응답이 Google Sheets에 저장된 상태로 관리되는 구조를 전제로 설명하고 있으며, 이를 통해 응답 데이터의 구조를 명확히 유지하고 후속 분석이나 검토에도 활용할 수 있다. [Source](https://help.zapier.com/hc/en-us/articles/8495997064845-How-to-get-started-with-Google-Forms-on-Zapier)

그다음으로 자동화 결과를 저장할 별도의 결과 시트를 준비하였다. 이 시트에는 제출 시각, 이름, 이메일, 점수, 판정 결과, 알림 채널, 비고 등의 컬럼을 생성하였다. 원본 응답 시트와 결과 기록 시트를 분리한 이유는, 입력 데이터와 자동화 처리 결과를 구분하여 관리하기 위함이며, 동시에 자동화 흐름의 입력과 출력이 혼재하지 않도록 하기 위함이다. Zapier의 Google Sheets 앱은 `Create Spreadsheet Row` 액션을 제공하므로, 자동화 결과를 행 단위로 누적 기록하는 데 적합하다. [Source](https://help.zapier.com/hc/en-us/articles/8495994922509-How-to-get-started-with-Google-Sheets-on-Zapier)

이후 Zapier에 로그인한 뒤 새 Zap을 생성하였다. 첫 단계에서는 Trigger 앱으로 Google Forms를 선택하고, Trigger Event로 `New Form Response`를 설정하였다. 이 트리거는 새로운 폼 응답이 제출되는 즉시 Zap 실행을 시작하는 역할을 한다. 계정 연결 단계에서는 Google 계정을 인증하고, 대상이 되는 Google Form을 선택한 뒤 테스트를 수행하였다. 테스트 과정에서는 최근에 제출된 응답 데이터를 불러와 이름, 이메일, 점수 등의 필드가 정상적으로 인식되는지 확인하였다. 이 단계는 이후 조건 분기와 데이터 매핑의 기준이 되는 입력 구조를 검증하는 과정으로 중요하다. [Source](https://help.zapier.com/hc/en-us/articles/8495997064845-How-to-get-started-with-Google-Forms-on-Zapier)

트리거 설정이 완료된 이후에는 조건 분기를 위해 `Paths by Zapier` 단계를 추가하였다. Zapier의 Paths 기능은 하나의 Zap 안에서 서로 다른 규칙을 기준으로 여러 경로를 구성할 수 있도록 해 주며, “If A happens, do X / If B happens, do Y”와 같은 조건 분기 로직을 구현하는 데 사용된다. 본 프로젝트에서는 두 개의 경로를 생성하였다. 첫 번째 경로는 점수가 80점 이상인 경우를 처리하는 합격 경로로 설정하였고, 두 번째 경로는 점수가 80점 미만인 경우를 처리하는 불합격 경로로 설정하였다. 각 경로에서는 `Custom rules`를 사용하여 조건을 작성하였다. 합격 경로에는 `Score >= 80`, 불합격 경로에는 `Score < 80` 조건을 설정하였다. 이로써 하나의 입력 데이터가 점수값에 따라 서로 다른 후속 액션으로 분기되도록 설계하였다. Zapier 문서에 따르면 Paths는 분기형 워크플로우에 적합하며, 경로는 왼쪽에서 오른쪽 순서로 순차적으로 평가 및 실행된다. [Source](https://help.zapier.com/hc/en-us/articles/8496288555917-Add-branching-logic-to-Zap-workflows-with-Paths) [Source](https://help.zapier.com/hc/en-us/articles/34372501750285-Use-conditional-logic-to-filter-and-split-your-Zap-workflows)

합격 경로에는 먼저 Google Sheets의 `Create Spreadsheet Row` 액션을 추가하여 결과 기록 시트에 합격 데이터를 저장하도록 구성하였다. 이 단계에서는 Google Form 응답에서 전달된 제출 시각, 이름, 이메일, 점수 값을 각각 결과 시트의 대응 컬럼에 매핑하였고, 결과 컬럼에는 `PASS`, 알림 채널 컬럼에는 `Slack` 또는 `Discord`, 비고 컬럼에는 `점수 80 이상`이라는 고정값을 입력하였다. 이를 통해 자동화 실행 결과가 구조화된 형태로 누적 기록되도록 하였다. Zapier의 Google Sheets 액션은 새로운 행을 생성하는 방식으로 데이터를 저장하므로, 이후 결과 확인 및 검토에 적합하다. [Source](https://help.zapier.com/hc/en-us/articles/8495994922509-How-to-get-started-with-Google-Sheets-on-Zapier)

합격 경로의 두 번째 액션으로는 Slack 또는 Discord에 메시지를 전송하는 단계를 추가하였다. Slack을 사용할 경우 `Send Channel Message` 액션을 선택하여 지정한 채널에 메시지를 발송하도록 설정하였고, 메시지 내용에는 이름, 이메일, 점수, 결과를 포함하여 담당자가 즉시 상황을 파악할 수 있도록 구성하였다. 예를 들어 “합격 처리 완료, 이름: ○○○, 이메일: ○○○, 점수: ○○점, 결과: PASS”와 같은 형식으로 메시지를 작성하였다. Slack은 Zapier를 통해 채널 메시지를 자동 전송할 수 있지만, 연결된 Slack 계정이 해당 채널에 접근 가능해야 하며, 잘못된 채널 또는 연결 계정을 사용할 경우 `channel_not_found` 오류가 발생할 수 있다. 따라서 메시지 전송 전에 올바른 워크스페이스와 채널 권한을 확인하였다. [Source](https://help.zapier.com/hc/en-us/articles/8495993391629-How-to-get-started-with-Slack-on-Zapier) [Source](https://help.zapier.com/hc/en-us/articles/31661561373837-Slack-error-channel-not-found)

불합격 경로 역시 동일한 방식으로 구성하였다. 먼저 Google Sheets의 `Create Spreadsheet Row` 액션을 추가하여 결과 시트에 불합격 데이터를 저장하였으며, 이때 결과 컬럼에는 `FAIL`, 비고 컬럼에는 `점수 80 미만`을 입력하였다. 이후 Slack 또는 Discord의 `Send Channel Message` 액션을 통해 불합격 알림 메시지를 전송하도록 하였다. 메시지 구성은 합격 경로와 동일하되, 최종 결과값을 `FAIL`로 변경하여 불합격자에 대한 후속 조치가 가능하도록 하였다. 이와 같이 각각의 경로가 동일한 입력 구조를 사용하면서도 서로 다른 처리 결과를 생성하도록 구성함으로써, 조건 분기 자동화가 실제 업무 흐름에 어떻게 적용될 수 있는지를 구현하였다. [Source](https://help.zapier.com/hc/en-us/articles/8495994922509-How-to-get-started-with-Google-Sheets-on-Zapier) [Source](https://help.zapier.com/hc/en-us/articles/8495993391629-How-to-get-started-with-Slack-on-Zapier) [Source](https://help.zapier.com/hc/en-us/articles/8496048271501-How-to-get-started-with-Discord-on-Zapier)

구현 완료 후에는 실제 폼 응답 데이터를 사용하여 분기 동작을 검증하였다. 첫 번째 테스트에서는 점수를 90점으로 입력한 응답을 제출하여 합격 경로가 정상적으로 실행되는지 확인하였다. 그 결과 Zapier가 Google Form 응답을 즉시 감지하고, 합격 경로를 통해 결과 시트에 `PASS`를 기록한 뒤 Slack 또는 Discord로 합격 알림을 전송하는 것을 확인하였다. 두 번째 테스트에서는 점수를 65점으로 입력한 응답을 제출하여 불합격 경로를 검증하였다. 이 경우에는 결과 시트에 `FAIL`이 기록되고, 불합격 메시지가 자동으로 전송되는 것을 확인하였다. 이를 통해 동일한 입력 구조가 점수값에 따라 두 개의 상이한 액션 경로로 분기된다는 점을 검증하였다. [Source](https://help.zapier.com/hc/en-us/articles/8496288555917-Add-branching-logic-to-Zap-workflows-with-Paths)

본 자동화에서 사용한 조건 분기 방식은 Filter가 아니라 Paths를 선택한 것이 핵심적인 설계 요소였다. Zapier의 조건 로직 문서에 따르면 Filter는 하나의 조건을 기준으로 Zap의 이후 단계를 계속 실행할지 중단할지를 결정하는 선형 구조에 적합한 반면, Paths는 여러 조건에 따라 서로 다른 후속 작업을 수행하는 분기형 구조에 적합하다. 이번 프로젝트에서는 합격과 불합격 모두에서 각각 별도의 시트 기록과 메시지 전송이 필요하였으므로, 단일 선형 흐름보다는 분기형 구조가 요구되었고 이에 따라 Paths를 활용하였다. [Source](https://help.zapier.com/hc/en-us/articles/34372501750285-Use-conditional-logic-to-filter-and-split-your-Zap-workflows)

마지막으로 이 구현은 Zapier의 플랜 제약도 함께 고려해야 한다. Zapier 가격 안내에 따르면 Free 플랜은 월 100 tasks와 two-step Zap workflows를 포함하는 수준이므로, Trigger 이후 Paths와 여러 액션 단계가 이어지는 이번 구조는 일반적으로 Professional 이상의 환경에서 구현하는 것이 적합하다. 따라서 실제 과제 수행에서는 기능 사용 가능 여부와 단계 수 제한을 함께 확인해야 하며, 이러한 플랫폼 제약은 도구 선택 시 고려해야 할 중요한 요소로 볼 수 있다. [Source](https://zapier.com/pricing)

종합하면, 본 구현은 Google Forms의 즉시 트리거, Paths를 이용한 조건 분기, Google Sheets의 결과 기록, Slack 또는 Discord의 자동 알림 전송을 하나의 Zap 안에 통합한 사례이다. 이를 통해 사용자가 제출한 폼 응답을 자동으로 판별하고, 결과를 체계적으로 저장하며, 담당자에게 즉시 전달하는 노코드 자동화 워크플로우를 설계하고 구현할 수 있었다. 또한 Trigger, Action, 조건 분기 요소를 모두 포함하고 있어 과제의 필수 요구사항을 충족하는 실습 사례로 활용할 수 있다. [Source](https://help.zapier.com/hc/en-us/articles/8495997064845-How-to-get-started-with-Google-Forms-on-Zapier) [Source](https://help.zapier.com/hc/en-us/articles/8496288555917-Add-branching-logic-to-Zap-workflows-with-Paths) [Source](https://help.zapier.com/hc/en-us/articles/8495994922509-How-to-get-started-with-Google-Sheets-on-Zapier)