**Error Code ID:** KVAInvalidEntityCustomerError

**Description**
  
The specified VLAN Id _X_ is not available on VM Network _******_

**Explanation**

During Azure Arc resource bridge (ARB) deployment on a SCVMM managed datacenter, there are a few validations done before the ARB VM deployment. Once such validation is for the network configuration used during ARB deployment script execution. Once the ARB VM is created, it will be assigned an IP from the StartIP and EndIP range which is prompted by CLI during the onboarding script execution. If the network configuration specified is not correct, the IP assigned from this range on ARB VM will not be reachable from workstation machine and the ARB deployment will fail. 

Below is how the user experience looks like once the ARB deployment is triggered from workstation machine and if there is no VMM IPPool configured as part of VMM Infrastructure chosen as the deployment target for the ARB during the script execution:

![alt text](VlanIDCLIFlow.png)

To summarize :

If there are IPPools present as part of VMM Server and associated with the chosen HostGroup/Cloud, it will be used for ARB deployment and the user will not be prompted to provide any VLAN ID as it is already configured within the Logical Network definition.

If there are no IPPools associated with the chosen HostGroup/Cloud, custom IP pool input experience will kick in and custom IP inputs will be prompted to the user (as shown in the above image). As part of this experience, the user is asked to provide VLAN ID and if the specified VLAN ID is incorrect, one will end up in getting the above validate error. Below are a few ways to identify the correct VLAN ID:

**If VMM Logical switch is used.**

i) Open VMM Console

ii) Identiy the Logical Network associated with the VM Network which was selected during ARB deployment. ***Open VMM Console -> VM Network -> Double Click and open the VM Network which was used during deployment***. It will be associated with a logical network. Take a note of the same.

iii) Check the properties of Host (within the HG or associated with Cloud which was selected during ARB deployment) and identify which Logical Switch is associated with the same. ***Host Properties -> Virtual Switches***

iv) ***Open Fabric -> Logical Switches -> Logical Switch property (Name identified from previous step) -> Uplinks (port profile name)***. This will have a few Logical Network Definitions (Network sites) associated with Logical Network and marked as enabled. Match and identiy the Logical Network Definitions (Network Site) associated with the Logical Network (as identified in step (ii) above)

v) ***Open VMM Console -> Fabric ->  Logical Networks -> Open Logical Network (identified in Step (ii) above) -> Network Site***. There can be more than one Network site associated with the Logical Network. Each will have a VLAN ID associated with the same. If there is no VLAN ID present, it is assumed that VLAN ID will be 0 which means VLAN is disabled. Consider updating the Network site in the above step with the correct VLAN ID which is being planned to be used during ARB deployment and use the same during ARB deployment. 

vi) If there are some issues/open points due to which any existing Network Site is not updated with the expected VLAN ID, you can create a new network site and fill the VLAN ID and subnets. As part of this flow, only if you click on Add -> Network site, it will open text boxes to provide VLAN ID and subnet. Save the configuration. Once the Network site is created, you need to update the uplink port profile so that it allows us to use this network site for ARB deployment. ***VMM Console -> Fabric -> Port Profiles -> Open Port profile (identied as part of step (iv) above i.e Uplinks (port profile name)) -> Network Configuration -> Select the newly created Network Site (Logical Network Definition)***


**If Standard switch is used.**
   
i) Open VMM Console

ii) Identiy the Logical Network associated with the VM Network which was selected during ARB deployment. ***Open VMM Console -> VM Network -> Double Click and open the VM Network which is used for deployment***. It will be associated with a logical network, take a note of the same(say LN1).

iii) Check the properties of Host (within the HG or associated with Cloud which was selected during ARB deployment) to identify the Standard Switch associated with the logical network LN1. Note down its "Network Adapter". ***Host Properties -> Virtual Switches***
      
iv) Go to ***Host Properties -> Hardware -> Network adapters -> Network Adapter identified in step (iii) -> Logical Network connectivity.*** Check for "Checked" VLANID-subnet pairs for the logical network LN1 and take note of the appropriate one.
  
  Case 1 - If no VLANID-Subnet pairs are "checked", check the once you want to use.
  Case 2 - If no VLANID-Subnet pairs are present for the logical network LN1, go to **Fabric -> Networking -> Logical Networks -> Logical Network LN1 -> Network Site** and create one. This would then start reflecting back in the Host properties and can be used.

Once you are good with the above validation, you can run the below command from the workstation machine to check if the VLAN ID error is still being encountered.

1) Open an administrator PowerShell prompt in the same machine from where the ARB VM deployment script was executed.
2) Browse to the same folder from where the onboarding script was executed as it will have three .yaml files which help with quick validations. The three .yaml files present in the path will be named as (say if the ARB name was testvmmrb in the onboarding script downloaded from Azure portal) -
   testvmmrb-appliance.yaml, testvmmrb-infra.yaml, testvmmrb-resource.yaml. Note the filename will change based on the ARB name but the extensions -appliance.yaml, -infra.yaml, -resource.yaml will remain the same.

3) Run the below command:
   az arcappliance validate scvmm --configfile .\testvmmrb-appliance.yaml

If VMM Logical Switch is not used from VMM for VM network management and is being done through Standard Switch, you can convert the Standard Switch to VMM Logical Switch or try creating a new Logical Switch. Follow this [article](https://learn.microsoft.com/en-us/system-center/vmm/network-switch?view=sc-vmm-2025) which has details on how to create a VMM logical switch or convert a standard switch to a logical switch.
