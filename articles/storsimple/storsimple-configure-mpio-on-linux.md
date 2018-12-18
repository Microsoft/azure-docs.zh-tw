---
title: 在 StorSimple Linux 主機上設定 MPIO | Microsoft Docs
description: 在連線到執行 CentOS 6.6 之 Linux 主機的 StorSimple 上設定 MPIO
services: storsimple
documentationcenter: NA
author: alkohli
manager: jeconnoc
editor: tysonn
ms.assetid: ca289eed-12b7-4e2e-9117-adf7e2034f2f
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/09/2018
ms.author: alkohli
ms.openlocfilehash: d1188b40021fbb221bc19af6d4a5397f7ba8f800
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39439867"
---
# <a name="configure-mpio-on-a-storsimple-host-running-centos"></a>在執行 CentOS 的 StorSimple 主機上設定 MPIO
本文說明在 Centos 6.6 主機伺服器上設定多重路徑 IO (MPIO) 所需的步驟。 主機伺服器會連線到您的 Microsoft Azure StorSimple 裝置，以透過 iSCSI 啟動器取得高可用性。 文中詳細描述多重路徑裝置的自動探索，以及 StorSimple 磁碟區特有的設定。

此程序適用於 StorSimple 8000 系列裝置的所有模型。

> [!NOTE]
> StorSimple 雲端設備無法使用此程序。 如需詳細資訊，請參閱〈如何設定雲端設備的主機伺服器〉。


## <a name="about-multipathing"></a>關於多重路徑
多重路徑功能可讓您在主機伺服器與儲存體裝置之間設定多個 I/O 路徑。 這些 I/O 路徑是可包含不同纜線、開關、網路介面及控制站的實體 SAN 連線。 多重路徑功能會彙總 I/O 路徑，以設定與所有彙總路徑相關聯的新裝置。

多重路徑有雙重目的：

* **高可用性**︰如果 I/O 路徑的任何元素 (例如纜線、開關、網路介面或控制站) 失敗，它會提供替代路徑。
* **負載平衡**︰視您的儲存體裝置的組態而定，它可透過偵測 I/O 路徑上的負載及動態重新平衡這些負載來改善效能。

### <a name="about-multipathing-components"></a>關於多重路徑元件
Linux 中的多重路徑是由核心元件和下列的使用者空間元件所組成。

* **核心**︰主要元件是 *device-mapper* ，可重新路由傳送 I/O 及支援路徑和路徑群組的容錯移轉。

* **使用者空間**：這些是 *multipath-tools* ，可藉由指示 device-mapper 多重路徑模組該怎麼做來管理多重路徑的裝置。 這些工具包括：
   
   * **Multipath**︰列出及設定多重路徑的裝置。
   * **Multipathd**︰可執行多重路徑及監視路徑的精靈。
   * **Devmap-name**︰提供有意義的裝置名稱給 udev for devmaps。
   * **Kpartx**︰將線性 devmaps 對應至裝置資料分割，讓多重路徑對應可分割。
   * **Multipath.conf**︰多重路徑精靈的組態檔，用來覆寫內建組態資料表。

### <a name="about-the-multipathconf-configuration-file"></a>關於 multipath.conf 組態檔
組態檔 `/etc/multipath.conf` 讓使用者可以設定許多多重路徑功能。 `multipath` 命令和核心精靈 `multipathd` 會使用此檔案中找到的資訊。 只有在多重路徑裝置的設定期間才會參考此檔案。 務必在您執行 `multipath` 命令之前進行所有的變更。 如果您之後修改該檔案，則必須停止後重新啟動 multipathd，變更才會生效。

multipath.conf 有五個區段：

- **系統層級的預設值** *(defaults)*：您可以覆寫系統層級預設值。
- **列入封鎖清單的裝置** *(blacklist)*：您可以指定不受 device-mapper 控制的裝置清單。
- **封鎖清單例外狀況** (blacklist_exceptions)：您可以識別要被視為多重路徑裝置的特定裝置，即使這些裝置已列入封鎖清單。
- **儲存體控制站特定設定** *(devices)*：您可以指定將套用到裝置的組態設定 (具有廠商和產品資訊)。
- **裝置特定設定** *(multipaths)*：您可以使用本節來微調個別 LUN 的組態設定。

