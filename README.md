[inverno-core-root]: https://github.com/inverno-io/inverno-core
[inverno-core-root-doc]: https://github.com/inverno-io/inverno-core/tree/master/doc/reference-guide.md
[inverno-mods-root]: https://github.com/inverno-io/inverno-mods
[inverno-mods-root-doc]: https://github.com/inverno-io/inverno-mods/tree/master/doc/reference-guide.md
[inverno-oss-parent]: https://github.com/inverno-io/inverno-oss-parent
[inverno-tool-maven-plugin]: https://github.com/inverno-io/inverno-tools/tree/master/inverno-maven-plugin

[jdk]: https://jdk.java.net/
[maven]: https://maven.apache.org/download.cgi
[log4j-2]: https://logging.apache.org/log4j/2.x/index.html
[log4j-2-config]: https://logging.apache.org/log4j/2.x/manual/configuration.html
[apache-license]: https://www.apache.org/licenses/LICENSE-2.0

# Inverno Distribution

[![CI/CD](https://github.com/inverno-io/inverno-dist/actions/workflows/maven.yml/badge.svg)](https://github.com/inverno-io/inverno-dist/actions/workflows/maven.yml)

The Inverno distribution provides a parent POM `io.inverno.dist:inverno-parent` and a BOM `io.inverno.dist:inverno-dependencies` for developing Inverno components and applications.

The parent POM inherits from the BOM which inherits from the [Inverno OSS parent][inverno-oss-parent] POM. It provides basic build configuration for building Inverno components and applications, including dependency management and plugins configuration. It especially includes configuration for the [Inverno Maven plugin][inverno-tool-maven-plugin].

The BOM specifies the [Inverno core][inverno-core-root] and [Inverno modules][inverno-mods-root] dependencies as well as OSS dependencies.

The Inverno distribution thus defines a consistent sets of dependencies and configuration for developing, building, packaging and distributing Inverno components and applications. Upgrading the Inverno framework version of a project boils down to upgrade the Inverno distribution version which is the version of the Inverno parent POM or the Inverno BOM.

## Requirements

The Inverno framework requires [JDK][jdk] 15 or later and [Apache Maven][maven] 3.6.0 or later.

> The Inverno compiler (when displaying bean dependency cycles), the Inverno tools (when displaying the progress bar) and the standard application banner (displayed when bootstrapping an Inverno application) output unicode characters which are supported out of the box by Linux or MacOS terminals but unfortunately not by the Windows terminal for which the unicode support must be enabled explicitly, this can be done in Regional Settings > Administrative > Change System Local > Use Unicode UTF-8 for worldwide language support. Another viable solution is to use the Git bash on Windows which also supports unicode out of the box. Please note that this is purely cosmetic and has no impact on the applications.

## Creating an Inverno project

The recommended way to start a new Inverno project is to create a Maven project which inherits from the `io.inverno.dist:inverno-parent` project, we might also want to add a dependency to `io.inverno:inverno-core` in order to create an Inverno module with IoC/DI:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>io.inverno.dist</groupId>
        <artifactId>inverno-parent</artifactId>
        <version>${VERSION_INVERNO_DIST}</version>
    </parent>
    <groupId>io.inverno.example</groupId>
    <artifactId>sample-app</artifactId>
    <version>1.0.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>io.inverno</groupId>
            <artifactId>inverno-core</artifactId>
        </dependency>
    </dependencies>
</project>
```

That is all we need to develop, run, build, package and distribute a basic Inverno component or application. The Inverno parent POM provides dependency management and Java compiler configuration to invoke the Inverno compiler during the build process as well as Inverno tools configuration to be able to run and package the Inverno component or application.

If it is not possible to inherit from the Inverno parent POM, we can also declare the Inverno BOM `io.inverno.dist:inverno-dependencies` in the `<dependencyManagement/>` section to benefit from dependency management but loosing plugins configuration which must then be recovered from the Inverno parent POM.

```xml
<project>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>io.inverno.dist</groupId>
                <artifactId>inverno-dependencies</artifactId>
                <version>${VERSION_INVERNO_DIST}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

Inverno modules dependencies can be added in the `<dependencies/>` section of the project POM. For instance the following dependencies can be added to develop a REST microservice application:

```xml
<project>
    <dependencies>
        <dependency>
            <groupId>io.inverno.mod</groupId>
            <artifactId>inverno-boot</artifactId>
        </dependency>
        <dependency>
            <groupId>io.inverno.mod</groupId>
            <artifactId>inverno-web</artifactId>
        </dependency>
    </dependencies>
</project>
```

Please refer to the [Inverno core documentation][inverno-core-root-doc] and [Inverno modules documentation][inverno-mods-root-doc] to learn how to develop with IoC/DI and how to use Inverno modules.

### Developing a simple Inverno application

We can now start developing a sample REST application. An Inverno component or application is a regular Java module annotated with `@io.inverno.core.annotation.Module`, so the first thing we need to do is to create Java module descriptor `module-info.java` under `src/main/java` which is where Maven finds the sources to compile.

```java
@io.inverno.core.annotation.Module
module io.inverno.example.sample_app {
    requires io.inverno.mod.boot;
    requires io.inverno.mod.web.server;
}
```

Note that we declared the `io.inverno.mod.boot` and `io.inverno.mod.web.server` module dependencies since we want to create a REST application, please refer to the [Inverno modules documentation][inverno-mods-root-doc] to learn more.

We then can create the main class of our sample REST application in `src/main/java/io/inverno/example/sample_app/App.java`:

```java
package io.inverno.example.sample_app;

import io.inverno.core.annotation.Bean;
import io.inverno.core.v1.Application;
import io.inverno.mod.base.resource.MediaTypes;
import io.inverno.mod.web.server.annotation.WebController;
import io.inverno.mod.web.server.annotation.WebRoute;

@Bean
@WebController
public class App {

    @WebRoute( path = "/message", produces = MediaTypes.TEXT_PLAIN)
    public String getMessage() {
        return "Hello, world!";
    }

    public static void main(String[] args) {
        Application.with(new Sample_app.Builder()).run();
    }
}
```

### Configuring logging

Inverno framework is using [Log4j 2][log4j-2] for logging, Inverno application logging can be activated by adding the dependency to `org.apache.logging.log4j:log4j-core`:

```xml
<project>
    <dependencies>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
        </dependency>
    </dependencies>
</project>
```

> If you don't include this dependency at runtime, Log4j falls back to the `SimpleLogger` implementation provided with the API and configured using `org.apache.logging.log4j.simplelog.*` system properties. The log level can then be configured by setting `-Dorg.apache.logging.log4j.simplelog.level=INFO` system property when running the application.

Log4j 2 provides a default configuration with a default root logger level set to `ERROR`, resulting in no info messages being output when starting an application. This can be changed by setting `-Dorg.apache.logging.log4j.level=INFO` system property when running the application.

However the recommended way is to provide a specific `log4j2.xml` logging configuration file in the project resources under `src/main/resources`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration xmlns="http://logging.apache.org/log4j/2.0/config"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://logging.apache.org/log4j/2.0/config https://raw.githubusercontent.com/apache/logging-log4j2/rel/2.14.0/log4j-core/src/main/resources/Log4j-config.xsd"
    status="WARN" shutdownHook="disable">

    <Appenders>
        <Console name="LogToConsole" target="SYSTEM_OUT">
             <PatternLayout pattern="%d{DEFAULT} %highlight{%-5level} [%t] %c{1.} - %msg%n%ex"/>
        </Console>
    </Appenders>
    <Loggers>
        <Root level="info">
            <AppenderRef ref="LogToConsole"/>
        </Root>
    </Loggers>
</Configuration>
```

> Note that the Log4j shutdown hook must be disabled so as not to interfere with the Inverno application shutdown hook, if it is not disabled, application shutdown logs might be dropped.

> We could have chosen to provide a default logging configuration in the Inverno framework itself, but we preferred to stick to standard Log4j 2 configuration rules in order to keep things simple so please refer to the [Log4j 2 configuration documentation][log4j-2-config] to learn how to configure logging.

### Running the application

The application is now ready and can be run using the `inverno:run` goal:

```plaintext
$ mvn inverno:run

...
[INFO] --- inverno:${VERSION_INVERNO_TOOLS}:run (default-cli) @ sample-app ---
[INFO] Running project: io.inverno.example.sample_app@1.0.0-SNAPSHOT...
 [═══════════════════════════════════════════════ 100 % ══════════════════════════════════════════════] 
2021-04-08 23:50:35,261 INFO  [main] i.w.c.v.Application - Inverno is starting...


     ╔════════════════════════════════════════════════════════════════════════════════════════════╗
     ║                      , ~~ ,                                                                ║
     ║                  , '   /\   ' ,                                                            ║
     ║                 , __   \/   __ ,      _                                                    ║
     ║                ,  \_\_\/\/_/_/  ,    | |  ___  _    _  ___   __  ___   ___                 ║
     ║                ,    _\_\/_/_    ,    | | / _ \\ \  / // _ \ / _|/ _ \ / _ \                ║
     ║                ,   __\_/\_\__   ,    | || | | |\ \/ /|  __/| | | | | | |_| |               ║
     ║                 , /_/ /\/\ \_\ ,     |_||_| |_| \__/  \___||_| |_| |_|\___/                ║
     ║                  ,     /\     ,                                                            ║
     ║                    ,   \/   ,                                  -- ${VERSION_INVERNO_CORE} --                 ║
     ║                      ' -- '                                                                ║
     ╠════════════════════════════════════════════════════════════════════════════════════════════╣
     ║ Java runtime        : OpenJDK Runtime Environment                                          ║
     ║ Java version        : 16+36-2231                                                           ║
     ║ Java home           : /home/jkuhn/Devel/jdk/jdk-16                                         ║
     ║                                                                                            ║
     ║ Application module  : io.inverno.example.sample_app                                        ║
     ║ Application version : 1.0.0-SNAPSHOT                                                       ║
     ║ Application class   : io.inverno.example.sample_app.App                                    ║
     ║                                                                                            ║
     ║ Modules             :                                                                      ║
     ║  * ...                                                                                     ║
     ╚════════════════════════════════════════════════════════════════════════════════════════════╝


2021-04-08 23:50:35,266 INFO  [main] i.w.e.s.Sample_app - Starting Module io.inverno.example.sample_app...
2021-04-08 23:50:35,266 INFO  [main] i.w.m.b.Boot - Starting Module io.inverno.mod.boot...
2021-04-08 23:50:35,446 INFO  [main] i.w.m.b.Boot - Module io.inverno.mod.boot started in 179ms
2021-04-08 23:50:35,446 INFO  [main] i.w.m.w.Web - Starting Module io.inverno.mod.web.server...
2021-04-08 23:50:35,446 INFO  [main] i.w.m.h.s.Server - Starting Module io.inverno.mod.http.server...
2021-04-08 23:50:35,446 INFO  [main] i.w.m.h.b.Base - Starting Module io.inverno.mod.http.base...
2021-04-08 23:50:35,450 INFO  [main] i.w.m.h.b.Base - Module io.inverno.mod.http.base started in 3ms
2021-04-08 23:50:35,545 INFO  [main] i.w.m.h.s.i.HttpServer - HTTP Server (nio) listening on http://0.0.0.0:8080
2021-04-08 23:50:35,546 INFO  [main] i.w.m.h.s.Server - Module io.inverno.mod.http.server started in 99ms
2021-04-08 23:50:35,546 INFO  [main] i.w.m.w.Web - Module io.inverno.mod.web.server started in 99ms
2021-04-08 23:50:35,546 INFO  [main] i.w.e.s.Sample_app - Module io.inverno.example.sample_app started in 281ms

```

We can now test the application:

```plaintext
$ curl http://127.0.0.1:8080/message
Hello, world!
```

The application can be gracefully shutdown by pressing `Ctrl-c`.

### Packaging the application image

In order to create a native image containing the application and all its dependencies including JDK's dependencies, we can simply invoke the `inverno:package-app` goal:

```plaintext
$ mvn inverno:package-app

...
 [INFO] --- inverno:${VERSION_INVERNO_TOOLS}:package-app (default-cli) @ sample-app ---
[INFO] Building application image: /home/jkuhn/Devel/git/frmk/io.inverno.example.sample-app/target/maven-inverno/application_linux_amd64/sample-app-1.0.0-SNAPSHOT...
 [═══════════════════════════════════════════════  69 % ═══════════════>                              ] Packaging project application...
```

> This uses `jpackage` tool which is an incubating feature in JDK<16, if you intend to build an application image with an old JDK, you'll need to explicitly add the `jdk.incubator.jpackage` module in `MAVEN_OPTS`:
> ```plaintext
> $ export MAVEN_OPTS="--add-modules jdk.incubator.jpackage"
> ```

This will create a ZIP archive containing a native application distribution `target/sample-app-1.0.0-SNAPSHOT-application_linux_amd64.zip` which will be deployed to the local Maven repository and eventually to a remote Maven repository.

Then in order to install the application on a compatible platform, we just need to download the archive corresponding to the platform, extract it to some location and run the application. Luckily for us this can be done quite easily with Maven dependency plugin:

```plaintext
$ mvn dependency:unpack -Dartifact=io.inverno.example:sample-app:1.0.0-SNAPSHOT:zip:application_linux_amd64 -DoutputDirectory=./
...
$ ./sample-app-1.0.0-SNAPSHOT/bin/sample-app
...
```

It is also possible to create platform specific package such as `.deb` or a `.msi` by defining particular formats in the Inverno Maven plugin configuration:

```xml
<project>
    <build>
        <plugins>
            <plugin>
                <groupId>io.inverno.tool</groupId>
                <artifactId>inverno-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <id>package-app</id>
                        <phase>package</phase>
                        <goals>
                            <goal>package-app</goal>
                        </goals>
                        <configuration>
                            <formats>
                                <format>zip</format>
                                <format>deb</format>
                            </formats>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

```plaintext
$ mvn package
...
```

> Note that there is no cross-platform support and a given platform specific format must be built on the platform it runs on.

Such platform-specific package can then be downloaded and installed using the right package manager:

```plaintext
$ mvn dependency:copy -Dartifact=io.inverno.example:sample-app:1.0.0-SNAPSHOT:deb:application_linux_amd64 -DoutputDirectory=./
...
$ sudo dpkg -i sample-app-1.0.0-SNAPSHOT-application_linux_amd64.deb
...
$ /opt/sample-app/bin/sample-app
...
```

The Inverno Maven plugin allows to create various application images including Docker or OCI container images, please refer to the [Inverno Maven plugin documentation][inverno-tool-maven-plugin] to learn more.

## License

The Inverno Framework is released under version 2.0 of the [Apache License][apache-license].

