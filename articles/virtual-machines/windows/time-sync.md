---
title: Azure 中 Windows VM 的時間同步 | Microsoft Docs
description: Windows 虛擬機器的時間同步。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 09/017/2018
ms.author: cynthn
ms.openlocfilehash: 7fadf4a8bcf545229dd604829780e9837ad5a94a
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45987309"
---
# <a name="time-sync-for-windows-vms-in-azure"></a>Azure 中 Windows VM 的時間同步

時間同步對於安全性和事件相互關聯性而言相當重要。 有時候時間同步會用於分散式交易實作。 多個電腦系統之間的時間精確度是透過同步處理來達成。 同步處理可能會受到多種因素影響，包括重新開機以及時間來源和擷取時間的電腦之間的網路流量。 

Azure 現在支援採用 Windows Server 2016 的基礎結構。 Windows Server 2016 的演算法經過改良，用來修正時間和調整本機時鐘以便與 UTC 同步。  Windows Server 2016 也改良了 VMICTimeSync 服務，可控管 VM 與主機同步以獲得準確時間的方式。 改良功能包括更精確的 VM 開始或 VM 還原初始時間，以及提供給 Windows 時間 (W32time) 的樣本插斷延遲修正。 


>[!NOTE]
>如需 Windows 時間服務的快速概觀，請觀看本[高階概觀影片](https://aka.ms/WS2016TimeVideo)。
>
> 如需詳細資訊，請參閱 [Windows Server 2016 的準確時間](https://docs.microsoft.com/windows-server/networking/windows-time-service/accurate-time)。 

## <a name="overview"></a>概觀

電腦時鐘精確度的衡量方式，是依據電腦時鐘與國際標準時間 (UTC) 時間標準的接近程度。 UTC 是由多國的精準原子鐘樣本所定義；原子鐘在 300 年內只會誤差一秒。 但是直接讀取 UTC 需使用專用的硬體。 替代方式為系統將時間伺服器同步為 UTC，並從其他電腦存取以提供延展性和強固性。 每部電腦都有持續運作的時間同步處理服務，知道需使用哪些時間伺服器，且會定期檢查電腦時鐘是否需要修正，並視需要調整時間。 

Azure 主機會與內部 Microsoft 時間伺服器同步，這些時間伺服器會透過 GPS 天線從 Microsoft 所屬的 Stratum 1 裝置擷取時間。 Azure 中的虛擬機器可以依賴其主機將準確的時間 (*主機時間*) 傳遞至 VM，也可以由 VM 直接從時間伺服器取得時間，或結合兩種方式。 

虛擬機器與主機的互動也可能會影響時鐘。 進行[記憶體保留維修](maintenance-and-updates.md#memory-preserving-maintenance)期間，VM 會暫停最多 30 秒。 比方說，維修開始之前 VM 時鐘顯示上午 10:00:00，並持續 28 秒。 VM 繼續執行後，VM 上的時鐘仍會顯示上午 10:00:00，也就是有 28 秒的誤差。 為了修正這個誤差，VMICTimeSync 服務會監視主機上發生的狀況，並提示您為了進行補償需在 VM 上做的變更。

若沒有時間同步功能，VM 上的時鐘將會累積錯誤。 若只有一部 VM，除非工作負載需要高度精準的計時功能，否則不會造成顯著的影響。 但是在多數情況下，我們都有多部互相連接並使用時間來追蹤交易的 VM，而且整個部署中的時間必須保持一致。 當 VM 之間的時間有所不同，可能會產生以下影響：

- 驗證將會失敗。 Kerberos 這類安全性通訊協定或憑證相依的技術，仰賴整個系統中的時間維持一致。 
- 如果記錄 (或其他資料) 認知的時間有所差異，很難想像系統會發生什麼問題。 同一個事件看起來可能像發生在不同時間，導致難以建立相互關聯。
- 當時鐘有所誤差，計費功能也可能會計算錯誤。

採用 Windows Server 2016 做為客體作業系統可達成 Windows 部署的最佳結果，Windows Server 2016 確保您可使用最新的時間同步處理改良功能。

## <a name="configuration-options"></a>組態選項

為裝載於 Azure 中 Windows VM 設定時間同步時有三個選項可選：

- 主機時間和 time.windows.com。 這是 Azure Marketplace 映像中的預設設定。
- 僅限主機。
- 使用另一個外部時間伺服器並搭配或不搭配使用主機時間。


### <a name="use-the-default"></a>使用預設值

預設的 Windows OS VM 映像已針對 w32time 設為從兩個來源進行同步處理： 

- NtpClient 提供者，其會向 time.windows.com 取得資訊。
- VMICTimeSync 服務，用來將主機時間傳遞給 VM，並在 VM 暫停以進行維修後予以修正。 Azure 主機使用 Microsoft 所屬的 Stratum 1 裝置來維持時間的準確性。

w32time 會依照以下優先順序決定偏好的時間提供者：組織層層級、根延遲、根散佈、時間位移。 在大部分情況下，w32time 偏好 time.windows.com 而非主機，因為 time.windows.com 會報告較低層級的組織層。 

針對加入網域的機器，網域本身會建立時間同步階層，但樹系根仍需從其他地方取得時間，且以下考量仍適用。


### <a name="host-only"></a>僅限主機 

由於 time.windows.com 為公用 NTP 伺服器，與其同步時間需透過網際網路傳送流量，而不同的封包延遲可能會對時間同步的品質造成負面影響。切換為僅限主機同步以移除 time.windows.com，可改善時間同步結果。

如果在使用預設設定時發生時間同步問題，切換為僅限主機時間同步為合理的解決方式。 嘗試使用僅限主機同步，確認是否可改善 VM 上的時間同步結果。 

### <a name="external-time-server"></a>外部時間伺服器

如果您有特定的時間同步需求，另提供使用外部時間伺服器的選項。 外部時間伺服器可提供特定的時間，用於測試案例相當實用，可確保與裝載於非 Microsoft 資料中心的機器之間的時間一致性，或以特殊方式處理閏秒。

您可以將外部伺服器與 VMICTimeSync 服務和 VMICTimeProvide 結合，以便提供與預設設定類似的結果。 

## <a name="check-your-configuration"></a>檢查設定


檢查 NtpClient 時間提供者是設定為使用明確的 NTP 伺服器 (NTP)，還是使用網域時間同步 (NT5DS)。

```
w32tm /dumpreg /subkey:Parameters | findstr /i "type"
```

如果 VM 使用 NTP，您會看到下列輸出：

```
Value Name                 Value Type          Value Data
Type                       REG_SZ              NTP
```

若要查看 NtpClient 時間提供者使用哪個時間伺服器，請在提升權限的命令提示字元中輸入：

```
w32tm /dumpreg /subkey:Parameters | findstr /i "ntpserver"
```

如果 VM 使用預設值，則輸出看起來如下：

```
NtpServer                  REG_SZ              time.windows.com,0x8
```


若要查看目前使用的時間提供者。

```
w32tm /query /source
```


以下是您可能看到的輸出，以及其代表的意義：
    
- **time.windows.com** - 在預設設定中，w32time 會向 time.windows.com 取得時間。 時間同步處理品質取決網際網路連線能力，且會受到封包延遲影響。 這是預設設定的一般輸出。
- **VM IC Time Synchronization Provider** - VM 從主機同步時間。 如果您加入僅限主機時間同步，或者 NtpServer 當下無法使用，通常會看到此結果。 
- *您的網域伺服器* - 目前的機器位於某個網域中，且該網域定義了時間同步階層。
- *其他伺服器* - w32time 已明確設為向這部其他伺服器取得時間。 時間同步品質取決於此時間伺服器的品質。
- **Local CMOS Clock** - 時鐘未同步處理。 如果重新開機後 w32time 尚未有足夠的時間啟動，或當所有設定的時間來源均無法使用，您可能會得到此輸出。


## <a name="opt-in-for-host-only-time-sync"></a>加入僅限主機時間同步

Azure 持續致力於改善主機上的時間同步功能，且可保證所有時間同步基礎結構均共置在 Microsoft 所屬的資料中心內。 如果偏好使用 time.windows.com 做為主要時間來源的預設設定發生時間同步問題，您可以使用下列命令來加入僅限主機時間同步。

將 VMIC 提供者標示為啟用。 

```
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\w32time\TimeProviders\VMICTimeProvider /v Enabled /t REG_DWORD /d 1 /f
```

將 NTPClient 提供者標示為停用。

```
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\w32time\TimeProviders\NtpClient /v Enabled /t REG_DWORD /d 0 /f
```

重新啟動 w32time 服務。

```
net stop w32time && net start w32time
```


## <a name="windows-server-2012-and-r2-vms"></a>Windows Server 2012 與 R2 VM 

Windows Server 2012 與 Windows Server 2012 R2 的時間同步預設設定有所不同。依照預設，w32time 設定為偏好服務的低負荷而非精確的時間。 

如果您想讓 Windows Server 2012 和 2012 R2 部署改為使用偏好精確時間的較新預設值，您可以套用下列設定。

更新 w32time 輪詢並更新間隔以配合 Windows Server 2016 設定。

```
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\w32time\Config /v MinPollInterval /t REG_DWORD /d 6 /f
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\w32time\Config /v MaxPollInterval /t REG_DWORD /d 10 /f
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\w32time\Config /v UpdateInterval /t REG_DWORD /d 100 /f
w32tm /config /update
```

為了讓 w32time 使用新的輪詢間隔，系統會將 NtpServers 標示為使用新輪詢間隔。 如果伺服器標註使用 0x1 位元旗標遮罩，便會覆寫此機制，且 w32time 會改用 SpecialPollInterval。 請確定指定的 NTP 伺服器使用 0x8 旗標或未使用任何旗標：

檢查哪些旗標用於所使用的 NTP 伺服器。

```
w32tm /dumpreg /subkey:Parameters | findstr /i "ntpserver"
```

## <a name="next-steps"></a>後續步驟

以下是時間同步的更多詳細資訊：

- [Windows 時間服務工具和設定](https://docs.microsoft.com/windows-server/networking/windows-time-service/Windows-Time-Service-Tools-and-Settings)
- [Windows Server 2016 改良功能](https://docs.microsoft.com/windows-server/networking/windows-time-service/windows-server-2016-improvements)
- [Windows Server 2016 的精確時間](https://docs.microsoft.com/windows-server/networking/windows-time-service/accurate-time)
- [設定 Windows 時間服務以達到高精確度環境所需的支援界限](https://docs.microsoft.com/windows-server/networking/windows-time-service/support-boundary)


