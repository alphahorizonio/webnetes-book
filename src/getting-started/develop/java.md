# Java

## Table of Contents

<!-- toc -->

## Language Support Status

| Native | WebAssembly | Networking | Concurrency |
| ------ | ----------- | ---------- | ----------- |
| ✅     | ✅          | ❌[^1]     | ✅          |

## Hello, world!

Let's start by writing a hello world app in Java with native and WebAssembly targets. The full source code of this example is available on [GitHub](https://github.com/alphahorizonio/webnetes/tree/main/examples/rust_hello_world).

### POM

Create `pom.xml` with the following content:

<details>
	<summary>XML Source</summary>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>dev.webnetes.examples.java.hello_world</groupId>
    <artifactId>hello_world</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>jar</packaging>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.8</java.version>
        <teavm.version>0.7.0-dev-1145</teavm.version>
        <mainClass>dev.webnetes.examples.java.hello_world.Application</mainClass>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.teavm</groupId>
            <artifactId>teavm-classlib</artifactId>
            <version>${teavm.version}</version>
        </dependency>
        <dependency>
            <groupId>org.teavm</groupId>
            <artifactId>teavm-metaprogramming-impl</artifactId>
            <version>${teavm.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-dependency-plugin</artifactId>
            <version>3.1.2</version>
            <type>maven-plugin</type>
        </dependency>
    </dependencies>

    <repositories>
        <repository>
            <id>central</id>
            <name>Maven Central</name>
            <layout>default</layout>
            <url>https://repo1.maven.org/maven2</url>
        </repository>

        <repository>
            <id>teavm-dev</id>
            <url>https://dl.bintray.com/konsoletyper/teavm</url>
        </repository>
    </repositories>

    <pluginRepositories>
        <pluginRepository>
            <id>central</id>
            <name>Maven Central</name>
            <layout>default</layout>
            <url>https://repo1.maven.org/maven2</url>
        </pluginRepository>

        <pluginRepository>
            <id>teavm-dev</id>
            <url>https://dl.bintray.com/konsoletyper/teavm</url>
        </pluginRepository>
    </pluginRepositories>

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.2.4</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <manifestEntries>
                                        <Main-Class>${mainClass}</Main-Class>
                                    </manifestEntries>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

            <plugin>
                <groupId>org.teavm</groupId>
                <artifactId>teavm-maven-plugin</artifactId>
                <version>${teavm.version}</version>
                <executions>
                    <execution>
                        <id>web-client</id>
                        <phase>prepare-package</phase>
                        <goals>
                            <goal>compile</goal>
                        </goals>
                        <configuration>
                            <mainClass>${mainClass}</mainClass>
                            <optimizationLevel>FULL</optimizationLevel>
                            <targetType>WEBASSEMBLY</targetType>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

</details>

Don't forget to change `groupId` etc. to more appropriate values.

### Source Code

Create `src/main/java/dev/webnetes/examples/java/hello_world/Application.java` with the following content:

```java
package dev.webnetes.examples.java.hello_world;

public class Application {
    public static void main(String[] args) {
        System.out.println("Hello, world!");
    }
}
```

It prints `Hello, world!` to standard output.

### Make Configuration

Create a `Makefile` with the following content:

```Makefile
all:
	@docker run -v ${PWD}:/src:z -v ${PWD}/.m2:/root/.m2:z maven sh -c 'cd /src && mvn clean install'
	@docker run -v ${PWD}:/src:z alphahorizonio/wasi-sdk sh -c 'cd /src && wasm-opt --asyncify -O target/javascript/classes.wasm -o target/javascript/classes.wasm'

clean:
	@rm -rf target

seed: all
	@docker run -it -v ${PWD}/target:/target:z --entrypoint=/bin/sh schaurian/webtorrent-hybrid -c "/usr/local/bin/webtorrent-hybrid seed /target/javascript/classes.wasm"
```

It provides three targets:

- `make` builds both the native & the WebAssembly target
- `make clean` removes the built targets
- `make seed` seeds the WebAssembly target, which will come in handy later.

All of them use Docker, so there is no need to install any dependencies on the host.

### Next Steps

Build the native & the WebAssembly target:

```shell
$ make
```

Run the native target:

```shell
$ java -jar target/hello_world-1.0-SNAPSHOT.jar
Hello, world!
```

The native target works! Now continue to [Distribute](../distribute.md) to learn how to run the WebAssembly target.

[^1]: No Java bindings to [unisockets](https://github.com/alphahorizonio/unisockets) have been implemented yet.
