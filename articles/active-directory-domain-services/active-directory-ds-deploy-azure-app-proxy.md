---
title: Azure Active Directory 網域服務︰部署 Azure Active Directory 應用程式 Proxy | Microsoft Docs
description: 在 Active Directory Domain Services 受控網域上使用 Azure AD 應用程式
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: 938a5fbc-2dd1-4759-bcce-628a6e19ab9d
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/22/2018
ms.author: maheshu
ms.openlocfilehash: 11967e850e9d626edf757526b8ae7d320bad1a4e
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39504340"
---
# <a name="deploy-azure-ad-application-proxy-on-an-azure-ad-domain-services-managed-domain"></a>在 Azure Active Directory Domain Services 受控網域上部署 Azure AD 應用程式
Azure Active Directory (AD) 應用程式 Proxy 可藉由發佈要透過網際網路存取的內部部署應用程式，協助您支援遠端背景工作角色。 使用 Azure AD 網域服務，您現在可以提升執行內部部署的舊版應用程式並隨即轉移至 Azure 基礎結構服務。 然後，您可以使用 Azure AD 應用程式 Proxy 發佈這些應用程式，為您組織中的使用者提供安全遠端存取。

如果您是 Azure AD 應用程式 Proxy 的新手，可至下列文章：[如何為內部部署應用程式提供安全的遠端存取](../active-directory/manage-apps/application-proxy.md)深入了解這個功能。

[!INCLUDE [active-directory-ds-prerequisites.md](../../includes/active-directory-ds-prerequisites.md)]

## <a name="before-you-begin"></a>開始之前
若要執行本文中所列的工作，您需要︰

1. 有效的 **Azure 訂用帳戶**。
2. **Azure AD 目錄** - 與內部部署目錄或僅限雲端的目錄同步處理。
3. 需要 **Azure AD 基本或進階授權**才能使用 Azure AD 應用程式 Proxy。
4. **Azure AD 網域服務** 必須已針對 Azure AD 目錄啟用。 如果還沒有啟用，請按照 [入門指南](active-directory-ds-getting-started.md)所述的所有工作來進行。

<br>

## <a name="task-1---enable-azure-ad-application-proxy-for-your-azure-ad-directory"></a>工作 1 - 針對您的 Azure AD 目錄啟用 Azure AD 應用程式 Proxy
執行下列步驟為您的 Azure AD 目錄啟用 Azure AD 應用程式 Proxy。

