# Jobs API — Architecture Decision Record (ADR)

<style>
table {
  vertical-align: top;
}
td {
  vertical-align: top;
}
th {
  vertical-align: top;
}
</style>


## A. Architecture Decision Pack (ADR-style)


### Problem Statement

**Core Business Need**: Developers need a reliable, secure platform to submit long-running, asynchronous jobs via REST API with blockchain confirmation for final attestation. The platform must support multiple tenants, provide usage-based billing, and ensure high availability.

**Challenge Context**: This is a design-led exercise to demonstrate architectural decision-making skills across API gateway surface and minimal on-chain interface. The focus is on API governance, security-by-default, reliability thinking, and usage metering literacy rather than implementation.

**Key Constraints and Requirements**:

**Legend**:
  - *(R)* = Explicitly Required
  - *(S)* = Suggested for Completeness
  - *(I)* = Implied by Context

**Functional Requirements**:

- **Core Off-Chain Operations**:
  - *(R)* **Job Submission**: REST API endpoints for submitting long-running, asynchronous jobs
  - *(R)* **Idempotency**: Robust idempotency strategy
  - *(R)* **Input Validation**: API handler input validation logic
  - *(I)* **Request Timeouts**: Request timeout handling with retry policies and circuit breakers in case downstream services have issues
  - *(I)* **Job Queues**: Use queues for async job processing
  - *(R)* **Job State Management**: Job state management logic for lifecycle transitions
  - *(I)* **Job Processing**: Asynchronous job processing by async job handlers
  - *(R)* **Job Lifecycle**: Job status transitions (`created` → `processing` → `completed` → `confirming` → `confirmed`) with event emission for usage metering and billing
  - *(R)* **Job Metering**: Capture usage events for billing system
  - *(I)* **Error Paths (Basic)**: Support for `failed` job status
  - *(S)* **Error Paths (Advanced)**: Support for `cancelled`, `rejected` job statuses
  - *(S)* **Job Error Handling**: Support for cancelled, rejected, failed job statuses with error details
  - *(S)* **Job Retry Logic**: Support for job processing retries and dead letter queue (DLQ) processing
  - *(S)* **Job Cancellation**: Allow users to cancel jobs before completion
  - *(S)* **Job Notifications**: Notify users when jobs complete or fail
  - *(R)* **Error Model**: Error model implementation across all API handlers
  - *(I)* **Job Status Checks**: Ability to check job status
  - *(I)* **Job History**: Track job execution history for billing system and debugging
  - *(S)* **Job Output Access**: Ability to retrieve job outputs
- **Core On-Chain Operations**:
  - *(R)* **Job Confirmation**: Smart contract functions for job confirmation on EVM-compatible blockchain
  - *(R)* **Job Status Check**
    - *(R)* **Real-time**: Emit and listen for smart-contract events
    - *(I)* **Fallback** Read smart contract state and/or events

**Non-Functional Requirements**:

- **Security**:
  - **Access Control**:
    - *(R)* **Authentication**: Define authentication model, token lifecycle (expiry, rotation)
    - *(R)* **Authorization**: Enforce least-privilege access for API endpoints, multi-tenant isolation
    - *(R)* **On-Chain Access Control**: Define who can confirm jobs (smart contract access control)
  - **Traffic Control**:
    - *(R)* **Rate Limiting**: Define per-tenant rate limits
    - *(R)* **Quotas**: Define usage quotas
    - *(R)* **Abuse Protection**: Define abuse protection mechanisms
  - **Attack Prevention**:
    - *(R)* **Replay Attack Prevention**: Prevent confirming the same job twice
    - *(I)* **Privacy Protection**: Prevent correlation
    - *(I)* **Integrity Protection**: Ensure job inputs/outputs integrity by including job hash in blockchain confirmation
  - **Emergency Controls**:
    - *(I)* **Emergency Pause**: Ability to pause confirmations for security issues
    - *(S)* **Multi-Signature Requirements**: Require multiple signatures for critical confirmations
- **Performance**:
  - **Core Performance**:
    - *(I)* **Resource Usage**: Define system resource requirements (CPU, memory, storage)
    - *(R)* **API Latency**: Define p95 latency criteria for job submission and status checks
    - *(R)* **Rate Limiting**: Define per-tenant rate limits and quotas
  - **Processing Performance**:
    - *(I)* **Throughput**: Define job processing throughput requirements
    - *(I)* **Concurrency**: Define concurrent job processing limits
    - *(I)* **Scalability**: Handle growing number of tenants and job volume
  - **Optimization**:
    - *(I)* **Caching**: Define caching strategies for performance optimization
  - **Blockchain Performance**:
    - *(I)* **Gas Optimization**: Optimize gas costs for blockchain confirmations
    - *(I)* **Transaction Batching**: Batch multiple job confirmations for efficiency
    - *(I)* **Event-Only Storage**: Store minimal data on-chain, emit events for details
    - *(I)* **Storage Optimization**: Use efficient data structures and packed structs
    - *(I)* **Gas Price Optimization**: Dynamic gas price adjustment based on network conditions
- **Observability**:
  - *(I)* **Logging**: Basic logging, metrics, distributed tracing
  - *(I)* **Monitoring**: Basic performance monitoring and alerting
- **Reliability**:
  - *(R)* **Availability**: Define system uptime requirements
  - *(R)* **Error Budget**: Define error budget policy
  - *(I)* **Health Checks**: Monitor system health and blockchain connectivity
  - *(I)* **Load Balancing**: Distribute load across multiple instances
  - *(I)* **Scaling**: Define horizontal and vertical scaling strategies for growing load
  - **Failure Handling**:
    - *(I)* **Data Backups**: Define backup procedures
    - *(I)* **Redundancy/Failover**: Define backup systems and failover procedures
    - *(I)* **Fault Tolerance**: Graceful handling of failures with retry mechanisms
    - *(I)* **Circuit Breakers**: Prevent cascade failures in distributed systems
    - *(S)* **Recovery/Rollbacks**: Recovery procedures and rollback capabilities for system failures with RTO/RPO targets
    - *(I)* **Blockchain State Management**:
      - *(I)* **State Reconciliation**: Periodically verify and maintain job state consistency across off-chain and on-chain systems
      - *(I)* **Transaction Retry Logic**: Analyze failure types, adjust gas parameters, and retry failed blockchain transactions with manual intervention for business logic, security, and data integrity failures
      - *(I)* **Event Replay**: Replay missed blockchain events during downtime
  - **Monitoring & Alerting**:
    - *(I)* **Performance Monitoring**: Real-time performance monitoring and metrics collection
    - *(I)* **Alerting**: Automated alerting for reliability issues and system failures
- *(S)* **Compliance**: Data retention policies, audit trails, regulatory requirements, privacy protection
- *(S)* **Usability**: Developer experience, API documentation, error messages
- *(S)* **Maintainability**: Clear separation of concerns, comprehensive testing, CI/CD pipelines


### Goals & Success Criteria

**Primary Objectives**:
- **Reliable Jobs Platform**: Build a secure, high-availability platform for long-running asynchronous jobs
- **Blockchain Integration**: Provide final attestation on EVM-compatible blockchain
- **Multi-tenant Platform**: Support multiple tenants with proper isolation
- **Usage-based Billing**: Enable accurate billing through event capture

**Success Metrics and KPIs**:

**Key Success Indicators**:
- **API Governance**: Define REST API endpoints, versioning strategy, error model, and idempotency strategy
- **Platform Policies**: Define authentication, rate limits, quotas, and abuse protection policies
- **Metering Logic**: Define usage events for billing system and invoice mapping
- **System Design**: Define on-chain interface and API handler logic
- **Reliability**: Define success rate, p95 latency criteria, and error budget policy


### Options Considered for `Transport`

| **Transport** | **Use Cases** | **Latency** | **Throughput** | **CPU Usage** | **Memory Usage** | **Network Usage** | **Scaling & Limits** |
|---------------|---------------|-------------|----------------|---------------|------------------|-------------------|---------------------|
| **REST + Polling** | 🔧 Simple APIs<br/>📊 Low-frequency updates<br/>✅ Basic status checks | ❌ **Worst**: 200-1000ms (polling delay) | ❌ **Worst**: 10-100 req/sec (polling overhead) | ⚠️ **Medium**: 10-25%<br/>• HTTP request/response overhead<br/>• Connection setup/teardown<br/>• Response generation | ✅ **Best**: 5-15KB<br/>• HTTP request/response buffers<br/>• Connection metadata<br/>• Stateless processing | ⚠️ **High**: 1-5KB<br/>• Full HTTP request/response<br/>• Headers + payload overhead<br/>• Connection establishment | ⚠️ **Limited**: 6/browser, 10K/server<br/>⚠️ **Connection-bound scaling** |
| **REST + Long Polling** | ⚡ Near real-time APIs<br/>🔄 Reduced polling overhead | ⚠️ **Moderate**: 50-200ms (reduced delay) | ⚠️ **Moderate**: 100-1K req/sec (reduced overhead) | ⚠️ **Medium**: 15-30%<br/>• Connection state management<br/>• Timeout handling<br/>• Request queuing | ✅ **Best**: 5-15KB<br/>• HTTP request/response buffers<br/>• Connection metadata<br/>• Stateless processing | ⚠️ **High**: 1-5KB<br/>• Full HTTP request/response<br/>• Headers + payload overhead<br/>• Connection establishment | ⚠️ **Limited**: 6/browser, 10K/server<br/>⚠️ **Connection-hold scaling** |
| **MQTT** | 🌐 IoT devices<br/>📡 Sensor data<br/>🔄 Lightweight messaging | ✅ **Good**: 5-20ms (lightweight pub/sub) | ⚠️ **Moderate**: 5K-20K msgs/sec (broker complexity) | ✅ **Good**: 5-15%<br/>• Lightweight protocol overhead<br/>• Minimal processing<br/>• Efficient pub/sub | ✅ **Best**: 5-15KB<br/>• Minimal connection state<br/>• Lightweight protocol<br/>• Stateless processing | ✅ **Good**: 50-200B<br/>• Minimal framing overhead<br/>• Binary data support<br/>• Lightweight protocol | ✅ **Good**: 10K-100K connections<br/>✅ **Good**: Horizontal scaling capability, stateless |
| **Server-Sent Events** | 🔔 Real-time notifications<br/>📡 One-way streaming<br/>📈 Live dashboards | ✅ **Good**: 20-100ms (one-way streaming) | ⚠️ **Moderate**: 1K-10K events/sec (6 SSE connections/browser) | ⚠️ **Medium**: 10-25%<br/>• Stream management overhead<br/>• Connection state tracking<br/>• Event processing | ⚠️ **Medium**: 30-80KB<br/>• Stream buffering overhead<br/>• Event queuing<br/>• Connection state management | ⚠️ **Medium**: 500B-2KB<br/>• HTTP headers + SSE framing<br/>• Event data overhead<br/>• Stream management | ⚠️ **Limited**: 6/browser, 10K/server<br/>⚠️ **Stream-management scaling** |
| **Webhooks** | 🖥️ Server-to-server<br/>📢 Event notifications<br/>🔗 External integrations | ✅ **Good**: 10-100ms (push-based delivery) | ⚠️ **Moderate**: 1K-10K req/sec (external dependency) | ⚠️ **Medium**: 10-25%<br/>• HTTP client processing<br/>• Retry logic overhead<br/>• Delivery tracking | ✅ **Best**: 5-15KB<br/>• HTTP request/response buffers<br/>• Connection metadata<br/>• Stateless processing | ⚠️ **High**: 1-5KB<br/>• Full HTTP request/response<br/>• Headers + payload overhead<br/>• External endpoint delivery | ⚠️ **Server-capacity bound**<br/>⚠️ **External dependency scaling** |
| **WebSockets** | 💬 Real-time chat<br/>🎮 Gaming, collaboration<br/>🔄 Bidirectional streaming | ✅ **Good**: 5-50ms (bidirectional messaging) | ✅ **Good**: 10K-100K msgs/sec (persistent connections) | ✅ **Good**: 5-15%<br/>• Efficient persistent connections<br/>• Minimal framing overhead<br/>• Optimized for real-time | ⚠️ **Worst**: 100-300KB<br/>• Full persistent connection state<br/>• Message buffers<br/>• Client tracking | ✅ **Good**: 100-500B<br/>• Frame overhead + payload<br/>• Minimal framing (2-14 bytes)<br/>• Binary data support | ⚠️ **Limited**: 6/browser, 10K/server<br/>❌ **Worst**: Sticky sessions required |
| **gRPC Streaming** | 🚀 High-performance APIs<br/>🔗 Microservices communication<br/>⚡ Ultra-low latency systems | ✅ **Best**: 2-15ms (bidirectional streaming) | ✅ **Best**: 50K-500K req/sec (high performance) | ✅ **Good**: 8-15%<br/>• Binary serialization overhead<br/>• HTTP/2 multiplexing<br/>• Protocol buffer processing | ⚠️ **Medium**: 50-150KB<br/>• Stream management overhead<br/>• Protocol Buffer serialization<br/>• HTTP/2 connection state | ✅ **Good**: 50-200B<br/>• HTTP/2 + Protocol Buffer overhead<br/>• Binary serialization<br/>• Header compression | ⚠️ **Limited**: 6/browser, 10K/server<br/>⚠️ **HTTP/2 multiplexing limits** |


### Options Considered for `API Architecture`

| **Option** | **Latency** | **Throughput** | **Complexity** | **Scaling** | **Security** |
|------------|-------------|----------------|----------------|------------|-------------|
| **RESTful API with Short-Polling** | ❌ **Worst**: 200-1000ms (polling delay) | ❌ **Worst**: 100-500 req/sec (polling overhead) | ✅ **Low**: Simple HTTP setup, stateless retry | ✅ **Good**: Easy horizontal scaling | ✅ **Good**: Simple (standard HTTP auth) |
| **RESTful API with Long-Polling** | ⚠️ **Moderate**: 50-200ms (reduced delay) | ⚠️ **Moderate**: 500-2000 req/sec (reduced overhead) | ✅ **Low**: Simple HTTP setup, stateless retry | ✅ **Good**: Easy horizontal scaling | ✅ **Good**: Simple (standard HTTP auth) |
| **MQTT API** | ✅ **Good**: 5-20ms (lightweight pub/sub) | ⚠️ **Moderate**: 5000-20000 msgs/sec (broker complexity) | ✅ **Low**: Simple lightweight protocol, minimal setup | ✅ **Good**: Horizontal scaling capability, stateless | ⚠️ **Moderate**: Authentication, authorization, TLS required |
| **RESTful API with Server-Sent Events** | ✅ **Good**: 10-50ms (one-way streaming) | ⚠️ **Moderate**: 1000-5000 events/sec (6 SSE connections/browser) | ✅ **Low**: Simple HTTP streaming, browser reconnection | ✅ **Good**: Horizontal scaling capability, 6 SSE connections/browser | ⚠️ **Moderate**: Connection auth required |
| **RESTful API with Webhooks** | ✅ **Good**: 5-20ms (push-based delivery) | ⚠️ **Moderate**: 2000-10000 req/sec (external dependency) | ⚠️ **Moderate**: Webhook delivery system, HTTP retry logic | ✅ **Good**: Horizontal scaling capability, stateless | ⚠️ **Moderate**: Security coordination required |
| **RESTful API with WebSockets** | ✅ **Good**: 1-10ms (bidirectional messaging) | ⚠️ **Moderate**: 500-2000 msgs/sec (sticky sessions, memory) | ⚠️ **Moderate**: WebSocket server setup, connection management | ⚠️ **Moderate**: Sticky sessions required | ⚠️ **Moderate**: Connection auth required |
| **gRPC with Streaming** | ✅ **Best**: 0.5-5ms (bidirectional streaming) | ✅ **Best**: 10000-50000 req/sec (high performance) | ⚠️ **Moderate**: Protobuf schema, code generation, connection pooling | ✅ **Good**: Horizontal scaling capability, stateless | ⚠️ **Moderate**: mTLS, certificate management required |
| **GraphQL API with Subscriptions** | ⚠️ **Moderate**: 20-100ms (transport dependent) | ⚠️ **Moderate**: 1000-3000 req/sec (subscription scaling) | ❌ **High**: GraphQL server + transport + schema + resolvers, subscription management | ⚠️ **Moderate**: Subscription scaling challenges | ✅ **Good**: Simple (standard HTTP auth) |


### Options Considered for `Async Job Processing`

| **Option** | **Use Cases** | **Key Tradeoffs** | **Performace** | **Delivery Guarantees** | **Ordering Guarantees** | **Scalability** | **Reliability** | **Complexity** | **Cost** | **Example Services** |
|------------|----------------|-------------------|-------------------------|----------------|----------------|----------------|-----------------|-------------|----------------|-------------------|
| **Cloud Job Services (AWS, GCP, Azure)** | **Cloud-native, Managed services**<br/>• Startups, MVPs, rapid prototyping<br/>• Serverless architectures<br/>• Multi-region deployments<br/>• Teams preferring managed solutions | ⚠️ **Vendor Lock-in**: Cloud dependency<br/>⚠️ **Cost**: Pay-per-use pricing<br/>⚠️ **Control**: Limited configuration | ✅ **Low Latency**: 10-100ms<br/><br/>⚠️ **Moderate Throughput**: 10K-100K msgs/sec | 🔄 At-least-once<br/>⚡ At-most-once | 📋 FIFO ordering (SQS FIFO, Service Bus sessions)<br/>⚠️ **Strictness**: Moderate (FIFO only within message group/partition) | ✅ High<br/>• SQS ~300 TPS/queue<br/>• Pub/Sub ~1M msgs/sec/topic<br/>• Service Bus ~1K-10K msgs/sec/queue) | ✅ High (99.9-99.95% uptime SLA:<br/>• High-performance infrastructure<br/>• Geographic distribution<br/>• Network optimization and edge computing) | ✅ Simple (cloud API integration + managed service) | ⚠️ Moderate (pay-per-use pricing, scales with usage) | • **AWS**: SQS, SNS, EventBridge<br/>• **GCP**: Pub/Sub, Cloud Tasks<br/>• **Azure**: Service Bus, Event Grid<br/>• **Multi-cloud**: CloudAMQP, MessageBird |
| **Redis-based Job Queue** | **Ultra-low latency, High-throughput**<br/>• Real-time gaming, trading systems<br/>• Caching layer job processing<br/>• Cost-sensitive high-performance needs<br/>• Teams with Redis expertise | ⚠️ **Memory Bound**: Limited by RAM<br/>⚠️ **Data Loss Risk**: Memory-first design<br/>✅ **Cost Effective**: Fixed infrastructure costs | ✅ **Ultra-Low Latency**: 0.4-2ms<br/><br/>✅ **High Throughput**: 4M-23M ops/sec | 🔄 At-least-once<br/>⚡ At-most-once | 📋 FIFO ordering (Redis Streams)<br/>✅ **Strictness**: High (strict FIFO within stream) | ⚠️ Moderate<br/>• Redis Streams: 500K-5M ops/sec<br/>• Cluster scaling: 10M+ ops/sec<br/>• Pipeline operations: 22.9M ops/sec) | ✅ High (Redis reliability with clustering for replication and, snapshots and AOF logs for durability | ⚠️ Moderate (Redis infrastructure, cluster setup + standard Redis monitoring) | ✅ Low (fixed infrastructure costs) | • **Open Source**: Redis, KeyDB, DragonflyDB<br/>• **Managed**: AWS ElastiCache, GCP Memorystore, Azure Cache<br/>• **Cloud**: Redis Cloud, Upstash, Aiven<br/>• **Enterprise**: Redis Enterprise, Red Hat OpenShift |
| **Database-based Job Queue** | **ACID compliance, Simple integration**<br/>• Financial systems, audit trails<br/>• Teams with strong DB expertise<br/>• Existing database infrastructure<br/>• Batch processing workflows | ⚠️ **Performance**: DB bottlenecks<br/>⚠️ **Scaling**: Vertical scaling limits<br/>✅ **ACID**: Transaction guarantees | ⚠️ **Moderate Latency**: 10-100ms<br/><br/>⚠️ **Moderate Throughput**: 100K-1M ops/sec | 🔄 At-least-once<br/>⚡ At-most-once<br/>🎯 Exactly-once (ACID) | 📋 FIFO ordering (per table/queue)<br/>✅ **Strictness**: High (strict FIFO within queue, ACID guarantees) | ✅ High<br/>• PostgreSQL ~10K-100K TPS<br/>• MySQL ~5K-50K TPS<br/>• optimized can reach 100K-1M ops/sec) | ✅ High (with a proper database fault tolerance mechanisms) | ✅ Simple (simple database tables + standard database maintenance) | ✅ Low (database only, no additional infrastructure) | • **Open Source**: PostgreSQL, MySQL, SQLite<br/>• **Enterprise**: SQL Server, Oracle, IBM DB2<br/>• **Cloud**: AWS RDS, GCP Cloud SQL, Azure SQL<br/>• **NoSQL**: MongoDB, CouchDB, ArangoDB |
| **RabbitMQ** | **Complex routing, Microservices**<br/>• Enterprise message queuing<br/>• Complex routing requirements<br/>• Microservices communication<br/>• Teams with RabbitMQ expertise | ⚠️ **Scaling**: Queue affinity limits<br/>⚠️ **Performance**: Erlang VM overhead<br/>✅ **Mature**: Battle-tested, stable | ⚠️ **Moderate Latency**: 10-50ms<br/><br/>⚠️ **Moderate Throughput**: 10K-200K msgs/sec | 🔄 At-least-once<br/>⚡ At-most-once | 📋 FIFO ordering (per queue)<br/>✅ **Strictness**: High (strict FIFO within queue) | ⚠️ Moderate<br/>• Single broker: 10K-50K msgs/sec<br/>• Clustered: 50K-200K msgs/sec<br/>• Limited by Erlang VM and message durability<br/>• Queue affinity limits horizontal scaling) | ✅ High (with clustering, HA, persistent messages:<br/>• Message queuing adds overhead<br/>• Persistence and reliability impact performance<br/>• Network I/O and disk operations<br/>• Erlang VM processing overhead) | ⚠️ Moderate (broker setup, clustering) <br/><br/>OR<br/><br/> ✅ Simple (managed service/enterprise support) | ✅ Low (fixed infrastructure costs) <br/><br/>OR<br/><br/> ❌ High (enterprise support) | • **Open Source**: RabbitMQ, RabbitMQ Streams<br/>• **Managed**: CloudAMQP, AWS MQ, Azure Service Bus<br/>• **Enterprise**: Pivotal RabbitMQ, VMware Tanzu<br/>• **Cloud**: Heroku RabbitMQ, DigitalOcean |
| **Apache Kafka** | **Event streaming, Data pipelines**<br/>• Event sourcing, CQRS patterns<br/>• Real-time data streaming<br/>• Analytics, data lakes<br/>• High-scale event processing | ✅ **Scalability**: Horizontal partitioning<br/>✅ **Durability**: Persistent, replayable<br/>⚠️ **Operational**: Complex monitoring | ✅ **Low Latency**: 1-20ms<br/><br/>✅ **High Throughput**: 1M-100M msgs/sec | 🔄 At-least-once<br/>⚡ At-most-once<br/>🎯 Exactly-once (Kafka 0.11+) | 📋 FIFO ordering (per partition)<br/>✅ **Strictness**: High (strict FIFO within partition) | ✅ High<br/>• Single partition: 1M-10M msgs/sec<br/>• Multi-partition: 10M-100M msgs/sec<br/>• Horizontal scaling with partitions) | ✅ High (replication, fault tolerance, audit trail, replay capability:<br/>• Single partition: 1-5ms<br/>• Multi-partition: 5-20ms<br/>• Optimized for high-throughput streaming) | ❌ High (complex setup, clustering + partition rebalancing, consumer group management, schema registry) <br/><br/>OR<br/><br/> ✅ Simple (managed service/enterprise support) | ✅ Low (fixed infrastructure costs) <br/><br/>OR ❌<br/><br/> High (enterprise support, replay capability) | • **Open Source**: Apache Kafka, Redpanda<br/>• **Managed**: Confluent Cloud, AWS MSK, Azure Event Hubs<br/>• **Enterprise**: Confluent Platform, Cloudera<br/>• **Cloud**: GCP Pub/Sub, AWS Kinesis, Azure Event Grid |
| **Apache Pulsar** | **Enterprise message governance**<br/>• Built-in multi-tenancy isolation<br/>• Enterprise geo-replication<br/>• Complex message routing, transformation<br/>• Advanced topic policies, quotas | ⚠️ **Performance**: Complex architecture overhead<br/>✅ **Enterprise**: Built-in multi-tenancy, geo-replication<br/>⚠️ **Operational**: Complex monitoring | ⚠️ **Moderate Latency**: 10-50ms<br/><br/>⚠️ **Moderate Throughput**: 100K-10M msgs/sec | 🔄 At-least-once<br/>⚡ At-most-once | 📋 FIFO ordering (per topic)<br/>✅ **Strictness**: High (strict FIFO within topic) | ✅ High<br/>• Single topic: 100K-1M msgs/sec<br/>• Multi-topic: 1M-10M msgs/sec<br/>• Complex architecture limits performance) | ✅ High (geo-replication, fault tolerance, durable message storage:<br/>• Broker + BookKeeper + Zookeeper overhead<br/>• Geo-replication adds latency<br/>• Durable message storage impact<br/>• Multi-component architecture complexity) | ❌ High (complex setup: broker, BookKeeper, Zookeeper + broker management, topic policies, monitoring) <br/><br/>OR<br/><br/> ✅ Simple (managed service) | ✅ Low (fixed infrastructure costs) <br/><br/>OR ❌<br/><br/> High (enterprise support) | • **Open Source**: Apache Pulsar, Apache BookKeeper<br/>• **Managed**: StreamNative Cloud, DataStax Pulsar<br/>• **Enterprise**: StreamNative Platform<br/>• **Cloud**: AWS MSK, Azure Event Hubs (Pulsar mode) |
| **NATS JetStream** | **IoT, Edge computing, Simple systems**<br/>• IoT, sensor data processing<br/>• Edge computing scenarios<br/>• Lightweight, simple deployments<br/>• Real-time messaging | ✅ **Performance**: Ultra-low latency<br/>✅ **Simplicity**: Lightweight, minimal config<br/>⚠️ **Ecosystem**: Smaller community | ✅ **Ultra-Low Latency**: 1-5ms<br/><br/>✅ **High Throughput**: 1M-10M msgs/sec | 🔄 At-least-once<br/>⚡ At-most-once | 📋 FIFO ordering (per stream)<br/>✅ **Strictness**: High (strict FIFO within stream) | ✅ High<br/>• Single server ~100K-1M msgs/sec<br/>• Clustered ~1M-10M msgs/sec<br/>• realistic performance) | ✅ High (fault tolerance, persistence) | ⚠️ Moderate (lightweight server, minimal config + configuration management, built-in monitoring) <br/><br/>OR<br/><br/> ✅ Simple (managed service) | ✅ Low (minimal resource requirements) or ⚠️ Moderate (managed service costs) | • **Open Source**: NATS, NATS JetStream<br/>• **Managed**: NATS Cloud, Synadia NGS<br/>• **Enterprise**: Synadia NGS Enterprise<br/>• **Cloud**: AWS NATS, GCP NATS, Azure NATS |


