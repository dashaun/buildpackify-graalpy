# paketo-graalpy


### Prerequisites
- [SDKman](https://sdkman.io)
- Docker
- [Pack](https://github.com/buildpacks/pack) 

### Getting Started

```bash
# Initialize the shell with GraalVM and Maven
sdk env install
# Build the GraalPy native image
./mvnw -Pnative package
# Execute the native version of the Python code: hello.py
./target/buildpackify
```

### Create an optimized OCI Image

```bash
pack build buildpackify \
--builder paketobuildpacks/builder-jammy-base \
--buildpack paketo-buildpacks/syft \
--buildpack paketo-buildpacks/graalvm \
--buildpack paketo-buildpacks/java \
--buildpack paketo-buildpacks/maven \
--env BP_MAVEN_ACTIVE_PROFILES=native \
--env BP_MAVEN_BUILT_ARTIFACT=target/buildpackify-1.0-SNAPSHOT.zip \
--env BP_NATIVE_IMAGE=true \
--env MVN=/layers/paketo-buildpacks_maven/maven/bin/mvn
```

### What you need to know

The project creates a `virtual file system` that works like a Python `venv` but inside the JVM.
All of Python config is set up as part of the `context` with the `GraalPy` class.

The `GraalPy` class implements the `Hello` interface using the Python code in `hello.py`.

The Java code can access the `native` version of the Python code using the provided `polyglot API`

### Background Details

These are the steps I took to set up this project. Provided for informational purposes only.

```bash
sdk install java 21.0.3-graal
sdk use java 21.0.3-graal
sdk install maven
sdk env init
```

```bash
mvn archetype:generate \
  -DarchetypeGroupId=org.graalvm.python \
  -DarchetypeArtifactId=graalpy-archetype-polyglot-app \
  -DarchetypeVersion=24.0.0
```
> groupId: com.dashaun.paketo.graalpy
> artifactId: buildpackify
> version: 1.0-SNAPSHOT
> package: com.dashaun.paketo.graalpy

Now we have a `maven` project that includes some simple `python` code and is ready to go!

### Info from the archetype generation

This simple project is meant as a jumping off point for a polyglot Python Java application on GraalVM.

To build a standalone native executable you need to use a [GraalVM JDK with Native Image](https://www.graalvm.org/downloads/) and run:

```
mvn package -Pnative
```
