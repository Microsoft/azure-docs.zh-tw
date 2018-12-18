---
title: 更換 Azure Stack 整合系統上的縮放單位節點 | Microsoft Docs
description: 了解如何更換 Azure Stack 整合系統上的實體縮放單位節點。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.assetid: f9434689-ee66-493c-a237-5c81e528e5de
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/10/2018
ms.author: mabrigg
ms.openlocfilehash: 1b37b150dad4951a4ade81f226b515ce9cae9053
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44377049"
---
# <a name="replace-a-scale-unit-node-on-an-azure-stack-integrated-system"></a>更換 Azure Stack 整合系統上的縮放單位節點

「適用於：Azure Stack 整合系統」

本文說明更換 Azure Stack 整合系統上的實體電腦 (也稱為「縮放單位節點」) 的一般程序。 實際的縮放單位節點更換步驟將因原始設備製造商 (OEM) 硬體廠商而異。 如需您系統專屬的詳細步驟，請參閱廠商的現場可更換單元 (FRU) 文件。

下列流程圖顯示更換整個縮放單位節點的一般 FRU 程序。

![更換節點程序的流程圖](media/azure-stack-replace-node/replacenodeflow.png)

*根據硬體的實體條件，可能不需要此動作。

## <a name="review-alert-information"></a>檢閱警示資訊

如果縮放單位節點已關閉，您就會收到下列重大警示：

- 節點並未連接到網路控制站
- 無法存取節點以供虛擬機器放置
- 縮放單位節點已離線

![縮放單位已關閉的警示清單](media/azure-stack-replace-node/nodedownalerts.png)

如果您開啟「縮放單位節點已離線」警示，警示描述會包含無法存取的縮放單位節點。 您也可能會在於硬體生命週期主機上執行之 OEM 特定的監視解決方案中收到其他警示。

![節點離線警示的詳細資料](media/azure-stack-replace-node/nodeoffline.png)

## <a name="scale-unit-node-replacement-process"></a>縮放單位節點更換程序

提供下列步驟以作為縮放單位節點更換程序的高階概觀。 如需您系統專屬的詳細步驟，請參閱 OEM 硬體廠商的 FRU 文件。 請勿在未參考您 OEM 提供之文件的情況下依照這些步驟執行。

1. 使用[清空](azure-stack-node-actions.md#scale-unit-node-actions)動作來使縮放單位節點進入維護模式。 根據硬體的實體條件，可能不需要此動作。

   > [!NOTE]
   > 在任何情況下，只能同時清空一個節點並關機，而不中斷 S2D (儲存空間直接存取)。

2. 如果節點的電源仍然開啟，請使用[關閉電源](azure-stack-node-actions.md#scale-unit-node-actions)動作。 根據硬體的實體條件，可能不需要此動作。
 
   > [!NOTE]
   > 在關閉電源動作無法運作的罕見情況下，請改用基礎板管理控制器 (BMC) Web 介面。

1. 更換實體電腦。 一般而言，這由您的 OEM 硬體廠商來完成。
2. 使用[修復](azure-stack-node-actions.md#scale-unit-node-actions)動作，將實體電腦新增至縮放單位。
3. 使用具有特殊權限的端點來[檢查虛擬磁碟修復狀態](azure-stack-replace-disk.md#check-the-status-of-virtual-disk-repair)。 利用新的資料磁碟機，根據系統負載與已耗用的空間而定，完整的儲存體修復作業可能需要數小時的時間。
4. 當修復動作完成之後，驗證已自動關閉所有作用中警示。

## <a name="next-steps"></a>後續步驟

- 如需更換熱插拔實體磁碟的資訊，請參閱[更換磁碟](azure-stack-replace-disk.md)。 
- 如需更換非熱插拔的硬體元件相關資訊，請參閱[更換硬體元件](azure-stack-replace-component.md)。
