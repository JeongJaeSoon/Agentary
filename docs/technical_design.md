# Agentary 기술 설계 문서 (v1.2)

이 문서는 `prd.md`에 정의된 목표를 달성하기 위한 구체적인 기술 스택, 아키텍처, 개발 환경 설계를 정의합니다.

## 1. 최종 목표 아키텍처 (AWS 기반)

프로젝트의 최종 목표는 AWS의 관리형 서비스를 최대한 활용하여 안정적이고 확장 가능한 시스템을 구축하는 것입니다.

- **Frontend (Next.js):** `AWS Amplify`를 통해 빌드/배포 자동화.
- **Backend (FastAPI):** `Amazon ECS on AWS Fargate`를 사용하여 서버리스 컨테이너로 운영.
- **RAG Pipeline:** `Amazon Bedrock Knowledge Bases`를 활용하여 데이터 수집, 임베딩, 인덱싱 파이프라인을 완전 자동화.
- **Vector DB:** `Amazon OpenSearch Service Serverless`를 사용하여 운영 부담 없이 벡터 검색 기능 사용.
- **CI/CD:** `AWS CodePipeline`을 통해 Git-to-Deploy 파이프라인 구축.
- **보안:** `AWS Secrets Manager`를 통해 모든 민감 정보 관리.

## 2. MVP 단계 아키텍처 및 기술 스택

MVP 단계에서는 빠른 개발과 검증에 초점을 맞추되, 향후 프로덕션으로의 전환이 용이하도록 설계합니다.

### 2.1. 핵심 애플리케이션

| 컴포넌트 | 기술 | 설명 |
| :--- | :--- | :--- |
| **Backend API** | **FastAPI** | Python 기반의 고성능 비동기 API 서버. |
| **MVP UI** | **Streamlit** | AI/백엔드 개발자가 직접 빠르게 프로토타입 UI를 구축. |
| **Data Indexer** | **별도 Python App** | API 서버와 분리된 독립적인 데이터 연동/임베딩 애플리케이션. |
| **Observability**| **Langfuse** | LLM 애플리케이션의 실행 추적, 로깅, 피드백 관리를 위한 통합 플랫폼. |

#### 2.1.1. MVP UI 상세 요구사항 (Streamlit)

- **답변 출처 표시:** AI 답변 하단에 해당 내용의 근거가 된 문서(e.g., Confluence 페이지 제목, GitHub 파일 경로) 목록을 명시하여 사용자가 사실을 검증할 수 있도록 합니다.
- **상세 피드백 수집:**
    - 답변마다 👍/👎 버튼을 배치합니다.
    - 버튼 클릭 시, 선택적으로 상세 이유를 기입할 수 있는 텍스트 입력 창을 제공하여 구체적인 피드백을 수집하고 Langfuse에 기록합니다.

### 2.2. LLM 및 Observability

- **LLM 선정:** **OpenAI** 모델 (`gpt-4o` 등)을 사용합니다. 모델명은 `OPENAI_MODEL_NAME` 환경 변수를 통해 쉽게 교체할 수 있도록 구현하여 유연성을 확보합니다.
- **실행 추적 및 로깅:** **Langfuse**를 공식 Observability 도구로 도입합니다.
    - **역할:** LangChain과의 콜백 연동을 통해 RAG 파이프라인의 모든 실행 과정(Trace), 단계별 입/출력, 비용, 지연 시간, 사용자 피드백을 자동으로 추적하고 시각화합니다.
    - **기대 효과:** 복잡한 AI 로직의 디버깅 효율을 극대화하고, 수집된 데이터를 기반으로 시스템 성능을 분석하고 개선의 근거로 활용합니다.

### 2.3. 데이터 모델: 메타데이터 스키마

검색 정확도와 필터링 기능을 극대화하기 위해 VectorDB에 저장되는 모든 문서 조각(Chunk)은 다음의 통일된 메타데이터 스키마를 따릅니다.

