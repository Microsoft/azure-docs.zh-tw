---
title: Azure 監視代理程式概觀 | Microsoft Docs
description: 本文提供可支援 Azure VM 監視的 Azure 代理程式詳細概觀。
services: log-analytics
documentationcenter: log-analytics
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/28/2018
ms.author: magoedte
ms.openlocfilehash: a399c3968e5ee1e2d1f6d623a68dbb1e15cef212
ms.sourcegitcommit: d1eefa436e434a541e02d938d9cb9fcef4e62604
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/28/2018
ms.locfileid: "37088416"
---
# <a name="overview-of-the-azure-agents-to-monitor-azure-virtual-machines"></a>用以監視 Azure 虛擬機器的 Azure 代理程式概觀
Microsoft Azure 提供多種方法，可從 Azure 或其他執行 Microsoft Windows 和 Linux 的雲端提供者中裝載的虛擬機器，收集不同類型的資料。  本文將協助說明每個代理程式的差異和可用功能，以便您判斷哪個代理程式將支援服務管理或一般監視需求。  

## <a name="comparing-azure-diagnostic-and-log-analytics-agent"></a>比較 Azure 診斷和 Log Analytics 代理程式
在 Azure 中，目前有兩種代理程式可用於監視 Azure VM - Azure 診斷擴充功能和適用於 Linux 和 Windows 的 Log Analytics 代理程式。  基本上，這些代理程式的設計訴求是收集計量和記錄，並轉送至存放庫。 不過，其相似之處僅止於此。  

[Azure 診斷擴充功能](../monitoring-and-diagnostics/azure-diagnostics.md)是一個代理程式 (已在 2010 年 Azure 雲端服務正式推出後提供)，可供從 Azure IaaS 資源 (例如 VM) 簡單收集診斷資料，並將它保存到 Azure 儲存體。  放入儲存體後，您便選擇使用其中一個可用工具進行檢視，例如 [Visual Studio 中的伺服器總管](../vs-azure-tools-storage-resources-server-explorer-browse-manage.md)和 [Azure 儲存體總管](../vs-azure-tools-storage-manage-with-storage-explorer.md)。

您可以選擇收集：

* 一組預先定義的作業系統效能計數器和事件記錄，也您可以指定要收集的項目。 
* IIS Web 伺服器的所有要求和/或失敗要求
* .NET 應用程式追蹤輸出記錄
* Windows 事件追蹤 (ETW) 事件 
* 從 syslog 收集事件  
* 損毀傾印 

或者，也可以將資料轉送至 [Application Insights](../application-insights/app-insights-cloudservices.md)、[Log Analytics](../log-analytics/log-analytics-overview.md)，或使用[事件中樞](../event-hubs/event-hubs-what-is-event-hubs.md)的非 Azure 服務。 

若要進行不只需要收集計量和一部分記錄的進階監視，一定要有適用於 Windows 和 Linux 的 Log Analytics 代理程式。  使用代理程式，您就能夠利用 Azure 服務 (例如自動化和 Log Analytics，包括它們所提供的整組功能) 在 Azure VM 的生命週期中提供全方位的管理。 其中包括：

* 作業系統更新的 [Azure 自動化更新管理](../automation/automation-update-management.md)
* [Azure 自動化 Desired State Configuration](../automation/automation-dsc-overview.md) 用以維護一致的組態狀態
* 使用 [Azure 自動化變更追蹤與清查](../automation/automation-change-tracking.md)來追蹤組態變更
* 透過 Log Analytics 從 OS 和託管應用程式收集自訂記錄，例如 [FluentD](../log-analytics/log-analytics-data-sources-json.md)、[自訂記錄](../log-analytics/log-analytics-data-sources-custom-logs.md)、[MySQL 和 Apache](../log-analytics/log-analytics-data-sources-linux-applications.md)
* [Application Insights](https://docs.microsoft.com/azure/application-insights/) 和 [Azure 資訊安全中心](https://docs.microsoft.com/azure/security-center/)等 Azure 服務會在原生狀態將其資料直接儲存在 Log Analytics 中。  

## <a name="next-steps"></a>後續步驟

- 請參閱[使用 Log Analytics 從您環境中的電腦收集資料](../log-analytics/log-analytics-concept-hybrid.md)，以檢閱需求和可用方法，進而將代理程式部署至您資料中心或其他雲端環境中的電腦。
- 請參閱[收集關於 Azure 虛擬機器的資料](../log-analytics/log-analytics-quick-collect-azurevm.md)，以進行從 Azure VM 收集資料的設定。 