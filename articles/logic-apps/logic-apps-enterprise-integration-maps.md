---
title: 使用 XSLT 對應來轉換 XML - Azure Logic Apps | Microsoft Docs
description: 新增 XSLT 對應，以在採用 Enterprise Integration Pack 的 Azure Logic Apps 中轉換 XML
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.assetid: 90f5cfc4-46b2-4ef7-8ac4-486bb0e3f289
ms.date: 07/08/2016
ms.openlocfilehash: c5e5e0a0a3f8bd5feedc00d5bbfb76a1453ccc84
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43123551"
---
# <a name="add-maps-for-xml-transformation-in-azure-logic-apps-with-enterprise-integration-pack"></a>在採用 Enterprise Integration Pack 的 Azure Logic Apps 中新增 XML 轉換的對應

企業整合會使用對應在各種格式之間轉換 XML 資料。 對應是一份 XML 文件，可定義應該將文件中的哪些資料轉換成其他格式。 

## <a name="why-use-maps"></a>為什麼要使用對應？

假設您會定期收到客戶的 B2B 訂單或發票，而該客戶使用 YYYMMDD 格式的日期。 不過，在您的組織中，您是以 MMDDYYY 格式儲存日期。 您可以使用對應，先將 YYYMMDD 日期格式*轉換*為 MMDDYYY，然後再將訂單或發票儲存於客戶活動資料庫中。


## <a name="how-do-i-create-a-map"></a>如何建立對應？

您可以使用適用於 Visual Studio 2015 的[企業整合套件](logic-apps-enterprise-integration-overview.md "了解企業整合套件")來建立 BizTalk 整合專案。 接著，您可以建立整合對應檔案，以便以視覺方式對應兩個 XML 結構描述檔案中的項目。 建置此專案之後，您將擁有 XSLT 文件。

如果對應有外部組件的參考，兩者均必須上傳到整合帳戶。 需依照特定順序上傳，先上傳組件，再上傳參考組件的對應。


## <a name="how-do-i-add-a-map"></a>如何新增對應？

1. 在 Azure 入口網站中，選取 [瀏覽]。

    ![](./media/logic-apps-enterprise-integration-overview/overview-1.png)

2. 在篩選搜尋方塊中輸入**整合**，然後從結果清單選取 [整合帳戶]。

    ![](./media/logic-apps-enterprise-integration-overview/overview-2.png)

3. 選取要將對應新增到其中的整合帳戶。

    ![](./media/logic-apps-enterprise-integration-overview/overview-3.png)

4. 選取 [對應] 圖格。

    ![](./media/logic-apps-enterprise-integration-maps/map-1.png)

5. 在 [對應] 頁面開啟之後，選擇 [新增]。

    ![](./media/logic-apps-enterprise-integration-maps/map-2.png)  

6. 在 [名稱] 中輸入對應的名稱。 若要上傳對應檔案，請選擇 [對應] 文字方塊右邊的資料夾圖示。 上傳程序完成之後，請選擇 [確定]。

    ![](./media/logic-apps-enterprise-integration-maps/map-3.png)

7. 在 Azure 將您的對應新增到您的整合帳戶之後，您會看到螢幕上出現一個訊息，該訊息說明您的對應檔案是否已新增。 取得此訊息之後，請選擇 [對應]**Maps** 圖格，以便檢視新增的對應。

    ![](./media/logic-apps-enterprise-integration-maps/map-4.png)


## <a name="how-do-i-add-an-assembly"></a>如何新增組件？
開啟要上傳組件的企業整合帳戶。

1. 選擇 [組件] 圖格。

    ![integrationaccount-assembly-tile](./media/logic-apps-enterprise-integration-maps/assemblytile.png)

2. 在 [組件] 頁面開啟之後，選擇 [新增]。 請為組件輸入**名稱**。 若要上傳組件檔案，請選擇 [組件] 文字方塊右邊的資料夾圖示。 上傳程序完成之後，請選擇 [確定]。

    ![add-assembly](./media/logic-apps-enterprise-integration-maps/assemblyfile.png)


## <a name="how-do-i-edit-a-map"></a>如何編輯對應？

您必須上傳具有您要之變更的新對應檔案。 您可以先下載對應以進行編輯。

若要上傳將取代現有對應的新對應，請依照這些步驟執行。

1. 選擇 [對應] 圖格。

2. 在 [對應] 頁面開啟之後，選取您要編輯的對應。

3. 在 [對應] 頁面上，選擇 [更新]。

    ![](./media/logic-apps-enterprise-integration-maps/edit-1.png)

4. 在檔案選擇器中，選取您要上傳的對應檔案，然後選取 [開啟]。

    ![](./media/logic-apps-enterprise-integration-maps/edit-2.png)

## <a name="how-to-delete-a-map"></a>如何刪除對應？

1. 選擇 [對應] 圖格。

2. 在 [對應] 頁面開啟之後，選取您要刪除的對應。

3. 選擇 [刪除]。

    ![](./media/logic-apps-enterprise-integration-maps/delete.png)

4. 確認您要刪除該對應。

    ![](./media/logic-apps-enterprise-integration-maps/delete-confirmation-1.png)

## <a name="next-steps"></a>後續步驟
* [深入了解企業整合套件](logic-apps-enterprise-integration-overview.md "了解企業整合套件")  
* [深入了解合約](../logic-apps/logic-apps-enterprise-integration-agreements.md "了解企業整合合約")  
* [深入了解轉換](logic-apps-enterprise-integration-transform.md "了解企業整合轉換")  

