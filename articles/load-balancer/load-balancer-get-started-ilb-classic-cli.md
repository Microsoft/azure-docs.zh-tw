---
title: 建立內部負載平衡器 - Azure 傳統 CLI | Microsoft Docs
description: 瞭解如何使用 Azure 傳統 CLI 在傳統部署模型中建立內部負載平衡器
services: load-balancer
documentationcenter: na
author: genlin
manager: cshepard
editor: ''
tags: azure-service-management
ms.assetid: becbbbde-a118-4269-9444-d3153f00bf34
ms.service: load-balancer
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/18/2018
ms.author: genli
ms.openlocfilehash: fb8929b31fa4325b996ddf4c5ec48e4acb0b930a
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46966884"
---
# <a name="get-started-creating-an-internal-load-balancer-using-the-azure-classic-cli"></a>開始使用 Azure 傳統 CLI 建立內部負載平衡器

> [!div class="op_single_selector"]
> * [PowerShell](../load-balancer/load-balancer-get-started-ilb-classic-ps.md)
> * [Azure CLI](../load-balancer/load-balancer-get-started-ilb-classic-cli.md)
> * [雲端服務](../load-balancer/load-balancer-get-started-ilb-classic-cloud.md)

[!INCLUDE [load-balancer-get-started-ilb-intro-include.md](../../includes/load-balancer-get-started-ilb-intro-include.md)]

> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有二種：[Resource Manager 和傳統](../azure-resource-manager/resource-manager-deployment-model.md)。  本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 了解如何[使用 Resource Manager 模型執行這些步驟](load-balancer-get-started-ilb-arm-cli.md)。

[!INCLUDE [load-balancer-get-started-ilb-scenario-include.md](../../includes/load-balancer-get-started-ilb-scenario-include.md)]

## <a name="to-create-an-internal-load-balancer-set-for-virtual-machines"></a>建立虛擬機器的內部負載平衡器集

若要建立內部負載平衡器集以及向其傳送流量的伺服器，您必須執行下列動作：

1. 建立會是連入流量端點的內部負載平衡執行個體，連入流量會在負載平衡集合的不同伺服器之間進行負載平衡。
2. 新增對應到虛擬機器 (可接收連入流量) 的端點。
3. 設定伺服器以將其流量傳送到內部負載平衡執行個體的虛擬 IP (VIP) 位址。

## <a name="step-by-step-creating-an-internal-load-balancer-using-classic-cli"></a>使用傳統 CLI 逐步建立內部負載平衡器

本指南會根據上述案例，說明如何建立內部負載平衡器。

1. 如果您從未使用過傳統 CLI，請參閱 [安裝和設定 Azure CLI](../cli-install-nodejs.md)，並依照指示進行，直到選取您的 Azure 帳戶和訂用帳戶為止。
2. 執行 **azure config mode** 命令，以切換為傳統模式，如下所示。

    ```azurecli
    azure config mode asm
    ```

    預期的輸出：

        info:    New mode is asm

## <a name="create-endpoint-and-load-balancer-set"></a>建立端點與負載平衡器集

此案例假設在稱為「mytestcloud」的雲端服務中，具有虛擬機器「DB1」和「DB2」。 這些虛擬機器使用稱為我的「testvnet」，且具子網路「subnet-1」的虛擬網路。

本指南會使用連接埠 1433 作為私用連接埠與本機連接埠，以建立內部負載平衡器集。

此為常見案例：若您在後端具有 SQL 虛擬機器，則可使用內部負載平衡器確保不會透過公用 IP 位址直接公開資料庫伺服器。

### <a name="step-1"></a>步驟 1

使用 `azure network service internal-load-balancer add`建立內部負載平衡器集。

```azurecli
azure service internal-load-balancer add --serviceName mytestcloud --internalLBName ilbset --subnet-name subnet-1 --static-virtualnetwork-ipaddress 192.168.2.7
```

如需詳細資訊，請參閱 `azure service internal-load-balancer --help` 。

您可使用 `azure service internal-load-balancer list` *cloud service name*命令，檢查內部負載平衡器屬性。

以下遵循輸出範例：

    azure service internal-load-balancer list my-testcloud
    info:    Executing command service internal-load-balancer list
    + Getting cloud service deployment
    data:    Name    Type     SubnetName  StaticVirtualNetworkIPAddress
    data:    ------  -------  ----------  -----------------------------
    data:    ilbset  Private  subnet-1    192.168.2.7
    info:    service internal-load-balancer list command OK


### <a name="step-2"></a>步驟 2

