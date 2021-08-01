# Quarkus - Scala3

This extension provides support for Scala 3 in Quarkus.

It uses the `scala3-interfaces` library to avoid user lock-in to any specific version of the Scala 3 compiler (Dotty).
Instead, the reflection-based API is used and compilation is done by invoking the compiler version the user has installed from the runtime classpath.

For more information and background context on this, there are notes in the `Scala3CompilationProvider.java` file.

Additionally, passing compiler flags when in Dev Mode is supported through the use of an environment variable (`QUARKUS_SCALA3_COMPILER_ARGS`) which allows you to mirror your existing Maven/Gradle (at the time of writing Gradle does not yet support Scala 3) compilation configuration.


## Installation

If you want to use this extension, you need to add the `io.quarkiverse.scala:quarkus-scala3` extension first.
In your `pom.xml` file, add:

```xml
<dependency>
    <groupId>io.quarkiverse.scala</groupId>
    <artifactId>quarkus-scala3</artifactId>
</dependency>
```

Then, you will need to install the Scala 3 compiler, the Scala Maven plugin, and to fix an odd bug with the way that the Scala 3 compiler Maven dependencies are resolved.

For some reason, the wrong version of `scala-library` (a transitive dependency: `scala3-compiler_3` -> `scala3-library_3` -> `scala-library`) is resolved.

This causes binary incompatibilities -- and Scala to break. In order to fix this, you just need to manually align the version of `scala-library` to the one listed as used by the version of `scala3-library_3` that's the same as the `scala3-compiler_3` version.

So for `scala3-compiler_3` = `3.0.0`, then `scala3-library_3` = `3.0.0`, and we check the `scala-library` version it uses:
- https://mvnrepository.com/artifact/org.scala-lang/scala3-library_3/3.0.0

Here, we can see that it was compiled with `2.13.5` in it's dependencies. So that's what we set in ours:

```xml
<properties>
    <scala-maven-plugin.version>4.5.3</scala-maven-plugin.version>
    <scala.version>3.0.0</scala.version>
    <scala-library.version>2.13.5</scala-library.version>
</properties>

<dependencies>
    <!-- Scala Dependencies -->
    <dependency>
        <groupId>org.scala-lang</groupId>
        <artifactId>scala3-compiler_3</artifactId>
        <version>${scala.version}</version>
    </dependency>
    <dependency>
        <!-- Version manually aligned to scala3-library_3:3.0.0 dependency -->
        <groupId>org.scala-lang</groupId>
        <artifactId>scala-library</artifactId>
        <version>${scala-library.version}</version>
    </dependency>
</dependencies>

<build>
    <sourceDirectory>src/main/scala</sourceDirectory>
    <testSourceDirectory>src/test/scala</testSourceDirectory>

    <!-- REST OF CONFIG -->

    <plugin>
        <groupId>net.alchim31.maven</groupId>
        <artifactId>scala-maven-plugin</artifactId>
        <version>${scala-maven-plugin.version}</version>
        <executions>
            <execution>
                <id>scala-compile-first</id>
                <phase>process-resources</phase>
                <goals>
                    <goal>add-source</goal>
                    <goal>compile</goal>
                </goals>
            </execution>
            <execution>
                <id>scala-test-compile</id>
                <phase>process-test-resources</phase>
                <goals>
                    <goal>add-source</goal>
                    <goal>testCompile</goal>
                </goals>
            </execution>
        </executions>
        <configuration>
            <scalaVersion>${scala.version}</scalaVersion>
            <!-- Some solid defaults, change if you like -->
            <args>
                <arg>-deprecated</arg>
                <arg>-explain</arg>
                <arg>-feature</arg>
                <arg>-Ysafe-init</arg>
            </args>
        </configuration>
    </plugin>
</build>
```

Finally, the last thing you want to do is make sure that you mirror any compiler args you have set up when you run in Dev Mode.

To do this, just run the dev command with a prefix of the environment variable set. The format is comma-delimited:

```sh
QUARKUS_SCALA3_COMPILER_ARGS="-deprecated,-explain,-feature,-Ysafe-init" mvn quarkus:dev
```

You might save this as a bash/powershell/batch script for convenience.