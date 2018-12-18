---
title: 使用企業安全性套件管理 HDInsight 叢集 - Azure
description: 了解如何使用企業安全性套件管理 HDInsight 叢集。
services: hdinsight
ms.service: hdinsight
author: omidm1
ms.author: omidm
ms.reviewer: mamccrea
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 08/24/2018
ms.openlocfilehash: 02a77ef9589a42a6f33087ba7e22efc3144a8f2c
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46973553"
---
# <a name="manage-hdinsight-clusters-with-enterprise-security-package"></a>使用企業安全性套件管理 HDInsight 叢集
了解 HDInsight 企業安全性套件 (ESP) 中的使用者與角色，以及如何管理 ESP 叢集。

## <a name="use-vscode-to-link-to-domain-joined-cluster"></a>使用 VSCode 連結至已加入網域的叢集

您可以使用 Ambari 受控使用者名稱來連結正常的叢集，也可以使用網域使用者名稱 (例如：user1@contoso.com) 來連結安全性 hadoop 叢集。
1. 選取 **CTRL+SHIFT+P** 以開啟命令選擇區，然後輸入 **HDInsight: Link a cluster**。

   ![連結叢集命令](./media/apache-domain-joined-manage/link-cluster-command.png)

2. 輸入 HDInsight 叢集 URL -> 輸入使用者名稱 -> 輸入密碼 -> 選取叢集類型 -> 如果通過驗證，它會顯示成功資訊。
   
   ![連結叢集對話方塊](./media/apache-domain-joined-manage/link-cluster-process.png)

   > [!NOTE]
   > 如果叢集已登入 Azure 訂用帳戶並連結叢集，則會使用連結的使用者名稱和密碼。 
   
3. 您可以使用 **List cluster** 命令來查看連結的叢集。 您現在可以將指令碼提交至此連結的叢集。

   ![連結的叢集](./media/apache-domain-joined-manage/linked-cluster.png)

4. 您也可以從命令選擇區輸入 **HDInsight: Unlink a cluster** 以取消連結叢集。

## <a name="use-intellij-to-link-to-domain-joined-cluster"></a>使用 IntelliJ 連結至已加入網域的叢集

您可以使用 Ambari 受控使用者名稱來連結正常的叢集，也可以使用網域使用者名稱 (例如：user1@contoso.com) 來連結安全性 hadoop 叢集。 
1. 從 [Azure 總管] 中按一下 [連結叢集]。

   ![連結叢集操作功能表](./media/apache-domain-joined-manage/link-a-cluster-context-menu.png)

2. 輸入 [叢集名稱]、[使用者名稱] 和 [密碼]。 如果驗證失敗，您需要檢查使用者名稱和密碼。 您也可以選擇新增 [儲存體帳戶]、[儲存體金鑰]，然後從 [儲存體容器] 選取容器。 儲存體資訊適用於左側樹狀目錄中的儲存體總管
   
   ![連結叢集對話方塊](./media/apache-domain-joined-manage/link-a-cluster-dialog.png)

   > [!NOTE]
   > 如果叢集已登入 Azure 訂用帳戶並連結叢集，我們會使用連結的儲存體金鑰、使用者名稱和密碼。
   > ![IntelliJ 中的儲存體總管](./media/apache-domain-joined-manage/storage-explorer-in-IntelliJ.png)

   
3. 如果輸入資訊正確無誤，您可以在 [HDInsight] 節點中看見連結的叢集。 您現在可以將應用程式提交至此連結的叢集。

   ![連結的叢集](./media/apache-domain-joined-manage/linked-cluster-intellij.png)

4. 您也可以從 [Azure 總管] 取消連結叢集。
   
   ![取消連結的叢集](./media/apache-domain-joined-manage/unlink.png)

## <a name="use-eclipse-to-link-to-domain-joined-cluster"></a>使用 Eclipse 連結至已加入網域的叢集

