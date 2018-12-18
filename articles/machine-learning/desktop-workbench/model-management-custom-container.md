---
title: 自訂用來部署 Azure ML 模型的容器映像 | Microsoft Docs
description: 此文章說明如何針對 Azure Machine Learning 模型自訂容器映像
services: machine-learning
author: tedway
ms.author: tedway
manager: mwinkle
ms.reviewer: mldocs, raymondl
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.date: 3/26/2018
ROBOTS: NOINDEX
ms.openlocfilehash: d4cfb7067510ec06df3319035dee5e2195cb2f9d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46997534"
---
# <a name="customize-the-container-image-used-for-azure-ml-models"></a>自訂用於 Azure ML 模型的容器映像

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 



此文章說明如何針對 Azure Machine Learning 模型自訂容器映像。  Azure ML Workbench 會使用容器來部署機器學習模型。 模型會隨著其相依性一起部署，而 Azure ML 會從模型、相依性和相關聯的檔案建置映像。

## <a name="how-to-customize-the-docker-image"></a>如何自訂 Docker 映像
使用下列選項自訂 Azure ML 部署的 Docker 映像：

1. `dependencies.yml` 檔案：若要管理可從 [PyPi]( https://pypi.python.org/pypi) 安裝的相依性，您可以使用 Workbench 專案的 `conda_dependencies.yml` 檔案，或建立您自己的檔案。 這是用來安裝可透過 pip 安裝之 Python 相依性的建議方法。

   範例 CLI 命令：
   ```azurecli
   az ml image create -n <my Image Name> --manifest-id <my Manifest ID> -c amlconfig\conda_dependencies.yml
   ```

   範例 conda_dependencies 檔案： 
   ```yaml
   name: project_environment
   dependencies:
      - python=3.5.2
      - scikit-learn
      - ipykernel=4.6.1
      
      - pip:
        - azure-ml-api-sdk==0.1.0a11
        - matplotlib
   ```
        
2. Docker 步驟檔案：您可以使用此選項，透過安裝無法從 PyPi 安裝的相依性，自訂已部署的映像。 

   該檔案應包含像是 DockerFile 的 Docker 安裝步驟。 檔案中允許下列命令： 

    RUN、ENV、ARG、LABEL、EXPOSE

   範例 CLI 命令：
   ```azurecli
   az ml image create -n <my Image Name> --manifest-id <my Manifest ID> --docker-file <myDockerStepsFileName> 
   ```

   Image、Manifest 和 Service 命令接受 docker-file 檔案旗標。

   範例 Docker 步驟檔案：
   ```docker
   # Install tools required to build the project
   RUN apt-get update && apt-get install -y git libxml2-dev
   # Install library dependencies
   RUN dep ensure -vendor-only
   ```

> [!NOTE]
> Azure ML 容器的基底映像為 Ubuntu，且無法變更。 如果您指定不同的基底映像，則將會忽略它。

## <a name="next-steps"></a>後續步驟
自訂容器映像之後，就可以將它部署到叢集以供大規模使用。  如需有關設定叢集以便部署 Web 服務的詳細資訊，請參閱[模型管理設定](deployment-setup-configuration.md)。 
