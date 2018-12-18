---
title: Azure SQL Database 受控執行個體連線應用程式 | Microsoft Docs
description: 本文討論如何將您的應用程式連線到 Azure SQL Database 受控執行個體。
services: sql-database
ms.service: sql-database
ms.subservice: managed-instance
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: bonova, carlrab
manager: craigg
ms.date: 09/14/2018
ms.openlocfilehash: f57d582aacad568811314494c0ed614839ccabba
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47221735"
---
# <a name="connect-your-application-to-azure-sql-database-managed-instance"></a>將您的應用程式連線到 Azure SQL Database 受控執行個體

您在決定如何及何處裝載應用程式時，目前有多個選擇。 
 
您可以選擇使用 Azure App Service 或一些 Azure 的虛擬網路 (VNet) 整合式選項 (例如 Azure App Service 環境、虛擬機器、虛擬機器擴展集)，在雲端中裝載應用程式。 您也可以採用混合式雲端方法，將您的應用程式留在內部部署環境。 
 
無論您的選擇是什麼，您都可以將它連線到受控執行個體。  

![高可用性](./media/sql-database-managed-instance/application-deployment-topologies.png)  
## <a name="connect-an-application-inside-the-same-vnet"></a>連線相同 VNet 內的應用程式 

這個案例最簡單。 VNet 內的虛擬機器即使位於不同的子網路，彼此也可以直接連線。 這表示如果您要連線 Azure 應用程式環境或虛擬機器內的應用程式，只要適當地設定連接字串即可。  
 
## <a name="connect-an-application-inside-a-different-vnet"></a>連線不同 VNet 內的應用程式 

這個案例比較複雜一點，因為受控執行個體在它自己的 VNet 中具有私人 IP 位址。 若要連線，應用程式需要存取受控執行個體部署所在的 VNet。 因此，首先您必須建立應用程式與受控執行個體 VNet 之間的連線。 VNet 不一定要在相同的訂用帳戶中，此案例就可以運作。 
 
有兩個選項可以連線 VNet： 
- [Azure 虛擬網路對等互連](../virtual-network/virtual-network-peering-overview.md) 
- VNet 對 VNet VPN 閘道 ([Azure 入口網站](../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)、[PowerShell](../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md)、[Azure CLI](../vpn-gateway/vpn-gateway-howto-vnet-vnet-cli.md)) 
 
對等互連選項是偏好選項，因為對等互連使用 Microsoft 骨幹網路，所以從連線的觀點來看，對等互連 VNet 與相同 VNet 中虛擬機器之間的延遲沒有明顯差異。 VNet 對等互連受限於相同區域中的網路。  
 
