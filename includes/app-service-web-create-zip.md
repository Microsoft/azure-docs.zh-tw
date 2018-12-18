## <a name="create-a-project-zip-file"></a>建立專案 ZIP 檔案

確定您仍在專案範例的根目錄中。 在專案中建立所有項目的 ZIP 封存。 下列命令會使用您終端機中的預設工具：

```
# Bash
zip -r myAppFiles.zip .

# PowerShell
Compress-Archive -Path * -DestinationPath myAppFiles.zip
``` 

稍後，您要將此 ZIP 檔案上傳至 Azure，並將它部署到 App Service。