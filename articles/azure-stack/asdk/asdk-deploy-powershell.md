---
title: 部署 Azure Stack - PowerShell | Microsoft Docs
description: 在本文中，您會使用 PowerShell 從命令列安裝 ASDK。
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
ms.custom: ''
ms.date: 09/10/2018
ms.author: jeffgilb
ms.reviewer: misainat
ms.openlocfilehash: c6b2387360973cd4e65b5a1e4ba483abf5ea9070
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44716021"
---
# <a name="deploy-the-asdk-from-the-command-line"></a>從命令列部署 ASDK
ASDK 是種測試和部署環境，可供您部署以評估及示範 Azure Stack 的功能和服務。 要讓其正常執行，您將需要準備環境硬體並執行一些指令碼 (這將需要數小時的時間)。 在那之後，您就可以登入系統管理員和使用者入口網站，開始使用 Azure Stack。

## <a name="prerequisites"></a>必要條件 
準備開發套件主機電腦。 規劃您的硬體、軟體及網路。 裝載開發套件的電腦 (開發套件主機) 必須符合硬體、軟體及網路需求。 此外，您還必須在使用 Azure Active Directory (Azure AD) 或「Active Directory 同盟服務 (AD FS)」之間選擇。 開始部署之前，請務必符合這些先決條件，如此安裝程序才能順暢執行。 

部署 ASDK 之前，請先確定您所規劃的開發套件主機電腦硬體、作業系統和網路設定，都符合安裝 ASDK 的最低需求。

**[檢閱 ASDK 部署規劃考量](asdk-deploy-considerations.md)**

