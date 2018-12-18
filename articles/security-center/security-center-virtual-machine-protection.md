---
title: 保護 Azure 資訊安全中心內的機器和應用程式 | Microsoft Docs
description: 本文件說明資訊安全中心為了協助您保護虛擬機器和電腦以及 Web 應用程式和 App Service 環境所提供的建議。
services: security-center
documentationcenter: na
author: rkarlin
manager: MBaldwin
editor: ''
ms.assetid: 47fa1f76-683d-4230-b4ed-d123fef9a3e8
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/28/2018
ms.author: rkarlin
ms.openlocfilehash: 7cf5f86d9a2d121ff54c40e27c6bc50847a4dfdf
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43132625"
---
# <a name="protecting-your-machines-and-applications-in-azure-security-center"></a>保護 Azure 資訊安全中心內的機器和應用程式
「Azure 資訊安全中心」會分析 Azure 資源的安全性狀態。 當資訊安全中心發現潛在的安全性弱點時，它會建立可引導您完成所需控制之設定程序的建議。 這些建議適用於下列 Azure 資源類型︰虛擬機器 (VM) 和電腦、應用程式、網路、SQL、身分識別與存取。

本文說明適用於機器和應用程式的建議。

## <a name="monitoring-security-health"></a>監視安全性健全狀況
您可在 [資訊安全中心 – 概觀] 儀表板上監視資源的安全性狀態。 [資源] 區段提供為每個資源類型識別到的問題數目和安全性狀態。

您可以選取 [建議] 來檢視所有問題的清單。 如需有關如何套用建議的詳細資訊，請參閱[實作 Azure 資訊安全中心的安全性建議](security-center-recommendations.md)。

如需計算和應用程式服務建議的完整清單，請參閱[建議](security-center-virtual-machine-recommendations.md)。

若要繼續，請選取 [資源] 或 [資訊安全中心] 主功能表下方的 [計算和應用程式]。
![資訊安全中心儀表板][1]

## <a name="monitor-compute-and-app-services"></a>監視計算和應用程式服務
[計算] 下方有四個索引標籤：

- **概觀**：資訊安全中心所識別的監視和建議。
- **VM 和電腦**：列出您的 VM、電腦和各項目目前的安全性狀態。
- **雲端服務**︰列出資訊安全中心所監視的 Web 和背景工作角色。
- **應用程式服務 (預覽)**：列出您的應用程式服務環境和各項目目前的安全性狀態。
若要繼續，請選取 [資源] 或 [資訊安全中心] 主功能表下方的 [計算和應用程式]。

![計算][2]

在您可有多個區段的每個索引標籤中，以及在每個區段中，您可以選取一個個別的選項，以查看有關解決該特定問題之建議步驟的更多詳細資訊。

### <a name="monitoring-recommendations"></a>監視建議
本節顯示已初始化以進行自動佈建的 VM 和電腦總數，及其目前的狀態。 在此範例中有一個建議：[監視代理程式健康情況問題]。 選取這個建議。

![監視代理程式健康情況問題][3]

[監視代理程式健康情況問題] 隨即開啟。 其中會列出資訊安全中心無法成功監視的 VM 和電腦。 選取 VM 或電腦即可獲得詳細資訊。 [監視狀態] 會提供資訊安全中心為何無法監視的原因。 請參閱[資訊安全中心疑難排解指南](security-center-troubleshooting-guide.md)，以取得 [監視狀態] 值、描述和解決步驟的清單。

### 未受監視的 VM 和電腦<a name="unmonitored-vms-and-computers"></a>
如果機器未執行 Microsoft Monitoring Agent 擴充功能，VM 或電腦就不會由資訊安全中心監視。 機器可能已安裝本機代理程式，例如 OMS 直接代理程式或 SCOM 代理程式。 具有這些代理程式的機器會識別為未受監視，因為資訊安全中心無法完整支援這些代理程式。 若要能完整享有資訊安全中心的所有功能，則需要 Microsoft Monitoring Agent 擴充功能。

除了已安裝的本機代理程式，您還可以在未受監視的 VM 或電腦上安裝擴充功能。 以相同方式設定這兩種代理程式，並連線至相同工作區。 這可讓資訊安全中心與 Microsoft Monitoring Agent 擴充功能互動和收集資料。 請參閱[啟用 VM 擴充公能](../log-analytics/log-analytics-quick-collect-azurevm.md)，取得如何安裝 Microsoft Monitoring Agent 擴充功能的指示。

