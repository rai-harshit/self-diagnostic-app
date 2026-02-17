# ARCHITECTURE.md - Self Diagnostic App

## Introduction

This document outlines the architectural vision for the "Self Diagnostic App," a new project focused on providing users with a native mobile experience to access diagnostic tools. The architecture is designed to be production-ready from day one, prioritizing simplicity, speed to build, and scalability using a full serverless backend and a modern React/TypeScript frontend stack.

Our primary goal is to deliver a robust, secure, and user-friendly application capable of handling a complete transactional flow—from user signup and content purchase to diagnostic submission—for a general consumer audience. This document serves as a comprehensive guide for the AI coding tool to ensure all components are built with a consistent and efficient architectural approach.

## Project Decisions

The following decisions were made during project planning, serving as the foundation for the architectural choices detailed below:

| Question | Decision |
| :-------------------------------------- | :--------------------------------------------------------------------------------------------------------------------- |
| What is the primary goal of this project | Build a production-ready v1                                                                                            |
| Who is the main user | Consumer / general audience                                                                                            |
| What platforms are in scope right now | Mobile app first, web later                                                                                            |
| Which primary frontend stack should be used | React / Next.js with TypeScript                                                                                        |
| What is the main outcome users should achieve in their first session | Complete a transactional flow (signup, purchase, submission)                                                           |
| Which backend style do you prefer | Full serverless (functions as a service, BaaS)                                                                         |
| What is the priority for architecture | Simplicity and speed to build                                                                                          |
| How should data be stored | Relational DB (PostgreSQL/MySQL, strong consistency)                                                                   |

---

## High-Level Overview

The Self Diagnostic App employs a mobile-first, serverless architecture centered around a core set of services accessible via a secure API Gateway. The frontend will be built with React Native for the mobile application, ensuring a performant and consistent user experience. The backend leverages Functions-as-a-Service (FaaS) for business logic, backed by a strongly consistent relational database for core data. Managed services are preferred across the board to maximize development velocity and operational simplicity while ensuring production readiness.

### Core Principles:

*   **Serverless First:** Minimize operational overhead and scale automatically.
*   **API-Centric:** All components communicate via well-defined APIs.
*   **Security by Design:** Authentication, authorization, and data protection are integral.
*   **Scalable & Resilient:** Leveraging cloud-native services for reliability.
*   **Developer Experience:** TypeScript for type safety and maintainability across the stack.

## Module Map

```mermaid
graph TD
    subgraph User Interfaces
        MA[Mobile Application (React Native)]
        WA(Web Application (Next.js - Later))
    end

    subgraph Backend Services
        APIGW(API Gateway)
        AUTH(Authentication Service)
        DE(Diagnostic Engine Service)
        PP(Payment Processing Service)
        AN(Analytics Service)
        NOTIF(Notification Service)
        CMS(Content Management System)
        FS(File Storage Service)
    end

    subgraph Data Stores
        UD(User Database)
        CD(Content Database)
    end

    subgraph Infrastructure
        SAS(Static Asset Storage)
        CDN(Content Delivery Network)
    end

    MA -- REST/GraphQL --> APIGW
    WA -- REST/GraphQL --> APIGW

    APIGW -- Route Requests --> AUTH
    APIGW -- Route Requests --> DE
    APIGW -- Route Requests --> PP
    APIGW -- Route Requests --> AN
    APIGW -- Route Requests --> NOTIF
    APIGW -- Route Requests --> CMS

    AUTH -- Reads/Writes --> UD
    DE -- Reads --> CD
    DE -- Reads/Writes --> UD
    PP -- Reads/Writes --> UD
    AN -- Writes --> AN_DB[Analytics Data Store]
    NOTIF -- Triggers --> External_Notif[Email/SMS/Push Providers]
    CMS -- Reads/Writes --> CD
    CMS -- Reads/Writes --> FS

    SAS -- Delivers --> CDN
    CDN -- Caches/Serves --> MA
    CDN -- Caches/Serves --> WA
    CDN -- Caches/Serves --> APIGW
    FS -- Stores Assets --> CDN

    style MA fill:#f9f,stroke:#333,stroke-width:2px
    style APIGW fill:#bbf,stroke:#333,stroke-width:2px
    style AUTH fill:#bbf,stroke:#333,stroke-width:2px
    style DE fill:#bbf,stroke:#333,stroke-width:2px
    style PP fill:#bbf,stroke:#333,stroke-width:2px
    style UD fill:#ccf,stroke:#333,stroke-width:2px
    style CD fill:#ccf,stroke:#333,stroke-width:2px
```

