---
title: 將 Azure 資源的記錄與 SIEM 系統整合 | Microsoft Docs
description: 了解 Azure 記錄整合、其主要功能及運作方式。
services: security
documentationcenter: na
author: TomShinder
manager: MBaldwin
editor: TerryLanfear
ms.assetid: 9c1346e1-baf8-4975-b2f2-42ae05b2dc0a
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/07/2018
ms.author: TomSh
ms.custom: azlog
ms.openlocfilehash: 3c875060a7abdf4431026e79ce966efdc89e4e77
ms.sourcegitcommit: 4e36ef0edff463c1edc51bce7832e75760248f82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/08/2018
ms.locfileid: "35236173"
---
# <a name="introduction-to-azure-log-integration"></a>Azure 記錄整合簡介

>[!IMPORTANT]
> Azure 記錄整合功能將於 2019 年 6 月 1 日淘汰。 AzLog 下載將於 2018 年 6 月 27 日停用。 如需繼續進行的指導，請檢閱 [Use Azure monitor to integrate with SIEM tools](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/) (使用 Azure 監視器與 SIEM 工具整合) 一文 

Azure 記錄整合可讓您簡化將 Azure 記錄與您的內部部署安全性資訊與事件管理 (SIEM) 系統整合的工作。

 整合 Azure 記錄的建議方法是使用您 SIEM 廠商的連接器。 Azure 監視器可讓您將記錄串流至事件中樞，而 SIEM 廠商可以撰寫連接器來進一步將記錄從事件中樞整合至 SIEM。  如需此運作方式的描述，請依照[監視用於監視資料事件中樞的資料流](../monitoring-and-diagnostics/monitor-stream-monitoring-data-event-hubs.md)中的指示進行。 該文也會列出已提供直接 Azure 連接器的 SIEM 清單。  

> [!IMPORTANT]
> 如果您的主要興趣在於收集虛擬機器記錄，大多數 SIEM 廠商會將此選項包含在其解決方案中。 使用 SIEM 廠商的連接器永遠是較好的替代做法。

Azure 記錄整合功能的相關文件仍會繼續維護，直到此功能被淘汰為止。

請進一步閱讀以深入了解 Azure 記錄整合功能：

Azure 記錄整合會從 Windows 事件檢視器記錄收集 Windows 事件，從 Azure 資源收集 [Azure 活動記錄](../monitoring-and-diagnostics/monitoring-overview-activity-logs.md)、[Azure 資訊安全中心警示](../security-center/security-center-intro.md)和 [Azure 診斷記錄](../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md)。 整合可協助您的 SIEM 解決方案為內部部署或雲端中的所有資產，提供一個整合儀表板。 您可以使用儀表板，來接收、彙總、相互關聯及分析安全性事件的警示。

> [!NOTE]
> Azure 記錄整合目前僅支援 Azure Commercial 和 Azure Government 雲端。 不支援其他雲端。

![Azure 記錄整合流程][1]

## <a name="what-logs-can-i-integrate"></a>可以整合哪些記錄檔？

Azure 會為每個 Azure 服務產生大量記錄。 記錄呈現三種記錄類型：

* **控制/管理記錄**：可讓您看到 [Azure Resource Manager](../azure-resource-manager/resource-group-overview.md) 的 CREATE、UPDATE 和 DELETE 作業。 Azure 活動記錄是此類型記錄的一個範例。
* **資料層記錄**：可讓您看到當您使用 Azure 資源時所引發的事件。 本類型的記錄檔範例是 Windows 事件檢視器**系統**、**安全性**和 Windows 虛擬機器中的**應用程式**頻道。 另一個範例是透過 Azure 監視器設定的 Azure 診斷記錄。
* **已處理的事件**：提供已為您處理且已分析的事件和警示資訊。 此類型事件的一個範例為 Azure 資訊安全中心警示。 Azure 資訊安全中心會處理並分析您的訂用帳戶，以提供與您目前安全性狀態相關的警示。

Azure 記錄整合支援 ArcSight、QRadar 及 Splunk。 請向您的 SIEM 廠商確認，以了解該廠商是否有原生的連接器。 如果有原生的連接器，請勿使用 Azure 記錄整合。

如果沒有其他選項可用，可考慮使用 Azure 記錄整合。 下表包含我們的建議：

|SIEM | 客戶已在使用 Azure 記錄整合器 | 客戶正在調查 SIEM 整合選項|
|---------|--------------------------|-------------------------------------------|
|**Splunk** | 開始移轉到 [Splunk 的 Azure 監視器附加元件](https://splunkbase.splunk.com/app/3534/) \(英文\)。 | 使用 [Splunk 連接器](https://splunkbase.splunk.com/app/3534/) \(英文\)。 |
|**QRadar** | 移轉到或開始使用[將 Azure 監視資料串流至事件中樞以供外部工具取用](../monitoring-and-diagnostics/monitor-stream-monitoring-data-event-hubs.md)最後一節中所述的 QRadar 連接器。 | 使用[將 Azure 監視資料串流至事件中樞以供外部工具取用](../monitoring-and-diagnostics/monitor-stream-monitoring-data-event-hubs.md)最後一節中所述的 QRadar 連接器。 |
|**ArcSight** | 繼續使用 Azure 記錄整合，直到有可用的連接器為止，然後移轉至以連接器為基礎的解決方案。  | 考慮使用 Azure Log Analytics 作為替代方案。 除非您願意在連接器變成可用時完成移轉程序，否則請不要上架到 Azure 記錄整合。 |

> [!NOTE]
> 雖然 Azure 記錄整合是免費的解決方案，但仍會有與記錄檔資訊儲存體相關的 Azure 儲存體成本。

如果需要協助，您可以建立[支援要求](../azure-supportability/how-to-create-azure-support-request.md)。 [服務] 請選取 [記錄整合]。

## <a name="next-steps"></a>後續步驟

本文為您介紹了 Azure 記錄整合。 若要深入了解 Azure 記錄整合和支援的記錄類型，請參閱下列文章：

* [開始使用 Azure 記錄整合](security-azure-log-integration-get-started.md)。 此教學課程將引導您完成安裝 Azure 記錄整合。 它也會說明如何整合 Windows Azure 診斷 (WAD) 儲存體的記錄、Azure 活動記錄、Azure 資訊安全中心警示，以及 Azure Active Directory 稽核記錄。
* [Azure 記錄整合常見問題集 (FAQ)](security-azure-log-integration-faq.md)。 此常見問題集會回答 Azure 記錄整合的相關常見問題。
* 深入了解如何[將 Azure 監視資料串流至事件中樞以供外部工具取用](../monitoring-and-diagnostics/monitor-stream-monitoring-data-event-hubs.md)。

<!--Image references-->
[1]: ./media/security-azure-log-integration-overview/azure-log-integration.png
