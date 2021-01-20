---
title: 在 Azure AD 中 Azure AD Connect 雲端同步的必要條件
description: 本文說明雲端同步處理所需的必要條件和硬體需求。
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 12/11/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: b83c9b0ece933ad71810c50e89ae296aa218ec75
ms.sourcegitcommit: 8a74ab1beba4522367aef8cb39c92c1147d5ec13
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/20/2021
ms.locfileid: "98613305"
---
# <a name="prerequisites-for-azure-ad-connect-cloud-sync"></a>Azure AD Connect 雲端同步處理的必要條件
本文提供有關如何選擇和使用 Azure Active Directory (Azure AD) 將雲端同步處理作為身分識別解決方案的指引。

## <a name="cloud-provisioning-agent-requirements"></a>雲端佈建代理程式需求
您需要下列各項，才能使用 Azure AD Connect cloud sync：

- 網域系統管理員或企業系統管理員認證，可建立 Azure AD Connect Cloud Sync gMSA (群組受管理的服務帳戶) 來執行代理程式服務。 
- 不是來賓使用者的 Azure AD 租使用者的混合式身分識別系統管理員帳戶。
- 佈建代理程式的 Windows 2012 R2 或更新版本內部部署伺服器。  此伺服器應該是以 [Active Directory 系統管理層模型](/windows-server/identity/securing-privileged-access/securing-privileged-access-reference-material)為基礎的第0層伺服器。
- 內部部署防火牆設定。

## <a name="group-managed-service-accounts"></a>群組受管理的服務帳戶
群組受管理的服務帳戶是受控網域帳戶，可提供自動密碼管理、簡化的服務主體名稱 (SPN) 管理、將管理委派給其他系統管理員的能力，以及將這項功能延伸到多部伺服器。  Azure AD Connect Cloud Sync 支援並使用 gMSA 來執行代理程式。  系統會在安裝期間提示您輸入系統管理認證，以便建立此帳戶。  帳戶會顯示為 (domain\provAgentgMSA $) 。  如需 gMSA 的詳細資訊，請參閱 [群組受管理的服務帳戶](/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) 

### <a name="prerequisites-for-gmsa"></a>GMSA 的必要條件：
1.  GMSA 網域樹系中的 Active Directory 架構必須更新為 Windows Server 2012
2.  網域控制站上的[POWERSHELL RSAT 模組](/windows-server/remote/remote-server-administration-tools)
3.  網域中至少有一個網域控制站必須執行 Windows Server 2012。
4.  安裝代理程式的已加入網域伺服器必須是 Windows Server 2012 或更新版本。

### <a name="custom-gmsa-account"></a>自訂 gMSA 帳戶
如果您要建立自訂 gMSA 帳戶，您必須確定帳戶具有下列許可權。

|類型 |名稱 |存取 |套用至| 
|-----|-----|-----|-----|
|Allow |gMSA 帳戶 |讀取所有屬性 |子系裝置物件| 
|Allow |gMSA 帳戶|讀取所有屬性 |子系 InetOrgPerson 物件| 
|Allow |gMSA 帳戶 |讀取所有屬性 |子系電腦物件| 
|Allow |gMSA 帳戶 |讀取所有屬性 |子系 foreignSecurityPrincipal 物件| 
|Allow |gMSA 帳戶 |完全控制 |子系群組物件| 
|Allow |gMSA 帳戶 |讀取所有屬性 |子系使用者物件| 
|Allow |gMSA 帳戶 |讀取所有屬性 |子系連絡人物件| 
|Allow |gMSA 帳戶 |建立/刪除使用者物件|此物件和所有子系物件| 

