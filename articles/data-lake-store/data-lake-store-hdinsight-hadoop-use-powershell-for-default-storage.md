---
title: 使用 PowerShell 建立以 Data Lake Store 做為預設儲存體的 HDInsight 叢集 | Microsoft Docs'
description: 使用 Azure PowerShell 建立及使用 HDInsight 叢集與 Azure Data Lake Store
services: data-lake-store,hdinsight
documentationcenter: ''
author: nitinme
manager: jhubbard
editor: cgronlun
ms.assetid: 8917af15-8e37-46cf-87ad-4e6d5d67ecdb
ms.service: data-lake-store
ms.devlang: na
ms.topic: article
ms.date: 05/29/2018
ms.author: nitinme
ms.openlocfilehash: 71b6b507952793e34a0e0413d7d652640680dab7
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34625700"
---
# <a name="create-hdinsight-clusters-with-data-lake-store-as-default-storage-by-using-powershell"></a>使用 PowerShell 建立以 Data Lake Store 做為預設儲存體的 HDInsight 叢集

> [!div class="op_single_selector"]
> * [使用 Azure 入口網站](data-lake-store-hdinsight-hadoop-use-portal.md)
> * [使用 PowerShell (針對預設儲存體)](data-lake-store-hdinsight-hadoop-use-powershell-for-default-storage.md)
> * [使用 PowerShell (針對額外儲存體)](data-lake-store-hdinsight-hadoop-use-powershell.md)
> * [使用 Resource Manager](data-lake-store-hdinsight-hadoop-use-resource-manager-template.md)

了解如何使用 Azure PowerShell 來設定以 Azure Data Lake Store 做為預設儲存體的 HDInsight 叢集。 如需有關如何建立以 Data Lake Store 做為額外儲存體的 HDInsight 叢集之指示，請參閱[建立以 Data Lake Store 做為額外儲存體的 HDInsight 叢集](data-lake-store-hdinsight-hadoop-use-powershell.md)。

以下是使用 HDInsight 搭配 Data Lake Store 的一些重要考量：

* HDInsight 3.5 和 3.6 版提供建立可存取 Data Lake Store 作為預設儲存體之 HDInsight 叢集的選項。

* HDInsight Premium 叢集「不提供」建立可存取 Data Lake Store 做為預設儲存體之 HDInsight 叢集的選項。

若要使用 PowerShell 來設定 HDInsight 搭配 Data Lake Store 運作，請遵循接下來五小節中的指示操作。

## <a name="prerequisites"></a>先決條件

開始本教學課程之前，請確定您符合下列需求：

