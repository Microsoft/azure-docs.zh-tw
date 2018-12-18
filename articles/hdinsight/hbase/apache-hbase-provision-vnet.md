---
title: 在虛擬網路上建立 HBase 叢集 - Azure
description: 開始在 Azure HDInsight 中使用 HBase。 了解如何在 Azure 虛擬網路上建立 HDInsight HBase 叢集。
services: hdinsight,virtual-network
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 02/22/2018
ms.author: jasonh
ms.openlocfilehash: a3686e99f9bab754cdc9cfc801961ea47560a111
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43043230"
---
# <a name="create-hbase-clusters-on-hdinsight-in-azure-virtual-network"></a>在 Azure 虛擬網路的 HDInsight 上建立 HBase 叢集
了解如何在 [Azure 虛擬網路][1]中建立 Azure HDInsight HBase 叢集。

由於 HBase 叢集已與虛擬網路整合，因此能夠部署到與您應用程式相同的虛擬網路，讓應用程式得以和 HBase 直接通訊。 其優點包括：

* Web 應用程式可以直接連接到 HBase 叢集節點，因而能夠使用 HBase Java 遠端程序呼叫 (RPC) API 來進行通訊。
* 透過使流量不須經過多個閘道器及負載平衡器，以提升其效能。
* 能夠以更安全的方式處理敏感資訊，而不會暴露公用端點。

