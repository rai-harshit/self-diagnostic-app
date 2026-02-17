# Self Diagnostic App - AI Development Prompts

This document provides a comprehensive set of instructions and guidelines for AI coding tools (e.g., Cursor, Copilot) to assist in building the "Self Diagnostic App". The goal is to produce a production-ready v1, prioritizing simplicity and speed, with a mobile-first approach, using a React/Next.js/TypeScript frontend and a full serverless backend with a relational database.

---

## 1. System Prompt

This prompt establishes the foundational context for all subsequent tasks.

```prompt
You are an expert software engineer specializing in modern full-stack development. You are tasked with building a production-ready v1 of a self-diagnostic mobile application.

**Project Goal:** Build a production-ready v1 of a self-diagnostic application for a consumer/general audience.
**Platform Priority:** Mobile app first (React Native), web later (Next.js).
**Frontend Stack:** React, Next.js, TypeScript. Use React Native for the mobile app.
**Backend Style:** Full serverless (Functions as a Service, Backend as a Service). Prioritize AWS services where concrete examples are needed, but keep the architecture general enough to be adaptable.
**Database:** Relational Database (PostgreSQL preferred for strong consistency).
**Architecture Priority:** Simplicity and speed to build. Avoid over-engineering.
**Key User Flow:** Users should be able to complete a transactional flow (signup, purchase, submission of a diagnostic query) in their first session.

**General Guidelines:**
*   **Code Quality:** Write clean, readable, maintainable, and well-documented TypeScript code.
*   **Error Handling:** Implement robust error handling for all services and UI components.
*   **Security:** Prioritize security best practices (e.g., input validation, authentication, authorization, secure data storage).
*   **Testing:** Include basic unit tests for critical functions and components.
*   **Naming Conventions:** Use camelCase for variables/functions, PascalCase for components/types.
*   **Modularity:** Break down features into small, testable, and reusable modules/components.
*   **Dependencies:** Use established, well-maintained libraries. Justify any new or complex dependencies.
*   **API Design:** Favor RESTful API principles for backend endpoints.
*   **Deployment:** Consider deployment implications (e.g., environment variables, build processes) in your code.
*   **Performance:** Optimize for good user experience, especially on mobile.

**Specific Stack Choices for Concrete Implementation Examples:**
*   **Mobile:** React Native (Expo recommended for faster iteration).
*   **Web:** Next.js (with TypeScript).
*   **Backend:** AWS Lambda (Node.js runtime), AWS API Gateway, AWS Cognito (for auth), AWS RDS (PostgreSQL).
*   **Database ORM/Query Builder:** Prisma or Drizzle ORM for TypeScript interaction with PostgreSQL.
*   **File Storage:** AWS S3.
*   **CDN:** AWS CloudFront.

I will provide specific prompts for each component. Ensure all code generated adheres to these guidelines and stack choices.
```

---

## 2. Style & Stack Constraints

These are overarching guidelines for all code generation tasks.

*   **Language:** TypeScript strictly.
*   **Frontend Frameworks:** React Native for mobile, Next.js for web.
*   **Backend Architecture:** Serverless functions (e.g., AWS Lambda).
*   **Database:** PostgreSQL, accessed via a robust ORM like Prisma or Drizzle ORM.
*   **Styling (Frontend):** For React Native, use `StyleSheet.create`. For Next.js, use a modern CSS-in-JS solution (e.g., Styled Components, Emotion) or utility-first CSS (Tailwind CSS) for simplicity and speed.
*   **State Management (Frontend):** React Context API or Zustand/Jotai for simplicity; avoid Redux unless explicitly necessary for complex global state.
*   **API Communication (Frontend):** `fetch` API or `axios`.
*   **Configuration:** Environment variables (`.env` files, AWS SSM for production).
*   **Logging:** Basic console logging initially, consider structured logging for production.
*   **API Responses:** Standardized JSON format, including error objects.

---

## 3. Common Patterns & Conventions

