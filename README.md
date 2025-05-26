# GitHub Rate Limit Monitor — Azure Logic App + Function

> Automatically monitor GitHub API usage and receive alerts when rate limits are running low.

---

## 🚀 What is this?

This project is a production-grade proof of concept (POC) that tracks your GitHub API rate limits and notifies your team (via email) when they fall below a specified threshold (e.g., 200 remaining calls).

It combines a **GitHub App** (for secure JWT auth), an **Azure Function** (to retrieve access tokens and API data), and a **Logic App** (for orchestration and notification).

---

## 💡 Why You Need This

GitHub imposes rate limits on its API usage, which can disrupt workflows if exceeded. By proactively monitoring the remaining rate limit, you can:

* Avoid CI/CD failures
* Maintain data pipeline integrity
* Alert your team early and mitigate downtime

---

## 🔧 Prerequisites

To follow this setup, you will need:

* An **Azure Subscription** with permissions to create Logic Apps and Function Apps
* A **GitHub Organization** or repo where you can create a **GitHub App**
* **Azure CLI** and **Visual Studio Code** with Azure Functions extension
* **Azure Key Vault** for securely storing secrets like `.pem` keys

---

## 💪 Setup Overview

Follow these steps in order:

1. **[Create GitHub App](./github-app-setup.md)**
   Set up an App with minimal permissions and download the `.pem` file

2. **[Configure Azure Key Vault](./key-vault-setup.md)**
   Store your `.pem` file securely and allow access to Logic App

3. **[Deploy Azure Function](./azure-function.md)**
   This function will use the `.pem` to generate a JWT and fetch GitHub tokens

4. **[Create Logic App](./logic-app.md)**
   Orchestrates the process and sends alerts via email

5. **(Optional) [Troubleshooting Tips](./troubleshooting.md)**
   Debug common issues, check deployment results, test integrations

---

## 📌 Additional Notes

* This is ideal for DevOps teams, internal platform squads, and automation-focused orgs
* Supports extensions like: Microsoft Teams alerts, custom alert thresholds, Log Analytics integration

---

## 🐽 Useful Links

* [Install a GitHub App](https://docs.github.com/en/apps/using-github-apps/installing-your-own-github-app)
* [GitHub Rate Limit Docs](https://docs.github.com/en/rest/rate-limit)
* [Azure Logic Apps Overview](https://learn.microsoft.com/en-us/azure/logic-apps/)
* [Azure Function App Docs](https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview)

---

Ready? Start with [GitHub App Setup](./Github-app-setup.md) ✅
