---
title: Azure 應用程式閘道中的 Web 應用程式防火牆要求大小限制與排除清單 - Azure 入口網站 | Microsoft Docs
description: 此文章說明如何使用 Azure 入口網站來設定「應用程式閘道」中的 Web 應用程式防火牆要求大小限制與排除清單設定。
documentationcenter: na
services: application-gateway
author: vhorne
manager: jpconnock
ms.service: application-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.custom: ''
ms.workload: infrastructure-services
ms.date: 09/24/2018
ms.author: victorh
ms.openlocfilehash: 995e003422d5a94fe57174dc9733c870e4e003aa
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46965486"
---
# <a name="web-application-firewall-request-size-limits-and-exclusion-lists-public-preview"></a>Web 應用程式防火牆要求大小限制與排除清單 (公開預覽)

Azure 應用程式閘道 Web 應用程式防火牆 (WAF) 提供 Web 應用程式的保護。 此文章說明 WAF 要求大小限制與排除清單設定。

> [!IMPORTANT]
> WAF 要求大小限制與排除清單的設定目前處於公開預覽狀態。 此預覽版是在沒有服務等級協定的情況下提供，不建議用於生產工作負載。 可能不支援特定功能，或可能已經限制功能。 如需詳細資訊，請參閱 [Microsoft Azure 預覽專用的補充使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

## <a name="waf-request-size-limits"></a>WAF 要求大小限制
Web 應用程式可讓使用者設定介於上下限範圍之間的要求大小限制。 下列是兩個可用的大小限制：
- 要求本文大小欄位的上限是以 KB 為單位指定，而且控制整體的要求大小限制，但不包括任何檔案上傳。 此欄位值的範圍可從最小 1 KB 到最大 128 KB。 要求本文大小的預設值為 128 KB。
- 檔案上傳限制欄位是以 MB 為單位指定，而且它會控管允許的檔案上傳大小上限。 這個欄位的最小值為 1MB，最大值為 500 MB。 檔案上傳限制的預設值為 100 MB。

WAF 也可提供可設定的旋鈕，以便開啟或關閉要求本文檢查。 根據預設，要求本文檢查是啟用的。 如果關閉要求本文檢查，WAF 就不會評估 HTTP 訊息本文的內容。 在此情況下，WAF 會繼續針對標頭、Cookie 與 URI 強制執行 WAF 規則。 如果要求本文檢查關閉，則最大要求本文大小欄位就不適用，而且也無法設定。 關閉要求本文檢查可讓要傳送給 WAF 的訊息大於 128 KB。 不過，訊息本文並未經過檢查，而可能會有漏洞。

## <a name="waf-exclusion-lists"></a>WAF 排除清單

WAF 排除清單可讓使用者略過 WAF 評估的特定要求屬性。 常見範例是用於驗證或密碼欄位的 Active Directory 插入式權杖。 這類屬性較可能包含特殊字元，而會觸發 WAF 規則的誤判。 一旦屬性新增至 WAF 排除清單，該屬性就不會列入任何已設定和作用中 WAF 規則的考慮。 排除清單的範圍是全域的。
您可以將要求標頭、要求 Cookie，或要求查詢字串引數新增至 WAF 排除清單。 您可以指定要精確比對要求標頭、Cookie 或查詢字串屬性，或者也可以選擇指定部分相符即可。 以下是支援的比對條件運算子： 
- **等於**：此運算子適用於精確比對。 例如，若要選取名為 **bearerToken**”** 的標頭，請使用等於運算子搭配設定為 **bearerToken** 的選取器。 
- **開頭為**：此運算子會比對開頭與指定之選取器值相符的所有欄位。 
- **結尾為**：此運算子會比對結尾與指定之選取器值相符的所有欄位。 
- **包含**：此運算子會比對包含與指定之選取器值相符的所有欄位。

在所有情況下，比對都不會區分大小寫，且不允許使用規則運算式作為選取器。

## <a name="next-steps"></a>後續步驟

在您設定 WAF 設定之後，您可以了解如何檢視 WAF 記錄。 如需詳細資訊，請參閱[應用程式閘道診斷](application-gateway-diagnostics.md#diagnostic-logging)。