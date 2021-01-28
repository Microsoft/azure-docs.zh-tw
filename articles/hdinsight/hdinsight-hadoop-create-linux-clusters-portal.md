---
title: 使用網頁瀏覽器建立 Apache Hadoop 叢集，Azure HDInsight
description: 瞭解如何在 HDInsight 上建立 Apache Hadoop、Apache HBase、Apache Storm 或 Apache Spark 叢集。 使用網頁瀏覽器和 Azure 入口網站。
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 08/06/2020
ms.openlocfilehash: c68d342cf21d69fa97ba3d5171ba596662fd845f
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98945814"
---
# <a name="create-linux-based-clusters-in-hdinsight-by-using-the-azure-portal"></a>使用 Azure 入口網站在 HDInsight 中建立 Linux 型叢集

[!INCLUDE [selector](../../includes/hdinsight-create-linux-cluster-selector.md)]

Azure 入口網站是 Web 架構的管理工具，可用來管理裝載於 Microsoft Azure 雲端中的服務和資源。 在本文中，您會了解如何使用入口網站來建立 Linux 型 Azure HDInsight 叢集。 您可以從 [建立 HDInsight](./hdinsight-hadoop-provision-linux-clusters.md)叢集取得其他詳細資料。

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

Azure 入口網站會公開大部分的叢集屬性。 藉由使用 Azure Resource Manager 範本，您可以隱藏許多詳細資料。 如需詳細資訊，請參閱[使用 Resource Manager 範本在 HDInsight 中建立 Apache Hadoop 叢集](hdinsight-hadoop-create-linux-clusters-arm-templates.md)。

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="create-clusters"></a>建立叢集

[!INCLUDE [secure-transfer-enabled-storage-account](../../includes/hdinsight-secure-transfer.md)]