*   **Folder Structure:** Organize by feature or domain, e.g., `src/features/auth`, `src/services`, `src/components`.
*   **Data Models:** Define clear TypeScript interfaces/types for all data structures (API request/response, database entities).
*   **Service Layer (Backend):** Isolate business logic into separate service files/modules that interact with the database or other external services.
*   **API Gateway Integration:** Use API Gateway as the entry point for all serverless functions.
*   **Authentication Flow:** OAuth 2.0 / OIDC with AWS Cognito for user authentication.
*   **Database Migrations:** Use an ORM's migration system (e.g., Prisma Migrate or Drizzle Migrations).
*   **Error Handling:** Custom error classes for different error types (e.g., `NotFoundError`, `UnauthorizedError`).
*   **Constants/Enums:** Define application-wide constants and enums in dedicated files.

---

## 4. Things to Avoid (Anti-patterns, Wrong Libraries, etc.)

*   **Over-engineering:** Do not introduce unnecessary complexity (e.g., microservices for every single function, overly complex design patterns, multiple message queues) unless explicitly requested.
*   **Blocking I/O:** Use asynchronous patterns everywhere, especially in serverless functions.
*   **Excessive Global State (Frontend):** Avoid excessive use of global state if local component state or simpler hooks suffice.
*   **Direct Database Access from Frontend:** Never directly expose database credentials or allow direct access from client-side code. All database interactions must happen via secure backend APIs.
*   **Hardcoded Secrets:** All sensitive information must be stored as environment variables or in a secure secrets manager (e.g., AWS Secrets Manager).
*   **Monolithic Frontend/Backend Codebase:** Keep frontend and backend codebases separate for clear boundaries, even within a monorepo.
*   **Unoptimized Images/Assets:** Consider compression and CDN usage from the start to ensure good performance.

---

## 5. Component-Specific Continuation Prompts

These prompts are designed to be used sequentially or independently for building specific parts of the project. Each prompt assumes the **System Prompt** and **General Guidelines** are already established.

### 5.1. Mobile Application (React Native)

```prompt
Implement the core "Mobile Application" using React Native with Expo and TypeScript. Focus on setting up the initial project structure, navigation, and a basic onboarding flow that includes signup/login. Ensure the UI is clean and user-friendly for a consumer audience.

**Requirements:**
1.  Initialize a new Expo project with TypeScript (`expo init my-app --template blank-typescript`).
2.  Set up React Navigation (Stack Navigator for authentication flow, Tab Navigator for main app content).
3.  Create a `SignInScreen`, `SignUpScreen`, and a placeholder `DashboardScreen` (post-login).
4.  Implement basic form validation for signup/login fields (e.g., email format, password strength).
5.  Showcase how to integrate with the `Authentication Service` by defining an API client (assume API endpoints like `/auth/signup`, `/auth/signin`).
6.  Ensure styling follows React Native best practices (`StyleSheet.create` for components).
7.  Add a basic splash screen and app icon using Expo configuration.
8.  Include basic error display for API call failures (e.g., using an alert or a temporary banner).
```

### 5.2. Backend API Gateway (AWS API Gateway)

```prompt
Set up the "Backend API Gateway" using AWS API Gateway. Design a basic RESTful API structure that will serve as the entry point for all external requests to the serverless functions.

**Requirements:**
1.  Define a root API Gateway (`/api/v1`) using Serverless Framework or AWS SAM.
2.  Create placeholder routes for core functionalities:
    *   Authentication: `/auth/{proxy+}` (for signup, signin, etc.)
    *   Diagnostic Queries: `/diagnose/{proxy+}` (for submit, history)
    *   User Profiles: `/users/{proxy+}` (for getting user info)
    *   Payments: `/payment/{proxy+}` (for initiating payments, webhooks)
3.  Configure each placeholder route to integrate with a mock AWS Lambda function (e.g., a simple "hello world" Node.js Lambda) for initial setup.
4.  Implement comprehensive CORS configuration to allow requests from the React Native and Next.js applications (specifying allowed origins, methods, headers).
5.  Describe how to secure API endpoints using an AWS Cognito Authorizer, demonstrating its application to a protected route (e.g., `/users/me`).
6.  Provide a basic `serverless.yml` or AWS SAM template structure defining these API Gateway resources and their Lambda integrations.
```

