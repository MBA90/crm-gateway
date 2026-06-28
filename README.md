# crm-gateway

API gateway for the CRM platform, built with Spring Cloud Gateway. It serves as the
single entry point that routes incoming requests to the downstream CRM services.

## Tech stack

- Java 21
- Spring Boot 3.5.16
- Spring Cloud Gateway (Server WebMVC) — Spring Cloud 2025.0.3
- Spring Boot Actuator
- Maven

## Requirements

- JDK 21+
- Maven (or use the bundled `./mvnw` wrapper)

## Configuration

Application settings live in `src/main/resources/application.yaml`:

| Property                  | Default       | Description              |
| ------------------------- | ------------- | ------------------------ |
| `spring.application.name` | `crm-gateway` | Application name         |
| `server.port`            | `8100`        | Port the gateway listens on |

Add Spring Cloud Gateway route definitions to this file to forward traffic to the
downstream CRM services.

## Building

```bash
./mvnw clean package
```

This produces an executable JAR under `target/`, which can be run with:

```bash
java -jar target/crm-gateway-0.0.1-SNAPSHOT.jar
```

## Testing

```bash
./mvnw test
```

## Running

```bash
./mvnw spring-boot:run
```

The gateway starts on port **8100** by default (see `src/main/resources/application.yaml`).

## Monitoring

Actuator endpoints are available under `/actuator` for health checks and metrics.