## Component Relationships and Data Flow

This section details each component's role and its interactions within the architecture.

1.  **Mobile Application (core)**
    *   **Description:** The primary user interface, built with **React Native** (TypeScript), targeting iOS and Android. It provides the full diagnostic workflow, user profile management, and purchase capabilities.
    *   **Interactions:** Communicates exclusively with the **Backend API Gateway** for all dynamic data, authentication, and transactional operations. Static assets are fetched via the **Content Delivery Network**.
    *   **Data Flow:**
        *   Sends user input, diagnostic requests, authentication credentials, and payment initiation requests to **API Gateway**.
        *   Receives diagnostic results, user profile data, content, and payment confirmations from **API Gateway**.
        *   Fetches static content (images, videos) from **CDN**.

2.  **Backend API Gateway (core)**
    *   **Description:** The single entry point for all external client requests (Mobile App, Web App, CMS). It handles request routing, basic validation, authentication enforcement, and rate limiting. Implemented using a managed service (e.g., AWS API Gateway).
    *   **Interactions:** Routes requests to the **Authentication Service**, **Diagnostic Engine Service**, **Payment Processing Service**, **Analytics Service**, and **Notification Service** (via internal service calls or direct function invocations). Can integrate directly with **CDN** for caching.
    *   **Data Flow:**
        *   Receives HTTP requests from **Mobile Application** (and **Web Application** later).
        *   Forwards requests to appropriate serverless functions.
        *   Returns responses from serverless functions back to clients.

3.  **Authentication Service (core)**
    *   **Description:** Manages user registration, login, session management (JWTs), password resets, and user profile data. It also handles authorization logic (e.g., checking user roles, content access rights). Implemented as a set of serverless functions (e.g., AWS Lambda) backed by a managed authentication service (e.g., AWS Cognito).
    *   **Interactions:** Persists user data in the **User Database**. Authenticates requests originating from the **API Gateway**.
    *   **Data Flow:**
        *   Receives user signup/login requests from **API Gateway**.
        *   Interacts with **User Database** to store/retrieve user credentials and profiles.
        *   Issues/validates JWTs for **API Gateway** and client applications.
        *   Notifies **Analytics Service** of user authentication events.

4.  **Diagnostic Engine Service (core)**
    *   **Description:** The core intelligence of the application. Processes user-submitted diagnostic input, applies business logic based on defined algorithms and content, and generates diagnostic results. Implemented as serverless functions.
    *   **Interactions:** Retrieves diagnostic questions and algorithms from the **Content Database**. Stores user's diagnostic history and results in the **User Database**.
    *   **Data Flow:**
        *   Receives diagnostic requests and user inputs from **API Gateway**.
        *   Queries **Content Database** for relevant questions, rules, and outcomes.
        *   Processes logic and generates diagnostic results.
        *   Stores diagnostic session history and results in **User Database**.
        *   Sends diagnostic completion events to **Analytics Service**.

5.  **User Database (core)**
    *   **Description:** A relational database (e.g., AWS RDS PostgreSQL) storing all user-centric data: user accounts, profiles, preferences, diagnostic history, purchase records, and subscription statuses. Ensures strong consistency for transactional data.
    *   **Interactions:** Accessed by **Authentication Service**, **Diagnostic Engine Service**, and **Payment Processing Service**.
    *   **Data Flow:**
        *   Stores user credentials (hashed), profile attributes, and session tokens.
        *   Records user diagnostic history and results.
        *   Maintains purchase and subscription records.

