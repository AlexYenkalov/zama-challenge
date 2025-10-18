# Platform Policies (Gateway)

## Table of Contents

### 1. Authentication
- [Options Considered](#options-considered)
- [Trade-offs Analysis](#trade-offs)
- [Chosen Approach](#chosen-approach)
- [Auth Method](#auth-method)
- [Token Format](#token-format)
- [Token Lifecycle](#token-lifecycle)
- [Multi-factor Authentication](#multi-factor-authentication)

### 2. Authorization
- [Options Considered](#options-considered-1)
- [Trade-offs Analysis](#trade-offs-1)
- [Chosen Approach](#chosen-approach-1)
- [Access Control](#access-control)
- [Resource Isolation](#resource-isolation)
- [Permission Model](#permission-model)

### 3. Rate Limiting & Quotas
- [Options Considered](#options-considered-2)
- [Trade-offs Analysis](#trade-offs-2)
- [Chosen Approach](#chosen-approach-2)
- [Multi-Layer Traffic Control](#multi-layer-traffic-control)
- [Rate Limit Tiers](#rate-limit-tiers)
- [Throttling Behavior](#throttling-behavior)
- [Quota Enforcement](#quota-enforcement)
- [Quota Monitoring](#quota-monitoring)

### 4. Abuse Protection
- [Options Considered](#options-considered-3)
- [Trade-offs Analysis](#trade-offs-3)
- [Chosen Approach](#chosen-approach-3)
- [WAF Rules](#waf-rules)
- [DDoS Protection](#ddos-protection)
- [Bot Detection](#bot-detection)
- [Threat Intelligence](#threat-intelligence)

### 5. Content Security
- [Options Considered](#options-considered-4)
- [Trade-offs Analysis](#trade-offs-4)
- [Chosen Approach](#chosen-approach-4)
- [Input Validation](#input-validation)
- [Size Limits](#size-limits)
- [Content Filtering](#content-filtering)
- [Data Protection](#data-protection)

---

## 1. Authentication

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

---

## 2. Authorization
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

---

## 3. Rate Limiting & Quotas

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

- **Chosen Approach**: *Kong* + *Multi-layer Rate Limiting with Token Bucket + Leaky Bucket*
  - **Rationale**: 
    - Comprehensive protection against different attack vectors while maintaining performance
    - **Kong Gateway Selection Rationale**: Kong provides <3ms latency and 137K+ RPS performance that's 20x faster than cloud providers, offers native support for our Token Bucket + Leaky Bucket algorithms, has a clear cost progression from free to enterprise ($250-2000), supports flexible deployment strategies, and outperforms alternatives like NGINX Plus and Traefik for API management use cases.

  - **Kong Cost**: Depends on plan selection
    - **Free Plan**: $0/month (Basic rate limiting, community support) - for development/testing
    - **Pro Plan**: $250/month (Advanced rate limiting, support, monitoring) - for SMB production
    - **Enterprise Plan**: $500-2000/month (Full enterprise features, SLA, compliance) - for enterprise
  - **Upgrade Path**: Kong Free → Kong Pro → Enterprise solutions (AWS, Azure, GCP) for advanced features

  - **Multi-Layer Traffic Control**
    - **Layer 1 (Per-IP)**: 100 QPS fallback for unauthenticated requests (infrastructure protection)
    - **Layer 2 (Token Bucket)**: 1000 RPS sustained, 2000 burst per tenant (fair usage enforcement)
    - **Layer 3 (Leaky Bucket)**: 200 requests per 100ms per tenant (micro-burst protection)

  - **Rate limit tiers**: 
    - **Per-IP Tier**: 100 QPS for unauthenticated requests (prevents infrastructure abuse)
    - **Per-Tenant Tier**: 1000 RPS sustained, 2000 burst capacity (ensures fair resource allocation)
    - **Per-Endpoint Tier**: Same as per-tenant (consistent limits across all endpoints)
    - **Global Tier**: No global limits (tenant isolation prevents cross-tenant impact)

- **Throttling behavior**: 
  - **429 Too Many Requests**: Standard rate limit exceeded response
  - **Retry-After Headers**: Time until rate limit resets (1-60 seconds)
  - **Rate Limit Headers**: X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset

- **Quotas**

  - **Quota**: *Enforcement*: 

    - **Monthly Job Quotas**: Per-tenant, per calendar month (Julian calendar) based on pricing plan
    - **Soft Cap (80%)**: 202 Accepted with warning headers, notifications via email/in-app/dashboard
    - **Hard Cap (100%)**: 429 Too Many Requests, complete job creation blocking, read ops still allowed
    - **Quota Reset**: First day of each month at 00:00:00 UTC, simultaneous reset for all tenants

    - **Plan Changes**: 
      - **Upgrades**: New quota applied immediately (new total - already used)
      - **Downgrades**: Keep current quota until month end, new quota from next month

    - **Idempotency Handling**: Duplicate requests with idempotency keys don't count against quotas

    - **Storage**: 
      - **Primary**: MongoDB (transactions, durability, rich analytics)
      - **Cache Layer**: Redis (replication of quota counters, <2ms latency for quota checks)

  - **Quota**: *Monitoring*: 
  
    - **Performance Metrics**: <5ms latency overhead per layer, <3ms for Kong Gateway
    - **Effectiveness Metrics**: Layer 1 blocks 60-80% of abuse, Layer 2 handles 15-30%, Layer 3 catches 5-15%
    - **Capacity Planning**: Usage growth trends and capacity projections, daily/weekly/monthly usage trends
    - **Alert Thresholds**: >1% total requests blocked, >5ms latency degradation, >60% rate limiter capacity
    - **Quota Alerts**: 
      - **Soft Cap**: 80% quota usage triggers email/in-app/dashboard notifications
      - **Hard Cap**: 100% quota exceeded triggers immediate blocking and notifications
      - **System Health**: Quota system failures, latency/throughput violations
      - **Usage Patterns**: Unusual usage patterns and potential abuse detection
    - **Tenant Monitoring**: Single tenant >5% of blocks indicates abuse, 3x more blocks than others indicates unfair limits
    - **Security Alerts**: >50 requests/minute from single IP, >30% blocks from single country

---

## 4. Abuse Protection

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

- **WAF rules**: *OWASP Protection with Custom Rules*
  - **Standard Protection**: SQL injection, XSS, CSRF, command injection, directory traversal
  - **Custom Rules**: Tenant-specific rate limiting, API endpoint protection
  - **Rule Management**: Automated rule updates, false positive tuning, performance optimization
  - **Monitoring**: Real-time attack detection, rule effectiveness metrics, false positive tracking

- **DDoS protection**: *Multi-layer DDoS Mitigation*
  - **Volumetric Attacks**: Rate limiting at edge, traffic shaping, bandwidth throttling
  - **Application-layer**: Request pattern analysis, connection limiting, resource exhaustion prevention
  - **Infrastructure**: Global edge network, automatic scaling, traffic distribution
  - **Response**: Automatic blocking, traffic redirection, service degradation protection

- **Bot detection**: *Advanced Behavioral Analysis*
  - **Credential Stuffing**: Login attempt pattern detection, account lockout mechanisms
  - **Scraping Protection**: Request frequency analysis, user agent validation, behavioral fingerprinting
  - **CAPTCHA Integration**: Challenge-response for suspicious activity, progressive difficulty
  - **ML Detection**: Machine learning models for bot behavior identification, continuous learning

- **Threat intelligence**: *Real-time Security Feeds*
  - **IP Reputation**: Malicious IP blocking, geographic restrictions, known attacker databases
  - **Attack Patterns**: Signature-based detection, anomaly detection, zero-day protection
  - **Update Frequency**: Real-time threat feed updates, automatic rule deployment
  - **Integration**: Cloudflare threat intelligence, third-party security feeds, custom threat data

---

## 5. Content Security

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

- **Input validation**: *JSON Schema Validation with Sanitization*
  - **Schema Validation**: Enforced at edge for all POST bodies, specific error details, 100ms timeout
  - **Sanitization**: Automatic stripping of unknown headers, query params, and body params
  - **Error Handling**: 400 Bad Request with specific validation errors, 415 Unsupported Media Type
  - **Performance**: Pre-compiled schemas, memory caching, hot reload for updates
  - **Security**: Schema integrity validation, injection attack prevention, recursive loop protection

- **Size limits**: *Comprehensive Request Limits*
  - **Request Body**: Maximum 1 MiB per request (prevents DoS, ensures reasonable payload size)
  - **Headers**: Maximum 8 KiB total headers, 4 KiB per individual header (prevents header flooding)
  - **Schema Size**: Maximum 1 MiB per schema (prevents memory exhaustion)
  - **Error Response**: 413 Payload Too Large with size limit information
  - **Business Justification**: Prevents resource exhaustion, ensures fair usage, protects infrastructure

- **Content filtering**: *Strict Content-Type Enforcement*
  - **POST Requests**: `application/json` required, strict Content-Type checking
  - **Validation**: Immediate rejection of incorrect Content-Type headers
  - **Error Response**: 415 Unsupported Media Type with supported Content-Type information
  - **Security**: Prevents type confusion attacks, ensures proper content handling
  - **Performance**: Fast Content-Type checking with minimal overhead

- **Data protection**: *Multi-layer Data Security*
  - **Encryption**: TLS 1.3 for transit, AES-256 for sensitive data at rest
  - **PII Handling**: Automatic detection and masking of personally identifiable information
  - **Compliance**: GDPR, CCPA, SOC2 compliance for data protection requirements
  - **Access Control**: Role-based access to sensitive data, audit logging for data access
  - **Retention**: Automated data retention policies, secure deletion procedures
