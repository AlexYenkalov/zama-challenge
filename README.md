# Table of Contents

## Overview
This document provides a comprehensive overview of all Architecture Decision Records (ADRs) for the Zama Jobs Platform.

---

## ADR Documents

### 01. [Problem Statement](./01-problem-statement.md)
**Core Business Need & Challenge Context**

- **Executive Summary**: Core business need, challenge context, key constraints
- **Requirements Legend**: Requirements classification system
- **Functional Requirements**: Core off-chain and on-chain operations
- **Non-Functional Requirements**: Security, performance, reliability, compliance

### 02. [Goals & Success Criteria](./02-goals-and-success-criteria.md)
**Objectives & Success Metrics**

- **Primary Objectives**
- **Success Metrics & KPIs**

### 03. [Comparative Analysis](./03-comparative-analysis.md)
**Technical Options Analysis**

- **Transport Layer**: REST, SSE, WebSockets, Webhooks, gRPC, MQTT
- **API Architecture**: RESTful, GraphQL with various transport options
- **Job Processing**: Cloud services, Redis, Database, RabbitMQ, Kafka, Pulsar, NATS
- **Blockchain Integration**: Transaction creation and event handling patterns

### 04. [Questions](./04-questions.md)
**Assumptions & Stakeholder Input**

- **Use Case Questions**: System use cases and exclusions
- **Infrastructure & Expertise**: Existing infrastructure, financial constraints, multi-cloud strategy
- **Architecture & Design**: Simplicity vs performance, scale, operational constraints
- **Security & Compliance**: Security boundaries, compliance requirements, data privacy
- **Performance & Scale**: Throughput, success rates, processing delays
- **Business & Project**: Market positioning, customer needs, competitive analysis
- **Implementation & Operations**: Development approach, timeline, resource requirements

### 05. [Decision Summary](./05-decision-summary.md)
**Selected Approach & Technology Stack**

- **Executive Summary**: Selected approach, rationale, key benefits, accepted trade-offs
- **Recommended Technology Stack**: API architecture, data storage, message queuing, compute infrastructure, blockchain integration, authentication & authorization, monitoring & observability, billing system

### 06. [API Governance](./06-api-governance.md)
**API Shape, Versioning, Error Handling & Idempotency**

- **API Shape & Endpoints**: System, Jobs, Users, Webhook Management, Billing, Analytics
- **Versioning Strategy**: Path-based versioning with deprecation notice
- **Error Model**: RFC7807 Problem+JSON implementation
- **Idempotency Strategy**: Client-controlled with UUID

### 07. [Platform Policies](./07-platform-policies.md)
**Authentication, Authorization, Rate Limiting & Security**

- **Authentication**: OIDC/OAuth 2.0 with MFA (Passkeys, TOTP)
- **Authorization**: RBAC with OAuth 2.0 scopes + tenant isolation
- **Rate Limiting & Quotas**: Kong Gateway with multi-layer protection
- **Abuse Protection**: Cloudflare WAF with multi-layered defense
- **Content Security**: Multi-layered content security with edge validation

### 08. [Metering Logic](./08-metering-logic.md)
**Usage Events, Billing System & Invoice Mapping**

- **Usage Events**: Hybrid approach with job lifecycle and compute metrics
- **Billing System**: Lago open-source billing for flexible experimentation
- **Event Processing**: Hybrid event processing with real-time billing
- **Invoice Mapping**: Tiered pricing with volume discounts

### 09. [Data Models](./09-data-models.md)
**Job Entities, Tenant Management & Database Architecture**

- **Job Entities**: Core job data models and TypeScript interfaces
- **Tenant, User, and Pricing Plan Entities**: Multi-tenant and billing models
- **Billing Metric Types**: Usage tracking and billing events
- **Database Architecture**: Hybrid multi-database architecture (PostgreSQL, MongoDB, Redis)

### 10. [API Handler Logic](./10-api-handler-logic.md)
**Handler Execution Flows & Performance**

- **Execution Duration**: Performance expectations for each handler type
- **Handler Execution Flows**: Job creation, processing, confirmation, and reconciliation flows
- **Event-driven Architecture**: AWS EventBridge for reliable message publishing
- **Error Handling**: Context-specific retry policies

### 11. [On-Chain Interface](./11-on-chain-nterface.md)
**EVM Smart Contract Design & Deployment**

- **Contract Upgradability**: Trustless vs upgradeable analysis
- **Security Considerations**: Access control patterns and security trade-offs
- **Smart Contract Logic**: Event-only, Event & Mapping, Event & Struct, Batch confirmation implementations
- **Multi-Chain Cost Analysis**: Cost comparison across 10 blockchain networks

### 12. [Reliability](./12-reliability.md)
**SLOs, Error Budgets, Incident Response & Monitoring**

- **Service Level Objectives**: 99.9% for POST/PUT/DELETE, 99.95% for GET operations
- **Error Budget Policy**: Moderate approach for new services with learning-oriented burn rates
- **Incident Response Procedures**: 4-level escalation with comprehensive trigger combinations
- **Monitoring and Observability**: API latency breakdown and business criticality analysis
- **Alerting**: 30+ metrics with appropriate thresholds

### 13. [Security](./13-security.md)
**Authentication & MFA Options Analysis**

- **Authentication & Authorization Options**: Comparison across 5 different approaches
- **Multi-Factor Authentication Options**: Comparison across 6 different MFA methods
- **Decision References**: Links to platform policies document for actual decisions

---

## Document Relationships

### Core Foundation
- **01-Problem Statement** → Defines the business requirements and constraints
- **02-Goals & Success Criteria** → Establishes success metrics and KPIs
- **03-Comparative Analysis** → Evaluates technical approaches
- **04-Questions** → Captures key considerations and decisions

### Architecture Decisions
- **05-Decision Summary** → Consolidates all major architectural decisions
- **06-API Governance** → Defines API design standards and practices
- **07-Platform Policies** → Establishes security, authentication, and platform policies
- **08-Metering Logic** → Defines billing and usage tracking approach

### Implementation Details
- **09-Data Models** → Defines data structures and database architecture
- **10-API Handler Logic** → Details API implementation and execution flows
- **11-On-Chain Interface** → Defines blockchain integration and smart contracts
- **12-Reliability** → Establishes SLOs, monitoring, and incident response
- **13-Security** → Documents security options and trade-offs

---

## Key Decisions Summary

### **Architecture**: RESTful API with Server-Sent Events and Webhooks
### **Authentication**: OIDC/OAuth 2.0 with MFA (Passkeys + TOTP)
### **Authorization**: RBAC with OAuth 2.0 scopes + tenant isolation
### **Billing**: Lago open-source with tiered pricing + volume discounts
### **Database**: Hybrid (PostgreSQL + MongoDB + Redis)
### **Blockchain**: Batch confirmation with immutable contracts
### **Reliability**: 99.9% SLO with comprehensive monitoring
### **Security**: Multi-layered defense with Cloudflare WAF

---

## Usage Guidelines

1. **For New Team Members**: Start with 01-Problem Statement, then 05-Decision Summary
2. **For Technical Implementation**: Focus on 06-API Governance, 09-Data Models, 10-API Handler Logic
3. **For Security Review**: Review 07-Platform Policies, 11-On-Chain Interface, 13-Security
4. **For Operations**: Reference 12-Reliability for SLOs, monitoring, and incident response
5. **For Business Planning**: Use 08-Metering Logic for billing and pricing decisions

---

## Additional Resources

- **[Reflection](./reflection.md)**: If I had more time
- **[OpenAPI Specification](./openapi.yaml)**: API specification and documentation
