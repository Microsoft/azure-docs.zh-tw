---
title: 使用虛擬網路延伸 HDInsight - Azure
description: 了解如何使用 Azure 虛擬網路將 HDInsight 連接到其他雲端資源或您的資料中心內的資源
services: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 07/26/2018
ms.openlocfilehash: 98c62f54e2413bd67600db182c452d0d5965f239
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46972176"
---
# <a name="extend-azure-hdinsight-using-an-azure-virtual-network"></a>使用 Azure 虛擬網路延伸 Azure HDInsight

[!INCLUDE [classic-cli-warning](../../includes/requires-classic-cli.md)]

了解如何搭配使用 HDInsight 與 [Azure 虛擬網路](../virtual-network/virtual-networks-overview.md)。 使用 Azure 虛擬網路可啟用下列案例：

* 從內部部署網路直接連線至 HDInsight。

* 將 HDInsight 連線至 Azure 虛擬網路中的資料存放區。

* 直接存取無法透過網際網路公開使用的 Hadoop 服務。 例如，Kafka API 或 HBase Java API。

> [!WARNING]
> 本文件中的資訊需要了解 TCP/IP 網路。 如果您不熟悉 TCP/IP 網路，則應該與之前在生產網路中修改的人員合作。

> [!IMPORTANT]
> 如果您要尋找使用 Azure 虛擬網路將 HDInsight 連線到內部部署網路的逐步指引，請參閱[將 HDInsight 連線至內部部署網路](connect-on-premises-network.md)文件。

## <a name="planning"></a>規劃

規劃在虛擬網路中安裝 HDInsight 時，您必須回答的問題如下：

