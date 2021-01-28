---
title: 在 Azure HDInsight 中使用 Data Lake Storage Gen1 搭配 Hadoop
description: 了解如何從 Azure Data Lake Storage Gen1 查詢資料，以及如何儲存分析的結果。
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017,seoapr2020
ms.date: 04/24/2020
ms.openlocfilehash: 35941f585a0ae5c0d3915c769db5b18737b299f0
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98945417"
---
# <a name="use-data-lake-storage-gen1-with-azure-hdinsight-clusters"></a>搭配 Azure HDInsight 叢集使用 Data Lake Storage Gen1

> [!Note]
> 針對已改善的效能和新功能，將新 HDInsight 叢集搭配 [Azure Data Lake Storage Gen2](hdinsight-hadoop-use-data-lake-storage-gen2.md) 使用。

若要分析 HDInsight 叢集中的資料，您可以將資料儲存在 [`Azure Blob storage`](../storage/common/storage-introduction.md) 、 [Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-overview.md)或 [Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md)中。 所有儲存體選項都可讓您安全地刪除用於計算的 HDInsight 叢集，而不會遺失使用者資料。

在本文中，您將了解 Data Lake Storage Gen1 與 HDInsight 叢集搭配運作的方式。 若要瞭解 Azure Blob 儲存體與 HDInsight 叢集搭配運作的方式，請參閱 [搭配 Azure HDInsight 叢集使用 Azure blob 儲存體](hdinsight-hadoop-use-blob-storage.md)。 如需建立 HDInsight 叢集的詳細資訊，請參閱[在 HDInsight 中建立 Apache Hadoop 叢集](hdinsight-hadoop-provision-linux-clusters.md)。

> [!NOTE]  
> Data Lake Storage Gen1 一律透過安全通道存取，因此不會有 `adls` 檔案系統配置名稱。 您會一律使用 `adl`。

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="availability-for-hdinsight-clusters"></a>HDInsight 叢集的可用性

Apache Hadoop 支援預設檔案系統的概念。 預設檔案系統意指預設配置和授權。 也可用來解析相對路徑。 在 HDInsight 叢集建立程式期間，將 Azure 儲存體中的 blob 容器指定為預設檔案系統。 或者，使用 HDInsight 3.5 和更新版本時，您可以選取 Azure Blob 儲存體或 Azure Data Lake Storage Gen1 作為預設的檔案系統，但有一些例外狀況。 叢集與儲存體帳戶必須在相同區域內託管。

HDInsight 叢集可透過兩種方式來使用 Data Lake Storage Gen1︰

* 作為預設儲存體
* 作為額外儲存體，以 Azure Blob 儲存體作為預設儲存體。

目前，只有一些 HDInsight 叢集類型/版本支援使用 Data Lake Storage Gen1 作為預設儲存體和其他儲存體帳戶：

| HDInsight 叢集類型 | 使用 Data Lake Storage Gen1 作為預設儲存體 | 使用 Data Lake Storage Gen1 作為其他儲存體| 備註 |
|------------------------|------------------------------------|---------------------------------------|------|
| HDInsight 4.0 版 | 否 | 否 |HDInsight 4.0 不支援 ADLS Gen1 |
| HDInsight 3.6 版 | 是 | 是 | HBase 以外的|
| HDInsight 3.5 版 | 是 | 是 | HBase 以外的|
| HDInsight 3.4 版 | 否 | 是 | |
| HDInsight 3.3 版 | 否 | 否 | |
| HDInsight 3.2 版 | 否 | 是 | |
| Storm | | |您可以使用 Data Lake Storage Gen1 從 Storm 拓撲寫入資料。 您也可以使用 Data Lake Storage Gen1 作為參考資料，然後再由「風暴」拓撲讀取。|

> [!WARNING]  
> Azure Data Lake Storage Gen1 不支援 HDInsight HBase

使用 Data Lake Storage Gen1 作為額外的儲存體帳戶不會影響效能。 或從叢集中讀取或寫入 Azure blob 儲存體的功能。

## <a name="use-data-lake-storage-gen1-as-default-storage"></a>使用 Data Lake Storage Gen1 作為預設儲存體

當使用 Data Lake Storage Gen1 作為預設儲存體部署 HDInsight 時，與叢集相關的檔案會儲存在 `adl://mydatalakestore/<cluster_root_path>/` 中，其中 `<cluster_root_path>` 是您在 Data Lake Storage 中建立的資料夾名稱。 藉由指定每個叢集的根路徑，您可針對多個叢集使用相同的 Data Lake Storage 帳戶。 因此在您的設定中︰

