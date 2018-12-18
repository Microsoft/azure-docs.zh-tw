---
title: 如何為 SSL IP 位址變更進行準備 - Azure
description: 如果您的 SSL IP 位址即將變更，請了解該怎麼做，以讓您的應用程式可在變更後繼續運作。
services: app-service\web
author: cephalin
manager: cfowler
editor: ''
ms.service: app-service-web
ms.workload: web
ms.topic: article
ms.date: 06/28/2018
ms.author: cephalin
ms.openlocfilehash: e8558b4c3c7dafca8d4fff7e2aae0597a66c031d
ms.sourcegitcommit: 615403e8c5045ff6629c0433ef19e8e127fe58ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39576537"
---
# <a name="how-to-prepare-for-an-ssl-ip-address-change"></a>如何為 SSL IP 位址變更進行準備

如果您收到通知 Azure App Service 應用程式的 SSL IP 位址即將變更，請遵循本文中的指示，以發佈現有的 SSL IP 位址並指派新位址。

## <a name="release-ssl-ip-addresses-and-assign-new-ones"></a>發佈 SSL IP 位址並指派新的位址

1.  開啟 [Azure 入口網站](https://portal.azure.com)。

2.  在左側導覽功能表中選取 [應用程式服務]。

3.  從清單中選取 App Service 應用程式。

4.  在 [設定] 標題底下，按一下左側導覽中的 [SSL 設定]。

5. 在 [SSL 繫結] 區段，選取主機名稱列。 在開啟的編輯器中，從 [SSL 類型] 下拉式功能表中選擇 [SNI SSL]，然後按一下 [新增繫結]。 當您看到作業成功訊息時，現有的 IP 位址已發佈。

6.  在 [SSL 繫結] 區段中，再次選取具有憑證的同一主機名稱列。 在開啟的編輯器中，這次從 [SSL 類型] 下拉式功能表中選擇 [以 IP 為主的 SSL]，然後按一下 [新增繫結]。 當您看到作業成功訊息時，您已取得新的 IP 位址。

7.  如果您的網域註冊入口網站 (第三方 DNS 提供者或 Azure DNS) 中已設定 A 記錄 (直接指向 IP 位址的 DNS 記錄)，請將現有的 IP 位址取代為新產生的位址。 遵循下一節中的指示，以找出新的 IP 位址。

## <a name="find-the-new-ssl-ip-address-in-the-azure-portal"></a>在 Azure 入口網站中找出新的 SSL IP 位址

1.  等待幾分鐘，然後開啟 [Azure 入口網站](https://portal.azure.com)。

2.  在左側導覽功能表中選取 [應用程式服務]。

3.  從清單中選取 App Service 應用程式。

4.  在 [設定] 標題底下，按一下左側導覽中的 [屬性]，並找到標示為 [虛擬 IP 位址] 的區段。

5. 複製 IP 位址，並重新設定您的網域記錄或 IP 機制。

## <a name="next-steps"></a>後續步驟

本文說明如何針對由 Azure 起始的 IP 位址變更進行準備。 如需有關 Azure App Service 中 IP 位址的詳細資訊，請參閱 [Azure App Service 中的 SSL 和 SSL IP 位址](app-service-ip-addresses.md)。
