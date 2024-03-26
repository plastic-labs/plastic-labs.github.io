---
title: Release Notes 03.21.24
date: 03.21.24
tags:
  - releases
  - announcements
  - honcho
  - dev
  - ml
  - research
---
## News

Research-y week in the lab: 

- [[Achieving SOTA on OpenToM with DSPy|Blog post on achieving theory of mind SOTA with DSPy!]]
  
- [Private Beta Waitlist Sign-up](https://plasticlabs.typeform.com/honchobeta)

- [Fresh Docs](https://docs.honcho.dev)
  
- [Honcho v0.0.6](https://github.com/plastic-labs/honcho/tree/v0.0.6)  
  
See you [in Discord](https://discord.gg/plasticlabs) 🥽

## Honcho v0.0.6

ADDED
- Full docker-compose for API and Database
- Full docstring coverage
- Code coverage tests
- Add LangChain to Honcho message converter in both directions
- Synonym `init` function that acts the same as `initialize`

CHANGED
- Refactored API server into multiple route files
- Harvester renamed to deriver

FIXED
- API Response schema removed unnecessary fields
- OTEL logging to properly work with async database engine
- `fly.toml` default settings
