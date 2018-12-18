---
title: 包含檔案
description: 包含檔案
services: service-bus-messaging
author: spelluru
ms.service: service-bus-messaging
ms.topic: include
ms.date: 07/03/2018
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 377ea6629223c365ef3d00c756929bf23ddffde5
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43702626"
---
若要開始在 Azure 中使用服務匯流排傳訊實體，您必須先使用 Azure 中的唯一名稱建立命名空間。 命名空間提供範圍容器，可在應用程式內定址服務匯流排資源。

若要建立命名空間：

1. 登入 [Azure 入口網站][Azure portal]。
2. 在入口網站的左方瀏覽窗格中，依序按一下 [+ 建立資源]、[整合] 及 [服務匯流排]。
3. 在 [建立命名空間]  對話方塊中，輸入命名空間名稱。 系統會立即檢查此名稱是否可用。
4. 確定命名空間名稱可用之後，請選擇定價層 ([基本]、[標準] 或 [進階])。 如果您想要使用[主題和訂用帳戶](../articles/service-bus-messaging/service-bus-queues-topics-subscriptions.md#topics-and-subscriptions)，請務必選擇標準或進階。 基本定價層不支援主題/訂用帳戶。
5. 在 [訂用帳戶]  欄位中，選擇要在其中建立命名空間的 Azure 訂用帳戶。
6. 在 [資源群組]  欄位中，選擇將存留命名空間的現有資源群組，或是建立新的資源群組。      
7. 在 [位置] 中，選擇應裝載您命名空間的國家或地區。
   
    ![建立命名空間][create-namespace]
8. 按一下頁面底部的 [新增] 。 此時系統會建立並啟用命名空間。 系統為帳戶提供資源時，您可能需要等幾分鐘。

### <a name="obtain-the-management-credentials"></a>取得管理認證

建立新命名空間會自動產生初始共用存取簽章 (SAS) 規則，其利用相關聯的主要和次要金鑰組，分別授與命名空間的所有層面的完全控制權。 請參閱[服務匯流排驗證與授權](../articles/service-bus-messaging/service-bus-authentication-and-authorization.md)，以深入了解取得如何建立進一步規則，其對於一般傳送者和接收者的權限限制更多。 若要複製初始規則，請遵循下列步驟： 

1. 按一下 [所有資源]，然後按一下新建立的命名空間名稱。
2. 在命名空間視窗中，按一下 [共用存取原則]。
3. 在 [共用存取原則] 畫面中，按一下 **RootManageSharedAccessKey**。
   
    ![connection-info][connection-info]
4. 在 [原則：RootManageSharedAccessKey] 視窗中，按一下 [主要連接字串] 旁邊的 [複製] 按鈕，將連接字串複製到剪貼簿以供後續使用。 將此值貼到記事本或一些其他暫存位置。
   
    ![connection-string][connection-string]

5. 重複前一個步驟，複製 [主要金鑰] 的值並貼到暫存位置以供稍後使用。

<!--Image references-->

[create-namespace]: ./media/service-bus-create-namespace-portal/create-namespace.png
[connection-info]: ./media/service-bus-create-namespace-portal/connection-info.png
[connection-string]: ./media/service-bus-create-namespace-portal/connection-string.png
[Azure portal]: https://portal.azure.com
