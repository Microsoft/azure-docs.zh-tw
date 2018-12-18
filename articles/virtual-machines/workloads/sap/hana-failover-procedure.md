---
title: Azure 上的 SAP HANA (大型執行個體) 至災害網站的 HANA 容錯移轉程序 | Microsoft Docs
description: Azure 上的 SAP HANA (大型執行個體) 至災害網站的 HANA 容錯移轉程序
services: virtual-machines-linux
documentationcenter: ''
author: saghorpa
manager: jeconnoc
editor: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/10/2018
ms.author: saghorpa
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 48bea302f2f8e878275e6a7c222d703e0f31f239
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44492812"
---
# <a name="disaster-recovery-failover-procedure"></a>災害復原容錯移轉程序


>[!IMPORTANT]
>本說明文件不可用 SAP HANA 管理說明文件或 SAP 附註取代。 本文預期讀者對 SAP HANA 管理和操作有深入的了解和專業知識，尤其是關於備份、還原、高可用性和災害復原的主題。 在這份說明文件中，會顯示 SAP HANA Studio 的螢幕擷取畫面。 SAP 管理工具和工具本身的畫面內容、結構和性質，可能會因 SAP HANA 的版本而異。

容錯移轉至 DR 網站時，須考量兩種情況：

- 您需要將 SAP HANA 資料庫恢復到最新的資料狀態。 在這種情況下，您可以使用自助服務指令碼來執行容錯移轉，而不需要連絡 Microsoft。 但是對於容錯回復，您需要與 Microsoft 合作。
- 您可以還原至非最新複寫快照集的儲存體快照集。 在此情況下，您需要與 Microsoft 合作。 

>[!NOTE]
>需要在代表 DR 單位的 HANA 大型執行個體單位上執行下列步驟。 
 
若要還原到最新複寫的儲存體快照集，請執行下列步驟： 

1. 在您執行的 HANA 大型執行個體災害復原單位上，關閉非生產的 HANA 執行個體。 這是因為已預先安裝休眠中的 HANA 生產執行個體。
1. 確定沒有任何 SAP HANA 程序處於執行狀態。 使用下列命令進行這項檢查：`/usr/sap/hostctrl/exe/sapcontrol –nr <HANA instance number> - function GetProcessList`。 您應該會從輸出中看到 **hdbdaemon** 程序處於已停止狀態，而且已沒有其他 HANA 程序處於執行中或已啟動狀態。
1. 在 DR 網站 HANA 大型執行個體單位上，執行指令碼 azure_hana_dr_failover.pl。 該指令碼要求還原 SAP HANA SID。 執行要求時，請輸入一個或唯一的 SAP HANA SID，其已複寫並保留在 DR 網站中 HANA 大型執行個體單位上的 HANABackupCustomerDetails.txt 檔案。 

      如果您想要容錯移轉多個 SAP HANA 執行個體，您需要多次執行指令碼。 執行要求時，輸入您想要容錯移轉及還原的 SAP HANA SID。 完成時，指令碼會顯示新增至 HANA 大型執行個體單位的磁碟區掛接點清單。 此清單也包含還原的 DR 磁碟區。

1. 透過使用 Linux 作業系統命令，將還原的災害復原磁碟區掛接到災害復原網站中的 HANA 大型執行個體單位。 
1. 啟動休眠的 SAP HANA 生產執行個體。
1. 如果您選擇複製交易記錄備份的記錄以降低 RPO 時間，必須將這些交易記錄備份合併到新掛接 DR 的 /hana/logbackups 目錄。 請勿覆寫現有的備份。 複製尚未隨著儲存體快照集的最新複寫一起複寫的較新備份。
1. 您也可以從已複寫到 DR Azure 區域中 /hana/shared/PRD 磁碟區的快照集，還原單一檔案。 

您也可以測試 DR 容錯移轉，而不會影響實際的複寫關係。 若要執行測試容錯移轉，請遵循上述步驟 1 和 2，然後繼續進行下列的步驟 3。

>[!IMPORTANT]
>「請勿」透過**測試容錯移轉**的流程以及步驟 3 中介紹的指令碼，在 DR 網站中建立的執行個體上執行任何生產異動。 該命令會建立一組與主要站台沒有任何關聯的磁碟區。 如此一來，「不」可能同步回主要站台。 

容錯移轉測試的步驟 3：

