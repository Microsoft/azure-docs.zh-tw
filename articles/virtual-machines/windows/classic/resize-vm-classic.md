---
title: 針對以傳統部署模型建立的 Windows VM 調整大小 - Azure | Microsoft Docs
description: 使用 Azure Powershell 可調整以傳統部署模型建立的 Windows 虛擬機器大小。
services: virtual-machines-windows
documentationcenter: ''
author: Drewm3
manager: jeconnoc
editor: ''
tags: azure-service-management
ms.assetid: e3038215-001c-406e-904d-e0f4e326a4c7
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 10/19/2016
ms.author: drewm
ms.openlocfilehash: 6afc8125462a659d0892e7dbb53f1f71789815cc
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
ms.locfileid: "30913616"
---
# <a name="resize-a-windows-vm-created-in-the-classic-deployment-model"></a>對以傳統部署模型建立的 Windows VM 調整大小
本文說明如何使用 Azure Powershell 調整以傳統部署模型建立的 Windows VM 大小。

在考慮調整 VM 大小的能力時，有兩個概念可控制調整虛擬機器大小時可用的大小範圍。 第一個概念是 VM 部署所在的區域。 區域中可用的 VM 大小清單可在 [Azure 區域] 網頁上的 [服務] 索引標籤中找到。 第二個概念是目前裝載 VM 的實體硬體。 裝載 VM 的實體伺服器會一起聚集在共同的實體硬體叢集中。 根據所需的新 VM 大小是否受目前裝載 VM 的硬體叢集所支援，會有不同的 VM 大小變更方法。

> [!IMPORTANT] 
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 您也可以[對以 Resource Manager 部署模型建立的 VM 調整大小](../resize-vm.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

## <a name="add-your-account"></a>新增帳戶
您必須設定 Azure PowerShell 來使用傳統 Azure 資源。 請遵循下列步驟，設定 Azure PowerShell 以管理傳統資源。

1. 在 PowerShell 提示字元中，輸入 `Add-AzureAccount`，然後按一下 **Enter** 鍵。 
2. 輸入與您 Azure 訂用帳戶相關聯的電子郵件地址，並按一下 [繼續] 。 
3. 輸入您帳戶的密碼。 
4. 按一下 [ **登入**]。 

## <a name="resize-in-the-same-hardware-cluster"></a>在相同的硬體叢集中調整大小
若要將 VM 的大小調整為裝載 VM 的硬體叢集中可用的大小，請執行下列步驟。

1. 執行下列 PowerShell 命令，列出裝載 VM 所在雲端服務的硬體叢集中可用的 VM 大小。
   
    ```powershell
    Get-AzureService | where {$_.ServiceName -eq "<cloudServiceName>"}
    ```
2. 執行下列命令來調整 VM 的大小。
   
    ```powershell
    Get-AzureVM -ServiceName <cloudServiceName> -Name <vmName> | Set-AzureVMSize -InstanceSize <newVMSize> | Update-AzureVM
    ```

## <a name="resize-on-a-new-hardware-cluster"></a>在新的硬體叢集上調整大小
若要將 VM 的大小調整為裝載 VM 的硬體叢集中不適用的大小，雲端服務和雲端服務中的所有 VM 都必須重新建立。 每個雲端服務都會裝載在單一硬體叢集中，因此雲端服務中所有 VM 的大小都必須是硬體叢集可支援的大小。 下列步驟會說明如何刪除再重新建立雲端服務，以調整 VM 的大小。

1. 執行下列 PowerShell 命令，列出區域中可用的 VM 大小。 
   
    ```powershell
    Get-AzureLocation | where {$_.Name -eq "<locationName>"}
    ```
2. 記下所含 VM 要調整大小的雲端服務中每個 VM 的所有組態設定。 
3. 在已選取選項來保留每個 VM 之磁碟的情況下，刪除雲端服務中的所有 VM。
4. 使用所需的 VM 大小重新建立要調整大小的 VM。
5. 使用現在裝載雲端服務之硬體叢集中可用的 VM 大小，重新建立雲端服務中先前擁有的其他所有 VM。

您可以在[這裡](https://github.com/Azure/azure-vm-scripts)找到刪除雲端服務並使用新 VM 大小重新建立雲端服務的範例指令碼。 

## <a name="next-steps"></a>後續步驟
* [對以 Resource Manager 部署模型建立的 VM 調整大小](../resize-vm.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

