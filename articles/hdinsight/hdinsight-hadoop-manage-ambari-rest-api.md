---
title: 使用 Ambari REST API 監視和管理 Hadoop - Azure HDInsight
description: 了解如何使用 Ambari 來監視和管理 Azure HDInsight 中的 Hadoop 叢集。 在本檔中，您將瞭解如何使用 HDInsight 叢集隨附的 Ambari REST API。
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 04/29/2020
ms.openlocfilehash: 1d4e6f0d6a0242cda919364965a61e4314927d87
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98945583"
---
# <a name="manage-hdinsight-clusters-by-using-the-apache-ambari-rest-api"></a>使用 Apache Ambari REST API 來管理 HDInsight 叢集

[!INCLUDE [ambari-selector](../../includes/hdinsight-ambari-selector.md)]

了解如何使用 Apache Ambari REST API 來管理和監視 Azure HDInsight 中的 Apache Hadoop 叢集。

## <a name="what-is-apache-ambari"></a>什麼是 Apache Ambari

Apache Ambari 藉由提供容易使用的 web UI （其 [REST api](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md)支援），來簡化 Hadoop 叢集的管理和監視。  以 Linux 為基礎的 HDInsight 叢集預設會提供 Ambari。

## <a name="prerequisites"></a>先決條件

* HDInsight 上的 Hadoop 叢集。 請參閱[開始在 Linux 上使用 HDInsight](hadoop/apache-hadoop-linux-tutorial-get-started.md)。

