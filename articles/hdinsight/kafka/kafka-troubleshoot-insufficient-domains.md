---
title: Azure HDInsight 中的區域錯誤沒有足夠的容錯網域
description: 叢集建立失敗，因為 Azure HDInsight 的區域中沒有足夠的容錯網域
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 08/09/2019
ms.openlocfilehash: 3f7d866d1c9b8c8437bc0f84acca47e0b8631895
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98939055"
---
# <a name="scenario-cluster-creation-failed-due-to-not-sufficient-fault-domains-in-region-in-azure-hdinsight"></a>案例：叢集建立失敗，因為 `not sufficient fault domains in region` Azure HDInsight 中

本文說明與 Azure HDInsight 叢集互動時，問題的疑難排解步驟和可能的解決方式。

## <a name="issue"></a>問題

嘗試建立 Apache Kafka 叢集時，會收到類似的錯誤訊息 `not sufficient fault domains in region` 。

## <a name="cause"></a>原因

容錯網域是 Azure 資料中心內基礎硬體的邏輯群組。 每個容錯網域會共用通用電源和網路交換器。 實作 HDInsight 叢集內節點的虛擬機器和受控磁碟會分散於這些容錯網域。 此架構會限制實體硬體故障的潛在影響。

每個 Azure 區域有特定數目的容錯網域。 如需網域清單以及它們所包含的容錯網域數目，請參閱 [可用性設定組](../../virtual-machines/manage-availability.md)的相關檔。

在 HDInsight 中，Kafka 叢集必須布建在至少有三個容錯網域的區域中。

## <a name="resolution"></a>解決方案

如果您想要建立叢集的區域沒有足夠的容錯網域，請與產品小組聯繫，以允許布建叢集，即使沒有三個容錯網域也一樣。

## <a name="next-steps"></a>後續步驟

如果您沒有看到您的問題，或無法解決您的問題，請瀏覽下列其中一個管道以取得更多支援：

* 透過 [Azure 社群支援](https://azure.microsoft.com/support/community/)獲得由 Azure 專家所提供的解答。

* 連線至 [@AzureSupport](https://twitter.com/azuresupport) - 這是用來改善客戶體驗的官方 Microsoft Azure 帳戶。 將 Azure 社群連線到正確的資源：解答、支援和專家。

* 如果需要更多協助，您可在 [Azure 入口網站](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/)提交支援要求。 從功能表列中選取 [支援] 或開啟 [說明 + 支援] 中樞。 如需詳細資訊，請參閱[如何建立 Azure 支援要求](../../azure-portal/supportability/how-to-create-azure-support-request.md)。 您可透過 Microsoft Azure 訂閱來存取訂閱管理和帳單支援，並透過其中一項 [Azure 支援方案](https://azure.microsoft.com/support/plans/)以取得技術支援。