---
title: 建立使用 IPv6 的公用負載平衡器 - Azure CLI | Microsoft Docs
description: 了解如何使用 Azure CLI 建立採用 IPv6 的公用負載平衡器。
services: load-balancer
documentationcenter: na
author: KumudD
manager: jeconnoc
tags: azure-resource-manager
keywords: ipv6, azure load balancer, 雙重堆疊, 公用 ip, 原生 ipv6, 行動, iot
ms.assetid: a1957c9c-9c1d-423e-9d5c-d71449bc1f37
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/25/2018
ms.author: kumud
ms.openlocfilehash: 39a3f1a400abae4a9a30c07b7352518b55d0daf1
ms.sourcegitcommit: b5ac31eeb7c4f9be584bb0f7d55c5654b74404ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/23/2018
ms.locfileid: "42746813"
---
# <a name="create-a-public-load-balancer-with-ipv6-using-azure-cli"></a>使用 Azure CLI 建立採用 IPv6 的公用負載平衡器


Azure 負載平衡器是第 4 層 (TCP、UDP) 負載平衡器。 負載平衡器可藉由在負載平衡器集合中，將連入流量分散於雲端服務或虛擬機器中狀況良好的服務執行個體之間，來提供高可用性。 負載平衡器也會在多個連接埠或多個 IP 位址 (或兩者) 上顯示這些服務。

## <a name="example-deployment-scenario"></a>範例部署案例

下圖說明使用本文所述範例範本部署的負載平衡解決方案。

![負載平衡器案例](./media/load-balancer-ipv6-internet-cli/lb-ipv6-scenario-cli.png)

在此案例中，您將建立下列 Azure 資源：

* 兩部虛擬機器 (VM)
* 虛擬網路介面，用於每個已指派 IPv4 和 IPv6 位址的 VM
* 配置有 IPv4 和 IPv6 公用 IP 位址的公用負載平衡器
* 包含兩個 VM 的可用性設定組
* 兩個負載平衡規則，用以對應公用 VIP 至私人端點

## <a name="deploy-the-solution-by-using-azure-cli"></a>使用 Azure CLI 來部署解決方案

下列步驟說明如何使用 Azure CLI 建立公用負載平衡器。 使用 CLI 時，您會個別建立並設定物件，然後將它們放在一起來建立資源。

若要部署負載平衡器，請建立並設定下列物件：

* **前端 IP 組態**：包含傳入網路流量的公用 IP 位址。
* **後端位址集區**：包含虛擬機器的網路介面 (NIC)，可從負載平衡器接收網路流量。
* **負載平衡規則**：包含將負載平衡器上的公用連接埠對應至後端位址集區中的連接埠的規則。
* **輸入 NAT 規則**：包含網路位址轉譯 (NAT) 規則，可將負載平衡器上的公用連接埠對應至後端位址集區中特定虛擬機器的連接埠。
* **探查**：包含用來檢查後端位址集區中虛擬機器執行個體可用性的健全狀況探查。

## <a name="set-up-azure-cli"></a>設定 Azure CLI

在此範例中，您會在 PowerShell 命令視窗中執行 Azure CLI 工具。 若要改善可讀性與重複使用，您可以使用 PowerShell 的指令碼處理功能而非 Azure PowerShell Cmdlet。

1. [安裝和設定 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)，方法是遵循所連結文章內的步驟並登入 Azure 帳戶。

2. 設定 PowerShell 變數以搭配使用 Azure CLI 命令：

    ```powershell
    $subscriptionid = "########-####-####-####-############"  # enter subscription id
    $location = "southcentralus"
    $rgName = "pscontosorg1southctrlus09152016"
    $vnetName = "contosoIPv4Vnet"
    $vnetPrefix = "10.0.0.0/16"
    $subnet1Name = "clicontosoIPv4Subnet1"
    $subnet1Prefix = "10.0.0.0/24"
    $subnet2Name = "clicontosoIPv4Subnet2"
    $subnet2Prefix = "10.0.1.0/24"
    $dnsLabel = "contoso09152016"
    $lbName = "myIPv4IPv6Lb"
    ```

## <a name="create-a-resource-group-a-load-balancer-a-virtual-network-and-subnets"></a>建立資源群組、負載平衡器、虛擬網路和子網路

1. 建立資源群組：

    ```azurecli
    az group create --name $rgName --location $location
    ```

2. 建立負載平衡器：

    ```azurecli
    $lb = az network lb create --resource-group $rgname --location $location --name $lbName
    ```

3. 建立虛擬網路：

    ```azurecli
    $vnet = az network vnet create  --resource-group $rgname --name $vnetName --location $location --address-prefixes $vnetPrefix
    ```

