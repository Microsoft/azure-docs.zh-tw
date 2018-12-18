---
title: 在 Azure Stack 中委派供應項目 | Microsoft Docs
description: 了解如何讓其他人負責建立供應項目和為您註冊使用者。
services: azure-stack
documentationcenter: ''
author: brenduns
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/07/2018
ms.author: brenduns
ms.reviewer: alfredop
ms.openlocfilehash: 112586d3ee5f49eab9adb72d41a210e2dd9828d8
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "41946513"
---
# <a name="delegate-offers-in-azure-stack"></a>在 Azure Stack 中委派供應項目

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

身為 Azure Stack 操作員，您常常會想要讓其他人負責註冊使用者和建立訂用帳戶。 例如，如果您是服務提供者，或許需要請轉銷商註冊客戶並代表您管理客戶。 或者，如果您是企業內中央 IT 群組的一員，則可將使用者註冊委派給其他 IT 人員。

與您親力親為相比，委派可讓您更輕鬆地接觸和管理更多的使用者，如下圖所示。 

![委派層級](media/azure-stack-delegated-provider/image1.png)

藉由委派，委派的提供者便可管理供應項目 (委派的供應項目)，而客戶則可在該供應項目下取得訂用帳戶，無須系統管理員介入。 

## <a name="understand-delegation-roles-and-steps"></a>了解委派角色和步驟

### <a name="delegation-roles"></a>委派角色

下列角色是委派的一部分：

* 「Azure Stack 操作員」會管理 Azure Stack 基礎結構及建立供應項目範本。 此操作員可委派他人將供應項目提供給其租用戶。

* 委派的 Azure Stack 操作員是在名為「委派的提供者」的訂用帳戶中具備「擁有者」或「參與者」權限的使用者。 他們可以隸屬於其他組織，例如其他 Azure Active Directory (Azure AD) 租用戶。

* 「使用者」會註冊使用供應項目，並用它們來管理其工作負載、建立 VM、儲存資料等等。

### <a name="delegation-steps"></a>委派步驟

設定委派有兩個基本步驟：

1. 藉由讓使用者訂閱僅包含訂用帳戶服務的供應項目，建立委派的提供者訂用帳戶。 接著，訂閱此供應項目的使用者便可藉由為其他使用者註冊委派的供應項目，將這些供應項目延伸至其他使用者。

2. 向委派的提供者委派供應項目。 此供應項目可讓委派的提供者建立訂用帳戶，或將此供應項目延伸至其使用者。 委派的提供者現在可以取得該供應項目，並將它提供給其他使用者。

下一張圖顯示設定委派的步驟。

![建立委派的提供者，並讓它們能夠註冊使用者](media/azure-stack-delegated-provider/image2.png)

**委派的提供者需求**

若要成為委派的提供者，使用者必須藉由建立訂用帳戶來與主要提供者建立關係。 此訂用帳戶會將委派的提供者識別為有權代表主要提供者呈現委派的供應項目。

建立此關聯性之後，Azure Stack 操作員即可委派供應項目給委派的提供者。 委派的提供者可以取得供應項目、為其重新命名 (但不能變更其本質)，並提供給客戶。

## <a name="delegation-walkthrough"></a>委派逐步介紹

下列各節逐步介紹如何實際建立委派的提供者、委派供應項目，以及確認使用者可以註冊委派的供應項目。

### <a name="set-up-roles"></a>設定角色

若要使用本逐步介紹，除了 Azure Stack 操作員帳戶，您還需要兩個 Azure AD 帳戶。 如果您沒有這兩個帳戶，則需要加以建立。 這些帳戶可以屬於任何 Azure AD 使用者，而且可稱為委派的提供者和使用者。

| **角色** | **組織的權限** |
| --- | --- |
| 委派的提供者 |使用者 |
| 使用者 |使用者 |

### <a name="identify-the-delegated-provider"></a>識別委派的提供者

1. 以 Azure Stack 操作員身分登入系統管理員入口網站。

1. 若要建立可讓使用者成為委派提供者的供應項目：

   a.  [建立方案](azure-stack-create-plan.md)。
       此方案應該僅包含訂閱服務。 本文使用名為 **PlanForDelegation** 的方案作為範例。

   b.  根據此方案[建立供應項目](azure-stack-create-offer.md)。 本文使用名為 **OfferToDP** 的供應項目作為範例。

   c.  將委派的提供者新增為此供應項目的訂閱者，方法是選取 [訂閱] > [新增] > [新的租用戶訂閱]。

   ![新增委派的提供者來做為訂閱者](media/azure-stack-delegated-provider/image3.png)

   > [!NOTE]
   > 如同所有 Azure Stack 供應項目，您可以選擇讓供應項目公開並讓使用者註冊使用它，或是讓它保持私用並由 Azure Stack 操作員管理註冊。 委派的提供者通常是一個小型群組。 您會想控制可加入該群組的成員，所以在大部分情況下讓它保持私用是有意義的。

