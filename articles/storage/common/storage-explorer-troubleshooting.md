---
title: Azure 儲存體總管疑難排解指南 | Microsoft Docs
description: Azure 兩個偵錯功能的概觀
services: virtual-machines
author: Deland-Han
ms.service: virtual-machines
ms.topic: troubleshooting
ms.date: 06/15/2018
ms.author: delhan
ms.component: common
ms.openlocfilehash: eb72d92496addacd82e4d30df625b4f9c0c823e5
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39521025"
---
# <a name="azure-storage-explorer-troubleshooting-guide"></a>Azure 儲存體總管疑難排解指南

Microsoft Azure 儲存體總管是一個獨立應用程式，可讓您在 Windows、macOS 和 Linux 上輕鬆使用 Azure 儲存體資料。 應用程式可以連線至裝載於 Azure、National Clouds 和 Azure Stack 上的儲存體帳戶。

本指南摘要說明儲存體總管中常見的問題解決方案。

## <a name="error-self-signed-certificate-in-certificate-chain-and-similar-errors"></a>錯誤：憑證鏈結中的自我簽署憑證 (和類似錯誤)

憑證錯誤是由下列兩種情況的其中一個所造成：

1. 應用程式是透過「透明 proxy」連線，這表示伺服器 (例如您公司的伺服器) 是使用自我簽署憑證攔截 HTTPS 流量、解密再加密。
2. 您正在執行的應用程式，會將自我簽署的 SSL 憑證插入您收到的 HTTPS 訊息中。 會插入憑證的應用程式例子包括防毒軟體和網路流量檢查軟體。

當儲存體總管看見自我簽署或不受信任的憑證，就無法再得知收到的 HTTPS 訊息是否已更改。 如果您有一份自我簽署憑證，便可以執行下列步驟，指示儲存體總管信任該憑證：

1. 取得憑證的 Base-64 編碼 X.509 (.cer) 複本
2. 按一下 [編輯] > [SSL 憑證] > [匯入憑證]，然後使用檔案選擇器來尋找、選取及開啟 .cer 檔案

此問題也可能是因為有多個憑證 (根憑證與中繼憑證)。 必須新增這兩個憑證，才能解決錯誤。

如果您不確定憑證來自何處，可以嘗試下列步驟找出憑證：

