**Error Code ID:** KVAInvalidEntityCustomerError

**Error message**
  
VMM Server doesn't have write permission into the specified library share : \\\\\\\\LibraryServer\\\\LibraryShare. Please check if Import-SCLibraryPhysicalResource command and Get-SCLibraryShare command works from VMM Server.  Check VMM jobs to see if any of the above mentioned VMM jobs failed with any error. Please use a library share for which there is a write permission associated. To validate the write permission try running Import-SCLibraryPhysicalResource to copy a dummy file using the same useraccount


**Explanation**

During Azure Arc resource bridge (ARB) deployment on a SCVMM managed datacenter, a few validations are performed before the ARB VM deployment. One such validation is to validate if the the library server specified has write permissions. VMM Server can only create the ARB VM if the required VHDX/VHD file is present in the library share. For ARB VM deployment, a special customized Mariner VHDX is required and the same is downloaded during the onboarding script execution and copied to the VMM library share. Hence, the onboarding script prompts to select a library share during the onboarding script execution and on selection, validation check for the write permission on the chosen library share happens. In addition to write permission, a minimum of 10GB of free space is required in the selected library share.

If the above error message is encountered, you can validate if the libary share has the write permissions by following the below steps:
- RDP to the VMM server machine.
- Open a PowerShell admin prompt and browse to SystemDrive\ProgramData folder (for e.g C:\ProgramData)
- Create a dummy PowerShell file (for e.g test.ps1) inside the path
- Run the below cmdlets now:
   - $server = Get-SCVMMServer -ComputerName localhost
   - Import-SCLibraryPhysicalResource -SourcePath 'SystemDrvie\ProgramData\powershellfilename' -SharePath <LibrarySharePath> -VMMServer $server -OverwriteExistingFiles
     
     for e.g if LibrarySharePath was \\testserver\testshare and filename was test1.ps1 then the above command will look like
     Import-SCLibraryPhysicalResource -SourcePath 'C:\ProgramData\test1.ps1' -SharePath '\\testserver\testshare' -VMMServer $server -OverwriteExistingFiles

- Monitor the VMM jobs from the VMM console to ensure if the above Import command is working. Once it works, refresh the library share from the VMM console and verify if the test1.ps1 is present inside the library share.

- If the Import-SCLibraryPhysicalResource is stuck for a long time or if it fails, debug the error. Once you fix the same, execute the ARB onboarding script again. 

- Few other known causes of the Import command stuck could be BitsTCPPort conflict issue, or certicate issue between VMM Server and library share.

- Once this works, re-run the appliance onboarding script from the workstation machine and select the library share which was fixed by the above debugging.

- You can also file a support ticket to fix the library issue by working with Microsoft engineers. 
