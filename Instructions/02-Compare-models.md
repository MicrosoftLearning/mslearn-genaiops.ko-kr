---
lab:
  title: 모델 카탈로그에서 언어 모델 비교
  description: 생성형 AI 프로젝트에 적합한 모델을 비교하여 선택하는 방법을 알아봅니다.
---

## 모델 카탈로그에서 언어 모델 비교

사용 사례를 정의한 경우 모델 카탈로그를 사용하여 AI 모델이 문제를 해결하는지 여부를 탐색할 수 있습니다. 모델 카탈로그를 사용하여 배포할 모델을 선택한 다음 비교하여 요구 사항에 가장 적합한 모델을 탐색할 수 있습니다.

이 연습에서는 Azure AI 파운드리 포털의 모델 카탈로그를 통해 두 언어 모델을 비교합니다.

이 연습은 약 **30**분 정도 소요됩니다.

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

## 모델 비교

유추 인프라가 Azure에서 완전히 관리되는 입력으로 이미지를 허용하는 세 가지 모델이 있다는 것을 알고 계실 것입니다. 이제 두 가지를 비교하여 어떤 것이 사용 사례에 적합한지 결정해야 합니다.

1. 새 브라우저 탭의 `https://ai.azure.com`에서 [Azure AI 파운드리 포털](https://ai.azure.com)을 열고 Azure 자격 증명을 사용하여 로그인합니다.
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

## Cloud Shell에서 개발 환경 설정

신속하게 실험해보고 반복하려면 Cloud Shell에서 Python 스크립트 집합을 사용합니다.

1. Azure AI 파운드리 포털에서 프로젝트의 **개요** 페이지를 봅니다.
1. **프로젝트 세부 정보** 영역에서 **프로젝트 연결 문자열**을 확인합니다.
1. 메모장에 문자열을 저장합니다. 이 연결 문자열 사용하여 클라이언트 응용 프로그램에서 프로젝트에 연결합니다.
1. Azure Portal 탭으로 돌아가서 Cloud Shell을 열고(이전에 닫은 경우) 다음 명령을 실행하여 이 연습에 사용된 코드 파일이 있는 폴더로 이동합니다.

     ```powershell
    cd ~/mslearn-genaiops/Files/02/
     ```

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 필요한 라이브러리를 설치합니다.

    ```powershell
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects openai matplotlib
    ```

1. 제공된 구성 파일을 열려면 다음 명령을 입력합니다.

    ```powershell
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. 코드 파일에서 **your_project_connection_string** 자리 표시자를 프로젝트의 연결 문자열로 바꿉니다(Azure AI 파운드리 포털의 프로젝트 **개요** 페이지에서 복사함). 연습에 사용된 첫 번째 및 두 번째 모델이 각각 **gpt-4o** 및 **gpt-4o-mini**인지 확인합니다.
1. 자리 표시자를 바꾼 *후* 코드 편집기에서 **CTRL+S** 명령 또는 **마우스 오른쪽 단추 클릭 > 저장**을 사용하여 변경 내용을 저장한 다음, **CTRL+Q** 명령 또는 **마우스 오른쪽 단추 클릭 > 끝내기**를 사용하여 Cloud Shell 명령줄을 열어둔 채 코드 편집기를 닫습니다.

## 배포된 모델에 프롬프트 보내기

이제 배포된 모델에 다른 프롬프트를 보내는 여러 스크립트를 실행합니다. 이러한 상호 작용은 나중에 Azure Monitor에서 관찰할 수 있는 데이터를 생성합니다.

1. 제공된 **첫 번째 스크립트를 보려면** 다음 명령을 실행합니다.

    ```powershell
   code model1.py
    ```

스크립트는 이 연습에 사용된 이미지를 데이터 URL로 인코딩합니다. 이 URL은 첫 번째 텍스트 프롬프트와 함께 채팅 완성 요청에 직접 이미지를 포함하는 데 사용됩니다. 다음으로, 스크립트는 모델의 응답을 출력하고 채팅 기록에 추가한 다음, 두 번째 프롬프트를 제출합니다. 두 번째 프롬프트는 나중에 관찰된 메트릭을 더 중요하게 인식하도록 하기 위해 제출되고 저장되지만 코드의 선택적 섹션에 대한 주석 처리를 제거하여 두 번째 응답도 출력으로 사용할 수 있습니다.

1. 코드 편집기 아래의 Cloud Shell 명령줄 창에서 다음 명령을 입력하여 **첫 번째** 스크립트를 실행합니다.

    ```powershell
   python model1.py
    ```

    모델은 추가 분석을 위해 Application Insights로 캡처되는 응답을 생성합니다. 두 번째 모델을 사용하여 차이점을 살펴보겠습니다.

1. 코드 편집기 아래의 Cloud Shell 명령줄 창에서 다음 명령을 입력하여 **두 번째** 스크립트를 실행합니다.

    ```powershell
   python model2.py
    ```

    이제 두 모델의 출력이 어떤 방식으로 다른가요?

    > **참고**: 필요에 따라 코드 블록을 복사하고, 명령 `code your_filename.py`를 실행하고, 편집기에서 코드를 붙여넣고, 파일을 저장한 다음, 명령 `python your_filename.py`를 실행하여 답변으로 지정된 스크립트를 테스트할 수 있습니다. 스크립트가 성공적으로 실행된 경우 다운로드하거나 `download imgs/gpt-4o.jpg` 또는 `download imgs/gpt-4o-mini.jpg`를 사용하여 다운로드 할 수 있는 저장된 이미지가 생성됩니다.

## 모델의 토큰 사용량 비교

마지막으로 각 모델에 대해 시간에 따라 처리된 토큰 수를 그림으로 그리는 세 번째 스크립트를 실행합니다. 이 데이터는 Azure Monitor에서 가져옵니다.

1. 마지막 스크립트를 실행하기 전에 Azure Portal에서 Azure AI 서비스의 리소스 ID를 복사해야 합니다. Azure AI Services 리소스의 개요 페이지로 이동하여 **JSON 보기**를 선택합니다. 리소스 ID를 복사하고 코드 파일의 `your_resource_id` 자리 표시자를 바꿉니다.

    ```powershell
   code plot.py
    ```

1. 변경 내용을 저장합니다.

1. 코드 편집기 아래의 Cloud Shell 명령줄 창에서 다음 명령을 입력하여 **세 번째** 스크립트를 실행합니다.

    ```powershell
   python plot.py
    ```

1. 스크립트가 완료되면 다음 명령을 입력하여 메트릭 플롯을 다운로드합니다.

    ```powershell
   download imgs/plot.png
    ```

## 결론

플롯을 검토하고 이전에 관찰된 정확도 및 비용 차트의 벤치마크 값을 기억하면서 사용 사례에 가장 적합한 모델을 결정할 수 있나요? 출력 정확도의 차이가 생성된 토큰 수와 그로 인한 비용 차이보다 더 큰 가치를 가지나요?

## 정리

Azure AI 서비스 탐색을 마친 경우 이 연습에서 만든 리소스를 삭제하여 불필요한 Azure 비용이 발생하지 않도록 해야 합니다.

1. Azure Portal이 포함된 브라우저 탭으로 돌아가서(또는 새 브라우저 탭에서 [Azure Portal](https://portal.azure.com?azure-portal=true)을 다시 열고) 이 연습에 사용된 리소스를 배포한 리소스 그룹의 콘텐츠를 확인합니다.
1. 도구 모음에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.
