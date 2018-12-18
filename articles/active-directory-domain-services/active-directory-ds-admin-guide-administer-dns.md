---
title: Azure Active Directory Domain Services：管理受控網域上的 DNS | Microsoft Docs
description: 管理 Azure Active Directory Domain Services 受控網域上的 DNS
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
ms.openlocfilehash: f20b2859f72087e208e8963fb18b297c7c670f4f
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39504279"
---
# <a name="administer-dns-on-an-azure-ad-domain-services-managed-domain"></a>管理 Azure AD 網域服務受控網域上的 DNS
Azure Active Directory 網域服務包括可為受控網域提供 DNS 解析的 DNS (網域名稱解析) 伺服器。 有時候，您可能需要在受控網域上設定 DNS。 您可能需要為未新增網域的機器建立 DNS 記錄、設定負載平衡器的虛擬 IP 位址，或設定外部 DNS 轉寄站。 基於這個理由，屬於「AAD DC 系統管理員」群組的使用者會獲授與受控網域上的 DNS 系統管理權限。

[!INCLUDE [active-directory-ds-prerequisites.md](../../includes/active-directory-ds-prerequisites.md)]

## <a name="before-you-begin"></a>開始之前
若要完成本文中所列的工作，您需要：

1. 有效的 **Azure 訂用帳戶**。
2. **Azure AD 目錄** - 與內部部署目錄或僅限雲端的目錄同步處理。
3. **Azure AD 網域服務** 必須已針對 Azure AD 目錄啟用。 如果還沒有啟用，請按照 [入門指南](active-directory-ds-getting-started.md)所述的所有工作來進行。
4. **已加入網域的虛擬機器** ，您可在其中管理 Azure AD 網域服務受控網域。 如果您沒有這類虛擬機器，請依照名為[將 Windows 虛擬機器加入受控網域](active-directory-ds-admin-guide-join-windows-vm.md)一文所述的所有工作進行操作。
5. 您需要目錄中 **屬於「AAD DC 系統管理員」群組之使用者帳戶** 的認證，才能管理受控網域的 DNS。

<br>

## <a name="task-1---create-a-domain-joined-virtual-machine-to-remotely-administer-dns-for-the-managed-domain"></a>工作 1 - 建立已加入網域的虛擬機器以從遠端管理受控網域的 DNS
使用 Active Directory 管理中心 (ADAC) 或 AD PowerShell 等熟悉的 Active Directory 系統管理工具，可以從遠端管理 Azure AD 網域服務受控網域。 同樣地，使用 DNS 伺服器系統管理工具，可以從遠端管理受控網域的 DNS。

Azure AD 目錄中的系統管理員沒有權限，不能透過遠端桌面連接受控網域上的網域控制站。 「AAD DC 系統管理員」群組的成員可以使用 DNS 伺服器工具，從已加入受控網域的 Windows Server/用戶端電腦遠端管理受控網域的 DNS。 DNS 伺服器工具是遠端伺服器管理工具 (RSAT) 選用功能的一部分。

第一個工作是建立已加入受控網域的 Windows Server 虛擬機器。 如需相關指示，請參閱標題為 [將 Windows Server 虛擬機器加入 Azure Active Directory Domain Services 受控網域](active-directory-ds-admin-guide-join-windows-vm.md)的文章。

