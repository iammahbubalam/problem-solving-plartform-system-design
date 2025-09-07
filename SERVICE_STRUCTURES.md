# 📁 INDIVIDUAL SERVICE FOLDER STRUCTURES

## 🔐 **AUTH-SERVICE** (Repository: auth-service)

```
auth-service/
├── src/
│   ├── main/
│   │   ├── java/com/platform/auth/
│   │   │   ├── AuthServiceApplication.java
│   │   │   ├── config/
│   │   │   │   ├── SecurityConfig.java
│   │   │   │   ├── RedisConfig.java
│   │   │   │   ├── JwtConfig.java
│   │   │   │   └── OAuth2Config.java
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
│   │   │   ├── security/
│   │   │   │   ├── JwtAuthenticationFilter.java
│   │   │   │   ├── JwtTokenProvider.java
│   │   │   │   └── CustomUserDetailsService.java
│   │   │   └── exception/
│   │   │       ├── AuthException.java
│   │   │       ├── TokenExpiredException.java
│   │   │       └── InvalidCredentialsException.java
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── application-dev.yml
│   │       ├── application-prod.yml
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
├── docker/
│   ├── Dockerfile
│   └── docker-compose.yml
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   └── secret.yaml
├── helm/
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
├── .gitignore
├── pom.xml
├── README.md
└── docs/
    ├── API.md
    └── DEPLOYMENT.md
```

---

## 👤 **USER-SERVICE** (Repository: user-service)

