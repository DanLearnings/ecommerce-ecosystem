# E-commerce Microservices Ecosystem - A Modern Cloud-Native Architecture Journey

Welcome to the central hub for the E-commerce Microservices Ecosystem! This repository contains the high-level architectural documentation and serves as the main entrypoint for understanding the entire distributed system.

This project is a hands-on exploration of building a production-grade e-commerce platform using microservices architecture, Spring Cloud, event-driven patterns, and modern DevOps practices.

---

## 🏛️ Architecture Overview

The ecosystem is built upon a **microservices architecture**, where each service is independently deployable, scalable, and maintainable. Services communicate both **synchronously** (REST/HTTP) and **asynchronously** (RabbitMQ messaging) depending on the use case.

```
┌─────────────────────────────────────────────────────────────┐
│                   Service Registry (Eureka)                  │
│                        Port 8761                             │
└──────────────────────┬──────────────────────────────────────┘
                       │
          ┌────────────┼────────────┬─────────────┐
          │            │            │             │
┌─────────▼──────┐ ┌──▼──────┐ ┌──▼────────┐ ┌──▼───────────┐
│ Config Server  │ │   API   │ │ Inventory │ │Notification  │
│   Port 8888    │ │ Gateway │ │  Service  │ │   Service    │
│                │ │Port 8080│ │ Port 8081 │ │  Port 8083   │
└────────┬───────┘ └─────────┘ └───────────┘ └──────┬───────┘
         │                                            │
         ▼                                            │
┌────────────────┐          ┌──────────────────┐     │
│  Config Repo   │          │    RabbitMQ      │◄────┘
│   (GitHub)     │          │  Ports 5672      │
│                │          │       15672      │
└────────────────┘          └──────────────────┘
```

### Main Components:

