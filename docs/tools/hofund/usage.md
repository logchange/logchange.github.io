# Usage Guide

This guide covers detailed usage scenarios, configuration examples, and advanced integration patterns for Hofund.

## Table of Contents

- [Connection Types](#connection-types)
- [HTTP Connections](#http-connections)
- [Database Connections](#database-connections)
- [Custom Connections](#custom-connections)
- [Configuration Options](#configuration-options)
- [Advanced Features](#advanced-features)
- [Integration Patterns](#integration-patterns)
- [Monitoring and Visualization](#monitoring-and-visualization)

## Connection Types

Hofund supports multiple connection types, each designed for specific monitoring scenarios:

- **HTTP**: REST APIs, web services, microservices
- **DATABASE**: PostgreSQL, Oracle, H2, MySQL
- **QUEUE**: Message queues and brokers
- **FTP**: File transfer services

## HTTP Connections

### Simple HTTP Connection

The easiest way to monitor an HTTP endpoint:

```java
@Configuration
public class HttpConnectionsConfig {
    
    @Bean
    public SimpleHofundHttpConnection paymentApiConnection() {
        return new SimpleHofundHttpConnection(
            "payment-api", 
            "https://payment-service.example.com/actuator/health"
        );
    }
    
    @Bean
    public SimpleHofundHttpConnection userServiceConnection() {
        return new SimpleHofundHttpConnection(
            "user-service", 
            "http://user-service:8080/health"
        );
    }
}
```

### HTTP Connection with Version Requirements

Monitor services with specific version requirements:

```java
@Bean
public SimpleHofundHttpConnection criticalServiceConnection() {
    return new SimpleHofundHttpConnection(
        "critical-service", 
        "https://critical-service.example.com/actuator/health"
    ).withRequiredVersion("2.1.0");
}
```

### HTTP Connection with Custom Method

Use different HTTP methods for health checks:

```java
@Bean
public SimpleHofundHttpConnection postBasedHealthCheck() {
    return new SimpleHofundHttpConnection(
        "post-health-service", 
        "https://service.example.com/api/health", 
        RequestMethod.POST
    );
}
```

### Custom HTTP Connection with AbstractHofundBasicHttpConnection

For more control over HTTP connections:

```java
@Component
public class CustomPaymentHealthCheck extends AbstractHofundBasicHttpConnection {

    @Value("${hofund.connection.payment.target:payment-service}")
    private String target;

    @Value("${hofund.connection.payment.url:http://payment-service:8080}")
    private String baseUrl;

    @Value("${hofund.connection.payment.health-endpoint:/actuator/health}")
    private String healthEndpoint;

    @Override
    protected String getTarget() {
        return target;
    }

    @Override
    protected String getUrl() {
        return baseUrl + healthEndpoint;
    }

    @Override
    protected RequestMethod getRequestMethod() {
        return RequestMethod.GET; // Default, can be overridden
    }

    @Override
    protected CheckingStatus getCheckingStatus() {
        // Can be ACTIVE or INACTIVE based on conditions
        return CheckingStatus.ACTIVE;
    }

    @Override
    protected String getRequiredVersion() {
        return "1.5.0"; // Optional version requirement
    }
}
```

### Conditional HTTP Connection

Enable/disable connections based on environment:

```java
@Component
@ConditionalOnProperty(name = "hofund.connections.external.enabled", havingValue = "true")
public class ExternalServiceHealthCheck extends AbstractHofundBasicHttpConnection {

    @Override
    protected String getTarget() {
        return "external-api";
    }

    @Override
    protected String getUrl() {
        return "https://external-api.example.com/health";
    }

    @Override
    protected CheckingStatus getCheckingStatus() {
        // Disable in local development
        if (isLocalEnvironment()) {
            return CheckingStatus.INACTIVE;
        }
        return CheckingStatus.ACTIVE;
    }

    private boolean isLocalEnvironment() {
        return Arrays.asList(environment.getActiveProfiles())
                    .contains("local");
    }
}
```

## Database Connections

Hofund automatically detects and monitors supported database connections:

### Supported Databases

- **PostgreSQL**: Automatically detected from DataSource
- **Oracle**: Automatically detected from DataSource  
- **H2**: Automatically detected from DataSource
- **MySQL**: Automatically detected from DataSource

### Multiple Database Connections

```yaml
# application.yml
spring:
  datasource:
    primary:
      url: jdbc:postgresql://localhost:5432/primary_db
      username: user
      password: password
    secondary:
      url: jdbc:mysql://localhost:3306/secondary_db  
      username: user
      password: password
```

```java
@Configuration
public class DatabaseConfiguration {
    
    @Bean
    @Primary
    @ConfigurationProperties("spring.datasource.primary")
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create().build();
    }
    
    @Bean
    @ConfigurationProperties("spring.datasource.secondary") 
    public DataSource secondaryDataSource() {
        return DataSourceBuilder.create().build();
    }
}
```

## Custom Connections

### Manual Connection Creation

Create connections with custom logic:

```java
@Configuration
public class CustomConnectionsConfig {
    
    @Bean
    public HofundConnection redisConnection() {
        return new HofundConnection(
            "redis-cache",                           // target name
            "redis://localhost:6379",               // URL
            Type.QUEUE,                             // connection type
            new AtomicReference<>(() -> {           // connection function
                try {
                    // Your custom connection logic
                    Jedis jedis = new Jedis("localhost", 6379);
                    String response = jedis.ping();
                    jedis.close();
                    
                    return HofundConnectionResult.http(
                        Status.UP, 
                        HofundConnectionResult.NOT_APPLICABLE
                    );
                } catch (Exception e) {
                    return HofundConnectionResult.http(
                        Status.DOWN, 
                        HofundConnectionResult.UNKNOWN
                    );
                }
            }),
            "Redis cache connection"                // description
        );
    }
    
    @Bean
    public HofundConnection mqttConnection() {
        return new HofundConnection(
            "mqtt-broker",
            "mqtt://mqtt.example.com:1883",
            Type.QUEUE,
            new AtomicReference<>(() -> {
                try {
                    // MQTT connection test logic
                    MqttClient client = new MqttClient("tcp://mqtt.example.com:1883", 
                                                      MqttClient.generateClientId());
                    client.connect();
                    boolean isConnected = client.isConnected();
                    client.disconnect();
                    
                    return HofundConnectionResult.http(
                        isConnected ? Status.UP : Status.DOWN,
                        HofundConnectionResult.NOT_APPLICABLE
                    );
                } catch (Exception e) {
                    return HofundConnectionResult.http(Status.DOWN, HofundConnectionResult.UNKNOWN);
                }
            }),
            "MQTT message broker"
        );
    }
}
```

### Custom ConnectionFunction

Reusable connection functions:

```java
public class CustomConnectionFunctions {
    
    public static ConnectionFunction tcpConnectionFunction(String host, int port) {
        return () -> {
            try (Socket socket = new Socket()) {
                socket.connect(new InetSocketAddress(host, port), 5000);
                return HofundConnectionResult.http(Status.UP, HofundConnectionResult.NOT_APPLICABLE);
            } catch (IOException e) {
                return HofundConnectionResult.http(Status.DOWN, HofundConnectionResult.UNKNOWN);
            }
        };
    }
    
    public static ConnectionFunction httpConnectionWithTimeout(String url, int timeoutMs) {
        return () -> {
            try {
                HttpURLConnection connection = (HttpURLConnection) new URL(url).openConnection();
                connection.setConnectTimeout(timeoutMs);
                connection.setReadTimeout(timeoutMs);
                connection.setRequestMethod("GET");
                
                int responseCode = connection.getResponseCode();
                if (responseCode >= 200 && responseCode < 300) {
                    return HofundConnectionResult.http(Status.UP, connection);
                } else {
                    return HofundConnectionResult.http(Status.DOWN, HofundConnectionResult.UNKNOWN);
                }
            } catch (Exception e) {
                return HofundConnectionResult.http(Status.DOWN, HofundConnectionResult.UNKNOWN);
            }
        };
    }
}
```

## Configuration Options

### Application Information Configuration

```yaml
# application.yml
hofund:
  info:
    application:
      name: my-application           # Application name (lowercase recommended)
      version: 1.2.3                # Application version
      type: microservice            # Application type (default: app)
      icon: docker                  # Grafana icon (default: docker)
```

```properties
# application.properties
hofund.info.application.name=my-application
hofund.info.application.version=1.2.3
hofund.info.application.type=microservice
hofund.info.application.icon=docker
```

### Git Information Configuration

Override git information if needed:

```yaml
hofund:
  git-info:
    commit:
      id: abc123def456                        # Full commit ID
      id-abbrev: abc123f                      # Abbreviated commit ID
    dirty: false                             # Whether working directory had uncommitted changes
    branch: main                             # Git branch name
    build:
      host: build-server                     # Build machine hostname
      time: "2025-08-30T20:45:00+0100"      # Build timestamp
```

### Connection-Specific Properties

Configure connections through properties:

```yaml
# Custom connection configuration
hofund:
  connections:
    payment:
      enabled: true
      url: https://payment-api.example.com
      timeout: 5000
      required-version: "2.1.0"
    
    external-service:
      enabled: false  # Disabled in this environment
      url: https://external-service.com
```

## Advanced Features

### Environment-Based Connection Disabling

Disable specific connections using environment variables:

```bash
# Disable payment-api connection
export HOFUND_CONNECTION_PAYMENT_API_DISABLED=true

# Disable user-service connection  
export HOFUND_CONNECTION_USER_SERVICE_DISABLED=1
```

The environment variable format: `HOFUND_CONNECTION_<TARGET>_DISABLED`
- `<TARGET>` is the uppercase target name with special characters replaced by underscores
- Values: `true` (case-insensitive) or `1`

### Connection Status Table

Display connection status during application startup:

```java
@Configuration
public class ConnectionMonitoringConfig {

    private static final Logger log = LoggerFactory.getLogger(ConnectionMonitoringConfig.class);

    @Bean
    public CommandLineRunner printConnectionsTable(HofundConnectionsTable connectionsTable) {
        return args -> {
            log.info("Current connection status:\n{}", connectionsTable.print());
        };
    }
}
```

Example output:
```
+----------+--------------+----------+----------------------------------------------+---------+------------------+
| TYPE     | NAME         | STATUS   | URL                                          | VERSION | REQUIRED VERSION |
+----------+--------------+----------+----------------------------------------------+---------+------------------+
| DATABASE | primary      | UP       | jdbc:postgresql://localhost:5432/mydb       | N/A     | N/A              |
| HTTP     | payment-api  | UP       | https://payment-api.example.com/health       | 2.1.0   | 2.0.0            |
| HTTP     | user-service | DOWN     | http://user-service:8080/health              | UNKNOWN | N/A              |
| QUEUE    | redis-cache  | UP       | redis://localhost:6379                       | N/A     | N/A              |
+----------+--------------+----------+----------------------------------------------+---------+------------------+
```

### Version Validation

Hofund can validate that connected services meet minimum version requirements:

```java
@Bean
public SimpleHofundHttpConnection versionSensitiveConnection() {
    return new SimpleHofundHttpConnection(
        "critical-api",
        "https://critical-api.example.com/actuator/health"
    ).withRequiredVersion("1.5.0");
}
```

If the detected version is lower than required, warnings will be logged.

### Custom Icons

Use custom Grafana icons for visual distinction:

```java
@Bean 
public HofundConnection customIconConnection() {
    HofundConnection connection = new HofundConnection(
        "special-service",
        "https://special-service.com/health", 
        Type.HTTP,
        new AtomicReference<>(connectionFunction),
        "Special service with custom icon"
    );
    connection.setIcon("cloud");  // Use Grafana's cloud icon
    return connection;
}
```

Available icons: database, file-alt, share-alt, channel-add, cloud, docker, etc.
See [Grafana Icons](https://developers.grafana.com/ui/latest/index.html?path=/story/iconography-icon--icons-overview) for complete list.

## Integration Patterns

### Microservices Architecture

```java
@Configuration
public class MicroservicesMonitoring {
    
    @Bean
    public SimpleHofundHttpConnection orderService() {
        return new SimpleHofundHttpConnection("order-service", "http://order-service:8080/actuator/health");
    }
    
    @Bean
    public SimpleHofundHttpConnection inventoryService() {
        return new SimpleHofundHttpConnection("inventory-service", "http://inventory-service:8080/actuator/health");
    }
    
    @Bean  
    public SimpleHofundHttpConnection notificationService() {
        return new SimpleHofundHttpConnection("notification-service", "http://notification-service:8080/actuator/health");
    }
}
```

### Multi-Environment Configuration

```yaml
# application-dev.yml
hofund:
  info:
    application:
      icon: "wrench"  # Development icon
  connections:
    external:
      enabled: false  # Disable external connections in dev

---
# application-prod.yml  
hofund:
  info:
    application:
      icon: "docker"  # Production icon
  connections:
    external:
      enabled: true
```

### Service Discovery Integration

```java
@Configuration
@EnableDiscoveryClient
public class ServiceDiscoveryMonitoring {
    
    @Autowired
    private DiscoveryClient discoveryClient;
    
    @Bean
    public List<SimpleHofundHttpConnection> dynamicConnections() {
        List<SimpleHofundHttpConnection> connections = new ArrayList<>();
        
        // Discover services and create connections dynamically
        discoveryClient.getServices().forEach(serviceName -> {
            List<ServiceInstance> instances = discoveryClient.getInstances(serviceName);
            if (!instances.isEmpty()) {
                ServiceInstance instance = instances.get(0);
                connections.add(new SimpleHofundHttpConnection(
                    serviceName,
                    instance.getUri() + "/actuator/health"
                ));
            }
        });
        
        return connections;
    }
}
```

### Circuit Breaker Integration

```java
@Component
public class CircuitBreakerHealthCheck extends AbstractHofundBasicHttpConnection {
    
    @Autowired
    private CircuitBreaker circuitBreaker;
    
    @Override
    protected String getTarget() {
        return "circuit-breaker-service";
    }
    
    @Override
    protected String getUrl() {
        return "https://external-service.com/api/health";
    }
    
    @Override
    protected CheckingStatus getCheckingStatus() {
        // Don't check if circuit breaker is open
        return circuitBreaker.getState() == CircuitBreaker.State.OPEN 
            ? CheckingStatus.INACTIVE 
            : CheckingStatus.ACTIVE;
    }
}
```

## Monitoring and Visualization

### Prometheus Queries

Useful Prometheus queries for monitoring:

```promql
# Application uptime
hofund_info

# Connection status (1=UP, 0=DOWN, -1=INACTIVE)
hofund_connection

# Git information
hofund_git_info

# Services that are down
hofund_connection{} == 0

# Count of healthy connections per application
sum(hofund_connection{} == 1) by (source)

# Connection availability over time
avg_over_time(hofund_connection{}[1h])
```

### Grafana Dashboard Setup

1. Import the provided dashboard: [hofund-node-graph.json](https://github.com/logchange/hofund/raw/master/grafana-dashboards/hofund-node-graph.json)
2. Configure Prometheus data source
3. Set up alerts for connection failures

### Alerting Rules

Example Prometheus alerting rules:

```yaml
groups:
  - name: hofund-alerts
    rules:
      - alert: ServiceDown
        expr: hofund_connection{} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Service {{ $labels.target }} is down"
          description: "Connection from {{ $labels.source }} to {{ $labels.target }} has been down for more than 1 minute."

      - alert: ServiceVersionMismatch
        expr: hofund_connection{detected_version!="UNKNOWN",required_version!="UNKNOWN",detected_version!=required_version} == 1
        for: 0m
        labels:
          severity: warning
        annotations:
          summary: "Service version mismatch detected"
          description: "Service {{ $labels.target }} version {{ $labels.detected_version }} does not match required version {{ $labels.required_version }}"
```

This comprehensive usage guide should help you implement Hofund effectively in various scenarios, from simple HTTP monitoring to complex microservices architectures.