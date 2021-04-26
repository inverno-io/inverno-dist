# Winter Distribution

The Winter distribution provides a parent POM `io.winterframework.dist:winter-parent` and a BOM `io.winterframework.dist:winter-dependencies` for developing Winter components and applications.

The parent POM inherits from the BOM which inherits from the [Winter OSS parent](https://github.com/winterframework-io/winter-oss-parent) POM. It provides basic build configuration for building Winter components and applications, including dependency management and plugins configuration. It especially includes configuration for the [Winter Maven plugin](https://github.com/winterframework-io/winter-tools/tree/master/winter-maven-plugin).

The BOM specifies the [Winter core](https://github.com/winterframework-io/winter) and [Winter modules](https://github.com/winterframework-io/winter-mods) dependencies as well as OSS dependencies.

The Winter distribution thus defines a consistent sets of dependencies and configuration for developing, building, packaging and distributing Winter components and applications. Upgrading the Winter framework version of a project boils down to upgrade the Winter distribution version which is the version of the Winter parent POM or the Winter BOM.

## Requirements

The Winter framework requires [JDK](https://jdk.java.net/) 9 or later and [Apache Maven](https://maven.apache.org/download.cgi) 3.6.0 or later.

## Creating a Winter project

### Project POM

The recommended way to start a new Winter project is to create a Maven project which inherits from the `io.winterframework.dist:winter-parent` project, we might also want to add a dependency to `io.winterframework:winter-core` in order to create a Winter module with IoC/DI:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>io.winterframework.dist</groupId>
        <artifactId>winter-parent</artifactId>
        <version>1.0.0</version>
    </parent>
    <groupId>io.winterframework.example</groupId>
    <artifactId>sample-app</artifactId>
    <version>1.0.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>io.winterframework</groupId>
            <artifactId>winter-core</artifactId>
        </dependency>
    </dependencies>
</project>
```

That is all we need to develop, run, build, package and distribute a basic Winter component or application. The Winter parent POM provides dependency management and Java compiler configuration to invoke the Winter compiler during the build process as well as Winter tools configuration to be able to run and package the Winter component or application.

If it is not possible to inherit from the Winter parent POM, we can also declare the Winter BOM `io.winterframework.dist:winter-dependencies` in the `<dependencyManagement/>` section to benefit from dependency management but loosing plugins configuration which must then be recovered from the Winter parent POM.

```xml
<project>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>io.winterframework.dist</groupId>
                <artifactId>winter-dependencies</artifactId>
                <version>1.0.0</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

Winter modules dependencies can be added in the `<dependencies/>` section of the project POM. For instance the following dependencies can be added to develop a REST microservice application:

```xml
<project>
    <dependencies>
        <dependency>
            <groupId>io.winterframework.mod</groupId>
            <artifactId>winter-boot</artifactId>
        </dependency>
        <dependency>
            <groupId>io.winterframework.mod</groupId>
            <artifactId>winter-web</artifactId>
        </dependency>
    </dependencies>
</project>
```

Please refer to the [Winter core documentation](https://github.com/winterframework-io/winter/tree/master/doc/reference-guide.md) and [Winter modules documentation](https://github.com/winterframework-io/winter-mods/tree/master/doc/reference-guide.md) to learn how to develop with IoC/DI and how to use Winter modules.

### Developing a sample Winter application

We can now start developing a sample REST application. A Winter component or application is a regular Java module annotated with `@io.winterframework.core.annotation.Module`, so the first thing we need to do is to create Java module descriptor `module-info.java` under `src/main/java` which is where Maven finds the sources to compile.

```java
@io.winterframework.core.annotation.Module
module io.winterframework.example.sample_app {
    requires io.winterframework.mod.boot;
    requires io.winterframework.mod.web;
}
```

Note that we declared the `io.winterframework.mod.boot` and `io.winterframework.mod.web` module dependencies since we want to create a REST application, please refer to the [Winter modules documentation](https://github.com/winterframework-io/winter-mods/tree/master/doc/reference-guide.md) to learn more.

We then can create the main class of our sample REST application in `src/main/java/io/winterframework/example/sample_app/App.java`:

```java
package io.winterframework.example.sample_app;

import io.winterframework.core.annotation.Bean;
import io.winterframework.core.v1.Application;
import io.winterframework.mod.base.resource.MediaTypes;
import io.winterframework.mod.web.annotation.WebController;
import io.winterframework.mod.web.annotation.WebRoute;

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

Winter framework is using [Log4j 2](https://logging.apache.org/log4j/2.x/index.html) for logging, Winter application logging can be activated by adding the dependency to `org.apache.logging.log4j:log4j-core`:

```
<project>
    <dependencies>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
        </dependency>
    </dependencies>
</project>
```

Log4j 2 provides a default configuration with a default root logger level set to `ERROR`, resulting in no info messages being output when starting an application. This can be changed by setting `-Dorg.apache.logging.log4j.level=INFO` system property when running the application.

> If you don't include this dependency at runtime, Log4j falls back to the `SimpleLogger` implementation provided with the API and configured using `org.apache.logging.log4j.simplelog.*` system properties. The log level can then be configured by setting `-Dorg.apache.logging.log4j.simplelog.level=INFO` system property when running the application.

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

> Note that the Log4j shutdown hook must be disabled so as not to interfere with the Winter application shutdown hook, if it is not disabled, application shutdown logs might be dropped.

> We could have chosen to provide a default logging configuration in the Winter framework itself, but we preferred to stick to standard Log4j 2 configuration rules in order to keep things simple so please refer to the [Log4j 2 configuration documentation](https://logging.apache.org/log4j/2.x/manual/configuration.html) to learn how to configure logging.

### Running the application

The application is now ready and can be run using the `winter:run` goal:

```
$ mvn winter:run

...
[INFO] --- winter-maven-plugin:1.0.0-SNAPSHOT:run (default-cli) @ sample-app ---
[INFO] Running project: io.winterframework.example.sample_app@1.0.0-SNAPSHOT...
 [═══════════════════════════════════════════════ 100 % ══════════════════════════════════════════════] 
2021-04-08 23:50:35,261 INFO  [main] i.w.c.v.Application - Winter is starting...


     ╔════════════════════════════════════════════════════════════════════════════════════════════╗
     ║                       , ~~ ,                                                               ║
     ║                   , '   /\   ' ,                 _                                         ║
     ║                  , __   \/   __ ,       _     _ (_)        _                               ║
     ║                 ,  \_\_\/\/_/_/  ,     | | _ | | _   ___  | |_   ___   __                  ║
     ║                 ,    _\_\/_/_    ,     | |/_\| || | / _ \ | __| / _ \ / _|                 ║
     ║                 ,   __\_/\_\__   ,     \  / \  /| || | | || |_ |  __/| |                   ║
     ║                  , /_/ /\/\ \_\ ,       \/   \/ |_||_| |_| \__| \___||_|                   ║
     ║                   ,     /\     ,                                                           ║
     ║                     ,   \/   ,                        -- 1.0.2 --                          ║
     ║                       ' -- '                                                               ║
     ╠════════════════════════════════════════════════════════════════════════════════════════════╣
     ║ Java runtime        : OpenJDK Runtime Environment                                          ║
     ║ Java version        : 16+36-2231                                                           ║
     ║ Java home           : /home/jkuhn/Devel/jdk/jdk-16                                         ║
     ║                                                                                            ║
     ║ Application module  : io.winterframework.example.sample_app                                ║
     ║ Application version : 1.0.0-SNAPSHOT                                                       ║
     ║ Application class   : io.winterframework.example.sample_app.App                            ║
     ║                                                                                            ║
     ║ Modules             :                                                                      ║
     ║  * ...                                                                                     ║
     ╚════════════════════════════════════════════════════════════════════════════════════════════╝


2021-04-08 23:50:35,266 INFO  [main] i.w.e.s.Sample_app - Starting Module io.winterframework.example.sample_app...
2021-04-08 23:50:35,266 INFO  [main] i.w.m.b.Boot - Starting Module io.winterframework.mod.boot...
2021-04-08 23:50:35,446 INFO  [main] i.w.m.b.Boot - Module io.winterframework.mod.boot started in 179ms
2021-04-08 23:50:35,446 INFO  [main] i.w.m.w.Web - Starting Module io.winterframework.mod.web...
2021-04-08 23:50:35,446 INFO  [main] i.w.m.h.s.Server - Starting Module io.winterframework.mod.http.server...
2021-04-08 23:50:35,446 INFO  [main] i.w.m.h.b.Base - Starting Module io.winterframework.mod.http.base...
2021-04-08 23:50:35,450 INFO  [main] i.w.m.h.b.Base - Module io.winterframework.mod.http.base started in 3ms
2021-04-08 23:50:35,545 INFO  [main] i.w.m.h.s.i.HttpServer - HTTP Server (nio) listening on http://0.0.0.0:8080
2021-04-08 23:50:35,546 INFO  [main] i.w.m.h.s.Server - Module io.winterframework.mod.http.server started in 99ms
2021-04-08 23:50:35,546 INFO  [main] i.w.m.w.Web - Module io.winterframework.mod.web started in 99ms
2021-04-08 23:50:35,546 INFO  [main] i.w.e.s.Sample_app - Module io.winterframework.example.sample_app started in 281ms

```

We can now test the application:

```
$ curl http://127.0.0.1:8080/message
Hello, world!
```

The application can be gracefully shutdown by pressing `Ctrl-c`.

### Building the application image

In order to create a native image containing the application and all its dependencies including JDK's dependencies, we can simply invoke the `winter:build-app` goal:

```
$ mvn package winter:build-app

...
[INFO] Building application image: /home/jkuhn/Devel/git/frmk/io.winterframework.example.sample-app/target/maven-winter/application_linux_amd64/sample-app-1.0.0-SNAPSHOT...
 [═══════════════════════════════════════════════  67 % ═════════════>                                ] Creating archive sample-app-1.0.0-SNAPSHOT-application_linux_amd64.zip...
```

This will create a ZIP archive containing a native application distribution `target/sample-app-1.0.0-SNAPSHOT-application_linux_amd64.zip` which will be deployed to the local Maven repository and eventually to a remote Maven repository.

Then in order to install the application on a compatible platform, we just need to download the archive corresponding to the platform, extract it to some location and run the application. Luckily for us this can be done quite easily with Maven dependency plugin:

```
$ mvn dependency:unpack -Dartifact=io.winterframework.example:sample-app:1.0.0-SNAPSHOT:zip:application_linux_amd64 -DoutputDirectory=./
...
$ ./sample-app-1.0.0-SNAPSHOT/bin/sample-app
...
```

It is also possible to create platform specific package such as `.deb` or a `.msi` by defining particular formats in the Winter Maven plugin configuration:

```xml
<project>
    <build>
        <plugins>
            <plugin>
                <groupId>io.winterframework.tool</groupId>
                <artifactId>winter-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <id>build-app</id>
                        <phase>package</phase>
                        <goals>
                            <goal>build-app</goal>
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

```
$ mvn package
...
```

> Note that there is no cross-platform support and a given platform specific format must be built on the platform it runs on.

Such platform-specific package can then be downloaded and installed using the right package manager:

```
$ mvn dependency:copy -Dartifact=io.winterframework.example:sample-app:1.0.0-SNAPSHOT:deb:application_linux_amd64 -DoutputDirectory=./
...
$ sudo dpkg -i sample-app-1.0.0-SNAPSHOT-application_linux_amd64.deb
...
$ /opt/sample-app/bin/sample-app
...
```

The Winter Maven plugin allows to create various application images including Docker or OCI container images, please refer to the [Winter Maven plugin documentation](https://github.com/winterframework-io/winter-tools/tree/master/winter-maven-plugin) to learn more.

