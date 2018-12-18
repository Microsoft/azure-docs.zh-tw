---
title: 在 Azure Stack 中訂閱供應項目 | Microsoft Docs
description: 在 Azure Stack 中建立供應項目的訂用帳戶
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: 7f3f8683-ef09-4838-92ed-41f2fddbbbed
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 09/05/2018
ms.author: sethm
ms.openlocfilehash: b6739a194f6374cf90e6508f4a4316892daaf3dd
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49079237"
---
# <a name="create-subscriptions-to-offers-in-azure-stack"></a>在 Azure Stack 中建立供應項目的訂用帳戶

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

當您[建立供應項目](azure-stack-create-offer.md)後，使用者必須要有此供應項目的訂用帳戶才能開始使用。 有兩種方式可讓使用者訂閱供應項目：

- 若是身為雲端操作員，您可以從系統管理員入口網站為使用者建立訂用帳戶。 您建立的訂用帳戶可用於公用和私用供應項目。
- 若是身為租用戶使用者，您可以在利用使用者入口網站時訂閱公用供應項目。  

## <a name="create-a-subscription-as-a-cloud-operator"></a>以雲端操作員的身分建立訂用帳戶

雲端操作員可使用系統管理員入口網站，來為使用者建立供應項目的訂用帳戶。  您可以為自己的目錄租用戶成員建立訂用帳戶。  若已啟用[多租用戶](azure-stack-enable-multitenancy.md)，您也可以為其他目錄租用戶中的使用者建立訂用帳戶。

如果不想讓租用戶建立自己的訂用帳戶，請將供應項目設為私用，然後為租用戶建立訂用帳戶。 此方法常用於將 Azure Stack 與外部帳務系統或服務類別目錄系統整合時。

為使用者建立訂用帳戶之後，該使用者就可以登入使用者入口網站，並看到其已訂閱供應項目。  

### <a name="to-create-a-subscription-for-a-user"></a>為使用者建立訂用帳戶

1. 在系統管理員入口網站中，移至**使用者訂用帳戶。**
2. 選取 [新增] 。 在 [新增使用者訂用帳戶] 之下，輸入下列資訊：  

   - **顯示名稱** – 用於識別訂用帳戶的易記名稱，這會顯示為「使用者訂用帳戶名稱」。
   - **使用者** – 從適用此訂用帳戶的可用目錄租用戶中指定使用者。 使用者名稱會顯示為「擁有者」。  使用者名稱的格式取決於您的身分識別解決方案。 例如︰

     - **Azure AD：** `<user1>@<contoso.onmicrosoft.com>`

     - **AD FS：** `<user1>@<azurestack.local>` 

   - **目錄租用戶** – 選取使用者帳戶所屬的目錄租用戶。 如果您未啟用多租用戶，則只有您的本機目錄租用戶可使用。

3. 選取 [供應項目]。 在 [供應項目] 底下，選擇此訂用帳戶的 [供應項目]。 因為您要為使用者建立訂用帳戶，所以請選取 [私用] 作為可存取性狀態。

4. 選取 [建立] 以建立訂用帳戶。 您會在 [使用者訂用帳戶] 下看到新的訂用帳戶。 當使用者登入使用者入口網站時，他們就會看到訂用帳戶的詳細資料。

### <a name="to-make-an-add-on-plan-available"></a>提供附加方案

雲端操作員可以隨時將附加方案新增至先前建立的訂用帳戶：

1. 在管理員入口網站中，選取 [所有服務]，然後在 [系統管理資源] 類別下，選取 [使用者訂用帳戶]。 選取您想變更的訂用帳戶。

2. 選取 [附加方案]，然後選取 [+新增]。  

3. 在 [新增方案] 下，選取您想要作為附加方案的方案。

## <a name="create-a-subscription-as-a-user"></a>以使用者身分建立訂用帳戶

身為使用者，您可以登入使用者入口網站，以找到目錄租用戶 (組織) 的公用供應項目和附加方案，並加以訂閱。

>[!NOTE]
>如果 Azure Stack 環境支援[多租用戶](azure-stack-enable-multitenancy.md)，則也可以從遠端目錄租用戶訂閱供應項目。

### <a name="to-subscribe-to-an-offer"></a>訂閱供應項目

1. [登入](azure-stack-connect-azure-stack.md) Azure Stack 使用者入口網站 ( https://portal.local.azurestack.external) ，然後選取 [取得訂用帳戶]。

   ![取得訂用帳戶](media/azure-stack-subscribe-plan-provision-vm/image01.png)
  
2. 在 [取得訂用帳戶] 下，於 [顯示名稱] 中輸入訂用帳戶的易記名稱。 選取 [供應項目]，然後在 [選擇供應項目] 下挑選供應項目。 選取 [建立] 以建立訂用帳戶。

   ![建立供應項目](media/azure-stack-subscribe-plan-provision-vm/image02.png)
  
3. 訂閱供應項目之後，請重新整理入口網站以查看哪些服務是新訂用帳戶的一部分。
4. 若要查看您建立的訂用帳戶，請選取 [所有服務]，然後選取 [一般] 類別之下的 [訂用帳戶]。 選取訂用帳戶以查看訂用帳戶的詳細資料。  

### <a name="to-subscribe-to-an-add-on-plan"></a>訂閱附加方案

如果供應項目有附加方案，您可以隨時將該方案新增至您的訂用帳戶。  

1. 在使用者入口網站中，選取 [所有服務]。 接著，選取 [一般] 類別之下的 [訂用帳戶]，然後選取您想要變更的訂用帳戶。 如果有任何可用的附加方案，[+新增方案] 就會可供使用，並且會有 [附加方案] 的圖格。

   >[!NOTE]
   >如果無法使用 [+新增方案]，則表示與該訂用帳戶相關聯的供應項目沒有任何附加方案。

1. 選取 [+新增方案] 或 [附加方案] 圖格。 在 [附加方案] 下，選取您想要新增的方案。

## <a name="next-steps"></a>後續步驟

[佈建虛擬機器](azure-stack-provision-vm.md)
