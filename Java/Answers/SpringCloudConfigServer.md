# Spring Cloud Config Server

## What is it?

Spring Cloud Config Server is a **centralized external configuration management** system for distributed microservices. Instead of each service having its own `application.properties` or `application.yml` baked into its JAR, all configuration lives in one place — typically a Git repository — and each service fetches its config at startup (and optionally at runtime).

```
[Git Repo / Vault / File System]
         |
  [Config Server]  ← single source of truth
    /    |    \
[svc-A] [svc-B] [svc-C]   ← each fetches its own config at startup
```

---

## Why We Use It

| Problem without it | How Config Server solves it |
|---|---|
| Config changes require a rebuild + redeploy | Change value in Git, refresh — no redeploy needed |
| Different values for dev/staging/prod hardcoded in JARs | Profile-based config (`application-prod.yml`) served per environment |
| Secrets scattered across services | Centralized, can back it with Vault for encryption |
| No audit trail for config changes | Git history = full audit log |
| Inconsistency across 20 microservices | One repo, one source of truth |

---

## How I Used It in Projects

### Setup — Config Server

```java
// pom.xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableConfigServer          // single annotation to turn it on
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

```yaml
# application.yml of the Config Server itself
server:
  port: 8888

spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/myorg/config-repo   # where configs live
          default-label: main
          search-paths: '{application}'               # subfolder per service
```

---

### Config Repository Structure (Git)

```
config-repo/
├── application.yml           # shared config for ALL services
├── payment-service/
│   ├── application.yml       # payment-service defaults
│   ├── application-dev.yml   # dev overrides
│   └── application-prod.yml  # prod overrides
└── order-service/
    ├── application.yml
    └── application-prod.yml
```

---

### Setup — Config Client (each microservice)

```java
// pom.xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

```yaml
# bootstrap.yml (loaded before application.yml — critical!)
spring:
  application:
    name: payment-service       # tells config server which config to fetch
  config:
    import: "configserver:http://localhost:8888"
  profiles:
    active: prod                # fetches application-prod.yml
```

The Config Server URL it hits:
```
GET http://localhost:8888/payment-service/prod
```
Config Server resolves this to: `application.yml` + `payment-service/application.yml` + `payment-service/application-prod.yml` (merged, most specific wins).

---

### Consuming Config Values in Code

```java
@Component
public class PaymentConfig {

    @Value("${payment.gateway.url}")
    private String gatewayUrl;

    @Value("${payment.timeout.ms:5000}")   // 5000 as default
    private int timeoutMs;
}
```

Or with `@ConfigurationProperties` (preferred for grouped config):

```java
@ConfigurationProperties(prefix = "payment")
@Component
public class PaymentProperties {
    private String gatewayUrl;
    private int timeoutMs;
    // getters/setters
}
```

---

### Runtime Refresh (no restart needed)

Add the Actuator dependency and annotate beans that should pick up new values:

```java
// pom.xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```yaml
# application.yml of the client
management:
  endpoints:
    web:
      exposure:
        include: refresh   # expose the /actuator/refresh endpoint
```

```java
@RefreshScope           // bean re-created when /actuator/refresh is hit
@Component
public class PaymentConfig {
    @Value("${payment.gateway.url}")
    private String gatewayUrl;
}
```

To push new config live:
```bash
# 1. Push change to Git
git push origin main

# 2. Hit the refresh endpoint on the service
POST http://payment-service:8080/actuator/refresh
```

For many services, we used **Spring Cloud Bus** (backed by Kafka or RabbitMQ) to broadcast the refresh event to all instances at once instead of hitting each one manually.

---

## Key Interview Points

1. **Config Server fetches config from Git on every request** (with caching) — Git is the source of truth, not the server's memory.

2. **Priority order** (most specific wins):
   `{app}-{profile}.yml` > `{app}.yml` > `application-{profile}.yml` > `application.yml`

3. **bootstrap.yml vs application.yml** — Config client uses `bootstrap.yml` to connect to Config Server *before* the main context loads. Without it, the service won't know where to fetch its config.

4. **Failure behavior** — By default, if Config Server is unreachable, the client fails fast. You can set `spring.cloud.config.fail-fast=false` to fall back to local config.

5. **Security** — In production we used Spring Cloud Config with Vault backend to store secrets encrypted, never in plain text in Git.

---

## One-Line Summary for Interviews

> "Spring Cloud Config Server externalizes all microservice configuration into a Git-backed centralized store, so we can change any config value across environments without rebuilding or redeploying services — we just push to Git and call the refresh endpoint."
