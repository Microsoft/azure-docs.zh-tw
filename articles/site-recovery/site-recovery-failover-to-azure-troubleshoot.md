---
title: 針對容錯移轉至 Azure 失敗進行疑難排解 | Microsoft Docs
description: 本文說明如何針對容錯移轉至 Azure 的常見錯誤進行疑難排解
services: site-recovery
documentationcenter: ''
author: ponatara
manager: abhemraj
editor: ''
ms.assetid: ''
ms.service: site-recovery
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: storage-backup-recovery
ms.date: 09/11/2018
ms.author: ponatara
ms.openlocfilehash: 6be71424e30c5783a03b157171b3f5acd0160e65
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44391005"
---
# <a name="troubleshoot-errors-when-failing-over-a-virtual-machine-to-azure"></a>針對將虛擬機器容錯移轉至 Azure 時的錯誤進行疑難排解

將虛擬機器容錯移轉至 Azure 時，您可能會收到下列錯誤。 若要進行疑難排解，請使用針對每種錯誤狀況所述的步驟。

## <a name="failover-failed-with-error-id-28031"></a>容錯移轉失敗，錯誤識別碼為 28031

Site Recovery 無法在 Azure 中建立已容錯移轉的虛擬機器。 它可能會因為下列其中一個原因而發生：

