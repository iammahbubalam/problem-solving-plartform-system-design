# 🚀 COMPLETE LOCAL-TO-K8S EXECUTION PLAN

## 🎯 **OVERVIEW: LOCAL DEVELOPMENT TO KUBERNETES DEPLOYMENT**

This plan will set up a complete competitive programming platform with:
- **API Gateway** (Kong) + **Reverse Proxy** (HAProxy/Nginx)
- **Messaging** (Kafka + RabbitMQ) + **gRPC** inter-service communication
- **Databases** (PostgreSQL + MongoDB + Redis + Elasticsearch)
- **Local CI/CD Pipeline** (Jenkins/GitLab Runner/Tekton)
- **Kubernetes Deployment** (Local Kind/Minikube → GCP migration ready)

---

## 📅 **WEEK 1: COMPLETE INFRASTRUCTURE FOUNDATION**

### **Day 1: Local Development Environment Setup**

#### **Step 1: Install Required Tools (PowerShell)**
```powershell
# Install Chocolatey (if not installed)
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# Install development tools
choco install openjdk21 -y
choco install maven -y
choco install docker-desktop -y
choco install git -y
choco install vscode -y
choco install kubectl -y
choco install helm -y
choco install kind -y                    # For local Kubernetes
choco install terraform -y
choco install nodejs -y                  # For additional tooling
choco install postman -y                 # For API testing

# Verify installations
java -version
mvn -version
docker --version
kubectl version --client
helm version
kind version
```

#### **Step 2: Create Local Kubernetes Cluster**
```powershell
# Create Kind cluster for local development
kind create cluster --name competitive-platform --config=kind-config.yaml

# Create kind-config.yaml
@"
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
- role: worker
- role: worker
"@ | Out-File -FilePath kind-config.yaml -Encoding UTF8

# Verify cluster
kubectl cluster-info
kubectl get nodes
```

#### **Step 3: Setup Local Docker Registry**
```powershell
# Create local Docker registry for building images
docker run -d -p 5000:5000 --name registry registry:2

# Configure Docker to use local registry
# Add to Docker Desktop settings: insecure-registries: ["localhost:5000"]
```

### **Day 2-3: Repository and Infrastructure Setup**

#### **Step 1: Create and Clone All Repositories**
```powershell
# Create main workspace
mkdir d:\competitive-platform
cd d:\competitive-platform

# Clone repositories (create these on GitHub/GitLab first)
$repos = @(
    "shared-contracts",
    "platform-infrastructure", 
    "auth-service",
    "user-service",
    "problem-service",
    "testcase-service",
    "submission-service",
    "build-service",
    "execution-service",
    "contest-service",
    "notification-service"
)

foreach ($repo in $repos) {
    git clone https://github.com/yourusername/$repo.git
}
```

