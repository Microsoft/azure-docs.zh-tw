---
title: 包含檔案
description: 包含檔案
services: signalr
author: wesmc7777
ms.service: signalr
ms.topic: include
ms.date: 04/17/2018
ms.author: wesmc
ms.custom: include file
ms.openlocfilehash: c26e2de2c492c51c35527979ef0450ed7c2de499
ms.sourcegitcommit: d7725f1f20c534c102021aa4feaea7fc0d257609
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37100822"
---
1. 若要建立新的 Azure SignalR Service 資源，請先登入 [Azure 入口網站](https://portal.azure.com)。 在頁面的左上方，按一下 [+ 建立資源]。 在 [搜尋 Marketplace] 文字方塊中，輸入 **SignalR Service**，然後按 **Enter**。

2. 在結果中按一下 [SignalR Service]，然後按一下 [建立]。

3. 在新的 **SignalR** 設定頁面中，為新的 SignalR 資源新增下列設定：

    | Name | 建議值 | 說明 |
    | ---- | ----------------- | ----------- |
    | 資源名稱 | *testsignalr* | 輸入要對 SignalR 資源使用的唯一資源名稱。 名稱必須是介於 1 到 63 個字元的字串，而且只能包含數字、字母和 `-` 字元。 名稱的開頭或結尾不能是 `-` 字元，且連續的 `-` 字元無效。|
    | 訂用帳戶 | 選擇您的訂用帳戶 |  選取您要用來測試 SignalR 的 Azure 訂用帳戶。 如果您的帳戶僅有一個訂用帳戶，則會自動加以選取，而且不會顯示 [訂用帳戶] 下拉式清單。|
    | 資源群組 | 建立名為 SignalRTestResources 的新資源群組| 選取或建立 SignalR 資源的資源群組。 此群組可用於組織多個資源，以便能夠藉由刪除資源群組來同時刪除多個資源。 如需詳細資訊，請參閱[使用資源群組管理您的 Azure 資源](../articles/azure-resource-manager/resource-group-overview.md)。 |
    | 位置 | 美國東部 | 使用 [位置] 來指定 SignalR 資源的裝載所在地理位置。 為獲得最佳效能，建議您在與應用程式其他元件相同的區域中建立資源。 |
    | 定價層 | *免費* | 目前有**免費**和**標準**選項可供使用。 |
    | 釘選到儀表板 | ✔ | 核取此方塊可將資源釘選到儀表板以方便尋找。 |

4. 按一下頁面底部的 [新增] 。 部署需要幾分鐘的時間才能完成。

5. 部署完成後，按一下 [設定] 下方的 [金鑰]。 複製主要金鑰連接字串。 您稍後會將此字串用於設定應用程式，使其使用 Azure SignalR Service 資源。

    連接字串的格式如下：
    
        Endpoint=<service_endpoint>;AccessKey=<access_key>;
