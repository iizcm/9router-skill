---
title: 9Router - Skill
description: AI Agent Skill for multi-provider LLM routing, API setup, and endpoint testing
---

# 9Router - Skill

[![License: MIT](https://img.shields.io/badge/license-MIT-blue?style=for-the-badge&logo=opensourceinitiative&logoColor=white)](LICENSE)
[![Universal AI Agent Skill](https://img.shields.io/badge/universal-AI_agent_skill-orange?style=for-the-badge&logo=artillery&logoColor=white)(#)
[![Category: DevOps](https://img.shields.io/badge/category-devops-purple?style=for-the-badge&logo=wix&logoColor=white)(#)

## Overview / Ringkasan

**English:**
A universal AI agent skill that enables your agent to handle **multi-provider LLM routing, API setup, and endpoint testing**. Works with any AI agent platform (Hermes, Claude Code, OpenClaw, etc.).

**Bahasa Indonesia:**
Skill AI agent universal yang memungkinkan agen Anda menangani **multi-provider LLM routing, API setup, dan pengujian endpoint**. Bekerja dengan semua platform AI agent (Hermes, Claude Code, OpenClaw, dll).

## Installation / Instalasi

### Hermes Agent
```bash
hermes skills install https://raw.githubusercontent.com/iizcm/9router-skill/main/SKILL.md --name 9router --yes
```

### Manual / Universal
Place `SKILL.md` in your agent's skills directory:
```
~/.<your-platform>/skills/9router/SKILL.md
```

## Usage / Penggunaan

1. Load the skill in your agent (see platform-specific install above).
2. Refer to the `SKILL.md` file for usage instructions.
3. Adapt examples to your environment — placeholder values are marked `<LIKE_THIS>`.

## Token / API Safety

- **Never commit real secrets.** Use `<YOUR_API_KEY>`, `<YOUR_WALLET_ADDRESS>`, `<example.com>` in examples.
- Store credentials in `~/.<your-platform>/.env` with `chmod 600`.

## License

MIT — see [LICENSE](LICENSE).
