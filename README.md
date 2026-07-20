# crm-gateway

API gateway for the CRM platform, built with Spring Cloud Gateway. It serves as the
single entry point that routes incoming requests to the downstream CRM services and
enforces authentication as an OAuth2 resource server.

## Tech stack

- Java 21
- Spring Boot 3.5.16
- Spring Cloud Gateway (Server WebMVC) — Spring Cloud 2025.0.3
- Spring Security OAuth2 Resource Server (JWT)
- Spring Boot Actuator
- Maven

## Requirements

- JDK 21+
- Maven (or use the bundled `./mvnw` wrapper)
- A running OAuth2 authorization server (Keycloak) for JWT issuance

## Security

The gateway acts as an OAuth2 resource server. Every request must carry a valid
`Bearer` JWT — all endpoints require authentication (`anyRequest().authenticated()`).
Tokens are validated against the configured issuer.

- **Sessions:** stateless (no server-side session, no cookies)
- **CSRF:** disabled (no cookie-based auth surface)
- **CORS:** allows origin `http://localhost:5174` with methods
  `GET, POST, PUT, PATCH, DELETE, OPTIONS` and headers `Authorization, Content-Type`

Security is configured in
`src/main/java/com/crm/gateway/config/ResourceServerConfig.java`.

## Configuration

Application settings live in `src/main/resources/application.yaml`:

| Property                                                        | Default                                  | Description                    |
| -------------------------------------------------------------- | ---------------------------------------- | ------------------------------ |
| `spring.application.name`                                      | `crm-gateway`                            | Application name               |
| `server.port`                                                  | `8100`                                   | Port the gateway listens on    |
| `spring.security.oauth2.resourceserver.jwt.issuer-uri`         | `http://localhost:8180/realms/crm-realm` | Keycloak realm issuer URI      |

### Routing

Routes are defined under `spring.cloud.gateway.server.webmvc.routes`. Current routes:

| Route ID           | Path predicate          | Target URI              | Filters         |
| ------------------ | ----------------------- | ----------------------- | --------------- |
| `customer-service` | `/services/customer/**` | `http://localhost:8201` | `StripPrefix=2` |
| `account-service`  | `/services/account/**`  | `http://localhost:8202` | `StripPrefix=2` |

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
Ensure the Keycloak authorization server is reachable at the configured `issuer-uri`,
otherwise requests will be rejected during JWT validation.

## Monitoring

Actuator endpoints are available under `/actuator` for health checks and metrics.

## Docker

A `Dockerfile` is provided that packages the built JAR on top of an
`eclipse-temurin:21-jre-alpine` base image. Build the JAR first, then the image:

```bash
./mvnw clean package
docker build -t crm-gateway .
```

Run the container, mapping the gateway's port (`8100`):

```bash
docker run --rm -p 8100:8100 crm-gateway
```

> **Note:** the `Dockerfile` currently declares `EXPOSE 8201`, but the gateway
> listens on `8100`. `EXPOSE` is documentation-only and does not affect the
> published port; use `-p 8100:8100` when running.

## CI/CD

A `Jenkinsfile` defines a declarative pipeline that:

1. **Build** — runs `mvn clean package -DskipTests` and archives the resulting JAR.
2. **Docker Build & Push** — logs in to Docker Hub, builds the image tagged with the
   Jenkins `BUILD_NUMBER`, and pushes it, then removes the local image afterward.

The pipeline requires JDK 21 (`jdk-21`) and Maven (`maven-3.9`) tool installations
configured in Jenkins, plus a `docker-hub-credentials` username/password credential.
Update the `DOCKER_HUB_USER` environment variable in the `Jenkinsfile` to your own
Docker Hub account.