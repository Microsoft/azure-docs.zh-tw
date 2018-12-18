---
title: 在 Microsoft Azure 上開始使用 Cloud Foundry | Microsoft Docs
description: 在 Microsoft Azure 上執行 OSS 或 Pivotal Cloud Foundry
services: virtual-machines-linux
documentationcenter: ''
author: seanmck
manager: jeconnoc
editor: ''
tags: ''
keywords: ''
ms.assetid: 2a15ffbf-9f86-41e4-b75b-eb44c1a2a7ab
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 01/19/2017
ms.author: seanmck
ms.openlocfilehash: 18aa5ab16a1690fdcead46ebc15d72837f62612d
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44302464"
---
# <a name="cloud-foundry-on-azure"></a>Azure 上的 Cloud Foundry

Cloud Foundry 是開放原始碼的平台即服務 (PaaS)，可用於建置、部署和操作以各種不同語言和架構開發的 12-factor 應用程式。 本文件說明您可用來在 Azure 上執行 Cloud Foundry 的選項，以及應如何開始使用。

## <a name="cloud-foundry-offerings"></a>Cloud Foundry 供應項目

有兩種形式的 Cloud Foundry 可在 Azure 上執行：開放原始碼的 Cloud Foundry (OSS CF) 和 Pivotal Cloud Foundry (PCF)。 OSS CF 是完全[開放原始碼](https://github.com/cloudfoundry)的 Cloud Foundry 版本，可透過 Cloud Foundry Foundation 來管理。 Pivotal Cloud Foundry 是由 Pivotal Software Inc. 所提供的 Cloud Foundry 企業散發版本。我們將查看這兩個供應項目之間的差異。

### <a name="open-source-cloud-foundry"></a>開放原始碼的 Cloud Foundry

您可以在 Azure 上部署 OSS Cloud Foundry，方法是使用 [GitHub 上提供的指示](https://github.com/cloudfoundry-incubator/bosh-azure-cpi-release/blob/master/docs/guidance.md)，先部署 BOSH 導向器，然後再部署 Cloud Foundry。 若要深入了解如何使用 OSS CF，請參閱 Cloud Foundry Foundation 所提供的[文件](https://docs.cloudfoundry.org/)。

Microsoft 透過下列社群管道提供 OSS CF 的最佳支援：

- #<a name="bosh-azure-cpi-channel-on-cloud-foundry-slackhttpsslackcloudfoundryorg"></a>[Cloud Foundry Slack (英文)](https://slack.cloudfoundry.org/) 上的 bosh-azure-cpi 管道
- [cf-bosh 郵寄清單 (英文)](https://lists.cloudfoundry.org/pipermail/cf-bosh)
- GitHub 上關於 [CPI (英文)](https://github.com/cloudfoundry-incubator/bosh-azure-cpi-release/issues) 和 [Service Broker (英文)](https://github.com/Azure/meta-azure-service-broker/issues) 的問題

>[!NOTE]
> 對於您 Azure 資源 (例如，您執行 Cloud Foundry 的虛擬機器) 的支援層級是以您的 Azure 支援合約為根據。 最佳的社群支援僅適用於 Cloud Foundry 特有的元件。

### <a name="pivotal-cloud-foundry"></a>Pivotal Cloud Foundry

Pivotal Cloud Foundry 包含與 OSS 散發版本相同的核心平台，以及一組專屬的管理工具和企業支援。 若要在 Azure 上執行 PCF，您必須取得 Pivotal 的授權。 Azure Marketplace 提供的 PCF 供應項目包括 90 天試用版授權。

這些工具包括 [Pivotal Operations Manager (英文)](http://docs.pivotal.io/pivotalcf/customizing/)、可簡化部署和管理 Cloud Foundry Foundation 的 Web 應用程式，以及 [Pivotal Apps Manager (英文)](https://docs.pivotal.io/pivotalcf/console/)，此為可用來管理使用者和應用程式的 Web 應用程式。

除了以上針對 OSS CF 列出的支援管道，PCF 授權還會賦與您連絡 Pivotal 以取得支援的權利。 Microsoft 和 Pivotal 也已經啟用支援工作流程，可讓您連絡其中一方以取得協助，並讓您根據問題的所在位置適當地路由傳送查詢。

## <a name="azure-service-broker"></a>Azure Service Broker

Cloud Foundry 鼓勵使用 ["twelve-factor app" (英文)](https://12factor.net/) 方法，明確區分無狀態應用程式程序和可設定狀態的備份服務。 [Service Broker (英文)](https://docs.cloudfoundry.org/services/api.html) 提供一致的方式來佈建備份服務並繫結至應用程式。 [Azure Service Broker (英文)](https://github.com/Azure/meta-azure-service-broker) 會透過此管道提供一些主要的 Azure 服務，包括 Azure 儲存體和 Azure SQL。

如果您使用 Pivotal Cloud Foundry，您也可以從 Pivotal Network，[以圖格形式取得](https://docs.pivotal.io/azure-sb/installing.html)此 Service Broker。

## <a name="related-resources"></a>相關資源

### <a name="azure-devops-services-plugin"></a>Azure DevOps Services 外掛程式

Cloud Foundry 非常適合敏捷式軟體開發，包括使用持續整合 (CI) 和持續傳遞 (CD)。 如果您使用 Azure DevOps Services 來管理專案，並且想要設定以 Cloud Foundry 為目標的 CI/CD 管線，您可以使用 [Azure DevOps Services Cloud Foundry 組建擴充功能](https://marketplace.visualstudio.com/items?itemName=ms-vsts.cloud-foundry-build-extension)。 此外掛程式讓您能夠簡單地設定和自動化部署至 Cloud Foundry，而不論是在 Azure 或另一個環境中執行。

## <a name="next-steps"></a>後續步驟

- [從 Azure Marketplace 部署 Pivotal Cloud Foundry](https://azure.microsoft.com/marketplace/partners/pivotal/pivotal-cloud-foundryazure-pcf/)
- [在 Azure 將應用程式部署至 Cloud Foundry](./cloudfoundry-deploy-your-first-app.md)
