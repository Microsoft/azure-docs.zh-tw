---
title: 具有 App Service 環境的多層式安全性架構
description: 實作具有 App Service 環境的多層式安全性架構。
services: app-service
documentationcenter: ''
author: stefsch
manager: erikre
editor: ''
ms.assetid: 73ce0213-bd3e-4876-b1ed-5ecad4ad5601
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/30/2016
ms.author: stefsch
ms.openlocfilehash: 29c928c7d81eb3a2532f735be9132b49db5da373
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/20/2018
ms.locfileid: "34356200"
---
# <a name="implementing-a-layered-security-architecture-with-app-service-environments"></a>實作具有 App Service 環境的多層式安全性架構。
## <a name="overview"></a>概觀
由於 App Service 環境提供部署至虛擬網路的隔離執行階段環境，因此開發人員能夠建立多層式安全性架構，針對每個實體應用程式層提供不同層級的網路存取。

常見的需求之一，是要隱藏對 API 後端的一般網際網路存取，而只允許由上游 Web 應用程式呼叫 API。  [網路安全性群組 (NSG)][NetworkSecurityGroups] 可用於包含 App Service 環境的子網路，用以限制對 API 應用程式的公用存取。

下圖顯示的範例架構，具有部署在 App Service 環境上的 WebAPI 型應用程式。  三個不同的 Web 應用程式執行個體部署在三個不同的 App Service 環境上，使後端呼叫相同的 WebAPI 應用程式。

![概念性架構][ConceptualArchitecture] 

綠色加號指出包含 "apiase" 之子網路上的網路安全性群組，允許來自於上游 Web 應用程式的傳入呼叫，及其本身的呼叫。  不過，相同的網路安全群組明確拒絕網際網路對一般輸入流量的存取。 

此文的其餘部分將逐步解說在包含 "apiase" 的子網路上設定網路安全群組所需的步驟。

## <a name="determining-the-network-behavior"></a>判斷網路行為
若要知道需要哪些網路安全性規則，您必須判斷哪些網路用戶端將能夠聯繫包含 API 應用程式的 App Service 服務環境，而哪些用戶端將遭到封鎖。

由於[網路安全性群組 (NSG)][NetworkSecurityGroups] 會套用至子網路，而 App Service 環境部署在子網路中，因此 NSG 中包含的規則將會套用至**所有**在 App Service 環境上執行的應用程式。  舊本文的範例架構而言，一旦將網路安全性群組套用至包含 "apiase" 的子網路後，所有在 "apiase" App Service 環境上執行的應用程式都將受到相同安全性規則集的保護。 

* **判斷上游呼叫端的輸出 IP 位址：** 上游呼叫端的 IP 位址為何？  這些位址在 NSG 中必須明確地允許存取。  由於 App Service 環境之間的呼叫會被視為「網際網路」呼叫，所以指派給這三個上游 App Service 環境的輸出 IP 位址，在 "apiase" 子網路的 NSG 中必須是允許存取的。   若想進一步了解如何判斷在 App Service 環境中執行之應用程式的輸出 IP 位址，請參閱[網路架構][NetworkArchitecture]概觀文章。
* **後端 API 應用程式是否需要呼叫本身？**  後端應用程式有時候需要呼叫本身，這是常會被忽略而難以察覺的情況。  如果 App Service 環境上的後端 API 應用程式需要呼叫本身，也會將其視為「網際網路」呼叫。  在範例架構中，這必須允許 "apiase" App Service 環境的輸出 IP 位址進行存取。

## <a name="setting-up-the-network-security-group"></a>設定網路安全性群組
得知輸出 IP 位址集後，下一步是要建構網路安全群組。  您可以針對以 Resource Manager 為基礎的虛擬網路以及傳統虛擬網路建立網路安全性群組。  下列範例說明如何在傳統虛擬網路上使用 Powershell 建立和設定 NSG。

在範例架構中，環境位於美國中南部，因此會在該區域中建立空的 NSG：

    New-AzureNetworkSecurityGroup -Name "RestrictBackendApi" -Location "South Central US" -Label "Only allow web frontend and loopback traffic"

首先要為 Azure 管理基礎結構新增明確的允許規則，如 App Service 環境的[輸入流量][InboundTraffic]相關文章所說明。

    #Open ports for access by Azure management infrastructure
    Get-AzureNetworkSecurityGroup -Name "RestrictBackendApi" | Set-AzureNetworkSecurityRule -Name "ALLOW AzureMngmt" -Type Inbound -Priority 100 -Action Allow -SourceAddressPrefix 'INTERNET' -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '454-455' -Protocol TCP

