---
title: 建立 Azure Marketplace 供應項目的非技術性必要條件 | Microsoft Docs
description: 了解建立和部署供應項目到 Azure Marketplace 供他人購買的要求。
services: marketplace-publishing
documentationcenter: ''
author: HannibalSII
manager: hascipio
editor: ''
ms.assetid: 3dae463b-8f48-4f52-8fa8-4e3975f09f43
ms.service: marketplace
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: Azure
ms.workload: na
ms.date: 08/18/2016
ms.author: hascipio
ms.openlocfilehash: 4f86d444a2f2b97fd8605d480db358813bc39fd3
ms.sourcegitcommit: d16b7d22dddef6da8b6cfdf412b1a668ab436c1f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39714184"
---
# <a name="general-prerequisites-for-creating-an-offer-for-the-azure-marketplace"></a>建立 Azure Marketplace 供應項目的一般必要條件
了解推動供應項目建立程序所需的一般商務程序必要條件。

## <a name="ensure-that-you-are-registered-as-a-seller-with-microsoft"></a>請確定您已經以賣方身分向 Microsoft 註冊
如需向 Microsoft 註冊賣方帳戶的詳細指示，請前往 [帳戶建立和註冊](marketplace-publishing-accounts-creation-registration.md)。

* **如果貴公司已在開發人員中心註冊為賣方，而您想建立新的供應項目** ，請使用在開發人員中心註冊所用的電子郵件識別碼登入發佈入口網站。 這是讓開發人員中心和發佈入口網站彼此連結的必要步驟。
* **如果貴公司已在開發人員中心註冊為賣方，而您想編輯現有的供應項目** ，請使用管理員帳戶或使用已新增為發佈入口網站共同管理員的帳戶登入發佈入口網站。 以下是新增共同管理員帳戶的步驟。

## <a name="steps-to-add-a-co-admin-in-the-publishing-portal"></a>在發佈入口網站新增共同管理員的步驟
發佈入口網站的管理員可以將其他開發應用程式的公司成員新增為發佈入口網站中的共同管理員。 **假設您是管理員** ，以下是新增共同管理員的步驟。

> [!NOTE]
> 若為新使用者，在您於發佈入口網站新增共同管理員之前，請確定您已在發佈入口網站建立至少一個應用程式。 這是必要步驟，因為只有在發佈入口網站建立至少一個應用程式後，[發行者]  索引標籤才會顯示。
> 
> 

1. 請確定共同管理員電子郵件識別碼是 Microsoft 帳戶 (MSA)。 如果不是，請使用此 [連結](https://signup.live.com/signup?uaid=0089f09ccae94043a0f07c2aaf928831&lic=1)將它註冊為 MSA。
2. 嘗試新增共同管理員之前，請確定管理員帳戶下至少有一個應用程式。
3. 上述步驟完成之後，請以共同管理員電子郵件識別碼登入再登出發佈入口網站。
4. 現在請使用管理員電子郵件識別碼登入發佈入口網站。
5. 瀏覽至 [發行者] -> [選取您的帳戶] -> [管理員] -> [新增共同管理員] \(如以下螢幕擷取畫面所示)
   
    ![繪圖](media/marketplace-publishing-pre-requisites/imgAddAdmin_05.png)
6. 請確定在發佈程序的各階段 (例如開發人員中心、發佈入口網站) 提供的電子郵件識別碼，在與 Microsoft 進行通訊時會受到監視。
7. 在開發人員中心註冊時，請避免使用與單一人員相關聯的帳戶。 為了不受限於個人使用，我們會建議您這麼做。
8. 如果您遇到和開發人員中心註冊相關的任何問題，請使用此[連結](https://developer.microsoft.com/en-us/windows/support)提出票證。

## <a name="steps-to-delete-a-co-admin-in-the-publishing-portal"></a>在發佈入口網站刪除共同管理員的步驟
**假設您是管理員** ，以下是刪除共同管理員的步驟。

1. 使用管理員電子郵件識別碼登入發佈入口網站。
2. 瀏覽至 [發行者] -> 選取您的帳戶 -> [管理員] -> [共同管理員]。
3. 按一下要刪除之共同管理員旁的 [X]  按鈕 (如下方示螢幕擷取畫面所示)。
   
    ![繪圖](media/marketplace-publishing-pre-requisites/imgDeleteAdmin_03.png)

> [!IMPORTANT]
> 如果您只要發佈免費供應項目 (或自備授權)，就不必填寫公司稅務和銀行資訊。
> 
> 您必須先完成公司註冊才能開始使用。 不過，您的公司在 Microsoft 開發人員中心帳戶中處理稅務和銀行資訊時，開發人員就可以開始著手在 [發佈入口網站](https://publish.windowsazure.com)中建立虛擬機器映像、取得認證以及在 Azure 預備環境中加以測試。 只有在將供應項目發佈到 Azure Marketplace 的最後一個步驟時，才必須完成賣方帳戶的核准程序。
> 
> 

## <a name="acquire-an-azure-pay-as-you-go-subscription"></a>取得 Azure「隨用隨付」訂用帳戶
這是您要用來建立 VM 映像和提交映像到 [Azure Marketplace](https://azure.microsoft.com/marketplace/)的訂用帳戶。 如果您還沒有訂用帳戶，請在 https://account.windowsazure.com/signup?offer=ms-azr-0003p 進行註冊。

## <a name="sell-from-countries"></a>「銷售來源」國家/地區
> [!WARNING]
> 如果要在 Azure Marketplace 上銷售服務，請務必確認您的註冊實體是其中一個經過核准的「銷售來源」國家/地區。 這項限制是基於付款和稅務理由。 我們正積極在不久的將來擴充這份國家/地區清單，所以請密切注意。 如需完整清單，請參閱 [Azure Marketplace 參與原則](http://go.microsoft.com/fwlink/?LinkID=526833)的第 1 節 b 項。
> 
> 

## <a name="next-steps"></a>後續步驟
一旦滿足非技術性的必要條件後，下一步是特定供應項目的技術性必要條件。 按一下您想要在 Azure Marketplace 建立之個別供應項目類型的文章連結。

* [VM 技術性必要條件](marketplace-publishing-vm-image-creation-prerequisites.md)
* [解決方案範本技術性必要條件](marketplace-publishing-solution-template-creation-prerequisites.md)

## <a name="see-also"></a>另請參閱
* [使用者入門：如何將供應項目發佈至 Azure Marketplace](marketplace-publishing-getting-started.md)