## <a name="configure-multipathing-on-storsimple-connected-to-linux-host"></a>在連線到 Linux 主機的 StorSimple 上設定多重路徑
可以設定連線到 Linux 主機的 StorSimple 裝置，以取得高可用性和負載平衡。 例如，如果 Linux 主機有兩個連接到 SAN 的介面，而裝置有兩個連接到 SAN 的介面，以致這些介面位於相同的子網路上，則會有 4 個路徑可用。 不過，如果裝置瀚主機介面上的每個 DATA 介面位於不同的 IP 子網路上 (且不可路由傳送)，則只有 2 路徑可用。 您可以設定多重路徑，以便自動探索所有可用的路徑、選擇這些路徑的負載平衡演算法、套用僅限 StorSimple 磁碟區的特定組態設定，然後啟用和驗證多重路徑。

下列程序描述有兩個網路介面的 StorSimple 裝置連接到有兩個網路介面的主機時，要如何設定多重路徑。

## <a name="prerequisites"></a>必要條件
本節將詳細說明 CentOS 伺服器和 StorSimple 裝置的設定必要條件。

### <a name="on-centos-host"></a>在 CentOS 主機上
1. 確定 CentOS 主機已啟用 2 個網路介面。 輸入：
   
    `ifconfig`
   
    下列範例顯示兩個網路介面 (`eth0` 和 `eth1`) 都出現在主機上時的輸出。
   
        [root@centosSS ~]# ifconfig
        eth0  Link encap:Ethernet  HWaddr 00:15:5D:A2:33:41  
          inet addr:10.126.162.65  Bcast:10.126.163.255  Mask:255.255.252.0
          inet6 addr: 2001:4898:4010:3012:215:5dff:fea2:3341/64 Scope:Global
          inet6 addr: fe80::215:5dff:fea2:3341/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
         RX packets:36536 errors:0 dropped:0 overruns:0 frame:0
          TX packets:6312 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:13994127 (13.3 MiB)  TX bytes:645654 (630.5 KiB)
   
        eth1  Link encap:Ethernet  HWaddr 00:15:5D:A2:33:42  
          inet addr:10.126.162.66  Bcast:10.126.163.255  Mask:255.255.252.0
          inet6 addr: 2001:4898:4010:3012:215:5dff:fea2:3342/64 Scope:Global
          inet6 addr: fe80::215:5dff:fea2:3342/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:25962 errors:0 dropped:0 overruns:0 frame:0
          TX packets:11 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:2597350 (2.4 MiB)  TX bytes:754 (754.0 b)
   
        loLink encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:12 errors:0 dropped:0 overruns:0 frame:0
          TX packets:12 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:720 (720.0 b)  TX bytes:720 (720.0 b)
1. 在 CentOS 伺服器上安裝 *iSCSI-initiator-utils* 。 執行下列步驟以安裝 *iSCSI-initiator-utils*。
   
   1. 以 `root` 身分登入到 CentOS 主機。
   1. 安裝 *iSCSI-initiator-utils*。 輸入：
      
       `yum install iscsi-initiator-utils`
   1. 在成功安裝 *iSCSI-Initiator-utils* 之後，啟動 iSCSI 服務。 輸入：
      
       `service iscsid start`
      
       有時候，`iscsid` 可能無法實際啟動，而可能需要 `--force` 選項
   1. 若要確保 iSCSI 啟動器會在開機時啟用，請使用 `chkconfig` 命令來啟用此服務。
      
       `chkconfig iscsi on`
   1. 若要驗證該服務是否正確設定，請執行以下命令：
      
       `chkconfig --list | grep iscsi`
      
       下方顯示一項範例輸出。
      
           iscsi   0:off   1:off   2:on3:on4:on5:on6:off
           iscsid  0:off   1:off   2:on3:on4:on5:on6:off
      
       在上述範例中，您可以看到 iSCSI 環境將在開機時於執行層級 2、 3、 4 和 5 執行。
1. 安裝 *device-mapper-multipath*。 輸入：
   
    `yum install device-mapper-multipath`
   
    將會開始安裝。 在系統提示您進行確認時，輸入 **Y** 。

### <a name="on-storsimple-device"></a>在 StorSimple 裝置上
您的 StorSimple 裝置應該︰