```python
class DocumentMetadata(TypedDict):
    source_type: str      # 'confluence', 'github', 'slack' 등 데이터 소스 유형
    source_id: str        # Confluence 페이지 ID, GitHub 파일 경로 등 데이터 소스 내 고유 식별자
    document_id: str      # 시스템 전체에서 고유한 문서 ID (e.g., 'confluence-12345')
    chunk_id: str         # 문서 내에서 각 Chunk의 고유 ID (e.g., hash(document_id + chunk_text))
    author: Optional[str] # 문서 작성자
    created_at: Optional[datetime] # 문서 생성일
    tags: List[str]       # 관련 태그 (e.g., 'frontend', 'auth', 'react')
```

### 2.4. Vector DB 전략: "바꿔 끼우기" 설계

Vector DB는 교체가 용이하도록 **추상화 계층(Abstraction Layer)**을 도입합니다.

1. **`VectorDBInterface` 정의:** `upsert_documents`, `similarity_search` 등 모든 Vector DB가 구현해야 할 공통 메서드를 담은 추상 클래스를 정의합니다.
2. **구현체 작성:**
    - **MVP용:** `ChromaDBImpl` 클래스가 인터페이스를 상속받아 파일 기반 `ChromaDB`의 실제 로직을 구현합니다.
    - **미래용:** `OpenSearchImpl` 등 다른 DB 구현체를 동일한 인터페이스로 추가할 수 있습니다.
3. **팩토리(Factory) 패턴:** 애플리케이션은 설정값(`VECTOR_DB_TYPE`)에 따라 `get_vectordb_client` 팩토리 함수를 통해 적절한 DB 구현체 인스턴스를 받아 사용합니다. 이를 통해 DB 교체 시 API나 Indexer의 코드 수정이 필요 없게 됩니다.

**MVP 선택:** **`ChromaDB`**
> **사유:** 별도 서버 없이 파일 기반으로 동작하여 설정이 극도로 간단하고, LangChain 등과의 생태계 지원이 가장 풍부하여 MVP 개발 속도를 극대화할 수 있습니다.

### 2.5. 데이터 연동 파이프라인 (Indexer)

- **분리된 애플리케이션:** API 서버의 성능과 안정성을 보장하기 위해, 리소스를 많이 사용하는 데이터 연동 및 임베딩 프로세스는 별도의 애플리케이션으로 분리합니다.
- **실행 전략:** `backend/indexer/run.py`와 같이 단독 실행 가능한 스크립트로 개발하여, **수동 실행**(개발/디버깅)과 **자동 스케줄링**(운영)을 모두 지원합니다.
- **동기화 전략: "Full Sync with Deletion"**
  1. **ID 목록 수집:** 데이터 소스(e.g., Confluence)에서 현재 유효한 모든 문서의 ID 목록을 가져옵니다.
  2. **문서 처리 및 저장:** 각 문서를 가져와 처리(Chunking, Embedding)하고 VectorDB에 `upsert`합니다.
  3. **오래된 데이터 삭제:** 동기화 시작 시점의 VectorDB에 존재하던 문서 ID 중, 1번에서 수집한 최신 ID 목록에 없는 모든 문서를 VectorDB에서 삭제하여 데이터 정합성을 유지합니다.
- **에러 핸들링:** 특정 문서 처리 중 오류 발생 시, 해당 오류를 Langfuse에 로그로 기록하고 건너뛰어 전체 동기화 프로세스가 중단되지 않도록 합니다.

## 3. 개발 환경 구성

모든 팀원이 일관되고 효율적인 개발 환경을 공유하기 위해 다음 도구를 사용합니다.

### 3.1. Python 환경: `uv`

- **역할:** `venv`와 `pip`를 대체하는 Rust 기반의 차세대 Python 패키지 관리 도구.
- **사용법:**
  1. `uv venv` 명령으로 `.venv` 가상 환경 생성.
  2. `uv pip install -r requirements.txt`로 매우 빠르게 의존성 설치.

### 3.2. 통합 개발 환경: `Tilt`

