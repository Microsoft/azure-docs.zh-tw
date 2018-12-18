---
title: 教學課程 - 將自訂網域新增至 Azure CDN 端點 | Microsoft Docs
description: 在本教學課程中，您會將 Azure CDN 端點對應至自訂網域。
services: cdn
documentationcenter: ''
author: mdgattuso
manager: danielgi
editor: ''
ms.service: cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 06/11/2018
ms.author: magattus
ms.custom: mvc
ms.openlocfilehash: 67621cb95cbf5072ab7b72770f43c0b68785c78e
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49092095"
---
# <a name="tutorial-add-a-custom-domain-to-your-azure-cdn-endpoint"></a>教學課程：將自訂網域新增至 Azure CDN 端點
本教學課程說明如何將自訂網域新增至 Azure 內容傳遞網路 (CDN) 端點。 使用 CDN 端點來傳遞內容時，如果您想要在 CDN URL 中顯示您自己的網域名稱，則需要自訂網域。 有可見的網域名稱對您的客戶而言較為方便，並且也有助於宣傳商標。 

當您在設定檔中建立 CDN 端點後，端點名稱 (azureedge.net 的子網域) 會包含在依預設傳遞 CDN 內容的 URL 中 (例如 https:\//contoso.azureedge.net/photo.png)。 為了方便起見，Azure CDN 會提供在自訂網域與 CDN 端點之間建立關聯的選項。 使用此選項時，您會在 URL 中使用自訂網域來傳遞內容，而不是使用端點名稱 (例如 https:\//www.contoso.com/photo.png)。 

在本教學課程中，您了解如何：
> [!div class="checklist"]
> - 建立 CNAME DNS 記錄。
> - 將自訂網域與您的 CDN 端點產生關聯。
> - 驗證自訂網域。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>必要條件

您必須先建立 CDN 設定檔和至少一個 CDN 端點，才能完成本教學課程中的步驟。 如需詳細資訊，請參閱[快速入門：建立 Azure CDN 設定檔和端點](cdn-create-new-endpoint.md)。

如果您還沒有自訂網域，必須先向網域提供者購買。 如需範例，請參閱[購買自訂網域名稱](https://docs.microsoft.com/azure/app-service/custom-dns-web-site-buydomains-web-app)。

如果您使用 Azure 來裝載您的 [DNS 網域](https://docs.microsoft.com/azure/dns/dns-overview)，必須將網域提供者的網域名稱系統 (DNS) 委派給 Azure DNS。 如需詳細資訊，請參閱[將網域委派給 Azure DNS](https://docs.microsoft.com/azure/dns/dns-delegate-domain-azure-dns)。 反之，如果您使用網域提供者來處理 DNS 網域，請繼續執行[建立 CNAME DNS 記錄](#create-a-cname-dns-record)。


## <a name="create-a-cname-dns-record"></a>建立 CNAME DNS 記錄

您必須先透過網域提供者指向您的 CDN 端點以建立正式名稱 (CNAME) 記錄，才可在 Azure CDN 端點使用自訂網域。 CNAME 記錄是一種將來源網域名稱對應至目的地網域名稱的 DNS 記錄。 在 Azure CDN 中，來源網域名稱是您的自訂網域名稱，而目的地網域名稱是您的 CDN 端點主機名稱。 在 Azure CDN 驗證您所建立的 CNAME 記錄後，定址到來源自訂網域 (例如 www.contoso.com) 的流量會路由至指定的目的地 CDN 端點主機名稱 (例如 contoso.azureedge.net)。 

自訂網域及其子網域一次只能與單一端點相關聯。 不過，您可以使用多個 CNAME 記錄，將來自相同自訂網域的不同子網域用於不同的 Azure 服務端點。 您也可以將具有不同子網域的自訂網域對應至相同的 CDN 端點。


## <a name="map-the-temporary-cdnverify-subdomain"></a>對應暫時 cdnverify 子網域

在對應生產環境中的現有網域時，會有特殊事項需要考量。 當您在 Azure 入口網站中註冊自訂網域時，該網域可能會有短暫的期間停止運作。 為了避免 Web 流量中斷，請先將自訂網域對應至具有 Azure cdnverify 子網域的 CDN 端點主機名稱，以建立暫時 CNAME 對應。 透過此方法，使用者在 DNS 對應期間將可持續存取您的網域而不被中斷。 

若非如此，如果您是第一次使用自訂網域，而且其間沒有生產流量正在執行，您可以直接將自訂網域對應至 CDN 端點。 請繼續執行[對應永久自訂網域](#map-the-permanent-custom-domain)。

建立 cdnverify 子網域的 CNAME 記錄：

1. 登入自訂網域的網域提供者網站。

2. 找出管理 DNS 記錄的頁面 (可查閱提供者的文件或，尋找網站上標示為**網域名稱**、**DNS** 或**名稱伺服器管理**的區域)。 

3. 建立自訂網域的 CNAME 記錄項目，並完成下表說明的欄位 (欄位名稱可能有所不同)：

    | 來源                    | 類型  | 目的地                     |
    |---------------------------|-------|---------------------------------|
    | cdnverify.www.contoso.com | CNAME | cdnverify.contoso.azureedge.net |

    - 來源：以下列格式輸入您的自訂網域名稱 (包括 cdnverify 子網域)：cdnverify._&lt;自訂網域名稱&gt;。 例如 cdnverify.www.contoso.com。

    - 類型：輸入 *CNAME*。

    - 目的地：以下列格式輸入您的 CDN 端點主機名稱 (包括 cdnverify 子網域)：cdnverify._&lt;端點名稱&gt;_.azureedge.net。 例如 cdnverify.contoso.azureedge.net。

4. 儲存您的變更。

例如，GoDaddy 網域註冊機構的程序如下所示：

1. 登入並選取您要使用的自訂網域。

2. 在 [網域] 區段中選取 [管理全部]，然後選取 [DNS] | [管理區域]。

3. 針對 [網域名稱]，輸入您的自訂網域，然後選取 [搜尋]。

4. 在 [DNS 管理] 頁面中選取 [新增]，然後在 [類型] 清單中選取 [CNAME]。

5. 完成 CNAME 項目的下列欄位：

    ![CNAME 項目](./media/cdn-map-content-to-custom-domain/cdn-cdnverify-cname-entry.png)

    - 類型：保留已選取的 [CNAME]。

    - 主機：輸入要使用的自訂網域的子網域，包括 cdnverify 子網域名稱。 例如 cdnverify.www。

    - 指向：輸入 CDN 端點的主機名稱，包括 cdnverify 子網域名稱。 例如 cdnverify.contoso.azureedge.net。 

    - TTL：保留已選取的 [1 小時]。

6. 選取 [ **儲存**]。
 
    CNAME 項目會新增至 DNS 記錄資料表。

    ![DNS 記錄資料表](./media/cdn-map-content-to-custom-domain/cdn-cdnverify-dns-table.png)


## <a name="associate-the-custom-domain-with-your-cdn-endpoint"></a>將自訂網域與您的 CDN 端點產生關聯

在註冊您的自訂網域之後，您可以將其新增至 CDN 端點。 

1. 登入 [Azure 入口網站](https://portal.azure.com/)，然後瀏覽至要對應到自訂網域的端點所在的 CDN 設定檔。
    
2. 在 [CDN 設定檔] 頁面上，選取要與自訂網域產生關聯的 CDN 端點。

   [端點] 頁面隨即開啟。
    
3. 選取 [自訂網域]。 

   ![CDN 自訂網域按鈕](./media/cdn-map-content-to-custom-domain/cdn-custom-domain-button.png)

   [新增自訂網域] 頁面隨即開啟。

4. 對於 [端點主機名稱]，系統會從您的 CDN 端點 URL 衍生要作為 CNAME 記錄之目的地網域的端點主機名稱，並預先填入：*&lt;endpoint hostname&gt;*.azureedge.net。 無法予以變更。

5. 針對 [自訂主機名稱]，請輸入您的自訂網域 (包括子網域)，以作為 CNAME 記錄的來源網域。 例如 www.contoso.com 或 cdn.contoso.com。 請勿使用 cdnverify 子網域名稱。

   ![CDN 自訂網域對話方塊](./media/cdn-map-content-to-custom-domain/cdn-add-custom-domain.png)

6. 選取 [新增] 。

   Azure 會確認您所輸入的自訂網域名稱有 CNAME 記錄存在。 如果 CNAME 正確，就會驗證您的自訂網域。 

   新的自訂網域設定傳播至所有 CDN 邊緣節點可能需要一些時間： 
    - 若為**來自 Microsoft 的標準 Azure CDN** 設定檔，通常會在 10 分鐘內完成傳播。 
    - 若為**來自 Akamai 的標準 Azure CDN** 設定檔，通常會在一分鐘內完成傳播。 
    - 若為**來自 Verizon 的標準 Azure CDN** 和**來自 Verizon 的進階 Azure CDN** 設定檔，通常會在 10 分鐘內完成傳播。   


## <a name="verify-the-custom-domain"></a>驗證自訂網域

完成註冊自訂網域之後，請確認自訂網域參照您的 CDN 端點。
 
1. 確定您有在端點上快取的公用內容。 例如，如果您的 CDN 端點與儲存體帳戶相關聯，則 Azure CDN 會快取公用容器中的內容。 若要測試自訂網域，請確認您的容器設定為允許公用存取，且至少包含一個檔案。

2. 在瀏覽器中，使用自訂網域瀏覽至檔案的位址。 例如，如果您的自訂網域是 cdn.contoso.com，則快取檔案的 URL 應會類似於下列 URL：http:\//cdn.contoso.com/my-public-container/my-file.jpg。 確認結果與您直接在 *&lt;endpoint hostname&gt;*.azureedge.net 上存取 CDN 端點時相同。


## <a name="map-the-permanent-custom-domain"></a>對應永久自訂網域

如果您已確認 cdnverify 子網域已成功對應至您的端點 (或者，如果您使用不在生產環境中的新自訂網域)，則可以將自訂網域直接對應至 CDN 端點主機名稱。

若要建立自訂網域的 CNAME 記錄：

1. 登入自訂網域的網域提供者網站。

2. 找到管理 DNS 記錄的頁面 (可查閱提供者的文件或，尋找網站上標示**網域名稱**、**DNS** 或**名稱伺服器管理**的區域)。 

3. 建立自訂網域的 CNAME 記錄項目，並完成下表說明的欄位 (欄位名稱可能有所不同)：

    | 來源          | 類型  | 目的地           |
    |-----------------|-------|-----------------------|
    | www.contoso.com | CNAME | contoso.azureedge.net |

    - 來源：輸入您的自訂網域名稱 (例如 www.contoso.com)。

    - 類型：輸入 *CNAME*。

    - 目的地：輸入您的 CDN 端點主機名稱。 此名稱必須是下列格式：_&lt;端點名稱&gt;_.azureedge.net。 例如 contoso.azureedge.net。

4. 儲存您的變更。

5. 如果您先前曾建立暫時 cdnverify 子網域 CNAME 記錄，請將其刪除。 

6. 如果您是第一次在生產環境中使用此自訂網域，請遵循[將自訂網域與您的 CDN 端點產生關聯](#associate-the-custom-domain-with-your-cdn-endpoint)和[驗證自訂網域](#verify-the-custom-domain)的步驟。

例如，GoDaddy 網域註冊機構的程序如下所示：

1. 登入並選取您要使用的自訂網域。

2. 在 [網域] 區段中選取 [管理全部]，然後選取 [DNS] | [管理區域]。

3. 針對 [網域名稱]，輸入您的自訂網域，然後選取 [搜尋]。

4. 在 [DNS 管理] 頁面中選取 [新增]，然後在 [類型] 清單中選取 [CNAME]。

5. 完成 CNAME 項目的欄位：

    ![CNAME 項目](./media/cdn-map-content-to-custom-domain/cdn-cname-entry.png)

    - 類型：保留已選取的 [CNAME]。

    - 主機：輸入要使用的自訂網域的子網域。 例如 www 或 cdn。

    - 指向：輸入 CDN 端點的主機名稱。 例如 contoso.azureedge.net。 

    - TTL：保留已選取的 [1 小時]。

6. 選取 [ **儲存**]。
 
    CNAME 項目會新增至 DNS 記錄資料表。

    ![DNS 記錄資料表](./media/cdn-map-content-to-custom-domain/cdn-dns-table.png)

7. 如果您有 cdnverify CNAME 記錄，請選取它旁邊的鉛筆圖示，然後選取資源回收筒圖示。

8. 選取 [刪除]，將 CNAME 記錄刪除。


## <a name="clean-up-resources"></a>清除資源

在先前的步驟中，您已將自訂網域新增至 CDN 端點。 如果您不想再讓您的端點與自訂網域相關聯，您可以執行下列步驟以移除自訂網域：
 
1. 在您的 CDN 設定檔中，選取您要移除的自訂網域所在的端點。

2. 在 [端點] 頁面的 [自訂網域] 下，以滑鼠右鍵按一下您要移除的自訂網域，然後從內容功能表中選取 [刪除]。  

   自訂網域會與您的端點解除關聯。


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> - 建立 CNAME DNS 記錄。
> - 將自訂網域與您的 CDN 端點產生關聯。
> - 驗證自訂網域。

請移至下一個教學課程，以了解如何在 Azure CDN 自訂網域上設定 HTTPS。

> [!div class="nextstepaction"]
> [教學課程：在 Azure CDN 自訂網域上設定 HTTPS](cdn-custom-ssl.md)


