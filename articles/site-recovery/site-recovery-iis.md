---
title: 使用 Azure Site Recovery 複寫多層式 IIS 型 Web 應用程式 | Microsoft Docs
description: 了解如何使用 Azure Site Recovery 複寫 IIS Web 伺服陣列虛擬機器。
services: site-recovery
author: nsoneji
manager: gauravd
ms.service: site-recovery
ms.topic: article
ms.date: 07/06/2018
ms.author: nisoneji
ms.openlocfilehash: 830ced767a34302a635b9e685a2aee60c95fc81f
ms.sourcegitcommit: a06c4177068aafc8387ddcd54e3071099faf659d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/09/2018
ms.locfileid: "37920843"
---
# <a name="replicate-a-multi-tier-iis-based-web-application"></a>複寫多層式 IIS 型 Web 應用程式

應用程式軟體是組織中商務產能的引擎。 各種 web 應用程式可在組織提供不同的用途。 對組織而言，有些應用程式 (例如用於薪資處理的應用程式、財務應用程式及客戶面向的網站) 可能極為重要。 若要避免降低生產力，組織務必能夠持續啟動並執行這些應用程式。 更重要的是，讓這些應用程式一直處於可用狀態，有助於防止組織的品牌或形象受損。

重要的 Web 應用程式通常會設定為多層式應用程式：Web、資料庫及應用程式分別位於不同層上。 除了會分散於各層外，應用程式也可在每一層中使用多部伺服器來平衡流量負載。 此外，各層之間和 Web 伺服器上的對應可能會以靜態 IP 位址為基礎。 在容錯移轉時，需要更新其中一些對應，尤其是如果在 Web 伺服器上設定了多個網站。 如果 Web 應用程式使用 SSL，您就必須更新憑證繫結。

不是以複寫為基礎的傳統復原方法，涉及備份各種組態檔、登錄設定、繫結、自訂元件 (COM 或 .NET)、內容和憑證。 檔案會透過一組手動步驟來復原。 備份和手動復原檔案的傳統修復方法都很麻煩、容易發生錯誤，且不可調整。 例如，您可能會輕易忘記要備份憑證。 在容錯移轉之後，除了為伺服器購買新的憑證以外，您就沒有任何選擇。

理想的災害復原解決方案支援針對複雜的應用程式架構建立復原方案模型。 您也應該能夠將自訂的步驟新增至復原方案，以處理各層之間的應用程式對應。 如果發生災害，應用程式對應可提供單鍵、準確命中的解決方案，進而導致 RTO 減少。

本文說明如何使用 [Azure Site Recovery](site-recovery-overview.md) 來保護以 Internet Information Services (IIS) 為基礎的 Web 應用程式。 本文涵蓋將三層式 IIS 型 Web 應用程式複寫至 Azure 的最佳做法、如何進行災害復原演練，以及如何將應用程式容錯移轉至 Azure。

## <a name="prerequisites"></a>先決條件

在開始之前，請確定您了解如何執行下列工作：

* [將虛擬機器複寫至 Azure](vmware-azure-tutorial.md)
* [設計復原網路](site-recovery-network-design.md)
* [執行測試容錯移轉至 Azure](site-recovery-test-failover-to-azure.md)
* [執行容錯移轉到 Azure](site-recovery-failover.md)
* [複寫網域控制站](site-recovery-active-directory.md)
* [複寫 SQL Server](site-recovery-sql.md)

## <a name="deployment-patterns"></a>部署模式
IIS 型 Web 應用程式通常會遵循下列其中一個部署模式︰

**部署模式 1**

具有應用程式要求路由 (ARR)、IIS 伺服器及 SQL Server 的 IIS 型 Web 伺服陣列。

![有三個層級的 IIS 型 Web 伺服陣列圖表](./media/site-recovery-iis/deployment-pattern1.png)

**部署模式 2**

具有 ARR、IIS 伺服器、應用程式伺服器及 SQL Server 的 IIS 型 Web 伺服陣列。

![有四個層級的 IIS 型 Web 伺服陣列圖表](./media/site-recovery-iis/deployment-pattern2.png)

## <a name="site-recovery-support"></a>Site Recovery 支援

為了在本文中提供範例，我們在 Windows Server 2012 R2 Enterprise 上使用 VMware 虛擬機器搭配 IIS 7.5。 因為 Site Recovery 複寫並非應用程式特有，所以本文中的建議應套用於下表中所列的案例，以及針對其他版本的 IIS 套用。

