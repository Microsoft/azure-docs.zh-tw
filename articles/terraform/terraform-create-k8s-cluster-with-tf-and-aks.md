---
title: 以 Azure Kubernetes Service (AKS) 和 Terraform 建立 Kubernetes 叢集
description: 教學課程說明如何以 Azure Kubernetes Service 和 Terraform 建立 Kubernetes 叢集
services: terraform
ms.service: terraform
keywords: terraform, devops, 虛擬機器, Azure, kubernetes
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.topic: tutorial
ms.date: 09/08/2018
ms.openlocfilehash: f261c59193349d55d407e6079002b75884273e84
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46960238"
---
# <a name="create-a-kubernetes-cluster-with-azure-kubernetes-service-and-terraform"></a>以 Azure Kubernetes Service 和 Terraform 建立 Kubernetes 叢集
[Azure Kubernetes Service (AKS)](/azure/aks/) 可以管理裝載 Kubernetes 的環境；因此，您不需具備容器協調流程專業知識，就能快速、輕鬆地部署及管理容器化應用程式。 也可透過佈建、升級與依需求調整資源，以無需讓應用程式離線的方式來消除進行中作業及維護之間的界線。

在本教學課程中，您將了解如何執行下列工作，以使用 [Terraform](http://terraform.io) 和 AKS 建立 [Kubernetes](https://www.redhat.com/en/topics/containers/what-is-kubernetes) 叢集：

> [!div class="checklist"]
> * 使用 HCL (HashiCorp 語言) 定義 Kubernetes 叢集
> * 使用 Terraform 和 AKS 建立 Kubernetes 叢集
> * 使用 kubectl 工具測試 Kubernetes 叢集的可用性

## <a name="prerequisites"></a>必要條件

- **Azure 訂用帳戶**：如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。

- **設定 Terraform**：請遵循 [Terraform 和設定 Azure 的存取](/azure/virtual-machines/linux/terraform-install-configure)一文中的指示

- **Azure 服務主體**：請遵循[使用 Azure CLI 建立 Azure 服務主體](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest#create-the-service-principal)一文中**建立服務主體**一節的指示。 記下 appId、displayName、密碼和租用戶欄位內的值。

## <a name="create-the-directory-structure"></a>建立目錄結構
第一個步驟是先建立目錄，用來存放供練習使用的 Terraform 組態檔。

1. 瀏覽至 [Azure 入口網站](http://portal.azure.com)。

1. 開啟 [Azure Cloud Shell](/azure/cloud-shell/overview)。 如果您先前未選取環境，請選取 **Bash** 作為您的環境。

    ![Cloud Shell 提示](./media/terraform-create-k8s-cluster-with-tf-and-aks/azure-portal-cloud-shell-button-min.png)

1. 切換至 `clouddrive` 目錄。

    ```bash
    cd clouddrive
    ```

1. 建立名為 `terraform-aks-k8s` 的目錄。

    ```bash
    mkdir terraform-aks-k8s
    ```

1. 將目錄變更為新的目錄：

    ```bash
    cd terraform-aks-k8s
    ```

## <a name="declare-the-azure-provider"></a>宣告 Azure 提供者
建立宣告 Azure 提供者的 Terraform 組態檔。

1. 在 Cloud Shell 中建立名稱為 `main.tf` 的檔案。

    ```bash
    vi main.tf
    ```

1. 選取 I 鍵輸入插入模式。

1. 將下列程式碼貼到編輯器中：

    ```JSON
    provider "azurerm" {
        version = "~>1.5"
    }

    terraform {
        backend "azurerm" {}
    }

    ```

1. 選取 **Esc** 鍵結束插入模式。

1. 輸入下列命令來儲存檔案及結束 vi 編輯器：

    ```bash
    :wq
    ```

## <a name="define-a-kubernetes-cluster"></a>定義 Kubernetes 叢集
建立宣告 Kubernetes 叢集資源的 Terraform 組態檔。

1. 在 Cloud Shell 中建立名稱為 `k8s.tf` 的檔案。

    ```bash
    vi k8s.tf
    ```

1. 選取 I 鍵輸入插入模式。

1. 將下列程式碼貼到編輯器中：

    ```JSON
    resource "azurerm_resource_group" "k8s" {
        name     = "${var.resource_group_name}"
        location = "${var.location}"
    }

    resource "azurerm_kubernetes_cluster" "k8s" {
        name                = "${var.cluster_name}"
        location            = "${azurerm_resource_group.k8s.location}"
        resource_group_name = "${azurerm_resource_group.k8s.name}"
        dns_prefix          = "${var.dns_prefix}"

        linux_profile {
            admin_username = "ubuntu"

            ssh_key {
            key_data = "${file("${var.ssh_public_key}")}"
            }
        }

        agent_pool_profile {
            name            = "default"
            count           = "${var.agent_count}"
            vm_size         = "Standard_DS2_v2"
            os_type         = "Linux"
            os_disk_size_gb = 30
        }

        service_principal {
            client_id     = "${var.client_id}"
            client_secret = "${var.client_secret}"
        }

        tags {
            Environment = "Development"
        }
    }
    ```

    上述程式碼可設定叢集名稱、位置和 resource_group_name； 此外，也能設定屬於完整網域名稱 (FQDN) 一部分的 dns_prefix 值 (完整網域名稱可用來存取叢集)。

    **linux_profile** 記錄可讓您配置使用 SSH 登入背景工作角色節點的設定。

    透過 AKS，您只需要為背景工作角色節點付費。 **agent_pool_profile** 記錄可設定這些背景工作角色節點的詳細資料。 **agent_pool_profile 記錄**涵蓋要建立的背景工作角色節點數量，以及背景工作角色節點的類型。 若您日後需要擴充或縮減叢集規模，可修改此記錄中的 **count** 值。

1. 選取 **Esc** 鍵結束插入模式。

1. 輸入下列命令來儲存檔案及結束 vi 編輯器：

    ```bash
    :wq
    ```

## <a name="declare-the-variables"></a>宣告變數

1. 在 Cloud Shell 中建立名稱為 `variables.tf` 的檔案。

    ```bash
    vi variables.tf
    ```

1. 選取 I 鍵輸入插入模式。

1. 將下列程式碼貼到編輯器中：

    ```JSON
    variable "client_id" {}
    variable "client_secret" {}

    variable "agent_count" {
        default = 3
    }

    variable "ssh_public_key" {
        default = "~/.ssh/id_rsa.pub"
    }

    variable "dns_prefix" {
        default = "k8stest"
    }

    variable cluster_name {
        default = "k8stest"
    }

    variable resource_group_name {
        default = "azure-k8stest"
    }

    variable location {
        default = "Central US"
    }
    ```

1. 選取 **Esc** 鍵結束插入模式。

1. 輸入下列命令來儲存檔案及結束 vi 編輯器：

    ```bash
    :wq
    ```

## <a name="create-a-terraform-output-file"></a>建立 Terraform 輸出檔
[Terraform 輸出檔](https://www.terraform.io/docs/configuration/outputs.html)可讓您定義 Terraform 套用方案後要對使用者醒目提示的值，以及讓您能使用 `terraform output` 命令查詢此輸出檔。 在本節中，您可建立允許透過 [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) 存取叢集的輸出檔。

1. 在 Cloud Shell 中建立名稱為 `output.tf` 的檔案。

    ```bash
    vi output.tf
    ```

1. 選取 I 鍵輸入插入模式。

1. 將下列程式碼貼到編輯器中：

    ```JSON
    output "client_key" {
        value = "${azurerm_kubernetes_cluster.k8s.kube_config.0.client_key}"
    }

    output "client_certificate" {
        value = "${azurerm_kubernetes_cluster.k8s.kube_config.0.client_certificate}"
    }

    output "cluster_ca_certificate" {
        value = "${azurerm_kubernetes_cluster.k8s.kube_config.0.cluster_ca_certificate}"
    }

    output "cluster_username" {
        value = "${azurerm_kubernetes_cluster.k8s.kube_config.0.username}"
    }

    output "cluster_password" {
        value = "${azurerm_kubernetes_cluster.k8s.kube_config.0.password}"
    }

    output "kube_config" {
        value = "${azurerm_kubernetes_cluster.k8s.kube_config_raw}"
    }

    output "host" {
        value = "${azurerm_kubernetes_cluster.k8s.kube_config.0.host}"
    }
    ```

1. 選取 **Esc** 鍵結束插入模式。

1. 輸入下列命令來儲存檔案及結束 vi 編輯器：

    ```bash
    :wq
    ```

## <a name="set-up-azure-storage-to-store-terraform-state"></a>設定 Azure 儲存體以儲存 Terraform 狀態
Terraform 可透過 `terraform.tfstate` 檔案在本機追蹤狀態。 此模式在單一人員環境中運作良好。 不過，在更常見的多人環境中，您必須追蹤使用 [Azure 儲存體](/azure/storage/)的伺服器的狀態。 在本節中，您可擷取必要的儲存體帳戶資訊 (帳戶名稱和帳戶金鑰)，以及建立儲存體容器，而系統會將 Terraform 狀態資訊儲存在該容器中。

1. 在 Azure 入口網站中，選取左側功能表中的 [所有服務]。

1. 選取 [儲存體帳戶]。

1. 在 [儲存體帳戶] 索引標籤中，選取要讓 Terraform 儲存狀態的帳戶名稱。 例如，您可以使用第一次開啟 Cloud Shell 時建立的儲存體帳戶。  Cloud Shell 建立的儲存體帳戶名稱通常會以 `cs` 開頭，其後加上由數字和字母組成的隨機字串。 **請記下您選取的儲存體帳戶名稱，稍後需要用到。**

1. 在 [儲存體帳戶] 索引標籤中，選取 [存取金鑰]。

    ![[儲存體帳戶] 功能表](./media/terraform-create-k8s-cluster-with-tf-and-aks/storage-account.png)

1. 請記下 **key1** 的**金鑰**值。 (選取金鑰右邊的圖示，將值複製到剪貼簿。)

    ![儲存體帳戶存取金鑰](./media/terraform-create-k8s-cluster-with-tf-and-aks/storage-account-access-key.png)

1. 在 Cloud Shell 中，為您的 Azure 儲存體帳戶建立容器 (使用適用於 Azure 儲存體帳戶的適當值，取代 &lt;YourAzureStorageAccountName> 和 &lt;YourAzureStorageAccountAccessKey> 預留位置中的值)。

    ```bash
    az storage container create -n tfstate --account-name <YourAzureStorageAccountName> --account-key <YourAzureStorageAccountKey>
    ```

## <a name="create-the-kubernetes-cluster"></a>建立 Kubernetes 叢集
在本節中，您可了解如何使用 `terraform init` 命令建立資源，即您在前述小節中建立的組態檔所定義的資源。

1. 在 Cloud Shell 中初始化 Terraform (使用適用於 Azure 儲存體帳戶的適當值，取代 &lt;YourAzureStorageAccountName> 和 &lt;YourAzureStorageAccountAccessKey> 預留位置中的值)。

    ```bash
    terraform init -backend-config="storage_account_name=<YourAzureStorageAccountName>" -backend-config="container_name=tfstate" -backend-config="access_key=<YourStorageAccountAccessKey>" -backend-config="key=codelab.microsoft.tfstate" 
    ```
    
    `terraform init` 命令會顯示後端和提供者外掛程式已成功初始化：

    ![「terraform 初始化」結果範例](./media/terraform-create-k8s-cluster-with-tf-and-aks/terraform-init-complete.png)

1. 匯出服務主體認證。 將 &lt;your-client-id> 和 &lt;your-client-secret> 預留位置分別取代為與您服務主體相關聯的**應用程式識別碼**和**密碼**。

    ```bash
    export TF_VAR_client_id=<your-client-id>
    export TF_VAR_client_secret=<your-client-secret>
    ```

1. 執行 `terraform plan` 命令以建立用來定義基礎結構元素的 Terraform 方案。 

    ```bash
    terraform plan -out out.plan
    ```

    `terraform plan` 命令可顯示您執行 `terraform apply` 命令時將會建立的資源：

    ![「terraform 方案」結果範例](./media/terraform-create-k8s-cluster-with-tf-and-aks/terraform-plan-complete.png)

1. 執行 `terraform apply` 命令以套用方案，並建立 Kubernetes 叢集。 建立 Kubernetes 叢集的程序可能需要數分鐘，而這會導致 Cloud Shell 工作階段逾時。如果 Cloud Shell 工作階段逾時，請遵循「[從 Cloud Shell 逾時復原](#recover-from-a-dloud-shell-timeout)」一節中的步驟以完成教學課程。

    ```bash
    terraform apply out.plan
    ```

    `terraform apply` 命令可顯示建立資源 (即組態檔中定義的資源) 的結果：

    ![「terraform 套用」結果範例](./media/terraform-create-k8s-cluster-with-tf-and-aks/terraform-apply-complete.png)

1. 在 Azure 入口網站中，選取左側功能表中的 [所有服務] 以查看針對新 Kubernetese 叢集建立的資源。

    ![Cloud Shell 提示](./media/terraform-create-k8s-cluster-with-tf-and-aks/k8s-resources-created.png)

## <a name="recover-from-a-cloud-shell-timeout"></a>從 Cloud Shell 逾時復原
如果 Cloud Shell 工作階段逾時，您可執行下列步驟進行復原：

1. 開啟 Cloud Shell 工作階段。

1. 變更為包含 Terraform 組態檔的目錄。

    ```bash
    cd /clouddrive/terraform-aks-k8s
    ```

1. 執行以下命令：

    ```bash
    export KUBECONFIG=./azurek8s
    ```
    
## <a name="test-the-kubernetes-cluster"></a>測試 Kubernetes 叢集
Kubernetes 工具可用來驗證新建立的叢集。

1. 從 Terraform 狀態取得 Kubernetes 組態，並將其儲存在 kubectl 可讀取的檔案中。

    ```bash
    echo "$(terraform output kube_config)" > ./azurek8s
    ```

1. 設定環境變數，以便 kubectl 挑選正確的組態。

    ```bash
    export KUBECONFIG=./azurek8s
    ```

1. 驗證叢集的健康情況。

    ```bash
    kubectl get nodes
    ```

    您可查看背景工作角色節點的詳細資料，這些節點應該都處於 [就緒] 狀態，如下圖所示：

    ![kubectl 工具可讓您驗證 Kubernetes 叢集的健康情況](./media/terraform-create-k8s-cluster-with-tf-and-aks/kubectl-get-nodes.png)

## <a name="next-steps"></a>後續步驟
在本文中，您可了解如何使用 Terraform 和 AKS 建立 Kubernetes 叢集。 以下有一些額外的資源，可協助您深入了解 Azure 上的 Terraform： 

 [Microsoft.com 中的 Terraform 中樞](https://docs.microsoft.com/azure/terraform/)  
 [Terraform Azure 提供者文件](http://aka.ms/terraform)  
 [Terraform Azure 提供者來源](http://aka.ms/tfgit)  
 [Terraform Azure 模組](http://aka.ms/tfmodules)
