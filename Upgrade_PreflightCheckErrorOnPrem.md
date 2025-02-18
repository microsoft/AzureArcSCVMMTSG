**Error Code ID:** PreflightCheckErrorOnPrem

**Error message**

 During the Azure Arc resource bridge (ARB) upgrade, a series of pre-flight validations take place. Any errors at this stage indicate missing pre-requisites. This document addresses the following two pre-flight validation errors encountered during the ARB upgrade: 

 - Upgrade Operation Failed with error: The specified network settings are incorrect: Control Plane Endpoint IP is invalid: Control Plane Endpoint IP is overlapping with the IP address pool

 - Upgrade Operation Failed with error: The specified network settings are incorrect: Gateway IP is invalid: Gateway IP is overlapping with the IP address pool

**Explanation**

Given below are some of the reasons for the above errors and the recommended steps to resolve the same:

- The control plane IP used during the ARB deployment is within the IP pool range. The IP pool for ARB deployment is defined by a start and end IP range. The control plane IP should not fall within this range. The pre-requisite for ARB deployment specifies that the control plane IP must be in the same subnet as the IP pool but should not be part of the IP pool range. 

- The gateway IP used during the ARB deployment is within the IP pool range. The prerequisite for ARB deployment states that the gateway IP should not be within the start and end IP range of the IP pool.

If the upgrade fails due to any of the above errors, the only recovery option is to perform a fresh onboarding using a Disaster Recovery (DR) approach. This involves deleting the existing ARB Azure resource, recreating a new ARB with the same configuration, and re-mapping the SCVMM resources.

This [document](https://learn.microsoft.com/en-us/azure/azure-arc/system-center-virtual-machine-manager/disaster-recovery) provides a detailed guide on performing a fresh onboarding using a Disaster Recovery (DR).