接著要新增兩個規則，以允許從第一個上游 App Service 環境 ("fe1ase") 進行 HTTP 和 HTTPS 呼叫。

    #Grant access to requests from the first upstream web front-end
    Get-AzureNetworkSecurityGroup -Name "RestrictBackendApi" | Set-AzureNetworkSecurityRule -Name "ALLOW HTTP fe1ase" -Type Inbound -Priority 200 -Action Allow -SourceAddressPrefix '65.52.xx.xyz'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '80' -Protocol TCP
    Get-AzureNetworkSecurityGroup -Name "RestrictBackendApi" | Set-AzureNetworkSecurityRule -Name "ALLOW HTTPS fe1ase" -Type Inbound -Priority 300 -Action Allow -SourceAddressPrefix '65.52.xx.xyz'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '443' -Protocol TCP

為第二個和第三個上游 App Service 環境 ("fe2ase" 和 "fe3ase") 重複相同步驟。

    #Grant access to requests from the second upstream web front-end
    Get-AzureNetworkSecurityGroup -Name "RestrictBackendApi" | Set-AzureNetworkSecurityRule -Name "ALLOW HTTP fe2ase" -Type Inbound -Priority 400 -Action Allow -SourceAddressPrefix '191.238.xyz.abc'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '80' -Protocol TCP
    Get-AzureNetworkSecurityGroup -Name "RestrictBackendApi" | Set-AzureNetworkSecurityRule -Name "ALLOW HTTPS fe2ase" -Type Inbound -Priority 500 -Action Allow -SourceAddressPrefix '191.238.xyz.abc'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '443' -Protocol TCP

    #Grant access to requests from the third upstream web front-end
    Get-AzureNetworkSecurityGroup -Name "RestrictBackendApi" | Set-AzureNetworkSecurityRule -Name "ALLOW HTTP fe3ase" -Type Inbound -Priority 600 -Action Allow -SourceAddressPrefix '23.98.abc.xyz'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '80' -Protocol TCP
    Get-AzureNetworkSecurityGroup -Name "RestrictBackendApi" | Set-AzureNetworkSecurityRule -Name "ALLOW HTTPS fe3ase" -Type Inbound -Priority 700 -Action Allow -SourceAddressPrefix '23.98.abc.xyz'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '443' -Protocol TCP

最後，授與後端 API 的 App Service 環境輸出 IP 位址的存取權，使其能夠回呼本身。

    #Allow apps on the apiase environment to call back into itself
    Get-AzureNetworkSecurityGroup -Name "RestrictBackendApi" | Set-AzureNetworkSecurityRule -Name "ALLOW HTTP apiase" -Type Inbound -Priority 800 -Action Allow -SourceAddressPrefix '70.37.xyz.abc'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '80' -Protocol TCP
    Get-AzureNetworkSecurityGroup -Name "RestrictBackendApi" | Set-AzureNetworkSecurityRule -Name "ALLOW HTTPS apiase" -Type Inbound -Priority 900 -Action Allow -SourceAddressPrefix '70.37.xyz.abc'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '443' -Protocol TCP

不需要任何其他網路安全性，因為依預設每個 NSG 有一組預設規則會封鎖來自網際網路的輸入存取。

網路安全性群組中的完整規則清單顯示如下。  請留意最後一個規則 (反白顯示) 如何封鎖來自所有呼叫端 (已明確獲取存取權的呼叫端除外) 的輸入存取。

![NSG 組態][NSGConfiguration] 

最後一個步驟是將 NSG 套用至包含 "apiase" App Service 環境的子網路。

     #Apply the NSG to the backend API subnet
    Get-AzureNetworkSecurityGroup -Name "RestrictBackendApi" | Set-AzureNetworkSecurityGroupToSubnet -VirtualNetworkName 'yourvnetnamehere' -SubnetName 'API-ASE-Subnet'

NSG 套用至子網路後，將只有三個上游 App Service 環境以及包含 API 後端的 App Service 環境能夠呼叫 "apiase" 環境。

## <a name="additional-links-and-information"></a>其他連結和資訊
關於 [網路安全性群組](../../virtual-network/security-overview.md)的資訊。

了解[輸出 IP 位址][NetworkArchitecture]和 App Service 環境。

App Service 環境所使用的[網路連接埠][InboundTraffic]。

[!INCLUDE [app-service-web-try-app-service](../../../includes/app-service-web-try-app-service.md)]

<!-- LINKS -->
[NetworkSecurityGroups]: https://azure.microsoft.com/documentation/articles/virtual-networks-nsg/
[NetworkArchitecture]:  app-service-app-service-environment-network-architecture-overview.md
[InboundTraffic]:  app-service-app-service-environment-control-inbound-traffic.md

<!-- IMAGES -->
[ConceptualArchitecture]: ./media/app-service-app-service-environment-layered-security/ConceptualArchitecture-1.png
[NSGConfiguration]:  ./media/app-service-app-service-environment-layered-security/NSGConfiguration-1.png
