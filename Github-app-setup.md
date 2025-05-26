# GitHub App Setup

This guide walks you through setting up a GitHub App to authenticate API calls via JWT. You'll need the App ID, private key (.pem), and Installation ID to proceed.

---

## 🔧 Step 1: Create a GitHub App

1. Go to your **GitHub Organization** → Settings → **Developer settings** → **GitHub Apps**
2. Click **New GitHub App**
3. Fill in the required fields:

   * **App name**: `<your_app_name>`
   * **Homepage URL**: `https://github.com/<your_org>`
   * **Webhook URL**: *leave blank*
   * **Permissions** (minimum required):

     * **Read-only** for `Rate limits`, `Repository metadata`
   * **Subscribe to events**: *none required for this use case*
4. Save the app

---

## 🔐 Step 2: Generate Credentials

1. Click **Generate a private key** and download the `.pem` file
2. Copy the **App ID** (e.g. `1299603`)
3. Install the app into your **organization** (not individual repo)
4. Get the [GitHub App Installation ID](https://stackoverflow.com/questions/74462420/where-can-we-find-github-apps-installation-id
)
---

## 🔎 Example Reference

* App ID: `<your_app_id>`
* Installation ID: `<your_installation_id>`
* PEM File: `<path_to_your_.pem>` (we'll store this in Azure Key Vault next)

---

## 🖇️ Next Step

[Configure Azure Key Vault](./Key-vault-setup.md) to securely store the `.pem` file ✅
