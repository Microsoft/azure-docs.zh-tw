---
title: 將 HDInsight 連線到內部部署網路 - Azure HDInsight
description: 了解如何在 Azure 虛擬網路中建立 HDInsight 叢集，然後將它連線到您的內部部署網路。 了解如何使用自訂的 DNS 伺服器來設定 HDInsight 與內部部署網路之間的解析名稱。
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 02/23/2018
ms.author: jasonh
ms.openlocfilehash: b9b744edbcb2abce70487c84c85a0aae71494e52
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43105863"
---
# <a name="connect-hdinsight-to-your-on-premises-network"></a>將 HDInsight 連線至內部部署網路

了解如何使用 Azure 虛擬網路和 VPN 閘道，將 HDInsight 連線到內部部署網路。 本文件提供下列規劃資訊：

* 在連線至內部部署網路的 Azure 虛擬網路中使用 HDInsight。

* 設定虛擬網路與內部部署網路之間的 DNS 名稱解析。

* 設定網路安全性群組來限制網際網路存取 HDInsight。

* HDInsight 在虛擬網路上提供的連接埠。

## <a name="create-the-virtual-network-configuration"></a>建立虛擬網路設定

使用下列文件可了解如何建立已連線到內部部署網路的 Azure 虛擬網路：
    
* [使用 Azure 入口網站](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md)

* [使用 Azure PowerShell](../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md)

* [使用 Azure CLI](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-cli.md)

## <a name="configure-name-resolution"></a>設定名稱解析

若要允許聯結網路中的 HDInsight 和資源依名稱進行通訊，您必須執行下列動作：

* 在 Azure 虛擬網路中建立自訂的 DNS 伺服器。

* 將虛擬網路設定為使用自訂的 DNS 伺服器，而不使用預設的 Azure 遞迴解析程式。

* 設定自訂的 DNS 伺服器與您的內部部署 DNS 伺服器之間的轉送。

此設定可啟用下列行為：

* 對於完整網域名稱包含__虛擬網路__之 DNS 尾碼的要求會轉寄到自訂的 DNS 伺服器。 然後，自訂的 DNS 伺服器會將這些要求轉寄到 Azure 遞迴解析程式，以傳回 IP 位址。

* 所有其他要求都會轉寄到內部部署 DNS 伺服器。 即使是公用網際網路資源 (例如 microsoft.com) 的要求也會轉寄到內部部署 DNS 伺服器進行名稱解析。

在下列圖表中，綠線表示的資源要求會以虛擬網路的 DNS 尾碼結尾。 藍線表示在內部部署網路或公用網際網路上的資源要求。

![圖表說明如何解決本文件所使用之設定中的 DNS 要求](./media/connect-on-premises-network/on-premises-to-cloud-dns.png)

### <a name="create-a-custom-dns-server"></a>建立自訂的 DNS 伺服器

> [!IMPORTANT]
> 您必須先建立和設定 DNS 伺服器，然後再將 HDInsight 安裝到虛擬網路中。

