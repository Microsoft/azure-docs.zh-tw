---
title: 連線到 Azure Stack | Microsoft Docs
description: 了解如何連線到 Azure Stack。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.assetid: 3cebbfa6-819a-41e3-9f1b-14ca0a2aaba3
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 06/06/2018
ms.author: mabrigg
ms.openlocfilehash: c1932f2ed0486fb56e467466c0fed53702e8f9b0
ms.sourcegitcommit: 50f82f7682447245bebb229494591eb822a62038
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/08/2018
ms.locfileid: "35248681"
---
# <a name="connect-to-azure-stack-development-kit"></a>連線至 Azure Stack 開發套件

適用於：Azure Stack 開發套件

若要管理資源，您必須先連線到「Azure Stack 開發套件」。 在本文中，我們會說明連線至開發套件要採取的步驟。 您可以使用下列其中一個連線選項：

* [遠端桌面連線](#connect-with-remote-desktop)。 當您使用「遠端桌面連線」進行連線時，單一使用者可以快速連線至開發套件。
* [虛擬私人網路 (VPN)](#connect-with-vpn)。 當您使用 VPN 進行連線時，多名使用者可以同時從 Azure Stack 基礎結構外部的用戶端進行連線。 VPN 連線需要進行某些設定。

<a name="connect-to-azure-stack-with-remote-desktop"></a>
##  <a name="connect-to-azure-stack-by-using-remote-desktop-connection"></a>使用遠端桌面連線來連線到 Azure Stack

單一並行使用者可以透過「遠端桌面連線」在操作員入口網站或使用者入口網站中管理資源。

1. 開啟「遠端桌面連線」並連線到開發套件。 針對使用者名稱，輸入 **AzureStack\AzureStackAdmin**。 使用您在設定 Azure Stack 時指定的操作員密碼。  

2. 在開發套件電腦上，開啟 [伺服器管理員]。 選取 [本機伺服器]，清除 [Internet Explorer 增強式安全性] 核取方塊，然後關閉 [伺服器管理員]。

3. 若要開啟[使用者入口網站](azure-stack-key-features.md#portal)，請移至 https://portal.local.azurestack.external/。 使用使用者認證來登入。 若要開啟 Azure Stack [操作員入口網站](azure-stack-key-features.md#portal)，請移至 https://adminportal.local.azurestack.external/。 使用您在安裝期間指定的 Azure Active Directory (Azure AD) 認證來登入。

<a name="connect-to-azure-stack-with-vpn"></a>
## <a name="connect-to-azure-stack-by-using-vpn"></a>使用 VPN 來連線到 Azure Stack

您可以建立一個連線至「Azure Stack 開發套件」的分割通道 VPN 連線。 您可以使用 VPN 連線來存取 Azure Stack 操作員入口網站、使用者入口網站及安裝在本機的工具 (例如 Visual Studio 和 PowerShell)，來管理 Azure Stack 資源。 Azure AD 和 Active Directory 同盟服務 (AD FS) 部署都支援 VPN 連線。 VPN 連線可讓多個用戶端同時連線到 Azure Stack。

> [!NOTE]
> VPN 連線無法連線到 Azure Stack 基礎結構 VM。

### <a name="prerequisites"></a>先決條件

1. 在您的本機電腦上安裝 [Azure Stack 相容的 Azure PowerShell](azure-stack-powershell-install.md)。  
2. 下載[處理 Azure Stack 所需的工具](azure-stack-powershell-download.md)。

### <a name="set-up-vpn-connectivity"></a>設定 VPN 連線能力

若要建立與開發套件的 VPN 連線，請在以 Windows 為基礎的本機電腦上，以系統管理員身分開啟 Windows PowerShell。 然後，執行下列指令碼 (針對您的環境更新 IP 位址和密碼值)：

```PowerShell
# Configure Windows Remote Management (WinRM), if it's not already configured.
winrm quickconfig  

Set-ExecutionPolicy RemoteSigned

# Import the Connect module.
Import-Module .\Connect\AzureStack.Connect.psm1

# Add the development kit computer’s host IP address and certificate authority (CA) to the list of trusted hosts. Make sure you update the IP address and password values for your environment.

$hostIP = "<Azure Stack host IP address>"

$Password = ConvertTo-SecureString `
  "<operator's password provided when deploying Azure Stack>" `
  -AsPlainText `
  -Force

Set-Item wsman:\localhost\Client\TrustedHosts `
  -Value $hostIP `
  -Concatenate

# Create a VPN connection entry for the local user.
Add-AzsVpnConnection `
  -ServerAddress $hostIP `
  -Password $Password

```

如果設定成功，**azurestack** 會出現在您的 VPN 連線清單。

![網路連線](media/azure-stack-connect-azure-stack/image3.png)  

### <a name="connect-to-azure-stack"></a>連線至 Azure Stack

請使用下列其中一個方法來連線到 Azure Stack 執行個體：  

* 使用 `Connect-AzsVpn ` 命令：
    
  ```PowerShell
  Connect-AzsVpn `
    -Password $Password
  ```

  出現提示時，請信任 Azure Stack 主機，然後將來自 **AzureStackCertificateAuthority** 的憑證安裝到您本機電腦的憑證存放區中。 (提示可能會被 PowerShell 視窗隱藏。)

* 在您的本機電腦上，選取 [網路設定] > [VPN] > [azurestack] > [連線]。 出現登入提示時，輸入使用者名稱 (**AzureStack\AzureStackAdmin**) 和密碼。

### <a name="test-vpn-connectivity"></a>測試 VPN 連線能力

若要測試入口網站連線，請開啟網頁瀏覽器，然後移至使用者入口網站 (https://portal.local.azurestack.external/)) 或操作員入口網站 (https://adminportal.local.azurestack.external/))。 登入並建立資源。  

## <a name="next-steps"></a>後續步驟

[將虛擬機器提供給您的 Azure Stack 使用者](azure-stack-tutorial-tenant-vm.md)
