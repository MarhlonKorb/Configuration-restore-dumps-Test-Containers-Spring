import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.BeforeAll;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.annotation.DirtiesContext;
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.containers.wait.strategy.Wait;
import org.testcontainers.utility.MountableFile;

import java.time.Duration;

/**
 * Abstract class for configuring the integration test environment.
* Any class that extends it will have access to a PostgreSQL database container for test execution.
* The container is destroyed after each test class finishes.
 */
@DirtiesContext(classMode = DirtiesContext.ClassMode.AFTER_CLASS)
@SpringBootTest(classes = YourApplicationName.class)
public abstract class AbstractPostgresIntegrationTests {

    protected static PostgreSQLContainer<?> firstDb;
    protected static PostgreSQLContainer<?> secondDb;

    @BeforeAll
    static void startContainersAndRestore() {

        firstDb = new PostgreSQLContainer<>("postgres:17-alpine")
                .withDatabaseName("database-name")
                .withUsername("postgres")
                .withPassword("postgres")
                .waitingFor(Wait.forListeningPort())
                .withStartupTimeout(Duration.ofMinutes(2));

        secondDb = new PostgreSQLContainer<>("postgres:17-alpine")
                .withDatabaseName("database-name-2")
                .withUsername("postgres")
                .withPassword("postgres")
                .waitingFor(Wait.forListeningPort())
                .withStartupTimeout(Duration.ofMinutes(2));

        firstDb.start();
        secondDb.start();

        restoreBackup(firstDb, "db/migration/dump/first-db.backup");
        restoreBackup(secondDb, "db/migration/dump/second-db.backup");
    }

    @AfterAll
    static void stopContainers() {
        if (firstDb != null) firstDb.stop();
        if (secondDb != null) secondDb.stop();
    }

    private static void restoreBackup(PostgreSQLContainer<?> container, String classpathBackup) {

        try {
            container.copyFileToContainer(
                    MountableFile.forClasspathResource(classpathBackup),
                    "/restore.backup"
            );

            var result = container.execInContainer(
                    "pg_restore",
                    "-U", container.getUsername(),
                    "-d", container.getDatabaseName(),
                    "--clean",
                    "--if-exists",
                    "--no-owner",
                    "--no-privileges",
                    "/restore.backup"
            );

            if (result.getExitCode() != 0) {
                throw new IllegalStateException("""
                        ERROR ON RESTORE
                        STDOUT:
                        %s
                        STDERR:
                        %s
                        """.formatted(result.getStdout(), result.getStderr()));
            }

        } catch (Exception e) {
            throw new RuntimeException("Error on restore backup: " + classpathBackup, e);
        }
    }

    @DynamicPropertySource
    static void overrideProperties(DynamicPropertyRegistry registry) {

        // // First db config (principal)
        registry.add("spring.datasource.url", firstDb::getJdbcUrl);
        registry.add("spring.datasource.username", firstDb::getUsername);
        registry.add("spring.datasource.password", firstDb::getPassword);

        // Second db config
        registry.add("spring.datasource-er.url", secondDb::getJdbcUrl);
        registry.add("spring.datasource-er.username", secondDb::getUsername);
        registry.add("spring.datasource-er.password", secondDb::getPassword);

        // Hardening
        // registry.add("spring.flyway.enabled", () -> "false");
        // registry.add("spring.jpa.hibernate.ddl-auto", () -> "none");
    }
}
