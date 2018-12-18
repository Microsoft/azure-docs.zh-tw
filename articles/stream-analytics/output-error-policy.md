---
title: 在 Azure 串流分析中輸出錯誤原則
description: 了解 Azure 串流分析中可用的輸出錯誤處理原則。
services: stream-analytics
author: sidram
ms.author: sidram
manager: jeanb
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 07/27/2018
ms.openlocfilehash: f854e8ce35ac9d0a99b4a7dc1cdc66724114a5c4
ms.sourcegitcommit: e3d5de6d784eb6a8268bd6d51f10b265e0619e47
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/01/2018
ms.locfileid: "39392565"
---
# <a name="output-error-policy"></a>輸出錯誤原則
本文說明可以在 Azure 串流分析中設定的輸出資料錯誤處理原則。

輸出資料錯誤處理原則僅適用於當串流分析作業所產生的輸出事件不符合目標接收器的結構描述時所發生的資料轉換錯誤。 您可以選擇 [重試] 或 [卸除] 來設定此原則。 當串流分析作業執行時，在 Azure 入口網站的 [設定] 下選取 [錯誤原則]，以進行選取。

![錯誤原則 - 位置](./media/stream-analytics-error-policy/stream-analytics-error-policy-locate.PNG)


## <a name="retry"></a>重試
在錯誤發生時，Azure 串流分析會無限期地重試寫入事件，直到寫入成功為止。 重試不會逾時。 最終所有後續事件都會遭到正在重試的事件封鎖，而無法處理。 此選項是預設輸出錯誤處理原則。

## <a name="drop"></a>卸除
Azure 串流分析會卸除任何導致資料轉換錯誤的輸出事件。 無法復原卸除的事件以供稍後重新處理。


無論輸出錯誤處理原則組態為何，所有暫時性錯誤 (例如網路錯誤) 都會重試。


## <a name="next-steps"></a>後續步驟
[Azure 串流分析的疑難排解指南](stream-analytics-troubleshooting-guide.md)
