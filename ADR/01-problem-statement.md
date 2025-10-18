# Problem Statement

## Table of Contents

### 1. Executive Summary
- [Core Business Need](#core-business-need)
- [Challenge Context](#challenge-context)
- [Key Constraints and Requirements](#key-constraints-and-requirements)

### 2. Requirements Legend
- [Requirements Classification](#requirements-classification)

### 3. Functional Requirements
- [Core Off-Chain Operations](#core-off-chain-operations)
  - [Job Submission & Management](#job-submission--management)
  - [Job Processing & Lifecycle](#job-processing--lifecycle)
  - [Error Handling & Recovery](#error-handling--recovery)
  - [Job Status & History](#job-status--history)
- [Core On-Chain Operations](#core-on-chain-operations)
  - [Job Confirmation](#job-confirmation)
  - [Job Status Verification](#job-status-verification)

### 4. Non-Functional Requirements
- [Security Requirements](#security-requirements)
  - [Access Control](#access-control)
  - [Traffic Control](#traffic-control)
  - [Attack Prevention](#attack-prevention)
  - [Emergency Controls](#emergency-controls)
- [Performance Requirements](#performance-requirements)
  - [Core Performance](#core-performance)
  - [Processing Performance](#processing-performance)
  - [Optimization Strategies](#optimization-strategies)
  - [Blockchain Performance](#blockchain-performance)
- [Observability Requirements](#observability-requirements)
- [Reliability Requirements](#reliability-requirements)
  - [Availability & Error Budget](#availability--error-budget)
  - [Health Monitoring](#health-monitoring)
  - [Failure Handling](#failure-handling)
  - [Blockchain State Management](#blockchain-state-management)
  - [Monitoring & Alerting](#monitoring--alerting)
- [Additional Requirements](#additional-requirements)
  - [Compliance](#compliance)
  - [Usability](#usability)
  - [Maintainability](#maintainability)

---

## Core Business Need

Developers need a reliable, secure platform to submit long-running, asynchronous jobs via REST API with blockchain confirmation for final attestation. The platform must support multiple tenants, provide usage-based billing, and ensure high availability.

## Challenge Context

This is a design-led exercise to demonstrate architectural decision-making skills across API gateway surface and minimal on-chain interface. The focus is on API governance, security-by-default, reliability thinking, and usage metering literacy rather than implementation.

## Key Constraints and Requirements

### Requirements Classification

**Legend**:
  - *(R)* = Explicitly Required
  - *(S)* = Suggested for Completeness
  - *(I)* = Implied by Context

## Functional Requirements

### Core Off-Chain Operations

#### Job Submission & Management
  - *(R)* **Job Submission**: REST API endpoints for submitting long-running, asynchronous jobs
  - *(R)* **Idempotency**: Robust idempotency strategy
  - *(R)* **Input Validation**: API handler input validation logic
  - *(I)* **Request Timeouts**: Request timeout handling with retry policies and circuit breakers in case downstream services have issues
  - *(I)* **Job Queues**: Use queues for async job processing

#### Job Processing & Lifecycle
  - *(R)* **Job State Management**: Job state management logic for lifecycle transitions
  - *(I)* **Job Processing**: Asynchronous job processing by async job handlers
  - *(R)* **Job Lifecycle**: Job status transitions (`created` → `processing` → `completed` → `confirming` → `confirmed`) with event emission for usage metering and billing
  - *(R)* **Job Metering**: Capture usage events for billing system

#### Error Handling & Recovery
  - *(I)* **Error Paths (Basic)**: Support for `failed` job status
  - *(S)* **Error Paths (Advanced)**: Support for `cancelled`, `rejected` job statuses
  - *(S)* **Job Error Handling**: Support for cancelled, rejected, failed job statuses with error details
  - *(S)* **Job Retry Logic**: Support for job processing retries and dead letter queue (DLQ) processing
  - *(S)* **Job Cancellation**: Allow users to cancel jobs before completion
  - *(S)* **Job Notifications**: Notify users when jobs complete or fail
  - *(R)* **Error Model**: Error model implementation across all API handlers

#### Job Status & History
  - *(I)* **Job Status Checks**: Ability to check job status
  - *(I)* **Job History**: Track job execution history for billing system and debugging
  - *(S)* **Job Output Access**: Ability to retrieve job outputs

### Core On-Chain Operations

#### Job Confirmation
  - *(R)* **Job Confirmation**: Smart contract functions for job confirmation on EVM-compatible blockchain

#### Job Status Verification
  - *(R)* **Job Status Check**
    - *(R)* **Real-time**: Emit and listen for smart-contract events
    - *(I)* **Fallback** Read smart contract state and/or events

## Non-Functional Requirements

### Security Requirements

#### Access Control
  - *(R)* **Authentication**: Define authentication model, token lifecycle (expiry, rotation)
  - *(R)* **Authorization**: Enforce least-privilege access for API endpoints, multi-tenant isolation
  - *(R)* **On-Chain Access Control**: Define who can confirm jobs (smart contract access control)

#### Traffic Control
  - *(R)* **Rate Limiting**: Define per-tenant rate limits
  - *(R)* **Quotas**: Define usage quotas
  - *(R)* **Abuse Protection**: Define abuse protection mechanisms

#### Attack Prevention
  - *(R)* **Replay Attack Prevention**: Prevent confirming the same job twice
  - *(I)* **Privacy Protection**: Prevent correlation
  - *(I)* **Integrity Protection**: Ensure job inputs/outputs integrity by including job hash in blockchain confirmation

#### Emergency Controls
  - *(I)* **Emergency Pause**: Ability to pause confirmations for security issues
  - *(S)* **Multi-Signature Requirements**: Require multiple signatures for critical confirmations
### Performance Requirements

#### Core Performance
  - *(I)* **Resource Usage**: Define system resource requirements (CPU, memory, storage)
  - *(R)* **API Latency**: Define p95 latency criteria for job submission and status checks
  - *(R)* **Rate Limiting**: Define per-tenant rate limits and quotas

#### Processing Performance
  - *(I)* **Throughput**: Define job processing throughput requirements
  - *(I)* **Concurrency**: Define concurrent job processing limits
  - *(I)* **Scalability**: Handle growing number of tenants and job volume

#### Optimization Strategies
  - *(I)* **Caching**: Define caching strategies for performance optimization

#### Blockchain Performance
  - *(I)* **Gas Optimization**: Optimize gas costs for blockchain confirmations
  - *(I)* **Transaction Batching**: Batch multiple job confirmations for efficiency
  - *(I)* **Event-Only Storage**: Store minimal data on-chain, emit events for details
  - *(I)* **Storage Optimization**: Use efficient data structures and packed structs
  - *(I)* **Gas Price Optimization**: Dynamic gas price adjustment based on network conditions
### Observability Requirements
  - *(I)* **Logging**: Basic logging, metrics, distributed tracing
  - *(I)* **Monitoring**: Basic performance monitoring and alerting

### Reliability Requirements

#### Availability & Error Budget
  - *(R)* **Availability**: Define system uptime requirements
  - *(R)* **Error Budget**: Define error budget policy

#### Health Monitoring
  - *(I)* **Health Checks**: Monitor system health and blockchain connectivity
  - *(I)* **Load Balancing**: Distribute load across multiple instances
  - *(I)* **Scaling**: Define horizontal and vertical scaling strategies for growing load

#### Failure Handling
  - *(I)* **Data Backups**: Define backup procedures
  - *(I)* **Redundancy/Failover**: Define backup systems and failover procedures
  - *(I)* **Fault Tolerance**: Graceful handling of failures with retry mechanisms
  - *(I)* **Circuit Breakers**: Prevent cascade failures in distributed systems
  - *(S)* **Recovery/Rollbacks**: Recovery procedures and rollback capabilities for system failures with RTO/RPO targets

#### Blockchain State Management
  - *(I)* **State Reconciliation**: Periodically verify and maintain job state consistency across off-chain and on-chain systems
  - *(I)* **Transaction Retry Logic**: Analyze failure types, adjust gas parameters, and retry failed blockchain transactions with manual intervention for business logic, security, and data integrity failures
  - *(I)* **Event Replay**: Replay missed blockchain events during downtime

#### Monitoring & Alerting
  - *(I)* **Performance Monitoring**: Real-time performance monitoring and metrics collection
  - *(I)* **Alerting**: Automated alerting for reliability issues and system failures

### Additional Requirements

#### Compliance
  - *(S)* **Compliance**: Data retention policies, audit trails, regulatory requirements, privacy protection

#### Usability
  - *(S)* **Usability**: Developer experience, API documentation, error messages

#### Maintainability
  - *(S)* **Maintainability**: Clear separation of concerns, comprehensive testing, CI/CD pipelines
