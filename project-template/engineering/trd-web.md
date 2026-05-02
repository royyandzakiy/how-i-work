# TRD: [Project Name Template]

**Version:** `1.0` (Last Updated: 2026-4-29)
**PRD:** [Link to Internal PRD Document]

---

## 1. System Architecture

### 1.1 Architecture Pattern

Two independent repositories implementing a client–server architecture:

```
+---------------------+       HTTPS/REST       +----------------------+
|   Vue.js Frontend   |  <------------------>  |   FastAPI Backend    |
|   (Vue Router)      |                        | (REST API Layer)     |
+---------------------+                        +----------------------+
          ↑                                              ↓
          |                                    +----------------------+
          |                                    | PostgreSQL Database  |
          |                                    | Redis Cache          |
          |                                    +----------------------+
          |
   +--------------------+
   | Keycloak / SSO     |
   +--------------------+
```

### 1.2 Technology Stack

| Layer | Technology | Notes |
| --- | --- | --- |
| **Frontend** | Vue 3, Vue Router 4, TailwindCSS, Axios, TanStack Query | SPA with modular routing |
| **Backend** | FastAPI (Python), SQLAlchemy | RESTful API, modular per domain |
| **Database** | PostgreSQL 16 | Normalized relational schema |
| **Cache** | Redis 7 | Used for monitoring and aggregation caching |
| **Authentication** | Keycloak (OAuth2 / OpenID Connect) → JWT | Single Sign-On integration |
| **Deployment** | AWS ECS (Frontend + API) | Container-based |
| **CI/CD** | GitHub Actions → AWS CodeDeploy | Automated build & deploy |
| **Monitoring** | DataDog | API latency, request tracing |
| **Logging** | Python structlog (JSON logs to stdout) | Centralized structured logging |

### 1.3 Architecture Decision Record (ADR)

| **Date of Approval** | **Decision** | **Description** |
| --- | --- | --- |
| [Date] | Approved/Ongoing | Motivation: [Link to Internal Design Doc] |

### 1.4 Additional Technical Documents

- **Architecture Backlog**: [Internal Link]
- **API Contract**: [Internal Link]
- **ERD**: [Internal Link]
- **C4 Diagram**: [Internal Link]

---

## 2. Repositories

### 2.1 Frontend Repository

**Repository Link:** [Internal Link]
**Framework:** Vue 3 + Vite
**Structure:**

```
src/
 ├─ modules/
 │   ├─ dashboard/
 │   ├─ resources/
 │   ├─ schedules/
 │   └─ metrics/
 ├─ components/
 ├─ composables/
 ├─ utils/
 ├─ router/
 └─ main.ts
```

**Dependencies:**

- `tanstack-query` – client-side caching
- `axios` – HTTP client
- `vee-validate` + `zod` – form validation
- `echarts` – dashboards
- `tailwindcss` – styling framework

**Build & Deploy:**

```shell
npm run build
```

Artifacts served by **AWS CloudFront** + **S3**.

**Env vars:**

```
VITE_API_BASE_URL=https://api.internal-system.com/v1
```

### 2.2 Backend Repository

**Repository Link:** [Internal Link]
**Framework:** FastAPI (Python 3.11)
**Structure:**

```
src/
 ├─ main.py
 ├─ modules/
 │   ├─ dashboard/
 │   ├─ resources/
 │   ├─ schedules/
 │   ├─ metrics/
 ├─ common/
 ├─ config/
 ├─ database/
 └─ middleware/
```

**Dependencies:**

- `sqlalchemy`
- `asyncpg`
- `pydantic`
- `redis`
- `python-dotenv`

**Environment Variables:**

```shell
PORT=8000
DATABASE_URL=postgresql://user:pass@db.internal:5432/app_db
REDIS_URL=redis://cache.internal:6379
JWT_SECRET=super_secure_key
KEYCLOAK_REALM=<realm>
KEYCLOAK_CLIENT_ID=<client_id>
KEYCLOAK_CLIENT_SECRET=<secret>
```

