---
title: 在 Azure Log Analytics 中建立自訂的儀表板 | Microsoft Docs
description: 本指南可幫助您了解 Log Analytics 儀表板如何以視覺效果顯示所有儲存的記錄搜尋，讓您以單一方式檢視您的環境。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: abb07f6c-b356-4f15-85f5-60e4415d0ba2
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/08/2017
ms.author: magoedte
ms.component: na
ms.openlocfilehash: 7b17b23d30b15fe71dcf031ac67f0c54fc3cfe3e
ms.sourcegitcommit: 5892c4e1fe65282929230abadf617c0be8953fd9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37129307"
---
# <a name="create-a-custom-dashboard-for-use-in-log-analytics"></a>建立用於 Log Analytics 中的自訂儀表板

本指南可幫助您了解 Log Analytics 儀表板如何以視覺效果顯示所有儲存的記錄搜尋，讓您以單一方式檢視您的環境。

>[!NOTE]
> 您無法再編輯現有的 [我的儀表板]。 這項功能正逐漸被取代。

![範例儀表板](./media/log-analytics-dashboards/oms-dashboards-example-dash.png)

您在 OMS 入口網站中建立的所有自訂儀表板，也都可在 OMS 行動應用程式中使用。 如需應用程式的詳細資訊，請參閱下列頁面。

* [Microsoft Store 中的 OMS 行動應用程式](http://www.windowsphone.com/store/app/operational-insights/4823b935-83ce-466c-82bb-bd0a3f58d865)
* [Apple iTunes 中的 OMS 行動應用程式](https://itunes.apple.com/app/microsoft-operations-management/id1042424859?mt=8)

![行動儀表板](./media/log-analytics-dashboards/oms-search-mobile.png)

## <a name="how-do-i-create-my-dashboard"></a>如何建立儀表板？
若要開始，請移至 OMS [概觀] 頁面。 您會在左側看到 [我的儀表板] 圖格。 按一下磚，可向下鑽研儀表板。

![概觀](./media/log-analytics-dashboards/oms-dashboards-overview.png)

## <a name="adding-a-tile"></a>新增磚
在儀表板中，磚是由您儲存的記錄搜尋所形成。 OMS 內建許多預先儲存的記錄搜尋，讓您可以隨時開始。 使用下列步驟來大致了解如何開始。

在 [我的儀表板] 檢視中，直接按一下 [自訂] 進入自訂模式。

![圖示](./media/log-analytics-dashboards/oms-dashboards-pictorial01.png)

 在頁面右側開啟的面板會顯示工作區所有儲存的記錄搜尋。 若要以圖格來呈現已儲存的記錄搜尋，請將滑鼠停留在已儲存的搜尋，然後按一下**加號**符號。

![新增磚 1](./media/log-analytics-dashboards/oms-dashboards-pictorial02.png)

當您按一下**加號**符號時，[我的儀表板] 檢視中會出現新的圖格。

![新增磚 2](./media/log-analytics-dashboards/oms-dashboards-pictorial03.png)

## <a name="edit-a-tile"></a>編輯磚
在 [我的儀表板] 檢視中，直接按一下 [自訂] 進入自訂模式。 按一下您想要編輯的磚。 右側面板即會變更為可編輯狀態，並提供一組選項：

![編輯磚](./media/log-analytics-dashboards/oms-dashboards-pictorial04.png)

![編輯磚](./media/log-analytics-dashboards/oms-dashboards-pictorial05.png)

### <a name="tile-visualizations"></a>磚視覺效果
有三種圖格視覺效果可供您選擇：

| 圖表類型 | 作用 |
| --- | --- |
| ![長條圖](./media/log-analytics-dashboards/oms-dashboards-bar-chart.png) |將已儲存之記錄搜尋結果的時間軸顯示為長條圖，或是依欄位顯示結果清單 (根據您的記錄搜尋是否根據欄位彙總結果而定)。 |
| ![計量](./media/log-analytics-dashboards/oms-dashboards-metric.png) |在圖格中以數字顯示記錄搜尋結果總數。 度量磚可讓您設定一個閾值，當達到該閾值時就讓磚發亮顯示。 |
| ![線條](./media/log-analytics-dashboards/oms-dashboards-line.png) |將已儲存之記錄搜尋結果命中和值的時間軸顯示為折線圖。 |

### <a name="threshold"></a>閾值
您可以使用「度量」視覺效果在磚上建立閾值。 請選取磚，在磚上建立閾值。 選擇是否要在值高於或低於選擇的閾值時讓磚發亮顯示，然後在下面設定閾值。

## <a name="organizing-the-dashboard"></a>排列儀表板
若要排列儀表板，請瀏覽到 [我的儀表板] 檢視，然後按一下 [自訂] 進入自訂模式。 按一下並拖曳您想要移動的磚，然後將磚移動到您想要的位置。

![排列儀表板](./media/log-analytics-dashboards/oms-dashboards-organize.png)

## <a name="remove-a-tile"></a>移除磚
若要移除圖格，請瀏覽到 [我的儀表板] 檢視，然後按一下 [自訂] 進入自訂模式。 選取您想要移除的圖格，然後在右側面板上選取 [移除圖格] 。

![移除磚](./media/log-analytics-dashboards/oms-dashboards-remove-tile.png)

## <a name="next-steps"></a>後續步驟
* 在 Log Analytics 中建立[警示](log-analytics-alerts.md)，以產生通知並修復問題。
