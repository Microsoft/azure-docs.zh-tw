---
title: Azure 資源健康狀態概觀 | Microsoft Docs
description: Azure 資源健康狀態的概觀
services: Resource health
documentationcenter: ''
author: shawntabrizi
manager: ''
editor: ''
ms.assetid: 85cc88a4-80fd-4b9b-a30a-34ff3782855f
ms.service: service-health
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: Supportability
ms.date: 03/27/2018
ms.author: shawn.tabrizi
ms.openlocfilehash: 99e996f182aac774f2e2565d87fd0debaba1b2d1
ms.sourcegitcommit: c3d53d8901622f93efcd13a31863161019325216
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/29/2018
ms.locfileid: "30263117"
---
# <a name="azure-resource-health-overview"></a>Azure 資源健康狀態概觀
 
「Azure 資源健康狀態」可協助您在 Azure 問題影響您的資源時，進行診斷並取得支援。 它會通知您資源的目前及過去的健康狀態。 並提供可協助您減輕問題的技術支援。

[Azure 狀態](https://status.azure.com)會通知您影響大規模客戶的服務問題，而資源健康狀態會提供資源健康狀態的個人化儀表板。 資源健康狀態會顯示您的資源過去因 Azure 服務問題而無法使用的所有時間。 這可讓您更輕鬆了解是否違反 SLA。 

## <a name="resource-definition-and-health-assessment"></a>資源定義和健康狀態評估
資源是 Azure 服務的特定執行個體，例如：虛擬機器、Web 應用程式或 SQL 資料庫。

資源健康狀態依賴不同的 Azure 服務所發出的訊號，來評估資源健康與否。 如果資源健康狀態不佳，資源健康狀態會分析其他資訊來判斷問題的來源。 它也會識別 Microsoft 為修正這個問題所採取的動作，或您可以採取哪些動作來解決問題的原因。 

如需健康狀態評估方式的其他詳細資訊，請檢閱 [Azure 資源健康狀態](resource-health-checks-resource-types.md)中的資源類型及健康狀態檢查完整清單。

## <a name="health-status"></a>健康狀態
資源的健康會顯示為下列其中一種狀態。

### <a name="available"></a>可用
[可用] 狀態表示服務未偵測到任何會影響資源健康狀態的事件。 如果資源在過去 24 小時內從未規劃的停機時間復原，您會看到 [最近已解決] 通知。

![處於 [可用] 狀態並出現 [最近已解決] 通知的虛擬機器](./media/resource-health-overview/Available.png)

### <a name="unavailable"></a>無法使用
[無法使用] 狀態表示服務偵測到會影響資源健康狀態的進行中平台或非平台事件。

#### <a name="platform-events"></a>平台事件
平台事件是由 Azure 基礎結構的多個元件觸發。 其中包括已排程的動作 (例如計劃性維護) 和非預期的事件 (例如非計劃性主機重新開機)。

資源健康狀態提供有關事件和復原程序的其他詳細資料。 即使您沒有作用中的 Microsoft 支援合約，它也可讓您連絡支援。

![由於平台事件而處於 [無法使用] 狀態的虛擬機器](./media/resource-health-overview/Unavailable.png)

#### <a name="non-platform-events"></a>非平台事件
非平台事件是由使用者的動作所觸發。 例如，停止虛擬機器或達到 Redis 快取的連線數目上限。

![由於非平台事件而處於 [無法使用] 狀態的虛擬機器](./media/resource-health-overview/Unavailable_NonPlatform.png)

### <a name="unknown"></a>不明
[不明] 健康狀態表示資源健康狀態超過 10 分鐘未收到此資源的相關資訊。 雖然此狀態並非資源狀態的明確指示，卻是疑難排解程序中的重要資料點。

如果資源如預期般執行，幾分鐘後資源的狀態會變更為 [可用]。

如果您遇到資源問題，[不明] 健康狀態可能暗示資源受到平台事件影響。

![處於 [不明] 狀態的虛擬機器](./media/resource-health-overview/Unknown.png)

### <a name="degraded"></a>已降級
[已降級] 健康狀態表示資源已偵測到效能降低，不過仍可供使用。
不同資源對於其指定資源的降級有自己的準則。

![處於 [已降級] 狀態的虛擬機器](./media/resource-health-overview/degraded.png)

## <a name="reporting-an-incorrect-status"></a>報告不正確的狀態
如果您認為目前的健康狀態不正確，您可以選取 [報告不正確的健康狀態] 來告知我們。 在您受到 Azure 問題影響的情況下，建議您從 [資源健康狀態] 連絡支援人員。 

![用於提交不正確狀態相關資訊的方塊](./media/resource-health-overview/incorrect-status.png)

## <a name="historical-information"></a>歷程記錄資訊
您可以在 [資源健康狀態] 的 [健康史] 區段中存取最多 14 天的健康狀態歷程記錄。 

![過去兩週的資源健康狀態事件清單](./media/resource-health-overview/history-blade.png)

## <a name="getting-started"></a>開始使用
若要開啟一個資源的資源健康狀態：
1.  登入 Azure 入口網站。
2.  瀏覽至您的資源。
3.  在左窗格中的資源功能表上，選取 [資源健康狀態]。

![從 [資源] 檢視開啟 [資源健康狀態]](./media/resource-health-overview/from-resource-blade.png)

選取 [所有服務]，然後在篩選文字方塊中輸入**資源健康狀態**，也可以存取資源健康狀態。 在 [說明 + 支援] 窗格中，選取[資源健康狀態](https://ms.portal.azure.com/#blade/Microsoft_Azure_Monitoring/AzureMonitoringBrowseBlade/resourceHealth)。

![從 [所有服務] 開啟 [資源健康狀態]](./media/resource-health-overview/FromOtherServices.png)

## <a name="next-steps"></a>後續步驟

若要深入了解資源健康狀態，請參閱下列資源：
-  [Azure 資源健康狀態中的資源類型和健康狀態檢查](resource-health-checks-resource-types.md)
-  [關於 Azure 資源健康狀態的常見問題集](resource-health-faq.md)




