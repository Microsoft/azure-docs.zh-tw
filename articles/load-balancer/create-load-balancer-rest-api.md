---
title: 使用 REST API 建立 Azure Load Balancer | Microsoft Docs
description: 了解如何使用 REST API 建立 Azure Load Balancer。
services: load-balancer
documentationcenter: na
author: KumudD
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: load-balancer
ms.date: 06/06/2018
ms.author: kumud
ms.openlocfilehash: ca952fa4fbea742121e579b28be35d834f17eade
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44056676"
---
# <a name="create-an-azure-basic-load-balancer-using-rest-api"></a>使用 REST API 建立 Azure 基本負載平衡器

Azure Load Balancer 會根據規則和健康情況探查，將抵達負載平衡器前端的新輸入流程，分送給各個後端集區執行個體。 負載平衡器有兩種 SKU：基本和標準。 若要了解兩個 SKU 版本之間的差異，請參閱 [Load Balancer SKU 比較](load-balancer-overview.md#skus)。
 
此操作說明示範如何使用 [Azure REST API](/rest/api/azure/) 建立 Azure 基本負載平衡器，協助在 Azure 虛擬網路內的多個 VM 間平衡連入要求的負載。 在 [Azure Load Balancer REST 參考](/rest/api/load-balancer/)中可取得完整參考文件和其他範例。
 
## <a name="build-the-request"></a>建立要求
使用下列 HTTP PUT 要求來建立新的 Azure 基本負載平衡器。
 ```HTTP
  PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/loadBalancers/{loadBalancerName}?api-version=2018-02-01
  ```
### <a name="uri-parameters"></a>URI 參數

|名稱  |在  |必要 |類型 |說明 |
|---------|---------|---------|---------|--------|
|subscriptionId   |  path       |  True       |   字串      |  可唯一識別 Microsoft Azure 訂用帳戶的訂用帳戶認證。 訂用帳戶識別碼會構成每個服務呼叫 URI 的一部分。      |
|resourceGroupName     |     path    | True        |  字串       |   資源群組的名稱。     |
|loadBalancerName     |  path       |      True   |    字串     |    負載平衡器的名稱。    |
|api-version    |   query     |  True       |     字串    |  用戶端 API 版本。      |



### <a name="request-body"></a>Request body

唯一必要的參數是 `location`。 如果您未定義 *SKU* 版本，預設會建立基本負載平衡器。  使用[選擇性參數](https://docs.microsoft.com/rest/api/load-balancer/loadbalancers/createorupdate#request-body)來自訂負載平衡器。

| 名稱 | 類型 | 說明 |
| :--- | :--- | :---------- |
| location | 字串 | 資源位置。 使用 [List Locations](https://docs.microsoft.com/rest/api/resources/subscriptions/listlocations) 作業取得目前的位置清單。 |


## <a name="example-create-and-update-a-basic-load-balancer"></a>範例：建立及更新基本負載平衡器

在此範例中，請先建立基本負載平衡器及其資源。 接下來，您會設定負載平衡器資源，包括前端 IP 組態、後端位址集區、負載平衡規則、健康情況探查，以及輸入 NAT 規則。

使用以下範例建立負載平衡器之前，請先建立名為 vnetlb 的虛擬網路，其在 [美國東部] 位置名為 rg1 的資源群組中有一個名為 subnetlb 的子網路。

### <a name="step-1-create-a-basic-load-balancer"></a>步驟 1. 建立基本負載平衡器
在此步驟中，您會在 [美國東部] 位置的 rg1 資源群組中建立名為 lb 的基本負載平衡器。
#### <a name="sample-request"></a>範例要求

  ```HTTP    
  PUT https://management.azure.com/subscriptions/subid/resourceGroups/rg1/providers/Microsoft.Network/loadBalancers/lb?api-version=2018-02-01
  ```
#### <a name="request-body"></a>Request body

  ```JSON
   {
    "location": "eastus",
   }
  ```
### <a name="step-2-configure-load-balancer-resources"></a>步驟 2. 設定負載平衡器資源
在這個步驟中，您會設定負載平衡器 Ib 資源，包括前端 IP 組態 (fe-lb)、後端位址集區 (be-lb)、負載平衡規則 (rulelb)、健康情況探查 (probe-lb)，以及輸入 NAT 規則 (in-nat-rule)。
#### <a name="sample-request"></a>範例要求

  ```HTTP    
  PUT https://management.azure.com/subscriptions/subid/resourceGroups/rg1/providers/Microsoft.Network/loadBalancers/lb?api-version=2018-02-01
  ```
#### <a name="request-body"></a>Request body

  ```JSON
{
  "properties": {
    "frontendIPConfigurations": [
      {
        "name": "fe-lb",
        "properties": {
          "subnet": {
            "id": "/subscriptions/subid/resourceGroups/rg1/providers/Microsoft.Network/virtualNetworks/vnetlb/subnets/subnetlb"
          },
          "loadBalancingRules": [
            {
              "id": "/subscriptions/subid/resourceGroups/rg1/providers/Microsoft.Network/loadBalancers/lb/loadBalancingRules/rulelb"
            }  ],
          "inboundNatRules": [
            {
              "id": "/subscriptions/subid/resourceGroups/rg1/providers/Microsoft.Network/loadBalancers/lb/inboundNatRules/in-nat-rule"
            }  ]  }  }  ],
    "backendAddressPools": [
      {
        "name": "be-lb",
        "properties": {
          "loadBalancingRules": [
            {
              "id": "/subscriptions/subid/resourceGroups/rg1/providers/Microsoft.Network/loadBalancers/lb/loadBalancingRules/rulelb"
            }  ]   }   }   ],
    "loadBalancingRules": [
      {
        "name": "rulelb",
        "properties": {
          "frontendIPConfiguration": {
            "id": "/subscriptions/subid/resourceGroups/rg1/providers/Microsoft.Network/loadBalancers/lb/frontendIPConfigurations/fe-lb"
          },
          "frontendPort": 80,
          "backendPort": 80,
          "enableFloatingIP": true,
          "idleTimeoutInMinutes": 15,
          "protocol": "Tcp",
          "loadDistribution": "Default",
          "backendAddressPool": {
            "id": "/subscriptions/subid/resourceGroups/rg1/providers/Microsoft.Network/loadBalancers/lb/backendAddressPools/be-lb"
          },
          "probe": {
            "id": "/subscriptions/subid/resourceGroups/rg1/providers/Microsoft.Network/loadBalancers/lb/probes/probe-lb"
          }  }  }  ],
    "probes": [
      {
        "name": "probe-lb",
        "properties": {
          "protocol": "Http",
          "port": 80,
          "requestPath": "healthcheck.aspx",
          "intervalInSeconds": 15,
          "numberOfProbes": 2,
          "loadBalancingRules": [
            {
              "id": "/subscriptions/subid/resourceGroups/rg1/providers/Microsoft.Network/loadBalancers/lb/loadBalancingRules/rulelb"
            }  ]  }  } ],
    "inboundNatRules": [
      {
        "name": "in-nat-rule",
        "properties": {
          "frontendIPConfiguration": {
            "id": "/subscriptions/subid/resourceGroups/rg1/providers/Microsoft.Network/loadBalancers/lb/frontendIPConfigurations/fe-lb"
          },
          "frontendPort": 3389,
          "backendPort": 3389,
          "enableFloatingIP": true,
          "idleTimeoutInMinutes": 15,
          "protocol": "Tcp"
        } } ],
    "inboundNatPools": [],
    "outboundNatRules": []
  }  }
```
