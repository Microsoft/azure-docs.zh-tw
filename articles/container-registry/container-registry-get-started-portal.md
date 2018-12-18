---
title: 快速入門 - 使用 Azure 入口網站在 Azure 中建立私用的 Docker 登錄
description: 快速了解如何使用 Azure 入口網站建立私用的 Docker 容器登錄。
services: container-registry
author: mmacy
manager: jeconnoc
ms.service: container-registry
ms.topic: quickstart
ms.date: 03/03/2018
ms.author: marsma
ms.custom: mvc
ms.openlocfilehash: 55da52e0d314c353c669c56ad918c4dd6bef44c3
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32169800"
---
# <a name="quickstart-create-a-container-registry-using-the-azure-portal"></a>快速入門：使用 Azure 入口網站建立容器登錄

Azure Container Registry 是 Azure 中的私人 Docker 登錄，您可以在其中儲存並管理私人 Docker 容器映像。 在本快速入門中，您將使用 Azure 入口網站建立容器登錄、將容器映像推送到登錄中，最後將容器從登錄部署至 Azure 容器執行個體 (ACI)。

若要完成此快速入門，您必須在本機上有 Docker 帳戶。 Docker 提供可輕鬆在 [Mac][docker-mac]、[Windows][docker-windows] 或 [Linux][docker-linux] 系統上設定 Docker 的套件。

## <a name="sign-in-to-azure"></a>登入 Azure

在 https://portal.azure.com 登入 Azure 入口網站。

## <a name="create-a-container-registry"></a>建立容器登錄庫

選取 [建立資源] > [容器] > [Azure Container Registry]。

![在 Azure 入口網站中建立容器登錄][qs-portal-01]

輸入 [登錄名稱] 和 [資源群組] 的值。 登錄名稱在 Azure 內必須是唯一的，且包含 5-50 個英數字元。 建立名為 `myResourceGroup` 的新資源群組，[SKU] 請選取 [基本]。 選取 [建立] 以部署 ACR 執行個體。

![在 Azure 入口網站中建立容器登錄][qs-portal-03]

在本快速入門中，我們會建立「基本」登錄。 有數個不同的 SKU 提供 Azure Container Registry，簡略說明於下表。 如需個別項目更詳細的資訊，請參閱[容器登錄 SKU][container-registry-skus]。

[!INCLUDE [container-registry-sku-matrix](../../includes/container-registry-sku-matrix.md)]

當 [部署成功] 訊息出現時，請在入口網站中選取容器登錄，然後選取 [存取金鑰]。

![在 Azure 入口網站中建立容器登錄][qs-portal-05]

在 [管理使用者] 下，選取 [啟用]。 請將下列值記下：

* 登入伺服器
* 使用者名稱
* password

您在透過 Docker CLI 使用登錄時，請使用下列步驟中的這些值。

![在 Azure 入口網站中建立容器登錄][qs-portal-06]

## <a name="log-in-to-acr"></a>登入 ACR

發送和提取容器映像之前，您必須登入 ACR 執行個體。 若要這樣做，請使用 [docker login][docker-login] 命令。 將*使用者名稱*、*密碼*和*登入伺服器*取代為您在上一個步驟中記下的值。

```bash
docker login --username <username> --password <password> <login server>
```

完成後，此命令會傳回 `Login Succeeded`。 您也可能會看到安全性警告，建議您使用 `--password-stdin` 參數。 其使用超出本文的範圍時，建議您遵循此最佳做法。 如需詳細資訊，請參閱 [docker login][docker-login] 命令參考。

## <a name="push-image-to-acr"></a>推送映像到 ACR

若要將映像推送到 Azure Container Registry，您必須先有映像。 如有需要，請執行下列命令，以從 Docker Hub 提取現有的映像。

```bash
docker pull microsoft/aci-helloworld
```

在將映像推送至您的登錄之前，必須使用 ACR 登入伺服器名稱來標記映像。 使用 [docker tag][docker-tag] 命令來標記映像。 將登入伺服器取代為您先前記錄的登入伺服器名稱。

```bash
docker tag microsoft/aci-helloworld <login server>/aci-helloworld:v1
```

最後，使用 [docker push][docker-push] 將映像推送到 ACR 執行個體。 將登入伺服器取代為 ACR 執行個體的登入伺服器名稱。

```bash
docker push <login server>/aci-helloworld:v1
```

成功 `docker push` 命令的輸出類似於：

