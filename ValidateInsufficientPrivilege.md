**Error Code ID:** KVAInvalidEntityCustomerError

**Error message**
  
Insufficient privileges. The specified username for appliance VM creation is not a member of SCVMM Administrator profile. Please check if the user is added as a member of Administrator User Role profile on VMM Server.

**Explanation**

During the Azure Arc resource bridge (ARB) deployment on a SCVMM managed datacenter, there are a few validations done before the actual ARB VM deployment. Once such validation is to verify if the account used to perform the deployment is an administrator. The user account which does the ARB VM deployment needs to be an administrator on the Windows Server machine on which the VMM Service is running and also an administrator as part of VMM Server user profile. Given below are some of the reasons and the recommended steps to confirm the root cause for the above issue:

- Open VMM Console -> Settings -> User Role -> Administrator Profile
- Check if the username which was used during onboarding script execution is part of the administrator profile. The username can also be part of some domain admin and the domain admin can be added as part of the administrator profile.

Once you are good with the above validation, you can run the below command from the workstation machine to check if the above error is still encountered:

1) Open an administrator PowerShell prompt in the same machine from where the ARB VM deployment script was executed.
2) Browse to the same folder from where the onboarding script was executed as it will have three .yaml files which help with quick validations. The three .yaml files present in the path will be named as (say if the ARB name was testvmmrb in the onboarding script downloaded from Azure portal) -
   testvmmrb-appliance.yaml , testvmmrb-infra.yaml, testvmmrb-resource.yaml 
   
   Note the filename will change based on the ARB name but the extensions -appliance.yaml, -infra.yaml, -resource.yaml will remain the same.

3) Run the below command: 

   az arcappliance validate scvmm --configfile .\testvmmrb-appliance.yaml

4) Once prompted for credentials, use the username in the format (domain\username) and check if the above error is fixed.
   
5) If still not solved and the used account was a domain admin, add the the account explicitly as part of VMM server administrator profile and retry the command in Step 3 above. VMM Console -> Setting -> User Role -> Administrator Profile -> add the account directly in the administrator profile

6) If the account used was not a domain admin and it is added as part of VMM Server administrator profile directly, ensure the domain name specified is the NetBIOS name.

    For e.g 

    In VMM Administrator profile if you have added abcdef\testuser1, while ARB onboarding script execution, you should use the same abcdef\testuser1 and not anything different like abc.def\testuser1. (abc.def can still be a valid domain name and which can be used for abcdef)

7) Once the debugging is complete, re-run the ARB onboarding script with the same username and FQDN with which the debugging was performed.
