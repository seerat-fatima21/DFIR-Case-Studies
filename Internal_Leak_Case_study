# Incident Response: Internal Data Leak Case Study

## Overview
A simulated incident where a sales representative accidentally shared an internal-only link with a business partner, leading to sensitive product details and customer analytics being posted publicly on social media.

This project documents my analysis and response to the incident, following core DFIR (Digital Forensics & Incident Response) principles.

## Incident Summary

*   **Status:** Resolved
*   **Severity:** High
*   **Primary Failure:** Misconfiguration of sharing permissions violating the **Principle of Least Privilege**.
*   **Frameworks Applied:** NIST CSF, NIST SP 800-53 (AC-6)

## Analysis (The 5 W's)

*   **What** An internal folder link containing unannounced product data was posted publicly.
*   **When** During a sales call; discovered via social media monitoring shortly after.
*   **Where** The link was posted on a business partner's social media page.
*   **Who** A sales representative and an external partner.
*   **Why**  The folder was shared with a link that provided excessive access (`Anyone with the link`).
   
## Recommendations & Remediation

Based on the **NIST Framework**, I recommend the following:

- **Reconfigure all sensitive shares:** Configure folders so that only Specific people or Internal only can access them, instead of using Anyone with the link.
- **Implement Role-Based Access Control (RBAC):** Ensure users only have access to data necessary for their job function.
- **Prevent External Sharing:** For highly sensitive folders, external sharing should be disabled entirely through policy.
- **Schedule access reviews:** Regularly check who has access to sensitive files to ensure only the appropriate personnel have permission.

## Conclusion

This incident highlights how a simple misconfiguration in access controls, combined with human error, can lead to a significant data leak. Adhering to the **Least Privilege** principle is not just a technical control but a critical cultural one that must be supported by clear policies, employee training, and enforcing technologies.

---
*Disclaimer: This is an analysis of a simulated incident for educational purposes.*
