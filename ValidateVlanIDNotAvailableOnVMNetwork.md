**Error Code ID:** KVAInvalidEntityCustomerError

**Description**
  
    { _errorCode_: _InvalidEntityError_, _errorResponse_: _{\n\_message\_: \_The specified VLAN Id _X_ is not available on VM Network _******_\_\n}_, _errorMetadata_: { _errorCategory_: __ } }

**Explanation**

During Azure Resource Bridge Deployment for SCVMM there are some validations done before the actual appliance VM deployment.
Once such validate is to verify if the network configuration used for appliance deployment is valid or not. Once the appliance VM is created it will be assigned an IP from the StartIP and EndIP range which is prompted by CLI. If the newtork configuration specified is not correct the IP assigned from this range on appliance VM will not be reachable from workstation machine and deployment will fail. 

Below is how the user experience looks like once the azure resource bridge deployment is triggered from workstation mahcine and if there is no VMM IPPool configured as part of VMM Infra.

![alt text](VlanIDCLIFlow.png)

To summarize :

If there are IPPools present as part of VMM Server and associated with the target HostGroups/Clouds it will be used for Azure Resource Bridge deployment and in that flow user will not be prompted to provide any Vlan ID as it is already configured within the Logical Network Defination.

If there are no IPPools configured, custom IP pool experience will kick in and prompted to the user (as shown in the above image).
As part of this experience user is asked to provide VLAN ID and if the specified VLAN is not provided correctly one will endup in getting the above validate error.

**How to Get the correct VLAN ID**

Please not the ARB deployment currently only supports VMM Logical Switch and not the Standard switch.

We will first talk about how to identify the correct VLAN if VMM Logical switch is used.

i) Open VMM Console

ii) First we need to identiy the Logical Network associated with the VM Network which was selected for ARB deployment
    As we already know the VM Network name we can ***open VMM Console -> VM Network -> Double Click and open the VM Network which is used for deployment***. It will be associated with a logical network, please take a note of the same.

iii) Check the properties of Host (within the HG or associated with Cloud) which is selected for ARB deployment and identify which  Logical switch is associated with the same.
      ***Host Properties -> Virtual Switches***

iv) ***Open Fabric -> Logical Switches -> Logical Switch property (Name identified from previous step) -> Uplinks (port profile name)***
This will have few set of Logical Network Definations (network site) associated with Logical Network and marked as enabled.

v) Match and identiy the Logical Network Definations (network site) associated with the Logical Network (as idenitfied in step ii above)

vi) ***Open VMM Console -> Fabric ->  Logical Networks -> Open Logical Network (idneitfied in Step ii above) -> Network Site***

    As part of this there can be more than one Network site associated with the Logical Network. Each will have a VLAN ID associated with the same. If there is no vlan ID present it is assumed that vlan ID will be 0 which means vlan is disabled.
    
    Please consider updating the Network site in above step with correct vlan which is being planned to be used and use the same during ARB deployment.

    Incase if there is some issues/open points due to which any existing Network Site is not updated with the expected VLAN we can create a new network site and fill the vlan and subnets.
    
    As part of this flow only if we click on ADD -> Network site,  it will open text boxes to provide vlan ID and subnet. Please create the same and save the ocnfiguration.

    Once the Network site is created we need to go an update the Uplink port profile so that it allows us to use this network site for ARB deployment.

    VMM Console -> Fabric -> Port Profiles -> Open Port profile (identied as part of step iv above i.e Uplinks (port profile name))
    -> Network Configuration -> Select the newly created Network Site (Logical Network Defination)

Once we are good with the above validation we can run the below command from the workstation machine to check if the above reported error is still being reported or not.

1) Open an administrator powershell prompt in the same machine from where the appliance deployment script was executed.
2) Browse to the same folder from where the script was executed as it will have three yaml which we can use for quick validation for this error.
   The three files present in the above path will be named as (say if the RB name was testvmmrb in the onboarding script downloaded from azure portal) -
   testvmmrb-appliance.yaml , testvmmrb-infra.yaml, testvmmrb-resource.yaml 
   
   Please note the filename will change based on the RB name but the extensions -appliance.yaml, -infra.yaml, -resource.yaml will be there.

3) Run the below command -
   az arcappliance validate scvmm --configfile .\testvmmrb-appliance.yaml


If VMM Logical Switch is not used from VMM for VM network management and is being done through standard switch, please try considering Standard Switch to VMM Logical switch or try creating a new Logical Switch. Please follow the below link which captures details on how to create a VMM logical switch or covert a standard switch to a logical switch.

https://learn.microsoft.com/en-us/system-center/vmm/network-switch?view=sc-vmm-2022
