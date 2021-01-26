---
title: Azure HDInsight 的版本資訊
description: Azure HDInsight 的最新版本資訊。 取得 Hadoop、Spark、R Server、Hive 等的開發秘訣和詳細資料。
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.service: hdinsight
ms.topic: conceptual
ms.date: 11/12/2020
ms.openlocfilehash: 5c414a11085a6a37dee6be522dcf513e8990e5e2
ms.sourcegitcommit: a055089dd6195fde2555b27a84ae052b668a18c7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/26/2021
ms.locfileid: "98786346"
---
# <a name="azure-hdinsight-release-notes"></a>Azure HDInsight 版本資訊

本文提供有關 **最新** Azure HDInsight 版本更新的資訊。 如需有關較早版本的詳細資訊，請參閱 [HDInsight 版本資訊封存](hdinsight-release-notes-archive.md)。

## <a name="summary"></a>摘要

Azure HDInsight 是最受企業客戶歡迎的其中一項服務，可供 Azure 上的開放原始碼分析使用。

如果您想要訂閱版本資訊，請觀賞 [此 GitHub 存放庫](https://github.com/hdinsight/release-notes/releases)上的版本。

## <a name="release-date-11182020"></a>發行日期：11/18/2020

此版本適用于 HDInsight 3.6 和 HDInsight 4.0。 在數天內，所有區域都可以使用 HDInsight 發行版本。 此處的發行日期為第一個區域發行日期。 如果您沒有看到以下變更，請在數天內等待發行在您的區域中。

## <a name="new-features"></a>新功能
### <a name="auto-key-rotation-for-customer-managed-key-encryption-at-rest"></a>靜態客戶管理金鑰加密的自動金鑰輪替
從這個版本開始，客戶可以使用 Azure KeyValut 不限版本的加密金鑰 Url 來進行客戶管理的金鑰加密。 HDInsight 會在金鑰過期或取代為新的版本時，自動輪替這些金鑰。 如需詳細資訊，請參閱[此處](./disk-encryption.md)。

### <a name="ability-to-select-different-zookeeper-virtual-machine-sizes-for-spark-hadoop-and-ml-services"></a>能夠為 Spark、Hadoop 和 ML 服務選取不同的 Zookeeper 虛擬機器大小
HDInsight 先前不支援針對 Spark、Hadoop 和 ML 服務叢集類型自訂 Zookeeper 節點大小。 它會預設為 A2_v2/A2 的虛擬機器大小，並提供免費的費用。 在此版本中，您可以選取最適合您案例的 Zookeeper 虛擬機器大小。 A2_v2/A2 以外的虛擬機器大小的 Zookeeper 節點將會收取費用。 A2_v2 和 A2 虛擬機器仍提供免費的費用。

### <a name="moving-to-azure-virtual-machine-scale-sets"></a>移至 Azure 虛擬機器擴展集
HDInsight 現在會使用 Azure 虛擬機器來佈建叢集。 從這個版本開始，服務會逐漸遷移至 [Azure 虛擬機器擴展集](../virtual-machine-scale-sets/overview.md)。 整個過程可能需要數個月的時間。 遷移您的區域和訂用帳戶之後，新建立的 HDInsight 叢集將會在沒有客戶動作的虛擬機器擴展集上執行。 不需要中斷變更。

## <a name="deprecation"></a>淘汰
### <a name="deprecation-of-hdinsight-36-ml-services-cluster"></a>淘汰 HDInsight 3.6 ML 服務叢集
HDInsight 3.6 ML 服務叢集類型將于 31 2020 年12月結束支援。 客戶將無法在 31 2020 年12月之後建立新的 3.6 ML 服務叢集。 現有的叢集將會以現狀執行，不再有 Microsoft 支援。 請在 [此](./hdinsight-component-versioning.md#available-versions)檢查 HDInsight 版本和叢集類型的支援期限。

### <a name="disabled-vm-sizes"></a>停用的 VM 大小
從 16 2020 年11月開始，HDInsight 將會封鎖新客戶使用 standand_A8、standand_A9、standand_A10 和 standand_A11 VM 大小來建立叢集。 過去三個月內使用這些 VM 大小的現有客戶將不會受到影響。 從 9 2021 年1月開始，HDInsight 將會封鎖所有使用 standand_A8、standand_A9 standand_A10 和 standand_A11 VM 大小來建立叢集的客戶。 現有的叢集將會依原樣執行。 請考慮移至 HDInsight 4.0，以避免潛在的系統/支援中斷。

## <a name="behavior-changes"></a>行為變更
### <a name="add-nsg-rule-checking-before-scaling-operation"></a>在調整作業之前新增 NSG 規則檢查
HDInsight 將網路安全性群組新增 (Nsg) 和使用者定義的路由 (Udr) 檢查調整作業。 除了建立叢集之外，還會針對叢集調整進行相同的驗證。 這種驗證有助於防止無法預期的錯誤。 如果未通過驗證，則無法調整規模。 若要深入瞭解如何正確設定 Nsg 和 Udr，請參閱 [HDInsight 管理 IP 位址](./hdinsight-management-ip-addresses.md)。

## <a name="upcoming-changes"></a>即將推出的變更
即將發行的版本中將會發生下列變更。

### <a name="default-cluster-vm-size-will-be-changed-to-ev3-family"></a>預設叢集 VM 大小將會變更為 Ev3 系列
從下一版開始 (于) 年1月底，預設叢集 VM 大小將從 D 系列變更為 Ev3 系列。 這種變更適用于前端節點和背景工作節點。 若要避免這項變更，請指定您要在 ARM 範本中使用的 VM 大小。

### <a name="default-cluster-version-will-be-changed-to-40"></a>預設叢集版本將會變更為4。0
自2021年2月起，HDInsight 叢集的預設版本將自3.6 變更為4.0。 如需可用版本的詳細資訊，請參閱 [可用的版本](./hdinsight-component-versioning.md#available-versions)。 深入瞭解[HDInsight 4.0](./hdinsight-version-release.md)的新功能

### <a name="os-version-upgrade"></a>作業系統版本升級
HDInsight 正在將作業系統版本從16.04 升級為18.04。 升級將在2021年4月之前完成。

### <a name="hdinsight-36-end-of-support-on-june-30-2021"></a>HDInsight 3.6 年 6 30 2021 月終止支援
HDInsight 3.6 即將結束支援。 自 30 2021 年6月起，客戶無法建立新的 HDInsight 3.6 叢集。 現有的叢集將會以現狀執行，不再有 Microsoft 支援。 請考慮移至 HDInsight 4.0，以避免潛在的系統/支援中斷。

## <a name="bug-fixes"></a>錯誤修正
HDInsight 會持續改善叢集的可靠性和效能。 

## <a name="component-version-change"></a>元件版本變更
此發行版本沒有任何元件版本變更。 您可以在 [本](./hdinsight-component-versioning.md)檔中找到 hdinsight 4.0 和 hdinsight 3.6 目前的元件版本。

## <a name="known-issues"></a>已知問題
### <a name="prevent-hdinsight-cluster-vms-from-rebooting-periodically"></a>防止 HDInsight 叢集 VM 定期重新開機

自2020年11月起，您可能已注意到 HDInsight 叢集 Vm 定期重新開機。 這可能是由下列原因所造成：

1.  已在您的叢集上啟用 Clamav。 新的 azsec clamav 套件會耗用大量的記憶體，以觸發節點重新開機。 
2.  CRON 作業會每日排程，以監視 Azure 服務所使用的憑證授權單位單位清單 (Ca) 。 當有新的 CA 憑證可供使用時，腳本會將憑證新增至 JDK 信任存放區，並排定重新開機。

針對這兩個問題，HDInsight 會為所有執行中的叢集部署修正程式並套用修補程式。 若要立即套用修正程式並避免非預期的 Vm 重新開機，您可以在所有叢集節點上執行下列腳本動作，作為持續性腳本動作。 在修正和修補完成之後，HDInsight 會張貼另一個通知。
```
https://hdiconfigactions.blob.core.windows.net/linuxospatchingrebootconfigv02/replace_cacert_script.sh
https://healingscriptssa.blob.core.windows.net/healingscripts/ChangeOOMPolicyAndApplyLatestConfigForClamav.sh
```