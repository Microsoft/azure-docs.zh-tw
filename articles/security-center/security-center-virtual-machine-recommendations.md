---
title: Azure 資訊安全中心的虛擬機器建議 | Microsoft Docs
description: 本文件說明 Azure 資訊安全中心為了協助您保護虛擬機器和電腦以及 Web 應用程式和 App Service 環境所提供的建議。
services: security-center
documentationcenter: na
author: rkarlin
manager: MBaldwin
editor: ''
ms.assetid: f5ce7f62-7b12-4bc0-b418-4a2f9ec49ca1
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/18/2018
ms.author: rkarlin
ms.openlocfilehash: 7e73d6236f76c58307bb552aeee03bafe66addcc
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43132360"
---
# <a name="understand-azure-security-center-resource-recommendations"></a>了解 Azure 資訊安全中心資源建議


## <a name="recommendations"></a>建議
請參考下表，以利了解可用的計算和 SQL 服務和應用程式服務建議，以及每一項建議在套用後將產生的作用。

### <a name="computers"></a>電腦
| 建議 | 說明 |
| --- | --- |
| [啟用訂用帳戶的資料收集](security-center-enable-data-collection.md) |建議您為每個訂用帳戶和訂用帳戶中的所有虛擬機器 (VM) 開啟安全性原則中的 [資料收集]。 |
| [為 Azure 儲存體帳戶啟用加密](security-center-enable-encryption-for-storage-account.md) | 建議您為待用資料啟用「Azure 儲存體服務加密」。 「儲存體服務加密」(SSE) 會在資料被寫入 Azure 儲存體時加密資料，並於擷取資料之前將其解密。 SSE 目前僅適用於 Azure Blob 服務，可用於區塊 Blob、分頁 Blob 和附加 Blob。 若要深入了解，請參閱[待用資料的儲存體服務加密](../storage/common/storage-service-encryption.md)。</br>只有在 Resource Manager 儲存體帳戶上才支援 SSE。 目前不支援傳統儲存體帳戶。 若要了解傳統和 Resource Manager 部署模型，請參閱 [Azure 部署模型](../azure-classic-rm.md)。 |
| [修復安全性設定](security-center-remediate-os-vulnerabilities.md) |建議您讓作業系統組態符合建議的安全性設定規則，例如不允許儲存密碼。 |
| [套用系統更新](security-center-apply-system-updates.md) |建議您將遺漏的系統安全性與重大更新部署到 VM。 |
| [套用 Just-in-Time 網路存取控制](security-center-just-in-time.md) | 建議您套用 Just-in-Time 虛擬機器存取。 Just-in-Time 為預覽功能，由資訊安全中心的標準層提供。 若要深入了解資訊安全中心的定價層，請參閱[價格](security-center-pricing.md)。 |
| [在系統更新之後重新開機](security-center-apply-system-updates.md#reboot-after-system-updates) |建議您重新啟動 VM 以完成套用系統更新的程序。 |
| [安裝端點保護](security-center-install-endpoint-protection.md) |建議您將反惡意程式碼程式佈建到 VM (僅適用於 Windows VM)。 |
| [啟用 VM 代理程式](security-center-enable-vm-agent.md) |可讓您查看哪些 VM 需要「VM 代理程式」。 為了佈建修補程式掃描、基準掃描及反惡意程式碼程式，必須在 VM 上安裝「VM 代理程式」。 預設會為從 Azure Marketplace 部署的 VM 安裝「VM 代理程式」。 如需如何安裝 VM 代理程式的相關資訊，請參閱 [VM 代理程式和擴充功能 – 第 2 部分](http://azure.microsoft.com/blog/2014/04/15/vm-agent-and-extensions-part-2/) 。 |
| [套用磁碟加密](security-center-apply-disk-encryption.md) |建議您使用 Azure 磁碟加密來加密您的 VM 磁碟 (Windows 和 Linux VM)。 建議您的 VM 上的作業系統和資料磁碟區都進行加密。 |
| [更新作業系統版本](security-center-update-os-version.md) |建議您將雲端服務的作業系統 (OS) 版本更新為作業系統系列可用的最新版本。  若要深入了解雲端服務，請參閱 [雲端服務概觀](../cloud-services/cloud-services-choose-me.md)。 |
| [未安裝弱點評估](security-center-vulnerability-assessment-recommendations.md) |建議在 VM 上安裝弱點評估解決方案。 |
| [修復弱點](security-center-vulnerability-assessment-recommendations.md#review-the-recommendation) |可讓您查看 VM 上安裝的弱點評估解決方案所偵測到的系統和應用程式弱點。 |

### <a name="app-services"></a>應用程式服務
| 建議 | 說明 |
| --- | --- |
| App Service 應只可經由 HTTPS 存取 | 建議您限制只能透過 HTTPS 存取 App Service。 |
| Web 應用程式應停用 Web 通訊端| 建議您仔細檢閱如何在 Web 應用程式中使用 Web 通訊端。  Web 通訊端通訊協定易受不同類型的安全性威脅攻擊。 |
| 對 Web 應用程式使用自訂網域 | 建議您使用自訂網域來防止 Web 應用程式遭受常見攻擊，例如網路釣魚及其他 DNS 相關攻擊。 |
| 設定 Web 應用程式的 IP 限制 | 建議您定義允許存取應用程式的 IP 位址清單。  使用 IP 限制可防止 Web 應用程式遭受常見攻擊。 |
| 請勿允許所有 ('*') 資源來存取應用程式 | 建議您不要將 WEBSITE_LOAD_CERTIFICATES 參數設定為 ‘*’。將參數設定為 '*' 表示所有憑證都會載入 Web 應用程式的個人憑證存放區中。  這會導致最小權限的原則遭到濫用，原因是網站不可能需要在執行階段存取所有憑證。 |
| CORS 不應允許每項資源存取應用程式 | 建議您只允許必要網域與 Web 應用程式互動。 跨原始資源共用 (CORS) 不應允許所有網域存取 Web 應用程式。 |
| 為 Web 應用程式使用支援的最新 .NET Framework | 建議您使用最新的 .NET Framework 版本，以取得最新的安全性類別。 使用較舊的類別和類型會讓應用程式變得容易遭受攻擊。 |
| 為 Web 應用程式使用支援的最新 JAVA 版本 | 建議您使用最新的 Java 版本，以取得最新的安全性類別。 使用較舊的類別和類型會讓應用程式變得容易遭受攻擊。 |
| 為 Web 應用程式使用支援的最新 PHP 版本 | 建議您使用最新的 PHP 版本，以取得最新的安全性類別。 使用較舊的類別和類型會讓應用程式變得容易遭受攻擊。 |
| [新增 Web 應用程式防火牆](security-center-add-web-application-firewall.md) |建議您為 Web 端點部署「Web 應用程式防火牆」(WAF)。 系統會針對任何具有相關聯網路安全性群組 (包含開放輸入 Web 連接埠 (80,443)) 的公開 IP (執行個體層級 IP 或負載平衡 IP)，顯示 WAF 建議。</br></br>資訊安全中心建議您佈建 WAF，協助對抗以虛擬機器和 App Service 環境上的 Web 應用程式為目標的攻擊。 App Service 環境 (ASE) 是Azure App Service 的 [Premium](https://azure.microsoft.com/pricing/details/app-service/) 服務方案選項，可提供完全隔離和專用的環境，以便安全地執行 Azure App Service 應用程式。 若要深入了解 ASE，請參閱 [App Service 環境的文件](../app-service/environment/intro.md)。</br></br>您可以將這些應用程式加入現有的 WAF 部署，以保護資訊安全中心的多個 Web 應用程式。 |
| [完成應用程式保護](security-center-add-web-application-firewall.md#finalize-application-protection) |若要完成 WAF 組態，必須將流量重新路由至 WAF 設備。 遵循這項建議會完成必要的設定變更。 |
| 為 Web 應用程式使用支援的最新 Node.js 版本 | 建議您使用最新的 Node.js 版本，以取得最新的安全性類別。 使用較舊的類別和類型會讓應用程式變得容易遭受攻擊。 |
| CORS 不應允許每項資源存取函式應用程式 | 建議您只允許必要網域與 Web 應用程式互動。 跨原始資源共用 (CORS) 不應允許所有網域存取函式應用程式。 |
| 對函式應用程式使用自訂網域 | 建議您使用自訂網域來防止函式應用程式遭受常見攻擊，例如網路釣魚及其他 DNS 相關攻擊。 |
| 設定函式應用程式的 IP 限制 | 建議您定義允許存取應用程式的 IP 位址清單。 使用 IP 限制可防止函式應用程式遭受常見攻擊。 |
| 函式應用程式應只可經由 HTTPS 存取 | 建議您限制只能透過 HTTPS 存取函式應用程式。 |
| 函式應用程式的遠端偵錯應關閉 | 建議您關閉不再需要使用的函式應用程式偵錯功能。 遠端偵錯需要在函式應用程式上開啟輸入連接埠。 |
| 函數應用程式應停用 Web 通訊端 | 建議您仔細檢閱如何在函式應用程式中使用 Web 通訊端。 Web 通訊端通訊協定易受不同類型的安全性威脅攻擊。 |


## <a name="next-steps"></a>後續步驟
若要深入了解適用於其他 Azure 資源類型的建議，請參閱下列文章︰

* [在 Azure 資訊安全中心監視身分識別和存取](security-center-identity-access.md)
* [保護 Azure 資訊安全中心內的網路](security-center-network-recommendations.md)
* [保護 Azure 資訊安全中心內的 Azure SQL 服務](security-center-sql-service-recommendations.md)

如要深入了解資訊安全中心，請參閱下列主題：

* [保護 Azure 資訊安全中心內的機器和應用程式](security-center-virtual-machine-protection.md)
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
