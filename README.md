# Hermes Intelligent Routing

A reusable Codex/Hermes skill for configuring difficulty-aware model routing in Hermes Agent.

This skill helps structure Hermes model selection around:

- task difficulty
- high-cost vs low-cost model pools
- billing metadata such as `plan`, `usage_api`, `free`, and `local`
- fallback chains
- future multi-runtime templates for the same model

## What It Does

The skill teaches Hermes or Codex how to configure routing like:

1. classify request difficulty
2. choose a high-cost or low-cost pool
3. choose a candidate within that pool by billing preference
4. fall back to a local model when needed

It is especially useful when the user wants to balance:

- quality
- subscription usage
- token/API cost
- local-model reliability

## Files

- `SKILL.md`  
  Core skill instructions and routing workflow
- `agents/openai.yaml`  
  UI metadata for skill discovery
- `references/config-template.md`  
  Reusable config templates for Hermes
- `LICENSE`  
  MIT license

## Example Use Cases

- Configure Hermes so simple tasks use a cheaper paid model and hard tasks use a stronger paid model.
- Add OpenRouter as a token-style backup route without making it the default.
- Prepare a future template for the same model under separate `plan` and `usage_api` credentials.
- Keep Ollama as a last-resort emergency fallback.

## Install

Copy this folder into a Codex skills directory, for example:

```powershell
Copy-Item -Recurse `
  "hermes-intelligent-routing" `
  "$env:USERPROFILE\\.codex\\skills\\hermes-intelligent-routing"
```

If you are packaging it as a standalone repository, the repository root should contain:

- `SKILL.md`
- `agents/openai.yaml`
- `references/`
- `LICENSE`

## Recommended Repository Name

`hermes-intelligent-routing`

## License

MIT
