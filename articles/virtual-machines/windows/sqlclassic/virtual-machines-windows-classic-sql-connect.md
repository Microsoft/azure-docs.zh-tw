---
title: 連線到 Azure 上的 SQL Server 虛擬機器 (傳統) | Microsoft Docs
description: 了解如何連接在 Azure 虛擬機器上執行的 SQL Server。 本主題使用傳統部署模型。 案例會視網路組態和用戶端的位置而有所不同。
services: virtual-machines-windows
documentationcenter: na
author: rothja
manager: craigg
tags: azure-service-management
ms.assetid: 416948af-454f-4cfe-8fd2-7cf971cbd3e9
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 01/31/2017
ms.author: jroth
experimental_id: d51f3cc6-753b-4e
ms.openlocfilehash: c856c8c67d410a3b528c4f8b12b1225cf395bca4
ms.sourcegitcommit: a1e1b5c15cfd7a38192d63ab8ee3c2c55a42f59c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "29398364"
---
# <a name="connect-to-a-sql-server-virtual-machine-on-azure-classic-deployment"></a>連線到 Azure 上的 SQL Server 虛擬機器 (傳統部署)
> [!div class="op_single_selector"]
> * [Resource Manager](../sql/virtual-machines-windows-sql-connect.md)
> * [傳統](../classic/sql-connect.md)
> 
> 

