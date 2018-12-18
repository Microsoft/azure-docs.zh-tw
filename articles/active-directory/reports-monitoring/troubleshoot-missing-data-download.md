---
title: 疑難排解︰已下載的 Azure Active Directory 活動記錄中的遺漏資料 | Microsoft Docs
description: 提供您已下載的 Azure Active Directory 活動記錄中遺漏資料的解決方案。
services: active-directory
documentationcenter: ''
author: priyamohanram
manager: mtillman
editor: ''
ms.assetid: ffce7eb1-99da-4ea7-9c4d-2322b755c8ce
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.component: report-monitor
ms.date: 01/15/2018
ms.author: priyamo
ms.reviewer: dhanyahk
ms.openlocfilehash: 9138da42eeb87e45b86be10aff67792ee6de09b4
ms.sourcegitcommit: 744747d828e1ab937b0d6df358127fcf6965f8c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "42144016"
---
# <a name="i-cant-find-any-data-in-the-azure-active-directory-activity-logs-i-downloaded"></a>我在下載的 Azure Active Directory 活動記錄中找不到任何資料


## <a name="symptoms"></a>徵兆

我已下載活動記錄 (稽核或登入)，卻沒看到我所選擇時間的所有記錄。 原因為何？ 

 ![報告](./media/troubleshoot-missing-data-download/01.png)
 

## <a name="cause"></a>原因

當您在 Azure 入口網站中下載活動記錄時，我們會將級別限制為 5000 筆記錄，依最新記錄先排序。 

## <a name="resolution"></a>解決方案

您可以利用 [Azure AD 報告 API](concept-reporting-api.md) 在任何指定時間點擷取最多一萬筆記錄。 我們建議的方法是定期執行指令碼，呼叫報告 API 以增量方式來擷取一段時間的記錄 (例如，每天或每週)。

## <a name="next-steps"></a>後續步驟
請參閱 [Azure Active Directory 報告常見問題集](reports-faq.md)。

