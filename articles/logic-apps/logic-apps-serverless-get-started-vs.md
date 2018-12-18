---
title: 使域 Visual Studio 建置無伺服器的應用程式 | Microsoft Docs
description: 在 Visual Studio 中使用 Azure Functions 和 Azure Logic Apps 建置、部署及管理您的第一個無伺服器應用程式
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.assetid: d565873c-6b1b-4057-9250-cf81a96180ae
ms.custom: vs-azure
ms.topic: article
ms.date: 08/01/2018
ms.openlocfilehash: f5555d9a60934529bf8fed6db6a18dd783f46075
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44297463"
---
# <a name="build-your-first-serverless-app-with-azure-logic-apps-and-azure-functions---visual-studio"></a>使用 Azure Logic Apps 和 Azure Functions 建置您的第一個無伺服器應用程式 - Visual Studio

您可以在 Azure 中使用 [Azure Logic Apps](../logic-apps/logic-apps-overview.md) 和 [Azure Functions](../azure-functions/functions-overview.md) 等無伺服器工具和功能，快速開發及部署雲端應用程式。 本文說明如何開始在 Visual Studio 中建置無伺服器應用程式，而該應用程式會使用可呼叫 Azure 函式的邏輯應用程式。 若要深入了解 Azure 中的無伺服器解決方案，請參閱[採用 Functions 和 Logic Apps 的 Azure 無伺服伺服器](../logic-apps/logic-apps-serverless-overview.md)。

## <a name="prerequisites"></a>必要條件

