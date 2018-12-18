---
title: 連線至 Azure Blob 儲存體 - Azure Logic Apps | Microsoft Docs
description: 使用 Azure Logic Apps 在 Azure 儲存體中建立和管理 Blob
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.topic: article
ms.date: 05/21/2018
tags: connectors
ms.openlocfilehash: d8658740772ed4a11fdfd70a0c925ac1b597dd69
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47452015"
---
# <a name="create-and-manage-blobs-in-azure-blob-storage-with-azure-logic-apps"></a>使用 Azure Logic Apps 在 Azure Blob 儲存體中建立和管理 Blob

此文章說明如何使用 Azure Blob 儲存體連接器，從邏輯應用程式內存取和管理 Azure 儲存體帳戶中儲存為 Blob 的檔案。 這樣一來，您就可以建立邏輯應用程式，來自動執行用於管理檔案的工作和工作流程。 例如，您可以建置邏輯應用程式，以在儲存體帳戶中建立、取得、更新和刪除檔案。

假設您有一個會在 Azure 網站上進行更新的工具。 並將它作為邏輯應用程式的觸發程序。 當此事件發生時，您可以讓邏輯應用程式更新 Blob 儲存體容器中的某些檔案，而這是邏輯應用程式中的其中一個動作。 

> [!NOTE]
> Logic Apps 不支援透過防火牆直接連線至 Azure 儲存體帳戶。 若要存取這些儲存體帳戶，請使用以下任一個選項： 
>
> * 建立[整合服務環境](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md)，此環境可連線至 Azure 虛擬網路中的資源。 
> 
> * 如果您已經使用「API 管理」，則可以針對這個案例使用這個服務。 如需詳細資訊，請參閱[簡單的企業整合架構](http://aka.ms/aisarch)。

如果您還不熟悉邏輯應用程式，請檢閱[什麼是 Azure Logic Apps？](../logic-apps/logic-apps-overview.md)和[快速入門：建立第一個邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。
如需連接器專屬的技術資訊，請參閱 <a href="https://docs.microsoft.com/connectors/azureblobconnector/" target="blank">Azure Blob 儲存體連接器參考</a>。

## <a name="prerequisites"></a>必要條件

* 如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 

* [Azure 儲存體帳戶和儲存體容器](../storage/blobs/storage-quickstart-blobs-portal.md)

* 需要從中存取 Azure Blob 儲存體帳戶的邏輯應用程式。 若要使用 Azure Blob 儲存體觸發程序來啟動邏輯應用程式，您需要[空白邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。 

<a name="add-trigger"></a>

## <a name="add-blob-storage-trigger"></a>新增 Blob 儲存體觸發程序

在 Azure Logic Apps 中，每個邏輯應用程式都必須使用[觸發程序](../logic-apps/logic-apps-overview.md#logic-app-concepts)啟動，而該觸發程序會在特定事件發生或符合特定條件時引發。 每次引發觸發程序時，Logic Apps 引擎都會建立邏輯應用程式執行個體，並開始執行應用程式的工作流程。

此範例說明如何在儲存體容器中有新增或更新的 Blob 屬性時，使用 **Azure Blob 儲存體 - 新增或修改 Blob 時 (僅限屬性)** 觸發程序來啟動邏輯應用程式工作流程。 

1. 在 Azure 入口網站或 Visual Studio 中，建立空白的邏輯應用程式，以開啟邏輯應用程式設計工具。 這個範例會使用 Azure 入口網站。

2. 在搜尋方塊中，輸入「azure blob」作為篩選條件。 從觸發程序清單中，選取您想要的觸發程序。

   此範例會使用下列觸發程序：**Azure Blob 儲存體 - 新增或修改 Blob 時 (僅限屬性)**

   ![選取觸發程序](./media/connectors-create-api-azureblobstorage/azure-blob-trigger.png)

3. 如果系統提示您輸入連線詳細資料，請[立即建立 Blob 儲存體連線](#create-connection)。 或者，如果連線已存在，請提供觸發程序的必要資訊。

   在此範例中，請選取您想要監視的容器和資料夾。

   1. 在 [容器] 方塊中，選取 [資料夾] 圖示。

   2. 在資料夾清單中，選擇右角括弧 ( **>** )，然後一直瀏覽，直到您找到並選取您想要的資料夾。 

      ![選取資料夾](./media/connectors-create-api-azureblobstorage/trigger-select-folder.png)

   3. 選取您想要讓觸發程序檢查資料夾是否有變更的間隔和頻率。

4. 當您完成時，請在設計工具的工具列上，選擇 [儲存]。

5. 現在，繼續針對您想要使用觸發程序結果來執行的工作，於邏輯應用程式中新增一或多個動作。

<a name="add-action"></a>

## <a name="add-blob-storage-action"></a>新增 Blob 儲存體動作

在 Azure Logic Apps 中，[動作](../logic-apps/logic-apps-overview.md#logic-app-concepts)是工作流程中跟隨在觸發程序或另一個動作之後的步驟。 在此範例中，邏輯應用程式會使用[循環觸發程序](../connectors/connectors-native-recurrence.md)來啟動。

1. 在 Azure 入口網站或 Visual Studio 的邏輯應用程式設計工具中，開啟邏輯應用程式。 這個範例會使用 Azure 入口網站。

2. 在邏輯應用程式設計工具的觸發程序或動作下，選擇 [新增步驟] > [新增動作]。

   ![新增動作](./media/connectors-create-api-azureblobstorage/add-action.png) 

   若要在現有步驟之間新增動作，請將滑鼠放在連接箭頭上。 
   選擇顯示的加號 (**+**)，然後選擇 [新增動作]。

3. 在搜尋方塊中，輸入「azure blob」作為篩選條件。 從 [動作] 清單中，選取您想要的動作。

   此範例會使用下列動作：**Azure Blob 儲存體 - 取得 Blob 內容**

   ![選取動作](./media/connectors-create-api-azureblobstorage/azure-blob-action.png) 

4. 如果系統提示您輸入連線詳細資料，請[立即建立 Azure Blob 儲存體連線](#create-connection)。 或者，如果連線已存在，請提供動作的必要資訊。 

   針對此範例，請選取您想要的檔案。

   1. 從 [Blob] 方塊中，選取 [資料夾] 圖示。
  
      ![選取資料夾](./media/connectors-create-api-azureblobstorage/action-select-folder.png)

   2. 根據 Blob 的**識別碼**，尋找並選取您想要的檔案。 您可以在先前所述 Blob 儲存體觸發程序所傳回的 Blob 中繼資料內，找到此**識別碼**。

5. 當您完成時，請在設計工具的工具列上，選擇 [儲存]。
若要測試邏輯應用程式，請確定所選取的資料夾包含 Blob。

此範例只會取得 Blob 的內容。 若要檢視內容，請新增另一個動作，以使用另一個連接器建立具有 Blob 的檔案。 例如，新增 OneDrive 動作來根據 Blob 內容建立檔案。

<a name="create-connection"></a>

## <a name="connect-to-storage-account"></a>連線至儲存體帳戶

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

[!INCLUDE [Create a connection to Azure blob storage](../../includes/connectors-create-api-azureblobstorage.md)]

## <a name="connector-reference"></a>連接器參考

如需連接器的 Swagger 檔案所敘述的技術詳細資料 (例如，觸發程序、動作和限制)，請參閱[連接器的參考頁面](/connectors/azureblobconnector/)。 

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)
