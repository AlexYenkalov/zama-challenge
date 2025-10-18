# Decision Summary

## Table of Contents

### 1. Executive Summary
- [Selected Approach](#selected-approach)
- [Rationale](#rationale)
- [Key Benefits](#key-benefits)
- [Accepted Trade-offs](#accepted-trade-offs)

### 2. Recommended Technology Stack
- [API Architecture](#api-architecture)
- [Data Storage](#data-storage)
- [Message Queuing](#message-queuing)
- [Compute Infrastructure](#compute-infrastructure)
- [Blockchain Integration](#blockchain-integration)
- [Authentication & Authorization](#authentication--authorization)
- [Monitoring & Observability](#monitoring--observability)
- [Billing System](#billing-system)

---

## Executive Summary

### Selected Approach
**Selected Approach**: *RESTful Serverless API with SSE/Webhooks using Cloud-Native Managed Services*

### Rationale
Based on our assumptions of no pre-existing infrastructure, no deep infrastructure expertise, budget minimization, and fast-paced environment with low operational overhead, we recommend a cloud-native approach using managed services.

### Key Benefits
- **Low Operational Overhead**: Managed services reduce maintenance burden
- **Cost Optimization**: Pay-as-you-go model aligns with budget constraints
- **No Infrastructure Expertise Required**: Managed services handle complexity
- **Fast Implementation**: Pre-built services accelerate development
- **Built-in Reliability**: Cloud providers offer 99.9%+ SLA guarantees
- **Automatic Scaling**: Handles spiky/unpredictable load patterns
- **At-Least-Once Delivery**: AWS EventBridge/SNS/SQS support this requirement
- **Idempotency Support**: Application-level idempotency implementation

### Accepted Trade-offs
- **Vendor Lock-in**: Reduced portability across cloud providers
- **Limited Customization**: Less control over infrastructure configuration
- **Ongoing Costs**: Operational expenses vs. upfront CapEx
- **Dependency Risk**: Reliance on cloud provider availability

## Recommended Technology Stack

### API Architecture
- **RESTful Serverless API**
  - Kong *(api gateway, rate limiting, abuse protection)*
  - AWS Lambda *(api endpoints with business logic)*
- **Notification Mechanisms**
  - Server-Sent Events *(real-time updates for web clients)*
  - Webhooks *(event-driven notifications for integrations)*

### Data Storage
- **Primary**:
  - Managed MongoDB: MongoDB Atlas *(jobs)*
  - Managed Redis: Upstash *(cache: quotas, jobs)*
  - AWS S3 IA/Glacier *(archives: logs, metrics)*

### Message Queuing
- **Primary**:
  - AWS SNS FIFO + AWS SQS FIFO *(job pipeline events)*
- **Upgrade Path**:
  - AWS SNS -> AWS EventBridge *(advanced event routing)*
  - Add Kinesis Data Streams *(high throughput, real-time processing, even-sourcing)*

### Compute Infrastructure
- **Primary**:
  - AWS Lambda *(short runs e.g. handler for job creation, etc)*
  - AWS EC2 Spot *(long runs e.g. handlers for job processing, etc)*
- **Upgrade Path**:
  - AWS Fargate Spot *(advanced orchestration & minimal operational effort)*
  - AWS ECS with EC2 Spot *(advanced control & cost optimization)*

### Blockchain Integration
- **Primary**:
  - Infura
- **Upgrade Path**:
  - Add another RPC as backup *(fault tollerance)*
  - Add Alchemy *(webhook support)*
  - Add QuickNode *(lower latency)*

### Authentication & Authorization
- **Primary**:
  - Auth0
    - OIDC/OAuth2.0
      - PKCE flow *(single page appliations)*
      - client_credentials flow *(machine-to-machine)*
    - MFA
      - Passkeys *(primary mfa)*
      - TOTP *(secondary mfa)*
- **Upgrade Path**:
  - BetterAuth (OIDC/OAuth2.0 + Passkeys + TOTP) *(open source)*
    - Requires DBs for auth data persistence
      - Managed PostgreSQL: Supabase

### Monitoring & Observability
- **Primary**:
  - OpenTelemetry *(standardized instrumentation, redaction for sensitive data)*
  - AWS CloudWatch *(logs, metrics, alerts, dashboards)*
  - AWS SNS *(notifications, automation)*
  - AWS Cost Explorer *(dashboards, alerts)*
- **Upgrade Path**:
  - Extend with:
    - AWS S3 IA/Glacier *(data retention)*
    - AWS X-Ray *(tracing)*
    - AWS CloudTrail *(security auditing)*
    - AWS OpenSearch *(advances log analytics)*
    - AWS Managed Grafana *(advances dashboards)*
  - Migrate to:
    - DataDog *(less ops, higher cost @ high load)* 
    - ELK + Jaeger + Prometheus + Grafana *(more ops, lower cost @ high load)* 

### Billing System
- **Primary**
  - Lago *(open source, written in golang, mature docs)*
- **Upgrade Path**:
  - **Build** Own simple service
  - **Buy**: Lago (managed), Alguna, Maxio, 


<!-- **Enhanced Decision Criteria**:
- **Performance Requirements**: [Latency, throughput, resource usage requirements]
- **Scalability Needs**: [Connection limits, horizontal scaling requirements]
- **Real-time Requirements**: [Event ordering, state management, network resilience needs]
- **Operational Complexity**: [Maintenance, monitoring, debugging requirements]
- **Security Requirements**: [Authentication, authorization, compliance needs]
- **Cost Considerations**: [Infrastructure, development, operational costs]
- **Team Expertise**: [Existing skills, learning curve, training requirements] -->
