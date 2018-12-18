---
title: 在 Azure 上建立 Cloud Foundry
description: 深入了解如何設定在 Azure 上佈建 Cloud Foundry PCF 叢集所需的參數
services: Cloud Foundry
documentationcenter: CloudFoundry
author: ruyakubu
manager: brunoborges
editor: ruyakubu
ms.assetid: ''
ms.author: ruyakubu
ms.date: 09/13/2018
ms.devlang: ''
ms.service: Cloud Foundry
ms.tgt_pltfrm: multiple
ms.topic: tutorial
ms.workload: web
ms.openlocfilehash: a0a3379a8a2579080d9b686917395feec9cf8f3d
ms.sourcegitcommit: 609c85e433150e7c27abd3b373d56ee9cf95179a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "48250558"
---
# <a name="create-cloud-foundry-on-azure"></a>在 Azure 上建立 Cloud Foundry

本教學課程提供快速步驟來建立及產生在 Azure 上佈建 Pivotal Cloud Foundry PCF 叢集所需的參數。  Pivotal Cloud Foundry 解決方案可以藉由在 Azure [MarketPlace](https://azuremarketplace.microsoft.com/marketplace/apps/pivotal.pivotal-cloud-foundry) 上執行搜尋來找到。

![替代影像文字](media/deploy/pcf-marketplace.png "在 Azure 中搜尋 Pivotal Cloud Foundry")


## <a name="generate-an-ssh-public-key"></a>產生 SSH 公開金鑰

使用 Windows、Mac 或 Linux 來產生公開 SSH 金鑰的方式有很多種。

```Bash
ssh-keygen -t rsa -b 2048
```
- 按一下這裡以查看適用於您環境的[指示]( https://docs.microsoft.com/azure/virtual-machines/linux/ssh-from-windows)。

## <a name="create-a-service-principal"></a>建立服務主體

> [!NOTE]
>
> 建立服務主體需要擁有者帳戶權限。  此外，您可以撰寫指令碼來自動化服務主體的建立作業。 例如，使用 Azure CLI [az ad sp create-for-rbac](https://docs.microsoft.com/cli/azure/ad/sp?view=azure-cli-latest)。

1. 登入 Azure 帳戶。

    `az login`

    ![替代影像文字](media/deploy/az-login-output.png "Azure CLI 登入")
 
    複製 “id” 值作為您的**訂用帳戶識別碼**，複製 **tenantId** 值供稍後使用。

2. 設定此組態的預設訂用帳戶。

    `az account set -s {id}`

3. 為您的 PCF 建立 AAD 應用程式，並且指定唯一的英數字元密碼。  儲存密碼作為您的 **clientSecret**，供稍後使用。

    `az ad app create --display-name "Svc Prinicipal for OpsManager" --password {enter-your-password} --homepage "{enter-your-homepage}" --identifier-uris {enter-your-homepage}`

    複製輸出中的 “appId” 值作為 **ClientID**，供稍後使用。

    > [!NOTE]
    >
    > 選擇您自己的應用程式首頁和識別碼 URI。  例如 http://www.contoso.com。

4. 使用您的新 “appId” 建立服務主體。

    `az ad sp create --id {appId}`

5. 將您服務主體的權限角色設為**參與者**。

    `az role assignment create --assignee “{enter-your-homepage}” --role “Contributor” `

    或者，您也可以使用…

    `az role assignment create --assignee {service-princ-name} --role “Contributor” `

    ![替代影像文字](media/deploy/svc-princ.png "服務主體角色指派")

6. 確認您可以使用 appId、password 和 tenantId 來成功登入服務主體。

    `az login --service-principal -u {appId} -p {your-passward}  --tenant {tenantId}`

7. 使用您在前面複製的**訂用帳戶識別碼**、**tenantId**、**clientID** 和 **clientSecret** 值，以下列格式建立 .json 檔案。  儲存檔案。

    ```json
    {
        "subscriptionID": "{enter-your-subscription-Id-here}",
        "tenantID": "{enter-your-tenant-Id-here}",
        "clientID": "{enter-your-app-Id-here}",
        "clientSecret": "{enter-your-key-here}"
    }
    ```

## <a name="get-the-pivotal-network-token"></a>取得 Pivotal Network 權杖

1. 註冊或登入您的 [Pivotal Network](https://network.pivotal.io) 帳戶。
2. 按一下頁面右上方的設定檔名稱，然後選取 [編輯設定檔]。
3. 捲動至頁面底部，然後複製**舊版 API 權杖**的值。  這是您稍後會用到的 **Pivotal Network 權杖**。

## <a name="provision-your-cloud-foundry-on-azure"></a>在 Azure 上佈建 Cloud Foundry

1. 現在您已經有了[在 Azure 上佈建 Pivotal Cloud Foundry](https://azuremarketplace.microsoft.com/marketplace/apps/pivotal.pivotal-cloud-foundry) 叢集所需的所有參數。
2. 輸入參數，並建立您的 PCF 叢集。

## <a name="verify-the-deployment-and-log-into-the-pivotal-ops-manager"></a>驗證部署並登入 Pivotal Ops Manager

1. 您的 PCF 叢集應該會顯示部署狀態。

    ![替代影像文字](media/deploy/deployment.png "Azure 部署狀態")

2. 按一下左側導覽中的 [部署] 連結以取得 PCF Ops Manager 的認證，然後在下一頁按一下 [部署名稱]。
3. 在左側導覽中按一下 [輸出] 連結，以顯示 PCF Ops Manager 的 URL、使用者名稱和密碼。  “OPSMAN-FQDN” 值是 URL。
 
    ![替代影像文字](media/deploy/deploy-outputs.png "Cloud Foundry 部署輸出")
 
4. 在網頁瀏覽器中啟動 URL，並且輸入上一個步驟中的認證以登入。

    ![替代影像文字](media/deploy/pivotal-login.png "Pivotal 登入頁面")
         
    > [!NOTE]
    >
    > 如果 Internet Explorer 瀏覽器由於網站不安全警告訊息而發生失敗，請按一下 [詳細資訊] 和 [繼續前往網頁]。  若為 Firefox，按一下 [進階] 並且新增憑證以繼續。

5. 您的 PCF Ops Manager 應該會顯示已部署的 Azure 執行個體。 現在您可以開始在這裡部署和管理您的應用程式！
               
    ![替代影像文字](media/deploy/ops-mgr.png "Pivotal 中的已部署 Azure 執行個體")
 
