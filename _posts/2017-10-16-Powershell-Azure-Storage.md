--- 
layout: post
title: "Connecting to Windows Azure Storage through PowerShell"
author: "Author"
comments: true
tags : [Azure, Powershell]
---

I came across this paper describing the Windows Azure Storage design [here](). 

And with that, I stopped thinking about Windows Azure Storage as as an extended file system and started thinking of it as a distributed, highly available, noSql persistence layer provided by the Azure platform.

That changes things quite a bit.

## Windows Azure Storage - *not an extended file system*.

It provides all three of Consistency, Availability and Partition tolerance, which apparently violates CAP, but has actually been made possible through some heavy lifting as detailed in the paper.

It offers five different sorts of distributed data services 
*   Blobs 
    -   Block blob *for streaming data*
    -   Page blob *for Random Access data*
    -   Append blob *for Append-only data*
*   Table *for unstructured tabular data* 
*   Queues *for sequential processing*
*   Disks *are usually usually VHDs for IAAS*
*   Files - *are usually premium NAS for lift-and-shift*

Only problem, is, all of this through the Azure Portal is a lot of clicks. Now, all Azure services offer REST apis, and PowerShell access. So, let's see if all those clicks can be mutated into key-strokes, logging into and accessing Windows Azure Storage through PowerShell commands.

### A short but important note.

After a while of fiddling around with Azure PowerShell commands, you will find that there are two versions of almost everything.

```Login-AzureAccount``` vs ```Login-AzureRmAccount```

```Get-AzureStorageAccount``` vs ```Get-AzureRmStorageAccount```

This non ```Rm``` and ```Rm``` options indicate Azure operational models before and after there was a Azure concept called *resource group*. A *resource group* is a logical grouping of Azure physical services - viz. storage, VMs, etc. 

To read more, look [here](https://blogs.technet.microsoft.com/meamcs/2016/12/22/difference-between-azure-service-manager-and-azure-resource-manager/) and [here](https://azuredepot.com/2017/01/02/azure-login-options/).

In our case, we need to be conscious that wherever there is a choice, we need to go with the ```Rm``` versions.

## Setting Up PowerShell for Windows Azure

For a fresh machine, we need to have the ```AzureRM``` package installed. On an elevated (Administrator) prompt

- Verify you have ```PowerShellGet``` Module installed. 

    ```Get-Module PowerShellGet -list | Select-Object Name,Version,Path```

-   Install Azure RM

    ```Install-Module AzureRM```

-   If you have a previous version of Azure PowerShell installed that includes the Service Management module, you may receive an error. In that case, do this.

    ```Install-Module AzureRM -AllowClobber```

-   To verify ```AzureRM``` was installed run 

    ```Get-Module -ListAvailable -Name AzureRM```

## Azure Credentials for PowerShell Session

Close the current PS prompt and open a non-elevated one. 

-   Login to Azure

    ```Login-AzureRmAccount``` - This now does the Oauth dance and takes you to a login screen, takes your input, authenticates and returns to the prompt.

-   Add Azure Account to PowerShell

    ```Add-AzureAccount``` - This should show you subscription details on command prompt

        Id/Type/Subscriptions/Tenants
        
        id@mail.com/User/d81de17a-cc30-4bee-a874-27d1eea6062/{aea858f0-512d-4649-8228-d78fd9ef3c76}
    
-   Set your current subscription as the default.

    ```Select-AzureSubscription -Current -SubscriptionId "your-subscription-id"```

-   To verify, run 

    ```Get-AzureRmContext```

And that is it. We are all set. 

To access a specific storage account, try this - 

```Get-AzureRmStorageAccount -ResourceGroupName "your-resource-group" -AccountName "account-name" | Select StorageAccountName```

You should see your StorageAccountName on your console.

*Note: do create the StorageAccount through the portal before this.*

So here we are, interacting with our Azure Storage Accounts through PowerShell and the command prompt.










