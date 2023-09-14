# OWASP Top 10
> Created: 2023-02-25 10:27

# A01:2021 – Broken Access Control

## Overview

Moving up from the fifth position, 94% of applications were tested for some form of broken access control with the average incidence rate of 3.81%, and has the most occurrences in the contributed dataset with over 318k. Notable Common Weakness Enumerations (CWEs) included are _CWE-200: Exposure of Sensitive Information to an Unauthorized Actor_, _CWE-201: Insertion of Sensitive Information Into Sent Data_, and _CWE-352: Cross-Site Request Forgery_.

## Description

Access control enforces policy such that users cannot act outside of their intended permissions. Failures typically lead to unauthorized information disclosure, modification, or destruction of all data or performing a business function outside the user's limits. Common access control vulnerabilities include:

+ Violation of the **principle of least privilege** or deny by default, where access should only be granted for particular capabilities, roles, or users, but is available to anyone.   
+ **Bypassing access control checks** by modifying the URL (parameter tampering or force browsing), internal application state, or the HTML page, or by using an attack tool modifying API requests.
+ Permitting viewing or editing someone else's account, by providing its unique identifier (**insecure direct object references**)
+ Accessing API with **missing access controls** for POST, PUT and DELETE.
+ **Elevation of privilege**. Acting as a user without being logged in or acting as an admin when logged in as a user.   
+ **Metadata manipulation**, such as replaying or tampering with a JSON Web Token (JWT) access control token, or a cookie or hidden field manipulated to elevate privileges or abusing JWT invalidation.
+ **CORS misconfiguration** allows API access from unauthorized/untrusted origins.
+ Force browsing to authenticated pages as an unauthenticated user or to privileged pages as a standard user.

## How to Prevent

Access control is only effective in **trusted server-side code** or server-less API, where the attacker cannot modify the access control check or metadata.

- Except for public resources, **deny by default**.
- Implement access control mechanisms once and **re-use them** throughout the application, including minimizing Cross-Origin Resource Sharing (CORS) usage.
- Model access controls should enforce **record ownership** rather than accepting that the user can create, read, update, or delete any record.
- Unique application **business limit requirements** should be enforced by domain models.
- Disable **web server directory listing** and ensure **file metadata** (e.g., .git) and backup files are not present within web roots.
- **Log access control failures**, alert admins when appropriate (e.g., repeated failures).
- **Rate limit** API and controller access to minimize the harm from automated attack tooling.
- **Stateful session identifiers** should be invalidated on the server after logout. Stateless JWT tokens should rather be **short-lived** so that the window of opportunity for an attacker is minimized. For longer lived JWTs it's highly recommended to follow the OAuth standards to **revoke access**.

Developers and QA staff should include functional access control unit and integration tests.

# A02:2021 – Cryptographic Failures

## Overview

Shifting up one position to #2, previously known as _Sensitive Data Exposure_, which is more of a broad symptom rather than a root cause, the focus is on failures related to cryptography (or lack thereof). Which often lead to exposure of sensitive data. Notable Common Weakness Enumerations (CWEs) included are _CWE-259: Use of Hard-coded Password_, _CWE-327: Broken or Risky Crypto Algorithm_, and _CWE-331 Insufficient Entropy_.

## Description

The first thing is to determine the protection needs of data in transit and at rest. For example, passwords, credit card numbers, health records, personal information, and business secrets require extra protection, mainly if that data falls under privacy laws, e.g., EU's General Data Protection Regulation (GDPR), or regulations, e.g., financial data protection such as PCI Data Security Standard (PCI DSS). For all such data:

-   Is any data transmitted in clear text? This concerns protocols such as HTTP, SMTP, FTP also using TLS upgrades like STARTTLS. External internet traffic is hazardous. Verify all internal traffic, e.g., between load balancers, web servers, or back-end systems.
-   Are any old or weak cryptographic algorithms or protocols used either by default or in older code?
-   Are default crypto keys in use, weak crypto keys generated or re-used, or is proper key management or rotation missing? Are crypto keys checked into source code repositories?
-   Is encryption not enforced, e.g., are any HTTP headers (browser) security directives or headers missing?
-   Is the received server certificate and the trust chain properly validated?
-   Are initialization vectors ignored, reused, or not generated sufficiently secure for the cryptographic mode of operation? Is an insecure mode of operation such as ECB in use? Is encryption used when authenticated encryption is more appropriate?
-   Are passwords being used as cryptographic keys in absence of a password base key derivation function?
-   Is randomness used for cryptographic purposes that was not designed to meet cryptographic requirements? Even if the correct function is chosen, does it need to be seeded by the developer, and if not, has the developer over-written the strong seeding functionality built into it with a seed that lacks sufficient entropy/unpredictability?
-   Are deprecated hash functions such as MD5 or SHA1 in use, or are non-cryptographic hash functions used when cryptographic hash functions are needed?
-   Are deprecated cryptographic padding methods such as PKCS number 1 v1.5 in use?
-   Are cryptographic error messages or side channel information exploitable, for example in the form of padding oracle attacks?

See ASVS Crypto (V7), Data Protection (V9), and SSL/TLS (V10)

## How to Prevent

Do the following, at a minimum, and consult the references:

-   Classify data processed, stored, or transmitted by an application. Identify which data is sensitive according to privacy laws, regulatory requirements, or business needs.
-   Don't store sensitive data unnecessarily. Discard it as soon as possible or use PCI DSS compliant tokenization or even truncation. Data that is not retained cannot be stolen.
-   Make sure to encrypt all sensitive data at rest.
-   Ensure up-to-date and strong standard algorithms, protocols, and keys are in place; use proper key management.
-   Encrypt all data in transit with secure protocols such as TLS with forward secrecy (FS) ciphers, cipher prioritization by the server, and secure parameters. Enforce encryption using directives like HTTP Strict Transport Security (HSTS).
-   Disable caching for response that contain sensitive data.
-   Apply required security controls as per the data classification.
-   Do not use legacy protocols such as FTP and SMTP for transporting sensitive data.
-   Store passwords using strong adaptive and salted hashing functions with a work factor (delay factor), such as Argon2, scrypt, bcrypt or PBKDF2.
-   Initialization vectors must be chosen appropriate for the mode of operation. For many modes, this means using a CSPRNG (cryptographically secure pseudo random number generator). For modes that require a nonce, then the initialization vector (IV) does not need a CSPRNG. In all cases, the IV should never be used twice for a fixed key.
-   Always use authenticated encryption instead of just encryption.
-   Keys should be generated cryptographically randomly and stored in memory as byte arrays. If a password is used, then it must be converted to a key via an appropriate password base key derivation function.
-   Ensure that cryptographic randomness is used where appropriate, and that it has not been seeded in a predictable way or with low entropy. Most modern APIs do not require the developer to seed the CSPRNG to get security.
-   Avoid deprecated cryptographic functions and padding schemes, such as MD5, SHA1, PKCS number 1 v1.5 .
-   Verify independently the effectiveness of configuration and settings.

#todo 

----

## References
1. https://owasp.org/Top10/