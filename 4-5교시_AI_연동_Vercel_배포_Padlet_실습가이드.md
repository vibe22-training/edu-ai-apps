# 창의재단 교사연수 입문반 AI 연동 Vercel 배포 Padlet 제출 실습가이드

3교시에서 만든 퀴즈 생성기와 학습활동 추천 앱에 Microsoft Foundry를 연결합니다. 내 PC에서 결과를 확인한 다음 GitHub에 올리고, Vercel 주소에서도 같은 입력을 시험합니다. 마지막으로 저장소, 배포 주소, 현장 적용 체크리스트를 Padlet의 각 칸에 제출합니다.

| 실습 | 완료 기준 | 통합 교안 |
| --- | --- | --- |
| 로컬 AI 연동 | 퀴즈 5문항과 활동 3개가 카드로 표시되고, 빈 입력은 안내 후 중단 | 94, 99, 102–114쪽 |
| GitHub 저장소 | README와 앱 파일이 보이고 강사가 링크를 열 수 있음 | 121–125쪽 (원본 95–99쪽) |
| Vercel 배포 | 배포 주소에서 두 앱이 실행되고 짝의 기기에서도 열림 | 126–129, 131–133쪽 (원본 100–103, 105–107쪽) |
| Padlet 제출 | 각 칸에 저장소·배포 주소·체크리스트가 하나씩 보임 | 119, 125, 133, 137, 141쪽 |

[통합 슬라이드 대응표](./통합_슬라이드_대응표.csv)

## 1. 프로젝트와 연결 정보 준비

1. VS Code에서 3교시에 작업한 `edu-ai-apps` 폴더를 엽니다. 탐색기 맨 위에 `public` 폴더와 `시스템 프롬프트.txt`가 보이는지 확인합니다. `public` 안에는 `index.html`, `quiz.html`, `activity.html`, `style.css`가 있습니다.
2. 프로젝트 맨 위에 `.gitignore`를 만들고 아래 내용을 저장합니다. 왼쪽 **소스 제어**에 **리포지토리 초기화** 버튼이 보이면 지금 눌러 둡니다.

```gitignore
node_modules/
.env
.env.*
env_local.txt
.vercel/
```

3. `.gitignore`와 같은 위치에 **`.env.local`** 파일을 만듭니다. 아래는 파일 모양입니다. 세 줄의 값은 강사에게 받은 `env_local.txt`의 값으로 바꿉니다. `AI_DEPLOYMENT`에는 안내받은 배포명을 그대로 넣습니다.

```dotenv
AI_ENDPOINT=https://example-resource.openai.azure.com/openai/v1/
AI_API_KEY=REPLACE_WITH_CLASS_KEY
AI_DEPLOYMENT=example-deployment
```

실제 값은 선생님 PC의 `.env.local`과 배포 프로젝트의 환경변수에만 입력합니다. Copilot에 보여 줄 때는 변수 이름이나 예시 값을 사용합니다. `.env.local`이 `.env.local.txt`로 저장되지 않았는지 VS Code 탐색기에서 확인하세요.

## 2. Microsoft Foundry를 연결하고 내 PC에서 확인

