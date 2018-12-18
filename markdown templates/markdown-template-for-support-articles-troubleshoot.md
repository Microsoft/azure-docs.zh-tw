---
title: 後面接續的 42 個字元 | Microsoft Docs
description: 在搜尋引擎中顯示於標題下方。 您在這裡有更多空間，請使用比標題更多的關鍵字和更具描述性的說明
services: service-name
documentationcenter: dev-center-name
author: GitHub-alias-of-only-one-author
manager: manager-alias
editor: ''
tags: top-support-issue
ms.service: required
ms.devlang: may be required
ms.topic: article
ms.tgt_pltfrm: may be required
ms.workload: required
ms.date: mm/dd/yyyy
ms.author: Your MSFT alias or your full email address;semicolon separates two or more
ms.openlocfilehash: e791ff0377137557b6e9f38a8e1b6b35be948ab6
ms.sourcegitcommit: e19f6a1709b0fe0f898386118fbef858d430e19d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/16/2018
ms.locfileid: "26730549"
---
# <a name="title-maximum-120-characters-target-the-primary-keyword"></a>標題 (最多 120 個字元，目標為主要關鍵字)
*在描述中使用 2-3 個次要關鍵字。*

*根據您的案例，選取下列其中一個免責聲明。如果您的文章與部署模型無關，請忽略此錯誤。*

[!INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-rm-include.md)] 傳統部署模型。

[!INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-classic-include.md)]

[!INCLUDE [learn-about-deployment-models](../../learn-about-deployment-models-both-include.md)]

[開頭段落]

* 簡短描述本文可協助進行疑難排解的特定問題，以及常見的根本原因。
* 開頭段落適合使用與標題中不同的關鍵字，但請勿過於冗長。句子應該通順，且可以輕鬆了解。
* 例外狀況 (選用) - 列出本文中未涵蓋的相關案例。例如，「本文中未涵蓋 Linux/OSS 案例」。

這些 {錯誤}|{問題} 是因為 {非常廣泛的原因} 而發生。

以下是開頭段落的範例。

當您嘗試連線到 Azure SQL Database 時，會遇到的常見連線錯誤為：

* 使用者登入失敗。密碼變更失敗。
* 密碼驗證失敗。
* 無法授權存取指定的訂用帳戶。

因為您沒有存取資料來源的權限，所以會發生這些錯誤。

如果是關於計費主題的文章，請包括下列附註 (下列附註與本文底部的附註稍有不同)：

> [!NOTE]
> 如果您對於本文任何內容有需要進一步的協助，請 [連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) 以快速解決您的問題。
> 
> 

如果「不是」計費的文章，請包括下列參考：
[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="troubleshooting-guidance-optional"></a>疑難排解指引 (選用)
* 若本指南適用範圍太廣，請使用此區段。
* 請勿深入探討。將它保留作為高階指引。

以下是疑難排解指引的範例。

一般而言，只要錯誤不是表示「不支援所要求的虛擬機器大小」，您一律都可以稍後再重試，因為到時叢集可能釋放足夠的資源來滿足您的要求。如果問題是在於不支援要求的虛擬機器大小，請嘗試不同的虛擬機器大小。否則，唯一的選項就是移除釘選的條件約束。

## <a name="troubleshooting-steps"></a>疑難排解步驟
請依使用性和簡易性順序列出解決方案，也就是將最有效且實用的解決方案放在第一位。

*選取適用於您情況的其中一個版本。*

| <em>版本 1：您的文章與部署模型無關</em> | <em>版本 2：Resource Manager 和傳統大致相同的步驟</em> | <em>版本 3：Resource Manager 和傳統大致不同的步驟。 <br />在此情況下，使用 <a href="https://github.com/Azure/azure-content-pr/blob/master/contributor-guide/custom-markdown-extensions.md#simple-selectors">GitHub 中的簡單選取器技術</a>。 <br />請注意：ARM 的 VM 文件為例外狀況，且不應該使用 ARM/傳統選取器。</em> |
|:--- |:--- |:--- |
| <p><h3>[問題 1] \ |[錯誤 1]</h3><h4>原因</h4>[原因詳細資料]</p><p><h4>解決方案 1</h4><em>(最簡單且最有效)</em></p><ol><li>[步驟 1]</li><li>[步驟 2]</li></ol><p><h4>解決方案 2</h4><em>(較不簡單或有效)</em></p><ol><li>[步驟 1]</li><li>[步驟 2]</li></ol><p><h3>[問題 2] \ |[錯誤 2]</h3><h4>原因</h4>[原因詳細資料]</p><p><h4>解決方案 1</h4><em>(最簡單且最有效)</em></p><ol><li>[步驟 1]</li><li>[步驟 2]</li></ol><p><h4>解決方案 2</h4><em>(最簡單且最有效)</em></p><ol><li>[步驟 1]</li><li>[步驟 2]</li></ol><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /> |

## <a name="next-steps"></a>後續步驟
*如果使用者需採取 1-3 個具體、高度相關的後續步驟，請包含此區段。如果沒有任何後續步驟，請加以刪除。此處不適合連結清單。如果包含後續步驟的連結，請務必包含文字，解釋後續步驟為何相關/重要。*

如果是關於計費主題的文章，請包括下列附註 (下列附註與本文開頭的附註稍有不同)：

> [!NOTE]
> 如果您仍有其他問題，請 [連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) 以快速解決您的問題。
> 
> 

