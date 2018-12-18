---
title: StorSimple 8000 系列系統需求 | Microsoft Docs
description: 描述 Microsoft Azure StorSimple 解決方案的軟體、網路及高可用性需求和最佳作法。
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
ms.date: 09/28/2017
ms.author: alkohli
ms.openlocfilehash: 1a9cdf31c5924d22d968cd99383417ba371cd1c3
ms.sourcegitcommit: 9cc3d9b9c36e4c973dd9c9028361af1ec5d29910
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2018
ms.locfileid: "28011056"
---
# <a name="storsimple-8000-series-software-high-availability-and-networking-requirements"></a>StorSimple 8000 系列軟體、高可用性和網路需求

## <a name="overview"></a>概觀

歡迎使用 Microsoft Azure StorSimple。 本文描述重要的系統需求和 StorSimple 裝置以及存取裝置之儲存體用戶端的最佳做法。 建議您先仔細檢閱資訊，再部署 StorSimple 系統，然後在部署和後續作業期間，必要時回顧參考。

系統需求包括：

* **儲存體用戶端的軟體需求** - 描述支援的作業系統和那些作業系統的其他需求。
* **StorSimple 裝置的網路需求** - 提供需要在防火牆中開啟以允許 iSCSI、雲端或管理流量的連接埠相關資訊。
* **StorSimple 的高可用性需求** - 描述高可用性需求，以及 StorSimple 裝置和主機電腦的最佳作法。

## <a name="software-requirements-for-storage-clients"></a>儲存體用戶端的軟體需求

下列軟體需求適用於存取 StorSimple 裝置的儲存體用戶端。

