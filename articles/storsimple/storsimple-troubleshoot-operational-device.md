---
title: 疑難排解已部署的 StorSimple 裝置 | Microsoft Docs
description: 描述如何診斷和修正在已部署且可運作的 StorSimple 裝置上所發生的錯誤。
services: storsimple
documentationcenter: NA
author: SharS
manager: carmonm
editor: ''
ms.assetid: ea5d89ae-e379-423f-b68b-53785941d9d0
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 11/03/2017
ms.author: v-sharos
ms.openlocfilehash: cf037f7f1c1384b654a7144485d38f569eb7c167
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32187329"
---
# <a name="troubleshoot-an-operational-storsimple-device"></a>可運作的 StorSimple 裝置疑難排解
> [!NOTE]
> StorSimple 的傳統入口網站已過時。 按照淘汰排程，StorSimple 裝置管理員會自動移至新的 Azure 入口網站。 您將收到關於此移動的電子郵件和入口網站通知。 本文件也即將遭到淘汰。 若有關於移動的任何問題，請參閱[常見問題集：移至 Azure 入口網站](storsimple-8000-move-azure-portal-faq.md)。

## <a name="overview"></a>概觀
本文提供實用的疑難排解指導方針，可用來解決在部署 StorSimple 裝置且可運作之後您可能遇到的問題。 文中將描述常見問題、可能原因和建議的步驟，可協助您解決在執行 Microsoft Azure StorSimple 時可能遇到的問題。 此資訊適用於 StorSimple 內部部署實體裝置和 StorSimple 虛擬裝置。

在本文節尾，您可以找到您可能會在 Microsoft Azure StorSimple 作業期間遭遇到的錯誤代碼清單，以及您要解決錯誤所採取的步驟。 

## <a name="setup-wizard-process-for-operational-devices"></a>適用於可運作裝置的安裝精靈程序
您可以使用安裝精靈 ([Invoke-HcsSetupWizard][1]) 檢查裝置設定，並視需要採取更正動作。

當您在先前已設定且可運作的裝置上執行安裝精靈時，程序流程是不一樣的。 您只能變更下列項目：

* IP 位址、子網路遮罩及閘道器
* 主要 DNS 伺服器
* 主要 NTP 伺服器
* 選用的 Web Proxy 設定

安裝精靈不會執行與密碼收集和裝置註冊相關的操作。

## <a name="errors-that-occur-during-subsequent-runs-of-the-setup-wizard"></a>在安裝精靈後續執行期間發生的錯誤
下表說明當您在可運作的裝置上執行安裝精靈時可能遇到的錯誤、產生錯誤的可能原因，以及建議用來解決這些錯誤的動作。 

| 編號 | 錯誤訊息或情況 | 可能的原因 | 建議的動作 |
|:--- |:--- |:--- |:--- |
| 1 |錯誤 350032：此裝置已經停用。 |如果您在已停用的裝置上執行安裝精靈，就會看到這個錯誤。 |[連絡 Microsoft 支援服務](storsimple-contact-microsoft-support.md) 以進行後續步驟。 已停用的裝置無法處於執行狀態。 您可能需要進行原廠重設，裝置才能再次啟用。 |
| 2 |Invoke-HcsSetupWizard：ERROR_INVALID_FUNCTION (發生例外狀況於 HRESULT：0x80070001) |DNS 伺服器更新失敗。 DNS 設定是全域設定，並會在所有已啟用的網路介面上加以套用。 |啟用介面並重新套用 DNS 設定。 因為這些設定是全域的，所以這樣做可能會中斷其他已啟用介面的網路。 |
| 3 |裝置會在 StorSimple Manager 服務入口網站中顯示為線上，但是當您嘗試完成最小安裝並儲存設定時，操作失敗。 |在初始安裝期間，並未設定 Web Proxy，即使已有實際的 Proxy 伺服器存在也一樣。 |使用 [Test-HcsmConnection cmdlet][2] 找出錯誤。 [連絡 Microsoft 支援服務](storsimple-contact-microsoft-support.md) 。 |
| 4 |Invoke-HcsSetupWizard：值未落在預期的範圍內。 |這個錯誤是因為不正確的子網路遮罩而引發。 可能的原因： <ul><li> 子網路遮罩遺失或空白。</li><li>Ipv6 首碼格式不正確。</li><li>介面具備雲端功能，但閘道遺失或不正確。</li></ul>請注意，如果透過安裝精靈設定，DATA 0 自動具備雲端功能。 |若要判斷問題所在，請使用子網路 0.0.0.0 或 256.256.256.256，然後查看輸出。 視需要針對子網路遮罩、閘道器及 Ipv6 首碼輸入正確的值。 |

## <a name="error-codes"></a>錯誤碼
錯誤以數字順序列出。

| 錯誤號碼 | 錯誤文字或描述 | 建議的使用者動作 |
|:--- |:--- |:--- |
| 10502 |存取儲存體帳戶時發生錯誤。 |請等候幾分鐘，然後再次嘗試。 如果問題持續發生，請連絡 Microsoft 支援服務以進行後續步驟。 |
| 40017 |備份作業失敗，因為在裝置上找不到備份原則中指定的磁碟區。 |請重試備份作業，如果錯誤持續發生，請連絡 Microsoft 支援。 後續步驟。 |
| 40018 |備份作業失敗，因為在裝置上找不到備份原則中指定的磁碟區。 |請重試備份作業，如果錯誤持續發生，請連絡 Microsoft 支援。 後續步驟。 |
| 390061 |系統忙碌或無法使用。 |請等候幾分鐘，然後再次嘗試。 如果問題持續發生，請連絡 Microsoft 支援服務以進行後續步驟。 |
| 390143 |發生錯誤碼為 390143 的錯誤。 (未知的錯誤。) |如果問題持續發生，請連絡 Microsoft 支援服務以進行後續步驟。 |

## <a name="next-steps"></a>後續步驟
如果您無法解決問題，請 [連絡 Microsoft 支援服務](storsimple-contact-microsoft-support.md) ，尋求協助。 

[1]: https://technet.microsoft.com/%5Clibrary/Dn688135(v=WPS.630).aspx
[2]: https://technet.microsoft.com/%5Clibrary/Dn715782(v=WPS.630).aspx
