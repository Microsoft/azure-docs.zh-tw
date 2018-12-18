---
title: 在 Azure 資訊安全中心監視和處理安全性事件 | Microsoft Docs
description: 了解如何使用資訊安全中心的事件儀表板，查看 Azure 虛擬機器 (VM) 和非 Azure 電腦的安全性事件。
services: security-center
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: ''
ms.assetid: 70c076ef-3ad4-4000-a0c1-0ac0c9796ff1
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/05/2017
ms.author: terrylan
ms.openlocfilehash: 367067874b167268bd690a9e0b55412e92e08122
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23040653"
---
# <a name="monitoring-and-processing-security-events-in-azure-security-center"></a>在 Azure 資訊安全中心監視和處理安全性事件
事件儀表板可對一段時間下來收集的安全性事件提供事件數的概觀，並列出可能需要注意的事件。  

> [!NOTE]
> 若要使用這項功能，工作區必須執行 Log Analytics 第 2 版，並位於資訊安全中心的標準層。 如需標準層的詳細資訊，請參閱資訊安全中心[價格頁面](security-center-pricing.md)。
>
>

## <a name="what-is-a-security-event"></a>什麼是安全性事件？
資訊安全中心使用 Microsoft Monitoring Agent 收集機器的各種安全性相關組態和事件，並將這些事件儲存在您的工作區。 這類資料的範例包括：作業系統記錄檔 (Windows 事件記錄檔)、執行中的程序，以及與資訊安全中心整合的安全性解決方案之中的事件。 Microsoft Monitoring Agent 也會將損毀傾印檔案複製到工作區。

## <a name="events-processed-dashboard"></a>已處理事件的儀表板
您可從資訊安全中心主功能表或資訊安全中心 [概觀] 刀鋒視窗存取 [事件] 儀表板。  

![已處理事件的儀表板][1]

[資訊安全中心] 下的 [事件] 圖格顯示了從 Azure VM 和非 Azure 電腦流入資訊安全中心的事件數量。

**事件儀表板**可對一段時間下來收集的事件提供事件數量概觀，並列出事件清單。

 ![儀表板][2]

 儀表板的上半部列出過去一週處理所有事件的趨勢。 儀表板的下半部列出需要注意的事件，並依類型列出所有事件：

 - **需要注意的事件**包含資訊安全中心提供的事件查詢，以及您建立並加入的事件查詢。 也可以在儀表板上快速檢視每一項需要注意的事件的數量。
 - **依類型的所有事件**顯示正在接收的事件類型，以及每個類型的數量。 事件類型範例包括 SecurityEvent、CommonSecurityLog、WindowsFirewall 和 W3CIISLog。