### <a name="prerequisites"></a>必要條件
開始進行本教學課程之前，您必須具備下列項目：

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)。
* **具有 Azure PowerShell 的工作站**。 請參閱 [安裝及使用 Azure PowerShell](https://azure.microsoft.com/documentation/videos/install-and-use-azure-powershell/)。

## <a name="create-hbase-cluster-into-virtual-network"></a>在虛擬網路上建立 HBase 叢集
在本節中，您會使用 [Azure Resource Manager 範本](../../azure-resource-manager/resource-group-template-deploy.md)，在 Azure 虛擬網路中建立以 Linux 為基礎的 HBase 叢集與相依的 Azure 儲存體帳戶。 如需其他叢集建立方法及了解各項設定，請參閱 [建立 HDInsight 叢集](../hdinsight-hadoop-provision-linux-clusters.md)。 如需有關使用範本在 HDInsight 中建立 Hadoop 叢集的詳細資訊，請參閱 [使用 Azure Resource Manager 範本在 HDInsight 中建立 Hadoop 叢集](../hdinsight-hadoop-create-linux-clusters-arm-templates.md)

> [!NOTE]
> 某些屬性已以硬式編碼方式寫入範本。 例如︰
>
> * **位置**：美國東部 2
> * **叢集版本**：3.6
> * **叢集背景工作節點計數**：2
> * **預設儲存體帳戶**：唯一的字串
> * **虛擬網路名稱**：&lt;叢集名稱>-vnet
> * **虛擬網路位址空間**10.0.0.0/16
> * **子網路名稱**：subnet1
> * **子網路位址範圍**：10.0.0.0/24
>
> 使用範本時，&lt;叢集名稱> 會取代為您提供的叢集名稱。
>
>

1. 按一下以下影像，在 Azure 入口網站中開啟範本。 範本位在 [Azure 快速入門範本 (英文)](https://azure.microsoft.com/resources/templates/101-hdinsight-hbase-linux-vnet/)。

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-hbase-linux-vnet%2Fazuredeploy.json" target="_blank"><img src="./media/apache-hbase-provision-vnet/deploy-to-azure.png" alt="Deploy to Azure"></a>
2. 從 [自訂部署] 刀鋒視窗中，輸入下列屬性：

   * **訂用帳戶**︰選取用來建立 HDInsight 叢集、相依儲存體帳戶和 Azure 虛擬網路的 Azure 訂用帳戶。
   * **資源群組**：選取 [新建] 並指定新的資源群組名稱。
   * **位置**：選取資源群組的位置。
   * **ClusterName**：輸入要建立的 Hadoop 叢集的名稱。
   * **叢集登入名稱和密碼**：預設登入名稱是 **admin**。
   * **SSH 使用者名稱和密碼**：預設使用者名稱是 **sshuser**。  您可以將它重新命名。
   * **我同意上方所述的條款及條件**：(選取)
3. 按一下 [購買]。 大約需要 20 分鐘的時間來建立叢集。 一旦建立叢集後，您可以在入口網站按一下 [叢集] 刀鋒視窗來開啟它。

完成本教學課程之後，您可以刪除叢集。 利用 HDInsight，您的資料會儲存在 Azure 儲存體中，以便您在未使用叢集時安全地進行刪除。 您也需支付 HDInsight 叢集的費用 (即使未使用)。 由於叢集費用是儲存體費用的許多倍，所以刪除未使用的叢集符合經濟效益。 如需有關刪除叢集的指示，請參閱[使用 Azure 入口網站管理 HDInsight 中的 Hadoop 叢集](../hdinsight-administer-use-management-portal.md#delete-clusters)。

若要開始使用新的 HBase 叢集，您可以使用＜ [開始在 HDInsight 中搭配使用 HBase 與 Hadoop](./apache-hbase-tutorial-get-started-linux.md)＞中提供的程序。

## <a name="connect-to-the-hbase-cluster-using-hbase-java-rpc-apis"></a>使用 HBase Java RPC API 連接到 HBase 叢集
1. 相同的 Azure 虛擬網路和相同的子網路中建立基礎結構即服務 (IaaS) 虛擬機器。 如需建立新的 IaaS 虛擬機器的指示，請參閱[建立執行 Windows Server 的虛擬機器](../../virtual-machines/windows/quick-create-portal.md)。 當遵循本文件中的步驟時，您必須針對網路設定使用下列值：

   * **虛擬網路**：&lt;叢集名稱>-vnet
   * **子網路**：subnet1

   > [!IMPORTANT]
   > 將 &lt;叢集名稱> 取代為在上一個步驟中建立 HDInsight 叢集時使用的名稱。
   >
   >

   使用這些值會將虛擬機器放置在與 HDInsight 叢集相同的虛擬網路和子網路。 此組態可讓它們彼此直接通訊。 有一個使用空白邊緣節點建立 HDInsight 叢集的方法。 邊緣節點可用來管理叢集。  如需詳細資訊，請參閱 [Use empty edge nodes in HDInsight (在 HDInsight 中使用空白的邊緣節點)](../hdinsight-apps-use-edge-node.md)。

2. 使用 Java 應用程式從遠端連接到 HBase 時，您必須使用完整網域名稱 (FQDN)。 若要決定此名稱，您必須取得 HBase 叢集的連線特定 DNS 尾碼。 若要這麼做，您可以使用下列其中一種方法：

   * 使用網頁瀏覽器進行 Ambari 呼叫︰

     瀏覽至 https://&lt;ClusterName>.azurehdinsight.net/api/v1/clusters/&lt;ClusterName>/hosts?minimal_response=true。 結果是具有 DNS 尾碼的 JSON 檔案。
   * 使用 Ambari 網站︰

     1. 瀏覽至 https://&lt;ClusterName>.azurehdinsight.net。
     2. 按一下頂端功能表中的 [主機]  。
   * 使用 Curl 進行 REST 呼叫︰

    ```bash
        curl -u <username>:<password> -k https://<clustername>.azurehdinsight.net/ambari/api/v1/clusters/<clustername>.azurehdinsight.net/services/hbase/components/hbrest
    ```

     在傳回的 JavaScript 物件標記法 (JSON) 資料中，找出 "host_name" 項目。 其中包含叢集中節點的 FQDN。 例如︰

         ...
         "host_name": "wordkernode0.<clustername>.b1.cloudapp.net
         ...

     以叢集名稱開頭的網域名稱部分就是 DNS 尾碼。 例如，mycluster.b1.cloudapp.net。
   * 使用 Azure PowerShell

     使用下列 Azure PowerShell 指令碼來註冊 **Get-ClusterDetail** 函數，此函數可用來傳回 DNS 尾碼：

    ```powershell
        function Get-ClusterDetail(
            [String]
            [Parameter( Position=0, Mandatory=$true )]
            $ClusterDnsName,
            [String]
            [Parameter( Position=1, Mandatory=$true )]
            $Username,
            [String]
            [Parameter( Position=2, Mandatory=$true )]
            $Password,
            [String]
            [Parameter( Position=3, Mandatory=$true )]
            $PropertyName
            )
        {
        <#
            .SYNOPSIS
            Displays information to facilitate an HDInsight cluster-to-cluster scenario within the same virtual network.
            .Description
            This command shows the following 4 properties of an HDInsight cluster:
            1. ZookeeperQuorum (supports only HBase type cluster)
                Shows the value of HBase property "hbase.zookeeper.quorum".
            2. ZookeeperClientPort (supports only HBase type cluster)
                Shows the value of HBase property "hbase.zookeeper.property.clientPort".
            3. HBaseRestServers (supports only HBase type cluster)
                Shows a list of host FQDNs that run the HBase REST server.
            4. FQDNSuffix (supports all cluster types)
                Shows the FQDN suffix of hosts in the cluster.
            .EXAMPLE
            Get-ClusterDetail -ClusterDnsName {clusterDnsName} -Username {username} -Password {password} -PropertyName ZookeeperQuorum
            This command shows the value of HBase property "hbase.zookeeper.quorum".
            .EXAMPLE
            Get-ClusterDetail -ClusterDnsName {clusterDnsName} -Username {username} -Password {password} -PropertyName ZookeeperClientPort
            This command shows the value of HBase property "hbase.zookeeper.property.clientPort".
            .EXAMPLE
            Get-ClusterDetail -ClusterDnsName {clusterDnsName} -Username {username} -Password {password} -PropertyName HBaseRestServers
            This command shows a list of host FQDNs that run the HBase REST server.
            .EXAMPLE
            Get-ClusterDetail -ClusterDnsName {clusterDnsName} -Username {username} -Password {password} -PropertyName FQDNSuffix
            This command shows the FQDN suffix of hosts in the cluster.
        #>

            $DnsSuffix = ".azurehdinsight.net"

            $ClusterFQDN = $ClusterDnsName + $DnsSuffix
            $webclient = new-object System.Net.WebClient
            $webclient.Credentials = new-object System.Net.NetworkCredential($Username, $Password)

            if($PropertyName -eq "ZookeeperQuorum")
            {
                $Url = "https://" + $ClusterFQDN + "/ambari/api/v1/clusters/" + $ClusterFQDN + "/configurations?type=hbase-site&tag=default&fields=items/properties/hbase.zookeeper.quorum"
                $Response = $webclient.DownloadString($Url)
                $JsonObject = $Response | ConvertFrom-Json
                Write-host $JsonObject.items[0].properties.'hbase.zookeeper.quorum'
            }
            if($PropertyName -eq "ZookeeperClientPort")
            {
                $Url = "https://" + $ClusterFQDN + "/ambari/api/v1/clusters/" + $ClusterFQDN + "/configurations?type=hbase-site&tag=default&fields=items/properties/hbase.zookeeper.property.clientPort"
                $Response = $webclient.DownloadString($Url)
                $JsonObject = $Response | ConvertFrom-Json
                Write-host $JsonObject.items[0].properties.'hbase.zookeeper.property.clientPort'
            }
            if($PropertyName -eq "HBaseRestServers")
            {
                $Url1 = "https://" + $ClusterFQDN + "/ambari/api/v1/clusters/" + $ClusterFQDN + "/configurations?type=hbase-site&tag=default&fields=items/properties/hbase.rest.port"
                $Response1 = $webclient.DownloadString($Url1)
                $JsonObject1 = $Response1 | ConvertFrom-Json
                $PortNumber = $JsonObject1.items[0].properties.'hbase.rest.port'

                $Url2 = "https://" + $ClusterFQDN + "/ambari/api/v1/clusters/" + $ClusterFQDN + "/services/hbase/components/hbrest"
                $Response2 = $webclient.DownloadString($Url2)
                $JsonObject2 = $Response2 | ConvertFrom-Json
                foreach ($host_component in $JsonObject2.host_components)
                {
                    $ConnectionString = $host_component.HostRoles.host_name + ":" + $PortNumber
                    Write-host $ConnectionString
                }
            }
            if($PropertyName -eq "FQDNSuffix")
            {
                $Url = "https://" + $ClusterFQDN + "/ambari/api/v1/clusters/" + $ClusterFQDN + "/services/YARN/components/RESOURCEMANAGER"
                $Response = $webclient.DownloadString($Url)
                $JsonObject = $Response | ConvertFrom-Json
                $FQDN = $JsonObject.host_components[0].HostRoles.host_name
                $pos = $FQDN.IndexOf(".")
                $Suffix = $FQDN.Substring($pos + 1)
                Write-host $Suffix
            }
        }
    ```

     執行 Azure PowerShell 指令碼之後，透過下列命令使用 **Get-ClusterDetail** 函數傳回 DNS 尾碼。 使用此命令時，請指定您的 HDInsight HBase 叢集名稱、管理員名稱和管理員密碼。

    ```powershell
        Get-ClusterDetail -ClusterDnsName <yourclustername> -PropertyName FQDNSuffix -Username <clusteradmin> -Password <clusteradminpassword>
    ```

     此命令會傳回 DNS 尾碼。 例如， **yourclustername.b4.internal.cloudapp.net**。


<!--
3.    Change the primary DNS suffix configuration of the virtual machine. This enables the virtual machine to automatically resolve the host name of the HBase cluster without explicit specification of the suffix. For example, the *workernode0* host name will be correctly resolved to workernode0 of the HBase cluster.

    To make the configuration change:

    1. RDP into the virtual machine.
    2. Open **Local Group Policy Editor**. The executable is gpedit.msc.
    3. Expand **Computer Configuration**, expand **Administrative Templates**, expand **Network**, and then click **DNS Client**.
    - Set **Primary DNS Suffix** to the value obtained in step 2:

        ![hdinsight.hbase.primary.dns.suffix](./media/apache-hbase-provision-vnet/PrimaryDNSSuffix.png)
    4. Click **OK**.
    5. Reboot the virtual machine.
-->

若要驗證虛擬機器能夠與 HBase 叢集通訊，請從虛擬機器使用命令 `ping headnode0.<dns suffix>` 。 例如，ping headnode0.mycluster.b1.cloudapp.net。

若要在 Java 應用程式中使用此資訊，您可以依照＜ [使用 Maven 建置在 HDInsight (Hadoop) 上使用 HBase 的 Java 應用程式](./apache-hbase-build-java-maven-linux.md) ＞(英文) 中的步驟來建立應用程式。 若要讓應用程式連接到遠端 HBase 伺服器，請修改此範例中的 **hbase-site.xml** 檔案，以使用 Zookeeper 的 FQDN。 例如︰

    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>zookeeper0.<dns suffix>,zookeeper1.<dns suffix>,zookeeper2.<dns suffix></value>
    </property>

> [!NOTE]
> 如需 Azure 虛擬網路中名稱解析的詳細資訊，包括如何使用您自己的 DNS 伺服器，請參閱[名稱解析 (DNS)](../../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md)。
>
>

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何建立 HBase 叢集。 若要深入了解，請參閱：

* [開始使用 HDInsight](../hadoop/apache-hadoop-linux-tutorial-get-started.md)
* [在 HDInsight 中使用空白邊緣節點](../hdinsight-apps-use-edge-node.md)
* [在 HDInsight 中設定 HBase 複寫](apache-hbase-replication.md)
* [在 HDInsight 中建立 Hadoop 叢集](../hdinsight-hadoop-provision-linux-clusters.md)
* [開始在 HDInsight 中搭配使用 HBase 與 Hadoop](./apache-hbase-tutorial-get-started-linux.md)
* [虛擬網路概觀](../../virtual-network/virtual-networks-overview.md)

[1]: http://azure.microsoft.com/services/virtual-network/
[2]: http://technet.microsoft.com/library/ee176961.aspx
[3]: http://technet.microsoft.com/library/hh847889.aspx

