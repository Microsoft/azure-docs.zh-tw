---
title: 使用 Azure 自動化追蹤變更
description: 「變更追蹤」解決方案可協助您識別您環境中發生的軟體及 Windows 服務變更。
services: automation
ms.service: automation
ms.component: change-inventory-management
author: georgewallace
ms.author: gwallace
ms.date: 08/31/2018
ms.topic: conceptual
manager: carmonm
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 0707726ec86b0a0c69d1ec752ebd6761327f3f0f
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43669478"
---
# <a name="track-changes-in-your-environment-with-the-change-tracking-solution"></a>使用變更追蹤解決方案來追蹤環境中的變更

本文將協助您使用變更追蹤解決方案，輕鬆地找出您環境中的變更。 此解決方案會追蹤 Windows 與 Linux 軟體、Windows 和 Linux 檔案、Windows 登錄機碼、Windows 服務以及 Linux 精靈的變更。 識別組態變更可協助您找出操作問題。

受監視的伺服器上安裝的軟體、Windows 服務、Windows 登錄和檔案以及 Linux 精靈的變更，會傳送至雲端中的 Log Analytics 服務進行處理。 會將邏輯套用至接收的資料，且雲端服務會記錄資料。 使用 [變更追蹤] 儀表板上的資訊，您可以輕鬆地看到您的伺服器基礎結構中所做的變更。

## <a name="supported-windows-operating-systems"></a>支援的 Windows 作業系統

Windows 代理程式正式支援下列 Windows 作業系統版本：

* Windows Server 2008 R2 或更新版本

## <a name="supported-linux-operating-systems"></a>支援的 Linux 作業系統

以下為正式支援的 Linux 散發套件。 不過，Linux 代理程式也可能在未列出的其他散發套件上執行。 除非另有說明，列出的每個主要版本都支援所有次要版本。  

* Amazon Linux 2012.09 至 2015.09 (x86/x64)
* CentOS Linux 5、6 和 7 (x86/x64)  
* Oracle Linux 5、6 和 7 (x86/x64)
* Red Hat Enterprise Linux Server 5、6 和 7 (x86/x64)
* Debian GNU/Linux 6、7 和 8 (x86/x64)
* Ubuntu 12.04 LTS、14.04 LTS、16.04 LTS (x86/x64)
* SUSE Linux Enterprise Server 11 和 12 (x86/x64)

## <a name="enable-change-tracking-and-inventory"></a>啟用變更追蹤和清查

若要開始追蹤變更，您必須為自動化帳戶啟用變更追蹤和清查解決方案。

1. 在 Azure 入口網站中，瀏覽至您的自動化帳戶
2. 選取 [組態] 下的 [變更追蹤]。
3. 選取現有的 Log Analytics 工作區或 [建立新的工作區]，然後按一下 [啟用]。

這會為您的自動化帳戶啟用解決方案。 啟用解決方案可能需要 15 分鐘。 解決方案啟用時，會有藍色橫幅通知您。 瀏覽回 [變更追蹤] 頁面，以管理解決方案。

## <a name="configuring-change-tracking-and-inventory"></a>設定變更追蹤和清查

若要了解如何將電腦登入解決方案，請瀏覽：[登入自動化解決方案](automation-onboard-solutions-from-automation-account.md)。 一旦您有已安裝變更追蹤和清查解決方案的機器，即可設定要追蹤的項目。當您啟用新的檔案或登錄機碼加以追蹤時，其變更追蹤和清查都會啟用。

如需追蹤 Windows 和 Linux 上的檔案變更，則會使用檔案的 MD5 雜湊。 這些雜湊接著用來偵測自上次清查後是否已進行變更。

### <a name="configure-linux-files-to-track"></a>設定要追蹤的 Linux 檔案

使用下列步驟，設定要在 Linux 電腦上追蹤的檔案：

