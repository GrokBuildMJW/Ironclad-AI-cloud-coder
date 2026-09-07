# Ironclad with a cloud coder

Ironclad is the conductor on **your** machine. A **cloud coder** is a remote model that does the thinking. You do not need a Spark or a 4090 for this path. You do need an LLM: without one, Ironclad boots and then refuses chat and process runs.

Public sources will live in the `ironclad` repo. This tree explains the idea. It is not those sources.

![A local computer sending a light path up into a remote cloud](docs/images/hero-local-to-cloud.jpg)

## The one picture

![Ironclad on your machine, coder model in the cloud, prompts out and answers back](docs/images/01-who-does-what.svg)

| Stays local | Goes to the cloud |
|---|---|
| Ironclad server, vault, tools, approvals | The prompt Ironclad builds |
| Workspace files on disk | Tool *schemas* (names and argument shapes) |
| Running `list_dir` and other tools | Never a raw mount of your disk |

The model never “logs into” your PC. Ironclad calls it, then Ironclad runs tools itself.

## Two ways to attach the cloud

![HTTP profile versus vendor CLI](docs/images/02-two-ways.svg)

**Way A — HTTP profile.** Ironclad POSTs to `<base_url>/chat/completions` (OpenAI-compatible, streaming). You declare `egress: external` and an API key as `${env:CLOUD_CODER_KEY}`.

**Way B — vendor CLI.** Ironclad starts a binary on `PATH` (`claude`, `grok`, `codex`, `kimi`, …). That CLI still talks to the vendor. If the binary is missing, Ironclad **refuses**. It does not silently switch to HTTP.

Pick one path and configure that path. Callers do not type a model name into the chat box. `llm.orchestrator_profile` (or the one `active` profile) chooses.

## Config sketch (no secrets)

```yaml
llm:
  base_url_allowed_hosts:
    - api.example.com
  orchestrator_profile: cloud
  profiles:
    cloud:
      base_url: https://api.example.com/v1
      api_key: ${env:CLOUD_CODER_KEY}
      model: example-coder
      egress: external
      model_identity:
        kind: hosted
        provider: example-provider
        model_id: example-coder
      input_usd_per_1m_tokens: 1.25
      output_usd_per_1m_tokens: 5.0
      max_output_tokens: 4096
      max_concurrent_requests: 4
      default_thinking_policy: disabled
```

- Put the real host in `base_url_allowed_hosts` or Ironclad will not call it.
- `egress: external` is a declaration, not inferred from the URL.
- External profiles **must** declare both USD rates. That is how the budget middleware can stop a runaway turn before money leaves.
- A local Spark/4090 profile may omit prices (zero-cost). Do not copy that onto a cloud profile.

![API key in the environment, USD rates required for external egress](docs/images/03-secrets-and-cost.svg)

Set `CLOUD_CODER_KEY` in the process that starts Ironclad. Never commit the token. An unresolved `${env:…}` means no HTTP request is sent.

## What this costs

You skip GPU hardware. You pay the vendor per token. GitHub Actions is unrelated (that is the [self-hosted CI lab](https://github.com/GrokBuildMJW/Ironclad-AI-self-hosted-GitHub-CI)).

Treat prompts as leaving the building: repo excerpts Ironclad puts in the request go to the vendor.

## When to use which lab

| Path | Hardware | LLM |
|---|---|---|
| [Minimal hardware](https://github.com/GrokBuildMJW/Ironclad-AI-minimal-hardware) | One PC, Docker | Cloud (this repo) or a small local API |
| This repo | Same PC, no extra GPU | **Cloud coder** |
| [Qwen3-Coder on RTX 4090](https://github.com/GrokBuildMJW/Qwen3-Coder-30B-A3B-Q4_K_M-llama.cpp-RTX-4090) | Consumer GPU | Local coder |
| [Flash-Next on Spark](https://github.com/GrokBuildMJW/Qwen3.8-Flash-Next-NVFP4-SGLang-DGX-Spark) | DGX Spark | Local orchestrator |
| [Self-hosted CI](https://github.com/GrokBuildMJW/Ironclad-AI-self-hosted-GitHub-CI) | Mini-PCs | Not an LLM path |

Minimal Docker plus this cloud profile is the usual “no Spark” combo: engine in a container, brain in the cloud.

## What this is not

- Not a local GPU recipe.
- Not a promise that cloud output stays on your LAN.
- Not a vendor comparison and not an API-key store.

## License

MIT for the notes and diagrams in this tree.
