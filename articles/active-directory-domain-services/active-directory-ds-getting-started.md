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
ms.openlocfilehash: b6651c038a2b3abd15b8b0587e6a0e95832401b1
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39502306"
---
# <a name="enable-azure-active-directory-domain-services-using-the-azure-portal"></a>使用 Azure 入口網站啟用 Azure Active Directory Domain Services
本文說明如何使用 Azure 入口網站啟用 Azure Active Directory Domain Services (Azure AD DS)。


## <a name="before-you-begin"></a>開始之前
若要完成本文中所列的工作，您需要：

* 有效的 **Azure 訂用帳戶**。
* **Azure AD 目錄** - 與內部部署目錄或僅限雲端的目錄同步處理。
* **Azure 訂用帳戶必須與 Azure AD 目錄相關聯**。
* 您必須擁有 Azure AD 目錄的**全域管理員**權限，才能啟用 Azure AD Domain Services。


## <a name="enable-azure-ad-domain-services"></a>啟用 Azure AD 網域服務

若要啟動 [啟用 Azure AD Domain Services] 精靈，請完成下列步驟：

1. 移至 [Azure 入口網站](https://portal.azure.com)。
2. 在左窗格中，按一下 [建立資源]。
3. 在 [新增] 頁面中，將「網域服務」輸入到搜尋列。

    ![搜尋網域服務](./media/getting-started/search-domain-services.png)

4. 按一下以從搜尋建議清單中選取 [Azure AD Domain Services]。 在 [Azure AD Domain Services] 頁面中，按一下 [建立] 按鈕。

    ![網域服務檢視](./media/getting-started/domain-services-blade.png)

5. [啟用 Azure AD Domain Services] 精靈隨即啟動。


## <a name="task-1-configure-basic-settings"></a>工作 1：設定基本設定
在精靈的 [基本] 分頁中，指定受控網域的 DNS 網域名稱。 您也可以選擇應該部署受控網域的資源群組和 Azure 位置。

![設定基本](./media/getting-started/domain-services-blade-basics.png)

1. 為受控網域選擇 [DNS 網域名稱]。

   > [!NOTE]
   > **用於選取 DNS 網域名稱的指導方針**
   > * **內建網域名稱：** 根據預設，精靈會為您指定目錄的預設/內建網域名稱 (使用 **.onmicrosoft.com** 尾碼)。 如果您選擇透過網際網路啟用受控之網域的安全 LDAP 存取，則在從這個網域名稱的公用 CA 建立公用 DNS 記錄或取得安全 LDAP 憑證時應該會發生問題。 Microsoft 擁有 .onmicrosoft.com 網域，因此 CA 不會為這個網域發行憑證保證。
   * **自訂網域名稱：** 您也可以輸入自訂網域名稱。 在此範例中，自訂網域名稱是 contoso100.com。
   * **網域尾碼：** 我們一般會建議您避免使用不可路由的網域名稱尾碼。 例如，最好避免使用 DNS 網域名稱 'contoso.local' 來建立網域。 '.local' DNS 尾碼是不可路由的，因此會導致 DNS 解析發生問題。
   * **網域前置詞限制：** 指定網域名稱的前置詞 (例如，contoso100.com 網域名稱中的 contoso100) 必須包含 15 個以內的字元。 您無法使用超過 15 個字元的前置詞來建立受控網域。
   * **網路名稱衝突：** 確定虛擬網路中還沒有您為受控網域選擇的 DNS 網域名稱。 具體來說，請檢查是否有下列情況︰
       * 您在虛擬網路上已有包含相同 DNS 網域名稱的 Active Directory 網域。
       * 您打算啟用受控網域的虛擬網路具有與內部部署網路的 VPN 連線。 在此案例中，確定您在內部部署網路上沒有使用相同 DNS 網域名稱的網域。
       * 您在虛擬網路上已有具有該名稱的雲端服務。
    >

2. 選取您要在其中建立受控網域的 Azure **訂用帳戶**。

3. 選取受控網域應該隸屬的**資源群組**。 選擇 [新建] 或 [使用現有] 選項，以選取資源群組。

4. 選擇應該在其中建立受控網域的 Azure**位置**。 在精靈的 [網路] 分頁上，您只會看到屬於您所選取之位置的虛擬網路。

5. 按一下 [確定] 以繼續前往精靈的 [網路] 頁面。


## <a name="next-step"></a>後續步驟
[工作 2：設定網路基本設定](active-directory-ds-getting-started-network.md)
