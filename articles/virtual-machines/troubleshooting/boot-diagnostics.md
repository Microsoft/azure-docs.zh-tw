---
title: Azure 中的 VM 開機診斷 | Microsoft Docs
description: Azure 中的虛擬機器兩個偵錯功能的概觀
services: virtual-machines
author: Deland-Han
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines
ms.topic: troubleshooting
ms.date: 06/15/2018
ms.author: delhan
ms.openlocfilehash: 64641f8acfe7b58763756e2a0707fa799ee804b2
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47411745"
---
# <a name="how-to-use-boot-diagnostics-to-troubleshoot-virtual-machines-in-azure"></a>如何使用開機診斷對 Azure 中的虛擬機器進行疑難排解

Azure 現在支援兩種偵錯功能︰Azure 虛擬機器 Resource Manager 部署模型的主控台輸出和螢幕擷取畫面支援。 

將自己的映像送至 Azure 或甚至啟動其中一個平台映像時，虛擬機器進入不可開機狀態的原因有很多。 這些功能可讓您輕鬆地診斷及復原開機失敗的虛擬機器。

若為 Linux 虛擬機器，您可以在入口網站輕鬆地檢視主控台記錄的輸出。 若為 Windows 和 Linux 虛擬機器，Azure 也可讓您從 Hypervisor 查看 VM 的螢幕擷取畫面。 所有區域中的 Azure 虛擬機器都支援這兩項功能。 請注意，螢幕擷取畫面和輸出最多可能需要 10 分鐘的時間才會出現在您的儲存體帳戶中。

## <a name="common-boot-errors"></a>常見的開機錯誤

- [0xC000000E](https://support.microsoft.com/help/4010129)
- [0xC000000F](https://support.microsoft.com/help/4010130)
- [0xC0000011](https://support.microsoft.com/help/4010134)
- [0xC0000034](https://support.microsoft.com/help/4010140)
- [0xC0000098](https://support.microsoft.com/help/4010137)
- [0xC00000BA](https://support.microsoft.com/help/4010136)
- [0xC000014C](https://support.microsoft.com/help/4010141)
- [0xC0000221](https://support.microsoft.com/help/4010132)
- [0xC0000225](https://support.microsoft.com/help/4010138)
- [0xC0000359](https://support.microsoft.com/help/4010135)
- [0xC0000605](https://support.microsoft.com/help/4010131)
- [找不到作業系統](https://support.microsoft.com/help/4010142)
- [開機失敗或 INACCESSIBLE_BOOT_DEVICE](https://support.microsoft.com/help/4010143)

## <a name="enable-diagnostics-on-a-new-virtual-machine"></a>在新的虛擬機器上啟用診斷
1. 在 Azure 入口網站建立新的虛擬機器時，請在部署模型下拉式清單中選取 [Azure Resource Manager]︰
 
    ![Resource Manager](./media/virtual-machines-common-boot-diagnostics/screenshot3.jpg)

2. 在 [設定] 中啟用 [開機診斷]，然後選取要放置這些診斷檔案的儲存體帳戶。
 
    ![建立 VM](./media/virtual-machines-common-boot-diagnostics/create-storage-account.png)

    > [!NOTE]
    > 開機診斷功能不支援進階儲存體帳戶。 如果您使用進階儲存體帳戶來進行開機診斷，可能會在啟動虛擬機器時收到 StorageAccountTypeNotSupported 錯誤。
    >
    > 

3. 如果您正以 Azure Resource Manager 範本進行部署，請巡覽至您的虛擬機器的資源並附加診斷設定檔區段。 請記得使用 “2015-06-15” API 版本標頭。

    ```json
    {
          "apiVersion": "2015-06-15",
          "type": "Microsoft.Compute/virtualMachines",
          … 
    ```

4. 診斷設定檔可讓您選取想要放置這些記錄的儲存體帳戶。

    ```json
            "diagnosticsProfile": {
                "bootDiagnostics": {
                "enabled": true,
                "storageUri": "[concat('http://', parameters('newStorageAccountName'), '.blob.core.windows.net')]"
                }
            }
            }
        }
    ```

若要在已啟用開機診斷的情況下部署範例虛擬機器，請至此查看我們的存放庫。

## <a name="enable-boot-diagnostics-on-existing-virtual-machine"></a>在現有的虛擬機器上啟用開機診斷 

若要在現有的虛擬機器上啟用開機診斷功能，請遵循下列步驟：

1. 登入 [Azure 入口網站](https://portal.azure.com)，然後選取虛擬機器。
2. 在 [支援 + 疑難排解] 中，選取 [開機診斷]  > [設定]，將狀態變更為 [開啟]，然後選取儲存體帳戶。 
4. 確認 [開機診斷] 選項已選取，然後儲存變更。

    ![更新現有的 VM](./media/virtual-machines-common-boot-diagnostics/enable-for-existing-vm.png)

3. 重新啟動 VM 才會生效。


