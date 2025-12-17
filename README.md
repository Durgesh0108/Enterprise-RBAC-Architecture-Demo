# Project Title: Enterprise Workflow Automation & RBAC Architecture

> **Note:** This repository serves as a technical case study and architectural overview. The source code is proprietary intellectual property of **R5 Advertising** and cannot be shared publicly. This document outlines the architectural decisions, database design, and optimization strategies employed during development.

## 1. Abstract
This project addressed critical operational bottlenecks in a multi-departmental organization (Marketing, Accounts, Design). The primary objective was to replace decentralized, manual data tracking with a **Centralized Monolithic Architecture**. By implementing **Role-Based Access Control (RBAC)** and **Database Normalization**, the system achieved an **85% reduction in billing latency** and ensured strict data consistency across 6 distinct departments.

## 2. Technical Stack
* **Frontend:** Next.js (React) - Utilized for dynamic client-side interactions.
* **Backend:** Node.js & Express.js - RESTful API architecture.
* **Database:** MongoDB (Mongoose) - Implemented complex aggregations and indexing.
* **Authentication:** Clerk / Custom JWT Middleware - For secure session management.
* **Infrastructure:** AWS (S3 for media storage), Vercel (Deployment).

## 3. The Engineering Challenge (Problem Statement)
Prior to this implementation, the organization faced significant scalability hurdles:
1.  **Data Redundancy:** Client data was duplicated across "Marketing" and "Accounts" spreadsheets, leading to synchronization errors.
2.  **Race Conditions:** Concurrent editing of billing ledgers by multiple staff members resulted in overwrites and financial discrepancies.
3.  **Security Risks:** Lack of granular permission levels meant junior staff had unrestricted access to sensitive financial data.

## 4. Architectural Solution & Methodology

### A. Database Normalization & Schema Design
To resolve data redundancy, I transitioned from flat-file storage to a relational-style schema within MongoDB. We separated `Client Entities` from `Project States`.

*Pseudocode Logic (Schema Reference):*
```javascript
// OLD WAY: Duplicated data (Inefficient)
// Invoice = { clientName: "ABC", clientAddress: "Mumbai", amount: 5000 }

// NEW WAY: Normalized Reference (Efficient)
const InvoiceSchema = new Schema({
  clientId: { 
    type: Schema.Types.ObjectId, 
    ref: 'Client', // References the single source of truth
    index: true    // Indexed for O(1) retrieval speed
  },
  projectIds: [{ type: Schema.Types.ObjectId, ref: 'Project' }],
  ledgerEntry: { type: Number, required: true },
  generatedAt: { type: Date, default: Date.now }
});
