# Live Demo Scenario — The 14-Step Weather CLI

This is the presenter-facing live script for the Quarkus + picocli weather CLI demo. The demo is staged as a magician-and-assistant act: you (the presenter) face the audience and give short, declarative instructions; the AI assistant performs the trick by editing code and explaining what it did.

Each of the 14 cards has four fields:

| Field | Meaning |
|---|---|
| You say | Natural-language instruction the presenter gives (pasteable into any AI) |
| AI does | Expected file edits |
| AI explains | The what / how-Quarkus-helped block the AI should produce |
| You run | Command for the presenter's terminal + expected output |

---

## Phase 1: Base CLI (Steps 1–5)

### Card 1: Scaffold

**You say:** "Set us up — a Quarkus command-line app called weather, group com.amadeus, with picocli."

**AI does:**
- Hands the presenter the scaffold command `quarkus create cli com.amadeus:weather:0.0.1-SNAPSHOT` (the `create cli` template wires in the picocli extension automatically). The browser generator `https://code.quarkus.io/?g=com.amadeus&a=weather&v=0.0.1-SNAPSHOT` is the fallback.
- Generates project with picocli extension
- Confirms `pom.xml` has correct group/artifact/version coordinates

**AI explains:**
**What I did:** Scaffolded a new Quarkus CLI project with the picocli extension for command-line parsing.
**How Quarkus helped:** A single `quarkus create cli` command stamps out a ready-to-run, native-capable picocli project — coordinates, Maven wrapper, Dockerfiles and the picocli extension all wired in, zero boilerplate.
**Try it:** `quarkus create cli com.amadeus:weather:0.0.1-SNAPSHOT && cd weather && mvn clean package` — expected: BUILD SUCCESS and a runnable JAR in `target/quarkus-app/`.

**You run:**
```bash
quarkus create cli com.amadeus:weather:0.0.1-SNAPSHOT
cd weather
mvn clean package
```
Expected: project scaffolded, then BUILD SUCCESS

---

### Card 2: Package & run

**You say:** "Now let it greet a city."

**AI does:**
- Confirms the default main command runs and accepts arguments
- Verifies the basic picocli setup is functional

**AI explains:**
**What I did:** Verified the scaffolded CLI accepts arguments and runs successfully.
**How Quarkus helped:** Out of the box, Quarkus + picocli wires the main command with argument parsing — no manual setup.
**Try it:** `java -jar ./target/quarkus-app/quarkus-run.jar Istanbul` — expected: output showing the argument was processed.

**You run:**
```bash
java -jar ./target/quarkus-app/quarkus-run.jar Istanbul
```
Expected: Output showing "Istanbul" was received

---

### Card 3: Silence the noise

**You say:** "Quiet the stage — no Quarkus banner, fewer logs."

**AI does:**
- Adds to `src/main/resources/application.properties`:
```properties
quarkus.banner.enabled=false
quarkus.log.category."io.quarkus".level=WARN
```

**AI explains:**
**What I did:** Disabled the Quarkus banner and set Quarkus framework logs to WARN level.
**How Quarkus helped:** Simple config keys in `application.properties` control banner and logging — no XML, no code.
**Try it:** `mvn clean package` then re-run the JAR — expected: clean output, no banner or INFO logs.

**You run:**
```bash
mvn clean package
java -jar ./target/quarkus-app/quarkus-run.jar Istanbul
```
Expected: No banner, minimal logs

---

### Card 4: Native binary

**You say:** "Make it a real native executable."

