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

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white) ![llama.cpp](https://img.shields.io/badge/llama.cpp-333333?style=for-the-badge) ![Kokoro](https://img.shields.io/badge/Kokoro_TTS-6E40C9?style=for-the-badge) ![SQLite](https://img.shields.io/badge/SQLite-003B57?style=for-the-badge&logo=sqlite&logoColor=white) ![WebSockets](https://img.shields.io/badge/WebSockets-010101?style=for-the-badge) ![Qt](https://img.shields.io/badge/PySide6-41CD52?style=for-the-badge&logo=qt&logoColor=white) ![pytest](https://img.shields.io/badge/pytest-0A9EDC?style=for-the-badge&logo=pytest&logoColor=white) ![systemd](https://img.shields.io/badge/systemd-30D475?style=for-the-badge&logo=linux&logoColor=black)

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
