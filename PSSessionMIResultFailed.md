**CreateConfigKvaCustomerError**

**Error message**
  
  failed to get new provider:  SCVMM PSSession creation failed. Please check if the specfied username and password are correct. Ensure VMM Server is running and reachable from the machine where the script is executed Kindly check if the user is an Administrator on VMM Server Machine and is part of Administrator user role in the VMM Server User Role profile.
  New-PSSession: [********] Connecting to remote server ******** failed with the following error message : MI_RESULT_FAILED For more information, see the about_Remote_Troubleshooting Help topic.

**Explanation**

During Azure Arc resource bridge (ARB) deployment on a SCVMM managed datacenter, a remote PowerShell session creation request triggers from the workstation machine from which the deployment script is executed to the VMM server machine. The user will be prompted to provide a Fully-Qualified Domain Name (FQDN) which can be rolename/nodename or an IP address.

MI_RESULT_FAILED can be encountered because of multiple reasons. Given below is one of the reasons and the recommended steps to confirm the same:

1) The FQDN or IP address specified to connect to the VMM server is incorrect. Ping or nslookup to the VMM Server FQDN/IP is not going through.

For a standalone VMM Setup we need to specific either the full FQDN of the VMM server machine or the IP address. For a highly available (HA) VMM setup, we can either specify the HAVMM role name or the active node's FQDN or IP address. To validate this,

- RDP inside the VMM server machine using the FQDN or the IP address which was specified for ARB VM deployment
- Open the service pane to check the running services (services.msc)
- Check if SCVMMService (System Service Virtual Machine Manager) is running.

Once the service is up and running, run the below command from the workstation machine to ensure the creation of a PowerShell session and then proceed with the execution of the ARB VM deployment script.

     **New-PSSession -ComputerName <fqdn/IP> -Authentication Negotiate -Credential 'domain\username'** 

