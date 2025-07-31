---
lab:
  title: Prompty를 사용하여 프롬프트 엔지니어링 살펴보기
  description: Prompty를 사용하여 언어 모델의 다양한 프롬프트를 신속하게 테스트 및 개선하고 최상의 결과를 위해 생성 및 오케스트레이션되는지 확인하는 방법을 알아봅니다.
---

## Prompty를 사용하여 프롬프트 엔지니어링 살펴보기

이 연습에는 약 **45**분이 소요됩니다.

> **참고**: 이 연습에서는 Azure AI 파운드리에 대해 잘 알고 있다고 가정하므로 일부 지침은 보다 적극적인 탐색 및 실습 학습을 장려하기 위해 의도적으로 상세히 설명되어 있지 않습니다.

## 소개

아이디어를 개발하는 동안 언어 모델을 사용하여 다양한 프롬프트를 빠르게 테스트하고 개선하려고 합니다. Azure AI 파운드리 포털의 플레이그라운드를 통해 프롬프트 엔지니어링에 접근하거나 코드 우선 접근 방식을 위해 prompty를 사용하는 등 다양한 방법으로 접근할 수 있습니다.

이 연습에서는 Azure AI 파운드리를 통해 배포된 모델을 사용하여 Azure Cloud Shell의 Prompty를 통해 수행하는 프롬프트 엔지니어링을 살펴봅니다.

## 환경 설정

이 연습의 작업을 완료하려면 다음이 필요합니다.

- Azure AI 파운드리 허브,
- Azure AI 파운드리 프로젝트,
- 배포된 모델(예: GPT-4o)

### Azure AI 허브 및 프로젝트 만들기

> **참고**: Azure AI 프로젝트가 이미 있는 경우 이 절차를 건너뛰고 기존 프로젝트를 사용할 수 있습니다.

