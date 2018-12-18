---
title: Azure 資訊安全中心常見問題集 (FAQ) | Microsoft Docs
description: 這個常見問題集回答「Azure 資訊安全中心」的相關問題。
services: security-center
documentationcenter: na
author: rkarlin
manager: MBaldwin
editor: ''
ms.assetid: be2ab6d5-72a8-411f-878e-98dac21bc5cb
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/30/2018
ms.author: rkarlin
ms.openlocfilehash: b93ad6793bb8e431ba318772c780a06c792dcd9a
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43126755"
---
# <a name="azure-security-center-frequently-asked-questions-faq"></a>Azure 資訊安全中心常見問題集 (FAQ)
這個常見問題集回答「Azure 資訊安全中心」的相關問題，此資訊安全中心是一項針對 Microsoft Azure 資源的安全性提供更深入的洞悉和更佳控制的服務，可協助您預防、偵測及回應威脅。

> [!NOTE]
> 從 2017 年 6 月初開始，資訊安全中心會使用 Microsoft Monitoring Agent 來收集和儲存資料。 如需詳細資訊，請參閱 [Azure 資訊安全中心平台移轉](security-center-platform-migration.md)。 本文章中的資訊說明轉換至 Microsoft Monitoring Agent 後的資訊安全中心功能。
>
>

## <a name="general-questions"></a>一般問題
### <a name="what-is-azure-security-center"></a>什麼是 Azure 資訊安全中心？
Azure 資訊安全中心利用加強對 Azure 資源的能見度及安全性控制權，以預防、偵測並回應威脅。 它提供您訂用帳戶之間的整合式安全性監視和原則管理，協助您偵測可能會忽略的威脅，且適用於廣泛的安全性解決方案生態系統。

