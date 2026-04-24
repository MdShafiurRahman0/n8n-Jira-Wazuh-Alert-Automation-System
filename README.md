```markdown
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
```

**Build the Workflow in the n8n UI:**
1. Add a **Webhook Node**: Set the Method to `POST` and note the Test/Production URL.
2. Add an **IF / Filter Node**: Set the condition to only pass if `{{ $json.body.rule.level }}` is `>= 10`.
3. Add a **Jira Software Node**: 
   * **Resource:** Issue
   * **Operation:** Create
   * **Project:** Select your Jira project.
   * **Issue Type:** Task or Bug.
   * **Summary:** `Wazuh Alert: {{ $json.body.rule.description }}`
   * **Description:** Add relevant details like Agent Name, Rule ID, and full log data.

### 2. Configure Jira Credentials in n8n

1. Go to your [Atlassian Account Security settings](https://id.atlassian.com/manage-profile/security/api-tokens).
2. Click **Create API token** and copy it.
3. In n8n, open your Jira node, create a new credential:
   * **Email:** Your Jira login email.
   * **API Token:** Paste the generated token.
   * **Domain:** `your-domain.atlassian.net`.

### 3. Wazuh Manager Configuration

We need to tell Wazuh to forward high-level alerts to the n8n webhook. 

Open the Wazuh configuration file:
```bash
sudo nano /var/ossec/etc/ossec.conf
```

Add the following `<integration>` block anywhere inside the `<ossec_config>` section. Replace the `<hook_url>` with your actual n8n webhook endpoint:

```xml
  <integration>
    <name>custom-n8n</name>
    <hook_url>[http://100.119.65.50:5678/webhook/wazuh-to-jira](http://100.119.65.50:5678/webhook/wazuh-to-jira)</hook_url>
    <level>10</level>
    <alert_format>json</alert_format>
  </integration>
```
*(Note: The `<level>` tag ensures only alerts of level 10 or higher are sent to n8n, saving bandwidth and preventing ticket spam).*

Restart the Wazuh Manager to apply the changes:
```bash
sudo systemctl restart wazuh-manager
```

---

## 🧪 Testing the Integration

1. In n8n, click **Listen for Test Event** on your Webhook node.
2. Trigger a high-level alert on one of your Wazuh agents (e.g., multiple failed SSH logins, or manually triggering a rule).
3. Alternatively, use `curl` from the Wazuh manager to simulate an alert being sent to n8n:

```bash
curl -X POST [http://100.119.65.50:5678/webhook-test/wazuh-to-jira](http://100.119.65.50:5678/webhook-test/wazuh-to-jira) \
     -H "Content-Type: application/json" \
     -d '{
           "rule": {
             "level": 12,
             "description": "Test Critical Security Event",
             "id": "100002"
           },
           "agent": {
             "name": "Test-Agent-01",
             "ip": "192.168.1.100"
           },
           "full_log": "This is a simulated log entry for testing Jira integration."
         }'
```

4. Check n8n to ensure the workflow executed successfully.
5. Check your Jira board to confirm the new ticket has been created with the correct mapping!

## 📌 Maintenance Notes

* Always ensure the n8n server has a static IP or resolvable DNS name so Wazuh doesn't lose connection.
* Review your Wazuh rule levels periodically. If Jira gets too noisy, increase the `<level>` threshold in `ossec.conf` to 12 or 14.
```