```
user-service/
├── src/
│   ├── main/
│   │   ├── java/com/platform/user/
│   │   │   ├── UserServiceApplication.java
│   │   │   ├── config/
│   │   │   │   ├── DatabaseConfig.java
│   │   │   │   ├── RedisConfig.java
│   │   │   │   ├── ElasticsearchConfig.java
│   │   │   │   └── KafkaConfig.java
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
│   │   │   ├── kafka/
│   │   │   │   ├── UserEventProducer.java
│   │   │   │   └── UserEventConsumer.java
│   │   │   └── cache/
│   │   │       ├── UserCacheService.java
│   │   │       └── LeaderboardCacheService.java
│   │   └── resources/
│   │       ├── application.yml
│   │       └── db/migration/
│   └── test/
├── docker/
├── k8s/
├── helm/
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
│   │   │   ├── config/
│   │   │   │   ├── DatabaseConfig.java
│   │   │   │   ├── ElasticsearchConfig.java
│   │   │   │   ├── RedisConfig.java
│   │   │   │   └── SecurityConfig.java
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
│   │   │   ├── grpc/
│   │   │   │   ├── ProblemGrpcService.java
│   │   │   │   └── ProblemValidationService.java
│   │   │   ├── search/
│   │   │   │   ├── ProblemIndexer.java
│   │   │   │   ├── SearchQueryBuilder.java
│   │   │   │   └── SearchResultMapper.java
│   │   │   └── cache/
│   │   │       ├── ProblemCacheService.java
│   │   │       └── SearchCacheService.java
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── elasticsearch/
│   │       │   └── problem-mapping.json
│   │       └── db/migration/
│   └── test/
├── docker/
├── k8s/
├── helm/
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
│   │   │   ├── config/
│   │   │   │   ├── DatabaseConfig.java
│   │   │   │   ├── EncryptionConfig.java
│   │   │   │   └── SecurityConfig.java
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
│   │   │   ├── grpc/
│   │   │   │   ├── TestcaseGrpcService.java
│   │   │   │   └── TestcaseValidationService.java
│   │   │   ├── security/
│   │   │   │   ├── TestcaseEncryption.java
│   │   │   │   └── AccessControlService.java
│   │   │   └── validator/
│   │   │       ├── InputValidator.java
│   │   │       ├── OutputValidator.java
│   │   │       └── FormatValidator.java
│   │   └── resources/
│   │       ├── application.yml
│   │       └── db/migration/
│   └── test/
├── docker/
├── k8s/
├── helm/
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
│   │   │   ├── config/
│   │   │   │   ├── DatabaseConfig.java
│   │   │   │   ├── KafkaConfig.java
│   │   │   │   ├── RedisConfig.java
│   │   │   │   └── WebSocketConfig.java
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
│   │   │   ├── websocket/
│   │   │   │   ├── SubmissionStatusHandler.java
│   │   │   │   └── WebSocketEventPublisher.java
│   │   │   ├── grpc/
│   │   │   │   └── SubmissionGrpcService.java
│   │   │   └── validator/
│   │   │       ├── CodeValidator.java
│   │   │       ├── LanguageValidator.java
│   │   │       └── SubmissionValidator.java
│   │   └── resources/
│   │       ├── application.yml
│   │       └── db/migration/
│   └── test/
├── docker/
├── k8s/
├── helm/
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
│   │   │   ├── config/
│   │   │   │   ├── DockerConfig.java
│   │   │   │   ├── KafkaConfig.java
│   │   │   │   ├── RedisConfig.java
│   │   │   │   └── ResourceConfig.java
│   │   │   ├── controller/
│   │   │   │   ├── BuildController.java
│   │   │   │   ├── BuildHistoryController.java
│   │   │   │   └── BuildStatusController.java
│   │   │   ├── service/
│   │   │   │   ├── BuildService.java
│   │   │   │   ├── CompilerService.java
│   │   │   │   ├── DockerService.java
│   │   │   │   ├── CacheService.java
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
│   │   │   ├── compiler/
│   │   │   │   ├── JavaCompiler.java
│   │   │   │   ├── PythonCompiler.java
│   │   │   │   ├── CppCompiler.java
│   │   │   │   ├── JavaScriptRunner.java
│   │   │   │   └── CompilerFactory.java
│   │   │   ├── docker/
│   │   │   │   ├── DockerManager.java
│   │   │   │   ├── ContainerBuilder.java
│   │   │   │   └── ImageManager.java
│   │   │   └── monitoring/
│   │   │       ├── ResourceMonitor.java
│   │   │       └── BuildMetrics.java
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── docker-templates/
│   │       │   ├── java.dockerfile
│   │       │   ├── python.dockerfile
│   │       │   ├── cpp.dockerfile
│   │       │   └── javascript.dockerfile
│   │       └── compiler-configs/
│   └── test/
├── docker/
├── k8s/
├── helm/
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
│   │   │   ├── config/
│   │   │   │   ├── SandboxConfig.java
│   │   │   │   ├── KafkaConfig.java
│   │   │   │   ├── RedisConfig.java
│   │   │   │   └── SecurityConfig.java
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
│   │   │   │   ├── SecurityPolicy.java
│   │   │   │   └── ResourceLimiter.java
│   │   │   ├── executor/
│   │   │   │   ├── CodeExecutor.java
│   │   │   │   ├── TestCaseRunner.java
│   │   │   │   └── ResultAnalyzer.java
│   │   │   └── monitoring/
│   │   │       ├── ResourceMonitor.java
│   │   │       ├── PerformanceTracker.java
│   │   │       └── ExecutionMetrics.java
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── sandbox-policies/
│   │       │   ├── java-policy.yml
│   │       │   ├── python-policy.yml
│   │       │   └── cpp-policy.yml
│   │       └── resource-limits/
│   └── test/
├── docker/
├── k8s/
├── helm/
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
│   │   │   ├── config/
│   │   │   │   ├── DatabaseConfig.java
│   │   │   │   ├── RedisConfig.java
│   │   │   │   ├── WebSocketConfig.java
│   │   │   │   └── SchedulingConfig.java
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
│   │   │   ├── scheduling/
│   │   │   │   ├── ContestScheduler.java
│   │   │   │   ├── ContestStartTask.java
│   │   │   │   └── ContestEndTask.java
│   │   │   └── scoring/
│   │   │       ├── ScoringAlgorithm.java
│   │   │       ├── ICPCScoring.java
│   │   │       └── CodeforceScoring.java
│   │   └── resources/
│   │       ├── application.yml
│   │       └── db/migration/
│   └── test/
├── docker/
├── k8s/
├── helm/
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
│   │   │   ├── config/
│   │   │   │   ├── EmailConfig.java
│   │   │   │   ├── SmsConfig.java
│   │   │   │   ├── RabbitMQConfig.java
│   │   │   │   ├── WebSocketConfig.java
│   │   │   │   └── TemplateConfig.java
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
│   │   │   ├── template/
│   │   │   │   ├── TemplateEngine.java
│   │   │   │   ├── TemplateProcessor.java
│   │   │   │   └── TemplateValidator.java
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
├── docker/
├── k8s/
├── helm/
├── pom.xml
└── README.md
```

