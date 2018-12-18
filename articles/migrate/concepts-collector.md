---
title: 關於 Azure Migrate 中的收集器設備 | Microsoft Docs
description: 提供 Azure Migrate 中收集器設備的相關資訊。
author: snehaamicrosoft
ms.service: azure-migrate
ms.topic: conceptual
ms.date: 09/28/2018
ms.author: snehaa
services: azure-migrate
ms.openlocfilehash: b79045e54b9c2ee4846f2216704a419e0ff85501
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47434427"
---
# <a name="about-the-collector-appliance"></a>關於收集器設備

 此文章提供「Azure Migrate 收集器」的相關資訊。

「Azure Migrate 收集器」是一個輕量型設備，可用來探索內部部署 vCenter 環境，目的是要在移轉至 Azure 之前，先使用 [Azure Migrate](migrate-overview.md) 服務進行評量。  


## <a name="deploying-the-collector"></a>部署收集器

您需使用 OVF 範本來部署收集器設備：

- 您從 Azure 入口網站中的 Azure Migrate 專案下載 OVF 範本。 您將所下載的檔案匯入至 vCenter Server 以設定收集器設備 VM。
- VMware 從 OVF 設定一個具有 4 個核心、8 GB RAM 及一個 80 GB 磁碟的 VM。 作業系統是 Windows Server 2012 R2 (64 位元)。
- 當您執行收集器時，會執行一些先決條件檢查，以確定收集器可以連線至 Azure Migrate。

