---
title: 在 Azure Batch 集區中設定節點端點 | Microsoft Docs
description: 如何設定或停用 Azure Batch 集區內之計算節點上的 SSH 或 RDP 連接埠存取權。
services: batch
author: dlepow
manager: jeconnoc
ms.service: batch
ms.topic: article
ms.date: 02/13/2018
ms.author: danlep
ms.openlocfilehash: 5898206761e5029f94b6d1f1b48223481ae2ca13
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/20/2018
ms.locfileid: "34358722"
---
# <a name="configure-or-disable-remote-access-to-compute-nodes-in-an-azure-batch-pool"></a>設定或停用 Azure Batch 集區內計算節點的遠端存取權

根據預設，Batch 允許具備網路連線能力的[節點使用者](/rest/api/batchservice/computenode/adduser)從外部連線到 Batch 集區內的計算節點。 例如，使用者可利用遠端桌面 (RDP) 透過連接埠 3389 連線至 Windows 集區中的計算節點。 同樣地，根據預設，使用者可利用安全殼層 (SSH) 透過連接埠 22 連線至 Linux 集區中的計算節點。 

在您的環境中，您可能需要限制或停用這些預設的外部存取設定。 您可以使用 Batch API 設定 [PoolEndpointConfiguration](/rest/api/batchservice/pool/add#poolendpointconfiguration) 屬性，來修改這些設定。 

## <a name="about-the-pool-endpoint-configuration"></a>關於集區端點設定
端點設定包含一或多個前端連接埠的[網路位址轉譯 (NAT) 集區](/rest/api/batchservice/pool/add#inboundnatpool)。 (請勿將 NAT 集區與計算節點的 Batch 集區混淆。)您可設定每個 NAT 集區以覆寫集區計算節點上的預設連線設定。 

每個 NAT 集區設定都包含一或多個[網路安全性群組 (NSG) 規則](/rest/api/batchservice/pool/add#networksecuritygrouprule)。 每個 NSG 規則皆會允許或拒絕通往端點的特定網路流量。 您可選擇要允許或拒絕所有流量、以[服務標籤](../virtual-network/security-overview.md#service-tags) (例如「網際網路」) 識別的網路流量，或來自特定 IP 位址或子網路的流量。

### <a name="considerations"></a>考量
* 集區端點設定是屬於集區[網路設定](/rest/api/batchservice/pool/add#NetworkConfiguration)的一部分。 網路設定可選擇性地包含將集區加入 [Azure 虛擬網路](batch-virtual-network.md)的設定。 如果您在虛擬網路中設定集區，您可以建立使用虛擬網路中位址設定的 NSG 規則。
* 當您設定 NAT 集區時，可以設定多個 NSG 規則。 系統會依照規則優先順序檢查規則。 一旦套用規則，就不會再測試規則是否符合。


## <a name="example-deny-all-rdp-traffic"></a>範例：拒絕所有 RDP 流量

下列 C# 程式碼片段示範如何設定 Windows 集區中計算節點上的 RDP 端點，以拒絕所有網路流量。 此端點使用範圍 *60000 - 60099* 的連接埠前端集區。 

```csharp
pool.NetworkConfiguration = new NetworkConfiguration
{
    EndpointConfiguration = new PoolEndpointConfiguration(new InboundNatPool[]
    {
      new InboundNatPool("RDP", InboundEndpointProtocol.Tcp, 3389, 60000, 60099, new NetworkSecurityGroupRule[]
        {
            new NetworkSecurityGroupRule(162, NetworkSecurityGroupRuleAccess.Deny, "*"),
        })
    })    
};
```

## <a name="example-deny-all-ssh-traffic-from-the-internet"></a>範例：拒絕來自網際網路的所有 SSH 流量

下列 Python 程式碼片段示範如何設定 Linux 集區中計算節點上的 SSH 端點，以拒絕所有網際網路流量。 此端點使用範圍 *4000 - 4100* 的連接埠前端集區。 

```python
pool.network_configuration=batchmodels.NetworkConfiguration(
    endpoint_configuration=batchmodels.PoolEndpointConfiguration(
        inbound_nat_pools=[batchmodels.InboundNATPool(
            name='SSH',
            protocol='tcp',
            backend_port=22,
            frontend_port_range_start=4000,
            frontend_port_range_end=4100,
            network_security_group_rules=[
                batchmodels.NetworkSecurityGroupRule(
                priority=170,
                access=batchmodels.NetworkSecurityGroupRuleAccess.deny,
                source_address_prefix='Internet'
                )
            ]
        )
        ]
    ) 
)
```

## <a name="example-allow-rdp-traffic-from-a-specific-ip-address"></a>範例：允許來自特定 IP 位址的 RDP 流量

下列 C# 程式碼片段示範如何設定 Windows 集區中計算節點上的 RDP 端點，以只允許來自 IP 位址 *198.51.100.7* 的 RDP 存取。 第二個 NSG 規則會拒絕不相符之 IP 位址的流量。

```csharp
pool.NetworkConfiguration = new NetworkConfiguration
{
    EndpointConfiguration = new PoolEndpointConfiguration(new InboundNatPool[]
    {
        new InboundNatPool("RDP", InboundEndpointProtocol.Tcp, 3389, 7500, 8000, new NetworkSecurityGroupRule[]
        {   
            new NetworkSecurityGroupRule(179,NetworkSecurityGroupRuleAccess.Allow, "198.51.100.7"),
            new NetworkSecurityGroupRule(180,NetworkSecurityGroupRuleAccess.Deny, "*")
        })
    })    
};
```

## <a name="example-allow-ssh-traffic-from-a-specific-subnet"></a>範例：允許來自特定子網路的 SSH 流量

下列 Python 程式碼片段示範如何設定 Linux 集區中計算節點上的 SSH 端點，以只允許來自子網路 *192.168.1.0/24* 的存取。 第二個 NSG 規則會拒絕不相符之子網路的流量。

```python
pool.network_configuration=batchmodels.NetworkConfiguration(
    endpoint_configuration=batchmodels.PoolEndpointConfiguration(
        inbound_nat_pools=[batchmodels.InboundNATPool(
            name='SSH',
            protocol='tcp',
            backend_port=22,
            frontend_port_range_start=4000,
            frontend_port_range_end=4100,
            network_security_group_rules=[
                batchmodels.NetworkSecurityGroupRule(
                priority=170,
                access='allow',
                source_address_prefix='192.168.1.0/24'
                ),
                batchmodels.NetworkSecurityGroupRule(
                priority=175,
                access='deny',
                source_address_prefix='*'
                )
            ]
        )
        ]
    )
)
```

## <a name="next-steps"></a>後續步驟

- 如需 Azure 中 NSG 規則的相關詳細資訊，請參閱[使用網路安全性群組來篩選網路流量](../virtual-network/security-overview.md)。

- 如需 Batch 的深入概觀，請參閱[使用 Batch 開發大規模的平行計算解決方案](batch-api-basics.md)。

