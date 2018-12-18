---
title: Log Analytics 資料安全性 | Microsoft Docs
description: 深入了解 Log Analytics 如何保護您的隱私權和保護您的資料安全。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: a33bb05d-b310-4f2c-8f76-f627e600c8e7
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/19/2018
ms.author: magoedte
ms.component: na
ms.openlocfilehash: b91b906db1307343a50ffc3be07d562091f2e335
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46978776"
---
# <a name="log-analytics-data-security"></a>Log Analytics 資料安全性
本文件旨在提供 Log Analytics (Azure 監視器的功能) 的特定資訊，以補充 [Azure 信任中心](../security/security-microsoft-trust-center.md)上的資訊。  

本文說明 Log Analytics 資料收集、處理和保全的方式。 您可以使用代理程式連接到 Web 服務、使用 System Center Operations Manager 來收集操作資料，或從 Azure 診斷擷取資料供 Log Analytics 使用。 

Log Analytics 服務會使用下列方法安全地管理您以雲端為基礎的資料：

* 資料隔離
* 資料保留
* 實體安全性
* 事件管理
* 法規遵循
* 安全性標準認證

如有任何問題、建議或關於下列任一項資訊的問題 (包括我們的安全性原則)，請與我們連絡：[Azure 支援選項](http://azure.microsoft.com/support/options/)。

## <a name="sending-data-securely-using-tls-12"></a>使用 TLS 1.2 安全地傳送資料 

為了確保資料傳送至 Log Analytics 時的安全性，我們強烈建議您將代理程式設定為至少使用傳輸層安全性 (TLS) 1.2。 我們已發現較舊版本的 TLS/安全通訊端層 (SSL) 較易受到攻擊，而且在其目前的運作中仍允許回溯相容性，因此並**不建議使用**這些版本，很快地，業界也會捨棄這些舊版通訊協定的支援。 

[PCI 安全標準委員會](https://www.pcisecuritystandards.org/)已設定 [2018 年 6 月 30 日作為最後期限](https://www.pcisecuritystandards.org/pdfs/PCI_SSC_Migrating_from_SSL_and_Early_TLS_Resource_Guide.pdf)，在此之後將停用舊版 TLS/SSL，並升級至更安全的通訊協定。 當 Azure 捨棄舊版支援後，如果您的代理程式無法透過 TLS 1.2 (至少) 進行通訊，就無法將資料傳送至 Log Analytics。 

除非有絕對必要，否則我們不建議將代理程式明確地設定為只使用 TLS 1.2，因為這樣可能會中斷平台層級的安全性功能，此功能可在更安全的較新通訊協定 (例如 TLS 1.3) 推出時，自動偵測並加以運用。 

### <a name="platform-specific-guidance"></a>平台專屬的指引

|平台/語言 | 支援 | 相關資訊 |
| --- | --- | --- |
|Linux | Linux 發行版本通常會依賴 [OpenSSL](https://www.openssl.org) 來取得 TLS 1.2 支援。  | 請檢查 [OpenSSL 變更記錄](https://www.openssl.org/news/changelog.html)來確認支援的 OpenSSL 版本。|
| Windows 8.0 - 10 | 支援，而且已預設為啟用。 | 請確認您仍在使用[預設設定](https://docs.microsoft.com/windows-server/security/tls/tls-registry-settings)。  |
| Windows Server 2012 - 2016 | 支援，而且已預設為啟用。 | 請確認您仍在使用[預設設定](https://docs.microsoft.com/windows-server/security/tls/tls-registry-settings) |
| Windows 7 SP1 和 Windows Server 2008 R2 SP1 | 支援，但預設為不啟用。 | 請參閱[傳輸層安全性 (TLS) 登錄設定](https://docs.microsoft.com/windows-server/security/tls/tls-registry-settings)頁面，了解詳細的啟用方式。  |
| Windows Server 2008 SP2 | TLS 1.2 支援需要更新。 | 請在 Windows Server 2008 SP2 中參閱[新增 TLS 1.2 支援的更新](https://support.microsoft.com/help/4019276/update-to-add-support-for-tls-1-1-and-tls-1-2-in-windows-server-2008-s)。 |

## <a name="data-segregation"></a>資料隔離
在 Log Analytics 服務內嵌您的資料之後，資料會以邏輯方式在整個服務的每個元件上分開。 每個工作區加上標記的所有資料。 這項標記作業在整個資料生命週期持續發生，它會強制執行服務的每個層級。 在您選取的區域中，您的資料會儲存在儲存體叢集中的專用資料庫。

## <a name="data-retention"></a>資料保留
系統會根據價格方案來儲存及保留已編製索引的記錄檔搜尋資料。 如需詳細資訊，請參閱 [Log Analytics 定價](https://azure.microsoft.com/pricing/details/log-analytics/)。

作為您[訂用帳戶合約](https://azure.microsoft.com/support/legal/subscription-agreement/)的一部分，Microsoft 會依據每個合約的條款保留您的資料。  刪除資料時，我們也會刪除資料所在的 Azure 儲存體帳戶。  移除客戶資料時，不會終結任何實體磁碟機。  

下表列出一些可用的解決方案，並提供它們所收集的資料類型範例。

| **方案** | **資料類型** |
| --- | --- |
| 容量和效能 |效能資料和中繼資料 |
| 更新管理 |中繼資料和狀態資料 |
| 記錄檔管理 |使用者定義的事件記錄檔、Windows 事件記錄檔和/或 IIS 記錄檔 |
| 變更追蹤 |軟體清查、Windows 服務和 Linux 精靈中繼資料，以及 Windows/Linux 檔案中繼資料 |
| SQL 和 Active Directory 評估 |WMI 資料、登錄資料、效能資料和 SQL Server 動態管理檢視結果 |

下表顯示資料類型範例：

| **資料類型** | **欄位** |
| --- | --- |
| 警示 |警示名稱、警示描述、BaseManagedEntityId、問題識別碼、IsMonitorAlert、RuleId、ResolutionState、優先順序、嚴重性、分類、擁有者、ResolvedBy、TimeRaised、TimeAdded、LastModified、LastModifiedBy、LastModifiedExceptRepeatCount、TimeResolved、TimeResolutionStateLastModified、TimeResolutionStateLastModifiedInDB、RepeatCount |
| 組態 |CustomerID、AgentID、EntityID、ManagedTypeID、ManagedTypePropertyID、CurrentValue、ChangeDate |
| Event |EventId、EventOriginalID、BaseManagedEntityInternalId、RuleId、PublisherId、PublisherName、FullNumber、Number、Category、ChannelLevel、LoggingComputer、EventData、EventParameters、TimeGenerated、TimeAdded <br>**注意：** 當您使用自訂欄位將事件寫入 Windows 事件記錄檔時，Log Analytics 會收集它們。 |
| 中繼資料 |BaseManagedEntityId、ObjectStatus、OrganizationalUnit、ActiveDirectoryObjectSid、PhysicalProcessors、NetworkName、IPAddress、ForestDNSName、NetbiosComputerName、VirtualMachineName、LastInventoryDate、HostServerNameIsVirtualMachine、IP 位址、NetbiosDomainName、LogicalProcessors、DNSName、DisplayName、DomainDnsName、ActiveDirectorySite、PrincipalName、OffsetInMinuteFromGreenwichTime |
| 效能 |ObjectName、CounterName、PerfmonInstanceName、PerformanceDataId、PerformanceSourceInternalID、SampleValue、TimeSampled、TimeAdded |
| State |StateChangeEventId、StateId、NewHealthState、OldHealthState、Context、TimeGenerated、TimeAdded、StateId2、BaseManagedEntityId、MonitorId、HealthState、LastModified、LastGreenAlertGenerated、DatabaseTimeModified |

## <a name="physical-security"></a>實體安全性
Log Analytics 服務是由 Microsoft 人員所管理，所有活動都有記錄並且可供稽核。 Log Analytics 會作為 Azure 服務操作，並符合所有 Azure 合規性與安全性需求。 您可以在 [Microsoft Azure 安全性概觀](http://download.microsoft.com/download/6/0/2/6028B1AE-4AEE-46CE-9187-641DA97FC1EE/Windows%20Azure%20Security%20Overview%20v1.01.pdf)的第 18 頁上檢視 Azure 資產之實體安全性的詳細資料。 不再負責管理 Log Analytics 服務的人員，其用來確保區域安全的實體存取權限 (包括傳輸和終止) 將會在一個工作天內變更。 若要了解我們使用的全域實體基礎結構，請參閱 [Microsoft 資料中心](https://azure.microsoft.com/global-infrastructure/)。

## <a name="incident-management"></a>事件管理
Log Analytics 具備所有 Microsoft 服務都會遵守的事件管理程序。 總結來說，我們：

* 使用共同責任模型，在此模型中，一部分的安全性責任歸屬 Microsoft，一部分則歸屬客戶
* 管理 Azure 安全性事件：
  * 在偵測到事件時啟動調查
  * 由待命的事件回應小組成員評估事件的影響和嚴重性。 根據證據，評估結果不一定會導致進一步上報給安全性回應小組。
  * 由安全性回應專家診斷事件，以進行技術或鑑識調查、找出圍堵、緩和及因應策略。 如果安全性小組認為客戶資料可能已公開給非法或未經授權的個人，則會開始並行執行客戶事件通知程序。  
  * 恢復穩定並從事件中復原。 事件回應小組會建立復原計畫來減輕問題。 隔離受影響系統之類的危機圍堵步驟可能會立即進行，與診斷並行。 度過立即的危險之後，可能會進行較長期的緩和措施。  
  * 將事件結案並進行事後檢討。 事件回應小組會建立概述事件細節的事後檢討，以便修改原則、程序和處理程序來防止事件再次發生。
* 通知客戶發生了安全性事件：
  * 確定受影響客戶的範圍，並盡可能為受影響當事人提供詳盡的通知
  * 建立通知，為客戶提供足夠詳盡的資訊，在不會過度延遲通知程序的情況下，讓他們能夠在他們那一端進行調查，並符合他們對其使用者所做的任何承諾。
  * 確認並視需要宣佈事件。
  * 在不會導致不合理延遲並符合法律或契約承諾的情況下，以事件通知告知客戶。 安全性事件的通知會以 Microsoft 選擇的任何方式 (包括透過電子郵件) 傳送給客戶的一或多位系統管理員。
* 進行小組準備及訓練：
  * Microsoft 人員必須完成安全性和認知訓練，這可幫助他們識別及報告可疑的安全性問題。  
  * 在 Microsoft Azure 服務工作的操作員身負圍繞在裝載客戶資料之機密系統存取權的附加訓練義務。
  * Microsoft 安全性回應人員獲得專門針對其角色的訓練

如果發生客戶資料遺失事件，我們會在一天內通知每位客戶。 不過，服務從未發生過客戶資料遺失。 

如需 Microsoft 如何回應安全性事件的詳細資訊，請參閱[雲端中的 Microsoft Azure 安全性回應](https://gallery.technet.microsoft.com/Azure-Security-Response-in-dd18c678/file/150826/4/Microsoft%20Azure%20Security%20Response%20in%20the%20cloud.pdf)。

## <a name="compliance"></a>法規遵循
Log Analytics 軟體開發和服務小組的資訊安全性及治理程式可支援其商務需求，並且會遵守 [Microsoft Azure 信任中心](https://azure.microsoft.com/support/trust-center/)和 [Microsoft 信任中心合規性所述的法律與法規](https://www.microsoft.com/en-us/trustcenter/compliance/default.aspx)。 上述位置也會描述 Log Analytics 建立安全性需求、識別安全性控制，以及管理和監視風險的方式。 每年我們都會檢閱原則、標準、程序和指導方針。

每個開發小組成員都會獲得正式的應用程式安全性訓練。 在內部，我們使用版本控制系統來開發軟體。 每個軟體專案都受到版本控制系統的保護。

Microsoft 備有會監看及評估 Microsoft 中所有服務的安全性和法規遵循小組。 資訊安全人員會組成小組，且他們與開發 Log Analytics 的工程小組並無關聯。 安全人員有他們自己的管理鏈，並且會獨立評估產品和服務，以確保安全性與法規遵循。

Microsoft 的董事會會收到有關 Microsoft 所有資訊安全程式的年度報表通知。

Log Analytics 軟體開發和服務小組會積極地與 Microsoft 法律和規範小組及其他產業合作夥伴合作，以取得各種認證。

## <a name="certifications-and-attestations"></a>認證和證明
Azure Log Analytics 符合下列需求︰

* [ISO/IEC 27001](http://www.iso.org/iso/home/standards/management-standards/iso27001.htm)
* [ISO/IEC 27018:2014](http://www.iso.org/iso/home/store/catalogue_tc/catalogue_detail.htm?csnumber=61498)
* [ISO 22301](https://azure.microsoft.com/blog/iso22301/)
* PCI 安全標準委員會的[支付卡產業 (PCI 相容) 資料安全標準 (PCI DSS)](https://www.microsoft.com/en-us/TrustCenter/Compliance/PCI)。
* [服務組織控制 (SOC) 1 類型 1 和 SOC 2 類型 1](https://www.microsoft.com/en-us/TrustCenter/Compliance/SOC1-and-2) 相容
* 擁有 HIPAA 商業夥伴合約之公司的 [HIPAA 和 HITECH](https://www.microsoft.com/en-us/TrustCenter/Compliance/hipaa)
* Windows 通用工程準則
* Microsoft 高可信度電腦運算 (英文)
* 作為 Azure 服務，Log Analytics 使用的元件會遵守 Azure 的合規性需求。 若要深入了解，請前往 [Microsoft 信任中心法規遵循](https://www.microsoft.com/en-us/trustcenter/compliance/default.aspx)。

> [!NOTE]
> 在某些認證/證明中，Log Analytics 會以舊名稱 *Operational Insights* 列出。
>
>

## <a name="cloud-computing-security-data-flow"></a>雲端運算安全性資料流程
下圖顯示的雲端安全性架構為您公司的資訊流程，以及當移至 Log Analytics 服務時如何受到保護，最後由您在 Azure 入口網站中看到。 圖表後面詳述每個步驟的詳細資訊。

![Log Analytics 資料收集與安全性的映像](./media/log-analytics-data-security/log-analytics-data-security-diagram.png)

## <a name="1-sign-up-for-log-analytics-and-collect-data"></a>1.註冊使用 Log Analytics 並收集資料
若貴組織要將資料傳送至 Log Analytics，您要設定 Azure 虛擬機器上執行的 Windows 或 Linux 代理程式，或是在您的環境或其他雲端提供者中的虛擬或實體電腦上。  如果您是使用 Operations Manager，則您要從管理群組設定 Operations Manager 代理程式。 使用者 (可能是您、其他個別使用者或一群人) 會建立一或多個 Log Analytics 工作區，並使用下列其中一個帳戶來註冊代理程式：

* [組織識別碼](../active-directory/fundamentals/sign-up-organization.md)
* [Microsoft 帳戶 - Outlook、Office Live、MSN](https://account.microsoft.com/account)

Log Analytics 工作區是收集、彙總、分析以及呈現資料的位置。 工作區主要是做為資料分割資料，每個工作區都是唯一的。 例如，您可能需要使用一個工作區管理實際執行資料，及使用另一個工作區管理測試資料。 工作區也會協助系統管理員控制資料的使用者存取。 每個工作區可以有多個與其關聯的使用者帳戶，每個使用者帳戶可以存取多個 Log Analytics 工作區。 您必須根據資料中心區域建立工作區。 每個工作區都會複寫到區域中的其他資料中心，這主要是為了確保 Log Analytics 服務可用性。

針對 Operations Manager，Operations Manager 管理群組就會建立與 Log Analytics 服務的連線。 接著，您要設定管理群組中哪些代理程式受控系統允許收集資料，並將資料傳送至服務。 根據已啟用的解決方案，這些解決方案中的資料會從 Operations Manager 管理伺服器直接傳送給 Log Analytics 服務，或者，因為代理程式受控系統上收集的資料量，而將這些解決方案中的資料從代理程式直接傳送給服務。 對於不受 Operations Manager 監視的系統，每個系統都會安全地直接連線到 Log Analytics 服務。

連線系統與 Log Analytics 服務之間的所有通訊都會加密。 會使用 TLS (HTTPS) 通訊協定來加密。  隨後會進行 Microsoft SDL 程序，使用最先進的密碼編譯通訊協定確保 Log Analytics 保持最新狀態。

會收集 Log Analytics 資料的每個類型代理程式。 所收集的資料類型取決於使用的解決方案類型。 若要查看資料集合摘要，請參閱[從方案庫新增 Log Analytics 方案](log-analytics-add-solutions.md)。 此外，大部分方案都會有更詳細的集合資訊。 解決方案是預先定義的檢視、記錄搜尋查詢、資料收集規則，以及處理邏輯的組合。 只有系統管理員可以使用 Log Analytics 來匯入方案。 在匯入解決方案之後，便會移到 Operations Manager 管理伺服器 (如果使用的話)，然後移至您所選擇的代理程式。 之後，代理程式會收集資料。

## <a name="2-send-data-from-agents"></a>2.從代理程式傳送資料
您使用註冊金鑰來註冊所有類型的代理程式，而代理程式與 Log Analytics 服務之間會使用憑證型驗證和 SSL 在連接埠 443 建立安全的連線。 Log Analytics 會使用秘密存放區來產生及維護金鑰。 私密金鑰每 90 天會輪替一次，其儲存在 Azure 中，並由遵守嚴格法規與相容性作法的 Azure 作業人員管理。

向 Log Analytics 工作區註冊的管理群組會透過 Operations Manager 來建立與 Operations Manager 管理伺服器的安全 HTTPS 連線。

對於 Azure 虛擬機器上執行的 Windows 或 Linux 代理程式，唯讀的儲存體金鑰可用來讀取 Azure 資料表中的診斷事件。  

透過向整合 Log Analytics 的 Operations Manager 管理群組報告的任何代理程式，如果管理伺服器無法因任何理由與服務通訊，所收集的資料就會本機儲存在管理伺服器上的暫時快取。   它們會在兩小時內，每隔 8 分鐘嘗試重新傳送資料。  對於略過管理伺服器並直接傳送至 Log Analytics 的資料，行為會與 Windows 代理程式是一致的。  

Windows 或管理伺服器代理程式的快取資料會受到作業系統的認證存放區保護。 如果服務在 2 個小時後無法處理資料，代理程式會將資料排入佇列。 如果佇列已滿，代理程式會開始卸除資料類型，最先卸除的是效能資料。 代理程式佇列限制為登錄機碼，因此您可以視需要加以修改。 收集的資料會加以壓縮，並傳送給服務，略過 Operations Manager 管理群組資料庫，因此不會對它們產生任何負載。 傳送收集的資料之後，會從快取中移除。

如上所述，管理伺服器或直接連線之代理程式中的資料會透過 SSL 傳送到 Microsoft Azure 資料中心。 (選擇性) 您可以使用 ExpressRoute 為資料提供額外的安全性。 ExpressRoute 可供直接從現有 WAN 網路 (例如網路服務提供者所提供的多重通訊協定標籤交換 (MPLS) VPN) 連線至 Azure。 如需詳細資訊，請參閱 [ExpressRoute](https://azure.microsoft.com/services/expressroute/)。

## <a name="3-the-log-analytics-service-receives-and-processes-data"></a>3.Log Analytics 服務接收和處理資料
Log Analytics 服務會確保內送資料是來自信任的來源，方法是驗證憑證和與 Azure 驗證的資料完整性。 接著，未經處理的資料會儲存在 Azure 事件中樞，在資料最終會待用儲存的區域中。 所儲存的資料類型取決於匯入和用來收集資料的解決方案類型。 然後，Log Analytics 服務會處理未經處理的資料，並將這些資料內嵌至資料庫內。

儲存在資料庫中已收集資料的保留期，會取決於所選的定價方案。 對於「免費」層，收集的資料可使用七天。 對於「付費」層，收集的資料根據預設可供使用 31 天，但可以延長為 730 天。 資料會以待用加密的形式儲存在 Azure 儲存體，以確保資料機密性，並使用本地備援儲存體 (LRS) 在本地區域內複寫資料。 過去兩週的資料也會儲存在以 SSD 為基礎的快取，而且此快取目前未加密。  我們目前正致力於支援加密以 SSD 為基礎的快取。      

## <a name="4-use-log-analytics-to-access-the-data"></a>4.使用 Log Analytics 來存取資料
如需存取 Log Analytics 工作區，請使用組織帳戶或您先前設定的 Microsoft 帳戶來登入 Azure 入口網站。 入口網站與 Log Analytics 服務之間的所有流量都會透過安全的 HTTPS 通道傳送。 在使用入口網站時，使用者用戶端 (網頁瀏覽器) 上會產生工作階段識別碼，且資料會儲存在本機快取中，直到工作階段終止為止。 終止時便會刪除快取。 未包含個人識別資訊的用戶端 Cookie 不會自動移除。 工作階段 Cookie 會標示為 HTTPOnly，並受到保護。 經過預先決定的閒置時間後，Azure 入口網站工作階段就會終止。

## <a name="next-steps"></a>後續步驟
* 請遵循 [Azure VM 快速入門](log-analytics-quick-collect-azurevm.md)，了解如何針對您的 Azure VM 使用 Log Analytics 收集資料。  

*  如果您需要在您的環境中，從實體或虛擬 Windows 或 Linux 電腦收集資料，請參閱 [Linux 電腦的快速入門](log-analytics-quick-collect-linux-computer.md)或 [Windows 電腦的快速入門](log-analytics-quick-collect-windows-computer.md)

