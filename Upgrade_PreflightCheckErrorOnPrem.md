**Error Code ID:** PreflightCheckErrorOnPrem

**Error message**

 During the appliance upgrade, a series of pre-flight validations take place. Any errors at this stage indicate missing prerequisites. This document addresses the following two pre-flight validation errors encountered during the ARB upgrade. - 

 - Upgrade Operation Failed with error: The specified network settings are incorrect: Control Plane Endpoint IP is invalid: Control Plane Endpoint IP is overlapping with the IP address pool

 - Upgrade Operation Failed with error: The specified network settings are incorrect: Gateway IP is invalid: Gateway IP is overlapping with the IP address pool

**Explanation**

- The control plane IP used during the Day 0 ARB deployment is within the IP pool range. The IP pool for deployment is defined by a start and end IP range. For a valid deployment, the control plane IP should not fall within this range. The prerequisite for ARB deployment specifies that the control plane IP must be in the same subnet as the IP pool but should not be part of the IP pool range.

- The gateway IP used for the Day 0 ARB deployment overlaps with the IP pool range. The prerequisite for ARB deployment states that the gateway IP should not be within the start and end IP range of the IP pool.

If the customer upgrade fails due to any of the above errors, the only recovery option is to perform Disaster Recovery (DR) process. Disaster Recovery (DR) involves deleting the existing ARB resource, recreating a new ARB with the same configuration, and remapping the SCVMM resources.

The following document provides a detailed guide on performing a Disaster Recovery (DR).

https://learn.microsoft.com/en-us/azure/azure-arc/system-center-virtual-machine-manager/disaster-recovery

Disaster recovery is essentialy deletion of older ARB resource and recreation of the a new ARB with the same configuration and remapping of the SCVMM resources.