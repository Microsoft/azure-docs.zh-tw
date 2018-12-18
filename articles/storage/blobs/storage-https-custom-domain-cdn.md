---
title: 使用 Azure CDN 透過 HTTPS 以自訂網域存取 blob
description: 了解如何整合 Azure CDN 與 Blob 儲存體，以透過 HTTPS 使用自訂網域存取 blob
services: storage
author: michaelhauss
ms.service: storage
ms.topic: article
ms.date: 06/26/2018
ms.author: mihauss
ms.component: blobs
ms.openlocfilehash: 7c4acc7d0832442b94735619ea3a01cb319da993
ms.sourcegitcommit: d4c076beea3a8d9e09c9d2f4a63428dc72dd9806
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/01/2018
ms.locfileid: "39398250"
---
# <a name="using-the-azure-cdn-to-access-blobs-with-custom-domains-over-https"></a>使用 Azure CDN 透過 HTTPS 以自訂網域存取 blob
Azure 內容傳遞網路 (CDN) 現在支援自訂網域名稱使用 HTTPS。 您可以利用這個功能，使用您的自訂網域透過 HTTPS 存取儲存體 blob。 若要這樣做，您必須先在您的 blob 或 Web 端點啟用 Azure CDN，並將 CDN 對應至自訂網域名稱。 一旦您採取這些步驟，啟用自訂網域 HTTPS 就會簡化為一步啟用和完整憑證管理，而且不須要 CDN 價格以外的費用。

這項功能很重要，因為它讓您可以在傳輸敏感的 Web 應用程式資料時保護隱私與資料完整性。 使用 SSL 通訊協定透過 HTTPS 提供流量，可確保資料在網際網路上傳遞時已經過加密。 HTTPS 提供信任與認證，並保護您的 Web 應用程式免於攻擊。

> [!NOTE]  
> 除了提供自訂網域名稱的 SSL 支援，Azure CDN 也可協助您調整應用程式規模，以便在世界各地供應高頻寬內容。 若要深入了解，請參閱 [Azure CDN 概觀](../../cdn/cdn-overview.md)。

## <a name="quick-start"></a>快速入門
以下是您的自訂 Blob 儲存體端點啟用 HTTPS 所需的步驟︰

1.  [整合 Azure 儲存體帳戶與 Azure CDN](../../cdn/cdn-create-a-storage-account-with-cdn.md)。
    如果您還沒有在 Azure 入口網站中建立儲存體帳戶，本文將逐步引導您。

    > [!NOTE]  
    > 在 Azure 儲存體中的靜態網站支援預覽期間，從 [來源類型] 下拉式功能表選取 [自訂來源]，以新增您的儲存體 Web 端點。 在 Azure 入口網站中，您必須從 CDN 設定檔中執行此作業，而不是直接在儲存體帳戶中執行。

2.  [將 Azure CDN 內容對應至自訂網域](../../cdn/cdn-map-content-to-custom-domain.md)。
3.  [在 Azure CDN 自訂網域上啟用 HTTPS](../../cdn/cdn-custom-ssl.md)。

## <a name="shared-access-signatures"></a>共用存取簽章
如果您的 Blob 儲存體端點設定為不允許匿名讀取權限，則必須在您對您的自訂網域提出的每個要求中提供[共用存取簽章 (SAS)](../common/storage-dotnet-shared-access-signature-part-1.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) 權杖。 根據預設，Blob 儲存體端點不允許匿名讀取權限。 如需共用存取簽章的詳細資訊，請參閱[管理對容器與 blob 的匿名讀取權限](storage-manage-access-to-resources.md)。

Azure CDN 不理會任何加在 SAS 權杖上的限制。 例如，所有 SAS 權杖都有到期時間。 這表示使用過期的 SAS 仍可存取內容，直到內容從 CDN 邊緣節點上被清除。 您可以設定快取的回應標頭，以控制可在 CDN 上快取資料多久時間。 如需指示請參閱[在 Azure CDN 中管理 Azure 儲存體 Blob 的到期](../../cdn/cdn-manage-expiration-of-blob-content.md)。

如果您為同一 blob 端點建立多個 SAS URL，建議您開啟 Azure CDN 的查詢字串快取。 這是為了確保每個 URL 會被當作唯一實體。 如需詳細資訊，請參閱[使用查詢字串控制 Azure CDN 快取行為](../../cdn/cdn-query-string.md)。

## <a name="http-to-https-redirection"></a>HTTP 至 HTTPS 的重新導向
您可以選擇將 HTTP 流量重新導向至 HTTPS。 這需要使用 Verizon 上提供的 Azure CDN 進階供應項目。 您必須用下列規則[使用 Azure CDN 規則引擎覆寫 HTTP 行為](../../cdn/cdn-rules-engine.md)：

![](./media/storage-https-custom-domain-cdn/redirect-to-https.png)

"Cdn-endpoint-name" 是指您為 CDN 端點設定的名稱。 您可以從下拉式清單中選取其值。 "Origin-path" 指的是您的靜態內容所在的原始儲存體帳戶的路徑。 如果您將所有靜態內容裝載在單一容器中，則將 "origin-path" 取代為該容器的名稱。

如需深入了解規則，請參閱 [Azure CDN 規則引擎功能](../../cdn/cdn-rules-engine-reference-features.md)。

## <a name="pricing-and-billing"></a>價格和計費
當您透過 Azure CDN 存取 blob 時，需支付邊緣節點和起點 (Blob 儲存體) 之間流量的 [Blob 儲存體價格](https://azure.microsoft.com/pricing/details/storage/blobs/)，以及從邊緣節點存取資料的 [CDN 價格](https://azure.microsoft.com/pricing/details/cdn/)。

例如，假設您的儲存體帳戶在美國西部，且使用 Azure CDN 存取它。 如果有人在英國嘗試透過 CDN 存取該儲存體帳戶中的其中一個 blob，Azure 會先檢查該 blob 最接近英國的邊緣節點。 如果找到，它會存取該 blob 複本且會使用 CDN 定價，因為是正在 CDN 上存取。 如果找不到，Azure 會將 blob 複製到邊緣節點，這會產生輸出和交易費用 (如「Blob 儲存體價格」中所述)，然後存取邊緣節點上的檔案，而這會產生 CDN 費用。

查看 [CDN 價格頁面](https://azure.microsoft.com/pricing/details/cdn/)時請注意，自訂網域名稱的 HTTPS 支援只適用於 Verizon 的 Azure CDN 產品 (「標準」和「進階」)。

## <a name="next-steps"></a>後續步驟
* [針對 Blob 儲存體端點設定自訂網域名稱](storage-custom-domain-name.md)
* [Azure 儲存體中的靜態網站託管 (預覽)](storage-blob-static-website.md)
