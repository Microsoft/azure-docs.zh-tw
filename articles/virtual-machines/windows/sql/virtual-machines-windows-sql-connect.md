---
title: 連線到 SQL Server 虛擬機器 (資源管理員) | Microsoft Docs
description: 了解如何連接在 Azure 虛擬機器上執行的 SQL Server。 本主題使用傳統部署模型。 案例會視網路組態和用戶端的位置而有所不同。
services: virtual-machines-windows
documentationcenter: na
author: rothja
manager: craigg
tags: azure-resource-manager
ms.assetid: aa5bf144-37a3-4781-892d-e0e300913d03
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 12/12/2017
ms.author: jroth
ms.openlocfilehash: 522ece2528e43c1037dc6bb707201ecda8074dd9
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38705957"
---
# <a name="connect-to-a-sql-server-virtual-machine-on-azure"></a>連線到 Azure 上的 SQL Server 虛擬機器

## <a name="overview"></a>概觀

本主題說明如何連線到在 Azure 虛擬機器上執行的 SQL Server 執行個體。 它涵蓋了部分[一般連線能力案例](#connection-scenarios)，然後提供[在入口網站中變更連線設定的步驟](#change)。 如果您需要疑難排解或設定入口網站以外的連線，請參閱本主題結尾的[手動設定](#manual)。 

如需有關佈建和連線能力的完整逐步解說，請參閱 [在 Azure 上佈建 SQL Server 虛擬機器](virtual-machines-windows-portal-sql-server-provision.md)。

## <a name="connection-scenarios"></a>連接案例

用戶端連線到在虛擬機器上執行的 SQL Server 方式，取決於用戶端的位置與網路組態。

如果您是在 Azure 入口網站中佈建 SQL Server VM，可以選擇指定 **SQL 連線能力**的類型。

![佈建期間的公用 SQL 連線能力選項](./media/virtual-machines-windows-sql-connect/sql-vm-portal-connectivity.png)

連線能力的選項包括：

| 選項 | 說明 |
|---|---|
| **公開** | 透過網際網路連線到 SQL Server |
| **私用** | 連接相同虛擬網路中的 SQL Server |
| **本機** | 在相同的虛擬機器上本機連線到 SQL Server | 

下列各節更詳細說明**公開**和**私用**選項。

## <a name="connect-to-sql-server-over-the-internet"></a>連接網際網路中的 SQL Server

如果您需要從網際網路連線到您的 SQL Server 資料庫引擎，請於佈建期間，在入口網站中針對 [SQL 連線能力]類型選取 [公開]。 入口網站會自動執行下列步驟：

* 啟用 SQL Server 的 TCP/IP 通訊協定。
* 設定防火牆規則，以開啟 SQL Server TCP 連接埠 (預設為 1433)。
* 可進行公用存取所需的 SQL Server 驗證。
* 將 VM 上的網路安全性群組設定為 SQL Server 連接埠上的所有 TCP 流量。

> [!IMPORTANT]
> SQL Server Developer 和 Express 版本的虛擬機器映像不會自動啟用 TCP/IP 通訊協定。 在 Developer 和 Express 版本中，您必須在建立 VM 之後，使用「SQL Server 組態管理員」來[手動啟用 TCP/IP 通訊協定](#manualtcp)。

任何能夠存取網際網路的用戶端都能藉由指定虛擬機器的公用 IP 位址或指派給該 IP 位址的任何 DNS 標籤，連線到 SQL Server 執行個體。 如果 SQL Server 連接埠是 1433，則不需要在連接字串中指定它。 下列連接字串會使用 SQL 驗證 (您也可以使用公用 IP 位址) 連線到具有 DNS 標籤的 SQL VM`sqlvmlabel.eastus.cloudapp.azure.com`。

```
Server=sqlvmlabel.eastus.cloudapp.azure.com;Integrated Security=false;User ID=<login_name>;Password=<your_password>
```

用戶端雖然可透過網際網路連線，但這不表示任何人都可以連接您的 SQL Server。 外部用戶端必須要有正確的使用者名稱和密碼。 不過，為了增加安全性，您可以避免使用已知的連接埠 1433。 舉例來說，如果您將 SQL Server 設定為在連接埠 1500 進行接聽，並建立適當的防火牆和網路安全性群組規則，您就可以在伺服器名稱附加連接埠號碼來進行連線。 下列範例會將自訂連接埠號碼 **1500** 新增至伺服器名稱，來改變上一個：

```
Server=sqlvmlabel.eastus.cloudapp.azure.com,1500;Integrated Security=false;User ID=<login_name>;Password=<your_password>"
```

> [!NOTE]
> 當您透過網際網路查詢 VM 中的 SQL Server 時，所有從 Azure 資料中心傳出的資料都會以一般[輸出資料傳輸價格](https://azure.microsoft.com/pricing/details/data-transfers/)計費。

## <a name="connect-to-sql-server-within-a-virtual-network"></a>連線到相同虛擬網路內的 SQL Server

當您在入口網站中針對 [SQL 連線能力]類型選擇 [私用] 時，Azure 會將大部分設定設定為與**公用**的相同。 其中一項差異是沒有任何網路安全性群組規則可允許 SQL Server 連接埠 (預設為 1433) 上的外部流量。

> [!IMPORTANT]
> SQL Server Developer 和 Express 版本的虛擬機器映像不會自動啟用 TCP/IP 通訊協定。 在 Developer 和 Express 版本中，您必須在建立 VM 之後，使用「SQL Server 組態管理員」來[手動啟用 TCP/IP 通訊協定](#manualtcp)。

私用連線能力通常與[虛擬網路](../../../virtual-network/virtual-networks-overview.md)搭配使用，可進行數個情節。 您可以連接在相同虛擬網路中的 VM，即使這些 VM 位於不同的資源群組也可以。 [站對站 VPN](../../../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md)可讓您建立能將 VM 連接至內部部署網路和電腦的混合式架構。

虛擬網路也可讓您將 Azure VM 加入網域。 這是在 SQL Server 使用的 Windows 驗證的唯一方式。 其他連接案例則需要使用者名稱和密碼進行 SQL 驗證。

假設您已在虛擬網路中設定 DNS，您便可以在連接字串中指定 SQL Server VM 電腦名稱來連線到 SQL Server 執行個體。 下列範例假設「Windows 驗證」也已設定妥當，且使用者已獲得存取 SQL Server 執行個體的權限。

```
Server=mysqlvm;Integrated Security=true
```

## <a id="change"></a>變更 SQL 連線能力設定

您可以在 Azure 入口網站中變更您 SQL Server 虛擬機器的連線能力設定。

1. 在 Azure 入口網站中，選取 [虛擬機器]。

2. 選取 SQL Server VM。

3. 在 [設定] 下，按一下 [SQL Server 組態]。

4. 將 **SQL 連線能力層級**變更為必要的設定。 可選擇性地使用此區域來變更 SQL Server 連接埠或 SQL 驗證設定。

   ![變更 SQL 連線能力](./media/virtual-machines-windows-sql-connect/sql-vm-portal-connectivity-change.png)

5. 稍候幾分鐘才能完成更新。

   ![SQL VM 更新通知](./media/virtual-machines-windows-sql-connect/sql-vm-updating-notification.png)

## <a id="manualtcp"></a> 針對 Developer 和 Express 版本啟用 TCP/IP

變更 SQL Server 連線能力連線時，Azure 不會對 SQL Server Developer 和 Express 版本自動啟用 TCP/IP 通訊協定。 以下步驟說明如何以手動方式啟用 TCP/IP，以便您經由 IP 位址從遠端連線。

首先，使用遠端桌面連線到 SQL Server 電腦。

[!INCLUDE [Connect to SQL Server VM with remote desktop](../../../../includes/virtual-machines-sql-server-remote-desktop-connect.md)]

接下來，使用 **SQL Server 組態管理員**來啟用 TCP/IP 通訊協定。

[!INCLUDE [Connect to SQL Server VM with remote desktop](../../../../includes/virtual-machines-sql-server-connection-tcp-protocol.md)]

## <a name="connect-with-ssms"></a>以 SSMS 連線

下列步驟示範如何建立 Azure VM 的選擇性 DNS 標籤，然後與 SQL Server Management Studio (SSMS) 連線。

[!INCLUDE [Connect to SQL Server in a VM Resource Manager](../../../../includes/virtual-machines-sql-server-connection-steps-resource-manager.md)]

## <a id="manual"></a>手動設定和疑難排解

雖然入口網站提供自動設定連線的選項，但是您最好知道如何手動設定連線。 了解需求也有助於疑難排解。

下表列出連線到 Azure VM 中執行的 SQL Server 的需求。

| 需求 | 說明 |
|---|---|
| [啟用 SQL Server 驗證模式](https://docs.microsoft.com/sql/database-engine/configure-windows/change-server-authentication-mode#SSMSProcedure) | 除非您已經在虛擬網路上設定 Active Directory，否則，從遠端連線至 VM 需要 SQL Server 驗證。 |
| [建立 SQL 登入](https://docs.microsoft.com/sql/relational-databases/security/authentication-access/create-a-login) | 如果您使用的是 SQL 驗證，則需要包含使用者名稱和密碼，且具備目標資料庫權限的 SQL 登入。 |
| [啟用 TCP/IP 通訊協定](#manualTCP) | SQL Server 必須允許透過 TCP 連線。 |
| [啟用 SQL Server 連接埠的防火牆規則](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access) | VM 上的防火牆必須允許 SQL Server 連接埠 (預設 1433) 上的連入流量。 |
| [建立 TCP 1433 的網路安全性群組規則](../../../virtual-network/manage-network-security-group.md#create-a-security-rule) | 如果您想要透過網際網路連線，則必須允許 VM 接收 SQL Server 連接埠 (預設 1433) 上的流量。 只有本機和虛擬網路之間的連線不需要如此。 這是在 Azure 入口網站所需的唯一步驟。 |

> [!TIP]
> 在入口網站中設定連線時，系統會為您完成上表中的步驟。 僅使用這些步驟確認您的設定，或手動設定 SQL Server 的連線。

## <a name="next-steps"></a>後續步驟

若要查看佈建指示以及連線步驟，請參閱 [在 Azure 上佈建 SQL Server 虛擬機器](virtual-machines-windows-portal-sql-server-provision.md)。

如需在 Azure VM 中執行 SQL Server 的其他相關主題，請參閱 [Azure 虛擬機器上的 SQL Server](virtual-machines-windows-sql-server-iaas-overview.md)。