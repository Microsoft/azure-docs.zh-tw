---
title: 整合 Azure DNS 與您的 Azure 資源 | Microsoft Docs
description: 了解如何使用 Azure DNS 為您的 Azure 資源提供 DNS。
services: dns
documentationcenter: na
author: vhorne
manager: jeconnoc
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 1/19/2018
ms.author: victorh
ms.openlocfilehash: 9c0ccb7c5ccb3eced32ea5bcd9f1961c5ccd2110
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39173838"
---
# <a name="use-azure-dns-to-provide-custom-domain-settings-for-an-azure-service"></a>使用 Azure DNS 為 Azure 服務提供自訂網域設定

Azure DNS 提供自訂網域的 DNS，可用於任何支援自訂網域或具有完整網域名稱 (FQDN) 的 Azure 資源。 例如，您有一個 Azure Web 應用程式，而且想要使用 contoso.com 或 www.contoso.com 作為 FQDN 讓使用者存取它。 本文章會引導您使用 Azure DNS 設定您的 Azure 服務，以便使用自訂網域。

## <a name="prerequisites"></a>必要條件

為了在您的自訂網域 Azure DNS，您必須先將您的網域委派給 Azure DNS。 如需如何設定名稱伺服器以便進行委派的指示，請參閱[將網域委派給 Azure DNS](./dns-delegate-domain-azure-dns.md)。 一旦將您的網域委派給您的 Azure DNS 區域，您就能夠設定所需的 DNS 記錄。

