---
lab:
  title: 모델 카탈로그에서 언어 모델 비교
---

## 모델 카탈로그에서 언어 모델 비교

사용 사례를 정의한 경우 모델 카탈로그를 사용하여 AI 모델이 문제를 해결하는지 여부를 탐색할 수 있습니다. 모델 카탈로그를 사용하여 배포할 모델을 선택한 다음 비교하여 요구 사항에 가장 적합한 모델을 탐색할 수 있습니다.

이 연습에서는 Azure AI 파운드리 포털의 모델 카탈로그를 통해 두 언어 모델을 비교합니다.

이 연습은 약 **25**분 정도 소요됩니다.

## 시나리오

학생들이 Python에서 코딩하는 방법을 배울 수 있도록 앱을 빌드하려는 경우를 상상해 보세요. 앱에서 학생들이 코드를 작성하고 평가하는 데 도움을 줄 수 있는 자동화된 튜터가 필요합니다. 한 연습에서 학생들은 다음 예제 이미지를 기반으로 파이 차트를 그리는 데 필요한 Python 코드를 작성해야 합니다:

![수학(34.9%), 물리학(28.6%), 화학(20.6%) 및 영어(15.9%)의 섹션이 있는 시험에서 획득한 점수를 보여 주는 파이 차트](./images/demo.png)

이미지를 입력으로 허용하고 정확한 코드를 생성할 수 있는 언어 모델을 선택해야 합니다. 이러한 기준을 충족하는 사용 가능한 모델은 GPT-4 Turbo, GPT-4o 및 GPT-4o mini입니다.

먼저 Azure AI 파운드리 포털에서 이 모델을 사용하는 데 필요한 리소스를 배포해 보겠습니다.

## Azure AI 허브 및 프로젝트 만들기

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
    Get-AzCognitiveServicesAccountKey -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property Key1
     ```

1. 이러한 값은 나중에 사용될 때 복사합니다.

## 모델 비교

유추 인프라가 Azure에서 완전히 관리되는 입력으로 이미지를 허용하는 세 가지 모델이 있다는 것을 알고 계실 것입니다. 이제 두 가지를 비교하여 어떤 것이 사용 사례에 적합한지 결정해야 합니다.

1. 웹 브라우저에서 [Azure AI Foundry 포털](https://ai.azure.com)(`https://ai.azure.com`)을 열고 Azure 자격 증명을 사용하여 로그인합니다.
1. 메시지가 표시되면 이전에 만든 AI 프로젝트를 선택합니다.
1. 왼쪽 메뉴를 사용하여 **모델 카탈로그** 페이지로 이동합니다.
1. **모델 비교**(검색창에서 필터 옆에 있는 버튼을 찾습니다)를 선택합니다.
1. 선택한 모델을 제거합니다.
1. 비교하려는 세 가지 모델(**gpt-4**, **gpt-4o**, **gpt-4o-mini**)을 하나씩 추가합니다. **gpt-4**의 경우, 이미지를 입력으로 허용하는 유일한 버전이므로 선택한 버전이 **turbo-2024-04-09**인지 확인합니다.
1. x-축을 **정확도**로 변경합니다.
1. y-축이 **비용**으로 설정되어 있는지 확인합니다.

플롯을 검토하고 다음 질문에 대답해 보세요.

- *어느 모델이 더 정확한가?*
- *어느 모델이 더 저렴한가? * 

벤치마크 메트릭 정확도는 공개적으로 사용 가능한 제네릭 데이터 세트를 기반으로 계산됩니다. 플롯에서 토큰당 비용이 가장 높지만 정확도가 가장 높지는 않으므로 모델 중 하나를 이미 필터링할 수 있습니다. 결정을 내리기 전에 사용 사례와 관련된 나머지 두 모델의 출력 품질을 살펴보겠습니다.

## 로컬 개발 환경 설정

빠르게 실험하고 반복하려면 VS(Visual Studio) Code에서 Python 코드가 포함된 Notebook을 사용합니다. 로컬 아이디어 발상에 사용할 수 있도록 VS Code를 준비해 보겠습니다.

1. VS Code를 열고 **복제**하여 다음 Git 리포지토리([https://github.com/MicrosoftLearning/mslearn-genaiops.git](https://github.com/MicrosoftLearning/mslearn-genaiops.git))를 만듭니다.
1. 로컬 드라이브에 복제본을 저장하고 복제한 후 폴더를 엽니다.
1. VS Code 탐색기(왼쪽 창)에서 **Files/02** 폴더에 있는 **02-Compare-models.ipynb** Notebook을 엽니다.
1. Notebook의 모든 셀을 실행합니다.

## 정리

Azure AI 서비스 탐색을 마친 경우 이 연습에서 만든 리소스를 삭제하여 불필요한 Azure 비용이 발생하지 않도록 해야 합니다.

1. Azure Portal이 포함된 브라우저 탭으로 돌아가서(또는 새 브라우저 탭에서 [Azure Portal](https://portal.azure.com?azure-portal=true)을 다시 열고) 이 연습에 사용된 리소스를 배포한 리소스 그룹의 콘텐츠를 확인합니다.
1. 도구 모음에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.
