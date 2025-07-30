---
lab:
  title: RAG 시스템 오케스트레이션
  description: 앱에서 RAG(검색 증강 생성) 시스템을 구현하여 생성된 응답의 정확도와 관련성을 향상시키는 방법을 알아봅니다.
---

## RAG 시스템 오케스트레이션

RAG(검색 증강 생성) 시스템은 대규모 언어 모델의 강력한 성능과 효율적인 검색 메커니즘을 결합하여 생성된 응답의 정확성과 관련성을 향상시킵니다. 오케스트레이션을 위한 LangChain과 AI 기능을 위한 Azure AI 파운드리를 활용하면 데이터 집합에서 관련 정보를 검색하고 일관된 응답을 생성하는 강력한 파이프라인을 만들 수 있습니다. 이 연습에서는 환경을 설정하고, 데이터를 전처리하고, 임베딩을 만들고, 인덱스를 빌드하여 궁극적으로 RAG 시스템을 효과적으로 구현할 수 있도록 하는 단계를 안내합니다.

이 연습은 약 **30**분 정도 소요됩니다.

## 시나리오

런던에 있는 호텔을 추천하는 앱을 빌드한다고 상상해 보세요. 앱에서는 호텔을 추천할 뿐만 아니라 사용자가 호텔에 대해 궁금해할 수 있는 질문에 답할 수 있는 에이전트가 필요합니다.

생성형 답변을 제공하기 위해 GPT-4 모델을 선택했습니다. 이제 다른 사용자의 리뷰를 기반으로 모델에 그라운딩 데이터를 제공하여 채팅의 동작이 개인화된 추천을 제공하도록 안내하는 RAG 시스템을 구성하려고 합니다.

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
    cd ~/mslearn-genaiops/Files/04/
     ```

1. 다음 명령을 입력하여 가상 환경을 활성화하고 필요한 라이브러리를 설치합니다.

    ```powershell
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv langchain-text-splitters langchain-community langchain-openai
    ```

1. 제공된 구성 파일을 열려면 다음 명령을 입력합니다.

    ```powershell
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. 코드 파일에서 **your_azure_openai_service_endpoint** 및 **your_azure_openai_service_api_key** 자리 표시자를 이전에 복사한 엔드포인트 및 키 값으로 바꿉니다.
1. 자리 표시자를 바꾼 *후* 코드 편집기에서 **CTRL+S** 명령 또는 **마우스 오른쪽 단추 클릭 > 저장**을 사용하여 변경 내용을 저장한 다음, **CTRL+Q** 명령 또는 **마우스 오른쪽 단추 클릭 > 끝내기**를 사용하여 Cloud Shell 명령줄을 열어둔 채 코드 편집기를 닫습니다.

## RAG 구현

이제 데이터를 수집 및 전처리하고, 포함을 만들고, 벡터 저장소 및 인덱스를 빌드하는 스크립트를 실행하여 궁극적으로 RAG 시스템을 효과적으로 구현할 수 있습니다.

1. 다음 명령을 실행하여 제공된 **스크립트를 봅니다**.

    ```powershell
   code RAG.py
    ```

1. 스크립트를 검토하고 호텔 리뷰가 있는 .csv 파일을 기반 데이터로 사용하는지 확인합니다. 명령 `download app_hotel_reviews.csv`를 실행하고 파일을 열어 이 파일의 내용을 볼 수 있습니다.
1. 명령줄에서 다음 명령을 입력하여 **스크립트를 실행합니다**.

    ```
   python RAG.py
    ```

1. 애플리케이션이 실행되면 질문(예: `Where can I stay in London?`)을 제시한 다음, 나중에 보다 구체적인 사항을 문의할 수 있습니다.

## 결론

이 연습에서는 기본 구성 요소를 사용하여 일반적인 RAG 시스템을 빌드했습니다. 사용자 고유의 문서를 통해 모델의 응답을 제공하여, LLM이 응답을 작성할 때 사용하는 기반 데이터를 제공합니다. 엔터프라이즈 솔루션의 경우 생성형 AI를 엔터프라이즈 콘텐츠로 제한할 수 있습니다.

## 정리

Azure AI 서비스 탐색을 마친 경우 이 연습에서 만든 리소스를 삭제하여 불필요한 Azure 비용이 발생하지 않도록 해야 합니다.

1. Azure Portal이 포함된 브라우저 탭으로 돌아가서(또는 새 브라우저 탭에서 [Azure Portal](https://portal.azure.com?azure-portal=true)을 다시 열고) 이 연습에 사용된 리소스를 배포한 리소스 그룹의 콘텐츠를 확인합니다.
1. 도구 모음에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.
