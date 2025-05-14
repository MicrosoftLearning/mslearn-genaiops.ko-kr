---
lab:
  title: 추적을 사용하여 생성형 AI 앱 분석 및 디버그
---

# 추적을 사용하여 생성형 AI 앱 분석 및 디버그

이 연습에는 약 **30**분이 소요됩니다.

> **참고**: 이 연습에서는 Azure AI 파운드리에 대해 잘 알고 있다고 가정하므로 일부 지침은 보다 적극적인 탐색 및 실습 학습을 장려하기 위해 의도적으로 상세히 설명되어 있지 않습니다.

## 소개

이 연습에서는 하이킹 여행을 추천하고 야외 장비를 제안하는 다단계 생성형 AI 도우미를 실행합니다. Azure AI 유추 SDK의 추적 기능을 사용하여 애플리케이션이 실행되는 방식을 분석하고 모델 및 주변 논리에 의해 결정된 주요 결정 사항을 식별합니다.

배포된 모델과 상호 작용하여 실제 사용자 경험을 시뮬레이션하고, 사용자 입력에서 모델 응답, 후처리에 이르는 애플리케이션의 각 단계를 추적하고, Azure AI 파운드리에서 추적 데이터를 확인합니다. 이는 추적이 어떻게 가시성을 향상시키고 디버깅을 간소화하며 생성형 AI 애플리케이션의 성능 최적화를 지원하는지 이해하는 데 도움이 됩니다.

## 환경 설정

이 연습의 작업을 완료하려면 다음이 필요합니다.

- Azure AI 파운드리 허브,
- Azure AI 파운드리 프로젝트,
- 배포된 모델(예: GPT-4o),
- 연결된 Application Insights 리소스.

### AI 파운드리 허브 및 프로젝트 만들기

허브와 프로젝트를 빠르게 설정하기 위해 Azure AI 파운드리 포털 UI를 사용하는 간단한 지침이 아래에 제공됩니다.