```
The push refers to a repository [uniqueregistryname.azurecr.io/aci-helloworld]
7c701b1aeecd: Pushed
c4332f071aa2: Pushed
0607e25cc175: Pushed
d8fbd47558a8: Pushed
44ab46125c35: Pushed
5bef08742407: Pushed
v1: digest: sha256:f2867748615cc327d31c68b1172cc03c0544432717c4d2ba2c1c2d34b18c62ba size: 1577
```

## <a name="list-container-images"></a>列出容器映像

若要在您的 ACR 執行個體中列出映像，請瀏覽至入口網站中的登錄，然後選取 [存放庫]，以及選取您使用 `docker push` 建立的存放庫。

在此範例中，我們選取 **aci-helloworld** 存放庫，且可以在 **TAGS**之下看到 `v1` 標記的映像。

![在 Azure 入口網站中建立容器登錄][qs-portal-09]

## <a name="deploy-image-to-aci"></a>將映像部署至 ACI

為了從登錄部署到執行個體，我們必須瀏覽至存放庫 (aci-helloworld)，然後按一下 v1 旁邊的省略符號。

![從入口網站啟動 Azure 容器執行個體][qs-portal-10]

將會出現操作功能表，請選取 [執行執行個體]：

![啟動 ACI 操作功能表][qs-portal-11]

填入 [容器名稱]，確定已選取正確的訂用帳戶，選取現有的 [資源群組] "myResourceGroup"，然後按一下 [確定] 以啟動 Azure容器執行個體。

![啟動 ACI 部署選項][qs-portal-12]

當部署開始時，系統會在入口網站儀表板上放置一個圖格，以指出部署進度。 部署完成時，此圖格會更新以顯示新的 **mycontainer** 容器群組。

![ACI 部署狀態][qs-portal-13]

選取 mycontainer 容器群組以顯示容器群組屬性。 記下容器群組的 [IP 位址]，以及容器的 [狀態]。

![ACI 容器詳細資料][qs-portal-14]

## <a name="view-the-application"></a>檢視應用程式

一旦容器處於 [執行中] 狀態，請使用您喜愛的瀏覽器瀏覽至您在上一個步驟中記下的 IP 位址，以顯示應用程式。

![瀏覽器中的 Hello World 應用程式][qs-portal-15]

## <a name="clean-up-resources"></a>清除資源

若要清除資源，請在入口網站中瀏覽至 **myResourceGroup** 資源群組。 載入資源群組後，請按一下 [刪除資源群組] 來移除資源群組、Azure Container Registry 及所有的 Azure 容器執行個體。

![在 Azure 入口網站中建立容器登錄][qs-portal-08]

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已使用 Azure CLI 建立 Azure Container Registry，以及透過 Azure 容器執行個體啟動其執行個體。 請繼續進行 Azure 容器執行個體教學課程，以深入了解 ACI。

> [!div class="nextstepaction"]
> [Azure Container Instances 教學課程][container-instances-tutorial-prepare-app]

<!-- IMAGES -->
[qs-portal-01]: ./media/container-registry-get-started-portal/qs-portal-01.png
[qs-portal-02]: ./media/container-registry-get-started-portal/qs-portal-02.png
[qs-portal-03]: ./media/container-registry-get-started-portal/qs-portal-03.png
[qs-portal-04]: ./media/container-registry-get-started-portal/qs-portal-04.png
[qs-portal-05]: ./media/container-registry-get-started-portal/qs-portal-05.png
[qs-portal-06]: ./media/container-registry-get-started-portal/qs-portal-06.png
[qs-portal-07]: ./media/container-registry-get-started-portal/qs-portal-07.png
[qs-portal-08]: ./media/container-registry-get-started-portal/qs-portal-08.png
[qs-portal-09]: ./media/container-registry-get-started-portal/qs-portal-09.png
[qs-portal-10]: ./media/container-registry-get-started-portal/qs-portal-10.png
[qs-portal-11]: ./media/container-registry-get-started-portal/qs-portal-11.png
[qs-portal-12]: ./media/container-registry-get-started-portal/qs-portal-12.png
[qs-portal-13]: ./media/container-registry-get-started-portal/qs-portal-13.png
[qs-portal-14]: ./media/container-registry-get-started-portal/qs-portal-14.png
[qs-portal-15]: ./media/container-registry-get-started-portal/qs-portal-15.png

<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-login]: https://docs.docker.com/engine/reference/commandline/login/
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- LINKS - internal -->
[container-instances-tutorial-prepare-app]: ../container-instances/container-instances-tutorial-prepare-app.md
[container-registry-skus]: container-registry-skus.md
