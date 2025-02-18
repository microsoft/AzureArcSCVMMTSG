**Error Code ID:** KVAInvalidEntityCustomerError

**Error message**
  
Insufficient privileges. The specified username for appliance VM creation is not a member of SCVMM Administrator profile. Please check if the user is added as a member of Administrator User Role profile on VMM Server.

**Explanation**

During the Azure Arc resource bridge deployment (ARB) on a SCVMM managed datacenter, there are a few validations done before the actual ARB VM deployment. Once such validation is to verify if the account used to perform the deployment is an administrator. The user account which does the ARB VM deployment needs to be an administrator on the Windows Server machine on which the VMM Service is running and also an administrator as part of VMM Server user profile. 

Steps to debug the same -
- Open VMM Console -
   Setting -> User Role -> Administrator Profile
- Check is the username which was used is part of the administrator profile or not.
- The same can also be part of some domain admin and the domain admin can also be added as part of the administrator profile.

Once we are good with the above validation we can run the below command from the workstation machine to check if the above reported error is still being reported or not.

1) Open an administrator powershell prompt in the same machine from where the appliance deployment script was executed.
2) Browse to the same folder from where the script was executed as it will have three yaml which we can use for quick validation for this error.
   The three files present in the above path will be named as (say if the RB name was testvmmrb in the onboarding script downloaded from azure portal) -
   testvmmrb-appliance.yaml , testvmmrb-infra.yaml, testvmmrb-resource.yaml 
   
   Please note the filename will change based on the RB name but the extensions -appliance.yaml, -infra.yaml, -resource.yaml will be there.

3) Run the below command -
   az arcappliance validate scvmm --configfile .\testvmmrb-appliance.yaml

4) Once prompted for credentials please use the username in the format (domain\username) and check if the above error is fixed or not.
5) If not solved and the used account was a domain admin please add the the account expliciltiy as part of vmm server administrator profile and retry the command in step 3 above.
   VMM Console -> Setting -> User Role -> Administrator Profile -> add the account directly in the administrator profile

6) If the account used was not domain admin and the same is added present as part of VMM Server administrator profile and still the error is coming please ensure the domain name specified is the netbios name.

    For e.g 

    In VMM Administrator profile if we have add abcdef\testuser1. While RB creation we should use the same abcdef\testuser1 and not something like abc.def\testuser1. (where abc.def can still be a valid domain name and which can be used for abcdef)

7) Once this works please re-run the appliance onbaording script with the same username and fqdn which worked above.
