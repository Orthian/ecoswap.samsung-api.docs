---
sidebar_position: 2
---
# Environments

EcoSwap provides a tiered environment structure that supports the entire lifecycle of your integration—from initial proof-of-concept work to final, production-level operations. By leveraging these environments, you can methodically test functionality, ensure system robustness, and maintain high standards of performance and security.

| Environment | Base URL                      | Primary Purpose                                     | Typical Use Cases                                              | Data & Configuration Characteristics                               |
|-------------|--------------------------------|-----------------------------------------------------|----------------------------------------------------------------|--------------------------------------------------------------------|
| Development | `https://dev.ecoswapvn.com`    | **Initial Integration & Iterative Testing**         | - Validating basic API connectivity<br/>- Testing JWT authentication and token refresh flows<br/>- Experimenting with encryption/decryption to ensure compatibility | Frequently updated configurations. Data is often mock or transient. Rapid iteration is expected. |
| Staging     | `https://stg.ecoswapvn.com`    | **Pre-Production & UAT**                           | - Conducting User Acceptance Testing (UAT) in a production-like setting<br/>- Verifying device grading logic end-to-end<br/>- Assessing error handling, fallback scenarios, and performance benchmarks | More stable configurations closer to production standards. Test data is representative of real-world conditions. Enhanced monitoring and logging provide insights into behavior under realistic workloads. |
| Production  | `https://admin.ecoswapvn.com`  | **Live Operations & Customer-Facing Deployment**    | - Handling actual Sales Orders and encrypted payloads<br/>- Maintaining continuous uptime and high performance<br/>- Ensuring secure processing of sensitive customer information | Fully hardened environment with stringent security, high reliability, and performance. Real customer data demands strict adherence to compliance, monitoring, and incident response protocols. |

### Recommended Lifecycle Progression

1. **Development Environment:**  
   Begin integration efforts here. Implement the initial connection logic, retrieve and refresh tokens, and test basic encryption/decryption routines. This environment is designed for rapid iteration and learning. Anticipate frequent changes in configurations and endpoints as you refine your integration.

2. **Staging Environment:**  
   Once core functionalities are validated in Development, transition to Staging for a more rigorous examination. Conduct UAT under realistic conditions to confirm that your workflows—such as handling `SO_CREATE`, submitting price and grade updates, and processing `SO_RESULT` events—operate reliably. Utilize test datasets that mimic production scenarios. Performance and error-handling testing at this stage ensures smoother production rollouts.

3. **Production Environment:**  
   Move to Production only after successfully passing UAT in Staging. In Production, your integration interacts with genuine customer data and live Sales Orders. At this stage, focus on maintaining high service availability, adherence to security protocols, and accurate processing of trade-in devices. Implement robust monitoring, alerting, and logging to detect and resolve issues swiftly, minimizing impact on end users.

### Key Considerations

- **Security and Compliance:** Across all environments, protect sensitive data. In Production, enforce the strictest security measures, including access control, encrypted communication, and compliance with relevant regulations.
- **Monitoring and Observability:** Use Staging and Production environments to implement and refine monitoring dashboards, alerts, and logs. Observability is crucial for diagnosing issues quickly and ensuring a high-quality customer experience.
- **Change Management:** Test any configuration changes or updates in Development or Staging before applying them to Production. A disciplined change management practice reduces the risk of service disruptions.

By following this structured environment strategy, you can confidently progress from concept to high-quality production deployment, ensuring a stable, secure, and scalable integration with EcoSwap’s trade-in services.
