# order-service

Sample Spring Boot 3 application using Java 21.

Build and test:
```bash
mvn clean test package
```

Build Docker image from this directory:
```bash
docker build -t order-service:local .
```

Health endpoint: `GET /health`