> [!TIP]
> 安裝作業系統之後，您可以使用 [Azure Stack 部署需求檢查工具](https://gallery.technet.microsoft.com/Deployment-Checker-for-50e0f51b)來確認您的硬體符合所有需求。

## <a name="download-and-extract-the-deployment-package"></a>下載部署套件並解壓縮
確認您的開發套件主機電腦符合安裝 ASDK 的基本需求後，下一個步驟是下載 ASDK 部署套件並解壓縮。 開發套件包括 Cloudbuilder.vhdx 檔案，這是個包含可開機作業系統和 Azure Stack 安裝檔的虛擬硬碟。

您可以將部署套件下載到開發套件主機或另一部電腦。 解壓縮後的部署檔案會佔用 60 GB 的可用磁碟空間，因此使用另一部電腦可協助降低開發套件主機的硬體需求。

**[下載 Azure Stack 開發套件 (ASDK) 並解壓縮](asdk-download.md)**

## <a name="prepare-the-development-kit-host-computer"></a>準備開發套件主機電腦
必須先備妥環境，並將系統設定為從 VHD 開機，才可以在主機電腦上安裝 ASDK。 在此步驟之後，開發套件主機會開機進入 Cloudbuilder.vhdx (一個包含可開機作業系統和 Azure Stack 安裝檔的虛擬硬碟)。

使用 PowerShell 將 ASDK 主機電腦設定為從 CloudBuilder.vhdx 開機。 這些命令會將 ASDK 主機電腦設定為從已下載並解壓縮的 Azure Stack 虛擬硬碟 (CloudBuilder.vhdx) 開機。 完成這些步驟之後，重新啟動 ASDK 主機電腦。

若要將 ASDK 主機電腦設定為從 CloudBuilder.vhdx 開機：

  1. 以系統管理員身分啟動命令提示字元。
  2. 執行 `bcdedit /copy {current} /d "Azure Stack"`
  3. 複製 (CTRL + C) 傳回的 CLSID 值，包括必要的 {}。 這個值也就是 {CLSID}，且必須在其餘步驟中貼入 (CTRL + V 或按一下滑鼠右鍵)。
  4. 執行 `bcdedit /set {CLSID} device vhd=[C:]\CloudBuilder.vhdx` 
  5. 執行 `bcdedit /set {CLSID} osdevice vhd=[C:]\CloudBuilder.vhdx` 
  6. 執行 `bcdedit /set {CLSID} detecthal on` 
  7. 執行 `bcdedit /default {CLSID}`
  8. 若要驗證開機設定，請執行 `bcdedit`。 
  9. 確定 CloudBuilder.vhdx 檔案已移至 C:\ 磁碟機的根目錄 (C:\CloudBuilder.vhdx)，並重新啟動開發套件主機電腦。 ASDK 主機電腦重新啟動時，應該會從 CloudBuilder.vhdx 虛擬機器硬碟開機，開始 ASDK 部署。 

> [!IMPORTANT]
> 請先確保您可直接實際存取開發套件主機電腦，或可進行 KVM 存取，然後才重新啟動。 虛擬機器第一次啟動時，會提示您完成 Windows Server 安裝。 此時請提供您用來登入開發套件主機電腦的相同系統管理員認證。 

### <a name="prepare-the-development-kit-host-using-powershell"></a>使用 PowerShell 來準備開發套件主機 
開發套件主機電腦成功開機進入 CloudBuilder.vhdx 映像後，請以您用來登入開發套件主機電腦的相同本機系統管理員認證來登入 (這也是您在主機電腦從 VHD 開機時，要完成 Windows Server 安裝的最後一部分所提供的認證)。 

> [!NOTE]
> 或者，您也可以在安裝 ASDK 前，「先」設定 [Azure Stack 遙測設定](asdk-telemetry.md#set-telemetry-level-in-the-windows-registry)。

開啟提升權限的 PowerShell 主控台，並執行本節中所述的命令，將 ASDK 部署到開發套件主機上。

> [!IMPORTANT] 
> ASDK 安裝支援正好一個網路介面卡 (NIC) 進行網路運作。 如果您有多個 NIC，在執行部署指令碼之前，請先確定僅啟用一個 NIC (並停用其他所有 NIC)。

您可使用 Azure AD 或 Windows Server AD FS 作為識別提供者來部署 Azure Stack。 Azure Stack、資源提供者和其他應用程式使用這兩者的方式相同。

> [!TIP]
> 如果您未提供任何設定參數 (請參閱 InstallAzureStackPOC.ps1 選用參數和下方範例)，系統會提示您輸入必要參數。

### <a name="deploy-azure-stack-using-azure-ad"></a>使用 Azure AD 部署 Azure Stack 
若要**使用 Azure AD 作為識別提供者**來部署 Azure Stack，您必須具有網際網路連線能力 (直接或透過透明 Proxy)。 

使用 Azure AD，執行下列 PowerShell 命令來部署開發套件：

  ```powershell
  cd C:\CloudDeployment\Setup     
  $adminpass = Get-Credential Administrator     
  .\InstallAzureStackPOC.ps1 -AdminPassword $adminpass.Password
  ```

進入 ASDK 安裝幾分鐘，系統會提示您輸入 Azure AD 認證。 您必須提供 Azure AD 租用戶的全域管理員認證。 

部署之後，就不需要 Azure Active Directory 全域管理員權限。 不過，某些作業可能需要全域管理員認證。 例如，需要獲派權限的資源提供者安裝程式指令碼或新功能。 您可以暫時恢復帳戶的全域管理員權限，或使用擁有「預設提供者訂用帳戶」的個別全域管理員帳戶。

### <a name="deploy-azure-stack-using-ad-fs"></a>使用 AD FS 部署 Azure Stack 
若要**使用 AD FS 作為識別提供者**來部署開發套件，請執行下列 PowerShell 命令 (您只需要新增 -UseADFS 參數)： 

  ```powershell
  cd C:\CloudDeployment\Setup     
  $adminpass = Get-Credential Administrator 
  .\InstallAzureStackPOC.ps1 -AdminPassword $adminpass.Password -UseADFS
  ```

在 AD FS 部署中，預設戳記目錄服務會作為識別提供者。 用來登入的預設帳戶是 azurestackadmin@azurestack.local，而密碼將會設定為您在 PowerShell 設定命令中提供的密碼。

部署程序可能需要幾小時的時間，在這段期間內，系統會自動重新開機一次。 當部署成功時，PowerShell 主控台會顯示：**COMPLETE: Action ‘Deployment’**。 如果部署失敗，您可以使用 -rerun 參數再次嘗試執行指令碼。 或者，您可以從頭開始[重新部署 ASDK](asdk-redeploy.md)。

> [!IMPORTANT]
> 如果您想要監視部署進度，在 ASDK 主機重新開機之後，您必須以 AzureStack\AzureStackAdmin 身分登入。 如果您在主機電腦重新啟動 (並加入 azurestack.local 網域) 之後，以本機系統管理員的身分登入，將不會看到部署進度。 請勿重新執行部署，而是以 azurestack 的身分登入來驗證執行狀態。


#### <a name="azure-ad-deployment-script-examples"></a>Azure AD 部署指令碼範例
您可以為整個 Azure AD 部署轉寫指令碼。 以下是一些加上註解且包含部分選用參數的範例。

如果您的 Azure AD 身分識別只與**一個** Azure AD 目錄相關聯：
```powershell
cd C:\CloudDeployment\Setup 
$adminpass = Get-Credential Administrator 
$aadcred = Get-Credential "<Azure AD global administrator account name>" 
.\InstallAzureStackPOC.ps1 -AdminPassword $adminpass.Password -InfraAzureDirectoryTenantAdminCredential $aadcred -TimeServer 52.168.138.145 #Example time server IP address.
```

如果您的 Azure AD 身分識別與**一個以上的** Azure AD 目錄相關聯：
```powershell
cd C:\CloudDeployment\Setup 
$adminpass = Get-Credential Administrator 
$aadcred = Get-Credential "<Azure AD global administrator account name>" #Example: user@AADDirName.onmicrosoft.com 
.\InstallAzureStackPOC.ps1 -AdminPassword $adminpass.Password -InfraAzureDirectoryTenantAdminCredential $aadcred -InfraAzureDirectoryTenantName "<Azure AD directory in the form of domainname.onmicrosoft.com or an Azure AD verified custom domain name>" -TimeServer 52.168.138.145 #Example time server IP address.
```

如果您的環境並未啟用 DHCP，則您必須將下列其他參數包含在上述其中一個選項 (已提供範例使用方式)： 

```powershell
.\InstallAzureStackPOC.ps1 -AdminPassword $adminpass.Password -InfraAzureDirectoryTenantAdminCredential $aadcred -NatIPv4Subnet 10.10.10.0/24 -NatIPv4Address 10.10.10.3 -NatIPv4DefaultGateway 10.10.10.1 -TimeServer 10.222.112.26
```

### <a name="asdk-installazurestackpocps1-optional-parameters"></a>ASDK InstallAzureStackPOC.ps1 選用參數
|參數|必要/選用|說明|
|-----|-----|-----|
|AdminPassword|必要|在開發套件部署過程中建立的所有虛擬機器上，設定本機系統管理員帳戶和其他所有使用者帳戶。 此密碼必須符合主機上的目前本機系統管理員密碼。|
|InfraAzureDirectoryTenantName|必要|設定租用戶目錄。 使用此參數來指定 AAD 帳戶有權限管理多重目錄的特定目錄。 AAD 目錄租用戶的完整名稱 (採用 .onmicrosoft.com 形式或 Azure AD 驗證的自訂網域名稱)。|
|TimeServer|必要|使用此參數來指定特定時間伺服器。 此參數必須以有效的時間伺服器 IP 位址形式提供。 不支援伺服器名稱。|
|InfraAzureDirectoryTenantAdminCredential|選用|設定 Azure Active Directory 使用者名稱與密碼。 這些 Azure 認證必須是組織識別碼。|
|InfraAzureEnvironment|選用|選取您要用來註冊此 Azure Stack 部署的 Azure 環境。 選項包括公用 Azure、Azure - 中國、Azure - 美國政府。|
|DNSForwarder|選用|DNS 伺服器會在 Azure Stack 部署期間建立。 若要允許解決方案內的電腦解析戳記以外的名稱，請提供您現有架構 DNS 伺服器。 戳記內的 DNS 伺服器會將未知的名稱解析要求轉送至這部伺服器。|
|NatIPv4Address|DHCP NAT 支援所需|設定 MAS-BGPNAT01 的靜態 IP 位址。 只有當 DHCP 無法指派有效的 IP 位址來存取網際網路時，才使用此參數。|
|NatIPv4Subnet|DHCP NAT 支援所需|透過 NAT 支援用於 DHCP 的 IP 子網路前置詞。 只有當 DHCP 無法指派有效的 IP 位址來存取網際網路時，才使用此參數。|
|PublicVlanId|選用|設定 VLAN 識別碼。 只有當主機與 MAS-BGPNAT01 必須設定 VLAN 識別碼來存取實體網路 (以及網際網路) 時，才使用此參數。 例如，.\InstallAzureStackPOC.ps1 -Verbose -PublicVLan 305|
|Rerun|選用|使用這個旗標來重新執行部署。 系統會使用所有先前的輸入。 不支援重新輸入先前提供的資料，因為會產生數個唯一值並使用於部署。|


## <a name="perform-post-deployment-configurations"></a>執行部署後設定
安裝 ASDK 後，建議您進行一些安裝後的檢查，並變更一些設定。 您可以使用 test-AzureStack Cmdlet 來驗證您的安裝是否成功，還有安裝 Azure Stack PowerShell 和 GitHub 工具。 

您也應該重設密碼到期原則，以確保開發套件主機的密碼不會在評估期間結束前到期。

> [!NOTE]
> 或者，您也可以在「安裝 ASDK 後」，再設定 [Azure Stack 遙測設定](asdk-telemetry.md#enable-or-disable-telemetry-after-deployment)。

**[部署 ASDK 後的工作](asdk-post-deploy.md)**

## <a name="register-with-azure"></a>向 Azure 註冊
您必須向 Azure 註冊 Azure Stack，如此您才能夠[將 Azure 市集項目下載到 Azure Stack](asdk-marketplace-item.md)。

**[向 Azure 註冊 Azure Stack](asdk-register.md)**

## <a name="next-steps"></a>後續步驟
恭喜！ 完成這些步驟之後，您將會擁有一個具備[系統管理員](https://adminportal.local.azurestack.external)和[使用者](https://portal.local.azurestack.external)入口網站的開發套件環境。 

[ASDK 安裝後設定工作](asdk-post-deploy.md)

