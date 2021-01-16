---
title: 建立 Windows 虛擬桌面主機集區 PowerShell - Azure
description: 如何使用 PowerShell Cmdlet 在 Windows 虛擬桌面中建立主機集區。
author: Heidilohr
ms.topic: how-to
ms.date: 10/02/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 9ec900f0537030d3ed0d1c875e8125806159bd51
ms.sourcegitcommit: 25d1d5eb0329c14367621924e1da19af0a99acf1
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/16/2021
ms.locfileid: "98251449"
---
# <a name="create-a-windows-virtual-desktop-host-pool-with-powershell"></a>使用 PowerShell 建立 Windows 虛擬桌面主機集區

>[!IMPORTANT]
>此內容適用於具有 Azure Resource Manager Windows 虛擬桌面物件的 Windows 虛擬桌面。 如果您使用不含 Azure Resource Manager 物件的 Windows 虛擬桌面 (傳統版)，請參閱[這篇文章](./virtual-desktop-fall-2019/create-host-pools-powershell-2019.md)。

主機集區是 Windows 虛擬桌面租用戶環境中一或多個相同虛擬機器的集合。 每個主機集區都可與多個 RemoteApp 群組、一個桌面應用程式群組及多個工作階段主機建立關聯。

## <a name="prerequisites"></a>Prerequisites

本文假設您已遵循[設定 PowerShell 模組](powershell-module.md)中的指示完成作業。

## <a name="use-your-powershell-client-to-create-a-host-pool"></a>使用 PowerShell 用戶端建立主機集區

執行下列 Cmdlet 來登入 Windows 虛擬桌面環境：

```powershell
New-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -WorkspaceName <workspacename> -HostPoolType <Pooled|Personal> -LoadBalancerType <BreadthFirst|DepthFirst|Persistent> -Location <region> -DesktopAppGroupName <appgroupname>
```

此 Cmdlet 會建立主機集區、工作空間和桌面應用程式群組。 也會向工作空間註冊桌面應用程式群組。 您可使用此 Cmdlet 來建立工作空間，或使用現有的工作空間。

執行下一個 Cmdlet 以建立註冊權杖來授權工作階段主機加入主機集區，並將其儲存到本機電腦上的新檔案。 您可使用 -ExpirationHours 參數來指定註冊權杖的有效期長短。

>[!NOTE]
>權杖的到期日不能少於一小時，也不能超過一個月。 如果 *-ExpirationTime* 的設定超出此限制，Cmdlet 就不會建立權杖。

```powershell
New-AzWvdRegistrationInfo -ResourceGroupName <resourcegroupname> -HostPoolName <hostpoolname> -ExpirationTime $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
```

例如，如果想要建立會在兩個小時內過期的權杖，請執行此 Cmdlet：

```powershell
New-AzWvdRegistrationInfo -ResourceGroupName <resourcegroupname> -HostPoolName <hostpoolname> -ExpirationTime $((get-date).ToUniversalTime().AddHours(2).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
```

然後執行此 Cmdlet，以將 Azure Active Directory 使用者新增至主機集區的預設桌面應用程式群組。

```powershell
New-AzRoleAssignment -SignInName <userupn> -RoleDefinitionName "Desktop Virtualization User" -ResourceName <hostpoolname+"-DAG"> -ResourceGroupName <resourcegroupname> -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
```

執行此下一個 Cmdlet，以將 Azure Active Directory 使用者新增至主機集區的預設桌面應用程式群組：

```powershell
New-AzRoleAssignment -ObjectId <usergroupobjectid> -RoleDefinitionName "Desktop Virtualization User" -ResourceName <hostpoolname+"-DAG"> -ResourceGroupName <resourcegroupname> -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
```

