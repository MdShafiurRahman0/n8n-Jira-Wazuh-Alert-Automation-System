# Wazuh to Jira Alert Automation via n8n

This repository contains the setup guide, logic, and configuration for automating security alert ticketing. It captures high-severity alerts from Wazuh, processes them using n8n, and automatically creates actionable tickets in Jira.

## 🧠 System Logic & Workflow

1. **Trigger (Wazuh):** The Wazuh Manager detects a security event that meets a specific threshold (e.g., rule level >= 10) and triggers an integration script to send a JSON payload via HTTP POST to an n8n Webhook.
2. **Ingestion (n8n Webhook):** The n8n Webhook node receives the JSON payload containing alert details (rule description, IP, agent name, full log).
3. **Filtering & Routing (n8n Switch/IF):** n8n parses the data. You can apply logic here to ignore specific noisy rules or route different agent alerts to different Jira projects.
4. **Data Transformation (n8n Set):** n8n formats the raw Wazuh data into a clean, readable structure for the Jira ticket summary and description.
5. **Action (Jira Software):** n8n authenticates with Jira via API Token and creates a new Issue (Task/Bug/Incident) with the mapped Wazuh data.

---

## 📋 Prerequisites

* **Wazuh Manager:** Access to edit the `ossec.conf` file.
* **n8n Instance:** Running and accessible from the Wazuh Manager.
* **Jira Cloud Account:** Administrator access to generate an API token and a target Project Key (e.g., `SEC`, `IT`).

---

## 🚀 Step-by-Step Implementation

### 1. n8n Setup & Workflow Creation

If you don't have n8n running, you can quickly spin it up via Docker on your server:

```bash
# SSH into your server (e.g., ssh sh@100.119.65.50) and run:
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
