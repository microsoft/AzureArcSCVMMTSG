**Error Code ID:** CreateConfigKvaCustomerError

**Error message**
  
  failed to get new provider:  SCVMM PSSession creation failed. Please check if the specified username and password are correct. Ensure VMM Server is running and reachable from the machine where the script is executed. Kindly check if the user is an Administrator on VMM Server Machine and is part of Administrator user role in the VMM Server User Role profile.   : New-PSSession : ***** Connecting to remote server ****** failed with the following  error message : Access is denied. For more information, see the about_Remote_Troubleshooting Help topic.

**Explanation**

During the Azure Arc resource bridge deployment (ARB) on a SCVMM managed datacenter, a remote PowerShell session creation request triggers from the workstation machine from which the deployment script is executed to the VMM server machine. The user will be prompted to provide a Fully-Qualified Domain Name (FQDN) which can be rolename/nodename or an IP address.

Access denied error can be encountered because of multiple reasons. Given below are some of the reasons and the recommended steps to confirm the same:

1) The user account (say domain\username) used to create the remote session is not an administrator in the VMM server machine. To validate this,
   - RDP into the VMM server machine.
   - Search for lusrmgr.msc
   - Go to Group->Administrators and check if the account used during the execution of the ARB deployment script is added here
   - Open a PowerShell admininstrator prompt from the workstation machine and run the below command

     **New-PSSession -ComputerName <fqdn/IP> -Authentication Negotiate -Credential 'domain\username'**

2) Username or password provided is incorrect. To validate this, open a PowerShell admininstrator prompt from the workstation machine and run the below command

     **New-PSSession -ComputerName <fqdn/IP> -Authentication Negotiate -Credential 'domain\username'**