請參閱[監視代理程式健康情況問題](security-center-troubleshooting-guide.md#mon-agent)，深入了解資訊安全中心無法成功監視為了自動佈建而初始化之 VM 和電腦的原因。

### <a name="recommendations"></a>建議
此區段會分別就資訊安全中心所監視的每個 VM 和電腦、Web 和背景工作角色、Azure App Service Web 應用程式和 Azure App Service 環境提供一組建議。 第一個資料行會列出建議。 第二個資料行則顯示受該項建議影響的資源總數。 第三個資料行顯示問題的嚴重性，如下列螢幕擷取畫面所示：

![建議][4]

每個建議在經選取後，都會有一組可供執行的動作。 如果，如果您選取 [遺漏系統更新]，將會出現遺漏修補程式的 VM 和電腦數目，以及遺漏更新的嚴重性，如下列螢幕擷取畫面所示：

![套用系統更新][5]

[套用系統更新] 有圖形格式的重大更新摘要 (一個適用於 Windows，一個適用於 Linux)。 第二部分有包含下列資訊的表格：

- **名稱**：遺漏更新的名稱。
- **VM 和電腦數目**：遺漏此更新的 VM 和電腦總數。
- **更新嚴重性**：說明該特定建議的嚴重性：

    - **重大**：某個有意義的資源 (應用程式、虛擬機器或網路安全性群組) 有弱點存在，並且需要注意。
    - **重要**：需要執行非關鍵性步驟或其他步驟才能完成程序或消除弱點。
    - **中**：應該處理但不需要立即注意的弱點。 (預設不會顯示嚴重性低的建議，但是如果您要檢視它們，則可以篩選嚴重性低的建議)。


- **狀態**：建議的目前狀態：

    - **未處理**：建議尚未處理。
    - **進行中**：正在將建議套用到資源，且您不需要採取任何動作。
    - **已解決**︰建議動作已完成  (若問題已解決，該項目會呈現暗灰色)。

若要檢視建議的詳細資料，請按一下清單中遺漏更新的名稱。

![建議的詳細資料][6]

> [!NOTE]
> 此處的安全性建議與 [建議] 圖格下的相同。 如需如何處理建議的詳細資訊，請參閱[實作 Azure 資訊安全中心的安全性建議](security-center-recommendations.md)一文。
>
>

### <a name="vms-and-computers"></a>VM 和電腦
[VM 和電腦] 區段提供所有 VM 和電腦的建議概觀。 每個資料行代表一組建議，如下列螢幕擷取畫面所示：

![VM 和電腦的建議][7]

此清單中列出四種類型的圖示：

![非 Azure 電腦][8] 非 Azure 電腦。

![Azure Resource Manager VM][9] Azure Resource Manager VM。

![Azure 傳統 VM][10] Azure 傳統 VM。

![從工作區識別的 VM][11] 僅只從已檢視訂用帳戶中的工作區識別的 VM。 這包括來自向此訂用帳戶中的工作區回報之其他訂用帳戶的 VM，以及使用 SCOM 直接代理程式安裝的 VM (沒有資源識別碼)。

每個建議底下出現的圖示，可協助您快速識別需要關注的 VM 和電腦，以及建議的類型。 您也可以使用 [篩選] 選項，選取您會在這個畫面上看到的選項。

![Filter][12]

在上述範例中，有一個 VM 具有與端點保護有關的重要建議。 選取該 VM 可取得其詳細資訊：

![重大建議][13]

您可在此處查看 VM 或電腦的安全性詳細資料。 您可以在底部看到建議的動作和每個問題的嚴重性。

### <a name="cloud-services"></a>雲端服務
對於雲端服務，若作業系統版本過時，便會建立建議，如下列螢幕擷取畫面所示︰

![雲端服務][14]

在您具有建議的情況下 (這不是前一個範例的案例)，您需要依照建議步驟來更新作業系統版本。 有可用的更新時，您會收到警示 (紅色或橙色 - 取決於問題的嚴重性)。 當您選取 WebRole1 (使用自動部署至 IIS 的 Web 應用程式執行 Windows Server) 或 WorkerRole1 (使用自動部署至 IIS 的 Web 應用程式執行 Windows Server) 資料列中的這個警示，您可看到這項建議的相關詳細資料，如下列螢幕擷取畫面所示︰

![WorkerRole1][15]

若要查看有關這項建議的更多規範說明，請按一下 [描述] 資料行之下的 [更新OS 版本]。

![更新作業系統版本][16]

### <a name="app-services-preview"></a>應用程式服務 (預覽)

> [!NOTE]
> 監視 App Service 是預覽功能，僅適用於資訊安全中心的標準層。 若要深入了解資訊安全中心的定價層，請參閱[價格](security-center-pricing.md)。
>
>

在 [應用程式服務] 下方，您會看到應用程式服務環境的清單，並可根據資訊安全中心執行的評估檢視健康情況摘要。

![應用程式服務][17]

此清單中列出三種類型的圖示：

![應用程式服務環境][18] 應用程式服務環境。

![Web 應用程式][19] Web 應用程式。

![函式應用程式][24] 函式應用程式。

1. 選取 Web 應用程式。 摘要檢視隨即開啟，並有三個索引標籤：

  - **建議**：根據資訊安全中心所執行的失敗評估。
  - **已通過的評估**：資訊安全中心所執行、已通過的評估清單。
  - **無法使用的評估**：列出因錯誤或建議與特定應用程式服務無關而無法執行的評估

  [建議] 下方會列出為選取的 Web 應用程式提供的建議，和每項建議的嚴重性。

  ![摘要檢視][20]

2. 選取建議可獲得建議的描述，以及狀況不良、狀況良好和未掃描的資源清單。

  ![建議描述][21]

  [已通過的評估] 底下有已通過的評估清單。  這些評估的嚴重性永遠是綠色的。

  ![已通過的評估][22]

3. 從清單中選取已通過的評估，以取得評估的描述，以及狀況不良和狀況良好的資源清單，和未掃描的資源清單。 狀況不良的資源會有索引標籤，但由於已通過評估，因此該清單一律是空白的。

    ![狀況良好的資源][23]



## <a name="next-steps"></a>後續步驟
若要深入了解適用於其他 Azure 資源類型的建議，請參閱下列文章︰


* [了解 Azure 資訊安全中心對虛擬機器的建議](security-center-virtual-machine-recommendations.md)
* [在 Azure 資訊安全中心監視身分識別和存取](security-center-identity-access.md)
* [保護 Azure 資訊安全中心內的網路](security-center-network-recommendations.md)
* [保護 Azure 資訊安全中心內的 Azure SQL 服務](security-center-sql-service-recommendations.md)

如要深入了解資訊安全中心，請參閱下列主題：

* [在 Azure 資訊安全中心設定安全性原則](security-center-policies.md) -- 了解如何為您的 Azure 訂用帳戶及資源群組設定安全性原則。
* [管理與回應 Azure 資訊安全中心的安全性警示](security-center-managing-and-responding-alerts.md) -- 了解如何管理與回應安全性警示。
* [Azure 資訊安全中心常見問題集](security-center-faq.md) -- 尋找有關使用服務的常見問題。

<!--Image references-->
[1]: ./media/security-center-virtual-machine-recommendations/overview.png
[2]: ./media/security-center-virtual-machine-recommendations/compute.png
[3]: ./media/security-center-virtual-machine-recommendations/monitoring-agent-health-issues.png
[4]: ./media/security-center-virtual-machine-recommendations/compute-recommendations.png
[5]: ./media/security-center-virtual-machine-recommendations/apply-system-updates.png
[6]: ./media/security-center-virtual-machine-recommendations/missing-update-details.png
[7]: ./media/security-center-virtual-machine-recommendations/vm-computers.png
[8]: ./media/security-center-virtual-machine-recommendations/security-center-monitoring-icon1.png
[9]: ./media/security-center-virtual-machine-recommendations/security-center-monitoring-icon2.png
[10]: ./media/security-center-virtual-machine-recommendations/security-center-monitoring-icon3.png
[11]: ./media/security-center-virtual-machine-recommendations/security-center-monitoring-icon4.png
[12]: ./media/security-center-virtual-machine-recommendations/filter.png
[13]: ./media/security-center-virtual-machine-recommendations/vm-detail.png
[14]: ./media/security-center-virtual-machine-recommendations/security-center-monitoring-fig1-new006-2017.png
[15]: ./media/security-center-virtual-machine-recommendations/security-center-monitoring-fig8-new3.png
[16]: ./media/security-center-virtual-machine-recommendations/security-center-monitoring-fig8-new4.png
[17]: ./media/security-center-virtual-machine-recommendations/app-services.png
[18]: ./media/security-center-virtual-machine-recommendations/ase.png
[19]: ./media/security-center-virtual-machine-recommendations/web-app.png
[20]: ./media/security-center-virtual-machine-recommendations/recommendation.png
[21]: ./media/security-center-virtual-machine-recommendations/recommendation-desc.png
[22]: ./media/security-center-virtual-machine-recommendations/passed-assessment.png
[23]: ./media/security-center-virtual-machine-recommendations/healthy-resources.png
[24]: ./media/security-center-virtual-machine-recommendations/function-app.png
