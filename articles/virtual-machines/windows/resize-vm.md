---
title: Resize a virtual machine using the Azure portal or PowerShell 
description: Change the VM size used for an Azure virtual machine.
author: cynthn
ms.service: virtual-machines
ms.workload: infrastructure
ms.topic: how-to
ms.date: 01/13/2020
ms.author: cynthn 
ms.custom: devx-track-azurepowershell

---
# Resize a virtual machine using the Azure portal or PowerShell

**Applies to:** :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets 

This article shows you how to move a VM to a different [VM size](../sizes.md).

After you create a virtual machine (VM), you can scale the VM up or down by changing the VM size. In some cases, you must deallocate the VM first. This can happen if the new size is not available on the hardware cluster that is currently hosting the VM.

If your VM uses Premium Storage, make sure that you choose an **s** version of the size to get Premium Storage support. For example, choose Standard_E4**s**_v3 instead of Standard_E4_v3.

## Use the portal

1. Open the [Azure portal](https://portal.azure.com).
1. Open the page for the virtual machine.
1. In the left menu, select **Size**.
1. Pick a new size from the list of available sizes and then select **Resize**.


If the virtual machine is currently running, changing its size will cause it to be restarted. Stopping the virtual machine may reveal additional sizes.

## Use PowerShell to resize a VM not in an availability set

Set some variables. Replace the values with your own information.

```powershell
$resourceGroup = "myResourceGroup"
$vmName = "myVM"
```

List the VM sizes that are available in the region where the VM is hosted. 
   
```powershell
Get-AzVMSize -ResourceGroupName $resourceGroup -VMName $vmName 
```

If the size you want is listed, run the following commands to resize the VM. If the desired size is not listed, go on to step 3.
   
```powershell
$vm = Get-AzVM -ResourceGroupName $resourceGroup -VMName $vmName
$vm.HardwareProfile.VmSize = "<newVMsize>"
Update-AzVM -VM $vm -ResourceGroupName $resourceGroup
```

If the size you want is not listed, run the following commands to deallocate the VM, resize it, and restart the VM. Replace **\<newVMsize>** with the size you want.
   
```powershell
Stop-AzVM -ResourceGroupName $resourceGroup -Name $vmName -Force
$vm = Get-AzVM -ResourceGroupName $resourceGroup -VMName $vmName
$vm.HardwareProfile.VmSize = "<newVMSize>"
Update-AzVM -VM $vm -ResourceGroupName $resourceGroup
Start-AzVM -ResourceGroupName $resourceGroup -Name $vmName
```

> [!WARNING]
> Deallocating the VM releases any dynamic IP addresses assigned to the VM. The OS and data disks are not affected. 
> 
> 

## Use PowerShell to resize a VM in an availability set

If the new size for a VM in an availability set is not available on the hardware cluster currently hosting the VM, then all VMs in the availability set will need to be deallocated to resize the VM. You also might need to update the size of other VMs in the availability set after one VM has been resized. To resize a VM in an availability set, perform the following steps.

```powershell
$resourceGroup = "myResourceGroup"
$vmName = "myVM"
```

List the VM sizes that are available on the hardware cluster where the VM is hosted. 
   
```powershell
Get-AzVMSize -ResourceGroupName $resourceGroup -VMName $vmName 
```

If the desired size is listed, run the following commands to resize the VM. If it is not listed, go to the next section.
   
```powershell
$vm = Get-AzVM -ResourceGroupName $resourceGroup -VMName $vmName 
$vm.HardwareProfile.VmSize = "<newVmSize>"
Update-AzVM -VM $vm -ResourceGroupName $resourceGroup
```
	
If the size you want is not listed, continue with the following steps to deallocate all VMs in the availability set, resize VMs, and restart them.

Stop all VMs in the availability set.
   
```powershell
$availabilitySetName = "<availabilitySetName>"
$as = Get-AzAvailabilitySet -ResourceGroupName $resourceGroup -Name $availabilitySetName
$virtualMachines = $as.VirtualMachinesReferences |  Get-AzResource | Get-AzVM
$virtualMachines |  Stop-AzVM -Force -NoWait  
```

Resize and restart the VMs in the availability set.
   
```powershell
$availabilitySetName = "<availabilitySetName>"
$newSize = "<newVmSize>"
$as = Get-AzAvailabilitySet -ResourceGroupName $resourceGroup -Name $availabilitySetName
$virtualMachines = $as.VirtualMachinesReferences |  Get-AzResource | Get-AzVM
$virtualMachines | Foreach-Object { $_.HardwareProfile.VmSize = $newSize }
$virtualMachines | Update-AzVM
$virtualMachines | Start-AzVM
```

## Next steps

For additional scalability, run multiple VM instances and scale out. For more information, see [Automatically scale machines in a Virtual Machine Scale Set](../../virtual-machine-scale-sets/tutorial-autoscale-powershell.md).
