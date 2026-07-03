---
name: quarkus-weather-cli-demo
description: Use when presenting or driving the live "Quarkus + picocli weather CLI" conference demo — the magician-and-assistant act where the presenter gives short declarative instructions and the AI incrementally builds a native Java command-line weather app in 14 steps. Triggers include "run the weather CLI demo", "let's build the Quarkus CLI", or any card/step from the scenario.
---

# Quarkus Weather CLI — Live Demo

## Overview

This skill drives a live conference demo that incrementally builds a **Quarkus + picocli weather CLI** in 14 steps. The finished app fetches the current temperature from the Open-Meteo API, compiles to a native binary, and ships as a container.

The demo is a **magician-and-assistant act**: the **presenter** (magician) faces the audience and says *what* to build, one short instruction at a time; **you** (the AI assistant) perform the trick — you do the *how*: edit code instantly, explain what you did and how Quarkus made it easy, then hand the presenter the exact command to run next.

**Everything needed is self-contained.** [scenario.md](scenario.md) holds all 14 steps with every code snippet inline. You do not need any other repository files — the project is scaffolded fresh with `quarkus create cli` in Step 1.

## The Interaction Model (magician & assistant)

- The **presenter** gives short, declarative commands ("Now — make it speak its temperature…"). They are the interlocutor with the audience.
- **You** quietly drive the demo. For each instruction respond with, **in this order**:
  1. **Code edits** — the file changes that fulfill the instruction.
  2. **Explanation** — the fixed-format block (see Explanation Contract). Its `**Try it:**` line *is* the cue forward — the exact command for the presenter to run. Do not emit a separate "You run" block; the `Try it` line covers it.
  3. **Optionally**, one short sentence nudging toward the next step ("Once that prints, we'll give it real data."). Keep it to a single line; never let it replace or duplicate the `Try it` command.

## Division of Labor

- **You edit files.** You may also run **fast, safe project-shaping commands** yourself when a card calls for it — specifically `quarkus extension add …` (Cards 5, 10). These are quick and non-interactive.
- **You never trigger slow or environment-dependent commands yourself**: native compilation (`quarkus build --native`), any `mvn` invocation (`mvn clean package`, `mvn test`, `mvn quarkus:dev`), container builds, and running the binary. Hand each of those to the **presenter** with the exact command and expected output.

## Explanation Contract

After every step, emit this exact block. Tone: concise, confident, emphasizing that Quarkus made the step trivial.

```
**What I did:** …
**How Quarkus helped:** …
**Try it:** `<command for the presenter to run>` — expected: …
```

## Quarkus-CLI Principles

- **picocli command tree** via `@TopCommand` + `@Command(subcommands = …)`; picocli auto-generates help. This demo uses a **flat tree**: `Main` (`@TopCommand`) points `subcommands` directly at `Get` — no intermediate holder command.
- **Separate command code from business code**; wire with `@Inject`.
- **REST client** via `@RegisterRestClient(configKey = "…")`; the host URL comes from `quarkus.rest-client.<key>.url`; Jackson auto-maps JSON to DTOs.
- **CLI tests** via `@QuarkusMainTest` + `@Launch`, checking output and exit code.
- **Configuration** in `application.properties`, overridable via system properties / env vars / profiles.

## Project Conventions

- **Group ID**: `com.amadeus` · **Artifact ID**: `weather` · **Version**: `0.0.1-SNAPSHOT`
- **Java version**: 21 · **Quarkus version**: `3.26.2`
- **Package structure**: flat `com.amadeus` package (no sub-packages).
- **Scaffolding**: prefer `quarkus create cli com.amadeus:weather:0.0.1-SNAPSHOT` (the `create cli` template wires in the picocli extension automatically). The `https://code.quarkus.io/?g=com.amadeus&a=weather&v=0.0.1-SNAPSHOT` web generator is the fallback.

## Scope Guardrails

Stay faithful to [scenario.md](scenario.md) — it is the source of truth for what gets built and in what order.

**Explicitly excluded (YAGNI — do not add these):**
- No CI / release / Sonar GitHub workflows
- No assembly-zip distribution plumbing
- No second example commands (`say`, `github`, `version`)
- No two-logger (technical + console) setup

## How to Run the Demo

1. Confirm you have an empty working directory (Step 1 scaffolds the project into it).
2. Read [scenario.md](scenario.md) fully so you know the whole arc.
3. For each of the 14 cards, when the presenter gives you the "You say" instruction (in their own words), respond with: code edits → Explanation block → the "You run" command as the cue forward.
4. Never run slow builds yourself — hand `quarkus build --native`, `mvn …`, and container builds to the presenter.
5. If the presenter jumps steps or improvises, find the matching card in scenario.md and stay faithful to its code.
