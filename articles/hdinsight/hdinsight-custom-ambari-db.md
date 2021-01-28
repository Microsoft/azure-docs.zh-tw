---
title: Azure HDInsight 上的自訂 Apache Ambari 資料庫
description: 瞭解如何使用您自己的自訂 Apache Ambari 資料庫建立 HDInsight 叢集。
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 01/12/2021
ms.openlocfilehash: fe38ddc594060c78a2d26e9b25476e38736b4cf7
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98946052"
---
# <a name="set-up-hdinsight-clusters-with-a-custom-ambari-db"></a>使用自訂 Ambari DB 設定 HDInsight 叢集

Apache Ambari 簡化了 Apache Hadoop 叢集的管理和監視。 Ambari 提供容易使用的 web UI 和 REST API。 Ambari 包含於 HDInsight 叢集中，可用來監視叢集及進行設定變更。

在一般叢集建立中（如在 [hdinsight 中設定](hdinsight-hadoop-provision-linux-clusters.md)叢集所述），Ambari 會部署在由 hdinsight 管理且無法供使用者存取的 [S0 Azure SQL Database](../azure-sql/database/resource-limits-dtu-single-databases.md#standard-service-tier) 中。

自訂 Ambari DB 功能可讓您在所管理的外部資料庫中部署新的叢集和設定 Ambari。 部署是使用 Azure Resource Manager 範本來完成。 這項功能具有下列優點：

- 自訂-您可以選擇資料庫的大小和處理容量。 如果您有大型的叢集處理密集型工作負載，較低規格的 Ambari 資料庫可能會成為管理作業的瓶頸。
- 彈性-您可以視需要調整資料庫以符合您的需求。
- 控制-您可以使用符合組織需求的方式來管理資料庫的備份和安全性。

本文的其餘部分將討論下列幾點：

- 使用自訂 Ambari DB 功能的需求
- 使用您自己的 Apache Ambari Apache 資料庫布建 HDInsight 叢集時所需的步驟

## <a name="custom-ambari-db-requirements"></a>自訂 Ambari DB 需求

您可以部署具有所有叢集類型和版本的自訂 Ambari DB。 多個叢集無法使用相同的 Ambari DB。

自訂 Ambari DB 具有下列其他需求：

- 資料庫名稱不能包含連字號或空格
- 您必須有現有的 Azure SQL DB 伺服器和資料庫。
- 您為 Ambari 安裝程式提供的資料庫必須是空的。 預設 dbo 架構中不應該有任何資料表。
- 用來連接到資料庫的使用者應該擁有資料庫的 SELECT、CREATE TABLE 和 INSERT 許可權。
- 開啟選項，以允許在您將裝載 Ambari 的伺服器上 [存取 Azure 服務](../azure-sql/database/vnet-service-endpoint-rule-overview.md#azure-portal-steps) 。
- 防火牆規則中必須允許來自 HDInsight 服務的管理 IP 位址。 請參閱 [HDInsight 管理 ip 位址](hdinsight-management-ip-addresses.md) ，以取得必須新增至伺服器層級防火牆規則的 IP 位址清單。

當您在外部資料庫中裝載 Apache Ambari DB 時，請記住下列幾點：

- 您必須負責保存 Ambari 的 Azure SQL DB 額外成本。
- 定期備份您的自訂 Ambari DB。 Azure SQL Database 會自動產生備份，但備份保留時間範圍會有所不同。 如需詳細資訊，請參閱[了解自動 SQL Database 備份](../azure-sql/database/automated-backups-overview.md)。

## <a name="deploy-clusters-with-a-custom-ambari-db"></a>使用自訂 Ambari DB 部署叢集

若要建立使用您自己的外部 Ambari 資料庫的 HDInsight 叢集，請使用 [自訂 AMBARI DB 快速入門範本](https://github.com/Azure/azure-quickstart-templates/tree/master/101-hdinsight-custom-ambari-db)。

編輯中的參數， `azuredeploy.parameters.json` 以指定將保留 Ambari 的新叢集和資料庫的相關資訊。

您可以使用 Azure CLI 開始部署。 取代 `<RESOURCEGROUPNAME>` 為您要在其中部署叢集的資源群組。

```azurecli
az deployment group create --name HDInsightAmbariDBDeployment \
    --resource-group <RESOURCEGROUPNAME> \
    --template-file azuredeploy.json \
    --parameters azuredeploy.parameters.json
```

## <a name="database-sizing"></a>資料庫大小調整

下表提供根據 HDInsight 叢集的大小來選取 Azure SQL DB 層的指導方針。

| 背景工作節點數目 | 需要的資料庫層 |
|---|---|
| <= 4 | S0 |
| >4 && <= 8 | S1 |
| >8 && <= 16 | S2 |
| >16 && <= 32 | S3 |
| >32 && <= 64 | S4 |
| >64 && <= 128 | P2 |
| >128 | 請連絡支援人員 |

## <a name="next-steps"></a>後續步驟

- [在 Azure HDInsight 中使用外部中繼資料存放區](hdinsight-use-external-metadata-stores.md)
