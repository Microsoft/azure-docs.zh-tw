## <a name="deploy-template-from-cloud-shell"></a>從 Cloud Shell 部署範本

您可以使用 [Cloud Shell](../articles/cloud-shell/overview.md) 來部署您的範本。 不過，您必須先將範本載入 Cloud Shell 的儲存體帳戶。 如果您尚未使用 Cloud Shell，請參閱 [Azure Cloud Shell 概觀](../articles/cloud-shell/overview.md)以取得如何設定的相關資訊。

1. 登入 [Azure 入口網站](https://portal.azure.com)。

1. 選取您的 Cloud Shell 資源群組。 名稱模式為 `cloud-shell-storage-<region>`。

   ![選取資源群組](./media/resource-manager-cloud-shell-deploy/select-cs-resource-group.png)

1. 選取 Cloud Shell 的儲存體帳戶。

   ![選取儲存體帳戶](./media/resource-manager-cloud-shell-deploy/select-storage.png)

1. 選取 [Blob] 。

   ![選取 Blob](./media/resource-manager-cloud-shell-deploy/select-blobs.png)

1. 選取 [+ 容器]。

   ![新增容器](./media/resource-manager-cloud-shell-deploy/add-container.png)

1. 為您的容器提供名稱和存取層級。 本文中的範例範本不包含機密資訊，因此允許匿名讀取權限。 選取 [確定] 。

   ![提供容器值](./media/resource-manager-cloud-shell-deploy/provide-container-values.png)

1. 選取您建立的容器。

   ![選取新容器](./media/resource-manager-cloud-shell-deploy/select-container.png)

1. 選取 [上傳] 。

   ![上傳 Blob](./media/resource-manager-cloud-shell-deploy/upload-blob.png)

1. 尋找並上傳您的範本。

   ![上傳檔案](./media/resource-manager-cloud-shell-deploy/find-and-upload-template.png)

1. 上傳之後，選取範本。

   ![選取新範本](./media/resource-manager-cloud-shell-deploy/select-new-template.png)

1. 複製 URL。

   ![複製 URL](./media/resource-manager-cloud-shell-deploy/copy-url.png)

1. 開啟提示字元。

   ![開啟 Cloud Shell](./media/resource-manager-cloud-shell-deploy/start-cloud-shell.png)
