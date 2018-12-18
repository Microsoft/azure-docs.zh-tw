---
title: 使用共用存取簽章限制存取 - Azure HDInsight
description: 深入了解使用共用存取簽章限制 HDInsight 對儲存在 Azure 儲存體 blob 中的資料的存取。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 04/23/2018
ms.author: jasonh
ms.openlocfilehash: 76e40f5178457f366ed386dba7a1817ddde3c8ac
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43090223"
---
# <a name="use-azure-storage-shared-access-signatures-to-restrict-access-to-data-in-hdinsight"></a>使用 Azure 儲存體共用存取簽章來限制 HDInsight 對資料的存取

HDInsight 對於與叢集建立關聯之 Azure 儲存體帳戶中的資料具有完整存取權。 您可以在 Blob 容器上使用共用存取簽章來限制對資料的存取。 共用存取簽章 (SAS) 是一種 Azure 儲存體帳戶功能，可讓您限制資料的存取權。 例如，提供資料的唯讀存取。

> [!IMPORTANT]
> 對於使用 Apache Ranger 的解決方案，請考慮使用已加入網域的 HDInsight。 如需詳細資訊，請參閱[設定已加入網域的 HDInsight](./domain-joined/apache-domain-joined-configure.md) 文件。

> [!WARNING]
> HDInsight 必須有叢集預設儲存體的完整存取權。

## <a name="requirements"></a>需求

* Azure 訂用帳戶
* C# 或 Python。 提供 C# 範例程式碼做為 Visual Studio 解決方案。

  * Visual Studio 的版本必須是 2013、2015 或 2017
  * Python 的版本必須是 2.7 或更新版本

