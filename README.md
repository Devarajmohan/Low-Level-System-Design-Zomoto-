# Low Level system design

# Project Title :

## Smart Meal Planning System for Zomato

# Problem Statement :

__Modern users struggle to maintain a healthy diet because existing meal-planning apps lack personalization, accurate nutrition tracking, and seamless automation. Most tools rely on manual input, offer generic plans, and fail to integrate groceries, health data, or AI-driven insights. As a result, users abandon these apps due to poor UX, limited accessibility, and inconsistent results__

# Solution :

Our solution delivers a fully personalized meal-planning ecosystem that adapts to each user’s health goals and lifestyle. It automates nutrition tracking, optimizes meals in real time, and unifies planning, groceries, and progress into one seamless flow. With scalable architecture and cross-platform access, it offers a smoother, smarter experience than existing tools.
AI-driven personalization for dietary needs and preferences


* Automatic nutrition tracking using OCR and a large food database


* Real-time meal optimization based on daily habits and health data


* End-to-end flow: meal planning → groceries → tracking → progress


* Scalable microservices with multi-platform support


* Minimal manual effort for maximum adoption


.

# Functional Requirements :

1. User Management — users can register, log in, update profiles, and manage preferences.

2. Personalized Meal Planning — AI generates daily/weekly meal plans based on goals, preferences, and restrictions.

3. Meal Logging — users can log meals manually, via image recognition, or barcode scanning.

4. Nutrition Tracking — system tracks calories, macros, and key health metrics automatically.

5. Grocery List Generation — auto-create grocery lists based on meal plans with options to edit or mark items.

6. Progress & Analytics — users can view weekly/monthly progress, nutrition trends, and AI insights.

7. Notifications — reminders for meals, hydration, goals, and grocery tasks.

8. Cross-Platform Access — seamless usage across web, mobile, and wearables.


 
# Non-Functional Requirements :

1. Performance — low latency APIs and fast response times under peak load.

2. Scalability — system must scale horizontally and vertically to support millions of users.

3. Reliability — high uptime, consistent data, and robust error-handling mechanisms.

4. Security — strong encryption, authentication, authorization, and OWASP-compliant protections.

5. Usability — intuitive UI/UX with accessibility standards (WCAG) and smooth onboarding.

6. Maintainability — modular architecture, clear documentation, and automated CI/CD deployment.

7. Availability — redundant services, failover support, and disaster recovery setup.

8. Monitoring — real-time logs, metrics, and alerts for system health and performance.


# Diagram :

