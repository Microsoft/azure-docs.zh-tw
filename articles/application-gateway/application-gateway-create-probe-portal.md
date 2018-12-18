---
title: 建立自訂探查 - Azure 應用程式閘道 - Azure 入口網站 | Microsoft Docs
description: 了解如何使用入口網站建立應用程式閘道的自訂探查
services: application-gateway
documentationcenter: na
author: vhorne
manager: jpconnock
editor: ''
tags: azure-resource-manager
ms.assetid: 33fd5564-43a7-4c54-a9ec-b1235f661f97
ms.service: application-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/26/2017
ms.author: victorh
ms.openlocfilehash: 45737c1c378ec56a5e2bedec8c1f7b7bc7ba6225
ms.sourcegitcommit: c47ef7899572bf6441627f76eb4c4ac15e487aec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/04/2018
ms.locfileid: "33203902"
---
# <a name="create-a-custom-probe-for-application-gateway-by-using-the-portal"></a>使用入口網站建立應用程式閘道的自訂探查

> [!div class="op_single_selector"]
> * [Azure 入口網站](application-gateway-create-probe-portal.md)
> * [Azure Resource Manager PowerShell](application-gateway-create-probe-ps.md)
> * [Azure 傳統 PowerShell](application-gateway-create-probe-classic-ps.md)

在本文中，您會透過 Azure 入口網站將自訂探查新增到現有的應用程式閘道。 對於具有特定健康狀態檢查頁面的應用程式，或是在預設 Web 應用程式上不提供成功回應的應用程式，自訂探查非常實用。

## <a name="before-you-begin"></a>開始之前

如果您還沒有應用程式閘道，請瀏覽[建立應用程式閘道](application-gateway-create-gateway-portal.md)以建立要使用的應用程式閘道。

## <a name="createprobe"></a>建立探查

您可以透過入口網站中兩個步驟的程序來設定探查。 第一個步驟是建立探查。 在第二個步驟中，將探查新增至應用程式閘道的後端 http 設定。

1. 登入 [Azure 入口網站](https://portal.azure.com)。 如果您沒有帳戶，可以註冊[免費試用一個月](https://azure.microsoft.com/free)

1. 在 Azure 入口網站的 [我的最愛] 窗格中，按一下 [所有資源]。 按一下 [所有資源] 刀鋒視窗中的應用程式閘道。 如果您選取的訂用帳戶已有幾個資源，您可以在 [依名稱篩選] 方塊中輸入 partners.contoso.net 輕鬆地存取應用程式閘道。

1. 按一下 [探查]，然後按一下 [加入] 按鈕來新增探查。

  ![已填入資訊的 [新增探查] 刀鋒視窗][1]

1. 在 [新增健康狀態探查] 刀鋒視窗中，填入探查的必要資訊，然後在完成時按一下 [確定]。

  |**設定** | **值** | **詳細資料**|
  |---|---|---|
  |**名稱**|customProbe|此值是可在入口網站中存取的易記探查名稱。|
  |**通訊協定**|HTTP 或 HTTPS | 健康狀態探查所使用的通訊協定。|
  |**Host**|亦即 contoso.com|此值是用於探查的主機名稱。 只有當應用程式閘道上設定多站台時適用，否則請使用 '127.0.0.1'。 此值與 VM 主機名稱不同。|
  |**路徑**|/ 或另一個路徑|自訂探查完整 URL 的其餘部分。 有效路徑的開頭為 '/'。 針對 http://contoso.com 的預設路徑，只要使用 '/' 即可 |
  |**間隔 (秒)**|30|執行探查以檢查健康狀態的頻率。 建議您不要設定低於 30 秒。|
  |**逾時 (秒)**|30|探查逾時前所等待的時間。逾時間隔需要高到足以進行 http 呼叫，以確保可使用後端的健康狀態頁面。|
  |**狀況不良臨界值**|3|視為狀況不良的失敗嘗試次數。 臨界值為 0 表示，如果健康狀態檢查失敗，後端會被立即斷定為狀況不良。|

  > [!IMPORTANT]
  > 主機名稱與伺服器名稱不同。 此值是在應用程式伺服器上執行的虛擬主機名稱。 探查會傳送到 http://(host name):(port from httpsetting)/urlPath

## <a name="add-probe-to-the-gateway"></a>將探查新增到閘道

既然已經建立探查，此時即可將它新增到閘道。 探查設定是在應用程式閘道的後端 http 設定上設定。

1. 按一下應用程式閘道的 [HTTP 設定] 來開啟組態刀鋒視窗，然後按一下視窗中列出的目前後端 http 設定。

  ![[https 設定] 視窗][2]

1. 在 [appGatewayBackEndHttpSettings] 設定刀鋒視窗中，選取 [使用自訂探查] 核取方塊，然後在 [自訂探查] 下拉式清單中選擇在[建立探查](#createprobe)一節中建立的探查。
完成時，按一下 [儲存] 即可套用設定。

預設探查會檢查 Web 應用程式的預設存取權。 建立自訂探查之後，應用程式閘道便可使用定義的自訂路徑來監視後端伺服器的健康狀態。 應用程式閘道會根據定義的準則，以檢查探查中指定的路徑。 如果對 host:Port/path 的呼叫沒有傳回 HTTP 200-399 狀態回應，當達到狀況不良閾值時，系統就會將該伺服器從輪替的行列中移除。 探查會繼續在狀況不良的執行個體上執行，以判斷它何時再次變成狀況良好。 系統將執行個體重新加回至狀況良好的伺服器集區之後，流量就會再次開始流向它，而對執行個體的探查則會如常依據使用者指定的間隔繼續執行。

## <a name="next-steps"></a>後續步驟

若要了解如何設定與「Azure 應用程式閘道」搭配運作的「SSL 卸載」，請參閱 [設定 SSL 卸載](application-gateway-ssl-portal.md)

[1]: ./media/application-gateway-create-probe-portal/figure1.png
[2]: ./media/application-gateway-create-probe-portal/figure2.png

