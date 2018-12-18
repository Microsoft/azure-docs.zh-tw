---
title: 了解 Azure NetApp Files 的儲存體階層 | Microsoft Docs
description: 描述儲存體階層，包括 Azure NetApp Files 帳戶、容量集區和磁碟區。
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 03/28/2018
ms.author: b-juche
ms.openlocfilehash: 6f5ed4e7ede9a098d69b7a40f44dd60f9b400472
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39010985"
---
# <a name="understand-the-storage-hierarchy-of-azure-netapp-files"></a>了解 Azure NetApp Files 的儲存體階層

在 Azure NetApp Files 中建立磁碟區之前，您必須先購買及設定佈建容量的集區。  若要設定容量集區，您必須具有 NetApp 帳戶。 了解儲存體階層有助於設定和管理您的 Azure NetApp Files 資源。

## <a name="azure_netapp_files_account"></a>NetApp 帳戶

- NetApp 帳戶可充當組成容量集區的系統管理群組。  
- NetApp 帳戶與一般 Azure 儲存體帳戶不同。 
- NetApp 帳戶的範圍具有區域性。   
- 您在一個區域中可以有多個 NetApp 帳戶，但是每個 NetApp 帳戶都只繫結至單一區域。

## <a name="capacity_pools"></a>容量集區

- 容量集區是依據其佈建容量進行測量。  
- 容量是以您購買的固定 SKU (例如 4-TB 容量) 進行佈建。
- 容量集區只能有一個服務等級。  
  目前只有進階服務等級可用。
- 每個容量集區都只屬於一個 NetApp 帳戶。  
- 容量集區不能跨越 NetApp 帳戶移動。   
  例如，在下方[儲存體階層的概念圖表](#conceptual_diagram_of_storage_hierarchy)中，容量集區 1 無法從美國東部 NetApp 帳戶移至美國西部 2 NetApp 帳戶。  

## <a name="volumes"></a>磁碟區

- 磁碟區是依據邏輯容量耗用量進行測量，可以調整到每個磁碟區最多 100 TB。
- 磁碟區的容量耗用量是根據其集區的佈建容量進行計算。
- 每個磁碟區都只屬於一個集區，但是一個集區可以包含多個磁碟區。 
- 您可以在相同的 NetApp 帳戶內，於集區之間移動磁碟區。    
  例如，在下方[儲存體階層的概念圖表](#conceptual_diagram_of_storage_hierarchy)中，您可以將磁碟區從容量集區 1 移至容量集區 2。

## <a name="conceptual_diagram_of_storage_hierarchy"></a>儲存體階層的概念圖表 
下列範例顯示 Azure 訂用帳戶、NetApp 帳戶、容量集區和磁碟區之間的關係。   

![儲存體階層的概念圖表](../media/azure-netapp-files/azure-netapp-files-storage-hierarchy.png)

## <a name="next-steps"></a>後續步驟

1. [建立 NetApp 帳戶](azure-netapp-files-create-netapp-account.md)
2. [設定容量集區](azure-netapp-files-set-up-capacity-pool.md)
3. [建立適用於 Azure NetApp Files 的磁碟區](azure-netapp-files-create-volumes.md)
4. [設定磁碟區的匯出原則 (選用)](azure-netapp-files-configure-export-policy.md)