### Options Considered for `Integration of Blockchain Transaction Creation`

| **Option** | **Key Tradeoffs** | **Gas Efficiency** | **Reliability** | **Scalability** | **Complexity** |
|------------|-------------------|-------------------|----------------|----------------|----------------|
| **Immediate Blockchain Transaction Call** | ⚠️ **Simple but expensive**:<br/>• ✅ Simplest implementation<br/>• ❌ Highest gas costs<br/>• ❌ No fault tolerance<br/>• ❌ Tightly coupled | ❌ **Worst**:<br/>• Individual transaction per job<br/>• Highest gas costs per job| ❌ **Worst**:<br/>• Tightly coupled<br/>• No fault tolerance | ⚠️ **Moderate**:<br/>• Individual transaction processing<br/>• Chain TPS limits execution | ✅ **Low**:<br/>• Direct blockchain integration<br/>• Simple blockchain tx processing |
| **Queue-based Async Transaction Call Execution** | ⚠️ **Moderate reliability but expensive**:<br/>• ⚠️ Moderate reliability<br/>• ✅ Low complexity<br/>• ❌ Still high gas costs<br/>• ❌ No gas optimization | ❌ **Worst**:<br/>• Individual transaction per job<br/>• High gas costs per job| ⚠️ **Moderate**:<br/>• Decoupled logic<br/>• Queue-based fault tolerance<br/>• Context-aware retry flow for error recovery | ⚠️ **Moderate**:<br/>• Individual transaction processing<br/>• Chain TPS limits execution | ✅ **Low**:<br/>• Reuse existing async processing infra<br/>• Simple blockchain tx processing |
| **Volume-constrained Transaction Batching** | ✅ **Best gas efficiency**:<br/>• ✅ Highest gas optimization<br/>• ✅ Best scalability<br/>• ⚠️ Moderate complexity<br/>• ⚠️ Volume-dependent batching | ✅ **Highest**:<br/>• Batch processing (volume-constrained)<br/>• Significant gas optimization | ✅ **High**:<br/>• Decoupled logic<br/>• Volume-constrained batching<br/>• Fault isolation between batches | ✅ **Highest**:<br/>• Batch scaling<br/>• Optimized processing | ⚠️ **Moderate**:<br/>• Batch coordination setup<br/>• Batch management, coordination |
| **Time-constrained Transaction Batching** | ✅ **Predictable batching**:<br/>• ✅ Good gas optimization<br/>• ✅ Predictable timing<br/>• ⚠️ Moderate complexity<br/>• ⚠️ Time-dependent batching | ✅ **High**:<br/>• Batch processing (time-constrained)<br/>• Good gas optimization | ✅ **High**:<br/>• Decoupled logic<br/>• Time-constrained batching<br/>• Fault isolation between batches | ✅ **High**:<br/>• Scheduled scaling<br/>• Optimized batching | ⚠️ **Moderate**:<br/>• Scheduler setup, job state tracking<br/>• Scheduler maintenance, job state management |


### Options Considered for `Integration of Blockchain Event Handling`

| **Option** | **Key Tradeoffs** | **Latency** | **Event Ordering** | **Reliability** | **Scalability** | **Complexity** |
|------------|-------------------|-------------|-------------------|----------------|----------------|----------------|
| **Direct Event Subscription** | ⚠️ **Simple but fragile**:<br/>• ✅ Simplest implementation<br/>• ✅ No additional infrastructure<br/>• ❌ No fault tolerance<br/>• ❌ Manual failover setup | ✅ **Ultra-Low**:<br/>• 1-5ms event processing<br/>• No queue delays<br/>• Direct RPC connection<br/>• Real-time processing | ✅ **High**:<br/>• Block-based ordering<br/>• Guaranteed ordering (single subscriber)<br/>• No race conditions (single subscriber)<br/>• Event deduplication possible | ❌ **Low**:<br/>• No retry mechanisms<br/>• Single point of failure<br/>• No event persistence<br/>• No fault tolerance | ⚠️ **Moderate**:<br/>• RPC connection limits (100-1000)<br/>• Chain TPS limits (15-100 TPS)<br/>• No horizontal scaling<br/>• Manual failover setup | ✅ **Low**:<br/>• Direct blockchain integration<br/>• Simple event processing<br/>• Minimal setup |
| **Webhook-based Event Processing** | ⚠️ **External dependency**:<br/>• ✅ Simple integration<br/>• ✅ No infrastructure<br/>• ❌ External dependency<br/>• ❌ Limited control | ⚠️ **Moderate**:<br/>• 100-500ms webhook delay<br/>• Network latency (50-200ms)<br/>• External service processing<br/>• Retry mechanisms | ⚠️ **Moderate**:<br/>• Webhook delivery order<br/>• No guaranteed ordering<br/>• External service control<br/>• Limited deduplication | ⚠️ **Moderate**:<br/>• External service reliability<br/>• Limited retry control<br/>• Service dependency<br/>• No event persistence | ⚠️ **Moderate**:<br/>• Webhook rate limits (100-1000/min)<br/>• External service scaling<br/>• Limited horizontal scaling<br/>• Service dependency | ✅ **Low**:<br/>• Simple webhook integration<br/>• Minimal setup<br/>• External service management |
| **Event-Driven with Message Queue** | ✅ **Reliable and practical**:<br/>• ✅ High reliability<br/>• ✅ Fault tolerance<br/>• ✅ Moderate complexity<br/>• ⚠️ Queue infrastructure | ⚠️ **Moderate**:<br/>• 10-100ms queue processing<br/>• Retry mechanisms<br/>• Event persistence overhead<br/>• Queue-based delays | ✅ **High**:<br/>• FIFO queue ordering<br/>• Guaranteed processing order<br/>• No race conditions<br/>• Event deduplication | ✅ **High**:<br/>• Queue-based fault tolerance<br/>• Automatic retry mechanisms<br/>• Event persistence<br/>• Dead letter queues | ✅ **High**:<br/>• Queue-based scaling<br/>• Horizontal scaling (10-100 workers)<br/>• Load distribution<br/>• Auto-scaling support | ⚠️ **Moderate**:<br/>• Queue setup and management<br/>• Event processing logic<br/>• Monitoring and alerting |
| **Event Store with CQRS** | ✅ **Audit & Replay**:<br/>• ✅ Event sourcing benefits<br/>• ✅ Audit trail<br/>• ⚠️ High complexity<br/>• ⚠️ Event store overhead | ⚠️ **Moderate**:<br/>• 50-200ms event store processing<br/>• CQRS projection updates<br/>• Event replay capabilities<br/>• Projection rebuild time | ✅ **High**:<br/>• Strict event ordering<br/>• Event versioning<br/>• Temporal consistency<br/>• Event replay ordering | ✅ **High**:<br/>• Event store persistence<br/>• CQRS fault tolerance<br/>• Event replay on failure<br/>• Snapshot management | ✅ **High**:<br/>• Event store scaling<br/>• CQRS projection scaling<br/>• Read/write separation<br/>• Independent scaling | ⚠️ **High**:<br/>• Event store setup<br/>• CQRS implementation<br/>• Projection management |
| **Event Streaming with Kafka** | ✅ **Stream Processing**:<br/>• ✅ Real-time processing<br/>• ❌ Very high complexity<br/>• ❌ Kafka infrastructure | ✅ **Low**:<br/>• 1-10ms streaming latency<br/>• Real-time processing<br/>• High throughput (1M+ events/sec)<br/>• Low-latency streaming | ✅ **High**:<br/>• Partition-based ordering<br/>• Guaranteed ordering per partition<br/>• Stream processing guarantees<br/>• Event deduplication | ✅ **High**:<br/>• Kafka fault tolerance<br/>• Partition replication<br/>• Automatic failover<br/>• Event persistence | ✅ **High**:<br/>• Kafka horizontal scaling<br/>• Partition-based scaling<br/>• High throughput (1M+ events/sec)<br/>• Auto-scaling support | ❌ **Very High**:<br/>• Kafka cluster setup<br/>• Stream processing logic<br/>• Partition management<br/>• Monitoring and tuning |


### Questions for Stakeholder Input & Our Assumptions

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


### Decision Summary

**Selected Approach: Cloud-Native Managed Services with Message Queues**

**Rationale:**
Based on our assumptions of no pre-existing infrastructure, no deep infrastructure expertise, budget minimization, and fast-paced environment with low operational overhead, we recommend a cloud-native approach using managed services.

**Key Benefits:**
- **Low Operational Overhead**: Managed services reduce maintenance burden
- **Cost Optimization**: Pay-as-you-go model aligns with budget constraints
- **No Infrastructure Expertise Required**: Managed services handle complexity
- **Fast Implementation**: Pre-built services accelerate development
- **Built-in Reliability**: Cloud providers offer 99.9%+ SLA guarantees
- **Automatic Scaling**: Handles spiky/unpredictable load patterns
- **At-Least-Once Delivery**: AWS EventBridge/SNS/SQS support this requirement
- **Idempotency Support**: Application-level idempotency implementation

**Accepted Trade-offs:**
- **Vendor Lock-in**: Reduced portability across cloud providers
- **Limited Customization**: Less control over infrastructure configuration
- **Ongoing Costs**: Operational expenses vs. upfront CapEx
- **Dependency Risk**: Reliance on cloud provider availability

**Recommended Technology Stack for MVP stage:**

- **API Architecture**:
  - RESTful Serverless API
    - Kong *(api gateway, rate limiting, abuse protection)*
    - AWS Lambda *(api endpoints with business logic)*
  - Notification Mechanisms
    - Server-Sent Events *(real-time updates for web clients)*
    - Webhooks *(event-driven notifications for integrations)*

- **Data Storage**
  - **Primary**:
    <!-- - Managed PostgreSQL: Supabase -->
    - Managed MongoDB: MongoDB Atlas *(jobs)*
    - Managed Redis: Upstash *(cache: quotas, jobs)*
    - AWS S3 IA/Glacier *(archives: logs, metrics)*

- **Message Queuing**
  - **Primary**:
    - AWS SNS FIFO + AWS SQS FIFO *(job pipeline events)*
  - **Upgrade Path**:
    - AWS SNS -> AWS EventBridge *(advanced event routing)*
    - Add Kinesis Data Streams *(high throughput, real-time processing, even-sourcing)*

- **Compute Infrastructure**
  - **Primary**:
    - AWS Lambda *(short runs e.g. handler for job creation, etc)*
    - AWS EC2 Spot *(long runs e.g. handlers for job processing, etc)*
  - **Upgrade Path**:
    - AWS Fargate Spot *(advanced orchestration & minimal operational effort)*
    - AWS ECS with EC2 Spot *(advanced control & cost optimization)*
    
- **Blockchain Integration**:
  - **Primary**:
    - Infura
  - **Upgrade Path**:
    - Add another RPC as backup *(fault tollerance)*
    - Add Alchemy *(webhook support)*
    - Add QuickNode *(lower latency)*

- **Authentication & Authorization**
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

- **Monitoring & Observability**
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

- **Billing System**
  - **Primary**
    - Lago *(open source, written in golang, mature docs)*
  - **Upgrade Path**:
    - **Build** Own simple service
    - **Buy**: Lago (managed), Alguna, Maxio, 

**Enhanced Decision Criteria**:
- **Performance Requirements**: [Latency, throughput, resource usage requirements]
- **Scalability Needs**: [Connection limits, horizontal scaling requirements]
- **Real-time Requirements**: [Event ordering, state management, network resilience needs]
- **Operational Complexity**: [Maintenance, monitoring, debugging requirements]
- **Security Requirements**: [Authentication, authorization, compliance needs]
- **Cost Considerations**: [Infrastructure, development, operational costs]
- **Team Expertise**: [Existing skills, learning curve, training requirements]


### API Governance

#### API Shape & Endpoints

- **API endpoints**

  - **System**
    - GET /system/health *(health check)*
    - GET /system/ready *(readiness check)*

  - **Jobs**
    - GET /jobs
    - GET /jobs/:id
    - GET /jobs/:id/status *(dedicated endpoint, prevents over-fetching)*
    - GET /jobs/:id/outputs *(dedicated endpoint, prevents over-fetching, paginated)*
    - GET /jobs/:id/stream *(separate endpoint for Server-Sent Events)*
    - POST /jobs *(single endpoint, handles either one or many jobs)*
    - PUT /jobs/:id/status *(dedicated endpoint, simplifies extension of status types)*
    - DELETE /jobs/:id
    - POST /jobs/reconcile *(internal endpoint, automates retying DLQ job(s))*

  - **Users**
    - GET /users
    - GET /users/:id
    - POST /users/invite
    - PUT /users/:id
    - DELETE /users/:id

  - **Webhook Management**
    - GET /webhooks
    - GET /webhooks/:id
    - POST /webhooks
    - PUT /webhooks/:id
    - DELETE /webhooks/:id

  - **Billing**
    - GET /billing/usage *(usage data i.e. quota usage)*
    - GET /billing/invoices *(issued invoices)*

  - **Analytics** *(business metrics and reporting)*
    - GET /analytics/jobs

#### Versioning Strategy

- **Options Considered**:
  - **Path-Based Versioning**:
    - ✅ CDN/cache optimization
    - ✅ Debugging excellence
    - ✅ Client simplicity
    - ❌ Maintenance overhead
      - ✅ But provides isolation
    - ❌ Client must update URLs
      - ✅ But this is manageable with SDKs
  - **Header-Based Versioning** / **Media Type Versioning**
    - ✅ Clean URLs, HTTP content negotiation
    - ❌ CDN/cache complexity
    - ❌ Harder to test in browser
  - **Query Parameter Versioning**
    - ✅ No routing changes needed
    - ✅ Zero-chnage migration to a new version 
      - ❌ But this could suddenly break production
    - ❌ CDN/cache complexity
    - ❌ Mixing concerns in URL

- **Trade-offs**: [CDN/cache friendly vs clean URLs vs simplicity vs adoption]
  - **CDN/cache Complexity vs Maintenance Overhead**
  - **HTTP content negotiation vs Debugging Excellence**
  - **HTTP content negotiation vs Client Simplicity**

- **Chosen Approach**:
  - Path-Based Versioning

- **Deprecation Notice**:
  - 12 months *(adopted industry wide)*
  - 24 months *(x2 for enterprises)*

#### Error Model
**TODO: Define comprehensive error handling with alternatives**
- **Options Considered**: [RFC7807 vs custom error format vs HTTP status only]
  - **RFC7807 Problem+JSON**
    - ✅ RFC7807 is the official standard 
    - ✅ Consistent error format across all endpoints
    - ✅ Can add custom fields and types
    - ✅ Clear error messages for developers
    - ✅ Self-documenting error types
    - ❌ Developers need to understand RFC7807
    - ❌ Larger response payload
  - **Custom Error Format**
    - ✅ Can customize format as needed
    - ✅ Smaller response payload
    - ✅ Fast to implement
    - ❌ No detailed error information
    - ❌ Hard to troubleshoot issues: no error details
  - **HTTP Status Only**
    - ✅ Very simple to implement
    - ✅ Minimal response payload
    - ✅ Quick to implement
    - ❌ No detailed error information
    - ❌ Hard to troubleshoot issues: no error details

- **Trade-offs**: [Standardization vs simplicity vs client experience]
  - **Client Simplicity vs Developer Simplicity**

- **Chosen Approach**: *Client Simplicity*

- **Error Response Format**:
  - **Content-Type**: application/problem+json
  - **Error Response Example**
    ```json
    {
      "type": "https://api.jobs.com/problems/validation-error",
      "title": "Validation Error",
      "status": 400,
      "detail": "The request body contains invalid data",
      "instance": "/api/v1/jobs",
      "traceId": "550e8400-e29b-41d4-a716-446655440000",
      "errors": [
        {
          "field": "name",
          "code": "REQUIRED",
          "message": "Name is required"
        },
        {
          "field": "email",
          "code": "INVALID_FORMAT",
          "message": "Email format is invalid"
        },
        {
          "field": "priority",
          "code": "INVALID_VALUE",
          "message": "Priority must be 'low', 'medium', or 'high'"
        }
      ],
      "documentation": "https://docs.api.jobs.com/errors/validation-error"
    }
    ```

- **HTTP status code mapping**
  
- **Client Errors (4xx)**
  - 400 `bad_request`
  - 401 `unauthorized`
  - 403 `forbidden`
  - 404 `not_found`
  - 409 `conflict`
  - 413 `payload_too_large`
  - 415 `unsupported_media_type`
  - 422 `unprocessable_entity`
  - 429 `too_many_requests`

- **Server Errors (5xx)**
  - 500 `internal_server_error`
  - 502 `bad_gateway`
  - 503 `service_unavailable`
  - 504 `gateway_timeout`

#### Idempotency Strategy

- **Options Considered**:
  - **Client-controlled with UUID**
    - ✅ Client control over retry behavior
    - ❌ Key collision risk
      - ✅ But can be prevented by using UUID v4 as exmaple
    - ❌ Client must generate key
  - **Request Content Hash-based** & **Client-controlled** *(similar to HMAC API Key)*
    - ✅ Content-based deduplication
    - ✅ Collision-resistant
    - ❌ Client must generate key
    - ❌ Performance overhead
  - **Request Content Hash-based** & **Server-managed**
    - ✅ Cient simplicity *(as server manages deduplication)*
    - ✅ Content-based deduplication
    - ✅ Collision-resistant
    - ❌ Server must generate key based of a request content
    - ❌ Some dups can go through due to race conditions

- **Trade-offs**: [Client control vs server simplicity vs safety vs flexibility]
  - **Client Simplicity vs Client Control**
  - **Client Simplicity vs Deduplication Guarantees**

- **Chosen Approach**: *Client-controlled with UUID*
  - **Rationale** It balances client simplicity and deduplication guarantees.

- **Default Cache TTL**: 24 hours





### Platform Policies (Gateway)

#### Authentication

- **Options Considered**: 
  - **Self-hosted OIDC/OAuth 2.0 with Better-Auth**:
    - ✅ Simple setup
    - ✅ Free
    - ✅ High security
  - **Self-hosted OIDC/OAuth 2.0 Services**
    - ✅ Medium setup complexity
    - ✅ High security
    - ✅ Comprehensive audit logs
    - ❌ Requires server maintenance
  - **Managed OIDC/OAuth 2.0 Services**:
    - ✅ Simple setup
    - ✅ High security
    - ✅ Enterprise-grade compliance
    - ❌ Medium cost per user
  - **HMAC API Keys**:
    - ✅ Simple implementation
    - ✅ High security with replay protection
    - ❌ No user context and limited access management
  - **Better-Auth API Keys**:
    - ✅ Simple integration
    - ✅ Medium security
    - ❌ no HMAC/replay protection

- **Trade-offs**: 
  - **Security & Simplicity (win-win)**: Better-Auth provides good balance with OIDC/OAuth 2.0 standards while maintaining simplicity
  - **Cost vs Features**: Self-hosted Better-Auth offers low cost with comprehensive features vs managed services with higher cost but enterprise compliance
  - **Maintenance vs Control**: Better-Auth reduces maintenance overhead while providing sufficient control for our use case
  - **Compliance vs Speed**: Better-Auth enables fast implementation while providing foundation for future compliance requirements

- **Chosen Approach**: *Managed OIDC/OAuth 2.0 Service with MFA (Passkeys, TOTP) -> Auth0*
  - **Rationale**: Enterprise-grade and feature-rich auth in days

- **Auth method**: OIDC/OAuth 2.0 Authorization Code Flow with PKCE
  - **Rationale**: Most secure OAuth flow, prevents authorization code interception, supports public clients
  - **Flow**: Client redirects to authorization server → user authenticates → authorization code returned → client exchanges code for tokens
  - **PKCE**: Code challenge/verifier prevents code interception attacks

- **Token format**: JWT with RS256 signing
  - **Structure**: Standard JWT with iss, sub, aud, exp, iat, scope claims
  - **Signing**: RS256 (RSA with SHA-256) for asymmetric key validation
  - **Claims**: User ID, tenant ID, scopes, roles, expiration time
  - **Security**: Token validation with public key verification, no shared secrets

- **Token lifecycle**:
  - **Access Token**: 5-minute expiry for security, short-lived to limit exposure
  - **Refresh Token**: 30-day expiry with rotation on each use for security
  - **Rotation**: Automatic refresh token rotation prevents token reuse attacks
  - **Revocation**: Immediate token revocation on logout or security events
  - **Key Management**: RSA key pairs with 2048-bit keys, automatic key rotation every 90 days

