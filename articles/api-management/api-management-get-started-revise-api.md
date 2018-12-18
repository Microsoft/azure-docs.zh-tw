---
title: 在 Azure API 管理中使用修訂安全地進行不間斷的變更 | Microsoft Docs
description: 依照本教學課程的步驟以了解如何使用 API 管理中的修訂進行不間斷的變更。
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
ms.date: 11/19/2017
ms.author: apimpm
ms.openlocfilehash: 864c269da61bcea885249021a44fe0259ca83e90
ms.sourcegitcommit: d98d99567d0383bb8d7cbe2d767ec15ebf2daeb2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/10/2018
ms.locfileid: "33935159"
---
# <a name="use-revisions-to-make-non-breaking-changes-safely"></a>使用修訂項目安全地進行不間斷的變更
當您的 API 已準備就緒，而且開發人員也已準備好使用時，對該 API 的變更通常需要特別小心，同時避免中斷 API 的呼叫者作業。 這對於讓開發人員了解您所做的更改也相當有用。 我們可以使用**修訂**在 Azure API 管理中進行此動作。 如需詳細資訊，請參閱[版本與修訂](https://blogs.msdn.microsoft.com/apimanagement/2017/09/14/versions-revisions/)和[使用 Azure API 管理的 API 版本設定](https://blogs.msdn.microsoft.com/apimanagement/2017/09/13/api-versioning-with-azure-api-management/)。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 新增修訂
> * 對您的修訂進行不間斷變更
> * 請讓您的修訂為最新版本，並新增變更記錄項目
> * 瀏覽開發人員入口網站，以查看變更與變更記錄檔

![開發人員入口網站上的變更記錄檔](media/api-management-getstarted-revise-api/azure_portal.PNG)

## <a name="prerequisites"></a>先決條件

+ 完成下列快速入門：[建立 Azure API 管理執行個體](get-started-create-service-instance.md)。
+ 同時也請完成下列教學課程：[匯入和發佈您的第一個 API](import-and-publish.md)。

## <a name="add-a-new-revision"></a>新增修訂

1. 選取 [API] 頁面。
2. 從 API 清單選取 [會議 API] \(或其他您要新增修訂的 API)。
3. 從頁面頂端附近的功能表按一下 [修訂] 索引標籤。
4. 選取 [+ 新增修訂]

    > [!TIP]
    > 您也可以在 API 上選擇操作功能表 (**...**) 中的 [新增修訂]。
    
    ![畫面頂端附近的修訂功能表](media/api-management-getstarted-revise-api/TopMenu.PNG)

5. 提供新修訂的描述，以協助您記住其用途。
6. 選取 [建立] 
7. 現在已建立新的修訂。

    > [!NOTE]
    > 原始的 API 會保留在**修訂 1**。 這是您的使用者將繼續呼叫的修訂，直到您選擇進行其他修訂為止。

## <a name="make-non-breaking-changes-to-your-revision"></a>對您的修訂進行不間斷變更

1. 從 API 清單選取 [會議 API]。
2. 選取畫面頂端附近的 [設計] 索引標籤。
3. 請注意，**修訂選取器** (在 [設計] 索引標籤的正上方) 會顯示您目前的修訂為**修訂 2**。

    > [!TIP]
    > 使用修訂選取器在您要進行的修訂之間切換。

4. 選取 [+ 新增作業]。
5. 將您的新作業設為 **POST**，並將作業的「名稱與顯示名稱」設為**測試**
6. **儲存**您的新作業。
7. 現在，我們已對**修訂 2** 進行變更。 使用頁面頂端附近的 [修訂選取器]，即可切換回**修訂 1**。
8. 請注意，新的作業不會出現在**修訂 1**。 

## <a name="make-your-revision-current-and-add-a-change-log-entry"></a>請讓您的修訂為最新版本，並新增變更記錄項目
1. 從頁面頂端附近的功能表選取 [修訂] 索引標籤。

    ![修訂畫面上的 [修訂] 功能表。](media/api-management-getstarted-revise-api/RevisionsMenu.PNG)
1. 針對**修訂 2** 開啟操作功能表 (**...**)。
2. 選取 [設為最新]。 如果您想要發佈這項變更的相關注意事項，請核取 [發佈到此 API 的公用變更記錄檔]。
3. 選取 [發佈到此 API 的公用變更記錄檔]
4. 提供開發人員查看的變更說明，例如**測試修訂。已新增「測試」作業。**
5. **修訂 2** 目前為最新狀態。

## <a name="browse-the-developer-portal-to-see-changes-and-change-log"></a>瀏覽開發人員入口網站，以查看變更與變更記錄檔
1. 在 Azure 入口網站中，選取 [API]
2. 從頂端功能表選取 [開發人員入口網站]。
3. 選取 [API]，然後選取 [會議 API]。
4. 請注意，您現在可以使用新的**測試**作業。
5. 從 API 名稱下方選取 [API 變更記錄]。
6. 請注意，您的變更記錄項目會出現在此清單中。

    ![開發人員入口網站](media/api-management-getstarted-revise-api/developer_portal.PNG)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 新增修訂
> * 對您的修訂進行不間斷變更
> * 請讓您的修訂為最新版本，並新增變更記錄項目
> * 瀏覽開發人員入口網站，以查看變更與變更記錄檔

前進到下一個教學課程：

> [!div class="nextstepaction"]
> [為您的 API 發佈多個版本](api-management-get-started-publish-versions.md)