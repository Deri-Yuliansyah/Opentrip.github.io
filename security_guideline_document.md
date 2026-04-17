# Security Guideline Document for Open Trip Website

This document outlines the security requirements, best practices, and implementation guidance for the **Open Trip Website**. It aligns with OWASP Top 10 2025, PCI DSS v4.0, and modern cloud security standards to ensure our platform is secure by design.

---

## 1. Core Security Principles

1. **Security by Design**  
   Integrate security at every phase—from requirements, architecture, and development to testing and deployment.
2. **Least Privilege**  
   Grant minimum necessary permissions to users, services, and database accounts.
3. **Defense in Depth**  
   Layer controls (network, application, data) so no single failure compromises the system.
4. **Fail Securely**  
   On error, default to deny. Do not leak stack traces, internal paths, or PII to users.
5. **Secure Defaults & Simplicity**  
   Ship with secure configuration out of the box. Favor clear, maintainable code over complex solutions.

---

## 2. Architecture & Trust Boundaries

- **Frontend (Next.js + React)**
  - Runs in user browsers—treat as untrusted environment.
  - Communicates over HTTPS to backend API.
- **Backend (Node.js/Express + TypeScript)**
  - Hosts REST API, Socket.io chat, webhooks, and admin endpoints.
  - Logic layer enforces authentication, authorization, and input validation.
- **Database (MongoDB Atlas)**
  - Stores users, trips, bookings, payments, reviews, notifications.
  - Access only via backend service account with scoped roles.
- **Third-Party Integrations**
  - **Payments**: Stripe Connect, PayPal SDK, Midtrans (future).
  - **Email**: SendGrid/Mailgun.
  - **Maps**: Google Maps JavaScript API.
  - **Calendar**: Google Calendar API / iCal.

Draw explicit trust boundaries between client, server, and external services. All data crossing these boundaries must be validated, sanitized, and encrypted in transit.

---

## 3. Authentication & Access Control

### 3.1 Robust Authentication
- **Email/Password**: Enforce strong password policy (min 12 characters, mixed-case, numbers, symbols).  
- **Password Storage**: Use Argon2 or bcrypt with per-user salt.
- **Account Verification**: Email confirmation flows before first login.
- **Forgot Password**: One-time, time-bound tokens stored hashed in DB.

### 3.2 Session & Token Security
- **JWT**:  
  - Use `HS256` or `RS256` with securely managed keys.  
  - Include `exp`, `iat`, `iss`, and `sub`.  
  - Store tokens in Secure, HttpOnly, SameSite=Strict cookies.
- **Idle & Absolute Timeouts**:  
  - Idle timeout: 15 minutes.  
  - Absolute expiry: 24 hours.
- **Logout**: Invalidate refresh tokens server-side.
- **MFA**: Step-up authentication for high-risk actions (e.g., payout setup).

### 3.3 Role-Based Access Control (RBAC)
- Define roles: `Admin`, `TripLeader`, `Participant`.
- Authorize every endpoint via middleware—never trust client-supplied role claims.
- Prevent IDOR by validating that `userId` in path/query belongs to the authenticated principal.

---

## 4. Input Handling & Output Encoding

### 4.1 Server-Side Validation
- **JSON Schema** for API requests. Reject unknown properties.
- **Parametrized Queries & ORM**: Use Mongoose with strictly typed models to prevent injection.
- **Strict Type Checking** via TypeScript.

### 4.2 Prevent Injection Attacks
- **No String Interpolation** in database queries.
- **Sanitize** file upload names to prevent path traversal.
- **Whitelist** file types (images: JPG/PNG) and size limits (max 5 MB).

### 4.3 Cross-Site Scripting (XSS)
- **Context-Aware Escaping**: Always HTML-encode user-supplied data in React with `dangerouslySetInnerHTML` only after sanitization.
- **Content Security Policy (CSP)** header to restrict script sources.

### 4.4 CSRF Protection
- **Synchronizer Token Pattern** on all state-changing requests.
- For API calls, require custom header `X-CSRF-Token` or use SameSite=strict cookies.

