# Frequently Asked Questions (FAQ)

> GitHub repository: [https://github.com/logchange/hofund](https://github.com/logchange/hofund)

This document addresses common questions, troubleshooting tips, and best practices for using Hofund.

## Table of Contents

- [General Questions](#general-questions)
- [Installation & Setup](#installation--setup)
- [Configuration](#configuration)
- [Connection Monitoring](#connection-monitoring)
- [Metrics & Monitoring](#metrics--monitoring)
- [Troubleshooting](#troubleshooting)
- [Performance & Best Practices](#performance--best-practices)
- [Integration Issues](#integration-issues)

## General Questions

### What is Hofund and why should I use it?

Hofund is a monitoring tool for Spring Boot applications that provides:
- **Connection health monitoring**: Monitor HTTP APIs, databases, and custom services
- **Prometheus metrics**: Expose standardized metrics for monitoring systems
- **Visual monitoring**: Integration with Grafana for system architecture visualization
- **Git information tracking**: Track deployment information and versions
- **Service discovery**: Automatic detection of database connections

It's particularly useful for microservices architectures where you need to monitor the health and connectivity of multiple services.

### How is Hofund different from Spring Boot Actuator?

Hofund complements Spring Boot Actuator by:
- **External monitoring**: Monitors external services and APIs, not just internal application health
- **Connection-focused**: Specifically designed for monitoring connections between services
- **Visual representation**: Provides graph-based visualization of service dependencies
- **Version tracking**: Tracks and validates service versions
- **Prometheus-ready**: Specifically designed for Prometheus metric collection

### What's the meaning behind the name "Hofund"?

Hofund (pronounced "ho-fund") is the sword of Heimdall in Norse mythology. It serves as the key to activate the Bifrost Bridge, connecting the nine realms. Similarly, Hofund the tool connects and monitors your services, providing a bridge of visibility across your system architecture.

## Installation & Setup

### Do I need to include Spring Boot Actuator separately?

Yes, you need to include Spring Boot Actuator and Micrometer Prometheus registry:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

### What Java versions are supported?

- **Hofund 2.x.x**: Java 17+ (hofund-core supports Java 8+)
- **Hofund 1.x.x (deprecated)**: Java 8+

### Can I use Hofund with Spring Boot 2.x?

Hofund 2.x.x requires Spring Boot 3.3.0+. For Spring Boot 2.x (2.2.0 to 3.2.X), use Hofund 1.x.x (deprecated).

### The git-commit-id-maven-plugin is not generating git.properties

Ensure the plugin configuration includes:

```xml
<configuration>
    <generateGitPropertiesFile>true</generateGitPropertiesFile>
    <failOnNoGitDirectory>false</failOnNoGitDirectory>
    <injectAllReactorProjects>true</injectAllReactorProjects>
</configuration>
```

If you're not in a Git repository, set `failOnNoGitDirectory` to `false`.

## Configuration

### How do I configure application name and version using Maven properties?

Use Maven property placeholders in your configuration:

```properties
hofund.info.application.name=@project.name@
hofund.info.application.version=@project.version@
```

Ensure resource filtering is enabled in your `pom.xml`:

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

### Can I override git information manually?

Yes, you can override any git property:

```yaml
hofund:
  git-info:
    commit:
      id: "custom-commit-id"
      id-abbrev: "abc123f"
    branch: "main"
    dirty: "false"
    build:
      host: "build-server"
      time: "2025-08-30T20:45:00+0100"
```

### How do I disable all connection checking temporarily?

You can disable individual connections using environment variables:

```bash
export HOFUND_CONNECTION_SERVICE_NAME_DISABLED=true
```

Or conditionally disable connections in code:

```java
@Override
protected CheckingStatus getCheckingStatus() {
    return someCondition ? CheckingStatus.INACTIVE : CheckingStatus.ACTIVE;
}
```

### Why is my application name appearing in uppercase in logs but lowercase in metrics?

This is expected behavior. Hofund automatically converts application names to lowercase for metrics to ensure consistency in Prometheus queries and Grafana visualizations.

## Connection Monitoring

### My HTTP connection always shows as DOWN even though the service is running

Check these common issues:

1. **URL accessibility**: Ensure the URL is accessible from your application's network context
2. **HTTP method**: Some services only respond to specific HTTP methods (GET, POST)
3. **Authentication**: Services requiring authentication will fail health checks
4. **Firewall/Network**: Network restrictions might block the connection
5. **SSL certificates**: HTTPS endpoints with invalid certificates will fail

```properties
# Debug by enabling logging
logging.level.dev.logchange.hofund=DEBUG
```

### How do I monitor services that require authentication?

For simple authentication, extend `AbstractHofundBasicHttpConnection`:

```java
@Component
public class AuthenticatedHealthCheck extends AbstractHofundBasicHttpConnection {
    
    @Override
    protected String getTarget() {
        return "authenticated-service";
    }
    
    @Override
    protected String getUrl() {
        return "https://service.com/health?token=" + getAuthToken();
    }
    
    private String getAuthToken() {
        // Your token retrieval logic
        return tokenService.getToken();
    }
}
```

For complex authentication, create a custom `HofundConnection` with a custom `ConnectionFunction`.

### Can I monitor non-HTTP services like databases, Redis, or message queues?

Yes! Hofund automatically detects supported databases (PostgreSQL, Oracle, H2). For other services, create custom connections:

```java
@Bean
public HofundConnection redisConnection() {
    return new HofundConnection(
        "redis",
        "redis://localhost:6379",
        Type.QUEUE,
        new AtomicReference<>(() -> {
            try (Jedis jedis = new Jedis("localhost", 6379)) {
                jedis.ping();
                return HofundConnectionResult.http(Status.UP, HofundConnectionResult.NOT_APPLICABLE);
            } catch (Exception e) {
                return HofundConnectionResult.http(Status.DOWN, HofundConnectionResult.UNKNOWN);
            }
        }),
        "Redis cache"
    );
}
```

### What does "UNKNOWN" version mean?

"UNKNOWN" appears when:
- The target service doesn't expose version information
- The version information is in an unexpected format
- There's an error retrieving version information

Expected version format from target service:
```json
{
  "application": {
    "name": "service-name",
    "version": "1.2.3"
  }
}
```

### How do I set up version requirements and validation?

```java
@Bean
public SimpleHofundHttpConnection versionValidatedService() {
    return new SimpleHofundHttpConnection(
        "critical-service",
        "https://service.com/actuator/health"
    ).withRequiredVersion("2.1.0");
}
```

Hofund will log warnings if the detected version is lower than the required version.

## Metrics & Monitoring

### I don't see hofund metrics in the Prometheus endpoint

1. **Check endpoint exposure**:
   ```properties
   management.endpoints.web.exposure.include=prometheus
   ```

2. **Verify actuator is working**: Visit `/actuator/prometheus` directly

3. **Check for exceptions**: Look for startup errors in logs

4. **Confirm dependencies**: Ensure micrometer-registry-prometheus is included

### How do I interpret the metric values?

**hofund_connection values:**
- `1`: Connection is UP (healthy)
- `0`: Connection is DOWN (failed)
- `-1`: Connection is INACTIVE (not being checked)

**hofund_info values:**
- Always `1.0` when the application is running

**hofund_git_info values:**
- Always `1.0`, actual information is in labels

### Can I create custom metrics alongside Hofund?

Yes! Hofund uses standard Micrometer APIs. You can add your own metrics:

```java
@Component
public class CustomMetrics {
    
    private final Counter customCounter;
    
    public CustomMetrics(MeterRegistry meterRegistry) {
        this.customCounter = Counter.builder("custom_metric")
            .description("My custom metric")
            .register(meterRegistry);
    }
}
```

### Why are my database connections not showing up automatically?

Automatic database detection works for:
- Standard Spring Boot DataSource beans
- Supported databases: PostgreSQL, Oracle, H2

If your setup doesn't match these criteria, create manual connections:

```java
@Bean
public HofundConnection customDbConnection() {
    return new HofundConnection(
        "custom-db",
        "jdbc:custom://localhost:5432/mydb",
        Type.DATABASE,
        new AtomicReference<>(() -> {
            try {
                // Test database connection
                dataSource.getConnection().close();
                return HofundConnectionResult.db(Status.UP);
            } catch (Exception e) {
                return HofundConnectionResult.db(Status.DOWN);
            }
        }),
        "Custom database"
    );
}
```

## Troubleshooting

### Application fails to start with "URL cannot end with '/prometheus'"

This error prevents recursive dependencies. Check your connection URLs:

**❌ This will fail:**
```java
new SimpleHofundHttpConnection("bad", "http://service/prometheus");
```

**✅ This is correct:**
```java
new SimpleHofundHttpConnection("good", "http://service/actuator/health");
```

### Connection table is not printing during startup

Ensure you've configured the connection table bean:

```java
@Bean
public CommandLineRunner printConnectionsTable(HofundConnectionsTable table) {
    return args -> log.info("\n{}", table.print());
}
```

### High memory usage with many connections

For high-frequency monitoring:

1. **Increase check intervals**: Implement custom timing logic
2. **Use connection pooling**: For database connections
3. **Implement circuit breakers**: Prevent cascade failures
4. **Monitor inactive connections**: Set connections to INACTIVE when not needed

### SSL certificate issues with HTTPS connections

For development/testing environments:

```java
// Not recommended for production!
@Component
public class InsecureHttpConnection extends AbstractHofundBasicHttpConnection {
    @Override
    protected HttpURLConnection createConnection() throws IOException {
        HttpURLConnection conn = super.createConnection();
        if (conn instanceof HttpsURLConnection) {
            ((HttpsURLConnection) conn).setHostnameVerifier((hostname, session) -> true);
        }
        return conn;
    }
}
```

Better approach: Fix SSL certificates or use HTTP in development.

## Performance & Best Practices

### How often does Hofund check connections?

Connection checks are performed:
- When Prometheus scrapes metrics (on-demand)
- The frequency depends on your Prometheus scrape interval
- No background checking happens by default

### Best practices for connection URLs

1. **Use dedicated health endpoints**: `/actuator/health`, `/health`, `/status`
2. **Avoid business logic endpoints**: Don't use endpoints that trigger business operations
3. **Use internal network addresses**: Avoid external DNS when possible
4. **Keep URLs simple**: Avoid complex query parameters

### Should I monitor every service?

Monitor services that:
- ✅ Are critical to your application's functionality
- ✅ Have clear health endpoints
- ✅ Are external dependencies
- ✅ Are frequently accessed

Don't monitor:
- ❌ Services that are rarely used
- ❌ Internal implementation details
- ❌ Services without proper health endpoints

### How many connections can Hofund handle?

Hofund is lightweight and can handle hundreds of connections. Consider:
- Each connection check adds latency to Prometheus scraping
- Network timeouts can slow down metric collection
- Use connection pooling for database connections

### Environment-specific configurations

**application-dev.yml:**
```yaml
hofund:
  info:
    application:
      icon: "wrench"
  connections:
    external:
      enabled: false
```

**application-prod.yml:**
```yaml
hofund:
  info:
    application:
      icon: "docker"
  connections:
    external:
      enabled: true
```

## Integration Issues

### Grafana dashboard shows no data

1. **Check Prometheus data source**: Ensure Grafana can connect to Prometheus
2. **Verify metric names**: Look for `hofund_*` metrics in Prometheus
3. **Check time ranges**: Ensure the dashboard time range includes data
4. **Confirm scraping**: Verify Prometheus is scraping your application

### Service discovery integration problems

When using service discovery:

```java
@Bean
@ConditionalOnProperty(name = "hofund.service-discovery.enabled", havingValue = "true")
public List<SimpleHofundHttpConnection> discoveredConnections(DiscoveryClient discoveryClient) {
    return discoveryClient.getServices().stream()
        .filter(serviceName -> !serviceName.equals(applicationName)) // Don't monitor self
        .map(serviceName -> {
            List<ServiceInstance> instances = discoveryClient.getInstances(serviceName);
            if (!instances.isEmpty()) {
                ServiceInstance instance = instances.get(0);
                return new SimpleHofundHttpConnection(
                    serviceName,
                    instance.getUri() + "/actuator/health"
                );
            }
            return null;
        })
        .filter(Objects::nonNull)
        .collect(Collectors.toList());
}
```

### Docker/Kubernetes networking issues

Common networking problems in containerized environments:

1. **Service names**: Use Kubernetes service names instead of localhost
2. **Port mapping**: Ensure ports are correctly exposed
3. **Network policies**: Check if network policies block connections
4. **DNS resolution**: Verify service discovery is working

```yaml
# Kubernetes service example
hofund:
  connections:
    user-service:
      url: "http://user-service.default.svc.cluster.local:8080/actuator/health"
```

### Version detection not working

Target services must expose version information in this format:

```json
{
  "application": {
    "name": "service-name",
    "version": "1.2.3"
  }
}
```

If your service uses a different format, version detection will show "UNKNOWN".

---

For more specific issues not covered here, check the [GitHub issues](https://github.com/logchange/hofund/issues) or create a new issue with detailed information about your setup and the problem you're experiencing.