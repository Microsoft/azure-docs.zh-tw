---
title: 設定 Azure 虛擬網路中的 HBase 叢集複寫
description: 了解如何針對負載平衡、高可用性、零停機時間移轉和更新，以及災害復原來設定 HDInsight 版本之間的 HBase 複寫。
services: hdinsight,virtual-network
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 09/15/2018
ms.openlocfilehash: 51f5f3b9742de45b1b72104c8cf08079d0719763
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47224371"
---
# <a name="set-up-hbase-cluster-replication-in-azure-virtual-networks"></a>設定 Azure 虛擬網路中的 HBase 叢集複寫

了解如何在虛擬網路內或 Azure 中的兩個虛擬網路之間設定 HBase 複寫。

叢集複寫會使用來源推入方法。 HBase 叢集可以是來源、目的地，或可同時滿足兩個角色。 複寫不是同步進行。 複寫的目標最終會一致。 如果來源在複寫啟用時接收到資料行系列的編輯，該編輯會傳播到所有目的地叢集。 當資料從一個叢集複寫到另一個時，會追蹤來源叢集和已取用資料的所有叢集，以避免複寫迴圈。

在本教學課程中，您會設定來源與目的地之間的複寫。 若需其他叢集拓撲，請參閱 [Apache HBase 參考指南](http://hbase.apache.org/book.html#_cluster_replication)。

以下是適用於單一虛擬網路的 HBase 複寫使用案例︰

* 負載平衡。 例如，您可以在目的地叢集上執行掃描或 MapReduce 工作，以及在來源叢集上內嵌資料。
* 新增高可用性。
* 在 HBase 叢集之間移轉資料。
* 將 Azure HDInsight 叢集從一個版本升級到另一個版本。

以下是適用於兩個虛擬網路的 HBase 複寫使用案例︰

* 設定災害復原。
* 負載平衡與分割應用程式。
* 新增高可用性。

您可以從 [GitHub](https://github.com/Azure/hbase-utils/tree/master/replication) 使用[指令碼動作](../hdinsight-hadoop-customize-cluster-linux.md)指令碼複寫叢集。

## <a name="prerequisites"></a>必要條件
開始進行本教學課程之前，您必須擁有 Azure 訂用帳戶。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)。

## <a name="set-up-the-environments"></a>設定環境

您有三個組態選項：

- 有兩個 HBase 叢集位於單一 Azure 虛擬網路中。
- 有兩個 HBase 叢集位於相同區域但不同的兩個虛擬網路中。
- 有兩個 HBase 叢集位於不同區域且不同的兩個虛擬網路中 (異地複寫)。

本文涵蓋異地複寫案例。

為了協助您設定環境，我們建立了一些 [Azure Resource Manager 範本](../../azure-resource-manager/resource-group-overview.md)。 如果您偏好使用其他方法設定環境，請參閱：

- [在 HDInsight 中建立 Hadoop 叢集](../hdinsight-hadoop-provision-linux-clusters.md)
- [在 Azure 虛擬網路上建立 HBase 叢集](apache-hbase-provision-vnet.md)

### <a name="set-up-two-virtual-networks-in-two-different-regions"></a>在兩個不同區域中設定兩個虛擬網路

若要使用會兩不同區域建立兩個虛擬網路，並在 VNet 之間建立 VPN 連線的範本，請選取下列 [部署至 Azure] 按鈕。 範本定義會儲存於[公用 Blob 儲存體](https://hditutorialdata.blob.core.windows.net/hbaseha/azuredeploy.json)。

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fhditutorialdata.blob.core.windows.net%2Fhbaseha%2Fazuredeploy.json" target="_blank"><img src="./media/apache-hbase-replication/deploy-to-azure.png" alt="Deploy to Azure"></a>

範本中的一些硬式編碼值：

**VNet 1**

| 屬性 | 值 |
|----------|-------|
| 位置 | 美國西部 |
| VNet 名稱 | &lt;ClusterNamePrevix>-vnet1 |
| 位址空間首碼 | 10.1.0.0/16 |
| 子網路名稱 | subnet 1 |
| 子網路首碼 | 10.1.0.0/24 |
| 子網路 (閘道) 名稱 | GatewaySubnet (無法變更) |
| 子網路 (閘道) 首碼 | 10.1.255.0/27 |
| 閘道名稱 | vnet1gw |
| 閘道類型 | Vpn |
| 閘道 VPN 類型 | RouteBased |
| 閘道 SKU | 基本 |
| 閘道 IP | vnet1gwip |

**VNet 2**

| 屬性 | 值 |
|----------|-------|
| 位置 | 美國東部 |
| VNet 名稱 | &lt;ClusterNamePrevix>-vnet2 |
| 位址空間首碼 | 10.2.0.0/16 |
| 子網路名稱 | subnet 1 |
| 子網路首碼 | 10.2.0.0/24 |
| 子網路 (閘道) 名稱 | GatewaySubnet (無法變更) |
| 子網路 (閘道) 首碼 | 10.2.255.0/27 |
| 閘道名稱 | vnet2gw |
| 閘道類型 | Vpn |
| 閘道 VPN 類型 | RouteBased |
| 閘道 SKU | 基本 |
| 閘道 IP | vnet1gwip |

## <a name="setup-dns"></a>設定 DNS

在最後一節，範本會在兩個虛擬網路中各建立一個 Ubuntu 虛擬機器。  而在這一節，您會在兩個 DNS 虛擬機器上安裝 Bind，然後在兩個虛擬機器上設定 DNS 轉送。

為了安裝 Bind，您需要尋找兩個 DNS 虛擬機器的公用 IP 位址。

1. 開啟 [Azure 入口網站](https://portal.azure.com)。
2. 選取 [資源群組] > [資源群組名稱] > [vnet1DNS] 來開啟 DNS 虛擬機器。  資源群組名稱是您在上一個程序中所建立的名稱。 預設 DNS 虛擬機器名稱是 vnet1DNS 和 vnet2NDS。
3. 選取 [屬性] 以開啟虛擬網路的屬性頁面。
4. 記下 [公用 IP 位址]，並另外確認 [私人 IP 位址]。  私人 IP 位址應該是 **10.1.0.4** (如果是 vnet1DNS) 和 **10.2.0.4** (如果是 vnet2DNS)。  
5. 變更兩個虛擬網路的 DNS 伺服器，使用預設 (Azure 提供) 的 DNS 伺服器，以允許對內及對外存取，在下列步驟中下載套件來安裝 Bind。

若要安裝 Bind，請使用下列程序：

1. 使用 SSH 連線至 DNS 虛擬機器的__公用 IP 位址__。 下列範例會連線到 40.68.254.142 的虛擬機器：

    ```bash
    ssh sshuser@40.68.254.142
    ```

    將 `sshuser` 取代為建立 DNS 虛擬機器時所指定的 SSH 使用者帳戶。

    > [!NOTE]
    > 有多種方式可取得 `ssh` 公用程式。 在 Linux、Unix 及 macOS 上，它會提供作為作業系統的一部分。 如果您是使用 Windows，請考慮下列選項的其中之一：
    >
    > * [Azure Cloud Shell](../../cloud-shell/quickstart.md)
    > * [在 Windows 10 上 Ubuntu 上的 Bash](https://msdn.microsoft.com/commandline/wsl/about)
    > * [Git (https://git-scm.com/)](https://git-scm.com/)
    > * [OpenSSH (https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH)](https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH)

2. 若要安裝 Bind，使用下列 SSH 工作階段中的命令：

    ```bash
    sudo apt-get update -y
    sudo apt-get install bind9 -y
    ```

3. 將 Bind 設定為把名稱解析要求轉送到您的內部 DNS 伺服器。 若要這樣做，請使用下列文字作為 `/etc/bind/named.conf.options` 檔案的內容：

    ```
    acl goodclients {
        10.1.0.0/16; # Replace with the IP address range of the virtual network 1
        10.2.0.0/16; # Replace with the IP address range of the virtual network 2
        localhost;
        localhost;
    };
    
    options {
        directory "/var/cache/bind";
        recursion yes;
        allow-query { goodclients; };

        forwarders {
            168.63.129.16; #This is the Azure DNS server
        };

        dnssec-validation auto;

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
    };
    ```
    
    > [!IMPORTANT]
    > 將 `goodclients` 區段中的值取代為兩個虛擬網路的 IP 位址範圍。 本章節會定義此 DNS 伺服器接受要求的來源位址。

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

        vnet1DNS.icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net

    `icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net` 文字是此虛擬網路的 __DNS 尾碼__。 儲存這個值以便稍後使用。

    您也必須從另一個 DNS 伺服器找出 DNS 尾碼。 您在下一步需要用到它。

5. 若要將 Bind 設定為解析虛擬網路內資源的 DNS 名稱，請使用下列文字作為 `/etc/bind/named.conf.local` 檔案的內容：

    ```
    // Replace the following with the DNS suffix for your virtual network
    zone "v5ant3az2hbe1edzthhvwwkcse.bx.internal.cloudapp.net" {
            type forward;
            forwarders {10.2.0.4;}; # The Azure recursive resolver
    };
    ```

    > [!IMPORTANT]
    > 您必須將 `v5ant3az2hbe1edzthhvwwkcse.bx.internal.cloudapp.net` 取代為其他虛擬網路的 DNS 尾碼。 轉寄站 IP 是 DNS 伺服器在其他虛擬網路的私人 IP 位址。

    若要編輯這個檔案，請使用下列命令：

    ```bash
    sudo nano /etc/bind/named.conf.local
    ```

    若要儲存檔案，請使用 __Ctrl+X__、__Y__ 和 __Enter__ 鍵。

6. 若要啟動 Bind，請使用下列命令：

    ```bash
    sudo service bind9 restart
    ```

7. 若要確認該 Bind 可以解析另一個虛擬網路中的資源名稱，請使用下列命令：

    ```bash
    sudo apt install dnsutils
    nslookup vnet2dns.v5ant3az2hbe1edzthhvwwkcse.bx.internal.cloudapp.net
    ```

    > [!IMPORTANT]
    > 將 `vnet2dns.v5ant3az2hbe1edzthhvwwkcse.bx.internal.cloudapp.net` 取代為另一個網路中 DNS 虛擬機器的完整網域名稱 (FQDN)。
    >
    > 將 `10.2.0.4` 取代為另一個虛擬網路中自訂 DNS 伺服器的__內部 IP 位址__。

    回應看起來類似下列文字：

    ```
    Server:         10.2.0.4
    Address:        10.2.0.4#53
    
    Non-authoritative answer:
    Name:   vnet2dns.v5ant3az2hbe1edzthhvwwkcse.bx.internal.cloudapp.net
    Address: 10.2.0.4
    ```

    到目前為止，您仍無法在沒有指定 DNS 伺服器 IP 位址的情況下查閱其他網路的 IP 位址。

### <a name="configure-the-virtual-network-to-use-the-custom-dns-server"></a>將虛擬網路設定為使用自訂的 DNS 伺服器

若要將虛擬網路設定為使用自訂的 DNS 伺服器，而不使用 Azure 遞迴解析程式，請使用下列步驟：

1. 在 [Azure 入口網站](https://portal.azure.com)中，選取虛擬網路，然後選取 [DNS 伺服器]。

2. 選取 [自訂]，並輸入自訂 DNS 伺服器的__內部 IP 位址__。 最後，選取 [儲存]。

6. 在 vnet1 中開啟 DNS 伺服器虛擬機器，然後按一下 [重新啟動]。  您必須重新啟動虛擬網路中的所有虛擬機器，才能讓 DNS 組態生效。
7. 重複步驟，為 vnet2 設定自訂 DNS 伺服器。

若要測試 DNS 組態，您可以使用 SSH 連線至兩個 DNS 虛擬機器，然後使用另一個虛擬網路的 DNS 伺服器主機名稱對該 DNS 伺服器執行 ping。 如果沒有作用，請使用下列命令來檢查 DNS 狀態：

```bash
sudo service bind9 status
```

## <a name="create-hbase-clusters"></a>建立 HBase 叢集

使用下列組態在兩個虛擬網路中各建立一個 HBase 叢集：

- **資源群組名稱**︰使用和您在虛擬網路中所建立的名稱相同的資源群組名稱。
- **叢集類型**：HBase
- **版本**：HBase 1.1.2 (HDI 3.6)
- **位置**：使用與虛擬網路相同的位置。  根據預設，vnet1 是「美國西部」，vnet2 是「美國東部」。
- **儲存體**︰為叢集建立新的儲存體帳戶。
- **虛擬網路** (從入口網站上的 [進階] 設定)：選取您在上一個程序中所建立的 vnet1。
- **子網路**：範本中所使用的預設名稱為 **subnet1**。

若要確定環境的設定是否正確，您必須能夠對兩個叢集之間的前端節點 FQDN 執行 ping。

## <a name="load-test-data"></a>載入測試資料

當您複寫叢集時，您必須指定要複寫的資料表。 在本節中，您會把部分資料載入到來源叢集中。 在下一節中，您將會啟用兩個叢集之間的複寫。

若要建立一個**連絡人**資料表，並在此資料表中插入一些資料，請依照 [HBase 教學課程：開始使用 HDInsight 中的 Apache HBase](apache-hbase-tutorial-get-started-linux.md) 中的指示進行操作。

## <a name="enable-replication"></a>啟用複寫

下列步驟會說明如何從 Azure 入口網站呼叫指令碼動作指令碼。 如需了解如何使用 Azure PowerShell 和 Azure 傳統 CLI 來執行指令碼動作，請參閱[使用指令碼動作來自訂 HDInsight 叢集](../hdinsight-hadoop-customize-cluster-linux.md)。

**從 Azure 入口網站啟用 HBase 複寫**

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 開啟來源 HBase 叢集。
3. 從 [叢集] 功能表中，選擇 [指令碼動作]。
4. 在頁面的頂端，選取 [提交新項目] 。
5. 選取或輸入下列資訊︰

  1. **名稱**：輸入「啟用複寫」。
  2. **Bash 指令碼 URL**：輸入 **https://raw.githubusercontent.com/Azure/hbase-utils/master/replication/hdi_enable_replication.sh**。
  3.  **前端**：務必選取此項目。 清除其他節點類型。
  4. **參數**：下列範例參數會針對所有現有的資料表啟用複寫，然後將來源叢集的所有資料複製到目的地叢集：

          -m hn1 -s <source cluster DNS name> -d <destination cluster DNS name> -sp <source cluster Ambari password> -dp <destination cluster Ambari password> -copydata
    
    >[!note]
    >
    > 針對來源與目的地叢集 DNS 名稱，使用主機名稱而非 FQDN。

6. 選取 [建立] 。 指令碼執行需要花費一些時間，特別是在使用 **-copydata** 引數的情況下。

必要的引數︰

|名稱|說明|
|----|-----------|
|-s, --src-cluster | 指定來源 HBase 叢集的 DNS 名稱。 例如：-s hbsrccluster, --src-cluster=hbsrccluster |
|-d, --dst-cluster | 指定目的地 (複本) HBase 叢集的 DNS 名稱。 例如：-s dsthbcluster, --src-cluster=dsthbcluster |
|-sp, --src-ambari-password | 指定來源 HBase 叢集上 Ambari 的管理員密碼。 |
|-dp, --dst-ambari-password | 指定目的地 HBase 叢集上 Ambari 的管理員密碼。|

選擇性的引數︰

|名稱|說明|
|----|-----------|
|-su, --src-ambari-user | 指定來源 HBase 叢集上 Ambari 的管理員使用者名稱。 預設值為 **admin**。 |
|-du, --dst-ambari-user | 指定目的地 HBase 叢集上 Ambari 的管理員使用者名稱。 預設值為 **admin**。 |
|-t, --table-list | 指定要複寫的資料表。 例如：--table-list="table1;table2;table3"。 如果您未指定資料表，則會複寫所有現有的 HBase 資料表。|
|-m, --machine | 指定用來執行指令碼動作的前端節點。 值為 **hn1** 或 **hn0**。 因為 **hn0** 前端節點通常較忙碌，我們建議您使用 **hn1**。 如果您從 HDInsight 入口網站或 Azure PowerShell 以指令碼動作執行 $0 指令碼，則使用此選項。|
|-cp, -copydata | 在已啟用複寫的資料表上，啟用現有資料的移轉。 |
|-rpm, -replicate-phoenix-meta | 在 Phoenix 系統資料表上啟用複寫。 <br><br>*請謹慎使用此選項。* 建議您在使用此指令碼前，於複本叢集上重新建立 Phoenix 資料表。 |
|-h, --help | 顯示使用資訊。 |

[指令碼](https://github.com/Azure/hbase-utils/blob/master/replication/hdi_enable_replication.sh)的 `print_usage()` 區段有參數的詳細說明。

成功部署指令碼動作之後，您可以使用 SSH 連接到目的地 HBase 叢集，並確認已複寫資料。

### <a name="replication-scenarios"></a>複寫案例

下列清單會顯示一些一般使用情況和其參數設定︰

- **在兩個叢集之間的所有資料表上啟用複寫**。 這種情況下，資料表上不需要複製或移轉現有資料，且不會使用 Phoenix 資料表。 使用下列參數︰

        -m hn1 -s <source cluster DNS name> -d <destination cluster DNS name> -sp <source cluster Ambari password> -dp <destination cluster Ambari password>  

- **在特定資料表上啟用複寫**。 如要在 table1、table2 和 table3 上啟用複寫，請使用下列參數：

        -m hn1 -s <source cluster DNS name> -d <destination cluster DNS name> -sp <source cluster Ambari password> -dp <destination cluster Ambari password> -t "table1;table2;table3"

- **在特定資料表上啟用複寫，並複製現有資料**。 如要在 table1、table2 和 table3 上啟用複寫，請使用下列參數：

        -m hn1 -s <source cluster DNS name> -d <destination cluster DNS name> -sp <source cluster Ambari password> -dp <destination cluster Ambari password> -t "table1;table2;table3" -copydata

- **在所有資料表上啟用複寫，並將 Phoenix 中繼資料從來源複寫到目的地**。 Phoenix 中繼資料複寫並非萬無一失。 請謹慎使用。 使用下列參數︰

        -m hn1 -s <source cluster DNS name> -d <destination cluster DNS name> -sp <source cluster Ambari password> -dp <destination cluster Ambari password> -t "table1;table2;table3" -replicate-phoenix-meta

## <a name="copy-and-migrate-data"></a>複製並移轉資料

啟用複寫後，有兩個可用來複製或移轉資料的個別指令碼動作指令碼：

- [適用於小型資料表的指令碼](https://raw.githubusercontent.com/Azure/hbase-utils/master/replication/hdi_copy_table.sh) (表格約數 GB 大小，且整體複製預估會在一小時內完成)

- [適用於大型資料表的指令碼](https://raw.githubusercontent.com/Azure/hbase-utils/master/replication/nohup_hdi_copy_table.sh) (預估複製時間會超過一小時的表格)

您可以依照[啟用複寫](#enable-replication)中描述的相同程序，來呼叫指令碼動作。 使用下列參數︰

    -m hn1 -t <table1:start_timestamp:end_timestamp;table2:start_timestamp:end_timestamp;...> -p <replication_peer> [-everythingTillNow]

[指令碼](https://github.com/Azure/hbase-utils/blob/master/replication/hdi_copy_table.sh)的 `print_usage()` 區段有參數的詳細說明。

### <a name="scenarios"></a>案例

- **針對到目前為止 (目前時間戳記) 所有已編輯的資料列複製特定資料表 (test1、test2 和 test3)**：

        -m hn1 -t "test1::;test2::;test3::" -p "zk5-hbrpl2;zk1-hbrpl2;zk5-hbrpl2:2181:/hbase-unsecure" -everythingTillNow
  或：

        -m hn1 -t "test1::;test2::;test3::" --replication-peer="zk5-hbrpl2;zk1-hbrpl2;zk5-hbrpl2:2181:/hbase-unsecure" -everythingTillNow


- **以指定時間範圍複製特定資料表**：

        -m hn1 -t "table1:0:452256397;table2:14141444:452256397" -p "zk5-hbrpl2;zk1-hbrpl2;zk5-hbrpl2:2181:/hbase-unsecure"


## <a name="disable-replication"></a>停用複寫

若要停用複寫，可從 [GitHub](https://raw.githubusercontent.com/Azure/hbase-utils/master/replication/hdi_disable_replication.sh) 使用另一個指令碼動作指令碼。 您可以依照[啟用複寫](#enable-replication)中描述的相同程序，來呼叫指令碼動作。 使用下列參數︰

    -m hn1 -s <source cluster DNS name> -sp <source cluster Ambari password> <-all|-t "table1;table2;...">  

[指令碼](https://raw.githubusercontent.com/Azure/hbase-utils/master/replication/hdi_disable_replication.sh)的 `print_usage()` 區段有參數的詳細說明。

### <a name="scenarios"></a>案例

- **停用所有資料表上的複寫**：

        -m hn1 -s <source cluster DNS name> -sp Mypassword\!789 -all
  或

        --src-cluster=<source cluster DNS name> --dst-cluster=<destination cluster DNS name> --src-ambari-user=<source cluster Ambari user name> --src-ambari-password=<source cluster Ambari password>

- **停用特定資料表 (table1、table2 和 table3) 上的複寫**：

        -m hn1 -s <source cluster DNS name> -sp <source cluster Ambari password> -t "table1;table2;table3"

## <a name="next-steps"></a>後續步驟

在本文中，您已了解如何在虛擬網路內或兩個虛擬網路之間設定 HBase 複寫。 若要深入了解 HDInsight 與 HBase ，請參閱下列文章：

* [開始使用 HDInsight 中的 Apache HBase](./apache-hbase-tutorial-get-started-linux.md)
* [HDInsight HBase 概觀](./apache-hbase-overview.md)
* [在 Azure 虛擬網路上建立 HBase 叢集](./apache-hbase-provision-vnet.md)

