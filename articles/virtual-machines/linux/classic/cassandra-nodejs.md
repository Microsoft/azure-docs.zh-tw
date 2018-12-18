---
title: 從 Node.js 在 Azure 中的 Linux 上執行 Cassandra 叢集
description: 如何從 Node.js 應用程式在 Azure 虛擬機器的 Linux 上執行 Cassandra 叢集
services: virtual-machines-linux
documentationcenter: nodejs
author: craigshoemaker
manager: routlaw
editor: ''
tags: azure-service-management
ms.assetid: 30de1f29-e97d-492f-ae34-41ec83488de0
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 08/17/2017
ms.author: cshoe
ms.openlocfilehash: b1945c68f0e320c834ae93a590f420403263a0fd
ms.sourcegitcommit: d7725f1f20c534c102021aa4feaea7fc0d257609
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37098935"
---
# <a name="run-a-cassandra-cluster-on-linux-in-azure-with-nodejs"></a>使用 Node.js 在 Azure 中的 Linux 上執行 Cassandra 叢集

> [!IMPORTANT] 
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 請參閱 [Datastax Enterprise](https://azure.microsoft.com/documentation/templates/datastax) 的 Resource Manager 範本和 [CentOS 上的 Spark 叢集和 Cassandra](https://azure.microsoft.com/documentation/templates/spark-and-cassandra-on-centos/)。

## <a name="overview"></a>概觀
Microsoft Azure 是一個開放雲端平台，可執行 Microsoft 和非 Microsoft 軟體，包括作業系統、應用程式伺服器、傳訊中介軟體，以及來自商業和開放原始碼模型的 SQL 和 NoSQL 資料庫。 如果要在包括 Azure 在內的公用雲端上建立具有恢復功能的服務，應用程式伺服器和儲存層都必須要有仔細的規劃和審慎的架構。 Cassandra 的分散式儲存架構天生就有助於建置可針對叢集失敗容錯的高可用性系統。 Cassandra 為 cassandra.apache.org 的 Apache Software Foundation 維護的雲端規模 NoSQL 資料庫。Cassandra 是以 Java 撰寫的。 因此它可同時在 Windows 和 Linux 平台上執行。

本文的重點將說明如何使用虛擬機器和虛擬網路，在 Ubuntu 上部署 Cassandra 作為單一和多重資料中心叢集。 用於實際執行環境最佳化工作負載的叢集部署超出本文的範圍，因為它需要多磁碟節點設定、適當的通道拓撲設計及資料模型來支援所需的複寫、資料一致性、輸送量和高可用性需求。

本文採用基本的方式說明建置 Cassandra 叢集所牽涉的工作，比起 Docker、Chef 或 Puppet，前者可讓基礎結構部署輕鬆許多。  

## <a name="the-deployment-models"></a>部署模型
Microsoft Azure 網路功能可供部署獨立的私人叢集，透過限制其存取權以達到精細的網路安全性。  由於本文是關於基本層級的 Cassandra 部署，因此我們不會著重在一致性層級和輸送量的最佳儲存體設計。 以下是假設叢集的網路功能需求清單：

* 外部系統無法從 Azure 內部或外部存取 Cassandra 資料庫
* Cassandra 叢集必須位於 thrift 流量的負載平衡器後方
* 在每個資料中心部署 Cassandra 節點為兩個群組以提高叢集可用性
* 鎖定叢集，只有應用程式伺服器陣列可直接存取資料庫
* 除了 SSH 以外，沒有其他公用網路端點
* 每個 Cassandra 節點需要固定的內部 IP 位址

Cassandra 可以部署到單一 Azure 區域或多個區域，視工作負載的分散特性而定。 多重區域部署模型透過相同的 Cassandra 基礎結構，可用來服務更接近特定地理位置的使用者。 Cassandra 的內建節點複寫負責同步處理來自多個資料中心的多重主機寫入，並且對應用程式提供一致的資料檢視。 多重區域部署也可以協助降低更多 Azure 服務中斷的風險。 Cassandra 的可微調一致性和複寫拓撲有助於滿足應用程式不同的 RPO 需求。

### <a name="single-region-deployment"></a>單一區域部署
讓我們從單一區域部署開始，並學習建立多重區域模型。 Azure 虛擬網路功能將用來建立獨立的子網路，以滿足先前所描述的網路安全性需求。  建立單一區域部署中所述的程序使用 Ubuntu 14.04 LTS 和 Cassandra 2.08。 不過，程序要改採用其他 Linux 版本也非常容易。 下列是單一區域部署的一些系統特性。  

**高可用性：** Cassandra 節點 (如圖 1 所示) 已部署至兩個可用性設定組，因此，節點會散佈於多個容錯網域之間，以獲取高可用性。 利用每個可用性設定組進行標註的 VM 會對應至 2 個容錯網域。 Azure 會使用容錯網域的概念來管理意外停機時間 (例如，硬體或軟體失敗)。 升級網域 (例如主機或客體作業系統修補/升級、應用程式升級) 的概念用於管理排定停機時間。 如需用來取得高可用性之容錯和升級網域的角色，請參閱 [Azure 應用程式的災害復原與高可用性](http://msdn.microsoft.com/library/dn251004.aspx) 。

![單一區域部署](./media/cassandra-nodejs/cassandra-linux1.png)

圖 1：單一區域部署

請注意，在撰寫本文時，Azure 不允許將一組 VM 明確對應到特定容錯網域。因此，即使使用如圖 1 所示的部署模型，統計上來說有可能所有的虛擬機器都對應到兩個容錯網域而不是四個。

**負載平衡 Thrift 流量：** Web 伺服器內部的 Thrift 用戶端程式庫會透過內部負載平衡器連線到叢集。 這需要將內部負載平衡器新增到「資料」子網路的程序 (請參閱圖 1)，這個子網路位於裝載 Cassandra 叢集的雲端服務內容中。 一旦定義內部負載平衡器之後，每個節點所需要的負載平衡端點，必須使用具有先前定義之負載平衡器名稱的負載平衡集註解來新增。 如需詳細資訊，請參閱 [Azure內部負載平衡 ](../../../load-balancer/load-balancer-internal-overview.md)。

**叢集種子：** 務必選取可用性最高的節點作為種子，因為新的節點會與種子節點進行通訊以探索叢集的拓撲。 將每個可用性設定組中的一個節點指定為種子節點，可避免單一失敗點。

**複寫因數和一致性層級：** Cassandra 內建的高可用性和資料耐久性的特點在於複寫因數 (RF - 儲存在叢集上的每個資料列複本數) 和一致性層級 (將結果傳回呼叫端之前要讀取/寫入的複本數)。 複寫因數是在 KEYSPACE (類似關聯式資料庫) 建立期間所指定，而一致性層級是在發出 CRUD 查詢的同時所指定。 如需一致性詳細資料和仲裁計算的公式，請參閱 Cassandra 文件中的 [一致性設定](https://docs.datastax.com/en/cassandra/3.0/cassandra/dml/dmlConfigConsistency.html) 。

Cassandra 支援兩種類型的資料完整性模型 – 一致性和最終一致性。「複寫因素」和「一致性層級」將一起判斷資料會在寫入作業完成時就達成一致，或者最終才達成一致。 例如，指定「一致性層級」為 QUORUM，將一律確保在任何一致性層級時的資料一致性，低於達到 QUORUM (例如 ONE) 所需寫入的複本數目，會造成資料在最終才達成一致。

以上顯示的 8 節點叢集，使用 3 的複寫因素和 QUORUM (為了一致性而讀取或寫入 2 個節點) 讀取/寫入的一致性層級，在應用程式開始注意失敗之前，每個複寫群組最多只能有 1 個節點可避免理論性遺失。 這假設所有關鍵值空間都已經順利平衡讀取/寫入要求。  針對已部署的叢集使用下列參數：

單一區域 Cassandra 叢集設定：

| 叢集參數 | 值 | 備註 |
| --- | --- | --- |
| 節點數目 (N) |8 |叢集中的節點總數 |
| 複寫因素 (RF) |3 |指定資料列的複本數目 |
| 一致性層級 (寫入) |QUORUM[(RF/2) +1) = 2] 公式結果無條件捨去 |在回應傳送至呼叫者之前，最多寫入 2 個複本；第 3 個複本會以最終一致的方式寫入。 |
| 一致性層級 (讀取) |QUORUM [(RF/2) +1= 2] 公式結果無條件捨去 |在傳送回應給呼叫者之前讀取 2 個複本。 |
| 複寫策略 |如需 NetworkTopologyStrategy 的詳細資訊，請參閱 Cassandra 文件中的 [資料複寫](https://docs.datastax.com/en/cassandra/3.0/cassandra/architecture/archDataDistributeAbout.html) (英文) |了解部署拓撲並將複本放在節點上，最後所有複本就不會都位於相同的機架上 |
| Snitch |如需 GossipingPropertyFileSnitch 的詳細資訊，請參閱 Cassandra 文件中的 [Switches](https://docs.datastax.com/en/cassandra/3.0/cassandra/architecture/archSnitchesAbout.html) (英文) |NetworkTopologyStrategy 使用 Snitch 的概念來了解拓撲。 GossipingPropertyFileSnitch 在將各個節點對應到資料中心與機架時提供較好的控制。 叢集再使用 Gossip 來散佈這項資訊。 相對於 PropertyFileSnitch，這在動態 IP 設定中簡單許多 |

**Cassandra 叢集的 Azure 考量：** Microsoft Azure 虛擬機器功能會使用 Azure Blob 儲存體來取得磁碟持續性；Azure 儲存體會基於高持久性因素來為每個磁碟儲存三個複本。 這表示插入 Cassandra 資料表的每列資料已經儲存在三個複本中。 因此，即使複寫因數 (RF) 為 1，也已經處理資料一致性。 複寫因數是 1 的主要問題在於，即使單一 Cassandra 節點失敗，應用程式還是會經歷停機時間。 不過，如果節點因為 Azure 網狀架構控制器所辨識出的問題 (例如，硬體、系統軟體失敗) 而關閉，它將會使用相同的儲存體磁碟機，在其位置中佈建新的節點。 佈建新節點來取代舊節點，可能需要幾分鐘的時間。  類似於如客體 OS 變更、Cassandra 升級及應用程式變更等預定維護活動，Azure 網狀架構控制器會在叢集中執行節點的輪流升級。  輪流升級也可能會一次關閉數個節點，因此，叢集可能會遇到數個磁碟分割短暫停機的狀況。 不過，資料不會因為內建的 Azure 儲存體備援而遺失。  

對於部署到 Azure 但不需要高可用性的系統 (例如，大約 99.9，相當於每年 8.76 個小時，請參閱[高可用性](http://en.wikipedia.org/wiki/High_availability)以了解詳細資料)，您可以採用 RF=1 和一致性層級=ONE 來執行。  對於具有高可用性需求的應用程式，RF=3 和一致性層級=QUORUM 可容忍其中一個節點之其中一個複本的停機時間。 不能在傳統部署 (例如內部部署) 中使用 RF=1，因為如磁碟損壞所產生的問題可能會導致資料遺失。   

## <a name="multi-region-deployment"></a>多重區域部署
Cassandra 的資料中心感知複寫和上述的一致性模型有助於進行多重區域部署，而不需使用任何外部工具。 這與傳統的關聯式資料庫不同，後者在設定資料庫鏡像以進行多重主機寫入時相當複雜。 Cassandra 的多重區域設定有助於包括下列的使用案例：

**以近接性為基礎的部署：** 清除租用戶使用者與區域對應的多租用戶應用程式，可以因為多地區叢集的低延遲而受益。 例如，適用於教育機構的學習管理系統可以在美國東部和美國西部區域部署分散式叢集，分別作為適用於交易與分析的校園。 資料可以在讀取和寫入期間維持本機一致性，而且最終可在這兩個區域間維持一致性。 其他的範例還有媒體發佈、電子商務，以及可為地理位置集中之使用者提供服務的一切 (這是此部署模型的絕佳使用案例)。

**高可用性：** 備援是取得軟體和硬體高可用性的關鍵因數；如需詳細資訊，請參閱＜在 Microsoft Azure 上建置可靠的雲端系統＞。 在 Microsoft Azure 上，實現真正備援的唯一可靠方式就是部署多區域叢集。 應用程式可部署於主動/主動或主動/被動模式中，如果其中一個區域已關閉，Azure 流量管理員就能將流量重新導向至作用中的區域。  使用單一區域部署時，如果可用性是 99.9，則兩個區域的部署可以達到 99.9999 的可用性，計算公式如下：(1-(1-0.999) * (1-0.999))*100)。如需詳細資訊，請參閱上述文件。

**災害復原：** 如果設計正確，多區域的 Cassandra 叢集就能承受重大的資料中心中斷。 如果某個區域已關閉，部署到其他區域的應用程式就能開始為使用者提供服務。 和所有其他的商務持續實作一樣，應用程式必須能對非同步管線中的資料所產生的部分資料遺失進行容錯。 不過，相較於傳統資料庫復原程序所花費的時間，Cassandra 能夠更快速地進行復原。 圖 2 顯示每個區域中有八個節點的典型多區域部署模型。 這兩個區域是彼此具備相同對稱性的鏡映影像。真實世界的設計取決於工作負載類型 (例如，交易或分析)、RPO、RTO、資料一致性，以及可用性需求。

![多區域部署](./media/cassandra-nodejs/cassandra-linux2.png)

圖 2：多區域 Cassandra 部署

### <a name="network-integration"></a>網路整合
部署至私人網路、位於兩個區域的虛擬機器設定組使用 VPN 通道彼此通訊。 VPN 通道連線在網路部署程序期間佈建的兩個軟體閘道。 這兩個區域有類似的網路架構，分別稱為 "Web" 和「資料」子網路。Azure 網路功能允許建立多個子網路，以及根據網路安全性的需要套用 ACL。 設計資料中心之間的叢集拓撲時，需要考量網路流量的通訊延遲與經濟影響。

### <a name="data-consistency-for-multi-data-center-deployment"></a>多重資料中心部署的資料一致性
分散式部署需要留意叢集拓撲對輸送量和高可用性的影響。 選取 RF 和一致性層級時，也必須以不需根據所有資料中心可用性進行仲裁的這種方式。
對於需要高度一致性的系統，LOCAL_QUORUM 的一致性層級 (針對讀取和寫入) 可確保來自本機節點的本機讀取和寫入達到要求，同時資料會以非同步方式複寫到遠端資料中心。  表 2 摘要多重區域叢集 (稍後詳細說明) 的設定詳細資料。

**兩個區域的 Cassandra 叢集組態**

| 叢集參數 | 值 | 備註 |
| --- | --- | --- |
| 節點數目 (N) |8 + 8 |叢集中的節點總數 |
| 複寫因素 (RF) |3 |指定資料列的複本數目 |
| 一致性層級 (寫入) |LOCAL_QUORUM [(sum(RF)/2) +1) = 4] 公式結果無條件捨去 |2 個節點會以同步方式寫入第一個資料中心；仲裁所需的其他 2 個節點會以非同步方式寫入第二個資料中心。 |
| 一致性層級 (讀取) |LOCAL_QUORUM ((RF/2) +1) = 2 公式結果無條件捨去 |只會達到來自一個區域的讀取要求；在回應傳送回用戶端之前，會讀取 2 個節點。 |
| 複寫策略 |如需 NetworkTopologyStrategy 的詳細資訊，請參閱 Cassandra 文件中的 [資料複寫](https://docs.datastax.com/en/cassandra/3.0/cassandra/architecture/archDataDistributeAbout.html) (英文) |了解部署拓撲並將複本放在節點上，最後所有複本就不會都位於相同的機架上 |
| Snitch |如需 GossipingPropertyFileSnitch 的詳細資訊，請參閱 Cassandra 文件中的 [Snitches](https://docs.datastax.com/en/cassandra/3.0/cassandra/architecture/archSnitchesAbout.html) (英文) |NetworkTopologyStrategy 使用 Snitch 的概念來了解拓撲。 GossipingPropertyFileSnitch 在將各個節點對應到資料中心與機架時提供較好的控制。 叢集再使用 Gossip 來散佈這項資訊。 相對於 PropertyFileSnitch，這在動態 IP 設定中簡單許多 |

## <a name="the-software-configuration"></a>軟體設定
部署期間將使用下列軟體版本：

<table>
<tr><th>軟體</th><th>來源</th><th>版本</th></tr>
<tr><td>JRE    </td><td>[JRE 8](http://www.oracle.com/technetwork/java/javase/downloads/server-jre8-downloads-2133154.html) </td><td>8U5</td></tr>
<tr><td>JNA    </td><td>[JNA](https://github.com/twall/jna) </td><td> 3.2.7</td></tr>
<tr><td>Cassandra</td><td>[Apache Cassandra 2.0.8](http://www.apache.org/dist/cassandra/)</td><td> 2.0.8</td></tr>
<tr><td>Ubuntu    </td><td>[Microsoft Azure](https://azure.microsoft.com/) </td><td>14.04 LTS</td></tr>
</table>

您下載 JRE 時，必須手動接受 Oracle 授權。 因此，為了簡化部署，請將所有必要的軟體下載到桌面。 然後將它上傳至 Ubuntu 範本映像，以便為叢集部署做準備。

將上述軟體下載到本機電腦上已知的下載目錄 (例如，Windows 上的 %TEMP%/downloads，或者大多數 Linux 散發套件或 Mac 上的 ~/Downloads)。

### <a name="create-ubuntu-vm"></a>建立 UBUNTU VM
在此步驟的程序中，您會建立包含必要軟體的 Ubuntu 映像，讓映像可以重複用於佈建多個 Cassandra 節點。  

#### <a name="step-1-generate-ssh-key-pair"></a>步驟 1：產生 SSH 金鑰組
在佈建階段，Azure 需要 PEM 或 DER 編碼的 X509 公用金鑰。 請參考如何在 Azure 上使用 SSH 搭配 Linux (英文) 上的指示來產生公用/私密金鑰組。 如果您計劃在 Windows 或 Linux 上使用 putty.exe 作為 SSH 用戶端，您必須使用 puttygen.exe，將 PEM 編碼的 RSA 私密金鑰轉換為 PPK 格式。 如需此操作的指示，請參閱上述網頁。

#### <a name="step-2-create-ubuntu-template-vm"></a>步驟 2：建立 Ubuntu 範本 VM
若要建立範本 VM，請登入 Azure 入口網站，並使用下列順序：依序按一下 [新增]、[計算]、[虛擬機器]、[從資源庫]、[UBUNTU]、[Ubuntu Server 14.04 LTS]，然後按一下向右箭號。 如需說明如何建立 Linux VM 的教學課程，請參閱建立執行 Linux 的虛擬機器 (英文)。

在 [虛擬機器設定] 的第 1 個畫面上輸入下列資訊：

<table>
<tr><th>欄位名稱              </td><td>       欄位值               </td><td>         備註                </td><tr>
<tr><td>版本發行日期    </td><td> 從下拉式清單選取日期</td><td></td><tr>
<tr><td>[虛擬機器名稱]    </td><td> cass-template                   </td><td> 這是 VM 的主機名稱 </td><tr>
<tr><td>層次                     </td><td> 標準                           </td><td> 保留預設值              </td><tr>
<tr><td>大小                     </td><td> A1                              </td><td>根據 IO 需求選取 VM。針對此目的保留預設值 </td><tr>
<tr><td> [新使用者名稱] 中             </td><td> localadmin                       </td><td> "admin" 在 Ubuntu 12.xx 以及更新版本中是保留的使用者名稱</td><tr>
<tr><td> 驗證         </td><td> 按一下核取方塊                 </td><td>如果您想要以 SSH 金鑰確保安全，請核取此方塊 </td><tr>
<tr><td> 憑證             </td><td> 公用金鑰憑證的檔案名稱 </td><td> 使用先前產生的公開金鑰</td><tr>
<tr><td> 新密碼    </td><td> 強式密碼 </td><td> </td><tr>
<tr><td> 確認密碼    </td><td> 強式密碼 </td><td></td><tr>
</table>

在 [虛擬機器設定] 的第 2 個畫面上輸入下列資訊：

<table>
<tr><th>欄位名稱             </th><th> 欄位值                       </th><th> 備註                                 </th></tr>
<tr><td> 雲端服務    </td><td> 建立新的雲端服務    </td><td>雲端服務是一個運算如虛擬機器等資源的容器</td></tr>
<tr><td> 雲端服務 DNS 名稱    </td><td>ubuntu-template.cloudapp.net    </td><td>提供一個機器中立的負載平衡器名稱</td></tr>
<tr><td> [區域/同質群組/虛擬網路] </td><td>    美國西部    </td><td> 選取您 Web 應用程式存取 Cassandra 叢集時的來源地區</td></tr>
<tr><td>儲存體帳戶 </td><td>    使用預設值    </td><td>使用預設的儲存體帳戶或在特定區域中預先建立的儲存體帳戶</td></tr>
<tr><td>可用性集合 </td><td>    None </td><td>    保留為空白</td></tr>
<tr><td>端點    </td><td>使用預設值 </td><td>    使用預設的 SSH 設定 </td></tr>
</table>

按一下向右箭號，保留畫面 #3 上的預設值。 按一下「檢查」按鈕完成 VM 佈建程序。 請稍候幾分鐘，名稱為 “ubuntu-template” 的 VM 應該會變成「執行中」狀態。

### <a name="install-the-necessary-software"></a>安裝必要軟體
#### <a name="step-1-upload-tarballs"></a>步驟 1：上傳 tarball
使用 scp 或 pscp，並使用下列命令格式將先前下載的軟體複製到 ~/downloads 目錄中：

##### <a name="pscp-server-jre-8u5-linux-x64targz-localadminhk-cas-templatecloudappnethomelocaladmindownloadsserver-jre-8u5-linux-x64targz"></a>pscp server-jre-8u5-linux-x64.tar.gz localadmin@hk-cas-template.cloudapp.net:/home/localadmin/downloads/server-jre-8u5-linux-x64.tar.gz
針對 JRE 和 Cassandra 重複上述命令。

#### <a name="step-2-prepare-the-directory-structure-and-extract-the-archives"></a>步驟 2：準備目錄結構並解壓縮封存
登入 VM、建立目錄結構，然後以進階使用者的身分使用下列 bash 指令碼解壓縮軟體：

    #!/bin/bash
    CASS_INSTALL_DIR="/opt/cassandra"
    JRE_INSTALL_DIR="/opt/java"
    CASS_DATA_DIR="/var/lib/cassandra"
    CASS_LOG_DIR="/var/log/cassandra"
    DOWNLOADS_DIR="~/downloads"
    JRE_TARBALL="server-jre-8u5-linux-x64.tar.gz"
    CASS_TARBALL="apache-cassandra-2.0.8-bin.tar.gz"
    SVC_USER="localadmin"

    RESET_ERROR=1
    MKDIR_ERROR=2

    reset_installation ()
    {
       rm -rf $CASS_INSTALL_DIR 2> /dev/null
       rm -rf $JRE_INSTALL_DIR 2> /dev/null
       rm -rf $CASS_DATA_DIR 2> /dev/null
       rm -rf $CASS_LOG_DIR 2> /dev/null
    }
    make_dir ()
    {
       if [ -z "$1" ]
       then
          echo "make_dir: invalid directory name"
          exit $MKDIR_ERROR
       fi

       if [ -d "$1" ]
       then
          echo "make_dir: directory already exists"
          exit $MKDIR_ERROR
       fi

       mkdir $1 2>/dev/null
       if [ $? != 0 ]
       then
          echo "directory creation failed"
          exit $MKDIR_ERROR
       fi
    }

    unzip()
    {
       if [ $# == 2 ]
       then
          tar xzf $1 -C $2
       else
          echo "archive error"
       fi

    }

    if [ -n "$1" ]
    then
       SVC_USER=$1
    fi

    reset_installation
    make_dir $CASS_INSTALL_DIR
    make_dir $JRE_INSTALL_DIR
    make_dir $CASS_DATA_DIR
    make_dir $CASS_LOG_DIR

    #unzip JRE and Cassandra
    unzip $HOME/downloads/$JRE_TARBALL $JRE_INSTALL_DIR
    unzip $HOME/downloads/$CASS_TARBALL $CASS_INSTALL_DIR

    #Change the ownership to the service credentials

    chown -R $SVC_USER:$GROUP $CASS_DATA_DIR
    chown -R $SVC_USER:$GROUP $CASS_LOG_DIR
    echo "edit /etc/profile to add JRE to the PATH"
    echo "installation is complete"


如果您將此指令碼貼到 vim 視窗，請務必使用下列命令移除歸位字元 (‘\r”)：

    tr -d '\r' <infile.sh >outfile.sh

#### <a name="step-3-edit-etcprofile"></a>步驟 3：編輯 etc/profile
在結尾附加下列內容：

    JAVA_HOME=/opt/java/jdk1.8.0_05
    CASS_HOME= /opt/cassandra/apache-cassandra-2.0.8
    PATH=$PATH:$HOME/bin:$JAVA_HOME/bin:$CASS_HOME/bin
    export JAVA_HOME
    export CASS_HOME
    export PATH

#### <a name="step-4-install-jna-for-production-systems"></a>步驟 4：安裝適用於實際執行系統的 JNA
使用下列命令序列：下列命令會將 jna-3.2.7.jar 和 jna-platform-3.2.7.jar 安裝到 /usr/share.java 目錄：sudo apt-get install libjna-java

在 $CASS_HOME/lib 目錄中建立符號連結，以便 Cassandra 啟動指令碼可以找到這些 jar：

    ln -s /usr/share/java/jna-3.2.7.jar $CASS_HOME/lib/jna.jar

    ln -s /usr/share/java/jna-platform-3.2.7.jar $CASS_HOME/lib/jna-platform.jar

#### <a name="step-5-configure-cassandrayaml"></a>步驟 5：設定 cassandra.yaml
編輯每個 VM 的 cassandra.yaml 以反映所有虛擬機器所需的設定，[您將在實際佈建期間調校此設定]：

<table>
<tr><th>欄位名稱   </th><th> 值  </th><th>    備註 </th></tr>
<tr><td>cluster_name </td><td>    “CustomerService”    </td><td> 使用能反映您部署的名稱</td></tr>
<tr><td>listen_address    </td><td>[保留為空白]    </td><td> 刪除 “localhost” </td></tr>
<tr><td>rpc_addres   </td><td>[保留為空白]    </td><td> 刪除 “localhost” </td></tr>
<tr><td>種子    </td><td>"10.1.2.4、10.1.2.6、10.1.2.8"    </td><td>指定為種子的所有 IP 位址清單。</td></tr>
<tr><td>endpoint_snitch </td><td> org.apache.cassandra.locator.GossipingPropertyFileSnitch </td><td> 由 NetworkTopologyStrateg 使用來表示資料中心和 VM 的機架</td></tr>
</table>

#### <a name="step-6-capture-the-vm-image"></a>步驟 6：擷取 VM 映像
使用先前建立的主機名稱 (hk-ca-template.cloudapp.net) 和 SSH 私密金鑰登入虛擬機器。 請參閱「如何在 Azure 上使用 SSH 搭配 Linux」了解如何使用命令 ssh 或 putty.exe 登入的詳細資訊。

執行下列動作擷取映像：

##### <a name="1-deprovision"></a>1.取消佈建
使用命令 “sudo waagent –deprovision+user” 移除虛擬機器執行個體的特定資訊。 請參閱 [如何擷取 Linux 虛擬機器作為範本使用](capture-image-classic.md) ，以了解映像擷取程序的詳細資訊。

##### <a name="2-shut-down-the-vm"></a>2：關閉 VM
確定已反白顯示虛擬機器，然後按一下底部命令列中的 [關機] 連結。

##### <a name="3-capture-the-image"></a>3：擷取映像
確定已反白顯示虛擬機器，然後按一下底部命令列中的 [擷取] 連結。 在下一個畫面中，指定 [映像名稱] \(例如 hk-cas-2-08-ub-14-04-2014071)、適當的 [映像描述]，然後按一下「確認」記號以完成擷取程序。

此程序需要幾秒鐘的時間，然後您應該就可以在映像庫的 [我的映像] 區段中找到映像。 成功擷取映像之後，來源 VM 就會自動刪除。 

## <a name="single-region-deployment-process"></a>單一區域部署程序
**步驟 1：建立虛擬網路** 登入 Azure 入口網站，並使用下表中顯示的屬性，建立虛擬網路 (傳統)。 如需程序的詳細步驟，請參閱[建立使用 Azure 入口網站的虛擬網路 (傳統)](../../../virtual-network/virtual-networks-create-vnet-classic-pportal.md)。      

<table>
<tr><th>VM 屬性名稱</th><th>值</th><th>備註</th></tr>
<tr><td>Name</td><td>vnet-cass-west-us</td><td></td></tr>
<tr><td>區域</td><td>美國西部</td><td></td></tr>
<tr><td>DNS 伺服器</td><td>None</td><td>請忽略此項，因為我們不使用 DNS 伺服器</td></tr>
<tr><td>位址空間</td><td>10.1.0.0/16</td><td></td></tr>    
<tr><td>起始 IP</td><td>10.1.0.0</td><td></td></tr>    
<tr><td>CIDR </td><td>/16 (65531)</td><td></td></tr>
</table>

新增下列子網路：

<table>
<tr><th>Name</th><th>起始 IP</th><th>CIDR</th><th>備註</th></tr>
<tr><td>Web</td><td>10.1.1.0</td><td>/24 (251)</td><td>Web 伺服陣列的子網路</td></tr>
<tr><td>data</td><td>10.1.2.0</td><td>/24 (251)</td><td>資料庫節點的子網路</td></tr>
</table>

透過網路安全性群組可保護「資料」與 Web 子網路，但這已超出本文的討論範圍。  

**步驟 2：佈建虛擬網路** 使用先前建立的映像，您將在雲端伺服器 “hk-c-svc-west” 中建立下列虛擬機器，並將它們繫結到個別的子網路，如下所示：

<table>
<tr><th>機器名稱    </th><th>子網路    </th><th>IP 位址    </th><th>可用性設定組</th><th>DC/機架</th><th>是否為種子？</th></tr>
<tr><td>hk-c1-west-us    </td><td>data    </td><td>10.1.2.4    </td><td>hk-c-aset-1    </td><td>dc=WESTUS 機架=rack1 </td><td>yes</td></tr>
<tr><td>hk-c2-west-us    </td><td>data    </td><td>10.1.2.5    </td><td>hk-c-aset-1    </td><td>dc=WESTUS 機架=rack1    </td><td>否 </td></tr>
<tr><td>hk-c3-west-us    </td><td>data    </td><td>10.1.2.6    </td><td>hk-c-aset-1    </td><td>dc=WESTUS 機架=rack2    </td><td>yes</td></tr>
<tr><td>hk-c4-west-us    </td><td>data    </td><td>10.1.2.7    </td><td>hk-c-aset-1    </td><td>dc=WESTUS 機架=rack2    </td><td>否 </td></tr>
<tr><td>hk-c5-west-us    </td><td>data    </td><td>10.1.2.8    </td><td>hk-c-aset-2    </td><td>dc=WESTUS 機架=rack3    </td><td>yes</td></tr>
<tr><td>hk-c6-west-us    </td><td>data    </td><td>10.1.2.9    </td><td>hk-c-aset-2    </td><td>dc=WESTUS 機架=rack3    </td><td>否 </td></tr>
<tr><td>hk-c7-west-us    </td><td>data    </td><td>10.1.2.10    </td><td>hk-c-aset-2    </td><td>dc=WESTUS 機架=rack4    </td><td>yes</td></tr>
<tr><td>hk-c8-west-us    </td><td>data    </td><td>10.1.2.11    </td><td>hk-c-aset-2    </td><td>dc=WESTUS 機架=rack4    </td><td>否 </td></tr>
<tr><td>hk-w1-west-us    </td><td>Web    </td><td>10.1.1.4    </td><td>hk-w-aset-1    </td><td>                       </td><td>N/A</td></tr>
<tr><td>hk-w2-west-us    </td><td>Web    </td><td>10.1.1.5    </td><td>hk-w-aset-1    </td><td>                       </td><td>N/A</td></tr>
</table>

建立上述 VM 清單需要下列程序：

1. 在特定區域中建立空的雲端服務
2. 從先前擷取的映像建立 VM，並將它附加到先前建立的虛擬網路，然後對所有 VM 重複此步驟
3. 將內部負載平衡器加入雲端服務，並將它附加到「資料」子網路
4. 對先前建立的每個 VM，透過連線到先前建立之內部負載平衡器的負載平衡集，新增 thrift 流量的負載平衡端點

您可以使用 Azure 入口網站執行上述程序；使用 Windows 電腦 (如果您沒有 Windows 電腦的存取權，則使用 Azure 上的 VM)，使用下列 PowerShell 指令碼自動佈建所有 8 個 VM。

**清單 1：佈建虛擬機器的 PowerShell 指令碼**

        #Tested with Azure Powershell - November 2014
        #This powershell script deployes a number of VMs from an existing image inside an Azure region
        #Import your Azure subscription into the current Powershell session before proceeding
        #The process: 1. create Azure Storage account, 2. create virtual network, 3.create the VM template, 2. create a list of VMs from the template

        #fundamental variables - change these to reflect your subscription
        $country="us"; $region="west"; $vnetName = "your_vnet_name";$storageAccount="your_storage_account"
        $numVMs=8;$prefix = "hk-cass";$ilbIP="your_ilb_ip"
        $subscriptionName = "Azure_subscription_name";
        $vmSize="ExtraSmall"; $imageName="your_linux_image_name"
        $ilbName="ThriftInternalLB"; $thriftEndPoint="ThriftEndPoint"

        #generated variables
        $serviceName = "$prefix-svc-$region-$country"; $azureRegion = "$region $country"

        $vmNames = @()
        for ($i=0; $i -lt $numVMs; $i++)
        {
           $vmNames+=("$prefix-vm"+($i+1) + "-$region-$country" );
        }

        #select an Azure subscription already imported into Powershell session
        Select-AzureSubscription -SubscriptionName $subscriptionName -Current
        Set-AzureSubscription -SubscriptionName $subscriptionName -CurrentStorageAccountName $storageAccount

        #create an empty cloud service
        New-AzureService -ServiceName $serviceName -Label "hkcass$region" -Location $azureRegion
        Write-Host "Created $serviceName"

        $VMList= @()   # stores the list of azure vm configuration objects
        #create the list of VMs
        foreach($vmName in $vmNames)
        {
           $VMList += New-AzureVMConfig -Name $vmName -InstanceSize ExtraSmall -ImageName $imageName |
           Add-AzureProvisioningConfig -Linux -LinuxUser "localadmin" -Password "Local123" |
           Set-AzureSubnet "data"
        }

        New-AzureVM -ServiceName $serviceName -VNetName $vnetName -VMs $VMList

        #Create internal load balancer
        Add-AzureInternalLoadBalancer -ServiceName $serviceName -InternalLoadBalancerName $ilbName -SubnetName "data" -StaticVNetIPAddress "$ilbIP"
        Write-Host "Created $ilbName"
        #Add add the thrift endpoint to the internal load balancer for all the VMs
        foreach($vmName in $vmNames)
        {
            Get-AzureVM -ServiceName $serviceName -Name $vmName |
                Add-AzureEndpoint -Name $thriftEndPoint -LBSetName "ThriftLBSet" -Protocol tcp -LocalPort 9160 -PublicPort 9160 -ProbePort 9160 -ProbeProtocol tcp -ProbeIntervalInSeconds 10 -InternalLoadBalancerName $ilbName |
                Update-AzureVM

            Write-Host "created $vmName"     
        }

**步驟 3：在每個 VM 上設定 Cassandra**

登入 VM 並執行下列項目：

* 編輯 $CASS_HOME/conf/cassandra-rackdc.properties 指定資料中心和機架內容：
  
       dc =EASTUS, rack =rack1
* 編輯 cassandra.yaml 設定種子節點，如下所示：
  
       Seeds: "10.1.2.4,10.1.2.6,10.1.2.8,10.1.2.10"

**步驟 4：啟動 VM，然後測試叢集**

登入其中一個節點 (例如 hk-c1-west-us)，然後執行下列命令查看叢集的狀態：

       nodetool –h 10.1.2.4 –p 7199 status

您應該會看到類似下面 8 節點叢集的畫面：

<table>
<tr><th>狀態</th><th>位址    </th><th>載入    </th><th>權杖    </th><th>擁有 </th><th>主機識別碼    </th><th>機架</th></tr>
<tr><th>UN    </td><td>10.1.2.4     </td><td>87.81 KB    </td><td>256    </td><td>38.0%    </td><td>Guid (已移除)</td><td>rack1</td></tr>
<tr><th>UN    </td><td>10.1.2.5     </td><td>41.08 KB    </td><td>256    </td><td>68.9%    </td><td>Guid (已移除)</td><td>rack1</td></tr>
<tr><th>UN    </td><td>10.1.2.6     </td><td>55.29 KB    </td><td>256    </td><td>68.8%    </td><td>Guid (已移除)</td><td>rack2</td></tr>
<tr><th>UN    </td><td>10.1.2.7     </td><td>55.29 KB    </td><td>256    </td><td>68.8%    </td><td>Guid (已移除)</td><td>rack2</td></tr>
<tr><th>UN    </td><td>10.1.2.8     </td><td>55.29 KB    </td><td>256    </td><td>68.8%    </td><td>Guid (已移除)</td><td>rack3</td></tr>
<tr><th>UN    </td><td>10.1.2.9     </td><td>55.29 KB    </td><td>256    </td><td>68.8%    </td><td>Guid (已移除)</td><td>rack3</td></tr>
<tr><th>UN    </td><td>10.1.2.10     </td><td>55.29 KB    </td><td>256    </td><td>68.8%    </td><td>Guid (已移除)</td><td>rack4</td></tr>
<tr><th>UN    </td><td>10.1.2.11     </td><td>55.29 KB    </td><td>256    </td><td>68.8%    </td><td>Guid (已移除)</td><td>rack4</td></tr>
</table>

## <a name="test-the-single-region-cluster"></a>測試單一區域叢集
使用下列步驟測試叢集：

1. 使用 Powershell 命令 Get-AzureInternalLoadbalancer Cmdlet 取得內部負載平衡器的 IP 位址 (例如 10.1.2.101)。 命令的語法如下所示：Get-AzureLoadbalancer –ServiceName "hk-c-svc-west-us” [顯示內部負載平衡器以及其 IP 位址的詳細資訊]
2. 使用 Putty 或 ssh 登入 Web 伺服陣列 VM (例如 hk-w1-west-us)
3. 執行 $CASS_HOME/bin/cqlsh 10.1.2.101 9160
4. 使用下列 CQL 命令來確認叢集是否正常運作：
   
     CREATE KEYSPACE customers_ks WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : 3 };   USE customers_ks;   CREATE TABLE Customers(customer_id int PRIMARY KEY, firstname text, lastname text);   INSERT INTO Customers(customer_id, firstname, lastname) VALUES(1, 'John', 'Doe');   INSERT INTO Customers(customer_id, firstname, lastname) VALUES (2, 'Jane', 'Doe');
   
     SELECT * FROM Customers;

您應該會看到如下列結果的內容：

<table>
  <tr><th> customer_id </th><th> firstname </th><th> lastname </th></tr>
  <tr><td> 1 </td><td> John </td><td> Doe </td></tr>
  <tr><td> 2 </td><td> Jane </td><td> Doe </td></tr>
</table>

在步驟 4 中建立的 keyspace 使用 replication_factor 為 3 的 SimpleStrategy。 如果是單一資料中心部署，建議使用 SimpleStrategy，如果是多重資料中心部署，則建議使用 NetworkTopologyStrategy。 replication_factor 為 3 會針對節點失敗提供容錯。

## <a id="tworegion"> </a>多區域部署程序
您將運用已經完成的單一區域部署，然後重複相同程序來安裝第二個區域。 單一和多重區域部署之間的主要差異在於區域間通訊的 VPN 通道設定；您將從網路安裝開始、佈建 VM 以及設定 Cassandra。

### <a name="step-1-create-the-virtual-network-at-the-2nd-region"></a>步驟 1：在第 2 個區域建立虛擬網路
登入 Azure 入口網站，並使用表格中所示的屬性來建立虛擬網路。 如需此程序的詳細步驟，請參閱[在 Azure 入口網站中設定純雲端虛擬網路](../../../virtual-network/virtual-networks-create-vnet-classic-pportal.md)。      

<table>
<tr><th>屬性名稱    </th><th>值    </th><th>備註</th></tr>
<tr><td>Name    </td><td>vnet-cass-east-us</td><td></td></tr>
<tr><td>區域    </td><td>美國東部</td><td></td></tr>
<tr><td>DNS 伺服器        </td><td></td><td>請忽略此項，因為我們不使用 DNS 伺服器</td></tr>
<tr><td>設定點對站 VPN</td><td></td><td>        略過此項</td></tr>
<tr><td>設定站台對站台 VPN</td><td></td><td>        略過此項</td></tr>
<tr><td>位址空間    </td><td>10.2.0.0/16</td><td></td></tr>
<tr><td>起始 IP    </td><td>10.2.0.0    </td><td></td></tr>
<tr><td>CIDR    </td><td>/16 (65531)</td><td></td></tr>
</table>

新增下列子網路：

<table>
<tr><th>Name    </th><th>起始 IP    </th><th>CIDR    </th><th>備註</th></tr>
<tr><td>Web    </td><td>10.2.1.0    </td><td>/24 (251)    </td><td>Web 伺服陣列的子網路</td></tr>
<tr><td>data    </td><td>10.2.2.0    </td><td>/24 (251)    </td><td>資料庫節點的子網路</td></tr>
</table>


### <a name="step-2-create-local-networks"></a>步驟 2：建立區域網路
Azure 虛擬網路功能中的「區域網路」是一個 Proxy 位址空間，對應至包括私人雲端或其他 Azure 區域的遠端站台。 此 Proxy 位址空間繫結至一個遠端閘道，將網路路由到正確的網路功能目的地。 請參閱 [設定 VNet 對 VNet 連線](../../../vpn-gateway/virtual-networks-configure-vnet-to-vnet-connection.md) ，以取得建立 VNET 對 VNET 連線的指示。

根據下列詳細資料建立兩個區域網路：

| 網路名稱 | VPN 閘道位址 | 位址空間 | 備註 |
| --- | --- | --- | --- |
| hk-lnet-map-to-east-us |23.1.1.1 |10.2.0.0/16 |建立區域網路時，提供預留位置閘道位址。 建立閘道之後，就會填入真實的閘道位址。 請確定位址空間完全符合相對的遠端 VNET。在此案例中是在美國東部地區建立的 VNET。 |
| hk-lnet-map-to-west-us |23.2.2.2 |10.1.0.0/16 |建立區域網路時，提供預留位置閘道位址。 建立閘道之後，就會填入真實的閘道位址。 請確定位址空間完全符合相對的遠端 VNET。在此案例中是在美國西部地區建立的 VNET。 |

### <a name="step-3-map-local-network-to-the-respective-vnets"></a>步驟 3：將「區域」網路對應至個別 VNET
從 Azure 入口網站中，選取每個 VNet、按一下 [設定]，勾選 [連線到區域網路]，並根據下列詳細資料選取區域網路：

| 虛擬網路 | 區域網路 |
| --- | --- |
| hk-vnet-west-us |hk-lnet-map-to-east-us |
| hk-vnet-east-us |hk-lnet-map-to-west-us |

### <a name="step-4-create-gateways-on-vnet1-and-vnet2"></a>步驟 4：在 VNET1 和 VNET2 上建立閘道
從兩個虛擬網路的儀表板，按一下 [建立閘道] 可觸發 VPN 閘道的佈建程序。 請稍候幾分鐘，每個虛擬網路的儀表板應該會顯示實際的閘道位址。

### <a name="step-5-update-local-networks-with-the-respective-gateway-addresses"></a>步驟 5：更新「區域」網路的個別「閘道」位址
編輯兩個區域網路，使用剛才佈建的閘道實際 IP 位址取代預留位置閘道 IP 位址。 使用下列對應：

<table>
<tr><th>區域網路    </th><th>虛擬網路閘道</th></tr>
<tr><td>hk-lnet-map-to-east-us </td><td>hk-vnet-west-us 的閘道</td></tr>
<tr><td>hk-lnet-map-to-west-us </td><td>hk-vnet-east-us 的閘道</td></tr>
</table>

### <a name="step-6-update-the-shared-key"></a>步驟 6：更新共用金鑰
使用下列 Powershell 指令碼來更新每個 VPN 閘道的 IPSec 金鑰 [針對這兩個閘道使用目的金鑰]：Set-AzureVNetGatewayKey -VNetName hk-vnet-east-us -LocalNetworkSiteName hk-lnet-map-to-west-us -SharedKey D9E76BKK Set-AzureVNetGatewayKey -VNetName hk-vnet-west-us -LocalNetworkSiteName hk-lnet-map-to-east-us -SharedKey D9E76BKK

### <a name="step-7-establish-the-vnet-to-vnet-connection"></a>步驟 7：建立 VNET 對 VNET 連線
從 Azure 入口網站中，使用兩個虛擬網路的 [儀表板] 功能表建立閘道對閘道的連接。 使用底部工具列的 [連線] 功能表項目。 請稍候幾分鐘，儀表板應該會以圖形方式顯示連線詳細資料。

### <a name="step-8-create-the-virtual-machines-in-region-2"></a>步驟 8：在第 2 個區域中建立虛擬機器
依照在第 1 個區域中部署所述的下列相同步驟，建立 Ubuntu 映像，或者將映像 VHD 檔案複製到位於第 2 個區域的 Azure 儲存體帳戶，然後建立映像。 使用此映像，並在新的雲端服務 hk-c-svc-east-us 建立下列虛擬機器清單：

| 機器名稱 | 子網路 | IP 位址 | 可用性設定組 | DC/機架 | 是否為種子？ |
| --- | --- | --- | --- | --- | --- |
| hk-c1-east-us |data |10.2.2.4 |hk-c-aset-1 |dc=EASTUS 機架=rack1 |yes |
| hk-c2-east-us |data |10.2.2.5 |hk-c-aset-1 |dc=EASTUS 機架=rack1 |否 |
| hk-c3-east-us |data |10.2.2.6 |hk-c-aset-1 |dc=EASTUS 機架=rack2 |yes |
| hk-c5-east-us |data |10.2.2.8 |hk-c-aset-2 |dc=EASTUS 機架=rack3 |yes |
| hk-c6-east-us |data |10.2.2.9 |hk-c-aset-2 |dc=EASTUS 機架=rack3 |否 |
| hk-c7-east-us |data |10.2.2.10 |hk-c-aset-2 |dc=EASTUS 機架=rack4 |yes |
| hk-c8-east-us |data |10.2.2.11 |hk-c-aset-2 |dc=EASTUS 機架=rack4 |否 |
| hk-w1-east-us |Web |10.2.1.4 |hk-w-aset-1 |N/A |N/A |
| hk-w2-east-us |Web |10.2.1.5 |hk-w-aset-1 |N/A |N/A |

依照與第 1 個區域相同的指示，但是使用 10.2.xxx.xxx 位址空間。

### <a name="step-9-configure-cassandra-on-each-vm"></a>步驟 9：在每個 VM 上設定 Cassandra
登入 VM 並執行下列項目：

1. 編輯 $CASS_HOME/conf/cassandra-rackdc.properties，以下列格式指定資料中心和機架內容：dc =EASTUS rack =rack1
2. 編輯 cassandra.yaml 設定種子節點：種子："10.1.2.4,10.1.2.6,10.1.2.8,10.1.2.10,10.2.2.4,10.2.2.6,10.2.2.8,10.2.2.10"

### <a name="step-10-start-cassandra"></a>步驟 10：啟動 Cassandra
登入每個 VM，執行下列命令在背景啟動 Cassandra：$CASS_HOME/bin/cassandra

## <a name="test-the-multi-region-cluster"></a>測試多重區域叢集
現在 Cassandra 已部署 16 個節點，每個 Azure 區域中有 8 個節點。 這些節點如果有共通的叢集名稱和種子節點設定，就是位於相同的叢集中。 使用下列程序測試叢集：

### <a name="step-1-get-the-internal-load-balancer-ip-for-both-the-regions-using-powershell"></a>步驟 1：使用 PowerShell 取得這兩個區域的內部負載平衡器 IP
* Get-AzureInternalLoadbalancer -ServiceName "hk-c-svc-west-us"
* Get-AzureInternalLoadbalancer -ServiceName "hk-c-svc-east-us"  
  
    請注意顯示的 IP 位址 (例如西部 - 10.1.2.101，東部 - 10.2.2.101)。

### <a name="step-2-execute-the-following-in-the-west-region-after-logging-into-hk-w1-west-us"></a>步驟 2：登入 hk-w1-west-us 後，在西部區域執行下列命令
1. 執行 $CASS_HOME/bin/cqlsh 10.1.2.101 9160
2. 執行下列 CQL 命令：
   
     CREATE KEYSPACE customers_ks   WITH REPLICATION = { 'class' : 'NetworkToplogyStrategy', 'WESTUS' : 3, 'EASTUS' : 3};   USE customers_ks;   CREATE TABLE Customers(customer_id int PRIMARY KEY, firstname text, lastname text);   INSERT INTO Customers(customer_id, firstname, lastname) VALUES(1, 'John', 'Doe');   INSERT INTO Customers(customer_id, firstname, lastname) VALUES (2, 'Jane', 'Doe');   SELECT * FROM Customers;

您應該會看到如下顯示：

| customer_id | firstname | lastname |
| --- | --- | --- |
| 1 |John |Doe |
| 2 |Jane |Doe |

### <a name="step-3-execute-the-following-in-the-east-region-after-logging-into-hk-w1-east-us"></a>步驟 3：登入 hk-w1-east-us 後，在東部區域執行下列命令
1. 執行 $CASS_HOME/bin/cqlsh 10.2.2.101 9160
2. 執行下列 CQL 命令：
   
     USE customers_ks;   CREATE TABLE Customers(customer_id int PRIMARY KEY, firstname text, lastname text);   INSERT INTO Customers(customer_id, firstname, lastname) VALUES(1, 'John', 'Doe');   INSERT INTO Customers(customer_id, firstname, lastname) VALUES (2, 'Jane', 'Doe');   SELECT * FROM Customers;

您應該會看到針對西部區域執行時相同的顯示畫面：

| customer_id | firstname | lastname |
| --- | --- | --- |
| 1 |John |Doe |
| 2 |Jane |Doe |

執行一些插入動作，並且查看複寫到叢集 west-us 部分的那些插入項目。

## <a name="test-cassandra-cluster-from-nodejs"></a>透過 Node.js 測試 Cassandra 叢集
使用先前在 "Web" 層中建立的其中一個 Linux VM，您將執行簡單的 Node.js 指令碼來讀取先前插入的資料

**步驟 1：安裝 Node.js 和 Cassandra 用戶端**

1. 安裝 Node.js 和 npm
2. 使用 npm 安裝節點套件"cassandra-client"
3. 在殼層提示字元中執行下列指令碼，顯示擷取資料的 json 字串：
   
        var pooledCon = require('cassandra-client').PooledConnection;
        var ksName = "custsupport_ks";
        var cfName = "customers_cf";
        var hostList = ['internal_loadbalancer_ip:9160'];
        var ksConOptions = { hosts: hostList,
                             keyspace: ksName, use_bigints: false };
   
        function createKeyspace(callback){
           var cql = 'CREATE KEYSPACE ' + ksName + ' WITH strategy_class=SimpleStrategy AND strategy_options:replication_factor=1';
           var sysConOptions = { hosts: hostList,  
                                 keyspace: 'system', use_bigints: false };
           var con = new pooledCon(sysConOptions);
           con.execute(cql,[],function(err) {
           if (err) {
             console.log("Failed to create Keyspace: " + ksName);
             console.log(err);
           }
           else {
             console.log("Created Keyspace: " + ksName);
             callback(ksConOptions, populateCustomerData);
           }
           });
           con.shutdown();
        }
   
        function createColumnFamily(ksConOptions, callback){
          var params = ['customers_cf','custid','varint','custname',
                        'text','custaddress','text'];
          var cql = 'CREATE COLUMNFAMILY ? (? ? PRIMARY KEY,? ?, ? ?)';
        var con =  new pooledCon(ksConOptions);
          con.execute(cql,params,function(err) {
              if (err) {
                 console.log("Failed to create column family: " + params[0]);
                 console.log(err);
              }
              else {
                 console.log("Created column family: " + params[0]);
                 callback();
              }
          });
          con.shutdown();
        }
   
        //populate Data
        function populateCustomerData() {
           var params = ['John','Infinity Dr, TX', 1];
           updateCustomer(ksConOptions,params);
   
           params = ['Tom','Fermat Ln, WA', 2];
           updateCustomer(ksConOptions,params);
        }
   
        //update also inserts the record if none exists
        function updateCustomer(ksConOptions,params)
        {
          var cql = 'UPDATE customers_cf SET custname=?,custaddress=? where custid=?';
          var con = new pooledCon(ksConOptions);
          con.execute(cql,params,function(err) {
              if (err) console.log(err);
              else console.log("Inserted customer : " + params[0]);
          });
          con.shutdown();
        }
   
        //read the two rows inserted above
        function readCustomer(ksConOptions)
        {
          var cql = 'SELECT * FROM customers_cf WHERE custid IN (1,2)';
          var con = new pooledCon(ksConOptions);
          con.execute(cql,[],function(err,rows) {
              if (err)
                 console.log(err);
              else
                 for (var i=0; i<rows.length; i++)
                    console.log(JSON.stringify(rows[i]));
            });
           con.shutdown();
        }
   
        //exectue the code
        createKeyspace(createColumnFamily);
        readCustomer(ksConOptions)

## <a name="conclusion"></a>結論
Microsoft Azure 是一個富彈性的平台，可以執行 Microsoft 與開放原始碼軟體，如本練習中所示。 透過將叢集節點分散在多個容錯網域，可以將高度可用的 Cassandra 叢集部署在單一資料中心。 您也可以跨越多個地理位置遙遠的 Azure 區域部署 Cassandra 叢集做為災害防禦系統。 Azure 搭配 Cassandra 可建構現今的網際網路等級服務所需，且具有高擴充性、高可用性以及可進行災害復原的雲端服務。  

## <a name="references"></a>參考
* [http://cassandra.apache.org](http://cassandra.apache.org)
* [http://www.datastax.com](http://www.datastax.com)
* [http://www.nodejs.org](http://www.nodejs.org)

