---
title: 在 Azure Data Factory 中使用資料管理閘道來獲得高可用性 | Microsoft Docs
description: 本文說明如何藉由新增更多節點來相應放大資料管理閘道，以及藉由增加節點中可以並行執行的作業數來相應增加。
services: data-factory
documentationcenter: ''
author: nabhishek
manager: craigg
editor: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: abnarain
robots: noindex
ms.openlocfilehash: 58f612906af55575e9d42307af924ea0a8501ca1
ms.sourcegitcommit: 744747d828e1ab937b0d6df358127fcf6965f8c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "42144805"
---
# <a name="data-management-gateway---high-availability-and-scalability-preview"></a>資料管理閘道 - 高可用性和延展性 (預覽)
> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱 [自我裝載整合執行階段](../create-self-hosted-integration-runtime.md)。 


本文可協助您使用資料管理閘道/整合來設定高可用性和延展性解決方案。    

> [!NOTE]
> 本文假設您已熟悉 Integration Runtime (即先前的「資料管理閘道」) 的基本概念。 如果您並不熟悉，請參閱[資料管理閘道](data-factory-data-management-gateway.md)。

>**資料管理閘道版本 2.12.xxxx.x 和更新版本正式支援此預覽功能**。 請確定您使用版本 2.12.xxxx.x 或更新版本。 在[此處](https://www.microsoft.com/download/details.aspx?id=39717)下載最新版本的資料管理閘道。

## <a name="overview"></a>概觀
您可以從入口網站，將安裝在多個內部部署機器上的資料管理閘道關聯到單一邏輯閘道。 這些機器稱為**節點**。 您最多可以將**四個節點**關聯到一個邏輯閘道。 讓邏輯閘道擁有多個節點 (安裝了閘道的內部部署機器) 的好處如下：  

- 提升內部部署機器和雲端資料存放區之間的資料移動效能。  
- 如果某個節點因為某些原因而關閉，其他節點仍可供用來移動資料。 
- 如果某個節點必須離線以進行維護，其他節點仍可供用來移動資料。

您也可以設定節點中可以執行的**並行資料移動作業**數目，以相應增加在內部部署機器與雲端資料存放區之間移動資料的能力。 

您可以使用 Azure 入口網站來監視這些節點的狀態，以幫助您決定是要在邏輯閘道中新增還是移除節點。 

## <a name="architecture"></a>架構 
下圖可概述資料管理閘道之延展性和可用性功能的架構： 

![資料管理閘道 - 高可用性和延展性](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-high-availability-and-scalability.png)

**邏輯閘道**是您在 Azure 入口網站中新增至資料處理站的閘道。 以前，您只能將一個內部部署 Windows 機器關聯到安裝了邏輯閘道的資料管理閘道。 這個內部部署閘道機器稱為節點。 現在，您最多可以將**四個實體節點**關聯到邏輯閘道。 具有多個節點的邏輯閘道稱為**多節點閘道**。  

而這些節點全都是**作用中**狀態。 它們全都可以處理資料移動作業，以在內部部署機器和雲端資料存放區之間移動資料。 其中一個節點會同時作為發送器節點和背景工作節點。 群組中的其他節點則是背景工作節點。 **發送器**節點會從雲端服務提取資料移動工作/作業，並將其發送到背景工作節點 (包括發送器節點本身)。 **背景工作**節點會執行資料移動作業，以在內部部署機器和雲端資料存放區之間移動資料。 所有節點都是背景工作節點。 只有一個節點可以既是發送器節點，又是背景工作節點。    

一般來說，您一開始可以先建立一個節點，然後在現有節點已無法應付資料移動負載時**相應放大**以新增更多節點。 您也可以藉由增加可以在節點上執行的並行作業數目，來**相應增加**閘道節點的資料移動能力。 單一節點的閘道也能擁有這樣的能力 (即使該閘道尚未啟用延展性和可用性功能)。 

具有多個節點的閘道會讓所有節點的資料存放區認證保持同步。 如果節點之間的連線發生問題，認證可能就不會同步。當您為使用閘道的內部部署資料存放區設定認證時，它會將認證儲存在發送器/背景工作節點上。 發送器節點會與其他背景工作節點進行同步處理。 這個程序稱為**認證同步**。節點之間的通訊通道可以使用公開的 SSL/TLS 憑證來**加密**。 

## <a name="set-up-a-multi-node-gateway"></a>設定多節點閘道
本節假設您已經看過下列兩篇文章或熟悉這些文章中的概念： 

- [資料管理閘道](data-factory-data-management-gateway.md) - 提供詳細的閘道概觀。
- [在內部部署和雲端資料存放區之間移動資料](data-factory-move-data-between-onprem-and-cloud.md) - 包含如何使用具有單一節點之閘道的逐步指示。  

> [!NOTE]
> 在內部部署 Windows 機器上安裝資料管理閘道之前，請先參閱[主要文章](data-factory-data-management-gateway.md#prerequisites)中所列出的必要條件。

1. 在[逐步解說](data-factory-move-data-between-onprem-and-cloud.md#create-gateway)中，於建立邏輯閘道時啟用**高可用性和延展性**功能。 

    ![資料管理閘道 - 啟用高可用性和延展性](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-enable-high-availability-scalability.png)
2. 在 [設定] 頁面上，使用 [快速設定] 或 [手動設定] 連結在第一個節點 (內部部署 Windows 機器) 上安裝閘道。

    ![資料管理閘道 - 快速設定或手動設定](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-express-manual-setup.png)

    > [!NOTE]
    > 如果您使用 [快速設定] 選項，節點之間在通訊時不會加密。 節點名稱和機器名稱相同。 如果節點之間的通訊需要加密，或是您想要指定您所選擇的節點名稱，請使用 [手動設定]。 您之後就無法編輯節點名稱。
3. 如果您選擇 [快速設定]
    1. 閘道安裝成功之後，您會看到下列訊息：

        ![資料管理閘道 - 快速設定成功](media/data-factory-data-management-gateway-high-availability-scalability/express-setup-success.png)
    2. 遵循[這些指示](data-factory-data-management-gateway.md#configuration-manager)來啟動「資料管理組態管理員」。 您會看到閘道名稱、節點名稱、狀態等資訊。

        ![資料管理閘道 - 安裝成功](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-installation-success.png)
4. 如果您選擇 [手動設定]：
    1. 從 Microsoft 下載中心下載安裝套件，加以執行以在您的機器上安裝閘道。
    2. 使用 [設定] 頁面中的 [驗證金鑰] 來註冊閘道。
    
        ![資料管理閘道 - 安裝成功](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-authentication-key.png)
    3. 在 [新增閘道節點] 頁面上，您可以為閘道節點提供自訂**名稱**。 根據預設，節點名稱和機器名稱相同。    

        ![資料管理閘道 - 指定名稱](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-name.png)
    4. 在下一頁中，您可以選擇是否要**為節點之間的通訊啟用加密**。 按一下 [略過] 可停用加密 (預設)。

        ![資料管理閘道 - 啟用加密](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-node-encryption.png)  
    
        > [!NOTE]
        > 在邏輯閘道中只有單一閘道節點時才支援變更加密模式。 若要在閘道有多個節點時變更加密模式，請執行下列步驟：只留下一個節點，其他全都刪除，變更加密模式，然後重新新增節點。
        > 
        > 如需 TLS/SSL 憑證的使用需求清單，請參閱 [TLS/SSL 憑證需求](#tlsssl-certificate-requirements)一節。 
    5. 在閘道安裝成功之後，按一下 [啟動組態管理員]：
    
        ![手動設定 - 啟動組態管理員](media/data-factory-data-management-gateway-high-availability-scalability/manual-setup-launch-configuration-manager.png)   
    6. 您會在節點 (內部部署 Windows 機器) 上看到「資料管理閘道組態管理員」，裡面會顯示連線狀態、**閘道名稱**和**節點名稱**。  

        ![資料管理閘道 - 安裝成功](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-installation-success.png)

        > [!NOTE]
        > 如果要在 Azure VM 上佈建閘道，可以使用[此 Azure Resource Manager 範本](https://github.com/Azure/azure-quickstart-templates/tree/master/101-mutiple-vms-with-data-management-gateway)。 這個指令碼會建立邏輯閘道、為安裝了資料管理閘道軟體的 VM 進行設定，並向邏輯閘道註冊這些 VM。 
6. 在 Azure 入口網站啟動 [閘道] 頁面： 
    1. 在入口網站中的資料處理站首頁上，按一下 [已連結的服務]。
    
        ![Data Factory 首頁](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-home-page.png)
    2. 選取**閘道**以查看 [閘道] 頁面：
    
        ![Data Factory 首頁](media/data-factory-data-management-gateway-high-availability-scalability/linked-services-gateway.png)
    4. 您會看到 [閘道] 頁面：   

        ![具有單一節點之閘道的檢視](media/data-factory-data-management-gateway-high-availability-scalability/gateway-first-node-portal-view.png) 
7. 在工具列上按一下 [新增節點] 來對邏輯閘道新增節點。 如果您打算使用快速設定，請從將新增為閘道節點的內部部署機器執行此步驟。 

    ![資料管理閘道 - 新增節點功能表](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-add-node-menu.png)
8. 步驟與第一個節點的設定類似。 如果您選擇手動安裝選項，組態管理員 UI 可讓您設定節點名稱： 

    ![組態管理員 - 安裝第二個閘道](media/data-factory-data-management-gateway-high-availability-scalability/install-second-gateway.png)
9. 成功在節點上安裝閘道之後，組態管理員工具會顯示下列畫面：  

    ![組態管理員 - 安裝第二個閘道成功](media/data-factory-data-management-gateway-high-availability-scalability/second-gateway-installation-successful.png)
10. 如果您在入口網站中開啟 [閘道] 頁面，您現在會看到兩個閘道節點： 

    ![入口網站中具有兩個節點的閘道](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-multi-node-monitoring.png)
11. 若要刪除閘道節點，請在工具列上按一下 [刪除節點]，選取您要刪除的節點，然後在工具列中按一下 [刪除]。 這個動作會從群組中刪除選取的節點。 請注意，此動作不會從節點 (內部部署 Windows 機器) 解除安裝資料管理閘道軟體。 請在內部部署機器上使用控制台中的 [新增或移除程式] 來解除安裝閘道。 當您從節點中解除安裝閘道時，系統便會自動在入口網站中刪除該節點。   

## <a name="upgrade-an-existing-gateway"></a>升級現有閘道
您可以升級現有閘道來使用高可用性和延展性功能。 這項功能僅適用於資料管理閘道版本 >= 2.12.xxxx 的節點。 您可以在資料管理閘道組態管理員的 [說明] 索引標籤中查看機器上所安裝之資料管理閘道的版本。 

1. 從 [Microsoft 下載中心](https://www.microsoft.com/download/details.aspx?id=39717)下載並執行 MSI 安裝套件，以將內部部署機器上的閘道更新為最新版本。 如需詳細資料，請參閱[安裝](data-factory-data-management-gateway.md#installation)一節。  
2. 瀏覽至 Azure 入口網站。 啟動您資料處理站的 [Data Factory] 頁面。 按一下 [已連結的服務] 圖格來啟動 [已連結的服務] 頁面。 選取閘道以啟動**閘道頁面**。 按一下並啟用 [預覽功能]，如下圖所示： 

    ![資料管理閘道 - 啟用預覽功能](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-existing-gateway-enable-high-availability.png)   
2. 在入口網站啟用預覽功能後，關閉所有頁面。 重新開啟**閘道頁面**以查看新的預覽使用者介面 (UI)。
 
    ![資料管理閘道 - 啟用預覽功能成功](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-preview-success.png)

    ![資料管理閘道 - 預覽 UI](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-preview.png)

    > [!NOTE]
    > 在升級期間，第一個節點的名稱就是機器的名稱。 
3. 現在，新增節點。 在 [閘道] 頁面上，按一下 [新增節點]。  

    ![資料管理閘道 - 新增節點功能表](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-add-node-menu.png)

    請遵循上一節的指示來設定節點。 

### <a name="installation-best-practices"></a>安裝最佳做法

- 為閘道器設定主機電腦上的電源計劃，使電腦不休眠。 如果主機電腦休眠，閘道器不會回應資料要求。
- 請備份與閘道器相關聯的憑證。
- 確定所有節點都有類似的設定 (建議選項) 以獲得理想的效能。 
- 至少新增兩個節點，以確保高可用性。  

### <a name="tlsssl-certificate-requirements"></a>TLS/SSL 憑證需求
以下是用來保護整合執行階段節點間通訊之 TLS/SSL 憑證的需求：

- 憑證必須是受信任的 X509 v3 公開憑證。 建議您使用公開 (第三方) 憑證授權單位 (CA) 所發出的憑證。
- 每個整合執行階段節點皆必須信任此憑證，以及執行認證管理員應用程式的用戶端電腦。 
> [!NOTE]
> 從複製精靈/Azure 入口網站安全地設定認證時，會使用認證管理員應用程式。 此外亦可以從與內部部署/私人資料存放區位於相同網路的所有電腦上，啟動此應用程式。
- 支援萬用字元憑證。 若您的 FQDN 名稱為 **node1.domain.contoso.com**，則您可使用 ***.domain.contoso.com** 做為憑證的主體名稱。
- 由於系統僅會使用主體別名的最後一個項目，其他所有項目則會因目前的限制而遭到忽略，因此不建議使用 SAN 憑證。 例如 若您具有 SAN 憑證，且其 SAN 為 **node1.domain.contoso.com** 和 **node2.domain.contoso.com**，則您僅可在 FQDN 為 **node2.domain.contoso.com** 的電腦上使用此憑證。
- 支援 Windows Server 2012 R2 所支援的任何 SSL 憑證金鑰大小。
- 不支援使用 CNG 金鑰的憑證。

#### <a name="faq-when-would-i-not-enable-this-encryption"></a>常見問題集：何時不會啟用此加密？
啟用加密會為您的基礎結構增加特定成本 (擁有公用憑證)，因此在以下情況可能會跳過啟用加密：
- 在信任的網路或是採用 IP/SEC 等透明加密的網路上，執行整合執行階段時。 由於這種頻道通訊僅限在信任的網路內部運作，因此可能不需要額外使用加密。
- 整合執行階段未在生產環境中執行時。 這可協助降低 TLS/SSL 憑證的成本。


## <a name="monitor-a-multi-node-gateway"></a>監視多節點閘道
### <a name="multi-node-gateway-monitoring"></a>多節點閘道監視
在 Azure 入口網站中，您可以檢視每個節點近乎即時的資源使用率 (CPU、記憶體、網路 (輸入/輸出) 等) 快照集以及閘道節點的狀態。 

![資料管理閘道 - 多節點監視](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-multi-node-monitoring.png)

您可以在 [閘道] 頁面啟用 [進階設定] 以查看進階的計量，例如**網路** (輸入/輸出)、**角色和認證狀態** (有助於偵錯閘道問題) 和**並行作業** (執行中/限制) (可在效能微調期間據以修改/變更)。 下表說明 [閘道節點] 清單中的資料行：  

監視屬性 | 說明
:------------------ | :---------- 
名稱 | 邏輯閘道和閘道相關聯節點的名稱。  
狀態 | 邏輯閘道和閘道節點的狀態。 範例：線上/離線/受限制/等等。如需這些狀態的相關資訊，請參閱[閘道狀態](#gateway-status)一節。 
版本 | 顯示邏輯閘道和每個閘道節點的版本。 邏輯閘道的版本取決於群組中大多數節點的版本。 如果邏輯閘道設定中有不同版本的節點，則只有版本號碼和邏輯閘道相同的節點會正常運作。 其他節點會進入受限制模式，並需要加以手動更新 (如果自動更新失敗才需要這麼做)。 
可用的記憶體 | 閘道節點上可用的記憶體。 這個值是近乎即時的快照集。 
CPU 使用率 | 閘道節點的 CPU 使用率。 這個值是近乎即時的快照集。 
網路功能 (輸入/輸出) | 閘道節點的網路使用率。 這個值是近乎即時的快照集。 
並行作業 (執行中/限制) | 每個節點上執行的作業或工作數目。 這個值是近乎即時的快照集。 限制表示每個節點的最大並行作業數。 這個值會根據機器大小來定義。 您可以提高限制以在進階案例 (CPU/記憶體/網路並未充分使用，但活動會逾時的案例) 中相應增加並行作業執行能力。單一節點的閘道也能擁有這樣的能力 (即使該閘道尚未啟用延展性和可用性功能)。 如需詳細資訊，請參閱[調整考量](#scale-considerations)一節。 
角色 | 有兩種角色 – 發送器和背景工作角色。 所有節點都是背景工作角色，這表示它們全都能用來執行作業。 發送器節點只有一個，可用來提取雲端服務中的工作/作業，並發送到不同的背景工作節點 (包括發送器節點本身)。 

![資料管理閘道 - 進階的多節點監視](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-multi-node-monitoring-advanced.png)

### <a name="gateway-status"></a>閘道狀態

下表提供**閘道節點**的可能狀態： 

狀態  | 註解/案例
:------- | :------------------
線上 | 節點已連線至 Data Factory 服務。
離線 | 節點已離線。
升級中 | 節點正在自動更新。
限制 | 由於連線問題。 可能是因為 HTTP 連接埠 8050 問題、服務匯流排連線問題或認證同步問題。 
非使用中 | 節點所在的組態不同於其他大多數節點的組態。<br/><br/> 節點無法連線至其他節點時，便會處於非使用中狀態。 


下表提供**邏輯閘道**的可能狀態。 閘道的狀態取決於閘道節點的狀態。 

狀態 | 註解
:----- | :-------
需要註冊 | 此邏輯閘道還沒有已註冊的節點
線上 | 閘道節點已連線
離線 | 沒有處於線上狀態的節點。
限制 | 此閘道中的節點並非全都處於健康情況良好的狀態。 這個狀態是某些節點可能會關閉的警告！ <br/><br/>可能是因為發送器/背景工作節點有認證同步問題。 

### <a name="pipeline-activities-monitoring"></a>管線/活動監視
Azure 入口網站可為管線監視提供細微的節點層級詳細資料。 例如，它會顯示哪些活動在哪個節點上執行。 這項資訊有助於了解特定節點的效能問題 (例如由於網路節流而導致)。 

![資料管理閘道 - 管線的多節點監視](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-multi-node-monitoring-pipelines.png)

![資料管理閘道 - 管線詳細資料](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-multi-node-pipeline-details.png)

## <a name="scale-considerations"></a>調整考量

### <a name="scale-out"></a>相應放大
當**可用記憶體偏低**而 **CPU 使用率偏高**時，新增節點有助於相應放大機器所能承受的負載。 如果活動因為逾時或閘道節點離線而失敗，對閘道新增節點將有所助益。
 
### <a name="scale-up"></a>相應增加
當可用的記憶體和 CPU 並未充分使用，但閒置容量已為 0，就應該增加節點上可執行的並行作業數目來進行相應增加。 當閘道超載而導致活動逾時，您也可以相應增加。 如下圖所示，您可以提高節點的容量上限。 建議您一開始先提高為兩倍。  

![資料管理閘道 - 調整考量](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-scale-considerations.png)


## <a name="known-issuesbreaking-changes"></a>已知問題/重大變更

- 單一邏輯閘道目前最多可以有四個實體閘道節點。 如果基於效能考量而需要四個以上的節點，請傳送電子郵件給 [DMGHelp@microsoft.com](mailto:DMGHelp@microsoft.com)。
- 您無法從另一個邏輯閘道使用驗證金鑰來重新註冊閘道節點，以跳脫目前的邏輯閘道。 若要重新註冊，請從節點解除安裝閘道、重新安裝閘道，然後使用驗證金鑰向其他邏輯閘道進行註冊。 
- 如果您的所有閘道節點需要 HTTP Proxy，請在 diahost.exe.config 和 diawp.exe.config 中設定該 Proxy，並使用伺服器管理員來確定所有節點都有相同的 diahost.exe.config 和 diawip.exe.config。如需詳細資訊，請參閱[設定 Proxy 設定](data-factory-data-management-gateway.md#configure-proxy-server-settings)一節。 
- 若要在閘道組態管理員中變更節點間通訊的加密模式，請在入口網站中只留下一個節點而刪除其他所有節點。 然後，於加密模式變更後重新新增節點。
- 如果您選擇要加密節點間的通訊通道，請使用官方的 SSL 憑證。 自我簽署憑證可能會造成連線問題，因為相同的憑證可能不會受到其他機器上的憑證授權單位清單所信任。 
- 當節點版本低於邏輯閘道版本時，您就無法向邏輯閘道註冊閘道節點。 請從入口網站中刪除邏輯閘道的所有節點，以便您可以註冊比它的版本還低的節點 (降級)。 如果您刪除邏輯閘道的所有節點，請以手動方式對該邏輯閘道安裝並註冊新的節點。 在此情況下不支援快速設定。
- 您無法使用快速設定來對仍在使用雲端認證的現有邏輯閘道安裝節點。 您可以從閘道組態管理員的 [設定] 索引標籤查看認證的儲存位置。
- 您無法使用快速設定來對已啟用節點間加密的現有邏輯閘道安裝節點。 設定加密模式牽涉到手動新增憑證，因此您無法再選擇快速安裝。 
- 若要從內部部署環境複製檔案，請勿再使用 \\localhost 或 C:\files，因為您可能無法透過所有節點存取 localhost 或本機磁碟機。 請改用 \\ServerName\files 來指定檔案的位置。


## <a name="rolling-back-from-the-preview"></a>從預覽復原 
若要從預覽復原，請留下一個節點，其他的節點全都刪除。 刪除哪一個節點都行，但請確保邏輯閘道中至少有一個節點。 您可以藉由在機器上解除安裝閘道或使用 Azure 入口網站來刪除節點。 在 Azure 入口網站的 [Data Factory] 頁面中，按一下 [已連結的服務] 來啟動 [已連結的服務] 頁面。 選取閘道以啟動 [閘道] 頁面。 在 [閘道] 頁面中，您可以看到與閘道相關聯的節點。 該頁面可讓您從閘道中刪除節點。
 
在刪除之後，按一下同一個 Azure 入口網站頁面中的 [預覽功能]，並停用預覽功能。 您已將您的閘道重設為只有一個節點的 GA (正式上市) 閘道。


## <a name="next-steps"></a>後續步驟
請檢閱下列文章：
- [資料管理閘道](data-factory-data-management-gateway.md) - 提供詳細的閘道概觀。
- [在內部部署和雲端資料存放區之間移動資料](data-factory-move-data-between-onprem-and-cloud.md) - 包含如何使用具有單一節點之閘道的逐步指示。 
