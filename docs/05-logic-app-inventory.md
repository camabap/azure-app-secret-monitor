# 05 - Logic App Inventory Configuration

This document provides a comprehensive, step-by-step guide for configuring a Logic App to monitor Azure AD application registration client secrets and send the data to a custom table..

---

## 🎯 Objective

This Logic App will:

- Run daily on a schedule
- Query Microsoft Graph for App Registrations
- Filter applications based on a prefix (lab scenario)
- Save data to a custom table

---

## 🧱 Prerequisites

- App Registration configured (see `03-app-registration.md`)
- Required permissions:
  - `Application.Read.All`
- Azure subscription

---

## ⚠️ Lab Behavior (Important)

For lab purposes, the Logic App only evaluates applications whose name starts with:
**Demo-**

This is controlled by a variable: **prefix = "Demo-"**

You can modify it later:

"" → evaluate all applications  
"prod-" → production apps  
"dev-" → development apps

## 🚀 Step 1 – Create Logic App

1. Go to Azure Portal  
2. Click **Create Resource**  
3. Search **Logic App (Consumption)**  

## Configuration

| Setting | Value |
|--------|------|
| Subscription | `<Your Subscription>` |
| Resource Group | `<Your Resource Group>` |
| Name | `la-app-secret-law` |
| Region | `<Your Region>` |

4. Click **Review + Create**
5. Go to Your Logc App, Click **Identity** and In System Assigned Tab on **Status**, Click **On**
6. Select **Logic App Designer**

---

## ⏰ Step 2 – Configure Trigger (Recurrence)

Add trigger: **Recurrence**

## Configuration - Parameters

| Property  | Value |
|-----------|-------|
| Frequency | Day |
| Interval | 1 |
| Time zone | `<Your Time Zone>` |
| Hour   | 2 | 
| Minute | 0 |

💡 **Note:** Hour and Minute define the daily execution time.  
> In this example, the Logic App runs at **02:00 AM**.

---

## 🔧 Step 3 – Initialize Variables

Add action: **Initialize variable**

Add those Variables:
---
**### prefix**

| Property | Value |
|----------|------|
| Name | prefix |
| Type | String |
| Value | Demo- |

---

**### thresholdDays**

| Property | Value |
|----------|------|
| Name | thresholdDays |
| Type | Integer |
| Value | 30 |

---

**### tenantId**

| Property | Value |
|----------|------|
| Name | tenantId |
| Type | String |
| Value | `<tenantId>` |

---

**### clientId**

| Property | Value |
|----------|------|
| Name | clientId |
| Type | String |
| Value | `<clientId>` |

---

**### clientSecret**

| Property | Value |
|----------|------|
| Name | clientSecret |
| Type | String |
| Value | `<clientSecret>` |

> ⚠️ In production, store this in Azure Key Vault

---

**### notifyTo**

| Property | Value |
|----------|------|
| Name | notifyTo |
| Type | String |
| Value | `<your-email@domain.com>` |

---

**### expiringItems**

| Property | Value |
|----------|------|
| Name | expiringItems |
| Type | Array |
| Value | empty |

---

## 🌐 Step 4 – HTTP Action (Microsoft Graph)

Add action: **HTTP**, Rename: **HTTP_Servidores**

### Method: **GET**
### URI : **https://graph.microsoft.com/v1.0/applications?$select=id,appId,displayName,passwordCredentials&$filter=startswith(displayName,'@{variables('prefix')}')&$count=true**
### Headers

| Key | Value |
|-----|------|
| Accept | application/json |
| ConsistencyLevel | eventual |

### Authentication
Use: **Active Directory OAuth**

| Property | Value |
|----------|------|
| Authority | https://login.microsoftonline.com |
| Tenant | @{variables('tenantId')} |
| Audience | https://graph.microsoft.com |
| Client ID | @{variables('clientId')} |
| Secret | @{variables('clientSecret')} |

---

## 🧾 Step 5 – Parse JSON

Add action: **Parse JSON**, Rename: **Parse_JSON_Servidores**

### Content: **@body('HTTP_Servidores')**

### Schema

```json
{
  "value": [
    {
      "id": "string",
      "appId": "string",
      "displayName": "string",
      "passwordCredentials": [
        {
          "displayName": "string",
          "keyId": "string",
          "startDateTime": "string",
          "endDateTime": "string"
        }
      ]
    }
  ]
}
```

---

## 🔁 Step 6 – Loop Application

Add action: **For Each**  
Input: **@body('Parse_JSON_Servidores')?['value']**

---

## 🔁 Step 7 – Loop Client Secrets

Inside previous loop: 
Add action: **For Each** 
Input: **@items('For_each')?['passwordCredentials']**

---

## 🧪 Step 8 – Compose Record

Add action: **COMPOSE** 
Input: 
```json
{
  "type": "Compose",
  "inputs": {
    "ApplicationName": "@{items('For_each')?['displayName']}",
    "SecretName": "@{items('For_each_1')?['displayName']}",
    "SecretExpirationDate": "@{items('For_each_1')?['endDateTime']}"
  }
}
```

---

## 🌐 Step 9 – HTTP Action (Microsoft Graph) - Send Data to the Custom Table

Add action: **HTTP**

### Method: **POST**
### URI : <Your DCE Log Ingestion>/dataCollectionRules/<You DCR Immutable ID>/streams/<Your Custom Table>?api-version=2023-01-01

### Headers

| Key | Value |
|-----|------|
| Accept | application/json |
| ConsistencyLevel | eventual |

### Body
```json
[
"@outputs('Compose')"
]
```

### Authentication
Use: **Managed identity** as Authentication Type

| Property | Value |
|----------|------|
| Managed Identity | system-assigned managed identity |
| Audience | `https://monitor.azure.com` |

**Where to Find Information to URI**

**`<Your DCE Log Ingestion>`**

Go to Monitor -> Data Collection Endpoints -> Click Your Data Collection Endpoint
In the Overview Page -> Copy **Log Ingestion URI**

**`<You DCR Immutable ID>`**

Go to Monitor -> Data Collection Rules -> Click The Data Collection Rule of the Custom Table
In the Overview Page -> Copy **ImmutableID**

**`<Your Custom Table>`** -> Custom-`<Custom Table Name>`, Ex: Custom-EntraAppSecrets_CL
