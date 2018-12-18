---
title: 關於 Azure 中使用 Azure Site Recovery 進行 Azure 災害復原的網路功能 | Microsoft Docs
description: 提供使用 Azure Site Recovery 複寫 Azure VM 的網路功能概觀。
services: site-recovery
author: sujayt
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 07/06/2018
ms.author: sujayt
ms.openlocfilehash: 77c445920041653ffb72d31e1dcfe4c368fb6642
ms.sourcegitcommit: a06c4177068aafc8387ddcd54e3071099faf659d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/09/2018
ms.locfileid: "37915920"
---
# <a name="about-networking-in-azure-to-azure-replication"></a>關於 Azure 中進行 Azure 複寫的網路功能



本文提供您使用 [Azure Site Recovery](site-recovery-overview.md)，將 Azure VM 從一個區域複寫和復原到另一個區域時的網路功能指引。

## <a name="before-you-start"></a>開始之前

了解 Site Recovery 如何針對[這種情況](azure-to-azure-architecture.md)提供災害復原。

## <a name="typical-network-infrastructure"></a>一般網路基礎結構

下圖說明適用於 Azure VM 上執行之應用程式的一般 Azure 環境：

![客戶環境](./media/site-recovery-azure-to-azure-architecture/source-environment.png)

如果您使用 Azure ExpressRoute 或從內部部署網路到 Azure 的 VPN 連線，環境將如下所示：

![客戶環境](./media/site-recovery-azure-to-azure-architecture/source-environment-expressroute.png)

通常會使用防火牆和網路安全性群組 (NSG) 來保護網路。 防火牆會使用以 URL 或 IP 為基礎的白名單來控制網路連線能力。 NSG 提供規則，使用 IP 位址範圍來控制網路連線能力。

>[!IMPORTANT]
> Site Recovery 不支援使用經過驗證的 Proxy 來控制網路連線能力，而且無法啟用複寫。


## <a name="outbound-connectivity-for-urls"></a>URL 的輸出連線能力

如果您使用以 URL 為基礎的防火牆 Proxy 來控制輸出連線能力，請允許這些 Site Recovery URL：


**URL** | **詳細資料**  
--- | ---
*.blob.core.windows.net | 需要此項目方可從 VM 將資料寫入來源地區的快取儲存體帳戶中。
login.microsoftonline.com | 需要此項目方可進行 Site Recovery 服務 URL 的授權和驗證。
*.hypervrecoverymanager.windowsazure.com | 需要此項目方可從 VM 進行 Site Recovery 服務通訊。
*.servicebus.windows.net | 需要此項目方可從 VM 寫入 Site Recovery 監視和診斷資料。

## <a name="outbound-connectivity-for-ip-address-ranges"></a>IP 位址範圍的輸出連線能力

如果您使用以 IP 為基礎的防火牆 Proxy 或 NSG 規則來控制輸出連線能力，就必須允許這些 IP 範圍。

