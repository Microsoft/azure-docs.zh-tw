---
title: 在 Azure 中的 Service Fabric 上建立 Linux 容器應用程式 | Microsoft Docs
description: 在此快速入門中，您將使用您的應用程式建置 Docker 映像、將映像推送到容器登錄，然後將您的容器部署至 Service Fabric 叢集。
services: service-fabric
documentationcenter: linux
author: suhuruli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: python
ms.topic: quickstart
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 04/11/2018
ms.author: suhuruli
ms.custom: mvc
ms.openlocfilehash: c84470936dfc9610f23dacb55d8d8643a2651f71
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46948395"
---
# <a name="quickstart-deploy-linux-containers-to-service-fabric"></a>快速入門：將 Linux 容器部署至 Service Fabric

Azure Service Fabric 是一個分散式系統平台，可讓您部署及管理可調整和可信賴的微服務與容器。

本快速入門示範如何將 Linux 容器部署到 Service Fabric 叢集。 完成後，您會有一個投票應用程式，它是由在 Service Fabric 叢集中執行的 Python Web 前端和 Redis 後端所組成。 此外，您也將了解如何容錯移轉應用程式，以及如何調整叢集中的應用程式。

![投票應用程式網頁][quickstartpic]

在此快速入門中，您會在 Azure Cloud Shell 中使用 Bash 環境執行 Service Fabric CLI 命令。 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/)。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果這是您第一次執行 Cloud Shell，系統會要求您設定 `clouddrive` 檔案共用。 您可以接受預設值，或連結現有的檔案共用。 若要深入了解，請參閱[設定 `clouddrive` 檔案共用](https://docs.microsoft.com/azure/cloud-shell/persisting-shell-storage#set-up-a-clouddrive-file-share)。

## <a name="get-the-application-package"></a>取得應用程式套件

若要將容器部署至 Service Fabric，您需要一組可描述個別容器和應用程式的資訊清單檔案 (應用程式定義)。

在 Cloud Shell 中，使用 git 複製應用程式定義的複本；然後將目錄切換至您複本中的 `Voting` 目錄。

```bash
git clone https://github.com/Azure-Samples/service-fabric-containers.git

cd service-fabric-containers/Linux/container-tutorial/Voting
```

## <a name="create-a-service-fabric-cluster"></a>建立 Service Fabric 叢集

若要將應用程式部署至 Azure，您需要用來執行該應用程式的 Service Fabric 叢集。 合作對象叢集可提供快速建立 Service Fabric 叢集的簡單方式。 合作對象叢集是 Azure 上裝載的免費、限時 Service Fabric 叢集，由 Service Fabric 小組所執行。 您可以使用合作對象叢集來部署應用程式，並了解平台。 叢集會使用單一的自我簽署憑證，以確保節點對節點和用戶端對節點的安全性。

登入並加入 [Linux 叢集](http://aka.ms/tryservicefabric)。 藉由按一下 [PFX] 連結，將 PFX 憑證下載至您的電腦。 按一下**讀我檔案**連結以尋找憑證密碼，以及關於如何設定各種環境以使用憑證的指示。 請將 [歡迎] 頁面和 [讀我檔案] 頁面保持為開啟，您將在下列步驟使用其中的某些指示。

> [!Note]
> 每小時可用的合作對象叢集數目有限。 如果您在嘗試註冊合作對象叢集時收到錯誤，可以等候一段時間再重試，也可以依照[在 Azure 上建立 Service Fabric 叢集](service-fabric-tutorial-create-vnet-and-linux-cluster.md)中的步驟，在您的訂用帳戶中建立叢集。
>
>如果您建立自己的叢集，則必須注意，Web 前端服務設定為在連接埠 80 上接聽傳入流量。 請確定您的叢集中已開啟該連接埠。 (如果您使用合作對象叢集，此連接埠已開啟。)
>

## <a name="configure-your-environment"></a>設定您的環境

Service Fabric 提供了數項可用來管理叢集及其應用程式的工具：

- Service Fabric Explorer，此為以瀏覽器為基礎的工具。
- Service Fabric 命令列介面 (CLI)，此 CLI 會在 Azure CLI 之上執行。 
- PowerShell 命令。

在此快速入門中，您會在 Cloud Shell 和 Service Fabric Explorer 中使用 Service Fabric CLI。 以下幾節將說明如何安裝使用這些工具連線至安全叢集所需的憑證。

### <a name="configure-certificate-for-the-service-fabric-cli"></a>設定 Service Fabric CLI 的憑證

若要在 Cloud Shell 中使用 CLI，您必須將憑證 PFX 檔案上傳至 Cloud Shell，然後用它來建立 PEM 檔案。

1. 若要將憑證上傳至您目前在 Cloud Shell 中的工作目錄，請將憑證 PFX 檔案下載到您的電腦後存放的資料夾位置，拖放到您的 Cloud Shell 視窗中。

2. 若要將 PFX 檔案轉換為 PEM 檔案，請使用下列命令。 (對於合作對象叢集，您可以從 [讀我檔案] 頁面上的指示中，複製您的 PFX 檔案和密碼適用的特定命令。)

    ```bash
    openssl pkcs12 -in party-cluster-1486790479-client-cert.pfx -out party-cluster-1486790479-client-cert.pem -nodes -passin pass:1486790479
    ```

### <a name="configure-certificate-for-service-fabric-explorer"></a>設定 Service Fabric Explorer 的憑證

若要使用 Service Fabric Explorer，您必須將您從合作對象叢集網站下載的憑證 PFX 檔案匯入至憑證存放區 (Windows 或 Mac) 或瀏覽器本身 (Ubuntu)。 您需要 PFX 私密金鑰密碼；此密碼可從 [讀我檔案] 頁面取得。

請使用您最熟悉的方法在您的系統上匯入憑證。 例如︰

- 在 Windows 上：按兩下 PFX 檔案，並依照提示在您的個人存放區中安裝憑證：`Certificates - Current User\Personal\Certificates`。 或者，您可以使用 **ReadMe** 指示中的 PowerShell 命令。
- 在 Mac 上：按兩下 PFX 檔案，並依照提示在您的 Keychain 中安裝憑證。
- 在 Ubuntu 上：Mozilla Firefox 是 Ubuntu 16.04 中的預設瀏覽器。 若要將憑證匯入 Firefox 中，請按一下瀏覽器右上角的功能表按鈕，然後按一下 [選項]。 在 [喜好設定] 頁面上，使用搜尋方塊搜尋「憑證」。 按一下 [檢視憑證]，選取 [您的憑證] 索引標籤上，按一下 [匯入]，並依照提示匯入憑證。

   ![在 Firefox 上安裝憑證](./media/service-fabric-quickstart-containers-linux/install-cert-firefox.png)

## <a name="deploy-the-service-fabric-application"></a>部署 Service Fabric 應用程式

1. 在 Cloud Shell 中，使用 CLI 連線至 Azure 中的 Service Fabric 叢集。 此端點是叢集的管理端點。 您在上一節中建立了 PEM 檔案。 (對於合作對象叢集，您可以從 [讀我檔案] 頁面上的指示中，複製您的 PEM 檔案和管理端點適用的特定命令。)

    ```bash
    sfctl cluster select --endpoint https://linh1x87d1d.westus.cloudapp.azure.com:19080 --pem party-cluster-1277863181-client-cert.pem --no-verify
    ```

2. 使用安裝指令碼將投票應用程式定義複製到叢集、註冊應用程式類型，以及建立應用程式的執行個體。

    ```bash
    ./install.sh
    ```

3. 開啟網頁瀏覽器，並瀏覽至叢集的 Service Fabric Explorer 端點。 此端點具有下列格式：**https://\<my-azure-service-fabric-cluster-url>:19080/Explorer**；例如 `https://linh1x87d1d.westus.cloudapp.azure.com:19080/Explorer`。 </br>(對於合作對象叢集，您可以在 [歡迎] 頁面上找到叢集的 Service Fabric Explorer 端點。)

4. 展開 [應用程式] 節點，您可以看到投票應用程式類型和您建立的執行個體現在有一個項目。

    ![Service Fabric Explorer][sfx]

5. 若要連線至執行中的容器，請開啟網頁瀏覽器，並瀏覽至叢集的 URL，例如 `http://linh1x87d1d.westus.cloudapp.azure.com:80`。 您應會在瀏覽器中看到投票應用程式。

    ![投票應用程式網頁][quickstartpic]

> [!NOTE]
> 您也可以使用 Docker Compose 來部署 Service Fabric 應用程式。 例如，您可以使用 Docker Compose 搭配下列命令，在叢集上部署及安裝應用程式。
>  ```bash
> sfctl compose create --deployment-name TestApp --file-path ../docker-compose.yml
> ```

## <a name="fail-over-a-container-in-a-cluster"></a>容錯移轉叢集中的容器

Service Fabric 可確保如果發生失敗，容器執行個體會自動移至叢集中的其他節點。 您也可以手動清空容器的節點，並將它們依正常程序移至叢集中的其他節點。 Service Fabric 提供數種方式讓您調整服務。 在下列步驟中，您會使用 Service Fabric Explorer。

若要容錯移轉前端容器，請執行下列步驟：

1. 在您的叢集中開啟 Service Fabric Explorer，例如 `https://linh1x87d1d.westus.cloudapp.azure.com:19080/Explorer`。
2. 按一下樹狀檢視中的 **fabric:/Voting/azurevotefront** 節點，然後展開分割區節點 (以 GUID 表示)。 請注意樹狀檢視中的節點名稱，其中顯示哪些節點上的容器目前正在執行，例如 `_nodetype_4`。
3. 展開樹狀檢視中的 [節點] 節點。 按一下正在執行容器之節點旁邊的省略符號 (...)。
4. 選擇 [重新啟動] 以重新啟動節點並確認重新啟動動作。 重新啟動會造成容器容錯移轉至叢集中的其他節點。

    ![Service Fabric Explorer 中的節點檢視][sfxquickstartshownodetype]

## <a name="scale-applications-and-services-in-a-cluster"></a>調整叢集中的應用程式和服務

您可以在整個叢集內輕鬆地調整 Service Fabric 服務，以符合服務的負載。 您可以藉由變更叢集中執行的執行個體數目來調整服務。

若要調整 Web 前端服務，請執行下列步驟：

1. 在您的叢集中開啟 Service Fabric Explorer，例如 `https://linh1x87d1d.westus.cloudapp.azure.com:19080`。
2. 按一下樹狀檢視中 **fabric:/Voting/azurevotefront** 節點旁邊的省略符號 (三個點)，然後選擇 [調整服務]。

    ![Service Fabric Explorer 調整服務啟動][containersquickstartscale]

    您現在可以選擇調整 Web 前端服務的執行個體數目。

3. 將數字變更為 **2**，然後按一下 [調整服務]。
4. 按一下樹狀檢視中的 **fabric:/Voting/azurevotefront** 節點，然後展開分割區節點 (以 GUID 表示)。

    ![Service Fabric Explorer 調整服務完成][containersquickstartscaledone]

    您現在可以看到服務有兩個執行個體。 在樹狀檢視中，您可看到執行個體在哪些節點上執行。

藉由這項簡單的管理工作，您已讓前端服務可用來處理使用者負載的資源倍增。 請務必了解，您不需要多個服務執行個體，就能讓服務確實可靠地執行。 如果服務失敗，Service Fabric 可確保會有新的服務執行個體在叢集中執行。

## <a name="clean-up-resources"></a>清除資源

1. 使用範本中提供的解除安裝指令碼 (uninstall.sh)，刪除叢集中的應用程式執行個體並取消註冊應用程式類型。 此指令碼會花一些時間來清除執行個體，因此您不應在此指令碼之後立即執行安裝指令碼。 您可以使用 Service Fabric Explorer 來判斷何時已移除執行個體以及取消註冊應用程式類型。

    ```bash
    ./uninstall.sh
    ```

2. 如果您已完成叢集的處理，可以從憑證存放區中移除憑證。 例如︰
   - 在 Windows 上：使用[憑證 MMC 嵌入式管理單元](https://docs.microsoft.com/dotnet/framework/wcf/feature-details/how-to-view-certificates-with-the-mmc-snap-in)。 在新增嵌入式管理單元時，請務必選取 [我的使用者帳戶]。 瀏覽至 `Certificates - Current User\Personal\Certificates` 並移除憑證。
   - 在 Mac 上：使用 Keychain 應用程式。
   - 在 Ubuntu 上：依照您用來檢視憑證的步驟，將憑證移除。

3. 如果您不想繼續使用 Cloud Shell，可以刪除與其相關聯的儲存體帳戶，以避免產生相關費用。 關閉 Cloud Shell 工作階段。 在 Azure 入口網站中，按一下與 Cloud Shell 相關聯的儲存體帳戶，然後按一下頁面頂端的 [刪除]，並回應提示。

## <a name="next-steps"></a>後續步驟

在此快速入門中，您已在 Azure 中將 Linux 容器應用程式部署至 Service Fabric 叢集、對應用程式執行容錯移轉，並調整叢集中的應用程式。 若要深入了解如何在 Service Fabric 中使用 Linux 容器，請繼續進行教學課程以了解 Linux 容器應用程式。

> [!div class="nextstepaction"]
> [建立 Linux 容器應用程式](./service-fabric-tutorial-create-container-images.md)

[sfx]: ./media/service-fabric-quickstart-containers-linux/containersquickstartappinstance.png
[quickstartpic]: ./media/service-fabric-quickstart-containers-linux/votingapp.png
[sfxquickstartshownodetype]:  ./media/service-fabric-quickstart-containers-linux/containersquickstartrestart.png
[containersquickstartscale]: ./media/service-fabric-quickstart-containers-linux/containersquickstartscale.png
[containersquickstartscaledone]: ./media/service-fabric-quickstart-containers-linux/containersquickstartscaledone.png
