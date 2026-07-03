# Quarkus Weather CLI — Live Demo

A conference demo that builds a **Quarkus + picocli weather CLI** from scratch, live, in **14 short steps**. The finished app fetches the current temperature from the [Open-Meteo](https://open-meteo.com/) API, compiles to a **native binary**, and ships as a **container image**.

The whole point is to show how little code Quarkus needs to get from an empty directory to a production-ready, native, containerized CLI.

## The Act: magician & assistant

The demo is staged as a **magician-and-assistant act**:

- **You (the presenter)** face the audience and give short, declarative instructions — *"Now — make it speak its temperature…"*. You never touch the code; you say *what* to build.
- **The AI assistant** performs the trick. For each instruction it edits the code instantly, explains what it did and how Quarkus made it easy, then hands you the exact command to run next.

This split keeps the presenter's attention on the audience while the AI does the typing.

## Prerequisites

Install these before the demo:

- **JDK 21**
- **Quarkus CLI** — `quarkus` ([install guide](https://quarkus.io/guides/cli-tooling))
- **Maven** (or use the generated `./mvnw` wrapper)
- **GraalVM / Mandrel** for native builds, or Docker/Podman for containerized native builds
- **Docker or Podman** — for the container image step

Versions used by the demo: **Quarkus 3.26.2**, **Java 21**. Project coordinates: `com.amadeus:weather:0.0.1-SNAPSHOT`.

## Running the Demo

1. Start in an **empty working directory** — Step 1 scaffolds the project into it with `quarkus create cli`.
2. Drive the AI assistant with the `quarkus-weather-cli-demo` skill. It reads the full 14-step scenario and knows the whole arc.
3. For each step, give the natural-language instruction; the AI edits files and gives you the command to run in your terminal.

**Division of labor during the demo:**

- The **AI** edits files and runs fast, safe project-shaping commands (`quarkus extension add …`).
- **You** run the slow or environment-dependent commands — native compilation (`quarkus build --native`), any `mvn` invocation, container builds, and running the binary.

## What Gets Built (the 14 steps)

### Phase 1 — Base CLI
1. **Scaffold** — `quarkus create cli com.amadeus:weather:0.0.1-SNAPSHOT` stamps out a native-capable picocli project.
2. **Package & run** — confirm the CLI accepts arguments.
3. **Silence the noise** — disable the banner and lower Quarkus log levels via `application.properties`.
4. **Native binary** — `quarkus build --native` compiles a standalone executable; trim the HTTP entrypoint from `Dockerfile.native`.
5. **Container image** — add `quarkus-container-image-docker` and a `native-container` Maven profile to publish `weather:latest`.

### Phase 2 — CLI with Commands
6. **Command tree** — a flat picocli tree: `Main` (`@TopCommand`) → `Get` (the `get` command).
7. **Parameters & option** — latitude, longitude, and an optional `--location` name.
8. **Implement `get`** — print the temperature (placeholder value for now).
9. **Tests** — `@QuarkusMainTest` + `@Launch` verify exit codes and output.

### Phase 3 — CLI with Business Logic
10. **REST client extension** — add `rest-client-jackson`.
11. **Client interface + DTOs** — `@RegisterRestClient` interface describing the Open-Meteo endpoint, plus JSON-mapped DTOs.
12. **Business service** — a `@Singleton` service that injects the REST client.
13. **Wire real temperature** — `@Inject` the service into `Get` and fetch live data.
14. **Configure & ship** — point the REST client at Open-Meteo in `application.properties`, then rebuild the native binary and container.

## The Finished App

```bash
# Native binary (after quarkus build --native, aliased to `weather`)
weather get 41 29 --location=Istanbul
# → The current temperature in Istanbul is 24 ºC

# Containerized native binary
podman run -it weather:latest temp --in Istanbul 41 29
```

By the end you've built a production-ready Quarkus CLI that parses commands with picocli, calls a REST API with a type-safe client, compiles to a native binary, and ships as a container image — with Quarkus handling all the infrastructure.
