**CreateConfigKvaCustomerError**

**Description**
  
  failed to get new provider:  SCVMM PSSession creation failed. Please check if the specfied username and password are correct. Ensure VMM Server is running and reachable from the machine where the script is executed Kindly check if the user is an Administrator on VMM Server Machine and is part of Administrator user role in the VMM Server User Role profile.
  New-PSSession: [********] Connecting to remote server ******** failed with the following error message : MI_RESULT_FAILED For more information, see the about_Remote_Troubleshooting Help topic.

**Explanation**

During Appliance Resource Bridge Deployment for SCVMM there is a remote Powershell session creation request triggered from the workstation machine to the VMM Server machine. User is prompted to provide a FQDN, it can be rolename/nodename or an IP.


MI_RESULT_FAILED because of multiple reason -
1) The FQDN or IP specified to connect to VMM server is wrong. Ping or nslookup to the VMM Server FQDN/IP is not working.

For standalone VMM Setup we need to specific either the full fqdn of the vmm server machine or the IP. For HAVMM setup we can either specify the HAVMM role Name or active node FQDN or IP.
To debug this further one can RDP inside the VMM Server machine using the FQDN or IP and check it is a correct endpoint and VMM Service is running.

Steps to validate if VMM Service is running -
- RDP inside the VMM Server machine using the IP/FQDN which was specified for appliance VM deployment
- Open the service pane to look for what all services are running (services.msc)
- Check if SCVMMService (System Service Virtual Machine Manager) is running.

Once the service is up and running please run the below command from workstation machine to ensure the session creation is working fine and then proceed with the appliance VM deployment script.

     **New-PSSession -ComputerName <fqdn/IP> -Authentication Negotiate -Credential 'domain\username'** 