1. 從您的自動化帳戶中，選取 [組態管理] 下的 [變更追蹤]。 按一下 [編輯設定] \(齒輪符號)。
2. 在 [變更追蹤] 頁面上選取 [Linux 檔案]，然後按一下 [+ 新增]，以新增要追蹤的新檔案。
3. 在 [為變更追蹤新增 Linux 檔案] 上，輸入要追蹤的檔案或目錄資訊，然後按一下 [儲存]。

|屬性  |說明  |
|---------|---------|
|已啟用     | 判斷是否已套用設定。        |
|項目名稱     | 要追蹤之檔案的易記名稱。        |
|群組     | 以邏輯方式分組檔案的群組名稱。        |
|輸入路徑     | 要檢查檔案的路徑。 例如："/etc/*.conf"       |
|路徑類型     | 要追蹤的項目類型，可能值為 [檔案] 和 [目錄]。        |
|遞迴     | 決定在尋找所要追蹤的項目時是否使用遞迴。        |
|使用 Sudo     | 此設定會決定在檢查項目時是否使用 sudo。         |
|連結     | 此設定會決定在周遊目錄時處理符號連結的方式。<br> **忽略** - 忽略符號連結，而不包含參考的檔案/目錄。<br>**遵循** - 在遞迴期間遵循符號連結，並且包含參考的檔案/目錄。<br>**管理** - 遵循符號連結並允許變更傳回的內容。     |
|上傳所有的檔案內容設定| 開啟或關閉追蹤變更上的檔案內容上傳。 可用的選項：**True** 或 **False**。|

> [!NOTE]
> 不建議選擇「管理」連結選項。 不支援檔案內容擷取。

### <a name="configure-windows-files-to-track"></a>設定要追蹤的 Windows 檔案

使用下列步驟，設定要在 Windows 電腦上追蹤的檔案：

1. 從您的自動化帳戶中，選取 [組態管理] 下的 [變更追蹤]。 按一下 [編輯設定] (齒輪符號)。
2. 在 [變更追蹤] 頁面上選取 [Windows 檔案]，然後按一下 [+ 新增]，以新增要追蹤的新檔案。
3. 在 [為變更追蹤新增 Windows 檔案] 上，輸入要追蹤之檔案的資訊，然後按一下 [儲存]

|屬性  |說明  |
|---------|---------|
|已啟用     | 判斷是否已套用設定。        |
|項目名稱     | 要追蹤之檔案的易記名稱。        |
|群組     | 以邏輯方式分組檔案的群組名稱。        |
|輸入路徑     | 要檢查檔案的路徑，例如："c:\temp\\\*.txt"<br>您也可以使用環境變數，例如 "%winDir%\System32\\\*.*"       |
|遞迴     | 決定在尋找所要追蹤的項目時是否使用遞迴。        |
|上傳所有的檔案內容設定| 開啟或關閉追蹤變更上的檔案內容上傳。 可用的選項：**True** 或 **False**。|

## <a name="wildcard-recursion-and-environment-settings"></a>萬用字元、遞迴和環境設定

遞迴可讓您指定萬用字元，以簡化跨目錄的追蹤，而環境變數則可讓您以多個或動態的磁碟機名稱跨環境追蹤檔案。 以下列出您在設定遞迴時所應知悉的一般資訊：

