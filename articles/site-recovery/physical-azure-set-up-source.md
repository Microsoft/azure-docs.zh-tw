---
title: 設定來源環境 (實體伺服器至 Azure) | Microsoft Docs
description: 這篇文章說明如何設定內部部署環境，開始將執行 Windows 或 Linux 的實體伺服器複寫至 Azure。
services: site-recovery
author: rayne-wiselman
ms.service: site-recovery
ms.topic: article
ms.date: 07/21/2018
ms.author: raynew
ms.openlocfilehash: 0cbba45ce49667293d8f16bf370424acd70ff78b
ms.sourcegitcommit: 248c2a76b0ab8c3b883326422e33c61bd2735c6c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/23/2018
ms.locfileid: "39213477"
---
# <a name="set-up-the-source-environment-physical-server-to-azure"></a>設定來源環境 (實體伺服器至 Azure)

這篇文章說明如何設定內部部署環境，開始將執行 Windows 或 Linux 的實體伺服器複寫至 Azure。

## <a name="prerequisites"></a>必要條件

本文假設您已經有：
- [Azure 入口網站] (http://portal.azure.com "Azure 入口網站")中的復原服務保存庫。
- 可供安裝設定伺服器的實體電腦。
- 如果您已在您要安裝設定伺服器所在的電腦上停用 TLS 1.0，請確定已啟用 TLS 1.2，並在電腦上安裝 .NET Framework 4.6 或更新版本 (在已停用強式密碼編譯的情況下)。 [深入了解](https://support.microsoft.com/help/4033999/how-to-resolve-azure-site-recovery-agent-issues-after-disabling-tls-1)。

### <a name="configuration-server-minimum-requirements"></a>組態伺服器最低需求
下表列出組態伺服器的最低硬體、軟體與網路需求。
[!INCLUDE [site-recovery-configuration-server-requirements](../../includes/site-recovery-configuration-and-scaleout-process-server-requirements.md)]

> [!NOTE]
> 組態伺服器不支援 HTTPS 型 Proxy 伺服器。

## <a name="choose-your-protection-goals"></a>選擇您的保護目標

1. 在 Azure 入口網站中，移至 [復原服務] 保存庫刀鋒視窗，然後選取您的保存庫。
2. 在保存庫的 [資源] 功能表中，按一下 [開始使用] > [Site Recovery] > [步驟 1：準備基礎結構] > [保護目標]。

    ![選擇目標](./media/physical-azure-set-up-source/choose-goals.png)
3. 在 [保護目標] 中選取 [至 Azure] 和 [未虛擬化/其他]，然後按一下 [確定]。

    ![選擇目標](./media/physical-azure-set-up-source/physical-protection-goal.png)

## <a name="set-up-the-source-environment"></a>設定來源環境

1. 在 [準備來源] 中，如果您沒有組態伺服器，請按一下 [+組態伺服器] 來新增一部伺服器。

  ![設定來源](./media/physical-azure-set-up-source/plus-config-srv.png)
2. 在 [加入伺服器] 刀鋒視窗中，檢查 [組態伺服器] 是否出現在 [伺服器類型] 中。
4. 下載 Site Recovery 統一安裝的安裝檔案。
5. 下載保存庫註冊金鑰。 執行「整合安裝」時，您需要該註冊金鑰。 該金鑰在產生後會維持 5 天有效。

    ![設定來源](./media/physical-azure-set-up-source/set-source2.png)
6. 在作為組態伺服器的電腦上，執行「Azure Site Recovery 整合安裝」，以安裝組態伺服器、處理伺服器和主要目標伺服器。

#### <a name="run-azure-site-recovery-unified-setup"></a>執行 Azure Site Recovery 統一安裝

> [!TIP]
> 如果您電腦系統時鐘的時間與當地時間差五分鐘以上，組態伺服器註冊將會失敗。 開始安裝之前，請先將您的系統時鐘與[時間伺服器](https://technet.microsoft.com/windows-server-docs/identity/ad-ds/get-started/windows-time-service/windows-time-service)同步。

[!INCLUDE [site-recovery-add-configuration-server](../../includes/site-recovery-add-configuration-server.md)]

> [!NOTE]
> 您可以透過命令列安裝組態伺服器。 [深入了解](physical-manage-configuration-server.md#install-from-the-command-line)。


## <a name="common-issues"></a>常見問題

[!INCLUDE [site-recovery-vmware-to-azure-install-register-issues](../../includes/site-recovery-vmware-to-azure-install-register-issues.md)]


## <a name="next-steps"></a>後續步驟

下一個步驟是在 Azure 中[設定目標環境](physical-azure-set-up-target.md)。
