# azure-app-secret-monitor
Azure solution to monitor client secrets expiration and send alerts

# Azure App Registration Secret Expiration Monitor

## 📌 Overview

This solution provides an end-to-end approach to monitor Azure AD Application (App Registration) client secrets expiration and proactively notify stakeholders before expiration occurs.

It combines Azure-native services to:
- Continuously scan App Registrations
- Identify client secrets nearing expiration
- Send proactive email alerts
- Store inventory data for monitoring and reporting
- Provide visualization through an Azure Workbook

---

## 🎯 Problem Statement

Expired client secrets can lead to:
- Application outages
- Authentication failures
- Security incidents
- Lack of governance visibility

This solution helps organizations proactively manage secret lifecycle and improve operational resilience.

---

## 🏗️ Architecture (High-Level)

The solution is composed of the following components:

1. **Azure AD App Registration**
   - Used to authenticate and retrieve application metadata
   - Requires permissions to read App Registrations

2. **Azure Communication Services (Email)**
   - Sends notification emails for expiring secrets

3. **Logic App – Secret Expiration Alert**
   - Retrieves all App Registrations
   - Iterates through client secrets
   - Identifies secrets expiring within a defined threshold (e.g., 30 days)
   - Sends email alerts

4. **Logic App – Secret Inventory Collection**
   - Retrieves App Registrations and secret metadata
   - Sends structured data to Log Analytics

5. **Log Analytics Workspace**
   - Stores custom data about applications and secrets
   - Uses custom table ingestion

6. **Data Collection Endpoint (DCE) & Data Collection Rule (DCR)**
   - Used to ingest custom data into Log Analytics

7. **Azure Workbook**
   - Provides visualization of:
     - Expiring secrets
     - Secret distribution
     - Application inventory

---

## 🔄 End-to-End Flow

1. Logic App queries Azure AD for App Registrations
2. Client secrets are enumerated
3. Expiration date is evaluated
4. If expiration < defined threshold:
   - Email notification is sent
5. All application + secret data is stored in Log Analytics
6. Workbook reads data and presents insights

---

## 📂 Repository Structure
