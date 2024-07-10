# Buildpackify-GraalPy

- your Python code isn't running as awesome as it could be.
- your data scientists shouldn't be creating Dockerfile for production.
- you would like to save some money on your application infrastructure.
- you want to sleep well, instead of worrying about production container security.

This project tries to help.

[Paketo](https://paketo.io) makes amazing buildpacks for multiple languages, including Python.
For one use case, I recommended using the [Paketo buildpack for Python](https://paketo.io/docs/howto/python/) to a customer, to replace their Dockerfile.
The buildpack worked, but the resulting container was still multiple GB in size.

We needed another way.

We decided to bring in [GraalPy](https://www.graalvm.org/python/) and give it a try.
Of course it worked, GraalPy was able to optimize the Python project.
But, it didn't help with creating a production-ready, enterprise-grade container.

```text
Friends don't let friends use Dockerfile.
```

So we used the GraalPy `Maven Archetype` to generate a project, that we could deploy with Paketo buildpacks.

This project takes `Python` wraps it with `Java` and connects them via the polyglot API and creates an OCI image using Paketo Buildpacks.

### Prerequisites
- [Docker](https://docker.com)
- [Pack](https://github.com/buildpacks/pack) 

### Getting Started - Create an optimized OCI Image for Python

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
> Now you have an OCI image to run

```bash
# Run the OCI image
docker run buildpackify
```
> Q.E.D.

### Development Setup

- Using [SDKman](https://sdkman.io)

```bash
# Initialize the shell with GraalVM and Maven
sdk env install
# Build the GraalPy native image
mvn -Pnative package
# Execute the native version of the Python code: hello.py
./target/buildpackify
```

### Quick recap

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
> - groupId: com.dashaun.paketo.graalpy
> - artifactId: buildpackify
> - version: 1.0-SNAPSHOT
> - package: com.dashaun.paketo.graalpy

Now we have a `maven` project that includes some simple `python` code and is ready to go!

### Info from the archetype generation

This simple project is meant as a jumping off point for a polyglot Python Java application on GraalVM.

To build a standalone native executable you need to use a [GraalVM JDK with Native Image](https://www.graalvm.org/downloads/) and run:

```
mvn package -Pnative
```

### The additional steps I took to go from Archetype to Buildpackified

- Added Procfile
- Added assembly config, xml and plugin