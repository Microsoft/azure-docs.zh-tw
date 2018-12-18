---
title: 使用 Visual Studio 管理邏輯應用程式 - Azure Logic Apps | Microsoft Docs
description: 使用 Visual Studio Cloud Explorer 管理邏輯應用程式和其他 Azure 資產
services: logic-apps
ms.service: logic-apps
author: ecfan
ms.author: estfan
manager: jeconnoc
ms.topic: article
ms.custom: mvc
ms.date: 03/15/2018
ms.reviewer: klam, LADocs
ms.suite: integration
ms.openlocfilehash: d4de75238e48b8eb955095b5a3823f2fed799fae
ms.sourcegitcommit: fab878ff9aaf4efb3eaff6b7656184b0bafba13b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2018
ms.locfileid: "42445638"
---
# <a name="manage-logic-apps-with-visual-studio"></a>使用 Visual Studio 管理邏輯應用程式

雖然您可以在 <a href="https://portal.azure.com" target="_blank">Azure 入口網站</a>中建立、編輯、管理和部署邏輯應用程式，但是當您想要將邏輯應用程式新增至原始檔控制、發佈不同的版本，以及針對不同的部署環境建立 [Azure Resource Manager](../azure-resource-manager/resource-group-overview.md) 範本時，您也可以使用 Visual Studio。 使用 Visual Studio Cloud Explorer，您可以尋找並管理您的邏輯應用程式以及其他 Azure 資源。 例如，您可以開啟、下載、編輯、執行、檢視執行歷程記錄、停用和啟用已在 Azure 入口網站部署的邏輯應用程式。 如果您不曾在 Visual Studio 中使用 Azure Logic Apps，請了解[如何使用 Visual Studio 建立邏輯應用程式](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md)。

