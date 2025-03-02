---
title: Configure Postman for Azure Media Services REST API calls
description: This article describes how to configure Postman for Media Services REST API calls.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
---
# Configure Postman for Media Services v2 REST API calls

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

[!INCLUDE [v2 deprecation notice](../latest/includes/v2-deprecation-notice.md)]

This tutorial shows you how to configure **Postman** so it can be used to call Azure Media Services (AMS) REST APIs. The tutorial shows how to import environment and collection files into **Postman**. The collection contains grouped definitions of HTTP requests that call Azure Media Services (AMS) REST APIs. The environment file contains variables that are used by the collection.

This environment and collection is used in articles that show how to achieve various tasks with Azure Media Services REST APIs.

## Prerequisites

- Install the [Postman](https://www.getpostman.com/) REST client to execute the REST APIs shown in some of the AMS REST tutorials. 

    We are using **Postman** but any REST tool would be suitable. Other alternatives are: **Visual Studio Code** with the REST plugin or **Telerik Fiddler**. 

## Configure the environment 

1. Create a .json file that contains the environment variables used in AMS tutorials. Name the file (for example, **AzureMediaServices.postman_environment.json**). Open the file and paste the code that defines the Postman environment from [this code listing](postman-environment.md). 
2. Open the **Postman**.
3. On the right of the screen, select the **Manage environment** option.

    ![Screenshot shows the Manage Environment option selected.](./media/media-services-rest-upload-files/postman-create-env.png)
4. From the **Manage environment** dialog, click **Import**.
5. Browse and select the **AzureMediaServices.postman_environment.json** file.
6. The **AzureMedia** environment is added.
7. Close the dialog.
8. Select the **AzureMedia** environment.

    ![Screenshot shows the AzureMedia environment selected.](./media/media-services-rest-upload-files/postman-choose-env.png)

## Configure the collection

1. Create a .json file that contains the **Postman** collection with all the operations that are needed to upload a file to Media Services. Name the file (for example, **AzureMediaServicesOperations.postman_collection.json**). Open the file and paste the code that defines the **Postman** collection from [this code listing](postman-collection.md).
2. Click **Import** to import the collection file.
3. Choose the **AzureMediaServicesOperations.postman_collection.json** file.

    ![Screenshot shows the IMPORT dialog box with Choose Files selected.](./media/media-services-rest-upload-files/postman-import-collection.png)

## Next steps

Check out the [upload assets](media-services-rest-upload-files.md) article.  
