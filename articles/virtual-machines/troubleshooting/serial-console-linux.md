---
title: Azure 虛擬機器序列主控台 | Microsoft Docs
description: Azure 虛擬機器的雙向序列主控台。
services: virtual-machines-linux
documentationcenter: ''
author: harijay
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 09/11/2018
ms.author: harijay
ms.openlocfilehash: 642bf03ecef7f6db25c51671635d96ef7baed91a
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47411183"
---
# <a name="virtual-machine-serial-console"></a>虛擬機器序列主控台


Azure 上的「虛擬機器序列主控台」可讓您存取 Linux 虛擬機器的文字型主控台。 這個序列連線會連至虛擬機器的 COM1 序列埠，讓您存取未依存於虛擬機器的網路或作業系統狀態的虛擬機器。 目前只能透過 Azure 入口網站來存取虛擬機器的序列主控台，並且只允許對虛擬機器具有「VM 參與者」或更高存取權的使用者執行此操作。 

如需 Windows VM 的序列主控台文件，請[按一下這裡](../windows/serial-console.md)。

> [!Note] 
> 適用於虛擬機器的序列主控台已在全球 Azure 區域公開推出。 序列主控台目前尚未在 Azure Government 或「Azure 中國」雲端中提供。


## <a name="prerequisites"></a>必要條件 

* 您必須使用資源管理部署模型。 不支援傳統部署。 
* 虛擬機器必須啟用[開機診斷](boot-diagnostics.md)功能 - 請參閱以下螢幕擷取畫面。

    ![](./media/virtual-machines-serial-console/virtual-machine-serial-console-diagnostics-settings.png)

