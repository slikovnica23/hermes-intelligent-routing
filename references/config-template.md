# Hermes Intelligent Routing Config Template

Use this as a starting point when wiring Hermes routing into a real user config.

## Minimal paid + local template

```yaml
model:
  provider: minimax-cn
  default: MiniMax-M2.7
  billing_mode: plan
  billing_unit: requests

fallback_providers:
  - provider: custom:ollama-emergency-local
    model: qwen3:8b-64k
    billing_mode: local
    billing_unit: none

smart_model_routing:
  enabled: true
  difficulty:
    strategy: heuristic
    high_threshold: 5
    long_chars: 220
    long_words: 42
  selection:
    high_pool: high_cost
    low_pool: low_cost
    preferred_billing_modes:
      - plan
      - usage_api
  pools:
    high_cost:
      - provider: minimax-cn
        model: MiniMax-M2.7
        billing_mode: plan
        billing_unit: requests
    low_cost:
      - provider: minimax-cn
        model: MiniMax-M2.5
        billing_mode: plan
        billing_unit: requests
```

## Same model, two real billing runtimes

Use this only when the same model can actually be called through two distinct credentials or endpoints.

```yaml
smart_model_routing:
  enabled: true
  selection:
    high_pool: high_cost
    low_pool: low_cost
    preferred_billing_modes:
      - plan
      - usage_api
  pools:
    high_cost:
      - provider: custom:minimax-plan-m27
        model: MiniMax-M2.7
        billing_mode: plan
        billing_unit: requests
      - provider: custom:minimax-api-m27
        model: MiniMax-M2.7
        billing_mode: usage_api
        billing_unit: tokens
```

## Publishing Notes

If publishing this skill as its own GitHub repository:

- keep `SKILL.md` at the repository root
- keep `agents/openai.yaml` in `agents/`
- keep reference docs under `references/`
- include a permissive license such as MIT at the repository root