- **Multi-factor authentication**: Passkeys (FIDO2/WebAuthn) as primary MFA
  - **Primary MFA**: Passkeys for phishing-resistant, hardware-based authentication
  - **Fallback MFA**: TOTP (Time-based One-Time Password) for compatibility
  - **Rationale**: Passkeys provide highest security with user-friendly experience, TOTP ensures broad device compatibility
  - **Implementation**: Better-Auth handles FIDO2/WebAuthn integration with fallback to TOTP
  - **Compliance**: Both methods are NIST recommended for high-security applications

#### Authorization
**Authorization model with access control analysis**

- **Options Considered**:
  - **RBAC (Role-Based Access Control)**: Users assigned to roles, roles have permissions, simple to understand and implement
  - **ABAC (Attribute-Based Access Control)**: Context-aware decisions based on user attributes, resource attributes, environment conditions
  - **OAuth 2.0 Scopes**: Fine-grained permissions embedded in access tokens, standard OAuth approach
  - **Permission-based**: Direct user-to-permission mapping, maximum flexibility but complex management
  - **ACLs (Access Control Lists)**: Resource-centric permissions, good for simple scenarios but doesn't scale well

- **Trade-offs**:
  - **Flexibility vs Simplicity**: RBAC provides good balance - flexible enough for most use cases while remaining simple to understand and implement
  - **Performance vs Granularity**: OAuth scopes provide fine-grained control with minimal performance overhead through token-based authorization
  - **Maintenance vs Security**: RBAC reduces maintenance overhead while providing sufficient security for multi-tenant isolation
  - **Standardization vs Customization**: OAuth 2.0 scopes provide industry-standard approach with broad ecosystem support

- **Chosen Approach**: *RBAC with OAuth 2.0 Scopes + Tenant Isolation*
  - **Rationale**: Combines the simplicity of RBAC with the granularity of OAuth scopes, providing both user-friendly role management and fine-grained API access control

- **Access control**: **RBAC with OAuth 2.0 scope enforcement**
  - **Role Hierarchy**: 
    - **Tenant Viewer**: Read-only access to job status and basic tenant information
    - **Tenant Developer**: Job submission, job status checking, limited user management
    - **Tenant Admin**: Full access to tenant resources, user management, billing
  - **OAuth Scopes**: 
    - **Jobs**
    - `jobs:status:read` - Read job information and status
    - `jobs:outputs:read` - Read job information, outputs (result[, error])
    - `jobs:submit` - Submit new jobs
    - `jobs:cancel` - Cancel jobs
    - `jobs:confirm` - Confirm jobs on blockchain
    - `jobs:global:reconciliation` - Execute jobs reconciliation
    - **Users**
      - `users:write` - Create and update tenant users
      - `users:read` - Read tenant users
      - `users:delete` - Remove users from tenant
    - **Billing**
      - `billing:read` - Access billing information
    - **Analytics**
      - `analytics:read` - Access tenant analytics and reports
  - **Scope-to-Role Mapping**: Each role has predefined scope combinations, preventing privilege escalation
    - **Tenant Viewer** *(own tenant only)* - **Can see job information, status, and outputs**: 
      - `jobs:status:read`
      - `users:read` 
      - `users:update` *(only self)*
    - **Tenant Developer** *(own tenant only)* - **Can see job information, status, and outputs**:
      - `jobs:status:read`
      - `jobs:outputs:read` 
      - `jobs:submit`
      - `jobs:cancel`
      - `users:read`
      - `users:update` *(only self)*
    - **Tenant Admin** *(own tenant only)* - **Full access to job data**:
      - `jobs:status:read`
      - `jobs:submit`
      - `jobs:cancel`
      - `jobs:confirm`
      - `users:read`
      - `users:update` *(only self)*
      - `users:invite`
      - `users:delete`
      - `billing:read`
      - `analytics:read` 

- **Resource isolation**: *Multi-tenant data segregation with security boundaries*
  - **Tenant Context**: All API requests include tenant ID in JWT claims, enforced at API gateway level
  - **Data Segregation**: Database queries automatically filtered by tenant ID, preventing cross-tenant data access

- **Permission model**: *Hierarchical permissions with business logic*
  - **Job Operations (tenant-scoped)**:
    - `jobs:status:read` - Read job information and status
    - `jobs:outputs:read` - Read job information and outputs `([result, error])` (potentially sensitive data access)
    - `jobs:submit` - Submit new jobs (requires job input validation)
    - `jobs:cancel` - Cancel jobs
    - `jobs:confirm` - Confirm jobs on blockchain
  - **Job Operations (global)**:
    - `jobs:global:reconciliation` - Execute jobs reconciliation
  - **User Management (tenant-scoped)**:
    - `users:invite` - Invite new tenant users
    - `users:read` - Read tenant users
    - `users:update` - Update tenant user
    - `users:delete` - Delete tenant users
  - **Billing**:
    - `billing:read` - Access billing information
  - **Analytics**:
    - `analytics:read` - Access tenant analytics and reports

#### Rate Limiting & Quotas

- **Options Considered**: 
  - **Self-hosted API Gateway**:
    - **Kong Gateway**: $0-2000/month, <3ms latency, advanced rate limiting, API gateway, strong community
    - **NGINX Plus**: $2.5K/year, <2ms latency, high performance, full control, maintenance overhead
    - **Traefik**: Free, <5ms latency, cloud-native, Docker/Kubernetes, community support
  - **Kong's Real Competitors**:
    - **NGINX Plus**: Kong's main competitor - higher performance but more complex setup
    - **Traefik**: Kong's lightweight competitor - simpler but fewer features
    - **AWS API Gateway**: Not a real competitor due to 50-100ms latency (unacceptable)
    - **Azure API Management**: Not a real competitor due to 40-80ms latency (unacceptable)
    - **Google Cloud Endpoints**: Not a real competitor due to 30-60ms latency (unacceptable)
  - **Managed Cloud Services**:
    - **AWS API Gateway**: $3.50 per million requests, 50-100ms latency, AWS integration, managed service
    - **Azure API Management**: $0.035/hour + $0.00035/request, 40-80ms latency, Azure integration, enterprise features
    - **Google Cloud Endpoints**: $3.00 per million requests, 30-60ms latency, Google Cloud integration, ML-based protection
  - **Service Mesh**:
    - **Istio Service Mesh**: Free, 5-15ms latency, Kubernetes-native, complex setup, enterprise-grade
  - **Rate Limiting Algorithms**:
    - **Token Bucket**: Allows sustained rates with burst capacity, smooth traffic handling
    - **Leaky Bucket**: Smooths traffic spikes, prevents micro-bursts, protects downstream services
    - **Sliding Window**: Accurate rate limiting but higher memory usage and complexity
    - **Fixed Window**: Simple implementation but allows burst at window boundaries
    - **Per-IP Fallback**: Infrastructure protection before authentication

- **Trade-offs**: 
  - **Algorithm Trade-offs**:
    - **Accuracy vs Performance**: Token bucket provides good accuracy with <3ms latency overhead
    - **Memory vs Complexity**: Multi-layer approach uses more memory but provides comprehensive protection
    - **Protection vs Usability**: Layered defense may block legitimate traffic but prevents abuse
  - **Provider Trade-offs**:
    - **Kong vs NGINX Plus**: Kong (better features, easier setup) vs NGINX Plus (higher performance, more complex)
    - **Kong vs Traefik**: Kong (better performance, more features) vs Traefik (simpler, cloud-native)
    - **Kong vs Cloud Providers**: Kong (performance, control) vs Cloud Providers (managed service, vendor lock-in)
  - **Performance Trade-offs**:
    - **Latency**: Kong (<3ms) vs NGINX Plus (<2ms) vs Traefik (<5ms) vs Cloud Providers (30-100ms) - **Kong wins on latency vs most competitors**
    - **Performance vs Maintenance**: Kong (<3ms, self-hosted) vs Cloud Providers (30-100ms, zero maintenance)
  - **Feature Trade-offs**:
    - **Cost vs Features**: Kong Gateway provides advanced features with reasonable cost
    - **Features vs Simplicity**: Kong (advanced features) vs Traefik (simple setup) vs NGINX Plus (high performance)

- **Chosen Approach**: *Multi-layer Rate Limiting with Token Bucket + Leaky Bucket*
  - **Rationale**: Comprehensive protection against different attack vectors while maintaining performance
  - **Layer 1 (Per-IP)**: 10 QPS fallback for unauthenticated requests (infrastructure protection)
  - **Layer 2 (Token Bucket)**: 100 RPS sustained, 200 burst per tenant (fair usage enforcement)
  - **Layer 3 (Leaky Bucket)**: 20 requests per 100ms per tenant (micro-burst protection)
  - **Implementation**: Kong Gateway for rate limiting with <3ms latency overhead
  - **Cost**: Depends on plan selection
    - **Free Plan**: $0/month (Basic rate limiting, community support) - for development/testing
    - **Pro Plan**: $250/month (Advanced rate limiting, support, monitoring) - for SMB production
    - **Enterprise Plan**: $500-2000/month (Full enterprise features, SLA, compliance) - for enterprise
  - **Upgrade Path**: Kong Free → Kong Pro → Enterprise solutions (AWS, Azure, GCP) for advanced features

- **Rate limit tiers**: 
  - **Per-IP Tier**: 10 QPS for unauthenticated requests (prevents infrastructure abuse)
  - **Per-Tenant Tier**: 100 RPS sustained, 200 burst capacity (ensures fair resource allocation)
  - **Per-Endpoint Tier**: Same as per-tenant (consistent limits across all endpoints)
  - **Global Tier**: No global limits (tenant isolation prevents cross-tenant impact)

- **Quota enforcement**: 
  - **Monthly Job Quotas**: Per-tenant, per calendar month (Julian calendar) based on pricing plan
  - **Soft Cap (80%)**: 202 Accepted with warning headers, notifications via email/in-app/dashboard
  - **Hard Cap (100%)**: 429 Too Many Requests, complete job creation blocking, read ops still allowed
  - **Quota Reset**: First day of each month at 00:00:00 UTC, simultaneous reset for all tenants
  - **Plan Changes**: 
    - **Upgrades**: New quota applied immediately (new total - already used)
    - **Downgrades**: Keep current quota until month end, new quota from next month
  - **Idempotency Handling**: Duplicate requests with idempotency keys don't count against quotas
  - **Storage**: 
    - **Primary**: Database (ACID compliance, durability, rich analytics)
    - **Cache Layer**: Redis (replication of quota counters, <2ms latency for quota checks)
    - **Architecture**: Database as source of truth, Redis as high-performance cache for quota enforcement
    - **Sync Strategy**: Database writes trigger Redis cache updates, cache miss falls back to database
  - **Increment**: Only on 2XX responses, idempotent replays don't double-charge

- **Throttling behavior**: 
  - **429 Too Many Requests**: Standard rate limit exceeded response
  - **Retry-After Headers**: Time until rate limit resets (1-60 seconds)
  - **Rate Limit Headers**: X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset
  - **Error Format**: RFC7807 with specific error codes (ip_rate_limit_exceeded, rate_limit_exceeded)
  - **Client Guidance**: Clear error messages with retry timing information

- **Quota monitoring**: 
  - **Performance Metrics**: <5ms latency overhead per layer, <3ms for Kong Gateway
  - **Effectiveness Metrics**: Layer 1 blocks 60-80% of abuse, Layer 2 handles 15-30%, Layer 3 catches 5-15%
  - **Quota Utilization**: Real-time quota usage across all tenants, usage patterns by tenant segment
  - **Capacity Planning**: Usage growth trends and capacity projections, daily/weekly/monthly usage trends
  - **Alert Thresholds**: >1% total requests blocked, >5ms latency degradation, >60% rate limiter capacity
  - **Quota Alerts**: 
    - **Soft Cap**: 80% quota usage triggers email/in-app/dashboard notifications
    - **Hard Cap**: 100% quota exceeded triggers immediate blocking and notifications
    - **System Health**: Quota system failures, latency/throughput violations
    - **Usage Patterns**: Unusual usage patterns and potential abuse detection
  - **Tenant Monitoring**: Single tenant >5% of blocks indicates abuse, 3x more blocks than others indicates unfair limits
  - **Security Alerts**: >50 requests/minute from single IP, >30% blocks from single country
  - **Error Responses**: RFC7807 compliant with specific codes (quota_exceeded, quota_system_unavailable)

#### Abuse Protection

- **Options Considered**: 
  - **Managed Cloud WAF**:
    - **Cloudflare WAF**: $20/month, <20ms latency, advanced bot protection, real-time threat intelligence, 200+ data centers, most popular SMB choice
    - **AWS WAF**: $1-5 per million requests, 50-100ms latency, extensive custom rules, AWS integration, industry standard for AWS workloads
    - **Google Cloud Armor**: $1-3 per million requests, 30-60ms latency, Google Cloud integration, ML-based protection, growing market share
    - **Azure Application Gateway WAF**: $0.025/hour + $0.008/GB, 40-80ms latency, Azure integration, OWASP protection, Microsoft ecosystem standard
  - **Enterprise Managed WAF**:
    - **Akamai WAF**: $5K-15K/month, 20-50ms latency, enterprise features, 300+ data centers, global CDN, enterprise market leader
    - **Imperva WAF**: $8K-20K/month, 30-80ms latency, compliance-focused, advanced threat intelligence, enterprise-grade, compliance leader
  - **Self-hosted WAF**:
    - **NGINX Plus**: $2.5K/year, <2ms latency, full control, custom rules, high performance, maintenance overhead
    - **F5 BIG-IP**: $15K-50K/year, 10-30ms latency, on-premises/cloud, advanced traffic management, enterprise-grade

- **Trade-offs**: 
  - **Managed vs Self-hosted**: Managed WAF (zero maintenance, vendor lock-in) vs Self-hosted (full control, maintenance overhead)
  - **Cost vs Features**: Managed Cloud ($20-5K/month) vs Enterprise Managed ($5K-20K/month) vs Self-hosted ($2.5K-50K/year)
    - **Cloudflare**: 
      - $0 (Basic: WAF, limited features)
      - $20/month (Pro: Advanced WAF, bot protection)
      - $200/month (Business: Enterprise WAF features)
      - $500/month (Enterprise: Full enterprise features, custom rules)
    - **AWS WAF**: $1-5 per million requests (usage-based)
    - **Google/Azure**: $1-3 per million requests (usage-based)
    - **Akamai**: $5K-15K/month (Enterprise)
    - **Imperva**: $8K-20K/month (Enterprise)
    - **Self-hosted**: $2.5K-50K/year (one-time + maintenance)
  - **Performance vs Maintenance**: Self-hosted (highest performance, maintenance) vs Managed (good performance, zero maintenance)
  - **Vendor Lock-in vs Integration**: Cloud providers (native integration) vs Independent (multi-cloud) vs Self-hosted (no lock-in)
  - **Global vs Regional**: Enterprise Managed (global CDN) vs Managed Cloud (regional) vs Self-hosted (single location)
  - **Compliance vs Cost**: Enterprise Managed (SOC2/PCI compliance, $5K-20K/month) vs Managed Cloud (basic compliance, $20-5K/month)
  - **Customization vs Simplicity**: Self-hosted (full customization) vs Managed (simplified management) vs Enterprise (extensive features)

- **Chosen Approach**: *Cloudflare WAF with Multi-layered Defense*
  - **Rationale**: Best value for money, <20ms latency, advanced bot protection, real-time threat intelligence
  - **Cost**: Depends on plan selection
    - **Free Plan**: $0/month (Basic WAF, limited features) - for development/testing
    - **Pro Plan**: $20/month (Advanced WAF, bot protection) - for SMB production
    - **Business Plan**: $200/month (Enterprise WAF features) - for growing businesses
    - **Enterprise Plan**: $500/month (Full enterprise features, custom rules) - for enterprise
  - **Performance**: <20ms latency with 200+ global data centers
  - **Security**: OWASP protection, advanced bot detection, DDoS mitigation included
  - **Upgrade Path**: Cloudflare → AWS WAF (if on AWS) → Enterprise WAF (if compliance required)

- **WAF rules**: **OWASP Protection with Custom Rules**
  - **Standard Protection**: SQL injection, XSS, CSRF, command injection, directory traversal
  - **Custom Rules**: Tenant-specific rate limiting, API endpoint protection
  - **Rule Management**: Automated rule updates, false positive tuning, performance optimization
  - **Monitoring**: Real-time attack detection, rule effectiveness metrics, false positive tracking

- **DDoS protection**: **Multi-layer DDoS Mitigation**
  - **Volumetric Attacks**: Rate limiting at edge, traffic shaping, bandwidth throttling
  - **Application-layer**: Request pattern analysis, connection limiting, resource exhaustion prevention
  - **Infrastructure**: Global edge network, automatic scaling, traffic distribution
  - **Response**: Automatic blocking, traffic redirection, service degradation protection

- **Bot detection**: **Advanced Behavioral Analysis**
  - **Credential Stuffing**: Login attempt pattern detection, account lockout mechanisms
  - **Scraping Protection**: Request frequency analysis, user agent validation, behavioral fingerprinting
  - **CAPTCHA Integration**: Challenge-response for suspicious activity, progressive difficulty
  - **ML Detection**: Machine learning models for bot behavior identification, continuous learning

- **Threat intelligence**: **Real-time Security Feeds**
  - **IP Reputation**: Malicious IP blocking, geographic restrictions, known attacker databases
  - **Attack Patterns**: Signature-based detection, anomaly detection, zero-day protection
  - **Update Frequency**: Real-time threat feed updates, automatic rule deployment
  - **Integration**: Cloudflare threat intelligence, third-party security feeds, custom threat data

#### Content Security

- **Options Considered**: 
  - **JSON Schema Validation**: Comprehensive validation, performance overhead, security against injection
  - **Input Sanitization**: Automatic cleaning, false positives, performance impact
  - **Content Filtering**: Malicious content detection, accuracy requirements, false positive handling
  - **Data Protection**: Encryption at rest/transit, PII handling, compliance requirements
  - **Size Limits**: Request body limits, file upload limits, DoS protection
  - **Content-Type Enforcement**: Strict media type validation, security against type confusion

- **Trade-offs**: 
  - **Security vs Performance**: Comprehensive validation provides security but adds latency overhead
  - **Accuracy vs Usability**: Strict validation prevents attacks but may block legitimate requests
  - **Maintenance vs Automation**: Manual schema management vs automated validation with false positives
  - **Compliance vs Cost**: Full data protection vs performance and cost implications
  - **Validation vs Speed**: Edge validation (fast) vs backend validation (comprehensive)

- **Chosen Approach**: *Multi-layered Content Security with Edge Validation*
  - **Rationale**: Comprehensive protection against injection attacks while maintaining performance
  - **Edge Validation**: JSON schema validation at edge for immediate rejection of malformed requests
  - **Performance**: <100ms validation timeout, pre-compiled schemas, lazy loading for large schemas
  - **Security**: Schema injection prevention, malicious pattern blocking, recursive validation protection

- **Input validation**: **JSON Schema Validation with Sanitization**
  - **Schema Validation**: Enforced at edge for all POST bodies, specific error details, 100ms timeout
  - **Sanitization**: Automatic stripping of unknown headers, query params, and body params
  - **Error Handling**: 400 Bad Request with specific validation errors, 415 Unsupported Media Type
  - **Performance**: Pre-compiled schemas, memory caching, hot reload for updates
  - **Security**: Schema integrity validation, injection attack prevention, recursive loop protection

- **Size limits**: **Comprehensive Request Limits**
  - **Request Body**: Maximum 1 MiB per request (prevents DoS, ensures reasonable payload size)
  - **Headers**: Maximum 8 KiB total headers, 4 KiB per individual header (prevents header flooding)
  - **Schema Size**: Maximum 1 MiB per schema (prevents memory exhaustion)
  - **Error Response**: 413 Payload Too Large with size limit information
  - **Business Justification**: Prevents resource exhaustion, ensures fair usage, protects infrastructure

- **Content filtering**: **Strict Content-Type Enforcement**
  - **POST Requests**: `application/json` required, strict Content-Type checking
  - **Validation**: Immediate rejection of incorrect Content-Type headers
  - **Error Response**: 415 Unsupported Media Type with supported Content-Type information
  - **Security**: Prevents type confusion attacks, ensures proper content handling
  - **Performance**: Fast Content-Type checking with minimal overhead

- **Data protection**: **Multi-layer Data Security**
  - **Encryption**: TLS 1.3 for transit, AES-256 for sensitive data at rest
  - **PII Handling**: Automatic detection and masking of personally identifiable information
  - **Compliance**: GDPR, CCPA, SOC2 compliance for data protection requirements
  - **Access Control**: Role-based access to sensitive data, audit logging for data access
  - **Retention**: Automated data retention policies, secure deletion procedures

### Metering Logic

#### Usage Events

- **Options Considered**: 
  - **API Call Events**: Track individual API calls and API handler results
    - granular billing
    - high volume
  - **Comprehensive Events**: All job lifecycle events (detailed billing, rich analytics)
  - **Custom Events**: Business-specific events (`job creation` -> `job completion` -> `job confirmation`)
    - tailored billing
    - complex implementation
  - **Custom Events**: Business-specific events
    - tailored billing
    - complex implementation

- **Trade-offs**: 
  - **Granularity vs Volume**: API call events provide granular billing but generate high event volume
  - **Billing Complexity vs Insights**: Comprehensive events provide rich data but complex billing logic
  - **Implementation vs Value**: Custom events require more work but provide business-specific value
  - **Event Types vs Business Logic**: API call events (technical) vs job lifecycle events (business-focused)

- **Chosen Approach**: *Comprehensive Job Lifecycle Events*
  - **Rationale**: Complete job lifecycle tracking enables accurate billing and rich business insights
  - **Business Value**: Enables usage-based pricing, capacity planning, and customer analytics
  - **Billing Accuracy**: Track actual resource consumption throughout job lifecycle
  - **Compliance**: Meet audit requirements with complete event trail

- **Event types**: *Job Lifecycle Events with Business Justification*
  - **Job Creation Event**: 
    - **Purpose**: track job submission and resource allocation
    - **Business Value**: Billing for job initiation, usage analytics
    - **Metrics**: priority, resource requirements, submission source, submission timestamp
  - **Job Completion Event**:
    - **Purpose**: track successful job execution and resource consumption
    - **Business Value**: Billing for actual compute time, job completion success rate analytics
    - **Metrics**: execution time, resource consumption, output size, completion timestamp
  - **Job Confirmation Event**:
    - **Purpose**: track user acknowledgment and job finalization
    - **Business Value**: billing for job confirmation, job confirmation success rate analytics
    - **Metrics**: chain id, block number, confirmation number, confirmation timestamp

- **Event schema**: *Structured Event Schema with Validation*
  - **Common Fields**: event ID, event type, tenant ID, user ID, correlation ID, job ID, job type
  - **Job Creation Schema**: priority, resource requirements, submission source, submission timestamp
  - **Job Completion Schema**: execution time, resource consumption, output size, completion timestamp
  - **Job Confirmation Schema**: confirmation method, user feedback, delivery status
  - **Validation Rules**: chain id, block number, confirmation number, confirmation timestamp

- **Event correlation**: *Job Lifecycle Traceability*
  - **Correlation ID**: Unique identifier linking all events for a single job
  - **Job ID**: Primary identifier for job lifecycle tracking
  - **Tenant ID**: Multi-tenant isolation and billing
  - **User ID**: User-specific analytics and billing
  - **Traceability**: Complete audit trail from creation to confirmation
  - **Analytics**: Job lifecycle analysis, performance metrics, user behavior

