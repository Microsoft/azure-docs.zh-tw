---
title: StorSimple 8000 系列部署問題的疑難排解 | Microsoft Docs
description: 說明如何診斷和修正您第一次部署 StorSimple 時發生的錯誤。
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 07/03/2017
ms.author: alkohli
ms.openlocfilehash: 61719d482a4db1c737bbe38277f2ac3b2d684b63
ms.sourcegitcommit: 4597964eba08b7e0584d2b275cc33a370c25e027
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2018
ms.locfileid: "37342420"
---
# <a name="troubleshoot-storsimple-device-deployment-issues"></a>StorSimple 裝置部署問題的疑難排解
## <a name="overview"></a>概觀
本文提供對於 Microsoft Azure StorSimple 部署很有幫助的疑難排解指引。 文中將說明常見問題、可能原因和建議的步驟，可協助您解決在設定 StorSimple 時可能遇到的問題。 

此資訊適用於 StorSimple 8000 系列實體裝置和 StorSimple 雲端設備。

> [!NOTE]
> 您可能會在第一次部署裝置時面臨裝置設定的相關問題，或者它們可能在裝置處於運作狀態時發生。 本文著重於第一次部署問題的疑難排解。 若要對可運作裝置進行疑難排解，請移至[使用診斷工具對可運作裝置進行疑難排解](storsimple-8000-diagnostics.md)。

本文也會說明適用於疑難排解 StorSimple 部署的工具，並提供疑難排解的逐步範例。

## <a name="first-time-deployment-issues"></a>第一次部署問題
如果您在第一次部署裝置時發生問題，請考慮下列各項：

