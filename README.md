# spring-overclocked-utils

A Java library providing reflection-based object mapping/validation
utilities and a Spring Boot REST request/response layer (pagination,
generic JPA specifications, global exception handling) — published as a
**single artifact**.

```
io.amir:spring-overclocked-utils:1.0.5
```

## Why one artifact

The codebase is organized into two source directories for readability:

| Directory                              | What lives there                                                             |
|-----------------------------------------|-------------------------------------------------------------------------------|
| [`common/`](common)                     | Standalone reflection / object-copier / data-mapping / Bean Validation utils  |
| [`request-response/`](request-response) | Spring Boot request/response layer, built on top of `common`                  |

but they are **not** separate Gradle modules and are **not** published
separately. `build.gradle` wires both directories into one `sourceSet`, so
Gradle's normal `jar` task naturally produces a single jar that physically
contains every class from both — no shading/fat-jar plugin needed, since
nothing outside this project's own source is being merged in.

Third-party dependencies (Spring Boot, Jackson, Hibernate Validator, ...)
are **not** bundled into the jar — they remain normal Maven dependencies
declared in the published POM. Baking a Spring Boot web stack into a
library jar would break at runtime for every real consumer, who already has
their own Spring Boot on the classpath.

Net effect: consumers add exactly one line and get everything.

## Requirements

- Java 25+
- Gradle (via the included wrapper, `./gradlew`)

## Building

```bash
./gradlew build
```

Compiles both source directories as one module, runs tests, and assembles
the jar, sources jar, and javadoc jar. No Maven Central credentials are
needed for this — they're only required for the `publish*` tasks below.

## Using it in another project

### Gradle (Groovy DSL)

```groovy
dependencies {
    implementation "io.amir:spring-overclocked-utils:1.0.5"
}
```

### Gradle (Kotlin DSL)

```kotlin
dependencies {
    implementation("io.amir:spring-overclocked-utils:1.0.5")
}
```

### Maven

```xml
<dependency>
    <groupId>io.amir</groupId>
    <artifactId>spring-overclocked-utils</artifactId>
    <version>1.0.5</version>
</dependency>
```

### Directly from GitHub (without Maven Central)

If you'd rather consume this repository straight from GitHub instead of
Maven Central, use [JitPack](https://jitpack.io/), which builds and hosts
tagged commits of any public GitHub repo on demand — no publishing setup
required on your end.

```groovy
repositories {
    maven { url 'https://jitpack.io' }
}

dependencies {
    implementation 'com.github.Amir-UL-Islam:request-response:<tag-or-commit>'
}
```

Push a Git tag (e.g. `v1.0.0`) and reference that tag as the version.

## Publishing a release to Maven Central

Publishing is handled by the
[`com.vanniktech.maven.publish`](https://github.com/vanniktech/gradle-maven-publish-plugin)
plugin, targeting Sonatype's **Central Portal** — the only supported route
for new releases since OSSRH (`oss.sonatype.org` / `s01.oss.sonatype.org`)
was retired on 2025-06-30.

### One-time setup

1. Create a [Central Portal account](https://central.sonatype.org/register/central-portal/)
   and [register the `io.amir` namespace](https://central.sonatype.org/register/namespace/).
2. Generate a [user token](https://central.sonatype.org/publish/generate-portal-token/)
   on the Central Portal — this gives you a username/password pair used
   only for publishing, not your login credentials.
3. Create a GPG key pair and
   [distribute the public key](https://central.sonatype.org/publish/requirements/gpg/#distributing-your-public-key)
   to a public key server (Maven Central verifies the signature against it).
4. Export the private key in ASCII-armored form for Gradle's in-memory
   signing:
   ```bash
   gpg --export-secret-keys --armor <key-id> > private-key.asc
   ```

**Never commit any of the values below to the repository.** Provide them
either in `~/.gradle/gradle.properties` (not this project's own
`gradle.properties`) or as environment variables, e.g. in CI:

```properties
mavenCentralUsername=<central portal user token username>
mavenCentralPassword=<central portal user token password>

signing.keyId=<last 8 chars of your GPG key id>
signing.password=<GPG key passphrase, if any>
signing.secretKeyRingFile=/absolute/path/to/secring.gpg
```

or, as environment variables (typical for CI):

```bash
export ORG_GRADLE_PROJECT_mavenCentralUsername=...
export ORG_GRADLE_PROJECT_mavenCentralPassword=...
export ORG_GRADLE_PROJECT_signingInMemoryKey="$(cat private-key.asc)"
export ORG_GRADLE_PROJECT_signingInMemoryKeyPassword=...
```

### Releasing

1. Bump `version` in `gradle.properties`.
2. Upload and manually publish from the Central Portal UI:
   ```bash
   ./gradlew publishToMavenCentral
   ```
   then visit [Deployments](https://central.sonatype.com/publishing/deployments)
   and click **Publish**.

   — or, to upload *and* release in one step, set
   `mavenCentralAutomaticPublishing=true` in `gradle.properties` (or pass
   `publishToMavenCentral(true)` in the DSL), then run the same command.
3. Artifacts typically become available on Maven Central 10–30 minutes
   after publishing completes.

## Project layout

```
spring-overclocked-utils/
├── build.gradle             # the single build file
├── settings.gradle
├── gradle.properties        # versions & coordinates
├── LICENSE                  # Apache License 2.0
├── common/
│   └── src/main/java/...    # plain source directory, not a Gradle module
└── request-response/
    └── src/main/java/...    # plain source directory, not a Gradle module
```

## License

Apache License 2.0 — see [LICENSE](LICENSE).
