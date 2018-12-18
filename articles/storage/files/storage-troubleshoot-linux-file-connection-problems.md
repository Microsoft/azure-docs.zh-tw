---
title: 針對 Linux 中的 Azure 檔案服務問題進行疑難排解 | Microsoft Docs
description: 針對 Linux 中的 Azure 檔案服務問題進行疑難排解
services: storage
author: jeffpatt24
tags: storage
ms.service: storage
ms.topic: article
ms.date: 05/11/2018
ms.author: jeffpatt
ms.component: files
ms.openlocfilehash: 0f99913ab252b94d475f920bd734e68ff5f3b3d3
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39525115"
---
# <a name="troubleshoot-azure-files-problems-in-linux"></a>針對 Linux 中的 Azure 檔案服務問題進行疑難排解

本文列出當您從 Linux 用戶端連線時，與 Microsoft Azure 檔案服務相關的常見問題。 文中也會提供這些問題的可能原因和解決方案。

<a id="permissiondenied"></a>
## <a name="permission-denied-disk-quota-exceeded-when-you-try-to-open-a-file"></a>當您嘗試開啟檔案時，「[使用權限被拒] 超出磁碟配額」

在 Linux 中，您會收到類似以下的錯誤訊息︰

**<filename> [使用權限被拒] 超出磁碟配額**

### <a name="cause"></a>原因

您已達到檔案所允許的同時開啟控點上限。

### <a name="solution"></a>解決方法

關閉一些控點以減少同時開啟的控點數，然後再次嘗試操作。 如需詳細資訊，請參閱 [Microsoft Azure 儲存體效能與延展性檢查清單](../common/storage-performance-checklist.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)。

<a id="slowfilecopying"></a>
## <a name="slow-file-copying-to-and-from-azure-files-in-linux"></a>從 Linux 中的 Azure 檔案服務複製檔案或將檔案複製到其中的速度變慢

