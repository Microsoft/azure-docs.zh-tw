---
title: 管理警示執行個體
description: 管理整個 Azure 中的警示執行個體
author: anantr
services: monitoring
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: anantr
ms.component: alerts
ms.openlocfilehash: 614e65ca791559f4649963ee82940c3f96beddb3
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46948446"
---
# <a name="manage-alert-instances"></a>管理警示執行個體
透過 Azure 監視器的[整合警示體驗](https://aka.ms/azure-alerts-overview)，您現在可以在單一窗格，看到整個 Azure 跨越多個訂用帳戶中所有不同類型的警示。 此文章逐步引導您檢視警示執行個體，以及深入使用入口網站來找到特定警示執行個體進行疑難排解。

1. 有三種方式可進入 [警示] 頁面

   + 在[入口網站](https://portal.azure.com/)中選取 [監視]，然後在 [監視] 區段下選擇 [警示]。  
    ![監視](./media/monitoring-alerts-managing-alerts-instances/monitoring-alerts-managing-alert-instances-toc.jpg)
  
   + 您可以從特定**資源**的內容中瀏覽 [警示]。 開啟資源之後，透過其目錄瀏覽到 [監視] 區段，然後在為該特定資源預先篩選的登陸頁面，選擇 [警示]。
   
    ![監視](./media/monitoring-alerts-managing-alerts-instances/alert-resource.JPG)
    
   + 您可以從特定**資源群組**的內容中瀏覽 [警示]。 開啟資源群組之後，透過其目錄瀏覽到 [監視] 區段，然後在為該特定資源群組預先篩選的登陸頁面，選擇 [警示]。    
   
    ![監視](./media/monitoring-alerts-managing-alerts-instances/alert-rg.JPG)

1.  您會進入 [警示摘要] 頁面，該頁面提供整個 Azure 中所有警示執行個體的概觀。 您可以選取**多個訂用帳戶** (以 5 個為限) 或跨**資源群組**、特定**資源**或**時間範圍**篩選，來修改摘要檢視。 按一下 [警示總計] 或任何嚴重性級區，移至警示的清單檢視。     
    ![警示摘要](./media/monitoring-alerts-managing-alerts-instances/alerts-summary.jpg)
 
1.  您會進入 [所有警示] 頁面，其中會列出整個 Azure 中的所有警示執行個體。如果您是從警示通知來到入口網站，則可以使用提供的篩選條件，將範圍縮小至該特定警示執行個體。 (**注意**：如果您按一下任何嚴重性級區才來到此頁面時，當您進入時，已針對該嚴重性預先篩選該清單)。 除了先前頁面上提供的篩選條件，您現在也可以根據監視服務 (例如，計量的平台)、監視器條件 (已引發或已解決)、嚴重性、警示狀態 (新增/已通知/已關閉) 或智慧群組識別碼來篩選。

    ![所有警示](./media/monitoring-alerts-managing-alerts-instances/all-alerts.jpg)

    > [!NOTE]
    >  如果您按一下任何嚴重性級區才來到此頁面時，當您進入此頁面時，已針對該嚴重性預先篩選該清單。
 
1.  按一下任何警示執行個體，隨即會開啟 [警示詳細資料] 頁面，讓您將深入了解到該特定警示執行個體的相關資訊。   
![警示詳細資料](./media/monitoring-alerts-managing-alerts-instances/alert-details.jpg)  
