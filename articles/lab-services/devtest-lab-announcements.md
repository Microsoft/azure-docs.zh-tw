---
title: 將公告張貼到 Azure DevTest Labs 中的實驗室 | Microsoft Docs
description: 了解如何將公告張貼到 Azure DevTest Labs 中的實驗室
services: devtest-lab,virtual-machines
documentationcenter: na
author: spelluru
manager: ''
editor: ''
ms.assetid: 67a09946-4584-425e-a94c-abe57c9cbb82
ms.service: devtest-lab
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/17/2018
ms.author: spelluru
ms.openlocfilehash: ecfaf24d1122b711a93e1335b79acbbc4235bdae
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39049944"
---
# <a name="post-an-announcement-to-a-lab-in-azure-devtest-labs"></a>將公告張貼到 Azure DevTest Labs 中的實驗室

身為實驗室系統管理員，您可以將自訂公告張貼在現有的實驗室中，通知使用者有關實驗室的近期變動或新增項目。 例如，您可以通知使用者有關：

- 可用的新 VM 大小
- 目前無法使用的映像
- 實驗室原則更新

張貼以後，公告會顯示在實驗室的 [概觀] 頁面上，而使用者可加以選取來取得詳細資訊。

公告功能主要用於臨時通知。  不再需要公告以後，您可以輕鬆地將它停用。

## <a name="steps-to-post-an-announcement-in-an-existing-lab"></a>在現有實驗室中張貼公告的步驟

1. 登入 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)。
1. 視需要選取 [所有服務]，然後從清單中選取 [DevTest Labs]。 (您的實驗室可能已顯示在 [所有資源] 下方的儀表板上)。
1. 從實驗室清單中，選取您想要在其中張貼公告的實驗室。  
1. 在實驗室的 [概觀] 區域中選取 [組態和原則]。  

    ![組態和原則按鈕](./media/devtest-lab-announcements/devtestlab-config-and-policies.png)

1. 在左側的 [設定] 之下，選取 [實驗室公告]。

    ![實驗室公告按鈕](./media/devtest-lab-announcements/devtestlab-announcements.png)

1. 若要在這個實驗室中建立使用者訊息，請將 [啟用] 設定為 [是]。

1. 您可以輸入**到期日**以指定日期和時間，此時間過後就不再對使用者顯示公告。 如果您未輸入到期日，直到您停用之前都會持續公告。

   > [!NOTE]
   > 公告到期之後，就不會再對使用者顯示，但是仍存在於 [實驗室公告] 窗格。 您可以對它進行編輯，然後重新啟用它，讓它再次作用。
   >
   >

1. 輸入 [公告標題] 和 [公告文字]。

   標題可多達 100 個字元，而且會在實驗室的 [概觀] 頁面上對使用者顯示。 如果使用者選取標題，就會顯示公告文字。

   公告文字接受 Markdown。 當您輸入公告文字時，您可以在畫面底部的 [預覽] 區域中檢視訊息。

    ![用於建立訊息的實驗室公告畫面。](./media/devtest-lab-announcements/devtestlab-post-announcement.png)


1. 一旦準備好張貼公告，請選取 [儲存]。

當您不想要再對實驗室使用者顯示此公告時，請返回 [實驗室公告] 頁面，然後將 [啟用] 設定為 [否]。 如果您指定了到期日，該日期和時間到了之後就會自動停用公告。

## <a name="steps-for-users-to-view-an-announcement"></a>使用者檢視公告的步驟

1. 從 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)選取一個實驗室。

1. 如果實驗室有為其張貼的公告，則資訊通知會顯示在實驗室的 [概觀] 頁面頂端。 此資訊通知是在建立公告時所指定的公告標題。

    ![概觀頁面上的實驗室公告](./media/devtest-lab-announcements/devtestlab-user-announcement.png)

1. 使用者可以選取訊息以檢視整個公告。

    ![實驗室公告的詳細資訊](./media/devtest-lab-announcements/devtestlab-user-announcement-text.png)

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="azure-resource-manager-template"></a>Azure Resource Manager 範本
您可以指定宣告作為 Azure Resource Manager 範本的一部分，如下列範例所示： 

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "name": {
            "type": "string",
            "defaultValue": "devtestlabfromarm"
        },
        "regionId": {
            "type": "string",
            "defaultValue": "eastus"
        }
    },
    "resources": [
        {
            "apiVersion": "2017-04-26-preview",
            "name": "[parameters('name')]",
            "type": "Microsoft.DevTestLab/labs",
            "location": "[parameters('regionId')]",
            "tags": {},
            "properties": {
                "labStorageType": "Premium",
                "announcement":
                {
                    "title": "Important! Three images are currently inaccessible. Click for more information.",
                    "markdown":"# Images are inaccessible\n\nThe following 3 images are currently not available for use: \n\n- image1\n- image2\n- image3\n\nI am working to fix the problem ASAP.",
                    "enabled": "Enabled",
                    "expirationDate":"2018-12-31T06:00:00+00:00",
                    "expired": "false"
                },
                "support": {
                    "markdown": "",
                    "enabled": "Enabled"
                }                
            },
            "resources": [
                {
                    "apiVersion": "2017-04-26-preview",
                    "name": "LabVmsShutdown",
                    "location": "[parameters('regionId')]",
                    "type": "schedules",
                    "dependsOn": [
                        "[resourceId('Microsoft.DevTestLab/labs', parameters('name'))]"
                    ],
                    "properties": {
                        "status": "Enabled",
                        "timeZoneId": "Eastern Standard Time",
                        "dailyRecurrence": {
                            "time": "1900"
                        },
                        "taskType": "LabVmsShutdownTask",
                        "notificationSettings": {
                            "status": "Disabled",
                            "timeInMinutes": 30
                        }
                    }
                },
                {
                    "apiVersion": "2017-04-26-preview",
                    "name": "[concat('Dtl', parameters('name'))]",
                    "type": "virtualNetworks",
                    "location": "[parameters('regionId')]",
                    "dependsOn": [
                        "[resourceId('Microsoft.DevTestLab/labs', parameters('name'))]"
                    ]
                }
            ]
        }
    ]
}
```

您可以使用下列其中一種方式，來部署 Azure Resource Manager 範本：

- [Azure 入口網站](../azure-resource-manager/resource-group-template-deploy-portal.md)
- [Azure PowerShell](../azure-resource-manager/resource-group-template-deploy.md)
- [Azure CLI](../azure-resource-manager/resource-group-template-deploy-cli.md)
- [REST API](../azure-resource-manager/resource-group-template-deploy-rest.md)

## <a name="next-steps"></a>後續步驟
* 如果您變更或設定實驗室原則，您可以張貼公告來通知使用者。 [設定原則和排程](devtest-lab-set-lab-policy.md)提供的資訊是關於如何使用自訂的原則，在訂用帳戶內套用限制和慣例。
* 瀏覽 [DevTest Labs Azure Resource Manager 快速入門範本資源庫 (英文)](https://github.com/Azure/azure-devtestlab/tree/master/Samples)。
