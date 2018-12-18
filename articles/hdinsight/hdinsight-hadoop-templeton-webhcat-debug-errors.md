---
title: 了解並解決 HDInsight 上的 WebHCat 錯誤 - Azure
description: 了解 WebHCat 在 HDInsight 上傳回的常見錯誤以及如何解決這些問題。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/16/2018
ms.author: jasonh
ms.openlocfilehash: 3c4ce93a12d0ecb86552e9f73d83f623e71385a0
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43103400"
---
# <a name="understand-and-resolve-errors-received-from-webhcat-on-hdinsight"></a>了解並解決 HDInsight 上從 WebHCat 收到的錯誤

了解搭配 HDInsight 使用 WebHCat 時遇到的錯誤，以及如何解決它們。 WebHCat 是由用戶端工具 (例如 Azure PowerShell 與 Data Lake Tools for Visual Studio) 在內部使用。

## <a name="what-is-webhcat"></a>什麼是 WebHCat？

[WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat) 是適用於 [HCatalog](https://cwiki.apache.org/confluence/display/Hive/HCatalog) 的 REST API，這是 Hadoop 的資料表和儲存體管理層。 WebHCat 預設會在 HDInsight 叢集上啟用，並由各種工具用來提交工作、取得工作狀態等，而不需登入叢集。

## <a name="modifying-configuration"></a>修改組態

> [!IMPORTANT]
> 因為已超過設定的上限，而發生本文件中所列的幾個錯誤。 當解決步驟提到您可以變更值時，您必須使用下列其中一項來執行變更：

* 針對 **Windows** 叢集：使用指令碼動作在叢集建立期間設定此值。 如需詳細資訊，請參閱 [開發指令碼動作](hdinsight-hadoop-script-actions.md)。

* 針對 **Linux** 叢集：使用 Ambari (Web 或 REST API) 來修改此值。 如需詳細資訊，請參閱 [使用 Ambari 管理 HDInsight](hdinsight-hadoop-manage-ambari.md)

> [!IMPORTANT]
> Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](hdinsight-component-versioning.md#hdinsight-windows-retirement)。

### <a name="default-configuration"></a>預設組態

如果超過下列預設值，可能會使得 WebHCat 效能變差或導致發生錯誤：

| 設定 | 作用 | 預設值 |
| --- | --- | --- |
| [yarn.scheduler.capacity.maximum-applications][maximum-applications] |可以同時為作用中 (擱置或執行中) 的工作數目上限 |10,000 |
| [templeton.exec.max-procs][max-procs] |可以同時提供服務的要求數目上限 |20 |
| [mapreduce.jobhistory.max-age-ms][max-age-ms] |將保留作業歷程記錄的天數 |7 天 |

## <a name="too-many-requests"></a>要求太多

**HTTP 狀態碼**：429

| 原因 | 解決方案 |
| --- | --- |
| 您已超出 WebHCat 每分鐘提供服務的並行要求上限 (預設值為 20) |減少您的工作負載以確保不會提交超過並行要求數目上限，或藉由修改 `templeton.exec.max-procs`來提高並行要求限制。 如需詳細資訊，請參閱[修改組態](#modifying-configuration) 。 |

## <a name="server-unavailable"></a>無法使用伺服器

**HTTP 狀態碼**：503

| 原因 | 解決方案 |
| --- | --- |
| 此狀態碼通常發生於叢集的主要和次要 HeadNode 間的容錯移轉期間 |等候兩分鐘，然後重試作業 |

## <a name="bad-request-content-could-not-find-job"></a>不正確的要求內容：找不到工作

**HTTP 狀態碼**：400

| 原因 | 解決方案 |
| --- | --- |
| 工作歷程記錄清除器已清除工作詳細資料 |工作歷程記錄的預設保留期間為 7 天。 您可以透過修改 `mapreduce.jobhistory.max-age-ms` 來變更預設保留期間。 如需詳細資訊，請參閱[修改組態](#modifying-configuration) 。 |
| 工作因為容錯移轉而遭到刪除 |最多重試工作提交兩分鐘 |
| 使用的工作識別碼無效 |檢查工作識別碼是否正確 |

## <a name="bad-gateway"></a>閘道不正確

**HTTP 狀態碼**：502

| 原因 | 解決方案 |
| --- | --- |
| 內部記憶體回收會發生於 WebHCat 程序中 |等候記憶體回收完成，或重新啟動 WebHCat 服務 |
| 等候 ResourceManager 服務回應時逾時。 此錯誤發生於作用中應用程式的數目達到設定的最大值時 (預設值為 10,000) |等候目前執行中的工作完成，或修改 `yarn.scheduler.capacity.maximum-applications`以提高並行工作限制。 如需詳細資訊，請參閱[修改組態](#modifying-configuration)一節。 |
| 嘗試在 `Fields` 設定為 `*` 時透過 [GET /jobs](https://cwiki.apache.org/confluence/display/Hive/WebHCat+Reference+Jobs) 呼叫擷取所有作業 |不要擷取*所有*作業詳細資料。 請改為使用 `jobid` 只擷取大於特定作業識別碼的作業詳細資料。或者，不要使用 `Fields` |
| WebHCat 服務在 HeadNode 容錯移轉期間關閉 |等候兩分鐘，然後重試作業 |
| 有 500 個以上透過 WebHCat 提交的擱置工作 |等到目前擱置的工作完成，再送出更多工作 |

[maximum-applications]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.3/bk_system-admin-guide/content/setting_application_limits.html
[max-procs]: https://cwiki.apache.org/confluence/display/Hive/WebHCat+Configure#WebHCatConfigure-WebHCatConfiguration
[max-age-ms]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.0.6.0/ds_Hadoop/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml
