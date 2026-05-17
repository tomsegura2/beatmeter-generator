# Building Beatmeter Generator from source

This guide explains how to compile, run, and package Beatmeter Generator from a source checkout.

## Build overview

Beatmeter Generator is a Scala desktop application built with [sbt](https://www.scala-sbt.org/). The project is configured for:

- Scala 2.12.8
- sbt 1.2.8
- JavaFX 11 modules for Linux, macOS, and Windows
- `sbt-assembly` for producing a runnable fat JAR named `beatmeter-generator.jar`

The application entry point is `io.gitlab.sklavedaniel.beatmetergenerator.editor.BeatEditor`.

## Prerequisites

Install these tools before building:

1. **Git** for cloning the repository.
2. **JDK 11**. The project depends on JavaFX 11 artifacts and the optional runtime-image command in `generate-runtime.txt` is written for the Java 11 module set. Newer JDKs may work for compilation, but JDK 11 is the safest baseline.
3. **sbt**. The repository pins the sbt launcher version in `project/build.properties`; any normal sbt installation should download and use that version automatically.
4. **JWave source tree** for BPM detection. The sbt build adds `lib/JWave/src` as an unmanaged source directory, so that directory must contain the `jwave` Java package before compiling.

On Debian/Ubuntu-like systems, the system packages usually look similar to:

```sh
sudo apt-get update
sudo apt-get install git openjdk-11-jdk sbt
```

If your distribution does not provide sbt, install it using the official sbt installation instructions for your platform.

## Clone the repository

```sh
git clone <repository-url> beatmeter-generator
cd beatmeter-generator
```

If you are already working from a checkout, run all remaining commands from the repository root.

## Add the JWave sources

Create the unmanaged source directory expected by `build.sbt` and put the JWave `src` contents there:

```sh
mkdir -p lib
# Example: after obtaining a JWave source checkout or archive
cp -R /path/to/JWave/src lib/JWave/src
```

After this step, a file such as `lib/JWave/src/jwave/Transform.java` should exist. If it does not, `sbt compile` will fail with missing `jwave` imports from `WaveletBPMDetection.scala`.

## Compile

```sh
sbt compile
```

The first run downloads Scala, sbt plugins, JavaFX artifacts, and the library dependencies declared in `build.sbt`, so it can take several minutes.

## Run from source

```sh
sbt run
```

The build forks the application into a separate JVM. On headless servers or CI runners, launching the GUI may fail unless a graphical display is available.

## Run checks

The repository includes a Scalastyle configuration and plugin. Run it with:

```sh
sbt scalastyle
```

There is no dedicated test suite in this repository at the time of writing. Use `sbt compile` and `sbt scalastyle` as the baseline source checks.

## Build the runnable JAR

```sh
sbt assembly
```

The assembled application is written to:

```text
target/scala-2.12/beatmeter-generator.jar
```

Run it with:

```sh
java -jar target/scala-2.12/beatmeter-generator.jar
```

Use a JDK/JRE compatible with JavaFX 11. If the application cannot initialize JavaFX media or graphics on your platform, prefer running with JDK 11 and verify that your desktop session has audio and display support.

## Optional: create a stripped runtime image

The repository includes `generate-runtime.txt` with a `jlink` command for creating a local Java runtime image:

```sh
jlink --no-header-files --no-man-pages --add-modules java.desktop,jdk.jsobject,jdk.localedata,jdk.net,jdk.unsupported,jdk.unsupported.desktop --output java-runtime
```

Run that command with a JDK that provides `jlink`. The resulting `java-runtime` directory can be used as a starting point when preparing a self-contained distribution.

## Optional: build a Windows MSI package

The sbt build defines a custom `packageMSI` task. It is intended for a prepared packaging environment and expects:

- a Windows JRE staged at `/root/jre-11-win-64/`
- `wixl` available on `PATH`
- the assembled JAR produced by the `assembly` task

Run it with:

```sh
sbt packageMSI
```

If those packaging prerequisites are missing, the task will fail. For normal development builds, use `sbt assembly` instead.

## Troubleshooting

### `object jwave is not a member of package` or `not found: type Transform`

The JWave unmanaged sources are missing or copied to the wrong directory. Confirm that the JWave package is located under `lib/JWave/src/jwave`.

### JavaFX or GUI startup errors

Use JDK 11 first, then confirm that you are running in a graphical desktop session. The application is a JavaFX desktop application and is not expected to launch successfully in a headless-only shell.

### Dependency download failures

Re-run the sbt command after checking network access. sbt must be able to download dependencies from the resolvers declared in `build.sbt` and the plugin artifacts declared under `project/`.
