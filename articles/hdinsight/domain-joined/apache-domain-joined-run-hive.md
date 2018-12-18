---
title: 在具有企業安全性套件的 HDInsight 中設定 Hive 原則 - Azure
description: 了解如何在具有企業安全性套件的 Azure HDInsight 服務中設定 Hive 的 Apache Ranger 原則。
services: hdinsight
ms.service: hdinsight
author: omidm1
ms.author: omidm
ms.reviewer: mamccrea
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: bdad0bac0d320b641359df4093ae10c9505b7de2
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46993420"
---
# <a name="configure-hive-policies-in-hdinsight-with-enterprise-security-package"></a>在具有企業安全性套件的 HDInsight 中設定 Hive 原則
了解如何針對 Hive 設定 Apache Ranger 原則。 在本文中，您會建立兩個 Ranger 原則來限制 hivesampletable 的存取權。 HDInsight 叢集隨附 hivesampletable。 設定原則之後，您可以使用 Excel 和 ODBC 驅動程式連接到 HDInsight 中的 Hive 資料表。

## <a name="prerequisites"></a>必要條件
* 具有企業安全性套件的 HDInsight 叢集。 請參閱[設定具有 ESP 的 HDInsight 叢集](apache-domain-joined-configure.md)。
* 安裝 Office 2016、Office 2013 Professional Plus、Office 365 Pro Plus、Excel 2013 Standalone 或 Office 2010 Professional Plus 的工作站。

## <a name="connect-to-apache-ranger-admin-ui"></a>連線到 Apache Ranger 系統管理 UI
**到 Ranger 系統管理 UI**

1. 從瀏覽器連線到 Ranger 系統管理 UI。 URL 是 https://&lt;ClusterName>.azurehdinsight.net/Ranger/。

   > [!NOTE]
   > Ranger 會使用與 Hadoop 叢集不同的認證。 若要避免瀏覽器使用快取的 Hadoop 認證，請使用新的 InPrivate 瀏覽器視窗連線至 Ranger 系統管理 UI。
   >
   >
2. 使用叢集系統管理員網域使用者名稱和密碼進行登入：

    ![HDInsight ESP Ranger 首頁](./media/apache-domain-joined-run-hive/hdinsight-domain-joined-ranger-home-page.png)

    目前，Ranger 僅適用於 Yarn 和 Hive。

