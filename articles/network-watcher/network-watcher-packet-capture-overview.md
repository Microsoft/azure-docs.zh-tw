---
title: Azure 網路監看員中的封包擷取簡介 | Microsoft Docs
description: 本頁提供網路監看員封包擷取功能的概觀
services: network-watcher
documentationcenter: na
author: jimdial
manager: timlt
editor: ''
ms.assetid: 3a81afaa-ecd9-4004-b68e-69ab56913356
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: jdial
ms.openlocfilehash: 152cc8fb61aa6115c7b5863e4d798db9e7aa5b7c
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23036833"
---
# <a name="introduction-to-variable-packet-capture-in-azure-network-watcher"></a>Azure 網路監看員中的變數封包擷取簡介

網路監看員變數封包擷取可讓您建立封包擷取工作階段來追蹤虛擬機器的流入和流出流量。 封包擷取有助於被動和主動地診斷網路異常。 其他用途包括收集網路統計資料、取得有關網路入侵的資訊，以及偵錯用戶端與伺服器間的通訊等等。

封包擷取是透過網路監看員從遠端啟動的虛擬機器擴充功能。 這項功能可以減輕在所需虛擬機器上手動執行封包擷取的工作負擔，進而省下寶貴的時間。 可以透過入口網站、PowerShell、CLI 或 REST API 觸發封包擷取。 觸發封包擷取方式的其中一個範例是使用虛擬機器警示。 系統會為擷取工作階段提供篩選器，以確保您擷取到您想要監視的流量。 篩選是根據 5 個 Tuple (通訊協定、本機 IP 位址、遠端 IP 位址、本機連接埠，以及遠端連接埠) 的資訊。 擷取的資料會儲存在本機磁碟或儲存體 blob。 每個訂用帳戶在每個區域皆有 10 個封包擷取工作階段的限制。 此限制僅適用於工作階段，且不會套用到已儲存的封包擷取檔案，無論該檔案是位於 VM 本機上還是位於儲存體帳戶中。

> [!IMPORTANT]
> 封包擷取需要虛擬機器擴充功能 `AzureNetworkWatcherExtension`。 若要在 Windows VM 上安裝擴充功能，請瀏覽[適用於 Windows 的 Azure 網路監看員代理程式虛擬機器擴充功能](../virtual-machines/windows/extensions-nwa.md)，若要在 Linux VM 上安裝，則請瀏覽[適用於 Linux 的 Azure 網路監看員代理程式虛擬機器擴充功能](../virtual-machines/linux/extensions-nwa.md)。

若要將擷取的資訊減少為只有您想要的資訊，可以針對封包擷取工作階段使用下列選項︰

**擷取設定**

|屬性|說明|
|---|---|
|**每個封包的最大位元組 (位元組)** | 來自每個封包所擷取的位元組，如果保留空白，會擷取所有位元組。 來自每個封包所擷取的位元組，如果保留空白，會擷取所有位元組。 如果您僅需要 IPv4 標頭 – 請在這裡指定 34 |
|**每個工作階段的最大位元組 (位元組)** | 所擷取其中的位元組總數，一旦達到值時工作階段隨即結束。|
|**時間限制 (秒)** | 在封包擷取工作階段上設定時間限制。 預設值為 18000 秒或 5 小時。|

**篩選 (選用)**

|屬性|說明|
|---|---|
|**通訊協定** | 用來篩選封包擷取的通訊協定。 可用的值為 TCP、UDP 和 All。|
|**本機 IP 位址** | 這個值會將封包擷取篩選為其中本機 IP 位址符合此篩選值的封包。|
|**本機連接埠** | 這個值會將封包擷取篩選為其中本機連接埠符合此篩選值的封包。|
|**遠端 IP 位址** | 這個值會將封包擷取篩選為其中遠端 IP 符合此篩選值的封包。|
|**遠端連接埠** | 這個值會將封包擷取篩選為其中遠端連接埠符合此篩選值的封包。|

### <a name="next-steps"></a>後續步驟

了解如何透過入口網站管理封包擷取，請造訪[在 Azure 入口網站中管理封包擷取](network-watcher-packet-capture-manage-portal.md)或若使用 PowerShell，請造訪[使用 PowerShell 管理封包擷取](network-watcher-packet-capture-manage-powershell.md)。

請造訪[建立觸發的警示封包擷取](network-watcher-alert-triggered-packet-capture.md)來了解如何根據虛擬機器警示建立主動式封包擷取

<!--Image references-->
[1]: ./media/network-watcher-packet-capture-overview/figure1.png













