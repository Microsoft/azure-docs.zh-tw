---
title: Azure 虛擬機器序列主控台 | Microsoft Docs
description: Azure 虛擬機器的雙向序列主控台。
services: virtual-machines-windows
documentationcenter: ''
author: harijay
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 08/07/2018
ms.author: harijay
ms.openlocfilehash: e1884048d0f02de1b3a354bc4dac2b3e98dcccc9
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47411872"
---
# <a name="virtual-machine-serial-console"></a>虛擬機器序列主控台


Azure 上的「虛擬機器序列主控台」可讓您存取 Windows 虛擬機器的文字型主控台。 這個序列連線會連至虛擬機器的 COM1 序列埠，讓您存取未依存於虛擬機器的網路或作業系統狀態的虛擬機器。 目前只能透過 Azure 入口網站來存取虛擬機器的序列主控台，並且只允許對虛擬機器具有「VM 參與者」或更高存取權的使用者執行此操作。 

如需 Linux VM 的序列主控台文件，請[按一下這裡](serial-console-linux.md)。

> [!Note] 
> 適用於虛擬機器的序列主控台已在全球 Azure 區域公開推出。 序列主控台目前尚未在 Azure Government 或「Azure 中國」雲端中提供。

 

## <a name="prerequisites"></a>必要條件 

* 您必須使用資源管理部署模型。 不支援傳統部署。 
* 虛擬機器必須啟用[開機診斷](boot-diagnostics.md)功能 

    ![](../media/virtual-machines-serial-console/virtual-machine-serial-console-diagnostics-settings.png)

