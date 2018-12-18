---
title: 利用一個 Azure Data Lake Store 帳戶使用多個 HDInsight 叢集 - Azure
description: 了解如何利用單一 Data Lake Store 帳戶來使用多個 HDInsight 叢集
keywords: hdinsight 儲存體,hdfs,結構化資料,非結構化資料, data lake store
services: hdinsight,storage
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 02/21/2018
ms.author: jasonh
ms.openlocfilehash: 0e74406d79484483053ca7d4b89b096c3f70e298
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43095119"
---
# <a name="use-multiple-hdinsight-clusters-with-an-azure-data-lake-store-account"></a>利用一個 Azure Data Lake Store 帳戶使用多個 HDInsight 叢集

從 HDInsight 3.5 版開始，您可以使用 Azure Data Lake Store 帳戶建立 HDInsight 叢集，作為預設檔案系統。
Data Lake Store 支援無限制的儲存空間，不僅適合裝載大量資料，也適合裝載多個共用單一 Data Lake Store 帳戶的 HDInsight 叢集。 如需有關如何使用 Data Lake Storage 建立 HDInsight 叢集作為儲存體的指示，請參閱[快速入門：在 HDInsight 中設定叢集](../storage/data-lake-storage/quickstart-create-connect-hdi-cluster.md)。

本文提供建議給 Data Lake Store 管理員來設定單一共用的 Data Lake Store 帳戶，此帳戶可以跨多個**作用中的** HDInsight 叢集使用。 這些建議適用於將多個安全和不安全的 Hadoop 叢集裝載於共用的 Data Lake Store 帳戶上。


## <a name="data-lake-store-file-and-folder-level-acls"></a>Data Lake Store 檔案和資料夾等級 ACL

本文的其餘部分假設您了解 Azure Data Lake Store 中的檔案和資料夾等級 ACL，這在 [Azure Data Lake Store 中的存取控制](../data-lake-store/data-lake-store-access-control.md)中有詳細的說明。

## <a name="data-lake-store-setup-for-multiple-hdinsight-clusters"></a>多個 HDInsight 叢集的 Data Lake Store 設定
讓我們以兩層的資料夾階層為例子，說明利用一個 Data Lake Store 帳戶來使用多個 HDInsight 叢集時的建議。 假設您具有 Data Lake Store 帳戶，而資料夾結構為 **/clusters/finance**。 在此結構中，財務組織所需的所有叢集都可以使用 /clusters/finance 作為儲存位置。 未來，如果有另一個組織 (假設是「行銷」) 需要使用同一個 Data Lake Store 帳戶建立 HDInsight 叢集，他們將會建立 /clusters/marketing。 現在，我們只要使用 **/clusters/finance** 即可。

為了讓 HDInsight 叢集有效利用此資料夾結構，Data Lake Store 管理員必須指派適當的權限，如下表所述。 表格中顯示的權限對應至「存取 ACL」，而不是「預設 ACL」。 


|資料夾  |權限  |擁有使用者  |擁有群組  | 具名使用者 | 具名使用者權限 | 具名群組 | 具名群組權限 |
|---------|---------|---------|---------|---------|---------|---------|---------|
|/ | rwxr-x--x  |admin |admin  |服務主體 |--x  |FINGRP   |r-x         |
|/clusters | rwxr-x--x |admin |admin |服務主體 |--x  |FINGRP |r-x         |
|/clusters/finance | rwxr-x--t |admin |FINGRP  |服務主體 |rwx  |-  |-     |

在表格中，

- **admin** 是 Data Lake Store 帳戶的建立者和管理員。
- **服務主體**是與帳戶相關聯的 Azure Active Directory (AAD) 服務主體。
- **FINGRP** 是 AAD 中建立的使用者群組，內含來自財務組織的使用者。

