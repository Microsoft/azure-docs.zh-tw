---
title: 連線到 Dropbox - Azure Logic Apps | Microsoft Docs
description: 使用 Dropbox REST API 和 Azure Logic Apps上傳及管理檔案
author: ecfan
manager: jeconnoc
ms.author: estfan
ms.date: 07/15/2016
ms.topic: article
ms.service: logic-apps
services: logic-apps
ms.reviewer: klam, LADocs
ms.suite: integration
tags: connectors
ms.openlocfilehash: 256a0b34d5050e17abe5bb98ca0c13ab0b61787e
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43094433"
---
# <a name="get-started-with-the-dropbox-connector"></a>開始使用 Dropbox 連接器
連線到 Dropbox 來管理您的檔案。 您可以執行各種動作，例如上傳、更新、取得及刪除 Dropbox 中的檔案。

若要使用[任何連接器](apis-list.md)，您必須先建立邏輯應用程式。 您可以從[立即建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)來開始。

## <a name="connect-to-dropbox"></a>連接至 Dropbox。
您必須先建立與服務的連線，才能透過邏輯應用程式存取任何服務。 連線可讓邏輯應用程式與另一個服務連線。 例如，若要連線至 Dropbox，您必須先建立 Dropbox *連線*。 若要建立連線，您需要提供平常用來存取所要連線之服務的認證。 因此，在 Dropbox 範例中，您需要 Dropbox 帳戶的認證，才能建立與 Dropbox 的連線。 

### <a name="create-a-connection-to-dropbox"></a>建立 Dropbox 連線
> [!INCLUDE [Steps to create a connection to Dropbox](../../includes/connectors-create-api-dropbox.md)]
> 
> 

## <a name="use-a-dropbox-trigger"></a>使用 Dropbox 觸發程序
觸發程序是可用來啟動邏輯應用程式中所定義之工作流程的事件。 [深入了解觸發程序](../logic-apps/logic-apps-overview.md#logic-app-concepts)。

在此範例中，我們將使用**建立檔案時**觸發程序。 當此觸發程序發生時，我們會呼叫**使用路徑來取得檔案內容** Dropbox 動作。 

1. 在 Logic Apps 設計工具的搜尋方塊中輸入 *dropbox*，然後選取 [Dropbox - 建立檔案時] 觸發程序。      
   ![](../../includes/media/connectors-create-api-dropbox/using-dropbox-trigger.PNG)  
2. 選取您想要追蹤檔案建立所在的資料夾。 選取 [...]\(以紅色方塊識別)，並瀏覽至您想要針對觸發程序的輸入而選取的資料夾。  
   ![](../../includes/media/connectors-create-api-dropbox/using-dropbox-trigger-2.PNG)  

## <a name="use-a-dropbox-action"></a>使用 Dropbox 動作
動作是由邏輯應用程式中定義的工作流程所執行的作業。 [深入了解動作](../logic-apps/logic-apps-overview.md#logic-app-concepts)。

現在已新增觸發程序，請遵循下列步驟來新增將會取得新檔案內容的動作。

1. 選取 [+ 新的步驟] 來新增您想要在新檔案建立時採取的動作。  
   ![](../../includes/media/connectors-create-api-dropbox/using-dropbox-action.PNG)
2. 選取 [新增動作]。 這會開啟搜尋方塊，您可以在其中搜尋任何想要採取的動作。  
   ![](../../includes/media/connectors-create-api-dropbox/using-dropbox-action-2.PNG)
3. 輸入 *dropbox* 以搜尋與 Dropbox 相關的動作。  
4. 選取 [Dropbox - 使用路徑來取得檔案內容]，做為在選取的 Dropbox 資料夾中建立新檔案時所要採取的動作。 動作控制區塊便會開啟。 如果您未曾授權邏輯應用程式存取您的 Dropbox 帳戶，系統會提示您這麼做。  
   ![](../../includes/media/connectors-create-api-dropbox/using-dropbox-action-3.PNG)  
5. 選取 [...]\(位於 [檔案路徑] 控制項的右側)，並瀏覽至您想要使用的檔案路徑。 或者，使用**檔案路徑**權杖來加速建立邏輯應用程式。  
   ![](../../includes/media/connectors-create-api-dropbox/using-dropbox-action-4.PNG)  
6. 儲存您的工作並在 Dropbox 中建立新檔案，以啟動您的工作流程。  

## <a name="connector-specific-details"></a>連接器特定的詳細資料

檢視 Swagger 中定義的任何觸發程序和動作，另請參閱[連接器詳細資料](/connectors/dropbox/)的所有限制。

## <a name="more-connectors"></a>其他連接器
返回 [API 清單](apis-list.md)。