* 至少有兩個介面已啟用 iSCSI。 若要確認這兩個介面已在 StorSimple 裝置上啟用 iSCSI，請在 StorSimple 裝置的 Azure 傳統入口網站中執行下列步驟︰
  
  1. 登入 StorSimple 裝置的傳統入口網站。
  1. 選取 StorSimple Manager 服務，按一下 [裝置]  ，然後選擇該特定 StorSimple 裝置。 按一下 [設定]  並驗證網路介面設定。 下面顯示的螢幕擷取畫面包含已啟用 iSCSI 的兩個網路介面。 以下 DATA 2 和 DATA 3 兩個 10 GbE 介面都已啟用 iSCSI。
     
      ![MPIO StorSimple DATA 2 設定](./media/storsimple-configure-mpio-on-linux/IC761347.png)
     
      ![MPIO StorSimple DATA 3 設定](./media/storsimple-configure-mpio-on-linux/IC761348.png)
     
      在 [設定]  頁面中
     
     1. 確定這兩個網路介面都已啟用 iSCSI。 [啟用 iSCSI] 欄位應設定為 [是]。
     1. 確定網路介面的速度相同，兩者都應該是 1 GbE 或 10 GbE。
     1. 請記下已啟用 iSCSI 的介面的 IPv4 位址，並加以儲存供稍後用於主機上。
* 應可從 CentOS 伺服器存取 StorSimple 裝置上的 iSCSI 介面。
      若要確認這點，您需要在主機伺服器上提供 StorSimple 已啟用 iSCSI 的網路介面的 IP 位址。 使用的命令以及 DATA2 (10.126.162.25) 和 DATA3 (10.126.162.26) 的對應輸出如下所示：
  
        [root@centosSS ~]# iscsiadm -m discovery -t sendtargets -p 10.126.162.25:3260
        10.126.162.25:3260,1 iqn.1991-05.com.microsoft:storsimple8100-shx0991003g44mt-target
        10.126.162.26:3260,1 iqn.1991-05.com.microsoft:storsimple8100-shx0991003g44mt-target

### <a name="hardware-configuration"></a>硬體組態
建議您連接不同路徑上的兩個 iSCSI 網路介面，以獲得備援。 下圖顯示可取得高可用性以及使 CentOS 伺服器與 StorSimple 裝置的多重路徑達到平衡負載的建議硬體組態。

![連線至 Linux 主機之 StorSimple 的 MPIO 硬體設定](./media/storsimple-configure-mpio-on-linux/MPIOHardwareConfigurationStorSimpleToLinuxHost2M.png)

如上圖所示：

* StorSimple 裝置處於具有兩個控制器的主動-被動組態中。
* 兩個 SAN 開關會連接到您的裝置控制器。
* 兩個 iSCSI 啟動器會在 StorSimple 裝置上啟用。
* CentOS 主機已啟用 2 個網路介面。

如果主機和資料介面均可路由傳送，則上述組態將會在您的裝置與主機之間產生 4 個不同的路徑。

> [!IMPORTANT]
> * 建議您不要對多重路徑功能混合使用 1 GbE 與 10 GbE 網路介面。 使用兩個網路介面時，這兩個介面的類型應要完全相同。
> * 在您的 StorSimple 裝置上，DATA0、DATA1、DATA4 和 DATA5 為 1 GbE 介面，而 DATA2 和 DATA3 則為 10 GbE 網路介面。
> 
> 

## <a name="configuration-steps"></a>組態步驟
多重路徑的設定步驟涉及設定自動探索的可用路徑、指定要使用的負載平衡演算法、啟用多重路徑及最後驗證組態。 下列各節會詳細討論這些步驟。

### <a name="step-1-configure-multipathing-for-automatic-discovery"></a>步驟 1：為自動探索設定多重路徑
系統可以自動探索及設定多重路徑支援的裝置。

1. 初始化 `/etc/multipath.conf` 檔案。 輸入：
   
     `mpathconf --enable`
   
    上述命令會建立 `sample/etc/multipath.conf` 檔案。
1. 啟動多重路徑服務。 輸入：
   
    `service multipathd start`
   
    您會看到下列輸出︰
   
    `Starting multipathd daemon:`
1. 啟用多重路徑的自動探索。 輸入：
   
    `mpathconf --find_multipaths y`
   
    這將會修改 `multipath.conf` 的預設區段，如下所示：
   
        defaults {
        find_multipaths yes
        user_friendly_names yes
        path_grouping_policy multibus
        }

