---
title: 대울경 AI Workflow 해커톤 - Studio Agent Playbook
eyebrow: Daegu · Ulsan · Gyeongbuk University X Upstage · AI Workflow Hackathon
lede: Upstage Studio에서 문서 처리 에이전트를 직접 만들고 실행해 보며, 해커톤 전에 Studio를 미리 경험하는 과정을 안내합니다.
---

# 0. 시작하기 전에

이 해커톤에서 팀은 실무 문서 업무가 실제로 돌아가는 Studio 워크플로우를 만듭니다.

이 Playbook은 예시 에이전트 3개를 Studio에서 직접 만들어 보며 Studio 사용법을 익히는 데 집중합니다. 예시는 Parse, Classify, Extract 같은 가장 기초적인 노드만 씁니다.

## 0-1. 해커톤에서 만드는 것

팀은 실무 문서 업무 하나를 골라 Studio 워크플로우로 처리합니다. 결과가 맞는지 확인할 수 있도록 테스트 데이터셋과 정답지를 함께 만듭니다.

```TEXT
실무 문서 입력 → Studio 워크플로우(Parse, Classify, Extract 등) → 결과 JSON → 정답지와 비교
```

| 산출물 | 내용 |
| --- | --- |
| Studio 워크플로우 | 업무 흐름에 맞게 노드를 조합한 에이전트 |
| 테스트 데이터셋 | 실무에서 실제로 들어오는 문서와 예외 케이스 |
| 정답지 | 문서마다 워크플로우가 내야 할 정답 |
| 설계서 | 다룰 업무, 출력 키, 노드 구성과 그렇게 짠 이유 |

## 0-2. 사전 준비

| 항목 | 용도 |
| --- | --- |
| Upstage Console 계정과 API Key | Studio 접속, 코드로 에이전트 실행 |
| Upstage Studio 접속 | 에이전트 생성, 실행, 설정 내보내기 |
| 스타터 코드 | 예시 문서, 예시 에이전트 설정, 실행 코드 |
| Python 3.11 이상 | 코드로 실행할 때만 필요합니다. 선택입니다 |

에이전트는 Studio 화면에서 만듭니다. 1장부터 5장까지는 코드 없이 진행합니다.

