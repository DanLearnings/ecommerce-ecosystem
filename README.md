# E-commerce Microservices Ecosystem - A Modern Cloud-Native Architecture Journey

Welcome to the central hub for the E-commerce Microservices Ecosystem! This repository contains the high-level architectural documentation and serves as the main entrypoint for understanding the entire distributed system.

This project is a hands-on exploration of building a production-grade e-commerce platform using microservices architecture, Spring Cloud, and modern DevOps practices. This document outlines the architecture, the technology stack, and the development journey through each phase, highlighting key lessons learned along the way.

You can follow the project's progress and tasks on the **GitHub Project Board** (coming soon).

## 🏛️ Architecture Overview

The ecosystem is built upon a **microservices architecture**, where each service is independently deployable, scalable, and maintainable. This architectural style was chosen to provide flexibility, fault isolation, and the ability to scale individual components based on demand.

```
┌─────────────────────────────────────────────────────────────┐
│                      Service Registry                        │
│                    (Eureka Server - 8761)                    │
└─────────────────────────────────────────────────────────────┘
                              ▲
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
┌───────▼────────┐   ┌────────▼────────┐   ┌───────▼────────┐
│ Config Server  │   │  API Gateway    │   │   Inventory    │
│  (Port 8888)   │   │  (Port 8080)    │   │   Service      │
│                │   │                 │   │  (Port 8081)   │
└────────────────┘   └─────────────────┘   └────────────────┘
        │                     │
        │                     │
        ▼                     ▼
┌─────────────────┐   ┌─────────────────┐
│  Config Repo    │   │  Order Service  │
│   (GitHub)      │   │  (Port 8082)    │
│                 │   │  [PLANNED]      │
└─────────────────┘   └─────────────────┘
```

### Main Components:

