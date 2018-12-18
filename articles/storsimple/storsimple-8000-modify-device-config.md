---
title: 修改 StorSimple 8000 系列裝置組態 | Microsoft Docs
description: 描述如何使用 StorSimple 裝置管理員服務來重新設定已部署的 StorSimple 裝置。
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 09/28/2017
ms.author: alkohli
ms.openlocfilehash: 13ff24c24a881297775fa5f65821e53ceb83c351
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23108756"
---
# <a name="use-the-storsimple-device-manager-service-to-modify-your-storsimple-device-configuration"></a>使用 StorSimple 裝置管理員服務來修改 StorSimple 裝置組態

## <a name="overview"></a>概觀

Azure 入口網站之 [設定] 刀鋒視窗中的 [裝置設定] 區段包含所有裝置參數，可讓您重新設定 StorSimple 裝置管理員服務所管理的 StorSimple 裝置。 本教學課程說明如何使用 [設定] 刀鋒視窗來執行下列裝置層級的工作：

* 修改裝置的易記名稱
* 修改裝置的時間設定
* 指派次要 DNS
* 修改網路介面
* 交換或重新指派 IP

## <a name="modify-device-friendly-name"></a>修改裝置的易記名稱

您可以使用 Azure 入口網站來變更裝置名稱，並對它指派您所選擇的唯一易記名稱。 使用裝置上的 [一般設定] 刀鋒視窗，可修改裝置的易記名稱。 易記名稱可以包含任何字元，且長度上限為 64 個字元。

> [!NOTE] 
> 裝置設定完成之前，您只能在 Azure 入口網站中修改裝置名稱。 一旦完成最基本的裝置設定，您就無法變更裝置名稱。

![[一般設定] 中的裝置名稱](./media/storsimple-8000-modify-device-config/modify-general-settings3.png)

系統會為連線到 StorSimple 裝置管理員服務的 StorSimple 裝置指派預設名稱。 預設名稱通常會反映裝置的序號。 例如，長度為 15 個字元的預設裝置名稱，如 8600-SHX0991003G44HT，將表示以下項目：

* **8600** – 表示裝置型號。
* **SHX** – 表示製造場所。
* **0991003** – 表示特定產品。
* **G44HT**– 最後 5 位數會以遞增方式來建立唯一的序號。 這可能不是連續的組合。

## <a name="modify-device-description"></a>修改裝置描述

使用裝置上的 [一般設定] 刀鋒視窗，可修改裝置的描述。

![[一般設定] 中的裝置描述](./media/storsimple-8000-modify-device-config/modify-general-settings4.png)

裝置描述通常有助於識別擁有者和裝置的實體位置。 [描述] 欄位不得超過 256 個字元。

## <a name="modify-time-settings"></a>修改時間設定

您的裝置必須同步時間才能驗證雲端儲存空間服務提供者。 使用裝置上的 [一般設定] 刀鋒視窗，可修改裝置的時間設定。

