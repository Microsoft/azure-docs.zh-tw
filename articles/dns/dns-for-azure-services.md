---
title: 使用 Azure DNS 搭配其他 Azure 服務 | Microsoft Docs
description: 了解如何使用 Azure DNS 來解析其他 Azure 服務的名稱
services: dns
documentationcenter: na
author: vhorne
manager: jeconnoc
editor: ''
tags: azure dns
ms.assetid: e9b5eb94-7984-4640-9930-564bb9e82b78
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.custom: H1Hack27Feb2017
ms.workload: infrastructure-services
ms.date: 09/21/2016
ms.author: victorh
ms.openlocfilehash: 5d9886e16aa1921963f3d91d0fbd4da9287f7e54
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46989136"
---
# <a name="how-azure-dns-works-with-other-azure-services"></a>Azure DNS 如何與其他 Azure 服務搭配運作

Azure DNS 是一種託管的 DNS 管理與名稱解析服務。 這可讓您為其他應用程式和已在 Azure 中部署的服務建立公用 DNS 名稱。 為您的自訂網域中的 Azure 服務建立名稱，就和為您的服務新增正確類型的記錄一樣簡單。

* 針對動態配置的 IP 位址，您可以建立 DNS CNAME 記錄，對應至 Azure 為您的服務建立的 DNS 名稱。 DNS 標準禁止您在區域頂點使用 CNAME 記錄，但您可以改為使用別名記錄。 如需詳細資訊，請參閱[教學課程：設定別名記錄來參考 Azure 公用 IP 位址](tutorial-alias-pip.md)。
* 若使用靜態配置的 IP 位址，您可以用任何名稱建立 DNS A 記錄，包括在區域頂點的「裸網域」名稱。

下表概述可用於各種 Azure 服務的支援記錄類型。 如您在表中所見，Azure DNS 只支援網際網路面向網路資源的 DNS 記錄。 Azure DNS 無法用於內部私人位址的名稱解析。

| Azure 服務 | 網路介面 | 說明 |
| --- | --- | --- |
| 應用程式閘道 |[前端公用 IP](dns-custom-domain.md#public-ip-address) |您可以建立 DNS A 或 CNAME 記錄。 |
| 負載平衡器 |[前端公用 IP](dns-custom-domain.md#public-ip-address)  |您可以建立 DNS A 或 CNAME 記錄。 負載平衡器可以有動態指派的 IPv6 公用 IP 位址。 因此，您必須建立用於 IPv6 位址的 CNAME 記錄。 |
| 流量管理員 |公開名稱 |您可以建立對應至指派給您流量管理員設定檔之 trafficmanager.net 名稱的別名記錄。 如需詳細資訊，請參閱[教學課程：使用流量管理員來設定別名記錄以支援頂點網域名稱](tutorial-alias-tm.md)。 |
| 服務雲端 |[公用 IP](dns-custom-domain.md#public-ip-address) |若使用靜態配置的 IP 位址，您可以建立 DNS A 記錄。 若使用動態配置的 IP 位址，您必須建立對應至 *cloudapp.net* 名稱的 CNAME 記錄。|
| App Service 方案 | [外部 IP](dns-custom-domain.md#app-service-web-apps) |若使用外部 IP 位址，您可以建立 DNS A 記錄。 若不是，則必須建立對應至 azurewebsites.net 名稱的 CNAME 記錄。 如需詳細資訊，請參閱 [將自訂網域名稱對應至 Azure 應用程式](../app-service/app-service-web-tutorial-custom-domain.md) |
| Resource Manager VM |[公用 IP](dns-custom-domain.md#public-ip-address) |Resource Manager VM 可以有公用 IP 位址。 帶有公用 IP 位址的 VM 也可能擺在負載平衡器後。 您可以為公用位址建立 DNS A、CNAME 或別名記錄。 這個自訂名稱可以用來略過負載平衡器上的 VIP。 |
| 傳統 VM |[公用 IP](dns-custom-domain.md#public-ip-address) |使用 PowerShell 或 CLI 建立的傳統 VM 可設定為使用動態或靜態 (保留) 的虛擬位址。 您可以分別建立 DNS CNAME 或 A 記錄。 |
