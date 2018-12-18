---
title: 智慧偵測 - Azure Application Insights 偵測到的潛在記憶體流失 | Microsoft Docs
description: 使用 Azure Application Insights 監視應用程式的潛在記憶體流失。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: ea2a28ed-4cd9-4006-bd5a-d4c76f4ec20b
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 12/12/2017
ms.author: mbullwin
ms.openlocfilehash: 25d26db3cd11fff7f7e9ba472247a920ecddea33
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47090729"
---
# <a name="memory-leak-detection-preview"></a>記憶體流失偵測 (預覽)

Application Insights 會自動分析應用程式中每個處理序的記憶體耗用量，並警告您潛在的記憶體流失或增加的記憶體耗用量。

這項功能不需要任何特殊的設定，除了為您的應用程式[設定效能計數器](https://docs.microsoft.com/azure/application-insights/app-insights-performance-counters)以外。 當您的應用程式產生足夠的記憶體效能計數器遙測資料 (例如，私用位元組) 時，它就會作用。

## <a name="when-would-i-get-this-type-of-smart-detection-notification"></a>何時會取得這個類型的智慧偵測通知？
在一或多個處理序和/或一或多部機器 (這些是您應用程式的一部分) 中，長時間內記憶體耗用量持續增加會產生典型通知。 我們使用機器學習演算法來偵測符合記憶體流失模式的記憶體耗用量增加情況。

## <a name="does-my-app-really-have-a-problem"></a>我的應用程式真的有問題嗎？
否，通知並不表示您的應用程式一定有問題。 雖然記憶體流失模式通常表示應用程式問題，這些模式對特定的處理序而言可能很典型，或可能有自然的商業理由，而且可以忽略。

## <a name="how-do-i-fix-it"></a>如何修正問題？
通知包括要在診斷分析程序中支援的診斷資訊：
1. **分級。** 通知會顯示記憶體增加數量 (以 GB 為單位)，以及記憶體增加的時間範圍。 這可協助您將優先順序指派給此問題。
2. **範圍。** 有多少部機器呈現記憶體流失模式？ 潛在記憶體流失期間觸發多少例外狀況？ 可以從通知取得這項資訊。
3. **診斷。** 偵測包含記憶體流失模式，顯示一段時間的處理序記憶體耗用量。 您也可以使用連結到支援資訊的相關項目和報表，協助您進一步診斷問題。
