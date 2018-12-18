---
title: 在 Azure Site Recovery 中將指令碼新增至復原方案 | Microsoft Docs
description: 深入了解在 Azure 中將 System Center Virtual Machine Manager (VMM) 指令碼新增至復原方案的必要條件。
services: site-recovery
documentationcenter: ''
author: rajani-janaki-ram
manager: rochakm
editor: ''
ms.assetid: 72408c62-fcb6-4ee2-8ff5-cab1218773f2
ms.service: site-recovery
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: storage-backup-recovery
ms.date: 07/06/2018
ms.author: rajanaki
ms.openlocfilehash: 0b2bb17c85f76498e11ea3f007d55d7488f249cf
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39426882"
---
# <a name="add-a-vmm-script-to-a-recovery-plan"></a>將 VMM 指令碼新增至復原方案

本文說明如何建立 System Center Virtual Machine Manager (VMM) 指令碼，並將其新增至 [Azure Site Recovery](site-recovery-overview.md) 中的復原方案。

在這篇文章下方或 [Azure 復原服務論壇](https://social.msdn.microsoft.com/forums/azure/home?forum=hypervrecovmgr)中張貼意見或問題。

## <a name="prerequisites"></a>必要條件

在您的復原計畫中，您可以使用 PowerShell 指令碼。 若要從復原方案進行存取，您必須撰寫指令碼並將指令碼放在 VMM 文件庫中。 當您撰寫指令碼時，請記住下列考量：

* 請確認指令碼使用的是 Try-Catch 區塊，以便能適當地處理例外狀況。
    - 如果在指令碼中發生例外狀況，指令碼會隨即停止執行，而工作也會顯示為失敗。
    - 如果發生錯誤，指令碼的剩餘部分將不會執行。
    - 如果在執行未規劃的容錯移轉時發生錯誤，復原計畫將會繼續執行。
    - 如果在執行未規劃的容錯移轉時發生錯誤，復原計畫將會停止。 請修正指令碼，確認其能如預期執行，然後再次執行復原方案。
        - `Write-Host` 命令無法在復原方案指令碼中運作。 如果您在指令碼中使用 `Write-Host` 命令，指令碼就會失敗。 若要建立輸出，建立接著會執行主要指令碼的 proxy 指令碼。 若要確定所有的輸出都經由管道輸出，請使用 **\>\>** 命令。
        - 如果指令碼未在 600 秒內傳回則會逾時。
        - 如果有任何資料寫入 STDERR 中，則該指令碼將分類為失敗。 此資訊會顯示在指令碼的執行詳細資料中。

* 復原方案中的指令碼會依據 VMM 服務帳戶中的內容執行。 請確定此帳戶具有指令碼所在的遠端共用讀取權限。 測試指令碼，使用與 VMM 服務帳戶相同層級的使用者權限來執行。
* 會在 Windows PowerShell 模組中傳遞 VMM Cmdlet。 安裝 VMM 主控台時，會一併安裝模組。 若要將模組載入您的指令碼中，請在指令碼中使用下列命令︰ 

    `Import-Module -Name virtualmachinemanager`

    如需詳細資訊，請參閱[開始使用 Windows PowerShell 和 VMM](https://technet.microsoft.com/library/hh875013.aspx)。
* 確認您的 VMM 部署中至少有一部文件庫伺服器。 根據預設，VMM 伺服器的程式庫共用路徑位於本機的 VMM 伺服器上。 資料夾名稱為 MSCVMMLibrary。

  如果您的程式庫共用路徑位於遠端 (或如果是位於本機上，但未與 MSCVMMLibrary 共用)，請依以下所示設為共用，使用 \\libserver2.contoso.com\share\ 為例：
  
  1. 開啟登錄編輯程式，並移至 **HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\Azure Site Recovery\Registration**。

  1. 將值 **ScriptLibraryPath** 變更為 **\\\libserver2.contoso.com\share\\**。 指定完整的 FQDN。 提供共用位置的權限。 這是共用的根節點。 若要檢查根節點，在 VMM 中，請移至文件庫中的根節點。 所開啟的路徑是路徑的根目錄。 這是您必須在變數中使用的路徑。

  1. 測試指令碼，方法是使用具有與 VMM 服務帳戶相同層級的使用者權限之使用者帳戶來執行。 使用這些使用者權限來驗證獨立的測試指令碼是以其在復原方案中相同的執行方式來執行。 在 VMM 伺服器上，設定要略過的執行原則，如下所示：

     a. 以系統管理員身分開啟 **64 位元 Windows PowerShell** 主控台。
     
     b. 輸入 **Set-executionpolicy bypass**。 如需詳細資訊，請參閱[使用 Set-ExecutionPolicy Cmdlet](https://technet.microsoft.com/library/ee176961.aspx)。

     > [!IMPORTANT]
     > 只能在 64 位元 PowerShell 主控台中設定 **Set-executionpolicy bypass**。 如果您將它設定為 32 位元 PowerShell 主控台，指令碼就不會執行。

## <a name="add-the-script-to-the-vmm-library"></a>將指令碼新增至 VMM 程式庫

如果您有 VMM 來源站台，可在 VMM 伺服器上建立指令碼。 然後，在復原方案中包含指令碼。

1. 在程式庫共用中，建立新的資料夾。 例如，\<VMM 伺服器名稱>\MSSCVMMLibrary\RPScripts。 將資料夾放在來源和目標 VMM 伺服器上。
1. 建立指令碼。 例如，命名 RPScript 指令碼。 請確認指令碼可以正常運作。
1. 將指令碼放在來源和目標 VMM 伺服器的 \<VMM 伺服器名稱>\MSSCVMMLibrary 資料夾中。

## <a name="add-the-script-to-a-recovery-plan"></a>將指令碼新增至復原方案

在將 VM 或複寫群組新增至復原方案並建立方案之後，您可以將指令碼新增至該群組。

1. 開啟復原計畫。
1. 在 [步驟] 清單中，選取一個項目。 然後，選取 [指令碼] 或 [手動動作]。
1. 指定要在已選取項目之前或之後新增指令碼或動作。 若要將指令碼的位置向上或向下移動，請選取 [上移] 和 [下移] 按鈕。
1. 如果您新增 VMM 指令碼，請選取 [容錯移轉至 VMM 指令碼]。 在 [指令碼路徑] 中，輸入要共用的相對路徑。 例如，輸入 **\RPScripts\RPScript.PS1**。
1. 如果新增 Azure 自動化 Runbook，請指定 Runbook 所在的 Azure 自動化帳戶。 然後，選取您需要使用的 Azure Runbook 指令碼。
1. 若要確保指令碼會如預期般運作，請執行復原方案的測試容錯移轉。


## <a name="next-steps"></a>後續步驟
* 深入了解[如何執行容錯移轉](site-recovery-failover.md)。

