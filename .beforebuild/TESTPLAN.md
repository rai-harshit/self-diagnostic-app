# TESTPLAN.md for Self-Diagnostic App v1

## 1. Introduction

This document outlines the test plan for the initial production-ready version (v1) of the Self-Diagnostic Mobile Application. The primary goal for this phase is to ensure a robust and reliable user experience, particularly focusing on the core transactional flow: user signup, purchase of premium features, and successful completion of a diagnostic submission.

Given the priority for simplicity and speed to build, this plan focuses on practical, actionable steps for verifying critical functionality and integrations. While "production-ready" implies thoroughness, we will prioritize covering the main user journeys and essential system components over exhaustive edge cases in this initial phase. The mobile application is the primary platform in scope, with consideration for future web capabilities.

## 2. Testing Principles & Strategy

Our testing strategy for v1 is guided by the project's preferences:

*   **Mobile-First Focus**: All primary test efforts will target the native mobile application interface, ensuring its stability and usability.
*   **End-to-End Transactional Flow**: High priority will be given to testing the entire user journey from registration through to content access, diagnostic completion, and payment.
*   **Core Component Verification**: Each critical system component, from the frontend to backend services and databases, will be verified for its individual functionality and integration points.
*   **Data Integrity**: Due to the preference for a relational database with strong consistency, data storage and retrieval will be carefully checked to ensure accuracy and consistency of user profiles, diagnostic history, and content.
*   **Error Handling**: Basic error scenarios for common user actions (e.g., invalid login, failed payment) will be included to ensure the application provides appropriate feedback and graceful degradation.
*   **Simplicity and Speed**: Test cases will be practical, direct, and focused on verifying core functionality rather than overly complex or niche scenarios, aligning with the "speed to build" priority.
*   **Security (Basic)**: Verification that basic authentication and authorization mechanisms are functioning as expected.

## 3. Suggested Testing Tools & Frameworks

To align with the specified technology stack and development preferences, the following tools and frameworks are recommended for testing:

*   **Frontend (Mobile App - React Native / similar React-based mobile framework)**
    *   **Unit/Integration Testing**: `Jest` for JavaScript/TypeScript testing, `React Testing Library` for testing React components (behavior and accessibility).
    *   **End-to-End Testing**: `Appium` (for native mobile apps) or `Detox` (if using React Native) for simulating real user interactions on devices/emulators.
