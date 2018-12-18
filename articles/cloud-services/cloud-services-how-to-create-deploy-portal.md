---
title: 如何建立和部署雲端服務 | Microsoft Docs
description: 了解如何使用 Azure 入口網站建立和部署雲端服務。
services: cloud-services
documentationcenter: ''
author: jpconnock
manager: timlt
editor: ''
ms.assetid: 56ea2f14-34a2-4ed9-857c-82be4c9d0579
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/18/2017
ms.author: jeconnoc
ms.openlocfilehash: ece3511d0d3c12be13a649385ea9b6dd5c75bfe0
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44303702"
---
# <a name="how-to-create-and-deploy-a-cloud-service"></a>如何建立和部署雲端服務
Azure 入口網站提供兩種方法讓您建立和部署雲端服務：「快速建立」和「自訂建立」。

本主題說明如何使用「快速建立」方法建立新的雲端服務，然後使用 [上傳]  上傳雲端服務封裝並在 Azure 中部署。 當您使用這個方法時，Azure 入口網站會在過程中提供便利的連結，讓您完成所有要求。 如果您準備在建立雲端服務時加以部署，可以同時使用 [自訂建立] 進行這兩項作業。

> [!NOTE]
> 如果您計劃從 Azure DevOps 發佈您的雲端服務，請使用快速建立，然後從 Azure 快速入門或儀表板設定 Azure DevOps 發佈。 如需詳細資訊，請參閱[使用 Azure DevOps 持續傳遞至 Azure][TFSTutorialForCloudService]，或參閱 [快速入門] 頁面的說明。
>
>

## <a name="concepts"></a>概念
需要三個元件才能部署應用程式成為 Azure 中的雲端服務：

* **服務定義**  
  雲端服務定義檔 (.csdef) 定義服務模型，包括角色數目。
* **服務組態**  
  雲端服務組態檔 (.cscfg) 提供雲端服務和個別角色的組態設定，包括角色執行個體數。
* **服務封裝**  
  服務封裝 (.cspkg) 包含應用程式程式碼和組態以及服務定義檔。

您可以在 [這裡](cloud-services-model-and-package.md)深入了解這些內容，以及如何建立封裝。

## <a name="prepare-your-app"></a>準備您的應用程式
您部署雲端服務之前，必須先從應用程式程式碼和雲端服務組態檔 (.cscfg) 建立雲端服務封裝 (.cspkg)。 Azure SDK 提供準備這些必要部署檔案的工具。 您可以從 [Azure 下載](https://azure.microsoft.com/downloads/) 頁面安裝 SDK，使用您偏好的語言開發應用程式程式碼。

三個雲端服務功能需要特別組態，您才能匯出服務封裝：

* 如果您要部署使用安全通訊端層 (SSL) 進行資料加密的雲端服務，請針對 SSL [設定應用程式](cloud-services-configure-ssl-certificate-portal.md#modify) 。
* 如果您要設定角色執行個體的遠端桌面連線，請 [設定遠端桌面的角色](cloud-services-role-enable-remote-desktop-new-portal.md) 。
* 如果您要設定雲端服務的詳細資訊監視，請啟用雲端服務的 Azure 診斷。 *最小監視* (預設監視層級) 使用從角色執行個體 (虛擬機器) 的主機作業系統收集的效能計數器。 *詳細資訊監視* 會按照角色執行個體內的效能資料來收集其他度量，以便進一步分析應用程式處理期間發生的問題。 若要了解如何啟用 Azure 診斷，請參閱 [在 Azure 中啟用診斷](cloud-services-dotnet-diagnostics.md)。

若要使用 Web 角色或背景工作角色的部署來建立雲端服務，您必須 [建立服務封裝](cloud-services-model-and-package.md#servicepackagecspkg)。

## <a name="before-you-begin"></a>開始之前
* 如果您尚未安裝 Azure SDK，請按一下 [安裝 Azure SDK] 開啟 [Azure 下載頁面](https://azure.microsoft.com/downloads/)，然後依照您偏好使用的程式碼開發語言下載 SDK。 (您稍後將有機會這麼做。)
* 如果任何角色執行個體需要憑證，請建立憑證。 雲端服務需要含有私密金鑰的 .pfx 檔。 您建立並部署雲端服務時，可以將憑證上傳至 Azure。

## <a name="create-and-deploy"></a>建立和部署
1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 按一下 [建立資源] > [計算]，然後向下捲動至 [雲端服務] 並按一下。

    ![發佈您的雲端服務](media/cloud-services-how-to-create-deploy-portal/create-cloud-service.png)
3. 在新的 [雲端服務] 窗格中，輸入 [DNS 名稱] 的值。
4. 建立新的 [資源群組]  或選取現有的資源群組。
5. 選取 [位置] 。
6. 按一下 [封裝] 。 這會開啟 [上傳套件] 窗格。 填寫必要欄位。 如果您的任一個角色包含單一執行個體，請確定核取 [即使一個或多個角色包含單一執行個體，也要部署]  。
7. 請確定已選取 [開始部署]  。
8. 按一下 [確定] 以關閉 [上傳套件] 窗格。
9. 如果您沒有任何憑證可新增，請按一下 [建立] 。

    ![發佈您的雲端服務](media/cloud-services-how-to-create-deploy-portal/select-package.png)

## <a name="upload-a-certificate"></a>上傳憑證
如果您的部署套件 [設定為使用憑證](cloud-services-configure-ssl-certificate-portal.md#modify)，您現在可以上傳憑證。

1. 選取 [憑證]，然後在 [新增憑證] 窗格中選取 SSL 憑證 .pfx 檔案，並提供憑證的 [密碼]。
2. 按一下 [附加憑證]，然後按一下 [新增憑證] 窗格中的 [確定]。
3. 按一下 [雲端服務] 窗格中的 [建立]。 當部署達到了 [就緒]  狀態時，您可以繼續進行接下來的步驟。

    ![發佈您的雲端服務](media/cloud-services-how-to-create-deploy-portal/attach-cert.png)

## <a name="verify-your-deployment-completed-successfully"></a>確認部署是否成功完成
1. 按一下雲端服務執行個體。

    狀態應該會顯示服務為 [正在執行] 。
2. 在 [基本功能] 下，按一下 [網站 URL]，在網頁瀏覽器中開啟您的雲端服務。

    ![CloudServices_QuickGlance](./media/cloud-services-how-to-create-deploy-portal/running.png)

[TFSTutorialForCloudService]: http://go.microsoft.com/fwlink/?LinkID=251796

## <a name="next-steps"></a>後續步驟
* [雲端服務的一般設定](cloud-services-how-to-configure-portal.md)。
* 設定 [自訂網域名稱](cloud-services-custom-domain-name-portal.md)。
* [管理您的雲端服務](cloud-services-how-to-manage-portal.md)。
* 設定 [SSL 憑證](cloud-services-configure-ssl-certificate-portal.md)。