### 5.3. Authentication Service (AWS Lambda, Cognito, RDS)

```prompt
Implement the "Authentication Service" using AWS Lambda, AWS Cognito for user pool management, and PostgreSQL (via RDS) for storing extended user profiles. This service will handle user registration, login, token management, and basic user profile management.

**Requirements:**
1.  Set up an AWS Cognito User Pool with email as the primary alias for user authentication (signup, signin, password reset, email verification).
2.  Create AWS Lambda functions (Node.js with TypeScript) for API Gateway integration:
    *   `signup`: Registers a user in Cognito.
    *   `signin`: Authenticates user against Cognito, returns JWT tokens.
    *   `refreshToken`: Refreshes access tokens using refresh tokens.
    *   `forgotPassword`, `resetPassword`: Handle password recovery.
3.  Implement a Cognito `PostConfirmation` trigger Lambda function. This Lambda will:
    *   Be invoked after a user successfully confirms their account in Cognito.
    *   Store a basic user profile (e.g., `id` (UUID), `cognitoId`, `email`, `createdAt`, `updatedAt`) in the `users` table of the PostgreSQL `User Database`.
4.  Define API Gateway endpoints for all the Lambda functions listed above.
5.  Define clear TypeScript interfaces for all request/response payloads (e.g., `SignUpRequest`, `SignInResponse`).
6.  Use Prisma or Drizzle ORM to interact with the PostgreSQL database for user profile storage.
7.  Include robust error handling (e.g., for Cognito errors, database errors) and structured logging.
```

### 5.4. Diagnostic Engine Service (AWS Lambda, RDS)

```prompt
Implement the "Diagnostic Engine Service" using AWS Lambda and PostgreSQL. This service will process diagnostic queries from users, apply logic based on content, and generate results.

**Requirements:**
1.  Create AWS Lambda functions (Node.js with TypeScript) to handle diagnostic requests.
2.  Define an API Gateway endpoint (e.g., `POST /diagnose/submit`) that accepts user input (e.g., `symptoms: string[]`, `answers: Record<string, string>`). This endpoint must be secured with a Cognito Authorizer.
3.  Implement basic diagnostic logic within the Lambda:
    *   Fetch relevant diagnostic `questions` and `algorithms` from the `Content Database` based on initial user input or session context.
    *   Process user answers against predefined rules/algorithms (start with a simple rule-based system or decision tree defined in the `algorithms` table `rules` JSONB field).
    *   Generate a preliminary diagnostic result or the next set of questions to ask the user.
4.  Store a history of diagnostic sessions and results in the `diagnostic_sessions` table of the `User Database`, linking it to the authenticated `userId`.
5.  Use Prisma or Drizzle ORM to interact with both `User Database` and `Content Database`.
6.  Include robust input validation, error handling (e.g., for invalid input, missing content, logic errors), and comprehensive logging.
```

### 5.5. User Database (AWS RDS PostgreSQL)