1. Azure AI 파운드리 포털로 이동하여 [https://ai.azure.com](https://ai.azure.com)을(를) 엽니다
1. Azure 자격 증명을 사용하여 로그인합니다.
1. 프로젝트 만들기:
    1. **모든 허브 + 프로젝트**로 이동합니다.
    1. **+ 새 프로젝트**를 선택합니다.
    1. **프로젝트 이름**을 입력합니다.
    1. 메시지가 표시되면 **새 허브를 만듭니다**.
    1. 허브 사용자 지정:
        1. **구독**, **리소스 그룹**, **위치** 등을 선택합니다.
        1. **새 Azure AI 서비스** 리소스를 연결합니다(AI 검색 건너뛰기).
    1. 검토하고 **만들기**를 선택합니다.
1. **배포가 완료될 때까지 기다립니다**(~1~2분).

### 모델 배포

모니터링할 수 있는 데이터를 생성하려면 먼저 모델을 배포하고 상호 작용해야 합니다. 지침에서는 GPT-4o 모델을 배포하라는 메시지가 표시되지만 사용 가능한 Azure OpenAI Service 컬렉션의 **모든 모델을 사용할 수 있습니다**.

1. 왼쪽 메뉴에서 **내 자산**에서 **모델 + 엔드포인트** 페이지를 선택합니다.
1. **베이스 모델**을 배포하고 **gpt-4o**를 선택합니다.
1. **배포 세부 정보를 사용자 지정합니다**.
1. **용량**을 **분당 5k 토큰(TPM)** 으로 설정합니다.

허브 및 프로젝트가 준비되었으며 필요한 모든 Azure 리소스가 자동으로 프로비전됩니다.

### Application Insights 연결

Application Insights를 Azure AI 파운드리의 프로젝트에 연결하여 분석을 위한 데이터 수집을 시작합니다.

1. Azure AI 파운드리 포털에서 프로젝트 열기
1. 왼쪽 메뉴에서 **추적** 페이지를 선택합니다.
1. **앱에 연결할 새** Application Insights 리소스를 만듭니다.
1. **Application Insights 리소스 이름**을 입력합니다.

이제 Application Insights가 프로젝트에 연결되었으며 분석을 위해 데이터가 수집되기 시작합니다.

## Cloud Shell을 사용하여 생성형 AI 앱 실행

Azure Cloud Shell에서 Azure AI 파운드리 프로젝트에 연결하고 생성형 AI 애플리케이션의 일부로 배포된 모델과 프로그래밍 방식으로 상호 작용합니다.

### 배포된 모델과 상호 작용

먼저 배포된 모델과 상호 작용하기 위해 인증하는 데 필요한 정보를 검색합니다. 그런 다음, Azure Cloud Shell에 액세스하고 생성형 AI 앱의 코드를 업데이트합니다.

1. Azure AI 파운드리 포털에서 프로젝트의 **개요** 페이지를 봅니다.
1. **프로젝트 세부 정보** 영역에서 **프로젝트 연결 문자열**을 확인합니다.
1. 메모장에 문자열을 **저장**합니다. 이 연결 문자열 사용하여 클라이언트 응용 프로그램에서 프로젝트에 연결합니다.
1. 새 브라우저 탭을 엽니다(Azure AI 파운드리 포털을 기존 탭에서 열어 두기).
1. 새 탭에서 [Azure Portal](https://portal.azure.com)(`https://portal.azure.com`)로 이동한 다음 메시지가 나타나면 Azure 자격 증명을 사용하여 로그인합니다.
1. 페이지 상단의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 구독에 저장소가 없는 ***PowerShell*** 환경을 선택합니다.
1. Cloud Shell 도구 모음의 **설정** 메뉴에서 **클래식 버전으로 이동**을 선택합니다.

    **<font color="red">계속하기 전에 Cloud Shell의 클래식 버전으로 전환했는지 확인합니다.</font>**

1. Cloud Shell 창에서 다음 명령을 입력하여 실행합니다.

    ```
    rm -r mslearn-genaiops -f
    git clone https://github.com/microsoftlearning/mslearn-genaiops mslearn-genaiops
    ```

    이 명령은 이 연습의 코드 파일이 포함된 GitHub 리포지토리를 복제합니다.

1. 리포지토리가 복제된 후 애플리케이션 코드 파일이 포함된 폴더로 이동합니다.  

    ```
   cd mslearn-genaiops/Files/08
    ```

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 필요한 라이브러리를 설치합니다.

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference azure-monitor-opentelemetry
    ```

1. 제공된 구성 파일을 열려면 다음 명령을 입력합니다.

    ```
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. 코드 파일에서:

    1. **your_project_connection_string** 자리 표시자를 프로젝트의 연결 문자열로 바꿉니다(Azure AI 파운드리 포털의 프로젝트 **개요** 페이지에서 복사함).
    1. **your_model_deployment** 자리 표시자를 GPT-4o 모델 배포에 할당한 이름(기본값으로 `gpt-4o`)으로 바꿉니다.

1. *자리 표시자를 바꾼 후* 코드 편집기에서 **CTRL+S** 명령을 사용하거나 **마우스 오른쪽 단추를 클릭하고 저장**하여 **변경 내용을 저장**합니다.

### 생성형 AI 앱에 대한 코드 업데이트

이제 환경이 설정되고 .env 파일이 구성되었으므로 실행할 AI 도우미 스크립트를 준비할 차례입니다. AI 프로젝트에 연결하고 Application Insights를 사용하도록 설정하려면 다음을 수행해야 합니다.

- 배포된 모델과 상호 작용합니다.
- 프롬프트를 지정하는 함수를 정의합니다.
- 모든 함수를 호출하는 기본 흐름을 정의합니다.

이 세 부분을 시작 스크립트에 추가합니다.

1. 제공된 **스크립트를 열려면** 다음 명령을 실행합니다.

    ```
   code start-prompt.py
    ```

    여러 키 줄이 비어 있거나 빈 # 주석으로 표시된 것을 볼 수 있습니다. 아래의 올바른 줄을 복사하여 적절한 위치에 붙여넣어 스크립트를 완료해야 합니다.

1. 스크립트에서 **# 모델을 호출하고 추적을 처리하는 함수**를 찾습니다.
1. 이 주석 아래에 다음 코드를 붙여넣습니다.

    ```
   def call_model(system_prompt, user_prompt, span_name):
        with tracer.start_as_current_span(span_name) as span:
            span.set_attribute("session.id", SESSION_ID)
            span.set_attribute("prompt.user", user_prompt)
            start_time = time.time()
    
            response = chat_client.complete(
                model=model_name,
                messages=[SystemMessage(system_prompt), UserMessage(user_prompt)]
            )
    
            duration = time.time() - start_time
            output = response.choices[0].message.content
            span.set_attribute("response.time", duration)
            span.set_attribute("response.tokens", len(output.split()))
            return output
    ```

1. 스크립트에서 **# 사용자 선호도에 따라 하이킹을 추천하는 함수**를 찾습니다.
1. 이 주석 아래에 다음 코드를 붙여넣습니다.

    ```
   def recommend_hike(preferences):
        with tracer.start_as_current_span("recommend_hike") as span:
            prompt = f"""
            Recommend a named hiking trail based on the following user preferences.
            Provide only the name of the trail and a one-sentence summary.
            Preferences: {preferences}
            """
            response = call_model(
                "You are an expert hiking trail recommender.",
                prompt,
                "recommend_model_call"
            )
            span.set_attribute("hike_recommendation", response.strip())
            return response.strip()
    ```

1. 스크립트에서 **# ---- Main Flow ----** 를 찾습니다.
1. 이 주석 아래에 다음 코드를 붙여넣습니다.

    ```
   if __name__ == "__main__":
       with tracer.start_as_current_span("trail_guide_session") as session_span:
           session_span.set_attribute("session.id", SESSION_ID)
           print("\n--- Trail Guide AI Assistant ---")
           preferences = input("Tell me what kind of hike you're looking for (location, difficulty, scenery):\n> ")

           hike = recommend_hike(preferences)
           print(f"\n✅ Recommended Hike: {hike}")

           # Run profile function


           # Run match product function


           print(f"\n🔍 Trace ID available in Application Insights for session: {SESSION_ID}")
    ```

1. 스크립트에서 **변경한 내용을 저장**합니다.
1. 코드 편집기 아래의 Cloud Shell 명령줄 창에서 다음 명령을 입력하여 **스크립트를 실행**합니다.

    ```
   python start-prompt.py
    ```

1. 예를 들면 다음과 같이 찾고 있는 하이킹 종류에 대해 설명합니다.

    ```
   A one-day hike in the mountains
    ```

    모델은 Application Insights로 캡처되는 응답을 생성합니다. **Azure AI 파운드리 포털**에서 추적을 시각화할 수 있습니다.

> **참고**: 모니터링 데이터가 Azure Monitor에 표시되는 데 몇 분 정도 걸릴 수 있습니다.

## Azure AI 파운드리 포털에서 추적 데이터 보기

스크립트를 실행한 후 AI 애플리케이션의 실행 추적을 캡처했습니다. 이제 Azure AI 파운드리에서 Application Insights를 사용하여 탐색합니다.

> **참고:** 나중에 코드를 다시 실행하고 Azure AI 파운드리 포털에서 추적을 다시 확인합니다. 먼저 이를 시각화하기 위해 추적을 찾을 수 있는 위치를 살펴보겠습니다.

### Azure AI 파운드리 포털로 이동합니다.

1. **Cloud Shell을 계속 열어 두세요!** 여기로 돌아와 코드를 업데이트하고 다시 실행합니다.
1. **Azure AI 파운드리 포털**이 열려 있는 브라우저의 탭으로 이동합니다.
1. 왼쪽 메뉴에서 **추적**을 선택합니다.
1. *데이터가 표시되지 않으면* 보기를 **새로고침**합니다.
1. 추적 **train_guide_session**을 선택하여 자세한 내용을 보여 주는 새 창을 엽니다.

### 추적을 검토합니다.

이 보기에는 트레일 가이드 AI 도우미의 한 전체 세션에 대한 추적이 표시됩니다.

- **최상위 범위**: trail_guide_session 이것은 상위 범위입니다. 이는 도우미의 전체 실행을 처음부터 끝까지 나타냅니다.

- **중첩된 하위 범위**: 들여쓰기된 각 줄은 중첩된 작업을 나타냅니다. 다음을 찾을 수 있습니다.

    - **recommend_hike**는 하이킹을 결정하는 논리를 캡처합니다.
    - **recommend_model_call**은 recommend_hike 내부의 call_model()에 의해 만들어진 범위입니다.
    - **chat gpt-4o**는 실제 LLM 상호 작용을 표시하기 위해 Azure AI 유추 SDK에 의해 자동으로 계측됩니다.

1. 원하는 범위를 클릭하면 다음을 볼 수 있습니다.

    1. 해당 기간입니다.
    1. 사용자 프롬프트, 사용된 토큰, 응답 시간과 같은 특성입니다.
    1. **span.set_attribute(...)** 와 연결된 모든 오류 또는 사용자 지정 데이터입니다.

## 코드에 더 많은 함수 추가


1. 다음 명령을 실행하여 **스크립트를 다시 엽니다.**

    ```
   code start-prompt.py
    ```

1. 스크립트에서 **# 추천 하이킹에 대한 여정 프로필을 생성하는 함수**를 찾습니다.
1. 이 주석 아래에 다음 코드를 붙여넣습니다.

    ```
   def generate_trip_profile(hike_name):
       with tracer.start_as_current_span("trip_profile_generation") as span:
           prompt = f"""
           Hike: {hike_name}
           Respond ONLY with a valid JSON object and nothing else.
           Do not include any intro text, commentary, or markdown formatting.
           Format: {{ "trailType": ..., "typicalWeather": ..., "recommendedGear": [ ... ] }}
           """
           response = call_model(
               "You are an AI assistant that returns structured hiking trip data in JSON format.",
               prompt,
               "trip_profile_model_call"
           )
           print("🔍 Raw model response:", response)
           try:
               profile = json.loads(response)
               span.set_attribute("profile.success", True)
               return profile
           except json.JSONDecodeError as e:
               print("❌ JSON decode error:", e)
               span.set_attribute("profile.success", False)
               return {}
    ```

1. 스크립트에서 **# 추천 장비를 카탈로그에 있는 제품과 매칭하는 함수**를 찾습니다.
1. 이 주석 아래에 다음 코드를 붙여넣습니다.

    ```
   def match_products(recommended_gear):
       with tracer.start_as_current_span("product_matching") as span:
           matched = []
           for gear_item in recommended_gear:
               for product in mock_product_catalog:
                   if any(word in product.lower() for word in gear_item.lower().split()):
                       matched.append(product)
                       break
           span.set_attribute("matched.count", len(matched))
           return matched
    ```

1. 스크립트에서 **# 프로필 함수 실행**을 찾습니다.
1. 이 주석 아래에 **맞추어** 다음 코드를 붙여넣습니다.

    ```
           profile = generate_trip_profile(hike)
           if not profile:
           print("Failed to generate trip profile. Please check Application Insights for trace.")
           exit(1)

           print(f"\n📋 Trip Profile for {hike}:")
           print(json.dumps(profile, indent=2))
    ```

1. 스크립트에서 **# 매칭 제품 함수 실행**을 찾습니다.
1. 이 주석 아래에 **맞추어** 다음 코드를 붙여넣습니다.

    ```
           matched = match_products(profile.get("recommendedGear", []))
           print("\n🛒 Recommended Products from Lakeshore Retail:")
           print("\n".join(matched))
    ```

1. 스크립트에서 **변경한 내용을 저장**합니다.
1. 코드 편집기 아래의 Cloud Shell 명령줄 창에서 다음 명령을 입력하여 **스크립트를 실행**합니다.

    ```
   python start-prompt.py
    ```

1. 예를 들면 다음과 같이 찾고 있는 하이킹 종류에 대해 설명합니다.

    ```
   I want to go for a multi-day adventure along the beach
    ```

> **참고**: 모니터링 데이터가 Azure Monitor에 표시되는 데 몇 분 정도 걸릴 수 있습니다.

### Azure AI 파운드리 포털에서 새 추적 보기

1. Azure AI 파운드리 포털로 다시 이동합니다.
1. 동일한 이름의 **trail_guide_session**을 가진 새 추적이 나타납니다. 필요한 경우 보기를 새로 고칩니다.
1. 새 추적을 선택하여 더 자세한 보기를 엽니다.
1. 새로 중첩된 하위 범위 **trip_profile_ Generation** 및 **product_matching**을 검토합니다.
1. **product_matching**을 선택하고 표시되는 메타데이터를 검토합니다.

    product_matching 함수에 **span.set_attribute("matched.count", len(matched))** 를 포함했습니다. 키-값 쌍 **matched.count**와 일치하는 변수 길이로 특성을 설정하여 이 정보를 **product_matching** 추적에 추가했습니다. 메타데이터의 **특성**에서 이 키-값 쌍을 찾을 수 있습니다.

## (선택 사항) 오류 추적

시간이 있으면 오류가 발생한 경우 추적을 사용하는 방법을 검토할 수 있습니다. 오류가 발생할 가능성이 있는 스크립트가 제공됩니다. 이를 실행하고 추적을 검토합니다.

이것은 사용자가 도전하기 위해 고안된 연습이며 지침이 의도적으로 상세하게 설명되어 있지 않습니다.

1. Cloud Shell에서 **error-prompt.py** 스크립트를 엽니다. 이 스크립트는 **start-prompt.py** 스크립트와 동일한 디렉터리에 있습니다. 콘텐츠를 검토합니다.
1. **error-prompt.py** 스크립트를 실행합니다. 메시지가 표시되면 명령줄에 응답을 제공합니다.
1. *출력* 메시지에는 **여정 프로필 생성 실패가 포함되어 있어야 합니다. 추적을 위해 Application Insights를 확인합니다.**.
1. **trip_profile_generation**에 대한 추적으로 이동하여 오류가 발생한 이유를 검사합니다.

<br>
<details>
<summary><b>답변을 얻습니다</b>: 오류가 발생한 이유는 무엇인가요...</summary><br>
<p>generate_trip_profile 함수에 대한 LLM 추적을 검사하는 경우 도우미의 응답에는 출력을 코드 블록으로 형식 지정하는 백틱 및 단어 json이 포함됩니다.

표시에 유용하지만 출력이 더 이상 유효한 JSON이 아니므로 코드에서 문제가 발생합니다. 이로 인해 추가 처리 중에 구문 분석 오류가 발생합니다.

이 오류는 LLM이 출력의 특정 형식을 준수하도록 지시하는 방법으로 인해 발생할 수 있습니다. 사용자 프롬프트에 지침을 포함시키는 것이 시스템 프롬프트에 넣는 것보다 더 효과적입니다.</p>
</details>

## 다른 랩을 찾을 수 있는 위치

[Azure AI 파운드리 학습 포털](https://ai.azure.com)에서 추가 랩 및 연습을 탐색하거나 해당 과정의 **랩 섹션**에서 제공되는 다른 활동을 참조할 수 있습니다.
