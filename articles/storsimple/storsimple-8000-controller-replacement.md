---
title: 更換 StorSimple 8000 系列裝置控制器 | Microsoft Docs
description: 說明如何取下並更換 StorSimple 8000 系列裝置上的一個或兩個控制器模組。
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
ms.workload: TBD
ms.date: 06/05/2017
ms.author: alkohli
ms.openlocfilehash: 849eccff114c2fd6d952e44d095d0cc89a238675
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23109126"
---
# <a name="replace-a-controller-module-on-your-storsimple-device"></a>更換 StorSimple 裝置上的控制器模組
## <a name="overview"></a>概觀
本教學課程說明如何取下並更換 StorSimple 裝置中的一個或兩個控制器模組。 它也會討論單一和雙重控制器更換案例的基礎邏輯。

> [!NOTE]
> 在執行控制器更換之前，我們建議您一律將控制器韌體更新至最新版本。
> 
> 若要避免損害您的 StorSimple 裝置，請勿退出控制器，直到 LED 顯示成下列其中一個：
> 
> * 所有燈都已關閉。
> * LED 3、![綠色勾號圖示](./media/storsimple-controller-replacement/HCS_GreenCheckIcon.png)和![紅色叉號圖示](./media/storsimple-controller-replacement/HCS_RedCrossIcon.png)是閃爍，而 LED 0 和 LED 7 是**開啟**。


下表顯示支援的控制器更換案例。

