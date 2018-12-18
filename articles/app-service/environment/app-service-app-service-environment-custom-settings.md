---
title: App Service 環境的自訂設定
description: App Service 環境的自訂組態設定
services: app-service
documentationcenter: ''
author: stefsch
manager: nirma
editor: ''
ms.assetid: 1d1d85f3-6cc6-4d57-ae1a-5b37c642d812
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 08/22/2016
ms.author: stefsch
ms.custom: mvc
ms.openlocfilehash: d60cdca78c143996fa5935726db0631321c9e2fe
ms.sourcegitcommit: b854df4fc66c73ba1dd141740a2b348de3e1e028
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2017
ms.locfileid: "26129510"
---
# <a name="custom-configuration-settings-for-app-service-environments"></a>App Service 環境的自訂組態設定
## <a name="overview"></a>概觀
因為 App Service 環境對單一客戶是隔離的，所以有某些專門套用到 App Service 環境的組態設定。 本文記錄各種可供 App Service 環境使用的特定自訂。

如果您沒有 App Service 環境，請參閱 [如何建立 App Service 環境](app-service-web-how-to-create-an-app-service-environment.md)。

您可以使用新 **clusterSettings** 屬性的陣列儲存 App Service 環境自訂。 這個屬性是在 *hostingEnvironments* Azure Resource Manager 實體的「屬性」字典中找到的。

下列縮寫的 Resource Manager 範本程式碼片段顯示 **clusterSettings** 屬性︰

    "resources": [
    {
       "apiVersion": "2015-08-01",
       "type": "Microsoft.Web/hostingEnvironments",
       "name": ...,
       "location": ...,
       "properties": {
          "clusterSettings": [
             {
                 "name": "nameOfCustomSetting",
                 "value": "valueOfCustomSetting"
             }
          ],
          "workerPools": [ ...],
          etc...
       }
    }

**clusterSettings** 屬性可以包含在 Resource Manager 範本中，以更新 App Service 環境。

## <a name="use-azure-resource-explorer-to-update-an-app-service-environment"></a>使用 Azure 資源總管更新 App Service 環境
或者，您可以使用 [Azure 資源總管](https://resources.azure.com)更新 App Service 環境。  

1. 在 [資源總管] 中，移至 App Service 環境的節點 ([訂用帳戶] > [resourceGroups] > [提供者] > [Microsoft.Web] > [hostingEnvironments])。 接著，按一下您想要更新的特定 App Service 環境。
2. 按一下右窗格上方工具列中的 [讀取/寫入]  ，允許在資源總管中進行互動式編輯。  
3. 按一下藍色的 [編輯]  按鈕，開放編輯 Resource Manager 範本。
4. 向下捲動到右窗格底部。 **clusterSettings** 屬性位在最底部，您可以在此輸入或更新其值。
5. 輸入 (或複製並貼上) 您希望 **clusterSettings** 屬性出現的組態值陣列。  
6. 按一下右窗格上方的綠色 [PUT]  按鈕，確認 App Service 環境的變更。

不過，送出變更後，約需 30 分鐘乘以 App Service 環境中前端數量的時間，變更才會生效。
例如，如果 App Service 環境有四個前端，大約需要兩個小時才能完成組態更新。 實行組態變更時，App Service 環境上就不會進行其他調整作業或組態變更作業。

## <a name="disable-tls-10"></a>停用 TLS 1.0
客戶的常見問題是，尤其是處理 PCI 規範稽核的客戶，如何明確停用其應用程式的 TLS 1.0。

透過下列 **clusterSettings** 項目可以停用 TLS 1.0︰

        "clusterSettings": [
            {
                "name": "DisableTls1.0",
                "value": "1"
            }
        ],

## <a name="change-tls-cipher-suite-order"></a>變更 TLS 加密套件順序
來自客戶的另一個問題是，他們是否可以修改由其伺服器交涉的加密的清單，而這可透過修改 **clusterSettings** 來達成，如下所示。 您可以從[此 MSDN 文章](https://msdn.microsoft.com/library/windows/desktop/aa374757\(v=vs.85\).aspx)擷取可用加密套件的清單。

        "clusterSettings": [
            {
                "name": "FrontEndSSLCipherSuiteOrder",
                "value": "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384_P256,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256_P256,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384_P256,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256_P256,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA_P256,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA_P256"
            }
        ],

> [!WARNING]
> 如果對安全通道無法了解的加密套件設定了不正確的值，對您的伺服器的所有 TLS 通訊可能會停止運作。 在這種情況下，您必須從 **clusterSettings** 移除 FrontEndSSLCipherSuiteOrder 項目，並提交更新的 Resource Manager 範本以還原回預設的加密套件設定。  請謹慎使用這項功能。
> 
> 

## <a name="get-started"></a>開始使用
Azure 快速入門 Resource Manager 範本網站包含具有 [建立 App Service 環境](https://azure.microsoft.com/documentation/templates/201-web-app-ase-create/)基本定義的範本。

<!-- LINKS -->

<!-- IMAGES -->