- **Event durability**: *Reliable Event Processing*
  - **At-least-once Delivery**: Guarantee events are not lost during processing
  - **Idempotency**: Handle duplicate events gracefully with idempotent processing
  - **Retry Logic**: Automatic retry for failed event processing with exponential backoff
  - **Dead Letter Queue**: Handle permanently failed events for manual review
  - **Event Ordering**: Maintain event sequence for accurate lifecycle tracking
  - **Retention Period**: 14 days default retention (AWS SQS allows 1-365 days, 14 days balances cost vs compliance)
  - **Backup Strategy**: Event replication across multiple data centers for disaster recovery

#### Billing System

- **Options Considered**: 
  - **Pricing Models**:
    - **Usage-based Pricing**: Pay per job executed (transparent, scales with value)
    - **Tiered Pricing**: Fixed monthly plans with usage limits (predictable revenue, customer choice)
    - **Hybrid Pricing**: Base subscription + usage overage (revenue stability + growth)
    - **Freemium Model**: Free tier + paid upgrades (customer acquisition, conversion funnel)
    - **Enterprise Pricing**: Custom pricing for large customers (high-value deals, relationship-based)
  - **Billing System Architecture**:
    - **Build Custom Billing**: In-house development (full control, high complexity, high cost)
    - **Modern Usage-Based Platforms**:
      - **Metronome**: Usage-based billing platform (modern, flexible, expensive)
      - **m3ter**: Usage-based billing infrastructure (connective tissue for CRM/ERP, mid-market focus)
      - **Alguna**: Unified monetization engine (AI/SaaS/FinTech focus, Y Combinator backed)
      - **Maxio**: End-to-end revenue operations (subscription + usage billing, comprehensive)
      - **Lago**: Open-source usage-based billing (developer-friendly, cost-effective)
      - **Togai**: Usage-based billing platform (flexible pricing models, API-first)
    - **Traditional Billing Platforms**:
      - **Stripe Billing**: Payment-focused billing (integrated, limited usage billing, cost-effective)
      - **Zuora**: Enterprise subscription management (enterprise-grade, complex, expensive)
      - **Recurly**: Subscription management (developer-friendly, moderate features, moderate cost)

- **Trade-offs**: 
  - **Pricing Model Trade-offs**:
    - **Revenue Predictability vs Customer Flexibility**: Fixed pricing (predictable) vs usage-based (flexible)
    - **Implementation Complexity vs Market Positioning**: Simple pricing vs competitive differentiation
    - **Customer Acquisition vs Revenue Optimization**: Freemium (acquisition) vs paid-only (revenue)
    - **Scalability vs Profitability**: Usage-based scales with growth vs fixed pricing ensures margins
  - **Billing System Trade-offs**:
    - **Customization vs Time-to-Market**: Build (full control, medium effort) vs Buy (limited customization, small effort)
    - **Cost vs Features**: Build (high initial cost, ongoing maintenance) vs Buy (recurring fees, feature limitations)
    - **Compliance vs Control**: Build (full responsibility) vs Buy (vendor handles compliance, less control)
    - **Integration vs Flexibility**: Build (perfect integration) vs Buy (API limitations, vendor dependencies)

- **Vendor Comparison Analysis**:
  - **Modern Usage-Based Platforms**:
    - **Metronome**: 
      - **Strengths**: Purpose-built for usage-based billing, flexible pricing models, real-time metering
      - **Weaknesses**: Expensive ($2K+/month), complex setup, overkill for simple use cases
      - **Best For**: High-volume usage-based billing, complex pricing models
      - **Cost**: $2,000-10,000/month (enterprise pricing)
    - **m3ter**: 
      - **Strengths**: Connective tissue for CRM/ERP, revenue leak prevention (1-3% recovery), mid-market focus
      - **Weaknesses**: Newer platform, limited enterprise features, focused on usage-based only
      - **Best For**: Mid-market companies with complex usage-based billing, revenue leak prevention
      - **Cost**: Contact for pricing (mid-market focused)
    - **Alguna**: 
      - **Strengths**: Y Combinator backed, AI/SaaS/FinTech focus, unified monetization engine, no-code pricing
      - **Weaknesses**: Newer platform, limited enterprise features, focused on specific verticals
      - **Best For**: AI companies, FinTech, modern SaaS with complex pricing models
      - **Cost**: Contact for pricing (startup-friendly)
    - **Maxio**: 
      - **Strengths**: End-to-end revenue operations, comprehensive subscription + usage billing, strong integrations
      - **Weaknesses**: Complex for simple use cases, expensive for small businesses
      - **Best For**: Growing SaaS companies, complex revenue operations
      - **Cost**: $500-5,000/month (tiered pricing)
    - **Lago**: 
      - **Strengths**: Open-source, developer-friendly, cost-effective, flexible pricing models
      - **Weaknesses**: Self-hosted complexity, limited support, requires technical expertise
      - **Best For**: Developer-focused teams, cost-conscious startups, custom requirements
      - **Cost**: Free (open-source) + hosting costs
    - **Togai**: 
      - **Strengths**: API-first, flexible pricing models, usage-based billing, developer-friendly
      - **Weaknesses**: Newer platform, limited enterprise features, smaller ecosystem
      - **Best For**: API-first companies, usage-based billing, developer-focused teams
      - **Cost**: Contact for pricing (usage-based pricing)
  - **Traditional Billing Platforms**:
    - **Stripe Billing**: 
      - **Strengths**: Integrated payment processing, global reach, developer-friendly, cost-effective
      - **Weaknesses**: Limited usage-based billing, basic subscription management
      - **Best For**: Payment processing + basic subscriptions, global businesses
      - **Cost**: 0.5% + $0.30 per transaction, $0.50 per subscription
    - **Chargebee**: 
      - **Strengths**: Comprehensive subscription management, tax handling, dunning management
      - **Weaknesses**: Complex for simple use cases, expensive for small businesses
      - **Best For**: Complex subscription businesses, enterprise customers
      - **Cost**: $249-2,499/month (tiered pricing)
    - **Zuora**: 
      - **Strengths**: Enterprise-grade, complex pricing models, revenue recognition
      - **Weaknesses**: Very expensive, complex implementation, overkill for startups
      - **Best For**: Large enterprises, complex revenue models
      - **Cost**: $10,000+/month (enterprise pricing)
    - **Recurly**: 
      - **Strengths**: Developer-friendly, good API, reasonable pricing
      - **Weaknesses**: Limited advanced features, smaller ecosystem
      - **Best For**: Mid-market businesses, developer-focused teams
      - **Cost**: $149-1,499/month (tiered pricing)
    - **Braintree**: 
      - **Strengths**: PayPal integration, global payment methods, fraud protection
      - **Weaknesses**: Limited billing features, PayPal dependency
      - **Best For**: Payment processing focus, PayPal ecosystem
      - **Cost**: 2.9% + $0.30 per transaction

- **Build vs Buy Deep Analysis**:
  - **Build Custom Billing**:
    - **Development Time**: 12-18 months (full-time team of 3-5 engineers)
    - **Initial Cost**: $500K-1M (development + infrastructure)
    - **Ongoing Cost**: $200K-400K/year (maintenance + compliance)
    - **Risks**: Compliance complexity, payment security, tax calculations, international regulations
    - **Benefits**: Perfect customization, no vendor lock-in, full control
  - **Buy Existing Solution**:
    - **Implementation Time**: 1-3 months (integration + customization)
    - **Initial Cost**: $10K-50K (setup + integration)
    - **Ongoing Cost**: $50K-200K/year (licensing + transaction fees)
    - **Risks**: Vendor lock-in, limited customization, dependency on vendor roadmap
    - **Benefits**: Faster time-to-market, proven security, compliance handled, ongoing updates
  - **Hybrid Approach (Lago + Stripe)**:
    - **Implementation Time**: 2-4 months (Lago setup + Stripe integration)
    - **Initial Cost**: $20K-50K (Lago hosting + Stripe setup + integration)
    - **Ongoing Cost**: $10K-30K/year (hosting + Stripe fees + maintenance)
    - **Risks**: Self-hosted complexity, technical expertise required, ongoing maintenance
    - **Benefits**: No vendor lock-in, full customization, cost-effective, proven payment processing

- **Pricing tiers**: **Multi-tier Pricing with Business Justification**
  - **Free Tier**: 
    - **Included**: 10 jobs/month, basic support, standard SLA
    - **Business Value**: Customer acquisition, product trial, market penetration
    - **Limitations**: No advanced features, limited support, basic SLA
  - **Starter Tier ($29/month)**:
    - **Included**: 100 jobs/month, priority support, enhanced SLA
    - **Business Value**: Entry-level paid customers, predictable revenue
    - **Target**: Small businesses, individual developers
  - **Professional Tier ($99/month)**:
    - **Included**: 500 jobs/month, priority support, advanced features, enhanced SLA
    - **Business Value**: Growing businesses, higher revenue per customer
    - **Target**: Professional, scaling startups
  - **Business Tier ($299/month)**:
    - **Included**: 2000 jobs/month, dedicated support, all features, premium SLA
    - **Business Value**: Large customers, high-value relationships
    - **Target**: Business customers, high-volume users
  - **Enterprise**:
    - **Included**: Unlimited jobs, custom features, dedicated support, custom SLA
    - **Business Value**: Strategic partnerships, maximum revenue per customer
    - **Target**: Fortune 500, strategic accounts

- **Usage metrics**: **Job-based Billing with Accuracy Requirements**
  - **Primary Metric**: Jobs-related (job creation + completion + confirmation)
  - **Secondary Metrics**: 
    - **Compute Time**: Actual execution time for resource-based pricing
    - **Data Volume**: Input/output size for storage-based pricing
    - **API Calls**: Individual API usage for granular billing
  - **Accuracy Requirements**: 
    - **Event Tracking**: 99.9% accuracy for billing events
    - **Usage Measurement**: Real-time tracking with 99.99% uptime
    - **Billing Calculation**: Automated calculation with audit trail
  - **Overage Pricing**: 
    - **Starter**: $0.50 per additional job
    - **Professional**: $0.30 per additional job
    - **Enterprise**: $0.20 per additional job

- **Invoice structure**: **Detailed Billing with Audit Trail**
  - **Line Items**: 
    - **Base Subscription**: Monthly recurring charge
    - **Usage Charges**: Overage jobs at tier-specific rates
    - **Compute Time**: Resource consumption charges
    - **Data Transfer**: Input/output data charges
    - **Support**: Priority support charges (if applicable)
  - **Calculations**: 
    - **Automated**: Real-time usage tracking and calculation
    - **Audit Trail**: Complete event history for billing verification
    - **Adjustments**: Credit for failed jobs, refunds for overages
  - **Transparency**: 
    - **Usage Dashboard**: Real-time usage tracking for customers
    - **Billing Details**: Line-by-line breakdown of charges
    - **Predictions**: Usage forecasting and cost optimization

- **Billing frequency**: **Monthly Billing with Usage Tracking**
  - **Base Subscription**: Monthly recurring billing
  - **Usage Charges**: Monthly billing for overage usage
  - **Payment Terms**: Net 30 for enterprise, immediate for others
  - **Customer Preferences**: 
    - **Self-service**: Credit card for immediate payment
    - **Enterprise**: Invoice-based billing with payment terms
    - **Usage Alerts**: Real-time notifications for usage approaching limits

#### Event Processing

- **Options Considered**: 
  - **Real-time Processing**: Immediate event processing (low latency, high complexity, high cost)
  - **Batch Processing**: Periodic event processing (high latency, low complexity, low cost)
  - **Hybrid Processing**: Real-time for critical events, batch for analytics (balanced approach)
  - **Event-driven Processing**: Event-triggered processing (responsive, complex orchestration)
  - **Stream Processing**: Continuous event stream processing (high throughput, complex setup)

- **Trade-offs**: 
  - **Latency vs Throughput**: Real-time (low latency) vs batch (high throughput)
  - **Complexity vs Cost**: Real-time (complex, expensive) vs batch (simple, cheap)
  - **Reliability vs Performance**: Event-driven (reliable) vs stream (high performance)
  - **Business Requirements vs Technical Complexity**: Billing accuracy vs implementation effort

- **Chosen Approach**: *Hybrid Event Processing with Real-time Billing*
  - **Rationale**: Real-time processing for billing events, batch processing for analytics
  - **Business Value**: Immediate billing accuracy + cost-effective analytics processing
  - **Technical Benefits**: Low latency for billing + high throughput for analytics
  - **Architecture**: Event-driven billing + batch analytics + stream processing for high-volume events

- **Event ingestion**: *Multi-source Event Collection with Reliability Guarantees*
  - **Primary Sources**: 
    - **Job Lifecycle Events**: Creation, completion, confirmation from application
    - **API Gateway Events**: Rate limiting, quota usage from Kong Gateway
    - **Infrastructure Events**: System metrics, performance data from monitoring
  - **Reliability Guarantees**: 
    - **At-least-once Delivery**: Guarantee events are not lost during ingestion
    - **Idempotency**: Handle duplicate events gracefully with deduplication
    - **Retry Logic**: Automatic retry for failed event ingestion with exponential backoff
    - **Dead Letter Queue**: Handle permanently failed events for manual review
  - **Ingestion Architecture**: 
    - **Event Bus**: AWS SNS/EventBridge + SQS for reliable event queuing with 14-day retention
    - **Event Routing**: AWS EventBridge for complex event patterns and routing
      - **Event Rules**: Route events based on content and patterns (job type, tenant, priority)
      - **Event Transformation**: Transform events for different consumers (billing, analytics, alerts)
      - **Event Filtering**: Filter events before routing to reduce processing costs
      - **Event Integration**: Connect with external services and AWS services
    - **Event Store**: Database + Redis cache for event persistence and high-performance access
    - **Event Validation**: Schema validation and business rule validation at ingestion

- **Event processing**: *Multi-tier Processing with Business Requirements*
  - **Real-time Processing**: 
    - **Billing Events**: Immediate processing for job creation, completion, confirmation
    - **Quota Events**: Real-time quota tracking and enforcement
    - **Alert Events**: Immediate processing for system alerts and notifications
    - **Processing Engine**: AWS Lambda for real-time event processing
  - **Batch Processing**: 
    - **Analytics Events**: Daily/weekly/monthly analytics processing
    - **Reporting Events**: Periodic report generation and data aggregation
    - **Audit Events**: Compliance and audit trail processing
    - **Processing Engine**: AWS Lambda + Database for batch analytics processing
  - **Event-driven Processing**: 
    - **Business Logic**: Event-triggered business rules and workflows
    - **Notifications**: Event-triggered customer notifications and alerts
    - **Workflows**: Event-driven business process automation
    - **Processing Engine**: AWS Lambda for event-driven serverless processing

- **Late arrival handling**: *Temporal Event Processing with Adjustment Logic*
  - **Event Ordering**: Maintain event sequence for accurate lifecycle tracking
  - **Late Events**: Handle events that arrive out of order or after processing window
  - **Adjustment Logic**: 
    - **Billing Adjustments**: Credit for late-arriving completion events
    - **Quota Adjustments**: Update quota usage for late-arriving events
    - **Analytics Adjustments**: Recalculate analytics for late-arriving events
  - **Temporal Windows**: 
    - **Billing Window**: 24-hour window for late-arriving billing events
    - **Analytics Window**: 7-day window for late-arriving analytics events
    - **Audit Window**: 30-day window for late-arriving audit events
  - **Reconciliation**: 
    - **Event Correlation**: Link related events across temporal windows
    - **State Management**: Maintain event state for late-arriving events
    - **Adjustment Processing**: Automated adjustment for late-arriving events

- **Reconciliation**: *Billing Accuracy Verification with Audit Requirements*
  - **Event Verification**: 
    - **Completeness Check**: Verify all expected events are received
    - **Consistency Check**: Verify event data consistency across sources
    - **Accuracy Check**: Verify billing calculations are correct
  - **Audit Requirements**: 
    - **Event Trail**: Complete audit trail for all billing events
    - **Calculation Log**: Detailed log of all billing calculations
    - **Adjustment Log**: Log of all billing adjustments and corrections
    - **Compliance**: Meet regulatory requirements for billing accuracy
  - **Reconciliation Process**: 
    - **Daily Reconciliation**: Daily verification of billing accuracy
    - **Monthly Reconciliation**: Monthly audit of billing calculations
    - **Quarterly Reconciliation**: Quarterly compliance and audit review
    - **Exception Handling**: Automated exception detection and manual review
  - **Accuracy Metrics**: 
    - **Event Accuracy**: 99.9% accuracy for billing events
    - **Calculation Accuracy**: 99.99% accuracy for billing calculations
    - **Reconciliation Accuracy**: 100% accuracy for audit reconciliation

#### Invoice Mapping

- **Options Considered**: 
  - **Simple Aggregation**: Basic usage summation (simple, limited flexibility)
  - **Complex Calculations**: Multi-factor billing with discounts and adjustments (flexible, complex)
  - **Tiered Pricing**: Volume-based pricing tiers (predictable, market standard)
  - **Volume Discounts**: Usage-based discount calculations (customer-friendly, complex)
  - **Enterprise Pricing**: Custom pricing for large customers (high-value, relationship-based)

- **Trade-offs**: 
  - **Simplicity vs Accuracy**: Simple aggregation (easy implementation, limited accuracy) vs complex calculations (accurate billing, complex implementation)
  - **Customer Value vs Revenue**: Volume discounts (customer-friendly, lower margins) vs fixed pricing (revenue-focused, higher margins)
  - **Automation vs Control**: Automated billing (efficient, less control) vs manual billing (more control, inefficient)
  - **Scalability vs Customization**: Standard pricing (scalable, limited customization) vs custom pricing (high customization, limited scalability)

- **Chosen Approach**: *Hybrid Tiered Pricing with Volume Discounts*
  - **Rationale**: Combines predictable tiered pricing with customer-friendly volume discounts
  - **Business Value**: Attracts customers with volume discounts while maintaining revenue predictability
  - **Market Positioning**: Competitive pricing that scales with customer usage
  - **Revenue Model**: Base subscription + usage overage + volume discounts

- **Line item calculation**: *Multi-factor Billing with Business Rules*
  - **Base Subscription**: Monthly recurring charge based on tier (Free, Starter, Professional, Business, Enterprise)
  - **Usage Charges**: Overage jobs at tier-specific rates ($0.20-$0.50 per job)
  - **Compute Time**: Resource consumption charges based on actual execution time
  - **Data Transfer**: Input/output data charges for large datasets
  - **Volume Discounts**: 
    - **2-4% discount**: For customers using 80-100% of tier limits (based on Salesforce, HubSpot)
    - **4-6% discount**: For enterprise customers with high volume (based on AWS, Google Cloud)
    - **6-8% discount**: For strategic partnerships (rare, case-by-case, based on Microsoft Azure, Stripe)
    - **No overage discounts**: Overage charged at full rates (market standard)
  - **Business Rules**: 
    - **Minimum Billing**: $1 minimum charge per month
    - **Overage Grace**: 1% overage allowed before additional charges (TODO: Analyze market data and customer feedback to optimize)
    - **Volume Thresholds**: Discounts apply after 30 days of consistent usage
    - **Enterprise Negotiation**: Custom pricing for strategic accounts

- **Tiered pricing**: *Volume-based Pricing with Market Analysis*
  - **Free Tier**: $0/month, 10 jobs/month (customer acquisition)
  - **Starter Tier**: $29/month, 100 jobs/month ($0.50 overage) (small businesses)
  - **Professional Tier**: $99/month, 500 jobs/month ($0.30 overage) (medium businesses)
  - **Business Tier**: $299/month, 2000 jobs/month ($0.20 overage) (large businesses)
  - **Enterprise Tier**: Custom pricing, unlimited jobs (strategic accounts)
  - **Volume Discounts**: 
    - **Starter**: 2% discount for 80-100% usage (based on Salesforce Starter)
    - **Professional**: 3% discount for 80-100% usage (based on HubSpot Professional)
    - **Business**: 4% discount for 80-100% usage (based on AWS Business)
    - **Enterprise**: 6-8% discount based on volume and relationship (based on Microsoft Azure Enterprise)
    - **No overage discounts**: Overage charged at full rates (market standard)
  - **Market Analysis**: 
    - **Competitive Pricing**: 20-30% below market leaders
    - **Value Proposition**: Transparent pricing with volume discounts
    - **Customer Acquisition**: Free tier drives adoption and conversion
    - **Revenue Growth**: Volume discounts encourage usage growth

- **Adjustments**: *Automated Credits with Approval Workflow*
  - **Automatic Credits**: 
    - **Failed Jobs**: 100% credit for jobs that fail due to system issues
    - **Overbilling**: Automatic credit for billing errors detected within 24 hours
    - **Service Outages**: Pro-rated credit for service downtime
    - **Quota Errors**: Credit for incorrect quota enforcement
  - **Manual Credits**: 
    - **Customer Disputes**: Credit for billing disputes with approval workflow
    - **Goodwill Credits**: Customer retention credits with manager approval
    - **Promotional Credits**: Marketing and sales credits with approval
    - **Refunds**: Full refunds for service cancellations within 30 days
  - **Approval Workflow**: 
    - **Automatic**: System-verified credits (failed jobs, overbilling)
    - **Manager Approval**: Credits >$100 require manager approval
    - **Director Approval**: Credits >$1000 require director approval
    - **Audit Trail**: Complete log of all credits and approvals

- **Invoice delivery**: *Multi-channel Delivery with Guarantees*
  - **Delivery Methods**: 
    - **Email**: PDF invoice sent to customer email address
    - **Portal**: Invoice available in customer dashboard
    - **API**: Programmatic access for enterprise customers
    - **Mail**: Physical invoices for enterprise customers (if requested)
  - **Delivery Guarantees**: 
    - **Email Delivery**: 99.9% delivery success rate with retry logic
    - **Portal Access**: 99.99% uptime for customer dashboard
    - **API Access**: 99.9% uptime for programmatic access
    - **Delivery Confirmation**: Email delivery receipts and read confirmations
  - **Customer Preferences**: 
    - **Self-service**: Email delivery for immediate payment
    - **Enterprise**: Portal + API access for invoice management
    - **Notifications**: Email alerts for new invoices and payment reminders
    - **Format Options**: PDF, CSV, JSON formats for different use cases

## B. System Interface & Logic Design

### Data Models

**Note**: Pricing Plan relasted model are relevant if we gonna use **open source** solution from **Lago**

- **Entities**
  - *Tenant*
  - *User*
  - *Pricing Plan Subscription*
  - *Pricing Plan Declaration*
  - *Pricing Plan Quota*
  - *Pricing Plan Quota Usage Event*

- **Relationships**: 
  - **One-to-Many**: *Tenant* → *User*
  - **One-to-One**: *Tenant* → *Pricing Plan Subscription*
  - **One-to-One**: *Pricing Plan Subscription* → *Pricing Plan Declaration*
  - **One-to-Many**: *Pricing Plan Declarati*on → *Pricing Plan Quota*
  - **One-to-Many**: *Pricing Plan Quota* → *Pricing Plan Quota Usage Event*