若要在 Visual Studio 中建置無伺服器的應用程式，您需要下列項目：

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請先[註冊免費的 Azure 帳戶](https://azure.microsoft.com/free/)。

* [Visual Studio 2017](https://www.visualstudio.com/vs/) 或 Visual Studio 2015 - Community、Professional 或 Enterprise

* [Microsoft Azure SDK](https://azure.microsoft.com/downloads/) (2.9.1 或更新版本)

* [Azure PowerShell](https://github.com/Azure/azure-powershell#installation)

* [Azure Logic Apps Tools for Visual Studio 2017](https://marketplace.visualstudio.com/items?itemName=VinaySinghMSFT.AzureLogicAppsToolsforVisualStudio-18551) 或 [Visual Studio 2015 版本](https://marketplace.visualstudio.com/items?itemName=VinaySinghMSFT.AzureLogicAppsToolsforVisualStudio)

  您可以直接從 Visual Studio Marketplace 下載並安裝 Azure Logic Apps Tools，或了解[如何從 Visual Studio 內部安裝此擴充功能](https://docs.microsoft.com/visualstudio/ide/finding-and-using-visual-studio-extensions)。 
  請務必在完成安裝之後重新啟動 Visual Studio。 

* [Azure Functions 核心工具](https://www.npmjs.com/package/azure-functions-core-tools)，適用於在本機對 Functions 進行偵錯

* 使用 Visual Studio 中內嵌的 Logic Apps 設計工具時能夠存取 Web

  設計工具需要網際網路連線才能在 Azure 中建立資源，以及從邏輯應用程式中的連接器讀取屬性和資料。 
  例如，如果您使用 Dynamics CRM Online 連接器，則設計工具會檢查您的 CRM 執行個體，以取得可用的預設和自訂屬性。

## <a name="create-resource-group-project"></a>建立資源群組專案

若要開始進行，請為您的無伺服器應用程式建立 [Azure 資源群組專案](../azure-resource-manager/vs-azure-tools-resource-groups-deployment-projects-create-deploy.md)。 在 Azure 中，您會在資源群組內建立資源，該資源群組是一個邏輯集合，用於針對整個應用程式 (當作單一資產) 組織、管理及部署資源。 對於 Azure 中的無伺服器應用程式，您的資源群組包含 Azure Logic Apps 和 Azure Functions 資源。 深入了解 [Azure 資源群組和資源](../azure-resource-manager/resource-group-overview.md)。

1. 啟動 Visual Studio，並以您的 Azure 帳戶登入。 

1. 在 [檔案] 功能表上，選取 [新增] > [專案]。 

   ![在 Visual Studio 中建立新專案](./media/logic-apps-serverless-get-started-vs/create-new-project-visual-studio.png)

1. 在 [已安裝] 之下，選取 **Visual C#** 或 **Visual Basic**。 選取 [雲端] > [Azure 資源群組]。

   如果 [雲端] 類別或 [Azure 資源群組] 不存在，請確定已安裝 Azure SDK for Visual Studio。

1. 為您的專案提供名稱和位置，然後選擇 [確定]。 

   Visual Studio 會提示您選取範本。 
   您可以從空白、邏輯應用程式或其他範本著手，但此範例使用 Azure 快速入門範本來建置無伺服器應用程式，其中包含邏輯應用程式和 Azure 函式的呼叫。

   若只要在 Visual Studio 中建立邏輯應用程式，請選取 [邏輯應用程式] 範本。 此範本會建立可在 Logic Apps 設計工具中開啟的空白邏輯應用程式，而不需將解決方案預先部署到 Azure 資源群組中。

1. 在 [從這個位置顯示範本] 之下，選取 [Azure 快速入門 (github/Azure/azure-quickstart-template)]。 

1. 在搜尋方塊中，輸入 "logic-app" 作為篩選條件，選取此無伺服器快速入門範本，然後選擇 [確定]：**101-logic-app-and-function-app**

   ![選取 Azure 快速入門範本](./media/logic-apps-serverless-get-started-vs/select-template.png)

   Visual Studio 會為您的資源群組專案建立並開啟解決方案。 
   您選取的快速入門範本會在您的資源群組專案中建立名為 `azuredeploy.json` 的部署範本。 
   此部署範本包含簡單邏輯應用程式的定義，該邏輯應用程式會在 HTTP 要求上觸發、呼叫 Azure 函式，以及傳回結果作為 HTTP 回應。 
   
   ![新增無伺服器解決方案](./media/logic-apps-serverless-get-started-vs/create-serverless-solution.png)

1. 接下來，您必須先將解決方案部署至 Azure，才能開啟部署範本並檢閱無伺服器應用程式的資源。 

## <a name="deploy-your-solution"></a>部署您的解決方案

您必須先具備已在 Azure 中部署的 Azure 資源群組，才可以在 Visual Studio 中使用 Logic Apps 設計工具開啟邏輯應用程式。 設計工具可接著在您的邏輯應用程式中建立資源和服務的連線。 針對此工作，從 Visual Studio 將您的解決方案部署至 Azure 入口網站。

1. 在 [方案總管] 中，開啟您資源專案的捷徑功能表，然後選取 [部署] > [新增]。

   ![建立資源群組的新部署](./media/logic-apps-serverless-get-started-vs/deploy.png)

1. 如果尚未選取，請選取您的 Azure 訂用帳戶以及您要在其中部署的資源群組。 選擇 [部署]。

   ![部署設定](./media/logic-apps-serverless-get-started-vs/deploy-to-resource-group.png)

1. 如果 [編輯參數] 方塊出現，請提供在部署時要用於邏輯應用程式和 Azure 函式應用程式的資源名稱，然後儲存您的設定。 確定您的函式應用程式使用全域唯一的名稱。

   ![提供邏輯應用程式和函式應用程式的名稱](./media/logic-apps-serverless-get-started-vs/logic-function-app-name-parameters.png)

   當 Visual Studio 開始部署至您指定的資源群組時，您的解決方案部署狀態就會出現在 Visual Studio 的 [輸出] 視窗。 
   部署完成之後，您的邏輯應用程式會在 Azure 入口網站中上線。

## <a name="edit-logic-app-in-visual-studio"></a>在 Visual Studio 中編輯邏輯應用程式

您的解決方案已經部署至您的資源群組，請使用 Logic Apps 設計工具開啟邏輯應用程式，以便編輯和變更邏輯應用程式。

1. 在 [方案總管] 中，開啟 `azuredeploy.json` 檔案的捷徑功能表，然後選取 [使用 Logic Apps 設計工具開啟]。

   ![在 Logic Apps 設計工具中開啟 "azuredeploy.json"](./media/logic-apps-serverless-get-started-vs/open-logic-app-designer.png)

1. 在 [邏輯應用程式屬性] 方塊出現之後，如果尚未選取，請在 [訂用帳戶] 之下選取您的 Azure 訂用帳戶。 在 [資源群組] 之下，選取您部署解決方案的資源群組和所在位置，然後選擇 [確定]。

   ![邏輯應用程式屬性](./media/logic-apps-serverless-get-started-vs/logic-app-properties.png)

   在 Logic Apps 設計工具開啟之後，您就可以繼續新增步驟或變更工作流程，以及儲存您的更新。

   ![在 Logic Apps 設計工具中開啟的邏輯應用程式](./media/logic-apps-serverless-get-started-vs/opened-logic-app.png)

## <a name="create-azure-functions-project"></a>建立 Azure Functions 專案

若要使用 JavaScript、Python、F#、PowerShell、Batch 或 Bash 建立 Functions 專案和函式，請依照[使用 Azure Functions Core Tools](../azure-functions/functions-run-local.md) 一文中的步驟操作。 若要使用 C# 在您的解決方案內開發 Azure 函式，您可以依照[發佈 .NET 類別庫作為函式應用程式](https://blogs.msdn.microsoft.com/appserviceteam/2017/03/16/publishing-a-net-class-library-as-a-function-app/)一文中的步驟操作來使用 C# 類別庫。

## <a name="deploy-functions-from-visual-studio"></a>從 Visual Studio 部署函式

您的部署範本會從 `azuredeploy.json` 檔案中變數所指定的 Git 存放庫，部署您解決方案中擁有的任何 Azure 函式。 如果您在解決方案中建立並撰寫 Functions 專案，您可以將該專案簽入 Git 原始檔控制 (例如 GitHub 或 Azure DevOps)，然後更新 `repo` 變數，以便此範本部署 Azure 函式。

## <a name="manage-logic-apps-and-view-run-history"></a>管理邏輯應用程式及檢視執行歷程記錄

對於已部署在 Azure 中的邏輯應用程式，您仍然可從 Visual Studio 編輯、管理、檢視執行歷程記錄，以及停用這些應用程式。 

1. 從 Visual Studio 中的 [檢視] 功能表，開啟 [Cloud Explorer]。 

1. 在 [所有訂用帳戶] 之下，選取與您要管理的邏輯應用程式相關聯的 Azure 訂用帳戶，然後選擇 [套用]。

1. 在 [Logic Apps] 下，選取您的邏輯應用程式。 從該應用程式的捷徑功能表，選取 [使用邏輯應用程式編輯器開啟]。 

您現在可以將已經發佈的邏輯應用程式下載到您的資源群組專案中。 所以即便您可能已在 Azure 入口網站中啟動邏輯應用程式，您仍然可在 Visual Studio 中匯入和管理該應用程式。 如需詳細資訊，請參閱[使用 Visual Studio 管理邏輯應用程式](../logic-apps/manage-logic-apps-with-visual-studio.md)。

## <a name="next-steps"></a>後續步驟

* [建置無伺服器的社交儀表板](logic-apps-scenario-social-serverless.md)
* [使用 Visual Studio 管理邏輯應用程式](manage-logic-apps-with-visual-studio.md)
* [邏輯應用程式工作流程定義語言](logic-apps-workflow-definition-language.md)