* 使用序列主控台的帳戶必須具有 VM 的[參與者角色](../../role-based-access-control/built-in-roles.md)和[開機診斷](boot-diagnostics.md)儲存體帳戶。 
* 您為其存取序列主控台的虛擬機器，也必須具有密碼型帳戶。 您可以使用 VM 存取擴充的[重設密碼](https://docs.microsoft.com/azure/virtual-machines/extensions/vmaccess#reset-password)功能來建立一個帳戶 - 請參閱以下螢幕擷取畫面。

    ![](../media/virtual-machines-serial-console/virtual-machine-serial-console-reset-password.png)

## <a name="get-started-with-serial-console"></a>開始使用序列主控台
只有透過 [Azure 入口網站](https://portal.azure.com)，才能存取虛擬機器的序列主控台。 以下是透過入口網站存取虛擬機器的序列主控台時須執行的步驟。

  1. 開啟 Azure 入口網站
  2. 在左功能表中，選取 [虛擬機器]。
  3. 按一下清單中的 VM。 將會開啟該 VM 的概觀頁面。
  4. 向下捲動至 [支援與疑難排解] 區段，然後按一下 [序列主控台] 選項。 將會開啟含有序列主控台的新窗格，並開始連線。


## <a name="enable-serial-console-in-custom-or-older-images"></a>在自訂或舊版的映像中啟用序列主控台
Azure 上的新版 Windows Server 映像將會預設啟用[特殊系統管理主控台](https://technet.microsoft.com/library/cc787940(v=ws.10).aspx) (SAC)。 在伺服器版本的 Windows 上可支援 SAC，但在用戶端版本 (例如 Windows 10、Windows 8 或 Windows 7) 上則不支援。 若要為在 2018 年 2 月之前建立的 Windows 虛擬機器啟用序列主控台，請使用下列步驟： 

1. 透過「遠端桌面」連線至您的 Windows 虛擬機器
2. 從系統管理命令提示字元中執行下列命令 
* `bcdedit /ems {current} on`
* `bcdedit /emssettings EMSPORT:1 EMSBAUDRATE:115200`
3. 重新啟動系統以啟用 SAC 主控台

![](/media/virtual-machines-serial-console/virtual-machine-windows-serial-console-connect.gif)

如有需要，也可以離線啟用 SAC：

1. 將您想要為其設定 SAC 的 Windows 磁碟以資料磁碟的形式連結至現有的 VM。 
2. 從系統管理命令提示字元中執行下列命令 
* `bcdedit /store <mountedvolume>\boot\bcd /ems {default} on`
* `bcdedit /store <mountedvolume>\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200`

### <a name="how-do-i-know-if-sac-is-enabled"></a>如何得知 SAC 是否已啟用？

如果未啟用 [SAC] (https://technet.microsoft.com/library/cc787940(v=ws.10).aspx)，序列主控台將不會顯示 SAC 提示。 在某些情況下會顯示 VM 健康情況資訊，在其他情況下則會空白。 如果您使用在 2018 年 2 月之前建立的 Windows Server 映像，SAC 可能不會啟用。

## <a name="enable-the-windows-boot-menu-in-serial-console"></a>在序列主控台中啟用 Windows 開機功能表 

如果您需要讓 Windows 開機載入器提示顯示在序列主控台中，您可以將下列額外選項新增至開機組態資料。 如需詳細資訊，請參閱 [bcdedit](https://docs.microsoft.com/windows-hardware/drivers/devtest/bcdedit--set)

1. 透過「遠端桌面」連線至您的 Windows 虛擬機器
2. 從系統管理命令提示字元中執行下列命令 
* `bcdedit /set {bootmgr} displaybootmenu yes`
* `bcdedit /set {bootmgr} timeout 5`
* `bcdedit /set {bootmgr} bootems yes`
3. 重新啟動系統以啟用開機功能表

## <a name="use-serial-console-for-nmi-calls-in-windows-vms"></a>使用序列主控台在 Windows VM 中進行 NMI 呼叫
非遮罩式插斷 (NMI) 旨在建立虛擬機器上軟體不會忽略的訊號。 在過去，NMI 已用來監視系統上需要特定回應時間的硬體問題。  現在，程式設計人員和系統管理員通常會使用 NMI 作為對無回應系統進行偵錯和疑難排解的機制。

使用命令列中的鍵盤圖示，即可使用序列主控台將 NMI 傳送至 Azure 虛擬機器，如下所示。 NMI 序列傳遞出去後，虛擬機器組態將會控制系統的回應方式。 您可以將 Windows 設定成在收到 NMI 時當機並建立記憶體傾印。

![](../media/virtual-machines-serial-console/virtual-machine-windows-serial-console-nmi.png) <br>

如需有關將 Windows 設定成在收到 NMI 時建立損毀傾印的資訊，請參閱：[如何在 Windows 型系統上使用 NMI 來產生完整的損毀傾印檔案或核心損毀傾印檔案](https://support.microsoft.com/en-us/help/927069/how-to-generate-a-complete-crash-dump-file-or-a-kernel-crash-dump-file) \(機器翻譯\)

## <a name="disable-serial-console"></a>停用序列主控台
根據預設，所有訂用帳戶都已啟用所有 VM 的序列主控台存取權。 您可以在訂用帳戶層級或 VM 層級停用序列主控台。

> [!Note]       
> 若要啟用或停用訂用帳戶的序列主控台，您必須有該訂用帳戶的寫入權限。 這包包括 (但不限於) 系統管理員或擁有者角色。 自訂角色也可能會有寫入權限。

### <a name="subscription-level-disable"></a>訂用帳戶層級的停用
透過[停用主控台 REST API 呼叫](https://aka.ms/disableserialconsoleapi)，即可針對整個訂用帳戶停用「序列主控台」。 您可以使用 [API 文件] 頁面上提供的 [試用] 功能，針對訂用帳戶停用和啟用「序列主控台」。 請輸入您的 `subscriptionId`、在 `default` 欄位中輸入"default"，然後按一下 [執行]。 目前尚未提供 Azure CLI 命令，稍後將會提供。 請[試用這裡的 REST API 呼叫](https://aka.ms/disableserialconsoleapi)。

![](../media/virtual-machines-serial-console/virtual-machine-serial-console-rest-api-try-it.png)

或者，您也可以在 Cloud Shell 中使用下面的一組命令 (所顯示的 Bash 命令)，針對訂用帳戶停用、啟用序列主控台及檢視該主控台的停用狀態。 

* 針對訂用帳戶取得序列主控台的停用狀態：
    ```azurecli-interactive
    $ export ACCESSTOKEN=($(az account get-access-token --output=json | jq .accessToken | tr -d '"')) 

    $ export SUBSCRIPTION_ID=$(az account show --output=json | jq .id -r)

    $ curl "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/providers/Microsoft.SerialConsole/consoleServices/default?api-version=2018-05-01" -H "Authorization: Bearer $ACCESSTOKEN" -H "Content-Type: application/json" -H "Accept: application/json" -s | jq .properties
    ```
* 針對訂用帳戶停用序列主控台：
    ```azurecli-interactive 
    $ export ACCESSTOKEN=($(az account get-access-token --output=json | jq .accessToken | tr -d '"')) 

    $ export SUBSCRIPTION_ID=$(az account show --output=json | jq .id -r)

    $ curl -X POST "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/providers/Microsoft.SerialConsole/consoleServices/default/disableConsole?api-version=2018-05-01" -H "Authorization: Bearer $ACCESSTOKEN" -H "Content-Type: application/json" -H "Accept: application/json" -s -H "Content-Length: 0"
    ```
* 針對訂用帳戶啟用序列主控台：
    ```azurecli-interactive
    $ export ACCESSTOKEN=($(az account get-access-token --output=json | jq .accessToken | tr -d '"')) 

    $ export SUBSCRIPTION_ID=$(az account show --output=json | jq .id -r)

    $ curl -X POST "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/providers/Microsoft.SerialConsole/consoleServices/default/enableConsole?api-version=2018-05-01" -H "Authorization: Bearer $ACCESSTOKEN" -H "Content-Type: application/json" -H "Accept: application/json" -s -H "Content-Length: 0"
    ```

### <a name="vm-level-disable"></a>VM 層級的停用
您可以藉由停用特定 VM 的開機診斷設定，停用該 VM 的序列主控台。 只要從 Azure 入口網站關閉開機診斷，系統就會停用該 VM 的序列主控台。

## <a name="serial-console-security"></a>序列主控台安全性 

### <a name="access-security"></a>存取安全性 
僅限具有虛擬機器之 [VM 參與者](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor)或更高存取權的使用者，才能存取序列主控台。 如果您的 AAD 租用戶要求使用 Multi-Factor Authentication，則存取序列主控台時也會需要 MFA，因為會透過 [Azure 入口網站](https://portal.azure.com)進行其存取。

### <a name="channel-security"></a>通道安全性
在網路上來回傳送的所有資料都會經過加密。

### <a name="audit-logs"></a>稽核記錄
目前對序列主控台進行的所有存取都會記錄在虛擬機器的[開機診斷](https://docs.microsoft.com/azure/virtual-machines/linux/boot-diagnostics)記錄中。 這些記錄的存取權均由 Azure 虛擬機器系統管理員所擁有和控制。  

>[!CAUTION] 
雖然系統不會記錄主控台的存取密碼，但如果在主控台內執行的命令包含或會輸出密碼、祕密、使用者名稱或任何其他形式的「個人識別資訊」(PII)，系統就會在實作序列主控台之回捲功能的過程中，將這些資訊與所有其他可見文字一起寫入至虛擬機器開機診斷記錄。 這些記錄是循環性的，並且只有具備診斷儲存體帳戶之讀取權限的個人才能存取這些記錄，不過，建議您針對任何可能涉及祕密和/或 PII 的情況，都依循使用「遠端桌面」的最佳做法。 

### <a name="concurrent-usage"></a>並行使用方式
如果在某位使用者已連線至序列主控台的情況下，另一位使用者成功取得該相同虛擬機器的存取權，系統就會將第一位使用者中斷連線，而讓第二位使用者連線，其方式類似於讓第一位使用者起立並離開實體主控台，而讓新使用者坐下進行操作。

>[!CAUTION] 
這意謂著系統不會將中斷連線的使用者登出！ 強制在中斷連線時登出的功能 (透過 SIGHUP 或類似的機制) 仍在規劃中。 就 Windows 而言，在 SAC 中會啟用自動逾時，不過，針對 Linux，您則可以設定終端機逾時設定。 

## <a name="common-scenarios-for-accessing-serial-console"></a>存取序列主控台的常見案例 
案例          | 序列主控台中的動作                
:------------------|:-----------------------------------------
不正確的防火牆規則 | 存取序列主控台，然後修正 Windows 防火牆規則。 
檔案系統損毀/檢查 | 存取序列主控台，然後復原檔案系統。 
RDP 設定問題 | 存取序列主控台，然後變更設定。 請移至 [RDP 文件](https://docs.microsoft.com/windows-server/remote/remote-desktop-services/clients/remote-desktop-allow-access)以開始作業。
網路鎖定系統| 透過入口網站存取序列主控台以管理系統。 [序列主控台 CMD 和 PowerShell 文件](serial-console-cmd-ps-commands.md)中列出了部分網路命令。 
與開機載入器互動 | 透過序列主控台存取 BCD。 請移至[讓開機功能表顯示在序列主控台中](#enabling-boot-menu-to-show-in-the-serial-console)以開始作業。 

## <a name="accessibility"></a>協助工具
協助工具是 Azure 序列主控台的按鍵焦點。 為此，我們已確保視障和聽障人士以及無法使用滑鼠的使用者都能存取序列主控台。

### <a name="keyboard-navigation"></a>鍵盤導覽
使用鍵盤上的 `tab` 鍵，在 Azure 入口網站中的序列主控台介面上導覽。 您的位置會在螢幕上醒目顯示。 若要離開序列主控台刀鋒視窗的焦點，請在鍵盤上按 `Ctrl + F6`。

### <a name="use-serial-console-with-a-screen-reader"></a>透過螢幕助讀程式使用序列主控台
序列主控台附有內建的螢幕助讀程式支援。 在開啟螢幕助讀程式的情況下瀏覽，可讓螢幕助讀程式大聲讀出目前所選按鈕的替代文字。

## <a name="errors"></a>Errors
大多數錯誤本質上都是暫時性的，重試連線將可解決這些錯誤。 下表顯示錯誤及緩和措施清單

Error                            |   緩和 
:---------------------------------|:--------------------------------------------|
無法擷取 '<VMNAME>' 的開機診斷設定。 若要使用序列主控台，請確定已針對此 VM 啟用診斷開機診斷。 | 請確定 VM 已啟用[開機診斷](boot-diagnostics.md)。 
VM 處於已停止 (已解除配置) 狀態。 啟動 VM 並重試序列主控台連線。 | 虛擬機器必須處於已啟動狀態，才能存取序列主控台
您沒有可使用此 VM 序列主控台的必要權限。 請確定您至少具有「VM 參與者」角色權限。| 若要存取序列主控台，必須具有特定權限才能存取。 如需詳細資料，請參閱[存取需求](#prerequisites)
無法判斷開機診斷儲存體帳戶 '<STORAGEACCOUNTNAME>' 的資源群組。 請確認已針對此 VM 啟用開機診斷，且您具有此儲存體帳戶的存取權。 | 若要存取序列主控台，必須具有特定權限才能存取。 如需詳細資料，請參閱[存取需求](#prerequisites)
存取此 VM 的開機診斷儲存體帳戶時，遇到「禁止」回應。 | 請確定開機診斷沒有帳戶防火牆。 必須要有可存取的開機診斷儲存體帳戶，序列主控台才能運作。
Web 通訊端已關閉或無法開啟。 | 您可能需要將 `*.console.azure.com` 列入白名單。 有一個更詳細但較冗長的方法，就是將 [Microsoft Azure 資料中心 IP 範圍](https://www.microsoft.com/en-us/download/details.aspx?id=41653)列入白名單，而此範圍會經常變更。
連線至 Windows VM 時只有顯示健康情況資訊| 如果您的 Windows 映像未啟用特殊系統管理主控台，則會顯示此資訊。 請參閱[存取 Windows 的序列主控台](#access-serial-console-for-windows)，以取得在 Windows VM 上手動啟用 SAC 的指示。 您可以在 [Windows 健康情況訊號](https://github.com/Microsoft/azserialconsole/blob/master/Known_Issues/Windows_Health_Info.md)找到詳細資訊。

## <a name="known-issues"></a>已知問題 
我們已了解序列主控台的一些問題。 以下是這些問題的清單，以及減輕其影響的步驟。

問題                             |   緩和 
:---------------------------------|:--------------------------------------------|
在出現連線橫幅後按 Enter 鍵並未顯示登入提示 | 請參閱此頁面：[按 Enter 鍵沒有任何作用](https://github.com/Microsoft/azserialconsole/blob/master/Known_Issues/Hitting_enter_does_nothing.md)。 如果您執行自訂 VM、強化設備，或是導致 Windows 無法正確地連線至序列埠的 GRUB 組態，則可能發生此情形。
在核心偵錯啟用時，無法在出現 SAC 提示時輸入 | 透過 RDP 連線至 VM，並從更高權限的命令提示字元執行 `bcdedit /debug {current} off`。 如果您無法執行 RDP 連線，可以改為將作業系統磁碟連結至另一個 Azure VM，並在連結作為資料磁碟時使用 `bcdedit /store <drive letter of data disk>:\boot\bcd /debug <identifier> off` 加以修改，然後再將磁碟交換回來。
在 SAC 中貼到 PowerShell 中時，如果原始內容中有重複字元，就會產生第三個字元 | 因應措施是從目前的工作階段中回溯卸載 PSReadLine 模組。 執行 `Remove-Module PSReadLine` 從目前的工作階段卸載 PSReadLine 模組 - 這並不會刪除或解除安裝該模組。
某些鍵盤輸入會產生奇怪的 SAC 輸出 (例如 `[A``[3~`) | SAC 提示字元不支援[VT100](https://aka.ms/vtsequences) 逸出序列。
無法貼上很長的字串 | 序列主控台會將貼上至終端機的字串長度限制為 2048 個字元。 這是為了防止序列連接埠頻寬負荷過度。

## <a name="frequently-asked-questions"></a>常見問題集 

**問：如何傳送意見反應？**

A. 前往 https://aka.ms/serialconsolefeedback \(英文\) 來以問題形式提供意見反應。 或者，您也可以透過 azserialhelp@microsoft.com 或在 http://feedback.azure.com 的虛擬機器類別中傳送意見反應 (較不建議)

**問：序列主控台是否支援複製/貼上？**

A. 是，它支援。 請使用 Ctrl + Shift + C 與 Ctrl + Shift + V 來針對終端機進行複製及貼上。

**問：誰可以針對我的訂用帳戶啟用或停用序列主控台？**

A. 若要以訂用帳戶層級的規模啟用或停用序列主控台，您必須有該訂用帳戶的寫入權限。 具有寫入權限的角色包括 (但不限於) 系統管理員或擁有者角色。 自訂角色也可能會有寫入權限。

**問：誰可以存取我 VM 的序列主控台？**

A. 您必須具有參與者或更高層級的存取權，才能存取 VM 的序列主控台。 

**問：我的序列主控台沒有顯示任何內容，該怎麼辦？**

A. 您映像的序列主控台存取設定很有可能是錯誤的。 請參閱[在自訂或舊版的映像中啟用序列主控台](#Enable-Serial-Console-in-custom-or-older-images)，以取得設定映像以啟用序列主控台的詳細資料。

**問：序列主控台是否適用於虛擬機器擴展集？**

A. 目前尚不支援存取虛擬機器擴展集執行個體的序列主控台。

## <a name="next-steps"></a>後續步驟
* 如需您可以在 Windows SAC 中使用之 CMD 及 PowerShell 命令的深入指南，請按一下[這裡](serial-console-cmd-ps-commands.md)。
* 「序列主控台」也適用於 [Linux](serial-console-linux.md) VM。
* 深入了解[開機診斷](boot-diagnostics.md)。