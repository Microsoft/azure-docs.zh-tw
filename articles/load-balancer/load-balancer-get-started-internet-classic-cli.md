---
title: 建立網際網路對向負載平衡器 - Azure 傳統 CLI | Microsoft Docs
description: 了解如何使用 Azure 傳統 CLI 在傳統部署模型中建立網際網路面向的負載平衡器
services: load-balancer
documentationcenter: na
author: genlin
manager: cshepard
tags: azure-service-management
ms.assetid: e433a824-4a8a-44d2-8765-a74f52d4e584
ms.service: load-balancer
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/18/2018
ms.author: genli
ms.openlocfilehash: a91a53070985a24ccd87f16254f652fe48c42c5a
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46988150"
---
# <a name="get-started-creating-an-internet-facing-load-balancer-classic-in-the-azure-classic-cli"></a>開始在 Azure 傳統 CLI 中建立網際網路面向的負載平衡器 (傳統)

> [!div class="op_single_selector"]
> * [PowerShell](../load-balancer/load-balancer-get-started-internet-classic-ps.md)
> * [Azure 傳統 CLI](../load-balancer/load-balancer-get-started-internet-classic-cli.md)
> * [Azure 雲端服務](../load-balancer/load-balancer-get-started-internet-classic-cloud.md)

[!INCLUDE [load-balancer-get-started-internet-intro-include.md](../../includes/load-balancer-get-started-internet-intro-include.md)]

> [!IMPORTANT]
> 使用 Azure 資源之前，請務必了解 Azure 目前有 Azure Resource Manager 和「傳統」兩種部署模型。 在使用任何 Azure 資源之前，請先確認您了解 [部署模型和工具](../azure-classic-rm.md) 。 您可以按一下本文頂端的索引標籤，檢視不同工具的文件。 本文涵蓋之內容包括傳統部署模型。 您也可以 [了解如何使用 Azure 資源管理員建立網際網路面向的負載平衡器](load-balancer-get-started-internet-arm-ps.md)。

[!INCLUDE [requires-classic-cli](../../includes/contains-classic-cli-content.md)]

[!INCLUDE [load-balancer-get-started-internet-scenario-include.md](../../includes/load-balancer-get-started-internet-scenario-include.md)]

## <a name="create-an-internet-facing-load-balancer-using-cli"></a>使用 CLI 建立網際網路對向負載平衡器

本指南根據上述案例說明如何建立網際網路面向的負載平衡器。

1. 如果您從未使用過 Azure 傳統 CLI，請參閱 [安裝和設定 Azure 傳統 CLI](../cli-install-nodejs.md)，並依照指示進行，直到選取您的 Azure 帳戶和訂用帳戶為止。
2. 執行 **azure config mode** 命令，以切換為傳統模式，如下所示。

    ```azurecli
    azure config mode asm
    ```

    預期的輸出：

        info:    New mode is asm

## <a name="create-endpoint-and-load-balancer-set"></a>建立端點與負載平衡器集

此案例假設虛擬機器 "web1" 和 "web2" 已經建立。
本指南將使用連接埠 80 作為公用連接埠，以及 80 作為本機連接埠，建立負載平衡器集。 探查連接埠也設定在連接埠 80 上，並將負載平衡器集稱為 "lbset"

### <a name="step-1"></a>步驟 1

針對虛擬機器 "web1" 使用 `azure network vm endpoint create` 建立第一個端點和負載平衡器集

```azurecli
azure vm endpoint create web1 80 --local-port 80 --protocol tcp --probe-port 80 --load-balanced-set-name lbset
```

### <a name="step-2"></a>步驟 2

將第二個虛擬機器 "web2" 新增到負載平衡器集。

```azurecli
azure vm endpoint create web2 80 --local-port 80 --protocol tcp --probe-port 80 --load-balanced-set-name lbset
```

### <a name="step-3"></a>步驟 3

使用 `azure vm show` 確認負載平衡器設定

```azurecli
azure vm show web1
```

