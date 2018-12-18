---
title: 使用 C# 來建立和管理 Azure 虛擬機器 | Microsoft Docs
description: 使用 C# 和 Azure Resource Manager 來部署 Azure 虛擬機器及所有支援它的資源。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: 87524373-5f52-4f4b-94af-50bf7b65c277
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 07/17/2017
ms.author: cynthn
ms.openlocfilehash: 99c8e71e0a31f6ef2f4fb2087fb8678c68a052a7
ms.sourcegitcommit: 1362e3d6961bdeaebed7fb342c7b0b34f6f6417a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/18/2018
ms.locfileid: "31525924"
---
# <a name="create-and-manage-windows-vms-in-azure-using-c"></a>在 Azure 中使用 C# 建立並管理 Windows VM #

[Azure 虛擬機器](overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) (VM) 需要數個支援的 Azure 資源。 本文涵蓋使用 C# 建立、管理和刪除 VM 資源。 您會了解如何：

> [!div class="checklist"]
> * 建立 Visual Studio 專案
> * 安裝套件
> * 建立認證
> * 建立資源
> * 執行管理工作
> * 刪除資源
> * 執行應用程式

執行這些步驟大約需要 20 分鐘的時間。

## <a name="create-a-visual-studio-project"></a>建立 Visual Studio 專案

1. 如果您尚未安裝 [Visual Studio](https://docs.microsoft.com/visualstudio/install/install-visual-studio)，請進行安裝。 在 [工作負載] 分頁上選取 [.NET 桌面開發]，然後按一下 [安裝]。 在摘要中，您可以看到系統自動為您選取 [NET Framework 4 – 4.6 開發工具]。 如果您已安裝 Visual Studio，您可以使用 Visual Studio Launcher 新增 .NET 工作負載。
2. 在 Visual Studio 中，按一下 [檔案] > [新增] > [專案]。
3. 在 [範本] > [Visual C#] 中，選取 [主控台應用程式 (.NET Framework)]，針對專案名稱輸入 myDotnetProject，選取專案的位置，然後按一下 [確定]。

## <a name="install-the-package"></a>安裝套件

NuGet 套件是安裝完成這些步驟所需之程式庫的最簡單方式。 若要取得在 Visual Studio 中所需要的程式庫，請執行下列步驟：

1. 按一下 [工具] > [NuGet 套件管理員]，然後按一下 [Package Manager Console]。
2. 在主控台中輸入以下命令：

    ```
    Install-Package Microsoft.Azure.Management.Fluent
    ```

## <a name="create-credentials"></a>建立認證

在您開始此步驟之前，請確定您可以存取 [Active Directory 服務主體](../../azure-resource-manager/resource-group-create-service-principal-portal.md)。 您也應該記錄應用程式識別碼、驗證金鑰以及租用戶識別碼，您在稍後的步驟會需要這些項目。

### <a name="create-the-authorization-file"></a>建立授權檔

1. 在 [方案總管] 中，於 [myDotnetProject] 上按一下滑鼠右鍵 > [新增] > [新增項目]，然後選取 [Visual C# 項目] 中的 [文字檔]。 將檔案命名為 *azureauth.properties*，然後按一下 [新增]。
2. 新增下列授權屬性：

    ```
    subscription=<subscription-id>
    client=<application-id>
    key=<authentication-key>
    tenant=<tenant-id>
    managementURI=https://management.core.windows.net/
    baseURL=https://management.azure.com/
    authURL=https://login.windows.net/
    graphURL=https://graph.windows.net/
    ```

    以您的訂用帳戶 ID 取代 **&lt;subscription-id&gt;**、以 Active Directory 應用程式識別碼取代 **&lt;application-id&gt;**、以應用程式金鑰取代 **&lt;authentication-key&gt;**，以及以租用戶識別碼取代 **&lt;tenant-id&gt;**。

3. 儲存 azureauth.properties 檔案。 
4. 在 Windows 中名為 AZURE_AUTH_LOCATION 的環境變數上，設定您所建立之授權檔的完整路徑。 例如，可以使用下列 PowerShell 命令：

    ```
    [Environment]::SetEnvironmentVariable("AZURE_AUTH_LOCATION", "C:\Visual Studio 2017\Projects\myDotnetProject\myDotnetProject\azureauth.properties", "User")
    ```

### <a name="create-the-management-client"></a>建立管理用戶端

1. 開啟您建立之專案的 Program.cs 檔案，然後將這些 using 陳述式新增至檔案頂端的現有陳述式：

    ```
    using Microsoft.Azure.Management.Compute.Fluent;
    using Microsoft.Azure.Management.Compute.Fluent.Models;
    using Microsoft.Azure.Management.Fluent;
    using Microsoft.Azure.Management.ResourceManager.Fluent;
    using Microsoft.Azure.Management.ResourceManager.Fluent.Core;
    ```

2. 若要建立管理用戶端，請將以下程式碼新增到 Main 方法：

    ```
    var credentials = SdkContext.AzureCredentialsFactory
        .FromFile(Environment.GetEnvironmentVariable("AZURE_AUTH_LOCATION"));

    var azure = Azure
        .Configure()
        .WithLogLevel(HttpLoggingDelegatingHandler.Level.Basic)
        .Authenticate(credentials)
        .WithDefaultSubscription();
    ```

## <a name="create-resources"></a>建立資源

### <a name="create-the-resource-group"></a>建立資源群組

所有資源都必須包含在[資源群組](../../azure-resource-manager/resource-group-overview.md)中。

若要指定應用程式的值並建立資源群組，請將以下程式碼新增到 Main 方法：

```
var groupName = "myResourceGroup";
var vmName = "myVM";
var location = Region.USWest;
    
Console.WriteLine("Creating resource group...");
var resourceGroup = azure.ResourceGroups.Define(groupName)
    .WithRegion(location)
    .Create();
```

### <a name="create-the-availability-set"></a>建立可用性設定組

維護您應用程式所使用的虛擬機器時，使用[可用性設定組](tutorial-availability-sets.md)可讓您更加輕鬆。

若要建立可用性設定組，請將以下程式碼新增到 Main 方法：

```
Console.WriteLine("Creating availability set...");
var availabilitySet = azure.AvailabilitySets.Define("myAVSet")
    .WithRegion(location)
    .WithExistingResourceGroup(groupName)
    .WithSku(AvailabilitySetSkuTypes.Managed)
    .Create();
```

### <a name="create-the-public-ip-address"></a>建立公用 IP 位址

必須要有[公用 IP 位址](../../virtual-network/virtual-network-ip-addresses-overview-arm.md)才能與虛擬機器進行通訊。

若要為虛擬機器建立公用 IP 位址，請將以下程式碼新增到 Main 方法：
   
```
Console.WriteLine("Creating public IP address...");
var publicIPAddress = azure.PublicIPAddresses.Define("myPublicIP")
    .WithRegion(location)
    .WithExistingResourceGroup(groupName)
    .WithDynamicIP()
    .Create();
```

### <a name="create-the-virtual-network"></a>建立虛擬網路

虛擬機器必須在[虛擬網路](../../virtual-network/virtual-networks-overview.md)的子網路中。

若要建立子網路和虛擬網路，請將以下程式碼新增到 Main 方法：

```
Console.WriteLine("Creating virtual network...");
var network = azure.Networks.Define("myVNet")
    .WithRegion(location)
    .WithExistingResourceGroup(groupName)
    .WithAddressSpace("10.0.0.0/16")
    .WithSubnet("mySubnet", "10.0.0.0/24")
    .Create();
```

### <a name="create-the-network-interface"></a>建立網路介面

虛擬機器需要網路介面以在虛擬網路上通訊。

若要建立網路介面，請將以下程式碼新增到 Main 方法：

```
Console.WriteLine("Creating network interface...");
var networkInterface = azure.NetworkInterfaces.Define("myNIC")
    .WithRegion(location)
    .WithExistingResourceGroup(groupName)
    .WithExistingPrimaryNetwork(network)
    .WithSubnet("mySubnet")
    .WithPrimaryPrivateIPAddressDynamic()
    .WithExistingPrimaryPublicIPAddress(publicIPAddress)
    .Create();
 ```

### <a name="create-the-virtual-machine"></a>建立虛擬機器

既然您已經建立所有支援的資源，您可以建立虛擬機器。

若要建立虛擬機器，請將以下程式碼新增到 Main 方法：

```
Console.WriteLine("Creating virtual machine...");
azure.VirtualMachines.Define(vmName)
    .WithRegion(location)
    .WithExistingResourceGroup(groupName)
    .WithExistingPrimaryNetworkInterface(networkInterface)
    .WithLatestWindowsImage("MicrosoftWindowsServer", "WindowsServer", "2012-R2-Datacenter")
    .WithAdminUsername("azureuser")
    .WithAdminPassword("Azure12345678")
    .WithComputerName(vmName)
    .WithExistingAvailabilitySet(availabilitySet)
    .WithSize(VirtualMachineSizeTypes.StandardDS1)
    .Create();
```

> [!NOTE]
> 本教學課程中會建立執行 Windows Server 作業系統版本的虛擬機器。 若要深入了解如何選取其他映像，請參閱 [使用 Windows PowerShell 和 Azure CLI 來瀏覽和選取 Azure 虛擬機器映像](../linux/cli-ps-findimage.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。
> 
>

如果您想要使用現有的磁碟而不使用市集映像，請使用以下程式碼：

```
var managedDisk = azure.Disks.Define("myosdisk")
    .WithRegion(location)
    .WithExistingResourceGroup(groupName)
    .WithWindowsFromVhd("https://mystorage.blob.core.windows.net/vhds/myosdisk.vhd")
    .WithSizeInGB(128)
    .WithSku(DiskSkuTypes.PremiumLRS)
    .Create();

azure.VirtualMachines.Define("myVM")
    .WithRegion(location)
    .WithExistingResourceGroup(groupName)
    .WithExistingPrimaryNetworkInterface(networkInterface)
    .WithSpecializedOSDisk(managedDisk, OperatingSystemTypes.Windows)
    .WithExistingAvailabilitySet(availabilitySet)
    .WithSize(VirtualMachineSizeTypes.StandardDS1)
    .Create();
```

## <a name="perform-management-tasks"></a>執行管理工作

在虛擬機器的生命週期內，您可以執行一些管理工作，例如啟動、停止或刪除虛擬機器。 此外，您可以建立程式碼來自動執行重複或複雜的工作。

當您需要使用 VM 來執行任何操作時，就必須取得其執行個體：

```
var vm = azure.VirtualMachines.GetByResourceGroup(groupName, vmName);
```

### <a name="get-information-about-the-vm"></a>取得 VM 的相關資訊

若要取得虛擬機器的相關資訊，請將以下程式碼新增到 Main 方法：

```
Console.WriteLine("Getting information about the virtual machine...");
Console.WriteLine("hardwareProfile");
Console.WriteLine("   vmSize: " + vm.Size);
Console.WriteLine("storageProfile");
Console.WriteLine("  imageReference");
Console.WriteLine("    publisher: " + vm.StorageProfile.ImageReference.Publisher);
Console.WriteLine("    offer: " + vm.StorageProfile.ImageReference.Offer);
Console.WriteLine("    sku: " + vm.StorageProfile.ImageReference.Sku);
Console.WriteLine("    version: " + vm.StorageProfile.ImageReference.Version);
Console.WriteLine("  osDisk");
Console.WriteLine("    osType: " + vm.StorageProfile.OsDisk.OsType);
Console.WriteLine("    name: " + vm.StorageProfile.OsDisk.Name);
Console.WriteLine("    createOption: " + vm.StorageProfile.OsDisk.CreateOption);
Console.WriteLine("    caching: " + vm.StorageProfile.OsDisk.Caching);
Console.WriteLine("osProfile");
Console.WriteLine("  computerName: " + vm.OSProfile.ComputerName);
Console.WriteLine("  adminUsername: " + vm.OSProfile.AdminUsername);
Console.WriteLine("  provisionVMAgent: " + vm.OSProfile.WindowsConfiguration.ProvisionVMAgent.Value);
Console.WriteLine("  enableAutomaticUpdates: " + vm.OSProfile.WindowsConfiguration.EnableAutomaticUpdates.Value);
Console.WriteLine("networkProfile");
foreach (string nicId in vm.NetworkInterfaceIds)
{
    Console.WriteLine("  networkInterface id: " + nicId);
}
Console.WriteLine("vmAgent");
Console.WriteLine("  vmAgentVersion" + vm.InstanceView.VmAgent.VmAgentVersion);
Console.WriteLine("    statuses");
foreach (InstanceViewStatus stat in vm.InstanceView.VmAgent.Statuses)
{
    Console.WriteLine("    code: " + stat.Code);
    Console.WriteLine("    level: " + stat.Level);
    Console.WriteLine("    displayStatus: " + stat.DisplayStatus);
    Console.WriteLine("    message: " + stat.Message);
    Console.WriteLine("    time: " + stat.Time);
}
Console.WriteLine("disks");
foreach (DiskInstanceView disk in vm.InstanceView.Disks)
{
    Console.WriteLine("  name: " + disk.Name);
    Console.WriteLine("  statuses");
    foreach (InstanceViewStatus stat in disk.Statuses)
    {
        Console.WriteLine("    code: " + stat.Code);
        Console.WriteLine("    level: " + stat.Level);
        Console.WriteLine("    displayStatus: " + stat.DisplayStatus);
        Console.WriteLine("    time: " + stat.Time);
    }
}
Console.WriteLine("VM general status");
Console.WriteLine("  provisioningStatus: " + vm.ProvisioningState);
Console.WriteLine("  id: " + vm.Id);
Console.WriteLine("  name: " + vm.Name);
Console.WriteLine("  type: " + vm.Type);
Console.WriteLine("  location: " + vm.Region);
Console.WriteLine("VM instance status");
foreach (InstanceViewStatus stat in vm.InstanceView.Statuses)
{
    Console.WriteLine("  code: " + stat.Code);
    Console.WriteLine("  level: " + stat.Level);
    Console.WriteLine("  displayStatus: " + stat.DisplayStatus);
}
Console.WriteLine("Press enter to continue...");
Console.ReadLine();
```

### <a name="stop-the-vm"></a>停止 VM

您可以停止虛擬機器並保留其所有的設定，但仍繼續計費，或您可以停止虛擬機器並將其解除配置。 當解除配置虛擬機器時，與其相關聯的所有資源也都會解除配置且其計費會結束。

若要停止虛擬機器而不解除配置，請將以下程式碼新增到 Main 方法：

```
Console.WriteLine("Stopping vm...");
vm.PowerOff();
Console.WriteLine("Press enter to continue...");
Console.ReadLine();
```

如果您想要解除配置虛擬機器，請將 PowerOff 呼叫變更為下列程式碼︰

```
vm.Deallocate();
```

### <a name="start-the-vm"></a>啟動 VM

若要啟動虛擬機器，請將以下程式碼新增到 Main 方法：

```
Console.WriteLine("Starting vm...");
vm.Start();
Console.WriteLine("Press enter to continue...");
Console.ReadLine();
```

### <a name="resize-the-vm"></a>調整 VM 的大小

決定虛擬機器的大小時，應該考慮部署的許多層面。 如需詳細資訊，請參閱 [VM 大小](sizes.md)。  

若要變更虛擬機器的大小，請將以下程式碼新增到 Main 方法：

```
Console.WriteLine("Resizing vm...");
vm.Update()
    .WithSize(VirtualMachineSizeTypes.StandardDS2) 
    .Apply();
Console.WriteLine("Press enter to continue...");
Console.ReadLine();
```

### <a name="add-a-data-disk-to-the-vm"></a>將資料磁碟新增至 VM

若要將資料磁碟新增到虛擬機器，請將以下程式碼新增到 Main 方法，以新增一個大小為 2 GB、LUN 為 0 且快取類型為 ReadWrite 的資料磁碟：

```
Console.WriteLine("Adding data disk to vm...");
vm.Update()
    .WithNewDataDisk(2, 0, CachingTypes.ReadWrite) 
    .Apply();
Console.WriteLine("Press enter to delete resources...");
Console.ReadLine();
```

## <a name="delete-resources"></a>刪除資源

由於您需要為在 Azure 中使用的資源付費，因此刪除不再需要的資源一律是理想的做法。 如果您想要刪除虛擬機器及所有支援的資源，您只需要刪除資源群組。

若要刪除資源群組，請將以下程式碼新增到 Main 方法：

```
azure.ResourceGroups.DeleteByName(groupName);
```

## <a name="run-the-application"></a>執行應用程式

此主控台應用程式從開始到完成的完整執行應該需要五分鐘左右。 

1. 若要執行主控台應用程式，請按一下 [啟動]。

2. 在您按 **Enter** 以開始刪除資源之前，可以先花幾分鐘的時間來確認 Azure 入口網站中的資源建立情況。 請按一下部署狀態來查看該項部署的相關資訊。

## <a name="next-steps"></a>後續步驟
* 使用 [利用 C# 和 Resource Manager 範本來部署 Azure 虛擬機器](csharp-template.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)中的資訊，以利用範本來建立虛擬機器。
* 深入了解關於使用 [Azure Libraries for .NET](https://docs.microsoft.com/dotnet/azure/?view=azure-dotnet)。

