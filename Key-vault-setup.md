# Azure Key Vault Setup

Securely storing your GitHub App's private key (.pem) is essential. Azure Key Vault allows your Azure services to retrieve secrets without hardcoding them.

---

## 📁 Step 1: Create a Key Vault

Use Azure CLI:

```bash
az keyvault create \
  --name <your_key_vault_name> \
  --resource-group <your_resource_group> \
  --location <your_region>
```

> ⚠️ Use lowercase for Key Vault names. They must be globally unique.

---

## 📂 Step 2: Upload `.pem` File

Assuming you saved the private key as `github-app.pem`:

```bash
az keyvault secret set \
  --vault-name <your_key_vault_name> \
  --name "github-app-pem" \
  --file ./github-app.pem
```

---

## 🚪 Step 3: Grant Access to Logic App

Enable Managed Identity on your Logic App:

1. Azure Portal → Logic App → **Identity**
2. Enable **System Assigned** Identity and Save
3. Copy the **Object ID**

Then run:

```bash
az keyvault set-policy \
  --name <your_key_vault_name> \
  --resource-group <your_resource_group> \
  --object-id <your_logic_app_object_id> \
  --secret-permissions get list
```

---

## 🖇️ Next Step

[Deploy Azure Function](./azure-function.md) to authenticate and retrieve GitHub tokens ✅
