---
title: 如何使用 Azure CLI 和 IoT 擴充功能來管理 IoT 中樞裝置佈建服務 | Microsoft Docs
description: 了解如何使用 Azure CLI 和 IoT 擴充功能來管理 IoT 中樞裝置佈建服務
author: chrissie926
ms.author: menchi
ms.date: 01/17/2018
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
manager: briz
ms.openlocfilehash: cfbebf8570ee044698b0f4e0abdd58370b04f759
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46992859"
---
# <a name="how-to-use-azure-cli-and-the-iot-extension-to-manage-the-iot-hub-device-provisioning-service"></a>如何使用 Azure CLI 和 IoT 擴充功能來管理 IoT 中樞裝置佈建服務

[Azure CLI](https://docs.microsoft.com/cli/azure?view=azure-cli-latest) 是一個開放原始碼跨平台命令列工具，用來管理 Azure 資源 (例如 IoT Edge)。 Azure CLI 適用於 Windows、Linux 和 MacOS。 Azure CLI 可讓您管理 Azure IoT 中樞資源、裝置佈建服務執行個體，以及現成的連結中樞。

IoT 擴充功能以裝置管理和完整 IoT Edge 功能來擴充 Azure CLI 的功能。

在本教學課程中，您會先完成設定 Azure CLI 和 IoT 擴充功能的步驟。 然後您會了解如何執行 CLI 命令，以執行基本裝置佈建服務作業。 

## <a name="installation"></a>安裝 

### <a name="step-1---install-python"></a>步驟 1 - 安裝 Python

需要 [Python 2.7x 或 Python 3.x](https://www.python.org/downloads/)。

### <a name="step-2---install-the-azure-cli"></a>步驟 2 - 安裝 Azure CLI

請遵循[安裝指示](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)在您的環境中設定 Azure CLI。 您的 Azure CLI 版本至少必須是 2.0.24 或更新版本。 使用 `az –version` 進行驗證。 這個版本支援 az 擴充命令並引進 Knack 命令架構。 在 Windows 上進行安裝的最簡單方式就是下載並安裝 [MSI](https://aka.ms/InstallAzureCliWindows)。

### <a name="step-3---install-iot-extension"></a>步驟 3 - 安裝 IoT 擴充功能

[IoT 擴充功能讀我檔案](https://github.com/Azure/azure-iot-cli-extension)說明安裝此擴充功能的數種方式。 最簡單的方式就是執行 `az extension add --name azure-cli-iot-ext`。 安裝之後，您可以使用 `az extension list` 來驗證目前安裝的擴充功能，或使用 `az extension show --name azure-cli-iot-ext` 來查看有關 IoT 擴充功能的詳細資料。 若要移除此擴充功能，您可以使用 `az extension remove --name azure-cli-iot-ext`。


## <a name="basic-device-provisioning-service-operations"></a>基本裝置佈建服務作業
此範例說明如何登入您的 Azure 帳戶、建立 Azure 資源群組 (可保存 Azure 解決方案相關資源的容器)、建立 IoT 中樞、建立裝置佈建服務、列出現有的裝置佈建服務，以及使用 CLI 命令建立連結的 IoT 中樞。 

開始之前，請先完成先前所述的安裝步驟。 如果您沒有 Azure 帳戶，可以立即[建立一個免費帳戶](https://azure.microsoft.com/free/?v=17.39a)。 


### <a name="1-log-in-to-the-azure-account"></a>1.登入 Azure 帳戶
  
    az login

![登入][1]

### <a name="2-create-a-resource-group-iothubblogdemo-in-eastus"></a>2.在 eastus 中建立資源群組 IoTHubBlogDemo

    az group create -l eastus -n IoTHubBlogDemo

![建立資源群組][2]


### <a name="3-create-two-device-provisioning-services"></a>3.建立兩個裝置佈建服務

    az iot dps create --resource-group IoTHubBlogDemo --name demodps

![建立裝置佈建服務][3]

    az iot dps create --resource-group IoTHubBlogDemo --name demodps2

### <a name="4-list-all-the-existing-device-provisioning-services-under-this-resource-group"></a>4.列出此資源群組之下的所有現有裝置佈建服務

    az iot dps list --resource-group IoTHubBlogDemo

![列出裝置佈建服務][4]


### <a name="5-create-an-iot-hub-blogdemohub-under-the-newly-created-resource-group"></a>5.在新建立的資源群組之下建立 IoT 中樞 blogDemoHub

    az iot hub create --name blogDemoHub --resource-group IoTHubBlogDemo

![建立 IoT 中樞][5]

### <a name="6-link-one-existing-iot-hub-to-a-device-provisioning-service"></a>6.將一個現有的 IoT 中樞連結至裝置佈建服務

    az iot dps linked-hub create --resource-group IoTHubBlogDemo --dps-name demodps --connection-string <connection string> -l westus

![連結中樞][5]

<!-- Images -->
[1]: ./media/how-to-manage-dps-with-cli/login.jpg
[2]: ./media/how-to-manage-dps-with-cli/create-resource-group.jpg
[3]: ./media/how-to-manage-dps-with-cli/create-dps.jpg
[4]: ./media/how-to-manage-dps-with-cli/list-dps.jpg
[5]: ./media/how-to-manage-dps-with-cli/create-hub.jpg
[6]: ./media/how-to-manage-dps-with-cli/link-hub.jpg


## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 註冊裝置
> * 啟動裝置
> * 確認裝置已註冊

前進到下一個教學課程，以了解如何跨負載平衡中樞來佈建多個裝置。 

> [!div class="nextstepaction"]
> [跨負載平衡 IoT 中樞來佈建裝置](./tutorial-provision-multiple-hubs.md)
