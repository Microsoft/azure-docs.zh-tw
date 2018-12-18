---
title: 包含檔案
description: 包含檔案
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 09/06/2018
ms.author: cherylmc
ms.openlocfilehash: 01a62fe7abb8a79f9afc08c0ff707cdfbb97ddac
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44343221"
---
每個使用點對站連線至 VNet 的用戶端電腦都必須安裝用戶端憑證。 用戶端憑證是從根憑證產生，並安裝在每部用戶端電腦上。 如果未安裝有效的用戶端憑證，且用戶端嘗試連線至 VNet，驗證將會失敗。

您可以為每個用戶端產生唯一的憑證，也可以對多個用戶端使用相同的憑證。 產生唯一用戶端憑證的優點是能夠撤銷單一憑證。 否則，如果多個用戶端使用相同的用戶端憑證，而您需要撤銷它時，就必須為所有使用該憑證進行驗證的用戶端產生並安裝新的憑證。

您可以使用下列方法來產生用戶端憑證︰

- **企業憑證︰**

  - 如果您使用企業憑證解決方案，請以一般的名稱值格式 'name@yourdomain.com' 產生用戶端憑證，而不要使用 'domain name\username' 格式。
  - 請確定用戶端憑證所根據的憑證範本，是以「用戶端驗證」(而不是「智慧卡登入」等) 作為使用清單中第一個項目的「使用者」憑證範本。您可以按兩下用戶端憑證，然後檢視 [詳細資料] > [增強金鑰使用方法]，來檢查憑證。

- **自我簽署的根憑證：** 您務必遵循以下其中一篇 P2S 憑證文章中的步驟。 否則，您所建立的用戶端憑證將無法與 P2S 連線相容，而且用戶端會在嘗試連線時收到錯誤訊息。 下列任一篇文章中的步驟都會產生相容的用戶端憑證： 

  * [Windows 10 PowerShell 指示](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site.md#clientcert)：這些指示需要 Windows 10 和 PowerShell，以產生憑證。 產生的憑證可以安裝在任何支援的 P2S 用戶端。
  * [MakeCert 指示](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site-makecert.md)：如果您無法存取 Windows 10 電腦來產生憑證，則可使用 MakeCert。 MakeCert 已被取代，但您仍可使用 MakeCert 來產生憑證。 產生的憑證可以安裝在任何支援的 P2S 用戶端。
  * [Linux 指示](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site-linux.md)

  當您使用先前的指示從自我簽署根憑證產生用戶端憑證，它會自動安裝在您用來產生它的電腦上。 如果您想要在另一部用戶端電腦上安裝用戶端憑證，您需要將它匯出為 .pfx (包含整個憑證鏈結)。 這會建立一個 .pfx 檔案，其中包含用戶端驗證成功所需的根憑證資訊。 如需匯出憑證的步驟，請參閱[憑證 - 匯出用戶端憑證](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site.md#clientexport)。