6.  **Content Database (core)**
    *   **Description:** A relational database (e.g., AWS RDS PostgreSQL) specifically for diagnostic content: questions, possible answers, diagnostic algorithms (e.g., rule sets, decision trees), explanations, and links to related media assets.
    *   **Interactions:** Primarily read by the **Diagnostic Engine Service**. Written to by the **Content Management System**.
    *   **Data Flow:**
        *   Stores diagnostic questions, options, logic rules, and result templates.
        *   Stores metadata for content assets (e.g., image URLs from **File Storage Service**).

7.  **Content Management System (recommended)**
    *   **Description:** A web-based application (could be a separate Next.js app or a SaaS CMS) allowing content creators to manage, create, and publish diagnostic content, questions, algorithms, and associated media.
    *   **Interactions:** Writes to the **Content Database** and uploads/manages files in the **File Storage Service**. Accesses via **API Gateway** (with appropriate administrative authorization).
    *   **Data Flow:**
        *   Sends content creation/update requests to **API Gateway**, which routes to dedicated backend functions.
        *   Uploads media files to **File Storage Service**.
        *   Receives content data from **API Gateway** for display and editing.

8.  **Web Application (recommended)**
    *   **Description:** A web-based interface (built with **Next.js and TypeScript**) for users to access the diagnostic application, planned for a later stage.
    *   **Interactions:** Will interact with the **Backend API Gateway** for all dynamic data and the **Content Delivery Network** for static assets, similar to the Mobile Application.
    *   **Data Flow:** (Future state, mirrors Mobile Application)

9.  **Analytics Service (recommended)**
    *   **Description:** Collects and processes user behavior, application usage, diagnostic flow progression, and performance data for business insights. Implemented using serverless functions for event ingestion (e.g., AWS Kinesis/Lambda) and a data warehousing solution.
    *   **Interactions:** Receives events from the **Mobile Application** (via **API Gateway**), **Authentication Service**, **Diagnostic Engine Service**, and **Payment Processing Service**.
    *   **Data Flow:**
        *   Receives usage events (e.g., "diagnostic started," "question answered," "purchase completed," "login success").
        *   Stores and processes data for reporting and dashboards.

10. **Notification Service (recommended)**
    *   **Description:** Manages sending various notifications to users, such as welcome emails, password reset links, diagnostic result summaries, subscription reminders, or marketing messages. Utilizes external email/SMS/push notification providers. Implemented as serverless functions.
    *   **Interactions:** Triggered by **Authentication Service** (e.g., welcome, password reset), **Diagnostic Engine Service** (e.g., results ready), and **Payment Processing Service** (e.g., purchase confirmation).
    *   **Data Flow:**
        *   Receives notification requests with user IDs and message templates.
        *   Fetches user contact details from **User Database** (if necessary).
        *   Calls external notification APIs (e.g., SendGrid, Twilio, Firebase Cloud Messaging).

11. **Payment Processing Service (core)**
    *   **Description:** Manages secure processing of user payments and subscriptions. Integrates with a third-party payment gateway (e.g., Stripe, PayPal). Implemented as serverless functions.
    *   **Interactions:** Creates/manages customer and subscription records in the **User Database**. Communicates with the chosen payment gateway.
    *   **Data Flow:**
        *   Receives payment initiation requests (e.g., purchase specific diagnostic content, subscription) from **API Gateway**.
        *   Interacts with the external payment gateway for secure transaction processing.
        *   Updates user's purchase/subscription status in the **User Database**.
        *   Sends payment success/failure events to **Analytics Service** and triggers **Notification Service** for confirmations.

