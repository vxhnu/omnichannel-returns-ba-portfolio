# Process Flow Document 01 — As-Is Legacy Manual Return Flow
**Project:** Enterprise Omnichannel Return & Exchange SaaS Engine  
**Author:** Vishnu Rameshan  
**Status:** Baseline Reference Model

---

## 1. Context & Operational Bottlenecks

In the legacy "As-Is" environment, fast-fashion brands process return requests manually through fragmented customer support channels (Email, WhatsApp, Instagram DMs). This disconnected workflow causes severe operational friction:

1. **High Resolution Latency:** Support agents spend 15–20 minutes per ticket verifying invoice dates and manually checking courier serviceability.
2. **Delayed Cash-on-Delivery (COD) Refunds:** Collecting customer bank account details over email requires manual entries into net-banking platforms, leading to a 7-to-10 business day payout delay.
3. **100% Cash Leakage:** Because there is no automated exchange nudge prompt at the point of return initiation, over 90% of requests result in hard cash refunds.
4. **Zero Channel Synchronization:** Physical store counters cannot verify online order receipts, forcing customers who visit physical stores to ship parcels back via mail.

---

## 2. As-Is Process Flowchart (Mermaid.js)

```mermaid
graph TD
    A[Customer receives item] --> B[Wants size exchange or return]
    B --> C[Sends Email / Instagram DM to Support Team]
    C --> D[Support Agent manually verifies invoice timestamp]
    
    D --> E{Order Delivered <= 7 Days?}
    E -- No --> F[Agent manually types Rejection Email]
    E -- Yes --> G[Agent emails form requesting Bank Details / UPI VPA]
    
    G --> H[Customer fills and emails back Bank Details]
    H --> I[Agent logs into Courier Admin to book pickup]
    I --> J[Courier picks up parcel in 3 to 5 Days]
    J --> K[Parcel in transit to central warehouse: 5 to 7 Days]
    
    K --> L[Warehouse Agent receives parcel & inspects garment]
    L --> M{Passes Quality Inspection?}
    M -- No --> N[Garment rejected; Dispute ticket raised]
    M -- Yes --> O[Accounts Team manually logs into Bank Portal]
    O --> P[Manual NEFT / RTGS Wire Transfer executed]
    P --> Q[Refund Credited after 7-10 Business Days]
