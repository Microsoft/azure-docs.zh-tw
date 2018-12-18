---
title: 連線到 Azure Stack | Microsoft Docs
description: 了解如何連線到 Azure Stack
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
ms.date: 08/22/2017
ms.author: mabrigg
ms.openlocfilehash: 21015d31a738d3ad57048fe4a703bf78dda7e40c
ms.sourcegitcommit: ab3b2482704758ed13cccafcf24345e833ceaff3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/06/2018
ms.locfileid: "37865765"
---
# <a name="connect-to-azure-stack"></a>連線至 Azure Stack

若要管理資源，您必須連線到「Azure Stack 開發套件」。 此文將詳細說明連線到開發套件所需的步驟。 您可以使用下列其中一個連線選項：

* [遠端桌面](#connect-with-remote-desktop)：可讓單一並行使用者從開發套件快速連線。
* [虛擬私人網路 (VPN)](#connect-with-vpn)：可讓多個並行使用者從 Azure Stack 基礎結構外部的用戶端連線 (需要設定)。

## <a name="connect-to-azure-stack-with-remote-desktop"></a>使用遠端桌面來連線到 Azure Stack
使用「遠端桌面」連線時，單一並行使用者可以使用入口網站來管理資源。

1. 開啟一個「遠端桌面連線」並連線到開發套件。 輸入 **AzureStack\AzureStackAdmin** 作為使用者名稱，並輸入您在 Azure Stack 設定期間所提供的系統管理密碼。  

2. 從開發套件電腦開啟 [伺服器管理員]，按一下 [本機伺服器]、關閉 [Internet Explorer 增強式安全性]，然後關閉 [伺服器管理員]。

3. 若要開啟入口網站，請巡覽至 (https://portal.local.azurestack.external/)，並使用使用者認證來登入。


## <a name="connect-to-azure-stack-with-vpn"></a>使用 VPN 來連線到 Azure Stack

您可以建立一個連至「Azure Stack 開發套件」的分割通道「虛擬私人網路」(VPN) 連線。 透過 VPN 連線，您可以存取系統管理員入口網站、使用者入口網站及安裝在本機的工具 (例如 Visual Studio 和 PowerShell)，以管理 Azure Stack 資源。 Azure Active Directory(AAD) 和「Active Directory 同盟服務」(AD FS) 型部署都支援 VPN 連線。 VPN 連線可讓多個用戶端同時連線到 Azure Stack。 

> [!NOTE] 
> 這個 VPN 連線並無法連線到 Azure Stack 基礎結構 VM。 

### <a name="prerequisites"></a>先決條件

* 在您的本機電腦上安裝 [Azure Stack 相容的 Azure PowerShell](azure-stack-powershell-install.md)。  
* 下載[處理 Azure Stack 所需的工具](azure-stack-powershell-download.md)。 

### <a name="configure-vpn-connectivity"></a>設定 VPN 連線

若要建立連至開發套件的 VPN 連線，請從您的本機 Windows 型電腦開啟已提高權限的 PowerShell 工作階段，然後執行下列指令碼 (請務必針對您的環境更新 IP 位址和密碼值)：

```PowerShell 
# Configure winrm if it's not already configured
winrm quickconfig  

Set-ExecutionPolicy RemoteSigned

# Import the Connect module
Import-Module .\Connect\AzureStack.Connect.psm1 

# Add the development kit computer’s host IP address & certificate authority (CA) to the list of trusted hosts. Make sure to update the IP address and password values for your environment. 

$hostIP = "<Azure Stack host IP address>"

$Password = ConvertTo-SecureString `
  "<Administrator password provided when deploying Azure Stack>" `
  -AsPlainText `
  -Force

Set-Item wsman:\localhost\Client\TrustedHosts `
  -Value $hostIP `
  -Concatenate

# Create a VPN connection entry for the local user
Add-AzsVpnConnection `
  -ServerAddress $hostIP `
  -Password $Password

```

如果設定成功，您應該會在 VPN 連線清單中看到 **azurestack**。

![網路連線](media/azure-stack-connect-azure-stack/image3.png)  

### <a name="connect-to-azure-stack"></a>連線至 Azure Stack

請使用下列兩種方法之一來連線到 Azure Stack 執行個體：  

* 透過使用 `Connect-AzsVpn ` 命令： 
    
  ```PowerShell
  Connect-AzsVpn `
    -Password $Password
  ```

  出現提示時，請信任 Azure Stack 主機，然後將來自 **AzureStackCertificateAuthority** 的憑證安裝到您本機電腦的憑證存放區中。 (提示可能會出現在 PowerShell 工作階段視窗後面)。 

* 開啟您本機電腦的 [網路設定] > [VPN] > 按一下 [azurestack] > [連線]。 出現登入提示時，輸入使用者名稱 (AzureStack\AzureStackAdmin) 和密碼。

### <a name="test-the-vpn-connectivity"></a>測試 VPN 連線

若要測試入口網站連線，請開啟網際網路瀏覽器，巡覽至使用者入口網站 (https://portal.local.azurestack.external/)，然後登入並建立資源。  

## <a name="next-steps"></a>後續步驟



