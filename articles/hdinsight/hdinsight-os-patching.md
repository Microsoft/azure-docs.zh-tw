---
title: 以 Linux 為基礎的 HDInsight 叢集設定作業系統修補排程 - Azure
description: 了解如何為以 Linux 為基礎的 HDInsight 叢集設定作業系統修補排程。
services: hdinsight
author: omidm1
ms.author: omidm
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 03/21/2017
ms.openlocfilehash: 8bde1e8846dbaee957e2498ea4fae0c5cf79a913
ms.sourcegitcommit: 744747d828e1ab937b0d6df358127fcf6965f8c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "42140056"
---
# <a name="os-patching-for-hdinsight"></a>HDInsight 的作業系統修補 
HDInsight 作為受控 Hadoop 服務，會負責修補 HDInsight 叢集所使用之基礎 VM 的作業系統。 自 2016 年 8 月 1 日起，我們已變更以 Linux 為基礎之 HDInsight 叢集 (3.4 版或更新版本) 的客體 OS 修補原則。 新原則的目標是大幅減少因為修補而產生的重新開機次數。 新的原則將會在每個星期一或星期四 UTC 上午 12 時開始，以交錯方式在任何指定叢集的節點之間，繼續修補 Linux 叢集上的虛擬機器 (VM)。 不過，任何指定的 VM 每隔 30 天只會因為客體 OS 修補而最多重新開機一次。 此外，新建立的叢集在叢集建立之後，也不會在 30 天內第一次重新開機。 VM 重新開機後，修補便會生效。

## <a name="how-to-configure-the-os-patching-schedule-for-linux-based-hdinsight-clusters"></a>如何為以 Linux 為基礎的 HDInsight 叢集設定作業系統修補排程
HDInsight 叢集中的虛擬機器有時候需要重新開機，以便系統可以安裝重要的安全性修補程式。 自 2016 年 8 月 1 日起，新式的以 Linux 為基礎的 HDInsight 叢集 (3.4 版或更新版本) 會使用下列排程來重新開機︰

1. 在 30 天的期間內，叢集中的虛擬機器最多只能為了修補而重新開機一次。
2. 重新開機作業會於 12AM UTC 開始。
3. 叢集中的虛擬機器會錯開進行重新開機程序，因此在重新開機程序進行期間，叢集仍可供使用。
4. 新建立之叢集的第一次重新開機，最快要在叢集建立日期的 30 天之後才會發生。

您可以使用本文所述的指令碼動作來修改作業系統修補排程，如下所示︰
1. 啟用或停用自動重新開機
2. 設定重新開機頻率 (重新開機間隔天數)
3. 設定要在星期幾重新開機

> [!NOTE]
> 此指令碼動作只適用於在 2016 年 8 月 1 日之後建立的以 Linux 為基礎的 HDInsight 叢集。 VM 重新開機後，修補才會生效。 
>

## <a name="how-to-use-the-script"></a>如何使用指令碼 

使用此指令碼時需要下列資訊︰
1. 指令碼位置： https://hdiconfigactions.blob.core.windows.net/linuxospatchingrebootconfigv01/os-patching-reboot-config.sh。HDInsight 會使用此 URI 在叢集中的所有虛擬機器上尋找和執行指令碼。
  
2. 指令碼會套用到的叢集節點類型︰headnode、workernode、zookeeper。 此指令碼必須套用至叢集中的所有節點類型。 如果未套用至某個節點類型，則該節點類型的虛擬機器會繼續使用先前的修補排程。


3.  參數︰此指令碼接受三個數值參數︰

    | 參數 | 定義 |
    | --- | --- |
    | 啟用/停用自動重新開機 |0 或 1。 值為 0 會停用自動重新開機，1 則會啟用自動重新開機。 |
    | 頻率 |7 到 90 (含)。 需要重新開機的修補程式在將虛擬機器重新開機之前所要等候的天數。 |
    | 星期幾 |1 到 7 (含)。 值為 1 表示虛擬機器應該在星期一重新開機，7 則表示星期日。例如，使用參數 1 60 2，會導致虛擬機器每隔 60 天 (最多) 就會在星期二自動重新開機。 |
    | 持續性 |對現有叢集套用指令碼動作時，您可以將指令碼標示為持續性。 持續性指令碼會在透過調整作業將新的 workernode 新增至叢集時套用。 |

> [!NOTE]
> 在對現有叢集套用此指令碼時，您必須將其標示為持續性。 否則，透過調整作業所建立的新節點會使用預設的修補排程。
如果您在進行叢集建立程序時套用指令碼，該指令碼會自動成為持續性狀態。
>

## <a name="next-steps"></a>後續步驟

如需使用指令碼動作的特定步驟，請參閱[使用指令碼動作來自訂以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)中的以下章節：

* [在建立叢集期間使用指令碼動作](hdinsight-hadoop-customize-cluster-linux.md#use-a-script-action-during-cluster-creation)
* [將指令碼動作套用到執行中的叢集](hdinsight-hadoop-customize-cluster-linux.md#apply-a-script-action-to-a-running-cluster)
