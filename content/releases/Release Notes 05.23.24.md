---
title: Release Notes 05.23.24
date: 05.23.24
tags:
  - releases
  - announcements
  - honcho
  - dev
  - blog
---
## News

Honcho health improvements:

- More docs overhaul

- Issue templates and contribution guides

- Reliability improvements

[Sign-up for the private beta](https://plasticlabs.typeform.com/honchobeta) and start building personalized agent experiences.

[Join Discord](https://discord.gg/plasticlabs), introduce yourself, and tell us what you're working on.

[Visit our open-source repo](https://github.com/plastic-labs/honcho) and get your hands dirty.

## Honcho

ADDED
- Issue templates to repo
- Updated discord starter template
- Updated examples to honcho-python repository
- LangChain message converter integration

FIXED
- metadata fields are treated as dicts in SDKs rather than base object types

CHANGED
- HONCHO_AUTH_TOKEN is now HONCHO_API_KEY
- Get users and get sessions return 4xx exceptions if nothing is found.  

REMOVED
- DB_TYPE from .env.template
