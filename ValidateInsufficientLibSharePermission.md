**Error Code ID:** KVAInvalidEntityCustomerError

**Error message**
  
VMM Server doesn't have write permission into the specified library share : \\\\\\\\LibraryServer\\\\LibraryShare. Please check if Import-SCLibraryPhysicalResource command and Get-SCLibraryShare command works from VMM Server.  Check VMM jobs to see if any of the above mentioned VMM jobs failed with any error. Please use a library share for which there is a write permission associated. To validate the write permission try running Import-SCLibraryPhysicalResource to copy a dummy file using the same useraccount


**Explanation**

During Azure Arc resource bridge (ARB) deployment on a SCVMM managed datacenter, there are a few validations performed before the ARB VM deployment. One such validation is to validate if the the library server specified has write permissions or not. VMM Server can only create the ARB VM if the required VHDX/VHD file is present with the library share. For ARB VM deployment we need a special customized mariner VHDX and the same is downloaded during the script execution and copied to the VMM Library share. This is why user is prompted to select a library share during the initial stage of deployment and validate check if there is a write permission or not. Also we need a minimum of 10GB of free space as part of the selected library share.

If the above error is seen one can validate is the write permissions are there as part of library share of not using the below steps -
- RDP to the vmm server machine.
- Open a powershell admin prompt and browse to SystemDrvie\ProgramData folder (for e.g C:\ProgramData)
- Create a dummy powershell file (for e.g test.ps1) inside the path
- Run the below set of cmdlets now :-
   - $server = Get-SCVMMServer -ComputerName localhost
   - Import-SCLibraryPhysicalResource -SourcePath 'SystemDrvie\ProgramData\powershellfilename' -SharePath <LibrarySharePath> -VMMServer $server -OverwriteExistingFiles
     
     for e.g if LibrarySharePath was \\testserver\testshare and filename was test1.ps1 then the above command will look like
     Import-SCLibraryPhysicalResource -SourcePath 'C:\ProgramData\test1.ps1' -SharePath '\\testserver\testshare' -VMMServer $server -OverwriteExistingFiles

- Please monitor the VMM jobs from VMM console if the above Import command worked or not. Once the same works refresh the library share from VMM console and verify if the test1.ps1 is present inside the share or not.

- If the Import-SCLibraryPhysicalResource hangs for long time or fails please debug the error and once the same is fixed please run the appliance onboarding script again. You can also file VMM support ticket to fix the library issue.
  Few known causes of the Import command hanging could be some BitsTCPPort conflict issue, or some certicate issue between VMM Server and library share.

- Once this works please re-run the appliance onbaording script from thr workstation machine and select the library share which was fixed as part of above debugging

