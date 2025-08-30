# Reference Documentation

This document provides complete API reference, configuration options, and technical specifications for Hofund.

## Table of Contents

- [Core Classes and Interfaces](#core-classes-and-interfaces)
- [Configuration Properties](#configuration-properties)
- [Connection Types](#connection-types)
- [Metrics Reference](#metrics-reference)
- [Environment Variables](#environment-variables)
- [Maven Configuration](#maven-configuration)
- [Advanced Configuration](#advanced-configuration)
- [API Methods](#api-methods)

## Core Classes and Interfaces

### HofundConnection

The main class representing a connection to monitor.

```java
public class HofundConnection {
    public HofundConnection(String target, String url, Type type, 
                           AtomicReference<ConnectionFunction> fun, String description)
    
    // Getters
    public String getTarget()
    public String getUrl()
    public Type getType()
    public AtomicReference<ConnectionFunction> getFun()
    public String getDescription()
    public String getIcon()
    
    // Setters
    public void setIcon(String icon)
    public void setRequiredVersion(String requiredVersion)
    
    // Utility methods
    public String toTargetTag()
    public String getEdgeId(HofundInfoProvider infoProvider)
    public List<Tag> getTags(HofundInfoProvider infoProvider)
    public static String getEnvVarName(String target)
}
```

**Constructor Parameters:**
- `target`: Name of the resource being monitored
- `url`: Connection URL (cannot end with "/prometheus")
- `type`: Connection type (HTTP, DATABASE, QUEUE, FTP)
- `fun`: Connection function for health checking
- `description`: Optional description

### ConnectionFunction

Functional interface for implementing custom connection logic.

```java
@FunctionalInterface
public interface ConnectionFunction {
    HofundConnectionResult getConnection();
}
```

### HofundConnectionResult

Represents the result of a connection check.

```java
public class HofundConnectionResult {
    // Constants
    public static final String UNKNOWN = "UNKNOWN";
    public static final String NOT_APPLICABLE = "N/A";
    
    // Factory methods
    public static HofundConnectionResult db(Status status)
    public static HofundConnectionResult http(Status status, String version)
    public static HofundConnectionResult http(Status status, HttpURLConnection connection)
    
    // Getters
    public Status getStatus()
    public String getVersion()
}
```

**Status Values:**
- `UP`: Connection is healthy
- `DOWN`: Connection failed
- `INACTIVE`: Connection monitoring is disabled

### AbstractHofundBasicHttpConnection

Abstract base class for HTTP connection monitoring.

```java
public abstract class AbstractHofundBasicHttpConnection {
    // Abstract methods (must implement)
    protected abstract String getTarget()
    protected abstract String getUrl()
    
    // Optional overrides
    protected RequestMethod getRequestMethod() // Default: GET
    protected CheckingStatus getCheckingStatus() // Default: ACTIVE
    protected String getRequiredVersion() // Default: null
    protected String getIcon() // Default: based on type
    
    // Final method
    public final HofundConnection toHofundConnection()
}
```

### SimpleHofundHttpConnection

Concrete implementation for simple HTTP monitoring.

```java
public class SimpleHofundHttpConnection extends AbstractHofundBasicHttpConnection {
    // Constructors
    public SimpleHofundHttpConnection(String target, String url)
    public SimpleHofundHttpConnection(String target, String url, RequestMethod method)
    public SimpleHofundHttpConnection(String target, String url, CheckingStatus status)
    public SimpleHofundHttpConnection(String target, String url, RequestMethod method, CheckingStatus status)
    
    // Fluent methods
    public SimpleHofundHttpConnection withRequiredVersion(String version)
}
```

### HofundConnectionsProvider

Interface for providing collections of connections.

```java
public interface HofundConnectionsProvider {
    List<HofundConnection> getConnections();
}
```

### HofundConnectionsTable

Utility class for displaying connection status in table format.

```java
public class HofundConnectionsTable {
    public HofundConnectionsTable(List<HofundConnectionsProvider> connectionsProviders)
    public String print()
}
```

## Configuration Properties

### Application Information (`hofund.info`)

```yaml
hofund:
  info:
    application:
      name: string          # Application name (required)
      version: string       # Application version (required)
      type: string         # Application type (default: "app")
      icon: string         # Grafana icon name (default: "docker")
```

**Application Type Examples:**
- `microservice`
- `web-app`
- `api`
- `backend`
- `frontend`
- `database`

**Icon Options:**
Use any [Grafana built-in icon](https://developers.grafana.com/ui/latest/index.html?path=/story/iconography-icon--icons-overview). Common options:
- `docker` (default)
- `database`
- `cloud`
- `share-alt`
- `wrench`
- `channel-add`
- `file-alt`
- `empty` (to disable icon)

### Git Information (`hofund.git-info`)

```yaml
hofund:
  git-info:
    commit:
      id: string           # Full commit hash
      id-abbrev: string    # Abbreviated commit hash
    dirty: string          # "true"/"false" - uncommitted changes
    branch: string         # Git branch name
    build:
      host: string         # Build machine hostname
      time: string         # Build timestamp (ISO format)
```

**Default Values:**
All properties default to values from `git.properties` file generated by `git-commit-id-maven-plugin`.

## Connection Types

### Type Enumeration

```java
public enum Type {
    HTTP,        // REST APIs, web services
    DATABASE,    // Relational databases
    QUEUE,       // Message queues, brokers
    FTP          // File transfer services
}
```

### Default Icons by Type

- `DATABASE` → `"database"`
- `FTP` → `"file-alt"`
- `HTTP` → `"share-alt"`
- `QUEUE` → `"channel-add"`

### RequestMethod Enumeration

```java
public enum RequestMethod {
    GET,     // Default for most health checks
    POST,    // For services requiring POST
    PUT,     // Custom implementations
    DELETE,  // Custom implementations
    HEAD,    // Lightweight checks
    OPTIONS, // CORS preflight
    PATCH,   // Custom implementations
    TRACE    // Debug purposes
}
```

### CheckingStatus Enumeration

```java
public enum CheckingStatus {
    ACTIVE,    // Connection will be monitored
    INACTIVE   // Connection monitoring disabled
}
```

## Metrics Reference

### hofund_info

**Description:** Basic information about the application  
**Type:** Gauge  
**Value:** Always `1.0`

**Labels:**
- `application_name`: Application name (lowercase)
- `application_version`: Application version
- `id`: Same as application_name

**Example:**
```
hofund_info{application_name="user-service",application_version="1.2.3",id="user-service"} 1.0
```

### hofund_connection

**Description:** Connection status monitoring  
**Type:** Gauge  
**Values:**
- `1.0`: Connection UP
- `0.0`: Connection DOWN
- `-1.0`: Connection INACTIVE

**Labels:**
- `id`: Edge ID (source-target_type or source-target)
- `source`: Source application name (lowercase)
- `target`: Target service name (with type suffix for DB/QUEUE)
- `type`: Connection type (lowercase)
- `detected_version`: Version detected from target service
- `required_version`: Required version (if specified)

**Example:**
```
hofund_connection{id="user-service-payment-api",source="user-service",target="payment-api",type="http",detected_version="2.1.0",required_version="2.0.0"} 1.0
```

### hofund_git_info

**Description:** Git and build information  
**Type:** Gauge  
**Value:** Always `1.0`

**Labels:**
- `branch`: Git branch name
- `build_host`: Build machine hostname
- `build_time`: Build timestamp
- `commit_id`: Abbreviated commit hash
- `dirty`: Whether working directory had uncommitted changes

**Example:**
```
hofund_git_info{branch="main",build_host="ci-server",build_time="2025-08-30T20:45:00+0100",commit_id="abc123f",dirty="false"} 1.0
```

## Environment Variables

### Connection Disabling

Format: `HOFUND_CONNECTION_<TARGET>_DISABLED`

Where `<TARGET>` is:
- Original target name in uppercase
- Special characters replaced with underscores
- Only alphanumeric characters and underscores allowed

**Examples:**
```bash
# Disable "payment-api" connection
HOFUND_CONNECTION_PAYMENT_API_DISABLED=true

# Disable "user.service" connection  
HOFUND_CONNECTION_USER_SERVICE_DISABLED=1

# Disable "order-service-v2" connection
HOFUND_CONNECTION_ORDER_SERVICE_V2_DISABLED=true
```

**Accepted Values:**
- `true` (case-insensitive)
- `1`

## Maven Configuration

### Required Dependencies

```xml
<dependencies>
    <!-- Hofund starter -->
    <dependency>
        <groupId>dev.logchange.hofund</groupId>
        <artifactId>hofund-spring-boot-starter</artifactId>
        <version>2.10.1</version>
    </dependency>
    
    <!-- Spring Boot Actuator -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    
    <!-- Prometheus metrics -->
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-registry-prometheus</artifactId>
    </dependency>
</dependencies>
```

### Git Commit Plugin Configuration

```xml
<plugin>
    <groupId>io.github.git-commit-id</groupId>
    <artifactId>git-commit-id-maven-plugin</artifactId>
    <version>9.0.2</version>
    <executions>
        <execution>
            <id>get-the-git-infos</id>
            <goals>
                <goal>revision</goal>
            </goals>
            <phase>initialize</phase>
        </execution>
    </executions>
    <configuration>
        <generateGitPropertiesFile>true</generateGitPropertiesFile>
        <failOnNoGitDirectory>false</failOnNoGitDirectory>
        <injectAllReactorProjects>true</injectAllReactorProjects>
        <!-- Optional: customize git properties file location -->
        <gitPropertiesPluginDir>${project.basedir}/src/main/resources</gitPropertiesPluginDir>
    </configuration>
</plugin>
```

### Resource Filtering (for Maven placeholders)

```xml
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>
```

## Advanced Configuration

### Auto-Configuration Classes

Hofund provides these Spring Boot auto-configuration classes:

- `HofundInfoAutoConfiguration`: Configures application information
- `HofundGitInfoAutoConfiguration`: Configures git information  
- `HofundDefaultGitInfoProperties`: Loads default git properties
- `HofundConnectionMeterAutoConfiguration`: Configures connection metrics
- `HofundNodeMeterAutoConfiguration`: Configures node metrics
- `HofundEdgeMeterAutoConfiguration`: Configures edge metrics

### Conditional Configuration

```java
// Enable only in specific environments
@ConditionalOnProperty(name = "hofund.enabled", havingValue = "true", matchIfMissing = true)

// Enable only when actuator is present
@ConditionalOnClass(MeterRegistry.class)

// Enable only with specific profiles
@Profile("!test")
```

### Custom MeterBinder

```java
@Component
public class CustomHofundMeterBinder implements MeterBinder {
    
    @Override
    public void bindTo(MeterRegistry registry) {
        // Custom metrics logic
        Gauge.builder("custom_hofund_metric")
            .description("Custom metric description")
            .register(registry, this, obj -> calculateValue());
    }
    
    private double calculateValue() {
        // Your metric calculation logic
        return 42.0;
    }
}
```

### Connection Provider Implementation

```java
@Component
public class CustomConnectionsProvider implements HofundConnectionsProvider {
    
    @Override
    public List<HofundConnection> getConnections() {
        List<HofundConnection> connections = new ArrayList<>();
        
        // Add your custom connections
        connections.add(createDatabaseConnection());
        connections.add(createQueueConnection());
        
        return connections;
    }
    
    private HofundConnection createDatabaseConnection() {
        return new HofundConnection(
            "custom-db",
            "jdbc:postgresql://db:5432/mydb",
            Type.DATABASE,
            new AtomicReference<>(() -> {
                // Custom database health check logic
                return HofundConnectionResult.db(Status.UP);
            }),
            "Custom database connection"
        );
    }
}
```

## API Methods

### Version Utility Methods

```java
// Version comparison and validation
public class Version {
    public static Version of(String version)
    public boolean isUnknown()
    public boolean isNotApplicable()
    public int compareTo(Version other)
    public String toString()
}
```

### String Utilities

```java
public class StringUtils {
    public static boolean isEmpty(String str)
    public static boolean isNotEmpty(String str)
    public static String emptyIfNull(String str)
}
```

### Connection Result Factory Methods

```java
// For database connections (version always N/A)
HofundConnectionResult.db(Status.UP)
HofundConnectionResult.db(Status.DOWN)

// For HTTP connections with explicit version
HofundConnectionResult.http(Status.UP, "1.2.3")
HofundConnectionResult.http(Status.DOWN, HofundConnectionResult.UNKNOWN)

// For HTTP connections with version extraction from response
HofundConnectionResult.http(Status.UP, httpUrlConnection)
```

### Environment Variable Name Generation

```java
// Generate environment variable name for connection disabling
String envVarName = HofundConnection.getEnvVarName("payment-api");
// Returns: "HOFUND_CONNECTION_PAYMENT_API_DISABLED"

String envVarName2 = HofundConnection.getEnvVarName("user.service-v2");
// Returns: "HOFUND_CONNECTION_USER_SERVICE_V2_DISABLED"
```

## Compatibility Matrix

| Hofund Version | Spring Boot | Java Version | Status |
|----------------|-------------|-------------|---------|
| 2.10.1         | 3.3.0+      | 17+         | Current |
| 2.9.0          | 3.3.0+      | 17+         | Supported |
| 2.8.0          | 3.3.0+      | 17+         | Supported |
| 1.0.X          | 2.2.0-3.2.X | 8+          | Deprecated |

## Integration Points

### With Spring Boot Actuator
- Requires `/actuator/prometheus` endpoint exposure
- Uses Micrometer MeterRegistry for metrics
- Integrates with health indicators

### With Prometheus
- Metrics follow Prometheus naming conventions
- Labels are sanitized for Prometheus compatibility
- Gauge metrics with help text and type information

### With Grafana
- Provides ready-to-use dashboard JSON
- Supports node graph visualization
- Color coding for connection status

---

This reference documentation covers all public APIs, configuration options, and integration points available in Hofund. For implementation examples and best practices, refer to the [usage guide](usage.md).