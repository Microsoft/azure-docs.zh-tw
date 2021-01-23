---
title: 針對雲端服務 (傳統) 部署問題進行疑難排解 |Microsoft Docs
description: 將雲端服務部署至 Azure 時，可能會發生幾個常見的問題。 本文章提供其中部分問題的解決方案。
ms.topic: article
ms.service: cloud-services
ms.date: 10/14/2020
ms.author: tagore
author: tanmaygore
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: 7b3d7a9a674aab3976da9399f71ff4d8df08eb62
ms.sourcegitcommit: 6272bc01d8bdb833d43c56375bab1841a9c380a5
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2021
ms.locfileid: "98741072"
---
# <a name="troubleshoot-azure-cloud-services-classic-deployment-problems"></a>針對 Azure 雲端服務 (傳統) 部署問題進行疑難排解

> [!IMPORTANT]
> [Azure 雲端服務 (延伸支援) ](../cloud-services-extended-support/overview.md) 是 Azure 雲端服務產品的新 Azure Resource Manager 型部署模型。透過這種變更，在以 Azure Service Manager 為基礎的部署模型上執行的 Azure 雲端服務，已重新命名為雲端服務 (傳統) ，而且所有新的部署都應該使用 [雲端服務 (延伸支援) ](../cloud-services-extended-support/overview.md)。

當您將雲端服務應用程式封裝部署至 Azure 時，您可以從 Azure 入口網站中的 [屬性]  窗格取得部署的相關資訊。 您可以利用此窗格中的詳細資料來排解雲端服務的問題，也可以在開啟新的支援要求時將這項資訊提供給 Azure 支援。

您可以依照下列方式找到 [屬性]  窗格：

* 在 Azure 入口網站中，依序按一下雲端服務的部署、[所有設定] 和 [屬性]。

> [!NOTE]
> 您可以按一下窗格右上角的圖示，將 [屬性] 窗格的內容複製到剪貼簿。
>
>

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="problem-i-cannot-access-my-website-but-my-deployment-is-started-and-all-role-instances-are-ready"></a>問題：我無法存取我的網站，但是我已啟動部署且所有角色執行個體皆已就緒
入口網站中顯示的網站 URL 連結未包含連接埠。 網站的預設連接埠為 80。 如果您的應用程式設定為在不同的連接埠執行，您必須在存取網站時將正確的連接埠號碼新增至 URL。

1. 在 Azure 入口網站中，按一下您的雲端服務部署。
2. 在 Azure 入口網站的 [屬性] 窗格中，查看角色執行個體的連接埠 (在 [輸入端點] 下方)。
3. 如果連接埠不是 80，請在存取應用程式時將正確的連接埠值新增至 URL。 若要指定非預設連接埠，請輸入 URL，後面依序加上冒號 (:) 和不含空格的連接埠號碼。

## <a name="problem-my-role-instances-recycled-without-me-doing-anything"></a>問題：我的角色執行個體無故自行回收
當 Azure 偵測到有問題的節點，並將角色執行個體移至新節點時，會自動進行服務修復。 發生此情況時，您可能會看見角色執行個體自動回收。 若要確認是否在執行服務修復：

1. 在 Azure 入口網站中，按一下您的雲端服務部署。
2. 在 Azure 入口網站的 [屬性]  窗格中檢閱相關資訊，並判斷在您觀察角色回收的期間是否執行了服務修復。

在主機 OS 和客體 OS 升級期間，角色大約每個月會回收一次。  
如需詳細資訊，請參閱部落格文章 [角色執行個體由於作業系統升級而重新啟動](/archive/blogs/kwill/role-instance-restarts-due-to-os-upgrades)

## <a name="problem-i-cannot-do-a-vip-swap-and-receive-an-error"></a>問題：我無法進行 VIP 交換並收到錯誤訊息
在部署更新進行期間不允許 VIP 交換。 部署更新可能在下列時間點自動執行：

* 新的客體作業系統可供使用時，和您設定要自動更新時。
* 服務修復執行時。

若要確認自動更新是否使您無法進行 VIP 交換：

1. 在 Azure 入口網站中，按一下您的雲端服務部署。
2. 在 Azure 入口網站的 [屬性] 窗格中，查看 [狀態] 的值。 如果是 **[就緒]**，則檢查 **[最後一個作業]**，看看是否是最近發生的某個作業讓您無法進行 VIP 交換。
3. 為生產部署重複步驟 1 和 2。
4. 如果自動更新正在進行中，請等候它完成再嘗試進行 VIP 交換。

## <a name="problem-a-role-instance-is-looping-between-started-initializing-busy-and-stopped"></a>問題：角色執行個體在 [已啟動]、[初始化中]、[忙碌] 和 [已停止] 之間循環
這種情況可能表示應用程式的程式碼、封裝或組態檔發生問題。 在此情況下，您應該能夠看見狀態每隔幾分鐘就會變更，且 Azure 入口網站可能顯示 [回收中]、[忙碌] 或 [正在初始化] 之類的訊息。 這表示應用程式發生了某些錯誤，導致角色執行個體無法執行。

如需如何對此問題進行疑難排解的詳細資訊，請參閱部落格文章 [Azure PaaS 計算診斷資料](/archive/blogs/kwill/windows-azure-paas-compute-diagnostics-data)和[導致角色回收的常見問題](cloud-services-troubleshoot-common-issues-which-cause-roles-recycle.md)。

## <a name="problem-my-application-stopped-working"></a>問題：我的應用程式停止運作
1. 在 Azure 入口網站中，按一下 [角色執行個體]。
2. 在 Azure 入口網站的 [屬性]  窗格中，考量下列狀況以解決您的問題：
   * 如果角色執行個體最近停止了 (您可以檢查 **[中止計數]** 的值)，則部署可能在更新。 請等候並觀察角色執行個體是否會自行恢復運作。
   * 如果角色執行個體處於 [忙碌] 狀態，請檢查應用程式的程式碼，查看 [StatusCheck](/previous-versions/azure/reference/ee758135(v=azure.100)) 事件是否已處理。 您可能需要新增或修正處理此事件的程式碼。
   * 請瀏覽部落格文章 [Azure PaaS 計算診斷資料](/archive/blogs/kwill/windows-azure-paas-compute-diagnostics-data)中的診斷資料及疑難排解案例。

> [!WARNING]
> 如果您回收雲端服務，您會重設部署的屬性，而有效清除原始問題的資訊。
>
>

## <a name="next-steps"></a>後續步驟
檢視更多雲端服務的 [疑難排解文章](./cloud-services-allocation-failures.md) 。

若要了解如何利用 Azure PaaS 電腦診斷資料對雲端服務角色問題進行疑難排解，請參閱 [Kevin Williamson 的部落格系列](/archive/blogs/kwill/windows-azure-paas-compute-diagnostics-data)。