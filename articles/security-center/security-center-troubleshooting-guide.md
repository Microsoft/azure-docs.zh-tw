---
title: Azure 資訊安全中心疑難排解指南 | Microsoft Docs
description: 本文件有助於在 Azure 資訊安全中心排解疑難問題。
services: security-center
documentationcenter: na
author: rkarlin
manager: mbaldwin
editor: ''
ms.assetid: 44462de6-2cc5-4672-b1d3-dbb4749a28cd
ms.service: security-center
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/26/2018
ms.author: rkarlin
ms.openlocfilehash: eebdff338454b1fb50b27d5b3d8c1c37d28f6b6f
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43121197"
---
# <a name="azure-security-center-troubleshooting-guide"></a>Azure 資訊安全中心疑難排解指南
本指南適用於組織目前採用 Azure 資訊安全中心，且需要針對資訊安全中心相關問題進行疑難排解的資訊技術 (IT) 專業人員、資訊安全性分析師和雲端系統管理員。

>[!NOTE]
>從 2017 年 6 月初開始，資訊安全中心會使用 Microsoft Monitoring Agent 來收集和儲存資料。 若要深入了解，請參閱 [Azure 資訊安全中心平台移轉](security-center-platform-migration.md)。 本文中的資訊說明轉換至 Microsoft Monitoring Agent 後的資訊安全中心功能。
>

