**Error Code ID:** PostOperationsError

**Error message**

 waiting for kubeadmcontrolplane to be provisioned: default/********: timed out waiting for the condition.

**Explanation**

During the Azure Arc resource bridge deployment (ARB) on a SCVMM managed datacenter, a VM is created which serves as the ARB. There is a kubernetes cluster which runs inside this ARB VM. This error normally suggest that the kubernetes control plane is not provisioned properly with the Kubernetes Cluster.

This can happen due to multiple reasons. The cause and debugging steps included here are -

1) Please check if Gateway and DNS are properly configured with the appliance VM. To bring up the Kubernetes control plane, we create a Remote PowerShell session from one of the pods running inside the appliance VM to the VMM Server.
   - To debug this further, we can create one test Windows/Linux VM using the same IP Pool or IP range and VLAN ID with the same DNS and gateway which was used for appliance VM creation. The advantage here is that we can log in to the test VM and debug the configuration, but we can't log in to the appliance VM to debug this further. Once we log in into the test VM, we can try creating a remote PowerShell session with the VMM Server to check if that is working fine or not. Please double-check if the IP is assigned in the same subnet as the appliance VM and DNS/VLAN ID and gateway are also the same. Below is the command which we can run from within the test VM -

   **New-PSSession -ComputerName <fqdn/IP> -Authentication Negotiate -Credential 'domain\username'**

2) To rule out if this is not a DNS issue, instead of trying with FQDN for appliance deployment, please pass the role name IP or node IP in case of standalone deployment during appliance deployment. 

3) Please ensure the WinRM ports are allowed for all the IPs which are specified in the range set as part of the IP pool used for appliance VM deployment.

   - VMM IP Pool: If your SCVMM server is behind a firewall, all the IPs in this IP Pool and the Control Plane IP should be allowed to communicate through WinRM ports. The default WinRM ports are 5985 and 5986.
   - Custom IP range: Ensure that your VM network has three continuous free IP addresses. If your SCVMM server is behind a firewall, all the IPs in this IP range and the Control Plane IP should be allowed to communicate through WinRM ports. The default WinRM ports are 5985 and 5986. 

   
   To debug this further, please capture Wireshark logs and check if there are any TCP handshake issues seen from the appliance VM IP to the VMM Server, as that will help to identify if there is a firewall rule blocking the communication between the appliance VM and the VMM Server.
