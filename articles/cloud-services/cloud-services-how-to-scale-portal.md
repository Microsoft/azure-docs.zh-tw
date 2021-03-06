---
title: 在入口網站中自動調整雲端服務 (傳統) |Microsoft Docs
description: 了解如何使用入口網站在 Azure 中設定雲端服務 web 角色或背景工作角色的自動調整規則。
ms.topic: article
ms.service: cloud-services
ms.date: 10/14/2020
ms.author: tagore
author: tanmaygore
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: a3e7f72dbe16c51280b922da2b5fc6550dee1d34
ms.sourcegitcommit: 6272bc01d8bdb833d43c56375bab1841a9c380a5
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2021
ms.locfileid: "98743350"
---
# <a name="how-to-configure-auto-scaling-for-a-cloud-service-classic-in-the-portal"></a>如何在入口網站中設定雲端服務 (傳統) 的自動調整

> [!IMPORTANT]
> [Azure 雲端服務 (延伸支援) ](../cloud-services-extended-support/overview.md) 是 Azure 雲端服務產品的新 Azure Resource Manager 型部署模型。透過這種變更，在以 Azure Service Manager 為基礎的部署模型上執行的 Azure 雲端服務，已重新命名為雲端服務 (傳統) ，而且所有新的部署都應該使用 [雲端服務 (延伸支援) ](../cloud-services-extended-support/overview.md)。

針對雲端服務背景工作角色設定條件，以觸發相應縮小或相應放大作業。 適用於角色的條件可以 CPU、磁碟或角色的網路負載為根據。 您也可以根據訊息佇列或一些與您訂用帳戶相關聯的其他 Azure 資源的計量來設定條件。

> [!NOTE]
> 本文著重於雲端服務 web 和背景工作角色。 當您直接建立虛擬機器 (傳統) 時，它會裝載於雲端服務中。 如果要調整標準虛擬機器，您可以將它與[可用性設定組](/previous-versions/azure/virtual-machines/windows/classic/configure-availability-classic)產生關聯，還可以手動開啟或關閉它們。

## <a name="considerations"></a>考量
在設定應用程式的調整之前，您應該先考量下列資訊：

* 調整受到核心使用量影響。

    較大型的角色執行個體會使用較多核心。 您只能在訂用帳戶的核心限制內調整應用程式。 例如，假設您的訂用帳戶有 20 個核心的限制。 如果您使用兩個中型大小的雲端服務來執行應用程式 (總計 4 個核心)，則您最多只能在訂用帳戶中相應增加所剩餘的 16 個核心的其他雲端服務部署。 如需關於大小的詳細資訊，請參閱[雲端服務的大小](cloud-services-sizes-specs.md)。

* 您可以依據佇列訊息臨界值來調整。 如需如何使用佇列的詳細資訊，請參閱 [如何使用佇列儲存體服務](../storage/queues/storage-dotnet-how-to-use-queues.md)。

* 您也可以調整與您訂用帳戶相關聯的其他資源。

* 若要對應用程式啟用高可用性，您應該確定應用程式是以兩個以上的角色執行個體來部署。 如需詳細資訊，請參閱 [服務等級協定](https://azure.microsoft.com/support/legal/sla/)。

* 所有角色都處於 **就緒** 狀態時，才會出現自動調整。  


## <a name="where-scale-is-located"></a>調整所在之處
當您選取雲端服務之後，應該會看見雲端服務刀鋒視窗。

1. 在 [雲端服務] 刀鋒視窗的 [角色和執行個體] 圖格上，選取雲端服務的名稱。   
   **重要**︰請務必按一下雲端服務角色，而不是角色底下的角色執行個體。

    ![[角色和實例] 圖格的螢幕擷取畫面，其中包含背景工作角色和 [S B 佇列 1] 選項（以紅色標示）。](./media/cloud-services-how-to-scale-portal/roles-instances.png)
2. 選取 [調整]  磚。

    ![[作業] 頁面的螢幕擷取畫面，其中銷售磚以紅色標示。](./media/cloud-services-how-to-scale-portal/scale-tile.png)

## <a name="automatic-scale"></a>自動調整
您可以使用下列任一種模式來設定角色的調整規模設定：**手動** 或 **自動**。 「手動」是您預期的模式，您會設定執行個體的絕對計數。 但是，「自動」讓您能夠設定規則來控制您應該調整的方式和程度。

將 [調整規模依據] 選項設定為 [排程及效能規則]。

![使用設定檔和規則的映射雲端服務調整規模設定](./media/cloud-services-how-to-scale-portal/schedule-basics.png)

1. 現有的設定檔。
2. 新增父設定檔的規則。
3. 新增另一個設定檔。

選取 [新增設定檔]。 設定檔決定您要用於調整規模的模式︰**一律**、**週期性**、**固定日期**。

當您設定設定檔與規則之後，請選取頂端的 [儲存]  圖示。

#### <a name="profile"></a>設定檔
設定檔會設定調整規模的最小和最大執行個體個數，而且也會在此調整範圍作用中時。

* **Always**

    一律讓此範圍的執行個體個數保持可用狀態。  

    ![一律調整的雲端服務](./media/cloud-services-how-to-scale-portal/select-always.png)
* **週期性**

    選擇一組要調整的一週天數。

    ![具有週期性排程的雲端服務調整規模](./media/cloud-services-how-to-scale-portal/select-recurrence.png)
* **固定日期**

    要調整角色的固定日期範圍。

    ![具有固定日期的雲端服務調整規模](./media/cloud-services-how-to-scale-portal/select-fixed.png)

當您設定設定檔之後，請選取設定檔刀鋒視窗底部的 [確定]  按鈕。

#### <a name="rule"></a>規則
規則會新增至設定檔，並顯示將觸發調整規模的條件。

規則觸發程序是以雲端服務的計量 (CPU 使用量、磁碟活動或網路活動) 為依據，您可以在其中新增條件值。 此外，您可以根據訊息佇列或一些與您訂用帳戶相關聯的其他 Azure 資源的計量來設定觸發程序。

![[規則] 對話方塊的螢幕擷取畫面，其中的 [計量名稱] 選項以紅色框住。](./media/cloud-services-how-to-scale-portal/rule-settings.png)

當您設定規則之後，請選取規則刀鋒視窗底部的 [確定]  按鈕。

## <a name="back-to-manual-scale"></a>回到手動調整
瀏覽至[調整規模設定](#where-scale-is-located)，並將 [調整規模依據] 選項設定為 [手動輸入的執行個體計數]。

![具有設定檔與規則的雲端服務調整規模設定](./media/cloud-services-how-to-scale-portal/manual-basics.png)

這個設定會移除該角色的自動調整，然後您可以直接設定執行個體計數。

1. 調整 (手動或自動) 選項。
2. 角色執行個體滑桿，可用來設定要調整的執行個體。
3. 要調整之角色的執行個體。

當您設定調整規模設定之後，請選取頂端的 [儲存]  圖示。