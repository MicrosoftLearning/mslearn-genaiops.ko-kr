---
lab:
  title: 가상 데이터 세트를 사용하여 모델 최적화
  description: 가상 데이터 세트를 만들고 이를 사용하여 모델의 성능과 안정성을 향상시키는 방법을 알아봅니다.
---

## 가상 데이터 세트를 사용하여 모델 최적화

생성형 AI 애플리케이션을 최적화하려면 데이터 세트를 활용하여 모델의 성능과 안정성을 향상시켜야 합니다. 개발자는 가상 데이터를 사용하여 실제 데이터에 없는 다양한 시나리오 및 특수한 사례를 시뮬레이션할 수 있습니다. 또한 모델의 출력 평가는 고품질의 신뢰할 수 있는 AI 애플리케이션을 확보하는 데 매우 중요합니다. 전체 최적화 및 평가 프로세스는 이러한 작업을 간소화하는 강력한 도구와 프레임워크를 제공하는 Azure AI 평가 SDK를 사용하여 효율적으로 관리할 수 있습니다.

이 연습은 약 **30**분\*이 소요됩니다.

> \* 이 예상 기간에 연습이 끝나고 선택적으로 진행하는 작업은 포함되지 않습니다.
## 시나리오

박물관에서 방문자의 경험을 향상시키기 위해 AI 기반 스마트 가이드 앱을 빌드하고 싶다고 상상해 보세요. 이 앱은 역사적 인물에 대한 질문에 답하는 것을 목표로 합니다. 앱의 응답을 평가하려면 이러한 성격과 업무의 다양한 측면을 포괄하는 종합적인 가상 질문-답변 데이터 세트를 만들어야 합니다.

생성형 답변을 제공하기 위해 GPT-4 모델을 선택했습니다. 이제 상황에 맞는 상호 작용을 생성하는 시뮬레이터를 함께 만들어 다양한 시나리오에서 AI의 성능을 평가하려고 합니다.

먼저 이 애플리케이션을 빌드하는 데 필요한 리소스를 배포해 보겠습니다.

## Azure AI 허브 및 프로젝트 만들기

