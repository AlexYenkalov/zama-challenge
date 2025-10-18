# Data Models

## Table of Contents

### 1. Job Entities
- [Entity Interfaces](#entity-interfaces)
  - [Problem Detail](#problem-detail)
  - [Result / Error](#result--error)
  - [Tx Receipt](#tx-receipt)
  - [Job Entity](#job-entity)
  - [Job State Entity](#job-state-entity)
  - [Job Input Entity](#job-input-entity)
  - [Job Output Entity](#job-output-entity)

### 2. Tenant, User, and Pricing Plan Entities
- [Scope](#scope)
- [Relationships](#relationships)
- [Entity Interfaces](#entity-interfaces-1)
  - [Tenant](#tenant)
  - [Pricing Plan Declaration](#pricing-plan-declaration)
  - [Pricing Plan Quota Declaration](#pricing-plan-quota-declaration)
  - [Pricing Plan Subscription](#pricing-plan-subscription)
  - [Pricing Plan Quota Usage Event](#pricing-plan-quota-usage-event)

### 3. Billing Metric Types
- [Metric Types](#billing-metric-types)

### 4. Database Architecture
- [Options Considered](#options-considered)
- [Trade-offs Analysis](#trade-offs)
- [Chosen Approach](#chosen-approach)

---

**Note**: Pricing Plan models are relevant if we gonna use **open source** solution from **Lago**

## 1. Job Entities

  - **Entity Interfaces**

    - **Problem Detail (ERC 9...)**
    ```ts
    interface ProblemDetail {
      // TODO
    }
    ```

    - **Result / Error**
    ```ts
      type ResultData = unnkown | null;
      type ResultError = unnkown | null | ProblemDetail;

      interface Result {
        status: boolean
        data: ResultData;
        error: ResultError;
      }
    ```

    - **Tx Receipt**
    ```ts
    interface TxReceipt {
      // TODO
    }
    ```

    - **Job**
    ```ts
      /**
       * Job Entity
        * - References tenant_id and author_id (user_id)
        * - Tracks job lifecycle from creation to completion
        */
      interface JobEntity {
        @IsPrimaryKey()
        @IsStringProperty()
        id: string;

        @IsStringProperty()
        type: string;

        @IsForeignKey()
        @IsStringProperty()
        author_id: string;

        @IsForeignKey()
        @IsStringProperty()
        tenant_id: string;

        @IsObjectProperty()
        state: JobStateEntity;

        @IsArrayProperty()
        inputs: JobInputEntity[];

        @IsArrayProperty({ isNullable: true })
        outputs?: JobOutputEntity[] | null;

        @IsDateProperty()
        created_at: Date;

        @IsDateProperty({ isNullable: true })
        updated_at?: Date | null;

        @IsDateProperty({ isNullable: true })
        queued_at?: Date | null;

        @IsDateProperty({ isNullable: true })
        processed_at?: Date | null;

        @IsDateProperty({ isNullable: true })
        confirmed_at?: Date | null;

        @IsDateProperty({ isNullable: true })
        cancelled_at?: Date | null;

        @IsDateProperty({ isNullable: true })
        rejected_at?: Date | null;

        @IsDateProperty({ isNullable: true })
        failed_at?: Date | null;
      }

      /**
       * Job State Entity
        */
      interface JobStateEntity {
        @IsEnumProperty(JobStatusEnum)
        status: JobStatusEnum;
      }

      /**
       * Job Status
      */
      enum JobStatusEnum {
        Unknown = 0
        Queued = 1,
        Processing = 2,
        Processed = 3,
        Confirming = 4,
        Confirmed = 4,
        Cancelled = 5,
        Rejected = 6,
        Failed = 7,
      }

      /**
       * Job Input Entity
      * - type is a URL, it can by dynamically extended, and shall be progrmatically validated, potentially via dynamic job schema registration procedure
      */
      interface JobInputEntity {
        @IsStringProperty()
        id: string;

        @IsURL()
        type: string;

        [key: string]: unknown;
      }

      /**
       * Job Output Entity
        * - Job result/output data
        * - Can be either Result or ResultDataWithTxReceipt
        */
      type JobOutputEntityType = Result | ResultWithTxReceipt;

      interface ResultWithTxReceipt {
        status: boolean
        data: ResultDataWithTxReceipt;
        error: ResultError;
      }

      interface ResultDataWithTxReceipt {
        tx_receipt: TxReceipt;
      }
    ```

---

## 2. Tenant, User, and Pricing Plan Entities
  
  - **Scope**
    - Tenant
    - User
    - Pricing Plan Subscription
    - Pricing Plan Declaration
    - Pricing Plan Quota
    - Pricing Plan Quota Usage Event

  - **Relationships**: 
    - **One-to-Many**: *Tenant* → *User*
    - **One-to-One**: *Tenant* → *Pricing Plan Subscription*
    - **One-to-One**: *Pricing Plan Subscription* → *Pricing Plan Declaration*
    - **One-to-Many**: *Pricing Plan Declaration* → *Pricing Plan Quota*
    - **One-to-Many**: *Pricing Plan Quota* → *Pricing Plan Quota Usage Event*

  - **Entity Interfaces**

    - **Tenant**
    ```ts
      /**
       * Tenant
        * - gets pricing_plan_subscription_id assigned
        * - Deleted status is terminal
        * - captures deletion date
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

    ```

    - **Pricing Plan Declaration**
    ```ts
      /**
       * Pricing Plan Declaration for tenant subscription
        * - type is limited to predefined values i.e. dynamic pricing plan extension is not supported
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
    ```

    - **Pricing Plan Quota Declaration**
    ```ts
      /**
      * Pricing Plan Quota Declaration
        * - type is a URL, it can by dynamically extended, and shall be progrmatically validated
        * - references `pricing_plan_declaration_id
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
    ```

    - **Pricing Plan Quota Subscription**
    ```ts
      /**
       * Pricing Plan Subscription
        * - isolated to `tenant_id`
        * - references `pricing_plan_declaration_id`
        * - Cancelled status is terminal
        * - captures subscription date
        * - captures cancellation date
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
    ```

    - **Pricing Plan Quota Usage Event**
    ```ts
      /**
       * Pricing Plan Quota Usage Event
        * - type is a URL, it can by dynamically extended, and shall be progrmatically validated
        *   - {tenant_id}.example.com/{version}/{event_namespace}/{event_id}/{event_type}?{k}={v},{k}={v},{k}={v}
        *   - example.com/v1/jobs/007/created}?priority=1
        *   - example.com/v1/jobs/007/completed?compute_ms=10000,compute_bytes=100000
        *   - example.com/v1/jobs/007/confirmed}?tx_receipt=0x..
        *   - example.com/v1/jobs/007/failed}?error_code=999,error_message="..."
        */
      interface PricingPlanQuotaUsageEventBaseEntity {
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
      }

      interface PricingPlanQuotaUsageEventBaseEntity extends PricingPlanQuotaUsageEventBaseEntity {
        // Type-dependent: order-bound fields

        @IsStringProperty({ isNullable: true })
        priority: string | null;

        // Type-dependent: resource-bound fields

        @IsIntegerProperty({ isNullable: true })
        compute_ms: number | null;

        @IsIntegerProperty({ isNullable: true })
        compute_bytes: number | null;

        // Type-dependent: blockchain-bound fields

        @IsStringProperty({ isNullable: true })
        tx_receipt: string | null;

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
      ``` 

---

## 3. Billing Metric Types
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

---

## 4. Database Architecture

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
