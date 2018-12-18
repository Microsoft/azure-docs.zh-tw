---
title: 來自 Verizon 的自訂報告 | Microsoft Docs
description: 您可以使用下列報告檢視 CDN 的使用模式：頻寬、傳輸的資料、點擊、快取狀態、快取點擊率、已傳輸的 IPV4/IPV6 資料。
services: cdn
documentationcenter: ''
author: mdgattuso
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/11/2017
ms.author: magattus
ms.openlocfilehash: 75a1fb4beb4025dc3ec63d29944331d07be35d8a
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49092061"
---
# <a name="custom-reports-from-verizon"></a>來自 Verizon 的自訂報告

[!INCLUDE [cdn-verizon-only](../../includes/cdn-verizon-only.md)]

透過管理入口網站使用 Verizon 設定檔的 Verizon 自訂報告，您可以定義要針對 Edge CNAME 報告收集的資料類型。


## <a name="accessing-verizon-custom-reports"></a>存取 Verizon 自訂報告
1. 在 [CDN 設定檔] 刀鋒視窗中，按一下 [管理]  按鈕。
   
    ![CDN 設定檔 [管理] 按鈕](./media/cdn-reports/cdn-manage-btn.png)
   
    隨即開啟 CDN 管理入口網站。
2. 將滑鼠移至 [分析] 索引標籤上，然後將滑鼠移至 [自訂報告] 飛出視窗上。 按一下 [Edge CNAMEs]。
   
    ![CDN 管理入口網站 - 自訂報告功能表](./media/cdn-reports/cdn-custom-reports.png)

## <a name="edge-cnames-custom-report"></a>Edge CNAMES 自訂報告
Edge CNAMES 自訂報告提供已啟用自訂報告記錄之 Edge CNAMEs 的命中數和已傳輸資料統計資料。 Edge CNAMEs 是由 Azure CDN 端點主機名稱和任何相關聯的自訂網域主機名稱所組成。 

自訂報告資料記錄會在您啟用 Edge CNAME 的自訂報告功能一小時之後開始。 您可以產生特定平台或所有平台的 Edge CNAMEs 報告，來檢視報告資料。 此報告的涵蓋範圍僅限於在指定時段內收集到自訂報告資料的 Edge CNAMEs。 Edge CNAMEs 報告根據 [計量] 選項中所定義的計量，包含前 10 名 Edge CNAMEs 的圖形和資料表格。 

請定義下列的報告選項產生自訂報表：

- 計量：支援下列選項：

   - 命中：指出已導向至啟用自訂報告功能之 Edge CNAME 的要求總數。 此計量不包含傳回給用戶端的狀態碼。

   - 已傳輸資料：指出針對導向至已啟用自訂報告功能之 Edge CNAME 的要求，從 Edge Server 傳輸至 HTTP 用戶端 (例如，網頁瀏覽器) 的資料總量。 已傳輸的資料量是將 HTTP 回應標頭與回應本文相加來計算。 因此，每個資產已傳輸的資料量會大於實際的檔案大小。

- 群組：決定要在橫條圖下方顯示的統計資料類型。 支援下列選項：

   - HTTP 回應碼：依傳回給用戶端的 HTTP 回應碼 (例如 200、403 等等) 組織統計資料。 

   - 快取狀態：依快取狀態組織統計資料。


若要設定報告的日期範圍，您可以從下拉式清單選取預先定義的日期範圍，例如 [今天] 或 [本週]，或者可以選取 [自訂] 並按一下日曆圖示，手動輸入日期範圍。 

選取日期範圍之後，按一下 [執行] 以產生報告。

您可以按一下 [執行] 按鈕右邊的 Excel 符號，以 Excel 格式匯出資料。

![CNAMEs 報告](./media/cdn-reports/cdn-cnames-report.png)

## <a name="edge-cnames-custom-report-fields"></a>Edge CNAMES 自訂報告欄位

| 欄位                     | 說明   |
|---------------------------|---------------|
| 2xx                       | 指出產生 2xx HTTP 狀態碼 (例如 200 OK) 之 Edge CNAME 的要求總數或已傳輸資料 (MB)。 |
| 3xx                       | 指出產生 3xx HTTP 狀態碼 (例如 302 已找到或 304 未修改) 之 Edge CNAME 的要求總數或已傳輸資料 (MB)。 |
| 4xx                       | 指出產生 4xx HTTP 狀態碼 (例如 400 錯誤的要求、403 禁止或 404 找不到) 之 Edge CNAME 的要求總數或已傳輸資料 (MB)。 |
| 5xx                       | 指出產生 5xx HTTP 狀態碼 (例如 500 內部伺服器或或 502 錯誤的閘道) 之 Edge CNAME 的要求總數或已傳輸資料 (MB)。 |
| 快取名中百分比               | 指出從快取直接為要求者服務的可快取要求百分比。 |
| 快取點擊                | 指出產生快取命中 (TCP_EXPIRED_HIT、TCP_HIT 或 TCP_PARTIAL_HIT) 之 Edge CNAME 的要求總數或已傳輸資料 (MB)。 當找到所要求內容的快取版本時，就會發生快取命中。 |
| 已傳輸資料 (MB)     | 指出 Edge CNAME 從 Edge Server 傳輸到 HTTP 用戶端 (網頁瀏覽器) 的資料總量 (MB)。 已傳輸的資料量是將 HTTP 回應標頭與回應本文相加來計算。 因此，每個資產已傳輸的資料量會大於實際的檔案大小。 |
| 說明               | 依主機名稱識別 Edge CNAME |
| 點擊                      | 指出 Edge CNAME 的要求總數 |
| 遺漏                    | 指出產生快取遺漏 (TCP_CLIENT_REFRESH_MISS、TCP_EXPIRED_MISS 或 TCP_MISS) 之 Edge CNAME 的要求總數或已傳輸資料 (MB)。 當接受要求的 Edge Server 上未快取要求的內容時，就會發生快取遺漏。 | 
| 無快取                  | 指出產生 CONFIG_NOCACHE 快取狀態碼之 Edge CNAME 的要求總數或已傳輸資料 (MB)。  |
| 其他                     | 指出 Edge CNAME 中產生 2xx - 5xx 範圍以外之 HTTP 狀態碼的要求總數或已傳輸資料 (MB)。 |
| 平台                  | 指出處理 Edge CNAME 流量的平台。 |
| 未指定               | 指出未記錄快取狀態碼或 HTTP 狀態碼資訊之 Edge CNAME 的要求總數或已傳輸資料 (MB)。  |
| 無法快取               | 指出產生 UNCACHEABLE 快取狀態碼之 Edge CNAME 的要求總數或已傳輸資料 (MB)。  |


## <a name="considerations"></a>考量
只能產生過去 18 個月內的報告。

