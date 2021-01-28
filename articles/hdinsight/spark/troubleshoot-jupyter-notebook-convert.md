---
title: 無法在 Azure HDInsight 中建立 Jupyter Notebook
description: 瞭解與 Azure HDInsight 叢集互動時問題的疑難排解步驟和可能的解決方法。
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 02/11/2020
ms.openlocfilehash: d5f6612e22522452efb5eecd14c6c825bf29e6d5
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98938676"
---
# <a name="unable-to-create-jupyter-notebook-in-azure-hdinsight"></a>無法在 Azure HDInsight 中建立 Jupyter Notebook

本文說明與 Azure HDInsight 叢集互動時，問題的疑難排解步驟和可能的解決方式。

## <a name="issue"></a>問題

啟動 Jupyter Notebook 時，您會收到錯誤訊息，其中包含：

```error
Cannot convert notebook to v5 because that version doesn't exist
```

## <a name="cause"></a>原因

版本不相符。

## <a name="resolution"></a>解決方案

1. 使用 [ssh 命令](../hdinsight-hadoop-linux-use-ssh-unix.md)來連線到您的叢集。 編輯以下命令並將 CLUSTERNAME 取代為您叢集的名稱，然後輸入命令：

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. 藉 `_version.py` 由執行下列命令來開啟：

    ```bash
    sudo nano /usr/bin/anaconda/lib/python2.7/site-packages/nbformat/_version.py
    ```

1. 將 **5** 變更為 **4** ，讓修改過的行顯示如下：

    ```python
    version_info = (4, 0, 3)
    ```

    輸入 **Ctrl + X**、 **Y**、 **Enter** 來儲存變更。

1. 從網頁瀏覽器瀏覽至 `https://CLUSTERNAME.azurehdinsight.net/#/main/services/JUPYTER`，其中 `CLUSTERNAME` 是叢集的名稱。

1. 重新開機 Jupyter 服務。

## <a name="next-steps"></a>後續步驟

如果您沒有看到您的問題，或無法解決您的問題，請瀏覽下列其中一個管道以取得更多支援：

* 透過 [Azure 社群支援](https://azure.microsoft.com/support/community/)獲得由 Azure 專家所提供的解答。

* 連線至 [@AzureSupport](https://twitter.com/azuresupport) - 這是用來改善客戶體驗的官方 Microsoft Azure 帳戶。 將 Azure 社群連線到正確的資源：解答、支援和專家。

* 如果需要更多協助，您可在 [Azure 入口網站](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/)提交支援要求。 從功能表列中選取 [支援] 或開啟 [說明 + 支援] 中樞。 如需詳細資訊，請參閱[如何建立 Azure 支援要求](../../azure-portal/supportability/how-to-create-azure-support-request.md)。 您可透過 Microsoft Azure 訂閱來存取訂閱管理和帳單支援，並透過其中一項 [Azure 支援方案](https://azure.microsoft.com/support/plans/)以取得技術支援。
