---
title: Microsoft Azure 雲端服務之連線能力和網路服務問題的常見問題集 | Microsoft Docs
description: 此文章列出 Microsoft Azure 雲端服務之連線能力和網路服務的相關常見問題集。
services: cloud-services
documentationcenter: ''
author: genlin
manager: cshepard
editor: ''
tags: top-support-issue
ms.assetid: 84985660-2cfd-483a-8378-50eef6a0151d
ms.service: cloud-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/23/2018
ms.author: genli
ms.openlocfilehash: 034d59c39628a08c389c5ceb67c5872bbea10d59
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47223163"
---
# <a name="connectivity-and-networking-issues-for-azure-cloud-services-frequently-asked-questions-faqs"></a>Azure 雲端服務之連線能力和網路服務問題：常見問題集 (FAQ)

此文章包含 [Azure 雲端服務](https://azure.microsoft.com/services/cloud-services)之連線能力和網路服務問題的相關常見問題集。 如需有關規模大小的資訊，請參閱[雲端服務 VM 大小頁面](cloud-services-sizes-specs.md)。

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="i-cant-reserve-an-ip-in-a-multi-vip-cloud-service"></a>無法在多 VIP 的雲端服務中保留 IP。
首先，請確定您想要保留其 IP 的虛擬機器執行個體已開啟。 其次，請確定您會將保留的 IP 同時用於預備與生產部署。 *勿* 於部署正在升級時變更設定。

## <a name="how-do-i-use-remote-desktop-when-i-have-an-nsg"></a>當我有 NSG 時，應如何使用遠端桌面？
將規則加入到 NSG，以允許連接埠 **3389** 和 **20000** 上的流量。 遠端桌面使用者連接埠 **3389**。 系統已為雲端服務執行個體進行負載平衡，因此您無法直接控制要連線的執行個體。 RemoteForwarder 與 RemoteAccess 代理程式會管理遠端桌面通訊協定 (RDP) 流量，並允許用戶端傳送 RDP Cookie 並指定要連線的個別執行個體。 RemoteForwarder 與 RemoteAccess 代理程式要求您必須開啟連接埠 **20000**，這在您使用 NSG 的情況下可能是封鎖的。

## <a name="can-i-ping-a-cloud-service"></a>是否可偵測雲端服務？

否，不能使用一般的 "ping"/ICMP 通訊協定來進行。 不允許透過 Azure Load Balancer 進行 ICMP 通訊協定。

若要測試連線能力，建議您進行連接埠偵測。 當 Ping.exe 使用 ICMP 時，您可以使用諸如 PSPing、Nmap 及 telnet 等其他工具來測試對特定 TCP 連接埠的連線。

如需詳細資訊，請參閱[使用連接埠偵測而非 ICMP 來測試 Azure VM 連線能力](https://blogs.msdn.microsoft.com/mast/2014/06/22/use-port-pings-instead-of-icmp-to-test-azure-vm-connectivity/)。

## <a name="how-do-i-prevent-receiving-thousands-of-hits-from-unknown-ip-addresses-that-might-indicate-a-malicious-attack-to-the-cloud-service"></a>如何避免接收來自未知 IP 位址的成千上萬個叫用，其可能表示對雲端服務的惡意攻擊？
Azure 會實作多層的網路安全性，可保護其平台服務免於遭受分散式阻斷服務 (DDoS) 攻擊。 Azure DDoS 防禦系統是屬於 Azure 的連續監視流程，會透過滲透測試持續進行改良。 此 DDoS 防禦系統的設計，不僅可承受來自外部的攻擊，還能承受來自其他 Azure 租用戶的攻擊。 如需詳細資訊，請參閱 [Azure 網路安全性](http://download.microsoft.com/download/C/A/3/CA3FC5C0-ECE0-4F87-BF4B-D74064A00846/AzureNetworkSecurity_v3_Feb2015.pdf)。

您也可以建立啟動工作，選擇性地封鎖一些特定的 IP 位址。 如需詳細資訊，請參閱[封鎖特定 IP 位址](cloud-services-startup-tasks-common.md#block-a-specific-ip-address)。

## <a name="when-i-try-to-rdp-to-my-cloud-service-instance-i-get-the-message-the-user-account-has-expired"></a>當我嘗試 RDP 到我的雲端服務執行個體時，會收到這個訊息：「使用者帳戶已過期」。
當您略過 RDP 設定中所設定的到期日時，可能會收到「此使用者帳戶已過期」的錯誤訊息。 您可以從入口網站中變更到期日，方法是遵循下列步驟：

1. 登入 [Azure 入口網站](https://portal.azure.com)、移至您的雲端服務，然後選取 [遠端桌面] 索引標籤。

2. 選取 [生產] 或 [預備] 部署位置。

3. 變更「**到期日**」的日期，然後儲存設定。

您現在應該能夠 RDP 到您的電腦。

## <a name="why-is-azure-load-balancer-not-balancing-traffic-equally"></a>為什麼 Azure Load Balancer 並未平均平衡流量？
如需如何內部負載平衡器運作方式的相關資訊，請參閱 [Azure Load Balancer 的新分散模式](https://azure.microsoft.com/blog/azure-load-balancer-new-distribution-mode/)。

所用的分配演算法是 5 個 Tuple (來源 IP、來源連接埠、目的地 IP、目的地連接埠和通訊協定類型) 的雜湊，將流量對應至可用的伺服器。 它只在傳輸工作階段內提供綁定。 相同 TCP 或 UDP 工作階段中的封包會被導向至負載平衡端點後面的相同資料中心 IP (DIP) 執行個體。 當用戶端關閉並重新開啟連線或從相同的來源 IP 啟動新的工作階段時，來源連接埠便會變更，進而導致流量進入不同的 DIP 端點。

## <a name="how-can-i-redirect-incoming-traffic-to-the-default-url-of-my-cloud-service-to-a-custom-url"></a>如何將我雲端服務預設 URL 的傳入流量重新導向自訂的 URL？ 

IIS 的 URL Rewrite 模組可用來將流向雲端服務 (例如，\*.cloudapp.net) 預設 URL 的流量重新導向至某些自訂名稱/URL。 因為 URL Rewrite 模組預設會在 Web 角色上啟用，且其規則是在應用程式的 web.config 中進行設定，因此一律可在 VM 上使用，與重新開機/重新安裝映像無關。 如需詳細資訊，請參閱

- [建立 URL Rewrite 模組的重寫規則](https://docs.microsoft.com/iis/extensions/url-rewrite-module/creating-rewrite-rules-for-the-url-rewrite-module)
- [移除預設的連結](https://stackoverflow.com/questions/32286487/azure-website-how-to-remove-default-link?answertab=votes#tab-top)

## <a name="how-can-i-blockdisable-incoming-traffic-to-the-default-url-of-my-cloud-service"></a>如何封鎖/停用我雲端服務的預設 URL 傳入流量？ 

您可以阻止傳向您雲端服務 (例如 \*.cloudapp.net) 預設 URL/名稱的傳入流量。 在雲端服務定義 (*.csdef) 檔案中的站台繫結組態下，將主機標頭設為自訂的 DNS 名稱 (例如 www.MyCloudService.com)，如下所示： 
 

    <?xml version="1.0" encoding="utf-8"?> 
    <ServiceDefinition name="AzureCloudServicesDemo" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition" schemaVersion="2015-04.2.6"> 
      <WebRole name="MyWebRole" vmsize="Small"> 
        <Sites> 
          <Site name="Web"> 
            <Bindings> 
              <Binding name="Endpoint1" endpointName="Endpoint1" hostHeader="www.MyCloudService.com" /> 
            </Bindings> 
          </Site> 
        </Sites> 
        <Endpoints> 
          <InputEndpoint name="Endpoint1" protocol="http" port="80" /> 
        </Endpoints> 
        <ConfigurationSettings> 
          <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" /> 
        </ConfigurationSettings> 
      </WebRole> 
    </ServiceDefinition> 
 
因為此主機標頭繫結會透過 csdef 檔強制執行，所以服務只能透過自訂名稱 "www.MyCloudService.com" 存取。 所有傳向 "*.cloudapp.net" 網域的傳入要求一律都會失敗。 如果您在服務中使用自訂的 SLB 探查或內部負載平衡器，封鎖預設的服務 URL/名稱就可能會干擾探查行為。 

## <a name="how-can-i-make-sure-the-public-facing-ip-address-of-a-cloud-service-never-changes"></a>如何確定雲端服務的公眾對應 IP 位址永遠不會變更？

若要確定您雲端服務 (也稱為 VIP) 的公眾對應 IP 位址永遠不會變更，以便幾個特定用戶端可常態地將它設為允許清單，我們建議您使用與它相關聯的保留 IP。 否則，如果您刪除部署，Azure 提供的虛擬 IP 會從您的訂用帳戶中解除配置。 若要成功地進行 VIP 交換作業，您需要有生產位置和預備位置的個別保留 IP。 如果沒有這些 IP，交換作業將會失敗。 若要保留 IP 位址，並將它與您的雲端服務相關聯，請參閱下列文章：
 
- [保留現有雲端服務的 IP 位址](../virtual-network/virtual-networks-reserved-public-ip.md#reserve-the-ip-address-of-an-existing-cloud-service)
- [使用服務組態檔建立保留的 IP 至雲端服務的關聯](../virtual-network/virtual-networks-reserved-public-ip.md#associate-a-reserved-ip-to-a-cloud-service-by-using-a-service-configuration-file) 

如果您的角色有多個執行個體時，將 RIP 與您的雲端服務建立關聯就應該不會導致任何停機時間。 或者，您可以將您 Azure 資料中心的 IP 範圍列入允許清單。 您可以在 [Microsoft 下載中心](https://www.microsoft.com/en-us/download/details.aspx?id=41653)找到所有 Azure IP 範圍。 

這個檔案包含 Azure 資料中心使用的 IP 位址範圍 (包括計算、SQL 和儲存體範圍)。 每週會公佈已更新的檔案，以反映目前已部署的範圍及任何即將進行的 IP 範圍變更。 出現在檔案中的新範圍至少有一週的時間不會在資料中心中使用。 請每週下載新的 .xml 檔案，並在您的站台上執行必要的變更，以正確識別在 Azure 中執行的服務。 Azure Express Route 使用者可能會注意到，在每個月的第一週會使用此檔案來更新 Azure 空間的 BGP 公告。 

## <a name="how-can-i-use-azure-resource-manager-virtual-networks-with-cloud-services"></a>如何使用 Azure Resource Manager 虛擬網路搭配雲端服務？ 

雲端服務不能放在 Azure Resource Manager 虛擬網路中。 資源管理員虛擬網路和傳統部署虛擬網路都可以透過對等互連來連線。 如需詳細資訊，請參閱[虛擬網路對等互連](../virtual-network/virtual-network-peering-overview.md)。


## <a name="how-can-i-get-the-list-of-public-ips-used-by-my-cloud-services"></a>如何取得我雲端服務所使用的公用 IP 清單？

您可以使用下列 PS 指令碼來取得您訂用帳戶下雲端服務的公用 IP 清單

    $services = Get-AzureService  | Group-Object -Property ServiceName

    foreach ($service in $services) 
    {
        "Cloud Service '$($service.Name)'"

        $deployment = Get-AzureDeployment -ServiceName $service.Name 
        "VIP - " +  $deployment.VirtualIPs[0].Address
        "================================="
    }
