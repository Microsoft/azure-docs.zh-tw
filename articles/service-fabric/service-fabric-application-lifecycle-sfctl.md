---
title: 使用 Azure Service Fabric CLI 來管理 Azure Service Fabric 應用程式 (sfctl)
description: 了解如何使用 Azure Service Fabric CLI 在 Azure Service Fabric 叢集中部署和移除應用程式
services: service-fabric
author: Christina-Kang
manager: timlt
ms.service: service-fabric
ms.topic: conceptual
ms.date: 07/31/2018
ms.author: bikang
ms.openlocfilehash: 2e71996e22fee34b29139fdf19764c47616beb1d
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39492744"
---
# <a name="manage-an-azure-service-fabric-application-by-using-azure-service-fabric-cli-sfctl"></a>使用 Azure Service Fabric CLI 來管理 Azure Service Fabric 應用程式 (sfctl)

了解如何建立和刪除在 Azure Service Fabric 叢集中執行的應用程式。

## <a name="prerequisites"></a>必要條件

* 安裝 Service Fabric CLI。 然後選取 Service Fabric 叢集。 如需詳細資訊，請參閱[開始使用 Service Fabric CLI](service-fabric-cli.md)。

* 備妥 Service Fabric 應用程式封裝以供部署。 如需有關如何撰寫及封裝應用程式的詳細資訊，請參閱 [Service Fabric 應用程式模型](service-fabric-application-model.md)。

## <a name="overview"></a>概觀

若要部署新應用程式，請完成下列步驟：

1. 將應用程式封裝上傳到 Service Fabric 映像存放區。
2. 佈建應用程式類型。
3. 刪除映像存放區內容。
4. 指定和建立應用程式。
5. 指定和建立服務。

若要移除現有的應用程式，請完成下列步驟：

1. 刪除應用程式。
2. 將已關聯的應用程式類型解除佈建。

## <a name="deploy-a-new-application"></a>部署新應用程式

若要部署新應用程式，請完成下列工作：

### <a name="upload-a-new-application-package-to-the-image-store"></a>將新應用程式套件上傳到映像存放區

建立應用程式之前，將應用程式封裝上傳到 Service Fabric 映像存放區。

例如，如果您的應用程式封裝在 `app_package_dir` 目錄中，使用下列命令上傳目錄：

```azurecli
sfctl application upload --path ~/app_package_dir
```

針對大型應用程式套件，您可以指定 `--show-progress` 選項以顯示上傳進度。

### <a name="provision-the-application-type"></a>佈建應用程式類型

上傳完成時，佈建應用程式。 若要佈建應用程式，請使用下列命令：

```azurecli
sfctl application provision --application-type-build-path app_package_dir
```

值 `application-type-build-path` 是您上傳應用程式封裝的目錄名稱。

### <a name="delete-the-application-package"></a>刪除應用程式封裝

建議您在成功註冊應用程式之後，移除應用程式套件。  從映像存放區刪除應用程式套件會釋放系統資源。  保留未使用的應用程式套件會耗用磁碟儲存空間，並會導致應用程式效能問題。 

若要從映像存放區中刪除應用程式封裝，請使用下列命令：

```azurecli
sfctl store delete --content-path app_package_dir
```

`content-path` 必須是您建立應用程式時上傳的目錄名稱。

### <a name="create-an-application-from-an-application-type"></a>從應用程式類型建立應用程式

佈建應用程式之後，使用下列命令為應用程式命名並建立應用程式：

```azurecli
sfctl application create --app-name fabric:/TestApp --app-type TestAppType --app-version 1.0
```

`app-name` 是您想要用於應用程式執行個體的名稱。 您可以從先前佈建的應用程式資訊清單取得額外的參數。

應用程式名稱必須以 `fabric:/` 前置詞作為開頭。

### <a name="create-services-for-the-new-application"></a>為新的應用程式建立服務

建立應用程式之後，從該應用程式建立服務。 在下列範例中，我們會從應用程式建立新的無狀態服務。 先前佈建的應用程式封裝之中的服務資訊清單中會定義您可以從應用程式建立的服務。

```azurecli
sfctl service create --app-id TestApp --name fabric:/TestApp/TestSvc --service-type TestServiceType \
--stateless --instance-count 1 --singleton-scheme
```

## <a name="verify-application-deployment-and-health"></a>確認應用程式部署和健康情況

若要驗證所有項目的健康情況良好，請使用下列健康情況命令：

```azurecli
sfctl application list
sfctl service list --application-id TestApp
```

若要確認服務是否狀況良好，請使用類似的命令來取出服務和應用程式的健康情況：

```azurecli
sfctl application health --application-id TestApp
sfctl service health --service-id TestApp/TestSvc
```

狀況良好的服務和應用程式的 `Ok` 值是 `HealthState`。

## <a name="remove-an-existing-application"></a>移除現有的應用程式

若要移除應用程式，請完成下列工作：

### <a name="delete-the-application"></a>刪除應用程式

若要刪除應用程式，請使用下列命令：

```azurecli
sfctl application delete --application-id TestEdApp
```

### <a name="unprovision-the-application-type"></a>將應用程式類型解除佈建

刪除應用程式之後，如果不再需要應用程式，可以解除佈建應用程式類型。 若要解除佈建應用程式類型，請使用下列命令：

```azurecli
sfctl application unprovision --application-type-name TestAppType --application-type-version 1.0
```

類型名稱和類型版本必須與先前佈建的應用程式資訊清單中的名稱和版本相符。

## <a name="upgrade-application"></a>升級應用程式

建立應用程式之後，您可以重複相同的一組步驟來佈建第二個版本的應用程式。 然後，隨著 Service Fabric 應用程式升級，您可以轉換為執行第二個版本的應用程式。 如需詳細資訊，請參閱 [Service Fabric 應用程式升級](service-fabric-application-upgrade.md)上的文件。

若要執行升級，請先使用與以前相同的命令，佈建下一版的應用程式：

```azurecli
sfctl application upload --path ~/app_package_dir_2
sfctl application provision --application-type-build-path app_package_dir_2
sfctl store delete --content-path app_package_dir_2
```

然後才建議您執行受監視的自動升級，藉由執行下列命令來啟動升級：

```azurecli
sfctl application upgrade --app-id TestApp --app-version 2.0.0 --parameters "{\"test\":\"value\"}" --mode Monitored
```

升級會以指定的任何設定覆寫現有參數。 如有必要，應用程式參數應該當作引數傳遞至升級命令。 應用程式參數應該編碼為 JSON 物件。

若要擷取先前指定的任何參數，您可以使用 `sfctl application info` 命令。

當應用程式升級正在進行時，可以使用 `sfctl application upgrade-status` 命令來擷取狀態。

最後，如果升級正在進行中但是必須取消，您可以使用 `sfctl application upgrade-rollback` 來復原升級。

## <a name="next-steps"></a>後續步驟

* [Service Fabric CLI 基本概念](service-fabric-cli.md)
* [在 Linux 上開始使用 Service Fabric](service-fabric-get-started-linux.md)
* [Service Fabric 應用程式升級](service-fabric-application-upgrade.md)