* Windows 10 上 Ubuntu 的 Bash。  本文中的命令列範例會在 Windows 10 上使用 Bash 殼層。 如需安裝步驟，請參閱 [Windows 10 適用於 Linux 的 Windows 子系統的安裝指南](/windows/wsl/install-win10)。  其他 [Unix 殼層](https://www.gnu.org/software/bash/)也可正常運作。  您可以在 Windows 命令提示字元中使用一些稍微修改的範例。  或者，您可以使用 Windows PowerShell。

* jq，這是命令列 JSON 處理器。  請參閱 [https://stedolan.github.io/jq/](https://stedolan.github.io/jq/)。

* Windows PowerShell  或者，您也可以使用 Bash。

## <a name="base-uniform-resource-identifier-for-ambari-rest-api"></a>Ambari Rest API 的基底統一資源識別項

 Ambari REST API on HDInsight 的基底統一資源識別項 (URI) 是 `https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME` ，其中 `CLUSTERNAME` 是您的叢集名稱。  Uri 中的叢集名稱會區分 **大小寫**。  雖然完整功能變數名稱中的叢集名稱 (FQDN) 一部分的 URI (`CLUSTERNAME.azurehdinsight.net`) 不區分大小寫，但 uri 中的其他出現專案會區分大小寫。

## <a name="authentication"></a>驗證

連線到 HDInsight 上的 Ambari 需要 HTTPS。 請使用您在叢集建立期間所提供的管理帳戶名稱 (預設值是 **admin**) 和密碼。

針對企業安全性套件叢集，而不是使用完整的使用者名稱（例如 `admin` ） `username@domain.onmicrosoft.com` 。

## <a name="examples"></a>範例

### <a name="setup-preserve-credentials"></a>設定 (保留認證) 

保留您的認證，以避免在每個範例中重新輸入。  叢集名稱將會在個別的步驟中保留。

**答： Bash**  
以您的實際密碼取代，以編輯以下腳本 `PASSWORD` 。  然後輸入命令。

```bash
export password='PASSWORD'
```  

**B. PowerShell**  

```powershell
$creds = Get-Credential -UserName "admin" -Message "Enter the HDInsight login"
```

### <a name="identify-correctly-cased-cluster-name"></a>識別正確的大小寫叢集名稱

叢集名稱的實際大小寫可能會與您預期的不同。  此處的步驟會顯示實際的大小寫，然後將它儲存在所有後續範例的變數中。

編輯下面的腳本，將取代為 `CLUSTERNAME` 您的叢集名稱。 然後輸入命令。  (FQDN 的叢集名稱不區分大小寫。 ) 

```bash
export clusterName=$(curl -u admin:$password -sS -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters" | jq -r '.items[].Clusters.cluster_name')
echo $clusterName
```  

```powershell
# Identify properly cased cluster name
$resp = Invoke-WebRequest -Uri "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters" `
    -Credential $creds -UseBasicParsing
$clusterName = (ConvertFrom-Json $resp.Content).items.Clusters.cluster_name;

# Show cluster name
$clusterName
```

### <a name="parsing-json-data"></a>剖析 JSON 資料

下列範例會使用 [jq](https://stedolan.github.io/jq/) 或 [convertfrom-string JSON](/powershell/module/microsoft.powershell.utility/convertfrom-json) 來剖析 Json 回應檔，並只顯示 `health_report` 結果中的資訊。

```bash
curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName" \
| jq '.Clusters.health_report'
```  

```powershell
$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName" `
    -Credential $creds -UseBasicParsing
$respObj = ConvertFrom-Json $resp.Content
$respObj.Clusters.health_report
```

### <a name="get-the-fqdn-of-cluster-nodes"></a>取得叢集節點的 FQDN

您可能需要知道叢集節點 (FQDN) 的完整功能變數名稱。 您可以使用下列範例，輕鬆地擷取叢集中不同節點的 FQDN：

**所有節點**  

```bash
curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/hosts" \
| jq -r '.items[].Hosts.host_name'
```  

```powershell
$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/hosts" `
    -Credential $creds -UseBasicParsing
$respObj = ConvertFrom-Json $resp.Content
$respObj.items.Hosts.host_name
```

**前端節點**  

```bash
curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/HDFS/components/NAMENODE" \
| jq -r '.host_components[].HostRoles.host_name'
```

```powershell
$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/HDFS/components/NAMENODE" `
    -Credential $creds -UseBasicParsing
$respObj = ConvertFrom-Json $resp.Content
$respObj.host_components.HostRoles.host_name
```

**背景工作節點**  

```bash
curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/HDFS/components/DATANODE" \
| jq -r '.host_components[].HostRoles.host_name'
```

```powershell
$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/HDFS/components/DATANODE" `
    -Credential $creds -UseBasicParsing
$respObj = ConvertFrom-Json $resp.Content
$respObj.host_components.HostRoles.host_name
```

**Zookeeper 節點**

```bash
curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/ZOOKEEPER/components/ZOOKEEPER_SERVER" \
| jq -r ".host_components[].HostRoles.host_name"
```

```powershell
$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/ZOOKEEPER/components/ZOOKEEPER_SERVER" `
    -Credential $creds -UseBasicParsing
$respObj = ConvertFrom-Json $resp.Content
$respObj.host_components.HostRoles.host_name
```

### <a name="get-the-internal-ip-address-of-cluster-nodes"></a>取得叢集節點的內部 IP 位址

本節中的範例所傳回的 IP 位址無法直接透過網際網路存取。 它們只能在包含 HDInsight 叢集的 Azure 虛擬網路內進行存取。

如需使用 HDInsight 和虛擬網路的詳細資訊，請參閱 [規劃 hdinsight 的虛擬網路](hdinsight-plan-virtual-network-deployment.md)。

若要尋找 IP 位址，您必須知道叢集節點的內部完整網域名稱 (FQDN)。 一旦您擁有 FQDN，就可以取得主機的 IP 位址。 下列範例會先查詢 Ambari，以取得所有主機節點的 FQDN。 然後查詢會 Ambari 每個主機的 IP 位址。

```bash
for HOSTNAME in $(curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/hosts" | jq -r '.items[].Hosts.host_name')
do
    IP=$(curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/hosts/$HOSTNAME" | jq -r '.Hosts.ip')
  echo "$HOSTNAME <--> $IP"
done
```  

```powershell
$uri = "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/hosts" 
$resp = Invoke-WebRequest -Uri $uri -Credential $creds -UseBasicParsing
$respObj = ConvertFrom-Json $resp.Content
foreach($item in $respObj.items) {
    $hostName = [string]$item.Hosts.host_name
    $hostInfoResp = Invoke-WebRequest -Uri "$uri/$hostName" `
        -Credential $creds -UseBasicParsing
    $hostInfoObj = ConvertFrom-Json $hostInfoResp
    $hostIp = $hostInfoObj.Hosts.ip
    "$hostName <--> $hostIp"
}
```

### <a name="get-the-default-storage"></a>取得預設儲存體

HDInsight 叢集必須使用 Azure 儲存體帳戶或 Data Lake Storage 作為預設儲存體。 在建立叢集之後，您可以使用 Ambari 來擷取這項資訊。 例如，如果您想要讀取/寫入資料至 HDInsight 以外的容器。

下列範例會從叢集擷取預設儲存體組態：

```bash
curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations/service_config_versions?service_name=HDFS&service_config_version=1" \
| jq -r '.items[].configurations[].properties["fs.defaultFS"] | select(. != null)'
```

```powershell
$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations/service_config_versions?service_name=HDFS&service_config_version=1" `
    -Credential $creds -UseBasicParsing
$respObj = ConvertFrom-Json $resp.Content
$respObj.items.configurations.properties.'fs.defaultFS'
```

> [!IMPORTANT]  
> 這些範例會傳回套用至伺服器的第一個組態 (`service_config_version=1`)，其包含這項資訊。 如果您擷取在叢集建立後已修改過的值，您可能需要列出組態版本並擷取最新的版本。

傳回值會類似下列其中一個範例︰

* `wasbs://CONTAINER@ACCOUNTNAME.blob.core.windows.net` - 這個值表示叢集是使用 Azure 儲存體帳戶做為預設儲存體。 `ACCOUNTNAME` 值是儲存體帳戶的名稱。 `CONTAINER` 部分是儲存體帳戶中 blob 容器的名稱。 容器是叢集的 HDFS 相容儲存體的根目錄。

* `abfs://CONTAINER@ACCOUNTNAME.dfs.core.windows.net` - 這個值表示叢集會使用 Azure Data Lake Storage Gen2 作為預設儲存體。 `ACCOUNTNAME` 和 `CONTAINER` 值與先前所述的 Azure 儲存體具有相同的意義。

* `adl://home` - 這個值表示叢集會使用 Azure Data Lake Storage Gen1 作為預設儲存體。

    若要尋找 Data Lake Storage 帳戶名稱，請使用下列範例：

    ```bash
    curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations/service_config_versions?service_name=HDFS&service_config_version=1" \
    | jq -r '.items[].configurations[].properties["dfs.adls.home.hostname"] | select(. != null)'
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations/service_config_versions?service_name=HDFS&service_config_version=1" `
        -Credential $creds -UseBasicParsing
    $respObj = ConvertFrom-Json $resp.Content
    $respObj.items.configurations.properties.'dfs.adls.home.hostname'
    ```

    傳回值類似 `ACCOUNTNAME.azuredatalakestore.net`，其中 `ACCOUNTNAME` 是 Data Lake Storage 帳戶的名稱。

    若要在包含叢集儲存體的 Data Lake Storage 內尋找此目錄，請使用下列範例：

    ```bash
    curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations/service_config_versions?service_name=HDFS&service_config_version=1" \
    | jq -r '.items[].configurations[].properties["dfs.adls.home.mountpoint"] | select(. != null)'
    ```  

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations/service_config_versions?service_name=HDFS&service_config_version=1" `
        -Credential $creds -UseBasicParsing
    $respObj = ConvertFrom-Json $resp.Content
    $respObj.items.configurations.properties.'dfs.adls.home.mountpoint'
    ```

    傳回值類似 `/clusters/CLUSTERNAME/`。 這個值是 Data Lake Storage 帳戶內的路徑。 這個路徑是叢集的 HDFS 相容檔案系統的根目錄。  

> [!NOTE]  
> [Azure PowerShell](/powershell/azure/)提供的[>new-azhdinsightcluster](/powershell/module/az.hdinsight/get-azhdinsightcluster)指令程式也會傳回叢集的儲存體資訊。

### <a name="get-all-configurations"></a>取得所有設定

取得可供您的叢集使用的組態。

```bash
curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName?fields=Clusters/desired_configs"
```

```powershell
$respObj = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName`?fields=Clusters/desired_configs" `
    -Credential $creds -UseBasicParsing
$respObj.Content
```

此範例會傳回 JSON 檔，其中包含已安裝元件的目前設定。 請參閱 *標記* 值。 下列範例是從 Spark 叢集類型傳回之資料的摘要。

```json
"jupyter-site" : {
  "tag" : "INITIAL",
  "version" : 1
},
"livy2-client-conf" : {
  "tag" : "INITIAL",
  "version" : 1
},
"livy2-conf" : {
  "tag" : "INITIAL",
  "version" : 1
},
```

### <a name="get-configuration-for-specific-component"></a>取得特定元件的設定

取得您想要的元件設定。 在下列範例中，以前一個要求傳回的標記值取代 `INITIAL`。

```bash
curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations?type=livy2-conf&tag=INITIAL"
```

```powershell
$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations?type=livy2-conf&tag=INITIAL" `
    -Credential $creds -UseBasicParsing
$resp.Content
```

此範例會傳回 JSON 文件，其中包含 `livy2-conf` 元件的目前組態。

### <a name="update-configuration"></a>更新設定

1. 建立 `newconfig.json`。  
   修改，然後輸入下列命令：

   * 取代 `livy2-conf` 為新的元件。
   * 取代 `INITIAL` 為 `tag` 從 [取得所有](#get-all-configurations)設定取出的實際值。

     **答： Bash**

     ```bash
     curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations?type=livy2-conf&tag=INITIAL" \
     | jq --arg newtag $(echo version$(date +%s%N)) '.items[] | del(.href, .version, .Config) | .tag |= $newtag | {"Clusters": {"desired_config": .}}' > newconfig.json
     ```

     **B. PowerShell**  
     PowerShell 腳本會使用 [jq](https://stedolan.github.io/jq/)。  編輯 `C:\HD\jq\jq-win64` 下方以反映實際的 [jq](https://stedolan.github.io/jq/)路徑和版本。

     ```powershell
     $epoch = Get-Date -Year 1970 -Month 1 -Day 1 -Hour 0 -Minute 0 -Second 0
     $now = Get-Date
     $unixTimeStamp = [math]::truncate($now.ToUniversalTime().Subtract($epoch).TotalMilliSeconds)
     $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations?type=livy2-conf&tag=INITIAL" `
       -Credential $creds -UseBasicParsing
     $resp.Content | C:\HD\jq\jq-win64 --arg newtag "version$unixTimeStamp" '.items[] | del(.href, .version, .Config) | .tag |= $newtag | {"Clusters": {"desired_config": .}}' > newconfig.json
     ```

     Jq 是用來將從 HDInsight 擷取到的資料轉換至新的組態範本。 具體來說，這些範例會執行下列動作：

   * 建立唯一的值，其中包含字串 "version" 和日期，會儲存在 `newtag`。

   * 建立新設定的根檔。

   * 取得 `.items[]` 陣列的內容，並且新增在 **desired_config** 元素下。

   * 刪除 `href`、`version` 和 `Config` 元素，因為提交新組態時不需要這些元素。

   * 使用 `version#################` 的值新增 `tag` 元素。 數字部分是根據目前的日期。 每個組態都必須有唯一的標記。

     最後，將資料儲存至 `newconfig.json` 文件。 此文件結構應會顯示為類似下列範例：

     ```json
     {
       "Clusters": {
         "desired_config": {
           "tag": "version1552064778014",
           "type": "livy2-conf",
           "properties": {
             "livy.environment": "production",
             "livy.impersonation.enabled": "true",
             "livy.repl.enableHiveContext": "true",
             "livy.server.csrf_protection.enabled": "true",
               ....
           },
         },
       }
     }
     ```

2. 編輯 `newconfig.json`。  
   開啟 `newconfig.json` 文件，並且在 `properties` 物件中修改/新增值。 下列範例會將 `"livy.server.csrf_protection.enabled"`的值從 `"true"`變更為 `"false"`。

    ```json
    "livy.server.csrf_protection.enabled": "false",
    ```

    完成修改之後，請儲存檔案。

3. 提交 `newconfig.json` 。  
   使用以下命令將更新的組態提交至 Ambari。

    ```bash
    curl -u admin:$password -sS -H "X-Requested-By: ambari" -X PUT -d @newconfig.json "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName"
    ```

    ```powershell
    $newConfig = Get-Content .\newconfig.json
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName" `
        -Credential $creds -UseBasicParsing `
        -Method PUT `
        -Headers @{"X-Requested-By" = "ambari"} `
        -Body $newConfig
    $resp.Content
    ```  

    這些命令會將檔案 **newconfig.js** 的內容提交至叢集做為新的設定。 要求會傳回 JSON 文件。 這份文件中的 **versionTag** 元素應符合您所提交的版本，**configs** 物件將會包含您所要求的組態變更。

### <a name="restart-a-service-component"></a>重新啟動服務元件

此時，Ambari web UI 會指出需要重新開機 Spark 服務，新的設定才會生效。 使用下列步驟重新啟動服務。

1. 使用下列程式來啟用 Spark2 服務的維護模式：

    ```bash
    curl -u admin:$password -sS -H "X-Requested-By: ambari" \
    -X PUT -d '{"RequestInfo": {"context": "turning on maintenance mode for SPARK2"},"Body": {"ServiceInfo": {"maintenance_state":"ON"}}}' \
    "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK2"
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK2" `
        -Credential $creds -UseBasicParsing `
        -Method PUT `
        -Headers @{"X-Requested-By" = "ambari"} `
        -Body '{"RequestInfo": {"context": "turning on maintenance mode for SPARK2"},"Body": {"ServiceInfo": {"maintenance_state":"ON"}}}'
    ```

2. 確認維護模式  

    這些命令會將 JSON 文件傳送至伺服器，該伺服器以維護模式開啟。 您可以使用下列要求驗證服務現在處於維護模式中：

    ```bash
    curl -u admin:$password -sS -H "X-Requested-By: ambari" \
    "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK2" \
    | jq .ServiceInfo.maintenance_state
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK2" `
        -Credential $creds -UseBasicParsing
    $respObj = ConvertFrom-Json $resp.Content
    $respObj.ServiceInfo.maintenance_state
    ```

    傳回值是 `ON`。

3. 接下來，使用下列內容來關閉 Spark2 服務：

    ```bash
    curl -u admin:$password -sS -H "X-Requested-By: ambari" \
    -X PUT -d '{"RequestInfo":{"context":"_PARSE_.STOP.SPARK2","operation_level":{"level":"SERVICE","cluster_name":"CLUSTERNAME","service_name":"SPARK"}},"Body":{"ServiceInfo":{"state":"INSTALLED"}}}' \
    "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK2"
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK2" `
        -Credential $creds -UseBasicParsing `
        -Method PUT `
        -Headers @{"X-Requested-By" = "ambari"} `
        -Body '{"RequestInfo":{"context":"_PARSE_.STOP.SPARK2","operation_level":{"level":"SERVICE","cluster_name":"CLUSTERNAME","service_name":"SPARK"}},"Body":{"ServiceInfo":{"state":"INSTALLED"}}}'
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
    > 這個 URI 所傳回的 `href` 值會使用叢集節點的內部 IP 位址。 若要從叢集外部使用它，請將部分取代為叢集 `10.0.0.18:8080` 的 FQDN。  

4. 確認要求。  
    以 `29` 先前步驟傳回的實際值取代，以編輯下列命令 `id` 。  下列命令會擷取要求的狀態：

    ```bash
    curl -u admin:$password -sS -H "X-Requested-By: ambari" \
    "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/requests/29" \
    | jq .Requests.request_status
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/requests/29" `
        -Credential $creds -UseBasicParsing
    $respObj = ConvertFrom-Json $resp.Content
    $respObj.Requests.request_status
    ```

    `COMPLETED` 的回應表示要求已完成。

5. 當先前的要求完成之後，請使用下列各項來啟動 Spark2 服務。

    ```bash
    curl -u admin:$password -sS -H "X-Requested-By: ambari" \
    -X PUT -d '{"RequestInfo":{"context":"_PARSE_.START.SPARK2","operation_level":{"level":"SERVICE","cluster_name":"CLUSTERNAME","service_name":"SPARK"}},"Body":{"ServiceInfo":{"state":"STARTED"}}}' \
    "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK2"
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK2" `
        -Credential $creds -UseBasicParsing `
        -Method PUT `
        -Headers @{"X-Requested-By" = "ambari"} `
        -Body '{"RequestInfo":{"context":"_PARSE_.START.SPARK2","operation_level":{"level":"SERVICE","cluster_name":"CLUSTERNAME","service_name":"SPARK"}},"Body":{"ServiceInfo":{"state":"STARTED"}}}'
    $resp.Content
    ```

    服務現在使用新的組態。

6. 最後，使用以下命令來關閉維護模式。

    ```bash
    curl -u admin:$password -sS -H "X-Requested-By: ambari" \
    -X PUT -d '{"RequestInfo": {"context": "turning off maintenance mode for SPARK2"},"Body": {"ServiceInfo": {"maintenance_state":"OFF"}}}' \
    "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK2"
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/SPARK2" `
        -Credential $creds -UseBasicParsing `
        -Method PUT `
        -Headers @{"X-Requested-By" = "ambari"} `
        -Body '{"RequestInfo": {"context": "turning off maintenance mode for SPARK2"},"Body": {"ServiceInfo": {"maintenance_state":"OFF"}}}'
    ```

## <a name="next-steps"></a>後續步驟

如需 REST API 的完整參考，請參閱 [Apache Ambari API 參考 V1](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md) \(英文\)。  另請參閱 [授權使用者進行 Apache Ambari Views](./hdinsight-authorize-users-to-ambari.md)