# Document 02 — Functional Requirements, Business Rules & NFR Specifications
**Project:** Enterprise Omnichannel Return & Exchange SaaS Engine  
**Author:** Vishnu Rameshan  
**Status:** Approved Technical Blueprint

---

## 1. Functional Requirements Document (FRD)

The Functional Requirements below define the mandatory system actions, workflows, and automated behaviors required across the platform. All requirements follow the enterprise standard format: **"The [System] SHALL [Action] WHEN [Condition] IN ORDER TO [Business Goal]."**

### 1.1 Customer Portal & Initiation (`FR-PORT`)

| Requirement ID | Module / Feature | Priority | Requirement Description |
| :--- | :--- | :--- | :--- |
| **FR-PORT-001** | Order Authentication | Must Have | The system **SHALL** validate customer order numbers and phone numbers via OTP **WHEN** a user accesses the portal **IN ORDER TO** prevent unauthorized order access. |
| **FR-PORT-002** | Eligibility Verification | Must Have | The system **SHALL** disable item selection and display policy badges **WHEN** an item's delivery timestamp exceeds 7 calendar days **IN ORDER TO** enforce merchant return windows. |
| **FR-PORT-003** | Reason Taxonomies | Must Have | The system **SHALL** require users to select a mandatory reason code (e.g., Size Too Small, Defective, Wrong Item) and upload at least 1 image **WHEN** requesting a return **IN ORDER TO** capture quality audit data. |

### 1.2 Exchange Nudge Engine (`FR-NDG`)

| Requirement ID | Module / Feature | Priority | Requirement Description |
| :--- | :--- | :--- | :--- |
| **FR-NDG-001** | Size Swap Routing | Must Have | The system **SHALL** query real-time OMS inventory and display immediate size swap options **WHEN** a user selects "Size Too Small/Large" **IN ORDER TO** retain order revenue. |
| **FR-NDG-002** | Store Credit Incentive | Must Have | The system **SHALL** calculate and add a merchant-configured bonus credit (e.g., +₹150) **WHEN** a user selects Store Credit over cash refund **IN ORDER TO** incentivize repeat purchases. |
| **FR-NDG-003** | Dynamic Cart Upsell | Should Have | The system **SHALL** allow users to select a different product category and pay the differential amount via gateway **WHEN** exchanging for a higher-priced item **IN ORDER TO** drive positive incremental GMV. |

### 1.3 Payment Gateways & UPI Payouts (`FR-PAY`)

| Requirement ID | Module / Feature | Priority | Requirement Description |
| :--- | :--- | :--- | :--- |
| **FR-PAY-001** | VPA Real-Time Validation | Must Have | The system **SHALL** validate UPI VPA syntax and account holder registration asynchronously **WHEN** a customer submits payout details **IN ORDER TO** eliminate failed transaction charges. |
| **FR-PAY-002** | Webhook Instant Payout | Must Have | The system **SHALL** invoke RazorpayX API to credit customer bank accounts **WHEN** a `PICKUP_SUCCESS` webhook is received from the reverse carrier **IN ORDER TO** achieve < 60-second refund TAT. |
| **FR-PAY-003** | Payout Fallback Handler | Must Have | The system **SHALL** flag payout status as `REQUIRES_MANUAL_REVIEW` and alert support **WHEN** automated UPI payout execution returns a gateway failure code **IN ORDER TO** prevent duplicate disbursements. |

### 1.4 Reverse Logistics & Carriers (`FR-LOG`)

| Requirement ID | Module / Feature | Priority | Requirement Description |
| :--- | :--- | :--- | :--- |
| **FR-LOG-001** | Waybill & Manifest Gen | Must Have | The system **SHALL** generate a reverse shipping label PDF and register pickup tasks with carrier APIs **WHEN** a return request is submitted **IN ORDER TO** automate reverse courier booking. |
| **FR-LOG-002** | Doorstep Scan Webhook | Must Have | The system **SHALL** ingest and parse real-time GPS barcode scan events **WHEN** the field agent scans the parcel barcode at the doorstep **IN ORDER TO** trigger downstream state transitions. |

### 1.5 Omnichannel & Store Sync (`FR-OMS`)

