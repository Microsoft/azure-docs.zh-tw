---
title: 在邏輯應用程式中新增延遲 | Microsoft Docs
description: 延遲和延遲直到動作的概觀，以及如何搭配 Azure 邏輯應用程式使用它們。
services: ''
documentationcenter: ''
author: jeffhollan
manager: erikre
editor: ''
tags: connectors
ms.assetid: 915f48bf-3bd8-4656-be73-91a941d0afcd
ms.service: logic-apps
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/18/2016
ms.author: jehollan
ms.openlocfilehash: 6cde5b8ba8d770a07199816286b666e952394de1
ms.sourcegitcommit: be9a42d7b321304d9a33786ed8e2b9b972a5977e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/19/2018
ms.locfileid: "27963667"
---
# <a name="get-started-with-the-delay-and-delay-until-actions"></a>開始使用延遲和延遲直到動作
透過延遲和「延遲直到」動作，您可以完成工作流程案例。

例如，您可以：

* 等到工作日再透過電子郵件傳送狀態更新。
* 延遲工作流程，直到 HTTP 呼叫有時間來完成，才繼續進行並擷取結果。

若要開始在邏輯應用程式中使用延遲動作，請參閱 [建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。

## <a name="use-the-delay-actions"></a>使用延遲動作
動作是由邏輯應用程式中定義的工作流程所執行的作業。 [深入了解動作](connectors-overview.md)。

以下是如何在邏輯應用程式中使用延遲步驟的範例順序︰

1. 在新增觸發程序之後，按一下 [新增步驟]  以新增動作。
2. 搜尋「延遲」  以顯示延遲動作。 在此範例中，我們將會選取 [延遲] 。
   
    ![延遲動作](./media/connectors-native-delay/using-action-1.png)
3. 完成任何動作屬性以設定延遲。
   
    ![延遲設定](./media/connectors-native-delay/using-action-2.png)
4. 按一下 [儲存]  來發佈及啟動邏輯應用程式。

## <a name="action-details"></a>動作詳細資料
循環觸發程序具有下列可設定的屬性。

### <a name="delay-action"></a>延遲動作
此動作會讓執行延遲一段時間間隔。
標示 * 代表必要欄位。

| 顯示名稱 | 屬性名稱 | 說明 |
| --- | --- | --- |
| 計數 * |count |要延遲的時間單位數 |
| 單位 * |unit |時間單位：`Second`、`Minute`、`Hour` 或 `Day` |

<br>

### <a name="delay-until-action"></a>延遲直到動作
此動作會讓執行延遲到指定的日期/時間。
標示 * 代表必要欄位。

| 顯示名稱 | 屬性名稱 | 說明 |
| --- | --- | --- |
| 年 * |timestamp |要延遲到的年度 (GMT) |
| 月 * |timestamp |要延遲到的月份 (GMT) |
| 日 * |timestamp |要延遲到的日期 (GMT) |

<br>

## <a name="next-steps"></a>後續步驟
立即試用平台和 [建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。 您可以查看我們的 [API 清單](apis-list.md)，以探索邏輯應用程式中其他可用的連接器。