在 DR 網站 HANA 大型執行個體單位上，執行指令碼 **azure_hana_test_dr_failover.pl**。 此指令碼「不會」停止主要站台與 DR 網站之間的複寫關聯性。 相反地，該指令碼會複製 DR 存放磁碟區。 複製程序成功之後，複製的磁碟區會還原至最新的快照集狀態，然後再掛接到 DR 單位。 該指令碼要求還原 SAP HANA SID。 請輸入一個或唯一的 SAP HANA SID，其已複寫並保留在 DR 網站中 HANA 大型執行個體單位上的 HANABackupCustomerDetails.txt 檔案。 

如果您想要測試多個 SAP HANA 執行個體，您需要多次執行指令碼。 執行要求時，輸入您想要測試容錯移轉的執行個體 SAP HANA SID。 完成時，指令碼會顯示新增至 HANA 大型執行個體單位的磁碟區掛接點清單。 此清單也包含複製的 DR 磁碟區。

繼續前往步驟 4。

   >[!NOTE]
   >如果您需要容錯移轉至 DR 網站以救援幾小時前已刪除的部分資料，且需要將 DR 磁碟區設定為較早的快照集，則會套用此程序。 

1. 在您執行的 HANA 大型執行個體災害復原單位上，關閉非生產的 HANA 執行個體。 這是因為已預先安裝休眠中的 HANA 生產執行個體。
1. 確定沒有任何 SAP HANA 程序處於執行狀態。 使用下列命令進行這項檢查：`/usr/sap/hostctrl/exe/sapcontrol –nr <HANA instance number> - function GetProcessList`。 您應該會從輸出中看到 **hdbdaemon** 程序處於已停止狀態，而且已沒有其他 HANA 程序處於執行中或已啟動狀態。
1. 決定要讓災害復原網站還原到哪個快照集名稱或 SAP HANA 備份識別碼。 在真實的災害復原案例中，此快照集通常會是最新的快照集。 如果您需要復原遺失的資料，請挑選較早的快照集。
1. 透過高優先順序的支援要求連絡 Azure 支援。 要求快照集 (具有快照集名稱和日期) 的還原或 DR 網站上的 HANA 備份識別碼。 預設值是作業端只會還原 /hana/data 磁碟區。 如果您也想要有 / hana/logbackups 磁碟區，您需要特別指明。 請勿還原 /hana/shared 磁碟區。 相反地，在您針對 PRD 重新掛接 /hana/shared 之後，您應該選擇特定的檔案，例如 **.snapshot** 目錄及其子目錄中的 global.ini。 

   在作業端，系統將會進行下列步驟：

   a. 停止將快照集從生產磁碟區複寫至災害復原磁碟區。 如果生產網站中斷是您需要執行災害復原程序的原因，則可能已經發生中斷。
   
   b. 此儲存體快照集名稱或具有您所選備份識別碼的快照集會在災害復原磁碟區上還原。
   
   c. 還原後，災害復原磁碟區即可供掛接到災害復原區域中的 HANA 大型執行個體單位。
      
1. 將災害復原磁碟區掛接至災害復原網站中的 HANA 大型執行個體單位。 
1. 啟動休眠的 SAP HANA 生產執行個體。
1. 如果您選擇複製交易記錄備份的記錄以降低 RPO 時間，必須將這些交易記錄備份合併到新掛接 DR 的 /hana/logbackups 目錄。 請勿覆寫現有的備份。 複製尚未隨著儲存體快照集的最新複寫一起複寫的較新備份。
1. 您也可以從已複寫到 DR Azure 區域中 /hana/shared/PRD 磁碟區的快照集，還原單一檔案。

後續步驟涉及根據所還原的儲存體快照集和可用的交易記錄備份來復原 SAP HANA 生產執行個體：

1. 使用 SAP HANA Studio，將備份位置變更為 **/hana/logbackups**。
   ![變更 DR 復原的備份位置](./media/hana-overview-high-availability-disaster-recovery/change_backup_location_dr1.png)

1. SAP HANA 會掃描所有備份檔案位置，並建議可作為還原目的地的最新交易記錄備份。 掃描可能需要幾分鐘的時間，完成後會出現如下所示的畫面：![DR 復原的交易記錄備份清單](./media/hana-overview-high-availability-disaster-recovery/backup_list_dr2.PNG)

1. 調整某些預設設定：

      - 清除 [使用差異備份]。
      - 選取 [初始化記錄區域]。

   ![設定初始化記錄區域](./media/hana-overview-high-availability-disaster-recovery/initialize_log_dr3.PNG)

1. 選取 [完成]。

   ![完成 DR 還原](./media/hana-overview-high-availability-disaster-recovery/finish_dr4.PNG)