## <a name="create-domain-users"></a>建立網域使用者
如需如何建立 hiveruser1 與 hiveuser2 的相關資訊，請參閱[建立具有 ESP 的 HDInsight 叢集](apache-domain-joined-configure-using-azure-adds.md#create-a-hdinsight-cluster-with-esp)。 在本教學課程中，您會使用兩個使用者帳戶。

## <a name="create-ranger-policies"></a>建立 Ranger 原則
在這一節中，您會建立兩個 Ranger 原則以供存取 hivesampletable。 您會提供不同資料行集的選取權限。 兩個使用者都是使用[建立具有 ESP 的 HDInsight 叢集](apache-domain-joined-configure-using-azure-adds.md#create-a-hdinsight-cluster-with-esp)中的指示建立。 在下一節中，您將在 Excel 中測試這兩個原則。

**建立 Ranger 原則**

1. 開啟 Ranger 系統管理 UI。 請參閱[連接到 Apache Ranger 系統管理 UI](#connect-to-apache-ranager-admin-ui)。
2. 按一下 [Hive] 下的 **&lt;ClusterName>_hive**。 您會看到兩個預先設定的原則。
3. 按一下 [新增原則]，然後輸入下列值︰

   * 原則名稱︰read-hivesampletable-all
   * Hive 資料庫︰預設值
   * 資料表：hivesampletable
   * Hive 資料欄：*
   * 選取使用者：hiveuser1
   * 權限：選取

     ![HDInsight ESP Ranger Hive 原則設定](./media/apache-domain-joined-run-hive/hdinsight-domain-joined-configure-ranger-policy.png).

     > [!NOTE]
     > 如果 [選取使用者] 中未填入網域使用者，請稍等一下讓 Ranger 與 AAD 同步處理。
     >
     >
4. 按一下 [新增] 以儲存規則。
5. 重複最後兩個步驟，使用下列屬性建立另一個原則︰

   * 原則名稱︰read-hivesampletable-devicemake
   * Hive 資料庫︰預設值
   * 資料表：hivesampletable
   * Hive 資料行︰clientid、devicemake
   * 選取使用者：hiveuser2
   * 權限：選取

## <a name="create-hive-odbc-data-source"></a>建立 Hive ODBC 資料來源
在[建立 Hive ODBC 資料來源](../hadoop/apache-hadoop-connect-excel-hive-odbc-driver.md)中可找到相關指示。  

 | 屬性  |說明 |
 | --- | --- |
 | 資料來源名稱 | 為資料來源指定名稱 |
 | Host | 輸入 &lt;HDInsightClusterName>.azurehdinsight.net。 例如，myHDICluster.azurehdinsight.net |
 | Port | 使用 **443** (此連接埠已從 563 變更為 443)。 |
 | 資料庫 | 使用**預設值** |
 | Hive 伺服器類型 | 選取 [Hive Server 2] |
 | 機制 | 選取 [Azure HDInsight 服務] |
 | HTTP 路徑 | 保留為空白。 |
 | 使用者名稱 | 輸入 hiveuser1@contoso158.onmicrosoft.com。 更新網域名稱 (如果不同的話)。 |
 | 密碼 | 輸入 hiveuser1 的密碼。 |

請務必先按一下 [測試]，再儲存資料來源。

## <a name="import-data-into-excel-from-hdinsight"></a>從 HDInsight 將資料匯入 Excel 中
在上一節中，您已設定兩個原則。  hiveuser1 具有所有資料行的選取權限，而 hiveuser2 具有兩個資料行的選取權限。 本節中，您可以模擬兩位使用者將資料匯入 Excel 中。

1. 在 Excel 中開啟新的或現有的活頁簿。
2. 在 [資料] 索引標籤上按一下 [從其他資料來源]，然後按一下 [從資料連接精靈]，以啟動 [資料連接精靈]。

    ![開啟資料連接精靈][img-hdi-simbahiveodbc.excel.dataconnection]
3. 選取 [ODBC DSN] 作為資料來源，然後按 [下一步]。
4. 從 ODBC 資料來源中，選取您在上一個步驟中建立的資料來源名稱，然後按 [下一步] 。
5. 在精靈中重新輸入叢集的密碼，然後按一下 [確定]。 等待 [選取資料庫及資料表]  對話方塊開啟。 這可能需要幾秒鐘的時間。
6. 選取 **hivesampletable**，然後按 [下一步]。
7. 按一下 [完成]。
8. 在 [匯入資料] 對話方塊中，您可以變更或指定查詢。 若要執行此動作，請按一下 [屬性]。 這可能需要幾秒鐘的時間。
9. 按一下 [定義] 索引標籤。命令文字如下：

       SELECT * FROM "HIVE"."default"."hivesampletable"

   您定義的 Ranger 原則，hiveuser1 會有所有資料行的選取權限。  所以此查詢適用於 hiveuser1 的認證，但此查詢不適用於 hiveuser2 的認證。

   ![連接屬性][img-hdi-simbahiveodbc-excel-connectionproperties]
10. 按一下 [確定]  以關閉 [連接屬性] 對話方塊。
11. 按一下 [確定] 以關閉 [匯入資料] 對話方塊。  
12. 重新輸入 hiveuser1 的密碼，然後按一下 [確定]。 經過數秒後，資料即會匯入至 Excel。 完成時，您會看到 11 個資料行的資料。

測試您在上一節中建立的第二個原則 (read-hivesampletable-devicemake)

1. 在 Excel 中新增工作表。
2. 依照上一個程序匯入資料。  您會進行的唯一變更是使用 hiveuser2 的認證，而非使用 hiveuser1 的認證。 這會失敗，因為 hiveuser2 只有查看兩個資料行的權限。 您會收到下列錯誤︰

        [Microsoft][HiveODBC] (35) Error from Hive: error code: '40000' error message: 'Error while compiling statement: FAILED: HiveAccessControlException Permission denied: user [hiveuser2] does not have [SELECT] privilege on [default/hivesampletable/clientid,country ...]'.
3. 依照相同程序匯入資料。 這次使用 hiveuser2 的認證，並且修改 select 陳述式，從︰

        SELECT * FROM "HIVE"."default"."hivesampletable"

    至：

        SELECT clientid, devicemake FROM "HIVE"."default"."hivesampletable"

    完成時，您會看到已匯入 2 個資料行的資料。

## <a name="next-steps"></a>後續步驟
* 如需設定具有企業安全性套件的 HDInsight 叢集，請參閱[設定具有 ESP 的 HDInsight 叢集](apache-domain-joined-configure.md)。
* 如需管理具有 ESP 的 HDInsight 叢集，請參閱[管理具有 ESP 的 HDInsight 叢集](apache-domain-joined-manage.md)。
* 如需在具有 ESP 的 HDInsight 叢集上使用 SSH 執行 Hive 查詢，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md#domainjoined)。
* 如需使用 Hive JDBC 連接 Hive，請參閱[使用 Hive JDBC 驅動程式連接到 Azure HDInsight 上的 Hive](../hadoop/apache-hadoop-connect-hive-jdbc-driver.md)
* 如需使用 Hive ODBC 將 Excel 連接到 Hadoop，請參閱[使用 Microsoft Hive ODBC 驅動程式將 Excel 連接到 Hadoop](../hadoop/apache-hadoop-connect-excel-hive-odbc-driver.md)
* 如需使用 Power Query 將 Excel 連接到 Hadoop，請參閱[使用 Power Query 將 Excel 連接到 Hadoop](../hadoop/apache-hadoop-connect-excel-power-query.md)
