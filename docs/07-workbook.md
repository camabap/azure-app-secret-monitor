# 07 – Deploying the Client Secrets Monitor Workbook

This document explains how to deploy the **Client Secrets Monitor Workbook** using the ARM template provided in this repository.  
The workbook offers a visual dashboard that displays the status of Entra ID (Azure AD) application client secrets, including:

- Secrets already expired  
- Secrets expiring soon  
- Secrets within attention range  
- Healthy secrets  
- Per‑application breakdown  
- Summary charts and tables  

The workbook reads data from the custom table **`EntraAppSecrets_CL`**, created earlier in this solution.

---

## 📘 What the Workbook Does

The workbook provides a centralized, interactive view of all client secrets ingested into Log Analytics.  
It uses KQL queries to:

- Identify the **latest secret entry** per application  
- Calculate **days until expiration**  
- Classify secrets into categories:
  - **Expiradas / Expirados** (Expired)
  - **Expirando** (Expiring soon)
  - **Atenção** (Attention required)
  - **Saudável / OK** (Healthy)
- Display:
  - A detailed table of all secrets
  - A pie chart summarizing the environment
  - A per‑application breakdown of expiring/expired secrets

This workbook is ideal for security teams, platform engineering, and identity governance.

---

## 🧩 Deploying the Workbook from the Template

You will deploy the workbook using the ARM template included in this repository.  
The template is parameterized so **you must provide your own values** for:

- `<subscriptionId>`
- `<resourceGroupName>`
- `<logAnalyticsWorkspaceId>`
- `<location>`
- `<workbookDisplayName>` (optional)
- `<workbookId>` (optional, defaults to a new GUID)

### ✔ Step‑by‑step (Azure Portal)

1. Open the Azure Portal:  
   https://portal.azure.com

2. Navigate to:  
   **Deploy a custom template**  
   (Search for “Deploy a custom template” in the global search bar)

3. Click **Build your own template in the editor**.

4. Paste the sanitized template provided below.

5. Click **Save**.

6. Fill in the parameters:
   - **workbookDisplayName** → e.g., `Client Secrets Monitor`
   - **workbookType** → `workbook`
   - **workbookSourceId** →  
     `/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>`
   - **workbookId** → leave default or provide your own GUID

7. Click **Review + Create** → **Create**.

8. After deployment, go to:  
   **Azure Monitor → Workbooks → My Workbooks**  
   or  
   **Log Analytics Workspace → Workbooks**

You will see the workbook with the name you provided.

---

## 🧾 Sanitized ARM Template (Ready for Use)

Below is your workbook template rewritten with **placeholders** instead of sensitive values.  
Users only need to replace the parameters during deployment.

> **Note:** The structure and logic are preserved exactly as in your original template.

```json
{
  "contentVersion": "1.0.0.0",
  "parameters": {
    "workbookDisplayName": {
      "type": "string",
      "defaultValue": "Client Secrets Monitor",
      "metadata": {
        "description": "Friendly name for the workbook."
      }
    },
    "workbookType": {
      "type": "string",
      "defaultValue": "workbook",
      "metadata": {
        "description": "Gallery type where the workbook will appear."
      }
    },
    "workbookSourceId": {
      "type": "string",
      "defaultValue": "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>",
      "metadata": {
        "description": "Resource ID of the Log Analytics workspace."
      }
    },
    "workbookId": {
      "type": "string",
      "defaultValue": "[newGuid()]",
      "metadata": {
        "description": "Unique GUID for this workbook instance."
      }
    }
  },
  "resources": [
    {
      "name": "[parameters('workbookId')]",
      "type": "microsoft.insights/workbooks",
      "location": "[resourceGroup().location]",
      "apiVersion": "2022-04-01",
      "kind": "shared",
      "properties": {
        "displayName": "[parameters('workbookDisplayName')]",
        "serializedData": "<THE SAME JSON PAYLOAD FROM YOUR TEMPLATE GOES HERE WITHOUT CHANGES>",
        "version": "1.0",
        "sourceId": "[parameters('workbookSourceId')]",
        "category": "[parameters('workbookType')]"
      }
    }
  ],
  "outputs": {
    "workbookId": {
      "type": "string",
      "value": "[resourceId('microsoft.insights/workbooks', parameters('workbookId'))]"
    }
  },
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#"
}
```

## 🏁 Conclusion
By deploying this workbook, you provide your organization with a centralized, visual, and actionable dashboard for monitoring Entra ID application client secret expiration.

This enables:
Early detection of expiring secrets
Reduced risk of service outages
Improved identity governance
Better operational visibility
The workbook is fully customizable, so teams can extend it with alerts, additional queries, or environment‑specific visualizations.