Azure AI 파운드리 포털을 통해 Azure AI 프로젝트를 수동으로 만들고 연습에 사용된 모델을 배포할 수 있습니다. 그러나 [Azure Developer CLI(azd)](https://aka.ms/azd)와 함께 템플릿 애플리케이션을 사용하여 이 프로세스를 자동화할 수도 있습니다.

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

### Cloud Shell에서 가상 환경 설정

신속하게 실험해보고 반복하려면 Cloud Shell에서 Python 스크립트 집합을 사용합니다.

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 이 연습에 사용된 코드 파일이 있는 폴더로 이동합니다.

     ```powershell
    cd ~/mslearn-genaiops/Files/03/
     ```

1. 다음 명령을 입력하여 가상 환경을 활성화하고 필요한 라이브러리를 설치합니다.

    ```powershell
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv openai tiktoken azure-ai-projects prompty[azure]
    ```

1. 제공된 구성 파일을 열려면 다음 명령을 입력합니다.

    ```powershell
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. 코드 파일에서 **ENDPOINTNAME** 및 **APIKEY** 자리 표시자를 이전에 복사한 엔드포인트 및 키 값으로 바꿉니다.
1. 자리 표시자를 바꾼 *후* 코드 편집기에서 **CTRL+S** 명령 또는 **마우스 오른쪽 단추 클릭 > 저장**을 사용하여 변경 내용을 저장한 다음, **CTRL+Q** 명령 또는 **마우스 오른쪽 단추 클릭 > 끝내기**를 사용하여 Cloud Shell 명령줄을 열어둔 채 코드 편집기를 닫습니다.

## 시스템 프롬프트 최적화

생성형 AI의 기능을 유지하면서 시스템 프롬프트의 길이를 최소화하는 것이 대규모 배포의 기본 사항입니다. 프롬프트가 짧을수록 AI 모델이 처리하는 토큰 수가 줄어들고 계산 리소스도 더 적게 사용되어 응답 시간이 단축됩니다.

1. 제공된 응용 프로그램 파일을 열려면 다음 명령을 입력합니다.

    ```powershell
   code optimize-prompt.py
    ```

    코드를 검토하면 스크립트가 미리 정의된 시스템 프롬프트가 이미 있는 `start.prompty` 템플릿 파일을 실행한다는 점을 확인할 수 있습니다.

1. `code start.prompty`를 실행하여 시스템 프롬프트를 확인합니다. 의도를 명확하고 효과적으로 유지하면서 어떻게 간결하게 만들 수 있는지 고려합니다. 예시:

   ```python
   original_prompt = "You are a helpful assistant. Your job is to answer questions and provide information to users in a concise and accurate manner."
   optimized_prompt = "You are a helpful assistant. Answer questions concisely and accurately."
   ```

   불필요한 단어는 제거하고 핵심 지침에 집중합니다. 최적화된 프롬프트를 파일에 저장합니다.

### 최적화 테스트 및 검증

품질 저하 없이 토큰 사용량을 줄이려면 프롬프트 변경 내용을 테스트하는 것이 중요합니다.

1. `code token-count.py`를 실행하여 연습에 제공된 토큰 카운터 앱을 열고 확인합니다. 위의 예제에서 제공된 것과 다른 최적화된 프롬프트를 사용한 경우 이 앱에서도 사용할 수 있습니다.

1. `python token-count.py`를 사용하여 스크립트를 실행하고 토큰 수의 차이를 확인합니다. 최적화된 프롬프트가 여전히 고품질의 응답을 생성하는지 확인합니다.

## 사용자 상호 작용 분석

사용자가 앱과 상호 작용하는 방식을 이해하면 토큰 사용량을 늘리는 패턴을 식별하는 데 도움이 됩니다.

1. 사용자 프롬프트의 샘플 데이터 세트를 검토합니다.

    - **"*전쟁과 평화*의 음모를 요약해 주세요."**
    - **"고양이에 관한 흥미로운 사실에는 무엇이 있나요?"**
    - **"AI를 사용하여 공급망을 최적화하는 스타트업에 대한 자세한 비즈니스 계획을 작성해 주세요."**
    - **"'Hello, how are you?'를 프랑스어로 번역해 주세요."**
    - **"10살 어린이에게 양자 얽힘에 대해 설명해 주세요."**
    - **"공상 과학 단편 소설을 위한 창의적인 아이디어 10가지를 제안해 주세요."**

    각각에 대해 AI가 생성할 응답이 **짧은 답변**, **중간 길이의 답변**, **길고 복잡한 답변** 중 어느 유형에 해당하는지 확인합니다.

1. 분류를 검토합니다. 어떤 패턴을 알 수 있나요? 고려 사항:

    - **추상화** 수준(예: 창의적인 내용 및 사실 기반 내용)이 길이에 영향을 주나요?
    - **개방형 프롬프트**가 더 긴 경향이 있나요?
    - **지시의 복잡성**(예: "10살 어린이에게 설명하듯이")이 응답에 어떤 영향을 주나요?

1. 다음 명령을 입력하여 **optimize-prompt** 응용 프로그램을 실행합니다.

    ```
   python optimize-prompt.py
    ```

1. 위에 제공된 샘플 중 일부를 사용하여 분석을 확인합니다.
1. 이제 다음의 긴 형식 프롬프트를 사용하고 출력 결과를 검토합니다.

    ```
   Write a comprehensive overview of the history of artificial intelligence, including key milestones, major contributors, and the evolution of machine learning techniques from the 1950s to today.
    ```

1. 이 프롬프트를 다시 작성하여 다음을 수행합니다.

    - 범위 제한
    - 간결성에 대한 기대치 설정
    - 서식 또는 구조를 사용하여 응답 안내

1. 응답을 비교하여 더 간결한 답변을 얻었는지 확인합니다.

> **참고**: `token-count.py`를 사용하여 두 응답의 토큰 사용량을 비교할 수 있습니다.
<br>
<details>
<summary><b>다시 작성된 프롬프트의 예:</b></summary><br>
<p>"AI 역사에서 중요한 5가지 이정표를 글머리 기호로 요약해 주세요."</p>
</details>

## [**선택 사항**] 실제 시나리오에서 최적화 적용

1. 빠르고 정확한 답변을 제공해야 하는 고객 지원 챗봇을 구축한다고 상상해 보세요.
1. 최적화된 시스템 프롬프트와 템플릿을 챗봇의 코드에 통합합니다(*`optimize-prompt.py`를 시작점으로 사용할 수 있음*).
1. 다양한 사용자 쿼리로 챗봇을 테스트하여 효율적이고 효과적으로 응답하는지 확인합니다.

## 결론

프롬프트 최적화는 비용을 절감하고 생성형 AI 애플리케이션의 성능을 개선하기 위한 핵심 기술입니다. 프롬프트를 간결하게 작성하고, 템플릿을 활용하며, 사용자 상호 작용을 분석함으로써 더 효율적이고 확장 가능한 솔루션을 구축할 수 있습니다.

## 정리

Azure AI 서비스 탐색을 마친 경우 이 연습에서 만든 리소스를 삭제하여 불필요한 Azure 비용이 발생하지 않도록 해야 합니다.

1. Azure Portal이 포함된 브라우저 탭으로 돌아가서(또는 새 브라우저 탭에서 [Azure Portal](https://portal.azure.com?azure-portal=true)을 다시 열고) 이 연습에 사용된 리소스를 배포한 리소스 그룹의 콘텐츠를 확인합니다.
1. 도구 모음에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.
