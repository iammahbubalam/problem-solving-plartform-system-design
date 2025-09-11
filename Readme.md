# High-Scale Competitive Programming Platform - System Design

## Table of Contents
1. [System Overview](#system-overview)
2. [Technology Stack & Justification](#technology-stack--justification)
3. [Service Architecture](#service-architecture)
4. [Communication Patterns](#communication-patterns)
5. [Caching Strategy](#caching-strategy)
6. [Message Queue Architecture](#message-queue-architecture)
7. [API Gateway & Load Balancing](#api-gateway--load-balancing)
8. [Detailed Request Flows](#detailed-request-flows)
9. [Performance & Scaling](#performance--scaling)

## System Overview

A high-performance competitive programming platform designed to handle **1M+ concurrent users** with **10K+ RPS** through modern cloud-native architecture. Built for extreme scalability using event-driven microservices, advanced caching, and intelligent load distribution.

### Core Design Principles
- **Event-Driven Architecture**: Asynchronous processing for high throughput
- **CQRS + Event Sourcing**: Separate read/write models for optimal performance  
- **Circuit Breaker Pattern**: Fault tolerance and graceful degradation
- **Horizontal Auto-Scaling**: Dynamic resource allocation based on real-time metrics
- **Multi-Layer Caching**: Redis Cluster + CDN for sub-10ms response times

## Technology Stack & Justification

### Application Layer
- **Spring Boot 3.2 + Java 21**: Virtual threads, reactive programming, enterprise-grade framework
- **Spring WebFlux**: Non-blocking I/O, handles 100K+ concurrent requests per instance
- **gRPC with Spring**: Binary protocol for inter-service communication, type-safe contracts
- **WebSockets**: Real-time contest updates with STOMP protocol

### API Gateway & Reverse Proxy
- **Kong Gateway**: High-performance, plugin-based API gateway (50K+ RPS)
- **HAProxy**: Layer 4/7 load balancer, SSL termination, health checks
- **Reasons**: Kong provides advanced rate limiting, auth plugins, circuit breakers
- **Alternative**: Envoy Proxy for service mesh capabilities

### Infrastructure & Orchestration
- **Kubernetes**: Container orchestration, auto-scaling, service discovery
- **Istio Service Mesh**: Traffic management, security policies, observability
- **NGINX (Reverse Proxy)**: Static content serving, request routing, compression
- **Helm Charts**: Declarative deployment and configuration management

### Data Storage Strategy
- **PostgreSQL (Clustered)**: ACID compliance, complex queries, user/contest data
- **MongoDB (Sharded)**: High write throughput for submissions and logs
- **Redis Cluster**: Distributed caching, session management, leaderboards
- **Elasticsearch**: Full-text search for problems, advanced filtering

### Message Queue & Event Streaming
- **Apache Kafka**: Event sourcing, high throughput (1M+ events/sec), durability
- **Redis Pub/Sub**: Real-time notifications, contest live updates  
- **RabbitMQ**: Dead letter queues, retry mechanisms, complex routing

### Observability & Monitoring
- **Micrometer + Prometheus**: Spring Boot metrics, custom business metrics
- **Grafana**: Real-time dashboards, alerting, performance visualization
- **Zipkin**: Distributed tracing for Spring Boot microservices
- **ELK Stack**: Centralized logging with Logstash parsing

## Detailed System Design Flows for ALL Endpoints

### Message Queue Strategy Distribution
```
┌─────────────────────────────────────────────────────────────────┐
│                    MESSAGE QUEUE USAGE                                      │
│                                                                             │
│  Apache Kafka: High-volume event streaming                                  │
│  ├── Submission events (25K/min)                                            │
│  ├── User activity events (100K/min)                           │
│  └── Contest events (50K/min)                                  │
│                                                                 │
│  RabbitMQ: Critical reliable operations                        │
│  ├── Email notifications with retry                            │
│  ├── Payment processing                                        │
│  ├── User registration verification                            │
│  └── Failed message handling (DLQ)                             │
│                                                                 │
│  Redis Pub/Sub: Real-time notifications                        │
│  ├── Contest live updates                                      │
│  ├── WebSocket broadcasts                                      │
│  └── System alerts                                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Service Architecture


### 🔄 **Communication Protocol Breakdown**

| **Communication Type** | **Protocol** | **Use Cases** | **Services** |
|------------------------|--------------|---------------|-------------|
| **External API** | REST + WebFlux | Client ↔ Services | All Services |
| **Inter-Service** | gRPC (Binary) | Service ↔ Service | Auth ↔ All, Submission ↔ Problem/Contest |
| **Event Streaming** | Kafka | Async Processing | Submission → Build → Execution |
| **Real-time Updates** | WebSocket + STOMP | Live Contest Updates | Contest Service |
| **Cache Operations** | Redis Protocol | Fast Data Access | All Services ↔ Redis |
| **Message Queues** | AMQP | Critical Operations | Email/SMS via RabbitMQ |
| **Service Mesh** | mTLS + HTTP/2 | Secure Communication | All Inter-service calls |

### 🗂️ **Service-to-Cache/DB Mapping**

| **Service** | **Redis Usage** | **Primary DB** | **Secondary DB** | **Message Broker** |
|-------------|-----------------|----------------|------------------|-------------------|
| **Auth Service** | Sessions, Rate Limits | PostgreSQL | - | RabbitMQ (Email) |
| **User Service** | Profile Cache, Leaderboards | PostgreSQL | Elasticsearch | Kafka (Events) |
| **Problem Service** | Problem Cache, Search Results | PostgreSQL | Elasticsearch | Kafka (Events) |
| **Testcase Service** | Validation Cache | PostgreSQL | - | - |
| **Submission Service** | Status Cache | MongoDB | PostgreSQL | Kafka (Producer) |
| **Build Service** | Build Cache | MongoDB | - | Kafka (Consumer) |
| **Execution Service** | Result Cache | MongoDB | - | Kafka (Consumer) |
| **Contest Service** | Live Leaderboards, Pub/Sub | PostgreSQL | - | Redis Pub/Sub |
| **Notification Service** | Template Cache | - | - | RabbitMQ (Consumer) |

### 🏗️ Multi-Tier Architecture Overview

```
╔═══════════════════════════════════════════════════════════════════════════════════════════════════════════╗
║                                        🌍 CLIENT ECOSYSTEM                                                ║
║                                                                                                           ║
║    📱 Mobile App      💻 Web App        🔧 CLI Tools       🤖 Third-party APIs     📊 Admin Dashboard   ║
║    (React Native)     (React/Vue)       (Official SDK)    (Partner Integration)    (Analytics Panel)    ║
║    • iOS/Android      • PWA Support     • Language Tools  • Contest Platforms     • System Metrics     ║
║    • Push Notifications• Real-time UI   • Bulk Operations • Educational Sites     • User Management    ║
║    • Offline Mode     • Dark/Light Mode • CI/CD Integration• API Monetization      • Content Moderation ║
╚═══════════════════════════════════════════════════════════════════════════════════════════════════════════╝
                                                    │
                                            ⚡ HTTPS/WSS/gRPC
                                                    ▼
╔═══════════════════════════════════════════════════════════════════════════════════════════════════════════╗
║                                    🛡️ REVERSE PROXY & CDN LAYER                                          ║
║                                                                                                           ║
║  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐              ║
║  │   🌐 CloudFlare │    │   ⚖️ HAProxy    │    │  📡 NGINX CDN   │    │  🔒 SSL Gateway │              ║
║  │   • DDoS Protect│    │   Cluster       │    │  • Static Assets│    │  • Cert Manager │              ║
║  │   • Global CDN  │    │   • L4/L7 LB    │    │  • Image Optim  │    │  • TLS 1.3      │              ║
║  │   • Bot Protect │    │   • Health Check│    │  • Compression  │    │  • HSTS Headers │              ║
║  │   • Rate Limit  │    │   • Failover    │    │  • Edge Caching │    │  • Certificate  │              ║
║  │   • Analytics   │    │   • Sticky Sess │    │  • Geo-routing  │    │    Auto-renewal │              ║
║  └─────────────────┘    └─────────────────┘    └─────────────────┘    └─────────────────┘              ║
║                                                                                                           ║
║  📊 Performance: 50K+ RPS │ 🌍 99.9% Uptime │ ⚡ <20ms Latency │ 🔐 Zero Trust Security                ║
╚═══════════════════════════════════════════════════════════════════════════════════════════════════════════╝
                                                    │
                                            📡 Intelligent Routing
                                                    ▼
╔═══════════════════════════════════════════════════════════════════════════════════════════════════════════╗
║                                      🚪 API GATEWAY ORCHESTRATION (Kong)                                 ║
║                                                                                                           ║
║  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐              ║
║  │  🔑 Auth Plugin │    │ 📊 Rate Limiter │    │ 🔄 Circuit Break│    │ 📈 Analytics    │              ║
║  │  • JWT Validation│   │ • Redis Backend │    │ • Hystrix Pattern│   │ • Request Logs  │              ║
║  │  • OAuth2 Flow  │    │ • Sliding Window│    │ • Health Checks │    │ • Metrics Export│              ║
║  │  • RBAC Control │    │ • Custom Quotas │    │ • Fallback APIs │    │ • Performance   │              ║
║  │  • API Key Mgmt │    │ • Burst Handling│    │ • Retry Logic   │    │   Monitoring    │              ║
║  └─────────────────┘    └─────────────────┘    └─────────────────┘    └─────────────────┘              ║
║                                                                                                           ║
║  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐              ║
║  │ 🔄 Transform    │    │ 📦 Response     │    │ 🌐 API Gateway │    │ 🔌 Plugin       │              ║
║  │ • Request/Resp  │    │   Caching       │    │   Clustering    │    │   Ecosystem     │              ║
║  │ • Data Mapping  │    │ • Smart Cache   │    │ • High Avail.   │    │ • Custom Plugins│              ║
║  │ • Validation    │    │ • TTL Management│    │ • Auto-scaling  │    │ • Lua Scripts   │              ║
║  │ • Enrichment    │    │ • Cache Warming │    │ • Load Balancing│    │ • Marketplace   │              ║
║  └─────────────────┘    └─────────────────┘    └─────────────────┘    └─────────────────┘              ║
║                                                                                                           ║
║  🎯 Features: 100+ Plugins │ 🚀 50K+ RPS │ 🔒 Enterprise Security │ 📊 Real-time Analytics             ║
╚═══════════════════════════════════════════════════════════════════════════════════════════════════════════╝
                                                    │
                                              🕸️ Service Mesh
                                                    ▼
╔═══════════════════════════════════════════════════════════════════════════════════════════════════════════╗
║                                        🕸️ SERVICE MESH (Istio)                                          ║
║                                                                                                           ║
║  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐              ║
║  │ 🔒 Security     │    │ 📡 Traffic Mgmt │    │ 👁️ Observability│    │ 🔄 Resilience   │              ║
║  │ • mTLS Auto     │    │ • Smart Routing │    │ • Distributed   │    │ • Circuit Break │              ║
║  │ • Zero Trust    │    │ • A/B Testing   │    │   Tracing       │    │ • Retries       │              ║
║  │ • Policy Engine │    │ • Canary Deploy │    │ • Metrics       │    │ • Timeouts      │              ║
║  │ • AuthZ Policies│    │ • Traffic Split │    │ • Service Graph │    │ • Bulkhead      │              ║
║  └─────────────────┘    └─────────────────┘    └─────────────────┘    └─────────────────┘              ║
║                                                                                                           ║
║  🌟 Service Discovery │ 🔄 Load Balancing │ 📊 Telemetry │ 🛡️ Security Policies                       ║
╚═══════════════════════════════════════════════════════════════════════════════════════════════════════════╝
                                                    │
                                            🏗️ Microservices Layer
                                                    ▼
╔═══════════════════════════════════════════════════════════════════════════════════════════════════════════╗
║                                   🚀 SPRING BOOT MICROSERVICES ECOSYSTEM                                ║
║                                                                                                           ║
║  ┌──────────────────────┐  ┌──────────────────────┐  ┌──────────────────────┐  ┌──────────────────────┐ ║
║  │    🔐 Auth Service    │  │   👤 User Service    │  │  📝 Problem Service  │  │ 📋 Testcase Service │ ║
║  │                      │  │                      │  │                      │  │                      │ ║
║  │ 🛠️ Spring WebFlux    │  │ 🛠️ Spring WebFlux    │  │ 🛠️ Spring WebFlux    │  │ 🛠️ Spring WebFlux    │ ║
║  │ ⚡ Reactive Streams  │  │ ⚡ Reactive Streams  │  │ ⚡ Reactive Streams  │  │ ⚡ Reactive Streams  │ ║
║  │ 🔗 gRPC + REST       │  │ 🔗 gRPC + REST       │  │ 🔗 gRPC + REST       │  │ 🔗 gRPC + REST       │ ║
║  │ 🎯 JWT Validation    │  │ 🎯 Profile Mgmt      │  │ 🎯 CRUD + Search     │  │ 🎯 Secure Storage   │ ║
║  │ 🔄 Session Store     │  │ 🔄 Statistics        │  │ 🔄 Elasticsearch     │  │ 🔄 Encryption       │ ║
║  │ 📊 OAuth2/OIDC       │  │ 📊 Leaderboards      │  │ 📊 Recommendations   │  │ 📊 Validation       │ ║
║  │                      │  │                      │  │                      │  │                      │ ║
║  │ 📈 Replicas: 2→50    │  │ 📈 Replicas: 2→30    │  │ 📈 Replicas: 2→40    │  │ 📈 Replicas: 2→20   │ ║
║  │ ⚡ 30K RPS           │  │ ⚡ 25K RPS           │  │ ⚡ 35K RPS           │  │ ⚡ 15K RPS           │ ║
║  │ 🎯 <20ms Latency     │  │ 🎯 <15ms Latency     │  │ 🎯 <25ms Latency     │  │ 🎯 <30ms Latency    │ ║
║  └──────────────────────┘  └──────────────────────┘  └──────────────────────┘  └──────────────────────┘ ║
║                                                                                                           ║
║  ┌──────────────────────┐  ┌──────────────────────┐  ┌──────────────────────┐  ┌──────────────────────┐ ║
║  │ 📤 Submission Service│  │  🔨 Build Service    │  │ ⚙️ Execution Service │  │ 🏆 Contest Service   │ ║
║  │                      │  │                      │  │                      │  │                      │ ║
║  │ 🛠️ Spring WebFlux    │  │ 🛠️ Spring WebFlux    │  │ 🛠️ Spring WebFlux    │  │ 🛠️ Spring WebFlux    │ ║
║  │ ⚡ Event Sourcing    │  │ ⚡ Docker Engine     │  │ ⚡ K8s Jobs          │  │ ⚡ WebSocket STOMP   │ ║
║  │ 🔗 Kafka Events      │  │ 🔗 Multi-language    │  │ 🔗 Sandboxed Exec   │  │ 🔗 Real-time Updates │ ║
║  │ 🎯 Code Validation   │  │ 🎯 Dependency Cache  │  │ 🎯 Resource Limits   │  │ 🎯 Live Leaderboard │ ║
║  │ 🔄 Status Tracking   │  │ 🔄 Security Scan     │  │ 🔄 Result Analysis   │  │ 🔄 Contest Mgmt     │ ║
║  │ 📊 Analytics         │  │ 📊 Build Metrics     │  │ 📊 Performance Data  │  │ 📊 Real-time Stats  │ ║
║  │                      │  │                      │  │                      │  │                      │ ║
║  │ 📈 Replicas: 2→60    │  │ 📈 Replicas: 2→100   │  │ 📈 Replicas: 2→200   │  │ 📈 Replicas: 2→30   │ ║
║  │ ⚡ 20K Sub/min       │  │ ⚡ 8K Builds/min     │  │ ⚡ 12K Exec/min      │  │ ⚡ 15K Concurrent   │ ║
║  │ 🎯 <30ms Latency     │  │ 🎯 <5min Build       │  │ 🎯 <30sec Execution  │  │ 🎯 <10ms Updates    │ ║
║  └──────────────────────┘  └──────────────────────┘  └──────────────────────┘  └──────────────────────┘ ║
║                                                                                                           ║
║  🔄 Inter-Service Communication: gRPC (Binary Protocol) │ 📊 Health Checks │ 🔄 Auto-scaling HPA       ║
╚═══════════════════════════════════════════════════════════════════════════════════════════════════════════╝
                                                    │
                                          📨 Event-Driven Architecture
                                                    ▼
╔═══════════════════════════════════════════════════════════════════════════════════════════════════════════╗
║                                   🌊 EVENT STREAMING & MESSAGING LAYER                                   ║
║                                                                                                           ║
║  ┌──────────────────────────────────────┐    ┌──────────────────────────────────────┐                  ║
║  │         🚀 Apache Kafka Cluster      │    │       ⚡ Redis Pub/Sub Network       │                  ║
║  │                                      │    │                                      │                  ║
║  │  📊 Topics & Partitions:             │    │  🔄 Real-time Channels:             │                  ║
║  │  • user-events (100K/min)           │    │  • contest-updates                  │                  ║
║  │  • submission-events (25K/min)      │    │  • live-notifications               │                  ║
║  │  • build-events (8K/min)            │    │  • system-alerts                    │                  ║
║  │  • execution-events (12K/min)       │    │  • leaderboard-updates              │                  ║
║  │  • contest-events (50K/min)         │    │  • websocket-broadcasts             │                  ║
║  │                                      │    │                                      │                  ║
║  │  🎯 Features:                        │    │  🎯 Features:                        │                  ║
║  │  • Event Sourcing                   │    │  • Sub-second Latency               │                  ║
║  │  • CQRS Pattern                     │    │  • Pattern Matching                 │                  ║
║  │  • Dead Letter Queues               │    │  • Message Filtering                │                  ║
║  │  • Exactly-once Delivery            │    │  • Connection Scaling               │                  ║
║  │  • Stream Processing (KStreams)     │    │  • Cluster Replication              │                  ║
║  └──────────────────────────────────────┘    └──────────────────────────────────────┘                  ║
║                                                                                                           ║
║  ┌──────────────────────────────────────┐                                                               ║
║  │         🐰 RabbitMQ Cluster          │      🔄 Message Patterns:                                     ║
║  │                                      │      • Event Sourcing (Kafka)                                ║
║  │  🎯 Critical Operations:             │      • Pub/Sub (Redis)                                        ║
║  │  • Email Notifications              │      • Work Queues (RabbitMQ)                                 ║
║  │  • Payment Processing               │      • Request/Reply (gRPC)                                   ║
║  │  • User Registration                │      • Message Routing (AMQP)                                 ║
║  │  • Failed Message Handling          │      • Circuit Breaker Integration                            ║
║  │  • Retry Mechanisms                 │                                                               ║
║  │  • Dead Letter Queues               │      📊 Throughput: 1M+ Events/sec                           ║
║  │  • Priority Queuing                 │      ⚡ Latency: <100ms end-to-end                           ║
║  └──────────────────────────────────────┘      🔒 Guaranteed Delivery & Ordering                       ║
║                                                                                                           ║
╚═══════════════════════════════════════════════════════════════════════════════════════════════════════════╝
                                                    │
                                            💾 Persistent Storage
                                                    ▼
╔═══════════════════════════════════════════════════════════════════════════════════════════════════════════╗
║                                      💾 POLYGLOT PERSISTENCE LAYER                                       ║
║                                                                                                           ║
║  ┌──────────────────────┐  ┌──────────────────────┐  ┌──────────────────────┐  ┌──────────────────────┐ ║
║  │   🐘 PostgreSQL      │  │    🍃 MongoDB        │  │   🔴 Redis Cluster   │  │  🔍 Elasticsearch    │ ║
║  │    Cluster           │  │    Sharded           │  │                      │  │    Cluster           │ ║
║  │                      │  │                      │  │                      │  │                      │ ║
║  │ 🎯 ACID Compliance   │  │ 🎯 High Write        │  │ 🎯 In-Memory Cache   │  │ 🎯 Full-text Search  │ ║
║  │ 🔗 Complex Relations │  │    Throughput        │  │ ⚡ Sub-ms Latency    │  │ 📊 Analytics Engine │ ║
║  │ 📊 Advanced Queries  │  │ 📄 Document Store    │  │ 🔄 Session Store     │  │ 🔍 Faceted Search   │ ║
║  │ 🔒 Row-level Security│  │ ⚡ Horizontal Scale  │  │ 📊 Leaderboards      │  │ 📈 Real-time Index  │ ║
║  │                      │  │                      │  │ 🎯 Pub/Sub           │  │ 🌐 Multi-language   │ ║
║  │ 📦 Data Types:       │  │ 📦 Data Types:       │  │                      │  │                      │ ║
║  │ • User Profiles      │  │ • Submissions Code   │  │ 📦 Cache Types:      │  │ 📦 Index Types:     │ ║
║  │ • Problem Metadata  │  │ • Build Logs         │  │ • User Sessions      │  │ • Problem Content   │ ║
║  │ • Contest Data       │  │ • Execution Results  │  │ • API Responses      │  │ • User Activity     │ ║
║  │ • User Statistics    │  │ • File Storage       │  │ • Database Queries   │  │ • System Logs       │ ║
║  │ • Audit Logs         │  │ • Binary Data        │  │ • Computed Results   │  │ • Search Queries    │ ║
║  │                      │  │                      │  │ • Real-time Data     │  │ • Analytics Data    │ ║
║  │ 🔄 Master-Replica    │  │ 🔄 Sharded Cluster   │  │                      │  │                      │ ║
║  │ 📈 Read Replicas     │  │ 📈 Auto-sharding     │  │ 🔄 Cluster Mode      │  │ 🔄 Multi-node       │ ║
║  │ ⚡ Connection Pool   │  │ ⚡ Write Scaling     │  │ 📈 Memory Optimize   │  │ 📈 Shard Balancing  │ ║
║  │ 🔒 Encryption at Rest│  │ 🔒 Field Encryption  │  │ ⚡ Persistence       │  │ ⚡ Query Cache      │ ║
║  └──────────────────────┘  └──────────────────────┘  └──────────────────────┘  └──────────────────────┘ ║
║                                                                                                           ║
║  📊 Performance Metrics:                                                                                  ║
║  • PostgreSQL: 10K+ TPS │ • MongoDB: 50K+ Writes/sec │ • Redis: 1M+ Ops/sec │ • Elasticsearch: 5K+ QPS║
║  • High Availability: 99.99% │ • Auto-scaling │ • Backup & Recovery │ • Disaster Recovery             ║
║                                                                                                           ║
╚═══════════════════════════════════════════════════════════════════════════════════════════════════════════╝
                                                    │
                                          📊 Observability & Monitoring
                                                    ▼
╔═══════════════════════════════════════════════════════════════════════════════════════════════════════════╗
║                                   📊 OBSERVABILITY & MONITORING STACK                                    ║
║                                                                                                           ║
║  ┌──────────────────────┐  ┌──────────────────────┐  ┌──────────────────────┐  ┌──────────────────────┐ ║
║  │  📈 Prometheus       │  │   📊 Grafana         │  │  🔍 Zipkin/Jaeger    │  │  📝 ELK Stack        │ ║
║  │  Metrics Collection  │  │   Visualization      │  │  Distributed Tracing │  │  Centralized Logs    │ ║
║  │                      │  │                      │  │                      │  │                      │ ║
║  │ • Spring Actuator    │  │ • Real-time Dashbrd │  │ • Request Tracing    │  │ • Elasticsearch      │ ║
║  │ • Custom Metrics     │  │ • Business KPIs     │  │ • Latency Analysis   │  │ • Logstash           │ ║
║  │ • Infrastructure     │  │ • Performance Alerts│  │ • Error Tracking     │  │ • Kibana             │ ║
║  │ • Application Health │  │ • SLA Monitoring    │  │ • Service Dependency │  │ • Filebeat           │ ║
║  └──────────────────────┘  └──────────────────────┘  └──────────────────────┘  └──────────────────────┘ ║
║                                                                                                           ║
║  🎯 Monitoring Targets: 99.9% Uptime │ <100ms API Response │ <30sec Processing │ Real-time Alerting    ║
╚═══════════════════════════════════════════════════════════════════════════════════════════════════════════╝
```

### �️ Service-Level Architecture Deep Dive

### Missing Critical Services (Added to Architecture)

#### 9. Plagiarism Detection Service (Spring Boot + WebFlux)
- **Purpose**: Code similarity detection, academic integrity enforcement
- **Tech**: Spring Boot + NLP libraries + Machine Learning + Redis + PostgreSQL
- **Scale**: 2→15 replicas (handles 5K checks/min)
- **Features**: AST-based comparison, ML similarity detection, whitelist management

#### 10. Security Scanner Service (Spring Boot + WebFlux) 
- **Purpose**: Static code analysis, vulnerability detection, malicious code prevention
- **Tech**: Spring Boot + OWASP tools + Custom patterns + MongoDB
- **Scale**: 2→20 replicas (handles 3K scans/min)
- **Features**: Pattern matching, dependency scanning, runtime behavior analysis

#### 11. Resource Manager Service (Spring Boot + WebFlux)
- **Purpose**: Compute resource allocation, quota management, capacity planning
- **Tech**: Spring Boot + Kubernetes API + Prometheus + Redis
- **Scale**: 2→10 replicas (handles resource requests)
- **Features**: Resource pooling, quota enforcement, auto-scaling triggers

#### 12. Notification Service (Spring Boot + WebFlux)
- **Purpose**: Multi-channel notifications, email/SMS/push delivery
- **Tech**: Spring Boot + RabbitMQ + Redis + External APIs
- **Scale**: 2→25 replicas (handles 50K notifications/min)
- **Features**: Template management, delivery tracking, preference handling

#### 13. Audit Service (Spring Boot + WebFlux)
- **Purpose**: Security audit, compliance tracking, activity logging
- **Tech**: Spring Boot + Elasticsearch + Kafka + PostgreSQL
- **Scale**: 2→15 replicas (handles audit events)
- **Features**: Tamper-proof logging, compliance reports, anomaly detection

#### **Complete Service List Summary**:
```
✅ 13 Core Microservices:
1️⃣  Auth Service - Authentication & authorization
2️⃣  User Service - Profile & statistics management
3️⃣  Problem Service - Problem CRUD & search
4️⃣  Testcase Service - Test case management & security
5️⃣  Submission Service - Code submission handling
6️⃣  Build Service - Code compilation & dependency management
7️⃣  Execution Service - Secure code execution & sandboxing
8️⃣  Contest Service - Contest management & real-time updates
9️⃣  Plagiarism Detection Service - Code similarity & integrity
🔟 Security Scanner Service - Vulnerability & malicious code detection
1️⃣1️⃣ Resource Manager Service - Compute allocation & quota management
1️⃣2️⃣ Notification Service - Multi-channel delivery (email/SMS/push)
1️⃣3️⃣ Audit Service - Compliance tracking & security logging
```

### Service Responsibilities

#### 1. Auth Service (Spring Boot + WebFlux)
- **Purpose**: Authentication, authorization, session management
- **Tech**: Spring Security + WebFlux + Redis + JWT + OAuth2
- **Scale**: 2→50 replicas (handles 30K RPS)
- **Features**: Multi-factor auth, session clustering, rate limiting

#### 2. User Service (Spring Boot + WebFlux)
- **Purpose**: User profiles, statistics, leaderboards, social features
- **Tech**: Spring Data R2DBC + Redis + PostgreSQL + Kafka
- **Scale**: 2→30 replicas (handles 25K RPS)
- **Features**: CQRS pattern, cached leaderboards, user analytics

#### 3. Problem Service (Spring Boot + WebFlux)
- **Purpose**: Problem CRUD, search, categorization, recommendations
- **Tech**: Spring Data R2DBC + Elasticsearch + Redis + PostgreSQL
- **Scale**: 2→40 replicas (handles 35K RPS)
- **Features**: Full-text search, AI-powered recommendations, caching

#### 4. Testcase Service (Spring Boot + WebFlux)
- **Purpose**: Test case management, validation, security
- **Tech**: Spring Data R2DBC + Redis + PostgreSQL + Encryption
- **Scale**: 2→20 replicas (handles 15K RPS)
- **Features**: Encrypted storage, validation pipelines, access control

#### 5. Submission Service (Spring Boot + WebFlux)
- **Purpose**: Code submission handling, history, status tracking
- **Tech**: Spring Data MongoDB + Kafka + Redis + PostgreSQL
- **Scale**: 2→60 replicas (handles 20K submissions/min)
- **Features**: Event sourcing, async processing, submission analytics

#### 6. Build Service (Spring Boot + WebFlux)
- **Purpose**: Code compilation, dependency management, Docker builds
- **Tech**: Spring Boot + Docker + Kafka + Redis + MongoDB
- **Scale**: 2→100 replicas (handles 8K builds/min)
- **Features**: Multi-language support, cached dependencies, security scanning

#### 7. Execution Service (Spring Boot + WebFlux)
- **Purpose**: Secure code execution, resource monitoring, sandboxing
- **Tech**: Spring Boot + Kubernetes Jobs + Kafka + Redis + MongoDB
- **Scale**: 2→200 replicas (handles 12K executions/min)
- **Features**: Sandboxed execution, resource limits, timeout handling

## � **Data Consistency & Transaction Management**

### **Cross-Service Transaction Patterns**
```
Saga Pattern Implementation:
├── Submission Processing Saga:
│   ├── Compensating Transactions: Rollback failed submission states
│   ├── Coordination: Centralized orchestrator via Kafka events
│   ├── State Management: Redis for saga state tracking
│   └── Timeout Handling: Automatic compensation after 5 minutes

CQRS (Command Query Responsibility Segregation):
├── Write Models: PostgreSQL for transactional operations
├── Read Models: Elasticsearch + Redis for optimized queries
├── Event Sourcing: Complete audit trail via Kafka events
├── Eventual Consistency: Acceptable lag of 1-2 seconds
└── Data Synchronization: Event-driven updates across services

Two-Phase Commit (Limited Use):
├── User Registration: Profile + Email verification coordination
├── Contest Registration: Payment + Participation record creation
├── Problem Creation: Problem metadata + Test case creation
└── Rollback Strategy: Compensating transactions for failures
```

### **Database Consistency Models**
```
PostgreSQL (Strong Consistency):
├── ACID Transactions: User profiles, contest data, financial records
├── Foreign Key Constraints: Data integrity enforcement
├── Read Replicas: Eventual consistency (max 1-second lag)
├── Connection Pooling: HikariCP with max 20 connections per service
└── Backup Strategy: WAL-E with point-in-time recovery

MongoDB (Eventual Consistency):
├── Write Concern: majority for critical data, 1 for logs
├── Read Concern: majority for consistent reads, local for analytics
├── Sharding Strategy: Hash-based on submission_id
├── Replica Sets: 3-node replica sets per shard
└── Conflict Resolution: Last-write-wins with timestamp ordering

Redis (In-Memory Consistency):
├── Cluster Mode: Hash slot distribution across nodes
├── Persistence: RDB + AOF for durability
├── Replication: Async replication with configurable lag
├── Failover: Redis Sentinel for automatic master election
└── Data Expiration: TTL-based cleanup for temporary data
```

### **Event Ordering & Message Guarantees**
```
Kafka Event Ordering:
├── Partition Strategy: Hash by entity ID (user_id, problem_id)
├── Producer Config: acks=all, retries=MAX_INT, enable.idempotence=true
├── Consumer Config: enable.auto.commit=false, isolation.level=read_committed
├── Exactly-Once Semantics: Transactional producers and consumers
└── Dead Letter Topics: Failed message handling with manual intervention

Message Delivery Guarantees:
├── At-Least-Once: Default for non-critical events (analytics)
├── Exactly-Once: Critical operations (payments, contest results)
├── At-Most-Once: Fire-and-forget for notifications
├── Ordering Guarantees: Within partition ordering for related events
└── Duplicate Detection: Idempotency keys for operation deduplication
```

### **Cache Consistency Strategies**
```
Cache Invalidation Patterns:
├── Write-Through: Update cache and database synchronously
├── Write-Behind: Update database asynchronously from cache
├── Cache-Aside: Application manages cache and database separately
├── Event-Driven Invalidation: Kafka events trigger cache updates
└── TTL-Based Expiration: Time-based cache expiration for safety

Multi-Level Cache Coherence:
├── L1 (Application): Invalidate via event bus
├── L2 (Redis): Invalidate via pub/sub notifications
├── L3 (CDN): Invalidate via API calls or TTL expiration
├── Consistency Window: Max 5-second inconsistency allowed
└── Cache Warming: Proactive cache population for hot data
```

## �🚨 **Error Handling & Failure Scenarios**

### **Database Failure Scenarios**
```
PostgreSQL Cluster Failure:
├── Primary Node Down: Automatic failover to standby (30s RTO)
├── Read Replica Lag: Circuit breaker activates, route to primary
├── Connection Pool Exhaustion: Queue requests, scale read replicas
├── Data Corruption: Point-in-time recovery from WAL backups
└── Split-brain: Use Consul for leader election and quorum

MongoDB Shard Failure:
├── Shard Unavailable: Requests return partial results + error indication
├── Balancer Issues: Manual chunk migration, monitor shard distribution  
├── Config Server Down: Read-only mode until config servers restored
├── Network Partition: Use majority read/write concern for consistency
└── Data Loss: Restore from point-in-time backups with oplog replay

Redis Cluster Failure:
├── Node Failure: Automatic failover to replica node (1s RTO)
├── Cluster Split: Use cluster quorum for split-brain protection
├── Memory Exhaustion: Implement LRU eviction + scale cluster
├── Network Partition: Use Redis Sentinel for automatic failover
└── Data Loss: Graceful degradation to database queries
```

### **Service Communication Failures**
```
gRPC Communication Failures:
├── Service Unavailable: Circuit breaker with exponential backoff
├── Timeout: 5-second timeout with 3 retry attempts
├── Network Partition: Fallback to cached data or degraded functionality
├── Load Balancing: Health check based routing with automatic failover
└── Version Mismatch: Backward compatibility with graceful degradation

Kafka Message Failures:
├── Broker Down: Producer buffering + automatic leader election
├── Consumer Lag: Scale consumer groups + partition rebalancing
├── Message Loss: Use acks=all + min.insync.replicas=2
├── Duplicate Messages: Implement idempotent consumers with deduplication
└── Poison Messages: Dead letter topic with manual intervention

Kong Gateway Failures:
├── Instance Down: HAProxy health check removes from pool
├── Rate Limit Redis Down: Fallback to local rate limiting
├── Plugin Failure: Bypass failed plugin, log error for investigation
├── SSL Certificate Expiry: Automated renewal with Let's Encrypt
└── Configuration Error: Blue-green deployment with rollback capability
```

### **Security Incident Response**
```
DDoS Attack Response:
├── Detection: Prometheus alerts on traffic spikes + CloudFlare protection
├── Mitigation: Rate limiting escalation + IP blacklisting
├── Traffic Analysis: Identify attack patterns + update firewall rules
├── Resource Scaling: Auto-scale affected services + increase quotas
└── Communication: Status page updates + incident response team

Data Breach Response:
├── Detection: Anomalous access patterns + unauthorized queries
├── Containment: Revoke compromised tokens + reset affected passwords
├── Investigation: Audit log analysis + forensic data collection
├── Notification: User notification + regulatory compliance reporting
└── Recovery: Security patches + access control improvements

Code Injection Prevention:
├── Input Validation: Strict schema validation + content filtering
├── Code Sandboxing: Isolated execution environment + system call filtering
├── Static Analysis: Automated code scanning + pattern detection
├── Runtime Protection: Monitor execution behavior + kill suspicious processes
└── Incident Response: Quarantine malicious code + alert security team
```

### **Performance Degradation Handling**
```
High Latency Response:
├── Detection: P95 latency > 100ms triggers alert
├── Analysis: Distributed tracing identifies bottleneck service
├── Mitigation: Auto-scale affected service + enable caching
├── Traffic Shaping: Route traffic to healthy instances
└── Optimization: Query optimization + cache warming

Resource Exhaustion:
├── CPU Throttling: HPA scales pods when CPU > 70%
├── Memory Pressure: Implement graceful degradation + increase limits
├── Storage Full: Automated cleanup + storage expansion
├── Network Congestion: Load balancing + traffic prioritization
└── Connection Limits: Connection pooling + circuit breakers

Contest Load Spike:
├── Predictive Scaling: Pre-scale services 30 minutes before contest
├── Traffic Prioritization: Contest traffic gets highest priority
├── Graceful Degradation: Disable non-essential features during peak
├── Queue Management: Fair queuing for submission processing
└── Communication: Real-time status updates to participants
```

## Communication Patterns

### 1. External API Communication (REST + WebFlux)
- **Frontend to Services**: RESTful APIs with Spring WebFlux for non-blocking I/O
- **Response Format**: JSON with standardized error handling
- **Authentication**: JWT tokens validated at Kong Gateway
- **Rate Limiting**: Applied at Kong with Redis backend
- **Performance**: 100K+ concurrent requests per service instance

### 2. Inter-Service Communication (gRPC)
- **Protocol**: Binary gRPC for high performance between microservices
- **Connection Management**: Connection pooling with keep-alive
- **Load Balancing**: Client-side load balancing with health checks
- **Timeouts**: Configurable timeouts with circuit breaker pattern
- **Performance**: 2-5ms latency between services

### 3. Event-Driven Communication (Kafka + Spring)
- **Event Publishing**: Asynchronous event publishing for non-critical operations
- **Topic Strategy**: Partitioned by entity ID for ordering guarantees
- **Consumer Groups**: Multiple consumers for horizontal scaling
- **Error Handling**: Dead letter queues via RabbitMQ for failed events
- **Throughput**: 1M+ events per second across topics

### 4. Real-time Communication (WebSocket + STOMP)
- **Protocol**: WebSocket with STOMP for structured messaging
- **Connection Management**: Persistent connections with heartbeat
- **Broadcasting**: Redis Pub/Sub for message distribution
- **Scaling**: Sticky sessions with consistent hashing
- **Performance**: 100K+ concurrent WebSocket connections per instance
```

## Caching Strategy

### Multi-Layer Caching Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                    CACHING LAYERS                               │
│                                                                 │
│  L1: Application Cache (Spring Cache)                          │
│  ├── Caffeine Cache for hot data (1-5min TTL)                  │
│  ├── Problem metadata, user sessions, frequent queries         │
│  └── Capacity: 100MB per service instance                      │
│                                                                 │
│  L2: Redis Cluster (Distributed Cache)                         │
│  ├── User profiles, problem data, submission status            │
│  ├── Session store, leaderboards, search results              │
│  ├── TTL: 5min-2hours based on data type                       │
│  └── Capacity: 200GB+ with auto-sharding                       │
│                                                                 │
│  L3: NGINX Reverse Proxy Cache                                 │
│  ├── Static content, API responses, compressed assets          │
│  ├── TTL: 1-24 hours with cache invalidation                   │
│  └── Edge locations for <20ms latency                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Cache Patterns & Strategy
- **Write-Through Cache**: Critical user data with immediate consistency
- **Cache-Aside Pattern**: Read-heavy data like problem metadata  
- **Write-Behind Cache**: Analytics data with eventual consistency
- **Cache Invalidation**: Event-driven cache eviction via Kafka
- **Cache Warming**: Predictive loading of frequently accessed data
- **Performance Targets**: 95%+ cache hit ratio, <10ms cache response time

## Message Queue Architecture

### Event Processing Patterns
- **Saga Pattern**: Distributed transaction management for complex workflows
- **Event Sourcing**: Complete audit trail of all system events
- **CQRS**: Separate read/write models for optimal performance
- **Dead Letter Queues**: Failed message handling with manual intervention
- **Circuit Breaker**: Fault tolerance for message processing
- **Retry Logic**: Exponential backoff with jitter for failed operations

## Kong API Gateway & Load Balancing

### Kong API Gateway Features
- **Advanced Rate Limiting**: Redis-backed rate limiting with burst handling
- **Authentication Plugins**: JWT validation, OAuth2, API key management
- **Response Caching**: Intelligent caching with cache-control headers
- **Circuit Breaker**: Automatic failover with health check integration
- **Request/Response Transformation**: Data format conversion and validation
- **Analytics & Monitoring**: Request metrics with Prometheus integration

### **Kong Plugin Configuration Examples**:
```yaml
# Rate Limiting Plugin
rate-limiting:
  minute: 1000
  hour: 10000
  policy: redis
  redis_host: redis-cluster
  fault_tolerant: true

# JWT Plugin
jwt:
  secret_is_base64: false
  algorithm: RS256
  claims_to_verify: ["exp", "iat"]

# Circuit Breaker Plugin
circuit-breaker:
  failure_threshold: 5
  recovery_timeout: 30
  window_size: 60
```

### Load Balancing Strategy
- **HAProxy**: Layer 4/7 load balancing with health checks
- **Kong**: Application-level load balancing with circuit breakers
- **Istio**: Service mesh load balancing with advanced traffic management
- **Consistent Hashing**: Session affinity for stateful operations
- **Geographic Routing**: Request routing based on client location
- **Auto-Scaling Integration**: Dynamic backend pool management
```

## Detailed System Design Flows for ALL Endpoints

### 1. Authentication Service Flows

#### 1.1 User Registration Flow (POST /api/v1/auth/register)
```
Frontend → HAProxy → Kong → Auth Service → Check Existing → Validate → PostgreSQL → RabbitMQ
    │         │       │         │             │              │         │           │
Validate   SSL      Rate      Parse         Check          Validate   Store       Queue
Input     Term     Limit     Request       Duplicate      Password   User        Email
          │        │         │             Email          Complex    Trans       Event
      Health    100/min    Validate       Redis          Rules      ACID        │
      Check    per IP     Email          Cache          bcrypt     Rollback    DLQ
                         Format                          Cost 12    on Fail     

Phase 1: Request Validation
1. HAProxy: SSL termination, health checks, load balancing
2. Kong: Rate limiting (100 reg/min/IP), request size validation
3. Auth Service: Email format validation, password complexity check

Phase 2: Duplicate Check & Validation
4. Redis Cache: Check existing email (fast lookup)
5. PostgreSQL: Verify email uniqueness (if cache miss)
6. Password validation: Minimum 8 chars, special characters, numbers

Phase 3: User Creation & Notification
7. PostgreSQL: ACID transaction for user creation with encrypted password
8. Redis: Cache email verification token (15min TTL)
9. RabbitMQ: Reliable email queue with retry mechanism
10. Rollback: If email queue fails, rollback user creation

Message Flow:
- RabbitMQ Exchange: "user.operations" (Topic Exchange)
- Queue: "email.verification" with DLQ "email.verification.failed"
- Routing Key: "user.registration.verify"
- Retry Policy: 3 attempts with exponential backoff (1s, 4s, 16s)
- DLQ: Manual intervention for failed email deliveries
- Audit: All registration attempts logged to Kafka for analytics
```

#### 1.2 User Login Flow (POST /api/v1/auth/login)
```
Frontend → HAProxy → Kong → Auth Service → Redis → PostgreSQL → Session → JWT → Kafka
    │         │       │         │           │         │           │       │       │
Credentials SSL     Rate      Parse       Cache     Hash        Create  Generate Publish
Validation Term    Limit     Request     Check     Compare     Session Tokens   Event
            │       │         │           │         │           │       │       │
        Health   1000/hr    Validate    Profile   bcrypt      Redis   RS256   Analytics
        Check   per IP     Format      Cache     Verify      Store   Sign    Stream

Phase 1: Request Processing & Rate Limiting
1. HAProxy: SSL termination, connection pooling, health checks
2. Kong: Advanced rate limiting with Redis backend (1000/hr per IP)
3. Auth Service: Input validation, credential format check

Phase 2: User Lookup & Cache Strategy
4. Redis Cluster: Check user profile cache "user_profile:{email}"
5. If cache hit: Skip database query, proceed to password verification
6. If cache miss: Query PostgreSQL for user credentials
7. Cache user profile for 1 hour with LRU eviction

Phase 3: Authentication & Session Management
8. Password verification: bcrypt compare with stored hash (cost 12)
9. Failed attempt tracking: Redis "failed_attempts:{userId}" (5 attempts/24hr)
10. Session creation: Generate session ID, store in Redis with 24hr TTL
11. JWT generation: RS256 signature with user claims and session ID

Phase 4: Response & Analytics
12. Cache refresh: Update user profile cache with last login time
13. Kafka event: Publish "user_login_event" for analytics and monitoring
14. Response: Return JWT access token + refresh token to client

Security Measures:
- bcrypt password hashing (cost 12, salt rounds)
- JWT with RS256 signature and 1-hour expiration
- Refresh token rotation every 24 hours
- Rate limiting with sliding window (Redis-based)
- Account lockout after 5 failed attempts
- Session invalidation on suspicious activity

Cache Strategy:
- Redis Key: "user_profile:{userId}" (1 hour TTL)
- Redis Key: "failed_attempts:{userId}" (24 hour TTL)
- Redis Key: "session:{sessionId}" (24 hour TTL)
- Cache Hit Ratio Target: 95%
- Fallback: PostgreSQL read replica with 2-second timeout
```

#### 1.3 Token Refresh Flow (POST /api/v1/auth/refresh)
```
Frontend → Kong → Auth Service → Redis → JWT Generation → Blacklist → Kafka
    │       │         │           │         │               │           │
Refresh   Validate   Verify      Check     Generate        Update      Publish
Token     Headers    Signature   Whitelist New Tokens     Blacklist   Audit
          │         │           │         │               │           │
      JWT Plugin  Decode      Refresh    Create          Old Token   Security
      Extract     Claims      Token      New Pair        Invalidate  Event

Phase 1: Token Validation
1. Kong: Extract refresh token from Authorization header
2. Kong JWT Plugin: Basic token format validation
3. Auth Service: Verify refresh token signature and expiration
4. Redis: Check refresh token whitelist "refresh_tokens:{tokenId}"

Phase 2: Security Checks
5. Validate token hasn't been used recently (prevent replay attacks)
6. Check user account status (active, not suspended)
7. Verify token family (detect token theft via rotation)
8. Rate limiting: 10 refresh attempts per hour per user

Phase 3: Token Rotation & Blacklisting
9. Generate new JWT access token (1 hour expiration)
10. Generate new refresh token (24 hour expiration)
11. Blacklist old refresh token in Redis (24 hour TTL)
12. Update refresh token whitelist with new token
13. If theft detected: Invalidate all user sessions

Phase 4: Response & Audit
14. Return new token pair to client
15. Kafka: Publish "token_refresh_event" for security monitoring
16. Update user's last activity timestamp
17. Log refresh attempt for security audit

Security Features:
- Refresh token rotation on every use
- Token family tracking for theft detection
- Blacklist management with TTL cleanup
- Rate limiting per user account
- Audit trail for all token operations
- Automatic session invalidation on breach

Error Handling:
- Invalid token: Return 401 with specific error code
- Expired token: Require full re-authentication
- Rate limit exceeded: Return 429 with retry-after header
- Service unavailable: Return 503 with fallback options
```

### 2. User Management Service Flows

#### 2.1 Get User Profile (GET /api/v1/users/profile)
```
Frontend → Kong → User Service → Redis → PostgreSQL → Response Assembly
    │       │         │           │         │           │
Profile   JWT       Process     Cache     Query         Format
Request  Validate   Request     Check     User Data     Response
         │         │           │         │             │
     Extract     Parse         L1 Cache  Read          Enrich
     Claims      User ID       Hit/Miss  Replica       Data

Phase 1: Authentication & Request Processing
1. Kong: JWT validation and user claim extraction
2. User Service: Parse user ID from JWT claims
3. Input validation: Check request parameters and headers
4. Permission check: Ensure user can access requested profile

Phase 2: Multi-Layer Caching Strategy
5. L1 Application Cache: Check Caffeine cache (5min TTL)
6. L2 Redis Cache: Check distributed cache "user_profile:{userId}" (1hr TTL)
7. Cache hit: Return cached data with freshness timestamp
8. Cache miss: Proceed to database query

Phase 3: Database Query & Data Assembly
9. PostgreSQL: Query user profile from read replica
10. Query optimization: Use prepared statements and connection pooling
11. Data enrichment: Calculate dynamic statistics (submission count, rank)
12. Privacy filtering: Remove sensitive data based on privacy settings

Phase 4: Response Caching & Return
13. Update L1 cache: Store in application memory (Caffeine)
14. Update L2 cache: Store in Redis with compression
15. Response assembly: Format JSON with proper HTTP headers
16. Add cache-control headers: "max-age=3600, must-revalidate"

Performance Optimizations:
- Connection pooling: HikariCP with 20 connections per instance
- Read replicas: Route read queries to PostgreSQL replicas
- Compression: GZIP compression for responses > 1KB
- CDN caching: Cache static profile assets for 24 hours

Cache Strategy Details:
- L1 (Caffeine): 100MB max, 5min TTL, LRU eviction
- L2 (Redis): 1 hour TTL, compressed JSON storage
- L3 (CDN): Static assets cached globally for 24 hours
- Cache warming: Preload popular user profiles
- Cache invalidation: Event-driven via Kafka messages

Error Handling:
- Cache failure: Graceful degradation to database
- Database timeout: Return cached data if available
- Service unavailable: Return 503 with retry-after header
- Rate limiting: 1000 requests per hour per user
```

#### 2.2 Update User Profile (PUT /api/v1/users/profile)
```
Frontend → Kong → User Service → PostgreSQL → Kafka → Cache Invalidation → RabbitMQ
    │       │         │           │           │         │                 │
Profile   Auth      Validate    Transaction  Publish   Redis            Notify
Update   Check     Input       Update       Event     Eviction         Services

Event-Driven Updates:
- Kafka Topic: "user_profile_events"
- Partitioning: By user_id for ordering
- Consumers: Analytics Service, Recommendation Service
- RabbitMQ: Critical notifications (email preferences changed)

Cache Invalidation Strategy:
- Redis: Evict user profile cache
- NGINX: Purge CDN cache for user assets
- Elasticsearch: Async index update via Kafka
```

### 3. Problem Service Flows

#### 3.1 Get Problem List (GET /api/v1/problems)
```
Frontend → NGINX → Kong → Problem Service → Redis → PostgreSQL → Elasticsearch
    │       │       │         │             │         │             │
Request  Cache    Rate      Parse          Cache     Query         Enrich
         Check   Limit     Params         Check     Problems      Metadata

Caching Strategy:
- NGINX: Static problem assets (images, examples)
- Redis: Paginated problem lists (10min TTL)
- Application: Hot problem metadata (5min TTL)

Performance Optimizations:
- Database: Read replicas for query distribution
- Elasticsearch: Problem metadata indexing
- Redis: Sorted sets for leaderboards
- Kong: Response caching plugin
```

#### 3.2 Search Problems (GET /api/v1/problems/search)
```
Frontend → Kong → Problem Service → Redis → Elasticsearch → PostgreSQL → Analytics
    │       │         │             │         │               │           │
Search    Rate      Parse          Cache     Full-text       User        Track
Query    Limit     Query          Check     Search          Data        Searches

Search Architecture:
- Elasticsearch: Multi-field search (title, description, tags)
- Redis: Search result caching (5min TTL)
- PostgreSQL: User-specific data (solved status)
- Kafka: Search analytics events
- ClickHouse: Search pattern analysis

Search Features:
- Auto-complete with Redis
- Faceted search with Elasticsearch aggregations
- Personalized results based on user history
- Search suggestions via machine learning
```

#### 3.3 Create Problem (POST /api/v1/problems)
```
Frontend → Kong → Auth Service → Problem Service → PostgreSQL → Elasticsearch → Kafka
    │       │         │             │               │           │               │
Problem   JWT       Check         Validate        Store       Index           Publish
Data     Validate   Creator       Content         Problem     Search          Event
                   Role                                      Data

Validation Pipeline:
1. Kong: Request size validation (max 1MB)
2. Auth Service: Creator role verification  
3. Problem Service: Content validation, test case validation
4. PostgreSQL: Transactional storage
5. Elasticsearch: Async indexing for search
6. Kafka: Problem creation events
7. RabbitMQ: Email notifications to subscribers

Data Consistency:
- Two-phase commit for problem + test cases
- Eventual consistency for search indexing
- Saga pattern for complex problem creation
```

### 4. Submission Service Flows (MOST COMPLEX)

#### 4.1 Submit Solution (POST /api/v1/submissions) - CORRECTED FLOW
```
Phase 1: Request Validation & Rate Limiting
Frontend → HAProxy → Kong → Auth Service → Submission Service
    │         │       │         │             │
Code +      Load     Rate      Validate     Validate
Metadata    Balance  Limit     JWT Token    Request
            │        │         │            │
        SSL Term   10/min    Extract User   Code Size
        Health     per user   Context       < 100KB
        Check

Phase 2: Pre-submission Validation
Submission Service → Problem Service → Contest Service → Plagiarism Service
        │               │               │               │
    Validate           Get Problem     Check Contest   Check
    Language          Constraints     Window/Rules    Similarity
    Support           Time/Memory     Registration    Threshold
                     Limits          Status          

Phase 3: Security & Storage
Submission Service → Security Scanner → MongoDB → Redis → Kafka
        │               │               │         │       │
    Code              Scan for        Store     Cache   Publish
    Sanitization      Malicious       Source    Status  Event
                     Patterns        Code      Track   Stream

Phase 4: Asynchronous Build Processing
Kafka → Build Service → Docker → Dependency Cache → Security Scan → MongoDB
  │         │           │         │                 │               │
Event    Language     Create     Download          Vulnerability   Store
Stream   Detection    Container  Cached Deps       Scan           Build Result

Phase 5: Asynchronous Execution Processing  
Kafka → Execution Service → Resource Manager → K8s Jobs → Testcase Service
  │         │                │                 │           │
Build     Resource          Allocate         Create       Get Encrypted
Event     Validation        CPU/Memory       Pod          Test Cases

Phase 6: Test Execution & Result Processing
K8s Pod → Code Execution → Output Collection → Result Analysis → MongoDB → Kafka
    │          │              │                │               │         │
Sandbox    Run Against     Capture           Compare         Store     Publish
Isolation  Test Cases      stdout/stderr     Expected        Final     Completion
Network    Time/Memory     Exit Codes        Outputs         Result    Event
Isolation  Monitoring

Detailed Process Flow:

Request Processing (Phase 1):
1. Frontend: Basic syntax validation and code highlighting
2. HAProxy: SSL termination, load balancing to Kong instances
3. Kong: Rate limiting (10 submissions/min/user) with Redis backend
4. Auth Service: JWT validation via gRPC, extract user context
5. Submission Service: Input validation (code size < 100KB)

Pre-submission Checks (Phase 2):
6. Problem Service gRPC: Validate problem exists and get constraints
7. Contest Service gRPC: Check if submission within contest time window
8. User quota validation: Check daily submission limits (100/day)
9. Plagiarism Service: Quick similarity check against recent submissions
10. Language validation: Ensure submitted language is supported

Security & Initial Storage (Phase 3):
11. Code sanitization: Remove potentially harmful code patterns
12. Security scanner: Static analysis for malicious code injection
13. MongoDB storage: Store submission with PENDING status and encrypted code
14. Redis cache: Store submission status for real-time tracking
15. Kafka event: Publish SubmissionCreatedEvent with submission metadata

Build Processing (Phase 4 - Async):
16. Build Service: Kafka consumer receives SubmissionCreatedEvent
17. Language detection: Determine build strategy (Java/Python/C++/etc.)
18. Docker container: Create isolated build environment per language
19. Dependency resolution: Download and cache dependencies (Maven/pip/npm)
20. Code compilation: Compile source code with error handling
21. Security scanning: Deep scan of compiled artifacts
22. Build result: Store success/failure with logs in MongoDB
23. Kafka event: Publish BuildCompletedEvent

Execution Processing (Phase 5 - Async):
24. Execution Service: Kafka consumer receives BuildCompletedEvent
25. Resource Manager: Validate and allocate compute resources
26. Kubernetes Job: Create execution pod with strict resource limits:
    - CPU: 1 core maximum
    - Memory: 512MB maximum  
    - Network: Isolated (no external access)
    - Disk: 1GB temporary storage
    - Timeout: 30 seconds per test case
27. Testcase Service gRPC: Retrieve encrypted test cases for problem

Test Execution (Phase 6):
28. Sandboxed execution: Run code against each test case in isolation
29. Output capture: Collect stdout, stderr, exit codes, timing data
30. Resource monitoring: Track CPU, memory usage during execution
31. Result comparison: Compare outputs with expected results
32. Verdict calculation: Determine final status (AC/WA/TLE/MLE/CE/RE)
33. Result aggregation: Compile all test case results and metrics
34. MongoDB storage: Store final results with execution metadata
35. Redis update: Update submission status cache
36. Kafka event: Publish ExecutionCompletedEvent
37. Cleanup: Remove Kubernetes resources and temporary files

Error Handling & Resilience:
- Circuit Breaker: Execution service protection during high load
- Dead Letter Queue: Failed submissions via RabbitMQ for manual review
- Retry Logic: Exponential backoff for transient failures
- Monitoring: Prometheus alerts for build/execution failures
- Timeout Handling: Graceful termination of long-running processes
- Resource Cleanup: Automatic cleanup of failed execution pods

Data Consistency:
- Event Sourcing: Complete audit trail via Kafka events
- Saga Pattern: Compensating transactions for failed workflows
- Eventual Consistency: MongoDB and Redis sync via events
- Transaction Boundaries: Clear boundaries between services
- Idempotency: All operations designed to be safely retryable
```

### 5. Contest Service Flows

#### 5.1 Join Contest (POST /api/v1/contests/{id}/join)
```
Frontend → Kong → Contest Service → PostgreSQL → Redis → WebSocket → Kafka
    │       │         │               │           │         │           │
Join      Auth      Validate        Update      Cache     Notify      Analytics
Request  Check     Contest         Participants Leaderboard Others     Event

Real-time Architecture:
- WebSocket: STOMP protocol for real-time updates
- Redis Pub/Sub: Contest participant notifications
- Redis Sorted Sets: Live leaderboard management
- Kafka: Contest analytics events
```

#### 5.2 Live Contest Updates (WebSocket)
```
WebSocket → Kong → Contest Service → Redis Pub/Sub → Kafka → ClickHouse
    │        │         │               │             │         │
Connect    Route    Maintain          Subscribe     Stream    Analytics
           │       Connections        Channels      Events    Storage

Real-time Data Flow:
1. WebSocket connection with JWT validation
2. Redis Pub/Sub for low-latency broadcasting
3. Kafka for durable event streaming
4. ClickHouse for real-time analytics
5. Prometheus for connection monitoring

Performance Targets:
- 100K concurrent WebSocket connections
- <100ms update latency
- 99.9% message delivery
- Auto-scaling based on connection count
```

### 6. Build & Execution Service Flows (Internal)

#### 6.1 Build Process Flow (Corrected)
```
Kafka Event → Build Service → Language Detection → Docker → Dependency Cache → Security → Result
     │            │              │                 │         │                │         │
Submission      Parse          Determine         Create     Download         Scan      Update
Event          Metadata       Build Strategy    Container  Dependencies     Code      Status

Phase 1: Event Processing & Language Detection
1. Kafka Consumer: Receive SubmissionCreatedEvent from submission topic
2. Build Service: Parse submission metadata (language, problem ID, user ID)
3. Language Detection: Determine build strategy based on file extension/content
4. Validation: Verify language is supported and build tools are available
5. Resource Check: Ensure build capacity available (max 50 concurrent builds)

Phase 2: Build Environment Preparation
6. Docker Management: Pull/create language-specific build container:
   - Java: OpenJDK 21 + Maven 3.9
   - Python: Python 3.11 + pip
   - C++: GCC 12 + CMake
   - JavaScript: Node.js 20 + npm
7. Volume Mounting: Mount source code and dependency cache volumes
8. Network Isolation: Restrict network access during build
9. Resource Limits: Apply CPU (2 cores) and memory (1GB) limits

Phase 3: Dependency Resolution & Caching
10. Cache Check: Look for cached dependencies using content hash
11. Dependency Download: Download required packages if not cached:
    - Java: Maven dependencies from central repository
    - Python: pip packages from PyPI
    - C++: System libraries and headers
    - JavaScript: npm packages from registry
12. Cache Update: Store downloaded dependencies with TTL (24 hours)
13. Vulnerability Scan: Check dependencies for known security issues

Phase 4: Code Compilation & Security Scanning
14. Source Code Preparation: Copy source code into build container
15. Compilation Process: Execute language-specific build commands:
    - Java: `mvn compile` with error handling
    - Python: Syntax validation and bytecode compilation
    - C++: `g++ -O2 -std=c++17` with optimization
    - JavaScript: Transpilation and bundling if needed
16. Static Analysis: Scan source code for:
    - Malicious system calls
    - Network access attempts
    - File system manipulation
    - Resource exhaustion patterns
17. Build Artifact Creation: Generate executable or bytecode

Phase 5: Result Processing & Cleanup
18. Build Validation: Verify compilation success and artifact integrity
19. Artifact Storage: Store build artifacts in secure storage (encrypted)
20. Metadata Update: Record build time, success rate, error messages
21. MongoDB Update: Store build result with detailed logs
22. Redis Cache: Update build status for real-time tracking
23. Container Cleanup: Remove build container and temporary files
24. Kafka Event: Publish BuildCompletedEvent with build status

Phase 6: Error Handling & Monitoring
25. Build Failures: Capture compilation errors and provide user feedback
26. Timeout Handling: Kill builds exceeding 5-minute time limit
27. Resource Exhaustion: Handle memory/disk space issues gracefully
28. Retry Logic: Retry transient failures up to 3 times
29. Dead Letter Queue: Send failed builds to RabbitMQ for manual review
30. Metrics Collection: Record build time, success rate, error categories

Security & Reliability Features:
- Sandboxed Builds: All builds run in isolated Docker containers
- Dependency Scanning: Automatic vulnerability detection
- Resource Limits: Prevent resource exhaustion attacks
- Network Isolation: No external network access during builds
- Code Scanning: Static analysis for malicious patterns
- Audit Trail: Complete logging of all build activities

Performance Optimizations:
- Dependency Caching: Reduce build time through smart caching
- Parallel Builds: Support multiple concurrent builds per instance
- Build Queue Management: Prioritize based on user tier and contest status
- Resource Scaling: Auto-scale build instances based on queue depth
- Incremental Builds: Cache intermediate build artifacts when possible
```

#### 6.2 Execution Process Flow (Corrected)
```
Build Event → Execution Service → Resource Manager → K8s Jobs → Test Execution → Result Collection
     │            │                 │                │           │               │
Build          Parse             Allocate         Create      Run Against     Aggregate
Completed      Metadata          Resources        Pod         Test Cases      Results

Phase 1: Event Processing & Resource Allocation
1. Kafka Consumer: Receive BuildCompletedEvent from build topic
2. Execution Service: Parse build metadata and verify artifacts
3. Resource Manager: Check available compute capacity
4. Queue Management: Add to execution queue with priority based on:
   - Contest submissions (highest priority)
   - Premium users (high priority)  
   - Regular submissions (normal priority)
5. Resource Allocation: Reserve CPU, memory, and storage resources

Phase 2: Execution Environment Setup
6. Kubernetes Job Creation: Generate execution pod spec with:
   - CPU Limit: 1 core maximum
   - Memory Limit: 512MB maximum
   - Storage: 1GB ephemeral storage
   - Network Policy: No external network access
   - Security Context: Non-root user, read-only filesystem
7. Pod Scheduling: Deploy pod to available Kubernetes node
8. Container Initialization: Start execution container with runtime environment
9. Artifact Loading: Mount compiled code and required runtime libraries

Phase 3: Test Case Retrieval & Preparation
10. Testcase Service gRPC: Request encrypted test cases for problem
11. Decryption: Decrypt test cases using problem-specific keys
12. Test Case Validation: Verify test case integrity and format
13. Input/Output Preparation: Prepare test inputs and expected outputs
14. Execution Order: Randomize test case execution order for security

Phase 4: Code Execution & Monitoring
15. Security Sandbox: Initialize secure execution environment:
    - chroot jail for filesystem isolation
    - cgroup limits for resource control
    - seccomp filters for system call restrictions
    - namespace isolation for process isolation
16. Test Case Execution: For each test case:
    a. Start execution timer
    b. Provide input via stdin
    c. Monitor CPU and memory usage
    d. Capture stdout and stderr
    e. Record exit code and signals
    f. Enforce time limit (30 seconds per test case)
17. Real-time Monitoring: Track resource usage and execution progress

Phase 5: Result Analysis & Verdict Calculation
18. Output Comparison: Compare actual vs expected outputs:
    - Exact match for most problems
    - Floating-point tolerance for numerical problems
    - Custom comparators for special problems
19. Verdict Determination: Calculate final status:
    - AC (Accepted): All test cases passed
    - WA (Wrong Answer): Output mismatch
    - TLE (Time Limit Exceeded): Execution timeout
    - MLE (Memory Limit Exceeded): Memory exhaustion
    - RE (Runtime Error): Program crash or exception
    - CE (Compilation Error): Build failure
20. Performance Metrics: Calculate execution statistics:
    - Total execution time
    - Peak memory usage
    - Test cases passed/failed
    - Performance percentile

Phase 6: Result Storage & Cleanup
21. Result Assembly: Compile comprehensive execution report
22. MongoDB Storage: Store execution results with detailed metrics
23. Redis Update: Update submission status cache with final verdict
24. Kafka Event: Publish ExecutionCompletedEvent with results
25. Resource Cleanup: Delete Kubernetes job and associated resources
26. Cleanup Verification: Ensure no resources are leaked

Phase 7: Error Handling & Edge Cases
27. Execution Timeout: Handle long-running processes:
    - Send SIGTERM after time limit
    - Force kill with SIGKILL after grace period
    - Clean up zombie processes
28. Memory Exhaustion: Handle out-of-memory conditions:
    - Monitor memory usage via cgroups
    - Terminate process when limit exceeded
    - Return MLE verdict with usage stats
29. System Errors: Handle infrastructure failures:
    - Kubernetes node failures
    - Network partitions
    - Storage unavailability
30. Retry Logic: Implement intelligent retry for transient failures:
    - Exponential backoff with jitter
    - Maximum 3 retry attempts
    - Different handling for different error types

Security Features:
- Sandboxed Execution: Complete isolation from host system
- Network Isolation: No external network access during execution
- Filesystem Isolation: Read-only access to system files
- Resource Limits: Strict CPU, memory, and time constraints
- System Call Filtering: Block dangerous system calls
- Process Isolation: Prevent interference between executions

Performance Optimizations:
- Pod Reuse: Reuse execution pods for similar submissions
- Parallel Execution: Run multiple test cases concurrently when safe
- Resource Preallocation: Pre-warm execution environments
- Intelligent Scheduling: Optimize pod placement across nodes
- Result Caching: Cache results for identical submissions (contest mode disabled)

Monitoring & Observability:
- Execution Metrics: Track success rates, timing, resource usage
- Error Tracking: Categorize and monitor execution failures
- Performance Monitoring: Real-time execution performance dashboards
- Alerting: Automated alerts for execution service issues
- Audit Trail: Complete logging of all execution activities
```

### Performance & Monitoring Integration

#### Observability Stack Usage:
```
Application → Micrometer → Prometheus → Grafana → Alerting
     │            │           │           │         │
Business      Spring Boot   Metrics     Dashboards  PagerDuty
Metrics       Integration   Storage     + Alerts    + Slack

Zipkin → Jaeger → Service Map → Performance Analysis
  │        │          │              │
Traces   Aggregation  Visualization  Optimization
```

#### Key Metrics Tracked:
- **Latency**: P50, P95, P99 for all endpoints
- **Throughput**: RPS per service with scaling triggers  
- **Error Rates**: 4xx/5xx responses with alerting
- **Resource Usage**: CPU/Memory with auto-scaling
- **Business Metrics**: Submissions/min, contest participation
- **Cache Performance**: Hit ratios, eviction rates

### 2. User Management Service Flows

#### 2.1 Get User Profile (GET /api/v1/users/profile)
```
Step 1: Authentication & Authorization
Frontend → Kong → Auth Service (gRPC) → User Service
    │       │         │                     │
JWT Token  Extract   ValidateToken()    Get Profile
          + Validate   Response          from Cache/DB

Step 2: Data Retrieval with Caching
User Service → Redis → PostgreSQL → Response Assembly
     │         │         │             │
Check Cache  Cache Hit   Query User   Format Response
             Return     + Statistics   + Enrich Data

Detailed Process:
1. Kong extracts JWT from Authorization header
2. Kong calls Auth Service gRPC ValidateToken(token)
3. Auth Service verifies JWT signature + expiration
4. User Service receives validated user context
5. Check Redis cache: key="user_profile:{userId}"
6. If cache miss, query PostgreSQL with optimized query
7. Enrich profile with statistics (async calculation)
8. Cache result for 1 hour, return to client
```

#### 2.2 Update User Profile (PUT /api/v1/users/profile)
```
Frontend → Kong → Auth Service → User Service → PostgreSQL → Kafka
    │       │         │             │             │           │
Profile   JWT       Validate      Validate     Update      Publish
Data     Extract    Token         Input       Database     Event

Detailed Process:
1. Validate request payload (name, bio, preferences)
2. Check user permissions (can only update own profile)
3. Transaction: UPDATE user_profiles SET ... WHERE user_id = ?
4. Invalidate cached profile in Redis
5. Publish UserProfileUpdatedEvent to Kafka
6. Other services consume event for eventual consistency
7. Return updated profile data to client
```

#### 2.3 Get User Statistics (GET /api/v1/users/{userId}/stats)
```
Frontend → Kong → User Service → Redis → ClickHouse → Response
    │       │         │           │         │           │
User ID   Rate      Check Cache  Cache     Aggregate   Formatted
Request   Limit     Stats        Miss      Data        Statistics

Detailed Process:
1. Kong rate limiting: 1000 requests/hour per user
2. Check Redis cache: "user_stats:{userId}:{date}"
3. If cache miss, query ClickHouse for aggregated data:
   - Problems solved by difficulty
   - Submission success rate
   - Contest participation history
   - Language usage statistics
4. Cache results for 1 hour (stats don't change frequently)
5. Return formatted statistics to client
```

### 3. Problem Service Flows

#### 3.1 Get Problem List (GET /api/v1/problems)
```
Frontend → Kong → Problem Service → Redis → PostgreSQL + Elasticsearch
    │       │         │             │         │
Query     Rate      Parse Params   Check    Query with
Params    Limit     + Build Query  Cache    Pagination

Detailed Process:
1. Parse query parameters: page, size, difficulty, category, tags
2. Build cache key: "problems_list:{hash(params)}"
3. Check Redis cache for paginated results
4. If cache miss, query PostgreSQL with LIMIT/OFFSET
5. For each problem, check user's solve status
6. Enrich with metadata (like count, difficulty distribution)
7. Cache results for 10 minutes
8. Return paginated response with metadata
```

#### 3.2 Search Problems (GET /api/v1/problems/search)
```
Frontend → Kong → Problem Service → Redis → Elasticsearch → PostgreSQL
    │       │         │             │         │               │
Search    Rate      Parse Query    Cache     Full-text       User Data
Query     Limit     + Filters      Check     Search          Enrichment

Detailed Process:
1. Parse search query and filters (difficulty, tags, solved status)
2. Build Elasticsearch query with:
   - Multi-field search (title, description, tags)
   - Filters and aggregations
   - Highlighting for search terms
3. Check Redis cache: "search:{query_hash}"
4. Execute Elasticsearch query with pagination
5. For each result, add user-specific data (solved status)
6. Cache search results for 5 minutes
7. Return enriched search results with facets
```

#### 3.3 Create Problem (POST /api/v1/problems)
```
Frontend → Kong → Auth Service → Problem Service → PostgreSQL → Kafka
    │       │         │             │               │           │
Problem   JWT       Check Role    Validate        Store       Publish
Data     Validate   CREATOR       Content         Problem     Event

Detailed Process:
1. Kong validates JWT and extracts user claims
2. Auth Service verifies user has PROBLEM_CREATOR role
3. Problem Service validates problem content:
   - Title uniqueness
   - Description format
   - Test cases validity
   - Time/memory limits
4. Transaction: Insert into problems, test_cases tables
5. Trigger Elasticsearch indexing for search
6. Publish ProblemCreatedEvent to Kafka
7. Cache invalidation for problem lists
8. Return created problem with generated ID
```

### 4. Testcase Service Flows

#### 4.1 Get Test Cases (GET /api/v1/testcases/{problemId})
```
Frontend → Kong → Auth Service → Testcase Service → Redis → PostgreSQL
    │       │         │             │                 │         │
Problem   JWT       Check         Check              Cache     Encrypted
ID       Validate   Permissions   Access             Check     Test Cases

Detailed Process:
1. Validate user permissions (creator or admin only)
2. Check Redis cache: "testcases:{problemId}"
3. If cache miss, query PostgreSQL for encrypted test cases
4. Decrypt test cases in memory (AES-256-GCM)
5. Filter based on user role:
   - Creator: All test cases
   - Regular user: Only sample test cases
6. Cache decrypted test cases for 30 minutes
7. Return filtered test cases to client
```

#### 4.2 Add Test Case (POST /api/v1/testcases/{problemId})
```
Frontend → Kong → Auth Service → Testcase Service → PostgreSQL → Cache
    │       │         │             │                 │           │
Test Case JWT       Verify        Validate          Store       Invalidate
Data     Validate   Creator       + Encrypt         Encrypted   Cache

Detailed Process:
1. Verify user is problem creator or admin
2. Validate test case format and size limits
3. Check for duplicate test cases (hash comparison)
4. Encrypt test case data with problem-specific key
5. Transaction: INSERT into test_cases table
6. Invalidate related caches in Redis
7. Publish TestCaseAddedEvent to Kafka
8. Return success response with test case ID
```

### 5. Submission Service Flows

#### 5.1 Submit Solution (POST /api/v1/submissions) - DETAILED
```
Step 1: Request Validation & Rate Limiting
Frontend → HAProxy → Kong → Auth Service → Submission Service
    │         │       │         │             │
Code +      Load     Rate      Validate     Validate
Metadata    Balance  Limit     JWT Token    Request
            │        │         │            │
        SSL Term   10/min    Extract User   Code Size
        Health     per user   Context       < 100KB
        Check

Step 2: Pre-submission Checks
Submission Service → Redis → PostgreSQL → Problem Service (gRPC)
        │            │         │             │
    Check Last       Cache     User          Problem
    Submission       Check     Status        Validation
    Interval         │         │             │
                 User Quota  Contest       Time Limits
                            Status        Memory Limits

Step 3: Code Storage & Event Publishing
Submission Service → MongoDB → Kafka → Redis
        │            │         │       │
    Generate        Store      Publish Cache
    Submission      Source     Event   Status
    ID              Code       │       │
                              │    Status:
                           Build    QUEUED
                           Event

Step 4: Asynchronous Processing Chain
Kafka → Build Service → Execution Service → Result Processing
  │         │              │                 │
Event    Process         Execute           Update
Stream   Build           Code              Results

Detailed Process:
1. **Frontend Validation**: Syntax highlighting, basic validation
2. **HAProxy**: SSL termination, health checks, load balancing
3. **Kong Rate Limiting**: 10 submissions/minute per user
4. **Auth Service gRPC**: JWT validation, extract user context
5. **Submission Service Validation**:
   - Code size validation (< 100KB)
   - Language support check
   - Contest submission window validation
   - User quota check (daily submission limit)
6. **Problem Service gRPC**: Get problem constraints
7. **MongoDB Storage**: Store submission with PENDING status
8. **Kafka Event**: Publish SubmissionCreatedEvent
9. **Redis Cache**: Store submission status for real-time updates
10. **Response**: Return submission ID and QUEUED status

Build Service Processing (Async):
11. **Kafka Consumer**: Receive SubmissionCreatedEvent
12. **Docker Build**: Create isolated container
13. **Code Compilation**: Language-specific compilation
14. **Dependency Installation**: Cached dependencies
15. **Build Validation**: Syntax errors, security scan
16. **Kafka Event**: Publish BuildCompletedEvent
17. **Status Update**: Update submission status in MongoDB

Execution Service Processing (Async):
18. **Kafka Consumer**: Receive BuildCompletedEvent
19. **Kubernetes Job**: Create execution pod with resource limits
20. **Test Case Execution**: Run against all test cases
21. **Result Collection**: Gather execution metrics
22. **Kafka Event**: Publish ExecutionCompletedEvent
23. **Final Update**: Update submission with final results
```

#### 5.2 Get Submission Result (GET /api/v1/submissions/{submissionId}/result)
```
Frontend → Kong → Auth Service → Submission Service → Redis → MongoDB
    │       │         │             │                 │         │
Sub ID    JWT       Check          Check            Cache     Full
Request   Validate  Owner          Permission       Check     Result

Detailed Process:
1. Validate user can access submission (owner or admin)
2. Check Redis cache: "submission_result:{submissionId}"
3. If cache miss, query MongoDB for complete result
4. Format response with test case results, metrics
5. Cache result for 1 hour (results don't change)
6. Return detailed execution results
```

### 6. Build Service Flows (Internal)

#### 6.1 Process Build (Internal Kafka Consumer)
```
Kafka Event → Build Service → Docker → Maven/NPM → Security Scan → Result
     │            │           │         │            │            │
Submission      Parse        Create     Download     Vulnerability  Update
Created         Language     Container  Dependencies  Scan          Status

Detailed Process:
1. **Event Consumption**: Receive SubmissionCreatedEvent from Kafka
2. **Language Detection**: Determine build strategy (Java, Python, C++, etc.)
3. **Docker Container**: Spin up language-specific build container
4. **Dependency Resolution**: Download and cache dependencies
5. **Code Compilation**: Compile source code with error handling
6. **Security Scanning**: Check for malicious code patterns
7. **Artifact Creation**: Create executable artifact
8. **Resource Cleanup**: Cleanup temporary files and containers
9. **Event Publishing**: Publish BuildCompletedEvent with status
10. **Metrics Collection**: Record build time, success rate
```

### 7. Execution Service Flows (Internal)

#### 7.1 Execute Code (Internal Kafka Consumer)
```
Kafka Event → Execution Service → K8s Job → Test Execution → Result Collection
     │            │                 │          │               │
Build          Parse             Create      Run Against     Aggregate
Completed      Metadata          Pod         Test Cases      Results

Detailed Process:
1. **Event Consumption**: Receive BuildCompletedEvent
2. **Resource Allocation**: Create Kubernetes Job with limits:
   - CPU: 1 core max
   - Memory: 512MB max
   - Timeout: 30 seconds per test case
3. **Test Case Retrieval**: Get encrypted test cases from Testcase Service
4. **Sandboxed Execution**: Run code in isolated container
5. **Output Comparison**: Compare with expected outputs
6. **Metrics Collection**: Execution time, memory usage
7. **Result Aggregation**: Calculate final verdict
8. **Event Publishing**: Publish ExecutionCompletedEvent
9. **Resource Cleanup**: Clean up Kubernetes resources
```

### 8. Contest Service Flows

#### 8.1 Join Contest (POST /api/v1/contests/{contestId}/join)
```
Frontend → Kong → Auth Service → Contest Service → PostgreSQL → WebSocket
    │       │         │             │               │           │
Join      JWT       Validate      Check Contest    Add         Notify
Request   Validate  User          Status          Participant  Others

Detailed Process:
1. Validate contest is in ACTIVE status
2. Check user isn't already participating
3. Verify contest capacity not exceeded
4. Add participant record with join timestamp
5. Update contest participant count
6. Send WebSocket notification to other participants
7. Return participant details and contest info
```

#### 8.2 Live Contest Updates (WebSocket /api/v1/contests/{contestId}/live)
```
WebSocket Connection → Kong → Contest Service → Redis Pub/Sub → Real-time Updates
         │              │         │               │               │
    User Connects     Route      Maintain        Subscribe      Broadcast
    with JWT         WebSocket   Connection      to Events      to Users

Detailed Process:
1. **WebSocket Handshake**: Establish connection with JWT validation
2. **Connection Management**: Store connection in memory map
3. **Redis Subscription**: Subscribe to contest-specific channels
4. **Event Processing**: Handle submission events, ranking changes
5. **Real-time Broadcasting**: Send updates to connected clients
6. **Connection Cleanup**: Handle disconnections gracefully

Live Update Types:
- New submissions
- Ranking changes
- Problem clarifications
- Contest announcements
- Time remaining updates
```

## Performance Metrics & SLAs

### Service Performance Targets
| Service | Avg Latency | 95th Percentile | Max RPS | Error Rate |
|---------|-------------|-----------------|---------|------------|
| Auth Service | 20ms | 50ms | 30K | < 0.1% |
| User Service | 15ms | 40ms | 25K | < 0.1% |
| Problem Service | 25ms | 60ms | 35K | < 0.1% |
| Submission Service | 30ms | 80ms | 15K | < 0.5% |
| Contest Service | 10ms | 30ms | 20K | < 0.1% |

### System-wide SLAs
- **API Availability**: 99.9% (8.76 hours downtime/year)
- **Submission Processing**: 95% complete within 30 seconds
- **Real-time Updates**: < 200ms latency for contest updates
- **Cache Hit Ratio**: > 90% for frequently accessed data
- **Error Rate**: < 0.5% across all endpoints

## Performance & Scaling

### Auto-Scaling Strategy
- **Horizontal Pod Autoscaler**: CPU and memory-based scaling triggers
- **Custom Metrics Scaling**: RPS and business metrics-based scaling
- **Vertical Pod Autoscaler**: Automatic resource request optimization
- **Cluster Autoscaler**: Node-level scaling for resource optimization
- **Predictive Scaling**: ML-based scaling for anticipated load patterns

### Scaling Triggers
- **CPU Utilization**: Scale out at 70%, scale in at 30%
- **Memory Utilization**: Scale out at 80%, scale in at 40%
- **Request Rate**: Scale out at 1K RPS per instance
- **Queue Depth**: Scale out when Kafka lag > 1000 messages
- **Response Time**: Scale out when P95 latency > 100ms

#### Service Level Objectives (SLOs)
- **API Response Time**: 95th percentile < 100ms
- **Availability**: 99.9% uptime (8.76 hours downtime/year)
- **Submission Processing**: 99% completion within 30 seconds
- **Contest Real-time Updates**: < 200ms latency
- **Cache Hit Ratio**: > 95% for frequently accessed data

## API Endpoints

### Authentication Service Endpoints
```
POST   /api/v1/auth/register          - User registration
POST   /api/v1/auth/login             - User login  
POST   /api/v1/auth/logout            - User logout
POST   /api/v1/auth/refresh           - Refresh JWT token
POST   /api/v1/auth/forgot-password   - Password reset request
POST   /api/v1/auth/reset-password    - Reset password with token
GET    /api/v1/auth/verify/{token}    - Email verification
```

### User Management Service
```
GET    /api/v1/users/profile          - Get user profile
PUT    /api/v1/users/profile          - Update user profile
GET    /api/v1/users/{userId}/stats   - Get user statistics
GET    /api/v1/users/{userId}/submissions - Get user submission history
GET    /api/v1/users/leaderboard      - Get global leaderboard
GET    /api/v1/users/search           - Search users
```

### Problem Service
```
GET    /api/v1/problems               - List problems with pagination
GET    /api/v1/problems/{problemId}   - Get specific problem
GET    /api/v1/problems/search        - Search problems with filters
POST   /api/v1/problems               - Create new problem (Creator only)
PUT    /api/v1/problems/{problemId}   - Update problem (Creator only)
DELETE /api/v1/problems/{problemId}   - Delete problem (Creator only)
GET    /api/v1/problems/categories    - Get all categories
GET    /api/v1/problems/tags          - Get all tags
```

### Testcase Service
```
GET    /api/v1/testcases/{problemId}     - Get test cases for problem
POST   /api/v1/testcases/{problemId}     - Add test case (Creator only)
PUT    /api/v1/testcases/{testcaseId}    - Update test case
DELETE /api/v1/testcases/{testcaseId}    - Delete test case
POST   /api/v1/testcases/validate       - Validate test cases
```

### Submission Service
```
POST   /api/v1/submissions             - Submit solution
GET    /api/v1/submissions/{submissionId} - Get submission details
GET    /api/v1/submissions/{submissionId}/result - Get submission result
GET    /api/v1/submissions/user/{userId} - Get user submissions
GET    /api/v1/submissions/problem/{problemId} - Get problem submissions
DELETE /api/v1/submissions/{submissionId} - Delete submission
```

### Build Service (Internal)
```
POST   /internal/v1/build              - Build code (internal only)
GET    /internal/v1/build/{buildId}/status - Get build status
GET    /internal/v1/build/{buildId}/logs - Get build logs
POST   /internal/v1/build/validate     - Validate build environment
```

### Execution Service (Internal)
```
POST   /internal/v1/execute            - Execute code (internal only)
GET    /internal/v1/execute/{executionId}/status - Get execution status
GET    /internal/v1/execute/{executionId}/result - Get execution result
POST   /internal/v1/execute/test       - Test execution environment
```

### Contest Service
```
GET    /api/v1/contests                - List active contests
GET    /api/v1/contests/{contestId}    - Get contest details
POST   /api/v1/contests                - Create contest (Admin only)
PUT    /api/v1/contests/{contestId}    - Update contest
POST   /api/v1/contests/{contestId}/join - Join contest
GET    /api/v1/contests/{contestId}/leaderboard - Get contest leaderboard
GET    /api/v1/contests/{contestId}/problems - Get contest problems
POST   /api/v1/contests/{contestId}/submissions - Submit during contest
WebSocket /api/v1/contests/{contestId}/live - Live contest updates
```

---

## 🔧 **Comprehensive Technology Stack Explanation**

### **🚀 Application Layer Technologies**

#### **Spring Boot 3.2 + Java 21**
**Purpose**: Enterprise-grade microservices framework with modern Java features
**Why Used**: 
- Virtual Threads (Project Loom) for handling massive concurrency
- Reactive programming support for non-blocking I/O
- Extensive ecosystem and enterprise support
- Auto-configuration and production-ready features

**Specific Use Cases**:
```
✅ Used In:
├── All 13 microservices as the base framework
├── REST API endpoint creation and management
├── Dependency injection and configuration management
├── Health checks and metrics exposure (Actuator)
├── Database connectivity and transaction management
└── Security implementation (Spring Security)

📍 Key Tasks:
├── User authentication and authorization
├── Problem CRUD operations
├── Submission processing workflows
├── Contest management and real-time updates
├── Build and execution service orchestration
└── Notification and audit service operations
```

#### **Spring WebFlux (Reactive Streams)**
**Purpose**: Non-blocking, reactive web framework for high-concurrency applications
**Why Used**:
- Handle 100K+ concurrent connections per instance
- Backpressure handling for system stability
- Event-driven architecture support
- Memory efficiency compared to traditional servlet stack

**Specific Use Cases**:
```
✅ Used In:
├── All HTTP endpoints across all microservices
├── Real-time data streaming (contest updates)
├── High-throughput submission processing
├── Async communication between services
└── WebSocket connections for live updates

📍 Key Tasks:
├── Contest participant connections (15K concurrent)
├── Submission status streaming
├── Real-time leaderboard updates
├── Problem search with auto-complete
└── User activity feed processing
```

#### **gRPC with Protocol Buffers**
**Purpose**: High-performance, language-agnostic RPC framework for inter-service communication
**Why Used**:
- Binary protocol (faster than JSON/REST)
- Type-safe contracts with proto files
- Built-in load balancing and retries
- Streaming support for real-time data

**Specific Use Cases**:
```
✅ Used In:
├── Auth Service ↔ All other services (token validation)
├── Submission Service ↔ Problem Service (validation)
├── Submission Service ↔ Contest Service (eligibility check)
├── Execution Service ↔ Testcase Service (test case retrieval)
├── Build Service ↔ Security Scanner (code analysis)
└── User Service ↔ Problem Service (solved status)

📍 Key Tasks:
├── JWT token validation (30K+ calls/min)
├── Problem constraint retrieval
├── User permission checking
├── Test case decryption and delivery
└── Cross-service data consistency checks
```

### **🌐 API Gateway & Load Balancing Technologies**

#### **Kong Gateway**
**Purpose**: Cloud-native API gateway with plugin ecosystem
**Why Used**:
- Advanced rate limiting with Redis backend
- 100+ built-in plugins for extensibility
- JWT validation and OAuth2 support
- Circuit breaker and retry mechanisms

**Specific Use Cases**:
```
✅ Used In:
├── All external API requests (primary entry point)
├── Rate limiting enforcement (10 submissions/min/user)
├── JWT token validation and extraction
├── Request/response transformation
├── API analytics and monitoring
└── Circuit breaker for service protection

📍 Key Tasks:
├── User registration rate limiting (100/min/IP)
├── Contest submission throttling
├── API version management
├── Request routing to appropriate services
├── Security policy enforcement
└── Real-time traffic analytics
```

#### **HAProxy**
**Purpose**: High-performance Layer 4/7 load balancer and reverse proxy
**Why Used**:
- SSL termination and certificate management
- Health check-based load balancing
- Geographic routing capabilities
- Session affinity for stateful operations

**Specific Use Cases**:
```
✅ Used In:
├── SSL/TLS termination for all HTTPS traffic
├── Load balancing between Kong instances
├── Health check monitoring of backend services
├── Geographic traffic routing
├── Session persistence for WebSocket connections
└── Failover and redundancy management

📍 Key Tasks:
├── SSL certificate handling and renewal
├── Traffic distribution during contest spikes
├── Automatic failover during service outages
├── Connection pooling and optimization
└── Geographic routing for global users
```

#### **NGINX + CloudFlare CDN**
**Purpose**: Static content delivery and edge caching
**Why Used**:
- Global content distribution network
- Image optimization and compression
- DDoS protection and bot mitigation
- Edge caching for static assets

**Specific Use Cases**:
```
✅ Used In:
├── Static asset delivery (images, CSS, JS)
├── Problem statement images and diagrams
├── User profile avatars and assets
├── Contest banners and promotional content
├── Documentation and help pages
└── Mobile app asset delivery

📍 Key Tasks:
├── Problem images and examples caching
├── User avatar delivery optimization
├── Contest media asset distribution
├── API response caching (read-heavy endpoints)
└── Global latency reduction (<20ms)
```

### **💾 Data Storage Technologies**

#### **PostgreSQL (Clustered)**
**Purpose**: ACID-compliant relational database for structured data
**Why Used**:
- Strong consistency and data integrity
- Complex query support with joins
- Mature ecosystem and tooling
- Excellent performance for OLTP workloads

**Specific Use Cases**:
```
✅ Used In:
├── User profiles and authentication data
├── Problem metadata and categorization
├── Contest information and standings
├── Financial transactions and payments
├── User statistics and rankings
└── Audit logs and compliance data

📍 Key Tasks:
├── User registration and profile management
├── Problem categorization and tagging
├── Contest participant tracking
├── Payment processing and billing
├── Leaderboard calculation and caching
└── Referential integrity enforcement

📊 Schema Examples:
├── users (id, email, username, password_hash, created_at)
├── problems (id, title, description, difficulty, category_id)
├── contests (id, name, start_time, end_time, type)
├── submissions (id, user_id, problem_id, status, submitted_at)
└── user_statistics (user_id, problems_solved, contest_rating)

📈 Scaling Configuration:
├── Master-Replica Setup: 1 master + 3 read replicas
├── Connection Pooling: HikariCP (20 connections per service)
├── Read/Write Split: 80% reads → replicas, 20% writes → master
├── Backup Strategy: WAL-E with point-in-time recovery
└── High Availability: Automatic failover with 30s RTO
```

#### **MongoDB (Sharded)**
**Purpose**: Document database for high-write throughput and flexible schema
**Why Used**:
- High write performance for submissions
- Flexible schema for varied data structures
- Horizontal sharding for scalability
- JSON-like document storage

**Specific Use Cases**:
```
✅ Used In:
├── Source code storage (encrypted)
├── Build logs and compilation output
├── Execution results and test case outputs
├── System logs and debugging information
├── Binary file storage and metadata
└── Submission analytics and metrics

📍 Key Tasks:
├── Storing 25K+ submissions per minute
├── Build log aggregation and storage
├── Execution trace and performance data
├── Error logs and debugging information
├── File upload handling (code, assets)
└── Real-time submission status tracking

📊 Collection Examples:
├── submissions: {code, language, status, metadata}
├── build_logs: {submission_id, output, errors, duration}
├── execution_results: {submission_id, test_results, metrics}
├── file_uploads: {file_id, content, metadata, encryption}
└── system_logs: {timestamp, service, level, message}
```

#### **Redis Cluster**
**Purpose**: In-memory data structure store for caching and real-time data
**Why Used**:
- Sub-millisecond latency for cache operations
- Advanced data structures (sorted sets, hashes)
- Pub/Sub for real-time messaging
- Persistence options for durability

**Specific Use Cases**:
```
✅ Used In:
├── User session management
├── API response caching
├── Rate limiting counters
├── Real-time leaderboards (sorted sets)
├── Pub/Sub for live updates
└── Distributed locking

📍 Key Tasks:
├── JWT session storage (24hr TTL)
├── Problem list caching (10min TTL)
├── User profile caching (1hr TTL)
├── Contest leaderboard real-time updates
├── Rate limiting enforcement
└── WebSocket connection management

📊 Data Structure Usage:
├── Strings: session:{id} → session_data
├── Hashes: user:{id} → {name, email, stats}
├── Sorted Sets: leaderboard:{contest_id} → {user_id: score}
├── Lists: recent_submissions:{user_id} → [submission_ids]
└── Sets: active_users → {user_ids}
```

#### **Elasticsearch**
**Purpose**: Search engine and analytics platform for full-text search
**Why Used**:
- Advanced full-text search capabilities
- Real-time indexing and analytics
- Faceted search and aggregations
- Scalable distributed architecture

**Specific Use Cases**:
```
✅ Used In:
├── Problem search and discovery
├── User activity analytics
├── System log aggregation (ELK stack)
├── Code similarity search
├── Contest statistics and reporting
└── Auto-complete suggestions

📍 Key Tasks:
├── Problem title/description search
├── Tag-based problem filtering
├── User submission history search
├── System performance analytics
├── Error log analysis and alerting
└── Business intelligence and reporting

📊 Index Examples:
├── problems: {title, description, tags, difficulty}
├── user_activity: {user_id, action, timestamp, metadata}
├── system_logs: {service, level, message, timestamp}
├── submissions: {code_content, language, status, metrics}
└── contests: {name, participants, problems, statistics}
```

### **📨 Message Queue & Event Streaming Technologies**

#### **Apache Kafka**
**Purpose**: Distributed streaming platform for high-throughput event processing
**Why Used**:
- Handle 1M+ events per second
- Durable, persistent message storage
- Event sourcing and CQRS pattern support
- Stream processing capabilities

**Specific Use Cases**:
```
✅ Used In:
├── Submission processing pipeline
├── User activity event streaming
├── Contest event broadcasting
├── Build and execution coordination
├── Analytics data collection
└── Cross-service event notifications

📍 Key Tasks:
├── Submission events (25K/min)
├── User login/logout events (100K/min)
├── Contest participation events (50K/min)
├── Build completion notifications
├── Execution result publishing
└── System audit trail maintenance

📊 Topic Structure:
├── submission-events: {user_id, problem_id, code, timestamp}
├── user-events: {user_id, action, metadata, timestamp}
├── contest-events: {contest_id, type, participant_data}
├── build-events: {submission_id, status, logs, artifacts}
└── execution-events: {submission_id, results, metrics}
```

#### **RabbitMQ**
**Purpose**: Message broker for reliable, critical operations
**Why Used**:
- Guaranteed message delivery
- Complex routing patterns
- Dead letter queues for failure handling
- Priority queuing support

**Specific Use Cases**:
```
✅ Used In:
├── Email notification delivery
├── Payment processing workflows
├── Critical system alerts
├── Failed operation retry handling
├── User registration verification
└── Contest notification broadcasting

📍 Key Tasks:
├── Email verification sending (with retries)
├── Payment confirmation processing
├── Contest start/end notifications
├── System maintenance alerts
├── Failed submission reprocessing
└── User communication management

📊 Queue Configuration:
├── email.verification (DLQ: email.verification.failed)
├── payment.processing (Priority queue)
├── contest.notifications (Fanout exchange)
├── system.alerts (Topic exchange)
└── failed.operations (DLQ with manual intervention)
```

#### **Redis Pub/Sub**
**Purpose**: Real-time messaging for instant notifications
**Why Used**:
- Sub-second message delivery
- Pattern-based subscriptions
- Memory-based speed
- Simple publish/subscribe model

**Specific Use Cases**:
```
✅ Used In:
├── Contest live updates
├── WebSocket message broadcasting
├── Real-time leaderboard changes
├── System status notifications
├── Cache invalidation signals
└── Live user activity feeds

📍 Key Tasks:
├── Contest leaderboard live updates
├── Submission status real-time tracking
├── User online/offline status
├── System maintenance notifications
├── Cache invalidation coordination
└── WebSocket connection management

📊 Channel Patterns:
├── contest:{contest_id}:updates
├── user:{user_id}:notifications
├── leaderboard:{contest_id}:changes
├── system:maintenance:alerts
└── cache:invalidate:{service}
```

### **🏗️ Infrastructure & Orchestration Technologies**

#### **Kubernetes**
**Purpose**: Container orchestration platform for microservices deployment
**Why Used**:
- Automatic scaling and load balancing
- Service discovery and networking
- Rolling deployments and rollbacks
- Resource management and isolation

**Specific Use Cases**:
```
✅ Used In:
├── All microservice deployments
├── Code execution job scheduling
├── Database and cache deployments
├── Monitoring and logging infrastructure
├── CI/CD pipeline automation
└── Resource quota management

📍 Key Tasks:
├── Auto-scaling services based on load (HPA)
├── Managing execution pods for code testing
├── Service discovery and load balancing
├── Rolling updates with zero downtime
├── Resource allocation and limits enforcement
└── Cluster networking and security policies

📊 Resource Configuration:
├── Auth Service: 2-50 replicas, 500m CPU, 1Gi RAM
├── Submission Service: 2-60 replicas, 1 CPU, 2Gi RAM
├── Execution Jobs: Dynamic pods, 1 CPU, 512Mi RAM, 30s timeout
├── Build Service: 2-100 replicas, 2 CPU, 4Gi RAM
└── Database Pods: StatefulSets with persistent volumes

📋 Kubernetes Resource Policies:
├── Resource Quotas: namespace-level limits
├── Limit Ranges: default and max resource constraints
├── Pod Disruption Budgets: maintain availability during updates
├── Network Policies: service-to-service communication rules
└── Security Policies: Pod Security Standards enforcement
```

#### **Istio Service Mesh**
**Purpose**: Microservices communication and security management
**Why Used**:
- Automatic mTLS between services
- Traffic management and load balancing
- Observability and distributed tracing
- Security policy enforcement

**Specific Use Cases**:
```
✅ Used In:
├── Inter-service communication security
├── Traffic routing and canary deployments
├── Service-to-service authentication
├── Request tracing and monitoring
├── Circuit breaker implementation
└── Policy-based access control

📍 Key Tasks:
├── Automatic TLS certificate management
├── Service mesh observability
├── Canary deployments for new features
├── Traffic splitting for A/B testing
├── Service dependency visualization
└── Zero-trust security implementation
```

#### **Docker**
**Purpose**: Containerization for consistent deployment and isolation
**Why Used**:
- Consistent environment across dev/staging/prod
- Process isolation and security
- Language-specific runtime environments
- Resource control and monitoring

**Specific Use Cases**:
```
✅ Used In:
├── All microservice packaging and deployment
├── Code execution environment isolation
├── Build environment standardization
├── Database and cache containerization
├── CI/CD pipeline standardization
└── Development environment consistency

📍 Key Tasks:
├── Microservice containerization
├── Code execution sandboxing (Java, Python, C++, etc.)
├── Build environment isolation
├── Database deployment standardization
├── Development environment setup
└── Security boundary enforcement

📊 Container Images:
├── leetcode/auth-service:latest
├── leetcode/submission-service:latest
├── leetcode/execution-java:openjdk21
├── leetcode/execution-python:python3.11
├── leetcode/execution-cpp:gcc12
└── leetcode/build-service:latest
```

### **📊 Monitoring & Observability Technologies**

#### **Prometheus + Micrometer**
**Purpose**: Metrics collection and monitoring for system observability
**Why Used**:
- Time-series metrics storage
- Spring Boot integration via Micrometer
- Powerful query language (PromQL)
- Alert manager integration

**Specific Use Cases**:
```
✅ Used In:
├── Application performance monitoring
├── Infrastructure metrics collection
├── Business KPI tracking
├── Alert threshold management
├── Auto-scaling trigger metrics
└── SLA compliance monitoring

📍 Key Tasks:
├── API response time tracking (P50, P95, P99)
├── Request rate monitoring (RPS per service)
├── Error rate tracking (4xx, 5xx responses)
├── Resource utilization monitoring (CPU, memory)
├── Business metrics (submissions/min, active users)
└── Cache hit ratio monitoring

📊 Metrics Examples:
├── http_requests_total{service="auth", status="200"}
├── submission_processing_duration_seconds
├── active_contest_participants_gauge
├── cache_hit_ratio{cache="redis", type="user_profile"}
└── execution_jobs_running{language="java"}
```

#### **Grafana**
**Purpose**: Visualization and dashboarding for metrics and logs
**Why Used**:
- Real-time dashboard creation
- Multi-datasource support
- Alert visualization and management
- Business intelligence reporting

**Specific Use Cases**:
```
✅ Used In:
├── Real-time system monitoring dashboards
├── Business KPI visualization
├── Alert management and escalation
├── Performance trend analysis
├── Capacity planning insights
└── SLA compliance reporting

📍 Key Tasks:
├── System health monitoring dashboards
├── Contest participation analytics
├── Service performance visualization
├── Error rate and latency tracking
├── Resource utilization trending
└── Business metrics reporting

📊 Dashboard Categories:
├── System Overview: Overall system health and performance
├── Service Metrics: Individual microservice performance
├── Business KPIs: User engagement and platform usage
├── Infrastructure: Kubernetes cluster and resource usage
└── Security: Authentication, rate limiting, and threats
```

#### **Zipkin/Jaeger**
**Purpose**: Distributed tracing for microservices request tracking
**Why Used**:
- Request flow visualization across services
- Latency bottleneck identification
- Service dependency mapping
- Performance optimization insights

**Specific Use Cases**:
```
✅ Used In:
├── Cross-service request tracing
├── Performance bottleneck identification
├── Service dependency analysis
├── Error correlation across services
├── Latency optimization
└── Debugging complex workflows

📍 Key Tasks:
├── Submission processing pipeline tracing
├── Authentication flow analysis
├── Contest participation workflow tracking
├── Problem search performance analysis
├── Build and execution pipeline monitoring
└── User journey optimization

📊 Trace Examples:
├── User Login: Frontend → Kong → Auth → User Service
├── Submission: Frontend → Kong → Submission → Problem → Contest
├── Problem Search: Frontend → Kong → Problem → Elasticsearch
├── Contest Join: Frontend → Kong → Contest → User → Notification
└── Code Execution: Submission → Build → Security → Execution
```

#### **ELK Stack (Elasticsearch, Logstash, Kibana)**
**Purpose**: Centralized logging, log analysis, and visualization
**Why Used**:
- Centralized log aggregation
- Real-time log analysis
- Pattern detection and alerting
- Compliance and audit trail

**Specific Use Cases**:
```
✅ Used In:
├── Centralized application logging
├── System error analysis and debugging
├── Security audit trail maintenance
├── Performance log analysis
├── Compliance reporting
└── Operational intelligence

📍 Key Tasks:
├── Application error tracking and analysis
├── Security event monitoring and alerting
├── Performance log correlation
├── Audit trail maintenance for compliance
├── Operational troubleshooting
└── Business intelligence from logs

📊 Log Categories:
├── Application Logs: Service-specific application events
├── Access Logs: HTTP request/response logging
├── Error Logs: Exception and error tracking
├── Security Logs: Authentication and authorization events
└── Audit Logs: Compliance and regulatory tracking
```

### **🔒 Security & Additional Technologies**

#### **Spring Security + OAuth2 + JWT**
**Purpose**: Authentication, authorization, and security management
**Why Used**:
- Industry-standard security frameworks
- Token-based stateless authentication
- Role-based access control (RBAC)
- Integration with external identity providers

**Specific Use Cases**:
```
✅ Used In:
├── User authentication and session management
├── API endpoint protection
├── Role-based access control
├── Token validation and refresh
├── Security policy enforcement
└── Integration with external OAuth providers

📍 Key Tasks:
├── User login and registration security
├── API endpoint protection and authorization
├── Contest access control and permissions
├── Problem creation and modification permissions
├── Admin panel access control
└── Third-party integration authentication
```

#### **Encryption & Data Protection**
**Purpose**: Data security and privacy protection
**Why Used**:
- Regulatory compliance (GDPR, CCPA)
- Sensitive data protection
- Test case security
- User privacy maintenance

**Specific Use Cases**:
```
✅ Used In:
├── Source code encryption in MongoDB
├── Test case encryption and secure storage
├── User PII data protection
├── Payment information security
├── Session data encryption
└── Database encryption at rest

📍 Key Tasks:
├── Encrypting submitted source code
├── Protecting test case data from unauthorized access
├── Securing user personal information
├── Payment data encryption and tokenization
├── Secure session management
└── Compliance with data protection regulations
```

---

## 🎯 **Technology Integration Summary**

This competitive programming platform leverages **20+ cutting-edge technologies** in a carefully orchestrated architecture:

### **High-Throughput Processing Chain**:
```
User Request → HAProxy (SSL) → Kong (Gateway) → Spring Boot (Service) → 
PostgreSQL/MongoDB (Storage) → Kafka (Events) → Redis (Cache) → 
Kubernetes (Orchestration) → Prometheus (Monitoring)
```

### **Real-Time Processing Pipeline**:
```
Code Submission → Security Scan → Build (Docker) → Execute (K8s Jobs) → 
Results → Real-time Updates (Redis Pub/Sub) → WebSocket (Live Updates)
```

### **Data Flow Architecture**:
```
Write Path: API → Service → Database → Event (Kafka) → Cache Invalidation
Read Path: API → Cache (Redis) → Database (if miss) → Response
Search Path: API → Elasticsearch → Enrichment → Response
```

This system design provides **enterprise-grade scalability**, leveraging modern technologies with proper justification for each choice. The architecture supports horizontal scaling from 1 to 1M+ users while maintaining sub-100ms response times through intelligent caching, event-driven processing, and optimized data access patterns.

**Each technology serves a specific purpose** in creating a robust, scalable, and maintainable competitive programming platform that can handle massive concurrent loads while providing an excellent user experience.

---

## 🚨 **Disaster Recovery & Business Continuity**

### **Backup Strategy**
```
Database Backups:
├── PostgreSQL: WAL-E continuous archiving + daily full backups
├── MongoDB: Replica set with 3-node configuration + point-in-time recovery
├── Redis: RDB snapshots every 5 minutes + AOF persistence
├── Elasticsearch: Snapshot repository with S3 backend
└── File Storage: Multi-region S3 replication with versioning

Recovery Time Objectives (RTO):
├── Database: 30 seconds (automatic failover)
├── Application Services: 2 minutes (Kubernetes restart)
├── Full System: 15 minutes (disaster recovery)
└── Data Recovery: 4 hours (worst-case point-in-time restore)
```

### **Multi-Region Deployment**
```
Primary Region (US-East):
├── All services active with full traffic
├── Primary database instances
├── Real-time monitoring and alerting
└── 99.9% of traffic processing

Secondary Region (EU-West):
├── Read replicas and standby services
├── Disaster recovery backups
├── Reduced capacity for emergency scenarios
└── Automatic failover capabilities

Geographic Distribution:
├── CDN: Global edge locations (50+ cities)
├── DNS: Route 53 with health check-based routing
├── Load Balancing: Cross-region failover
└── Data Replication: Async replication with <5min lag
```

### **Incident Response Plan**
```
Severity Levels:
├── P1 (Critical): System down, data loss, security breach
├── P2 (High): Degraded performance, partial outage
├── P3 (Medium): Non-critical feature issues
└── P4 (Low): Cosmetic issues, documentation updates

Response Times:
├── P1: 15 minutes (24/7 on-call)
├── P2: 2 hours (business hours)
├── P3: 24 hours
└── P4: Next sprint planning

Escalation Matrix:
├── L1: DevOps engineer (initial response)
├── L2: Senior backend engineer (technical lead)
├── L3: Principal architect (system design expert)
└── L4: CTO/VP Engineering (business decisions)
```
