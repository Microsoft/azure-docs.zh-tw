---
title: LUIS 應用程式的應用程式儀表板 | Microsoft Docs
description: 了解應用程式儀表板，這個視覺化報告工具可讓您以速覽方式監視您的應用程式。
services: cognitive-services
author: diberry
manager: cjgronlund
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 03/01/2018
ms.author: diberry
ms.openlocfilehash: 518227d9f4165a08fafefa357de255d97c710f61
ms.sourcegitcommit: 44fa77f66fb68e084d7175a3f07d269dcc04016f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/24/2018
ms.locfileid: "39224495"
---
# <a name="application-dashboard"></a>應用程式儀表板
應用程式儀表板可讓您以速覽方式監視您的應用程式。 [儀表板] 會在您開啟應用程式時顯示，做法是在 [我的應用程式] 頁面上按一下應用程式名稱，然後從頂端面板中選取 [儀表板]。 

> [!CAUTION]
> 如果您想要 LUIS 的最新計量，您必須：
> * 使用在 Azure 中建立的 LUIS [端點金鑰](luis-how-to-azure-subscription.md)
> * 對所有端點要求 (包括 LUIS [API](https://aka.ms/luis-endpoint-apis) 和 Bot) 使用 LUIS 端點金鑰
> * 對每個 LUIS 應用程式使用不同的端點金鑰。 請勿對所有應用程式使用單一端點金鑰。 端點金鑰是在金鑰層級，而不是在應用程式層級進行追蹤。  

[儀表板] 頁面會顯示 LUIS 應用程式的概觀，包括目前模型狀態以及一段時間的[端點](luis-glossary.md#endpoint)使用量。 <!--The following image shows the **Dashboard** page.-->

<!-- TBD: Get a working screen shot
![The Dashboard](./media/luis-how-to-use-dashboard/dashboard.png)
-->

<!-- TBD: IS THIS STILL TRUE?
At the top of the **Dashboard** page, a contextual notification bar constantly displays notifications to update you on the required or recommended actions appropriate for the current state of your app. It also provides useful tips and alerts as needed. A detailed description of the data reported on the **Dashboard** page follows.
-->
  
## <a name="app-status"></a>應用程式狀態
儀表板會顯示應用程式的訓練和發佈狀態，包括最後訓練並發佈應用程式的日期和時間。  

![儀表板 - 應用程式狀態](./media/luis-how-to-use-dashboard/app-state.png)

## <a name="model-data-statistics"></a>模型資料統計資料
儀表板會顯示應用程式中現有的意圖、實體及標示的語句總數。 

![應用程式資料統計資料](./media/luis-how-to-use-dashboard/app-model-count.png)

## <a name="endpoint-hits"></a>端點叫用
儀表板會顯示 LUIS 應用程式接收的端點叫用總數，讓您顯示所指定期間內的叫用。 顯示的叫用總數是以下兩者的總和：使用[端點金鑰](./luis-concept-keys.md#endpoint-key)的端點叫用以及使用[撰寫金鑰](./luis-concept-keys.md#authoring-key)的端點叫用。

<!-- TBD: this image is old but I don't have a new one based on usage -->
![端點叫用](./media/luis-how-to-use-dashboard/dashboard-endpointhits.png)

> [!NOTE] 
> 最新的端點叫用次數位於 Azure 入口網站的 LUIS 服務概觀上。 
 
### <a name="total-endpoint-hits"></a>端點叫用總數
從應用程式建立到目前日期為止，您的應用程式所收到的端點叫用總數。

### <a name="endpoint-hits-per-period"></a>每個期間的端點叫用數
在過去一段時間內收到的叫用數目 (依照日期顯示)。 開始與結束日期之間的點代表落在這段期間內的天數。 將滑鼠指標暫留在每個點上，以查看期間內每一天的叫用次數。 

若要選取要在圖表上檢視的期間：
 
1. 按一下 [其他設定] ![[其他設定] 按鈕](./media/luis-how-to-use-dashboard/Dashboard-Settings-btn.png) 以存取期間清單。 您可以選取從一週到一年的期間。 

    ![每個期間的端點叫用數](./media/luis-how-to-use-dashboard/timerange.png)

2. 從清單中選取一個期間，然後按一下返回箭頭 ![返回箭頭](./media/luis-how-to-use-dashboard/Dashboard-backArrow.png) 以顯示圖表。

### <a name="key-usage"></a>金鑰使用量
從應用程式的端點金鑰取用的叫用次數。 如需端點金鑰的詳細資訊，請參閱 [LUIS 中的金鑰](luis-concept-keys.md)。 
  
## <a name="intent-breakdown"></a>意圖分解
[意圖分解] 會根據標示的語句或端點叫用來顯示意圖分解。 此摘要圖表會顯示應用程式中每個意圖的相對重要性。 當您將滑鼠指標暫留在配量時，會看到意圖名稱以及它在標示的語句/端點叫用總數中所代表的百分比。 

![意圖分解](./media/luis-how-to-use-dashboard/intent-breakdown.png)

若要控制分解是以標示的語句或端點叫用為基礎：

1. 按一下 [其他設定] ![[其他設定] 按鈕](./media/luis-how-to-use-dashboard/Dashboard-Settings-btn.png) 以存取如下圖所示的清單：

    ![意圖分解清單](./media/luis-how-to-use-dashboard/intent-breakdown-based-on.png)
2. 從清單中選取一個值，然後按一下返回箭頭 ![返回箭頭](./media/luis-how-to-use-dashboard/Dashboard-backArrow.png) 以顯示圖表。

## <a name="entity-breakdown"></a>實體分解
此儀表版會根據標示的語句或端點叫用來顯示實體分解。 此摘要圖表會顯示應用程式中每個實體的相對重要性。 當您將滑鼠指標暫留在配量時，會看到實體名稱以及在標示的語句/端點叫用中所佔的百分比。 

![實體分解](./media/luis-how-to-use-dashboard/entity-breakdown.png)

若要控制分解是以標示的語句或端點叫用為基礎：

1. 按一下 [其他設定] ![[其他設定] 按鈕](./media/luis-how-to-use-dashboard/Dashboard-Settings-btn.png) 以存取如下圖所示的清單：

    ![實體分解清單](./media/luis-how-to-use-dashboard/entity-breakdown-based-on.png)
2. 從清單中選取一個值，然後按一下返回箭頭 ![返回箭頭](./media/luis-how-to-use-dashboard/Dashboard-backArrow.png) 據以顯示圖表。
