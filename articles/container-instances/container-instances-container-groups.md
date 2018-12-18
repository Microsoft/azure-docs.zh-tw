---
title: Azure Container Instances 容器群組
description: 了解容器群組在 Azure Container Instances 中的運作方式
services: container-instances
author: seanmck
manager: jeconnoc
ms.service: container-instances
ms.topic: article
ms.date: 03/20/2018
ms.author: seanmck
ms.custom: mvc
ms.openlocfilehash: cb3d8c27a82c7dfc5fd71c1c7d589e81890e5cfb
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32163294"
---
# <a name="container-groups-in-azure-container-instances"></a>Azure Container Instances 中的容器群組

在 Azure Container Instances 中，最上層的資源就是容器群組。 本文說明容器群組為何，以及這些群組能夠實現的案例類型。

## <a name="how-a-container-group-works"></a>容器群組的運作方式

容器群組是容器的集合，這些容器會排程在相同的主機電腦上。 容器群組中的容器會共用生命週期、區域網路和存放磁碟區。 容器群組類似於 [Kubernetes][kubernetes-pod] 和 [DC/OS][dcos-pod] 中的 *pod* 概念。

下圖舉例說明包含多個容器的容器群組：

![容器群組圖表][container-groups-example]

此範例群組容器：

* 已排程在單一主機電腦上。
* 受指派 DNS 名稱標籤。
* 會公開單一的公用 IP 位址，以及一個公開的連接埠。
* 包括兩個容器。 一個容器接聽連接埠 80，另一個容器接聽連接埠 5000。
* 包含兩個 Azure 檔案共用作為磁碟區掛接，且每個容器會在本機掛接其中一個共用。

> [!NOTE]
> 多容器群組目前僅限於 Linux 容器。 雖然我們致力於將所有功能帶入 Windows 容器，但是您可以在 [Azure 容器執行個體配額和區域可用性](container-instances-quotas.md)中找到目前的平台差異。

## <a name="deployment"></a>部署

容器*群組*的最小資源配置為 1 個 vCPU 和 1 GB 記憶體。 您可以使用小於 1 個 vCPU 和 1 GB 記憶體來佈建容器群組內的個別*容器*。 在容器群組內，您可以在以容器群組層級建立的限制內，將資源散發自訂為多個容器。 例如，各自包含 0.5 個 vCPU 的兩個容器位於配置 1 個 vCPU 的容器群組內。

## <a name="networking"></a>網路

容器群組會共用 IP 位址以及該 IP 位址上的連接埠命名空間。 若要讓外部用戶端能夠連線到該群組內的容器，您必須從該容器公開該 IP 位址上的連接埠。 群組內的容器會共用連接埠命名空間，因此不支援連接埠對應。 群組內的容器可以在它們已公開的連接埠上透過 localhost 彼此連線，即使這些連接埠並未在群組的 IP 位址上對外公開也是如此。

## <a name="storage"></a>儲存體

您可以指定要在容器群組內掛接的外部磁碟區。 您可以將這些磁碟區對應到群組之個別容器內的特定路徑。

## <a name="common-scenarios"></a>常見案例

如果您想要將單一功能性工作分割成少量的容器映像，多個容器的群組會很實用。 然後，這些映像就可以由不同小組傳遞，且具有個別的資源需求。

使用範例可能包括：

* 一個應用程式容器和一個記錄容器。 記錄容器收集主應用程式所輸出的記錄和計量，並將這些資料寫入到長期存放區。
* 一個應用程式容器和一個監視容器。 監視容器會定期向應用程式發出要求，以確保它會保持執行狀態並正確回應，如果沒有正確回應則引發警示。
* 一個容器提供 Web 應用程式，一個容器從原始檔控制提取最新內容。

## <a name="next-steps"></a>後續步驟

了解如何使用 Azure Resource Manager 範本來部署多個容器的容器群組：

> [!div class="nextstepaction"]
> [部署容器群組](container-instances-multi-container-group.md)

<!-- IMAGES -->
[container-groups-example]: ./media/container-instances-container-groups/container-groups-example.png

<!-- LINKS - External -->
[dcos-pod]: https://dcos.io/docs/1.10/deploying-services/pods/
[kubernetes-pod]: https://kubernetes.io/docs/concepts/workloads/pods/pod/
