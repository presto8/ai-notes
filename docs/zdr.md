# Zero data retention

Zero data retention (ZDR) means the provider answers a request without storing
the prompt, the completion, or the code in between. It is a promise about
retention, not about access: the provider still has to process the request to
answer it, and even under ZDR some providers keep abuse-monitoring logs for a
short window. The point is that the data is not kept and not used to train
future models.

This matters more with agents than with chat. An agent sends whole files, shell
output, configuration, and sometimes credentials, and it sends them again and
again as it works.

## What I check

Before I point an agent at a provider or a router, I check three things:

- Retention window. Zero days is the strong case. Some models keep traffic for
  30 or 55 days for abuse monitoring. The window is usually a property of the
  model and the upstream provider, not of the router, so I check per model, not
  per brand.
- Training. Retention and training are separate decisions. A provider can keep
  data for a month and still not train on it, or it can offer a free or
  discounted tier in exchange for training on your prompts. I look for a
  per-model answer.
- The whole path. A router with ZDR does not mean the upstream provider has
  ZDR. The request can leave the router and land on a provider that keeps it.
  Routers handle this differently: some route only to ZDR-capable upstreams and
  fail otherwise, and some leave the choice to you.

## Two examples

OpenCode Go lists retention per model (mostly 0 days, 30 or 55 days for some,
and a couple of contributor tiers that train on your data), and a workspace
blocks paid endpoints that train by default.

Command Code enforces ZDR with one flag, `CMD_ZDR=1`, or with the
`x-cmd-zdr: 1` header on its API. With the flag on, every request runs on a
ZDR-capable upstream or fails; on models where ZDR is not included, the request
can cost more.

## A label is only as good as the verification

A ZDR label is only as good as the verification behind it, and some
aggregators repeat a provider's claim without testing it. In November 2026,
after outside researchers found that some providers were not upholding their
data policies, OpenRouter removed 125 endpoints from ZDR eligibility and
off-boarded three providers. It says it is now adapting the researchers' tests,
checking network paths for forwarding, and requiring validation before it
restores a ZDR label
([HN discussion](https://news.ycombinator.com/item?id=50595554),
[announcement](https://openrouter.ai/blog/announcements/data-processing-incidents-on-inference-providers/)).
