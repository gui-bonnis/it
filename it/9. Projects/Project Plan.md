# Content to create

## Table of Contents

- [Content to create](#content-to-create)
  - [Table of Contents](#table-of-contents)
  - [About ](#about-)
  - [1 Serie - Hello World ](#1-serie---hello-world-)
    - [Create Hello World API](#create-hello-world-api)
      - [Objective](#objective)
      - [Tasks](#tasks)
    - [Improve Hello World API Quality](#improve-hello-world-api-quality)
      - [Objective](#objective-1)
      - [Tasks](#tasks-1)
    - [Improve Hello World API Test](#improve-hello-world-api-test)
      - [Objective](#objective-2)
      - [Tasks](#tasks-2)
  - [Improve Hello World API Deploy](#improve-hello-world-api-deploy)
      - [Objective](#objective-3)
      - [Tasks](#tasks-3)
  - [Improve Hello World API Deployment Quality](#improve-hello-world-api-deployment-quality)
      - [Objective](#objective-4)
      - [Tasks](#tasks-4)
  - [2 Serie - Microservices](#2-serie---microservices)
    - [Create Microservices](#create-microservices)
      - [Objective](#objective-5)
      - [Tasks](#tasks-5)
    - [Database Migration](#database-migration)
      - [Objective](#objective-6)
      - [Tasks](#tasks-6)
    - [Messaging](#messaging)
      - [Objective](#objective-7)
      - [Tasks](#tasks-7)
    - [Metrics, Monitoring \& Tracing](#metrics-monitoring--tracing)
      - [Objective](#objective-8)
      - [Tasks](#tasks-8)
    - [Logging](#logging)
      - [Objective](#objective-9)
      - [Tasks](#tasks-9)
    - [Proxy, Gateway API, Coordination \& Service Discovery](#proxy-gateway-api-coordination--service-discovery)
      - [Objective](#objective-10)
      - [Tasks](#tasks-10)
    - [Security](#security)
      - [Objective](#objective-11)
      - [Tasks](#tasks-11)
    - [Cluster Scalability and Resilience](#cluster-scalability-and-resilience)
      - [Objective](#objective-12)
      - [Tasks](#tasks-12)
    - [Services Scalability and Resilience](#services-scalability-and-resilience)
      - [Objective](#objective-13)
      - [Tasks](#tasks-13)
    - [Releases](#releases)
      - [Objective](#objective-14)
      - [Tasks](#tasks-14)
  - [3 Serie - Automation](#3-serie---automation)
    - [Create Infrastructure as Code](#create-infrastructure-as-code)
      - [Objective](#objective-15)
      - [Tasks](#tasks-15)
    - [Improve IaC](#improve-iac)
      - [Objective](#objective-16)
      - [Tasks](#tasks-16)
  - [4 Serie - Software Architecture](#4-serie---software-architecture)
    - [Java Features](#java-features)
      - [Objective](#objective-17)
      - [Tasks](#tasks-17)
    - [Java Reactive](#java-reactive)
      - [Objective](#objective-18)
      - [Tasks](#tasks-18)
    - [Design Patterns](#design-patterns)
      - [Objective](#objective-19)
      - [Tasks](#tasks-19)
    - [Achitectural Patterns](#achitectural-patterns)
      - [Objective](#objective-20)
      - [Tasks](#tasks-20)
  - [5 Serie - Frameworks](#5-serie---frameworks)
    - [Springboot](#springboot)
      - [Objective](#objective-21)
      - [Tasks](#tasks-21)
    - [Quarkus](#quarkus)
      - [Objective](#objective-22)
      - [Tasks](#tasks-22)
    - [Coding](#coding)
      - [Objective](#objective-23)
      - [Tasks](#tasks-23)
    - [RestFULL API](#restfull-api)
      - [Objective](#objective-24)
      - [Tasks](#tasks-24)
    - [GraphQL API](#graphql-api)
      - [Objective](#objective-25)
      - [Tasks](#tasks-25)
    - [WebSocket API](#websocket-api)
      - [Objective](#objective-26)
      - [Tasks](#tasks-26)
    - [Testing](#testing)
      - [Objective](#objective-27)
      - [Tasks](#tasks-27)
  - [6 Serie - Cloud Architecture](#6-serie---cloud-architecture)
    - [Design Patterns](#design-patterns-1)
      - [Objective](#objective-28)
      - [Tasks](#tasks-28)
    - [Azure](#azure)
      - [Objective](#objective-29)
      - [Tasks](#tasks-29)
    - [AWS](#aws)
      - [Objective](#objective-30)
      - [Tasks](#tasks-30)
    - [GCP](#gcp)
      - [Objective](#objective-31)
      - [Tasks](#tasks-31)
  - [7 Serie - Solution Architecture](#7-serie---solution-architecture)
    - [Solution Quality](#solution-quality)
      - [Objective](#objective-32)
      - [Tasks](#tasks-32)
  - [8 Serie - CNCF Services](#8-serie---cncf-services)
    - [Cloud Container](#cloud-container)
      - [Objective](#objective-33)
      - [Tasks](#tasks-33)
    - [Cloud Networking](#cloud-networking)
      - [Objective](#objective-34)
      - [Tasks](#tasks-34)
    - [Cloud Storage](#cloud-storage)
      - [Objective](#objective-35)
      - [Tasks](#tasks-35)
  - [9 Serie - Serveless](#9-serie---serveless)
      - [Objective](#objective-36)
      - [Tasks](#tasks-36)
  - [10 Serie - Data Processing](#10-serie---data-processing)
      - [Objective](#objective-37)
      - [Tasks](#tasks-37)

## About <a name = "about"></a>

## 1 Serie - Hello World <a name = "1SerieHelloWorld"></a>

### Create Hello World API<a name = "create-hello-world-api"></a>

#### Objective
Create and Deploy Hello World API at Kubernetes Clusters

#### Tasks
1. Create Quarkus Web API with Gradle
2. Create Kubernetes Cluster with Kind
3. Install Gitlab
4. Create Gitlab CICD Pipeline using Gitlab
5. Deploy API into the same Kubernetes Cluster
6. Deploy API into another Kubernetes Cluster


### Improve Hello World API Quality<a name = "improve-hello-world-api-quality"></a>

#### Objective
Improve Hello World API Quality through CICD Pipeline check analysis

#### Tasks
1. Create Quarkus Web API with Gradle
2. Create Kubernetes Cluster with Kind
3. Gitlab
3.1. Install Gitlab
3.2. Create Gitlab Pipeline using Gitlab
4. SonarQube
4.1. Install SonarQube
4.2. Update Gitlab Pipeline to validate code Quality
5. SAST
5.1. Install SAST
5.2. Update Gitlab Pipeline to validate code Quality
6. DAST
6.1. Install DAST
6.2. Update Gitlab Pipeline to validate code Quality
7. OWASP
7.1. Install OWASP
7.2. Update Gitlab Pipeline to validate code Quality
8. CheckMarx OR Similar
8.1. Install CheckMarx
8.2. Update Gitlab Pipeline to validate code Quality


### Improve Hello World API Test<a name = "improve-hello-world-api-test"></a>

#### Objective
Improve Hello World API Quality through CICD Pipeline test analysis

#### Tasks
1. Create Quarkus Web API with Gradle
2. Create Kubernetes Cluster with Kind
3. Gitlab
3.1. Install Gitlab
3.2. Create Gitlab Pipeline using Gitlab
4. Test Pyramid (Mockito)
4.1. Unit Test
4.2. Component Test
4.3. Contract Test
4.4. Integration Test
4.5. End to End Test
5. Test Pyramid (JUnit)
5.1. Unit Test
5.2. Component Test
5.3. Contract Test
5.4. Integration Test
5.5. End to End Test
7. Performance test (JMeter)
8. Security test ?


## Improve Hello World API Deploy<a name = "improve-hello-world-api-deploy"></a>

#### Objective
Improve Hello World API Deploy through CICD Pipeline

#### Tasks
1. Create Quarkus Web API with Gradle
2. Create Kubernetes Cluster with Kind
3. Gitlab
3.1. Install Gitlab
4. Continuous Deployment (Flux)
5. Continuous Deployment (Argo)
6. Continuous Deployment (Helm)
7. Continuous Deployment (Kustomize)
8. Docker Registry (Harbor)
9. Continuous Deployment (Jenkins)


## Improve Hello World API Deployment Quality<a name = "improve-hello-world-api-deployment-quality"></a>

#### Objective
Improve Hello World API Deployment Quality through CICD Pipeline

#### Tasks
1. Create Quarkus Web API with Gradle
2. Create Kubernetes Cluster with Kind
3. Gitlab
3.1. Install Gitlab
4. Kubescan
5. Datree
6. Trivy
7. OPA
8. Kubesec


## 2 Serie - Microservices<a name = "2SerieMicroservices"></a>

### Create Microservices<a name = "2SerieMicroservicesProject1"></a>

#### Objective
Create and Deploy Microservices with Database

#### Tasks
1. Create Quarkus Web API with Gradle
2. Create Kubernetes Cluster with Kind
3. Gitlab
3.1. Install Gitlab
3.2 Create Gitlab Pipeline using Gitlab
4. Relational Database (Postgres)
4.1. Install
4.2. Deploy 
4.3. Configure
4.4. Update pipeline
5. NoSQL (MongoDB)
5.1. Install
5.2. Deploy 
5.3. Configure
5.4. Update pipeline
6. Memory Database (Redis)
6.1. Install
6.2. Deploy 
6.3. Configure
6.4. Update pipeline

### Database Migration<a name = "2SerieMicroservicesProject2"></a>

#### Objective
Improve Microservices Deployment with Database Migration

#### Tasks
Database migration (liquibase)
Database migration (flyway)

### Messaging<a name = "2SerieMicroservicesProject3"></a>

#### Objective
Create and Deploy Microservices with Messaging

#### Tasks
1. Create Quarkus Web API with Gradle
2. Create Kubernetes Cluster with Kind
3. Gitlab
3.1. Install Gitlab
3.2 Create Gitlab Pipeline using Gitlab
4. Messaging (Kafka)
4.1. Install
4.2. Deploy 
4.3. Configure
4.4. Update pipeline
5. Messaging (RabbitMQ)
5.1. Install
5.2. Deploy 
5.3. Configure
5.4. Update pipeline
5. Messaging (Cloud Events)
5.1. Install
5.2. Deploy 
5.3. Configure
5.4. Update pipeline


### Metrics, Monitoring & Tracing<a name = "metrics-monitoring--tracing"></a>

#### Objective
Improve microservice Quality through Metrics, Monitoring & Tracing

#### Tasks
1. Create Quarkus Microservices
2. Create Kubernetes Cluster with Kind
3. Gitlab
3.1. Install Gitlab
3.2. Create Gitlab Pipeline using Gitlab
4. Memory Database (Redis)
4.1. Install
4.2. Deploy 
4.3. Configure
4.4. Update pipeline
5. Monitoring & Tracing (Istio)
5.1. Install
5.2. Deploy Prometheus
5.3. Deploy Graphana
5.4. Deploy Istio
5.5. Configure
6. Monitoring & Tracing (Linkerd)
6.1. Install
6.2. Deploy Prometheus
6.3. Deploy Kiali
6.4. Deploy Linkerd
6.5. Configure
7. Metrics (Open Telemetry)


### Logging<a name = "2SerieMicroservicesProject5"></a>

#### Objective
Improve microservice Quality through Logging

#### Tasks
1. Create Quarkus Microservices
2. Create Kubernetes Cluster with Kind
3. Gitlab
3.1. Install Gitlab
3.2. Create Gitlab Pipeline using Gitlab
4. Memory Database (Redis)
4.1. Install
4.2. Deploy 
4.3. Configure
4.4. Update pipeline
5. Logging (Elastic Search)
5.1. Install
5.2. Deploy Elastic Search
5.3. Deploy Kibana
5.5. Configure
6. Logging (FluentD)
6.1. Install
6.2. Deploy Prometheus
6.3. Deploy Graphana
6.4. Deploy FluentD
6.5. Configure
7. Logging (Graylog)
7.1. Install
7.4. Deploy Graylog
7.5. Configure
8. Logging (Datadog)
8.1. Install
8.4. Deploy Datadog
8.5. Configure

### Proxy, Gateway API, Coordination & Service Discovery<a name = "2SerieMicroservicesProject6"></a>

#### Objective
Improve microservice Quality through Proxy, Gateway API, Coordination & Service Discovery

#### Tasks
1. Create Quarkus Web API with Gradle
2. Create Kubernetes Cluster with Kind
3. Gitlab
3.1. Install Gitlab
3.2. Create Gitlab Pipeline using Gitlab
4. Memory Database (Redis)
4.1. Install
4.2. Deploy 
4.3. Configure
4.4. Update pipeline
5. Load balance ()
6. Ingress or gateway
6.1 Core DNS
6.2 Emissary Ingress
6.3 Nginx
6.4 Envoy

### Security<a name = "2SerieMicroservicesProject7"></a>

#### Objective
Improve Microservices Security with Key Management

#### Tasks
- Database secrets
- Message secrets
- Spiffe
- Valt


### Cluster Scalability and Resilience<a name = "2SerieMicroservicesProject8"></a>

#### Objective
Improve Microservices Cluster Scalability and Resilience

#### Tasks
Kubernetes Probes
Messaging Scaling (Keda)

### Services Scalability and Resilience<a name = "2SerieMicroservicesProject9"></a>

#### Objective
Improve Services Scalability and Resilience

#### Tasks
- HA (Postgres)
- HA (Mongo)
- HA (Redis)
- HA (Elastic Search)
- HA (Kafka)
- HA (RabitMQ)
- HA (Valt)


### Releases<a name = "2SerieMicroservicesProject10"></a>

#### Objective
Improve Microservices Release

#### Tasks
- Rolling Deploy
- Blue Green
- Canarie Release (Istio)
- Feature Flagging (Open Feature)


## 3 Serie - Automation<a name = "3SerieAutomation"></a>

### Create Infrastructure as Code<a name = "3SerieAutomationProject1"></a>

#### Objective
Create Infrastructure as Code (IaC)

#### Tasks
- Terraform
  - Local
  - AWS
  - GCP
  - AZURE
- Cloud Custodian
  - Local
  - AWS
  - GCP
  - AZURE
- KubeEdge
  - Local
  - AWS
  - GCP
  - AZURE

### Improve IaC<a name = "3SerieAutomationProject2"></a>

#### Objective
Improve Infrastructure as Code (IaC)

#### Tasks
- Centralized Police (Open Policy Agent)
- Centralized IAM (KeyCloak)
- Centralized Infrastructure Management (Portainer)
- Centralized Infrastructure Management (Rancher)
- Centralized Development Management (Backstage)


## 4 Serie - Software Architecture<a name = "4SerieSoftwareArchitecture"></a>

### Java Features<a name = "4SerieSoftwareArchitectureProject1"></a>
#### Objective
Create Java Features

#### Tasks
Show case all Java 8, 11, 17, 21 Features


### Java Reactive<a name = "4SerieSoftwareArchitectureProject2"></a>

#### Objective
Create Java Reactive Features

#### Tasks
Show case all Java Reactive Features


### Design Patterns<a name = "4SerieSoftwareArchitectureProject3"></a>

#### Objective
Create Design Patterns

#### Tasks
All Design Patterns Using Java

### Achitectural Patterns<a name = "4SerieSoftwareArchitectureProject4"></a>

#### Objective
Create Architectural Patterns

#### Tasks
- DRY
- KISS
- SOLID
- DDD
- Clean
- Hexagonal
- CQRS
- Event Sourcing
- Outbox

## 5 Serie - Frameworks<a name = "5SerieFrameworks"></a>

### Springboot<a name = "5SerieFrameworksProject1"></a>

#### Objective
Create SpringBoot Features

#### Tasks
Show case Spring boot


### Quarkus<a name = "5SerieFrameworksProject2"></a>

#### Objective
Create Quarkus Features

#### Tasks
Show case Quarkus

### Coding<a name = "5SerieFrameworksProject3"></a>

#### Objective
Create Coding

#### Tasks
- Logging
- Versioning
- Documentation
- Security (JWT, OAUTH2)
- JPA, JDBC
- Configuration (Application.yml)?


### RestFULL API<a name = "5SerieFrameworksProject4"></a>

#### Objective
Create RESTFULL API

#### Tasks
- All Verbs
- Documentation
- Security (JWT, OAUTH2)
- Error Handling


### GraphQL API<a name = "5SerieFrameworksProject5"></a>

#### Objective
Create GraphQL API

#### Tasks
- Documentation
- Security (JWT, OAUTH2)
- Error Handling


### WebSocket API<a name = "5SerieFrameworksProject6"></a>

#### Objective
Create WebSocket API

#### Tasks
- Documentation
- Security (JWT, OAUTH2)
- Error Handling


### Testing<a name = "5SerieFrameworksProject7"></a>

#### Objective
Create Testing Automation

#### Tasks
- Selenium
- TestContainers
- Automation
- Fluent Assertions
- Mutation test


## 6 Serie - Cloud Architecture<a name = "6SerieCloudArchitecture"></a>

### Design Patterns<a name = "6SerieSoftwareArchitectureProject1"></a>

#### Objective
Create Cloud Design Patterns

#### Tasks
- Config
- Load Balance
- Integration Patterns:
  - Handling multiple reactive microservices communication efficiently
  - Gateway Aggregator Pattern: Reducing network latency and acting as a facade for complex backend services
  - Scatter Gather Pattern: Routing requests to multiple backend services and aggregating their responses
  - Orchestrator Pattern (Saga - for parallel workflow): Coordinating multiple backend services in complex workflows
  - Orchestrator Pattern (Saga - for sequential workflow): Managing sequential workflows by replacing chained microservice calls
  - Splitter Pattern: Processing individual elements in a list of repeating elements
- Resilient Patterns:
  - Building robust and resilient reactive microservices
  - Timeout Pattern: Handling unresponsive remote services by setting appropriate timeouts
  - Retry Pattern: Dealing with intermittent network or remote service issues by retrying failed requests
  - Circuit Breaker Pattern: Protecting services and meeting SLAs when dependent remote services are unhealthy or unreachable
  - Rate Limiter Pattern: Safeguarding services from DDoS attacks and controlling network call limits
  - Bulkhead Pattern: Allocating resources based on priority to prevent one feature's failure from affecting the entire application
- Coreography Pattern (Saga - for parallel workflow)
- Side Car
- Anti Corruption
- BFF
- Trottling
- Rate Limiting
- Retry Pattern

### Azure<a name = "6SerieSoftwareArchitectureProject2"></a>

#### Objective
Create Azure Cloud

#### Tasks
All Azure components


### AWS<a name = "6SerieSoftwareArchitectureProject3"></a>

#### Objective
Create AWS Cloud

#### Tasks
All AWS components

### GCP<a name = "6SerieSoftwareArchitectureProject4"></a>

#### Objective
Create GCP Cloud

#### Tasks
All GCP components


## 7 Serie - Solution Architecture<a name = "7SerieSolutionArchitecture"></a>

### Solution Quality<a name = "7SerieSolutionArchitectureProject1"></a>
#### Objective
Improve Solution Quality

#### Tasks
Chaos Engineering (Litmus Chaos)
Metadate Management (Datahub)


## 8 Serie - CNCF Services<a name = "8SerieCNCFServices"></a>

### Cloud Container<a name = "8SerieCNCFServicesProject1"></a>
#### Objective
Improve Microservices Storage with Container

#### Tasks
ContainerD


### Cloud Networking<a name = "8SerieCNCFServicesProject2"></a>

#### Objective
Improve Microservices Networking with Cloud Native Network

#### Tasks
Cilium


### Cloud Storage<a name = "8SerieCNCFServicesProject1"></a>

#### Objective
Improve Microservices Storage with Cloud Storage

#### Tasks
Rook



## 9 Serie - Serveless<a name = "9SerieServeless"></a>

#### Objective
Create Serveless

#### Tasks
All Serveless components and configurations


## 10 Serie - Data Processing<a name = "10SerieDataProcessing"></a>

#### Objective
Create Data Processing Solution

#### Tasks




TOOLS
Docker
