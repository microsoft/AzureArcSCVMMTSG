**Error Code ID:** PostOperationsError

**Error message**
  
 Timeout occurred due to management machine being unable to reach the appliance VM IP, <IP>. Please ensure that the requirements are met: https://aka.ms/arb-machine-reqs: dial tcp [IP:Port>]: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond.

**Explanation**

During the Azure Arc resource bridge deployment (ARB) on a SCVMM managed datacenter, a VM is created which serves as the ARB. This VM is assigned an IP from the VM Network and IP ranges/IP Pool which was input during onboarding script execution. This error mainly suggest that from the workstation machine from which the onboarding script is executed, the ARB VM IP is not reachable/pingable. This IP should be reachable as during the onboarding process we do FTP to download a test file within the ARB VM.

This reachability error can be encountered because of multiple reasons. The debugging steps included here are -

1) Ensure from VMM console that the ARB VM is deployed properly and IPs are assigned to the ARB VM from the same VM network and IP range/IP Pool which was selected/provided while executing the onboarding script. ARB VM can be identified by an unusually long name with the words "control plane" in the VM name and "Arc Appliance VM" in the VM description.
2) Try running - ping <ARB VM IP> from workstation machine.
3) If the ping is not working, use VMM console to view hardware properties of the ARB VM.
   
   VMM Console -> VM and Services -> All Hosts -> Look for ARB VM -> Properties -> Hardware Configuration -> Network Adapter
   
   Validate if the VLAN ID is configured properly or not. In case if the same is not configured properly, please change it manually to the valid VLAN ID and try pinging the ARB VM IP again.

   After changing the VLAN ID if the ARB is pingable, consider reconfiguring the VMM networking such that the uplink port profile associated with the switch is only having one Logical Network Definition. This will ensure that the ARB VM created will always be assigned a correct VLAN ID. We request you to file a support case or reach out to us through arc-vmm-feedback@microsoft.com to share your feedback on this to allow us to continually improve the product.

4) If the ARB VM is pingable, and you still encountered the above timeout error came, work with your network team to check if port 22 is unblocked. Another quick test to root-cause this issue is to create a test Linux VM using the same VM Network, and doing a SSH to the test VM IP from the workstation machine. This will help in identifying the root cause for the blocked port issue.