### <a name="how-do-i-get-azure-security-center"></a>我要如何取得 Azure 資訊安全中心？
「Azure 資訊安全中心」是藉由您的 Microsoft Azure 訂用帳戶啟用，並可從 [Azure 入口網站](https://azure.microsoft.com/features/azure-portal/)存取。 ([登入入口網站](https://portal.azure.com)，選取 [瀏覽]，然後捲動至 [資訊安全中心])。  

## <a name="billing"></a>計費
### <a name="how-does-billing-work-for-azure-security-center"></a>Azure 資訊安全中心如何計費？
資訊安全中心提供兩個層級：

「免費層」提供 Azure 資源安全性狀態的可見度、基本安全性原則、安全性建議以及與合作夥伴的安全性產品和服務的整合。

「標準層」加上了進階威脅偵測功能，包括威脅情報、行為分析、異常偵測、安全性事件，以及威脅歸因報告。 標準層的前 60 天免費。 如果您在超過 60 天後選擇繼續使用服務，服務會自動開始計費。  若要升級，請選取安全性原則中的[定價層](https://docs.microsoft.com/azure/security-center/security-center-pricing)。

## <a name="permissions"></a>權限
Azure 資訊安全中心會使用[角色型存取控制 (RBAC)](../role-based-access-control/role-assignments-portal.md)，以提供可在 Azure 中指派給使用者、群組與服務的[內建角色](../role-based-access-control/built-in-roles.md)。

資訊安全中心會評估資源的組態，以識別安全性問題與弱點。 在「資訊安全中心」中，當您獲指派為資源所屬的訂用帳戶或資源群組「擁有者」、「參與者」或「讀取者」角色時，您只會看到與資源相關的項目。

請參閱[Azure 資訊安全中心的權限](security-center-permissions.md)以深入了解角色與資訊安全中心允許的動作。

## <a name="data-collection-agents-and-workspaces"></a>資料收集、代理程式及工作區
資訊安全中心會從您的 Azure 虛擬機器 (VM) 和非 Azure 電腦收集資料，以監視安全性漏洞與威脅。 資料是使用 Microsoft Monitoring Agent 收集而得，收集的方式是讀取機器的各種安全性相關組態和事件記錄檔，並將資料複製到工作區進行分析。

### <a name="am-i-billed-for-log-analytics-on-the-workspaces-created-by-security-center"></a>資訊安全中心所建立工作區上的 Log Analytics 是否需要付費？
否。 資訊安全中心所建立的工作區雖然設定以每個節點的 Log Analytics 來計費，但實際上不會產生 Log Analytics 費用。 資訊安全中心的計費一律根據您的資訊安全中心的安全性原則，以及工作區安裝的解決方案：

- **免費層** – 資訊安全中心在預設工作區啟用 'SecurityCenterFree' 解決方案。 免費層不須付費。
- **標準層** – 資訊安全中心在預設工作區啟用 'Security' 解決方案。

如需詳細資訊，請參閱[資訊安全中心價格](https://azure.microsoft.com/pricing/details/security-center/)。 價格頁面反映了自 2017 年起 6 月起安全性資料儲存體及依比例計費的改變。

> [!NOTE]
> 資訊安全中心所建立工作區的 Log Analytics 定價層不會影響資訊安全中心的收費。
>
>

### <a name="what-qualifies-a-vm-for-automatic-provisioning-of-the-microsoft-monitoring-agent-installation"></a>安裝 Microsoft Monitoring Agent 自動佈建的 VM 需要什麼資格？
符合下列條件的 Windows 或 Linux IaaS 虛擬機器：

- 虛擬機器目前尚未安裝 Microsoft Monitoring Agent 擴充功能。
- 虛擬機器處於執行狀態。
- 已安裝 Windows 或 Linux 虛擬機器代理程式。
- 虛擬機器沒有作為 Web 應用程式防火牆或新一代防火牆等設備使用。

### <a name="can-i-delete-the-default-workspaces-created-by-security-center"></a>我可以刪除資訊安全中心所建立的預設工作區嗎？
**建議您不要刪除預設工作區。** 資訊安全中心會使用預設工作區儲存您虛擬機器傳來的安全性資料。  如果您刪除了工作區，資訊安全中心無法收集此資料，特定安全性建議及警示就無法使用。

若要復原，請將連線到刪除工作區之虛擬機器上的 Microsoft Monitoring Agent 移除。 資訊安全中心會重新安裝代理程式，並建立新的預設工作區。

### <a name="how-can-i-use-my-existing-log-analytics-workspace"></a>如何使用我現有的 Log Analytics 工作區？

您可以選取現有的 Log Analytics 工作區來儲存資訊安全中心收集的資料。 若要使用現有的 Log Analytics 工作區：

- 工作區必須與您選取的 Azure 訂用帳戶相關聯。
- 至少必須擁有讀取權限以存取工作區。

若要選取現有的 Log Analytics 工作區：

1. 在 [安全性原則 - 資料收集] 下，選取 [使用其他工作區]。

   ![使用其他工作區][5]

2. 從下拉式功能表中，選取要儲存收集資料的工作區。

   > [!NOTE]
   > 在下拉式功能表中，只會顯示您可存取，而且在您 Azure 訂用帳戶中的工作區。
   >
   >

3. 選取 [ **儲存**]。
4. 選取 [儲存] 之後，會詢問您是否要重新設定受監控的虛擬機器。

   - 如果您希望新的工作區設定**僅套用在新的虛擬機器**，請選取 [否]。 新的工作區設定只會套用在新安裝的代理程式，以及新探索到的未安裝 Microsoft Monitoring Agent 之虛擬機器。
   - 如果您希望新的工作區設定**套用在所有虛擬機器**，請選取 [是]。 此外，每個連線到資訊安全中心建立之工作區的虛擬機器會重新連線到新的目標工作區。

   > [!NOTE]
   > 如果您選取 [是]，您必須刪除資訊安全中心建立的工作區，直到所有虛擬機器已重新連線至新的目標工作區。 如果過早刪除工作區，這項作業將會失敗。
   >
   >

   - 選取 [取消] 以取消作業。

### <a name="what-if-the-microsoft-monitoring-agent-was-already-installed-as-an-extension-on-the-vm"></a>如果虛擬機器上 Microsoft Monitoring Agent 已經以擴充功能的形式安裝了呢？
資訊安全中心不會覆寫既存的使用者工作區連線。 資訊安全中心會將虛擬機器送出的資料儲存到已經連線的工作區。 資訊安全中心會更新擴充功能版本以包含虛擬機器的 Azure 資源識別碼，進而支援資訊安全中心的使用方式。

### <a name="what-if-i-had-a-microsoft-monitoring-agent-installed-on-the-machine-but-not-as-an-extension"></a>如果我已經在機器上安裝了 Microsoft Monitoring Agent，但不是以擴充功能的形式呢？
如果 Microsoft Monitoring Agent 已經直接安裝在虛擬機器上 (而非作為 Azure 擴充功能)，資訊安全中心不會再安裝 Microsoft Monitoring Agent，且資訊安全監控的功能會受到限制。

如需詳細資訊，請參閱下一節：[如果虛擬機器上已安裝 SCOM 或 OMS 直接代理程式，會發生什麼狀況？](#scomomsinstalled)

### 如果我的 VM 上已安裝 SCOM 或 OMS 直接代理程式，會發生什麼狀況？<a name="scomomsinstalled"></a>
資訊安全中心無法事先識別是否已安裝代理程式。  資訊安全中心會嘗試安裝 Microsoft Monitoring Agent 擴充功能，然後因為已安裝的代理程式而發生失敗。  此失敗可防止代理程式的連線設定覆寫其工作區，並避免建立多重主目錄。

> [!NOTE]
> 代理程式版本會更新為最新的 OMS 代理程式版本。  這也適用於 SCOM 使用者。
>
>

### <a name="what-is-the-impact-of-removing-these-extensions"></a>移除這些擴充功能會有什麼影響？
如果您移除了 Microsoft Monitoring 擴充功能，資訊安全中心會無法收集虛擬機器送出的安全性資料，且特定安全性建議及警示將無法使用。 24 小時內，資訊安全中心會判定虛擬機器缺少了擴充功能，並重新安裝。

### <a name="how-do-i-stop-the-automatic-agent-installation-and-workspace-creation"></a>我要如何避免自動安裝代理程式和建立工作區？
您可以在安全性原則中關閉訂用帳戶的自動佈建，但不建議您這麼做。 關閉自動佈建會限制資訊安全中心的建議和警示。 標準層上的訂用帳戶需要進行自動佈建。 若要停用自動佈建：

1. 如果您的訂用帳戶設定為標準層，請開啟該訂用帳戶的安全性原則，並選取**免費**層。

   ![定價層][1]

2. 接下來，請選取 [安全性原則 – 資料收集] 刀鋒視窗上的 [關閉]，來停用自動佈建。
   ![資料收集][2]

### <a name="should-i-opt-out-of-the-automatic-agent-installation-and-workspace-creation"></a>我應該要退出自動安裝代理程式和建立工作區嗎？

> [!NOTE]
> 如果您選擇退出自動佈建，請務必檢閱[退出的影響為何？](#what-are-the-implications-of-opting-out-of-automatic-provisioning)和[退出時的建議步驟](#what-are-the-recommended-steps-when-opting-out-of-automatic-provisioning)區段。
>
>

如果您符合下列條件，您可以選擇退出自動佈建：

- 資訊安全中心的自動代理程式安裝適用於整個訂用帳戶。  您無法將自動安裝套用到虛擬機器的子集。 如果有重要虛擬機器無法使用 Microsoft Monitoring Agent 安裝，您應該退出自動佈建。
- Microsoft Monitoring Agent 擴充功能的安裝程序會更新代理程式的版本。 這適用於直接代理程式和 SCOM 代理程式。 如果已安裝的 SCOM 代理程式是 2012 版並且已升級，那麼當 SCOM 伺服器也是 2012 版時，管理性功能可能會遺失。 如果已安裝的 SCOM 代理程式是 2012 版，您應考慮退出自動佈建。
- 如果您在訂用帳戶 (集中式工作區) 之外還有自訂工作區，則您應該退出自動佈建。 您可以手動安裝 Microsoft Monitoring Agent 擴充功能，然後將其連線至您的工作區，而不是由資訊安全中心覆寫連線。
- 如果您不想在每個訂用帳戶上建立多個工作區，而且訂用帳戶中有您自己的自訂工作區，則您有兩個選項：

   1. 您可以退出自動佈建。 移轉之後，依循[我可以如何使用現有 Log Analytics 工作區？](#how-can-i-use-my-existing-log-analytics-workspace)中的說明來設定預設工作區的設定
   2. 或者，您可以讓移轉程序完成，讓 Microsoft Monitoring Agent 安裝在虛擬機器上，然後讓虛擬機器連線至已建立的工作區。 接著，藉由選擇重新設定已安裝的代理程式，在設定預設工作區的設定上，選取您的自訂工作區。 如需詳細資訊，請參閱[如何使用我現有的 Log Analytics 工作區？](#how-can-i-use-my-existing-log-analytics-workspace)

### <a name="what-are-the-implications-of-opting-out-of-automatic-provisioning"></a>退出自動佈建有什麼影響？
移轉完成後，資訊安全中心會無法收集虛擬機器送出的安全性資料，且特定安全性建議及警示將無法使用。 如果您選擇退出，您應該手動安裝 Microsoft Monitoring Agent。 請參閱[退出時的建議步驟](#what-are-the-recommended-steps-when-opting-out-of-automatic-provisioning)。

### <a name="what-are-the-recommended-steps-when-opting-out-of-automatic-provisioning"></a>退出自動佈建時的建議步驟是什麼？
您應手動安裝 Microsoft Monitoring Agent，讓資訊安全中心可以收集您虛擬機器所傳來的安全性資料，並提供建議和警示。 請參閱[將 Windows 電腦連接到 Azure 中的 Log Analytics 服務](../log-analytics/log-analytics-windows-agent.md)，以取得安裝指南。

您可以將代理程式連線到任何現有的自訂工作區，或資訊安全中心建立的工作區。 如果自訂工作區並沒有啟用的 'Security' 或 'SecurityCenterFree' 解決方案，則您必須套用解決方案。 若要套用，請選取自訂工作區或訂用帳戶，並透過**安全性原則 - 定價層**刀鋒視窗來套用定價層。

   ![定價層][1]

資訊安全中心會根據選取的定價層，在工作區上啟用正確的解決方案。

### 我要如何移除資訊安全中心安裝的 OMS 擴充功能？<a name="remove-oms"></a>
您可以手動移除 Microsoft Monitoring Agent。 關閉資料收集會限制資訊安全中心的建議和警示，所以不建議您這麼做。

> [!NOTE]
> 如果已啟用資料收集，資訊安全中心將會在您移除代理程式後重新安裝程式。  您必須停用資料收集，再手動移除代理程式。 若要了解如何停用資料收集，請參閱[我要如何避免自動安裝代理程式和建立工作區？](#how-do-i-stop-the-automatic-agent-installation-and-workspace-creation?)
>
>

若要手動移除代理程式：

1.  在入口網站中，開啟**Log Analytics**。
2.  在 [Log Analytics] 刀鋒視窗中，選取工作區：
3.  選取您不想監控的虛擬機器，並選取 [中斷連線]。

   ![移除代理程式][3]

> [!NOTE]
> 如果 Linux VM 已經有非擴充功能的 OMS 代理程式，移除功能的同時也會移除代理程式，客戶必須要重新安裝。
>
>
### <a name="how-do-i-disable-data-collection"></a>我要如何停用資料收集？
預設會關閉自動佈建。 您可以在安全性原則中關閉這項設定，隨時停用資源的自動佈建。 強烈建議進行自動佈建，以便取得安全性警示，以及系統更新、作業系統弱點和端點保護的建議。

若要停用資料收集，請[登入 Azure 入口網站](https://portal.azure.com)，選取 [瀏覽]，選取 [資訊安全中心]，然後選取 [選取原則]。 選取您想要停用自動佈建的訂用帳戶。 當您選取訂用帳戶時，[安全性原則 - 資料收集] 隨即開啟。 在 [自動佈建] 之下，選取 [關閉]。

### <a name="how-do-i-enable-data-collection"></a>我要如何啟用資料收集？
您可以在安全性原則中為您的 Azure 訂用帳戶啟用資料收集。 啟用資料收集。 [登入 Azure 入口網站](https://portal.azure.com)，選取 [瀏覽]，選取 [資訊安全中心]，然後選取 [安全性原則]。 選取您想要啟用自動佈建的訂用帳戶。 當您選取訂用帳戶時，[安全性原則 - 資料收集] 隨即開啟。 在 [自動佈建] 之下，選取 [開啟]。

### <a name="what-happens-when-data-collection-is-enabled"></a>啟用資料收集時會發生什麼情況？
啟用自動佈建時，資訊安全中心會在所有支援的 Azure VM 和任何新建立的 VM 上佈建 Microsoft Monitoring Agent。 強烈建議使用自動佈建，但是手動代理程式安裝也可行。 [深入了解如何安裝 Microsoft Monitoring Agent 擴充功能](../log-analytics/log-analytics-quick-collect-azurevm.md#enable-the-log-analytics-vm-extension)。 

代理程式支援處理序建立事件 4688，及事件 4688 中的 *CommandLine* 欄位。 EventLog 會記錄在 VM 上建立的新處理序，並由資訊安全中心的偵測服務監視。 如需針對每個新處理序所記錄之詳細資料的資訊，請參閱 [4688 中的描述欄位](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4688#fields) \(英文\)。 代理程式也會收集在 VM 上建立的 4688 事件並將它們儲存在搜尋中。

代理程式也會啟用[自適性應用程式控制](security-center-adaptive-application.md)的資料收集，資訊安全中心會在稽核模式中設定本機 AppLocker 原則以允許所有應用程式。 這會導致 AppLocker 產生事件，然後由資訊安全中心收集並利用。 請務必注意，在已經設定 AppLocker 原則的機器上不會設定此原則。 

如果客戶提供[安全性連絡資訊](security-center-provide-security-contact-details.md)，當資訊安全中心在 VM 上偵測到可疑的活動時，客戶就會收到通知電子郵件。 資訊安全中心的安全性警示儀表板也會顯示警示。



### <a name="does-the-monitoring-agent-impact-the-performance-of-my-servers"></a>Monitoring Agent 是否會影響伺服器的效能？
代理程式只耗用少量的系統資源，對效能的影響應該很小。 如需有關效能影響和代理程式與擴充的詳細資訊，請參閱[規劃和操作指南](security-center-planning-and-operations-guide.md#data-collection-and-storage)。

### <a name="where-is-my-data-stored"></a>我的資料會儲存在何處？
從這個代理程式收集的資料會儲存在與您的訂用帳戶相關聯的現有 Log Analytics 工作區或新的工作區中。 如需詳細資訊，請參閱[資料安全性](security-center-data-security.md)。

## 現有的 Log Analytics 客戶<a name="existingloganalyticscust"></a>

### <a name="does-security-center-override-any-existing-connections-between-vms-and-workspaces"></a>資訊安全中心是否會覆寫任何虛擬機器及工作區之間現有的連線？
如果虛擬機器已經以擴充功能的形式安裝 Microsoft Monitoring Agent，資訊安全中心不會覆寫既存的工作區連線。 相反地，資訊安全中心會使用現有的工作區。

資訊安全中心解決方案若尚未安裝，此時會安裝至工作區，且此解決方案只會套用至相關的虛擬機器。 當您新增解決方案時，依預設會該解決方案會自動部署到與您 Log Analytics 工作區連線的所有 Windows 與 Linux 代理程式。 [解決方案目標設定](../operations-management-suite/operations-management-suite-solution-targeting.md)讓您能將範圍套用至解決方案。

如果 Microsoft Monitoring Agent 已經直接安裝在虛擬機器上 (而非作為 Azure 擴充功能)，資訊安全中心不會再安裝 Microsoft Monitoring Agent，且資訊安全監控的功能會受到限制。

### <a name="does-security-center-install-solutions-on-my-existing-log-analytics-workspaces-what-are-the-billing-implications"></a>資訊安全中心是否會在我現有的 Log Analytics 工作區上安裝解決方案？ 這會對計費造成什麼影響？
當資訊安全中心發現已經有虛擬機器連線到您建立的工作區時，資訊安全中心會根據您的定價層啟用此工作區上的解決方案。 這些解決方案會透過[方案目標設定](https://docs.microsoft.com/azure/operations-management-suite/operations-management-suite-solution-targeting)，只套用至相關的 Azure 虛擬機器，所以收費不會改變。

- **免費層** – 資訊安全中心在工作區安裝 SecurityCenterFree 解決方案。 免費層不須付費。
- **標準層** – 資訊安全中心在工作區安裝 'Security' 解決方案。

   ![預設工作區的解決方案][4]

### <a name="i-already-have-workspaces-in-my-environment-can-i-use-them-to-collect-security-data"></a>我的環境中已經有工作區，我是否能使用這些工作區來收集安全性資料？
如果虛擬機器已經以擴充套件的形式安裝 Microsoft Monitoring Agent，資訊安全中心會使用現有已連線的工作區。 資訊安全中心解決方案若尚未安裝，此時會安裝至工作區，且透過[解決方案目標設定](https://docs.microsoft.com/azure/operations-management-suite/operations-management-suite-solution-targeting)，此解決方案只會套用至相關的虛擬機器。

資訊安全中心在虛擬機器上安裝 Microsoft Monitoring Agent 時，會使用資訊安全中心建立的預設工作區。

### <a name="i-already-have-security-solution-on-my-workspaces-what-are-the-billing-implications"></a>我的工作區上已經有安全性解決方案。 這會對計費造成什麼影響？
Security & Audit 解決方案會用來啟用 Azure 虛擬機器上資訊安全中心標準層的功能。 如果工作區已經安裝有 Security & Audit 解決方案，資訊安全中心會使用現有的解決方案。 收費不會改變。

## <a name="using-azure-security-center"></a>使用 Azure 資訊安全中心
### <a name="what-is-a-security-policy"></a>什麼是安全性原則？
安全性原則可針對指定之訂用帳戶內的資源，定義一組建議的控制機制。 在「Azure 資訊安全中心」中，您可以根據公司的安全性需求，以及每個訂用帳戶中應用程式的類型或資料的機密性，為您的 Azure 訂用帳戶定義原則。

「Azure 資訊安全中心」中啟用的安全性原則將會決定安全性建議與監視。 若要深入了解安全性原則，請參閱 [Azure 資訊安全中心的安全性健康情況監視](security-center-monitoring.md)。

### <a name="who-can-modify-a-security-policy"></a>誰可以修改安全性原則？
若要修改安全性原則，您必須是安全性系統管理員或是訂用帳戶的擁有者或參與者。

若要了解如何設定安全性原則，請參閱[在 Azure 資訊安全中心設定安全性原則](security-center-policies.md)。

### <a name="what-is-a-security-recommendation"></a>什麼是安全性建議？
「Azure 資訊安全中心」會分析 Azure 資源的安全性狀態。 當發現潛在的安全性弱點時，就會產生相關建議。 這些建議會引導您完成設定所需控制項的程序。 範例包括：

* 佈建反惡意程式碼，以協助識別及移除惡意軟體
* [網路安全性群組](../virtual-network/security-overview.md)與規則，以控制對虛擬機器的流量
* 佈建 Web 應用程式防火牆，以協助抵禦以 Web 應用程式為目標的攻擊
* 部署遺漏的系統更新
* 處理不符合建議基準的作業系統組態

這裡只會顯示 [安全性原則] 中已啟用的建議。

### <a name="how-can-i-see-the-current-security-state-of-my-azure-resources"></a>我要如何查看 Azure 資源目前的安全性狀態？
[資訊安全中心概觀] 刀鋒視窗會根據電腦、網路、儲存體與資料，以及應用程式，來分類顯示環境的安全性狀態。 每個資源類型都有一個指標，顯示是否已識別出任何潛在的安全性弱點。 按一下每個圖格都會顯示資訊安全中心所識別的安全性問題，以及訂用帳戶內資源的詳細目錄。

### <a name="what-triggers-a-security-alert"></a>什麼會觸發安全性警示？
「Azure 資訊安全中心」會自動收集、分析及整合下列來源的記錄檔資料：Azure 資源、網路，以及反惡意程式碼和防火牆等合作夥伴解決方案。 偵測到威脅時，會建立安全性警示。 偵測範例包括：

* 已受到危害的虛擬機器與已知的惡意 IP 位址進行通訊
* 使用 Windows 錯誤報告偵測到進階的惡意程式碼
* 針對虛擬機器的暴力密碼破解攻擊
* 來自已整合的合作夥伴安全性解決方案 (例如「反惡意程式碼」或「Web 應用程式防火牆」) 的安全性警示

### <a name="whats-the-difference-between-threats-detected-and-alerted-on-by-microsoft-security-response-center-versus-azure-security-center"></a>在 Microsoft Security Response Center 與 Azure 資訊安全中心，偵測到威脅和收到警示有何差異？
Microsoft Security Response Center (MSRC) 執行 Azure 網路和基礎結構的選取安全性監視，並接收來自協力廠商的威脅情報和濫用客訴。 當 MSRC 察覺到有非法或未經授權的合作對象存取客戶資料，或有客戶不遵守可接受用途的條款使用 Azure，安全性事件管理員就會通知客戶。 通知方式通常是傳送電子郵件給 Azure 資訊安全中心中指定的安全性連絡人，如未指定安全性連絡人，則通知 Azure 訂用帳戶擁有者。

資訊安全中心是一項 Azure 服務，它會持續監視客戶的 Azure 環境，以及套用分析自動偵測廣泛的潛在惡意活動。 這些偵測會顯示為資訊安全中心儀表板中的安全性警示。

### <a name="which-azure-resources-are-monitored-by-azure-security-center"></a>Azure 資訊安全中心會監視哪些 Azure 資源？
Azure 資訊安全中心會監視下列 Azure 資源：

* 虛擬機器 (VM) (包括 [雲端服務](../cloud-services/cloud-services-choose-me.md))
* Azure 虛擬網路
* Azure SQL 服務
* Azure 儲存體帳戶
* Azure Web Apps (在 [App Service 環境](../app-service/environment/intro.md)中)
* 與您的 Azure 訂用帳戶整合的合作夥伴解決方案，例如 VM 和 App Service 環境上的 Web 應用程式防火牆

## <a name="virtual-machines"></a>虛擬機器
### <a name="what-types-of-virtual-machines-are-supported"></a>支援哪些類型的虛擬機器？
對於使用[傳統與 Resource Manager 部署模型](../azure-classic-rm.md)建立的虛擬機器 (VM)，提供監視和建議。

如需支援之平台的清單，請參閱 [Azure 資訊安全中心支援的平台](security-center-os-coverage.md)。

### <a name="why-doesnt-azure-security-center-recognize-the-antimalware-solution-running-on-my-azure-vm"></a>為什麼 Azure 資訊安全中心無法辨識我的 Azure VM 上所執行的反惡意程式碼軟體解決方案？
Azure 資訊安全中心能辨識透過 Azure 擴充功能安裝的反惡意程式碼軟體。 例如，資訊安全中心無法偵測您在提供的映像中預先安裝的反惡意程式碼軟體，或是您使用自己的程序 (例如組態管理系統) 在虛擬機器上安裝的反惡意程式碼軟體。

### <a name="why-do-i-get-the-message-missing-scan-data-for-my-vm"></a>為什麼我會從 VM 收到「遺失掃描資料」訊息？
當沒有 VM 的掃瞄資料時會顯示此訊息。 在 Azure 資訊安全中心中啟用資料收集之後，可能需要花費一些時間 (少於一小時) 才能填入掃描資料。 在初始填入掃描資料之後，您可能會收到此訊息，因為完全沒有掃描資料或是沒有最近的掃描資料。 掃描不會填入處於停止狀態之 VM 的資料。 如果最近沒有填入掃描資料 (根據 Windows 代理程式的保留原則，預設值為 30 天)，可能也會顯示此訊息。

### <a name="how-often-does-security-center-scan-for-operating-system-vulnerabilities-system-updates-and-endpoint-protection-issues"></a>資訊安全中心多久掃描作業系統弱點、系統更新和端點保護問題一次？
資訊安全中心掃描弱點、更新和問題的延遲為：

- 作業系統安全性設定：在 48 小時內更新資料
- 系統更新：在 24 小時內更新資料
- 端點保護問題：在 8 小時內更新資料

資訊安全中心通常會每小時掃描新的資料一次。 上述延遲值是沒有最新掃描或掃描失敗的最糟情況。

### <a name="why-do-i-get-the-message-vm-agent-is-missing"></a>為什麼我會收到「VM 代理程式已遺失」訊息？
VM 代理程式必須安裝在 VM 上，才能啟用資料收集。 預設會為從 Azure Marketplace 部署的 VM 安裝「VM 代理程式」。 如需如何在其他 VM 上安裝 VM 代理程式的資訊，請參閱部落格文章 [VM Agent and Extensions (VM 代理程式和擴充功能)](https://azure.microsoft.com/blog/vm-agent-and-extensions-part-2/)。


<!--Image references-->
[1]: ./media/security-center-platform-migration-faq/pricing-tier.png
[2]: ./media/security-center-platform-migration-faq/data-collection.png
[3]: ./media/security-center-platform-migration-faq/remove-the-agent.png
[4]: ./media/security-center-platform-migration-faq/solutions.png
[5]: ./media/security-center-platform-migration-faq/use-another-workspace.png
