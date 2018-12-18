---
title: 教學課程 - 使用 Power BI Desktop 連線至 Azure Analysis Services | Microsoft Docs
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: tutorial
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: owend
ms.openlocfilehash: 4096b9f77ba841abfcb4f29f2827aefacf22103f
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37446583"
---
# <a name="tutorial-connect-with-power-bi-desktop"></a>教學課程 - 使用 Power BI Desktop 進行連線

在本教學課程中，您會使用 Power BI Desktop 連線至伺服器上的 adventureworks 範例模型資料庫。 您所完成的工作會模擬一般使用者對模型的連線，並從模型資料建立基本報告。

> [!div class="checklist"]
> * 從入口網站取得您的伺服器名稱
> * 使用 Power BI Desktop 進行連線
> * 建立基本報告

## <a name="prerequisites"></a>先決條件

- [新增 adventureworks 範例模型資料庫](../analysis-services-create-sample-model.md)至您的伺服器。
- 具有 adventureworks 範例模型資料庫的[*讀取*](../analysis-services-server-admins.md)權限。
- [安裝最新的 Power BI Desktop](https://powerbi.microsoft.com/desktop)。

## <a name="log-in-to-the-azure-portal"></a>登入 Azure 入口網站
在本教學課程中，您會登入入口網站並僅取得伺服器名稱。 一般而言，使用者會取得伺服器管理員所提供的伺服器名稱。

登入[入口網站](https://portal.azure.com/)。

## <a name="get-server-name"></a>取得伺服器名稱
若要從 Power BI Desktop 連線至您的伺服器，首先您必須要有伺服器名稱。 您可以從入口網站取得伺服器名稱。

在 [Azure 入口網站] > 伺服器 > [概觀]  >  [伺服器名稱] 中，複製伺服器名稱。
   
   ![在 Azure 中取得伺服器名稱](./media/analysis-services-tutorial-pbid/aas-copy-server-name.png)

## <a name="connect-in-power-bi-desktop"></a>在 Power BI Desktop 中連線

1. 在 Power BI Desktop 中，按一下 [取得資料] > [Azure] > [Azure Analysis Services 資料庫]。

   ![取得資料中的連線](./media/analysis-services-tutorial-pbid/aas-pbid-connect-aasserver.png)

2. 在 [伺服器] 中貼上伺服器名稱，然後在 [資料庫] 中輸入 **adventureworks**，然後按一下 [確定]。

   ![指定伺服器名稱和模型資料庫](./media/analysis-services-tutorial-pbid/aas-pbid-connect-aas-servername.png)

3. 出現提示時，請輸入您的登入認證。 您所輸入的帳戶至少必須具有 adventureworks 範例模型資料庫的讀取權限。

    adventureworks 模型會在 Power BI Desktop 中開啟，並在 [報告] 檢視中顯示一個空白報告。 [欄位] 清單會顯示所有非隱藏的模型物件。 連線狀態會顯示在右下角。

4. 在 [視覺效果] 中選取 [群組橫條圖]，然後按一下 [格式] (滾筒刷圖示)，接著開啟 [資料標籤]。 

   ![視覺效果](./media/analysis-services-tutorial-pbid/aas-pbid-visualizations-report.png)

5. 在 [欄位] > [網際網路銷售] 資料表中，選取 [網際網路銷售總計] 和 [盈餘] 量值。 在 [產品類別] 資料表中，選取 [產品類別名稱]。

   ![完成報告](./media/analysis-services-tutorial-pbid/aas-pbid-complete-report.png)

    花幾分鐘建立不同的視覺效果並進行資料和計量的配量，以探索 adventureworks 範例模型。 當您對報告感到滿意時，請務必加以儲存。

## <a name="clean-up-resources"></a>清除資源

不再需要報告時請不要加以儲存，或刪除您先前儲存的檔案。

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何使用 Power BI Desktop 連線至伺服器上的資料模型，並建立基本報告。 如果您不熟悉如何建立資料模型，請參閱 [Adventure Works 網際網路銷售表格式資料模型教學課程](aas-adventure-works-tutorial.md)。