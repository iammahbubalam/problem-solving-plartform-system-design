# 📁 SIMPLIFIED SERVICE STRUCTURES

## 🔐 **AUTH-SERVICE** (Repository: auth-service)

```
auth-service/
├── src/
│   ├── main/
│   │   ├── java/com/platform/auth/
│   │   │   ├── AuthServiceApplication.java
│   │   │   ├── controller/
│   │   │   │   ├── AuthController.java
│   │   │   │   ├── OAuth2Controller.java
│   │   │   │   └── TokenController.java
│   │   │   ├── service/
│   │   │   │   ├── AuthService.java
│   │   │   │   ├── JwtService.java
│   │   │   │   ├── OAuth2Service.java
│   │   │   │   └── SessionService.java
│   │   │   ├── repository/
│   │   │   │   ├── UserRepository.java
│   │   │   │   └── SessionRepository.java
│   │   │   ├── model/
│   │   │   │   ├── User.java
│   │   │   │   ├── Role.java
│   │   │   │   ├── Permission.java
│   │   │   │   └── Session.java
│   │   │   ├── dto/
│   │   │   │   ├── LoginRequest.java
│   │   │   │   ├── LoginResponse.java
│   │   │   │   ├── TokenRequest.java
│   │   │   │   └── TokenResponse.java
│   │   │   ├── grpc/
│   │   │   │   ├── AuthGrpcService.java
│   │   │   │   └── TokenValidationService.java
│   │   │   └── exception/
│   │   │       ├── AuthException.java
│   │   │       ├── TokenExpiredException.java
│   │   │       └── InvalidCredentialsException.java
│   │   └── resources/
│   │       ├── application.yml
│   │       └── db/migration/
│   │           ├── V1__Create_users_table.sql
│   │           ├── V2__Create_roles_table.sql
│   │           └── V3__Create_permissions_table.sql
│   └── test/
│       └── java/com/platform/auth/
│           ├── AuthServiceApplicationTests.java
│           ├── controller/
│           ├── service/
│           └── integration/
├── Dockerfile                           # Service-specific Dockerfile
├── docker-compose.yml                  # Local development setup
├── k8s/                                # Kubernetes manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
├── .dockerignore
├── .gitignore
├── pom.xml                             # Includes platform-contracts dependency
└── README.md
```

**📦 pom.xml includes:**
```xml
<dependency>
    <groupId>com.platform</groupId>
    <artifactId>platform-contracts</artifactId>
    <version>${contracts.version}</version>
</dependency>
```

