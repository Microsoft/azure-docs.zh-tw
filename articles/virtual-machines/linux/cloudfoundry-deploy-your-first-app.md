---
title: 將第一個應用程式部署到 Microsoft Azure 上的 Cloud Foundry | Microsoft Docs
description: 將應用程式部署到 Azure 上的 Cloud Foundry
services: virtual-machines-linux
documentationcenter: ''
author: seanmck
manager: jeconnoc
editor: ''
tags: ''
keywords: ''
ms.assetid: 8fa04a58-56ad-4e6c-bef4-d02c80d4b60f
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 06/14/2017
ms.author: seanmck
ms.openlocfilehash: 6e2fa77273ef35fae6c3b232cb36fa913faf879d
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44299044"
---
# <a name="deploy-your-first-app-to-cloud-foundry-on-microsoft-azure"></a>將第一個應用程式部署到 Microsoft Azure 上的 Cloud Foundry

[Cloud Foundry](http://cloudfoundry.org) 是 Microsoft Azure 上可用的熱門開放原始碼應用程式平台。 在本文中，我們會示範如何在 Azure 環境中部署和管理 Cloud Foundry 上的應用程式。

## <a name="create-a-cloud-foundry-environment"></a>建立 Cloud Foundry 環境

有幾個選項可以用來在 Azure 上建立 Cloud Foundry 環境：

- 使用 Azure Marketplace 中的 [Pivotal Cloud Foundry 供應項目][pcf-azuremarketplace]以建立標準環境，其中包含 PCF OPS Manager 和 Azure Service Broker。 您可以在 Pivotal 文件中找到部署市集供應項目的[完整指示][pcf-azuremarketplace-pivotaldocs]。
- 建立自訂的環境，方法是[手動部署 Pivotal Cloud Foundry][pcf-custom]。
- [直接部署開放原始碼 Cloud Foundry 套件][oss-cf-bosh]，方法是設定 [BOSH](http://bosh.io) 導向器，這是一個 VM，它會協調 Cloud Foundry 環境的部署。

> [!IMPORTANT] 
> 如果您正在從 Azure Marketplace 部署 PCF，請記下存取 Pivotal Apps Manager 所需的 SYSTEMDOMAINURL 和管理員認證，這兩個項目都在市集部署指南中提及。 需要這兩個項目才能完成本教學課程。 對於市集部署，SYSTEMDOMAINURL 的形式為 https://system.*ip-address*.cf.pcfazure.com。

## <a name="connect-to-the-cloud-controller"></a>連線至 Cloud Controller

Cloud Controller 是到 Cloud Foundry 環境以部署和管理應用程式的主要進入點。 核心 Cloud Controller API (CCAPI) 是 REST API，但是可以透過各種工具存取。 在此情況下，我們透過 [Cloud Foundry CLI][cf-cli] 與它互動。 您可以在 Linux、MacOS 或 Windows 上安裝 CLI，但是如果您完全不想要安裝，它可以預先安裝在 [Azure Cloud Shell][cloudshell-docs]。

若要登入，請在您從市集部署取得的 SYSTEMDOMAINURL 前面加上 `api`。 由於預設部署使用自我簽署憑證，您也應該包含 `skip-ssl-validation` 參數。

```bash
cf login -a https://api.SYSTEMDOMAINURL --skip-ssl-validation
```

系統會提示您登入 Cloud Controller。 使用您從市集部署步驟取得的管理員帳戶認證。

Cloud Foundry 提供組織和空間 作為命名空間，以隔離共用部署內的小組和環境。 PCF 市集部署包含預設系統組織和建立以包含基礎元件的一組空間，例如自動調整服務和 Azure Service Broker。 目前，選擇系統空間。


## <a name="create-an-org-and-space"></a>建立組織和空間

如果您輸入 `cf apps`，您會看到一組系統應用程式，已部署在系統組織內的系統空間。 

您應該為系統應用程式保留系統組織，因此請建立組織和空間以容納我們的範例應用程式。

```bash
cf create-org myorg
cf create-space dev -o myorg
```

使用目標命令以切換至新的組織和空間：

```bash
cf target -o testorg -s dev
```

現在，當您部署應用程式時，會自動在新的組織和空間中建立。 若要確認目前在新的組織/空間中沒有應用程式，請再次輸入 `cf apps`。

> [!NOTE] 
> 如需組織和空間以及它們如何用於角色型存取控制 (RBAC) 的詳細資訊，請參閱[Cloud Foundry 文件][cf-orgs-spaces-docs]。

## <a name="deploy-an-application"></a>部署應用程式

讓我們使用名為 Hello Spring Cloud 的範例 Cloud Foundry 應用程式，該應用程式是以 Java 撰寫，並且根據 [Spring Framework](http://spring.io) 和 [Spring Boot](http://projects.spring.io/spring-boot/)。

### <a name="clone-the-hello-spring-cloud-repository"></a>複製 Hello Spring Cloud 存放庫

Hello Spring Cloud 範例應用程式可以在 GitHub 上取得。 將它複製到您的環境，並且變更到新的目錄：

```bash
git clone https://github.com/cloudfoundry-samples/hello-spring-cloud
cd hello-spring-cloud
```

### <a name="build-the-application"></a>建置應用程式

使用 [Apache Maven](http://maven.apache.org) 建置應用程式。

```bash
mvn clean package
```

### <a name="deploy-the-application-with-cf-push"></a>使用 cf push 部署應用程式

您可以使用 `push` 命令，將大部分的應用程式部署到 Cloud Foundry：

```bash
cf push
```

當您發送應用程式時，Cloud Foundry 會偵測應用程式類型 (在此案例中為 Java 應用程式)，並且識別其相依性 (在此案例中為 Spring Framework)。 然後它會將執行程式碼所需的所有項目封裝至獨立容器映像，稱為 droplet。 最後，Cloud Foundry 會在環境中其中一部可用的機器上排程應用程式，並且建立您可以在該位置取得它的 URL，該 URL 可以在命令的輸出中取得。

![cf push 命令的輸出][cf-push-output]

若要查看 hello-spring-cloud 應用程式，請在瀏覽器中開啟提供的 URL：

![Hello Spring Cloud 的預設 UI][hello-spring-cloud-basic]

> [!NOTE] 
> 若要深入了解 `cf push` 期間會發生什麼狀況，請參閱 Cloud Foundry 文件中的[應用程式如何暫存][cf-push-docs]。

## <a name="view-application-logs"></a>檢視應用程式記錄

您可以使用 Cloud Foundry CLI 以根據應用程式的名稱檢視其記錄：

```bash
cf logs hello-spring-cloud
```

根據預設，記錄命令會使用 tail，在寫入之後顯示新的記錄。 若要查看出現的新記錄，請在瀏覽器中重新整理 hello-spring-cloud 應用程式。

若要檢視已寫入的記錄，請新增 `recent` 參數：

```bash
cf logs --recent hello-spring-cloud
```

## <a name="scale-the-application"></a>調整應用程式

根據預設，`cf push` 只會建立應用程式的單一執行個體。 若要確保高可用性，並且啟用相應放大以獲得較高的輸送量，您通常要執行應用程式的多個執行個體。 您可以使用 `scale` 命令，輕鬆地相應放大已部署的應用程式：

```bash
cf scale -i 2 hello-spring-cloud
```

在應用程式上執行 `cf app` 命令，會顯示 Cloud Foundry 正在建立應用程式的另一個執行個體。 應用程式啟動之後，Cloud Foundry 會自動對它啟動負載平衡流量。


## <a name="next-steps"></a>後續步驟

- [閱讀 Cloud Foundry 文件][cloudfoundry-docs]
- [設定適用於 Cloud Foundry 的 Azure DevOps Services 外掛程式][vsts-plugin]
- [設定適用於 Cloud Foundry 的 Microsoft Log Analytics Nozzle][loganalytics-nozzle]

<!-- LINKS -->

[pcf-azuremarketplace]: https://azuremarketplace.microsoft.com/marketplace/apps/pivotal.pivotal-cloud-foundry
[pcf-custom]: https://docs.pivotal.io/pivotalcf/1-10/customizing/azure.html
[oss-cf-bosh]: https://github.com/cloudfoundry-incubator/bosh-azure-cpi-release/tree/master/docs
[pcf-azuremarketplace-pivotaldocs]: https://docs.pivotal.io/pivotalcf/customizing/pcf_azure.html
[cf-cli]: https://github.com/cloudfoundry/cli
[cloudshell-docs]: https://docs.microsoft.com/azure/cloud-shell/overview
[cf-orgs-spaces-docs]: https://docs.cloudfoundry.org/concepts/roles.html
[spring-boot]: https://projects.spring.io/spring-boot/
[spring-framework]: http://spring.io
[cf-push-docs]: https://docs.cloudfoundry.org/concepts/how-applications-are-staged.html
[cloudfoundry-docs]: https://docs.cloudfoundry.org
[vsts-plugin]: https://github.com/Microsoft/vsts-cloudfoundry
[loganalytics-nozzle]: https://github.com/Azure/oms-log-analytics-firehose-nozzle

<!-- IMAGES -->
[cf-push-output]: ./media/cloudfoundry-deploy-your-first-app/cf-push-output.png
[hello-spring-cloud-basic]: ./media/cloudfoundry-deploy-your-first-app/hello-spring-cloud-basic.png
