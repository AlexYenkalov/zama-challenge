# Metering Logic

## Table of Contents

- [Usage Events](#1-usage-events)
- [Billing System](#2-billing-system)
- [Event Processing](#3-event-processing)
- [Invoice Mapping](#4-invoice-mapping)

---

## 1. Usage Events

- **Options Considered**: 
  - **API Volume Utilization**: Job processing volume, GB
  - **API Compute Utilization**: Job processing compute, MS
  - **API Call Events**: Track individual API calls
  - **Business Events**: All job lifecycle events

- **Trade-offs**: 
  - **Granularity vs Volume**: API call events provide granular billing but generate high event volume
  - **Billing Complexity vs Insights**: Comprehensive events provide rich data but complex billing logic
  - **Implementation vs Value**: Custom events require more work but provide business-specific value
  - **Event Types vs Business Logic**: API call events (technical) vs job lifecycle events (business-focused)

- **Chosen Approach**: `Hybrid`
  - **Rationale**: Since we're in the discovery phase, experimenting with different approaches will help us understand what works best and learn from our customers.

  - **Event Types**:
    - `Created Job Count`
    - `Processed Job Count`
    - `Confirmed Job Count`
    - `Job Status Updates Consumed` (manual fetches, Server-Send Events)
    - `Job Compute Time`
    - `Job Data Volume`

---

## 2. Billing System

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
    - **Braintree**: 
      - **Strengths**: PayPal integration, global payment methods, fraud protection
      - **Weaknesses**: Limited billing features, PayPal dependency
      - **Best For**: Payment processing focus, PayPal ecosystem
      - **Cost**: 2.9% + $0.30 per transaction
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

- **Chosen Option**: `Trial Lago (open source)`
  - **Rationale**: Lago is perfect for our discovery phase because it's free, open-source, and gives us complete control over billing logic without vendor lock-in, while its modern architecture and developer-friendly APIs let us experiment with different pricing models and learn customer behavior patterns.

---

## 3. Event Processing

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

- **Chosen Approach**: `Hybrid Event Processing with Real-time Billing`
  - **Rationale**: Real-time processing for billing events, batch processing for analytics
  - **Business Value**: Immediate billing accuracy + cost-effective analytics processing
  - **Technical Benefits**: Low latency for billing + high throughput for analytics
  - **Architecture**: Event-driven billing + batch analytics + stream processing for high-volume events

- **Event ingestion**: `Multi-source Event Collection with Reliability Guarantees`
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
    - **Event Store**: MongoDB for event persistence and Redis cache high-performance access

---

## 4. Invoice Mapping

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

- **Chosen Approach**: `Hybrid Tiered Pricing with Volume Discounts`
  - **Rationale**: Combines predictable tiered pricing with customer-friendly volume discounts
  - **Business Value**: Attracts customers with volume discounts while maintaining revenue predictability
  - **Market Positioning**: Competitive pricing that scales with customer usage
  - **Revenue Model**: Base subscription + usage overage + volume discounts

- **Line item calculation**: `Multi-factor Billing with Business Rules`
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

- **Tiered pricing**: `Volume-based Pricing with Market Analysis`
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

- **Adjustments**: `Automated Credits with Approval Workflow`
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

- **Invoice delivery**: `Multi-channel Delivery with Guarantees`
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
