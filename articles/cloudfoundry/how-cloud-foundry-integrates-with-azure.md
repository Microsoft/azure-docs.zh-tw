---
title: Cloud Foundry 如何與 Azure 整合 | Microsoft Docs
description: 說明 Cloud Foundry 如何利用 Azure 服務來提升企業體驗
services: virtual-machines-linux
documentationcenter: ''
author: ningk
manager: jeconnoc
editor: ''
tags: Cloud-Foundry
ms.assetid: 00c76c49-3738-494b-b70d-344d8efc0853
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 05/11/2018
ms.author: ningk
ms.openlocfilehash: 689730edcc98a23c82373ae8d36c3b831b33c076
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39627435"
---
# <a name="integrate-cloud-foundry-with-azure"></a>整合 Cloud Foundry 與 Azure

[Cloud Foundry](https://docs.cloudfoundry.org/) 是一個在雲端提供者的 IaaS 平台上執行的 PaaS 平台。 它可以跨雲端提供者提供一致的應用程式部屬體驗。 此外，它也可以與各種 Azure 服務整合，可提供企業級 HA、延展性並節省成本。
[Cloud Foundry 有 6 個子系統](https://docs.cloudfoundry.org/concepts/architecture/)，可以線上彈性調整，包括：路由、驗證、應用程式生命週期管理、服務管理、傳訊及監視。 針對每個子系統，您可以設定 Cloud Foundry 來利用對應的 Azure 服務。 

![Azure 整合架構上的 Cloud Foundry](media/CFOnAzureEcosystem-colored.png)

## <a name="1-high-availability-and-scalability"></a>1.高可用性和延展性
### <a name="managed-disk"></a>受控磁碟
Bosh 利用 Azure CPI (雲端提供者介面) 來進行磁碟的建立與刪除例行工作。 預設會使用非受控磁碟。 客戶必須手動建立儲存體帳戶，然後在 CF 資訊清單檔中設定那些帳戶。 這是因為每個儲存體帳戶有磁碟數目限制的緣故。
現在有[受控磁碟](https://azure.microsoft.com/services/managed-disks/)可供使用，可為虛擬機器提供受控的安全可靠磁碟儲存體。 客戶不再需要處理儲存體帳戶來調整規模及達到 HA。 Azure 會自動安排磁碟。 不論是新的還是現有的部署，在 CF 部署期間，Azure CPI 都會處理受控磁碟的建立或移轉。 這是藉由 PCF 1.11 來提供支援。 您也可以探索開放原始碼的 Cloud Foundry [受控磁碟指引](https://github.com/cloudfoundry-incubator/bosh-azure-cpi-release/tree/master/docs/advanced/managed-disks) \(英文)\ 來參考相關資訊。 
### <a name="availability-zone-"></a>可用性區域 *
Cloud Foundry 作為雲端原生應用程式平台，是設計成具備[四層高可用性](https://docs.pivotal.io/pivotalcf/2-1/concepts/high-availability.html)。 前三個軟體失敗層級可由 CF 系統本身處理，但平台容錯則是由雲端提供者所提供。 您應該以雲端提供者的平台 HA 解決方案來保護主要的 CF 元件。 這包括 GoRouters、Diego Brains、CF 資料庫及服務圖格。 資料中心內叢集之間預設會使用 [Azure 可用性設定組](https://github.com/cloudfoundry-incubator/bosh-azure-cpi-release/tree/master/docs/advanced/deploy-cloudfoundry-with-availability-sets)來進行容錯。
好消息是 [Azure 可用性區域](https://docs.microsoft.com/azure/availability-zones/az-overview )現已發行，讓容錯功能更上一層，可跨資料中心提供低延遲備援。
「Azure 可用性區域」會將一組 VM 放在 2 個以上的資料中心內，其中每一組 VM 都是其他組的備援，藉此方式達到 HA。 如果其中一個區域停止運作，另一組仍然會正常運作，而不受災害牽連。
> [!NOTE] 
> 「Azure 可用性區域」尚未提供給所有區域使用，請查看最新的[支援區域清單公告](https://docs.microsoft.com/azure/availability-zones/az-overview)。 針對開放原始碼的 Cloud Foundry，請查看 [適用於開放原始碼 Cloud Foundry 的 Azure 可用性區域指引](https://github.com/cloudfoundry-incubator/bosh-azure-cpi-release/tree/master/docs/advanced/availability-zone) \(英文\)。

## <a name="2-network-routing"></a>2.網路路由
預設會使用 Azure 基本負載平衡器來處理連入 CF API/應用程式要求，將它們轉送給 GoRouters。 Diego Brain、MySQL、ERT 等 CF 元件也可以使用負載平衡器來平衡流量負載，以達到 HA。 此外，Azure 還提供一組完全受控的負載平衡解決方案。 如果您要尋找 TLS 終止 (SSL 卸載) 或每一 HTTP/HTTPS 要求的應用程式層處理，請考慮使用「應用程式閘道」。 針對第 4 層的高可用性和延展性負載平衡，請考慮使用標準負載平衡器。
### <a name="azure-application-gateway-"></a>Azure 應用程式閘道 *
[Azure 應用程式閘道](https://docs.microsoft.com/azure/application-gateway/application-gateway-introduction)提供各種第 7 層負載平衡功能，包括 SSL 卸載、端對端 SSL、Web 應用程式防火牆、Cookie 型工作階段親和性等等。 您可以[在開放原始碼的 Cloud Foundry 中設定應用程式閘道](https://github.com/cloudfoundry-incubator/bosh-azure-cpi-release/tree/master/docs/advanced/application-gateway)。 針對 PCF，請查看 [PCF 2.1 版本資訊](https://docs.pivotal.io/pivotalcf/2-1/pcf-release-notes/opsmanager-rn.html#azure-application-gateway) \(英文\) 來進行 POC 測試。

### <a name="azure-standard-load-balancer-"></a>Azure Standard Load Balancer *
Azure Load Balancer 是第 4 層負載平衡器。 它可用來在負載平衡集內的服務執行個體之間分配流量。 標準版除了基本版功能之外，還提供[進階功能](https://docs.microsoft.com/azure/load-balancer/load-balancer-overview)。 例如 1. 後端集區的 VM 數上限從 100 個提高到 1000 個。  2. 端點現在可支援多個可用性設定組，而不是單一可用性設定組。  3. 額外的功能，例如 HA 連接埠、更豐富的監視資料等等。如果您要移至「Azure 可用性區域」，標準負載平衡器是必備要件。 針對新的部署，建議您從 Azure Standard Load Balancer 開始著手。 

## <a name="3-authentication"></a>3.驗證 
[Cloud Foundry 使用者帳戶和驗證](https://docs.cloudfoundry.org/concepts/architecture/uaa.html)是 CF 及其各種元件的中央身分識別管理服務。 [Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-whatis) 是 Microsoft 的多租用戶雲端式目錄和身分識別管理服務。 預設會使用 UAA 來進行 Cloud Foundry 驗證。 UAA 作為進階選項也支援 Azure AD 作為外部使用者存放區。 Azure AD 使用者可以使用其 LDAP 身分識別來存取 Cloud Foundry，而無需 Cloud Foundry 帳戶。 請依照這些步驟[在 PCF 中設定適用於 UAA 的 Azure AD](http://docs.pivotal.io/p-identity/1-6/azure/index.html)(英文\)。

## <a name="4-data-storage-for-cloud-foundry-runtime-system"></a>4.Cloud Foundry 執行階段系統的資料儲存體
Cloud Foundry 提供絕佳的擴充性，可使用 Azure BlobStore 或 Azure MySQL/PostgreSQL 服務作為應用程式執行階段系統儲存體。
### <a name="azure-blobstore-for-cloud-foundry-cloud-controller-blobstore"></a>適用於 Cloud Foundry 雲端控制器 BlobStore 的 Azure BlobStore
「雲端控制器」BlobStore 是一個用於建置套件、Droplet、套件及資源集區的資料存放區。 預設會將 NFS 伺服器用於「雲端控制器」BlobStore。 為了避免單一失敗點情況，請使用「Azure Blob 儲存體」作為外部存放區。 請查看 [Cloud Foundry 文件](https://docs.cloudfoundry.org/deploying/common/cc-blobstore-config.html) \(英文\) 來了解背景，以及 [Pivotal Cloud Foundry 中的選項](https://docs.pivotal.io/pivotalcf/2-0/customizing/azure.html) \(英文\)。

### <a name="mysqlpostgresql-as-cloud-foundry-elastic-run-time-database-"></a>作為 Cloud Foundry Elastic Runtime 資料庫的 MySQL/PostgreSQL *
CF Elastic Runtime 需要兩個主要的系統資料庫：
#### <a name="ccdb"></a>CCDB 
「雲端控制器」資料庫。  「雲端控制器」提供 REST API 端點供用戶端存取系統。 CCDB 會儲存「雲端控制器」之組織、空間、服務、使用者角色等等的資料表。
#### <a name="uaadb"></a>UAADB 
適用於「使用者帳戶和驗證」的資料庫。 此資料庫會儲存使用者驗證相關資料，例如已加密的使用者名稱和密碼。

預設可以使用本機系統資料庫 (MySQL)。 若要達到 HA 及進行調整，請利用 Azure 受控 MySQL 或 PostgreSQL 服務。 這裡提供[使用開放原始碼 Cloud Foundry 為 CCDB、UAADB 及其他系統資料庫啟用 Azure MySQL/PostgreSQL](https://github.com/cloudfoundry-incubator/bosh-azure-cpi-release/tree/master/docs/advanced/configure-cf-external-databases-using-azure-mysql-postgres-service)的指示。

## <a name="5-open-service-broker"></a>5.Open Service Broker
Azure Service Broker 提供一致的介面來管理應用程式對 Azure 服務的存取。 新的 [Open Service Broker for Azure 專案](https://github.com/Azure/open-service-broker-azure)提供單一且簡單的方式，可將服務提供給遍及 Cloud Foundry、OpenShift 及 Kubernetes 的應用程式。 如需有關 PCF 的部署指示，請參閱 [Azure Open Service Broker for PCF 圖格](https://network.pivotal.io/products/azure-open-service-broker-pcf/) \(英文\)。

## <a name="6-metrics-and-logging"></a>6.計量與記錄
Azure Log Analytics Nozzle 是一個 Cloud Foundry 元件，可將計量從 [Cloud Foundry Loggregator Firehose](https://docs.cloudfoundry.org/loggregator/architecture.html) 轉送至 [Azure Log Analytics](https://azure.microsoft.com/services/log-analytics/)。 在 Nozzle 的協助之下，您可以收集、檢視及分析多個部署的 CF 系統健康情況和效能計量。
請按一下[這裡](https://docs.microsoft.com/azure/cloudfoundry/cloudfoundry-oms-nozzle)，以了解如何將 Azure Log Analytics Nozzle 部署到開放原始碼和 Pivotal Cloud Foundry 環境，然後從 Azure Log Analytics OMS 主控台存取資料。 
> [!NOTE]
> 從 PCF 2.0 開始，預設會將 BOSH 的 VM 健康情況計量轉送到 Loggregator Firehose，並整合到 Azure Log Analytics OMS 主控台中。

## <a name="7-cost-saving"></a>7.節省成本
### <a name="cost-saving-for-devtest-environments"></a>節省開發/測試環境成本
#### <a name="b-series-"></a>B 系列：*
雖然通常針對 Pivotal Cloud Foundry 生產環境建議使用 F 和 D VM 系列，但新的「高載」[B 系列](https://azure.microsoft.com/blog/introducing-b-series-our-new-burstable-vm-size/)帶來了新選項。 B 系列高載的 VM 非常適合不需要持續性完整 CPU 效能的工作負載，例如 Web 伺服器、小型資料庫和開發與測試環境。 這些工作負載通常具有高載的效能需求。 相較於 $0.05/小時 (F1)，它是 $0.012/小時 (B1)，如需詳細資料，請參閱 [VM 大小](https://docs.microsoft.com/azure/virtual-machines/linux/sizes-general)和[價格](https://azure.microsoft.com/pricing/details/virtual-machines/linux/)。 
#### <a name="managed-standard-disk"></a>受控標準磁碟： 
若要確保在生產環境中獲得可靠的效能，建議使用進階磁碟。  使用[受控磁碟](https://azure.microsoft.com/services/managed-disks/)時，標準儲存體也可以提供類似的可靠性，但效能不同。 對於對效能不敏感的工作負載 (例如開發/測試或非重要環境)，受控標準磁碟提供成本較低的替代選項。  
### <a name="cost-saving-in-general"></a>節省一般成本 
#### <a name="significant-vm-cost-saving-with-azure-reservations"></a>使用 Azure 保留來大幅節省 VM 成本： 
現今所有 CF VM 都是使用「隨選」價格計費，即使環境通常無限期保持運作也一樣。 現在您能夠以 1 或 3 年為期來保留 VM 容量，然後獲得 45-65% 折扣。 折扣會在計費系統中套用，不會對您的環境收費。 如需詳細資料，請參閱 [Azure 保留的運作方式](https://azure.microsoft.com/pricing/reserved-vm-instances/)。 
#### <a name="managed-premium-disk-with-smaller-sizes"></a>大小較小的受控進階磁碟： 
受控磁碟針對進階和標準磁碟都可支援較小的磁碟大小，例如 P4 (32 GB) 和 P6 (64 GB)。 如果您有小型工作負載，從標準進階磁碟移轉至受控進階磁碟時，便可節省成本。
#### <a name="utilizing-azure-first-party-services"></a>利用 Azure 第一方服務： 
利用 Azure 的第一方服務除了可獲得上述各節中所提到的 HA 和可靠性之外，還可降低長期的管理成本。 

Pivotal 已針對 PCF 客戶推出[小型使用量 ERT](https://docs.pivotal.io/pivotalcf/2-0/customizing/small-footprint.html)，元件僅共置到 4 個 VM 中，最多可執行 2500 個應用程式執行個體。 現在透過 [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/pivotal.pivotal-cloud-foundry)即可取得試用版。

## <a name="next-steps"></a>後續步驟
Azure 整合功能會先在 [開放原始碼的 Cloud Foundry](https://github.com/cloudfoundry-incubator/bosh-azure-cpi-release/tree/master/docs/advanced/)提供，然後才在 Pivotal Cloud Foundry 提供。 標示 * 的功能表示仍無法透過 PCF 取得。 此外，Cloud Foundry 與 Azure Stack 的整合也不在本文件的涵蓋範圍中。
如需了解有關標示 * 之功能的 PCF 支援或 Cloud Foundry 與 Azure Stack 的整合，請與您的 Pivotal 和 Microsoft 帳戶管理員連絡，以取得最新狀態。 