```prompt
Design and set up the "User Database" using AWS RDS for PostgreSQL. This database will store user accounts, preferences, diagnostic history, and purchase records.

**Requirements:**
1.  Define the database schema using Prisma schema syntax or Drizzle ORM schema for PostgreSQL.
2.  Include the following tables:
    *   `User`:
        *   `id`: UUID (Primary Key)
        *   `cognitoId`: String (Unique, links to Cognito User Pool)
        *   `email`: String (Unique, user's email)
        *   `firstName`: String (Optional)
        *   `lastName`: String (Optional)
        *   `preferences`: JSONB (Stores user-specific settings, e.g., `{ "notification_enabled": true }`)
        *   `createdAt`: DateTime
        *   `updatedAt`: DateTime
    *   `DiagnosticSession`:
        *   `id`: UUID (Primary Key)
        *   `userId`: UUID (Foreign Key to `User.id`)
        *   `startTime`: DateTime
        *   `endTime`: DateTime (Optional, when session completes)
        *   `inputData`: JSONB (User's initial symptoms, answers)
        *   `results`: JSONB (Diagnostic outcome, recommendations)
        *   `status`: Enum (`PENDING`, `COMPLETED`, `CANCELLED`)
        *   `createdAt`: DateTime
        *   `updatedAt`: DateTime
    *   `Purchase`:
        *   `id`: UUID (Primary Key)
        *   `userId`: UUID (Foreign Key to `User.id`)
        *   `stripePaymentIntentId`: String (Unique, from Stripe)
        *   `amount`: Decimal
        *   `currency`: String (e.g., 'USD')
        *   `status`: Enum (`PENDING`, `COMPLETED`, `FAILED`, `REFUNDED`)
        *   `purchaseDate`: DateTime
        *   `itemId`: String (Identifier for what was purchased, e.g., 'premium_subscription', 'single_diagnostic')
        *   `itemType`: Enum (`SUBSCRIPTION`, `ONE_TIME_PRODUCT`)
3.  Generate initial migrations for these tables using the chosen ORM.
4.  Provide examples of ORM queries for:
    *   Creating a new user profile upon Cognito post-confirmation.
    *   Logging a new diagnostic session.
    *   Updating a diagnostic session with results.
    *   Fetching a user's entire diagnostic history.
    *   Recording a new purchase.
5.  Ensure proper indexing for common query patterns (e.g., `userId` on `DiagnosticSession` and `Purchase`, `email` on `User`).
```

### 5.6. Content Database (AWS RDS PostgreSQL)

```prompt
Design and set up the "Content Database" using AWS RDS for PostgreSQL. This database will house all diagnostic content, questions, algorithms, and related media assets.

**Requirements:**
1.  Define the database schema using Prisma schema syntax or Drizzle ORM schema for PostgreSQL.
2.  Include the following tables:
    *   `DiagnosticTopic`:
        *   `id`: UUID (Primary Key)
        *   `name`: String (e.g., "Headache Diagnosis", "Fever Assessment")
        *   `description`: String
        *   `category`: String (e.g., "General", "Neurology")
        *   `isActive`: Boolean (For publishing/unpublishing content)
        *   `createdAt`: DateTime
        *   `updatedAt`: DateTime
    *   `Question`:
        *   `id`: UUID (Primary Key)
        *   `topicId`: UUID (Foreign Key to `DiagnosticTopic.id`)
        *   `text`: String (The actual question)
        *   `type`: Enum (`MULTIPLE_CHOICE`, `SINGLE_CHOICE`, `FREE_TEXT`, `YES_NO`)
        *   `options`: JSONB (e.g., `["Yes", "No"]` or `[{"label": "Mild", "value": "mild"}]` for multiple choice)
        *   `order`: Int (Order within a topic's flow)
        *   `mediaAssetId`: UUID (Optional, Foreign Key to `MediaAsset.id` for question-related images/videos)
        *   `createdAt`: DateTime
        *   `updatedAt`: DateTime
    *   `Algorithm`:
        *   `id`: UUID (Primary Key)
        *   `topicId`: UUID (Foreign Key to `DiagnosticTopic.id`)
        *   `name`: String (e.g., "Headache Severity Algorithm")
        *   `rules`: JSONB (Represents a simple decision tree or rule set, e.g., `[{"questionId": "...", "answer": "...", "nextQuestionId": "..."}, {"condition": "...", "result": "..."}]`)
        *   `version`: String (e.g., '1.0.0')
        *   `isActive`: Boolean
        *   `createdAt`: DateTime
        *   `updatedAt`: DateTime
    *   `MediaAsset`:
        *   `id`: UUID (Primary Key)
        *   `fileName`: String
        *   `url`: String (Full URL to the S3 object)
        *   `mimeType`: String (e.g., 'image/jpeg')
        *   `altText`: String (Optional, for accessibility)
        *   `uploadedByUserId`: UUID (Foreign Key to `User.id` in User Database, if CMS users are in UserDB)
        *   `createdAt`: DateTime
        *   `updatedAt`: DateTime
3.  Generate initial migrations for these tables using the chosen ORM.
4.  Provide examples of ORM queries for:
    *   Fetching all questions for a specific diagnostic topic.
    *   Retrieving an algorithm's rules.
    *   Creating new diagnostic content (topic, question, algorithm).
    *   Linking media assets to questions.
5.  Ensure proper indexing for foreign keys and `topicId` for efficient content retrieval.
```

