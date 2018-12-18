---
title: 在 Azure 入口網站自動調整虛擬機器擴展集 | Microsoft Docs
description: 如何在 Azure 入口網站中建立虛擬機器擴展集的自動調整規則
services: virtual-machine-scale-sets
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 88886cad-a2f0-46bc-8b58-32ac2189fc93
ms.service: virtual-machine-scale-sets
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/29/2018
ms.author: cynthn
ms.openlocfilehash: 4ae9d8053f7463c069a7305bbd07a91a387e31d2
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46972295"
---
# <a name="automatically-scale-a-virtual-machine-scale-set-in-the-azure-portal"></a>如何在 Azure 入口網站中自動調整虛擬機器擴展集
當您建立擴展集時，您會定義您想要執行的 VM 執行個體數目。 當您的應用程式需求變更時，您可以自動增加或減少 VM 執行個體數目。 自動調整的能力可讓您在整個應用程式的生命週期中，跟上客戶的需求或對應用程式效能變更做出回應。

本文示範如何在 Azure 入口網站中建立自動調整規則，用以監視擴展集內的 VM 執行個體效能。 這些自動調整規則可增加或減少 VM 執行個體的數目，以回應這些效能計量。 您也可以透過 [Azure PowerShell](tutorial-autoscale-powershell.md) 或 [Azure CLI](tutorial-autoscale-cli.md) 來完成這些步驟。


## <a name="prerequisites"></a>必要條件
若要建立自動調整規則，您會需要現有的虛擬機器擴展集。 您可以使用 [Azure 入口網站](quick-create-portal.md)、[Azure PowerShell](quick-create-powershell.md) 或 [Azure CLI](quick-create-cli.md) 來建立擴展集。


## <a name="create-a-rule-to-automatically-scale-out"></a>建立自動相應放大的規則
如果您的應用程式需求增加，您擴展集內 VM 執行個體上的負載也會跟著增加。 如果這樣的負載增加會持續而非只是短暫的需求，您就可以設定自動調整規則來增加擴展集中的 VM 執行個體數目。 建立這些 VM 執行個體並部署應用程式後，擴展集就會開始透過負載平衡器將流量分散給它們。 您可以控制要監視哪些計量 (例如 CPU 或磁碟)、應用程式負載必須符合給定的閾值多久，以及要將多少個 VM 執行個體新增至擴展集。

1. 請開啟 Azure 入口網站並從儀表板左側的功能表選取 [資源群組]。
2. 請選取包含您擴展集的資源群組，然後從資源清單中選擇您的擴展集。
3. 請從擴展集視窗左側的功能表中選擇 [調整]。 請選取按鈕以 [啟用自動調整]：

    ![在 Azure 入口網站中啟用自動調整](media/virtual-machine-scale-sets-autoscale-portal/enable-autoscale.png)

4. 請輸入您設定的名稱，例如 [自動調整]，然後選取 [新增規則]。

5. 讓我們建立規則，以便平均 CPU 負載在 10 分鐘期間內大於 70% 時，增加擴展集內 VM 執行個體的數目。 觸發此規則時，VM 執行個體數目會增加 20%。 若擴展集內 VM 執行個體的數目很少，您可以將 [作業] 設定為 [增加計數]，然後針對 [執行個體計數] 指定 *1* 或 *2*。 若擴展集內 VM 執行個體的數目很多，則增加 VM 執行個體數目 10% 或 20% 可能更加適當。

    請針對您的規則指定下列設定：
    
    | 參數              | 說明                                                                                                         | 值          |
    |------------------------|---------------------------------------------------------------------------------------------------------------------|----------------|
    | *時間彙總*     | 定義應該如何彙總收集的計量以進行分析。                                                | 平均值        |
    | *計量名稱*          | 要監視並套用擴展集動作的效能計量。                                                   | Percentage CPU |
    | *時間粒紋統計資料* | 定義在每個時間粒紋中所收集的計量，應如何彙總以便進行分析。                             | 平均值        |
    | *運算子*             | 用以比較計量資料與閾值之間差異的運算子。                                                     | 大於   |
    | *閾值*            | 讓自動調整規則觸發動作的百分比。                                                 | 70             |
    | *Duration*             | 在比較計量與閾值之前監視的時間長短。                                   | 10 分鐘     |
    | *作業*            | 定義擴展集在套用規則時該相應增加或相應減少，以及該增加或減少多少                        | 增加多少百分比 |
    | *執行個體計數*       | 觸發此規則時，應該變更多少 VM 執行個體百分比。                                            | 20             |
    | *冷卻時間 (分鐘)*  | 再次套用規則前需要等待多久時間，以便讓自動調整動作生效。 | 5 分鐘      |

    下列範例會示範在 Azure 入口網站所建立，且符合這些設定的規則：

    ![建立自動調整規則以增加 VM 執行個體的數目](media/virtual-machine-scale-sets-autoscale-portal/rule-increase.png)

6. 若要建立規則，請選取 [新增]


## <a name="create-a-rule-to-automatically-scale-in"></a>建立規則以便自動相應縮小
在夜晚或週末，您的應用程式需求可能會減少。 若這樣的負載減少會持續一段時間，您就可以設定自動調整規則來減少擴展集中的 VM 執行個體數目。 這個相應縮小的動作可減少執行擴展集的成本，因為只會執行符合目前需求所需的執行個體數目。