*   **Backend (Serverless Functions - TypeScript/JavaScript)**
    *   **Unit/Integration Testing**: `Jest` for individual function logic, `supertest` for API integration testing if functions are exposed via HTTP endpoints directly.
    *   **API Testing (Manual/Automated)**: `Postman` or `Insomnia` for manual API endpoint testing, `Newman` (Postman's CLI companion) for automating API tests in CI/CD pipelines.
*   **Database (PostgreSQL/MySQL)**
    *   **Data Validation**: SQL queries for direct database inspection (e.g., verifying user data, purchase records, diagnostic results). Integration tests with ORM/database client.
*   **General/Performance/Security**
    *   **Load Testing (Basic)**: `k6` or `JMeter` for light load testing on critical API endpoints to ensure basic scalability for v1.
    *   **Security Testing (Basic)**: Manual penetration testing, using tools like `OWASP ZAP` for automated vulnerability scanning on exposed endpoints.

## 4. Smoke Test Cases

The following smoke tests cover the critical user journeys and component integrations for the Self-Diagnostic App v1. They are organized by feature area for clarity.

### 4.1. User Onboarding & Authentication

1.  **User Registration (Happy Path)**
    *   **Description**: A new user successfully registers for an account via the mobile application.
    *   **Components Verified**: Mobile Application, Backend API Gateway, Authentication Service, User Database.
    *   **Expected Result**: User account is created, user is logged in, and their profile details (e.g., email) are correctly stored in the User Database. A welcome notification is sent.

2.  **User Login (Happy Path)**
    *   **Description**: An existing user successfully logs into the mobile application using valid credentials.
    *   **Components Verified**: Mobile Application, Backend API Gateway, Authentication Service, User Database.
    *   **Expected Result**: User is successfully authenticated and can access the application's main features.

3.  **User Registration (Error Path - Duplicate Email)**
    *   **Description**: A user attempts to register with an email address already associated with an existing account.
    *   **Components Verified**: Mobile Application, Backend API Gateway, Authentication Service.
    *   **Expected Result**: The mobile application displays an appropriate error message (e.g., "Email already registered"), and no new account is created.

4.  **User Login (Error Path - Invalid Credentials)**
    *   **Description**: A user attempts to log in with incorrect email or password.
    *   **Components Verified**: Mobile Application, Backend API Gateway, Authentication Service.
    *   **Expected Result**: The mobile application displays an appropriate error message (e.g., "Invalid credentials"), and the user remains unauthenticated.

5.  **User Logout**
    *   **Description**: A logged-in user successfully logs out of the mobile application.
    *   **Components Verified**: Mobile Application, Backend API Gateway, Authentication Service.
    *   **Expected Result**: User session is terminated, and the user is redirected to the login/registration screen.

### 4.2. Diagnostic Content & Engine

6.  **Content Retrieval & Display**
    *   **Description**: The mobile application successfully fetches and displays the initial diagnostic questions/categories.
    *   **Components Verified**: Mobile Application, Backend API Gateway, Diagnostic Engine Service, Content Database, Content Delivery Network, Static Asset Storage.
    *   **Expected Result**: Relevant diagnostic content (text, images, media) is loaded quickly and displayed correctly on the mobile device.

7.  **Content Management System (Basic)**
    *   **Description**: A new piece of diagnostic content is published via the Content Management System.
    *   **Components Verified**: Content Management System, Content Database.
    *   **Expected Result**: The new content is successfully saved in the Content Database and is available for retrieval by the Diagnostic Engine.

### 4.3. Diagnostic Session Flow

8.  **Complete Diagnostic Session (Happy Path)**
    *   **Description**: A user starts a diagnostic, answers all required questions, and successfully receives a diagnostic result.
    *   **Components Verified**: Mobile Application, Backend API Gateway, Diagnostic Engine Service, User Database, Content Database.
    *   **Expected Result**: A clear diagnostic result is presented to the user, and their diagnostic history is accurately saved in the User Database.

9.  **Diagnostic Session Progress Saving**
    *   **Description**: A user starts a diagnostic, answers some questions, and then closes/leaves the application, then resumes.
    *   **Components Verified**: Mobile Application, Backend API Gateway, Diagnostic Engine Service, User Database.
    *   **Expected Result**: Upon returning, the user can resume the diagnostic from where they left off, and previous answers are preserved.

### 4.4. Payment & Subscriptions

10. **Successful Premium Content Purchase**
    *   **Description**: A user successfully completes a purchase for premium diagnostic content or a subscription.
    *   **Components Verified**: Mobile Application, Backend API Gateway, Payment Processing Service, User Database, Notification Service.
    *   **Expected Result**: The payment is processed successfully, the user gains access to the purchased content, their purchase record is updated in the User Database, and a payment confirmation notification is sent.

11. **Payment Failure Handling**
    *   **Description**: A user attempts a purchase with invalid payment details (e.g., expired card).
    *   **Components Verified**: Mobile Application, Backend API Gateway, Payment Processing Service.
    *   **Expected Result**: The mobile application displays an appropriate error message from the Payment Processing Service, and no charge is made.

12. **View Purchase History**
    *   **Description**: A user navigates to their profile to view their past purchases or active subscriptions.
    *   **Components Verified**: Mobile Application, Backend API Gateway, User Database.
    *   **Expected Result**: The user's past purchases and current subscription status are accurately displayed.

### 4.5. Notifications & Analytics

13. **Welcome/Confirmation Notification Delivery**
    *   **Description**: Verify that welcome emails (upon registration) or purchase confirmation notifications are sent.
    *   **Components Verified**: Authentication Service/Payment Processing Service -> Notification Service.
    *   **Expected Result**: The user receives the expected notification (e.g., email, push notification) in their inbox/device.

14. **Basic Analytics Data Collection**
    *   **Description**: A user performs key actions (e.g., login, complete diagnostic, make purchase).
    *   **Components Verified**: Mobile Application -> Analytics Service.
    *   **Expected Result**: Basic user interaction data (e.g., "Login Event", "Diagnostic Completed Event", "Purchase Successful Event") is successfully sent to the Analytics Service for processing.

### 4.6. System Health & Infrastructure

15. **API Gateway Accessibility**
    *   **Description**: Make a direct request to a non-authenticated public API endpoint (e.g., health check endpoint).
    *   **Components Verified**: Backend API Gateway.
    *   **Expected Result**: The endpoint responds with a successful HTTP status code (200 OK) and appropriate payload.

16. **Static Asset Loading**
    *   **Description**: The mobile app successfully loads all necessary static assets (images, icons, fonts).
    *   **Components Verified**: Mobile Application, Static Asset Storage, Content Delivery Network.
    *   **Expected Result**: All static assets are loaded without errors, and the UI displays correctly.