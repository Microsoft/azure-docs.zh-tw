---
title: 在 Azure 中建立 Linux SQL Server 2017 VM | Microsoft Docs
description: 本教學課程會示範如何在 Azure 入口網站中建立 Linux SQL Server 2017 虛擬機器。
services: virtual-machines-linux
author: rothja
ms.author: jroth
manager: jhubbard
ms.date: 05/11/2018
ms.topic: hero-article
tags: azure-service-management
ms.devlang: na
ms.service: virtual-machines-sql
ms.workload: iaas-sql-server
ms.technology: database-engine
ms.openlocfilehash: b86dd47c112c38bc65c045158787d19b470899a0
ms.sourcegitcommit: c52123364e2ba086722bc860f2972642115316ef
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/11/2018
ms.locfileid: "34071708"
---
# <a name="provision-a-linux-sql-server-virtual-machine-in-the-azure-portal"></a>在 Azure 入口網站中佈建 Linux SQL Server 虛擬機器

> [!div class="op_single_selector"]
> * [Linux](provision-sql-server-linux-virtual-machine.md)
> * [Windows](../../windows/sql/virtual-machines-windows-portal-sql-server-provision.md)

在本快速入門教學課程中，您可以使用 Azure 入口網站來建立已安裝 SQL Server 2017 的 Linux 虛擬機器。

在本教學課程中，您將：