若要建立使用[繫結](https://www.isc.org/downloads/bind/) DNS 軟體的 Linux VM，請使用下列步驟：

> [!NOTE]
> 下列步驟會使用 [Azure 入口網站](https://portal.azure.com)來建立 Azure 虛擬機器。 如需建立虛擬機器的其他方式，請參閱下列文件：
>
> * [建立 VM - Azure CLI](../virtual-machines/linux/quick-create-cli.md)
> * [建立虛擬機器 - Azure PowerShell](../virtual-machines/linux/quick-create-portal.md)

1. 從 [Azure 入口網站](https://portal.azure.com)，選取__+__、__Compute__ 和 __Ubuntu Server 16.04 LTS__。

    ![建立 Ubuntu 虛擬機器](./media/connect-on-premises-network/create-ubuntu-vm.png)

2. 從 [基本概念] 區段中，輸入下列資訊：

    * __名稱__：識別此虛擬機器的易記名稱。 例如，__DNSProxy__。
    * __使用者名稱__︰SSH 帳戶的名稱。
    * __SSH 公開金鑰__或__密碼__：SSH 帳戶的驗證方法。 我們建議使用公開金鑰，因為它們較為安全。 如需詳細資訊，請參閱[建立及使用適用於 Linux VM 的 SSH 金鑰](../virtual-machines/linux/mac-create-ssh-keys.md)文件。
    * __資源群組__：選取 [使用現有]，然後選取包含稍早所建立之虛擬網路的資源群組。
    * __位置__：選取與 VM 相同的位置。

    ![虛擬機器基本設定](./media/connect-on-premises-network/vm-basics.png)

    將其他項目保留在預設值，然後選取 [確定]。

3. 從 [選擇大小] 區段中，選取 VM 大小。 在本教學課程中，選取最小與最低的成本選項。 若要繼續，請使用 [選取] 按鈕。

4. 從 [設定] 區段中，輸入下列資訊：

    * __虛擬網路__：選取您稍早建立的虛擬網路。

    * __子網路__：選取虛擬網路的預設子網路。 請__勿__選取 VPN 閘道使用的子網路。

    * __診斷儲存體帳戶__：選取現有的儲存體帳戶或建立新的儲存體帳戶。

    ![虛擬網路設定](./media/connect-on-premises-network/virtual-network-settings.png)

    將其他項目保留在預設值，然後選取 [確定] 以繼續。

5. 從 [購買] 區段中，選取 [購買] 按鈕以建立虛擬機器。

6. 一旦建立虛擬機器之後，會顯示其 [概觀] 區段。 從左側的清單中選取 [屬性]。 儲存 [公用 IP 位址] 和 [私人 IP 位址] 的值。 下一節將使用此資訊。

    ![公用和私人 IP 位址](./media/connect-on-premises-network/vm-ip-addresses.png)

### <a name="install-and-configure-bind-dns-software"></a>安裝並設定 Bind (DNS 軟體)

1. 使用 SSH 連線至虛擬機器的__公用 IP 位址__。 下列範例會連線到 40.68.254.142 的虛擬機器：

    ```bash
    ssh sshuser@40.68.254.142
    ```

    將 `sshuser` 取代為建立叢集時所指定的 SSH 使用者帳戶。

    > [!NOTE]
    > 有多種方式可取得 `ssh` 公用程式。 在 Linux、Unix 及 macOS 上，它會提供作為作業系統的一部分。 如果您是使用 Windows，請考慮下列選項的其中之一：
    >
    > * [Azure Cloud Shell](../cloud-shell/quickstart.md)
    > * [在 Windows 10 上 Ubuntu 上的 Bash](https://msdn.microsoft.com/commandline/wsl/about)
    > * [Git (https://git-scm.com/)](https://git-scm.com/)
    > * [OpenSSH (https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH)](https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH)

2. 若要安裝 Bind，使用下列 SSH 工作階段中的命令：

    ```bash
    sudo apt-get update -y
    sudo apt-get install bind9 -y
    ```

3. 若要將 Bind 設定為將名稱解析要求轉寄到您的內部部署 DNS 伺服器，請使用下列文字作為 `/etc/bind/named.conf.options` 檔案的內容：

        acl goodclients {
            10.0.0.0/16; # Replace with the IP address range of the virtual network
            10.1.0.0/16; # Replace with the IP address range of the on-premises network
            localhost;
            localnets;
        };

        options {
                directory "/var/cache/bind";

                recursion yes;

                allow-query { goodclients; };

                forwarders {
                192.168.0.1; # Replace with the IP address of the on-premises DNS server
                };

                dnssec-validation auto;

                auth-nxdomain no;    # conform to RFC1035
                listen-on { any; };
        };

    > [!IMPORTANT]
    > 將 `goodclients` 區段中的值取代為虛擬網路與內部部署網路的 IP 位址範圍。 本章節會定義此 DNS 伺服器接受要求的來源位址。
    >
    > 將 `forwarders` 區段中的 `192.168.0.1` 項目取代為內部部署 DNS 伺服器的 IP 位址。 此項目會將 DNS 要求路由到您的內部部署 DNS 伺服器以進行解析。

    若要編輯這個檔案，請使用下列命令：

    ```bash
    sudo nano /etc/bind/named.conf.options
    ```

    若要儲存檔案，請使用 __Ctrl+X__、__Y__ 和 __Enter__ 鍵。

4. 在 SSH 工作階段中，使用下列命令：

    ```bash
    hostname -f
    ```

    此命令會傳回類似下列文字的值：

        dnsproxy.icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net

    `icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net` 文字是此虛擬網路的 __DNS 尾碼__。 儲存這個值以便稍後使用。

5. 若要將 Bind 設定為解析虛擬網路內資源的 DNS 名稱，請使用下列文字作為 `/etc/bind/named.conf.local` 檔案的內容：

        // Replace the following with the DNS suffix for your virtual network
        zone "icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net" {
            type forward;
            forwarders {168.63.129.16;}; # The Azure recursive resolver
        };

    > [!IMPORTANT]
    > 您必須將 `icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net` 取代為您先前擷取的 DNS 尾碼。

    若要編輯這個檔案，請使用下列命令：

    ```bash
    sudo nano /etc/bind/named.conf.local
    ```

    若要儲存檔案，請使用 __Ctrl+X__、__Y__ 和 __Enter__ 鍵。

6. 若要啟動 Bind，請使用下列命令：

    ```bash
    sudo service bind9 restart
    ```

7. 若要確認該 Bind 可以解析內部部署網路中的資源名稱，請使用下列命令：

    ```bash
    sudo apt install dnsutils
    nslookup dns.mynetwork.net 10.0.0.4
    ```

    > [!IMPORTANT]
    > 將 `dns.mynetwork.net` 取代為內部部署網路中的資源完整網域名稱 (FQDN)。
    >
    > 將 `10.0.0.4` 取代為虛擬網路中自訂 DNS 伺服器的 __內部 IP 位址__。

    回應看起來類似下列文字：

        Server:         10.0.0.4
        Address:        10.0.0.4#53

        Non-authoritative answer:
        Name:   dns.mynetwork.net
        Address: 192.168.0.4

### <a name="configure-the-virtual-network-to-use-the-custom-dns-server"></a>將虛擬網路設定為使用自訂的 DNS 伺服器

若要將虛擬網路設定為使用自訂的 DNS 伺服器，而不使用 Azure 遞迴解析程式，請使用下列步驟：

1. 在 [Azure 入口網站](https://portal.azure.com)中，選取虛擬網路，然後選取 [DNS 伺服器]。

2. 選取 [自訂]，並輸入自訂 DNS 伺服器的__內部 IP 位址__。 最後，選取 [儲存]。

    ![設定網路的自訂 DNS 伺服器](./media/connect-on-premises-network/configure-custom-dns.png)

### <a name="configure-the-on-premises-dns-server"></a>設定內部部署 DNS 伺服器

在上一節中，您已將自訂 DNS 伺服器設定為將要求轉寄到內部部署 DNS 伺服器。 接下來，您必須將內部部署 DNS 伺服器設定為將要求轉寄到自訂 DNS 伺服器。

如需有關如何設定您 DNS 伺服器的特定步驟，請參閱您 DNS 伺服器軟體的文件。 尋找如何設定__條件式轉寄站__的步驟。

條件式轉寄只會轉寄特定 DNS 尾碼的要求。 在此情況下，您必須設定虛擬網路 DNS 尾碼的轉寄站。 應將這個尾碼的要求轉寄到自訂 DNS 伺服器的 IP 位址。 

下列文字是 **Bind** DNS 軟體條件式轉寄站設定的範例：

    zone "icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net" {
        type forward;
        forwarders {10.0.0.4;}; # The custom DNS server's internal IP address
    };

如需有關在 **Windows Server 2016** 上使用 DNS 的資訊，請參閱 [Add-DnsServerConditionalForwarderZone](https://technet.microsoft.com/itpro/powershell/windows/dnsserver/add-dnsserverconditionalforwarderzone) 文件...

一旦您設定了內部部署 DNS 伺服器後，就可以從內部部署網路使用 `nslookup`，以確認您可以解析虛擬網路中的名稱。 下列範例 

```bash
nslookup dnsproxy.icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net 196.168.0.4
```

這個範例會使用 196.168.0.4 的內部部署 DNS 伺服器來解析自訂 DNS 伺服器的名稱。 將 IP 位址取代為內部部署 DNS 伺服器的 IP 位址。 將 `dnsproxy` 位址取代為自訂 DNS 伺服器的完整網域名稱。

## <a name="optional-control-network-traffic"></a>選擇性：控制網路流量

您可以使用網路安全性群組 (NSG) 或使用者定義路由 (UDR) 來控制網路流量。 NSG 可讓您篩選輸入和輸出流量，並允許或拒絕流量。 UDR 可讓您控制虛擬網路、網際網路及內部部署網路的資源之間流量流動的方式。

> [!WARNING]
> HDInsight 需要 Azure 雲端中來自特定 IP 位址的輸入存取，以及不受限制的輸出存取。 使用 NSG 或 UDR 來控制流量時，您必須執行下列步驟：

1. 尋找包含您虛擬網路之位置的 IP 位址。 如需依位置的必要 IP 清單，請參閱[必要的 IP 位址](./hdinsight-extend-hadoop-virtual-network.md#hdinsight-ip)。

2. 針對步驟 1 中所識別的 IP 位址，允許來自那些 IP 位址的輸入流量。

   * 如果您使用 __NSG__：請針對那些 IP 位址，在連接埠 __443__ 上允許「輸入」流量。
   * 如果您使用 __UDR__：請針對那些 IP 位址，將路由的 [下一個躍點] 類型設定為 [網際網路]。

如需使用 Azure PowerShell 或 Azure CLI 建立 NSG 的範例，請參閱[使用 Azure 虛擬網路擴充 HDInsight](./hdinsight-extend-hadoop-virtual-network.md#hdinsight-nsg) 文件。

## <a name="create-the-hdinsight-cluster"></a>建立 HDInsight 叢集

> [!WARNING]
> 您必須先設定自訂 DNS 伺服器，然後再將 HDInsight 安裝到虛擬網路中。

使用[使用 Azure 入口網站建立 HDInsight 叢集](./hdinsight-hadoop-create-linux-clusters-portal.md)文件中的步驟來建立 HDInsight 叢集。

> [!WARNING]
> * 叢集建立期間，您必須選擇包含您虛擬網路的位置。
>
> * 在設定的__進階設定__部分，您必須選取稍早建立的虛擬網路與子網路。

## <a name="connecting-to-hdinsight"></a>連線至 HDInsight

HDInsight 上大部分的文件都假設您透過網際網路擁有叢集存取權。 例如，您可以連線至位於 https://CLUSTERNAME.azurehdinsight.net 的叢集。 這個位址會使用公用閘道，如果您已經使用 NSG 或 UDR 來限制網際網路的存取，則無法使用。

某些文件在從 SSH 工作階段連線到叢集時也會參考 `headnodehost`。 此位址只有從叢集內的節點才可使用，而無法在透過虛擬網路連線的用戶端上使用。

若要透過虛擬網路直接連線至 HDInsight，請使用下列步驟：

1. 若要探索 HDInsight 叢集節點的內部完整網域名稱，請使用下列方法之一：

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

2. 若要判斷可提供服務的連接埠，請參閱 [Hadoop 服務在 HDInsight 上所使用的連接埠](./hdinsight-hadoop-port-settings-for-services.md)文件。

    > [!IMPORTANT]
    > 前端節點上裝載的某些服務一次只會在一個節點上作用。 如果您嘗試在一個前端節點上存取服務但卻失敗，請切換至其他的前端節點。
    >
    > 例如，Ambari 一次只在一個前端節點上作用。 如果您嘗試在一個前端節點上存取 Ambari 而傳回 404 錯誤，表示它正在其他前端節點上執行中。

## <a name="next-steps"></a>後續步驟

* 如需在虛擬網路中使用 HDInsight 的詳細資訊，請參閱[使用 Azure 虛擬網路擴充 HDInsight](./hdinsight-extend-hadoop-virtual-network.md)。

* 如需有關 Azure 虛擬網路的詳細資訊，請參閱 [Azure 虛擬網路概觀](../virtual-network/virtual-networks-overview.md)。

* 如需網路安全性群組的詳細資訊，請參閱[網路安全性群組](../virtual-network/security-overview.md)。

* 如需使用者定義路由的詳細資訊，請參閱[使用者定義路由和 IP 轉送](../virtual-network/virtual-networks-udr-overview.md)。
