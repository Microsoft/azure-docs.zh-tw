---
title: 管理 HPC Pack 叢集計算節點 |Microsoft Docs
description: 了解可在 Azure 中新增、移除、啟動和停止 HPC Pack 2012 R2 叢集計算節點的 PowerShell 指令碼工具
services: virtual-machines-windows
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
tags: azure-service-management,hpc-pack
ms.assetid: 4193f03b-94e9-4704-a7ad-379abde063a9
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-multiple
ms.workload: big-compute
ms.date: 12/29/2016
ms.author: danlep
ms.openlocfilehash: 453f53be15b24b96f183b4935cc45fc97ad058bd
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
---
# <a name="manage-the-number-and-availability-of-compute-nodes-in-an-hpc-pack-cluster-in-azure"></a>在 Azure 的 HPC Pack 叢集中管理計算節點的數目和可用性
如果您已在 Azure VM 中建立 HPC Pack 2012 R2 叢集，您可能會需要可輕易地在叢集中新增、移除、啟動 (佈建) 或停止 (解除佈建) 一些計算節點 VM 的方法。 若要執行這些工作，請執行安裝在前端節點 VM 上的 Azure PowerShell 指令碼。 這些指令碼可協助您控制 HPC Pack 叢集資源的數目和可用性，讓您得以控制成本。

> [!IMPORTANT] 
> 本文適用於 Azure 中使用傳統部署模型建立的 HPC Pack 2012 R2 叢集。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。
> 此外，本文中所述的 PowerShell 指令碼不適用於 HPC Pack 2016。

## <a name="prerequisites"></a>先決條件
* **Azure VM 中的 HPC Pack 2012 R2 叢集**：在傳統部署模型中建立 HPC Pack 2012 R2 叢集。 例如，您可以使用 Azure Marketplace 中的 HPC Pack 2012 R2 VM 映像和 Azure PowerShell 指令碼，將部署自動化。 如需相關資訊和必要條件，請參閱[使用 HPC Pack IaaS 部署指令碼建立 HPC 叢集](hpcpack-cluster-powershell-script.md)。
  
    部署之後，會在前端節點的 %CCP\_HOME%bin 資料夾中發現節點管理指令碼。 以系統管理員身分執行每個指令碼。
* **Azure 發佈設定檔或管理憑證**：您必須前端節點上執行下列其中一項：
  
  * **匯入 Azure 發佈設定檔**。 若要這麼做，請在前端節點上執行下列 Azure PowerShell Cmdlet：
    
    ```PowerShell
    Get-AzurePublishSettingsFile
    
    Import-AzurePublishSettingsFile –PublishSettingsFile <publish settings file>
    ```
  * **前端節點上設定 Azure 管理憑證**。 如果您有 .cer 檔案，請在 CurrentUser\My certificate store 中將其匯入，並為您的 Azure 環境 (AzureCloud 或 AzureChinaCloud) 執行下列 Azure PowerShell Cmdlet：
    
    ```PowerShell
    Set-AzureSubscription -SubscriptionName <Sub Name> -SubscriptionId <Sub ID> -Certificate (Get-Item Cert:\CurrentUser\My\<Cert Thrumbprint>) -Environment <AzureCloud | AzureChinaCloud>
    ```

## <a name="add-compute-node-vms"></a>新增計算節點 VM
使用 **Add-HpcIaaSNode.ps1** 指令碼新增計算節點。

### <a name="syntax"></a>語法
```PowerShell
Add-HPCIaaSNode.ps1 [-ServiceName] <String> [-ImageName] <String>
 [-Quantity] <Int32> [-InstanceSize] <String> [-DomainUserName] <String> [[-DomainUserPassword] <String>]
 [[-NodeNameSeries] <String>] [<CommonParameters>]

```
### <a name="parameters"></a>參數
* **ServiceName**：會新增計算節點 VM 之雲端服務的名稱。
* **ImageName**：Azure VM 映像名稱，透過 Azure 入口網站或 Azure PowerShell Cmdlet **Get-AzureVMImage** 可以取得此名稱。 這些映像必須符合下列需求：
  
  1. 必須安裝 Windows 作業系統。
  2. 必須在計算節點角色中安裝 HPC Pack。
  3. 映像必須是使用者類別中的私人映像，而不是公用 Azure VM 映像。
* **Quantity**：要新增的計算節點 VM 數目。
* **InstanceSize**：計算節點 VM 的大小。
* **DomainUserName**：網域使用者名稱，用來將新的 VM 加入網域中。
* **DomainUserPassword**：網域使用者的密碼。
* **NodeNameSeries** (選用)：計算節點的命名模式。 格式必須為 &lt;*Root\_Name*&gt;&lt;*Start\_Number*&gt;%。 例如，MyCN%10% 表示從 MyCN11 開始的一系列計算節點名稱。 如果未指定，指令碼會使用 HPC 叢集中已設定的節點命名序列。

