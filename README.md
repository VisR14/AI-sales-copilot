# Simple FMCG Sales Copilot

A small API for classifying fictional FMCG field-sales updates, checking claims against a verified stock table, recommending a safe next step, and escalating uncertain/high-impact cases.

## Run locally

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate
# macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
uvicorn app:app --reload
```

Open http://127.0.0.1:8000/docs.

Run tests:

```bash
pytest -q
```

## Endpoints

- `POST /analyze` with `{"update": "..."}` returns the required JSON.
- `GET /health` reports whether the app is using `mock` or `openai`.
- `GET /schema` describes allowed values.

## LLM mode

By default, the app uses deterministic mock rules and does not need a key. To enable OpenAI, set `OPENAI_API_KEY` as a server-side environment variable and optionally set `OPENAI_MODEL`. Never commit the key.

The real LLM adapter has not been live-validated just by running mock-mode tests. Evaluate it separately before claiming live-model performance.

## Human review and safety

- Product-safety reports involving a customer feeling unwell receive critical urgency and human review.
- Reported stockouts that conflict with the verified stock table are escalated.
- Competitor pricing claims do not authorize a price match.
- Embedded instructions in field updates are treated as untrusted data.
- Model/API errors return a safe review-required response.
- The API validates the response with Pydantic.

## Evaluation

Expected versus actual results, pass/fail criteria, and the injection failure that was fixed are in [EVALUATION.md](EVALUATION.md). Those results are from the mock adapter (`OPENAI_API_KEY` unset). They do not validate live model behavior.

## Deployment (Render)

1. Push this folder to a GitHub repository.
2. Create a Render Web Service from that repository.
3. Build command: `pip install -r requirements.txt`
4. Start command: `uvicorn app:app --host 0.0.0.0 --port $PORT`
5. Optional: add `OPENAI_API_KEY` in the host's environment settings.
6. Verify `/health`, `/docs`, and the five required cases after deploy.

The included in-memory rate limit is a basic single-process demo control; it resets on restart and is not suitable for multi-instance production. No login is required. Use only fictional data.

## Known limitations

- Simple keyword rules can miss paraphrases and can misclassify edge cases.
- The mock adapter is not an LLM; live model results need separate evaluation.
- In-memory rate limiting is not distributed.
- No persistent audit database, authentication, or monitoring.
- Evidence traceability is basic; a production version should attach source IDs/spans and validate quotes against source text.
- This is a take-home prototype, not a production deployment.

## AI-tool usage disclosure

I used Cursor’s coding assistant to help write the API, the tests, the sample requests, and the evaluation notes, and to check the project against the brief. I ran the tests myself, read the pass and fail results, and rewrote the evaluation notes in my own words. I also checked the injection case before pushing. No API key is in the code or the repository.