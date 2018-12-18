---
title: 使用 Azure Migrate 探索及評估要移轉到 Azure 的內部部署 VMware VM | Microsoft Docs
description: 說明如何使用 Azure Migrate 服務，探索及評估要移轉到 Azure 的內部部署 VMware VM。
author: rayne-wiselman
ms.service: azure-migrate
ms.topic: tutorial
ms.date: 09/21/2018
ms.author: raynew
ms.custom: mvc
ms.openlocfilehash: b2bb6636aef9e26a81988d344f04f23c23ea1622
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47161874"
---
# <a name="discover-and-assess-on-premises-vmware-vms-for-migration-to-azure"></a>探索及評估要移轉到 Azure 的內部部署 VMware VM

[Azure Migrate](migrate-overview.md) 服務會評估要移轉至 Azure 的內部部署工作負載。

在此教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立 Azure Migrate 用來探索內部部署 VM 的帳戶
> * 建立 Azure Migrate 專案。
> * 設定內部部署收集器虛擬機器 (VM)，以探索要評估的內部部署 VMware VM。
> * 將 VM 分組並建立評估。


如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。


## <a name="prerequisites"></a>先決條件

- **VMWare**：您計劃移轉的虛擬機器必須透過執行版本 5.5、6.0 或 6.5 的 vCenter Server 來管理。 此外，您還需要一部執行版本 5.0 或更新版本的 ESXi 主機來部署收集器 VM。
- **vCenter Server 帳戶**：您需要一個唯讀帳戶來存取 vCenter Server。 Azure Migrate 會使用此帳戶來探索內部部署 VM。
- **權限**：在 vCenter Server 上，您需要權限，才能透過匯入 .OVA 格式的檔案來建立 VM。
- **統計資料設定**：此先決條件僅適用於單次探索模型。 若要使單次探索能夠運作，vCenter Server 的統計資料設定應該先設為層級 3，再開始部署。 如果低於層級 3，還是能進行評估，但不會收集儲存體和網路的效能資料。 在此情況下，將根據 CPU 和記憶體的效能資料以及磁碟和網路介面卡的組態資料來提出大小建議。

## <a name="create-an-account-for-vm-discovery"></a>建立用於 VM 探索的帳戶

Azure Migrate 需要存取 VMware 伺服器，才能自動探索 VM 以進行評量。 建立具有下列屬性的 VMware 帳戶。 您會在 Azure Migrate 設定過程中指定此帳戶。

- 使用者類型：至少是唯讀使用者
- 權限：資料中心物件 –> 傳播至子物件、角色=唯讀
- 詳細資訊：在資料中心層級指派的使用者，且能夠存取資料中心內的所有物件。
- 如果要限制存取權，請將具備 [傳播至子物件] 權限的 [沒有存取權] 角色指派給子物件 (vSphere 主機、資料存放區、VM 與網路)。


## <a name="sign-in-to-the-azure-portal"></a>登入 Azure 入口網站