### <a name="source-and-target"></a>來源與目標

案例 | 至次要網站 | 至 Azure
--- | --- | ---
Hyper-V | yes | yes
VMware | yes | yes
實體伺服器 | 否 | yes
Azure|NA|yes

## <a name="replicate-virtual-machines"></a>複寫虛擬機器

若要開始將所有 IIS Web 伺服陣列虛擬機器複寫至 Azure，請遵循[在 Site Recovery 中測試容錯移轉到 Azure](site-recovery-test-failover-to-azure.md)中的指引。

如果您使用靜態 IP 位址，您可以指定您想要虛擬機器使用的 IP 位址。 若要設定 IP 位址，請移至 [計算和網路] 設定 > [目標 IP]。

![顯示如何在 Site Recovery 計算和網路窗格中設定目標 IP 的螢幕擷取畫面](./media/site-recovery-active-directory/dns-target-ip.png)

## <a name="create-a-recovery-plan"></a>建立復原計畫
復原方案支援在容錯移轉期間對多層式應用程式中的各層進行排序。 排序有助於維持應用程式的一致性。 當您為多層式 Web 應用程式建立復原方案時，請完成[使用 Site Recovery 建立復原方案](site-recovery-create-recovery-plans.md)中所述的步驟。

### <a name="add-virtual-machines-to-failover-groups"></a>將虛擬機器新增至容錯移轉群組
典型的多層式 IIS Web 應用程式包含下列元件：
* 具有 SQL 虛擬機器的資料庫層。
* Web 層，其中包含 IIS 伺服器和應用程式層。 

根據層級將虛擬機器新增至不同的群組：

1. 建立復原計畫。 在群組 1 之下新增資料庫層虛擬機器。 這可確保資料庫層虛擬機器最後關閉且最先啟動。
1. 在群組 2 之下新增應用程式層虛擬機器。 這可確保應用程式層虛擬機器在資料庫層啟動之後啟動。
1. 在群組 3 中新增 Web 層虛擬機器。 這可確保 Web 層虛擬機器在應用程式層啟動之後啟動。
1. 在群組 4 中新增負載平衡虛擬機器。 這可確保負載平衡虛擬機器在 Web 層啟動之後啟動。

