---
title: 包含檔案
description: 包含檔案
services: virtual-network
author: genlin
ms.service: virtual-network
ms.topic: include
ms.date: 04/13/2018
ms.author: genli
ms.custom: include file
ms.openlocfilehash: 749cc5a5e5b8417abe602b7e37c103a26cc4dc03
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31805177"
---
## <a name="how-to-create-a-classic-vnet-using-azure-cli"></a>如何使用 Azure CLI 建立傳統 VNet
您可以使用 Azure CLI 透過任何 Windows、Linux 或 OSX 電腦命令提示字元管理 Azure 資源。

1. 如果您從未使用過 Azure CLI，請參閱 [安裝和設定 Azure CLI](../articles/cli-install-nodejs.md) ，並依照指示進行，直到選取您的 Azure 帳戶和訂用帳戶為止。
2. 若要建立 VNet 和子網路，請執行 **azure network vnet create** 命令：
   
            azure network vnet create --vnet TestVNet -e 192.168.0.0 -i 16 -n FrontEnd -p 192.168.1.0 -r 24 -l "Central US"
   
    預期的輸出：
   
            info:    Executing command network vnet create
            + Looking up network configuration
            + Looking up locations
            + Setting network configuration
            info:    network vnet create command OK
   
   * **--vnet**。 即將建立的 VNet 名稱。 在本文案例中為 TestVNet
   * **-e (或 --address-space)**。 VNet 位址空間。 在本文案例中為 192.168.0.0
   * **-i (或 -cidr)**。 CIDR 格式的網路遮罩。 在本文案例中為 16。
   * **-n (或 --subnet-name**)。 第一個子網路的名稱。 在本文案例中為 FrontEnd。
   * **-p (或 --subnet-start-ip)**。 子網路的起始 IP 位址或子網路位址空間。 在本文案例中為 192.168.1.0。
   * **-r (或 --subnet-cidr)**。 CIDR 格式的子網路網路遮罩。 在本文案例中為 24。
   * **-l (或 --location)**。 在其中建立 VNet 的 Azure 區域。 在本文案例中為「美國中部」。
3. 若要建立子網路，請執行 **azure network vnet subnet create** 命令：
   
            azure network vnet subnet create -t TestVNet -n BackEnd -a 192.168.2.0/24
   
    上述命令的預期輸出：
   
            info:    Executing command network vnet subnet create
            + Looking up network configuration
            + Creating subnet "BackEnd"
            + Setting network configuration
            + Looking up the subnet "BackEnd"
            + Looking up network configuration
            data:    Name                            : BackEnd
            data:    Address prefix                  : 192.168.2.0/24
            info:    network vnet subnet create command OK
   
   * **-t (或 --vnet-name)**。 將會建立子網路的 VNet 名稱。 在本文案例中為 TestVNet。
   * **-n (or --name)**。 新子網路的名稱。 在本文案例中為 BackEnd。
   * **-a (或 --address-prefix)**。 子網路 CIDR 區塊。 在本文案例中為 192.168.2.0/24。
4. 若要檢視新 Vnet 的屬性，請執行 **azure network vnet show** 命令：
   
            azure network vnet show
   
    上述命令的預期輸出：
   
            info:    Executing command network vnet show
            Virtual network name: TestVNet
            + Looking up the virtual network sites
            data:    Name                            : TestVNet
            data:    Location                        : Central US
            data:    State                           : Created
            data:    Address space                   : 192.168.0.0/16
            data:    Subnets:
            data:      Name                          : FrontEnd
            data:      Address prefix                : 192.168.1.0/24
            data:
            data:      Name                          : BackEnd
            data:      Address prefix                : 192.168.2.0/24
            data:
            info:    network vnet show command OK