### <a name="step-2-configure-multipathing-for-storsimple-volumes"></a>步驟 2：為 StorSimple 磁碟區設定多重路徑
根據預設，所有裝置都會列入 multipath.conf 檔案的封鎖清單並遭到略過。 您必須建立封鎖清單例外狀況，允許 StorSimple 裝置中的磁碟區啟動多重路徑。

1. 編輯 `/etc/mulitpath.conf` 檔案。 輸入：
   
    `vi /etc/multipath.conf`
1. 找出 multipath.conf 檔案中的 blacklist_exceptions 區段。 您的 StorSimple 裝置必須列為本節中的封鎖清單例外狀況。 您可以在此檔案中取消註解相關行，如下所示修改該檔案 (僅使用您所用裝置的特定模型)︰
   
        blacklist_exceptions {
            device {
                       vendor  "MSFT"
                       product "STORSIMPLE 8100*"
            }
            device {
                       vendor  "MSFT"
                       product "STORSIMPLE 8600*"
            }
           }

### <a name="step-3-configure-round-robin-multipathing"></a>步驟 3：設定循環配置資源多重路徑
此負載平衡演算法會以平衡且循環配置資源的方式，使用作用中控制器的所有可用多重路徑。

1. 編輯 `/etc/multipath.conf` 檔案。 輸入：
   
    `vi /etc/multipath.conf`
1. 在 `defaults` 區段之下，將 `path_grouping_policy` 設定為 `multibus`。 `path_grouping_policy` 指定預設路徑群組原則，以要套用到未指定的多重路徑。 defaults 區段會如下所示。
   
        defaults {
                user_friendly_names yes
                path_grouping_policy multibus
        }

> [!NOTE]
> 最常見的 `path_grouping_policy` 值包括︰
> 
> * failover = 每一個優先順序群組 1 個路徑
> * multibus = 1 個優先順序群組中的所有有效路徑
> 
> 

### <a name="step-4-enable-multipathing"></a>步驟 4：啟用多重路徑
1. 重新啟動 `multipathd` 精靈。 輸入：
   
    `service multipathd restart`
1. 輸出應如下所示：
   
        [root@centosSS ~]# service multipathd start
        Starting multipathd daemon:  [OK]

