---
title: 以太坊權威證明聯盟 - Azure
description: 使用以太坊權威證明聯盟解決方案，來部署和設定多成員的聯盟以太坊網路
services: azure-blockchain
keywords: ''
author: CodyBorn
ms.author: coborn
ms.date: 8/2/2018
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: brendal
manager: vamelech
ms.openlocfilehash: 12fab05ef7c459419c8aaf319098fddaae1416e6
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "48241243"
---
# <a name="ethereum-proof-of-authority-consortium"></a>以太坊權威證明聯盟

## <a name="overview"></a>概觀
[此解決方案](https://portal.azure.com/?pub_source=email&pub_status=success#create/microsoft-azure-blockchain.azure-blockchain-ethereumethereum-poa-consortium)的設計訴求是，讓您以最低限度的 Azure 和以太坊知識，更輕鬆地部署、設定和控管多成員的聯盟權威證明以太坊網路。

透過少數使用者輸入及 Azure 入口網站中的一鍵部署，每個成員都可以使用 Microsoft Azure 計算、網路和儲存體服務，在全球各地佈建網路應用。 每個成員的網路應用都會包含一組負載平衡的交易節點，讓應用程式或使用者可與其互動而提交以太坊交易。

## <a name="concepts"></a>概念

### <a name="terminology"></a>術語

-   **共識** - 透過區塊的驗證和建立在分散式網路中同步處理資料的動作。

-   **聯盟成員** - 在區塊鏈網路方面達成共識的實體。

-   **管理員** - 用來管理指定聯盟成員參與情形的以太坊帳戶。

-   **驗證器** - 與代表管理員參與共識的以太坊帳戶相關聯的機器。

### <a name="proof-of-authority"></a>權威證明

對於剛接觸區塊鏈社群的人們來說，若要在 Azure 上以簡單且易於設定的方式了解此技術，此版解決方案會是個很好的機會。 工作量證明是一種 Sybil 防禦機制，可利用計算成本自我管理網路，並達到公平參與的目的。 這非常適用於競相以加密貨幣提升網路安全性的匿名、開放式區塊鏈網路。 不過，在私人/聯盟網路中，基礎以太幣並無任何價值。 權威證明這個替代通訊協定更適合用於所有共識參與者皆具知名度且信譽良好的許可網路。 權威證明不需要進行採礦，不僅更有效率，且同時仍保有拜占庭容錯功能。

### <a name="consortium-governance"></a>聯盟控管

由於權威證明須依賴許可的網路授權清單來保持網路健全性，因此務必要提供合理的機制，用以修改此權限清單。 每個部署都隨附一組智慧合約和入口網站，用於此許可清單的鏈結控管。 當提議的變更通過聯盟成員的多數決議後，即會執行變更。 如此，即能夠以公開透明的方式加入新的共識參與者或移除不當參與者，而助長誠信的網路。

### <a name="admin-account"></a>管理帳戶

在權威證明節點的部署期間，系統會要求您提供管理員以太坊位址。 您可以使用數種不同的機制來產生和保護此以太坊帳戶。 在此位址新增為網路的授權後，您即可使用此帳戶來參與控管。 此管理員帳戶也會用來將共識參與此委派給在此部署的過程中建立的驗證器節點。 由於僅使用公用以太坊位址，因此每個管理員都會有足夠的彈性可運用符合其所需安全性模型的方式來保護其私密金鑰。

### <a name="validator-node"></a>驗證器節點

在權威證明通訊協定中，驗證器節點會取代傳統的礦工節點。 每個驗證器都有唯一的以太坊身分識別，可新增至智慧合約權限清單。 驗證器一旦列入此清單中，即可參與區塊建立程序。 若要深入了解此程序，請參閱同位檢查的 [Authority Round 共識](https://wiki.parity.io/Aura)相關文件。 每個聯盟成員可在五個區域間佈建兩個或更多驗證器節點，以達到異地備援的目的。 驗證器節點會與其他驗證器節點通訊，使基礎分散式總帳的狀態達到一致。
為了確保網路上的參與公平性，每個聯盟成員所使用的驗證器數目不可超過網路上的第一個成員 (如果第一個成員部署了三個驗證器，則每個成員最多只能有三個驗證器)。

### <a name="identity-store"></a>身分識別存放區

由於每個成員將會有多個同時執行的驗證器節點，且每個節點都必須具有許可的身分識別，因此驗證器必須能夠安全地取得在網路上具有唯一性的有效身分識別。 為此，我們建置了會在每個成員的訂用帳戶中部署的身分識別存放區，用以安全地保存已產生的以太坊身分識別。 在部署後，協調流程容器會為每個驗證器產生一個以太坊私密金鑰，並將其儲存在 Azure Key Vault 中。 同位檢查節點在啟動之前，會先租用未使用的身分識別，以確保該身分識別不會由其他節點所取用。 身分識別會提供給用戶端，讓它取得開始建立區塊的授權。 如果主控 VM 發生運作中斷的情形，身分識別租用即會釋出，讓替代節點可在未來繼續使用其身分識別。

### <a name="bootnode-registrar"></a>啟動節點註冊機構

為了簡化連線機制，每個成員都會在[資料 API 端點](#data-api)上裝載一組連線資訊。 這項資料包含啟動節點的清單，而這些節點可供加入的成員作為對等互連節點。 在此資料 API 中，我們會維護最新的啟動節點清單

### <a name="bring-your-own-operator"></a>自備操作員

聯盟成員通常會想要參與網路控管，但不想要運作及維護其基礎結構。 不同於傳統系統，由單一操作員負責處理整個網路，可能不利於非集中式的區塊鏈系統模型。 各個聯盟成員可以不要雇用集中式仲介機構來運作網路，而將基礎結構管理委派給他們所選擇的操作員。 如此，即可採行能夠讓每個成員選擇運作其本身的基礎結構或將作業委派給不同合作夥伴的混合式模型。 委派的作業工作流程的運作方式如下：

1.  **聯盟成員**產生以太坊位址 (包含私密金鑰)

2.  **聯盟成員**將公用以太坊位址提供給**操作員**

3.  **操作員**使用我們的 Azure Resource Manager 解決方案部署並設定 PoA 驗證器節點

4.  **操作員**將 RPC 和管理端點提供給**聯盟成員**

5.  **聯盟成員**使用其私密金鑰簽署要求，接受驗證器節點**運算子**已代表他們執行部署並參與

### <a name="azure-monitor"></a>Azure 監視器

此解決方案也隨附 Azure 監視器，用以追蹤節點和網路統計資料。 這可以讓應用程式開發人員檢視基礎區塊鏈，以追蹤區塊產生統計資料。 網路操作員可以使用 Azure 監視器，透過基礎結構的統計資料和可查詢的記錄快速偵測並防止網路中斷的狀況。 如需詳細資訊，請參閱[服務監視](#service-monitoring)。

### <a name="deployment-architecture"></a>部署架構

#### <a name="description"></a>說明

此解決方案可部署單一區域或多區域型的多成員以太坊聯盟網路。 根據預設，RPC 和對等互連端點可透過公用 IP 來存取，以簡化跨訂用帳戶和雲端的連線。 建議您利用[同位檢查的授權合約](https://wiki.parity.io/Permissioning)進行應用程式層級的存取控制。 我們也支援在 VPN 後方部署、利用 VNet 閘道進行跨訂用帳戶連線的網路。 這種部署方式較為複雜，因此建議您先從公用 IP 模型開始著手。

#### <a name="consortium-member-overview"></a>聯盟成員概觀

每個聯盟成員部署都包含：

-   執行 PoA 驗證器的虛擬機器

-   分散 RPC、對等互連和控管 DApp 要求的 Azure Load Balancer

-   保護驗證器身分識別的 Azure Key Vault

-   裝載持續性網路資訊和協調租用的 Azure 儲存體

-   彙總記錄和效能統計資料的 Azure 監視器

-   允許透過私人 VNet 進行 VPN 連線的 VNet 閘道 (選擇性)

![部署架構](./media/ethereum-poa-deployment/deployment-architecture.png)

我們利用 Docker 容器維持可靠性及進行模組化。 我們可以使用 Azure Container Registry 來裝載及提供已建立版本的映像，作為每個部署的一部分。 容器映像包含：

-   協調器

    -   在部署期間執行一次

    -   產生身分識別和控管合約

    -   將身分識別儲存在身分識別存放區中

-   同位檢查用戶端

    -   租用身分識別存放區中的身分識別

    -   探索並連線至對等節點

-   EthStats 代理程式

    -   透過 RPC 收集本機記錄和統計資料，並推送至 Azure 監視器

-   控管 DApp

    -   用來與控管合約互動的 Web 介面

## <a name="how-to-guides"></a>使用說明指南
### <a name="governance-dapp"></a>控管 DApp

權威證明的核心是非集中式控管。 控管 DApp 是一組預先部署的[智慧合約](https://github.com/Azure-Samples/blockchain/tree/master/ethereum-on-azure/)和 Web 應用程式，用來控管網路上的授權。
授權可細分為管理員身分識別和驗證器節點。
管理員有權將共識參與委派給一組驗證器節點。 管理員也可經由投票讓其他管理員加入或退出網路。

![控管 dapp](./media/ethereum-poa-deployment/governance-dapp.png)

-   **非集中式控管 -** 網路授權的變更可透過選定管理員的鏈結投票來管理。

-   **驗證器委派 -** 授權可以管理其在每個 PoA 部署中設定的驗證器節點。

-   **可稽核的變更歷程記錄 -** 每項變更都會記錄在透明且可稽核的區塊鏈上。

#### <a name="getting-started-with-governance"></a>控管入門
若要透過「控管 DApp」執行任何一種交易，您將必須利用乙太坊電子錢包。  最直接的方法是使用瀏覽器內電子錢包 (例如 [MetaMask](https://metamask.io))；不過，由於這些是部署在網路上的智慧合約，因此您也可以將您與「控管」合約的互動自動化。

安裝 MetaMask 之後，請在瀏覽器中瀏覽至「控管 DApp」。  您可以在部署電子郵件中或透過 Azure 入口網站在部署輸出中找到該 URL。  如果您未安裝瀏覽器內電子錢包，便無法執行執行任何動作；不過，您將仍然能夠讀取管理員狀態。  

#### <a name="becoming-an-admin"></a>成為管理員
如果您是第一個在網路上部署的成員，您就會自動成為管理員，而您的同位節點則會被列為「驗證器」。  如果您是加入網路的，則必須獲得現有管理員集的多數 (超過 50%) 票選，才能成為管理員。  如果您選擇不要成為管理員，您的節點將仍然會同步處理並驗證區塊鏈；不過，它們將不會參與區塊建立程序。 若要開始投票程序以成為管理員，請按一下 [Nominate] \(提名\) 並輸入您的乙太坊位址和別名。

![提名](./media/ethereum-poa-deployment/governance-dapp-nominate.png)

#### <a name="candidates"></a>候選
選取 [Candidates] \(候選\) 索引標籤時，會顯示一組目前的候選管理員。  在候選人獲得目前管理員的多數票選之後，該候選人就會升格為管理員。若要投票給某位候選人，請選取該資料列，然後按一下頂端的 [Vote in] \(投票選出\)。  如果您改變投票心意，可以選取該候選人，然後按一下 [Rescind vote] \(撤銷投票\)。

![候選](./media/ethereum-poa-deployment/governance-dapp-candidates.png)


#### <a name="admins"></a>管理員
[Admins] \(管理員\) 索引標籤會顯示一組目前的管理員，並可讓您進行投票。  當一位管理員失去超過 50% 的支持度時，系統就會移除他在網路上的管理員身分。  此管理員所擁有的所有驗證器節點都會失去驗證器狀態，而成為網路上的交易節點。  管理員可能因任何原因被移除；不過，須由聯盟事先就原則達成一致意見。

![管理員](./media/ethereum-poa-deployment/governance-dapp-admins.png)

#### <a name="validators"></a>驗證程式
選取左側功能表中的 [Validators] \(驗證器\) 索引標籤時，會顯示此執行個體的目前已部署同位節點及其目前狀態 (節點類型)。  請注意，每個聯盟成員在此清單中都有一組不同的驗證器，因為此檢視代表目前已部署的聯盟成員。  如果這是新部署的執行個體，而您尚未新增驗證器，則會向您顯示 [Add Validators] \(新增驗證器\) 選項。  選取此選項時，會自動選擇一組區域平衡的同位節點，並將它們指派給您的驗證器集。  如果您部署的節點超出允許的容量，剩餘的節點就會變成網路上的交易節點。

指派每個驗證器的位址時，會自動透過 Azure 中的[身分識別存放區](#identity-store)指派。  如果某個節點停止運作，它就會放棄其身分識別，讓您部署中的另一個節點取代它。  這可確保維持您共識參與度的高可用性。

![驗證程式](./media/ethereum-poa-deployment/governance-dapp-validators.png)

#### <a name="consortium-name"></a>聯盟名稱
所有管理員都可以更新顯示在頁面頂端的「聯盟名稱」。  選取左上方的齒輪圖示，即可更新「聯盟名稱」。

#### <a name="account-menu"></a>帳戶功能表
右上方是您的乙太坊帳戶別名和頭像。  如果您是管理員，便能夠更新您的別名。

![帳戶](./media/ethereum-poa-deployment/governance-dapp-account.png)

### <a name="deploy-ethereum-proof-of-authority"></a>部署以太坊權威證明

以下是多方部署流程的範例：

1.  三個成員分別使用 MetaMask 產生了以太坊帳戶

2.  *成員 A* 提供其以太坊公用位址，部署了以太坊 PoA

3.  *成員 A* 將聯盟 URL 提供給*成員 B* 和*成員 C*

4.  *成員 B* 和*成員 C* 提供其以太坊公用位址和*成員 A* 的聯盟 URL，部署了以太坊 PoA

5.  *成員 A* 將*成員 B* 票選為管理員

6.  *成員 A* 和*成員 B* 都將*成員 C* 票選為管理員

此程序需要可支援部署數個虛擬機器和受控磁碟的 Azure 訂用帳戶。 如有必要，請[建立免費的 Azure 帳戶](https://azure.microsoft.com/free/)以開始作業。

當訂用帳戶受到保護後，請移至 Azure 入口網站。 選取 [+] 和 [Marketplace (檢視全部)]，並搜尋以太坊 PoA 聯盟。

以下幾節將引導您逐步設定網路中第一個成員的配置。 部署流程分成下列五個步驟：基本概念、部署區域、網路大小和效能、以太坊設定、Azure 監視器。

#### <a name="basics"></a>基本概念

在 [基本概念] 下方，為任何部署指定標準參數值，例如訂用帳戶、資源群組及基本虛擬機器屬性。

每個參數的詳細描述如下：

參數名稱|說明|允許的值|預設值
---|---|---|---
建立新網路或加入現有網路？|建立新網路或加入已存在的聯盟網路|建立新的 加入現有的|建立新的
電子郵件地址 (選擇性)|您將會在部署完成收到電子郵件通知，內含部署的相關資訊。|有效的電子郵件地址|NA
VM 使用者名稱|每個所部署 VM 的管理員使用者名稱 (僅限英數字元)|1-64 個字元|NA
驗證類型|用來向虛擬機器驗證的方法。|密碼或 SSH 公開金鑰|密碼
密碼 (驗證類型 = 密碼)|每個所部署虛擬機器的管理員帳戶密碼。  密碼必須包含下列其中 3 項要求：1 個大寫字元、1 個小寫字元、1 個數字與 1 個特殊字元。 所有 VM 一開始都有相同的密碼，但您可以在佈建之後變更密碼。|12-72 個字元|NA
SSH 金鑰 (驗證類型 = 公開金鑰)|用於遠端登入的安全殼層金鑰。||NA
訂用帳戶|要對其部署聯盟網路的訂用帳戶||NA
資源群組|要對其部署聯盟網路的資源群組。||NA
位置|資源群組的 Azure 區域。||NA

範例部署如下所示：![基本刀鋒視窗](./media/ethereum-poa-deployment/basic-blade.png)

#### <a name="deployment-regions"></a>部署區域

接下來，在 [部署區域] 下方指定區域數目的輸入，以根據指定的區域數目部署聯盟網路和選取的 Azure 區域。 使用者最多可部署 5 個區域。 建議您選擇第一個區域，以符合 [基本] 區段中的資源群組位置。 針對開發或測試網路，建議讓每個成員使用單一區域。 針對生產環境，建議您部署於兩個或更多區域，以達到高可用性。

每個參數的詳細描述如下：

  參數名稱|說明|允許的值|預設值
  ---|---|---|---
  區域數目|要部署聯盟網路的區域數目|1、2、3、4、5|1
  第一個區域|要部署聯盟網路的第一個區域|所有允許的 Azure 區域|NA
  第二個區域|要部署聯盟網路的第二個區域 (只有在選取 2 作為區域數目時才會顯示)|所有允許的 Azure 區域|NA
  第三個區域|要部署聯盟網路的第三個區域 (只有在選取 3 作為區域數目時才會顯示)|所有允許的 Azure 區域|NA
  第四個區域|要部署聯盟網路的第四個區域 (只有在選取 4 作為區域數目時才會顯示)|所有允許的 Azure 區域|NA
  第五個區域|要部署聯盟網路的第五個區域 (只有在選取 5 作為區域數目時才會顯示)|所有允許的 Azure 區域|NA

範例部署如下所示：![部署區域](./media/ethereum-poa-deployment/deployment-regions.png)

#### <a name="network-size-and-performance"></a>網路大小和效能

接下來，在 [網路大小和效能] 下方指定聯盟網路大小的輸入，例如驗證器節點的數量和大小。
驗證器節點的儲存體大小將指定區塊鏈的潛在大小。 此值可在部署之後變更。

每個參數的詳細描述如下：

  參數名稱|說明|允許的值|預設值
  ---|---|---|---
  負載平衡的驗證器節點數目|要佈建為網路一部分的驗證器節點數目|2-15|2
  驗證器節點儲存體效能|支援每個已部署驗證器節點的受控磁碟類型。|標準 SSD 或進階|標準 SSD
  驗證器節點的虛擬機器大小|驗證器節點所使用的虛擬機器大小。|標準 A、標準 D、標準 D-v2、標準 F 系列、標準 DS 與標準 FS|標準 D1 v2

[儲存體定價詳細資料](https://azure.microsoft.com/pricing/details/managed-disks/)

[虛擬機器定價詳細資料](https://azure.microsoft.com/pricing/details/virtual-machines/windows/)

請注意，「虛擬機器」和「儲存層」會影響網路效能。  根據所需的成本效益，建議您使用下列 SKU：

  虛擬機器 SKU|儲存層|價格|Throughput|Latency
  ---|---|---|---|---
  F1|標準 SSD|低|低|高
  D2_v3|標準 SSD|中|中|中
  F16s|進階 SSD|高|高|低

範例部署如下所示：![網路大小和效能](./media/ethereum-poa-deployment/network-size-and-performance.png)

#### <a name="ethereum-settings"></a>以太坊設定

接下來，在 [以太坊設定] 下方指定以太坊的相關組態設定，例如網路識別碼和以太坊帳戶密碼或創世區塊。

每個參數的詳細描述如下：

  參數名稱|說明|允許的值|預設值
  ---|---|---|---
聯盟成員識別碼|與每個聯盟網路成員相關聯的識別碼，用來設定 IP 位址空間，以避免發生衝突。 如果使用私人網路，成員識別碼在相同網路的不同組織間應該是唯一的。  即使相同組織會部署到多個區域，仍然需要唯一成員識別碼。 請記下此參數值，因為您將必須與其他加入的成員共用此值，以確保沒有衝突。|0-255|NA
網路識別碼|為要部署的聯盟以太坊網路指定網路識別碼。  每個以太坊網路都有自己的網路識別碼，1 是公用網路的識別碼。|5 - 999,999,999|10101010
管理員以太坊位址|用來參與 PoA 控管的以太坊帳戶位址。  建議使用 MetaMask 來產生以太坊位址。|42 個以 0x 開頭的英數字元|NA
進階選項|以太坊設定的進階選項|啟用或停用|停用
公用 IP (進階選項 = 啟用)|將網路部署在 VNet 閘道後方，並移除對等互連存取權。 如果選取此選項，則所有成員都必須使用 VNet 閘道，連線才能相容。|公用 IP 私人 VNet|公用 IP
區塊燃料限值 (進階選項 = 啟用)|網路的開始區塊燃料限值|任何數值|50,000,00
區塊重新封印期 (秒)|當網路上沒有任何交易時，將建立空白區塊的頻率。 頻率越高，越快有定局，但會增加儲存成本。|任何數值|15
交易權限合約 (進階選項 = 啟用)|交易授權合約的位元組程式碼。 將智慧合約的部署和執行限定於許可的以太坊帳戶清單。|合約位元組程式碼|NA

範例部署如下所示：![以太坊設定](./media/ethereum-poa-deployment/ethereum-settings.png)

#### <a name="monitoring"></a>監視

[監視] 刀鋒視窗可讓您設定網路的 Log Analytics 資源。 監視代理程式會收集並顯示您網路中實用的計量和記錄，讓您能夠快速檢查網路健康情況或進行問題偵錯。

  參數名稱|說明|允許的值|預設值
  ---|---|---|---
監視|用以啟用「監視」的選項|啟用或停用|啟用
連線至現有的 Log Analytics|建立新的 Log Analytics 執行個體，或加入現有的執行個體|建立新的或加入現有的|新建
監視器位置 (連線至現有的 Log Analytics = 建立新的)|將部署新 Log Analytics 執行個體的區域|所有 Log Analytics 區域|NA
現有的 Log Analytics 工作區識別碼 (連線至現有的 Log Analytics = 加入現有的)|現有 Log Analytics 執行個體的工作區識別碼||NA
現有的 Log Analytics 主要金鑰 (連線至現有的 Log Analytics = 加入現有的)|用來連線至現有 Log Analytics 執行個體的主要金鑰||NA


範例部署如下所示：![Azure 監視器](./media/ethereum-poa-deployment/azure-monitor.png)

#### <a name="summary"></a>總結

按一下 [摘要] 刀鋒視窗以檢閱指定的輸入，並執行基本的預先部署驗證。 在部署之前，您可以下載範本和參數。

檢閱法律和隱私權條款，然後按一下 [購買] 以進行部署。 如果部署包含 VNet 閘道，則需要 45 到 50 分鐘才能完成部署。

#### <a name="post-deployment"></a>部署後

##### <a name="deployment-output"></a>部署輸出

部署完成後，您將可透過確認電子郵件或 Azure 入口網站來存取必要參數。 在這些參數中，您會看到：

-   以太坊 RPC 端點

-   控管儀表板 URL

-   Azure 監視器 URL

-   資料 URL

-   VNet 閘道資源識別碼 (選擇性)

##### <a name="confirmation-email"></a>確認電子郵件

如果您提供電子郵件地址 ([基本區段](#basics))，則會將含有部署輸出資訊的電子郵件傳送至該電子郵件地址。

![部署電子郵件](./media/ethereum-poa-deployment/deployment-email.png)

##### <a name="portal"></a>入口網站

當部署順利完成，且所有資源皆已佈建後，您將可在資源群組中檢視輸出參數。

1.  在入口網站中找出您的資源群組

2.  瀏覽至 [部署]

3.  選取名稱與您的資源群組相同的最上層部署。

4.  選取 [輸出]

### <a name="growing-the-consortium"></a>擴增聯盟

若要擴充您的聯盟，您必須先連接實體網路。
使用以公用 IP 為基礎的部署時，第一個步驟將可順暢執行。 如果部署於 VPN 後方，請參閱[連接 VNet 閘道](#connecting-vnet-gateways)一節，以在部署新成員的過程中執行網路連線。  在部署完成之後，請使用[控管 DApp](#governance-dapp) 來成為網路管理員。

#### <a name="new-member-deployment"></a>新成員部署

1.  讓加入的成員共用下列資訊。 這項資訊可從您的部署後電子郵件或入口網站部署輸出中取得。

    -  聯盟資料 URL

    -  您已部署的節點數目

    -  VNet 閘道資源識別碼 (如果使用 VPN)

2.  部署成員在部署其網路目前狀態時應使用[相同的解決方案](https://portal.azure.com/?pub_source=email&pub_status=success#create/microsoft-azure-blockchain.azure-blockchain-ethereumethereum-poa-consortium)，並留意下列事項：

    -  選取 [加入現有的]

    -  選擇與網路上的其餘成員相同的驗證器節點數目，以確保能正確呈現

    -  使用[先前的步驟](#step-1-add-the-new-admin)中所提供的相同以太坊位址

    -  在 [以太坊設定] 索引標籤上傳入提供的 [聯盟資料 URL]

    -  如果網路的其餘部分位於 VPN 後方，請在 [進階] 區段下方選取 [私人 VNet]

#### <a name="connecting-vnet-gateways"></a>連接 VNet 閘道

如果您已使用預設公用 IP 設定進行部署，則可以忽略此步驟。 在使用私人網路的情況下，會透過 VNet 閘道連線連接不同的成員。 在成員可以加入網路並查看交易流量之前，現有的成員必須在 VPN 閘道上執行最後的設定，以接受連線。 這表示加入成員的以太坊節點須在連線建立後才能執行。 建議在聯盟中您建立備援網路連線 (網格)，以降低單一失敗點的可能性。

新成員部署後，現有成員必須設定對新成員的 VNet 閘道連線，以完成雙向連線。 為了達到此目的，現有成員將需要：

1.  連線端成員的 VNet 閘道資源識別碼 (請參閱部署輸出)

2.  共用連線金鑰

現有成員必須執行下列 PowerShell 指令碼，以完成連線。 建議使用在入口網站中位於右側導覽列頂端的 Azure Cloud Shell。

![Cloud Shell](./media/ethereum-poa-deployment/cloud-shell.png)

```Powershell
$MyGatewayResourceId = "<EXISTING_MEMBER_RESOURCEID>"
$OtherGatewayResourceId = "<NEW_MEMBER_RESOURCEID]"
$ConnectionName = "Leader2Member"
$SharedKey = "<NEW_MEMBER_KEY>"

## $myGatewayResourceId tells me what subscription I am in, what ResourceGroup and the VNetGatewayName
$splitValue = $MyGatewayResourceId.Split('/')
$MySubscriptionid = $splitValue[2]
$MyResourceGroup = $splitValue[4]
$MyGatewayName = $splitValue[8]

## $otherGatewayResourceid tells me what the subscription and VNet GatewayName are
$OtherGatewayName = $OtherGatewayResourceId.Split('/')[8]
$Subscription=Select-AzureRmSubscription -SubscriptionId $MySubscriptionid

## create a PSVirtualNetworkGateway instance for the gateway I want to connect to
$OtherGateway=New-Object Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
$OtherGateway.Name = $OtherGatewayName
$OtherGateway.Id = $OtherGatewayResourceId
$OtherGateway.GatewayType = "Vpn"
$OtherGateway.VpnType = "RouteBased"

## get a PSVirtualNetworkGateway instance for my gateway
$MyGateway = Get-AzureRmVirtualNetworkGateway -Name $MyGatewayName -ResourceGroupName $MyResourceGroup

## create the connection
New-AzureRmVirtualNetworkGatewayConnection -Name $ConnectionName -ResourceGroupName $MyResourceGroup -VirtualNetworkGateway1 $MyGateway -VirtualNetworkGateway2 $OtherGateway -Location $MyGateway.Location -ConnectionType Vnet2Vnet -SharedKey $SharedKey -EnableBgp $True
```

### <a name="service-monitoring"></a>服務監視

您可以透過部署電子郵件中的連結，或在部署輸出中找出參數 \[OMS\_PORTAL\_URL\]，來找出您的 Azure 監視器入口網站。

入口網站會先顯示整體的網路統計資料和節點概觀。

![監視器類別](./media/ethereum-poa-deployment/monitor-categories.png)

選取 [節點概觀] 會將您導向至入口網站，以檢視每個節點的基礎結構統計資料。

![節點統計資料](./media/ethereum-poa-deployment/node-stats.png)

選取 [網路統計資料] 會引導您檢視以太坊網路統計資料。

![網路統計資料](./media/ethereum-poa-deployment/network-stats.png)

#### <a name="sample-log-analytics-queries"></a>範例 Log Analytics 查詢

在這些儀表板後方會有一組可查詢的原始記錄。 您可以使用這些原始記錄來自訂儀表板、調查失敗原因，或設定閾值警示。 以下提供一組可在記錄搜尋工具中執行的範例查詢：

##### <a name="lists-blocks-that-have-been-reported-by-more-than-one-validator-useful-to-help-find-chain-forks"></a>列出已有多個驗證器報告的區塊。 有助於尋找鏈結分支。

```sql
MinedBlock_CL
| summarize DistinctMiners = dcount(BlockMiner_s) by BlockNumber_d, BlockMiner_s
| where DistinctMiners > 1
```

##### <a name="get-average-peer-count-for-a-specified-validator-node-averaged-over-5-minute-buckets"></a>取得指定的驗證器節點在平均 5 分鐘的值區內平均的對等節點計數。

```sql
let PeerCountRegex = @"Syncing with peers: (\d+) active, (\d+) confirmed, (\d+)";
ParityLog_CL
| where Computer == "vl-devn3lgdm-reg1000001"
| project RawData, TimeGenerated
| where RawData matches regex PeerCountRegex
| extend ActivePeers = extract(PeerCountRegex, 1, RawData, typeof(int))
| summarize avg(ActivePeers) by bin(TimeGenerated, 5m)
```

### <a name="ssh-access"></a>SSH 存取

基於安全考量，網路群組安全性規則依預設會拒絕 SSH 連接埠存取。 若要存取 PoA 網路中的虛擬機器執行個體，您必須將此規則變更為 [允許]\"\"

1.  首先，從 Azure 入口網站進入已部署資源群組的 [概觀] 區段。

    ![SSH 概觀](./media/ethereum-poa-deployment/ssh-overview.png)

2.  選取您想要存取之 VM 區域的「網路安全性群組」

    ![SSH NSG](./media/ethereum-poa-deployment/ssh-nsg.png)

3.  選取 \"allow-ssh\" 規則

    ![ssh-allow](./media/ethereum-poa-deployment/ssh-allow.png)

4.  將 [動作]\"\" 變更為 [允許]

    ![SSH 啟用允許](./media/ethereum-poa-deployment/ssh-enable-allow.png)

5.  按一下 [儲存]\"\" (變更可能需要數分鐘才會套用)

現在，您可以透過 SSH 和您提供的管理員使用者名稱和密碼/SSH 金鑰，從遠端連線至驗證器節點的虛擬機器。
所要執行來存取第一個驗證器節點的 SSH 命令在範本部署輸出參數中會列為 'SSH\_TO\_FIRST\_VL\_NODE\_REGION1' (就範例部署而言為：ssh -p 4000 poaadmin\@leader4vb.eastus.cloudapp.azure.com)。 若要前往其他交易節點，則將通訊埠編號加上 1 (例如，第一個交易節點在連接埠 4000 上)。

如果您先前部署至多個區域，請將上述命令變更為該區域中的負載平衡器的 DNS 名稱或 IP 位址。 若要尋找其他區域的 DNS 名稱或 IP 位址，請尋找具有命名慣例 \*\*\*\*\*-lbpip-reg\# 的資源，並檢視其 DNS 名稱和 IP 位址屬性。

### <a name="azure-traffic-manager-load-balancing"></a>Azure 流量管理員負載平衡

Azure 流量管理員可跨不同區域的多個部署路由傳入的流量，藉以縮短停機時間及改善 PoA 網路的回應能力。 內建的健康情況檢查以及自動重新路由有助於確保 RPC 端點和控管 DApp 的高可用性。 如果您已部署至多個區域，並已做好生產準備，則可利用這項功能。

使用流量管理員可以：

-   透過自動容錯移轉改善 PoA 網路的可用性。

-   以最低的網路延遲將使用者路由至 Azure 位置，以加快網路回應速度。

如果您決定建立流量管理員設定檔，您可以使用設定檔的 DNS 名稱來存取網路。 當其他聯盟成員已新增至網路後，流量管理員也可用來在已部署的驗證器之間進行負載平衡。

#### <a name="creating-a-traffic-manager-profile"></a>建立流量管理員設定檔

在 Azure 入口網站中按一下 [建立資源]\"\" 按鈕後，搜尋並選取 [流量管理員設定檔]\"\"。

![搜尋 Azure 流量管理員](./media/ethereum-poa-deployment/traffic-manager-search.png)

為設定檔指定唯一名稱，並選取在 PoA 部署期間所建立的資源群組。 按一下 [建立] 按鈕以進行部署。

![建立流量管理員](./media/ethereum-poa-deployment/traffic-manager-create.png)

在部署完成後，選取資源群組中的執行個體。 用來存取流量管理員的 DNS 名稱可在 [概觀] 索引標籤中找到

![找出流量管理員 DNS](./media/ethereum-poa-deployment/traffic-manager-dns.png)

選取 [端點] 索引標籤，然後按一下 [新增] 按鈕。 為端點提供一個唯一的名稱。 將 [目標資源類型] 變更為 [公用 IP 位址]。 接著，取得第一個區域的負載平衡器所具備的公用 IP 位址。

![路由流量管理員](./media/ethereum-poa-deployment/traffic-manager-routing.png)

為已部署網路中的每個區域重複前述步驟。 端點處於\"已啟用\"狀態後，將會自動載入，且區域會依據流量管理員的 DNS 名稱達到平衡。 現在，您可以使用此 DNS 名稱來取代本文件其他步驟中的 \[CONSORTIUM\_DATA\_URL\] 參數。

### <a name="data-api"></a>資料 API

每個聯盟成員都會裝載其他成員連線至網路所需的資訊。 現有成員會在成員部署之前提供 [CONSORTIUM_DATA_URL]。 在部署時，加入的成員將會從 JSON 介面在下列端點擷取資訊：

`<CONSORTIUM_DATA_URL>/networkinfo`

回應會包含對加入的成員有用的資訊 (創世區塊、驗證器集合約 ABI、啟動節點)，以及對現有的成員有用的資訊 (驗證器位址)。 我們建議使用此標準來擴充跨雲端提供者的聯盟。 此 API 會傳回具有下列結構的 JSON 格式回應：
```json
{
  "$id": "",
  "type": "object",
  "definitions": {},
  "$schema": "http://json-schema.org/draft-07/schema#",
  "properties": {
    "majorVersion": {
      "$id": "/properties/majorVersion",
      "type": "integer",
      "title": "This schema’s major version",
      "default": 0,
      "examples": [
        0
      ]
    },
    "minorVersion": {
      "$id": "/properties/minorVersion",
      "type": "integer",
      "title": "This schema’s minor version",
      "default": 0,
      "examples": [
        0
      ]
    },
    "bootnodes": {
      "$id": "/properties/bootnodes",
      "type": "array",
      "items": {
        "$id": "/properties/bootnodes/items",
        "type": "string",
        "title": "This member’s bootnodes",
        "default": "",
        "examples": [
          "enode://a348586f0fb0516c19de75bf54ca930a08f1594b7202020810b72c5f8d90635189d72d8b96f306f08761d576836a6bfce112cfb6ae6a3330588260f79a3d0ecb@10.1.17.5:30300",
          "enode://2d8474289af0bb38e3600a7a481734b2ab19d4eaf719f698fe885fb239f5d33faf217a860b170e2763b67c2f18d91c41272de37ac67386f80d1de57a3d58ddf2@10.1.17.4:30300"
        ]
      }
    },
    "valSetContract": {
      "$id": "/properties/valSetContract",
      "type": "string",
      "title": "The ValidatorSet Contract Source",
      "default": "",
      "examples": [
        "pragma solidity 0.4.21;\n\nimport \"./SafeMath.sol\";\nimport \"./Utils.sol\";\n\ncontract ValidatorSet …"
      ]
    },
    "adminContract": {
      "$id": "/properties/adminContract",
      "type": "string",
      "title": "The AdminSet Contract Source",
      "default": "",
      "examples": [
        "pragma solidity 0.4.21;\nimport \"./SafeMath.sol\";\nimport \"./SimpleValidatorSet.sol\";\nimport \"./Admin.sol\";\n\ncontract AdminValidatorSet is SimpleValidatorSet { …"
      ]
    },
    "adminContractABI": {
      "$id": "/properties/adminContractABI",
      "type": "string",
      "title": "The Admin Contract ABI",
      "default": "",
      "examples": [
        "[{\"constant\":false,\"inputs\":[{\"name\":\"proposedAdminAddress\",\"type\":\"address\"},…"
      ]
    },
    "paritySpec": {
      "$id": "/properties/paritySpec",
      "type": "string",
      "title": "The Parity client spec file",
      "default": "",
      "examples": [
        "\n{\n \"name\": \"PoA\",\n \"engine\": {\n \"authorityRound\": {\n \"params\": {\n \"stepDuration\": \"2\",\n \"validators\" : {\n \"safeContract\": \"0x0000000000000000000000000000000000000006\"\n },\n \"gasLimitBoundDivisor\": \"0x400\",\n \"maximumExtraDataSize\": \"0x2A\",\n \"minGasLimit\": \"0x2FAF080\",\n \"networkID\" : \"0x9a2112\"\n }\n }\n },\n \"params\": {\n \"gasLimitBoundDivisor\": \"0x400\",\n \"maximumExtraDataSize\": \"0x2A\",\n \"minGasLimit\": \"0x2FAF080\",\n \"networkID\" : \"0x9a2112\",\n \"wasmActivationTransition\": \"0x0\"\n },\n \"genesis\": {\n \"seal\": {\n \"authorityRound\": {\n \"step\": \"0x0\",\n \"signature\": \"0x0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000\"\n }\n },\n \"difficulty\": \"0x20000\",\n \"gasLimit\": \"0x2FAF080\"\n },\n \"accounts\": {\n \"0x0000000000000000000000000000000000000001\": { \"balance\": \"1\", \"builtin\": { \"name\": \"ecrecover\", \"pricing\": { \"linear\": { \"base\": 3000, \"word\": 0 } } } },\n \"0x0000000000000000000000000000000000000002\": { \"balance\": \"1\", \"builtin\": { \"name\": \"sha256\", \"pricing\": { \"linear\": { \"base\": 60, \"word\": 12 } } } },\n \"0x0000000000000000000000000000000000000003\": { \"balance\": \"1\", \"builtin\": { \"name\": \"ripemd160\", \"pricing\": { \"linear\": { \"base\": 600, \"word\": 120 } } } },\n \"0x0000000000000000000000000000000000000004\": { \"balance\": \"1\", \"builtin\": { \"name\": \"identity\", \"pricing\": { \"linear\": { \"base\": 15, \"word\": 3 } } } },\n \"0x0000000000000000000000000000000000000006\": { \"balance\": \"0\", \"constructor\" : \"…\" }\n }\n}"
      ]
    },
    "errorMessage": {
      "$id": "/properties/errorMessage",
      "type": "string",
      "title": "Error message",
      "default": "",
      "examples": [
        ""
      ]
    },
    "addressList": {
      "$id": "/properties/addressList",
      "type": "object",
      "properties": {
        "addresses": {
          "$id": "/properties/addressList/properties/addresses",
          "type": "array",
          "items": {
            "$id": "/properties/addressList/properties/addresses/items",
            "type": "string",
            "title": "This member’s validator addresses",
            "default": "",
            "examples": [
              "0x00a3cff0dccc0ecb6ae0461045e0e467cff4805f",
              "0x009ce13a7b2532cbd89b2d28cecd75f7cc8c0727"
            ]
          }
        }
      }
    }
  }
}

```
## <a name="tutorials"></a>教學課程

### <a name="programmatically-interacting-with-a-smart-contract"></a>以程式設計方式與智慧合約互動

> [!WARNING]
> 切勿透過網路傳送以太坊私密金鑰！ 請確定每個交易都已先在本機簽署，再透過網路傳送已簽署的交易。

在下列範例中，我們會使用 *ethereumjs-wallet* 產生以太坊位址，使用 *ethereumjs-tx* 在本機簽署，然後使用 *web3* 將原始交易傳送至以太坊 RPC 端點。

我們在此範例中將使用下列簡單的 Hello-World 智慧合約：

```javascript
pragma solidity ^0.4.11;
contract postBox {
    string message;
    function postMsg(string text) public {
        message = text;
    }
    function getMsg() public view returns (string) {
        return message;
    }
}
```

此範例假設合約已部署。 您可以使用 *solc* 和 *web3* 以程式設計方式部署合約。 首先請安裝下列節點模組：
```
sudo npm install web3@0.20.2
sudo npm install ethereumjs-tx@1.3.6
sudo npm install ethereumjs-wallet@0.6.1
```
此 nodeJS 指令碼將執行下列作業：

-   建構原始交易：postMsg

-   使用產生的私密金鑰簽署交易

-   將已簽署的交易提交至以太坊網路

```javascript
var ethereumjs = require('ethereumjs-tx')
var wallet = require('ethereumjs-wallet')
var Web3 = require('web3')

// TODO Replace with your contract address
var address = "0xfe53559f5f7a77125039a993e8d5d9c2901edc58";
var abi = [{"constant": false,"inputs": [{"name": "text","type": "string"}],"name": "postMsg","outputs": [],"payable": false,"stateMutability": "nonpayable","type": "function"},{"constant": true,"inputs": [],"name": "getMsg","outputs": [{"name": "","type": "string"}],"payable": false,"stateMutability": "view","type": "function"}];

// Generate a new Ethereum account
var account = wallet.generate();
var accountAddress = account.getAddressString()
var privateKey = account.getPrivateKey();

// TODO Replace with your RPC endpoint
var web3 = new Web3(new Web3.providers.HttpProvider(
    "http://testzvdky-dns-reg1.eastus.cloudapp.azure.com:8545"));

// Get the current nonce of the account
web3.eth.getTransactionCount(accountAddress, function (err, nonce) {
   var data = web3.eth.contract(abi).at(address).postMsg.getData("Hello World");
   var rawTx = {
     nonce: nonce,
     gasPrice: '0x00',
     gasLimit: '0x2FAF080',
     to: address,
     value: '0x00',
     data: data
   }
   var tx = new ethereumjs(rawTx);

   tx.sign(privateKey);

   var raw = '0x' + tx.serialize().toString('hex');
   web3.eth.sendRawTransaction(raw, function (txErr, transactionHash) {
     console.log("TX Hash: " + transactionHash);
     console.log("Error: " + txErr);
   });
 });
```

### <a name="deploy-smart-contract-with-truffle"></a>使用 Truffle 來部署智慧合約

-   安裝必要的程式庫

```javascript
npm init

npm install truffle-hdwallet-provider --save
```
-   在 truffle.js 中，新增下列程式碼，透過提供助憶片語 (MetaMask / Settings / Reveal Seed Words)，將 MetaMask 帳戶解除鎖定並將 PoA 節點設定為進入點

```javascript
var HDWalletProvider = require("truffle-hdwallet-provider");

var rpc_endpoint = "XXXXXX";
var mnemonic = "twelve words you can find in metamask/settings/reveal seed words";

module.exports = {
  networks: {
    development: {
      host: "localhost",
      port: 8545,
      network_id: "*" // Match any network id
    },
    poa: {
      provider: new HDWalletProvider(mnemonic, rpc_endpoint),
      network_id: 3,
      gasPrice : 0
    }
  }
};

```

-   部署至 PoA 網路

```javascript
$ truffle migrate --network poa
```

### <a name="debug-smart-contract-with-truffle"></a>使用 Truffle 對智慧合約進行偵錯

Truffle 具有可用來對智慧合約進行偵錯的本機開發網路。 您可以在[這裡](http://truffleframework.com/tutorials/debugging-a-smart-contract)找到完整的教學課程。

### <a name="webassembly-wasm-support"></a>WebAssembly (WASM) 支援

在新部署的 PoA 網路上已為您啟用 WebAssembly 支援。 它可讓您以任何能夠 transpile 為 Web 組件的語言 (Rust、C、C++) 進行智慧合約開發。 如需詳細資訊，請參閱下列連結

-   WebAssembly 的同位檢查概觀 - <https://wiki.parity.io/WebAssembly-Home>

-   同位檢查技術的教學課程 - <https://github.com/paritytech/pwasm-tutorial>

## <a name="reference"></a>參考

### <a name="faq"></a>常見問題集

#### <a name="i-notice-there-are-many-transactions-on-the-network-that-i-didnt-send-where-are-these-coming-from"></a>我發現網路上有許多我未傳送的交易。 這些交易來自何處？

將[個人 API](https://web3js.readthedocs.io/en/1.0/web3-eth-personal.html) 解除鎖定並不安全。 Bot 會接聽已解除鎖定的以太坊帳戶，並嘗試清空資金。 Bot 會假設這些帳戶含有真實的以太幣，並嘗試率先竊取餘額。 請勿在網路上啟用個人 API。 您應手動使用 MetaMask 之類的電子錢包預先簽署交易，或依照[以程式設計方式與智慧合約互動](#programmatically-interacting-with-a-smart-contract)一節中的說明，以程式設計方式預先簽署交易。

#### <a name="how-to-ssh-onto-a-vm"></a>如何透過 SSH 連線至 VM？

基於安全考量，並不會顯示 SSH 連接埠。 請遵循[本指南以啟用 SSH 連接埠](#ssh-access)。

#### <a name="how-do-i-set-up-an-audit-member-or-transaction-nodes"></a>如何設定稽核成員或交易節點？

交易節點是一組與網路對等互連、但未參與共識的同位檢查用戶端。 這些節點仍可用來提交以太坊交易，以及讀取智慧合約狀態。
這非常適合作為讓網路上非權威聯盟成員得以進行稽核的機制。 若要使用此機制，請從「擴增聯盟」的步驟 2 開始執行即可。

#### <a name="why-are-metamask-transactions-taking-a-long-time"></a>MetaMask 交易為何十分耗時？

為了確保能夠以正確的順序接收交易，每個以太坊交易都會隨附累加的 nonce。 如果您將 MetaMask 中的帳戶用於不同的網路上，則必須重設 nonce 值。 請依序按一下設定圖示 (3 橫列) [設定]、[重設帳戶]。 交易歷程記錄將會清除，屆時您即可重新提交交易。

#### <a name="do-i-need-to-specify-gas-fee-in-metamask"></a>我是否需要在 MetaMask 中指定燃料費？

乙太幣在權威證明聯盟中派不上用場。 因此，在於 MetaMask 中提交交易時，無須指定燃料費。

#### <a name="what-should-i-do-if-my-deployment-fails-due-to-failure-to-provision-azure-oms"></a>如果因無法佈建 Azure OMS 而導致我的部署失敗，該怎麼辦？

監視是一個選擇性功能。 在某些因無法順利佈建「Azure 監視器」資源而導致部署失敗的罕見狀況下，您可以在沒有「Azure 監視器」的情況下重新部署。

#### <a name="are-public-ip-deployments-compatible-with-private-network-deployments"></a>公用 IP 部署與私人網路部署是否相容？

否，對等互連需要雙向通訊，因此整個網路必須全都是公用或私人的。

#### <a name="what-is-the-expected-transaction-throughput-of-proof-of-authority"></a>什麼是權威證明的預期交易輸送量？

交易輸送量高度倚賴交易和網路拓撲的類型。  我們已透過跨多區域部署的網路，使用簡單交易進行平均每秒 400 個交易的基準測試。

#### <a name="how-do-i-subscribe-to-smart-contract-events"></a>如何訂閱智慧合約事件？

「以太坊權威證明」現在支援 Web 通訊端。  請查看您的部署電子郵件或部署輸出，以找出 Web 通訊端 URL 和連接埠。

## <a name="next-steps"></a>後續步驟

使用[以太坊權威證明聯盟](https://portal.azure.com/?pub_source=email&pub_status=success#create/microsoft-azure-blockchain.azure-blockchain-ethereumethereum-poa-consortium)解決方案開始作業。
