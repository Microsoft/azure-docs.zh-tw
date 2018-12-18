---
title: 包含檔案
description: 包含檔案
services: firewall
author: vhorne
ms.service: ''
ms.topic: include
ms.date: 9/14/2018
ms.author: victorh
ms.custom: include file
ms.openlocfilehash: 1242cf80ffd283471eb507612949a81d4b3fd594
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47401551"
---
### <a name="what-is-azure-firewall"></a>何謂 Azure 防火牆？

Azure 防火牆是受控、雲端式網路安全性服務，可以保護您的 Azure 虛擬網路資源。 它是完全具狀態防火牆即服務，具有內建的高可用性和不受限制的雲端延展性。 您可以橫跨訂用帳戶和虛擬網路集中建立、強制執行以及記錄應用程式和網路連線原則。

### <a name="what-capabilities-are-supported-in-azure-firewall"></a>Azure 防火牆中支援哪些功能？  

* 具狀態防火牆即服務
* 內建高可用性且不受限制的雲端延展性
* FQDN 篩選
* FQDN 標記
* 網路流量篩選規則
* 輸出 SNAT 支援
* 輸入 DNAT 支援
* 跨 Azure 訂用帳戶和 VNET，集中建立、強制執行以及記錄應用程式和網路連線原則
* 與 Azure 監視器完全整合以進行記錄和分析 

### <a name="what-is-the-pricing-for-azure-firewall"></a>Azure 防火牆的定價為何？

Azure 防火牆有固定成本和變動成本：

* 固定費用：每小時每個防火牆 1.25 美元
* 變動費用：防火牆處理的每 GB 0.03 美元 (輸入或輸出)

### <a name="what-is-the-typical-deployment-model-for-azure-firewall"></a>Azure 防火牆的一般部署模型是什麼？

您可以將「Azure 防火牆」部署在任何虛擬網路上，但客戶通常會將其部署在中央虛擬網路上，然後以中樞和支點模型方式，將其他虛擬網路與其對等互連。 接著，您便可以從對等互連的虛擬網路設定預設路由，使其指向此中央防火牆虛擬網路。

### <a name="how-can-i-install-the-azure-firewall"></a>如何安裝 Azure 防火牆？

您可以透過使用 Azure 入口網站、PowerShell、REST API 或範本，來設定「Azure 防火牆」。 如需逐步指示，請參閱[教學課程：使用 Azure 入口網站部署和設定 Azure 防火牆](../articles/firewall/tutorial-firewall-deploy-portal.md)。

### <a name="what-are-some-azure-firewall-concepts"></a>有哪些 Azure 防火牆概念？

Azure 防火牆支援規則和規則集合。 規則集合是一組共用相同順序和優先順序的規則。 規則集合會依其優先順序執行。 網路規則集合的優先順序高於應用程式規則集合，而所有規則都將終止。

規則集合有兩種類型：

* *應用程式規則*：能夠讓您設定可從子網路存取的完整網域名稱 (FQDN)。 
* *網路規則*：能夠讓您設定包含來源位址、通訊協定、目的地連接埠及目的地位址的規則。 

### <a name="does-azure-firewall-support-inbound-traffic-filtering"></a>Azure 防火牆是否支援輸入流量篩選？

Azure 防火牆支援輸入和輸出篩選。 輸入保留適用於非 HTTP/S 通訊協定。 例如 RDP、SSH 及 FTP 通訊協定。
 
### <a name="which-logging-and-analytics-services-are-supported-by-the-azure-firewall"></a>Azure 防火牆支援哪些記錄和分析服務？

「Azure 防火牆」已與「Azure 監視器」整合，可檢視和分析防火牆記錄。 記錄可以傳送至 Log Analytics、「Azure 儲存體」或「事件中樞」。 這些記錄可以在 Log Analytics 中進行分析，或透過不同的工具 (例如 Excel 和 Power BI) 進行分析。 如需詳細資訊，請參閱[教學課程：監視 Azure 防火牆記錄](../articles/firewall/tutorial-diagnostics.md)。

### <a name="how-does-azure-firewall-work-differently-from-existing-services-such-as-nvas-in-the-marketplace"></a>Azure 防火牆的運作方式與市集中現有的服務 (例如 NVA) 有何不同？

Azure 防火牆是一種基本防火牆服務，可以處理特定的客戶案例。 預期的情況是您將會混合使用協力廠商 NVA 與「Azure 防火牆」。 使用上，首重相輔相成。
 
