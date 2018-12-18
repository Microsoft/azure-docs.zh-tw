---
title: 建立和註冊發佈者帳戶 | Microsoft Docs
description: 關於建立 Microsoft 開發人員帳戶的指示，經過核准後即可在 Azure Marketplace 上銷售各種供應項目類型。
services: Azure Marketplace
documentationcenter: ''
author: HannibalSII
manager: hascipio
editor: ''
ms.assetid: 5a2fe68d-2967-463f-8af6-42bed07e3eaa
ms.service: marketplace
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/04/2017
ms.author: hascipio
ms.openlocfilehash: 24dc50a6c78769877bdc07b9521a9d2e33837937
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45579610"
---
# <a name="create-a-microsoft-developer-account"></a>建立 Microsoft 開發人員帳戶
本文將逐步引導您完成要成為 Azure Marketplace 核准的 Microsoft 開發人員所必須執行的帳戶建立和註冊程序。

## <a name="1-create-a-microsoft-account"></a>1.建立 Microsoft 帳戶
若要開始發佈程序，您必須建立一個 Microsoft 帳戶。 此帳戶將用來註冊 **Microsoft 開發人員中心**和 **Azure 發行入口網站**。 您所有的 Azure Marketplace 供應項目應該只會共有一個 Microsoft 帳戶。 其不應該特別屬於服務或供應項目。

構成使用者名稱的電子郵件地址應該位於您的網域中，並由您的 IT 小組所控制。 所有和發佈相關的活動都應透過此帳戶完成。

> [!WARNING]
> Microsoft 帳戶註冊不支援使用 **Azure** 和 **Microsoft** 之類的單字。 請避免使用這些字，以完成帳戶建立及註冊程序。
>
>

### <a name="guidelines-for-company-accounts"></a>公司帳戶的指導方針
建立公司帳戶時，如果有多人需要以開啟該帳戶的 Microsoft 帳戶進行登入來存取該帳戶，請遵循下列指導方針。

