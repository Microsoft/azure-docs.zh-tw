---
title: 針對已加入混合式 Azure Active Directory 的下層裝置進行疑難排解 | Microsoft Docs
description: 針對已加入混合式 Azure Active Directory 的下層裝置進行疑難排解。
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: mtillman
ms.assetid: cdc25576-37f2-4afb-a786-f59ba4c284c2
ms.service: active-directory
ms.component: devices
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/23/2018
ms.author: markvi
ms.reviewer: jairoc
ms.openlocfilehash: 2c50ba1abfe3681a39b39bf52f127efd9d518aef
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43041863"
---
# <a name="troubleshooting-hybrid-azure-active-directory-joined-down-level-devices"></a>針對已加入混合式 Azure Active Directory 的下層裝置進行疑難排解 

本文章僅適用於下列裝置： 

- Windows 7 
- Windows 8.1 
- Windows Server 2008 R2 
- Windows Server 2012 
- Windows Server 2012 R2 
 

對於 Windows 10 或 Windows Server 2016，請參閱[針對已加入混合式 Azure Active Directory 的 Windows 10 和 Windows Server 2016 裝置進行疑難排解](troubleshoot-hybrid-join-windows-current.md)。

本文章假設您[設定已加入混合式 Azure Active Directory 的裝置](hybrid-azuread-join-plan.md)來支援下列案例：

- 裝置型條件式存取

- [設定的企業漫遊](../active-directory-windows-enterprise-state-roaming-overview.md)

- [Windows Hello 企業版](https://docs.microsoft.com/windows/security/identity-protection/hello-for-business/hello-identity-verification) 





本文章提供有關如何解決潛在問題的疑難排解指引。  

**您應該知道的事情：** 

- 每位使用者的裝置數目上限是以裝置為主。 舉例來說：如果 *jdoe* 和 *jharnett* 登入裝置，在 [使用者] 資訊索引標籤中就會為每位使用者建立個別的註冊 (DeviceID)。  

- 您可以將裝置的初始註冊/加入設定為在登入或鎖定/解除鎖定時嘗試執行。 工作排程器工作可能會觸發 5 分鐘的延遲。 

- 您可以在使用者資訊索引標籤上，因為重新安裝作業系統或以手動方式重新註冊來取得適用於裝置的多個項目。 

- 如果是 Windows 7 SP1 或 Windows Server 2008 R2 SP1，請確定已安裝 [KB4284842](https://support.microsoft.com/help/4284842)。 此更新可避免未來的驗證因客戶在變更密碼之後遺失存取受保護金鑰的權限而失敗。

## <a name="step-1-retrieve-the-registration-status"></a>步驟 1：擷取註冊狀態 

**確認註冊狀態：**  

1. 使用已執行混合式 Azure AD 加入的使用者帳戶登入。

2. 以系統管理員身分開啟命令提示字元 

3. 輸入 `"%programFiles%\Microsoft Workplace Join\autoworkplace.exe" /i`

此命令會顯示一個對話方塊，可提供您有關加入狀態的更多詳細資料。

![Workplace Join for Windows](./media/troubleshoot-hybrid-join-windows-legacy/01.png)


## <a name="step-2-evaluate-the-hybrid-azure-ad-join-status"></a>步驟 2：評估混合式 Azure AD 加入狀態 

如果未成功加入混合式 Azure AD，對話方塊會提供有關所發生問題的詳細資料。

**最常見的問題包括：**

- AD FS 或 Azure AD 設定不正確

    ![Workplace Join for Windows](./media/troubleshoot-hybrid-join-windows-legacy/02.png)

- 您不是以網域使用者身分登入

    ![Workplace Join for Windows](./media/troubleshoot-hybrid-join-windows-legacy/03.png)
    
    這種情況可能是由幾種不同的原因所致︰
    
    - 登入的使用者不是網域使用者 (例如本機使用者)。 對下層裝置的混合式 Azure AD 聯結僅支援網域使用者。
    
    - Autoworkplace.exe 無法以無訊息方式使用 Azure AD 或 AD FS 進行驗證。 這可能是因為 Azure AD URL 的輸出網路連線問題而引起的。 也可能是已針對使用者啟用/設定 Multi-Factor Authentication (MFA)，而且未在同盟伺服器中設定 WIAORMUTLIAUTHN。 另一個可能原因是主領域探索 (HRD) 頁面正在等候使用者互動，而導致 **autoworkplace.exe** 無法以無訊息方式要求權杖。
    
    - 貴組織使用的是 Azure AD 無縫單一登入，`https://autologon.microsoftazuread-sso.com` 或 `https://aadg.windows.net.nsatc.net` 不在裝置的 IE 內部網路設定中，並且未針對內部網路區域啟用 [允許透過指令碼更新至狀態列]。

- 已達到配額限制

    ![Workplace Join for Windows](./media/troubleshoot-hybrid-join-windows-legacy/04.png)

- 服務沒有回應 

    ![Workplace Join for Windows](./media/troubleshoot-hybrid-join-windows-legacy/05.png)

您也可以在事件記錄檔的 [應用程式及服務記錄檔] > [Microsoft-Workplace Join] 底下找到狀態資訊
  
**混合式 Azure AD 加入失敗的最常見原因包括：** 

- 您的電腦未連線至組織的內部網路，或不在可連線到內部部署 AD 網域控制站的 VPN 上。

- 您是使用本機電腦帳戶來登入您的電腦。 

- 服務組態問題： 

  - 同盟伺服器已設定為支援 **WIAORMULTIAUTHN**。 

  - 您電腦的樹系沒有「服務連接點」物件指向 Azure AD 中已驗證的網域名稱 

  - 使用者已達到裝置限制。 

## <a name="next-steps"></a>後續步驟

如有問題，請參閱[裝置管理常見問題集](faq.md)  