12. **File Storage Service (recommended)**
    *   **Description:** Stores user-submitted files (e.g., attachments if supported) and static media assets (images, videos) used by the application content. A highly scalable object storage solution (e.g., AWS S3).
    *   **Interactions:** Uploads/downloads are managed by **Content Management System** (for content assets) and potentially the **Mobile Application** (for user-submitted files, via pre-signed URLs from **API Gateway**).
    *   **Data Flow:**
        *   Stores images, videos, documents related to diagnostic content or user submissions.
        *   Provides URLs for assets to **Content Database** and **Content Delivery Network**.

13. **Static Asset Storage (recommended)**
    *   **Description:** Dedicated storage for public static files, including frontend build artifacts (JavaScript bundles, CSS, HTML), images, and other assets served directly to clients. Typically a bucket in an object storage service (e.g., AWS S3).
    *   **Interactions:** Directly serves assets to the **Content Delivery Network**.
    *   **Data Flow:**
        *   Hosts compiled React Native web bundles (if using webviews), Web Application build outputs.
        *   Source for **Content Delivery Network**.

14. **Content Delivery Network (recommended)**
    *   **Description:** Distributes static assets and caches API responses closer to users for improved performance, reliability, and reduced latency. (e.g., AWS CloudFront).
    *   **Interactions:** Pulls assets from **Static Asset Storage** and **File Storage Service**. Can cache responses from the **API Gateway**.
    *   **Data Flow:**
        *   Serves cached static content (images, JS/CSS bundles) directly to **Mobile Application** and **Web Application**.
        *   Can cache specific API responses from **API Gateway** (e.g., publicly available content lists).

## Key Architectural Decisions and Rationale

1.  **Full Serverless Backend (FaaS + BaaS):**
    *   **Decision:** All backend logic will be implemented using serverless functions (e.g., AWS Lambda) fronted by an API Gateway (e.g., AWS API Gateway). Managed Backend-as-a-Service (BaaS) solutions (e.g., AWS Cognito for Auth) will be used where appropriate.
    *   **Rationale:** Aligns perfectly with "Full serverless (functions as a service, BaaS)" and "Simplicity and speed to build." This approach eliminates server provisioning and management, offers automatic scaling, pay-per-execution cost models, and high availability by default, making it ideal for a "production-ready v1" with minimal operational overhead.

2.  **Relational Database (PostgreSQL):**
    *   **Decision:** PostgreSQL will be the chosen database technology, deployed as a managed service (e.g., AWS RDS PostgreSQL).
    *   **Rationale:** Directly addresses "Relational DB (PostgreSQL/MySQL, strong consistency)." PostgreSQL provides strong transactional consistency, mature ACID properties, and robust querying capabilities essential for managing critical user data, diagnostic history, and payment records, especially for a "transactional flow (signup, purchase, submission)."

3.  **React Native for Mobile First, Next.js for Web:**
    *   **Decision:** The initial "Mobile Application" will be built using React Native with TypeScript. The "Web Application" (planned for later) will use Next.js with TypeScript.
    *   **Rationale:** Honors "Mobile app first, web later" and "React / Next.js with TypeScript." React Native allows for a native mobile experience while leveraging a single codebase (and skill set) for both iOS and Android, accelerating "speed to build." Next.js provides a powerful, performant, and SEO-friendly framework for the future web interface, maintaining stack consistency.

4.  **API Gateway as the Sole Backend Entry Point:**
    *   **Decision:** All external client requests *must* pass through the Backend API Gateway. There will be no direct client-to-function communication.
    *   **Rationale:** Essential for "production-ready v1." This centralizes security (authentication/authorization at the edge), request validation, routing, caching, and rate limiting. It provides a robust, manageable, and observable interface for the entire backend.

5.  **Managed Services Preference:**
    *   **Decision:** Whenever possible, prefer cloud provider managed services (e.g., AWS S3 for storage, AWS CloudFront for CDN, AWS Cognito for authentication, AWS RDS for database) over self-hosted solutions.
    *   **Rationale:** Directly supports "Simplicity and speed to build" and "production-ready v1." Managed services reduce the operational burden, accelerate development by abstracting infrastructure concerns, and typically come with built-in scalability, high availability, and security features.

