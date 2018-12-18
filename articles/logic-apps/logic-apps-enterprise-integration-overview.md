---
title: B2B 企業整合概觀 - Azure Logic Apps | Microsoft Docs
description: 使用 Azure Logic Apps 與 Enterprise Integration Pack 來建置適用於企業整合解決方案的自動化 B2B 工作流程
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.assetid: dd517c4d-1701-4247-b83c-183c4d8d8aae
ms.date: 09/08/2016
ms.openlocfilehash: b2e2c81914e8c0440b358d59c7f0248db46b6c50
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43124287"
---
# <a name="overview-b2b-enterprise-integration-scenarios-in-azure-logic-apps-with-enterprise-integration-pack"></a>概觀：採用 Enterprise Integration Pack 的 Azure Logic Apps 中的 B2B 企業整合案例

對於企業對企業 (B2B) 工作流程以及使用 Azure Logic Apps 進行無接縫通訊，您可以搭配 Microsoft 的雲端解決方案「企業整合套件」啟用企業整合案例。 組織能以電子方式來交換訊息，即使他們使用不同的通訊協定與格式。 該套件會將不同的格式轉換為組織系統的可以解譯並處理的格式。 組織可以透過業界標準通訊協定 (包括 [AS2](../logic-apps/logic-apps-enterprise-integration-as2.md)、[X12](logic-apps-enterprise-integration-x12.md) 與 [EDIFACT](../logic-apps/logic-apps-enterprise-integration-edifact.md)) 來交換訊息。 您也可以使用加密與數位簽章來保護訊息的安全。

若您熟悉 BizTalk Server 或 Microsoft Azure BizTalk 服務，您會發現企業整合功能很容易使用，因為大部分的概念是類似的。 有一個主要差異是企業整合會使用整合帳戶，來簡化對於 B2B 通訊中所使用構件的儲存與管理。 

就架構而言，企業整合套件是以「整合帳戶」為基礎。 這些帳戶是雲端型容器，其中儲存您的所有構件，例如結構描述、合作夥伴、憑證、對應與合約。 您可以使用這些構件來設計、部署及維護您的 B2B 應用程式，以及針對邏輯應用程式建置 B2B 工作流程。 但是，在您可以使用這些構件之前，您必須先將您的整合帳戶連結到您的邏輯應用程式。 之後，您的邏輯應用程式就可以存取您整合帳戶的構件。

## <a name="why-should-you-use-enterprise-integration"></a>為什麼應該使用企業整合？

* 透過企業整合，您可以在一個位置 (亦即您的整合帳戶) 儲存您的所有構件。
* 您可以使用 Azure Logic Apps 與其所有連接器來建置 B2B 工作流程，並整合協力廠商軟體即服務 (SaaS) 應用程式、內部部署應用程式與自訂應用程式。
* 您可以使用 Azure 函式為您的邏輯應用程式建立自訂程式碼。

## <a name="how-to-get-started-with-enterprise-integration"></a>如何開始使用企業整合？

您可以透過 **Azure 入口網站**中的邏輯應用程式設計工具，使用企業整合套件來建置及管理 B2B 應用程式。 您也可以使用 [PowerShell](https://docs.microsoft.com/powershell/module/azurerm.logicapp "Logic apps PowerShell") 來管理邏輯應用程式。

以下是在 Azure 入口網站中建立應用程式之前必須執行的高階步驟︰

![概觀影像](media/logic-apps-enterprise-integration-overview/overview-0.png)  

## <a name="what-are-some-common-scenarios"></a>有哪些常見的案例？

企業整合支援下列產業標準：

* EDI - 電子資料交換
* EAI - 企業應用程式整合

## <a name="heres-what-you-need-to-get-started"></a>若要開始，您需要：

* 具有整合帳戶的 Azure 訂用帳戶
* 可建立對應和結構描述的 Visual Studio 2015
* [Visual Studio 2015 2.0 適用的 Microsoft Azure Logic Apps 企業整合工具](https://aka.ms/vsmapsandschemas)  

## <a name="try-it-now"></a>立刻嘗試

[部署可運作的 AS2 傳送與接收邏輯應用程式](https://github.com/Azure/azure-quickstart-templates/tree/master/201-logic-app-as2-send-receive)，這些應用程式使用 Azure Logic Apps 的 B2B 功能。

## <a name="learn-more"></a>深入了解
* [合約](../logic-apps/logic-apps-enterprise-integration-agreements.md "了解企業整合合約")
* [企業對企業 (B2B) 案例](../logic-apps/logic-apps-enterprise-integration-b2b.md "了解如何建立具有 B2B 功能的 Logic Apps")  
* [憑證](logic-apps-enterprise-integration-certificates.md "了解企業整合憑證")
* [一般檔案編碼/解碼](logic-apps-enterprise-integration-flatfile.md "了解如何將一般檔案內容編碼和解碼")  
* [整合帳戶](../logic-apps/logic-apps-enterprise-integration-accounts.md "了解整合帳戶")
* [對應](../logic-apps/logic-apps-enterprise-integration-maps.md "了解企業整合對應")
* [合作夥伴](logic-apps-enterprise-integration-partners.md "了解企業整合夥伴")
* [結構描述](logic-apps-enterprise-integration-schemas.md "了解企業整合結構描述")
* [XML 訊息驗證](logic-apps-enterprise-integration-xml.md "了解如何使用 Logic Apps 驗證 XML 訊息")
* [XML 轉換](logic-apps-enterprise-integration-transform.md "了解企業整合對應")
* [企業整合連接器](../connectors/apis-list.md "了解企業整合套件連接器")
* [整合帳戶中繼資料](../logic-apps/logic-apps-enterprise-integration-metadata.md "深入了解整合帳戶中繼資料")
* [監視 B2B 訊息](logic-apps-monitor-b2b-message.md "深入了解監視 B2B 訊息")
* [在 OMS 入口網站中追蹤 B2B 訊息](logic-apps-track-b2b-messages-omsportal.md "深入了解在 OMS 入口網站中追蹤 B2B 訊息")