---

## 5. Data Protection & Privacy

### 5.1 Encryption
- **In Transit**: Enforce TLS 1.3 on frontend and backend (HSTS header).  
- **At Rest**: Enable MongoDB Atlas encryption.  
- **Sensitive Fields**: Encrypt PII (phone, email) using field-level encryption if necessary.

### 5.2 Secrets Management
- Store all API keys, DB credentials, and JWT private keys in a vault (e.g., AWS Secrets Manager).  
- Rotate keys quarterly or on suspected compromise.

### 5.3 Minimization & Retention
- Only collect fields documented in PRD.  
- Implement data retention policies: delete inactive users and old logs after 1 year.

### 5.4 Logging & Monitoring
- **Mask** sensitive fields in logs.  
- Centralize logs to SIEM (e.g., Elastisearch, Splunk).  
- Implement anomaly alerting (e.g., multiple failed logins, unusual payout requests).

---

## 6. API & Service Security

### 6.1 HTTPS & Rate Limiting
- **Enforce HTTPS**: Redirect all HTTP to HTTPS in Vercel/Load Balancer.  
- **Rate Limit**: 100 requests per IP per minute on public endpoints; stricter on auth and payment routes.

### 6.2 CORS Policy
- Allow only trusted origins (`*.our-opentrip.com`).  
- Preflight responses must include `Access-Control-Allow-Credentials: true` and restricted methods.

### 6.3 Webhooks Security
- **Signing**: Validate Stripe & PayPal webhooks via their signature headers.  
- **Replay Protection**: Reject events older than 5 minutes.

---

## 7. Web Application Security Hygiene

- **HTTP Security Headers**:
  - `Content-Security-Policy`, `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`, `Expect-CT`, `Strict-Transport-Security`.
- **Cookie Attributes**: `Secure`, `HttpOnly`, `SameSite=Strict`.
- **Subresource Integrity (SRI)**: For any CDN-loaded scripts/styles.
- **Debugging**: Disable verbose error pages in production. Return generic messages to clients.

---

## 8. Infrastructure & Configuration Management

- **Harden Servers**: Disable unused ports/services; apply CIS Benchmarks to EC2/Heroku instances.
- **Network Segmentation**: Place DB in a private subnet; only backend hosts can reach it.
- **TLS Configuration**: Use AWS ACM/Vercel managed certs; disable TLS < 1.3.
- **Patching & Updates**: Automate OS and dependency updates; schedule monthly maintenance windows.

---

## 9. Dependency Management

- **SBOM & SCA**: Maintain a Software Bill of Materials. Integrate Snyk or Dependabot in CI/CD.
- **Lockfiles**: Commit `package-lock.json` to source.  
- **Periodic Audits**: Quarterly dependency review; upgrade to patched versions promptly.
- **Minimal Footprint**: Audit third-party libs; remove unused code to reduce attack surface.

---

## 10. Monitoring, Incident Response & Testing

- **Penetration Testing**: Annual external pen-test against OWASP Top 10:2025.
- **SAST & DAST**: Integrate static analysis (ESLint, SonarQube) and dynamic vulnerability scans (OWASP ZAP) in GitHub Actions.
- **WAF**: Deploy cloud WAF (e.g., Cloudflare/AWS WAF) with:
  - Positive security model for APIs (block non-schema-conforming JSON).  
  - Behavioral bot mitigation to prevent inventory hoarding.  
  - Geo-blocking for unused regions.
- **Incident Playbooks**: Define steps for breach containment, forensic analysis, public disclosure, and key rotation.

---

## 11. Summary & Next Steps

- Review and sign off this document with Product, Engineering, and Security teams.
- Incorporate controls into JIRA tickets and definition of done for each story.
- Schedule periodic security reviews and updates every 6 months or after major feature launches.

By following these guidelines, we will ensure our Open Trip Website remains secure, resilient, and trustworthy for travelers and organizers alike.
