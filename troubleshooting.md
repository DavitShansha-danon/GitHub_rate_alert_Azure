# Troubleshooting Guide

This guide provides solutions to common issues encountered during the GitHub Rate Limit Alert setup.

---

## 🟠 No Output from Function Deployment

If you run:

```bash
func azure functionapp publish <your_function_app_name>
```

and don't see your function listed at the end:

```
Functions in <your_function_app_name>:
    GenerateGitHubJWT - [POST] https://<your_func>.azurewebsites.net/api/GenerateGitHubJWT
```

### ✅ Solution:

* Check folder structure: Ensure `index.js` is inside the correct folder (e.g., `GenerateGitHubJWT/`)
* Confirm `function.json` is present
* Verify no `main` entry in `package.json`

---

## 🛑 Key Vault Access Denied

If your Logic App fails to read the `.pem` secret:

```
The user, group or application does not have secrets get permission on key vault...
```

### ✅ Solution:

* Make sure Logic App’s **System Assigned Identity** is enabled
* Run:

```bash
az keyvault set-policy \
  --name <your_kv_name> \
  --resource-group <your_rg> \
  --object-id <logic_app_object_id> \
  --secret-permissions get list
```

---

## ❌ GitHub Token Request Fails

If the POST to GitHub installation token endpoint fails:

```
401 Unauthorized or 403 Forbidden
```

### ✅ Solution:

* Ensure the JWT from your Function is valid (check time and expiration)
* Verify correct **App ID**, **Installation ID**, and `.pem` usage
* Double-check token headers, especially:

  ```json
  "Accept": "application/vnd.github+json"
  ```

---

## 📭 Email Not Sending

Email condition is met but nothing is received:

### ✅ Solution:

* Ensure Outlook 365 connector is properly authenticated in Logic App
* Check recipient address
* Add logging steps before sending to verify condition result

---

## 🧪 Test Inputs

Want to simulate the alert condition?

* Temporarily modify the Logic App's **condition** to always return true
* Or test Azure Function manually in Postman with dummy `.pem`

---

## 🧭 Still Stuck?

* Check Logic App Run History for failed step
* Review Application Insights logs (if enabled on Function App)
* Re-deploy Function or reset Logic App connections as needed

---

Return to: [README](./README.md) or [Logic App Setup](./logic-app.md)
