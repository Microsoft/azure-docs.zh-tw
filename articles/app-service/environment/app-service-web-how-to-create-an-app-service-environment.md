---
title: 如何建立 App Service 環境 v1
description: App Service 環境 v1 的建立流程說明
services: app-service
documentationcenter: ''
author: ccompy
manager: stefsch
editor: ''
ms.assetid: 81bd32cf-7ae5-454b-a0d2-23b57b51af47
ms.service: app-service
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 7/11/2017
ms.author: ccompy
ms.openlocfilehash: 2741ea2931ddd7989fc05e1cddbeedb80bf30410
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/21/2018
ms.locfileid: "29386617"
---
# <a name="how-to-create-an-app-service-environment-v1"></a>如何建立 App Service 環境 v1 

> [!NOTE]
> 這篇文章是關於 App Service 環境 v1。 有較新版本的 App Service 環境，更易於使用，並且可以在功能更強大的基礎結構上執行。 若要深入了解新版本，請從 [App Service 環境簡介](intro.md)開始。
> 

### <a name="overview"></a>概觀
App Service 環境 (ASE) 是 Azure App Service 的進階服務選項，可提供多租用戶戳記中不提供的增強式設定功能。 ASE 功能基本上會將 Azure App Service 部署到客戶的虛擬網路中。 若要更深入了解 App Service Environment 所提供的功能，請閱讀[什麼是 App Service Environment][WhatisASE] 文件。

### <a name="before-you-create-your-ase"></a>建立 ASE 之前
務必注意您無法變更的項目。 建立後，您無法變更 ASE 的相關層面是：

* 位置
* 訂用帳戶
* 資源群組
* 使用的 VNet
* 使用的子網路 
* 子網路大小

挑選 VNet 然後指定子網路時，請確定它足夠大以容納未來的成長。 

### <a name="creating-an-app-service-environment-v1"></a>建立 App Service 環境 v1
若要建立 App Service 環境 v1，您可以在 Azure Marketplace 中搜尋 App Service Environment v1，或透過 [建立資源]  ->  [Web + 行動]  ->  [App Service 環境] 執行這項操作。 若要建立 ASEv1：

1. 提供 ASE 的名稱。 您為 ASE 指定的名稱將用於在 ASE 中建立的應用程式。 如果 ASE 的名稱是 appsvcenvdemo，則子網域名稱會是 .appsvcenvdemo.p.azurewebsites.net。 如果您以此建立名為 mytestapp 的應用程式，會定址於 mytestapp.appsvcenvdemo.p.azurewebsites.net。 您無法在 ASE 的名稱中使用空白字元。 如果您在名稱中使用大寫字元，則網域名稱會是該名稱的全小寫版本。 如果您是使用 ILB，ASE 名稱就不會用於您的子網域中，但是會在 ASE 建立期間明確指定。
   
    ![][1]
2. 選取您的訂用帳戶。 您的 ASE 使用的訂用帳戶也會套用到您在該 ASE 中建立的所有應用程式。 您無法將您的 ASE 放在另一個訂用帳戶的 VNet 中。
3. 選取或指定新的資源群組。 用於 ASE 的資源群組必須是與用於您的 VNet 的相同。 如果您選取既有的 VNet，您的 ASE 資源群組選取項目將會更新，以反映 VNet 的資源群組。
   
    ![][2]
4. 請選取您的虛擬網路及位置選項。 您可以選擇建立新的 VNet，或選取既有的 VNet。 如果您選取新的 VNet，則可以指定名稱和位置。 新的 VNet 會有位址範圍 192.168.250.0/23，和定義為 192.168.250.0/24 名為 **default** 的子網路。 您也可以僅選取既有的傳統或 Resource Manager VNet。 [VIP 類型] 選項會決定您的 ASE 是否可以從網際網路 (外部) 直接存取，或者它使用內部負載平衡器 (ILB)。 若要深入了解，請參閱[在 App Service 環境中使用內部負載平衡器][ILBASE]。 如果您選取 [外部] VIP 類型，則可以選取系統針對 IPSSL 用途會建立幾個外部 IP 位址。 如果您選取 [內部]，則需要指定 ASE 將使用的子網域。 ASE 可以部署到使用公用位址範圍*或* RFC1918 位址空間 (也就是私人位址) 的虛擬網路。 若要搭配使用虛擬網路與公用位址範圍，您必須事先建立 VNet。 選取既有的 VNet 時，您必須在 ASE 建立期間建立新的子網路。 **您無法在入口網站中使用預先建立的子網路。如果您使用 Resource Manager 範本建立 ASE，則可以使用既有的子網路建立 ASE。** 若要使用此處的資訊透過範本建立 ASE，[從範本建立 App Service 環境][ILBAseTemplate]，以及這裡[從範本建立 ILB App Service 環境][ASEfromTemplate]。

### <a name="details"></a>詳細資料
一個 ASE 是使用 2 個前端和 2 個背景工作角色建立。 前端可做為 HTTP/HTTPS 端點，將流量傳送到背景工作角色 (也就是裝載您的應用程式的角色)。 建立 ASE 之後，您可以調整數量，且甚至可以在這些資源集區上設定自動調整規則。 如需更多關於手動調整、管理及監視 App Service 環境的詳細資料，請前往此處：[如何設定 App Service 環境][ASEConfig] 

只有一個 ASE 可以存在於 ASE 所使用的子網路中。 子網路無法用於 ASE 以外的任何項目

### <a name="after-app-service-environment-v1-creation"></a>在 App Service 環境 v1 建立之後
建立 ASE 之後，您可以調整：

* 前端的數量 (最小值：2)
* 背景工作的數量 (最小值：2)
* IP SSL 可用的 IP 位址數目
* 前端或背景工作所使用的計算資源大小 (前端大小下限為 P2)

以下有更多關於手動調整、管理及監視 App Service 環境的詳細資料：[如何設定 App Service 環境][ASEConfig] 

如需自動調整的相關資訊，請參閱：[如何設定 App Service 環境的自動調整][ASEAutoscale]

有其他無法自訂的相依性，例如資料庫和儲存體。 這些都是由 Azure 處理並由系統隨附。 系統儲存體對於整個 App Service 環境最多可支援 500 GB，且 Azure 會根據系統規模的需要來調整資料庫。

## <a name="getting-started"></a>開始使用
若要開始使用 App Service 環境 v1，請參閱 [App Service 環境 v1 簡介][WhatisASE]

[!INCLUDE [app-service-web-try-app-service](../../../includes/app-service-web-try-app-service.md)]

<!--Image references-->
[1]: ./media/app-service-web-how-to-create-an-app-service-environment/asecreate-basecreateblade.png
[2]: ./media/app-service-web-how-to-create-an-app-service-environment/asecreate-vnetcreation.png

<!--Links-->
[WhatisASE]: app-service-app-service-environment-intro.md
[ASEConfig]: app-service-web-configure-an-app-service-environment.md
[AppServicePricing]: http://azure.microsoft.com/pricing/details/app-service/ 
[ASEAutoscale]: app-service-environment-auto-scale.md
[ILBASE]: app-service-environment-with-internal-load-balancer.md
[ILBAseTemplate]: http://azure.microsoft.com/documentation/templates/201-web-app-ase-create/
[ASEfromTemplate]: app-service-app-service-environment-create-ilb-ase-resourcemanager.md
