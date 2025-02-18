**Error Code ID:** PostOperationsError

**Error message**

 waiting for kubeadmcontrolplane to be provisioned: default/********: timed out waiting for the condition.

**Explanation**

During the Azure Arc resource bridge deployment (ARB) on a SCVMM managed datacenter, a VM is created which serves as the ARB. There is a kubernetes cluster which runs inside this ARB VM. This error normally suggest that the kubernetes control plane is not provisioned properly with the Kubernetes Cluster.

This error can happen due to multiple reasons. Given below are some of the reasons and the recommended steps to confirm the same:

1) Check if the Gateway and DNS are properly configured with the ARB VM. To bring up the Kubernetes control plane, we create a Remote PowerShell session from one of the pods running inside the ARB VM to the VMM Server.
   - To debug this further, you can create one test Windows/Linux VM using the same IP Pool or IP range and VLAN ID with the same DNS and gateway which was used for ARB VM creation. The advantage here is that we can log in to the test VM and debug the configuration, but we can't log in to the ARB VM to debug this further. Once you log in into the test VM, you can try creating a remote PowerShell session with the VMM Server to check if that is working fine. Check if the IP is assigned in the same subnet as the ARB VM and DNS/VLAN ID and Gateway are also the same. Below is the command which you can run from within the test VM -

          **New-PSSession -ComputerName <fqdn/IP> -Authentication Negotiate -Credential 'domain\username'** 

2) To rule out a DNS issue causing this error, instead of entering the FQDN during the onboarding script execution, input the role name IP or node IP in case of standalone deployment. 

3) Ensure the WinRM ports are allowed for all the IPs which are specified in the range set as part of the IP pool used for ARB VM deployment.

   - VMM IP Pool: If your SCVMM server is behind a firewall, all the IPs in this IP Pool and the Control Plane IP should be allowed to communicate through WinRM ports. The default WinRM ports are 5985 and 5986.
   - Custom IP range: Ensure that your VM network has three continuous free IP addresses. If your SCVMM server is behind a firewall, all the IPs in this IP range and the Control Plane IP should be allowed to communicate through WinRM ports. The default WinRM ports are 5985 and 5986. 

If the above steps are not helpful, to debug this issue further, capture Wireshark logs and check if there are any TCP handshake issues seen from the ARB VM IP to the VMM Server. That will help to identify if there is a firewall rule blocking the communication between the ARB VM and the VMM Server.
