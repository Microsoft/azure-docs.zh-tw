---
title: 將 Azure 訂用帳戶擁有權轉移給另一個帳戶 |Microsoft 文件
description: 描述如何將 Azure 訂用帳戶轉移給另一位使用者，以及一些關於此程序的常見問題集 (FAQ)
keywords: 轉移 azure 訂用帳戶,azure 轉移訂用帳戶,將 azure 訂用帳戶移到另一個帳戶,azure 變更訂用帳戶擁有者,將 azure 訂用帳戶轉移給另一個帳戶
services: ''
documentationcenter: ''
author: genlin
manager: jlian
editor: ''
tags: billing,top-support-issue
ms.assetid: c8ecdc1e-c9c5-468c-a024-94ae41e64702
ms.service: billing
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/15/2018
ms.author: cwatson
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 7e9b5d1a84417cf6e6264211ea6e542ab712f9fe
ms.sourcegitcommit: 42405ab963df3101ee2a9b26e54240ffa689f140
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47422590"
---
# <a name="transfer-ownership-of-an-azure-subscription-to-another-account"></a>將 Azure 訂用帳戶的擁有權轉移給另一個帳戶

將您的訂用帳戶轉移給帳戶中心的另一位使用者，可變更帳戶管理員，並交出訂用帳戶的帳單擁有權。 若要將您的訂用帳戶變更至不同的供應項目，請參閱[切換至不同的 Azure 訂用帳戶供應項目](billing-how-to-switch-azure-offer.md)。

> [!IMPORTANT]
> 
> 如果您將訂用帳戶轉移至新的 Azure AD 租用戶，[角色型存取控制 (RBAC)](../role-based-access-control/overview.md) 中的所有角色指派會從來源租用戶中永久刪除，而且不會移轉至目標租用戶。

