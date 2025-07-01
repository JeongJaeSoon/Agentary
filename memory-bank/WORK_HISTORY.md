# Agentary 개발 작업 기록

이 문서는 Agentary 프로젝트의 주요 개발 작업 및 진행 상황을 기록합니다.

## 2025-07-02 - 개발 환경 초기 설정 및 Tiltfile 구성

### 상세 내용

- `backend/` 디렉토리 및 초기 파일 (`main.py`, `app.py`, `run.py`, `requirements.txt`) 생성.
- `uv`를 사용하여 Python 가상 환경 설정 및 의존성 설치.
- `Tiltfile`을 구성하여 FastAPI 백엔드 및 Streamlit MVP UI 서비스 정의.
- `http_probe`를 `readiness_probe`로 변경하고 적절한 엔드포인트 (`/monitor`) 추가.
- Streamlit 이메일 입력 프롬프트 비활성화를 위한 `STREAMLIT_SERVER_HEADLESS` 환경 변수 설정.
- `.gitignore` 파일 생성 및 `.env.example` 파일 추가.

### 특이사항 / 결정 사항

- `Tiltfile`에서 `local_resource`의 포트 노출은 `resource_port`나 `port` 인자가 아닌, `http_probe` 또는 `serve_cmd` 내에서 포트를 직접 열어주는 방식으로 처리.
- `uv run`을 사용하여 가상 환경 활성화 없이 명령어 실행.

---