## <a name="task-2---install-dns-server-tools-on-the-virtual-machine"></a>工作 2 - 在虛擬機器上安裝 DNS 伺服器工具
完成下列步驟，在已加入網域的虛擬機器上安裝 DNS 管理工具。 如需有關[安裝和使用遠端伺服器管理工具](https://technet.microsoft.com/library/hh831501.aspx)的詳細資訊，請參閱 TechNet。

1. 瀏覽至 Azure 入口網站。 在左側面板上按一下 [所有資源]。 找出並按一下您在工作 1 中建立的虛擬機器。
2. 在 [概觀] 索引標籤上按一下 [連線] 按鈕。隨即建立並下載遠端桌面通訊協定 (.rdp) 檔案。

    ![連線至 Windows 虛擬機器](./media/active-directory-domain-services-admin-guide/connect-windows-vm.png)
3. 若要連線至您的 VM，請開啟下載的 RDP 檔案。 出現提示時，按一下 [連線]。 使用屬於「AAD DC 系統管理員」群組之使用者的認證。 例如，'bob@domainservicespreview.onmicrosoft.com'。 您可能會在登入過程中收到憑證警告。 按一下 [是] 或 [繼續] 來連線。

4. 在 [開始] 畫面中開啟 [伺服器管理員] 。 按一下 [伺服器管理員] 視窗中央窗格內的 [新增角色及功能]  。

    ![啟動虛擬機器上的伺服器管理員](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager.png)
5. 在 [新增角色及功能精靈] 的 [開始之前] 頁面上，按 [下一步]。

    ![[開始之前] 頁面](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager-add-roles-begin.png)
6. 在 [安裝類型] 頁面上，保持勾選 [角色型或功能型安裝] 選項，然後按 [下一步]。

    ![[安裝類型] 頁面](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager-add-roles-type.png)
7. 在 [伺服器選擇] 頁面上，從伺服器集區中選取目前的虛擬機器，然後按 [下一步]。

    ![[伺服器選擇] 頁面](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager-add-roles-server.png)
8. 在 [伺服器角色] 頁面上，按 [下一步]。
9. 在 [功能] 頁面上，按一下以展開 [遠端伺服器管理工具] 節點，然後按一下以展開 [角色管理工具] 節點。 從角色管理工具清單中，選取 [DNS 伺服器工具] 功能。

    ![[功能] 頁面](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager-add-roles-dns-tools.png)
10. 在 [確認] 頁面上，按一下 [安裝] 以在虛擬機器上安裝 DNS 伺服器工具功能。 順利完成功能安裝時，按一下 [關閉] 以結束 [新增角色及功能] 精靈。

    ![確認電子郵件](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager-add-roles-dns-confirmation.png)

## <a name="task-3---launch-the-dns-management-console-to-administer-dns"></a>工作 3 - 啟動 DNS 管理主控台來管理 DNS
現在，您可以使用 Windows Server DNS 工具來管理受控網域上的 DNS。

> [!NOTE]
> 您必須是「AAD DC 系統管理員」群組的成員，才能在受控網域上管理 DNS。
>
>

1. 從 [開始] 畫面中，按一下 [系統管理工具] 。 您應該會看到安裝在虛擬機器上的 [DNS]  主控台。

    ![系統管理工具 - DNS 主控台](./media/active-directory-domain-services-admin-guide/install-rsat-dns-tools-installed.png)
2. 按一下 [DNS]  以啟動 DNS 管理主控台。
3. 在 [連線到 DNS 伺服器] 對話方塊中，按一下 [下列電腦]，然後輸入受控網域的 DNS 網域名稱 (例如 'contoso100.com')。

    ![DNS 主控台 - 連線到網域](./media/active-directory-domain-services-admin-guide/dns-console-connect-to-domain.png)
4. DNS 主控台連線到受控網域。

    ![DNS 主控台 - 管理網域](./media/active-directory-domain-services-admin-guide/dns-console-managed-domain.png)
5. 您現在可以使用 DNS 主控台來為已在其中啟用 AAD 網域服務之虛擬網路內的電腦新增 DNS 項目。

> [!WARNING]
> 使用 DNS 管理工具來管理受控網域的 DNS 時，請務必小心。 務必要確定您 **並未刪除或修改網域中的網域服務所使用的內建 DNS 記錄**。 內建 DNS 記錄包括網域 DNS 記錄、名稱伺服器記錄和其他用於 DC 位置的記錄。 如果您修改這些記錄，虛擬網路上的網域服務會中斷。
>
>

如需管理 DNS 的詳細資訊，請參閱 [Technet 上的 DNS 工具文章](https://technet.microsoft.com/library/cc753579.aspx)。

## <a name="related-content"></a>相關內容
* [Azure AD Domain Services - 入門指南](active-directory-ds-getting-started.md)
* [將 Windows Server 虛擬機器加入 Azure Active Directory Domain Services 受控網域](active-directory-ds-admin-guide-join-windows-vm.md)
* [Administer an Azure AD Domain Services managed domain (管理 Azure AD 網域服務受控網域)](active-directory-ds-admin-guide-administer-domain.md)
* [DNS 系統管理工具](https://technet.microsoft.com/library/cc753579.aspx)