### 5.7. Content Management System (Next.js, Admin Panel)

```prompt
Implement a basic "Content Management System" (CMS) as a web application using Next.js with TypeScript. This CMS will allow content creators to manage and publish diagnostic content.

**Requirements:**
1.  Set up a new Next.js project with TypeScript.
2.  Create an authenticated admin dashboard accessible only to authorized users. This should integrate with the `Authentication Service` (e.g., via Cognito User Pool for admin users, or simple token-based authentication for internal tools).
3.  Implement CRUD (Create, Read, Update, Delete) interfaces for `DiagnosticTopic`, `Question`, and `Algorithm` from the `Content Database`. These interfaces should be intuitive for content creators.
4.  Integrate file uploads for `MediaAsset` records:
    *   When creating/updating content, allow content creators to upload images/files.
    *   This should leverage the `File Storage Service` (AWS S3 pre-signed URLs) for secure uploads.
    *   Store the resulting S3 URL in the `MediaAsset` table.
5.  Use a simple UI component library (e.g., Chakra UI, Ant Design, or even basic HTML/CSS with Tailwind CSS) for rapid development of forms and tables.
6.  Define API routes in Next.js (or separate dedicated Lambda functions for the CMS backend) that interact with the `Content Database` using the chosen ORM.
7.  Ensure proper form validation, error handling, and user feedback for all content submission and modification actions.
8.  Prioritize simplicity for v1; focus on core content management functionalities, not on advanced CMS features like versioning or workflows.
```

### 5.8. Web Application (Next.js)

```prompt
Implement the "Web Application" using Next.js with TypeScript. This will be a secondary interface for users to access the diagnostic application, mirroring the mobile app's functionality.

**Requirements:**
1.  Set up a new Next.js project with TypeScript, configured for server-side rendering (SSR) or static site generation (SSG) where appropriate.
2.  Aim for code reusability with the mobile app where possible (e.g., shared API client logic, shared types/interfaces).
3.  Implement the same core user flow as the mobile app: signup, signin, and submitting a diagnostic query.
4.  Create corresponding pages for `SignIn`, `SignUp`, `Dashboard`, and `DiagnosticFlow` that are responsive for various screen sizes.
5.  Integrate with the `Authentication Service`, `Diagnostic Engine Service`, and `Payment Processing Service` via the `Backend API Gateway`.
6.  Use a consistent styling approach (e.g., Tailwind CSS or a simple CSS-in-JS solution) across the application.
7.  Implement client-side state management for the diagnostic flow (e.g., using React Context or Zustand/Jotai).
8.  Display the user's diagnostic history and purchased items, retrieved from the `User Database` via API.
```

### 5.9. Analytics Service (AWS Lambda, Kinesis/Firehose, S3)

