---
title: 包含檔案
description: 包含檔案
services: iot-accelerators
author: dominicbetts
ms.service: iot-accelerators
ms.topic: include
ms.date: 08/16/2018
ms.author: dobett
ms.custom: include file
ms.openlocfilehash: b2b4bfc6aa03039a7eca402f7a9af083a44f0829
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43346609"
---
## <a name="access-the-virtual-machine"></a>存取虛擬機器

下列步驟會在 Azure Cloud Shell 中使用 `az` 命令。 如果您想要的話，也可以在開發機器上[安裝 Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-azure-cli)，並在本機執行命令。

下列步驟說明如何設定 Azure 虛擬機器，以允許進行 **SSH** 存取。 所顯示的步驟假設您為解決方案加速器選擇的名稱是 **contoso-simulation** -- 請將此值更換為部署名稱：

1. 列出包含解決方案加速器資源的資源群組內容：

    ```azurecli-interactive
    az resource list -g contoso-simulation -o table
    ```

    請記下虛擬機器的名稱、公用 IP 位址和網路安全性群組 - 稍後需要用到這些值。

1. 更新網路安全性群組以允許進行 SSH 存取。 下列命令假設網路安全性群組的名稱是 **contoso-simulation-nsg** -- 請將此值更換為網路安全性群組的名稱：

    ```azurecli-interactive
    az network nsg rule update --name SSH --nsg-name contoso-simulation-nsg -g contoso-simulation --access Allow -o table
    ```

    請只在測試和開發期間啟用 SSH 存取。 如果您啟用 SSH，[您應儘速再次加以停用](https://docs.microsoft.com/azure/security/azure-security-network-security-best-practices#disable-rdpssh-access-to-azure-virtual-machines)。

1. 將虛擬機器上的 **azureuser** 密碼，更新為您知道的密碼。 當您執行下列命令時，請選擇您自己的密碼：

    ```azurecli-interactive
    az vm user update --name vm-vikxv --username azureuser --password YOURSECRETPASSWORD  -g contoso-simulation
    ```

1. 尋找虛擬機器的公用 IP 位址。 下列命令假設虛擬機器的名稱是 **vm-vikxv** -- 請將此值更換為您先前記下的虛擬機器名稱：

    ```azurecli-interactive
    az vm list-ip-addresses --name vm-vikxv -g contoso-simulation -o table
    ```

    請記下虛擬機器的公用 IP 位址。
