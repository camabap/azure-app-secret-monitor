# 07 - Deploying the Client Secrets Monitor Workbook

This document explains how to deploy the **Client Secrets Monitor Workbook** using the Gallery template provided in this repository.  
The workbook offers a visual dashboard that displays the status of Entra ID (Azure AD) application client secrets, including:

- Secrets already expired  
- Secrets expiring soon  
- Secrets within attention range  
- Healthy secrets  
- Per‑application breakdown  
- Summary charts and tables  

The workbook reads data from the custom table **`EntraAppSecrets_CL`**, created earlier in this solution.

---

## 📘 What the Workbook Does?

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

### 🧾 Sanitized ARM Template (Ready for Use)

Below is your workbook template.

## How to Deploy?

# Workbook setup instructions

Follow the steps below to create and load your custom workbook in **Azure Log Analytics**.

---

1. Open your **Log Analytics Workspace** in the Azure Portal.
2. Go to Workbooks
3. In the left navigation menu, under **Monitoring**, click **Workbooks**.
4. Click **New+** to create a new workbook.
5. Click the **Code** icon **`</>`** to switch to the JSON editor.
6. Delete all existing content and paste the new JSON code bellow.

### Attention: The workbook queries the `EntraAppSecrets_CL` table. If your table has a different name, you must replace all occurrences of `EntraAppSecrets_CL`.

> **Note:** The structure and logic are preserved exactly as in your original template.

