# Process Flow Document 02 — To-Be Automated Headless SaaS Flow
**Project:** Enterprise Omnichannel Return & Exchange SaaS Engine  
**Author:** Senior Business Analyst (Accenture Song Portfolio)  
**Status:** Target System Architecture

---

## 1. Architectural Narrative & Automation Goals

The "To-Be" workflow transitions the brand from a manual support pipeline to an automated self-service architecture. 

### Key Innovations:
* **Self-Service Initiation:** Shopper manages returns via a white-labeled web portal in under 60 seconds.
* **Exchange Nudge Logic:** Prior to refund approval, the engine offers 1-click size swaps and store credit bonus incentives (+₹150).
* **Doorstep Scan Instant Payout:** Webhook integration with reverse carriers (Delhivery) triggers instant RazorpayX UPI payouts the moment the delivery driver scans the parcel at the customer's doorstep.
* **Omnichannel (O2O) Store Return:** Real-time Point-of-Sale (POS) API lookup enables physical store clerks to accept online returns on the spot.

---

## 2. To-Be Process Flowchart (Mermaid.js)

```mermaid
graph TD
    A[Shopper clicks 'Return/Exchange' on Brand Site] --> B[SaaS Portal loads via CDN]
    B --> C{Rule Check: Order Age <= 7 Days?}
    
    C -- No --> D[Display 'Return Window Expired' Policy Notice]
    C -- Yes --> E{Rule Check: Category Returnable?}
    
    E -- No (e.g., Innerwear) --> F[Grey Out Checkbox + Render Tooltip]
    E -- Yes --> G[Shopper selects item to return]
    
    G --> H[Engine launches Exchange Nudge Modal]
    
    H --> I{Shopper Selection}
    I -- Size Swap --> J[Call Merchant Inventory API]
    J -- Stock Available --> K[Reserve Stock & Create Exchange Order]
    
    I -- Store Credit (+₹150 Bonus) --> L[Issue Instant Digital Gift Card Code]
    
    I -- Cash Refund --> M[Prompt for UPI VPA ID]
    M --> N[Asynchronous VPA Syntax & Account Verification]
    N -- Invalid VPA --> O[Display Real-Time Validation Error]
    N -- Valid VPA --> P[Trigger Delhivery API Reverse Courier Pickup]
    
    P --> Q[Courier Driver arrives for Doorstep Pickup]
    Q --> R[Driver scans parcel barcode at doorstep]
    R --> S[Carrier API sends 'PICKUP_SUCCESS' Webhook to Engine]
    
    S --> T{Engine Fraud Scoring Engine Check}
    T -- Score > Threshold (High Risk) --> U[Hold Payout for Warehouse Physical QC]
    T -- Score <= Threshold (Pass) --> V[Trigger RazorpayX API Instant Payout]
    
    V --> W[UPI Payout credited in < 60 Seconds]
    K --> X[Sync Inventory with Store POS & Central Warehouse]
    L --> X
    W --> X
