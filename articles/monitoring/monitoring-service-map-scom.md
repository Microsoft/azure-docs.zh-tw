---
title: 服務對應與 System Center Operations Manager 的整合 | Microsoft Docs
description: 服務對應是 Azure 中的一個解決方案，可自動探索 Windows 和 Linux 系統上的應用程式元件，並對應服務之間的通訊。 本文說明使用服務對應在 Operations Manager 中自動建立分散式應用程式圖表。
services: monitoring
documentationcenter: ''
author: daveirwin1
manager: jwhit
editor: tysonn
ms.assetid: e8614a5a-9cf8-4c81-8931-896d358ad2cb
ms.service: monitoring
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/21/2017
ms.author: bwren;dairwin
ms.openlocfilehash: 6fbc49584b040f952fdff147207864d2d1f6377e
ms.sourcegitcommit: 870d372785ffa8ca46346f4dfe215f245931dae1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/08/2018
ms.locfileid: "33887438"
---
# <a name="service-map-integration-with-system-center-operations-manager"></a>服務對應與 System Center Operations Manager 的整合
  > [!NOTE]
  > 這項功能處於公開預覽狀態。
  > 
  
服務對應可自動探索 Windows 和 Linux 系統上的應用程式元件，並對應服務之間的通訊。 服務對應可讓您以您對伺服器的理解方式 (一個提供重要服務的互連系統) 來檢視它們。 服務對應不需要進行任何設定，只要安裝代理程式就能顯示橫跨任何 TCP 連接架構的伺服器、處理程序和連接埠之間的連線。 如需詳細資訊，請參閱[服務對應文件]( monitoring-service-map.md)。

透過這項服務對應與 System Center Operations Manager 之間的整合，您便可以在根據服務對應中動態相依性對應的 Operations Manager 中，自動建立分散式應用程式圖表。

