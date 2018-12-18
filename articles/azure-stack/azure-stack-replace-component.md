---
title: 取代 Azure Stack 縮放單位節點上的硬體元件 | Microsoft Docs
description: 了解如何取代 Azure Stack 整合式系統上的硬體元件。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.assetid: c6e036bf-8c80-48b5-b2d2-aa7390c1b7c9
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 1/29/2018
ms.author: mabrigg
ms.openlocfilehash: 7018f0122ab1ef11d64cce8a9adf58419d0e9ba7
ms.sourcegitcommit: 9d317dabf4a5cca13308c50a10349af0e72e1b7e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/01/2018
ms.locfileid: "28919707"
---
# <a name="replace-a-hardware-component-on-an-azure-stack-scale-unit-node"></a>取代 Azure Stack 縮放單位節點上的硬體元件

「適用於：Azure Stack 整合系統」

本文說明更換非熱插拔硬體元件的一般程序。 實際取代步驟會因原始設備製造商 (OEM) 硬體廠商而異。 如需您 Azure Stack 整合式系統專屬的詳細步驟，請參閱廠商的現場可更換單元 (FRU) 文件。

非熱插拔元件包含下列各項：

- CPU*
- 記憶體*
- 主機板/基礎板管理控制器 (BMC)/視訊卡
- 磁碟控制卡/主機匯流排介面卡 (HBA)/背板
- 網路介面卡 (NIC)
- 作業系統磁碟*
- 資料磁碟機 (不支援熱插拔的磁碟機，例如 PCI-e 附加介面卡)*

*這些元件可能支援熱插拔，但可能會隨著廠商實作方式而不同。 如需詳細步驟，請參閱 OEM 廠商的 FRU 文件。

下列流程圖顯示更換非熱插拔硬體元件的一般 FRU 程序。

![顯示元件更換流程的流程圖](media/azure-stack-replace-component/replacecomponentflow.PNG)

*根據硬體的實體條件，可能不需要此動作。

\*\*不論 OEM 硬體廠商是否會執行元件更換和更新，韌體都可能根據支援合約而有所不同。

## <a name="review-alert-information"></a>檢閱警示資訊

Azure Stack 健康狀態和監視系統會追蹤儲存空間直接存取所控制之網路介面卡和資料磁碟機的健康情況。 它不會追蹤其他硬體元件。 針對所有其他硬體元件，在硬體生命週期主機上執行之廠商特定硬體監視解決方案中引發警示。  

## <a name="component-replacement-process"></a>元件更換程序

下列步驟提供元件取代流程的高階概觀。 請勿在未參考您 OEM 提供之 FRU 文件的情況下遵循這些步驟執行。

1. 使用[清空](azure-stack-node-actions.md#scale-unit-node-actions)動作，讓縮放單位節點進入維護模式。 根據硬體的實體條件，可能不需要此動作。

   > [!NOTE]
   > 在任何情況下，只能同時清空一個節點並關機，而不中斷 S2D (儲存空間直接存取)。

2. 縮放單位節點處於維護模式之後，請使用[關閉電源](azure-stack-node-actions.md#scale-unit-node-actions)動作。 根據硬體的實體條件，可能不需要此動作。

   > [!NOTE]
   > 在關閉電源動作無法運作的罕見情況下，請改用基礎板管理控制器 (BMC) Web 介面。

3. 更換損毀的硬體元件。 不論 OEM 硬體廠商是否執行元件更換，都可能會根據支援合約而不同。  
4. 更新韌體。 請遵循使用硬體生命週期主機的廠商特定韌體更新程序，確定已取代的硬體元件已套用核准的韌體層級。 不論 OEM 硬體廠商是否執行這個步驟，都可能會根據支援合約而不同。  
5. 使用[修復](azure-stack-node-actions.md#scale-unit-node-actions)動作，將縮放單位節點回復到縮放單位。
6. 使用具有特殊權限的端點來[檢查虛擬磁碟修復狀態](azure-stack-replace-disk.md#check-the-status-of-virtual-disk-repair)。 利用新的資料磁碟機，根據系統負載與已耗用的空間而定，完整的儲存體修復作業可能需要數小時的時間。
7. 當修復動作完成之後，驗證已自動關閉所有作用中警示。

## <a name="next-steps"></a>後續步驟

- 如需更換熱插拔實體磁碟的資訊，請參閱[更換磁碟](azure-stack-replace-disk.md)。
- 如需取代實體節點的資訊，請參閱[取代縮放單位節點](azure-stack-replace-node.md)。