Studio의 전체 기능은 [Upstage Studio 사용 설명서](https://console.upstage.ai/ko/docs/studio)에서 확인합니다. 이 Playbook에 없는 기능과 최신 화면은 사용 설명서를 기준으로 합니다.

스타터 코드는 GitHub 저장소에서 받습니다.

[스타터 코드 저장소](https://github.com/SuWanKim-code/daeul-hackathon-studio-starter)

```bash
git clone https://github.com/SuWanKim-code/daeul-hackathon-studio-starter.git
```

스타터 코드의 `data/` 폴더에 예시 문서 3개가 있습니다. 2장부터 4장까지 이 문서를 업로드해 씁니다.

| 파일 | 내용 |
| --- | --- |
| `data/Resume.pdf` | 가상 지원자의 이력서 |
| `data/JobDescription.pdf` | 채용공고 |
| `data/Matching_Input.pdf` | 이력서와 채용공고 추출 결과를 한 문서로 합친 비교용 문서 |

화면은 Studio 이전 버전 기준입니다. 메뉴 위치와 이름이 조금 다를 수 있습니다.

# 1. Studio 이해하기

Studio 에이전트는 노드를 이어 붙인 문서 처리 파이프라인입니다. 노드마다 하나의 역할을 맡습니다.

## 1-1. 노드 종류

Studio 워크플로우는 아래 노드를 조합해 만듭니다. 예시 에이전트는 이 가운데 Parse, Classify, Extract만 써서 Studio를 경험해 보는 구성입니다.

| 노드 | 하는 일 | 예 |
| --- | --- | --- |
| Parse | PDF, 이미지, 오피스 문서, HWP 등을 텍스트로 바꾸면서 제목, 문단, 표 같은 구조를 함께 읽습니다. 모든 워크플로우의 시작 노드입니다 | 입력이 문서 파일인 모든 경우 |
| Classify | 문서를 정해 둔 문서종으로 나누는 분기점입니다. 한 파일에 여러 문서가 섞여 있으면 나눈 뒤 분류할 수 있습니다 | 이력서, 자기소개서, 포트폴리오를 나눠 각각 처리 |
| Extract | 스키마에 정의한 항목만 구조화된 형태로 뽑습니다. 항목 설명이 정확도를 가장 크게 좌우합니다 | 이력서에서 이름, 경력, 기술 추출 |
| Instruct | 앞 노드 결과를 받아 요약, 변환, 검증, 판단을 합니다. 정해 둔 선택지 중 하나를 골라 뒤 노드를 나눌 수도 있습니다 | 합격, 보류, 불합격 판정과 사유 작성 |
| Validate | 추출한 값이 정해 둔 조건을 만족하는지 점검합니다. 규칙마다 필수와 선택을 정하고, 결과는 초록, 노랑, 빨강으로 판정합니다 | 경력 3년 이상, 연락처 기재를 필수 규칙으로 두고 지원자마다 판정 |
| Merge | 나뉜 문서들의 결과를 하나로 합쳐 다음 노드에 넘깁니다 | 이력서, 자기소개서, 포트폴리오에서 뽑은 결과를 지원자 한 명의 데이터로 합침 |
| Match | 추출한 값과 일치하는 기록을 찾아 연결합니다. Extract 바로 뒤에 연결합니다 | 청구서와 발주서 대조, 지원자 명단 엑셀과 이력서 추출 결과 대조 |
| Export, Import | Export는 실행 결과를 컬렉션에 저장하고, Import는 컬렉션에 쌓인 결과를 가져와 이어서 처리합니다. 캔버스 양 끝의 시작, 종료 표지에서 설정합니다 | 매일 영수증 결과를 Export하고 월말 정산 워크플로우가 Import해 집계 |

매번 같은 기준으로 걸러야 하는 판정은 Validate, 사람처럼 판단이 필요한 일은 Instruct가 맡습니다. 나뉜 경로의 값을 함께 비교하려면 Merge로 먼저 합칩니다.

노드별 설정과 주의 사항은 [Upstage Studio 사용 설명서](https://console.upstage.ai/ko/docs/studio)에서 자세히 확인합니다. 노드 설정은 `노드 설정` 페이지에 있습니다.

Studio가 잘 푸는 일은 사람이 문서를 읽고, 옮겨 적고, 분류하고, 비교하는 반복 작업입니다. 문서가 없는 예측 문제나 실시간 검색은 Studio만으로 풀기 어렵습니다.

## 1-2. 예시 에이전트 3개

이력서 1차 검토 업무를 예시로 에이전트 3개를 만듭니다. 앞의 두 개는 추출, 마지막 하나는 분류 후 평가입니다.

| 에이전트 | 입력 문서 | 노드 구성 | 작업 유형 | 난이도 |
| --- | --- | --- | --- | --- |
| Resume Agent | `data/Resume.pdf` | Parse, Extract | 문서 정보 추출 | 기본 |
| JD Agent | `data/JobDescription.pdf` | Parse, Extract | 문서 정보 추출 | 기본 |
| Matching Agent | `data/Matching_Input.pdf` | Parse, Classify, Extract 3개 | 문서 분류 후 정보 추출 | 도전 |

세 에이전트는 같은 순서로 만듭니다. 홈 입력창에 업무를 설명해 채팅으로 만드는 방법도 있습니다. 이 Playbook은 노드 구성을 직접 정하는 방법으로 진행합니다.

```TEXT
워크플로우 만들기 → 작업 유형 선택 → 이름 지정 → 문서 업로드와 자동 설정 → 노드 확인 → 스키마 설계 → 실행 → 결과 JSON 확인
```

스키마 항목 이름의 기준은 스타터 코드의 `agent_configs/` 폴더에 있는 설정 파일입니다. 실행 결과 예시는 `examples/` 폴더에 있습니다.

# 2. Resume Agent 만들기

Resume Agent는 이력서 PDF에서 이름, 연락처, 기술, 경력, 프로젝트, 학력을 뽑아 JSON으로 만듭니다.

## 2-1. 만들기

1. 사이드바의 `AI 워크플로우` 에서 `워크플로우 만들기` 를 선택합니다.
2. 작업 유형 화면에서 `문서 정보 추출` 을 선택하고 `만들기` 를 선택합니다.
3. 에이전트 이름을 `Resume Agent` 로 바꿉니다.
4. `업로드` 로 `data/Resume.pdf` 를 올리고 `자동 설정` 을 선택합니다. Studio가 문서를 분석해 Parse 노드와 기본 Extract 스키마를 만듭니다.
5. 에이전트 편집기에서 `변환` 노드와 `Extract-1` 노드가 연결되어 있는지 확인합니다. Parse 노드는 따로 설정하지 않습니다.

![워크플로우 만들기 화면](media/09.jpg)

![문서 정보 추출 작업 유형 선택](media/10.jpg)

![에이전트 이름 지정](media/12.jpg)

![이력서 업로드](media/13.jpg)

![에이전트 자동 설정 선택](media/14.jpg)

![에이전트 편집기](media/15.jpg)

## 2-2. Extract 스키마

`Extract-1` 노드를 선택하고 오른쪽 패널에서 항목을 아래 표와 같이 맞춥니다. 자동 설정이 만든 항목은 이름이 다를 수 있으니 표 기준으로 수정하고 추가합니다. 유형은 모두 String, 목록은 모두 켭니다.

| 항목명 | 설명 |
| --- | --- |
| `name` | 지원자의 이름 |
| `email` | 이메일주소 |
| `phone` | 휴대폰번호 |
| `location` | 거주 지역 |
| `position` | 지원 직무 |
| `summary` | 이력서 전체를 2~3문장으로 요약한 내용 |
| `skills` | 기술 스택목록 |
| `company` | 경력 회사명 |
| `career_position` | 경력 직무 |
| `career_period` | 근무 기간 |
| `career_description` | 주요 업무 요약 |
| `project_name` | 프로젝트명 |
| `project_period` | 프로젝트 수행 기간 |
| `project_description` | 프로젝트 설명 |
| `project_skills` | 프로젝트에서 사용한 기술 목록 |
| `school` | 학교명 |
| `major` | 전공명 |
| `education_period` | 재학 기간 |
| `certifications` | 자격증 목록 |
| `core_competencies` | 핵심역량 |

![Extract 스키마 설정](media/16.jpg)

여러 개가 나올 수 있는 정보는 목록으로 정의합니다. 하나의 문자열로 묶으면 나중에 비교하기 어렵습니다.

설명 칸은 AI에게 주는 힌트입니다. 기술명은 대표 명칭으로 통일하고, 날짜 형식을 정하고, 핵심 역량은 3~6개로 제한하도록 적으면 결과 품질이 좋아집니다.

## 2-3. 실행과 확인

오른쪽 위 `실행` 버튼을 선택합니다. 결과 패널의 `JSON` 탭에서 스키마대로 채워진 결과를 확인합니다.

![Resume Agent 실행 결과](media/18.jpg)

- 이름이 `Resume Agent` 이고 Parse와 `Extract-1` 노드가 연결되어 있습니다.
- `Extract-1` 항목 20개가 표와 이름이 같고 모두 목록으로 설정되어 있습니다.
- 실행 결과 JSON에서 `name`, `skills`, `company` 가 채워졌습니다.

# 3. JD Agent 만들기

JD Agent는 채용공고 PDF에서 회사명, 모집 직무, 담당 업무, 필수 기술, 우대 기술, 자격 요건을 뽑습니다. 만드는 과정은 2장과 같고 스키마만 다릅니다.

## 3-1. 만들기

1. `워크플로우 만들기` 에서 `문서 정보 추출` 을 선택하고, 이름을 `JD Agent` 로 바꿉니다.
2. `data/JobDescription.pdf` 를 업로드하고 `자동 설정` 을 선택합니다.
3. 편집기에 Parse와 `Extract-1` 노드가 만들어졌는지 확인합니다.

![채용공고 업로드](media/22.jpg)

![JD Agent 편집기](media/24.jpg)

## 3-2. Extract 스키마

회사명, 직무처럼 하나뿐인 값은 목록을 끄고, 담당 업무와 기술처럼 여러 개인 값은 목록을 켭니다. 유형은 모두 String입니다.

| 항목명 | 설명 | 목록 |
| --- | --- | --- |
| `company_name` | 회사명 | - |
| `position` | 모집직무명 | - |
| `responsibilities` | 담당업무 | 켬 |
| `required_skills` | 필수기술 | 켬 |
| `preferred_skills` | 우대기술 | 켬 |
| `qualifications` | 자격요건 | 켬 |
| `preferred_qualifications` | 우대사항 | 켬 |
| `experience_required` | 요구경력 | - |
| `education_required` | 요구 학력 | - |
| `employment_type` | 고용형태 | - |
| `location` | 근무위치 | - |

![JD Extract 스키마](media/25.jpg)

## 3-3. 실행과 확인

`실행` 을 선택하고 `JSON` 탭에서 `required_skills` 가 항목별 배열로 들어왔는지 확인합니다.

![JD Agent 실행 결과](media/26.jpg)

- 이름이 `JD Agent` 이고 Parse와 `Extract-1` 노드가 연결되어 있습니다.
- 항목 11개의 이름과 목록 설정이 표와 같습니다.
- 실행 결과에서 `required_skills`, `preferred_skills` 가 배열로 나옵니다.

# 4. Matching Agent 만들기

Matching Agent는 이력서와 채용공고를 비교해 적합도를 평가합니다. 앞의 두 에이전트가 추출이라면 이 에이전트는 판단입니다. 그래서 Parse와 Extract 사이에 Classify 노드가 들어갑니다.

Classify 노드가 문서를 High, Medium, Low Match 중 하나로 분류하고, 분류마다 연결된 Extract 노드가 실행됩니다. 세 Extract 노드는 같은 스키마를 씁니다.

## 4-1. 만들기

1. `워크플로우 만들기` 에서 `문서 분류 후 정보 추출` 을 선택하고, 이름을 `Matching Agent` 로 바꿉니다.
2. `data/Matching_Input.pdf` 를 업로드하고 `자동 설정` 을 선택합니다.
3. 편집기에 Parse, Classify, Extract 노드가 만들어졌는지 확인합니다.

![문서 분류 후 정보 추출 작업 유형 선택](media/27.jpg)

![Matching Agent 편집기](media/30.jpg)

## 4-2. Classify 노드

`Classify-1` 노드를 선택하고 문서종을 세 가지로 정의합니다. 자동 설정이 만든 문서종은 지우고 아래 표대로 입력합니다. 설명은 AI가 분류할 때 쓰는 기준이므로 명확하게 씁니다.

| 문서종 | 설명 |
| --- | --- |
| High Match | 지원자가 채용 공고와 매우 높은 적합도를 보이는 경우 |
| Medium Match | 일부 요구사항은 충족하지만 보완이 필요한 경우 |
| Low Match | 요구 기술 및 경험이 부족하여 적합도가 낮은 경우 |

![Classify 문서종 설정](media/31.jpg)

## 4-3. Extract 스키마

`Classify-1` 의 세 갈래에 각각 Extract 노드를 연결합니다. 노드 이름은 `Extract_HighMatch`, `Extract_MediumMatch`, `Extract_LowMatch` 로 짓고, 세 노드 모두 아래 항목 10개를 넣습니다. `overall_score` 만 Number이고 나머지는 String이며, 목록은 모두 켭니다.

| 항목명 | 설명 |
| --- | --- |
| `match_level` | 최종 적합도 분류. High Match, Medium Match, Low Match |
| `overall_score` | 전체 적합도 점수. 0~100 |
| `summary` | 전체 평가 요약 |
| `matched_skills` | JD와 일치하는 기술 스택 |
| `missing_skills` | JD에는 있으나 Resume에는 없는 기술 |
| `matched_experience` | JD와 일치하는 경력 및 경험 |
| `missing_requirements` | JD의 요구사항 중 부족한 항목 |
| `strengths` | 지원자의 강점 |
| `weaknesses` | 지원자의 보완점 |
| `recommendations` | 개선 및 학습 추천 사항 |

![Extract 스키마 설정](media/32.jpg)

:::note 세 Extract 노드의 스키마
세 Extract 노드의 항목 이름, 유형, 목록 설정을 똑같이 맞춰야 합니다. 한 노드라도 다르면 분류 결과에 따라 출력 키가 달라져 결과를 같은 방식으로 읽을 수 없습니다.
:::

## 4-4. 실행과 확인

`실행` 을 선택합니다. Classify가 적합도를 정하고 해당 Extract 노드가 평가 결과를 만듭니다. `JSON` 탭에서 `match_level`, `overall_score`, `recommendations` 를 확인합니다.

![Matching Agent 실행 결과](media/33.jpg)

- 이름이 `Matching Agent` 이고 Parse, `Classify-1`, Extract 3개 노드가 연결되어 있습니다.
- `Classify-1` 문서종이 High Match, Medium Match, Low Match 세 개입니다.
- Extract 노드 3개의 항목 10개가 모두 같습니다.
- 실행 결과에 `match_level` 과 `overall_score` 가 있습니다.

# 5. 결과 확인과 내보내기

## 5-1. 결과 JSON 구조

결과 JSON의 키는 마지막 노드의 스키마 항목 이름과 같습니다. 목록을 켠 항목은 배열, 끈 항목은 문자열로 나옵니다.

| 에이전트 | 구조 | 예시 파일 |
| --- | --- | --- |
| Resume Agent | 모든 항목이 배열입니다 | `examples/Resume.json` |
| JD Agent | 하나뿐인 값은 문자열, 여러 개인 값은 배열입니다 | `examples/JobDescription.json` |
| Matching Agent | 모든 항목이 배열이며 `match_level` 이 첫 항목입니다 | `examples/Matching_Input.json` |

결과가 예시와 다르면 스키마 항목 이름, 유형, 목록 설정을 먼저 확인합니다.

## 5-2. 워크플로우 설정 내보내기

완성한 에이전트의 설정은 JSON 파일로 내보낼 수 있습니다. 미션 제출에 이 파일을 씁니다.

1. Upstage Studio에서 내보낼 에이전트를 엽니다.
2. 오른쪽 위 더보기 `⋯` 메뉴를 엽니다.
3. `워크플로우 설정 내보내기` 를 선택합니다. 에이전트 설정이 JSON 파일로 내려받아집니다.
4. 에이전트마다 반복합니다. 워크플로우 목록에서 `워크플로우 설정 일괄 내보내기` 로 여러 개를 한 번에 내보낼 수도 있습니다.

![더보기 메뉴의 워크플로우 설정 내보내기](media/export-01.jpg)

## 5-3. API 정보 확인

에이전트 화면 오른쪽 위 메뉴에서 `API로 이용하기` 를 엽니다. `agt_` 로 시작하는 Agent ID와 Config ID를 복사합니다. 6장에서 코드로 실행할 때 씁니다.

![API로 이용하기 화면](media/19.jpg)

# 6. 코드로 실행하기

이 장은 선택입니다. Studio 화면에서는 문서를 하나씩 실행합니다. 스타터 코드를 쓰면 같은 에이전트를 여러 문서에 한 번에 실행하고 결과 JSON을 파일로 모을 수 있습니다.

## 6-1. 스타터 코드 구성

| 파일 | 하는 일 |
| --- | --- |
| `run_agent.py` | 문서 1개 또는 폴더 안 문서 전부를 에이전트에 실행하고 결과를 저장합니다 |
| `agent_client.py` | Studio 에이전트를 호출합니다. Job 생성, 상태 조회, 결과 JSON 파싱을 맡습니다 |
| `upload.py` | 문서를 Upstage Files API로 올리고 File ID를 받습니다 |
| `config.py` | API Key, Agent ID, 폴더 경로를 읽습니다 |
| `agent_configs/` | 예시 에이전트 3개의 Studio 설정 파일입니다 |
| `data/` | 예시 입력 문서입니다 |
| `examples/` | 예시 실행 결과입니다 |

```TEXT
문서 → upload.py로 File ID 발급 → agent_client.py로 에이전트 실행 → result/문서이름.json
```

## 6-2. 설치와 환경변수

가상환경을 만들고 패키지를 설치합니다.

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Windows에서는 `.venv\Scripts\activate` 로 가상환경을 켭니다.

`.env.example` 을 복사해 `.env` 를 만들고 값을 넣습니다. Agent ID와 Config ID는 5-3에서 복사한 값입니다.

```TEXT
UPSTAGE_API_KEY=up_xxxxxxxxxxxxxxxx
AGENT_ID=agt_xxxxxxxxxxxxxxxx
CONFIG_ID=1
```

API Key는 `.env` 에만 둡니다. 코드나 GitHub에 올리지 않습니다.

## 6-3. 실행

문서 1개를 실행합니다.

```bash
python run_agent.py data/Resume.pdf
```

폴더를 주면 폴더 안 문서를 전부 실행합니다. 팀 테스트 데이터셋을 한 폴더에 모아 실행할 때 씁니다.

```bash
python run_agent.py my_dataset/
```

`.env` 와 다른 에이전트로 실행하려면 옵션을 줍니다.

```bash
python run_agent.py data/JobDescription.pdf --agent-id agt_xxxx --config-id 1
```

결과는 `result/` 폴더에 문서 이름으로 저장됩니다. 실패한 문서는 실행이 끝날 때 목록으로 나옵니다.

# 7. 팀 워크플로우에 적용하기

예시 에이전트를 만드는 과정을 팀이 고른 실무 문서 업무에 그대로 적용합니다.

## 7-1. 업무와 출력 키 정하기

- 입력이 문서이고, 결과가 맞다 틀리다로 확인되는 업무를 고릅니다. 추출값이나 적격, 부적격 판정이 해당합니다.
- 출력 키를 먼저 정합니다. 항목명은 영문 snake_case로 씁니다.
- 마지막 노드의 출력 키는 정답지 키와 같아야 합니다. 정한 뒤에는 바꾸지 않습니다.
- 확실하지 않은 값은 추측하지 않고 `검토 필요` 로 출력하도록 설계합니다.

## 7-2. 노드 구성하기

- 업무 흐름을 단계로 나누고 단계마다 맞는 노드를 고릅니다. 1-1의 표를 참고합니다.
- 문서 종류나 판정 결과에 따라 처리가 달라지면 Classify로 나눕니다.
- Classify 뒤의 Extract 노드들은 스키마를 똑같이 맞춥니다.
- Extract 항목 설명에 문서 어디를 보고 뽑을지 적습니다. 설명이 정확도를 좌우합니다.

## 7-3. 테스트 데이터셋으로 확인하기

깔끔한 샘플 하나로는 워크플로우가 실무에서 돌아가는지 알 수 없습니다. 아래 유형의 문서를 섞어 실행해 봅니다.

| 유형 | 설명 |
| --- | --- |
| 단순 | 양식이 깔끔하고 값이 모두 맞는 기본 문서 |
| 양식·문맥 다양성 | 같은 업무지만 양식이나 표기가 다른 문서 |
| 문서 결합 | 여러 문서가 한 파일에 섞여 들어오는 경우 |
| 저품질 | 스캔이나 촬영으로 읽기 어려운 문서 |
| 예외·함정 | 서류 누락, 값 불일치처럼 `검토 필요` 가 정답인 경우 |

실무 문서에 개인정보나 기밀이 있으면 쓰지 않습니다. 가상 문서를 만들거나 공개 문서를 씁니다.

# 부록 A. 문제 해결

### Studio

| 증상 | 해결 |
| --- | --- |
| 자동 설정이 만든 항목 이름이 표와 다름 | 표 기준으로 이름을 고치고 빠진 항목을 추가합니다 |
| 결과 JSON에서 일부 항목이 비어 있음 | 항목 설명에 문서 어디를 보고 뽑을지 구체적으로 적습니다 |
| 여러 값이 하나의 문자열로 나옴 | 해당 항목의 목록 설정을 켭니다 |
| 분류에 따라 결과 키가 달라짐 | Classify 뒤 Extract 노드들의 스키마를 똑같이 맞춥니다 |

### 코드 실행

| 증상 | 해결 |
| --- | --- |
| `다음 환경변수가 설정되지 않았습니다.` | `.env` 가 `config.py` 와 같은 폴더에 있는지, 값이 비어 있지 않은지 확인합니다 |
| `401 Unauthorized` | `UPSTAGE_API_KEY` 를 확인합니다. Console에서 다시 발급할 수 있습니다 |
| `Agent Not Found`, 404 | Studio에서 Agent ID를 다시 복사합니다. 앞뒤 공백을 확인합니다 |
| status `failed` | Studio에서 같은 파일로 직접 실행해 어느 노드에서 실패하는지 확인합니다 |
| `output_text가 JSON 형식이 아닙니다.` | 마지막 노드가 정해진 스키마로 출력하는지 확인합니다 |
| `in_progress` 가 오래 계속됨 | 페이지 수가 많은 문서는 오래 걸립니다. 2~3쪽 문서로 먼저 확인합니다 |
| `ModuleNotFoundError` | 가상환경을 켠 상태에서 `pip install -r requirements.txt` 를 다시 실행합니다 |

# 부록 B. 참고 링크

| 항목 | 링크 |
| --- | --- |
| Upstage Studio 사용 설명서 | https://console.upstage.ai/ko/docs/studio |
| 워크플로우 만들기 | https://console.upstage.ai/ko/docs/studio/create |
| 노드 설정 | https://console.upstage.ai/ko/docs/studio/key-features |
| 실행과 결과 검토 | https://console.upstage.ai/ko/docs/studio/feedback-loop |
| 공유와 라이브러리 | https://console.upstage.ai/ko/docs/studio/sharing |
| 트러블슈팅과 FAQ | https://console.upstage.ai/ko/docs/studio/troubleshooting |
| Agents API | https://console.upstage.ai/ko/docs/agents |
