# RUNBOOK.md: Self-Diagnostic Application (Production v1)

Welcome to the runbook for our new Self-Diagnostic Application! This document outlines the project's architecture, setup instructions, and operational guidelines, tailored to achieve a production-ready v1 with a focus on simplicity and speed for our mobile-first, consumer-facing application.

---

## Project Decisions

The following decisions were made during project planning, shaping the architecture and development approach outlined in this runbook:

| Question | Decision |
|----------|----------|
| What is the primary goal of this project | Build a production-ready v1 |
| Who is the main user | Consumer / general audience |
| What platforms are in scope right now | Mobile app first, web later |
| Which primary frontend stack should be used | React / Next.js with TypeScript |
| What is the main outcome users should achieve in their first session | Complete a transactional flow (signup, purchase, submission) |
| Which backend style do you prefer | Full serverless (functions as a service, BaaS) |
| What is the priority for architecture | Simplicity and speed to build |
| How should data be stored | Relational DB (PostgreSQL/MySQL, strong consistency) |

---

## 1. Project Overview

This project aims to build a Self-Diagnostic Application, starting with a native mobile experience, followed by a web application. The core functionality revolves around guiding users through diagnostic processes, capturing their input, providing results, and managing their profiles, history, and subscriptions. The architecture prioritizes a full serverless backend on AWS for scalability and ease of operations, with a strong relational database for data consistency.

Our immediate goal is to deliver a production-ready v1 that enables users to complete a full transactional flow: sign up, make a purchase (e.g., for premium diagnostic access), and submit their first diagnostic query.

---

## 2. Prerequisites

Ensure you have the following installed and configured on your development machine:

*   **Node.js (LTS Version)**: Required for both frontend (React Native) and backend (Serverless Framework, AWS Lambda functions).
    *   `node -v` (e.g., v18.x or v20.x)
*   **npm or yarn**: Package manager for Node.js projects.
    *   `npm -v` or `yarn -v`
*   **Git**: Version control.
    *   `git --version`
*   **AWS CLI**: To configure AWS credentials and interact with AWS services.
    *   `aws --version`
    *   Configure with `aws configure` (requires AWS Access Key ID, Secret Access Key, and default region).
*   **Serverless Framework CLI**: For deploying serverless applications to AWS.
    *   `npm install -g serverless`
    *   `serverless --version`