| 受支援的作業系統 | 必要版本 | 其他需求/注意事項 |
| --- | --- | --- |
| Windows Server |2008 R2 SP1、2012、2012 R2、2016 |StorSimple iSCSI 磁碟區僅支援在下列 Windows 磁碟類型上使用：<ul><li>基本磁碟上的簡單磁碟區</li><li>動態磁碟上的簡單及鏡像磁碟區</li></ul>僅支援作業系統中原生提供的軟體 iSCSI 啟動器。 不支援硬體 iSCSI 啟動器。<br></br>如果您使用 StorSimple iSCSI 磁碟區，則支援 Windows Server 2012 和 2016 精簡佈建及 ODX 功能。<br><br>StorSimple 可以建立精簡佈建和完整佈建的磁碟區。 它無法建立部分佈建的磁碟區。<br><br>重新格式化精簡佈建的磁碟區可能需要很長的時間。 建議刪除磁碟區，然後建立新的磁碟區，而不是重新格式化。 不過，如果您仍然偏好重新格式化磁碟區︰<ul><li>先執行下列命令再重新格式化，以避免空間回收延遲︰ <br>`fsutil behavior set disabledeletenotify 1`</br></li><li>格式化完成後，使用下列命令來重新啟用空間回收︰<br>`fsutil behavior set disabledeletenotify 0`</br></li><li>將 [KB 2878635](https://support.microsoft.com/kb/2870270) 中所述的 Windows Server 2012 Hotfix 套用到您的 Windows Server 電腦。</li></ul></li></ul></ul> 如果您要設定 StorSimple Snapshot Manager 或 StorSimple Adapter for SharePoint，請移至[選用元件的軟體需求](#software-requirements-for-optional-components)。 |
| VMware ESX |5.5 和 6.0 |受 VMware vSphere 支援為 iSCSI 用戶端。 StorSimple 裝置上的 VMWare vSphere 支援 VAAI 區塊功能。 |
| Linux RHEL/CentOS |5、6 和 7 |支援具備 Open-iSCSI 啟動器第 5 版、第 6 版和第 7 版的 Linux iSCSI 用戶端。 |
| Linux |SUSE Linux 11 | |

> [!NOTE]
> StorSimple 目前不支援 IBM AIX。


## <a name="software-requirements-for-optional-components"></a>選用元件的軟體需求

選擇性 StorSimple 元件 (StorSimple Snapshot Manager 和 StorSimple Adapter for SharePoint) 有下列軟體需求。

| 元件 | 主機平台 | 其他需求/注意事項 |
| --- | --- | --- |
| StorSimple Snapshot Manager |Windows Server 2008 R2 SP1、2012、2012 R2 |在 Windows Server 上需要使用 StorSimple Snapshot Manager，才能備份/還原鏡像動態磁碟及進行任何應用程式一致備份。<br> StorSimple Snapshot Manager 僅在 Windows Server 2008 R2 SP1 (64 位元)、Windows Server 2012 R2 和 Windows Server 2012 上受到支援。<ul><li>如果您使用 Window Server 2012，您必須先安裝 .NET 3.5 – 4.5，才能安裝 StorSimple Snapshot Manager。</li><li>如果您使用 Windows Server 2008 R2 SP1，您必須先安裝 Windows Management Framework 3.0，才能安裝 StorSimple Snapshot Manager。</li></ul> |
| StorSimple Adapter for SharePoint |Windows Server 2008 R2 SP1、2012、2012 R2 |<ul><li>StorSimple Adapter for SharePoint 僅在 SharePoint 2010 和 SharePoint 2013 上受到支援。</li><li>RBS 需要 SQL Server Enterprise Edition、2008 R2 或 2012 版。</li></ul> |

## <a name="networking-requirements-for-your-storsimple-device"></a>StorSimple 裝置的網路需求。

您的 StorSimple 裝置是鎖定的裝置。 不過，您的防火牆中必須開啟連接埠，以允許 iSCSI、雲端和管理流量。 下表列出必須在防火牆中開啟的連接埠。 在這個資料表中，*in* 或 *inbound* 指的是輸入用戶端要求存取裝置的方向。 *Out* 或 *outbound* 指的是 StorSimple 裝置於外部傳送資料至部署之上的方向：例如，輸出到網際網路。

| 連接埠號碼 <sup>1,2</sup> | 內或外 | 連接埠範圍 | 必要 | 注意 |
| --- | --- | --- | --- | --- |
| TCP 80 (HTTP)<sup>3</sup> |外 |WAN |否 |<ul><li>輸出連接埠用於網際網路存取以擷取更新。</li><li>輸出 Web Proxy 可由使用者設定。</li><li>若要允許系統更新，此連接埠也必須為控制器固定 IP 開啟。</li></ul> |
| TCP 443 (HTTPS)<sup>3</sup> |外 |WAN |yes |<ul><li>輸出連接埠用來存取雲端中的資料。</li><li>輸出 Web Proxy 可由使用者設定。</li><li>若要允許系統更新，此連接埠也必須為控制器固定 IP 開啟。</li><li>在這兩個控制器上也使用此連接埠進行記憶體回收。</li></ul> |
| UDP 53 (DNS) |外 |WAN |在某些情況下，請參閱附註。 |只有當您使用網際網路 DNS 伺服器時，才需要此連接埠。 |
| UDP 123 (NTP) |外 |WAN |在某些情況下，請參閱附註。 |只有當您使用網際網路 NTP 伺服器時，才需要此連接埠。 |
| TCP 9354 |外 |WAN |yes |StorSimple 裝置使用輸出連接埠與 StorSimple 裝置管理員服務通訊。 |
| 3260 (iSCSI) |在 |LAN |否 |此連接埠用來透過 iSCSI 存取資料。 |
| 5985 |在 |LAN |否 |輸入連接埠由 StorSimple Snapshot Manager 用來與 StorSimple 裝置通訊。<br>當您透過 HTTPS 從遠端連線到 Windows PowerShell for StorSimple，也會使用此連接埠。 |
| 5986 |在 |LAN |否 |當您透過 HTTPS 從遠端連線到 Windows PowerShell for StorSimple，便會使用此連接埠。 |

<sup>1</sup> 公用網際網路上沒有必須開啟的輸入連接埠。

<sup>2</sup> 如果多個連接埠都有閘道器設定，將會根據下列[連接埠路由](#routing-metric)所述的連接埠路由順序，決定輸出路由的流量順序。

<sup>3</sup> StorSimple 裝置上的控制器固定 IP 必須可路由傳送，且能夠直接連線到網際網路或透過設定的 Web Proxy。 固定 IP 位址用來為裝置更新提供服務和記憶體回收。 如果裝置控制器無法透過固定 IP 連線到網際網路，您將無法更新 StorSimple 裝置，而且記憶體回收將不會正常運作。

> [!IMPORTANT]
> 請確定防火牆不會修改或解密 StorSimple 裝置和 Azure 之間的任何 SSL 流量。


### <a name="url-patterns-for-firewall-rules"></a>防火牆規則的 URL 模式

網路系統管理員通常可以根據 URL 模式設定進階防火牆規則，來篩選輸入和輸出流量。 您的 StorSimple 裝置和 StorSimple 裝置管理員服務取決於其他 Microsoft 應用程式，例如 Azure 服務匯流排、Azure Active Directory 存取控制、儲存體帳戶和 Microsoft Update 伺服器。 與這些應用程式相關聯的 URL 模式可以用來設定防火牆規則。 請務必了解與這些應用程式相關聯的 URL 模式可以變更。 接著，您將需要網路系統管理員監控 StorSimple 的防火牆規則，並在需要時更新。

我們建議您，在大部分情況下，根據固定 IP 位址為輸出流量設定防火牆規則。 不過，您可以使用下列資訊設定建立安全環境所需的進階防火牆規則。

> [!NOTE]
> 裝置 (來源) IP 應該一律設定為所有啟用的網路介面。 目的地 IP 應該設為 [Azure 資料中心 IP 範圍](https://www.microsoft.com/en-us/download/confirmation.aspx?id=41653)。


#### <a name="url-patterns-for-azure-portal"></a>Azure 入口網站的 URL 模式

| URL 模式 | 元件/功能 | 裝置 IP |
| --- | --- | --- |
| `https://*.storsimple.windowsazure.com/*`<br>`https://*.accesscontrol.windows.net/*`<br>`https://*.servicebus.windows.net/*`<br>`https://login.windows.net` |StorSimple 裝置管理員服務<br>存取控制服務<br>Azure 服務匯流排<br>驗證服務 |啟用雲端功能的網路介面 |
| `https://*.backup.windowsazure.com` |裝置註冊 |僅限資料 0 |
| `http://crl.microsoft.com/pki/*`<br>`http://www.microsoft.com/pki/*` |憑證撤銷 |啟用雲端功能的網路介面 |
| `https://*.core.windows.net/*` <br>`https://*.data.microsoft.com`<br>`http://*.msftncsi.com` |Azure 儲存體帳戶和監視 |啟用雲端功能的網路介面 |
| `http://*.windowsupdate.microsoft.com`<br>`https://*.windowsupdate.microsoft.com`<br>`http://*.update.microsoft.com`<br> `https://*.update.microsoft.com`<br>`http://*.windowsupdate.com`<br>`http://download.microsoft.com`<br>`http://wustat.windows.com`<br>`http://ntservicepack.microsoft.com` |Microsoft Update 伺服器<br> |僅限控制站的固定 IP |
| `http://*.deploy.akamaitechnologies.com` |Akamai CDN |僅限控制站的固定 IP |
| `https://*.partners.extranet.microsoft.com/*`<br>`https://dcupload.microsoft.com/`<br>`https://*.support.microsoft.com/` |支援封裝 |啟用雲端功能的網路介面 |

#### <a name="url-patterns-for-azure-government-portal"></a>Azure Government 入口網站的 URL 模式

| URL 模式 | 元件/功能 | 裝置 IP |
| --- | --- | --- |
| `https://*.storsimple.windowsazure.us/*`<br>`https://*.accesscontrol.usgovcloudapi.net/*`<br>`https://*.servicebus.usgovcloudapi.net/*`<br>`https://login.microsoftonline.us` |StorSimple 裝置管理員服務<br>存取控制服務<br>Azure 服務匯流排<br>驗證服務 |啟用雲端功能的網路介面 |
| `https://*.backup.windowsazure.us` |裝置註冊 |僅限資料 0 |
| `http://crl.microsoft.com/pki/*`<br>`http://www.microsoft.com/pki/*` |憑證撤銷 |啟用雲端功能的網路介面 |
| `https://*.core.usgovcloudapi.net/*` <br>`https://*.data.microsoft.com`<br>`http://*.msftncsi.com` |Azure 儲存體帳戶和監視 |啟用雲端功能的網路介面 |
| `http://*.windowsupdate.microsoft.com`<br>`https://*.windowsupdate.microsoft.com`<br>`http://*.update.microsoft.com`<br> `https://*.update.microsoft.com`<br>`http://*.windowsupdate.com`<br>`http://download.microsoft.com`<br>`http://wustat.windows.com`<br>`http://ntservicepack.microsoft.com` |Microsoft Update 伺服器<br> |僅限控制站的固定 IP |
| `http://*.deploy.akamaitechnologies.com` |Akamai CDN |僅限控制站的固定 IP |
| `https://*.partners.extranet.microsoft.com/*`<br>`https://dcupload.microsoft.com/`<br>`https://*.support.microsoft.com/` |支援封裝 |啟用雲端功能的網路介面 |

### <a name="routing-metric"></a>路由度量

路由度量與介面和閘道器 (將資料路由到指定的網路) 相關聯。 路由度量用於路由通訊協定，如果它知道到相同目的地有多個路徑存在，則會計算到指定目的地的最佳路徑。 路由計量的值越低，建議採用的指數越高。

在 StorSimple 內容中，如果多個網路介面和閘道器設定為通道流量，路由度量會派上用場，判斷使用介面的相對順序。 使用者無法變更路由度量。 不過您可以使用 `Get-HcsRoutingTable` Cmdlet 列印您的 StorSimple 裝置上的路由資料表 (和度量)。 如需 Get-HcsRoutingTable Cmdlet 的詳細資訊，請參閱 [針對 StorSimple 部署進行疑難排解](storsimple-troubleshoot-deployment.md)。

以下說明 Update 2 和更新版本中所使用的路由度量演算法。

* 一組預先決定的值已指派給網路介面。
* 當網路介面已啟用雲端或已停用雲端功能，但是已設定閘道器時，請考量以下所示的範例資料表，其中包含指派給各種網路介面的值。 請注意，此處指派的值僅為範例值。

    | Linux | 已啟用雲端 | 已停用雲端且具有閘道器 |
    |-----|---------------|---------------------------|
    | Data 0  | 1            | -                        |
    | Data 1  | 2            | 20                       |
    | Data 2  | 3            | 30                       |
    | Data 3  | 4            | 40                       |
    | Data 4  | 5            | 50                       |
    | Data 5  | 6            | 60                       |


* 雲端流量透過網路介面路由的順序為：
  
    Data 0 > Data 1 > Date 2 > Data 3 > Data 4 > Data 5
  
    這也可以由下列範例來說明。
  
    請考慮具有兩個已啟用雲端網路介面 (Data 0 和 Data 5) 的 StorSimple 裝置。 Data 1 到 Data 4 已停用雲端，但是具有已設定的閘道器。 針對此裝置路由流量的順序為：
  
    Data 0 (1) > Data 5 (6) > Data 1 (20) > Data 2 (30) > Data 3 (40) > Data 4 (50)
  
    以括號括住的數字表示個別的路由度量。
  
    如果 Data 0 失敗，雲端流量將會透過 Data 5 路由。 假設已在其他所有網路上設定閘道器，如果 Data 0 和 Data 5 失敗，則雲端流量會通過 Data 1。
* 如果已啟用雲端網路介面失敗，則會重試 3 次 (有 30 秒的延遲) 以連線到介面。 如果所有重試失敗，會將流量路由至路由資料表決定的下一個可用已啟用雲端介面。 如果所有已啟用雲端網路介面失敗，則裝置將容錯移轉至另一個控制器 (在此案例中無需重新開機)。
* 如果有已啟用 iSCSI 網路介面的 VIP 失敗，則會重試 3 次 (有 2 秒的延遲)。 這種行為與舊版相同。 如果所有 iSCSI 網路介面都失敗，會發生控制器容錯移轉 (伴隨重新開機)。
* 有 VIP 失敗時，您的 StorSimple 裝置上也會引發警示。 如需詳細資訊，請移至 [警示快速參考](storsimple-8000-manage-alerts.md)。
* 根據重試，iSCSI 將會優先於雲端。
  
    請考慮下列範例：StorSimple 裝置已啟用兩個網路介面，Data 0 和 Data 1。 Data 0 已啟用雲端功能，而 Data 1 已啟用雲端和 iSCSI 功能。 此裝置上沒有其他網路介面啟用雲端或 iSCSI。
  
    如果 Data 1 失敗，假設它是最後一個 iSCSI 網路介面，這會導致控制器容錯移轉至其他控制器上的 Data 1。

### <a name="networking-best-practices"></a>網路最佳作法

除了上述的網路需求，如需最佳效能的 StorSimple 解決方案，請遵循下列最佳作法：

* 請確定您的 StorSimple 裝置有專用的 40 Mbps 頻寬 (或以上) 隨時可用。 此頻寬不應與其他應用程式共用 (或應該透過使用 QoS 原則保證配置)。
* 請確定隨時都可以使用網路連線到網際網路。 裝置的零星或不可靠網際網路連線 (包含毫無網際網路連線能力) 將導致不受支援的組態。
* 藉由在裝置上擁有專用的網路介面以存取 iSCSI 和雲端，可以隔離 iSCSI 和雲端流量。 如需詳細資訊，請參閱如何在您的 StorSimple 裝置上 [修改網路介面](storsimple-8000-modify-device-config.md#modify-network-interfaces) 。
* 請勿針對網路介面使用連結彙總控制通訊協定 (LACP) 組態。 這個組態不受支援。

## <a name="high-availability-requirements-for-storsimple"></a>StorSimple 的高可用性需求

隨附於 StorSimple 方案的硬體平台具有可用性及可靠性功能，為資料中心裡高度可用的容錯儲存體基礎結構打下基礎。 不過，您應遵守需求和最佳作法，以協助確保 StorSimple 解決方案的可用性。 部署 StorSimple 之前，請仔細檢閱下列 StorSimple 裝置和連線主機電腦的需求和最佳作法。

如需監視和維護 StorSimple 裝置之硬體元件的詳細資訊，請前往[使用 StorSimple 裝置管理員服務監視硬體元件和狀態](storsimple-8000-monitor-hardware-status.md)和 [StorSimple 硬體元件更換](storsimple-8000-hardware-component-replacement.md)。

### <a name="high-availability-requirements-and-procedures-for-your-storsimple-device"></a>StorSimple 裝置的高可用性需求和程序

仔細檢閱下列資訊，以確保 StorSimple 裝置的高可用性。

#### <a name="pcms"></a>PCM

StorSimple 裝置包含備援、可熱交換的電源和冷卻模組 (PCM)。 每個 PCM 具有足夠的容量來提供服務給整個底座。 若要確保高可用性，必須安裝這兩個 PCM。

* 將您的 PCM 連接到不同的電源來源，以在其中一個電源來源故障時提供可用性。
* 如果一個 PCM 失敗，請立即要求替代品。
* 當您具備替代品並已準備好安裝它時，請移除失敗的 PCM。
* 請勿同時移除兩個 PCM。 PCM 模組包含備用電池模組。 移除這兩個 PCM 會導致關機時沒有電池保護，而且不會儲存裝置狀態。 如需電池的詳細資訊，請移至 [維護備用電池模組](storsimple-8000-battery-replacement.md#maintain-the-backup-battery-module)。

#### <a name="controller-modules"></a>控制器模組

StorSimple 裝置包括備援、可熱交換的控制器模組。 控制器模組以主動/被動方式運作。 一個控制器模組隨時都保持主動狀態，可提供服務，而另一個控制器模組則保持被動。 被動控制器模組已開啟電源，並且可在主動控制器模組失敗或移除時開始運作。 每個控制器模組具有足夠的容量來提供服務給整個底座。 必須安裝兩個控制器模組，以確保高可用性。

* 請隨時確定兩個控制器模組皆已安裝。
* 如果一個控制器模組失敗，請立即要求替代品。
* 當您具備替代品並已準備好安裝它時，請移除失敗的控制器模組。 長時間移除模組會影響氣流，因此也會影響系統的冷卻。
* 請確定兩個控制器模組的網路連線都相同，而且已連線的網路介面具有相同的網路組態。
* 如果一個控制器模組失敗或需要替代品，請先確定另一個控制器模組處於主動狀態，才取代失敗的控制器模組。 若要確認控制器為作用中，請移至 [識別您裝置上的作用中控制器](storsimple-8000-controller-replacement.md#identify-the-active-controller-on-your-device)。
* 請勿同時移除兩個控制器模組。 如果正在進行控制器容錯移轉，請勿關閉待命控制器模組或從底座將其移除。
* 控制器容錯移轉後，請先等待至少五分鐘，才移除其中一個控制器模組。

#### <a name="network-interfaces"></a>網路介面

每個 StorSimple 裝置控制器模組有四個 1 GB 和兩個 10 GB 乙太網路介面。

* 請確定兩個控制器模組的網路連線都相同，而且控制器模組介面已連線的網路介面具有相同的網路組態。
* 如果可能，請跨不同的交換器部署網路連線，以確保網路裝置失敗事件中的服務可用性。
* 拔除唯一或最後一個剩餘之已啟用 iSCSI 的介面 (使用指派的 IP) 時，請先停用介面，然後拔除纜線。 如果介面已先拔除，它會造成主動的控制器容錯移轉到被動的控制器。 如果被動控制器也已拔除其對應的介面，兩個控制器會先多次重新開機，然後才固定在一個控制器上。
* 將至少兩個資料介面從每個控制器模組連線到網路。
* 如果您已啟用這兩個 10 Gbe 介面，請跨不同的交換器部署這些介面。
* 如果可能，請在伺服器上使用 MPIO，以確保伺服器可容許連結、網路或介面失敗。

如需有關建立裝置網路以提供高可用性和效能的詳細資訊，請移至[安裝您的 8100 裝置](storsimple-8100-hardware-installation.md#cable-your-storsimple-8100-device)或[安裝您的 StorSimple 8600 裝置](storsimple-8600-hardware-installation.md#cable-your-storsimple-8600-device)。

#### <a name="ssds-and-hdds"></a>SSD 與 HDD

StorSimple 裝置包含使用鏡像空間保護的固態硬碟 (SSD) 與硬碟 (HDD)。 使用鏡像空間可確保裝置能夠容許一或多個 SSD 或 HDD 的失敗。

* 請確定已安裝所有 SSD 和 HDD 模組。
* 如果一個 SSD 或 HDD 失敗，請立即要求替代品。
* 如果 SSD 或 HDD 失敗或需要替代品，請確定您只移除了需要替代品的 SSD 或 HDD。
* 請勿於任何時間從系統移除一個以上的 SSD 或 HDD。
  特定類型 (HDD、SSD) 的 2 個以上的磁碟失敗或在短時間範圍內的連續失敗可能會導致系統故障和潛在資料遺失。 如果發生這種情況，請 [連絡 Microsoft 支援](storsimple-8000-contact-microsoft-support.md) 尋求協助。
* 在更換期間，請在 [硬體健康狀態] 刀鋒視窗中的 [共用元件] 監視 SSD 和 HDD 中的磁碟機。 綠色核取狀態表示磁碟狀況良好或確定，而紅色驚嘆號點則表示失敗的 SSD 或 HDD。
* 建議您設定所有需要保護之磁碟區的雲端快照集，以防系統失敗。

#### <a name="ebod-enclosure"></a>EBOD 機箱

除了主要機箱之外，StorSimple 裝置模型 8600 還包括磁碟擴充群 (EBOD) 機箱。 EBOD 包含使用鏡像空間保護的 EBOD 控制器和硬碟 (HDD)。 使用鏡像空間可確保裝置能夠容許一或多個 HDD 的失敗。 EBOD 機箱已透過備援 SAS 纜線連接至主要機箱。

* 請隨時確定這兩個 EBOD 機箱控制器模組，兩條 SAS 纜線，以及所有硬碟都已安裝。
* 如果 EBOD 機箱控制器模組故障，請立即要求替代品。
* 如果一個 EBOD 機箱控制器模組失敗，請先確定另一個控制器模組處於主動狀態，才取代失敗的模組。 若要確認控制器為作用中，請移至 [識別您裝置上的作用中控制器](storsimple-8000-controller-replacement.md#identify-the-active-controller-on-your-device)。
* 在更換 EBOD 控制器模組期間，請存取 [監視] > [硬體健康狀態]，持續監視 StorSimple 裝置管理員服務中的元件狀態。
* 如果 SAS 纜線失敗或需要替代品 (Microsoft 支援應涉入這類決定)，請確定您只移除了需要替代品的 SAS 纜線。
* 請勿於任何時間同時從系統移除兩條 SAS 纜線。

### <a name="high-availability-recommendations-for-your-host-computers"></a>主機電腦的高可用性建議

仔細檢閱這些最佳作法，以確保連接至 StorSimple 裝置之主機的高可用性。

* 使用[二節點檔案伺服器叢集組態][1]設定 StorSimple。 藉由移除失敗的單點，以及在主機端上建置備援，整個解決方案會變得高度可用。
* 使用 Windows Server 2012 (SMB 3.0) 持續可用的 (CA) 共用，以在儲存體控制器的容錯移轉期間獲得高可用性。 如需設定檔案伺服器叢集和 Windows Server 2012 持續可用的共用之其他資訊，請參閱此 [影片示範](http://channel9.msdn.com/Events/IT-Camps/IT-Camps-On-Demand-Windows-Server-2012/DEMO-Continuously-Available-File-Shares)。

## <a name="next-steps"></a>後續步驟

* [了解 StorSimple 系統限制](storsimple-8000-limits.md)。
* [了解如何部署 StorSimple 解決方案](storsimple-8000-deployment-walkthrough-u2.md)。

<!--Reference links-->
[1]: https://technet.microsoft.com/library/cc731844(v=WS.10).aspx
