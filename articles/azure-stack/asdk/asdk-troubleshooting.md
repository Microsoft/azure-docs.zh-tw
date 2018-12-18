---
title: Microsoft Azure Stack 疑難排解 | Microsoft Docs
description: Azure Stack 開發套件 (ASDK) 疑難排解資訊。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/15/2018
ms.author: jeffgilb
ms.reviewer: misainat
ms.openlocfilehash: 999017e24f6c4c24735b7e0cc818bf20aaabab62
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49338568"
---
# <a name="microsoft-azure-stack-development-kit-asdk-troubleshooting"></a>Microsoft Azure Stack 開發套件 (ASDK) 疑難排解
此文件提供 ASDK 的常見疑難排解資訊。 若您遇到未記載的問題，請務必檢查 [Azure Stack MSDN 論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=azurestack)以取得進一步的協助與資訊。  

> [!IMPORTANT]
> 因為 ASDK 屬於評估環境，因此 Microsoft 客戶支援服務 (CSS) 不會提供官方支援。

本節中針對疑難排解問題的建議衍生自數個來源，而且不保證能解決您的問題。 程式碼範例係依現況提供，而且無法保證程式碼會產生預期結果。 當產品功能改良時，此節可能也會更新。

## <a name="deployment"></a>部署
### <a name="deployment-failure"></a>部署失敗
若安裝失敗，您可以使用部署指令碼的 -rerun 選項從失敗的步驟重新開始部署 - 如以下範例所示：

  ```powershell
  cd C:\CloudDeployment\Setup
  .\InstallAzureStackPOC.ps1 -Rerun
  ```

### <a name="at-the-end-of-the-deployment-the-powershell-session-is-still-open-and-doesnt-show-any-output"></a>在部署結束時，PowerShell 工作階段仍開啟，而且不會顯示任何輸出
此行為可能只是 PowerShell 命令視窗預設行為的結果 (當已選取它時)。 開發套件部署已成功，但當您選取視窗時，指令碼已暫停。 您可以尋找命令視窗標題列的「選取」字樣以驗證已完成安裝。 按 ESC 鍵以將它取消選取，之後將顯示完成訊息。

## <a name="virtual-machines"></a>虛擬機器
### <a name="default-image-and-gallery-item"></a>預設映像與資源庫項目
在 Azure Stack 中部署 VM 之前，必須先新增 Windows Server 映像與資源庫項目。

### <a name="after-restarting-my-azure-stack-host-some-vms-may-not-automatically-start"></a>重新啟動我的 Azure Stack 主機之後，某些 VM 可能不會自動啟動。
將您的主機重新開機之後，您可能會注意到 Azure Stack 服務並非立即可用。 這是因為 Azure Stack [基礎結構 VM](asdk-architecture.md#virtual-machine-roles) 與 RP 需要一些時間來檢查一致性，但最終將會自動啟動。

您可能也會注意到當 Azure Stack 開發套件主機重新開機之後，租用戶 VM 未自動重新啟動。 這是已知問題，而且只需要手動執行一些步驟就能讓它們上線：

1.  在 Azure Stack 開發套件主機上，從 [開始] 功能表啟動 [容錯移轉叢集管理員]。
2.  選取叢集 **S-Cluster.azurestack.local**。
3.  選取 [角色]。
4.  租用戶 VM 將會顯示為「已儲存」狀態。 一旦所有巳基礎結構 VM 都正常執行，在租用戶 VM 上按一下滑鼠右鍵，然後選取 [啟動] 以繼續該 VM。

### <a name="i-have-deleted-some-virtual-machines-but-still-see-the-vhd-files-on-disk-is-this-behavior-expected"></a>我已刪除某些虛擬機器，但仍在磁碟上看到 VHD 檔案。 這是不是預期行為？
是，這是預期行為。 將它設計成這樣的原因是：

* 當您刪除 VM 時，不會刪除 VHD。 磁碟是資源群組中的獨立資源。
* 刪除儲存體帳戶後，Azure Resource Manager 將會立即反映刪除情況，但其中可能包含的磁碟仍保留在儲存體中，直到執行記憶體回收為止。

若您看到「孤立」的 VHD，您必須知道它是否屬於已刪除之儲存體帳戶的資料夾。 若該儲存體帳戶未被刪除，則它們仍然存在於該處就是正常情況。

您可以在[管理儲存體帳戶](.\.\azure-stack-manage-storage-accounts.md)中深入了解如何設定保留閾值與隨選回收。

## <a name="storage"></a>儲存體
### <a name="storage-reclamation"></a>儲存體回收
已回收的容量最多可能需要 14 小時才會顯示在入口網站中。 空間回收取決於各種因素，包括區塊 Blob 存放區中內部容器檔案的使用量百分比。 因此，視刪除的資料量而定，我們無法保證當記憶體回收行程執行時可回收多少空間。

## <a name="next-steps"></a>後續步驟
[造訪 Azure Stack 支援論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=azurestack)

