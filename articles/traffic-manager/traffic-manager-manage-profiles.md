---
title: 管理 Azure 流量管理員設定檔 | Microsoft Docs
description: 本文會協助您建立、停用、啟用及刪除 Azure 流量管理員設定檔。
services: traffic-manager
documentationcenter: ''
author: kumudd
manager: timlt
editor: ''
ms.assetid: f06e0365-0a20-4d08-b7e1-e56025e64f66
ms.service: traffic-manager
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/10/2017
ms.author: kumud
ms.openlocfilehash: e19f8f373fed8c7e9ab64f2fc9e34eba182af717
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/21/2018
ms.locfileid: "29398051"
---
# <a name="manage-an-azure-traffic-manager-profile"></a>管理 Azure 流量管理員設定檔

流量管理員設定檔會使用流量路由方法來控制分散到您雲端服務或網站端點的流量。 本文說明如何建立及管理這些設定檔。

## <a name="create-a-traffic-manager-profile"></a>建立流量管理員設定檔

您可以使用 Azure 入口網站來建立流量管理員設定檔。 建立設定檔之後，您就可以在 Azure 入口網站中設定端點、監視及其他設定。 流量管理員每個設定檔最多支援 200 個端點。 不過，大部分的使用案例僅需要少量的端點。

### <a name="to-create-a-traffic-manager-profile"></a>若要建立流量管理員設定檔

1. 從瀏覽器登入 [Azure 入口網站](http://portal.azure.com)。 如果您沒有帳戶，您可以註冊[免費試用一個月](https://azure.microsoft.com/free/)。 
2. 按一下 [建立資源] > [網路] > [流量管理員設定檔] > [建立]。
4. 在 [建立流量管理員設定檔] 中，如下所示操作：
    1. 在 [名稱]中，提供設定檔的名稱。 此名稱在 trafficmanager.net 區域內必須是唯一的，而且會產生 DNS 名稱 <name>, trafficmanager.net，用以存取您的流量管理員設定檔。
    2. 在 [路由方法] 中，選取 [優先順序]路由方法。
    3. 在 [訂用帳戶] 中，選取您要用來建立此設定檔的訂用帳戶
    4. 在 [資源群組]中，建立新的資源群組來放置此設定檔。
    5. 在 [資源群組位置] 中，選取資源群組的位置。 這項設定是指資源群組的位置，完全不影響將部署到全球的流量管理員設定檔。
    6. 按一下頁面底部的 [新增] 。
    7. 當流量管理員設定檔的全球部署完成時，它會列為個別資源群組的其中一個資源。

## <a name="disable-enable-or-delete-a-profile"></a>停用、啟用或刪除設定檔

您可以停用現有的設定檔，流量管理員就不會將使用者要求轉介到設定的端點。 當您停用流量管理員設定檔時，設定檔和設定檔內含的資訊將保持不變，而且可在流量管理員介面中進行編輯。  當您重新啟用設定檔時，就會繼續轉介。 當您在 Azure 入口網站中建立流量管理員設定檔時，它會自動啟用。 如果您決定不再需要某個設定檔，可以將它刪除。

### <a name="to-disable-a-profile"></a>停用設定檔

1. 如果您使用的是自訂網域名稱，則變更網際網路 DNS 伺服器上的 CNAME 記錄，它就不會再指向流量管理員設定檔。
2. 流量會停止導向至透過流量管理員設定檔設定的端點。
3. 從瀏覽器登入 [Azure 入口網站](http://portal.azure.com)。
2. 在入口網站的搜尋列中，搜尋您想要修改的**流量管理員設定檔**名稱，然後按一下結果中顯示的流量管理員設定檔。
3. 按一下 [概觀] > [停用]。
4. 確認停用流量管理員設定檔。

### <a name="to-enable-a-profile"></a>啟用設定檔

1. 從瀏覽器登入 [Azure 入口網站](http://portal.azure.com)。
2. 在入口網站的搜尋列中，搜尋您想要修改的**流量管理員設定檔**名稱，然後按一下結果中顯示的流量管理員設定檔。
3. 按一下 [概觀] > [啟用]。
1. 如果您使用的是自訂網域名稱，請在網際網路 DNS 伺服器上建立 CNAME 資源記錄，以指向流量管理員設定檔的網域名稱。
2. 流量會再次導向至各端點。

### <a name="to-delete-a-profile"></a>刪除設定檔

1. 請確定網際網路 DNS 伺服器上的 DNS 資源記錄所使用的 CNAME 資源記錄，不再指向流量管理員設定檔的網域名稱。
2. 在入口網站的搜尋列中，搜尋您想要修改的**流量管理員設定檔**名稱，然後按一下結果中顯示的流量管理員設定檔。
3. 按一下 [概觀] > [刪除]。
4. 確認刪除流量管理員設定檔。

## <a name="next-steps"></a>後續步驟

* [新增端點。](traffic-manager-endpoints.md)
* [設定優先順序路由方法](traffic-manager-configure-priority-routing-method.md)
* [設定地理路由方法](traffic-manager-configure-geographic-routing-method.md) 
* [設定加權路由方法](traffic-manager-configure-weighted-routing-method.md)
* [設定效能路由方法](traffic-manager-configure-performance-routing-method.md)