1. 再次選擇 [新增規則]。
2. 建立規則，以便平均 CPU 負載在 10 分鐘期間內低於 30% 時，減少擴展集內 VM 執行個體的數目。 觸發此規則時，VM 執行個體的數目會減少 20%。

    請比照上一個規則所使用的方法辦理。 請針對您的規則調整下列設定：
    
    | 參數              | 說明                                                                                                          | 值          |
    |------------------------|----------------------------------------------------------------------------------------------------------------------|----------------|
    | *運算子*             | 用以比較計量資料與閾值之間差異的運算子。                                                      | 小於   |
    | *閾值*            | 讓自動調整規則觸發動作的百分比。                                                 | 30             |
    | *作業*            | 定義擴展集在套用規則時該相應增加或相應減少，以及該增加或減少多少                         | 減少多少百分比 |
    | *執行個體計數*       | 觸發此規則時，應該變更多少 VM 執行個體百分比。                                             | 20             |

3. 若要建立規則，請選取 [新增]


## <a name="define-autoscale-instance-limits"></a>定義自動調整執行個體的限制
您的自動調整設定檔必須定義 VM 執行個體的最小、最大和預設數目。 套用自動調整規則時，這些執行個體限制可確保您相應放大時不會超過執行個體的數目上限，或相應縮小時不會少於執行個體的數目下限。

1. 請設定下列執行個體限制：

    | 最小值 | 最大值 | 預設值|
    |---------|---------|--------|
    | 2       | 10      | 2      |

2. 若要套用自動調整規則和執行個體限制，請選取 [儲存]。


## <a name="monitor-number-of-instances-in-a-scale-set"></a>監視擴展集內執行個體的數目
若要查看 VM 執行個體的數目和狀態，請從擴展集視窗左側的功能表中選取 [執行個體]。 狀態會指出是否因擴展集自動相應放大而 [正在建立] VM 執行個體，或因擴展集自動相應縮小而 [正在刪除] VM 執行個體。

![檢視擴展集 VM 執行個體的清單](media/virtual-machine-scale-sets-autoscale-portal/view-instances.png)


## <a name="autoscale-based-on-a-schedule"></a>根據排程自動調整
在先前的範例中，是根據基本主機計量 (如 CPU 使用量) 而自動相應放大或相應縮小。 您也可以根據排程來建立自動調整規則。 這些以排程為基礎的規則可讓您在應用程式需求預期增加之前 (例如核心工作時間)，自動相應增加 VM 執行個體的數目；並在預期需求減少時 (例如週末)，自動相應減少執行個體的數目。

1. 請從擴展集視窗左側的功能表中選擇 [調整]。 若要刪除在先前範例中所建立的現有自動調整規則，請選擇垃圾筒圖示。

    ![刪除現有的自動調整規則](media/virtual-machine-scale-sets-autoscale-portal/delete-rules.png)

2. 請選擇 [新增縮放條件]。 請選取規則名稱旁的鉛筆圖示並提供名稱，如 [在每個工作日相應放大]。

    ![重新命名預設的自動調整規則](media/virtual-machine-scale-sets-autoscale-portal/rename-rule.png)

3. 請選取選項按鈕，以便 [縮放到特定執行個體計數]。
4. 若要相應增加執行個體數目，請輸入 *10* 作為執行個體計數。
5. 請針對 [排程] 類型選擇 [重複特定天數]。
6. 請選取所有工作日 (星期一到星期五)。
7. 請選擇適當的時區，然後指定 [開始時間] 為 *09:00*。
8. 請再次選擇 [新增縮放條件]。 請重複此程序以建立名為「*在晚上進行縮放*」的排程 (會調整為 *3* 個執行個體，每個工作天重複，在 *18:00* 開始)。
9. 若要套用以排程為基礎的自動調整規則，請選取 [儲存]。

    ![建立依排程調整的自動調整規則](media/virtual-machine-scale-sets-autoscale-portal/schedule-autoscale.PNG)

若要查看您的自動調整規則會如何套用，請選取橫跨 [調整] 視窗頂端的 [執行歷程記錄]。 圖形和事件清單會顯示觸發自動調整規則的時間，以及您的擴展集內 VM 執行個體數目是增加或減少。


## <a name="next-steps"></a>後續步驟
在本文中，您已學到如何使用自動調整規則進行水平縮放，以及增加或減少擴展集內 VM 執行個體的「數目」。 您也可以進行垂直縮放，增加或減少 VM 執行個體的「大小」。 如需詳細資訊，請參閱[使用虛擬機器擴展集垂直自動調整](virtual-machine-scale-sets-vertical-scale-reprovision.md)。

如需有關如何管理 VM 執行個體的資訊，請參閱[使用 Azure PowerShell 管理虛擬機器擴展集](virtual-machine-scale-sets-windows-manage.md)。

若要了解如何在自動調整規則觸發時產生警示，請參閱[使用自動調整動作在 Azure 監視器中傳送電子郵件和 Webhook 警示通知](../monitoring-and-diagnostics/insights-autoscale-to-webhook-email.md)。 您也可以[使用稽核記錄，在 Azure 監視器中傳送電子郵件和 Webhook 警示通知](../monitoring-and-diagnostics/insights-auditlog-to-webhook-email.md)。
