---
title: 在 Azure HDInsight 中使用 Data Lake Store 搭配 Hadoop
description: 了解如何從 Azure Data Lake Store 查詢資料，以及如何儲存分析的結果。
services: hdinsight,storage
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 07/23/2018
ms.openlocfilehash: 1f2863ac13a6b94a226cdc01a57f6554f75625ae
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43103672"
---
# <a name="use-data-lake-store-with-azure-hdinsight-clusters"></a>使用 Data Lake Store 搭配 Azure HDInsight 叢集

若要分析 HDInsight 叢集中的資料，您可以在 [Azure 儲存體](../storage/common/storage-introduction.md)、[Azure Data Lake Store](../data-lake-store/data-lake-store-overview.md)，或兩者中儲存資料。 這兩種儲存體選項都可讓您安全地刪除用於計算的 HDInsight 叢集，而不會遺失使用者資料。

在本文中，您將了解 Data Lake Store 與 HDInsight 叢集搭配運作的方式。 若要深入了解 Azure 儲存體與 HDInsight 叢集搭配運作的方式，請參閱[使用 Azure 儲存體搭配 Azure HDInsight 叢集](hdinsight-hadoop-use-blob-storage.md)。 如需建立 HDInsight 叢集的詳細資訊，請參閱[在 HDInsight 中建立 Hadoop 叢集](hdinsight-hadoop-provision-linux-clusters.md)。

> [!NOTE]
> Data Lake Store 一律透過安全通道存取，因此不會有 `adls` 檔案系統配置名稱。 您會一律使用 `adl`。
> 


## <a name="availability-for-hdinsight-clusters"></a>HDInsight 叢集的可用性

Hadoop 支援預設檔案系統的概念。 預設檔案系統意指預設配置和授權。 也可用來解析相對路徑。 進行 HDInsight 叢集建立程序時，您可以指定 Azure Blob 儲存體中的 Blob 容器作為預設檔案系統，或在使用 HDInsight 3.5 和更新版本時，選取 Azure 儲存體或 Azure Data Lake Store 作為預設檔案系統，有一些例外狀況。 

HDInsight 叢集可透過兩種方式來使用 Data Lake Store︰

* 作為預設儲存體
* 作為其他儲存體，搭配 Azure 儲存體 Blob 作為預設儲存體。

目前，只有一些 HDInsight 叢集類型/版本支援使用 Data Lake Store 來作為預設儲存體和其他儲存體帳戶：

| HDInsight 叢集類型 | Data Lake Store 作為預設儲存體 | Data Lake Store 作為其他儲存體| 注意 |
|------------------------|------------------------------------|---------------------------------------|------|
| HDInsight 3.6 版 | 是 | 是 | |
| HDInsight 3.5 版 | 是 | 是 | HBase 的例外狀況|
| HDInsight 3.4 版 | 否 | 是 | |
| HDInsight 3.3 版 | 否 | 否 | |
| HDInsight 3.2 版 | 否 | 是 | |
| Storm | | |您可以使用 Data Lake Store 從 Storm 拓撲寫入資料。 您也可以使用 Data Lake Store 做為參考資料，該資料稍後可以由 Storm 拓撲讀取。|

使用 Data Lake Store 作為其他儲存體帳戶，並不會影響效能或從叢集讀取或寫入至 Azure 儲存體的能力。


## <a name="use-data-lake-store-as-default-storage"></a>使用 Data Lake Store 作為預設儲存體

使用 Data Lake Store 作為預設儲存體部署 HDInsight 時，儲存在 Data Lake Store 中的叢集相關檔案會位在下列位置︰

    adl://mydatalakestore/<cluster_root_path>/

其中 `<cluster_root_path>` 是您在 Data Lake Store 中建立的資料夾名稱。 藉由指定每個叢集的根路徑，您可針對多個叢集使用相同的 Data Lake Store 帳戶。 因此在您的設定中︰

* Cluster1 可以使用路徑 `adl://mydatalakestore/cluster1storage`
* Cluster2 可以使用路徑 `adl://mydatalakestore/cluster2storage`

請注意，這兩個叢集都使用相同的 Data Lake Store 帳戶 **mydatalakestore**。 每個叢集都會在 Data Lake Store 中存取自己的根檔案系統。 特別是 Azure 入口網站的部署經驗會提示您使用像是 **/clusters/\<clustername >** 的資料夾名稱做為根路徑。

若要能夠使用 Data Lake Store 作為預設儲存體，您必須授與服務主體存取下列路徑：

- Data Lake Store 帳戶根目錄。  例如：adl://mydatalakestore/。
- 所有叢集資料夾的資料夾。  例如：adl://mydatalakestore/clusters。
- 叢集的資料夾。  例如：adl://mydatalakestore/clusters/cluster1storage。