> [!IMPORTANT]
> 從 Visual Studio 部署或發佈邏輯應用程式會覆寫該應用程式在 Azure 入口網站中的版本。 因此，如果您想要保留在 Azure 入口網站所做的變更，在下一次從 Visual Studio 部署或發佈之前，請確定您已從 Azure 入口網站[重新整理 Visual Studio 中的邏輯應用程式](#refresh)。

<a name="requirements"></a>

## <a name="prerequisites"></a>必要條件

* 如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。

* 如果您還沒有以下這些工具，請加以下載並安裝： 

  * <a href="https://www.visualstudio.com/downloads" target="_blank">Visual Studio 2017 或 Visual Studio 2015 - Community 版本或更高版本</a>。 
  本快速入門使用免費的 Visual Studio Community 2017。

  * <a href="https://azure.microsoft.com/downloads/" target="_blank">Azure SDK (2.9.1 或更新版本)</a> 和 <a href="https://github.com/Azure/azure-powershell#installation" target="_blank">Azure PowerShell</a>

  * <a href="https://marketplace.visualstudio.com/items?itemName=VinaySinghMSFT.AzureLogicAppsToolsforVisualStudio-18551" target="_blank">Azure Logic Apps Tools for Visual Studio 2017</a> 或 <a href="https://marketplace.visualstudio.com/items?itemName=VinaySinghMSFT.AzureLogicAppsToolsforVisualStudio" target="_blank">Visual Studio 2015 版本</a> 
  
    您可以直接從 Visual Studio Marketplace 下載並安裝 Azure Logic Apps Tools，或了解<a href="https://docs.microsoft.com/visualstudio/ide/finding-and-using-visual-studio-extensions" target="_blank">如何從 Visual Studio 內部安裝此擴充功能</a>。 
    請務必在完成安裝之後重新啟動 Visual Studio。

* 使用內嵌的 Logic Apps 設計工具時能夠存取 Web

  設計工具需要網際網路連線才能在 Azure 中建立資源，以及從邏輯應用程式中的連接器讀取屬性和資料。 
  例如，如果您使用 Dynamics CRM Online 連接器，則設計工具會檢查您的 CRM 執行個體，以取得可用的預設和自訂屬性。

<a name="find-logic-apps-vs"></a>

## <a name="find-your-logic-apps"></a>尋找邏輯應用程式

您可以在 Visual Studio 中使用 Cloud Explorer，尋找與您的 Azure 訂用帳戶相關聯、且部署在 Azure 入口網站中的所有邏輯應用程式。

1. 開啟 Visual Studio。 在 [檢視] 功能表上選取 [Cloud Explorer]。

2. 在 Cloud Explorer 中，選擇 [帳戶管理]。 選取與您的邏輯應用程式相關聯的 Azure 訂用帳戶，然後選擇 [套用]。 例如︰

   ![選擇 [帳戶管理]](./media/manage-logic-apps-with-visual-studio/account-management-select-Azure-subscription.png)

2. 根據您要依 [資源群組] 還是 [資源類型] 來搜尋，遵循下列相關步驟：

   * **資源群組**：在您的 Azure 訂用帳戶底下，Cloud Explorer 會顯示與該訂用帳戶相關聯的所有資源群組。 
   展開包含您邏輯應用程式的資源群組，然後選取您的邏輯應用程式。

   * **資源類型**：在您的 Azure 訂用帳戶底下，展開 [邏輯應用程式]。 在 Cloud Explorer 顯示與您訂用帳戶相關聯的所有已部署邏輯應用程式後，選取您的邏輯應用程式。

<a name="open-designer"></a>

## <a name="open-in-visual-studio"></a>在 Visual Studio 中開啟

在 Visual Studio 中，您可以開啟先前直接透過 Azure 入口網站建立或部署的邏輯應用程式，或開啟使用 Visual Studio 以 Azure Resource Manager 專案形式所建立或部署的邏輯應用程式。

1. 開啟 Cloud Explorer，並尋找您的邏輯應用程式。 

2. 在邏輯應用程式的捷徑功能表上，選取 [使用邏輯應用程式編輯器開啟]。

   這個範例會顯示依資源類型所搜尋到的邏輯應用程式，因此您的邏輯應用程式會出現在 [邏輯應用程式] 區段下。

  ![從 Azure 入口網站開啟已部署的邏輯應用程式](./media/manage-logic-apps-with-visual-studio/open-logic-app-in-editor.png)

   邏輯應用程式在 Logic Apps 設計工具中開啟之後，您可以在設計工具的底部選擇 [程式碼檢視]，以便檢閱基礎邏輯應用程式定義結構。 
   如果您想要為邏輯應用程式建立部署範本，請了解針對該邏輯應用程式要[如何下載 Azure Resource Manager 範本](#download-logic-app)。 深入了解 [Resource Manager 範本](../azure-resource-manager/resource-group-overview.md#template-deployment)。

<a name="download-logic-app"></a>

## <a name="download-from-azure"></a>從 Azure 下載

您可以從 <a href="https://portal.azure.com" target="_blank">Azure 入口網站</a>下載邏輯應用程式，並將它們儲存為 [Azure Resource Manager](../azure-resource-manager/resource-group-overview.md) 範本。 之後您可以在本機使用 Visual Studio 編輯範本，並針對不同部署環境自訂邏輯應用程式。 下載邏輯應用程式會自動將其在 [Resource Manager 範本](../azure-resource-manager/resource-group-overview.md#template-deployment)中的定義「參數化」，這也會使用 JavaScript 物件標記法 (JSON)。

1. 在 Visual Studio 中，開啟 Cloud Explorer，然後尋找並選取您想要從 Azure 下載的邏輯應用程式。

2. 在該應用程式的捷徑功能表上，選取 [使用邏輯應用程式編輯器開啟]。

   邏輯應用程式設計工具隨即開啟，並顯示邏輯應用程式。 
   若要檢閱邏輯應用程式的基礎定義和結構，請在設計工具的底部選擇 [程式碼檢視]。 

3. 在設計工具的工具列上，選擇 [下載]。

   ![選擇 [下載]](./media/manage-logic-apps-with-visual-studio/download-logic-app.png)

4. 當系統提示您輸入位置時，瀏覽至該位置，並以 JSON (.json) 檔案格式儲存邏輯應用程式定義的 Resource Manager 範本。 

您的邏輯應用程式定義會出現在 Resource Manager 範本的 `resources` 子區段中。 您現在可以使用 Visual Studio 來編輯邏輯應用程式定義和 Resource Manager 範本。 您也可以將範本新增為 Visual Studio 解決方案中的 Azure Resource Manager 專案。 了解[邏輯應用程式在 Visual Studio 中的 Resource Manager 專案](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md)。 

<a name="refresh"></a>

## <a name="refresh-from-azure"></a>從 Azure 重新整理

如果您在 Azure 入口網站中編輯您的邏輯應用程式，並想要保留這些變更，請務必使用這些變更重新整理該應用程式在 Visual Studio 中的版本。 

* 在 Visual Studio 中，選擇邏輯應用程式設計工具工具列上的 [重新整理]。

  -或-

* 在 Visual Studio Cloud Explorer 中，開啟邏輯應用程式的捷徑功能表，然後選取 [重新整理]。 

![使用更新內容重新整理邏輯應用程式](./media/manage-logic-apps-with-visual-studio/refresh-logic-app.png)

## <a name="publish-logic-app-updates"></a>發佈邏輯應用程式的更新

當您準備好要從 Visual Studio 將您的邏輯應用程式更新部署到 Azure 時，請在邏輯應用程式設計工具的工具列上，選擇 [發佈]。

![發佈更新後的邏輯應用程式](./media/manage-logic-apps-with-visual-studio/publish-logic-app.png)

## <a name="manually-run-your-logic-app"></a>手動執行邏輯應用程式

您可以從 Visual Studio 手動觸發部署在 Azure 中的邏輯應用程式。 在邏輯應用程式設計工具的工具列上，選擇 [執行觸發程序]。

![手動執行邏輯應用程式](./media/manage-logic-apps-with-visual-studio/manually-run-logic-app.png)

## <a name="review-run-history"></a>檢閱執行歷程記錄

若要檢查邏輯應用程式執行的狀態並診斷問題，您可以檢閱這些執行在 Visual Studio 中的詳細資料，例如輸入和輸出。

1. 在 Cloud Explorer 中，開啟邏輯應用程式的捷徑功能表，然後選取 [開啟執行歷程記錄]。

   ![開啟執行歷程記錄](./media/manage-logic-apps-with-visual-studio/view-run-history.png)

2. 若要檢閱特定執行的詳細資料，請對該執行按兩下。 例如︰

   ![詳細的執行歷程記錄](./media/manage-logic-apps-with-visual-studio/view-run-history-details.png)
  
   > [!TIP]
   > 若要依屬性排序資料表，請選擇該屬性的資料行標頭。 

3. 展開您要檢閱其輸入和輸出的步驟。 例如︰

   ![檢視每個步驟的輸入和輸出](./media/manage-logic-apps-with-visual-studio/run-inputs-outputs.png)

## <a name="disable-or-enable-logic-app"></a>停用或啟用邏輯應用程式

您可以在下一次觸發程序條件符合時停止引發觸發程序，而無需刪除您的邏輯應用程式。 停用邏輯應用程式可防止 Logic Apps 引擎建立和執行邏輯應用程式未來的工作流程執行個體。
在 Cloud Explorer 中，開啟邏輯應用程式的捷徑功能表，然後選取 [停用]。

![停用邏輯應用程式](./media/manage-logic-apps-with-visual-studio/disable-logic-app.png)

> [!NOTE]
> 當您停用邏輯應用程式後，就不會具現化新的執行。 所有進行中和擱置的執行將會繼續直到完成，這可能需要一些時間。 

當您準備好讓邏輯應用程式繼續作業時，可以重新啟動邏輯應用程式。 在 Cloud Explorer 中，開啟邏輯應用程式的捷徑功能表，然後選取 [啟用]。

![啟用邏輯應用程式](./media/manage-logic-apps-with-visual-studio/enable-logic-app.png)

## <a name="delete-your-logic-app"></a>刪除邏輯應用程式

若要從 Azure 入口網站刪除您的邏輯應用程式，請在 Cloud Explorer 中開啟邏輯應用程式的捷徑功能表，然後選取 [刪除]。

![刪除邏輯應用程式](./media/manage-logic-apps-with-visual-studio/delete-logic-app.png)

> [!NOTE]
> 當您刪除邏輯應用程式後，就不會具現化新的執行。 所有進行中和擱置的執行都會取消。 如果您有數千個執行，加以取消可能需要很長的時間。 

## <a name="next-steps"></a>後續步驟

您已在本文中了解如何使用 Visual Studio 管理已部署的邏輯應用程式。 接下來，請了解如何針對部署自訂邏輯應用程式定義：

> [!div class="nextstepaction"]
> [以 JSON 撰寫邏輯應用程式定義](../logic-apps/logic-apps-author-definitions.md)