### <a name="example"></a>範例
下列範例會根據 VM 映像 *hpccnimage1*，在雲端服務 *hpcservice1* 中新增 20 個大型計算節點 VM。

```PowerShell
Add-HPCIaaSNode.ps1 –ServiceName hpcservice1 –ImageName hpccniamge1
–Quantity 20 –InstanceSize Large –DomainUserName <username>
-DomainUserPassword <password>
```


## <a name="remove-compute-node-vms"></a>移除計算節點 VM
使用 **Remove-HpcIaaSNode.ps1** 指令碼移除計算節點。

### <a name="syntax"></a>語法
```PowerShell
Remove-HPCIaaSNode.ps1 -Name <String[]> [-DeleteVHD] [-Force] [-WhatIf] [-Confirm] [<CommonParameters>]

Remove-HPCIaaSNode.ps1 -Node <Object> [-DeleteVHD] [-Force] [-Confirm] [<CommonParameters>]
```

### <a name="parameters"></a>參數
* **Name**：要移除之叢集節點的名稱。 支援萬用字元。 參數集名稱是 Name。 您無法同時指定 **Name** 和 **Node** 參數。
* **Node**：要移除之節點的 HpcNode 物件，可透過 HPC PowerShell Cmdlet [Get-HpcNode](https://technet.microsoft.com/library/dn887927.aspx)取得。 參數集名稱是 Node。 您無法同時指定 **Name** 和 **Node** 參數。
* **DeleteVHD** (選擇性)：針對已移除的 VM 進行相關磁碟的刪除時所使用的設定。
* **Force** (選擇性)：在移除 HPC 節點前強制使其離線的設定。
* **Confirm** (選擇性)：執行命令前先行確認的提示。
* **WhatIf**：用來說明您所執行的命令未實際執行時將會有何狀況的設定。

### <a name="example"></a>範例
下列範例會使名稱開頭為 *HPCNode-CN-* 的節點強制離線，然後移除節點及其相關聯的磁碟。

```PowerShell
Remove-HPCIaaSNode.ps1 –Name HPCNodeCN-* –DeleteVHD -Force
```

## <a name="start-compute-node-vms"></a>啟動計算節點 VM
使用 **Start-HpcIaaSNode.ps1** 指令碼啟動計算節點。

### <a name="syntax"></a>語法
```PowerShell
Start-HPCIaaSNode.ps1 -Name <String[]> [<CommonParameters>]

Start-HPCIaaSNode.ps1 -Node <Object> [<CommonParameters>]
```
### <a name="parameters"></a>參數
* **Name**：要啟動之叢集節點的名稱。 支援萬用字元。 參數集名稱是 Name。 您無法同時指定 **Name** 和 **Node** 參數。
* **Node**- 要啟動之節點的 HpcNode 物件，可透過 HPC PowerShell Cmdlet [Get-HpcNode](https://technet.microsoft.com/library/dn887927.aspx)取得。 參數集名稱是 Node。 您無法同時指定 **Name** 和 **Node** 參數。

### <a name="example"></a>範例
下列範例會啟動名稱開頭為 *HPCNode-CN-*的節點。

```PowerShell
Start-HPCIaaSNode.ps1 –Name HPCNodeCN-*
```

## <a name="stop-compute-node-vms"></a>停止計算節點 VM
使用 **Stop-HpcIaaSNode.ps1** 指令碼停止計算節點。

### <a name="syntax"></a>語法
```PowerShell
Stop-HPCIaaSNode.ps1 -Name <String[]> [-Force] [<CommonParameters>]

Stop-HPCIaaSNode.ps1 -Node <Object> [-Force] [<CommonParameters>]
```

### <a name="parameters"></a>參數
* **Name**- 要停止之叢集節點的名稱。 支援萬用字元。 參數集名稱是 Name。 您無法同時指定 **Name** 和 **Node** 參數。
* **Node**：要停止之節點的 HpcNode 物件，可透過 HPC PowerShell Cmdlet [Get-HpcNode](https://technet.microsoft.com/library/dn887927.aspx)取得。 參數集名稱是 Node。 您無法同時指定 **Name** 和 **Node** 參數。
* **Force** (選擇性)：在停止 HPC 節點前強制使其離線的設定。

### <a name="example"></a>範例
下列範例會使名稱開頭為 *HPCNode-CN-* 的節點強制離線，然後停止節點。

```PowerShell
Stop-HPCIaaSNode.ps1 –Name HPCNodeCN-* -Force
```

## <a name="next-steps"></a>後續步驟
* 若要根據叢集上目前工作的工作負載，自動增加或縮減叢集節點的方法，請參閱[在 Azure 中根據叢集工作負載自動增加和縮減 HPC Pack 叢集資源](hpcpack-cluster-node-autogrowshrink.md)。

