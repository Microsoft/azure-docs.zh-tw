---
title: 將其他 Azure 儲存體帳戶新增至 HDInsight
description: 了解如何將其他 Azure 儲存體帳戶新增至現有的 HDInsight 叢集。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 04/23/2018
ms.author: jasonh
ms.custom: H1Hack27Feb2017,hdinsightactive
ms.openlocfilehash: bdf4c7527e2c0056b2de2321fbae280e9008cb1b
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46990666"
---
# <a name="add-additional-storage-accounts-to-hdinsight"></a>將其他儲存體帳戶新增至 HDInsight

了解如何使用指令碼動作，將其他 Azure 儲存體帳戶新增至 HDInsight。 這份文件中的步驟會在現有的 Linux 架構 HDInsight 叢集中新增儲存體帳戶。

> [!IMPORTANT]
> 本文件中的資訊是有關如何在建立叢集之後，將其他儲存體新增至叢集。 如需在叢集建立期間新增儲存體帳戶的資訊，請參閱[使用 Hadoop、Spark、Kafka 等在 HDInsight 中設定叢集](hdinsight-hadoop-provision-linux-clusters.md)。

## <a name="how-it-works"></a>運作方式

此指令碼採用下列參數︰

* __Azure 儲存體帳戶名稱__：要新增至 HDInsight 叢集的儲存體帳戶名稱。 執行指令碼之後，HDInsight 可以讀取和寫入此儲存體帳戶中儲存的資料。

* __Azure 儲存體帳戶金鑰__︰授與存取權給儲存體帳戶的金鑰。

* __-p__ (選用)︰若已指定，金鑰不會加密，而且會以純文字方式儲存在 core-site.xml 檔案。

在處理期間，此指令碼會執行下列動作：

* 如果儲存體帳戶已經存在於叢集的 core-site.xml 組態中，則指令碼會結束，不會執行任何進一步的動作。

* 確認儲存體帳戶存在並可使用金鑰來存取。

* 使用叢集認證加密金鑰。

* 將儲存體帳戶新增至 core-site.xml 檔案。

* 停止並重新啟動 Oozie、YARN、MapReduce2 和 HDFS 服務。 停止並啟動這些服務可讓它們使用新的儲存體帳戶。

> [!WARNING]
> 不支援在與 HDInsight 叢集不同的位置中使用儲存體帳戶。

## <a name="the-script"></a>指令碼