### <a name="step-5-verify-multipathing"></a>步驟 5：驗證多重路徑
1. 如下所示，先確定 iSCSI 連接是以 StorSimple 裝置建立︰
   
   a. 探索您的 StorSimple 裝置。 輸入：
      
    ```
    iscsiadm -m discovery -t sendtargets -p  <IP address of network interface on the device>:<iSCSI port on StorSimple device>
    ```
    
    當 DATA0 的 IP 位址為 10.126.162.25 且連接埠 3260 在 StorSimple 裝置上開啟以便處理輸出 iSCSI 流量時，輸出會如下所示︰
    
    ```
    10.126.162.25:3260,1 iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target
    10.126.162.26:3260,1 iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target
    ```

    從上述輸出複製 StorSimple 裝置的 IQN ( `iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target`)。

   b. 使用目標 IQN 連接到此裝置。 StorSimple 裝置在此是 iSCSI 目標。 輸入：

    ```
    iscsiadm -m node --login -T <IQN of iSCSI target>
    ```

    下列範例顯示目標 IQN 為 `iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target`的輸出。 此輸出指出您已順利連接到您的裝置上已啟用 iSCSI 的兩個網路介面。

    ```
    Logging in to [iface: eth0, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.25,3260] (multiple)
    Logging in to [iface: eth1, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.25,3260] (multiple)
    Logging in to [iface: eth0, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.26,3260] (multiple)
    Logging in to [iface: eth1, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.26,3260] (multiple)
    Login to [iface: eth0, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.25,3260] successful.
    Login to [iface: eth1, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.25,3260] successful.
    Login to [iface: eth0, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.26,3260] successful.
    Login to [iface: eth1, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.26,3260] successful.
    ```

    如果您在此只看到一個主機介面和兩個路徑，您必須在主機上針對 iSCSI 啟用這兩個介面。 您可以遵循 [Linux 文件中的詳細指示](https://access.redhat.com/documentation/Red_Hat_Enterprise_Linux/5/html/Online_Storage_Reconfiguration_Guide/iscsioffloadmain.html)。

1. 磁碟區會從 StorSimple 裝置公開至 CentOS 伺服器。 如需詳細資訊，請參閱[步驟 6：建立磁碟區](storsimple-8000-deployment-walkthrough-u2.md#step-6-create-a-volume) (透過 StorSimple 裝置上的 Azure 入口網站)。

1. 驗證可用的路徑。 輸入：

      ```
      multipath –l
      ```

      下列範例顯示透過兩個可用路徑連接到單一主機網路介面的 StorSimple 裝置上兩個網路介面的輸出。

        ```
        mpathb (36486fd20cc081f8dcd3fccb992d45a68) dm-3 MSFT,STORSIMPLE 8100
        size=100G features='0' hwhandler='0' wp=rw
        `-+- policy='round-robin 0' prio=0 status=active
        |- 7:0:0:1 sdc 8:32 active undef running
        `- 6:0:0:1 sdd 8:48 active undef running
        ```

        The following example shows the output for two network interfaces on a StorSimple device connected to two host network interfaces with four available paths.

        ```
        mpathb (36486fd27a23feba1b096226f11420f6b) dm-2 MSFT,STORSIMPLE 8100
        size=100G features='0' hwhandler='0' wp=rw
        `-+- policy='round-robin 0' prio=0 status=active
        |- 17:0:0:0 sdb 8:16 active undef running
        |- 15:0:0:0 sdd 8:48 active undef running
        |- 14:0:0:0 sdc 8:32 active undef running
        `- 16:0:0:0 sde 8:64 active undef running
        ```

        After the paths are configured, refer to the specific instructions on your host operating system (Centos 6.6) to mount and format this volume.

## <a name="troubleshoot-multipathing"></a>多重路徑疑難排解
如果您在多重路徑設定期間遇到任何問題，請參閱本節所提供的一些實用秘訣。

問： 我未看見 `multipath.conf` 檔案的變更生效。

A. 如果您已對 `multipath.conf` 檔案進行任何變更，您必須重新啟動多重路徑服務。 輸入以下命令：

    service multipathd restart

問： 我已在 StorSimple 裝置上啟用兩個網路介面以及在主機上啟用兩個網路介面。 當我列出可用的路徑時，我只看到兩個路徑。 我預計看見四個可用的路徑。

A. 確定這兩個路徑位於相同的子網路上並可路由傳送。 如果網路介面位於不同的 vLAN 上且不可路由傳送，您只會看到兩個路徑。 確認這點的唯一方法就是確定您可以從 StorSimple 裝置上的網路介面存取這兩個主機介面。 您必須[連絡 Microsoft 支援](storsimple-8000-contact-microsoft-support.md)，因為這項驗證只能透過支援工作階段完成。

問： 當我列出可用的路徑時，我看不到任何輸出。

A. 通常，看不到任何多重路徑的路徑，即暗示多重路徑精靈有問題，而最可能是 `multipath.conf` 檔案有問題。

此外，也值得檢查您可以在連接到目標後實際看到某些磁碟，因為多重路徑清單沒有回應也可能表示您沒有任何磁碟。

* 使用下列命令來重新掃描 SCSI 匯流排：
  
    `$ rescan-scsi-bus.sh ` (屬於 sg3_utils 封包)
* 輸入下列命令：
  
    `$ dmesg | grep sd*`
     
     或
  
    `$ fdisk –l`
  
    這些將傳回最近新增磁碟的詳細資料。
* 若要判斷它是否為 StorSimple 磁碟，請使用下列命令︰
  
    `cat /sys/block/<DISK>/device/model`
  
    這會傳回一個字串，判斷它是否為 StorSimple 磁碟。

有一個比較不可能的可能原因也可能是 iscsid pid 過時。 使用下列命令從 iSCSI 工作階段登出：

    iscsiadm -m node --logout -p <Target_IP>

對 ISCSI 目標 (也就是您的 StorSimple 裝置) 上所有連接的網路介面重複此命令。 從所有 iSCSI 工作階段登出後，請使用 iSCSI 目標 IQN 來重新建立 iSCSI 工作階段。 輸入以下命令：

    iscsiadm -m node --login -T <TARGET_IQN>


問： 我不確定我的裝置是否已列入允許清單。

A. 若要驗證您的裝置是否已列入允許清單，請使用下列疑難排解互動式命令︰

    multipathd –k
    multipathd> show devices
    available block devices:
    ram0 devnode blacklisted, unmonitored
    ram1 devnode blacklisted, unmonitored
    ram2 devnode blacklisted, unmonitored
    ram3 devnode blacklisted, unmonitored
    ram4 devnode blacklisted, unmonitored
    ram5 devnode blacklisted, unmonitored
    ram6 devnode blacklisted, unmonitored
    ram7 devnode blacklisted, unmonitored
    ram8 devnode blacklisted, unmonitored
    ram9 devnode blacklisted, unmonitored
    ram10 devnode blacklisted, unmonitored
    ram11 devnode blacklisted, unmonitored
    ram12 devnode blacklisted, unmonitored
    ram13 devnode blacklisted, unmonitored
    ram14 devnode blacklisted, unmonitored
    ram15 devnode blacklisted, unmonitored
    loop0 devnode blacklisted, unmonitored
    loop1 devnode blacklisted, unmonitored
    loop2 devnode blacklisted, unmonitored
    loop3 devnode blacklisted, unmonitored
    loop4 devnode blacklisted, unmonitored
    loop5 devnode blacklisted, unmonitored
    loop6 devnode blacklisted, unmonitored
    loop7 devnode blacklisted, unmonitored
    sr0 devnode blacklisted, unmonitored
    sda devnode whitelisted, monitored
    dm-0 devnode blacklisted, unmonitored
    dm-1 devnode blacklisted, unmonitored
    dm-2 devnode blacklisted, unmonitored
    sdb devnode whitelisted, monitored
    sdc devnode whitelisted, monitored
    dm-3 devnode blacklisted, unmonitored


如需詳細資訊，移至 [對多重路徑使用疑難排解互動式命令](http://www.centos.org/docs/5/html/5.1/DM_Multipath/multipath_config_confirm.html)。

## <a name="list-of-useful-commands"></a>有用的命令清單
| 在系統提示您進行確認時，輸入  | 命令 | 說明 |
| --- | --- | --- |
| **iSCSI** |`service iscsid start` |啟動 iSCSI 服務 |
| &nbsp; |`service iscsid stop` |停止 iSCSI 服務 |
| &nbsp; |`service iscsid restart` |重新啟動 iSCSI 服務 |
| &nbsp; |`iscsiadm -m discovery -t sendtargets -p <TARGET_IP>` |探索指定位址上的可用目標 |
| &nbsp; |`iscsiadm -m node --login -T <TARGET_IQN>` |登入 iSCSI 目標 |
| &nbsp; |`iscsiadm -m node --logout -p <Target_IP>` |登出 iSCSI 目標 |
| &nbsp; |`cat /etc/iscsi/initiatorname.iscsi` |列印 iSCSI 啟動器名稱 |
| &nbsp; |`iscsiadm –m session –s <sessionid> -P 3` |檢查 iSCSI 工作階段的狀態以及在主機上探索到的磁碟區 |
| &nbsp; |`iscsi –m session` |顯示在主機與 StorSimple 裝置之間建立的所有 iSCSI 工作階段 |
|  | | |
| **多重路徑** |`service multipathd start` |啟動多重路徑 daemon |
| &nbsp; |`service multipathd stop` |停止多重路徑 daemon |
| &nbsp; |`service multipathd restart` |重新啟動多重路徑 daemon |
| &nbsp; |`chkconfig multipathd on` </br> 或 </br> `mpathconf –with_chkconfig y` |啟用多重路徑 daemon 以在開機時啟動 |
| &nbsp; |`multipathd –k` |啟動互動式主控台以進行疑難排解 |
| &nbsp; |`multipath –l` |列出多重路徑連接和裝置 |
| &nbsp; |`mpathconf --enable` |在 `/etc/mulitpath.conf` |
|  | | |

## <a name="next-steps"></a>後續步驟
當您在 Linux 主機上設定 MPIO 時，您可能也需要參考下列 CentoS 6.6 文件︰

* [在 CentOS 上設定 MPIO](http://www.centos.org/docs/5/html/5.1/DM_Multipath/setup_procedure.html)
* [Linux 訓練指南](http://linux-training.be/linuxsys.pdf)