1. 在 [Azure 入口網站](http://portal.azure.com)中，以系統管理員身分登入。

2. 按一下 [Azure Active Directory] 以啟動目錄概觀。 按一下 [企業應用程式]。

    ![選取 Azure AD 目錄](./media/app-proxy/app-proxy-enable-start.png)
3. 按一下 [應用程式 Proxy]。 如果您沒有 Azure AD Basic 或 Azure AD Premium 訂用帳戶，您會看到一個選項可啟用試用版。 將 [啟用應用程式 Proxy？] 切換為 [啟用]，然後按一下 [儲存]。

    ![啟用應用程式 Proxy](./media/app-proxy/app-proxy-enable-proxy-blade.png)
4. 若要下載連接器，請按一下 [連接器] 按鈕。

    ![下載連接器](./media/app-proxy/app-proxy-enabled-download-connector.png)
5. 在下載頁面上，接受授權條款和隱私權合約，並按一下 [下載] 按鈕。

    ![確認下載](./media/app-proxy/app-proxy-enabled-confirm-download.png)


## <a name="task-2---provision-domain-joined-windows-servers-to-deploy-the-azure-ad-application-proxy-connector"></a>工作 2 - 佈建已加入網域的 Windows 伺服器來部署 Azure AD 應用程式 Proxy 連接器
您需要可安裝 Azure AD 應用程式 Proxy 連接器的已加入網域 Windows Server 虛擬機器。 對於某些應用程式，您可以選擇佈建已安裝連接器的多部伺服器。 這個部署選項為您提供更高的可用性，並協助處理更大量的驗證負載。

在相同的虛擬網路 (或連線/對等互連的虛擬網路) 上佈建連接器伺服器，其中已您啟用 Azure AD 網域服務受控網域。 同樣地，裝載您透過應用程式 Proxy 發佈之應用程式的伺服器需要安裝在相同的 Azure 虛擬網路上。

若要佈建連接器伺服器，請依照名為[將 Windows 虛擬機器加入受控網域](active-directory-ds-admin-guide-join-windows-vm.md)一文所述的工作進行操作。


## <a name="task-3---install-and-register-the-azure-ad-application-proxy-connector"></a>工作 3 - 安裝與註冊 Azure AD 應用程式 Proxy 連接器
在過去，您會佈建 Windows Server 虛擬機器，並將它加入受控網域。 在這個工作中，您會在此虛擬機器上安裝 Azure AD 應用程式 Proxy 連接器。

1. 將連接器安裝套件複製到安裝 Azure AD Web 應用程式 Proxy 連接器的 VM。

2. 在虛擬機器上執行 **AADApplicationProxyConnectorInstaller.exe**。 接受軟體授權條款。

    ![接受安裝的條款](./media/app-proxy/app-proxy-install-connector-terms.png)
3. 在安裝期間，系統將提示您使用 Azure AD 目錄的應用程式 Proxy 註冊連接器。
    * 提供您的 **Azure AD 全域管理員認證**。 您的全域管理員租用戶可能與您的 Microsoft Azure 認證不同。
    * 用於註冊連接器的系統管理員帳戶必須屬於與您啟用應用程式 Proxy 服務的位置相同的目錄。 例如，如果租用戶網域為 contoso.com，則系統管理員應該是 admin@contoso.com，或該網域上的其他有效別名。
    * 如果 [IE 增強式安全性設定] 在您要安裝連接器的伺服器上開啟，可能會封鎖註冊畫面。 若要允許存取，請依照錯誤訊息中的指示。 請確定已停用 [Internet Explorer 增強式安全性]。
    * 如果連接器註冊不成功，請參閱 [針對應用程式 Proxy 進行疑難排解](../active-directory/manage-apps/application-proxy-troubleshoot.md)。

    ![安裝的連接器](./media/app-proxy/app-proxy-connector-installed.png)
4. 若要確保連接器正確運作，請執行 Azure AD 應用程式 Proxy 連接器疑難排解。 執行疑難排解程式之後，您應該看到成功的報告。

    ![疑難排解程式成功](./media/app-proxy/app-proxy-connector-troubleshooter.png)
5. 您應該會在Azure AD 目錄中看到新安裝的連接器列在應用程式 Proxy 頁面上。

    ![](./media/app-proxy/app-proxy-connector-page.png)

> [!NOTE]
> 您可以選擇在多部伺服器上安裝連接器，以確保驗證透過 Azure AD 應用程式 Proxy 發佈之應用程式的高可用性。 執行上述的相同步驟，以在其他加入您受控網域的伺服器上安裝連接器。
>
>

## <a name="next-steps"></a>後續步驟
您已設定 Azure AD 應用程式 Proxy，並將其與您 Azure Active Directory Domain Services 的受控網域進行整合。

* **將您的應用程式移轉到 Azure 虛擬機器︰** 您可以從內部部署伺服器提升應用程式並隨即轉移到加入至受控網域的 Azure 虛擬機器。 如此一來，可協助您刪掉執行伺服器內部部署的基礎結構成本。

* **使用 Azure AD 應用程式 Proxy 發佈應用程式︰** 使用 Azure AD 應用程式 Proxy 發佈您的 Azure 虛擬機器上所執行的應用程式。 如需詳細資訊，請參閱[使用 Azure AD 應用程式 Proxy 發佈應用程式](../active-directory/manage-apps/application-proxy-publish-azure-portal.md)


## <a name="deployment-note---publish-iwa-integrated-windows-authentication-applications-using-azure-ad-application-proxy"></a>部署附註 - 使用 Azure AD 應用程式 Proxy 發佈 IWA (整合式 Windows 驗證) 應用程式
授與應用程式 Proxy 連接器權限來模擬使用者並代表其傳送和接收權杖，以使用「整合式 Windows 驗證」(IWA) 啟用應用程式的單一登入。 設定連接器的 Kerberos 限制委派 (KCD)，以授與存取受控網域上的資源所需的權限。 在受控網域上使用資源型 KCD 機制來提高安全性。


### <a name="enable-resource-based-kerberos-constrained-delegation-for-the-azure-ad-application-proxy-connector"></a>針對 Azure AD 應用程式 Proxy 連接器啟用資源型 Kerberos 限制委派
應設定 Kerberos 限制委派 (KCD) 的 Azure 應用程式 Proxy 連接器，以便它可以在受控網域上模擬使用者。 在 Azure AD 網域服務受控網域上，您沒有網域系統管理員權限。 因此，**無法在受控網域上設定傳統帳戶層級 KCD**。

使用本[文章](active-directory-ds-enable-kcd.md)中所述的資源型 KCD。

> [!NOTE]
> 您必須是「AAD DC 系統管理員」群組的成員，才能使用 AD PowerShell cmdlet 來管理受控網域。
>
>

使用 Get-adcomputer PowerShell cmdlet 來擷取已安裝 Azure AD 應用程式 Proxy 連接器之電腦的設定。
```powershell
$ConnectorComputerAccount = Get-ADComputer -Identity contoso100-proxy.contoso100.com
```

之後，使用組 Set-ADComputer cmdlet 來設定資源伺服器的資源型 KCD。
```powershell
Set-ADComputer contoso100-resource.contoso100.com -PrincipalsAllowedToDelegateToAccount $ConnectorComputerAccount
```

如果您已在受控網域上部署多個應用程式 Proxy 連接器，您需要設定每個此類連接器執行個體的資源型 KCD。


## <a name="related-content"></a>相關內容
* [Azure AD Domain Services - 入門指南](active-directory-ds-getting-started.md)
* [在受控網域上設定 Kerberos 限制委派](active-directory-ds-enable-kcd.md)
* [Kerberos 限制委派概觀](https://technet.microsoft.com/library/jj553400.aspx)
