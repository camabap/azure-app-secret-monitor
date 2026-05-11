# 04 - Logic App Alert Configuration

This document provides a **step-by-step guide** to configure a Logic App that monitors Azure AD App Registration client secret expiration and sends alerts using Azure Communication Services.

---

## 🎯 Objective

This Logic App will:

- Run daily on a schedule
- Query Microsoft Graph for App Registrations
- Filter applications based on a prefix (lab scenario)
- Detect secrets expiring within a defined threshold
- Send alert emails

---

## 🧱 Prerequisites

- App Registration configured (see `03-app-registration.md`)
- Azure Communication Services Email configured
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
| Name | `la-app-secret-monitor` |
| Region | `<Your Region>` |

4. Click **Review + Create**
5. Go to Your Logic App and Select **Logic App Designer**

---

## ⏰ Step 2 – Configure Trigger (Recurrence)

Add trigger: **Recurrence**

## Configuration - Parameters

| Property  | Value |
|-----------|-------|
| Frequency | Day |
| Interval | 1 |
| Time zone | <Your Time Zone> |
| Hour   | 2 | #Hour and Minutes to Run Daily, 2:00AM in this example
| Minute | 0 |

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

## 🔁 Step 6 – Loop Application

Add action: **For Each**, Rename: **For_each_App**  
Input: **@body('Parse_JSON_Servidores')?['value']**


## 🔁 Step 7 – Loop Client Secrets

Inside previous loop: 
Add action: **For Each**, Rename: **For_each_client_secret**  
Input: **@items('For_each_App')?['passwordCredentials']**


## 🧪 Step 8 – Debug (Optional)

Add action: **COMPOSE**, Rename: **Compose_Debug**  
Input: **@item()**


## 🔍 Step 9 – Condition

Add action: **Condition**, Rename: **Checa_Secrets_a_Expirar**  

Expression: 
**@less(
  ticks(item()?['endDateTime']),
  ticks(addDays(utcNow(), variables('thresholdDays')))
)**


## ✉️ Step 10 – Compose Email Body (Inside True)

Add action: **COMPOSE**, Rename: **Compose_email_body**, inside True (Green) Side  

In Parameters, enter:
**<p><strong>Atenção: Client Secret próximo de expirar</strong></p>

<p><strong>Aplicação:</strong> @{items('For_each_App')?['displayName']}</p>
<p><strong>AppId:</strong> @{items('For_each_App')?['appId']}</p>

<p><strong>Client Secret:</strong> @{items('For_each_client_secret')?['displayName']}</p>
<p><strong>Hint:</strong> @{items('For_each_client_secret')?['hint']}</p>
<p><strong>Data de expiração:</strong> @{items('For_each_client_secret')?['endDateTime']}</p>

<p>Este secret expira em menos de 30 dias. Favor tomar as ações necessárias.</p>
**


## 📧 Step 11 – Send Email  

Add action: Send Email (From Azure Communication Service)

---

### Configuration - Parameters

| Property        | Value |
|-----------------|-------|
| From | `<your-sender@azurecomm.net>` |
| To   | `<your-email@domain.com>`     |
| Subject | `[ALERT] Secret expiring - @{items('For_each_App')?['displayName']}` |
| Body  | Output from Compose_email_body |
| Importance | High |
``

---

## 🔍 Where to get each configuration value

This section explains where to retrieve the **From** required in **Step 11 – Send Email**, which is mandatory for Azure Communication Services Email.

---

### 📧 From (IMPORTANT)

| Property | Description |
|----------|------------|
| From | Email address configured in Azure Communication Services |

✅ This value **must come from Azure Communication Services (ACS)**.

It **will NOT work** with:

- Personal email accounts (e.g., Gmail, Outlook.com)
- Office 365 mailbox
- Any address not registered in ACS

---

### 📍 How to find the Sender Address

1. Go to Azure Portal  
2. Navigate to: **Azure Communication Services**
3. Select your resource

In the left menu, go to: **Email → Domains**
Click your verified domain (e.g., yourdomain.azurecomm.net)
Click : **MailFromAddresses**
Open: **Sender usernames**

Copy one of the available MailFrom Addresses: **DoNotReply@<your-acs-domain>.azurecomm.net**

🛡️ Security Best Practices

Store secrets in Azure Key Vault
Prefer Managed Identity (future improvement)

Use least privilege:
Application.Read.All

Enable diagnostics logs

📌 Summary
This Logic App:

Runs daily
Queries Microsoft Graph securely
Filters apps using a configurable prefix
Detects expiring secrets
Sends alerts automatically
