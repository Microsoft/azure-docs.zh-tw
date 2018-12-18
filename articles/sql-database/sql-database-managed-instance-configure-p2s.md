---
title: 設定點對站連線 - Azure SQL Database 受控執行個體 | Microsoft Docs
description: 使用 SQL Server Management Studio，利用點對站連線從內部部署用戶端電腦連線到 Azure SQL Database 受控執行個體。
services: sql-database
ms.service: sql-database
ms.subservice: managed-instance
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: carlrab, bonova, jovanpop
manager: craigg
ms.date: 09/06/2018
ms.openlocfilehash: fc51d7191deb8242075b28fbb42adc2ff9ae2739
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47162996"
---
# <a name="configure-a-point-to-site-connection-to-connect-to-an-azure-sql-database-managed-instance-from-on-premises-computer"></a>設定點對站連線以從內部部署電腦連線到 Azure SQL Database 受控執行個體

此快速入門示範如何使用 [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/sql-server-management-studio-ssms) (SSMS)，利用點對站連線從內部部署用戶端電腦連線到 Azure SQL Database 受控執行個體。 如需點對站連線的詳細資訊，請參閱[關於點對站 VPN](../vpn-gateway/point-to-site-about.md)

## <a name="prerequisites"></a>必要條件

此快速入門：
- 使用[建立受控執行個體](sql-database-managed-instance-get-started.md)這個快速入門中所建立的資源作為起點。
- 您的內部部署用戶端電腦上需要 PowerShell 5.1 與 Azure PowerShell 5.4.2 或更高版本。
- 您的內部部署用戶端電腦上需要最新版 [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/sql-server-management-studio-ssms) (SSMS)

## <a name="attach-a-vpn-gateway-to-your-managed-instance-virtual-network"></a>將 VPN 閘道連結到您的受控執行個體虛擬網路

1. 在您的內部部署用戶端電腦上開啟 PowerShell。
2. 複製此 PowerShell 指令碼並貼上。 此指令碼會將 VPN 閘道連結到您在[建立受控執行個體](sql-database-managed-instance-get-started.md)快速入門中所建立的受控執行個體虛擬網路。 此指令碼會執行下列三個步驟：
  - 在用戶端電腦上建立並安裝憑證
  - 計算未來的 VPN 閘道子網路 IP 範圍
  - 建立 GatewaySubnet
  - 部署 Azure Resource Manager 範本，該範本會將 VPN 閘道連結到 VPN 子網路

   ```powershell
   $scriptUrlBase = 'https://raw.githubusercontent.com/Microsoft/sql-server-samples/master/samples/manage/azure-sql-db-managed-instance/attach-vpn-gateway'

   $parameters = @{
       subscriptionId = '<subscriptionId>'
       resourceGroupName = '<resourceGroupName>'
       virtualNetworkName = '<virtualNetworkName>'
       certificateNamePrefix  = '<certificateNamePrefix>'
       }

   Invoke-Command -ScriptBlock ([Scriptblock]::Create((iwr ($scriptUrlBase+'/attachVPNGateway.ps1?t='+ [DateTime]::Now.Ticks)).Content)) -ArgumentList $parameters, $scriptUrlBase 
   ```

3. 在 PowerShell 指令碼中提供必要參數。 `<subscriptionId>`、`<resourceGroup>` 與 `<virtualNetworkName>` 的值應符合在[建立受控執行個體](sql-database-managed-instance-get-started.md)快速入門中所使用的值。 `<certificateNamePrefix>` 的值可以是您自己選擇的字串。

4. 執行 PowerShell 指令碼。

## <a name="create-a-vpn-connection-to-your-managed-instance"></a>建立受控執行個體的 VPN 連線

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 開啟您在虛擬網路閘道中所建立的資源群組，然後開啟虛擬網路閘道資源。

    ![虛擬網路閘道資源](./media/sql-database-managed-instance-configure-p2s/vnet-gateway.png)  

3. 按一下 [點對站設定]，然後按一下 [下載 VPN 用戶端]。

    ![下載 VPN 用戶端](./media/sql-database-managed-instance-configure-p2s/download-vpn-client.png)  
4. 從 zip 檔案解壓縮檔案，然後開啟解壓縮後的資料夾。 
5. 瀏覽至 [WindowsAmd64] 資料夾，然後開啟 **VpnClientSetupAmd64.exe** 檔案。
6. 如果您收到 [Windows 已保護您的電腦] 訊息，請按一下 [其他資訊]，然後按一下 [仍要執行]。

    ![安裝 VPN 用戶端](./media/sql-database-managed-instance-configure-p2s/vpn-client-defender.png)\
7. 在 [使用者帳戶控制] 對話方塊中，按一下 [是] 以繼續。
8. 在 [MyNewVNet] 對話方塊中，按一下 [是] 以安裝 MyNewVNet 的 VPN 用戶端。

## <a name="connect-to-the-vpn-connection"></a>連線到 VPN 連線。

1. 移至您用戶端電腦上的 VPN 連線，並按一下 [MyNewVNet] 以建立對此 VNet 的連線。

    ![VPN 連線](./media/sql-database-managed-instance-configure-p2s/vpn-connection.png)  
2. 按一下 [連線]。
3. 在 [MyNewVNet] 對話方塊中，按一下 [連線]。

    ![VPN 連線](./media/sql-database-managed-instance-configure-p2s/vpn-connection2.png)  
4. 出現連線管理員需要提高的權限以更新路由表的提示時，請按一下 [繼續]。
5. 在 [使用者帳戶控制] 對話方塊中，按一下 [是] 以繼續。

    ![VPN 連線](./media/sql-database-managed-instance-configure-p2s/vpn-connection-succeeded.png)  

   您已建立對受控執行個體 VNet 的 VPN 連線。

## <a name="use-ssms-to-connect-to-the-managed-instance"></a>使用 SSMS 連線到受控執行個體

1. 在內部部署用戶端電腦上，開啟 SQL Server Management Studio (SSMS)。
 
2. 在 [連接到伺服器] 對話方塊中，於 [伺服器名稱] 方塊中輸入您受控執行個體的完整 [主機名稱]，選取 [SQL Server 驗證]，提供您的登入和密碼，然後按一下 [連線]。

    ![SSMS 連線](./media/sql-database-managed-instance-configure-vm/ssms-connect.png)  

連線之後，您可以檢視 [資料庫] 節點中的系統和使用者資料庫，以及 [安全性]、[伺服器物件]、[複寫]、[管理]、[SQL Server 代理程式] 和 [XEvent Profiler] 節點中的各種物件。

## <a name="next-steps"></a>後續步驟

- 如需如何從 Azure 虛擬機器連線的快速入門，請參閱[設定點對站連線](sql-database-managed-instance-configure-p2s.md)
- 如需有關應用程式連線選項的概觀，請參閱[將您的應用程式連線至受控執行個體](sql-database-managed-instance-connect-app.md)。
- 若要將現有 SQL Server 資料庫從內部部署還原到受控執行個體，您可以使用 [Azure 資料庫移轉服務 (DMS) 來進行移轉](../dms/tutorial-sql-server-to-managed-instance.md)以從資料庫備份檔案還原，或使用 [T-SQL RESTORE 命令](sql-database-managed-instance-get-started-restore.md)以從資料庫備份檔案還原。
