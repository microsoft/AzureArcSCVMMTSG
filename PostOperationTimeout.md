**Error Code ID:** PostOperationsError

**Error message**
  
 Timeout occurred due to management machine being unable to reach the appliance VM IP, <IP>. Please ensure that the requirements are met: https://aka.ms/arb-machine-reqs: dial tcp [IP:Port>]: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond.

**Explanation**

During the Azure Arc resource bridge deployment (ARB) on a SCVMM managed datacenter, a VM is created which serves as the ARB. This VM is assigned an IP from the VM Network and IP ranges/IP Pool which was input during onboarding script execution. This error mainly suggest that from the workstation machine from which the onboarding script is executed, the ARB VM IP is not reachable/pingable. This IP should be reachable as during the onboarding process we do FTP to download a test file within the ARB VM.

This reachability error can be encountered because of multiple reasons. The debugging steps included here are -

1) Ensure from VMM Server that the appliance VM is deployed properly and IP's are assigned to the VM from the same vmnetwork and IP ranged/IP Pool which was selected/provided during the script execution creation config phase.
2) Try running - ping <appliance VM IP> from workstation machine.
3) If the ping is not working use VMM console to view Hardware properties of the appliance VM.
   
   VMM Console -> VM and Services -> All Hosts -> Look for appliance VM -> Properties -> Hardware Configuration -> Network Adapter
   
   Validate if the VLAN ID is configured properly or not.
   Incase if the same is not configured properly please change it manually to the valid VLAN ID and retry if the ping is working or not.

   Considering after changing the VLAN ID if the ping is working the workaround to solve this will be to reconfigure the vmm networking such the uplink port profile associated with the switch is only having one Logical Network Definition such the appliance VM created will always be assigned a correct VLAN ID. Also please file a support case so that dev team can work out to improve the experience in subsequent release.

5) Consider if the ping to appliance VM IP was already working but still the above timeout error came please work with your network team to check if port 22 is blocked or not.
   Another quick test to check this will be to create a test linux VM using the same vm Network and try doing ssh to the test VM IP from the workstation machine. This will help in identifying more on the blocked port issue.
