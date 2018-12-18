---
title: 管理 Azure 資訊安全中心的安全性建議 | Microsoft Docs
description: 本文件將逐步引導您了解「Azure 資訊安全中心」的建議如何協助您保護 Azure 資源及遵守安全性原則。
services: security-center
documentationcenter: na
author: rkarlin
manager: MBaldwin
editor: ''
ms.assetid: 86c50c9f-eb6b-4d97-acb3-6d599c06133e
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/18/2018
ms.author: rkarlin
ms.openlocfilehash: 3af84e0659d1d62079c239ad6e650d31c9f9a6f1
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46125352"
---
# <a name="managing-security-recommendations-in-azure-security-center"></a>管理 Azure 資訊安全中心的安全性建議
本文件將逐步引導您了解如何使用「Azure 資訊安全中心」的建議來協助您保護 Azure 資源。

> [!NOTE]
> 本文件將使用範例部署來介紹服務。  本文件不是一份逐步解說指南。
>
>

## <a name="what-are-security-recommendations"></a>有哪些安全性建議？
資訊安全中心會定期分析 Azure 資源的安全性狀態。 當資訊安全中心識別潛在的安全性弱點時，它會建立建議。 這些建議會引導您完成設定所需控制項的程序。

## <a name="implementing-security-recommendations"></a>實作安全性建議
### <a name="set-recommendations"></a>設定建議
在 [在 Azure 資訊安全中心設定安全性原則](security-center-policies.md)中，您將了解如何：

* 設定安全性原則。
* 開啟資料收集。
* 選擇要在您的安全性原則中看見的建議。

目前的原則建議是以系統更新、基準規則、反惡意程式碼程式、子網路與網路介面上的[網路安全性群組](../virtual-network/security-overview.md)、SQL Database 稽核、SQL Database 透明資料加密及 Web 應用程式防火牆為中心。  [設定安全性原則](security-center-policies.md) 提供每個建議選項的描述。

### <a name="monitor-recommendations"></a>監視建議
設定安全性原則之後，「資訊安全中心」會分析您資源的安全性狀態，以識別潛在的弱點。 [概觀] 下方的 [建議] 圖格可讓您知道資訊安全中心識別出的建議總數量。

![建議圖格][1]

若要查看每個建議的詳細資訊，請選取 [概觀] 下方的 [建議] 圖格。 [建議] 隨即開啟。

![篩選建議][2]

您可以篩選建議。 若要篩選建議，請選取 [建議] 刀鋒視窗上的 [篩選]。 即會開啟 [篩選]  刀鋒視窗，您可以選取想要查看的嚴重性和狀態值。

系統會以表格格式顯示建議，其中每一行代表一個特定的建議。 這個表格的資料行包括：

* **描述**：說明建議及要如何處理它。
* **資源**：列出這個建議適用的資源。
* **狀態**：描述建議的目前狀態：
  * **未處理**：建議尚未處理。
  * **進行中**：正在將建議套用到資源，且您不需要採取任何動作。
  * **已解決**：建議已經完成 (在此情況下，該行會呈現灰色)。
* **嚴重性**：描述該特定建議的嚴重性：
  * **高**：某個有意義的資源 (例如應用程式、VM 或網路安全性群組) 有弱點存在，並且需要注意。
  * **中**：有弱點存在，需要非關鍵性步驟或其他步驟才能將其消除或完成程序。
  * **低**：應該處理但不需要立即注意的弱點存在。 (預設不會顯示嚴重性低的建議，但是如果您要查看它們，則可以篩選嚴重性低的建議。)

請使用下表做為參考，以協助您了解可用的建議，以及如果套用建議，每一個建議所產生的作用。

> [!NOTE]
> 您會想要了解 Azure 資源的 [傳統和 Resource Manager 部署模型](../azure-classic-rm.md) 。
>
>