4. 在此虛擬網路中，建立兩個子網路：

    ```azurecli
    $subnet1 = az network vnet subnet create --resource-group $rgname --name $subnet1Name --address-prefix $subnet1Prefix --vnet-name $vnetName
    $subnet2 = az network vnet subnet create --resource-group $rgname --name $subnet2Name --address-prefix $subnet2Prefix --vnet-name $vnetName
    ```

## <a name="create-public-ip-addresses-for-the-front-end-pool"></a>建立前端集區的公用 IP 位址

1. 設定 PowerShell 變數：

    ```powershell
    $publicIpv4Name = "myIPv4Vip"
    $publicIpv6Name = "myIPv6Vip"
    ```

2. 建立前端集區的公用 IP 位址：

    ```azurecli
    $publicipV4 = az network public-ip create --resource-group $rgname --name $publicIpv4Name --location $location --version IPv4 --allocation-method Dynamic --dns-name $dnsLabel
    $publicipV6 = az network public-ip create --resource-group $rgname --name $publicIpv6Name --location $location --version IPv6 --allocation-method Dynamic --dns-name $dnsLabel
    ```

    > [!IMPORTANT]
    > 負載平衡器會使用公用 IP 的網域標籤作為其完整網域名稱 (FQDN)。 這是一項來自傳統部署的變更，該部署使用雲端服務名稱作為負載平衡器 FQDN。
    >
    > 在此範例中，FQDN 是 *contoso09152016.southcentralus.cloudapp.azure.com*。

## <a name="create-front-end-and-back-end-pools"></a>建立前端和後端集區

在本節中，您會建立下列 IP 集區：
* 前端 IP 集區，接收負載平衡器上的連入網路流量。
* 後端 IP 集區，前端集區在其中傳送負載平衡網路流量。

1. 設定 PowerShell 變數：

    ```powershell
    $frontendV4Name = "FrontendVipIPv4"
    $frontendV6Name = "FrontendVipIPv6"
    $backendAddressPoolV4Name = "BackendPoolIPv4"
    $backendAddressPoolV6Name = "BackendPoolIPv6"
    ```

2. 建立前端 IP 集區，並將它與在上個步驟中建立的公用 IP 與負載平衡器產生關聯。

    ```azurecli
    $frontendV4 = az network lb frontend-ip create --resource-group $rgname --name $frontendV4Name --public-ip-address $publicIpv4Name --lb-name $lbName
    $frontendV6 = az network lb frontend-ip create --resource-group $rgname --name $frontendV6Name --public-ip-address $publicIpv6Name --lb-name $lbName
    $backendAddressPoolV4 = az network lb address-pool create --resource-group $rgname --name $backendAddressPoolV4Name --lb-name $lbName
    $backendAddressPoolV6 = az network lb address-pool create --resource-group $rgname --name $backendAddressPoolV6Name --lb-name $lbName
    ```

## <a name="create-the-probe-nat-rules-and-load-balancer-rules"></a>建立探查、NAT 規則和負載平衡器規則

此範例會建立下列項目：

* 探查規則，用以檢查連到 TCP 連接埠 80 的連線。
* NAT 規則，用以將連接埠 3389 上的所有傳入流量轉譯為 RDP 的連接埠 3389。\*
* NAT 規則，用以將連接埠 3391 上的所有傳入流量轉譯為遠端桌面通訊協定 (RDP) 的連接埠 3389。\*
* 負載平衡器規則，將連接埠 80 上的所有傳入流量，負載平衡至後端集區中位址的連接埠 80。

\* NAT 規則與負載平衡器後方的特定虛擬機器執行個體產生關聯。 系統會將抵達連接埠 3389 的網路流量傳送給特定虛擬機器和與 NAT 規則關聯的連接埠。 您必須為 NAT 規則指定通訊協定 (UDP 或 TCP)。 您無法對相同連接埠同時指派這兩個通訊協定。

1. 設定 PowerShell 變數：

    ```powershell
    $probeV4V6Name = "ProbeForIPv4AndIPv6"
    $natRule1V4Name = "NatRule-For-Rdp-VM1"
    $natRule2V4Name = "NatRule-For-Rdp-VM2"
    $lbRule1V4Name = "LBRuleForIPv4-Port80"
    $lbRule1V6Name = "LBRuleForIPv6-Port80"
    ```

2. 建立探查。

    下列範例會建立 TCP 探查，它每隔 15 秒會檢查與後端 TCP 連接埠 80 的連線。 連續兩次失敗後，它會將後端資源標記為無法使用。

    ```azurecli
    $probeV4V6 = az network lb probe create --resource-group $rgname --name $probeV4V6Name --protocol tcp --port 80 --interval 15 --threshold 2 --lb-name $lbName
    ```

