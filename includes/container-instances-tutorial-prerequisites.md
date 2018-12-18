---
title: 包含檔案
description: 包含檔案
services: container-instances
author: dlepow
ms.service: container-instances
ms.topic: include
ms.date: 03/20/2018
ms.author: danlep
ms.custom: include file
ms.openlocfilehash: da63a5418ab94623f6ce3c9f35a085dd8b198d1a
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48858089"
---
您必須滿足下列需求，才能完成本教學課程：

**Azure CLI**：您必須在本機電腦上安裝 Azure CLI 2.0.29 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI][azure-cli-install]。

**Docker**：本教學課程假設使用者對核心 Docker 概念有基本認識，像是容器、容器映像和基本 `docker` 命令。 如需 Docker 和容器基本概念的入門，請參閱 [Docker 概觀][docker-get-started]。

**Docker 引擎**：若要完成本教學課程，您需要在本機安裝 Docker 引擎。 Docker 提供可在 [macOS][docker-mac]、[Windows][docker-windows] 和 [Linux][docker-linux] 上設定 Docker 環境的套件。

> [!IMPORTANT]
> 因為 Azure Cloud shell 不包含 Docker 精靈，所以您「必須」在「本機電腦」上安裝 Azure CLI 和 Docker 引擎，才能完成本教學課程。 您無法在此教學課程中使用 Azure Cloud Shell。

<!-- LINKS - External -->
[docker-get-started]: https://docs.docker.com/engine/docker-overview/
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- LINKS - Internal -->
[azure-cli-install]: /cli/azure/install-azure-cli
