---
title: Azure Analysis Services 教學課程第 13 課：部署 | Microsoft Docs
description: 說明如何將教學課程專案部署至 Azure Analysis Services。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 84cdae1694608814167641417781cd22c12656a4
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37443923"
---
# <a name="deploy"></a>部署

在這堂課中，您將會設定部署屬性，包括指定要部署至的 Azure Analysis Services 伺服器，以及模型的名稱。 然後，您會將模型部署至該執行個體。 部署模型之後，使用者就可以利用報告用戶端應用程式來連線至您的模型。 若要深入了解，請參閱 [Deploy to Azure Analysis Services](https://docs.microsoft.com/azure/analysis-services/analysis-services-deploy) (部署至 Azure Analysis Services)。  
  
這堂課的預估完成時間：**5 分鐘**  
  
## <a name="prerequisites"></a>先決條件  
本文是表格式模型教學課程的一部分，請依序完成。 在這堂課中執行工作之前，您必須已完成上一堂課︰[第 12 課︰在 Excel 中進行分析](../tutorials/aas-lesson-12-analyze-in-excel.md)。  

> [!IMPORTANT]  
> 您在遠端 Analysis Services 伺服器上必須具有[系統管理員權限](../analysis-services-server-admins.md)，才能部署到它。  

> [!IMPORTANT]  
> 如果您已將 AdventureWorksDW2014 範例資料庫安裝在內部部署 SQL Server，且想要將模型部署到 Azure Analysis Services 伺服器，則需要[內部部署資料閘道](../analysis-services-gateway.md)。
  
## <a name="deploy-the-model"></a>部署模型  
  
#### <a name="to-configure-deployment-properties"></a>設定部署屬性  

  
1.  在 [方案總管] 中，以滑鼠右鍵按一下 [AW 網際網路銷售] 專案，然後按一下 [屬性]。  
  
2.  在 [AW 網際網路銷售屬性頁] 對話方塊中，請在 [部署伺服器] 下的 [伺服器] 屬性中，輸入完整伺服器名稱。  

    ![aas 第 13 課部署屬性](../tutorials/media/aas-lesson13-deploy-property.png)
  
3.  在 [資料庫] 屬性中，輸入 **Adventure Works 網際網路銷售**。  
  
4.  在 [模型名稱] 屬性中，輸入 **Adventure Works 網際網路銷售模型**。  
  
5.  驗證您的選取項目，然後按一下 [確定]。  
  
#### <a name="to-deploy-the-adventure-works-internet-sales"></a>部署 Adventure Works 網際網路銷售
  
1.  在 [方案總管] 中，以滑鼠右鍵按一下 [AW 網際網路銷售] 專案 > [建置]。  

2.  以滑鼠右鍵按一下 [AW 網際網路銷售] 專案 > [部署]。

    當部署至 Azure Analysis Services 時，可能會提示您輸入您的帳戶。 請輸入您的組織帳戶和密碼，例如 nancy@adventureworks.com。 此帳戶必須在伺服器上的 Admins 中。
  
    [部署] 對話方塊隨即出現，並顯示中繼資料和模型中每個資料表的部署狀態。  
    
    ![aas 第 13 課部署狀態](../tutorials/media/aas-lesson13-deploy-status.png)
  
3. 當部署順利完成時，請繼續並按一下 [關閉]。  
  

這堂課說明從 SSDT 部署表格式模型的最常見且最簡單的方法。 進階部署選項 (例如「部署精靈」或使用 XMLA 和 AMO 達成自動化) 會提供更大的彈性、一致性和已排程的部署。 若要深入了解，請參閱[表格式模型解決方案部署](https://docs.microsoft.com/sql/analysis-services/tabular-models/tabular-model-solution-deployment-ssas-tabular)。

## <a name="conclusion"></a>結論  
恭喜！ 您已完成製作和部署第一個 Analysis Services 表格式模型。 本教學課程協助引導您完成建立表格式模型的常見工作。 現在已部署 Adventure Works 網際網路銷售模型，接下來，您可以使用 SQL Server Management Studio 管理模型，並建立處理指令碼和備份計劃。 使用者現在也可以使用報告用戶端應用程式來連線至模型，例如 Microsoft Excel 或 Power BI。  

![aas 第 13 課 ssms](../tutorials/media/aas-lesson13-ssms.png)
  
  
  
## <a name="whats-next"></a>後續步驟
[使用 Power BI Desktop 進行連線](../analysis-services-connect-pbi.md)   
[補充課程 - 動態安全性](../tutorials/aas-supplemental-lesson-dynamic-security.md)   
[補充課程 - 詳細資料列](../tutorials/aas-supplemental-lesson-detail-rows.md)   
[補充課程 - 不完全階層](../tutorials/aas-supplemental-lesson-ragged-hierarchies.md)   