-   **🏛️ [ecommerce-ecosystem](https://github.com/DanLearnings/ecommerce-ecosystem):** (You are here) The main entrypoint and architectural documentation hub.
-   **🔍 [ecommerce-service-registry](https://github.com/DanLearnings/ecommerce-service-registry):** Eureka Server for service discovery and registration.
-   **⚙️ [ecommerce-config-server](https://github.com/DanLearnings/ecommerce-config-server):** Centralized configuration management using Spring Cloud Config.
-   **🌐 [ecommerce-api-gateway](https://github.com/DanLearnings/ecommerce-api-gateway):** API Gateway built with Spring Cloud Gateway WebFlux for routing and load balancing.
-   **📦 [ecommerce-inventory-service](https://github.com/DanLearnings/ecommerce-inventory-service):** Business microservice responsible for product catalog and inventory management.
-   **🛒 [ecommerce-order-service](https://github.com/DanLearnings/ecommerce-order-service):** Business microservice for order processing (planned).
-   **📝 [ecommerce-config-repo](https://github.com/DanLearnings/ecommerce-config-repo):** Git repository storing externalized configurations for all services.

## 🛠️ Tech Stack

-   **Backend Framework:** Java 21, Spring Boot 3.5.7
-   **Cloud Framework:** Spring Cloud 2025.0.0 
-   **Service Discovery:** Netflix Eureka
-   **API Gateway:** Spring Cloud Gateway (WebFlux)
-   **Configuration:** Spring Cloud Config (Git-backed)
-   **Database:** H2 (in-memory, development) → PostgreSQL (production planned)
-   **Build Tool:** Maven
-   **Containerization:** Docker
-   **Orchestration:** Docker Compose (local), Kubernetes (planned)
-   **CI/CD:** GitHub Actions (planned)

---

## 🚀 The Development Journey & Key Learnings

This project was built iteratively in distinct phases, each with its own challenges and lessons.

### Phase 1: Infrastructure Services (Service Registry & Config Server)

Before building business logic, I established the foundational infrastructure services that enable the microservices ecosystem to function.

-   **What I did:** Created the Eureka Server for service discovery and the Config Server to centralize configuration management. Set up a Git repository to store externalized configurations, allowing services to pull their settings dynamically at startup.
-   **💡 Key Learning:** The importance of **starting with infrastructure** became clear. Without service discovery and centralized configuration, managing multiple microservices would quickly become a nightmare of hardcoded URLs and scattered configuration files. The Config Server's ability to pull configurations from Git also introduced version control for application settings, which is crucial for tracking changes and enabling rollbacks.

### Phase 2: API Gateway (The Single Entry Point)

With infrastructure in place, I built the API Gateway to serve as the single entry point for all client requests.

-   **What I did:** Implemented Spring Cloud Gateway using the WebFlux (reactive) variant. Configured automatic service discovery via Eureka, allowing the gateway to dynamically route requests to registered services without manual configuration. Exposed health and monitoring endpoints via Spring Boot Actuator.
-   **💡 Key Learning:** The most significant lesson was understanding the **difference between Gateway MVC and Gateway WebFlux**. Initially, I attempted to use the MVC variant (`spring-cloud-starter-gateway-server-webmvc`), but encountered serious limitations with service discovery and routing. After extensive debugging and research, I discovered that **WebFlux is the recommended and fully-functional variant** for Spring Cloud Gateway. The configuration hierarchy also matters: `spring.cloud.gateway.server.webflux.discovery.locator` is the correct path for WebFlux, not just `spring.cloud.gateway.discovery.locator`. This experience reinforced the value of consulting tutorials and real-world examples alongside official documentation.

### Phase 3: Business Services (Inventory Service)

With the infrastructure and gateway operational, I developed the first business microservice.

-   **What I did:** Built a complete REST API for product inventory management following the **Controller → Service → Repository** architectural pattern. Implemented full CRUD operations, integrated with Spring Data JPA and H2 database, and configured the service to register with Eureka and pull configurations from the Config Server. Added business logic for stock management (check stock, decrease stock).
-   **💡 Key Learning:** The biggest challenge was ensuring the service correctly **integrated with the Config Server**. Initially, the service ignored the externalized configuration and used default values. The solution involved using `spring.config.import: "configserver:http://localhost:8888"` in the `application.yml` file. Removing `optional:` from this configuration ensures the service **fails fast** if it cannot connect to the Config Server, which is better than running with incorrect settings. I also learned the importance of proper **service startup order**: Service Registry → Config Server → API Gateway → Business Services.

### Phase 4: End-to-End Testing & Validation

This phase focused on validating that all components work together as a cohesive system.

-   **What I did:** Created a comprehensive Postman collection to test the entire request flow: from the API Gateway to the Inventory Service via Eureka-based service discovery. Validated that products can be created, listed, updated, and deleted through the gateway. Confirmed that the gateway correctly routes requests based on service names registered in Eureka.
-   **💡 Key Learning:** Seeing the entire ecosystem work together was incredibly rewarding. The most important validation was confirming that **requests through the gateway (port 8080) and direct requests to the service (port 8081) produced identical results**. This proved that the gateway, Eureka, and the business service were all correctly integrated. Testing also revealed the value of having both direct and gateway-routed endpoints available during development for debugging purposes.

### Phase 5: Containerization (Docker)

After validating the entire ecosystem running locally with Maven, I containerized each service to enable consistent deployment across different environments.

-   **What I did:** Created **multi-stage Dockerfiles** for all four services (Service Registry, Config Server, API Gateway, and Inventory Service). Implemented best practices including non-root users for security, health checks, optimized layer caching, and Alpine-based images for smaller footprint. Configured Docker networking to enable service-to-service communication using service names. Tested each service individually as a container and validated the entire stack running in Docker.
-   **💡 Key Learning:** The transition from local Maven execution to containerized deployment revealed several important considerations. **Environment variables** became crucial for configuration, replacing hardcoded `localhost` references with container service names (e.g., `config-server` instead of `localhost:8888`). The **startup order** became even more critical in containers, requiring proper use of `depends_on` and health checks. I learned that **multi-stage builds** significantly reduce image size by separating the build environment (with Maven) from the runtime environment (with only JRE). The Config Server required special attention as it needed Git installed in the container to clone the configuration repository. Most importantly, I discovered that **Docker networking** automatically provides DNS resolution between containers, making service discovery seamless when combined with Eureka.

---

## 🏁 Current Status & Next Steps

The project has successfully reached a significant milestone: a fully functional microservices infrastructure with one operational business service, all integrated, tested, and containerized.

**Current Status:**
- ✅ Service Registry (Eureka) - Operational
- ✅ Config Server - Operational
- ✅ API Gateway (WebFlux) - Operational
- ✅ Inventory Service - Fully functional with CRUD operations
- ✅ End-to-end testing via Postman - Validated
- ✅ Docker containerization - All services containerized with multi-stage builds
- ✅ Docker networking - Services communicate via Docker network

**Immediate Next Steps:**

1.  **Docker Compose:** Create a single `docker-compose.yml` file to orchestrate all services with one command, including proper dependency management and health checks.
2.  **Documentation Updates:** Complete individual README files for each microservice with Docker-specific instructions.
3.  **Order Service:** Implement the second business microservice with inter-service communication using Feign Client.
4.  **Resilience:** Add Circuit Breaker pattern using Resilience4j to handle service failures gracefully.

**Future Enhancements:**

1.  **Persistent Database:** Replace H2 with PostgreSQL for production-grade data persistence.
2.  **Security:** Implement JWT-based authentication and authorization at the API Gateway level.
3.  **Observability:** Integrate distributed tracing (Zipkin/Micrometer) and centralized logging (ELK Stack).
4.  **CI/CD Pipeline:** Automate build, test, Docker image creation, and deployment with GitHub Actions.
5.  **Kubernetes Deployment:** Create Kubernetes manifests and deploy to a cloud-based cluster.
6.  **API Documentation:** Add OpenAPI/Swagger documentation for all REST endpoints.

---

## 📚 How to Run Locally

### Prerequisites

-   **Java 21** or higher
-   **Maven 3.8+**
-   **Git**
-   **Docker Desktop** (for containerized deployment)

---

## 🐳 Option 1: Running with Docker (Recommended)

Docker provides a consistent, isolated environment for all services.

### Quick Start

```bash
# 1. Ensure Docker is running
docker --version

# 2. Create Docker network
docker network create ecommerce-network

# 3. Start Service Registry
docker run -d \
  --name service-registry \
  --network ecommerce-network \
  -p 8761:8761 \
  ecommerce-service-registry:latest

# 4. Start Config Server (wait 30 seconds after Service Registry)
docker run -d \
  --name config-server \
  --network ecommerce-network \
  -p 8888:8888 \
  ecommerce-config-server:latest

# 5. Start API Gateway (wait 30 seconds after Config Server)
docker run -d \
  --name api-gateway \
  --network ecommerce-network \
  -p 8080:8080 \
  ecommerce-api-gateway:latest

# 6. Start Inventory Service (wait 30 seconds after API Gateway)
docker run -d \
  --name inventory-service \
  --network ecommerce-network \
  -p 8081:8081 \
  ecommerce-inventory-service:latest
```

### Building Docker Images

If you need to build the images locally:

```bash
# Service Registry
cd ecommerce-service-registry
docker build -t ecommerce-service-registry:latest .

# Config Server
cd ecommerce-config-server
docker build -t ecommerce-config-server:latest .

# API Gateway
cd ecommerce-api-gateway
docker build -t ecommerce-api-gateway:latest .

# Inventory Service
cd ecommerce-inventory-service
docker build -t ecommerce-inventory-service:latest .
```

### Verification

```bash
# Check all containers are running
docker ps

# Check Eureka Dashboard
open http://localhost:8761

# Test API Gateway health
curl http://localhost:8080/actuator/health

# Test Inventory Service via Gateway
curl http://localhost:8080/inventory-service/products
```

### Viewing Logs

```bash
# View logs for a specific service
docker logs -f service-registry
docker logs -f config-server
docker logs -f api-gateway
docker logs -f inventory-service
```

### Stopping Services

```bash
# Stop all services
docker stop service-registry config-server api-gateway inventory-service

# Remove containers
docker rm service-registry config-server api-gateway inventory-service

# Remove network
docker network rm ecommerce-network
```

---

## 💻 Option 2: Running with Maven (Development)

For active development, you may prefer running services directly with Maven.

### Startup Order (Important!)

Services must be started in this specific order:

1.  **Service Registry** (port 8761)
2.  **Config Server** (port 8888)
3.  **API Gateway** (port 8080)
4.  **Inventory Service** (port 8081)

### Step-by-Step Instructions

#### 1. Clone All Repositories

```bash
git clone https://github.com/DanLearnings/ecommerce-service-registry.git
git clone https://github.com/DanLearnings/ecommerce-config-server.git
git clone https://github.com/DanLearnings/ecommerce-api-gateway.git
git clone https://github.com/DanLearnings/ecommerce-inventory-service.git
```

#### 2. Start Service Registry

```bash
cd ecommerce-service-registry
mvn spring-boot:run
```

**Validation:** Open http://localhost:8761 - You should see the Eureka dashboard.

#### 3. Start Config Server

```bash
cd ecommerce-config-server
mvn spring-boot:run
```

**Validation:** 
- Check http://localhost:8761 - CONFIG-SERVER should appear as registered
- Test config endpoint: http://localhost:8888/inventory-service/default

#### 4. Start API Gateway

```bash
cd ecommerce-api-gateway
mvn spring-boot:run
```

**Validation:**
- Check http://localhost:8761 - API-GATEWAY should appear as registered
- Test health: http://localhost:8080/actuator/health

#### 5. Start Inventory Service

```bash
cd ecommerce-inventory-service
mvn spring-boot:run
```

**Validation:**
- Check http://localhost:8761 - INVENTORY-SERVICE should appear as registered
- Test direct access: http://localhost:8081/products (should return `[]`)
- Test via gateway: http://localhost:8080/inventory-service/products (should return `[]`)

---

## 🧪 Testing the API

### Create a Product (via API Gateway)

```bash
curl -X POST http://localhost:8080/inventory-service/products \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Notebook Dell Inspiron",
    "description": "Laptop with 16GB RAM and 512GB SSD",
    "price": 3500.00,
    "quantity": 10,
    "sku": "DELL-NB-001"
  }'
```

### List All Products (via API Gateway)

```bash
curl http://localhost:8080/inventory-service/products
```

### Get Product by ID

```bash
curl http://localhost:8080/inventory-service/products/1
```

### Update Product

```bash
curl -X PUT http://localhost:8080/inventory-service/products/1 \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Notebook Dell Inspiron Updated",
    "description": "Updated laptop description",
    "price": 3800.00,
    "quantity": 8,
    "sku": "DELL-NB-001"
  }'
```

### Delete Product

```bash
curl -X DELETE http://localhost:8080/inventory-service/products/1
```

---

## 🐛 Troubleshooting

### Service fails to start with "Port already in use"

**Solution:** Check if another service is running on the same port and stop it.

```bash
# Windows
netstat -ano | findstr :8080
taskkill /PID <PID> /F

# Linux/Mac
lsof -i :8080
kill -9 <PID>
```

### Docker: Service fails to register with Eureka

**Solution:**
1. Ensure Service Registry container is running and healthy
2. Verify all containers are on the same Docker network: `docker network inspect ecommerce-network`
3. Check container logs: `docker logs <container-name>`
4. Ensure environment variable `EUREKA_CLIENT_SERVICEURL_DEFAULTZONE` uses the container name: `http://service-registry:8761/eureka/`

### Docker: Config Server cannot clone Git repository

**Solution:**
1. Verify the Git repository URL is correct
2. Ensure the repository is public or authentication is configured
3. Check Config Server logs: `docker logs config-server`
4. Verify Git is installed in the container (should be included in the Dockerfile)

### Docker: Services cannot communicate

**Solution:**
1. Verify all containers are on the same network: `docker network inspect ecommerce-network`
2. Test connectivity: `docker exec api-gateway ping service-registry`
3. Ensure service names match container names in environment variables
4. Check firewall settings on your host machine

### API Gateway returns 404 for service requests

**Solution:**
1. Verify the service is registered in Eureka (check http://localhost:8761)
2. Ensure you're using the correct URL pattern: `http://localhost:8080/{service-name}/{endpoint}`
3. Check that Gateway is using WebFlux, not MVC
4. Verify the `spring.cloud.gateway.server.webflux.discovery.locator.enabled: true` configuration
5. Wait 30-60 seconds after service startup for registration to complete

### Config Server cannot find configurations

**Solution:**
1. Verify the Git repository URL in `application.yml`
2. Ensure the repository is public or SSH keys are configured
3. Check the `search-paths` configuration if using subdirectories
4. Test manually: http://localhost:8888/{service-name}/default

### "Cannot resolve symbol" in IntelliJ IDEA

**Solution:**
1. Maven → Reload All Maven Projects
2. File → Invalidate Caches / Restart
3. Rebuild Project

---

## 📖 Additional Resources

-   [Spring Cloud Documentation](https://spring.io/projects/spring-cloud)
-   [Spring Cloud Gateway Documentation](https://spring.io/projects/spring-cloud-gateway)
-   [Netflix Eureka Wiki](https://github.com/Netflix/eureka/wiki)
-   [Spring Cloud Config Documentation](https://cloud.spring.io/spring-cloud-config/)
-   [Docker Documentation](https://docs.docker.com/)
-   [Docker Compose Documentation](https://docs.docker.com/compose/)

---

## 🤝 Contributing

This is a learning project, but suggestions and improvements are welcome! Feel free to open issues or submit pull requests.

---

## 📄 License

This project is for educational purposes.

---

## 👨‍💻 Author

**Dan Learnings**
- GitHub: [@DanrleyBrasil](https://github.com/DanrleyBrasil)
- Organization: [DanLearnings](https://github.com/DanLearnings)

---

**Last Updated:** October 29, 2025
