---
title: Deploy multiple instances of resources in Bicep
description: Use loops and arrays in a Bicep file to deploy multiple instances of resources.

author: mumian
ms.author: jgao
ms.topic: conceptual
ms.date: 08/30/2021
---

# Resource iteration in Bicep

This article shows you how to create more than one instance of a resource in your Bicep file. You can add a loop to a `resource` declaration and dynamically set the number of resources to deploy. You avoid repeating syntax in your Bicep file.

You can also use a loop with [modules](loop-modules.md), [properties](loop-properties.md), [variables](loop-variables.md), and [outputs](loop-outputs.md).

If you need to specify whether a resource is deployed at all, see [condition element](conditional-resource-deployment.md).

## Syntax

Loops can be used declare multiple resources by:

- Using a loop index.

  ```bicep
  resource <resource-symbolic-name> '<resource-type>@<api-version>' = [for <index> in range(<start>, <stop>): {
    <resource-properties>
  }]
  ```

  For more information, see [Loop index](#loop-index).

- Iterating over an array.

  ```bicep
  resource <resource-symbolic-name> '<resource-type>@<api-version>' = [for <item> in <collection>: {
    <resource-properties>
  }]
  ```

  For more information, see [Loop array](#loop-array).

- Iterating over an array and index.

  ```bicep
  resource <resource-symbolic-name> '<resource-type>@<api-version>' = [for (<item>, <index>) in <collection>: {
    <resource-properties>
  }]
  ```

  For more information, see [Loop array and index](#loop-array-and-index).

## Loop limits

The Bicep file's loop iterations can't be a negative number or exceed 800 iterations.

## Loop index

The following example creates the number of storage accounts specified in the `storageCount` parameter.

```bicep
param rgLocation string = resourceGroup().location
param storageCount int = 2

resource storageAcct 'Microsoft.Storage/storageAccounts@2021-02-01' = [for i in range(0, storageCount): {
  name: '${i}storage${uniqueString(resourceGroup().id)}'
  location: rgLocation
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'Storage'
}]
```

Notice the index `i` is used in creating the storage account resource name.

## Loop array

The following example creates one storage account for each name provided in the `storageNames` parameter.

```bicep
param rgLocation string = resourceGroup().location
param storageNames array = [
  'contoso'
  'fabrikam'
  'coho'
]

resource storageAcct 'Microsoft.Storage/storageAccounts@2021-02-01' = [for name in storageNames: {
  name: '${name}${uniqueString(resourceGroup().id)}'
  location: rgLocation
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'Storage'
}]
```

If you want to return values from the deployed resources, you can use a loop in the [output section](loop-outputs.md).

## Loop array and index

The following example uses both the array element and index value when defining the storage account.

```bicep
param storageAccountNamePrefix string

var storageConfigurations = [
  {
    suffix: 'local'
    sku: 'Standard_LRS'
  }
  {
    suffix: 'geo'
    sku: 'Standard_GRS'
  }
]

resource storageAccountResources 'Microsoft.Storage/storageAccounts@2021-02-01' = [for (config, i) in storageConfigurations: {
  name: '${storageAccountNamePrefix}${config.suffix}${i}'
  location: resourceGroup().location
  properties: {
    supportsHttpsTrafficOnly: true
    accessTier: 'Hot'
    encryption: {
      keySource: 'Microsoft.Storage'
      services: {
        blob: {
          enabled: true
        }
        file: {
          enabled: true
        }
      }
    }
  }
  kind: 'StorageV2'
  sku: {
    name: config.sku
  }
}]
```

## Resource iteration with condition

The following example shows a nested loop combined with a filtered resource loop. Filters must be expressions that evaluate to a boolean value.

```bicep
resource parentResources 'Microsoft.Example/examples@2020-06-06' = [for parent in parents: if(parent.enabled) {
  name: parent.name
  properties: {
    children: [for child in parent.children: {
      name: child.name
      setting: child.settingValue
    }]
  }
}]
```

Filters are also supported with [module loops](loop-modules.md).

## Deploy in batches

By default, Resource Manager creates resources in parallel. When you use a loop to create multiple instances of a resource type, those instances are all deployed at the same time. The order in which they're created isn't guaranteed. There's no limit to the number of resources deployed in parallel, other than the total limit of 800 resources in the Bicep file.

You might not want to update all instances of a resource type at the same time. For example, when updating a production environment, you may want to stagger the updates so only a certain number are updated at any one time. You can specify that a subset of the instances be batched together and deployed at the same time. The other instances wait for that batch to complete.

To serially deploy instances of a resource, add the [batchSize decorator](./file.md#resource-and-module-decorators). Set its value to the number of instances to deploy concurrently. A dependency is created on earlier instances in the loop, so it doesn't start one batch until the previous batch completes.

```bicep
param rgLocation string = resourceGroup().location

@batchSize(2)
resource storageAcct 'Microsoft.Storage/storageAccounts@2021-02-01' = [for i in range(0, 4): {
  name: '${i}storage${uniqueString(resourceGroup().id)}'
  location: rgLocation
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'Storage'
}]
```

For purely sequential deployment, set the batch size to 1.

## Iteration for a child resource

You can't use a loop for a nested child resource. To create more than one instance of a child resource, change the child resource to a top-level resource.

For example, suppose you typically define a file service and file share as nested resources for a storage account.

```bicep
resource stg 'Microsoft.Storage/storageAccounts@2021-02-01' = {
  name: 'examplestorage'
  location: resourceGroup().location
  kind: 'StorageV2'
  sku: {
    name: 'Standard_LRS'
  }
  resource service 'fileServices' = {
    name: 'default'
    resource share 'shares' = {
      name: 'exampleshare'
    }
  }
}
```

To create more than one file share, move it outside of the storage account. You define the relationship with the parent resource through the `parent` property.

The following example shows how to create a storage account, file service, and more than one file share:

```bicep
resource stg 'Microsoft.Storage/storageAccounts@2021-02-01' = {
  name: 'examplestorage'
  location: resourceGroup().location
  kind: 'StorageV2'
  sku: {
    name: 'Standard_LRS'
  }
}

resource service 'Microsoft.Storage/storageAccounts/fileServices@2021-02-01' = {
  name: 'default'
  parent: stg
}

resource share 'Microsoft.Storage/storageAccounts/fileServices/shares@2021-02-01' = [for i in range(0, 3): {
  name: 'exampleshare${i}'
  parent: service
}]
```

## Next steps

- For other uses of the loop, see:
  - [Property iteration in Bicep](loop-properties.md)
  - [Variable iteration in Bicep](loop-variables.md)
  - [Output iteration in Bicep](loop-outputs.md)
- To set dependencies on resources that are created in a loop, see [Set resource dependencies](./resource-declaration.md#set-resource-dependencies).