執行下列 Cmdlet，以將註冊權杖匯出為變數，稍後用以[將虛擬機器註冊至 Windows 虛擬桌面主機集區](#register-the-virtual-machines-to-the-windows-virtual-desktop-host-pool)。

```powershell
$token = Get-AzWvdRegistrationInfo -ResourceGroupName <resourcegroupname> -HostPoolName <hostpoolname>
```

## <a name="create-virtual-machines-for-the-host-pool"></a>建立主機集區的虛擬機器

現在即可建立能夠加入至 Windows 虛擬桌面主機集區的 Azure 虛擬機器。

您有多種方式可建立虛擬機器：

- [從 Azure 資源庫映像建立虛擬機器](../virtual-machines/windows/quick-create-portal.md#create-virtual-machine)
- [從受控映像建立虛擬機器](../virtual-machines/windows/create-vm-generalized-managed.md)
- [從非受控映像建立虛擬機器](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-user-image-data-disks)

>[!NOTE]
>如果要使用 Windows 7 作為主機 OS 來部署虛擬機器，則建立和部署程序會略有不同。 如需詳細資料，請參閱[在 Windows 虛擬桌面上部署 Windows 7 虛擬機器](./virtual-desktop-fall-2019/deploy-windows-7-virtual-machine.md)。

建立工作階段主機虛擬機器之後，[將 Windows 授權套用至工作階段主機 VM](./apply-windows-license.md#apply-a-windows-license-to-a-session-host-vm)，不用支付其他授權費用即可執行 Windows 或 Windows Server 虛擬機器。

## <a name="prepare-the-virtual-machines-for-windows-virtual-desktop-agent-installations"></a>準備虛擬機器以安裝 Windows 虛擬桌面代理程式

您必須先執行下列工作來備妥虛擬機器，才能安裝 Windows 虛擬桌面代理程式，並向 Windows 虛擬桌面主機集區註冊虛擬機器：

- 電腦必須加入網域。 這可讓進入的 Windows 虛擬桌面使用者能從其 Azure Active Directory 帳戶對應至其 Active Directory 帳戶，以成功存取虛擬機器。
- 如果虛擬機器執行的是 Windows Server 作業系統，則必須安裝遠端桌面工作階段主機 (RDSH) 角色。 RDSH 角色可供正確安裝 Windows 虛擬桌面代理程式。

為成功加入網域，請在每部虛擬機器上執行下列作業：

1. 使用建立虛擬機器時所提供的認證來[連線到虛擬機器](../virtual-machines/windows/quick-create-portal.md#connect-to-virtual-machine)。
2. 在虛擬機器上，啟動 [控制台]，然後選取 [系統]。
3. 依序選取 [電腦名稱]、[變更設定] 以及 [變更...]
4. 選取 [網域]，然後輸入虛擬網路上的 Active Directory 網域。
5. 使用具有加入網域電腦權限的網域帳戶進行驗證。

    >[!NOTE]
    > 如果您要將 VM 加入 Azure Active Directory Domain Services (Azure AD DS) 環境，請確定您的網域加入使用者也是 [AAD DC 系統管理員群組](../active-directory-domain-services/tutorial-create-instance-advanced.md#configure-an-administrative-group)的成員。

>[!IMPORTANT]
>建議您不要啟用任何停用 Windows Installer 的原則或設定。 如果您停用 Windows Installer，服務將無法在您的工作階段主機上安裝代理程式更新，而您的工作階段主機將無法正常運作。

## <a name="register-the-virtual-machines-to-the-windows-virtual-desktop-host-pool"></a>向 Windows 虛擬桌面主機集區註冊虛擬機器

向 Windows 虛擬桌面主機集區註冊虛擬機器，就像安裝 Windows 虛擬桌面代理程式一樣簡單。

請在每部虛擬機器上執行下列作業，以註冊 Windows 虛擬桌面代理程式：

1. 使用建立虛擬機器時所提供的認證來[連線到虛擬機器](../virtual-machines/windows/quick-create-portal.md#connect-to-virtual-machine)。
2. 下載並安裝 Windows 虛擬桌面代理程式。
   - 下載 [Windows 虛擬桌面代理程式](https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrmXv)。
   - 執行安裝程式。 當安裝程式要求您輸入註冊權杖時，請輸入您從 **AzWvdRegistrationInfo Cmdlet 取得** 的值。
3. 下載並安裝 Windows 虛擬桌面代理程式開機載入器。
   - 下載 [Windows 虛擬桌面代理程式開機載入器](https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrxrH)。
   - 執行安裝程式。

>[!IMPORTANT]
>為了保護您在 Azure 中的 Windows 虛擬桌面環境，建議您不要在 VM 上開啟輸入連接埠 3389。 Windows 虛擬桌面不需要開啟輸入連接埠 3389 讓使用者存取主機集區的 VM。 如果您為了要進行疑難排解而必須開啟連接埠 3389，建議您使用 [Just-In-Time VM 存取](../security-center/security-center-just-in-time.md)。 我們也不建議將 VM 指派給公用 IP。

## <a name="update-the-agent"></a>更新代理程式

如果您是下列其中一種情況，您必須更新代理程式：

- 您想要將先前註冊的工作階段主機遷移至新的主機集區
- 更新之後，工作階段主機未出現在您的主機集區中

若要更新代理程式：

1. 以系統管理員身分登入 VM。
2. 移至 [ **服務**]，然後停止 **Rdagent** 和 **遠端桌面代理程式載入** 器進程。
3. 接下來，尋找代理程式和開機載入器 Msi。 它們會位於 **C:\DeployAgent** 資料夾中，或是您在安裝時所儲存的位置。
4. 尋找下列檔案並將它們卸載：
     
     - RDInfra. RDAgent. verx. x. x. x. x
     - RDInfra. RDAgentBootLoader. 安裝程式-x64

   若要卸載這些檔案，請在每個檔案名上按一下滑鼠右鍵，然後選取 [ **卸載**]。
5. （選擇性）您也可以移除下列登錄設定：
     
     - Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\RDInfraAgent
     - Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\RDAgentBootLoader

6. 卸載這些專案之後，這應該會移除與舊主機集區的所有關聯。 如果您想要將此主機重新註冊至服務，請遵循向 [Windows 虛擬桌面主機集區註冊虛擬機器](create-host-pools-powershell.md#register-the-virtual-machines-to-the-windows-virtual-desktop-host-pool)中的指示。


## <a name="next-steps"></a>後續步驟

建立好主機集區之後，即可在其中填入 RemoteApp。 若要深入了解如何管理 Windows 虛擬桌面中的應用程式，請參閱「管理應用程式群組」教學課程。

> [!div class="nextstepaction"]
> [管理應用程式群組教學課程](./manage-app-groups.md)