## <a name="overview"></a>概觀
本主題說明如何連線到在 Azure 虛擬機器上執行的 SQL Server 執行個體。 其中涵蓋一些[一般連線案例](#connection-scenarios)並提供[在 Azure VM 中設定 SQL Server 連線的詳細步驟](#steps-for-configuring-sql-server-connectivity-in-an-azure-vm)。

> [!IMPORTANT] 
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../azure-resource-manager/resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 如果您是使用 Resource Manager VM，請參閱 [使用 Resource Manager 連接到 Azure 上的 SQL Server 虛擬機器](../sql/virtual-machines-windows-sql-connect.md)。

## <a name="connection-scenarios"></a>連接案例
用戶端連接在虛擬機器上執行的 SQL Server 方式，取決於用戶端的位置與電腦/網路組態。 這些案例包括：

* [連接相同雲端服務中的 SQL Server](#connect-to-sql-server-in-the-same-cloud-service)
* [連接網際網路中的 SQL Server](#connect-to-sql-server-over-the-internet)
* [連接相同虛擬網路中的 SQL Server](#connect-to-sql-server-in-the-same-virtual-network)

> [!NOTE]
> 在使用其中任一種方法來連接之前，您必須遵循 [本文章中的步驟設定連線能力](#steps-for-configuring-sql-server-connectivity-in-an-azure-vm)。
> 
> 

### <a name="connect-to-sql-server-in-the-same-cloud-service"></a>連接相同雲端服務中的 SQL Server
可以在相同的雲端服務中建立多個虛擬機器。 若要了解此虛擬機器案例，請參閱 [如何連接虛擬機器與虛擬網路或雲端服務](../classic/connect-vms-classic.md#connect-vms-in-a-standalone-cloud-service)。 此案例為當虛擬機器上的用戶端嘗試連接到在同一個雲端服務中執行的另一部虛擬機器上的 SQL Server。

在此案例中，您可以使用 VM **名稱** (在入口網站中也稱為**電腦名稱**或**主機名稱**) 來連接。 這是您在建立期間提供給 VM 的名稱。 例如，如果將您的 SQL VM 命名為 **mysqlvm**，則在相同雲端服務中的用戶端 VM 將可以使用下列連接字串來連接：

    "Server=mysqlvm;Integrated Security=false;User ID=<login_name>;Password=<your_password>"

### <a name="connect-to-sql-server-over-the-internet"></a>連接網際網路中的 SQL Server
如果您希望透過網際網路連接您的 SQL Server 資料庫引擎，您必須建立虛擬機器端點以進行傳入 TCP 通訊。 此 Azure 組態步驟能將傳入 TCP 連接埠流量導向虛擬機器可存取的 TCP 連接埠。

若要透過網際網路連接，您必須使用 VM 的 DNS 名稱和 VM 端點連接埠號碼 (稍後在本文中設定)。 若要尋找 DNS 名稱，請瀏覽至 Azure 入口網站，然後選取 [虛擬機器 (傳統)]。 然後選取您的虛擬機器。 [摘要] 區段中會顯示 [DNS 名稱]。

例如，假設名為 **mysqlvm** 的傳統虛擬機器，其 DNS 名稱為 **mysqlvm7777.cloudapp.net**，且 VM 端點是 **57500**。 假設已適當設定連線能力，則使用下列連接字串，就能從網際網路上任何位置存取該虛擬機器：

    "Server=mycloudservice.cloudapp.net,57500;Integrated Security=false;User ID=<login_name>;Password=<your_password>"

用戶端雖然可透過網際網路連線，但這不表示任何人都可以連接您的 SQL Server。 外部用戶端必須要有正確的使用者名稱和密碼。 為了增加安全性，請勿使用知名的 1433 連接埠做為公用虛擬機器端點。 請盡可能考慮在您的端點加入 ACL 來限制流量，只開放給您允許的用戶端。 如需有關在端點中使用 ACL 的指示，請參閱 [在端點上管理 ACL](../classic/setup-endpoints.md#manage-the-acl-on-an-endpoint)。

> [!NOTE]
> 請務必注意，當您使用此技術與 SQL Server 通訊時，所有從 Azure 資料中心傳出的資料都會以一般 [輸出資料傳輸價格](https://azure.microsoft.com/pricing/details/data-transfers/)計費。
> 
> 

### <a name="connect-to-sql-server-in-the-same-virtual-network"></a>連接相同虛擬網路中的 SQL Server
[虛擬網路](../../../virtual-network/virtual-networks-overview.md) 讓其他案例變得可行。 您可連接相同虛擬網路中的 VM，即使這些 VM 位於不同的雲端服務也沒關係。 [站對站 VPN](../../../vpn-gateway/vpn-gateway-site-to-site-create.md)可讓您建立能將 VM 連接至內部部署網路和電腦的混合式架構。

虛擬網路也可讓您將 Azure VM 加入網域。 這是在 SQL Server 使用的 Windows 驗證的唯一方式。 其他連接案例則需要使用者名稱和密碼進行 SQL 驗證。

如果您要設定網域環境及 Windows 驗證，您就不需要使用本文中的步驟來設定公用端點或是 SQL 驗證及登入。 在此案例中，您可以在連接字串中指定 SQL Server VM 名稱來連接 SQL Server 執行個體。 下列範例假設 Windows 驗證也已設定，且使用者已獲得存取 SQL Server 執行個體的權限。

    "Server=mysqlvm;Integrated Security=true"

## <a name="steps-for-configuring-sql-server-connectivity-in-an-azure-vm"></a>Azure VM 中設定 SQL Server 連線的步驟
下列步驟示範如何使用 SQL Server Management Studio (SSMS) 透過網際網路連線到 SQL Server 執行個體。 不過，相同的步驟適用於讓您在內部部署或 Azure 中執行的應用程式可存取 SQL Server 虛擬機器。

您必須先完成後續各小節描述的下列工作，才能從其他 VM 或網際網路連接 SQL Server 的執行個體：

* [為虛擬機器建立 TCP 端點](#create-a-tcp-endpoint-for-the-virtual-machine)
* [在 Windows 防火牆中開啟 TCP 連接埠](#open-tcp-ports-in-the-windows-firewall-for-the-default-instance-of-the-database-engine)
* [設定 SQL Server 以接聽 TCP 通訊協定](#configure-sql-server-to-listen-on-the-tcp-protocol)
* [設定 SQL Server 以進行混合模式驗證](#configure-sql-server-for-mixed-mode-authentication)
* [建立 SQL Server 驗證登入](#create-sql-server-authentication-logins)
* [決定虛擬機器的 DNS 名稱](#determine-the-dns-name-of-the-virtual-machine)
* [從另一部電腦連接到 Database Engine](#connect-to-the-database-engine-from-another-computer)

連線路徑如以下圖表總結：

![連接 SQL Server 虛擬機器](../../../../includes/media/virtual-machines-sql-server-connection-steps/SQLServerinVMConnectionMap.png)

[!INCLUDE [Connect to SQL Server in a VM Classic TCP Endpoint](../../../../includes/virtual-machines-sql-server-connection-steps-classic-tcp-endpoint.md)]

[!INCLUDE [Connect to SQL Server in a VM](../../../../includes/virtual-machines-sql-server-connection-steps.md)]

[!INCLUDE [Connect to SQL Server in a VM Classic Steps](../../../../includes/virtual-machines-sql-server-connection-steps-classic.md)]

## <a name="next-steps"></a>後續步驟
如果您計畫針對高可用性和嚴重損壞修復使用 AlwaysOn 可用性群組，您應該考慮實作接聽程式。 資料庫用戶端會連接至接聽程式，而非直接連接其中一個 SQL Server 執行個體。 接聽程式路由傳送用戶端至可用性群組中的主要複本。 如需詳細資訊，請參閱 [設定 Azure 中 AlwaysOn 可用性群組的 ILB 接聽程式](../classic/ps-sql-int-listener.md)。

請務必檢閱在 Azure 虛擬機器上執行之 SQL Server 的所有安全性最佳做法。 如需詳細資訊，請參閱 [Azure 虛擬機器中的 SQL Server 安全性考量](../sql/virtual-machines-windows-sql-security.md)。

[探索學習路徑](https://azure.microsoft.com/documentation/learning-paths/sql-azure-vm/) ：Azure 虛擬機器上的 SQL Server。 

如需在 Azure VM 中執行 SQL Server 的其他相關主題，請參閱 [Azure 虛擬機器上的 SQL Server](../sql/virtual-machines-windows-sql-server-iaas-overview.md)。

