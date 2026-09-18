# OpenRouter

[OpenRouter](https://openrouter.ai) is a router that sits in front of many
model providers (Anthropic, OpenAI, Google, Meta, DeepSeek, Mistral,
Groq-hosted models, and many open-weight models) behind one OpenAI-compatible
endpoint and API key. I use it to reach models outside my main Anthropic and
Foundry setup, without a separate account and key for each provider.

## Reasons I use it

- One key, many models. Any tool that already speaks the OpenAI chat
  completions API can point at OpenRouter (`https://openrouter.ai/api/v1`)
  and reach the full model catalog. You only change the model name.
- Good for trying a model once. When I want to compare a response against an
  open-weight or non-Anthropic model, OpenRouter is the fastest path. I do
  not need a separate account for that provider.
- Fallback routing. If a request's primary provider or model is down or
  rate-limited, OpenRouter can send the request to a different provider or
  model. You can configure this per request.
- Popularity data. OpenRouter publishes which models its users call the
  most. The [rankings page](https://openrouter.ai/rankings) shows token
  volume by model, and you can filter it by category, such as programming
  or roleplay. The [models page](https://openrouter.ai/models?order=top-weekly)
  sorts the full catalog by weekly usage. This is a fast way to see which
  models people use in practice, not only which ones benchmark well.
- An automatic model. The [Auto Router](https://openrouter.ai/openrouter/auto)
  (`openrouter/auto`) reads each prompt and picks a model for it from a
  curated set. Use it when you do not want to choose a model yourself. The
  response says which model it used. Read the
  [Auto Router documentation](https://openrouter.ai/docs/features/model-routing)
  for the details.

## How to point a tool at it

Most of my agent CLIs and editor tools accept a custom OpenAI-compatible base
URL. To use OpenRouter with them: set the base URL to
`https://openrouter.ai/api/v1`, set the API key, and pick a model from the
OpenRouter catalog. Each model name carries a provider prefix, for example
`deepseek/deepseek-v4.1`.

## Fees when you add funds

OpenRouter charges a processing fee, close to 5 percent, on every deposit,
and it applies a minimum charge. A small deposit pays a much higher effective
rate because of that minimum. Add at least $15 at a time to keep the fee from
taking an outsized share of the deposit. I pay with a rewards credit card that
returns 2 percent cash back. This brings my net fee down to close to 3
percent.