* Cluster1 可以使用路徑 `adl://mydatalakestore/cluster1storage`
* Cluster2 可以使用路徑 `adl://mydatalakestore/cluster2storage`

請注意，這兩個叢集都使用相同的 Data Lake Storage Gen1 帳戶 **mydatalakestore**。 每個叢集都會在 Data Lake Storage 中存取自己的根檔案系統。 Azure 入口網站部署體驗會提示您針對根路徑使用資料夾名稱（**例如 \<clustername> /clusters/** ）。

若要使用 Data Lake Storage Gen1 作為預設儲存體，您必須授與服務主體存取下列路徑：

* 刪除 Data Lake Storage Gen1 帳戶根目錄。  例如：adl://mydatalakestore/。
* 所有叢集資料夾的資料夾。  例如：adl://mydatalakestore/clusters。
* 叢集的資料夾。  例如：adl://mydatalakestore/clusters/cluster1storage。

如需建立服務主體和授與存取權的詳細資訊，請參閱設定 Data Lake Storage 存取。

### <a name="extracting-a-certificate-from-azure-keyvault-for-use-in-cluster-creation"></a>從 Azure Key Vault 擷取憑證以用於建立叢集

如果您服務主體的憑證儲存在 Azure Key Vault 中，您必須將憑證轉換成正確的格式。 下列程式碼片段示範如何進行轉換。

首先，從 Key Vault 下載憑證，並擷取 `SecretValueText`。

```powershell
$certPassword = Read-Host "Enter Certificate Password"
$cert = (Get-AzureKeyVaultSecret -VaultName 'MY-KEY-VAULT' -Name 'MY-SECRET-NAME')
$certValue = [System.Convert]::FromBase64String($cert.SecretValueText)
```

接下來，將 `SecretValueText` 轉換為憑證。

```powershell
$certObject = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2 -ArgumentList $certValue,$null,"Exportable, PersistKeySet"
$certBytes = $certObject.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pkcs12, $certPassword.SecretValueText);
$identityCertificate = [System.Convert]::ToBase64String($certBytes)
```

然後，您可以使用 `$identityCertificate` 來部署新的叢集，如下列程式碼片段所示：

```powershell
New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile $pathToArmTemplate `
    -identityCertificate $identityCertificate `
    -identityCertificatePassword $certPassword.SecretValueText `
    -clusterName  $clusterName `
    -clusterLoginPassword $SSHpassword `
    -sshPassword $SSHpassword `
    -servicePrincipalApplicationId $application.ApplicationId
```

## <a name="use-data-lake-storage-gen1-as-additional-storage"></a>使用 Data Lake Storage Gen1 作為其他儲存體

您也可以使用 Data Lake Storage Gen1 作為叢集的其他儲存體。 在這種情況下，叢集預設儲存體可以是 Azure Blob 儲存體或 Azure Data Lake Storage Gen1 帳戶。 針對 Azure Data Lake Storage Gen1 中儲存的資料執行 HDInsight 作業作為額外儲存體時，請使用完整路徑。 例如：

`adl://mydatalakestore.azuredatalakestore.net/<file_path>`

現在 URL 中沒有 **cluster_root_path** 。 這是因為在此情況下，Data Lake Storage 不是預設儲存體。 您只需要提供檔案的路徑。

若要使用 Data Lake Storage Gen1 作為額外的儲存體，請將服務主體存取權授與儲存檔案的路徑。  例如：

`adl://mydatalakestore.azuredatalakestore.net/<file_path>`

如需建立服務主體和授與存取權的詳細資訊，請參閱設定 Data Lake Storage 存取。

## <a name="use-more-than-one-data-lake-storage-gen1-account"></a>使用一個以上的 Data Lake Storage Gen1 帳戶

將 Data Lake Storage 帳戶新增為額外的帳戶，並新增一個以上的 Data Lake Storage 帳戶即可完成。 為 HDInsight 叢集授與一或多個 Data Lake Storage 帳戶中資料的許可權。 請參閱下面的設定 Data Lake Storage Gen1 存取權。

## <a name="configure-data-lake-storage-gen1-access"></a>設定 Data Lake Storage Gen1 存取

若要設定從 HDInsight 叢集 Azure Data Lake Storage Gen1 存取，您必須擁有 Azure Active directory (Azure AD) 服務主體。 只有 Azure AD 系統管理員才可以建立服務主體。 必須使用憑證來建立服務主體。 如需詳細資訊，請參閱[快速入門：在 HDInsight 中設定叢集](./hdinsight-hadoop-provision-linux-clusters.md)以及[使用自我簽署憑證來建立服務主體](../active-directory/develop/howto-authenticate-service-principal-powershell.md#create-service-principal-with-self-signed-certificate)。

> [!NOTE]  
> 如果您即將使用 Azure Data Lake Storage Gen1 作為 HDInsight 叢集的額外儲存體，強烈建議您如本文所述建立叢集時執行此作業。 將 Azure Data Lake Storage Gen1 新增為現有 HDInsight 叢集的額外儲存體不是支援的案例。

如需存取控制模型的詳細資訊，請參閱 [Azure Data Lake Storage Gen1 中的存取控制](../data-lake-store/data-lake-store-access-control.md)。

## <a name="access-files-from-the-cluster"></a>從叢集存取檔案

有數種方式可讓您從 HDInsight 叢集存取 Data Lake Storage 中的檔案。

* **使用完整格式名稱**。 使用這種方法，您可以針對想要存取的檔案提供完整路徑。

    ```
    adl://<data_lake_account>.azuredatalakestore.net/<cluster_root_path>/<file_path>
    ```

* **使用簡短路徑格式**。 使用這種方法，您可以利用以下方式取代到叢集根目錄的路徑：

    ```
    adl:///<file path>
    ```

* **使用相對路徑**。 使用這種方法，您可以針對想要存取的檔案，只提供相對路徑。

    ```
    /<file.path>/
    ```

### <a name="data-access-examples"></a>資料存取範例

範例是以叢集前端節點的 [ssh 連線](./hdinsight-hadoop-linux-use-ssh-unix.md)為基礎。 這些範例會使用這三個 URI 配置。 `DATALAKEACCOUNT`將和取代 `CLUSTERNAME` 為相關的值。

#### <a name="a-few-hdfs-commands"></a>一些 hdfs 命令

1. 在本機儲存體上建立檔案。

    ```bash
    touch testFile.txt
    ```

1. 在叢集儲存體上建立目錄。

    ```bash
    hdfs dfs -mkdir adl://DATALAKEACCOUNT.azuredatalakestore.net/clusters/CLUSTERNAME/sampledata1/
    hdfs dfs -mkdir adl:///sampledata2/
    hdfs dfs -mkdir /sampledata3/
    ```

1. 將資料從本機儲存體複製到叢集儲存體。

    ```bash
    hdfs dfs -copyFromLocal testFile.txt adl://DATALAKEACCOUNT.azuredatalakestore.net/clusters/CLUSTERNAME/sampledata1/
    hdfs dfs -copyFromLocal testFile.txt adl:///sampledata2/
    hdfs dfs -copyFromLocal testFile.txt /sampledata3/
    ```

1. 列出叢集儲存體上的目錄內容。

    ```bash
    hdfs dfs -ls adl://DATALAKEACCOUNT.azuredatalakestore.net/clusters/CLUSTERNAME/sampledata1/
    hdfs dfs -ls adl:///sampledata2/
    hdfs dfs -ls /sampledata3/
    ```

#### <a name="creating-a-hive-table"></a>建立 Hive 資料表

顯示三個檔案位置以供說明之用。 若為實際執行，請僅使用其中一個 `LOCATION` 項目。

```hql
DROP TABLE myTable;
CREATE EXTERNAL TABLE myTable (
    t1 string,
    t2 string,
    t3 string,
    t4 string,
    t5 string,
    t6 string,
    t7 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
STORED AS TEXTFILE
LOCATION 'adl://DATALAKEACCOUNT.azuredatalakestore.net/clusters/CLUSTERNAME/example/data/';
LOCATION 'adl:///example/data/';
LOCATION '/example/data/';
```

## <a name="identify-storage-path-from-ambari"></a>識別來自 Ambari 的儲存體路徑

若要識別已設定之預設存放區的完整路徑，請流覽至 **HDFS**  >  **設定，** 然後 `fs.defaultFS` 在 [篩選] 輸入方塊中輸入。

## <a name="create-hdinsight-clusters-with-access-to-data-lake-storage-gen1"></a>建立可存取 Data Lake Storage Gen1 的 HDInsight 叢集

如需建立可存取 Data Lake Storage Gen1 的 HDInsight 叢集詳細指示，請使用下列連結。

* [使用入口網站](./hdinsight-hadoop-provision-linux-clusters.md)
* [使用 PowerShell (搭配 Data Lake Storage Gen1 作為預設儲存體)](../data-lake-store/data-lake-store-hdinsight-hadoop-use-powershell-for-default-storage.md)
* [使用 PowerShell (搭配 Data Lake Storage Gen1 作為其他儲存體)](../data-lake-store/data-lake-store-hdinsight-hadoop-use-powershell.md)
* [使用 Azure 範本](../data-lake-store/data-lake-store-hdinsight-hadoop-use-resource-manager-template.md)

## <a name="refresh-the-hdinsight-certificate-for-data-lake-storage-gen1-access"></a>重新整理存取 Data Lake Storage Gen1 時會用到的 HDInsight 憑證

下列範例 PowerShell 程式碼會從本機檔案或 Azure Key Vault 讀取憑證，然後使用新憑證更新 HDInsight 叢集，以存取 Azure Data Lake Storage Gen1。 提供您自己的 HDInsight 叢集名稱、資源組名、訂用帳戶識別碼、 `app ID` 憑證的本機路徑。 在系統提示時輸入密碼。

```powershell-interactive
$clusterName = '<clustername>'
$resourceGroupName = '<resourcegroupname>'
$subscriptionId = '01234567-8a6c-43bc-83d3-6b318c6c7305'
$appId = '01234567-e100-4118-8ba6-c25834f4e938'
$addNewCertKeyCredential = $true
$certFilePath = 'C:\localfolder\adls.pfx'
$KeyVaultName = "my-key-vault-name"
$KeyVaultSecretName = "my-key-vault-secret-name"
$certPassword = Read-Host "Enter Certificate Password"
# certSource
# 0 - create self signed cert
# 1 - read cert from file path
# 2 - read cert from key vault
$certSource = 0

Login-AzAccount
Select-AzSubscription -SubscriptionId $subscriptionId

if($certSource -eq 0)
{
    Write-Host "Generating new SelfSigned certificate"

    $cert = New-SelfSignedCertificate -CertStoreLocation "cert:\CurrentUser\My" -Subject "CN=hdinsightAdlsCert" -KeySpec KeyExchange
    $certBytes = $cert.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pkcs12, $certPassword);
    $certString = [System.Convert]::ToBase64String($certBytes)
}
elseif($certSource -eq 1)
{

    Write-Host "Reading the cert file from path $certFilePath"

    $cert = new-object System.Security.Cryptography.X509Certificates.X509Certificate2($certFilePath, $certPassword)
    $certString = [System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes($certFilePath))
}
elseif($certSource -eq 2)
{

    Write-Host "Reading the cert file from Azure Key Vault $KeyVaultName"

    $cert = (Get-AzureKeyVaultSecret -VaultName $KeyVaultName -Name $KeyVaultSecretName)
    $certValue = [System.Convert]::FromBase64String($cert.SecretValueText)
    $certObject = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2 -ArgumentList $certValue, $null,"Exportable, PersistKeySet"

    $certBytes = $certObject.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pkcs12, $certPassword.SecretValueText);

    $certString =[System.Convert]::ToBase64String($certBytes)
}

if($addNewCertKeyCredential)
{
    Write-Host "Creating new KeyCredential for the app"
    $keyValue = [System.Convert]::ToBase64String($cert.GetRawCertData())
    New-AzADAppCredential -ApplicationId $appId -CertValue $keyValue -EndDate $cert.NotAfter -StartDate $cert.NotBefore
    Write-Host "Waiting for 7 minutes for the permissions to get propagated"
    Start-Sleep -s 420 #7 minutes
}

Write-Host "Updating the certificate on HDInsight cluster..."

Invoke-AzResourceAction `
    -ResourceGroupName $resourceGroupName `
    -ResourceType 'Microsoft.HDInsight/clusters' `
    -ResourceName $clusterName `
    -ApiVersion '2015-03-01-preview' `
    -Action 'updateclusteridentitycertificate' `
    -Parameters @{ ApplicationId = $appId; Certificate = $certString; CertificatePassword = $certPassword.ToString() } `
    -Force
```

## <a name="next-steps"></a>後續步驟

在本文中，您已了解如何搭配 HDInsight 使用 HDFS 相容的 Azure Data Lake Storage Gen1。 此儲存體可讓您建立可調整、長期封存的資料取得解決方案。 並使用 HDInsight 來解除鎖定儲存的結構化和非結構化資料內的資訊。

如需詳細資訊，請參閱

* [快速入門：在 HDInsight 中設定叢集](./hdinsight-hadoop-provision-linux-clusters.md)
* [使用 Azure PowerShell 建立 HDInsight 叢集以使用 Data Lake Storage Gen1](../data-lake-store/data-lake-store-hdinsight-hadoop-use-powershell.md)
* [將資料上傳至 HDInsight](hdinsight-upload-data.md)
* [使用 Azure Blob 儲存體共用存取簽章來限制 HDInsight 的資料存取](hdinsight-storage-sharedaccesssignature-permissions.md)