如需建立服務主體和授與存取權的詳細資訊，請參閱[設定 Data Lake Store 存取](#configure-data-lake-store-access)。


## <a name="use-data-lake-store-as-additional-storage"></a>使用 Data Lake Store 作為其他儲存體

您也可以使用 Data Lake Store 做為叢集的其他儲存體。 在這種情況下，叢集預設儲存體可以是 Azure 儲存體 Blob 或 Data Lake Store 帳戶。 如果您正在作為其他儲存體的 Data Lake Store 上針對其儲存的資料執行 HDInsight 作業，必須使用檔案的完整路徑。 例如︰

    adl://mydatalakestore.azuredatalakestore.net/<file_path>

請注意，現在 URL 中沒有任何 **cluster_root_path**。 這是因為在此情況下 Data Lake Store 不是預設儲存體，因此您只需要提供檔案的路徑。

若要能夠使用 Data Lake Store 作為其他儲存體，您只需要將您儲存檔案之位置的路徑存取權授與服務主體即可。  例如︰

    adl://mydatalakestore.azuredatalakestore.net/<file_path>

如需建立服務主體和授與存取權的詳細資訊，請參閱[設定 Data Lake Store 存取](#configure-data-lake-store-access)。


## <a name="use-more-than-one-data-lake-store-accounts"></a>使用多個 Data Lake Store 帳戶

在一個或多個 Data Lake Store 帳戶中提供 HDInsight 叢集的權限，即可完成新增 Data Lake Store 帳戶作為其他帳戶，以及新增一個以上的 Data Lake Store 帳戶。 請參閱[設定 Data Lake Store 存取](#configure-data-lake-store-access)。

## <a name="configure-data-lake-store-access"></a>設定 Data Lake Store 存取

若要從 HDInsight 叢集設定 Data Lake Store 存取，您必須擁有 Azure Active Directory (Azure AD) 服務主體。 只有 Azure AD 系統管理員才可以建立服務主體。 必須使用憑證來建立服務主體。 如需詳細資訊，請參閱[快速入門：在 HDInsight 中設定叢集](../storage/data-lake-storage/quickstart-create-connect-hdi-cluster.md)以及[使用自我簽署憑證來建立服務主體](../azure-resource-manager/resource-group-authenticate-service-principal.md#create-service-principal-with-self-signed-certificate)。

> [!NOTE]
> 如果您即將使用 Azure Data Lake Store 作為 HDInsight 叢集的額外儲存體，強烈建議您如本文所述建立叢集時執行此作業。 將 Azure Data Lake Store 新增為現有 HDInsight 叢集的額外儲存體不是支援的案例。
>

## <a name="access-files-from-the-cluster"></a>從叢集存取檔案

有數種方式可讓您從 HDInsight 叢集存取 Data Lake Store 中的檔案。

* **使用完整格式名稱**。 使用這種方法，您可以針對想要存取的檔案提供完整路徑。

        adl://mydatalakestore.azuredatalakestore.net/<cluster_root_path>/<file_path>

* **使用簡短路徑格式**。 使用這種方法，您可以利用 adl:/// 將路徑向上取代至叢集根目錄。 因此在上述範例中，您可以將 `adl://mydatalakestore.azuredatalakestore.net/<cluster_root_path>/`取代為 `adl:///`。

        adl:///<file path>

* **使用相對路徑**。 使用這種方法，您可以針對想要存取的檔案，只提供相對路徑。 例如，如果檔案的完整路徑如下︰

        adl://mydatalakestore.azuredatalakestore.net/<cluster_root_path>/example/data/sample.log

    您可以改用此相對路徑來存取相同的 sample.log 檔案。

        /example/data/sample.log

## <a name="create-hdinsight-clusters-with-access-to-data-lake-store"></a>建立可存取 Data Lake Store 的 HDInsight 叢集

如需建立可存取 Data Lake Store 的 HDInsight 叢集詳細指示，請使用下列連結。

* [使用入口網站](../storage/data-lake-storage/quickstart-create-connect-hdi-cluster.md)
* [使用 PowerShell (搭配 Data Lake Store 做為預設儲存體)](../data-lake-store/data-lake-store-hdinsight-hadoop-use-powershell-for-default-storage.md)
* [使用 PowerShell (搭配 Data Lake Store 做為其他儲存體)](../data-lake-store/data-lake-store-hdinsight-hadoop-use-powershell.md)
* [使用 Azure 範本](../data-lake-store/data-lake-store-hdinsight-hadoop-use-resource-manager-template.md)

## <a name="refresh-the-hdinsight-certificate-for-data-lake-store-access"></a>重新整理存取 Data Lake Store 時會用到的 HDInsight 憑證

下列範例 PowerShell 程式碼會讀取本機憑證檔，並使用新憑證更新 HDInsight 叢集，以存取 Data Lake Store。 提供您自己的 HDInsight 叢集名稱、資源群組名稱、訂用帳戶識別碼、應用程式識別碼、憑證的本機路徑。 在系統提示時輸入密碼。

```powershell-interactive
$clusterName = '<clustername>'
$resourceGroupName = '<resourcegroupname>'
$subscriptionId = '01234567-8a6c-43bc-83d3-6b318c6c7305'
$appId = '01234567-e100-4118-8ba6-c25834f4e938'
$generateSelfSignedCert = $false
$addNewCertKeyCredential = $true
$certFilePath = 'C:\localfolder\adls.pfx'
$certPassword = Read-Host "Enter Certificate Password"

if($generateSelfSignedCert)
{
    Write-Host "Generating new SelfSigned certificate"
    
    $cert = New-SelfSignedCertificate -CertStoreLocation "cert:\CurrentUser\My" -Subject "CN=hdinsightAdlsCert" -KeySpec KeyExchange
    $certBytes = $cert.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pkcs12, $certPassword);
    $certString = [System.Convert]::ToBase64String($certBytes)
}
else
{

    Write-Host "Reading the cert file from path $certFilePath"

    $cert = new-object System.Security.Cryptography.X509Certificates.X509Certificate2($certFilePath, $certPassword)
    $certString = [System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes($certFilePath))
}

Login-AzureRmAccount

if($addNewCertKeyCredential)
{
    Write-Host "Creating new KeyCredential for the app"
    $keyValue = [System.Convert]::ToBase64String($cert.GetRawCertData())
    New-AzureRmADAppCredential -ApplicationId $appId -CertValue $keyValue -EndDate $cert.NotAfter -StartDate $cert.NotBefore
    Write-Host "Waiting for 30 seconds for the permissions to get propagated"
    Start-Sleep -s 30
}

Select-AzureRmSubscription -SubscriptionId $subscriptionId
Write-Host "Updating the certificate on HDInsight cluster..."

Invoke-AzureRmResourceAction `
    -ResourceGroupName $resourceGroupName `
    -ResourceType 'Microsoft.HDInsight/clusters' `
    -ResourceName $clusterName `
    -ApiVersion '2015-03-01-preview' `
    -Action 'updateclusteridentitycertificate' `
    -Parameters @{ ApplicationId = $appId; Certificate = $certString; CertificatePassword = $certPassword.ToString() } `
    -Force
```

## <a name="next-steps"></a>後續步驟
在本文中，您已了解如何搭配 HDInsight 使用 HDFS 相容的 Azure Data Lake Store。 這可讓您建立可調整、長期封存的資料取得解決方案，並利用 HDInsight 來揭開儲存的結構化和非結構化資料內的資訊。

如需詳細資訊，請參閱

* [開始使用 Azure HDInsight][hdinsight-get-started]
* [快速入門：在 HDInsight 中設定叢集](../storage/data-lake-storage/quickstart-create-connect-hdi-cluster.md)
* [使用 Azure PowerShell 建立 HDInsight 叢集以使用 Data Lake Store](../data-lake-store/data-lake-store-hdinsight-hadoop-use-powershell.md)
* [將資料上傳至 HDInsight][hdinsight-upload-data]
* [搭配 HDInsight 使用 Hivet][hdinsight-use-hive]
* [搭配 HDInsight 使用 Pig][hdinsight-use-pig]
* [使用 Azure 儲存體共用存取簽章來限制使用 HDInsight 對資料的存取][hdinsight-use-sas]

[hdinsight-use-sas]: hdinsight-storage-sharedaccesssignature-permissions.md
[powershell-install]: /powershell/azureps-cmdlets-docs
[hdinsight-creation]: hdinsight-hadoop-provision-linux-clusters.md
[hdinsight-get-started]:hadoop/apache-hadoop-linux-tutorial-get-started.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-use-hive]:hadoop/hdinsight-use-hive.md
[hdinsight-use-pig]:hadoop/hdinsight-use-pig.md

[blob-storage-restAPI]: http://msdn.microsoft.com/library/windowsazure/dd135733.aspx
[azure-storage-create]:../storage/common/storage-create-storage-account.md

[img-hdi-powershell-blobcommands]: ./media/hdinsight-hadoop-use-blob-storage/HDI.PowerShell.BlobCommands.png
[img-hdi-quick-create]: ./media/hdinsight-hadoop-use-blob-storage/HDI.QuickCreateCluster.png
[img-hdi-custom-create-storage-account]: ./media/hdinsight-hadoop-use-blob-storage/HDI.CustomCreateStorageAccount.png  
