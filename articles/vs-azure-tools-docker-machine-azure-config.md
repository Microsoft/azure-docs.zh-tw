---
title: 用 Docker 電腦在 Azure 中建立 Docker 主機 | Microsoft Docs
description: 說明如何使用 Docker 電腦在 Azure 中建立 Docker 主機。
services: azure-container-service
documentationcenter: na
author: mlearned
manager: douge
editor: ''
ms.assetid: 7a3ff6e1-fa93-4a62-b524-ab182d2fea08
ms.service: multiple
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: multiple
ms.date: 06/08/2016
ms.author: mlearned
ms.openlocfilehash: 766d327a87ed13e04166d71c3d9ae0a1e7a66d19
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23124496"
---
# <a name="create-docker-hosts-in-azure-with-docker-machine"></a>使用 Docker-Machine 在 Azure 中建立 Docker 主機
執行 [Docker](https://www.docker.com/) 容器時需要執行 Docker 精靈的主機 VM。
本主題說明如何使用 [docker-machine](https://docs.docker.com/machine/) 命令，來建立在 Azure 中執行並使用 Docker 精靈所設定的新 Linux VM。 

**附註：** 

* *本文是依據 docker-machine 0.9.0-rc2 版或更新版本*
* *在不久的未來，將透過 docker-machine 支援 Windows Containers*

## <a name="create-vms-with-docker-machine"></a>使用 Docker 電腦建立 VM
搭配使用 `docker-machine create` 命令與 `azure` 驅動程式，在 Azure 中建立 Docker 主機 VM。 

Azure 驅動程式需要您的訂用帳戶識別碼。 您可以使用 [Azure CLI](cli-install-nodejs.md) 或 [Azure 入口網站](https://portal.azure.com)來擷取您的「Azure 訂用帳戶」。 

**使用 Azure 入口網站**

* 從左導覽頁面中選取 [訂用帳戶]，並複製訂用帳戶識別碼。

**使用 Azure CLI**

* 輸入 ```azure account list``` ，並複製訂用帳戶識別碼。

輸入 `docker-machine create --driver azure` 以查看選項和其預設值。
您也可以查看 [Docker Azure 驅動程式文件](https://docs.docker.com/machine/drivers/azure/) ，以取得詳細資訊。 

下列範例採用[預設值](https://github.com/docker/machine/blob/master/drivers/azure/azure.go#L22)，但會視需要設定下列值： 

* 與所產生的公用 IP 位址和憑證關聯之名稱的 azure-dns。 這是虛擬機器的 DNS 名稱。 接著，VM 就可以安全地停止、釋出動態 IP，還能夠在 VM 以新的 IP 重新啟動之後重新連線。 該區域必須有唯一的名稱前置詞 UNIQUE_DNSNAME_PREFIX.westus.cloudapp.azure.com。
* VM 上用以進行對外網際網路存取的開啟連接埠 80
* 用以使用更快速進階儲存體的 VM 大小
* 用來作為 VM 磁碟的進階儲存體

```
docker-machine create -d azure --azure-subscription-id <Your AZURE_SUBSCRIPTION_ID> --azure-dns <Your UNIQUE_DNSNAME_PREFIX> --azure-open-port 80 --azure-size Standard_DS1_v2 --azure-storage-type "Premium_LRS" mydockerhost 
```

## <a name="choose-a-docker-host-with-docker-machine"></a>使用 docker-machine 選擇 Docker 主機
docker-machine 中有您主機的項目之後，即可在執行 docker 命令時設定預設主機。

## <a name="using-powershell"></a>使用 PowerShell
```powershell
docker-machine env MyDockerHost | Invoke-Expression 
```

## <a name="using-bash"></a>使用 Bash
```bash
eval $(docker-machine env MyDockerHost)
```

您現在可以對指定的主機執行 docker 命令

```
docker ps
docker info
```

## <a name="run-a-container"></a>執行容器
設定主機之後，您現在可以執行簡單的 Web 伺服器，來測試是否已正確設定主機。
我們在此使用標準 nginx 映像，指定它應該接聽連接埠 80，而且如果主機 VM 重新啟動，則容器也會重新啟動 (`--restart=always`)。 

```bash
docker run -d -p 80:80 --restart=always nginx
```

輸出應該類似下面這樣：

```
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
efd26ecc9548: Pull complete
a3ed95caeb02: Pull complete
83f52fbfa5f8: Pull complete
fa664caa1402: Pull complete
Digest: sha256:12127e07a75bda1022fbd4ea231f5527a1899aad4679e3940482db3b57383b1d
Status: Downloaded newer image for nginx:latest
25942c35d86fe43c688d0c03ad478f14cc9c16913b0e1c2971cb32eb4d0ab721
```

## <a name="test-the-container"></a>測試容器
使用 `docker ps`檢查執行中容器：

```bash
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                         NAMES
d5b78f27b335        nginx               "nginx -g 'daemon off"   5 minutes ago       Up 5 minutes        0.0.0.0:80->80/tcp, 443/tcp   goofy_mahavira
```

若要查看執行中的容器，請輸入 `docker-machine ip <VM name>`，找出要在瀏覽器中輸入的 IP 位址：

```
PS C:\> docker-machine ip MyDockerHost
191.237.46.90
```

![執行 ngnix 容器](./media/vs-azure-tools-docker-machine-azure-config/nginxsuccess.png)

## <a name="summary"></a>總結
docker-machine 可讓您在 Azure 中輕鬆地佈建 docker 主機，以進行個別 docker 主機驗證。
如需實際執行裝載容器，請參閱 [Azure Container Service](http://aka.ms/AzureContainerService)

若要使用 Visual Studio 開發 .NET Core 應用程式，請參閱 [Docker Tools for Visual Studio](http://aka.ms/DockerToolsForVS)