此時應該會出現進度視窗，如下所示。 請記住，這是具有 3 節點相應放大 SAP HANA 組態的災害復原還原範例。

![還原進度](./media/hana-overview-high-availability-disaster-recovery/restore_progress_dr5.PNG)

如果還原似乎卡在 [完成] 畫面，而且未顯示進度畫面，請確認背景工作節點上的所有 SAP HANA 執行個體是否正在執行。 如有必要，請手動啟動 SAP HANA 執行個體。


## <a name="failback-from-a-dr-to-a-production-site"></a>從 DR 容錯回復到生產網站
您可以從 DR 容錯回復到生產網站。 讓我們看看以下情況：因為生產 Azure 區域發生問題而容錯移轉至災害復原網站，而不是因為您需要復原遺失的資料。 您已在災害復原網站中執行您的 SAP 生產工作負載一段時間。 由於生產網站中的問題已經解決，因此您想容錯回復到生產網站。 因為您不能遺失資料，所以退回生產網站的作業會涉及許多步驟，並且要與 SAP HANA on Azure 營運小組密切合作。 在問題獲得解決之後，由您決定是否觸發營運小組來開始反向同步處理到生產網站的程序。

這是要採取的步驟順序：

1. SAP HANA on Azure 營運小組取得觸發程序，以從災害復原存放磁碟區 (目前代表生產狀態) 同步處理生產存放磁碟區。 處於此狀態時，生產網站中的 HANA 大型執行個體單位會關閉。
1. SAP HANA on Azure 營運小組會監視複寫程序，並確保能趕上通知您的時間。
1. 您會關閉使用災害復原網站中生產 HANA 執行個體的應用程式。 接著執行 HANA 交易記錄備份。 然後將在災害復原網站中 HANA 大型執行個體單位上執行的 HANA 執行個體停止。
1. 在災害復原網站中 HANA 大型執行個體單位上執行的 HANA 執行個體關閉之後，營運小組必須再次手動同步處理磁碟區。
1. SAP HANA on Azure 營運小組會再次啟動生產網站中的 HANA 大型執行個體單位，然後將它轉交給您。 確定在您啟動 HANA 大型執行個體單位時，SAP HANA 執行個體是處於關閉狀態。
1. 您可執行和先前容錯移轉至災害復原網站時所執行的相同資料庫還原步驟。

## <a name="monitor-disaster-recovery-replication"></a>監視災害復原複寫

您可藉由執行`azure_hana_replication_status.pl` 指令碼來監視儲存體複寫進度的狀態。 此指令碼必須從正在災害復原位置中執行的單位來執行，才可如預期般運作。 無論複寫是否為作用中，指令碼都有作用。 您可以對災害復原位置中的租用戶所具有的每個 HANA 大型執行個體單位執行這個指令碼。 但不能將它用來取得開機磁碟區的詳細資料。

使用此命令呼叫指令碼：
```
./azure_hana_replication_status.pl
```

輸出會依磁碟區細分成下列各區段：  

- 連結狀態
- 目前的複寫活動
- 所複寫的最新快照集 
- 最新快照集的大小
- 目前在快照集間的延隔時間 (最後一次完成的快照式複寫與現在時刻)

連結狀態會顯示為 [作用中] \(除非位置之間的連結已關閉)，或是顯示為目前有正在進行容錯移轉的事件。 複寫活動會指出資料目前正在複寫、處於閒置，或者連結目前是否正發生其他活動。 最後複寫的快照集應該只會顯示為 `snapmirror…`。 然後會顯示最後一個快照集的大小。 最後則會顯示延隔時間。 延隔時間代表從排程的複寫時間到複寫完成時的時間。 資料複寫的延隔時間可能會超過一個小時，特別是初始的複寫，即使複寫已啟動亦然。 延隔時間會持續增加，直到進行中的複寫完成。

以下是輸出的範例：

```
hana_data_hm3_mnt00002_t020_dp
-------------------------------------------------
Link Status: Broken-Off
Current Replication Activity: Idle
Latest Snapshot Replicated: snapmirror.c169b434-75c0-11e6-9903-00a098a13ceb_2154095454.2017-04-21_051515
Size of Latest Snapshot Replicated: 244KB
Current Lag Time between snapshots: -   ***Less than 90 minutes is acceptable***
```

**後續步驟**
- 請參閱[從 HANA 端進行監視和疑難排解](hana-monitor-troubleshoot.md)。
