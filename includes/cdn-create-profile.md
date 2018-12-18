---
title: 包含檔案
description: 包含檔案
services: cdn
author: SyntaxC4
ms.service: cdn
ms.topic: include
ms.date: 05/24/2018
ms.author: cfowler
ms.custom: include file
ms.openlocfilehash: e431e7c45dc2cb41db5b9fa762abc908cc76b375
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34665277"
---
## <a name="create-a-new-cdn-profile"></a>建立新的 CDN 設定檔

CDN 設定檔是 CDN 端點的容器，會指定定價層。

1. 在 Azure 入口網站的左上方，選取 [建立資源]。 
    
    [新增] 窗格隨即出現。
   
2. 選取 [Web + 行動]，然後選取 [CDN]。
   
    ![選取 CDN 資源](./media/cdn-create-profile/cdn-new-resource.png)

    此時會顯示 [CDN 設定檔] 窗格。

3. 對於 CDN 設定檔設定，使用下表中所指定的值：
   
    | 設定  | 值 |
    | -------- | ----- |
    | **名稱** | 輸入 *my-cdn-profile-123* 作為設定檔名稱。 此名稱必須是全域唯一的；如果已在使用中，您可以輸入不同名稱。 |
    | **訂用帳戶** | 從下拉式清單中，選取 Azure 訂用帳戶。 |
    | **資源群組** | 選取 [新建]，並輸入 *my-resource-group-123* 作為資源群組名稱。 此名稱必須是全域唯一的。 如果此名稱已在使用中，您可以輸入不同名稱或是選取 [使用現有項目]，並從下拉式清單中選取 [my-resource-group-123]。 | 
    | **資源群組位置** | 從下拉式清單中選取 [美國中部]。 |
    | **定價層** | 從下拉式清單中選取 [標準 Verizon]。 |
    | **立即建立新的 CDN 端點** | 保留為未選取。 |  
   
    ![新增 CDN 設定檔](./media/cdn-create-profile/cdn-new-profile.png)

4. 選取 [釘選至儀表板]，以在設定檔建立後將其儲存至儀表板。
    
5. 選取 [建立] 以建立設定檔。 

    對於 [來自 Microsoft 的 Azure CDN 標準] 設定檔，設定檔通常會在兩個小時內完成。 

