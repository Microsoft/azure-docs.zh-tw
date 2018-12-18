---
title: 管理 Azure Stack 的實際記憶體容量 | Microsoft Docs
description: 監視和管理 Azure Stack 可用的儲存空間。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.assetid: 84518E90-75E1-4037-8D4E-497EAC72AAA1
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 09/28/2018
ms.author: mabrigg
ms.reviewer: Thomas.Roettinger
ms.openlocfilehash: a914d20f61b5b632e792ca29f6c201964db4a203
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47452134"
---
# <a name="manage-physical-memory-capacity-for-azure-stack"></a>管理 Azure Stack 的實際記憶體容量

「適用於：Azure Stack 整合系統」

若要提高 Azure Stack 的總可用記憶體容量，您可以新增其他記憶體。 在 Azure Stack 中，您的實體伺服器也稱為「縮放單位節點」。 所有屬於單一縮放單位成員的縮放單位節點都必須具有相同的記憶體數量。

> [!note]  
> 繼續之前，請參閱您的硬體製造商文件，確認您的製造商是否支援實體記憶體升級。 OEM 硬體廠商支援合約可能會要求廠商進行實體伺服器機架放置和裝置韌體更新。

下列流程圖顯示將記憶體新增至每個縮放單位節點的一般程序。

![將記憶體新增至每個縮放單位節點](media\azure-stack-manage-storage-physical-capacity\process-to-add-memory-to-scale-unit.png)

## <a name="add-memory-to-an-existing-node"></a>將記憶體新增至現有的節點
下列步驟提供新增記憶體程序的高階概觀。 

> [!Warning]  
請勿在未參考您 OEM 提供之文件的情況下依照這些步驟執行。

> [!Warning]  
因為不支援輪流記憶體升級，所以必須關閉整個縮放單位。

1. 使用[啟動和停止 Azure Stack](azure-stack-start-and-stop.md) 一文中記載的步驟來停止 Azure Stack。
2. 利用硬體製造商的文件來升級每部實體電腦上的記憶體。
3. 使用[啟動和停止 Azure Stack](azure-stack-start-and-stop.md) 一文中的步驟來啟用 Azure Stack。

## <a name="next-steps"></a>後續步驟

 - 若要了解如何在 Azure Stack 中管理儲存體帳戶，以便根據業務需求來尋找、復原及回收儲存體容量，請參閱[在 Azure Stack 中管理儲存體帳戶](azure-stack-manage-storage-accounts.md)。
 - 若要了解 Azure Stack 雲端操作員如何監視和管理其 Azure Stack 部署的儲存容量，請參閱[管理 Azure Stack 的儲存體容量](azure-stack-manage-storage-shares.md)。 
