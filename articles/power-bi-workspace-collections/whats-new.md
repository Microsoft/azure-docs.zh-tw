---
title: Power BI 工作區集合的新功能
description: 取得 Power BI 工作區集合新功能的最新資訊
services: power-bi-embedded
author: markingmyname
ROBOTS: NOINDEX
ms.assetid: 2794ae98-b9a7-45df-b6e1-962a395b91fa
ms.service: power-bi-embedded
ms.topic: article
ms.workload: powerbi
ms.date: 09/20/2017
ms.author: maghan
ms.openlocfilehash: 0fe3116934ac4ab9b1e3fd3b5efcd6627df0ed3d
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43046924"
---
# <a name="whats-new-in-power-bi-workspace-collections"></a>Power BI 工作區集合的新功能

**Power BI 工作區集合**的更新會定期發行。 但是，並非所有發行版本都會包含供使用者使用的新功能，某些版本著重在後端服務功能。 我們將在這裡強調說明與使用者互動的新功能。

> [!IMPORTANT]
> Power BI 工作區集合已被取代，只能使用到 2018 年 6 月或您的合約所指出的時間。 建議您進行規劃以移轉至 Power BI Embedded，以免應用程式發生中斷。 如需如何將資料移轉至 Power BI Embedded 的資訊，請參閱[如何將 Power BI 工作區集合的內容移轉至 Power BI Embedded](https://powerbi.microsoft.com/documentation/powerbi-developer-migrate-from-powerbi-embedded/)。

## <a name="march-2017"></a>2017 年 3 月

**功能**

* [建立新報告](create-report-from-dataset.md)
* [報告另存新檔](save-reports.md)
* 內嵌「讀取/編輯/建立新的」模式的報告 
* [在編輯/讀取模式之間切換報告](toggle-mode.md)

**使用 REST API 的資料連線能力**

* [建立資料集](https://msdn.microsoft.com/library/azure/mt778875.aspx)
* 推送資料 

**管理 API**

* 複製報告和資料集
* 將報告繫結至不同資料集

**範例**

* [JavaScript 報告內嵌範例](https://microsoft.github.io/PowerBI-JavaScript/demo)已更新

## <a name="december-2016"></a>2016 年 12 月

* [新的 JavaScript 內嵌範例](https://microsoft.github.io/PowerBI-JavaScript/demo/)

## <a name="october-2016"></a>2016 年 10 月

* [使用 Power BI 工作區集合和 R 的進階分析](https://powerbi.microsoft.com/blog/r-in-pbie/) \(英文\)

## <a name="august-31-2016"></a>2016 年 8 月 31 日
這個版本中包含︰

* 支援 [進階篩選和頁面瀏覽](interact-with-reports.md)的全新 JavaScript SDK。
* 加拿大中部資料中心內現已支援 Power BI 工作區集合。 查看 [資料中心狀態](https://azure.microsoft.com/status/)。

## <a name="july-11-2016"></a>2016 年 7 月 11 日
這個版本中包含︰

* **好消息！** Power BI 工作區集合服務不再是預覽，現在已 GA (正式推出)。  
* 所有 REST API 已從 **/beta** 移至 **/v1.0**。
* .NET 和 JavaScript SDK 已針對 **v1.0**更新。
* 現在可以直接使用 API 金鑰來驗證 power BI API 呼叫。 只有內嵌需要應用程式權杖。 在這個過程中，佈建和開發權杖已在 v1.0 API 中被取代，但是它們將繼續在 Beta 版中運作，直到 2016 年 12 月 30 日為止。 若要深入了解，請參閱[使用 Power BI 工作區集合驗證和授權](app-token-flow.md)。
* 應用程式權杖和內嵌報告的資料列層級安全性 (RLS) 支援。 若要深入了解，請參閱 [Power BI 工作區集合的資料列層級安全性](row-level-security.md)。
* 已針對所有 **v1.0** API 呼叫，更新範例應用程式。
* 適用於 Azure SDK、PowerShell 和 CLI 的 Power BI 工作區集合支援。
* 使用者可以將視覺效果資料匯出到 **.csv**。
* Power BI 工作區集合現在受到與 Microsoft Azure 相同的所有語言/地區設定的支援。 若要深入了解，請參閱 [Azure - 語言](http://social.technet.microsoft.com/wiki/contents/articles/4234.windows-azure-extent-of-localization.aspx)。

