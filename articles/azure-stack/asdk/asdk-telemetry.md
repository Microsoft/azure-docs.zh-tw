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
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/17/2018
ms.author: jeffgilb
ms.reviewer: misainat
ms.openlocfilehash: e50cb33acc95567a24afb19f88f88a3f586e0124
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47224745"
---
# <a name="azure-stack-telemetry"></a>Azure Stack 遙測

Azure Stack 系統資料或遙測會透過「已連線使用者體驗」自動上傳至 Microsoft。 從 Azure Stack 遙測收集的資料主要由 Microsoft 小組用來改善我們的客戶體驗，以及進行安全性、健康情況、品質和效能分析。

遙測為 Azure Stack 運算子，可以提供寶貴的企業部署深入解析，並提供有助於形塑未來 Azure Stack 版本的資訊。

> [!NOTE]
> 此外，也可以將 Azure Stack 設定為將使用量資訊轉送至 Azure 進行計費。 選擇隨用隨付計費方式的多節點 Azure Stack 客戶一定要這麼做。 使用量報告是經由遙測獨立進行控制，而選擇容量模式的多節點 Azure Stack 客戶或 Azure Stack 開發套件的使用者不需使用此功能。 針對上述案例，可以[使用註冊指令碼](https://docs.microsoft.com/azure/azure-stack/azure-stack-usage-reporting)來關閉使用量報告。

Azure Stack 遙測是以 Windows Server 2016「已連線使用者體驗與遙測」元件為基礎，該元件使用 [Windows 事件追蹤 (ETW)](https://msdn.microsoft.com/library/dn904632(v=vs.85).aspx) 追蹤記錄技術來蒐集和儲存遙測事件和資料。 Azure Stack 元件會使用相同的記錄技術，發佈使用公用作業系統事件記錄和追蹤 API 蒐集的事件和資料。 Azure Stack 元件範例包括網路資源提供者、儲存體資源提供者、監視資源提供者和更新資源提供者。 「已連線使用者體驗與遙測」元件使用 SSL 加密資料，並使用憑證關聯透過 HTTPS 將遙測資料傳輸至 Microsoft 資料管理服務。

> [!NOTE]
> 若要支援遙測資料流程，必須在您的網路中開放連接埠 443 (HTTPS)。 「已連線使用者體驗與遙測」元件會連線到 Microsoft 資料管理服務 (位於 https://v10.vortex-win.data.microsoft.com )。 「已連線使用者體驗與遙測」元件也會連線到 https://settings-win.data.microsoft.com 來下載組態資訊。

## <a name="privacy-considerations"></a>隱私權考量
ETW 服務會將遙測資料傳送回到受保護的雲端儲存體。 最小權限原則會支配遙測資料的存取權。 只有具備有效商務需求的 Microsoft 人員，才能夠存取遙測資料。 Microsoft 不會與第三方共用客戶個人資料，但客戶自行決定或基於 [Azure Stack 隱私權聲明](https://privacy.microsoft.com/PrivacyStatement)中所述的有限用途除外。 我們會與 OEM 和夥伴共用商務報告，其中包含匿名的彙總遙測資訊。 內部 Microsoft 小組 (包括隱私權、法務及資料管理利害關係人) 會進行資料共用決策。

Microsoft 相信並實踐資訊最小化。 我們努力只蒐集我們需要的資訊，而且只有在提供服務所需或進行分析時才會儲存它。 許多有關 Azure Stack 系統和 Azure 服務運作方式的資訊會在六個月內刪除。 摘要或彙總資料會保留更長一段時間。

我們了解客戶資訊的隱私權和安全性都非常重要。 我們採用了體貼且完善的方法，透過 Azure Stack 保護客戶隱私權和客戶資料。 IT 系統管理員有控制項可隨時自訂功能和隱私權設定。 我們對於透明度和信任的承諾很清楚：
- 我們對客戶公開我們所蒐集的資料類型。
- 我們會掌控企業客戶—他們可以自訂自己的隱私權設定。
- 我們將客戶隱私權和安全性放在第一位。
- 我們以公開透明的方式使用遙測。
- 我們使用遙測來改善客戶體驗。

Microsoft 不打算蒐集敏感資訊，例如信用卡號碼、使用者名稱和密碼、電子郵件地址或其他同樣敏感的資訊。 如果我們判斷不小心收到了敏感資訊，我們會予以刪除。

## <a name="examples-of-how-microsoft-uses-the-telemetry-data"></a>Microsoft 如何使用遙測資料的範例
遙測扮演重要的角色，可協助我們迅速找出並修正客戶部署和組態的重大可靠性問題。 深入解析我們所蒐集的遙測資料，可協助我們快速找出服務或硬體組態的問題。 Microsoft 向客戶取得此資料及推動生態系統改善的能力，有助於提高我們的整合式 Azure Stack 解決方案品質。

遙測也可協助 Microsoft 進一步了解客戶如何部署元件、使用功能，以及使用服務來達成業務目標。 取得該資料的深入解析，可協助您在會直接影響客戶體驗和工作負載的領域中，設定其工程投資的優先順序。

範例包括：用戶端的容器、儲存體使用量，以及與 Azure Stack 角色相關聯的網路組態。 我們也會使用深入解析來推動某些管理和監視解決方案的改善和智能運用。 這可協助客戶診斷品質問題，並透過進行較少 Microsoft 支援呼叫來節省金錢。

## <a name="manage-telemetry-collection"></a>管理遙測收集
我們不建議您在組織中關閉遙測，因為遙測可提供推動產品功能和穩定性改進的資料。 不過，我們承認在某些情況下，這可能是必要的。

在這些情況下，您可以使用部署前的登錄設定或使用部署後的遙測端點，設定傳送給 Microsoft 的遙測層級。

### <a name="set-telemetry-level-in-the-windows-registry"></a>在 Windows 登錄中設定遙測層級
在部署 Azure Stack 之前，Windows 登錄編輯程式用於在實體主機電腦上手動設定遙測層級。 如果管理原則 (例如群組原則) 已經存在，它會覆寫此登錄設定。

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

遙測層級會累積，可分類為四個層級 (0-3)：

**0 (安全性)**。 僅限安全性資料。 協助保護作業系統安全所需的資訊，包括「已連線使用者體驗與遙測」元件設定和 Windows Defender 相關資料。 在此層級不會發出任何 Azure Stack 特定遙測。

**1 (基本)**。 安全性資料，以及基本健康情況和品質資料。 基本裝置資訊，包括：品質相關資料、應用程式相容性、應用程式使用量資料，以及來自 [安全性] 層級的資料。 將您的遙測層級設定為 [基本]，即可啟用 Azure Stack 遙測。 在此層級蒐集的資料包括：

- **基本裝置資訊**，有助於讓您了解生態系統中原生和虛擬化 Windows Server 2016 執行個體的類型和組態，其中包括：
  - 機器屬性，例如 OEM、機號。
  - 網路屬性，例如網路介面卡的數目和速度。
  - 處理器和記憶體屬性，例如核心數、記憶體大小。
  - 儲存體屬性，例如磁碟機數目、類型和大小。
- **遙測功能**，包括已上傳事件、已卸除事件的百分比，以及最後上傳時間。
- **品質相關資訊**，可協助 Microsoft 初步了解 Azure Stack 的執行情況。 範例是特定硬體組態上的重大警示計數。
- **相容性資料**有助於讓您了解系統和虛擬機器上已安裝哪些資源提供者，並找出潛在的相容性問題。

**2 (增強)**。 額外深入解析，包括：作業系統和其他 Azure Stack 服務的使用方式、執行方式、進階可靠性資料，以及來自 [基本] 和 [安全性] 層級的資料。

**3 (完整)**。 找出及協助修正問題所需的全部資料，加上來自 [安全性]、[基本] 和 [增強] 層級的資料。

> [!NOTE]
> 預設遙測層級值為 2 (增強)。

關閉 Windows 和 Azure Stack 遙測會停用 SQL 遙測。 如需有關 Windows Server 遙測設定含意的詳細資訊，請參考 [Windows 遙測白皮書](https://aka.ms/winservtelemetry)。

> [!IMPORTANT]
> 這些遙測層級只適用於 Microsoft Azure Stack 元件。 Azure Stack 硬體夥伴在硬體生命週期主機中執行的非 Microsoft 軟體元件和服務，可能會與這些遙測層級之外的雲端服務通訊。 您應該與 Azure Stack 硬體解決方案提供者合作，了解其遙測原則，以及如何選擇加入或退出。

### <a name="enable-or-disable-telemetry-after-deployment"></a>在部署後啟用或停用遙測

若要在部署之後啟用或停用遙測，您必須能夠存取 ERCS VM 上公開的特殊權限端點 (PEP)。
1.  若要啟用：`Set-Telemetry -Enable`
2.  若要停用：`Set-Telemetry -Disable`

參數詳細資料：
> .PARAMETER Enable - 開啟遙測資料上傳

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
[新增市集項目](asdk-marketplace-item.md)
