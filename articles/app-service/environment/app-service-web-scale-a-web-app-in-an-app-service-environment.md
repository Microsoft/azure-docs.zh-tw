---
title: 如何在 App Service 環境中調整應用程式
description: 在 App Service 環境中調整應用程式
services: app-service
documentationcenter: ''
author: ccompy
manager: stefsch
editor: jimbe
ms.assetid: 78eb1e49-4fcd-49e7-b3c7-f1906f0f22e3
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/17/2016
ms.author: ccompy
ms.openlocfilehash: d04a5fce920dae25507cdf2f64832574e24c51dd
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "22986313"
---
# <a name="scaling-apps-in-an-app-service-environment"></a>在 App Service 環境中調整應用程式
在 Azure App Service 中，您通常有三件事可以調整：

* 定價方案
* 背景工作角色大小 
* 執行個體數目。

在 ASE 中不需要選取或變更定價方案。  就功能而言，它已經是 Premium 定價功能層級。  

關於背景工作角色大小，ASE 系統管理員可以指派要用於每個背景工作角色集區的計算資源大小。  這表示您可以有具 P4 計算資源的背景工作集區 1，以及具 P1 計算資源的背景工作集區 2 (如有需要的話)。  它們並沒有大小順序。  如需大小及其定價的詳細資訊，請參閱此處的文件 [Azure App Service 定價][AppServicePricing]。  以下是 App Service 環境中的 Web 應用程式和 App Service 方案的調整選項：

* 背景工作集區選取
* 執行個體數目

如需變更任一項目，都可以透過針對 ASE 託管之 App Service 方案所顯示的適用 UI 完成。  

![][1]

ASP 相應增加的數量無法超過 ASP 所在背景工作集區中可用的計算資源數量。  如果背景工作集區中需要計算資源，您必須讓 ASE 系統管理員增加資源。  如需重新設定 ASE 的資訊，請閱讀以下資訊：[如何設定 App Service 環境][HowtoConfigureASE]。  您也可能需利用 ASE 自動調整功能，以根據排程或計量增加容量。  若要取得設定 ASE 環境本身自動調整的相關詳細資訊，請參閱[如何設定 App Service 環境的自動調整][ASEAutoscale]。

您可以使用來自不同背景工作集區或相同背景工作集區的計算資源，建立多個 App Service 方案。  例如，如果在背景工作集區 1 中有 (10) 個可用的計算資源，您可以選擇使用 (6) 個計算資源建立一個 App Service 方案，而第二個 App Service 方案使用 (4) 個計算資源。

### <a name="scaling-the-number-of-instances"></a>調整執行個體數目
當您初次在 App Service 環境中建立 Web 應用程式時，它會從 1 個執行個體開始。  您可以再相應放大至其他的執行個體，為應用程式提供額外的計算資源。   

如果 ASE 有足夠的容量，這就很簡單。  您可移至具有您要相應增加之站台的 App Service 方案，然後選取 [調整]。  這會開啟 UI，您可以在當中為 ASP 手動設定調整或設定自動調整規則。  若要手動調整應用程式，只需將 [調整依據] 設為 [手動輸入的執行個體計數]。  從這裡拖曳滑桿至所需的數量，或將所需數量輸入滑桿旁邊的方塊。  

![][2] 

在 ASE 中的 ASP 自動調整規則與一般運作方式相同。  您可以選取 [調整依據] 下方的 [CPU 百分比]，根據 CPU 百分比建立 ASP 的自動調整規則，或使用 [排程和效能規則] 建立更複雜的規則。  若要查看有關設定自動調整更完整的詳細資訊，請參閱此處的指南 [Azure App Service 中調整 Web 應用程式規模][AppScale]。 

### <a name="worker-pool-selection"></a>背景工作集區選取
如前所述，背景工作角色集區選項需透過 ASP UI 存取。  開啟您想要調整的 ASP 刀鋒視窗，選取 [背景工作集區]。  您會看到您已在 App Service 環境中設定的所有背景工作集區。  如果您只有一個背景工作集區，您只會看到一個集區列出。  若要變更 ASP 所在的背景工作角色集區，您只需選取 App Service 方案所要移入的背景工作角色集區。  

![][3]

將 ASP 從一個背景工作角色集區移到另一個集區之前，請務必確定您有足夠的容量可容納該 ASP。  在背景工作集區清單中，不只會列出背景工作集區名稱，您也可以查看該背景工作集區中有多少可用的背景工作。  請確定有足夠的執行個體可包含您的 App Service 方案。  如果您需要在背景工作集區中移入更多計算資雲，則必須讓您的 ASE 系統管理員加以新增。  

> [!NOTE]
> 從一個背景工作集區移出 ASP，會導致該 ASP 中的應用程式冷啟動。  這可能會導致要求的執行速度變慢，因為您的應用程式在新的計算資源上是冷啟動。  使用 Azure App Service 中的[應用程式準備功能][AppWarmup]可以避免冷啟動。  本文所述的應用程式初始化模組對冷啟動也有作用，因為當應用程式在新的計算資源上冷啟動時，也會叫用初始化程序。 
> 
> 

## <a name="getting-started"></a>開始使用
若要開始使用 App Service 環境，請參閱[如何建立 App Service 環境][HowtoCreateASE]

<!--Image references-->
[1]: ./media/app-service-web-scale-a-web-app-in-an-app-service-environment/aseappscale-aspblade.png
[2]: ./media/app-service-web-scale-a-web-app-in-an-app-service-environment/aseappscale-manualscale.png
[3]: ./media/app-service-web-scale-a-web-app-in-an-app-service-environment/aseappscale-sizescale.png

<!--Links-->
[WhatisASE]: app-service-app-service-environment-intro.md
[ScaleWebapp]: ../web-sites-scale.md
[HowtoCreateASE]: app-service-web-how-to-create-an-app-service-environment.md
[HowtoConfigureASE]: app-service-web-configure-an-app-service-environment.md
[CreateWebappinASE]: app-service-web-how-to-create-a-web-app-in-an-ase.md
[Appserviceplans]: ../azure-web-sites-web-hosting-plans-in-depth-overview.md
[AppServicePricing]: http://azure.microsoft.com/pricing/details/app-service/ 
[ASEAutoscale]: app-service-environment-auto-scale.md
[AppScale]: ../web-sites-scale.md
[AppWarmup]: http://ruslany.net/2015/09/how-to-warm-up-azure-web-app-during-deployment-slots-swap/
