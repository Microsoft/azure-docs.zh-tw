---
title: 使用 Azure App Service Environment
description: 如何在 Azure App Service Environment 中建立、發佈及調整應用程式
services: app-service
documentationcenter: na
author: ccompy
manager: stefsch
ms.assetid: a22450c4-9b8b-41d4-9568-c4646f4cf66b
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/13/2017
ms.author: ccompy
ms.openlocfilehash: 80abe29c80898b691aa6e5e47bf068a9e69e50e4
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44303365"
---
# <a name="use-an-app-service-environment"></a>使用 App Service Environment #

## <a name="overview"></a>概觀 ##

Azure App Service Environment (ASE) 是 Azure App Service 到客戶之 Azure 虛擬網路中子網路的部署。 其中包括：

- **前端**：前端為 App Service Environment (ASE) 中 HTTP/HTTPS 終止的位置。
- **背景工作**：這些背景工作是裝載應用程式的資源。
- **資料庫**：資料庫會保留定義環境的資訊。
- **儲存體**：儲存體可用來裝載客戶發佈的應用程式。

> [!NOTE]
> App Service Environment 有兩個版本：ASEv1 和 ASEv2。 在 ASEv1 中，您必須先管理資源，才可加以使用。 若要了解如何設定及管理 ASEv1，請參閱[設定 App Service Environment v1][ConfigureASEv1]。 本文的其餘部分著重於 ASEv2。
>
>

您可以使用適用於應用程式存取的外部 VIP 或內部 VIP，來部署 ASE (ASEv1 和 ASEv2)。 使用外部 VIP 的部署常稱為外部 ASE。 內部版本稱為 ILB ASE，因為它是使用內部負載平衡器 (ILB)。 若要深入了解 ILB ASE，請參閱[建立和使用 ILB ASE][MakeILBASE]。

## <a name="create-a-web-app-in-an-ase"></a>在 ASE 中建立 Web 應用程式 ##

若要在 ASE 中建立 Web 應用程式，可以使用和一般建立程序相同的程序，但有幾個小差異。 當您建立新的 App Service 方案時：

- 您可以挑選 ASE 作為位置，而不用挑選部署應用程式的地理位置。
- 在 ASE 中建立的所有 App Service 方案必須在「隔離」定價層中。

如果您沒有 ASE，則可以遵循[建立 App Service Environment][MakeExternalASE] 中的指示來建立一個 ASE。

若要在 ASE 中建立 Web 應用程式：

1. 選取 [建立資源]  >  [Web + 行動]  >  [Web 應用程式]。

1. 輸入 Web 應用程式的名稱。 如果您已在 ASE 中選取 App Service 方案，則應用程式的網域名稱會反映 ASE 的網域名稱。

    ![選取 Web 應用程式名稱][1]

1. 選取一個訂用帳戶。

1. 輸入新資源群組的名稱，或選取 [使用現有] 並從下拉式清單中挑選一個。

1. 選取您的作業系統。 

    * 在 ASE 中裝載 Linux 應用程式是新的預覽功能，因此建議您不要將 Linux 應用程式新增到目前執行生產工作負載的 ASE。 
    * 將 Linux 應用程式新增至 ASE，表示 ASE 也會處於預覽模式。 

1. 選取 ASE 中現有的 App Service 方案，或透過下列步驟建立一個新的方案：

    a. 選取 [新建]。

    b. 輸入 App Service 方案的名稱。

    c. 在 [位置] 下拉式清單中選取您的 ASE。 在 ASE 中裝載 Linux 應用程式目前只能在 6 個區域中進行：**美國西部、美國東部、西歐、北歐、澳洲東部以及東南亞。** 

    d. 選取 [隔離] 定價層。 選取 [選取] 。

    e. 選取 [確定] 。
    
    ![隔離定價層][2]

    > [!NOTE]
    > Linux Web 應用程式和 Windows Web 應用程式不能在相同的 App Service 方案中，但可位於相同的 App Service Environment 中。 
    >

1. 選取 [建立] 。

## <a name="how-scale-works"></a>調整方式 ##

每個 App Service 應用程式都會在 App Service 方案中執行。 容器模型是：保存 App Service 方案的環境，和保存應用程式的 App Service 方案。 當您調整應用程式時，您可調整 App Service 方案，因而調整相同方案中的所有應用程式。

在 ASEv2 中，當您調整 App Service 方案時，系統會自動新增所需的基礎結構。 新增基礎結構後，調整作業會有時間延遲。 在 ASEv1 中，您必須先在其中新增所需的基礎結構，才能建立或相應放大 App Service 方案。 