## <a name="troubleshooting-guide"></a>疑難排解指南
本指南說明如何針對資訊安全中心相關問題進行疑難排解。 大多數在資訊安全中心進行的疑難排解作業會先查看失敗元件的[稽核記錄](https://azure.microsoft.com/updates/audit-logs-in-azure-preview-portal/)。 透過稽核記錄檔，您可以判斷︰

* 已發生的作業
* 起始作業的人員
* 發生作業的時間
* 作業的狀態
* 其他可能協助您研究作業的屬性值

稽核記錄檔包含在您的資源上執行的所有寫入作業 (PUT、POST、DELETE)，但不包含讀取作業 (GET)。

## <a name="microsoft-monitoring-agent"></a>Microsoft Monitoring Agent
資訊安全中心會使用 Microsoft Monitoring Agent (這是 Log Analytics 服務所使用的相同代理程式) 從 Azure 虛擬機器收集安全性資料。 啟用資料收集且代理程式已正確安裝在目標電腦之後，以下處理序應在執行中︰

* HealthService.exe

如果您開啟服務管理主控台 (services.msc)，您也會看到執行中的 Microsoft Monitoring Agent 服務，如下所示：

![服務](./media/security-center-troubleshooting-guide/security-center-troubleshooting-guide-fig5.png)

若要查看您擁有的代理程式版本，請開啟 [工作管理員]，在 [處理序] 索引標籤中找出 [Microsoft Monitoring Agent 服務]，以滑鼠右鍵按一下它並按一下 [屬性]。 在 [詳細資料] 索引標籤上，查看如下所示的檔案版本：

![檔案](./media/security-center-troubleshooting-guide/security-center-troubleshooting-guide-fig6.png)


## <a name="microsoft-monitoring-agent-installation-scenarios"></a>Microsoft Monitoring Agent 安裝案例
在電腦上安裝 Microsoft Monitoring Agent 時，有兩個可產生不同結果的安裝案例。 支援的案例如下：

* **資訊安全中心自動安裝的代理程式**：在此案例中，您能夠在兩個位置 (資訊安全中心和記錄搜尋) 檢視警示。 您會收到電子郵件通知，這些通知會寄送至在資源所屬訂用帳戶的安全性原則中設定的電子郵件地址。
.
* **手動安裝於 Azure 中 VM 上的代理程式**：在此案例中，如果您使用在 2017 年 2 月之前下載並手動安裝的代理程式，唯有依據工作區所屬的訂用帳戶篩選，您才能夠在資訊安全中心入口網站中檢視警示。 在您依據資源所屬的訂用帳戶篩選時，您將無法看到任何警示。 您會收到電子郵件通知，這些通知會寄送至在工作區所屬訂用帳戶的安全性原則中設定的電子郵件地址。

>[!NOTE]
> 若要避免第二個案例中說明的行為，務必下載最新版的代理程式。
>

## 監視代理程式健康情況問題<a name="mon-agent"></a>
[監視狀態] 可定義資訊安全中心無法成功監視為了自動佈建而初始化之 VM 和電腦的原因。 下表顯示 [監視狀態] 值、描述和解決步驟。

| 監視狀態 | 說明 | 解決步驟 |
|---|---|---|
| 擱置代理程式安裝 | Microsoft Monitoring Agent 安裝仍在執行中。  安裝作業可能需要多達數小時的時間。 | 等到自動安裝完成為止。 |
| 電源為關閉狀態 | VM 已停止。  Microsoft Monitoring Agent 只能安裝於執行中的 VM。 | 重新啟動 VM。 |
| Azure VM 代理程式遺漏或無效 | 尚未安裝 Microsoft Monitoring Agent。  資訊安全中心需要有效的 Azure VM 代理程式，才能安裝擴充功能。 | 在 VM 上安裝、重新安裝或升級 Azure VM 代理程式。 |
| VM 尚未處於安裝就緒的狀態  | 因為 VM 尚未處於安裝就緒的狀態，所以尚未安裝 Microsoft Monitoring Agent。 因為 VM 代理程式或 VM 佈建有問題，所以 VM 尚未處於安裝就緒的狀態。 | 檢查 VM 的狀態。 返回入口網站中的 [虛擬機器] 並選取 VM，以取得狀態資訊。 |
|安裝失敗 - 一般錯誤 | 已安裝 Microsoft Monitoring Agent，但因為發生錯誤而失敗。 | [手動安裝擴充功能](../log-analytics/log-analytics-quick-collect-azurevm.md#enable-the-log-analytics-vm-extension)或將擴充功能解除安裝，資訊安全中心就會嘗試再次安裝。 |
| 安裝失敗 - 已安裝本機代理程式 | Microsoft Monitoring Agent 安裝失敗。 資訊安全中心發現已安裝於 VM 上的本機代理程式 (OMS 或 SCOM)。 為了避免多路連接的設定 (在此設定中，VM 會向兩個不同的工作區回報)，已停止 Microsoft Monitoring Agent 的安裝。 | 解決方式有兩種：[手動安裝擴充功能](../log-analytics/log-analytics-quick-collect-azurevm.md#enable-the-log-analytics-vm-extension)並將它連線到您所需的工作區。 或者，將您所需的工作區設定為預設工作區，並啟用代理程式的自動佈建功能。  請參閱[啟用自動佈建](security-center-enable-data-collection.md)。 |
| 代理程式無法連線到工作區 | 已安裝 Microsoft Monitoring Agent，但因為網路連線問題而失敗。  請檢查是否有網際網路存取權，或已針對代理程式設定有效的 HTTP Proxy。 | 請參閱[監視代理程式網路需求](#troubleshooting-monitoring-agent-network-requirements)。 |
| 代理程式已連線到遺漏或未知的工作區 | 資訊安全中心發現安裝於 VM 上的 Microsoft Monitoring Agent 已連線到其無權存取的工作區。 | 有兩種情況會發生這種情形。 工作區已遭刪除而不復存在。 重新安裝具有正確工作區的代理程式，或將代理程式解除安裝並允許資訊安全中心完成其自動佈建安裝。 第二種情況是工作區屬於資訊安全中心無權存取的訂用帳戶。 資訊安全中心需有訂用帳戶，才可允許 Microsoft 安全性資源提供者進行存取。 若要啟用這項功能，請向 Microsoft 安全性資源提供者註冊訂用帳戶。 此作業可經由 API、PowerShell、入口網站完成，而在資訊安全中心的 [概觀] 儀表板中直接篩選訂用帳戶亦可完成。 如需詳細資訊，請參閱[資源提供者和類型](../azure-resource-manager/resource-manager-supported-services.md#portal)。 |
| 代理程式沒有回應或缺少識別碼 | 即使已安裝代理程式，資訊安全中心仍無法從 VM 擷取已掃描的安全性資料。 | 代理程式並未回報任何資料，包括活動訊號。 代理程式可能已損毀，或有物件封鎖流量。 或者，代理程式雖回報資料，但遺漏 Azure 資源識別碼，因此無法比對資料與 Azure VM。 若要對 Linux 進行疑難排解，請參閱[適用於 Linux 的 OMS 代理程式疑難排解指南](https://github.com/Microsoft/OMS-Agent-for-Linux/blob/master/docs/Troubleshooting.md#im-not-seeing-any-linux-data-in-the-oms-portal)。 若要對 Windows 進行疑難排解，請參閱[對 Windows 虛擬機器進行疑難排解](https://github.com/MicrosoftDocs/azure-docs/blob/8c53ac4371d482eda3d85819a4fb8dac09996a89/articles/log-analytics/log-analytics-azure-vm-extension.md#troubleshooting-windows-virtual-machines)。 |
| 未安裝代理程式 | 資料收集已啟用。 | 在安全性原則中開啟資料收集，或手動安裝 Microsoft Monitoring Agent。 |


## 對監視代理程式網路需求進行疑難排解<a name="mon-network-req"></a>
代理程式若要連線到資訊安全中心並向其註冊，就必須能夠存取網路資源，包括連接埠號碼和網域 URL。

- 對於 Proxy 伺服器，您需要確保代理程式設定中已設定了適當的 Proxy 伺服器資源。 如需有關[如何變更 Proxy 設定](https://docs.microsoft.com/azure/log-analytics/log-analytics-windows-agents#configure-proxy-settings)的詳細資訊，請參閱這篇文章。
- 對於限制網際網路存取的防火牆，您需要設定防火牆以允許存取 Log Analytics。 您不需要在代理程式設定中進行任何動作。

下表說明通訊所需資源。

| 代理程式資源 | 連接埠 | 略過 HTTPS 檢查 |
|---|---|---|
| *.ods.opinsights.azure.com | 443 | 是 |
| *.oms.opinsights.azure.com | 443 | 是 |
| *.blob.core.windows.net | 443 | 是 |
| *.azure-automation.net | 443 | 是 |

如果您遇到代理程式的登入問題，請務必閱讀[如何針對 Operations Management Suite 登入問題進行疑難排解](https://support.microsoft.com/en-us/help/3126513/how-to-troubleshoot-operations-management-suite-onboarding-issues)一文。


## <a name="troubleshooting-endpoint-protection-not-working-properly"></a>針對端點保護無法正常運作的問題進行疑難排解

客體代理程式為 [Microsoft Antimalware](../security/azure-security-antimalware.md) 擴充功能所做一切的父處理序。 當客體代理程式處理序失敗時，做為客體代理程式子處理序執行的 Microsoft Antimalware 可能也會失敗。  在類似案例中，建議您確認下列選項：

- 目標 VM 是否為自訂映像，而且 VM 的建立者從未安裝客體代理程式。
- 如果目標為 Linux VM 而不是 Windows VM，則在 Linux VM 上安裝反惡意程式碼擴充功能的 Windows 版本將會失敗。 Linux 客體代理程式對於 OS 版本和所需的套件具有特定需求，而且如果未符合這些需求，VM 代理程式也無法在該處運作。
- VM 是否是使用舊版客體代理程式所建立。 如果是，您應該注意，某些舊版代理程式可能不會將本身自動更新為較新版本，而這會導致此問題。 如果建立您自己的映像，請一律使用最新版的客體代理程式。
- 某些第三方系統管理軟體可能會停用客體代理程式，或封鎖特定檔案位置的存取。 如果您在 VM 上安裝了第三方軟體，請確定代理程式位於排除清單上。
- 某些防火牆設定或網路安全性群組 (NSG) 可能會封鎖進出客體代理程式的網路流量。
- 某些存取控制清單 (ACL) 可能會防止磁碟存取。
- 磁碟空間不足會封鎖客體代理程式而使其無法正確運作。

預設會停用 Microsoft Antimalware 使用者介面，如需如何視需要啟用此功能的詳細資訊，請閱讀[在 Azure Resource Manager VM 後續部署上啟用 Microsoft Antimalware 使用者介面 (英文)](https://blogs.msdn.microsoft.com/azuresecurity/2016/03/09/enabling-microsoft-antimalware-user-interface-post-deployment/)。

## <a name="troubleshooting-problems-loading-the-dashboard"></a>針對載入儀表板的問題進行疑難排解

如果您遇到載入資訊安全中心儀表板的問題，請確定向資訊安全中心註冊訂用帳戶的使用者 (也就是第一位使用訂用帳戶開啟資訊安全中心的使用者) 以及想要開啟資料收集的使用者，應該是訂用帳戶的「擁有者」或「參與者」。 另外，從那時起，訂用帳戶上具有「讀取者」身分的使用者就可以看到儀表板/警示/建議/原則。

## <a name="contacting-microsoft-support"></a>連絡 Microsoft 支援服務
使用本文提供的指導方針可識別一些問題，您也可能發現的其他問題則記載於資訊安全中心的公開 [論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=AzureSecurityCenter)。 不過，如果您需要進一步疑難排解，您可以使用 **Azure 入口網站**開啟新的支援要求，如下所示︰

![Microsoft 支援服務](./media/security-center-troubleshooting-guide/security-center-troubleshooting-guide-fig2.png)


## <a name="see-also"></a>另請參閱
在本文件中，您已了解如何在「Azure 資訊安全中心」設定安全性原則。 若要深入了解「Azure 資訊安全中心」，請參閱下列主題：

* [Azure 資訊安全中心規劃和操作指南](security-center-planning-and-operations-guide.md) — 了解如何規劃及了解採用 Azure 資訊安全中心的設計考量。
* [Azure 資訊安全中心的安全性健全狀況監視](security-center-monitoring.md) — 了解如何監視 Azure 資源的健全狀況。
* [管理與回應 Azure 資訊安全中心的安全性警示](security-center-managing-and-responding-alerts.md) — 了解如何管理與回應安全性警示。
* [使用 Azure 資訊安全中心監視合作夥伴解決方案](security-center-partner-solutions.md) — 了解如何監視合作夥伴解決方案的健全狀況。
* [Azure 資訊安全中心常見問題集](security-center-faq.md) — 尋找有關使用服務的常見問題。
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/) — 尋找有關 Azure 安全性與相容性的部落格文章。
