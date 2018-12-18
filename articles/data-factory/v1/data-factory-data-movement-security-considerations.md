---
title: Azure Data Factory 中資料移動的安全性考量 | Microsoft Docs
description: 了解如何在 Azure Data Factory 中保護資料移動。
services: data-factory
documentationcenter: ''
author: nabhishek
manager: craigg
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: abnarain
robots: noindex
ms.openlocfilehash: 222558a6596c676034e52812d3b2dd0c77e1466b
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37046896"
---
# <a name="azure-data-factory---security-considerations-for-data-movement"></a>Azure Data Factory - 資料移動的安全性考量

> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱 [Data Factory 的資料移動安全性考量](../data-movement-security-considerations.md)。

## <a name="introduction"></a>簡介
本文說明 Azure Data Factory 中資料移動服務用來保護您資料的基本安全性基礎結構。 Azure Data Factory 管理資源建置在 Azure 安全性基礎結構上，並使用 Azure 提供的所有可能安全性措施。

在 Data Factory 方案中，您可以建立一或多個資料 [管線](data-factory-create-pipelines.md)。 管線是一起執行某個工作的活動所組成的邏輯群組。 這些管線位於建立 Data Factory 的區域中。 

雖然 Data Factory 只有在「美國西部」、「美國東部」及「北歐」區域才有提供，但[全球數個區域](data-factory-data-movement-activities.md#global)都有提供資料移動服務。 Data Factory 服務可確保資料不會離開某個地區/區域，除非在資料移動服務尚未部署到該區域的情況下，您明確指示服務使用替代的區域。 

Azure Data Factory 本身除了用於雲端資料存放區的已連結服務認證 (會使用憑證加密) 之外，並不會儲存任何資料。 它可讓您建立資料導向工作流程，藉由使用其他區域或內部部署環境中的[計算服務](data-factory-compute-linked-services.md)，協調[所支援資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)之間的資料移動和資料處理。 它也可讓您使用程式設計方式和 UI 機制 [監視和管理工作流程](data-factory-monitor-manage-pipelines.md) 。

使用 Azure Data Factory 進行的資料移動已通過下列各項規範的「認證」：
-   [HIPAA/HITECH](https://www.microsoft.com/en-us/trustcenter/Compliance/HIPAA)  
-   [ISO/IEC 27001](https://www.microsoft.com/en-us/trustcenter/Compliance/ISO-IEC-27001)  
-   [ISO/IEC 27018](https://www.microsoft.com/en-us/trustcenter/Compliance/ISO-IEC-27018) 
-   [CSA STAR](https://www.microsoft.com/en-us/trustcenter/Compliance/CSA-STAR-Certification)
     
如果您對 Azure 法規遵循以及 Azure 如何保護其專屬基礎結構感興趣，請瀏覽 [Microsoft 信任中心](https://microsoft.com/en-us/trustcenter/default.aspx)。 

在本文中，我們會檢閱下列兩個資料移動案例中的安全性考量︰ 

- **雲端案例** - 在此案例中，可透過網際網路公開存取您的來源和目的地。 這些包括受控雲端儲存體服務 (例如「Azure 儲存體」、「Azure SQL 資料倉儲」、Azure SQL Database、Azure Data Lake Store、Amazon S3、Amazon Redshift)、SaaS 服務 (例如 Salesforce)，以及 Web 通訊協定 (例如 FTP 和 OData)。 如需完整的支援資料來源清單，請參閱[這裡](data-factory-data-movement-activities.md#supported-data-stores-and-formats)。
- **混合式案例**- 在此案例中，您的來源或目的地是在防火牆之後或在內部部署的公司網路內，或是資料存放區位於私人網路/虛擬網路中 (最常見的是來源) 而不可公開存取。 裝載在虛擬機器上的資料庫伺服器也屬於此案例的涵蓋範圍。

## <a name="cloud-scenarios"></a>雲端案例
### <a name="securing-data-store-credentials"></a>保護資料存放區認證
Azure Data Factory 可透過使用「受 Microsoft 管理的憑證」來「加密」資料存放區認證，為這些認證提供保護。 這些憑證每隔「兩年」會輪替一次 (包括憑證更新和憑證移轉)。 這些已加密的認證會安全地儲存在「受 Azure Data Factory 管理服務管理的 Azure 儲存體」中。 如需有關「Azure 儲存體」安全性的詳細資訊，請參閱 [Azure 儲存體安全性概觀](../../security/security-storage-overview.md)。

### <a name="data-encryption-in-transit"></a>傳輸中資料加密
如果雲端資料存放區支援 HTTPS 或 TLS，則 Data Factory 中資料移動服務與雲端資料存放區之間的所有資料傳輸，都會透過安全通道 HTTPS 或 TLS。

> [!NOTE]
> 所有連到 **Azure SQL Database** 和「Azure SQL 資料倉儲」的連線在資料透過傳輸進出資料庫時，一律都需要加密 (SSL/TLS)。 使用 JSON 編輯器來編寫管線時，請在「連接字串」中新增 **encryption** 屬性並將它設定為 **true**。 當您使用[複製精靈](data-factory-azure-copy-wizard.md)時，精靈預設會設定這個屬性。 針對「Azure 儲存體」，您可以在連接字串中使用 **HTTPS**。

### <a name="data-encryption-at-rest"></a>待用資料加密
有些資料存放區支援待用資料加密。 建議您為這些資料存放區啟用資料加密機制。 

#### <a name="azure-sql-data-warehouse"></a>Azure SQL 資料倉儲
「Azure SQL 資料倉儲」中的「透明資料加密」(TDE) 可以對待用資料執行即時加密和解密，協助防止惡意活動的威脅。 用戶端並不會察覺到這個過程。 如需詳細資訊，請參閱[保護 SQL 資料倉儲中的資料庫](../../sql-data-warehouse/sql-data-warehouse-overview-manage-security.md)。

#### <a name="azure-sql-database"></a>連接字串
Azure SQL Database 也支援透明資料加密 (TDE)，TDE 可在不需變更應用程式的情況下，對資料執行即時加密和解密，協助防止惡意活動的威脅。 用戶端並不會察覺到這個過程。 若要深入了解，請參閱 [Azure SQL Database 的透明資料加密](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption-with-azure-sql-database)。 

#### <a name="azure-data-lake-store"></a>Azure Data Lake Store
Azure Data Lake Store 也針對儲存在帳戶中的資料提供加密功能。 啟用加密功能時，Data Lake Store 會在保存資料之前先自動加密資料，並在擷取資料之前先解密資料，因此存取該資料的用戶端並不會察覺這個過程。 如需詳細資訊，請參閱 [Azure Data Lake Store 安全性](../../data-lake-store/data-lake-store-security-overview.md)。 

#### <a name="azure-blob-storage-and-azure-table-storage"></a>Azure Blob 儲存體和 Azure 資料表儲存體
「Azure Blob 儲存體」和「Azure 資料表儲存體」支援「儲存體服務加密」(SSE)，此功能會在將資料保存到儲存體之前先自動加密資料，並在擷取資料之前先解密資料。 如需詳細資訊，請參閱[待用資料的 Azure 儲存體服務加密](../../storage/common/storage-service-encryption.md)。

#### <a name="amazon-s3"></a>Amazon S3
Amazon S3 同時支援用戶端和伺服器的待用資料加密。 如需詳細資訊，請參閱[使用加密來保護資料 (英文)](http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingEncryption.html)。 目前，Data Factory 並不支援虛擬私人雲端 (VPC) 內的 Amazon S3。

#### <a name="amazon-redshift"></a>Amazon Redshift
Amazon Redshift 支援叢集待用資料加密。 如需詳細資訊，請參閱 [Amazon Redshift 資料庫加密 (英文)](http://docs.aws.amazon.com/redshift/latest/mgmt/working-with-db-encryption.html)。 目前，Data Factory 並不支援 VPC 內的 Amazon Redshift。 

#### <a name="salesforce"></a>Salesforce
Salesforce 支援「Shield 平台加密」，可加密所有檔案、附件、自訂欄位。 如需詳細資訊，請參閱[了解 Web 伺服器 OAuth 驗證流程 (英文)](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/intro_understanding_web_server_oauth_flow.htm)。  

## <a name="hybrid-scenarios-using-data-management-gateway"></a>混合式案例 (使用資料管理閘道)
混合式案例需要您在內部部署網路中或是虛擬網路 (Azure) 或虛擬私人雲端 (Amazon) 內安裝「資料管理閘道」。 此閘道必須能夠存取本機資料存放區。 如需有關閘道的詳細資訊，請參閱[資料管理閘道](data-factory-data-management-gateway.md)。 

![「資料管理閘道」的通道](media/data-factory-data-movement-security-considerations/data-management-gateway-channels.png)

「命令通道」可允許 Data Factory 中的資料移動服務與「資料管理閘道」之間進行通訊。 此通訊包含活動的相關資訊。 資料通道會用來在內部部署資料存放區與雲端資料存放區之間傳輸資料。    

### <a name="on-premises-data-store-credentials"></a>內部部署資料存放區認證
內部部署資料存放區的認證會儲存在本機 (不是在雲端)。 您可以透過三種不同的方式設定這些認證。 

- 從「Azure 入口網站」/「複製精靈」透過 HTTPS 使用**純文字** (較不安全)。 認證會以純文字形式傳送到內部部署閘道。
- **從複製精靈使用 JavaScript 密碼編譯程式庫**。
- 使用 **Click-Once 型認證管理員應用程式**。 Click-Once 應用程式會在能夠存取閘道的內部部署電腦上執行，並為資料存放區設定認證。 此選項及下一個選項是最安全的選項。 認證管理員應用程式預設會在具有閘道的電腦上使用連接埠 8050 來進行安全通訊。  
- 使用 [New-AzureRmDataFactoryEncryptValue](/powershell/module/azurerm.datafactories/New-AzureRmDataFactoryEncryptValue) PowerShell Cmdlet 來加密認證。 此 cmdlet 會使用閘道器設定用來加密認證的憑證。 您可以使用此 Cmdlet 所傳回的已加密認證，並將它新增到與 [New-AzureRmDataFactoryLinkedService](/powershell/module/azurerm.datafactories/new-azurermdatafactorylinkedservice) Cmdlet 搭配使用之 JSON 檔案中 **connectionString** 的 **EncryptedCredential** 元素中，或新增到入口網站之「Data Factory 編輯器」的 JSON 程式碼片段中。 此選項及 Click-Once 應用程式是最安全的選項。 

#### <a name="javascript-cryptography-library-based-encryption"></a>JavaScript 密碼編譯程式庫型加密
您可以從[複製精靈](data-factory-copy-wizard.md)使用 [JavaScript 密碼編譯程式庫](https://www.microsoft.com/download/details.aspx?id=52439)來加密資料存放區認證。 當您選取此選項時，「複製精靈」會擷取閘道的公開金鑰，然後使用它來加密金鑰存放區認證。 這些認證會由閘道電腦解密，並受到 Windows [DPAPI](https://msdn.microsoft.com/library/ms995355.aspx) 保護。

**支援的瀏覽器：** IE8、IE9、IE10、IE11、Microsoft Edge，以及最新版 Firefox、Chrome、Opera、Safari 瀏覽器。 

#### <a name="click-once-credentials-manager-app"></a>Click-Once 認證管理員應用程式
您可以在編寫管線時，從「Azure 入口網站」/「複製精靈」啟動 Click-Once 型認證管理員應用程式。 此應用程式可確保不會透過網路以純文字形式傳輸認證。 根據預設，它會在具有閘道的電腦上使用連接埠 **8050** 來進行安全通訊。 您可以視需要變更此連接埠。  
  
![閘道的 HTTPS 連接埠](media/data-factory-data-movement-security-considerations/https-port-for-gateway.png)

「資料管理閘道」目前使用單一「憑證」。 此憑證是在安裝閘道時所建立的憑證 (適用於在 2016 年 11 月以後建立及 2.4.xxxx.x 版或更新版本的「資料管理閘道」)。 您可以使用自己的 SSL/TLS 憑證來取代此憑證。 Click-Once 認證管理員應用程式會使用此憑證來安全地連接到閘道電腦，以設定資料存放區認證。 它會使用具有閘道之電腦上的 Windows [DPAPI](https://msdn.microsoft.com/library/ms995355.aspx)，將資料存放區認證安全地儲存在內部部署環境。 

> [!NOTE]
> 舊版閘道如果是在 2016 年 11 月以前所安裝或版本為 2.3.xxxx.x，會繼續使用在雲端加密及儲存的認證。 即使您將閘道升級到最新版本，這些認證也不會移轉到內部部署電腦上    
  
| 閘道版本 (建立時) | 認證儲存位置 | 認證加密/安全性 | 
| --------------------------------- | ------------------ | --------- |  
| 低於或等於 2.3.xxxx.x | 雲端 | 使用憑證 (與認證管理員應用程式所使用的不同) 進行加密 | 
| 高於或等於 2.4.xxxx.x | 內部部署 | 透過 DPAPI 進行保護 | 
  

### <a name="encryption-in-transit"></a>傳輸中加密
與 Azure 服務進行通訊時，所有資料傳輸都會透過安全通道 **HTTPS** 和 **TLS over TCP**，以防止攔截式攻擊。
 
您也可以使用 [IPSec VPN](../../vpn-gateway/vpn-gateway-about-vpn-devices.md) 或 [ExpressRoute](../../expressroute/expressroute-introduction.md) 來進一步保護內部部署網路與 Azure 之間的通訊通道。

虛擬網路是您網路在雲端的邏輯呈現方式。 您可以透過設定 IPSec VPN (站台對站台) 或 ExpressRoute (私用對等)，將內部部署網路連接到 Azure 虛擬網路 (VNet)     

下表根據混合式資料移動的不同來源和目的地位置組合，摘要說明網路和閘道組態的建議事項。

| 來源 | 目的地 | 網路組態 | 閘道安裝 |
| ------ | ----------- | --------------------- | ------------- | 
| 內部部署 | 部署在虛擬網路中的虛擬機器和雲端服務 | IPSec VPN (點對站或站台對站台) | 閘道可以安裝在內部部署環境或 VNet 中的 Azure 虛擬機器 (VM) 上 | 
| 內部部署 | 部署在虛擬網路中的虛擬機器和雲端服務 | ExpressRoute (私用對等) | 閘道可以安裝在內部部署環境或 VNet 中的 Azure VM 上 | 
| 內部部署 | 具有公用端點的 Azure 型服務 | ExpressRoute (公用對等) | 閘道必須安裝在內部部署環境 | 

下列各圖說明在使用「資料管理閘道」的情況下，利用 ExpressRoute 和 IPSec VPN (搭配「虛擬網路」)，在內部部署資料庫與 Azure 服務之間移動資料：

**ExpressRoute：**
 
![使用 ExpressRoute 搭配閘道](media/data-factory-data-movement-security-considerations/express-route-for-gateway.png) 

**IPSec VPN：**

![IPSec VPN 搭配閘道](media/data-factory-data-movement-security-considerations/ipsec-vpn-for-gateway.png)

### <a name="firewall-configurations-and-whitelisting-ip-address-of-gateway"></a>防火牆組態及將閘道的 IP 位址加入允許清單

#### <a name="firewall-requirements-for-on-premisesprivate-network"></a>內部部署/私人網路的防火牆需求  
在企業中，「公司防火牆」會在組織的中央路由器上執行。 而「Windows 防火牆」則是在安裝閘道的本機電腦上以精靈的形式執行。 

下表提供「公司防火牆」的「輸出連接埠」和網域需求。

| 網域名稱 | 輸出連接埠 | 說明 |
| ------------ | -------------- | ----------- | 
| `*.servicebus.windows.net` | 443、80 | 必須提供此資訊，閘道才能連接到 Data Factory 中的資料移動服務 |
| `*.core.windows.net` | 443 | 當您使用[分段複製](data-factory-copy-activity-performance.md#staged-copy)功能時，可供閘道用來連接到「Azure 儲存體帳戶」。 | 
| `*.frontend.clouddatahub.net` | 443 | 必須提供此資訊，閘道才能連接到 Azure Data Factory 服務。 | 
| `*.database.windows.net` | 1433   | (選擇性) 當您的目的地是 Azure SQL Database/「Azure SQL 資料倉儲」時，需要提供此資訊。 若要在不開啟連接埠 1433 的情況下，將資料複製到 Azure SQL Database/「Azure SQL 資料倉儲」，請使用分段複製功能。 | 
| `*.azuredatalakestore.net` | 443 | (選擇性) 當您的目的地是 Azure Data Lake Store 時，需要提供此資訊。 | 

> [!NOTE] 
> 您可能需要依個別資料來源所需，在公司防火牆層級管理連接埠/將網域加入白名單。 此表格僅使用 Azure SQL Database、「Azure SQL 資料倉儲」、Azure Data Lake Store 作為範例。   

下表提供「Windows 防火牆」的「輸入連接埠」需求。

| 輸入連接埠 | 說明 | 
| ------------- | ----------- | 
| 8050 (TCP) | 必須提供此資訊，認證管理員應用程式才能為閘道上的內部部署資料存放區安全地設定認證。 | 

![閘道連接埠需求](media\data-factory-data-movement-security-considerations/gateway-port-requirements.png) 

#### <a name="ip-configurations-whitelisting-in-data-store"></a>資料存放區中的 IP 組態/白名單設定
有些雲端資料存放區也會要求必須將存取它們的電腦之 IP 位址加入允許清單。 請確定在防火牆中已將閘道電腦的 IP 位址正確地加入允許清單並進行設定。

下列雲端資料存放區會要求必須將閘道電腦的 IP 位址加入允許清單。 在這些資料存放區中，有些可能預設不會要求將 IP 位址加入白名單。 

- [Azure SQL Database](../../sql-database/sql-database-firewall-configure.md) 
- [Azure SQL 資料倉儲](../../sql-data-warehouse/sql-data-warehouse-get-started-provision.md)
- [Azure Data Lake Store](../../data-lake-store/data-lake-store-secure-data.md#set-ip-address-range-for-data-access)
- [Azure Cosmos DB](../../cosmos-db/firewall-support.md)
- [Amazon Redshift](http://docs.aws.amazon.com/redshift/latest/gsg/rs-gsg-authorize-cluster-access.html) 

## <a name="frequently-asked-questions"></a>常見問題集

**問：** 是否可以跨不同的 Data Factory 共用閘道？
**答：** 我們尚未支援這項功能。 我們正積極處理這個問題。

**問：** 閘道需要什麼連接埠才能運作？
**答：** 閘道會建立 HTTP 型連線來開啟網際網路。 必須開啟**輸出連接埠 443 和 80**，閘道才能建立此連線。 針對「認證管理員」應用程式，請只在電腦層級 (而非公司防火牆層級) 開啟**輸入連接埠 8050**。 如果使用 Azure SQL Database 或「Azure SQL 資料倉儲」作為來源/目的地，則也需要開啟 **1433** 連接埠。 如需詳細資訊，請參閱[防火牆組態及將 IP 位址加入白名單](#firewall-configurations-and-whitelisting-ip-address-of gateway)一節。 

**問：** 閘道有什麼憑證需求？
**答：** 目前閘道必須要有認證管理員應用程式用來安全地設定資料存放區認證的憑證。 此憑證是閘道安裝程式所建立並設定的自我簽署憑證。 您可以改用自己的 TLS/SSL 憑證。 如需詳細資訊，請參閱 [Click-Once 認證管理員應用程式](#click-once-credentials-manager-app)一節。 

## <a name="next-steps"></a>後續步驟
如需有關複製活動效能的資訊，請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)。

 
