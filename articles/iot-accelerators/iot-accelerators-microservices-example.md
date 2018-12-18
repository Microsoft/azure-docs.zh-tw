---
title: 變更並重新部署微服務 | Microsoft Docs
description: 本教學課程示範如何在遠端監視中變更並重新部署微服務
author: dominicbetts
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 04/19/2018
ms.topic: conceptual
ms.openlocfilehash: 0b206d7b56fc8a65c422a4ce22b2f5585e71c8da
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47219409"
---
# <a name="customize-and-redeploy-a-microservice"></a>自訂和重新部署微服務

本教學課程示範如何編輯遠端監視解決方案中的其中一個微服務，建置該微服務的映像，將該映像部署到 Docker Hub，然後將它用於遠端監視解決方案。 為了引入此概念，本教學課程會使用基本情節，讓您可在其中呼叫微服務 API，並將狀態訊息從 "Alive and Well" 變更為 "New Edits Made Here!"。

遠端監視解決方案所使用的微服務，是使用提取自 Docker Hub 的 Docker 映像所建置。 

在本教學課程中，您了解如何：

>[!div class="checklist"]
> * 在遠端監視解決方案中編輯和建置微服務
> * 建置 Docker 映像
> * 將 Docker 映像推送至 Docker Hub
> * 提取新的 Docker 映像
> * 將變更視覺化 

## <a name="prerequisites"></a>必要條件

為了依循本教學課程進行操作，您需要：

