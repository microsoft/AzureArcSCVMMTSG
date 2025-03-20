**Error Code ID:** KVAAlreadyDeployedCustomerError

**Error message**

An appliance already exists with this configuration (deployment phase: MachineRunning) - please call Delete before reattempting deployment.

**Explanation**

During the Azure Arc resource bridge deployment, an Appliance virtual machine is created on the selected Hostgroup/Cloud in the VMM Server. If a virtual machine with the same name already exists on the given VMM Server, you may encounter the above error.

To resolve this, check if a virtual machine with the same name exists on any host associated with the VMM Server and manually delete it before proceeding.

If you re-run the onboarding script from the same folder where the three configuration YAML files (*-resource.yaml, *-infra.yaml, and *-appliance.yaml) from the previous attempt are stored, the script automatically appends the -force flag to the Appliance creation command. This flag ensures that any existing virtual machine with the same name is deleted before creating a new one.

However, there are scenarios where the -force flag may not resolve the issue, and the error may still appear during script execution. This could be due to the virtual machine being in a failed state, such as HostNotResponding or CreationFailed, where deletion is not supported.

- Check the host where the virtual machine was created and resolve any issues with the host.
- Manually delete the virtual machine from the VMM Server.
- Delete the Arc resource bridge resource from Azure.
- Re-run the Onboarding script.

<br>

> **Recommendation:**  It is best practice to run the onboarding script from a separate folder, as the generated configuration YAML files are useful for Day-N operations. However, if re-running from the same folder, ensure the -force flag is used to clean up any previous resources before retrying.
