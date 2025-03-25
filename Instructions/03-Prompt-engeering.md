---
lab:
  title: Prompty를 사용하여 프롬프트 엔지니어링 살펴보기
---

## Prompty를 사용하여 프롬프트 엔지니어링 살펴보기

아이디어를 개발하는 동안 언어 모델을 사용하여 다양한 프롬프트를 빠르게 테스트하고 개선하려고 합니다. Azure AI 파운드리 포털의 플레이그라운드를 통해 프롬프트 엔지니어링에 접근하거나 코드 우선 접근 방식을 위해 prompty를 사용하는 등 다양한 방법으로 접근할 수 있습니다.

이 연습에서는 Azure AI 파운드리를 통해 배포된 모델을 사용하여 Visual Studio Code에서 prompty로 프롬프트 엔지니어링을 살펴봅니다.

이 연습은 약 **40**분 정도 소요됩니다.

## 시나리오

학생들이 Python에서 코딩하는 방법을 배울 수 있도록 앱을 빌드하려는 경우를 상상해 보세요. 앱에서 학생들이 코드를 작성하고 평가하는 데 도움을 줄 수 있는 자동화된 튜터가 필요합니다. 하지만 채팅 앱이 모든 답변만 제공하는 것은 바람직하지 않을 수 있습니다. 학생이 진행 방법에 대해 생각할 수 있도록 개인화된 힌트를 받기를 원합니다.

실험을 시작할 GPT-4 모델을 선택했습니다. 이제 프롬프트 엔지니어링을 적용하여 채팅의 행동을 개인화된 힌트를 생성하는 튜터로 안내하고자 합니다.

먼저 Azure AI 파운드리 포털에서 이 모델을 사용하는 데 필요한 리소스를 배포해 보겠습니다.

## Azure AI 허브 및 프로젝트 만들기

> **참고**: Azure AI 허브 및 프로젝트가 이미 있는 경우 이 절차를 건너뛰고 기존 프로젝트를 사용할 수 있습니다.

