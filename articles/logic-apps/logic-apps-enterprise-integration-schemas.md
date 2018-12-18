---
title: 因增 XML 驗證的結構描述 - Azure Logic Apps | Microsoft Docs
description: 建立結構描述以在採用 Enterprise Integration Pack 的 Azure Logic Apps 中驗證 XML 文件
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.assetid: 56c5846c-5d8c-4ad4-9652-60b07aa8fc3b
ms.date: 07/29/2016
ms.openlocfilehash: e03346da1c2b77f885c39d5329f990684979c56e
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43123068"
---
# <a name="validate-xml-with-schemas-in-azure-logic-apps-with-enterprise-integration-pack"></a>在採用 Enterprise Integration Pack 的 Azure Logic Apps 中使用結構描述來驗證 XML

結構描述可確認您收到的 XML 文件都有效，而且包含具有預先定義格式的預期資料。 結構描述也可用來驗證在 B2B 案例中交換的訊息。

## <a name="add-a-schema"></a>新增結構描述

1. 在 Azure 入口網站中，選取 [所有服務]。

    ![Azure 入口網站中的 [所有服務]](media/logic-apps-enterprise-integration-schemas/overview-11.png)

2. 在篩選搜尋方塊中輸入**整合**，然後從結果清單中選取 [整合帳戶]。

    ![篩選搜尋方塊](media/logic-apps-enterprise-integration-schemas/overview-21.png)

3. 選取要在其中新增結構描述的**整合帳戶**。

    ![整合帳戶清單](media/logic-apps-enterprise-integration-schemas/overview-31.png)

4. 選擇 [結構描述] 圖格。

    ![範例整合帳戶 "Schemas"](media/logic-apps-enterprise-integration-schemas/schema-11.png)

### <a name="add-a-schema-file-smaller-than-2-mb"></a>新增小於 2 MB 的結構描述檔案

1. 在開啟的 [結構描述] 刀鋒視窗中 (從先前的步驟)，選擇 [新增]。

    ![[結構描述] 刀鋒視窗，[新增]](media/logic-apps-enterprise-integration-schemas/schema-21.png)

2. 輸入結構描述名稱。 選取 [結構描述] 方塊旁的資料夾圖示以上傳結構描述檔案。 上傳程序完成之後，請選取 [確定]。

    ![[新增結構描述] 的螢幕擷取畫面，反白顯示了 [小型檔案]](media/logic-apps-enterprise-integration-schemas/schema-31.png)

### <a name="add-a-schema-file-larger-than-2-mb-up-to-8-mb-maximum"></a>新增大於 2 MB 的結構描述檔案 (最大 8 MB)

這些步驟視 Blob 容器存取層級而異：[公開] 或 [無匿名存取]。

**判斷此存取層級**

1.  開啟 [Azure 儲存體總管]。 

2.  在 [Blob 容器] 下，選取您要的 Blob 容器。 

3.  選取 [安全性]、[存取層級]。

如果 blob 安全性存取層級為**公用**，請遵循下列步驟。

![Azure 儲存體總管，已反白選取 [Blob 容器]、[安全性] 與 [公開]](media/logic-apps-enterprise-integration-schemas/blob-public.png)

1. 將結構描述上傳至儲存體帳戶，並複製 URI。

    ![儲存體帳戶，已反白顯示 URI](media/logic-apps-enterprise-integration-schemas/schema-blob.png)

2. 在 [新增結構描述] 中選取 [大型檔案]，然後在 [內容 URI] 文字方塊中提供 URI。

    ![[結構描述]，已反白顯示 [新增] 按鈕和 [大型檔案]](media/logic-apps-enterprise-integration-schemas/schema-largefile.png)

如果 blob 安全性存取層級為**無匿名存取**，請遵循下列步驟。

![Azure 儲存體總管，已反白顯示 [Blob 容器]、[安全性] 和 [無匿名存取]](media/logic-apps-enterprise-integration-schemas/blob-1.png)

1. 將結構描述上傳至儲存體帳戶。

    ![儲存體帳戶](media/logic-apps-enterprise-integration-schemas/blob-3.png)

2. 產生結構描述的共用存取簽章。

    ![儲存體帳戶，已反白顯示 [共用存取簽章] 索引標籤](media/logic-apps-enterprise-integration-schemas/blob-2.png)

3. 在 [新增結構描述] 中選取 [大型檔案]，然後在 [內容 URI] 文字方塊中提供共用存取簽章 URI。

    ![[結構描述]，已反白顯示 [新增] 按鈕和 [大型檔案]](media/logic-apps-enterprise-integration-schemas/schema-largefile.png)

4. 在整合帳戶的 [結構描述] 刀鋒視窗中，您現在應該會看到新增的結構描述。

    ![整合帳戶，已反白顯示 [結構描述] 和新的結構描述](media/logic-apps-enterprise-integration-schemas/schema-41.png)

## <a name="edit-schemas"></a>編輯結構描述

1. 選擇 [結構描述] 圖格。

2. 在 [結構描述] 刀鋒視窗開啟之後，選取您要編輯的結構描述。

3. 在 [結構描述] 刀鋒視窗上，選擇 [編輯]。

    ![[結構描述] 刀鋒視窗](media/logic-apps-enterprise-integration-schemas/edit-12.png)

4. 選取您要編輯的結構描述檔案，然後選取 [開啟]。

    ![開啟要編輯的結構描述檔案](media/logic-apps-enterprise-integration-schemas/edit-31.png)

Azure 會顯示訊息，說明結構描述已順利上傳。

## <a name="delete-schemas"></a>刪除結構描述

1. 選擇 [結構描述] 圖格。

2. 在 [結構描述] 刀鋒視窗開啟之後，選取您要刪除的結構描述。

3. 在 [結構描述] 刀鋒視窗上，選擇 [刪除]。

    ![[結構描述] 刀鋒視窗](media/logic-apps-enterprise-integration-schemas/delete-12.png)

4. 若要確認您要刪除選取的結構描述，請選擇 [是]。

    ![[刪除結構描述] 確認訊息](media/logic-apps-enterprise-integration-schemas/delete-21.png)

    在 [結構描述] 刀鋒視窗中，結構描述清單會重新整理，而且已不再包含您刪除的結構描述。

    ![您的整合帳戶，已反白顯示 [結構描述]](media/logic-apps-enterprise-integration-schemas/delete-31.png)

## <a name="next-steps"></a>後續步驟
* [深入了解企業整合套件](logic-apps-enterprise-integration-overview.md "了解企業整合套件")。  

