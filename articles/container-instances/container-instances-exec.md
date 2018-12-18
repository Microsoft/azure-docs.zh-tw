---
title: 在 Azure 容器執行個體的執行中容器內執行命令
description: 了解如何在目前正在 Azure 容器執行個體中執行的容器內執行命令
services: container-instances
author: mmacy
manager: jeconnoc
ms.service: container-instances
ms.topic: article
ms.date: 03/30/2018
ms.author: marsma
ms.openlocfilehash: 43211f620efb16cbcd722d3d386b1bb862fc6280
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32165641"
---
# <a name="execute-a-command-in-a-running-azure-container-instance"></a>在執行中的 Azure 容器執行個體內執行命令

Azure 容器執行個體支援在執行中容器內執行命令。 在您已經啟動的容器中執行命令，在應用程式開發和疑難排解期間特別有用。 此功能的常見用法是啟動互動式殼層，以便在執行中容器內進行問題偵錯。

## <a name="run-a-command-with-azure-cli"></a>使用 Azure CLI 執行命令

在 [Azure CLI][azure-cli] 中，搭配 [az container exec][az-container-exec]，於執行中容器內執行命令：

```azurecli
az container exec --resource-group <group-name> --name <container-group-name> --exec-command "<command>"
```

例如，若要在 Nginx 容器中啟動 Bash 殼層：

```azurecli
az container exec --resource-group myResourceGroup --name mynginx --exec-command "/bin/bash"
```

在下列範例輸出中，會在執行中的 Linux 容器內啟動 Bash 殼層，以提供 `ls` 執行所在的終端機：

```console
$ az container exec --resource-group myResourceGroup --name mynginx --exec-command "/bin/bash"
root@caas-83e6c883014b427f9b277a2bba3b7b5f-708716530-2qv47:/# ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
root@caas-83e6c883014b427f9b277a2bba3b7b5f-708716530-2qv47:/# exit
exit
Bye.
```

在此範例中，會在執行中的 Nanoserver 容器內啟動命令提示字元：

```console
$ az container exec --resource-group myResourceGroup --name myiis --exec-command "cmd.exe"
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\>dir
 Volume in drive C has no label.
 Volume Serial Number is 76E0-C852

 Directory of C:\

03/23/2018  09:13 PM    <DIR>          inetpub
11/20/2016  11:32 AM             1,894 License.txt
03/23/2018  09:13 PM    <DIR>          Program Files
07/16/2016  12:09 PM    <DIR>          Program Files (x86)
03/13/2018  08:50 PM           171,616 ServiceMonitor.exe
03/23/2018  09:13 PM    <DIR>          Users
03/23/2018  09:12 PM    <DIR>          var
03/23/2018  09:22 PM    <DIR>          Windows
               2 File(s)        173,510 bytes
               6 Dir(s)  21,171,609,600 bytes free

C:\>exit
Bye.
```

## <a name="multi-container-groups"></a>多個容器群組

如果您的[容器群組](container-instances-container-groups.md)具有多個容器 (例如應用程式容器和記錄側車)，請使用 `--container-name` 指定要在其中執行命令的容器名稱。

例如，容器群組 *mynginx* 中有兩個容器，即 *nginx-app* 和 *logger*。 在 *nginx-app* 容器中啟動殼層：

```azurecli
az container exec --resource-group myResourceGroup --name mynginx --container-name nginx-app --exec-command "/bin/bash"
```

## <a name="restrictions"></a>限制

Azure 容器執行個體目前支援使用 [az container exec][az-container-exec] 啟動單一處理序，而您無法傳遞命令引數。 例如，您無法像在 `sh -c "echo FOO && echo BAR"` 中鏈結命令，也無法執行 `echo FOO`。

## <a name="next-steps"></a>後續步驟

請參閱[在 Azure 容器執行個體中進行容器和部署問題的疑難排解](container-instances-troubleshooting.md)，以了解其他疑難排解工具和常見部署問題。

<!-- LINKS - internal -->
[az-container-create]: /cli/azure/container#az-container-create
[az-container-exec]: /cli/azure/container#az-container-exec
[azure-cli]: /cli/azure