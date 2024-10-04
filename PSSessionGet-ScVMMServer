**CreateConfigKvaCustomerError**

**Description**
  
  failed to get new provider:  SCVMM PSSession creation failed. Please check if the specfied username and password are correct. Ensure VMM Server is running and reachable from the machine where the script is executed Kindly check if the user is an Administrator on VMM Server Machine and is part of Administrator user role in the VMM Server User Role profile.
  : Error in executing get-scvmmserver command on the pwsh console.Get-SCVMMServer: Unable to connect to the VMM management server localhost. The Virtual Machine Manager service on that server did not respond. (Error ID: 1602)
  Verify that Virtual Machine Manager has been installed on the server and that the Virtual Machine Manager service is running. Then try to connect again. If the problem persists, restart the Virtual Machine Manager service.


**Explanation**

During Appliance Resource Bridge Deployment for SCVMM there is a remote Powershell session creation request triggered from the workstation machine to the VMM Server machine. User is prompted to provide a FQDN, it can be rolename/nodename or an IP.


The above error can also be reported because of multiple reason -
1) There is no VMM Service running as part of the specified FQDN or IP specified to connect to VMM server.

For standalone VMM Setup we need to specific either the full fqdn of the vmm server machine or the IP. For HAVMM setup we can either specify the HAVMM role Name or active node FQDN or IP.
To debug this further one can RDP inside the VMM Server machine using the FQDN or IP and check it is a correct endpoint and VMM Service is running.

Steps to validate if VMM Service is running -
- RDP inside the VMM Server machine using the IP/FQDN which was specified for appliance VM deployment
- Open the service pane to look for what all services are running (services.msc)
- Check if SCVMMService (System Service Virtual Machine Manager) is running.

Once the service is up and running please run the below command from workstation machine to ensure the session creation is working fine and then proceed with the appliance VM deployment script.

     **New-PSSession -ComputerName <fqdn/IP> -Authentication Negotiate -Credential 'domain\username'** 