브라우저가 내 프로젝트의 `/api/generate`로 입력을 보냅니다. 서버는 `.env.local`의 연결 정보로 Foundry에 요청하고, 브라우저는 돌아온 배열을 카드로 보여 줍니다. 퀴즈부터 연결해 확인하고 같은 서버 함수에 활동 추천을 붙입니다. [Microsoft Foundry Responses API](https://learn.microsoft.com/en-us/azure/foundry/openai/how-to/responses)

### 2-1. Copilot에 서버 함수 만들기 요청

VS Code Copilot Chat에 아래 내용을 붙여넣고 제안된 변경 목록을 확인합니다. 2~3교시에 작성한 `design-quiz.md`와 `design-activity.md`가 있으면 함께 참조합니다.

```text
교안의 퀴즈 생성기와 학습활동 추천 앱에 공용 서버를 붙여 줘.
docs/design-api.md에 입출력을 적고, api/generate.js와 로컬 실행용 server.js를 만들어 줘.
기존 public의 HTML과 CSS는 그대로 유지해 줘.

- POST /api/generate가 JSON을 받게 해 줘.
- 퀴즈 입력은 {type: 'quiz', subject, unit, difficulty}로 정해 줘.
- 추천 입력은 {type: 'activity', level, unit, objective}로 정해 줘.
- 필수 입력에서 앞뒤 공백을 제거하고, 비어 있거나 type이 틀리면 400을 반환해 줘.
- OpenAI SDK의 responses.create에 AI_ENDPOINT를 baseURL, AI_API_KEY를 apiKey,
  AI_DEPLOYMENT를 model로 전달해 줘. 환경변수는 process.env에서 읽어 줘.
- 프로젝트 루트의 ‘시스템 프롬프트.txt’를 instructions로 읽고,
  사용자 입력의 첫 줄에 type: quiz 또는 type: activity를 넣어 줘.
- response.output_text를 JSON으로 파싱해 줘. 성공 응답은 그대로 JSON 배열로 반환하고,
  퀴즈는 문항 5개, 추천은 활동 3개인지와 제공 규칙의 각 필드를 확인해 줘.
  형식 오류는 한 번만 다시 요청하고, 계속 실패하면 화면에 보여 줄 짧은 오류를 반환해 줘.
- 모델 호출에는 store: false와 적당한 출력 토큰 상한을 넣어 줘.
  SDK 자동 재시도는 끄고, 오류 로그에는 단계와 상태 코드만 남겨 줘.
- api/generate.js는 Vercel Node.js 핸들러인 export default async function (req, res)로 작성해 줘.
  로컬 server.js도 같은 핸들러를 사용하게 해 줘.
- server.js는 dotenv로 .env.local을 읽고 Express로 public과 POST /api/generate를 제공해 줘.
  포트는 3001로 해 줘. 서버가 시작된 뒤 요청을 받을 때 환경변수를 읽어 줘.
  package.json은 ES 모듈 설정(type: module)을 사용해 줘.
```

### 2-2. 화면을 공용 서버에 연결

아래 프롬프트로 각 화면의 목업 결과를 실제 응답으로 바꿉니다. 기존 양식이 없으면 먼저 입력란과 결과 영역을 추가하도록 요청합니다.

```text
기존 화면을 유지하면서 public/quiz.html의 문항 만들기 버튼을 POST /api/generate에 연결해 줘.
Content-Type은 application/json을 쓰고 {type: 'quiz', subject, unit, difficulty}를 보내 줘.
성공하면 받은 배열을 기존 결과 카드에 표시해 줘. 결과 문구는 textContent로 넣어 줘.
빈 입력이면 화면에 안내하고 API 호출을 멈춰 줘. 요청 중에는 버튼을 잠그고,
실패하면 입력을 유지하면서 다시 시도할 수 있게 해 줘.
퀴즈가 확인되면 public/activity.html에도 같은 방식으로 붙여 줘.
추천은 {type: 'activity', level, unit, objective}를 보내고 결과 카드 3개를 표시해 줘.
제출 처리에서 기존 목업 데이터 대신 서버 응답을 사용해 줘.
```

### 2-3. 실행하고 두 경우를 확인

VS Code에서 **터미널 → 새 터미널**을 엽니다. 이전 `npx serve public`이 켜져 있으면 해당 터미널에서 `Ctrl+C`를 누릅니다. 프로젝트 맨 위의 `package.json`이 보이는 폴더에서 실행합니다.

```bash
npm install openai express dotenv
node server.js
```

브라우저에서 `http://localhost:3001`을 엽니다. 서버를 켠 터미널은 확인이 끝날 때까지 열어 둡니다.

| 화면 | 정상 입력 예시 | 기대 결과 | 일부 항목을 비운 경우 |
| --- | --- | --- | --- |
| 퀴즈 생성기 | 교과 `수학` / 단원 `분수` / 난이도 `쉬움` | 보기 4개가 있는 문항 카드 5개. 정답 번호와 해설 한 개를 직접 확인 | 단원 안내가 나타나고 생성 요청이 멈춤 |
| 학습활동 추천 | 학생 수준 `초등 3학년` / 단원명 `분수` / 학습 목표 `분수로 전체와 부분의 관계를 설명한다` | 활동 카드 3개. 준비물·소요 시간·진행 순서가 보임 | 학습 목표 안내가 나타나고 추천 요청이 멈춤 |

- [ ] 정상 입력의 결과가 실제 응답이며, 단원이나 학습 목표를 바꿨을 때 새 주제에 맞는 결과가 나온다.
- [ ] 빈 입력 안내가 표시되고 입력을 고친 뒤 다시 실행할 수 있다.

## 3. GitHub에 올리고 첫 번째 Padlet 제출

### 3-1. 배포 설정 파일 저장

프로젝트 맨 위에 `vercel.json`을 추가합니다. 기존 파일이 있으면 두 항목을 반영합니다. HTML은 `public`에서 제공하고 서버 함수에는 `시스템 프롬프트.txt`도 함께 배포합니다. [Vercel 프로젝트 설정](https://vercel.com/docs/project-configuration/vercel-json#functions)

```json
{
  "outputDirectory": "public",
  "functions": {
    "api/generate.js": {
      "includeFiles": "시스템 프롬프트.txt"
    }
  }
}
```

### 3-2. 저장소 공개 범위와 파일 확인

1. `README.md`에 작품 소개, 로컬 실행 명령, 필요한 환경변수 **이름**, 테스트에 쓴 가상 입력을 적습니다.
2. 아래 두 줄을 터미널에서 실행합니다. 첫 줄에서 `.env.local`이 표시되고, 두 번째 줄은 비어 있어야 합니다. 키 파일을 GitHub에 올린 적이 있다면 키 교체를 위해 강사에게 알립니다. [GitHub Git 제외 설정](https://docs.github.com/en/get-started/git-basics/ignoring-files)

```bash
git check-ignore .env.local
git ls-files -- .env.local env_local.txt
```

3. **소스 제어**에서 변경 목록을 보고 커밋합니다. 처음이라면 **Publish to GitHub**를 실행해 연수 계정에 저장소를 만듭니다. 이미 연결되었다면 **Sync Changes** 또는 **Push**를 누릅니다. [VS Code 소스 제어 안내](https://code.visualstudio.com/docs/sourcecontrol/quickstart#step-4-sync-with-the-server)
4. GitHub에서 README, `public/`, `api/generate.js`, `package.json`, `시스템 프롬프트.txt`, `vercel.json`이 보이는지 확인합니다. `.env.local`과 `env_local.txt`가 저장소 목록에 없는지 확인합니다. 비공개 저장소는 안내받은 강사 계정에 접근 권한을 부여하고, 강사가 실제로 열 수 있는지 확인합니다.
5. Padlet의 **산출물(Github 링크)** 칸에 `제출 1` 글 틀로 올립니다.

## 4. Vercel에 배포하고 두 번째 Padlet 제출

### 4-1. 저장소 가져오기와 환경변수 입력

1. Vercel에서 GitHub 계정으로 로그인한 뒤 **Add New → Project → Import**에서 방금 올린 저장소를 선택합니다. 목록에 없으면 Vercel이 연결한 GitHub 계정과 해당 저장소 권한부터 확인합니다. [Vercel의 GitHub 연결 안내](https://vercel.com/docs/git/vercel-for-github#missing-git-repository)
2. **Framework Preset → Other**를 선택합니다. **Root Directory**는 `public`과 `api`가 함께 있는 폴더입니다. 두 폴더가 저장소 맨 위에 있으면 기본값(`.`)을 유지합니다. HTML을 따로 빌드할 명령 없이 진행합니다. `vercel.json`의 Output Directory는 `public`입니다. [Vercel 빌드 설정](https://vercel.com/docs/builds/configure-a-build#output-directory)
3. **Environment Variables**에서 `AI_ENDPOINT`, `AI_API_KEY`, `AI_DEPLOYMENT`를 각각 등록합니다. 값만 붙여넣고 이름이나 `=`는 같이 넣지 않습니다. 공개할 배포가 사용하는 **Production** 환경에 적용한 뒤 **Deploy**를 누릅니다.
4. 배포 후 값을 추가하거나 바꿨다면 **Deployments → 해당 배포 → Redeploy**를 실행합니다. 변수 변경은 새 배포부터 적용됩니다. [Vercel 환경변수 안내](https://vercel.com/docs/environment-variables/managing-environment-variables)

### 4-2. 다른 기기에서 결과 확인

1. Vercel 프로젝트의 **Domains**에서 제출할 **Production** 주소를 엽니다. PC와 휴대전화에서 확인합니다. 휴대전화를 쓸 수 없으면 다른 PC나 시크릿 창을 사용합니다.
2. 앞의 정상 입력과 빈 입력을 두 앱에서 다시 시험합니다. 정상 입력에서는 카드가 나와야 하고, 빈 입력에서는 안내가 보여야 합니다. 정상 결과 화면을 캡처합니다.
3. Padlet **산출물(Vercel 배포링크)** 칸에 `제출 2` 글 틀과 캡처를 올립니다. 짝의 주소를 열어 같은 테스트를 한 번 해 보고 잘 된 점이나 막힌 점을 한 줄로 전달합니다.

## 5. Padlet에 제출할 글 틀

해당 칸의 `+`를 누르고 아래 글 틀을 붙여넣습니다. 이름이 같은 칸인지 확인하고 **게시(Publish)**를 누릅니다. 게시한 뒤 그 칸에서 내 글과 링크를 한 번 더 열어 봅니다. [연수 Padlet 열기](https://padlet.com/elixirr/ai-s025io7rg4hzn60pkqos) · [Padlet 게시 방법](https://padlet.help/l/en/article/7hwq09wbop-jak-p-id-m-p-sp-vek-do-padletu)

### 제출 1 산출물(Github 링크)

저장소 파일과 강사 접근을 확인한 직후 (통합 125쪽).

```text
작품명:
GitHub 저장소 URL:
README와 두 앱 확인: 완료 / 확인 중
비밀 설정 파일 제외: 완료 / 확인 중
강사 접근 확인: 완료 / 확인 요청
```

### 제출 2 산출물(Vercel 배포링크)

배포 주소에서 앱을 실행한 직후 (통합 133쪽). 결과 캡처는 이 글에 파일로 첨부합니다.

```text
작품명:
Vercel 배포 URL:
한 줄 소개:
퀴즈 생성기 실행: 완료 / 확인 중
학습활동 추천 실행: 완료 / 확인 중
결과 화면: 파일 첨부
```

### 제출 3 산출물(현장 적용 체크리스트)

짝의 피드백을 받은 뒤 수업 마무리 전에 게시합니다 (통합 137·141쪽).

```text
적용할 과목·단원:
가상 자료 또는 익명 자료 사용 계획:
교사가 확인할 것(예: 정답·학년 수준·수업 목표):
동료 피드백에서 반영할 점과 공유 방법:
다음 학기에 시도할 활동 한 가지:
```

예시로 초등 3학년 분수 단원에서 가상 입력으로 형성평가 초안을 만들 수 있습니다. 정답과 해설은 교사가 검산합니다. 짝에게 보기가 길다는 의견을 받았다면 보기 길이를 줄여 한 차시에서 시험하고, 학년 협의에서 결과를 공유합니다.

## 막혔을 때 확인할 곳

| 보이는 문제 | 바로 할 일 |
| --- | --- |
| PowerShell에서 `npm.ps1` 실행이 차단됨 | VS Code 터미널 메뉴에서 **Command Prompt**를 열거나 `npm.cmd install openai express dotenv`로 실행 |
| 내 PC에서 화면만 열리고 생성이 안 됨 | 주소가 `localhost:3001`인지, `node server.js` 터미널이 켜져 있는지, `.env.local`이 프로젝트 맨 위에 있는지 확인 |
| 배포 주소만 생성이 안 됨 | Vercel Runtime Logs에서 `/api/generate` 호출을 확인. 화면 경로 404는 폴더·함수 이름, Foundry 단계의 401은 키, Foundry 단계의 404는 엔드포인트와 배포명, 파일 읽기 오류는 `includeFiles`부터 확인 |
| 짝의 브라우저에서 Vercel 로그인이 필요함 | Production 주소인지 확인하고, 수업에서 허용된 공유 범위와 배포 보호 상태를 강사와 확인 |
| 429 또는 응답 지연 | 짝과 시간차를 두고 요청. 브라우저 안내가 뜨면 잠시 기다렸다 다시 한 번 실행 |
| Vercel 로그인·배포 또는 API 확인이 늦어짐 | 로컬 결과를 보존하고 GitHub와 체크리스트를 먼저 제출. Vercel 칸에는 `배포 확인 중`과 막힌 단계만 기록하고, 성공 확인 뒤 URL과 화면을 추가 |

연수 종료 전 통합 141쪽에서 세 게시물을 확인합니다. 연수용 키 관리는 통합 144쪽 안내를 따릅니다.
