---
title: 如何為輸入 IP 位址變更進行準備 - Azure
description: 如果您的輸入 IP 位址即將變更，請了解該怎麼做，以讓您的應用程式可在變更後繼續運作。
services: app-service\web
author: cephalin
manager: cfowler
editor: ''
ms.service: app-service-web
ms.workload: web
ms.topic: article
ms.date: 06/28/2018
ms.author: cephalin
ms.openlocfilehash: 28741e858b0c938ec8b2b2ff983106c6b08e18fc
ms.sourcegitcommit: 615403e8c5045ff6629c0433ef19e8e127fe58ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39578213"
---
# <a name="how-to-prepare-for-an-inbound-ip-address-change"></a>如何為輸入 IP 位址變更進行準備

如果您收到通知 Azure App Service 應用程式的輸入 IP 位址即將變更，請遵循本文中的指示。

## <a name="determine-if-you-have-to-do-anything"></a>判斷您是否需要執行任何動作

* 選項 1：如果您的 App Service 應用程式並沒有自訂網域，則不需執行任何動作。

* 選項 2：如果您的網域註冊入口網站 (第三方 DNS 提供者或 Azure DNS) 中只設定了 CNAME 記錄 (指向 URI 的 DNS 記錄)，則不需執行任何動作。

* 選項 3：如果您的網域註冊入口網站 (第三方 DNS 提供者或 Azure DNS) 中已設定 A 記錄 (直接指向 IP 位址的 DNS 記錄)，請將現有的 IP 位址取代為新位址。 遵循下一節中的指示，以找出新的 IP 位址。

* 選項 4：如果您的應用程式具有負載平衡器、IP 篩選器，或任何其他需要您應用程式 IP 位址的 IP 機制，請將現有的 IP 位址取代為新位址。 遵循下一節中的指示，以找出新的 IP 位址。

## <a name="find-the-new-inbound-ip-address-in-the-azure-portal"></a>在 Azure 入口網站中找出新的輸入 IP 位址

提供給您應用程式的新輸入 IP 位址是位於入口網站的**虛擬 IP 位址**欄位。 這個新 IP 位址和舊 IP 位址現在都會連線到您的應用程式，稍後舊位址將會中斷連線。

1.  開啟 [Azure 入口網站](https://portal.azure.com)。

2.  在左側導覽功能表中選取 [應用程式服務]。

3.  從清單中選取 App Service 應用程式。

4.  若該應用程式為函式應用程式，請參閱[函式應用程式輸入 IP 位址](../azure-functions/ip-addresses.md#function-app-inbound-ip-address)。

4.  在 [設定] 標題底下，按一下左側導覽中的 [屬性]，並找到標示為 [虛擬 IP 位址] 的區段。

5. 複製 IP 位址，並重新設定您的網域記錄或 IP 機制。

## <a name="next-steps"></a>後續步驟

本文說明如何針對由 Azure 起始的 IP 位址變更進行準備。 如需有關 Azure App Service 中 IP 位址的詳細資訊，請參閱 [Azure App Service 中的輸入和輸出 IP 位址](app-service-ip-addresses.md)。