您可以使用 Ambari 受控使用者名稱來連結正常的叢集，也可以使用網域使用者名稱 (例如：user1@contoso.com) 來連結安全性 hadoop 叢集。
1. 從 [Azure 總管] 中按一下 [連結叢集]。

   ![連結叢集操作功能表](./media/apache-domain-joined-manage/link-a-cluster-context-menu.png)

2. 輸入 [叢集名稱]、[使用者名稱] 及 [密碼]，然後按一下 [確定] 按鈕連結叢集。 您也可以選擇輸入 [儲存體帳戶]、[儲存體金鑰]，然後選取 [儲存體容器]，讓儲存體總管在左側樹狀檢視中工作
   
   ![連結叢集對話方塊](./media/apache-domain-joined-manage/link-cluster-dialog.png)
   
   > [!NOTE]
   > 如果叢集已登入 Azure 訂用帳戶並連結叢集，我們會使用連結的儲存體金鑰、使用者名稱和密碼。
   > ![Eclipse 中的儲存體總管](./media/apache-domain-joined-manage/storage-explorer-in-Eclipse.png)

3. 如果輸入資訊正確無誤，則按一下 [確定] 按鈕之後，您就可以在 [HDInsight] 節點中看見連結的叢集。 您現在可以將應用程式提交至此連結的叢集。

   ![連結的叢集](./media/apache-domain-joined-manage/linked-cluster-intellij.png)

4. 您也可以從 [Azure 總管] 取消連結叢集。
   
   ![取消連結的叢集](./media/apache-domain-joined-manage/unlink.png)

## <a name="access-the-clusters-with-enterprise-security-package"></a>以企業安全性套件存取叢集。

企業安全性套件 (先前稱為 HDInsight Premium) 提供對叢集的多位使用者存取，其驗證是由 Active Directory 執行，並且由 Apache Ranger 和 Storage ACL (ADLS ACL) 授權。 授權會提供多位使用者間的安全界限，只允許特殊權限的使用者根據授權原則存取資料。

安全性和使用者隔離對於使用企業安全性套件的 HDInsight 叢集很重要。 為了符合這些需求，會封鎖使用企業安全性套件之叢集的 SSH 存取。 下表顯示每個叢集類型的建議存取方法：