* 使用序列主控台的 Azure 帳戶必須具有 VM 的[參與者角色](../../role-based-access-control/built-in-roles.md)和[開機診斷](boot-diagnostics.md)儲存體帳戶。 
* 您為其存取序列主控台的虛擬機器，也必須具有密碼型帳戶。 您可以使用 VM 存取擴充的[重設密碼](https://docs.microsoft.com/azure/virtual-machines/extensions/vmaccess#reset-password)功能來建立一個帳戶 - 請參閱以下螢幕擷取畫面。

    ![](./media/virtual-machines-serial-console/virtual-machine-serial-console-reset-password.png)

* 如需了解 Linux 發行版本特定的設定，請參閱[存取 Linux 的序列主控台](#Serial-Console-Linux-distro-availability)



## <a name="get-started-with-serial-console"></a>開始使用序列主控台
只有透過 [Azure 入口網站](https://portal.azure.com)，才能存取虛擬機器的序列主控台。 以下是透過入口網站存取虛擬機器之序列主控台的步驟 

  1. 開啟 Azure 入口網站
  1. (如果您的 VM 有使用密碼驗證的使用者，請跳過此步驟) 按一下 [重設密碼] 刀鋒視窗，以新增具有使用者名稱/密碼驗證的使用者
  1. 在左功能表中，選取 [虛擬機器]。
  1. 按一下清單中的 VM。 將會開啟該 VM 的概觀頁面。
  1. 向下捲動至 [支援與疑難排解] 區段，然後按一下 [序列主控台] 選項。 將會開啟含有序列主控台的新窗格，並開始連線。

![](./media/virtual-machines-serial-console/virtual-machine-linux-serial-console-connect.gif)

### 

> [!NOTE] 
> 序列主控台需要一個已設定密碼的本機使用者。 此時，僅設定 SSH 公開金鑰的 VM 將無法登入至序列主控台。 若要建立具有密碼的本機使用者，請使用 [VM 存取擴充功能](https://docs.microsoft.com/azure/virtual-machines/linux/using-vmaccess-extension) \(可透過在入口網站中按一下 [重設密碼] 來使用此功能\)，然後建立具有密碼的本機使用者。
> 您也可以[使用 GRUB 來進入單一使用者模式](./serial-console-grub-single-user-mode.md)，以重設帳戶中的系統管理員密碼。

## <a name="serial-console-linux-distro-availability"></a>序列主控台 Linux 散發版本可用性
為了讓序列主控台正確運作，必須設定讓客體作業系統將主控台訊息讀取並寫入至序列連接埠。 大多數[已背書的 Azure Linux 發行版本](https://docs.microsoft.com/azure/virtual-machines/linux/endorsed-distros)都已預設設定序列主控台。 只要在 Azure 入口網站中按一下 [序列主控台] 區段，即可存取該主控台。 

發行版本      | 序列主控台存取
:-----------|:---------------------
Red Hat Enterprise Linux    | Azure 上提供的 Red Hat Enterprise Linux 映像已預設啟用主控台存取。 
CentOS      | Azure 上提供的 CentOS 映像已預設啟用主控台存取。 
Ubuntu      | Azure 上提供的 Ubuntu 映像已預設啟用主控台存取。
CoreOS      | Azure 上提供的 CoreOS 映像已預設啟用主控台存取。
SUSE        | Azure 上提供的較新 SLES 映像已預設啟用主控台存取。 如果您使用 Azure 上較舊的 SLES 版本 (10 或更舊)，請依照這篇 [KB 文章](https://www.novell.com/support/kb/doc.php?id=3456486) \(英文\) 來啟用序列主控台。 
Oracle Linux        | Azure 上提供的 Oracle Linux 映像已預設啟用主控台存取。
自訂 Linux 映像     | 若要啟用自訂 Linux VM 映像的序列主控台，請在 `/etc/inittab` 中啟用主控台存取以在 `ttyS0` 上執行終端機。 以下是在 inittab 檔案中新增此設定的範例：`S0:12345:respawn:/sbin/agetty -L 115200 console vt102`。 如需有關如何正確建立自訂映像的詳細資訊，請參閱[在 Azure 中建立及上傳 Linux VHD](https://aka.ms/createuploadvhd)。 如果您正在建置自訂核心，則可以考慮啟用 `CONFIG_SERIAL_8250=y` 與 `CONFIG_MAGIC_SYSRQ_SERIAL=y` 核心旗標。 設定檔通常位於 /boot/ 底下以供進一步探索。

## <a name="common-scenarios-for-accessing-serial-console"></a>存取序列主控台的常見案例 
案例          | 序列主控台中的動作                
:------------------|:-----------------------------------------
損毀的 FSTAB 檔案 | 按 `Enter` 鍵以繼續操作，然後使用文字編輯器來修正 FSTAB 檔案。 您可能必須在單一使用者模式下，才能進行此操作。 請參閱[如何修正 FSTAB 問題](https://support.microsoft.com/help/3206699/azure-linux-vm-cannot-start-because-of-fstab-errors) \(機器翻譯\) 和[使用序列主控台來存取 GRUB 和單一使用者模式](serial-console-grub-single-user-mode.md)來開始著手。
不正確的防火牆規則 | 存取序列主控台，然後修正 iptable。 
檔案系統損毀/檢查 | 存取序列主控台，然後復原檔案系統。 
SSH/RDP 設定問題 | 存取序列主控台，然後變更設定。 
網路鎖定系統| 透過入口網站存取序列主控台以管理系統。 
與開機載入器互動 | 透過序列主控台存取 GRUB。 請移至[使用序列主控台來存取 GRUB 和單一使用者模式](serial-console-grub-single-user-mode.md)來開始著手。 

## <a name="disable-serial-console"></a>停用序列主控台
根據預設，所有訂用帳戶都已啟用所有 VM 的序列主控台存取權。 您可以在訂用帳戶層級或 VM 層級停用序列主控台。

> [!Note] 
> 若要啟用或停用訂用帳戶的序列主控台，您必須有該訂用帳戶的寫入權限。 這包包括 (但不限於) 系統管理員或擁有者角色。 自訂角色也可能會有寫入權限。

### <a name="subscription-level-disable"></a>訂用帳戶層級的停用
透過[停用主控台 REST API 呼叫](https://aka.ms/disableserialconsoleapi)，即可針對整個訂用帳戶停用序列主控台。 您可以使用 [API 文件] 頁面上提供的 [試用] 功能，針對訂用帳戶停用和啟用「序列主控台」。 請輸入您的 `subscriptionId`、在 `default` 欄位中輸入"default"，然後按一下 [執行]。 目前尚未提供 Azure CLI 命令，稍後將會提供。 請[試用這裡的 REST API 呼叫](https://aka.ms/disableserialconsoleapi)。

![](./media/virtual-machines-serial-console/virtual-machine-serial-console-rest-api-try-it.png)

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
雖然系統不會記錄主控台的存取密碼，但如果在主控台內執行的命令包含或會輸出密碼、祕密、使用者名稱或任何其他形式的「個人識別資訊」(PII)，系統就會在實作序列主控台之回捲功能的過程中，將這些資訊與所有其他可見文字一起寫入至虛擬機器開機診斷記錄。 這些記錄是循環性的，並且只有具備診斷儲存體帳戶之讀取權限的個人才能存取這些記錄，不過，建議您針對任何可能涉及祕密和/或 PII 的情況，都依循使用 SSH 主控台的最佳做法。 

### <a name="concurrent-usage"></a>並行使用方式
如果在某位使用者已連線至序列主控台的情況下，另一位使用者成功取得該相同虛擬機器的存取權，系統就會將第一位使用者中斷連線，而讓第二位使用者連線，其方式類似於讓第一位使用者起立並離開實體主控台，而讓新使用者坐下進行操作。

>[!CAUTION] 
這意謂著系統不會將中斷連線的使用者登出！ 強制在中斷連線時登出的功能 (透過 SIGHUP 或類似的機制) 仍在規劃中。 就 Windows 而言，在 SAC 中會啟用自動逾時，不過，針對 Linux，您則可以設定終端機逾時設定。 若要這麼做，只需在您用來登入主控台之使用者的 .bash_profile 或 .profile 中新增 `export TMOUT=600`即可讓工作階段在 10 分鐘後逾時。

## <a name="accessibility"></a>協助工具
協助工具是 Azure 序列主控台的按鍵焦點。 為此，我們已確保視障和聽障人士以及無法使用滑鼠的使用者都能存取序列主控台。

### <a name="keyboard-navigation"></a>鍵盤導覽
使用鍵盤上的 `tab` 鍵，在 Azure 入口網站中的序列主控台介面上導覽。 您的位置會在螢幕上醒目顯示。 若要離開序列主控台刀鋒視窗的焦點，請在鍵盤上按 `Ctrl + F6`。

### <a name="use-serial-console-with-a-screen-reader"></a>透過螢幕助讀程式使用序列主控台
序列主控台附有內建的螢幕助讀程式支援。 在開啟螢幕助讀程式的情況下瀏覽，可讓螢幕助讀程式大聲讀出目前所選按鈕的替代文字。

## <a name="errors"></a>Errors
大多數錯誤本質上都是暫時性的，通常重試序列主控台連線便可解決這些錯誤。 下表顯示錯誤及緩和措施清單

Error                            |   緩和 
:---------------------------------|:--------------------------------------------|
無法擷取 '<VMNAME>' 的開機診斷設定。 若要使用序列主控台，請確定已針對此 VM 啟用診斷開機診斷。 | 請確定 VM 已啟用[開機診斷](boot-diagnostics.md)。 
VM 處於已停止 (已解除配置) 狀態。 啟動 VM 並重試序列主控台連線。 | 虛擬機器必須處於已啟動狀態，才能存取序列主控台
您沒有可使用此 VM 序列主控台的必要權限。 請確定您至少具有「VM 參與者」角色權限。| 若要存取序列主控台，必須具有特定權限才能存取。 如需詳細資料，請參閱[存取需求](#prerequisites)
無法判斷開機診斷儲存體帳戶 '<STORAGEACCOUNTNAME>' 的資源群組。 請確認已針對此 VM 啟用開機診斷，且您具有此儲存體帳戶的存取權。 | 若要存取序列主控台，必須具有特定權限才能存取。 如需詳細資料，請參閱[存取需求](#prerequisites)
Web 通訊端已關閉或無法開啟。 | 您可能需要將 `*.console.azure.com` 列入白名單。 有一個更詳細但較冗長的方法，就是將 [Microsoft Azure 資料中心 IP 範圍](https://www.microsoft.com/en-us/download/details.aspx?id=41653)列入白名單，而此範圍會經常變更。
存取此 VM 的開機診斷儲存體帳戶時，遇到「禁止」回應。 | 請確定開機診斷沒有帳戶防火牆。 必須要有可存取的開機診斷儲存體帳戶，序列主控台才能運作。

## <a name="known-issues"></a>已知問題 
我們已了解序列主控台的一些問題。 以下是這些問題的清單，以及減輕其影響的步驟。

問題                           |   緩和 
:---------------------------------|:--------------------------------------------|
在出現連線橫幅後按 Enter 鍵並未顯示登入提示 | 請參閱此頁面：[按 Enter 鍵沒有任何作用](https://github.com/Microsoft/azserialconsole/blob/master/Known_Issues/Hitting_enter_does_nothing.md)。 如果您執行的是自訂 VM、強化設備，或是導致 Linux 無法正確地連線至序列埠的 GRUB 組態，則可能發生此情形。
序列主控台文字只會佔用部分的螢幕大小 (通常在使用文字編輯器之後) | 序列主控台不支援對視窗大小進行交涉 ([RFC 1073](https://www.ietf.org/rfc/rfc1073.txt))，這表示系統將不會傳送 SIGWINCH 訊號以更新螢幕大小，而且 VM 將無法得知您終端機的大小。 建議您安裝 xterm 或其他提供「調整大小」命令的類似公用程式。 執行「調整大小」即可修正此問題。
無法貼上很長的字串 | 序列主控台會將貼上至終端機的字串長度限制為 2048 個字元。 這是為了防止序列連接埠頻寬負荷過度。


## <a name="frequently-asked-questions"></a>常見問題集 
**問：如何傳送意見反應？**

答： 前往 https://aka.ms/serialconsolefeedback \(英文\) 來以問題形式提供意見反應。 或者，您也可以透過 azserialhelp@microsoft.com 或在 http://feedback.azure.com \(英文\) 的虛擬機器類別中傳送意見反應 (較不建議)

**問：序列主控台是否支援複製/貼上？**

答： 是，它支援。 請使用 Ctrl + Shift + C 與 Ctrl + Shift + V 來針對終端機進行複製及貼上。

**問：我是否可以使用序列主控台，而不使用 SSH 連線？**

答： 雖然這在技術上應該是可行的，但序列主控台主要是要作為無法透過 SSH 進行連線時的疑難排解工具。 基於以下兩個原因，建議您不要以序列主控台取代 SSH：

1. 序列主控台是純文字的連線，並沒有和 SSH 一樣多的頻寬，因此很難在序列主控台中進行重度使用 GUI 的互動。
1. 目前，只要有使用者名稱和密碼即可存取序列主控台。 SSH 金鑰的安全性遠高於使用者名稱/密碼的組合，因此就登入安全性的觀點而言，我們建議 SSH 應比序列主控台優先使用。

**問：誰可以針對我的訂用帳戶啟用或停用序列主控台？**

答： 若要以訂用帳戶層級的規模啟用或停用序列主控台，您必須有該訂用帳戶的寫入權限。 具有寫入權限的角色包括 (但不限於) 系統管理員或擁有者角色。 自訂角色也可能會有寫入權限。

**問：誰可以存取我 VM 的序列主控台？**

答： 您必須具有參與者或更高層級的存取權，才能存取 VM 的序列主控台。 

**問：我的序列主控台沒有顯示任何內容，該怎麼辦？**

答： 您映像的序列主控台存取設定很有可能是錯誤的。 請參閱[存取 Linux 的序列主控台](#Access-Serial-Console-for-Linux)以取得設定映像來啟用序列主控台的詳細資料。

**問：序列主控台是否適用於虛擬機器擴展集？**

答： 目前尚不支援存取虛擬機器擴展集執行個體的序列主控台。

**問：我只使用 SSH 金鑰驗證來設定 VM，我是否仍然可以使用序列主控台來連線到我的 VM？** 答：是。 序列主控台並不需要 SSH 金鑰，因此您只需要設定使用者名稱/密碼組合就可以。 若要這樣做，您可以使用入口網站中的 [重設密碼] 刀鋒視窗，並使用那些認證來登入序列主控台。

## <a name="next-steps"></a>後續步驟
* 使用「序列主控台」[在開機時進入GRUB 並進入單一使用者模式](serial-console-grub-single-user-mode.md)
* 使用「序列主控台」來進行[NMI 和 SysRq 呼叫](serial-console-nmi-sysrq.md)
* 「序列主控台」也適用於 [Windows](../windows/serial-console.md) VM
* 深入了解[開機診斷](boot-diagnostics.md)