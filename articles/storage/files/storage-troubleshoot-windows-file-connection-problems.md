---
title: 針對 Windows 中的 Azure 檔案服務問題進行疑難排解 | Microsoft Docs
description: 針對 Windows 中的 Azure 檔案服務問題進行疑難排解
services: storage
author: jeffpatt24
tags: storage
ms.service: storage
ms.topic: article
ms.date: 05/11/2018
ms.author: jeffpatt
ms.component: files
ms.openlocfilehash: 935d4a3ba3fc3199177be5bd4e70f82239c3c971
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39529683"
---
# <a name="troubleshoot-azure-files-problems-in-windows"></a>針對 Windows 中的 Azure 檔案服務問題進行疑難排解

本文列出當您從 Windows 用戶端連線時，與 Microsoft Azure 檔案服務相關的常見問題。 文中也會提供這些問題的可能原因和解決方案。 除了本文中的疑難排解步驟，您也可以使用 [AzFileDiagnostics](https://gallery.technet.microsoft.com/Troubleshooting-tool-for-a9fa1fe5) \(英文\) 來確保 Windows 用戶端環境具備正確的必要條件。 AzFileDiagnostics 會自動偵測本文中提及的大部分徵兆，並協助設定您的環境以取得最佳效能。 您也可以在 [Azure 檔案共用疑難排解員](https://support.microsoft.com/help/4022301/troubleshooter-for-azure-files-shares)中找到此資訊，其提供步驟以協助您解決連線/對應/掛接 Azure 檔案共用的問題。


<a id="error53-67-87"></a>
## <a name="error-53-error-67-or-error-87-when-you-mount-or-unmount-an-azure-file-share"></a>當您嘗試掛接或取消掛接 Azure 檔案共用時發生錯誤 53、錯誤 67 或錯誤 87

當您嘗試從內部部署或不同資料中心掛接檔案共用時，可能會收到下列錯誤：

- 發生系統錯誤 53。 找不到網路路徑。
- 發生系統錯誤 67。 找不到網路名稱。
- 發生系統錯誤 87。 參數錯誤。

### <a name="cause-1-unencrypted-communication-channel"></a>原因 1：通訊通道未加密

基於安全考量，如果通訊通道未加密，而且未從 Azure 檔案共用所在的相同資料中心進行連線嘗試，與 Azure 檔案共用的連線就會遭到封鎖。 唯有當使用者的用戶端作業系統支援 SMB 加密，才會提供通訊通道加密。

Windows 8、Windows Server 2012 和更新版本的每個系統交涉都要求包含支援加密的 SMB 3.0。

### <a name="solution-for-cause-1"></a>原因 1 的解決方案

從執行下列其中一項的用戶端連線：

- 符合 Windows 8 和 Windows Server 2012 或更新版本的需求
- 從與用於 Azure 檔案共用的 Azure 儲存體帳戶相同之資料中心的虛擬機器連線

### <a name="cause-2-port-445-is-blocked"></a>原因 2：連接埠 445 遭到封鎖

如果連接埠 445 至 Azure 檔案服務資料中心的輸出通訊遭到封鎖，可能會發生系統錯誤 53 或系統錯誤 67。 若要查看 ISP 是否允許從連接埠 445 進行存取的摘要，請參閱 [TechNet](http://social.technet.microsoft.com/wiki/contents/articles/32346.azure-summary-of-isps-that-allow-disallow-access-from-port-445.aspx) \(英文\)。

若要了解這是否為「系統錯誤 53」訊息的原因，您可以使用 Portqry 查詢 TCP:445 端點。 如果篩選顯示 TCP:445 端點，則 TCP 連接埠會被封鎖。 查詢範例如下：

  `g:\DataDump\Tools\Portqry>PortQry.exe -n [storage account name].file.core.windows.net -p TCP -e 445`

如果 TCP 連接埠 445 被網路路徑規則所封鎖，您將看到下列輸出：

  `TCP port 445 (microsoft-ds service): FILTERED`

如需如何使用 Portqry 的詳細資訊，請參閱 [Portqry.exe 命令列公用程式說明](https://support.microsoft.com/help/310099)。

### <a name="solution-for-cause-2"></a>原因 2 的解決方案

聯絡您的 IT 部門，要求開啟連接埠 445 輸出到 [Azure IP 範圍](https://www.microsoft.com/download/details.aspx?id=41653) \(英文\)。

### <a name="cause-3-ntlmv1-is-enabled"></a>原因 3：已啟用 NTLMv1

如果用戶端上已啟用 NTLMv1 通訊，就會發生系統錯誤 53 或系統錯誤 87。 Azure 檔案僅支援 NTLMv2 驗證。 啟用 NTLMv1 會使用戶端變得較不安全。 因此，Azure 檔案服務會封鎖通訊。 

若要判斷這是否為錯誤的原因，請確認已將下列登錄子機碼的值設為 3：

**HKLM\SYSTEM\CurrentControlSet\Control\Lsa > LmCompatibilityLevel**

如需詳細資訊，請參閱 TechNet 上的 [LmCompatibilityLevel](https://technet.microsoft.com/library/cc960646.aspx) 主題。

### <a name="solution-for-cause-3"></a>原因 3 的解決方案

在下列登錄子機碼中，將 **LmCompatibilityLevel** 值還原為預設值 3：

  **HKLM\SYSTEM\CurrentControlSet\Control\Lsa**

<a id="error1816"></a>
## <a name="error-1816-not-enough-quota-is-available-to-process-this-command-when-you-copy-to-an-azure-file-share"></a>當您複製到 Azure 檔案共用時發生錯誤 1816「配額不足無法處理此命令」

### <a name="cause"></a>原因

當您到達同時開啟的控制代碼上限時 (此為針對掛接檔案共用之電腦上的檔案所允許的上限)，即會發生錯誤 1816。

### <a name="solution"></a>解決方法

關閉一些控制代碼以減少同時開啟的控制代碼數，然後再試一次。 如需詳細資訊，請參閱 [Microsoft Azure 儲存體效能與延展性檢查清單](../common/storage-performance-checklist.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)。

<a id="slowfilecopying"></a>
## <a name="slow-file-copying-to-and-from-azure-files-in-windows"></a>從 Windows 中的 Azure 檔案服務複製檔案或將檔案複製到其中的速度變慢

當您嘗試將檔案傳輸到 Azure 檔案服務時，可能會看到效能變慢。

- 如果您沒有特定的 I/O 大小需求下限，建議您使用 1 MB 的 I/O 大小以獲得最佳效能。
-   如果您知道擴充寫入檔案的最終大小，而且當檔案上未寫入的結尾中有零時您的軟體不會產生相容性問題，則請事先設定檔案大小，而不是將每次寫入設為擴充寫入。
-   使用正確的複製方法：
    -   針對兩個檔案共用之間的所有傳輸，使用 [AzCopy](../common/storage-use-azcopy.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)。
    -   在內部部署電腦上的檔案共用之間，使用 [Robocopy](https://blogs.msdn.microsoft.com/granth/2009/12/07/multi-threaded-robocopy-for-faster-copies/) \(英文\)。

### <a name="considerations-for-windows-81-or-windows-server-2012-r2"></a>針對 Windows 8.1 或 Windows Server 2012 R2 的考量

若用戶端執行 Windows 8.1 或 Windows Server 2012 R2，請確定已安裝 [KB3114025](https://support.microsoft.com/help/3114025) Hotfix。 此 Hotfix 可改善建立和關閉控制代碼的效能。

您可以執行下列指令碼來檢查是否已安裝此 Hotfix：

`reg query HKLM\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters\Policies`

如果已安裝 hotfix，則會顯示下列輸出︰

`HKEY_Local_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters\Policies {96c345ef-3cac-477b-8fcd-bea1a564241c} REG_DWORD 0x1`

> [!Note]
> 自 2015 年 12 月起，Azure Marketplace 中的 Windows Server 2012 R2 映像預設已安裝 Hotfix KB3114025。

<a id="shareismissing"></a>
## <a name="no-folder-with-a-drive-letter-in-my-computer"></a>[我的電腦] 中沒有任何含磁碟機代號的資料夾

如果您以系統管理員身分使用 net use 對應 Azure 檔案共用，該共用似乎就會遺失。

### <a name="cause"></a>原因

根據預設，Windows 檔案總管不會以系統管理員身分執行。 如果您從系統管理命令提示字元執行 net use，就是以系統管理員身分對應網路磁碟機。 因為對應的磁碟機是以使用者為中心，如果磁碟機掛接在不同的使用者帳戶下，登入的使用者帳戶不會顯示此磁碟機。

### <a name="solution"></a>解決方法
從非系統管理員命令掛接共用。 或者，您可以依照[此 TechNet 主題](https://technet.microsoft.com/library/ee844140.aspx)設定 **EnableLinkedConnections** 登錄值。

<a id="netuse"></a>
## <a name="net-use-command-fails-if-the-storage-account-contains-a-forward-slash"></a>如果儲存體帳戶包含斜線，net use 命令就會失敗

### <a name="cause"></a>原因

Net use 命令會將斜線 (/) 解譯為命令列選項。 如果您的使用者帳戶名稱開頭為斜線，磁碟機對應將會失敗。

### <a name="solution"></a>解決方法

您可以使用下列其中一種方式來解決這個問題：

- 執行下列 PowerShell 命令：

  `New-SmbMapping -LocalPath y: -RemotePath \\server\share -UserName accountName -Password "password can contain / and \ etc" `

  從批次檔中，您可以使用此方式執行命令：

  `Echo new-smbMapping ... | powershell -command –`

- 除非斜線是第一個字元，否則，可以用雙引號括住金鑰來解決此問題。 如果是，可使用互動模式分開輸入您的密碼，或者，重新產生金鑰，以取得不是斜線開頭的金鑰。

<a id="cannotaccess"></a>
## <a name="application-or-service-cannot-access-a-mounted-azure-files-drive"></a>應用程式或服務無法存取掛接的 Azure 檔案服務磁碟機

### <a name="cause"></a>原因

磁碟機是按每個使用者掛接。 如果您的應用程式或服務正在與掛接磁碟機之帳戶不同的使用者帳戶下執行，應用程式將不會看到該磁碟機。

### <a name="solution"></a>解決方法

使用下列其中一個解決方案：

-   從應用程式所屬的相同使用者帳戶掛接磁碟機。 您可以使用 PsExec 之類的工具。
- 在 net use 命令的使用者名稱和密碼參數中傳遞儲存體帳戶名稱和金鑰。
- 使用 cmdkey 命令以將認證新增至認證管理員。 透過互動式登入或使用 runas，從服務帳戶內容底下的命令列執行這個命令。
  
  `cmdkey /add:<storage-account-name>.file.core.windows.net /user:AZURE\<storage-account-name> /pass:<storage-account-key>`
- 直接對應共用而不使用對應磁碟機代號。 某些應用程式可能無法正確地重新連線至磁碟機代號，因此使用完整的 UNC 路徑可能比較可靠。 

  `net use * \\storage-account-name.file.core.windows.net\share`

遵循這些指示執行之後，當您針對系統/網路服務帳戶執行 net use 時，可能會收到下列錯誤訊息：「發生系統錯誤 1312。 指定的登入工作階段不存在。 可能已被終止。」 如果發生這種情況，請確定傳遞至 net use 的使用者名稱包含網域資訊 (例如，"[storage account name].file.core.windows.net")。

<a id="doesnotsupportencryption"></a>
## <a name="error-you-are-copying-a-file-to-a-destination-that-does-not-support-encryption"></a>「您正將檔案複製到不支援加密的目的地」錯誤

透過網路複製檔案時，該檔案會在來源電腦上解密、以純文字傳送，並在目的地上重新加密。 不過，您可能會在嘗試複製加密的檔案時看到下列錯誤：「您正在將檔案複製到不支援加密的目的地。」

### <a name="cause"></a>原因
如果您使用加密檔案系統 (EFS)，可能會發生此問題。 BitLocker 加密的檔案可以複製到 Azure 檔案服務。 不過，Azure 檔案服務不支援 NTFS EFS。

### <a name="workaround"></a>因應措施
若要透過網路複製檔案，您必須先將它解密。 使用下列其中一種方法：

- 使用 **copy /d** 命令。 它可讓加密的檔案另存為目的地上的解密檔案。
- 設定下列登錄機碼：
  - 路徑 = HKLM\Software\Policies\Microsoft\Windows\System
  - 數值類型 = DWORD
  - Name = CopyFileAllowDecryptedRemoteDestination
  - Value = 1

請注意，設定登錄機碼會影響所有對網路共用所做的複製作業。

## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員。
如果仍需要協助，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。