* 以 Linux 為基礎的 HDInsight 叢集「或」[Azure PowerShell][powershell] - 如果您有以 Linux 為基礎的現有叢集，則可以使用 Ambari 將共用存取簽章新增至叢集。 如果沒有，您可以使用 Azure PowerShell 建立叢集，並在叢集建立期間新增共用存取簽章。

    > [!IMPORTANT]
    > Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](hdinsight-component-versioning.md#hdinsight-windows-retirement)。

* 範例檔案來自 [https://github.com/Azure-Samples/hdinsight-dotnet-python-azure-storage-shared-access-signature](https://github.com/Azure-Samples/hdinsight-dotnet-python-azure-storage-shared-access-signature)。 此儲存機制包含下列項目：

  * Visual Studio 專案，可以建立儲存體容器、預存原則，以及搭配 HDInsight 使用的 SAS
  * Python 指令碼，可以建立儲存體容器、預存原則，以及搭配 HDInsight 使用的 SAS
  * PowerShell 指令碼，可以建立 HDInsight 叢集，並將它設定為使用 SAS。

## <a name="shared-access-signatures"></a>共用存取簽章

共用存取簽章有兩種格式：

* 臨機操作：開始時間、到期時間與 SAS 之權限全都標示於 SAS URI 中。

* 預存存取原則：預存存取原則會在資源容器中定義，例如 Blob 容器。 原則可以用來管理一或多個共用存取簽章的限制。 當您將 SAS 與預存存取原則建立關聯時，SAS 會繼承為該預存存取原則所定義的限制 (開始時間、過期時間和權限)。

這兩種格式間的差異對於以下這一個重要案例而言相當重要：撤銷。 SAS 是一種 URL，因此取得 SAS 的任何人都可以使用它，無論起先要求的人是誰。 如果是公開發佈 SAS，則全世界的人都可以使用此 SAS。 散佈的 SAS 在發生以下四個情況其中之一之前都會持續有效：

1. 已到達 SAS 上指定的過期時間。

2. 已到達 SAS 參考的預存存取原則所指定的到期時間。 下列情節會導致過期時間到達：

    * 時間間隔已過。
    * 預存存取原則之過期時間修改為過去的時間。 改變過期時間是撤銷 SAS 的方法之一。

3. 已刪除 SAS 所參考之預存存取原則，這是撤銷 SAS 的另外一種方法。 如果您以相同的名稱重新建立預存存取原則，舊原則的所有 SAS 權杖都會有效 (如果未超過 SAS 的到期時間)。 如果您打算撤銷 SAS，且如果您要使用未來的過期時間來重新建立存取原則，則務必使用不同的名稱。

4. 系統會重新產生用來建立 SAS 的帳戶金鑰。 重新產生金鑰會造成使用舊金鑰的所有應用程式驗證失敗。 將所有元件更新為新金鑰。

> [!IMPORTANT]
> 共用存取簽章 URI 會與用來建立簽章的帳戶金鑰，以及相關聯的預存的存取原則 (如果有的話) 產生關聯。 如果未指定任何預存的存取原則，則撤銷共用存取簽章的唯一方式是變更帳戶金鑰。

建議您一律使用預存的存取原則。 使用預存原則時，可以視需求撤銷簽章或延長過期時間。 此文件的步驟使用預存存取原則來產生 SAS。

如需共用存取簽章的詳細資訊，請參閱 [了解 SAS 模型](../storage/common/storage-dotnet-shared-access-signature-part-1.md)。

### <a name="create-a-stored-policy-and-sas-using-c"></a>使用 C\# 建立預存原則和 SAS

1. 在 Visual Studio 中開啟解決方案。

2. 在 [方案總管] 中，於 **SASToken** 專案上按一下滑鼠右鍵，然後選取 [屬性]。

3. 選取 [設定]  ，並新增下列項目的值：

   * StorageConnectionString：您想要為其建立預存原則和 SAS 的儲存體帳戶的連接字串。 其格式應為 `DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=mykey`，其中 `myaccount` 是儲存體帳戶名稱，而 `mykey` 是儲存體帳戶金鑰。

   * ContainerName：您想要限制存取的儲存體帳戶中的容器。

   * SASPolicyName：針對將要建立的預存原則使用的名稱。

   * FileToUpload：上傳至容器之檔案的路徑。

4. 執行專案。 在 SAS 產生後，將顯示類似下列文字的資訊：

        Container SAS token using stored access policy: sr=c&si=policyname&sig=dOAi8CXuz5Fm15EjRUu5dHlOzYNtcK3Afp1xqxniEps%3D&sv=2014-02-14

    儲存 SAS 原則權杖、儲存體帳戶名稱和容器名稱。 將儲存體帳戶與您的 HDInsight 叢集相關聯時，會使用這些值。

### <a name="create-a-stored-policy-and-sas-using-python"></a>使用 Python 建立預存原則和 SAS

1. 開啟 SASToken.py 檔案並且變更下列值：

   * policy\_name：針對將要建立的預存原則使用的名稱。

   * storage\_account\_name：您的儲存體帳戶名稱。

   * storage\_account\_key：儲存體帳戶的金鑰。

   * storage\_container\_name：您想要限制存取的儲存體帳戶中的容器。

   * example\_file\_path：上傳至容器之檔案的路徑。

2. 執行指令碼。 指令碼完成時，它會顯示類似下列文字的 SAS 權杖：

        sr=c&si=policyname&sig=dOAi8CXuz5Fm15EjRUu5dHlOzYNtcK3Afp1xqxniEps%3D&sv=2014-02-14

    儲存 SAS 原則權杖、儲存體帳戶名稱和容器名稱。 將儲存體帳戶與您的 HDInsight 叢集相關聯時，會使用這些值。

## <a name="use-the-sas-with-hdinsight"></a>搭配 HDInsight 使用 SAS

在建立 HDInsight 叢集時，您必須指定主要儲存體帳戶，您可以選擇性地指定其他儲存體帳戶。 這兩種新增儲存體的方法都需要所使用的儲存體帳戶和容器的完整存取權。

若要使用共用存取簽章來限制對容器的存取，請將自訂項目新增至叢集的 [核心網站] 組態。

* 對於**以 Windows 為基礎**或**以 Linux 為基礎**的 HDInsight 叢集，您可以使用 PowerShell 在建立叢集期間新增該項目。
* 對於**以 Linux 為基礎**的 HDInsight 叢集，在建立叢集之後使用 Ambari 變更組態。

### <a name="create-a-cluster-that-uses-the-sas"></a>建立使用 SAS 的叢集

建立使用 SAS 的 HDInsight 叢集的範例包含在儲存機制的 `CreateCluster` 目錄。 若要使用它，請使用下列步驟：

1. 在文字編輯器中開啟 `CreateCluster\HDInsightSAS.ps1` 檔案，並在文件開頭修改下列值。

    ```powershell
    # Replace 'mycluster' with the name of the cluster to be created
    $clusterName = 'mycluster'
    # Valid values are 'Linux' and 'Windows'
    $osType = 'Linux'
    # Replace 'myresourcegroup' with the name of the group to be created
    $resourceGroupName = 'myresourcegroup'
    # Replace with the Azure data center you want to the cluster to live in
    $location = 'North Europe'
    # Replace with the name of the default storage account to be created
    $defaultStorageAccountName = 'mystorageaccount'
    # Replace with the name of the SAS container created earlier
    $SASContainerName = 'sascontainer'
    # Replace with the name of the SAS storage account created earlier
    $SASStorageAccountName = 'sasaccount'
    # Replace with the SAS token generated earlier
    $SASToken = 'sastoken'
    # Set the number of worker nodes in the cluster
    $clusterSizeInNodes = 3
    ```

    例如，將 `'mycluster'` 變更為您想要建立的叢集的名稱。 建立儲存體帳戶和 SAS 權杖時，SAS 值應該符合先前步驟中的值。

    一旦您變更值，請儲存檔案。

2. 開啟新的 Azure PowerShell 提示字元。 如果您不熟悉或尚未安裝 Azure PowerShell，請參閱[安裝和設定 Azure PowerShell][powershell]。

1. 從提示字元使用下列命令來驗證您的 Azure 訂用帳戶：

    ```powershell
    Connect-AzureRmAccount
    ```

    出現提示時，請以適用於您 Azure 訂用帳戶的帳戶登入。

    如果您的帳戶會與多個 Azure 訂用帳戶相關聯，您可能需要使用 `Select-AzureRmSubscription` 來選取您想要使用的訂用帳戶。

4. 從提示字元中，將目錄變更至包含 HDInsightSAS.ps1 檔案的 `CreateCluster` 目錄。 然後使用下列命令以執行指令碼

    ```powershell
    .\HDInsightSAS.ps1
    ```

    當指令碼執行時，它會在建立資源群組和儲存體帳戶時，將輸出記錄到 PowerShell 命令提示字元。 系統會提示您輸入 HDInsight 叢集的 HTTP 使用者。 這是用來保護叢集的 HTTP/s 存取的帳戶。

    如果您是建立以 Linux 為基礎的叢集，系統會提示您輸入 SSH 使用者帳戶名稱和密碼。 此帳戶是用來從遠端登入到叢集。

   > [!IMPORTANT]
   > 出現 HTTP/s 或 SSH 使用者名稱和密碼提示時，您必須提供符合下列準則的密碼：
   >
   > * 長度必須小於 10 個字元
   > * 必須包含至少一個數字
   > * 必須包含至少一個非英數字元
   > * 必須包含至少一個大寫或小寫字元

需要一段時間讓此指令碼完成，通常大約是 15 分鐘。 指令碼完成且沒有發生任何錯誤時，叢集即已建立。

### <a name="use-the-sas-with-an-existing-cluster"></a>對現有的叢集使用 SAS

如果您有現有的以 Linux 為基礎的叢集，您可以使用下列步驟，將 SAS 新增至 [核心網站]  組態：

1. 開啟叢集的 Ambari Web UI。 此頁面的位址是 https://YOURCLUSTERNAME.azurehdinsight.net。 出現提示時，使用您建立叢集時所使用的 admin 名稱 (admin) 和密碼來驗證叢集。

2. 叢 Ambari Web UI 的左側，選擇 [HDFS]，然後選取頁面中間的 [設定] 索引標籤。

3. 選取 [進階] 索引標籤，然後再向下捲動直到您找到 [自訂核心網站] 區段。

4. 展開 [自訂核心網站] 區段，然後捲動至結尾，並且選取 [新增屬性...] 連結。 針對 [金鑰] 和 [值] 欄位使用下列值：

   * **金鑰**：fs.azure.sas.CONTAINERNAME.STORAGEACCOUNTNAME.blob.core.windows.net
   * **值**：您先前執行的 C# 或 Python 應用程式傳回的 SAS

     將 **CONTAINERNAME** 取代為您用於 C# 或 SAS 應用程式的容器名稱。 將 **STORAGEACCOUNTNAME** 取代為您使用的儲存體帳戶名稱。

5. 按一下 [新增] 按鈕以儲存這個金鑰和值，然後按一下 [儲存] 按鈕以儲存組態變更。 出現提示時，加入變更的描述 (例如，「新增 SAS 儲存體存取權」)，然後按一下 [儲存]。

    變更都完成時按一下 [確定]  。

   > [!IMPORTANT]
   > 您必須重新啟動數個服務，變更才會生效。

6. 在 Ambari Web UI 中，選取左側清單中的 [HDFS]，然後從右側 [服務動作] 下拉式清單中選取 [重新啟動所有受影響項目]。 出現提示時，選取 [確認全部重新啟動]。

    對 MapReduce2 和 YARN 重複此程序。

7. 這些項目重新啟動之後，選取每一個項目，並從 [服務動作] 下拉式清單停用維護模式。

## <a name="test-restricted-access"></a>測試限制的存取

若要確認您已限制存取，請使用下列方法：

* 對於 **以 Windows 為基礎** 的 HDInsight 叢集，使用遠端桌面連接到叢集。 如需詳細資訊，請參閱[使用 RDP 連線至 HDInsight](hdinsight-administer-use-management-portal.md#connect-to-clusters-using-rdp)。

    連線之後，請使用桌面上的 [Hadoop 命令列] 圖示來開啟命令提示字元。

* 對於 **以 Linux 為基礎** 的 HDInsight 叢集，使用 SSH 連接到叢集。 如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。

連接到叢集後，使用下列步驟確認您在 SAS 儲存體帳戶僅能讀取和列出項目：

1. 若要列出容器中的內容物，請使用提示中的下列命令： 

    ```bash
    hdfs dfs -ls wasb://SASCONTAINER@SASACCOUNTNAME.blob.core.windows.net/
    ```

    將 **SASCONTAINER** 取代為針對 SAS 儲存體帳戶建立的容器名稱。 將 **SASACCOUNTNAME** 取代為用於 SAS 的儲存體帳戶名稱。

    此清單包含容器與 SAS 建立時上傳的檔案。

2. 使用下列命令以確認您可以讀取檔案的內容。 如同前一個步驟取代 **SASCONTAINER** 和 **SASACCOUNTNAME**。 將 **FILENAME** 取代為前一個命令中顯示的名稱：

    ```bash
    hdfs dfs -text wasb://SASCONTAINER@SASACCOUNTNAME.blob.core.windows.net/FILENAME
    ```

    此命令會列出檔案的內容。

3. 使用下列命令將檔案下載到本機檔案系統：

    ```bash
    hdfs dfs -get wasb://SASCONTAINER@SASACCOUNTNAME.blob.core.windows.net/FILENAME testfile.txt
    ```

    此命令會將檔案下載到本機檔案，名為 **testfile.txt**。

4. 使用下列命令將本機檔案上傳至 SAS 儲存體上的新檔案，名為 **testupload.txt** ：

    ```bash
    hdfs dfs -put testfile.txt wasb://SASCONTAINER@SASACCOUNTNAME.blob.core.windows.net/testupload.txt
    ```

    您會收到類似以下文字的訊息：

        put: java.io.IOException

    因為儲存體位置僅限讀取+列出，所以會發生此錯誤。 使用下列命令將資料放在叢集的預設儲存體，它是可寫入的：

    ```bash
    hdfs dfs -put testfile.txt wasb:///testupload.txt
    ```

    此時，作業應該已順利完成。

## <a name="troubleshooting"></a>疑難排解

### <a name="a-task-was-canceled"></a>已取消工作

**徵兆**：當使用 PowerShell 指令碼建立叢集時，您可能會收到下列錯誤訊息：

    New-AzureRmHDInsightCluster : A task was canceled.
    At C:\Users\larryfr\Documents\GitHub\hdinsight-azure-storage-sas\CreateCluster\HDInsightSAS.ps1:62 char:5
    +     New-AzureRmHDInsightCluster `
    +     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        + CategoryInfo          : NotSpecified: (:) [New-AzureRmHDInsightCluster], CloudException
        + FullyQualifiedErrorId : Hyak.Common.CloudException,Microsoft.Azure.Commands.HDInsight.NewAzureHDInsightClusterCommand

**原因**：如果您使用用於叢集或 (對於以 Linux 為基礎的叢集) SSH 使用者的 admin/HTTP 密碼，則會發生此錯誤。

**解決方案**：使用符合下列準則的密碼：

* 長度必須小於 10 個字元
* 必須包含至少一個數字
* 必須包含至少一個非英數字元
* 必須包含至少一個大寫或小寫字元

## <a name="next-steps"></a>後續步驟

現在您已經了解如何將有限存取權儲存體新增至您的 HDInsight 叢集，了解在叢集上使用資料的其他方法：

* [〈搭配 HDInsight 使用 Hivet〉](hadoop/hdinsight-use-hive.md)
* [搭配 HDInsight 使用 Pig](hadoop/hdinsight-use-pig.md)
* [〈搭配 HDInsight 使用 MapReduce〉](hadoop/hdinsight-use-mapreduce.md)

[powershell]: /powershell/azureps-cmdlets-docs