Azure AI 파운드리 포털을 통해 Azure AI 허브 및 프로젝트를 수동으로 만들고 연습에 사용된 모델을 배포할 수 있습니다. 그러나 [Azure Developer CLI(azd)](https://aka.ms/azd)와 함께 템플릿 애플리케이션을 사용하여 이 프로세스를 자동화할 수도 있습니다.

1. 웹 브라우저에서 [Azure Portal](https://portal.azure.com)(`https://portal.azure.com`)을 열고 Azure 자격 증명을 사용하여 로그인합니다.

1. 페이지 상단의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 ***PowerShell*** 환경을 선택합니다. Cloud Shell은 다음과 같이 Azure Portal 아래쪽 창에 명령줄 인터페이스를 제공합니다. Azure Cloud Shell 사용에 관한 자세한 내용은 [Azure Cloud Shell 설명서](https://docs.microsoft.com/azure/cloud-shell/overview)를 참조하세요.

    > **참고**: 이전에 *Bash* 환경을 사용하는 Cloud Shell을 만든 경우 ***PowerShell***로 전환합니다.

1. Cloud Shell 도구 모음의 **설정** 메뉴에서 **클래식 버전으로 이동**을 선택합니다.

    **<font color="red">계속하기 전에 Cloud Shell의 클래식 버전으로 전환했는지 확인합니다.</font>**

1. PowerShell 창에서 다음 명령을 입력하여 이 연습의 리포지토리를 복제합니다.

    ```powershell
   rm -r mslearn-genaiops -f
   git clone https://github.com/MicrosoftLearning/mslearn-genaiops
    ```

1. 리포지토리가 복제된 후 다음 명령을 입력하여 시작 템플릿을 초기화합니다. 
   
    ```powershell
   cd ./mslearn-genaiops/Starter
   azd init
    ```

1. 메시지가 표시되면 프로비전된 모든 리소스에 고유한 이름을 부여하는 기준으로 사용되므로 새 환경의 이름을 지정합니다.
        
1. 다음으로, 다음 명령을 입력하여 시작 템플릿을 실행합니다. 종속 리소스, AI 프로젝트, AI 서비스 및 온라인 엔드포인트를 사용하여 AI 허브를 프로비전합니다. 또한 GPT-4 Turbo, GPT-4o 및 GPT-4o mini 모델을 배포합니다.

    ```powershell
   azd up  
    ```

1. 메시지가 표시되면 사용할 구독을 선택한 다음 리소스 프로비전에 대해 다음 위치 중 하나를 선택합니다.
   - 미국 동부
   - 미국 동부 2
   - 미국 중북부
   - 미국 중남부
   - 스웨덴 중부
   - 미국 서부
   - 미국 서부 3
    
1. 스크립트가 완료될 때까지 기다리세요. 일반적으로 약 10분이 걸리지만 경우에 따라 더 오래 걸릴 수 있습니다.

    > **참고**: Azure OpenAI 리소스는 지역 할당량에 따라 테넌트 수준에서 제한됩니다. 나열된 지역에는 이 연습에 사용된 모델 형식에 대한 기본 할당량이 포함되어 있습니다. 지역을 무작위로 선택하면 단일 지역이 할당량 한도에 도달할 위험이 줄어듭니다. 할당량 한도에 도달하는 경우 다른 지역에 다른 리소스 그룹을 만들어야 할 수도 있습니다. [지역별 모델 가용성](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/models?tabs=standard%2Cstandard-chat-completions#global-standard-model-availability)에 대해 자세히 알아보기

    <details>
      <summary><b>문제 해결 팁</b>: 지정된 지역에서 사용할 수 있는 할당량 없음</summary>
        <p>선택한 지역에서 사용할 수 있는 할당량이 없어서 모델에 대한 배포 오류가 발생하는 경우 다음 명령을 실행해 봅니다.</p>
        <ul>
          <pre><code>azd env set AZURE_ENV_NAME new_env_name
   azd env set AZURE_RESOURCE_GROUP new_rg_name
   azd env set AZURE_LOCATION new_location
   azd up</code></pre>
        <code>new_env_name</code>, <code>new_rg_name</code>, <code>new_location</code>(을)를 새 값으로 바꿉니다. 새 위치는 <code>eastus2</code>, <code>northcentralus</code> 등과 같이 연습 시작 시 나열된 지역 중 하나에 있어야 합니다.
        </ul>
    </details>

1. 모든 리소스가 프로비전되면 다음 명령을 사용하여 엔드포인트 및 액세스 키를 AI 서비스 리소스로 가져옵니다. `<rg-env_name>` 및 `<aoai-xxxxxxxxxx>`(을)를 리소스 그룹 및 AI 서비스 리소스의 이름으로 바꿔야 한다는 점에 유의합니다. 둘 다 배포의 출력에 인쇄됩니다.

     ```powershell
    Get-AzCognitiveServicesAccount -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property endpoint
     ```

     ```powershell
    Get-AzCognitiveServicesAccountKey -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property Key1
     ```

1. 이러한 값은 나중에 사용될 때 복사합니다.

## Cloud Shell에서 개발 환경 설정

신속하게 실험해보고 반복하려면 Cloud Shell에서 Python 스크립트 집합을 사용합니다.

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 이 연습에 사용된 코드 파일이 있는 폴더로 이동합니다.

     ```powershell
    cd ~/mslearn-genaiops/Files/06/
     ```

1. 다음 명령을 입력하여 가상 환경을 활성화하고 필요한 라이브러리를 설치합니다.

    ```powershell
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-ai-evaluation azure-ai-projects promptflow wikipedia aiohttp openai==1.77.0
    ```

1. 제공된 구성 파일을 열려면 다음 명령을 입력합니다.

    ```powershell
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. 코드 파일에서 **your_azure_openai_service_endpoint** 및 **your_azure_openai_service_api_key** 자리 표시자를 이전에 복사한 엔드포인트 및 키 값으로 바꿉니다.
1. 자리 표시자를 바꾼 *후* 코드 편집기에서 **CTRL+S** 명령 또는 **마우스 오른쪽 단추 클릭 > 저장**을 사용하여 변경 내용을 저장한 다음, **CTRL+Q** 명령 또는 **마우스 오른쪽 단추 클릭 > 끝내기**를 사용하여 Cloud Shell 명령줄을 열어둔 채 코드 편집기를 닫습니다.

## 가상 데이터 생성

이제 가상 데이터 세트를 생성하고 이를 사용하여 미리 학습된 모델의 품질을 평가하는 스크립트를 실행합니다.

1. 다음 명령을 실행하여 제공된 **스크립트를 편집합니다**.

    ```powershell
   code generate_synth_data.py
    ```

1. 스크립트에서 **#Define 콜백 함수**를 찾습니다.
1. 이 주석 아래에 다음 코드를 붙여넣습니다.

    ```
    async def callback(
        messages: List[Dict],
        stream: bool = False,
        session_state: Any = None,  # noqa: ANN401
        context: Optional[Dict[str, Any]] = None,
    ) -> dict:
        messages_list = messages["messages"]
        # Get the last message
        latest_message = messages_list[-1]
        query = latest_message["content"]
        context = text
        # Call your endpoint or AI application here
        current_dir = os.getcwd()
        prompty_path = os.path.join(current_dir, "application.prompty")
        _flow = load_flow(source=prompty_path)
        response = _flow(query=query, context=context, conversation_history=messages_list)
        # Format the response to follow the OpenAI chat protocol
        formatted_response = {
            "content": response,
            "role": "assistant",
            "context": context,
        }
        messages["messages"].append(formatted_response)
        return {
            "messages": messages["messages"],
            "stream": stream,
            "session_state": session_state,
            "context": context
        }
    ```

    대상 콜백 함수를 지정하여 시뮬레이트할 애플리케이션 엔드포인트를 가져올 수 있습니다. 이 경우 Prompty 파일 `application.prompty`가 있는 LLM인 애플리케이션을 사용합니다. 위의 콜백 함수는 다음 작업을 수행하여 시뮬레이터에서 생성된 각 메시지를 처리합니다.
    * 최신 사용자 메시지를 검색합니다.
    * application.prompty에서 프롬프트 흐름을 로드합니다.
    * 프롬프트 흐름을 사용하여 응답을 생성합니다.
    * OpenAI 채팅 프로토콜을 준수하도록 응답 형식을 지정합니다.
    * 메시지 목록에 도우미의 응답을 추가합니다.

    >**참고**: Prompty 사용에 대한 자세한 내용은 [Prompty의 설명서](https://www.prompty.ai/docs)를 참조하세요.

1. 다음으로 **# 시뮬레이터 실행**을 찾습니다.
1. 이 주석 아래에 다음 코드를 붙여넣습니다.

    ```
    model_config = {
        "azure_endpoint": os.getenv("AZURE_OPENAI_ENDPOINT"),
        "api_key": os.getenv("AZURE_OPENAI_API_KEY"),
        "azure_deployment": os.getenv("AZURE_OPENAI_DEPLOYMENT"),
    }
    
    simulator = Simulator(model_config=model_config)
    
    outputs = asyncio.run(simulator(
        target=callback,
        text=text,
        num_queries=1,  # Minimal number of queries
    ))
    
    output_file = "simulation_output.jsonl"
    with open(output_file, "w") as file:
        for output in outputs:
            file.write(output.to_eval_qr_json_lines())
    ```

   위의 코드는 시뮬레이터를 초기화하고 실행하여 이전에 Wikipedia에서 추출한 텍스트를 기준으로 가상 대화를 생성합니다.

1. 다음으로, **#모델 평가**를 찾습니다.
1. 이 주석 아래에 다음 코드를 붙여넣습니다.

    ```
    groundedness_evaluator = GroundednessEvaluator(model_config=model_config)
    eval_output = evaluate(
        data=output_file,
        evaluators={
            "groundedness": groundedness_evaluator
        },
        output_path="groundedness_eval_output.json"
    )
    ```

    이제 데이터 세트가 있으므로 생성형 AI 애플리케이션의 품질과 효율성을 평가할 수 있습니다. 위의 코드에서는 근거를 품질 메트릭으로 사용합니다.

1. 변경 내용을 저장합니다.
1. 코드 편집기 아래의 Cloud Shell 명령줄 창에서 다음 명령을 입력하여 **스크립트를 실행**합니다.

    ```
   python generate_synth_data.py
    ```

    스크립트가 완료되면 `download simulation_output.jsonl` 및 `download groundedness_eval_output.json`을 실행하여 출력 파일을 다운로드하고 해당 콘텐츠를 검토할 수 있습니다. 근거 메트릭이 3.0에 가깝지 않은 경우 `application.prompty` 파일에서 `temperature`, `top_p`, `presence_penalty` 또는 `frequency_penalty`와 같은 LLM 매개 변수를 변경하고, 스크립트를 다시 실행하여 평가를 위한 새 데이터 세트를 생성할 수 있습니다. 다른 컨텍스트에 따라 가상 데이터 세트를 가져오도록 `wiki_search_term`을 변경할 수도 있습니다.

## (선택 사항) 모델 미세 조정

추가 시간이 있는 경우 생성된 데이터 세트를 사용하여 Azure AI 파운드리에서 모델을 미세 조정할 수 있습니다. 미세 조정은 클라우드 인프라 리소스에 따라 달라지며, 데이터 센터 용량 및 동시 수요에 따라 프로비전하는 데 걸리는 시간은 달라질 수 있습니다.

1. 새 브라우저 탭을 열고 `https://ai.azure.com`의 [Azure AI 파운드리 포털](https://ai.azure.com)로 이동하여 Azure 자격 증명을 사용하여 로그인합니다.
1. AI 파운드리 홈페이지에서 연습의 시작 부분에서 만든 프로젝트를 선택합니다.
1. 왼쪽 메뉴를 사용하여 **빌드 및 사용자 지정** 섹션 아래의 **미세 조정** 페이지로 이동합니다.
1. 새 미세 조정 모델을 추가하려면 버튼을 선택하고 **gpt-4o** 모델을 선택한 다음 **다음**을 선택합니다.
1. 다음 구성을 사용하여 모델을 **미세 조정**합니다.
    - **모델 버전**: *기본 버전 선택*
    - **사용자 지정 방법**: 감독됨
    - **모델 접미사**: `ft-travel`
    - **연결된 AI 리소스**: *허브를 만들 때 만들어진 연결을 선택합니다. 기본적으로 선택되어 있을 것입니다.*
    - **학습 데이터**: 파일 업로드

    <details>  
    <summary><b>문제 해결 팁</b>: 사용 권한 오류</summary>
    <p>사용 권한 오류가 표시되는 경우 다음 문제 해결 방법을 시도합니다.</p>
    <ul>
        <li>Azure Portal에서 AI 서비스 리소스를 선택합니다.</li>
        <li>리소스 관리의 ID 탭에서 시스템에서 할당된 관리 ID인지 확인합니다.</li>
        <li>관련된 스토리지 계정으로 이동합니다. IAM 페이지에서 역할 할당 <em>스토리지 Blob 데이터 소유자</em>를 추가합니다.</li>
        <li><strong>액세스 할당</strong>에서 <strong>관리 ID</strong>, <strong>+ 구성원 선택</strong>, <strong>모든 시스템 할당 관리 ID</strong>를 차례로 선택한 후 Azure AI 서비스 리소스를 선택합니다.</li>
        <li>새 설정을 검토하고 할당하여 저장하고 이전 단계를 다시 시도합니다.</li>
    </ul>
    </details>

    - **파일 업로드**: 이전 단계에서 다운로드한 JSONL 파일을 선택합니다.
    - **유효성 검사 데이터**: 없음
    - **작업 매개 변수**: *기본 설정 유지*
1. 미세 조정이 시작되고 완료하는 데 다소 시간이 걸릴 수 있습니다.

    > **참고**: 미세 조정 및 배포에는 상당한 시간이 걸릴 수 있으므로(30분 이상) 주기적으로 다시 확인해야 할 수 있습니다. 모델 작업을 미세 조정하고 **로그** 탭을 확인하여 지금까지의 진행 상황을 더 자세히 볼 수 있습니다.

## (선택 사항) 미세 조정된 모델 배포

미세 조정이 성공적으로 완료되면 미세 조정된 모델을 배포할 수 있습니다.

1. 세부 정보 페이지를 열려면 미세 조정 작업 링크를 선택합니다. **메트릭** 탭을 선택하고 미세 조정 메트릭을 탐색합니다.
1. 다음 구성을 사용하여 미세 조정된 모델을 배포합니다.
    - **배포 이름**: *모델 배포에 대한 고유한 이름*
    - **배포 유형**: 표준
    - **분당 토큰 속도 제한(천)**: 5K *(또는 5K 미만인 경우 구독에서 사용할 수 있는 최댓값)
    - **콘텐츠 필터**: 기본값
1. 배포가 완료될 때까지 기다렸다가 테스트할 수 있으며, 시간이 걸릴 수 있습니다. **프로비전 상태**가 성공할 때까지 확인합니다(업데이트된 상태를 보려면 브라우저를 새로 고쳐야 할 수 있음).
1. 배포가 준비되면 미세 조정된 모델로 이동하고 **플레이그라운드에서 열기**를 선택합니다.

    이제 미세 조정된 모델을 배포했으므로 기본 모델과 마찬가지로 채팅 플레이그라운드에서 테스트할 수 있습니다.

## 결론

이 연습에서는 사용자와 채팅 완성 앱 간의 대화를 시뮬레이트하는 가상 데이터 세트를 만들었습니다. 이 데이터 세트를 사용하면 앱 응답의 품질을 평가하고 미세 조정하여 원하는 결과를 얻을 수 있습니다.

## 정리

Azure AI 서비스 탐색을 마친 경우 이 연습에서 만든 리소스를 삭제하여 불필요한 Azure 비용이 발생하지 않도록 해야 합니다.

1. Azure Portal이 포함된 브라우저 탭으로 돌아가서(또는 새 브라우저 탭에서 [Azure Portal](https://portal.azure.com?azure-portal=true)을 다시 열고) 이 연습에 사용된 리소스를 배포한 리소스 그룹의 콘텐츠를 확인합니다.
1. 도구 모음에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.
