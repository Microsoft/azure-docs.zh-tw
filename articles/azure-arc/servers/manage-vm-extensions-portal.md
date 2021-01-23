---
title: 從 Azure 入口網站啟用 VM 擴充功能
description: 本文說明如何從 Azure 入口網站將虛擬機器擴充功能部署到在混合式雲端環境中執行的 Azure Arc 啟用的伺服器。
ms.date: 01/22/2020
ms.topic: conceptual
ms.openlocfilehash: 43bbcef28b77e7c7112880fdac1bbd4809791cef
ms.sourcegitcommit: 78ecfbc831405e8d0f932c9aafcdf59589f81978
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2021
ms.locfileid: "98728939"
---
# <a name="enable-azure-vm-extensions-from-the-azure-portal"></a>從 Azure 入口網站啟用 Azure VM 擴充功能

本文說明如何透過 Azure 入口網站，將 Azure Arc 啟用的伺服器所支援的 Azure VM 擴充功能部署和卸載至 Linux 或 Windows 混合式電腦。

> [!NOTE]
> Key Vault VM 延伸模組 (preview) 不支援從 Azure 入口網站部署、僅使用 Azure CLI、Azure PowerShell 或使用 Azure Resource Manager 範本。

## <a name="enable-extensions-from-the-portal"></a>從入口網站啟用擴充功能

您可以透過 Azure 入口網站，將 VM 擴充功能套用至伺服器管理的電腦。

1. 在瀏覽器中，移至 [Azure 入口網站](https://portal.azure.com)。

2. 在入口網站中，流覽至 [ **伺服器-Azure Arc** ]，然後從清單中選取您的混合式電腦。

3. 選擇 [ **擴充** 功能]，然後選取 [ **新增**]。 請從可用擴充功能清單選擇您想要的擴充功能，並遵循精靈中的指示。 在此範例中，我們會部署 Log Analytics VM 擴充功能。

    ![選取所選機器的 VM 擴充功能](./media/manage-vm-extensions/add-vm-extensions.png)

    下列範例顯示從 Azure 入口網站安裝 Log Analytics VM 擴充功能：

    ![安裝 Log Analytics VM 擴充功能](./media/manage-vm-extensions/mma-extension-config.png)

    若要完成安裝，您必須提供工作區識別碼和主要金鑰。 如果您不熟悉如何尋找此資訊，請參閱 [取得工作區識別碼和金鑰](../../azure-monitor/platform/log-analytics-agent.md#workspace-id-and-key)。

4. 確認所提供的必要資訊之後，請選取 [ **建立**]。 系統會顯示部署的摘要，您可以查看部署的狀態。

>[!NOTE]
>雖然多個擴充功能可以一起批次處理和處理，但它們是以順序安裝。 第一個延伸模組安裝完成之後，會嘗試安裝下一個擴充功能。

## <a name="list-extensions-installed"></a>已安裝清單延伸模組

您可以從 Azure 入口網站取得已啟用 Arc 之伺服器上的 VM 擴充功能清單。 請執行下列步驟來查看它們。

1. 在瀏覽器中，移至 [Azure 入口網站](https://portal.azure.com)。

2. 在入口網站中，流覽至 [ **伺服器-Azure Arc** ]，然後從清單中選取您的混合式電腦。

3. 選擇 [ **擴充** 功能]，並傳回已安裝的擴充功能清單。

    ![列出部署到所選電腦的 VM 擴充功能](./media/manage-vm-extensions/list-vm-extensions.png)

## <a name="uninstall-extension"></a>卸載擴充功能

您可以從 Azure 入口網站移除已啟用 Arc 之伺服器的一或多個延伸模組。 請執行下列步驟來移除擴充功能。

1. 在瀏覽器中，移至 [Azure 入口網站](https://portal.azure.com)。

2. 在入口網站中，流覽至 [ **伺服器-Azure Arc** ]，然後從清單中選取您的混合式電腦。

3. 選擇 [ **擴充** 功能]，然後從已安裝的擴充功能清單中選取擴充功能。

4. 選取 [ **卸載** ]，當系統提示您確認時，請選取 **[是]** 以繼續。

## <a name="next-steps"></a>後續步驟

- 您可以使用 [Azure CLI](manage-vm-extensions-cli.md)、 [PowerShell](manage-vm-extensions-powershell.md)或 [Azure Resource Manager 範本](manage-vm-extensions-template.md)來部署、管理和移除 VM 擴充功能。

- 疑難排解資訊可在 [VM 延伸模組指南](troubleshoot-vm-extensions.md)中找到。