#### **Step 2: Setup Complete Infrastructure Stack**
```powershell
cd platform-infrastructure

# Create comprehensive docker-compose.yml
cat > docker/local-development/docker-compose.yml << 'EOF'
version: '3.8'

services:
  # === API GATEWAY ===
  kong:
    image: kong:3.4
    environment:
      KONG_DATABASE: "off"
      KONG_DECLARATIVE_CONFIG: /kong/declarative/kong.yml
      KONG_PROXY_ACCESS_LOG: /dev/stdout
      KONG_ADMIN_ACCESS_LOG: /dev/stdout
      KONG_PROXY_ERROR_LOG: /dev/stderr
      KONG_ADMIN_ERROR_LOG: /dev/stderr
      KONG_ADMIN_LISTEN: 0.0.0.0:8001
      KONG_PROXY_LISTEN: 0.0.0.0:8000
    ports:
      - "8000:8000"   # Proxy port
      - "8001:8001"   # Admin API
    volumes:
      - ./infrastructure/api-gateway/kong/kong.yml:/kong/declarative/kong.yml
    networks:
      - platform-network

  # === REVERSE PROXY ===
  haproxy:
    image: haproxy:2.8
    ports:
      - "80:80"
      - "443:443"
      - "8404:8404"   # Stats
    volumes:
      - ./infrastructure/reverse-proxy/haproxy/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg
    depends_on:
      - kong
    networks:
      - platform-network

  # === DATABASES ===
  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: competitive_platform
      POSTGRES_USER: platform_user
      POSTGRES_PASSWORD: platform_pass
      POSTGRES_MULTIPLE_DATABASES: auth_db,user_db,problem_db,testcase_db,submission_db,contest_db
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./infrastructure/databases/postgres/init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - platform-network

  mongodb:
    image: mongo:7
    environment:
      MONGO_INITDB_ROOT_USERNAME: mongo_user
      MONGO_INITDB_ROOT_PASSWORD: mongo_pass
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db
      - ./infrastructure/databases/mongodb/init.js:/docker-entrypoint-initdb.d/init.js
    networks:
      - platform-network

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes --replica-read-only no
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - platform-network

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.10.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    networks:
      - platform-network

  # === MESSAGING ===
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"
    volumes:
      - zookeeper_data:/var/lib/zookeeper/data
    networks:
      - platform-network

  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092,PLAINTEXT_INTERNAL://kafka:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_INTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT_INTERNAL
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: true
    ports:
      - "9092:9092"
    volumes:
      - kafka_data:/var/lib/kafka/data
    networks:
      - platform-network

  kafka-ui:
    image: provectuslabs/kafka-ui:latest
    depends_on:
      - kafka
    environment:
      KAFKA_CLUSTERS_0_NAME: local
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:29092
    ports:
      - "8080:8080"
    networks:
      - platform-network

  rabbitmq:
    image: rabbitmq:3.12-management
    environment:
      RABBITMQ_DEFAULT_USER: rabbit_user
      RABBITMQ_DEFAULT_PASS: rabbit_pass
    ports:
      - "5672:5672"   # AMQP port
      - "15672:15672" # Management UI
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
      - ./infrastructure/messaging/rabbitmq/definitions.json:/etc/rabbitmq/definitions.json
    networks:
      - platform-network

  # === MONITORING ===
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
    networks:
      - platform-network

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: admin
    volumes:
      - grafana_data:/var/lib/grafana
      - ./monitoring/grafana/provisioning:/etc/grafana/provisioning
    networks:
      - platform-network

  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "16686:16686"  # UI
      - "14268:14268"  # HTTP collector
    environment:
      COLLECTOR_ZIPKIN_HOST_PORT: :9411
    networks:
      - platform-network

  # === CI/CD ===
  jenkins:
    image: jenkins/jenkins:lts
    ports:
      - "8181:8080"
      - "50000:50000"
    volumes:
      - jenkins_data:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
      - ./ci-cd/local/jenkins:/var/jenkins_home/init.groovy.d
    environment:
      JAVA_OPTS: "-Djenkins.install.runSetupWizard=false"
    networks:
      - platform-network

  # === SERVICE DISCOVERY (Optional - for gRPC) ===
  consul:
    image: consul:latest
    ports:
      - "8500:8500"
    command: "consul agent -dev -client=0.0.0.0 -ui"
    networks:
      - platform-network

volumes:
  postgres_data:
  mongodb_data:
  redis_data:
  elasticsearch_data:
  zookeeper_data:
  kafka_data:
  rabbitmq_data:
  prometheus_data:
  grafana_data:
  jenkins_data:

networks:
  platform-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
EOF
```