* [從資源庫建立 Linux SQL VM](#create)
* [透過 SSH 連線到您新的 VM](#connect)
* [變更 SA 密碼](#password)
* [設定遠端連線](#remote)

## <a name="prerequisites"></a>先決條件

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free) 。

## <a id="create"></a>建立已安裝 SQL Server 的 Linux VM

1. 登入 [Azure 入口網站](https://portal.azure.com/)。

1. 在左窗格中，按一下 [建立資源]。

1. 在 [建立資源] 窗格中，按一下 [計算]。

1. 按一下 [精選] 標題旁的 [查看全部]。

   ![查看所有 VM 映像](./media/provision-sql-server-linux-virtual-machine/azure-compute-blade.png)

1. 在 [搜尋] 方塊中，輸入 **SQL Server 2017**，然後按 **Enter** 鍵開始搜尋。

1. 按一下 [篩選] 圖示，將搜尋限制為 [以 Linux 作為基礎]、[Microsoft] 映像，然後按一下 [完成]。

    ![SQL Server 2017 VM 映像的搜尋篩選器](./media/provision-sql-server-linux-virtual-machine/searchfilter.png)

1. 從搜尋結果中選取 SQL Server 2017 Linux 映像。 本教學課程使用**免費的 SQL Server 授權：Red Hat Enterprise Linux 7.4 上的 SQL Server 2017 Developer**。

   > [!TIP]
   > Developer Edition 可讓您使用 Enterprise Edition 的功能進行測試或開發，但不需要 SQL Server 授權費用。 您只需支付執行 Linux VM 的費用。

1. 按一下頁面底部的 [新增] 。

1. 在 [基本概念] 視窗中，填入 Linux VM 的詳細資料。 

    ![基本概念視窗](./media/provision-sql-server-linux-virtual-machine/basics.png)

    > [!Note]
    > 您可以選擇使用 SSH 公開金鑰或密碼進行驗證。 SSH 較為安全。 如需有關如何產生 SSH 金鑰的指示，請參閱[在 Linux 和 Mac 上為 Azure 中的 Linux VM 建立 SSH 金鑰](https://docs.microsoft.com/azure/virtual-machines/virtual-machines-linux-mac-create-ssh-keys)。

1. 按一下 [確定]。

1. 在 [大小] 視窗中，選擇機器大小。 如需關於 VM 機器大小的詳細資訊，請參閱 [Linux VM 大小](https://docs.microsoft.com/azure/virtual-machines/virtual-machines-linux-sizes)。

    ![選擇 VM 大小](./media/provision-sql-server-linux-virtual-machine/vmsizes.png)

   > [!TIP]
   > 針對開發和功能測試，建議使用 **DS2** 或更高版本的 VM 大小。 針對效能測試，使用 **DS13** 或更高版本。

1. 按一下 [選取] 。

1. 在 [設定] 視窗中，選取 [選取公用輸入連接埠] 清單中的 [SSH (22)] 連接埠。 這是在本快速入門中連線並完成 SQL Server 組態所需執行的動作。 如果您想要從遠端連線至 SQL Server，請同時選取 [MS SQL (1433)]，以開啟網際網路連線所需的連接埠 1433。

   ![輸入連接埠](./media/provision-sql-server-linux-virtual-machine/port-settings.png)

1. 您可以對其他設定進行變更，或保留預設設定。 然後按一下 [確定] 。

1. 在 [摘要] 頁面上，按一下 [購買] 以建立 VM。

## <a id="connect"></a> 連線至 Linux VM

如果您已經使用 BASH 殼層，請使用 **ssh** 命令連線到 Azure VM。 在下列命令中，取代 VM 使用者名稱和 IP 位址以連線到您的 Linux VM。

```bash
ssh azureadmin@40.55.55.555
```

您可以在 Azure 入口網站中找到您 VM 的 IP 位址。

![Azure 入口網站中的 IP 位址](./media/provision-sql-server-linux-virtual-machine/vmproperties.png)

如果您是在 Windows 上執行，且沒有 BASH 殼層，可以安裝 SSH 用戶端，例如 PuTTY。

1. [下載並安裝 PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)。

1. 執行 PuTTY。

1. 在 [PuTTY 組態] 畫面上，輸入您 VM 的公用 IP 位址。

1. 按一下 [開啟]，並在提示時輸入您的使用者名稱和密碼。

如需有關連線至 Linux VM 的詳細資訊，請參閱[使用入口網站在 Azure 上建立 Linux VM](https://docs.microsoft.com/azure/virtual-machines/virtual-machines-linux-quick-create-portal#ssh-to-the-vm)。

## <a id="password"></a>變更 SA 密碼

新的虛擬機器會使用隨機的 SA 密碼安裝 SQL Server。 在使用 SA 登入連線到 SQL Server之前，您必須重設此密碼。

1. 連線到您的 Linux VM 之後，開啟新的命令終端機。

1. 使用下列命令變更 SA 密碼：

   ```bash
   sudo systemctl stop mssql-server
   sudo /opt/mssql/bin/mssql-conf set-sa-password
   ```

   出現提示時，輸入新的 SA 密碼和密碼確認。

1. 重新啟動 SQL Server 服務。

   ```bash
   sudo systemctl start mssql-server
   ```

## <a name="add-the-tools-to-your-path-optional"></a>將工具新增至您的路徑 (選擇性)

依預設會安裝數個 SQL Server [套件](sql-server-linux-virtual-machines-overview.md#packages)，包括 SQL Server 命令列工具套件。 工具套件包含 **sqlcmd** 和 **bcp** 工具。 為了方便起見，您可以選擇性地將工具路徑 `/opt/mssql-tools/bin/` 新增至您的 **PATH** 環境變數。

1. 執行下列命令可修改登入工作階段和互動式/非登入工作階段的 **PATH**：

   ```bash
   echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bash_profile
   echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
   source ~/.bashrc
   ```

## <a id="remote"></a>設定遠端連線

如果您需要在 Azure VM 上遠端連線到 SQL Server，就必須設定網路安全性群組上的輸入規則。 此規則可允許 SQL Server 所接聽之連接埠 (預設為 1433) 上的流量。 下列步驟說明如何使用此步驟的 Azure 入口網站。

> [!TIP]
> 如果您在佈建期間於設定中選取了輸入連接埠 [MS SQL (1433)]，則系統會為您執行這些變更。 您可以移至下一節，了解如何設定防火牆。

1. 在入口網站中，選取 [虛擬機器] ，然後選取 SQL Server VM。

1. 在屬性清單中，選取 [網路]。

1. 在 [網路] 視窗中，按一下 [輸入連接埠規則] 底下的 [新增] 按鈕。

   ![輸入連接埠規則](./media/provision-sql-server-linux-virtual-machine/networking.png)

1. 在 [服務] 清單中，選取 [MS SQL]。

    ![MS SQL 安全性群組規則](./media/provision-sql-server-linux-virtual-machine/sqlnsgrule.png)

1. 按一下 [確定]  以儲存 VM 的規則。

### <a name="open-the-firewall-on-rhel"></a>開啟 RHEL 上的防火牆

本教學課程會引導您建立 Red Hat Enterprise Linux (RHEL) VM。 如果您需要遠端連線至 RHEL VM，也需要開啟 Linux 防火牆上的通訊埠 1433。

1. [連線](#connect)到您的 RHEL VM。

1. 在 BASH 殼層中，執行下列命令：

   ```bash
   sudo firewall-cmd --zone=public --add-port=1433/tcp --permanent
   sudo firewall-cmd --reload
   ```

## <a name="next-steps"></a>後續步驟

既然您在 Azure 中擁有 SQL Server 2017 虛擬機器，就可以在本機使用 **sqlcmd** 連線，以執行 Transact-SQL 查詢。

如果您設定遠端 SQL Server 連線的 Azure VM，也應該能夠從遠端連線。 如需如何從 Windows 遠端連線到 Linux 上 SQL Server 的範例，請參閱[使用 Windows 上的 SSMS 連線到 Linux 上的 SQL Server](https://docs.microsoft.com/sql/linux/sql-server-linux-develop-use-ssms)。 若要使用 Visual Studio Code 連線，請參閱[使用 Visual Studio Code 來建立和執行 SQL Server 的 Transact-SQL 指令碼](https://docs.microsoft.com/sql/linux/sql-server-linux-develop-use-vscode)

如需更多關於 Linux 上 SQL Server 的一般資訊，請參閱 [Linux 上的 SQL Server 2017 概觀](https://docs.microsoft.com/sql/linux/sql-server-linux-overview)。 如需使用 SQL Server 2017 Linux 虛擬機器的詳細資訊，請參閱 [Azure 上的 SQL Server 2017 虛擬機器概觀](sql-server-linux-virtual-machines-overview.md)。