如需有關如何建立 AAD 應用程式 (這也會建立服務主體) 的指示，請參閱[建立 AAD 應用程式](../azure-resource-manager/resource-group-create-service-principal-portal.md#create-an-azure-active-directory-application)。 如需有關如何在 AAD 中建立使用者群組的指示，請參閱[在 Azure Active Directory 中管理群組](../active-directory/fundamentals/active-directory-groups-create-azure-portal.md)。

需要考慮的一些重要事項。

- 在叢集上使用儲存體帳戶**之前**，Data Lake Store 管理員必須以適當權限建立和佈建兩層的資料夾結構 (**/clusters/finance/**)。 建立叢集時不會自動建立此結構。
- 上述範例建議將擁有群組 **/clusters/finance** 設為 **FINGRP**，並允許 **r-x** 讓 FINGRP 從根目錄開始存取整個資料夾階層。 這可確保 FINGRP 的成員可以從根目錄開始瀏覽資料夾結構。
- 當不同的 AAD 服務主體可以在 **/clusters/finance** 下建立叢集時，黏著位元 (在 **finance** 資料夾上設定時) 可確保一個服務主體所建立的資料夾無法被其他服務主體刪除。
- 當資料夾結構和權限都備妥後，HDInsight 叢集建立程序就會在 **/clusters/finance/** 下建立特定的儲存位置。 例如，名稱為 fincluster01 之叢集的儲存體可能是 **/clusters/finance/fincluster01**。 下表顯示 HDInsight 叢集所建立之資料夾的擁有權和權限。

    |資料夾  |權限  |擁有使用者  |擁有群組  | 具名使用者 | 具名使用者權限 | 具名群組 | 具名群組權限 |
    |---------|---------|---------|---------|---------|---------|---------|---------|
    |/clusters/finanace/ fincluster01 | rwxr-x---  |服務主體 |FINGRP  |- |-  |-   |-  | 
   


## <a name="recommendations-for-job-input-and-output-data"></a>作業輸入和輸出資料的相關建議

我們建議將作業的輸入資料和作業的輸出儲存在 **/clusters** 以外的資料夾。 這可確保即使刪除叢集特定的資料夾以回收部分儲存空間，作業輸入和輸出仍然可供未來使用。 在這種情況下，請確定用來儲存作業輸入和輸出的資料夾層級，有提供適當的存取等級給服務主體。

## <a name="limit-on-clusters-sharing-a-single-storage-account"></a>共用單一儲存體帳戶的叢集所受的限制

可共用單一 Data Lake Store 帳戶的叢集數目上限，取決於這些叢集上執行的工作負載。 如果共用儲存體帳戶的叢集太多，或工作負載非常繁重，可能會導致對儲存體帳戶的輸入/輸出進行節流。

## <a name="support-for-default-acls"></a>預設 ACL 的支援

使用具名使用者存取權建立服務主體時 (如上表所示)，建議**不要**使用預設 ACL 來新增使用者。 如果使用預設 ACL 來佈建具名使用者存取權，將會導致將 770 權限指派給擁有使用者、擁有群組和其他人。 雖然此預設值 770 不會取走擁有使用者 (7) 或擁有群組 (7) 的權限，但會取走其他人 (0) 的所有權限。 這會在一個特定使用案例中產生已知問題，[已知問題和因應措施](#known-issues-and-workarounds)一節對此有詳細的討論。

## <a name="known-issues-and-workarounds"></a>已知問題和因應措施

本節列出搭配 Data Lake Store 使用 HDInsight 的已知問題及其因應措施。

### <a name="publicly-visible-localized-yarn-resources"></a>公開可見的當地語系化 YARN 資源

建立新的 Azure Data Lake Store 帳戶時，將會自動佈建根目錄並將存取 ACL 權限位元設為 770。 根資料夾的擁有使用者會設為帳戶建立使用者 (Data Lake Store 管理員)，而擁有群組會設為帳戶建立使用者的主要群組。 不會提供存取權給「其他人」。

已知這些設定會影響 [YARN 247](https://hwxmonarch.atlassian.net/browse/YARN-247) 中記錄的一個特定 HDInsight 使用案例。 作業提交會失敗，錯誤訊息如下︰

    Resource XXXX is not publicly accessible and as such cannot be part of the public cache.

如先前連結的 YARN JIRA 中所述，將公用資源當地語系化時，當地語系化人員會在遠端檔案系統上檢查所有要求之資源的權限，確保它們確實已公開。 任何不符合該條件的 LocalResource 將會被拒絕進行當地語系化。 權限檢查包括「其他人」的檔案讀取權限。 在 Azure Data Lake 上裝載 HDInsight 叢集時，這種情況不會自動發生，因為 Azure Data Lake 會在根資料夾等級拒絕「其他人」的所有存取權。

#### <a name="workaround"></a>因應措施
透過階層設定**其他人**的讀取和執行權限，例如在**/**、**/clusters** 和 **/clusters/finance**，如上表所示。

## <a name="see-also"></a>另請參閱

* [快速入門：在 HDInsight 中設定叢集](../storage/data-lake-storage/quickstart-create-connect-hdi-cluster.md)