**AI does:**
- Confirms native build configuration is present (from picocli extension) — no `application.properties` changes needed; the binary keeps its default name `target/weather-0.0.1-SNAPSHOT-runner`
- Drops the HTTP entrypoint arg in `weather/src/main/docker/Dockerfile.native` — this is a CLI, not a server, so `-Dquarkus.http.host=0.0.0.0` is dead weight (and would swallow the user's CLI args):
```dockerfile
ENTRYPOINT ["./application"]
```
(replacing the generated `ENTRYPOINT ["./application", "-Dquarkus.http.host=0.0.0.0"]`)
- Hands over the native build command (does NOT run it)
- Suggests a shell alias so the long runner name reads cleanly as `weather`:
```bash
alias weather=target/weather-0.0.1-SNAPSHOT-runner
```

**AI explains:**
**What I did:** Verified native build configuration is ready, trimmed the HTTP entrypoint arg in `Dockerfile.native` since this is a CLI, and set up a `weather` alias so the commands read naturally.
**How Quarkus helped:** Native executable support is built-in — one command compiles Java to a standalone binary.
**Try it:** `quarkus build --native` (takes a few minutes), then `file -b target/weather-0.0.1-SNAPSHOT-runner` to see it's a native binary, then alias it and run `weather YOU` — expected: instant startup, no JVM.

**You run:**
```bash
quarkus build --native
file -b target/weather-0.0.1-SNAPSHOT-runner
alias weather=target/weather-0.0.1-SNAPSHOT-runner
weather YOU
```
Expected: File shows "Mach-O 64-bit executable" or similar, instant startup

---

### Card 5: Container image

**You say:** "Now ship it as a container."

**AI does:**
- Runs: `quarkus extension add quarkus-container-image-docker`
- Adds to `pom.xml` the `native-container` Maven profile:
```xml
        <profile>
            <id>native-container</id>
            <properties>
                <quarkus.container-image.name>weather</quarkus.container-image.name>
                <quarkus.container-image.group></quarkus.container-image.group>
                <quarkus.docker.dockerfile-native-path>${basedir}/src/main/docker/Dockerfile.native</quarkus.docker.dockerfile-native-path>
                <quarkus.native.container-runtime>docker</quarkus.native.container-runtime>
                <quarkus.container-image.build>true</quarkus.container-image.build>
                <quarkus.native.container-build>true</quarkus.native.container-build>
                <quarkus.native.builder-image>quay.io/quarkus/ubi-quarkus-mandrel-builder-image:23.1-java21</quarkus.native.builder-image>
                <quarkus.native.enabled>true</quarkus.native.enabled>
                <quarkus.container-image.tag>latest</quarkus.container-image.tag>
                <assembly.skipAssembly>false</assembly.skipAssembly>
            </properties>
        </profile>
```

**AI explains:**
**What I did:** Added the container-image-docker extension and configured a native container build profile that publishes the image as `weather:latest` (empty group, `latest` tag).
**How Quarkus helped:** Quarkus generates the Dockerfile and builds the native container image — no manual Dockerfile authoring.
**Try it:** `mvn clean package -Pnative-container` (takes a few minutes), then `podman run -it weather:latest -h` — expected: help output from the containerized native binary.

**You run:**
```bash
mvn clean package -Pnative-container
podman run -it weather:latest -h
```
Expected: Container builds, help text shows

---

## Phase 2: CLI with Commands (Steps 6–9)

### Card 6: Command tree

**You say:** "Give it a proper command: `get`."

**AI does:**
- Creates `src/main/java/com/amadeus/Main.java`, pointing `subcommands` straight at `Get` (no intermediate `weather` holder):
```java
package com.amadeus;

import io.quarkus.picocli.runtime.annotations.TopCommand;
import picocli.CommandLine.Command;

@TopCommand
@Command(name = "weather", mixinStandardHelpOptions = true, subcommands = Get.class)
public class Main {

}
```
- Creates `src/main/java/com/amadeus/Get.java`:
```java
package com.amadeus;

import java.util.concurrent.Callable;

import picocli.CommandLine.Command;

@Command(name = "get", aliases = {"temp"})
public class Get implements Callable<Integer> {

    @Override
    public Integer call() throws Exception {
        // TODO Auto-generated method stub
        throw new UnsupportedOperationException("Unimplemented method 'call'");
    }

}
```

**AI explains:**
**What I did:** Built a flat command tree with `Main` as `@TopCommand` and `Get` registered directly as the `get` command — no intermediate `weather` subcommand, so it's invoked as `get` (or `weather get` once aliased to the binary).
**How Quarkus helped:** Quarkus auto-wires picocli's `@TopCommand` and `subcommands` — just declare classes, no registration code.
**Try it:** `mvn quarkus:dev` then type `get` in the dev console — expected: "UnsupportedOperationException" since `call()` isn't implemented yet.

**You run:**
```bash
mvn quarkus:dev
```
Then in dev prompt:
```
get
```
Expected: Exception (not implemented yet)

---

### Card 7: Parameters & option

**You say:** "Let it take a latitude, a longitude, and an optional location name."

**AI does:**
- Adds to `Get.java`:
```java
    @CommandLine.Parameters(arity = "1", description = "in degrees (41 for Istanbul)")
    public float latitude;

    @CommandLine.Parameters(arity = "1", description = "in degrees (29 for Istanbul)")
    public float  longitude;

    @CommandLine.Option(names = {"--location", "--in"}, description = "Name of the location")
    public Optional<String> location;
```
- Adds `import java.util.Optional;` at top

**AI explains:**
**What I did:** Added two `@Parameters` fields for latitude/longitude and one `@Option` field for an optional location name.
**How Quarkus helped:** Picocli's annotations make parameter binding declarative — no manual parsing or validation.
**Try it:** `get` in dev mode — expected: usage error showing the new parameters and option.

**You run:**
In dev prompt:
```
get
```
Expected: Error message showing required parameters

---

### Card 8: Implement get (placeholder)

**You say:** "Make it announce the temperature."

**AI does:**
- Implements `Get.call()` with placeholder logic:
```java
    @Override
    public Integer call() throws Exception {
        String loc = location.orElse(latitude + "," + longitude);
        float temp = 0f;
        System.out.printf("The current temperature in %s is %f ºC", loc, temp);
        return 0;
    }
```

**AI explains:**
**What I did:** Implemented `call()` to compute a location string and print the temperature (placeholder `0f` for now).
**How Quarkus helped:** Standard Java business logic — Quarkus stays out of the way.
**Try it:** `get 41 29` in dev mode — expected: "The current temperature in 41.0,29.0 is 0.000000 ºC".

**You run:**
In dev prompt:
```
get 41 29
```
Expected: Temperature printed as 0f

---

### Card 9: Tests

**You say:** "Prove it works — write the tests."

**AI does:**
- Creates `src/test/java/com/amadeus/GetTest.java`:
```java

package com.amadeus;


import org.junit.jupiter.api.*;

import io.quarkus.test.junit.main.Launch;
import io.quarkus.test.junit.main.LaunchResult;
import io.quarkus.test.junit.main.QuarkusMainTest;

import java.io.*;
import java.util.Optional;
import static org.junit.jupiter.api.Assertions.*;

@QuarkusMainTest
class GetTest {

  @Test
  @Launch(value = {"get"}, exitCode = 2)
  void noParameter(LaunchResult result) {
  }

  @Test
  @Launch(value = {"get", "41", "29"}, exitCode = 0)
  void works(LaunchResult result) {
    System.out.println("Output: " + result.getOutput());
    Assertions.assertNotNull(result.getOutput());
    Assertions.assertTrue(result.getOutput().contains("temp"));

  }
}
```

**AI explains:**
**What I did:** Created `GetTest` with `@QuarkusMainTest`, testing that missing parameters fail (exit code 2) and valid input succeeds (exit code 0).
**How Quarkus helped:** `@Launch` annotation runs the CLI as if from the command line — no test harness boilerplate.
**Try it:** `mvn test` — expected: both tests pass.

**You run:**
```bash
mvn test
```
Expected: Tests pass

---

## Phase 3: CLI with Business (Steps 10–14)

### Card 10: Add REST client extension

**You say:** "Now make it talk to a real weather service."

**AI does:**
- Runs: `quarkus extension add rest-client-jackson`

**AI explains:**
**What I did:** Added the REST client and Jackson extensions for calling HTTP JSON APIs.
**How Quarkus helped:** Extension auto-configures REST client with JSON serialization — zero manual setup.
**Try it:** Confirm in dev mode it still starts — expected: no errors, app reloads.

**You run:**
In dev mode (already running):
Expected: Hot reload, no errors

---

### Card 11: Client interface + DTOs

**You say:** "Describe the Open-Meteo endpoint."

**AI does:**
- Creates `src/main/java/com/amadeus/WeatherClient.java`:
```java
package com.amadeus;

import org.eclipse.microprofile.rest.client.inject.RegisterRestClient;

import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.QueryParam;

@RegisterRestClient(configKey = "weather")
public interface WeatherClient {


    @Path("/forecast")
    @GET
    TemperatureResult getTemperature(@QueryParam("latitude") float latitude, @QueryParam("longitude") float longitude, @QueryParam("current") String current);
}
```
- Creates `src/main/java/com/amadeus/TemperatureResult.java`:
```java
package com.amadeus;

public class TemperatureResult {
    
    public float latitude;
    public float longitude;
    public TemperatureCurrent current;
}
```
- Creates `src/main/java/com/amadeus/TemperatureCurrent.java`:
```java
package com.amadeus;

public class TemperatureCurrent {
    
    public String time;
    public float temperature_2m;

}
```

**AI explains:**
**What I did:** Defined the REST client interface with `@RegisterRestClient` and two DTO classes matching the Open-Meteo JSON schema.
**How Quarkus helped:** Quarkus REST client generates the HTTP calls from the interface — no manual HTTP code. Jackson auto-maps JSON to POJOs.
**Try it:** Dev mode reloads — expected: no errors.

**You run:**
In dev mode:
Expected: Hot reload, no errors

---

### Card 12: Business service

**You say:** "Put the logic behind a service."

**AI does:**
- Creates `src/main/java/com/amadeus/WeatherService.java`:
```java
package com.amadeus;

import org.eclipse.microprofile.rest.client.inject.RestClient;

import jakarta.inject.Singleton;

@Singleton
public class WeatherService {

    @RestClient 
    WeatherClient weatherClient;

    public int getCurrentTemperature(float latitude, float longitude) {
        return (int) weatherClient.getTemperature(latitude, longitude, "temperature_2m").current.temperature_2m;
    }
}
```

**AI explains:**
**What I did:** Created a `@Singleton` service that injects the REST client and wraps the temperature call.
**How Quarkus helped:** `@RestClient` injection works seamlessly — Quarkus manages the HTTP client lifecycle.
**Try it:** Dev mode reloads — expected: no errors.

**You run:**
In dev mode:
Expected: Hot reload, no errors

---

### Card 13: Wire real temperature

**You say:** "Connect the wires — real data now."

**AI does:**
- Updates `Get.java` to inject `WeatherService` and call it:
```java
package com.amadeus;

import java.util.Optional;
import java.util.concurrent.Callable;

import jakarta.inject.Inject;
import picocli.CommandLine;
import picocli.CommandLine.Command;

@Command(name = "get", aliases = {"temp"})
public class Get implements Callable<Integer> {

    @Inject 
    WeatherService weather;

    @CommandLine.Parameters(arity = "1", description = "in degrees (41 for Istanbul)")
    public float latitude;

    @CommandLine.Parameters(arity = "1", description = "in degrees (29 for Istanbul)")
    public float  longitude;

    @CommandLine.Option(names = {"--location", "--in"}, description = "Name of the location")
    public Optional<String> location;

    @Override
    public Integer call() throws Exception {
        String loc = location.orElse(latitude + "," + longitude);
        int temp = weather.getCurrentTemperature(latitude, longitude);
        System.out.printf("The current temperature in %s is %d ºC\n", loc, temp);
        return 0;
    }

}
```

**AI explains:**
**What I did:** Injected `WeatherService` into `Get` and replaced the placeholder `0f` with a real call to `weather.getCurrentTemperature()`, printing as `%d` instead of `%f`.
**How Quarkus helped:** CDI `@Inject` works in picocli commands — no manual wiring, just annotate.
**Try it:** `get 41 29 --location=Istanbul` in dev mode — expected: real temperature fetched from Open-Meteo (but will fail without config in next step).

**You run:**
In dev prompt:
```
get 41 29 --location=Istanbul
```
Expected: Error (REST client not configured yet)

---

### Card 14: Configure & ship

**You say:** "Point it at Open-Meteo and ship the native container."

**AI does:**
- Adds to `src/main/resources/application.properties`:
```properties
quarkus.rest-client.weather.url=https://api.open-meteo.com/v1
quarkus.rest-client.weather.scope=jakarta.inject.Singleton
# Because Injecting certificates in a binary would have taken too long to demo
quarkus.tls.trust-all=true
```

**AI explains:**
**What I did:** Configured the REST client URL, scope, and TLS trust-all (for demo simplicity).
**How Quarkus helped:** REST client configuration is just three properties — host, scope, TLS. No code changes.
**Try it:** `quarkus build --native` (takes a few minutes), then (with the `weather` alias from Card 4) `weather get 41 29 --location=Istanbul` — expected: real temperature printed. Then `mvn clean package -Pnative-container`, then `podman run -it weather:latest temp --in Istanbul 41 29` — expected: same output from the containerized binary.

**You run:**
```bash
quarkus build --native
weather get 41 29 --location=Istanbul
mvn clean package -Pnative-container
podman run -it weather:latest temp --in Istanbul 41 29
```
Expected: Real temperature from Open-Meteo, works in native binary and container

---

## Demo Complete

You've built a production-ready Quarkus CLI that:
- Parses commands with picocli
- Calls a REST API with type-safe clients
- Compiles to a native binary
- Ships as a container image

All in 14 steps, with Quarkus handling the infrastructure.
