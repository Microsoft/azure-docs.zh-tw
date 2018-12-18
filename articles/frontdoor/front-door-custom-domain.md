---
title: 教學課程 - 將自訂網域新增至 Azure Front Door 組態 | Microsoft Docs
description: 在本教學課程中，您將了解如何將自訂網域上架到 Azure Front Door。
services: frontdoor
documentationcenter: ''
author: sharad4u
editor: ''
ms.service: frontdoor
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 09/10/2018
ms.author: sharadag
ms.openlocfilehash: 8106c68397dea8d52c6d2daa2d09dfbc72c2a4c8
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46995052"
---
# <a name="tutorial-add-a-custom-domain-to-your-front-door"></a>教學課程：將自訂網域新增至您的 Front Door
本教學課程說明如何將自訂網域新增至您的 Front Door。 使用 Azure Front Door 進行應用程式傳遞時，如果您想要在使用者要求中顯示您自己的網域名稱，則需要自訂網域。 有可見的網域名稱對您的客戶而言較為方便，並且也有助於宣傳商標。

當您建立 Front Door 後，預設端點主機 (`azurefd.net` 的子網域) 依預設會包含在從您的後端傳遞 Front Door 內容的 URL 中 (例如 https:\//contoso.azurefd.net/activeusers.htm)。 為了方便起見，Azure Front Door 會提供在自訂網域與預設主機之間建立關聯的選項。 使用此選項時，您會使用 URL 中的自訂網域來傳遞內容，而不是使用 Front Door 擁有的網域名稱 (例如 https:\//www.contoso.com/photo.png)。 

在本教學課程中，您了解如何：
> [!div class="checklist"]
> - 建立 CNAME DNS 記錄。
> - 將自訂網域與您的 Front Door 產生關聯。
> - 驗證自訂網域。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>必要條件

您必須先建立 Front Door，才能完成本教學課程中的步驟。 如需詳細資訊，請參閱[快速入門：建立 Front Door](quickstart-create-front-door.md)。

如果您還沒有自訂網域，必須先向網域提供者購買。 如需範例，請參閱[購買自訂網域名稱](https://docs.microsoft.com/azure/app-service/custom-dns-web-site-buydomains-web-app)。

如果您使用 Azure 來裝載您的 [DNS 網域](https://docs.microsoft.com/azure/dns/dns-overview)，必須將網域提供者的網域名稱系統 (DNS) 委派給 Azure DNS。 如需詳細資訊，請參閱[將網域委派給 Azure DNS](https://docs.microsoft.com/azure/dns/dns-delegate-domain-azure-dns)。 反之，如果您使用網域提供者來處理 DNS 網域，請繼續執行[建立 CNAME DNS 記錄](#create-a-cname-dns-record)。


## <a name="create-a-cname-dns-record"></a>建立 CNAME DNS 記錄

您必須先透過網域提供者建立正式名稱 (CNAME) 記錄指向您 Front Door 的預設前端主機 (例如 contose.azurefd.net)，才可搭配使用自訂網域與您的 Front Door。 CNAME 記錄是一種將來源網域名稱對應至目的地網域名稱的 DNS 記錄。 在 Azure Front Door Service 中，來源網域名稱是您的自訂網域名稱，而目的地網域名稱是您的 Front Door 預設主機名稱。 在 Front Door 驗證您所建立的 CNAME 記錄後，定址到來源自訂網域 (例如 www.contoso.com) 的流量即會路由至指定的目的地 Front Door 預設前端主機 (例如 contoso.azurefd.net)。 

自訂網域及其子網域一次只能與單一 Front Door 相關聯。 不過，您可以使用多個 CNAME 記錄，將來自相同自訂網域的不同子網域用於不同的 Front Door。 您也可以將具有不同子網域的自訂網域對應至相同的 Front Door。


## <a name="map-the-temporary-afdverify-sub-domain"></a>對應暫時 afdverify 子網域

在對應生產環境中的現有網域時，會有特殊事項需要考量。 當您在 Azure 入口網站中註冊自訂網域時，該網域可能會有短暫的期間停止運作。 為了避免 Web 流量中斷，請先將自訂網域對應至具有 Azure afdverify 子網域的 Front Door 預設前端主機，以建立暫時 CNAME 對應。 透過此方法，使用者在 DNS 對應期間將可持續存取您的網域而不被中斷。

若非如此，如果您是第一次使用自訂網域，而且其間沒有生產流量正在執行，您可以直接將自訂網域對應至 Front Door。 請繼續執行[對應永久自訂網域](#map-the-permanent-custom-domain)。

若要建立 afdverify 子網域的 CNAME 記錄：

1. 登入自訂網域的網域提供者網站。

2. 找出管理 DNS 記錄的頁面 (可查閱提供者的文件或，尋找網站上標示為**網域名稱**、**DNS** 或**名稱伺服器管理**的區域)。 

3. 建立自訂網域的 CNAME 記錄項目，並完成下表說明的欄位 (欄位名稱可能有所不同)：

    | 來源                    | 類型  | 目的地                     |
    |---------------------------|-------|---------------------------------|
    | afdverify.www.contoso.com | CNAME | afdverify.contoso.azurefd.net |

    - 來源：以下列格式輸入您的自訂網域名稱 (包括 afdverify 子網域)：afdverify._&lt;自訂網域名稱&gt;_。 例如 afdverify.www.contoso.com。

    - 類型：輸入 *CNAME*。

    - 目的地：以下列格式輸入您的預設 Front Door 前端主機 (包括 afdverify 子網域)：afdverify._&lt;端點名稱&gt;_.azurefd.net。 例如 afdverify.contoso.azurefd.net。

4. 儲存您的變更。

例如，GoDaddy 網域註冊機構的程序如下所示：

1. 登入並選取您要使用的自訂網域。

2. 在 [網域] 區段中選取 [管理全部]，然後選取 [DNS] | [管理區域]。

3. 針對 [網域名稱]，輸入您的自訂網域，然後選取 [搜尋]。

4. 在 [DNS 管理] 頁面中選取 [新增]，然後在 [類型] 清單中選取 [CNAME]。

5. 完成 CNAME 項目的下列欄位：

    - 類型：保留已選取的 [CNAME]。

    - 主機：輸入要使用的自訂網域的子網域，包括 afdverify 子網域名稱。 例如 afdverify.www。

    - 指向：輸入預設 Front Door 前端主機的主機名稱，包括 afdverify 子網域名稱。 例如 afdverify.contoso.azurefd.net。 

    - TTL：保留已選取的 [1 小時]。

6. 選取 [ **儲存**]。
 
    CNAME 項目會新增至 DNS 記錄資料表。


## <a name="associate-the-custom-domain-with-your-front-door"></a>將自訂網域與您的 Front Door 產生關聯

在註冊您的自訂網域之後，您可以將其新增至 Front Door。

1. 登入 [Azure 入口網站](https://portal.azure.com/)，然後瀏覽至要對應到自訂網域的前端主機所在的 Front Door。
    
2. 在 [Front Door 設計工具] 頁面上，按一下 '+' 以新增自訂網域。
    
3. 指定 [自訂網域]。 

4. 針對 [前端主機]，系統會從您的 Front Door 衍生要在 CNAME 記錄中作為目的地網域的前端主機，並預先填入：*&lt;default hostname&gt;*.azurefd.net。 無法予以變更。

5. 針對 [自訂主機名稱]，請輸入您的自訂網域 (包括子網域)，以作為 CNAME 記錄的來源網域。 例如 www.contoso.com 或 cdn.contoso.com。 請勿使用 afdverify 子網域名稱。

6. 選取 [新增] 。

   Azure 會確認您所輸入的自訂網域名稱有 CNAME 記錄存在。 如果 CNAME 正確，就會驗證您的自訂網域。

>[!WARNING]
> 您**必須**確定 Front Door 中的每個前端主機 (包括自訂網域) 都具有一個路由規則，其中含有與其相關聯的預設路徑 ('/\*')。 也就是說，在您的所有路由規則中，至少必須有一個適用於您每個前端主機的路由規則，這類主機會以預設路徑 ('/\*') 來定義。 若未這麼做，可能會導致您的使用者流量無法正確路由。

## <a name="verify-the-custom-domain"></a>驗證自訂網域

完成自訂網域的註冊之後，請確認自訂網域會參考您的預設 Front Door 前端主機。
 
在瀏覽器中，使用自訂網域瀏覽至檔案的位址。 例如，如果您的自訂網域是 robotics.contoso.com，則快取檔案的 URL 應會類似於下列 URL：http:\//robotics.contoso.com/my-public-container/my-file.jpg。 確認結果與您直接在 *&lt;Front Door 主機&gt;*.azurefd.net 上存取 Front Door 時相同。


## <a name="map-the-permanent-custom-domain"></a>對應永久自訂網域

如果您已確認 afdverify 子網域已成功對應至您的 Front Door (或者，如果您使用不在生產環境中的新自訂網域)，則可以將自訂網域直接對應至預設 Front Door 前端主機。

若要建立自訂網域的 CNAME 記錄：

1. 登入自訂網域的網域提供者網站。

2. 找到管理 DNS 記錄的頁面 (可查閱提供者的文件或，尋找網站上標示**網域名稱**、**DNS** 或**名稱伺服器管理**的區域)。 

3. 建立自訂網域的 CNAME 記錄項目，並完成下表說明的欄位 (欄位名稱可能有所不同)：

    | 來源          | 類型  | 目的地           |
    |-----------------|-------|-----------------------|
    | www.contoso.com | CNAME | contoso.azurefd.net |

    - 來源：輸入您的自訂網域名稱 (例如 www.contoso.com)。

    - 類型：輸入 *CNAME*。

    - 目的地：輸入您的預設 Front Door 前端主機。 此名稱必須是下列格式：_&lt;主機名稱&gt;_.azurefd.net。 例如 contoso.azurefd.net。

4. 儲存您的變更。

5. 如果您先前曾建立暫時 afdverify 子網域 CNAME 記錄，請將其刪除。 

6. 如果您是第一次在生產環境中使用此自訂網域，請遵循[將自訂網域與您的 Front Door 產生關聯](#associate-the-custom-domain-with-your-front-door)和[驗證自訂網域](#verify-the-custom-domain)的步驟。

例如，GoDaddy 網域註冊機構的程序如下所示：

1. 登入並選取您要使用的自訂網域。

2. 在 [網域] 區段中選取 [管理全部]，然後選取 [DNS] | [管理區域]。

3. 針對 [網域名稱]，輸入您的自訂網域，然後選取 [搜尋]。

4. 在 [DNS 管理] 頁面中選取 [新增]，然後在 [類型] 清單中選取 [CNAME]。

5. 完成 CNAME 項目的欄位：

    - 類型：保留已選取的 [CNAME]。

    - 主機：輸入要使用的自訂網域的子網域。 例如 www 或設定檔。

    - 指向：輸入 Front Door 的預設主機名稱。 例如 contoso.azurefd.net。 

    - TTL：保留已選取的 [1 小時]。

6. 選取 [ **儲存**]。
 
    CNAME 項目會新增至 DNS 記錄資料表。

7. 如果您有 afdverify CNAME 記錄，請選取它旁邊的鉛筆圖示，然後選取資源回收筒圖示。

8. 選取 [刪除]，將 CNAME 記錄刪除。


## <a name="clean-up-resources"></a>清除資源

在先前的步驟中，您已將自訂網域新增至 Front Door。 如果您不想再讓您的 Front Door 與自訂網域相關聯，您可以執行下列步驟以移除自訂網域：
 
1. 在您的 Front Door 設計工具中，選取要移除的自訂網域。

2. 在自訂網域的內容功能表中，按一下 [刪除]。  

   自訂網域會與您的端點解除關聯。


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> - 建立 CNAME DNS 記錄。
> - 將自訂網域與您的 Front Door 產生關聯。
> - 驗證自訂網域。