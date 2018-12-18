---
title: 包含檔案
description: 包含檔案
services: virtual-network
author: genlin
ms.service: virtual-network
ms.topic: include
ms.date: 04/13/2018
ms.author: genli
ms.custom: include file
ms.openlocfilehash: 4ae4c3100ae13fdb05e17974b433b247128c1a50
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31805050"
---
## <a name="how-to-create-a-virtual-network-using-a-network-config-file-from-powershell"></a>如何使用 PowerShell 的網路組態檔建立虛擬網路
Azure 會使用 xml 檔案定義訂用帳戶可用的所有虛擬網路。 您可以下載這個檔案，加以編輯以進行修改或刪除現有的虛擬網路，以及建立新的虛擬網路。 在本教學課程中，您將了解如何下載這個檔案 (稱為網路組態或 netcfg 檔案)，以及如何編輯該檔案以建立新的虛擬網路。 若要深入了解網路組態檔，請參閱 [Azure 虛擬網路組態結構描述](https://msdn.microsoft.com/library/azure/jj157100.aspx)。

若要使用 PowerShell 建立具有 netcfg 檔案的虛擬網路，請完成下列步驟︰

1. 如果您從未使用過 Azure PowerShell，請完成[如何安裝和設定 Azure PowerShell](/powershell/azureps-cmdlets-docs) 文章中的步驟，然後登入 Azure 並選取您的訂用帳戶。
2. 從 Azure PowerShell 主控台中，使用 **Get-AzureVnetConfig** Cmdlet，執行下列命令，將網路組態檔下載至您電腦上的目錄： 
   
   ```powershell
   Get-AzureVNetConfig -ExportToFile c:\azure\NetworkConfig.xml
   ```
   
   預期的輸出：
  
      ```
      XMLConfiguration                                                                                                     
      ----------------                                                                                                     
      <?xml version="1.0" encoding="utf-8"?>...
      ```

3. 使用任何 XML 或文字編輯器應用程式，開啟您在步驟 2 中儲存的檔案，並尋找 **<VirtualNetworkSites>** 項目。 如果您已建立網路，每個網路都會顯示為其自身的 **<VirtualNetworkSite>** 項目。
4. 若要建立此案例所述的虛擬網路，請在 **<VirtualNetworkSites>** 元素正下方，新增下列 XML：

   ```xml
         <?xml version="1.0" encoding="utf-8"?>
         <NetworkConfiguration xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/ServiceHosting/2011/07/NetworkConfiguration">
           <VirtualNetworkConfiguration>
             <VirtualNetworkSites>
                 <VirtualNetworkSite name="TestVNet" Location="East US">
                   <AddressSpace>
                     <AddressPrefix>192.168.0.0/16</AddressPrefix>
                   </AddressSpace>
                   <Subnets>
                     <Subnet name="FrontEnd">
                       <AddressPrefix>192.168.1.0/24</AddressPrefix>
                     </Subnet>
                     <Subnet name="BackEnd">
                       <AddressPrefix>192.168.2.0/24</AddressPrefix>
                     </Subnet>
                   </Subnets>
                 </VirtualNetworkSite>
             </VirtualNetworkSites>
           </VirtualNetworkConfiguration>
         </NetworkConfiguration>
   ```
   
5. 儲存網路組態檔。
6. 從 Azure PowerShell 主控台中，使用 **Set-AzureVnetConfig** Cmdlet，執行下列命令以上傳網路組態檔： 
   
   ```powershell
   Set-AzureVNetConfig -ConfigurationPath c:\azure\NetworkConfig.xml
   ```
   
   傳回的輸出︰
   
      ```
      OperationDescription OperationId                          OperationStatus
      -------------------- -----------                          ---------------
      Set-AzureVNetConfig  <Id>                                 Succeeded 
      ```
   
   如果傳回的輸出中 **OperationStatus** 不是 Succeeded，請檢查 xml 檔案中的錯誤並再次完成步驟 6。

7. 從 Azure PowerShell 主控台中，使用 **Get-AzureVnetSite** Cmdlet，執行下列命令以確認已成功新增網路： 

   ```powershell
   Get-AzureVNetSite -VNetName TestVNet
   ```
   
   傳回 (縮短) 的輸出包含下列文字：
  
      ```
      AddressSpacePrefixes : {192.168.0.0/16}
      Location             : Central US
      Name                 : TestVNet
      State                : Created
      Subnets              : {FrontEnd, BackEnd}
      OperationDescription : Get-AzureVNetSite
      OperationStatus      : Succeeded
      ```
