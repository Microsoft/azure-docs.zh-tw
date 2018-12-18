---
title: 管理 Azure Log Analytics 中的資料成本 | Microsoft Docs
description: 了解如何在 Azure 中變更 Log Analytics 工作區的定價方案，以及管理資料量和保留原則。
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
ms.topic: conceptual
ms.date: 08/27/2018
ms.author: magoedte
ms.component: na
ms.openlocfilehash: 179d5c71a309fd79c23519544d1da6bcc07cda0c
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43044995"
---
# <a name="manage-cost-by-controlling-data-volume-and-retention-in-log-analytics"></a>藉由控制 Log Analytics 中的資料量與保留期來管理成本

> [!NOTE]
> 本文說明如何藉由設定資料保留期間來控制 Log Analytics 的成本。  請參閱下列文章以了解相關資訊。
> - [分析 Log Analytics 中的資料使用量](log-analytics-manage-cost-storage.md)說明如何分析資料使用量及提出警示。
> - [監視使用量和估計成本](../monitoring-and-diagnostics/monitoring-usage-and-estimated-costs.md)說明如何針對不同的定價模型，檢視多項 Azure 監視功能的使用量和估計成本。 其中也會說明如何變更定價模型。

Log Analytics 是為縮放及支援每日大量資料的收集、索引編製和儲存而設計，這些資料來源可能位於您企業內部或部署於 Azure 中。  雖然這可能是您組織的主要推動力，但成本效率終究是最基本的推動力。 為此，務必了解 Log Analytics 工作區的成本不只是以收集的資料量為基礎，也取決於選取的方案，以及您為連線來源所產生的資料選擇了多長的儲存時間。  

在本文中，我們會探討您可以如何積極地監視資料量和儲存體成長狀況，並定義限制來控制這些相關成本。 

資料成本是否會變得可觀取決於下列因素： 

- 您要從中收集資料的系統、基礎結構元件、雲端資源等的數目 
- 來源所建立的資料類型 (例如訊息佇列、記錄、事件、安全性相關資料，或效能計量) 
- 這些來源產生的資料量和內嵌至工作區的資料量 
- 在工作區中保留資料的期限  
- 已啟用的管理解決方案數目、資料來源及收集頻率 

> [!NOTE]
> 請參閱每個解決方案的文件，因為它會提供所收集資料數量的預估值。   

在 2018 年 7 月 1 日前簽署 Enterprise 合約的客戶，或已在訂用帳戶中建立 Log Analytics 工作區的客戶，仍然可以存取「免費」方案。 如果訂用帳戶未繫結至現有的 EA 註冊，當您在 2018 年 4 月 2 日之後於新的訂用帳戶中建立工作區時，不適用「免費」層。  「免費」層的資料保留期僅限 7 天。  如果是 [獨立] 或 [付費] 層，則可提供過去 31 天收集的資料。 「免費」層有每日 500 MB 的擷取限制，如果發現一直超出所允許的數量，您可以將工作區變更成付費方案，以收集超出此限制的資料。 