* 如果您正在疑難排解實體裝置，請確定硬體已安裝，並遵循[安裝您的 StorSimple 8100 裝置](storsimple-8100-hardware-installation.md)或[安裝您的 StorSimple 8600 裝置](storsimple-8600-hardware-installation.md)中所述的方式進行設定。
* 檢查部署的先決條件。 請確定您已備妥 [部署檢查清單](storsimple-8000-deployment-walkthrough-u2.md#deployment-configuration-checklist)中描述的所有資訊。
* 檢閱 StorSimple 版本資訊，以查看是否提及該問題。 版本資訊會包含已知安裝問題的因應措施。 

在裝置部署期間，使用者最常面臨的問題通常是在執行安裝精靈，以及透過 Windows PowerShell for StorSimple 註冊裝置時發生。 (您會使用 Windows PowerShell for StorSimple 來註冊並設定 StorSimple 裝置。 如需裝置註冊的詳細資訊，請參閱 [步驟 3：透過 Windows PowerShell for StorSimple 設定和註冊裝置](storsimple-8000-deployment-walkthrough-u2.md#step-3-configure-and-register-the-device-through-windows-powershell-for-storsimple))。

下列各節可協助您解決在第一次設定 StorSimple 裝置時遇到的問題。

## <a name="first-time-setup-wizard-process"></a>第一次執行安裝精靈程序
下列步驟概述安裝精靈程序。 如需詳細的安裝資訊，請參閱 [部署內部部署 StorSimple 裝置](storsimple-8000-deployment-walkthrough-u2.md)。

1. 執行 [Invoke-HcsSetupWizard](https://technet.microsoft.com/library/dn688135.aspx) Cmdlet 來啟動安裝精靈，此精靈將引導您完成其餘步驟。 
2. 設定網路：安裝精靈可讓您針對 StorSimple 裝置上的 DATA 0 網路介面進行網路設定。 這些設定包括下列各項：
   * 虛擬 IP (VIP)、子網路遮罩及閘道 - [Set-HcsNetInterface](https://technet.microsoft.com/library/dn688161.aspx) Cmdlet 會在背景中執行。 它會針對 StorSimple 裝置上的 DATA 0 網路介面設定 IP 位址、子網路遮罩及閘道。
   * 主要 DNS 伺服器 - [Set-HcsDnsClientServerAddress](https://technet.microsoft.com/library/dn688172.aspx) Cmdlet 會在背景中執行。 它會設定適用於 StorSimple 解決方案的 DNS 設定。
   * NTP 伺服器 - [Set-HcsNtpClientServerAddress](https://technet.microsoft.com/library/dn688138.aspx) Cmdlet 會在背景中執行。 它會設定適用於 StorSimple 解決方案的 NTP 伺服器設定。
   * 選用的 Web Proxy - [Set-HcsWebProxy](https://technet.microsoft.com/library/dn688154.aspx) Cmdlet 會在背景中執行。 它會設定適用於 StorSimple 解決方案的 Web Proxy 設定並加以啟用。
3. 設定密碼：下一個步驟是設定裝置系統管理員密碼。
   裝置系統管理員密碼可用來登入您的裝置。 預設裝置密碼是 **Password1**。
        
     > [!IMPORTANT]
     > 密碼是在註冊之前收集，但只有在您成功註冊裝置之後才會套用。 如果套用密碼失敗，系統將提示您再次提供密碼，直到收集到所需的密碼 (符合複雜性需求) 為止。
     
4. 註冊裝置：最後一個步驟是使用在 Microsoft Azure 中執行的 StorSimple 裝置管理員服務來註冊裝置。 註冊會要求您從 Azure 入口網站 [取得服務註冊金鑰](storsimple-8000-manage-service.md#get-the-service-registration-key)，並在安裝精靈中提供該金鑰。 **成功註冊裝置之後，即會提供您服務資料加密金鑰。請務必將此加密金鑰保留在安全的位置，因為在向服務註冊所有後續裝置時都需用到它。**

## <a name="common-errors-during-device-deployment"></a>裝置部署期間的常見錯誤
下表列出當您進行下列動作時可能遇到的常見錯誤：

* 設定所需的網路設定。
* 設定選用的 Web Proxy 設定。
* 設定裝置系統管理員密碼。
* 註冊裝置。

## <a name="errors-during-the-required-network-settings"></a>所需的網路設定期間發生錯誤
| 否。 | 錯誤訊息 | 可能的原因 | 建議的動作 |
| --- | --- | --- | --- |
| 1 |Invoke-HcsSetupWizard：這個命令只能在主動控制器上執行。 |設定是在被動控制站上執行。 |從主動控制器執行這個命令。 如需詳細資訊，請參閱 [識別裝置上的主動控制器](storsimple-8000-controller-replacement.md#identify-the-active-controller-on-your-device)。 |
| 2 |Invoke-HcsSetupWizard：裝置尚未就緒。 |在 DATA 0 上發生網路連線問題。 |檢查 DATA 0 上的實體網路連線。 |
| 3 |Invoke-HcsSetupWizard：有一個 IP 位址與網路上的另一個系統發生衝突 (例外狀況發生於 HRESULT：0x80070263)。 |提供給 DATA 0 的 IP 已經由另一個系統使用中。 |提供未使用的新 IP。 |
| 4 |Invoke-HcsSetupWizard：某個叢集資源失敗了 (例外狀況發生於 HRESULT：0x800713AE)。 |重複的 VIP。 提供的 IP 已經在使用中。 |提供未使用的新 IP。 |
| 5 |Invoke-HcsSetupWizard: 無效的 IPv4 位址。 |提供的 IP 位址格式不正確。 |檢查格式，然後再次提供您的 IP 位址。 如需詳細資訊，請參閱 [Ipv4 定址][1]。 |
| 6 |Invoke-HcsSetupWizard：無效的 IPv6 位址。 |提供的 IP 位址格式不正確。 |檢查格式，然後再次提供您的 IP 位址。 如需詳細資訊，請參閱 [Ipv6 定址][2]。 |
| 7 |Invoke-HcsSetupWizard：端點對應器中無更多可用的端點。 (例外狀況發生於 HRESULT：0x800706D9) |叢集功能無法運作。 |[連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md) 以進行後續步驟。 |

## <a name="errors-during-the-optional-web-proxy-settings"></a>選用的 Web Proxy 設定期間發生錯誤
| 編號 | 錯誤訊息 | 可能的原因 | 建議的動作 |
| --- | --- | --- | --- |
| 1 |Invoke-HcsSetupWizard：無效的參數 (例外狀況發生於 HRESULT：0x80070057) |針對 Proxy 設定提供的其中一個參數無效。 |未使用正確格式提供 URI。 請使用下列格式：//*<IP address or FQDN of the web proxy server>*:*<TCP port number>* |
| 2 |Invoke-HcsSetupWizard：RPC 伺服器無法使用 (例外狀況發生於 HRESULT：0x800706ba) |根本原因是下列其中一項︰<ol><li>叢集未啟動。</li><li>被動控制器無法與主動控制器通訊，而命令是從被動控制器執行。</li></ol> |依據根本原因︰<ol><li>[連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md) 以確定叢集已啟動。</li><li>從主動控制器執行這個命令。 如果您想要從被動控制器執行命令，就必須確保被動控制器能與主動控制器通訊。 如果此連線已中斷，您必須[連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md)。</li></ol> |
| 3 |Invoke-HcsSetupWizard：RPC 呼叫失敗 (例外狀況發生於 HRESULT：0x800706be) |叢集已關閉。 |[連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md) 以確定叢集已啟動。 |
| 4 |Invoke-HcsSetupWizard：找不到叢集資源 (例外狀況發生於 HRESULT：0x8007138f) |找不到叢集資源。 若未正確安裝，即會發生此情況。 |您可能需要將裝置重設為出廠預設設定。 [連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md) 以建立叢集資源。 |
| 5 |Invoke-HcsSetupWizard：叢集資源不在線上 (例外狀況發生於 HRESULT：0x8007138c) |叢集資源不在線上。 |[連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md) 以進行後續步驟。 |

## <a name="errors-related-to-device-administrator-password"></a>與裝置系統管理員密碼相關的錯誤
預設裝置系統管理員密碼是 **Password1**。 此密碼會在第一次登入之後過期；因此，您將需要使用安裝精靈來變更它。 當您第一次註冊裝置時，必須提供新的裝置系統管理員密碼。 

請確定您的密碼符合下列需求：

* 裝置系統管理員密碼長度應介於 8 到 15 個字元。
* 密碼應該包含下列 4 個字元類型的其中 3 個：小寫字母、大寫字母、數字和特殊字元。 
* 您的密碼不能與最近的 24 個密碼相同。

此外，請記住，密碼每年都會過期，而您只能在成功註冊裝置之後才能加以變更。 如果註冊因任何因素而失敗，密碼將不會變更。

如需裝置系統管理員密碼的詳細資訊，請移至[使用 StorSimple 裝置管理員服務變更 StorSimple 密碼](storsimple-8000-change-passwords.md)。

在設定裝置系統管理員和 StorSimple Snapshot Manager 密碼時，可能會遇到下列一或多個錯誤。

| 否。 | 錯誤訊息 | 建議的動作 |
| --- | --- | --- |
| 1 |密碼超過最大長度。 |您的裝置系統管理員密碼長度必須介於 8 到 15 個字元。 |
| 2 |密碼不符合所需的長度。 |您的裝置系統管理員密碼長度必須介於 8 到 15 個字元。|
| 3 |密碼必須包含小寫字元。 |密碼必須包含下列 4 個字元類型的其中 3 個：小寫字母、大寫字母、數字和特殊字元。 請確定您的密碼符合這些需求。 |
| 4 |密碼必須包含數字字元。 |密碼必須包含下列 4 個字元類型的其中 3 個：小寫字母、大寫字母、數字和特殊字元。 請確定您的密碼符合這些需求。 |
| 5 |密碼必須包含特殊字元。 |密碼必須包含下列 4 個字元類型的其中 3 個：小寫字母、大寫字母、數字和特殊字元。 請確定您的密碼符合這些需求。 |
| 6 |密碼必須包含下列 4 個字元類型的其中 3 個：大寫字母、小寫字母、數字和特殊字元。 |您的密碼未包含必要的字元類型。 請確定您的密碼符合這些需求。 |
| 7 |參數與確認不相符。 |請確定您的密碼符合所有需求，且您已正確輸入它。 |
| 8 |您的密碼與預設值不符。 |預設密碼為 *Password1*。 您需要在第一次登入之後變更這個密碼。 |
| 9 |您輸入的密碼與裝置密碼不符。 請重新輸入密碼。 |檢查密碼並再次輸入。 |

密碼是在裝置註冊之前收集，但只有在您成功註冊之後才會套用。 密碼復原工作流程需要註冊裝置。

> [!IMPORTANT]
> 一般而言，如果嘗試套用密碼失敗，則此軟體會重複嘗試收集密碼，直到成功為止。 有時會無法套用密碼，但很罕見。 在此情況下，您可以註冊裝置並繼續進行，但密碼將不會變更。 從 Azure 入口網站註冊之後，您可以變更裝置系統管理員密碼。


您可以透過 StorSimple 裝置管理員服務重設 Azure 入口網站中的密碼。 如需詳細資訊，請移至[變更裝置系統管理員密碼](storsimple-8000-change-passwords.md#change-the-device-administrator-password)。

## <a name="errors-during-device-registration"></a>裝置註冊期間發生錯誤
您使用在 Microsoft Azure 中執行的 StorSimple 裝置管理員服務來註冊裝置。 您可能會在裝置註冊期間遇到下列一或多個問題。

| 否。 | 錯誤訊息 | 可能的原因 | 建議的動作 |
| --- | --- | --- | --- |
| 1 |錯誤 350027：無法向 StorSimple 裝置管理員註冊裝置。 | |等候幾分鐘的時間，然後再次嘗試操作。 如果問題持續發生， 請 [連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md)。 |
| 2 |錯誤 350013：註冊裝置時發生錯誤。 這可能是因為服務註冊金鑰不正確而引發。 | |請使用正確的服務註冊金鑰再次註冊裝置。 如需詳細資訊，請參閱 [取得服務註冊金鑰](storsimple-8000-manage-service.md#get-the-service-registration-key) |
| 3 |錯誤 350063：StorSimple 裝置管理員服務通過驗證，但註冊失敗。 請在一段時間之後重試此操作。 |此錯誤表示已通過 ACS 驗證，但對服務所做的註冊呼叫失敗。 這可能是零星網路問題的結果。 |如果問題持續發生，請 [連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md)。 |
| 4 |錯誤 350049：無法在註冊期間連線服務。 |對服務進行呼叫時，收到 Web 例外狀況。 在某些情況下，稍後重試此操作或許可以修正此錯誤。 |請檢查您的 IP 位址和 DNS 名稱，然後重試此操作。 如果問題持續發生，請 [連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md) |
| 5 |錯誤 350031：裝置已經註冊。 | |不需採取任何動作。 |
| 6 |錯誤 350016：裝置註冊失敗。 | |請確定註冊金鑰正確。 |
| 7 |Invoke-HcsSetupWizard：註冊裝置時發生錯誤。這可能是因為 IP 位址或 DNS 名稱不正確所引發。 請檢查您的網路設定，然後再試一次。 如果問題持續發生，請 [contact Microsoft Support](storsimple-8000-contact-microsoft-support.md)。 (錯誤 350050) |確定裝置可以 Ping 外部網路。 如果您沒有外部網路的連線能力，註冊可能會失敗並產生這個錯誤。 這個錯誤可能是下列一或多個項目的組合：<ul><li>不正確的 IP</li><li>不正確的子網路</li><li>不正確的閘道</li><li>不正確的 DNS 設定</li></ul> |請參閱 [逐步疑難排解範例](#step-by-step-storsimple-troubleshooting-example)中的步驟。 |
| 8 |Invoke-HcsSetupWizard：由於發生內部服務錯誤 [0x1FBE2]，導致目前的作業失敗。 請稍後再重試操作。 如果問題持續發生， 請連絡 Microsoft 支援服務。 |這是從服務或代理程式針對所有使用者看不見的錯誤所擲回的一般錯誤。 最常見的原因可能是 ACS 驗證失敗。 失敗的原因可能是發生 NTP 伺服器設定問題，而且未正確設定裝置上的時間。 |更正時間 (如果有問題)，然後重試註冊操作。 如果您使用 Set-HcsSystem -Timezone 命令來調整時區，請將時區中的每個字大寫 (例如 "Pacific Standard Time")。  如果問題持續發生，請 [連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md) 以進行後續步驟。 |
| 9 |警告：無法啟動裝置。 您的設定裝置系統管理員和 StorSimple Snapshot Manager 密碼尚未變更。 |如果註冊失敗，裝置系統管理員和 StorSimple Snapshot Manager 密碼就不會變更。 | |

## <a name="tools-for-troubleshooting-storsimple-deployments"></a>適用於疑難排解 StorSimple 部署的工具
StorSimple 隨附數個工具，可用來疑難排解 StorSimple 解決方案。 其中包含：

* 支援封裝和裝置記錄。
* 專為疑難排解而設計的 Cmdlet。

## <a name="support-packages-and-device-logs-available-for-troubleshooting"></a>可用來疑難排解的支援封裝和裝置記錄
支援封裝包含可協助 Microsoft 支援小組疑難排解裝置問題的所有相關記錄。 您可以使用 Windows PowerShell for StorSimple，來產生您之後可與支援人員共用的加密支援封裝。

### <a name="to-view-the-logs-or-the-contents-of-the-support-package"></a>檢視支援封裝的記錄或內容
1. 使用 Windows PowerShell for StorSimple 產生支援封裝，如 [建立及管理支援封裝](storsimple-8000-create-manage-support-package.md)所述。
2. 在用戶端電腦上本機下載 [解密指令碼](https://gallery.technet.microsoft.com/scriptcenter/Script-to-decrypt-a-a8d1ed65) 。
3. 使用這個 [逐步程序](storsimple-8000-create-manage-support-package.md#edit-a-support-package) 來開啟和解密支援封裝。
4. 解密的支援封裝記錄格式為 etw/etvx 格式。 您可以執行下列步驟，在 Windows 事件檢視器中檢視這些檔案：
   
   1. 在 Windows 用戶端上執行 **eventvwr** 命令。 這將會啟動事件檢視器。
   2. 在 [動作] 窗格中，按一下 [開啟已儲存的記錄]，然後指向 etvx/etw 格式的記錄檔 (支援封裝)。 您現在可以檢視該檔案。 開啟檔案之後，您可以按一下滑鼠右鍵並將該檔案儲存為文字。
      
      > [!IMPORTANT]
      > 您也可以使用 **Get-WinEvent** Cmdlet，在 Windows PowerShell 中開啟這些檔案。 如需詳細資訊，請參閱 Windows PowerShell Cmdlet 參考文件中的 [Get-WinEvent](https://technet.microsoft.com/library/hh849682.aspx) 。
     
5. 在事件檢視器中開啟記錄時，尋找下列包含裝置設定相關問題的記錄：
   
   * hcs_pfconfig/Operational Log
   * hcs_pfconfig/Config
6. 在記錄檔中，搜尋與安裝精靈所呼叫的 Cmdlet 相關的字串。 如需這些 Cmdlet 的清單，請參閱 [第一次安裝精靈程序](#first-time-setup-wizard-process) 。
7. 如果您找不出問題的原因，可 [連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md) 以進行後續步驟。 當您連絡 Microsoft 支援服務以尋求協助時，請使用 [建立支援要求](storsimple-8000-contact-microsoft-support.md#create-a-support-request) 中的步驟。

## <a name="cmdlets-available-for-troubleshooting"></a>可用於疑難排解的 Cmdlet
請使用下列 Windows PowerShell Cmdlet 來偵測連線錯誤。

* `Get-NetAdapter`：使用這個 Cmdlet 來偵測網路介面的健康情況。
* `Test-Connection`：使用這個 Cmdlet 來檢查網路內部和外部的網路連線。
* `Test-HcsmConnection`：使用這個 Cmdlet 來檢查已成功註冊裝置的連線。
* `Sync-HcsTime`：執行此 Cmdlet 可顯示裝置的時間，並強制與 NTP 伺服器同步時間。
* `Enable-HcsPing` 與 `Disable-HcsPing`：這兩個 Cmdlet 可讓主機傳送 Ping 給您 StorSimple 裝置上的網路介面。 StorSimple 網路介面依預設不會回應 Ping 要求。
* `Trace-HcsRoute`：此 Cmdlet 可做為路由追蹤工具。 每過一段時間，它就會在前往最終目的地時將封包傳送到每個路由器，然後根據每個躍點傳回的封包計算結果。 由於 `Trace-HcsRoute` 會顯示封包在任何指定的路由器或連結中遺失的程度，因此您可以找出導致網路發生問題的路由器或連結。
* `Get-HcsRoutingTable`：此 Cmdlet 可顯示本機 IP 的路由表。

## <a name="troubleshoot-with-the-get-netadapter-cmdlet"></a>Get-NetAdapter Cmdlet 的疑難排解
當您在第一次裝置部署期間設定網路介面時，無法在 StorSimple 裝置管理員服務 UI 中取得硬體狀態，因為裝置尚未向該服務註冊。 此外，[硬體健康狀態] 刀鋒視窗不一定能夠正確反映裝置狀態，尤其是發生了會影響服務同步處理的問題。 在這些情況下，您可以使用 `Get-NetAdapter` Cmdlet，來判斷網路介面的健康情況和狀態。

### <a name="to-see-a-list-of-all-the-network-adapters-on-your-device"></a>查看裝置上所有網路介面卡的清單
1. 啟動 Windows PowerShell for StorSimple，然後鍵入 `Get-NetAdapter`。 
2. 使用 `Get-NetAdapter` Cmdlet 的輸出和下列指南來了解網路介面的狀態。
   
   * 如果介面的健康狀態良好且已啟用，**ifIndex** 狀態就會顯示為 **Up**。
   * 如果介面的健康狀態良好但實際上並未連線 (透過網路纜線)，**ifIndex** 就會顯示為 **Disabled**。
   * 如果介面的健康狀態良好但未啟用，**ifIndex** 狀態就會顯示為 **NotPresent**。
   * 如果介面不存在，它就不會出現在這份清單中。 StorSimple Manager 裝置管理員服務 UI 仍會顯示此介面處於失敗狀態。

如需如何使用這個 Cmdlet 的詳細資訊，請參閱 Windows PowerShell Cmdlet 參考中的 [Get-NetAdapter](https://docs.microsoft.com/powershell/module/netadapter/get-netadapter?view=win10-ps)。

下列各節示範來自 `Get-NetAdapter` Cmdlet 的輸出範例。

 在這些範例中，控制器 0 是被動控制站，並已使用如下的方式進行設定：

* 裝置上存在著 DATA 0、DATA 1、DATA 2 及 DATA 3 網路介面。
* DATA 4 和 DATA 5 網路介面卡並不存在，因此，它們不會列在輸出中。
* DATA 0 已啟用。

控制器 1 是主動控制器，並已使用如下的方式進行設定：

* 裝置上存在著 DATA 0、DATA 1、DATA 2、DATA 3、DATA 4 及 DATA 5 網路介面。
* DATA 0 已啟用。

**範例輸出 - 控制站 0**

以下是來自控制站 0 (被動控制站) 的輸出。 未連接 DATA 1、DATA 2 及 DATA 3。 不會列出 DATA 4 和 DATA 5，因為它們不存在於裝置上。

     Controller0>Get-NetAdapter
     Name                 InterfaceDescription                        ifIndex  Status
     ------               --------------------                        -------  ----------
     DATA3                Mellanox ConnectX-3 Ethernet Adapter #2     17       NotPresent
     DATA2                Mellanox ConnectX-3 Ethernet Adapter        14       NotPresent
     Ethernet 2           HCS VNIC                                    13       Up
     DATA1                Intel(R) 82574L Gigabit Network Co...#2     16       NotPresent
     DATA0                Intel(R) 82574L Gigabit Network Conn...     15       Up


**範例輸出 - 控制站 1**

以下是來自控制器 1 (主動控制器) 的輸出。 只在裝置上設定了 DATA 0 網路介面且正在運作中。

     Controller1>Get-NetAdapter
     Name                 InterfaceDescription                        ifIndex  Status
     ------               --------------------                        -------  ----------
     DATA3                Mellanox ConnectX-3 Ethernet Adapter        18       NotPresent
     DATA2                Mellanox ConnectX-3 Ethernet Adapter #2     19       NotPresent
     DATA1                Intel(R) 82574L Gigabit Network Co...#2     16       NotPresent
     DATA0                Intel(R) 82574L Gigabit Network Conn...     15       Up
     Ethernet 2           HCS VNIC                                    13       Up
     DATA5                Intel(R) Gigabit ET Dual Port Server...     14       NotPresent
     DATA4                Intel(R) Gigabit ET Dual Port Serv...#2     17       NotPresent


## <a name="troubleshoot-with-the-test-connection-cmdlet"></a>Test-Connection Cmdlet 的疑難排解
您可以使用 `Test-Connection` Cmdlet，來判斷 StorSimple 裝置是否可連線到外部網路。 如果已在安裝精靈中正確設定所有網路參數 (包括 DNS)，您就可以使用 `Test-Connection` Cmdlet 來傳送 Ping 給網路外部的已知位址，例如 outlook.com。

如果 ping 已停用，您應該啟用 ping 以進行此 Cmdlet 之連線問題的疑難排解。

請參閱下列來自 `Test-Connection` Cmdlet 的輸出範例。

> [!NOTE]
> 在第一個範例中，會以不正確的 DNS 設定裝置。 在第二個範例中，DNS 是正確的。

**範例輸出 - 不正確的 DNS**

在下列範例中，沒有任何 IPV4 與 IPV6 位址的輸出，表示不會解析 DNS。 這表示沒有外部網路的連線能力，而且必須提供正確的 DNS。

     Source        Destination     IPV4Address      IPV6Address
     ------        -----------     -----------      -----------
     HCSNODE0      outlook.com
     HCSNODE0      outlook.com
     HCSNODE0      outlook.com
     HCSNODE0      outlook.com

**範例輸出 - 正確的 DNS**

在下列範例中，DNS 會傳回的 IPV4 位址，表示已正確設定 DNS。 這確認具有外部網路的連線能力。

     Source        Destination     IPV4Address      IPV6Address
     ------        -----------     -----------      -----------
     HCSNODE0      outlook.com     132.245.92.194
     HCSNODE0      outlook.com     132.245.92.194
     HCSNODE0      outlook.com     132.245.92.194
     HCSNODE0      outlook.com     132.245.92.194

## <a name="troubleshoot-with-the-test-hcsmconnection-cmdlet"></a>Test-HcsmConnection Cmdlet 的疑難排解
針對已經連線且已向 StorSimple 裝置管理員服務註冊的裝置使用 `Test-HcsmConnection` Cmdlet。 這個 Cmdlet 可協助您確認已註冊裝置與相對應的 StorSimple 裝置管理員服務間的連線。 您可以在 Windows PowerShell for StorSimple 上執行這個命令。

### <a name="to-run-the-test-hcsmconnection-cmdlet"></a>執行 Test-HcsmConnection Cmdlet
1. 確定裝置已註冊。
2. 檢查裝置狀態。 如果裝置已停用、處於維護模式或離線狀態，您可能會看到下列其中一個錯誤：
   
   * ErrorCode.CiSDeviceDecommissioned - 這表示裝置已停用。
   * ErrorCode.DeviceNotReady - 這表示裝置處於維護模式。
   * ErrorCode.DeviceNotReady - 這表示裝置處於離線狀態。
3. 確認 StorSimple 裝置管理員服務正在執行中 (使用 [Get-ClusterResource](https://technet.microsoft.com/library/ee461004.aspx) Cmdlet)。 如果服務不在執行中，您可能會看到下列錯誤：
   
   * ErrorCode.CiSApplianceAgentNotOnline
   * ErrorCode.CisPowershellScriptHcsError - 這表示在執行 Get-ClusterResource 時發生例外狀況。
4. 檢查存取控制服務 (ACS) 權杖。 如果它擲回 Web 例外狀況，則可能是因為下列因素而產生的結果：閘道問題、遺漏 Proxy 驗證、不正確的 DNS 或驗證失敗。 您可能會看到下列錯誤：
   
   * ErrorCode.CiSApplianceGateway - 這表示 HttpStatusCode.BadGateway 例外狀況：名稱解析程式服務無法解析主機名稱。
   * ErrorCode.CiSApplianceProxy - 這表示 HttpStatusCode.ProxyAuthenticationRequired 例外狀況 (HTTP 狀態碼 407)：用戶端無法向 Proxy 伺服器進行驗證。
   * ErrorCode.CiSApplianceDNSError - 這表示 WebExceptionStatus.NameResolutionFailure 例外狀況：名稱解析程式服務無法解析主機名稱。
   * ErrorCode.CiSApplianceACSError - 這表示服務傳回驗證錯誤，但有連線能力。
     
     如果服務未擲回 Web 例外狀況，請檢查導致 ErrorCode.CiSApplianceFailure 產生的錯誤。 這表示設備失敗。
5. 檢查雲端服務連線。 如果服務擲回 Web 例外狀況，您可能會看到下列錯誤：
   
   * ErrorCode.CiSApplianceGateway - 這表示 HttpStatusCode.BadGateway 例外狀況：中繼 Proxy 伺服器收到來自其他 Proxy 或來自原始伺服器的不正確要求。
   * ErrorCode.CiSApplianceProxy - 這表示 HttpStatusCode.ProxyAuthenticationRequired 例外狀況 (HTTP 狀態碼 407)：用戶端無法向 Proxy 伺服器進行驗證。
   * ErrorCode.CiSApplianceDNSError - 這表示 WebExceptionStatus.NameResolutionFailure 例外狀況：名稱解析程式服務無法解析主機名稱。
   * ErrorCode.CiSApplianceACSError - 這表示服務傳回驗證錯誤，但有連線能力。
     
     如果服務未擲回 Web 例外狀況，請檢查導致 ErrorCode.CiSApplianceSaasServiceError 發生的錯誤。 這表示 StorSimple 裝置管理員服務發生問題。
6. 檢查 Azure 服務匯流排連線。 ErrorCode.CiSApplianceServiceBusError 表示裝置無法連接到服務匯流排。

記錄檔 CiSCommandletLog0Curr.errlog 和 CiSAgentsvc0Curr.errlog 將提供更多資訊，例如，例外狀況詳細資料。

如需如何使用 Cmdlet 的詳細資訊，請參閱 Windows PowerShell 參考說明文件中的 [Test-HcsmConnection](https://technet.microsoft.com/library/dn715782.aspx) 。

> [!IMPORTANT]
> 您可以針對主動和被動控制器執行這個 Cmdlet。

請參閱下列來自 `Test-HcsmConnection` Cmdlet 的輸出範例。

**範例輸出 – 成功註冊執行 StorSimple Update 3 的裝置**

      Controller1>Test-HcsmConnection

      Checking device registration state  ... Success
      Device registered successfully

      Checking primary NTP server [time.windows.com] ... Success

      Checking web proxy  ... NOT SET

      Checking primary IPv4 DNS server [10.222.118.154] ... Success
      Checking primary IPv6 DNS server  ... NOT SET
      Checking secondary IPv4 DNS server [10.222.120.24] ... Success
      Checking secondary IPv6 DNS server  ... NOT SET

      Checking device online  ... Success

      Checking device authentication  ... This will take a few minutes.
      Checking device authentication  ... Success

      Checking connectivity from device to service  ... This will take a few minutes.

      Checking connectivity from device to service  ... Success

      Checking connectivity from service to device  ... Success

      Checking connectivity to Microsoft Update servers  ... Success
      Controller1>

**範例輸出 – 離線裝置** 

這個範例是來自 Azure 入口網站中狀態為 [離線]  的裝置。

     Checking device registrationstate: Success
     Device is registered successfully
     Checking connectivity from device to SaaS.. Failure

該裝置無法使用目前的 Web Proxy 設定來連接。 這可能是 Web Proxy 設定問題或網路連線問題。 在此情況下，您應該確定 Web Proxy 設定正確無誤，而且您的 Web Proxy 伺服器在線上且可連線。

## <a name="troubleshoot-with-the-sync-hcstime-cmdlet"></a>使用 Sync-HcsTime Cmdlet 疑難排解
此 Cmdlet 可顯示裝置的時間。 如果裝置時間與 NTP 伺服器之間有時間差，您可以使用此 Cmdlet 強制同步裝置與 NTP 伺服器的時間。
- 如果裝置和 NTP 伺服器之間的時間差超過 5 分鐘，系統則會顯示警告。 
- 如果時間差超過 15 分鐘，裝置則會變成離線。 您還是可以使用此 Cmdlet 強制同步時間。 
- 不過，如果時間差超過 15 個小時，您便無法強制同步時間，且會顯示錯誤訊息。

**範例輸出 – 已使用 Sync-HcsTime 執行的強制同步時間作業**

     Controller0>Sync-HcsTime
     The current device time is 4/24/2015 4:05:40 PM UTC.

     Time difference between NTP server and appliance is 00.0824069 seconds. Do you want to resync time with NTP server?
     [Y] Yes [N] No (Default is "Y"): Y
     Controller0>

## <a name="troubleshoot-with-the-enable-hcsping-and-disable-hcsping-cmdlets"></a>使用 Enable-HcsPing 和 Disable-HcsPing Cmdlet 疑難排解
這兩個 Cmdlet 可確保您裝置上的網路介面正常回應 ICMP Ping 要求。 StorSimple 網路介面依預設不會回應 Ping 要求。 若要知道裝置是否處於線上和可以連線的狀態，執行此 Cmdlet 是最簡單的方式。

**範例輸出 – 啟用 HcsPing 和停用 HcsPing**

     Controller0>
     Controller0>Enable-HcsPing
     Successfully enabled ping.
     Controller0>
     Controller0>
     Controller0>Disable-HcsPing
     Successfully disabled ping.
     Controller0>

## <a name="troubleshoot-with-the-trace-hcsroute-cmdlet"></a>使用 Trace-HcsRoute Cmdlet 疑難排解
此 Cmdlet 可做為路由追蹤工具。 每過一段時間，它就會在前往最終目的地時將封包傳送到每個路由器，然後根據每個躍點傳回的封包計算結果。 由於此 Cmdlet 會顯示封包在任何指定的路由器或連結中遺失的程度，因此您可以找出可能導致網路發生問題的路由器或連結。

**示範如何使用 Trace-HcsRoute 追蹤封包的路由的範例輸出**

     Controller0>Trace-HcsRoute -Target 10.126.174.25

     Tracing route to contoso.com [10.126.174.25]
     over a maximum of 30 hops:
       0  HCSNode0 [10.126.173.90]
       1  contoso.com [10.126.174.25]

     Computing statistics for 25 seconds...
                 Source to Here   This Node/Link
     Hop  RTT    Lost/Sent = Pct  Lost/Sent = Pct  Address
       0                                           HCSNode0 [10.126.173.90]
                                     0/ 100 =  0%   |
       1    0ms     0/ 100 =  0%     0/ 100 =  0%  contoso.com
      [10.126.174.25]

     Trace complete.

## <a name="troubleshoot-with-the-get-hcsroutingtable-cmdlet"></a>使用 Get-HcsRoutingTable Cmdlet 疑難排解
此 Cmdlet 可顯示您 StorSimple 裝置的路由表。 路由表是一組規則，可以協助您判斷經由網際網路通訊協定 (IP) 傳輸的資料封包會將被導向的位置。

路由表會顯示將資料路由到指定的網路的介面和閘道器。 還提供路由計量，此計量是決定到達特定目的地所採取路徑的工具。 路由計量的值越低，建議採用的指數越高。

例如，如果您有 2 個已連接到網際網路的網路介面，DATA 2 和DATA 3。 如果 DATA 2 和DATA 3 的路由計量分別是 15 和 261，路由計量值較低的 DATA 2 便是建議用來連接網際網路的介面。

如果您的 StorSimple 裝置正在執行 Update 1，則 DATA 0 的網路介面是雲端流量的最佳選擇。 這表示即使有其他支援雲端的介面，系統還是會透過 DATA 0 路由雲端流量。

如果您執行 `Get-HcsRoutingTable` Cmdlet，但未指定任何參數 (如下列範例所示)，該 Cmdlet 會同時輸出 IPv4 和 IPv6 路由表。 您也可以指定 `Get-HcsRoutingTable -IPv4` 或 `Get-HcsRoutingTable -IPv6`，以取得相關的路由表。

      Controller0>
      Controller0>Get-HcsRoutingTable
      ===========================================================================
      Interface List
       14...00 50 cc 79 63 40 ......Intel(R) 82574L Gigabit Network Connection
       12...02 9a 0a 5b 98 1f ......Microsoft Failover Cluster Virtual Adapter
       13...28 18 78 bc 4b 85 ......HCS VNIC
        1...........................Software Loopback Interface 1
       21...00 00 00 00 00 00 00 e0 Microsoft ISATAP Adapter #2
       22...00 00 00 00 00 00 00 e0 Microsoft ISATAP Adapter #3
      ===========================================================================

      IPv4 Route Table
      ===========================================================================
      Active Routes:
      Network Destination        Netmask          Gateway       Interface  Metric
                0.0.0.0          0.0.0.0  192.168.111.100  192.168.111.101     15
              127.0.0.0        255.0.0.0         On-link         127.0.0.1    306
              127.0.0.1  255.255.255.255         On-link         127.0.0.1    306
        127.255.255.255  255.255.255.255         On-link         127.0.0.1    306
            169.254.0.0      255.255.0.0         On-link     169.254.1.235    261
          169.254.1.235  255.255.255.255         On-link     169.254.1.235    261
        169.254.255.255  255.255.255.255         On-link     169.254.1.235    261
          192.168.111.0    255.255.255.0         On-link   192.168.111.101    266
        192.168.111.101  255.255.255.255         On-link   192.168.111.101    266
        192.168.111.255  255.255.255.255         On-link   192.168.111.101    266
              224.0.0.0        240.0.0.0         On-link         127.0.0.1    306
              224.0.0.0        240.0.0.0         On-link     169.254.1.235    261
              224.0.0.0        240.0.0.0         On-link   192.168.111.101    266
        255.255.255.255  255.255.255.255         On-link         127.0.0.1    306
        255.255.255.255  255.255.255.255         On-link     169.254.1.235    261
        255.255.255.255  255.255.255.255         On-link   192.168.111.101    266
      ===========================================================================
      Persistent Routes:
        Network Address          Netmask  Gateway Address  Metric
                0.0.0.0          0.0.0.0  192.168.111.100       5
      ===========================================================================

      IPv6 Route Table
      ===========================================================================
      Active Routes:
       If Metric Network Destination      Gateway
        1    306 ::1/128                  On-link
       13    276 fd99:4c5b:5525:d80b::/64 On-link
       13    276 fd99:4c5b:5525:d80b::1/128
                                          On-link
       13    276 fd99:4c5b:5525:d80b::3/128
                                          On-link
       13    276 fe80::/64                On-link
       12    261 fe80::/64                On-link
       13    276 fe80::17a:4eba:7c80:727f/128
                                          On-link
       12    261 fe80::fc97:1a53:e81a:3454/128
                                          On-link
        1    306 ff00::/8                 On-link
       13    276 ff00::/8                 On-link
       12    261 ff00::/8                 On-link
       14    266 ff00::/8                 On-link
      ===========================================================================
      Persistent Routes:
        None

      Controller0>

## <a name="step-by-step-storsimple-troubleshooting-example"></a>StorSimple 逐步疑難排解範例
下列範例示範 StorSimple 部署的逐步疑難排解。 在範例案例中，裝置註冊會失敗，且會有錯誤訊息指出網路設定或 DNS 名稱不正確。

傳回的錯誤訊息如下：

     Invoke-HcsSetupWizard: An error has occurred while registering the device. This could be due to incorrect IP address or DNS name. Please check your network settings and try again. If the problems persist, contact Microsoft Support.
     +CategoryInfo: Not specified
     +FullyQualifiedErrorID: CiSClientCommunicationErros, Microsoft.HCS.Management.PowerShell.Cmdlets.InvokeHcsSetupWizardCommand

錯誤可能是因為下列其中一個原因所引發：

* 不正確的硬體安裝
* 故障的網路介面
* 不正確的 IP 位址、子網路遮罩、閘道、主要 DNS 伺服器或 Web Proxy
* 不正確的註冊金鑰
* 不正確的防火牆設定

### <a name="to-locate-and-fix-the-device-registration-problem"></a>找出並修正裝置註冊問題
1. 檢查裝置組態：在主動控制器上執行 `Invoke-HcsSetupWizard`。
   
   > [!NOTE]
   > 安裝精靈必須在主動控制器上執行。 若要確認您已連接到主動控制器，請查看序列主控台中顯示的橫幅。 該橫幅會指出您是連接到控制器 0 或控制器 1，而且該控制器是主動或被動。 如需詳細資訊，請參閱 [識別裝置上的主動控制器](storsimple-8000-controller-replacement.md#identify-the-active-controller-on-your-device)。
   
2. 確定裝置纜線已正確連接：檢查裝置背板的網路纜線。 纜線是裝置型號專用的。 如需詳細資訊，請參閱[安裝您的 StorSimple 8100 裝置](storsimple-8100-hardware-installation.md)或[安裝您的 StorSimple 8600 裝置](storsimple-8600-hardware-installation.md)。
   
   > [!NOTE]
   > 如果您使用的是 10 GbE 網路連接埠，就必須使用隨附的 QSFP-SFP 配接器和 SFP 纜線。 如需詳細資訊，請參閱[建議用於 10 GbE 連接埠的纜線、交換器及收發器清單](storsimple-supported-hardware-for-10-gbe-network-interfaces.md)。
  
3. 確認網路介面的健康情況：
   
   * 使用 Get-NetAdapter Cmdlet 來偵測  DATA 0 網路介面的健康情況。 
   * 如果連結未運作， **ifindex** 狀態將指出介面已關閉。 然後您必須檢查連接埠到設備與連接埠到交換器的網路連線。 您也必須排除不健全的纜線。 
   * 如果您懷疑主動控制器上的 DATA 0 連接埠故障，可透過連接到控制器 1 上的 DATA 0 連接埠來確認這一點。 若要確認這一點，可從裝置背後的控制站 0 拔除網路纜線、將纜線連接到控制站 1，然後再次執行 Get-NetAdapter Cmdlet。
     如果控制站上的 DATA 0 連接埠故障，請 [連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md) 以進行後續步驟。 您可能需要更換系統上的控制站。
4. 確認與交換器的連線能力：
   
   * 確定主要機箱上的控制站 0 和控制站 1 上的 DATA 0 網路介面位於同一個子網路上。 
   * 檢查集線器或路由器。 通常，您應該將這兩個控制站連接到相同的集線器或路由器。 
   * 確定用來連接的交換器在同一個 vLAN 上具有適用於這兩個控制站的 DATA 0。
5. 排除任何使用者錯誤：
   
   * 重新執行安裝精靈 (執行 **Invoke-HcsSetupWizard**)，並再次輸入值，以確定沒有任何錯誤。 
   * 確認已使用註冊金鑰。 相同的註冊金鑰可用來將多個裝置連線到 StorSimple 裝置管理員服務。 使用 [取得服務註冊金鑰](storsimple-8000-manage-service.md#get-the-service-registration-key) 中的程序，以確保您使用的是正確的註冊金鑰。
     
     > [!IMPORTANT]
     > 如果您有多個服務正在執行中，就必須確保會使用適當服務的註冊金鑰來註冊裝置。 如果您向錯誤的 StorSimple 裝置管理員服務註冊了裝置，就必須[連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md)以進行後續步驟。 您可能必須執行裝置的原廠重設 (這可能導致資料遺失)，然後將它連接到所需的服務。
     > 
     > 
6. 使用 Test-Connection Cmdlet 來確認您擁有外部網路的連線能力。 如需詳細資訊，請參閱 [Test-Connection Cmdlet 的疑難排解](#troubleshoot-with-the-test-connection-cmdlet)。
7. 檢查防火牆干擾。 如果您已確認虛擬 IP (VIP)、子網路、閘道和 DNS 設定都正確，但仍看見連線問題，則可能是您的防火牆封鎖了裝置與外部網路之間的通訊。 您必須確定 StorSimple 裝置上的連接埠 80 和 443 可用於傳出通訊。 如需詳細資訊，請參閱 [StorSimple 裝置的網路需求](storsimple-8000-system-requirements.md#networking-requirements-for-your-storsimple-device)。
8. 查看記錄。 請移至 [可用來疑難排解的支援封裝和裝置記錄](#support-packages-and-device-logs-available-for-troubleshooting)。
9. 如果上述步驟並未解決問題，請 [連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md) 以尋求協助。

## <a name="next-steps"></a>後續步驟
[了解如何使用診斷工具對 StorSimple 裝置進行疑難排解](storsimple-8000-diagnostics.md)。

<!--Link references-->

[1]: https://technet.microsoft.com/library/dd379547(v=ws.10).aspx
[2]: https://technet.microsoft.com/library/dd392266(v=ws.10).aspx 