- [深入了解](tutorial-assessment-vmware.md#create-the-collector-vm)如何建立收集器。


## <a name="collector-prerequisites"></a>收集器先決條件

收集器必須通過幾個先決條件檢查，以確保它可以透過網際網路連線到 Azure Migrate 服務，並上傳探索到的資料。

- **檢查網際網路連線**：收集器可以直接或透過 Proxy 連線至網際網路。
    - 先決條件檢查會確認是否能夠與[必要和選擇性 URL](#connect-to-urls) 連線。
    - 如果您可以直接連線至網際網路，則除了確定收集器可以連線至必要的 URL 之外，無須採取任何特定動作。
    - 如果您透過 Proxy 進行連線，請注意[下列需求](#connect-via-a-proxy)。
- **確認時間同步處理**：收集器應該與網際網路時間伺服器同步，以確保會驗證對服務提出的要求。
    - 您應該可以從收集器觸達 portal.azure.com url，以便驗證時間。
    - 如果機器未同步，您就必須將收集器 VM 的時間變更成與目前時間相符。 若要這樣做，請在 VM 上開啟管理員提示，執行 **w32tm /tz** 以檢查時區。 執行 **w32tm /resync** 以同步時間。
- **檢查收集器服務是否正在執行**：「Azure Migrate 收集器」服務應該正在收集器 VM 上執行。
    - 此服務會在機器開機時自動啟動。
    - 如果收集器服務未處於執行中，請從 [控制台] 啟動它。
    - 收集器服務會連線至 vCenter Server、收集 VM 中繼資料和效能資料，然後將資料傳送給 Azure Migrate 服務。
- **檢查是否已安裝 VMware PowerCLI 6.5**：收集器 VM 上必須安裝 VMware PowerCLI 6.5 PowerShell 模組，才能與 vCenter Server 進行通訊。
    - 如果收集器可以存取安裝該模組所需的 URL，在收集器部署期間便會自動安裝該模組。
    - 如果收集器在部署期間無法安裝該模組，您就必須[手動安裝](#install-vwware-powercli-module-manually)。
- **檢查與 vCenter Server 的連線**：收集器必須能夠連線至 vCenter Server 並查詢 VM、其中繼資料及效能計數器。 [驗證連線先決條件](#connect-to-vcenter-server)。


### <a name="connect-to-the-internet-via-a-proxy"></a>透過 Proxy 連線至網際網路

- 如果 proxy 伺服器需要驗證，您可以在設定收集器時，指定使用者名稱和密碼。
- Proxy 伺服器的 IP 位址/FQDN 應該以 *http://IPaddress* 或 *http://FQDN* 的形式指定。
- 僅支援 HTTP Proxy。 收集器不支援 HTTPS 型 Proxy 伺服器。
- 如果 Proxy 伺服器是攔截 Proxy，您就必須將 Proxy 憑證匯入至收集器 VM。
    1. 在收集器 VM 中，移至 [開始] 功能表 > [管理電腦憑證]。
    2. 在 [憑證] 工具中的 [憑證 - 本機電腦] 底下，尋找 [受信任的發行者] > [憑證]。

        ![憑證工具](./media/concepts-intercepting-proxy/certificates-tool.png)

    3. 將 Proxy 憑證複製到收集器 VM。 您可能需要向網路管理員取得該憑證。
    4. 按兩下來開啟憑證，然後按一下 [安裝憑證]。
    5. 在 [憑證匯入精靈] > [儲存位置] 中，選擇 [本機電腦]。

    ![憑證存放區位置](./media/concepts-intercepting-proxy/certificate-store-location.png)

    6. 選取 [將所有憑證放入以下的存放區] > [瀏覽] > [受信任的發行者]。 按一下 [完成]  以匯入憑證。

    ![憑證存放區](./media/concepts-intercepting-proxy/certificate-store.png)

    7. 確認憑證是否已如預期般匯入，並確認網際網路連線能力先決條件檢查是否如預期般運作。




### <a name="connect-to-urls"></a>連線至 URL

透過連線至 URL 清單，即可驗證連線能力檢查。

**URL** | **詳細資料**  | **先決條件檢查**
--- | --- | ---
*.portal.azure.com | 檢查是否能夠與 Azure 服務連線及時間是否同步。 | 必須具備 URL 存取權。<br/><br/> 如果無法連線，先決條件檢查就會失敗。
*.oneget.org:443<br/><br/> *.windows.net:443<br/><br/> *.windowsazure.com:443<br/><br/> *.powershellgallery.com:443<br/><br/> *.msecnd.net:443<br/><br/> *.visualstudio.com:443| 用來下載 PowerShell vCenter PowerCLI 模組。 | 不一定要具備 URL 存取權。<br/><br/> 先決條件檢查不會失敗。<br/><br/> 收集器 VM 上的自動模組安裝將會失敗。 您將必須手動安裝模組。


### <a name="install-vmware-powercli-module-manually"></a>手動安裝 VMware PowerCLI 模組

1. 使用[這些步驟](https://blogs.vmware.com/PowerCLI/2017/04/powercli-install-process-powershell-gallery.html)來安裝模組。 這些步驟同時提供線上和離線安裝說明。
2. 如果收集器 VM 處於離線狀態，而要安裝能夠存取網際網路之不同機器上的模組，您就必須從該機器將 VMware.* 檔案複製到收集器 VM。
3. 安裝之後，您可以重新啟動先決條件檢查，以確認是否已安裝 PowerCLI。

### <a name="connect-to-vcenter-server"></a>連線至 vCenter Server

收集器會連線至 vCenter Server，然後查詢 VM 中繼資料和效能計數器。 以下是您針對此連線所需的項目。

- 僅支援 vCenter Server 5.5、6.0 及 6.5 版。
- 您需要有一個具備以下摘要說明之權限的唯讀帳戶，才能進行探索。 只有能夠以該帳戶存取的資料中心才可供您存取來進行探索。
- 您預設會透過 FQDN 或 IP 位址連線至 vCenter Server。 如果 vCenter Server 在不同的連接埠上進行接聽，您就需使用 *IPAddress:Port_Number* 或 *FQDN:Port_Number* 形式來與其連線。
- 若要收集儲存體和網路功能的效能資料，必須將 vCenter Server 的統計資料設定設為層級 3。
- 如果層級低於 3，還是能夠進行探索，但不會收集效能資料。 可能會收集一些計數器，但其他計數器則會設定為零。
- 如果未收集儲存體和網路功能的效能資料，就會根據 CPU 和記憶體的效能資料，以及磁碟和網路介面卡的設定資料，提出評量大小建議。
- 收集器應該能透過網路看到 vCenter Server。

#### <a name="account-permissions"></a>帳戶權限

**帳戶** | **權限**
--- | ---
至少一個唯讀使用者帳戶 | 資料中心物件 –> 傳播至子物件、role=Read-only   


## <a name="collector-communications"></a>收集器通訊

下圖和表格摘要說明收集器的通訊方式。

![收集器通訊圖表](./media/tutorial-assessment-vmware/portdiagram.PNG)


**收集器通訊對象** | **連接埠** | **詳細資料**
--- | --- | ---
Azure Migrate 服務 | TCP 443 | 收集器會透過 SSL 443 與 Azure Migrate 服務進行通訊。
vCenter Server | TCP 443 | 收集器必須能夠與 vCenter Server 進行通訊。<br/><br/> 它預設會透過 443 與 vCenter 連線。<br/><br/> 如果 vCenter Server 在不同的連接埠上進行接聽，則該連接埠應該可供作為收集器上的傳出連接埠。
RDP | TCP 3389 |


## <a name="securing-the-collector-appliance"></a>保護收集器設備

建議您進行下列步驟來保護收集器設備：

- 請勿將管理員密碼分享或錯誤地寄託給未經授權的人員。
- 不使用時，關閉設備。
- 將設備放置在安全的網路中。
- 完成移轉之後，請刪除設備執行個體。
- 此外，移轉後，也請刪除磁碟備份檔案 (VMDK)，因為磁碟上可能已快取 vCenter 認證。

## <a name="os-license-in-the-collector-vm"></a>收集器 VM 上的 OS 授權

收集器隨附一個有效期為 180 天的 Windows Server 2012 R2 評估授權。 如果您收集器 VM 的評估期間即將到期，建議您下載新的 OVA 並建立新設備。

## <a name="updating-the-os-of-the-collector-vm"></a>更新收集器 VM 的 OS

雖然收集器設備有 180 天的評估授權，但您必須持續更新設備上的 OS，才能避免設備自動關機。

- 如果收集器 60 天未更新，就會開始自動關閉機器。
- 如果正在執行探索，則即使過了 60 天，也不會關閉機器。 完成探索之後，便會關閉機器。
- 如果您使用收集器超過 60 天，建議您執行 Windows 更新，讓機器隨時維持在最新狀態。

## <a name="upgrading-the-collector-appliance-version"></a>升級收集器設備版本

您無須再次下載 OVA，即可將收集器升級至最新版本。

1. 下載[最新列出的升級套件](concepts-collector-upgrade.md)
2. 若要確保下載的 Hotfix 是安全的，請開啟系統管理員命令視窗並執行下列命令來產生 ZIP 檔案的雜湊。 產生的雜湊應該符合針對特定版本所述的雜湊：

    ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```

    (使用範例：C:\>CertUtil -HashFile C:\AzureMigrate\CollectorUpdate_release_1.0.9.14.zip SHA256)
3. 將 ZIP 檔案複製到 Azure Migrate 收集器虛擬機器 (收集器設備)。
4. 以滑鼠右鍵按一下 ZIP 檔案並選取 [全部解壓縮]。
5. 在 [Setup.ps1] 上按一下滑鼠右鍵並選取 [用 PowerShell 執行]，然後依照螢幕上的指示安裝更新。


## <a name="discovery-methods"></a>探索方法

有兩種可供收集器設備用來進行探索的方法：單次探索或連續探索。


### <a name="one-time-discovery"></a>單次探索

收集器會與 vCenter Server 進行單次通訊以收集 VM 的相關中繼資料。 使用此方法時：

- 設備不會持續連線至 Azure Migrate 專案。
- 探索完成之後，內部部署環境中的變更並不會反映在 Azure Migrate 中。 若要反映任何變更，您必須再次於相同的專案中探索相同的環境。
- 針對此探索方法，您必須將 vCenter Server 中的統計資料設定設為層級 3。
- 將層級設定為 3 之後，最長需要一天的時間，才能產生效能計數器。 因此，建議您在一天後再執行探索。
- 收集 VM 的效能資料時，設備會倚賴儲存在 vCenter Server 中的歷史效能資料。 它會收集過去一個月的效能歷程記錄。
- Azure Migrate 會收集每個計量的平均計數器 (而不是尖峰計數器)。

### <a name="continuous-discovery"></a>持續探索

收集器設備會持續連線至 Azure Migrate 專案。

- 收集器會持續分析內部部署環境，每 20 秒便收集一次即時使用情況資料。
- 此模型並不倚賴 vCenter Server 統計資料設定來收集效能資料。
- 設備會彙總 20 秒範例，然後每 15 分鐘建立一個單一資料點。
- 為了建立資料點，設備會從 20 秒範例中選取尖峰值並將其傳送給 Azure。
- 您可以隨時從收集器停止連續分析。

> [!NOTE]
> 連續探索功能為預覽版。 如果 vCenter Server 統計資料設定未設為層級 3，建議您使用此方法。


## <a name="discovery-process"></a>探索程序

設定妥設備之後，您便可以執行探索。 其運作方式如下：

- 您依範圍執行探索。 這會探索在指定 vCenter 清查路徑中的所有 VM。
    - 您需一次設定一個範圍。
    - 此範圍可包含 1500 個以內的 VM。
    - 此範圍可以是資料中心、資料夾或 ESXi 主機。
- 連線至 vCenter Server 之後，您需透過指定用於收集的移轉專案來進行連線。
- 系統探索到 VM，並將其中繼資料和效能資料傳送給 Azure。 這些動作皆為收集工作的一部分。
    - 收集器設備會獲得一個特定的收集器識別碼，此識別碼就指定的機器而言會跨探索而持續存在。
    - 執行中的收集工作會獲得特定的工作階段識別碼。 此識別碼會隨著每個收集工作而有所不同，因此可用來進行疑難排解。

### <a name="collected-metadata"></a>所收集的中繼資料

收集器設備會探索下列 VM 靜態中繼資料：

- VM 顯示名稱 (在 vCenter Server 上)
- VM 的清查路徑 (vCenter Server 中的主機/資料夾)
- IP 位址
- MAC 位址
- 作業系統
- 核心、磁碟、NIC 數目
- 記憶體大小、磁碟大小
- VM、磁碟及網路的效能計數器。

#### <a name="performance-counters"></a>效能計數器

- **單次探索**：針對單次探索收集計數器時，請注意項列事項：

    - 最多可能需要 15 分鐘的時間，才能收集設定描述資料並傳送給專案。
    - 收集設定資料之後，最多可能需要一小時的時間，入口網站才會提供效能資料。
    - 在入口網站提供中繼資料之後，會出現 VM 清單，而您便可以開始建立用於評量的群組。
- **連續探索**：針對連續探索，請注意下列事項：
    - VM 的設定資料會在您開始探索後 1 小時可供使用
    - 效能資料會在 2 小時後可供使用。
    - 在您開始探索之後，請至少等候 一天讓設備分析環境，然後才建立評量。

**計數器** | **層級** | **個別裝置層級** | **對評量的影響**
--- | --- | --- | ---
cpu.usage.average | 1 | NA | 建議的虛擬機器大小和成本  
mem.usage.average | 1 | NA | 建議的虛擬機器大小和成本  
virtualDisk.read.average | 2 | 2 | 計算磁碟大小、儲存成本、虛擬機器大小
virtualDisk.write.average | 2 | 2  | 計算磁碟大小、儲存成本、虛擬機器大小
virtualDisk.numberReadAveraged.average | 1 | 3 |  計算磁碟大小、儲存成本、虛擬機器大小
virtualDisk.numberReadAveraged.average | 1 | 3 |   計算磁碟大小、儲存成本、VM 大小
net.received.average | 2 | 3 |  計算 VM 大小和網路成本                        |
net.transmitted.average | 2 | 3 | 計算 VM 大小和網路成本    

## <a name="next-steps"></a>後續步驟

[為內部部署 VMware VM 設定評量](tutorial-assessment-vmware.md)