| 案例 | 更換案例 | 適用程序 |
|:--- |:--- |:--- |
| 1 |一個控制器處於故障狀態，而另一個控制器為狀況良好並作用中。 |[單一控制器更換](#replace-a-single-controller)，其中描述[單一控制器更換背後的邏輯](#single-controller-replacement-logic)，以及[更換步驟](#single-controller-replacement-steps)。 |
| 2 |兩個控制器都故障，而且需要更換。 底座、磁碟和磁碟機箱的狀況良好。 |[雙重控制器更換](#replace-both-controllers)，其中描述[雙重控制器更換背後的邏輯](#dual-controller-replacement-logic)，以及[更換步驟](#dual-controller-replacement-steps)。 |
| 3 |交換來自相同裝置或來自不同裝置的控制器。 底座、磁碟和磁碟機箱的狀況良好。 |插槽不符警示訊息將出現。 |
| 4 |遺漏一個控制器，而另一個控制器故障。 |[雙重控制器更換](#replace-both-controllers)，其中描述[雙重控制器更換背後的邏輯](#dual-controller-replacement-logic)，以及[更換步驟](#dual-controller-replacement-steps)。 |
| 5 |一個或兩個控制器故障。 您無法透過序列主控台或 Windows PowerShell 遠端存取裝置。 |[連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md) 。 |
| 6 |控制器有不同的組建版本，原因可能是︰<ul><li>控制器有不同的軟體版本。</li><li>控制器有不同的韌體版本。</li></ul> |如果控制器軟體版本不同，更換邏輯會偵測到並更新更換控制器上的軟體版本。<br><br>如果控制器軔體版本不同，而且舊的韌體版本**無法**自動升級，則 Azure 入口網站中會出現警示訊息。 您應掃描更新並安裝韌體更新。</br></br>如果控制器軔體版本不同，而且舊的韌體版本可以自動升級，控制器更換邏輯會偵測到此情況，並且在控制器啟動之後，軔體將會自動更新。 |

您需要取下故障的控制器模組。 一或兩個控制器模組可能故障，這會導致單一控制器更換或雙重控制器更換。 如需更換程序和其背後邏輯的資訊，請參閱以下各項：

* [更換單一控制器](#replace-a-single-controller)
* [更換兩個控制器](#replace-both-controllers)
* [取下控制器](#remove-a-controller)
* [插入控制器](#insert-a-controller)
* [識別您裝置上的作用中控制器](#identify-the-active-controller-on-your-device)

> [!IMPORTANT]
> 取下及更換控制器之前，請閱讀 [StorSimple 硬體元件更換](storsimple-8000-hardware-component-replacement.md)中的安全資訊。
> 
> 

## <a name="replace-a-single-controller"></a>更換單一控制器
當 Microsoft Azure StorSimple 裝置上的兩個控制器之一故障、無法運作或遺漏時，您必須更換單一控制器。

### <a name="single-controller-replacement-logic"></a>單一控制器更換邏輯
在單一控制器更換中，您應該先取下故障的控制器。 (裝置中剩餘的控制器是作用中控制器)。當插入更換控制器時，會發生下列動作：

1. 更換控制器立即開始與 StorSimple 裝置進行通訊。
2. 作用中控制器的虛擬硬碟 (VHD) 快照會複製在更換控制器上。
3. 會修改快照，以便當更換控制器從這個 VHD 啟動時，系統會將它會辨識為待命控制器。
4. 完成修改後，更換控制器將啟動為待命控制器。
5. 兩個控制器同時執行時，叢集就會恢復上線。

### <a name="single-controller-replacement-steps"></a>單一控制器更換步驟
如果 Microsoft Azure StorSimple 裝置的其中一個控制器故障，請完成下列步驟。 (另一個控制器必須作用中並執行中。 如果兩個控制器都故障或無法運作，請移至 [雙重控制器更換步驟](#dual-controller-replacement-steps))。

> [!NOTE]
> 可能需要 30 – 45 分鐘，控制器才會重新啟動，並從單一控制器更換程序完全復原。 整個程序的時間總計 (包括接上纜線) 大約 2 小時。


#### <a name="to-remove-a-single-failed-controller-module"></a>若要取下單一故障的控制器模組
1. 在 Azure 入口網站中，移至 StorSimple 裝置管理員服務，按一下 [裝置]，然後按一下您想要監視的裝置名稱。
2. 移至 [監視] > [硬體健康狀態]。 控制器 0 或控制器 1 的狀態應該是紅色，表示故障。
   
   > [!NOTE]
   > 單一控制器更換中的故障控制器一律為待命控制器。
   
3. 使用圖 1 和下表來找出故障的控制器模組。
   
    ![裝置主要機箱模組的後擋板](./media/storsimple-controller-replacement/IC740994.png)
   
    **圖 1** StorSimple 裝置的背面
   
   | 標籤 | 說明 |
   |:--- |:--- |
   | 1 |PCM 0 |
   | 2 |PCM 1 |
   | 3 |控制器 0 |
   | 4 |控制器 1 |
4. 在故障的控制器上，從資料連接埠取下所有已連接的網路纜線。 如果您是使用 8600 機型，也請取下將控制器連接至 EBOD 控制器的SAS 纜線。
5. 依照[取下控制器](#remove-a-controller)中的步驟，取下故障的控制器。
6. 在取下故障控制器的同一插槽中安裝原廠更換品。 這樣會觸發單一控制器更換邏輯。 如需詳細資訊，請參閱[單一控制器更換邏輯](#single-controller-replacement-logic)。
7. 當單一控制器更換邏輯在背景中進行時，請重新連接纜線。 請完全依照更換之前連接纜線的相同方式，小心地連接所有纜線。
8. 在控制器重新啟動之後，請檢查 Azure 入口網站中的 [控制器狀態] 和 [叢集狀態]，以確認控制器回到良好的狀態且處於待命模式。

> [!NOTE]
> 如果您是透過序列主控台監視裝置，則可能會在控制器從更換程序中復原時看到多次重新啟動。 當序列主控台功能表呈現時，您便知道更換已完成。 如果功能表未在啟動控制器更換的兩個小時內出現，請[連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md)。
>
> 從 Update 4 開始，您也可以在裝置的 Windows PowerShell 介面中使用 Cmdlet `Get-HCSControllerReplacementStatus` 來監視控制器更換程序的狀態。
> 

## <a name="replace-both-controllers"></a>更換兩個控制器
當 Microsoft Azure StorSimple 裝置上的兩個控制器故障、無法運作或遺漏時，您必須更換兩個控制器。 

### <a name="dual-controller-replacement-logic"></a>雙重控制器更換
在雙重控制器更換中，先移除兩個故障的控制器，再插入更換控制器。 當插入兩個更換控制器時，會發生下列動作：

1. 插槽 0 中的更換控制器會檢查下列情況：
   
   1. 它是否使用目前版本的韌體和軟體？
   2. 它是否為叢集的一部分？
   3. 對等控制器是否執行中並構成叢集？
      
      如果上述狀況無一成立，則控制器會尋找最新的每日備份 (位於磁碟機 S 上的 **nonDOMstorage** )。 控制器會從備份複製 VHD 的最新快照。
2. 在位置 0 的控制站會使用快照集映像本身。
3. 同時，插槽 1 中的控制器會等到控制器 0 完成映像和啟動。
4. 在控制器 0 啟動之後，控制器 1 會偵測到控制器 0 所建立的叢集，這會觸發單一控制器更換邏輯。 如需詳細資訊，請參閱 [單一控制器更換邏輯](#single-controller-replacement-logic)。
5. 之後，兩個兩個控制器將執行中，而且叢集將恢復上線。

> [!IMPORTANT]
> 在雙重控制器更換後，於設定 StorSimple 裝置之後，務必進行裝置的手動備份。 直到過了 24 小時之後，才會觸發每日裝置組態備份。 與 [Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md) 合作，進行裝置的手動備份。


### <a name="dual-controller-replacement-steps"></a>雙重控制器更換步驟
當 Microsoft Azure StorSimple 裝置中的兩個控制器都故障時，需要這個工作流程。 這可能會發生在冷卻系統停止運作的資料中心，因此兩個控制器會在短時間內故障。 視 StorSimple 裝置是關閉還是開啟，以及您使用的是 8600 還是 8100 機型而定，會需要一組不同的步驟。

> [!IMPORTANT]
> 可能需要 45 分鐘到 1 小時，控制器才會重新啟動，並從雙重控制器更換程序完全復原。 整個程序的時間總計 (包括接上纜線) 大約 2.5 小時。

#### <a name="to-replace-both-controller-modules"></a>若要取下兩個控制器模組
1. 如果裝置已關閉，請略過此步驟並繼續進行下一個步驟。 如果裝置已開啟，請關閉裝置。
   
   1. 如果您使用的是 8600 機型，請先關閉主要機箱，然後再關閉 EBOD 機箱。
   2. 等到裝置完全關閉。 裝置背面的所有 LED 都將關閉。
2. 取下所有已連接至資料連接埠的網路纜線。 如果您使用的是 8600 機型，請一併取下將主要機箱連接至 EBOD 機箱的 SAS 纜線。
3. 從 StorSimple 裝置取下兩個控制器。 如需詳細資訊，請參閱[取下控制器](#remove-a-controller)。
4. 首先插入控制器 0 的原廠更換品，再插入控制器 1。 如需詳細資訊，請參閱[插入控制器](#insert-a-controller)。 這樣會觸發雙重控制器更換邏輯。 如需詳細資訊，請參閱[雙重控制器更換邏輯](#dual-controller-replacement-logic)。
5. 當雙重控制器更換邏輯在背景中進行時，請重新連接纜線。 請完全依照更換之前連接纜線的相同方式，小心地連接所有纜線。 請參閱[安裝 StorSimple 8100 裝置](storsimple-8100-hardware-installation.md)或[安裝 StorSimple 8600 裝置](storsimple-8600-hardware-installation.md)的＜佈線您的裝置＞一節中，您機型適用的詳細指示。
6. 開啟 StorSimple 裝置。 如果您使用的是 8600 機型：
   
   1. 確定首先開啟 EBOD 機箱。
   2. 等到 EBOD 機箱執行。
   3. 開啟主要機箱。
   4. 在第一個控制器重新啟動並處於狀況良好的狀態之後，系統就會執行。
      
      > [!NOTE]
      > 如果您是透過序列主控台監視裝置，則可能會在控制器從更換程序中復原時看到多次重新啟動。 當序列主控台功能表出現時，您便知道更換已完成。 如果功能表未在啟動控制器更換的 2.5 個小時內出現，請[連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md)。
     
## <a name="remove-a-controller"></a>取下控制器
請使用下列程序，從 StorSimple 裝置中取下故障的控制器模組。

> [!NOTE]
> 下圖適用於控制器 0。 若為控制器 1，這些將相反。


#### <a name="to-remove-a-controller-module"></a>若要取下控制器模組
1. 以姆指與食指抓住模組閂鎖。
2. 輕輕擠壓姆指與食指，以鬆開控制器閂鎖。
   
    ![鬆開控制器閂鎖](./media/storsimple-controller-replacement/IC741047.png)
   
    **圖 2** 鬆開控制器閂鎖
3. 請使用閂鎖做為把手，將控制器滑出底座。
   
    ![將控制器滑出底座](./media/storsimple-controller-replacement/IC741048.png)
   
    **圖 3** 將控制器滑出底座

## <a name="insert-a-controller"></a>插入控制器
在您從 StorSimple 裝置取下故障模組之後，請使用下列程序來安裝原廠提供的控制器模組。

#### <a name="to-install-a-controller-module"></a>若要安裝控制器模組
1. 查看介面連接器是否有任何損毀。 如果有任一連接器接腳壞掉或彎曲，請勿安裝該模組。
2. 當閂鎖完全鬆開時，請將控制器模組滑入底座。
   
    ![將控制器滑入底座](./media/storsimple-controller-replacement/IC741053.png)
   
    **圖 4** 將控制器滑入底座
3. 一旦插入控制器模組，請馬上關閉閂鎖，同時繼續將控制器模組推入底座。 閂鎖將扣上，以將控制器固定位。
   
    ![關閉控制器閂鎖](./media/storsimple-controller-replacement/IC741054.png)
   
    **圖 5** 關閉控制器閂鎖
4. 當閂鎖卡入定位時，即表示完成。 **正常** LED 現在應該亮起。
   
   > [!NOTE]
   > 最多可能需要 5 分鐘，控制器和 LED 即會啟動。
  
5. 若要確認更換成功，請在 Azure 入口網站中，瀏覽至 [監視] > [硬體健康狀態]，並確定控制器 0 及控制器 1 兩者都狀況良好 (狀態為綠色)。

## <a name="identify-the-active-controller-on-your-device"></a>識別您裝置上的作用中控制器
有許多情況，例如第一次裝置註冊或控制器更換，會要求您在 StorSimple 裝置上找出作用中控制器。 作用中控制器會處理所有磁碟韌體和網路作業。 您可以使用下列任一方法來識別作用中控制器：

* [使用 Azure 入口網站來識別作用中控制器](#use-the-azure-portal-to-identify-the-active-controller)
* [使用 Windows PowerShell for StorSimple 來識別作用中控制器](#use-windows-powershell-for-storsimple-to-identify-the-active-controller)
* [檢查實體裝置來識別作用中控制器](#check-the-physical-device-to-identify-the-active-controller)

接著說明上述各程序。

### <a name="use-the-azure-portal-to-identify-the-active-controller"></a>使用 Azure 入口網站來識別作用中控制器
在 Azure 入口網站中，瀏覽至您的裝置，並移至 [監視] > [硬體健康狀態]，捲動至 [控制器]區段。 在這裡您可以確認哪一個控制站作用中。

![識別 Azure 入口網站中的作用中控制器](./media/storsimple-controller-replacement/IC752072.png)

**圖 6** 顯示作用中控制器的 Azure 入口網站

### <a name="use-windows-powershell-for-storsimple-to-identify-the-active-controller"></a>使用 Windows PowerShell for StorSimple 來識別作用中控制器
透過序列主控台存取您的裝置時，會呈現橫幅訊息。 橫幅訊息包含基本裝置資訊，例如：型號、名稱、已安裝的軟體版本、您要存取的控制器的狀態等。 下圖顯示橫幅訊息的範例：

![序列橫幅訊息](./media/storsimple-controller-replacement/IC741098.png)

**圖 7** 橫幅訊息將控制器 0 顯示為作用中

您可以使用橫幅訊息，來判定您所連接的控制器是主動還是被動。

### <a name="check-the-physical-device-to-identify-the-active-controller"></a>檢查實體裝置來識別作用中控制器
若要識別裝置上的作用中控制器，請在主要機箱背面找出 DATA 5 連接埠上的藍色 LED。

如果此 LED 閃爍，控制器是作用中，而且另一個控制器處於待命模式。 使用下圖和下表來提供協助。

![包含資料連接埠的裝置主要機箱後擋板](./media/storsimple-controller-replacement/IC741055.png)

**圖 8** 具有資料連接埠和監視 LED 的主要機箱背面

| 標籤 | 說明 |
|:--- |:--- |
| 1-6 |DATA 0 – 5 個網路連接埠 |
| 7 |藍色 LED |

## <a name="next-steps"></a>後續步驟
深入了解 [StorSimple 硬體元件更換](storsimple-8000-hardware-component-replacement.md)。