* **Azure 訂用帳戶**：請移至[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。
* **Azure PowerShell 1.0 或更新版本**：請參閱 [如何安裝和設定 PowerShell](/powershell/azure/overview)。
* **Windows 軟體開發套件 (SDK)**︰若要安裝 Windows SDK，請移至 [Windows 10 下載和工具](https://dev.windows.com/downloads)。 SDK 是用來建立安全性憑證。
* **Azure Active Directory 服務主體**︰本教學課程說明如何在 Azure Active Directory (Azure AD) 中建立服務主體。 不過，您必須是 Azure AD 系統管理員，才能建立服務主體。 如果您是系統管理員，就可以略過這項先決條件並繼續進行本教學課程。

    >[!NOTE]
    >唯有您是 Azure AD 系統管理員，才可以建立服務主體。 您的 Azure AD 系統管理員必須先建立服務主體，您才能建立搭配 Data Lake Store 的 HDInsight 叢集。 必須使用憑證來建立服務主體，如[使用憑證建立服務主體](../azure-resource-manager/resource-group-authenticate-service-principal.md#create-service-principal-with-certificate-from-certificate-authority)所述。
    >

## <a name="create-a-data-lake-store-account"></a>建立 Data Lake Store 帳戶

若要建立 Data Lake Store 帳戶，請執行下列作業︰

1. 從您的桌面開啟 PowerShell 視窗，然後輸入下列程式碼片段。 當系統提示您登入時，請以其中一個訂用帳戶管理員或擁有者身分登入。 

        # Sign in to your Azure account
        Connect-AzureRmAccount

        # List all the subscriptions associated to your account
        Get-AzureRmSubscription

        # Select a subscription
        Set-AzureRmContext -SubscriptionId <subscription ID>

        # Register for Data Lake Store
        Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.DataLakeStore"

    > [!NOTE]
    > 如果您註冊 Data Lake Store 資源提供者並收到類似 `Register-AzureRmResourceProvider : InvalidResourceNamespace: The resource namespace 'Microsoft.DataLakeStore' is invalid` 的錯誤，您的訂用帳戶可能不在 Data Lake Store 的允許清單中。 若要針對 Data Lake Store 公開預覽版來啟用 Azure 訂用帳戶，請遵循[透過 Azure 入口網站開始使用 Azure Data Lake Store](data-lake-store-get-started-portal.md)中的指示操作。
    >

2. Data Lake Store 帳戶與 Azure 資源群組相關聯。 從建立資源群組著手。

        $resourceGroupName = "<your new resource group name>"
        New-AzureRmResourceGroup -Name $resourceGroupName -Location "East US 2"

    您應該會看到如下的輸出：

        ResourceGroupName : hdiadlgrp
        Location          : eastus2
        ProvisioningState : Succeeded
        Tags              :
        ResourceId        : /subscriptions/<subscription-id>/resourceGroups/hdiadlgrp

3. 建立 Data Lake Store 帳戶。 您指定的帳戶名稱必須只包含小寫字母和數字。

        $dataLakeStoreName = "<your new Data Lake Store name>"
        New-AzureRmDataLakeStoreAccount -ResourceGroupName $resourceGroupName -Name $dataLakeStoreName -Location "East US 2"

    您應該會看到如下的輸出：

        ...
        ProvisioningState           : Succeeded
        State                       : Active
        CreationTime                : 5/5/2017 10:53:56 PM
        EncryptionState             : Enabled
        ...
        LastModifiedTime            : 5/5/2017 10:53:56 PM
        Endpoint                    : hdiadlstore.azuredatalakestore.net
        DefaultGroup                :
        Id                          : /subscriptions/<subscription-id>/resourceGroups/hdiadlgrp/providers/Microsoft.DataLakeStore/accounts/hdiadlstore
        Name                        : hdiadlstore
        Type                        : Microsoft.DataLakeStore/accounts
        Location                    : East US 2
        Tags                        : {}

4. 使用 Data Lake Store 做為預設儲存體時，您需要指定根路徑，而叢集特定檔案會在叢集建立期間複製到該路徑。 若要建立根路徑 (也就是程式碼片段中的 **/clusters/hdiadlcluster**)，請使用下列 Cmdlet:

        $myrootdir = "/"
        New-AzureRmDataLakeStoreItem -Folder -AccountName $dataLakeStoreName -Path $myrootdir/clusters/hdiadlcluster


## <a name="set-up-authentication-for-role-based-access-to-data-lake-store"></a>設定 Data Lake Store 以角色為基礎的存取的驗證
每一個 Azure 訂用帳戶都與 Azure AD 實體相關聯。 透過 Azure 入口網站或 Azure Resource Manager API 存取訂用帳戶資源的使用者與服務，都必須先向 Azure AD 進行驗證。 您可以在 Azure 資源上為 Azure 訂用帳戶和服務指派適當的角色，以授與其存取權限。 對於服務，服務主體會識別 Azure AD 中的服務。

本章節將說明如何將 Azure 資源 (您稍早建立的 Data Lake Store 帳戶) 的存取權授與 HDInsight 等應用程式服務。 其方法是建立應用程式的服務主體，並透過 Azure PowerShell 將角色指派給它。

若要設定 Azure Data Lake 的 Active Directory 驗證，請執行下列兩節中的工作。

### <a name="create-a-self-signed-certificate"></a>建立自我簽署憑證
進行本節中的步驟之前，請確定您已安裝 [Windows SDK](https://dev.windows.com/en-us/downloads)。 您也必須建立一個目錄 (例如 *C:\mycertdir*)，以在其中建立憑證。

1. 在 PowerShell 視窗中，移至您安裝 Windows SDK 的位置 (通常是 *C:\Program Files (x86)\Windows Kits\10\bin\x86*)，並使用 [MakeCert][makecert] 公用程式來建立自我簽署的憑證和私密金鑰。 使用下列命令：

        $certificateFileDir = "<my certificate directory>"
        cd $certificateFileDir

        makecert -sv mykey.pvk -n "cn=HDI-ADL-SP" CertFile.cer -r -len 2048

    系統會提示您輸入私密金鑰密碼。 命令成功執行之後，您應該會在您指定的憑證目錄中看到 **CertFile.cer** 和 **mykey.pvk**。
2. 使用 [Pvk2Pfx][pvk2pfx] 公用程式將 MakeCert 建立的 .pvk 和 .cer 檔案轉換成 .pfx 檔案。 執行以下命令：

        pvk2pfx -pvk mykey.pvk -spc CertFile.cer -pfx CertFile.pfx -po <password>

    系統提示您時，請輸入您稍早指定的私密金鑰密碼。 您針對 **-po** 參數指定的值是與 .pfx 檔案相關聯的密碼。 命令成功完成之後，在您指定的憑證目錄中，您應該也會看到 **CertFile.pfx**。

### <a name="create-an-azure-ad-and-a-service-principal"></a>建立 Azure AD 和服務主體
在這一節中，您會建立 Azure AD 應用程式的服務主體、指派角色給服務主體，並藉由提供憑證驗證為服務主體。 若要在 Azure AD 中建立應用程式，請執行下列命令：

1. 在 PowerShell 主控台視窗中貼上下列 Cmdlet。 請確定您針對 **-DisplayName** 屬性指定的值是唯一的。 **-HomePage** 和 **-IdentiferUris** 的值是預留位置值，不會進行驗證。

        $certificateFilePath = "$certificateFileDir\CertFile.pfx"

        $password = Read-Host -Prompt "Enter the password" # This is the password you specified for the .pfx file

        $certificatePFX = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($certificateFilePath, $password)

        $rawCertificateData = $certificatePFX.GetRawCertData()

        $credential = [System.Convert]::ToBase64String($rawCertificateData)

        $application = New-AzureRmADApplication `
            -DisplayName "HDIADL" `
            -HomePage "https://contoso.com" `
            -IdentifierUris "https://mycontoso.com" `
            -CertValue $credential  `
            -StartDate $certificatePFX.NotBefore  `
            -EndDate $certificatePFX.NotAfter

        $applicationId = $application.ApplicationId
2. 使用應用程式識別碼建立服務主體。

        $servicePrincipal = New-AzureRmADServicePrincipal -ApplicationId $applicationId

        $objectId = $servicePrincipal.Id
3. 將服務主體存取權授與 Data Lake Store 根目錄，以及您稍早指定之根路徑中的所有資料夾。 使用下列 Cmdlet：

        Set-AzureRmDataLakeStoreItemAclEntry -AccountName $dataLakeStoreName -Path / -AceType User -Id $objectId -Permissions All
        Set-AzureRmDataLakeStoreItemAclEntry -AccountName $dataLakeStoreName -Path /clusters -AceType User -Id $objectId -Permissions All
        Set-AzureRmDataLakeStoreItemAclEntry -AccountName $dataLakeStoreName -Path /clusters/hdiadlcluster -AceType User -Id $objectId -Permissions All

## <a name="create-an-hdinsight-linux-cluster-with-data-lake-store-as-the-default-storage"></a>建立 HDInsight Linux 叢集搭配 Data Lake Store 做為預設儲存體

在本節中，您會建立一個 HDInsight Hadoop Linux 叢集搭配 Data Lake Store 做為預設儲存體。 對於此版本，HDInsight 叢集和 Data Lake Store 必須位於相同位置。

1. 擷取訂用帳戶的租用戶識別碼，並加以儲存以供日後使用。

        $tenantID = (Get-AzureRmContext).Tenant.TenantId

2. 使用下列 Cmdlet 建立 HDInsight 叢集︰

        # Set these variables

        $location = "East US 2"
        $storageAccountName = $dataLakeStoreName                       # Data Lake Store account name
        $storageRootPath = "<Storage root path you specified earlier>" # E.g. /clusters/hdiadlcluster
        $clusterName = "<unique cluster name>"
        $clusterNodes = <ClusterSizeInNodes>            # The number of nodes in the HDInsight cluster
        $httpCredentials = Get-Credential
        $sshCredentials = Get-Credential

        New-AzureRmHDInsightCluster `
               -ClusterType Hadoop `
               -OSType Linux `
               -ClusterSizeInNodes $clusterNodes `
               -ResourceGroupName $resourceGroupName `
               -ClusterName $clusterName `
               -HttpCredential $httpCredentials `
               -Location $location `
               -DefaultStorageAccountType AzureDataLakeStore `
               -DefaultStorageAccountName "$storageAccountName.azuredatalakestore.net" `
               -DefaultStorageRootPath $storageRootPath `
               -Version "3.6" `
               -SshCredential $sshCredentials `
               -AadTenantId $tenantId `
               -ObjectId $objectId `
               -CertificateFilePath $certificateFilePath `
               -CertificatePassword $password

    Cmdlet 成功完成後，您應該會看到列出叢集詳細資料的輸出。

## <a name="run-test-jobs-on-the-hdinsight-cluster-to-use-data-lake-store"></a>在 HDInsight 叢集上執行測試工作以使用 Data Lake Store
設定 HDInsight 叢集之後，您可以在叢集上執行測試工作，確保它可以存取 Data Lake Store。 若要這麼做，請執行範例 Hive 作業，以使用已在 Data Lake Store 中於 *<cluster root>/example/data/sample.log* 提供使用的範例資料建立資料表。

在這一節中，您將透過安全殼層 (SSH) 連線到您所建立的 HDInsight Linux 叢集，然後執行範例 Hive 查詢。

* 如果您使用 Windows 用戶端來透過 SSH 連線到叢集，請參閱[從 Windows 在 HDInsight 上搭配使用 SSH 與以 Linux 為基礎的 Hadoop](../hdinsight/hdinsight-hadoop-linux-use-ssh-windows.md)。
* 如果您使用 Linux 用戶端來透過 SSH 連線到叢集，請參閱[從 Linux 在 HDInsight 上搭配使用 SSH 與以 Linux 為基礎的 Hadoop](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md)。

1. 在連線之後，使用下列命令來啟動 Hive 命令列介面 (CLI)：

        hive
2. 使用 CLI 輸入下列陳述式，以使用 Data Lake Store 中的範例資料來建立名為 **vehicles** 的新資料表：

        DROP TABLE log4jLogs;
        CREATE EXTERNAL TABLE log4jLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string)
        ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
        STORED AS TEXTFILE LOCATION 'adl:///example/data/';
        SELECT t4 AS sev, COUNT(*) AS count FROM log4jLogs WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log' GROUP BY t4;

    您應該會在 SSH 主控台上看到查詢輸出。

    >[!NOTE]
    >上述 CREATE TABLE 命令中範例資料的路徑為 `adl:///example/data/`，其中 `adl:///` 是叢集根目錄。 遵循本教學課程中指定之叢集根目錄的範例，此命令為 `adl://hdiadlstore.azuredatalakestore.net/clusters/hdiadlcluster`。 您可以使用較短的替代路徑，或提供叢集根目錄的完整路徑。
    >

## <a name="access-data-lake-store-by-using-hdfs-commands"></a>使用 HDFS 命令存取 Data Lake Store
在您設定 HDInsight 叢集使用 Data Lake Store 之後，您可以使用分散式檔案系統 (HDFS) 殼層命令來存取存放區。

在這一節中，您將透過 SSH 連線到您所建立的 HDInsight Linux 叢集，然後執行 HDFS 命令。

* 如果您使用 Windows 用戶端來透過 SSH 連線到叢集，請參閱[從 Windows 在 HDInsight 上搭配使用 SSH 與以 Linux 為基礎的 Hadoop](../hdinsight/hdinsight-hadoop-linux-use-ssh-windows.md)。
* 如果您使用 Linux 用戶端來透過 SSH 連線到叢集，請參閱[從 Linux 在 HDInsight 上搭配使用 SSH 與以 Linux 為基礎的 Hadoop](../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md)。

連線之後，使用下列 HDFS 檔案系統命令列出 Data Lake Store 中的檔案。

    hdfs dfs -ls adl:///

您也可以使用 `hdfs dfs -put` 命令，將一些檔案上傳至 Data Lake Store，然後使用 `hdfs dfs -ls` 以確認是否成功上傳檔案。

## <a name="see-also"></a>另請參閱
* [搭配 Azure HDInsight 叢集使用 Data Lake Store](../hdinsight/hdinsight-hadoop-use-data-lake-store.md)
* [Azure 入口網站：建立 HDInsight 叢集以使用 Data Lake Store](data-lake-store-hdinsight-hadoop-use-portal.md)

[makecert]: https://msdn.microsoft.com/library/windows/desktop/ff548309(v=vs.85).aspx
[pvk2pfx]: https://msdn.microsoft.com/library/windows/desktop/ff550672(v=vs.85).aspx
