## <a name="export-an-api-definition"></a>匯出 API 定義
您從[為函式建立 OpenAPI 定義](../articles/azure-functions/functions-openapi-definition.md)取得函式的 OpenAPI 定義。 此程序的下一個步驟是匯出 API 定義，以便 PowerApps 和 Microsoft Flow 可以在自訂 API 中使用。

> [!IMPORTANT]
> 請記住，您必須使用與用於 PowerApps 和 Microsoft Flow 租用戶相同的認證，來登入 Azure。 這可讓 Azure 建立自訂 API，並將其提供給 PowerApps 和 Microsoft Flow。

1. 在 [Azure 入口網站](https://portal.azure.com)中，按一下函式應用程式名稱 (例如 **function-demo-energy**) > [平台功能][API 定義] > 。

    ![API 定義](media/functions-export-api-definition/api-definition.png)

1. 按一下 [Export to PowerApps + Flow] \(匯出至 PowerApps + Flow)。

    ![API 定義來源](media/functions-export-api-definition/export-api-1.png)

1. 在右窗格中，使用表格中所指定的設定。

    |設定|說明|
    |--------|------------|
    |**匯出模式**|選取 [快速] 以自動產生自訂 API。 選取 [手動] 以匯出 API 定義，但您必須接著手動將它匯入 PowerApps 和 Microsoft Flow。 如需詳細資訊，請參閱[匯出至 PowerApps 和 Microsoft Flow](../articles/azure-functions/app-service-export-api-to-powerapps-and-flow.md)。|
    |**環境**|選取應該儲存自訂 API 的目標環境。 如需詳細資訊，請參閱[環境概觀 (PowerApps)](https://powerapps.microsoft.com/tutorials/environments-overview/) 或[環境概觀 (Microsoft Flow)](https://us.flow.microsoft.com/documentation/environments-overview-admin/)。|
    |**自訂 API 名稱**|輸入名稱，例如 `Turbine Repair`。|
    |**API 金鑰名稱**|輸入應該會在自訂 API UI 中看到的應用程式和流程建立器名稱。 請注意，此範例包含有用的資訊。|
 
    ![匯出至 PowerApps 和 Microsoft Flow](media/functions-export-api-definition/export-api-2.png)

1. 按一下 [確定]。 自訂 API 現已建置並新增至您指定的環境。