如需有關如何將現有的代理程式升級為使用 gMSA 帳戶的步驟，請參閱 [群組受管理的服務帳戶](how-to-install.md#group-managed-service-accounts)。

### <a name="in-the-azure-active-directory-admin-center"></a>於 Azure Active Directory 管理中心

1. 在您的 Azure AD 租使用者上建立僅限雲端的混合式身分識別系統管理員帳戶。 如此一來，如果內部部署服務失敗或無法使用，即可管理租用戶設定。 瞭解如何 [新增僅限雲端的混合式身分識別系統管理員帳戶](../fundamentals/add-users-azure-active-directory.md)。 完成此步驟對於確保不會被租用戶封鎖至關重要。
1. 將一或多個[自訂網域名稱](../fundamentals/add-custom-domain.md)新增至 Azure AD 租用戶。 您的使用者可以使用其中一個網域名稱登入。

### <a name="in-your-directory-in-active-directory"></a>在 Active Directory 目錄中

執行 [IdFix 工具](/office365/enterprise/prepare-directory-attributes-for-synch-with-idfix)來準備目錄屬性以進行同步處理。

### <a name="in-your-on-premises-environment"></a>在內部部署環境中

1. 識別已加入網域、執行 Windows Server 2012 R2 或更新版本，且至少有 4 GB RAM 和 .NET 4.7.1+ 執行階段的主機伺服器。

2. 本機伺服器上的 PowerShell 執行原則必須設定為 Undefined 或 RemoteSigned。

3. 如果伺服器與 Azure AD 之間有防火牆，請設定下列項目：
    - 確定代理程式可透過下列連接埠對 Azure AD 提出 *輸出* 要求：

      | 連接埠號碼 | 使用方式 |
      | --- | --- |
      | **80** | 驗證 TLS/SSL 憑證時下載憑證撤銷清單 (CRL)。  |
      | **443** | 處理服務的所有輸出通訊。 |
      | **8080** (選擇性) | 如果無法使用連接埠 443，則代理程式會透過連接埠 8080 每 10 分鐘報告其狀態一次。 此狀態會顯示在 Azure 入口網站中。 |

    - 如果您的防火牆會根據原始使用者強制執行規則，請開啟這些連接埠，讓來自以網路服務形式執行之 Windows 服務的流量得以通行。
    - 如果防火牆或 Proxy 允許指定安全尾碼，請將連線新增至 \*.msappproxy.net 和 \*.servicebus.windows.net。 如果不允許建立，請允許存取每週更新的 [Azure 資料中心 IP 範圍](https://www.microsoft.com/download/details.aspx?id=41653)。
    - 代理程式必須可存取 login.windows.net 與 login.microsoftonline.com，才能進行初始註冊。 因此也請針對這些 URL 開啟您的防火牆。
    - 為了驗證憑證，請解除封鎖下列 URL：mscrl.microsoft.com:80、crl.microsoft.com:80、ocsp.msocsp.com:80 和 www\.microsoft.com:80。 這些 URL 會用於其他 Microsoft 產品的憑證驗證，因此可能已將這些 URL 解除封鎖。

    >[!NOTE]
    > 不支援在 Windows Server Core 上安裝雲端佈建代理程式。

### <a name="additional-requirements"></a>其他需求

- [Microsoft .NET Framework 4.7.1](https://www.microsoft.com/download/details.aspx?id=56116) 

#### <a name="tls-requirements"></a>TLS 需求

> [!NOTE]
> 傳輸層安全性 (TLS) 是為了安全通訊所提供的通訊協定。 變更 TLS 設定會影響整個樹系。 如需詳細資訊，請參閱[更新為啟用 TLS 1.1 和 TLS 1.2 作為 Windows WinHTTP 中的預設安全通訊協定](https://support.microsoft.com/help/3140245/update-to-enable-tls-1-1-and-tls-1-2-as-default-secure-protocols-in-wi)。

裝載 Azure AD Connect 雲端佈建代理程式的 Windows Server 必須先啟用 TLS 1.2，才能進行安裝。

若要啟用 TLS 1.2，請遵循下列步驟。

1. 設定下列登錄機碼：

    ```
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2]
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client] "DisabledByDefault"=dword:00000000 "Enabled"=dword:00000001
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server] "DisabledByDefault"=dword:00000000 "Enabled"=dword:00000001
    [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319] "SchUseStrongCrypto"=dword:00000001
    ```

1. 重新啟動伺服器。

## <a name="known-limitations"></a>已知的限制

以下是已知的限制：

### <a name="delta-synchronization"></a>差異同步處理

- 差異同步的群組範圍篩選不支援1500以上的成員。
- 當您刪除用來作為群組範圍篩選準則一部分的群組時，不會刪除群組成員的使用者。 
- 當您重新命名範圍中的 OU 或群組時，差異同步處理不會移除使用者。

### <a name="provisioning-logs"></a>佈建記錄
- 布建記錄無法清楚區分建立和更新作業。  您可能會看到用於更新的建立作業，以及建立的更新作業。

### <a name="group-re-naming-or-ou-re-naming"></a>群組重新命名或 OU 重新命名
- 如果您將 AD 中的群組或 OU 重新命名為指定設定的範圍，雲端同步作業將無法辨識 AD 中的名稱變更。 作業不會進入隔離狀態，且會維持狀況良好。


## <a name="next-steps"></a>後續步驟 

- [什麼是佈建？](what-is-provisioning.md)
- [什麼是雲端同步 Azure AD Connect？](what-is-cloud-sync.md)