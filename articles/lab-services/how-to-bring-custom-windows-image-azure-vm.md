---
title: Azure Lab Services - How to bring a Windows custom image from an Azure virtual machine
description: Describes how to bring a Windows custom image from an Azure virtual machine.
ms.date: 07/27/2021
ms.topic: how-to
---

# Bring a Windows custom image from an Azure virtual machine

The steps in this article show how to import a custom image that starts from an [Azure virtual machine (VM)](https://azure.microsoft.com/services/virtual-machines/). With this approach, you set up an image on an Azure VM and import the image into a shared image gallery so that it can be used within Azure Lab Services. Before you use this approach for creating a custom image, read [Recommended approaches for creating custom images](approaches-for-custom-image-creation.md) to decide the best approach for your scenario.

## Prerequisites

You'll need permission to create an Azure VM in your school's Azure subscription to complete the steps in this article.

## Prepare a custom image on an Azure VM

1. Create an Azure VM by using the [Azure portal](../virtual-machines/windows/quick-create-portal.md), [PowerShell](../virtual-machines/windows/quick-create-powershell.md), the [Azure CLI](../virtual-machines/windows/quick-create-cli.md), or an [Azure Resource Manager template](../virtual-machines/windows/quick-create-template.md).
    
    - When you specify the disk settings, ensure the disk's size is *not* greater than 128 GB.
    
1. Install software and make any necessary configuration changes to the Azure VM's image.

1. Optionally, you can generalize the image. Run [SysPrep](../virtual-machines/generalize.md#windows) if you need to create a generalized image. Otherwise, if you're creating a specialized image, you can skip to the next step.

    Create a specialized image if you want to maintain machine-specific information and user profiles. For more information about the differences between generalized and specialized images, see [Generalized and specialized images](../virtual-machines/shared-image-galleries.md#generalized-and-specialized-images).

## Import the custom image into a shared image gallery

1. In a shared image gallery, [create an image definition](../virtual-machines/windows/shared-images-portal.md#create-an-image-definition) or choose an existing image definition.
     - Choose **Gen 1** for the **VM generation**.
     - Choose whether you're creating a **specialized** or **generalized** image for the **Operating system state**.

    For more information about the values you can specify for an image definition, see [Image definitions](../virtual-machines/shared-image-galleries.md#image-definitions). 
    
    You can also choose to use an existing image definition and create a new version for your custom image.
    
1. [Create an image version](../virtual-machines/windows/shared-images-portal.md#create-an-image-version).
    - The **Version number** property uses the following format: *MajorVersion.MinorVersion.Patch*.   
    - For the **Source**, select **Disks and/or snapshots** from the dropdown list.
    - For the **OS disk** property, choose your Azure VM's disk that you created in previous steps.

    You can also import your custom image from an Azure VM to a shared image gallery by using PowerShell. For more information, see the script and ReadMe in [Bring image to shared image gallery script](https://github.com/Azure/azure-devtestlab/tree/master/samples/ClassroomLabs/Scripts/BringImageToSharedImageGallery/).

## Create a lab

[Create the lab](tutorial-setup-classroom-lab.md) in Lab Services, and select the custom image from the shared image gallery.

## Next steps

* [Shared image gallery overview](../virtual-machines/shared-image-galleries.md)
* [Attach or detach a shard image gallery](how-to-attach-detach-shared-image-gallery.md)
* [Use a shared image gallery](how-to-use-shared-image-gallery.md)