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

| Route ID           | Path predicate         | Target URI              | Filters        |
| ------------------ | ---------------------- | ----------------------- | -------------- |
| `customer-service` | `/services/customer/**` | `http://localhost:8201` | `StripPrefix=2` |

`StripPrefix=2` removes the `/services/customer` prefix before forwarding, so a
request to `/services/customer/orders` reaches the downstream service as `/orders`.

Add further Spring Cloud Gateway route definitions to this file to forward traffic
to additional downstream CRM services.

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