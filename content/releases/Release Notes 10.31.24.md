---
title: Release Notes 10.31.24
date: 10.31.24
tags:
  - releases
  - honcho
  - dev
  - yousim
---

## News

New Honcho Updates:

- [[Release-Notes-10.31.24#honcho-v0012|Honcho v0.0.12]]
- [Python SDK v0.0.15](https://pypi.org/project/honcho-ai/)
- [NodeJS SDK v0.0.6](https://www.npmjs.com/package/honcho-ai)

Honcho Demo [YouSim](https://yousim.ai) went [viral](https://x.com/courtlandleer/status/1851009358752076261)!

## Honcho v0.0.12

an Overhauled Deriver and Dialectic API!

ADDED

- GitHub Actions Testing
- Ability to disable derivations on a session using the `deriver_disabled` flag
- in a session's metadata
- `/v1/` prefix to all routes

CHANGED

- Environment variable to control deriver workers
- Changed `public_ids` to use [NanoID](https://github.com/ai/nanoid) and internal ID to use `BigInt`
- Dialectic Endpoint can take a list of queries
- Using `uv` for project management
- User Representations stored in a metamessage rather than using reserved collection
- Base model for Dialectic API and Deriver is now Claude 3.5 Sonnet
- Paginated `GET` requests now `POST` requests for better developer UX

REMOVED

- Mirascope Dependency
- Slowapi Dependency
- Opentelemetry Dependencies and Setup

## Links

- [Sign-up for the private beta](https://plasticlabs.typeform.com/honchobeta) & start building personalized agent experiences
- [Join our Discord](https://discord.gg/plasticlabs) & tell us what you're working on
- [Visit our open-source repo](https://github.com/plastic-labs/honcho) & get your hands dirty
- [Check out the docs](https://docs.honcho.dev)
- [Play with YouSim](https://yousim.ai)--portal to the multiverse of identity
