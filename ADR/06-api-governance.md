# API Governance

## Table of Contents

- [API Shape & Endpoints](#1-api-shape--endpoints)
  - [System Endpoints](#system-endpoints)
  - [Job Management Endpoints](#job-management-endpoints)
  - [User Management Endpoints](#user-management-endpoints)
  - [Webhook Management Endpoints](#webhook-management-endpoints)
  - [Billing Endpoints](#billing-endpoints)
  - [Analytics Endpoints](#analytics-endpoints)
- [Versioning Strategy](#2-versioning-strategy)
- [Error Model](#3-error-model)
- [Idempotency Strategy](#4-idempotency-strategy)

---

## 1. API Shape & Endpoints

### System Endpoints

| **Method** | **Endpoint** | **Description** |
|------------|--------------|-----------------|
| GET | `/system/health` | Health check |
| GET | `/system/ready` | Readiness check |

### Job Management Endpoints

| **Method** | **Endpoint** | **Description** |
|------------|--------------|-----------------|
| GET | `/jobs` | List jobs |
| GET | `/jobs/:id` | Get job details |
| GET | `/jobs/:id/status` | Get job status (dedicated endpoint, prevents over-fetching) |
| GET | `/jobs/:id/outputs` | Get job outputs (dedicated endpoint, prevents over-fetching, paginated) |
| GET | `/jobs/:id/stream` | Server-Sent Events stream |
| POST | `/jobs` | Create job(s) (single endpoint, handles either one or many jobs) |
| PUT | `/jobs/:id/status` | Update job status (dedicated endpoint, simplifies extension of status types) |
| DELETE | `/jobs/:id` | Delete job |
| POST | `/jobs/reconcile` | Reconcile jobs (internal endpoint, automates retrying jobs with issues) |

### User Management Endpoints

| **Method** | **Endpoint** | **Description** |
|------------|--------------|-----------------|
| GET | `/users` | List users |
| GET | `/users/:id` | Get user details |
| POST | `/users/invite` | Invite user |
| PUT | `/users/:id` | Update user |
| DELETE | `/users/:id` | Delete user |

### Webhook Management Endpoints

| **Method** | **Endpoint** | **Description** |
|------------|--------------|-----------------|
| GET | `/webhooks` | List webhooks |
| GET | `/webhooks/:id` | Get webhook details |
| POST | `/webhooks` | Create webhook |
| PUT | `/webhooks/:id` | Update webhook |
| DELETE | `/webhooks/:id` | Delete webhook |

### Billing Endpoints

| **Method** | **Endpoint** | **Description** |
|------------|--------------|-----------------|
| GET | `/billing/usage` | Get usage data (quota usage) |
| GET | `/billing/invoices` | Get issued invoices |

### Analytics Endpoints

| **Method** | **Endpoint** | **Description** |
|------------|--------------|-----------------|
| GET | `/analytics/jobs` | Get job analytics (business metrics and reporting) |

---

## 2. Versioning Strategy

- **Options Considered**:

<table>
<tr>
<th valign="top">Option</th>
<th valign="top">Pros</th>
<th valign="top">Cons</th>
</tr>
<tr>
<td valign="top">Path-Based Versioning</td>
<td valign="top">• CDN/cache optimization<br>• Debugging excellence<br>• Client simplicity<br>• Provides isolation</td>
<td valign="top">• Maintenance overhead<br>• Client must update URLs (but manageable with SDKs)</td>
</tr>
<tr>
<td valign="top">Header-Based Versioning / Media Type Versioning</td>
<td valign="top">• Clean URLs<br>• HTTP content negotiation</td>
<td valign="top">• CDN/cache complexity<br>• Harder to test in browser</td>
</tr>
<tr>
<td valign="top">Query Parameter Versioning</td>
<td valign="top">• No routing changes needed<br>• Zero-change migration to a new version</td>
<td valign="top">• Could suddenly break production<br>• CDN/cache complexity<br>• Mixing concerns in URL</td>
</tr>
</table>

- **Trade-offs**:
  - **CDN/cache Complexity vs Maintenance Overhead**
  - **HTTP content negotiation vs Debugging Excellence**
  - **HTTP content negotiation vs Client Simplicity**

- **Chosen Approach**:
  - Path-Based Versioning

- **Deprecation Notice**:
  - 12 months `(adopted industry wide)`
  - 24 months `(x2 for enterprises)`

---

## 3. Error Model
- **Options Considered**:

<table>
<tr>
<th valign="top">Option</th>
<th valign="top">Pros</th>
<th valign="top">Cons</th>
</tr>
<tr>
<td valign="top">RFC7807 Problem+JSON</td>
<td valign="top">• Official standard<br>• Consistent error format<br>• Custom fields and types<br>• Clear error messages<br>• Self-documenting error types</td>
<td valign="top">• Developers need to understand RFC7807<br>• Larger response payload</td>
</tr>
<tr>
<td valign="top">Custom Error Format</td>
<td valign="top">• Customizable format<br>• Smaller response payload<br>• Fast to implement</td>
<td valign="top">• No detailed error information<br>• Hard to troubleshoot issues</td>
</tr>
<tr>
<td valign="top">HTTP Status Only</td>
<td valign="top">• Very simple to implement<br>• Minimal response payload<br>• Quick to implement</td>
<td valign="top">• No detailed error information<br>• Hard to troubleshoot issues</td>
</tr>
</table>

- **Trade-offs**:
  - **Client Simplicity vs Developer Simplicity**

- **Chosen Approach**: `Client Simplicity`

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

<table>
<tr>
<th valign="top">Option</th>
<th valign="top">Pros</th>
<th valign="top">Cons</th>
</tr>
<tr>
<td valign="top">Client-controlled with UUID</td>
<td valign="top">• Client control over retry behavior<br>• Can prevent collision with UUID v4</td>
<td valign="top">• Key collision risk<br>• Client must generate key</td>
</tr>
<tr>
<td valign="top">Request Content Hash-based & Client-controlled</td>
<td valign="top">• Content-based deduplication<br>• Collision-resistant</td>
<td valign="top">• Client must generate key<br>• Performance overhead</td>
</tr>
<tr>
<td valign="top">Request Content Hash-based & Server-managed</td>
<td valign="top">• Client simplicity<br>• Content-based deduplication<br>• Collision-resistant</td>
<td valign="top">• Server must generate key based on request content<br>• Some dups can go through due to race conditions</td>
</tr>
</table>

- **Trade-offs**:
  - **Client Simplicity vs Client Control**
  - **Client Simplicity vs Deduplication Guarantees**

- **Chosen Approach**: `Client-controlled with UUID`
  - **Rationale** It balances client simplicity and deduplication guarantees.

- **Default Cache TTL**: 24 hours
