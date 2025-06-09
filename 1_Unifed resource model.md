This troubleshooting guide addresses two scenarios:

Scenario 1) A VM is Azure-enabled through SCVMM Azure VM Inventory but also has the Arc agent installed via one of the Arc-enabled servers methods. This results in two Azure resources for each of the impacted VMs under Azure Arc -> Machines tab. Start from Step 1 below to remove the duplicate resources and consolidate them into a single resource with the full capabilities of Azure Arc.

Scenario 2) A VM is not Azure-enabled but has the Arc agent installed via one of the Arc-enabled servers methods. You may encounter one of the two experiences: 
   - **‘Link to SCVMM’** option is missing in the SCVMM VM Inventory view due to a portal issue.
   - **‘Link to SCVMM’** is present, but there are a lot of Arc-for-Servers resources, and an automation would be helpful.

Start from Step 4 (optional) or Step 5 below to get the full range of Azure Arc capabilities for your SCVMM VMs.

Steps:

1. Retrieve the Resource ID of the VMMServer
2. Identify and Export Duplicate VM IDs
3. Remove the Azure representation of duplicate VM entries from Arc-enabled SCVMM Inventory
4. (Optional) Consolidate Resources into a Single Resource Group
5. Use CLI to Link Machines (Preview Extension)

> 💡 You’ll need the **Azure CLI** to perform the steps below. Either of the following will work:
- Azure CLI installed locally
- Azure CloudShell (available in the Azure portal)

### Step 1: Retrieve the Resource ID of the VMMServer:

1. Navigate to the VMMServer resource in the Azure portal.
2. Click **‘JSON View’** on the right-hand side.
3. Copy the **Resource ID** value.

### Step 2: Identify and export duplicate VM ARM IDs

Duplicate VM ARM IDs refer to the Azure representation of the VMs for which the Azure resource doesn't recognize the Arc agent being already installed. These are typically found under the VM Inventory view of your SCVMM server. If manually removing duplicate VMs via the portal is cumbersome (especially with many resources), you can automate this with Azure CLI.

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
Review the file and **remove any IDs** that should not be deleted. **VMs which don't have duplicates need not be in this list**

---

### Step 3: Remove the Azure respresentation of duplicate VM entries from Arc-enabled SCVMM Inventory

> [!NOTE]
> 💡 This step will **not delete the VMs from your VMMServer infrastructure in your on-premises datacenter**. It will **just delete the Azure (ARM) representations of the respective VMs.**

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

### Step 4 (Optional): Consolidate machines into a single Resource Group

> [!NOTE]
> 💡If you prefer to organize your resources into multiple subscriptions or resource groups, you can skip this step. This step is optional and not performing this step will not cause any functionality impact.

You can perform this step in the following cases:

1. Your Arc enabled servers resources are in a different resource group than your SCVMM server and you prefer to consolidate them. 
2. The **‘Link to SCVMM’** option is missing in the portal. This is a known portal issue and typically occurs when the Arc enabled servers resources are not in the same subscription and resource group as the SCVMM server. The portal fix for this issue is in progress. 

**Get the list of Arc enabled servers resources:**
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

> After moving the resources, the ‘Link to SCVMM’ should appear in the portal under 'Virtual hardware management' column against each VM which already has the Arc agent installed.
> > If the number of resources is large, you can use the CLI to link the resources to SCVMM by using the next step.

### Step 5: Use Azure CLI command to Link Machines

You can use an Azure CLI command available to manually link machines. The user executing the below CLI commands should have **Azure Arc SCVMM VM Contributor** or **Azure Arc SCVMM VM Administrator** built-in Azure roles assigned. Alternatively, the user can have a custom Azure role with the permissions to perform read and write operations in SCVMM VM resources and Hybrid Compute machines. 

1. Install the latest version of Azure CLI extension. To find your installed version and see if you need to update, run: 

```sh
az version
```

2. Run the linking command:

```sh
az scvmm vm create-from-machines --subscription contoso-sub --scvmm-id /subscriptions/01234567-0123-0123-0123-0123456789ab/resourceGroups/contoso-rg/providers/Microsoft.ScVmm/vmmServers/contoso-vmmserver
```

This will link your Arc enabled servers resources to the SCVMM server as expected.

