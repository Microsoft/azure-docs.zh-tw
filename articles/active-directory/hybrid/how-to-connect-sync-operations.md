---
title: Azure AD Connect 同步：作業工作和考量 | Microsoft Docs
description: 本主題描述 Azure AD Connect 同步處理的作業工作以及如何準備操作此元件。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: b29c1790-37a3-470f-ab69-3cee824d220d
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/13/2017
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 11390f1ad777d20e31c263b4a694ae5cb31f3fd3
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46305807"
---
# <a name="azure-ad-connect-sync-operational-tasks-and-consideration"></a>Azure AD Connect 同步處理：作業工作和考量
本主題的目標在於描述 Azure AD Connect 同步處理的操作工作。

## <a name="staging-mode"></a>預備模式
預備模式可以用於許多案例，包括：

* 高可用性。
* 測試和部署新的組態變更。
* 引進新的伺服器並解除委任舊伺服器。

利用預備模式中的伺服器，您可以在啟用伺服器之前變更組態並預覽變更。 它也可以讓您執行完整的匯入和完整的同步處理，以在生產環境中進行這些變更之前，確認所有變更皆如預期。

您可以在安裝期間選取狀態為 **預備模式**的伺服器。 此動作會讓伺服器進行匯入和同步處理，但它不會執行任何匯出。 預備模式的伺服器不會執行密碼同步處理或密碼回寫，即使您在安裝期間選取這些功能也一樣。 當您停用預備模式時，伺服器會開始匯出、啟用密碼同步處理及啟用密碼回寫。

> [!NOTE]
> 假設您有已啟用密碼雜湊同步處理功能的 Azure AD Connect。 當您啟用暫存模式時，伺服器會停止從內部部署 AD 同步處理密碼變更。 當您停用暫存模式時，伺服器會從最後停止的位置繼續同步處理密碼變更。 如果伺服器在暫存模式中停止一段較長時間，伺服器可能需要一些時間才能將期間內發生的所有密碼變更進行同步處理。
>
>

您仍然可以使用 Synchronization Service Manager 來強制執行匯出。

預備模式的伺服器會繼續收到來自 Active Directory 和 Azure AD 的變更。 這樣永遠都有最新的變更複本，並且可以非常快速地接管另一部伺服器的責任。 如果您對您的主要伺服器進行組態變更，則您有責任對預備模式的伺服器進行相同的變更。

對於具備較舊同步處理技術知識的人員，預備模式是不同的，因為伺服器有它自己的 SQL 資料庫。 此架構可讓預備模式伺服器位於不同的資料中心。

### <a name="verify-the-configuration-of-a-server"></a>驗證伺服器的組態
若要套用此方法，請遵循下列步驟：

