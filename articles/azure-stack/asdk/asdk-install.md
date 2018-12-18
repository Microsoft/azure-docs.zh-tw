---
title: 安裝 Azure Stack 開發套件 (ASDK) | Microsoft Docs
description: 說明如何安裝 Azure Stack 開發套件 (ASDK)。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/10/2018
ms.author: jeffgilb
ms.reviewer: misainat
ms.openlocfilehash: 2bdda273a32167f70633096d463be59884eca033
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44718216"
---
# <a name="install-the-azure-stack-development-kit-asdk"></a>安裝 Azure Stack 開發套件 (ASDK)
[準備 ASDK 主機電腦](asdk-prepare-host.md)後，可以使用本文中的下列步驟將 ASDK 部署到 CloudBuilder.vhdx 映像。

## <a name="install-the-asdk"></a>安裝 ASDK
本文中的步驟會說明如何使用藉由下載並執行 **asdk-installer.ps1** PowerShell 指令碼所提供的圖形化使用者介面 (GUI) 來部署 ASDK。

> [!NOTE]
> Azure Stack 開發套件的安裝程式使用者介面是以 WCF 和 PowerShell 為基礎的開放來源指令碼。


1. 在主機電腦成功開機進入 CloudBuilder.vhdx 映像之後，使用您在[準備 ASDK 安裝的開發套件主機電腦](asdk-prepare-host.md)時所指定的系統管理員認證進行登入。 這應該與開發套件主機本機系統管理員認證相同。
2. 開啟已提升權限的 PowerShell 主控台並執行 **&lt;磁碟機代號>\AzureStack_Installer\asdk-installer.ps1** 指令碼 (目前可能是在 Cloudbuilder.vhdx 映像中 C:\ 以外的其他磁碟機上)。 按一下 [Install] 。

    ![](media/asdk-install/1.PNG) 

3. 在識別提供者 [類型] 下拉式方塊中，選取 [Azure 雲端] 或 [AD FS]。 在 [本機系統管理員密碼] 底下的 [密碼] 方塊中，輸入本機系統管理員密碼 (必須符合目前設定的本機系統管理員密碼)，然後按 [下一步]。
    - **Azure 雲端**：將 Azure Active Directory (Azure AD) 設定為識別提供者。 若要使用此選項，您需有網際網路連線、Azure AD 目錄租用戶的完整名稱 (採用 *domainname*.onmicrosoft.com 形式或 Azure AD 驗證的自訂網域名稱)，以及所指定目錄的全域管理員認證。 部署之後，就不需要 Azure Active Directory 全域管理員權限。 不過，某些作業可能需要全域管理員認證。 例如，需要獲派權限的資源提供者安裝程式指令碼或新功能。 您可以暫時恢復帳戶的全域管理員權限，或使用擁有「預設提供者訂用帳戶」的個別全域管理員帳戶。
    - **AD FS**：預設戳記目錄服務會作為識別提供者。 用於登入的預設帳戶是 azurestackadmin@azurestack.local，而要使用的密碼是您在設定過程中所提供的密碼。

    ![](media/asdk-install/2.PNG) 
    
    > [!NOTE]
    > 為獲得最佳結果，即使您想要使用以 AD FS 作為識別提供者且已中斷連線的 Azure Stack 環境，最好還是在已連線到網際網路時安裝 ASDK。 如此一來，就可以在部署期間啟動開發套件安裝隨附的 Windows Server 2016 評估版。
4. 選取要用於開發套件的網路介面卡，然後按一下 [下一步]。

    ![](media/asdk-install/3.PNG)

5. 為 BGPNAT01 虛擬機器選取 DHCP 或靜態網路組態。
    > [!TIP]
    > BGPNAT01 VM 是邊緣路由器，其提供適用於 Azure Stack 的 NAT 和 VPN 功能。

    - **DHCP** (預設值)：虛擬機器會從 DHCP 伺服器取得 IP 網路組態。
    - **靜態**：只有當 DHCP 無法為 Azure Stack 指派有效的 IP 位址來存取網際網路時，才使用此選項。 **指定靜態 IP 位址 時，必須以 CIDR 格式 (例如，10.0.0.5/24) 一併指定子網路遮罩長度**。
    - 輸入有效的**時間伺服器 IP** 位址。 此必填欄位會設定開發套件所要使用的時間伺服器。 此參數必須以有效的時間伺服器 IP 位址形式提供。 不支援伺服器名稱。

      > [!TIP]
      > 若要尋找時間伺服器 IP 位址，請造訪 [pool.ntp.org](http://pool.ntp.org) 或 ping time.windows.com。 

    - **(選擇性)** 您可以設定下列值：
        - **VLAN 識別碼**：設定 VLAN 識別碼。 只有當主機與 AzS-BGPNAT01 必須設定 VLAN 識別碼來存取實體網路 (以及網際網路) 時，才使用此選項。 
        - **DNS 轉寄站**：DNS 伺服器會在 Azure Stack 部署期間建立。 若要允許解決方案內的電腦解析戳記以外的名稱，請提供您現有架構 DNS 伺服器。 戳記內的 DNS 伺服器會將未知的名稱解析要求轉送至這部伺服器。

    ![](media/asdk-install/4.PNG)

6. 在 [驗證網路介面卡內容] 頁面上，您將會看到一個進度列。 驗證完成時，按 [下一步]。

    ![](media/asdk-install/5.PNG)

9. 在 [摘要] 頁面上，按一下 [部署] 以在開發套件主機電腦上開始 ASDK 安裝。

    ![](media/asdk-install/6.PNG)

    > [!TIP]
    > 您也可以在此複製將用於安裝開發套件的 PowerShell 設定命令。 如果您需要[使用 PowerShell 在主機電腦上重新部署 ASDK](asdk-deploy-powershell.md)，此動作會有所助益。

10. 如果您執行的是 Azure AD 部署，系統會在設定開始後幾分鐘，提示您輸入 Azure AD 全域系統管理員帳戶認證。

    ![](media/asdk-install/7.PNG)

11. 部署程序會花上幾小時的時間，在這段期間內，主機電腦會自動重新開機一次。 如果您想要監視部署進度，在開發套件主機重新啟動後，請以 azurestack\AzureStackAdmin 的身分登入。 當部署成功時，PowerShell 主控台會顯示：**COMPLETE: Action ‘Deployment’**。 
    > [!IMPORTANT]
    > 如果您在電腦加入網域之後，以本機系統管理員的身分登入，將不會看到部署進度。 請勿重新執行部署，而是以 azurestack\AzureStackAdmin 的身分登入，以驗證執行狀態。

    ![](media/asdk-install/8.PNG)

恭喜，您已順利安裝 ASDK！

若部署因某些原因失敗，您可以從頭[重新部署](asdk-redeploy.md)或使用下列 PowerShell 命令，在已提升權限的相同 PowerShell 視窗中，從最後一個成功的步驟重新開始部署：

  ```powershell
  cd C:\CloudDeployment\Setup
  .\InstallAzureStackPOC.ps1 -Rerun
  ```

## <a name="next-steps"></a>後續步驟
[部署後的組態](asdk-post-deploy.md)