| 建議 | 說明 |
| --- | --- |
| [啟用訂用帳戶的資料收集](security-center-enable-data-collection.md) |建議您為每個訂用帳戶以及所有 Azure 虛擬機器 (VM) 和非 Azure 電腦開啟安全性原則中的資料收集。 |
| [修復安全性設定](security-center-remediate-os-vulnerabilities.md) |建議您讓作業系統設定符合建議的安全性設定規則，例如不允許儲存密碼。 |
| [套用系統更新](security-center-apply-system-updates.md) |建議您將遺漏的系統安全性與重大更新部署到您的 Windows 與 Linux VM 和電腦。 |
| [套用 Just-in-Time 網路存取控制](security-center-just-in-time.md) | 建議您套用 Just-in-Time 虛擬機器存取。 Just-in-Time 功能由資訊安全中心的標準層提供。 若要深入了解資訊安全中心的定價層，請參閱[價格](security-center-pricing.md)。 |
| [在系統更新之後重新開機](security-center-apply-system-updates.md#reboot-after-system-updates) |建議您重新啟動 VM 以完成套用系統更新的程序。 |
| [新增 Web 應用程式防火牆](security-center-add-web-application-firewall.md) |建議您為 Web 端點部署「Web 應用程式防火牆」(WAF)。 系統會針對任何具有相關聯網路安全性群組 (包含開放輸入 Web 連接埠 (80,443)) 的公開 IP (執行個體層級 IP 或負載平衡 IP)，顯示 WAF 建議。 </br>資訊安全中心建議您佈建 WAF，協助對抗以虛擬機器和 App Service 環境上的 Web 應用程式為目標的攻擊。 App Service 環境 (ASE) 是Azure App Service 的 [Premium](https://azure.microsoft.com/pricing/details/app-service/) 服務方案選項，可提供完全隔離和專用的環境，以便安全地執行 Azure App Service 應用程式。 若要深入了解 ASE，請參閱 [App Service 環境的文件](../app-service/environment/intro.md)。</br>您可以將這些應用程式加入現有的 WAF 部署，以保護資訊安全中心的多個 Web 應用程式。 |
| [完成應用程式保護](security-center-add-web-application-firewall.md#finalize-application-protection) |若要完成 WAF 組態，必須將流量重新路由至 WAF 設備。 遵循這項建議會完成必要的設定變更。 |
| [新增新一代防火牆](security-center-add-next-generation-firewall.md) |建議您新增由 Microsoft 合作夥伴提供的新一代防火牆 (NGFW)，以提升您的安全防護。 |
| [僅透過 NGFW 路由傳送流量](security-center-add-next-generation-firewall.md#route-traffic-through-ngfw-only) |建議您設定網路安全性群組 (NSG) 規則，強制透過您的 NGFW 傳送內送流量到 VM。 |
| [安裝端點保護](security-center-install-endpoint-protection.md) |建議您將反惡意程式碼程式佈建到 VM (僅適用於 Windows VM)。 |
| [啟用子網路/虛擬機器上的網路安全性群組](security-center-enable-network-security-groups.md) |建議您在子網路或 VM 上啟用 NSG。 |
| [透過網際網路面向端點限制存取](security-center-restrict-access-through-internet-facing-endpoints.md) |建議您為 NSG 設定輸入流量規則。 |
| [在 SQL Server 上啟用稽核與威脅偵測](security-center-enable-auditing-on-sql-servers.md) |建議您針對 Azure SQL Server 開啟稽核與威脅偵測。 (僅限 Azure SQL 服務。 不包含在虛擬機器上執行的 SQ。) |
| [在 SQL 資料庫上啟用稽核與威脅偵測](security-center-enable-auditing-on-sql-databases.md) |建議您針對 Azure SQL Database 開啟稽核與威脅偵測。 (僅限 Azure SQL 服務。 不包含在虛擬機器上執行的 SQ。) |
| [在 SQL 資料庫上啟用透明資料加密](security-center-enable-transparent-data-encryption.md) |建議您針對 SQL Database 啟用加密功能。 (僅限 Azure SQL 服務。) |
| [啟用 VM 代理程式](security-center-enable-vm-agent.md) |可讓您查看哪些 VM 需要「VM 代理程式」。 為了佈建修補程式掃描、基準掃描及反惡意程式碼程式，必須在 VM 上安裝「VM 代理程式」。 預設會為從 Azure Marketplace 部署的 VM 安裝「VM 代理程式」。 如需如何安裝 VM 代理程式的相關資訊，請參閱 [VM 代理程式和擴充功能 – 第 2 部分](http://azure.microsoft.com/blog/2014/04/15/vm-agent-and-extensions-part-2/) 。 |
| [套用磁碟加密](security-center-apply-disk-encryption.md) |建議您使用 Azure 磁碟加密來加密您的 VM 磁碟 (Windows 和 Linux VM)。 建議您的 VM 上的作業系統和資料磁碟區都進行加密。 |
| [提供安全性連絡人詳細資料](security-center-provide-security-contact-details.md) |建議您提供每個訂用帳戶安全性連絡人資訊。 連絡人資訊為電子郵件地址和電話號碼。 如果我們的安全性小組發現您的資源遭到入侵，就會用這項資訊連絡您。 |
| [更新作業系統版本](security-center-update-os-version.md) |建議您將雲端服務的作業系統 (OS) 版本更新為作業系統系列可用的最新版本。  若要深入了解雲端服務，請參閱 [雲端服務概觀](../cloud-services/cloud-services-choose-me.md)。 |
| [未安裝弱點評估](security-center-vulnerability-assessment-recommendations.md) |建議在 VM 上安裝弱點評估解決方案。 |
| [修復弱點](security-center-vulnerability-assessment-recommendations.md#review-the-recommendation) |可讓您查看 VM 上安裝的弱點評估解決方案所偵測到的系統和應用程式弱點。 |
| [為 Azure 儲存體帳戶啟用加密](security-center-enable-encryption-for-storage-account.md) | 建議您為待用資料啟用「Azure 儲存體服務加密」。 「儲存體服務加密」(SSE) 會在資料被寫入 Azure 儲存體時加密資料，並於擷取資料之前將其解密。 SSE 目前僅適用於 Azure Blob 服務，可用於區塊 Blob、分頁 Blob 和附加 Blob。 若要深入了解，請參閱[待用資料的儲存體服務加密](../storage/common/storage-service-encryption.md)。</br>只有在 Resource Manager 儲存體帳戶上才支援 SSE。 |
| [啟用自適性應用程式控制](security-center-adaptive-application.md) | 建議您在 Windows VM 上套用自適性應用程式控制。 此功能由資訊安全中心的標準層提供。 若要深入了解資訊安全中心的定價層，請參閱[價格](security-center-pricing.md)。 |
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
| 為訂用帳戶指定多位擁有者 | 建議您指定多位訂用帳戶擁有者，以便擁有系統管理員存取備援。 |
| 為訂用帳戶指定最多 3 位擁有者 | 建議您指定少於 3 位的訂用帳戶擁有者，以降低已遭入侵擁有者造成缺口的可能性。 |
| 為您訂用帳戶上具有擁有者權限的帳戶啟用 MFA | 建議您為所有具備系統管理員權限的訂用帳戶啟用 Multi-Factor Authentication (MFA)，以避免出現帳戶或資源缺口。 |
| 為您訂用帳戶中具有寫入權限的帳戶啟用 MFA | 建議您為所有具備寫入權限的訂用帳戶啟用 Multi-Factor Authentication (MFA)，以避免出現帳戶或資源缺口。 |
| 為您訂用帳戶中具有讀取權限的帳戶啟用 MFA | 建議您為所有具備讀取權限的訂用帳戶啟用 Multi-Factor Authentication (MFA)，以避免出現帳戶或資源缺口。 |
| 移除您訂用帳戶中具有讀取權限的外部帳戶 | 建議您移除訂用帳戶中具有讀取權限的外部帳戶，以避免出現未受監視的存取。 |
| 移除您訂用帳戶中具有寫入權限的外部帳戶 | 建議您移除訂用帳戶中具有寫入權限的外部帳戶，以避免出現未受監視的存取。 |
| 移除您訂用帳戶中具有擁有者權限的外部帳戶 | 建議您移除訂用帳戶中具有擁有者權限的外部帳戶，以避免出現未受監視的存取。 |
| 移除訂用帳戶中已淘汰的帳戶 | 建議您移除訂用帳戶中已淘汰的帳戶。 |
| 從訂用帳戶中移除具有擁有者權限的已淘汰帳戶 | 建議您移除訂用帳戶中具有擁有者權限的已淘汰帳戶。 |

### <a name="apply-recommendations"></a>套用建議
檢視所有建議之後，請決定應該先套用哪一個建議。 建議您使用嚴重性分級做為主要參數，以評估應該先套用哪些建議。

在上述的建議表格中選取一項建議並逐步進行，做為套用建議的範例。

## <a name="next-steps"></a>後續步驟
在本文件中，已向您介紹「資訊安全中心」的安全性建議。 如要深入了解資訊安全中心，請參閱下列主題：

* [在 Azure 資訊安全中心設定安全性原則](security-center-policies.md) — 了解如何為您的 Azure 訂用帳戶及資源群組設定安全性原則。
* [Azure 資訊安全中心的安全性健全狀況監視](security-center-monitoring.md) — 了解如何監視 Azure 資源的健全狀況。
* [管理與回應 Azure 資訊安全中心的安全性警示](security-center-managing-and-responding-alerts.md) — 了解如何管理與回應安全性警示。
* [使用 Azure 資訊安全中心監視合作夥伴解決方案](security-center-partner-solutions.md) — 了解如何監視合作夥伴解決方案的健全狀況。
* [Azure 資訊安全中心常見問題集](security-center-faq.md) — 尋找有關使用服務的常見問題。
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/) — 尋找有關 Azure 安全性與相容性的部落格文章。

<!--Image references-->
[1]: ./media/security-center-recommendations/recommendations-tile.png
[2]: ./media/security-center-recommendations/filter-recommendations.png
