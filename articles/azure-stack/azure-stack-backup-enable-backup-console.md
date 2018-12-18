---
title: 從系統管理入口網站啟用 Azure Stack 的備份 | Microsoft Docs
description: 透過系統管理入口網站啟用基礎結構備份服務，以便在發生失敗時，可以將 Azure Stack 還原。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: 56C948E7-4523-43B9-A236-1EF906A0304F
ms.service: azure-stack
ms.workload: naS
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/16/2018
ms.author: jeffgilb
ms.openlocfilehash: 6231ee760902618afedf64443690be0b02c4d0eb
ms.sourcegitcommit: d2f2356d8fe7845860b6cf6b6545f2a5036a3dd6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "41946474"
---
# <a name="enable-backup-for-azure-stack-from-the-administration-portal"></a>從系統管理入口網站啟用 Azure Stack 的備份
透過系統管理入口網站啟用基礎結構備份服務，以便 Azure Stack 產生備份。 發生[嚴重失敗](.\azure-stack-backup-recover-data.md)情況時，您可以透過雲端復原使用這些備份來還原環境。 雲端復原的目的在於確保您的操作員和使用者在復原完成後，可以重新登入入口網站。 使用者將會還原其訂用帳戶，包括角色型存取權限和角色、原始方案、供應項目，以及先前定義的計算、儲存體和網路配額。

不過，基礎結構備份服務不會備份 IaaS VM、網路設定和儲存體資源 (例如儲存體帳戶、Blob、資料表等)，因此雲端復原完成之後登入的使用者將不會看到其先前存在的任何資源。 服務也不會備份平台即服務 (PaaS) 資源和資料。 

系統管理員和使用者會負責在基礎結構備份程序之外，個別備份及還原 IaaS 和 PaaS 資源。 如需備份 IaaS 和 PaaS 資源的資訊，請參閱下列連結：

- [虛擬機器](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-manage-vm-protect)
- [App Service](https://docs.microsoft.com/azure/app-service/web-sites-backup)
- [SQL Server](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview)


## <a name="enable-or-reconfigure-backup"></a>啟用或重新設定備份

1. 開啟 [Azure Stack 系統管理入口網站](azure-stack-manage-portals.md)。
2. 選取 [更多服務] > [基礎結構的備份]。 在 [基礎結構備份] 刀鋒視窗中選擇 [設定]。
3. 輸入**備份儲存位置**的路徑。 針對裝載在不同裝置的檔案共用路徑，請使用通用命名慣例 (UNC) 字串。 UNC 字串會指定資源的位置，例如共用的檔案或裝置。 針對服務，您可以使用 IP 位址。 為了在災害發生之後確保備份資料的可用性，裝置應該位於不同的位置。

    > [!Note]  
    > 若您的環境支援從 Azure Stack 基礎結構網路到您企業環境的名稱解析，就可以使用 FQDN，而不必使用 IP。
    
4. 使用有足夠存取權可讀取和寫入檔案的網域和使用者名稱來輸入**使用者名稱**。 例如： `Contoso\backupshareuser`。
5. 輸入使用者的**密碼**。
6. 再次輸入密碼以**確認密碼**。
7. **頻率 (小時)** 可決定建立備份的頻率。 預設值為 12。 排程器所支援的最大值為 12，最小值為 4。 
8. **保留期間 (天)** 可決定在外部位置保留多少天的備份。 預設值為 7。 排程器所支援的最大值為 14，最小值為 2。 備份如果比保留期間舊，系統就會自動從外部位置刪除該備份。

    > [!Note]  
    > 如果您想要將比保留期間舊的備份封存，請務必在排程器刪除備份之前備份檔案。 如果您縮短備份保留期間 (例如從 7 天縮短成 5 天)，排程器將會刪除比新保留期間舊的所有備份。 請先確定您同意刪除那些備份，再更新此值。 

9. 在 [加密金鑰] 方塊中提供預先共用的金鑰。 備份檔案會使用此金鑰來加密。 請務必將此金鑰儲存在安全的位置。 在您第一次設定此金鑰後或在未來更換金鑰時，都無法再從這個介面檢視此金鑰。 若要建立金鑰，請執行下列 Azure Stack PowerShell 命令：
    ```powershell
    New-AzsEncryptionKeyBase64
    ```
10. 選取 [確定] 以儲存備份控制器設定。

    ![Azure Stack - 備份控制器設定](media\azure-stack-backup\backup-controller-settings.png)

## <a name="start-backup"></a>開始備份
若要開始備份，請按一下 [立即備份] 來開始隨選備份。 隨選備份不會修改下一個已排定備份的時間。 在工作完成之後，您可以在 [基本資訊]中確認設定：

![Azure Stack - 隨選備份](media\azure-stack-backup\scheduled-backup.png).

您也可以在 Azure Stack 系統管理電腦上執行 PowerShell Cmdlet **Start-AzsBackup**。 如需詳細資訊，請參閱[備份 Azure Stack](azure-stack-backup-back-up-azure-stack.md)。

## <a name="enable-or-disable-automatic-backups"></a>啟用或停用自動備份
當您啟用備份時，便會自動排定備份。 您可以在 [基本資訊] 中查看下一個排程備份時間。 

![Azure Stack - 隨選備份](media\azure-stack-backup\on-demand-backup.png)

如果您需要停用未來的已排定備份，請按一下 [停用自訂備份]。 停用自動備份會保留已設定的備份設定，並且會保留備份排程。 此動作只是告訴排程器略過未來的備份。 

![Azure Stack - 停用已排定的備份](media\azure-stack-backup\disable-auto-backup.png)

確認在 [基本資訊] 中已停用未來的已排定備份：

![Azure Stack - 確認已停用備份](media\azure-stack-backup\confirm-disable.png)

按一下 [啟用自動備份] 以通知排程器在排定的時間開始未來的備份。 

![Azure Stack - 啟用已排定的備份](media\azure-stack-backup\enable-auto-backup.png)


> [!Note]  
> 如果您在更新至 1807 之前已設定基礎結構備份，自動備份將會停用。 如此一來，由 Azure Stack 啟動的備份就不會與外部工作排程引擎所啟動的備份發生衝突。 在您停用任何外部工作排程器之後，請按一下 [啟用自動備份]。


## <a name="next-steps"></a>後續步驟

- 了解如何執行備份。 請參閱[備份 Azure Stack](azure-stack-backup-back-up-azure-stack.md )。
- 了解如何確認備份已執行完成。 請參閱[在系統管理入口網站中確認備份已完成](azure-stack-backup-back-up-azure-stack.md)。