```prompt
Set up a basic "Analytics Service" to collect and process user behavior and application performance data.

**Requirements:**
1.  Design a simple event schema for user actions (e.g., `{ eventName: string, userId: string, timestamp: string, data: Record<string, any> }`). Examples: `user_signed_up`, `diagnostic_started`, `diagnostic_completed`, `item_purchased`, `screen_viewed`.
2.  Create an AWS Lambda function (Node.js with TypeScript) that acts as an event receiver (e.g., `analyticsProcessor`). This Lambda will:
    *   Accept incoming analytics events via an API Gateway endpoint (e.g., `POST /analytics/event`).
    *   Perform basic validation on the event structure.
    *   Batch and send events to an AWS Kinesis Firehose delivery stream.
3.  Set up an AWS Kinesis Firehose delivery stream to ingest events from the `analyticsProcessor` Lambda.
4.  Configure the Firehose stream to deliver the raw events to an AWS S3 bucket (e.g., `my-app-analytics-raw-data`) for cost-effective storage.
5.  Implement a client-side utility in both React Native and Next.js applications to send these events to the `/analytics/event` API Gateway endpoint.
6.  Prioritize simplicity; complex real-time dashboards or advanced analytics processing are not needed for v1, just reliable data collection and raw storage.
```

### 5.10. Notification Service (AWS SES, SNS)

```prompt
Implement a basic "Notification Service" to handle sending various notifications to users via email and push notifications (for mobile).

**Requirements:**
1.  Set up AWS SES (Simple Email Service) for sending transactional emails (e.g., welcome emails, password reset links, purchase confirmations). Configure it to send from a verified domain/email.
2.  Create AWS Lambda functions (Node.js with TypeScript) that can be invoked by other services (e.g., Auth Service, Payment Service) to send notifications:
    *   `sendEmailNotification`: Takes recipient, subject, and body (or template name + data) as input, uses SES to send.
    *   `sendPushNotification`: Takes `userId` and message payload as input.
3.  For email notifications:
    *   Implement a function to send a templated email using SES. Provide an example of sending a "Welcome Email" after a user signs up (triggered by the `Authentication Service`).
4.  For push notifications (mobile):
    *   Outline the integration strategy with AWS SNS for mobile push notifications (e.g., for FCM/APNS endpoints).
    *   Describe how the mobile app would register its device token with the backend (e.g., a `POST /users/me/device-token` API endpoint). This token should be stored in the `User Database` (e.g., in `User`'s `preferences` JSONB or a separate `UserDevice` table).
    *   Implement the `sendPushNotification` Lambda to use SNS to send a push notification to a specific user's registered device(s).
5.  Define clear API interfaces (e.g., `POST /notify/email`, `POST /notify/push`) for other services to request notifications, secured with appropriate authorization.
```

### 5.11. Payment Processing Service (Stripe, AWS Lambda)

```prompt
Implement the "Payment Processing Service" to manage secure processing of user payments and subscriptions. Use Stripe for payment gateway integration.

**Requirements:**
1.  Integrate Stripe as the payment gateway.
2.  Create AWS Lambda functions (Node.js with TypeScript) for backend payment operations, secured with a Cognito Authorizer where appropriate:
    *   `createPaymentIntent`: (e.g., `POST /payment/intent`) Initiates a one-time payment, creating a Stripe `PaymentIntent`.
    *   `createCheckoutSession`: (e.g., `POST /payment/checkout`) For subscription checkout using Stripe Checkout.
    *   `handleStripeWebhook`: (e.g., `POST /payment/webhook`) Processes Stripe webhooks (e.g., `payment_succeeded`, `invoice.paid`, `customer.subscription.created`, `customer.subscription.deleted`). This endpoint must be publicly accessible by Stripe but securely verified using Stripe's webhook secret.
3.  Define API Gateway endpoints for these Lambda functions.
4.  Ensure all sensitive payment information is handled securely on the backend (Stripe's servers) and never directly exposed to the client.
5.  On successful payment/subscription (via webhook), update the `Purchase` table in the `User Database` and potentially trigger the `Notification Service` for a receipt.
6.  Implement robust error handling for payment failures and comprehensive logging.
7.  Describe client-side integration with Stripe using official SDKs:
    *   For React Native: using `@stripe/stripe-react-native` or similar, integrating with `createPaymentIntent`.
    *   For Next.js: using `@stripe/react-stripe-js` and `stripe-js`, integrating with `createPaymentIntent` or redirecting to Stripe Checkout.
```