* 您需要將 HDInsight 安裝到現有虛擬網路嗎？ 或者，您要建立新的網路嗎？

    如果您要使用現有虛擬網路，則可能需要先修改網路設定，才能安裝 HDInsight。 如需詳細資訊，請參閱[將 HDInsight 新增至現有虛擬網路](#existingvnet)一節。

* 您要將包含 HDInsight 的虛擬網路連線至另一個虛擬網路或內部部署網路嗎？

    若要輕鬆地跨網路使用資源，您可能需要建立自訂 DNS，並設定 DNS 轉送。 如需詳細資訊，請參閱[連線多個網路](#multinet)一節。

* 您要將輸入或輸出流量限制/重新導向至 HDInsight 嗎？

    HDInsight 必須具有 Azure 資料中心內特定 IP 位址的無限制通訊。 另外還有數個連接埠必須允許通過防火牆才能進行用戶端通訊。 如需詳細資訊，請參閱[控制網路流量](#networktraffic)一節。

## <a id="existingvnet"></a>將 HDInsight 新增至現有虛擬網路

使用本節中的步驟，以了解如何將新的 HDInsight 新增至現有 Azure 虛擬網路。

> [!NOTE]
> 您無法在虛擬網路中新增現有的 HDInsight 叢集。

1. 您要使用虛擬網路的傳統或 Resource Manager 部署模型嗎？

    HDInsight 3.4 和更新版本需要 Resource Manager 虛擬網路。 舊版 HDInsight 需要傳統虛擬網路。

    如果您的現有網路是傳統虛擬網路，則必須建立 Resource Manager 虛擬網路，然後連線兩者。 [將傳統 VNet 連線至新的 VNet](../vpn-gateway/vpn-gateway-connect-different-deployment-models-portal.md)。

    加入之後，Resource Manager 網路中所安裝的 HDInsight 就可以與傳統網路中的資源互動。

2. 您使用強制通道嗎？ 強制通道是一種子網路設定，可強制裝置的輸出網際網路流量以進行檢查和記錄。 HDInsight 不支援強制通道。 先移除強制通道，再將 HDInsight 安裝至子網路，或建立 HDInsight 的新子網路。

3. 您使用網路安全性群組、使用者定義路由或虛擬網路設備來限制傳入或傳出虛擬網路的流量嗎？

    HDInsight 是一個受控服務，需要 Azure 資料中心內數個 IP 位址的無限制存取權。 若要允許與這些 IP 位址的通訊，請更新任何現有網路安全性群組或使用者定義路由。

    HDInsight 會裝載多個使用各種連接埠的服務。 不會封鎖對這些連接埠的流量。 如需允許通過虛擬設備防火牆的連接埠清單，請參閱[安全性](#security)一節。

    若要尋找現有安全性設定，請使用下列 Azure PowerShell 或 Azure 傳統 CLI 命令：

    * 網路安全性群組

        ```powershell
        $resourceGroupName = Read-Input -Prompt "Enter the resource group that contains the virtual network used with HDInsight"
        get-azurermnetworksecuritygroup -resourcegroupname $resourceGroupName
        ```

        ```azurecli-interactive
        read -p "Enter the name of the resource group that contains the virtual network: " RESOURCEGROUP
        az network nsg list --resource-group $RESOURCEGROUP
        ```

        如需詳細資訊，請參閱[為網路安全性群組疑難排解](../virtual-network/diagnose-network-traffic-filter-problem.md)文件。

        > [!IMPORTANT]
        > 會根據規則優先順序依序套用網路安全性群組規則。 會套用第一個符合流量模式的規則，而且未針對該流量套用其他規則。 排序從最寬鬆到最嚴格權限的規則。 如需詳細資訊，請參閱[使用網路安全性群組來篩選網路流量](../virtual-network/security-overview.md)文件。

    * 使用者定義的路由

        ```powershell
        $resourceGroupName = Read-Input -Prompt "Enter the resource group that contains the virtual network used with HDInsight"
        get-azurermroutetable -resourcegroupname $resourceGroupName
        ```

        ```azurecli-interactive
        read -p "Enter the name of the resource group that contains the virtual network: " RESOURCEGROUP
        az network route-table list --resource-group $RESOURCEGROUP
        ```

        如需詳細資訊，請參閱[為路由疑難排解](../virtual-network/diagnose-network-routing-problem.md)文件。

4. 建立 HDInsight 叢集，並在設定期間選擇 Azure 虛擬網路。 使用下列文件中的步驟，以了解叢集建立程序：

    * [使用 Azure 入口網站建立 HDInsight](hdinsight-hadoop-create-linux-clusters-portal.md)
    * [使用 Azure PowerShell 建立 HDInsight](hdinsight-hadoop-create-linux-clusters-azure-powershell.md)
    * [使用 Azure 傳統 CLI 建立 HDInsight](hdinsight-hadoop-create-linux-clusters-azure-cli.md)
    * [使用 Azure Resource Manager 範本建立 HDInsight](hdinsight-hadoop-create-linux-clusters-arm-templates.md)

  > [!IMPORTANT]
  > 將 HDInsight 新增至虛擬網路是選擇性的設定步驟。 請務必在設定叢集時選取虛擬網路。

## <a id="multinet"></a>連線多個網路

多網路設定的最大挑戰是網路之間的名稱解析。

Azure 會針對安裝於虛擬網路中的 Azure 服務提供名稱解析。 這個內建名稱解析允許 HDInsight 使用完整網域名稱 (FQDN) 連線至下列資源：

* 網際網路上的任何可用資源。 例如，microsoft.com、google.com。

* 相同 Azure 虛擬網路中的任何資源，方法是使用資源的「內部 DNS 名稱」。 例如，使用預設名稱解析時，以下是指派給 HDInsight 背景工作節點的範例內部 DNS 名稱：

    * wn0-hdinsi.0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net
    * wn2-hdinsi.0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net

    這些節點可以使用內部 DNS 名稱彼此直接通訊，以及與 HDInsight 中的其他節點通訊。

預設名稱解析「不」允許 HDInsight 解析加入虛擬網路之網路中的資源名稱。 例如，通常會將內部部署網路加入虛擬網路。 只使用預設名稱解析，HDInsight 無法透過名稱存取內部部署網路中的資源。 反之亦然，內部部署網路中的資源無法透過名稱存取虛擬網路中的資源。

> [!WARNING]
> 您必須建立自訂 DNS 伺服器，並設定虛擬網路使用它，再建立 HDInsight 叢集。

若要啟用虛擬網路與所加入網路中資源之間的名稱解析，您必須執行下列動作：

1. 在要安裝 HDInsight 的 Azure 虛擬網路中建立自訂 DNS 伺服器。

2. 將虛擬網路設定為使用自訂 DNS 伺服器。

3. 尋找虛擬網路的 Azure 指派 DNS 尾碼。 此值與 `0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net` 類似。 如需尋找 DNS 尾碼的資訊，請參閱[範例：自訂 DNS](#example-dns) 一節。

4. 設定 DNS 伺服器之間的轉送。 設定取決於遠端網路類型。

    * 如果遠端網路是內部部署網路，請設定 DNS，如下所示：
        
        * 「自訂 DNS」(在虛擬網路中)：

            * 將虛擬網路 DNS 尾碼的要求轉送至 Azure 遞迴解析程式 (168.63.129.16)。 Azure 會處理虛擬網路中資源的要求

            * 將所有其他要求轉送至內部部署 DNS 伺服器。 內部部署 DNS 會處理所有其他名稱解析要求，即使是網際網路資源 (例如 Microsoft.com) 的要求也是一樣。

        * 內部部署 DNS：將虛擬網路 DNS 尾碼的要求轉送至自訂 DNS 伺服器。 自訂 DNS 伺服器接著會轉送至 Azure 遞迴解析程式。

        此設定會將完整網域名稱包含虛擬網路 DNS 尾碼的要求路由傳送至自訂 DNS 伺服器。 內部部署 DNS 伺服器會處理所有其他要求 (即使是針對公用網際網路位址)。

    * 如果遠端網路是另一個 Azure 虛擬網路，請設定 DNS，如下所示：

        * 「自訂 DNS」(在每個虛擬網路中)：

            * 將虛擬網路 DNS 尾碼的要求轉送至自訂 DNS 伺服器。 每個虛擬網路中的 DNS 會負責解析其網路內的資源。

            * 將所有其他要求轉送至 Azure 遞迴解析程式。 遞迴解析程式負責解析本機和網際網路資源。

        根據 DNS 尾碼，每個網路的 DNS 伺服器都會將要求轉送至另一個。 其他要求是使用 Azure 遞迴解析程式進行解析。

    如需每個設定的範例，請參閱[範例：自訂 DNS](#example-dns) 一節。

如需詳細資訊，請參閱 [VM 和角色執行個體的名稱解析](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md)文件。

## <a name="directly-connect-to-hadoop-services"></a>直接連線至 Hadoop 服務

HDInsight 上大部分的文件都假設您透過網際網路擁有叢集存取權。 例如，您可以連線至位於 https://CLUSTERNAME.azurehdinsight.net 的叢集。 這個位址會使用公用閘道，如果您已經使用 NSG 或 UDR 來限制網際網路的存取，則無法使用。

若要透過虛擬網路連線至 Ambari 和其他網頁，請使用下列步驟：

1. 若要探索 HDInsight 叢集節點的內部完整網域名稱 (FQDN)，請使用下列其中一種方法：

    ```powershell
    $resourceGroupName = "The resource group that contains the virtual network used with HDInsight"

    $clusterNICs = Get-AzureRmNetworkInterface -ResourceGroupName $resourceGroupName | where-object {$_.Name -like "*node*"}

    $nodes = @()
    foreach($nic in $clusterNICs) {
        $node = new-object System.Object
        $node | add-member -MemberType NoteProperty -name "Type" -value $nic.Name.Split('-')[1]
        $node | add-member -MemberType NoteProperty -name "InternalIP" -value $nic.IpConfigurations.PrivateIpAddress
        $node | add-member -MemberType NoteProperty -name "InternalFQDN" -value $nic.DnsSettings.InternalFqdn
        $nodes += $node
    }
    $nodes | sort-object Type
    ```

    ```azurecli
    az network nic list --resource-group <resourcegroupname> --output table --query "[?contains(name,'node')].{NICname:name,InternalIP:ipConfigurations[0].privateIpAddress,InternalFQDN:dnsSettings.internalFqdn}"
    ```

    在所傳回的節點清單中，尋找前端節點的 FQDN 並使用這些 FQDN 來連線至 Ambari 和其他 Web 服務。 例如，使用 `http://<headnode-fqdn>:8080` 存取 Ambari。

    > [!IMPORTANT]
    > 前端節點上裝載的某些服務一次只會在一個節點上作用。 如果您嘗試在一個前端節點上存取服務，但傳回 404 錯誤，請切換至其他的前端節點。

2. 若要判斷可提供服務的節點和連接埠，請參閱 [Hadoop 服務在 HDInsight 上所使用的連接埠](./hdinsight-hadoop-port-settings-for-services.md)文件。

## <a id="networktraffic"></a> 控制網路流量

Azure 虛擬網路中的網路流量可以使用下列方法進行控制：

* **網路安全性群組** (NSG) 可讓您篩選輸入和輸出網路流量。 如需詳細資訊，請參閱[使用網路安全性群組來篩選網路流量](../virtual-network/security-overview.md)文件。

    > [!WARNING]
    > HDInsight 不支援限制輸出流量。

* **使用者定義路由** (UDR) 定義流量在網路中的資源之間如何流動。 如需詳細資訊，請參閱[使用者定義路由和 IP 轉送](../virtual-network/virtual-networks-udr-overview.md)文件。

* **網路虛擬設備**會複寫裝置的功能，例如防火牆和路由器。 如需詳細資訊，請參閱[網路設備](https://azure.microsoft.com/solutions/network-appliances)文件。

HDInsight 是一個受控服務，需要 Azure 雲端中 Azure 健康狀態和管理服務的無限制存取權。 使用 NSG 和 UDR 時，您必須確定這些服務仍然可以與 HDInsight 進行通訊。

HDInsight 會在數個連接埠上公開服務。 使用虛擬設備防火牆時，您必須允許用於這些服務之連接埠的流量。 如需詳細資訊，請參閱[必要連接埠]一節。

### <a id="hdinsight-ip"></a> 具有網路安全性群組和使用者定義路由的 HDInsight

如果您規劃使用**網路安全性群組**或**使用者定義路由**來控制網路流量，請先執行下列動作，再安裝 HDInsight：

1. 識別您要用於 HDInsight 的 Azure 區域。

2. 識別 HDInsight 所需的 IP 位址。 如需詳細資訊，請參閱 [HDInsight 所需的 IP 位址](#hdinsight-ip)一節。

3. 建立或修改您要安裝 HDInsight 之子網路的網路安全性群組或使用者定義路由。

    * __網路安全性群組__：允許連接埠 __443__ 上來自 IP 位址的「輸入」流量。
    * __使用者定義路由__：建立每個 IP 位址的路由，並將 [下一個躍點類型] 設定為 [網際網路]。

如需網路安全性群組或使用者定義路由的詳細資訊，請參閱下列文件：

* [網路安全性群組](../virtual-network/security-overview.md)

* [使用者定義路由](../virtual-network/virtual-networks-udr-overview.md)

#### <a name="forced-tunneling"></a>強制通道

強制通道是一種使用者定義路由設定，其中來自子網路的所有流量都會強制流向特定網路或位置，例如內部部署網路。 HDInsight「不」支援強制通道。

## <a id="hdinsight-ip"></a> 所需的 IP 位址

> [!IMPORTANT]
> Azure 健康狀態和管理服務必須能夠與 HDInsight 通訊。 如果您使用網路安全性群組或使用者定義的路由，請允許來自這些服務之 IP 位址的流量到達 HDInsight。
>
> 如果您未使用網路安全性群組或使用者定義的路由來控制流量，可以忽略這個章節。

如果您是使用網路安全性群組或使用者定義的路由，必須允許來自 Azure 健康情況和管理服務的流量觸達 HDInsight。 您可以使用下列步驟來尋找必須允許的 IP 位址：

1. 您必須一律允許來自下列 IP 位址的流量：

    | IP 位址 | 允許的連接埠 | 方向 |
    | ---- | ----- | ----- |
    | 168.61.49.99 | 443 | 輸入 |
    | 23.99.5.239 | 443 | 輸入 |
    | 168.61.48.131 | 443 | 輸入 |
    | 138.91.141.162 | 443 | 輸入 |

2. 如果您的 HDInsight 叢集是位於下列其中一個區域中，就必須允許來自針對區域所列的 IP 位址之流量：

    > [!IMPORTANT]
    > 如果未列出您使用的 Azure 區域，就只能使用步驟 1 中的四個 IP 位址。

    | 國家 (地區) | 區域 | 允許的 IP 位址 | 允許的連接埠 | 方向 |
    | ---- | ---- | ---- | ---- | ----- |
    | 亞洲 | 東亞 | 23.102.235.122</br>52.175.38.134 | 443 | 輸入 |
    | &nbsp; | 東南亞 | 13.76.245.160</br>13.76.136.249 | 443 | 輸入 |
    | 澳大利亞 | 澳洲東部 | 104.210.84.115</br>13.75.152.195 | 443 | 輸入 |
    | &nbsp; | 澳大利亞東南部 | 13.77.2.56</br>13.77.2.94 | 443 | 輸入 |
    | 巴西 | 巴西南部 | 191.235.84.104</br>191.235.87.113 | 443 | 輸入 |
    | 加拿大 | 加拿大東部 | 52.229.127.96</br>52.229.123.172 | 443 | 輸入 |
    | &nbsp; | 加拿大中部 | 52.228.37.66</br>52.228.45.222 | 443 | 輸入 |
    | 中國 | 中國北部 | 42.159.96.170</br>139.217.2.219 | 443 | 輸入 |
    | &nbsp; | 中國東部 | 42.159.198.178</br>42.159.234.157 | 443 | 輸入 |
    | 歐洲 | 北歐 | 52.164.210.96</br>13.74.153.132 | 443 | 輸入 |
    | &nbsp; | 西歐| 52.166.243.90</br>52.174.36.244 | 443 | 輸入 |
    | 德國 | 德國中部 | 51.4.146.68</br>51.4.146.80 | 443 | 輸入 |
    | &nbsp; | 德國東北部 | 51.5.150.132</br>51.5.144.101 | 443 | 輸入 |
    | 印度 | 印度中部 | 52.172.153.209</br>52.172.152.49 | 443 | 輸入 |
    | &nbsp; | 印度南部 | 104.211.223.67<br/>104.211.216.210 | 443 | 輸入 |
    | 日本 | 日本東部 | 13.78.125.90</br>13.78.89.60 | 443 | 輸入 |
    | &nbsp; | 日本西部 | 40.74.125.69</br>138.91.29.150 | 443 | 輸入 |
    | 南韓 | 南韓中部 | 52.231.39.142</br>52.231.36.209 | 433 | 輸入 |
    | &nbsp; | 南韓南部 | 52.231.203.16</br>52.231.205.214 | 443 | 輸入
    | 英國 | 英國西部 | 51.141.13.110</br>51.141.7.20 | 443 | 輸入 |
    | &nbsp; | 英國南部 | 51.140.47.39</br>51.140.52.16 | 443 | 輸入 |
    | 美國 | 美國中部 | 13.67.223.215</br>40.86.83.253 | 443 | 輸入 |
    | &nbsp; | 美國東部 | 13.82.225.233</br>40.71.175.99 | 443 | 輸入 |
    | &nbsp; | 美國中北部 | 157.56.8.38</br>157.55.213.99 | 443 | 輸入 |
    | &nbsp; | 美國中西部 | 52.161.23.15</br>52.161.10.167 | 443 | 輸入 |
    | &nbsp; | 美國西部 | 13.64.254.98</br>23.101.196.19 | 443 | 輸入 |
    | &nbsp; | 美國西部 2 | 52.175.211.210</br>52.175.222.222 | 443 | 輸入 |

    如需用於 Azure Government 之 IP 位址的資訊，請參閱 [Azure Government Intelligence + Analytics](https://docs.microsoft.com/azure/azure-government/documentation-government-services-intelligenceandanalytics) 文件。

3. 如果您搭配使用自訂 DNS 伺服器與虛擬網路，則也必須允許從 __168.63.129.16__ 進行存取。 此位址是 Azure 遞迴解析程式。 如需詳細資訊，請參閱 [VM 和角色執行個體的名稱解析](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md)文件。

如需詳細資訊，請參閱[控制網路流量](#networktraffic)一節。

## <a id="hdinsight-ports"></a> 所需連接埠

如果您要使用網路**虛擬設備防火牆**來保護虛擬網路，則必須允許下列連接埠的輸出流量：

* 53
* 443
* 1433
* 11000-11999
* 14000-14999

如需特定服務的連接埠清單，請參閱 [HDInsight 上 Hadoop 服務所使用的連接埠](hdinsight-hadoop-port-settings-for-services.md)文件。

如需虛擬設備防火牆規則的詳細資訊，請參閱[虛擬設備案例](../virtual-network/virtual-network-scenario-udr-gw-nva.md)文件。

## <a id="hdinsight-nsg"></a>範例：網路安全性群組與 HDInsight

本節中的範例會示範如何建立網路安全性群組規則，以允許 HDInsight 與 Azure 管理服務進行通訊。 使用範例之前，請調整 IP 位址以符合您要使用之 Azure 區域的 IP 位址。 您可以在[具有網路安全性群組和使用者定義路由的 HDInsight](#hdinsight-ip) 一節中找到這項資訊。

### <a name="azure-resource-management-template"></a>Azure 資源管理範本

下列資源管理範本會建立限制輸入流量的虛擬網路，但允許來自 HDInsight 所需 IP 位址的流量。 此範本也會在虛擬網路中建立 HDInsight 叢集。

* [部署安全的 Azure 虛擬網路和 HDInsight Hadoop 叢集](https://azure.microsoft.com/resources/templates/101-hdinsight-secure-vnet/)

> [!IMPORTANT]
> 建立此範例中使用的 IP 位址，以符合您使用的 Azure 區域。 您可以在[具有網路安全性群組和使用者定義路由的 HDInsight](#hdinsight-ip) 一節中找到這項資訊。

### <a name="azure-powershell"></a>Azure PowerShell

使用下列 PowerShell 指令碼建立限制輸入流量的虛擬網路，並允許來自北歐區域之 IP 位址的流量。

> [!IMPORTANT]
> 建立此範例中使用的 IP 位址，以符合您使用的 Azure 區域。 您可以在[具有網路安全性群組和使用者定義路由的 HDInsight](#hdinsight-ip) 一節中找到這項資訊。

```powershell
$vnetName = "Replace with your virtual network name"
$resourceGroupName = "Replace with the resource group the virtual network is in"
$subnetName = "Replace with the name of the subnet that you plan to use for HDInsight"
# Get the Virtual Network object
$vnet = Get-AzureRmVirtualNetwork `
    -Name $vnetName `
    -ResourceGroupName $resourceGroupName
# Get the region the Virtual network is in.
$location = $vnet.Location
# Get the subnet object
$subnet = $vnet.Subnets | Where-Object Name -eq $subnetName
# Create a Network Security Group.
# And add exemptions for the HDInsight health and management services.
$nsg = New-AzureRmNetworkSecurityGroup `
    -Name "hdisecure" `
    -ResourceGroupName $resourceGroupName `
    -Location $location `
    | Add-AzureRmNetworkSecurityRuleConfig `
        -name "hdirule1" `
        -Description "HDI health and management address 52.164.210.96" `
        -Protocol "*" `
        -SourcePortRange "*" `
        -DestinationPortRange "443" `
        -SourceAddressPrefix "52.164.210.96" `
        -DestinationAddressPrefix "VirtualNetwork" `
        -Access Allow `
        -Priority 300 `
        -Direction Inbound `
    | Add-AzureRmNetworkSecurityRuleConfig `
        -Name "hdirule2" `
        -Description "HDI health and management 13.74.153.132" `
        -Protocol "*" `
        -SourcePortRange "*" `
        -DestinationPortRange "443" `
        -SourceAddressPrefix "13.74.153.132" `
        -DestinationAddressPrefix "VirtualNetwork" `
        -Access Allow `
        -Priority 301 `
        -Direction Inbound `
    | Add-AzureRmNetworkSecurityRuleConfig `
        -Name "hdirule3" `
        -Description "HDI health and management 168.61.49.99" `
        -Protocol "*" `
        -SourcePortRange "*" `
        -DestinationPortRange "443" `
        -SourceAddressPrefix "168.61.49.99" `
        -DestinationAddressPrefix "VirtualNetwork" `
        -Access Allow `
        -Priority 302 `
        -Direction Inbound `
    | Add-AzureRmNetworkSecurityRuleConfig `
        -Name "hdirule4" `
        -Description "HDI health and management 23.99.5.239" `
        -Protocol "*" `
        -SourcePortRange "*" `
        -DestinationPortRange "443" `
        -SourceAddressPrefix "23.99.5.239" `
        -DestinationAddressPrefix "VirtualNetwork" `
        -Access Allow `
        -Priority 303 `
        -Direction Inbound `
    | Add-AzureRmNetworkSecurityRuleConfig `
        -Name "hdirule5" `
        -Description "HDI health and management 168.61.48.131" `
        -Protocol "*" `
        -SourcePortRange "*" `
        -DestinationPortRange "443" `
        -SourceAddressPrefix "168.61.48.131" `
        -DestinationAddressPrefix "VirtualNetwork" `
        -Access Allow `
        -Priority 304 `
        -Direction Inbound `
    | Add-AzureRmNetworkSecurityRuleConfig `
        -Name "hdirule6" `
        -Description "HDI health and management 138.91.141.162" `
        -Protocol "*" `
        -SourcePortRange "*" `
        -DestinationPortRange "443" `
        -SourceAddressPrefix "138.91.141.162" `
        -DestinationAddressPrefix "VirtualNetwork" `
        -Access Allow `
        -Priority 305 `
        -Direction Inbound `
# Set the changes to the security group
Set-AzureRmNetworkSecurityGroup -NetworkSecurityGroup $nsg
# Apply the NSG to the subnet
Set-AzureRmVirtualNetworkSubnetConfig `
    -VirtualNetwork $vnet `
    -Name $subnetName `
    -AddressPrefix $subnet.AddressPrefix `
    -NetworkSecurityGroup $nsg
$vnet | Set-AzureRmVirtualNetwork
```

> [!IMPORTANT]
> 此範例示範如何新增規則，以允許所需 IP 位址上的輸入流量。 它不會包含可限制其他來源之輸入存取的規則。
>
> 下列範例示範如何啟用從網際網路進行 SSH 存取：
>
> ```powershell
> Add-AzureRmNetworkSecurityRuleConfig -Name "SSH" -Description "SSH" -Protocol "*" -SourcePortRange "*" -DestinationPortRange "22" -SourceAddressPrefix "*" -DestinationAddressPrefix "VirtualNetwork" -Access Allow -Priority 306 -Direction Inbound
> ```

### <a name="azure-classic-cli"></a>Azure 傳統 CLI

使用下列步驟建立限制輸入流量的虛擬網路，但允許來自 HDInsight 所需 IP 位址的流量。

1. 使用下列命令來建立名為 `hdisecure`的新網路安全性群組。 使用包含 Azure 虛擬網路的資源群組來取代 **RESOURCEGROUPNAME**。 使用群組建立所在的位置 (區域) 來取代 **LOCATION**。

    ```azurecli
    az network nsg create -g RESOURCEGROUPNAME -n hdisecure -l LOCATION
    ```

    建立群組之後，您會收到新群組的相關資訊。

2. 使用下列將規則新增至新的網路安全性群組，這些規則允許從 Azure HDInsight 健康狀態和管理服務透過連接埠 443 的輸入通訊。 將 **RESOURCEGROUPNAME** 取代為包含 Azure 虛擬網路的資源群組名稱。

    > [!IMPORTANT]
    > 建立此範例中使用的 IP 位址，以符合您使用的 Azure 區域。 您可以在[具有網路安全性群組和使用者定義路由的 HDInsight](#hdinsight-ip) 一節中找到這項資訊。

    ```azurecli
    az network nsg rule create -g RESOURCEGROUPNAME --nsg-name hdisecure -n hdirule1 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "52.164.210.96" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 300 --direction "Inbound"
    az network nsg rule create -g RESOURCEGROUPNAME --nsg-name hdisecure -n hdirule2 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "13.74.153.132" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 301 --direction "Inbound"
    az network nsg rule create -g RESOURCEGROUPNAME --nsg-name hdisecure -n hdirule2 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "168.61.49.99" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 302 --direction "Inbound"
    az network nsg rule create -g RESOURCEGROUPNAME --nsg-name hdisecure -n hdirule2 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "23.99.5.239" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 303 --direction "Inbound"
    az network nsg rule create -g RESOURCEGROUPNAME --nsg-name hdisecure -n hdirule2 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "168.61.48.131" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 304 --direction "Inbound"
    az network nsg rule create -g RESOURCEGROUPNAME --nsg-name hdisecure -n hdirule2 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "138.91.141.162" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 305 --direction "Inbound"
    ```

3. 若要擷取此網路安全性群組的唯一識別碼，請使用下列命令：

    ```azurecli
    az network nsg show -g RESOURCEGROUPNAME -n hdisecure --query 'id'
    ```

    此命令會傳回類似下列文字的值：

        "/subscriptions/SUBSCRIPTIONID/resourceGroups/RESOURCEGROUPNAME/providers/Microsoft.Network/networkSecurityGroups/hdisecure"

    如果未獲得預期的結果，請在命令中使用雙引號括住識別碼。

4. 使用下列命令將網路安全性群組套用至子網路。 使用上一個步驟傳回的值取代 __GUID__ 和 __RESOURCEGROUPNAME__ 的值。 將 __VNETNAME__ 和 __SUBNETNAME__ 取代為您想要建立的虛擬網路名稱和子網路名稱。

    ```azurecli
    az network vnet subnet update -g RESOURCEGROUPNAME --vnet-name VNETNAME --name SUBNETNAME --set networkSecurityGroup.id="/subscriptions/GUID/resourceGroups/RESOURCEGROUPNAME/providers/Microsoft.Network/networkSecurityGroups/hdisecure"
    ```

    此命令完成之後，您可以將 HDInsight 安裝至虛擬網路。

> [!IMPORTANT]
> 這些步驟只會開啟 Azure 雲端上 HDInsight 健康狀態和管理服務的存取權。 任何其他來自虛擬網路外部之對 HDInsight 叢集的存取都會遭到封鎖。 若要允許從外部虛擬網路存取，您必須新增額外的網路安全性群組規則。
>
> 下列範例示範如何啟用從網際網路進行 SSH 存取：
>
> ```azurecli
> az network nsg rule create -g RESOURCEGROUPNAME --nsg-name hdisecure -n hdirule5 --protocol "*" --source-port-range "*" --destination-port-range "22" --source-address-prefix "*" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 306 --direction "Inbound"
> ```

## <a id="example-dns"></a> 範例：DNS 設定

### <a name="name-resolution-between-a-virtual-network-and-a-connected-on-premises-network"></a>虛擬網路與已連線內部部署網路之間的名稱解析

此範例做出以下假設：

* 您的 Azure 虛擬網路會使用 VPN 閘道連線至內部部署網路。

* 虛擬網路中的自訂 DNS 伺服器會執行 Linux 或 Unix 作為作業系統。

* [Bind](https://www.isc.org/downloads/bind/) 安裝在自訂 DNS 伺服器上。

在虛擬網路的自訂 DNS 伺服器上：

1. 若要尋找虛擬網路的 DNS 尾碼，請使用 Azure PowerShell 或 Azure 傳統 CLI：

    ```powershell
    $resourceGroupName = Read-Input -Prompt "Enter the resource group that contains the virtual network used with HDInsight"
    $NICs = Get-AzureRmNetworkInterface -ResourceGroupName $resourceGroupName
    $NICs[0].DnsSettings.InternalDomainNameSuffix
    ```

    ```azurecli-interactive
    read -p "Enter the name of the resource group that contains the virtual network: " RESOURCEGROUP
    az network nic list --resource-group $RESOURCEGROUP --query "[0].dnsSettings.internalDomainNameSuffix"
    ```

2. 在虛擬網路的自訂 DNS 伺服器上，使用下列文字作為 `/etc/bind/named.conf.local` 檔案的內容：

    ```
    // Forward requests for the virtual network suffix to Azure recursive resolver
    zone "0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net" {
        type forward;
        forwarders {168.63.129.16;}; # Azure recursive resolver
    };
    ```

    將 `0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net` 值取代為虛擬網路的 DNS 尾碼。

    此設定會將虛擬網路 DNS 尾碼的所有 DNS 要求都路由傳送至 Azure 遞迴解析程式。

2. 在虛擬網路的自訂 DNS 伺服器上，使用下列文字作為 `/etc/bind/named.conf.options` 檔案的內容：

    ```
    // Clients to accept requests from
    // TODO: Add the IP range of the joined network to this list
    acl goodclients {
        10.0.0.0/16; # IP address range of the virtual network
        localhost;
        localnets;
    };

    options {
            directory "/var/cache/bind";

            recursion yes;

            allow-query { goodclients; };

            # All other requests are sent to the following
            forwarders {
                192.168.0.1; # Replace with the IP address of your on-premises DNS server
            };

            dnssec-validation auto;

            auth-nxdomain no;    # conform to RFC1035
            listen-on { any; };
    };
    ```
    
    * 將 `10.0.0.0/16` 值取代為虛擬網路的 IP 位址範圍。 此項目允許來自此範圍內位址的名稱解析要求。

    * 將內部部署網路的 IP 位址範圍新增至 `acl goodclients { ... }` 區段。  項目允許來自內部部署網路中資源的名稱解析要求。
    
    * 將 `192.168.0.1` 值取代為內部部署 DNS 伺服器的 IP 位址。 此項目會將所有其他 DNS 要求路由傳送至內部部署 DNS 伺服器。

3. 若要使用設定，請重新啟動 Bind。 例如： `sudo service bind9 restart`。

4. 將條件式轉寄站新增至內部部署 DNS 伺服器。 設定條件式轉寄站，以將步驟 1 中之 DNS 尾碼的要求傳送至自訂 DNS 伺服器。

    > [!NOTE]
    > 如需如何新增條件式轉寄站的詳細資訊，請參閱 DNS 軟體的文件。

完成這些步驟之後，您可以使用完整網域名稱 (FQDN) 連線至任一虛擬網路中的資源。 您現在可以將 HDInsight 安裝至虛擬網路。

### <a name="name-resolution-between-two-connected-virtual-networks"></a>兩個已連線虛擬網路之間的名稱解析

此範例做出以下假設：

* 您的兩個 Azure 虛擬網路是使用 VPN 閘道或對等進行連線。

* 兩個網路中的自訂 DNS 伺服器會執行 Linux 或 Unix 作為作業系統。

* [Bind](https://www.isc.org/downloads/bind/) 安裝在自訂 DNS 伺服器上。

1. 若要尋找兩個虛擬網路的 DNS 尾碼，請使用 Azure PowerShell 或 Azure 傳統 CLI：

    ```powershell
    $resourceGroupName = Read-Input -Prompt "Enter the resource group that contains the virtual network used with HDInsight"
    $NICs = Get-AzureRmNetworkInterface -ResourceGroupName $resourceGroupName
    $NICs[0].DnsSettings.InternalDomainNameSuffix
    ```

    ```azurecli-interactive
    read -p "Enter the name of the resource group that contains the virtual network: " RESOURCEGROUP
    az network nic list --resource-group $RESOURCEGROUP --query "[0].dnsSettings.internalDomainNameSuffix"
    ```

2. 使用下列文字作為自訂 DNS 伺服器上 `/etc/bind/named.config.local` 檔案的內容。 在這兩個虛擬網路的自訂 DNS 伺服器上進行這項變更。

    ```
    // Forward requests for the virtual network suffix to Azure recursive resolver
    zone "0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net" {
        type forward;
        forwarders {10.0.0.4;}; # The IP address of the DNS server in the other virtual network
    };
    ```

    將 `0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net` 值取代為「其他」虛擬網路的 DNS 尾碼。 此項目會將遠端網路 DNS 尾碼的要求路由傳送至該網路中的自訂 DNS。

3. 在這兩個虛擬網路的自訂 DNS 伺服器上，使用下列文字作為 `/etc/bind/named.conf.options` 檔案的內容：

    ```
    // Clients to accept requests from
    acl goodclients {
        10.1.0.0/16; # The IP address range of one virtual network
        10.0.0.0/16; # The IP address range of the other virtual network
        localhost;
        localnets;
    };

    options {
            directory "/var/cache/bind";

            recursion yes;

            allow-query { goodclients; };

            forwarders {
            168.63.129.16;   # Azure recursive resolver         
            };

            dnssec-validation auto;

            auth-nxdomain no;    # conform to RFC1035
            listen-on { any; };
    };
    ```
    
    * 將 `10.0.0.0/16` 和 `10.1.0.0/16` 值取代為虛擬網路的 IP 位址範圍。 此項目允許每個網路中的資源對 DNS 伺服器提出要求。

    不適用於虛擬網路 DNS 尾碼 (例如，microsoft.com) 的任何要求都是透過 Azure 遞迴解析程式所處理。

4. 若要使用設定，請重新啟動 Bind。 例如，兩部 DNS 伺服器上的 `sudo service bind9 restart`。

完成這些步驟之後，您可以使用完整網域名稱 (FQDN) 連線至虛擬網路中的資源。 您現在可以將 HDInsight 安裝至虛擬網路。

## <a name="next-steps"></a>後續步驟

* 如需設定 HDInsight 連線至內部部署網路的端對端範例，請參閱[將 HDInsight 連線至內部部署網路](./connect-on-premises-network.md)。
* 如需在 Azure 虛擬網路中設定 Hbase 叢集，請參閱[在 Azure 虛擬網路的 HDInsight 上建立 HBase 叢集](hbase/apache-hbase-provision-vnet.md)。
* 如需設定 HBase 異地複寫，請參閱[設定 Azure 虛擬網路中的 HBase 叢集複寫](hbase/apache-hbase-replication.md)。
* 如需 Azure 虛擬網路的詳細資訊，請參閱 [Azure 虛擬網路概觀](../virtual-network/virtual-networks-overview.md)。

* 如需網路安全性群組的詳細資訊，請參閱[網路安全性群組](../virtual-network/security-overview.md)。

* 如需使用者定義路由的詳細資訊，請參閱[使用者定義路由和 IP 轉送](../virtual-network/virtual-networks-udr-overview.md)。
