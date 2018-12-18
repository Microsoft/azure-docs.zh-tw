---
title: 使用 Azure API 管理發行 API 版本 |Microsoft Docs
description: 依照此教學課程的步驟，了解如何在 API 管理中發行多個版本。
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.custom: mvc
ms.topic: tutorial
ms.date: 06/15/2018
ms.author: apimpm
ms.openlocfilehash: a7e5051248a579b0943fa69620215b060bd1e235
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47092688"
---
# <a name="publish-multiple-versions-of-your-api"></a>為您的 API 發佈多個版本 

有時候，讓 API 的所有呼叫者使用完全相同的版本不太實際。 當呼叫者想要升級至更新版本時，他們希望能夠使用簡單易懂的方法來執行此操作。 您可以在 Azure API 管理中使用**版本**執行此操作。 如需詳細資訊，請參閱[版本與修訂](https://blogs.msdn.microsoft.com/apimanagement/2017/09/14/versions-revisions/)。

在此教學課程中，您將了解如何：

> [!div class="checklist"]
> * 將新版本新增至現有 API
> * 選擇版本配置
> * 將版本新增至產品
> * 瀏覽開發人員入口網站以查看版本

![開發人員入口網站上顯示的版本](media/api-management-getstarted-publish-versions/azure_portal.PNG)

## <a name="prerequisites"></a>先決條件

* 完成下列快速入門：[建立 Azure API 管理執行個體](get-started-create-service-instance.md)。
* 同時也請完成下列教學課程：[匯入和發佈您的第一個 API](import-and-publish.md)。

## <a name="add-a-new-version"></a>加入新版本

![API 內容功能表 - 新增版本](media/api-management-getstarted-publish-versions/AddVersionMenu.png)

1. 從 API 清單中選取 [Demo Conference API]。
2. 選取它旁邊的快顯功能表 (**...**)。
3. 選取 [+ 新增版本]。

    > [!TIP]
    > 當您第一次建立新的 API 時，也會啟用版本 - 請在 [新增 API] 畫面中選取 [要為此 API 設定版本嗎?]。

## <a name="choose-a-versioning-scheme"></a>選擇版本設定配置

Azure API 管理可讓您選擇讓呼叫者指定他們想要之 API 版本的方式。 您可以選取 [版本設定配置] 來指定要使用的 API 版本。 這個配置可以是**路徑、標頭或查詢字串**。 下列範例使用路徑來選取版本設定配置。

![新增版本畫面](media/api-management-getstarted-publish-versions/AddVersion.PNG)

1. 讓選取的**路徑**做為您的**版本設定配置**。
2. 新增 **v1** 做為**名稱**與**版本識別碼**。

    > [!TIP]
    > 如果您選取 [標題] 或 [查詢字串] 作為版本設定配置，您需要提供其他值 - 標題名稱或查詢字串參數。

3. 選取 [建立] 以設定您的新版本。
4. 在 API 清單中的 [範例會議 API] 下，您可以看見兩個不同的 API - **原始**和 **v1**。

    ![在 Azure 入口網站中 API 下列出的版本](media/api-management-getstarted-publish-versions/VersionList.PNG)

    > [!Note]
    > 如果您新增版本至未設定版本的 API，系統一律會建立**原始**版本 - 在預設 URL 上回應。 這可確保任何現有的呼叫者不會因為新增版本的程序而中斷。 如果您在開始時建立已啟用版本的新 API，則不會建立「原始」。

5. 您現在可以編輯 **v1**，並將其設定為與**原始**不同的 API。 對某個版本進行變更不會影響另一個版本。

## <a name="add-the-version-to-a-product"></a>將版本新增至產品

為了讓呼叫端看到新的版本，您必須將該版本新增至**產品**。

1. 從傳統部署模型頁面選取 [產品]。

    ![API 管理產品](media/api-management-getstarted-publish-versions/Products.png)

2. 選取 [無限制]。
3. 選取 [API]。
4. 選取 [新增] 。
5. 選取 [示範會議 API，版本 v1]。
6. 瀏覽至服務管理頁面，然後選取 [API]。

## <a name="browse-the-developer-portal-to-see-the-version"></a>瀏覽開發人員入口網站以查看版本

1. 從頂端功能表選取 [開發人員入口網站]。
2. 選取 [API]，請注意，[示範會議 API] 會顯示**原始**和 **v1** 版本。
3. 選取 [v1]。
4. 請注意清單中第一個作業的 [要求 URL]。 它會顯示 API URL 路徑，包含 **v1**。

    ![API 內容功能表 - 新增版本](media/api-management-getstarted-publish-versions/developer_portal.png)

## <a name="next-steps"></a>後續步驟

在此教學課程中，您將了解如何：

> [!div class="checklist"]
> * 將新版本新增至現有 API
> * 選擇版本配置 
> * 將版本新增至產品
> * 瀏覽開發人員入口網站以查看版本

前進到下一個教學課程：

> [!div class="nextstepaction"]
> [自訂開發人員入口網站頁面的風格](api-management-customize-styles.md)