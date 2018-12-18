---
title: 使用 Azure 排程器來建立排定的工作 - Azure 入口網站 | Microsoft Docs
description: 了解如何在 Azure 入口網站中使用 Azure 排程器來建立、排定及執行您的第一個自動化工作
services: scheduler
ms.service: scheduler
ms.suite: infrastructure-services
author: derek1ee
ms.author: deli
ms.reviewer: klam
ms.assetid: e69542ec-d10f-4f17-9b7a-2ee441ee7d68
ms.topic: hero-article
ms.date: 09/17/2018
ms.openlocfilehash: f1f7e67fbd5d8a9ebfae03c00eb0de36e86d9a97
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46949582"
---
# <a name="create-and-schedule-your-first-job-with-azure-scheduler---azure-portal"></a>使用 Azure 排程器來建立及排定您的第一個工作 - Azure 入口網站

> [!IMPORTANT]
> [Azure Logic Apps](../logic-apps/logic-apps-overview.md) 將會取代即將淘汰的「Azure 排程器」。 若要排定工作，請[改為嘗試 Azure Logic Apps](../scheduler/migrate-from-scheduler-to-logic-apps.md)。 

本教學課程說明如何輕鬆建立及排定工作，然後監視及管理該工作。 

如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。

## <a name="create-job"></a>建立工作

1. 登入 [Azure 入口網站](https://portal.azure.com/)。  

1. 在 Azure 主功能表上，選取 [建立資源]。 在搜尋方塊中，輸入「排程器」。 從結果清單中，選取 [排程器]，然後選擇 [建立]。

   ![建立排程器資源](./media/scheduler-get-started-portal/scheduler-v2-portal-marketplace-create.png)

   現在，建立一個將 GET 要求傳送至以下 URL 的工作：`http://www.microsoft.com/` 

1. 在 [排程器工作] 底下，輸入下列資訊：

   | 屬性 | 範例值 | 說明 |
   |----------|---------------|-------------| 
   | **名稱** | getMicrosoft | 工作的名稱 | 
   | **工作集合** | <*job-collection-name*> | 建立工作集合，或選取現有的集合。 | 
   | **訂用帳戶** | <*Azure-subscription-name*> | Azure 訂用帳戶的名稱 | 
   |||| 

1. 選取 [動作設定] - [設定]，提供下列資訊，然後在完成時選擇 [確定]：

   | 屬性 | 範例值 | 說明 |
   |----------|---------------|-------------| 
   | **動作** | **Http** | 要執行的動作類型 | 
   | **方法** | **Get** | 要呼叫的方法 | 
   | **URL** | **http://www.microsoft.com** | 目的地 URL | 
   |||| 
   
   ![定義工作](./media/scheduler-get-started-portal/scheduler-v2-portal-action-settings.png)

1. 選取 [排程] - [設定]，定義排程，然後在完成時選取 [確定]：

   雖然您可以建立一次性工作，但此範例會設定週期排程。

   | 屬性 | 範例值 | 說明 |
   |----------|---------------|-------------| 
   | **週期** | **週期性** | 一次性或週期性工作 | 
   | **開始時間** | <*今天的日期*> | 工作的開始日期 | 
   | **重複頻率** | **1 小時** | 週期間隔和頻率 | 
   | **結束** | **結束於**今天日期的兩天後 | 工作的結束日期 | 
   | **UTC 時差** | **UTC +08:00** | 「國際標準時間」(UTC) 與您位置觀測時間之間的差距 | 
   |||| 

   ![定義排程](./media/scheduler-get-started-portal/scheduler-v2-portal-recurrence-schedule.png)

1. 準備就緒時，請選擇 [建立]。

   在您建立工作之後，Azure 會部署您的工作 (這些工作會出現在 Azure 儀表板上)。 

1. 當 Azure 顯示部署成功的通知時，選擇 [釘選到儀表板]。 否則，請選擇 Azure 工具列上的 [通知] 圖示 (鐘)，然後選擇 [釘選到儀表板]。

## <a name="monitor-and-manage-jobs"></a>監視及管理工作

若要檢閱、監事及管理工作，請在 Azure 儀表板上選擇您的工作。 在 [設定] 底下，有您可針對工作進行檢閱及管理的區域：

![工作設定](./media/scheduler-get-started-portal/scheduler-v2-portal-job-overview-1.png)

如需有關這些區域的詳細資訊，請選取一個區域：

* [**屬性**](#properties)
* [**動作設定**](#action-settings)
* [**排程**](#schedule)
* [**歷程記錄**](#history)
* [**使用者**](#users)

<a name="properties"></a>

### <a name="properties"></a>屬性

若要檢視描述工作管理中繼資料的唯讀屬性，請選取 [屬性]。

![檢視工作屬性](./media/scheduler-get-started-portal/scheduler-v2-portal-job-properties.png)

<a name="action-settings"></a>

### <a name="action-settings"></a>動作設定

若要變更工作的進階設定，請選取 [工作設定]。 

![檢閱動作設定](./media/scheduler-get-started-portal/scheduler-v2-portal-job-action-settings.png)

| 動作類型 | 說明 | 
|-------------|-------------| 
| 所有類型 | 您可以變更 [重試原則] 和 [錯誤動作] 設定。 | 
| HTTP 和 HTTPS | 您可以將 [方法] 變更為任何允許的方法。 您也可以新增、刪除或變更標頭和基本驗證資訊。 | 
| 儲存體佇列| 您可以變更儲存體帳戶、佇列名稱、SAS 權杖及主體。 | 
| 服務匯流排 | 您可以變更命名空間、主題或佇列路徑、驗證設定、傳輸類型、訊息屬性及訊息本文。 | 
||| 

<a name="schedule"></a>

### <a name="schedule"></a>排程

如果您透過工作精靈設定排程，便可變更該排程，例如開始日期和時間、週期排程，以及週期性工作的結束日期和時間。
您也可以建置更[複雜的排程和進階週期](scheduler-advanced-complexity.md)。

若要變更檢視或變更工作的排程，請選取 [排程]：

![檢視工作排程](./media/scheduler-get-started-portal/scheduler-v2-portal-job-schedule.png)

<a name="history"></a>

### <a name="history"></a>歷程記錄

若要檢視所選工作之每個執行項目的相關計量，請選取 [歷程記錄]。 這些計量提供有關您工作健康情況的即時值，例如狀態、重試次數、發生次數、開始時間及結束時間。

![檢視工作歷程記錄和計量](./media/scheduler-get-started-portal/scheduler-v2-portal-job-history.png)

若要檢視每個執行項目的歷程記錄詳細資料 (例如每個執行項目的完整回應)，請在 [歷程記錄] 底下，選取每個執行項目。 

![檢視工作歷程記錄詳細資料](./media/scheduler-get-started-portal/scheduler-v2-portal-job-history-details.png)

<a name="users"></a>

### <a name="users"></a>使用者

您可以使用「Azure 角色型存取控制」(RBAC)，在細微層級管理每個使用者對「Azure 排程器」的存取權。 若要了解如何根據角色設定存取權，請參閱[使用 RBAC 來管理存取權](../role-based-access-control/role-assignments-portal.md)

## <a name="next-steps"></a>後續步驟

* 了解[概念、術語及實體階層](scheduler-concepts-terms.md)
* [建置複雜的排程和進階週期](scheduler-advanced-complexity.md)
* 了解[排程器高可用性和可靠性](scheduler-high-availability-reliability.md)
* 了解[限制、配額、預設值及錯誤碼](scheduler-limits-defaults-errors.md)
