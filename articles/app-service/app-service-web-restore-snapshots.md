---
title: 在 Azure 中還原應用程式
description: 了解如何從快照集還原應用程式。
services: app-service
documentationcenter: ''
author: ahmedelnably
manager: cfowler
editor: ''
ms.assetid: 4164f9b5-f735-41c6-a2bb-71f15cdda417
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: multiple
ms.topic: article
ms.date: 04/04/2018
ms.author: aelnably;nicking
ms.openlocfilehash: e1ae8fcc30323c865aa96937f43054515f293394
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33766660"
---
# <a name="restore-an-app-in-azure-from-a-snapshot"></a>在 Azure 中透過快照集還原應用程式
本文說明如何在 [Azure App Service](../app-service/app-service-web-overview.md) 中透過快照集還原應用程式。 您可以根據您的其中一個應用程式快照集，將應用程式還原為先前的狀態。 您不需要啟用快照集備份，平台會自動儲存所有應用程式的快照集以供進行資料復原。

快照集是累加的陰影複製，可提供數個比一般[備份](web-sites-backup.md)還要好的優點：
- 不會因為檔案鎖定而產生任何檔案複製錯誤。
- 沒有儲存體大小限制。
- 不需要設定。

從快照集還原是提供給在**進階**層或更高層級中執行的應用程式使用。 如需有關相應增加應用程式的詳細資訊，請參閱 [在 Azure 中調整應用程式規模](web-sites-scale.md)。

## <a name="limitations"></a>限制

- 此功能目前為預覽狀態。
- 您只能還原至相同的應用程式或屬於該應用程式的位置。
- App Service 會在還原期間停止目標應用程式或目標位置。
- App Service 會保留三個月的快照集供平台復原資料之用。
- 您只能還原最近 30 天的快照集。
 

## <a name="restore-an-app-from-a-snapshot"></a>從快照集還原應用程式

1. 在應用程式位於 [Azure 入口網站](https://portal.azure.com)的 [設定] 頁面上，按一下 [備份] 以顯示 [備份] 頁面。 然後按一下 [快照集] \(預覽\) 區段底下的 [還原]。
   
    ![](./media/app-service-web-restore-snapshots/1.png)

2. 在 [還原] 頁面中，選取要還原的快照集。
   
    ![](./media/app-service-web-restore-snapshots/2.png)
   
3. 在 [還原目的地] 中指定應用程式還原目的地。
   
    ![](./media/app-service-web-restore-snapshots/3.png)
   
   > [!WARNING]
   > 如果您選擇 [覆寫]，則系統會將應用程式目前檔案系統中的所有現有資料予以清除和覆寫。 按一下 [確定] 之前，請確定這是您想要執行的動作。
   > 
   > 
      
   > [!Note]
   > 由於目前的技術限制，您只能還原到相同縮放單位中的應用程式。 未來的版本將會移除這項限制。
   > 
   > 
   
    您可以選取將 [現有應用程式] 還原至位置。 使用此選項之前，您應該要已在應用程式中建立位置。

4. 您可以選擇還原站台設定。
   
    ![](./media/app-service-web-restore-snapshots/4.png)

5. 按一下 [確定]。
