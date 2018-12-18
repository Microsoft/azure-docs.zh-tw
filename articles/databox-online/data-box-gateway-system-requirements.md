---
title: Microsoft Azure 資料箱閘道的系統需求 | Microsoft Docs
description: 了解 Azure 資料箱閘道的軟體和網路需求
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: article
ms.date: 10/17/2018
ms.author: alkohli
ms.openlocfilehash: f70cd5de29cc3050142dfff905acae2008dab115
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49379117"
---
# <a name="azure-data-box-gateway-system-requirements-preview"></a>Azure 資料箱閘道的系統需求 (預覽)

本文會針對 Microsoft Azure 資料箱閘道解決方案以及連線至 Azure 資料箱閘道的用戶端，說明其各自的重要系統需求。 建議您先仔細檢閱此資訊再部署資料箱閘道，之後在進行部署和後續作業期間若有必要，也請回頭查閱。

資料箱閘道虛擬裝置的系統需求包括：

- **主機的軟體需求** - 說明支援的平台、本機組態 UI 的瀏覽器、SMB 用戶端，以及可以連線到裝置的主機其他需求。
- **裝置的網路需求** - 提供虛擬裝置作業的任何網路需求相關資訊。

> [!IMPORTANT]
> 資料箱閘道處於預覽狀態。 部署此解決方案之前，請先檢閱[預覽版使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。 

## <a name="supported-os-for-clients-connected-to-device"></a>支援的 OS (適用於連線至裝置的用戶端)

以下是支援的作業系統清單，這些系統適用於連線至資料箱閘道的用戶端或主機。

| **作業系統/平台** | **版本** |
| --- | --- |
| Windows Server |2012 R2 <br> 2016 |
| Windows |8, 10 |
| SUSE Linux |Enterprise Server 12 (x86_64)|
| Ubuntu |16.04.3 LTS|
| CentOS | 7.0 |

## <a name="supported-protocols-for-clients-accessing-device"></a>支援的通訊協定 (適用於存取裝置的用戶端)

|**通訊協定** |**版本**   |**注意事項**  |
|---------|---------|---------|
|SMB    | 2.X, 3.X      | 不支援 SMB 1。|
|NFS     | V3 和 V4        |         |

## <a name="supported-virtualization-platforms-for-device"></a>裝置的受支援虛擬化平台

| **作業系統/平台**  |**版本**   |**注意事項**  |
|---------|---------|---------|
|Hyper-V  |  2012 R2 <br> 2016  |         |
|VMware ESXi     | 6.0 <br> 6.5        |不支援 VMware 工具。         |


## <a name="supported-storage-accounts"></a>支援的儲存體帳戶

以下是資料箱閘道所支援的儲存體類型清單。

| **儲存體帳戶** | **注意事項** |
| --- | --- |
| 傳統 | 標準 |
| 一般用途  |標準；同時支援 V1 和 V2。 同時支援經常性儲存層和非經常性儲存層。 |


## <a name="supported-storage-types"></a>支援的儲存體類型

以下是資料箱閘道所支援的儲存體類型清單。

| **檔案格式** | **注意事項** |
| --- | --- |
| Azure 區塊 Blob | |
| Azure 分頁 Blob  | |
| Azure 檔案 | |

## <a name="supported-browsers-for-local-web-ui"></a>本機 Web UI 的受支援瀏覽器

以下是虛擬裝置本機 Web UI 支援瀏覽器的清單。

|[瀏覽器]  |版本  |其他需求/注意事項  |
|---------|---------|---------|
|Google Chrome   |最新版本         |         |
|Microsoft Edge    | 最新版本        |         |
|Internet Explorer     | 最新版本        |         |
|FireFox    |最新版本         |         |


## <a name="networking-requirements"></a>網路需求

下表列出必須在您的防火牆中開啟的連接埠，以允許 SMB、雲端或管理流量。 在這個資料表中，in 或 inbound 指的是輸入用戶端要求存取裝置的方向。 Out 或 outbound 指的是資料箱閘道裝置於外部傳送資料至部署之上的方向：例如，輸出到網際網路。

| 連接埠號碼| 內或外 | 連接埠範圍| 必要|   注意                                                             |                                                                                     |
|--------|---------|----------|--------------|----------------------|---------------|
| TCP 80 (HTTP)|外|WAN |否|輸出連接埠用於網際網路存取以擷取更新。 <br>輸出 Web Proxy 可由使用者設定。 |                          
| TCP 443 (HTTPS)|外|WAN|是|輸出連接埠用來存取雲端中的資料。<br>輸出 Web Proxy 可由使用者設定。|   
| UDP 53 (DNS)|外|WAN|在某些情況下<br>請參閱附註|只有當您使用網際網路 DNS 伺服器時，才需要此連接埠。<br>我們建議使用本機 DNS 伺服器。 |
| UDP 123 (NTP)|外|WAN|在某些情況下<br>請參閱附註|只有當您使用網際網路 NTP 伺服器時，才需要此連接埠。  |
| UDP 67 (DHCP)|外|WAN|在某些情況下<br>請參閱附註|只有當您使用 DHCP 伺服器時，才需要此連接埠。  |
| TCP 80 (HTTP)|在|LAN|是|這是裝置上用於本機管理本機 UI 的輸入連接埠。 <br>透過 HTTP 存取本機 UI 會自動重新導向至 HTTPS。  | 
| TCP 443 (HTTPS)|在|LAN|是|這是裝置上用於本機管理本機 UI 的輸入連接埠。 | 

## <a name="url-patterns-for-firewall-rules"></a>防火牆規則的 URL 模式

網路系統管理員通常可以根據 URL 模式設定進階防火牆規則，來篩選輸入和輸出流量。 您的資料箱閘道裝置和資料箱閘道服務取決於其他 Microsoft 應用程式，例如 Azure 服務匯流排、Azure Active Directory 存取控制、儲存體帳戶和 Microsoft Update 伺服器。 與這些應用程式相關聯的 URL 模式可以用來設定防火牆規則。 請務必了解與這些應用程式相關聯的 URL 模式可以變更。 接著，您將需要網路系統管理員監控資料箱閘道的防火牆規則，並在需要時更新。

我們建議您在大部分情況下，根據資料箱閘道固定 IP 位址為輸出流量設定防火牆規則。 不過，您可以使用下列資訊設定建立安全環境所需的進階防火牆規則。

> [!NOTE]
> - 裝置 (來源) IP 應該一律設定為所有啟用雲端功能的網路介面。
> - 目的地 IP 應該設定為 [Azure 資料中心 IP 範圍](https://www.microsoft.com/download/confirmation.aspx?id=41653)。

|     URL 模式                                                                                                                                                                                                                                                                                                                                                                                                                                       |     元件/功能                                                                             |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------|
|    https://*.databoxedge.azure.com/*<br>https://*.servicebus.windows.net/*<br>https://login.windows.net                                                                                                                                                                                                                                                                                                        |    Azure 資料箱閘道服務<br>Azure 服務匯流排<br>驗證服務    |
|    http://*.backup.windowsazure.com                                                                                                                                                                                                                                                                                                                                                                                                                   |    裝置啟用                                                                                    |
|    http://crl.microsoft.com/pki/*   http://www.microsoft.com/pki/*                                                                                                                                                                                                                                                                                                                                                                                    |    憑證撤銷                                                                               |
|    https://*.core.windows.net/*   https://*.data.microsoft.com   http://*.msftncsi.com                                                                                                                                                                                                                                                                                                                                                                |    Azure 儲存體帳戶和監視                                                                |
|    http://windowsupdate.microsoft.com<br>http://*.windowsupdate.microsoft.com<br>https://*.windowsupdate.microsoft.com<br>http://*.update.microsoft.com<br>https://*.update.microsoft.com<br>http://*.windowsupdate.com<br>http://download.microsoft.com<br>http://*.download.windowsupdate.com<br>http://wustat.windows.com<br>http://ntservicepack.microsoft.com<br>http://*.ws.microsoft.com<br>https://*.ws.microsoft.com<br>http://*.mp.microsoft.com        |    Microsoft Update 伺服器                                                                             |
|    http://*.deploy.akamaitechnologies.com                                                                                                                                                                                                                                                                                                                                                                                                             |    Akamai CDN                                                                                           |
|    https://*.partners.extranet.microsoft.com/*                                                                                                                                                                                                                                                                                                                                                                                                        |    支援封裝                                                                                      |
|    http://*.data.microsoft.com                                                                                                                                                                                                                                                                                                                                                                                                                        |    Windows 中的遙測服務，請參閱客戶經驗和診斷遙測的更新      |
|                                                                                                                                                                                                                                                                                                                                                                                                                                                       |                                                                                                         |



## <a name="internet-bandwidth"></a>網際網路頻寬

下列需求會套用至您資料箱閘道裝置適用的最低網際網路頻寬。

- 資料箱閘道隨時都有專用的 20 Mbps 網際網路頻寬 (或更多) 可用。 此頻寬不應與任何其他應用程式共用。 
- 資料箱閘道在使用網路節流時，有專用的 32 Mbps 網際網路頻寬 (或更多) 可用。

## <a name="next-step"></a>後續步驟

* [部署 Azure 資料箱閘道](data-box-gateway-deploy-prep.md)

