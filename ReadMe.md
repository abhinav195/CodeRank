# CodeRank



> A highly concurrent, fault-tolerant online code execution platform built on event-driven microservices.



CodeRank is an enterprise-grade backend ecosystem designed to safely compile and execute untrusted user code across multiple languages (Python, Java, C++, JavaScript). It leverages an asynchronous, event-driven architecture to maintain absolute stability and low latency under massive concurrent load.



## Key Features



* **Isolated Docker Sandboxing:** Dynamically provisions resource-constrained (CPU/Memory limits, read-only root FS, disabled networking) Docker containers for every execution to prevent host compromise.

* **Asynchronous Event-Driven Pipeline:** Decouples code ingestion from execution using Apache Kafka. Implements strict Retryable Topics and Dead Letter Queues (DLQs) to guarantee zero data loss during infrastructure outages.

* **High-Concurrency Edge Defense:** Centralized routing via Spring Cloud Gateway, implementing Resilience4j circuit breakers and Redis-backed token-bucket rate limiting to instantly drop abusive traffic before it hits the internal network.

* **Strict Role-Based Access Control (RBAC):** Stateless JWT authentication architecture cleanly separating Admin privileges (problem creation/publishing) from Standard User workflows (code submission and polling).

* **Distributed State Management:** CQRS-inspired flow utilizing PostgreSQL for authoritative state persistence and Redis Cache-Aside patterns for high-speed client polling.



## Prerequisites



Before running this project, ensure you have the following installed on your host machine:



* **Java 21 (JDK)**

* **Apache Maven 3.8+**

* **Docker & Docker Compose** (Ensure the Docker daemon is running)

* **Git**



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

