---
title: 部署 Azure AD 密碼保護 (預覽)
description: 部署 Azure AD 密碼保護 (預覽) 以禁止在內部部署環境中使用錯誤的密碼
services: active-directory
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/25/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: jsimmons
ms.openlocfilehash: 5928896ab3c89972b7912f686be045afc988b1cd
ms.sourcegitcommit: cfff72e240193b5a802532de12651162c31778b6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2018
ms.locfileid: "39308870"
---
# <a name="preview-deploy-azure-ad-password-protection"></a>預覽：部署 Azure AD 密碼保護

|     |
| --- |
| Azure AD 密碼保護和自訂的禁用密碼清單是 Azure Active Directory 的公開預覽版功能。 如需有關預覽版的詳細資訊，請參閱 [Microsoft Azure 預覽版增補使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)|
|     |

既然我們已了解[如何強制執行適用於 Windows Server Active Directory 的 Azure AD 密碼保護](concept-password-ban-bad-on-premises.md)，下一步就是規劃並執行部署。

## <a name="deployment-strategy"></a>部署策略

Microsoft 建議進行任何部署時，都以稽核模式開始。 稽核模式是預設的初始設定，除了可以繼續設定密碼之外，任何會被封鎖的行為都會在事件記錄檔中建立項目。 以稽核模式完全部署 Proxy 伺服器和 DC 代理程式之後，應該執行定期監視，以便判斷當強制執行密碼原則時，密碼原則強制執行會對使用者和環境造成什麼影響。

在稽核階段，許多組織都發現：

* 它們必須改善現有的作業程序，才能使用更安全的密碼。
* 使用者習慣於定期選擇不安全的密碼
* 它們必須通知使用者關於即將推出的安全性強制執行變更、此變更可能對他們造成的影響，並協助他們更深入了解如何選擇更安全的密碼。

以稽核模式執行此功能達一段合理的時間之後，強制執行設定即可從 [稽核] 翻轉成 [強制]，從而要求更安全的密碼。 在此期間進行專注的監視是個不錯的主意。

## <a name="known-limitation"></a>已知的限制

Azure AD 密碼保護 Proxy 預覽版有一個已知的限制。 不支援使用需要 MFA 的租用戶系統管理員帳戶。 Azure AD 密碼保護 Proxy 的未來更新將會支援使用需要 MFA 的系統管理員帳戶來進行 Proxy 註冊。

## <a name="single-forest-deployment"></a>單一樹系部署

Azure AD 密碼保護預覽版已搭配在兩部伺服器上部署 Proxy 服務，而且 DC 代理程式服務能以遞增方式部署至 Active Directory 樹系中的所有網域控制站。

![Azure AD 密碼保護元件如何一起運作](./media/concept-password-ban-bad-on-premises/azure-ad-password-protection.png)

### <a name="download-the-software"></a>下載軟體

