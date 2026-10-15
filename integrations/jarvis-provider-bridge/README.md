# Jarvis Provider Bridge (starter)

**Status: committed starter code; not deployed or verified against live provider credentials.**

This small HTTP adapter gives Jarvis one authenticated interface to call model APIs for OpenAI, Anthropic/Claude, Google Gemini, Perplexity, and xAI/Grok using LiteLLM. It does **not** control the consumer chat websites or reuse paid chat subscriptions. Each provider needs its own API credentials and may bill separately.

StarNet is a separate supervised execution harness, not another model provider. Use StarNet's documented MCP stdio bridge or OpenAI-style ingress after confirming the installed build's endpoint and authentication. Do not point Jarvis at guessed endpoints.

## Files

- `bridge.py` — FastAPI adapter, provider allowlist, bearer-token auth, opt-in live calls.
- `.env.example` — variable names and placeholders only; no secrets.
- `requirements.txt` — Python dependencies.

## Safety defaults

- Live provider calls are disabled unless `JARVIS_BRIDGE_ENABLED=true`.
- The service refuses to start without a long random `JARVIS_BRIDGE_TOKEN`.
- Every request must use the bearer token.
- Only provider IDs in the fixed allowlist are accepted; callers cannot submit arbitrary model names or API URLs.
- Prompt size and output tokens are capped.
- This bridge does not send emails, make calls, write files, deploy code, or execute arbitrary tools.
- Provider API calls can incur charges. Configure provider-side budgets/rate limits too; the local request cap is not a spend guarantee.
- Never commit a real `.env` file or API key. Store production secrets in a secret manager or protected runtime environment.

## Configure

1. Use Python 3.11+ in an isolated environment.
2. Install: `pip install -r requirements.txt`.
3. Copy `.env.example` to a local `.env` file **outside Git** and set a long random bridge token.
4. Set `JARVIS_BRIDGE_ENABLED=true` only after reviewing the code and choosing a small test budget.
5. Add only the provider API keys you actually have. Set each `*_MODEL` to a currently supported model ID from that provider's documentation. Model catalogs change; this starter deliberately avoids promising that a hard-coded model ID will stay valid.
6. Run locally: `uvicorn bridge:app --host 127.0.0.1 --port 8765`.
7. Check `GET /health` and `GET /v1/providers`; then send a low-cost test to `POST /v1/consult`.

Do not expose this unauthenticated to the public internet. If Jarvis runs on another machine, bind to a private interface only behind TLS and firewall rules.

## Request example

```bash
curl http://127.0.0.1:8765/v1/consult \
  -H "Authorization: Bearer $JARVIS_BRIDGE_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"provider":"perplexity","task_id":"test-001","objective":"Summarize the supplied text and list uncertainties.","context":"A short, non-sensitive test input."}'
```

Provider IDs: `openai`, `claude`, `gemini`, `perplexity`, `grok`.

## Jarvis routing contract

Jarvis remains the CEO/control plane. It should dispatch a bounded question to one specialist, collect the result with evidence, and request independent review where needed. Do not automatically send every user message to all five providers: it increases cost and may unnecessarily disclose data. Make multi-provider council runs an explicit, budget-checked Jarvis decision.

Suggested roles:
- OpenAI/ChatGPT API: primary synthesis and implementation planning.
- Claude API: code review / alternate implementation analysis.
- Gemini API: independent reasoning and Google ecosystem tasks.
- Perplexity API: source-led research.
- Grok API: adversarial critique and trend/context checks.
- StarNet: replaceable supervised executor/harness for bounded local tasks, reached through its documented MCP/ingress interface.

## Not yet complete

- No API keys have been provisioned or tested.
- No service is deployed.
- Jarvis/Floot/Lovable has not yet been configured to call this endpoint.
- StarNet MCP/ingress has not yet been connected or tested from Jarvis.
- No live multi-provider round-trip has passed.
