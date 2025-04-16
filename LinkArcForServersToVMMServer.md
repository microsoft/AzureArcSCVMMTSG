This TSG addresses two scenarios:

1. A VM is Azure-enabled but also has the Arc agent installed via the Arc-servers path. This results in two resources per VM, and we need to remove the duplicate resources. Start from Step 1.
   - One under the VMMServer
   - One in Arc-enabled servers  

2. A VM is not Azure-enabled but has the Arc agent installed via the Arc-servers path. Start from Step 4.
   - **‘Link to SCVMM’** option is missing in the SCVMM VM Inventory view due to a portal issue.
   - **‘Link to SCVMM’** is present, but there are a lot of Arc-for-Servers resources, and an automation would be helpful.

Steps :

1. Retrieve the Resource ID of the VMMServer
2. Identify and Export Duplicate VM IDs
3. Delete duplicate VM entries from Arc-enabled SCVMM Inventory
4. (Optional) Consolidate Resources into a Single Resource Group
5. Use CLI to Link Machines (Preview Extension)

> 💡 You’ll need the **Azure CLI** to perform the steps below. Either of the following will work:
- Azure CLI installed locally
- Azure CloudShell (available in the Azure portal)

### Step 1: Retrieve the Resource ID of the VMMServer:

1. Navigate to the VMMServer resource in the Azure portal.
2. Click **‘JSON View’** on the right-hand side.
3. Copy the **Resource ID** value.

### Step 2: Identify and Export Duplicate VM IDs

If manually removing VMs via the portal is cumbersome (especially with many resources), you can automate this with Azure CLI.

**Bash:**
```bash
az scvmm vmmserver inventory-item list \
  --resource-group 'contoso-rg' \
  --vmmserver 'contoso-vmmserver' \
  --query "[?kind=='VirtualMachine'].id" \
  --output tsv > vm_ids.txt
```

**PowerShell:**
```powershell
az scvmm vmmserver inventory-item list `
  --resource-group 'contoso-rg' `
  --vmmserver 'contoso-vmmserver' `
  --query "[?kind=='VirtualMachine'].id" `
  --output tsv > vm_ids.txt
```

This command exports all VM IDs under the specified VMMServer to `vm_ids.txt`.  
Review the file and **remove any IDs** that should not be deleted.

---

### Step 3: Delete duplicate VM entries from Arc-enabled SCVMM Inventory

> [!NOTE]
> 💡 This step will **not delete the VMs from your VMMServer infrastructure**. It will **just delete the azure representations of the respective VMs.**

**Bash:**
```bash
#!/bin/bash
count=0
total=$(wc -l < vm_ids.txt)

while IFS= read -r id; do
  count=$((count + 1))
  echo "[$count/$total] Deleting VM: $id"
  az scvmm vm delete --yes --delete-machine --ids "$id"
done < vm_ids.txt
```

**PowerShell:**
```powershell
$ids = Get-Content vm_ids.txt
$total = $ids.Count
$count = 0

foreach ($id in $ids) {
  $count++
  Write-Host "[$count/$total] Deleting VM: $id"
  az scvmm vm delete --yes --delete-machine --ids $id
}
```

---

### Step 4 (Optional): Consolidate Resources into a Single Resource Group

You can perform this step in the following cases:

1. Your Arc-for-Servers resources are in a different resource group than your VMMServer and you prefer to consolidate them
2. The **‘Link to VMMServer’** option is missing in the portal. This is a known portal issue and typically occurs when the Arc for Servers resources are not in the same resource group as the VMMServer. The portal fix is in progress.

**Get the list of Arc-for-Servers resources:**
```bash
az resource list --resource-group 'contoso-rg' --query "[?type=='Microsoft.HybridCompute/machines'].id" --output tsv > arc_servers.txt
```

**Move them to the SCVMM resource group:**

**Bash:**
```bash
az resource move --destination-group 'contoso-rg' --ids $(cat arc_servers.txt)
```

**PowerShell:**
```powershell
$ids = Get-Content arc_servers.txt
az resource move --destination-group 'contoso-rg' --ids $ids
```

> After moving the resources, the ‘Link to VMMServer’ should appear in the portal.
> > If the number of resources is large, you can use the CLI to link the resources to SCVMM by using the next step.

### Step 5: Use CLI to Link Machines (Preview Extension)

You can use a CLI command available in the **preview** version of the SCVMM extension to manually link machines.

1. Install the preview CLI extension:

```sh
az extension add --yes --upgrade --source https://nascarsayan.blob.core.windows.net/get/scvmm-1.1.3-py2.py3-none-any.whl
```

2. Run the linking command:

```sh
az scvmm vm create-from-machines \
  --subscription contoso-sub \
  --scvmm-id /subscriptions/01234567-0123-0123-0123-0123456789ab/resourceGroups/contoso-rg/providers/Microsoft.ScVmm/vmmServers/contoso-vmmserver
```

This will link your Arc for Servers resources to the VMMServer as expected.

