# 🚀 Agentary 개발 계획 및 To-Do List (체크리스트)

## Phase 1: MVP - "지식의 뇌" 만들기 (UI: Streamlit)

* **목표:** 최소한의 리소스로 핵심 기술 가설("우리 데이터로 AI가 똑똑한 답변을 할 수 있는가?") 검증.
* **주요 To-Do:**
  * **환경 설정:**
    * [x] 디렉토리 구조 설정 (`backend/`, `frontend/` 등).
    * [x] `uv`를 사용하여 Python 가상 환경 생성 및 의존성 관리 설정.
    * [x] `Tiltfile` 구성: FastAPI 백엔드 및 Streamlit MVP UI 자동 재시작 설정.
    * [ ] `.env` 및 `.env.example` 파일로 로컬 개발 환경 변수 관리 (OpenAI API Key, Langfuse 키 등).
  * **데이터 레이어 (RAG Pipeline):**
    * [ ] `VectorDBInterface` 추상화 계층 정의 (`backend/app/core/vectordb/interface.py`).
    * [ ] `ChromaDBImpl` 구현 (`backend/app/core/vectordb/chroma.py`) (파일 기반 ChromaDB 사용).
    * [ ] `backend/app/core/vectordb/__init__.py`에 팩토리 함수 `get_vectordb_client` 구현.
    * [ ] `Data Indexer` 애플리케이션 개발 (`backend/indexer/run.py`):
      * [ ] "Full Sync with Deletion" 동기화 전략 구현.
      * [ ] 개별 문서 처리 오류 핸들링 (Langfuse 로깅).
    * [ ] `ConfluenceConnector` 개발 (`backend/app/connectors/confluence.py`).
    * [ ] `GitHubConnector` 개발 (`backend/app/connectors/github.py`).
    * [ ] `Data Indexer`와 커넥터 연동하여 ChromaDB에 데이터 색인 파이프라인 구축.
  * **에이전트 코어 (AI Engine):**
    * [ ] LangChain 기반 단일 에이전트 구현 (`backend/app/core/agent.py` 등).
    * [ ] RAG 파이프라인과 연동하여 Confluence/GitHub 데이터 기반 Q&A 기능 구현.
    * [ ] `OPENAI_MODEL_NAME` 환경 변수를 통한 LLM (OpenAI `gpt-4o`) 설정.
  * **UI 레이어 (Streamlit MVP):**
    * [ ] Streamlit UI (`backend/app/mvp_ui/app.py`) 개발.
    * [ ] 채팅 인터페이스 구현.
    * [ ] AI 답변에 대한 출처 표시 (문서 제목, 파일 경로).
    * [ ] 상세 피드백 수집 기능 구현 (👍/👎 버튼, 텍스트 입력, Langfuse 연동).
  * **테스트:**
    * [ ] `Connector` 유닛 테스트 (Mock Response 기반) 작성 및 통과.
    * [ ] 핵심 로직 (Chunking, 메타데이터 생성) 유닛 테스트 작성 및 통과.
    * [ ] `ChromaDBImpl` 통합 테스트 작성 및 통과.
    * [ ] FastAPI `TestClient`를 사용한 API 엔드포인트 E2E 테스트 작성 및 통과.

## Phase 2: 정식 서비스 전환 - "행동하는 팔다리" 붙이기 (UI: Next.js)

* **목표:** 검증된 핵심 기능을 바탕으로, 전사 사용자를 위한 안정적이고 완성도 높은 플랫폼 구축.
* **주요 To-Do:**
  * **UI 레이어 (Next.js 프로덕션 UI):**
    * [ ] Next.js 프로젝트 스캐폴딩 (`frontend/`).
    * [ ] 프로덕션 수준의 UI/UX 개발 및 FastAPI 백엔드 연동.
  * **MCP (Microservice Connector Platform) 개발:**
    * [ ] Jira 티켓 상태 조회용 `JiraTool` 개발.
    * [ ] GitHub PR 상태 조회용 `GitHubTool` 개발.
    * [ ] 개발된 Tool들을 FastAPI 백엔드에 통합.
  * **에이전트 코어 (Agentic RAG 고도화):**
    * [ ] RAG 파이프라인과 MCP Tool을 조합하여 복잡한 질문 해결 기능 고도화.
    * [ ] LangChain의 `AgentExecutor` 등을 활용한 Tool 선택 로직 구현.
  * **IaC 기반 에이전트 관리:**
    * [ ] `agent.yaml` 스키마 정의 및 예시 파일 작성 (`agents/example_agent.yaml`).
    * [ ] GitHub Actions 기반 GitOps 파이프라인 구축 (`.github/workflows/`).
    * [ ] `AWS Secrets Manager`를 통한 민감 정보 관리 연동.

## Phase 3: 플랫폼 고도화

* **목표:** 전사적 사용 및 에이전트 생태계 구축.
* **주요 To-Do:**
  * [ ] **에이전트 생성 UI:** 비개발자도 에이전트를 쉽게 생성/설정할 수 있는 GUI 개발.
  * [ ] **에이전트 간 협업:** `LangGraph` 등 멀티-에이전트 협업 프레임워크 도입 및 구현.
  * [ ] **LLMOps:** 에이전트 성능 모니터링, A/B 테스트, 피드백 기반 자동 개선 시스템 구축.