> [!NOTE]
> 值得注意的事件包括 [Web 基準評估](https://docs.microsoft.com/azure/operations-management-suite/oms-security-web-baseline-assessment)。 Web 基準評估的目的是尋找可能有弱點的 Web 伺服器設定。

## <a name="view-processed-event-details"></a>檢視已處理事件的詳細資料
1. 在資訊安全中心主功能表下，選取 [事件]。
2. [事件儀表板] 工作區選取器可能會開啟。 如果您只有一個工作區，這個工作區選取器不會出現。 如果您有多個工作區，您需要需要選取工作區，才能檢視其處理的事件詳細資料。 如果您有多個工作區，請從清單中選取工作區。

  ![工作區清單][3]

3. [事件儀表板] 會開啟，顯示所選工作區的事件詳細資料。 您可以檢視需要注意的事件，以及依類型的所有事件。  在此範例中，已選取 [需要注意的事件]。

  ![需要注意的事件][4]

4. 您可以選取事件類型，查詢工作區下的更多資料。 在此範例中，已選取 [SecurityEvent]。

  ![選取事件類型][5]

5. [記錄搜尋] 隨即開啟，列出該事件類型的其他詳細資料。

  ![記錄搜尋][6]

## <a name="add-a-notable-event"></a>新增需要注意的事件
資訊安全中心提供需要注意的最新事件。 您可以使用 [Log Analytics 查詢語言](../log-analytics/log-analytics-search-reference.md)，根據自己的查詢來新增需要注意的事件。 我們會返回 [事件儀表板]，新增需要注意的事件。

1. 選取 [新增需要注意的事件]。

  ![新增需要注意的事件][7]

2. [新增自訂需要注意的事件] 隨即開啟。  在 [顯示名稱] 下，輸入需要注意的事件所用的名稱。 在 [搜尋查詢] 下，輸入事件的查詢。

  ![輸入您的查詢][8]

4. 選取 [確定] 。

## <a name="update-your-workspace-for-events-processing"></a>更新工作區以進行事件處理
工作區必須執行 Log Analytics 第 2 版，並位於資訊安全中心的標準層，才能使用資訊安全中心的事件處理。 [事件儀表板] 工作區選取器會識別不符合這些需求的工作區。

![工作區不符合需求][9]

如果工作區資料列：

- 顯示**需要更新** - 您需要將工作區更新至 Log Analytics 第 2 版
- 顯示**升級計畫** – 您必須將工作區升級至資訊安全中心的標準層
- 為空白 - 工作區符合需求，而且選取工作區會帶您到儀表板

> [!NOTE]
> 在 [事件儀表板] 下，[事件] 資料行會顯示每個工作區中的事件數量。  因為資訊安全中心的免費層會套用於該工作區，所以部分工作區的資料行會呈現空白。 在免費層中，資訊安全中心將收集事件，但是 Log Analytics 不會儲存事件，且儀表板不會提供事件。
>
>

## <a name="update-workspace-to-log-analytics-version-2"></a>將工作區更新至 Log Analytics 第 2 版
1. 選取**需要更新**的工作區。
2. [搜尋升級] 隨即開啟。 選取 [立即升級]。

  ![立即升級][10]

## <a name="upgrade-to-security-centers-standard-tier"></a>升級至資訊安全中心的標準層
1. 選取有**升級計畫**的工作區。
2. [事件儀表板] 隨即開啟。 選取 [試用事件儀表板]。

  ![試用儀表板][11]

3. 在 [上架至進階安全性] 下，選取您要升級的工作區。
4. 在 [價格] 下，選取 [標準]。
5. 選取 [ **儲存**]。

  ![升級至標準層][12]

## <a name="next-steps"></a>後續步驟
在本文中，您學會如何使用資訊安全中心的事件儀表板。 若要深入了解儀表板的運作方式，並撰寫您自己的事件查詢，請參閱：

- [什麼是 Log Analytics？](../log-analytics/log-analytics-overview.md) – Log Analytics 概觀
- [了解 Log Analytics 中的記錄搜尋](../log-analytics/log-analytics-log-search-new.md) - 說明記錄搜尋在 Log Analytics 中的使用方式，並且提供在建立記錄搜尋之前應該了解的概念
- [Log Analytics 搜尋參考](../log-analytics/log-analytics-search-reference.md) – 了解如何在記錄檔中使用查詢語言撰寫您自己的事件查詢

若要深入了解資訊安全中心，請參閱：

- [資訊安全中心概觀](security-center-intro.md) – 說明資訊安全中心的主要功能

<!--Image references-->
[1]: ./media/security-center-events-dashboard/events-processed.png
[2]: ./media/security-center-events-dashboard/dashboard.png
[3]: ./media/security-center-events-dashboard/view-processed-event.png
[4]: ./media/security-center-events-dashboard/notable-event.png
[5]: ./media/security-center-events-dashboard/events-by-type.png
[6]: ./media/security-center-events-dashboard/log-search-detail.png
[7]: ./media/security-center-events-dashboard/add-notable-event.png
[8]: ./media/security-center-events-dashboard/create-query.png
[9]: ./media/security-center-events-dashboard/requires-update.png
[10]: ./media/security-center-events-dashboard/search-upgrade.png
[11]: ./media/security-center-events-dashboard/try-dashboard.png
[12]: ./media/security-center-events-dashboard/onboard-workspace.png