### 5.12. File Storage Service (AWS S3)

```prompt
Set up the "File Storage Service" using AWS S3 for storing user-submitted files (e.g., diagnostic-related images) and other application assets requiring dynamic access.

**Requirements:**
1.  Create an AWS S3 bucket (e.g., `my-app-user-uploads`) specifically for user-submitted files.
2.  Define appropriate bucket policies for security:
    *   Restrict public access by default.
    *   Allow specific IAM roles (used by Lambda functions) to read/write.
3.  Create an AWS Lambda function (Node.js with TypeScript, e.g., `getSignedUploadUrl`) that generates pre-signed URLs for secure uploads from the client-side. This avoids exposing AWS credentials to the frontend.
4.  Implement an API Gateway endpoint (e.g., `POST /storage/upload-url`) that the client can call, providing desired file type and size. This endpoint should be secured with a Cognito Authorizer.
5.  The `getSignedUploadUrl` Lambda should return a pre-signed PUT URL and a corresponding GET URL (or the file key) to the client.
6.  Describe how to use these pre-signed PUT URLs in the React Native and Next.js applications to directly upload files to S3.
7.  Outline how to retrieve files: either by generating pre-signed GET URLs on demand for private files or by storing a public URL for non-sensitive assets directly in the database (e.g., `MediaAsset.url`).
```

### 5.13. Static Asset Storage (AWS S3)

```prompt
Set up "Static Asset Storage" using AWS S3 for frontend build artifacts, images, and other publicly accessible assets (distinct from user-submitted files).

**Requirements:**
1.  Create a dedicated AWS S3 bucket (e.g., `my-app-frontend-assets`) for static assets for both the Next.js web application and potentially assets for the React Native app.
2.  Configure the bucket for static website hosting if serving directly, or primarily as an origin for CloudFront.
3.  Describe how to configure a Next.js build process to output its static files (HTML, CSS, JS, images) to this bucket (e.g., using `next export` or a CI/CD pipeline step with `aws s3 sync`).
4.  Explain how to set up public read access for these assets (e.g., using a bucket policy that allows `s3:GetObject` for `*`).
5.  Mention that this bucket will serve as an origin for the `Content Delivery Network` (AWS CloudFront) to improve performance and reliability.
```

### 5.14. Content Delivery Network (AWS CloudFront)

