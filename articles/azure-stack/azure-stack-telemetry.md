---
title: Azure Stack 遙測 | Microsoft Docs
description: 描述如何使用 PowerShell 進行 Azure Stack 遙測設定。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 10/15/2018
ms.author: jeffgilb
ms.reviewer: comartin
ms.openlocfilehash: 6b73cf04d768381bcc0e27cc76b6c2a25d4d9a2c
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49341050"
---
# <a name="azure-stack-telemetry"></a>Azure Stack 遙測

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

Azure Stack 遙測會透過「已連線使用者體驗」自動將系統資料上傳至 Microsoft。 Microsoft 小組會使用 Azure Stack 遙測蒐集的資料來改善客戶體驗。 此資料也用於安全性、健康狀態、品質和效能分析。

若為 Azure Stack 運算子，遙測可以提供寶貴的企業部署深入解析，並提供有助於形塑未來 Azure Stack 版本的資訊。

> [!NOTE]
> 您也可以將 Azure Stack 設定為將使用量資訊轉送至 Azure 進行計費。 選擇隨用隨付計費方式的多節點 Azure Stack 客戶一定要這麼做。 使用量報告是經由遙測獨立進行控制，而選擇容量模式的多節點客戶或 Azure Stack 開發套件的使用者不需使用此功能。 若為上述案例，可以[使用註冊指令碼](https://docs.microsoft.com/azure/azure-stack/azure-stack-usage-reporting)來關閉使用量報告。

Azure Stack 遙測是以 Windows Server 2016「已連線使用者體驗與遙測」元件為基礎，該元件使用 [Windows 事件追蹤 (ETW)](https://msdn.microsoft.com/library/dn904632(v=vs.85).aspx) 追蹤記錄技術來蒐集及儲存事件和資料。 Azure Stack 元件會使用相同的技術，發佈使用公用作業系統事件記錄和追蹤 API 所蒐集的事件和資料。 Azure Stack 元件範例包括下列提供者：網路資源、儲存體資源、監視資源和更新資源。 「已連線使用者體驗與遙測」元件使用 SSL 加密資料，並使用憑證關聯透過 HTTPS 將資料傳輸至 Microsoft 資料管理服務。

> [!IMPORTANT]
> 若要啟用遙測資料流程，必須在您的網路中開放連接埠 443 (HTTPS)。 「已連線使用者體驗與遙測」元件會連線到 Microsoft 資料管理服務 (位於 https://v10.vortex-win.data.microsoft.com )。 「已連線使用者體驗與遙測」元件也會連線到 https://settings-win.data.microsoft.com 來下載組態資訊。

## <a name="privacy-considerations"></a>隱私權考量

ETW 服務會將遙測資料傳送回到受保護的雲端儲存體。 最小權限原則會支配遙測資料的存取權。 只有具備有效商務需求的 Microsoft 人員，才能夠存取遙測資料。 Microsoft 不會與第三方共用客戶個人資料，但客戶自行決定或基於 [Microsoft 隱私權聲明](https://privacy.microsoft.com/PrivacyStatement)中所述的有限用途除外。 我們會與 OEM 和夥伴共用商務報告，其中包含匿名的彙總資料。 內部 Microsoft 小組 (包括隱私權、法務及資料管理利害關係人) 會進行資料共用決策。

Microsoft 相信並實踐資訊最小化。 我們努力以只蒐集所需的資訊為目標，而且只有在提供服務所需或進行分析時才會加以儲存。 許多有關 Azure Stack 系統和 Azure 服務運作方式的資訊會在六個月內刪除。 摘要或彙總資料將會保留更長一段時間。

我們了解客戶資訊的隱私權和安全性都很重要。  Microsoft 採用了體貼且完善的方法，在 Azure Stack 中保護客戶隱私權和客戶資料。 IT 系統管理員有控制項可隨時自訂功能和隱私權設定。 我們對於透明度和信任的承諾很清楚：

- 我們對客戶公開我們所蒐集的資料類型。
- 我們會掌控企業客戶 — 他們可以自訂自己的隱私權設定。
- 我們將客戶隱私權和安全性放在第一位。
- 我們以公開透明的方式使用遙測資料。
- 我們使用遙測資料來改善客戶體驗。

Microsoft 不打算蒐集敏感性資料，例如信用卡號碼、使用者名稱和密碼、電子郵件地址或類似的敏感性資訊。 如果我們判斷不小心收到了敏感資訊，我們會予以刪除。

## <a name="examples-of-how-microsoft-uses-the-telemetry-data"></a>Microsoft 如何使用遙測資料的範例

遙測扮演重要的角色，有助於迅速找出並修正客戶部署和組態的重大可靠性問題。 遙測資料見解可協助找出服務或硬體組態的問題。 Microsoft 向客戶取得此資料及推動生態系統改善的能力，可提高整合式 Azure Stack 解決方案的品質。

遙測也可協助 Microsoft 進一步了解客戶如何部署元件、使用功能，以及使用服務來達成業務目標。 這些見解有助於在會直接影響客戶體驗和工作負載的領域中，設定其工程投資的優先順序。

範例包括：客戶使用容器、儲存體，以及與 Azure Stack 角色相關聯的網路組態。 我們也會使用見解來推動 Azure Stack 管理和監視解決方案的改善和智能運用。 這些改善可讓客戶更輕鬆地診斷問題，並藉由進行較少 Microsoft 支援呼叫來節省金錢。

## <a name="manage-telemetry-collection"></a>管理遙測收集

我們不建議關閉您組織中的遙測。 不過，在某些情況下，這可能是必要的。

在這些情況下，您可以使用部署 Azure Stack 前的登錄設定，或使用部署 Azure Stack 後的遙測端點，設定傳送給 Microsoft 的遙測層級。

### <a name="telemetry-levels-and-data-collection"></a>遙測層級和資料收集

變更遙測設定前，您應該了解遙測層級和每個層級會收集哪些資料。

遙測設定可分為四個累計層級 (0-3)，其分類如下：

**0 (安全性)**</br>
僅限安全性資料。 保護作業系統安全所需的資訊。 這包括「已連線使用者體驗與遙測」元件設定和 Windows Defender 相關資料。 在此層級不會發出任何 Azure Stack 特定遙測。

**1 (基本)**</br>
安全性資料，以及基本健康情況和品質資料。 基本裝置資訊，包括：品質相關資料、應用程式相容性、應用程式使用量資料，以及來自 [安全性] 層級的資料。 將您的遙測層級設定為 [基本]，即可啟用 Azure Stack 遙測。 在此層級蒐集的資料包括：

- *基本裝置資訊*，可讓您了解生態系統中原生和虛擬 Windows Server 2016 執行個體的類型和組態。 其中包括：

  - 機器屬性，例如 OEM 和型號。
  - 網路屬性，例如網路介面卡的數目及其速度。
  - 處理器和記憶體屬性，例如核心數以及安裝的記憶體數量。
  - 儲存體屬性，例如磁碟機的數目、類型和大小。

- *遙測功能*，包括已上傳事件、已卸除事件的百分比，以及最後資料上傳時間。
- *品質相關資訊*，可協助 Microsoft 初步了解 Azure Stack 的執行情況。 例如，特定硬體組態上的重大警示計數。
- *相容性資料*有助於讓您了解系統和虛擬機器上已安裝哪些資源提供者。 這可找出潛在的相容性問題。

**2 (增強)**</br>
額外的見解，包括：作業系統和其他 Azure Stack 服務的使用方式、這些服務的執行方式、進階可靠性資料，以及來自 [安全性] 和 [基本] 層級的資料。

> [!NOTE]
> 這是預設遙測設定。

**3 (完整)**</br>
找出及協助修正問題所需的全部資料，加上來自 [安全性]、[基本] 和 [增強] 層級的資料。

> [!IMPORTANT]
> 這些遙測層級只適用於 Microsoft Azure Stack 元件。 Azure Stack 硬體夥伴在硬體生命週期主機中執行的非 Microsoft 軟體元件和服務，可能會與這些遙測層級之外的雲端服務通訊。 您應該與 Azure Stack 硬體解決方案提供者合作，了解其遙測原則，以及如何選擇加入或退出。

關閉 Windows 和 Azure Stack 遙測也會停用 SQL 遙測。 如需有關 Windows Server 遙測設定含意的詳細資訊，請參考 [Windows 遙測白皮書](https://aka.ms/winservtelemetry)。

### <a name="asdk-set-the-telemetry-level-in-the-windows-registry"></a>ASDK：在 Windows 登錄中設定遙測層級

在部署 Azure Stack 之前，您可以使用 Windows 登錄編輯程式在實體主機電腦上手動設定遙測層級。 如果管理原則 (例如群組原則) 已經存在，它會覆寫此登錄設定。

在開發套件主機上部署 Azure Stack 之前，先開機進入 CloudBuilder.vhdx，然後在已提高權限的 PowerShell 視窗中執行下列指令碼：

```powershell
### Get current AllowTelmetry value on DVM Host
(Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\DataCollection" `
-Name AllowTelemetry).AllowTelemetry
### Set & Get updated AllowTelemetry value for ASDK-Host
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\DataCollection" `
-Name "AllowTelemetry" -Value '0' # Set this value to 0,1,2,or3.  
(Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\DataCollection" `
-Name AllowTelemetry).AllowTelemetry
```

### <a name="asdk-and-multi-node-enable-or-disable-telemetry-after-deployment"></a>ASDK 和多節點：在部署後啟用或停用遙測

若要在部署之後啟用或停用遙測，您必須能夠存取 ERCS VM 上公開的特殊權限端點 (PEP)。

1. 若要啟用：`Set-Telemetry -Enable`
2. 若要停用：`Set-Telemetry -Disable`

PARAMETER 詳細資料：
> .PARAMETER Enable - 開啟遙測資料上傳</br>
> .PARAMETER Disable - 關閉遙測資料上傳  

**用於啟用遙測的指令碼：**

```powershell
$ip = "<IP ADDRESS OF THE PEP VM>" # You can also use the machine name instead of IP here.
$pwd= ConvertTo-SecureString "<CLOUD ADMIN PASSWORD>" -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential ("<DOMAIN NAME>\CloudAdmin", $pwd)
$psSession = New-PSSession -ComputerName $ip -ConfigurationName PrivilegedEndpoint -Credential $cred
Invoke-Command -Session $psSession {Set-Telemetry -Enable}
if($psSession)
{
    Remove-PSSession $psSession
}
```

**用於停用遙測的指令碼：**

```powershell
$ip = "<IP ADDRESS OF THE PEP VM>" # You can also use the machine name instead of IP here.
$pwd= ConvertTo-SecureString "<CLOUD ADMIN PASSWORD>" -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential ("<DOMAIN NAME>\CloudAdmin", $pwd)
$psSession = New-PSSession -ComputerName $ip -ConfigurationName PrivilegedEndpoint -Credential $cred
Invoke-Command -Session $psSession {Set-Telemetry -Disable}
if($psSession)
{
    Remove-PSSession $psSession
}
```

## <a name="next-steps"></a>後續步驟

[向 Azure 註冊 Azure Stack](azure-stack-registration.md)