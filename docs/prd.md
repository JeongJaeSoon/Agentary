# Agentary

## 사내 LLM 기반 AI 개발 플랫폼 설계 제안서 (v1.2)

### 1. 개요 (Overview)

본 문서는 사내 개발자의 생산성을 높이고, 비개발 직군도 사내 기술 자산에 쉽게 접근할 수 있도록 돕는 LLM 기반 AI 개발 플랫폼 구축을 위한 설계를 정의합니다. 이 플랫폼은 사내 GitHub, Confluence, Jira 등의 데이터를 기반으로 질문에 답변하고, 궁극적으로는 여러 AI 에이전트가 협업하는 생태계를 지향합니다.

MVP 단계에서는 Python 기반의 Streamlit을 사용하여 핵심 기능을 빠르게 검증하고, 이후 정식 서비스 단계에서 React/Next.js 기반의 UI로 확장하는 것을 목표로 합니다.

### 2. 핵심 아키텍처 (Core Architecture)

플랫폼은 하이브리드 RAG(Hybrid RAG) 모델과 플러그인 기반의 도구(Tool) 아키텍처를 채택합니다.

* 정적 지식 (문서, 코드): Vector DB에 사전 색인하여 빠른 의미 기반 검색을 지원합니다.
* 동적 정보 (티켓 상태, PR 현황): 실시간 API 호출 도구를 통해 항상 최신 정보를 조회합니다.

#### 아키텍처 구성 요소

* UI Layer (MVP): Streamlit을 사용하여 AI/백엔드 개발자가 직접 구축하는 빠른 프로토타입 UI.
* Orchestration Layer (Backend): FastAPI 기반의 API 서버. 사용자 요청을 받아 적절한 에이전트와 도구를 선택하고 작업 흐름을 총괄합니다.
* Agent Core (AI Engine): LangChain 프레임워크를 사용하여 특정 역할을 부여받은 LLM. 계획을 수립하고, 도구를 사용하며, 최종 결과를 생성합니다.
* Tool Layer: 에이전트가 사용하는 기능 모음.
  * RAG Pipeline: Vector DB에서 정적 지식을 검색하는 도구.
  * MCP (Microservice Connector Platform): Jira, GitHub 등 외부 서비스의 API를 호출하여 실시간 정보를 가져오는 실시간 Tool 서버.
* Data Layer:
  * Vector DB: Confluence, GitHub 등의 정적 데이터를 벡터로 저장.
  * 외부 서비스 API: Jira, Slack 등 외부 서비스의 실시간 데이터 소스.
* Management Layer: 에이전트 설정, 데이터 연동, 보안을 관리하는 백엔드 시스템.

***

### 3. 에이전트 설정 및 관리: IaC (Infrastructure as Code)

모든 에이전트의 설정은 `agent.yaml` 파일을 통해 코드로 관리하며, GitOps 파이프라인을 통해 시스템에 자동 배포합니다.

* 에이전트 정의 (`agent.yaml`):
  * `metadata`: 에이전트의 이름, 설명, 소유자
  * `core`: 시스템 프롬프트, 사용할 LLM 모델 등
  * `data_sources`: RAG에 사용할 데이터 소스 (Confluence 스페이스, GitHub 레포 등)
  * `tools`: 사용할 실시간 API 도구 목록
* 워크플로우:
  1. Git Repo: 개발자가 `agent.yaml` 파일을 수정하고 PR을 생성.
  2. CI/CD (GitHub Actions): PR이 머지되면 파이프라인 실행.
  3. Validate & Deploy: YAML 유효성 검사 후, 에이전트 관리 서비스 API를 호출하여 시스템에 등록/갱신.
  4. Secure: API 키 등 민감 정보는 코드와 분리하여 Secret Manager를 통해 안전하게 주입.

***

### 4. 데이터 연동 방식: 플러그인 아키텍처

데이터 연동은 확장성을 극대화하기 위해 플러그인(커넥터) 방식으로 구현합니다.

* 역할 분담:
  * `agent.yaml`: 무엇을 연결할지 선언 (e.g., `type: confluence`).
  * 데이터 커넥터 (Backend): 어떻게 연결하고 데이터를 처리할지 실제 로직을 구현한 모듈 (`ConfluenceConnector.py`).
* 동작 원리: 시스템이 YAML의 `type`을 보고 해당 커넥터를 동적으로 로드하여 데이터 수집 및 처리를 위임합니다.
* 확장성: 새로운 데이터 소스(e.g., Notion)를 추가할 때, `NotionConnector` 모듈만 추가하면 되므로 시스템 수정이 최소화됩니다.

***

### 5. 단계별 개발 로드맵 (Phased Development Roadmap)

#### Phase 1: MVP - "지식의 뇌" 만들기 (UI: Streamlit) 🧠

* 목표: 가장 중요한 기술 가설("우리 데이터로 AI가 똑똑한 답변을 할 수 있는가?")을 최소한의 리소스로 빠르게 검증.
* 주요 개발 내용:
  1. 단일 에이전트: 특정 서비스(1개)에 대한 Q&A 기능.
  2. 핵심 데이터 소스 연동: Confluence와 GitHub 문서에 대한 RAG 파이프라인 구축.
  3. 초고속 프로토타입 UI: Streamlit을 사용하여 AI 개발자가 직접 채팅 인터페이스와 피드백 수집 기능을 구현. (프론트엔드 리소스 투입 X)
  4. 내부 피드백 수집: 완성된 Streamlit 데모를 통해 소수 핵심 유저로부터 피드백을 받아 빠르게 개선.
* 제외 대상: 이 단계에서는 실시간 정보 조회가 필요 없으므로 MCP(실시간 Tool 서버)는 개발하지 않습니다.

#### Phase 2: 정식 서비스 전환 - "행동하는 팔다리" 붙이기 (UI: Next.js) 🦾

* 목표: 검증된 핵심 기능을 바탕으로, 전사 사용자를 위한 안정적이고 완성도 높은 플랫폼 구축.
* 주요 개발 내용:
  1. 프로덕션 UI/UX: Next.js(React)를 사용하여 확장성 있고 세련된 정식 UI/UX 구축.
  2. MCP(실시간 Tool 서버) 개발: Jira, GitHub PR 상태를 조회하는 실시간 API 도구를 구현하고 MCP를 통해 제공.
  3. Agentic RAG 도입: 에이전트가 RAG 파이프라인과 MCP를 조합하여 복잡한 질문을 해결하도록 고도화.
  4. IaC 기반 에이전트 관리: `agent.yaml`과 GitOps 파이프라인을 통한 에이전트 관리 시스템 구축.

#### Phase 3: 플랫폼 고도화

* 목표: 전사적 사용 및 에이전트 생태계 구축.
* 주요 개발 내용:
  1. 에이전트 생성 UI: 비개발자도 쉽게 에이전트를 생성하고 설정할 수 있는 GUI 제공.
  2. 에이전트 간 협업: 특정 업무를 위해 여러 에이전트가 상호작용하는 기능 구현.
  3. LLMOps: 에이전트 성능 모니터링, A/B 테스트, 피드백 기반 자동 개선 등 운영 시스템 구축.
