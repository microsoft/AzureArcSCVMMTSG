# ArcEnabledScvmmTSG
Self-Debugging Guide for Appliance onboarding failure for SCVMM provider
This document will serve as a purpose to debug the errors which a user faces while Azure Resource Bridge onboarding for SCVMM provider.

Below is the link for ARB doc - 
https://learn.microsoft.com/en-us/azure/azure-arc/system-center-virtual-machine-manager/overview

To contribute to this repo please raise a PR in the below format -

Error Code
Error Description as reported in command failure
Error Code Explanation with Recommendation for diagnosability


For e.g

**CreateConfigKvaCustomerError**

**Description**
  
  failed to get new provider:  SCVMM PSSession creation failed. Please check if the specfied username and password are correct. Ensure VMM Server is running and reachable from the machine where the script is executed. Kindly check if the user is an Administrator on VMM Server Machine and is part of Administrator user role in the VMM Server User Role profile.   : New-PSSession : ***** Connecting to remote server ****** failed with the following  error message : Access is denied. For more information, see the about_Remote_Troubleshooting Help topic.

**Explanation**

During Appliance Resource Bridge Deployment for SCVMM there is a remote Powershell session creation request triggered from the workstation machine to the VMM Server machine. User is prompted to provide a FQDN, it can be rolename/nodename or an IP.

Access Denied error can be seen because of multiple reason -
1) The user account (say domain\username) used to create the remote session is not an adminitrator in the vmm server machine.
   - RDP into the VMM Server machine.
   - Search for lusrmgr.msc
   - Goto Group->Adminitrators and check if the account used is added over here or not
   - Open a powershell admin prompt from the workstation machine and run the below command to debug for any further issues

     **New-PSSession -ComputerName <fqdn/IP> -Authentication Negotiate -Credential 'domain\username'**