Azure AD 密碼保護有兩個可從 [Microsoft 下載](https://www.microsoft.com/download/details.aspx?id=57071)的必要安裝程式

### <a name="install-and-configure-the-azure-ad-password-protection-proxy-service"></a>安裝並設定 Azure AD 密碼保護 Proxy 服務

1. 選擇一或多個要裝載 Azure AD 密碼保護 Proxy 服務的伺服器。
   * 每個這類服務只能為一個樹系提供密碼原則，而且主機電腦必須已加入該樹系中某個網域 (同時支援根網域和子網域) 的網域。 為了讓 Azure AD 密碼保護 Proxy 服務完成其任務，樹系的每個網域中必須至少有一個 DC 與 Azure AD 密碼保護 Proxy 主機電腦之間有網路連線。
   * 支援在網域控制站上安裝並執行 Azure AD 密碼保護 Proxy 服務來進行測試，但網域控制站必須要有網際網路連線能力。

   > [!NOTE]
   > 公開預覽版支援每一樹系最多 2 個 Proxy 伺服器。

2. 使用 AzureADPasswordProtectionProxy.msi MSI 套件來安裝「密碼原則 Proxy 服務」軟體。
   * 軟體安裝並不需要重新開機。 您可以使用標準 MSI 程序來自動執行軟體安裝，例如：`msiexec.exe /i AzureADPasswordProtectionProxy.msi /quiet /qn`

3. 以系統管理員身分開啟 PowerShell 視窗。
   * Azure AD 密碼保護 Proxy 軟體包含一個名為 AzureADPasswordProtection 的新 PowerShell 模組。 下列步驟就是以從這個 PowerShell 模組執行各種 Cmdlet 為基礎，並假設您已開啟一個新的 PowerShell 視窗，而且已依下列方式匯入新的模組：
      * `Import-Module AzureADPasswordProtection`

      > [!NOTE]
      > 此安裝軟體會修改主機電腦的 PSModulePath 環境變數。 若要讓此變更生效，以便匯入及使用 AzureADPasswordProtection PowerShell 模組，您可能需要開啟一個全新的 PowerShell 主控台視窗。

   * 使用下列 PowerShell 命令來確認服務是否正在執行：`Get-Service AzureADPasswordProtectionProxy | fl`。
      * 結果應該會產生一個 **Status** 傳回 "Running" 的結果。

4. 註冊 Proxy。
   * 完成步驟 3 之後，Azure AD 密碼保護 Proxy 服務就會在電腦上執行，但還沒有與 Azure AD 通訊所需的必要認證。 必須使用 `Register-AzureADPasswordProtectionProxy` PowerShell Cmdlet 向 Azure AD 註冊，才能啟用該功能。 此 Cmdlet 除了樹系根網域中的內部部署 Active Directory 網域系統管理員權限之外，也需要您 Azure 租用戶的全域管理員認證。 所指定 Proxy 服務的註冊成功之後，額外的 `Register-AzureADPasswordProtectionProxy` 引動過程也會跟著執行成功，但並非必要。
      * 此 Cmdlet 可以依以下方式執行：

         ```
         $tenantAdminCreds = Get-Credential
         Register-AzureADPasswordProtectionProxy -AzureCredential $tenantAdminCreds
         ```

         只有當目前登入的使用者也是根網域的 Active Directory 網域系統管理員時，此範例才能運作。 有一個替代方案，就是透過 `-ForestCredential` 參數提供必要的網域認證。

   > [!TIP]
   > 第一次為指定的 Azure 租用戶執行此 Cmdlet 時，在 Cmdlet 執行完成之前，可能會有相當長的延遲 (很多秒)。 除非系統回報失敗，否則不應該將此延遲視為警示。

   > [!NOTE]
   > Azure AD 密碼保護 Proxy 服務的註冊步驟在服務的生命週期中應該只需執行一次。 之後，此 Proxy 服務就會自動執行所有其他必要的維護。 所指定樹系的註冊成功之後，額外的 'Register-AzureADPasswordProtectionProxy' 引動過程也會跟著執行成功，但並非必要。

5. 註冊樹系。
   * 內部部署 Active Directory 樹系必須使用 `Register-AzureADPasswordProtectionForest` PowerShell Cmdlet 以必要的認證初始化，才能與 Azure 進行通訊。 此 Cmdlet 除了樹系根網域中的內部部署 Active Directory 網域系統管理員權限之外，也需要您 Azure 租用戶的全域管理員認證。 此步驟會針對每一樹系執行一次。
      * 此 Cmdlet 可以依以下方式執行：

         ```
         $tenantAdminCreds = Get-Credential
         Register-AzureADPasswordProtectionForest -AzureCredential $tenantAdminCreds
         ```

         只有當目前登入的使用者也是根網域的 Active Directory 網域系統管理員時，此範例才能運作。 有一個替代方案，就是透過 -ForestCredential 參數提供必要的網域認證。

         > [!NOTE]
         > 如果您的環境中安裝了多個 Proxy 伺服器，則在上述程序中指定哪個 Proxy 伺服器無關緊要。

         > [!TIP]
         > 第一次為指定的 Azure 租用戶執行此 Cmdlet 時，在 Cmdlet 執行完成之前，可能會有相當長的延遲 (很多秒)。 除非系統回報失敗，否則不應該將此延遲視為警示。

   > [!NOTE]
   > Active Directory 樹系的註冊步驟在樹系的生命週期中應該只需執行一次。 之後，在樹系中執行的網域控制站代理程式就會自動執行所有其他必要的維護。 所指定樹系的註冊成功之後，額外的 `Register-AzureADPasswordProtectionForest` 引動過程也會跟著執行成功，但並非必要。

   > [!NOTE]
   > 為了讓 `Register-AzureADPasswordProtectionForest` 可成功執行，Proxy 伺服器的網域中至少要有一個 Windows Server 2012 或更新版本的網域控制站可用。 不過，在此步驟之前，不需要在任何網域控制站上安裝 DC 代理程式軟體。

6. 選擇性：將 Azure AD 密碼保護 Proxy 服務設定成在特定連接埠上進行接聽。
   * 網域控制站上的 Azure AD 密碼保護 DC 代理程式軟體會使用「透過 TCP 的 RPC」與 Azure AD 密碼保護 Proxy 服務進行通訊。 Azure AD 密碼保護密碼原則 Proxy 服務預設會在任何可用的動態 RPC 端點上進行接聽。 因網路拓撲或防火牆需求而有必要時，可以改為將服務設定為在特定的 TCP 連接埠上進行接聽。
      * 若要將服務設定成在靜態連接埠下執行，請使用 `Set-AzureADPasswordProtectionProxyConfiguration` Cmdlet。
         ```
         Set-AzureADPasswordProtectionProxyConfiguration –StaticPort <portnumber>
         ```

         > [!WARNING]
         > 您必須將服務停止後再重新啟動，這些變更才會生效。

      * 若要將服務設定成在動態連接埠下執行，請使用相同的程序，但必須將 StaticPort 設回零，像這樣：
         ```
         Set-AzureADPasswordProtectionProxyConfiguration –StaticPort 0
         ```

         > [!WARNING]
         > 您必須將服務停止後再重新啟動，這些變更才會生效。

   > [!NOTE]
   > 對連接埠設定進行任何變更之後，都必須手動重新啟動 Azure AD 密碼保護 Proxy 服務。 進行此性質的設定變更之後，並不需要重新啟動在網域控制站上執行的 DC 代理程式服務軟體。

   * 您可以使用 `Get-AzureADPasswordProtectionProxyConfiguration` Cmdlet 來查詢目前的服務設定，如以下範例所示：

      ```
      Get-AzureADPasswordProtectionProxyConfiguration | fl

      ServiceName : AzureADPasswordProtectionProxy
      DisplayName : Azure AD password protection Proxy
      StaticPort  : 0 
      ```

### <a name="install-the-azure-ad-password-protection-dc-agent-service"></a>安裝 Azure AD 密碼保護 DC 代理程式服務

* 使用 `AzureADPasswordProtectionDCAgent.msi` MSI 套件來安裝 Azure AD 密碼保護 DC 代理程式服務軟體：
   * 由於作業系統要求只能在重新開機時載入或卸載密碼篩選 dll，因此軟體安裝在安裝和解除安裝時確實需要重新開機。
   * 支援在尚未成為網域控制站的電腦上安裝 DC 代理程式服務。 在此情況下，服務將會啟動並執行，但會處於非作用中，直到電腦被升階成網域控制站為止。

   您可以使用標準 MSI 程序來自動執行軟體安裝，例如：`msiexec.exe /i AzureADPasswordProtectionDCAgent.msi /quiet /qn`

   > [!WARNING]
   > 此範例 msiexec 命令會導致立即重新開機；只要指定 `/norestart` 旗標，即可避免發生此情況。

安裝在網域控制站上並重新開機之後，Azure AD 密碼保護 DC 代理程式軟體安裝便已完成。 任何其他設定都無須進行，也無法進行。

## <a name="multiple-forest-deployments"></a>多個樹系部署

沒有任何要在多個樹系部署 Azure AD 密碼保護的額外需求。 如單一樹系部署小節所述，每個樹系都是以獨立方式設定的。 每個 Azure AD 密碼保護 Proxy 都只能支援其已加入樹系中的網域控制站。 不論 Active Directory 信任設定為何，所指定樹系中的 Azure AD 密碼保護軟體都不會知道部署在另一個樹系中的 Azure AD 密碼保護軟體。

## <a name="read-only-domain-controllers"></a>唯讀網域控制站

系統一律不會在唯讀網域控制站 (RODC) 上處理和保存密碼變更\設定；而是會轉送給可寫入的網域控制站進行處理和保存。 因此，不需要在 RODC 上安裝 DC 代理程式軟體。

## <a name="high-availability"></a>高可用性

當樹系中的網域控制站嘗試從 Azure 下載新原則或其他資料時，若要確保 Azure AD 密碼保護的高可用性，主要考量將會是 Proxy 伺服器的可用性。 公開預覽版支援每一樹系最多 2 個 Proxy 伺服器。 每個 DC 代理程式在決定要呼叫的 Proxy 伺服器時，都是使用簡單的循環配置資源型演算法，並且會略過沒有回應的 Proxy 伺服器。
與高可用性相關的尋常問題可藉由 DC 代理程式軟體的設計來解決。 DC 代理程式會保有最近所下載密碼原則的本機快取。 即使因任何原因而導致所有註冊的 Proxy 伺服器都變成無法使用，DC 代理程式仍會繼續強制執行其已快取的密碼原則。 大型部署中合理的密碼原則更新頻率通常是幾天，而非幾小時或更少。   因此，短暫的 Proxy 伺服器服務中斷並不會對 Azure AD 密碼保護功能或其安全性優點造成重大影響。

## <a name="next-steps"></a>後續步驟

既然您已在內部部署伺服器上安裝 Azure AD 密碼保護所需的服務，現在請完成[安裝後設定並收集報告資訊](howto-password-ban-bad-on-premises-operations.md)，以完成您的部署。

[Azure AD 密碼保護的概念性概觀](concept-password-ban-bad-on-premises.md)
