# 03 – App Registration Configuration

This document describes how to configure an Azure AD (Microsoft Entra ID) **App Registration** used to monitor client secret expiration across applications.

The solution relies on **Microsoft Graph API** to enumerate applications and credentials, following **least privilege and enterprise security best practices**.

---

## 🎯 Objective

Create and configure an App Registration that:

- Authenticates securely using **application (app-only) permissions**
- Reads application registrations and their credentials (client secrets)
- Enables automation scenarios (Logic Apps, Functions, etc.)

---

## 🧱 Prerequisites

- Azure subscription
- Permissions:
  - `Application Administrator` or `Cloud Application Administrator` (recommended)
  - OR ability to register applications in Microsoft Entra ID
- Access to Azure Portal

---

## 🚀 Step 1 – Create the App Registration

1. Open **Azure Portal**
2. Navigate to:
   - **Microsoft Entra ID** → **App registrations**
3. Click **+ New registration**
4. Configure:

| Field | Value |
|------|------|
| Name | `aad-secret-monitor-<environment>` |
| Supported account types | Accounts in this organizational directory only |
| Redirect URI | *(Leave empty)* |

5. Click **Register**

---

## 🆔 Step 2 – Capture Application Identifiers

After creation, capture the following values:

| Property | Description | Placeholder |
|----------|------------|-------------|
| Application (client) ID | Unique identifier of the app | `<clientId>` |
| Directory (tenant) ID | Azure AD tenant ID | `<tenantId>` |

---

## 🔐 Step 3 – Create a Client Secret

> ⚠️ **Security Best Practice**  
> Prefer **certificate-based authentication** in production environments. Use client secrets only if necessary.

1. Go to:
   - **Certificates & secrets**
2. Click **+ New client secret**
3. Configure:

| Field | Value |
|------|------|
| Description | `secret-monitor-auth` |
| Expiration | Recommended: 6–12 months |

4. Click **Add**
5. Copy the secret value immediately

| Property | Placeholder |
|----------|-------------|
| Client Secret | `<clientSecret>` |

> ⚠️ This value is only shown once. Store it securely (e.g., Azure Key Vault).

---

## 🔑 Step 4 – Assign Microsoft Graph Permissions

The application requires permissions to **read applications and credentials**.

1. Navigate to:
   - **API permissions**
2. Click **+ Add a permission**
3. Select:
   - **Microsoft Graph**
   - **Application permissions**

### Required Permissions

| Permission | Type | Description |
|------------|------|-------------|
| `Application.Read.All` | Application | Read all applications |
| `Directory.Read.All` *(optional but recommended)* | Application | Read directory data |

> ✅ **Minimum Required:**  
> - `Application.Read.All`

> ✅ **Recommended for reliability:**  
> - `Application.Read.All`  
> - `Directory.Read.All`

4. Click **Add permissions**

---

## ✅ Step 5 – Grant Admin Consent

1. On the **API permissions** page:
2. Click **Grant admin consent for <tenantId>**
3. Confirm

---

## 🔍 Step 6 – Validate Access to Microsoft Graph

You can validate access using a simple query.

### Example: List Applications

```http
GET https://graph.microsoft.com/v1.0/applications
Authorization: Bearer <accessToken>
