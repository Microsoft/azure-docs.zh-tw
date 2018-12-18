---
title: 將 Azure Active Directory 使用者同步至叢集 - Azure HDInsight
description: 從 Azure Active Directory 將已驗證的使用者同步至叢集。
services: hdinsight
ms.service: hdinsight
author: ashishthaps
ms.author: ashishth
ms.reviewer: mamccrea
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: b63f2566220d556f9695687dc743a7d47e27acf1
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46984308"
---
# <a name="synchronize-azure-active-directory-users-to-an-hdinsight-cluster"></a>將 Azure Active Directory 使用者同步至 HDInsight 叢集

[有企業安全性套件 (ESP) 的 HDInsight 叢集](hdinsight-domain-joined-introduction.md)會搭配 Azure Active Directory (Azure AD) 使用者使用增強式驗證，也會使用*角色型存取控制* (RBAC) 原則。 當您將使用者和群組新增至 Azure AD 時，您可以將需要存取權的使用者同步至您的叢集。

## <a name="prerequisites"></a>必要條件

如果您尚未這樣做，請[建立有企業安全性套件的 HDInsight 叢集](hdinsight-domain-joined-configure.md)。

## <a name="add-new-azure-ad-users"></a>新增 Azure AD 使用者

若要檢視您的主機，請開啟 Ambari Web UI。 每個節點會以新的自動升級設定進行更新。

1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至與 ESP 叢集相關聯的 Azure AD 目錄。

2. 從左側功能表中選取 [所有使用者]，然後選取 [新增使用者]。

    ![所有使用者窗格](./media/hdinsight-sync-aad-users-to-cluster/aad-users.png)

3. 完成新增使用者表單。 選取您建立用於指派叢集型權限的群組。 在此範例中，建立名為 "HiveUsers" 的群組，您可以對其指派新使用者。 用於建立 ESP 叢集的[範例指示](hdinsight-domain-joined-configure.md)包含新增兩個群組，分別是 `HiveUsers` 和 `AAD DC Administrators`。

    ![新增使用者窗格](./media/hdinsight-sync-aad-users-to-cluster/aad-new-user.png)

4. 選取 [建立] 。

## <a name="use-the-ambari-rest-api-to-synchronize-users"></a>使用 Ambari REST API 以同步使用者

屆時會同步在叢集建立程序期間指定的使用者群組。 使用者同步會每小時自動進行一次。 若要立即同步使用者或同步在叢集建立期間指定之群組以外的群組，請使用 Ambari REST API。

下列方法會搭配 Ambari REST API 使用 POST。 如需詳細資訊，請參閱[使用 Ambari REST API 管理 HDInsight 叢集](hdinsight-hadoop-manage-ambari-rest-api.md)。

1. [使用 SSH 連線至您的叢集](hdinsight-hadoop-linux-use-ssh-unix.md)。 從 Azure 入口網站中叢集的 [概觀] 窗格，選取 [安全殼層 (SSH)] 按鈕。

    ![安全殼層 (SSH)](./media/hdinsight-sync-aad-users-to-cluster/ssh.png)

2. 複製顯示的 `ssh` 命令，並將它貼到您的 SSH 用戶端。 出現提示時，輸入 SSH 使用者密碼。

3. 驗證之後，輸入下列命令：

    ```bash
    curl -u admin:<YOUR PASSWORD> -sS -H "X-Requested-By: ambari" \
    -X POST -d '{"Event": {"specs": [{"principal_type": "groups", "sync_type": "existing"}]}}' \
    "https://<YOUR CLUSTER NAME>.azurehdinsight.net/api/v1/ldap_sync_events"
    ```
    
    回應應該如下所示：

    ```json
    {
      "resources" : [
        {
          "href" : "http://hn0-hadoop.<YOUR DOMAIN>.com:8080/api/v1/ldap_sync_events/1",
          "Event" : {
            "id" : 1
          }
        }
      ]
    }
    ```

4. 若要查看同步狀態，請執行新的 `curl` 命令：

    ```bash
    curl -u admin:<YOUR PASSWORD> https://<YOUR CLUSTER NAME>.azurehdinsight.net/api/v1/ldap_sync_events/1
    ```
    
    回應應該如下所示：
    
    ```json
    {
      "href" : "http://hn0-hadoop.YOURDOMAIN.com:8080/api/v1/ldap_sync_events/1",
      "Event" : {
        "id" : 1,
        "specs" : [
          {
            "sync_type" : "existing",
            "principal_type" : "groups"
          }
        ],
        "status" : "COMPLETE",
        "status_detail" : "Completed LDAP sync.",
        "summary" : {
          "groups" : {
            "created" : 0,
            "removed" : 0,
            "updated" : 0
          },
          "memberships" : {
            "created" : 1,
            "removed" : 0
          },
          "users" : {
            "created" : 1,
            "removed" : 0,
            "skipped" : 0,
            "updated" : 0
          }
        },
        "sync_time" : {
          "end" : 1497994072182,
          "start" : 1497994071100
        }
      }
    }
    ```

5. 此結果會顯示狀態是「完成」，已建立一個新的使用者，並且將成員資格指派給使用者。 在此範例中，會將使用者指派給 "HiveUsers" 已同步的 LDAP 群組，因為使用者已新增至 Azure AD 中的該相同群組。

> [!NOTE]
> 上一個方法只會同步在叢集建立期間於網域設定之 **Access user group** 屬性中指定的 Azure AD 群組。 如需詳細資訊，請參閱[建立 HDInsight 叢集](domain-joined/apache-domain-joined-configure.md)。

## <a name="verify-the-newly-added-azure-ad-user"></a>確認新增的 Azure AD 使用者

開啟 [Ambari Web UI](hdinsight-hadoop-manage-ambari.md) 以確認已新增新的 Azure AD 使用者。 藉由瀏覽至 **`https://<YOUR CLUSTER NAME>.azurehdinsight.net`** 來存取 Ambari Web UI。 輸入叢集系統管理員使用者名稱和密碼。

1. 從 Ambari 儀表板中，選取 [管理員] 功能表底下的 [管理 Ambari]。

    ![管理 Ambari](./media/hdinsight-sync-aad-users-to-cluster/manage-ambari.png)

2. 在分頁左側，選取 [使用者 + 群組管理] 功能表群組底下的 [使用者]。

    ![使用者功能表項目](./media/hdinsight-sync-aad-users-to-cluster/users-link.png)

3. 新的使用者應該會列在 [使用者] 資料表中。 類型設定為 `LDAP` 而不是 `Local`。

    ![使用者分頁](./media/hdinsight-sync-aad-users-to-cluster/users.png)

## <a name="log-in-to-ambari-as-the-new-user"></a>以新的使用者身分登入 Ambari

當新的使用者 (或任何其他網域使用者) 登入 Ambari 時，他們會使用其完整的 Azure AD 使用者名稱和網域認證。  Ambari 會顯示使用者別名，這是使用者在 Azure AD 中的顯示名稱。 新的範例使用者具有使用者名稱 `hiveuser3@contoso.com`。 在 Ambari 中，這個新的使用者會顯示為 `hiveuser3`，但是使用者是以 `hiveuser3@contoso.com` 身分登入 Ambari。

## <a name="see-also"></a>另請參閱

* [在有 ESP 的 HDInsight 中設定 Hive 原則](hdinsight-domain-joined-run-hive.md)
* [管理有 ESP 的 HDInsight 叢集](hdinsight-domain-joined-manage.md)
* [授權使用者存取 Ambari](hdinsight-authorize-users-to-ambari.md)