## Invariants and Constraints

These are non-negotiable rules to maintain the integrity and principles of the architecture:

*   **Security First:** All data in transit and at rest must be encrypted. Authentication and authorization must be rigorously applied at the API Gateway and enforced by individual services. Never store raw user credentials.
*   **Stateless Serverless Functions:** Backend serverless functions (Lambdas) should be stateless. Any persistent data must be externalized to the User Database, Content Database, or File Storage Service. This ensures scalability and resilience.
*   **Relational Database as Source of Truth:** The User Database and Content Database are the authoritative sources for all persistent application data. Data integrity and strong consistency are paramount for transactional flows.
*   **API Gateway Centrality:** No client (mobile, web, CMS) may bypass the API Gateway to access backend services directly. All communication must flow through the API Gateway for security, monitoring, and control.
*   **TypeScript Strictness:** All frontend (React Native, Next.js) and backend serverless code must be written in TypeScript with strict type checking enabled to enhance code quality, maintainability, and reduce runtime errors.
*   **Loose Coupling, High Cohesion:** Services should be loosely coupled, communicating primarily via well-defined APIs. Each service should have high cohesion, being responsible for a single, well-defined domain.

## Database Schema Suggestions (PostgreSQL)

We will use a relational model, primarily within a single PostgreSQL instance for "User Database" and "Content Database" logical separation, leveraging schemas or distinct table prefixes if needed for clear organization.

```sql
-- Schema for User Database

-- Users Table (Authentication Service, Diagnostic Engine, Payment Processing)
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL, -- Stored securely
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_login_at TIMESTAMP WITH TIME ZONE,
    is_active BOOLEAN DEFAULT TRUE,
    preferences JSONB DEFAULT '{}' -- e.g., notification settings
);

-- User Diagnostic History Table (Diagnostic Engine)
CREATE TABLE user_diagnostics_history (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    diagnostic_id UUID NOT NULL REFERENCES content.diagnostics(id), -- Link to content
    started_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    completed_at TIMESTAMP WITH TIME ZONE,
    status VARCHAR(50) NOT NULL DEFAULT 'IN_PROGRESS', -- e.g., IN_PROGRESS, COMPLETED, CANCELLED
    user_inputs JSONB, -- Store user's specific answers/inputs for the session
    results JSONB, -- Store generated diagnostic results
    score INTEGER, -- Optional: a score or severity
    feedback_provided BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (diagnostic_id) REFERENCES content.diagnostics(id)
);

-- Payments Table (Payment Processing Service)
CREATE TABLE payments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    amount DECIMAL(10, 2) NOT NULL,
    currency VARCHAR(3) NOT NULL DEFAULT 'USD',
    payment_gateway_transaction_id VARCHAR(255) UNIQUE NOT NULL,
    status VARCHAR(50) NOT NULL, -- e.g., PENDING, SUCCESS, FAILED, REFUNDED
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    payment_method_details JSONB, -- Masked card info, PayPal ID, etc.
    item_type VARCHAR(50) NOT NULL, -- e.g., 'DIAGNOSTIC', 'SUBSCRIPTION_PLAN'
    item_id UUID -- References content.diagnostics.id or subscriptions.id
);

-- Subscriptions Table (Payment Processing Service)
CREATE TABLE subscriptions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    plan_id VARCHAR(100) NOT NULL, -- e.g., 'premium_monthly', 'basic_yearly'
    status VARCHAR(50) NOT NULL, -- e.g., ACTIVE, CANCELED, PAST_DUE
    start_date TIMESTAMP WITH TIME ZONE NOT NULL,
    end_date TIMESTAMP WITH TIME ZONE,
    auto_renew BOOLEAN DEFAULT TRUE,
    payment_id UUID REFERENCES payments(id), -- Last successful payment
    stripe_subscription_id VARCHAR(255), -- Or other gateway's subscription ID
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Schema for Content Database
CREATE SCHEMA content;

-- Diagnostic Definitions Table (Content Management System, Diagnostic Engine)
CREATE TABLE content.diagnostics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    category VARCHAR(100),
    estimated_time_minutes INTEGER,
    price DECIMAL(10, 2) NOT NULL DEFAULT 0.00, -- 0 for free diagnostics
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    algorithm_type VARCHAR(50) NOT NULL DEFAULT 'DECISION_TREE', -- e.g., DECISION_TREE, SCORE_BASED
    cover_image_url VARCHAR(2048) -- Link to File Storage Service
);

-- Questions Table (Content Management System, Diagnostic Engine)
CREATE TABLE content.questions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    diagnostic_id UUID NOT NULL REFERENCES content.diagnostics(id) ON DELETE CASCADE,
    question_text TEXT NOT NULL,
    question_type VARCHAR(50) NOT NULL, -- e.g., 'MULTIPLE_CHOICE', 'SINGLE_SELECT', 'TEXT_INPUT'
    order_in_diagnostic INTEGER NOT NULL,
    is_required BOOLEAN DEFAULT TRUE,
    media_url VARCHAR(2048), -- Link to File Storage Service (image, video, audio)
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Answers/Options Table (Content Management System, Diagnostic Engine)
CREATE TABLE content.answers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    question_id UUID NOT NULL REFERENCES content.questions(id) ON DELETE CASCADE,
    answer_text TEXT NOT NULL,
    value VARCHAR(255), -- Could be a score, a key, or next question ID
    is_correct BOOLEAN DEFAULT FALSE, -- If applicable for certain question types
    next_question_id UUID REFERENCES content.questions(id), -- For decision tree logic
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Content Assets Table (Content Management System, File Storage)
CREATE TABLE content.content_assets (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    asset_name VARCHAR(255) NOT NULL,
    file_type VARCHAR(50),
    s3_key VARCHAR(1024) UNIQUE NOT NULL, -- Key in S3 bucket
    cdn_url VARCHAR(2048), -- Full CDN URL for access
    uploaded_by UUID REFERENCES users(id), -- Optional: for tracking
    uploaded_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    description TEXT
);
```