**Deployment:**

- Deployed as a **Docker container** to **AWS ECS (Fargate)**.
- API base path: `/api/v1`
- Health check: `/api/v1/health`

---

## 3. Functional Modules

### 3.1 [Module name]

**Purpose:** [Module purpose]

**Endpoints:**

| Method | Endpoint | Description |
| --- | --- | --- |
| GET | `/api/v1/module` | List and filter module |
| POST | `/api/v1/module` | Create a new module |
| PUT | `/api/v1/module/:id` | Update module |

**Schema:**

```sql
CREATE TABLE modules (
  id UUID PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## 4. Data Flow

```
User
 ↓
Vue (Axios)
 ↓
FastAPI (Pydantic validation, service, repo)
 ↓
PostgreSQL ←→ Redis (cache)
```

**Example Flow:**
User edits workload → `PUT /api/v1/schedules/:id`
→ Backend validates DTO → Updates DB → Invalidates cache → Returns new data.

---

## 5. Non-Functional Requirements

| Category | Requirement |
| --- | --- |
| **Performance** | <1s filtering and update response |
| **Scalability** | 10,000 resources, 2,000 programs, 500k+ assignments |
| **Usability** | Chrome, Edge, Safari (desktop) |
| **Security** | SSO via Keycloak; JWT validation; HTTPS enforced |
| **Availability** | 99.5% uptime target |
| **Maintainability** | ESLint + Prettier + pytest (80% coverage) |
| **Extensibility** | Each module isolated and versioned independently |

---

## 6. Logging & Monitoring

| Layer | Tool | Purpose |
| --- | --- | --- |
| Frontend | Sentry | Capture UI/API errors |
| Backend | structlog | Structured logs (JSON) |
| Monitoring | DataDog | Request tracing, performance metrics |
| Dashboards | DataDog Dashboards | CPU, memory, error rate visualization |

---

## 7. Deployment

| Environment | URL | Purpose |
| --- | --- | --- |
| DEV | [Internal Link] | Development |
| PROD | [Internal Link] | Production |

**Pipeline:**

```
1. GitHub Actions → build & test
2. Push Docker image to Amazon ECR
3. AWS CodeDeploy → deploy to ECS Fargate
```

---

## 8. Error Handling

| Code | Message | Cause |
| --- | --- | --- |
| 400 | Bad Request | Validation failure |
| 401 | Unauthorized | Missing or invalid token |
| 403 | Forbidden | No access to resource |
| 404 | Not Found | Entity not found |
| 409 | Conflict | Duplicate entry |
| 500 | Internal Server Error | Unexpected runtime issue |

---

## 9. Security Overview

| Category | Implementation |
| --- | --- |
| **Authentication** | Keycloak OAuth2 → JWT tokens |
| **Authorization** | Role-based (Admin, Planner, Viewer) |
| **Transport Security** | HTTPS / TLS 1.2+ enforced |
| **Data Security** | No password storage (SSO only) |
| **Database Access** | IAM roles / secrets manager access to PostgreSQL |
| **Secrets Management** | Stored in AWS Secrets Manager |

---

## 10. Naming Conventions

| Context | Convention |
| --- | --- |
| DB Tables | snake_case |
| API Routes | kebab-case |
| Frontend Props | camelCase |
| Environment Vars | UPPER_CASE |
| DTOs | PascalCase |

---

## 11. Future Enhancements

-

---

## 12. Deployment Diagram

```
+-------------------------+        +-------------------------+
| Vue.js Frontend         | -----> | FastAPI Backend         |
| (AWS S3 + CloudFront)   |        | (AWS ECS Fargate)       |
+-------------------------+        +-------------------------+
          |                                    |
          |                                    v
          |                         +---------------------+
          |                         | PostgreSQL (AWS RDS)|
          |                         | Redis (AWS ElastiCache)|
          |                         +---------------------+
          |
          v
   +--------------------+
   | Keycloak / SSO     |
   +--------------------+
```