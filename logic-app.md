# Logic App Setup

This Logic App orchestrates the GitHub rate limit check by calling the Azure Function, authenticating, and sending an email if the API usage is near the limit.

---

## 🚪 Step 1: Create Logic App (Standard)

In Azure Portal:

* Resource: **Logic App (Standard)**
* Plan: **Workflow Standard**
* SKU: `WS1`
* Add a blank **Stateful Workflow** (e.g. `check-rate-limit`)

---

## ⏰ Step 2: Add Recurrence Trigger

* Trigger: **Recurrence**
* Every 10 minutes

---

## 🔑 Step 3: Get .pem Secret from Key Vault

**Action**: Get Secret (Key Vault)

* Name: `Get_secret`
* Secret Name: `github-app-pem`

> This fetches your GitHub App's private key securely

---

## 🛠️ Step 4: Call Azure Function

**Action**: HTTP (POST)

* URL: `https://<your_func>.azurewebsites.net/api/GenerateGitHubJWT`
* Headers: `{ "Content-Type": "application/json" }`
* Body:

```json
{
  "appId": "<your_app_id>",
  "pem": "@{body('Get_secret')?['value']}"
}
```

> Sends `.pem` and App ID to your Function to generate a JWT token

---

## 📡 Step 5: Request GitHub Token

**Action**: HTTP (POST)

* URL: `https://api.github.com/app/installations/<your_installation_id>/access_tokens`
* Headers:

```json
{
  "Authorization": "Bearer @{body('Call_Azure_Function')?['token']}",
  "Accept": "application/vnd.github+json",
  "X-GitHub-Api-Version": "2022-11-28"
}
```

---

## 🔍 Step 6: Check GitHub Rate Limits

**Action**: HTTP (GET)

* URL: `https://api.github.com/rate_limit`
* Headers:

```json
{
  "Authorization": "token @{body('HTTP_POST_GitHub_Token')?['token']}",
  "User-Agent": "logic-app-monitor"
}
```

---

## 🪠 Step 7: Parse JSON Response

**Action**: Parse JSON

* Content: `body('HTTP_Check_GitHub_Rate_Limits')`
* Schema:

```json
{
  "type": "object",
  "properties": {
    "rate": {
      "type": "object",
      "properties": {
        "limit": { "type": "integer" },
        "remaining": { "type": "integer" },
        "reset": { "type": "integer" }
      }
    }
  }
}
```

---

## ⚖️ Step 8: Add Condition & Send Email

**Condition**:

```json
{
  "and": [
    { "less": ["@body('Parse_JSON')?['rate']?['remaining']", 200] },
    { "not": { "equals": ["@body('Parse_JSON')?['rate']?['remaining']", 0] } }
  ]
}
```

If true, send email:

* Action: **Office 365 Outlook - Send an email (V2)**
* Subject: `⚠️ GitHub API Rate Limit Alert – Remaining Calls Low`
* Body:

```html
Hi team,<br><br>
🚨 <strong>GitHub API rate limit is running low.</strong><br><br>
📊 <strong>Details:</strong><br>
• <b>Remaining:</b> @{body('Parse_JSON')?['rate']?['remaining']}<br>
• <b>Limit:</b> @{body('Parse_JSON')?['rate']?['limit']}<br>
• <b>Reset Time (Unix):</b> @{body('Parse_JSON')?['rate']?['reset']}<br><br>
Please check automation jobs or API clients to avoid disruption.<br><br>
— GitHub Monitor (Logic App)
```

---

## 🖇️ Next Step

(Optional) [Troubleshooting Tips](./troubleshooting.md) for common issues ✅
