# Glossary

- **Spring Framework**: a Java framework whose core is a container that creates your objects and wires them together.
- **Spring Boot**: Spring plus opinionated defaults: starters, auto-configuration, and an embedded web server.
- **Maven / `pom.xml`**: build tool and its project file: dependencies, Java version, plugins.
- **Maven wrapper (`mvnw`)**: script that downloads and runs a pinned Maven version, so every machine builds the same way.
- **Starter**: a dependency with no code of its own that pulls in a tested set of libraries (e.g. `spring-boot-starter-webmvc`).
- **Parent POM (`spring-boot-starter-parent`)**: supplies compatible versions for hundreds of libraries, so our dependencies need no `<version>`.
- **Dependency scope**: `compile` (default), `runtime` (needed only when running, e.g. JDBC driver), `test` (tests only).
- **Classpath**: the set of jars and classes available to the running JVM.
- **Auto-configuration**: Boot inspects the classpath and config at startup and creates sensible beans (e.g. a DataSource if a JDBC driver is present).
- **Bean**: an object created and managed by the Spring container.
- **Application context**: the Spring container; holds all beans and their wiring.
- **Annotation**: a label on code (`@Something`) that does nothing alone; frameworks read it and act on it.
- **DataSource**: the object that hands out database connections.
- **HikariCP**: Spring Boot's default connection pool; keeps open DB connections ready for reuse.
- **JDBC URL**: `jdbc:postgresql://host:port/db`; tells Java which driver and which database to use.
- **Fail fast**: misconfiguration crashes at startup instead of at the first real request.
- **Embedded server (Tomcat)**: the web server runs inside our app process, started as a bean, not as a separate install.
- **Actuator**: Boot module exposing operational endpoints such as `/actuator/health`.
