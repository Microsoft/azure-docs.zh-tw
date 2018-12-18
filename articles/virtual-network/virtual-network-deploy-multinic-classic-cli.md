---
title: 建立具有多個 NIC 的 VM (傳統) - Azure 傳統 CLI | Microsoft Docs
description: 了解如何使用 Azure 傳統命令列介面 (CLI) 建立具有多個 NIC 的 VM (傳統)。
services: virtual-network
documentationcenter: na
author: genlin
manager: cshepard
editor: ''
tags: azure-service-management
ms.assetid: b436e41e-866c-439f-a7c7-7b4b041725ef
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/02/2016
ms.author: genli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 4ea53048819bfdad5c45e522115aa6e493dfc8bc
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46953352"
---
# <a name="create-a-vm-classic-with-multiple-nics-using-the-azure-classic-cli"></a>使用 Azure 傳統 CLI 建立具有多個 NIC 的 VM (傳統)

[!INCLUDE [virtual-network-deploy-multinic-classic-selectors-include.md](../../includes/virtual-network-deploy-multinic-classic-selectors-include.md)]

您可以在 Azure 中建立虛擬機器 (VM) 並將多個網路介面 (NIC) 連接至每個 VM。 有多個 NIC 時，可透過各個 NIC 分隔不同的流量類型。 例如，一個 NIC 可能與網際網路進行通訊，而另一個 NIC 則只與未連線到網際網路的內部資源進行通訊。 透過多個 NIC 分隔網路流量是許多網路虛擬設備 (例如應用程式交付和 WAN 最佳化解決方案) 所需的功能。

> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有二種：[Resource Manager 和傳統](../resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 了解如何使用 [Resource Manager 部署模型](../virtual-machines/linux/multiple-nics.md)執行這些步驟。

[!INCLUDE [virtual-network-deploy-multinic-scenario-include.md](../../includes/virtual-network-deploy-multinic-scenario-include.md)]

在下列步驟中，WEB 伺服器使用名為 *IaaSStory* 的資源群組，而 DB 伺服器使用名為 *IaaSStory-BackEnd* 的資源群組。

## <a name="prerequisites"></a>必要條件
您需要建立 *IaaSStory* 資源群組，其中含有此案例的所有必要資源，才能建立 DB 伺服器。 若要建立這些資源，請完成下列步驟。 依照[建立虛擬網路](virtual-networks-create-vnet-classic-cli.md)文章中的步驟建立虛擬網路。

[!INCLUDE [azure-cli-prerequisites-include.md](../../includes/azure-cli-prerequisites-include.md)]

## <a name="deploy-the-back-end-vms"></a>部署後端 VM
後端 VM 有賴於建立下列資源：

* **資料磁碟的儲存體帳戶**。 為取得更佳的效能，資料庫伺服器上的資料磁碟會使用需要進階儲存體帳戶的固態硬碟 (SSD) 技術。 請確定 Azure 的部署位置，以支援進階儲存體。
* **NIC**。 每部 VM 都會有兩個 NIC，一個用於資料庫存取，另一個用於管理。
* **可用性設定組**。 所有的資料庫伺服器都會加入單一的可用性設定組，確保在維護期間至少有一部 VM 啟動並執行。

### <a name="step-1---start-your-script"></a>步驟 1：啟動指令碼
[這裡](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/IaaS-Story/11-MultiNIC/classic/virtual-network-deploy-multinic-classic-cli.sh)可以下載所使用的完整 Bash 指令碼。 請完成下列步驟來變更指令碼，讓指令碼可在您的環境中運作：

1. 根據上述 [必要條件](#Prerequisites)中已部署的現有資源群組來變更下列變數的值。

    ```azurecli
    location="useast2"
    vnetName="WTestVNet"
    backendSubnetName="BackEnd"
    ```
2. 根據後端部署要使用的值，變更下列變數值。

    ```azurecli
    backendCSName="IaaSStory-Backend"
    prmStorageAccountName="iaasstoryprmstorage"
    image="0b11de9248dd4d87b18621318e037d37__RightImage-Ubuntu-14.04-x64-v14.2.1"
    avSetName="ASDB"
    vmSize="Standard_DS3"
    diskSize=127
    vmNamePrefix="DB"
    osDiskName="osdiskdb"
    dataDiskPrefix="db"
    dataDiskName="datadisk"
    ipAddressPrefix="192.168.2."
    username='adminuser'
    password='adminP@ssw0rd'
    numberOfVMs=2
    ```

### <a name="step-2---create-necessary-resources-for-your-vms"></a>步驟 2：為 VM 建立必要的資源
1. 為所有後端 VM 建立新的雲端服務。 請注意，資源群組名稱的 `$backendCSName` 變數，以及 Azure 區域之 `$location` 的使用方式。

    ```azurecli
    azure service create --serviceName $backendCSName \
        --location $location
    ```

2. 為您的 VM 要使用的作業系統和資料磁碟建立進階儲存體帳戶。

    ```azurecli
    azure storage account create $prmStorageAccountName \
        --location $location \
        --type PLRS
    ```

### <a name="step-3---create-vms-with-multiple-nics"></a>步驟 3：建立具有多個 NIC 的 VM
1. 根據 `numberOfVMs` 變數，啟動迴圈以建立多部 VM。

    ```azurecli
    for ((suffixNumber=1;suffixNumber<=numberOfVMs;suffixNumber++));
    do
    ```

2. 對於每個 VM，請分別為這兩個 NIC 的個別指定名稱和 IP 位址。

    ```azurecli
    nic1Name=$vmNamePrefix$suffixNumber-DA
    x=$((suffixNumber+3))
    ipAddress1=$ipAddressPrefix$x

    nic2Name=$vmNamePrefix$suffixNumber-RA
    x=$((suffixNumber+53))
    ipAddress2=$ipAddressPrefix$x
    ```

3. 建立 VM。 請注意使用 `--nic-config` 參數，其中包含具有名稱、子網路和 IP 位址的所有 NIC 清單。

    ```azurecli
    azure vm create $backendCSName $image $username $password \
        --connect $backendCSName \
        --vm-name $vmNamePrefix$suffixNumber \
        --vm-size $vmSize \
        --availability-set $avSetName \
        --blob-url $prmStorageAccountName.blob.core.windows.net/vhds/$osDiskName$suffixNumber.vhd \
        --virtual-network-name $vnetName \
        --subnet-names $backendSubnetName \
        --nic-config $nic1Name:$backendSubnetName:$ipAddress1::,$nic2Name:$backendSubnetName:$ipAddress2::
    ```

4. 針對每個 VM，請建立兩個資料磁碟。

    ```azurecli
    azure vm disk attach-new $vmNamePrefix$suffixNumber \
        $diskSize \
        vhds/$dataDiskPrefix$suffixNumber$dataDiskName-1.vhd

    azure vm disk attach-new $vmNamePrefix$suffixNumber \
        $diskSize \
        vhds/$dataDiskPrefix$suffixNumber$dataDiskName-2.vhd
    done
    ```

### <a name="step-4---run-the-script"></a>步驟 4：執行指令碼
現在您已根據需求下載並變更了指令碼，請執行指令碼來建立具有多個 NIC 的後端資料庫 VM。

1. 儲存您的指令碼並從 **Bash** 終端機執行。 您會看到初始的輸出，如下所示。

        info:    Executing command service create
        info:    Creating cloud service
        data:    Cloud service name IaaSStory-Backend
        info:    service create command OK
        info:    Executing command storage account create
        info:    Creating storage account
        info:    storage account create command OK
        info:    Executing command vm create
        info:    Looking up image 0b11de9248dd4d87b18621318e037d37__RightImage-Ubuntu-14.04-x64-v14.2.1
        info:    Looking up virtual network
        info:    Looking up cloud service
        info:    Getting cloud service properties
        info:    Looking up deployment
        info:    Creating VM

2. 幾分鐘後，執行將會結束，且您將會看到其餘的輸出，如下所示。

        info:    OK
        info:    vm create command OK
        info:    Executing command vm disk attach-new
        info:    Getting virtual machines
        info:    Adding Data-Disk
        info:    vm disk attach-new command OK
        info:    Executing command vm disk attach-new
        info:    Getting virtual machines
        info:    Adding Data-Disk
        info:    vm disk attach-new command OK
        info:    Executing command vm create
        info:    Looking up image 0b11de9248dd4d87b18621318e037d37__RightImage-Ubuntu-14.04-x64-v14.2.1
        info:    Looking up virtual network
        info:    Looking up cloud service
        info:    Getting cloud service properties
        info:    Looking up deployment
        info:    Creating VM
        info:    OK
        info:    vm create command OK
        info:    Executing command vm disk attach-new
        info:    Getting virtual machines
        info:    Adding Data-Disk
        info:    vm disk attach-new command OK
        info:    Executing command vm disk attach-new
        info:    Getting virtual machines
        info:    Adding Data-Disk
        info:    vm disk attach-new command OK

### <a name="step-5---configure-routing-within-the-vms-operating-system"></a>步驟 5 - 設定 VM 的作業系統內的路由

Azure DHCP 會將預設閘道指派給連接至虛擬機器的第一個 (主要) 網路介面。 Azure 不會將預設閘道指派給連接至虛擬機器的其他 (次要) 網路介面。 因此，依預設，您無法與次要網路介面中子網路之外的資源進行通訊。 不過，次要網路介面可與它們的子網路之外的資源通訊。 若要設定次要網路介面的路由，請參閱[在具有多個網路介面的虛擬機器作業系統內路由](virtual-network-network-interface-vm.md)。
