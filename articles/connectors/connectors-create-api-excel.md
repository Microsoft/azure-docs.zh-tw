---
title: 連線到 Excel Online - Azure Logic Apps | Microsoft Docs
description: 使用 Excel Online REST API 和 Azure Logic Apps 來管理資料
ms.service: logic-apps
services: logic-apps
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.suite: integration
tags: connectors
ms.topic: article
ms.date: 08/23/2018
ms.openlocfilehash: ceef6c5f32372bb69f6ce789e755bc540cb12ba1
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43051944"
---
# <a name="manage-excel-online-data-with-azure-logic-apps"></a>使用 Azure Logic Apps 來管理 Excel Online 資料

使用 Azure Logic Apps 和 Excel Online 連接器，您可以根據商務用 Excel Online 或適用於 OneDrive 的 Excel Online 中的資料，建立自動化的工作和工作流程。 此連接器所提供的動作，可協助您處理資料和管理試算表，例如： 

* 建立新的工作表和資料表。
* 取得及管理工作表、資料表和資料列。
* 新增單一資料列和索引鍵資料行。

然後，您可以搭配其他服務的動作，使用這些動作的輸出。 例如，如果您使用會每週建立工作表的動作，則可以使用其他動作，藉由使用 Office 365 Outlook 連接器，來傳送確認電子郵件。

如果您不熟悉邏輯應用程式，請檢閱[什麼是 Azure Logic Apps？](../logic-apps/logic-apps-overview.md)

> [!NOTE]
> [商務用 Excel Online](/connectors/excelonlinebusiness/) 和[適用於 OneDrive 的 Excel Online](/connectors/excelonline/) 連接器可以與 Azure Logic Apps 搭配運作，而且與[適用於 PowerApps 的 Excel 連接器](/connectors/excel/)不同。

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 

* 適用於您的工作帳戶或個人 Microsoft 帳戶的 [Office 365 帳戶](https://www.office.com/) 

  您的 Excel 資料可以是儲存體資料夾中 (例如在 OneDrive 中) 的逗號分隔值 (CSV) 檔案。 
  您也可以使用此相同的 CSV 檔案，與[一般檔案連接器](../logic-apps/logic-apps-enterprise-integration-flatfile.md)搭配使用。

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識

* 您要在其中存取 Excel Online 資料的邏輯應用程式。 此連接器只提供動作，因此若要啟動您的邏輯應用程式，請選取個別觸發程序，例如「週期」觸發程序。

## <a name="add-excel-action"></a>新增 Excel 動作

1. 在 [Azure 入口網站](https://portal.azure.com)中，如果邏輯應用程式尚未開啟，請在邏輯應用程式設計工具中開啟邏輯應用程式。

1. 在觸發程序下方，選擇 [新增步驟]。

1. 在搜尋方塊中，輸入 "excel" 作為篩選條件。 請在動作清單底下，選取您想要的動作。

1. 如果系統提示您登入 Office 365 帳戶，請選擇 [登入]。 

   您的認證會授權邏輯應用程式建立與 Excel Online 的連線，並存取資料。

1. 為選取的動作提供必要的詳細資料，並且建置邏輯應用程式的工作流程。

## <a name="connector-reference"></a>連接器參考

如需連接器的 Swagger 檔案所述技術詳細資料 (例如，動作和限制)，請參閱以下連接器參考頁面：

* [商務用 Excel Online](/connectors/excelonlinebusiness/) 
* [適用於 OneDrive 的 Excel Online](/connectors/excelonline/) 

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)
