---
title: 在 Azure 應用程式閘道上啟用端對端 SSL
description: 本文將簡介應用程式閘道端對端 SSL 支援。
services: application-gateway
author: amsriva
ms.service: application-gateway
ms.topic: article
ms.date: 8/6/2018
ms.author: amsriva
ms.openlocfilehash: 4575bed18697a5661d58dc350c24a9497f7c46ff
ms.sourcegitcommit: 615403e8c5045ff6629c0433ef19e8e127fe58ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39578808"
---
# <a name="overview-of-end-to-end-ssl-with-application-gateway"></a>應用程式閘道端對端 SSL 的概觀

應用程式閘道支援在閘道上終止 SSL，之後流量通常會以未加密狀態流至後端伺服器。 這項功能可讓 Web 伺服器不必再負擔昂貴的加密和解密成本。 但對某些客戶來說，對後端伺服器進行未加密的通訊並非可接受的選項。 此未加密的通訊可能是有安全性需要、合規性需求，或應用程式可能只接受安全連線。 對於這類應用程式，應用程式閘道可支援端對端 SSL 加密。

端對端 SSL 可讓您將機密資料以加密方式安全地傳輸到後端，同時又可利用應用程式閘道提供的第 7 層負載平衡功能的好處。 部分功能為 cookie 型工作階段親和性、URL 型路由、支援根據站台或能注入 X-Forwarded-* 標頭的路由。

當設定為端對端 SSL 通訊模式時，應用程式閘道會在閘道上終止 SSL 工作階段，並解密使用者流量。 然後，它會套用所設定的規則來選取要將流量路由傳送到的適當後端集區執行個體。 應用程式閘道接著再起始連往後端伺服器的新 SSL 連線，並先使　用後端伺服器的公開金鑰憑證重新加密資料，再將要求傳輸至後端。 若要啟用端對端 SSL，請將 **BackendHTTPSetting** 中的通訊協定設為 HTTPS，接著再套用到後端集區。 已啟用端對端 SSL 之後端集區中的每個後端伺服器，都必須設有憑證以便能夠進行安全通訊。

![端對端 SSL 案例][1]

在此範例中，使用端對端 SSL，將使用 TLS1.2 的要求路由傳送至 Pool1 中的後端伺服器。

## <a name="end-to-end-ssl-and-whitelisting-of-certificates"></a>端對端 SSL 和憑證白名單

應用程式閘道只會與已知的後端執行個體通訊，後者已將其憑證加入到應用程式閘道的允許清單。 若要啟用憑證允許清單，您必須將後端伺服器憑證的公開金鑰上傳至應用程式閘道 (不是根憑證)。 於是，只允許連接至已知和白名單中的後端。 其餘的後端會導致閘道錯誤。 自我簽署憑證僅供測試之用，並不建議用於生產工作負載。 這類憑證必須如先前步驟所述，加入應用程式閘道的白名單之中，才能使用。

> [!NOTE]
> 受信任的 Azure 服務 (例如 Azure Web Apps) 不需進行驗證憑證設定。

## <a name="next-steps"></a>後續步驟

了解端對端 SSL 後，請移至[使用 Azure 入口網站透過 SSL 終止來設定應用程式閘道](create-ssl-portal.md)，以使用端對端 SSL 建立應用程式閘道。

<!--Image references-->

[1]: ./media/ssl-overview/scenario.png