*   **React Native CLI**: For mobile application development.
    *   `npm install -g react-native-cli`
    *   Refer to the [React Native Environment Setup](https://reactnative.dev/docs/environment-setup) for detailed instructions on installing Xcode (macOS) / Android Studio (all OS) and configuring emulators/simulators.
*   **Docker Desktop**: Recommended for running a local PostgreSQL database for development.
    *   `docker --version`
*   **PostgreSQL Client**: A GUI tool like DBeaver, TablePlus, or pgAdmin for database inspection.

---

## 3. Environment Variables

Environment variables are crucial for configuring the application across different environments (development, staging, production). These will typically be managed by the Serverless Framework for backend deployments and by a `.env` file (or build configuration) for frontend projects.

**Placeholder Values:**
*   `us-east-1`: Replace with your desired AWS region.
*   `YOUR_..._KEY`: Replace with actual keys/secrets.
*   `YOUR_DB_...`: Replace with actual database credentials.
*   `http://localhost:3000`: Replace with actual local development URLs.

### 3.1. Mobile Application (React Native)

| Variable Name | Description | Placeholder Value |
| :------------ | :---------- | :---------------- |
| `API_BASE_URL` | Base URL for the Backend API Gateway. | `https://api.yourdomain.com/v1` (prod) <br> `http://localhost:3000` (dev) |
| `AWS_COGNITO_USER_POOL_ID` | User Pool ID for AWS Cognito. | `us-east-1_abcdefgh` |
| `AWS_COGNITO_CLIENT_ID` | Client ID for the Mobile App in Cognito. | `1234567890abcdef` |
| `STRIPE_PUBLISHABLE_KEY` | Public key for Stripe payments. | `pk_test_xxxxxxxxxxxxxxxxxxxxxx` |
| `S3_PUBLIC_BUCKET_URL` | URL for publicly accessible static assets. | `https://s3.us-east-1.amazonaws.com/your-app-static-assets` |
| `WEB_APP_URL` | (For "Forgot Password" links, etc.) | `https://web.yourdomain.com` |

### 3.2. Backend Services (AWS Lambda via Serverless Framework)

These will be configured in `serverless.yml` or managed via AWS SSM Parameter Store in production.

| Variable Name | Description | Placeholder Value |
| :------------ | :---------- | :---------------- |
| `AWS_REGION` | The AWS region where services are deployed. | `us-east-1` |
| `DB_HOST` | Database endpoint. Use `localhost` for local development. | `your-rds-endpoint.us-east-1.rds.amazonaws.com` (prod) <br> `localhost` (dev) |
| `DB_PORT` | Database port. | `5432` |
| `DB_USERNAME` | Database master username. | `admin` |
| `DB_PASSWORD` | Database master password. | `YOUR_DB_PASSWORD` |
| `DB_NAME` | Database name for users and content. | `diagnostic_app_db` |
| `COGNITO_USER_POOL_ID` | User Pool ID for AWS Cognito (backend). | `us-east-1_abcdefgh` |
| `STRIPE_SECRET_KEY` | Secret key for Stripe payments. | `sk_test_xxxxxxxxxxxxxxxxxxxxxx` |
| `S3_FILE_UPLOAD_BUCKET` | S3 bucket name for user-submitted files. | `your-app-user-files` |
| `S3_STATIC_ASSET_BUCKET` | S3 bucket name for static assets (e.g., images for content). | `your-app-static-assets` |
| `NOTIFICATION_SENDER_EMAIL` | Email address for sending notifications (e.g., via SES). | `no-reply@yourdomain.com` |
| `JWT_SECRET` | Secret for signing/verifying JWTs (if custom tokens are used). | `YOUR_VERY_STRONG_JWT_SECRET` |

---

## 4. Component Setup Instructions

This section details the setup for each core and recommended component. We'll prioritize core components and the mobile frontend as per project decisions.

### 4.1. User Database & Content Database (Core)

**Technology**: AWS RDS (PostgreSQL)

Both user data and content data will reside in the same PostgreSQL database instance for simplicity and speed to build, as consistency is paramount.

1.  **Local Setup (Docker)**:
    *   Create a `docker-compose.yml` file in your backend repository root:
        ```yaml
        version: '3.8'
        services:
          postgres:
            image: postgres:15
            container_name: diagnostic-db
            environment:
              POSTGRES_DB: diagnostic_app_db
              POSTGRES_USER: admin
              POSTGRES_PASSWORD: YOUR_DB_PASSWORD
            ports:
              - "5432:5432"
            volumes:
              - db_data:/var/lib/postgresql/data
        volumes:
          db_data:
        ```
    *   Start the database: `docker-compose up -d`
    *   Verify by connecting with a client (e.g., `psql -h localhost -p 5432 -U admin -d diagnostic_app_db`).
2.  **AWS Setup (Production-Ready)**:
    *   Using the AWS Console or AWS CLI, provision an AWS RDS PostgreSQL instance.
    *   Configure security groups to allow inbound connections from your AWS Lambda functions and optionally your local IP for development.
    *   Ensure backups, multi-AZ deployment (for production), and monitoring are enabled.
    *   **TODO**: Define initial schema migration scripts (e.g., using `knex.js`, `TypeORM migrations`, or `Prisma migrate`). These scripts will define tables for users, diagnostic sessions, content items, questions, answers, purchases, etc.

### 4.2. Backend API Gateway & Serverless Services (Core)

**Technology**: AWS API Gateway, AWS Lambda (Node.js/TypeScript), Serverless Framework

This will host our Authentication, Diagnostic Engine, and Payment Processing services.

1.  **Project Structure**:
    *   Create a new directory (e.g., `backend/`).
    *   Inside, initialize a Serverless project: `serverless create --template aws-nodejs-typescript --path services`
    *   Organize services logically (e.g., `backend/services/auth`, `backend/services/diagnostic`, `backend/services/payment`).
2.  **Shared Utilities**:
    *   **Database Client**: Implement a shared database connection utility (e.g., `pg` client or `Knex.js`) that reuses connections for efficiency in Lambda.
    *   **Error Handling**: Centralized error handling and logging (e.g., using a custom error class and structured logging with AWS CloudWatch).
    *   **Authentication Middleware**: A Lambda Authorizer or middleware to validate JWTs from Cognito.
3.  **Authentication Service**:
    *   **Technology**: AWS Cognito User Pools for user management, AWS Lambda functions for custom logic (e.g., pre/post-signup hooks, custom token generation).
    *   **Setup**:
        *   Create a new Serverless service `backend/services/auth`.
        *   Define Lambda functions for: `signup`, `login`, `refreshTokens`, `forgotPassword`, `updateProfile`.
        *   Integrate with AWS Cognito via the AWS SDK.
        *   **TODO**: Implement user profile storage in the User Database upon signup.
        *   **TODO**: Define Cognito User Pool resources in `serverless.yml`.
4.  **Diagnostic Engine Service**:
    *   **Technology**: AWS Lambda functions, interacting with Content Database.
    *   **Setup**:
        *   Create a new Serverless service `backend/services/diagnostic`.
        *   Define Lambda functions for: `startDiagnosticSession`, `submitAnswer`, `getDiagnosticResult`, `getDiagnosticHistory`.
        *   Implement the core diagnostic logic within these functions, querying the Content Database.
        *   **TODO**: Define the algorithm for processing diagnostic queries and generating results.
5.  **Payment Processing Service**:
    *   **Technology**: Stripe (or similar) SDK, AWS Lambda functions.
    *   **Setup**:
        *   Create a new Serverless service `backend/services/payment`.
        *   Define Lambda functions for: `createPaymentIntent`, `processSubscription`, `webhookHandler` (for Stripe events).
        *   Integrate with Stripe API for secure payment processing.
        *   **TODO**: Store purchase records in the User Database.

### 4.3. Mobile Application (Core)

**Technology**: React Native with TypeScript

1.  **Project Initialization**:
    *   `react-native init MobileApp --template react-native-template-typescript`
    *   Navigate into `MobileApp` directory.
2.  **Dependencies**:
    *   Install navigation library (e.g., `react-navigation`).
    *   Install state management (e.g., `react-query` or `redux-toolkit`).
    *   Install authentication library (e.g., `aws-amplify` for Cognito integration).
    *   Install payment library (e.g., `stripe-react-native`).
    *   **TODO**: Add necessary UI component libraries (e.g., `react-native-elements`, `react-native-paper`).
3.  **Authentication Flow**:
    *   Implement signup, login, password reset, and profile management screens.
    *   Integrate with the Backend API Gateway's authentication endpoints and AWS Cognito.
    *   Upon successful login, store user tokens securely (e.g., `react-native-keychain`).
4.  **Diagnostic Flow**:
    *   Develop screens for starting a diagnostic, displaying questions, capturing user input, and presenting results.
    *   Communicate with the Backend API Gateway's diagnostic endpoints.
5.  **Payment/Subscription Flow**:
    *   Implement screens for viewing subscription options and processing payments.
    *   Integrate with the Backend API Gateway's payment endpoints and `stripe-react-native`.
6.  **Navigation**: Set up a robust navigation structure (e.g., authenticated vs. unauthenticated flows).
7.  **Error Handling**: Implement global error boundaries and user-friendly error messages.

### 4.4. Content Management System (Recommended)

**Technology**: Headless CMS (e.g., Strapi, Contentful) or a lightweight Next.js/React admin app.

Given simplicity and speed, a simple headless CMS (like Strapi) deployed on a small EC2 instance or as a serverless app could work, or a dedicated admin web app built with Next.js. For initial v1, we can manually insert content or use seed scripts.

1.  **Option 1: Seed Scripts (Initial MVP)**
    *   **TODO**: Create Node.js scripts that insert diagnostic questions, content, and algorithms directly into the Content Database.
2.  **Option 2: Headless CMS (Recommended for Production)**
    *   Choose a headless CMS (e.g., Strapi, Contentful, DatoCMS).
    *   **TODO**: Define content models matching the schema of your Content Database (e.g., `DiagnosticTopic`, `Question`, `AnswerOption`, `ResultAlgorithm`).
    *   Configure API access for the Diagnostic Engine Service to retrieve content.
    *   If self-hosting (like Strapi), deploy to a dedicated EC2 instance or a container service (e.g., Fargate) and secure its access.

### 4.5. Web Application (Recommended - for later)

**Technology**: Next.js with TypeScript

This will largely mirror the mobile application's logic but adapted for a web interface.

1.  **Project Initialization**:
    *   `npx create-next-app@latest web-app --typescript --eslint --app --tailwind --src-dir`
2.  **Shared Logic**: Reuse API client logic, authentication hooks, and potentially some UI components where feasible (e.g., design system).
3.  **Key Features**: Authentication, diagnostic flow, payment flow.
4.  **SSR/SSG**: Leverage Next.js's capabilities for SEO-friendly content and faster initial loads for public pages (e.g., marketing, static diagnostic info).

### 4.6. Analytics Service (Recommended)

**Technology**: AWS Kinesis, Lambda, S3 or a third-party service (e.g., Mixpanel, Amplitude).

For a production v1, basic analytics are crucial.

1.  **Client-Side Tracking (Mobile/Web)**:
    *   Integrate a simple analytics SDK (e.g., AWS Amplify Analytics, Mixpanel SDK) into the Mobile Application and Web Application.
    *   **TODO**: Define key events to track (e.g., `App_Launch`, `Signup_Success`, `Diagnostic_Started`, `Question_Answered`, `Diagnostic_Completed`, `Purchase_Initiated`, `Purchase_Success`).
2.  **Backend Event Capture**:
    *   For critical backend events (e.g., `Payment_Processed`, `User_Data_Updated`), publish events to AWS Kinesis Data Firehose (for easy S3 export) or CloudWatch Logs for analysis.
    *   **TODO**: Implement a Lambda function to process Kinesis streams and enrich/store data in S3 (for data warehousing) or push to a third-party analytics provider.

### 4.7. Notification Service (Recommended)

**Technology**: AWS SNS (Push, SMS), AWS SES (Email), AWS Lambda.

1.  **Email Notifications**:
    *   Configure AWS SES for sending transactional emails (e.g., welcome, password reset, purchase confirmation).
    *   **TODO**: Create Lambda functions that trigger SES emails based on events (e.g., signup completion, payment success).
2.  **Push Notifications (Mobile)**:
    *   Integrate AWS SNS with APNS (iOS) and FCM (Android).
    *   **TODO**: Store device tokens in the User Database during mobile app signup/login.
    *   **TODO**: Create Lambda functions that publish messages to SNS topics/endpoints to send targeted push notifications.

### 4.8. File Storage Service & Static Asset Storage (Recommended)

**Technology**: AWS S3

1.  **Bucket Creation**:
    *   Create two S3 buckets:
        *   `your-app-user-files`: For user-submitted content (e.g., images for diagnostic context). Keep private, use pre-signed URLs for uploads/downloads.
        *   `your-app-static-assets`: For public assets like diagnostic content images, frontend build artifacts. Configure for public read access.
2.  **User File Uploads**:
    *   **TODO**: Implement a Lambda function that generates pre-signed S3 URLs for the mobile app to securely upload files directly to `your-app-user-files`.
    *   **TODO**: Implement a Lambda function that generates pre-signed S3 URLs for the mobile app to securely download private user files.
3.  **Static Asset Hosting**:
    *   Host images, videos, and other media used in diagnostic content in `your-app-static-assets`.
    *   Reference these assets in the Content Database with their S3 URLs.

### 4.9. Content Delivery Network (Recommended)

**Technology**: AWS CloudFront

1.  **CloudFront Distribution**:
    *   Create a CloudFront distribution that points to your `your-app-static-assets` S3 bucket as an origin.
    *   Configure caching policies to optimize delivery of static content.
    *   **TODO**: Configure another CloudFront distribution to cache API Gateway responses (if applicable and beneficial for performance).

---

## 5. How to Run the Project Locally

Getting the full serverless stack running locally can be challenging. We'll focus on mocking/localizing key components.

1.  **Start Local Database**:
    *   Ensure Docker is running.
    *   Navigate to your `backend/` directory (or wherever your `docker-compose.yml` is).
    *   `docker-compose up -d`
2.  **Run Backend Serverless Services (Local Emulation/Mocking)**:
    *   For each serverless service (e.g., `backend/services/auth`, `backend/services/diagnostic`, `backend/services/payment`):
        *   Navigate into the service directory.
        *   Install dependencies: `npm install` (or `yarn`).
        *   Use `serverless-offline` plugin to emulate Lambda and API Gateway locally.
            *   Install: `npm install --save-dev serverless-offline`
            *   Add to `serverless.yml` plugins section.
            *   Run: `serverless offline start`
        *   This will expose your Lambda functions as HTTP endpoints, typically starting at `http://localhost:3000` (or another configured port).
        *   **Important**: You might need to run multiple `serverless offline` instances on different ports if services are entirely separate. For simplicity, consider running *one* `serverless offline` instance that exposes *all* your backend functions for local dev.
3.  **Run Mobile Application**:
    *   Navigate to your `MobileApp/` directory.
    *   Install dependencies: `npm install` (or `yarn`).
    *   Ensure your local backend's API Gateway URL (`API_BASE_URL`) is correctly set in your mobile app's `.env` file (e.g., `http://localhost:3000`).
    *   Start the metro bundler and emulator/simulator:
        *   For iOS: `npm run ios`
        *   For Android: `npm run android`
    *   **Note**: For Android emulator to access `localhost` on your machine, use `http://10.0.2.2:PORT` instead of `http://localhost:PORT`.

---

## 6. Smoke Tests

After setting up and running the project locally or deploying to a development environment, perform these smoke tests to ensure core functionality is working.

1.  **Mobile Application - User Authentication Flow**:
    *   **Signup**:
        *   Open the mobile app.
        *   Navigate to the signup screen.
        *   Enter new user details (email, password).
        *   Verify successful account creation and auto-login or prompt for email verification (if configured).
        *   Check the User Database for the new user entry.
    *   **Login**:
        *   Log out if signed in.
        *   Navigate to the login screen.
        *   Enter credentials for an existing user.
        *   Verify successful login and access to the authenticated part of the app.
    *   **Forgot Password**:
        *   Initiate the "Forgot Password" flow.
        *   Enter a registered email address.
        *   Verify that a password reset email (or code) is sent (check email client or SES logs).
        *   Attempt to reset password using the received code/link.
2.  **Mobile Application - Diagnostic Flow**:
    *   **Start Diagnostic**:
        *   From the main screen, start a new diagnostic session.
        *   Verify that the first question or content item loads correctly.
    *   **Answer Questions**:
        *   Answer a few questions in sequence.
        *   Verify that answers are submitted and the next question loads.
    *   **Complete Diagnostic & Get Result**:
        *   Complete a full diagnostic path.
        *   Verify that a diagnostic result is displayed.
        *   Check the User Database for the saved diagnostic history.
3.  **Mobile Application - Payment/Subscription Flow**:
    *   **View Subscriptions**:
        *   Navigate to the subscription/premium features screen.
        *   Verify that available subscription plans are displayed.
    *   **Initiate Purchase (Test Mode)**:
        *   Select a subscription plan.
        *   Proceed to the payment screen.
        *   Enter Stripe test card details.
        *   Verify successful payment processing (UI confirmation).
        *   Check the User Database for the purchase record and updated user subscription status.
    *   **Access Premium Content**:
        *   After a successful test purchase, attempt to access a premium diagnostic feature.
        *   Verify that access is granted.
4.  **Backend API Gateway & Services**:
    *   Use a tool like Postman or Insomnia (or `curl`) to directly call a few backend endpoints:
        *   `POST /auth/signup` (with valid payload)
        *   `POST /auth/login` (with valid payload)
        *   `GET /diagnostic/questions` (with authentication token)
        *   Verify 2xx responses and expected data structures.

---

## 7. Common Troubleshooting Steps

Encountering issues is part of development. Here are some common problems and their solutions.

### 7.1. General Issues

*   **Environment Variables Not Loaded**:
    *   **Check**: Ensure `.env` files are correctly placed and named (e.g., `.env.development`, `.env.production`). For backend, check `serverless.yml` `environment` section or SSM parameters.
    *   **Solution**: Double-check variable names for typos. Restart processes after changing `.env` files.
*   **Dependencies Not Installed**:
    *   **Check**: Error messages like "command not found" or "module not found".
    *   **Solution**: Navigate to the specific component's directory (`MobileApp/`, `backend/services/auth/`, etc.) and run `npm install` (or `yarn`).
*   **Port Conflicts**:
    *   **Check**: `serverless offline` or `npm start` reporting "Address already in use".
    *   **Solution**: Identify the process using the port and kill it, or configure your application to use a different port (e.g., `serverless offline start --port 3001`).

### 7.2. Backend (Serverless) Issues

*   **Lambda Function Errors**:
    *   **Check**: Look at the console output of `serverless offline` for local errors. For deployed functions, check AWS CloudWatch Logs for the specific Lambda function.
    *   **Solution**: Analyze stack traces. Common causes include: missing environment variables, incorrect database connection strings, unhandled promises, or permission issues (`IAM roles`).
*   **Database Connection Failed**:
    *   **Check**: Error message "FATAL: password authentication failed" or "timeout".
    *   **Solution**:
        *   **Local**: Is Docker running? Is the container up? Are `DB_HOST`, `DB_PORT`, `DB_USERNAME`, `DB_PASSWORD` correct in your `.env` or Serverless configuration?
        *   **AWS RDS**: Check RDS instance status. Verify security group rules (inbound for port 5432 from Lambda's VPC/security group). Ensure `DB_HOST` is the correct RDS endpoint.
*   **API Gateway 500/502 Errors**:
    *   **Check**: This usually means the underlying Lambda function failed. Check CloudWatch Logs for the invoked Lambda.
    *   **Solution**: Debug the Lambda function as described above. Ensure API Gateway integration settings are correct.

### 7.3. Mobile Application Issues

*   **Metro Bundler Errors**:
    *   **Check**: Errors in the terminal where `npm run ios`/`android` was run.
    *   **Solution**: Clear cache (`npm start --reset-cache`). Ensure correct Node.js version. Reinstall `node_modules`.
*   **Network Requests Failing**:
    *   **Check**: Mobile app shows "Network Error" or requests time out.
    *   **Solution**:
        *   Verify `API_BASE_URL` in the mobile app's `.env` file.
        *   If using an Android emulator for local backend, remember to use `http://10.0.2.2:PORT` instead of `localhost`.
        *   Ensure your local backend (`serverless offline`) is actually running and accessible.
        *   For deployed backend, check if the domain is correct and there are no DNS issues.
*   **Build Errors (iOS/Android)**:
    *   **Check**: Errors during `npm run ios` or `npm run android` related to native modules.
    *   **Solution**:
        *   `cd ios && pod install` (for iOS).
        *   Clean Android build cache: `cd android && ./gradlew clean`.
        *   Follow React Native's troubleshooting for specific errors, often involves re-linking libraries or checking Xcode/Android Studio setup.

---