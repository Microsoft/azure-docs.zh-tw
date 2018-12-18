---
title: 使用 Ambari REST API 監視和管理 Hadoop - Azure HDInsight
description: 了解如何使用 Ambari 來監視和管理 Azure HDInsight 中的 Hadoop 叢集。 在本文件中，您將學習如何使用 HDInsight 叢集隨附的 Ambari REST API。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 04/23/2018
ms.author: jasonh
ms.openlocfilehash: d6e2bdba7e3536404f087dc468a0895d0be0c2a0
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43106014"
---
# <a name="manage-hdinsight-clusters-by-using-the-ambari-rest-api"></a>使用 Ambari REST API 管理 HDInsight 叢集

[!INCLUDE [ambari-selector](../../includes/hdinsight-ambari-selector.md)]

了解如何使用 Ambari REST API 來管理和監視 Azure HDInsight 中的 Hadoop 叢集。

Apache Ambari 提供容易使用的 Web UI 和 REST API，可簡化 Hadoop 叢集的管理和監視。 Ambari 會納入到使用 Linux 作業系統的 HDInsight 叢集中。 您可以使用 Ambari 來監視叢集並進行設定變更。

## <a id="whatis"></a>什麼是 Ambari

[Apache Ambari](http://ambari.apache.org) 會提供 Web UI 以供用來管理及監視 Hadoop 叢集。 開發人員可以使用 [Ambari REST API](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md)將這些功能整合到應用程式。

以 Linux 為基礎的 HDInsight 叢集預設會提供 Ambari。

## <a name="how-to-use-the-ambari-rest-api"></a>如何使用 Ambari REST API

> [!IMPORTANT]
> 這份文件中的資訊和範例需要使用 Linux 作業系統的 HDInsight 叢集。 如需詳細資訊，請參閱[開始使用 HDInsight](hadoop/apache-hadoop-linux-tutorial-get-started.md)。

這份文件中的範例是針對 Bourne shell (bash) 和 PowerShell 提供。 bash 範例是以 GNU bash 4.3.11 版本進行測試，但是應該使用其他 Unix shell。 PowerShell 範例是以 PowerShell 5.0 進行測試，但是應該使用 PowerShell 3.0 或更新版本。

如果使用 __Bourne shell__ (Bash)，您必須安裝下列項目：

* [cURL](http://curl.haxx.se/)：cURL 是公用程式，可以用來從命令列使用 REST API。 在本文件中，它用來與 Ambari REST API 進行通訊。

無論是使用 Bash 或 PowerShell，您也必須安裝 [jq](https://stedolan.github.io/jq/)。 Jq 是用來使用 JSON 文件的公用程式。 它用在**所有** Bash 範例中，以及**其中一個** PowerShell 範例中。

### <a name="base-uri-for-ambari-rest-api"></a>Ambari REST API 的基底 URI

Ambari REST API 在 HDInsight 上的基底 URI 是 https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME，其中 **CLUSTERNAME** 是您叢集的名稱。

> [!IMPORTANT]
> URI (CLUSTERNAME.azurehdinsight.net) 完整網域名稱 (FQDN) 部分中的叢集名稱區分大小寫，URI 中的其他部分也區分大小寫。 例如，如果您的叢集名稱為 `MyCluster`，有效的 URI 如下：
> 
> `https://mycluster.azurehdinsight.net/api/v1/clusters/MyCluster`
>
> `https://MyCluster.azurehdinsight.net/api/v1/clusters/MyCluster`
> 
> 下列 URI 會傳回錯誤，因為名稱的第二個項目不是正確的大小寫。
> 
> `https://mycluster.azurehdinsight.net/api/v1/clusters/mycluster`
>
> `https://MyCluster.azurehdinsight.net/api/v1/clusters/mycluster`

### <a name="authentication"></a>驗證

連線到 HDInsight 上的 Ambari 需要 HTTPS。 請使用您在叢集建立期間所提供的管理帳戶名稱 (預設值是 **admin**) 和密碼。

## <a name="examples-authentication-and-parsing-json"></a>範例︰驗證和剖析 JSON

下列範例示範如何針對基底 Ambari REST API 進行 GET 要求︰

```bash
curl -u admin -sS -G "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME"
```

> [!IMPORTANT]
> 這份文件中的 Bash 範例進行下列假設︰
>
> * 叢集的登入名稱是 `admin` 的預設值。
> * `$CLUSTERNAME` 包含叢集的名稱。 您可以藉由使用 `set CLUSTERNAME='clustername'` 來設定此值
> * 出現提示時，輸入叢集登入 (admin) 的密碼。

```powershell
$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName" `
    -Credential $creds
$resp.Content
```

> [!IMPORTANT]
> 這份文件中的 PowerShell 範例進行下列假設︰
>
> * `$creds` 是認證物件，包含叢集的系統管理員登入和密碼。 您可以藉由使用 `$creds = Get-Credential -UserName "admin" -Message "Enter the HDInsight login"` 來設定此值，並且在系統提示時提供認證。
> * `$clusterName` 是字串，包含叢集的名稱。 您可以藉由使用 `$clusterName="clustername"` 來設定此值。

兩個範例都會傳回開頭資訊類似下列內容的 JSON 文件︰

```json
{
"href" : "http://10.0.0.10:8080/api/v1/clusters/CLUSTERNAME",
"Clusters" : {
    "cluster_id" : 2,
    "cluster_name" : "CLUSTERNAME",
    "health_report" : {
    "Host/stale_config" : 0,
    "Host/maintenance_state" : 0,
    "Host/host_state/HEALTHY" : 7,
    "Host/host_state/UNHEALTHY" : 0,
    "Host/host_state/HEARTBEAT_LOST" : 0,
    "Host/host_state/INIT" : 0,
    "Host/host_status/HEALTHY" : 7,
    "Host/host_status/UNHEALTHY" : 0,
    "Host/host_status/UNKNOWN" : 0,
    "Host/host_status/ALERT" : 0
    ...
```

### <a name="parsing-json-data"></a>剖析 JSON 資料

下列範例會使用 `jq` 來剖析 JSON 回應文件，並且只顯示結果的 `health_report` 資訊。

```bash
curl -u admin -sS -G "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME" \
| jq '.Clusters.health_report'
```

PowerShell 3.0 版和更新版本提供 `ConvertFrom-Json` Cmdlet，將 JSON 文件轉換成更容易從 PowerShell 使用的物件。 下列範例會使用 `ConvertFrom-Json`，僅顯示結果的 `health_report` 資訊。

```powershell
$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName" `
    -Credential $creds
$respObj = ConvertFrom-Json $resp.Content
$respObj.Clusters.health_report
```

> [!NOTE]
> 雖然此文件的大多數範例使用 `ConvertFrom-Json` 來顯示回應文件的元素，而[更新 Ambari 組態](#example-update-ambari-configuration)範例會使用 jq。 此範例使用 jq，從 JSON 回應文件建構新的範本。

如需 REST API 的完整參考，請參閱 [Ambari API 參考 V1](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md)。

## <a name="example-get-the-fqdn-of-cluster-nodes"></a>範例：取得叢集節點的 FQDN

使用 HDInsight 時，您可能需要知道叢集節點的完整網域名稱 (FQDN)。 您可以使用下列範例，輕鬆地擷取叢集中不同節點的 FQDN：

* **所有節點**

    ```bash
    curl -u admin -sS -G "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/hosts" \
    | jq '.items[].Hosts.host_name'
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/hosts" `
        -Credential $creds
    $respObj = ConvertFrom-Json $resp.Content
    $respObj.items.Hosts.host_name
    ```

* **前端節點**：

    ```bash
    curl -u admin -sS -G "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/HDFS/components/NAMENODE" \
    | jq '.host_components[].HostRoles.host_name'
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/HDFS/components/NAMENODE" `
        -Credential $creds
    $respObj = ConvertFrom-Json $resp.Content
    $respObj.host_components.HostRoles.host_name
    ```

* **背景工作節點**

    ```bash
    curl -u admin -sS -G "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/HDFS/components/DATANODE" \
    | jq '.host_components[].HostRoles.host_name'
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/HDFS/components/DATANODE" `
        -Credential $creds
    $respObj = ConvertFrom-Json $resp.Content
    $respObj.host_components.HostRoles.host_name
    ```

* **Zookeeper 節點**

    ```bash
    curl -u admin -sS -G "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/ZOOKEEPER/components/ZOOKEEPER_SERVER" \
    | jq '.host_components[].HostRoles.host_name'
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/ZOOKEEPER/components/ZOOKEEPER_SERVER" `
        -Credential $creds
    $respObj = ConvertFrom-Json $resp.Content
    $respObj.host_components.HostRoles.host_name
    ```

## <a name="example-get-the-internal-ip-address-of-cluster-nodes"></a>範例︰取得叢集節點的內部 IP 位址

> [!IMPORTANT]
> 本章節中的範例所傳回的 IP 位址無法直接透過網際網路存取。 它們只能在包含 HDInsight 叢集的 Azure 虛擬網路內存取。
>
> 如需使用 HDInsight 和虛擬網路的詳細資訊，請參閱[使用自訂 Azure 虛擬網路擴充 HDInsight 功能](hdinsight-extend-hadoop-virtual-network.md)。

若要尋找 IP 位址，您必須知道叢集節點的內部完整網域名稱 (FQDN)。 一旦您擁有 FQDN，就可以取得主機的 IP 位址。 下列範例會先查詢所有主機節點之 FQDN 的 Ambari，然後查詢每部主機之 IP 位址的 Ambari。

```bash
for HOSTNAME in $(curl -u admin:$PASSWORD -sS -G "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/hosts" | jq -r '.items[].Hosts.host_name')
do
    IP=$(curl -u admin:$PASSWORD -sS -G "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/hosts/$HOSTNAME" | jq -r '.Hosts.ip')
  echo "$HOSTNAME <--> $IP"
done
```

> [!TIP]
> 先前的範例會提示您輸入密碼。 這個範例會執行 `curl` 命令多次，因此密碼提供為 `$PASSWORD` 以避免多次提示。

```powershell
$uri = "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/hosts"
$resp = Invoke-WebRequest -Uri $uri -Credential $creds
$respObj = ConvertFrom-Json $resp.Content
foreach($item in $respObj.items) {
    $hostName = [string]$item.Hosts.host_name
    $hostInfoResp = Invoke-WebRequest -Uri "$uri/$hostName" `
        -Credential $creds
    $hostInfoObj = ConvertFrom-Json $hostInfoResp 
    $hostIp = $hostInfoObj.Hosts.ip
    "$hostName <--> $hostIp"
}
```

## <a name="example-get-the-default-storage"></a>範例：取得預設儲存體

建立 HDInsight 叢集時，您必須使用 Azure 儲存體帳戶或 Data Lake Store 做為叢集的預設儲存體。 在建立叢集之後，您可以使用 Ambari 來擷取這項資訊。 例如，如果您想要讀取/寫入資料至 HDInsight 以外的容器。

下列範例會從叢集擷取預設儲存體組態：

```bash
curl -u admin -sS -G "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/configurations/service_config_versions?service_name=HDFS&service_config_version=1" \
| jq '.items[].configurations[].properties["fs.defaultFS"] | select(. != null)'
```

```powershell
$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations/service_config_versions?service_name=HDFS&service_config_version=1" `
    -Credential $creds
$respObj = ConvertFrom-Json $resp.Content
$respObj.items.configurations.properties.'fs.defaultFS'
```

> [!IMPORTANT]
> 這些範例會傳回套用至伺服器的第一個組態 (`service_config_version=1`)，其包含這項資訊。 如果您擷取在叢集建立後已修改過的值，您可能需要列出組態版本並擷取最新的版本。

傳回值會類似下列其中一個範例︰

* `wasb://CONTAINER@ACCOUNTNAME.blob.core.windows.net` - 這個值表示叢集是使用 Azure 儲存體帳戶做為預設儲存體。 `ACCOUNTNAME` 值是儲存體帳戶的名稱。 `CONTAINER` 部分是儲存體帳戶中 blob 容器的名稱。 容器是叢集的 HDFS 相容儲存體的根目錄。

* `adl://home` - 這個值表示叢集是使用 Azure Data Lake Store 做為預設儲存體。

    若要尋找 Data Lake Store 帳戶名稱，請使用下列範例︰

    ```bash
    curl -u admin -sS -G "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/configurations/service_config_versions?service_name=HDFS&service_config_version=1" \
    | jq '.items[].configurations[].properties["dfs.adls.home.hostname"] | select(. != null)'
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations/service_config_versions?service_name=HDFS&service_config_version=1" `
        -Credential $creds
    $respObj = ConvertFrom-Json $resp.Content
    $respObj.items.configurations.properties.'dfs.adls.home.hostname'
    ```

    傳回值類似 `ACCOUNTNAME.azuredatalakestore.net`，其中 `ACCOUNTNAME` 是 Data Lake Store 帳戶的名稱。

    若要尋找包含叢集儲存體的 Data Lake Store 內的目錄，請使用下列範例︰

    ```bash
    curl -u admin -sS -G "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/configurations/service_config_versions?service_name=HDFS&service_config_version=1" \
    | jq '.items[].configurations[].properties["dfs.adls.home.mountpoint"] | select(. != null)'
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations/service_config_versions?service_name=HDFS&service_config_version=1" `
        -Credential $creds
    $respObj = ConvertFrom-Json $resp.Content
    $respObj.items.configurations.properties.'dfs.adls.home.mountpoint'
    ```

    傳回值類似 `/clusters/CLUSTERNAME/`。 這個值是 Data Lake Store 帳戶內的路徑。 這個路徑是叢集的 HDFS 相容檔案系統的根目錄。 

> [!NOTE]
> [Azure PowerShell](/powershell/azure/overview) 提供的 `Get-AzureRmHDInsightCluster` Cmdlet 也會傳回叢集的儲存體資訊。


## <a name="example-get-configuration"></a>範例：取得組態

1. 取得可供您的叢集使用的組態。

    ```bash
    curl -u admin -sS -G "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME?fields=Clusters/desired_configs"
    ```

    ```powershell
    $respObj = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName`?fields=Clusters/desired_configs" `
        -Credential $creds
    $respObj.Content
    ```

    此範例會傳回 JSON 文件，其中包含叢集上安裝之元件的目前組態 (由「tag」值識別)。 下列範例是從 Spark 叢集類型傳回之資料的摘要。
   
   ```json
   "spark-metrics-properties" : {
       "tag" : "INITIAL",
       "user" : "admin",
       "version" : 1
   },
   "spark-thrift-fairscheduler" : {
       "tag" : "INITIAL",
       "user" : "admin",
       "version" : 1
   },
   "spark-thrift-sparkconf" : {
       "tag" : "INITIAL",
       "user" : "admin",
       "version" : 1
   }
   ```

2. 取得您感興趣之元件的組態。 在下列範例中，以前一個要求傳回的標記值取代 `INITIAL`。

    ```bash
    curl -u admin -sS -G "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/configurations?type=core-site&tag=INITIAL"
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations?type=core-site&tag=INITIAL" `
        -Credential $creds
    $resp.Content
    ```

    此範例會傳回 JSON 文件，其中包含 `core-site` 元件的目前組態。

## <a name="example-update-configuration"></a>範例︰更新組態

1. 取得目前的組態，Ambari 會將其儲存為「所需的組態」:

    ```bash
    curl -u admin -sS -G "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME?fields=Clusters/desired_configs"
    ```

    ```powershell
    Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName`?fields=Clusters/desired_configs" `
        -Credential $creds
    ```

    此範例會傳回 JSON 文件，其中包含叢集上安裝之元件的目前組態 (由「tag」值識別)。 下列範例是從 Spark 叢集類型傳回之資料的摘要。
   
    ```json
    "spark-metrics-properties" : {
        "tag" : "INITIAL",
        "user" : "admin",
        "version" : 1
    },
    "spark-thrift-fairscheduler" : {
        "tag" : "INITIAL",
        "user" : "admin",
        "version" : 1
    },
    "spark-thrift-sparkconf" : {
        "tag" : "INITIAL",
        "user" : "admin",
        "version" : 1
    }
    ```
   
    您需要從此清單中複製元件的名稱 (例如，**spark\_thrift\_sparkconf** 和 **tag** 值。

2. 使用下列命令以擷取元件和標記的組態：
   
    ```bash
    curl -u admin -sS -G "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/configurations?type=spark-thrift-sparkconf&tag=INITIAL" \
    | jq --arg newtag $(echo version$(date +%s%N)) '.items[] | del(.href, .version, .Config) | .tag |= $newtag | {"Clusters": {"desired_config": .}}' > newconfig.json
    ```

    ```powershell
    $epoch = Get-Date -Year 1970 -Month 1 -Day 1 -Hour 0 -Minute 0 -Second 0
    $now = Get-Date
    $unixTimeStamp = [math]::truncate($now.ToUniversalTime().Subtract($epoch).TotalMilliSeconds)
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations?type=spark-thrift-sparkconf&tag=INITIAL" `
        -Credential $creds
    $resp.Content | jq --arg newtag "version$unixTimeStamp" '.items[] | del(.href, .version, .Config) | .tag |= $newtag | {"Clusters": {"desired_config": .}}' > newconfig.json
    ```

    > [!NOTE]
    > 將 **spark-thrift-sparkconf** 和 **INITIAL** 取代為您想要擷取其組態的元件和標籤。
   
    Jq 是用來將從 HDInsight 擷取到的資料轉換至新的組態範本。 具體來說，這些範例會執行下列動作︰
   
    * 建立唯一的值，其中包含字串 "version" 和日期，會儲存在 `newtag`。

    * 建立新的所需組態的根文件。

    * 取得 `.items[]` 陣列的內容，並且新增在 **desired_config** 元素下。

    * 刪除 `href`、`version` 和 `Config` 元素，因為提交新組態時不需要這些元素。

    * 使用 `version#################` 的值新增 `tag` 元素。 數字部分是根據目前的日期。 每個組態都必須有唯一的標記。
     
    最後，將資料儲存至 `newconfig.json` 文件。 此文件結構應會顯示為類似下列範例：
     
     ```json
    {
        "Clusters": {
            "desired_config": {
            "tag": "version1459260185774265400",
            "type": "spark-thrift-sparkconf",
            "properties": {
                ....
            },
            "properties_attributes": {
                ....
            }
        }
    }
    ```

3. 開啟 `newconfig.json` 文件，並且在 `properties` 物件中修改/新增值。 下列範例會將 `"spark.yarn.am.memory"`的值從 `"1g"`變更為 `"3g"`。 它也會使用 `"256m"` 的值新增 `"spark.kryoserializer.buffer.max"`。
   
        "spark.yarn.am.memory": "3g",
        "spark.kyroserializer.buffer.max": "256m",
   
    完成修改之後，請儲存檔案。

4. 使用以下命令將更新的組態提交至 Ambari。
   
    ```bash
    curl -u admin -sS -H "X-Requested-By: ambari" -X PUT -d @newconfig.json "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME"
    ```

    ```powershell
    $newConfig = Get-Content .\newconfig.json
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName" `
        -Credential $creds `
        -Method PUT `
        -Headers @{"X-Requested-By" = "ambari"} `
        -Body $newConfig
    $resp.Content
    ```
   
    這些命令會將 **newconfig.json** 檔案的內容提交至叢集，做為新的所需組態。 要求會傳回 JSON 文件。 這份文件中的 **versionTag** 元素應符合您所提交的版本，**configs** 物件將會包含您所要求的組態變更。

### <a name="example-restart-a-service-component"></a>範例︰重新啟動服務元件

此時，如果您看一下 Ambari Web UI，Spark 服務就會指出它需要重新啟動，新組態才會生效。 使用下列步驟重新啟動服務。

1. 使用以下命令以啟用 Spark 服務的維護模式：

    ```bash
    curl -u admin -sS -H "X-Requested-By: ambari" \
    -X PUT -d '{"RequestInfo": {"context": "turning on maintenance mode for SPARK"},"Body": {"ServiceInfo": {"maintenance_state":"ON"}}}' \
    "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/SPARK"
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK" `
        -Credential $creds `
        -Method PUT `
        -Headers @{"X-Requested-By" = "ambari"} `
        -Body '{"RequestInfo": {"context": "turning on maintenance mode for SPARK"},"Body": {"ServiceInfo": {"maintenance_state":"ON"}}}'
    $resp.Content
    ```
   
    這些命令會將 JSON 文件傳送至伺服器，該伺服器以維護模式開啟。 您可以使用下列要求驗證服務現在處於維護模式中：
   
    ```bash
    curl -u admin -sS -H "X-Requested-By: ambari" \
    "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/SPARK" \
    | jq .ServiceInfo.maintenance_state
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK2" `
        -Credential $creds
    $respObj = ConvertFrom-Json $resp.Content
    $respObj.ServiceInfo.maintenance_state
    ```
   
    傳回值是 `ON`。

2. 接下來，使用下列命令來關閉服務：

    ```bash
    curl -u admin -sS -H "X-Requested-By: ambari" \
    -X PUT -d '{"RequestInfo":{"context":"_PARSE_.STOP.SPARK","operation_level":{"level":"SERVICE","cluster_name":"CLUSTERNAME","service_name":"SPARK"}},"Body":{"ServiceInfo":{"state":"INSTALLED"}}}' \
    "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/SPARK"
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK" `
        -Credential $creds `
        -Method PUT `
        -Headers @{"X-Requested-By" = "ambari"} `
        -Body '{"RequestInfo":{"context":"_PARSE_.STOP.SPARK","operation_level":{"level":"SERVICE","cluster_name":"CLUSTERNAME","service_name":"SPARK"}},"Body":{"ServiceInfo":{"state":"INSTALLED"}}}'
    $resp.Content
    ```
    
    回應如下列範例所示：
   
    ```json
    {
        "href" : "http://10.0.0.18:8080/api/v1/clusters/CLUSTERNAME/requests/29",
        "Requests" : {
            "id" : 29,
            "status" : "Accepted"
        }
    }
    ```
    
    > [!IMPORTANT]
    > 這個 URI 所傳回的 `href` 值會使用叢集節點的內部 IP 位址。 若要從叢集之外使用它，請將 '10.0.0.18:8080' 部分取代為叢集的 FQDN。 
    
    下列命令會擷取要求的狀態：

    ```bash
    curl -u admin -sS -H "X-Requested-By: ambari" \
    "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/requests/29" \
    | jq .Requests.request_status
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/requests/29" `
        -Credential $creds
    $respObj = ConvertFrom-Json $resp.Content
    $respObj.Requests.request_status
    ```

    `COMPLETED` 的回應表示要求已完成。

3. 前一個要求完成後，使用以下命令來啟動服務。
   
    ```bash
    curl -u admin -sS -H "X-Requested-By: ambari" \
    -X PUT -d '{"RequestInfo":{"context":"_PARSE_.STOP.SPARK","operation_level":{"level":"SERVICE","cluster_name":"CLUSTERNAME","service_name":"SPARK"}},"Body":{"ServiceInfo":{"state":"STARTED"}}}' \
    "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/SPARK"
    ```
   
    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK" `
        -Credential $creds `
        -Method PUT `
        -Headers @{"X-Requested-By" = "ambari"} `
        -Body '{"RequestInfo":{"context":"_PARSE_.STOP.SPARK","operation_level":{"level":"SERVICE","cluster_name":"CLUSTERNAME","service_name":"SPARK"}},"Body":{"ServiceInfo":{"state":"STARTED"}}}'
    ```
    服務現在使用新的組態。

4. 最後，使用以下命令來關閉維護模式。
   
    ```bash
    curl -u admin -sS -H "X-Requested-By: ambari" \
    -X PUT -d '{"RequestInfo": {"context": "turning off maintenance mode for SPARK"},"Body": {"ServiceInfo": {"maintenance_state":"OFF"}}}' \
    "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/SPARK"
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK" `
        -Credential $creds `
        -Method PUT `
        -Headers @{"X-Requested-By" = "ambari"} `
        -Body '{"RequestInfo": {"context": "turning off maintenance mode for SPARK"},"Body": {"ServiceInfo": {"maintenance_state":"OFF"}}}'
    ```

## <a name="next-steps"></a>後續步驟

如需 REST API 的完整參考，請參閱 [Ambari API 參考 V1](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md)。

