# Azure Function Setup (GitHub JWT)

This function generates a JWT using your GitHub App's `.pem` and calls the GitHub API to fetch rate limits.

---

## 🔧 Step 1: Initialize Project

```bash
mkdir github-jwt-func && cd github-jwt-func
func init . --worker-runtime node --language javascript
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

## 🚀 Step 4: Deploy to Azure

Make sure you've created your Azure Function App:

```bash
func azure functionapp publish <your_function_app_name>
```

You’ll get a URL like:

```
https://<your_func>.azurewebsites.net/api/GenerateGitHubJWT
```

Use this URL in the Logic App.

---

## 📌 NOTE: Missing Output After Function Deployment?

If you run the command:

```bash
func azure functionapp publish <your_function_app_name>
````

…and you don’t see this expected output:

```
Functions in func-gh-rate-limit-dev:
    GenerateGitHubJWT - [POST] https://<your_func>.azurewebsites.net/api/GenerateGitHubJWT
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

## 🖇️ Next Step

[Create Logic App](./logic-app.md) to call this function and send alerts ✅