> [!Important]
> 重要事項：若要允許多個使用者存取您的開發人員中心帳戶，建議使用 Azure Active Directory 將角色指派給個別使用者，該使用者可使用其個別 Azure AD 認證進行登入以存取該帳戶。 如需詳細資訊，請參閱[管理帳戶使用者](https://msdn.microsoft.com/windows/uwp/publish/manage-account-users)。

* 使用屬於貴公司網域，但不屬於單一個人的電子郵件地址，建立您的 Microsoft 帳戶 — 例如，windowsapps@fabrikam.com。
* 將此 Microsoft 帳戶的存取限制為可能最少的開發人員。
* 設定公司電子郵件通訊群組清單，其中包含每一位需要存取開發人員帳戶的使用者，並將此電子郵件地址新增至您的安全性資訊。 這可讓清單上的所有員工在必要時接收安全碼，以及管理您的 Microsoft 帳戶安全性資訊。 如果設定通訊群組清單不可行，則個人電子郵件帳戶的擁有者必須能在出現提示時存取及共用安全碼 (例如將新的安全性資訊新增至帳戶時，或是必須從新裝置存取安全性資訊時)。
* 新增重要小組成員可以存取的公司電話號碼 (不需要分機號碼)。
* 一般而言，讓程式開發人員使用受信任的裝置來登入公司的開發人員帳戶。 所有重要小組成員應該可以存取這些受信任的裝置。 這會降低存取帳戶時傳送安全碼的需求。
* 如果您需要允許從非信任的電腦帳戶存取，請將該存取限制為最多五個開發人員。 在理想情況下，這些開發人員應該從共用相同地理和網路位置的電腦存取該帳戶。
* 在 [https://account.live.com/proofs/Manage](https://account.live.com/proofs/Manage) 經常檢閱貴公司的安全性資訊，以確保全都是最新資訊。

您的開發人員帳戶主要應從受信任的電腦存取。 這點很重要，因為每個帳戶每週產生的安全碼數目有所限制。 此外，也可提供最順暢的登入體驗。

如需其他開發人員帳戶指導方針和安全性的詳細資訊，請按一下[這裡](https://msdn.microsoft.com/windows/uwp/publish/opening-a-developer-account#additional-guidelines-for-company-accounts)。

### <a name="instructions"></a>範例的指示
1. 開啟新的 Chrome Incognito 或 Internet Explorer InPrivate 瀏覽工作階段，確定您未登入現有帳戶。
2. 使用 [https://signup.live.com/signup.aspx](https://signup.live.com/signup.aspx) 連結，將電子郵件 (如上述指導方針的 windowsapp@fabrikam.com) 註冊為 Microsoft 帳戶。 請遵循下列指示：

   1. 將您的帳戶註冊為 Microsoft 帳戶時，您必須提供有效的電話號碼，讓系統能以簡訊或自動語音傳送帳戶驗證碼。
   2. 將您的帳戶註冊為 Microsoft 帳戶時，您必須提供有效的電子郵件識別碼來接收進行帳戶驗證的自動發送電子郵件。
3. 請確認傳送至 DL 的電子郵件地址。
4. 您現在可以在「Microsoft 開發人員中心」中開始使用新的 Microsoft 帳戶。

## <a name="2-register-your-account-in-microsoft-developer-center"></a>2.在 Microsoft 開發人員中心註冊您的帳戶
Microsoft 開發人員中心用於註冊一次公司資訊。 註冊者必須是公司的有效代表，而且必須提供個人資訊用來驗證其身分識別。 註冊的人員必須使用公司內共用的 Microsoft 帳戶， **而且 Azure 發佈入口網站中必須使用相同的帳戶。** 當您嘗試建立帳戶之前，應該先檢查公司是否確實未曾擁有 Microsoft 開發人員中心帳戶。 在這個程序期間，我們會收集公司地址資訊、銀行帳戶資訊和稅務資訊。 這些資訊通常可以從金融或商務連絡人處取得。

> [!IMPORTANT]
> 您必須完成下列開發人員設定檔元件，才能進行供應項目建立和部署程序的各個階段。
>
>

| 開發人員設定檔 | 開始草擬 | 預備 | 免費發佈與解決方案範本 | 商業發佈 |
| --- | --- | --- | --- | --- |
| 公司註冊 |必備 |必備 |必備 |必備 |
| 稅務設定檔識別碼 |選用 |選用 |選用 |必備 |
| 銀行帳戶 |選用 |選用 |選用 |必備 |

> [!NOTE]
> 虛擬機器僅支援自備授權 (BYOL)，並將它視為 **免費** 供應項目。
>
>

### <a name="register-your-company-account"></a>註冊您的公司帳戶
1. 開啟新的 Internet Explorer InPrivate 或 Chrome Incognito 瀏覽工作階段，確定您未登入個人帳戶。
2. 移至 [http://dev.windows.com/registration?accountprogram=azure](http://dev.windows.com/registration?accountprogram=azure) 以將您自己註冊為開發人員中心的賣方。 在您繼續之前，請先閱讀下列重要事項。

   > [!IMPORTANT]
   > 確定將在開發人員中心註冊的電子郵件識別碼或通訊群組清單 (為了不受限於個人使用，建議使用通訊群組清單) 已註冊為 Microsoft 帳戶。 如果還未註冊為 Microsoft 帳戶，請使用此 [連結](https://signup.live.com/signup?uaid=e479342fe2824efeb0c3d92c8f961fd3&lic=1)註冊。 此外，**不得使用隸屬於 Microsoft 公司網域 (亦即 @microsoft.com) 的任何電子郵件識別碼**在開發人員中心註冊。
   >
   >

    ![繪圖][img-signin]
3. 請完成 [協助我們保護您的帳戶] 精靈，它會透過電話號碼或電子郵件地址驗證您的身分識別。

    ![繪圖][img-verify]
4. 在 [註冊帳戶資訊] 區段中，從下拉式功能表選取您的 [帳戶國家/區域]  。

    ![繪圖](media/marketplace-publishing-accounts-creation-registration/imgRegisterCo_04.png)

   > [!WARNING]
   > **「銷售來源」國家/地區：** 如果要在 Azure Marketplace 上銷售服務，您的註冊實體必須是其中一個經過核准的「銷售來源」國家/地區。 這項限制是基於付款和稅務理由。 我們正積極在不久的將來擴充這份國家/地區清單，所以請密切注意。 如需詳細資訊，請參閱 [Marketplace 參與原則](http://go.microsoft.com/fwlink/?LinkID=526833)。
   >
   >
5. 在 [帳戶類型] 選取 [公司]，然後按一下 [下一步] 按鈕。

   > [!IMPORTANT]
   > 若要進一步了解帳戶類型，以及何者最適合您的選擇，請檢視頁面 [帳戶類型、位置和費用](https://msdn.microsoft.com/library/windows/apps/jj863494.aspx)
   >
   >

    ![繪圖](media/marketplace-publishing-accounts-creation-registration/imgRegisterCo_05.png)
6. 輸入 [發佈者顯示名稱] ，通常是您公司的名稱。

   > [!TIP]
   > 您的供應項目列出後，在開發人員中心輸入的發行者顯示名稱並不會顯示在 Azure Marketplace。 但是，必須填寫這項資訊以完成註冊程序。
   >
   >
7. 輸入 **連絡資訊** 以進行帳戶驗證。

   > [!IMPORTANT]
   > 您必須提供精確的連絡資訊，因為它將用於我們的驗證程序，讓您的公司在開發人員中心獲得核准。
   >
   >
8. 輸入 **公司核准者**的連絡資訊。 公司核准者是可以驗證您有權代表您的組織在開發人員中心建立帳戶的人員。 完成時，按一下 [下一步] 以移至 [付款] 區段。

    ![繪圖](media/marketplace-publishing-accounts-creation-registration/imgRegisterCo_08.png)
9. 輸入您的付款資訊以支付您的帳戶。 如果您有涵蓋註冊成本的促銷代碼，您可以在此處輸入。 否則，請提供您的信用卡資訊 (或受支援市場的 PayPal)。 當您完成時，按一下 [下一步] 以移至 [檢閱] 畫面。

    ![繪圖](media/marketplace-publishing-accounts-creation-registration/imgRegisterCo_09.png)
10. 檢閱您的帳戶資訊，並確認所有項目都正確。 然後，閱讀並接受[Microsoft Azure Marketplace 發佈者合約](http://go.microsoft.com/fwlink/?LinkID=699560)的條款和條件。 勾選方塊以表示您已閱讀並接受這些條款。
11. 按一下 [完成]  以確認您的註冊。 我們會傳送一則確認訊息到您的電子郵件地址。
12. 如果您只打算發佈免費供應項目，按一下 [移至 Azure Marketplace 發佈入口網站]  即可跳到第 3 節， [在發佈入口網站中註冊您的帳戶](#3-register-your-account-in-the-publishing-portal)。

如果您打算發佈商業供應項目 (例如採每小時計費模式的虛擬機器供應項目)，按一下 [更新您的帳戶資訊]  ，您必須在該處填寫開發人員中心帳戶的稅務和銀行資訊。

如果您想要之後再更新稅務和銀行資訊，您可以前往本文件的下一節 (亦即第 3 節)： [在發佈入口網站中註冊您的帳戶](#3-register-your-account-in-the-publishing-portal)，之後再使用 Azure 發佈入口網站中的連結返回此處。

> [!IMPORTANT]
> 對於商業供應項目，如果未填寫稅務和銀行帳戶資訊，則您無法將供應項目推送到生產環境。
>
>

如果您稍後想要更新稅務和銀行資訊，您可以前往第 3 節 [在發佈入口網站中註冊您的帳戶](#3-register-your-account-in-the-publishing-portal)，之後再使用 Azure 發佈入口網站中的連結返回此處。

### <a name="add-tax-and-banking-information"></a>新增稅務和銀行資訊
 如果您想要發佈商業供應項目供採購，則還需要新增付款和稅務資訊，然後在開發人員中心中提交以供驗證。 如果您只要發佈免費供應項目 (或 BYOL 供應項目)，則不需要加入這項資訊。 您可以稍後再新增，但請花一些時間驗證稅務資訊。 如果您知道將會提供商業供應項目供採購，建議您儘快新增。

**銀行資訊**

1. 使用您的 Microsoft 帳戶登入 [Microsoft 開發人員中心](http://dev.windows.com/registration?accountprogram=azure) 。
2. 按一下左側功能表中的 [支付帳戶]，在 [選擇付款方式] 下，按一下 [銀行帳戶] 或 [PayPal]。

   > [!IMPORTANT]
   > 如果您商業供應項目可供客戶在 Marketplace 購買，則這是您將收到購物付款金額的帳戶。
   >
   >
3. 輸入付款資訊，然後在滿意時按一下 [儲存]  。

   > [!IMPORTANT]
   > 如果您需要更新或變更您的付款帳戶，請遵循上述的相同步驟，以新資訊取代目前的資訊。 變更您的付款帳戶會延遲您的付款最多一次付款週期。 會發生此延遲是因為我們必須確認帳戶變更，就像我們在您第一次設定付款帳戶時所做的一樣。 您仍然會在帳戶確認之後收到全額款項。當期付款週期未付的款項會新增至下一期。
   >
   >
4. 按 [下一步] 。

**稅務資訊**

1. 使用您的 Microsoft 帳戶登入 [Microsoft 開發人員中心](http://dev.windows.com/registration?accountprogram=azure) (視需要)。
2. 按一下左側功能表中的 [稅務設定檔]  。
3. 在 [設定您的稅單]  頁面上，選取您的永久居住國家或區域，並選取您擁有主要國籍的國家或區域。 按 [下一步] 。
4. 輸入稅務詳細資料，然後按 [下一步] 。

> [!WARNING]
> 如果沒有在 Microsoft 開發人員中心帳戶中填寫稅務和銀行帳戶資訊，則您無法將商業供應項目推送到生產環境。
>
>

如果在開發人員中心註冊時發生問題，請記錄支援票證，如下所示：

1. 移至支援連結 [https://developer.microsoft.com/windows/support](https://developer.microsoft.com/windows/support)
2. 在 [與我們連絡] 區段中，按一下 [提交事件] 按鈕 (如以下螢幕擷取畫面所示)

    ![繪圖](media/marketplace-publishing-accounts-creation-registration/imgAddTax_02.png)
3. 針對 [問題類型] 選擇 [開發人員中心協助]，針對 [類別] 選擇 [發行和管理應用程式]。 然後按一下 [開始撰寫電子郵件] 按鈕。

    ![繪圖](media/marketplace-publishing-accounts-creation-registration/imgAddTax_03.png)
4. 您將被導向登入頁面。 使用任一 Microsoft 帳戶登入。 如果您沒有 Microsoft 帳戶，請使用此 [連結](https://signup.live.com/signup?uaid=0089f09ccae94043a0f07c2aaf928831&lic=1)建立一個。
5. 填入問題的詳細資訊，然後按一下 [提交]  按鈕以提交票證。

    ![繪圖](media/marketplace-publishing-accounts-creation-registration/imgAddTax_05.png)

## <a name="3-register-your-account-in-the-publishing-portal"></a>3.在發佈入口網站中註冊帳戶
[發佈入口網站](http://publish.windowsazure.com) 可用來發佈和管理您的供應項目。

1. 開啟新的 Chrome Incognito 或 Internet Explorer InPrivate 瀏覽工作階段，確定您未登入個人帳戶。
2. 移至 [http://publish.windowsazure.com](http://publish.windowsazure.com)。
3. 如果您是第一次登入發佈入口網站的新使用者，您必須使用註冊開發人員中心帳戶所用的電子郵件識別碼登入。 如此一來，您的開發人員中心帳戶和發佈入口網站帳戶就會彼此連結。 您之後可以遵循下列步驟來將其他開發應用程式的公司成員新增為發佈入口網站中的共同管理員。

如果您已被新增為發佈入口網站中的共同管理員，則您可使用您的共同管理員帳戶登入。

> [!TIP]
> 如需參與原則的說明，請參閱 [Azure 網站](https://azure.microsoft.com/support/legal/marketplace/participation-policies/)。
>
>

## <a name="4-steps-to-add-a-co-admin-in-the-publishing-portal"></a>4.在發佈入口網站新增共同管理員的步驟
**假設您是管理員** ，以下是新增共同管理員的步驟。

> [!NOTE]
> **若為新使用者** ，在您於發佈入口網站新增共同管理員之前，請確定您已在發佈入口網站建立至少一個應用程式。 這是必要步驟，因為只有在發佈入口網站建立至少一個應用程式後，[發行者]  索引標籤才會顯示。
>
>

1. 請確定共同管理員電子郵件識別碼是 Microsoft 帳戶 (MSA)。 如果不是，請使用此 [連結](https://signup.live.com/signup?uaid=0089f09ccae94043a0f07c2aaf928831&lic=1)將它註冊為 MSA。
2. 嘗試新增共同管理員之前，請確定管理員帳戶下至少有一個應用程式。
3. 上述步驟完成之後，請以共同管理員電子郵件識別碼登入再登出發佈入口網站。
4. 現在請使用管理員電子郵件識別碼登入發佈入口網站。
5. 瀏覽至 [發行者] -> [選取您的帳戶] -> [管理員] -> [新增共同管理員] \(如以下螢幕擷取畫面所示)

   ![繪圖](media/marketplace-publishing-accounts-creation-registration/imgAddAdmin_05.png)

## <a name="5-steps-to-delete-a-co-admin-in-the-publishing-portal"></a>5.在發佈入口網站刪除共同管理員的步驟
**假設您是管理員** ，以下是刪除共同管理員的步驟。

1. 使用管理員電子郵件識別碼登入發佈入口網站。
2. 瀏覽至 [發行者] -> 選取您的帳戶 -> [管理員] -> [共同管理員]。
3. 按一下要刪除之共同管理員旁的 [X]  按鈕 (如下方示螢幕擷取畫面所示)。

    ![繪圖](media/marketplace-publishing-accounts-creation-registration/imgDeleteAdmin_03.png)

## <a name="next-steps"></a>後續步驟
現在您的帳戶已建立並註冊，請透過檢閱 [非技術性必要條件](marketplace-publishing-pre-requisites.md)，來確定您滿足或符合所有發佈供應項目所需的非技術性必要條件。

## <a name="see-also"></a>另請參閱
* [使用者入門：如何將供應項目發佈至 Azure Marketplace](marketplace-publishing-getting-started.md)

[img-msalive]:media/marketplace-publishing-accounts-creation-registration/creating-msa-account-msa-live.jpg
[img-email]:media/marketplace-publishing-accounts-creation-registration/creating-msa-account-msa-verifyemail.jpg
[img-sd-url]:media/marketplace-publishing-accounts-creation-registration/seller-dashboard-incognito.jpg
[img-signin]:media/marketplace-publishing-accounts-creation-registration/seller-dashboard-login.jpg
[img-verify]:media/marketplace-publishing-accounts-creation-registration/seller-dashboard-verify.jpg
[img-sd-top]:media/marketplace-publishing-accounts-creation-registration/seller-dashboard-personal-acc-details.jpg
[img-sd-info]:media/marketplace-publishing-accounts-creation-registration/seller-dashboard-personal.jpg
[img-sd-type]:media/marketplace-publishing-accounts-creation-registration/seller-dashboard-personal-acc-type.jpg
[img-sd-mktg1]:media/marketplace-publishing-accounts-creation-registration/seller-dashboard-personal-comp-det1.jpg
[img-sd-mktg2]:media/marketplace-publishing-accounts-creation-registration/seller-dashboard-personal-comp-det2.jpg
[img-sd-addr]:media/marketplace-publishing-accounts-creation-registration/seller-dashboard-personal-comp-add.jpg
[img-sd-legal]:media/marketplace-publishing-accounts-creation-registration/seller-dashboard-personal-cmp.jpg
[img-sd-submit]:media/marketplace-publishing-accounts-creation-registration/seller-dashboard-approval.jpg

[link-msdndoc]: https://msdn.microsoft.com/library/jj552460.aspx
[link-sellerdashboard]: http://sellerdashboard.microsoft.com/
[link-pubportal]: https://publish.windowsazure.com
[link-single-vm]:marketplace-publishing-vm-image-creation.md
[link-single-vm-prereq]:marketplace-publishing-vm-image-creation-prerequisites.md
[link-multi-vm]:marketplace-publishing-solution-template-creation.md
[link-multi-vm-prereq]:marketplace-publishing-solution-template-creation-prerequisites.md
[link-datasvc]:marketplace-publishing-data-service-creation.md
[link-datasvc-prereq]:marketplace-publishing-data-service-creation-prerequisites.md
[link-devsvc]:marketplace-publishing-dev-service-creation.md
[link-devsvc-prereq]:marketplace-publishing-dev-service-creation-prerequisites.md
[link-pushstaging]:marketplace-publishing-push-to-staging.md