在多租用戶的 App Service 中，調整通常是立即性的，因為確實有可用的資源集區可用來支援它。 ASE 中沒有這類緩衝區，並在需要時配置資源。

在 ASE 中，您可以相應增加至 100 個執行個體。 這 100 個執行個體可以全都位於單一 App Service 方案中，或分散於多個 App Service 方案。

## <a name="ip-addresses"></a>IP 位址 ##

App Service 能夠將專用的 IP 位址配置給應用程式。 設定以 IP 為基礎的 SSL 之後，就可以使用這項功能，如下所述：[將現有的自訂 SSL 憑證繫結至 Azure Web Apps][ConfigureSSL]。 但是，在 ASE 中，有一個明顯的例外。 您無法新增額外的 IP 位址來供 ILB ASE 中以 IP 為基礎的 SSL 使用。

在 ASEv1 中，您必須先配置 IP 位址作為資源，才可以使用它們。 在 ASEv2 中，您可以如同在多租用戶的 App Service 中一樣，從您的應用程式使用它們。 ASEv2 中一律會有一個備用位址 (最多 30 個 IP 位址)。 每次您使用一個位址時，就會新增另一個位址，如此一律會有立即可供使用的位址。 配置另一個 IP 位址時，需有時間延遲，以避免快速連續新增 IP 位址。

## <a name="front-end-scaling"></a>前端調整大小 ##

在 ASEv2 中，當您相應放大 App Service 方案時，系統會自動新增背景工作來支援它們。 每個 ASE 建立時都會包含兩個前端。 此外，您的 App Service 方案中，每 15 個執行個體會有一個前端，因此前端會依據此規則自動相應放大。 例如，如果您有 15 個執行個體，則會有三個前端。 如果調整至 30 個執行個體，則您會有四個前端 ，依此類推。

這個前端數目應該足以適用於大部分情況。 但是，您也可用更快的速率相應放大。 您可以將前端與執行處理的比例變更至最低每五個執行個體有一個前端。 變更比例需付費。 如需詳細資訊，請參閱 [Azure App Service 價格][Pricing]。

前端資源是 ASE 的 HTTP/HTTPS 端點。 依照預設前端組態，每個前端的記憶體使用量始終大約為 60%。 客戶工作負載不會在前端上執行。 前端調整的關鍵因素是 CPU，其主要是由 HTTPS 流量所驅動。

## <a name="app-access"></a>應用程式存取 ##

在外部 ASE 中，您在建立應用程式時使用的網域，和多租用戶 App Service 的網域是不一樣的。 它包括了 ASE 的名稱。 如需有關如何建立外部 ASE 的詳細資訊，請參閱[建立 App Service Environment][MakeExternalASE]。 外部 ASE 中的網域名稱看起來像 *.&lt;asename&gt;.p.azurewebsites.net*。 例如，如果您的 ASE 名為 _external-ase_，而且您在該 ASE 中裝載名為 _contoso_ 的應用程式，則下列 URL 將可連至該應用程式：

- contoso.external-ase.p.azurewebsites.net
- contoso.scm.external-ase.p.azurewebsites.net

URL contoso.scm.external-ase.p.azurewebsites.net 用來存取 Kudu 主控台，或可利用 Web 部署發佈您的應用程式。 如需 Kudu 主控台的詳細資訊，請參閱 [Azure App Service 的 Kudu 主控台][Kudu]。 Kudu 主控台提供給您的 Web UI 可供偵錯、上傳檔案、編輯檔案及更多功能。

在 ILB ASE 中，您會在部署時決定網域。 如需有關如何建立 ILB ASE 的詳細資訊，請參閱[建立並使用 ILB ASE][MakeILBASE]。 如果您指定網域名稱 _ilb-ase.info_，則該 ASE 中的應用程式會在應用程式建立期間使用該網域。 對於名為 _contoso_ 的應用程式，則 URL 會是：

- contoso.ilb-ase.info
- contoso.scm.ilb-ase.info

## <a name="publishing"></a>發佈 ##

如同多租用戶 App Service，在 ASE 中，您可以透過下列各項發佈：

- Web 部署。
- FTP。
- 持續整合。
- 在 Kudu 主控台中拖放。
- IDE，例如 Visual Studio、Eclipse 或 IntelliJ IDEA。

使用外部 ASE 時，這些發行選項的行為相同。 如需詳細資訊，請參閱[在 Azure App Service 中部署][AppDeploy]。 