您可在新增第一個端點時，設定內部負載平衡器集。 您可以在此步驟中，將端點、虛擬機器和探查連接埠與內部負載平衡器集建立關聯。

```azurecli
azure vm endpoint create db1 1433 --local-port 1433 --protocol tcp --probe-port 1433 --probe-protocol tcp --probe-interval 300 --probe-timeout 600 --internal-load-balancer-name ilbset
```

### <a name="step-3"></a>步驟 3

使用 `azure vm show` *virtual machine name*

```azurecli
azure vm show DB1
```

輸出如下所示：

    azure vm show DB1
    info:    Executing command vm show
    + Getting virtual machines
    data:    DNSName "mytestcloud.cloudapp.net"
    data:    Location "East US 2"
    data:    VMName "DB1"
    data:    IPAddress "192.168.2.4"
    data:    InstanceStatus "ReadyRole"
    data:    InstanceSize "Standard_D1"
    data:    Image "a699494373c04fc0bc8f2bb1389d6106__Windows-Server-2012-R2-20151022-en.us-127GB.vhd"
    data:    OSDisk hostCaching "ReadWrite"
    data:    OSDisk name "db1-DB1-0-201511120457370846"
    data:    OSDisk mediaLink "https://XXXX.blob.core.windows.net/vhd"
    data:    OSDisk sourceImageName "a699494373c04fc0bc8f2bb1389d6106__Windows-Server-2012-R2-20151022-en.us-127GB.vhd"
    data:    OSDisk operatingSystem "Windows"
    data:    OSDisk iOType "Standard"
    data:    ReservedIPName ""
    data:    VirtualIPAddresses 0 address "137.116.64.107"
    data:    VirtualIPAddresses 0 name "db1ContractContract"
    data:    VirtualIPAddresses 0 isDnsProgrammed true
    data:    VirtualIPAddresses 1 address "192.168.2.7"
    data:    VirtualIPAddresses 1 name "ilbset"
    data:    Network Endpoints 0 localPort 5986
    data:    Network Endpoints 0 name "PowerShell"
    data:    Network Endpoints 0 port 5986
    data:    Network Endpoints 0 protocol "tcp"
    data:    Network Endpoints 0 virtualIPAddress "137.116.64.107"
    data:    Network Endpoints 0 enableDirectServerReturn false
    data:    Network Endpoints 1 localPort 3389
    data:    Network Endpoints 1 name "Remote Desktop"
    data:    Network Endpoints 1 port 60173
    data:    Network Endpoints 1 protocol "tcp"
    data:    Network Endpoints 1 virtualIPAddress "137.116.64.107"
    data:    Network Endpoints 1 enableDirectServerReturn false
    data:    Network Endpoints 2 localPort 1433
    data:    Network Endpoints 2 name "tcp-1433-1433"
    data:    Network Endpoints 2 port 1433
    data:    Network Endpoints 2 loadBalancerProbe port 1433
    data:    Network Endpoints 2 loadBalancerProbe protocol "tcp"
    data:    Network Endpoints 2 loadBalancerProbe intervalInSeconds 300
    data:    Network Endpoints 2 loadBalancerProbe timeoutInSeconds 600
    data:    Network Endpoints 2 protocol "tcp"
    data:    Network Endpoints 2 virtualIPAddress "192.168.2.7"
    data:    Network Endpoints 2 enableDirectServerReturn false
    data:    Network Endpoints 2 loadBalancerName "ilbset"
    info:    vm show command OK

## <a name="create-a-remote-desktop-endpoint-for-a-virtual-machine"></a>為虛擬機器建立遠端桌面端點

您可以使用 `azure vm endpoint create`建立遠端桌面端點，針對特定的虛擬機器將網路流量從公用連接埠轉送至本機連接埠。

```azurecli
azure vm endpoint create web1 54580 -k 3389
```

## <a name="remove-virtual-machine-from-load-balancer"></a>從負載平衡器移除虛擬機器

您可刪除關聯的端點，以將虛擬機器自內部負載平衡器集移除。 一旦移除端點，虛擬機器就不再屬於負載平衡器集。

透過上述範例，您可以使用命令 `azure vm endpoint delete`從內部負載平衡器「ilbset」移除針對虛擬機器「DB1」所建立的端點。

```azurecli
azure vm endpoint delete DB1 tcp-1433-1433
```

如需詳細資訊，請參閱 `azure vm endpoint --help` 。

## <a name="next-steps"></a>後續步驟

[使用來源 IP 同質性設定負載平衡器分配模式](load-balancer-distribution-mode.md)

[設定負載平衡器的閒置 TCP 逾時設定](load-balancer-tcp-idle-timeout.md)