| Requirement ID | Module / Feature | Priority | Requirement Description |
| :--- | :--- | :--- | :--- |
| **FR-OMS-001** | POS Barcode Lookup | Must Have | The system **SHALL** allow retail store associates to fetch online order details via barcode scan **WHEN** a customer presents an online item in-store **IN ORDER TO** process instant O2O returns. |
| **FR-OMS-002** | Inventory Auto-Restock | Must Have | The system **SHALL** increment local store POS inventory counts **WHEN** an in-store return passes physical QC inspection **IN ORDER TO** make inventory immediately available for resale. |

---

## 2. Business Rules Engine (BRE) Logic Matrix

The Business Rules Engine operates as a configurable policy layer decoupled from core software code, enabling merchants to adjust parameters via their SaaS dashboard.
| Rule ID | Policy Name | Parameter / Business Rule Description | Logic & Mathematical Formula |
| :--- | :--- | :--- | :--- |
| **BRE-ELG-01** | Policy Window Gate | Restricts return submission based on carrier delivery timestamp. | `Return_Eligible = (Current_Timestamp - Carrier_Delivery_Timestamp) <= Merchant_Policy_Days` |
| **BRE-EXC-01** | Non-Returnable Items | Excludes sanitary, clearance, or perishable items from return selection. | `Selection_Allowed = (Item_Category NOT IN ["Innerwear", "Socks", "Perfumes", "Clearance"])` |
| **BRE-CRD-01** | Store Credit Incentive | Applies a merchant-funded bonus to the item refund value. | `Total_Credit = Paid_Item_Price + Bonus_Amount` *(where Bonus_Amount = ₹150 default)* |
| **BRE-FEE-01** | Reverse Logistics Fee | Deducts shipping fee for cash refunds, waiving it for exchanges. | `Cash_Refund = Paid_Item_Price - Reverse_Courier_Fee` *(Fee waived if Option == Exchange OR Store_Credit)* |
| **BRE-FRD-01** | Fraud Risk Threshold | Holds instant payouts for accounts demonstrating suspicious activity. | `IF (Customer_Returns_30_Days >= 3 OR Account_Age_Days < 7) THEN Payout_Mode = "MANUAL_QC_HOLD"` |

---

## 3. Non-Functional Requirements (NFRs)

NFRs define system performance, security standards, scalability bounds, and operational resilience.

### 3.1 Performance & Latency Requirements (`NFR-PERF`)
* **NFR-PERF-01 (API Latency):** Core lookup APIs (order status, eligibility check) **MUST** respond within **< 200ms** at the 95th percentile under normal operating loads.
* **NFR-PERF-02 (Webhook Execution):** Doorstep carrier webhook parsing and RazorpayX instant payout initiation **MUST** execute within **< 60 seconds** end-to-end.
* **NFR-PERF-03 (Page Load Speed):** Customer-facing portal web pages **MUST** render fully interactive within **1.5 seconds** on 3G cellular connections.

### 3.2 Security & Compliance Specifications (`NFR-SEC`)
* **NFR-SEC-01 (Data Encryption):** All customer PII, bank account numbers, and UPI VPAs **MUST** be encrypted at rest using **AES-256** and in transit using **TLS 1.3**.
* **NFR-SEC-02 (API Authentication):** All inbound webhook payloads from reverse carriers **MUST** be verified using **HMAC SHA-256 signatures** to prevent request spoofing.
* **NFR-SEC-03 (PCI-DSS & Compliance):** Payment flow architecture **MUST** maintain full **PCI-DSS Level 1** compliance; financial data must never be logged in plain text.

### 3.3 Scalability & Reliability (`NFR-SCL` / `NFR-AVL`)
* **NFR-SCL-01 (Concurrent Load):** Cloud microservices **MUST** auto-scale horizontally to support up to **10,000 concurrent active checkout/return sessions** during flash sales.
* **NFR-AVL-01 (Uptime SLA):** Core platform microservices **MUST** guarantee **99.99% operational uptime** annually (maximum scheduled downtime < 52 minutes/year).
* **NFR-AVL-02 (Disaster Recovery):** Multi-region database replication **MUST** maintain a Recovery Point Objective (RPO) **< 1 minute** and Recovery Time Objective (RTO) **< 15 minutes**.
