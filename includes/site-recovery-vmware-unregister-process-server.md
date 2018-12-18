---
author: Rajeswari-Mamilla
ms.service: site-recovery
ms.topic: include
ms.date: 08/06/2018
ms.author: ramamill
ms.openlocfilehash: 81390d38b4c0c38b7ac6883ae2bf18c64542fa00
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39582929"
---
取消註冊處理序伺服器的步驟，隨著其與設定伺服器的連線狀態而有所不同。

### <a name="unregister-a-process-server-that-is-in-a-connected-state"></a>取消註冊處於已連線狀態的處理序伺服器

1. 以系統管理員身分遠端登入處理序伺服器。
2. 啟動 [控制台] 並開啟 [程式集] > [解除安裝程式]
3. 解除安裝名為 **Microsoft Azure Site Recovery Configuration/Process Server** 的程式
4. 完成步驟 3 後，您就可以解除安裝 **Microsoft Azure Site Recovery Configuration/Process Server 相依項目**

### <a name="unregister-a-process-server-that-is-in-a-disconnected-state"></a>取消註冊處於已中斷連線狀態的處理序伺服器

> [!WARNING]
> 如果沒辦法恢復處理序伺服器安裝所在的虛擬機器，使用下列步驟應很有用。

1. 以系統管理員身分登入組態伺服器。
2. 開啟系統管理命令提示字元並瀏覽至目錄 `%ProgramData%\ASR\home\svsystems\bin`。
3. 現在執行命令。

    ```
    perl Unregister-ASRComponent.pl -IPAddress <IP_of_Process_Server> -Component PS
    ```
4. 上述命令會提供處理序伺服器 (有重複項目時，可能不只一個) 清單，以及序號 (S.No)、IP 位址 (IP) 和處理序伺服器部署所在 VM 的名稱 (Name) 和 VM 的活動訊號 (Heartbeat)，如下所示。
    ![Unregister-cmd](media/site-recovery-vmware-unregister-process-server/Unregister-cmd.PNG)
5. 現在，輸入您想要取消註冊的處理序伺服器序號。
6. 這會從系統中清除處理序伺服器的詳細資料，並會顯示訊息：**Successfully unregistered server-name> (server-IP-address)**

