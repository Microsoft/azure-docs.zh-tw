---
title: 使用 VirtualBox 設定 Docker 主機 | Microsoft Docs
description: 使用 Docker 機器和 VirtualBox 來設定預設 Docker 執行個體的逐步指示。
services: azure-container-service
documentationcenter: na
author: mlearned
manager: douge
editor: ''
ms.assetid: 0b1335a2-7720-42a8-8260-4e06fc00c9f6
ms.service: multiple
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: multiple
ms.date: 06/08/2016
ms.author: mlearned
ms.openlocfilehash: 7c78f27fa948c15202e83df4ed42a805a414a72e
ms.sourcegitcommit: 744747d828e1ab937b0d6df358127fcf6965f8c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "42140466"
---
# <a name="configure-a-docker-host-with-virtualbox"></a>使用 VirtualBox 設定 Docker 主機
## <a name="overview"></a>概觀
本文會逐步引導您使用 Docker 機器和 VirtualBox 來設定預設 Docker 執行個體。 如果您使用 [Docker for Windows](https://www.docker.com/products/docker-desktop)，則不需要此組態。

## <a name="prerequisites"></a>必要條件
您的電腦必須安裝下列工具。

* [Docker 工具箱](https://github.com/docker/toolbox/releases)

## <a name="configuring-the-docker-client-with-windows-powershell"></a>使用 Windows PowerShell 設定 Docker 用戶端
若要設定 Docker 用戶端，只需開啟 Windows PowerShell，然後執行下列步驟︰

1. 建立預設 Docker 主機執行個體。
   
    ```PowerShell
    docker-machine create --driver virtualbox default
    ```
2. 確認預設執行個體已完成設定且正在執行。 (您應該會看到名為「default」的執行個體正在執行。
   
    ```PowerShell
    docker-machine ls 
    ```
   
    ![docker-machine ls 輸出][0]
3. 預設為目前的主機，並設定您的 Shell。
   
    ```PowerShell
    docker-machine env default | Invoke-Expression
    ```
4. 顯示作用中的 Docker 容器。 清單應該是空的。
   
    ```PowerShell
    docker ps
    ```
   
    ![docker ps 輸出][1]

> [!NOTE]
> 每次您將開發用機器重新開機時，就需要重新啟動您的本機 Docker 主機。
> 做法是在命令提示字元發出下列命令： `docker-machine start default`。
> 
> 

[0]: ./media/vs-azure-tools-docker-setup/docker-machine-ls.png
[1]: ./media/vs-azure-tools-docker-setup/docker-ps.png