|工作負載|案例|存取方法|
|--------|--------|-------------|
|Hadoop|Hive – 互動式作業/查詢 |<ul><li>[Beeline](#beeline)</li><li>[Hive 檢視](../hadoop/apache-hadoop-use-hive-ambari-view.md)</li><li>[ODBC/JDBC – Power BI](../hadoop/apache-hadoop-connect-hive-power-bi.md)</li><li>[Visual Studio 工具](../hadoop/apache-hadoop-visual-studio-tools-get-started.md)</li></ul>|
|Spark|互動式作業/查詢，PySpark 互動式|<ul><li>[Beeline](#beeline)</li><li>[Zeppelin with Livy](../spark/apache-spark-zeppelin-notebook.md)</li><li>[Hive 檢視](../hadoop/apache-hadoop-use-hive-ambari-view.md)</li><li>[ODBC/JDBC – Power BI](../hadoop/apache-hadoop-connect-hive-power-bi.md)</li><li>[Visual Studio 工具](../hadoop/apache-hadoop-visual-studio-tools-get-started.md)</li></ul>|
|Spark|批次案例 – Spark 提交，PySpark|<ul><li>[Livy](../spark/apache-spark-livy-rest-interface.md)</li></ul>|
|互動式查詢 (LLAP)|互動式|<ul><li>[Beeline](#beeline)</li><li>[Hive 檢視](../hadoop/apache-hadoop-use-hive-ambari-view.md)</li><li>[ODBC/JDBC – Power BI](../hadoop/apache-hadoop-connect-hive-power-bi.md)</li><li>[Visual Studio 工具](../hadoop/apache-hadoop-visual-studio-tools-get-started.md)</li></ul>|
|任意|安裝自訂應用程式|<ul><li>[指令碼動作](../hdinsight-hadoop-customize-cluster-linux.md)</li></ul>|

   > [!NOTE]
   > 企業安全性套件不會安裝/支援 Jupyter。

從安全性觀點來說，使用標準 API 會有所幫助。 此外，還有下列優點：

1.  **管理** – 您可以使用標準 API (Livy、HS2 等等) 管理您的程式碼並自動化作業。
2.  **稽核** – 使用 SSH 無法稽核哪些使用者 SSH 到叢集。 當作業是透過標準端點建構時，作業會在使用者內容中執行，所以這不是問題。 



### <a name="beeline"></a>使用 BeeLine 
在您的機器上安裝 Beeline，並透過公用網際網路連線，使用下列參數： 

```
- Connection string: -u 'jdbc:hive2://<clustername>.azurehdinsight.net:443/;ssl=true;transportMode=http;httpPath=/hive2'
- Cluster login name: -n admin
- Cluster login password -p 'password'
```

若您有在本機上安裝 Beeline 並透過公用網際網路連線，請使用下列參數： 

```
- Connection string: -u 'jdbc:hive2://<headnode-FQDN>:10001/;transportMode=http'
```

若要找出前端節點的完整網域名稱，請利用＜使用 Ambari REST API 管理 HDInsight＞文件中的資訊。














## <a name="users-of-hdinsight-clusters-with-esp"></a>有 ESP 的 HDInsight 叢集的使用者
非 ESP 的 HDInsight 叢集有兩個使用者帳戶，是在叢集建立期間建立的︰

* **Ambari 系統管理員**：此帳戶亦稱為「Hadoop 使用者」或「HTTP 使用者」。 此帳戶可以用來登入 Ambari，網址是 https://&lt;clustername>.azurehdinsight.net。 它也可以用來在 Ambari 檢視上執行查詢、透過外部工具 (例如 PowerShell、Templeton、Visual Studio) 執行作業、以及使用 Hive ODBC 驅動程式和 BI 工具 (例如 Excel、PowerBI、或 Tableau) 進行驗證。

除了 Ambari 系統管理員之外，使用 ESP 的 HDInsight 叢集有三個新使用者。

* **Ranger 系統管理員**︰此帳戶是本機的 Apache Ranger 系統管理員帳戶。 不是 Active Directory 網域使用者。 此帳戶可以用來設定原則、將其他使用者設為系統管理員、或委派系統管理工作 (讓某些使用者可以管理原則)。 預設的使用者名稱是 admin，密碼則是與 Ambari 系統管理員的密碼相同。 可以在 Ranger 的 [設定] 頁面中更新密碼。
* **叢集系統管理員網域使用者**︰此帳戶是被指定為 Hadoop 叢集系統管理員 (包括 Ambari 和 Ranger) 的 Active Directory 網域使用者。 您必須在叢集建立期間提供此使用者的認證。 此使用者有下列權限：

  * 在叢集建立期間，將機器加入網域，並將它們放入您指定的 OU 中。
  * 在叢集建立期間，於您指定的 OU 中建立服務主體。
  * 建立反向 DNS 項目。

    請注意，其他的 AD 使用者也有這些權限。

    叢集內有些端點 (例如，Templeton) 不受 Ranger 管理，因此不安全。 這些端點會針對所有使用者鎖定，只有叢集系統管理員網域使用者除外。
* **一般**︰在叢集建立期間，您可以提供多個 Active Directory 群組。 這些群組中的使用者都將同步至 Ranger 和 Ambari。 這些使用者是網域使用者，只有存取 Ranger 管理之端點 (例如Hiveserver2) 的權限。 所有 RBAC 原則和稽核皆不適用於這些使用者。

## <a name="roles-of-hdinsight-clusters-with-esp"></a>使用 ESP 之 HDInsight 叢集的角色
HDInsight 企業安全性套件有下列角色：

* 叢集系統管理員
* 叢集操作員
* 服務管理員
* 服務操作員
* 叢集使用者

**若要查看這些角色的權限**

1. 開啟 Ambari 管理 UI。  請參閱[開啟 Ambari 管理 UI](#open-the-ambari-management-ui)。
2. 按一下左側功能表中的 [角色]。
3. 按一下藍色問號可查看權限︰

    ![ESP HDInsight 角色權限](./media/apache-domain-joined-manage/hdinsight-domain-joined-roles-permissions.png)

## <a name="open-the-ambari-management-ui"></a>開啟 Ambari 管理 UI

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 開啟您的 HDInsight 叢集。 請參閱[列出和顯示叢集](../hdinsight-administer-use-management-portal.md#list-and-show-clusters)。
3. 按一下頂端功能表中的 [儀表板]  開啟 Ambari。
4. 使用叢集系統管理員網域使用者的名稱和密碼登入 Ambari：
5. 按一下右上角的 [管理] 下拉式功能表，然後按一下 [管理 Ambari]。

    ![ESP HDInsight 管理 Ambari](./media/apache-domain-joined-manage/hdinsight-domain-joined-manage-ambari.png)

    UI 看起來像這樣：

    ![ESP HDInsight Ambari 管理 UI](./media/apache-domain-joined-manage/hdinsight-domain-joined-ambari-management-ui.png)

## <a name="list-the-domain-users-synchronized-from-your-active-directory"></a>列出從您的 Active Directory 同步處理的網域使用者
1. 開啟 Ambari 管理 UI。  請參閱[開啟 Ambari 管理 UI](#open-the-ambari-management-ui)。
2. 按一下左側功能表中的 [使用者] 。 您應該會看到從您的 Active Directory 同步至 HDInsight 叢集的所有使用者。

    ![ESP HDInsight Ambari 管理 UI 清單使用者](./media/apache-domain-joined-manage/hdinsight-domain-joined-ambari-management-ui-users.png)

## <a name="list-the-domain-groups-synchronized-from-your-active-directory"></a>列出從您的 Active Directory 同步處理的網域群組
1. 開啟 Ambari 管理 UI。  請參閱[開啟 Ambari 管理 UI](#open-the-ambari-management-ui)。
2. 按一下左側功能表中的 [群組]。 您應該會看到從您的 Active Directory 同步至 HDInsight 叢集的所有群組。

    ![ESP HDInsight Ambari 管理 UI 清單群組](./media/apache-domain-joined-manage/hdinsight-domain-joined-ambari-management-ui-groups.png)

## <a name="configure-hive-views-permissions"></a>設定 Hive 檢視權限
1. 開啟 Ambari 管理 UI。  請參閱[開啟 Ambari 管理 UI](#open-the-ambari-management-ui)。
2. 按一下左側功能表中的 [檢視]。
3. 按一下 [HIVE] 以顯示詳細資料。

    ![ESP HDInsight Ambari 管理 UI Hive 檢視](./media/apache-domain-joined-manage/hdinsight-domain-joined-ambari-management-ui-hive-views.png)
4. 按一下 [Hive 檢視] 連結以設定 Hive 檢視。
5. 向下捲動至 [權限]。

    ![ESP HDInsight Ambari 管理 UI Hive 檢視設定權限](./media/apache-domain-joined-manage/hdinsight-domain-joined-ambari-management-ui-hive-views-permissions.png)
6. 按一下 [新增使用者] 或 [新增群組]，然後指定可以使用 Hive 檢視的使用者或群組。

## <a name="configure-users-for-the-roles"></a>設定角色的使用者
 若要查看角色及其權限的清單，請參閱[使用 ESP 之 HDInsight 叢集的角色](#roles-of-domain---joined-hdinsight-clusters)。

1. 開啟 Ambari 管理 UI。  請參閱[開啟 Ambari 管理 UI](#open-the-ambari-management-ui)。
2. 按一下左側功能表中的 [角色]。
3. 按一下 [新增使用者] 或 [新增群組] 以將使用者或群組指定至不同角色。

## <a name="next-steps"></a>後續步驟
* 如需使用企業安全性套件設定 HDInsight 叢集，請參閱[使用 ESP 設定 HDInsight 叢集](apache-domain-joined-configure.md)。
* 如需設定 Hive 原則和執行 Hive 查詢，請參閱[使用 ESP 針對 HDInsight 叢集設定 Hive 原則](apache-domain-joined-run-hive.md)。
