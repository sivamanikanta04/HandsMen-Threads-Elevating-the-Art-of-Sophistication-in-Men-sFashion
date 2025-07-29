# HandsMen Threads - Elevating the Art of Sophistication in Men's Fashion

---

## Description

Developing a premium fashion platform to streamline men’s bespoke tailoring and enhance customer experience through personalized styling and seamless order management.

---

## Overview

HandsMen Threads is a premium fashion platform designed to streamline men’s bespoke tailoring and enhance the customer experience through personalized styling, seamless order management, and automated processes.

This Salesforce project improves **data management**, **customer relations**, and **workflow automation** using **Salesforce CRM**, **Flows**, **Apex**, and **Batch Jobs**.

---

## Key Features

- **Automated Order Confirmations** – Customers receive confirmation emails upon order placement.
- **Dynamic Loyalty Program** – Loyalty status updates based on total purchases.
- **Proactive Stock Alerts** – Automatic emails sent to the warehouse team when inventory is low.
- **Scheduled Bulk Order Updates** – Daily batch jobs update financial records and inventory levels.
- **Data Quality Enforcement** – Validation rules and formula fields ensure accurate data.

---

## Project Phases

### Phase 1: Architecture & Planning

- Defined **Objects**:
  - `HandsMen_Customer__c`
  - `HandsMen_Product__c`
  - `HandsMen_Order__c`
  - `Inventory__c`
  - `Marketing_Campaign__c`
- Designed **Fields & Relationships** (Lookup & Master-Detail).
- Created **Formula Fields** for calculated values like Full Name and Stock Status.
- Set up **Validation Rules** for data quality.
- Planned **automation** using Flows, Apex, and Batch Jobs.
- Designed email templates for **Order Confirmation**, **Stock Alerts**, and **Loyalty Updates**.

### Phase 2: Development

- Created **Custom Tabs** and **App: HandsMen Threads**.
- Implemented **Record-Triggered Flows** for email alerts and loyalty program.
- Developed **Apex Trigger** (`OrderTrigger`) to enforce business rules on order quantity.
- Created **Batch Apex Class** (`InventoryBatchJob`) for inventory restocking.
- Set up **Profiles, Roles, and Permission Sets** for security and user access.
- Configured **Email Templates**:
  - Order Confirmation Email
  - Low Stock Alert Email
  - Loyalty Program Email

### Phase 3: Testing & QA

- **Unit Tests** for Apex triggers and batch jobs.
- Verified **flows and automation** with sample data.
- Validated **email alerts** by placing test orders.
- Tested **data validation rules** (e.g., email format, stock quantity).
- Ensured **security roles** work as expected (Sales, Inventory, Marketing).

### Phase 4: Deployment & Training

- Deployed the solution to **production org**.
- Conducted **end-user training** for Sales, Inventory, and Marketing teams.
- Provided **post-go-live monitoring** and support.

---

## Data Model

### Object Relationships

- **HandsMen Customer ↔ Marketing Campaign** – Lookup Relationship.
- **HandsMen Product ↔ HandsMen Order** – Lookup Relationship.
- **HandsMen Order ↔ HandsMen Customer** – Lookup Relationship.
- **Inventory ↔ HandsMen Product** – Master-Detail Relationship.

### Important Fields

- **HandsMen Customer**: FirstName, LastName, Email, Phone, Loyalty_Status, Full_Name (Formula).
- **Inventory**: Stock_Quantity, Stock_Status (Formula).
- **HandsMen Order**: Status, Quantity, Total_Amount, Postal_Code.
- **Marketing Campaign**: Campaign Name, Start Date, End Date.

---

## Validation Rules

- **HandsMen Order**: Total*Amount\_\_c <= 0 → *"Please Enter Correct Amount."\_
- **Inventory**: Stock*Quantity\_\_c <= 0 → *"The inventory count is never less than zero."\_
- **HandsMen Customer**: NOT CONTAINS(Email, "@gmail.com") → _"Please fill Correct Gmail."_

---

## Automation

### Flows

- **Order Confirmation Flow** – Sends confirmation email when `Order.Status = Confirmed`.
- **Stock Alert Flow** – Sends email alert if `Inventory.Stock_Quantity < 5`.
- **Loyalty Status Update Flow (Scheduled)** – Updates customer loyalty tier (Gold/Silver/Bronze) based on `Total_Purchases`.

### Apex

- **OrderTriggerHandler** – Validates order quantities based on order status.
- **OrderTrigger** – Runs validation before insert/update.
- **InventoryBatchJob** – Restocks products with less than 10 units and runs nightly.

---

## Security Setup

- **Profile**: Cloned Standard User → Platform 1.
- **Roles**:
  - CEO → Sales → Inventory → Marketing.
- **Permission Set**: `Permission_Platform_1` for CRUD access on key objects.
- **Users**: Created sample users (e.g., Niklaus, KOI) with assigned roles.

---

## Email Templates

1. **Order Confirmation Email**  
   Subject: _Your Order has been Confirmed!_  
   Body:

   ```html
   <p>Dear {!Order__c.Customer__c},</p>
   <p>Your order #{!Order__c.Name} has been confirmed!</p>
   <p>Thank you for shopping with us.</p>
   <p>Best Regards,<br />Sales Team</p>
   ```

2. **Low Stock Alert Email** – Notifies warehouse when inventory is below 5.
3. **Loyalty Program Email** – Updates customers about their loyalty status.

---

## Batch Job Scheduling

- **Daily Inventory Sync**:
  ```apex
  System.schedule('Daily Inventory Sync', '0 0 0 * * ?', new InventoryBatchJob());
  ```

---

## Testing Scenarios

1. **Create Order with Status = Confirmed** → Email confirmation should be triggered.
2. **Set Stock Quantity < 5** → Stock Alert Email should be sent.
3. **Update Customer Purchases > 1000** → Loyalty Status updates to Gold.
4. **Validation** – Enter invalid Total Amount or Email to see error messages.
5. **Run Batch Job** – Verify that low-stock products are restocked automatically.

---

## Technology Stack

- **Salesforce CRM**
- **Lightning App Builder**
- **Flows & Process Automation**
- **Apex & Apex Triggers**
- **Asynchronous Apex (Batch Apex)**
- **Profiles, Roles & Permission Sets**
- **Email Templates & Alerts**

---

## How to Use

1. **Login** to Salesforce.
2. Navigate to **HandsMen Threads App**.
3. Use **Tabs**: Customers, Products, Orders, Inventory, Marketing Campaigns.
4. **Create a test order** and verify email notifications.
5. **Check inventory** and loyalty updates from scheduled flows.

---

---

## Author

**Project by:** Sivamanikanta Gudla  
**Role:** Salesforce Developer / Admin