#### **Step 3: Create Configuration Files**
```powershell
# Kong API Gateway configuration
mkdir -p infrastructure/api-gateway/kong
cat > infrastructure/api-gateway/kong/kong.yml << 'EOF'
_format_version: "3.0"

services:
  - name: auth-service
    url: http://auth-service:8081
    routes:
      - name: auth-routes
        paths:
          - /api/v1/auth
        strip_path: true

  - name: user-service
    url: http://user-service:8082
    routes:
      - name: user-routes
        paths:
          - /api/v1/users
        strip_path: true

  - name: problem-service
    url: http://problem-service:8083
    routes:
      - name: problem-routes
        paths:
          - /api/v1/problems
        strip_path: true

  - name: submission-service
    url: http://submission-service:8085
    routes:
      - name: submission-routes
        paths:
          - /api/v1/submissions
        strip_path: true

plugins:
  - name: cors
    config:
      origins:
        - "*"
      methods:
        - "GET"
        - "POST"
        - "PUT"
        - "DELETE"
        - "OPTIONS"
      headers:
        - "Accept"
        - "Authorization"
        - "Content-Type"
        - "X-Requested-With"

  - name: rate-limiting
    config:
      minute: 100
      hour: 1000
EOF

# HAProxy configuration
mkdir -p infrastructure/reverse-proxy/haproxy
cat > infrastructure/reverse-proxy/haproxy/haproxy.cfg << 'EOF'
global
    daemon
    log stdout local0

defaults
    mode http
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms
    log global
    option httplog

frontend main
    bind *:80
    # Route to Kong API Gateway
    default_backend kong_api_gateway

backend kong_api_gateway
    balance roundrobin
    server kong1 kong:8000 check

# Stats page
listen stats
    bind *:8404
    stats enable
    stats uri /
    stats refresh 10s
    stats admin if TRUE
EOF
```

### **Day 4-5: Local CI/CD Pipeline Setup**

#### **Step 1: Setup Jenkins Pipeline**
```powershell
# Create Jenkins job configurations
mkdir -p ci-cd/local/jenkins/jobs

# Master build pipeline for all services
cat > ci-cd/local/jenkins/jobs/build-all-services.xml << 'EOF'
<?xml version='1.1' encoding='UTF-8'?>
<flow-definition plugin="workflow-job">
  <description>Build and deploy all microservices</description>
  <definition class="org.jenkinsci.plugins.workflow.cps.CpsFlowDefinition" plugin="workflow-cps">
    <script>
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'localhost:5000'
        KUBECONFIG = '/var/jenkins_home/.kube/config'
    }
    
    stages {
        stage('Checkout All Services') {
            parallel {
                stage('Auth Service') {
                    steps {
                        dir('auth-service') {
                            git branch: 'main', url: 'https://github.com/yourusername/auth-service.git'
                        }
                    }
                }
                stage('User Service') {
                    steps {
                        dir('user-service') {
                            git branch: 'main', url: 'https://github.com/yourusername/user-service.git'
                        }
                    }
                }
                // Add other services...
            }
        }
        
        stage('Build Shared Contracts') {
            steps {
                dir('shared-contracts') {
                    git branch: 'main', url: 'https://github.com/yourusername/shared-contracts.git'
                    sh 'gradle build publishToMavenLocal'
                }
            }
        }
        
        stage('Build and Test Services') {
            parallel {
                stage('Build Auth Service') {
                    steps {
                        dir('auth-service') {
                            sh 'mvn clean test'
                            sh 'mvn package -DskipTests'
                            sh 'docker build -t $DOCKER_REGISTRY/auth-service:latest .'
                            sh 'docker push $DOCKER_REGISTRY/auth-service:latest'
                        }
                    }
                }
                // Add other services...
            }
        }
        
        stage('Deploy to Local K8s') {
            steps {
                script {
                    sh 'helm upgrade --install platform-infrastructure ./platform-infrastructure/helm/environments/local-k8s --namespace infrastructure --create-namespace'
                    sh 'sleep 30' // Wait for infrastructure
                    sh 'helm upgrade --install auth-service ./auth-service/helm --namespace development --create-namespace'
                    // Deploy other services...
                }
            }
        }
        
        stage('Run Integration Tests') {
            steps {
                script {
                    sh 'kubectl wait --for=condition=ready pod -l app=auth-service --timeout=300s -n development'
                    // Run API tests, health checks, etc.
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
    </script>
  </definition>
</flow-definition>
EOF

# Create PowerShell script to setup Jenkins
cat > scripts/local-setup/setup-jenkins.ps1 << 'EOF'
# Setup Jenkins with required plugins
Write-Host "Setting up Jenkins..." -ForegroundColor Green

# Wait for Jenkins to be ready
do {
    Start-Sleep -Seconds 10
    $response = try { Invoke-WebRequest -Uri "http://localhost:8181" -UseBasicParsing } catch { $null }
} while ($null -eq $response)

# Install required plugins
$plugins = @(
    "workflow-aggregator",
    "docker-workflow", 
    "kubernetes",
    "kubernetes-cli",
    "pipeline-stage-view",
    "blueocean",
    "github",
    "maven-plugin",
    "gradle"
)

foreach ($plugin in $plugins) {
    Write-Host "Installing plugin: $plugin" -ForegroundColor Yellow
    # Jenkins CLI commands to install plugins
}

Write-Host "Jenkins setup complete!" -ForegroundColor Green
EOF
```

