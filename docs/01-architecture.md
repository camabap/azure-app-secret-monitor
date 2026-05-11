# Architecture

## 📌 Overview

This document describes the architecture of the Azure solution designed to monitor the expiration of Azure AD (Entra ID) App Registration client secrets.

The solution uses Azure-native services and automation workflows to:

- Continuously discover App Registrations
- Evaluate client secret expiration
- Send proactive notifications
- Store inventory data for analysis
- Provide visualization through dashboards

---

## 🏗️ High-Level Architecture

The solution is composed of the following core components:

- Azure AD App Registration (Authentication and data access)
- Azure Logic Apps (Automation and orchestration)
- Azure Communication Services (Email notifications)
- Log Analytics Workspace (Data storage and querying)
- Data Collection Endpoint (DCE) and Data Collection Rule (DCR) (Custom data ingestion)
- Azure Workbook (Visualization layer)

---

## 🔄 Architecture Flow

The solution operates in two main flows:

### 1. Alerting Flow (Real-time monitoring)

1. Logic App authenticates using App Registration  
2. Retrieves all App Registrations via Microsoft Graph API  
3. Iterates through each application  
4. Extracts client secrets (passwordCredentials)  
5. Evaluates expiration date  
6. If expiration is below defined threshold (e.g., 30 days):  
   → Sends email using Azure Communication Services  

---

### 2. Inventory & Monitoring Flow

1. Logic App retrieves all App Registrations  
2. Extracts relevant metadata:
   - Application Name
   - Application ID
   - Secret Name / Identifier
   - Expiration Date  
3. Formats data into structured payload  
4. Sends data to Log Analytics using DCR/DCE  
5. Data is stored in a custom table  

---

### 3. Visualization Flow

1. Azure Workbook queries Log Analytics  
2. Displays:
   - Expiring secrets  
   - Secret distribution  
   - Application overview  
3. Enables operational monitoring  

---

## 🧩 Component Interaction Diagram

```text
+-----------------------------+
| Azure AD App Registrations  |
+-------------+---------------+
              |
              v
+-----------------------------+
|        Logic App            |
|  (Secret Discovery Engine)  |
+-------------+---------------+
              |
      +-------+--------+
      |                |
      v                v
+-----------+   +--------------------+
| Alerting  |   | Data Collection     |
| Logic App |   | Logic App           |
+-----+-----+   +---------+----------+
      |                   |
      v                   v
+-------------+   +----------------------+
| Email Alert |   | Log Analytics        |
| (ACS)       |   | Custom Table         |
+-------------+   +----------+-----------+
                              |
                              v
                    +----------------------+
                    | Azure Workbook       |
                    | Visualization        |
                    +----------------------+
``