1. 安裝 Open SSL

    * [Windows](https://slproweb.com/products/Win32OpenSSL.html) (任一輕裝版足可應對)
    * Mac 和 Linux：應該包含在作業系統中
2. 執行 Open SSL

    * Windows：開啟安裝目錄，按一下 **/bin/**，再按兩下 **openssl.exe**。
    * Mac 和 Linux：從終端機執行 **openssl**。
3. 執行 `s_client -showcerts -connect microsoft.com:443`
4. 尋找自我簽署憑證。 如果不確定哪些是自我簽署的憑證，請尋找主旨 `("s:")` 和簽發者 `("i:")` 相同的所有位置。
5. 發現任何自我簽署的憑證時，請將每個憑證從 **-----BEGIN CERTIFICATE-----** 到 **-----END CERTIFICATE-----** (含) 的所有內容，複製並貼到新的 .cer 檔案。
6. 開啟儲存體總管，按一下 [編輯] > [SSL 憑證] > [匯入憑證]，然後使用檔案選擇器來尋找、選取及開啟您所建立的 .cer 檔案。

如果使用上述步驟找不到任何自我簽署的憑證，請透過意見反應工具與我們連絡，以取得更多協助。 或者，您可以選擇從命令列使用 `--ignore-certificate-errors` 旗標來啟動儲存體總管。 使用這個旗標啟動時，儲存體總管會忽略憑證錯誤。

## <a name="sign-in-issues"></a>登入問題

### <a name="reauthentication-loop-or-upn-change"></a>重新驗證迴圈或 UPN 變更
如果您處於重新驗證的迴圈中，或已變更其中一個帳戶的 UPN，請嘗試下列方法：
1. 移除所有的帳戶，然後關閉 [儲存體總管]
2. 從您的機器中刪除 .IdentityService 資料夾。 在 Windows 中，該資料夾位於 `C:\users\<username>\AppData\Local`。 對於 Mac 和 Linux，您可以在使用者目錄的根目錄中找到此資料夾。
3. 如果您使用 Mac 或 Linux，您也必須從您的作業系統金鑰儲存區中刪除 Microsoft.Developer.IdentityService 項目。 在 Mac 上，金鑰儲存區會是 "Gnome Keychain" 應用程式。 針對 Linux，此應用程式通常稱為 "Keyring"，但此名稱可能會因為您的散發版本不同而有差異。

## <a name="mac-keychain-errors"></a>Mac Keychain 錯誤
macOS 鑰匙圈有時會進入導致 [儲存體總管] 的驗證程式庫發生問題的狀態。 若要使鑰匙圈脫離這種狀態，請嘗試下列步驟：
1. 關閉 [儲存體總管]。
2. 開啟鑰匙圈 (**cmd + 空格鍵**，鍵入 keychain，按 Enter)。
3. 選取 [登入] 鑰匙圈。
4. 按一下掛鎖圖示以鎖定鑰匙圈 (完成時，掛鎖會以動畫方式顯示到鎖定的位置，視您所開啟的應用程式而定，它可能需要幾秒鐘的時間)。

    ![映像](./media/storage-explorer-troubleshooting/unlockingkeychain.png)

5. 啟動儲存體總管。
6. 快顯視窗應該會隨即出現，並包含類似「服務中樞想要存取 keychain」的訊息。 若有出現此快顯視窗，請輸入您的 Mac 系統管理員帳戶密碼，並按一下 [一律允許] (或是，如果沒有 [一律允許]，則按一下 [允許])。
7. 嘗試登入。

### <a name="general-sign-in-troubleshooting-steps"></a>一般登入疑難排解步驟
* 如果您位於 macOS 上，且登入視窗不曾出現在「正在等候驗證...」對話方塊中，請嘗試[這些步驟](#Mac-Keychain-Errors)
* 重新啟動儲存體總管
* 如果驗證視窗空白，請在關閉驗證對話方塊之前先等待至少一分鐘。
* 確認您的機器和儲存體總管都已正確設定 Proxy 和憑證設定。
* 如果您是在 Windows 上，而且能夠在相同機器上存取 Visual Studio 2017 並登入，請嘗試登入 Visual Studio 2017。 成功登入 Visual Studio 2017 之後，您應該能夠開啟 [儲存體總管]，並在 [帳戶] 面板中查看您的帳戶。 

如果這些方法都沒有用，請[在 GitHub 上開立問題](https://github.com/Microsoft/AzureStorageExplorer/issues)。

### <a name="missing-subscriptions-and-broken-tenants"></a>訂用帳戶遺失和租用戶損毀

如果成功登入後無法擷取您的訂用帳戶，請嘗試下列疑難排解方法：

* 確認您的帳戶可存取預期的訂用帳戶。 您可以登入想要使用的 Azure 環境入口網站，以確認存取。
* 確定已使用正確的 Azure 環境 (Azure、Azure 中國、Azure 德國、Azure 美國政府或自訂環境) 來登入。
* 如果您是在 proxy 背景，請確定已正確設定儲存體總管的 proxy。
* 嘗試移除再重新新增帳戶。
* 如果有 [詳細資訊] 連結，請查看針對失敗租用戶回報的錯誤訊息內容。 如果您不確定該如何處理看到的錯誤訊息，歡迎您在 [GitHub 上提出問題](https://github.com/Microsoft/AzureStorageExplorer/issues)。

## <a name="cannot-remove-attached-account-or-storage-resource"></a>無法移除連結的帳戶或儲存體資源

如果您透過使用者介面無法移除連結的帳戶或儲存體資源，則可以刪除下列資料夾，以手動刪除連結的所有資源：

* Windows：`%AppData%/StorageExplorer`
* macOS：`/Users/<your_name>/Library/Applicaiton Support/StorageExplorer`
* Linux：`~/.config/StorageExplorer`

> [!NOTE]
>  請先關閉儲存體總管，然後再刪除上述資料夾。

> [!NOTE]
>  如果您曾經匯入任何 SSL 憑證，請備份 `certs` 目錄的內容。 稍後，您可以使用備份來重新匯入 SSL 憑證。

## <a name="proxy-issues"></a>Proxy 問題

首先，確定您輸入的下列資訊都正確：

* Proxy URL 和連接埠號碼
* 使用者名稱和密碼 (如果 proxy 要求)

### <a name="common-solutions"></a>常見的解決方案

如果您仍然遇到問題，請嘗試下列疑難排解方法：

* 如果可以不使用 proxy 連接到網際網路，請確認儲存體總管不啟用 proxy 設定也能運作。 如果是這種情況，可能是 proxy 設定發生問題。 請和您的 proxy 系統管理員合作找出問題。
* 確認使用 proxy 伺服器的其他應用程式是否如預期般運作。
* 確認您可以連線至想要使用的 Azure 環境入口網站
* 確認您可以收到服務端點的回應。 在瀏覽器中輸入其中一個端點 URL。 如果可以連線，您應該會收到 InvalidQueryParameterValue 或類似的 XML 回應。
* 如果有其他人也用您的 proxy 伺服器使用儲存體總管，請確認他們可以連線。 如果他們可以連線，您可能必須連絡您的 proxy 伺服器管理員。

### <a name="tools-for-diagnosing-issues"></a>診斷問題的工具

如果您有網路工具，例如 Fiddler for Windows，您可以診斷問題，如下所示：

* 如果必須透過 proxy 工作，您可能必須設定網路工具透過 proxy 連線。
* 檢查網路工具使用的連接埠號碼。
* 輸入本機主機 URL 和網路工具的連接埠號碼，當做儲存體總管的 proxy 設定。 如果作業正確，您的網路工具就會開始記錄儲存體總管向管理和服務端點提出的網路要求。 例如，在瀏覽器中輸入 blob 端點 https://cawablobgrs.blob.core.windows.net/，便會收到類似下面的回應，代表雖然您無法存取，但資源確實存在。

![程式碼範例](./media/storage-explorer-troubleshooting/4022502_en_2.png)

### <a name="contact-proxy-server-admin"></a>連絡 proxy 伺服器管理員

如果 proxy 設定正確，您可能要連絡您的 proxy 伺服器管理員，並

* 確定 proxy 不會封鎖 Azure 管理或資源端點的流量。
* 確認 proxy 伺服器使用的驗證通訊協定。 儲存體總管目前不支援 NTLM proxy。

## <a name="unable-to-retrieve-children-error-message"></a>「無法擷取子系」錯誤訊息

如果透過 proxy 連線至 Azure，請確認您的 proxy 設定正確無誤。 如已獲授權可存取訂用帳戶或帳戶擁有者的資源，請確認您已閱讀或列出該資源的權限。

## <a name="issues-with-sas-url"></a>SAS URL 問題
如果您使用 SAS URL 連線到服務，而碰到此錯誤：

* 請確認 URL 提供讀取或列出資源的必要權限。
* 請確認 URL 尚未過期。
* 如果 SAS URL 是以存取原則為基礎，請確認尚未撤銷存取原則。

如果您不慎以無效的 SAS URL 進行連結，但是無法中斷連結，請遵循下列步驟：
1.  執行儲存體總管時，按下 F12 以開啟開發人員工具視窗。
2.  按一下 [應用程式] 索引標籤，然後在左邊樹狀目錄中，按一下 [本機儲存體] > file://。
3.  尋找與有問題的 SAS URI 服務類型相關聯的索引鍵。 例如，如果是 Blob 容器的 SAS URI 不正確，請尋找名為 `StorageExplorer_AddStorageServiceSAS_v1_blob` 的索引鍵。
4.  索引鍵的值應該是 JSON 陣列。 尋找與不正確 URI 相關聯的物件，並將它移除。
5.  按下 Ctrl+R，重新載入儲存體總管。

## <a name="linux-dependencies"></a>Linux 相依項目

針對 Ubuntu 16.04 以外的 Linux 散發版本，您可能需要手動安裝某些相依性。 一般而言，必要的套件如下：
* [.NET Core 2.x](https://docs.microsoft.com/dotnet/core/linux-prerequisites?tabs=netcore2x)
* `libsecret`
* `libgconf-2-4`
* 最新的 GCC

依據您的散發版本，您可能需要安裝其他套件。 儲存體總管[版本資訊](https://go.microsoft.com/fwlink/?LinkId=838275&clcid=0x409)包含某些散發版本的特定步驟。

## <a name="next-steps"></a>後續步驟

如果這些解決方法對您都沒有用，請[在 GitHub 上開立問題](https://github.com/Microsoft/AzureStorageExplorer/issues)。 您也可以使用左下角的 [回報問題到 GitHub] 按鈕，以快速前往 GitHub。

![意見反應](./media/storage-explorer-troubleshooting/feedback-button.PNG)
