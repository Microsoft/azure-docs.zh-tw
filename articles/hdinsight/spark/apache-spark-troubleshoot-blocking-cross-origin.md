---
title: Jupyter 404 錯誤-「封鎖跨原始來源 API」-Azure HDInsight
description: Jupyter 伺服器404「找不到」錯誤，因為 Azure HDInsight 中的「封鎖跨原始來源 API」
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 07/29/2019
ms.openlocfilehash: 27cd3aff859fd46679679ac12d3acc03fa6da158
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98929455"
---
# <a name="scenario-jupyter-server-404-not-found-error-due-to-blocking-cross-origin-api-in-azure-hdinsight"></a>案例： Jupyter 伺服器404「找不到」錯誤，因為 Azure HDInsight 中的「封鎖跨原始來源 API」

本文說明在 Azure HDInsight 叢集中使用 Apache Spark 元件時，疑難排解步驟和可能的解決方案。

## <a name="issue"></a>問題

當您存取 HDInsight 上的 Jupyter 服務時，您會看到錯誤方塊，指出「找不到」。 如果您檢查 Jupyter 記錄，將會看到如下的內容：

```log
[W 2018-08-21 17:43:33.352 NotebookApp] 404 PUT /api/contents/PySpark/notebook.ipynb (10.16.0.144) 4504.03ms referer=https://pnhr01hdi-corpdir.msappproxy.net/jupyter/notebooks/PySpark/notebook.ipynb
Blocking Cross Origin API request.  
Origin: https://xxx.xxx.xxx, Host: pnhr01.j101qxjrl4zebmhb0vmhg044xe.ax.internal.cloudapp.net:8001
```

您也可能會在 Jupyter 記錄檔的 [來源] 欄位中看到 IP 位址。

## <a name="cause"></a>原因

這項錯誤可能是由幾個原因所造成：

- 如果您已設定網路安全性群組 (NSG) 規則來限制對叢集的存取。 使用 NSG 規則限制存取權仍可讓您使用 IP 位址而非叢集名稱，直接存取 Apache Ambari 和其他服務。 不過，在存取 Jupyter 時，您可能會看到404「找不到」錯誤。

- 如果您已將 HDInsight 閘道指定為標準以外的自訂 DNS 名稱 `xxx.azurehdinsight.net` 。

## <a name="resolution"></a>解決方案

1. 修改下列兩個位置中的 jupyter.py 檔案：

    ```bash
    /var/lib/ambari-server/resources/common-services/JUPYTER/1.0.0/package/scripts/jupyter.py
    /var/lib/ambari-agent/cache/common-services/JUPYTER/1.0.0/package/scripts/jupyter.py
    ```

1. 尋找指出：的行， `NotebookApp.allow_origin='\"https://{2}.{3}\"'` 並將其變更為： `NotebookApp.allow_origin='\"*\"'` 。

1. 從 Ambari 重新開機 Jupyter 服務。

1. 在命令提示字元中輸入， `ps aux | grep jupyter` 應該會顯示它允許任何 URL 連接到它。

這比我們已經備妥的設定更不安全。 但是，它會被視為受限制的叢集存取，而且可以在已 NSG 的情況下，從外部連接到叢集。

## <a name="next-steps"></a>後續步驟

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]