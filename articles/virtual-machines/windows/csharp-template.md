---
title: 使用 C# 和 Resource Manager 範本部署 VM | Microsoft Docs
description: 了解如何使用 C# 和 Resource Manager 範本來部署 Azure VM。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: bfba66e8-c923-4df2-900a-0c2643b81240
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 07/14/2017
ms.author: cynthn
ms.openlocfilehash: e57505b4bd89a79af076dc4cf132c844ae0abd1d
ms.sourcegitcommit: 1362e3d6961bdeaebed7fb342c7b0b34f6f6417a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/18/2018
ms.locfileid: "31527819"
---
# <a name="deploy-an-azure-virtual-machine-using-c-and-a-resource-manager-template"></a>利用 C# 和 Resource Manager 範本來部署 Azure 虛擬機器
本文說明如何使用 C# 來部署 Azure Resource Manager 範本。 您建立的範本會在具有單一子網路的新虛擬網路中，部署執行 Windows Server 的單一虛擬機器。

如需虛擬機器資源的詳細說明，請參閱 [Azure Resource Manager 範本中的虛擬機器 (英文)](template-description.md)。 如需有關範本中所有資源的詳細資訊，請參閱 [Azure Resource Manager 範本逐步解說](../../azure-resource-manager/resource-manager-template-walkthrough.md)。

執行這些步驟大約需要 10 分鐘的時間。

## <a name="create-a-visual-studio-project"></a>建立 Visual Studio 專案

在此步驟中，您會確定是否已安裝 Visual Studio，並建立用來部署此範本的主控台應用程式。

