# Tiltfile for Agentary Development Environment

# FastAPI Backend API
local_resource('api-server',
  serve_cmd='uv run uvicorn backend.app.main:app --host 0.0.0.0 --port 8000',
  deps=['backend/app'],
  auto_init=True,
  readiness_probe=probe(
      period_secs=30,
      http_get=http_get_action(port=8000, path="/monitor")
  )
)

# Streamlit MVP UI
local_resource('mvp-ui',
  serve_cmd='STREAMLIT_SERVER_HEADLESS=true uv run streamlit run backend/app/mvp_ui/app.py --server.port 8501',
  deps=['backend/app/mvp_ui'],
  auto_init=True,
  readiness_probe=probe(
      period_secs=30,
      http_get=http_get_action(port=8501, path="/")
  )
)

# Data Indexer (manual trigger)
local_resource('indexer',
  cmd='uv run python backend/indexer/run.py',
  auto_init=False,
  resource_deps=['api-server']
)