3. 建立輸入 NAT 規則，允許 RDP 連線到後端資源：

    ```azurecli
    $inboundNatRuleRdp1 = az network lb inbound-nat-rule create --resource-group $rgname --name $natRule1V4Name --frontend-ip-name $frontendV4Name --protocol Tcp --frontend-port 3389 --backend-port 3389 --lb-name $lbName
    $inboundNatRuleRdp2 = az network lb inbound-nat-rule create --resource-group $rgname --name $natRule2V4Name --frontend-ip-name $frontendV4Name --protocol Tcp --frontend-port 3391 --backend-port 3389 --lb-name $lbName
    ```

4. 建立負載平衡器規則，依據接收要求的前端將流量傳送到不同後端連接埠。

    ```azurecli
    $lbruleIPv4 = az network lb rule create --resource-group $rgname --name $lbRule1V4Name --frontend-ip-name $frontendV4Name --backend-pool-name $backendAddressPoolV4Name --probe-name $probeV4V6Name --protocol Tcp --frontend-port 80 --backend-port 80 --lb-name $lbName
    $lbruleIPv6 = az network lb rule create --resource-group $rgname --name $lbRule1V6Name --frontend-ip-name $frontendV6Name --backend-pool-name $backendAddressPoolV6Name --probe-name $probeV4V6Name --protocol Tcp --frontend-port 80 --backend-port 8080 --lb-name $lbName
    ```

5. 檢查您的設定：

    ```azurecli
    az network lb show --resource-group $rgName --name $lbName
    ```

    預期的輸出：

        info:    Executing command network lb show
        info:    Looking up the load balancer "myIPv4IPv6Lb"
        data:    Id                              : /subscriptions/########-####-####-####-############/resourceGroups/pscontosorg1southctrlus09152016/providers/Microsoft.Network/loadBalancers/myIPv4IPv6Lb
        data:    Name                            : myIPv4IPv6Lb
        data:    Type                            : Microsoft.Network/loadBalancers
        data:    Location                        : southcentralus
        data:    Provisioning state              : Succeeded
        data:
        data:    Frontend IP configurations:
        data:    Name             Provisioning state  Private IP allocation  Private IP   Subnet  Public IP
        data:    ---------------  ------------------  ---------------------  -----------  ------  ---------
        data:    FrontendVipIPv4  Succeeded           Dynamic                                     myIPv4Vip
        data:    FrontendVipIPv6  Succeeded           Dynamic                                     myIPv6Vip
        data:
        data:    Probes:
        data:    Name                 Provisioning state  Protocol  Port  Path  Interval  Count
        data:    -------------------  ------------------  --------  ----  ----  --------  -----
        data:    ProbeForIPv4AndIPv6  Succeeded           Tcp       80          15        2
        data:
        data:    Backend Address Pools:
        data:    Name             Provisioning state
        data:    ---------------  ------------------
        data:    BackendPoolIPv4  Succeeded
        data:    BackendPoolIPv6  Succeeded
        data:
        data:    Load Balancing Rules:
        data:    Name                  Provisioning state  Load distribution  Protocol  Frontend port  Backend port  Enable floating IP  Idle timeout in minutes
        data:    --------------------  ------------------  -----------------  --------  -------------  ------------  ------------------  -----------------------
        data:    LBRuleForIPv4-Port80  Succeeded           Default            Tcp       80             80            false               4
        data:    LBRuleForIPv6-Port80  Succeeded           Default            Tcp       80             8080          false               4
        data:
        data:    Inbound NAT Rules:
        data:    Name                 Provisioning state  Protocol  Frontend port  Backend port  Enable floating IP  Idle timeout in minutes
        data:    -------------------  ------------------  --------  -------------  ------------  ------------------  -----------------------
        data:    NatRule-For-Rdp-VM1  Succeeded           Tcp       3389           3389          false               4
        data:    NatRule-For-Rdp-VM2  Succeeded           Tcp       3391           3389          false               4
        info:    network lb show

## <a name="create-nics"></a>建立 NIC

建立 NIC，並將它們與 NAT 規則、負載平衡器規則和探查產生關聯。

