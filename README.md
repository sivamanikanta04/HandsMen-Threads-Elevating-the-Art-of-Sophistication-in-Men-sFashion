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

## 1. Data Model

### **Custom Objects:**

1. **HandsMen Customer**
   - Fields: First Name, Last Name, Email, Phone, Loyalty Status, Total Purchases.
2. **HandsMen Product**
   - Fields: Product Name, Price, Stock Quantity, Description.
3. **HandsMen Order**
   - Fields: Order Name, Customer (Lookup), Product (Lookup), Quantity, Total Amount, Status (Confirmed/Pending/Rejection).
4. **Inventory**
   - Fields: Stock Quantity, Stock Status (Formula).
5. **Marketing Campaign**
   - Fields: Campaign Name, Start Date, End Date.

### **Relationships:**

- **Marketing Campaign → HandsMen Customer:** Lookup relationship.
- **HandsMen Product → HandsMen Order:** Lookup relationship.
- **HandsMen Order → HandsMen Customer:** Lookup relationship.
- **Inventory → HandsMen Product:** Master-Detail relationship.

### **Formula Fields:**

- **Inventory.Stock_Status\_\_c**  
  Formula:
  ```
  IF(Stock_Quantity__c > 10, "Available", "Low Stock")
  ```
- **HandsMen_Customer.Full_Name\_\_c**  
  Formula:
  ```
  FirstName__c + " " + LastName__c
  ```

---

## 2. Data Integrity with Validation Rules

### **HandsMen Order:**

- **Rule:** Total_Amount\_\_c <= 0
- **Error Message:** `Please Enter Correct Amount`

### **Inventory:**

- **Rule:** Stock_Quantity\_\_c <= 0
- **Error Message:** `The inventory count is never less than zero.`

### **HandsMen Customer:**

- **Rule:** NOT CONTAINS(Email, "@gmail.com")
- **Error Message:** `Please fill Correct Gmail`

---

## 3. App Manager Setup

- **App Name:** HandsMen Threads
- **Tabs:** HandsMen Customer, HandsMen Product, HandsMen Order, Inventory, Marketing Campaign.

---

## 4. Security Model

### **Profiles & Roles**

- **Profile:** Platform 1 (Cloned from Standard User).
  - Access to HandsMen Products and Inventory objects.
- **Roles:** CEO → Sales, Inventory, Marketing.

### **Users:**

- User 1: Niklaus.
- User 2: KOI.
- User 3: Larkin.

### **Permission Sets:**

- **Permission_Platform_1:** Full access to Customer and Order objects.

---

## 5. Automation

### **Email Templates:**

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
   ```html
   <p>Dear Inventory Manager,</p>
   <p>
     This is to inform you that the stock for the following product is running
     low:
   </p>
   <p>Product Name: {!Inventory__c.HandsMen_Product__c}</p>
   <p>Current Stock Quantity: {!Inventory__c.Stock_Quantity__c}</p>
   <p>Please take the necessary steps to restock this item immediately.</p>
   <p>Best Regards,</p>
   <p>nventory Monitoring System</p>
   ```
3. **Loyalty Program Email** – Updates customers about their loyalty status.
   ```html
   <p>
     Congratulations! You are now a {!HandsMen_Customer__c.Loyalty_Status__c}
     member and you are eligible for our Loyalty Rewards Program.
   </p>
   <p>
     Enjoy exclusive discounts, early access to offers, and special member
     benefits.
   </p>
   <p>Thank you for your continued Support.</p>
   ```

### **Flows:**

1. **Order Confirmation Flow:**

   - Triggered when Order**c.Status**c = "Confirmed".
   - Sends Order Confirmation Email Alert.

2. **Stock Alert Flow:**

   - Triggered when Inventory.Stock_Quantity\_\_c < 5.
   - Sends email to Inventory Manager.

3. **Loyalty Status Update Flow (Scheduled):**
   - Updates Loyalty Status based on Total Purchases (Gold/Silver/Bronze).

---

## 6. Apex Development

### **Apex Class: OrderTriggerHandler**

```apex
public class OrderTriggerHandler {
    public static void validateOrderQuantity(List<HandsMen_Order__c> orderList) {
        for (HandsMen_Order__c order : orderList) {
            if (order.Status__c == 'Confirmed') {
                if (order.Quantity__c == null || order.Quantity__c <= 500) {
                    order.Quantity__c.addError('For Status "Confirmed", Quantity must be more than 500.');
                }
            } else if (order.Status__c == 'Pending') {
                if (order.Quantity__c == null || order.Quantity__c <= 200) {
                    order.Quantity__c.addError('For Status "Pending", Quantity must be more than 200.');
                }
            } else if (order.Status__c == 'Rejection') {
                if (order.Quantity__c == null || order.Quantity__c != 0) {
                    order.Quantity__c.addError('For Status "Rejection", Quantity must be 0.');
                }
            }
        }
    }
}
```

### **Apex Trigger: OrderTrigger**

```apex
trigger OrderTrigger on HandsMen_Order__c (before insert, before update) {
    if (Trigger.isBefore && (Trigger.isInsert || Trigger.isUpdate)) {
        OrderTriggerHandler.validateOrderQuantity(Trigger.new);
    }
}
```

### **Batch Apex: InventoryBatchJob**

```apex
global class InventoryBatchJob implements Database.Batchable<SObject>, Schedulable {
    global Database.QueryLocator start(Database.BatchableContext BC) {
        return Database.getQueryLocator('SELECT Id, Stock_Quantity__c FROM Product__c WHERE Stock_Quantity__c < 10');
    }

    global void execute(Database.BatchableContext BC, List<SObject> records) {
        List<HandsMen_Product__c> productsToUpdate = new List<HandsMen_Product__c>();
        for (SObject record : records) {
            HandsMen_Product__c product = (HandsMen_Product__c) record;
            product.Stock_Quantity__c += 50;
            productsToUpdate.add(product);
        }
        if (!productsToUpdate.isEmpty()) {
            update productsToUpdate;
        }
    }

    global void finish(Database.BatchableContext BC) {
        System.debug('Inventory Sync Completed');
    }

    global void execute(SchedulableContext SC) {
        Database.executeBatch(new InventoryBatchJob(), 200);
    }
}
```

### **Scheduling the Batch Job**

Execute in **Anonymous Window**:

```
System.schedule('Daily Inventory Sync', '0 0 0 * * ?', new InventoryBatchJob());
```

---

## 7. Testing Strategy

### **Unit Testing for Apex:**

- Create test data for Orders, Products, and Inventory.
- Verify trigger conditions (Confirmed, Pending, Rejection).
- Validate that batch jobs update stock quantities correctly.

### **Flow Testing:**

- Change order status to Confirmed → Verify email.
- Reduce stock quantity below 5 → Verify alert email.
- Run scheduled flow → Check loyalty status updates.

---

## How to Use

1. **Login** to Salesforce.
2. Navigate to **HandsMen Threads App**.
3. Use **Tabs**: Customers, Products, Orders, Inventory, Marketing Campaigns.
4. **Create a test order** and verify email notifications.
5. **Check inventory** and loyalty updates from scheduled flows.

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

**Prepared By:** Sivamanikanta Gudla  
**Project Name:** HandsMen Threads  