- **역할:** 여러 서비스(FastAPI, Streamlit 등)를 한 번에 관리하고, 코드 변경 시 자동으로 재시작하며, 로그를 한 곳에서 모아보는 **"개발 환경 오케스트레이션 툴"**.
- **`Tiltfile` 구성:**
  - `api-server`: FastAPI 서버를 실행하고 `backend/app` 디렉토리 변경 시 자동 재시작.
  - `mvp-ui`: Streamlit 앱을 실행하고 `backend/app/mvp_ui` 디렉토리 변경 시 자동 재시작.
  - `indexer`: 데이터 동기화 Indexer를 수동으로 실행할 수 있는 버튼 제공.
- **개발자 워크플로우:**
  1. `tilt up` 명령어로 모든 개발 서버 시작.
  2. `http://localhost:10350`의 Tilt UI에서 모든 서비스의 상태와 로그를 중앙에서 관리.
  3. 코드 저장 시, Tilt가 변경을 감지하여 해당 서비스만 자동으로 리로드.

### 3.3. 설정 및 민감 정보 관리: 점진적 접근

- **로컬 환경:**
  - **`.env` 파일:** 프로젝트 루트의 `.env` 파일을 통해 `OPENAI_API_KEY`, `LANGFUSE_SECRET_KEY` 등 로컬 개발용 민감 정보를 관리합니다.
  - **`.env.example`:** 필요한 환경 변수 목록을 정의한 템플릿 파일을 제공하여 팀원의 환경 설정을 돕습니다.
  - **`.gitignore`:** `.env` 파일은 Git 추적에서 제외하여 민감 정보 유출을 방지합니다.
- **프로덕션 환경:**
  - **`AWS Secrets Manager`:** 코드 변경 없이 프로덕션으로 전환하기 위해, 애플리케이션은 항상 환경 변수에서 설정을 읽도록 구현합니다. 프로덕션 배포 시, AWS Secrets Manager에 저장된 값을 컨테이너의 환경 변수로 주입하여 안전하게 민감 정보를 관리합니다.

## 4. 테스트 전략

- **Unit Tests (`pytest`):**
  - **Connector:** 각 `Connector`가 외부 API의 Mock Response(미리 저장된 JSON)를 정확히 파싱하고 `Document` 객체로 변환하는지 검증합니다.
  - **Core Logic:** 텍스트 분할(Chunking), 메타데이터 생성 등 핵심 비즈니스 로직을 테스트합니다.
- **Integration Tests:**
  - **VectorDB:** `ChromaDBImpl`이 실제 파일 시스템에 데이터를 쓰고, 읽고, 삭제하는 기능이 `VectorDBInterface`에 맞춰 정상 동작하는지 검증합니다.
- **API Tests (E2E):**
  - **FastAPI `TestClient`:** 실제 API 엔드포인트를 호출하여, 유효한 요청에 대해 올바른 응답과 상태 코드를 반환하는지 종단 간 테스트를 수행합니다.
- **Observability 기반 평가:**
  - **Langfuse 데이터 활용:** Langfuse에 축적된 사용자 피드백(좋은 답변/나쁜 답변) 케이스를 분석하여, 이를 기반으로 회귀 테스트 셋을 구축하고 시스템의 답변 품질을 지속적으로 평가하고 개선합니다.

## 5. 디렉토리 구조

```directory
/Agentary
├── .github/
│   └── workflows/          # GitHub Actions (CI/CD) 파이프라인
├── .venv/                  # uv 가상 환경 (gitignore 처리)
├── agents/                 # IaC 기반 에이전트 설정 (`agent.yaml`)
│   └── example_agent.yaml
├── backend/
│   ├── app/                # 핵심 애플리케이션 소스 코드
│   │   ├── __init__.py
│   │   ├── main.py         # FastAPI 서버 진입점
│   │   ├── core/           # Agent Core, RAG 로직 등
│   │   │   └── vectordb/   # VectorDB 추상화 계층
│   │   │       ├── __init__.py # 팩토리 함수
│   │   │       ├── interface.py
│   │   │       └── chroma.py
│   │   ├── connectors/     # 데이터 연동 커넥터 (Confluence, GitHub 등)
│   │   ├── routers/        # FastAPI API 엔드포인트
│   │   └── mvp_ui/         # Streamlit 기반 MVP UI
│   │       └── app.py
│   ├── indexer/            # 데이터 연동/임베딩 Indexer 애플리케이션
│   │   └── run.py
│   ├── tests/              # 백엔드 테스트 코드
│   └── requirements.txt    # Python 패키지 의존성
├── docs/
│   ├── prd.md
│   └── technical_design.md # 본 문서
├── frontend/               # Phase 2: Next.js 프로덕션 UI
├── .gitignore
├── .env.example            # 로컬 개발 환경 변수 템플릿
└── Tiltfile                # Tilt 개발 환경 설정 파일
```

