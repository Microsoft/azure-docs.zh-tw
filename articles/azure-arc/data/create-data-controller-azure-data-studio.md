---
title: 在 Azure Data Studio 中建立資料控制器
description: 在 Azure Data Studio 中建立資料控制器
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 12/09/2020
ms.topic: how-to
ms.openlocfilehash: 22ad2d65710a3fc149f5a83fb511244ac3be2203
ms.sourcegitcommit: 78ecfbc831405e8d0f932c9aafcdf59589f81978
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2021
ms.locfileid: "98733234"
---
# <a name="create-data-controller-in-azure-data-studio"></a>在 Azure Data Studio 中建立資料控制器

您可以使用 [部署嚮導] 和 [筆記本] Azure Data Studio 建立資料控制站。

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="prerequisites"></a>Prerequisites

- 您需要存取 Kubernetes 叢集，並將您的 kubeconfig 檔案設定為指向您想要部署的 Kubernetes 叢集。
- 您必須 [安裝用戶端工具](install-client-tools.md) ，包括 **Azure Data Studio** Azure Data Studio 擴充功能 **Azure Arc** 和 **[!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)]** 。
- 您必須在 Azure Data Studio 中登入 Azure。  若要這樣做：輸入 CTRL/Command + SHIFT + P 來開啟命令文字視窗，然後輸入 **Azure**。  選擇 **Azure：登入**。   在面板中，按一下右上方的 + 圖示以新增 Azure 帳戶。

## <a name="use-the-deployment-wizard-to-create-azure-arc-data-controller"></a>使用部署嚮導來建立 Azure Arc 資料控制器

遵循下列步驟，使用 [部署嚮導] 建立 Azure Arc 資料控制器。

1. 在 Azure Data Studio 中，按一下左側導覽上的 [連接] 索引標籤。
2. 按一下 [連線] 面板頂端的 [ **...** ] 按鈕，然後選擇 [**新增部署 ...** ]。
3. 在 [新增部署嚮導] 中，選擇 [ **Azure Arc 資料控制器**]，然後按一下底部的 [ **選取** ] 按鈕。
4. 確定有必要的工具可供使用，並符合所需的版本。 **按 [下一步]**。
5. 使用預設 kubeconfig 檔案，或選取另一個檔案。  按一下 [下一步] 。
6. 選擇 Kubernetes 叢集內容。 按一下 [下一步] 。
7. 根據您的目標 Kubernetes 叢集，選擇部署設定檔。 **按 [下一步]**。
8. 如果您使用 Azure Red Hat OpenShift 或 Red Hat OpenShift 容器平臺，請套用安全性內容條件約束。 遵循在 [OpenShift 上針對 Azure Arc 啟用的資料服務套用安全性內容條件約束](how-to-apply-security-context-constraint.md)的指示。

   >[!IMPORTANT]
   >在 Azure Red Hat OpenShift 或 Red Hat OpenShift 容器平臺上，您必須在建立資料控制器之前套用安全性內容條件約束。

1. 選擇所需的訂用帳戶和資源群組。
1. 選取 Azure 位置。
   
   此處選取的 Azure 位置是 Azure 中的位置，在此位置中會儲存資料控制器和其所管理之資料庫實例的相關 *中繼資料* 。 在您的 Kubernetes 叢集中，資料控制器和資料庫實例實際上會 crewted。

10. 選取適當的連接模式。 深入瞭解 [連接模式](https://docs.microsoft.com/azure/azure-arc/data/connectivity)。 **按 [下一步]**。

    如果您選取 [直接連線模式]，則需要服務主體認證，如 [建立服務主體](upload-metrics-and-logs-to-azure-monitor.md#create-service-principal)中所述。

11. 輸入資料控制站的名稱，以及將建立資料控制器的命名空間。

    資料控制器和命名空間名稱將用來在 Kubernetes 叢集中建立自訂資源，因此必須符合 Kubernetes 的 [命名慣例](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names)。
    
    如果命名空間已存在，則會使用此命名空間（如果命名空間尚未包含其他 Kubernetes 物件-pod 等）。 如果命名空間不存在，將會嘗試建立命名空間。  在 Kubernetes 叢集中建立命名空間需要 Kubernetes 叢集系統管理員許可權。  如果您沒有 Kubernetes 叢集系統管理員許可權，請要求您的 Kubernetes 叢集系統管理員 [使用 Kubernetes 原生工具來執行建立資料控制](./create-data-controller-using-kubernetes-native-tools.md) 站中的前幾個步驟，而 Kubernetes 管理員必須先執行這些步驟，才能完成此嚮導。


12. 選取將在其中部署資料控制器的儲存類別。 
13.  輸入使用者名稱和密碼，並確認資料控制器系統管理員使用者帳戶的密碼。 按一下 [下一步] 。

14. 檢查部署設定。
15. 按一下 [ **部署** ]，將所需的設定或 **腳本部署至筆記本** ，以檢查部署指示或進行任何必要的變更，例如儲存類別名稱或服務類型。 按一下筆記本頂端的 [ **全部執行** ]。

## <a name="monitoring-the-creation-status"></a>監視建立狀態

建立控制器需要幾分鐘的時間才能完成。 您可以使用下列命令來監視另一個終端機視窗中的進度：

> [!NOTE]
>  下列範例命令假設您已建立名為 ' arc ' 的資料控制器和 Kubernetes 命名空間。  如果您使用不同的命名空間/資料控制器名稱，您可以將 ' arc ' 取代為您的名稱。

```console
kubectl get datacontroller/arc --namespace arc
```

```console
kubectl get pods --namespace arc
```

您也可以執行如下所示的命令，檢查任何特定 pod 的建立狀態。  這特別適用于針對任何問題進行疑難排解。

```console
kubectl describe po/<pod name> --namespace arc

#Example:
#kubectl describe po/control-2g7bl --namespace arc
```

## <a name="troubleshooting-creation-problems"></a>針對建立問題進行疑難排解

如果您在建立麻煩很快找上門時遇到任何問題，請參閱 [疑難排解指南](troubleshoot-guide.md)。