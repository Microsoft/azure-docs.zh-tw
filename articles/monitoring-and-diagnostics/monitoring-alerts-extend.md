---
title: 擴充 (複製) Log Analytics 至 Azure 警示 - 概觀
description: 將警示從 OMS 入口網站的 Log Analytics 複製到 Azure 警示的程序概觀，詳細說明常見的客戶考量。
author: msvijayn
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 05/24/2018
ms.author: vinagara
ms.component: alerts
ms.openlocfilehash: 13a84b7254207a9cfcfff4af43283130a0f6c587
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46998469"
---
# <a name="extend-log-analytics-alerts-to-azure-alerts"></a>將 Log Analytics 警示擴充到 Azure 警示
直到最近，Azure Log Analytics 剛剛加入了自己的警示功能，可根據 Log Analytics 資料主動通知您情況。 您之前可在 Microsoft Operations Management Suite 中管理警示規則。 全新警示體驗現已整合 Microsoft Azure 中各種服務之間的警示。 此功能即為 Azure 入口網站中的 Azure 監視器下的 **[警示]**，支援 Log Analytics 和 Azure Application Insights 中活動記錄、計量和記錄的警示。 

## <a name="benefits-of-extending-your-alerts"></a>擴充您警示的好處
在 Azure 入口網站建立和管理警示有幾項好處，例如：

- 不同於在 Operations Management Suite 入口網站中只能建立及檢視 250 個警示，在 Azure 警示中並沒有這項限制。
- 透過 Azure 警示，您可以管理、列舉和檢視所有警示類型。 之前您只能這樣處理 Log Analytics 警示，
- 現在您可以使用 [Azure 監視器角色](monitoring-roles-permissions-security.md)，將使用者的存取權限制於僅供監視和警示。
- 您可以在 Azure 警示使用[動作群組](monitoring-action-groups.md)。 此功能允許每個警示有多個動作。 您可以發送簡訊、撥打語音電話、叫用 Azure 自動化 Runbook、叫用 Webhook，並設定 IT 服務管理 (ITSM) 連接器。 

## <a name="process-of-extending-your-alerts"></a>擴充警示的程序
將警示從 Log Analytics 移至 Azure 警示的程序不包含以任何方式變更您的警示定義、查詢或設定。 唯一需要的變更是，在 Azure 中使用動作群組執行所有動作。 如果動作群組已經與警示相關聯，警示擴充至 Azure 時就會包含在內。

> [!NOTE]
> Microsoft 自 2018 年 5 月 14 日起，會定期自動將在 Log Analytics 公用雲端執行個體中建立的警示擴充至 Azure 警示，直到完成作業。 如果您建立[動作群組](monitoring-action-groups.md)時遇到問題，請使用[這些補救步驟](monitoring-alerts-extend-tool.md#troubleshooting)讓系統自動建立動作群組。 2018 年 7 月 5 日之前，您可以使用這些步驟。 *不適用於 Log Analytics 的 Azure Goverment and Soveriegn 雲端使用者*。 
> 

當您排程將 Log Analytics 工作區中的警示擴充至 Azure 以後，這些警示會繼續運作，且不會以任何方式妨礙您的設定。 當排程時，您的警示可能暫時無法進行修改，但您可以在這段時間內繼續建立新的 Azure 警示。 如果您嘗試編輯或從 Operations Management Suite 入口網站建立警示，您可以選擇繼續從 Log Analytics 工作區建立警示。 您也可以選擇在 Azure 入口網站中透過 Azure 警示建立警示。

 ![從 Log Analytics 或 Azure 警示建立警示的選項螢幕擷取畫面](./media/monitor-alerts-extend/ScheduledDirection.png)

> [!NOTE]
> 將警示從 Log Analytics 擴充至 Azure 不會向您的帳戶收取費用。 如果您遵循 [Azure 監視器定價原則](https://azure.microsoft.com/pricing/details/monitor/)中所述的限制及條件，用於查詢式 Log Analytics 警示的 Azure 警示也不會計費。  


### <a name="how-to-extend-your-alerts-voluntarily"></a>如何主動擴充您的警示
若要將您的警示擴充到 Azure 警示，您可以使用 Operations Management Suite 入口網站中提供的精靈，或者使用 API 以程式設計方式執行。 如需詳細資訊，請參閱[使用 Operations Management Suite 入口網站和 API 將警示擴充至 Azure](monitoring-alerts-extend-tool.md)。

## <a name="experience-after-extending-your-alerts"></a>擴充警示之後的體驗
將警示擴充至 Azure 警示後，您可如同往常一樣繼續在 Operations Management Suite 入口網站中加以管理。

![Operations Management Suite 入口網站的螢幕擷取畫面，包含列出的警示](./media/monitor-alerts-extend/PostExtendList.png)

當您嘗試編輯現有的警示，或在 Operations Management Suite 入口網站中建立新的警示時，您會自動重新導向 Azure 警示。  

> [!NOTE]
> 請確認已在 Azure 中正確指派需要加入或編輯警示的人員所需的相關權限。 若要了解您需要授予何種權限，請參閱[使用 Azure 監視器和警示的權限](monitoring-roles-permissions-security.md)。  
> 

您可以繼續透過 [Log Analytics API](../log-analytics/log-analytics-api-alerts.md) 和 [Log Analytics 資源範本](../monitoring/monitoring-solutions-resources-searches-alerts.md)建立警示。 這麼做時，請務必加入動作群組。

## <a name="next-steps"></a>後續步驟

* 了解[啟動將警示從 Log Analytics 擴充至 Azure](monitoring-alerts-extend-tool.md)的工具。
* 深入了解 [Azure 警示體驗](monitoring-overview-unified-alerts.md)。
* 了解如何建立 [Azure 警示中的記錄警示](monitor-alerts-unified-log.md)。
