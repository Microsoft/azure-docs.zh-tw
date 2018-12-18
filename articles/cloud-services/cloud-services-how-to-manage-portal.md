---
title: 常見的雲端服務管理工作 | Microsoft Docs
description: 了解如何在 Azure 入口網站中管理雲端服務。 這些範例使用 Azure 入口網站。
services: cloud-services
documentationcenter: ''
author: jpconnock
manager: timlt
editor: ''
ms.assetid: cb218ad9-77d4-4149-83db-71159c00767e
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/05/2017
ms.author: jeconnoc
ms.openlocfilehash: e9f4153c68f0a2a4ce83f900ff63152311163ff6
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40038038"
---
# <a name="manage-cloud-services-in-the-azure-portal"></a>在 Azure 入口網站中管理雲端服務
在 Azure 入口網站的 [雲端服務] 區域中，您可以：

* 更新服務角色或部署。
* 將預備部署升格為生產部署。
* 將資源連結至您的雲端服務，以便您查看資源相依性，並一起調整資源。
* 刪除雲端服務或部署。

如需如何調整雲端服務的詳細資訊，請參閱[在入口網站中設定雲端服務的自動調整](cloud-services-how-to-scale-portal.md)。

## <a name="update-a-cloud-service-role-or-deployment"></a>更新雲端服務角色或部署
如果您需要更新雲端服務的應用程式程式碼，請使用雲端服務刀鋒視窗上的 [ **更新** ]。 您可以更新單一角色或所有角色。 您可以上傳新的服務封裝或服務組態檔來更新。

1. 在 [Azure 入口網站][Azure portal]中，選取您想要更新的雲端服務。 這會開啟雲端服務執行個體刀鋒視窗。

2. 在刀鋒視窗上，選取 [更新]。

    ![更新按鈕](./media/cloud-services-how-to-manage-portal/update-button.png)

3. 使用新的服務封裝檔 (.cspkg) 和服務組態檔 (.cscfg) 更新部署。

    ![UpdateDeployment](./media/cloud-services-how-to-manage-portal/update-blade.png)

4. (選擇性) 更新儲存體帳戶和部署標籤。

5. 如果有任何角色只有一個角色執行個體，請選取 [即使一個或多個角色包含單一執行個體，也要部署] 核取方塊，讓升級能夠繼續。

    要讓 Azure 保證服務在雲端服務更新期間有 99.95% 的可用性，每個角色都至少必須有兩個角色執行個體 (虛擬機器)。 有兩個角色執行個體，其中一部虛擬機器會在另一部虛擬機器更新時處理用戶端要求。

6. 在套件上傳完成之後，選取 [開始部署] 核取方塊以套用更新。

7. 選取 [確定] 開始更新服務。

## <a name="swap-deployments-to-promote-a-staged-deployment-to-production"></a>交換部署以使預備部署升格為生產部署
當您決定部署新版的雲端服務時，可以在您的雲端服務預備環境中，預備並測試新版本。 使用 [交換]  將這兩個部署的 URL 位址互換，並將新版本升級為生產部署。

您可以在 [雲端服務]  頁面或儀表板交換部署。

1. 在 [Azure 入口網站][Azure portal]中，選取您想要更新的雲端服務。 這會開啟雲端服務執行個體刀鋒視窗。

2. 在刀鋒視窗上，選取 [交換]。

    ![[雲端服務交換] 按鈕](./media/cloud-services-how-to-manage-portal/swap-button.png)

3. 隨即開啟下列確認提示：

    ![Cloud Services Swap](./media/cloud-services-how-to-manage-portal/swap-prompt.png)

4. 確認部署資訊之後，選取 [是] 來交換部署。

    交換部署的速度很快，因為唯一變更的是部署的虛擬 IP 位址 (VIP)。

    為了節省運算成本，您可以在確認生產部署如預期運作之後刪除預備部署。

### <a name="common-questions-about-swapping-deployments"></a>交換部署的相關常見問題

**交換部署有哪些必要條件？**

成功的部署交換有兩個主要必要條件：

- 如果您想要讓生產環境位置使用靜態 IP 位址，就必須為您的預備位置也保留一個靜態 IP 位址。 否則，交換會失敗。

- 您的所有角色執行個體必須都處於執行中狀態，您才能執行交換。 您可以在 Azure 入口網站的 [概觀] 刀鋒視窗上檢查您的執行個體狀態。 或者，也可以使用 Windows PowerShell 的 [Get-AzureRole](/powershell/module/servicemanagement/azure/get-azurerole?view=azuresmps-3.7.0) 命令。

請注意，客體 OS 更新和服務修復作業也可能導致部署交換失敗。 如需詳細資訊，請參閱[對雲端服務部署問題進行疑難排解](cloud-services-troubleshoot-deployment-problems.md)。

**交換是否會導致我的應用程式停止運作？我應該如何處理該情況？**

如上一節中所述，部署交換通常很快，因為它只是 Azure 負載平衡器中的設定變更。 在某些情況下，它會花費 10 秒以上的時間，而導致暫時性的連線失敗。 若要限縮對您客戶造成的影響，請考慮實作[用戶端重試邏輯](../best-practices-retry-general.md)。

## <a name="delete-deployments-and-a-cloud-service"></a>刪除部署和雲端服務
刪除雲端服務之前，您必須先刪除每個現有的部署。

為了節省運算成本，您可以在確認生產部署如預期運作之後刪除預備部署。 您需要為停止的已部署角色執行個體支付運算成本。

使用下列程序，刪除部署或雲端服務。

1. 在 [Azure 入口網站][Azure portal]中，選取您想要刪除的雲端服務。 這會開啟雲端服務執行個體刀鋒視窗。

2. 在刀鋒視窗上，選取 [刪除]。

    ![[雲端服務刪除] 按鈕](./media/cloud-services-how-to-manage-portal/delete-button.png)

3. 若要刪除整個雲端服務，請選取 [雲端服務和其部署] 核取方塊。 您可以選擇 [生產部署] 或 [預備部署] 核取方塊。

    ![雲端服務刪除](./media/cloud-services-how-to-manage-portal/delete-blade.png)

4. 選取底部的 [刪除]。

5. 若要刪除雲端服務，請選取 [刪除雲端服務]。 然後，在確認提示處選取 [是]。

> [!NOTE]
> 當刪除雲端服務並且已設定詳細監視時，您必須從儲存體帳戶中手動刪除資料。 如需在哪尋找計量資料表的資訊，請參閱[雲端服務監視簡介](cloud-services-how-to-monitor.md)。


## <a name="find-more-information-about-failed-deployments"></a>尋找失敗部署的詳細資訊
[概觀] 刀鋒視窗上方有狀態列。 當您選取該列時，即會開啟新的刀鋒視窗，並顯示任何錯誤訊息。 如果部署不包含任何錯誤，則 [資訊] 刀鋒視窗會是空白的。

![雲端服務概觀](./media/cloud-services-how-to-manage-portal/status-info.png)



[Azure portal]: https://portal.azure.com

## <a name="next-steps"></a>後續步驟
* [雲端服務的一般設定](cloud-services-how-to-configure-portal.md)。
* 了解如何 [部署雲端服務](cloud-services-how-to-create-deploy-portal.md)。
* 設定 [自訂網域名稱](cloud-services-custom-domain-name-portal.md)。
* 設定 [SSL 憑證](cloud-services-configure-ssl-certificate-portal.md)。
