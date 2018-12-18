---
title: 在 Azure 中的 Windows VM 上安裝 Symantec Endpoint Protection | Microsoft Docs
description: 了解如何在以傳統部署模型所建立新的或現有的 Azure VM 上安裝和設定 Symantec Endpoint Protection 安全性延伸模組。
services: virtual-machines-windows
documentationcenter: ''
author: roiyz
manager: jeconnoc
editor: ''
tags: azure-service-management
ms.assetid: 19dcebc7-da6b-4510-907b-d64088e81fa2
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-multiple
ms.devlang: na
ms.topic: article
ms.date: 03/31/2017
ms.author: roiyz
ms.openlocfilehash: 455ca0ed1de5df3a184d900aeae286dfd5233a60
ms.sourcegitcommit: d16b7d22dddef6da8b6cfdf412b1a668ab436c1f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39716503"
---
# <a name="how-to-install-and-configure-symantec-endpoint-protection-on-a-windows-vm"></a>如何在 Windows VM 上安裝和設定 Symantec Endpoint Protection
> [!IMPORTANT] 
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../azure-resource-manager/resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。

本文說明如何在執行 Windows Server 的現有虛擬機器 (VM) 上，安裝和設定 Symantec Endpoint Protection 用戶端。 此完整用戶端包括服務 (例如病毒和間諜軟體防護、防火牆及入侵防禦)。 透過使用 VM 代理程式，用戶端會安裝為安全性延伸模組。

如果您已有 Symantec 的內部部署解決方案現有訂用帳戶，您可以用它來保護 Azure 虛擬機器的安全。 如果您還不是 Symantec 客戶，您可以註冊試用訂用帳戶。 如需此解決方案的詳細資訊，請參閱 [Microsoft Azure 平台上的 Symantec Endpoint Protection][Symantec]。 如果您已經是 Symantec 客戶，此頁面還提供授權資訊的連結，以及安裝用戶端的指示。

## <a name="install-symantec-endpoint-protection-on-an-existing-vm"></a>在現有 VM 上安裝 Symantec Endpoint Protection
在開始之前，您需要下列項目：

* 在本機電腦上安裝 Azure PowerShell 模組 0.8.2 版或更新版本。 您可以使用 **Get-Module azure | format-table version** 命令檢查已安裝的 Azure PowerShell 版本。 如需最新版本的指示與連結，請參閱[如何安裝和設定 Azure PowerShell][PS]。 使用 `Add-AzureAccount`登入您的 Azure 訂用帳戶。
* 在 Azure 虛擬機器上執行的 VM 代理程式。

首先，請確認虛擬機器上已安裝 VM 代理程式。 填寫雲端服務名稱和虛擬機器名稱，然後在系統管理員層級 Azure PowerShell 命令提示字元上執行下列命令。 取代括弧內 (包括 < 和 > 字元) 的所有項目。

> [!TIP]
> 如果您不知道雲端服務和虛擬機器的名稱，請執行 **Get-AzureVM** 以列出您目前訂用帳戶中的所有虛擬機器的名稱。

```powershell
$CSName = "<cloud service name>"
$VMName = "<virtual machine name>"
$vm = Get-AzureVM -ServiceName $CSName -Name $VMName
write-host $vm.VM.ProvisionGuestAgent
```

如果 **write-host** 命令顯示 **True**，則會安裝 VM 代理程式。 如果顯示 [False]，請參閱 Azure 部落格文章 [VM 代理程式與擴充功能 - 第 2 部分][Agent]中的指示和下載連結。

如果已安裝 VM 代理程式，請執行下列命令來安裝 Symantec Endpoint Protection 代理程式。

```powershell
$Agent = Get-AzureVMAvailableExtension -Publisher Symantec -ExtensionName SymantecEndpointProtection

Set-AzureVMExtension -Publisher Symantec –Version $Agent.Version -ExtensionName SymantecEndpointProtection \
    -VM $vm | Update-AzureVM
```

若要確認 Symantec 安全性延伸模組已安裝且是最新的：

1. 登入虛擬機器。 如需指示，請參閱[如何登入執行 Windows Server 的虛擬機器][Logon]。
2. 在 Windows Server 2008 R2 中，按一下 [開始] > [Symantec Endpoint Protection]。 在 Windows Server 2012 或 Windows Server 2012 R2 的 [開始] 畫面中，輸入 **Symantec**，然後按一下 [Symantec Endpoint Protection]。
3. 在 [狀態 - Symantec Endpoint Protection] 視窗的 [狀態] 索引標籤中，套用更新或視需要重新啟動。

## <a name="additional-resources"></a>其他資源
[如何登入執行 Windows Server 的虛擬機器][Logon]

[Azure VM 擴充功能與功能][Ext]

<!--Link references-->
[Symantec]: http://www.symantec.com/connect/blogs/symantec-endpoint-protection-now-microsoft-azure

[Create]:../windows/classic/tutorial.md

[PS]: /powershell/azureps-cmdlets-docs

[Agent]: http://go.microsoft.com/fwlink/p/?LinkId=403947

[Logon]:../windows/classic/connect-logon.md

[Ext]: http://go.microsoft.com/fwlink/p/?linkid=390493
