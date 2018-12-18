---
title: 安裝 DC/OS CLI | Microsoft Docs
description: 安裝 DC/OS CLI。
services: container-service
documentationcenter: ''
author: rgardler
manager: timlt
editor: ''
tags: acs, azure-container-service
keywords: 容器, 微服務, DC/OS, Azure
ms.service: container-service
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/10/2016
ms.author: rogardle
ms.openlocfilehash: a8ea47f158c0d666340815d2e039995c7483257f
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "22877253"
---
> [!NOTE]
> 這適用於使用 DC/OS 型 ACS 叢集時。 若是 Swarm 型 ACS 叢集，就不需要執行此工作。
> 
> 

首先， [連線到 DC/OS 型 ACS 叢集](../articles/container-service/container-service-connect.md)。 完成後，就可以使用下列命令在用戶端電腦上安裝 DC/OS CLI：

```bash
sudo pip install virtualenv
mkdir dcos && cd dcos
wget https://raw.githubusercontent.com/mesosphere/dcos-cli/master/bin/install/install-optout-dcos-cli.sh
chmod +x install-optout-dcos-cli.sh
./install-optout-dcos-cli.sh . http://localhost --add-path yes
```

如果您使用舊版 Python，您可能會注意到一些 "InsecurePlatformWarnings"。 您可以放心地忽略這些警告。

若要直接開始進行而不重新啟動殼層，請執行︰

```bash
source ~/.bashrc
```

當您啟動新殼層時，就不需要此步驟。

現在，您可以確認 CLI 是否已安裝：

```bash
dcos --help
```