**Indexing Suggestions:**
*   `users(email)`
*   `user_diagnostics_history(user_id, completed_at)`
*   `payments(user_id, status)`
*   `subscriptions(user_id, status)`
*   `content.diagnostics(is_active, category)`
*   `content.questions(diagnostic_id, order_in_diagnostic)`
*   `content.answers(question_id)`

## API Structure Suggestions

The API will be RESTful, served via the API Gateway, and implement JWT-based authentication (e.g., from AWS Cognito). All requests and responses will be JSON.

### Authentication Service Endpoints (`/auth`)

*   `POST /auth/signup`
    *   Request: `{ email, password, firstName, lastName }`
    *   Response: `{ userId, message, token }`
*   `POST /auth/login`
    *   Request: `{ email, password }`
    *   Response: `{ userId, token, refreshToken }`
*   `POST /auth/refresh`
    *   Request: `{ refreshToken }`
    *   Response: `{ token, refreshToken }`
*   `POST /auth/logout`
    *   Request: Headers: `Authorization: Bearer <token>`
    *   Response: `{ message: "Logged out successfully" }`
*   `GET /auth/me` (Authenticated)
    *   Response: `{ userId, email, firstName, lastName, preferences }`
*   `PUT /auth/me` (Authenticated)
    *   Request: `{ firstName, lastName, preferences }`
    *   Response: `{ userId, email, firstName, lastName, preferences }`

### Diagnostic Engine Service Endpoints (`/diagnostics`)

*   `GET /diagnostics` (Authenticated, optional: public for listing free diagnostics)
    *   Response: `[{ id, title, description, category, price, coverImageUrl, ... }]`