- **Entity Interfaces**
    ```typescript

    /**
     * Tenant
     * - it terminally captures deletion date
     */
    interface TenantEntity {
      @IsPrimaryKey()
      @IsUUID()
      id: string;
      
      @IsEnumProperty(TenantTypeEnum)
      type: TenantTypeEnum;

      @IsEnumProperty(TenantStatusEnum)
      status: TenantStatusEnum;

      @IsEmailAddressProperty()
      email_address: string;

      @IsStringProperty()
      name: string;

      @IsForeignKey()
      @IsStringProperty()
      pricing_plan_subscription_id?: string;
      
      @IsDateProperty()
      created_at: Date;
      
      @IsDateProperty({ isNullable: true })
      updated_at?: Date | null;

      @IsDateProperty({ isNullable: true })
      deleted_at?: Date | null;
    }

    /**
     * Supported transitions of tenant statuses:
     * - Active → Suspended: by client or by system admin
     * - Suspended → Active: by client or by system admin
     * - Active → Deleted: by client or by system admin (email can be reused)
     * - Suspended → Deleted: by client or by system admin (email can be reused)
     */
    enum TenantStatusEnum {
      Active = 1,
      Suspended = 2,
      Deleted = 3,
    }
    
    enum TenantTypeEnum {
      Individual = 1,
      Business = 2,
      Enterprise = 3,
    }

    /**
     * Pricing Plan Declaration for tenant subscription
     */
    interface PricingPlanDeclarationEntity {
      @IsPrimaryKey()
      @IsUUID()
      id: string;

      @IsEnumProperty(PricingPlanTypeEnum)
      type: PricingPlanTypeEnum;

      @IsDateProperty()
      created_at: Date;
      
      @IsDateProperty({ isNullable: true })
      updated_at?: Date | null;
    }

    enum PricingPlanTypeEnum {
      Free = 1,
      Starter = 2,
      Professional = 3,
      Business = 3,
      Enterprise = 3,
    }

    /**
     * Pricing Plan Quota Declaration
     * - assgined to `pricing_plan_declaration_id
     * - type URL examples:
     *   - {tenant_id}.example.com/{version}/{event_namespace}/{event_id}/{event_type}?{k}={v},{k}={v},{k}={v}
     *   - example.com/v1/jobs/007/created}?priority=1
     *   - example.com/v1/jobs/007/completed?compute_ms=10000,compute_bytes=100000
     *   - example.com/v1/jobs/007/confirmed}?chain_id=666,block_number=999,tx_hash=0x..
     *   - example.com/v1/jobs/007/failed}?error_code=555,error_message=billable_error_code,compute_ms=10000,compute_bytes=100000
     */
    interface PricingPlanQuotaDeclaration {
      @IsPrimaryKey()
      @IsUUID()
      id: string;

      @IsURL()
      type: string;

      @IsStringProperty()
      name: string;

      @IsStringProperty({ isNotEmpty: false })
      description: string;

      @IsIntegerProperty({ isGreaterThan: 0 })
      limit: number;

      @IsForeignKey()
      @IsStringProperty()
      pricing_plan_declaration_id: string;

      @IsDateProperty()
      created_at: Date;
      
      @IsDateProperty({ isNullable: true })
      updated_at?: Date | null;
    }

    /**
     * Pricing Plan Subscription
     * - assigned to `tenant_id`
     * - assigned to `pricing_plan_declaration_id`
     * - it terminally captures subscription date and cancellation date for the given pricing plan
     */
    interface PricingPlanSubscriptionEntity {
      @IsPrimaryKey()
      @IsUUID()
      id: string;

      @IsEnumProperty(PricingPlanSubscriptionStatusEnum)
      status: PricingPlanSubscriptionStatusEnum;

      @IsForeignKey()
      @IsStringProperty()
      tenant_id: string;

      @IsForeignKey()
      @IsStringProperty()
      pricing_plan_declaration_id: string;

      @IsDateProperty()
      created_at: Date;
      
      @IsDateProperty({ isNullable: true })
      updated_at?: Date | null;

      @IsDateProperty({ isNullable: true })
      subscribed_at?: Date | null;

      @IsDateProperty({ isNullable: true })
      cancelled_at?: Date | null;
    }

    /**
     * Supported transitions of pricing plan subscription statuses
     * - Active → Suspended: Payment failure, compliance issues, manual admin action
     * - Suspended → Active: Payment resolved, compliance cleared, manual admin action
     * - Active → Cancelled: Pricing plan cancelled, manual admin action
     * - Suspended → Cancelled: Pricing plan cancelled, manual admin action
     */
    enum PricingPlanSubscriptionStatusEnum {
      Active = 1,
      Suspended = 2,
      Cancelled = 3,
    }

    /**
     * Pricing Plan Quota Usage Event
     * - type URL examples:
     *   - {tenant_id}.example.com/{version}/{event_namespace}/{event_id}/{event_type}?{k}={v},{k}={v},{k}={v}
     *   - example.com/v1/jobs/007/created}?priority=1
     *   - example.com/v1/jobs/007/completed?compute_ms=10000,compute_bytes=100000
     *   - example.com/v1/jobs/007/confirmed}?chain_id=666,block_number=999,tx_hash=0x..
     *   - example.com/v1/jobs/007/failed}?error_code=555,error_message=billable_error_code,compute_ms=10000,compute_bytes=100000
     */
    interface PricingPlanQuotaUsageEventEntity {
      // Common fields
      @IsPrimaryKey()
      @IsUUID()
      id: string;

      @IsURL()
      type: string;

      @IsForeignKey()
      @IsStringProperty()
      tenant_id: string;

      @IsForeignKey()
      @IsStringProperty()
      user_id: string;

      @IsStringProperty()
      correlation_id: string;

      @IsDateProperty()
      created_at: Date;

      @IsDateProperty()
      originated_at: Date;

      // Type-dependent: order-bound fields

      @IsStringProperty({ isNullable: true })
      priority: string | null;

      // Type-dependent: resource-bound fields

      @IsIntegerProperty({ isNullable: true })
      compute_ms: number | null;

      @IsIntegerProperty({ isNullable: true })
      compute_bytes: number | null;

      // Type-dependent: blockchain-bound fields

      @IsIntegerProperty({ isNullable: true })
      chain_id: number | null;

      @IsIntegerProperty({ isNullable: true })
      block_number: number | null;

      @IsStringProperty({ isNullable: true })
      tx_hash: string | null;

      // Type-dependent: timestamp fields

      @IsDateProperty({ isNullable: true })
      job_creation_started_at?: Date | null;

      @IsDateProperty({ isNullable: true })
      job_creation_failed_at?: Date | null;

      @IsDateProperty({ isNullable: true })
      job_creation_completed_at?: Date | null;

      @IsDateProperty({ isNullable: true })
      job_processing_started_at?: Date | null;

      @IsDateProperty({ isNullable: true })
      job_processing_failed_at?: Date | null;

      @IsDateProperty({ isNullable: true })
      job_processing_completed_at?: Date | null;

      @IsDateProperty({ isNullable: true })
      job_confirmation_started_at?: Date | null;

      @IsDateProperty({ isNullable: true })
      job_confirmation_failed_at?: Date | null;

      @IsDateProperty({ isNullable: true })
      job_confirmation_completed_at?: Date | null;

      // Type-dependent: error-bound fields

      @IsIntegerProperty({ isNullable: true })
      error_code: number | null;

      @IsIntegerProperty({ isNullable: true })
      error_message: string | null;
    }

- **Billing Metric Types**:
  - **job_creation_started**: Job creation started
  - **job_creation_failed**: Job creation failed
  - **job_creation_completed**: Job creation completed
  - **job_processing_started**: Job processing started
  - **job_processing_failed**: Job processing by failed
  - **job_processing_completed**: Job processing completed
  - **job_confirmation_started**: Job confirmation started
  - **job_confirmation_failed**: Job confirmation by failed
  - **job_confirmation_completed**: Job confirmation completed
  - **quota_exceeded**: Quota limit exceeded
  - **billing_event**: Billing-related event

- **Options Considered**: Relational DBs, Document DBs, KeyValue DBs, Hybrid

- **Trade-offs**: 
  - **Availability vs Consistency**
  - **Native Document Support vs JSONB Overhead**

- **Chosen Approach**: *Hybrid Multi-Database Architecture*
  - **Rationale**: Different data types require different database technologies
  - **Business Value**: Optimal performance for each use case
  - **Technical Benefits & Purposes**:
     - *PostgreSQL* delivers strong consistency to auth data *(not relevant as we start with Auth0, BetterAuth to come later)*
     - *MongoDb* delivers native doc support for job outputs of various shapes
     - *Redis* delivers real-time usage quota data to API gateway

### API Handler Logic

**API Handler Execution Duration:**
- `job creation` - *up to 1 second*; simple db writes and event publishing;
- `job processing` - *from few seconds to minutes or hours*; job task dependent;
- `job confirmation send` - *up to few seconds*; blockchain tx signing;
- `job confirmation capture` - *up to 1 second*; simple db writes and event publishing;
- `job reconciliation` - *up to 1 second*; simple event publishing;

**API Handler Execution Flow:**
- `job-creation` handler:
    - *record telemetry*
    - **biz logic**
    - ✅ on success; **submitting a single transaction to AWS EventBridge**:
        - publish message: `record metrics`
        - publish message: `send job status`
        - publish message: `start job processing`
        - *record telemetry*
    - ❌ on error; **submitting a single transaction to AWS EventBridge**:
        - publish message: `record metrics`
        - publish message: `send job status`
        - *record telemetry*
        - handle retry **(error-context policies apply)**
        - return HTTP error
- `job-processing` handler:
    - *record telemetry*
    - **biz logic**
    - ✅ on success; **submitting a single transaction to AWS EventBridge**:
        - publish message: `record metrics`
        - publish message: `send job status`
        - publish message: `send job output`
        - publish message: `start job confirmation`
        - *record telemetry*
    - ❌ on error; **submitting a single transaction to AWS EventBridge**:
        - publish message: `record metrics`
        - publish message: `send job status`
        - *record telemetry*
        - handle retry **(error-context policies apply)**
- `job-confirmation-send` handler:
    - *record telemetry*
    - **biz logic**
      - (...)
      - generate tx
      - sign tx
      - send tx
      - validate RPC response
      - (...)
    - ✅ on success; **submitting a single transaction to AWS EventBridge**:
        - publish message: `record metrics`
        - publish message: `send job status`
        - publish message: `tx submitted`
          - triggers `job-confirmation-capture-orchestrator` handler to
        - *record telemetry*
    - ❌ on error; **submitting a single transaction to AWS EventBridge**:
        - publish message: `record metrics`
        - publish message: `send job status`
        - *record telemetry*
        - handle retry **(error-context policies apply)**
            - potentially blocking error due to *ordering requirement*
            - *manual intervention* may be required to review, resolve and resume
- `job-confirmation-capture-orchestrator` handler
    - *record telemetry*
    - check if `job-confirmation-capture` handler is running, if not, then spawn one
    - ✅ on success; **submitting a single transaction to AWS EventBridge**:
        - *record telemetry*
    - ❌ on error; **submitting a single transaction to AWS EventBridge**:
        - *record telemetry*
        - handle retry **(error-context policies apply)**
- `job-confirmation-capture` handler:
    - *record telemetry*
    - **biz logic**
      - (...)
      - subscribe for contract events for *real-time event processing*
      - preiodically (e.g. X hours) execute *event replay* for the last Y hours to capture yet unprocessed events which were missed due to potential issues
      - (...)
    - ✅ on success; **submitting a single transaction to AWS EventBridge**:
        - publish message: `record metrics`
        - publish message: `send job status`
        - trigger job confirmation output delivery flow
        - *record telemetry*
    - ❌ on error; **submitting a single transaction to AWS EventBridge**:
        - publish message: `record metrics`
        - publish message: `send job status`
        - *record telemetry*
        - handle retry **(error-context policies apply)**
- `job-reconciliation` handler
    - *record telemetry*
    - query to identify failed jobs that stuck in DLQ
    - filter jobs based of *request.body.errors_allowed_to_retry* array
    - publish messages:
        - `start job processing`
        - `start job confirmation`
    - ✅ on success; **submitting a single transaction to AWS EventBridge**:
        - publish message: `record metrics`
        - *record telemetry*
    - ❌ on error; **submitting a single transaction to AWS EventBridge**:
        - publish message: `record metrics`
        - *record telemetry*
        - handle retry **(error-context policies apply)**


### On-Chain Interface (EVM Smart Contract)

#### **Contract Upgradability**: *Trustless vs Upgradeable*

**Chosen Approach**: *Trustless (Immutable) + Migration Strategy*

**Rationale**: For a privacy-preserving job confirmation system, trustless contracts provide superior security, user trust, and regulatory compliance. The migration strategy allows for future enhancements while maintaining immutability benefits.

- **Trustless (Immutable) Contracts:**
  - Pros
    - ✅ **Security**: No admin keys or upgrade exploits  
    - ✅ **User Trust**: No central authority control  
    - ✅ **Privacy**: No admin access to job data  
    - ✅ **Gas Efficiency**: 5,700 gas per confirmation (minimal overhead)  
    - ✅ **Regulatory Compliance**: Immutable privacy guarantees  
    - ✅ **Audit Transparency**: All code visible and immutable  
    - ✅ **Decentralization**: No single point of failure  
    - ✅ **Cost**: Lower deployment and maintenance costs  
  - Cons
    - ❌ **No Bug Fixes**: Cannot fix bugs after deployment  
    - ❌ **No Feature Updates**: Cannot add new functionality  
    - ❌ **Migration Required**: Need new contract for changes  
    - ❌ **Data Loss**: Cannot migrate existing data  

- **Upgradeable Contracts (Proxy Pattern):**
  - Pros
    - ✅ **Bug Fixes**: Can fix bugs after deployment  
    - ✅ **Feature Updates**: Can add new functionality  
    - ✅ **No Migration**: Same contract address  
    - ✅ **Data Preservation**: Existing data remains  
  - Cons
    - ❌ **Gas Overhead**: ~2,100 gas per call (proxy overhead)  
    - ❌ **Admin Risk**: Admin can upgrade (centralization)  
    - ❌ **Trust Required**: Requires trust in admin  
    - ❌ **Complexity**: More complex to audit and deploy  
    - ❌ **Security Risk**: Upgrade mechanism can be exploited  
    - ❌ **Cost**: Higher deployment and maintenance costs  

- **Migration Strategy Benefits:**
  - ✅ **Clean Slate**: Each version is fully audited and immutable  
  - ✅ **User Choice**: Users can choose which version to use  
  - ✅ **No Forced Upgrades**: Users control their migration  
  - ✅ **Proven Security**: Each version is battle-tested  
  - ✅ **Regulatory Compliance**: Immutable privacy guarantees  
  - ✅ **Future-Proof**: Can add features without compromising security  

- **Recommended Approach for Jobs API:**
  1. **Phase 1 (MVP)**: Deploy trustless V1 contract
  2. **Phase 2 (Production)**: Deploy trustless V2 with enhancements
  3. **Phase 3 (Scale)**: Deploy trustless V3 with advanced features

This approach maximizes security, privacy, and user trust while allowing for future enhancements through clean migrations.

#### **Security Considerations**: *Access Control Patterns*

**Chosen Approach**: **AccessControl + Pausable Pattern**

**Rationale**: For a production job confirmation system, we need both role-based access control and emergency stop capability. The AccessControl pattern provides secure, audited permission management, while Pausable enables immediate response to security incidents. Timelock provides governance delay for admin functions but adds complexity for MVP, making it suitable for future upgrades when governance is needed.

- **Access Control Patterns Considered:**

  - **1. Confirmer Pattern (AccessControl)**
    - ✅ **Role-based security**: Multiple roles and permissions
    - ✅ **Audited**: OpenZeppelin AccessControl is battle-tested
    - ✅ **Gas efficient**: Optimized storage patterns
    - ✅ **Flexible**: Can add more roles later
    - ✅ **Standard**: Industry standard approach
    - ✅ **Production-ready**: Used by major DeFi protocols
    - ❌ **Admin risk**: Admin can grant/revoke roles
    - ❌ **Centralization**: Single admin controls permissions
    - ❌ **No emergency stop**: Cannot immediately halt contract

  - **2. Paused Pattern (Pausable)**
    - ✅ **Emergency stop**: Can pause contract immediately
    - ✅ **Security response**: Prevents further damage during investigation
    - ✅ **Recovery**: Can unpause after fix is deployed
    - ✅ **Audited**: OpenZeppelin Pausable is well-tested
    - ✅ **Immediate execution**: No delay for emergency actions
    - ❌ **Single point of failure**: Pauser can halt entire system
    - ❌ **No governance**: No community input on pause decisions
    - ❌ **Centralization**: Single pauser controls system

  - **3. Timelock Pattern (Governance)**
    - ✅ **Governance delay**: 24-48 hour delay for admin functions
    - ✅ **Community review**: All changes are visible before execution
    - ✅ **Security**: Prevents immediate admin changes
    - ✅ **Transparency**: All changes are transparent
    - ✅ **Decentralization**: Reduces admin power
    - ❌ **Complexity**: More complex to implement and maintain
    - ❌ **Slow response**: Cannot respond quickly to security issues
    - ❌ **Governance overhead**: Requires community coordination

- **Combined Security Model:**

  - **AccessControl + Pausable** *(Recommended)*
    - ✅ **Role-based permissions**: Secure access control
    - ✅ **Emergency response**: Immediate pause capability
    - ✅ **Audited components**: Both OpenZeppelin patterns are battle-tested
    - ✅ **Production-ready**: Used by major protocols
    - ✅ **Flexible**: Can add timelock later for governance
    - ✅ **Balanced**: Security + emergency response
    - ❌ **Admin risk**: Admin can grant roles and pause
    - ❌ **Centralization**: Single admin controls system

- **Security Trade-offs:**

  - **Immediate vs Delayed Response:**
    - **Paused**: Immediate response     to security issues
    - **Timelock**: Delayed response for governance changes
    - **Combined**: Immediate emergency + delayed governance

  - **Centralization vs Decentralization:**
    - **AccessControl**: Centralized role management
    - **Timelock**: Decentralized governance
    - **Combined**: Centralized emergency + decentralized governance

  - **Complexity vs Security:**
    - **Simple**: Basic access control only
    - **Advanced**: Access control + pause + timelock
    - **Balanced**: Access control + pause (recommended)

- **Security Benefits:**
  - ✅ **Role-based access**: Only authorized confirmers can confirm jobs
  - ✅ **Emergency stop**: Can pause contract if security issue found
  - ✅ **Audited patterns**: OpenZeppelin components are battle-tested
  - ✅ **Production-ready**: Used by major DeFi protocols
  - ✅ **Future-proof**: Can add timelock for governance later

#### **Contract Business Logic**

