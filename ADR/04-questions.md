# Questions for Stakeholder Input & Our Assumptions

## Table of Contents

### 1. Use Case Questions
- [System Use Cases](#system-use-cases)
- [Excluded Use Cases](#excluded-use-cases)

### 2. Infrastructure & Expertise Questions
- [Existing Infrastructure](#existing-infrastructure)
- [Financial Constraints](#financial-constraints)
- [Multi-cloud Strategy](#multi-cloud-strategy)
- [Deployment Constraints](#deployment-constraints)

### 3. Architecture & Design Questions
- [Simplicity vs Performance](#simplicity-vs-performance)
- [Scale and Growth](#scale-and-growth)
- [Operational Constraints](#operational-constraints)
- [Integration Requirements](#integration-requirements)

### 4. Security & Compliance Questions
- [Security Boundaries](#security-boundaries)
- [Compliance Requirements](#compliance-requirements)
- [Data Privacy](#data-privacy)
- [Key Management](#key-management)

### 5. Performance & Scale Questions
- [Throughput Requirements](#throughput-requirements)
- [Success Rate Targets](#success-rate-targets)
- [Processing Delays](#processing-delays)
- [Latency Requirements](#latency-requirements)
- [Volume Patterns](#volume-patterns)
- [Blockchain Requirements](#blockchain-requirements)

### 6. Business & Project Questions
- [Budget and Resources](#budget-and-resources)
- [Timeline Constraints](#timeline-constraints)
- [Success Criteria](#success-criteria)
- [Risk Tolerance](#risk-tolerance)
- [Business Impact](#business-impact)

### 7. Implementation & Operations Questions
- [Monitoring and Observability](#monitoring-and-observability)
- [Testing and Quality](#testing-and-quality)
- [Disaster Recovery](#disaster-recovery)
- [Data Reprocessing](#data-reprocessing)

---

## Questions and Assumptions

| **Question** | **Our Assumption** | **Stakeholder Answer** |
|--------------|-------------------|----------------------|
| **Use Case Questions** |
| What specific use cases should our job system support? | Long running async processing | _[To be filled by stakeholders]_ |
| What use cases should we explicitly NOT support? | Not specified (to be determined) | _[To be filled by stakeholders]_ |
| **Infrastructure & Expertise Questions** |
| What infrastructure and expertise do we already have? | No pre-existing infrastructure, no deep infrastructure expertise | _[To be filled by stakeholders]_ |
| What are our financial constraints and preferences? | Minimize budget by default (cost optimization priority) | _[To be filled by stakeholders]_ |
| How important is multi-cloud strategy and vendor lock-in avoidance? | Not specified | _[To be filled by stakeholders]_ |
| What are our infrastructure and deployment constraints? | Not specified | _[To be filled by stakeholders]_ |
| **Architecture & Design Questions** |
| Should we prioritize simplicity or performance optimization? | Simplicity (fast-paced environment with low operational overhead) | _[To be filled by stakeholders]_ |
| What's our expected scale and growth trajectory over the next 2-3 years? | Not specified | _[To be filled by stakeholders]_ |
| What are our operational and maintenance constraints? | Fast-paced environment with low operational overhead, no deep infrastructure expertise | _[To be filled by stakeholders]_ |
| What are our integration requirements with existing systems? | Not specified | _[To be filled by stakeholders]_ |
| **Security & Compliance Questions** |
| What should be our security boundaries for transaction signing keys? | Strict ordering per on-chain tx signer key basis | _[To be filled by stakeholders]_ |
| What compliance and legal retention requirements do we have? | Not specified | _[To be filled by stakeholders]_ |
| What are our data privacy and security requirements? | Not specified | _[To be filled by stakeholders]_ |
| What are our key management and rotation requirements? | Not specified | _[To be filled by stakeholders]_ |
| **Performance & Scale Questions** |
| What's our target job processing throughput? | Up to 50K RPS | _[To be filled by stakeholders]_ |
| What's our target job processing success rate? | 99.9% target availability | _[To be filled by stakeholders]_ |
| What's our maximum acceptable job processing delay? | Not specified | _[To be filled by stakeholders]_ |
| What are our latency requirements for job processing? | Not specified | _[To be filled by stakeholders]_ |
| What's our expected job volume and processing patterns? | Spiky or unpredictable load (safety net) | _[To be filled by stakeholders]_ |
| What are our blockchain network requirements and constraints? | Not specified | _[To be filled by stakeholders]_ |
| **Business & Project Questions** |
| What's our budget and resource allocation for this solution? | Minimize budget by default | _[To be filled by stakeholders]_ |
| What are our timeline constraints for implementation? | Not specified | _[To be filled by stakeholders]_ |
| What are our success criteria and acceptance criteria? | Not specified | _[To be filled by stakeholders]_ |
| What's our risk tolerance for system failures and data loss? | Unknown, but we may assume that no jobs shall be lost (safety net). And apply retry policy with exponential back-off to ensure self-healing but avoid loss or starvation with DLQ. DLQ blocks job sequence until manual intervantions to review, resolve, resume the given sequence of jobs. And replay capability from the job state to resolve various job processing issues. | _[To be filled by stakeholders]_ |
| What's the business impact of duplicate job execution vs job failure? | Unknown, but we can aim for at-least-once delviery guarantees with idempotency (sensible default for job processing) | _[To be filled by stakeholders]_ |
| **Implementation & Operations Questions** |
| What are our monitoring, alerting, and observability requirements? | Not specified | _[To be filled by stakeholders]_ |
| What are our testing, validation, and quality assurance requirements? | Not specified | _[To be filled by stakeholders]_ |
| What are our disaster recovery and business continuity requirements? | No jobs shall be lost, retry policy with exponential back-off, DLQ manual intervention, replay capability | _[To be filled by stakeholders]_ |
| Do we need historical data reprocessing capabilities? | Replay from the source to resolve job processing | _[To be filled by stakeholders]_ |
