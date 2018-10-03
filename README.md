## <a name="microsoft-open-source-code-of-conduct"></a>Microsoft 開放原始碼管理辦法

此專案採用 [Microsoft 開放原始碼管理辦法](https://opensource.microsoft.com/codeofconduct/)。
如需詳細資訊，請參閱[管理辦法常見問題集](https://opensource.microsoft.com/codeofconduct/faq/)，如有任何其他問題或意見請連絡 [opencode@microsoft.com](mailto:opencode@microsoft.com)。

## <a name="contribute-to-azure-technical-documentation"></a>參與編輯 Azure 技術文件
我們歡迎我們的社群 (使用者、客戶、夥伴、核心 Azure 產品單位以外的 MSFT 員工等) 以及在核心 Azure 產品單位工作的員工參與編輯。 您的參與方式取決於您的身分：

* **社群 - 小幅更新**：如果您自願參與小幅更新，即可在此存放庫中尋找文章，或瀏覽 [https://docs.microsoft.com/azure](https://docs.microsoft.com/azure) 上的文章，然後按一下文章中的 [編輯] 連結以前往文章的 GitHub 來源。 接著，只需使用 GitHub UI 進行您的更新。 或者，您可以隨意地建立存放庫分支，然後從您的分支提交更新。

* **社群 - 新的文章**：如果您屬於 Azure 社群且想要建立新的文章，便需要與員工合作，以協助透過將公用和私人存放庫中的工作相結合來帶入新的內容。

* **員工**：如果您是來自 Azure 服務產品小組的技術撰寫者、程式管理員或開發人員，而您的職責就是參與編輯或撰寫技術文件，則應該使用私人存放庫 (https://github.com/MicrosoftDocs/azure-docs-pr)。 如果要大幅變更現有文章、新增或變更圖片，或是參與編輯新文章，您需要建立此存放庫的分支、安裝 Git Bash、Markdown editor，並學習一些 Git 命令才行。 如需詳細資訊，請參閱[內部參與者指南](https://review.docs.microsoft.com/en-us/help/contribute/?branch=master)。


## <a name="about-your-contributions-to-azure-content"></a>關於您對 Azure 內容的參與
### <a name="minor-corrections"></a>小幅度修正
您在此儲存機制中針對文件和程式碼範例所提交的小幅度修正或釐清，將受到 [docs.microsoft.com 使用規定](https://docs.microsoft.com/legal/termsofuse)的約束。

### <a name="larger-submissions"></a>較大範圍的提交內容
如果您提交提取要求是為了對文件和程式碼範例進行全新或大幅度的變更，而且您不是 Microsoft 的員工，我們會在 GitHub 中傳送註解，要求您提交線上版的參與授權合約 (Contribution License Agreement, CLA)： 您必須先填寫線上表單，我們才會接受您的提取要求。

## <a name="tools-and-setup"></a>工具和安裝程式
社群參與者可以使用 GitHub UI 或建立存放庫分支來參與編輯。 員工應該造訪[內部參與者指南](https://review.docs.microsoft.com/en-us/help/contribute/?branch=master)，以取得如何參與編輯技術文件集的詳細資訊。

## <a name="repository-organization"></a>儲存機制組織方式
azure-docs 存放庫中的內容會遵循 https://docs.microsoft.com/azure 上文件的組織方式。 此儲存機制包含兩個根資料夾：

### <a name="articles"></a>\articles
*\articles* 資料夾包含格式化為 Markdown 檔的文件文章，其副檔名為 *.md*。 文章通常會依 Azure 服務分入群組。

*\articles* 資料夾內有 *\media* 資料夾可供存放根目錄文章媒體檔，而在 \media 資料夾內則是存有各文章內圖片的子資料夾。  每個服務資料夾內都含有不同的媒體資料夾來存放文章。 文章圖片資料夾的名稱和文章檔案的名稱相同，但不會有 *.md* 副檔名。

### <a name="includes"></a>\includes
您也可以建立特定的內容區段，以便重複地用來加入一個或多個文章之中。 

## <a name="how-to-use-markdown-to-format-your-topic"></a>如何使用 Markdown 來格式化主題
此儲存機制中的所有文章都使用 GitHub 風格的 Markdown。  其資源清單如下。

* [Markdown 基本概念](https://help.github.com/articles/markdown-basics/)
* [可列印的 Markdown 速查表](https://guides.github.com/pdfs/markdown-cheatsheet-online.pdf)


## <a name="labels"></a>標籤
在公用的 Azure 文件存放庫中，系統會指派自動化標籤來提取要求，有助於我們管理提取要求工作流程，並協助讓您知道您的提取要求做了些什麼：

* 相關的參與授權合約
  * 不需要 cla︰變更幅度相當小，而且不需要您登入 CLA。
  * 需要 cla︰變更幅度相對較大，需要您登入 CLA。
  * cla 簽章︰ 參與者簽署 CLA，因此提取要求可繼續進行作業以供檢閱。
* 傳送變更給作者︰會通知作者已暫止提取要求。
* 準備好合併：已準備好供我們的提取要求審查小組進行審查。