輸出將是：

    data:    DNSName "contoso.cloudapp.net"
    data:    Location "East US"
    data:    VMName "web1"
    data:    IPAddress "10.0.0.5"
    data:    InstanceStatus "ReadyRole"
    data:    InstanceSize "Standard_D1"
    data:    Image "a699494373c04fc0bc8f2bb1389d6106__Windows-Server-2012-R2-2015
    6-en.us-127GB.vhd"
    data:    OSDisk hostCaching "ReadWrite"
    data:    OSDisk name "joaoma-1-web1-0-201509251804250879"
    data:    OSDisk mediaLink "https://XXXXXXXXXXXXXXX.blob.core.windows.
    /vhds/joaomatest-web1-2015-09-25.vhd"
    data:    OSDisk sourceImageName "a699494373c04fc0bc8f2bb1389d6106__Windows-Se
    r-2012-R2-20150916-en.us-127GB.vhd"
    data:    OSDisk operatingSystem "Windows"
    data:    OSDisk iOType "Standard"
    data:    ReservedIPName ""
    data:    VirtualIPAddresses 0 address "XXXXXXXXXXXXXXXX"
    data:    VirtualIPAddresses 0 name "XXXXXXXXXXXXXXXXXXXX"
    data:    VirtualIPAddresses 0 isDnsProgrammed true
    data:    Network Endpoints 0 loadBalancedEndpointSetName "lbset"
    data:    Network Endpoints 0 localPort 80
    data:    Network Endpoints 0 name "tcp-80-80"
    data:    Network Endpoints 0 port 80
    data:    Network Endpoints 0 loadBalancerProbe port 80
    data:    Network Endpoints 0 loadBalancerProbe protocol "tcp"
    data:    Network Endpoints 0 loadBalancerProbe intervalInSeconds 15
    data:    Network Endpoints 0 loadBalancerProbe timeoutInSeconds 31
    data:    Network Endpoints 0 protocol "tcp"
    data:    Network Endpoints 0 virtualIPAddress "XXXXXXXXXXXX"
    data:    Network Endpoints 0 enableDirectServerReturn false
    data:    Network Endpoints 1 localPort 5986
    data:    Network Endpoints 1 name "PowerShell"
    data:    Network Endpoints 1 port 5986
    data:    Network Endpoints 1 protocol "tcp"
    data:    Network Endpoints 1 virtualIPAddress "XXXXXXXXXXXX"
    data:    Network Endpoints 1 enableDirectServerReturn false
    data:    Network Endpoints 2 localPort 3389
    data:    Network Endpoints 2 name "Remote Desktop"
    data:    Network Endpoints 2 port 58081
    info:    vm show command OK

## <a name="create-a-remote-desktop-endpoint-for-a-virtual-machine"></a>為虛擬機器建立遠端桌面端點

您可以使用 `azure vm endpoint create`建立遠端桌面端點，針對特定的虛擬機器將網路流量從公用連接埠轉送至本機連接埠。

```azurecli
azure vm endpoint create web1 54580 -k 3389
```

## <a name="remove-virtual-machine-from-load-balancer"></a>從負載平衡器移除虛擬機器

您必須從虛擬機器刪除與負載平衡器集相關聯的端點。 一旦移除端點，虛擬機器就不再屬於負載平衡器集。

使用上述的範例，您可以使用命令 `azure vm endpoint delete` 從負載平衡器 "lbset" 移除針對虛擬機器 "web1" 所建立的端點。

```azurecli
azure vm endpoint delete web1 tcp-80-80
```

> [!NOTE]
> 您可以使用命令 `azure vm endpoint --help` 探索更多管理端點的選項

## <a name="next-steps"></a>後續步驟

[開始設定內部負載平衡器](load-balancer-get-started-ilb-arm-ps.md)

[設定負載平衡器分配模式](load-balancer-distribution-mode.md)

[設定負載平衡器的閒置 TCP 逾時設定](load-balancer-tcp-idle-timeout.md)
