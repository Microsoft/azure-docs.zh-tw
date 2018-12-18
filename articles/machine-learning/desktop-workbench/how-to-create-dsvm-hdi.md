---
title: 如何建立 DSVM 和 HDI 作為 Azure ML 的計算目標
description: 建立 DSVM 和 HDI Spark 叢集作為 Azure ML 實驗的計算目標。
services: machine-learning
author: hning86
ms.author: haining
manager: mwinkle
ms.reviewer: jmartens, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.date: 09/26/2017
ROBOTS: NOINDEX
ms.openlocfilehash: 60abe46670353121ad308f8926a7ee178c76e74e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46951656"
---
# <a name="create-dsvm-and-hdi-spark-cluster-as-compute-targets"></a>建立 DSVM 及 HDI Spark 叢集作為計算目標

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)]

您可以輕鬆地相應增加或相應放大機器學習實驗，方法是新增其他計算目標，例如以 Ubuntu 為基礎的 DSVM (資料科學虛擬機器) 和 Apache Spark for Azure HDInsight 叢集。 本文會引導您逐步在 Azure 中建立這些計算目標。 如需 Azure ML 計算目標的詳細資訊，請參閱 [Azure Machine Learning 測試服務概觀](experimentation-service-configuration.md)。

>[!NOTE]
>您需要先確定您具有適當的權限可建立資源 (例如 Azure 中的 VM 和 HDI 叢集)，再繼續進行。 根據設定，這兩個資源也可能會耗用許多計算核心。 請確定您的訂用帳戶有足夠的虛擬 CPU 核心容量。 您一律可以連絡 Azure 支援人員，以增加訂用帳戶中允許的核心數目上限。

## <a name="create-an-ubuntu-dsvm-in-azure-portal"></a>在 Azure 入口網站中建立 Ubuntu DSVM

您可以從 Azure 入口網站建立 DSVM。 

1. 從 https://portal.azure.com 登入 Azure 入口網站
2. 按一下 [+新增] 連結，並搜尋「適用於 Linux 的資料科學虛擬機器」。
    ![Ubuntu](media/how-to-create-dsvm-hdi/ubuntu_dsvm.png)
4. 選擇清單中的 [適用於 Linux (Ubuntu) 的資料科學虛擬機器]，並遵循建立 DSVM 的螢幕指示。

>[!IMPORTANT]
>請務必選擇 [密碼] 作為 [驗證類型]。

![use pwd](media/how-to-create-dsvm-hdi/use_pwd.png)

## <a name="create-an-ubuntu-dsvm-using-azure-cli"></a>使用 azure-cli 建立 Ubuntu DSVM

您也可以使用 Azure 資源管理範本來部署 DSVM。

>[!NOTE]
>所有下列命令都是假設從 Azure ML 專案的根資料夾發出。

首先，使用慣用的文字編輯器在 `docs` 資料夾中建立 `mydsvm.json` 檔案 (如果專案根資料夾中沒有 `docs` 資料夾，則請建立一個)。我們會使用這個檔案來設定 Azure 資源管理範本的一些基本參數。 

複製下列 JSON 程式碼片段，並將其貼入 `mydsvm.json` 檔案，然後填入適當的值：

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
     "adminUsername": { "value" : "<admin username>"},
     "adminPassword": { "value" : "<admin password>"},
     "vmName": { "value" : "<vm name>"},
     "vmSize": { "value" : "<vm size>"}
  }
}
```

針對 _vmSize_ 欄位，您可以使用 [Ubuntu DSVM Azure 資源管理範本](https://github.com/Azure/DataScienceVM/blob/master/Scripts/CreateDSVM/Ubuntu/multiazuredeploywithext.json)中所列的任何支援 VM 大小。 建議您使用下列其中一種大小作為 Azure ML 的計算目標。 


>[!TIP]
> 針對[深度工作負載](how-to-use-gpu.md)，您可以部署至採用 GPU 的 VM。

- [一般用途 VM](../../virtual-machines/linux/sizes-general.md)
  - Standard_DS2_v2 
  - Standard_DS3_v2 
  - Standard_DS4_v2 
  - Standard_DS12_v2 
  - Standard_DS13_v2 
  - Standard_DS14_v2 
- [採用 GPU 的 VM](../../virtual-machines/linux/sizes-gpu.md)
  - Standard_NC6 
  - Standard_NC12 
  - Standard_NC24 
 

深入了解這些 [Azure 中的 Linux 虛擬機器大小](../../virtual-machines/linux/sizes.md)和其[定價資訊](https://azure.microsoft.com/pricing/details/virtual-machines/linux/)。

從 Azure ML Workbench 應用程式啟動 CLI Window，方法是按一下 [檔案] --> [開啟命令提示字元] 或 [開啟 PowerShell] 功能表項目。 

>[!NOTE]
>您也可以在已安裝 az-cli 的任何命令列環境中執行這項作業。

在命令列視窗中，輸入下列命令：

```azurecli
# first make sure you have a valid Azure authentication token
$ az account get-access-token

# if you don't have a valid token, please log in to Azure first. 
# if you already do, you can skip this step.
$ az login

# list all subscriptions you have access to
$ az account list -o table

# make sure you set the subscription you want to use to create DSVM as the current subscription
$ az account set -s <subscription name or Id>

# it is always a good idea to create a resource group for the VM and associated resources to live in.
# you can use any Azure region, but it is best to create them in the region where your Azure ML Experimentation account is, e.g. eastus2, westcentralus or australiaeast.
# also, only certain Azure regions has GPU-equipped VMs available.
$ az group create -n <resource group name> -l <azure region>

