# API Governance

## Table of Contents

### 1. API Shape & Endpoints
- [API Endpoints](#api-endpoints)
  - [System Endpoints](#system)
  - [Job Management](#jobs)
  - [User Management](#users)
  - [Webhook Management](#webhook-management)
  - [Billing](#billing)
  - [Analytics](#analytics)

### 2. Versioning Strategy
- [Options Considered](#options-considered)
- [Trade-offs Analysis](#trade-offs)
- [Chosen Approach](#chosen-approach)
- [Deprecation Notice](#deprecation-notice)

### 3. Error Model
- [Options Considered](#options-considered-1)
- [Trade-offs Analysis](#trade-offs-1)
- [Chosen Approach](#chosen-approach-1)
- [Error Response Format](#error-response-format)
- [HTTP Status Code Mapping](#http-status-code-mapping)

### 4. Idempotency Strategy
- [Options Considered](#options-considered-2)
- [Trade-offs Analysis](#trade-offs-2)
- [Chosen Approach](#chosen-approach-2)
- [Default Cache TTL](#default-cache-ttl)

---

## 1. API Shape & Endpoints

- **API endpoints**

  - **System**
    - GET /system/health *(health check)*
    - GET /system/ready *(readiness check)*

  - **Jobs**
    - GET /jobs
    - GET /jobs/:id
    - GET /jobs/:id/status *(dedicated endpoint for "status", prevents over-fetching)*
    - GET /jobs/:id/outputs *(dedicated endpoint for "outputs", prevents over-fetching, paginated)*
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

---

## 2. Versioning Strategy

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

---

## 3. Error Model
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

---

## 4. Idempotency Strategy

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
