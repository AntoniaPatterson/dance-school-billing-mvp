# Data Model (MVP)

This document defines the core data model for the Dance School Billing Automation platform.

The MVP supports:
- Monthly and term-based billing
- One-off custom fees (late fees now; costume/admin fees later)
- Bulk overdue invoice reminders
- Multiple siblings under one billing account
- Child safeguarding and admin information
- Internal child photos for registers (teacher use)
- External photo permissions for social media/marketing
- One address per billing account (including borough)

The model is designed to support future features (split billing, payments, refunds) without requiring redesign.

---

## Core Entities (MVP)

### BillingAccount
Represents who receives and pays invoices (usually a parent/guardian).
- billingAccountId (PK)
- primaryContactName
- email
- phone 
- status (active | inactive)
- createdAt
- updatedAt

---

### Address
Address record linked to a BillingAccount.
- addressId (PK)
- line1
- line2 (optional)
- city
- postcode
- borough 
- country (default UK)

---

### BillingAccountAddress
Links a BillingAccount to an Address.
MVP assumes one primary address per billing account.
- linkId (PK)
- billingAccountId (FK)
- addressId (FK)
- type (primary | billing | postal)
- isPrimary (true/false)

---

### Child
A child/student.

Notes:
- Child photos are for **internal identification only** (registers/teachers).
- External photo permissions apply only to marketing/social media use.

- childId (PK)
- fullName
- dateOfBirth 
- gender 
  (e.g. female | male )
- ethnicity
  (stored as free text or controlled list later)
- status (active | inactive)

- allergies (optional, free text)

- emergencyContactName 
- emergencyContactPhone 
- emergencyContactRelationship (optional)

- photoUrl (optional)
- photoUpdatedAt (optional)

- externalPhotoPermissionStatus (granted | denied | unknown)
- externalPhotoPermissionNotes (optional)

- createdAt
- updatedAt

---

### ChildBillingAccountLink
Links a child to one or more billing accounts.

MVP:
- Each child links to exactly one billing account.

Future:
- Multiple billing accounts per child with split percentages.

- linkId (PK)
- childId (FK)
- billingAccountId (FK)
- splitPercent (optional; e.g. 50/50 – NOT FOR MVP)
- isPrimary (optional)
- startDate (optional)
- endDate (optional)

---

### Class
A class offered by the school (e.g. Ballet Grade 1).
- classId (PK)
- name
- category (competitive | recreational | other)
- basePrice
- currency (GBP)
- billingCadence (monthly | term)
- isActive
- createdAt
- updatedAt

---

### Enrollment
Links a child to a class.
- enrollmentId (PK)
- childId (FK)
- classId (FK)
- startDate
- endDate (optional)
- status (active | ended)

---

### BillingPeriod
Represents a billable period (month or term).
- billingPeriodId (PK)
- type (month | term)
- name (e.g. "Jan 2026" or "Spring Term 2026")
- periodStart
- periodEnd

---

### Invoice
A bill issued to a BillingAccount for a BillingPeriod.
- invoiceId (PK)
- billingAccountId (FK)
- billingPeriodId (FK)
- issueDate
- dueDate
- status (draft | scheduled | issued | overdue | paid | void)
- totalAmount
- createdAt
- updatedAt

---

### InvoiceLineItem
Line items on an invoice: tuition, late fees, costume fees, admin fees, adjustments.
- lineItemId (PK)
- invoiceId (FK)
- type (tuition | fee | adjustment)
- feeCode (optional; e.g. LATE_FEE, COSTUME, ADMIN)
- description
- amount
- childId (optional)
- classId (optional)
- createdAt

---

### ReminderLog
Tracks invoice reminders sent.
- reminderId (PK)
- invoiceId (FK)
- sentAt
- channel (email)
- recipientEmail
- status (sent | failed)
- errorMessage (optional)

---

## Relationships

- BillingAccount 1 → many Invoice
- Invoice 1 → many InvoiceLineItem
- Child many → many Class (via Enrollment)
- Child 1 → many BillingAccount (via ChildBillingAccountLink)
- BillingPeriod 1 → many Invoice
- BillingAccount 1 → 1 primary Address (via BillingAccountAddress)

Notes:
- MVP assumes one BillingAccount per Child.
- Split billing is a future enhancement.

---

## MVP Feature Support

### 1) Bulk class price updates
- Prices live on Class (Class.basePrice).
- Price changes update affected tuition line items or create adjustment items.

### 2) One-off fees (late fee now; others later)
- Fees are InvoiceLineItems with type=fee and a feeCode.
- Supports any future fee without schema changes.

### 3) Bulk overdue invoice reminders
- Overdue invoices identified via Invoice.status and dueDate.
- Reminder sends logged in ReminderLog.

---

## Future Extensions (NOT FOR MVP)

### Payment
Tracks money received against an invoice.
- paymentId (PK)
- invoiceId (FK)
- amount
- paidAt
- method (cash | bank_transfer | card | other)
- reference (optional)
- status (captured | pending | failed)

### Refund
Tracks money returned to a payer (full or partial).
- refundId (PK)
- paymentId (FK)
- amount
- refundedAt
- reason (optional)
- status (processed | pending | failed)
- reference (optional)

Notes:
- Supports full and partial refunds.
- Multiple refunds can be applied to a single payment.
- Not required for MVP.