![Flow Diagram](https://github.com/user-attachments/assets/389f48ba-fb23-4e05-8bbc-4f745d87faea)

# Diagram Flow Explain :

## 0. Global Design Principles

 **1. Microservice Boundaries**
- Each box is an independently deployable service owning its own data.  
- Communicate via well-defined HTTP/gRPC APIs or async events (Kafka).

 **2. Synchronous vs Asynchronous**
- Use **synchronous calls (HTTP/gRPC)** for low-latency, user-facing flows (e.g., authentication, meal plan generation).
- Use **asynchronous event streams (Kafka)** for eventual consistency , analytics, ML training, background non-blocking tasks  

 **3. Idempotency & Retries**
- All external-facing writes must be **idempotent** (idempotency keys).
- Implement **backoff + retry** policies for transient failures.

**4. Observability**
- Metrics → **Prometheus**  
- Logs → **ELK stack**  
- Traces → **OpenTelemetry / Jaeger**  
- Apply consistent observability across all services.

**5. Security**
- Zero-trust architecture:
  - mTLS  
  - service mesh (e.g., Istio) *or* mutual TLS  
  - JWT-based service tokens  
- RBAC for internal APIs  
- OAuth 2.0 / OpenID Connect for user authentication

 **6. CI/CD & Infrastructure**
- Each microservice has a pipeline:  
  - tests → build → container image → canary/rolling deploy  
- IaC using **Terraform**  
- Use **Kubernetes** for orchestration.

**7. SLA & SLO**
- Define per-service performance standards:  
  - e.g., Meal Planner → **200ms p95 latency**, **99.9% uptime**

**8. Data Governance**
- Proper handling of PII  
- Encryption at rest & in transit  
- Data retention policies  
- GDPR / CCPA compliance


## 1. Client

**1. Purpose**
- Web or mobile application used by end users to:
  - request meal plans  
  - browse menus  
  - place orders  
  - update preferences  

**2.Responsibilities**
- Authenticate using **OAuth / OIDC / JWT tokens**.  
- Cache static assets via **CDN** (images, UI bundles).  
- Perform **client-side validation** for better user experience.

**3. Implementation**
- Mobile apps: **iOS / Android**  
- Web app: **SPA (React / Vue)**  
- Use platform SDKs for:
  - push notifications  
  - local caching / offline storage  

**4. Security**
- Store **refresh tokens** in secure storage.  
- Use **short-lived access tokens** for API calls.

**5. Telemetry**
- Collect:
  - client-side performance metrics  
  - error events  
- Tools: **Sentry** or custom in-house telemetry pipeline.



## 2. API Gateway

**1. Purpose**
- Acts as the single ingress for **all client traffic**.
- Handles:
  - routing  
  - authentication  
  - rate limiting  
  - request validation  
  - observability  
  - request aggregation  

**2. Capabilities**
- JWT token verification (or delegated to Auth service).  
- Rate limiting per:
  - user  
  - API key  
  - IP  
- Web Application Firewall (WAF) protections.  
- TLS termination with request/response size limits.  
- API versioning support and canary routing via headers.

**3. Implementation Options**
- **Kong**  
- **AWS API Gateway**  
- **GCP Cloud Endpoints**  
- **NGINX + Lua**  
- **Envoy** (often part of a service mesh)

**4. Design Considerations**
- Offload heavy authentication work when possible  
  (e.g., **JWT validation at gateway** before hitting services).  
- Enable caching for **idempotent GET requests**  
  (respecting correct cache-control headers).

**5. Observability**
- Emit **access logs** to ELK stack.  
- Record **latency, throughput, error metrics** into Prometheus.



## 3. Auth Service

 **1. Purpose**
- Manage user identity and authentication flows:
  - login  
  - token issuance (OAuth2 / OpenID Connect)  
  - refresh tokens  
  - social login  
  - password reset  
  - MFA (multi-factor authentication)  

**2. Data & Persistence**
- Store **credential metadata only** (never raw passwords).  
- Secure DB such as **PostgreSQL with encryption**.  
- Use **Secrets Manager / Vault** for managing keys, signing secrets, and environment secrets.  
- Store issued tokens and refresh tokens securely.

**3. API Endpoints**
- `/oauth/token`  
- `/authorize`  
- `/users`  
- `/password-reset`  
- `/mfa/verify`

**4. Security**
- Rate-limit all authentication endpoints.  
- Implement brute-force protection (IP + account-level).  
- Password hashing using **Argon2** or **Bcrypt**.  
- Enforce MFA for sensitive workflows (password change, account update, payment actions).

**5. Scalability**
- Horizontally scalable — stateless authentication using **JWT**.  
- Maintain a token **revocation list in Redis** for forced logouts and compromised tokens.

**6. Observability & Auditing**
- Log and audit:
  - login attempts  
  - failed authentication  
  - MFA verifications  
- Export audit logs to a **secure, immutable audit store**.




## 4. Meal Planner Service

 **1. Purpose**
- Central orchestrator for user flows.
- Generates daily/weekly meal plans.
- Suggests swaps and persists plans.
- Calls Recommendation, Nutrition, and Menu services.

 **2. APIs**
- **POST `/meal-plans`** – generate meal plan  
- **GET `/meal-plans/{id}`** – get meal plan  
- **PUT `/meal-plans/{id}/swap`** – swap a dish  
- **POST `/meal-plans/{id}/order`** – send meal plan to order flow  

 **3. Responsibilities**
- Validate user preferences and constraints.
- Coordinate across:
  - Recommendation Service (dish selection)
  - Nutrition Service (macros)
  - Menu Service (availability, images)
  - Redis (caching)
- Own minimal meal-plan schema.

**4. Data**
- Store meal plan documents in minimal DB (or MongoDB).
- Keep only dish IDs to prevent duplication.

**5. Consistency**
- Use read-after-write for instant UX.
- Eventual consistency allowed for background jobs.

**6. Scaling**
- Autoscale based on request load.
- Queue heavy plan-generation jobs.

**7. Fault Tolerance**
- Use circuit breakers for service failures.
- Provide fallback cached plans.

**8. Security**
- Validate JWT tokens from Gateway.
- Enforce user permission and ownership checks.

**9. Observability**
- Emit Kafka events for created/updated plans.
- Supports analytics + ML pipelines.

**10. Batch vs Realtime**
- Heavy computations run in async task queues (K8s Jobs, Celery).
- Return Job ID for polling.



## 5. Recommendation Service

**1. Purpose**
- Encapsulates model inference logic for personalized dish recommendations.
- Generates recommended dishes, swaps, and personalized items.
- Uses user profile, context (time/location), and popularity trends.

**2. Interfaces**
- **POST `/recommend`** – takes user_id, context, constraints → returns ranked dish IDs + scores.
- Supports gRPC for low-latency inference workloads.
- Designed for synchronous request-response.

**3. Internals**
- Realtime lightweight model server.
- May call external Model Server or embed one internally.
- Uses hybrid logic: heuristics + ML model scoring.

**4. Feature Inputs**
- Recent user activity & order history.
- Dietary + cuisine preferences.
- Time of day & contextual signals.
- Dish popularity metrics.
- Menu availability (from Menu Service).

**5. Caching**
- Cache frequent recommendation requests in Redis.
- Use short TTLs to keep results fresh and reduce model load.

**6. Scaling**
- Horizontally scalable microservice.
- GPU-backed nodes optional for heavy ML inference.

**7. Fallback**
- If model server fails → use popularity-based heuristic engine.
- Ensures degraded but functional recommendations.

**8. Observability**
- Log model inputs/outputs (no raw PII).
- Track drift metrics for ML debugging.
- Emit Kafka events for analytics + model monitoring.



## 6. Model Server

**1. Purpose**
- Hosts ML models for inference.
- Serves recommendations, scoring, and personalization models.
- Provides inference endpoints via HTTP or gRPC.

**2. Tech Stack**
- TensorFlow Serving
- TorchServe
- Triton Inference Server
- OR custom FastAPI microservice with ONNX models.

**3. Deployment**
- Fully containerized for portability.
- Auto-scaling based on inference load.
- Supports model versioning.
- Canary releases (A/B testing).
- Blue/green deployments for safe model updates.

**4. Monitoring**
- Track latency and throughput.
- Monitor input feature distributions.
- Perform model health checks.
- Detect model drift in real-time.

**5. Security**
- Only accessible by internal services (Recommendation, Meal Planner).
- Strictly no public endpoints.
- Use mTLS or service mesh policies.

**6. Model Lifecycle**
- Receives new models from ML training pipeline or model registry (MLflow).
- Supports rollbacks to previous versions.
- Automates loading/unloading of models safely.



## 7. Nutrition Service

**1. Purpose**
- Provide nutritional information for dishes (calories, macros, allergens).
- Calculate total nutrition values for meal plans.
- Perform dietary validations and safety checks.

**2. API**
- **GET `/nutrition/{dishId}`** – fetch nutrition for a specific dish.
- **POST `/nutrition/estimate`** – estimate nutrition for custom ingredients or user-created dishes.

**3. Data**
- Central Nutrition Database with structured nutrition facts.
- May integrate verified third-party nutrition datasets.
- Stores dish → ingredient → nutrition mappings.

**4. Responsibilities**
- Map recipes and ingredients to macro breakdowns.
- Flag allergens and dietary risks.
- Support custom user recipes with computed nutrition values.

**5. Implementation**
- Use PostgreSQL (relational) or MongoDB (document-style) depending on recipe structure.
- Deterministic nutrition calculations with strict testing.

**6. Caching**
- Cache high-frequency dish nutrition lookups in Redis.
- Short TTL to ensure updated nutrition values are reflected.

**7. Integration**
- Provides nutrition totals to Meal Planner.
- Supplies warnings, allergen flags, and macro summaries to UI.

**8. Compliance**
- Validate nutrition data quality.
- Track data provenance and versioning for regulatory compliance.



## 8. Menu Service

**1. Purpose**
- Acts as the source of truth for all dish metadata.
- Manages availability, pricing, vendor info, and image links.
- Powers Meal Planner, Recommendations, and Ordering flows.

**2. APIs**
- **GET `/menu/{dishId}`** – fetch dish details.
- **GET `/menu/available?date=...&location=...`** – fetch available dishes for a given day/location.
- **Admin Endpoints** – CRUD operations for managing menu items.

**3. Data**
- Dish database containing:
  - Dish ID  
  - Title & description  
  - Image URLs (CDN links)  
  - Ingredients list  
  - Vendor stock & availability  
  - Pricing metadata  

**4. Responsibilities**
- Enforce business rules such as:
  - Blackout dates  
  - Vendor-level constraints  
  - Stock availability  
- Maintain accurate, up-to-date menu metadata.

**5. Scaling**
- Read-heavy service → use DB read replicas.
- Serve images through a CDN for performance.
- Use Redis caching for frequently accessed dishes.

**6. Transactions**
- Use optimistic locking for safe menu updates.
- Event-sourcing optional for complex menu histories and audit trails.

**7. Integration**
- Provides authoritative availability to:
  - Meal Planner
  - Recommendation Service
  - Order Integration
- Ensures only orderable dishes appear to the user.

**8. Observability**
- Monitor:
  - 404 / missing dish errors  
  - Cache miss rates  
  - Slow DB or replica lag  
- Emit metrics to Prometheus and logs to ELK.




## 9. Redis Cache

**1. Purpose**
- Provide ultra-low-latency access to frequently used data.
- Cache hot data such as:
  - Recommendations
  - Menu lookups
  - Precomputed meal plans
- Reduce load on downstream services and databases.

**2. Patterns**
- Use **cache-aside** pattern (application reads → cache miss → fetch from DB → write to cache).
- Apply **TTLs** to prevent stale data buildup.
- Avoid long TTLs for fast-changing data (like dish availability).

**3. Cluster**
- Deploy **Redis Cluster** for:
  - High availability (HA)
  - Automatic sharding
  - Horizontal scaling
- Use replicas for read scalability.
- Enable persistence (AOF / RDB) depending on durability requirements.

**4. Cache Invalidation**
- Receive invalidation events from **Kafka** whenever:
  - Menu items update
  - Dish metadata changes
  - Availability changes
- Actively evict or update affected cache keys.

**5. Security**
- Restrict access using network firewall rules.
- Use Redis ACLs for command-level access control.
- Ensure in-cluster encryption if supported.



## 10. Order Integration Service

**1. Purpose**
- Handle creation of user orders.
- Integrate with external delivery partners (Zomato, Swiggy, etc.).
- Coordinate payments, delivery requests, and internal order tracking.
- Ensure reliable and consistent order processing.

**2. API**
- **POST `/orders`**
  - Validates dish availability.
  - Confirms pricing.
  - Checks user address & delivery feasibility.
  - Handles payment authorization.
- **GET `/orders/{id}`**
  - Retrieves order status and metadata.

**3. Design**
- Acts as an orchestration layer interacting with:
  - Menu Service (availability, price)
  - Payment Gateways (Stripe, Adyen)
  - Delivery Partner APIs
  - Order Database
- Manages the end-to-end order workflow across multiple systems.

**4. Idempotency & Reliability**
- Use **idempotency tokens** for safe order creation retries.
- Apply exponential backoff + delayed retries for partner API failures.
- Implement **compensating transactions** if a step fails mid-flow.
- Ensure no duplicate orders or payments.

**5. Transactions**
- Prefer **eventual consistency** for distributed systems.
- Place provisional holds on inventory (if applicable).
- Finalize order only after partner confirmation.

**6. Security**
- Comply with **PCI DSS** for payment flows.
- Never store raw card details—use payment provider tokens.
- Enforce authentication & request signing for external APIs.

**7. Observability**
- Emit order lifecycle events (created, confirmed, dispatched, failed).
- Provide logs for customer support and analytics.
- Track SLA metrics for partner APIs and internal steps.



## 11. Food Delivery APIs (External)

**1. Purpose**
- Provide third-party delivery execution capabilities.
- Enable placing, tracking, and updating food delivery orders.
- Integrate with external merchant ordering and logistics systems.

**2. Integration**
- Use partner-specific adapters to standardize response formats.
- Implement throttling, timeout handling, and retry logic.
- Normalize error codes and unify partner behavior for internal services.
- Maintain versioned API clients for each delivery provider.

**3. Operator Contracts**
- Track SLA metrics (latency, success rate, availability).
- Understand partner error semantics to define fallbacks.
- Implement degraded modes when a partner is down (failover to alternate provider).
- Log all partner interactions for compliance and troubleshooting.



## 12. Kafka Event Bus

**1. Purpose**
- Provides asynchronous event transport across microservices.
- Enables analytics pipelines, ML model training, and real-time updates.
- Supports change-data-capture workflows for downstream systems.

**2. Events**
- meal_plan.created
- meal_plan.updated
- order.created
- user.updated
- dish.viewed
- recommendation.clicked

**3. Schema**
- Use Avro or Protobuf with a centralized Schema Registry.
- Maintain backward/forward compatibility with careful versioning.
- Enforce strict schema evolution rules for producers and consumers.

**4. Retention & Topics**
- Create separate topics per domain or event type.
- Configure retention policies based on consumer requirements.
- Partition by user ID (or relevant key) to maintain ordering guarantees.

**5. Consumers**
- ML training pipelines.
- Analytics engines and dashboards.
- Feature store ingestion.
- Auditing and compliance systems.

**6. Scaling**
- Kafka cluster with replication factor for durability.
- Add partitions to increase throughput and parallelism.
- Use consumer groups for horizontal consumer scaling.

**7. Failure Handling**
- Use dead-letter queues for poison or malformed messages.
- Apply retry policies before pushing to DLQ.


## 13. Analytics Store

**1. Purpose**
- Long-term storage for analytical workloads.
- Supports product analytics, BI reporting, dashboards, and experimentation.
- Optimized for large-scale querying rather than real-time operations.

**2. Implementation**
- Use a cloud data warehouse (Snowflake / BigQuery / Redshift) or Hadoop data lake.
- Ingest data via streaming or batch ETL pipelines from Kafka.
- Maintain schema-on-write for warehouses and schema-on-read for data lakes.

**3. Data Design**
- Store denormalized event-level tables for fast analytical queries.
- Build aggregated rollups for dashboards and high-level metrics.
- Organize data by domain (orders, users, menu, recommendations).

**4. Access**
- Expose datasets through BI tools (Looker, Metabase, PowerBI).
- Enforce role-based access control for analysts, data engineers, and PMs.
- Provide semantic models or views for easier querying.



## 14. ML Training

**1. Purpose**
- Enable batch and online machine learning training workflows.
- Use historical events and engineered features to create new models.
- Ensure models are reproducible, validated, and production-ready.

**2. Workflow**
- Data ingestion from Kafka or ETL pipelines.
- Perform feature engineering and preprocessing.
- Train models → validate → register in model registry → deploy to Model Server.

**3. Orchestration**
- Use Airflow, Kubeflow, or Argo for pipeline execution and scheduling.
- Support DAG-based workflows for repeatable training processes.
- Enable monitoring, retries, and lineage tracking.

**4. Data**
- Consume features from the feature store and analytics warehouse.
- Maintain train/test/validation splits with reproducibility.
- Store metadata, datasets, and run configs for traceability.

**5. Experimentation**
- Track experiments using MLFlow or similar.
- Perform A/B tests and monitor rollout metrics.
- Support hyperparameter tuning and variant comparisons.

**6. Compute**
- Leverage GPU/TPU clusters for heavy ML workloads.
- Autoscale based on job demand.
- Use distributed training frameworks when required.



## 15. Feature Store

**1. Purpose**
- Centralized storage for machine learning features.
- Provides precomputed user, item, and contextual features.
- Ensures features are reusable across training and online inference.

**2. Types**
- Offline feature store for batch training (data warehouse / BigTable / S3).
- Online feature store for real-time inference (Redis or specialized stores).
- Supports point-in-time lookup to avoid data leakage.

**3. Consistency**
- Maintain feature parity between training and serving environments.
- Use same transformation logic for offline and online pipelines.
- Prevent drift using versioning and monitoring.

**4. Access Patterns**
- Low-latency reads for Model Server during inference.
- Bulk data joins for ML training pipelines.
- Supports caching and prefetching for performance.

**5. Security & Governance**
- Enforce access controls by role and service.
- Track lineage and provenance of all features.
- Audit logs for compliance and responsible ML.



## 16. User Profile Service

**1. Purpose**
- Store user preferences, allergies, dietary goals, and behavioral metadata.
- Maintain structured user profile data to support personalization.
- Provide a reliable source of truth for user-related configurations.

**2. Data Schema**
- Normalized database for user preferences and metadata.
- Strict separation of PII (e.g., address) and non-PII (e.g., taste profile).
- Schema supports extensions for dietary rules and custom preferences.

**3. APIs**
- `GET /users/{id}/preferences` → fetch user preferences and settings.
- `PUT /users/{id}/preferences` → update preferences, dietary choices, or restrictions.
- Designed for low latency and high read throughput.

**4. Privacy**
- Full consent management for data usage.
- Maintain data access logs for compliance and auditing.
- Provide export/delete features for GDPR and similar regulations.

**5. Scaling & Caching**
- Read-heavy workload optimized through Redis caching.
- Cache frequently accessed profile data with TTL or event-based invalidation.
- Scale horizontally for large traffic spikes (e.g., during meal plan updates).




## 17. User Preferences DB

**1. Purpose**
- Persistent storage for user preferences and personalization data.
- Structured relational schema for predictable access patterns.
- Acts as the authoritative store for preference-related queries.

**2. Backup & DR**
- Daily automated backups with configurable retention.
- Point-in-time recovery for accidental data loss scenarios.
- Cross-region replicas to ensure disaster recovery readiness.



## 18. CDN

**1. Purpose**
- Deliver static assets (dish images, icons) with low latency.
- Offload image traffic from Menu Service to improve performance.
- Enhance UI load times across regions.

**2. Implementation**
- Use cloud CDN providers (CloudFront, Fastly, Cloudflare).
- Support signed URLs for restricted or private content.
- Integrate with object storage (S3, GCS) for asset origins.

**3. Cache Invalidation**
- Use versioned asset URLs for automatic invalidation.
- Support purge APIs for immediate cache clearing on image updates.
- Avoid serving stale assets by syncing with Menu Service updates.



## 19. Dish Database

**1. Purpose**
- Acts as the authoritative storage for all dish metadata.
- Used primarily by the Menu Service for dish retrieval.
- Ensures consistent and up-to-date dish information across services.

**2. Schema**
- Fields include:
  - dish_id
  - vendor_id
  - ingredients
  - nutrition_reference
  - price
  - stock
  - last_updated
- Structured for fast lookups and integration with nutrition and menu systems.

**3. Indexes & Scaling**
- Index on `dish_id`, `vendor_id`, and frequently queried fields.
- Use read replicas to handle high read traffic.
- Apply sharding strategies if dataset grows large.



## 20. MongoDB Meal Plans

**1. Purpose**
- Store user meal plan documents with flexible, nested structures.
- Enable fast retrieval of user-specific daily/weekly meal plans.
- Support dynamic schema to evolve plan format easily.

**2. Design**
- Example document format:
  - `{ userId, planId, dates: [...], meals: [{ dishId, serving, nutrition }], created_at }`
- Document DB chosen for natural representation of hierarchical plan data.
- Supports updates to individual meals or entire plans.

**3. Scaling & Transactions**
- Use replication and sharding for horizontal scalability.
- Optional TTL indexing to auto-remove stale meal plans.
- MongoDB transactions ensure consistency across multiple documents when required.



## 21. Logging & Monitoring

**1. Purpose**
- Centralized logs, metrics, and search for debugging, alerting, and incident response.
- Supports observability across all microservices.

**2. Stack**
- Logs: Filebeat → Logstash → Elasticsearch → Kibana.
- Metrics: Prometheus + Grafana.
- Traces: Jaeger / OpenTelemetry.

**3. Retention & Alerts**
- Tiered retention: hot (7 days), warm (30 days), cold (archive).
- Alerts for:
  - Error rates
  - Latency p95/p99
  - CPU/memory saturation



## 22. Tracing

**1. Purpose**
- Provide distributed tracing across services.
- Helps track request flow and identify latency bottlenecks.

**2. Implementation**
- Use OpenTelemetry SDKs for all services.
- Propagate trace headers through gateway and Kafka (where feasible).

**3. Usage**
- Add spans for:
  - DB operations
  - External API calls
  - CPU-intensive tasks



## 23. Notification Service

**1. Purpose**
- Handle push notifications, emails, SMS, and in-app messages.
- Supports reminders, promotions, and order updates.

**2. API**
- `POST /notify` with:
  - channels
  - recipients
  - template_id
  - payload

**3. Delivery & Reliability**
- Use FCM, APNs, SendGrid, SES.
- Throttling + template engine.
- Retry mechanism + Dead-letter queue (DLQ).



## 24. Logging of Sensitive Data & Compliance

**1. PII Handling**
- Never log raw emails, phone numbers, or addresses.
- Use masking/redaction where needed.

 **2. Encryption**
- TLS for data in transit.
- KMS-managed AES-256 for data at rest.

**3. Compliance**
- Meet privacy and audit requirements (GDPR, SOC2).



## 25. Backups, DR, and Recovery

**1. Backups**
- Daily snapshots for RDBMS.
- Point-in-time recovery.
- Offsite and cross-region backups.

**2. Disaster Recovery**
- Multi-region deployment for critical systems.
- Automated failover + DNS routing.

**3. RTO / RPO**
- Define recovery expectations per service.
- Ensure documentation and rehearsed runbooks.



## 26. CI/CD, Testing, and Quality Gates

**1. Pipeline**
- Unit tests, integration tests, contract tests (Pact), E2E tests.
- Static analysis + security scanning.

**2. Deployment**
- Feature flags.
- Canary deployments with automated rollback.

**3. Resilience Testing**
- Chaos testing (Chaos Monkey) in staging.



## 27. Cost & Capacity Planning

**1. Estimation**
- Meal generation: CPU-heavy.
- Model inference: GPU/CPU depending on model.
- Kafka throughput sizing.

**2. Autoscaling**
- HPA based on CPU, latency, and queue depth.

**3. Optimization**
- Use caching and serverless for bursty workloads.



## 28. Security & Access Controls

**1. Network**
- Private subnets for internal clusters.
- Public subnets only for gateways.
- Strict security group rules.

**2. Auth & Secrets**
- OAuth2 for users.
- mTLS + short-lived tokens for internal services.
- Secrets stored in Vault / AWS Secrets Manager.

**3. Compliance & Testing**
- Regular penetration tests.
- Dependency and vulnerability scans.
- SOC2/GDPR readiness processes.



## 29. Data Models & Contracts

**1. Meal Plan Object**
- Represents a complete meal plan for a user.
- Stores days, meals, nutrition, and metadata.
- Used by Meal Planner, UI, and analytics.
- Example structure:
  ```json
  {
    "planId": "uuid",
    "userId": "uuid",
    "startDate": "YYYY-MM-DD",
    "endDate": "YYYY-MM-DD",
    "days": [
      {
        "date": "YYYY-MM-DD",
        "meals": [
          {"mealId": "uuid", "dishId": "uuid", "servings": 1, "nutrition": {"calories": 400}}
        ]
      }
    ],
    "createdAt": "timestamp",
    "metadata": {"source": "user" | "auto"}
  }
  ```

**2. Recommendation Request**
- Used by Recommendation Service to generate ranked dishes.
- Includes user context, preferences, and constraints.
- Lightweight request for fast inference.
- Example:
  ```json
  {
    "userId": "uuid",
    "context": {"mealType": "dinner", "diet": "vegan"},
    "constraints": {"maxCalories": 700}
  }
  ```

**3. Event Schema (Avro)**
- Used for Kafka event transport.
- Event: **meal_plan.created**
- Includes:
  - planId  
  - userId  
  - timestamp  
  - summary payload (e.g., number of days, total calories)
- Versioned via schema registry for compatibility.



## 30. Operational Runbook Items

**1. Common Incidents**
- Kafka broker downtime or partition imbalance.
- Model Server OOM or inference latency spikes.
- Database connection pool saturation or slow queries.
- Meal Planner high latency or cascading failures from downstream services.
- Third-party payment provider outages or API throttling.

**2. Runbook Guidelines**
- Define clear **detection rules** (alerts, thresholds, dashboards).
- Specify **mitigation actions**: circuit breakers, rate limiting, fallbacks, cache usage.
- Provide **escalation paths** to SRE/engineering owners.
- Include **post-mortem steps**: root cause analysis, action items, prevention tasks.

**3. SRE Practices**
- Maintain **error budgets** to balance feature velocity vs reliability.
- Track **SLOs** for latency, availability, and throughput.
- Continuously improve runbooks through incident reviews.