- 對應至來源區域儲存體帳戶的所有 IP 位址範圍
    - 為這個來源地區建立[儲存體服務標記](../virtual-network/security-overview.md#service-tags)型 NSG 規則。
    - 允許這些位址，方可從 VM 將該資料寫入到快取儲存體帳戶。
- 對應至 Office 365 [驗證與身分識別 IP V4 端點](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2#bkmk_identity)的所有 IP 位址範圍。
    - 如果未來將新的位址新增至 Office 365 範圍，您必須建立新的 NSG 規則。
- Site Recovery 服務端點 IP 位址 - 在 [XML 檔案](https://aka.ms/site-recovery-public-ips)中提供，並依據您的目標位置。
-  您可以[下載並使用此指令碼](https://aka.ms/nsg-rule-script) \(英文\)，在 NSG 上自動建立所需的規則。
- 建議您在測試 NSG 上建立必要的 NSG 規則，並確認沒有問題後，再於生產 NSG 上建立規則。


站台復原 IP 位址範圍如下：

   **目標** | **Site Recovery IP** |  **Site Recovery 監視 IP**
   --- | --- | ---
   東亞 | 52.175.17.132 | 13.94.47.61
   東南亞 | 52.187.58.193 | 13.76.179.223
   印度中部 | 52.172.187.37 | 104.211.98.185
   印度南部 | 52.172.46.220 | 104.211.224.190
   美國中北部 | 23.96.195.247 | 168.62.249.226
   北歐 | 40.69.212.238 | 52.169.18.8
   西歐 | 52.166.13.64 | 40.68.93.145
   美國東部 | 13.82.88.226 | 104.45.147.24
   美國西部 | 40.83.179.48 | 104.40.26.199
   美國中南部 | 13.84.148.14 | 104.210.146.250
   美國中部 | 40.69.144.231 | 52.165.34.144
   美國東部 2 | 52.184.158.163 | 40.79.44.59
   日本東部 | 52.185.150.140 | 138.91.1.105
   日本西部 | 52.175.146.69 | 138.91.17.38
   巴西南部 | 191.234.185.172 | 23.97.97.36
   澳洲東部 | 104.210.113.114 | 191.239.64.144
   澳大利亞東南部 | 13.70.159.158 | 191.239.160.45
   加拿大中部 | 52.228.36.192 | 40.85.226.62
   加拿大東部 | 52.229.125.98 | 40.86.225.142
   美國中西部 | 52.161.20.168 | 13.78.149.209
   美國西部 2 | 52.183.45.166 | 13.66.228.204
   英國西部 | 51.141.3.203 | 51.141.14.113
   英國南部 | 51.140.43.158 | 51.140.189.52
   英國南部 2 | 13.87.37.4| 13.87.34.139
   英國北部 | 51.142.209.167 | 13.87.102.68
   南韓中部 | 52.231.28.253 | 52.231.32.85
   南韓南部 | 52.231.298.185 | 52.231.200.144
   法國中部 | 52.143.138.106 | 52.143.136.55
   法國南部 | 52.136.139.227 |52.136.136.62


## <a name="example-nsg-configuration"></a>範例 NSG 設定

這個範例示範如何針對要複寫的 VM 設定 NSG 規則。

- 如果您使用 NSG 規則來控制輸出連線，請針對所有必要的 IP 位址範圍，將「允許 HTTPS 輸出」規則應用至連接埠：443。
- 此範例假設 VM 來源位置為「美國東部」，而目標位置為「美國中部」。

### <a name="nsg-rules---east-us"></a>NSG 規則：美國東部

1. 在 NSG 上針對 "Storage.EastUS" 建立輸出 HTTPS (443) 安全性規則，如以下螢幕擷取畫面所示。

      ![storage-tag](./media/azure-to-azure-about-networking/storage-tag.png)

2. 針對那些對應至 Office 365 [驗證和識別 IP V4 端點](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2#bkmk_identity)的所有 IP 位址範圍，建立輸出 HTTPS (443) 規則。
3. 針對那些對應至目標位置的站台復原 IP，建立輸出 HTTPS (443) 規則：

   **位置** | **Site Recovery 位址** |  **Site Recovery 監視 IP 位址**
    --- | --- | ---
   美國中部 | 40.69.144.231 | 52.165.34.144

### <a name="nsg-rules---central-us"></a>NSG 規則：美國中部

需要這些規則，才能啟用從目標區域到來源地區容錯移轉後的複寫：

1. 在 NSG 上針對 "Storage.CentralUS" 建立輸出 HTTPS (443) 安全性規則。

2. 針對那些對應至 Office 365 [驗證和識別 IP V4 端點](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2#bkmk_identity)的所有 IP 位址範圍，建立輸出 HTTPS (443) 規則。

3. 針對那些對應至來源位置的站台復原 IP 建立輸出 HTTPS (443) 規則：

   **位置** | **Site Recovery 位址** |  **Site Recovery 監視 IP 位址**
    --- | --- | ---
   美國中部 | 13.82.88.226 | 104.45.147.24

## <a name="network-virtual-appliance-configuration"></a>網路虛擬設備設定

如果您使用網路虛擬設備 (NVA) 來控制 VM 的輸出網路流量，那麼只要所有的複寫流量都通過 NVA，該設備便有可能進行節流。 建議您在虛擬網路中為「儲存體」建立一個網路服務端點，這樣複寫流量就不會流向 NVA。

### <a name="create-network-service-endpoint-for-storage"></a>為儲存體建立網路服務端點
您可以在虛擬網路中為「儲存體」建立一個網路服務端點，這樣複寫流量就不會離開 Azure 範圍。

- 選取您的 Azure 虛擬網路，然後按一下 [服務端點]。

    ![storage-endpoint](./media/azure-to-azure-about-networking/storage-service-endpoint.png)

- 按一下 [新增]，[新增服務端點] 索引標籤隨即開啟。
- 選取 [服務] 底下的 [Microsoft.Storage]，接下來選取 [子網路] 底下的必要子網路，然後按一下 [新增]。

>[!NOTE]
>不要限制虛擬網路存取您用於 ASR 的儲存體帳戶。 您應該允許從 [所有網路] 存取。

### <a name="forced-tunneling"></a>強制通道

您可以使用[自訂路由](../virtual-network/virtual-networks-udr-overview.md#custom-routes)覆寫 0.0.0.0/0 位址前置詞的 Azure 預設系統路由，並將 VM 流量導向內部部署網路虛擬設備 (NVA)，但不建議將此設定用於 Site Recovery 複寫。 如果您使用自訂路由，則應該在虛擬網路中為「儲存體」[建立虛擬網路服務端點](azure-to-azure-about-networking.md#create-network-service-endpoint-for-storage)，這樣一來，複寫流量就不會離開 Azure 界限。

## <a name="next-steps"></a>後續步驟
- [複寫 Azure 虛擬機器](site-recovery-azure-to-azure.md)來開始保護您的工作負載。
- 深入了解如何針對 Azure 虛擬機器容錯移轉[保留 IP 位址](site-recovery-retain-ip-azure-vm-failover.md)。
- 深入了解 [Azure 虛擬機器與 ExpressRoute](azure-vm-disaster-recovery-with-expressroute.md) 的災害復原。
