---
title: 將作用中的 DNS 名稱移轉至 Azure App Service | Microsoft Docs
description: 了解如何在完全不停機的情況下，將已指派給即時網站的自訂 DNS 網域名稱移轉至 Azure App Service。
services: app-service
documentationcenter: ''
author: cephalin
manager: erikre
editor: jimbe
tags: top-support-issue
ms.assetid: 10da5b8a-1823-41a3-a2ff-a0717c2b5c2d
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/28/2017
ms.author: cephalin
ms.openlocfilehash: a5d031622103183fa9aa7a3f3771a055fc16edb2
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39049971"
---
# <a name="migrate-an-active-dns-name-to-azure-app-service"></a>將作用中的 DNS 名稱移轉至 Azure App Service

本文示範如何在完全不停機的情況下，將作用中的 DNS 名稱移轉至 [Azure App Service](../app-service/app-service-web-overview.md)。

當您將即時網站及其 DNS 網域名稱移轉至 App Service 時，該 DNS 名稱已對即時流量提供服務。 您可以先將作用中的 DNS 名稱繫結至 App Service 應用程式，以避免 DNS 解析在移轉期間發生停機。

如果您不擔心 DNS 解析發生停機，請參閱[將現有的自訂 DNS 名稱對應至 Azure Web Apps](app-service-web-tutorial-custom-domain.md)。

## <a name="prerequisites"></a>必要條件

若要完成本操作說明：

- [確定您的 App Service 應用程式不是位於免費層](app-service-web-tutorial-custom-domain.md#checkpricing)。

## <a name="bind-the-domain-name-preemptively"></a>事先繫結網域名稱

當您事先繫結自訂網域時，要完成下列兩項之後才能對 DNS 記錄進行任何變更︰

- 確認網域擁有權
- 為您的應用程式啟用網域名稱

當您最後將自訂 DNS 名稱從舊網站移轉至 App Service 應用程式時，DNS 解析不會發生停機。

[!INCLUDE [Access DNS records with domain provider](../../includes/app-service-web-access-dns-records.md)]

### <a name="create-domain-verification-record"></a>建立網域驗證記錄

若要確認網域擁有權，請新增 TXT 記錄。 TXT 記錄會從 _awverify.&lt;子網域>_ 對應至 _&lt;應用程式名稱>.azurewebsites.net_。 

您需要的 TXT 記錄取決於您要移轉的 DNS 記錄。 如需範例，請參閱下表 (`@` 通常代表根網域)：

| DNS 記錄範例 | TXT 主機 | TXT 值 |
| - | - | - |
| \@ (root) | _awverify_ | _&lt;應用程式名稱>.azurewebsites.net_ |
| www (子網域) | _awverify.www_ | _&lt;應用程式名稱>.azurewebsites.net_ |
| \* (萬用字元) | _awverify.\*_ | _&lt;應用程式名稱>.azurewebsites.net_ |

在您的 DNS 記錄頁面中，記下您要移轉之 DNS 名稱的記錄類型。 App Service 支援 CNAME 與 A 記錄之間的對應。

### <a name="enable-the-domain-for-your-app"></a>為您的應用程式啟用網域

在 [Azure 入口網站](https://portal.azure.com)之應用程式頁面的左側導覽中，選取 [自訂網域]。 

![[自訂網域] 功能表](./media/app-service-web-tutorial-custom-domain/custom-domain-menu.png)

在 [自訂網域] 頁面中，選取 [新增主機名稱] 旁的 **+** 圖示。

![新增主機名稱](./media/app-service-web-tutorial-custom-domain/add-host-name-cname.png)

鍵入您新增 TXT 記錄的完整網域名稱，例如 `www.contoso.com`。 針對萬用字元網域 (例如 \*.contoso.com)，您可以使用符合萬用字元網域的任何 DNS 名稱。 

選取 [驗證]。

[新增主機名稱] 按鈕會啟用。 

確定將 [主機名稱記錄類型] 設定為您要移轉的 DNS 記錄類型。

選取 [新增主機名稱]。

![將 DNS 名稱新增至應用程式](./media/app-service-web-tutorial-custom-domain/validate-domain-name-cname.png)

可能需要一些時間，新的主機名稱才會反映在應用程式的 [自訂網域] 分頁中。 嘗試重新整理瀏覽器以更新資料。

![CNAME 記錄已新增](./media/app-service-web-tutorial-custom-domain/cname-record-added.png)

您的 DNS 名稱現已在您的 Azure 應用程式中啟用。 

## <a name="remap-the-active-dns-name"></a>重新對應作用中的 DNS 名稱

唯一剩下的工作是重新對應作用中的 DNS 記錄，以指向 App Service。 目前，它仍指向舊網站。

<a name="info"></a>

### <a name="copy-the-apps-ip-address-a-record-only"></a>複製應用程式的 IP 位址 (僅限 A 記錄)

如果您重新對應的是 CNAME 記錄，請略過本節。 

若要重新對應 A 記錄，您需要 App Service 應用程式的外部 IP 位址，如 [自訂網域] 頁面中所示。

選取右上角的 **X** 關閉 [新增主機名稱] 頁面。 

在 [自訂網域] 頁面中，複製應用程式的 IP 位址。

![入口網站瀏覽至 Azure 應用程式](./media/app-service-web-tutorial-custom-domain/mapping-information.png)

### <a name="update-the-dns-record"></a>更新 DNS 記錄

回到您網域提供者的 DNS 記錄頁面，選取要重新對應的 DNS 記錄。

在 `contoso.com` 根網域範例中，重新對應 A 或 CNAME 記錄，如下表中的範例所示： 

| FQDN 範例 | 記錄類型 | Host | 值 |
| - | - | - | - |
| contoso.com (根網域) | 具有使用  | `@` | 來自[複製應用程式的 IP 位址](#info)的 IP 位址 |
| www.contoso.com (子網域) | CNAME | `www` | _&lt;應用程式名稱>.azurewebsites.net_ |
| \*.contoso.com (萬用字元) | CNAME | _\*_ | _&lt;應用程式名稱>.azurewebsites.net_ |

儲存您的設定。

DNS 查詢應該會在 DNS 散佈發生後立即開始解析為 App Service 應用程式。

## <a name="next-steps"></a>後續步驟

了解如何將自訂 SSL 憑證繫結至 App Service。

> [!div class="nextstepaction"]
> [將現有的自訂 SSL 憑證繫結至 Azure Web Apps](app-service-web-tutorial-custom-ssl.md)
