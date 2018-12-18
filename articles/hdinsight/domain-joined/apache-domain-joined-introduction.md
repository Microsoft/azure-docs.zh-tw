---
title: 使用企業安全性套件維護 Hadoop 安全性的簡介
description: 了解企業安全性套件如何支援企業安全性的四大要件。
services: hdinsight
ms.service: hdinsight
author: omidm1
ms.author: omidm
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: cf94ba73d57763f5e05ed9b33d10bb335103400a
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46946386"
---
# <a name="an-introduction-to-hadoop-security-with-enterprise-security-package"></a>使用企業安全性套件維護 Hadoop 安全性的簡介

在過去，Azure HDInsight 僅支援單一使用者本機系統管理員。這很適合用於比較小型的應用程式團隊或部門。 Hadoop 型工作負載在企業間越來越受青睞，而 Active Directory 型驗證、多使用者支援和角色型存取控制等企業級功能的需求日益重要。 

您可以建立已加入 Active Directory 網域並具備企業安全性套件 (ESP) 的 HDInsight 叢集。 接著，您可以設定企業員工清單，指出可透過 Azure Active Directory 進行驗證以登入 HDInsight 叢集的員工。 沒有人能夠從企業外部登入或存取此 HDInsight 叢集。 

企業系統管理員可以使用 [Apache Ranger](http://hortonworks.com/apache/ranger/) \(英文\)，針對 Hive 安全性設定角色型存取控制 (RBAC)。 設定 RBAC 可限制資料存取只提供所需的資料。 最後，系統管理員可稽核由員工存取的資料，以及對存取控制原則所做的任何變更。 系統管理員因此便能高度控管公司的資源。

> [!NOTE]
> 本文所說明的新功能 (預覽版) 僅適用於下列叢集類型：Hadoop、Spark 及互動式查詢。 ESP 叢集現已啟用 Oozie。 若要存取 Oozie Web UI，使用者應該啟用[通道](../hdinsight-linux-ambari-ssh-tunnel.md)。

企業安全性包含四大要件：周邊安全性、驗證、授權和加密。

![企業安全性套件 HDInsight 叢集對企業安全性四大要件而言有哪些好處](./media/apache-domain-joined-introduction/hdinsight-domain-joined-four-pillars.png).

## <a name="perimeter-security"></a>周邊安全性
透過虛擬網路和 Azure VPN 閘道服務可達成 HDInsight 中的周邊安全性。 企業系統管理員可以在虛擬網路內部建立 ESP 叢集，並使用網路安全性群組 (防火牆規則) 限制對虛擬網路的存取。 只有輸入防火牆規則中所定義的 IP 位址能夠與 HDInsight 叢集通訊。 此設定可提供周邊安全性。

透過 VPN 閘道服務可以達成另一層的周邊安全性。 閘道是做為第一道防線，可抵禦任何對於 HDInsight 叢集的連入要求。 它會接受要求，驗證要求，之後才讓要求傳遞給叢集中的其他節點。 如此一來，閘道就可提供叢集中其他名稱和資料節點的周邊安全性。

## <a name="authentication"></a>驗證
企業系統管理員可以在[虛擬網路](https://azure.microsoft.com/services/virtual-network/)中建立具有 ESP 的 HDInsight 叢集。 HDInsight 叢集的所有節點都會加入企業所管理的網域。 透過使用 [Azure Active Directory Domain Services](../../active-directory-domain-services/active-directory-ds-overview.md) 即可達成此目的。 

企業員工可以藉由這項設定，便能使用其網域認證來登入叢集節點。 他們也可以使用其網域認證向其他已核准的端點 (例如 Ambari 檢視、ODBC、JDBC、PowerShell 和 REST API) 進行驗證，進而與叢集互動。 系統管理員可以完全控制透過這些端點與叢集互動的使用者數量。

## <a name="authorization"></a>Authorization
大多數企業所依循的最佳做法，就是確保並非每一位員工都能存取所有的企業資源。 同樣地，系統管理員可以針對叢集資源定義角色型存取控制原則。 

例如，系統管理員可以設定 [Apache Ranger](http://hortonworks.com/apache/ranger/) 以設定 Hive 的存取控制原則。 這項功能可確保員工只能存取順利工作所需的資料量。 叢集的 SSH 存取也僅限於系統管理員。

## <a name="auditing"></a>稽核
若要追蹤資源的未經授權存取或意外存取，就必須針對叢集資源和資料的所有存取進行稽核。 防止未經授權的使用者存取 HDInsight 叢集資源並保護資料是非常重要的。 

系統管理員可以檢視和報告所有對於 HDInsight 叢集資源和資料的存取。 針對在 Apache Ranger 支援的端點中所建立的存取控制原則，系統管理員也可以檢視和報告存取控制原則的所有變更。 

具有 ESP 的 HDInsight 叢集會使用熟悉的 Apache Ranger UI 來搜尋稽核記錄檔。 Ranger 會在後端使用 [Apache Solr](http://hortonworks.com/apache/solr/) 來儲存及搜尋記錄檔。

## <a name="encryption"></a>加密
若要符合組織安全性和合規性需求，保護資料就非常重要。 除了限制未經授權的員工存取資料之外，您還應該將資料加密。 

HDInsight 叢集的兩個資料存放區 (Azure Blob 儲存體和 Azure Data Lake Storage Gen1) 都支援透明伺服器端待用[資料加密](../../storage/common/storage-service-encryption.md)。 安全的 HDInsight 叢集會完美地與此伺服器端待用資料加密功能配合運作。

## <a name="next-steps"></a>後續步驟

* [規劃具有 ESP 的 HDInsight 叢集](apache-domain-joined-architecture.md)
* [設定具有 ESP 的 HDInsight 叢集](apache-domain-joined-configure.md)
* [管理具有 ESP 的 HDInsight 叢集](apache-domain-joined-manage.md)
* [為具有 ESP 的 HDInsight 叢集設定 Hive 原則](apache-domain-joined-run-hive.md)
* [搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md#domainjoined)

