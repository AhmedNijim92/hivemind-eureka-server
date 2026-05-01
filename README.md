# Eureka Server

Service discovery server for the HiveMind microservices platform. All services register here and discover each other through Eureka.

## Details

| Property | Value |
|----------|-------|
| **Port** | `8761` |
| **Database** | None |
| **Role** | Service Discovery |

## Build & Run

```bash
# Build
mvn clean package

# Run
java -jar target/*.jar

# Docker
docker build -t hivemind/eureka-server .
```

## Links

- [Main Repository](https://github.com/AhmedNijim92/hivemind-backend)
