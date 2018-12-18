---
title: 在 Azure Site Recovery 中管理處理序伺服器 | Microsoft Doc
description: 本文說明在 Azure Site Recovery 中管理針對 VMware VM 和 實體伺服器複寫設定的處理序伺服器。
author: Rajeswari-Mamilla
ms.service: site-recovery
ms.topic: conceptual
ms.date: 07/21/2018
ms.author: ramamill
ms.openlocfilehash: f8368aab1bc979492143d77a191a31afef754c4c
ms.sourcegitcommit: 248c2a76b0ab8c3b883326422e33c61bd2735c6c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/23/2018
ms.locfileid: "39213005"
---
# <a name="manage-process-servers"></a>管理處理序伺服器

根據預設，您將 VMware VM 或實體伺服器複寫至 Azure 時使用的處理序伺服器，會安裝在內部部署組態伺服器機器上。 您需要在幾個執行個體中設定個別的處理序伺服器：

- 針對大型部署，您可能需要額外的內部部署處理序伺服器才能調整容量。
- 針對容錯回復，您需要在 Azure 中設定臨時處理序伺服器。 容錯回復完成後，您可以刪除此 VM。 

本文摘要說明這些額外處理序伺服器的一般管理工作。

## <a name="upgrade-a-process-server"></a>升級處理序伺服器

升級在內部部署環境或在 Azure 中執行的處理序伺服器 (針對容錯回復目的)，如下所示：

[!INCLUDE [site-recovery-vmware-upgrade -process-server](../../includes/site-recovery-vmware-upgrade-process-server-internal.md)]

> [!NOTE]
  一般而言，當您為了容錯回復目的，在 Azure 中使用 Azure 資源庫映像來建立處理序伺服器時，它會執行可用的最新版本。 Site Recovery 小組會定期發行修正程式和增強功能，我們建議您將處理序伺服器保持在最新狀態。



## <a name="reregister-a-process-server"></a>重新註冊處理序伺服器

如果您需要重新註冊在內部部署環境或在 Azure 中執行的處理序伺服器，請使用組態伺服器執行下列動作：

[!INCLUDE [site-recovery-vmware-register-process-server](../../includes/site-recovery-vmware-register-process-server.md)]

儲存設定之後，請執行下列作業：

1. 在處理序伺服器上，開啟系統管理員命令提示字元。
2. 瀏覽至 **%PROGRAMDATA%\ASR\Agent** 資料夾，然後執行命令：

    ```
    cdpcli.exe --registermt
    net stop obengine
    net start obengine
    ```

## <a name="modify-proxy-settings-for-an-on-premises-process-server"></a>修改內部部署處理序伺服器的 Proxy 設定

如果處理序伺服器使用 Proxy 來連線到 Azure 中的 Site Recovery，當您需要修改現有 Proxy 設定時，請使用此程序。

1. 登入處理序伺服器電腦。 
2. 開啟系統管理 PowerShell 命令視窗並執行下列命令：
  ```
  $pwd = ConvertTo-SecureString -String MyProxyUserPassword
  Set-OBMachineSetting -ProxyServer http://myproxyserver.domain.com -ProxyPort PortNumber –ProxyUserName domain\username -ProxyPassword $pwd
  net stop obengine
  net start obengine
  ```
2. 瀏覽至 **%PROGRAMDATA%\ASR\Agent** 資料夾，然後執行下列命令：
  ```
  cmd
  cdpcli.exe --registermt

  net stop obengine

  net start obengine

  exit
  ```


## <a name="remove-a-process-server"></a>移除處理序伺服器

[!INCLUDE [site-recovery-vmware-unregister-process-server](../../includes/site-recovery-vmware-unregister-process-server.md)]


