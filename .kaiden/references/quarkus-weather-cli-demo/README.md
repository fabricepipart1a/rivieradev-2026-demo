# quarkus-weather-cli-demo (self-contained skill)

A drop-in Claude Code / Agent skill that drives the live "Quarkus + picocli weather CLI"
conference demo — a magician-and-assistant act where the presenter says *what* to build and
the AI does the *how*, incrementally building a native Java CLI in 14 steps.

## Contents

- `SKILL.md` — the interaction model, explanation contract, Quarkus-CLI principles, and scope guardrails.
- `scenario.md` — the 14-step presenter script with every code snippet inline. The single source of truth for what gets built.

Nothing else is required: Step 1 scaffolds the Maven project into an empty directory with
`quarkus create cli`, so no project files need to be bundled.

## Integrate into an empty repository

Copy this folder into the repo's skills directory:

```bash
mkdir -p <your-repo>/.claude/skills
cp -R quarkus-weather-cli-demo <your-repo>/.claude/skills/
```

Then, in that repo, invoke it with `/quarkus-weather-cli-demo` or just ask the AI to
"run the Quarkus weather CLI demo". The AI reads `SKILL.md`, then works through
`scenario.md` card by card as the presenter gives instructions.

## Prerequisites on the demo machine

- JDK 21, Maven (or the generated `mvnw` wrapper)
- Quarkus CLI (`quarkus`) — for scaffolding and native builds
- A container runtime (`podman` or `docker`) — for the container step
- GraalVM / Mandrel is fetched via the builder image during the native container build
