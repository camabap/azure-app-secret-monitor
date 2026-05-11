# Azure Communication Services (Email)

## 📌 Overview

In this solution, Azure Communication Services (ACS) is used to send email notifications when client secrets are close to expiration.

⚠️ This implementation was created for lab and demonstration purposes.

In production scenarios, you may also use other available connectors in Azure Logic Apps, such as:
- Office 365 Outlook connector
- SMTP connector
- SendGrid
- Custom APIs

Azure Logic Apps provides multiple connectors that can be used for email delivery depending on governance, security, and organizational requirements.

---

## 🎯 Objective

- Enable email notifications from the solution
- Configure a domain for sending emails
- Link the domain to the Communication Service

---

## 🧱 Architecture Context

This component is used by the Logic App responsible for alerting, allowing it to send emails when secrets are nearing expiration.

---

## 🔧 Step 1 – Create Azure Communication Service

1. Go to **Azure Portal**, Search for: **Communication Services**
2. Click **Create**

3. Fill in the required fields:

| Field | Value |
|-------|-------|
| Subscription | `<Your Subscription>` |
| Resource Group | `<Your Resource Group>` |
| Resource name | `<Your ACS Name>` |
| Region | `Global` |
| Data Location | Choose based on compliance (e.g., United States) |

4. Click **Review + Create**
5. Click **Create**

✅ Azure Communication Service is now created

---

## 🔧 Step 2 – Create Email Communication Service (Domain)

After creating ACS, you need a domain to send emails.

1. In Azure Portal, search for: **Email Communication Services**
2. Click **Create**
3. Fill in:

| Field | Value |
|-------|-------|
| Subscription | `<Your Subscription>` |
| Resource Group | `<Your Resource Group>` |
| Name | `<Your Email Service Name>` |
| Data Location | Same as ACS |

4. Click **Review + Create**
5. Click **Create**

------

## 🔧 Step 3 – Add Domain

After deployment:

1. Open the **Email Communication Service**
2. Open Your Resource
3. Click **1-Click Add** in **Add a Free Azure Domain**

✅ No DNS configuration required

## 🔧 Step 4 – Link Domain to Communication Service

Now you must link the domain to the ACS resource.

1. Go to your **Communication Service**
2. Navigate to: **Email → Domains**
3. Click: **Connect domain**
4. Select your subscription, resource group, **the email domain created previously** and the verified domains
5. Click **Connect**

✅ Domain is now linked

---

## 🔎 Validation

To confirm everything is correctly configured:

- Communication Service shows **linked domain**
- Email Communication Service shows domain as **Active**
- No errors in resource provisioning

---

## 🧩 Template Reference (Sanitized)

Below is an example (sanitized) of how the Communication Service is configured:
{
"type": "Microsoft.Communication/CommunicationServices",
"name": "",
"location": "global",
"properties": {
"dataLocation": "",
"linkedDomains": [
"/domains/"
]
}
}

## 🔐 Security Considerations

- Use RBAC to restrict access to Communication Services  
- Avoid exposing connection strings in Logic Apps  
- Prefer managed identities where possible  
- Validate domain ownership (for custom domains)  

---

## ⚠️ Notes

- This implementation uses Azure-managed domain for simplicity (lab scenario)
- Production environments should consider:
  - Custom domain
  - SPF/DKIM configuration
  - Email reputation management

---

## 🔄 Alternatives

Instead of ACS Email, you can use:

- Office 365 Outlook connector (recommended for internal enterprise usage)
- SMTP relay services
- SendGrid
- Custom APIs

Choice depends on:
- Security requirements
- Governance policies
- Email delivery requirements