```json
{
  "version": "Notebook/1.0",
  "items": [
    {
      "type": 1,
      "content": {
        "json": "## Client Secrets Monitor\n---\n\nMonitoração da Expiração dos Client Secrets\nAbaixo a visão geral dos client Secrets com Dias a expirar"
      },
      "name": "text - 2"
    },
    {
      "type": 3,
      "content": {
        "version": "KqlItem/1.0",
        "query": "let LatestSecrets =\r\n    EntraAppSecrets_CL\r\n    | summarize arg_max(TimeGenerated, *) by ApplicationName, SecretName;\r\n\r\nLatestSecrets\r\n| extend DaysToExpire = datetime_diff('day', SecretExpirationDate, now())\r\n| extend Status =\r\n    case(\r\n        DaysToExpire < 0, \"Expiradas\",\r\n        DaysToExpire <= 30, \"Expirando\",\r\n        DaysToExpire <= 90, \"Atencao\",\r\n        \"Saudavel\"\r\n    )\r\n| project\r\n    ApplicationName,\r\n    SecretName,\r\n    SecretExpirationDate,\r\n    DaysToExpire,\r\n    Status\r\n| order by DaysToExpire asc",
        "size": 0,
        "title": "Visão dos Client Secrets",
        "queryType": 0,
        "resourceType": "microsoft.operationalinsights/workspaces",
        "visualization": "table",
        "gridSettings": {
          "formatters": [
            {
              "columnMatch": "DaysToExpire",
              "formatter": 18,
              "formatOptions": {
                "thresholdsOptions": "colors",
                "thresholdsGrid": [
                  {
                    "operator": "<",
                    "thresholdValue": "0",
                    "representation": "redDark",
                    "text": "{0}{1}"
                  },
                  {
                    "operator": "<=",
                    "thresholdValue": "30",
                    "representation": "redBright",
                    "text": "{0}{1}"
                  },
                  {
                    "operator": "<=",
                    "thresholdValue": "90",
                    "representation": "yellow",
                    "text": "{0}{1}"
                  },
                  {
                    "operator": "Default",
                    "thresholdValue": null,
                    "representation": "green",
                    "text": "{0}{1}"
                  }
                ]
              }
            },
            {
              "columnMatch": "Status",
              "formatter": 18,
              "formatOptions": {
                "thresholdsOptions": "icons",
                "thresholdsGrid": [
                  {
                    "operator": "==",
                    "thresholdValue": "Expiradas",
                    "representation": "3",
                    "text": "{0}{1}"
                  },
                  {
                    "operator": "==",
                    "thresholdValue": "Expirando",
                    "representation": "4",
                    "text": "{0}{1}"
                  },
                  {
                    "operator": "==",
                    "thresholdValue": "Atencao",
                    "representation": "2",
                    "text": "{0}{1}"
                  },
                  {
                    "text": "{0}{1}"
                  },
                  {
                    "operator": "Default",
                    "thresholdValue": null,
                    "representation": "1",
                    "text": "{0}{1}"
                  }
                ]
              }
            }
          ]
        }
      },
      "name": "query - 2"
    },
    {
      "type": 3,
      "content": {
        "version": "KqlItem/1.0",
        "query": "let LatestSecrets =\r\n    EntraAppSecrets_CL\r\n    | summarize arg_max(TimeGenerated, *) by ApplicationName, SecretName;\r\n\r\nLatestSecrets\r\n| extend DaysToExpire = datetime_diff('day', SecretExpirationDate, now())\r\n| extend Status =\r\n    case(\r\n        DaysToExpire < 0, \"Expirados\",\r\n        DaysToExpire <= 30, \"Expirando\",\r\n        DaysToExpire <= 90, \"Atencao\",\r\n        \"OK\"\r\n    )\r\n| summarize TotalSecrets = count() by Status\r\n| order by Status asc",
        "size": 0,
        "title": "Visão Geral dos Client Secrets",
        "timeContext": {
          "durationMs": 86400000
        },
        "queryType": 0,
        "resourceType": "microsoft.operationalinsights/workspaces",
        "visualization": "piechart",
        "chartSettings": {
          "seriesLabelSettings": [
            {
              "seriesName": "Expirando",
              "color": "redBright"
            },
            {
              "seriesName": "OK",
              "color": "green"
            },
            {
              "seriesName": "Atencao",
              "color": "yellow"
            },
            {
              "color": "redDark"
            }
          ]
        }
      },
      "customWidth": "40",
      "name": "query - 2",
      "styleSettings": {
        "showBorder": true
      }
    },
    {
      "type": 3,
      "content": {
        "version": "KqlItem/1.0",
        "query": "let LatestSecrets =\r\n    EntraAppSecrets_CL\r\n    | summarize arg_max(TimeGenerated, *) by ApplicationName, SecretName;\r\n\r\nLatestSecrets\r\n| extend DaysToExpire = datetime_diff('day', SecretExpirationDate, now())\r\n| summarize\r\n    Expirando = countif(DaysToExpire between (0 .. 30)),\r\n    Expirados = countif(DaysToExpire < 0)\r\nby ApplicationName\r\n| where Expirando > 0 or Expirados > 0\r\n| order by Expirados desc, Expirando desc",
        "size": 0,
        "title": "Servidores com Client Secrets Expirados ou a Expirar",
        "timeContext": {
          "durationMs": 86400000
        },
        "queryType": 0,
        "resourceType": "microsoft.operationalinsights/workspaces",
        "gridSettings": {
          "formatters": [
            {
              "columnMatch": "Expirando",
              "formatter": 18,
              "formatOptions": {
                "thresholdsOptions": "colors",
                "thresholdsGrid": [
                  {
                    "text": "{0}{1}"
                  },
                  {
                    "operator": ">",
                    "thresholdValue": "0",
                    "representation": "redBright",
                    "text": "{0}{1}"
                  },
                  {
                    "operator": "Default",
                    "thresholdValue": null,
                    "representation": "green",
                    "text": "{0}{1}"
                  }
                ]
              }
            },
            {
              "columnMatch": "Expirados",
              "formatter": 18,
              "formatOptions": {
                "thresholdsOptions": "colors",
                "thresholdsGrid": [
                  {
                    "text": "{0}{1}"
                  },
                  {
                    "operator": ">",
                    "thresholdValue": "0",
                    "representation": "redBright",
                    "text": "{0}{1}"
                  },
                  {
                    "operator": "Default",
                    "thresholdValue": null,
                    "representation": "green",
                    "text": "{0}{1}"
                  }
                ]
              }
            },
            {
              "columnMatch": "ExpiringSoon",
              "formatter": 18,
              "formatOptions": {
                "thresholdsOptions": "colors",
                "thresholdsGrid": [
                  {
                    "operator": ">",
                    "thresholdValue": "0",
                    "representation": "redBright",
                    "text": "{0}{1}"
                  },
                  {
                    "operator": "Default",
                    "thresholdValue": null,
                    "representation": "green",
                    "text": "{0}{1}"
                  }
                ]
              }
            },
            {
              "columnMatch": "Expired",
              "formatter": 18,
              "formatOptions": {
                "thresholdsOptions": "colors",
                "thresholdsGrid": [
                  {
                    "text": "{0}{1}"
                  },
                  {
                    "operator": ">",
                    "thresholdValue": "0",
                    "representation": "redBright",
                    "text": "{0}{1}"
                  },
                  {
                    "operator": "Default",
                    "thresholdValue": null,
                    "representation": "green",
                    "text": "{0}{1}"
                  }
                ]
              }
            }
          ],
          "sortBy": [
            {
              "itemKey": "ApplicationName",
              "sortOrder": 1
            }
          ]
        },
        "sortBy": [
          {
            "itemKey": "ApplicationName",
            "sortOrder": 1
          }
        ]
      },
      "customWidth": "50",
      "name": "query - 3",
      "styleSettings": {
        "showBorder": true
      }
    }
  ],
  "$schema": "https://github.com/Microsoft/Application-Insights-Workbooks/blob/master/schema/workbook.json"
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