在發佈上的主要差異和 ILB ASE 有關。 使用 ILB ASE 時，所有發佈端點都只能透過 ILB 取得。 ILB 位於虛擬網路的 ASE 子網路中的私人 IP。 如果您沒有 ILB 的網路存取權，便無法在該 ASE 上發佈任何應用程式。 如[建立和使用 ILB ASE][MakeILBASE] 中所述，您需要在系統中設定應用程式的 DNS。 其中包含 SCM 端點。 如果未正確定義它們，就無法發佈。 您的 IDE 也需要有 ILB 的網路存取權，以便直接發佈到它。

以網際網路為基礎的 CI 系統 (例如 GitHub 和 Azure DevOps) 無法與 ILB ASE 搭配運作，因為發佈端點無法存取網際網路。 相反地，您需要使用會使用提取模型的 CI 系統，例如 Dropbox。

ILB ASE 中應用程式的發佈端點會使用用來建立 ILB ASE 的網域。 您可以在應用程式的發行設定檔，以及應用程式的入口網站刀鋒視窗中看到 (在 [概觀]  >  [基本資訊]，以及 [屬性] 中)。 

## <a name="pricing"></a>價格 ##

已建立名稱為**隔離**的價格 SKU，僅供與 ASEv2 搭配使用。 ASEv2 中裝載的所有 App Service 方案都位於「隔離」定價 SKU 中。 隔離的 App Service 方案費率會因區域而有所不同。 

除了 App Service 方案的價格，ASE 本身有固定費率。 固定費率不會隨著您的 ASE 大小而變動，而會給每 15 個 App Service 方案執行個體 1 個額外的前端，以此預設級別費率的 ASE 基礎結構來付費。  

如果每 15 個 App Service 方案執行個體 1 個前端的預設級別費率不夠快，您可以調整新增前端的比率或前端的大小。  當您調整比率或大小時，需要為預設時未加入的前端核心付費。  

例如，如果您將級別比率調整為 10，系統就會針對 App Service 方案中每 10 個執行個體新增一個前端。 固定費用涵蓋每隔 15 個執行個體 1 個前端的級別費率。 使用 10 級別比率時，您必須支付針對 10 個 App Service 方案執行個體新增的第三個前端費用。 在您達到 15 個執行個體時，不需要付費，因為它是自動新增的。

如果您將前端大小調整為 2 個核心，但不調整比率，則您需支付額外核心的費用。  ASE 已建立 2 個前端，所以即使低於自動調整閾值，如果大小增加到 2 個核心前端，您需支付 2 個額外核心的費用。

如需詳細資訊，請參閱 [Azure App Service 價格][Pricing]。

## <a name="delete-an-ase"></a>刪除 ASE ##

若要刪除 ASE： 

1. 請使用 [App Service Environment] 刀鋒視窗頂端的 [刪除]。 

1. 輸入 ASE 的名稱以確認您想要將它刪除。 當您刪除 ASE 時，將同時刪除其中包含的所有內容。 

    ![ASE 刪除][3]

<!--Image references-->
[1]: ./media/using_an_app_service_environment/usingase-appcreate.png
[2]: ./media/using_an_app_service_environment/usingase-pricingtiers.png
[3]: ./media/using_an_app_service_environment/usingase-delete.png


<!--Links-->
[Intro]: ./intro.md
[MakeExternalASE]: ./create-external-ase.md
[MakeASEfromTemplate]: ./create-from-template.md
[MakeILBASE]: ./create-ilb-ase.md
[ASENetwork]: ./network-info.md
[UsingASE]: ./using-an-ase.md
[UDRs]: ../../virtual-network/virtual-networks-udr-overview.md
[NSGs]: ../../virtual-network/security-overview.md
[ConfigureASEv1]: app-service-web-configure-an-app-service-environment.md
[ASEv1Intro]: app-service-app-service-environment-intro.md
[Functions]: ../../azure-functions/index.yml
[Pricing]: http://azure.microsoft.com/pricing/details/app-service/
[ARMOverview]: ../../azure-resource-manager/resource-group-overview.md
[ConfigureSSL]: ../web-sites-purchase-ssl-web-site.md
[Kudu]: http://azure.microsoft.com/resources/videos/super-secret-kudu-debug-console-for-azure-web-sites/
[AppDeploy]: ../app-service-deploy-local-git.md
[ASEWAF]: app-service-app-service-environment-web-application-firewall.md
[AppGW]: ../../application-gateway/application-gateway-web-application-firewall-overview.md
