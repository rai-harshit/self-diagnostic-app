# LIMITS.md - Self Diagnostic App (V1)

This document outlines the current limitations, architectural constraints, and explicit scope boundaries for the initial build of the Self Diagnostic App. As we embark on building a production-ready V1, understanding these limits is crucial for effective planning, development, and risk management.

## Project Decisions

The following decisions were made during project planning, which heavily influence the scope and architecture detailed below:

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

## Scope Decisions (V1)

Based on the project decisions, particularly the goal of a "production-ready v1," "mobile app first," and "simplicity and speed to build" with a "full serverless" backend, the following scope boundaries have been set:

*   **Primary Goal: Production-Ready V1:** This mandates a strong focus on core reliability, security hardening, robust error handling, and basic monitoring for the features that *are* in scope. This is not a throw-away prototype, but the foundation for future iterations.
*   **Main Outcome: Transactional Flow Completion:** The initial build prioritizes the critical path: user signup, successful purchase of a diagnostic, and submission of diagnostic input to receive results. All components must support this core flow reliably.
*   **Mobile App First (React Native Implied):** The primary focus is on delivering a native-feeling mobile experience. While the stated frontend stack is React/Next.js, for "mobile app first," we will leverage React Native to align with the React ecosystem preference and enable future code sharing with the web application. The Web Application is explicitly deferred.
*   **Full Serverless Backend (Functions as a Service, BaaS):** This choice prioritizes rapid development and reduced operational overhead. It directly influences architectural patterns, scaling behavior, and certain limitations.
*   **Relational DB (PostgreSQL/MySQL, Strong Consistency):** User data, diagnostic history, and transactional records will leverage a relational database for its strong consistency guarantees, which are vital for user accounts and payment processing.
*   **Architecture Priority: Simplicity and Speed to Build:** We will deliberately avoid over-engineering or introducing unnecessary layers of abstraction that could slow down initial development. Solutions will be pragmatic and direct, even if this means less flexibility in highly specific niche scenarios that are out of scope for V1.

## What is NOT Production-Ready Yet (Initial Build State)

As this project is new and not yet built, *nothing* is currently production-ready. The initial development phase will focus on implementing the core features. Before launching to a consumer/general audience as a "production-ready v1", the following aspects will require dedicated effort:

*   **Comprehensive Error Handling & Observability:** While basic logging will be implemented, a unified error reporting system, detailed traces, and robust operational dashboards will need to be developed and integrated.
*   **Full Security Hardening:** Beyond initial security measures (like TLS and basic authentication), extensive penetration testing, vulnerability scanning, and hardening of all infrastructure and application layers must occur.
*   **Performance Optimization & Load Testing:** Initial performance will be monitored, but dedicated load testing and optimization passes will be required to ensure the system scales efficiently under real-world consumer traffic.
*   **Disaster Recovery & Backup Strategies:** Formalized backup procedures, restoration plans, and testing of disaster recovery mechanisms will be put in place.
*   **Automated Deployment & Rollback:** Robust CI/CD pipelines supporting automated, reliable, and reversible deployments across environments will be essential.
*   **User Acceptance Testing (UAT):** Thorough UAT with target users will be conducted to validate the entire transactional flow and user experience.
*   **Cost Optimization:** While serverless offers elasticity, ongoing monitoring and optimization of cloud costs will be necessary post-launch.

## Known Architectural Limitations

The chosen architectural approach (full serverless, relational DB, simplicity-first) introduces specific limitations for V1:

1.  **Serverless Cold Starts:** Functions that are not frequently invoked may experience "cold starts," leading to increased latency for the *first* request. While this often has minimal impact on frequently used APIs, it can affect the initial user experience for less common features.
2.  **Database Connection Management with Serverless:** Managing a large pool of concurrent serverless functions hitting a relational database requires careful attention to connection pooling. Without proper management, the database can become a bottleneck due to connection exhaustion, even with robust auto-scaling of compute.
3.  **Vendor Lock-in:** Opting for a full serverless (FaaS, BaaS) strategy inherently ties the project to a specific cloud provider's ecosystem (e.g., AWS Lambda/DynamoDB, GCP Cloud Functions/Firestore, Azure Functions/Cosmos DB). While this accelerates development, migrating services to another provider would be a significant effort.
4.  **Complexity of Distributed Observability:** While simpler to deploy, serverless architectures are distributed. Tracing requests across multiple functions, API Gateway, and databases can be more complex than monitoring a monolithic application.
5.  **Relational Database Scaling Limits:** While relational databases offer strong consistency and can scale vertically well, eventually, horizontal scaling (sharding, extensive read replicas) becomes necessary for extreme loads. For V1, we will rely on vertical scaling and read replicas where appropriate, which will have a practical limit.
6.  **Potential for Unforeseen Costs:** While serverless is often cost-effective at scale, unpredictable usage patterns or inefficient function design can lead to higher-than-expected costs due to granular billing.

## Performance Considerations and Potential Bottlenecks

1.  **API Latency (Cold Starts):** As noted above, cold starts can introduce noticeable latency for infrequent API calls, especially if the diagnostic engine is invoked after a period of inactivity.
2.  **Diagnostic Engine Processing Time:** The complexity of the diagnostic logic and the volume of content processed will directly impact the response time of the Diagnostic Engine Service. Intensive computation could lead to longer function execution times and increased costs.
3.  **Database Throughput and Latency:** The User Database and Content Database are central. Unoptimized queries, high transaction volumes, or insufficient database provisioning can lead to read/write bottlenecks and overall system slowdowns.
4.  **Network Latency:** For a consumer-facing mobile app, varying network conditions (mobile data, Wi-Fi) and geographical distance to the cloud region will impact perceived performance. The CDN will mitigate this for static assets, but API calls are still subject to network conditions.
5.  **Payment Processing Integration:** Integration with external payment gateways introduces their own potential latency and points of failure, which are outside our direct control.
6.  **Content Delivery Network (CDN) Configuration:** While a CDN is planned, incorrect caching policies or misconfigurations could lead to stale content or unnecessary origin hits, impacting performance.

