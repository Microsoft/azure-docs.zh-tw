---
title: Azure Stack 上的 Azure 監視器 | Microsoft Docs
description: 了解 Azure Stack 上的 Azure 監視器。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/05/2018
ms.author: mabrigg
ms.openlocfilehash: b0cf2d7856a78bbe2aa531c6e872168e8e33b06a
ms.sourcegitcommit: d211f1d24c669b459a3910761b5cacb4b4f46ac9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "44021806"
---
# <a name="azure-monitor-on-azure-stack"></a>Azure Stack 上的 Azure 監視器

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

本文提供 Azure Stack 中 Azure 監視器服務的概觀。 文中會討論 Azure 監視器的作業，以及關於如何在 Azure Stack 上使用 Azure 監視器的其他資訊。 

如需簡介、概觀以及如何開始使用 Azure 監視器，請參閱全域 Azure 文章[開始使用 Azure 監視器](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-get-started)。

![Azure Stack [監視器] 刀鋒視窗](./media/azure-stack-metrics-azure-data/azs-monitor.png)

Azure 監視器是平台服務，提供監視 Azure 資源的單一來源。 您可以使用 Azure 監視器來視覺化、查詢、路由、封存，以及針對來自 Azure 資源的度量和記錄檔採取其他行動。 您可以使用 Azure Stack 管理員入口網站、監視器 PowerShell Cmdlet、跨平台 CLI 或 Azure 監視器 REST API 來使用此資料。 若要了解 Azure Stack 所支援的特定連線，請參閱[如何取用 Azure Stack 中的監視資料](azure-stack-metrics-monitor.md)

> [!Note]  
Azure Stack 開發套件無法使用計量和診斷記錄。

## <a name="prerequisites"></a>必要條件

在訂用帳戶的供應項目資源提供者設定上，註冊 **Microsoft.insights** 資源提供者。 您可以確認與訂用帳戶相關聯的供應項目中，是否有該資源提供者可供使用：

1. 開啟 Azure Stack 管理員入口網站。
2. 選取 [供應項目]。
3. 選取與訂用帳戶相關聯的供應項目。
4. 在 [設定] 下，選取 [資源提供者]。 
5. 在清單中尋找 **Microsoft.Insights**，並確認其狀態為 [已註冊]。

## <a name="overview"></a>概觀

和 Azure 上的 Azure 監視器一樣，Azure Stack 上的 Azure 監視器也可為大部分服務提供基底層級的基礎結構計量和記錄。

## <a name="azure-monitor-sources-compute-subset"></a>Azure 監視器來源：計算子集

![Azure 監視器來源 - 計算子集](media//azure-stack-metrics-azure-data/azs-monitor-computersubset.png)

Azure Stack 中的 **Microsoft.Compute** 資源提供者包括：
 - 虛擬機器 
 - 虛擬機器擴展集

### <a name="application---diagnostics-logs-application-logs-and-metrics"></a>應用程式 - 診斷記錄、應用程式記錄及計量

在具有 **Microsoft.Compute** 資源提供者的執行中 VM OS 上，可以執行應用程式。 這些應用程式和 VM 會發出自己的記錄和計量集合。 Azure 監視器依賴 Azure 診斷擴充功能 (Windows 或 Linux) 來收集大多數應用程式等級的計量和記錄。 

量值的類型包括：
 - 效能計數器
 - 應用程式記錄檔
 - Windows 事件記錄檔
 - .NET 事件來源
 - IIS 記錄檔
 - 以資訊清單為基礎的 ETW
 - 損毀傾印
 - 客戶錯誤記錄

> [!Note]  
> 不支援 Azure Stack 上的 Linux 診斷擴充功能。

### <a name="host-and-guest-vm-metrics"></a>主機和客體 VM 計量

先前所列的計算資源具有專用主機 VM 和客體 OS。 主機 VM 和客體 OS 等同於 Hyper-V Hypervisor 中的根 VM 和客體 VM。 您可以同時針對主機 VM 和客體 OS 收集其計量。 此外，您也可以針對客體 OS 收集診斷記錄。 [Azure Stack 上 Azure 監視器的受支援計量](azure-stack-metrics-supported.md)中有 Azure Stack 上主機和客體 VM 計量的可收集計量清單。 

### <a name="activity-log"></a>活動記錄檔

您可以搜尋活動記錄以取得 Azure Stack 基礎結構所見計算資源的相關資訊。 此記錄檔包含資源的建立或終結時間等資訊。 Azure Stack 上的活動記錄會與 Azure 一致。 如需詳細資訊，請參閱 [Azure 上的活動記錄概觀](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs)說明。 


## <a name="azure-monitor-sources-everything-else"></a>Azure 監視器來源：其他項目

![Azure 監視器來源 - 其他項目](media//azure-stack-metrics-azure-data/azs-monitor-othersubset.png)

### <a name="resources---metrics-and-diagnostics-logs"></a>資源 - 計量和診斷記錄

可收集的計量和診斷記錄視資源類型而異。 支援的計量中提供可針對 Azure Stack 上每個資源收集的計量清單。 如需詳細資訊，請參閱 [Azure Stack 上 Azure 監視器的受支援計量](azure-stack-metrics-supported.md)。

### <a name="activity-log"></a>活動記錄檔

所有計算資源的活動記錄都一樣。 

### <a name="uses-for-monitoring-data"></a>監視資料的用途

**儲存與封存**  

某些監視資料已經在 Azure 監視器中儲存一段時間且可供使用。 
 - 計量會儲存 90 天。 
 - 活動記錄項目會儲存 90 天。 
 - 診斷記錄不會儲存起來。
 - 請將資料封存至儲存體帳戶，來延長保留時間。

**查詢**  

您可以使用 Azure 監視器 REST API、跨平台命令列介面 (CLI) 命令、PowerShell Cmdlet 或 .NET SDK 來存取系統或是 Azure 儲存體中的資料。 

**視覺效果**

以圖形和圖表視覺化您的監視資料，協助您透過比查看資料本身更快的速度找出趨勢。 

幾個視覺化方法包括︰
 - 使用 Azure Stack 使用者和管理員入口網站
 - 將資料路由至 Microsoft Power BI
 - 將資料路由至協力廠商視覺化工具，無論是透過即時串流，或是讓該工具讀取 Azure 儲存體中的封存

## <a name="methods-of-accessing-azure-monitor-on-azure-stack"></a>存取 Azure Stack 上 Azure 監視器的方法

一般來說，您可以使用下列其中一個方法操作資料追蹤、路由及擷取。 並非所有方法都適用所有動作或資料類型。

 - [Azure Stack 入口網站](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-use-portal)
 - [PowerShell](https://docs.microsoft.com/azure/monitoring-and-diagnostics/insights-powershell-samples)
 - [跨平台命令列介面 (CLI)](https://docs.microsoft.com/azure/monitoring-and-diagnostics/insights-cli-samples)
 - [REST API](https://docs.microsoft.com/rest/api/monitor)
 - [.NET SDK](http://www.nuget.org/packages/Microsoft.Azure.Management.Monitor)

## <a name="next-steps"></a>後續步驟

在[取用 Azure Stack 中的監視資料](azure-stack-metrics-monitor.md)一文中深入了解在 Azure Stack 上監視資料取用量的選項。