* 追蹤多個檔案時需使用萬用字元
* 使用萬用字元時，這些字元只能在路徑的最後一個部分使用。 (例如 C:\folder\\**file** 或 /etc/*.conf)
* 如果環境變數具有無效路徑，驗證仍會成功，但執行清查時，該路徑將會失敗。
* 在設定路徑應避免使用 `c:\*.*` 這類的一般路徑，因為這會導致過多資料夾的周遊。

## <a name="configure-file-content-tracking"></a>設定檔案內容追蹤

您可以使用檔案內容變更追蹤，以檢視檔案變更之前和之後的內容。 此功能可用於 Windows 和 Linux 檔案。對於檔案的每項變更，檔案的內容會儲存在儲存體帳戶中，並以內嵌或並排方式來顯示變更之前和之後的檔案。 若要進一步了解，請參閱[檢視追蹤檔案的內容](change-tracking-file-contents.md)。

![檢視檔案中的變更](./media/change-tracking-file-contents/view-file-changes.png)

### <a name="configure-windows-registry-keys-to-track"></a>設定要追蹤的 Windows 登錄機碼

使用下列步驟，設定要在 Windows 電腦上追蹤的登錄機碼：

1. 從您的自動化帳戶中，選取 [組態管理] 下的 [變更追蹤]。 按一下 [編輯設定] (齒輪符號)。
2. 在 [變更追蹤] 頁面上選取 [Windows 登錄]，然後按一下 [+ 新增]，以新增要追蹤的新登錄機碼。
3. 在 [為變更追蹤新增 Windows 登錄] 上，輸入要追蹤之機碼的資訊，然後按一下 [儲存]。

|屬性  |說明  |
|---------|---------|
|已啟用     | 判斷是否已套用設定。        |
|項目名稱     | 要追蹤之檔案的易記名稱。        |
|群組     | 以邏輯方式分組檔案的群組名稱。        |
|Windows 登錄機碼   | 要檢查檔案的路徑。 例如："HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders\Common Startup"      |

## <a name="limitations"></a>限制

變更追蹤解決方案目前不支援下列項目︰

* Windows 登錄追蹤的遞迴
* 網路檔案系統

其他限制：

* [最大檔案大小] 資料行和值未用於目前實作中。
* 如果您在 30 分鐘的收集週期收集超過 2500 個檔案，則解決方案效能可能會下降。
* 網路流量高時，變更記錄最多可能需要六個小時才會顯示。
* 如果您在關閉電腦時修改組態，電腦可能會提出屬於舊有組態的變更。

## <a name="known-issues"></a>已知問題

「變更追蹤」解決方案目前有下列問題︰

* Windows 10 Creators Update 和 Windows Server 2016 Core RS3 機器不會收集 Hotfix 更新。

## <a name="change-tracking-data-collection-details"></a>「變更追蹤」資料收集詳細資訊

下表顯示各變更類型的資料收集頻率。 對於每個類型，目前狀態的資料快照集也至少會每隔 24 小時重新整理一次：

| **變更類型** | **頻率** |
| --- | --- |
| Windows 登錄 | 50 分鐘 |
| Windows 檔案 | 30 分鐘 |
| Linux 檔案 | 15 分鐘 |
| Windows 服務 | 10 秒到 30 分鐘</br> 預設值：30 分鐘 |
| Linux 精靈 | 5 分鐘 |
| Windows 軟體 | 30 分鐘 |
| Linux 軟體軟體 | 5 分鐘 |

### <a name="windows-service-tracking"></a>Windows 服務追蹤

Windows 服務的預設收集頻率為 30 分鐘。 若要設定頻率，請移至**變更追蹤**。 在 [Windows 服務] 索引標籤上的 [編輯設定] 下有一個滑桿，可以將 Windows 服務的收集頻率從 10 秒變更為長達 30 分鐘。 將滑桿移至您想要的頻率，它會自動儲存。

![Windows 服務滑桿](./media/automation-change-tracking/windowservices.png)

代理程式只會追蹤變更，這樣可最佳化代理程式的效能。 設定過高的閾值時，若服務還原成原始狀態，則可能會遺漏變更。 若將頻率設為較小的值，則可以攔截可能遺漏的變更。

> [!NOTE]
> 雖然代理程式可以追蹤 10 秒間隔內的變更，但資料仍需要幾分鐘才會顯示在入口網站中。 在入口網站中顯示所需時間內仍會追蹤並記錄變更。
  
### <a name="registry-key-change-tracking"></a>登錄機碼變更追蹤

監視登錄機碼變更的目的是找出第三方程式碼和惡意程式碼可啟用的擴充點。 下列清單顯示預先設定的登錄機碼清單。 這些機碼已設定，但並未啟用。 若要追蹤這些登錄機碼，必須每個都啟用。

> [!div class="mx-tdBreakAll"]
> |  |
> |---------|
> |**HKEY\_LOCAL\_MACHINE\Software\Classes\Directory\ShellEx\ContextMenuHandlers**     |
|&nbsp;&nbsp;&nbsp;&nbsp;監視直接連結到 Windows 檔案總管中，且通常使用 Explorer.exe 以內含方式執行的一般自動啟動項目。    |
> |**HKEY\_LOCAL\_MACHINE\Software\Microsoft\Windows\CurrentVersion\Group Policy\Scripts\Startup**     |
|&nbsp;&nbsp;&nbsp;&nbsp;監視在啟動時執行的指令碼。     |
> |**HKEY\_LOCAL\_MACHINE\Software\Microsoft\Windows\CurrentVersion\Group Policy\Scripts\Shutdown**    |
|&nbsp;&nbsp;&nbsp;&nbsp;監視在關機時執行的指令碼。     |
> |**HKEY\_LOCAL\_MACHINE\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Run**     |
|&nbsp;&nbsp;&nbsp;&nbsp;監視使用者登入其 Windows 帳戶之前載入的金鑰。 此金鑰供 64 位元電腦上執行的 32 位元程式使用。    |
> |**HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Active Setup\Installed Components**     |
|&nbsp;&nbsp;&nbsp;&nbsp;監視應用程式設定的變更。     |
> |**HKEY\_LOCAL\_MACHINE\Software\Classes\Directory\ShellEx\ContextMenuHandlers**|
|&nbsp;&nbsp;&nbsp;&nbsp;監視直接連結到 Windows 檔案總管中，且通常使用 Explorer.exe 以內含方式執行的一般自動啟動項目。|
> |**HKEY\_LOCAL\_MACHINE\Software\Classes\Directory\Shellex\CopyHookHandlers**|
|&nbsp;&nbsp;&nbsp;&nbsp;監視直接連結到 Windows 檔案總管中，且通常使用 Explorer.exe 以內含方式執行的一般自動啟動項目。|
> |**HKEY\_LOCAL\_MACHINE\Software\Microsoft\Windows\CurrentVersion\Explorer\ShellIconOverlayIdentifiers**|
|&nbsp;&nbsp;&nbsp;&nbsp;監視圖示覆疊處理常式註冊。|
|**HKEY\_LOCAL\_MACHINE\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Explorer\ShellIconOverlayIdentifiers**|
|&nbsp;&nbsp;&nbsp;&nbsp;針對在 64 位元電腦上執行的 32 位元程式，監視圖示覆疊處理常式註冊。|
> |**HKEY\_LOCAL\_MACHINE\Software\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects**|
|&nbsp;&nbsp;&nbsp;&nbsp;監視 Internet Explorer 新的瀏覽器協助程式物件外掛程式。 用以存取當前頁面的文件物件模型 (DOM) 並控制瀏覽。|
> |**HKEY\_LOCAL\_MACHINE\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects**|
|&nbsp;&nbsp;&nbsp;&nbsp;監視 Internet Explorer 新的瀏覽器協助程式物件外掛程式。 用以存取目前頁面的文件物件模型 (DOM) 並控制 64 位元電腦上執行之 32 位元程式的瀏覽。|
> |**HKEY\_LOCAL\_MACHINE\Software\Microsoft\Internet Explorer\Extensions**|
|&nbsp;&nbsp;&nbsp;&nbsp;監視新的 Internet Explorer 擴充功能，例如自訂工具功能表和自訂工具列按鈕。|
> |**HKEY\_LOCAL\_MACHINE\Software\Wow6432Node\Microsoft\Internet Explorer\Extensions**|
|&nbsp;&nbsp;&nbsp;&nbsp;針對在 64 位元電腦上執行的 32 位元程式，監視新的 Internet Explorer 擴充功能，例如自訂工具功能表和自訂工具列按鈕。|
> |**HKEY\_LOCAL\_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\Drivers32**|
|&nbsp;&nbsp;&nbsp;&nbsp;監視與 wavemapper、wave1 和 wave2、msacm.imaadpcm、.msadpcm、.msgsm610 和 vidc 相關聯的 32 位元驅動程式。 類似 SYSTEM.INI 檔案中的 [drivers] 區段。|
> |**HKEY\_LOCAL\_MACHINE\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32**|
|&nbsp;&nbsp;&nbsp;&nbsp;針對在 64 位元電腦上執行的 32 位元程式，監視與 wavemapper、wave1 和 wave2、msacm.imaadpcm、.msadpcm、.msgsm610 和 vidc 相關聯的 32 位元驅動程式。 類似 SYSTEM.INI 檔案中的 [drivers] 區段。|
> |**HKEY\_LOCAL\_MACHINE\System\CurrentControlSet\Control\Session Manager\KnownDlls**|
|&nbsp;&nbsp;&nbsp;&nbsp;監視已知或常用系統 DLL 清單；此系統可防止人員放入特洛伊木馬病毒版本的系統 DLL 來利用弱式應用程式目錄權限。|
> |**HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Notify**|
|&nbsp;&nbsp;&nbsp;&nbsp;監視可從 Winlogon 接收事件通知的套件清單，而 Winlogon 是 Windows 作業系統的互動式登入支援模型。|

## <a name="use-change-tracking"></a>使用變更追蹤

此解決方案啟用後，您可以在自動化帳戶中選取 [組態管理] 下的 [變更追蹤]，以檢視受監視電腦的變更摘要。

您可以檢視電腦的變更，然後向下鑽研每個事件的詳細資料。 您可以使用圖表頂端的下拉式清單，依據變更類型和時間範圍來限制圖表和詳細資訊。 您也可以按住並拖曳圖表，以選取自訂時間範圍。

![變更追蹤儀表板的影像](./media/automation-change-tracking/change-tracking-dash01.png)

按一下變更或事件，會顯示關於該變更的詳細資訊。 如範例所示，服務的啟動類型已從 [手動] 變更為 [自動]。

![變更追蹤詳細資料的影像](./media/automation-change-tracking/change-tracking-details.png)

## <a name="search-logs"></a>搜尋記錄

除了入口網站中提供的詳細資訊以外，您也可以對記錄執行搜尋。 在 [變更追蹤] 頁面開啟的情況下，按一下 [記錄分析]，[記錄搜尋] 頁面會隨即開啟。

### <a name="sample-queries"></a>範例查詢

下表提供此解決方案所收集之變更記錄的記錄搜尋範例：

|查詢  |說明  |
|---------|---------|
|ConfigurationData<br>&#124; where   ConfigDataType == "WindowsServices" and SvcStartupType == "Auto"<br>&#124; where SvcState == "Stopped"<br>&#124; summarize arg_max(TimeGenerated, *) by SoftwareName, Computer         | 針對已設為 [自動]、但回報為 [正在停止] 的 Windows 服務顯示最新的清查記錄<br>結果會限定為該 SoftwareName 和電腦的最新記錄      |
|ConfigurationChange<br>&#124; where ConfigChangeType == "Software" and ChangeCategory == "Removed"<br>&#124; order by TimeGenerated desc|顯示已移除之軟體的變更記錄|

## <a name="next-steps"></a>後續步驟

瀏覽變更追蹤的教學課程，以深入了解如何使用此解決方案：

> [!div class="nextstepaction"]
> [對您環境中的變更進行疑難排解](automation-tutorial-troubleshoot-changes.md)

* 使用 [Log Analytics 中的記錄檔搜尋](../log-analytics/log-analytics-log-searches.md) ，檢視詳細的變更追蹤資料。