-   **🏛️ [ecommerce-ecosystem](https://github.com/DanLearnings/ecommerce-ecosystem):** (You are here) The main entrypoint and architectural documentation hub
-   **🔍 [ecommerce-service-registry](https://github.com/DanLearnings/ecommerce-service-registry):** Eureka Server for service discovery and registration
-   **⚙️ [ecommerce-config-server](https://github.com/DanLearnings/ecommerce-config-server):** Centralized configuration management using Spring Cloud Config
-   **🌐 [ecommerce-api-gateway](https://github.com/DanLearnings/ecommerce-api-gateway):** API Gateway built with Spring Cloud Gateway WebFlux for routing and load balancing
-   **📦 [ecommerce-inventory-service](https://github.com/DanLearnings/ecommerce-inventory-service):** Business microservice for product catalog and inventory management
-   **📧 [ecommerce-notification-service](https://github.com/DanLearnings/ecommerce-notification-service):** Notification service for sending emails via SMTP and consuming RabbitMQ events
-   **🐰 RabbitMQ:** Message broker for asynchronous event-driven communication
-   **📝 [ecommerce-config-repo](https://github.com/DanLearnings/ecommerce-config-repo):** Git repository storing externalized configurations for all services

---

## 🛠️ Tech Stack

-   **Backend Framework:** Java 21, Spring Boot 3.5.7
-   **Cloud Framework:** Spring Cloud 2025.0.0 
-   **Service Discovery:** Netflix Eureka
-   **API Gateway:** Spring Cloud Gateway (WebFlux)
-   **Configuration:** Spring Cloud Config (Git-backed)
-   **Messaging:** RabbitMQ (AMQP protocol)
-   **Email:** JavaMail (SMTP - Gmail)
-   **Database:** H2 (in-memory, development) → PostgreSQL (planned)
-   **Build Tool:** Maven
-   **Containerization:** Docker (multi-stage builds)
-   **Orchestration:** Docker Compose
-   **CI/CD:** GitHub Actions (planned)
-   **Monitoring:** Spring Boot Actuator

---

## 🚀 The Development Journey & Key Learnings

This project was built iteratively in distinct phases, each with its own challenges and lessons.

### Phase 1: Infrastructure Services (Service Registry & Config Server)

Before building business logic, I established the foundational infrastructure services that enable the microservices ecosystem to function.

-   **What I did:** Created the Eureka Server for service discovery and the Config Server to centralize configuration management. Set up a Git repository to store externalized configurations, allowing services to pull their settings dynamically at startup.
-   **💡 Key Learning:** The importance of **starting with infrastructure** became clear. Without service discovery and centralized configuration, managing multiple microservices would quickly become a nightmare of hardcoded URLs and scattered configuration files. The Config Server's ability to pull configurations from Git also introduced version control for application settings, which is crucial for tracking changes and enabling rollbacks.

### Phase 2: API Gateway (The Single Entry Point)

With infrastructure in place, I built the API Gateway to serve as the single entry point for all client requests.

-   **What I did:** Implemented Spring Cloud Gateway using the WebFlux (reactive) variant. Configured automatic service discovery via Eureka, allowing the gateway to dynamically route requests to registered services without manual configuration.
-   **💡 Key Learning:** The most significant lesson was understanding the **difference between Gateway MVC and Gateway WebFlux**. Initially, I attempted to use the MVC variant, but encountered serious limitations with service discovery and routing. After debugging, I discovered that **WebFlux is the recommended and fully-functional variant** for Spring Cloud Gateway. The configuration hierarchy also matters: `spring.cloud.gateway.server.webflux.discovery.locator` is the correct path for WebFlux.

### Phase 3: Business Services (Inventory Service)

With the infrastructure and gateway operational, I developed the first business microservice.

-   **What I did:** Built a complete REST API for product inventory management following the **Controller → Service → Repository** architectural pattern. Implemented full CRUD operations, integrated with Spring Data JPA and H2 database, and configured the service to register with Eureka and pull configurations from the Config Server.
-   **💡 Key Learning:** The biggest challenge was ensuring the service correctly **integrated with the Config Server**. The solution involved using `spring.config.import: "configserver:http://config-server:8888"` in the application.yml file. Removing `optional:` from this configuration ensures the service **fails fast** if it cannot connect to the Config Server, which is better than running with incorrect settings.

### Phase 4: Containerization (Docker)

After validating the entire ecosystem running locally with Maven, I containerized each service to enable consistent deployment across different environments.

-   **What I did:** Created **multi-stage Dockerfiles** for all services. Implemented best practices including non-root users for security, health checks, optimized layer caching, and Alpine-based images for smaller footprint. Configured Docker networking to enable service-to-service communication using service names.
-   **💡 Key Learning:** The transition from local Maven execution to containerized deployment revealed several important considerations. **Environment variables** became crucial for configuration, replacing hardcoded `localhost` references with container service names. The **startup order** became even more critical in containers, requiring proper use of `depends_on` and health checks. **Multi-stage builds** significantly reduce image size by separating the build environment (with Maven) from the runtime environment (with only JRE).

### Phase 5: Event-Driven Architecture (RabbitMQ & Notification Service)

To enable asynchronous communication between services, I integrated RabbitMQ as a message broker and built the Notification Service.

-   **What I did:** Set up RabbitMQ with management UI, configured exchanges, queues, and bindings. Created the Notification Service to send emails via Gmail SMTP and consume events from RabbitMQ. Implemented event publishers and listeners using Spring AMQP. Designed the system to support multiple notification types (EMAIL, SMS, PUSH) and maintain a history of all notifications sent.
-   **💡 Key Learning:** This phase revealed the power of **event-driven architecture** for decoupling services. Instead of the Inventory Service directly calling the Notification Service (tight coupling), services now publish events to RabbitMQ, and interested services consume them. This approach provides **resilience** (messages persist even if the consumer is down), **scalability** (multiple consumers can process messages in parallel), and **flexibility** (new consumers can be added without modifying publishers). I also learned the importance of **message converters** (Jackson JSON) for seamless object serialization between services.

---

## 🏁 Current Status & Next Steps

The project has successfully reached a significant milestone: a fully functional microservices infrastructure with event-driven communication, all integrated, tested, and containerized.

**Current Status:**
- ✅ Service Registry (Eureka) - Operational
- ✅ Config Server - Operational
- ✅ API Gateway (WebFlux) - Operational
- ✅ Inventory Service - Fully functional with CRUD operations
- ✅ Notification Service - Email sending + RabbitMQ integration
- ✅ RabbitMQ Message Broker - Exchanges, queues, and consumers configured
- ✅ Docker Compose Orchestration - Single command deployment

**Immediate Next Steps:**

1.  **Order Service:** Implement order management with stock reservation logic
2.  **Payment Service:** Simulate payment processing with event publishing
3.  **Inter-Service Communication:** Implement Feign Clients for synchronous calls
4.  **Advanced Messaging:** Add more event types (OrderCreated, PaymentApproved, etc.)

**Future Enhancements:**

1.  **Persistent Database:** Replace H2 with PostgreSQL for production-grade data persistence
2.  **Security:** Implement JWT-based authentication and authorization at the API Gateway level
3.  **Observability:** Integrate distributed tracing (Zipkin) and centralized logging (ELK Stack)
4.  **Resilience:** Add Circuit Breaker pattern using Resilience4j
5.  **CI/CD Pipeline:** Automate build, test, Docker image creation, and deployment with GitHub Actions
6.  **Kubernetes Deployment:** Create Kubernetes manifests and deploy to a cloud-based cluster

---

## 📚 How to Run Locally

### Prerequisites

-   **Java 21** or higher
-   **Maven 3.8+**
-   **Git**
-   **Docker Desktop**

---

## 🐳 Running with Docker Compose (Recommended)

### Quick Start

```bash
# 1. Clone the ecosystem repository
git clone https://github.com/DanLearnings/ecommerce-ecosystem.git
cd ecommerce-ecosystem

# 2. Clone all microservice repositories (in parent directory)
cd ..
git clone https://github.com/DanLearnings/ecommerce-service-registry.git
git clone https://github.com/DanLearnings/ecommerce-config-server.git
git clone https://github.com/DanLearnings/ecommerce-api-gateway.git
git clone https://github.com/DanLearnings/ecommerce-inventory-service.git
git clone https://github.com/DanLearnings/ecommerce-notification-service.git

# 3. Return to ecosystem directory
cd ecommerce-ecosystem

# 4. Start all services with Docker Compose
docker-compose up -d

# 5. View logs
docker-compose logs -f

# 6. Check service status
docker-compose ps
```

### Expected Output

```
NAME                     STATUS          PORTS
rabbitmq                Up (healthy)    5672, 15672
service-registry        Up (healthy)    8761
config-server           Up (healthy)    8888
api-gateway             Up (healthy)    8080
inventory-service       Up (healthy)    8081
notification-service    Up (healthy)    8083
```

### Verification

```bash
# Eureka Dashboard
open http://localhost:8761

# RabbitMQ Management UI (admin/admin)
open http://localhost:15672

# API Gateway Health
curl http://localhost:8080/actuator/health

# Test Inventory Service via Gateway
curl http://localhost:8080/inventory-service/products

# Test Notification Service
curl -X POST http://localhost:8083/notifications/test \
  -H "Content-Type: application/json" \
  -d '{"recipient":"your-email@gmail.com","name":"Test User"}'
```

### Stopping Services

```bash
# Stop all services
docker-compose down

# Stop and remove volumes
docker-compose down -v
```

---

## 🧪 Testing the Complete Flow

### 1. Create a Product

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

### 2. List All Products

```bash
curl http://localhost:8080/inventory-service/products
```

### 3. Send Notification via RabbitMQ

**Via RabbitMQ UI (http://localhost:15672):**
1. Go to **Queues** → `notification-queue`
2. Click **Publish message**
3. Paste this JSON in **Payload**:

```json
{
  "eventType": "TEST_NOTIFICATION",
  "recipient": "your-email@gmail.com",
  "subject": "Test Email from E-commerce System",
  "body": "This email was sent through the event-driven architecture using RabbitMQ!",
  "relatedEntityType": "TEST",
  "relatedEntityId": "test-001"
}
```

4. Click **Publish message**
5. **You should receive an email!** 📧

---

## 🐛 Troubleshooting

### Docker: Service fails to start

**Solution:**
1. Check Docker Desktop is running
2. View container logs: `docker-compose logs <service-name>`
3. Verify all containers are healthy: `docker-compose ps`
4. Restart specific service: `docker-compose restart <service-name>`

### RabbitMQ: Cannot connect

**Solution:**
1. Ensure RabbitMQ container is healthy: `docker ps`
2. Check RabbitMQ logs: `docker-compose logs rabbitmq`
3. Verify port 5672 is not in use by another application
4. Access management UI: http://localhost:15672 (admin/admin)

### Notification Service: Emails not sending

**Solution:**
1. Verify Gmail SMTP credentials in `notification-service.yml` in the config repo
2. Ensure "App Password" is correctly configured (not regular Gmail password)
3. Check Notification Service logs: `docker-compose logs notification-service`
4. Test direct endpoint: `POST http://localhost:8083/notifications/test`

### Services not appearing in Eureka

**Solution:**
1. Wait 30-60 seconds after service startup for registration
2. Check service logs for Eureka registration errors
3. Verify `eureka.client.service-url.defaultZone` points to `http://service-registry:8761/eureka/`
4. Ensure all containers are on the same Docker network

---

## 📖 Additional Resources

-   [Spring Cloud Documentation](https://spring.io/projects/spring-cloud)
-   [Spring Cloud Gateway Documentation](https://spring.io/projects/spring-cloud-gateway)
-   [Netflix Eureka Wiki](https://github.com/Netflix/eureka/wiki)
-   [Spring Cloud Config Documentation](https://cloud.spring.io/spring-cloud-config/)
-   [RabbitMQ Tutorials](https://www.rabbitmq.com/getstarted.html)
-   [Spring AMQP Documentation](https://spring.io/projects/spring-amqp)
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

**Danrley Brasil**
- GitHub: [@DanrleyBrasil](https://github.com/DanrleyBrasil)
- Organization: [DanLearnings](https://github.com/DanLearnings)

---

**Last Updated:** November 2025