1. 設定 PowerShell 變數：

    ```powershell
    $nic1Name = "myIPv4IPv6Nic1"
    $nic2Name = "myIPv4IPv6Nic2"
    $subnet1Id = "/subscriptions/$subscriptionid/resourceGroups/$rgName/providers/Microsoft.Network/VirtualNetworks/$vnetName/subnets/$subnet1Name"
    $subnet2Id = "/subscriptions/$subscriptionid/resourceGroups/$rgName/providers/Microsoft.Network/VirtualNetworks/$vnetName/subnets/$subnet2Name"
    $backendAddressPoolV4Id = "/subscriptions/$subscriptionid/resourceGroups/$rgname/providers/Microsoft.Network/loadbalancers/$lbName/backendAddressPools/$backendAddressPoolV4Name"
    $backendAddressPoolV6Id = "/subscriptions/$subscriptionid/resourceGroups/$rgname/providers/Microsoft.Network/loadbalancers/$lbName/backendAddressPools/$backendAddressPoolV6Name"
    $natRule1V4Id = "/subscriptions/$subscriptionid/resourceGroups/$rgname/providers/Microsoft.Network/loadbalancers/$lbName/inboundNatRules/$natRule1V4Name"
    $natRule2V4Id = "/subscriptions/$subscriptionid/resourceGroups/$rgname/providers/Microsoft.Network/loadbalancers/$lbName/inboundNatRules/$natRule2V4Name"
    ```

2. 建立的每個後端 NIC，並新增 IPv6 組態：

    ```azurecli
    $nic1 = az network nic create --name $nic1Name --resource-group $rgname --location $location --private-ip-address-version "IPv4" --subnet $subnet1Id --lb-address-pools $backendAddressPoolV4Id --lb-inbound-nat-rules $natRule1V4Id
    $nic1IPv6 = az network nic ip-config create --resource-group $rgname --name "IPv6IPConfig" --private-ip-address-version "IPv6" --lb-address-pools $backendAddressPoolV6Id --nic-name $nic1Name

    $nic2 = az network nic create --name $nic2Name --resource-group $rgname --location $location --private-ip-address-version "IPv4" --subnet $subnet2Id --lb-address-pools $backendAddressPoolV4Id --lb-inbound-nat-rules $natRule2V4Id
    $nic2IPv6 = az network nic ip-config create --resource-group $rgname --name "IPv6IPConfig" --private-ip-address-version "IPv6" --lb-address-pools $backendAddressPoolV6Id --nic-name $nic2Name
    ```

## <a name="create-the-back-end-vm-resources-and-attach-each-nic"></a>建立後端 VM 資源並連接每個 NIC

若要建立 VM，您必須有儲存體帳戶。 為了負載平衡，VM 必須是可用性設定組的成員。 如需建立 VM 的詳細資訊，請參閱[使用 PowerShell 建立 Azure VM](../virtual-machines/virtual-machines-windows-ps-create.md?toc=%2fazure%2fload-balancer%2ftoc.json)。

1. 設定 PowerShell 變數：

    ```powershell
    $availabilitySetName = "myIPv4IPv6AvailabilitySet"
    $vm1Name = "myIPv4IPv6VM1"
    $vm2Name = "myIPv4IPv6VM2"
    $nic1Id = "/subscriptions/$subscriptionid/resourceGroups/$rgname/providers/Microsoft.Network/networkInterfaces/$nic1Name"
    $nic2Id = "/subscriptions/$subscriptionid/resourceGroups/$rgname/providers/Microsoft.Network/networkInterfaces/$nic2Name"
    $imageurn = "MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:latest"
    $vmUserName = "vmUser"
    $mySecurePassword = "PlainTextPassword*1"
    ```

    > [!WARNING]
    > 此範例使用純文字的 VM 使用者名稱和密碼。 當您以純文字使用這些認證時，請務必謹慎。 如需在 PowerShell 中更安全處理認證的做法，請參閱 [`Get-Credential`](https://technet.microsoft.com/library/hh849815.aspx) Cmdlet。

2. 建立可用性設定組：

    ```azurecli
    $availabilitySet = az vm availability-set create --name $availabilitySetName --resource-group $rgName --location $location
    ```

3. 建立與 NIC 關聯的虛擬機器：

    ```azurecli
    az vm create --resource-group $rgname --name $vm1Name --image $imageurn --admin-username $vmUserName --admin-password $mySecurePassword --nics $nic1Id --location $location --availability-set $availabilitySetName --size "Standard_A1" 

    az vm create --resource-group $rgname --name $vm2Name --image $imageurn --admin-username $vmUserName --admin-password $mySecurePassword --nics $nic2Id --location $location --availability-set $availabilitySetName --size "Standard_A1" 
    ```

## <a name="next-steps"></a>後續步驟

[開始設定內部負載平衡器](load-balancer-get-started-ilb-arm-cli.md)  
[設定負載平衡器分配模式](load-balancer-distribution-mode.md)  
[設定負載平衡器的閒置 TCP 逾時設定](load-balancer-tcp-idle-timeout.md)
