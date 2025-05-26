
### 📘 Azure Custom Role with Full Schema and CLI Commands

---

### 🧩 Complete Custom Role JSON Example

Save this JSON as `custom-role.json`.

```json
{
  "Name": "Custom Storage Reader",
  "Id": "11111111-1111-1111-1111-111111111111",  // Optional, required for updates
  "IsCustom": true,
  "Description": "Can view storage accounts and read blob data",
  "Actions": [
    "Microsoft.Storage/storageAccounts/read"
  ],
  "NotActions": [],
  "DataActions": [
    "Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read"
  ],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/<your-subscription-id>"
  ]
}
```

> 💡 `Id` is optional when creating a role. Azure will assign one if omitted. Use a valid GUID when updating.

---

## 🛠️ Azure CLI Commands for Custom Roles

### ✅ Create Custom Role

```bash
az role definition create --role-definition custom-role.json
# Update Custom Role
az role definition update --role-definition custom-role.json
# List Custom Roles
az role definition list --custom-role-only true -o table
# Get Role ID
az role definition list --name "Custom Storage Reader" --query "[].{RoleName:roleName, Id:id}" -o table
# Delete a Custom Role
az role definition delete --name "Custom Storage Reader"
```



---

## 👤 Assign Role to Users, Groups, or Managed Identities

### ➕ Assign to a User (by UPN or Object ID)

```bash
az role assignment create   --assignee <user-object-id-or-upn>   --role "Custom Storage Reader"   --scope /subscriptions/<subscription-id>
# Assign to a Group
az role assignment create   --assignee-object-id <group-object-id>   --role "Custom Storage Reader"   --scope /subscriptions/<subscription-id>
#Assign to a Managed Identity
az role assignment create   --assignee-object-id <managed-identity-object-id>   --role "Custom Storage Reader"   --scope /subscriptions/<subscription-id>/resourceGroups/<resource-group>
```




---

## 🔐 Understanding Actions vs DataActions

| Type         | Used For                                  | Examples                                   |
|--------------|-------------------------------------------|--------------------------------------------|
| `Actions`    | Management plane (resource control)        | Create/Delete storage accounts, view VMs   |
| `DataActions`| Data plane (resource content access)       | Read blob data, download secrets, list messages |



```
az account show --query user.name -o tsv

```