1. 登入 [Azure 入口網站](https://portal.azure.com)。

1. 在頂端功能表中，選取 [+ 建立資源]  。

    ![在 Azure 入口網站中建立新的叢集](./media/hdinsight-hadoop-create-linux-clusters-portal/azure-portal-create-resource.png "在 Azure 入口網站中建立新的叢集")

1. 選取 [分析]   > [Azure HDInsight]  以移至 [建立 HDInsight 叢集]  頁面。

## <a name="basics"></a>基本

![HDInsight 建立叢集基本概念](./media/hdinsight-hadoop-create-linux-clusters-portal/azure-portal-cluster-basics.png "在 Azure 入口網站中建立新的叢集")

在 [基本資料]  索引標籤中提供下列資訊：

|屬性 |描述 |
|---|---|
|訂用帳戶|從下拉式清單中，選取用於此叢集的 Azure 訂用帳戶。|
|資源群組|從下拉式清單中選取現有資源群組，或選取 [新建]  。|
|叢集名稱|輸入全域唯一名稱。|
|區域|從下拉式清單中，選取要在其中建立叢集的區域。|
|叢集類型|按一下 [ **選取叢集類型** ] 以開啟清單。 從清單中選取想要的叢集類型。 HDInsight 叢集有不同的類型。 它們與調整叢集時所針對的工作負載或技術對應。 目前沒有任何支援的方法可建立結合多個類型的叢集。|
|版本|從下拉式清單中選取 [版本]  。 如果您不知道要選擇哪一個項目，請使用預設版本。 如需詳細資訊，請參閱 [HDInsight 叢集版本](hdinsight-component-versioning.md)。|
|叢集登入使用者名稱|提供使用者名稱，預設值為 **admin**。|
|叢集登入密碼|請提供密碼。|
|確認叢集登入密碼|重新輸入密碼|
|安全殼層 (SSH) 使用者名稱|提供使用者名稱，預設值為 **sshuser**|
|將叢集登入密碼用於 SSH|如果您想要的 SSH 密碼與您先前指定的管理員密碼相同，請選取 [ **使用 ssh 的叢集登入密碼** ] 核取方塊。 如果不要，請提供 [密碼] 或 [公開金鑰]來驗證 SSH 使用者。 我們建議使用公開金鑰。 選擇底部的 [選取] 以儲存認證設定。  如需詳細資訊，請參閱[使用 SSH 連線到 HDInsight (Apache Hadoop)](hdinsight-hadoop-linux-use-ssh-unix.md)。|

選取 **[下一步：儲存體 >>** ] 前進至下一個索引標籤。

## <a name="storage"></a>儲存體

> [!WARNING] 
> 自2020年6月15日起，客戶將無法使用 HDInsight 建立新的服務主體。 請參閱使用 Azure Active Directory [建立服務主體和憑證](../active-directory/develop/howto-create-service-principal-portal.md) 。

![HDInsight 建立叢集儲存體](./media/hdinsight-hadoop-create-linux-clusters-portal/azure-portal-cluster-storage.png "在 Azure 入口網站儲存體中建立新的叢集")

### <a name="primary-storage"></a>主要儲存體

從 **主要儲存體類型** 的下拉式清單中，選取您的預設儲存體類型。 稍後要完成的欄位會根據您的選取專案而有所不同。 針對 **Azure 儲存體**：

1. 針對 [ **選取方法**]，選擇 [ **從清單選取**] 或 [ **使用存取金鑰**]。
    * 針對 [ **從清單選取**]，然後從下拉式清單中選取您的 **主要儲存體帳戶** ，或選取 [ **建立新** 的]。
    * 在 [ **使用存取金鑰**] 中，輸入您的 **儲存體帳戶名稱**。 然後提供 **存取金鑰**。

1. 針對 [ **容器**]，接受預設值，或輸入一個新值。

### <a name="additional-azure-storage"></a>其他 Azure 儲存體

選擇性：選取 [新增其他叢集儲存 **Azure 儲存體** ]。 不支援在與 HDInsight 叢集不同的區域中使用額外的儲存體帳戶。

### <a name="metastore-settings"></a>中繼存放區設定

選擇性：指定現有的 SQL Database，以將 Apache Hive、Apache Oozie 和或 Apache Ambari 中繼資料儲存在叢集之外。 用於中繼存放區的 Azure SQL Database 必須允許連線至其他 Azure 服務，包括 Azure HDInsight。 當您建立中繼存放區時，請勿使用包含破折號或連字號的資料庫名稱。 這些字元可能會導致叢集建立程序失敗。

> [!IMPORTANT]
> 針對支援中繼存放區的叢集圖形，預設中繼存放區會提供具有 **基本層 5 DTU 限制的 Azure SQL Database， (無法升級)**！ 適用于基本測試用途。 針對大型或生產工作負載，建議您遷移至外部中繼存放區。

選取 **[下一步]： [安全性 + 網路] >>** 進入下一個索引標籤。

## <a name="security--networking"></a>安全性 + 網路

![HDInsight 建立叢集安全性網路](./media/hdinsight-hadoop-create-linux-clusters-portal/azure-portal-cluster-security-networking.png "HDInsight 建立叢集安全性網路")

在 [ **安全性 + 網路** ] 索引標籤中，提供下列資訊：

|屬性 |描述 |
|---|---|
|企業安全性套件|選擇性：選取要使用 **企業安全性套件** 的核取方塊。 如需詳細資訊，請參閱 [使用 Azure Active Directory Domain Services 設定具有企業安全性套件的 HDInsight](./domain-joined/apache-domain-joined-configure-using-azure-adds.md)叢集。|
|TLS|選擇性：從下拉式清單選取 TLS 版本。 如需詳細資訊，請參閱[傳輸層安全性](./transport-layer-security.md)。|
|虛擬網路|選擇性：從下拉式清單中選取現有的虛擬網路和子網。 如需詳細資訊，請參閱 [規劃 Azure HDInsight 叢集的虛擬網路部署](hdinsight-plan-virtual-network-deployment.md)。 此文章包含虛擬網路的特定設定需求。|
|磁片加密設定|選用：選取核取方塊以使用加密。 如需詳細資訊，請參閱[客戶管理的金鑰磁碟加密](./disk-encryption.md)。|
|Kafka REST Proxy|這項設定僅適用于叢集類型 Kafka。 如需詳細資訊，請參閱 [使用 REST proxy](./kafka/rest-proxy.md)。|
|身分識別|選擇性：從下拉式清單中選取現有使用者指派的服務識別。 如需詳細資訊，請參閱 [Azure HDInsight 中的受控](./hdinsight-managed-identities.md)識別。|

選取 **[下一步：設定 + 定價] >>** 前進到下一個索引標籤。

## <a name="configuration--pricing"></a>設定 + 定價

![HDInsight 建立叢集設定](./media/hdinsight-hadoop-create-linux-clusters-portal/azure-portal-cluster-configuration.png "設定與定價索引標籤")

從 [設定 **+ 定價** ] 索引標籤中，提供下列資訊：

|屬性 |描述 |
|---|---|
|+ 新增應用程式|選用：選取您想要的任何應用程式。 Microsoft、獨立軟體廠商 (ISV) 或您可以開發這些應用程式。 如需詳細資訊，請參閱[在叢集建立期間安裝應用程式](hdinsight-apps-install-applications.md#install-applications-during-cluster-creation)。|
|節點大小|選擇性：選取不同大小的節點。|
|節點數目|選用：輸入指定節點類型的節點數目。 如果您計畫超過32個背景工作節點，請選取具有至少8個核心和 14 GB RAM 的前端節點大小。 請在建立叢集時規劃節點，或在建立後透過調整叢集規模的方式進行規劃。|
|啟用自動調整|選擇性：選取核取方塊以啟用此功能。 如需詳細資訊，請參閱 [Azure HDInsight 叢集自動調整規模](./hdinsight-autoscale-clusters.md)。|
|+ 新增腳本動作|選擇性：如果您想要在建立叢集時使用自訂腳本來自訂叢集，這個選項就可以使用。 如需有關指令碼動作的詳細資訊，請參閱[使用指令碼動作來自訂 Linux 型 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)。|

選取 [ **審核 + 建立 >>** ] 以驗證叢集設定，並前進到最後一個索引標籤。

## <a name="review--create"></a>檢閱 + 建立

![HDInsight 建立叢集摘要](./media/hdinsight-hadoop-create-linux-clusters-portal/azure-portal-cluster-review-create-hadoop.png "指定叢集節點的數目")

檢查設定。 選取 [建立] 以建立叢集。

建立叢集需要一些時間，通常約 20 分鐘左右。 監視 **通知** 以檢查佈建程序。

## <a name="post-creation"></a>建立後

在建立程序完成之後，從 [已成功部署] 通知中選取 [前往資源]。 叢集視窗會提供以下資訊。

![HDI Azure 入口網站叢集總覽](./media/hdinsight-hadoop-create-linux-clusters-portal/hdinsight-create-cluster-completed.png "叢集屬性")

視窗中的部分圖示會以下面方式說明：

|屬性 | 描述 |
|---|---|
|概觀|提供有關叢集的所有基本資訊。 例如名稱、其所屬資源群組、位置、作業系統，以及叢集儀表板的 URL。|
|叢集儀表板|將您導向至與叢集相關聯的 Ambari 入口網站。|
|SSH + 叢集登入|提供使用 SSH 存取叢集所需的資訊。|
|刪除|刪除 HDInsight 叢集。|

## <a name="delete-the-cluster"></a>選取叢集

請參閱 [使用您的瀏覽器、PowerShell 或 Azure CLI 來刪除 HDInsight](./hdinsight-delete-cluster.md)叢集。

## <a name="troubleshoot"></a>疑難排解

如果您在建立 HDInsight 叢集時遇到問題，請參閱[存取控制需求](./hdinsight-hadoop-customize-cluster-linux.md#access-control)。

## <a name="next-steps"></a>後續步驟

您已成功建立 HDInsight 叢集。 現在，請了解如何使用您的叢集。

* [搭配 HDInsight 使用 Apache Hive](hadoop/hdinsight-use-hive.md)
* [開始使用 HDInsight 上的 Apache HBase](hbase/apache-hbase-tutorial-get-started-linux.md)
* [使用指令碼動作來自訂 Linux 型 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)