#### **Step 2: Create Complete Setup Scripts**
```powershell
# Master setup script
cat > scripts/local-setup/setup-complete-environment.ps1 << 'EOF'
param(
    [switch]$SkipInfrastructure,
    [switch]$SkipServices,
    [switch]$SkipK8s
)

Write-Host "🚀 Setting up Complete Competitive Programming Platform" -ForegroundColor Cyan
Write-Host "============================================================" -ForegroundColor Cyan

# Step 1: Start Infrastructure
if (-not $SkipInfrastructure) {
    Write-Host "📦 Starting Infrastructure Stack..." -ForegroundColor Green
    cd docker/local-development
    docker-compose up -d
    
    Write-Host "⏳ Waiting for infrastructure to be ready..." -ForegroundColor Yellow
    Start-Sleep -Seconds 60
    
    # Verify infrastructure
    $services = @(
        @{Name="PostgreSQL"; Port=5432},
        @{Name="MongoDB"; Port=27017},
        @{Name="Redis"; Port=6379},
        @{Name="Kafka"; Port=9092},
        @{Name="RabbitMQ"; Port=5672},
        @{Name="Elasticsearch"; Port=9200},
        @{Name="Kong"; Port=8000},
        @{Name="Jenkins"; Port=8181}
    )
    
    foreach ($service in $services) {
        $connection = Test-NetConnection -ComputerName localhost -Port $service.Port -WarningAction SilentlyContinue
        if ($connection.TcpTestSucceeded) {
            Write-Host "✅ $($service.Name) is running" -ForegroundColor Green
        } else {
            Write-Host "❌ $($service.Name) failed to start" -ForegroundColor Red
        }
    }
}

# Step 2: Setup Kubernetes
if (-not $SkipK8s) {
    Write-Host "🎯 Setting up Kubernetes environment..." -ForegroundColor Green
    
    # Create namespaces
    kubectl create namespace infrastructure --dry-run=client -o yaml | kubectl apply -f -
    kubectl create namespace development --dry-run=client -o yaml | kubectl apply -f -
    kubectl create namespace monitoring --dry-run=client -o yaml | kubectl apply -f -
    
    # Deploy infrastructure to K8s
    helm upgrade --install platform-infrastructure ./helm/environments/local-k8s --namespace infrastructure --create-namespace
    
    Write-Host "⏳ Waiting for infrastructure pods..." -ForegroundColor Yellow
    kubectl wait --for=condition=ready pod -l component=infrastructure --timeout=300s -n infrastructure
}

# Step 3: Build and Deploy Services
if (-not $SkipServices) {
    Write-Host "🔨 Building and deploying services..." -ForegroundColor Green
    
    # Build shared contracts first
    cd ../../shared-contracts
    gradle build publishToMavenLocal
    
    # Build services
    $services = @("auth-service", "user-service", "problem-service", "testcase-service", "submission-service", "build-service", "execution-service", "contest-service", "notification-service")
    
    foreach ($service in $services) {
        Write-Host "Building $service..." -ForegroundColor Yellow
        cd ../$service
        mvn clean package -DskipTests
        docker build -t localhost:5000/$service:latest .
        docker push localhost:5000/$service:latest
        
        # Deploy to K8s
        helm upgrade --install $service ./helm --namespace development --create-namespace
    }
}

Write-Host "🎉 Setup Complete!" -ForegroundColor Green
Write-Host "Access points:" -ForegroundColor Cyan
Write-Host "- API Gateway: http://localhost:8000" -ForegroundColor White
Write-Host "- Jenkins: http://localhost:8181" -ForegroundColor White
Write-Host "- Grafana: http://localhost:3000 (admin/admin)" -ForegroundColor White
Write-Host "- Kafka UI: http://localhost:8080" -ForegroundColor White
Write-Host "- RabbitMQ: http://localhost:15672 (rabbit_user/rabbit_pass)" -ForegroundColor White
EOF
```

