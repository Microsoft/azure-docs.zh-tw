---
title: 建立外部 Azure App Service 環境
description: 說明如何在建立應用程式或獨立時建立 App Service 環境
services: app-service
documentationcenter: na
author: ccompy
manager: stefsch
ms.assetid: 94dd0222-b960-469c-85da-7fcb98654241
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/13/2017
ms.author: ccompy
ms.openlocfilehash: 0ee2654b313f453f7485c89c789edc8efc068d7b
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39442644"
---
# <a name="create-an-external-app-service-environment"></a>建立外部 App Service 環境 #

Azure App Service Environment (ASE) 是將 Azure App Service 部署到客戶 Azure 虛擬網路 (VNet) 中子網路的一種部署。 部署 App Service Environment (ASE) 有二種方法：

- 使用外部 IP 位址上的 VIP，通常稱為「外部 ASE」。
- 使用內部 IP 位址上的 VIP，通常稱為 ILB ASE，因為內部端點是內部負載平衡器 (ILB)。

本文說明如何建立外部 ASE。 如需 ASE 的概述，請參閱 [App Service Environment 簡介][Intro]。 如需有關如何建立 ILB ASE 的資訊，請參閱[建立並使用 ILB ASE][MakeILBASE]。

## <a name="before-you-create-your-ase"></a>建立 ASE 之前 ##

建立 ASE 之後，您就無法變更下列項目：

- 位置
- 訂用帳戶
- 資源群組
- 使用的 VNet
- 使用的子網路
- 子網路大小

> [!NOTE]
> 選取 VNet 並指定子網路時，請確定它足夠大以容納未來的成長及規模調整需求。 我們建議使用包含 256 個位址的 `/24` 大小。
>

## <a name="three-ways-to-create-an-ase"></a>建立 ASE 有三種方式 ##

建立 ASE 有三種方式：

- **建立 App Service 方案時**。 這個方法會在一個步驟中建立 ASE 和應用程式服務方案。
- **作為獨立動作**。 這個方法會建立獨立 ASE，也就是任何項目的 ASE。 這個方法是建立 ASE 的更進階流程。 您可以使用它來建立 ASE 並搭配 ILB。
- **從 Azure Resource Manager 範本**。 這個方法適用於進階使用者。 如需詳細資訊，請參閱[從範本建立 ASE][MakeASEfromTemplate]。

外部 ASE 具有公用 VIP，這表示 ASE 中所有對應用程式的 HTTP/HTTPS 流量都會叫用可存取網際網路的 IP 位址。 搭配 ILB 的 ASE 中有 ASE 所使用之子網路的 IP 位址。 裝載於 ILB ASE 中的應用程式並非直接向網際網路公開。

## <a name="create-an-ase-and-an-app-service-plan-together"></a>一起建立 ASE 和 App Service 方案 ##

App Service 方案是應用程式的容器。 當您在 App Service 中建立應用程式時，要選擇或建立 App Service 方案。 App Service Environment 可保存 App Service 方案，而 App Service 方案可保存應用程式。

若要在建立 App Service 方案時建立 ASE：