1. [準備](#prepare)
2. [組態](#configuration)
3. [匯入和同步處理](#import-and-synchronize)
4. [Verify](#verify)
5. [切換作用中的伺服器](#switch-active-server)

#### <a name="prepare"></a>準備
1. 安裝 Azure AD Connect、選取**預備模式**，然後取消選取安裝精靈中最後一個頁面上的 [啟動同步處理]。 此模式可讓您手動執行同步處理引擎。
   ![ReadyToConfigure](./media/how-to-connect-sync-operations/readytoconfigure.png)
2. 登出/登入，並從 [開始] 功能表中選取 [同步處理服務] 。

#### <a name="configuration"></a>組態
如果您已對主要伺服器進行變更，並想要將組態與預備伺服器進行比較，則請使用 [Azure AD Connect 組態文件產生器](https://github.com/Microsoft/AADConnectConfigDocumenter)。

#### <a name="import-and-synchronize"></a>匯入和同步處理
1. 選取 [連接器]，並選取第一個類型為 [Active Directory Domain Services] 的連接器。 按一下 [執行]，選取 [完整匯入] 和 [確定]。 對這種類型的所有連接器執行下列動作。
2. 選取 [Azure Active Directory (Microsoft)] 類型的連接器。 按一下 [執行]，選取 [完整匯入] 和 [確定]。
3. 確定仍然選取 [連接器] 索引標籤。 針對每一個 [Active Directory Domain Services] 類型的連接器按一下 [執行]、選取 [差異同步處理] 和 [確定]。
4. 選取 [Azure Active Directory (Microsoft)] 類型的連接器。 按一下 [執行]，選取 [差異同步處理] 和 [確定]。

您現在已預備匯出變更至 Azure AD 和內部部署 AD (如果您正在使用 Exchange 混合部署)。 接下來的步驟可讓您在實際開始匯出至目錄之前，檢查將要變更的項目。

#### <a name="verify"></a>Verify
1. 啟動 CMD 命令提示字元並移至 `%ProgramFiles%\Microsoft Azure AD Sync\bin`
2. 執行：`csexport "Name of Connector" %temp%\export.xml /f:x` 連接器名稱可以在同步處理服務中找到。 它的名稱類似 Azure AD 的 "contoso.com – AAD"。
3. 執行：`CSExportAnalyzer %temp%\export.xml > %temp%\export.csv` 您在 %temp% 中有名稱為 export.csv 的檔案，可在 Microsoft Excel 中加以檢查。 此檔案包含即將匯出的所有變更。
4. 對資料或組態進行必要的變更並再次執行這些步驟 (匯入和同步處理和驗證)，直到要匯出的變更皆如預期進行。

**了解 export.csv 檔案** 此檔案的大部分都簡單易懂。 要了解內容所需的一些縮寫：
* OMODT – 物件修改類型。 指出在物件層級的作業是否為新增、更新或刪除。
* AMODT – 屬性修改類型。 指出在屬性層級的作業是否為新增、更新或刪除。

**擷取通用識別碼** export.csv 檔案包含即將匯出的所有變更。 每一個資料列都與連接器空間中的物件變更對應，而物件則由 DN 屬性識別。 DN 屬性是指派給連接器空間中物件的唯一識別碼。 當 export.csv 中有許多資料列/變更需要分析時，您可能很難只根據 DN 屬性來判斷變更屬於哪個物件。 為了簡化分析變更的程序，請使用 csanalyzer.ps1 PowerShell 指令碼。 此指令碼會擷取物件的通用識別碼 (例如 displayName、userPrincipalName)。 若要使用指令碼：
1. 從 [CSAnalyzer](#appendix-csanalyzer) 區段將 PowerShell 指令碼複製到名為 `csanalyzer.ps1` 的檔案。
2. 開啟 PowerShell 視窗，然後瀏覽至您建立 PowerShell 指令碼的資料夾。
3. 執行：`.\csanalyzer.ps1 -xmltoimport %temp%\export.xml`。
4. 您現在已有一個可在 Microsoft Excel 中檢查的名為 **processedusers1.csv** 的檔案。 請注意，此檔案會提供 DN 屬性與通用識別碼 (例如 displayName 和 userPrincipalName) 的對應。 它目前並未包含即將匯出的實際屬性變更。

#### <a name="switch-active-server"></a>切換作用中的伺服器
1. 在目前作用中的伺服器上，關閉伺服器 (DirSync/FIM/Azure AD Sync) 讓它不會匯出至 Azure AD 或將它設為預備模式 (Azure AD Connect)。
2. 在**預備模式**的伺服器上執行安裝精靈並停用**預備模式**。
   ![ReadyToConfigure](./media/how-to-connect-sync-operations/additionaltasks.png)

## <a name="disaster-recovery"></a>災害復原
實作設計的一部分是規劃您在失去同步處理伺服器的災害中應如何應對。 有不同的模型可供使用，要使用哪一個模組取決於許多因素，包括：

* 您在 Azure AD 的停機期間無法變更物件的容錯能力如何？
* 如果您使用密碼同步化，使用者是否接受他們在內部部署變更時必須在 Azure AD 中使用舊密碼？
* 您是否對即時作業有相依性，例如密碼回寫？

根據這些問題的解答和組織的原則，可實作下列其中一個策略：

* 必要時重建。
* 具有備用的待命伺服器，稱為「預備模式」 。
* 使用虛擬機器。

如果您未使用內建的 SQL Express 資料庫，您也應該檢閱 [SQL 高可用性](#sql-high-availability) 一節。

### <a name="rebuild-when-needed"></a>必要時重建
必要時規劃伺服器重建為可行的策略。 安裝同步處理引擎並執行初始匯入和同步處理，通常可以在幾個小時內完成。 如果沒有可用的備用伺服器，則可以暫時使用網域控制站裝載同步處理引擎。

同步處理引擎伺服器不會儲存有關物件的任何狀態，因此可以從 Active Directory 與 Azure AD 中的資料重建資料庫。 **sourceAnchor** 屬性可用來聯結來自內部部署和雲端的物件。 如果您使用現有的內部部署與雲端物件重建伺服器，則同步處理引擎會在重新安裝時一起比對這些物件。 您需要記錄和儲存的項目是對伺服器進行的組態變更，例如篩選和同步處理規則。 這些自訂設定必須在您開始同步處理之前重新套用。

### <a name="have-a-spare-standby-server---staging-mode"></a>具有備用的待命伺服器 - 預備模式
如果您有更複雜的環境，則建議使用一或多個待命伺服器。 您可以在安裝期間啟用狀態為「預備模式」 的伺服器。

如需詳細資訊，請參閱[預備模式](#staging-mode)。

### <a name="use-virtual-machines"></a>使用虛擬機器
一般和受支援的方法是在虛擬機器中執行同步處理引擎。 如果主機有問題，可將內含同步處理引擎伺服器的映像移轉到另一部伺服器。

### <a name="sql-high-availability"></a>SQL 高可用性
如果您未使用 Azure AD Connect 隨附的 SQL Server Express，也應該考慮 SQL Server 的高可用性。 支援的高可用性解決方案包含 SQL 叢集和 AOA (「永遠開啟」可用性群組）。 不支援的解決方案包括鏡像。

Azure AD Connect 1.1.524.0 版中已新增 SQL AOA 支援。 您必須在安裝 Azure AD Connect 之前啟用 SQL AOA。 在安裝期間，Azure AD Connect 會偵測所提供的 SQL 執行個體是否已啟用 SQL AOA。 如果已啟用 SQL AOA，Azure AD Connect 會進一步指出 SQL AOA 已設定為使用同步複寫或非同步複寫。 設定可用性群組接聽程式時，建議將 RegisterAllProvidersIP 屬性設定為 0。 這是因為 Azure AD Connect 目前使用 SQL Native Client 來連線至 SQL，且 SQL Native Client 不支援使用 MultiSubNetFailover 屬性。

## <a name="appendix-csanalyzer"></a>附錄 CSAnalyzer
請參閱[驗證](#verify)一節，以了解如何使用此指令碼。

```
Param(
    [Parameter(Mandatory=$true, HelpMessage="Must be a file generated using csexport 'Name of Connector' export.xml /f:x)")]
    [string]$xmltoimport="%temp%\exportedStage1a.xml",
    [Parameter(Mandatory=$false, HelpMessage="Maximum number of users per output file")][int]$batchsize=1000,
    [Parameter(Mandatory=$false, HelpMessage="Show console output")][bool]$showOutput=$false
)

#LINQ isn't loaded automatically, so force it
[Reflection.Assembly]::Load("System.Xml.Linq, Version=3.5.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089") | Out-Null

[int]$count=1
[int]$outputfilecount=1
[array]$objOutputUsers=@()

#XML must be generated using "csexport "Name of Connector" export.xml /f:x"
write-host "Importing XML" -ForegroundColor Yellow

#XmlReader.Create won't properly resolve the file location,
#so expand and then resolve it
$resolvedXMLtoimport=Resolve-Path -Path ([Environment]::ExpandEnvironmentVariables($xmltoimport))

#use an XmlReader to deal with even large files
$result=$reader = [System.Xml.XmlReader]::Create($resolvedXMLtoimport) 
$result=$reader.ReadToDescendant('cs-object')
do 
{
    #create the object placeholder
    #adding them up here means we can enforce consistency
    $objOutputUser=New-Object psobject
    Add-Member -InputObject $objOutputUser -MemberType NoteProperty -Name ID -Value ""
    Add-Member -InputObject $objOutputUser -MemberType NoteProperty -Name Type -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name DN -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name operation -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name UPN -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name displayName -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name sourceAnchor -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name alias -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name primarySMTP -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name onPremisesSamAccountName -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name mail -Value ""

    $user = [System.Xml.Linq.XElement]::ReadFrom($reader)
    if ($showOutput) {Write-Host Found an exported object... -ForegroundColor Green}

    #object id
    $outID=$user.Attribute('id').Value
    if ($showOutput) {Write-Host ID: $outID}
    $objOutputUser.ID=$outID

    #object type
    $outType=$user.Attribute('object-type').Value
    if ($showOutput) {Write-Host Type: $outType}
    $objOutputUser.Type=$outType

    #dn
    $outDN= $user.Element('unapplied-export').Element('delta').Attribute('dn').Value
    if ($showOutput) {Write-Host DN: $outDN}
    $objOutputUser.DN=$outDN

    #operation
    $outOperation= $user.Element('unapplied-export').Element('delta').Attribute('operation').Value
    if ($showOutput) {Write-Host Operation: $outOperation}
    $objOutputUser.operation=$outOperation

    #now that we have the basics, go get the details

    foreach ($attr in $user.Element('unapplied-export-hologram').Element('entry').Elements("attr"))
    {
        $attrvalue=$attr.Attribute('name').Value
        $internalvalue= $attr.Element('value').Value

        switch ($attrvalue)
        {
            "userPrincipalName"
            {
                if ($showOutput) {Write-Host UPN: $internalvalue}
                $objOutputUser.UPN=$internalvalue
            }
            "displayName"
            {
                if ($showOutput) {Write-Host displayName: $internalvalue}
                $objOutputUser.displayName=$internalvalue
            }
            "sourceAnchor"
            {
                if ($showOutput) {Write-Host sourceAnchor: $internalvalue}
                $objOutputUser.sourceAnchor=$internalvalue
            }
            "alias"
            {
                if ($showOutput) {Write-Host alias: $internalvalue}
                $objOutputUser.alias=$internalvalue
            }
            "proxyAddresses"
            {
                if ($showOutput) {Write-Host primarySMTP: ($internalvalue -replace "SMTP:","")}
                $objOutputUser.primarySMTP=$internalvalue -replace "SMTP:",""
            }
        }
    }

    $objOutputUsers += $objOutputUser

    Write-Progress -activity "Processing ${xmltoimport} in batches of ${batchsize}" -status "Batch ${outputfilecount}: " -percentComplete (($objOutputUsers.Count / $batchsize) * 100)

    #every so often, dump the processed users in case we blow up somewhere
    if ($count % $batchsize -eq 0)
    {
        Write-Host Hit the maximum users processed without completion... -ForegroundColor Yellow

        #export the collection of users as a CSV
        Write-Host Writing processedusers${outputfilecount}.csv -ForegroundColor Yellow
        $objOutputUsers | Export-Csv -path processedusers${outputfilecount}.csv -NoTypeInformation

        #increment the output file counter
        $outputfilecount+=1

        #reset the collection and the user counter
        $objOutputUsers = $null
        $count=0
    }

    $count+=1

    #need to bail out of the loop if no more users to process
    if ($reader.NodeType -eq [System.Xml.XmlNodeType]::EndElement)
    {
        break
    }

} while ($reader.Read)

#need to write out any users that didn't get picked up in a batch of 1000
#export the collection of users as CSV
Write-Host Writing processedusers${outputfilecount}.csv -ForegroundColor Yellow
$objOutputUsers | Export-Csv -path processedusers${outputfilecount}.csv -NoTypeInformation
```

## <a name="next-steps"></a>後續步驟
**概觀主題**  

* [Azure AD Connect 同步處理：了解及自訂同步處理](how-to-connect-sync-whatis.md)  
* [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)  