### <a name="azure-stack-operator-creates-the-delegated-offer"></a>Azure Stack 操作員建立委派的供應項目

下一個步驟是建立您即將委派以及您的使用者將要使用的方案和供應項目。 建議完全依照您想要使用者看見它的方式定義此供應項目，因為委派的提供者無法變更方案和它所包含的配額。

1. 如果您是 Azure Stack 操作員，請[建立方案](azure-stack-create-plan.md)並根據方案建立[供應項目](azure-stack-create-offer.md)。 本文使用名為 **DelegatedOffer** 的供應項目作為範例。

   > [!NOTE]
   > 此供應項目不必是公開的，但您可以視需要將其公開。 不過在大部分情況下，您只希望委派的提供者具有此供應項目的存取權。 在如下列步驟所述委派私用供應項目後，委派的提供者即擁有其存取權。

1. 委派供應項目。 移至 **DelegatedOffer**。 然後在 [設定] 之下，選取 [委派的提供者] > [新增]。

1. 從下拉式清單為委派的提供者選取訂用帳戶，然後選取 [委派]。

   ![新增委派的提供者](media/azure-stack-delegated-provider/image4.png)

### <a name="delegated-provider-customizes-the-offer"></a>委派的提供者會自訂供應項目

以委派的提供者身分登入使用者入口網站，然後使用委派的供應項目作為範本來建立新的供應項目。

1. 選取 [新增] > [租用戶供應項目 + 方案] > [供應項目]。

    ![建立新的供應項目](media/azure-stack-delegated-provider/image5.png)

1. 將名稱指派給供應項目。 本文使用 **ResellerOffer** 作為範例。 選取要做為基礎的委派供應項目，然後選取 [建立]。

   ![指派名稱](media/azure-stack-delegated-provider/image6.png)

   >[!IMPORTANT]
   >請務必了解，委派的提供者只能選擇委派給他們的供應項目。 他們無法對這些供應項目進行變更。 只有 Azure Stack 操作員能夠變更這些供應項目，例如變更其方案和配額。 委派的提供者不會從基底方案和附加方案建構供應項目。 

3. 委派的提供者可透過自己的入口網站 URL 公開這些供應項目。 若要公開供應項目，請選取 [瀏覽]，然後選取 [供應項目]。 選取供應項目，然後選取 [變更狀態]。

4. 現在只有透過委派的入口網站，才能顯示公開的已委派供應項目。 若要尋找並變更此 URL：

    a.  選取 [瀏覽] > [更多服務] > [訂用帳戶]。 然後選取「委派的提供者訂用帳戶」。 例如，[DPSubscription] > [屬性]。

    b.  將入口網站 URL 複製到不同的位置，例如：記事本。

    ![選取委派的提供者訂用帳戶](media/azure-stack-delegated-provider/dpportaluri.png)  

   您已使用委派的提供者身分建立好委派的供應項目。 以委派的提供者身分登出，並關閉您所使用的瀏覽器視窗。

### <a name="sign-up-for-the-offer"></a>註冊取得供應項目

1. 在新瀏覽器視窗中，移至您在上一個步驟中儲存的委派入口網站 URL。 以使用者身分登入入口網站。

   >[!NOTE]
   >除非您使用委派的入口網站，否則無法看見委派的供應項目。

1. 在儀表板中，選取 [取得訂用帳戶]。 您會看到只有委派的提供者所建立的委派供應項目才會呈現給使用者。

   ![檢視和選取供應項目](media/azure-stack-delegated-provider/image8.png)

委派供應項目的程序已完成。 使用者現在可以藉由取得訂用帳戶，立即註冊此供應項目。

## <a name="move-subscriptions-between-delegated-providers"></a>在委派的提供者之間移動訂用帳戶

如有需要，可在屬於相同目錄租用戶之新的或現有的委派的提供者訂用帳戶之間，移動訂用帳戶。 使用 PowerShell Cmdlet [Move-AzsSubscription](https://docs.microsoft.com/powershell/module/azs.subscriptions.admin) 即可進行此操作。

此操作在下列情況下相當有用：
- 您為將接受委派之提供者角色的新小組成員進行培訓，而想要將先前在「預設提供者訂用帳戶」中建立的使用者訂用帳戶指派給此小組成員。
- 您有多個委派的提供者訂用帳戶在相同的目錄租用戶 (Azure Active Directory) 中，而需要在它們之間移動使用者訂用帳戶。 當小組成員在小組之間移動，而需要將其訂用帳戶配置到新小組時，便可能是這種情況。


## <a name="next-steps"></a>後續步驟

[佈建 VM](azure-stack-provision-vm.md)