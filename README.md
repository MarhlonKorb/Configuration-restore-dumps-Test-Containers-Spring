# Using `AbstractPostgresIntegrationTests`

This document explains how to use the `AbstractPostgresIntegrationTests` base class to run **Spring Boot integration tests with PostgreSQL containers** using Testcontainers.

---

## Purpose

`AbstractPostgresIntegrationTests` is an abstract base class designed to standardize and simplify database integration testing.

It provides:

* Fully isolated PostgreSQL environments per test class
* Deterministic database state via restored backups
* Automatic Spring datasource configuration
* Proper container lifecycle management

Each test class that extends this base class runs with **new PostgreSQL containers**, which are destroyed after the test class finishes.

---

## How It Works

For every test class that extends `AbstractPostgresIntegrationTests`, the following lifecycle is applied:

1. Two PostgreSQL containers are started before any tests run
2. Containers wait until they are ready to accept TCP connections
3. Database backups are copied into the containers
4. Backups are restored using `pg_restore`
5. Spring Boot datasource properties are injected dynamically
6. Flyway migrations and schema auto-generation are disabled
7. The Spring context is marked as dirty after the test class
8. Containers are stopped and removed

---

## Requirements

Make sure the following requirements are met:

* Java 17 or higher
* Docker running locally or in CI
* Spring Boot Test
* Testcontainers

---

## Database Backups

The class expects PostgreSQL backup files (`.backup`) to be available on the test classpath.

Recommended directory structure:

```
src/test/resources/
└── db/
    └── migration/
        └── dump/
            ├── first-db.backup
            └── second-db.backup
```

These backups are automatically restored when the containers start.

---

## How to Use

### 1. Extend the Base Class

Create your integration test class and extend `AbstractPostgresIntegrationTests`:

```java
class ExampleIntegrationTest
        extends AbstractPostgresIntegrationTests {

    @Test
    void shouldRunWithRestoredDatabase() {
        // Database is ready and restored
    }
}
```

No additional configuration is required.

---

## Spring Configuration

The base class dynamically overrides Spring properties at runtime using `@DynamicPropertySource`, including:

* Primary datasource JDBC URL, username, and password
* Secondary datasource configuration (if present)
* `spring.flyway.enabled = false`
* `spring.jpa.hibernate.ddl-auto = none`

This guarantees Spring always connects to the correct containerized databases.

---

## Test Isolation

Each test class:

* Uses brand-new PostgreSQL containers
* Starts with a freshly restored database
* Runs in a new Spring `ApplicationContext`

This prevents:

* Connection reuse errors
* Data leakage between tests
* Hidden test ordering dependencies

---

## Performance Considerations

Starting containers per test class increases execution time but significantly improves reliability.

Recommended usage:

* Integration tests with complex database state
* Critical business logic validation
* CI pipelines requiring deterministic behavior

Unit tests should remain separate and should not extend this base class.

---

## Common Mistakes to Avoid

* Reusing containers across test classes
* Hardcoding database ports
* Relying on Spring context caching
* Modifying database schema during tests

All of these are intentionally avoided by this base class.

---

## Summary

By extending `AbstractPostgresIntegrationTests`, your integration tests gain:

* Isolated PostgreSQL environments
* Repeatable and deterministic database state
* Automatic Spring configuration
* Clean container lifecycle management

This approach prioritizes **test correctness and stability** over execution speed, making it ideal for robust integration testing.

## Dependencies

```
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>postgresql</artifactId>
            <version>1.20.1</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>1.19.8</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.19.0</version>
            <scope>test</scope>
        </dependency>
```