如需詳細資訊，請參閱[自訂復原方案](site-recovery-runbook-automation.md#customize-the-recovery-plan)。


### <a name="add-a-script-to-the-recovery-plan"></a>將指令碼新增至復原方案
為了讓 IIS Web 伺服陣列可正常運作，您可能需要在容錯移轉後或測試容錯移轉期間，於 Azure 虛擬機器上執行某些作業。 您可以自動化某些容錯移轉後的作業。 例如，將對應的指令碼新增至復原方案，即可更新 DNS 項目、變更網站繫結，或變更連接字串。 [將 VMM 指令碼新增至復原方案](site-recovery-how-to-add-vmmscript.md)說明如何使用指令碼來設定自動化工作。

#### <a name="dns-update"></a>DNS 更新
如果是已設定動態 DNS 更新的 DNS，則虛擬機器通常會在啟動之後使用新的 IP 位址更新 DNS。 如果您想要新增一個明確的步驟來以虛擬機器的新 IP 位址更新 DNS，請新增[指令碼來更新 DNS 中的 IP](https://aka.ms/asr-dns-update)，作為復原方案群組上的容錯移轉後置動作。  

#### <a name="connection-string-in-an-applications-webconfig"></a>在應用程式 web.config 中的連接字串
連接字串會指定與網站進行通訊的資料庫。 如果連接字串含有資料庫虛擬機器的名稱，則容錯移轉後無須採取任何進一步的步驟。 應用程式可以自動與資料庫進行通訊。 此外，如果保留資料庫虛擬機器的 IP 位址，則不需要更新連接字串。 

如果連接字串使用 IP 位址來參考資料庫虛擬機器，則在容錯移轉後就必須更新連接字串。 例如，以下連接字串會指向 IP 位址為 127.0.1.2 的資料庫：

        <?xml version="1.0" encoding="utf-8"?>
        <configuration>
        <connectionStrings>
        <add name="ConnStringDb1" connectionString="Data Source= 127.0.1.2\SqlExpress; Initial Catalog=TestDB1;Integrated Security=False;" />
        </connectionStrings>
        </configuration>

若要更新 Web 層中的連接字串，請在復原方案中的群組 3 之後新增 [IIS 連線更新指令碼](https://aka.ms/asr-update-webtier-script-classic)。

#### <a name="site-bindings-for-the-application"></a>應用程式的網站繫結
每個網站都包含繫結資訊。 繫結資訊包括繫結的類型、IIS 伺服器接聽網站要求的 IP 位址、連接埠號碼，以及網站的主機名稱。 在容錯移轉期間，如果與這些繫結相關聯的 IP 位址發生變更，您可能需要更新這些繫結。

> [!NOTE]
>
> 如果您將網站繫結設定為 [全未指派]，則在容錯移轉後，就不需要更新此繫結。 此外，如果與網站相關聯的 IP 位址並未在容錯移轉後變更，您就不需要更新網站繫結。 (IP 位址的保留取決於網路架構以及指派給主要和復原網站的子網路。 對您的組織而言，更新它們可能不可行。)

![示範如何設定 SSL 繫結的螢幕擷取畫面](./media/site-recovery-iis/sslbinding.png)

如果您將 IP 位址與網站相關聯，請使用新的 IP 位址更新所有網站繫結。 若要變更網站繫結，請在復原方案中的群組 3 之後新增 [IIS Web 層更新指令碼](https://aka.ms/asr-web-tier-update-runbook-classic)。

#### <a name="update-the-load-balancer-ip-address"></a>更新負載平衡器 IP 位址
如果您有 ARR 虛擬機器，若要更新 IP 位址，請在群組 4 之後新增 [IIS ARR 容錯移轉指令碼](https://aka.ms/asr-iis-arrtier-failover-script-classic)。

#### <a name="ssl-certificate-binding-for-an-https-connection"></a>Https 連線的 SSL 憑證繫結
網站可擁有相關聯的 SSL 憑證，以協助確保 Web 伺服器與使用者瀏覽器之間的安全通訊。 如果網站具有 HTTPS 連線，而且有相關聯的 HTTPS 網站繫結可透過 SSL 憑證繫結來繫結至 IIS 伺服器的 IP 位址，則您必須在容錯移轉後，為該憑證新增一個具有 IIS 虛擬機器 IP 位址的新網站繫結。

您可以對這些元件簽發 SSL 憑證：

* 網站的完整網域名稱。
* 伺服器的名稱。
* 網域名稱的萬用字元憑證。  
* IP 位址。 如果針對 IIS 伺服器的 IP 位址簽發 SSL 憑證，就必須針對 Azure 網站上 IIS 伺服器的 IP 位址簽發另一個 SSL 憑證。 並且必須為此憑證建立額外的 SSL 繫結。 因此，我們建議不要使用針對 IP 位址所核發的 SSL 憑證。 此選項較不常用，而且很快就會按照新的憑證授權單位/瀏覽器論壇變更而被取代。

#### <a name="update-the-dependency-between-the-web-tier-and-the-application-tier"></a>更新 Web 層與應用程式層之間的相依性
如果您有以虛擬機器 IP 位址為基礎的應用程式特有相依性，則必須在容錯移轉後更新此相依性。

## <a name="run-a-test-failover"></a>執行測試容錯移轉

1. 在 Azure 入口網站中，選取您的復原服務保存庫。
2. 選取您為 IIS Web 伺服陣列建立的復原方案。
3. 選取 [測試容錯移轉]。
4. 若要啟動測試容錯移轉程序，請選取復原點和 Azure 虛擬網路。
5. 當次要環境啟動時，您即可執行驗證。
6. 完成驗證時，若要清除測試容錯移轉環境，請選取 [驗證完成]。

如需詳細資訊，請參閱[在 Site Recovery 中測試容錯移轉至 Azure](site-recovery-test-failover-to-azure.md)。

## <a name="run-a-failover"></a>執行容錯移轉

1. 在 Azure 入口網站中，選取您的復原服務保存庫。
1. 選取您為 IIS Web 伺服陣列建立的復原方案。
1. 選取 [容錯移轉]。
1. 若要啟動容錯移轉程序，請選取復原點。

如需詳細資訊，請參閱[在 Site Recovery 中容錯移轉](site-recovery-failover.md)。

## <a name="next-steps"></a>後續步驟
* 深入了解如何使用 Site Recovery [複寫其他應用程式](site-recovery-workload.md)。
