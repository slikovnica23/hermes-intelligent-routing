# Hermes Intelligent Routing

A reusable Codex/Hermes skill for setting up difficulty-aware model routing in Hermes Agent.

This skill helps turn Hermes model selection into a structured decision system:

1. classify task difficulty
2. choose a high-cost or low-cost model pool
3. choose a candidate inside that pool by billing preference
4. fall back to a local or backup route when needed

It is designed for people who want Hermes to balance:

- response quality
- subscription usage
- token/API cost
- local-model reliability
- future multi-runtime expansion

## What This Skill Covers

The skill guides Hermes or Codex through:

- high-cost vs low-cost model pool design
- `billing_mode` metadata such as `plan`, `usage_api`, `free`, and `local`
- fallback chains
- OpenRouter as a token-style route
- Ollama as an emergency fallback
- future templates for the same model under separate credentials

## When To Use It

Use this skill when you want to:

- configure Hermes so simple tasks use a cheaper route and hard tasks use a stronger route
- keep a paid subscription route as primary while still exposing token-style backups
- document whether a model path is `plan`, `usage_api`, `free`, or `local`
- prepare for a future setup where the same model has separate `plan` and `usage_api` credentials
- debug or refactor `~/.hermes/config.yaml` routing logic

## Repository Contents

- `SKILL.md`  
  Core routing workflow and editing rules
- `agents/openai.yaml`  
  Skill metadata for discovery in Codex-style environments
- `references/config-template.md`  
  Reusable config snippets for real Hermes setups
- `LICENSE`  
  MIT license

## Quick Example

This is the kind of Hermes config structure the skill is built to support:

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
  pools:
    high_cost:
      - provider: minimax-cn
        model: MiniMax-M2.7
        billing_mode: plan
        billing_unit: requests
      - provider: openrouter
        model: openai/gpt-oss-120b:free
        billing_mode: usage_api
        billing_unit: tokens
    low_cost:
      - provider: minimax-cn
        model: MiniMax-M2.5
        billing_mode: plan
        billing_unit: requests
      - provider: openrouter
        model: openai/gpt-oss-20b:free
        billing_mode: usage_api
        billing_unit: tokens
```

## Before / After

### Before

Many Hermes setups start with a single default model and a single fallback:

- every request goes to one paid model
- simple and hard tasks are billed the same way
- free or local routes are only switched manually
- future `plan` vs `usage_api` expansion is undocumented

### After

With this skill, the same Hermes setup becomes a layered routing system:

- simple requests can enter a lower-cost pool
- complex requests can enter a stronger pool
- `plan`, `usage_api`, `free`, and `local` routes can be described explicitly
- fallback behavior is documented and repeatable
- future runtime expansion has a clean template instead of ad hoc edits

## Practical Scenario

Example target setup:

- high-cost pool: `MiniMax-M2.7` as the stronger paid route
- low-cost pool: `MiniMax-M2.5` as the cheaper paid route
- token-style secondary routes: OpenRouter free models
- emergency fallback: local Ollama

In that setup, Hermes can be configured to behave like this:

- simple rewrite or summarization request -> lower-cost pool
- architecture, debugging, or migration request -> stronger pool
- primary route failure -> local fallback
- future token/API credential arrival -> plug into the existing pool model

## Installation

### Option 1: Copy into a Codex skills directory

```powershell
Copy-Item -Recurse `
  "hermes-intelligent-routing" `
  "$env:USERPROFILE\\.codex\\skills\\hermes-intelligent-routing"
```

### Option 2: Clone as a standalone repository

```powershell
git clone https://github.com/slikovnica23/hermes-intelligent-routing.git
```

If you are packaging this as a standalone repository, keep these files at the repository root:

- `SKILL.md`
- `agents/openai.yaml`
- `references/`
- `LICENSE`

## Design Notes

- The skill treats routing as a layered decision process, not a single model toggle.
- `billing_mode` metadata is useful even before a setup has multiple real runtime routes.
- A route becomes a true runtime choice only when Hermes can actually switch credentials, endpoints, or provider entries.
- The skill is compatible with both simpler `cheap_model` setups and richer pool-based routing setups.

## Good Fit

- Hermes users with one strong paid model and one cheaper paid model
- Hermes users who want OpenRouter free models as token-style secondary routes
- Hermes users who keep Ollama for local fallback
- Builders designing future `plan` vs `usage_api` runtime splits

## License

MIT
