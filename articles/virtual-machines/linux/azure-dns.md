---
title: Azure 中 Linux 虛擬機器的 DNS 名稱解析選項
description: Azure IaaS 中 Linux 虛擬機器的名稱解析案例 (包括提供的 DNS 服務)：混合式外部 DNS 和自備 DNS 伺服器。
services: virtual-machines
documentationcenter: na
author: RicksterCDN
manager: jeconnoc
editor: tysonn
ms.assetid: 787a1e04-cebf-4122-a1b4-1fcf0a2bbf5f
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/19/2016
ms.author: rclaus
ms.openlocfilehash: 93614d4889c9c884f25c5e05cd620e8303226323
ms.sourcegitcommit: 99a6a439886568c7ff65b9f73245d96a80a26d68
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2018
ms.locfileid: "39357761"
---
# <a name="dns-name-resolution-options-for-linux-virtual-machines-in-azure"></a>Azure 中 Linux 虛擬機器的 DNS 名稱解析選項
Azure 預設會提供單一虛擬網路中所有虛擬機器的 DNS 名稱解析。 您可以在 Azure 託管的虛擬機器上設定專屬 DNS 服務，以實作專屬 DNS 名稱解析解決方案。 下列案例應該可協助您選擇哪一種適合您的情況。

* [Azure 提供的名稱解析](#name-resolution-that-azure-provides)
* [使用專屬 DNS 伺服器的名稱解析](#name-resolution-using-your-own-dns-server)

您使用的名稱解析類型取決於虛擬機器和角色執行個體如何彼此通訊。

下表說明各種案例和對應的名稱解析解決方案：

| **案例** | **方案** | **尾碼** |
| --- | --- | --- |
| 相同虛擬網路中的角色執行個體或虛擬機器之間的名稱解析 |[Azure 提供的名稱解析](#azure-provided-name-resolution) |主機名稱或完整網域名稱 (FQDN) |
| 不同虛擬網路中的角色執行個體或虛擬機器之間的名稱解析 |客戶受控的 DNS 伺服器將虛擬網路之間的查詢轉送供 Azure (DNS Proxy) 解析。 請參閱[使用專屬 DNS 伺服器的名稱解析](#name-resolution-using-your-own-dns-server)。 |僅 FQDN |
| 解析 Azure 中角色執行個體或虛擬機器的內部部署電腦及伺服器名稱 |客戶管理的 DNS 伺服器 (例如，內部部署的網域控制站、本機唯讀網域控制站或使用區域傳輸同步的次要 DNS)。 請參閱[使用專屬 DNS 伺服器的名稱解析](#name-resolution-using-your-own-dns-server)。 |僅 FQDN |
| 從內部部署電腦解析 Azure 主機名稱 |將查詢轉送到所對應虛擬網路中客戶管理的 DNS Proxy 伺服器。 Proxy 伺服器將查詢轉送給 Azure 進行解析。 請參閱[使用專屬 DNS 伺服器的名稱解析](#name-resolution-using-your-own-dns-server)。 |僅 FQDN |
| 內部 IP 的反向 DNS |[使用專屬 DNS 伺服器的名稱解析](#name-resolution-using-your-own-dns-server) |n/a |

## <a name="name-resolution-that-azure-provides"></a>Azure 提供的名稱解析
除了公用 DNS 名稱的解析之外，Azure 也提供相同虛擬網路中的虛擬機器和角色執行個體的內部名稱解析。 在 Azure Resource Manager 型虛擬網路中，DNS 尾碼在虛擬網路之間是一致的，因此不需要 FQDN。 DNS 名稱可以指派給網路介面卡 (NIC) 和虛擬機器。 雖然 Azure 提供的名稱解析不需要任何設定，但它不適用於所有部署案例，如上表所示。

### <a name="features-and-considerations"></a>功能和注意事項
**功能：**

* 不需要設定，就能使用 Azure 提供的名稱解析。
* Azure 提供的名稱解析服務高度可用。 您不需要建立和管理專屬 DNS 伺服器的叢集。
* Azure 提供的名稱解析服務可與專屬 DNS 伺服器搭配使用，以解析內部部署及 Azure 主機名稱。
* 在虛擬網路的虛擬機器之間提供名稱解析，不需要 FQDN。
* 您可以使用最能描述部署的主機名稱，而不是使用自動產生的名稱。

<bpt id="p1">**</bpt>Considerations:<ept id="p1">**</ept>

* Azure 建立的 DNS 尾碼不能修改。
* 您無法手動註冊您自己的記錄。
* 不支援 WINS 和 NetBIOS。
* 主機名稱必須是 DNS 相容。
    名稱只能使用 0-9、a-z 和 '-'，無法以 '-' 開始或結束。 請參閱 RFC 3696 第 2 節。
* 每部虛擬機器的 DNS 查詢流量已經過節流。 節流應該不會影響大部分的應用程式。  如果觀察到要求節流，請確定用戶端快取已啟用。  如需詳細資訊，請參閱[充分利用 Azure 提供的名稱解析](#getting-the-most-from-name-resolution-that-azure-provides)。

### <a name="getting-the-most-from-name-resolution-that-azure-provides"></a>充分利用 Azure 提供的名稱解析
**用戶端快取：**

某些 DNS 查詢不會跨網路傳送。 用戶端快取可藉由解析本機快取的週期性 DNS 查詢，協助減少延遲以及改善網路不一致的恢復能力。 DNS 記錄包含存留時間 (TTL)，可讓快取盡可能長時間儲存記錄而不會影響記錄的有效性。 因此，用戶端快取適用於大部分的情況。

某些 Linux 發行版本預設不包含快取功能。 建議您在已經確認沒有本機快取之後，為每部 Linux 虛擬機器新增快取。

有數個不同的 DNS 快取封裝可供使用，例如 dnsmasq。 以下是在最常見的發行版本上安裝 dnsmasq 的步驟：

**Ubuntu (使用 resolvconf)**
  * 安裝 dnsmasq 封裝 (“sudo apt-get install dnsmasq”)。

SUSE (使用 netconf)：
1. 安裝 dnsmasq 封裝 (“sudo zypper install dnsmasq”)。
2. 啟用 dnsmasq 服務 (“systemctl enable dnsmasq.service”)。
3. 啟動 dnsmasq 服務 (“systemctl start dnsmasq.service”)。
4. 編輯 “/etc/sysconfig/network/config” 並將 NETCONFIG_DNS_FORWARDER="" 變更為 ”dnsmasq”。
5. 更新 resolv.conf ("netconfig update") 來設定快取作為本機 DNS 解析程式。

**CentOS by Rogue Wave Software (先前稱為 OpenLogic；使用 NetworkManager)**
1. 安裝 dnsmasq 封裝 (“sudo yum install dnsmasq”)。
2. 啟用 dnsmasq 服務 (“systemctl enable dnsmasq.service”)。
3. 啟動 dnsmasq 服務 (“systemctl start dnsmasq.service”)。
4. 將 “prepend domain-name-servers 127.0.0.1;” 新增至 “/etc/dhclient-eth0.conf”。
5. 重新啟動網路服務 (「服務網路重新啟動」) 來設定快取作為本機 DNS 解析程式

> [!NOTE]
> ：'dnsmasq' 封裝只是許多適用於 Linux 之 DNS 快取的其中一個。 在您使用它之前，請檢查您需求的適用性，而且沒有安裝其他快取。
>
>

**用戶端重試**

DNS 主要是 UDP 通訊協定。 因為 UDP 通訊協定並不保證訊息傳遞，所以 DNS 通訊協定本身會處理重試邏輯。 每個 DNS 用戶端 (作業系統) 可以展現不同的重試邏輯，根據建立者喜好設定而定：

* Windows 作業系統會在 1 秒後重試，然後再依序隔 2、4、4 秒後重試。
* 預設 Linux 安裝程式會在 5 秒之後重試。  您應該將此變更為以 1 秒的間隔重試 5 次。  

檢查 Linux 虛擬機器上目前的設定 'cat /etc/resolv.conf'，並查看 'options' 行，例如：

    options timeout:1 attempts:5

resolv.conf 檔案會自動產生且不可編輯。 新增 [選項] 行的特定步驟會因發行版本而有所不同：

**Ubuntu** (使用 resolvconf)
1. 將選項行新增至 '/etc/resolveconf/resolv.conf.d/head'。
2. 執行 'resolvconf -u' 以進行更新。

**SUSE** (使用 netconf)
1. 將 'timeout:1 attempts:5' 新增至 '/etc/sysconfig/network/config' 中的 NETCONFIG_DNS_RESOLVER_OPTIONS="" 參數。
2. 執行 'netconfig update' 以進行更新。

**CentOS by Rogue Wave Software (先前稱為 OpenLogic)** (使用 NetworkManager)
1. 將 'RES_OPTIONS="timeout:1 attempts:5"' 新增至 '/etc/sysconfig/network'。
2. 執行 'service network restart' 以進行更新。

## <a name="name-resolution-using-your-own-dns-server"></a>使用專屬 DNS 伺服器的名稱解析
您的名稱解析需求可能超過 Azure 所提供的功能。 例如，您可能需要虛擬網路之間的 DNS 解析。 為了涵蓋此案例，您可以使用專屬 DNS 伺服器。  

虛擬網路內的 DNS 伺服器可以將 DNS 查詢轉送到 Azure 的遞迴解析程式，以解析相同虛擬網路中的主機名稱。 例如，在 Azure 中執行的 DNS 伺服器可以回應其專屬 DNS 區域檔案的 DNS 查詢，並將所有其他查詢轉送到 Azure。 這項功能可讓虛擬機器查看您區域檔案中的項目，以及 Azure 提供的主機名稱 (透過轉寄站)。 Azure 遞迴解析程式的存取是透過虛擬 IP 168.63.129.16 所提供。

DNS 轉送也允許虛擬網路之間的 DNS 解析，並可讓您的內部部署電腦解析 Azure 提供的主機名稱。 為了解析虛擬機器的主機名稱，DNS 伺服器虛擬機器必須位於同一個虛擬網路中，且設定為將主機名稱查詢轉送到 Azure。 因為每個虛擬網路的 DNS 尾碼都不同，所以您可以使用條件性轉送規則來將 DNS 查詢傳送到正確的虛擬網路進行解析。 下圖顯示使用此方法進行虛擬網路間 DNS 解析的兩個虛擬網路及一個內部部署網路：

![虛擬網路之間的 DNS 解析](./media/azure-dns/inter-vnet-dns.png)

當您使用 Azure 提供的名稱解析時，會提供內部 DNS 尾碼給每部使用 DHCP 的虛擬機器。 當您使用自己的名稱解析解決方案時，不會提供此尾碼給虛擬機器，因為此尾碼會干擾其他 DNS 架構。 若要透過 FQDN 參照電腦，或要設定虛擬機器上的尾碼，您可以使用 PowerShell 或 API 來決定尾碼：

* 針對 Azure Resource Manager 所管理的虛擬網路，可透過[網路介面卡](https://msdn.microsoft.com/library/azure/mt163668.aspx)資源取得尾碼。 您也可以執行 `azure network public-ip show <resource group> <pip name>` 命令來顯示您公用 IP 的詳細資料 (包括 NIC 的 FQDN)。

如果將查詢轉送到 Azure 不符合您的需求，您就需要提供專屬的 DNS 解決方案。  您的 DNS 解決方案需要：

* 提供適當的主機名稱解析，例如透過 [DDNS](../../virtual-network/virtual-networks-name-resolution-ddns.md)。 如果您使用 DDNS，則可能需要停用 DNS 記錄清除。 Azure 的 DHCP 租用期很長，而清除可能會提前移除 DNS 記錄。
* 提供適當的遞迴解析來允許外部網域名稱的解析。
* 可從其服務的用戶端存取 (連接埠 53 上的 TCP 和 UDP)，且能夠存取網際網路。
* 受保護以防止來自網際網路的存取，降低外部代理程式的威脅。

> [!NOTE]
> 為了達到最佳效能，當您在 Azure DNS 伺服器中使用虛擬機器時，請停用 IPv6，並將[執行個體層級公用 IP](../../virtual-network/virtual-networks-instance-level-public-ip.md) 指派給每部 DNS 伺服器虛擬機器。  
>
>