---

## 🎯 **WEEK 2-4: SERVICE IMPLEMENTATION WITH INFRASTRUCTURE**

### **Implementation Order with Infrastructure Integration**

#### **Phase 1: Foundation Services (Week 2)**
1. **shared-contracts** - gRPC definitions, common DTOs
2. **auth-service** - JWT + OAuth2 + Redis sessions + PostgreSQL
3. **user-service** - User management + MongoDB + Elasticsearch search

#### **Phase 2: Core Platform (Week 3)**  
4. **problem-service** - Problem CRUD + Elasticsearch + PostgreSQL
5. **testcase-service** - Test cases + PostgreSQL + encryption
6. **submission-service** - Code submission + Kafka producer + PostgreSQL

#### **Phase 3: Execution Pipeline (Week 4)**
7. **build-service** - Docker builds + Kafka consumer/producer
8. **execution-service** - Sandboxed execution + result processing
9. **contest-service** - Contests + WebSocket + Redis leaderboard  
10. **notification-service** - Multi-channel + RabbitMQ + templates

---

## 🚀 **IMMEDIATE NEXT STEPS (Execute Now)**

### **Step 1: Create Repository Structure**
```powershell
# Execute this immediately
cd d:\VsCodeProject\leetcode
mkdir competitive-platform
cd competitive-platform

# Create all repositories on GitHub/GitLab, then clone them
# (Use the repository list from above)
```

### **Step 2: Start Infrastructure**
```powershell
# Navigate to platform-infrastructure and setup
cd platform-infrastructure
# Create docker-compose.yml as shown above
docker-compose -f docker/local-development/docker-compose.yml up -d
```

### **Step 3: Setup Kubernetes**
```powershell
# Create local Kind cluster
kind create cluster --name competitive-platform
kubectl cluster-info
```

### **Step 4: Run Setup Script**
```powershell
# Run the complete setup
.\scripts\local-setup\setup-complete-environment.ps1
```

### **Success Criteria for Week 1:**
- [ ] ✅ All infrastructure services running (Kong, HAProxy, PostgreSQL, MongoDB, Redis, Kafka, RabbitMQ, Elasticsearch)
- [ ] ✅ Local Kubernetes cluster operational  
- [ ] ✅ Jenkins CI/CD pipeline functional
- [ ] ✅ Local Docker registry working
- [ ] ✅ Monitoring stack operational (Prometheus, Grafana, Jaeger)
- [ ] ✅ All repositories cloned and initial structure created
- [ ] ✅ shared-contracts building and publishing locally
- [ ] ✅ auth-service integrated with PostgreSQL and Redis