1. 在 [Azure 入口網站](https://portal.azure.com/)中，選取 [建立資源]  >  [Web + 行動]  > [Web 應用程式]。

    ![建立 Web 應用程式][1]

1. 選取您的訂用帳戶。 會在相同的訂用帳戶中建立應用程式和 ASE。

1. 選取或建立資源群組。 您可以使用資源群組來管理相關的一組 Azure 資源。 當您為應用程式建立角色型存取控制規則時，資源群組也十分實用。 如需詳細資訊，請參閱 [Azure Resource Manager 概觀][ARMOverview]。

1. 選取您的 OS (Windows、Linux 或 Docker)。 

1. 選取 App Service 方案，然後選取 [新建]。 Linux Web 應用程式和 Windows Web 應用程式不能在相同的 App Service 方案中，但可位於相同的 App Service Environment 中。 

    ![新增 App Service 方案][2]

1. 在 [位置] 下拉式清單中，選取您需要建立 ASE 的區域。 如果您選取現有的 ASE，就不會建立新的 ASE。 會在您選取的 ASE 中建立 App Service 方案。 

1. 選取**定價層**，然後選擇其中一個**隔離的**定價 SKU。 如果您選擇**隔離** SKU 卡以及非 ASE 的位置，就會在該位置中建立新的 ASE。 若要啟動建立 ASE 的流程，請選取 [選取]。 **隔離** SKU 僅供與 ASE 搭配使用。 您也無法在 ASE 中使用**隔離**以外的其他任何定價 SKU。 

    ![定價層選取項目][3]

1. 輸入 ASE 的名稱。 此名稱是用於應用程式的可定址名稱。 如果 ASE 的名稱是 _appsvcenvdemo_，則網域名稱會是 .appsvcenvdemo.p.azurewebsites.net。 如果您建立名為 mytestapp 的應用程式，則可定址於 mytestapp.appsvcenvdemo.p.azurewebsites.net。 您無法在名稱中使用空白字元。 如果您使用大寫字元，則網域名稱會是該名稱的全小寫版本。

    ![新增 App Service 方案名稱][4]

1. 指定 Azure 虛擬網路詳細資料。 選取 [新建] 或 [選取現有]。 僅在選取的區域擁有 VNet 時，才可以使用選取現有 VNet 的選項。 如果您選取 [新建]，請輸入 VNet 的名稱。 會建立具有該名稱的 Resource Manager VNet。 它會使用選取區域中的位址空間 `192.168.250.0/23`。 如果您選取 [選取現有]，您需要：

    a. 如果您有多個位址區塊，請選取 VNet 位址區塊。

    b. 輸入新的子網路名稱。

    c. 選取子網路的大小。 請記得選取大小足以容納未來成長的 ASE。 建議是 `/25`，具有 128 個位址，而且可以處理最大大小的 ASE。 例如，不建議 `/28`，因為只有 16 個位址可供使用。 基礎結構會使用至少 7 個位址，而 Azure 網路會使用另外 5 個。 在 `/28` 子網路中，外部 ASE 只有最多 4 個 App Service 方案執行個體的規模，而 ILB ASE 只有 3 個 App Service 方案執行個體。

    d. 選取子網路 IP 範圍。

1. 選取 [建立] 以建立 ASE。 此流程也會建立 App Service 方案和應用程式。 ASE、App Service 方案和應用程式會在相同的訂用帳戶底下，同時在相同的資源群組中。 如果您的 ASE 需要個別資源群組，或如果您需要 ILB ASE，請遵循步驟讓 ASE 自行建立。

## <a name="create-an-ase-and-a-linux-web-app-using-a-custom-docker-image-together"></a>使用自訂 Docker 映像一起建立 ASE 和 Linux Web 應用程式

1. 在 [Azure 入口網站](https://portal.azure.com/)中，按一下 [建立資源] > [Web + 行動] > [適用於容器的 Web 應用程式]。 

    ![建立 Web 應用程式][7]

1. 選取您的訂用帳戶。 會在相同的訂用帳戶中建立應用程式和 ASE。

1. 選取或建立資源群組。 您可以使用資源群組來管理相關的一組 Azure 資源。 當您為應用程式建立角色型存取控制規則時，資源群組也十分實用。 如需詳細資訊，請參閱 [Azure Resource Manager 概觀][ARMOverview]。

1. 選取 App Service 方案，然後選取 [新建]。 Linux Web 應用程式和 Windows Web 應用程式不能在相同的 App Service 方案中，但可位於相同的 App Service Environment 中。 

    ![新增 App Service 方案][8]

1. 在 [位置] 下拉式清單中，選取您需要建立 ASE 的區域。 如果您選取現有的 ASE，就不會建立新的 ASE。 會在您選取的 ASE 中建立 App Service 方案。 

1. 選取**定價層**，然後選擇其中一個**隔離的**定價 SKU。 如果您選擇**隔離** SKU 卡以及非 ASE 的位置，就會在該位置中建立新的 ASE。 若要啟動建立 ASE 的流程，請選取 [選取]。 **隔離** SKU 僅供與 ASE 搭配使用。 您也無法在 ASE 中使用**隔離**以外的其他任何定價 SKU。 

    ![定價層選取項目][3]

1. 輸入 ASE 的名稱。 此名稱是用於應用程式的可定址名稱。 如果 ASE 的名稱是 _appsvcenvdemo_，則網域名稱會是 .appsvcenvdemo.p.azurewebsites.net。 如果您建立名為 mytestapp 的應用程式，則可定址於 mytestapp.appsvcenvdemo.p.azurewebsites.net。 您無法在名稱中使用空白字元。 如果您使用大寫字元，則網域名稱會是該名稱的全小寫版本。

    ![新增 App Service 方案名稱][4]

1. 指定 Azure 虛擬網路詳細資料。 選取 [新建] 或 [選取現有]。 僅在選取的區域擁有 VNet 時，才可以使用選取現有 VNet 的選項。 如果您選取 [新建]，請輸入 VNet 的名稱。 會建立具有該名稱的 Resource Manager VNet。 它會使用選取區域中的位址空間 `192.168.250.0/23`。 如果您選取 [選取現有]，您需要：

    a. 如果您有多個位址區塊，請選取 VNet 位址區塊。

    b. 輸入新的子網路名稱。

    c. 選取子網路的大小。 請記得選取大小足以容納未來成長的 ASE。 建議是 `/25`，具有 128 個位址，而且可以處理最大大小的 ASE。 例如，不建議 `/28`，因為只有 16 個位址可供使用。 基礎結構會使用至少 7 個位址，而 Azure 網路會使用另外 5 個。 在 `/28` 子網路中，外部 ASE 只有最多 4 個 App Service 方案執行個體的規模，而 ILB ASE 只有 3 個 App Service 方案執行個體。

    d. 選取子網路 IP 範圍。

1.  選取 [設定容器]。
    * 輸入您的自訂映像名稱 (您可以使用 Azure Container Registry、Docker Hub 和自己的私人登錄)。 如果您不想使用自己的自訂容器，您可以依照上述指示，只要自備程式碼並且在 Linux 上使用內建映像搭配 App Service。 

    ![設定容器][9]

1. 選取 [建立] 以建立 ASE。 此流程也會建立 App Service 方案和應用程式。 ASE、App Service 方案和應用程式會在相同的訂用帳戶底下，同時在相同的資源群組中。 如果您的 ASE 需要個別資源群組，或如果您需要 ILB ASE，請遵循步驟讓 ASE 自行建立。


## <a name="create-an-ase-by-itself"></a>由 ASE 本身建立 ##

如果您建立 ASE 獨立，它就不會包含任何項目。 空白 ASE 仍會產生基礎結構的每月費用。 請遵循下列步驟來建立具有 ILB 的 ASE，或是在它自己的資源群組中建立 ASE。 建立 ASE 之後，您可以使用一般流程在其中建立應用程式。 選取您新的 ASE 作為位置。

1. 在 Azure Marketplace 中搜尋「App Service 環境」，或選取 [建立資源]  >  [Web + 行動]  >  [App Service 環境]。 

1. 輸入您的 ASE 名稱。 這個名稱會用於 ASE 中建立的應用程式。 如果名稱是 mynewdemoase，子網域名稱就是 .mynewdemoase.p.azurewebsites.net。 如果您建立名為 mytestapp 的應用程式，則可定址於 mytestapp.mynewdemoase.p.azurewebsites.net。 您無法在名稱中使用空白字元。 如果您使用大寫字元，則網域名稱會是該名稱的全小寫版本。 如果您是使用 ILB，ASE 名稱就不會用於您的子網域中，但是會在 ASE 建立期間明確指定。

    ![ASE 命名][5]

1. 選取您的訂用帳戶。 此訂用帳戶也是所有應用程式在 ASE 中所使用的。 您無法將 ASE 放在另一個訂用帳戶中的 VNet。

1. 選取或指定新的資源群組。 用於 ASE 的資源群組必須是與用於您 VNet 的相同。 如果您選取現有的 VNet，您 ASE 的資源群組選取項目將會更新，以反映 VNet 的資源群組。 如果您是使用 Resource Manager 範本，可以使用不同於 VNet 資源群組的資源群組來建立 ASE。 若要從範本建立 ASE，請參閱[從範本建立 App Service 環境][MakeASEfromTemplate]。

    ![資源群組選取項目][6]

1. 選取您的 VNet 和位置。 您可以建立新的 VNet 或選取現有的 VNet： 

    * 如果您選取新的 VNet，就可以指定名稱和位置。 
    
    * 新的 VNet 會有位址範圍 192.168.250.0/23，和名為 default 的子網路。 子網路定義為 192.168.250.0/24。 您只能選取 Resource Manager VNet。 [VIP 類型] 選取項目會決定您的 ASE 是否可以從網際網路 (外部) 直接存取，或者它是使用 ILB。 若要深入了解這些選項，請參閱[在 App Service 環境中建立及使用內部負載平衡器][MakeILBASE]。 

      * 如果您針對 VIP 類型選取 [外部]，則可以選取系統針對以 IP 為主的 SSL 用途會建立幾個外部 IP 位址。 
    
      * 如果您針對 VIP 類型選取 [外部]，就必須指定您 ASE 使用的網域。 您可以將 ASE 部署到使用公用或私人位址範圍的 VNet。 若要搭配使用 VNet 與公用位址範圍，您必須事先建立 VNet。 
    
    * 如果您選取現有的 VNet，ASE 建立時就會建立新的子網路。 *您無法在入口網站中使用預先建立的子網路。如果您是使用 Resource Manager 範本，則可以使用現有的子網路建立 ASE。* 若要從範本建立 ASE，請參閱[從範本建立 App Service 環境][MakeASEfromTemplate]。

## <a name="app-service-environment-v1"></a>App Service 環境 v1 ##

您仍然可以建立 App Service Environment (ASEv1) 的第一個版本執行個體。 若要啟動該流程，請在 Marketplace 搜尋 **App Service Environment v1**。 您要使用與建立獨立 ASE 的相同方式來建立 ASE。 完成後，您的 ASEv1 會有兩個「前端」和兩個「背景工作角色」。 使用 ASEv1 時，您必須管理「前端」和「背景工作角色」。 當您建立 App Service 方案時，它們不會自動新增。 前端可作為 HTTP/HTTPS 端點，將流量傳送到背景工作角色。 背景工作角色是裝載應用程式的角色。 建立 ASE 之後，您就可以調整「前端」和「背景工作角色」的數量。 

若要深入了解 ASEv1，請參閱 [App Service Environment v1 簡介][ASEv1Intro]。 如需更多關於調整、管理及監視 ASEv1 的資訊，請參閱[如何設定 App Service Environment][ConfigureASEv1]。

<!--Image references-->
[1]: ./media/how_to_create_an_external_app_service_environment/createexternalase-create.png
[2]: ./media/how_to_create_an_external_app_service_environment/createexternalase-aspcreate.png
[3]: ./media/how_to_create_an_external_app_service_environment/createexternalase-pricing.png
[4]: ./media/how_to_create_an_external_app_service_environment/createexternalase-embeddedcreate.png
[5]: ./media/how_to_create_an_external_app_service_environment/createexternalase-standalonecreate.png
[6]: ./media/how_to_create_an_external_app_service_environment/createexternalase-network.png
[7]: ./media/how_to_create_an_external_app_service_environment/createexternalase-createwafc.png
[8]: ./media/how_to_create_an_external_app_service_environment/createexternalase-aspcreatewafc.png
[9]: ./media/how_to_create_an_external_app_service_environment/createexternalase-configurecontainer.png



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
[webapps]: ../app-service-web-overview.md
[mobileapps]: ../../app-service-mobile/app-service-mobile-value-prop.md
[Functions]: ../../azure-functions/index.yml
[Pricing]: http://azure.microsoft.com/pricing/details/app-service/
[ARMOverview]: ../../azure-resource-manager/resource-group-overview.md
