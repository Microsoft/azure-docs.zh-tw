---
title: Azure Active Directory Domain Services：將 Windows Server VM 加入受控網域 | Microsoft Docs
description: 將 Windows Server 虛擬機器加入 Azure AD DS
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: 29316313-c76c-4fb9-8954-5fa5ec82609e
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/21/2018
ms.author: maheshu
ms.openlocfilehash: f9ee68fda3bb5e0f5302c8d5c96da0515c05ce1d
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39503395"
---
# <a name="join-a-windows-server-virtual-machine-to-a-managed-domain"></a>將 Windows Server 虛擬機器加入受控網域
本文說明如何使用 Azure 入口網站來部署 Windows Server 虛擬機器。 然後會說明如何將虛擬機器加入 Azure Active Directory Domain Services (Azure AD DS) 受控網域。

[!INCLUDE [active-directory-ds-prerequisites.md](../../includes/active-directory-ds-prerequisites.md)]

## <a name="step-1-create-a-windows-server-virtual-machine"></a>步驟 1：建立 Windows Server 虛擬機器
若要建立 Windows 虛擬機器，並將其加入已啟用 Azure AD DS 的虛擬網路，請執行下列步驟：

1. 登入 [Azure 入口網站](http://portal.azure.com)。
2. 在左窗格頂端選取 [新增]。
3. 選取 [計算]，然後選取 [Windows Server 2016 Datacenter]。

    ![Windows Server 2016 資料中心連結](./media/active-directory-domain-services-admin-guide/create-windows-vm-select-image.png)

4. 在精靈的 [基本] 窗格中，設定虛擬機器的基本設定。

    ![[基本] 窗格](./media/active-directory-domain-services-admin-guide/create-windows-vm-basics.png)

    > [!TIP]
    > 您在此處輸入的使用者名稱和密碼是用於登入虛擬機器的本機系統管理員帳戶。 挑選強式密碼來保護虛擬機器，防止暴力密碼破解攻擊。 請勿在此輸入網域使用者帳戶的認證。
    >

5. 選取虛擬機器的 [大小]。 若要檢視更多大小，請選取 [檢視全部] 或變更 [支援的磁碟類型] 篩選條件。

    ![[選擇大小] 窗格](./media/active-directory-domain-services-admin-guide/create-windows-vm-size.png)

6. 在 [設定] 窗格中，選取您的 Azure AD DS 受控網域部署所在的虛擬網路。 挑選與您的受控網域部署所在的不同子網路。 對於其他設定，請保留預設值，然後選取 [確定]。

    ![虛擬機器的虛擬網路設定](./media/active-directory-domain-services-admin-guide/create-windows-vm-select-vnet.png)

    > [!TIP]
    > **挑選適當的虛擬網路和子網路。**
    >
    > 選取您的受控網域部署所在的虛擬網路，或使用虛擬網路對等互連與其連線的虛擬網路兩者之一。 如果您選取未連線的虛擬網路，則無法將虛擬機器加入受控網域中。
    >
    > 建議您將受控網域部署至專用的子網路。 因此，請勿挑選已啟用受控網域所在的子網路。

7. 對於其他設定，請保留預設值，然後選取 [確定]。
8. 在 [購買] 頁面上，檢閱設定然後選取 [確定] 來部署虛擬機器。
9. VM 部署已釘選到 Azure 入口網站儀表板。

    ![完成](./media/active-directory-domain-services-admin-guide/create-windows-vm-done.png)
10. 部署完成之後，您可以在 [概觀] 頁面上檢視虛擬機器的相關資訊。


## <a name="step-2-connect-to-the-windows-server-virtual-machine-by-using-the-local-administrator-account"></a>步驟 2︰使用本機系統管理員帳戶連線到 Windows Server 虛擬機器
接著，連線到新建立的 Windows Server 虛擬機器，以便將其加入網域。 使用您建立虛擬機器時所指定的本機系統管理員認證。

若要連線到虛擬機器，請執行下列步驟：

1. 在 [概觀] 窗格中，選取 [連接]。  
    隨即建立並下載遠端桌面通訊協定 (.rdp) 檔案。

    ![連線至 Windows 虛擬機器](./media/active-directory-domain-services-admin-guide/connect-windows-vm.png)

2. 若要連線至您的 VM，請開啟下載的 RDP 檔案。 如果出現提示，請選取 [連接]。
3. 輸入您在建立虛擬機器時所指定的**本機系統管理員認證** (例如 *localhost\mahesh*)。
4. 如果您在登入程序期間看見憑證警告，請選取 [是] 或 [繼續] 來連線。

此時，您應該已使用本機系統管理員認證登入到新建立的 Windows 虛擬機器。 下一個步驟是將虛擬機器加入網域。


## <a name="step-3-join-the-windows-server-virtual-machine-to-the-azure-ad-ds-managed-domain"></a>步驟 3︰將 Windows Server 虛擬機器加入 Azure AD DS 受控網域
若要將 Windows Server 虛擬機器加入受 Azure AD DS 管理的網域，請完成下列步驟：

1. 如「步驟 2」所示，連線到 Windows Server VM。 在 [開始] 畫面上開啟 [伺服器管理員]。
2. 在 [伺服器管理員] 視窗的左窗格中，選取 [本機伺服器]。

    ![在虛擬機器上開啟 [伺服器管理員] 視窗](./media/active-directory-domain-services-admin-guide/join-domain-server-manager.png)

3. 在 [屬性] 下，選取 [工作群組]。
4. 在 [系統屬性] 視窗中，選取 [變更] 來加入網域。

    ![[系統屬性] 視窗](./media/active-directory-domain-services-admin-guide/join-domain-system-properties.png)

5. 在 [網域] 方塊中，指定 Azure AD DS 受控網域名稱，然後選取 [確定]。

    ![指定要加入的網域](./media/active-directory-domain-services-admin-guide/join-domain-system-properties-specify-domain.png)

6. 系統會要求您輸入認證以加入網域。 使用可供「屬於 Azure AD DC 系統管理員群組的使用者」使用的認證。 只有此群組的成員才有權限可以將機器加入受控網域。

    ![可供指定認證的 [Windows 安全性] 視窗](./media/active-directory-domain-services-admin-guide/join-domain-system-properties-specify-credentials.png)

7. 您可以透過下列任一方式指定認證︰

   * **UPN 格式：**(建議) 指定 Azure AD 中所設定使用者帳戶的使用者主體名稱 (UPN) 尾碼。 在此範例中，使用者 bob 的 UPN 尾碼是 bob@domainservicespreview.onmicrosoft.com。

   * **SAMAccountName 格式**：您可以使用 SAMAccountName 格式指定帳戶名稱。 在此範例中，使用者 bob 必須輸入 CONTOSO100\bob。

     > [!TIP]
     > **建議使用 UPN 格式來指定認證。**
     >
     > 如果使用者的 UPN 前置詞太長 (例如 joehasareallylongname)，就可能自動產生 SAMAccountName。 如果您的 Azure AD 租用戶中有多個使用者擁有相同的 UPN 前置詞 (例如 bob)，則服務可能會自動產生它們的 SAMAccountName 格式。 在這些情況下，您可以使用 UPN 格式可靠地登入網域。
     >

8. 成功加入網域之後，會出現下列訊息來歡迎您加入網域。

    ![歡迎加入網域](./media/active-directory-domain-services-admin-guide/join-domain-done.png)

9. 若要完成加入網域，請重新啟動虛擬機器。

## <a name="troubleshoot-joining-a-domain"></a>對加入網域進行疑難排解
### <a name="connectivity-issues"></a>連線能力問題
如果虛擬機器找不到網域，請嘗試執行下列疑難排解步驟：

* 確認虛擬機器已連線到已啟用 Azure AD DS 的同一個虛擬網路。 否則，虛擬機器無法連線到或加入網域。

* 確認虛擬機器所在的虛擬網路接著會連線到已啟用 Azure AD DS 的虛擬網路。

* 嘗試 ping 受控網域的 DNS 網域名稱 (例如*ping contoso100.com*)。 如果您無法這麼做，請嘗試 ping 頁面上所顯示且您已在其中啟用 Azure AD DS 之網域的 IP 位址 (例如 ping 10.0.0.4)。 如果您可以 ping 該 IP 位址，但無法 ping 網域，則表示 DNS 的設定可能不正確。 檢查網域的 IP 位址以查看是否設定為虛擬網路的 DNS 伺服器。

* 請嘗試排清虛擬機器上的 DNS 解析程式快取 (ipconfig /flushdns)。

如果顯示的視窗要求您提供認證以加入網域，則表示您沒有連線問題。

### <a name="credentials-related-issues"></a>與認證相關的問題
如果您的認證有問題因而無法加入網域，請嘗試執行下列一或多個疑難排解步驟：

* 嘗試使用 UPN 格式來指定認證。 如果您的租用戶中有許多使用者具有相同的 UPN 前置詞，或您的 UPN 前置詞太長，系統可能就會自動為您的帳戶產生 SAMAccountName。 在這些情況下，您帳戶的 SAMAccountName 格式可能會與您在內部部署網域中預期或使用的格式不同。

* 嘗試使用屬於「AAD DC 系統管理員」群組的使用者帳戶認證。

* 確認您已為受控網域[啟用密碼同步化](active-directory-ds-getting-started-password-sync.md)。

* 確認您已使用 Azure AD 中所設定的使用者 UPN (例如 *bob@domainservicespreview.onmicrosoft.com*) 來登入。

* 等候足夠長的時間以讓密碼同步處理完成，如《快速入門指南》中所指定。

## <a name="related-content"></a>相關內容
* [Azure AD DS 快速入門指南](active-directory-ds-getting-started.md)
* [管理 Azure AD DS 受控網域](active-directory-ds-admin-guide-administer-domain.md)