__指令碼位置__：[https://hdiconfigactions.blob.core.windows.net/linuxaddstorageaccountv01/add-storage-account-v01.sh](https://hdiconfigactions.blob.core.windows.net/linuxaddstorageaccountv01/add-storage-account-v01.sh)

__需求__：

* 指令碼必須套用於__前端節點__。

## <a name="to-use-the-script"></a>若要使用指令碼

您可以從 Azure 入口網站、Azure PowerShell 或 Azure 傳統 CLI 使用此指令碼。 如需詳細資訊，請參閱[使用指令碼動作自訂 Linux 型 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md#apply-a-script-action-to-a-running-cluster)文件。

> [!IMPORTANT]
> 在使用自訂文件中提供的步驟時，請使用下列資訊來套用此指令碼：
>
> * 將所有的範例指令碼動作 URI 取代為此指令碼的 URI (https://hdiconfigactions.blob.core.windows.net/linuxaddstorageaccountv01/add-storage-account-v01.sh)。
> * 以即將新增至叢集的儲存體帳戶的 Azure 儲存體帳戶名稱和金鑰取代範例參數。 如果使用 Azure 入口網站，則必須以空格分隔這些參數。
> * 您不需要將此指令碼標示為 [已保存]，因為它會直接更新叢集的 Ambari 組態。

## <a name="known-issues"></a>已知問題

### <a name="storage-accounts-not-displayed-in-azure-portal-or-tools"></a>儲存體帳戶未顯示在 Azure 入口網站或工具中

在 Azure 入口網站中檢視 HDInsight 叢集時，選取 [屬性] 之下的 [儲存體帳戶] 項目，並不會顯示透過此指令碼動作新增的儲存體帳戶。 Azure PowerShell 與 Azure 傳統 CLI 也不會顯示其他儲存體帳戶。

沒有顯示儲存體資訊是因為指令碼只修改叢集的 core-site.xml 組態。 使用 Azure 管理 API 擷取叢集資訊時，不會使用這項資訊。

若要檢視使用此指令碼新增至叢集的儲存體帳戶資訊，請使用 Ambari REST API。 請使用下列命令，為您的叢集擷取此資訊：

```PowerShell
$creds = Get-Credential -UserName "admin" -Message "Enter the cluster login credentials"
$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations/service_config_versions?service_name=HDFS&service_config_version=1" `
    -Credential $creds
$respObj = ConvertFrom-Json $resp.Content
$respObj.items.configurations.properties."fs.azure.account.key.$storageAccountName.blob.core.windows.net"
```

> [!NOTE]
> 將 `$clusterName` 設定為 HDInsight 叢集的名稱。 將 `$storageAccountName` 設定為儲存體帳戶的名稱。 出現提示時，輸入叢集登入 (admin) 和密碼。

```Bash
curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/configurations/service_config_versions?service_name=HDFS&service_config_version=1" | jq '.items[].configurations[].properties["fs.azure.account.key.$STORAGEACCOUNTNAME.blob.core.windows.net"] | select(. != null)'
```

> [!NOTE]
> 將 `$PASSWORD` 設定為叢集登入 (admin) 帳戶密碼。 將 `$CLUSTERNAME` 設定為 HDInsight 叢集的名稱。 將 `$STORAGEACCOUNTNAME` 設定為儲存體帳戶的名稱。
>
> 此範例會使用 [curl (http://curl.haxx.se/)](http://curl.haxx.se/) 和 [jq (https://stedolan.github.io/jq/)](https://stedolan.github.io/jq/) 來擷取及剖析 JSON 資料。

使用此命令時，將 __CLUSTERNAME__ 替換為 HDInsight 叢集的名稱。 將 __PASSWORD__ 替換為叢集的 HTTP 登入密碼。 將 __STORAGEACCOUNT__ 替換為使用指令碼動作新增的儲存體帳戶名稱。 此命令傳回的資訊看起來類似下列文字：

    "MIIB+gYJKoZIhvcNAQcDoIIB6zCCAecCAQAxggFaMIIBVgIBADA+MCoxKDAmBgNVBAMTH2RiZW5jcnlwdGlvbi5henVyZWhkaW5zaWdodC5uZXQCEA6GDZMW1oiESKFHFOOEgjcwDQYJKoZIhvcNAQEBBQAEggEATIuO8MJ45KEQAYBQld7WaRkJOWqaCLwFub9zNpscrquA2f3o0emy9Vr6vu5cD3GTt7PmaAF0pvssbKVMf/Z8yRpHmeezSco2y7e9Qd7xJKRLYtRHm80fsjiBHSW9CYkQwxHaOqdR7DBhZyhnj+DHhODsIO2FGM8MxWk4fgBRVO6CZ5eTmZ6KVR8wYbFLi8YZXb7GkUEeSn2PsjrKGiQjtpXw1RAyanCagr5vlg8CicZg1HuhCHWf/RYFWM3EBbVz+uFZPR3BqTgbvBhWYXRJaISwssvxotppe0ikevnEgaBYrflB2P+PVrwPTZ7f36HQcn4ifY1WRJQ4qRaUxdYEfzCBgwYJKoZIhvcNAQcBMBQGCCqGSIb3DQMHBAhRdscgRV3wmYBg3j/T1aEnO3wLWCRpgZa16MWqmfQPuansKHjLwbZjTpeirqUAQpZVyXdK/w4gKlK+t1heNsNo1Wwqu+Y47bSAX1k9Ud7+Ed2oETDI7724IJ213YeGxvu4Ngcf2eHW+FRK"

此文字是已加密的金鑰範例，可用來存取儲存體帳戶。

### <a name="unable-to-access-storage-after-changing-key"></a>無法在變更金鑰之後存取儲存體

如果您變更儲存體帳戶的金鑰，HDInsight 就無法再存取儲存體帳戶。 HDInsight 在叢集的 core-site.xml 中使用金鑰的快取複本。 此快取副本必須更新以符合新的金鑰。

再次執行指令碼動作並__不會__更新金鑰，因為指令碼會查看儲存體帳戶的項目是否已經存在。 如果項目已經存在，就不會進行任何變更。

若要解決這個問題，您必須移除儲存體帳戶的現有項目。 執行下列步驟以移除現有項目：

1. 在 Web 瀏覽器中，對您的 HDInsight 叢集開啟 Ambari Web UI。 URI 為 https://CLUSTERNAME.azurehdinsight.net。 將 __CLUSTERNAME__ 取代為您叢集的名稱。

    出現提示時，輸入您叢集的 HTTP 登入使用者和密碼。

2. 從頁面左邊的服務清單中，選取 [HDFS]。 然後選取頁面中間的 [設定] 索引標籤。

3. 在 [篩選...] 欄位中，輸入 __fs.azure.account__ 值。 這會傳回已新增到叢集的任何其他儲存體帳戶項目。 有兩種項目類型：__keyprovider__ 和 __key__。 兩者都會包含儲存體帳戶名稱做為金鑰名稱的一部分。

    下列是名為 __mystorage__ 之儲存體帳戶的項目範例：

        fs.azure.account.keyprovider.mystorage.blob.core.windows.net
        fs.azure.account.key.mystorage.blob.core.windows.net

4. 在您識別您需要移除之儲存體帳戶的金鑰之後，請使用項目右邊的紅色 '-' 圖示將它刪除。 然後使用 [儲存] 按鈕來儲存您的變更。

5. 在儲存變更之後，使用指令碼動作將儲存體帳戶和新的金鑰值新增至叢集。

### <a name="poor-performance"></a>效能不佳

如果儲存體帳戶位於與 HDInsight 叢集不同的區域中，您可能會遇到效能不佳。 存取不同區域中的資料會傳送 Azure 資料中心外部和跨越公用網際網路的網路流量，這可能會造成延遲。

> [!WARNING]
> 不支援在 HDInsight 叢集以外的區域中使用儲存體帳戶。

### <a name="additional-charges"></a>額外費用

如果儲存體帳戶位於與 HDInsight 叢集不同的區域中，您可能會發現 Azure 帳單上出現額外輸出費用。 當資料離開區域資料中心時，就會產生輸出費用。 即使流量的目的地是不同區域中的其他 Azure 資料中心，仍會產生此費用。

> [!WARNING]
> 不支援在 HDInsight 叢集以外的區域中使用儲存體帳戶。

## <a name="next-steps"></a>後續步驟

您已了解如何將其他儲存體帳戶新增至現有的 HDInsight 叢集。 如需指令碼動作的詳細資訊，請參閱[使用指令碼動作自訂以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)