## Security Gaps to Address Before Production

While V1 aims to be production-ready, several security aspects require rigorous implementation and validation:

1.  **Robust Authentication & Authorization (AuthN/AuthZ):** The Authentication Service must be thoroughly vetted for strong password policies, secure session management, token validation, and granular authorization rules to ensure users can only access their own data.
2.  **Comprehensive Input Validation & Sanitization:** All user inputs (mobile app, CMS) must be rigorously validated and sanitized at both the client and server (Backend API Gateway, Diagnostic Engine, etc.) to prevent common vulnerabilities like injection attacks (SQL, XSS).
3.  **Data Encryption at Rest and in Transit:** All sensitive data (user details, payment info) must be encrypted at rest in databases and file storage, and all communication must use TLS/SSL.
4.  **Secrets Management:** API keys, database credentials, and other sensitive configurations must be stored and accessed securely using dedicated secrets management services, not hardcoded.
5.  **API Rate Limiting & Throttling:** Implementing rate limiting at the API Gateway is crucial to protect against denial-of-service (DoS) attacks and abusive usage patterns.
6.  **Vulnerability Scanning & Penetration Testing:** Before launch, independent security audits, automated vulnerability scanning, and manual penetration testing must be performed across the entire application and infrastructure.
7.  **Least Privilege Principle:** All serverless functions, services, and users (especially CMS users) must operate with the minimum necessary permissions required for their tasks.
8.  **Secure Content Management System (CMS):** The CMS needs robust authentication, authorization, audit logging, and strict data validation to prevent the introduction of malicious or malformed content.

## Features Explicitly Out of Scope for Initial Build (V1)

To maintain focus on "simplicity and speed to build" and the "mobile app first" directive, the following features are explicitly deferred:

*   **Web Application:** The web-based interface will be developed *after* the mobile application has reached a stable V1.
*   **Advanced Analytics Dashboards/Reporting:** While the Analytics Service will collect data, comprehensive, real-time dashboards or complex reporting features for administrators are out of scope for V1. Basic data access for analysis will be sufficient.
*   **Multi-Region Deployment/Global High Availability:** V1 will be deployed within a single primary cloud region. Achieving multi-region fault tolerance and global low-latency access is a future enhancement.
*   **Offline Mode for Mobile App:** The mobile application will require an active internet connection for all core functionalities. Complex offline data synchronization and conflict resolution are out of scope.
*   **User-Generated Diagnostic Content:** All diagnostic content, questions, and algorithms will be created and managed by content creators via the CMS. Users cannot submit or contribute to the diagnostic content itself.
*   **Extensive Internationalization/Localization:** The application will initially support a single primary language and locale. Broad multi-language support is a future consideration.
*   **Sophisticated AI/ML in Diagnostic Engine:** The Diagnostic Engine will rely on predefined logic and algorithms (e.g., rule-based systems, decision trees) for V1, not advanced machine learning models for diagnosis or recommendations.
*   **Complex Administrative Portal (beyond CMS):** Administrative functionalities will be minimal, primarily focusing on content management via the CMS. A dedicated, feature-rich admin portal for user management, system health, etc., is not planned for V1.

## Tested Environments and Platforms

For V1, testing will focus on the following environments and platforms:

*   **Mobile Application (React Native):**
    *   **iOS:** Latest 2 major iOS versions on iPhone form factors (e.g., iOS 16, iOS 17).
    *   **Android:** Latest 2 major Android versions on a selection of top-market devices (e.g., Android 13, Android 14) and emulators/simulators.
    *   **Form Factors:** Primarily phone devices. Tablet optimization is out of scope for V1.
*   **Backend Services:**
    *   **Cloud Provider:** Development, Staging (for UAT and integration testing), and Production environments, all hosted within the chosen serverless cloud platform (e.g., AWS, GCP, Azure).
*   **Content Management System (CMS):**
    *   Modern web browsers (latest 2 versions of Chrome, Firefox, Safari, Edge).

## Scaling Limitations

While serverless architectures are inherently scalable, certain components and architectural decisions for V1 introduce specific scaling limits:

*   **Relational Database Capacity:** While PostgreSQL/MySQL can scale vertically (larger instances) and horizontally for reads (read replicas), there will be a practical limit before sharding (splitting data across multiple database instances) becomes necessary for extremely high write loads or massive datasets. This is the most likely long-term scaling bottleneck for transactional data.
*   **Database Connection Pool Management:** The chosen approach for managing database connections from a potentially vast number of concurrent serverless functions will be critical. Inefficient connection pooling can lead to database connection exhaustion, even if the database itself has capacity.
*   **Serverless Concurrent Execution Limits:** Cloud providers impose limits on the maximum number of concurrent executions for serverless functions per region. While these limits are often high, extreme and sudden traffic spikes could theoretically hit these caps, requiring a request for limit increases.
*   **Content Database Read/Write Throughput:** If the Content Database faces exceptionally high read or write contention, especially with complex queries from the Diagnostic Engine, it could become a bottleneck.
*   **External Service Dependencies:** Scaling is also limited by the throughput and reliability of third-party services like the Payment Processing Service. We have limited control over their scaling capabilities.