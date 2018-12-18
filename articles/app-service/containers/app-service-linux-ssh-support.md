---
title: Linux 上的 Azure App Service 支援 SSH | Microsoft Docs
description: 了解如何使用 SSH 搭配 Linux 上的 Azure App Service。
keywords: azure app service, web 應用程式, linux, oss
services: app-service
documentationcenter: ''
author: wesmc7777
manager: cfowler
editor: ''
ms.assetid: 66f9988f-8ffa-414a-9137-3a9b15a5573c
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/25/2017
ms.author: wesmc
ms.openlocfilehash: 631933647e27428349fc1efeb17f62f4614f7f64
ms.sourcegitcommit: 42405ab963df3101ee2a9b26e54240ffa689f140
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47423301"
---
# <a name="ssh-support-for-azure-app-service-on-linux"></a>Linux 上的 Azure App Service 支援 SSH

[安全殼層 (SSH)](https://wikipedia.org/wiki/Secure_Shell) 通常用於從命令列終端機，遠端執行系統管理命令。 Linux 上的 App Service 會利用每個用於新 Web 應用程式的「執行階段堆疊」的內建 Docker 映像，來提供應用程式容器內的 SSH 支援。 

![執行階段堆疊](./media/app-service-linux-ssh-support/app-service-linux-runtime-stack.png)

對於自訂 Docker 映像，藉由在自訂映像中設定 SSH 伺服器。

您也可以使用 SSH 和 SFTP，直接從本機開發電腦連線到容器。

## <a name="open-ssh-session-in-browser"></a>在瀏覽器中開啟 SSH 工作階段

若要進行容器的 SSH 用戶端連線，您的應用程式應在執行中。

在瀏覽器中貼入下列 URL，並以您的應用程式名稱取代 \<app_name>：

```
https://<app_name>.scm.azurewebsites.net/webssh/host
```

如果您尚未經過驗證，必須向您的 Azure 訂用帳戶進行驗證才能連線。 驗證之後，您會看到瀏覽器中的殼層，您可以在其中執行您容器內的命令。

![SSH 連線](./media/app-service-linux-ssh-support/app-service-linux-ssh-connection.png)

## <a name="use-ssh-support-with-custom-docker-images"></a>使用 SSH 支援搭配自訂 Docker 映像

為使自訂 Docker 映像支援容器與 Azure 入口網站之用戶端的 SSH 通訊，請針對 Docker 映像執行下列步驟。

這些步驟會顯示在 Azure App Service 存放庫中，作為[範例](https://github.com/Azure-App-Service/node/blob/master/6.9.3/)。

1. 將 [`RUN`指示](https://docs.docker.com/engine/reference/builder/#run)中的 `openssh-server` 安裝納入映像的 Dockerfile，並將根帳戶的密碼設為 `"Docker!"`。

    > [!NOTE]
    > 此設定不允許容器的外部連線。 只可透過使用發佈認證進行驗證的 Kudu / SCM 網站存取 SSH。

    ```docker
    # ------------------------
    # SSH Server support
    # ------------------------
    RUN apt-get update \
        && apt-get install -y --no-install-recommends openssh-server \
        && echo "root:Docker!" | chpasswd
    ```

1. 將 [`COPY` 指示](https://docs.docker.com/engine/reference/builder/#copy)新增至 Dockerfile，以將 [sshd_config](http://man.openbsd.org/sshd_config) 檔案複製到 */etc/ssh/* 目錄。 組態檔需根據[這裡](https://github.com/Azure-App-Service/node/blob/master/8.2.1/sshd_config)的 Azure-App-Service GitHub 存放庫中的 sshd_config 檔案。

    > [!NOTE]
    > sshd_config 檔案必須包含下列項目，否則無法連線︰ 
    > * `Ciphers` 必須至少包含下列其中一個：`aes128-cbc,3des-cbc,aes256-cbc`。
    > * `MACs` 必須至少包含下列其中一個：`hmac-sha1,hmac-sha1-96`。

    ```docker
    COPY sshd_config /etc/ssh/
    ```

1. 針對 Dockerfile，請納入 [`EXPOSE` 指示](https://docs.docker.com/engine/reference/builder/#expose) 中的連接埠 2222。 雖然已知根密碼，但無法從網際網路存取連接埠 2222。 它是僅供內部使用的連接埠，只有私人虛擬網路之橋接網路內的容器可以存取。

    ```docker
    EXPOSE 2222 80
    ```

1. 請務必使用殼層指令碼來啟動 SSH 服務 (請參閱位於 [init_container.sh](https://github.com/Azure-App-Service/node/blob/master/6.9.3/startup/init_container.sh) 的範例)。

    ```bash
    #!/bin/bash
    service ssh start
    ```

Dockerfile 會使用 [`ENTRYPOINT` 指示](https://docs.docker.com/engine/reference/builder/#entrypoint)來執行指令碼。

    ```docker
    COPY init_container.sh /opt/startup
    ...
    RUN chmod 755 /opt/startup/init_container.sh
    ...
    ENTRYPOINT ["/opt/startup/init_container.sh"]
    ```

## <a name="open-ssh-session-from-remote-shell"></a>從遠端殼層開啟 SSH 工作階段

> [!NOTE]
> 此功能目前為預覽狀態。
>

使用 TCP 通道，您可以透過已驗證的 WebSocket 連線，在您的開發電腦與「用於容器的 Web App」之間建立網路連線。 它可讓您從所選的用戶端，開啟您的容器在 App Service 中執行的 SSH 工作階段。

若要開始，您必須安裝 [Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest)。 若要查看未安裝 Azure CLI 時的運作方式，請開啟 [Azure Cloud Shell](../../cloud-shell/overview.md)。 

執行 [az extension add](/cli/azure/extension?view=azure-cli-latest#az-extension-add)，以新增最新的 App Service 擴充功能：

```azurecli-interactive
az extension add --name webapp
```

如果您之前已經執行 `az extension add`，請改為執行 [az extension update](/cli/azure/extension?view=azure-cli-latest#az-extension-update)：

```azurecli-interactive
az extension update --name webapp
```

使用 [az webapp remote-connection create](/cli/azure/ext/webapp/webapp/remote-connection?view=azure-cli-latest#ext-webapp-az-webapp-remote-connection-create) 命令，開啟您應用程式的遠端連線。 為您的應用程式指定 _\<subscription\_id>_、_\<group\_name>_ 和 \_<app\_name>_。

```azurecli-interactive
az webapp remote-connection create --subscription <subscription_id> --resource-group <group_name> -n <app_name> &
```

> [!TIP]
> 命令尾端的 `&` 只是為了方便您使用 Cloud Shell。 它會在背景執行程序，以便您在相同的殼層中執行下一個命令。

命令輸出會為您提供開啟 SSH 工作階段所需的資訊。

```
Port 21382 is open
SSH is available { username: root, password: Docker! }
Start your favorite client and connect to port 21382
```

使用本機連接埠，以您所選的用戶端開啟包含您的容器的 SSH 工作階段。 下列範例會使用預設 [ssh](https://ss64.com/bash/ssh.html) 命令：

```azurecli-interactive
ssh root@127.0.0.1 -p <port>
```

在出現提示時，輸入 `yes` 繼續連線。 系統會接著提示您輸入密碼。 使用稍早所示的 `Docker!`。

```
Warning: Permanently added '[127.0.0.1]:21382' (ECDSA) to the list of known hosts.
root@127.0.0.1's password:
```

驗證後，您應會看到工作階段歡迎畫面。

```
  _____
  /  _  \ __________ _________   ____
 /  /_\  \___   /  |  \_  __ \_/ __ \
/    |    \/    /|  |  /|  | \/\  ___/
\____|__  /_____ \____/ |__|    \___  >
        \/      \/                  \/
A P P   S E R V I C E   O N   L I N U X

0e690efa93e2:~#
```

您現在已連線到您的連接器。 

嘗試執行 [top](https://ss64.com/bash/top.html) 命令。 您應能夠在程序清單中看到您應用程式的程序。 在下列範例輸出中，它是具有 `PID 263` 的程序。

```
Mem: 1578756K used, 127032K free, 8744K shrd, 201592K buff, 341348K cached
CPU:   3% usr   3% sys   0% nic  92% idle   0% io   0% irq   0% sirq
Load average: 0.07 0.04 0.08 4/765 45738
  PID  PPID USER     STAT   VSZ %VSZ CPU %CPU COMMAND
    1     0 root     S     1528   0%   0   0% /sbin/init
  235     1 root     S     632m  38%   0   0% PM2 v2.10.3: God Daemon (/root/.pm2)
  263   235 root     S     630m  38%   0   0% node /home/site/wwwroot/app.js
  482   291 root     S     7368   0%   0   0% sshd: root@pts/0
45513   291 root     S     7356   0%   0   0% sshd: root@pts/1
  291     1 root     S     7324   0%   0   0% /usr/sbin/sshd
  490   482 root     S     1540   0%   0   0% -ash
45539 45513 root     S     1540   0%   0   0% -ash
45678 45539 root     R     1536   0%   0   0% top
45733     1 root     Z        0   0%   0   0% [init]
45734     1 root     Z        0   0%   0   0% [init]
45735     1 root     Z        0   0%   0   0% [init]
45736     1 root     Z        0   0%   0   0% [init]
45737     1 root     Z        0   0%   0   0% [init]
45738     1 root     Z        0   0%   0   0% [init]
```

## <a name="next-steps"></a>後續步驟

您可以在 [Azure 論壇](https://social.msdn.microsoft.com/forums/azure/home?forum=windowsazurewebsitespreview)張貼問題和疑難。

如需「用於容器的 Web App」的詳細資訊，請參閱：

* [簡介從 VS Code 在 Azure App Service 上進行 Node.js 應用程式的遠端偵錯](https://medium.com/@auchenberg/introducing-remote-debugging-of-node-js-apps-on-azure-app-service-from-vs-code-in-public-preview-9b8d83a6e1f0)
* [如何針對用於容器的 Web 應用程式使用自訂 Docker 映像](quickstart-docker-go.md)
* [在 Linux 上的 Azure App Service 中使用 .NET Core](quickstart-dotnetcore.md)
* [在 Linux 上的 Azure App Service 中使用 Ruby](quickstart-ruby.md)
* [Azure App Service Web App for Containers 常見問題集](app-service-linux-faq.md)