> [!div class="nextstepaction"]
> [協助改善 Azure 計費文件](https://go.microsoft.com/fwlink/p/?linkid=2010091)

## <a name="transfer-ownership-of-an-azure-subscription"></a>轉移 Azure 訂用帳戶的擁有權

> [!VIDEO https://channel9.msdn.com/Series/Microsoft-Azure-Tutorials/Transfer-an-Azure-subscription/player]
>
>

1. 以帳戶管理員身分登入 [Azure 帳戶中心](https://account.windowsazure.com/Subscriptions)。若要找出誰是訂用帳戶的帳戶管理員，請參閱[常見問題集](#faq)。

1. 選取要移轉的訂用帳戶。

1. 將 [供應項目] 和 [供應項目識別碼][與支援供應項目清單對照](#supported)，確認您的訂閱是否符合自助式移轉的資格。

   ![在帳戶中心裡確認訂閱的供應項目識別碼](./media/billing-subscription-transfer/image0.png)
1. 按一下 [移轉訂用帳戶]。

   ![Azure 帳戶的訂用帳戶索引標籤](./media/billing-subscription-transfer/image1.png)
1. 指定接受者。

   > [!IMPORTANT]
   > 
   > 如果您將訂用帳戶轉移至新的 Azure AD 租用戶，[角色型存取控制 (RBAC)](../role-based-access-control/overview.md) 中的所有角色指派會從來源租用戶中永久刪除，而且不會移轉至目標租用戶。

   ![移轉訂用帳戶對話方塊](./media/billing-subscription-transfer/image2.PNG)

1. 接受者會自動收到含有接受連結的電子郵件。

   ![給接受者的訂用帳戶移轉電子郵件](./media/billing-subscription-transfer/image3.png)
1. 接受者按一下連結並遵循指示進行，包括輸入他們的付款資訊。

   ![第一個訂用帳戶移轉網頁](./media/billing-subscription-transfer/image4.png)

   ![第二個訂用帳戶移轉網頁](./media/billing-subscription-transfer/image5.png)
1. 成功！ 現在已移轉訂用帳戶。

<a id="EA"></a>

## <a name="transfer-subscription-ownership-for-enterprise-agreement-ea-customers"></a>轉送 Enterprise 合約 (EA) 客戶的訂用帳戶擁有權

企業系統管理員可以轉送註冊內之訂用帳戶的擁有權。 若要開始使用，請參閱 EA 入口網站中的[轉送帳戶擁有權](https://ea.azure.com/helpdocs/changeAccountOwnerForASubscription)。

## <a name="next-steps-after-accepting-ownership-of-a-subscription"></a>接受訂用帳戶擁有權後的後續步驟

1. 您現在是帳戶管理員。請檢閱並更新服務管理員、共同管理員和其他 RBAC 角色。 若要深入了解，請參閱[新增或變更 Azure 訂用帳戶管理員](billing-add-change-azure-subscription-administrator.md)和[使用 RBAC 和 Azure 入口網站來管理存取權](../role-based-access-control/role-assignments-portal.md)。
1. 更新與此訂用帳戶服務相關聯的認證，包括：
   1. 可將使用者管理權限授與給訂用帳戶資源的管理憑證。 如需詳細資訊，請參閱 [建立和上傳 Azure 的管理憑證](../cloud-services/cloud-services-certs-create.md)
   1. 服務 (例如儲存體) 的存取金鑰。 如需詳細資訊，請參閱[關於 Azure 儲存體帳戶](../storage/common/storage-create-storage-account.md)
   1. 服務 (例如 Azure 虛擬機器) 的遠端存取認證。 
1. 如果您正與合作夥伴協力作業，請考慮更新此訂用帳戶的合作夥伴 ID。 您可以在 [Azure 入口網站](https://portal.azure.com)更新合作夥伴識別碼。

<a id="supported"></a>

## <a name="whats-supported"></a>支援的項目：

下表中的供應項目和訂閱類型可採用自助式訂閱移轉。 您目前無法移轉免費試用版或 [Azure in Open (AIO)](https://azure.microsoft.com/offers/ms-azr-0111p/) 訂閱。 如需因應措施，請參閱[將資源移動到新的資源群組或訂用帳戶](../azure-resource-manager/resource-group-move-resources.md)。 若要移轉[贊助](https://azure.microsoft.com/offers/ms-azr-0036p/)或支援方案等其他訂閱，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)。

| 供應項目名稱                                                                             | 供應項目號碼 |
|----------------------------------------------------------------------------------------|--------------|
| [Enterprise 合約 (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/)\*|MS-AZR-0017P        |
| [Microsoft 合作夥伴網路](https://azure.microsoft.com/offers/ms-azr-0025p/)          | MS-AZR-0025P        |
| [MSDN 平台](https://azure.microsoft.com/offers/ms-azr-0062p/)                     | MS-AZR-0062P        |
| [隨用隨付](https://azure.microsoft.com/offers/ms-azr-0003p/)                      | MS-AZR-0003P        |
| [隨用隨付開發/測試](https://azure.microsoft.com/offers/ms-azr-0023p/)             | MS-AZR-0023P        |
| [Visual Studio Enterprise](https://azure.microsoft.com/offers/ms-azr-0063p/)           | MS-AZR-0063P        |
| [Visual Studio Enterprise：BizSpark](https://azure.microsoft.com/offers/ms-azr-0064p/) | MS-AZR-0064P        |
| [Visual Studio Professional](https://azure.microsoft.com/offers/ms-azr-0059p/)         | MS-AZR-0059P        |
| [Visual Studio Test Professional](https://azure.microsoft.com/offers/ms-azr-0060p/)    | MS-AZR-0060P        |

\* [透過 EA 入口網站](#EA)

<a id="faq"></a>

## <a name="frequently-asked-questions-faq"></a>常見問題集 (FAQ)

### <a name="whoisaa"></a> 訂用帳戶的帳戶管理員是誰？

帳戶管理員就是註冊或購買 Azure 訂閱的人員。 他們已獲得授權存取[帳戶中心](https://account.azure.com/Subscriptions)並可執行各種管理工作，像是建立訂用帳戶、取消訂用帳戶、變更訂用帳戶的計費方式，或變更服務管理員。 如果您不確定誰是訂用帳戶的帳戶管理員，請使用下列步驟來找出帳戶管理員。

1. 瀏覽 [Azure 入口網站中的 [訂用帳戶] 頁面](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)。
1. 選取您想要檢查的訂用帳戶，然後查看 [設定]。
1. 選取 [屬性] 。 該訂用帳戶的帳戶管理員會顯示在 [帳戶管理員]  方塊中。

### <a name="does-everything-transfer-including-resource-groups-vms-disks-and-other-running-services"></a>所有項目都會移轉嗎？ 包括資源群組、VM、磁碟和其他執行中的服務？

VM、磁碟和網站等所有資源都會轉移給新的擁有者。 不過，不會跨不同目錄轉送您所設定的任何[系統管理員角色](billing-add-change-azure-subscription-administrator.md)和[角色型存取控制 (RBAC)](../role-based-access-control/role-assignments-portal.md) 原則。 此外，[應用程式註冊](../active-directory/develop/quickstart-v1-integrate-apps-with-azure-ad.md)和其他租用戶特定服務並不會隨之轉移。

### <a id="no-button"></a> 為什麼看不到 [移轉訂用帳戶] 按鈕？

很抱歉，您的供應項目或國家/地區無法使用自助式訂閱移轉。 若要移轉訂閱，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)。

### <a name="does-a-subscription-transfer-result-in-any-service-downtime"></a>移轉訂閱會造成服務中斷嗎？

不會影響服務。 轉送訂用帳戶會在目前帳戶管理員之下取消訂用帳戶，並在接受者的帳戶之下建立訂用帳戶。 新的訂用帳戶與基礎 Azure 服務相關聯。 訂用帳戶 ID 維持不變。

### <a name="how-do-i-use-this-process-to-change-the-directory-for-subscription"></a>如何透過此程序變更訂閱的目錄？

Azure 訂用帳戶建立在帳戶管理員所屬的目錄中。 若要變更目錄，請將訂用帳戶轉移到目標目錄中的使用者帳戶。 當使用者完成步驟接受轉移時，訂用帳戶就會自動移至目標目錄。

### <a name="if-i-take-over-billing-ownership-of-a-subscription-from-another-organization-do-they-continue-to-have-access-to-my-resources"></a>如果我取得另一個組織的訂閱帳單擁有權，他們可以繼續存取我的資源嗎？

如果訂用帳戶轉移到另一個租用戶，與先前的租用戶相關聯的使用者會失去訂用帳戶的存取權。 即使使用者不再是服務管理員或共同管理員，他們仍可能透過其他安全性機制來存取訂用帳戶，包括：

* 可將使用者管理權限授與給訂用帳戶資源的管理憑證。 如需詳細資訊，請參閱[建立和上傳 Azure 的管理憑證](../cloud-services/cloud-services-certs-create.md)。
* 服務 (例如儲存體) 的存取金鑰。 如需詳細資訊，請參閱[關於 Azure 儲存體帳戶](../storage/common/storage-create-storage-account.md)。
* 服務 (例如 Azure 虛擬機器) 的遠端存取認證。

如果接受者需要限制對其資源的存取權，則應考慮更新所有與服務相關聯的密碼。 您可以使用下列步驟來更新大部分的資源：

  1. 移至 [Azure 入口網站](https://portal.azure.com)。
  2. 在 [中樞] 功能表中，選取 [所有資源]。
  3. 選取資源。
  4. 在資源刀鋒視窗中，按一下 [設定] 。 您可以在這裡檢視並更新現有的密碼。

### <a name="if-i-transfer-the-subscription-in-the-middle-of-the-billing-cycle-does-the-recipient-pay-for-the-entire-billing-cycle"></a>如果我在計費週期中途移轉訂閱，接受者需要支付整個計費週期的費用嗎？

傳送者負責支付移轉完成當時所報告的任何使用量。 接受者負責支付移傳以後所報告的使用量。 可能有某些使用量是在移轉之前發生，但在之後才報告。 使用量會包含在接受者的帳單中。

### <a name="does-the-recipient-have-access-to-usage-and-billing-history"></a>接受者可以存取使用和帳單記錄嗎？

  接受者可用的資訊只有最新帳單的金額，而如果訂用帳戶是在產生第一份帳單之前移轉，則可使用目前餘額。 其餘的使用量及帳單記錄不會隨著訂用帳戶一起移轉。

### <a name="can-the-offer-be-changed-during-a-transfer"></a>可以在移轉期間變更供應項目嗎？

供應項目必須維持不變。 若要變更供應項目，請參閱[切換至不同的 Azure 訂用帳戶供應項目](billing-how-to-switch-azure-offer.md)。

### <a name="can-i-transfer-a-subscription-to-a-user-account-in-another-country"></a>可以將訂閱移轉給其他國家/地區的使用者帳戶嗎？

否，不支援將訂用帳戶轉移給另一個國家/地區的使用者帳戶。 接受者的使用者帳戶必須在相同的國家/地區。

### <a name="can-the-recipient-use-a-different-payment-method"></a>接受者可以使用與該訂閱不同的付款方式嗎？

是。 但訂用帳戶帳單記錄會拆成兩個帳戶。  

### <a name="is-the-payment-method-impacted-after-i-transferred-an-azure-subscription"></a>移轉 Azure 訂閱會影響付款方式嗎？

若要接受訂閱移轉，您必須提供信用卡或類似的付款方式來支付訂閱費。 例如，如果 Bob 將訂用帳戶轉移給 Jane，而 Jane 接受轉移，則 Jane 必須提供用來支付訂用帳戶的付款方法。 完成轉移之後，就會向 Jane 收取訂用帳戶的費用，而不是向 Bob 收取。

### <a name="how-do-i-migrate-data-and-services-for-my-azure-subscription-to-new-subscription"></a>如何將 Azure 訂閱的資料與服務移轉到新的訂閱？

如果您無法轉移訂用帳戶的擁有權，則可手動移轉您的資源。 請參閱[將資源移動到新的資源群組或訂用帳戶](../azure-resource-manager/resource-group-move-resources.md)。

## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員。

如果仍需要協助，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。