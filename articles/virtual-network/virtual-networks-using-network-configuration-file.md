---
title: 設定 Azure 虛擬網路 (傳統) - 網路設定檔 | Microsoft Docs
description: 了解如何透過匯出、變更及匯入網路設定檔，來建立及修改虛擬網路 (傳統)。
services: virtual-network
documentationcenter: ''
author: genlin
manager: cshepard
editor: ''
tags: azure-service-management
ms.assetid: c29b9059-22b0-444e-bbfe-3e35f83cde2f
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/23/2017
ms.author: genli
ms.custom: ''
ms.openlocfilehash: 5267ce5c50e8a57843f0bc54165289b38013ad91
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46986110"
---
# <a name="configure-a-virtual-network-classic-using-a-network-configuration-file"></a>使用網路設定檔來設定虛擬網路 (傳統)
> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有二種： [資源管理員和傳統](../resource-manager-deployment-model.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署都使用 Resource Manager 部署模型。

您可以使用 Azure 傳統命令列介面 (CLI) 或 Azure PowerShell，透過網路設定檔建立及設定虛擬網路 (傳統)。 您無法使用網路設定檔並透過 Azure Resource Manager 部署模型，而建立或修改虛擬網路。 您無法使用 Azure 入口網站來使用網路設定檔建立或修改虛擬網路 (傳統)，但是您可以使用 Azure 入口網站在不使用網路設定檔的情況下建立虛擬網路 (傳統)。

使用網路設定檔建立及設定虛擬網路 (傳統) 需要匯出、變更及匯入檔案。

## <a name="export"></a>匯出網路設定檔

您可以使用 PowerShell 或 Azure 傳統 CLI 來匯出網路設定檔。 PowerShell 會匯出 XML 檔案，而 Azure 傳統 CLI 會匯出 json 檔案。

### <a name="powershell"></a>PowerShell
 
1. [安裝 Azure PowerShell 並登入 Azure](https://docs.microsoft.com/azure/azure-stack/azure-stack-powershell-install)。
2. 視需要變更下列命令中的目錄 (並確保它存在) 與檔案名稱，然後執行命令以匯出網路設定檔：

    ```powershell
    Get-AzureVNetConfig -ExportToFile c:\azure\networkconfig.xml
    ```

### <a name="azure-classic-cli"></a>Azure 傳統 CLI

1. [安裝 Azure 傳統 CLI](../cli-install-nodejs.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。 從傳統 CLI 命令提示字元完成剩餘步驟。
2. 輸入 `azure login` 命令來登入 Azure。
3. 透過輸入 `azure config mode asm` 命令確保您處於 asm 模式。
4. 視需要變更下列命令中的目錄 (並確保它存在) 與檔案名稱，然後執行命令以匯出網路設定檔：
    
    ```azurecli
    azure network export c:\azure\networkconfig.json
    ```

## <a name="create-or-modify-a-network-configuration-file"></a>建立或修改網路設定檔

網路設定檔是 XML 檔案 (使用 PowerShell 時) 或 json 檔案 (使用傳統 CLI 時)。 您可以在任何文字或 XML/json 編輯器中編輯檔案。 [網路設定檔結構描述設定](https://msdn.microsoft.com/library/azure/jj157100.aspx)一文包含所有設定的詳細資料。 如需設定的其他說明，請參閱[檢視虛擬網路與設定](manage-virtual-network.md#view-virtual-networks-and-settings)。 您對檔案進行的變更：

- 必須符合結構描述，否則匯入網路設定檔將會失敗。
- 會覆寫您訂用帳戶的任何現有網路設定，因此在進行修改時請特別小心。 例如，參考以下範例網路設定檔。 假設原始檔案包含兩個 **VirtualNetworkSite** 執行個體，而您變更了原始檔案，如範例所示。 匯入檔案時，Azure 會刪除您在檔案中移除之 **VirtualNetworkSite** 執行個體的虛擬網路。 這種簡化的情況假設虛擬網路中沒有資源，因為如果有，將無法刪除虛擬網路，且匯入將會失敗。

> [!IMPORTANT]
> Azure 會將已部署的子網路視為**使用中**。 使用中的子網路無法加以修改。 在修改網路設定檔中的子網路資訊之前，請先將已部署到該子網路的所有項目移到不修改的不同子網路。 如需詳細資訊，請參閱[將 VM 或角色執行個體移至不同的子網路](virtual-networks-move-vm-role-to-subnet.md)。

### <a name="example-xml-for-use-with-powershell"></a>與 PowerShell 搭配使用的範例 XML

下列範例網路設定檔會在美國東部 Azure 區域中建立名為 *myVirtualNetwork* 的虛擬網路，位址空間為 *10.0.0.0/16*。 該虛擬網路包含一個名為 *mySubnet*，且位址首碼為 *10.0.0.0/24* 的子網路。

```xml
<?xml version="1.0" encoding="utf-8"?>
<NetworkConfiguration xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/ServiceHosting/2011/07/NetworkConfiguration">
  <VirtualNetworkConfiguration>
    <Dns />
    <VirtualNetworkSites>
      <VirtualNetworkSite name="myVirtualNetwork" Location="East US">
        <AddressSpace>
          <AddressPrefix>10.0.0.0/16</AddressPrefix>
        </AddressSpace>
        <Subnets>
          <Subnet name="mySubnet">
            <AddressPrefix>10.0.0.0/24</AddressPrefix>
          </Subnet>
        </Subnets>
      </VirtualNetworkSite>
    </VirtualNetworkSites>
  </VirtualNetworkConfiguration>
</NetworkConfiguration>
```

如果您匯出的網路設定檔不包含任何內容，則可以複製上一個範例中的 XML，並將它貼上到新檔案中。

### <a name="example-json-for-use-with-the-classic-cli"></a>搭配傳統 CLI 使用的範例 JSON

下列範例網路設定檔會在美國東部 Azure 區域中建立名為 *myVirtualNetwork* 的虛擬網路，位址空間為 *10.0.0.0/16*。 該虛擬網路包含一個名為 *mySubnet*，且位址首碼為 *10.0.0.0/24* 的子網路。

```json
{
   "VirtualNetworkConfiguration" : {
      "Dns" : "",
      "VirtualNetworkSites" : [
         {
            "AddressSpace" : [ "10.0.0.0/16" ],
            "Location" : "East US",
            "Name" : "myVirtualNetwork",
            "Subnets" : [
               {
                  "AddressPrefix" : "10.0.0.0/24",
                  "Name" : "mySubnet"
               }
            ]
         }
      ]
   }
}
```

如果您匯出的網路設定檔不包含任何內容，則可以複製上一個範例中的 json，並將它貼上到新檔案中。

## <a name="import"></a>匯入網路設定檔

您可以使用 PowerShell 或傳統 CLI 來匯入網路設定檔。 PowerShell 會匯入 XML 檔案，而傳統 CLI 會匯入 json 檔案。 如果匯入失敗，請確認檔案符合[網路設定結構描述](https://msdn.microsoft.com/library/azure/jj157100.aspx)。 

### <a name="powershell"></a>PowerShell
 
1. [安裝 Azure PowerShell 並登入 Azure](https://docs.microsoft.com/azure/azure-stack/azure-stack-powershell-install)。
2. 視需要變更下列命令中的目錄與檔案名稱，然後執行命令以匯入網路設定檔：
 
    ```powershell
    Set-AzureVNetConfig  -ConfigurationPath c:\azure\networkconfig.xml
    ```

### <a name="azure-classic-cli"></a>Azure 傳統 CLI

1. [安裝 Azure 傳統 CLI](/cli/azure/install-cli-version-1.0.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。 從傳統 CLI 命令提示字元完成剩餘步驟。
2. 輸入 `azure login` 命令來登入 Azure。
3. 透過輸入 `azure config mode asm` 命令確保您處於 asm 模式。
4. 視需要變更下列命令中的目錄與檔案名稱，然後執行命令以匯入網路設定檔：

    ```azurecli
    azure network import c:\azure\networkconfig.json
    ```
