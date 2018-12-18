---
title: Azure Stack 中虛擬機器的差異與考量 | Microsoft Docs
description: 了解使用 Azure Stack 中虛擬機器時的差異與考量。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.assetid: 6613946D-114C-441A-9F74-38E35DF0A7D7
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/05/2018
ms.author: mabrigg
ms.openlocfilehash: 30cdb07f8e0d5481f34148b99fa86f2a1bb85e0b
ms.sourcegitcommit: d211f1d24c669b459a3910761b5cacb4b4f46ac9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "44022191"
---
# <a name="considerations-for-using-virtual-machines-in-azure-stack"></a>Azure Stack 中使用虛擬機器的考量

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

Azure Stack 虛擬機器提供隨選、可調整的計算資源。 部署虛擬機器 (VM) 之前，必須先了解 Azure Stack 與 Microsoft Azure 所提供的虛擬機器在功能上的差異。 本文會說明這些差異，並指明規劃虛擬機器部署方面的主要考量。 若要深入了解 Azure Stack 與 Azure 之間的大致差異，請參閱[主要考量](azure-stack-considerations.md)文章。

## <a name="cheat-sheet-virtual-machine-differences"></a>速查表：虛擬機器差異

| 功能 | Azure (全域) | Azure Stack |
| --- | --- | --- |
| 虛擬機器映像 | Azure Marketplace 包含您可用來建立虛擬機器的映像。 若要檢視 Azure Marketplace 中可用的映像清單，請參閱 [Azure Marketplace](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/category/compute?subcategories=virtual-machine-images&page=1) 頁面。 | Azure Stack 市集中預設沒有提供任何映像。 Azure Stack 雲端系統管理員應該先將映像發行或下載到 Azure Stack 市集，使用者才能使用這些映像。 |
| 虛擬機器大小 | Azure 支援各種不同的虛擬機器大小。 若要了解可用的大小和選項，請參考 [Windows 虛擬機器大小](../../virtual-machines/virtual-machines-windows-sizes.md)和 [Linux 虛擬機器大小](../../virtual-machines/linux/sizes.md)主題。 | Azure Stack 支援 Azure 中一部分可用的虛擬機器大小。 若要檢視支援的大小清單，請參考此文章的[虛擬機器大小](#virtual-machine-sizes)一節。 |
| 虛擬機器配額 | [配額限制](../../azure-subscription-service-limits.md#service-specific-limits)由 Microsoft 設定 | Azure Stack 雲端系統管理員在提供虛擬機器給其使用者之前，必須先指派配額。 |
| 虛擬機器擴充功能 |Azure 支援各種不同的虛擬機器擴充功能。 若要了解可用的擴充功能，請參考[虛擬機器擴充功能和功能](../../virtual-machines/windows/extensions-features.md)文章。| Azure Stack 支援 Azure 中一部分可用的擴充功能，且每個擴充功能都有特定的版本。 Azure Stack 雲端系統管理員可以選擇要將哪些擴充功能提供給其使用者使用。 若要檢視支援的擴充功能清單，請參考此文章的[虛擬機器擴充功能](#virtual-machine-extensions)一節。 |
| 虛擬機器網路 | 指派給租用戶虛擬機器的公用 IP 位址可透過網際網路存取。<br><br><br>Azure 虛擬機器有固定的 DNS 名稱 | 指派給租用戶虛擬機器的公用 IP 位址時，只能在「Azure Stack 開發套件」環境內存取。 使用者必須能夠透過 [RDP](azure-stack-connect-azure-stack.md#connect-to-azure-stack-with-remote-desktop) 或 [VPN](azure-stack-connect-azure-stack.md#connect-to-azure-stack-with-vpn) 存取「Azure Stack 開發套件」，才能連線到在 Azure Stack 中建立的虛擬機器。<br><br>在特定 Azure Stack 執行個體內建立之虛擬機器的 DNS 名稱會以雲端系統管理員所設定的值為基礎。 |
| 虛擬機器儲存體 | 支援[受控磁碟](../../virtual-machines/windows/managed-disks-overview.md)。 | Azure Stack 版本 1808 和更新版本可支援受控磁碟。 |
| API 版本 | Azure 一律會有所有虛擬機器功能的最新 API 版本。 | Azure Stack 支援特定的 Azure 服務及這些服務的特定 API 版本。 若要檢視支援的 API 版本清單，請參考此文章的 [API 版本](#api-versions)一節。 |
|虛擬機器可用性設定組|多個容錯網域 (每一區域 2 或 3 個)<br>多個更新網域<br>受控磁碟支援|多個容錯網域 (每一區域 2 或 3 個)<br>多個更新網域 (最多 20 個)<br>無受控磁碟支援|
|虛擬機器擴展集|支援自動調整|不支援自動調整。<br>使用入口網站、Resource Manager 範本或 PowerShell 將更多執行個體新增到擴展集。

## <a name="virtual-machine-sizes"></a>虛擬機器大小

Azure Stack 會施加資源限制，來避免資源 (伺服器本機和服務層級) 的過度耗用。這類限制會降低其他租用戶耗用資源所造成的影響，進而改善租用戶體驗。

- 虛擬機器的網路輸出已有頻寬上限。 Azure Stack 的上限與 Azure 的上限相同。
- 至於儲存體資源，Azure Stack 會實作儲存體 IOPS 限制，以避免租用戶為了存取儲存體而造成基本的資源過度耗用。
- 針對具有多個連結資料磁碟的虛擬機器，每個資料磁碟的最大輸送量是 500 IOPS (適用於 HDD) 和 2300 IOPS (適用於 SSD)。

下表列出 Azure Stack 所支援的虛擬機器及其設定：

| 類型           | 大小          | 支援的大小範圍 |
| ---------------| ------------- | ------------------------ |
|一般用途 |基本 A        |[A0 - A4](azure-stack-vm-sizes.md#basic-a)                   |
|一般用途 |標準 A     |[A0 - A7](azure-stack-vm-sizes.md#standard-a)              |
|一般用途 |D 系列       |[D1 - D4](azure-stack-vm-sizes.md#d-series)              |
|一般用途 |Dv2 系列     |[D1_v2 - D5_v2](azure-stack-vm-sizes.md#ds-series)        |
|一般用途 |DS 系列      |[DS1 - DS4](azure-stack-vm-sizes.md#dv2-series)            |
|一般用途 |DSv2 系列    |[DS1_v2 - DS5_v2](azure-stack-vm-sizes.md#dsv2-series)      |
|記憶體最佳化|D 系列       |[D11 - D14](azure-stack-vm-sizes.md#mo-d)            |
|記憶體最佳化|DS 系列      |[DS11 - DS14](azure-stack-vm-sizes.md#mo-ds)|
|記憶體最佳化|Dv2 系列     |[D11_v2 - DS14_v2](azure-stack-vm-sizes.md#mo-dv2)     |
|記憶體最佳化|DSv2-系列 -  |[DS11_v2 - DS14_v2](azure-stack-vm-sizes.md#mo-dsv2)    |

Azure Stack 與 Azure 之間的虛擬機器大小及其關聯資源數量是一致的。 包括記憶體數量、核心數目，以及可建立的資料磁碟數目/大小。 不過，相同大小 VM 的效能取決於特定 Azure Stack 環境的底層特性。

## <a name="virtual-machine-extensions"></a>虛擬機器擴充功能

 Azure Stack 包含一小組擴充功能。 您可以透過 Marketplace 摘要整合取得更新和額外的擴充功能。

使用下列 PowerShell 指令碼，來取得您 Azure Stack 環境中所提供虛擬機器擴充功能的清單：

```powershell
Get-AzureRmVmImagePublisher -Location local | `
  Get-AzureRmVMExtensionImageType | `
  Get-AzureRmVMExtensionImage | `
  Select Type, Version | `
  Format-Table -Property * -AutoSize
```

## <a name="api-versions"></a>API 版本

Azure Stack 中的虛擬機器功能支援下列 API 版本：

![VM 資源類型](media/azure-stack-vm-considerations/vm-resoource-types.png)

您可以使用下列 PowerShell 指令碼，來取得您 Azure Stack 環境中所提供虛擬機器功能的 API 版本：

```powershell
Get-AzureRmResourceProvider | `
  Select ProviderNamespace -Expand ResourceTypes | `
  Select * -Expand ApiVersions | `
  Select ProviderNamespace, ResourceTypeName, @{Name="ApiVersion"; Expression={$_}} | `
  where-Object {$_.ProviderNamespace -like “Microsoft.compute”}
```

如果雲端操作員將您的 Azure Stack 環境更新成較新的版本，所支援資源類型和 API 版本的清單可能會有所不同。

## <a name="windows-activation"></a>Windows 啟用

Windows 產品的使用必須使用符合產品使用權及 Microsoft 授權條款。 Azure Stack 會使用[自動虛擬機器啟用](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn303421(v%3dws.11)) (AVMA) 來啟動 Windows Server 虛擬機器 (VM)。

- Azure Stack 主機會使用 Windows Server 2016 的 AVMA 金鑰啟動 Windows。 所有執行 Windows Server 2012 或更新版本的 VM 均會自動啟動。
- 執行 Windows Server 2008 R2 的 VM 不會自動啟用，且必須使用 [MAK 啟用](https://technet.microsoft.com/library/ff793438.aspx)加以啟動。 若要使用 MAK 啟用，您必須提供您自己的產品金鑰。

Microsoft Azure 會使用 KMS 啟用來啟動 Windows VM。 將 VM 從 Azure Stack 移至 Azure 時，若遇到啟動方面的問題，請參閱[針對 Azure Windows 虛擬機器啟用問題進行疑難排解](https://docs.microsoft.com/azure/virtual-machines/windows/troubleshoot-activation-problems)。 如需其他資訊，請參閱 Azure 支援小組的部落格文章：[針對 Azure VM 上的 Windows 啟用失敗進行疑難排解](https://blogs.msdn.microsoft.com/mast/2017/06/14/troubleshooting-windows-activation-failures-on-azure-vms/) (英文)。

## <a name="next-steps"></a>後續步驟

[在 Azure Stack 中使用 PowerShell 來建立 Windows 虛擬機器](azure-stack-quick-create-vm-windows-powershell.md)
