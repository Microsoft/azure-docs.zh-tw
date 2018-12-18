---
title: 使用 Ansible 搭配 Azure
description: 簡介使用 Ansible 自動佈建雲端、進行組態管理和應用程式部署。
ms.service: ansible
keywords: ansible, azure, devops, 概觀, 雲端佈建, 組態管理, 應用程式部署, ansible 模組, ansible 腳本
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.date: 09/02/2018
ms.topic: article
ms.openlocfilehash: 977fef390c0efecd47ec5e19b1a82c05e2ecfd0f
ms.sourcegitcommit: 2d961702f23e63ee63eddf52086e0c8573aec8dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44160740"
---
# <a name="ansible-with-azure"></a>Ansible 搭配 Azure

[Ansible](http://www.ansible.com) 是一項開放原始碼產品，可自動佈建雲端、進行組態管理和應用程式部署。 您可以使用 Ansible 來佈建虛擬機器、容器和網路，並完成雲端基礎結構。 此外，Ansible 還可讓您將環境中的資源部署和設定自動化。

本文提供基本概觀，說明使用 Ansible 搭配 Azure 的一些優點。

## <a name="ansible-playbooks"></a>Ansible 腳本

[Ansible 腳本](http://docs.ansible.com/ansible/latest/playbooks.html)是 Ansible 的組態、部署和協調流程語言。 腳本可以描述您需要遠端系統強制執行的原則，或是一般 IT 流程中的一組步驟。 您可以使用 YAML 來建立腳本，從而定義組態的模型或流程。

## <a name="ansible-modules"></a>Ansible 模組

Ansible 包含 [Ansible 模組](http://docs.ansible.com/ansible/latest/modules_by_category.html)的套件，可直接在遠端主機上執行，或是透過[腳本](http://docs.ansible.com/ansible/latest/playbooks.html)執行。 使用者也可以建立自己的模組。 您可以使用模組來控制系統資源 (例如服務、套件或檔案)，或是執行系統命令。

為了與 Azure 服務互動，Ansible 包含 [Ansible 雲端模組](http://docs.ansible.com/ansible/list_of_cloud_modules.html#azure)的套件，所提供的工具能讓您在 Azure 上輕鬆地建立並協調基礎結構。 

## <a name="migrate-existing-workload-to-azure"></a>將現有的工作負載移轉至 Azure

一旦使用 Ansible 來定義您的基礎結構之後，您可以套用您應用程式的腳本，讓 Azure 視需要自動調整您的環境。 

## <a name="automate-cloud-native-application-in-azure"></a>將 Azure 中的雲端原生應用程式自動化

Ansible 可讓您使用諸如 [Azure Functions](https://azure.microsoft.com//services/functions/) 和 [Kubernetes on Azure](https://azure.microsoft.com/services/container-service/kubernetes/) 等 Azure 微服務，將 Azure 中的雲端原生應用程式自動化。  

## <a name="manage-deployments-with-dynamic-inventory"></a>使用動態清查管理部署
Ansible 能夠透過其[動態清查](http://docs.ansible.com/ansible/intro_dynamic_inventory.html)功能來提取 Azure 資源中的清查。 接著，您可以將現有的 Azure 部署標記，並透過 Ansible 管理這些標記的部署。

## <a name="additional-azure-marketplace-options"></a>Azure Marketplace 的其他選項
Red Hat 提供的 [Ansible Tower](https://azuremarketplace.microsoft.com/marketplace/apps/redhat.ansible-tower) Azure Marketplace 映像能夠協助組織調整 IT 自動化，以及管理各個實體、虛擬及雲端基礎結構之間的複雜部署。 Ansible Tower 所包含的功能可因應現今的企業需求，提供額外的可見性層級、控制項、安全性與效率。 Ansible Tower 可將 Azure 和 SSH 金鑰等認證加密，以便您將作業委派給新進的員工時，能夠避免將認證公開的風險。

## <a name="ansible-module-and-version-matrix-for-azure"></a>Azure 的 Ansible 模組和版本對照表
Ansible 隨附於許多模組，這些模組可以直接在遠端主機上或透過劇本來執行。
[Ansible 模組和版本對照表](./ansible-matrix.md)會列出 Azure 的 Ansible 模組，這些模組可以佈建 Azure 雲端資源，例如虛擬機器、網路及容器服務。 

## <a name="next-steps"></a>後續步驟
- [設定 Ansible](/azure/virtual-machines/linux/ansible-install-configure?toc=%2Fen-us%2Fazure%2Fansible%2Ftoc.json&bc=%2Fen-us%2Fazure%2Fbread%2Ftoc.json)
- [建立 Linux VM](/azure/virtual-machines/linux/ansible-create-vm?toc=%2Fen-us%2Fazure%2Fansible%2Ftoc.json&bc=%2Fen-us%2Fazure%2Fbread%2Ftoc.json)