## <a name="prerequisites"></a>先決條件
* 管理一組伺服器的 Operations Manager 管理群組 (2012 R2 或更新版)。
* 已啟用服務對應解決方案的 Log Analytics 工作區。
* 由 Operations Manager 管理並傳送資料至服務對應的一組伺服器 (至少一部)。 支援 Windows 和 Linux 伺服器。
* 可存取與 Log Analytics 工作區相關聯之 Azure 訂用帳戶的服務主體。 如需詳細資訊，請移至[建立服務主體](#creating-a-service-principal)。

## <a name="install-the-service-map-management-pack"></a>安裝服務對應管理組件
您必須透過匯入 Microsoft.SystemCenter.ServiceMap 管理組件配套 (Microsoft.SystemCenter.ServiceMap.mpb)，以啟用 Operations Manager 與服務對應之間的整合。 您可以從 [Microsoft 下載中心](https://www.microsoft.com/download/details.aspx?id=55763)下載管理組件配套。 此配套包含下列管理組件︰
* Microsoft Service Map Application Views
* Microsoft System Center Service Map Internal
* Microsoft System Center Service Map Overrides
* Microsoft System Center Service Map

## <a name="configure-the-service-map-integration"></a>設定服務對應整合
在您安裝服務對應管理組件之後，[管理] 窗格中的 [Operations Management Suite] 底下會顯示新的節點 [服務對應]。 

若要設定服務對應整合，請執行下列動作︰

1. 若要開啟設定精靈，在 [服務對應概觀] 窗格中，按一下 [新增工作區]。  

    ![[服務對應概觀] 窗格](media/monitoring-service-map/scom-configuration.png)

2. 在 [連線設定] 視窗中，輸入租用戶名稱或識別碼、應用程式識別碼 (也稱為使用者名稱或用戶端識別碼) 以及服務主體的密碼，然後按一下 [下一步]。 如需詳細資訊，請移至[建立服務主體](#creating-a-service-principal)。

    ![[連線設定] 視窗](media/monitoring-service-map/scom-config-spn.png)

3. 在 [訂閱選取項目] 視窗中，選取 Azure 訂用帳戶、Azure 資源群組 (包含 Log Analytics 工作區的那個)，以及 Log Analytics 工作區，然後按 [下一步]。

    ![Operations Manager 設定工作區](media/monitoring-service-map/scom-config-workspace.png)

4. 在 [機器群組選取項目] 視窗中，您要選擇您想要同步至 Operations Manager 的服務對應機器群組。 按一下 [新增/移除機器群組]，從 [可用的機器群組] 清單中選擇群組，然後按一下 [新增]。  當您完成群組選取時，按一下 [確定] 來完成。
    
    ![Operations Manager 設定機器群組](media/monitoring-service-map/scom-config-machine-groups.png)
    
5. 在 [伺服器選取] 視窗中，設定服務對應伺服器群組，以及您想要在 Operations Manager 和服務對應之間同步的伺服器。 按一下 [新增/移除伺服器]。   
    
    若要讓整合針對伺服器建立分散式應用程式圖表，該伺服器必須是︰

    * 由 Operations Manager 管理
    * 由服務對應管理
    * 列於服務對應伺服器群組中

    ![Operations Manager 設定群組](media/monitoring-service-map/scom-config-group.png)

6. 選擇性 - 選取 [管理伺服器] 資源集區來與 Log Analytics 通訊，然後按一下 [新增工作區]。

    ![Operations Manager 設定資源集區](media/monitoring-service-map/scom-config-pool.png)

    設定和註冊 Log Analytics 工作區可能需要一分鐘的時間。 完成設定之後，Operations Manager 就會初始化首次的服務對應同步處理。

    ![Operations Manager 設定資源集區](media/monitoring-service-map/scom-config-success.png)


## <a name="monitor-service-map"></a>監視服務對應
連線至 Log Analytics 工作區之後，Operations Manager 主控台的 [監視] 窗格中會顯示新的 [服務對應] 資料夾。

![Operations Manager [監視] 窗格](media/monitoring-service-map/scom-monitoring.png)

服務對應資料夾有四個節點︰
* **作用中警示**：列出有關 Operations Manager 和服務對應之通訊的所有作用中警示。  請注意，這些警示不是要同步至 Operations Manager 的 Log Analytics 警示。 

* **伺服器**：列出設定為從服務對應同步處理的受監視伺服器。

    ![Operations Manager [監視伺服器] 窗格](media/monitoring-service-map/scom-monitoring-servers.png)

* **機器群組相依性檢視**：列出從服務對應同步處理的所有機器群組。 您可以按一下任何群組，以檢視其分散式應用程式圖表。

    ![Operations Manager 分散式應用程式圖表](media/monitoring-service-map/scom-group-dad.png)

* **伺服器相依性檢視**：列出從服務對應同步處理的所有伺服器。 您可以按一下任何伺服器，以檢視其分散式應用程式圖表。

    ![Operations Manager 分散式應用程式圖表](media/monitoring-service-map/scom-dad.png)

## <a name="edit-or-delete-the-workspace"></a>編輯或刪除工作區
您可以透過 [服務對應概觀] 窗格 ([系統管理] 窗格 > [Operations Management Suite] > [服務對應]) 編輯或刪除已設定的工作區。 您現在只能設定一個 Log Analytics 工作區。

![Operations Manager [編輯工作區] 窗格](media/monitoring-service-map/scom-edit-workspace.png)

## <a name="configure-rules-and-overrides"></a>設定規則和覆寫
系統會建立規則 _Microsoft.SystemCenter.ServiceMapImport.Rule_，以便定期從服務對應擷取資訊。 若要變更同步時間，您可以設定規則的覆寫 ([撰寫] 窗格 > [規則] > [Microsoft.SystemCenter.ServiceMapImport.Rule])。

![Operations Manager [覆寫內容] 視窗](media/monitoring-service-map/scom-overrides.png)

* **Enabled**：啟用或停用自動更新。 
* **IntervalMinutes**︰重設更新之間的時間。 預設間隔是一小時。 如果您想要更頻繁地同步伺服器對應，您可以變更此值。
* **TimeoutSeconds**︰重設要求逾時之前的時間長度。 
* **TimeWindowMinutes**：重設查詢資料的時間間隔。 預設間隔為 60 分鐘。 服務對應所允許的最大值為 60 分鐘。

## <a name="known-issues-and-limitations"></a>已知問題與限制

目前的設計有以下問題和限制︰
* 您只能連線到單一 Log Analytics 工作區。
* 雖然您可以透過 [撰寫] 窗格將伺服器手動新增至服務對應伺服器群組，但這些伺服器的對應不會立即進行同步。  這些對應會在下一個同步週期從服務對應進行同步處理。
* 如果您對管理組件所建立的「分散式應用程式圖表」進行任何變更，這些變更可能會在下一次與服務對應進行同步時遭到覆寫。

## <a name="create-a-service-principal"></a>建立服務主體
如需建立服務主體的官方 Azure 文件，請參閱︰
* [使用 PowerShell 建立服務主體](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-authenticate-service-principal)
* [使用 Azure CLI 建立服務主體](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-authenticate-service-principal-cli)
* [使用 Azure 入口網站建立服務主體](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-create-service-principal-portal)

### <a name="feedback"></a>意見反應
您對「服務對應」或這份文件有任何意見反應要提供給我們嗎？ 請瀏覽我們的 [User Voice 頁面 (英文)](https://feedback.azure.com/forums/267889-log-analytics/category/184492-service-map)，您可以在此頁面提出功能建議或對現有的建議進行投票。
