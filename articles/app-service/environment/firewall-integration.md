---
title: 鎖定 Azure App Service Environment 的輸出流量
description: 描述如何與 Azure 防火牆整合來保護輸出流量
services: app-service
documentationcenter: na
author: ccompy
manager: stefsch
ms.assetid: 955a4d84-94ca-418d-aa79-b57a5eb8cb85
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/24/2018
ms.author: ccompy
ms.openlocfilehash: bcd5cb15c003ea146263a6e71a280f7da42ba830
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47167641"
---
# <a name="app-service-environment-certificates-overview"></a>App Service Environment 憑證概觀 

App Service Environment (ASE) 有一些外部相依性，它需要能夠存取，才能正確運作。 ASE 位於客戶 Azure 虛擬網路 (VNet)。 客戶必須允許 ASE 相依性流量，這對於想要鎖定從其 VNet 所有輸出流量的客戶而言會是個問題。

ASE 有一些輸入相依性。 輸入的管理流量不能透過防火牆裝置傳送。 此流量的來源位址已知，且發佈在 [App Service Environment 管理位址](https://docs.microsoft.com/azure/app-service/environment/management-addresses)文件。 您可以建立具有該資訊的網路安全性群組規則，來保護輸入的流量。

ASE 輸出相依性幾乎完全使用 FQDN 定義，它背後並沒有靜態位址。 缺乏靜態位址表示網路安全性群組 (NSG) 不能用來鎖定來自 ASE 的輸出流量。 地址經常變更，使得無法根據目前的解析度來設定規則，以用來建立 NSG。 

保護輸出位址的解決方案在於使用可以根據網域名稱控制輸出流量的防火牆裝置。 Azure 網路小組已經將新的網路裝置放入預覽，稱為 Azure 防火牆。 Azure 防火牆可以根據目的地的 DNS 名稱，來限制輸出 HTTP 和 HTTPS 流量。  

## <a name="configuring-azure-firewall-with-your-ase"></a>使用 ASE 設定 Azure 防火牆 

使用 Azure 防火牆來鎖定 ASE 輸出流量的步驟如下：

1. 在您的 ASE 所在或將來所在的 VNet 中，建立 Azure 防火牆。 [Azure 防火牆文件](https://docs.microsoft.com/azure/firewall/)
2. 從 Azure 防火牆 UI，選取 [App Service Environment FQDN] 標籤
3. 建立路由表，其中具有來自 [App Service Environment 管理位址]( https://docs.microsoft.com/azure/app-service/environment/management-addresses)的管理位址，以及網際網路的下一個躍點。 需要路由表項目，才能避免發生非對稱式路由問題。 
4. 針對底下＜IP 位址相依性＞所註明的 IP 位址相依性，新增具有網際網路下一個躍點的路由。 
5. 在路由表中新增 0.0.0.0/0 的路由，並且以 Azure 防火牆網路設備作為下一個躍點
6. 將您的 ASE 子網路建立 Azure SQL 和 Azure 儲存體的服務端點
7. 將您建立的路由表指派給 ASE 子網路  

## <a name="application-traffic"></a>應用程式流量 

上述步驟可讓您的 ASE 正常運作。 您仍然需要設定以容納應用程式需求。 針對已設定 Azure 防火牆的 ASE，其中的應用程式有兩個問題。  

- 應用程式相依性 FQDN 必須新增至 Azure 防火牆或路由表
- 必須為流量來源位址建立路由，以避免非對稱式路由問題

如果您的應用程式有相依性，它們必須新增至您的 Azure 防火牆。 建立應用程式規則來允許 HTTP/HTTPS 流量，以及建立網路規則來允許其他一切流量。 

如果您知道應用程式要求流量來源的位址範圍，可以將它新增到指派給您 ASE 子網路的路由表。 如果位址範圍很大或未指定，則可以使用網路設備，例如應用程式閘道，提供您一個位址以便新增至路由表。 如需設定應用程式閘道與 ILB ASE 的詳細資訊，請參閱[整合 ILB ASE 與應用程式閘道](https://docs.microsoft.com/azure/app-service/environment/integrate-with-application-gateway)


## <a name="dependencies"></a>相依性

Azure App Service 有一些外部相依性。 它們可以按照分類細分成數個主要領域：

- 如果您想要鎖住輸出網路流量的服務端點，應該設定支援服務端點的服務。
- IP 位址端點不會以網域名稱來定址。 這對於預期所有 HTTPS 流量都使用網域名稱的防火牆裝置而言，可能是個問題。 IP 位址端點應該新增至 ASE 子網路上設定的路由表。
- FQDN HTTP/HTTPS 端點可以放在您的防火牆裝置。
- 萬用字元 HTTP/HTTPS 端點是根據一些限定條件，可能隨著 ASE 而變的相依性。 
- 只有在您要將 Linux 應用程式部署到 ASE 時，才需要考量 Linux 相依性。 如果您沒有要將 Linux 應用程式部署到 ASE，則這些位址不需要新增至您的防火牆。 


#### <a name="service-endpoint-capable-dependencies"></a>服務端點功能相依性 

| 端點 |
|----------|
| Azure SQL |
| Azure 儲存體 |
| Azure 金鑰保存庫 |


#### <a name="ip-address-dependencies"></a>IP 位址相依性 

| 端點 |
|----------|
| 40.77.24.27:443 |
| 13.82.184.151:443 |
| 13.68.109.212:443 |
| 13.90.249.229:443 |
| 13.91.102.27:443 |
| 104.45.230.69:443 |
| 168.62.226.198:12000 |


#### <a name="fqdn-httphttps-dependencies"></a>FQDN HTTP/HTTPS 相依性 

| 端點 |
|----------|
|graph.windows.net:443 |
|login.live.com:443 |
|login.windows.com:443 |
|login.windows.net:443 |
|login.microsoftonline.com:443 |
|client.wns.windows.com:443 |
|definitionupdates.microsoft.com:443 |
|go.microsoft.com:80 |
|go.microsoft.com:443 |
|www.microsoft.com:80 |
|www.microsoft.com:443 |
|wdcpalt.microsoft.com:443 |
|wdcp.microsoft.com:443 |
|ocsp.msocsp.com:443 |
|mscrl.microsoft.com:443 |
|mscrl.microsoft.com:80 |
|crl.microsoft.com:443 |
|crl.microsoft.com:80 |
|www.thawte.com:443 |
|crl3.digicert.com:80 |
|ocsp.digicert.com:80 |
|csc3-2009-2.crl.verisign.com:80 |
|crl.verisign.com:80 |
|ocsp.verisign.com:80 |
|azperfcounters1.blob.core.windows.net:443 |
|azurewatsonanalysis-prod.core.windows.net:443 |
|global.metrics.nsatc.net:80   |
|az-prod.metrics.nsatc.net:443 |
|antares.metrics.nsatc.net:443 |
|azglobal-black.azglobal.metrics.nsatc.net:443 |
|azglobal-red.azglobal.metrics.nsatc.net:443 |
|antares-black.antares.metrics.nsatc.net:443 |
|antares-red.antares.metrics.nsatc.net:443 |
|maupdateaccount.blob.core.windows.net:443 |
|clientconfig.passport.net:443 |
|packages.microsoft.com:443 |
|schemas.microsoft.com:80 |
|schemas.microsoft.com:443 |
|management.core.windows.net:443 |
|management.core.windows.net:80 |
|www.msftconnecttest.com:80 |
|shavamanifestcdnprod1.azureedge.net:443 |
|validation-v2.sls.microsoft.com:443 |
|flighting.cp.wd.microsoft.com:443 |
|dmd.metaservices.microsoft.com:80 |
|admin.core.windows.net:443 |
|azureprofileruploads.blob.core.windows.net:443 |
|azureprofileruploads2.blob.core.windows.net:443 |
|azureprofileruploads3.blob.core.windows.net:443 |
|azureprofileruploads4.blob.core.windows.net:443 |
|azureprofileruploads5.blob.core.windows.net:443 |

#### <a name="wildcard-httphttps-dependencies"></a>萬用字元 HTTP/HTTPS 相依性 

| 端點 |
|----------|
|gr-Prod-\*.cloudapp.net:443 |
| \*.management.azure.com:443 |
| \*.update.microsoft.com:443 |
| \*.windowsupdate.microsoft.com:443 |
|grmdsprod\*mini\*.servicebus.windows.net:443 |
|grmdsprod\*lini\*.servicebus.windows.net:443 |
|grsecprod\*mini\*.servicebus.windows.net:443 |
|grsecprod\*lini\*.servicebus.windows.net:443 |
|graudprod\*mini\*.servicebus.windows.net:443 |
|graudprod\*lini\*.servicebus.windows.net:443 |

#### <a name="linux-dependencies"></a>Linux 相依項目 

| 端點 |
|----------|
|wawsinfraprodbay063.blob.core.windows.net:443 |
|registry-1.docker.io:443 |
|auth.docker.io:443 |
|production.cloudflare.docker.com:443 |
|download.docker.com:443 |
|us.archive.ubuntu.com:80 |
|download.mono-project.com:80 |
|packages.treasuredata.com:80|
|security.ubuntu.com:80 |

