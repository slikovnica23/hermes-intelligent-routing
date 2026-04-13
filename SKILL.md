---
name: hermes-intelligent-routing
description: Configure and maintain difficulty-aware model routing for Hermes Agent, including high-cost vs low-cost model pools, billing-mode metadata such as plan vs usage_api, OpenRouter or local fallbacks, and future multi-runtime templates. Use when Codex needs to set up, adjust, debug, or document Hermes config.yaml/.env routing behavior for cost control, reliability, or model selection strategy.
---

# Hermes Intelligent Routing

Configure Hermes so it chooses models by task difficulty first, then by billing route preference, and finally by fallback order.

Use this skill when working on Hermes routing logic or when editing `~/.hermes/config.yaml` and `~/.hermes/.env` to support:

- high-cost vs low-cost model pools
- `billing_mode` metadata such as `plan`, `usage_api`, `free`, or `local`
- fallback chains
- future runtime templates for the same model under different credentials

## Quick Start

When the user wants smart routing, follow this order:

1. Inspect the current `model`, `smart_model_routing`, `fallback_providers`, and `custom_providers` sections.
2. Identify what is actually available now:
   - paid primary models
   - token/API routes
   - free routes
   - local emergency routes
3. Preserve the user's preferred primary route unless they explicitly ask to invert priorities.
4. Configure `smart_model_routing.pools` so Hermes picks a pool by difficulty and a candidate by `billing_mode` preference.
5. Keep a local fallback in `fallback_providers` when possible.

## Routing Model

Treat Hermes routing as a four-step decision system:

1. Classify difficulty.
2. Choose a pool.
3. Choose a runtime candidate within that pool.
4. Fall back if the selected route fails.

The preferred structure is:

```yaml
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
    pool_billing_preferences:
      high_cost:
        - plan
        - usage_api
      low_cost:
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

## Billing Metadata

Use these fields on model candidates when available:

- `billing_mode`
  - `plan`
  - `usage_api`
  - `free`
  - `local`
- `billing_unit`
  - `requests`
  - `tokens`
  - `none`
- `billing_notes`
- `billing_strategy`

These fields are metadata for routing and documentation. They only become true runtime choices when Hermes can actually switch between different credentials or endpoints.

## Runtime Rules

Use these rules when deciding whether two candidates are truly different:

- Same model name plus different `api_key` can be a different runtime.
- Same model name plus different `base_url` can be a different runtime.
- Same provider plus same credentials is not a distinct runtime just because `billing_mode` text differs.

If the user has only one real credential path, keep billing metadata for documentation but do not claim Hermes can truly switch billing routes.

## Common Patterns

### Pattern 1: Two paid models plus local fallback

Use this when the user has a stronger paid model, a cheaper paid model, and Ollama:

- high-cost pool: stronger paid model
- low-cost pool: cheaper paid model
- fallback: local model

### Pattern 2: Paid primary plus OpenRouter token-style backup

Use this when the user mainly wants to stay on a paid subscription route but still wants a token-style candidate in the system:

- keep `plan` first in both pools
- add `usage_api` candidates after it
- remind the user that free OpenRouter routes may hit quota or rate limits

### Pattern 3: Same model, two billing runtimes

Use this only when the user really has two separate credentials or endpoints for the same model:

```yaml
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

## Editing Workflow

When applying changes:

1. Update `model` for the intended default route.
2. Update `smart_model_routing.selection` to match the user's cost preference.
3. Update `smart_model_routing.pools` with real candidates in priority order.
4. Update `fallback_providers` with the emergency route.
5. Add commented future templates if the user expects additional runtimes later.
6. Verify the resulting YAML by reading it back with Python and `yaml.safe_load`.

## Validation

After changes, validate at least these points:

- `smart_model_routing.enabled` is true when the user expects automatic routing.
- pool names referenced in `selection` exist in `pools`.
- every active candidate has both `provider` and `model`.
- `fallback_providers` is top-level YAML, not nested under another section.
- comments for future templates do not break YAML parsing.

## Reference Files

- For a reusable config snippet and publishing notes, read [references/config-template.md](references/config-template.md).
