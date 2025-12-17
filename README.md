# Project Title: Enterprise Workflow Automation & RBAC Architecture

> **Note:** This repository serves as a technical case study and architectural overview. The source code is proprietary intellectual property of **R5 Advertising** and cannot be shared publicly. This document outlines the architectural decisions, database design, and optimization strategies employed during development.

## 1. Abstract
This project addressed critical operational bottlenecks in a multi-departmental organization (Marketing, Accounts, Design). The primary objective was to replace decentralized, manual data tracking with a **Centralized Monolithic Architecture**. By implementing **Role-Based Access Control (RBAC)** and **Database Normalization**, the system achieved an **85% reduction in billing latency** and ensured strict data consistency across 6 distinct departments.

## 2. Technical Stack
* **Framework:** Next.js (Fullstack) - Leveraged for both React frontend and API routes (Backend).
* **Database:** MongoDB - NoSQL document storage.
* **ORM:** Prisma - Used for type-safe database queries and schema management.
* **Authentication:** Clerk - Managed secure sessions and user identity.
* **File Storage:** * **Cloudinary:** For optimizing and serving frontend media assets.
    * **AWS S3:** For secure archival of generated invoice PDFs and internal documents.

## 3. The Engineering Challenge (Problem Statement)
Prior to this implementation, the organization faced significant scalability hurdles:
1.  **Data Redundancy:** Client data was duplicated across "Marketing" and "Accounts" spreadsheets, leading to synchronization errors.
2.  **Race Conditions:** Concurrent editing of billing ledgers by multiple staff members resulted in overwrites and financial discrepancies.
3.  **Security Risks:** Lack of granular permission levels meant junior staff had unrestricted access to sensitive financial data.

## 4. Architectural Solution & Methodology

### A. Database Normalization (Prisma Schema)
To resolve data redundancy, I transitioned from flat-file storage to a relational-style schema within MongoDB using Prisma. We separated `Client` entities from `Invoice` states to ensure a single source of truth.

*Pseudocode Logic (schema.prisma reference):*
```prisma
// OLD WAY: Duplicated data (Inefficient)
// Invoice = { clientName: "ABC", clientAddress: "Mumbai", amount: 5000 }

// NEW WAY: Normalized Reference (Efficient)
model Client {
  id        String    @id @default(auto()) @map("_id") @db.ObjectId
  name      String
  invoices  Invoice[] // One-to-Many relation
}

model Invoice {
  id          String   @id @default(auto()) @map("_id") @db.ObjectId
  amount      Float
  status      String   @default("PENDING")
  
  // Relation establishing Single Source of Truth
  client      Client   @relation(fields: [clientId], references: [id])
  clientId    String   @db.ObjectId
  
  createdAt   DateTime @default(now())
}
```

### B. Role-Based Access Control (RBAC) ###
Since Next.js API routes were used, I implemented a wrapper function to validate Clerk session claims before executing server-side logic.

Pseudocode Logic (Security Layer):

```js
import { auth } from '@clerk/nextjs';

export async function POST(req) {
  const { sessionClaims } = auth();
  
  // 1. Check if user is authenticated
  if (!sessionClaims) return new Response("Unauthorized", { status: 401 });

  // 2. Role Verification (e.g., Only 'ADMIN' or 'ACCOUNTS' can generate bills)
  const userRole = sessionClaims.metadata.role;
  if (userRole !== 'ADMIN' && userRole !== 'ACCOUNTS') {
    return new Response("Forbidden: Insufficient Permissions", { status: 403 });
  }

  // 3. Execute Business Logic
  return await generateInvoice(req);
}
```

### C. Asynchronous Billing Automation ###
To prevent main-thread blocking during complex invoice PDF generation, the system utilizes an event-driven approach. When a project status is updated to `COMPLETED`, the system triggers a background process that compiles the data, generates the PDF, and uploads it to AWS S3 for secure storage.

## 5. Quantitative Results & Impact ##
* **Latency Reduction:** Automated invoice generation reduced the process from **4 hours to ~40 minutes (85% reduction)** per week.

* **Error Elimination:** Zero instances of data mismatch between Marketing and Accounts departments post-launch.

* **Scalability:** The system successfully handles concurrent requests from 20+ active users without degradation in API response time.
