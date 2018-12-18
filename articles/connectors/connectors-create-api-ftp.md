---
title: 連線到 FTP 伺服器 - Azure Logic Apps | Microsoft Docs
description: 使用 Azure Logic Apps 在 FTP 伺服器上建立、監視和管理檔案
author: ecfan
manager: jeconnoc
ms.author: estfan
ms.date: 07/22/2016
ms.topic: article
ms.service: logic-apps
services: logic-apps
ms.reviewer: klam, LADocs
ms.suite: integration
tags: connectors
ms.openlocfilehash: 4355a767d2ecd500662cdf4522e8a7e12de86b80
ms.sourcegitcommit: ab3b2482704758ed13cccafcf24345e833ceaff3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/06/2018
ms.locfileid: "37866146"
---
# <a name="get-started-with-the-ftp-connector"></a>開始使用 FTP 連接器
使用 FTP 連接器在 FTP 伺服器上監視、管理和建立檔案。 

若要使用[任何連接器](apis-list.md)，您必須先建立邏輯應用程式。 您可以從[立即建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)來開始。

## <a name="connect-to-ftp"></a>連接至 FTP
您必須先建立與服務的連線，才能透過邏輯應用程式存取任何服務。 [連線](connectors-overview.md)可讓邏輯應用程式與另一個服務連線。  

### <a name="create-a-connection-to-ftp"></a>建立至 FTP 的連線
> [!INCLUDE [Steps to create a connection to FTP](../../includes/connectors-create-api-ftp.md)]
> 
> 

## <a name="use-a-ftp-trigger"></a>使用 FTP 觸發程序
觸發程序是可用來啟動邏輯應用程式中所定義之工作流程的事件。 [深入了解觸發程序](../logic-apps/logic-apps-overview.md#logic-app-concepts)。  

> [!IMPORTANT]
> FTP 連接器需要可從網際網路存取且設定為以「被動」模式運作的 FTP 伺服器。 此外，FTP 連接器**與隱含 FTPS (FTP over SSL) 不相容**。 FTP 連接器只支援明確 FTPS (FTP over SSL)。  
> 
> 

在此範例中，我將告訴您如何使用 **FTP - 新增或修改檔案時**觸發程序，在檔案新增或修改於 FTP 伺服器時起始邏輯應用程式工作流程。 在企業範例中，您可以使用此觸發程序來監視代表客戶訂單的新檔案 FTP 資料夾。  然後，您可以使用 FTP 連接器動作 (例如**取得檔案內容**) 取得訂單的內容以進一步處理並儲存在訂單資料庫中。

1. 在邏輯應用程式設計工具的搜尋方塊中輸入 *ftp*，然後選取 [SFTP - 當新增或修改檔案時] 觸發程序   
   ![FTP 觸發程序影像 1](./media/connectors-create-api-ftp/ftp-trigger-1.png)  
   [當新增或修改檔案時] 控制項隨即開啟  
   ![FTP 觸發程序影像 2](./media/connectors-create-api-ftp/ftp-trigger-2.png)  
2. 選取位於控制項右側的 [...]  。 這會開啟資料夾選擇器控制項   
   ![FTP 觸發程序影像 3](./media/connectors-create-api-ftp/ftp-trigger-3.png)  
3. 選取 **>** (向右箭號)，並瀏覽尋找您要對新的或修改過檔案監視的資料夾。 選取資料夾，請注意資料夾現已顯示在 [資料夾] 控制項中。  
   ![FTP 觸發程序影像 4](./media/connectors-create-api-ftp/ftp-trigger-4.png)   

此時，邏輯應用程式已設有觸發程序，該觸發程序會在檔案於特定 FTP 資料夾中修改或建立時，開始執行工作流程中的其他觸發程序和動作。 

> [!NOTE]
> 若要讓邏輯應用程式能夠運作，它必須至少包含一個觸發程序和一個動作。 請依照下一節中的步驟來新增動作。  
> 
> 

## <a name="use-a-ftp-action"></a>使用 FTP 動作
動作是由邏輯應用程式中定義的工作流程所執行的作業。 [深入了解動作](../logic-apps/logic-apps-overview.md#logic-app-concepts)。  

您現已新增觸發程序，請遵循下列步驟來新增動作，該動作將會取得觸發程序所找到之新的或修改過檔案的內容。    

1. 選取 [+ 新增步驟] 來新增動作，以取得 FTP 伺服器上檔案的內容  
2. 選取 [新增動作]  連結。  
   ![FTP 動作影像 1](./media/connectors-create-api-ftp/ftp-action-1.png)  
3. 輸入 *FTP* 以搜尋與 FTP 相關的所有動作。
4. 選取 [FTP - 取得檔案內容]，做為在 FTP 資料夾中找到新的或修改過檔案時所要採取的動作。      
   ![FTP 動作影像 2](./media/connectors-create-api-ftp/ftp-action-2.png)  
   [取得檔案內容] 控制項隨即開啟。 **附註：** 如果您未曾授權邏輯應用程式存取您的 FTP 伺服器帳戶，系統會提示您這麼做。  
   ![FTP 動作影像 3](./media/connectors-create-api-ftp/ftp-action-3.png)   
5. 選取 [檔案] 控制項 (位於 **FILE*** 下方的空白處)。 在這裡，您可以使用 FTP 伺服器上找到之新的或修改過檔案中的各種屬性。  
6. 選取 [檔案內容] 選項。  
   ![FTP 動作影像 4](./media/connectors-create-api-ftp/ftp-action-4.png)   
7. 控制項已更新，這表示 [FTP - 取得檔案內容] 動作會取得 FTP 伺服器上新的或修改過檔案的*檔案內容*。      
   ![FTP 動作影像 5](./media/connectors-create-api-ftp/ftp-action-5.png)     
8. 儲存您的工作，然後將檔案加入至 FTP 資料夾，以測試您的工作流程。    

此時，邏輯應用程式已設有觸發程序來監視 FTP 伺服器上的資料夾，而當它在 FTP 伺服器上找到新的檔案或修改過的檔案時會起始工作流程。 

邏輯應用程式也已設有一個動作，以取得新的或修改過檔案的內容。

您現在可以新增另一個動作，例如 [SQL Server - 插入資料列](connectors-create-api-sqlazure.md)動作，已在 SQL Database 資料表中插入新的或修改過檔案的內容。  

## <a name="connector-specific-details"></a>連接器特定的詳細資料

檢視 Swagger 中定義的任何觸發程序和動作，另請參閱[連接器詳細資料](/connectors/ftpconnector/)的所有限制。 

## <a name="next-steps"></a>後續步驟
[建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)

