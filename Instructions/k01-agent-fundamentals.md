---
lab:
    title: 'AI Agent 개발 살펴보기'
    description: 'Azure AI Foundry 포털에서 Azure AI Agent 서비스를 탐색하여 AI 에이전트 개발의 첫걸음을 내딛습니다.'
---

# AI Agent 개발 살펴보기

이 실습에서는 Azure AI Foundry 포털의 Azure AI Agent 서비스를 사용하여 직원들의 경비 청구를 돕는 간단한 AI 에이전트를 만듭니다.

이 실습을 완료하는 데 약 **30**분이 소요됩니다.

> **참고**: 이 실습에서 사용되는 일부 기술은 미리 보기(preview) 상태이거나 활발히 개발 중입니다. 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

## Azure AI Foundry 프로젝트 및 에이전트 만들기

먼저 Azure AI Foundry 프로젝트를 만드는 것부터 시작하겠습니다.

1.  웹 브라우저에서 [Azure AI Foundry 포털](https://ai.azure.com) (`https://ai.azure.com`)을 열고 Azure 자격 증명으로 로그인합니다. 처음 로그인할 때 열리는 팁이나 빠른 시작 창은 닫아주세요. 필요한 경우 왼쪽 상단의 **Azure AI Foundry** 로고를 사용하여 홈페이지로 이동합니다. 홈페이지는 아래 이미지와 유사합니다(열려 있다면 **Help** 창을 닫으세요):

    ![Screenshot of Azure AI Foundry portal.](./Media/ai-foundry-home.png)

1.  홈페이지에서 **Create an agent**를 선택합니다.
2.  프로젝트를 만들라는 메시지가 나타나면 프로젝트에 유효한 이름을 입력합니다.
3.  **Advanced options**를 확장하고 다음 설정을 지정합니다:
    -   **Azure AI Foundry resource**: *Azure AI Foundry 리소스에 대한 유효한 이름*
    -   **Subscription**: *사용 중인 Azure 구독*
    -   **Resource group**: *기존 리소스 그룹을 선택하거나 새로 만듭니다.* (리소스 그룹은 관련된 Azure 리소스들을 담는 컨테이너 역할을 합니다.)
    -   **Region**: ***AI Foundry recommended*** 지역 중 하나를 선택합니다.*

    > \* 일부 Azure AI 리소스는 지역별 모델 할당량(quota)에 의해 제약을 받습니다. 실습 후반부에 할당량 제한을 초과하는 경우, 다른 지역에 다른 리소스를 만들어야 할 수도 있습니다.

1.  **Create**를 선택하고 프로젝트가 생성될 때까지 기다립니다.
1.  메시지가 표시되면, **Global standard** 또는 **Standard** 배포 유형(할당량 가용성에 따라 다름)을 사용하여 **gpt-4o** 모델을 배포하고, 배포 세부 정보를 사용자 지정하여 **Tokens per minute rate limit**를 50K로 설정합니다(50K 미만인 경우 사용 가능한 최댓값으로 설정).

    > **참고**: 분당 토큰 처리량(TPM)을 줄이면 사용 중인 구독에서 사용 가능한 할당량을 과도하게 사용하는 것을 방지할 수 있습니다. 50,000 TPM은 이 실습의 데이터를 처리하기에 충분합니다. 사용 가능한 할당량이 이보다 낮더라도 실습을 완료할 수는 있지만, 속도 제한을 초과하면 오류가 발생할 수 있습니다.

1.  프로젝트가 생성되면, 모델을 선택하거나 배포할 수 있도록 Agents playground가 자동으로 열립니다:

    ![Screenshot of a Azure AI Foundry project Agents playground.](./Media/ai-foundry-agents-playground.png)

    >**참고**: Agent와 프로젝트를 생성할 때 GPT-4o 기본 모델이 자동으로 배포됩니다.

기본 이름으로 생성된 에이전트와 기본 모델 배포를 확인할 수 있습니다.

## 에이전트 만들기

이제 모델이 배포되었으니 AI 에이전트를 만들 준비가 되었습니다. 이 실습에서는 회사 경비 정책에 대한 질문에 답변하는 간단한 에이전트를 만듭니다. 경비 정책 문서를 다운로드하여 에이전트의 *근거 데이터(grounding data)*로 사용할 것입니다.

**근거 데이터(Grounding Data)란?**
> AI 모델이 답변을 생성할 때 기반으로 삼는 특정 정보 소스를 의미합니다. 이를 통해 AI는 신뢰할 수 있는 특정 문서나 데이터베이스 내에서만 정보를 찾아 답변하게 되므로, 부정확하거나 관련 없는 정보를 생성하는 '환각(hallucination)' 현상을 크게 줄일 수 있습니다. 이 실습에서는 경비 정책 문서가 바로 근거 데이터가 됩니다.

1.  다른 브라우저 탭을 열고, `https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/01-agent-fundamentals/Expenses_Policy.docx`에서 [Expenses_policy.docx](https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/01-agent-fundamentals/Expenses_Policy.docx)를 다운로드하여 로컬에 저장합니다. 이 문서에는 가상의 Contoso 회사의 경비 정책에 대한 세부 정보가 포함되어 있습니다.
2.  Foundry Agents playground가 있는 브라우저 탭으로 돌아가 **Setup** 창을 찾습니다(채팅 창의 옆이나 아래에 있을 수 있습니다).
3.  **Agent name**을 `ExpensesAgent`로 설정하고, 이전에 만든 gpt-4o 모델 배포가 선택되었는지 확인한 후, **Instructions**에 다음 내용을 입력합니다:


   ```prompt
   You are an AI assistant for corporate expenses.
   You answer questions about expenses based on the expenses policy data.
   If a user wants to submit an expense claim, you get their email address, a description of the claim, and the amount to be claimed and write the claim details to a text file that the user can download.
 ```
    이 지침은 에이전트의 역할과 행동 방식을 정의하는 핵심적인 부분입니다. 에이전트는 이 지침에 따라 경비 정책 보조원으로서 작동하게 됩니다.

    ![Screenshot of the AI agent setup page in Azure AI Foundry portal.](./Media/ai-agent-setup.png)

4.  **Setup** 창 아래쪽의 **Knowledge** 헤더 옆에 있는 **+ Add**를 선택합니다. 그런 다음 **Add knowledge** 대화 상자에서 **Files**를 선택합니다.
5.  **Adding files** 대화 상자에서 `Expenses_Vector_Store`라는 이름의 새 벡터 저장소(vector store)를 만들고, 이전에 다운로드한 로컬 파일 **Expenses_policy.docx**를 업로드하여 저장합니다.

**벡터 저장소(Vector Store)란?**
> 텍스트와 같은 데이터를 수학적인 벡터 형태로 변환하여 저장하는 특수한 데이터베이스입니다. 이를 통해 AI는 단어의 의미적 유사성을 기반으로 매우 빠르고 정확하게 관련 정보를 검색할 수 있습니다. 에이전트는 사용자의 질문과 가장 관련성이 높은 정책 내용을 이 벡터 저장소에서 찾게 됩니다.

6.  **Setup** 창의 **Knowledge** 섹션에서 **Expenses_Vector_Store**가 목록에 표시되고 1개의 파일이 포함되어 있는지 확인합니다.
7.  **Knowledge** 섹션 아래의 **Actions** 옆에 있는 **+ Add**를 선택합니다. **Add action** 대화 상자에서 **Code interpreter**를 선택한 다음 **Save**를 선택합니다(코드 인터프리터를 위해 파일을 업로드할 필요는 없습니다).

    에이전트는 업로드한 문서를 지식 소스로 사용하여 응답의 *근거*를 마련합니다(즉, 이 문서의 내용에 기반하여 질문에 답변합니다). 또한, 필요에 따라 자체적으로 Python 코드를 생성하고 실행하여 작업을 수행하기 위해 코드 인터프리터 도구를 사용하게 됩니다.

**코드 인터프리터(Code Interpreter)란?**
> 에이전트에게 코드를 작성하고 실행할 수 있는 능력을 부여하는 강력한 도구입니다. 이를 통해 에이전트는 단순한 텍스트 응답을 넘어 파일 생성, 데이터 계산, 그래프 그리기 등 복잡한 작업을 동적으로 수행할 수 있습니다. 이 실습에서는 경비 청구 내역을 텍스트 파일로 만드는 데 사용됩니다.

## 에이전트 테스트하기

이제 에이전트를 만들었으니, playground 채팅에서 테스트해볼 수 있습니다.

1.  playground 채팅 입력창에 `What's the maximum I can claim for meals?` (식비로 청구할 수 있는 최대 금액은 얼마인가요?) 라고 입력하고 에이전트의 응답을 검토합니다. 응답은 에이전트 설정에 지식으로 추가한 경비 정책 문서의 정보를 기반으로 해야 합니다.

    > **참고**: 속도 제한 초과로 인해 에이전트가 응답하지 못하는 경우, 몇 초간 기다렸다가 다시 시도하세요. 구독에 사용 가능한 할당량이 부족하면 모델이 응답하지 못할 수 있습니다. 문제가 지속되면 **Models + endpoints** 페이지에서 모델의 할당량을 늘려보세요.

2.  다음 후속 프롬프트를 시도해보세요: `I'd like to submit a claim for a meal.` (식비 청구를 제출하고 싶습니다.) 그리고 응답을 검토합니다. 에이전트는 청구 제출에 필요한 정보를 물어봐야 합니다.
3.  에이전트에게 이메일 주소를 제공합니다. 예를 들어, `fred@contoso.com`과 같이 입력합니다. 에이전트는 응답을 확인하고 경비 청구에 필요한 나머지 정보(설명 및 금액)를 요청해야 합니다.
4.  청구 내용과 금액을 설명하는 프롬프트를 제출합니다. 예를 들어, `Breakfast cost me $20` (아침 식사 비용으로 20달러를 썼습니다.) 라고 입력합니다.
5.  에이전트는 코드 인터프리터를 사용하여 경비 청구 텍스트 파일을 준비하고, 다운로드할 수 있는 링크를 제공해야 합니다.

    ![Screenshot of the Agent Playground in Azure AI Foundry portal.](./Media/ai-agent-playground.png)

6.  텍스트 문서를 다운로드하여 열고 경비 청구 세부 정보를 확인합니다.

## 정리

실습을 마쳤으므로, 불필요한 리소스 사용을 방지하기 위해 생성한 클라우드 리소스를 삭제해야 합니다.

1.  [Azure portal](https://portal.azure.com) (`https://portal.azure.com`)을 열고 이 실습에서 사용한 허브 리소스를 배포한 리소스 그룹의 내용을 확인합니다.
2.  도구 모음에서 **Delete resource group**을 선택합니다.
3.  리소스 그룹 이름을 입력하고 삭제를 확인합니다.