This comprehensive setup gives you a production-ready local environment that you can develop against and easily migrate to GCP when ready.
choco install openjdk21 -y
choco install maven -y
choco install docker-desktop -y
choco install git -y
choco install vscode -y
choco install intellijidea-community -y
```

#### **Docker Setup for Local Development**
```powershell
# Start Docker Desktop
# Create docker network for all services
docker network create competitive-platform-network
```

### **Day 4-5: Infrastructure Foundation**

#### **Step 1: Setup shared-contracts first**
```bash
cd shared-contracts
# Initialize with basic structure
```

#### **Step 2: Setup platform-infrastructure**
```bash
cd platform-infrastructure
# Create Docker Compose for local development
```

#### **Step 3: Setup auth-service (Critical Path)**
```bash
cd auth-service
# Initialize Spring Boot project
```

---

## 🎯 **IMPLEMENTATION PRIORITY ORDER**

### **Phase 1: Core Foundation (Start Immediately)**

#### **1. shared-contracts (Day 1)**
- Create gRPC proto files
- Define common DTOs
- Setup build scripts
- **Why First**: All services depend on these contracts

#### **2. platform-infrastructure (Day 2)**
- Docker Compose for local development
- Database initialization scripts
- Basic Kubernetes manifests
- **Why Second**: Provides runtime environment for all services

#### **3. auth-service (Day 3-5)**
- JWT authentication
- Basic user model
- gRPC server for token validation
- **Why Third**: All other services need authentication

### **Phase 2: Core Services (Week 2)**

#### **4. user-service (Day 6-8)**
- User management
- Profile handling
- Integration with auth-service
- **Why Fourth**: Many services need user data

#### **5. problem-service (Day 9-10)**
- Problem CRUD operations
- Basic search functionality
- **Why Fifth**: Foundation for submission system

#### **6. testcase-service (Day 11-12)**
- Test case storage
- Basic validation
- Integration with problem-service
- **Why Sixth**: Needed for code execution

### **Phase 3: Execution Pipeline (Week 3)**

#### **7. submission-service (Day 13-15)**
- Code submission handling
- Queue management
- Status tracking
- **Why Seventh**: Entry point for code processing

#### **8. build-service (Day 16-17)**
- Code compilation
- Docker-based builds
- **Why Eighth**: Processes submissions

#### **9. execution-service (Day 18-19)**
- Code execution
- Result comparison
- **Why Ninth**: Final step in submission pipeline

### **Phase 4: Advanced Features (Week 4)**

#### **10. contest-service (Day 20-21)**
- Contest management
- Real-time leaderboards
- **Why Tenth**: Requires all core services to be ready

#### **11. notification-service (Day 22-23)**
- Multi-channel notifications
- Template management
- **Why Last**: Cross-cutting concern, can be added anytime

---

## 🛠️ **DETAILED FIRST WEEK TASKS**

### **Day 1: Setup shared-contracts**

```bash
cd shared-contracts

# Create basic proto files
mkdir -p proto/auth proto/user proto/problem proto/testcase proto/submission proto/build proto/execution proto/contest proto/notification proto/common

# Create basic build.gradle
cat > build.gradle << 'EOF'
plugins {
    id 'java-library'
    id 'com.google.protobuf' version '0.9.4'
}

java {
    sourceCompatibility = JavaVersion.VERSION_21
    targetCompatibility = JavaVersion.VERSION_21
}

dependencies {
    api 'io.grpc:grpc-protobuf:1.58.0'
    api 'io.grpc:grpc-stub:1.58.0'
    api 'javax.annotation:javax.annotation-api:1.3.2'
}

protobuf {
    protoc {
        artifact = 'com.google.protobuf:protoc:3.24.4'
    }
    plugins {
        grpc {
            artifact = 'io.grpc:protoc-gen-grpc-java:1.58.0'
        }
    }
    generateProtoTasks {
        all()*.plugins {
            grpc {}
        }
    }
}
EOF

# Create basic auth.proto
cat > proto/auth/auth.proto << 'EOF'
syntax = "proto3";

package auth;

option java_package = "com.platform.shared.auth";
option java_outer_classname = "AuthProto";

service AuthService {
  rpc ValidateToken(TokenValidationRequest) returns (TokenValidationResponse);
  rpc RefreshToken(RefreshTokenRequest) returns (RefreshTokenResponse);
}

message TokenValidationRequest {
  string token = 1;
}

message TokenValidationResponse {
  bool valid = 1;
  string user_id = 2;
  repeated string roles = 3;
  string error_message = 4;
}

message RefreshTokenRequest {
  string refresh_token = 1;
}

message RefreshTokenResponse {
  string access_token = 1;
  string refresh_token = 2;
  int64 expires_in = 3;
}
EOF

# Generate Java classes
./gradlew generateProto
```

### **Day 2: Setup platform-infrastructure**

```bash
cd platform-infrastructure

# Create docker-compose for local development
mkdir -p docker/development

cat > docker/development/docker-compose.yml << 'EOF'
version: '3.8'

services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: competitive_platform
      POSTGRES_USER: platform_user
      POSTGRES_PASSWORD: platform_pass
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - platform-network

  mongodb:
    image: mongo:7
    environment:
      MONGO_INITDB_ROOT_USERNAME: mongo_user
      MONGO_INITDB_ROOT_PASSWORD: mongo_pass
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db
    networks:
      - platform-network

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - platform-network

  kafka:
    image: confluentinc/cp-kafka:latest
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    ports:
      - "9092:9092"
    depends_on:
      - zookeeper
    networks:
      - platform-network

  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"
    networks:
      - platform-network

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.10.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    networks:
      - platform-network

