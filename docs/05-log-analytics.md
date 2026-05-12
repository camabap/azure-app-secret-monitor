# 05 - Log Analytics and data collection for Entra app secret monitoring

This document describes how to configure Azure Monitor and Log Analytics to receive and store data about Entra ID (Azure AD) application registration client secret expirations.

You will:

- Create a Log Analytics workspace  
- Create a Data Collection Endpoint (DCE)  
- Create a custom table for app secret metadata  
- Create the Data Collection Rule (DCR) as part of the custom table creation  

All examples use placeholders such as `<tenantId>`, `<subscriptionId>`, and `<resourceGroupName>`—replace them with values from your environment.

---

## 1. Prerequisites

- **Azure subscription:** You have an active subscription where you can deploy monitoring resources.
- **Permissions in Azure:**
  - **At subscription or resource group scope:**  
    - `Contributor` or a combination of:
      - `Log Analytics Contributor` on the workspace  
      - `Monitoring Contributor` for DCR/DCE  
- **Entra ID / Microsoft Graph app registration:**
  - An app registration in tenant `<tenantId>` that will:
    - Read application registrations and their secrets metadata.
    - Push data to the custom table (via your automation, not directly from Graph).

### 1.1 Microsoft Graph permissions (least privilege)

To read application registrations and their client secret metadata (name, expiration date, etc.), configure **application permissions** on your monitoring app registration:

- **Required Microsoft Graph application permissions:**
  - `Application.Read.All`  
    - Allows the app to read all application registrations, including `passwordCredentials` (secret metadata such as `displayName` and `endDateTime`).
- **Optional (only if you truly need it):**
  - `Directory.Read.All`  
    - Broader directory read; avoid unless your solution requires additional directory objects.

> **Best practice:**  
> - Do **not** grant `Application.ReadWrite.All` or `Directory.ReadWrite.All`—they exceed the needs of a read-only monitoring solution.  
> - Use **app-only** authentication with a certificate or client secret stored in Azure Key Vault.  
> - Ensure an Entra ID Global Administrator or Privileged Role Administrator grants **admin consent** for the selected permissions.

---

## 2. Create the Log Analytics workspace

You can create the workspace in a dedicated resource group, for example `<rg-monitoring>`.

1. **Sign in** to the Azure Portal with an account that has the required permissions.
2. In the Search menu, type and search for **Log Analytics Workspace**.
3. Click **+ Create**.

5. On the **Basics** tab:
   - **Subscription:** Select your subscription.
   - **Resource group:** Select or create a new resource Group.
   - **Name:** Enter a workspace name, for example: (`law-entra-app-secrets`).
   - **Region:** Choose the region where you want to host the workspace (e.g., `East US`, `Brazil South`).  
     - **Important:** Use the same region later for the DCE and DCR.

6. Review the settings and click **Review + create**, then **Create**.

> **Security best practice:**  
> - Restrict access to the workspace using RBAC (e.g., `Log Analytics Reader` for consumers, `Log Analytics Contributor` for admins).  
> - Avoid granting `Owner` unless strictly necessary.

---

## 3. Create the Data Collection Endpoint (DCE)

1. In the Azure portal, search for **Data collection endpoints**.
2. Click **+ Create**.

3. On the **Basics** tab:
   - **Subscription:** Select your subscription.
   - **Resource group:** Use the same as Log Analytics Workspace.
   - **Name:** Use the same naming convention as your template, for example: `DCE-ClientSecretExpiration`
   - **Region:** Select the same region as your Log Analytics workspace (e.g., `East US`).  

4. Click **Review + create**, then **Create**.

> **Best practice:**  
> - Keep DCEs dedicated per solution or per security boundary.  
> - If you later lock down network access, ensure your automation (Functions, Logic Apps, etc.) can still reach the DCE.

---

## 4. Create the custom table and DCR

You will now create a **custom table** in the Log Analytics workspace and, as part of that process, a **Data Collection Rule (DCR)** that routes data from the DCE into the table.

### 4.2 Create the custom table (portal)

1. In the Azure portal, go to your **Log Analytics workspace**.
2. In the left menu, under **Settings**, select **Tables**.
3. Click **+ Create**.
4. In Create Custom Log, On the **Basics** tab:
   - **Table name:** Give a Name to the Table, ex:`EntraAppSecrets_CL`, don't type the _CL, it will be added automatically  
     - The `_CL` suffix is standard for custom logs.
   - In **Table Plan**, select **Analytics**
   - In Data collection rule, click **Create a New Data Collection Rule**, **Give a Name** for the DCR, and Click **Done**
   - In Data collection Endpoint, Select your DCE created in item 3
    
7. Click **Next** to **Schema and Transformation**:

7.1 First create a json file and include the sample bellow, save the file:

```json
[
  {
    "ApplicationName": "demo-expiring-app-001",
    "SecretName": "lab-secret-001",
    "SecretExpirationDate": "2026-05-31T03:00:00Z"
  }
]   
```
7.2 Click **Upload Sample File** and attach this file  
7.3 You Will recieve This error: TimeGenerated field is not found in the sample provided. Please use the transformation editor to populate TimeGenerated column  in the destination table, **click Transformation editor**
7.4 In the Editor, Paste and **run this query**, then click **apply**:  

```kql
source
| extend TimeGenerated = now()
```
7.5 Click **Next** and **Create**

## 5. Summary

At this point, your environment should have:

A Log Analytics workspace <workspaceName> in resource group <rg-monitoring>.
A Data Collection Endpoint <dceName> (e.g., DCE-ClientSecretExpiration).
A custom table EntraAppSecrets_CL with columns for app secret metadata.
A Data Collection Rule <dcrName> that maps incoming JSON payloads to the custom table.