# now let's create the DSVM based on the JSON configuration file you created earlier.
# note we assume the mydsvm.json config file is placed in the "docs" sub-folder.
$ az group deployment create -g <resource group name> --template-uri https://raw.githubusercontent.com/Azure/DataScienceVM/master/Scripts/CreateDSVM/Ubuntu/azuredeploy.json --parameters @docs/mydsvm.json

# find the FQDN (fully qualified domain name) of the VM just created. 
# you can also use IP address from the next command if FQDN is not set.
$ az vm show -g <resource group name> -n <vm name> --query "fqdns"

# find the IP address of the VM just created
$ az vm show -g <resource group name> -n <vm name> --query "publicIps"
```
## <a name="attach-a-dsvm-compute-target"></a>連結 DSVM 計算目標
建立 DSVM 之後，現在即可將它連結至 Azure ML 專案。

```azurecli
# attach the DSVM compute target
# it is a good idea to use FQDN in case the IP address changes after you deallocate the VM and restart it
$ az ml computetarget attach remotedocker --name <compute target name> --address <ip address or FQDN> --username <admin username> --password <admin password> 

# prepare the Docker image on the DSVM 
$ az ml experiment prepare -c <compute target name>
```
現在，您應該已準備好在此 DSVM 上執行實驗。

## <a name="deallocate-a-dsvm-and-restart-it-later"></a>解除配置 DSVM，稍後再重新予以啟動
當您從 Azure ML 完成計算工作時，可以解除配置 DSVM。 這個動作會關閉 VM、釋放計算資源，但會保留虛擬磁碟。 解除配置 VM 後，就不會再向您收取計算成本費用。

解除配置 VM：

```azurecli
$ az vm deallocate -g <resource group name> -n <vm name>
```

若要重新啟動 VM，請使用 `az ml start` 命令：

```azurecli
$ az vm start -g <resource group name> -n <vm name>
```

## <a name="expand-the-dsvm-os-disk"></a>延伸 DSVM OS 磁碟
Ubuntu DSVM 隨附 50 GB OS 磁碟和 100 GB 資料磁碟。 Docker 將映像儲存在資料磁碟上，因為該磁碟提供更多空間。 用作 Azure ML 的計算目標時，Docker 引擎拉下 Docker 映像並在其上建置 conda 層可能會將磁碟空間用完。 您可能需要將該磁碟擴充到較大的大小 (例如 200 GB)，避免您在執行時發生「磁碟已滿」錯誤。 請參閱[如何使用 Azure CLI 擴充 Linux VM 上的虛擬硬碟](../../virtual-machines/linux/expand-disks.md)，以了解如何輕鬆地從 azure-cli 完成此動作。 

## <a name="create-an-apache-spark-for-azure-hdinsight-cluster-in-azure-portal"></a>在 Azure 入口網站中建立 Apache Spark for Azure HDInsight 叢集

若要執行相應放大 Spark 作業，您需要在 Azure 入口網站中建立 Apache Spark for Azure HDInsight 叢集。

1. 從 https://portal.azure.com 登入 Azure 入口網站
2. 按一下 [+新增] 連結，並搜尋 "HDInsight"。

    ![find hdi](media/how-to-create-dsvm-hdi/hdi.png)
    
3. 選擇清單中的 [HDInsight]，然後按一下 [建立] 按鈕。
4. 在 [基本] 設定畫面的 [叢集類型] 設定中，確定您選擇 [Spark] 作為 [叢集類型]、[Linux] 作為 [作業系統]，以及 [Spark 2.1.0 (HDI 3.6)] 作為 _Version。

    ![configure hdi](media/how-to-create-dsvm-hdi/configure_hdi.png)

    >[!IMPORTANT]
    >請注意，在上面的畫面中，叢集有 [叢集登入使用者名稱] 欄位和 [安全殼層 (SSH) 使用者名稱] 欄位。 這些是兩個不同的使用者身分識別，即使為了方便起見，您還是可以為這兩個登入指定相同的密碼。 [叢集登入使用者名稱] 是用來登入 HDI 叢集的管理 Web UI。 [SSH 登入使用者名稱] 是用來登入叢集的前端節點，而且這是 Azure ML 分派 Spark 作業所需的項目。

5. 選擇所需的叢集大小和節點大小，並完成建立精靈。 最多可能需要 30 分鐘的時間，叢集才能完成佈建。 

## <a name="attach-an-hdi-spark-cluster-compute-target"></a>連結 HDI Spark 叢集計算目標

建立 Spark HDI 叢集之後，現在即可將它連結至 Azure ML 專案。

```azurecli
# attach the HDI compute target
$ az ml computetarget attach cluster --name <compute target name> --address <cluster name, such as myhdicluster123.azurehdinsight.net> --username <ssh username> --password <ssh password> 

# prepare the conda environment on HDI
$ az ml experiment prepare -c <compute target name>
```
現在，您應該已準備好在此 Spark 叢集上執行實驗。

## <a name="next-steps"></a>後續步驟

深入了解：
- [Azure Machine Learning 測試服務概觀](experimentation-service-configuration.md)
- [Azure Machine Learning Workbench 測試服務組態檔](experimentation-service-configuration-reference.md)
- [Apache Spark for Azure HDInsight 叢集](https://azure.microsoft.com/services/hdinsight/apache-spark/)
- [資料科學虛擬機器](https://azure.microsoft.com/services/virtual-machines/data-science-virtual-machines/)
