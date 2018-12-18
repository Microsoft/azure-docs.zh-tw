---
title: 從 Azure Logic Apps 連線到 SFTP 帳戶 | Microsoft Docs
description: 藉由使用 Azure Logic Apps，讓針對 SFTP 伺服器監視、建立、管理、傳送及接收檔案的工作和工作流程自動化
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.assetid: 697eb8b0-4a66-40c7-be7b-6aa6b131c7ad
ms.topic: article
tags: connectors
ms.date: 08/24/2018
ms.openlocfilehash: 8f430477883543aa8f87eb3fb0fb49ab31e2d723
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43042033"
---
# <a name="monitor-create-and-manage-sftp-files-by-using-azure-logic-apps"></a>藉由使用 Azure Logic Apps 來監視、建立及管理 SFTP 檔案

您可以使用 Azure Logic Apps 和 SFTP 連接器，建立自動化的工作和工作流程，以便透過您在 [SFTP](https://www.ssh.com/ssh/sftp/) (英文) 伺服器上的帳戶，監視、建立、傳送及接收檔案，還有其他動作，例如：

* 監視檔案何時新增或變更。
* 取得、建立、複製、更新、列出以及刪除檔案。
* 取得檔案內容與中繼資料。
* 將封存檔案解壓縮到資料夾。

您可以使用觸發程序，從您的 SFTP 伺服器收到回應，並且讓輸出可供其他動作使用。 您可以在邏輯應用程式中使用動作，對 SFTP 伺服器上的檔案執行工作。 您也可以讓其他動作使用 SFTP 動作的輸出。 例如，如果您定期從 SFTP 伺服器擷取檔案，可以藉由使用 Office 365 Outlook 連接器或 Outlook.com 連接器，傳送關於這些檔案及其內容的電子郵件。
如果您不熟悉邏輯應用程式，請檢閱[什麼是 Azure Logic Apps？](../logic-apps/logic-apps-overview.md)

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 

* 您的 SFTP 主機伺服器位址和帳戶認證

   您的認證會授權邏輯應用程式建立連線並存取 SFTP 帳戶。

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識

* 您要在其中存取 SFTP 帳戶的邏輯應用程式。 若要開始使用 SFTP 觸發程序，請[建立空白邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。 若要使用 SFTP 動作，請使用其他觸發程序來啟動邏輯應用程式，例如「週期」觸發程序。

## <a name="connect-to-sftp"></a>連接至 SFTP

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. 登入 [Azure 入口網站](https://portal.azure.com)，如果邏輯應用程式尚未開啟，請在邏輯應用程式設計工具中開啟邏輯應用程式。

1. 針對空白邏輯應用程式，請在搜尋方塊中輸入 "sftp" 作為篩選條件。 請在觸發程序清單底下，選取您想要的觸發程序。 

   -或-

   若是現有的邏輯應用程式，請在想要新增動作的最後一個步驟底下，選擇 [新增步驟]。 
   在搜尋方塊中，輸入 "sftp" 作為篩選條件。 
   請在動作清單底下，選取您想要的動作。

   若要在步驟之間新增動作，將指標移至步驟之間的箭號。 
   選擇顯示的加號 (**+**)，然後選取 [新增動作]。

1. 為您的連線提供必要的詳細資料，然後選擇 [建立]。

1. 為您選取的觸發程序或動作提供必要的詳細資料，並且繼續建置邏輯應用程式的工作流程。

## <a name="examples"></a>範例

### <a name="sftp-trigger-when-a-file-is-added-or-modified"></a>SFTP 觸發程序：當新增或修改檔案時

此觸發程序會在偵測到 SFTP 伺服器上有檔案新增或變更時，啟動邏輯應用程式工作流程。 因此舉例來說，您可以新增條件，檢查檔案的內容，並且根據該內容是否符合指定條件，來決定是否取得該內容。 最後，您可以新增會取得檔案內容的動作，並將該內容放置於 SFTP 伺服器上的資料夾中。 

**企業範例**：您可以使用此觸發程序，來監視代表客戶訂單的新檔案 SFTP 資料夾。 然後，您可以使用 SFTP 動作 (例如**取得檔案內容**)，取得訂單的內容以進一步處理，並儲存在訂單資料庫中。

### <a name="sftp-action-get-content"></a>SFTP 動作：取得內容

此動作會從 SFTP 伺服器上的檔案取得內容。 舉例來說，您可以新增來自上一個範例中的觸發程序，以及新增檔案內容必須符合的條件。 如果條件為 true，則可以執行會取得內容的動作。 

## <a name="connector-reference"></a>連接器參考

如需觸發程序、動作和限制的技術詳細資訊，它們是由連接器的 OpenAPI (以前稱為 Swagger) 來描述，請檢閱連接器的[參考頁面](/connectors/sftpconnector/)。

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)