---

## 📋 **SHARED-CONTRACTS** (Repository: shared-contracts)

```
shared-contracts/
├── proto/
│   ├── auth/
│   │   ├── auth.proto
│   │   └── token.proto
│   ├── user/
│   │   ├── user.proto
│   │   └── profile.proto
│   ├── problem/
│   │   ├── problem.proto
│   │   └── search.proto
│   ├── testcase/
│   │   └── testcase.proto
│   ├── submission/
│   │   └── submission.proto
│   ├── build/
│   │   └── build.proto
│   ├── execution/
│   │   └── execution.proto
│   ├── contest/
│   │   └── contest.proto
│   └── notification/
│       └── notification.proto
├── java/
│   └── generated/
├── common/
│   ├── error-codes.proto
│   ├── pagination.proto
│   └── metadata.proto
├── scripts/
│   ├── generate-java.sh
│   ├── generate-python.sh
│   └── build.gradle
├── README.md
└── build.gradle
```

---

## 🏗️ **PLATFORM-INFRASTRUCTURE** (Repository: platform-infrastructure)

```
platform-infrastructure/
├── docker/
│   ├── local-development/
│   │   ├── docker-compose.yml              # Complete local stack
│   │   ├── docker-compose.override.yml     # Development overrides
│   │   ├── docker-compose.monitoring.yml   # Monitoring stack
│   │   ├── .env.example                    # Environment template
│   │   └── init-scripts/
│   │       ├── postgres-init.sql
│   │       ├── mongodb-init.js
│   │       ├── kafka-topics.sh
│   │       └── elasticsearch-indices.sh
│   ├── production/
│   │   └── docker-compose.prod.yml
│   └── infrastructure/
│       ├── api-gateway/
│       │   ├── kong/
│       │   │   ├── Dockerfile
│       │   │   ├── kong.yml
│       │   │   └── plugins/
│       │   └── nginx/
│       │       ├── Dockerfile
│       │       ├── nginx.conf
│       │       └── conf.d/
│       ├── databases/
│       │   ├── postgres/
│       │   │   ├── Dockerfile
│       │   │   ├── init.sql
│       │   │   └── postgresql.conf
│       │   ├── mongodb/
│       │   │   ├── Dockerfile
│       │   │   ├── init.js
│       │   │   └── mongod.conf
│       │   ├── redis/
│       │   │   ├── Dockerfile
│       │   │   ├── redis.conf
│       │   │   └── sentinel.conf
│       │   └── elasticsearch/
│       │       ├── Dockerfile
│       │       ├── elasticsearch.yml
│       │       └── jvm.options
│       ├── messaging/
│       │   ├── kafka/
│       │   │   ├── Dockerfile
│       │   │   ├── server.properties
│       │   │   └── topics/
│       │   ├── zookeeper/
│       │   │   ├── Dockerfile
│       │   │   └── zoo.cfg
│       │   └── rabbitmq/
│       │       ├── Dockerfile
│       │       ├── rabbitmq.conf
│       │       └── definitions.json
│       └── reverse-proxy/
│           ├── haproxy/
│           │   ├── Dockerfile
│           │   └── haproxy.cfg
│           └── nginx/
│               ├── Dockerfile
│               └── nginx.conf
├── kubernetes/
│   ├── local/                             # Local K8s cluster configs
│   │   ├── namespaces/
│   │   │   ├── development.yaml
│   │   │   ├── monitoring.yaml
│   │   │   └── infrastructure.yaml
│   │   ├── storage/
│   │   │   ├── postgres-pv.yaml
│   │   │   ├── mongodb-pv.yaml
│   │   │   ├── redis-pv.yaml
│   │   │   └── elasticsearch-pv.yaml
│   │   ├── infrastructure/
│   │   │   ├── postgres-deployment.yaml
│   │   │   ├── mongodb-deployment.yaml
│   │   │   ├── redis-deployment.yaml
│   │   │   ├── elasticsearch-deployment.yaml
│   │   │   ├── kafka-deployment.yaml
│   │   │   ├── zookeeper-deployment.yaml
│   │   │   ├── rabbitmq-deployment.yaml
│   │   │   ├── kong-deployment.yaml
│   │   │   └── haproxy-deployment.yaml
│   │   ├── services/
│   │   │   ├── auth-service.yaml
│   │   │   ├── user-service.yaml
│   │   │   ├── problem-service.yaml
│   │   │   ├── testcase-service.yaml
│   │   │   ├── submission-service.yaml
│   │   │   ├── build-service.yaml
│   │   │   ├── execution-service.yaml
│   │   │   ├── contest-service.yaml
│   │   │   └── notification-service.yaml
│   │   ├── configmaps/
│   │   │   ├── database-configs.yaml
│   │   │   ├── messaging-configs.yaml
│   │   │   ├── api-gateway-configs.yaml
│   │   │   └── monitoring-configs.yaml
│   │   ├── secrets/
│   │   │   ├── database-secrets.yaml
│   │   │   ├── jwt-secrets.yaml
│   │   │   └── messaging-secrets.yaml
│   │   ├── ingress/
│   │   │   ├── api-ingress.yaml
│   │   │   ├── monitoring-ingress.yaml
│   │   │   └── admin-ingress.yaml
│   │   └── monitoring/
│   │       ├── prometheus-deployment.yaml
│   │       ├── grafana-deployment.yaml
│   │       ├── jaeger-deployment.yaml
│   │       └── alertmanager-deployment.yaml
│   └── gcp/                               # GCP GKE configs
│       ├── gke-cluster.yaml
│       ├── node-pools.yaml
│       ├── workload-identity.yaml
│       └── cloud-sql-proxy.yaml
├── helm/
│   ├── infrastructure/                    # Infrastructure Helm charts
│   │   ├── api-gateway/
│   │   │   ├── kong/
│   │   │   │   ├── Chart.yaml
│   │   │   │   ├── values.yaml
│   │   │   │   ├── values-local.yaml
│   │   │   │   ├── values-gcp.yaml
│   │   │   │   └── templates/
│   │   │   └── nginx/
│   │   │       ├── Chart.yaml
│   │   │       ├── values.yaml
│   │   │       └── templates/
│   │   ├── databases/
│   │   │   ├── postgresql/
│   │   │   │   ├── Chart.yaml
│   │   │   │   ├── values-local.yaml
│   │   │   │   ├── values-gcp.yaml
│   │   │   │   └── templates/
│   │   │   ├── mongodb/
│   │   │   ├── redis/
│   │   │   └── elasticsearch/
│   │   ├── messaging/
│   │   │   ├── kafka/
│   │   │   │   ├── Chart.yaml
│   │   │   │   ├── values-local.yaml
│   │   │   │   ├── values-gcp.yaml
│   │   │   │   └── templates/
│   │   │   └── rabbitmq/
│   │   │       ├── Chart.yaml
│   │   │       ├── values.yaml
│   │   │       └── templates/
│   │   └── reverse-proxy/
│   │       ├── haproxy/
│   │       └── nginx/
│   ├── monitoring/                        # Monitoring stack
│   │   ├── prometheus/
│   │   │   ├── Chart.yaml
│   │   │   ├── values-local.yaml
│   │   │   ├── values-gcp.yaml
│   │   │   └── templates/
│   │   ├── grafana/
│   │   │   ├── Chart.yaml
│   │   │   ├── values-local.yaml
│   │   │   ├── values-gcp.yaml
│   │   │   └── templates/
│   │   │       ├── dashboards/
│   │   │       └── datasources/
│   │   ├── jaeger/
│   │   └── alertmanager/
│   └── environments/                      # Environment umbrella charts
│       ├── local-development/
│       │   ├── Chart.yaml
│       │   ├── values.yaml
│       │   ├── requirements.yaml
│       │   └── templates/
│       ├── local-k8s/
│       │   ├── Chart.yaml
│       │   ├── values.yaml
│       │   └── requirements.yaml
│       ├── gcp-dev/
│       └── gcp-prod/
├── terraform/
│   ├── local/
│   │   ├── k8s-cluster/
│   │   │   ├── main.tf
│   │   │   ├── variables.tf
│   │   │   └── outputs.tf
│   │   └── docker-registry/
│   ├── gcp/
│   │   ├── gke/
│   │   │   ├── main.tf
│   │   │   ├── variables.tf
│   │   │   ├── versions.tf
│   │   │   └── outputs.tf
│   │   ├── databases/
│   │   │   ├── cloud-sql.tf
│   │   │   ├── memorystore.tf
│   │   │   └── firestore.tf
│   │   ├── networking/
│   │   │   ├── vpc.tf
│   │   │   ├── subnets.tf
│   │   │   ├── firewall.tf
│   │   │   └── load-balancer.tf
│   │   ├── storage/
│   │   │   ├── gcs.tf
│   │   │   └── persistent-disks.tf
│   │   └── messaging/
│   │       ├── pub-sub.tf
│   │       └── cloud-tasks.tf
│   └── modules/
│       ├── gke-cluster/
│       ├── cloud-sql/
│       ├── vpc-network/
│       └── monitoring/
├── ci-cd/
│   ├── local/                            # Local CI/CD setup
│   │   ├── jenkins/
│   │   │   ├── Dockerfile
│   │   │   ├── jenkins.yaml
│   │   │   ├── plugins.txt
│   │   │   └── jobs/
│   │   │       ├── build-auth-service.xml
│   │   │       ├── build-user-service.xml
│   │   │       ├── build-problem-service.xml
│   │   │       ├── build-testcase-service.xml
│   │   │       ├── build-submission-service.xml
│   │   │       ├── build-build-service.xml
│   │   │       ├── build-execution-service.xml
│   │   │       ├── build-contest-service.xml
│   │   │       ├── build-notification-service.xml
│   │   │       └── deploy-to-k8s.xml
│   │   ├── gitlab-runner/
│   │   │   ├── docker-compose.yml
│   │   │   ├── config.toml
│   │   │   └── gitlab-ci-templates/
│   │   └── tekton/
│   │       ├── pipelines/
│   │       ├── tasks/
│   │       ├── triggers/
│   │       └── resources/
│   ├── github-actions/
│   │   ├── workflows/
│   │   │   ├── build-and-test.yml
│   │   │   ├── deploy-local.yml
│   │   │   └── deploy-gcp.yml
│   │   └── composite-actions/
│   └── templates/
│       ├── Jenkinsfile.template
│       ├── gitlab-ci.template.yml
│       └── github-workflow.template.yml
├── scripts/
│   ├── local-setup/
│   │   ├── setup-docker.ps1              # PowerShell scripts for Windows
│   │   ├── setup-k8s.ps1                 # Kind/Minikube setup
│   │   ├── setup-registry.ps1            # Local Docker registry
│   │   ├── setup-jenkins.ps1             # Jenkins CI/CD setup
│   │   ├── build-all-services.ps1        # Build all microservices
│   │   ├── deploy-infrastructure.ps1     # Deploy infrastructure
│   │   ├── deploy-services.ps1           # Deploy microservices
│   │   └── setup-monitoring.ps1          # Setup monitoring stack
│   ├── gcp-setup/
│   │   ├── setup-gke.ps1
│   │   ├── setup-databases.ps1
│   │   └── migrate-to-gcp.ps1
│   ├── development/
│   │   ├── start-local-env.ps1
│   │   ├── stop-local-env.ps1
│   │   ├── reset-databases.ps1
│   │   └── run-tests.ps1
│   └── utilities/
│       ├── health-check.ps1
│       ├── logs-collector.ps1
│       ├── backup-data.ps1
│       └── performance-test.ps1
├── monitoring/
│   ├── prometheus/
│   │   ├── prometheus.yml
│   │   ├── alert-rules/
│   │   │   ├── infrastructure.yml
│   │   │   ├── applications.yml
│   │   │   └── performance.yml
│   │   └── exporters/
│   ├── grafana/
│   │   ├── provisioning/
│   │   │   ├── dashboards/
│   │   │   └── datasources/
│   │   ├── dashboards/
│   │   │   ├── infrastructure-dashboard.json
│   │   │   ├── application-dashboard.json
│   │   │   ├── kafka-dashboard.json
│   │   │   ├── database-dashboard.json
│   │   │   └── grpc-dashboard.json
│   │   └── plugins/
│   ├── jaeger/
│   │   ├── jaeger-config.yml
│   │   └── sampling-strategies.json
│   └── alerting/
│       ├── alertmanager.yml
│       ├── notification-templates/
│       └── webhook-configs/
├── configs/
│   ├── local/
│   │   ├── application.yml              # Common config for local
│   │   ├── database-urls.yml
│   │   ├── messaging-endpoints.yml
│   │   └── api-gateway-routes.yml
│   ├── gcp/
│   │   ├── application-gcp.yml
│   │   ├── cloud-endpoints.yml
│   │   └── workload-identity.yml
│   └── shared/
│       ├── grpc-ports.yml
│       ├── health-check-endpoints.yml
│       └── service-discovery.yml
├── documentation/
│   ├── LOCAL_SETUP.md                   # Complete local setup guide
│   ├── K8S_DEPLOYMENT.md               # Kubernetes deployment guide
│   ├── CI_CD_SETUP.md                  # CI/CD pipeline setup
│   ├── MONITORING.md                   # Monitoring setup and usage
│   ├── TROUBLESHOOTING.md              # Common issues and solutions
│   ├── API_GATEWAY.md                  # API Gateway configuration
│   ├── DATABASE_SETUP.md               # Database configuration
│   ├── MESSAGING.md                    # Kafka/RabbitMQ setup
│   └── GCP_MIGRATION.md                # Migration guide to GCP
├── tests/
│   ├── integration/
│   │   ├── docker-compose.test.yml
│   │   ├── k8s-integration-tests/
│   │   └── api-gateway-tests/
│   ├── performance/
│   │   ├── load-test-configs/
│   │   ├── stress-test-scenarios/
│   │   └── benchmark-scripts/
│   └── e2e/
│       ├── playwright-tests/
│       ├── postman-collections/
│       └── grpc-tests/
├── .gitignore
├── README.md
├── ARCHITECTURE.md
└── CONTRIBUTING.md
```

This structure provides complete separation of concerns with each microservice in its own repository, while maintaining shared contracts and infrastructure components.
