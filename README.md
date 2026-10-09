<div align="center">

# Mayen v1

**A local, private assistant: you write to it in Turkish, it answers in spoken English.**
Everything runs on one machine with one 16 GB GPU; no cloud models.

> **Archived.** This is the first full version, built in August 2026. Mayen is now being redesigned from scratch
> (v2, in design). The code, the evaluation harness and all measurement reports stay here as a reference.

</div>

<details>
<summary>Table of contents</summary>

- [About the project](#about-the-project)
- [What works](#what-works)
- [Measuring instead of guessing](#measuring-instead-of-guessing)
- [Built with](#built-with)
- [What did not get done](#what-did-not-get-done)
- [Why a v2](#why-a-v2)
- [Repository map](#repository-map)
- [License](#license)

</details>

## About the project

Mayen is a personal assistant that keeps its data at home. In v1 the owner talks to it by text in Turkish and
it replies in English, out loud. Its tools cover the weather, date and time, notes, contacts, a course timetable,
system metrics, Wake-on-LAN, launching apps, media and volume control, window management, remembered facts and
reminders, which it later announces by voice.

The project was planned in writing first (`docs/ARCHITECTURE.md`, `docs/PLAN.md`) and then built in 29 small,
numbered steps (P0–P29) between 6 and 17 August 2026. Each step has a written "done" criterion and the
decisions taken along the way.

## What works

- **Agent loop with tools:** a tool registry and catalogue, tool calls constrained by GBNF grammars or by the
  model's native tool format, and a correction loop that lets the model fix an unknown tool, an unknown argument
  or a wrong argument type on the second try
- **Server and clients:** an asyncio server with a WebSocket protocol, a headless text client, a Qt (PySide6)
  desktop window with a status indicator, and a client audio path (microphone, endpointing, speaker)
- **Voice output:** Kokoro text-to-speech
- **Scheduler:** reminders and scheduled tasks that speak up on their own
- **Memory:** a bounded context window, summaries of trimmed history and persistent facts, each tagged with who said it
- **Data and operations:** SQLite with migrations and automatic backups, a process lock, and systemd user units
  that tell permanent configuration errors apart from temporary ones

## Measuring instead of guessing

Prompt, role-text and call-format changes were checked with an evaluation harness (`evals/`) instead of by feel.
Scenario sets cover tool accuracy, hallucination, memory, control cases and language rules; every ratio carries a
Wilson 95% interval, every run produces a raw report, and every decision has a written reading of what the
numbers do **and do not** show (see `docs/README.md`). Where a choice was a judgement call rather than a
measurement, the docs say so.

Some findings:
- The production setup ended as **Qwen 27B (IQ4_XS) on llama.cpp with the model's native tool-call format, and
  Kokoro on the CPU**. The model itself was a judgement call, not a measurement result. The native format was
  chosen because, in the text formats, the model sometimes faked a tool call inside its spoken answer. The 27B
  model alone took about 15.6 GB of the 16 GB card.
- A model scan (`docs/faz-c-model-taramasi.md`) rejected a small 2.6B model that produced no tool calls at all in
  the text formats, and found that a 35B mixture-of-experts model matched the 27B in healthy turns but, once it
  had made something up, kept going instead of recovering.
- The same language-rule text improved the 27B on three scenario sets and made the 35B worse on the same three:
  prompt changes have to be measured per model.

## Built with

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white) ![llama.cpp][llamacpp-badge] ![Kokoro](https://img.shields.io/badge/Kokoro_TTS-6E40C9?style=for-the-badge) ![SQLite](https://img.shields.io/badge/SQLite-003B57?style=for-the-badge&logo=sqlite&logoColor=white) ![WebSockets](https://img.shields.io/badge/WebSockets-010101?style=for-the-badge) ![Qt](https://img.shields.io/badge/PySide6-41CD52?style=for-the-badge&logo=qt&logoColor=white) ![pytest](https://img.shields.io/badge/pytest-0A9EDC?style=for-the-badge&logo=pytest&logoColor=white) ![systemd](https://img.shields.io/badge/systemd-30D475?style=for-the-badge&logo=linux&logoColor=black)

Also: asyncio, httpx, structlog, ruff, mypy.

## What did not get done

- **Speech recognition stayed a stub.** The STT choice was postponed; a fake STT treats the client's text as the
  transcript, so voice *input* never worked end to end.
- **Speaker recognition also stayed a stub**, so the four-level permission model (owner / registered person /
  pending / unknown) was never exercised with real voices.

## Why a v2

v1 assumed one machine, one conversation turn at a time and one owner. The redesign (v2) aims at a personal
assistant platform: modules that can run on different machines in a private network, several channels
(voice, text, notifications), turns that can run in parallel, and people with different roles. Rather than
stretch v1, it is being designed again from the start. The useful parts of v1 (adapters, tools, the data layer
and the evaluation harness) are the reference for that work.

## Repository map

| Path | What |
|---|---|
| `src/mayen/` | the application |
| `client/` | text and desktop clients |
| `services/` | model service wrappers (e.g. Kokoro) |
| `evals/` | evaluation harness and report generator |
| `docs/` | architecture, plan and every measurement report (in Turkish); start with `docs/README.md` |
| `config/` | systemd user units and configuration |

## License

Copyright © 2026 Sinan Gölcük. All rights reserved. The code is published for reference only; no license to use, copy or modify it is granted.

<!-- llama.cpp icon: ggml-org/llama.cpp media/llama1-icon-transparent.svg (MIT) -->
[llamacpp-badge]: https://img.shields.io/badge/llama.cpp-333333?style=for-the-badge&logo=data:image/svg%2bxml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiIHN0YW5kYWxvbmU9Im5vIj8+CjxzdmcKICAgaWQ9IkxheWVyXzEiCiAgIHZlcnNpb249IjEuMSIKICAgdmlld0JveD0iMCAwIDI1MCAyNTAiCiAgIHNvZGlwb2RpOmRvY25hbWU9ImxsYW1hMS1pY29uLXRyYW5zcGFyZW50LnN2ZyIKICAgd2lkdGg9IjI1MCIKICAgaGVpZ2h0PSIyNTAiCiAgIGlua3NjYXBlOnZlcnNpb249IjEuNC4yIChlYmYwZTk0MGQwLCAyMDI1LTA1LTA4KSIKICAgeG1sbnM6aW5rc2NhcGU9Imh0dHA6Ly93d3cuaW5rc2NhcGUub3JnL25hbWVzcGFjZXMvaW5rc2NhcGUiCiAgIHhtbG5zOnNvZGlwb2RpPSJodHRwOi8vc29kaXBvZGkuc291cmNlZm9yZ2UubmV0L0RURC9zb2RpcG9kaS0wLmR0ZCIKICAgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIgogICB4bWxuczpzdmc9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KICA8c29kaXBvZGk6bmFtZWR2aWV3CiAgICAgaWQ9Im5hbWVkdmlldzciCiAgICAgcGFnZWNvbG9yPSIjNTA1MDUwIgogICAgIGJvcmRlcmNvbG9yPSIjZmZmZmZmIgogICAgIGJvcmRlcm9wYWNpdHk9IjEiCiAgICAgaW5rc2NhcGU6c2hvd3BhZ2VzaGFkb3c9IjAiCiAgICAgaW5rc2NhcGU6cGFnZW9wYWNpdHk9IjAiCiAgICAgaW5rc2NhcGU6cGFnZWNoZWNrZXJib2FyZD0iMSIKICAgICBpbmtzY2FwZTpkZXNrY29sb3I9IiM1MDUwNTAiCiAgICAgaW5rc2NhcGU6em9vbT0iMi40OCIKICAgICBpbmtzY2FwZTpjeD0iNDkuNTk2Nzc0IgogICAgIGlua3NjYXBlOmN5PSIxODkuOTE5MzUiCiAgICAgaW5rc2NhcGU6d2luZG93LXdpZHRoPSIzNDQwIgogICAgIGlua3NjYXBlOndpbmRvdy1oZWlnaHQ9IjE0NDAiCiAgICAgaW5rc2NhcGU6d2luZG93LXg9IjAiCiAgICAgaW5rc2NhcGU6d2luZG93LXk9IjAiCiAgICAgaW5rc2NhcGU6d2luZG93LW1heGltaXplZD0iMSIKICAgICBpbmtzY2FwZTpjdXJyZW50LWxheWVyPSJMYXllcl8xIiAvPgogIDwhLS0gR2VuZXJhdG9yOiBBZG9iZSBJbGx1c3RyYXRvciAyOS4zLjEsIFNWRyBFeHBvcnQgUGx1Zy1JbiAuIFNWRyBWZXJzaW9uOiAyLjEuMCBCdWlsZCAxNTEpICAtLT4KICA8ZGVmcwogICAgIGlkPSJkZWZzMSI+CiAgICA8c3R5bGUKICAgICAgIGlkPSJzdHlsZTEiPgogICAgICAuc3QwIHsKICAgICAgICBmaWxsOiAjZmY4MjM2OwogICAgICB9CgogICAgICAuc3QxIHsKICAgICAgICBmaWxsOiAjZmZmOwogICAgICB9CgogICAgICAuc3QyIHsKICAgICAgICBmaWxsOiAjMWIxZjIwOwogICAgICB9CiAgICA8L3N0eWxlPgogIDwvZGVmcz4KICA8ZwogICAgIGlkPSJnNyI+CiAgICA8ZwogICAgICAgaWQ9Imc2IgogICAgICAgdHJhbnNmb3JtPSJ0cmFuc2xhdGUoLTk5NS41MTA2NiwtMTI5LjcwODc1KSI+CiAgICAgIDxwYXRoCiAgICAgICAgIGNsYXNzPSJzdDAiCiAgICAgICAgIGQ9Im0gMTE2My4zLDIyNi44IC0xMy41LDI0IGMgLTE3LjgsLTEzLjcgLTQ0LjIsLTE1LjcgLTYyLC0xIC0yOC43LDIzLjcgLTI2LjcsNzguNSAxOCw3OC44IDEyLjUsMCAyMy4xLC01LjkgMzQuNSwtOS44IGwgNiwyMy45IGMgLTEwLjEsNC43IC0yMC40LDkuNSAtMzEuNSwxMSAtMTAxLjIsMTMuOCAtOTUuNCwtMTMyLjMgLTMuOSwtMTM5LjkgMTkuMiwtMS42IDM2LjEsMy40IDUyLjUsMTMgeiIKICAgICAgICAgaWQ9InBhdGg0IiAvPgogICAgICA8cGF0aAogICAgICAgICBjbGFzcz0ic3QwIgogICAgICAgICBkPSJtIDEwOTMuNCwyMDMuOCBjIC0xNS40LDQuNiAtMjkuNywxMy4xIC00MC41LDI1IC0yLC0yNC4yIDMuNCwtNzMuMSAzMC4zLC04Mi43IDQsLTEuNCAxNy43LC00LjkgMTcuMywyLjIgLTAuNCw3LjEgLTkuOSwxOS4zIC0xMi4yLDI1LjkgLTQsMTEuNiAtMC4zLDE5LjYgNS4yLDI5LjcgeiIKICAgICAgICAgaWQ9InBhdGg1IiAvPgogICAgICA8cG9seWdvbgogICAgICAgICBjbGFzcz0ic3QwIgogICAgICAgICBwb2ludHM9IjExMzEuNCwzMDcuOCAxMTE2LjQsMzA3LjggMTExNi40LDI5MC44IDEwOTkuNCwyOTAuOCAxMDk5LjQsMjc2LjggMTExNC45LDI3Ni44IDExMTYuNCwyNzUuMyAxMTE2LjQsMjU4LjggMTEzMS40LDI1OC44IDExMzEuNCwyNzYuOCAxMTQ3LjQsMjc2LjggMTE0Ny40LDI5MC44IDExMzEuNCwyOTAuOCAiCiAgICAgICAgIGlkPSJwb2x5Z29uNSIgLz4KICAgICAgPHBvbHlnb24KICAgICAgICAgY2xhc3M9InN0MCIKICAgICAgICAgcG9pbnRzPSIxMTg2LjQsMjkwLjggMTE4Ni40LDMwNy44IDExNzEuNCwzMDcuOCAxMTcxLjQsMjkwLjggMTE1NS40LDI5MC44IDExNTUuNCwyNzYuOCAxMTcxLjQsMjc2LjggMTE3MS40LDI1OC44IDExODYuNCwyNTguOCAxMTg2LjQsMjc1LjMgMTE4Ny45LDI3Ni44IDEyMDMuNCwyNzYuOCAxMjAzLjQsMjkwLjggIgogICAgICAgICBpZD0icG9seWdvbjYiIC8+CiAgICAgIDxwYXRoCiAgICAgICAgIGNsYXNzPSJzdDAiCiAgICAgICAgIGQ9Im0gMTE0Mi4zLDE1Ni45IGMgMiwzIC05LjMsMTUuOSAtMTEuMSwxOS4yIC01LjIsOS44IC0xLjcsMTUuNCAyLjIsMjQuNyAtMTEuMywtMS43IC0yMS44LC0wLjMgLTMzLDEgMi41LC0yMS41IDE0LjYsLTUyLjggNDEuOSwtNDQuOSB6IgogICAgICAgICBpZD0icGF0aDYiIC8+CiAgICA8L2c+CiAgPC9nPgo8L3N2Zz4K
