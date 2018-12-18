---
title: 為 StorSimple 裝置設定 MPIO | Microsoft Docs
description: 描述如何針對與執行 Windows Server 2012 R2 的主機連線的 StorSimple 裝置設定多重路徑 I/O。
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 03/26/2018
ms.author: alkohli
ms.openlocfilehash: 4f2b094604f486d283574f4669fcad6f72bd4431
ms.sourcegitcommit: d74657d1926467210454f58970c45b2fd3ca088d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2018
ms.locfileid: "30245732"
---
# <a name="configure-multipath-io-for-your-storsimple-device"></a>為 StorSimple 裝置設定多重路徑 I/O

本教學課程說明您應在執行 Windows Server 2012 R2 並與 StorSimple 實體裝置連接的主機上，據以安裝和使用多重路徑 I/O (MPIO) 功能的步驟。 本文中的指導僅適用於 StorSimple 8000 系列實體裝置。 StorSimple 雲端設備目前不支援 MPIO。

Microsoft 為 Windows Server 中的多重路徑 I/O (MPIO) 功能建立支援，協助建立高可用性、容錯的 iSCSI 網路組態。 MPIO 使用備援實體路徑元件 — 配接器、纜線以及交換器 — 以在伺服器與存放裝置之間建立邏輯路徑。 如果元件故障而導致邏輯路徑失敗，多重路徑邏輯使用替代的 I/O 路徑，讓應用程式仍然可以存取其資料。 此外，依照您的設定，MPIO 也能藉由重新平衡所有路徑的負載來改善效能。 如需詳細資訊，請參閱 [MPIO 概觀](https://technet.microsoft.com/library/cc725907.aspx "MPIO 概觀 and features")。

您應該為 StorSimple 裝置設定 MPIO，才能獲得高可用性的 StorSimple 方案。 當您在執行 Windows Server 2012 R2 的主機伺服器上安裝 MPIO 時，伺服器才能容許連結、網路或介面失敗。

## <a name="mpio-configuration-summary"></a>MPIO 設定摘要

MPIO 是 Windows 伺服器預設不會安裝的選擇性功能。 您應該透過伺服器管理員將它安裝為功能。

請依照下列步驟在 StorSimple 裝置上設定 MPIO：

* 步驟 1：在 Windows Server 主機上安裝 MPIO
* 步驟 2：為 StorSimple 磁碟區設定 MPIO 
* 步驟 3：在主機上掛接 StorSimple 磁碟區
* 步驟 4：設定 MPIO 以獲得高可用性與負載平衡

上述步驟會在下列各節討論。

## <a name="step-1-install-mpio-on-the-windows-server-host"></a>步驟 1：在 Windows Server 主機上安裝 MPIO

若要在 Windows Server 主機上安裝這項功能，請完成下列程序。

#### <a name="to-install-mpio-on-the-host"></a>若要在主機上安裝 MPIO

1. 在 Windows Server 主機上開啟 [伺服器管理員]。 根據預設，伺服器管理員會在 Administrators 群組成員登入執行 Windows Server 2012 R2 或 Windows Server 2012 的電腦時啟動。 如果尚未開啟 [伺服器管理員]，請按一下 [開始] > [伺服器管理員]。
   
   ![伺服器管理員](./media/storsimple-configure-mpio-windows-server/IC740997.png)

2. 按一下 [伺服器管理員] > [儀表板] > [新增角色及功能]。 這樣會啟動 [新增角色及功能]  精靈。
   
   ![新增角色及功能精靈 1](./media/storsimple-configure-mpio-windows-server/IC740998.png)
3. 在 [新增角色及功能] 精靈中，執行下列步驟：
   
   1. 在 [開始之前] 頁面中按 [下一步]。
   2. 在 [選取安裝類型] 頁面上，接受 [角色型或功能型安裝] 的預設值。 按 [下一步] 。
   
       ![新增角色及功能精靈 2](./media/storsimple-configure-mpio-windows-server/IC740999.png)
   3. 在 [選取目的地伺服器] 頁面上，選擇 [從伺服器集區選取伺服器]。 應該會自動探索主機伺服器。 按 [下一步] 。
   4. 在 [選取伺服器角色] 頁面上，按 [下一步]。
   5. 在 [選取功能] 頁面上，選取 [多重路徑 I/O]，然後按 [下一步]。
   
       ![新增角色及功能精靈 5](./media/storsimple-configure-mpio-windows-server/IC741000.png)
   6. 在 [確認安裝選項] 頁面上，確認選取項目，然後選取 [需要時自動重新啟動目的地伺服器]，如下所示。 按一下 [Install] 。
   
       ![新增角色及功能精靈 8](./media/storsimple-configure-mpio-windows-server/IC741001.png)
   7. 完成安裝時，您會收到通知。 按一下 [關閉]  即可關閉精靈。
   
       ![新增角色及功能精靈 9](./media/storsimple-configure-mpio-windows-server/IC741002.png)

## <a name="step-2-configure-mpio-for-storsimple-volumes"></a>步驟 2：為 StorSimple 磁碟區設定 MPIO 

您必須設定 MPIO 才能識別 StorSimple 磁碟區。 若要設定 MPIO 以辨識 StorSimple 磁碟區，請執行下列步驟。

#### <a name="to-configure-mpio-for-storsimple-volumes"></a>若要為 StorSimple 磁碟區設定 MPIO 

1. 開啟 [MPIO 設定] 。 按一下 [伺服器管理員] > [儀表板] > [工具] > [MPIO]。
2. 在 [MPIO 內容] 對話方塊中，選取 [探索多重路徑] 索引標籤。
3. 選取 [新增 iSCSI 裝置支援]，然後按一下 [新增]。  
   ![MPIO 內容探索多重路徑](./media/storsimple-configure-mpio-windows-server/IC741003.png)
4. 當系統提示您將伺服器重新開機。
5. 在 [MPIO 內容] 對話方塊中，按一下 [MPIO 裝置] 索引標籤。按一下 [新增]。
    </br>![MPIO 內容 MPIO 裝置](./media/storsimple-configure-mpio-windows-server/IC741004.png)
6. 在 [新增 MPIO 支援] 對話方塊的 [裝置硬體識別碼] 下，輸入您的裝置序號。 若要取得裝置序號，請存取您的 StorSimple 裝置管理員服務。 瀏覽至 [裝置] > [儀表板]。 裝置序號會顯示在裝置儀表板右邊的 [快速概覽] 窗格。
    </br>
    ![新增 MPIO 支援](./media/storsimple-configure-mpio-windows-server/IC741005.png)
7. 當系統提示您將伺服器重新開機。

## <a name="step-3-mount-storsimple-volumes-on-the-host"></a>步驟 3：在主機上掛接 StorSimple 磁碟區

在 Windows 伺服器上設定 MPIO 之後，就能掛接在 StorSimple 裝置上建立的磁碟區，並可以利用 MPIO 獲得備援。 若要掛接磁碟區，請執行下列步驟。

#### <a name="to-mount-volumes-on-the-host"></a>若要在主機上掛接磁碟區

1. 在 Windows Server 主機上，開啟 [iSCSI 啟動器內容]  視窗。 按一下 [伺服器管理員] > [儀表板] > [工具] > [iSCSI 啟動器]。
2. 在 [iSCSI 啟動器內容] 對話方塊中，按一下 [探索] 索引標籤，然後按一下 [搜尋目標入口]。
3. 在 [搜尋目標入口] 對話方塊中，執行下列步驟：
   
   1. 輸入 StorSimple 裝置的 DATA 連接埠 IP 位址 (例如，輸入 DATA 0)。
   2. 按一下 [確定] 以返回 [iSCSI 啟動器內容] 對話方塊。
     
     > [!IMPORTANT]
     > **如果您使用私人網路進行 iSCSI 連線，請輸入連線到私人網路的 DATA 連接埠 IP 位址。**
    
4. 在裝置上針對第二個網路介面 (例如，DATA 1) 重複步驟 2-3 。 請記住，您應該為 iSCSI 啟用這些介面。 如需詳細資訊，請參閱[修改網路介面](storsimple-8000-modify-device-config.md#modify-network-interfaces)。
5. 在 [iSCSI 啟動器內容] 對話方塊中，選取 [目標] 索引標籤。 您應該會在 [探索到的目標] 下看到 StorSimple 裝置目標 IQN。

   ![iSCSI 啟動器內容目標索引標籤](./media/storsimple-configure-mpio-windows-server/IC741007.png)
   
6. 按一下 [連接]  ，以與 StorSimple 裝置建立 iSCSI 工作階段。 [連線到目標] 對話方塊隨即出現。
7. 在 [連線到目標] 對話方塊中，選取 [啟用多重路徑] 核取方塊。 按一下 [進階] 。
8. 在 [進階設定] 對話方塊中，執行下列步驟：
   
   1. 在 [本機介面卡] 下拉式清單中，選取 [Microsoft iSCSI 啟動器]。
   2. 在 [啟動器 IP]  下拉式清單中，選取主機的 IP 位址。
   3. 在 [目標入口 IP]  下拉式清單中，選取裝置介面的 IP。
   4. 按一下 [確定] 以返回 [iSCSI 啟動器內容] 對話方塊。
9. 按一下 [內容] 。 在 [內容] 對話方塊中，按一下 [新增工作階段]。
10. 在 [連線到目標] 對話方塊中，選取 [啟用多重路徑] 核取方塊。 按一下 [進階] 。
11. 在 [進階設定]  對話方塊中：

    1. 在 [本機介面卡]  下拉式清單中，選取 [Microsoft iSCSI 啟動器]。
    2. 在 [啟動器 IP]  下拉式清單中，選取對應到主機的 IP 位址。 在此情況下，您是將裝置上的兩個網路介面連線到主機上的單一網路介面。 因此，這個介面會和第一個工作階段提供的相同。
    3. 在 [目標入口 IP]  下拉式清單中，為裝置上啟用的第二個資料介面選取 IP 位址。
    4. 按一下 [確定]  以返回 [iSCSI 啟動器內容] 對話方塊。 您完成將第二個工作階段新增到目標。
12. 藉由瀏覽至 [伺服器管理員] > [儀表板] > [電腦管理]，來開啟 [電腦管理]。 在左窗格中，按一下 [存放裝置] > [磁碟管理]。 在 StorSimple 裝置上建立且是此主機可以看見的磁碟區，在 [磁碟管理] 下會顯示為新磁碟。
13. 初始化磁碟，並建立新的磁碟區。 在格式化程序期間，選取 64 KB 的區塊大小。
    
    ![磁碟管理](./media/storsimple-configure-mpio-windows-server/IC741008.png)
14. 在 [磁碟管理] 下，於 [磁碟] 按一下滑鼠右鍵，然後選取 [內容]。
15. 在 [StorSimple 模型 #### 多重路徑磁碟裝置內容] 對話方塊中，按一下 [MPIO] 索引標籤。
    
    ![StorSimple 8100 多重路徑磁碟 DeviceProp。](./media/storsimple-configure-mpio-windows-server/IC741009.png)
16. 在 [DSM 名稱] 區段中，按一下 [詳細資料] 並確認參數已設定為預設參數。 預設參數如下：
    
    * 路徑確認期間 = 30
    * 重試計數 = 3
    * PDO 移除期間 = 20
    * 重試間隔 = 1
    * 啟用路徑確認 = 未選取。

> [!NOTE]
> **請不要修改預設的參數。**


## <a name="step-4-configure-mpio-for-high-availability-and-load-balancing"></a>步驟 4：設定 MPIO 以獲得高可用性與負載平衡

多個工作階段必須以手動方式加入以宣告不同的路徑，才能獲得以多重路徑為基礎的高可用性與負載平衡。 比方說，如果主機有兩個介面連接到 iSCSI 網路，而裝置也有兩個介面連接到 iSCSI 網路，那麼您需要以正確的路徑排列組合設定四個工作階段 (如果每個 DATA 介面與主機介面都位在不同的 IP 子網路且不可路由時，將只需要兩個工作階段)。

「我們建議您在裝置和應用程式主機之間至少有 8 個作用中的平行工作階段。」 這可藉由在 Windows Server 系統上啟用 4 個網路介面來達成。 在 Windows Server 主機上的硬體或作業系統層級上，透過網路虛擬化技術使用實體網路介面或虛擬介面。 透過在裝置上使用這兩個網路介面，此設定將會有 8 個作用中的工作階段。 這項設定有助於將裝置和雲端輸送量最佳化。

> [!IMPORTANT]
> **建議您不要混合使用 1 GbE 與 10 GbE 網路介面。如果您使用兩個網路介面，這兩個介面的類型應要完全相同。**

下列程序描述有兩個網路介面的 StorSimple 裝置連接到有兩個網路介面的主機時，要如何新增工作階段。 這只會為您提供 4 個工作階段。 使用這個與具有兩個連接到具有四個網路介面之主機的網路介面的 StorSimple 裝置所使用的相同程序。 您必須設定 8 個工作階段，而不是此處所述的 4 個工作階段。

### <a name="to-configure-mpio-for-high-availability-and-load-balancing"></a>若要設定 MPIO 以獲得高可用性與負載平衡

1. 若要探索目標：在 [iSCSI 啟動器內容] 對話方塊的 [探索] 索引標籤中，然後按一下 [探索入口]。
2. 在 [連線到目標]  對話方塊中，輸入其中一個裝置網路介面的 IP 位址。
3. 按一下 [確定] 以返回 [iSCSI 啟動器內容] 對話方塊。
4. 在 [iSCSI 啟動器內容] 對話方塊中，選取 [目標] 索引標籤，反白探索到的目標，然後按一下 [連線]。 [連線到目標]  對話方塊隨即出現。
5. 在 [連線到目標]  對話方塊中：
   
   1. 保留 [將此連線新增到我的最愛目標清單]  預設選取的目標設定。 這樣一來，這部電腦每次重新啟動時，裝置都會自動嘗試重新連線。
   2. 選取 [啟用多重路徑]  核取方塊。
   3. 按一下 [進階] 。
6. 在 [進階設定]  對話方塊中：
   
   1. 在 [本機介面卡] 下拉式清單中，選取 [Microsoft iSCSI 啟動器]。
   2. 在 [啟動器 IP] 下拉式清單中，選取對應到主機上第一個介面 (iSCSI 介面) 的 IP 位址。
   3. 在 [目標入口 IP] 下拉式清單中，為裝置上啟用的第一個資料介面選取 IP 位址。
   4. 按一下 [確定]  以返回 [iSCSI 啟動器內容] 對話方塊。
7. 按一下 [內容] 並在 [內容] 對話方塊中，按一下 [新增工作階段]。
8. 在 [連線到目標] 對話方塊中，選取 [啟用多重路徑] 核取方塊，然後按一下 [進階]。
9. 在 [進階設定]  對話方塊中：
   
   1. 在 [本機介面卡] 下拉式清單中，選取 [Microsoft iSCSI 啟動器]。
   2. 在 [啟動器 IP] 下拉式清單中，選取對應到主機上第二個 iSCSI 介面的 IP 位址。
   3. 在 [目標入口 IP]  下拉式清單中，為裝置上啟用的第二個資料介面選取 IP 位址。
   4. 按一下 [確定] 以返回 [iSCSI 啟動器內容] 對話方塊。 您現已完成將第二個工作階段新增到目標。
10. 重複步驟 8-10，將其他工作階段 (路徑) 新增到目標。 主機上有兩個介面且裝置上也有兩個，您總共可以新增四個工作階段。
11. 在新增所需的工作階段 (路徑) 之後，請在 [iSCSI 啟動器內容] 對話方塊中，選取目標，然後按一下 [內容]。 在 [內容]  對話方塊的 [工作階段] 索引標籤中，針對可能的路徑排列組合記下其對應的四個工作階段識別碼。 若要取消工作階段，請選取工作階段識別碼旁邊的核取方塊，然後按一下 [中斷連線] 。
12. 若要檢視工作階段內顯示的裝置，請選取 [裝置]  索引標籤。若要為選取的裝置設定 MPIO 原則，請按一下 [MPIO] 。 [裝置詳細資料] 對話方塊隨即出現。 在 [MPIO] 索引標籤上，您可以選取適當 [負載平衡原則] 設定。 您也可以檢視 [使用中] 或 [待命] 路徑類型。

## <a name="next-steps"></a>後續步驟

深入了解[使用 StorSimple 裝置管理員服務修改 StorSimple 裝置設定](storsimple-8000-modify-device-config.md)。

