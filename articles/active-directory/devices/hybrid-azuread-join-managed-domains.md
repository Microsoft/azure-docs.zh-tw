---
title: 如何設定混合式 Azure Active Directory 已加入的裝置 | Microsoft Docs
description: 了解如何設定混合式 Azure Active Directory 已加入的裝置。
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: mtillman
editor: ''
ms.assetid: 54e1b01b-03ee-4c46-bcf0-e01affc0419d
ms.service: active-directory
ms.component: devices
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 08/25/2018
ms.author: markvi
ms.reviewer: sandeo
ms.openlocfilehash: c3c0d9bb6da00272cc7da0ad9f4fae978552c39b
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49352605"
---
# <a name="tutorial-configure-hybrid-azure-active-directory-join-for-managed-domains"></a>教學課程：設定適用於受控網域的混合式 Azure Active Directory Join

以類似的使用方式，裝置會成為您想要保護的另一個身分識別，也會用來隨時隨地保護您的資源。 您可以使用下列其中一種方法將裝置的身分識別導入 Azure AD 中，以達到此目標：

- Azure AD Join
- 混合式 Azure AD Join
- Azure AD 註冊

將您的裝置導入 Azure AD 中，您將可透過跨雲端和內部部署資源的單一登入 (SSO)，將使用者的生產力最大化。 在此同時，您可以利用[條件式存取](../active-directory-conditional-access-azure-portal.md)來保護對雲端和內部部署資源的存取。

在本教學課程中，您將了解如何為受控網域中的裝置設定混合式 Azure AD Join。

> [!div class="checklist"]
> * 設定混合式 Azure AD Join
> * 啟用舊版 Windows 裝置
> * 確認加入的裝置 
> * 疑難排解 


## <a name="prerequisites"></a>必要條件

本教學課程假設您已熟悉：
    
-  [Azure Active Directory 中的裝置管理簡介](../device-management-introduction.md)
    
-  [如何規劃混合式 Azure Active Directory Join 實作](hybrid-azuread-join-plan.md)

-  [如何控制裝置的混合式 Azure AD Join](hybrid-azuread-join-control.md)
  

若要設定本文中的的案例，您必須安裝[最新版的 Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594) (1.1.819.0 或更新版本)。 

確認 Azure AD Connect 已將您要加入混合式 Azure AD 之裝置的電腦物件同步處理至 Azure AD。 如果電腦物件屬於特定組織單位 (OU)，則您也必須在 Azure AD Connect 中設定這些 OU 的同步處理。

從 1.1.819.0 版開始，Azure AD Connect 即為您提供用來設定混合式 Azure AD Join 的精靈。 此精靈可讓您大幅簡化設定程序。 相關精靈會設定裝置註冊的服務連接點 (SCP)。

本文中的設定步驟以此精靈為基礎。 

要使用混合式 Azure AD Join，您的裝置必須能夠從組織的網路中存取下列 Microsoft 資源：  

- https://enterpriseregistration.windows.net
- https://login.microsoftonline.com
- https://device.login.microsoftonline.com
- https://autologon.microsoftazuread-sso.com (如果您正在使用或預計要使用無縫 SSO)