```bash

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

**Full File Architecture**
CodeRank/

├── coderank-auth/

│   ├── src/

│   │   ├── main/

│   │   │   ├── java/

│   │   │   │   └── com/

│   │   │   │       └── coderank/

│   │   │   │           └── auth/

│   │   │   │               ├── config/

│   │   │   │               │   └── RedisConfig.java  [1.1 KB]

│   │   │   │               ├── controller/

│   │   │   │               │   └── AuthController.java  [4.6 KB]

│   │   │   │               ├── dto/

│   │   │   │               │   ├── LoginRequest.java  [546.0 B]

│   │   │   │               │   ├── RegisterRequest.java  [841.0 B]

│   │   │   │               │   └── TokenResponse.java  [442.0 B]

│   │   │   │               ├── entity/

│   │   │   │               │   ├── RefreshToken.java  [1.1 KB]

│   │   │   │               │   └── User.java  [876.0 B]

│   │   │   │               ├── exception/

│   │   │   │               │   └── AuthExceptionHandler.java  [2.7 KB]

│   │   │   │               ├── repository/

│   │   │   │               │   ├── RefreshTokenRepository.java  [833.0 B]

│   │   │   │               │   └── UserRepository.java  [531.0 B]

│   │   │   │               ├── security/

│   │   │   │               │   ├── JwtTokenProvider.java  [2.4 KB]

│   │   │   │               │   └── SecurityConfig.java  [1.8 KB]

│   │   │   │               ├── service/

│   │   │   │               │   ├── LogoutService.java  [1.5 KB]

│   │   │   │               │   ├── RefreshTokenService.java  [3.6 KB]

│   │   │   │               │   ├── TokenCleanupService.java  [895.0 B]

│   │   │   │               │   ├── TokenRefreshService.java  [1.4 KB]

│   │   │   │               │   ├── UserAuthenticationService.java  [1.9 KB]

│   │   │   │               │   └── UserRegistrationService.java  [2.3 KB]

│   │   │   │               └── AuthServiceApplication.java  [582.0 B]

│   │   │   └── resources/

│   │   │       ├── db/

│   │   │       │   └── migration/

│   │   │       │       ├── V1__create_users_table.sql  [711.0 B]

│   │   │       │       ├── V2__create_refresh_tokens_table.sql  [1.1 KB]

│   │   │       │       └── V3__seed_admin_user.sql  [510.0 B]

│   │   │       └── application.yml  [1.1 KB]

│   │   └── test/

│   │       ├── java/

│   │       │   └── com/

│   │       │       └── coderank/

│   │       │           └── auth/

│   │       │               ├── security/

│   │       │               │   └── JwtTokenProviderTest.java  [3.3 KB]

│   │       │               ├── service/

│   │       │               │   ├── LogoutServiceTest.java  [3.1 KB]

│   │       │               │   ├── RefreshTokenServiceTest.java  [6.4 KB]

│   │       │               │   ├── TokenCleanupServiceTest.java  [1.2 KB]

│   │       │               │   ├── TokenRefreshServiceTest.java  [1.9 KB]

│   │       │               │   ├── UserAuthenticationServiceTest.java  [3.4 KB]

│   │       │               │   └── UserRegistrationServiceTest.java  [3.8 KB]

│   │       │               └── AuthServiceApplicationTest.java  [387.0 B]

│   │       └── resources/

│   │           └── application-test.yml  [570.0 B]

│   ├── .dockerignore  [332.0 B]

│   ├── codebase_context.txt  [38.5 KB]

│   ├── Dockerfile  [1.5 KB]

│   └── pom.xml  [4.3 KB]

├── coderank-common/

│   ├── src/

│   │   └── main/

│   │       └── java/

│   │           └── com/

│   │               └── coderank/

│   │                   └── common/

│   │                       ├── config/

│   │                       │   └── JacksonConfig.java  [852.0 B]

│   │                       ├── constants/

│   │                       │   ├── KafkaTopics.java  [1.6 KB]

│   │                       │   └── RedisKeys.java  [992.0 B]

│   │                       ├── dto/

│   │                       │   ├── request/

│   │                       │   │   └── ExecuteRequest.java  [787.0 B]

│   │                       │   └── response/

│   │                       │       ├── ErrorResponse.java  [1.3 KB]

│   │                       │       ├── ExecutionResultResponse.java  [1.1 KB]

│   │                       │       ├── SubmissionDetailResponse.java  [1.2 KB]

│   │                       │       └── SubmissionResponse.java  [855.0 B]

│   │                       ├── enums/

│   │                       │   ├── ExecutionStatus.java  [1.3 KB]

│   │                       │   ├── Language.java  [780.0 B]

│   │                       │   ├── UserRole.java  [548.0 B]

│   │                       │   └── Verdict.java  [880.0 B]

│   │                       ├── event/

│   │                       │   ├── CodeExecutionRequestEvent.java  [2.1 KB]

│   │                       │   ├── CodeExecutionResultEvent.java  [2.3 KB]

│   │                       │   └── StateUpdateEvent.java  [992.0 B]

│   │                       └── exception/

│   │                           ├── CodeRankException.java  [802.0 B]

│   │                           ├── InvalidRequestException.java  [287.0 B]

│   │                           └── ResourceNotFoundException.java  [421.0 B]

│   ├── codebase_context.txt  [29.2 KB]

│   └── pom.xml  [2.1 KB]

├── coderank-execution/

│   ├── src/

│   │   ├── main/

│   │   │   ├── java/

│   │   │   │   └── com/

│   │   │   │       └── coderank/

│   │   │   │           └── execution/

│   │   │   │               ├── client/

│   │   │   │               │   └── ProblemServiceClient.java  [3.4 KB]

│   │   │   │               ├── config/

│   │   │   │               │   ├── AsyncConfig.java  [905.0 B]

│   │   │   │               │   ├── DockerConfig.java  [1.4 KB]

│   │   │   │               │   ├── KafkaConfig.java  [6.6 KB]

│   │   │   │               │   ├── RedisConfig.java  [1004.0 B]

│   │   │   │               │   └── WebClientConfig.java  [521.0 B]

│   │   │   │               ├── consumer/

│   │   │   │               │   └── ExecutionRequestConsumer.java  [7.5 KB]

│   │   │   │               ├── docker/

│   │   │   │               │   └── DockerSandboxRunner.java  [9.8 KB]

│   │   │   │               ├── exception/

│   │   │   │               │   └── NonRetryableExecutionException.java  [638.0 B]

│   │   │   │               ├── model/

│   │   │   │               │   ├── ExecutionConfig.java  [891.0 B]

│   │   │   │               │   ├── ExecutionResult.java  [467.0 B]

│   │   │   │               │   └── TestCaseDto.java  [1.8 KB]

│   │   │   │               ├── service/

│   │   │   │               │   ├── CodeExecutionService.java  [14.9 KB]

│   │   │   │               │   └── LanguageConfigResolver.java  [3.1 KB]

│   │   │   │               └── ExecutionServiceApplication.java  [433.0 B]

│   │   │   └── resources/

│   │   │       └── application.yml  [2.1 KB]

│   │   └── test/

│   │       ├── java/

│   │       │   └── com/

│   │       │       └── coderank/

│   │       │           └── execution/

│   │       │               ├── client/

│   │       │               │   └── ProblemServiceClientTest.java  [7.0 KB]

│   │       │               ├── consumer/

│   │       │               │   └── ExecutionRequestConsumerTest.java  [9.9 KB]

│   │       │               ├── docker/

│   │       │               │   └── DockerSandboxRunnerTest.java  [12.0 KB]

│   │       │               ├── exception/

│   │       │               │   └── NonRetryableExecutionExceptionTest.java  [1.6 KB]

│   │       │               ├── model/

│   │       │               │   └── TestCaseDtoTest.java  [4.2 KB]

│   │       │               └── service/

│   │       │                   ├── CodeExecutionServiceTest.java  [27.5 KB]

│   │       │                   └── LanguageConfigResolverTest.java  [6.2 KB]

│   │       └── resources/

│   │           └── application.yml  [573.0 B]

│   ├── .dockerignore  [332.0 B]

│   ├── codebase_context.txt  [126.1 KB]

│   ├── Dockerfile  [2.2 KB]

│   └── pom.xml  [5.4 KB]

├── coderank-gateway/

│   ├── src/

│   │   ├── main/

│   │   │   ├── java/

│   │   │   │   └── com/

│   │   │   │       └── coderank/

│   │   │   │           └── gateway/

│   │   │   │               ├── config/

│   │   │   │               │   ├── GatewayConfig.java  [2.4 KB]

│   │   │   │               │   ├── JwtProperties.java  [525.0 B]

│   │   │   │               │   └── RedisConfig.java  [1.4 KB]

│   │   │   │               ├── controller/

│   │   │   │               │   └── FallbackController.java  [1.6 KB]

│   │   │   │               ├── filter/

│   │   │   │               │   ├── JwtValidationFilter.java  [6.2 KB]

│   │   │   │               │   ├── RateLimitFilter.java  [4.8 KB]

│   │   │   │               │   └── RequestIdFilter.java  [1.8 KB]

│   │   │   │               └── GatewayApplication.java  [334.0 B]

│   │   │   └── resources/

│   │   │       └── application.yml  [8.6 KB]

│   │   └── test/

│   │       ├── java/

│   │       │   └── com/

│   │       │       └── coderank/

│   │       │           └── gateway/

│   │       │               ├── controller/

│   │       │               │   └── FallbackControllerTest.java  [2.8 KB]

│   │       │               └── filter/

│   │       │                   ├── JwtValidationFilterTest.java  [10.5 KB]

│   │       │                   ├── RateLimitFilterTest.java  [6.1 KB]

│   │       │                   └── RequestIdFilterTest.java  [79.0 B]

│   │       └── resources/

│   │           └── application-test.yml  [832.0 B]

│   ├── .dockerignore  [332.0 B]

│   ├── Dockerfile  [1.5 KB]

│   └── pom.xml  [4.4 KB]

├── coderank-integration-tests/

│   ├── src/

│   │   └── test/

│   │       └── java/

│   │           └── com/

│   │               └── coderank/

│   │                   └── integration/

│   │                       └── CodeRankIntegrationTest.java  [33.6 KB]

│   └── pom.xml  [3.5 KB]

├── coderank-problem/

│   ├── src/

│   │   ├── main/

│   │   │   ├── java/

│   │   │   │   └── com/

│   │   │   │       └── coderank/

│   │   │   │           └── problem/

│   │   │   │               ├── client/

│   │   │   │               │   ├── SubmissionForwardException.java  [346.0 B]

│   │   │   │               │   ├── SubmissionRateLimitException.java  [354.0 B]

│   │   │   │               │   ├── SubmissionServiceClient.java  [3.1 KB]

│   │   │   │               │   └── SubmitCodeRequest.java  [1.0 KB]

│   │   │   │               ├── config/

│   │   │   │               │   ├── KafkaConfig.java  [2.8 KB]

│   │   │   │               │   ├── OpenApiConfig.java  [1.2 KB]

│   │   │   │               │   └── RedisConfig.java  [1.9 KB]

│   │   │   │               ├── controller/

│   │   │   │               │   ├── CompanyController.java  [1.6 KB]

│   │   │   │               │   ├── InternalProblemController.java  [1.6 KB]

│   │   │   │               │   ├── ProblemController.java  [6.4 KB]

│   │   │   │               │   └── TopicController.java  [1.6 KB]

│   │   │   │               ├── dto/

│   │   │   │               │   ├── request/

│   │   │   │               │   │   ├── CreateCompanyRequest.java  [517.0 B]

│   │   │   │               │   │   ├── CreateProblemRequest.java  [1.2 KB]

│   │   │   │               │   │   ├── CreateTopicRequest.java  [511.0 B]

│   │   │   │               │   │   ├── ProblemExampleRequest.java  [555.0 B]

│   │   │   │               │   │   ├── ProblemSubmitRequest.java  [906.0 B]

│   │   │   │               │   │   ├── TestCaseRequest.java  [540.0 B]

│   │   │   │               │   │   └── UpdateProblemRequest.java  [713.0 B]

│   │   │   │               │   └── response/

│   │   │   │               │       ├── CompanyResponse.java  [327.0 B]

│   │   │   │               │       ├── InternalTestCaseResponse.java  [572.0 B]

│   │   │   │               │       ├── ProblemDetailResponse.java  [965.0 B]

│   │   │   │               │       ├── ProblemExampleResponse.java  [433.0 B]

│   │   │   │               │       ├── ProblemSummaryResponse.java  [777.0 B]

│   │   │   │               │       ├── TestCaseResponse.java  [419.0 B]

│   │   │   │               │       └── TopicResponse.java  [325.0 B]

│   │   │   │               ├── entity/

│   │   │   │               │   ├── Company.java  [422.0 B]

│   │   │   │               │   ├── Problem.java  [2.5 KB]

│   │   │   │               │   ├── ProblemExample.java  [861.0 B]

│   │   │   │               │   ├── TestCase.java  [809.0 B]

│   │   │   │               │   ├── Topic.java  [417.0 B]

│   │   │   │               │   └── UserProblemState.java  [1.1 KB]

│   │   │   │               ├── enums/

│   │   │   │               │   ├── Difficulty.java  [90.0 B]

│   │   │   │               │   └── ProblemState.java  [100.0 B]

│   │   │   │               ├── exception/

│   │   │   │               │   └── ProblemExceptionHandler.java  [6.5 KB]

│   │   │   │               ├── kafka/

│   │   │   │               │   └── StateUpdateConsumer.java  [6.4 KB]

│   │   │   │               ├── mapper/

│   │   │   │               │   └── ProblemMapper.java  [3.5 KB]

│   │   │   │               ├── repository/

│   │   │   │               │   ├── CompanyRepository.java  [533.0 B]

│   │   │   │               │   ├── ProblemRepository.java  [1.3 KB]

│   │   │   │               │   ├── TestCaseRepository.java  [425.0 B]

│   │   │   │               │   ├── TopicRepository.java  [523.0 B]

│   │   │   │               │   └── UserProblemStateRepository.java  [551.0 B]

│   │   │   │               ├── security/

│   │   │   │               │   ├── GatewayAuthenticationFilter.java  [2.2 KB]

│   │   │   │               │   └── SecurityConfig.java  [3.8 KB]

│   │   │   │               ├── service/

│   │   │   │               │   ├── CompanyService.java  [1.9 KB]

│   │   │   │               │   ├── ProblemService.java  [10.5 KB]

│   │   │   │               │   └── TopicService.java  [1.8 KB]

│   │   │   │               └── ProblemServiceApplication.java  [582.0 B]

│   │   │   └── resources/

│   │   │       ├── db/

│   │   │       │   └── migration/

│   │   │       │       ├── V1__create_topics_table.sql  [279.0 B]

│   │   │       │       ├── V2__create_companies_table.sql  [294.0 B]

│   │   │       │       ├── V3__create_problems_table.sql  [2.1 KB]

│   │   │       │       ├── V4__create_problem_details_table.sql  [1.4 KB]

│   │   │       │       └── V5__create_user_problem_state_table.sql  [1.0 KB]

│   │   │       └── application.yml  [2.0 KB]

│   │   └── test/

│   │       ├── java/

│   │       │   └── com/

│   │       │       └── coderank/

│   │       │           └── problem/

│   │       │               ├── kafka/

│   │       │               │   └── StateUpdateConsumerTest.java  [12.9 KB]

│   │       │               ├── mapper/

│   │       │               │   └── ProblemMapperTest.java  [12.2 KB]

│   │       │               ├── service/

│   │       │               │   ├── CompanyServiceTest.java  [5.6 KB]

│   │       │               │   ├── ProblemServiceTest.java  [24.7 KB]

│   │       │               │   └── TopicServiceTest.java  [5.5 KB]

│   │       │               └── ProblemServiceApplicationTest.java  [788.0 B]

│   │       └── resources/

│   │           └── application-test.yml  [701.0 B]

│   ├── .dockerignore  [332.0 B]

│   ├── codebase_context.txt  [180.7 KB]

│   ├── Dockerfile  [1.5 KB]

│   └── pom.xml  [6.1 KB]

├── coderank-result-processor/

│   ├── src/

│   │   ├── main/

│   │   │   ├── java/

│   │   │   │   └── com/

│   │   │   │       └── coderank/

│   │   │   │           └── resultprocessor/

│   │   │   │               ├── client/

│   │   │   │               │   └── SubmissionServiceClient.java  [4.5 KB]

│   │   │   │               ├── config/

│   │   │   │               │   ├── KafkaConfig.java  [6.2 KB]

│   │   │   │               │   ├── OpenApiConfig.java  [1.3 KB]

│   │   │   │               │   ├── RedisConfig.java  [1.2 KB]

│   │   │   │               │   ├── SecurityConfig.java  [2.2 KB]

│   │   │   │               │   └── WebClientConfig.java  [1.5 KB]

│   │   │   │               ├── consumer/

│   │   │   │               │   └── ExecutionResultConsumer.java  [5.5 KB]

│   │   │   │               ├── dto/

│   │   │   │               │   └── UpdateSubmissionResultRequest.java  [1.5 KB]

│   │   │   │               ├── exception/

│   │   │   │               │   ├── GlobalExceptionHandler.java  [1.8 KB]

│   │   │   │               │   └── NonRetryableResultException.java  [619.0 B]

│   │   │   │               ├── service/

│   │   │   │               │   └── ResultProcessorService.java  [8.3 KB]

│   │   │   │               └── ResultProcessorApplication.java  [435.0 B]

│   │   │   └── resources/

│   │   │       └── application.yml  [2.2 KB]

│   │   └── test/

│   │       └── java/

│   │           └── com/

│   │               └── coderank/

│   │                   └── resultprocessor/

│   │                       ├── client/

│   │                       │   └── SubmissionServiceClientTest.java  [8.7 KB]

│   │                       ├── common/

│   │                       │   └── CommonModuleCoverageTest.java  [19.9 KB]

│   │                       ├── config/

│   │                       │   └── ConfigBeansTest.java  [7.3 KB]

│   │                       ├── consumer/

│   │                       │   └── ExecutionResultConsumerTest.java  [4.8 KB]

│   │                       ├── dto/

│   │                       │   └── UpdateSubmissionResultRequestTest.java  [3.1 KB]

│   │                       ├── exception/

│   │                       │   ├── GlobalExceptionHandlerTest.java  [4.9 KB]

│   │                       │   └── NonRetryableResultExceptionTest.java  [1.1 KB]

│   │                       ├── service/

│   │                       │   └── ResultProcessorServiceTest.java  [20.1 KB]

│   │                       └── ResultProcessorApplicationTest.java  [1.6 KB]

│   ├── .dockerignore  [332.0 B]

│   ├── codebase_context.txt  [51.8 KB]

│   ├── Dockerfile  [1.8 KB]

│   └── pom.xml  [4.7 KB]

├── coderank-submission/

│   ├── src/

│   │   ├── main/

│   │   │   ├── java/

│   │   │   │   └── com/

│   │   │   │       └── coderank/

│   │   │   │           └── submission/

│   │   │   │               ├── config/

│   │   │   │               │   ├── KafkaConfig.java  [5.8 KB]

│   │   │   │               │   ├── OpenApiConfig.java  [1.2 KB]

│   │   │   │               │   ├── RedisConfig.java  [985.0 B]

│   │   │   │               │   └── SecurityConfig.java  [1.7 KB]

│   │   │   │               ├── controller/

│   │   │   │               │   ├── InternalSubmissionController.java  [1.8 KB]

│   │   │   │               │   └── SubmissionController.java  [5.3 KB]

│   │   │   │               ├── dto/

│   │   │   │               │   ├── request/

│   │   │   │               │   │   ├── RunRequest.java  [692.0 B]

│   │   │   │               │   │   └── SubmitRequest.java  [704.0 B]

│   │   │   │               │   └── response/

│   │   │   │               │       ├── JobResultResponse.java  [2.4 KB]

│   │   │   │               │       ├── SubmissionDetailResponse.java  [1.4 KB]

│   │   │   │               │       └── SubmissionResponse.java  [1.0 KB]

│   │   │   │               ├── entity/

│   │   │   │               │   └── Submission.java  [2.8 KB]

│   │   │   │               ├── enums/

│   │   │   │               │   └── SubmissionType.java  [237.0 B]

│   │   │   │               ├── exception/

│   │   │   │               │   └── SubmissionExceptionHandler.java  [2.6 KB]

│   │   │   │               ├── kafka/

│   │   │   │               │   └── ExecutionResultConsumer.java  [5.8 KB]

│   │   │   │               ├── mapper/

│   │   │   │               │   └── SubmissionMapper.java  [1.7 KB]

│   │   │   │               ├── repository/

│   │   │   │               │   └── SubmissionRepository.java  [688.0 B]

│   │   │   │               ├── security/

│   │   │   │               │   └── PreAuthenticatedUserFilter.java  [2.2 KB]

│   │   │   │               ├── service/

│   │   │   │               │   ├── SubmissionService.java  [14.6 KB]

│   │   │   │               │   └── VerdictResolutionService.java  [2.7 KB]

│   │   │   │               └── SubmissionServiceApplication.java  [513.0 B]

│   │   │   └── resources/

│   │   │       ├── db/

│   │   │       │   └── migration/

│   │   │       │       └── V1__create_submissions_table.sql  [1.7 KB]

│   │   │       └── application.yml  [2.0 KB]

│   │   └── test/

│   │       ├── java/

│   │       │   └── com/

│   │       │       └── coderank/

│   │       │           └── submission/

│   │       │               ├── controller/

│   │       │               │   ├── InternalSubmissionControllerTest.java  [7.4 KB]

│   │       │               │   └── SubmissionControllerTest.java  [18.4 KB]

│   │       │               ├── exception/

│   │       │               │   └── SubmissionExceptionHandlerTest.java  [5.2 KB]

│   │       │               ├── kafka/

│   │       │               │   └── ExecutionResultConsumerTest.java  [9.8 KB]

│   │       │               ├── mapper/

│   │       │               │   └── SubmissionMapperTest.java  [8.4 KB]

│   │       │               ├── security/

│   │       │               │   └── PreAuthenticatedUserFilterTest.java  [7.7 KB]

│   │       │               └── service/

│   │       │                   ├── SubmissionServiceTest.java  [34.6 KB]

│   │       │                   └── VerdictResolutionServiceTest.java  [6.7 KB]

│   │       └── resources/

│   │           └── application.yml  [588.0 B]

│   ├── .dockerignore  [359.0 B]

│   ├── codebase_context.txt  [162.8 KB]

│   ├── Dockerfile  [1.6 KB]

│   └── pom.xml  [6.8 KB]

├── tests/

│   └── integration/

│       ├── CodeRank_Integration_Tests.postman_collection.json  [40.4 KB]

│       └── run_integration_tests.sh  [24.8 KB]

├── .dockerignore  [47.0 B]

├── .gitignore  [1.2 KB]

├── codebase_context.txt  [758.7 KB]

├── docker-compose.yml  [11.1 KB]

├── files.txt  [5.0 KB]

└── pom.xml  [5.7 KB]

**ReadMe End**
