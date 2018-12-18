---
title: 包含檔案
description: 包含檔案
services: machine-learning
author: sdgilley
ms.service: machine-learning
ms.author: sgilley
manager: cgronlund
ms.custom: include file
ms.topic: include
ms.date: 09/24/2018
ms.openlocfilehash: 6d6e6a2aea867c5b603d950a4bbaa33f14695f45
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47010900"
---
使用您將使用之 Azure 訂用帳戶的認證來登入 [Azure 入口網站](https://portal.azure.com/)。 如果您沒有 Azure 訂用帳戶，請立即建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

只有在 Edge、Chrome 與 Firefox 瀏覽器上才支援入口網站的工作區儀表板。

   ![Azure 入口網站](./media/aml-create-in-portal/portal-dashboard.png)

選取入口網站左上角的 [建立資源] 按鈕 (+)。

   ![在 Azure 入口網站中建立資源](./media/aml-create-in-portal/portal-create-a-resource.png)

在搜尋列中輸入「Machine Learning」。 選取名為 **Machine Learning 工作區**的搜尋結果。

   ![搜尋工作](./media/aml-create-in-portal/allservices-search.PNG)

在 [Machine Learning 工作區] 窗格中，捲動至底部並選取 [建立] 以開始。

   ![create](./media/aml-create-in-portal/portal-create-button.png)

在 [ML 工作區] 窗格中，設定您的工作區。

   欄位|說明
   ---|---
   工作區名稱 |輸入可識別您工作區的唯一名稱。  這裡我們使用 docs-ws。 名稱必須是整個資源群組中唯一的。 請使用可輕鬆回想並與其他人所建立之工作區有所區別的名稱。  
   訂用帳戶 |選擇您要使用的 Azure 訂用帳戶。 如果您有多個訂用帳戶，請選擇資源計費的適當訂用帳戶。
   資源群組 | 在您的訂用帳戶中使用現有的資源群組，或輸入名稱來建立新的資源群組。 資源群組是存放 Azure 方案相關資源的容器。  這裡我們使用 docs-aml。 
   位置 | 選擇最接近您的使用者與資料資源的位置。 這是建立工作區的位置。

   ![建立工作區](./media/aml-create-in-portal/workspace-create.png)

選取 [建立] 來開始建立程序。  建立工作區可能需要一些時間。

   若要檢查部署狀態，請選取工具上的通知圖示 (鈴鐺)。

   ![建立工作區](./media/aml-create-in-portal/notifications.png)

   完成時，會出現部署成功的訊息。  它也會出現在通知區段。   按一下 [前往資源] 按鈕以檢視新的工作區。