如果您的組織需要透過輸出 Proxy 存取網際網路，自 Windows 10 1709 起，您可以[使用群組原則物件 (GPO) 在電腦上設定 Proxy 設定](https://blogs.technet.microsoft.com/netgeeks/2018/06/19/winhttp-proxy-settings-deployed-by-gpo/)。 如果您的電腦執行任何早於 Windows 10 1709 的版本，您就必須實作 Web Proxy 自動探索 (WPAD)，Windows 10 電腦才能向 Azure AD 進行裝置註冊。 

如果您的組織需要透過已驗證的輸出 Proxy 存取網際網路，您就必須確定 Windows 10 電腦可以成功向輸出 Proxy 進行驗證。 Windows 10 電腦會使用電腦內容來執行裝置註冊，因此必須使用電腦內容來設定輸出 Proxy 驗證。 請向您的輸出 Proxy 提供者洽詢相關設定需求。 



## <a name="configure-hybrid-azure-ad-join"></a>設定混合式 Azure AD Join

若要使用 Azure AD Connect 設定混合式 Azure AD Join，您需要：

- Azure AD 租用戶的全域管理員認證。  

- 每個樹系的企業系統管理員認證。


**若要使用 Azure AD Connect 設定混合式 Azure AD Join：**

1. 啟動 Azure AD Connect，然後按一下 [設定]。

    ![歡迎使用](./media/hybrid-azuread-join-managed-domains/11.png)

2. 在 [其他工作] 頁面上選取 [設定裝置選項]，然後按 [下一步]。 

    ![其他工作](./media/hybrid-azuread-join-managed-domains/12.png)

3. 在 [概觀] 頁面上，按 [下一步]。 

    ![概觀](./media/hybrid-azuread-join-managed-domains/13.png)

4. 在 [連線到 Azure AD] 頁面上，輸入 Azure AD 租用戶的全域管理員認證。  

    ![連接至 Azure AD](./media/hybrid-azuread-join-managed-domains/14.png)

5. 在 [裝置選項] 頁面上選取 [設定混合式 Azure AD Join]，然後按 [下一步]。 

    ![裝置選項](./media/hybrid-azuread-join-managed-domains/15.png)

6. 在 [SCP] 頁面上，針對要以 Azure AD Connect 設定 SCP 的每個樹系執行下列步驟，然後按 [下一步]： 

    ![SCP](./media/hybrid-azuread-join-managed-domains/16.png)

    a. 選取樹系。

    b. 選取驗證服務。

    c. 按一下 [新增]，並輸入企業系統管理員認證。


7. 在 [裝置作業系統] 頁面上，選取 Active Directory 環境中的裝置所使用的作業系統，然後按 [下一步]。 

    ![裝置作業系統](./media/hybrid-azuread-join-managed-domains/17.png)


8. 在 [準備設定] 頁面上，按一下 [設定]。 

    ![準備設定](./media/hybrid-azuread-join-managed-domains/19.png)

9. 在 [設定完成] 頁面上，按一下 [結束]。 

    ![組態完成](./media/hybrid-azuread-join-managed-domains/20.png)




## <a name="enable-windows-down-level-devices"></a>啟用舊版 Windows 裝置

如果有些已加入網域的裝置是舊版 Windows 裝置，您需要︰

- 更新裝置設定
 
- 設定裝置註冊的近端內部網路設定

### <a name="update-device-settings"></a>更新裝置設定 

若要註冊舊版 Windows 裝置，您必須確定已設定允許使用者在 Azure AD 中註冊裝置的裝置設定。 在 Azure 入口網站中，您可以在下列位置找到此設定：

`Home > [Name of your tenant] > Devices - Device settings`  


    
下列原則必須設定為 [全部]：**使用者可以向 Azure AD 註冊其裝置**

![註冊裝置](media/hybrid-azuread-join-managed-domains/23.png)



### <a name="configure-the-local-intranet-settings-for-device-registration"></a>設定裝置註冊的近端內部網路設定

若要順利完成舊版 Windows 裝置的混合式 Azure AD Join，並避免在裝置向 Azure AD 進行驗證時出現憑證提示，您可以將原則推送到已加入網域的裝置，以在 Internet Explorer 中將下列 URL 新增至近端內部網路區域︰

- `https://device.login.microsoftonline.com`

- `https://autologon.microsoftazuread-sso.com` 。

此外，您必須在使用者的近端內部網路區域中啟用 [允許透過指令碼更新狀態列]。

## <a name="verify-the-registration"></a>確認註冊

若要確認 Azure 租用戶的裝置註冊狀態，您可以在 **[Azure Active Directory PowerShell 模組](/powershell/azure/install-msonlinev1?view=azureadps-2.0)** 中使用 **[Get-MsolDevice](https://docs.microsoft.com/powershell/msonline/v1/get-msoldevice)** Cmdlet。

使用 **Get-MSolDevice** Cmdlet 來檢查服務詳細資料時：

- 必須要有**裝置識別碼**與 Windows 用戶端上的識別碼相符的物件存在。
- **DeviceTrustType** 的值必須是 [已加入網域]。 這相當於 Azure AD 入口網站中的 [裝置] 頁面上所呈現的 [已加入混合式 Azure AD] 狀態。
- 在條件式存取中使用的裝置，[Enabled] 的值必須是 [True]，而 [DeviceTrustLevel] 必須是 [Managed]。 


**若要查看服務詳細資料：**

1. 以系統管理員的身分開啟 **Windows PowerShell**。

2. 輸入 `Connect-MsolService` 以連線至您的 Azure 租用戶。  

3. 輸入 `get-msoldevice -deviceId <deviceId>`。

6. 確認 [已啟用] 設為 [True]。





## <a name="troubleshoot-your-implementation"></a>對您的實作進行疑難排解

如果您為已加入網域的 Windows 裝置執行混合式 Azure AD Join 時遇到問題，請參閱：

- [對現行 Windows 裝置的混合式 Azure AD Join 進行疑難排解](troubleshoot-hybrid-join-windows-current.md)
- [對舊版 Windows 裝置的混合式 Azure AD Join 進行疑難排解](troubleshoot-hybrid-join-windows-legacy.md)


## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [設定適用於同盟網域的混合式 Azure Active Directory Join](hybrid-azuread-join-federated-domains.md)
> [手動設定混合式 Azure Active Directory Join](hybrid-azuread-join-manual-steps.md)

