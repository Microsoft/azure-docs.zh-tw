---
title: Azure Active Directory Domain Services：開始使用 | Microsoft Docs
description: 使用 Azure 入口網站啟用 Azure Active Directory Domain Services
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: ace1ed4a-bf7f-43c1-a64a-6b51a2202473
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/23/2018
ms.author: maheshu
ms.openlocfilehash: 2290273c1b998a2d75046fcbcf613762ddd588ee
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39503201"
---
# <a name="enable-azure-active-directory-domain-services-using-the-azure-portal"></a>使用 Azure 入口網站啟用 Azure Active Directory Domain Services


## <a name="task-3-configure-administrative-group"></a>工作 3：設定系統管理群組
在這項設定工作中，您可以在 Azure AD 目錄中建立系統管理群組。 這個特殊的系統管理群組稱為 *AAD DC 系統管理員*。 此群組的成員會獲得電腦的系統管理權限，而這類電腦已加入受控網域。 在加入網域的電腦上，這個群組會新增到系統管理員群組。 此外，此群組的成員可以使用遠端桌面，從遠端連接到已加入網域的電腦。

> [!NOTE]
> 您在使用 Azure Active Directory Domain Services 所建立的受控網域內，沒有「網域系統管理員」或「企業系統管理員」權限。 在受控網域上，服務會保留這些權限，並不會提供給租用戶中的使用者使用。 不過，您可以使用在此組態工作中建立的特殊系統管理群組，執行某些特殊權限的作業。 這些作業包括將電腦加入網域、隸屬於已加入網域之電腦上的管理群組，以及設定群組原則。
>

精靈會在 Azure AD 目錄中自動建立系統管理群組。 此群組名為「AAD DC 系統管理員」。 如果在 Azure AD 目錄中已有此名稱的現有群組存在，精靈會選取此群組。 您可以使用 [Administrator 群組] 精靈分頁設定群組成員資格。

1. 若要設定群組成員資格，請按一下 [AAD DC 系統管理員]。

    ![設定群組成員資格](./media/getting-started/domain-services-blade-admingroup.png)

2. 按一下 [新增成員] 按鈕，將使用者從 Azure AD 目錄新增至系統管理員群組。

3. 完成時，按一下 [確定] 以繼續前往精靈的 [摘要] 分頁。


## <a name="deploy-your-managed-domain"></a>部署受控網域

1. 在精靈的 [摘要] 分頁中，檢閱受控網域的組態設定。 您可以視需要返回精靈的任何步驟以進行變更。 完成時，按一下 [確定] 來建立新的受控網域。

    ![總結](./media/getting-started/domain-services-blade-summary.png)

2. 您會看到通知，顯示 Azure AD Domain Services 部署的進度。 按一下通知以查看部署的詳細進度。

    ![通知 - 部署進行中](./media/getting-started/domain-services-blade-deployment-in-progress.png)


## <a name="check-the-deployment-status-of-your-managed-domain"></a>檢查受控網域的部署狀態
佈建受控網域的程序可能需要一小時的時間。

1. 部署進行時，您可以在 [搜尋資源] 搜尋方塊中搜尋「網域服務」。 從搜尋結果選取 [Azure AD Domain Services]。 [Azure AD Domain Services] 刀鋒視窗會列出正在佈建的受控網域。

    ![找出正在佈建的受控網域](./media/getting-started/domain-services-provisioning-state-find-resource.png)

2. 按一下受控網域的名稱 (例如，'contoso100.com')，以查看受控網域的詳細資料。

    ![Domain Services - 佈建狀態](./media/getting-started/domain-services-provisioning-state.png)

3. [概觀] 索引標籤會顯示目前佈建的受控網域。 完整佈建之前，您無法設定受控網域。 完整佈建受控網域可能需要一小時的時間。

    ![Domain Services - 佈建狀態期間的概觀索引標籤 ](./media/getting-started/domain-services-provisioning-state-details.png)

4. 當受控網域完整佈建時，[概觀] 索引標籤會將網域狀態顯示為 [執行中]。

    ![Domain Services - 完全佈建後的 [概觀] 索引標籤](./media/getting-started/domain-services-provisioned.png)
    >[!NOTE]
    >在佈建過程中，Azure AD Domain Services 會在您的目錄中建立名為 "Domain Controller Services" 和 "AzureActiveDirectoryDomainControllerServices" 的企業應用程式。 處理受控網域時需要這些企業應用程式。 這些企業應用程式絕對不能刪除。
    >

5. 在 [屬性] 索引標籤上，您會看到網域控制站可供虛擬網路使用的兩個 IP 位址。

    ![Domain Services - 完整佈建後的屬性索引標籤](./media/getting-started/domain-services-provisioned-properties.png)


## <a name="need-help"></a>需要協助嗎？
佈建您的受控網域的兩個網域控制站可能需要一或兩小時的時間。 如果您的部署失敗，或停滯在「擱置中」狀態超過幾個小時，請隨意[連絡產品小組以取得協助](active-directory-ds-contact-us.md)。


## <a name="next-step"></a>後續步驟
[工作 4：更新 Azure 虛擬網路的 DNS 設定](active-directory-ds-getting-started-dns.md)