```prompt
Set up a "Content Delivery Network" using AWS CloudFront to distribute static assets and cache API responses.

**Requirements:**
1.  Create an AWS CloudFront distribution.
2.  Configure multiple origins for the distribution:
    *   One origin pointing to the `Static Asset Storage` S3 bucket for frontend build artifacts and publicly accessible content.
    *   Another origin pointing to the `Backend API Gateway` (specifically its custom domain or invoke URL).
3.  Configure Cache Behaviors for different paths:
    *   Default behavior (`/*`) to point to the static assets S3 origin, optimized for caching static content (e.g., aggressive caching headers for immutable assets).
    *   A specific behavior for API paths (e.g., `/api/v1/*`) pointing to the API Gateway origin.
4.  Implement basic caching policies for API routes within CloudFront where appropriate (e.g., cache `GET /api/v1/diagnose/content` for a short duration if content updates are not real-time critical). Ensure authorization headers are passed through for authenticated API calls.
5.  Ensure invalidation strategies are considered for content updates (e.g., how to invalidate CloudFront cache when a new frontend build is deployed or when static content in S3 is updated).
6.  Describe how to associate a custom domain (e.g., `app.mydomain.com`, `api.mydomain.com`) with the CloudFront distribution using AWS Certificate Manager.
```

---

## 6. Getting Started Mega-Prompt

This prompt will kick off the entire project by setting up the foundational elements.

```prompt
**Goal:** Initialize the "Self Diagnostic App" project by setting up the foundational elements for a production-ready v1, focusing on mobile-first, and establishing a robust monorepo structure.

**Tasks:**
1.  **Monorepo Setup:**
    *   Create a new root directory `self-diagnostic-app`.
    *   Inside, initialize a monorepo using `npm init -y` and configure `npm workspaces` (or `pnpm workspaces` or `yarn workspaces`) for:
        *   `apps/mobile` (for the React Native application)
        *   `apps/web` (for the Next.js application)
        *   `packages/backend` (for serverless functions, database schema, and shared types)
    *   Add a root `tsconfig.json`, `eslint.config.js`, and `prettier.config.js` with configurations applicable to all sub-projects, promoting code consistency.
    *   Create a root `README.md` file outlining the project structure and initial setup instructions.
2.  **Mobile Application Initial Setup (`apps/mobile`):**
    *   Initialize `apps/mobile` with Expo and TypeScript (`npx create-expo-app@latest apps/mobile --template blank-typescript`).
    *   Install `react-navigation/native` and `@react-navigation/stack` and set up a basic navigation structure in `apps/mobile/src/navigation/AppNavigator.tsx`.
    *   Create placeholder `SignInScreen.tsx`, `SignUpScreen.tsx` in `apps/mobile/src/screens/Auth`.
    *   Implement a minimal `App.tsx` to display the authentication navigation flow.
3.  **Backend Core Setup (`packages/backend`):**
    *   Initialize `packages/backend` as a Node.js TypeScript project (`npm init -y && npx tsc --init`).
    *   Install `serverless` framework and initialize a new `serverless.yml` for an AWS Node.js TypeScript project.
    *   Define the `users` table schema in `packages/backend/prisma/schema.prisma` (or Drizzle equivalent) for the `User Database` with `id` (UUID), `email` (unique), `cognitoId`, `createdAt`, `updatedAt`.
    *   Install Prisma (or Drizzle) client and generate initial migrations.
    *   Create a placeholder Lambda function (`packages/backend/src/functions/auth/signup.ts`) that will simulate user creation (return a mock user ID) and demonstrate Prisma/Drizzle client initialization.
    *   Configure this Lambda with an API Gateway endpoint in `serverless.yml`: `POST /api/v1/auth/signup`.
    *   Include a `Dockerfile` for local development if relevant for database setup or for future containerized deployments (optional but good practice for production-ready).
4.  **Configuration & Environment Variables:**
    *   Add `.env` files to `apps/mobile` and `packages/backend`.
    *   Define a placeholder `API_BASE_URL` in the frontend `.env` files, pointing to a local or mock backend URL.
    *   Define placeholder `DATABASE_URL` and `COGNITO_USER_POOL_ID` in the backend `.env` file.
    *   Provide clear instructions in the root `README.md` on how to set up local environment variables and AWS credentials.
5.  **Initial API Client:**
    *   In `apps/mobile/src/api/auth.ts`, create a simple TypeScript API client using `fetch` or `axios` to call the `/api/v1/auth/signup` endpoint.
    *   Showcase a basic signup flow integration: `SignUpScreen` form submission calls this API client -> (mock) success/error message display.

**Output:**
*   A well-structured monorepo with `apps/mobile`, `apps/web` (empty for now), and `packages/backend`.
*   `package.json` files for all projects with necessary dependencies.
*   Basic `App.tsx`, `SignInScreen.tsx`, `SignUpScreen.tsx` for mobile.
*   `serverless.yml` for backend with a placeholder `signup` Lambda and API Gateway configuration.
*   `prisma/schema.prisma` (or Drizzle schema) for the `User` table.
*   `tsconfig.json`, `eslint.config.js`, `prettier.config.js` at the root and for each package.
*   A comprehensive `README.md` in the root summarizing setup instructions, how to run each part (mobile, backend local), and initial architectural overview.

Remember to prioritize simplicity and speed while laying a solid foundation for production-ready v1.
```