> [!NOTE]
> 如果您選擇為付費層選取一個較長的保留期，就需要支付費用。 您可以隨時變更方案類型，如需有關定價的詳細資訊，請參閱[定價詳細資料](https://azure.microsoft.com/pricing/details/log-analytics/)。 

限制資料量並協助控制成本的兩種方式為設定每日上限和資料保留期。  

## <a name="review-estimated-cost"></a>檢閱估計成本
Log Analytics 可讓您根據最近的使用模式來輕鬆地了解可能的成本。  若要這樣做，請執行下列步驟。  

1. 登入 [Azure 入口網站](http://portal.azure.com)。 
2. 在 Azure 入口網站中，按一下 [所有服務]。 在資源清單中輸入 **Log Analytics**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 選取 [Log Analytics]。<br><br> ![Azure 入口網站](media/log-analytics-quick-collect-azurevm/azure-portal-01.png)<br><br>  
3. 在 Log Analytics 訂用帳戶窗格中，從左側窗格選取您的工作區，然後按一下 [使用量和估計成本]。<br><br> ![使用量和估計成本頁面](media/log-analytics-manage-cost-storage/usage-estimated-cost-dashboard-01.png)<br>

您可以在此檢閱當月的資料量。 這之中包括 Log Analytics 工作區中接收和保留的所有資料。  按一下頁面頂端的 [使用量詳細資料]可檢視使用量儀表板，其中的資訊包含以來源、電腦及供應項目為依據的資料量趨勢。 若要檢視和設定每日上限或修改保留期限，請按一下 [資料量管理]。
 
Log Analytics 費用會新增到您的 Azure 帳單中。 您可以在 Azure 入口網站中的 [帳務] 區段下，或在 [Azure 計費入口網站](https://account.windowsazure.com/Subscriptions)中，查看 Azure 帳單的詳細資料。  

## <a name="daily-cap"></a>每日上限
如果從 Azure 入口網站建立 Log Analytics 工作區，並選擇「免費」方案，則工作區會有每天 500 MB 的限制。 其他定價方案則沒有任何限制。 您可以為工作區設定每日上限和限制每日擷取，但請謹慎使用，因為您的目標是不要達到每日限制。  否則，您會遺失當天其餘時間的資料，這可能會影響其功能取決於工作區中一直在提供的最新資料的其他 Azure 服務和解決方案。  如此一來，您在資源健全狀況支援 IT 服務時觀察和接收警示的功能會受到影響。  每日上限旨在用來管理受控資源中未預期的資料量增加，以及保持在您的限制範圍內，或是純粹用於限制意外產生的工作區費用。  

若達到每日限制，將停止收集當天剩餘時間的需計費資料類型。 在所選取的 Log Analytics 工作區中，頁面頂端會出現警告橫幅，且系統會將作業事件傳送至 **LogManagement** 類別下的「Operation」 資料表。 一旦過了「每日限制的設定時間」下定義的重設時間後，資料收集就會繼續執行。 我們建議您根據此作業事件定義警示規則，設定為在達到每日資料限制時發出通知。 

### <a name="identify-what-daily-data-limit-to-define"></a>識別要定義的每日資料限制 
檢閱 [Log Analytics 使用量和估計成本](log-analytics-usage.md)，以了解資料擷取趨勢及要定義的每日資料量上限。 此謹慎考量，因為達到限制之後，您將無法監視您的資源。 

### <a name="manage-the-maximum-daily-data-volume"></a>管理每日資料量上限 
下列步驟說明如何設定限制，以管理 Log Analytics 每日擷取的資料量。  

1. 在工作區中，從左側窗格中選取 [使用量和估計成本]。
2. 在所選工作區的 [使用量和估計成本] 頁面上，按一下頁面頂端的 [資料量管理]。 
5. 每日上限預設為 [關閉] – 請按一下 [開啟] 來加以啟用，然後設定資料量限制 (GB/天)。<br><br> ![Log Analytics 設定資料限制](media/log-analytics-manage-cost-storage/set-daily-volume-cap-01.png)

### <a name="alert-when-limit-reached"></a>達到限制時產生警示
雖然我們會在您達到資料限制閾值時，於 Azure 入口網站中顯示視覺提示，但對於需要立刻處理的作業問題，此行為並不一定與您的管理方式一致。  若要接收警示通知，您可以在 Azure 監視器中建立新的警示規則。  若要進一步了解，請參閱[如何建立、檢視及管理警示](../monitoring-and-diagnostics/monitor-alerts-unified-usage.md)。      

為協助您開始使用，以下是警示的建議設定：

* 目標：選取您的 Log Analytics 資源
* 準則： 
   * 訊號名稱：自訂記錄搜尋
   * 搜尋查詢：Operation | where Detail has 'OverQuota'
   * 依據：結果數目
   * 條件：大於
   * 閾值：0
   * 週期：5 (分鐘)
   * 頻率︰5 (分鐘)
* 警示規則名稱：已達到每日資料限制
* 嚴重性：警告 (Sev 1)

一旦定義警示且達到限制後，警示就會觸發，並執行動作群組中定義的回應。 其可以透過電子郵件和文字簡訊通知您的小組，或使用 Webhook、自動化 Runbook 或[與外部 ITSM 方案整合](log-analytics-itsmc-overview.md#create-itsm-work-items-from-azure-alerts)來自動採取動作。 

## <a name="change-the-data-retention-period"></a>變更資料保留期 
下列步驟說明如何設定您工作區中的記錄資料保留時間。
 
1. 在工作區中，從左側窗格中選取 [使用量和估計成本]。
2. 在 [使用量和估計成本] 頁面上，按一下頁面頂端的 [資料量管理]。
5. 在窗格上，移動滑桿來增加或減少天數，然後按一下 [確定]。  如果您位於「免費」層，將無法修改資料保留期，必須升級至付費層，才能控制此設定。<br><br> ![變更工作區資料保留期設定](media/log-analytics-manage-cost-storage/manage-cost-change-retention-01.png)

## <a name="troubleshooting"></a>疑難排解
**問題**：如何針對 Log Analytics 不再收集資料的問題進行疑難排解？ 
**答**：如果您在免費定價層並在某天傳送了超過 500 MB 的資料，就會停止收集當天其餘資料。 達到每日限制是 Log Analytics 停止收集資料或資料似乎遺失的常見原因。  
當資料收集開始及停止時，Log Analytics 會建立 Operation 類型的事件。  
在搜尋中執行下列查詢，即可檢查您是否達到每日限制並遺失資料：Operation | where OperationCategory == 'Data Collection Status'   
當資料收集停止時，OperationStatus 為 Warning。 當資料收集開始時，OperationStatus 為 Succeeded。  
下表描述資料收集停止的原因，並建議為繼續資料收集所要採取的動作：  

|收集停止的原因| 解決方法| 
|-----------------------|---------|
|已達免費資料的每日限制<sup>1</sup>|請等到隔天自動重新開始收集，或變更為付費定價層。|
|已達到您在資料量管理中定義的每日限制|等到隔天自動重新開始收集，或如[管理每日資料量上限](#manage-the-maximum-daily-volume)中的描述，增加每日資料量限制|
|Azure 訂用帳戶處於暫停狀態，原因如下：<br> 免費試用已結束<br> Azure Pass 已過期<br> 已達每月消費限制 (例如 MSDN 或 Visual Studio 訂閱)|轉換成付費訂閱<br> 移除限制，或等到限制重設|

<sup>1</sup> 如果您的工作區在免費定價層，您每天最多可傳送 500 MB 的資料至服務。 當您達到每日限制時，資料收集就會停止，直到隔天再開始。 在資料收集停止時所傳送的資料不會編製索引，而且無法供搜尋使用。 當資料收集繼續時，只會處理新的傳送資料。 

Log Analytics 採用 UTC 時間。 各個工作區之間的重設時間並不相同，藉此防止所有已設定上限的工作區在同個時間開始擷取資料。 如果工作區達到每日限制，在過了「**每日限制的設定時間**」下定義的重設時間後，工作區即會繼續處理程序。<br><br> ![Log Analytics 限制 UTC 時區](media/log-analytics-manage-cost-storage/data-volume-mgmt-limit-utc.png)

**問題**：如何在資料收集停止時收到通知？ 
**答**：請使用「建立每日資料上限」警示中所述的步驟，以在資料收集停止時收到通知，並使用將動作新增至警示規則中所述的步驟，來設定警示規則的電子郵件、Webhook 或 Runbook 動作。 

## <a name="next-steps"></a>後續步驟  

若要判斷收集的資料有多少、哪些來源傳送這些資料及所傳送資料的不同類型，以協助管理取用和成本，請參閱[在 Log Analytics 中分析資料使用量](log-analytics-usage.md)。