![[一般設定] 中的裝置描述](./media/storsimple-8000-modify-device-config/modify-general-settings2.png)

 從下拉式清單中選取時區。 您最多可以指定兩個網路時間通訊協定 (NTP) 伺服器：

 - **主要 NTP 伺服器** - 當您使用適用於 StorSimple 的 Windows PowerShell 來設定您的裝置時，需要指定此組態。 您可以指定預設 Windows Server **time.windows.com** 作為 NTP 伺服器。 您可以透過 Azure 入口網站來檢視主要 NTP 伺服器組態，但是必須使用 Windows PowerShell 介面來進行變更。 使用 `Set-HcsNTPClientServerAddress` Cmdlet 可修改裝置的主要 NTP 伺服器。 如需詳細資訊，請移至 [Set-HcsNTPClientServerAddress] (https://technet.microsoft.com/library/dn688138.aspx) Cmdlet 的語法。

- **次要 NTP 伺服器** - 此組態為選擇性。 您可以使用入口網站來設定次要 NTP 伺服器。

設定 NTP 伺服器時，請確定您的網路允許 NTP 流量從您的資料中心通過網際網路。 指定公用 NTP 伺服器時，您必須確定網路防火牆和其他安全性裝置皆設定為允許 NTP 流量傳入及傳出外部網路。 若不允許雙向 NTP 流量，您必須使用內部 NTP 伺服器 (Windows 網域控制器會提供這個函式)。 如果您的裝置無法同步時間，它可能無法與您的雲端儲存空間提供者進行通訊。

若要查看公用 NTP 伺服器清單，請移至 [NTP 伺服器網頁](http://support.ntp.org/bin/view/Servers/WebHome)。

### <a name="what-happens-if-the-device-is-deployed-in-a-different-time-zone"></a>如果裝置部署在不同的時區會發生什麼狀況？

如果裝置部署在不同的時區，系統將會變更裝置的時區。 假設所有備份原則皆使用裝置時區，則備份原則會根據新的時區自動調整。 不需使用者介入。

## <a name="modify-dns-settings"></a>修改 DNS 設定

當您的裝置嘗試與雲端儲存空間服務提供者通訊時，系統會使用 DNS 伺服器。 使用裝置上的 [網路設定] 刀鋒視窗可檢視和修改設定的 DNS 設定。 

![[網路設定] 中的 DNS 設定](./media/storsimple-8000-modify-device-config/modify-network-settings1.png)

如需高可用性，您必須在初始裝置部署期間設定主要和次要 DNS 伺服器。

**主要 DNS 伺服器** - 在初始設定期間，您使用適用於 StorSimple 的 Windows PowerShell 先指定主要 DNS 伺服器。 但是您只能透過 Windows PowerShell 介面重新設定主要 DNS 伺服器。 使用 `Set-HcsDNSClientServerAddress` Cmdlet 可修改裝置的主要 DNS 伺服器。 如需詳細資訊，請移至 [Set-HcsDNSClientServerAddress](https://technet.microsoft.com/library/dn688138.aspx) Cmdlet 的語法。

**次要 DNS 伺服器** - 若要修改次要 DNS 伺服器，請在裝置的 Windows PowerShell 介面中使用 `Set-HcsDNSClientServerAddress` Cmdlet，或使用 Azure 入口網站中 StorSimple 裝置的 [網路設定] 刀鋒視窗。

若要在 Azure 入口網站中修改次要 DNS 伺服器，請執行下列步驟。

1. 移至您的 StorSimple 裝置管理員服務。 從裝置清單中，選取並按一下您的裝置。

2. 在 [設定] 刀鋒視窗中，移至 [裝置設定] > [網路]。 這會開啟 [網路設定] 刀鋒視窗。 按一下 [DNS 設定] 圖格。 修改次要 DNS 伺服器 IP 位址。

    ![修改次要 DNS 伺服器 IP 位址](./media/storsimple-8000-modify-device-config/modify-secondary-dns1.png)

4. 從命令列按一下 [儲存]，提示您確認時，請按一下 [確定]。

    ![儲存並確認變更](./media/storsimple-8000-modify-device-config/modify-secondary-dns-2.png)



## <a name="modify-network-interfaces"></a>修改網路介面

您的裝置具有六個裝置網路介面，其中四個是 1 GbE，而其中兩個是 10 Gbe。 這些介面都會標示為 DATA 0 至 DATA 5。 DATA 0、DATA 1、DATA 4 和 DATA 5 是 1 GbE，而 DATA 2 和 DATA 3 是 10 GbE 網路介面。

使用 [網路設定] 刀鋒視窗可設定每個要使用的介面。

![透過 [網路設定] 設定網路介面](./media/storsimple-8000-modify-device-config/modify-network-settings3.png)

若要確保高可用性，建議您的裝置上至少具有兩個 iSCSI 介面以及兩個已啟用雲端功能的介面。 我們建議停用未使用的介面，但並非必要。

對於每個網路介面，系統會顯示下列參數：

* **速度** – 並非使用者可設定的參數。 DATA 0、DATA 1、DATA 4 和 DATA 5 永遠是 1 GbE，而 DATA 2 和 DATA 3 是 10 GbE 介面。
  
  > [!NOTE]
  > 速度和雙工會一律自動交涉。 不支援 Jumbo 框架。
  
* **介面狀態** – 介面可以啟用或停用。 如果已啟用，則裝置會嘗試使用介面。 我們建議您僅啟用那些連線到網路的使用中介面。 停用任何您沒有使用的介面。
* **介面類型** – 此參數可讓您隔離 iSCSI 流量和雲端儲存空間流量。 此參數可以是下列其中一項：
  
  * **啟用雲端** – 啟用時，裝置將使用此介面與雲端通訊。
  * **啟用 iSCSI** – 啟用時，裝置將使用此介面與 iSCSI 主機通訊。
    
    我們建議您隔離 iSCSI 流量與雲端儲存空間流量。 也請注意如果您的主機位於裝置所在的相同子網路內，您不需要指派閘道器；不過，如果您的主機位於與裝置不同的子網路，您將必須指派一個閘道器。
* **IP 位址** – 當您設定任何網路介面時，必須設定虛擬 IP (VIP)。 這可以是 IPv4 或 IPv6 或同時使用。 IPv4 和 IPv6 位址系列皆支援裝置網路介面。 當您使用 IPv4 時，請以小數點十進位表示法指定 32 位元的 IP 位址 (*xxx.xxx.xxx.xxx*)。 當您使用 IPv6 時，僅需提供 4 位數前置詞，而裝置網路介面的 128 位元位址將會根據該前置詞自動產生。
* **子網路** – 這是指子網路遮罩，且透過 Windows PowerShell 介面設定。
* **閘道器** – 這是在嘗試與位於不同 IP 位址空間 (子網路) 內的節點通訊時此介面應該使用的預設閘道器。 預設閘道器必須位於相同的位址空間 (子網路)，如同介面 IP 位址，且取決於子網路遮罩。
* **固定 IP 位址** – 僅當您設定 DATA 0 介面時才能使用此欄位。 對於更新或疑難排解裝置等作業，您可能需要直接連線到裝置控制器。 固定 IP 位址可用來存取裝置上的主動和被動控制器。

> [!NOTE]
> * 若要確保適當的操作，請確認每個裝置介面所連線到交換器上的介面速度和雙工。 交換器介面應該針對乙太網路 (1000 Mbps) 進行交涉或設定，且為全雙工。 介面以較低速度運作或為半雙工時將導致效能問題。
> * 若要將中斷與停機時間降到最低，建議您針對裝置的 iSCSI 網路介面將會連接到的每個交換器連接埠上啟用 portfast。 如此可確保網路連線可以在發生容錯移轉時快速建立。

### <a name="configure-data-0"></a>設定 DATA 0

DATA 0 依預設已啟用雲端功能。 設定 DATA 0 時，您也需要設定兩個固定 IP 位址，每個控制器使用一個。 這些固定的 IP 位址可以用來直接存取裝置控制器，且當您在裝置上安裝更新，讓記憶體回收正常運作時，或當您存取控制器以進行疑難排解時很有用。

您可以透過 DATA 0 設定刀鋒視窗，重新設定固定的 IP 控制器。

![設定網路介面 - DATA 0](./media/storsimple-8000-modify-device-config/modify-network-settings2.png)

> [!NOTE]
> 控制器的固定 IP 位址可用來提供更新給裝置，以及讓空間回收演算法 (記憶體回收) 得以正常運作。 因此，固定 IP 必須可路由傳送且能夠連線到網際網路。

### <a name="configure-data-1---data-5"></a>設定 DATA 1 - DATA 5

您可以設定 DATA 1 - DATA 5 網路介面的所有網路設定，如以下螢幕擷取畫面所示：

![設定網路介面 DATA 1 - DATA 5](./media/storsimple-8000-modify-device-config/modify-network-settings4.png)


## <a name="swap-or-reassign-ips"></a>交換或重新指派 IP

目前，如果控制器上的任何網路介面已指派使用中的 VIP (由相同裝置或網路中的另一個裝置)，則控制器會進行容錯移轉。 如果您交換或重新指派裝置網路介面的 VIP 時，必須遵循適當的程序，否則可能會建立重複的 IP。

執行下列步驟來交換或重新指派任何網路介面的 VIP：

#### <a name="to-reassign-ips"></a>重新指派 IP

1. 清除兩個介面的 IP 位址。
2. 清除 IP 位址之後，指派新的 IP 位址給個別的介面。

## <a name="next-steps"></a>後續步驟

* 了解如何 [為 StorSimple 裝置設定 MPIO](storsimple-8000-configure-mpio-windows-server.md)。
* 了解如何[使用 StorSimple 裝置管理員服務管理 StorSimple 裝置](storsimple-8000-manager-service-administration.md)。

