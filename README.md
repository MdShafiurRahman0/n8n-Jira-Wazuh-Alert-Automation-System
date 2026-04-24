
# Wazuh to Jira Alert Automation via n8n
---

## 🧠 System Logic & Workflow

1.  **Trigger (Wazuh):** The Wazuh Manager detects a security event (Level >= 10) and sends a JSON payload via HTTP POST to n8n.
2.  **Ingestion (n8n Webhook):** The n8n Webhook node receives the alert data.
3.  **Filtering (n8n Filter):** Filters out noise and ensures only critical alerts proceed.
4.  **Transformation (n8n Set):** Formats the raw data into a clean Jira ticket format.
5.  **Action (Jira Software):** n8n creates a new Issue in Jira with all relevant details.

---

## 📋 Prerequisites

* **Wazuh Manager:** Admin access to `ossec.conf`.
* **n8n Instance:** Running (Docker or Cloud).
* **Jira Cloud:** API Token and Project Key.

---

## 🚀 Step-by-Step Implementation

### 1. n8n Setup (Docker)
Run this on your server:
```bash
docker run -it --rm --name n8n -p 5678:5678 -v ~/.n8n:/home/node/.n8n n8nio/n8n
```

### 2. Configure n8n Workflow
1.  **Webhook Node:** Create a `POST` webhook (e.g., `/wazuh-to-jira`).
2.  **Filter Node:** Set condition `{{ $json.body.rule.level }} >= 10`.
3.  **Jira Node:** * **Project:** Your Project Key.
    * **Summary:** `Wazuh Alert: {{ $json.body.rule.description }}`
    * **Description:** `Agent: {{ $json.body.agent.name }} | Log: {{ $json.body.full_log }}`

### 3. Jira Credentials
* **Email:** Your Jira login email.
* **API Token:** Get it from [Atlassian API Tokens](https://id.atlassian.com/manage-profile/security/api-tokens).
* **Domain:** `your-domain.atlassian.net`.

### 4. Wazuh Manager Configuration
Open `/var/ossec/etc/ossec.conf` and add:

```xml
<integration>
  <name>custom-n8n</name>
  <hook_url>[http://100.119.65.50:5678/webhook/wazuh-to-jira](http://100.119.65.50:5678/webhook/wazuh-to-jira)</hook_url>
  <level>10</level>
  <alert_format>json</alert_format>
</integration>
```

**Restart Wazuh:**
```bash
sudo systemctl restart wazuh-manager
```

---

## 🧪 Testing the Integration

Run this command from your terminal to simulate an alert:

```bash
curl -X POST [http://100.119.65.50:5678/webhook-test/wazuh-to-jira](http://100.119.65.50:5678/webhook-test/wazuh-to-jira) \
     -H "Content-Type: application/json" \
     -d '{
           "rule": {"level": 12, "description": "Test Security Event"},
           "agent": {"name": "Test-Agent-01"},
           "full_log": "Simulated log for testing."
         }'
```

---

## 📌 Maintenance
* Ensure n8n IP is static.
* Adjust `<level>` in `ossec.conf` to control ticket volume.
```