- **Options Considered**:

  - **Event-Only**
    - ✅ **Ultra-low gas**: *6,450* gas per confirmation
    - ✅ **Privacy preserving**: No correlatable data
    - ✅ **Block Timestamp**: Available from block metadata
      - ❌ Can be manipulated by miners
    - ❌ **Job confirmation check**: Using events only (off-chain)
    - ❌ **Job confirmation deduplication**: Using events only (off-chain)
    - ❌ **Job (integrity) hash verificationn**: Using events only (off-chain)
    - ❌ **Job confirmation date verification**: Using events only (off-chain)

    **Explicit Gas Breakdown** *(6,450)*:
      - CALL opcode: *2,100* gas
      - SLOAD (access control check): *2,100* gas
      - Input validation: *200* gas (jobHash + confirmedAt checks)
      - LOG3 opcode: *1,750* gas
      - Function execution: *300* gas

    **Gas Loss on Revert:**
      - If input validation fails: *2,300* gas lost (~$0.05 at 20 gwei)
        - CALL opcode: *2,100* gas (transaction initiation)
        - Input validation: *200* gas (jobHash + confirmedAt checks)
        - REVERT opcode: *0* gas (revert is free)

    ```ts
    contract EventOnlyJobConfirmation {
        // ============================================================================
        // JOB CONFIRMATION CONTRACT WITH EVENT-ONLY CONFIRMATION
        // ============================================================================
        //
        // DESIGN: Minimal on-chain state, maximum gas efficiency (6,450 gas)
        // SECURITY: Authorization + off-chain duplicate prevention
        // PRIVACY: No correlatable on-chain data, privacy-preserving identifiers
        
        // ============================================================================
        // CONSTANTS
        // ============================================================================
        
        // Input validation errors
        error InvalidJobHash();
        error InvalidConfirmedAt();
        
        // ============================================================================
        // STORAGE LAYOUT & GAS OPTIMIZATION
        // ============================================================================
        //
        // STORAGE DESIGN:
        // - No on-chain storage (event-only contract)
        // - Gas cost: 0 gas for storage operations (event-only)
        //
        // GAS OPTIMIZATION:
        // - No storage writes (SSTORE operations)
        // - Minimal on-chain state
        // - Event-only confirmation
        // - Custom errors for gas efficiency
        //
        
        // ============================================================================
        // EVENTS
        // ============================================================================
        
        /**
         * @dev Emitted when a job is confirmed
         * @param jobHash Job identifier (bytes32 hash of job data + salt)
         * @param confirmedAt Timestamp when the job was confirmed
         * 
         * GAS COST: 1,750 gas (LOG3 opcode)
         * MONITORING: Requires off-chain event indexing for duplicate detection
         */
        event JobConfirmed(bytes32 indexed jobHash, uint256 confirmedAt);
        
        // ============================================================================
        // CORE CONFIRMATION LOGIC
        // ============================================================================
        
        /**
         * @dev Confirm a job with event-only storage
         * 
         * GAS COST: 6,450 gas (CALL: 2,100 + SLOAD: 2,100 + validation: 200 + LOG3: 1,750 + execution: 300)
         * SECURITY: Authorization required, off-chain duplicate prevention
         * 
         * @param jobHash Job identifier (bytes32 hash of job data + salt)
         * @param confirmedAt Timestamp when the job was confirmed
         */
        function confirmJob(bytes32 jobHash, uint256 confirmedAt) external onlyAuthorizedConfirmer {
            if (jobHash == bytes32(0)) revert InvalidJobHash();
            if (confirmedAt == 0) revert InvalidConfirmedAt();

            emit JobConfirmed(jobHash, confirmedAt);
        }
    }
    ```

  - **Event & Mapping**
    - ❌ **Higher gas**: *26,450* gas per confirmation
    - ✅ **Privacy preserving**: Job hash built from *job data* and *salt*
      - ❌ But it still it could be a correlation factor
    - ✅ **Block Timestamp**: Available from block metadata
      - ❌ Can be manipulated by miners
    - ✅ **Job confirmation check**: Using events and on-chain state
    - ✅ **Job confirmation deduplication**: Using events and on-chain state
    - ✅ **Job (integrity) hash verification**: Using events and on-chain state
    - ❌ **Job confirmation date verification**: Using events only (off-chain)

    **Explicit Gas Breakdown** *(26,450)*:
      - CALL opcode: *2,100* gas
      - SLOAD (duplicate check): *2,100* gas
      - SSTORE (confirmation storage): *20,000* gas
      - LOG3 opcode: *1,750* gas
      - Function execution: *500* gas

    **Gas Loss on Revert:**
      - If basic validation fails: *2,300* gas lost (~$0.05 at 20 gwei)
        - CALL opcode: *2,100* gas (transaction initiation)
        - Input validation: *200* gas (jobHash + confirmedAt checks)
        - REVERT opcode: *0* gas (revert is free)
    ```ts
    contract EventAndMappingJobConfirmation {
        // ============================================================================
        // JOB CONFIRMATION CONTRACT WITH SIMPLE CONFIRMATION VIA EVENT & MAPPING IN STATE
        // ============================================================================
        //
        // DESIGN: Simple confirmation state management with efficient storage (26,450 gas)
        // SECURITY: Duplicate prevention + state validation
        // PRIVACY: Privacy-preserving identifiers with on-chain status queries
        // EFFICIENCY: Single storage slot per job with struct
        
        // ============================================================================
        // CONSTANTS
        // ============================================================================
        
        // Input validation errors
        error InvalidJobHash();
        error InvalidConfirmedAt();
        
        // State validation errors
        error AlreadyConfirmed();
        
        // ============================================================================
        // STORAGE LAYOUT & GAS OPTIMIZATION
        // ============================================================================
        //
        // STORAGE DESIGN:
        // - JobConfirmation struct: 1 storage slot (32 bytes total)
        // - confirmedAt: Slot 0 (32 bytes) - Confirmation timestamp
        // - Gas cost: 20,000 gas for first write, 5,000 gas for subsequent writes
        //
        // GAS OPTIMIZATION:
        // - Single slot struct for efficient storage
        // - Time-based validation for security
        // - Events for off-chain monitoring (1,750 gas for LOG3)
        // - No complex struct packing needed
        //
        // SECURITY CONSIDERATIONS:
        // - Time validation prevents invalid timestamps
        // - Duplicate prevention via struct existence check
        // - Job identifier (bytes32 hash of job data + salt)
        
        struct JobConfirmation {
            uint256 confirmedAt;  // Slot 0: 32 bytes - Confirmation timestamp
        }
        
        mapping(bytes32 => JobConfirmation) public confirmations;

        // ============================================================================
        // EVENTS
        // ============================================================================
        
        /**
         * @dev Emitted when a job is confirmed with time constraints
         * @param jobHash Job identifier (bytes32 hash of job data + salt)
         * @param confirmedAt Timestamp since when the job counts as confirmed
         * 
         * GAS COST: 1,750 gas (LOG3 opcode)
         * MONITORING: Enables off-chain event indexing and time-based monitoring
         */
        event JobConfirmed(bytes32 indexed jobHash, uint256 confirmedAt);
        
        
        // ============================================================================
        // CORE CONFIRMATION LOGIC
        // ============================================================================
        
        /**
         * @dev Confirm a job with time validation
         * 
         * GAS COST: 26,450 gas (CALL: 2,100 + SLOAD: 2,100 + SSTORE: 20,000 + LOG3: 1,750 + execution: 500)
         * SECURITY: Basic validation + duplicate prevention
         * 
         * BASIC VALIDATION:
         * - confirmedAt must be positive
         * 
         * @param jobHash Job identifier (bytes32 hash of job data + salt)
         * @param confirmedAt Timestamp when the job was confirmed
         */
        function confirmJob(bytes32 jobHash, uint256 confirmedAt) external {
            // Input validation (cheapest first)
            if (jobHash == bytes32(0)) revert InvalidJobHash();
            if (confirmedAt == 0) revert InvalidConfirmedAt();
            
            // Storage check (most expensive)
            if (confirmations[jobHash].confirmedAt != 0) revert AlreadyConfirmed();
             
            // Direct assignment (gas optimized)
            confirmations[jobHash].confirmedAt = confirmedAt;

            emit JobConfirmed(jobHash, confirmedAt);
        }
     
        // ============================================================================
        // STATUS QUERY FUNCTIONS
        // ============================================================================
        
        /**
         * @dev Get confirmed at timestamp
         * 
         * GAS COST: 2,100 gas (SLOAD opcode)
         * EFFICIENCY: Single storage read for complete confirmation data
         * 
         * @param jobHash Job identifier (bytes32 hash of job data + salt)
         * @return confirmedAt Timestamp when the job was confirmed
         */
        function getConfirmedAt(bytes32 jobHash) public view returns (uint256) {
            return confirmations[jobHash].confirmedAt;
        }
    }
    ```

  - **Event & Struct**
    - ❌ **Higher gas**: *30,850* gas per confirmation
    - ✅ **Privacy preserving**: Job fingerprint is a hash from *job* and *salt*
      - ❌ But it still it could be a correlation factor
    - ✅ **Block Timestamp**: Available from block metadata
      - ❌ Can be manipulated by miners
    - ✅ **Job confirmation check**: Using events and on-chain state
    - ✅ **Job confirmation deduplication**: Using events and on-chain state
    - ✅ **Job (integrity) hash verification**: Using events and on-chain state
    - ✅ **Job confirmation date verification**: Using events and on-chain state

    **Explicit Gas Breakdown** *(30,850)*:
      - CALL opcode: *2,100* gas
      - SLOAD (duplicate check): *2,100* gas
      - SLOAD (FUTURE_TOLERANCE): *2,100* gas
      - SLOAD (PAST_TOLERANCE): *2,100* gas
      - SSTORE (confirmation storage): *20,000* gas
      - LOG3 opcode: *1,750* gas
      - Function execution: *700* gas

    **Gas Loss on Revert:**
      - If time validation fails: *6,700* gas lost (~$0.13 at 20 gwei)
        - CALL opcode: *2,100* gas (transaction initiation)
        - Input validation: *100* gas (jobHash check)
        - SLOAD (FUTURE_TOLERANCE): *2,100* gas
        - SLOAD (PAST_TOLERANCE): *2,100* gas
        - Time validation logic: *300* gas
        - REVERT opcode: *0* gas (revert is free)
    ```ts


    contract EventAndStructJobConfirmation {
        // ============================================================================
        // JOB CONFIRMATION CONTRACT WITH SIMPLE CONFIRMATION VIA EVENT & STRUCT IN STATE
        // ============================================================================
        //
        // DESIGN: Simple confirmation with event emission (30,850 gas)
        // SECURITY: Duplicate prevention + time validation
        // PRIVACY: Job identifier (bytes32 hash of job data + salt)
        // EFFICIENCY: Struct-based storage with event monitoring
        
        // ============================================================================
        // CONSTANTS
        // ============================================================================
        
        // Input validation errors
        error InvalidJobHash();
        error InvalidConfirmedAt();
        
        // State validation errors
        error AlreadyConfirmed();
        
        // Constructor validation errors
        error FutureToleranceInvalid();
        error PastToleranceInvalid();
        
        // Time validation constants (configurable at deployment)
        uint256 public immutable FUTURE_TOLERANCE;  // Maximum seconds confirmedAt can be in the future
        uint256 public immutable PAST_TOLERANCE;   // Maximum seconds confirmedAt can be in the past
        
        // ============================================================================
        // CONSTRUCTOR
        // ============================================================================
        
        /**
         * @dev Constructor to set chain-specific time tolerances
         * 
         * GAS COST: 20,000 gas (constructor execution)
         * SECURITY: Set once at deployment, immutable thereafter
         * 
         * @param _futureTolerance Future tolerance in seconds
         * @param _pastTolerance Past tolerance in seconds
         */
        constructor(uint256 _futureTolerance, uint256 _pastTolerance) {
            if (_futureTolerance == 0) revert FutureToleranceInvalid();
            if (_pastTolerance == 0) revert PastToleranceInvalid();
            
            FUTURE_TOLERANCE = _futureTolerance;
            PAST_TOLERANCE = _pastTolerance;
        }
        
        // ============================================================================
        // STORAGE LAYOUT & GAS OPTIMIZATION
        // ============================================================================
        // 
        // STORAGE DESIGN:
        // - JobConfirmation struct: 1 storage slot (32 bytes total)
        // - confirmedAt: Slot 0 (32 bytes) - Confirmation timestamp
        // - Gas cost: 20,000 gas for first write, 5,000 gas for subsequent writes
        //
        // GAS OPTIMIZATION:
        // - Single slot struct for efficient storage
        // - Time-based validation for security
        // - Events for off-chain monitoring (1,750 gas for LOG3)
        // - No complex struct packing needed
        //
        // SECURITY CONSIDERATIONS:
        // - Time validation prevents invalid timestamps
        // - Duplicate prevention via struct existence check
        // - Job identifier (bytes32 hash of job data + salt)
        
        struct JobConfirmation {
            uint256 confirmedAt;  // Slot 0: 32 bytes - Confirmation timestamp
        }
        
        mapping(bytes32 => JobConfirmation) public confirmations;
        
        // ============================================================================
        // EVENTS
        // ============================================================================
        
        /**
         * @dev Emitted when a job is confirmed with time constraints
         * @param jobHash Job identifier (bytes32 hash of job data + salt)
         * @param confirmedAt Timestamp since when the job counts as confirmed
         * 
         * GAS COST: 1,750 gas (LOG3 opcode)
         * MONITORING: Enables off-chain event indexing and time-based monitoring
         */
        event JobConfirmed(bytes32 indexed jobHash, uint256 confirmedAt);
        
        // ============================================================================
        // CORE CONFIRMATION LOGIC
        // ============================================================================
        
        /**
         * @dev Confirm a job with time validation
         * 
         * GAS COST: 30,850 gas (CALL: 2,100 + SLOAD: 2,100 + SLOAD: 2,100 + SLOAD: 2,100 + SSTORE: 20,000 + LOG3: 1,750 + execution: 700)
         * SECURITY: Time validation + duplicate prevention
         * 
         * TIME VALIDATION:
         * - confirmedAt must be positive
         * - confirmedAt cannot be more than FUTURE_TOLERANCE in the future (configurable at deployment)
         * - confirmedAt cannot be older than PAST_TOLERANCE (configurable at deployment)
         * 
         * @param jobHash Job identifier (bytes32 hash of job data + salt)
         * @param confirmedAt Timestamp when the job was confirmed
         */
        function confirmJob(bytes32 jobHash, uint256 confirmedAt) external {
            // Input validation (cheapest first)
            if (jobHash == bytes32(0)) revert InvalidJobHash();
            
            // Combined time validation (gas optimized)
            if (
                confirmedAt == 0 |
                confirmedAt > block.timestamp + FUTURE_TOLERANCE |
                confirmedAt < block.timestamp - PAST_TOLERANCE
            ) {
                revert InvalidConfirmedAt();
            }
            
            // Storage check (most expensive)
            if (confirmations[jobHash].confirmedAt != 0) revert AlreadyConfirmed();
             
            // Direct assignment (gas optimized)
            confirmations[jobHash].confirmedAt = confirmedAt;

            emit JobConfirmed(jobHash, confirmedAt);
        }
        
        // ============================================================================
        // STATUS QUERY FUNCTIONS
        // ============================================================================
        
        /**
         * @dev Get confirmed at timestamp
         * 
         * GAS COST: 2,100 gas (SLOAD opcode)
         * EFFICIENCY: Single storage read for complete confirmation data
         * 
         * @param jobHash Job identifier (bytes32 hash of job data + salt)
         * @return confirmedAt Timestamp when the job was confirmed
         */
        function getConfirmedAt(bytes32 jobHash) public view returns (uint256) {
            return confirmations[jobHash].confirmedAt;
        }
    }
    ```

- **Trade-offs**:
  - **Trustless Deployment vs Upgradability**
  - **Trustless Deployment vs Pausability**
  - **Simple Mapping vs Extensible Struct**
  - **Ultra-low Cost vs On-chain State Checks**

- **Multi-Chain Cost Analysis per 1M On-chain Confirmation Transactions by implementation options**

| Blockchain | EventOnly | EventAndMapping | EventAndStruct | Best Choice |
|------------|-----------|-----------------|----------------|-------------|
| **Polygon** | $100 | $410 | $478 | EventOnly (77% savings) |
| **Harmony** | $100 | $410 | $478 | EventOnly (77% savings) |
| **Huobi** | $200 | $820 | $956 | EventOnly (77% savings) |
| **KuCoin** | $500 | $2,050 | $2,390 | EventOnly (77% savings) |
| **Cronos** | $1,000 | $4,100 | $4,780 | EventOnly (77% savings) |
| **Moonriver** | $10,000 | $41,000 | $47,800 | EventOnly (77% savings) |
| **Avalanche** | $20,000 | $82,000 | $95,600 | EventOnly (77% savings) |
| **BSC** | $30,000 | $123,000 | $143,400 | EventOnly (77% savings) |
| **Fantom** | $130,000 | $533,000 | $621,400 | EventOnly (77% savings) |
| **Ethereum** | $6,030,000 | $24,723,000 | $28,823,400 | EventOnly (77% savings) |

- **Chain-Specific Insights** (ordered by cost):
  - **Polygon/Harmony**: Cheapest options ($100 for EventOnly), ideal for high-volume applications
  - **Huobi**: Very low cost ($200 for EventOnly), good for cost-sensitive projects
  - **KuCoin**: Low cost ($500 for EventOnly), emerging ecosystem
  - **Cronos**: Moderate cost ($1,000 for EventOnly), Crypto.com ecosystem
  - **Moonriver**: Higher cost ($10,000 for EventOnly), Kusama parachain
  - **Avalanche**: Mid-range cost ($20,000 for EventOnly), good balance of features
  - **BSC**: Higher cost ($30,000 for EventOnly), but mature ecosystem
  - **Fantom**: High cost ($130,000 for EventOnly), fast finality
  - **Ethereum**: Most expensive ($6M+ for EventOnly), but highest security and decentralization
- **EventOnly savings**: Consistent 77% savings across all chains
  - **Cost range**: 60,000x difference between cheapest (Polygon/Harmony) and most expensive (Ethereum)

- **Chosen Approach**: *EventOnlyJobConfirmation with Multi-Chain Deployment Strategy and Trustless Deployment*
  - **Primary Contract**: EventOnlyJobConfirmation (6,450 gas)
  - **Deployment Strategy**: Deploy on Polygon/Harmony for cost efficiency, with Ethereum as premium option
  - **Rationale:**
    - *Gas Efficiency*:
      - 77% cost savings vs alternatives
      - **$100 (on Polygon)** vs **$6M (on Ethereum)** for **1M transactions**
    - *Security*: Event-driven audit trail with off-chain duplicate prevention
    - *Functionality*: Essential job confirmation without on-chain state complexity
    - *Scalability*:
      - Ultra-low cost enables high-volume applications
      - **0.1T transactions** at the cost of **$10K (on Polygon)**
    - *Multi-Chain*:
      - Deploy on cost-optimized chains
        - Polygon/Harmony for volume
        - Ethereum for security
    - *Trustless Deployment*:
      - **Contract Simplicity**: Minimal, simple and readable codebase reduces attack surface and audit complexity, while building user trust
      - **Contract Security**: No upgrade mechanisms, ensuring contract finality and security, eliminates upgrade-related attack vectors and admin key risks
      - **Compliance**: Immutable contracts provide audit trail and regulatory compliance benefits
      - **Cost Efficiency**: No proxy patterns or upgrade mechanisms reduce gas costs


## C. Reliability & Security Notes

### Reliability

#### **Service Level Objectives (SLOs) Analysis**

- **Options Considered**:
  - **99.9% (3 nines)**: 8.77 hours downtime/year, suitable for internal tools
  - **99.95% (3.5 nines)**: 4.38 hours downtime/year, suitable for business-critical APIs
  - **99.99% (4 nines)**: 52.6 minutes downtime/year, suitable for financial systems
  - **99.995% (4.5 nines)**: 26.3 minutes downtime/year, suitable for payment processing
  - **99.999% (5 nines)**: 5.26 minutes downtime/year, suitable for mission-critical systems

- **Trade-offs Analysis**:
  - **Cost vs Reliability**: Higher SLOs require more infrastructure, redundancy, and operational overhead
  - **Complexity vs Reliability**: Higher SLOs need more sophisticated monitoring, alerting, and incident response
  - **Customer Experience**: Higher SLOs provide better user experience but at exponentially higher costs
  - **Business Impact**: Downtime costs vary by business model - financial services need higher SLOs than content platforms

- **Chosen Approach**:
  - **SLOs**
    - *99.9%* for *POST/PUT/DELETE*
    - *99.95%* for *GET*
  - **Rationale**: 
    - POST operations are complex (job creation) and can tolerate brief outages
    - PUT operations are complex (job status updates) and can tolerate brief outages
    - DELETE operations are complex (job deletion) and can tolerate brief outages
    - GET operations are simpler (status checks, outputs) and need higher availability for user experience
    - Balanced approach between cost and reliability for a job processing API

#### **Error Budget Policy**

|------------------|----------------|---------------|------------------------------------|------------------------|--------------------------------|-------------------|
| **Option**       | **Err Budget** | **Burn Rate** | **Response Actions**               | **Pros**               | **Cons**                       | **Best For**      |
|------------------|----------------|---------------|------------------------------------|------------------------|--------------------------------|-------------------|
| *Zero-Tolerance* | 0.05% (4.4h/y) | 1.2x, 2x      | Immediate restrictions (freeze)    | Max reliability        | Restrictive, kills innovation  | Financial systems |
| *Conservative*   | 0.1% (8.8h/y)  | 2x, 10x       | Auto-reduce, freeze deployments    | Max safety             | Too restrictive, hurts growth  | Mature systems    |
| *Moderate*       | 0.1% (8.8h/y)  | 1.5x, 3x, 6x  | Learning-focused, root cause       | Balanced safety/growth | Requires more monitoring       | New services      |
| *Growth-First*   | 0.2% (17.5h/y) | 2x, 4x        | Min restrictions, focus on scaling | Optimized for growth   | Higher error rates acceptable  | Startups          |
| *Aggressive*     | 0.3% (26.3h/y) | 2x, 5x        | Min restrictions, focus on fixes   | Rapid iteration        | Higher risk tolerance          | Internal services |
| *Adaptive*       | 0.1%→0.05%     | 1.5x, 3x, 6x  | Start loose, tighten as matures    | Data-driven            | Complex to implement           | New services      |
|------------------|----------------|---------------|------------------------------------|------------------------|--------------------------------|-------------------|

- **Error Budget Definition** *(Moderate Approach for New Service)*:
  - **HIGH Criticality** (50% buffer): 0.1% error budget (8.77 hours/year) - *Zero tolerance for SLO violations during critical launch phase*
  - **MEDIUM Criticality** (35% buffer): 0.1% error budget (8.77 hours/year) - *Consistent with new service learning approach*
  - **LOW Criticality** (15% buffer): 0.1% error budget (8.77 hours/year) - *Unified error budget for new service simplicity*
  
  **Rationale**: New services require consistent, conservative error budgets across all criticality levels to ensure zero tolerance for SLO violations during the critical launch phase, while allowing for learning and optimization.

- **Endpoint-Specific Error Budgets** *(All 0.1% for New Service Simplicity)*:
  - **Jobs API** (HIGH): 0.1% (8.77h/year) - *Revenue-critical, zero tolerance*
  - **Users API** (HIGH): 0.1% (8.77h/year) - *Customer-facing, zero tolerance*
  - **Billing API** (HIGH): 0.1% (8.77h/year) - *Financial data, zero tolerance*
  - **Webhooks API** (HIGH): 0.1% (8.77h/year) - *Critical path, zero tolerance*
  - **Analytics API** (MEDIUM): 0.1% (8.77h/year) - *Consistent with new service approach*
  - **System API** (LOW): 0.1% (8.77h/year) - *Unified approach for simplicity*