*   `GET /diagnostics/{diagnosticId}`
    *   Response: `{ id, title, description, questions: [{ id, questionText, type, order, mediaUrl, options: [{ id, answerText, value }] }], ... }`
*   `POST /diagnostics/{diagnosticId}/start` (Authenticated)
    *   Request: `{}`
    *   Response: `{ userDiagnosticHistoryId, diagnosticId, startedAt, status }`
*   `POST /diagnostics/{userDiagnosticHistoryId}/answer` (Authenticated)
    *   Request: `{ questionId, userAnswer, currentQuestionOrder }`
    *   Response: `{ userDiagnosticHistoryId, nextQuestionId, isCompleted, progress, ... }`
*   `POST /diagnostics/{userDiagnosticHistoryId}/submit` (Authenticated)
    *   Request: `{ finalAnswers }`
    *   Response: `{ userDiagnosticHistoryId, status: "COMPLETED", results: { ... }, score }`
*   `GET /diagnostics/history` (Authenticated)
    *   Response: `[{ userDiagnosticHistoryId, diagnosticId, title, status, completedAt, score }]`
*   `GET /diagnostics/history/{userDiagnosticHistoryId}` (Authenticated)
    *   Response: `{ ...full history record including inputs and results }`

### Payment Processing Service Endpoints (`/payments`)

*   `POST /payments/checkout` (Authenticated)
    *   Request: `{ itemType, itemId, amount, currency, paymentMethodNonce/token }`
    *   Response: `{ paymentId, status: "SUCCESS", transactionDetails }`
*   `POST /payments/subscribe` (Authenticated)
    *   Request: `{ planId, paymentMethodNonce/token }`
    *   Response: `{ subscriptionId, status: "ACTIVE" }`
*   `GET /payments/subscriptions` (Authenticated)
    *   Response: `[{ id, planId, status, startDate, endDate, ... }]`
*   `PUT /payments/subscriptions/{subscriptionId}/cancel` (Authenticated)
    *   Response: `{ subscriptionId, status: "CANCELED" }`

### File Storage Service Endpoints (via API Gateway for secure uploads)

*   `GET /files/upload-url` (Authenticated, for user-submitted files)
    *   Request: `{ fileName, fileType }`
    *   Response: `{ uploadUrl, s3Key, cdnUrl }` (Pre-signed S3 URL for direct client upload)
*   `GET /files/{s3Key}` (Authenticated, if private)
    *   Response: Binary file or redirect to CDN.

## File/Folder Structure Recommendation

We will adopt a multi-repository approach for clear separation of concerns, which aids in independent deployment and team autonomy, aligning with simplicity and speed for separate concerns. Shared types will be managed through a dedicated package or replicated across repos for initial speed.

### 1. Mobile Application (`./self-diagnostic-mobile`)

A standard React Native project structure.

```
self-diagnostic-mobile/
├── node_modules/
├── src/
│   ├── assets/
│   │   ├── images/
│   │   └── fonts/
│   ├── components/       # Reusable UI components
│   │   ├── Button/
│   │   │   └── index.tsx
│   │   └── Card/
│   │       └── index.tsx
│   ├── hooks/            # Custom React Hooks
│   ├── navigation/       # React Navigation setup
│   │   ├── AppNavigator.tsx
│   │   └── types.ts
│   ├── screens/          # Top-level screen components
│   │   ├── Auth/
│   │   │   ├── LoginScreen.tsx
│   │   │   └── SignUpScreen.tsx
│   │   ├── Diagnostics/
│   │   │   ├── DiagnosticListScreen.tsx
│   │   │   └── DiagnosticFlowScreen.tsx
│   │   └── Profile/
│   │       └── UserProfileScreen.tsx
│   ├── services/         # API clients, authentication logic
│   │   ├── api.ts        # Axios instance, API Gateway client
│   │   └── auth.ts
│   ├── store/            # State management (e.g., Zustand, React Context)
│   ├── themes/           # Styling, constants
│   ├── types/            # Shared TypeScript types/interfaces (can be shared via npm package)
│   │   ├── api.ts
│   │   └── models.ts
│   └── App.tsx
├── .env
├── app.json
├── babel.config.js
├── tsconfig.json
├── package.json
└── README.md
```

