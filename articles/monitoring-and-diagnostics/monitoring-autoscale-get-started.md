---
title: 開始在 Azure 中自動調整規模
description: 了解如何在 Azure 中調整您的資源 Web 應用程式、雲端服務、虛擬機器或虛擬機器擴展集。
author: rajram
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 07/07/2017
ms.author: rajram
ms.component: autoscale
ms.openlocfilehash: e96e3abc05353aa3ea134b7d11256a29fe64c300
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46989221"
---
# <a name="get-started-with-autoscale-in-azure"></a>開始在 Azure 中自動調整規模
本文說明如何在 Microsoft Azure 入口網站中為您的資源設定自動調整規模。

Azure 監視器自動調整僅適用於[虛擬機器擴展集](https://azure.microsoft.com/services/virtual-machine-scale-sets/)、[雲端服務](https://azure.microsoft.com/services/cloud-services/)、[App Service - Web Apps](https://azure.microsoft.com/services/app-service/web/) 與 [API 管理服務](https://docs.microsoft.com/azure/api-management/api-management-key-concepts)。

## <a name="discover-the-autoscale-settings-in-your-subscription"></a>在您的訂用帳戶中探索自動調整規模設定
您可以在 Azure 監視器中探索適用於自動調整規模的所有資源。 如需逐步說明，請依循下列步驟：

1. 開啟 [Azure 入口網站。][1]
1. 按一下左窗格中的 Azure 監視器圖示。
  ![開啟 Azure 監視器][2]
1. 按一下 [自動調整規模]，檢視適用自動調整規模的所有資源，以及其目前的自動調整狀態。
  ![探索 Azure 監視器中的自動調整規模][3]

您可以使用頂端的篩選窗格來縮小清單範圍，以選取特定資源群組中的資源、特定的資源類型或特定的資源。

針對每個資源，您將會找到目前的執行個體計數及自動調整規模狀態。 自動調整規模狀態可以是：

- **未設定**︰您尚未針對此資源啟用自動調整規模。
- **已啟用**：您已針對此資源啟用自動調整規模。
- **已停用**：您已針對此資源停用自動調整規模。

## <a name="create-your-first-autoscale-setting"></a>建立您的第一個自動調整規模設定

現在讓我們逐步執行一個簡單的逐步解說，以建立您的第一個自動調整規模設定。

1. 開啟 Azure 監視器中的 [自動調整規模] 刀鋒視窗，然後選取您想要調整的資源。 (下列步驟會使用與某個 Web 應用程式相關聯的 App Service 方案。 您可以[在 5 分鐘內，將您的第一個 ASP.NET Web 應用程式建立在 Azure 中][4])
1. 請注意，目前的角色執行個體計數是 1。 按一下 [啟用自動調整規模]。
  ![適用於新 Web 應用程式的調整規模設定][5]
1. 提供調整規模設定的名稱，然後按一下 [新增規則]。 請注意，調整規模規則選項會在右邊開啟為內容窗格。 預設會將此設定為如果資源的 CPU 百分比超過百分之七十，就會將您的執行個體計數相應增加 1。 將其保留為預設值，然後按一下 [新增]。
  ![建立 Web 應用程式的調整規模設定][6]
1. 您現在已建立第一個調整規模規則。 請注意，UX 建議了最佳做法，並指出「建議規則中至少有一個調整規模」。 若要這樣做：

    a. 按一下 [新增規則]。

    b. 將**運算子**設定為**小於**。

    c. 將**閾值**設定為 **20**。

    d. 將**作業**設定為**減少計數依據**.

   您現在應該會有一個調整規模設定，其會根據 CPU 使用量進行相應放大/相應縮小。
   ![根據 CPU 調整規模][8]
1. 按一下 [檔案] 。

恭喜！ 您現在已成功建立第一個調整規模設定，可根據 CPU 使用量自動調整 Web 應用程式的規模。

> [!NOTE]
> 相同的步驟都適用於開始使用虛擬機器擴展集或雲端服務角色。

## <a name="other-considerations"></a>其他考量
### <a name="scale-based-on-a-schedule"></a>根據排程進行調整
除了根據 CPU 調整規模，您可以針對一週內的特定天數設定不同的調整規模。

1. 按一下 [新增調整規模的條件]。
1. 設定調整規模模式和規則，與預設條件一樣。
1. 針對排程選取 [重複特定天數]。
1. 選取天數，以及應套用調整規模條件的開始/結束時間。

![根據排程調整規模的條件][9]
### <a name="scale-differently-on-specific-dates"></a>在特定日期以不同方式調整規模
除了根據 CPU 調整規模，您可以針對特定日期設定不同的調整規模。

1. 按一下 [新增調整規模的條件]。
1. 設定調整規模模式和規則，與預設條件一樣。
1. 針對排程選取 [指定開始/結束日期]。
1. 選取應套用調整規模條件的開始/結束日期和開始/結束時間。

![根據日期調整規模的條件][10]

### <a name="view-the-scale-history-of-your-resource"></a>檢視資源的調整規模歷程記錄
每當相應增加或減少您的資源時，就會在活動記錄中記錄一個事件。 您可以藉由切換至 [執行歷程記錄] 索引標籤，來檢視過去 24 小時資源的調整規模歷程記錄。

![執行記錄][11]

如果您想要檢視完整的調整規模歷程記錄 (最多 90 天)，可選取 [按一下此處以查看詳細資料]。 活動記錄隨即開啟，其中含有針對您的資源與類別而預先選取的「自動調整規模」。

### <a name="view-the-scale-definition-of-your-resource"></a>檢視資源的調整規模定義
自動調整規模是 Azure Resource Manager 的資源。 您可以藉由切換至 [JSON] 索引標籤，在 JSON 中檢視調整規模定義。

![調整規模定義][12]

您可以視需要直接在 JSON 中進行變更。 變更將在儲存後生效。

### <a name="disable-autoscale-and-manually-scale-your-instances"></a>停用自動調整規模並手動調整執行個體規模
有時您可能想要停用目前的調整規模設定，並手動調整資源規模。

按一下頂端的 [停用自動調整規模] 按鈕。
![停用自動調整規模][13]

> [!NOTE]
> 此選項會停用您的設定。 不過，您可以重新啟用自動調整規模以取回設定。

您現在可以設定想要手動調整規模的執行個體數目。

![設定手動調整規模][14]

您隨時都能藉由依序按一下 [啟用自動調整規模] 和 [儲存]，來返回自動調整規模。

## <a name="next-steps"></a>後續步驟
- [建立活動記錄警示以監視訂用帳戶的所有自動調整引擎作業](https://github.com/Azure/azure-quickstart-templates/tree/master/monitor-autoscale-alert)
- [建立活動記錄警示以監視訂用帳戶中所有失敗的相應縮小/相應放大自動調整作業](https://github.com/Azure/azure-quickstart-templates/tree/master/monitor-autoscale-failed-alert)

<!--Reference-->
[1]:https://portal.azure.com
[2]: ./media/monitoring-autoscale-get-started/azure-monitor-launch.png
[3]: ./media/monitoring-autoscale-get-started/discover-autoscale-azure-monitor.png
[4]: https://docs.microsoft.com/azure/app-service/app-service-web-get-started-dotnet
[5]: ./media/monitoring-autoscale-get-started/scale-setting-new-web-app.png
[6]: ./media/monitoring-autoscale-get-started/create-scale-setting-web-app.png
[7]: ./media/monitoring-autoscale-get-started/scale-in-recommendation.png
[8]: ./media/monitoring-autoscale-get-started/scale-based-on-cpu.png
[9]: ./media/monitoring-autoscale-get-started/scale-condition-schedule.png
[10]: ./media/monitoring-autoscale-get-started/scale-condition-dates.png
[11]: ./media/monitoring-autoscale-get-started/scale-history.png
[12]: ./media/monitoring-autoscale-get-started/scale-definition-json.png
[13]: ./media/monitoring-autoscale-get-started/disable-autoscale.png
[14]: ./media/monitoring-autoscale-get-started/set-manualscale.png
