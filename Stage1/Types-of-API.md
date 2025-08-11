# Types of APIs and Their Real-Time Usage

## 1. Based on Access Level

### a) Open APIs (Public APIs)
- **Description:** Available for any developer to use, usually over the internet.
- **Security:** Often require an API key but are generally open.
- **Examples:**
  - OpenWeatherMap API (weather data)
  - Google Maps API (map embedding)
  - Twitter API (public tweets)
- **Use Case:** Apps needing public data (news, travel, dashboards).

---

### b) Partner APIs
- **Description:** Accessible only to specific business partners via agreements.
- **Security:** Require authentication and access control.
- **Examples:**
  - Payment gateways like Stripe, Razorpay
  - Airline booking APIs for travel partners
- **Use Case:** B2B integrations.

---

### c) Internal APIs (Private APIs)
- **Description:** Used within a company for internal apps.
- **Security:** Restricted to internal teams.
- **Examples:**
  - HR system connecting to payroll
  - Internal microservices
- **Use Case:** Internal system communication.

---

### d) Composite APIs
- **Description:** Combine multiple service/data calls into one request.
- **Examples:**
  - E-commerce checkout (payment + inventory update + email notification)
- **Use Case:** Reduce multiple API calls for efficiency.

---

## 2. Based on Technology / Protocol

### a) REST API (Representational State Transfer)
- **Description:** Uses HTTP methods (GET, POST, PUT, DELETE).
- **Data format:** JSON (commonly).
- **Examples:**
  - Instagram API (posts, media)
  - E-commerce product details
- **Why Popular:** Easy, scalable, widely supported.

---

### b) SOAP API (Simple Object Access Protocol)
- **Description:** XML-based, strict structure, enterprise-friendly.
- **Security:** WS-Security (XML Encryption, XML Signature, Authentication).
- **Examples:**
  - Banking & payment systems
  - Telecom billing
- **Why Used:** Strong security, reliability.

---

### c) GraphQL API
- **Description:** Clients request exactly the needed data.
- **Examples:**
  - Facebook API (fetch only specific profile fields)
  - E-commerce (fetch product name & price only)
- **Why Used:** Prevents over-fetching/under-fetching.

---

### d) WebSocket API
- **Description:** Real-time, two-way communication.
- **Examples:**
  - Live chat apps (WhatsApp Web)
  - Stock market live prices
- **Why Used:** Instant updates without refresh.

---

### e) gRPC API
- **Description:** High-performance RPC framework using Protocol Buffers.
- **Examples:**
  - Netflix microservices
  - Online gaming servers
- **Why Used:** Low latency, efficient.

---

## 3. SOAP API Security Overview
SOAP ensures security mainly through **WS-Security**:
- **Encryption (XML Encryption)** → Confidentiality
- **Digital Signature (XML Signature)** → Integrity & authentication
- **UsernameToken / Certificates** → Authentication
- **Timestamps & IDs** → Replay attack prevention
- **Advantage:** Message-level security (works across multiple hops).

---

## 4. REST vs Others in Security
- **REST/GraphQL/gRPC/WebSocket** → Usually rely on **transport-level** security (HTTPS/TLS).
- **SOAP** → Can use both transport-level and message-level security.

---

## 5. Quick Comparison Table

| API Type      | Encryption        | Signature Support  | Security Level      | Message-level Security? |
|---------------|-------------------|--------------------|---------------------|--------------------------|
| SOAP          | XML Encryption    | XML Signature      | Message + Transport | ✅ Yes |
| REST          | TLS/HTTPS         | JWT Signature      | Transport           | ❌ No |
| GraphQL       | TLS/HTTPS         | JWT Signature      | Transport           | ❌ No |
| WebSocket     | WSS (TLS)         | No built-in        | Transport           | ❌ No |
| gRPC          | TLS/mTLS          | JWT / mTLS Certs   | Transport           | ❌ No |

---

## 6. API Selection Guide

```text
Start
  |
  |-- Need to share data with public developers?
  |        ├─ Yes → REST API or GraphQL
  |        └─ No → Internal use
  |
  |-- Need real-time updates?
  |        ├─ Yes → WebSocket API or gRPC streaming
  |        └─ No → Standard API
  |
  |-- High-security/compliance project (banking, healthcare)?
  |        ├─ Yes → SOAP API with WS-Security
  |        └─ No → REST or gRPC
  |
  |-- Need flexible, specific data queries?
  |        ├─ Yes → GraphQL
  |        └─ No → REST
  |
  End