### <a name="what-is-the-difference-between-application-gateway-waf-and-azure-firewall"></a>應用程式閘道 WAF 與 Azure 防火牆有什麼不同？

Web 應用程式防火牆 (WAF) 是應用程式閘道的一個功能，可提供 Web 應用程式的集中式輸入保護，免於遭遇常見的攻擊和弱點。 「Azure 防火牆」可為所有連接埠和通訊協定提供輸出網路層級的保護，並可為輸出 HTTP/S 提供應用程式層級的保護。 非 HTTP/S 通訊協定的輸入保護 (例如：RDP、SSH、FTP) 是針對 Azure 防火牆 GA 暫時規劃的。

### <a name="what-is-the-difference-between-network-security-groups-nsgs-and-azure-firewall"></a>網路安全性群組 (NSG) 和 Azure 防火牆有什麼不同？

「Azure 防火牆」服務可補足網路安全性群組功能。 兩者結合時，可提供更好的「深度防禦」網路安全性。 網路安全性群組提供分散式網路層流量篩選，可限制每個訂用帳戶中虛擬網路內資源的流量。 「Azure 防火牆」是完全具狀態的集中式網路防火牆即服務，可跨不同的訂用帳戶和虛擬網路，提供網路層級和應用程式層級的保護。 

### <a name="how-do-i-set-up-azure-firewall-with-my-service-endpoints"></a>如何使用我的服務端點設定 Azure 防火牆？

若要安全存取 PaaS 服務，建議使用服務端點。 您可以選擇在「Azure 防火牆」子網路中啟用服務端點，並在已連線的支點虛擬網路上停用它們。 如此一來，您便可以享有這兩項功能的好處 -- 服務端點安全性和所有流量的集中記錄。

### <a name="how-can-i-stop-and-start-azure-firewall"></a>如何停止和啟動 Azure 防火牆？

您可以使用 Azure PowerShell 的「解除配置」和「配置」方法。

例如︰

```azurepowershell
# Stop an exisitng firewall

$azfw = Get-AzureRmFirewall -Name "FW Name” -ResourceGroupName "RG Name"
$azfw.Deallocate()
Set-AzureRmFirewall -AzureFirewall $azfw
```

```azurepowershell
#Start a firewall

$vnet = Get-AzureRmVirtualNetwork -ResourceGroupName "RG Name" -Name "VNet Name"
$publicip = Get-AzureRmPublicIpAddress -Name "Public IP Name" -ResourceGroupName " RG Name"
$azfw.Allocate($vnet,$publicip)
Set-AzureRmFirewall -AzureFirewall $azfw
```

> [!NOTE]
> 您必須將防火牆和公用 IP 重新配置到原始的資源群組和訂用帳戶。

### <a name="what-are-the-known-service-limits"></a>已知的服務限制有哪些？

* 「Azure 防火牆」的軟性限制為每個月每個防火牆 1000 TB。 
* 在中央虛擬網路中執行的「Azure 防火牆」執行個體有虛擬網路對等互連限制，亦即支點虛擬網路的數目上限為 50 個。  
* Azure 防火牆無法搭配全球對等互連使用，因此您應該在每個區域都至少部署一個防火牆。
* 「Azure 防火牆」可支援 1 萬個應用程式規則與 1 萬個網路規則。

### <a name="can-azure-firewall-in-a-hub-virtual-network-forward-and-filter-network-traffic-between-two-spoke-virtual-networks"></a>中樞虛擬網路中的 Azure 防火牆是否可以在兩個輪幅虛擬網路之間轉寄和篩選網路流量？

是，您可以在中樞虛擬網路中，使用 Azure 防火牆來路由傳送和篩選兩個輪幅虛擬網路之間的流量。 每個輪幅虛擬網路中的子網路，都必須有 UDR 指向 Azure 防火牆作為此案例的預設閘道，才能正常運作。

### <a name="can-azure-firewall-forward-and-filter-network-traffic-between-subnets-in-the-same-virtual-network"></a>Azure 防火牆可以在同一個虛擬網路的子網路之間轉寄和篩選網路流量嗎？

即使 UDR 指向 Azure 防火牆作為預設閘道，同一個虛擬網路或直接對等互連的虛擬網路內子網路之間的流量還是會直接路由傳送。 對於內部網路區隔的建議方法是使用網路安全性群組。 在此案例中若要將子網路對子網路流量傳送到防火牆，UDR 必須在這兩個子網路上同時明確包含目標子網路網路首碼。