1. 如果您尚未安裝 [Visual Studio](https://docs.microsoft.com/visualstudio/install/install-visual-studio)，請進行安裝。 在 [工作負載] 分頁上選取 [.NET 桌面開發]，然後按一下 [安裝]。 在摘要中，您可以看到系統自動為您選取 [NET Framework 4 – 4.6 開發工具]。 如果您已安裝 Visual Studio，您可以使用 Visual Studio Launcher 新增 .NET 工作負載。
2. 在 Visual Studio 中，按一下 [檔案] > [新增] > [專案]。
3. 在 [範本] > [Visual C#] 中，選取 [主控台應用程式 (.NET Framework)]，針對專案名稱輸入 myDotnetProject，選取專案的位置，然後按一下 [確定]。

## <a name="install-the-packages"></a>安裝套件

NuGet 套件是安裝完成這些步驟所需之程式庫的最簡單方式。 若要取得在 Visual Studio 中所需要的程式庫，請執行下列步驟：

1. 按一下 [工具] > [NuGet 套件管理員]，然後按一下 [Package Manager Console]。
2. 在主控台中輸入下列命令：

    ```
    Install-Package Microsoft.Azure.Management.Fluent
    Install-Package WindowsAzure.Storage
    ```

## <a name="create-the-files"></a>建立檔案

在此步驟中，您會建立部署資源的範本檔案，以及為範本提供參數值的參數檔案。 您也會建立用來執行 Azure Resource Manager 作業的授權檔案。

### <a name="create-the-template-file"></a>建立範本檔案

1. 在 [方案總管] 中，於 [myDotnetProject] 上按一下滑鼠右鍵 > [新增] > [新增項目]，然後選取 [Visual C# 項目] 中的 [文字檔]。 將檔案命名為 *CreateVMTemplate.json*，然後按一下 [新增]。
2. 將此 JSON 程式碼新增到您建立的檔案中：

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "adminUsername": { "type": "string" },
        "adminPassword": { "type": "securestring" }
      },
      "variables": {
        "vnetID": "[resourceId('Microsoft.Network/virtualNetworks','myVNet')]", 
        "subnetRef": "[concat(variables('vnetID'),'/subnets/mySubnet')]", 
      },
      "resources": [
        {
          "apiVersion": "2016-03-30",
          "type": "Microsoft.Network/publicIPAddresses",
          "name": "myPublicIPAddress",
          "location": "[resourceGroup().location]",
          "properties": {
            "publicIPAllocationMethod": "Dynamic",
            "dnsSettings": {
              "domainNameLabel": "myresourcegroupdns1"
            }
          }
        },
        {
          "apiVersion": "2016-03-30",
          "type": "Microsoft.Network/virtualNetworks",
          "name": "myVNet",
          "location": "[resourceGroup().location]",
          "properties": {
            "addressSpace": { "addressPrefixes": [ "10.0.0.0/16" ] },
            "subnets": [
              {
                "name": "mySubnet",
                "properties": { "addressPrefix": "10.0.0.0/24" }
              }
            ]
          }
        },
        {
          "apiVersion": "2016-03-30",
          "type": "Microsoft.Network/networkInterfaces",
          "name": "myNic",
          "location": "[resourceGroup().location]",
          "dependsOn": [
            "[resourceId('Microsoft.Network/publicIPAddresses/', 'myPublicIPAddress')]",
            "[resourceId('Microsoft.Network/virtualNetworks/', 'myVNet')]"
          ],
          "properties": {
            "ipConfigurations": [
              {
                "name": "ipconfig1",
                "properties": {
                  "privateIPAllocationMethod": "Dynamic",
                  "publicIPAddress": { "id": "[resourceId('Microsoft.Network/publicIPAddresses','myPublicIPAddress')]" },
                  "subnet": { "id": "[variables('subnetRef')]" }
                }
              }
            ]
          }
        },
        {
          "apiVersion": "2016-04-30-preview",
          "type": "Microsoft.Compute/virtualMachines",
          "name": "myVM",
          "location": "[resourceGroup().location]",
          "dependsOn": [
            "[resourceId('Microsoft.Network/networkInterfaces/', 'myNic')]"
          ],
          "properties": {
            "hardwareProfile": { "vmSize": "Standard_DS1" },
            "osProfile": {
              "computerName": "myVM",
              "adminUsername": "[parameters('adminUsername')]",
              "adminPassword": "[parameters('adminPassword')]"
            },
            "storageProfile": {
              "imageReference": {
                "publisher": "MicrosoftWindowsServer",
                "offer": "WindowsServer",
                "sku": "2012-R2-Datacenter",
                "version": "latest"
              },
              "osDisk": {
                "name": "myManagedOSDisk",
                "caching": "ReadWrite",
                "createOption": "FromImage"
              }
            },
            "networkProfile": {
              "networkInterfaces": [
                {
                  "id": "[resourceId('Microsoft.Network/networkInterfaces','myNic')]"
                }
              ]
            }
          }
        }
      ]
    }
    ```

3. 儲存 CreateVMTemplate.json 檔案。

### <a name="create-the-parameters-file"></a>建立參數檔案

若要為範本中所定義的資源參數指定值，您需建立包含這些值的參數檔案。

1. 在 [方案總管] 中，於 [myDotnetProject] 上按一下滑鼠右鍵 > [新增] > [新增項目]，然後選取 [Visual C# 項目] 中的 [文字檔]。 將檔案命名為 *Parameters.json*，然後按一下 [新增]。
2. 將此 JSON 程式碼新增到您建立的檔案中：

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "adminUserName": { "value": "azureuser" },
        "adminPassword": { "value": "Azure12345678" }
      }
    }
    ```

4. 儲存 Parameters.json 檔案。

### <a name="create-the-authorization-file"></a>建立授權檔

在部署範本之前，請先確定您可以存取 [Active Directory 服務主體](../../resource-group-authenticate-service-principal.md)。 從服務主體中，您會取得向 Azure Resource Manager 驗證要求的權杖。 您還應該記錄授權檔中所需的應用程式識別碼、驗證金鑰及租用戶識別碼。

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
4. 在 Windows 中名為 AZURE_AUTH_LOCATION 的環境變數上，設定您所建立之授權檔的完整路徑，例如，可以使用下列 PowerShell 命令：

    ```
    [Environment]::SetEnvironmentVariable("AZURE_AUTH_LOCATION", "C:\Visual Studio 2017\Projects\myDotnetProject\myDotnetProject\azureauth.properties", "User")
    ```
    
## <a name="create-the-management-client"></a>建立管理用戶端

1. 開啟您建立之專案的 Program.cs 檔案，然後將這些 using 陳述式新增至檔案頂端的現有陳述式：

    ```
    using Microsoft.Azure.Management.Compute.Fluent;
    using Microsoft.Azure.Management.Compute.Fluent.Models;
    using Microsoft.Azure.Management.Fluent;
    using Microsoft.Azure.Management.ResourceManager.Fluent;
    using Microsoft.Azure.Management.ResourceManager.Fluent.Core;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Blob;
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

## <a name="create-a-resource-group"></a>建立資源群組

若要指定應用程式的值，請將以下程式碼新增到 Main 方法：

```
var groupName = "myResourceGroup";
var location = Region.USWest;

var resourceGroup = azure.ResourceGroups.Define(groupName)
    .WithRegion(location)
    .Create();
```

## <a name="create-a-storage-account"></a>建立儲存體帳戶

部署範本和參數時，會從 Azure 中的儲存體帳戶部署。 在此步驟中，您需建立帳戶並上傳檔案。 

若要建立帳戶，請將以下程式碼新增到 Main 方法：

```
string storageAccountName = SdkContext.RandomResourceName("st", 10);

Console.WriteLine("Creating storage account...");
var storage = azure.StorageAccounts.Define(storageAccountName)
    .WithRegion(Region.USWest)
    .WithExistingResourceGroup(resourceGroup)
    .Create();

var storageKeys = storage.GetKeys();
string storageConnectionString = "DefaultEndpointsProtocol=https;"
    + "AccountName=" + storage.Name
    + ";AccountKey=" + storageKeys[0].Value
    + ";EndpointSuffix=core.windows.net";

var account = CloudStorageAccount.Parse(storageConnectionString);
var serviceClient = account.CreateCloudBlobClient();

Console.WriteLine("Creating container...");
var container = serviceClient.GetContainerReference("templates");
container.CreateIfNotExistsAsync().Wait();
var containerPermissions = new BlobContainerPermissions()
    { PublicAccess = BlobContainerPublicAccessType.Container };
container.SetPermissionsAsync(containerPermissions).Wait();

Console.WriteLine("Uploading template file...");
var templateblob = container.GetBlockBlobReference("CreateVMTemplate.json");
templateblob.UploadFromFileAsync("..\\..\\CreateVMTemplate.json").Result();

Console.WriteLine("Uploading parameters file...");
var paramblob = container.GetBlockBlobReference("Parameters.json");
paramblob.UploadFromFileAsync("..\\..\\Parameters.json").Result();
```

## <a name="deploy-the-template"></a>部署範本

從已建立的儲存體帳戶部署範本和參數。 

若要部署範本，請將以下程式碼新增到 Main 方法：

```
var templatePath = "https://" + storageAccountName + ".blob.core.windows.net/templates/CreateVMTemplate.json";
var paramPath = "https://" + storageAccountName + ".blob.core.windows.net/templates/Parameters.json";
var deployment = azure.Deployments.Define("myDeployment")
    .WithExistingResourceGroup(groupName)
    .WithTemplateLink(templatePath, "1.0.0.0")
    .WithParametersLink(paramPath, "1.0.0.0")
    .WithMode(Microsoft.Azure.Management.ResourceManager.Fluent.Models.DeploymentMode.Incremental)
    .Create();
Console.WriteLine("Press enter to delete the resource group...");
Console.ReadLine();
```

## <a name="delete-the-resources"></a>刪除資源

由於您需要為在 Azure 中使用的資源付費，因此刪除不再需要的資源一律是理想的做法。 您不需要從資源群組個別刪除每個資源。 刪除資源群組，其所有資源都將會自動刪除。 

若要刪除資源群組，請將以下程式碼新增到 Main 方法：

```
azure.ResourceGroups.DeleteByName(groupName);
```

## <a name="run-the-application"></a>執行應用程式

此主控台應用程式從開始到完成的完整執行應該需要五分鐘左右。 

1. 若要執行主控台應用程式，請按一下 [啟動]。

2. 在您按 **Enter** 以開始刪除資源之前，可以先花幾分鐘的時間來確認 Azure 入口網站中的資源建立情況。 請按一下部署狀態來查看該項部署的相關資訊。

## <a name="next-steps"></a>後續步驟
* 如果部署有問題，下一個步驟就是查看[使用 Azure Resource Manager 針對常見的 Azure 部署錯誤進行疑難排解](../../resource-manager-common-deployment-errors.md)。
* 檢閱[使用 C# 來部署 Azure 虛擬機器](csharp.md)，以了解如何部署虛擬機器及支援它的資源。