**🐳 Dockerfile example:**
```dockerfile
FROM openjdk:17-jdk-slim
RUN apt-get update && apt-get install -y redis-tools && rm -rf /var/lib/apt/lists/*
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s CMD curl -f http://localhost:8080/actuator/health || exit 1
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## 👤 **USER-SERVICE** (Repository: user-service)

```
user-service/
├── src/
│   ├── main/
│   │   ├── java/com/platform/user/
│   │   │   ├── UserServiceApplication.java
│   │   │   ├── controller/
│   │   │   │   ├── UserController.java
│   │   │   │   ├── ProfileController.java
│   │   │   │   ├── LeaderboardController.java
│   │   │   │   └── StatisticsController.java
│   │   │   ├── service/
│   │   │   │   ├── UserService.java
│   │   │   │   ├── ProfileService.java
│   │   │   │   ├── LeaderboardService.java
│   │   │   │   ├── StatisticsService.java
│   │   │   │   └── UserSearchService.java
│   │   │   ├── repository/
│   │   │   │   ├── UserRepository.java
│   │   │   │   ├── ProfileRepository.java
│   │   │   │   ├── StatisticsRepository.java
│   │   │   │   └── LeaderboardRepository.java
│   │   │   ├── model/
│   │   │   │   ├── User.java
│   │   │   │   ├── Profile.java
│   │   │   │   ├── Statistics.java
│   │   │   │   ├── Achievement.java
│   │   │   │   └── Preference.java
│   │   │   ├── dto/
│   │   │   │   ├── UserDto.java
│   │   │   │   ├── ProfileDto.java
│   │   │   │   ├── LeaderboardDto.java
│   │   │   │   └── StatisticsDto.java
│   │   │   ├── grpc/
│   │   │   │   ├── UserGrpcService.java
│   │   │   │   └── UserValidationService.java
│   │   │   └── kafka/
│   │   │       ├── UserEventProducer.java
│   │   │       └── UserEventConsumer.java
│   │   └── resources/
│   │       ├── application.yml
│   │       └── db/migration/
│   └── test/
├── Dockerfile                           # Service-specific Dockerfile
├── docker-compose.yml                  # Local development setup
├── k8s/                                # Kubernetes manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
├── .dockerignore
├── pom.xml
└── README.md
```

---

## 📝 **PROBLEM-SERVICE** (Repository: problem-service)

```
problem-service/
├── src/
│   ├── main/
│   │   ├── java/com/platform/problem/
│   │   │   ├── ProblemServiceApplication.java
│   │   │   ├── controller/
│   │   │   │   ├── ProblemController.java
│   │   │   │   ├── CategoryController.java
│   │   │   │   ├── TagController.java
│   │   │   │   └── SearchController.java
│   │   │   ├── service/
│   │   │   │   ├── ProblemService.java
│   │   │   │   ├── CategoryService.java
│   │   │   │   ├── TagService.java
│   │   │   │   ├── SearchService.java
│   │   │   │   └── DifficultyService.java
│   │   │   ├── repository/
│   │   │   │   ├── ProblemRepository.java
│   │   │   │   ├── CategoryRepository.java
│   │   │   │   ├── TagRepository.java
│   │   │   │   └── elasticsearch/
│   │   │   │       └── ProblemSearchRepository.java
│   │   │   ├── model/
│   │   │   │   ├── Problem.java
│   │   │   │   ├── Category.java
│   │   │   │   ├── Tag.java
│   │   │   │   ├── Difficulty.java
│   │   │   │   └── ProblemMetadata.java
│   │   │   ├── dto/
│   │   │   │   ├── ProblemDto.java
│   │   │   │   ├── ProblemCreateDto.java
│   │   │   │   ├── ProblemUpdateDto.java
│   │   │   │   └── SearchResultDto.java
│   │   │   └── grpc/
│   │   │       ├── ProblemGrpcService.java
│   │   │       └── ProblemValidationService.java
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── elasticsearch/
│   │       │   └── problem-mapping.json
│   │       └── db/migration/
│   └── test/
├── Dockerfile                           # Service-specific Dockerfile
├── docker-compose.yml                  # Local development setup
├── k8s/                                # Kubernetes manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
├── .dockerignore
├── pom.xml
└── README.md
```

---

## 🧪 **TESTCASE-SERVICE** (Repository: testcase-service)

```
testcase-service/
├── src/
│   ├── main/
│   │   ├── java/com/platform/testcase/
│   │   │   ├── TestcaseServiceApplication.java
│   │   │   ├── controller/
│   │   │   │   ├── TestcaseController.java
│   │   │   │   ├── SampleTestcaseController.java
│   │   │   │   └── ValidationController.java
│   │   │   ├── service/
│   │   │   │   ├── TestcaseService.java
│   │   │   │   ├── EncryptionService.java
│   │   │   │   ├── ValidationService.java
│   │   │   │   └── TestcaseGeneratorService.java
│   │   │   ├── repository/
│   │   │   │   ├── TestcaseRepository.java
│   │   │   │   └── SampleTestcaseRepository.java
│   │   │   ├── model/
│   │   │   │   ├── Testcase.java
│   │   │   │   ├── SampleTestcase.java
│   │   │   │   ├── TestcaseGroup.java
│   │   │   │   └── ValidationResult.java
│   │   │   ├── dto/
│   │   │   │   ├── TestcaseDto.java
│   │   │   │   ├── TestcaseCreateDto.java
│   │   │   │   ├── SampleTestcaseDto.java
│   │   │   │   └── ValidationResultDto.java
│   │   │   └── grpc/
│   │   │       ├── TestcaseGrpcService.java
│   │   │       └── TestcaseValidationService.java
│   │   └── resources/
│   │       ├── application.yml
│   │       └── db/migration/
│   └── test/
├── Dockerfile                           # Service-specific Dockerfile
├── docker-compose.yml                  # Local development setup
├── k8s/                                # Kubernetes manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
├── .dockerignore
├── pom.xml
└── README.md
```

---

## 📤 **SUBMISSION-SERVICE** (Repository: submission-service)

```
submission-service/
├── src/
│   ├── main/
│   │   ├── java/com/platform/submission/
│   │   │   ├── SubmissionServiceApplication.java
│   │   │   ├── controller/
│   │   │   │   ├── SubmissionController.java
│   │   │   │   ├── SubmissionHistoryController.java
│   │   │   │   └── SubmissionStatusController.java
│   │   │   ├── service/
│   │   │   │   ├── SubmissionService.java
│   │   │   │   ├── CodeValidationService.java
│   │   │   │   ├── LanguageDetectionService.java
│   │   │   │   ├── QueueService.java
│   │   │   │   └── StatusService.java
│   │   │   ├── repository/
│   │   │   │   ├── SubmissionRepository.java
│   │   │   │   └── SubmissionStatusRepository.java
│   │   │   ├── model/
│   │   │   │   ├── Submission.java
│   │   │   │   ├── SubmissionStatus.java
│   │   │   │   ├── SubmissionResult.java
│   │   │   │   └── Language.java
│   │   │   ├── dto/
│   │   │   │   ├── SubmissionDto.java
│   │   │   │   ├── SubmissionCreateDto.java
│   │   │   │   ├── SubmissionStatusDto.java
│   │   │   │   └── SubmissionResultDto.java
│   │   │   ├── kafka/
│   │   │   │   ├── SubmissionEventProducer.java
│   │   │   │   ├── BuildEventConsumer.java
│   │   │   │   └── ExecutionEventConsumer.java
│   │   │   └── websocket/
│   │   │       ├── SubmissionStatusHandler.java
│   │   │       └── WebSocketEventPublisher.java
│   │   └── resources/
│   │       ├── application.yml
│   │       └── db/migration/
│   └── test/
├── Dockerfile                           # Service-specific Dockerfile
├── docker-compose.yml                  # Local development setup
├── k8s/                                # Kubernetes manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
├── .dockerignore
├── pom.xml
└── README.md
```

---

## 🔨 **BUILD-SERVICE** (Repository: build-service)

```
build-service/
├── src/
│   ├── main/
│   │   ├── java/com/platform/build/
│   │   │   ├── BuildServiceApplication.java
│   │   │   ├── controller/
│   │   │   │   ├── BuildController.java
│   │   │   │   ├── BuildHistoryController.java
│   │   │   │   └── BuildStatusController.java
│   │   │   ├── service/
│   │   │   │   ├── BuildService.java
│   │   │   │   ├── CompilerService.java
│   │   │   │   ├── DockerService.java
│   │   │   │   └── ResourceMonitoringService.java
│   │   │   ├── repository/
│   │   │   │   ├── BuildRepository.java
│   │   │   │   └── BuildLogRepository.java
│   │   │   ├── model/
│   │   │   │   ├── Build.java
│   │   │   │   ├── BuildResult.java
│   │   │   │   ├── BuildLog.java
│   │   │   │   ├── CompilerConfig.java
│   │   │   │   └── BuildEnvironment.java
│   │   │   ├── dto/
│   │   │   │   ├── BuildRequestDto.java
│   │   │   │   ├── BuildResultDto.java
│   │   │   │   ├── BuildStatusDto.java
│   │   │   │   └── CompilerConfigDto.java
│   │   │   ├── kafka/
│   │   │   │   ├── SubmissionEventConsumer.java
│   │   │   │   └── BuildEventProducer.java
│   │   │   └── compiler/
│   │   │       ├── JavaCompiler.java
│   │   │       ├── PythonCompiler.java
│   │   │       ├── CppCompiler.java
│   │   │       ├── JavaScriptRunner.java
│   │   │       └── CompilerFactory.java
│   │   └── resources/
│   │       ├── application.yml
│   │       └── compiler-configs/
│   └── test/
├── Dockerfile                           # Service-specific Dockerfile
├── docker-compose.yml                  # Local development setup
├── k8s/                                # Kubernetes manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
├── .dockerignore
├── pom.xml
└── README.md
```

---

## ⚡ **EXECUTION-SERVICE** (Repository: execution-service)

```
execution-service/
├── src/
│   ├── main/
│   │   ├── java/com/platform/execution/
│   │   │   ├── ExecutionServiceApplication.java
│   │   │   ├── controller/
│   │   │   │   ├── ExecutionController.java
│   │   │   │   ├── ExecutionHistoryController.java
│   │   │   │   └── ExecutionStatusController.java
│   │   │   ├── service/
│   │   │   │   ├── ExecutionService.java
│   │   │   │   ├── SandboxService.java
│   │   │   │   ├── ResourceLimitService.java
│   │   │   │   ├── ResultComparisonService.java
│   │   │   │   └── PerformanceService.java
│   │   │   ├── repository/
│   │   │   │   ├── ExecutionRepository.java
│   │   │   │   ├── ExecutionLogRepository.java
│   │   │   │   └── PerformanceRepository.java
│   │   │   ├── model/
│   │   │   │   ├── Execution.java
│   │   │   │   ├── ExecutionResult.java
│   │   │   │   ├── ExecutionLog.java
│   │   │   │   ├── ResourceUsage.java
│   │   │   │   └── TestCaseResult.java
│   │   │   ├── dto/
│   │   │   │   ├── ExecutionRequestDto.java
│   │   │   │   ├── ExecutionResultDto.java
│   │   │   │   ├── ResourceUsageDto.java
│   │   │   │   └── TestCaseResultDto.java
│   │   │   ├── kafka/
│   │   │   │   ├── BuildEventConsumer.java
│   │   │   │   └── ExecutionEventProducer.java
│   │   │   ├── sandbox/
│   │   │   │   ├── DockerSandbox.java
│   │   │   │   ├── ContainerManager.java
│   │   │   │   └── SecurityPolicy.java
│   │   │   └── executor/
│   │   │       ├── CodeExecutor.java
│   │   │       ├── TestCaseRunner.java
│   │   │       └── ResultAnalyzer.java
│   │   └── resources/
│   │       ├── application.yml
│   │       └── sandbox-policies/
│   └── test/
├── Dockerfile                           # Service-specific Dockerfile
├── docker-compose.yml                  # Local development setup
├── k8s/                                # Kubernetes manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
├── .dockerignore
├── pom.xml
└── README.md
```

---

## 🏆 **CONTEST-SERVICE** (Repository: contest-service)

```
contest-service/
├── src/
│   ├── main/
│   │   ├── java/com/platform/contest/
│   │   │   ├── ContestServiceApplication.java
│   │   │   ├── controller/
│   │   │   │   ├── ContestController.java
│   │   │   │   ├── LeaderboardController.java
│   │   │   │   ├── ParticipationController.java
│   │   │   │   └── ContestAnalyticsController.java
│   │   │   ├── service/
│   │   │   │   ├── ContestService.java
│   │   │   │   ├── LeaderboardService.java
│   │   │   │   ├── ScoringService.java
│   │   │   │   ├── SchedulingService.java
│   │   │   │   └── AnalyticsService.java
│   │   │   ├── repository/
│   │   │   │   ├── ContestRepository.java
│   │   │   │   ├── ParticipationRepository.java
│   │   │   │   ├── LeaderboardRepository.java
│   │   │   │   └── ContestAnalyticsRepository.java
│   │   │   ├── model/
│   │   │   │   ├── Contest.java
│   │   │   │   ├── Participation.java
│   │   │   │   ├── Leaderboard.java
│   │   │   │   ├── ContestProblem.java
│   │   │   │   └── ContestAnalytics.java
│   │   │   ├── dto/
│   │   │   │   ├── ContestDto.java
│   │   │   │   ├── ContestCreateDto.java
│   │   │   │   ├── LeaderboardDto.java
│   │   │   │   ├── ParticipationDto.java
│   │   │   │   └── ContestAnalyticsDto.java
│   │   │   ├── websocket/
│   │   │   │   ├── LeaderboardWebSocketHandler.java
│   │   │   │   ├── ContestUpdatesHandler.java
│   │   │   │   └── WebSocketEventPublisher.java
│   │   │   ├── grpc/
│   │   │   │   └── ContestGrpcService.java
│   │   │   └── scoring/
│   │   │       ├── ScoringAlgorithm.java
│   │   │       ├── ICPCScoring.java
│   │   │       └── CodeforceScoring.java
│   │   └── resources/
│   │       ├── application.yml
│   │       └── db/migration/
│   └── test/
├── Dockerfile                           # Service-specific Dockerfile
├── docker-compose.yml                  # Local development setup
├── k8s/                                # Kubernetes manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
├── .dockerignore
├── pom.xml
└── README.md
```

---

## 🔔 **NOTIFICATION-SERVICE** (Repository: notification-service)

```
notification-service/
├── src/
│   ├── main/
│   │   ├── java/com/platform/notification/
│   │   │   ├── NotificationServiceApplication.java
│   │   │   ├── controller/
│   │   │   │   ├── NotificationController.java
│   │   │   │   ├── TemplateController.java
│   │   │   │   └── PreferenceController.java
│   │   │   ├── service/
│   │   │   │   ├── NotificationService.java
│   │   │   │   ├── EmailService.java
│   │   │   │   ├── SmsService.java
│   │   │   │   ├── PushNotificationService.java
│   │   │   │   ├── TemplateService.java
│   │   │   │   └── PreferenceService.java
│   │   │   ├── repository/
│   │   │   │   ├── NotificationRepository.java
│   │   │   │   ├── TemplateRepository.java
│   │   │   │   └── PreferenceRepository.java
│   │   │   ├── model/
│   │   │   │   ├── Notification.java
│   │   │   │   ├── NotificationTemplate.java
│   │   │   │   ├── NotificationPreference.java
│   │   │   │   └── NotificationChannel.java
│   │   │   ├── dto/
│   │   │   │   ├── NotificationDto.java
│   │   │   │   ├── TemplateDto.java
│   │   │   │   ├── PreferenceDto.java
│   │   │   │   └── NotificationRequestDto.java
│   │   │   ├── rabbitmq/
│   │   │   │   ├── NotificationConsumer.java
│   │   │   │   └── DeadLetterHandler.java
│   │   │   ├── providers/
│   │   │   │   ├── EmailProvider.java
│   │   │   │   ├── SmsProvider.java
│   │   │   │   ├── PushProvider.java
│   │   │   │   └── WebSocketProvider.java
│   │   │   └── websocket/
│   │   │       ├── NotificationWebSocketHandler.java
│   │   │       └── WebSocketNotificationPublisher.java
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── templates/
│   │       │   ├── email/
│   │       │   ├── sms/
│   │       │   └── push/
│   │       └── db/migration/
│   └── test/
├── Dockerfile                           # Service-specific Dockerfile
├── docker-compose.yml                  # Local development setup
├── k8s/                                # Kubernetes manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
├── .dockerignore
├── pom.xml
└── README.md
```

---

## 🏗️ **PLATFORM-INFRASTRUCTURE** (Repository: platform-infrastructure)
### **🎯 SINGLE SOURCE OF TRUTH FOR SHARED COMPONENTS & CONTRACTS**

```
platform-infrastructure/
├── services/                             # Git submodules (complete service repositories)
│   ├── auth-service/                     # Submodule -> https://github.com/your-org/auth-service.git
│   │   ├── src/
│   │   ├── Dockerfile
│   │   ├── k8s/
│   │   └── pom.xml
│   ├── user-service/                     # Submodule -> https://github.com/your-org/user-service.git
│   │   ├── src/
│   │   ├── Dockerfile
│   │   ├── k8s/
│   │   └── pom.xml
│   ├── problem-service/                  # Submodule -> https://github.com/your-org/problem-service.git
│   ├── testcase-service/                 # Submodule -> https://github.com/your-org/testcase-service.git
│   ├── submission-service/               # Submodule -> https://github.com/your-org/submission-service.git
│   ├── build-service/                    # Submodule -> https://github.com/your-org/build-service.git
│   ├── execution-service/                # Submodule -> https://github.com/your-org/execution-service.git
│   ├── contest-service/                  # Submodule -> https://github.com/your-org/contest-service.git
│   └── notification-service/             # Submodule -> https://github.com/your-org/notification-service.git
├── shared-contracts/                     # gRPC contracts and API definitions
│   ├── proto/
│   │   ├── auth/
│   │   │   ├── auth.proto
│   │   │   └── token.proto
│   │   ├── user/
│   │   │   ├── user.proto
│   │   │   └── profile.proto
│   │   ├── problem/
│   │   │   ├── problem.proto
│   │   │   └── search.proto
│   │   ├── testcase/
│   │   │   └── testcase.proto
│   │   ├── submission/
│   │   │   └── submission.proto
│   │   ├── build/
│   │   │   └── build.proto
│   │   ├── execution/
│   │   │   └── execution.proto
│   │   ├── contest/
│   │   │   └── contest.proto
│   │   ├── notification/
│   │   │   └── notification.proto
│   │   └── common/
│   │       ├── error-codes.proto
│   │       ├── pagination.proto
│   │       └── metadata.proto
│   ├── java/
│   │   └── generated/
│   ├── scripts/
│   │   ├── generate-contracts.sh         # Generate Java classes from proto
│   │   ├── publish-contracts.sh          # Publish to Maven repository
│   │   └── update-version.sh             # Version management
│   ├── pom.xml                          # Maven config for publishing contracts
│   └── README.md
├── shared-infrastructure/                 # Shared components only
│   ├── kong/                             # API Gateway
│   │   ├── Dockerfile
│   │   ├── k8s/
│   │   │   ├── deployment.yaml
│   │   │   ├── service.yaml
│   │   │   └── configmap.yaml
│   │   └── config/
│   │       ├── kong.conf
│   │       └── plugins/
│   ├── kafka/                            # Message Broker
│   │   ├── k8s/
│   │   │   ├── deployment.yaml
│   │   │   ├── service.yaml
│   │   │   └── configmap.yaml
│   │   └── config/
│   │       ├── server.properties
│   │       └── topics.yaml
│   ├── rabbitmq/                         # Alternative Message Broker
│   │   ├── k8s/
│   │   │   ├── deployment.yaml
│   │   │   ├── service.yaml
│   │   │   └── configmap.yaml
│   │   └── config/
│   │       ├── rabbitmq.conf
│   │       └── definitions.json
│   ├── databases/
│   │   ├── postgres/
│   │   │   ├── k8s/
│   │   │   │   ├── deployment.yaml
│   │   │   │   ├── service.yaml
│   │   │   │   └── configmap.yaml
│   │   │   └── init.sql
│   │   ├── mongodb/
│   │   │   ├── k8s/
│   │   │   │   ├── deployment.yaml
│   │   │   │   ├── service.yaml
│   │   │   │   └── configmap.yaml
│   │   │   └── init.js
│   │   ├── redis/
│   │   │   ├── k8s/
│   │   │   │   ├── deployment.yaml
│   │   │   │   ├── service.yaml
│   │   │   │   └── configmap.yaml
│   │   │   └── redis.conf
│   │   └── elasticsearch/
│   │       ├── k8s/
│   │       │   ├── deployment.yaml
│   │       │   ├── service.yaml
│   │       │   └── configmap.yaml
│   │       └── elasticsearch.yml
│   └── monitoring/
│       ├── prometheus/
│       │   ├── k8s/
│       │   │   ├── deployment.yaml
│       │   │   ├── service.yaml
│       │   │   └── configmap.yaml
│       │   └── config/
│       ├── grafana/
│       │   ├── k8s/
│       │   │   ├── deployment.yaml
│       │   │   ├── service.yaml
│       │   │   └── configmap.yaml
│       │   └── dashboards/
│       └── jaeger/
│           ├── k8s/
│           │   ├── deployment.yaml
│           │   ├── service.yaml
│           │   └── configmap.yaml
│           └── config/
├── deployment/                           # Orchestration scripts and configs
│   ├── docker-compose.yml               # Complete stack for local development
│   └── kubernetes/
│       ├── namespace.yaml
│       ├── deploy-infrastructure.yaml   # Deploy shared components first
│       ├── deploy-services.yaml         # Deploy services second (references service images)
│       ├── configmap.yaml              # Global configurations
│       ├── secrets.yaml                # All secrets
│       └── ingress.yaml                # Single ingress for all services
├── scripts/                            # Automation scripts
│   ├── deploy-infrastructure.sh        # Deploy shared components only
│   ├── deploy-services.sh              # Deploy all services (uses pre-built images)
│   ├── setup-local.sh                  # Local development setup
│   ├── teardown.sh                     # Clean shutdown
│   ├── scale-service.sh                # Scale individual services
│   ├── auto-scale-all.sh               # Apply auto-scaling to all services
│   ├── update-service.sh               # Rolling updates
│   └── backup.sh                       # Data backup
├── scaling/                            # Centralized scaling configurations
│   ├── environments/
│   │   ├── development/
│   │   │   ├── auth-service-hpa.yaml
│   │   │   ├── user-service-hpa.yaml
│   │   │   └── execution-service-hpa.yaml
│   │   ├── staging/
│   │   │   ├── auth-service-hpa.yaml
│   │   │   ├── user-service-hpa.yaml
│   │   │   └── execution-service-hpa.yaml
│   │   └── production/
│   │       ├── auth-service-hpa.yaml
│   │       ├── user-service-hpa.yaml
│   │       ├── build-service-hpa.yaml
│   │       ├── execution-service-hpa.yaml
│   │       └── contest-service-hpa.yaml
│   ├── policies/
│   │   ├── cpu-intensive-services.yaml  # For build/execution services
│   │   ├── memory-intensive-services.yaml
│   │   └── burst-capable-services.yaml  # For execution service
│   ├── global/
│   │   ├── default-hpa-template.yaml    # GLOBAL HPA template for all services
│   │   ├── default-resource-limits.yaml # GLOBAL resource limits for all services
│   │   └── global-scaling-config.yaml   # GLOBAL scaling behavior configuration
│   └── vertical-scaling/
│       ├── vpa-auth-service.yaml        # Vertical Pod Autoscaler
│       ├── vpa-build-service.yaml
│       └── vpa-execution-service.yaml
├── config/                              # Centralized configuration
│   ├── application.yaml                 # Common service config
│   ├── database-connections.yaml       # Database connection strings
│   ├── messaging-config.yaml           # Kafka/RMQ endpoints
│   ├── grpc-endpoints.yaml             # gRPC service discovery
│   ├── security-config.yaml            # JWT, OAuth configs
│   ├── monitoring-config.yaml          # Observability settings
│   └── global-scaling-defaults.yaml    # GLOBAL scaling configuration for all services
├── monitoring/                         # Observability configurations
│   ├── prometheus/
│   │   ├── rules/
│   │   └── targets.yaml
│   ├── grafana/
│   │   ├── dashboards/
│   │   │   ├── platform-overview.json
│   │   │   ├── service-metrics.json
│   │   │   ├── kafka-monitoring.json
│   │   │   └── grpc-monitoring.json
│   │   └── provisioning/
│   └── alerts/
│       ├── service-alerts.yaml
│       └── infrastructure-alerts.yaml
├── docs/
│   ├── QUICK_START.md                  # Get everything running in 5 minutes
│   ├── DEPLOYMENT.md                   # How to deploy to K8s
│   ├── SCALING.md                      # How to scale services
│   └── TROUBLESHOOTING.md              # Common issues
├── .env.example                        # Environment variables template
├── .gitignore
├── README.md                          # Platform documentation
└── Makefile                           # Simple commands: make deploy-infra, make deploy-services
```

**🔑 KEY PRINCIPLES (Updated for Standard Approach):**
- **Service Independence**: Each service owns its Dockerfile, docker-compose.yml, and k8s/ manifests
- **Platform-Infrastructure Focuses On**: Shared components (databases, message brokers, monitoring)
- **Deployment Orchestration**: Scripts reference pre-built service images from registries
- **Contract Management**: Centralized gRPC contracts published as Maven artifacts
- **Infrastructure Only**: No service-specific build or deployment configs
- **Standard Industry Practice**: Services are self-contained and independently deployable

---

## 🔄 **HOW SERVICES ACCESS SHARED CONTRACTS:**

### **📦 Maven Artifact Approach (Recommended):**

1. **Platform-Infrastructure publishes contracts:**
   ```bash
   cd platform-infrastructure/shared-contracts
   ./scripts/generate-contracts.sh    # Generate Java from proto
   ./scripts/publish-contracts.sh     # Publish to Maven repo
   ```

2. **Services consume contracts via Maven dependency:**
   ```xml
   <!-- In service pom.xml -->
   <dependency>
       <groupId>com.platform</groupId>
       <artifactId>platform-contracts</artifactId>
       <version>1.0.0</version>
   </dependency>
   ```

3. **Services import and use:**
   ```java
   import com.platform.contracts.auth.AuthServiceGrpc;
   import com.platform.contracts.user.UserServiceGrpc;
   ```

---

## 🚀 **DEPLOYMENT WORKFLOW (Standard Industry Approach):**

### **� Service Build & Deploy:**
```bash
# Each service builds and deploys independently
cd auth-service/
docker build -t auth-service:latest .
docker push registry.com/auth-service:latest
kubectl apply -f k8s/
```

### **🏗️ Infrastructure Setup:**
```bash
# Platform-infrastructure manages shared components only
cd platform-infrastructure/
./scripts/deploy-infrastructure.sh    # Deploy databases, kafka, etc.
./scripts/deploy-services.sh          # Reference service images from registry
```

### **�🔄 Contract Update Workflow:**
1. **Update contracts** in `platform-infrastructure/shared-contracts/`
2. **Generate & publish** new version: `./scripts/publish-contracts.sh`
3. **Update services** to use new version in their `pom.xml`
4. **Build & deploy services** individually from their repositories

### **🎯 Benefits of Standard Approach:**
- **Service Ownership**: Each team owns their build, deploy, and scaling
- **Independent Development**: Services can be developed and deployed separately
- **Standard Industry Practice**: Follows microservices best practices
- **Clear Separation**: Infrastructure vs application concerns
- **Flexibility**: Different services can use different technologies/databases
- **CI/CD Friendly**: Each service has its own pipeline and deployment cycle

This structure provides complete separation of concerns with each microservice in its own repository managing its own lifecycle, while platform-infrastructure focuses solely on shared components and contracts.


1. User Request → Ingress Controller (Load Balancer)
2. Ingress → Kong API Gateway (Rate Limiting + Load Balancing)  
3. Kong → submission-service (ClusterIP Service Load Balancing)
4. Service → Available submission-service pod (Round-Robin)
5. Pod processes request → Publishes to Kafka
6. Kafka → build-service pods (Auto-scaled based on queue depth)
7. Build complete → execution-service pods (Auto-scaled for burst)