Azure AI 파운드리 포털을 통해 Azure AI 허브 및 프로젝트를 수동으로 만들고 연습에 사용된 모델을 배포할 수 있습니다. 그러나 [Azure Developer CLI(azd)](https://aka.ms/azd)와 함께 템플릿 애플리케이션을 사용하여 이 프로세스를 자동화할 수도 있습니다.

1. 웹 브라우저에서 [Azure Portal](https://portal.azure.com)(`https://portal.azure.com`)을 열고 Azure 자격 증명을 사용하여 로그인합니다.

1. 페이지 상단의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 ***PowerShell*** 환경을 선택합니다. Cloud Shell은 다음과 같이 Azure Portal 아래쪽 창에 명령줄 인터페이스를 제공합니다. Azure Cloud Shell 사용에 관한 자세한 내용은 [Azure Cloud Shell 설명서](https://docs.microsoft.com/azure/cloud-shell/overview)를 참조하세요.

    > **참고**: 이전에 *Bash* 환경을 사용하는 Cloud Shell을 만든 경우 ***PowerShell***로 전환합니다.

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
        
1. 다음으로, 다음 명령을 입력하여 시작 템플릿을 실행합니다. 종속 리소스, AI 프로젝트, AI 서비스 및 온라인 엔드포인트를 사용하여 AI 허브를 프로비전합니다.

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
    Get-AzCognitiveServicesAccountKey -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property Key1
     ```

1. 이러한 값은 나중에 사용될 때 복사합니다.
   
## 로컬 개발 환경 설정

빠르게 실험하고 반복하려면 VS(Visual Studio) Code에서 prompty를 사용합니다. 로컬 아이디어 발상에 사용할 수 있도록 VS Code를 준비해 보겠습니다.

1. VS Code를 열고 **복제**하여 다음 Git 리포지토리([https://github.com/MicrosoftLearning/mslearn-genaiops.git](https://github.com/MicrosoftLearning/mslearn-genaiops.git))를 만듭니다.
1. 로컬 드라이브에 복제본을 저장하고 복제한 후 폴더를 엽니다.
1. VS Code의 확장 창에서 **prompty** 확장을 검색하여 설치합니다.
1. VS Code 탐색기(왼쪽 창)에서 **Files/03** 폴더를 마우스 오른쪽 단추로 클릭합니다.
1. 드롭다운 메뉴에서 **새 prompty**를 선택합니다.
1. **basic.prompty**라는 이름의 새로 만든 파일을 엽니다.
1. 오른쪽 상단의 **재생** 버튼을 선택하여 prompty 파일을 실행합니다(또는 F5를 누릅니다).
1. 로그인하라는 메시지가 표시되면 **허용**을 선택합니다.
1. Azure 계정을 선택하고 로그인합니다.
1. VS Code로 돌아가면 오류 메시지와 함께 **출력** 창이 열립니다. 배포된 모델이 지정되지 않았거나 찾을 수 없다는 오류 메시지가 표시됩니다.

오류를 해결하려면 prompty에서 사용할 모델을 구성해야 합니다.

## 프롬프트 메타데이터 업데이트

prompty 파일을 실행하려면 응답을 생성하는 데 사용할 언어 모델을 지정해야 합니다. 메타데이터는 rompty 파일의 *frontmatter*에 정의되어 있습니다. 모델 구성 및 기타 정보로 메타데이터를 업데이트해 보겠습니다.

1. Visual Studio Code 터미널 창을 엽니다.
1. 같은 폴더에 있는 **basic.prompty** 파일을 복사하고 복사본의 이름을 `chat-1.prompty`(으)로 바꿉니다.
1. **chat-1.prompty**를 열고 다음 필드를 업데이트하여 몇 가지 기본 정보를 변경합니다.

    - **이름:**

        ```yaml
        name: Python Tutor Prompt
        ```

    - **설명:**

        ```yaml
        description: A teaching assistant for students wanting to learn how to write and edit Python code.
        ```

    - **배포된 모델**:

        ```yaml
        azure_deployment: ${env:AZURE_OPENAI_CHAT_DEPLOYMENT}
        ```

1. 다음으로, **azure_deployment** 매개 변수 아래에 API 키에 대한 다음 자리 표시자를 추가합니다.

    - **엔드포인트 키**:

        ```yaml
        api_key: ${env:AZURE_OPENAI_API_KEY}
        ```

1. 업데이트된 prompty 파일을 저장합니다.

이제 prompty 파일에 필요한 모든 매개변수가 있지만 일부 매개변수는 자리 표시자를 사용하여 필요한 값을 얻습니다. 자리 표시자는 동일한 폴더의 **.env** 파일에 저장됩니다.

## 모델 구성 업데이트

prompty에서 사용하는 모델을 지정하려면 .env 파일에 모델의 정보를 제공해야 합니다.

1. **Files/03** 폴더에서 **.env** 파일을 엽니다.
1. 각 자리 표시자를 Azure Portal의 모델 배포 출력에서 이전에 복사한 값으로 업데이트합니다.

    ```yaml
    - AZURE_OPENAI_CHAT_DEPLOYMENT="gpt-4"
    - AZURE_OPENAI_ENDPOINT="<Your endpoint target URI>"
    - AZURE_OPENAI_API_KEY="<Your endpoint key>"
    ```

1. .env 파일을 저장합니다.
1. **chat-1.prompty** 파일을 다시 실행합니다.

이제 샘플 입력만 사용하므로 시나리오와 관련이 없더라도 AI에서 생성한 응답을 가져와야 합니다. 템플릿을 업데이트하여 AI 보조 교사로 만들어 보겠습니다.

## 샘플 섹션 편집

샘플 섹션에서는 prompty에 대한 입력을 지정하고 입력이 제공되지 않은 경우 사용할 기본값을 제공합니다.

1. 다음 매개 변수의 필드를 편집합니다.

    - **firstName**: 다른 이름을 선택합니다.
    - **context**: 이 전체 섹션을 제거합니다.
    - **질문**: 제공된 텍스트를 다음으로 바꿉니다.

    ```yaml
    What is the difference between 'for' loops and 'while' loops?
    ```

    이제 **샘플** 섹션이 다음과 같이 표시됩니다.
    
    ```yaml
    sample:
    firstName: Daniel
    question: What is the difference between 'for' loops and 'while' loops?
    ```

    1. 업데이트된 prompty 파일을 실행하고 출력을 검토합니다.