## 6. Phase 2 및 향후 고려사항

MVP 개발 완료 후, 다음 기능들을 순차적으로 도입하여 플랫폼을 고도화합니다.

### 6.1. 행동 수행 에이전트 (Agentic Action)

- **목표:** 정보 검색(RAG)을 넘어, 에이전트가 실제 시스템에 변경을 가하는 '행동'을 수행하도록 합니다.
- **구현 방안:**
  - **`Tool` 개발:** `prd.md`에 명시된 `MCP(Microservice Connector Platform)`를 통해 실제 외부 서비스를 제어하는 도구들을 개발합니다. (e.g., `GitHubTool` for PR 생성, `JiraTool` for 티켓 상태 변경)
  - **Agent Core 고도화:** `LangChain`의 `AgentExecutor` 등을 활용하여, LLM이 계획을 수립하고 적절한 `Tool`을 선택하여 호출하도록 RAG 파이프라인을 Agentic Loop로 확장합니다.

### 6.2. 자동 Wiki 생성 (Knowledge Summarization)

- **목표:** 특정 데이터 소스(e.g., GitHub 레포지토리)의 전체 내용을 요약하고 구조화하여 사람이 읽기 쉬운 Wiki 문서(e.g., Confluence 페이지)를 자동으로 생성하고 업데이트합니다.
- **구현 방안:**
  - **`WikiGenerator` 에이전트 개발:** 특정 데이터 소스 전체를 분석하고, 핵심 정보를 추출/요약하여 마크다운 형식의 문서로 생성하는 특수 목적의 에이전트를 개발합니다.
  - 이 에이전트는 주기적으로 실행되어 최신 정보를 반영한 Wiki를 유지보수합니다.

### 6.3. 사용자 피드백 루프 (Human-in-the-Loop)

- **목표:** 사용자의 피드백을 수집하고 이를 통해 시스템의 답변 정확도를 점진적으로 개선합니다.
- **구현 방안 (단계적 접근):**
  1. **피드백 수집 (MVP+):** UI에 답변에 대한 만족도(👍/👎) 평가 및 코멘트 기능을 추가합니다. 수집된 데이터(질문, 답변, 평가, Trace ID)는 **Langfuse**에 중앙화하여 기록하고 분석합니다.
  2. **피드백 활용 (Phase 2):**
      - **Few-shot Learning:** 좋은 평가를 받은 Q&A 쌍을 Langfuse에서 선별하여 LLM 프롬프트에 예시로 포함, 답변 품질을 향상시킵니다.
      - **Fine-tuning:** 수집된 대량의 피드백 데이터를 사용하여 기반 LLM 모델을 주기적으로 미세 조정(Fine-tuning)합니다.
      - **RAG 결과 랭킹 개선:** 좋은 평가를 받은 문서가 검색 결과에서 더 높은 순위를 차지하도록 랭킹 로직을 조정합니다.

### 6.4. 에이전트 간 협업 (Agent-to-Agent Collaboration)

- **목표:** 복잡한 태스크를 해결하기 위해 여러 전문 에이전트가 서로 상호작용하고 협력하는 생태계를 구축합니다.
- **구현 방안:** `LangChain`의 `LangGraph`와 같은 멀티-에이전트 협업 프레임워크를 도입하여, 특정 역할을 가진 에이전트(e.g., 기획 에이전트, 개발 에이전트, QA 에이전트)들이 정의된 워크플로우에 따라 작업을 주고받도록 설계합니다.