volumes:
  postgres_data:
  mongodb_data:
  redis_data:
  elasticsearch_data:

networks:
  platform-network:
    driver: bridge
EOF

# Create environment file
cat > docker/development/.env << 'EOF'
# Database Configuration
POSTGRES_DB=competitive_platform
POSTGRES_USER=platform_user
POSTGRES_PASSWORD=platform_pass

# MongoDB Configuration
MONGO_INITDB_ROOT_USERNAME=mongo_user
MONGO_INITDB_ROOT_PASSWORD=mongo_pass

# Application Configuration
JWT_SECRET=your-super-secret-jwt-key-here
REDIS_URL=redis://localhost:6379
KAFKA_BOOTSTRAP_SERVERS=localhost:9092
ELASTICSEARCH_URL=http://localhost:9200
EOF
```

### **Day 3: Setup auth-service**

```bash
cd auth-service

# Create Spring Boot project structure
mkdir -p src/main/java/com/platform/auth
mkdir -p src/main/resources
mkdir -p src/test/java/com/platform/auth

# Create pom.xml
cat > pom.xml << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
        <relativePath/>
    </parent>
    <groupId>com.platform</groupId>
    <artifactId>auth-service</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>auth-service</name>
    <description>Authentication and Authorization Service</description>
    <properties>
        <java.version>21</java.version>
        <spring-cloud.version>2023.0.0</spring-cloud.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-oauth2-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>0.12.3</version>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
            <version>0.12.3</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
            <version>0.12.3</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>net.devh</groupId>
            <artifactId>grpc-server-spring-boot-starter</artifactId>
            <version>2.15.0.RELEASE</version>
        </dependency>
        <!-- Add shared-contracts dependency -->
        <dependency>
            <groupId>com.platform</groupId>
            <artifactId>shared-contracts</artifactId>
            <version>1.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
EOF

# Create main application class
cat > src/main/java/com/platform/auth/AuthServiceApplication.java << 'EOF'
package com.platform.auth;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class AuthServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(AuthServiceApplication.class, args);
    }
}
EOF

# Create application.yml
cat > src/main/resources/application.yml << 'EOF'
server:
  port: 8081

spring:
  application:
    name: auth-service
  
  datasource:
    url: jdbc:postgresql://localhost:5432/competitive_platform
    username: platform_user
    password: platform_pass
    driver-class-name: org.postgresql.Driver
  
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
  
  data:
    redis:
      host: localhost
      port: 6379
      timeout: 2000ms

grpc:
  server:
    port: 9090

jwt:
  secret: your-super-secret-jwt-key-here
  expiration: 86400000 # 24 hours

logging:
  level:
    com.platform.auth: DEBUG
    org.springframework.security: DEBUG
EOF
```

---

## 🎯 **NEXT IMMEDIATE ACTIONS**

### **Right Now - Execute These Commands:**

1. **Start Infrastructure:**
```powershell
cd d:\VsCodeProject\leetcode
mkdir competitive-platform
cd competitive-platform

# Create and start infrastructure
git clone [your-platform-infrastructure-repo]
cd platform-infrastructure\docker\development
docker-compose up -d
```

2. **Setup shared-contracts:**
```powershell
cd ..\..\..
git clone [your-shared-contracts-repo]
cd shared-contracts
# Follow Day 1 tasks above
```

3. **Setup auth-service:**
```powershell
cd ..\
git clone [your-auth-service-repo]
cd auth-service
# Follow Day 3 tasks above
```

### **Success Criteria for Week 1:**
- [ ] All repositories created and cloned
- [ ] Docker infrastructure running (postgres, redis, kafka, etc.)
- [ ] shared-contracts building and generating Java classes
- [ ] auth-service compiling and starting up
- [ ] Basic JWT authentication working

This gives you a solid foundation to build upon. Once Week 1 is complete, we'll move to implementing the core business logic for each service.