* 沒有足夠的配額可用來建立虛擬機器：您可以移至 [訂用帳戶] -> [使用量 + 配額]，檢查可用的配額。 您可以開啟[新的支援要求](http://aka.ms/getazuresupport)來增加配額。

* 您嘗試在相同的可用性設定組中容錯移轉不同大小系列的虛擬機器。 確保您為相同可用性設定組中的所有虛擬機器，選擇相同的大小系列。 移至虛擬機器的 [計算] 和 [網路] 來變更大小，然後重試容錯移轉。

* 有訂用帳戶原則可避免建立虛擬機器。 將此原則變更為允許建立虛擬機器，然後重試容錯移轉。

## <a name="failover-failed-with-error-id-28092"></a>容錯移轉失敗，錯誤識別碼為 28092

Site Recovery 無法針對已容錯移轉的虛擬機器建立網路介面。 確定您有足夠的配額可用來在訂用帳戶中建立網路介面。 您可以移至 [訂用帳戶] -> [使用量 + 配額]，檢查可用的配額。 您可以開啟[新的支援要求](http://aka.ms/getazuresupport)來增加配額。 如果您有足夠的配額，這可能是間歇性問題，請重試一次此作業。 如果即使在重試之後，問題仍然存在，則在本文的結尾留言。  

## <a name="failover-failed-with-error-id-70038"></a>容錯移轉失敗，錯誤識別碼為 70038

Site Recovery 無法在 Azure 中建立已容錯移轉的傳統虛擬機器。 其發生原因可能是：

* 其中一個資源 (例如建立虛擬機器所需的虛擬網路) 不存在。 建立如虛擬機器的 [計算] 和 [網路] 設定之下提供的虛擬網路，或將此設定修改為已經存在的虛擬網路，然後重試容錯移轉。

## <a name="unable-to-connectrdpssh---vm-connect-button-grayed-out"></a>無法連線/RDP/SSH - [VM 連線] 按鈕呈現灰色

如果 Azure 中容錯移轉虛擬機器上的 [連線] 按鈕呈現灰色，而您未透過 Express Route 或網站間 VPN 連線來連線到 Azure，則請：

1. 移至 [虛擬機器] > [網路]，按一下所需網路介面的名稱。  ![network-interface](media/site-recovery-failover-to-azure-troubleshoot/network-interface.PNG)
2. 瀏覽至 [IP 組態]，然後按一下所需 IP 組態的名稱欄位。 ![IPConfigurations](media/site-recovery-failover-to-azure-troubleshoot/IpConfigurations.png)
3. 若要啟用公用 IP 位址，請按一下 [啟用]。 ![啟用 IP](media/site-recovery-failover-to-azure-troubleshoot/Enable-Public-IP.png)
4. 按一下 [設定必要設定] > [建立新項目]。 ![建立新項目](media/site-recovery-failover-to-azure-troubleshoot/Create-New-Public-IP.png)
5. 輸入公用位址的名稱，選擇 [SKU] 和 [指派] 的預設選項，然後按一下 [確定]。
6. 現在，為了儲存所做的變更，請按一下 [儲存]。
7. 關閉面板，然後瀏覽至虛擬機器的 [概觀] 區段來進行連線/RDP。

## <a name="unable-to-connectrdpssh---vm-connect-button-available"></a>無法連線/RDP/SSH - [VM 連線] 按鈕可供使用

如果 Azure 中容錯移轉虛擬機器上的 [連線] 按鈕可供使用 (未呈現灰色)，請檢查虛擬機器上的 [開機診斷]，並檢查[本文](../virtual-machines/windows/boot-diagnostics.md)中列出的錯誤。

1. 如果虛擬機器尚未啟動，請嘗試容錯移轉至較舊的復原點。
2. 如果虛擬機器內的應用程式未啟動，請嘗試容錯移轉至與應用程式一致的復原點。
3. 如果虛擬機器已加入網域，請確定網域控制站正確運作。 做法是依照以下提供的步驟進行操作：

    a. 在相同的網路中建立新的虛擬機器。

    b.  確定它能夠加入預期啟動已容錯移轉之虛擬機器的相同網域。

    c. 如果網域控制站**未**正常運作，請使用本機系統管理員帳戶來嘗試登入已容錯移轉的虛擬機器。
4. 如果您使用自訂的 DNS 伺服器，則請確定其可供連線。 做法是依照以下提供的步驟進行操作：

    a. 在相同的網路中建立新的虛擬機器，以及

    b. 檢查虛擬機器是否能夠使用自訂 DNS 伺服器進行名稱解析

>[!Note]
>若要啟用「開機診斷」以外的任何設定，將必須於容錯移轉之前，在虛擬機器中安裝「Azure VM 代理程式」

## <a name="unexpected-shutdown-message-event-id-6008"></a>非預期性關閉訊息 (事件識別碼 6008)

啟動 Windows VM 容錯移轉後，如果在復原的 VM 上收到非預期性關閉訊息，表示未在用於容錯移轉的復原點中擷取 VM 關閉狀態。 當您復原至 VM 未完全關閉的時間點時，就會發生這種情況。

一般而言，無須擔心此問題，對於非計劃性容錯移轉，通常可忽略此問題。 在計劃性容錯移轉的情況下，請確保在容錯移轉之前，正確關閉虛擬機器，並為擱置的複寫資料內部部署提供足夠的時間，將其傳送至 Azure。 然後使用 [容錯移轉](site-recovery-failover.md#run-a-failover) 畫面控制項 上的 [最新] 選項，以便將 Azure 上的任何擱置資料處理到復原點，然後將其用於 VM 容錯移轉。

## <a name="retaining-drive-letter-after-failover"></a>在容錯移轉之後保留磁碟機代號
若要在容錯移轉後，保留虛擬機器上的磁碟機代號，您可以將虛擬機器內部部署的 **SAN 原則**設為 **OnlineAll**。 [閱讀更多資訊](https://support.microsoft.com/help/3031135/how-to-preserve-the-drive-letter-for-protected-virtual-machines-that-are-failed-over-or-migrated-to-azure)。

## <a name="next-steps"></a>後續步驟
- 對 [Windows VM 的 RDP 連線](../virtual-machines/windows/troubleshoot-rdp-connection.md)進行移難排解
- 對 [Linux VM 的 SSH 連線](../virtual-machines/linux/detailed-troubleshoot-ssh-connection.md)進行疑難排解

如果您需要更多說明，則將您的查詢張貼在 [Site Recovery 論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=hypervrecovmgr)上或在本文件的結尾留言。 我們有一個能夠協助您的使用中社群。
