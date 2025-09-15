# Getting Started with Hofund

> GitHub repository: [https://github.com/logchange/hofund](https://github.com/logchange/hofund)

Hofund is a monitoring tool designed for Spring Boot applications that provides connection health checking, Prometheus metrics exposure, and seamless integration with Grafana dashboards.

## Quick Overview

Hofund helps you:
- Monitor application health and connections
- Expose metrics to Prometheus
- Visualize your system architecture in Grafana
- Track git-based deployment information

## Prerequisites

Before you start, ensure your project meets these requirements:

- **Java**: Version 17+ (hofund-core supports Java 8+)
- **Spring Boot**: Version 3.3.0+ for Hofund 2.x.x
- **Maven**: For dependency management

Required dependencies (usually included with Spring Boot 2.2.0+):
- spring-framework 5.2.12.RELEASE or later
- micrometer-io 1.3.0 or later  
- slf4j 1.7.28 or later

## Installation

### 1. Add Hofund Dependency

Add the following to your `pom.xml`:

```xml
<dependency>
    <groupId>dev.logchange.hofund</groupId>
    <artifactId>hofund-spring-boot-starter</artifactId>
    <version>2.10.1</version>
</dependency>
```

### 2. Add Git Commit Plugin

This plugin generates git information for your builds:

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
    </configuration>
</plugin>
```

### 3. Add Spring Boot Actuator

If not already present, add these dependencies:

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

## Basic Configuration

### 1. Enable Prometheus Endpoint

Add to your `application.properties`:

```properties
management.endpoints.web.exposure.include=prometheus
```

Or in `application.yml`:

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "prometheus"
```

### 2. Configure Application Information

Add basic application information to `application.properties`:

```properties
hofund.info.application.name=@project.name@
hofund.info.application.version=@project.version@
```

Or in `application.yml`:

```yaml
hofund:
  info:
    application:
      name: @project.name@
      version: @project.version@
```

## First Run

1. **Start your application**
2. **Check metrics endpoint**: Visit `http://localhost:8080/actuator/prometheus`
3. **Look for Hofund metrics**:

```text
# HELP hofund_info Basic information about application
# TYPE hofund_info gauge
hofund_info{application_name="my-app",application_version="1.0.0",id="my-app",} 1.0

# HELP hofund_git_info Basic information about application based on git
# TYPE hofund_git_info gauge
hofund_git_info{branch="main",build_host="my-computer",build_time="2025-08-30T20:45:00+0100",commit_id="abc123f",dirty="false",} 1.0
```

## Adding Your First Connection Check

Create a simple HTTP connection check:

```java
@Configuration
public class ConnectionsConfiguration {
    
    @Bean
    public SimpleHofundHttpConnection externalApiConnection() {
        return new SimpleHofundHttpConnection(
            "external-api", 
            "https://api.example.com/health"
        );
    }
}
```

After adding this configuration and restarting your application, you'll see a new metric:

```text
# HELP hofund_connection Current status of given connection
# TYPE hofund_connection gauge
hofund_connection{id="my-app-external-api",source="my-app",target="external-api",type="http",} 1.0
```

## Next Steps

- **Configure multiple connections**: See [usage.md](usage.md) for detailed configuration examples
- **Customize your setup**: Check [reference.md](reference.md) for all available options
- **Set up Grafana**: Import the provided dashboard for visual monitoring
- **Troubleshooting**: Visit [faq.md](faq.md) for common questions and solutions

## Verification

To verify everything is working correctly:

1. **Check application logs**: Look for connection status logs during startup
2. **Visit Prometheus endpoint**: Ensure hofund metrics are present
3. **Test connection**: If you configured connection checks, verify they appear in metrics

Your Hofund setup is now ready! The tool will automatically:
- Monitor your configured connections
- Expose metrics to Prometheus
- Provide detailed application information
- Track git-based deployment data

## Common First-Time Issues

- **Missing git.properties**: Ensure the git-commit-id-maven-plugin is properly configured
- **No metrics visible**: Check that actuator endpoints are exposed and accessible
- **Connection checks failing**: Verify URLs are accessible from your application's network context

For more detailed configuration and advanced features, continue to the [usage guide](usage.md).