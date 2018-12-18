---
title: 在使用流量管理員進行負載平衡的 Azure App Service 中設定 Web 應用程式的自訂網域名稱。
description: 在包含負載平衡的流量管理員的 Azure App Service 中使用 Web 應用程式的自訂網域名稱。
services: app-service\web
documentationcenter: ''
author: cephalin
manager: cfowler
editor: ''
ms.assetid: 0f96c0e7-0901-489b-a95a-e3b66ca0a1c2
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/17/2016
ms.author: cephalin
ms.openlocfilehash: a83b5dada8afff455375ae1606ecaac4c9ebe8e7
ms.sourcegitcommit: 4ea0cea46d8b607acd7d128e1fd4a23454aa43ee
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/15/2018
ms.locfileid: "42144221"
---
# <a name="configuring-a-custom-domain-name-for-a-web-app-in-azure-app-service-using-traffic-manager"></a>在使用流量管理員的 Azure App Service 中設定 Web 應用程式的自訂網域名稱
[!INCLUDE [web-selector](../../includes/websites-custom-domain-selector.md)]

[!INCLUDE [intro](../../includes/custom-dns-web-site-intro-traffic-manager.md)]

本文提供將自訂網域名稱使用於與[流量管理員](../traffic-manager/traffic-manager-overview.md)整合來進行負載平衡之 [App Service](app-service-web-overview.md) 應用程式的一般指示。

[!INCLUDE [tmwebsitefooter](../../includes/custom-dns-web-site-traffic-manager-notes.md)]

[!INCLUDE [introfooter](../../includes/custom-dns-web-site-intro-notes.md)]

<a name="understanding-records"></a>

## <a name="understanding-dns-records"></a>了解 DNS 記錄
[!INCLUDE [understandingdns](../../includes/custom-dns-web-site-understanding-dns-traffic-manager.md)]

<a name="bkmk_configsharedmode"></a>

## <a name="configure-your-web-apps-for-standard-mode"></a>將 Web 應用程式設定為標準模式
[!INCLUDE [modes](../../includes/custom-dns-web-site-modes-traffic-manager.md)]

<a name="bkmk_configurecname"></a>

## <a name="add-a-dns-record-for-your-custom-domain"></a>新增自訂網域的 DNS 記錄
> [!NOTE]
> 如果您已透過 Azure App Service Web Apps 購買網域，請略過下列步驟，並參閱 [購買 Web Apps 網域](custom-dns-web-site-buydomains-web-app.md) 文章的最後一個步驟。
> 
> 

若要將您的自訂網域與 Azure App Service 中的 Web 應用程式產生關聯，您必須在 DNS 資料表中為您的自訂網域新增一個項目。 使用網域提供者提供的管理工具來執行此作業。

[!INCLUDE [Access DNS records with domain provider](../../includes/app-service-web-access-dns-records-no-h.md)]

由於每個網域提供者的特殊要求可能有所不同，您會「從」自訂網域名稱 (例如 **contoso.com**) 對應「至」與您的 Web 應用程式整合的流量管理員網域名稱 (**contoso.trafficmanager.net**)。
   
> [!NOTE]
> 如果記錄已在使用中，而您需要事先將您的應用程式繫結到該記錄，您可以建立其他的 CNAME 記錄。 例如，若要事先將 **www.contoso.com** 繫結至 Web 應用程式，請建立從 **awverify.www** 至 **contoso.trafficmanager.net** 的 CNAME 記錄。 然後，您可以將 "www.contoso.com" 新增至 Web 應用程式，而不必變更 "www" CNAME 記錄。 如需詳細資訊，請參閱[在自訂網域中建立 Web 應用程式的 DNS 記錄][CREATEDNS]。
> 
> 

在網域提供者處完成 DNS 記錄的新增或修改後，請儲存變更。

<a name="enabledomain"></a>

## <a name="enable-traffic-manager"></a>啟用流量管理員
[!INCLUDE [modes](../../includes/custom-dns-web-site-enable-on-traffic-manager.md)]

## <a name="next-steps"></a>後續步驟
如需詳細資訊，請參閱 [Node.js 開發人員中心](http://azure.microsoft.com/develop/nodejs/)。

[!INCLUDE [app-service-web-try-app-service](../../includes/app-service-web-try-app-service.md)]

<!-- URL List -->

[CREATEDNS]: ../dns/dns-web-sites-custom-domain.md
