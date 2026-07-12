# CodeRank

> A highly concurrent, fault-tolerant online code execution platform built on event-driven microservices.


CodeRank is an enterprise-grade backend ecosystem designed to safely compile and execute untrusted user code across multiple languages (Python, Java, C++, JavaScript). It leverages an asynchronous, event-driven architecture to maintain absolute stability and low latency under massive concurrent load.


## Key Features


* **Isolated Docker Sandboxing:** Dynamically provisions resource-constrained (CPU/Memory limits, read-only root FS, disabled networking) Docker containers for every execution to prevent host compromise.

* **Asynchronous Event-Driven Pipeline:** Decouples code ingestion from execution using Apache Kafka. Implements strict Retryable Topics and Dead Letter Queues (DLQs) to guarantee zero data loss during infrastructure outages.

* **High-Concurrency Edge Defense:** Centralized routing via Spring Cloud Gateway, implementing Resilience4j circuit breakers and Redis-backed token-bucket rate limiting to instantly drop abusive traffic before it hits the internal network.

* **Strict Role-Based Access Control (RBAC):** Stateless JWT authentication architecture cleanly separating Admin privileges (problem creation/publishing) from Standard User workflows (code submission and polling).

* **Distributed State Management:** CQRS-inspired flow utilizing PostgreSQL for authoritative state persistence and Redis Cache-Aside patterns for high-speed client polling.
* **Full Observability Stack:** Native integration with Micrometer, exposing distributed traces to Jaeger and JVM metrics to Prometheus/Grafana.

* **Container Orchestration:** Fully containerized and deployable via Kubernetes manifests, governed by an automated Jenkins CI/CD pipeline.

## Prerequisites

Before running this project, ensure you have the following installed on your host machine:

* **Java 21 (JDK)**
* **Apache Maven 3.8+**
* **Docker & Docker Compose** (Ensure the Docker daemon is running)
* **Git**
* **kubectl & Minikube (For Kubernetes deployment)**
* **Jenkins (For CI/CD execution)**

## Local Setup Instructions

Follow these exact steps to compile the microservices and boot the complete ecosystem locally.

**1. Configure Local Docker Daemon**

The Execution Service requires access to the Docker API via the `docker-java` SDK. For local development on Docker Desktop:

* Open Docker Desktop Settings.

* Go to **General**.

* Check the box for **"Expose daemon on tcp://localhost:2375 without TLS"**.

* *Warning: Use this setting for local development only.*

**2. Pull Execution Base Images**

The sandboxing engine requires specific lightweight base images to execute user code. Pull these into your local Docker cache before starting the application:

docker pull eclipse-temurin:21-jdk-alpine
docker pull node:20-slim
docker pull gcc:13
docker pull python:3.11-slim

**3. Clone the repository**

git clone https://github.com/abhinav195/CodeRank.git
cd CodeRank

**4. Build the multi-module Maven project**

Compile the parent POM and all child microservices. We skip tests here to expedite the build process, as unit/integration tests require the testcontainers infrastructure to boot.
mvn clean install -DskipTests

**5. Boot the infrastructure and services**

The root directory contains the master docker-compose.yml. This command will spin up all backing services (PostgreSQL databases, Redis, Kafka/Zookeeper) followed by the 6 Spring Boot microservices.
docker-compose up -d --build

**6. Verify the cluster**

Check the logs to ensure the API Gateway (Port 8080) and downstream services are healthy.
docker-compose logs -f gateway

The API Gateway is now accessible at http://localhost:8080. All API interactions must route through this port.

**Observability Dashboards**

Once the cluster is running, the observability stack is automatically provisioned. Access the dashboards via the following local ports:

**Grafana (Metrics & JVM Health):** http://localhost:3000 (Default credentials: admin/admin)
**Prometheus (Scrape Targets):** http://localhost:9090
**Jaeger (Distributed Tracing UI):** http://localhost:16686

**Kubernetes & CI/CD Deployment**
This ecosystem is fully configured for local Kubernetes orchestration using the provided manifests and Jenkins pipeline.

**1. Initialize Local Cluster**
Start your local Minikube cluster and apply the namespace.
minikube start --cpus=6 --memory=8192
kubectl create namespace coderank

**2. Execute Jenkins Pipeline**
Access your local Jenkins server.
Create a new Pipeline job pointing to the Jenkinsfile in the root of this repository.
The pipeline will automatically build the Maven project, create Docker images, and apply the deployment manifests located in the /k8s directory to your Minikube cluster.

**3. Monitor Pod Health**
Ensure all microservices, Kafka brokers, and databases are running securely inside the cluster.
kubectl get pods -w

