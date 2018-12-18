---
title: 將現有應用程式快速部署至 Azure Service Fabric 叢集
description: 使用 Azure Service Fabric 叢集來裝載現有的 Node.js 應用程式與 Visual Studio。
services: service-fabric
documentationcenter: nodejs
author: rwike77
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 12/06/2017
ms.author: ryanwi
ms.openlocfilehash: 2e7511f0efa36243a9e7e8e25f1697bdd3e60b63
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39005289"
---
# <a name="host-a-nodejs-application-on-azure-service-fabric"></a>在 Azure Service Fabric 上裝載 Node.js 應用程式

本快速入門可協助您將現有的應用程式 (在此範例中為 Node.js) 部署到在 Azure 上執行的 Service Fabric 叢集。

## <a name="prerequisites"></a>必要條件

開始之前，請確定您已 [設定開發環境](service-fabric-get-started.md)。 這包括安裝 Service Fabric SDK 及 Visual Studio 2017 或 2015。

您也需有現有的 Node.js 應用程式可供部署。 本快速入門使用可在[這裡][download-sample]下載的簡單 Node.js 網站。 在下一個步驟中建立專案之後，請將此檔案解壓縮至 `<path-to-project>\ApplicationPackageRoot\<package-name>\Code\` 資料夾。

如果您沒有 Azure 訂用帳戶，請建立[免費帳戶][create-account]。

## <a name="create-the-service"></a>建立服務

以**系統管理員**身分啟動 Visual Studio。

使用 `CTRL`+`SHIFT`+`N` 建立專案

在 [新增專案]  對話方塊中，選擇 [雲端] > [Service Fabric 應用程式]。

將應用程式命名為 **MyGuestApp**，然後按 [確定]。

>[!IMPORTANT]
>Node.js 可以輕易打破 Windows 具有之路徑的 260 字元限制。 對專案本身使用短路徑，例如 **c:\code\svc1**。 選擇性地，您可以依照**[這些指示](https://stackoverflow.com/a/41687101/1664231)** 在 Windows 10 中啟用完整檔案路徑。
   
![Visual Studio 中的新增專案對話方塊][new-project]

您可以從下一個對話方塊建立任何類型的 Service Fabric 服務。 在本快速入門中，選擇 [來賓可執行檔]。

將服務命名為 **MyGuestService** 並設定下列值右邊的選項：

| 設定                   | 值 |
| ------------------------- | ------ |
| 程式碼封裝資料夾       | _&lt;包含 Node.js 應用程式的資料夾&gt;_ |
| 程式碼封裝行為     | 將資料夾內容複製到專案 |
| 程式                   | node.exe |
| 引數                 | server.js |
| 工作資料夾            | CodePackage |

按 [確定]。

![Visual Studio 中的新增服務對話方塊][new-service]

Visual Studio 會建立應用程式專案和動作項目服務專案，並在 [方案總管] 中加以顯示。

應用程式專案 (**MyGuestApp**) 未直接包含任何程式碼。 它反而會參考一組服務專案。 此外，它包含三種其他類型的內容：

* **發佈設定檔**  
用來管理不同環境的工具喜好設定。

* **指令碼**  
包含用來部署/升級應用程式的 PowerShell 指令碼。

* **應用程式定義**  
包含 ApplicationPackageRoot 下的應用程式資訊清單。 相關聯的應用程式參數檔案位於 *ApplicationParameters*下方，它們可以定義應用程式，讓您得以特別為指定的環境設定應用程式。
    
如需服務專案的內容概觀，請參閱 [開始使用 Reliable Services](service-fabric-reliable-services-quick-start.md)。

## <a name="set-up-networking"></a>設定網路

我們所部署的範例 Node.js 應用程式會使用連接埠 **80**，而我們需要告知 Service Fabric 我們必須公開該連接埠。

開啟專案中的 **ServiceManifest.xml** 檔案。 在資訊清單的底部，有已定義一個項目的 `<Resources> \ <Endpoints>`。 修改該項目以新增 `Port`、`Protocol` 和 `Type`。 

```xml
  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
      <Endpoint Name="MyGuestAppServiceTypeEndpoint" Port="80" Protocol="http" Type="Input" />
    </Endpoints>
  </Resources>
```

## <a name="deploy-to-azure"></a>部署至 Azure

如果您按下 **F5** 並執行專案，它就會部署到本機叢集。 不過，讓我們改為將它部署到 Azure。

以滑鼠右鍵按一下專案，然後選擇 [發佈...] 來開啟對話方塊，以發佈至 Azure。

![發佈至 Service Fabric 服務的 Azure 對話方塊][publish]

選取 **PublishProfiles\Cloud.xml** 目標設定檔。

如果您先前尚未這樣做，請選擇要部署至的 Azure 帳戶。 如果您沒有 Azure 帳戶，請[註冊一個][create-account]。

在 [連線端點] 之下，選取要部署 Service Fabric 叢集的位置。 如果您沒有 Service Fabric 叢集，請選取 &lt;建立新的叢集...&gt;，這會在網頁瀏覽器視窗中開啟 Azure 入口網站。 如需詳細資訊，請參閱[在入口網站中建立叢集](service-fabric-cluster-creation-via-portal.md#create-cluster-in-the-azure-portal)。 

當您建立 Service Fabric 叢集時，請務必將 [自訂端點] 設定設為 **80**。

![自訂端點的 Service fabric 節點類型組態][custom-endpoint]

建立新的 Service Fabric 叢集需要一些時間才能完成。 建立之後，請回到 [發佈] 對話方塊並選取 &lt;重新整理&gt;。 新的叢集會列在下拉式方塊中，請加以選取。

按 [發佈] 並等待部署完成。

這可能需要幾分鐘的時間。 完成之後，應用程式可能需要數分鐘的時間才能完全可用。

## <a name="test-the-website"></a>測試網站

在服務發佈之後，請在網頁瀏覽器中進行測試。 

首先，開啟 Azure 入口網站並尋找您的 Service Fabric 服務。

請檢查服務位址的 [概觀] 刀鋒視窗。 使用 [用戶端連線端點] 屬性中的網域名稱。 例如： `http://mysvcfab1.westus2.cloudapp.azure.com`。

![Azure 入口網站上的 Service Fabric 概觀刀鋒視窗][overview]

瀏覽至此位址，您將會在此看到 `HELLO WORLD` 回應。

## <a name="delete-the-cluster"></a>刪除叢集

請不要忘記刪除您在本快速入門中建立的所有資源，因為您需對那些資源付費。

## <a name="next-steps"></a>後續步驟
深入了解[客體可執行檔](service-fabric-guest-executables-introduction.md)。

<!-- Image References -->

[new-project]: ./media/quickstart-guest-app/new-project.png
[new-service]: ./media/quickstart-guest-app/template.png
[solution-exp]: ./media/quickstart-guest-app/solution-explorer.png
[publish]: ./media/quickstart-guest-app/publish.png
[overview]: ./media/quickstart-guest-app/overview.png
[custom-endpoint]: ./media/quickstart-guest-app/custom-endpoint.png

[download-sample]: https://github.com/MicrosoftDocs/azure-cloud-services-files/raw/temp/service-fabric-node-website.zip
[create-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