- **Burn Rate Policies** *(Learning-Oriented)*:
  - **Normal Burn Rate**: 1x (within SLO) - *Baseline performance*
  - **Learning Burn Rate**: 1.5x (new service discovery phase) - *Expected during learning, monitor and document*
  - **High Burn Rate**: 3x (investigate but don't panic) - *Alert on-call, continue learning*
  - **Critical Burn Rate**: 6x (90% of error budget consumed) - *Page engineer, focus on root cause*

- **Response Actions**:

  - **0.5x Burn Rate**: *Under-utilized budget*
    - Review SLO targets for optimization
    - Consider tightening error budget
    - Monitor for cost efficiency opportunities

  - **1.5x Burn Rate**: *Learning phase*
    - Monitor closely and document patterns
    - Set 30-day learning phase timeline
    - Define success criteria for learning exit
    - No action required, while in learning phase

  - **3x Burn Rate**: *High burn rate*
    - Alert on-call engineer 
    - Investigate root cause within 24 hours
    - Continue learning while investigating
    - Set escalation criteria if no resolution
      - See *Incident Response Procedures* below
  
  - **6x Burn Rate**: *Critical situation*
    - Page on-call engineer immediately 
    - Escalate to engineering manager
      - See *Incident Response Procedures* below
    - Focus on root cause analysis
    - Avoid auto-reduce rate limits (hurts growth)
    - Avoid freeze deployments (prevents iteration)

  - **10x+ Burn Rate**: *Emergency situation*
    - Activate emergency response team
    - Implement immediate rollback procedures
    - Communicate with stakeholders
      - See *Incident Response Procedures* below
    - Plan post-incident review


#### **Error Budget Policy Examples And Formulas**
  
  - **Example 0**

    - **Error Budget Rate**: *0.1% budget rate*
    - **Daily Error Rate**: *0.12% actual rate* i.e. *1.2x daily burn rate*

    - **Daily Burn Rate**: *1.2x daily burn rate*
      - Formaula: `Daily Error Rate` ÷ `Error Budget Rate`
      - Example: `0.12% daily error rate` ÷ `0.1% error budget rate`

    - **Annual Error Budget**: *8.77 hours/year*

    - **Daily Error Budget**: *0.024 hours/day*
      - Formaula: `Annual Error Budget` ÷ 365 days
      - Example: `8.77 hours/year` ÷ 365 days

    - **Daily Error Hours**: *0.029 hours/day*
      - Formaula: `Daily Error Budget` × `Daily Burn Rate`
      - Example: `0.024 hours/day` × `1.2x`

    - **Consumed Error Budget**: *0.87 hours*
      - Formaula: `Daily Error Hours` × 30 days
      - Example: `0.029 hours/day` × 30 days

    - **Remaining Error Budget**: *7.90 hours/year* i.e. *90%*
      - Formaula: (`Annual Error Budget` - `Consumed Error Budget`) ÷ `Annual Error Budget` × 100%
      - Example: (`8.77 hours/year` - `0.87 hours`) ÷ `8.77 hours/year` × 100%

    - **Time to Exhaustion**: *272 days*
      - Formaula: `Remaining Error Budget` ÷ `Daily Error Hours`
      - Example: `7.90 hours/year` ÷ `0.029 hours/day`

  - **Example 1: Normal Operation (Day 30)**
    - Annual Error Budget: 8.77 hours/year
    - Daily Error Budget: 8.77h ÷ 365 = 0.024 hours/day
    - Daily Error Rate: *0.075% (below budget)*
    - Daily Burn Rate: 0.075% ÷ 0.1% = *0.75x*
    - Consumed Error Budget: 30 days × 0.024h × 0.75 = 0.54 hours
    - Remaining Error Budget: (8.77h - 0.54h) ÷ 8.77h × 100% = 94%
    - Time to Exhaustion: 8.23h ÷ (0.024h × 0.75) = 456 days
    - Status: ✅ Healthy

  - **Example 2: Learning Phase (Day 45)**
    - Annual Error Budget: 8.77 hours/year
    - Daily Error Budget: 0.024 hours/day
    - Daily Error Rate: *0.15% (above budget)*
    - Daily Burn Rate: 0.15% ÷ 0.1% = *1.5x*
    - Consumed Error Budget: 45 days × 0.024h × 1.5 = 1.62 hours
    - Remaining Error Budget: (8.77h - 1.62h) ÷ 8.77h × 100% = **81%**
    - Time to Exhaustion: 7.15h ÷ (0.024h × 1.5) = 198 days
    - Status: ⚠️ **Learning Phase (expected)**

  - **Example 3: High Burn Rate (Day 60)**
    - Annual Error Budget: 8.77 hours/year
    - Daily Error Budget: 0.024 hours/day
    - Daily Error Rate: *0.3% (3x budget)*
    - Daily Burn Rate: 0.3% ÷ 0.1% = *3x*
    - Consumed Error Budget: 60 days × 0.024h × 3 = 4.32 hours
    - Remaining Error Budget: (8.77h - 4.32h) ÷ 8.77h × 100% = **51%**
    - Time to Exhaustion: 4.45h ÷ (0.024h × 3) = 62 days
    - Status: 🚨 **High Burn Rate (investigate)**

  - **Example 4: Critical Situation (Day 75)**
    - Annual Error Budget: 8.77 hours/year
    - Daily Budget: 0.024 hours/day
    - Daily Error Rate: *0.6% (6x budget)*
    - Daily Burn Rate: 0.6% ÷ 0.1% = *6x*
    - Consumed Error Budget: 75 days × 0.024h × 6 = 10.8 hours
    - Remaining Error Budget: (8.77h - 10.8h) ÷ 8.77h × 100% = **-23%**
    - Time to Exhaustion: 0 days (budget exhausted)
    - Status: 🔥 **Critical (page engineer immediately)**


#### **Incident Response Procedures**

- **Response Procedures**:
  1. **Detection**: Automated alerts trigger on-call
  2. **Assessment**: Determine severity and impact
  3. **Communication**: Update stakeholders via Slack/email
  4. **Mitigation**: Implement temporary fixes
  5. **Resolution**: Deploy permanent fixes
  6. **Post-mortem**: Document lessons learned

- **Communication Plan**:
  - **Internal**: Slack channels, email updates
  - **External**: Status page, customer notifications
  - **Escalation**: Phone calls for critical incidents

- **Multi-Trigger Combinations**:
  - **Time + Performance**: IMMEDIATE escalation (duration-based triggers)
  - **Business + System**: 15-minute escalation (P1 critical priority)
  - **All Triggers**: 15-minute escalation (P1 emergency priority)
  - **Performance + Error**: 15-minute escalation (P1 technical priority)
  - **Business + Customer**: 15-minute escalation (P1 impact priority)

|-------------------------|------------------------|-----------------------|-------------------|--------------------|-------------------|
| **Trigger Type**        | **Level 1**            | **Level 2**           | **Level 3**       | **Level 4**        | **Response Time** |
|-------------------------|------------------------|-----------------------|-------------------|--------------------|-------------------|
| **Time Trigger**        | 24h → On-call SWE      | 48h → Senior SWE      | 72h → Manager     | 96h+ → CTO/VP      | IMMEDIATE         |
| **Performance Trigger** | 4x → On-call SWE       | 5x → Senior SWE       | 6x → Manager      | 8x+ → CTO/VP       | 1h/30m/15m/15m    |
| **Customer Complaint**  | 1+ → On-call SWE       | 3+ → Senior SWE       | 5+ → Manager      | 10+ → CTO/VP       | 30m/15m/15m/15m   |
| **Business Impact**     | Customer → On-call SWE | Revenue → Senior SWE  | SLA → Manager     | Executive → CTO/VP | 30m/15m/15m/15m   |
| **Response Time**       | >500ms → On-call SWE   | >1s → Senior SWE      | >2s → Manager     | >5s → CTO/VP       | 30m/15m/15m/15m   |
| **Error Rate**          | >0.8% → On-call SWE    | >1.0% → Senior SWE    | >1.2% → Manager   | >1.5% → CTO/VP     | 30m/15m/15m/15m   |
| **Availability**        | <99.9% → On-call SWE   | <99.5% → Senior SWE   | <99.0% → Manager  | <95% → CTO/VP      | 30m/15m/15m/15m   |
|-------------------------|------------------------|-----------------------|-------------------|--------------------|-------------------|

- **Escalation Level Actions**:

  - **On-Call SWE Level** (IMMEDIATE response):
    - **Handles**: Individual triggers (Time, Performance, Customer complaints, Business impact, Response time, Error rate, Availability)
    - **Specific Actions**:
      - Initial incident assessment and triage
      - Basic troubleshooting and monitoring
      - Documentation of incident details
      - Escalation to senior engineer if needed

  - **Senior SWE Level** (15 minutes response):
    - **Handles**: Complex technical issues, advanced troubleshooting, architecture problems
    - **Specific Actions**:
      - Advanced technical investigation
      - Architecture review and system analysis
      - Implementation of technical fixes
      - Escalation to manager if needed

  - **Manager Level** (15 minutes response):
    - **Handles**: Business impact coordination, resource allocation, stakeholder communication
    - **Specific Actions**:
      - Resource allocation and team coordination
      - Technical investigation and root cause analysis
      - Stakeholder communication and status updates
      - Implementation of immediate fixes

  - **CTO/VP Level** (15 minutes response):
    - **Handles**: All Triggers (critical situations), company-wide issues, strategic decisions
    - **Specific Actions**:
      - Company-wide coordination and communication
      - Strategic decision making and resource allocation
      - Customer communication and relationship management
      - Post-incident review and process improvement 


#### **Monitoring and Observability**

##### **API Endppoints Latency Breakdown** + **Static** *p95* **Estimation** 

  - **GET /system/health** *(System Health Check)*:
    - **Availability**: 99.5% (43.8 hours downtime/year)
    - **Latency**: *(200 OK response, basic health status)*
      - **Targets**:
        - p95 ≤ 50ms *(Buffer: 15% - LOW business criticality factor applied)*
      - **Breakdown** *(Request Flow - Internal Health Check)*:
        - **Network Layer**:
          - 1-3ms: Local network latency *(Same VPC/region communication)*
          - 0ms: No SSL/TLS *(Internal health checks use HTTP)*
          - 0.5-1ms: Network jitter (overhead) *(Minimal internal network)*
        - **Application Layer**:
          - 0.5-1ms: OpenTelemetry instrumentation start *(OTEL span creation)*
          - 1-2ms: Health check logic *(Basic system checks - memory, CPU, app state)*
          - 1-2ms: Response generation *(JSON serialization with health status)*
          - 0.5-1ms: OpenTelemetry instrumentation end *(OTEL span completion, CloudWatch logs)*
        - **Total**: 3-8ms
        - **Missing Factors**: Peak load handling, Resource contention

  - **GET /system/ready** *(System Readiness Check)*:
    - **Availability**: 99.5% (43.8 hours downtime/year)
    - **Latency**: *(200 OK response, dependency health status)*
      - **Targets**:
        - p95 ≤ 100ms *(Buffer: 15% - LOW business criticality factor applied)*
      - **Breakdown** *(Request Flow - Internal Readiness Check)*:
        - **Network Layer**:
          - 1-3ms: Local network latency *(Same VPC/region communication)*
          - 0ms: No SSL/TLS *(Internal readiness checks use HTTP)*
          - 0.5-1ms: Network jitter (overhead) *(Minimal internal network)*
        - **Application Layer**:
          - 0.5-1ms: OpenTelemetry instrumentation start *(OTEL span creation)*
          - 15-35ms: External dependency readiness checks *(Critical service health verification)*
            - 5-10ms: MongoDB Atlas connection check *(Database connectivity and query execution)*
            - 3-6ms: Upstash Redis connection check *(Cache connectivity and basic operations)*
            - 4-8ms: AWS SNS health check *(Message queue connectivity and permissions)*
            - 4-8ms: AWS SQS health check *(Queue service connectivity and permissions)*
            - 3-5ms: Auth0 service check *(Authentication service connectivity and token validation)*
          - 2-5ms: Readiness status aggregation *(Dependency health compilation)*
          - 1-3ms: Response generation *(JSON serialization with health details)*
          - 0.5-1ms: OpenTelemetry instrumentation end *(OTEL span completion, CloudWatch logs)*
        - **Total**: 20-45ms
        - **Missing Factors**: Peak load handling, Resource contention
        - **Business Justification**: Ensures all critical external dependencies are operational before accepting traffic, preventing cascading failures

  - **POST /jobs** *(Job Creation)*:
    - **Availability**: 99.9% (8.77 hours downtime/year)
    - **Latency**: *(202 Accepted response, starts async pipeline)*
      - **Targets**:
        - p95 ≤ 263ms *(Buffer: 50% - HIGH business criticality factor applied)*
      - **Breakdown** *(Request Flow - Client to Server)*:
        - **Network Layer**:
          - 10-50ms: Geographic distribution *(Cross-region latency)*
          - 5-15ms: SSL/TLS handshake *(New connections only)*
          - 2-5ms: Network jitter (overhead) *(Network latency)*
        - **Load Balancer Layer**:
          - 1-3ms: Load balancer health checks *(Health check overhead)*
        - **Gateway Layer**:
          - 1-3ms: WAF protection *(Gateway security factor - modern WAFs)*
          - 0.5-2ms: Rate limiting check *(Redis-based rate limiting)*
          - 1-3ms: Quota validation *(Database quota checks with proper indexing)*
        - **Application Layer**:
          - 1-2ms: OpenTelemetry instrumentation start *(OTEL span creation)*
          - 2-5ms: Authentication/authorization *(JWT validation, OAuth2)*
          - 3-8ms: Request validation *(Request processing factor)*
            - 3-5ms: 1-10KB
            - 5-8ms: 10-100KB
            - 6-8ms: 100-256KB
          - 1-3ms: Idempotency check *(Redis-based idempotency)*
          - 1-3ms: Cache miss penalties *(Redis cache misses)*
          - 2-5ms: Database connection pooling *(Connection overhead)*
          - 5-15ms: Database load (job creation) *(Optimized database operations)*
            - *high value operation:* **job creation**
            - *5-10ms could be saved* if async via SNS/SQS
            - *keeping it sync* for **immediate client feedback**
          - 10-50ms: AWS SNS publish *(Event notifications)*
          - 1-3ms: Response generation *(JSON serialization)*
          - 1-2ms: OpenTelemetry instrumentation end *(OTEL span completion, CloudWatch logs)*
        - **Total**: 46-175ms
        - **Missing Factors**: Peak load handling, Resource contention

  - **PUT /jobs/:id** *(Job Status Update)*:
    - **Availability**: 99.9% (8.77 hours downtime/year)
    - **Latency**: *(200 OK response, immediate status update)*
      - **Targets**:
        - p95 ≤ 383ms *(Buffer: 50% - HIGH business criticality factor applied)*
      - **Breakdown** *(Request Flow - Client to Server)*:
        - **Network Layer**:
          - 10-50ms: Geographic distribution *(Cross-region latency)*
          - 5-15ms: SSL/TLS handshake *(New connections only)*
          - 2-5ms: Network jitter (overhead) *(Network latency)*
        - **Load Balancer Layer**:
          - 1-3ms: Load balancer health checks *(Health check overhead)*
        - **Gateway Layer**:
          - 1-3ms: WAF protection *(Gateway security factor - modern WAFs)*
          - 0.5-2ms: Rate limiting check *(Redis-based rate limiting)*
          - 1-3ms: Quota validation *(Database quota checks with proper indexing)*
        - **Application Layer**:
          - 1-2ms: OpenTelemetry instrumentation start *(OTEL span creation)*
          - 2-5ms: Authentication/authorization *(JWT validation, OAuth2)*
          - 3-8ms: Request validation *(Request processing factor)*
            - 3-5ms: 1-10KB
            - 5-8ms: 10-100KB
            - 6-8ms: 100-256KB
          - 1-3ms: Idempotency check *(Redis-based idempotency)*
          - 1-3ms: Cache miss penalties *(Redis cache misses)*
          - 2-5ms: Database connection pooling *(Connection overhead)*
          - 10-30ms: Job lookup *(Database lookup factor)*
          - 5-10ms: Status validation *(Business logic factor)*
          - 30-80ms: Database load (job status update) *(Optimized database operations)*
            - *high value operation*: **job status update**
            - *25-65ms could be saved* if async via SNS/SQS
            - *keeping it sync* for **immediate client feedback**
          - 10-20ms: Event emission *(Event processing factor)*
          - 1-3ms: Response generation *(JSON serialization)*
          - 1-2ms: OpenTelemetry instrumentation end *(OTEL span completion, CloudWatch logs)*
        - **Total**: 95-255ms
        - **Missing Factors**: Peak load handling, Resource contention

  - **DELETE /jobs/:id** *(Job Deletion)*:
    - **Availability**: 99.9% (8.77 hours downtime/year)
    - **Latency**: *(200 OK response, immediate deletion)*
      - **Targets**:
        - p95 ≤ 365ms *(Buffer: 35% - MEDIUM business criticality factor applied)*
      - **Breakdown** *(Request Flow - Client to Server)*:
        - **Network Layer**:
          - 10-50ms: Geographic distribution *(Cross-region latency)*
          - 5-15ms: SSL/TLS handshake *(New connections only)*
          - 2-5ms: Network jitter (overhead) *(Network latency)*
        - **Load Balancer Layer**:
          - 1-3ms: Load balancer health checks *(Health check overhead)*
        - **Gateway Layer**:
          - 1-3ms: WAF protection *(Gateway security factor - modern WAFs)*
          - 0.5-2ms: Rate limiting check *(Redis-based rate limiting)*
          - 1-3ms: Quota validation *(Database quota checks with proper indexing)*
        - **Application Layer**:
          - 1-2ms: OpenTelemetry instrumentation start *(OTEL span creation)*
          - 2-5ms: Authentication/authorization *(JWT validation, OAuth2)*
          - 3-8ms: Request validation *(Request processing factor)*
            - 3-5ms: 1-10KB
            - 5-8ms: 10-100KB
            - 6-8ms: 100-256KB
          - 1-3ms: Idempotency check *(Redis-based idempotency)*
          - 1-3ms: Cache miss penalties *(Redis cache misses)*
          - 2-5ms: Database connection pooling *(Connection overhead)*
          - 10-30ms: Job lookup *(Database lookup factor)*
          - 5-10ms: Deletion validation *(Business logic factor)*
          - 20-60ms: Database load (job deletion) *(Optimized database operations)*
            - *high value operation*: **job deletion**
            - *15-45ms could be saved* if async via SNS/SQS
            - *keeping it sync* for **immediate client feedback**
          - 20-50ms: Cleanup operations *(Cleanup processing factor)*
          - 10-20ms: Event emission *(Event processing factor)*
          - 1-3ms: Response generation *(JSON serialization)*
          - 1-2ms: OpenTelemetry instrumentation end *(OTEL span completion, CloudWatch logs)*
        - **Total**: 115-270ms
        - **Missing Factors**: Peak load handling, Resource contention

  - **GET /jobs/:id** *(Job: Basic Details)*:
    - **Availability**: 99.9% (8.77 hours downtime/year)
    - **Latency**: *(200 OK response, job status data)*
      - **Targets**:
        - p95 ≤ 210ms *(Buffer: 50% - HIGH business criticality factor applied)*
      - **Breakdown** *(Request Flow - Client to Server)*:
        - **Network Layer**:
          - 10-50ms: Geographic distribution *(Cross-region latency)*
          - 5-15ms: SSL/TLS handshake *(New connections only)*
          - 2-5ms: Network jitter (overhead) *(Network latency)*
        - **Load Balancer Layer**:
          - 1-3ms: Load balancer health checks *(Health check overhead)*
        - **Gateway Layer**:
          - 1-3ms: WAF protection *(Gateway security factor - modern WAFs)*
          - 0.5-2ms: Rate limiting check *(Redis-based rate limiting)*
          - 1-3ms: Quota validation *(Database quota checks with proper indexing)*
        - **Application Layer**:
          - 1-2ms: OpenTelemetry instrumentation start *(OTEL span creation)*
          - 2-5ms: Authentication/authorization *(JWT validation, OAuth2)*
          - 3-8ms: Request validation *(Request processing factor)*
            - 3-5ms: 1-10KB
            - 5-8ms: 10-100KB
            - 6-8ms: 100-256KB
          - 1-3ms: Idempotency check *(Redis-based idempotency)*
          - 1-3ms: Cache miss penalties *(Redis cache misses)*
          - 2-5ms: Database connection pooling *(Connection overhead)*
          - 2-5ms: Cache lookup (Redis) *(Cache hit optimization)*
          - 10-30ms: Database load (cache miss fallback) *(Optimized database operations)*
          - 5-15ms: Data serialization *(Data processing factor)*
          - 1-3ms: Response generation *(JSON serialization)*
          - 1-2ms: Monitoring/metrics collection *(Observability overhead)*
        - **Total**: 55-140ms
        - **Missing Factors**: Peak load handling, Resource contention

  - **GET /jobs/:id/outputs** *(Job Outputs)*:
    - **Availability**: 99.9% (8.77 hours downtime/year)
    - **Latency**: *(200 OK response, job outputs data)*
      - **Targets**:
        - p95 ≤ 218ms *(Buffer: 50% - HIGH business criticality factor applied)*
      - **Breakdown** *(Request Flow - Client to Server)*:
        - **Network Layer**:
          - 10-50ms: Geographic distribution *(Cross-region latency)*
          - 5-15ms: SSL/TLS handshake *(New connections only)*
          - 2-5ms: Network jitter (overhead) *(Network latency)*
        - **Load Balancer Layer**:
          - 1-3ms: Load balancer health checks *(Health check overhead)*
        - **Gateway Layer**:
          - 1-3ms: WAF protection *(Gateway security factor - modern WAFs)*
          - 0.5-2ms: Rate limiting check *(Redis-based rate limiting)*
          - 1-3ms: Quota validation *(Database quota checks with proper indexing)*
        - **Application Layer**:
          - 1-2ms: OpenTelemetry instrumentation start *(OTEL span creation)*
          - 2-5ms: Authentication/authorization *(JWT validation, OAuth2)*
          - 3-8ms: Request validation *(Request processing factor)*
            - 3-5ms: 1-10KB
            - 5-8ms: 10-100KB
            - 6-8ms: 100-256KB
          - 1-3ms: Idempotency check *(Redis-based idempotency)*
          - 1-3ms: Cache miss penalties *(Redis cache misses)*
          - 2-5ms: Database connection pooling *(Connection overhead)*
          - 2-5ms: Cache lookup (Redis) *(Cache hit optimization)*
          - 10-30ms: Database load (cache miss fallback) *(Optimized database operations)*
          - 5-15ms: Data serialization *(Data processing factor)*
          - 1-3ms: Response generation *(JSON serialization)*
          - 1-2ms: OpenTelemetry instrumentation end *(OTEL span completion, CloudWatch logs)*
        - **Total**: 57-145ms
        - **Missing Factors**: Peak load handling, Resource contention



  - TODO: *(Remaining API Endpoints)*

    - GET /jobs/:id/status *(dedicated endpoint, prevents over-fetching)*
    - GET /jobs/:id/stream *(separate endpoint for Server-Sent Events)*
    - POST /jobs/reconcile *(internal endpoint, automates retying DLQ job(s))*

    - GET /users
    - GET /users/:id`
    - POST /users/invite
    - PUT /users/:id
    - DELETE /users/:id

    - GET /webhooks
    - GET /webhooks/:id
    - POST /webhooks
    - PUT /webhooks/:id
    - DELETE /webhooks/:id

    - GET /billing/usage *(usage data i.e. quota usage)*
    - GET /billing/invoices *(issued invoices)*

    - GET /analytics/jobs

##### **Business Criticality**: *Market Benchmarks* for *Levels*, *Buffer Ranges*, and *Buffer Median*

|------------------------|-----------------------------|--------------------------------|-----------------------------|
| **Source**             | **HIGH**: *Rang* / *Median* | **MEDIUM**: *Range* / *Median* | **LOW**: *Range* / *Median* |
|------------------------|-----------------------------|--------------------------------|-----------------------------|
| *Google SRE*           | 40-50% / 45%                | 25-35% / 30%                   | 0-15% / 7.5%                |
| *Netflix SRE*          | 45-55% / 50%                | 30-35% / 37.5%                 | 5-15% / 10%                 |
| *AWS Well-Architected* | 40-50% / 45%                | 25-30% / 27.5%                 | 0-10% / 5%                  |
|------------------------|-----------------------------|--------------------------------|-----------------------------|

##### **Business Criticality**: *Our Guidelines*

|-------------------|----------|------------|---------|----------------------------------------------------------------------------------------------------------|
| **Buffer Type**   | **HIGH** | **MEDIUM** | **LOW** | **Use Case**                                                                                             |
|-------------------|----------|------------|---------|----------------------------------------------------------------------------------------------------------|
| **Buffer Lower**  | 40%      | 25%        | 0%      | **Cost-Optimized** - Mature systems, predictable load, budget constraints, acceptable SLO violation risk |
| **Buffer Median** | 45%      | 30%        | 7.5%    | **Standard Production** - Balanced risk/reward, most common use case, recommended default                |
| **Buffer Upper**  | 50%      | 35%        | 15%     | **Safety-Critical** - New systems, unpredictable load, zero tolerance for SLO violations                 |
|-------------------|----------|------------|---------|----------------------------------------------------------------------------------------------------------|

**Chosen Business Criticality Buffer**: *Buffer Upper* (50%/35%/15%)
  - **Rationale**:
    As a new service with no performance history, and zero tolerance for SLO violations during launch phase,
    We select conservative *Buffer Upper* approach to ensure safety margins for unknown system behavior,
    while continuous data collection enables rapid optimization based on real performance patterns rather than waiting for arbitrary 30-day periods.

**Latency Measurement Process**:
  1. **Launch with conservative estimates** using *Buffer Upper*, then collect production data continuously
  2. **Collect production data** continuously (minimum 7 days for initial validation)
  3. **Calculate actual ratios** for each endpoint from real performance data
  4. **Refine SLOs** based on measured values and business requirements
  5. **Re-validate** quarterly with new data for continuous optimization

|-------------------------------|------------|------------------|------------------------------------------------------------------------------------------------------------|
| **Step**                      | **Metric** |  **Formula**     | **Description**                                                                                            |
|-------------------------------|------------|------------------|------------------------------------------------------------------------------------------------------------|
| **1. Measure p0 latency**     | *p0*       | [measured_prod]  | **Fastest response time** - Best case performance, represents optimal system conditions                    |
| **2. Measure p50 latency**    | *p50*      | [measured_prod]  | **Median response time** - 50% of requests are faster, represents typical user experience                  |
| **3. Measure p90 latency**    | *p90*      | [measured_prod]  | **90th percentile** - 90% of requests are faster, represents good performance baseline for monitoring      |
| **4. Measure p95 latency**    | *p95*      | [measured_prod]  | **95th percentile** - 95% of requests are faster, represents worst-case for most users (SLO target)        |
| **5. Measure p99 latency**    | *p99*      | [measured_prod]  | **99th percentile** - 99% of requests are faster, represents tail latency for edge cases and optimization  |
| **6. Measure p100 latency**   | *p100*     | [measured_prod]  | **Slowest response time** - Worst case performance, represents system under stress conditions              |
| **8. Biz Criticality Buffer** | *BCB*      | *Buffer Upper*   | **Biz risk tolerance** - Buffer picked based on system maturity, load predictability, biz impact tolerance |
| **9. Define p95 SLO Target**  | *p95*      | p100 × (1 + BCB) | **SLO target with safety margin** - Conservative estimate for new service launch with biz risk protection  |
|-------------------------------|------------|------------------|------------------------------------------------------------------------------------------------------------|


##### **Business Criticality**: *Classification Decision Matrix*:

|-------------------------|-----------------------------------|--------------------------------|--------------------------------|
| **Factor**              | Level: **HIGH** *(40-55%)*        | Level: **MEDIUM** *(25-35%)*   | Level: **LOW** *(0-15%)*       |
|-------------------------|-----------------------------------|--------------------------------|--------------------------------|
| **Customer Impact**     | ✅ Direct customer interaction    | ⚠️ Limited customer interaction | ❌ No customer interaction     |
| **Revenue Impact**      | ✅ Direct revenue generation      | ⚠️ Indirect revenue support     | ❌ No revenue impact           |
| **Data Sensitivity**    | 🔒 Critical data (PII, financial) | 📊 Standard business data       | 🏠 Internal data only          |
| **Dependencies**        | 🔗 Critical path service          | 🔧 Supporting service           | 🔌 Independent service         |
| **Downtime Tolerance**  | ❌ Zero tolerance                 | ⚠️ Brief outages acceptable     | ✅ Extended outages acceptable |
| **Recovery Time**       | ⚡ < 5 minutes                     | ⏱️ < 30 minutes                 | 🕐 < 4 hours                   |
| **SLA Requirements**    | 📋 Strict SLA compliance          | 📄 Standard SLA compliance      | 📝 No SLA requirements         |
| **Incident Response**   | 🚨 Immediate response                 | 📞 Standard response            | 📧 Low priority response       |
| **Monitoring Priority** | 🔴 Critical alerts                | 🟡 Standard monitoring          | 🟢 Basic monitoring            |
|-------------------------|-----------------------------------|--------------------------------|--------------------------------|

##### **Business Criticality**: *Classification Decision Tree*

```ts
1. PRIMARY CLASSIFICATION (Required):
   ├─ Customer Impact = Direct + Revenue Impact = Direct → HIGH
   ├─ Customer Impact = Limited + Revenue Impact = Indirect → MEDIUM  
   └─ Customer Impact = None + Revenue Impact = None → LOW

2. SECONDARY VALIDATION (Override if applicable):
   ├─ Data Sensitivity = Critical → Promote to HIGH
   ├─ Dependencies = Critical path → Promote to HIGH
   └─ Recovery Time < 5 minutes → Promote to HIGH

3. CONFLICT RESOLUTION:
   ├─ If factors conflict, use HIGHEST applicable level
   ├─ If still unclear, default to MEDIUM for safety
   └─ Document reasoning for audit trail
```

##### **Business Criticality**: *API Endpoint Classification*

|--------------------------------------|-----------|------------|---------------------------------------------------------------------------------------------|
| **Endpoint**                         | **Level** | **Buffer** | **Justification**                                                                           |
|--------------------------------------|-----------|------------|---------------------------------------------------------------------------------------------|
| *GET* **/system/health**             | LOW       | 15%        | ❌ No customer interaction + ❌ No revenue impact + 🏠 Internal service                      |
| *GET* **/system/ready**              | LOW       | 15%        | ❌ No customer interaction + ❌ No revenue impact + 🏠 Internal service                      |
|--------------------------------------|-----------|------------|---------------------------------------------------------------------------------------------|
| *POST*   **/jobs**                   | **HIGH**  | **50%**    | ✅ Direct customer interaction + ✅ Direct revenue generation + 🔗 Critical path service     |
| *PUT*    **/jobs/{id}**              | **HIGH**  | **50%**    | ✅ Direct customer interaction + ✅ Direct revenue generation + 🔗 Critical path service     |
| *GET*    **/jobs/{id}**              | **HIGH**  | **50%**    | ✅ Direct customer interaction + ✅ Direct revenue generation + 🔗 Critical path service     |
| *DELETE* **/jobs/{id}**              | *MEDIUM*  | *35%*      | ⚠️ Limited customer interaction + ⚠️ Indirect revenue support + 🔧 Supporting service        |
| *GET*    **/jobs/{id}/outputs**      | **HIGH**  | **50%**    | ✅ Direct customer interaction + ✅ Direct revenue generation + 🔗 Critical path service     |
| *GET*    **/jobs/{id}/outputs/{id}** | *MEDIUM*  | *35%*      | ⚠️ Limited customer interaction + ⚠️ Indirect revenue support + 🔧 Supporting service        |
| *POST*   **/jobs/reconcile**         | LOW       | 15%        | ❌ No customer interaction + ❌ No revenue impact + 🏠 Internal service                      |
|--------------------------------------|-----------|------------|---------------------------------------------------------------------------------------------|
| *POST*   **/users**                  | **HIGH**  | **50%**    | ✅ Direct customer interaction + ✅ Direct revenue generation + 🔗 Critical path service     |
| *GET*    **/users/{id}**             | **HIGH**  | **50%**    | ✅ Direct customer interaction + ✅ Direct revenue generation + 🔗 Critical path service     |
| *PUT*    **/users/{id}**             | **HIGH**  | **50%**    | ✅ Direct customer interaction + ✅ Direct revenue generation + 🔗 Critical path service     |
| *DELETE* **/users/{id}**             | *MEDIUM*  | *35%*      | ⚠️ Limited customer interaction + ⚠️ Indirect revenue support + 🔧 Supporting service        |
|--------------------------------------|-----------|------------|---------------------------------------------------------------------------------------------|
| *POST* **/webhooks/events**          | **HIGH**  | **50%**    | ✅ Direct customer interaction + ✅ Direct revenue generation + 🔗 Critical path service     |
| *GET*  **/webhooks/subscriptions**   | *MEDIUM*  | *35%*      | ⚠️ Limited customer interaction + ⚠️ Indirect revenue support + 🔧 Supporting service        |
|--------------------------------------|-----------|------------|---------------------------------------------------------------------------------------------|
| *POST* **/billing/charges**          | **HIGH**  | **50%**    | ✅ Direct customer interaction + ✅ Direct revenue generation + 🔒 Critical data (financial) |
| *GET*  **/billing/invoices**         | **HIGH**  | **50%**    | ✅ Direct customer interaction + ✅ Direct revenue generation + 🔒 Critical data (financial) |
|--------------------------------------|-----------|------------|---------------------------------------------------------------------------------------------|
| *GET*  **/analytics/usage**          | *MEDIUM*  | *35%*      | ⚠️ Limited customer interaction + ⚠️ Indirect revenue support + 📊 Standard business data    |
| *GET* **/analytics/reports**         | *MEDIUM*  | *35%*      | ⚠️ Limited customer interaction + ⚠️ Indirect revenue support + 📊 Standard business data    |
|--------------------------------------|-----------|------------|---------------------------------------------------------------------------------------------|


##### **API Endpoint Latency Measurements (p0/p50/p99/100)** + **Estimated** *p95* **Targets**:

|--------------------------------------|-----------|------------|------------------------------------------------|
| **Endpoint**                         | **Level** | **Buffer** | **p0** | **50** | **99** | **100** | **p95**   |
|--------------------------------------|-----------|------------|------------------------------------------------|
| *GET* **/system/health**             | LOW       | 15%        |        |        |        |         | **50ms**  |
|--------------------------------------|-----------|------------|------------------------------------------------|
| *POST*   **/jobs**                   | **HIGH**  | **50%**    |        |        |        |         | **263ms** |
| *PUT*    **/jobs/{id}**              | **HIGH**  | **50%**    |        |        |        |         | **383ms** |
| *GET*    **/jobs/{id}**              | **HIGH**  | **50%**    |        |        |        |         | **210ms** |
| *DELETE* **/jobs/{id}**              | *MEDIUM*  | *35%*      |        |        |        |         | **365ms** |
| *GET*    **/jobs/{id}/outputs**      | **HIGH**  | **50%**    |        |        |        |         | **218ms** |
| *GET*    **/jobs/{id}/outputs/{id}** | *MEDIUM*  | *35%*      |        |        |        |         | **190ms** |
| *POST*   **/jobs/reconcile**         | LOW       | 15%        |        |        |        |         | **120ms** |
|--------------------------------------|-----------|------------|------------------------------------------------|
| *POST*   **/users**                  | **HIGH**  | **50%**    |        |        |        |         | **280ms** |
| *GET*    **/users/{id}**             | **HIGH**  | **50%**    |        |        |        |         | **200ms** |
| *PUT*    **/users/{id}**             | **HIGH**  | **50%**    |        |        |        |         | **320ms** |
| *DELETE* **/users/{id}**             | *MEDIUM*  | *35%*      |        |        |        |         | **250ms** |
|--------------------------------------|-----------|------------|------------------------------------------------|
| *POST* **/webhooks/events**          | **HIGH**  | **50%**    |        |        |        |         | **300ms** |
| *GET*  **/webhooks/subscriptions**   | *MEDIUM*  | *35%*      |        |        |        |         | **180ms** |
|--------------------------------------|-----------|------------|------------------------------------------------|
| *POST* **/billing/charges**          | **HIGH**  | **50%**    |        |        |        |         | **350ms** |
| *GET*  **/billing/invoices**         | **HIGH**  | **50%**    |        |        |        |         | **220ms** |
|--------------------------------------|-----------|------------|------------------------------------------------|
| *GET*  **/analytics/usage**          | *MEDIUM*  | *35%*      |        |        |        |         | **200ms** |
| *GET* **/analytics/reports**         | *MEDIUM*  | *35%*      |        |        |        |         | **250ms** |
|--------------------------------------|-----------|------------|------------------------------------------------|


#### **Alerting**

|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **Metric**                   | **Severity** | **Threshold**     | **Duration** | **Response**           | **Business Impact**          |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **Error Rate**               | Critical     | > 0.1%            | 2 minutes    | Immediate response     | User experience degraded     |
| **Error Rate**               | Warning      | > 0.05%           | 5 minutes    | Investigate & monitor  | Potential issues             |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **Availability**             | Critical     | < 99.9%           | 1 minute     | Immediate response     | Service degradation          |
| **Availability**             | Warning      | < 99.95%          | 3 minutes    | Monitor closely        | Performance impact           |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **Latency p95**              | Critical     | > 500ms           | 2 minutes    | Immediate response     | Performance impact           |
| **Latency p95**              | Warning      | > 300ms           | 5 minutes    | Performance monitoring | User experience              |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **POST /jobs Latency**       | Critical     | > 263ms           | 1 minute     | Immediate response     | Job creation degraded        |
| **POST /jobs Latency**       | Warning      | > 200ms           | 3 minutes    | Monitor job creation   | Job creation slow            |
| **PUT /jobs Latency**        | Critical     | > 383ms           | 1 minute     | Immediate response     | Job update degraded          |
| **PUT /jobs Latency**        | Warning      | > 300ms           | 3 minutes    | Monitor job updates    | Job update slow              |
| **GET /jobs Latency**        | Critical     | > 210ms           | 1 minute     | Immediate response     | Job retrieval degraded       |
| **GET /jobs Latency**        | Warning      | > 150ms           | 3 minutes    | Monitor job retrieval  | Job retrieval slow           |
| **DELETE /jobs Latency**     | Critical     | > 365ms           | 1 minute     | Immediate response     | Job deletion degraded        |
| **DELETE /jobs Latency**     | Warning      | > 250ms           | 3 minutes    | Monitor job deletion   | Job deletion slow            |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **POST /users Latency**      | Critical     | > 280ms           | 1 minute     | Immediate response     | User creation degraded       |
| **POST /users Latency**      | Warning      | > 200ms           | 3 minutes    | Monitor user creation  | User creation slow           |
| **GET /users Latency**       | Critical     | > 200ms           | 1 minute     | Immediate response     | User retrieval degraded      |
| **GET /users Latency**       | Warning      | > 150ms           | 3 minutes    | Monitor user retrieval | User retrieval slow          |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **POST /billing Latency**    | Critical     | > 350ms           | 1 minute     | Immediate response     | Billing degraded             |
| **POST /billing Latency**    | Warning      | > 250ms           | 3 minutes    | Monitor billing        | Billing slow                 |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **Webhook Delivery Latency** | Critical     | > 300ms           | 1 minute     | Immediate response     | Webhook degraded             |
| **Webhook Delivery Latency** | Warning      | > 200ms           | 3 minutes    | Monitor webhooks       | Webhook slow                 |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **Dependency Health**        | Critical     | < 100%            | 1 minute     | Immediate response     | External service issues      |
| **Dependency Health**        | Warning      | < 95%             | 3 minutes    | Monitor dependencies   | Potential cascading failures |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **Memory Usage**             | Critical     | > 90%             | 2 minutes    | Immediate response     | Resource exhaustion          |
| **Memory Usage**             | Warning      | > 80%             | 5 minutes    | Resource monitoring    | Performance degradation      |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **CPU Usage**                | Critical     | > 90%             | 2 minutes    | Immediate response     | Resource exhaustion          |
| **CPU Usage**                | Warning      | > 80%             | 5 minutes    | Resource monitoring    | Performance degradation      |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **Database Failures**        | Critical     | > 5%              | 1 minute     | Immediate response     | Database issues              |
| **Database Failures**        | Warning      | > 1%              | 3 minutes    | Database monitoring    | Potential data issues        |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **Job Processing Rate**      | Critical     | < 50% baseline    | 2 minutes    | Immediate response     | Job processing degraded      |
| **Job Processing Rate**      | Warning      | < 80% baseline    | 5 minutes    | Monitor job queue      | Potential job backlog        |
| **Job Completion Rate**      | Critical     | < 95%             | 2 minutes    | Immediate response     | Job completion issues        |
| **Job Completion Rate**      | Warning      | < 98%             | 5 minutes    | Monitor job success    | Job quality degradation      |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **Billing Event Loss**       | Critical     | > 1%              | 1 minute     | Immediate response     | Revenue impact               |
| **Billing Event Loss**       | Warning      | > 0.1%            | 3 minutes    | Monitor billing        | Potential revenue loss       |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **Webhook Delivery**         | Critical     | < 95% success     | 2 minutes    | Immediate response     | Webhook delivery issues      |
| **Webhook Delivery**         | Warning      | < 98% success     | 5 minutes    | Monitor webhooks       | Webhook reliability          |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **Tenant Isolation**         | Critical     | Any cross-tenant  | 0 minutes    | Immediate response     | Security breach              |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **API Rate Limit**           | Warning      | > 80% of limit    | 5 minutes    | Monitor usage          | Approaching limits           |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **Quota Exhaustion**         | Critical     | > 95% of quota    | 1 minute     | Immediate response     | Quota exceeded               |
| **Quota Exhaustion**         | Warning      | > 80% of quota    | 3 minutes    | Monitor quotas         | Approaching limits           |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **Network I/O**              | Critical     | > 90% bandwidth   | 2 minutes    | Immediate response     | Network saturation           |
| **Network I/O**              | Warning      | > 80% bandwidth   | 5 minutes    | Monitor network        | Network congestion           |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **Disk I/O**                 | Critical     | > 90% utilization | 2 minutes    | Immediate response     | Disk saturation              |
| **Disk I/O**                 | Warning      | > 80% utilization | 5 minutes    | Monitor disk           | Disk performance degradation |
| **Disk Space**               | Critical     | > 90% full        | 1 minute     | Immediate response     | Disk space exhaustion        |
| **Disk Space**               | Warning      | > 80% full        | 3 minutes    | Monitor disk space     | Approaching limits           |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **Authentication Failures**  | Critical     | > 5%              | 1 minute     | Immediate response     | Auth system issues           |
| **Authentication Failures**  | Warning      | > 2%              | 3 minutes    | Monitor auth           | Potential auth issues        |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **Rate Limit Violations**    | Critical     | > 10%             | 1 minute     | Immediate response     | API abuse detected           |
| **Rate Limit Violations**    | Warning      | > 5%              | 3 minutes    | Monitor rate limits    | Potential abuse              |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **Database Connection Pool** | Critical     | > 90%             | 1 minute     | Immediate response     | DB connection exhaustion     |
| **Database Connection Pool** | Warning      | > 80%             | 3 minutes    | Monitor DB connections | Approaching limits           |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **Cache Hit Rate**           | Critical     | < 80%             | 2 minutes    | Immediate response     | Cache performance degraded   |
| **Cache Hit Rate**           | Warning      | < 90%             | 5 minutes    | Monitor cache          | Cache efficiency low         |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|
| **Event Processing Lag**     | Critical     | > 5 minutes       | 1 minute     | Immediate response     | Event processing backlog     |
| **Event Processing Lag**     | Warning      | > 2 minutes       | 3 minutes    | Monitor events         | Event processing slow        |
|------------------------------|--------------|-------------------|--------------|------------------------|------------------------------|


#### **Reliability Testing**

<!-- - **Chaos Engineering**:
  - **Failure Injection**: Random service failures
  - **Load Testing**: Stress testing under high load
  - **Network Partitioning**: Simulate network failures
  - **Resource Exhaustion**: CPU, memory, disk space limits -->

<!-- - **Disaster Recovery**:
  - **RTO (Recovery Time Objective)**: 4 hours
  - **RPO (Recovery Point Objective)**: 1 hour
  - **Backup Strategy**: Daily backups with 30-day retention
  - **Failover**: Multi-region deployment with automatic failover -->

### Security

### Options Considered for `Authentication & Authorization`

| **Option** | **Initial Setup** | **Cost** | **Security** | **Access Management** | **Identity Sources** | **Maintenance** | **Compliance** | **Audit Logging** |
|------------|------------------|----------|-------------|---------------------|-------------------|-----------------|-------------------|-------------|
| **Self-hosted OIDC/OAuth 2.0 with Better-Auth** | ✅ Simple (Better-Auth plugin integration) | ✅ Low (infrastructure hosting only) | ✅ High (OIDC/OAuth 2.0 standard, secure, token refresh, token expiration) | ✅ High (OAuth scopes, roles, token revocation) | ✅ High (SAML, LDAP, social login, magic links, SMS/Email OTP) | ✅ Simple (Better-Auth handles OIDC server, user management) | ❌  Low (no compliance certifications yet) | ❌ N/A (not in-built yet, manual implementation required) |
| **Self-hosted OIDC/OAuth 2.0 Services** | ⚠️ Medium (Docker/K8s setup, configuration, modern tooling helps) | ✅ Low (infrastructure hosting only) | ✅ High (OIDC/OAuth 2.0 standard, secure, token refresh, token expiration) | ✅ High (OAuth scopes, roles, token revocation) | ✅ High (SAML, LDAP, social login, magic links, SMS/Email OTP, certificate-based) | ⚠️ Medium (server maintenance, modern solutions help) | ✅ High (SOC2, GDPR, HIPAA with proper configuration) | ✅ High (comprehensive audit logs, enterprise-grade) |
| **Managed OIDC/OAuth 2.0 Services** | ✅ Simple (Auth0/AWS Cognito/Azure AD integration) | ⚠️ Medium (per-user pricing, usage-based) | ✅ High (OIDC/OAuth 2.0 standard, secure, token refresh, token expiration, enterprise-grade, compliance) | ✅ High (OAuth scopes, roles, token revocation) | ✅ High (SAML, LDAP, social login, magic links, SMS/Email OTP, certificate-based, enterprise SSO) | ✅ Simple (managed service, automatic updates) | ✅ High (SOC2, GDPR, HIPAA, PCI DSS certified) | ✅ High (built-in audit logs, compliance reporting) |
| **HMAC API Keys** | ✅ Simple (Custom implementation) | ✅ Low (infrastructure hosting only) | ✅ High (HMAC signatures, replay protection, tamper detection, non-repudiation) | ❌ Low (no user context) | ❌ N/A (no identity sources, just API keys) | ⚠️ Medium (custom key management, manual rotation) | ❌ Low (no compliance features, extensive custom work required) | ❌ N/A (manual implementation required) |
| **Better-Auth API Keys** | ✅ Simple (Better-Auth library integration) | ✅ Low (infrastructure hosting only) | ⚠️ Medium (API keys, rate limiting, expiration, no HMAC/replay protection) | ⚠️ Medium (scopes, custom metadata, user context, but API key level not full OAuth 2.0) | ❌ N/A (no identity sources, just API keys) | ✅ Simple (Better-Auth handles key management, user management) | ❌ Low (no compliance certifications yet) | ❌ N/A (not in-built yet, manual implementation required) |

### Options Considered for `Multi-Factor Authentication (MFA)`

| **Option** | **Initial Setup** | **Cost** | **Security** | **Maintenance** | **Compliance** | **Audit Logging** |
|------------|------------------|----------|-------------|-----------------|-------------------|-------------|
| **MFA with Email OTP** | ✅ Simple (authentication service integration) | ✅ Low (email service costs, typically included in auth services) | ⚠️ Medium (email interception, account compromise risks) | ✅ Simple (authentication services typically handle Email OTP implementation) | ❌ Low (NIST recommends against email for high-security applications) | ✅ High (standard MFA audit logging) |
| **MFA with SMS OTP** | ✅ Simple (authentication service integration) | ✅ Low (SMS service costs, typically included in auth services) | ⚠️ Medium (SMS interception, SIM swapping risks) | ✅ Simple (authentication services typically handle SMS OTP implementation) | ❌ Low (NIST recommends against SMS for high-security applications) | ✅ High (standard MFA audit logging) |
| **MFA with TOTP** | ✅ Simple (authentication service integration) | ✅ None (no additional infrastructure) | ✅ High (time-based OTP, cryptographic security) | ✅ Simple (authentication services typically handle TOTP implementation) | ✅ High (RFC 6238 standard, NIST recommended for high-security applications) | ✅ High (standard MFA audit logging) |
| **MFA with Passkeys (FIDO2/WebAuthn)** | ✅ Simple (authentication service integration) | ✅ None (no additional infrastructure) | ✅ High (hardware security, phishing-resistant) | ✅ Simple (authentication services typically handle FIDO2/WebAuthn implementation) | ✅ High (FIDO2/WebAuthn standards, NIST recommended) | ✅ High (standard MFA audit logging) |
| **MFA with Push Notifications** | ✅ Simple (authentication services integration) | ✅ Low (push service costs, typically included in auth services) | ✅ High (device-based, user confirmation) | ✅ Simple (authentication services typically handle Push Notifications implementation) | ✅ High (NIST recommended for high-security applications) | ✅ High (standard MFA audit logging) |
| **MFA with Hardware Tokens** | ⚠️ Medium (hardware distribution, setup) | ⚠️ Medium (hardware costs per token) | ✅ High (hardware-based, offline security) | ⚠️ Medium (hardware management, distribution) | ✅ High (NIST recommended for high-security applications) | ✅ High (standard MFA audit logging) |

**Note**
*Chosen optiona and ratonale are mtioned above* at **Platform Policies (Gateway)**: **Authentication** + **Authorization**


<!-- ### Operational Excellence
**TODO: Define operational requirements with deployment analysis**
- **Options Considered**: [Blue-green vs Rolling vs Canary vs Feature flags vs A/B testing]
- **Trade-offs**: [Risk vs speed vs complexity vs cost vs reliability]
- **Chosen Approach**: [Blue-green with canary with rationale]
- Deployment: [How to deploy safely with rollback procedures]
- Monitoring: [Observability requirements with alerting thresholds]
- Incident management: [How to handle problems with escalation matrix]
- Capacity planning: [How to scale the system with growth projections] -->

<!-- ## D. Implementation Plan

### Phase 1: Core API
**TODO: Define MVP scope**
- Essential endpoints: [Which endpoints to build first]
- Basic authentication: [Minimal auth implementation]
- Core job processing: [Basic job lifecycle]
- Simple monitoring: [Basic observability]

### Phase 2: Enhanced Features
**TODO: Define second phase**
- Advanced features: [Rate limiting, quotas, etc.]
- Enhanced security: [MFA, advanced auth]
- Improved monitoring: [Comprehensive observability]
- Performance optimization: [Caching, optimization]

### Phase 3: Production Readiness
**TODO: Define production requirements**
- Production hardening: [Security, performance, reliability]
- Compliance: [Regulatory requirements]
- Documentation: [API docs, runbooks]
- Training: [Team knowledge transfer] -->

<!-- ## E. Risks & Mitigation

### Technical Risks
**TODO: Identify technical risks**
- Risk 1: [Description and impact]
  - Mitigation: [How to reduce risk]
  - Contingency: [What to do if risk materializes]
- Risk 2: [Description and impact]
  - Mitigation: [How to reduce risk]
  - Contingency: [What to do if risk materializes]

### Business Risks
**TODO: Identify business risks**
- Risk 1: [Description and impact]
  - Mitigation: [How to reduce risk]
  - Contingency: [What to do if risk materializes]
- Risk 2: [Description and impact]
  - Mitigation: [How to reduce risk]
  - Contingency: [What to do if risk materializes] -->

## F. Reflection

### If I had more time
- **Challenge Question**: "If I had more time: What is the one area of this design you would flesh out in more detail, and why?"
- **Primary Focus Area**: 
  - I would run through the comprehensive questionnaire to understand our constraints and goals better.
  - The table with all the questions and some current assumptions is provided above, look up for "Questions for Stakeholder Input & Our Assumptions".
- **Why This Area**: Sufficient context is critical to make good enough decisions and get the ball rolling.
- **Business Impact**: Having sufficient context helps to cut through the noise, avoid over-engineering, and deliver real business value to clients.
- **Technical Depth**: I would like to gather understanding of what we really sell in the given case and how we can deliver it to the client in the fastest way possible.

<!-- ### AI coding assistance
- **Challenge Question**: "AI coding assistance: If you used tools like Copilot or ChatGPT to help generate the OpenAPI spec or other parts of the design, what worked well and what did not?"
- **What Worked Well**: [What worked well when using AI tools? What specific use cases were most effective?]
- **What Didn't Work**: [What didn't work well or required manual intervention? What were the limitations?]
- **Quality Impact**: [How did AI assistance impact the quality and speed of the design? What were the trade-offs?]
- **Best Practices**: [What patterns emerged for effective AI tool usage? What would you recommend to others?]
- **Tool Selection**: [Which AI tools were most valuable for different aspects of the design? Why?] -->

---

## Appendix

### OpenAPI Specification
**TODO: Reference OpenAPI spec**
- Link to openapi.yaml file
- Key endpoints and examples
- Error response schemas
