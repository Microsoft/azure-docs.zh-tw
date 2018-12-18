---
title: Azure Stack 中的縮放單位節點動作 | Microsoft Docs
description: 了解如何檢視節點狀態和在 Azure Stack 整合系統上使用開啟電源、關閉電源、清空和繼續節點動作。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/14/2018
ms.author: mabrigg
ms.reviewer: ppacent
ms.openlocfilehash: 1f59f2ce6e3bf8d34ce225aa93da76ad523775e0
ms.sourcegitcommit: 4ea0cea46d8b607acd7d128e1fd4a23454aa43ee
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/15/2018
ms.locfileid: "41946457"
---
# <a name="scale-unit-node-actions-in-azure-stack"></a>Azure Stack 中的縮放單位節點動作

「適用於：Azure Stack 整合系統」

本文說明如何檢視縮放單位及其相關聯節點的狀態，以及如何使用可用的節點動作。 節點動作包括開啟電源、關閉電源、清空、繼續和修復。 一般而言，您可以於現場更換組件期間，或針對節點復原案例使用這些節點動作。

> [!Important]  
> 本文中說明的所有節點動作都應該一次以一個節點為目標。


## <a name="view-the-status-of-a-scale-unit-and-its-nodes"></a>檢視縮放單位和其節點的狀態

在管理員入口網站中，您可以輕鬆地檢視縮放單位和其相關聯節點的狀態。

若要檢視縮放單位的狀態：

1. 在 [區域管理] 圖格上，選取區域。
2. 在左側的 [基礎結構資源] 下，選取 [縮放單位]。
3. 在結果中，選取縮放單位。
 
您可以在此處檢視下列資訊：

- 區域名稱。 在 PowerShell 模組中，會使用 **-Location** 來參考區域名稱。
- 系統類型
- 邏輯核心數總計
- 記憶體總計
- 個別節點的清單和其狀態；**執行中**或**已停止**。

![顯示每個節點執行狀態的縮放單位圖格](media/azure-stack-node-actions/ScaleUnitStatus.PNG)

## <a name="view-information-about-a-scale-unit-node"></a>檢視縮放單位節點的相關資訊

如果您選取個別節點，您可以檢視下列資訊：

- 區域名稱
- 伺服器模型
- 基礎板管理控制器 (BMC) 的 IP 位址
- 作業狀態
- 核心數總計
- 記憶體量總計
 
![顯示每個節點執行狀態的縮放單位圖格](media/azure-stack-node-actions/NodeActions.PNG)

您也可以從這裡執行縮放單位節點動作。

## <a name="scale-unit-node-actions"></a>縮放單位節點動作

當您檢視縮放單位節點的相關資訊時，您也可以執行節點動作，例如：

- 清空與繼續
- 修復

節點的作業狀態會決定哪些選項可供使用。

### <a name="power-off"></a>關閉電源

[關閉電源] 動作會將節點關閉。 它與按下電源按鈕相同。 它**不會**傳送關機信號到作業系統。 針對計劃的關機作業，請確定您先清空縮放單位節點。

當節點處於擱置狀態，而且不再回應要求，通常會使用此動作。

> [!Important] 
> 此功能只可透過 PowerShell 取得。 之後將會在 Azure Stack 系統管理員入口網站中再度提供。


若要透過 PowerShell 執行關閉電源動作：

````PowerShell
  Stop-AzsScaleUnitNode -Location <RegionName> -Name <NodeName>
```` 

在關閉電源動作無法運作的罕見情況下，請改用 BMC Web 介面。

### <a name="power-on"></a>開啟電源

[開啟電源] 動作會將節點關閉。 它與按下電源按鈕相同。 

> [!Important] 
> 此功能只可透過 PowerShell 取得。 之後將會在 Azure Stack 系統管理員入口網站中再度提供。

若要透過 PowerShell 執行開啟電源動作：

````PowerShell
  Start-AzsScaleUnitNode -Location <RegionName> -Name <NodeName>
````

在開啟電源動作無法運作的罕見情況下，請改用 BMC Web 介面。

### <a name="drain"></a>清空

[清空] 動作會撤除所有作用中的工作負載，方法是將它們分散在該特定縮放單位的其餘節點之間。

這個動作通常用於現場更換組件期間，例如，更換整個節點。

> [!IMPORTANT]  
> 確定您只在已通知使用者、計劃性的維護期間清空節點。 在某些情況下，使用中的工作負載可能會導致中斷。

若要透過 PowerShell 執行清空動作：

  ````PowerShell
  Disable-AzsScaleUnitNode -Location <RegionName> -Name <NodeName>
  ````

### <a name="resume"></a>繼續

[繼續] 動作會繼續已清空的節點，並且將其標示為使用中以進行工作負載放置。 先前已在節點上執行的工作負載不會容錯回復。 (如果您清空節點然後關機，當您重新開啟節點的電源時，不會將它標示為使用中以進行工作負載放置。 準備就緒時，您必須使用繼續動作將節點標記為使用中。)

若要透過 PowerShell 執行繼續動作：

  ````PowerShell
  Enable-AzsScaleUnitNode -Location <RegionName> -Name <NodeName>
  ````

### <a name="repair"></a>修復

[修復] 動作會修復節點。 只針對下列其中一個案例使用它：

- 完整節點更換 (不論有無新資料磁碟)
- 硬體元件失敗並取代之後 (如果現場可更換單元 (FRU) 文件中有建議)。

> [!IMPORTANT]  
> 需要更換節點或個別硬體元件時，請參閱 FRU OEM 硬體廠商的文件，以了解確切步驟。 FRU 文件會指定在更換硬體元件之後是否需要執行修復動作。  

執行修復動作時，您需要指定 BMC IP 位址。 

若要透過 PowerShell 執行修復動作：

  ````PowerShell
  Repair-AzsScaleUnitNode -Location <RegionName> -Name <NodeName> -BMCIPAddress <BMCIPAddress>
  ````

## <a name="next-steps"></a>後續步驟

若要深入了解 Azure Stack 網狀架構系統管理員模組，請參閱 [Azs.Fabric.Admin](https://docs.microsoft.com/powershell/module/azs.fabric.admin/?view=azurestackps-1.4.0)。