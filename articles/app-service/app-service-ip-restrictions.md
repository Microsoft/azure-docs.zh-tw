---
title: Azure App Service IP 限制 | Microsoft Docs
description: 如何搭配使用 IP 限制和 Azure App Service
author: ccompy
manager: stefsch
editor: ''
services: app-service\web
documentationcenter: ''
ms.assetid: 3be1f4bd-8a81-4565-8a56-528c037b24bd
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: article
ms.date: 7/30/2018
ms.author: ccompy
ms.openlocfilehash: 9ffeb01a46b62202292f6e1ea91f47cd87954387
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47219123"
---
# <a name="azure-app-service-static-ip-restrictions"></a>Azure App Service 靜態 IP 限制 #

IP 限制可讓您針對允許存取應用程式的 IP 位址，定義有先後順序的允許/拒絕清單。 允許清單可以包含 IPv4 和 IPv6 位址。 有一個或多個項目時，清單結尾會有隱含的「拒絕全部」語句。 

IP 限制功能適用於所有 App Service 主控的工作負載，包括：Web 應用程式、API 應用程式、Linux 應用程式、Linux 容器應用程式及 Functions。 

如果有對您應用程式發出的要求，「來源」IP 位址就會根據 IP 限制清單受到評估。 如果根據清單中的規則，不允許該位址進行存取，則服務會回覆 [HTTP 403](https://en.wikipedia.org/wiki/HTTP_403) 狀態碼。

IP 限制功能會在 App Service 前端角色中實作，此角色是背景工作角色主機 (程式碼在此執行) 的上游。 因此，IP 限制實際上是網路 ACL。  

![IP 限制流程](media/app-service-ip-restrictions/ip-restrictions-flow.png)

在入口網站中，IP 限制功能曾經一度是 IIS 中 ipSecurity 功能之上的一個層級。 目前的 IP 限制功能已有所不同。 您仍然可以在應用程式的 web.config 中設定 ipSecurity，但以前端為基礎的 IP 限制規則，會在任何流量到達 IIS 之前加以套用。

## <a name="adding-and-editing-ip-restriction-rules-in-the-portal"></a>在入口網站中新增和編輯 IP 限制規則 ##

若要將 IP 限制規則新增至應用程式，請使用功能表來開啟 [網路] > [IP 限制]，然後按一下 [設定 IP 限制]

![App Service 網路功能選項](media/app-service-ip-restrictions/ip-restrictions.png)  

從 IP 限制 UI 中，您可以檢閱針對應用程式定義的 IP 限制規則清單。

![IP 限制清單](media/app-service-ip-restrictions/ip-restrictions-browse.png)

如果您的規則設定與此圖片中的一樣，則應用程式只會接受來自 131.107.159.0/24 的流量，並且拒絕來自任何其他 IP 位址的流量。

您可以按一下 [[+] 新增] 新增 IP 限制規則。 當您新增規則之後，該規則會立即生效。 規則會依據優先順序強制執行，從最小數字開始往上增加。 當您新增規則時 (即使只新增一個規則)，隱含的「拒絕所有」語句就會發生作用。 

![新增 IP 限制規則](media/app-service-ip-restrictions/ip-restrictions-add.png)

針對 IPv4 和 IPv6 位址，CIDR 標記法中必須指定 IP 位址標記法。 若要指定正確的位址，您可以使用類似 1.2.3.4/32 的位址，其中前四個八位元代表您的 IP 位址，而 /32 是遮罩。 適用於所有位址的 IPv4 CIDR 標記法是 0.0.0.0/0。 若要深入了解 CIDR 標記法，您可以參閱[無類別網域間路由](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing)。  

您可以按一下任何資料列，以編輯現有的 IP 限制規則。 編輯內容與優先順序的變更都會立即生效。

![編輯 IP 限制規則](media/app-service-ip-restrictions/ip-restrictions-edit.png)

若要刪除規則，請按一下您規則上的 [...]，然後按一下 [移除]。 

![刪除 IP 限制規則](media/app-service-ip-restrictions/ip-restrictions-delete.png)

## <a name="programmatic-manipulation-of-ip-restriction-rules"></a>以程式設計方式操作 IP 限制規則 ##

目前沒有任何 CLI 或 PowerShell 可用於新的 IP 限制功能，但您可以在資源管理員中使用應用程式設定上的 PUT 作業，來手動設定這些值。 例如，您可以使用 resources.azure.com，並編輯 ipSecurityRestrictions 區塊來新增必要的 JSON。 

您可以在資源管理員中的以下位置找到此資訊：

management.azure.com/subscriptions/**subscription ID**/resourceGroups/**resource groups**/providers/Microsoft.Web/sites/**web app name**/config/web?api-version=2018-02-01

先前的 JSON 語法範例為：

    "ipSecurityRestrictions": [
      {
        "ipAddress": "131.107.159.0/24",
        "action": "Allow",
        "tag": "Default",
        "priority": 100,
        "name": "allowed access"
      }
    ],