- 如果您沒有特定的 I/O 大小需求下限，建議您使用 1 MB 的 I/O 大小以獲得最佳效能。
- 如果您知道擴充寫入檔案的最終大小，而且當檔案上未寫入的結尾中有零時您的軟體不會產生相容性問題，則請事先設定檔案大小，而不是將每次寫入設為擴充寫入。
- 使用正確的複製方法：
    - 針對兩個檔案共用之間的所有傳輸，使用 [AzCopy](../common/storage-use-azcopy.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)。
    - 在內部部署電腦上的檔案共用之間，使用 [Robocopy](https://blogs.msdn.microsoft.com/granth/2009/12/07/multi-threaded-robocopy-for-faster-copies/) \(英文\)。

<a id="error112"></a>
## <a name="mount-error112-host-is-down-because-of-a-reconnection-time-out"></a>因連線逾時而發生「掛接錯誤 (112)：主機已關機」

當用戶端閒置很長時間時，Linux 用戶端上發生「112」掛接錯誤。 加長閒置時間之後，用戶端中斷連線且連線逾時。  

### <a name="cause"></a>原因

連線可能因下列原因而閒置：

-   使用預設的「軟」掛接選項時，造成無法重新建立 TCP 連線以連線到伺服器的網路通訊失敗
-   未出現在較舊核心中的最近重新連線修正

### <a name="solution"></a>解決方法

此 Linux 核心中的重新連線問題已隨下列變更修正：

- [修正重新連線在通訊端重新連線許久之後不會延遲 SMB3 工作階段重新連線 (英文)](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/fs/cifs?id=4fcd1813e6404dd4420c7d12fb483f9320f0bf93)
- [在通訊端重新連線之後立即呼叫 Echo 服務 (英文)](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=b8c600120fc87d53642476f48c8055b38d6e14c7)
- [CIFS：修正重新連線期間可能發生的記憶體損毀 (英文)](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/?id=53e0e11efe9289535b060a51d4cf37c25e0d0f2b)
- [CIFS：修正重新連線期間可能發生的 Mutex 雙重鎖定 (針對核心 4.9 版與更新版本)](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/?id=96a988ffeb90dba33a71c3826086fe67c897a183) \(英文\)

但是，這些變更可能尚未移植到所有 Linux 發行版本。 此修正和其他重新連線修正在下列常見 Linux 核心中進行：4.4.40、4.8.16 和 4.9.1. 您可以升級至其中一個建議的核心版本，以完成此修正。

### <a name="workaround"></a>因應措施

您可以指定硬掛接，以解決此問題。 這會強制用戶端一直等候直到連線建立或明確中斷，而且它可用來防止因為網路逾時而發生錯誤。 不過，這個因應措施可能會導致無限期等候。 請準備好視需要停止連線。

如果您無法升級至最新的核心版本，您可以使用下列因應措施解決此問題：在 Azure 檔案共用中保留一個每 30 秒 (或更短時間) 就會寫入的檔案。 這必須是寫入作業，例如重寫檔案的建立或修改日期。 否則，您可能會取得快取的結果，而您的作業可能不會觸發重新連線。

<a id="error115"></a>
## <a name="mount-error115-operation-now-in-progress-when-you-mount-azure-files-by-using-smb-30"></a>當您使用 SMB 3.0 掛接 Azure 檔案服務時，發生「掛接錯誤 (115)：作業進行中」

### <a name="cause"></a>原因

某些 Linux 發行版本尚未支援 SMB 3.0 的加密功能，如果使用者因為功能遺失而嘗試使用 SMB 3.0 來掛接 Azure 檔案服務，可能會收到「115」錯誤訊息。 使用 Ubuntu 16.04 或更新版本時，目前僅支援 SMB 3.0 搭配完整加密。

### <a name="solution"></a>解決方法

4.11 核心推出 Linux 的 SMB 3.0 適用的加密功能。 此功能讓您可從內部部署或不同 Azure 區域的 Azure 檔案共用進行裝載。 發佈時，這項功能已向前移植到 Ubuntu 17.04 和 Ubuntu 16.10。 如果您的 Linux SMB 用戶端不支援加密，在與檔案儲存體帳戶相同的資料中心上，從 Azure Linux VM 使用 SMB 2.1 掛接 Azure 檔案服務。

<a id="slowperformance"></a>
## <a name="slow-performance-on-an-azure-file-share-mounted-on-a-linux-vm"></a>掛接在 Linux VM 上的 Azure 檔案共用效能變慢

### <a name="cause"></a>原因

效能變慢可能的一個原因是已停用快取。

### <a name="solution"></a>解決方法

若要查看是否停用快取，請尋找 **cache=** 項目。 

**Cache=none** 表示已停用快取。  使用預設的 mount 命令或明確地為 mount 命令加上 **cache=strict** 選項來重新掛接共用，以確保啟用預設快取或 "strict" 快取模式。

在某些情況下，**serverino** 掛接選項可能導致 **ls** 命令對每個目錄項目執行 stat。 當您要列出大型目錄時，這個行為會導致效能降低。 您可以檢查 **/etc/fstab** 項目中的掛接選項：

`//azureuser.file.core.windows.net/cifs /cifs cifs vers=2.1,serverino,username=xxx,password=xxx,dir_mode=0777,file_mode=0777`

您也可以執行 **sudo mount | grep cifs** 命令並檢查其輸出 \(例如以下範例輸出\)，來檢查是否使用正確的選項：

`//azureuser.file.core.windows.net/cifs on /cifs type cifs (rw,relatime,vers=2.1,sec=ntlmssp,cache=strict,username=xxx,domain=X,uid=0,noforceuid,gid=0,noforcegid,addr=192.168.10.1,file_mode=0777, dir_mode=0777,persistenthandles,nounix,serverino,mapposix,rsize=1048576,wsize=1048576,actimeo=1)`

如果沒有 **cache=strict** 或 **serverino** 選項，請執行[文件](../storage-how-to-use-files-linux.md)中的掛接命令，將 Azure 檔案服務卸載並再次掛接。 然後，重新檢查 **/etc/fstab** 項目是否有正確的選項。

<a id="timestampslost"></a>
## <a name="time-stamps-were-lost-in-copying-files-from-windows-to-linux"></a>將檔案從 Windows 複製到 Linux 時，遺失時間戳記

在 Linux/Unix 平台上，如果檔案 1 和檔案 2 由不同的使用者所擁有，**cp -p** 命令會失敗。

### <a name="cause"></a>原因

COPYFILE 中的強制旗標 **f** 會導致在 Unix 上執行 **cp -p -f**。 此命令也無法保留您並不擁有之檔案的時間戳記。

### <a name="workaround"></a>因應措施

使用儲存體帳戶使用者來複製檔案：

- `Useadd : [storage account name]`
- `Passwd [storage account name]`
- `Su [storage account name]`
- `Cp -p filename.txt /share`

## <a name="cannot-connect-or-mount-an-azure-file-share"></a>無法連線到或裝載 Azure 檔案共用

### <a name="cause"></a>原因

此問題的常見原因為：


- 您使用的是不相容的 Linux 散發套件用戶端。 建議您使用下列的 Linux 散發套件來連線到 Azure 檔案共用：

* **具有對應掛接功能的最低建議版本 (SMB 2.1 版與 SMB 3.0 版)。**    
    
    |   | SMB 2.1 <br>(掛接在相同 Azure 區域內的 VM 上) | SMB 3.0 <br>(從內部部署環境和跨區域掛接) |
    | --- | :---: | :---: |
    | Ubuntu Server | 14.04+ | 16.04+ |
    | RHEL | 7+ | 7.5+ |
    | CentOS | 7+ |  7.5+ |
    | Debian | 8+ |   |
    | openSUSE | 13.2+ | 42.3+ |
    | SUSE Linux Enterprise Server | 12 | 12 SP3+ |

- 用戶端上未安裝 CIFS 公用程式。
- 用戶端上未安裝 SMB/CIFS 的最低版本 (2.1 版)。
- 用戶端不支援 SMB 3.0 加密。 SMB 3.0 加密可於 Ubuntu 16.4 和更新版本，以及 SUSE 12.3 和更新版本中使用。 其他散發套件需要核心 4.11 和更新版本。
- 您正在嘗試透過不支援的 TCP 通訊埠 445 連線到儲存體帳戶。
- 您正在嘗試從 Azure VM 連線到 Azure 檔案共用，而該 VM 與儲存體帳戶位於不同的區域。

### <a name="solution"></a>解決方法

若要解決此問題，請使用[適用於 Linux 上 Azure 檔案服務掛接錯誤的疑難排解工具](https://gallery.technet.microsoft.com/Troubleshooting-tool-for-02184089) \(英文\)。 此工具可協助您驗證用戶端執行環境、偵測可能造成 Azure 檔案服務存取錯誤的不相容用戶端設定、提供自行修正的規範指引，以及收集診斷追蹤。

## <a name="ls-cannot-access-ltpathgt-inputoutput-error"></a>ls：無法存取 '&lt;path&gt;'：輸入/輸出錯誤

當您嘗試使用 ls 命令列出 Azure 檔案共用中的檔案時，ls 命令會在列出檔案時停滯，而您會收到下列錯誤：

**ls：無法存取 '&lt;path&gt;'：輸入/輸出錯誤**


### <a name="solution"></a>解決方法
將 Linux 核心升級為下列已修正此問題的版本：

- 4.4.87+
- 4.9.48+
- 4.12.11+
- 大於或等於 4.13 的所有版本

## <a name="cannot-create-symbolic-links---ln-failed-to-create-symbolic-link-t-operation-not-supported"></a>無法建立符號連結 - ln：無法建立符號連結 't'：不支援作業

### <a name="cause"></a>原因
根據預設，使用 CIFS 在 Linux 上掛接 Azure 檔案共用，並不會啟用對符號連結的支援。 您會看到像這樣的錯誤：
```
ln -s linked -n t
ln: failed to create symbolic link 't': Operation not supported
```
### <a name="solution"></a>解決方法
Linux CIFS 用戶端不支援透過 SMB2/3 通訊協定，建立 Windows 樣式的符號連結。 Linux 用戶端目前支援另一種符號連結樣式，稱為 [Mishall + 法文符號連結] (https://wiki.samba.org/index.php/UNIX_Extensions#Minshall.2BFrench_symlinks)，用於建立和遵循作業。 需要符號連結的客戶可以使用 "mfsymlinks" 掛接選項。 因為這也是 Mac 使用的格式，所以通常會建議使用 "mfsymlinks"。

為了能使用符號連結，請將下列內容新增至 CIFS 掛接命令結尾：

```
,mfsymlinks
```

該命令看起來會像這樣：

```
sudo mount -t cifs //<storage-account-name>.file.core.windows.net/<share-name> <mount-point> -o vers=<smb-version>,username=<storage-account-name>,password=<storage-account-key>,dir_mode=0777,file_mode=0777,serverino,mfsynlinks
```

新增之後，您即可按照 [Wiki](https://wiki.samba.org/index.php/UNIX_Extensions#Storing_symlinks_on_Windows_servers) 上的建議建立符號連結。

## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員。

如果仍需要協助，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。
