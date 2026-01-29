# Architecture (High-Level)

This document describes the high-level architecture for the Dance School Billing Automation platform.

The MVP is an admin-only, cloud-native web application focused on automating billing workflows for dance schools.

---

## Core Architecture Components (MVP)

### 1) Admin Web App (Frontend)
- Used by school administrators
- Later extended for teachers (e.g. class registers with child photos)
- Provides screens to:
  - Manage children, classes, and enrollments
  - View and manage invoices
  - Add one-off fees (e.g. late fees)
  - Trigger bulk overdue invoice reminders

---

### 2) Authentication
- Secure admin login
- Controls access to billing and child data

---

### 3) API Layer
- Exposes authenticated endpoints to the frontend
- Handles request validation and authorization
- Routes requests to backend business logic

---

### 4) Business Logic (Backend)
Implements core workflows:
- Generate invoices for a billing period (monthly or term)
- Apply bulk class price updates
- Add one-off fees as invoice line items
- Identify overdue and outstanding invoices
- Send bulk reminder emails
- Record reminder send attempts and outcomes

---

### 5) Data Store
Stores all core system data:
- BillingAccount and Address
- Child, Class, Enrollment
- BillingPeriod
- Invoice and InvoiceLineItem
- ReminderLog

---

### 6) Email Service
- Sends invoice reminders
- Handles retries and failures
- Provides delivery feedback where available
- Integrates with backend logic for bulk sends

---

## Key MVP Flows

### Flow A: Bulk Overdue Invoice Reminders
1. Admin selects “Send reminders” in the UI
2. API authorizes the request
3. Backend queries invoices where:
   - status != paid
   - dueDate < today
4. Backend sends reminder emails
5. Backend writes a ReminderLog entry per invoice
6. UI shows a summary (sent / failed)

---

### Flow B: Add One-Off Fee
1. Admin selects an invoice
2. Admin adds a fee (e.g. £15 late fee)
3. Backend creates a new InvoiceLineItem
4. Invoice total is recalculated
5. Updated invoice is displayed

---

### Flow C: Bulk Class Price Update
1. Admin updates a class price
2. Backend identifies affected enrollments and invoices
3. Backend applies updates via:
   - adjusted tuition line items, or
   - adjustment line items
4. UI displays a summary of changes

---

## AWS Mapping (Draft – MVP)

The conceptual components map to AWS services as follows:

- **Frontend**
  - S3 (static site hosting)
  - CloudFront (content delivery)

- **Authentication**
  - Cognito (admin authentication)

- **API Layer**
  - API Gateway (HTTP API)

- **Backend Logic**
  - AWS Lambda (serverless functions)

- **Data Store**
  - DynamoDB (serverless NoSQL database)

- **Email Service**
  - Amazon SES

- **Observability**
  - CloudWatch (logs and metrics)

---

## Future Enhancements (NOT MVP)

### Communications & Activity Timeline
A unified view of all communication and interaction events related to an invoice.

- Track:
  - reminder emails sent
  - delivery failures
  - invoice link clicks
  - invoice views
- Enables a “Communications” tab per invoice

Future data concept:
- CommunicationLog / InvoiceEvent

---

### Financials Dashboard
A reporting and action screen for administrators.

Capabilities:
- Filter invoices by:
  - outstanding
  - overdue
  - billing period
  - billing account
- Display totals (amount due, overdue balances)
- One-click bulk reminder sending from filtered results

---

### Scheduled & Deferred Invoicing
- Pre-generate invoices for the year
- Control when invoices become payable (e.g. 1 week before due)
- Allow adjustments before invoices are issued

---

### Split Billing
- Multiple billing accounts per child
- Percentage-based invoice splitting
- Separate reminders and balances

---

### Payments & Refunds
- Record payments against invoices
- Support partial and full refunds
- Maintain financial audit history

---

## Design Principles

- MVP focuses on admin efficiency, not customer-facing features
- Data model drives architecture decisions
- Bulk operations are first-class features
- Future features are enabled by design, not bolted on
