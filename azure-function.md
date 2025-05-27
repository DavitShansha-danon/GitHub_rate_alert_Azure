# Azure Function Setup (GitHub JWT)

This function generates a JWT using your GitHub App's `.pem` and calls the GitHub API to fetch rate limits.

---

## ☁️ Step 0: Create Azure Function App via CLI

Before deploying, create your Azure Function App infrastructure using the Azure CLI:

```bash
# 1. Create a resource group (if needed)
az group create --name <your-resource-group> --location <your-location>

# 2. Create a storage account (required by Function App)
az storage account create \
  --name <your-storage-account> \
  --location <your-location> \
  --resource-group <your-resource-group> \
  --sku Standard_LRS

# 3. Create the Function App
az functionapp create \
  --name <your-function-app-name> \
  --resource-group <your-resource-group> \
  --consumption-plan-location <your-location> \
  --runtime node \
  --runtime-version 20 \
  --functions-version 4 \
  --storage-account <your-storage-account>
````

---

## 🔧 Step 1: Initialize Project

```bash
mkdir github-jwt-func && cd github-jwt-func
func init . --worker-runtime node --language javascript
```

---

💡 **Note:** If a `src/` folder is generated during `func init`, you can safely delete it to keep your structure clean and avoid conflicts with your custom function folder layout (`GenerateGitHubJWT/`).
This helps prevent confusion during local runs or deployment.

```bash
rm -rf src/
```

---

## 💡 Step 2: Create the Function

```bash
mkdir GenerateGitHubJWT
```

Inside `GenerateGitHubJWT/`, add:

### `index.js`

```js
const jwt = require('jsonwebtoken');

module.exports = async function (context, req) {
    const now = Math.floor(Date.now() / 1000);
    const appId = req.body.appId;
    const pem = req.body.pem;

    if (!appId || !pem) {
        context.res = { status: 400, body: { error: "Missing appId or pem" } };
        return;
    }

    const cleanPem = pem.replace(/\\n/g, '\n');
    const payload = { iat: now, exp: now + 600, iss: appId };

    try {
        const token = jwt.sign(payload, cleanPem, { algorithm: 'RS256' });
        context.res = { status: 200, headers: { "Content-Type": "application/json" }, body: { token } };
    } catch (err) {
        context.res = { status: 500, body: { error: err.message } };
    }
};
```

### `function.json`

```json
{
  "bindings": [
    { "authLevel": "anonymous", "type": "httpTrigger", "direction": "in", "name": "req", "methods": ["post"] },
    { "type": "http", "direction": "out", "name": "res" }
  ]
}
```

---

## 📆 Step 3: Install Dependencies

```bash
npm init -y
npm install jsonwebtoken
```

Edit `package.json` to remove the `main` line and keep only:

```json
"dependencies": {
  "jsonwebtoken": "^9.0.2"
}
```

---

## 📁 Approximate Final Folder Structure

```bash
github-jwt-func/
├── GenerateGitHubJWT/
│   ├── index.js
│   └── function.json
├── node_modules/
├── package.json
├── host.json
├── local.settings.json
```

---

## 🧪 (Optional) Step 3.5: Local Test

You can run the function locally before deploying:

```bash
func start
```

Then send a POST request to:

```
http://localhost:7071/api/GenerateGitHubJWT
```

Make sure your request body includes:

```json
{
  "appId": "<your-app-id>",
  "pem": "<your-pem-key>"
}
```

---

## 🚀 Step 4: Deploy to Azure

Deploy your function to the Azure Function App:

```bash
func azure functionapp publish <your-function-app-name>
```

You’ll get a URL like:

```
https://<your-function-app-name>.azurewebsites.net/api/GenerateGitHubJWT
```

Use this URL in your Logic App or test it with Postman.

---

## 📌 NOTE: Missing Output After Function Deployment?

If you run the command:

```bash
func azure functionapp publish <your-function-app-name>
```

…and you don’t see this expected output:

```
Functions in <your-function-app-name>:
    GenerateGitHubJWT - [POST] https://<your-function-app-name>.azurewebsites.net/api/GenerateGitHubJWT
```

👉 **Don’t worry.** This is normal if you haven’t yet connected your Function App to an input trigger.

### Why?

The function expects a request body containing:

* `"appId"` (GitHub App ID)
* `"pem"` (GitHub private key string)

Until you create a Logic App (or Postman test) that calls the function and provides those inputs, it won’t return anything meaningful during deployment.

### ✅ Solution

1. Create a Logic App (see next steps)
2. In that Logic App, define the `appId` and read the `.pem` from Key Vault
3. Once the Logic App is working, you can re-run the Function App manually or let the Logic App trigger it automatically.

---

## 🖇️ Next Step

[Create Logic App](./logic-app.md) to call this function and send alerts ✅