您可以為 [Azure Function 應用程式](#azure-function-app)、[公用 IP 位址](#public-ip-address)、[App Service (Web Apps)](#app-service-web-apps)、[Blob 儲存體](#blob-storage)、[Azure CDN](#azure-cdn) 設定虛名或自訂網域。

## <a name="azure-function-app"></a>Azure 函數應用程式

若要設定 Azure 函式應用程式的自訂網域，建立 CNAME 記錄以及函數應用程式本身的組態需。
 
瀏覽至 [其他]  >  [函數應用程式]，選取您的函數應用程式。 按一下 [平台功能]，然後按一下 [網路] 下方的[自訂網域]。

![函數應用程式刀鋒視窗](./media/dns-custom-domain/functionapp.png)

記下眼前 [自訂網域] 刀鋒視窗中的 URL，這個位址要作為建立之 DNS 記錄的別名。

![自訂網域刀鋒視窗](./media/dns-custom-domain/functionshostname.png)

瀏覽至您的 DNS 區域，按一下 [+ 記錄集]。 填寫 [新增記錄集] 刀鋒視窗中的資訊，然後按一下 [確定] 加以建立。

|屬性  |值  |說明  |
|---------|---------|---------|
|Name     | myFunctionApp        | 這個值以及網域名稱標籤是自訂網域名稱的 FQDN。        |
|類型     | CNAME        | 使用 CNAME 記錄會使用別名。        |
|TTL     | 1        | 1 為使用 1 小時        |
|TTL 單位     | 小時        | 使用小時作為時間量值單位         |
|Alias     | adatumfunction.azurewebsites.net        | 您正在為其建立別名的 DNS 名稱，在此範例中是 adatumfunction.azurewebsites.net (預設提供給函數應用程式的 DNS 名稱)。        |

瀏覽回到您的函數應用程式，按一下 [平台功能]，再按一下 [網路] 下的 [自訂網域]，然後按一下[主機名稱] 下的 [+ 新增主機名稱]。

在 [新增主機名稱] 刀鋒視窗的 [主機名稱] 文字欄位中輸入 CNAME 記錄，然後按一下 [驗證]。 如果能夠找到記錄，就會出現 [新增主機名稱] 按鈕。 按一下 [新增主機名稱] 以新增別名。

![函數應用程式的新增主機名稱刀鋒視窗](./media/dns-custom-domain/functionaddhostname.png)

## <a name="public-ip-address"></a>公用 IP 位址

如果服務使用公用 IP 位址資源 (例如，應用程式閘道、負載平衡器、雲端服務、Resource Manager 的 VM、傳統 VM)，若要設定此種服務的自訂網域，則要使用 CNAME 記錄。

瀏覽至 [網路]  >  [公用 IP 位址]，選取公用 IP 資源，然後按一下 [設定]。 記下顯示的 IP 位址。

![公用 IP 刀鋒視窗](./media/dns-custom-domain/publicip.png)

瀏覽至您的 DNS 區域，按一下 [+ 記錄集]。 填寫 [新增記錄集] 刀鋒視窗中的資訊，然後按一下 [確定] 加以建立。


|屬性  |值  |說明  |
|---------|---------|---------|
|Name     | mywebserver        | 這個值以及網域名稱標籤是自訂網域名稱的 FQDN。        |
|類型     | 具有使用         | 使用 A 記錄，因為該資源是 IP 位址。        |
|TTL     | 1        | 1 為使用 1 小時        |
|TTL 單位     | 小時        | 使用小時作為時間量值單位         |
|IP 位址     | <your ip address>       | 公用 IP 位址。|

![建立 A 記錄](./media/dns-custom-domain/arecord.png)

一旦建立 A 記錄，執行 `nslookup` 驗證記錄解析。

![公用 IP DNS 查閱](./media/dns-custom-domain/publicipnslookup.png)

## <a name="app-service-web-apps"></a>App Service (Web Apps)

以下步驟引導您設定 App Service Web 應用程式的自訂網域。

瀏覽至 [Web 與行動]  >  [App Service]，選取您要設定自訂網域名稱的資源，然後按一下 [自訂網域]。

記下眼前 [自訂網域] 刀鋒視窗中的 URL，這個位址要作為建立之 DNS 記錄的別名。

![自訂網域刀鋒視窗](./media/dns-custom-domain/url.png)

瀏覽至您的 DNS 區域，按一下 [+ 記錄集]。 填寫 [新增記錄集] 刀鋒視窗中的資訊，然後按一下 [確定] 加以建立。


|屬性  |值  |說明  |
|---------|---------|---------|
|Name     | mywebserver        | 這個值以及網域名稱標籤是自訂網域名稱的 FQDN。        |
|類型     | CNAME        | 使用 CNAME 記錄會使用別名。 如果資源使用 IP 位址，就會使用 A 記錄。        |
|TTL     | 1        | 1 為使用 1 小時        |
|TTL 單位     | 小時        | 使用小時作為時間量值單位         |
|Alias     | webserver.azurewebsites.net        | 您正在為其建立別名的 DNS 名稱，在此範例中是 webserver.azurewebsites.net (預設提供給 Web 應用程式的 DNS 名稱)。        |


![建立 CNAME 記錄](./media/dns-custom-domain/createcnamerecord.png)

瀏覽回到設定自訂網域名稱的 App Service。 按一下 [自訂網域] ，然後按一下 [主機名稱]。 若要新增您所建立的 CNAME 記錄，按一下 [+ 新增主機名稱]。

![圖 1](./media/dns-custom-domain/figure1.png)

處理程序完成之後，執行**nslookup** 驗證名稱解析的運作。

![圖 1](./media/dns-custom-domain/finalnslookup.png)

若要深入了解自訂網域對應至 App Service，請造訪[將現有的自訂 DNS 名稱對應至 Azure Web Apps](../app-service/app-service-web-tutorial-custom-domain.md?toc=%dns%2ftoc.json)。

如果您需要購買自訂網域，請造訪[購買 Azure Web Apps 的自訂網域名稱](../app-service/custom-dns-web-site-buydomains-web-app.md)，以深入了解 App Service 網域。

## <a name="blob-storage"></a>Blob 儲存體

下列步驟引導您使用 asverify 方法設定 blob 儲存體帳戶的 CNAME 記錄。 這個方法可確保沒有任何停機時間。

瀏覽至 [儲存體]  >  [儲存體帳戶]，選取您的儲存體帳戶，然後選取 [自訂網域]。 記下步驟 2 的 FQDN，這個值用來建立第一筆 CNAME 記錄

![Blob 儲存體自訂網域](./media/dns-custom-domain/blobcustomdomain.png)

瀏覽至您的 DNS 區域，按一下 [+ 記錄集]。 填寫 [新增記錄集] 刀鋒視窗中的資訊，然後按一下 [確定] 加以建立。


|屬性  |值  |說明  |
|---------|---------|---------|
|Name     | asverify.mystorageaccount        | 這個值以及網域名稱標籤是自訂網域名稱的 FQDN。        |
|類型     | CNAME        | 使用 CNAME 記錄會使用別名。        |
|TTL     | 1        | 1 為使用 1 小時        |
|TTL 單位     | 小時        | 使用小時作為時間量值單位         |
|Alias     | asverify.adatumfunctiona9ed.blob.core.windows.net        | 您正在為其建立別名的 DNS 名稱，在此範例中是 asverify.adatumfunctiona9ed.blob.core.windows.net (預設提供給儲存體帳戶的 DNS 名稱)。        |

按一下 [儲存體]  >  [儲存體帳戶] 以回到您的儲存體帳戶，選取您的儲存體帳戶，然後選取 [自訂網域]。 在文字方塊中輸入您建立的別名但不含 asverify 前置詞，核取 [使用間接 CNAME 驗證]，然後按一下 [儲存]。 一旦完成這個步驟，返回您的 DNS 區域，建立不含 asverify 前置詞的 CNAME 記錄。  此時，您就可以放心刪除具有 cdnverify 前置詞的 CNAME 記錄。

![Blob 儲存體自訂網域](./media/dns-custom-domain/indirectvalidate.png)

執行 `nslookup` 驗證 DNS 解析

若要深入了解自訂網域對應至 Blob 儲存體，請造訪[為您的 Blob 儲存體端點設定自訂網域名稱](../storage/blobs/storage-custom-domain-name.md?toc=%dns%2ftoc.json)。

## <a name="azure-cdn"></a>Azure CDN

下列步驟引導您使用 asverify 方法設定 CDN 端點的 CNAME 記錄。 這個方法可確保沒有任何停機時間。

瀏覽至 [網路]  >  [CDN 設定檔]，選取您的 CDN 設定檔，然後按一下 [一般] 下的 [端點]。

選取您使用的端點，按一下 [+ 自訂網域]。 請注意 [端點主機名稱]，因為這個值是 CNAME 記錄指向的記錄。

![CDN 自訂網域](./media/dns-custom-domain/endpointcustomdomain.png)

瀏覽至您的 DNS 區域，按一下 [+ 記錄集]。 填寫 [新增記錄集] 刀鋒視窗中的資訊，然後按一下 [確定] 加以建立。

|屬性  |值  |說明  |
|---------|---------|---------|
|Name     | cdnverify.mycdnendpoint        | 這個值以及網域名稱標籤是自訂網域名稱的 FQDN。        |
|類型     | CNAME        | 使用 CNAME 記錄會使用別名。        |
|TTL     | 1        | 1 為使用 1 小時        |
|TTL 單位     | 小時        | 使用小時作為時間量值單位         |
|Alias     | cdnverify.adatumcdnendpoint.azureedge.net        | 您正在為其建立別名的 DNS 名稱，在此範例中是 cdnverify.adatumcdnendpoint.azureedge.net (預設提供給儲存體帳戶的 DNS 名稱)。        |

按一下 [網路]  >  [CDN 設定檔] 以回到您的 CDN端點，選取您的 CDN 設定檔。 按一下 [+ 自訂網域] 並輸入您的 CNAME 記錄別名但不含 cdnverify 前置詞，然後按一下 [新增]。

一旦完成這個步驟，返回您的 DNS 區域，建立不含 asverify 前置詞的 CNAME 記錄。  此時，您就可以放心刪除具有 cdnverify 前置詞的 CNAME 記錄。 針對 CDN 以及如何設定自訂網域，而不經過中間註冊步驟，如需詳細資訊請造訪[將 Azure CDN 內容對應至自訂網域](../cdn/cdn-map-content-to-custom-domain.md?toc=%dns%2ftoc.json)。

## <a name="next-steps"></a>後續步驟

了解如何[為 Azure 裝載的服務設定反向 DNS](dns-reverse-dns-for-azure-services.md)。