> [!IMPORTANT]
> 由於[全域虛擬網路對等互連的條件約束](../virtual-network/virtual-network-manage-peering.md#requirements-and-constraints)，受控執行個體的 VNet 對等互連案例會受限於相同區域中的網路。 

## <a name="connect-an-on-premises-application"></a>連線內部部署應用程式 

受控執行個體只能透過私人 IP 位址存取。 若要從內部部署環境存取它，您需要進行應用程式與受控執行個體 VNet 之間的站對站連線。 
 
有兩個選項可以連線內部部署與 Azure VNet： 
- 站對站 VPN 連線 ([Azure 入口網站](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md)、[PowerShell](../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md)、[Azure CLI](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-cli.md)) 
- [ExpressRoute](../expressroute/expressroute-introduction.md) 連線  
 
如果您已成功建立內部部署環境與 Azure 的連線，但是無法建立與受控執行個體的連線，請檢查您的防火牆是否開啟 SQL 通訊埠 1433 的輸出連線，以及開啟 11000-12000 範圍的連接埠以便重新導向。 

## <a name="connect-an-application-on-the-developers-box"></a>連線開發人員方塊上的應用程式

受控執行個體只能透過私人 IP 位址存取，因此若要從開發人員方塊存取它，首先您需要進行開發人員方塊與受控執行個體 VNet 之間的連線。 若要這麼做，請使用原生 Azure 憑證驗證設定 VNet 的點對站連線。 如需詳細資訊，請參閱[設定點對站連線以從內部部署電腦連線到 Azure SQL Database 受控執行個體](sql-database-managed-instance-configure-p2s.md)。

## <a name="connect-from-on-premises-with-vnet-peering"></a>使用 VNet 對等互連從內部部署連線
客戶所實作的另一種案例是，VPN 閘道安裝在個別的虛擬網路中，而訂用帳戶則位於裝載受控執行個體的虛擬網路中。 這兩個虛擬網路隨後會對等互連。 下列範例架構圖顯示其實作方式。

![VNet 對等互連](./media/sql-database-managed-instance-connect-app/vnet-peering.png)

在您設定基本的基礎結構後，您必須修改某些設定，使 VPN 閘道可以看到裝載受控執行個體的虛擬網路中所包含的 IP 位址。 若要這麼做，請在 [對等互連設定] 下方設定下列特定變更。
1.  在裝載 VPN 閘道的 VNet 中移至 [對等互連]，接著移至受控執行個體的對等互連 VNet 連線，然後按一下 [允許閘道傳輸]。
2.  在裝載受控執行個體的 VNet 中移至 [對等互連]，接著移至 VPN 閘道的對等互連 VNet 連線，然後按一下 [使用遠端閘道]。

## <a name="connect-an-azure-app-service-hosted-application"></a>連線 Azure App Service 裝載應用程式 

受控執行個體只能透過私人 IP 位址存取，因此若要從 Azure App Service 存取它，首先您需要進行應用程式與受控執行個體 VNet 之間的連線。 請參閱[將您的應用程式與 Azure 虛擬網路整合](../app-service/web-sites-integrate-with-vnet.md)。  
 
如需疑難排解，請參閱[針對 VNet 和應用程式進行疑難排解](../app-service/web-sites-integrate-with-vnet.md#troubleshooting)。 如果無法建立連線，請嘗試[同步處理網路組態](sql-database-managed-instance-sync-network-configuration.md)。 
 
將 Azure App Service 連線到受控執行個體的特殊案例，就是當您將 Azure App Service 整合到已對等互連至受控執行個體 VNet 的網路時。 這種情況下需要設定下列組態： 

- 受控執行個體 VNet 不能有閘道  
- 受控執行個體 VNet 必須設定 [使用遠端閘道] 選項 
- 對等互連 VNet 必須設定 [允許閘道傳輸] 選項 
 
下圖說明此案例：

![整合式應用程式對等互連](./media/sql-database-managed-instance/integrated-app-peering.png)
 
## <a name="troubleshooting-connectivity-issues"></a>對連線問題進行疑難排解

若要對連線問題進行疑難排解，請檢閱以下幾點︰
- 如果您無法從位於相同 VNet、但子網路不同的 Azure 虛擬機器連線至受控執行個體時，請檢查您是否在 VM 子網路上設定了可能會封鎖存取的網路安全性群組。此外請注意，您必須在 SQL 連接埠 1433 和 11000-12000 範圍內的連接埠上開啟輸出連線，因為在 Azure 界限內透過重新導向連線時需要這些連接埠。 
- 確定已針對與 VNet 相關聯的路由表將 BGP 傳播設定為 [已啟用]。
- 如果使用 P2S VPN，請檢查 Azure 入口網站中的組態，以確認您是否看到**輸入/輸出**數值。 非零的數值表示 Azure 會將流量路由至內部部署，或從中輸出流量。

   ![輸入/輸出數值](./media/sql-database-managed-instance-connect-app/ingress-egress-numbers.png)

- 確認用戶端電腦 (執行 VPN 用戶端的電腦) 具有您需要存取之所有 Vnet 的路由項目。 路由會儲存在 `%AppData%\ Roaming\Microsoft\Network\Connections\Cm\<GUID>\routes.txt`.


   ![route.txt 中](./media/sql-database-managed-instance-connect-app/route-txt.png)

   如圖所示，每個相關的 VNet 都有兩個項目，且在入口網站中設定的 VPN 端點會有第三個項目。

   除此之外，也可以透過下列命令來檢查路由。 下列輸出顯示各種子網路的路由： 

   ```cmd
   C:\ >route print -4
   ===========================================================================
   Interface List
   14...54 ee 75 67 6b 39 ......Intel(R) Ethernet Connection (3) I218-LM
   57...........................rndatavnet
   18...94 65 9c 7d e5 ce ......Intel(R) Dual Band Wireless-AC 7265
   1...........................Software Loopback Interface 1
   Adapter===========================================================================
   
   IPv4 Route Table
   ===========================================================================
   Active Routes:
   Network Destination        Netmask          Gateway       Interface  Metric
          0.0.0.0          0.0.0.0       10.83.72.1     10.83.74.112     35
         10.0.0.0    255.255.255.0         On-link       172.26.34.2     43
     
         10.4.0.0    255.255.255.0         On-link       172.26.34.2     43
   ===========================================================================
   Persistent Routes:
   None
   ```

- 如果使用 VNet 對等互連，請確定您已依照[允許閘道傳輸和使用遠端閘道](#connect-from-on-premises-with-vnet-peering)的設定指示操作。 

## <a name="required-versions-of-drivers-and-tools"></a>所需的驅動程式和工具版本

如果您想要連線到受控執行個體，建議您至少使用下列版本的工具和驅動程式：

| 驅動程式/工具 | 版本 |
| --- | --- |
|.NET Framework | 4.6.1 (或 .NET Core) | 
|ODBC 驅動程式    | v17 |
|PHP 驅動程式 | 5.2.0 |
|JDBC 驅動程式    | 6.4.0 |
|Node.js 驅動程式 | 2.1.1 |
|OLEDB 驅動程式   | 18.0.2.0 |
|SSMS   | 17.8.1 或[更高版本](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-2017) |

## <a name="next-steps"></a>後續步驟

- 如需受控執行個體的詳細資訊，請參閱[受控執行個體是什麼](sql-database-managed-instance.md)。
- 如需示範如何新建受控執行個體的教學課程，請參閱[建立受控執行個體](sql-database-managed-instance-get-started.md)。
