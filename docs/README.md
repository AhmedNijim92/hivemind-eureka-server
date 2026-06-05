# Eureka Server

> HiveMind Service Discovery Registry

## Overview

The eureka-server provides Netflix Eureka-based service discovery for all HiveMind microservices. Each service registers itself on startup, and the API Gateway uses Eureka to resolve service names to actual network addresses for load-balanced routing.

## Service Info

| Property | Value |
|----------|-------|
| Port | 8761 |
| Service Name | `eureka-server` |
| Dashboard | http://localhost:8761 |
| Spring Boot | 3.3.5 |
| Spring Cloud | 2023.0.3 |
| Java | 17 |

## Architecture

```
┌──────────────────────────────────────────────────────┐
│                  Eureka Server (:8761)                │
│                                                      │
│  Registered Services:                                │
│  ┌─────────────────┐  ┌─────────────────┐          │
│  │ auth-service    │  │ user-service    │          │
│  │ :8081           │  │ :8082           │          │
│  └─────────────────┘  └─────────────────┘          │
│  ┌─────────────────┐  ┌─────────────────┐          │
│  │ group-service   │  │ post-service    │          │
│  │ :8083           │  │ :8084           │          │
│  └─────────────────┘  └─────────────────┘          │
│  ┌─────────────────┐  ┌─────────────────┐          │
│  │ meeting-service │  │ notification-   │          │
│  │ :8085           │  │ service :8086   │          │
│  └─────────────────┘  └─────────────────┘          │
│  ┌─────────────────┐  ┌─────────────────┐          │
│  │ media-service   │  │ api-gateway     │          │
│  │ :8087           │  │ :8080           │          │
│  └─────────────────┘  └─────────────────┘          │
│  ┌─────────────────┐                               │
│  │ config-server   │                               │
│  │ :8888           │                               │
│  └─────────────────┘                               │
└──────────────────────────────────────────────────────┘
```

## Configuration

```yaml
server:
  port: 8761

eureka:
  client:
    register-with-eureka: false  # Doesn't register itself
    fetch-registry: false        # No other Eureka instances to fetch from
  server:
    enable-self-preservation: false  # Disabled for dev (evict quickly)

spring:
  application:
    name: eureka-server
```

### Key Settings

| Setting | Value | Reason |
|---------|-------|--------|
| register-with-eureka | false | Standalone server (not a client) |
| fetch-registry | false | No peer Eureka servers |
| enable-self-preservation | false | Faster eviction during development |

**Note:** In production, enable self-preservation to avoid evicting healthy instances during network partitions.

## Dependencies

- spring-cloud-starter-netflix-eureka-server

That's it — Eureka Server has minimal dependencies by design.

## Running Locally

```bash
cd microservices/eureka-server
mvn spring-boot:run
```

Open the dashboard at http://localhost:8761 to see registered services.

## Startup Order

Eureka Server must start **first** before any other service. The recommended startup sequence:

1. **eureka-server** (this)
2. config-server
3. auth-service
4. user-service, group-service, post-service, meeting-service, notification-service, media-service
5. api-gateway (last — needs all services registered)

## Client Configuration

All other services register with Eureka using:

```yaml
eureka:
  client:
    service-url:
      defaultZone: ${EUREKA_SERVER:http://localhost:8761/eureka}
  instance:
    prefer-ip-address: true
```

## Production Considerations

- Deploy multiple Eureka instances (peer awareness) for HA
- Enable self-preservation mode
- Use DNS-based discovery or Kubernetes service discovery as alternative
- Consider Eureka security (basic auth on the registry)