登入 [Azure 入口網站](https://portal.azure.com)。

## <a name="create-a-project"></a>建立專案

1. 在 Azure 入口網站中，按一下 [建立資源]。
2. 搜尋 **Azure Migrate**，然後在搜尋結果中選取 [Azure Migrate] 服務。 接著，按一下 [建立]。
3. 指定專案名稱，以及專案的 Azure 訂用帳戶。
4. 建立新的資源群組。
5. 指定要建立專案的地理位置，然後按一下 [建立]。 您只能在美國地理位置建立 Azure Migrate 專案。 不過，您仍能針對任何目標 Azure 位置規劃移轉。 針對專案所指定的地理位置，僅會用於儲存從內部部署虛擬機器收集的中繼資料。

    ![Azure Migrate](./media/tutorial-assessment-vmware/project-1.png)


## <a name="download-the-collector-appliance"></a>下載收集器設備

Azure Migrate 會建立稱為「收集器設備」的內部部署 VM。 此 VM 會探索內部部署 VMware VM，並將其相關中繼資料傳送至 Azure Migrate 服務。 若要設定收集器設備，您可以下載 .OVA 檔案，並將它匯入內部部署 vCenter Server，以建立 VM。

1. 在 Azure Migrate 專案中，按一下 [開始使用] > [探索及評定] > [探索電腦]。
2. 在 [探索電腦] 中，有兩個選項可供設備使用。按一下 [下載] 以根據您的需求下載適當的設備。

    a. **單次探索**：此模型的設備會與 vCenter Server 通訊，以收集 VM 的相關中繼資料。 針對 VM 的效能資料收集，它會仰賴儲存在 vCenter Server 中的歷史效能資料，並收集上一個月的效能歷程記錄。 在此模型中，Azure Migrate 會收集每個計量的平均計數器 (相對於尖峰計數器)，[深入了解] (https://docs.microsoft.com/azure/migrate/concepts-collector#what-data-is-collected)。 由於它是單次探索，因此在探索之後將不會繼續反映出內部部署環境中的變更。 如果您想要反映變更，便必須針對相同的專案重新進行相同環境的探索。

    b. **連續探索**：此模型的設備會持續剖析內部部署環境，以收集每個 VM 的即時使用量資料。 在此模型中，會收集每個計量 (CPU 使用量、記憶體使用量等) 的尖峰計數器。 此模型並不需仰賴 vCenter Server 的統計資料設定來收集效能資料。 您可以隨時從設備停止持續性的剖析。

    > [!NOTE]
    > 連續探索功能目前處於預覽狀態。

3. 在 [複製專案認證] 中，複製專案識別碼和金鑰。 在設定收集器時，您會需要這些資料。

    ![下載 .ova 檔案](./media/tutorial-assessment-vmware/download-ova.png)

### <a name="verify-the-collector-appliance"></a>確認收集器設備

請先確認 .OVA 檔案是安全的，再進行部署。

1. 在存放下載檔案的目標電腦上，開啟系統管理員命令視窗。
2. 執行下列命令以產生 OVA 的雜湊：
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - 使用方式範例：```C:\>CertUtil -HashFile C:\AzureMigrate\AzureMigrate.ova SHA256```
3. 產生的雜湊應符合這些設定。

#### <a name="one-time-discovery"></a>單次探索

  OVA 1.0.9.14 版

  **演算法** | **雜湊值**
  --- | ---
  MD5 | 6d8446c0eeba3de3ecc9bc3713f9c8bd
  SHA1 | e9f5bdfdd1a746c11910ed917511b5d91b9f939f
  SHA256 | 7f7636d0959379502dfbda19b8e3f47f3a4744ee9453fc9ce548e6682a66f13c

  OVA 1.0.9.12 版

  **演算法** | **雜湊值**
  --- | ---
  MD5 | d0363e5d1b377a8eb08843cf034ac28a
  SHA1 | df4a0ada64bfa59c37acf521d15dcabe7f3f716b
  SHA256 | f677b6c255e3d4d529315a31b5947edfe46f45e4eb4dbc8019d68d1d1b337c2e

  OVA 1.0.9.8 版

  **演算法** | **雜湊值**
  --- | ---
  MD5 | b5d9f0caf15ca357ac0563468c2e6251
  SHA1 | d6179b5bfe84e123fabd37f8a1e4930839eeb0e5
  SHA256 | 09c68b168719cb93bd439ea6a5fe21a3b01beec0e15b84204857061ca5b116ff


  OVA 1.0.9.7 版

  **演算法** | **雜湊值**
  --- | ---
  MD5 | d5b6a03701203ff556fa78694d6d7c35
  SHA1 | f039feaa10dccd811c3d22d9a59fb83d0b01151e
  SHA256 | e5e997c003e29036f62bf3fdce96acd4a271799211a84b34b35dfd290e9bea9c

#### <a name="continuous-discovery"></a>連續探索

  OVA 1.0.10.4 版

  **演算法** | **雜湊值**
  --- | ---
  MD5 | 2ca5b1b93ee0675ca794dd3fd216e13d
  SHA1 | 8c46a52b18d36e91daeae62f412f5cb2a8198ee5
  SHA256 | 3b3dec0f995b3dd3c6ba218d436be003a687710abab9fcd17d4bdc90a11276be

## <a name="create-the-collector-vm"></a>建立收集器 VM

將下載的檔案匯入 vCenter Server。

1. 在 vSphere 用戶端主控台中，按一下 [檔案] > [部署 OVF 範本]。

    ![部署 OVF](./media/tutorial-assessment-vmware/vcenter-wizard.png)

2. 在 [部署 OVF 範本精靈] > [來源] 中，指定 .ova 檔案的位置。
3. 在 [名稱] 和 [位置] 中，指定收集器 VM 的易記名稱，以及要裝載 VM 的所在清查物件。
5. 在 [主機/叢集] 中，指定收集器 VM 的執行所在主機或叢集。
7. 在儲存體中，指定收集器 VM 的儲存目的地。
8. 在 [磁碟格式] 中，指定磁碟類型和大小。
9. 在 [網路對應] 中，指定收集器 VM 所要連線的網路。 此網路必須能夠連線到網際網路，以將中繼資料傳送至 Azure。
10. 檢閱並確認設定，然後按一下 [完成]。

## <a name="run-the-collector-to-discover-vms"></a>執行收集器來探索 VM

1. 在 vSphere 用戶端主控台中，以滑鼠右鍵按一下 [VM] > [開啟主控台]。
2. 提供設備的語言、時區和密碼喜好設定。
3. 在桌面上，按一下 [執行收集器] 捷徑。
4. 按一下收集器 UI 頂端列中的 [檢查更新]，並確認收集器正在最新版本上執行。 如果沒有，您可以選擇從連結下載最新的升級套件並更新收集器。
5. 在 Azure Migrate 收集器中，開啟 [設定必要條件]。
    - 接受授權條款，並閱讀第三方資訊。
    - 收集器會確認 VM 是否能夠存取網際網路。
    - 如果 VM 能夠透過 Proxy 存取網際網路，請按一下 [Proxy 設定]，然後指定 Proxy 位址和接聽連接埠。 如果 Proxy 需要驗證，請指定認證。 [深入了解](https://docs.microsoft.com/azure/migrate/concepts-collector#internet-connectivity)網際網路連線需求和收集器存取的 URL 清單。

    > [!NOTE]
    > 必須在表單 http://ProxyIPAddress 或 http://ProxyFQDN 中輸入 Proxy 位址。 僅支援 HTTP Proxy。 如果您有攔截 Proxy，且您未匯入 Proxy 憑證，則初始的網際網路連線可能會失敗；請[深入了解](https://docs.microsoft.com/azure/migrate/concepts-collector#internet-connectivity-with-intercepting-proxy)如何透過將 Proxy 憑證匯入作為收集器 VM 上的信任憑證，以解決此問題。

    - 收集器會檢查收集器服務是否正在執行。 根據預設，收集器 VM 上會安裝此服務。
    - 下載並安裝 VMware PowerCLI。

6. 在 [指定 vCenter Server 詳細資料] 中，執行下列動作：
    - 指定 vCenter Server 的名稱 (FQDN) 或 IP 位址。
    - 在 [使用者名稱] 和 [密碼] 中，指定收集器要用來探索 vCenter Server 上之 VM 的唯讀帳戶認證。
    - 在 [收集範圍] 中，選取 VM 探索的範圍。 收集器只能探索指定範圍內的 VM。 範圍可以設定為特定資料夾、資料中心或叢集。 它不應包含超過 1500 個 VM。 [深入了解](how-to-scale-assessment.md)如何探索較大的環境。

7. 在 [指定移轉專案] 中，指定您從入口網站所複製的 Azure Migrate 專案識別碼和金鑰。 如果您之前未複製，請從收集器 VM 開啟 Azure 入口網站。 在專案的 [概觀] 頁面中，按一下 [探索電腦]，然後複製值。  
8. 在 [檢視檢視收集進度] 中，監視探索狀態。 深入了解](https://docs.microsoft.com/azure/migrate/concepts-collector#what-data-is-collected) Azure Migrate 收集器會收集哪些資料。

> [!NOTE]
> 收集器只支援以「英文 (美國)」作為作業系統語言和收集器介面語言。
> 如果您對所要評估的機器變更設定，觸發程序將在您執行評估之前再次探索。 請在收集器中使用 [重新開始收集] 選項來執行此動作。 收集完成後，請在入口網站中選取 [重新計算] 選項進行評估，以取得更新的評估結果。



### <a name="verify-vms-in-the-portal"></a>在入口網站中確認 VM

針對單次探索，探索時間取決於您要探索的 VM 數目。 以 100 個 VM 作為例子，在收集器能停止執行之前，通常需要大約一小時的時間來完成設定及收集效能資料。 您可以在探索完成後立即建立評量 (同時包括以效能為基礎的評量，以及內部部署評量)。

針對連續探索 (處於預覽狀態)，收集器將會持續對內部部署環境進行剖析，並會持續以一小時為間隔傳送效能資料。 在開始探索的一小時後，您便可以在入口網站中檢閱電腦。 強烈建議您至少先等候一天的時間，再針對 VM 建立以效能為基礎的評量。

1. 在移轉專案中，按一下 [管理] > [機器]。
2. 確認您想要探索的 VM 出現在入口網站內。


## <a name="create-and-view-an-assessment"></a>建立和檢視評估

探索到 VM 之後，您可以將它們分組，並建立評估。

1. 在專案的 [概觀] 頁面中，按一下 [+建立評估]。
2. 按一下 [檢視全部] 來檢閱評估屬性。
3. 建立群組，並指定群組名稱。
4. 選取您想要新增至群組的機器。
5. 按一下 [建立評估] 以建立群組和評估。
6. 建立評估之後，在 [概觀] > [儀表板] 中檢視該評估。
7. 按一下 [匯出評估]，將其下載為 Excel 檔案。

> [!NOTE]
> 針對持續探索，強烈建議您在開始探索後，至少先等候一天的時間再建立評量。 如果您想要以最新的效能資料更新現有的評量，則可以對評量使用 [重新計算] 命令來更新它。

### <a name="assessment-details"></a>評量詳細資料

評量包含內部部署虛擬機器是否與 Azure 相容、在 Azure 中執行虛擬機器的正確虛擬機器大小，以及預估的 Azure 每月成本之相關資訊。

![評估報告](./media/tutorial-assessment-vmware/assessment-report.png)

#### <a name="azure-readiness"></a>Azure 移轉整備程度

評量中的 Azure 移轉整備程度檢視會顯示每部虛擬機器的整備狀態。 根據虛擬機器的屬性，每部虛擬機器可以標示為：
- 可供 Azure 使用
- 可有條件地供 Azure 使用
- 未備妥供 Azure 使用
- 移轉整備程度未知

對於已準備好的 VM，Azure Migrate 會建議這些 VM 在 Azure 中該有的大小。 由 Azure Migrate 進行的大小建議取決於評量屬性中指定的調整大小準則。 如果調整大小準則是以效能為基礎來調整，則會透過考量 VM (CPU 和記憶體) 和磁碟 (IOPS 和輸送量) 的效能歷程記錄來建議大小。 如果調整大小準則是「作為內部部署」，Azure Migrate 不會考慮 VM 和磁碟的效能資料。 透過查看 VM 內部部署的大小來建議 Azure 中的 VM 大小，而磁碟大小調整則是以評估屬性中指定的儲存體類型 (預設值為進階磁碟) 為基礎。 [深入了解](concepts-assessment-calculation.md)有關 Azure Migrate 調整大小的方式。

對於尚未就緒或者可有條件地供 Azure 使用的虛擬機器，Azure Migrate 會說明整備問題，並提供補救步驟。

對於 Azure Migrate 無法識別 Azure 移轉整備程度 (因為資料無法使用) 的虛擬機器會標示為移轉整備程度未知。

除了 Azure 移轉整備程度和大小調整，Azure Migrate 也會建議可供移轉虛擬機器使用的工具。 這需要在內部部署環境進行更深入的探索。 [深入探索](how-to-get-migration-tool.md)如何透過在內部部署電腦上安裝代理程式，以進行更深入的探索。 如果內部部署電腦上未安裝代理程式，則建議使用 [Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/site-recovery-overview) 進行原形移轉。 如果在內部部署電腦上安裝代理程式，則 Azure Migrate 會查看在電腦內部執行的程序，並識別電腦是否為資料庫電腦。 如果電腦是資料庫電腦，則建議使用 [Azure 資料庫移轉服務](https://docs.microsoft.com/azure/dms/dms-overview)，否則建議使用 Azure Site Recovery 作為移轉工具。

  ![評估整備](./media/tutorial-assessment-vmware/assessment-suitability.png)  

#### <a name="monthly-cost-estimate"></a>每月成本預估值

此檢視會顯示在 Azure 中執行 VM 的計算和儲存總成本，以及每部電腦的詳細資訊。 在計算成本估計值時，會考慮 Azure Migrate 針對一部電腦、其磁碟及評量屬性所做的大小建議。

> [!NOTE]
> Azure Migrate 所提供的成本估計適用於執行內部部署 VM 以作為 Azure 基礎結構即服務 (IaaS) VM。 Azure Migrate 不會考慮任何平台即服務 (PaaS) 或軟體即服務 (SaaS) 成本。

系統會彙總群組內所有 VM 之計算和儲存的每月預估成本。

![評估 VM 成本](./media/tutorial-assessment-vmware/assessment-vm-cost.png)

#### <a name="confidence-rating"></a>信賴評等

Azure Migrate 中的每個效能型評量會與信賴評等相關聯，信賴評等的範圍從 1 顆星到 5 顆星 (1 顆星最低，5 顆星最高)。 根據計算評量所需的資料點可用性，每個評量都會指派信賴評等。 評量的信賴評等可協助您評估 Azure Migrate 提供的大小建議之可靠性。 信賴評等不適用內部部署評估。

對於以效能為基礎的大小調整，Azure Migrate 需要 VM 的 CPU、記憶體使用量資料。 此外，對於連結至 VM 的每個磁碟，它都需要磁碟 IOPS 和輸送量資料。 同樣地，對於連接至虛擬機器的每個網路介面卡，Azure Migrate 需要輸入/輸出網路以進行以效能為基礎的大小調整。 如果上述的任何使用量數字在 vCenter Server 中無法取得，則 Azure Migrate 所完成的大小建議可能不可靠。 根據可用資料點的百分比提供評量的信賴評等如下：

   **資料點的可用性** | **信賴評等**
   --- | ---
   0%-20% | 1 顆星
   21%-40% | 2 顆星
   41%-60% | 3 顆星
   61%-80% | 4 顆星
   81%-100% | 5 顆星

由於下列原因，評量可能沒有所有可用資料點：

**單次探索**

- vCenter Server 中統計資料設定未設為等級 3。 由於單次探索模型仰賴 vCenter Server 的統計資料設定，如果 vCenter Server 中的統計資料設定低於層級 3，便不會從 vCenter Server 收集磁碟與網路的效能資料。 在此情況下，Azure Migrate 針對磁碟和網路所提供的建議是以使用率為基礎。 若不考量磁碟的 IOPS/輸送量，Azure Migrate 便無法識別磁碟在 Azure 中是否需要進階磁碟，因此在此情況下，Azure Migrate 會針對所有磁碟建議標準磁碟。
- vCenter Server 中的統計資料設定已設定為層級 3，以縮短探索開始前的持續時間。 例如，讓我們考慮以下案例：您今天將統計資料設定變更為等級 3，並在明天 (24 小時後) 使用收集器設備開始探索。 如果您要建立一天的評量，您可擁有所有資料點，且評量的信賴評等會是 5 顆星。 但如果您將評量屬性中的效能持續時間變更為一個月，信賴評等會關閉，因無法使用過去一個月的磁碟和網路效能資料。 如果您想考量過去一個月的效能資料，建議您將 vCenter Server 統計資料設定保留在等級 3 一個月，再開始進行探索。

**連續探索**

- 您未針對正在建立評量的持續時間剖析環境。 例如，如果您要建立的評量將效能持續時間設定為 1 天，您需要至少等待一天後再開始探索，才能收集到所有資料點。

**常見原因**  

- 少數虛擬機器在評量計算期間關閉。 如果有任何 VM 在某段期間內電源關閉，則無法收集這段時間的效能資料。
- 少數虛擬機器在評量計算期間才建立。 例如，如果您要建立過去一個月的效能記錄評量，但是少數虛擬機器在一週前才建立在環境中。 在這種情況下，新虛擬機器的效能記錄不會在整段期間中出現。

> [!NOTE]
> 如果任何評量的信賴評等低於 4 顆星，針對單次探索模型，建議您將 vCenter Server 的統計資料設定層級變更為 3，等候您要評量的持續時間 (1 天/1 週/1 個月)，然後再執行探索及評量。 針對連續探索模型，請至少等待一天讓設備剖析環境，然後對評量使用 [重新計算] 命令。 如果無法完成上述動作，則以效能為基礎的大小調整可能不可靠，且建議您變更評量屬性以切換至*內部部署大小調整*。

## <a name="next-steps"></a>後續步驟

- [深入了解](how-to-modify-assessment.md)如何根據您的需求自訂評估。
- 了解如何使用[機器相依性對應](how-to-create-group-machine-dependencies.md)來建立可信度高的評定群組
- [深入了解](concepts-assessment-calculation.md)評定的計算方式。
- [了解](how-to-scale-assessment.md)如何探索及評定大型 VMware 環境。
- [深入了解](resources-faq.md) Azure Migrate 常見問題集