>[!div class="checklist"]
> * [本機部署遠端監視解決方案加速器](iot-accelerators-remote-monitoring-deploy-local.md)
> * [Docker 帳戶](https://hub.docker.com/) \(英文\)
> * [Postman](https://www.getpostman.com/) \(英文\)：檢視 API 回應所需的項目

## <a name="call-the-api-and-view-response-status"></a>呼叫 API 並檢視回應狀態

在這一節中，您會呼叫預設的 IoT 中樞管理員微服務 API。 此 API 會傳回您稍後要藉由自訂微服務來變更的狀態訊息。

1. 確定遠端監視解決方案正在您的電腦上以本機方式執行。
2. 找出您已下載的 Postman 並開啟它。
3. 在 Postman 中，於 GET 中輸入下列內容： http://localhost:8080/iothubmanager/v1/status。
4. 檢視傳回內容，您應該會看到 "Status": "OK:Alive and Well"。

    ![活躍的 Postman 訊息](./media/iot-accelerators-microservices-example/postman-alive-well.png)

## <a name="change-the-status-and-build-the-image"></a>變更狀態並建置映像

現在將 IoT 中樞管理員微服務的狀態訊息變更為 "New Edits Made Here!"， 然後使用這個新的狀態重新建置 Docker 映像。 如果您在此處遇到問題，請參閱我們的[疑難排解](#Troubleshoot)一節。

1. 確定您的終端機已開啟，並變更為您已在其中複製遠端監視解決方案的目錄。 
2. 將目錄變更為 "azure-iot-pcs-remote-monitoring-dotnet/services/iothub-manager/WebService/v1/Controllers"。
3. 以您偏好的任何文字編輯器或 IDE 開啟 StatusController.cs。 
4. 找出下列程式碼：

    ```csharp
    return new StatusApiModel(true, "Alive and well");
    ```

    將它變更為下列程式碼，然後加以儲存。

    ```csharp
    return new StatusApiModel(true, "New Edits Made Here!");
    ```

5. 返回您的終端機，但現在變更為下列目錄："azure-iot-pcs-remote-monitoring-dotnet/services/iothub-manager/scripts/docker"。
6. 若要建置新的 Docker 映像，請輸入

    ```cmd/sh
    sh build
    ```

7. 若要確認已成功建立新的映像，請輸入

    ```cmd/sh
    docker images 
    ```

存放庫應該是 "azureiotpcs/iothub-manager-dotnet"。

![成功的 Docker 映像](./media/iot-accelerators-microservices-example/successful-docker-image.png)

## <a name="tag-and-push-the-image"></a>標記並推送映像
在您可以將新的 Docker 映像推送至 Docker Hub 之前，Docker 會預期該映像已被加上標記。 如果您在此處遇到問題，請參閱我們的[疑難排解](#Troubleshoot)一節。

1. 藉由輸入下列內容來找到您所建立 Docker 映像的映像識別碼：

    ```cmd/sh
    docker images
    ```

2. 若要以 "testing" 來標記映像，請輸入

    ```cmd/sh
    docker tag [Image ID] [docker ID]/iothub-manager-dotnet:testing 
    ```

3. 若要將新標記的映像推送至 Docker Hub，請輸入

    ```cmd/sh
    docker push [docker ID]/iothub-manager-dotnet:testing
    ```

4. 開啟網際網路瀏覽器，然後移至您的 [Docker Hub](https://hub.docker.com/) \(英文\) 並登入。
5. 您現在應該會在 Docker Hub 上看到新推送的 Docker 映像。
![Docker Hub 中的 Docker 映像](./media/iot-accelerators-microservices-example/docker-image-in-docker-hub.png)

## <a name="update-your-remote-monitoring-solution"></a>更新遠端監視解決方案
您現在需要更新本機 docker-compose.yml，以便從 Docker Hub 提取新的 Docker 映像。 如果您在此處遇到問題，請參閱我們的[疑難排解](#Troubleshoot)一節。

1. 返回終端機並變更為下列目錄："azure-iot-pcs-remote-monitoring-dotnet/services/scripts/local"。
2. 以您偏好的任何文字編輯器或 IDE 開啟 docker-compose.yml。
3. 找出下列程式碼：

    ```docker
    image: azureiotpcs/iothub-manager-dotnet:testing
    ```

    變更它以使其看起來如下圖，然後加以儲存。

    ```cmd/sh
    image: [docker ID]/iothub-manager-dotnet:testing
    ```

## <a name="view-the-new-response-status"></a>檢視新的回應狀態
最後，請重新部署遠端監視解決方案的本機執行個體，並在 Postman 中檢視新的狀態回應。

1. 返回您的終端機並變更為下列目錄："azure-iot-pcs-remote-monitoring-dotnet/scripts/local"。
2. 藉由在終端機輸入下列命令，來啟動遠端監視解決方案的本機執行個體：

    ```cmd/sh
    docker-compose up
    ```

3. 找出您已下載的 Postman 並開啟它。
4. 在 Postman 中，於 GET 中輸入下列要求： http://localhost:8080/iothubmanager/v1/status。 您現在應該會看到 "Status": "OK: New Edits Made Here!"。

![New Edits Made Here 的 Postman 訊息](./media/iot-accelerators-microservices-example/new-postman-message.png)

## <a name="Troubleshoot"></a>疑難排解

如果您在執行時遇到問題，請嘗試移除本機電腦上的 Docker 映像和容器。

1. 若要移除所有容器，您必須先停止所有執行中的容器。 開啟您的終端機並輸入

    ```cmd/sh
    docker stop $(docker ps -aq)
    docker rm $(docker ps -aq)
    ```
    
2. 若要移除所有映像，請開啟您的終端機並輸入 

    ```cmd/sh
    docker rmi $(docker images -q)
    ```

3. 您可以藉由輸入下列內容來檢查電腦上是否有任何容器

    ```cmd/sh
    docker ps -aq 
    ```

    如果您已成功移除所有容器，則應該不會顯示任何內容。

4. 您可以藉由輸入下列內容來檢查電腦上是否有任何映像

    ```cmd/sh
    docker images
    ```

    如果您已成功移除所有容器，則應該不會顯示任何內容。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

<!-- Repeat task list from intro -->
>[!div class="checklist"]
> * 在遠端監視解決方案中編輯和建置微服務
> * 建置 Docker 映像
> * 將 Docker 映像推送至 Docker Hub
> * 提取新的 Docker 映像
> * 將變更視覺化 

下一個要嘗試的動作是[在遠端監視解決方案中自訂裝置模擬器微服務](iot-accelerators-microservices-example.md)

如需關於遠端監視解決方案的開發人員詳細資訊，請參閱：

* [開發人員參考指南](https://github.com/Azure/azure-iot-pcs-remote-monitoring-dotnet/wiki/Developer-Reference-Guide)
<!-- Next tutorials in the sequence -->