### 2. Web Application (`./self-diagnostic-web`) - *Future Scope*

A standard Next.js project structure.

```
self-diagnostic-web/
├── node_modules/
├── public/               # Static assets
├── src/
│   ├── app/              # Next.js App Router (or `pages` for Pages Router)
│   │   ├── (auth)/
│   │   │   ├── login/page.tsx
│   │   │   └── signup/page.tsx
│   │   ├── (main)/
│   │   │   ├── diagnostics/page.tsx
│   │   │   ├── diagnostics/[id]/page.tsx
│   │   │   └── profile/page.tsx
│   │   ├── layout.tsx
│   │   └── page.tsx      # Home page
│   ├── components/       # Reusable UI components
│   ├── hooks/
│   ├── lib/              # Utility functions, API clients
│   │   ├── api.ts
│   │   └── auth.ts
│   ├── styles/           # Global styles, Tailwind config
│   ├── types/            # Shared TypeScript types/interfaces
│   │   ├── api.ts
│   │   └── models.ts
│   └── globals.css
├── .env
├── next.config.js
├── tsconfig.json
├── package.json
└── README.md
```

### 3. Backend Services (`./self-diagnostic-backend`)

A monorepo for all serverless functions and shared backend code. Each service (Authentication, Diagnostic Engine, etc.) maps to a logical grouping of functions.

```
self-diagnostic-backend/
├── node_modules/
├── package.json
├── tsconfig.json
├── .env
├── serverless.yml        # Main Serverless Framework config file
├── src/
│   ├── shared/
│   │   ├── utilities/    # Common helper functions (e.g., date formatting, error handling)
│   │   ├── models/       # Shared DB models/interfaces (e.g., User, Diagnostic, Payment)
│   │   └── validation/   # Common validation schemas (e.g., Joi, Zod)
│   ├── auth-service/     # Authentication Service functions
│   │   ├── handlers/
│   │   │   ├── signup.ts
│   │   │   ├── login.ts
│   │   │   └── me.ts
│   │   ├── repository.ts # DB interactions for users
│   │   └── types.ts
│   ├── diagnostic-engine-service/ # Diagnostic Engine functions
│   │   ├── handlers/
│   │   │   ├── get-diagnostics.ts
│   │   │   ├── start-diagnostic.ts
│   │   │   ├── submit-answer.ts
│   │   │   └── get-history.ts
│   │   ├── repository.ts # DB interactions for diagnostic content/history
│   │   ├── logic.ts      # Core diagnostic algorithm logic
│   │   └── types.ts
│   ├── payment-processing-service/ # Payment Processing functions
│   │   ├── handlers/
│   │   │   ├── create-checkout-session.ts
│   │   │   └── handle-webhook.ts
│   │   ├── repository.ts # DB interactions for payments/subscriptions
│   │   ├── payment-gateway.ts # Wrapper for Stripe/other
│   │   └── types.ts
│   ├── content-management-service/ # CMS Backend functions (if custom)
│   │   ├── handlers/
│   │   │   ├── create-diagnostic.ts
│   │   │   └── upload-asset.ts
│   │   └── repository.ts # DB interactions for content
│   ├── analytics-service/  # Analytics ingestion functions
│   │   ├── handlers/
│   │   │   └── ingest-event.ts
│   │   └── types.ts
│   ├── notification-service/ # Notification functions
│   │   ├── handlers/
│   │   │   └── send-email.ts
│   │   └── email-provider.ts # Wrapper for SendGrid/SES
│   └── file-storage-service/ # Functions for managing S3/CDN (e.g., presigned URLs)
│       ├── handlers/
│       │   └── get-upload-presigned-url.ts
│       └── s3-client.ts
└── README.md
```