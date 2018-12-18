---
title: Azure AD Connect 同步：函式參考 | Microsoft Docs
description: Azure AD Connect 同步處理中宣告式佈建運算式的參考。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: 4f525ca0-be0e-4a2e-8da1-09b6b567ed5f
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/12/2017
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 2302061d5a3eaf9ef51543b1304104e2dbe54bd9
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46307247"
---
# <a name="azure-ad-connect-sync-functions-reference"></a>Azure AD Connect 同步處理：函式參考
在 Azure AD Connect 中，函數是用來在同步處理期間操作屬性值。  
函式的語法可使用下列格式來表示：  
`<output type> FunctionName(<input type> <position name>, ..)`

如果函式是多載的且接受多種語法，即會列出所有的有效語法。  
函式是強型別的，且會確認傳入的類型與記載的類型相符。  
如果類型不符，即會擲回錯誤。

類型可以使用下列語法來表示：

*  – 二進位
*  – 布林值
*  – UTC 日期/時間
*  – 已知常數的列舉
*  – Expression, which is ected to evaluate to a Boolean
* **mvbin** – 多重值的二進位
* **mvstr** – 多重值的字串
* **mvref** – 多重值的參考
* **num** – 數值
* **ref** – 參考
* **str** – 字串
* **var** – (幾乎) 任何其他類型的變體
* **void** – 不會傳回值

類型為 **mvbin**、**mvstr** 和 **mvref** 的函式僅可用於多重值的屬性。 類型為 **bin**、**str** 和 **ref** 的函式可用於單一值和多重值的屬性。

## <a name="functions-reference"></a>函式參考
| 函數的清單 |  |  |  |  |
| --- | --- | --- | --- | --- | --- |
| **憑證** | | | | |
| [CertExtensionOids](#certextensionoids) |[CertFormat](#certformat) |[CertFriendlyName](#certfriendlyname) |[CertHashString](#certhashstring) | |
| [CertIssuer](#certissuer) |[CertIssuerDN](#certissuerdn) |[CertIssuerOid](#certissueroid) |[CertKeyAlgorithm](#certkeyalgorithm) | |
| [CertKeyAlgorithmParams](#certkeyalgorithmparams) |[CertNameInfo](#certnameinfo) |[CertNotAfter](#certnotafter) |[CertNotBefore](#certnotbefore) | |
| [CertPublicKeyOid](#certpublickeyoid) |[CertPublicKeyParametersOid](#certpublickeyparametersoid) |[CertSerialNumber](#certserialnumber) |[CertSignatureAlgorithmOid](#certsignaturealgorithmoid) | |
| [CertSubject](#certsubject) |[CertSubjectNameDN](#certsubjectnamedn) |[CertSubjectNameOid](#certsubjectnameoid) |[CertThumbprint](#certthumbprint) | |
[ CertVersion](#certversion) |[IsCert](#iscert) | | | |
| **轉換** | | | | |
| [CBool](#cbool) |[CDate](#cdate) |[CGuid](#cguid) |[ConvertFromBase64](#convertfrombase64) | |
| [ConvertToBase64](#converttobase64) |[ConvertFromUTF8Hex](#convertfromutf8hex) |[ConvertToUTF8Hex](#converttoutf8hex) |[CNum](#cnum) | |
| [CRef](#cref) |[CStr](#cstr) |[StringFromGuid](#StringFromGuid) |[StringFromSid](#stringfromsid) | |
| **日期 / 時間** | | | | |
| [DateAdd](#dateadd) |[DateFromNum](#datefromnum) |[FormatDateTime](#formatdatetime) |[Now](#now) | |
| [NumFromDate](#numfromdate) | | | | |
| **目錄** | | | | |
| [DNComponent](#dncomponent) |[DNComponentRev](#dncomponentrev) |[EscapeDNComponent](#escapedncomponent) | | |
| **評估** | | | | |
| [IsBitSet](#isbitset) |[IsDate](#isdate) |[IsEmpty](#isempty) |[IsGuid](#isguid) | |
| [IsNull](#isnull) |[IsNullOrEmpty](#isnullorempty) |[IsNumeric](#isnumeric) |[IsPresent](#ispresent) | |
| [IsString](#isstring) | | | | |
| **數學運算** | | | | |
| [BitAnd](#bitand) |[BitOr](#bitor) |[RandomNum](#randomnum) | | |
| **多重值** | | | | |
| [Contains](#contains) |[Count](#count) |[Item](#item) |[ItemOrNull](#itemornull) | |
| [Join](#join) |[RemoveDuplicates](#removeduplicates) |[Split](#split) | | |
| **程式流程** | | | | |
| [Error](#error) |[IIF](#iif) |[選取](#select) |[Switch](#switch) | |
| [其中](#where) |[With](#with) | | | |
| **文字** | | | | |
| [GUID](#guid) |[InStr](#instr) |[InStrRev](#instrrev) |[LCase](#lcase) | |
| [Left](#left) |[Len](#len) |[LTrim](#ltrim) |[Mid](#mid) | |
| [PadLeft](#padleft) |[PadRight](#padright) |[PCase](#pcase) |[Replace](#replace) | |
| [ReplaceChars](#replacechars) |[Right](#right) |[RTrim](#rtrim) |[Trim](#trim) | |
| [UCase](#ucase) |[Word](#word) | | | |

- - -
### <a name="bitand"></a>BitAnd
**說明：**  
BitAnd 函式會在值中設定指定的位元。

**語法：**  
`num BitAnd(num value1, num value2)`

* value1，value2：應該使用 AND 連結在一起的數值

**備註：**  
此函式會將這兩個參數轉換為二進位表示法，並將某一個位元設為：

* 0 - 如果 *mask* 和 *flag* 中有一個對應位元或這兩者為 0
* 1 - 如果這兩個對應位元都是 1。

換句話說，除非這兩個參數的對應位元都是 1，否則在所有情況下都會傳回 0。

**範例：**  
`BitAnd(&HF, &HF7)`  
傳回 7，因為十六進位 "F" AND "F7" 會評估為此值。

- - -
### <a name="bitor"></a>BitOr
**說明：**  
BitOr 函式會在值中設定指定的位元。

**語法：**  
`num BitOr(num value1, num value2)`

* value1，value2：應該使用 OR 連結在一起的數值

**備註：**  
此函式會將這兩個參數轉換成二進位表示法，而且，如果 mask 和 flag 中有一個的對應位元為 1 或兩者都是，就會將某一個位元設為 1，如果這兩個對應位元都為 0，則會設為 0。 換句話說，除非這兩個參數的對應位元都是 0，否則在所有情況下都會傳回 1。

- - -
### <a name="cbool"></a>CBool
**說明：**  
CBool 函式會根據評估的運算式傳回布林值

**語法：**  
`bool CBool(exp Expression)`

**備註：**  
如果運算式評估為非零值，CBool 就會傳回 True，否則會傳回 False。

**範例：**  
`CBool([attrib1] = [attrib2])`  

如果這兩個屬性的值相同，即會傳回 True。

- - -
### <a name="cdate"></a>CDate
**說明：**  
CDate 函式會傳回字串的 UTC DateTime。 DateTime 不是同步處理中的原生屬性類型，但有部分函式會用到。

**語法：**  
`dt CDate(str value)`

* Value：包含日期、時間和選擇性時區的字串

**備註：**  
傳回的字串一律以 UTC 來表示。

**範例：**  
`CDate([employeeStartTime])`  
根據員工的開始時間傳回 DateTime

`CDate("2013-01-10 4:00 PM -8")`  
傳回代表 "2013-01-11 12:00 AM" 的 DateTime


- - -
### <a name="certextensionoids"></a>CertExtensionOids
**說明：**  
傳回所有憑證物件重要延伸模組的 Oid 值。

**語法：**  
`mvstr CertExtensionOids(binary certificateRawData)`  
*   certificateRawData：X.509 憑證的位元組陣列表示。 位元組陣列可以是編碼的二進位 (DER) 或 Base64 編碼的 X.509 資料。

- - -
### <a name="certformat"></a>CertFormat
**說明：**  
傳回此 X.509v3 憑證的格式名稱。

**語法：**  
`str CertFormat(binary certificateRawData)`  
*   certificateRawData：X.509 憑證的位元組陣列表示。 位元組陣列可以是編碼的二進位 (DER) 或 Base64 編碼的 X.509 資料。

- - -
### <a name="certfriendlyname"></a>CertFriendlyName
**說明：**  
傳回憑證的相關聯別名。

**語法：**  
`str CertFriendlyName(binary certificateRawData)`  
*   certificateRawData：X.509 憑證的位元組陣列表示。 位元組陣列可以是編碼的二進位 (DER) 或 Base64 編碼的 X.509 資料。

- - -
### <a name="certhashstring"></a>CertHashString
**說明：**  
為傳回 X.509v3 憑證的 SHA1 雜湊值作為十六進位字串。

**語法：**  
`str CertHashString(binary certificateRawData)`  
*   certificateRawData：X.509 憑證的位元組陣列表示。 位元組陣列可以是編碼的二進位 (DER) 或 Base64 編碼的 X.509 資料。

- - -
### <a name="certissuer"></a>CertIssuer
**說明：**  
傳回發出 X.509v3 憑證的憑證授權單位名稱。

**語法：**  
`str CertIssuer(binary certificateRawData)`  
*   certificateRawData：X.509 憑證的位元組陣列表示。 位元組陣列可以是編碼的二進位 (DER) 或 Base64 編碼的 X.509 資料。

- - -
### <a name="certissuerdn"></a>CertIssuerDN
**說明：**  
傳回憑證簽發者的辨別名稱。

**語法：**  
`str CertIssuerDN(binary certificateRawData)`  
*   certificateRawData：X.509 憑證的位元組陣列表示。 位元組陣列可以是編碼的二進位 (DER) 或 Base64 編碼的 X.509 資料。

- - -
### <a name="certissueroid"></a>CertIssuerOid
**說明：**  
傳回憑證簽發者的 Oid。

**語法：**  
`str CertIssuerOid(binary certificateRawData)`  
*   certificateRawData：X.509 憑證的位元組陣列表示。 位元組陣列可以是編碼的二進位 (DER) 或 Base64 編碼的 X.509 資料。

- - -
### <a name="certkeyalgorithm"></a>CertKeyAlgorithm
**說明：**  
傳回此 X.509v3 憑證的金鑰演算法資訊作為字串。

**語法：**  
`str CertKeyAlgorithm(binary certificateRawData)`  
*   certificateRawData：X.509 憑證的位元組陣列表示。 位元組陣列可以是編碼的二進位 (DER) 或 Base64 編碼的 X.509 資料。

- - -
### <a name="certkeyalgorithmparams"></a>CertKeyAlgorithmParams
**說明：**  
傳回此 X.509v3 憑證的金鑰演算法參數作為十六進位字串。

**語法：**  
`str CertKeyAlgorithm(binary certificateRawData)`  
*   certificateRawData：X.509 憑證的位元組陣列表示。 位元組陣列可以是編碼的二進位 (DER) 或 Base64 編碼的 X.509 資料。

- - -
### <a name="certnameinfo"></a>CertNameInfo
**說明：**  
傳回憑證的主體與簽發者名稱。

**語法：**  
`str CertNameInfo(binary certificateRawData, str x509NameType, bool includesIssuerName)`  
*   certificateRawData：X.509 憑證的位元組陣列表示。 位元組陣列可以是編碼的二進位 (DER) 或 Base64 編碼的 X.509 資料。
*   X509NameType：主體的 X509NameType 值。
*   includesIssuerName：true 則包含簽發者名稱，否則為 false。

- - -
### <a name="certnotafter"></a>CertNotAfter
**說明：**  
傳回憑證失效的當地時間日期。

**語法：**  
`dt CertNotAfter(binary certificateRawData)`  
*   certificateRawData：X.509 憑證的位元組陣列表示。 位元組陣列可以是編碼的二進位 (DER) 或 Base64 編碼的 X.509 資料。

- - -
### <a name="certnotbefore"></a>CertNotBefore
**說明：**  
傳回憑證開始生效的當地時間日期。

**語法：**  
`dt CertNotBefore(binary certificateRawData)`  
*   certificateRawData：X.509 憑證的位元組陣列表示。 位元組陣列可以是編碼的二進位 (DER) 或 Base64 編碼的 X.509 資料。

- - -
### <a name="certpublickeyoid"></a>CertPublicKeyOid
**說明：**  
傳回 X.509v3 憑證公開金鑰的 Oid。

**語法：**  
`str CertKeyAlgorithm(binary certificateRawData)`  
*   certificateRawData：X.509 憑證的位元組陣列表示。 位元組陣列可以是編碼的二進位 (DER) 或 Base64 編碼的 X.509 資料。

- - -
### <a name="certpublickeyparametersoid"></a>CertPublicKeyParametersOid
**說明：**  
傳回 X.509v3 憑證公開金鑰參數的 Oid。

**語法：**  
`str CertPublicKeyParametersOid(binary certificateRawData)`  
*   certificateRawData：X.509 憑證的位元組陣列表示。 位元組陣列可以是編碼的二進位 (DER) 或 Base64 編碼的 X.509 資料。

- - -
### <a name="certserialnumber"></a>CertSerialNumber
**說明：**  
傳回 X.509v3 憑證的序號。

**語法：**  
`str CertSerialNumber(binary certificateRawData)`  
*   certificateRawData：X.509 憑證的位元組陣列表示。 位元組陣列可以是編碼的二進位 (DER) 或 Base64 編碼的 X.509 資料。

- - -
### <a name="certsignaturealgorithmoid"></a>CertSignatureAlgorithmOid
**說明：**  
傳回建立憑證簽章所用演算法的 Oid。

**語法：**  
`str CertSignatureAlgorithmOid(binary certificateRawData)`  
*   certificateRawData：X.509 憑證的位元組陣列表示。 位元組陣列可以是編碼的二進位 (DER) 或 Base64 編碼的 X.509 資料。

- - -
### <a name="certsubject"></a>CertSubject
**說明：**  
取得憑證的主體辨別名稱。

**語法：**  
`str CertSubject(binary certificateRawData)`  
*   certificateRawData：X.509 憑證的位元組陣列表示。 位元組陣列可以是編碼的二進位 (DER) 或 Base64 編碼的 X.509 資料。

- - -
### <a name="certsubjectnamedn"></a>CertSubjectNameDN
**說明：**  
傳回憑證的主體辨別名稱。

**語法：**  
`str CertSubjectNameDN(binary certificateRawData)`  
*   certificateRawData：X.509 憑證的位元組陣列表示。 位元組陣列可以是編碼的二進位 (DER) 或 Base64 編碼的 X.509 資料。

- - -
### <a name="certsubjectnameoid"></a>CertSubjectNameOid
**說明：**  
傳回憑證主體名稱的 Oid。

**語法：**  
`str CertSubjectNameOid(binary certificateRawData)`  
*   certificateRawData：X.509 憑證的位元組陣列表示。 位元組陣列可以是編碼的二進位 (DER) 或 Base64 編碼的 X.509 資料。

- - -
### <a name="certthumbprint"></a>CertThumbprint
**說明：**  
傳回憑證的指紋。

**語法：**  
`str CertThumbprint(binary certificateRawData)`  
*   certificateRawData：X.509 憑證的位元組陣列表示。 位元組陣列可以是編碼的二進位 (DER) 或 Base64 編碼的 X.509 資料。

- - -
### <a name="certversion"></a>CertVersion
**說明：**  
傳回憑證的 X.509 格式版本。

**語法：**  
`str CertThumbprint(binary certificateRawData)`  
*   certificateRawData：X.509 憑證的位元組陣列表示。 位元組陣列可以是編碼的二進位 (DER) 或 Base64 編碼的 X.509 資料。

- - -
### <a name="cguid"></a>CGuid
**說明：**  
CGuid 函式會將 GUID 的字串表示法轉換為二進位表示法。

**語法：**  
`bin CGuid(str GUID)`

* 使用此模式格式化的字串：xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx 或 {xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}

- - -
### <a name="contains"></a>Contains
**說明：**  
Contains 函式會在多重值屬性內尋找字串

**語法：**  
`num Contains (mvstring attribute, str search)` - 區分大小寫  
`num Contains (mvstring attribute, str search, enum Casetype)`  
`num Contains (mvref attribute, str search)` - 區分大小寫

* attribute：要搜尋的多重值屬性。
* search：要在屬性中尋找的字串。
* Casetype：CaseInsensitive 或 CaseSensitive。

傳回在多重值屬性中找到字串的索引。 如果找不到該字串，即會傳回 0。

**備註：**  
針對多重值字串屬性，搜尋會在值中尋找子字串。  
針對參考屬性，搜尋的字串必須完全符合要被視為相符的值。

**範例：**  
`IIF(Contains([proxyAddresses],"SMTP:")>0,[proxyAddresses],Error("No primary SMTP address found."))`  
如果 proxyAddresses 屬性具有主要電子郵件地址 (以大寫 "SMTP:" 表示)，就會傳回 proxyAddress 屬性，否則會傳回錯誤。

- - -
### <a name="convertfrombase64"></a>ConvertFromBase64
**說明：**  
ConvertFromBase64 函式會將指定的 base64 編碼值轉換為一般字串。

**語法：**  
`str ConvertFromBase64(str source)` - 假設使用 Unicode 進行編碼  
`str ConvertFromBase64(str source, enum Encoding)`

* source：Base64 編碼的字串  
* Encoding：Unicode、ASCII、UTF8

**範例**  
`ConvertFromBase64("SABlAGwAbABvACAAdwBvAHIAbABkACEA")`  
`ConvertFromBase64("SGVsbG8gd29ybGQh", UTF8)`

這兩個範例都會傳回 "*Hello world!*"

- - -
### <a name="convertfromutf8hex"></a>ConvertFromUTF8Hex
**說明：**  
ConvertFromUTF8Hex 函式會將指定的 UTF8 十六進位編碼值轉換為字串。

**語法：**  
`str ConvertFromUTF8Hex(str source)`

* source：UTF8 2 個位元組的編碼字串

**備註：**  
此函式與 ConvertFromBase64([],UTF8) 之間的差異在於結果支援 DN 屬性。  
Azure Active Directory 會使用此格式做為 DN。

**範例：**  
`ConvertFromUTF8Hex("48656C6C6F20776F726C6421")`  
傳回 "*Hello world!*"

- - -
### <a name="converttobase64"></a>ConvertToBase64
**說明：**  
ConvertToBase64 函式會將字串轉換為 Unicode Base64 字串。  
將整數陣列的值轉換為其對等的字串表示法，此表示法是以 Base-64 數字編碼的。

**語法：**  
`str ConvertToBase64(str source)`

**範例：**  
`ConvertToBase64("Hello world!")`  
傳回 "SABlAGwAbABvACAAdwBvAHIAbABkACEA"

- - -
### <a name="converttoutf8hex"></a>ConvertToUTF8Hex
**說明：**  
ConvertToUTF8Hex 函式會將字串轉換為 UTF8 十六進位編碼值。

**語法：**  
`str ConvertToUTF8Hex(str source)`

**備註：**  
Azure Active Directory 會使用此函式的輸出格式做為 DN 屬性格式。

**範例：**  
`ConvertToUTF8Hex("Hello world!")`  
傳回 48656C6C6F20776F726C6421

- - -
### <a name="count"></a>Count
**說明：**  
Count 函式會傳回多重值屬性中的元素個數

**語法：**  
`num Count(mvstr attribute)`

- - -
### <a name="cnum"></a>CNum
**說明：**  
CNum 函式會取得字串，並傳回數值資料類型。

**語法：**  
`num CNum(str value)`

- - -
### <a name="cref"></a>CRef
**說明：**  
將字串轉換為參考屬性

**語法：**  
`ref CRef(str value)`

**範例：**  
`CRef("CN=LC Services,CN=Microsoft,CN=lcspool01,CN=Pools,CN=RTC Service," & %Forest.LDAP%)`

- - -
### <a name="cstr"></a>CStr
**說明：**  
CStr 函式會轉換為字串資料類型。

**語法：**  
`str CStr(num value)`  
`str CStr(ref value)`  
`str CStr(bool value)`  

* value：可以是數值、參考屬性或布林值。

**範例：**  
`CStr([dn])`  
可能傳回 "cn=Joe,dc=contoso,dc=com"

- - -
### <a name="dateadd"></a>DateAdd
**說明：**  
傳回日期，其中包含已新增指定時間間隔的日期。

**語法：**  
`dt DateAdd(str interval, num value, dt date)`

* interval：字串運算式，此為您想要加入的時間間隔。 字串必須具有下列其中一個值：
  * yyyy 年
  * q 季
  * m 月
  * y 一年中第幾天
  * d 日
  * w 工作日
  * ww 周
  * h 小時
  * n 分鐘
  * s 秒
* value：您想要加入的單位數。 它可以是正數 (用以取得未來的日期) 或負數 (用以取得過去的日期)。
* date：DateTime 代表要加入間隔的日期。

**範例：**  
`DateAdd("m", 3, CDate("2001-01-01"))`  
新增 3 個月，並傳回代表 "2001-04-01" 的 DateTime。

- - -
### <a name="datefromnum"></a>DateFromNum
**說明：**  
DateFromNum 函式會將 AD 日期格式的值轉換為 DateTime 類型。

**語法：**  
`dt DateFromNum(num value)`

**範例：**  
`DateFromNum([lastLogonTimestamp])`  
`DateFromNum(129699324000000000)`  
傳回代表 2012-01-01 23:00:00 的 DateTime

- - -
### <a name="dncomponent"></a>DNComponent
**說明：**  
DNComponent 函式會從左邊傳回指定 DN 元件的值。

**語法：**  
`str DNComponent(ref dn, num ComponentNumber)`

* dn：要解譯的參考屬性
* ComponentNumber：DN 中要傳回的元件

**範例：**  
`DNComponent(CRef([dn]),1)`  
如果 dn 為 "cn=Joe,ou=…"，就會傳回 Joe

- - -
### <a name="dncomponentrev"></a>DNComponentRev
**說明：**  
DNComponentRev 函式會從右邊 (結尾處) 傳回指定 DN 元件的值。

**語法：**  
`str DNComponentRev(ref dn, num ComponentNumber)`  
`str DNComponentRev(ref dn, num ComponentNumber, enum Options)`

* dn：要解譯的參考屬性
* ComponentNumber - DN 中要傳回的元件
* Options：DC - 忽略所有含 "dc=" 的元件

**範例：**  
如果 dn 是 "cn=Joe,ou=Atlanta,ou=GA,ou=US, dc=contoso,dc=com"，則  
`DNComponentRev(CRef([dn]),3)`  
`DNComponentRev(CRef([dn]),1,"DC")`  
兩者都傳回 US。

- - -
### <a name="error"></a>Error
**說明：**  
Error 函式是用來傳回自訂錯誤。

**語法：**  
`void Error(str ErrorMessage)`

**範例：**  
`IIF(IsPresent([accountName]),[accountName],Error("AccountName is required"))`  
如果 accountName 屬性不存在，即會擲回有關物件的錯誤。

- - -
### <a name="escapedncomponent"></a>EscapeDNComponent
**說明：**  
EscapeDNComponent 函式會接受 DN 的一個元件並逸出它，以便在 LDAP 中顯示它。

**語法：**  
`str EscapeDNComponent(str value)`

**範例：**  
`EscapeDNComponent("cn=" & [displayName]) & "," & %ForestLDAP%)`  
確保即使 displayName 屬性具有必須在 LDAP 中逸出的字元，還是能夠在 LDAP 目錄中建立物件。

- - -
### <a name="formatdatetime"></a>FormatDateTime
**說明：**  
FormatDateTime 函式可用來將 DateTime 格式化為具有指定格式的字串

**語法：**  
`str FormatDateTime(dt value, str format)`

* value：DateTime 格式的值
* format：字串，表示要轉換的目標格式。

**備註：**  
您可以在此處找到格式的可能值：[使用者定義日期/時間格式 (Format 函式)](http://msdn2.microsoft.com/library/73ctwf33\(VS.90\).aspx)

**範例：**  

`FormatDateTime(CDate("12/25/2007"),"yyyy-mm-dd")`  
結果是 "2007-12-25"。

`FormatDateTime(DateFromNum([pwdLastSet]),"yyyyMMddHHmmss.0Z")`  
會產生 "20140905081453.0Z"

- - -
### <a name="guid"></a>Guid
**說明：**  
函式 GUID 會產生新的隨機 GUID

**語法：**  
`str Guid()`

- - -
### <a name="iif"></a>IIF
**說明：**  
IIF 函式會根據指定的條件傳回其中一組可能值。

**語法：**  
`var IIF(exp condition, var valueIfTrue, var valueIfFalse)`

* condition：可評估為 True 或 False 的任何值或運算式。
* valueIfTrue：條件評估為 True 時所傳回的值。
* valueIfFalse：條件評估為 false 時所傳回的值。

**範例：**  
`IIF([employeeType]="Intern","t-" & [alias],[alias])`  
 如果使用者是實習生，就會傳回開頭加上 "t-" 的使用者別名，否則會依原樣傳回使用者的別名。

- - -
### <a name="instr"></a>InStr
**說明：**  
InStr 函式會在字串中尋找第一個出現的子字串

**語法：**  

`num InStr(str stringcheck, str stringmatch)`  
`num InStr(str stringcheck, str stringmatch, num start)`  
`num InStr(str stringcheck, str stringmatch, num start , enum compare)`

* stringcheck：要搜尋的字串
* stringmatch：要尋找的字串
* start：開始尋找子字串的位置
* compare：vbTextCompare 或 vbBinaryCompare

**備註：**  
會傳回找到子字串的位置，如果找不到，則傳回 0。

**範例：**  
`InStr("The quick brown fox","quick")`  
評估為 5

`InStr("repEated","e",3,vbBinaryCompare)`  
評估為 7

- - -
### <a name="instrrev"></a>InStrRev
**說明：**  
InStrRev 函式會在字串中尋找最後一個出現的子字串

**語法：**  
`num InstrRev(str stringcheck, str stringmatch)`  
`num InstrRev(str stringcheck, str stringmatch, num start)`  
`num InstrRev(str stringcheck, str stringmatch, num start, enum compare)`

* stringcheck：要搜尋的字串
* stringmatch：要尋找的字串
* start：開始尋找子字串的位置
* compare：vbTextCompare 或 vbBinaryCompare

**備註：**  
會傳回找到子字串的位置，如果找不到，則傳回 0。

**範例：**  
`InStrRev("abbcdbbbef","bb")`  
傳回 7

- - -
### <a name="isbitset"></a>IsBitSet
**說明：**  
IsBitSet 函式會測試是否已設定位元

**語法：**  
`bool IsBitSet(num value, num flag)`

* value：評估的數值。flag：具有要評估之位元的數值

**範例：**  
`IsBitSet(&HF,4)`  
因為位元 "4" 是使用十六進位值 "F" 所設定，所以會傳回 True

- - -
### <a name="isdate"></a>IsDate
**說明：**  
如果運算式可評估為 DateTime 類型，則 IsDate 函式會評估為 True。

**語法：**  
`bool IsDate(var Expression)`

**備註：**  
用來判斷 CDate() 是否可能成功。

- - -
### <a name="iscert"></a>IsCert
**說明：**  
如果原始資料可以序列化為 .NET X509Certificate2 憑證物件，則傳回 true。

**語法：**  
`bool CertThumbprint(binary certificateRawData)`  
*   certificateRawData：X.509 憑證的位元組陣列表示。 位元組陣列可以是編碼的二進位 (DER) 或 Base64 編碼的 X.509 資料。
- - -
### <a name="isempty"></a>IsEmpty
**說明：**  
如果屬性存在於 CS 或 MV 中，但評估為空字串，則 IsEmpty 函式會評估為 True。

**語法：**  
`bool IsEmpty(var Expression)`

- - -
### <a name="isguid"></a>IsGuid
**說明：**  
如果字串可轉換為 GUID，則 IsGuid 函式會評估為 True。

**語法：**  
`bool IsGuid(str GUID)`

**備註：**  
GUID 定義為下列其中一種模式的字串： xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx 或 {xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}

用來判斷 CGuid() 是否可能成功。

**範例：**  
`IIF(IsGuid([strAttribute]),CGuid([strAttribute]),NULL)`  
如果 StrAttribute 具有 GUID 格式，即會傳回二進位表示法，否則會傳回 Null。

- - -
### <a name="isnull"></a>IsNull
**說明：**  
如果運算式評估為 Null，則 IsNull 函式會傳回 True。

**語法：**  
`bool IsNull(var Expression)`

**備註：**  
針對屬性，Null 表示該屬性不存在。

**範例：**  
`IsNull([displayName])`  
如果屬性不存在於 CS 或 MV 中，即會傳回 True。

- - -
### <a name="isnullorempty"></a>IsNullOrEmpty
**說明：**  
如果運算式為 Null 或空字串，則 IsNullOrEmpty 函式會傳回 True。

**語法：**  
`bool IsNullOrEmpty(var Expression)`

**備註：**  
針對屬性，如果屬性不存在，或存在但為空字串，即會評估為 True。  
此函式的相反函式名稱為 IsPresent。

**範例：**  
`IsNullOrEmpty([displayName])`  
如果屬性不存在於 CS 或 MV 中或為空字串，即會傳回 True。

- - -
### <a name="isnumeric"></a>IsNumeric
**說明：**  
IsNumeric 函式會傳回布林值，指出運算式是否可評估為數字類型。

**語法：**  
`bool IsNumeric(var Expression)`

**備註：**  
用來判斷 CNum() 是否可成功剖析運算式。

- - -
### <a name="isstring"></a>IsString
**說明：**  
如果運算式可評估為字串類型，則 IsString 函式會評估為 True。

**語法：**  
`bool IsString(var expression)`

**備註：**  
用來判斷 CStr() 是否可成功剖析運算式。

- - -
### <a name="ispresent"></a>IsPresent
**說明：**  
如果運算式評估為非 Null 且不是空字串，則 IsPresent 函式會傳回 True。

**語法：**  
`bool IsPresent(var expression)`

**備註：**  
這個函式的相反函式名稱為 IsNullOrEmpty。

**範例：**  
`Switch(IsPresent([directManager]),[directManager], IsPresent([skiplevelManager]),[skiplevelManager], IsPresent([director]),[director])`

- - -
### <a name="item"></a>Item
**說明：**  
Item 函式會從多重值字串/屬性傳回一個項目。

**語法：**  
`var Item(mvstr attribute, num index)`

* attribute：多重值的屬性
* index：多重值字串中項目的索引。

**備註：**  
Item 函式可以與 Contains 函式搭配使用，因為後者會將索引傳回多重值屬性中的項目。

如果索引超出範圍，即會擲回錯誤。

**範例：**  
`Mid(Item([proxyAddresses],Contains([proxyAddresses], "SMTP:")),6)`  
會傳回主要電子郵件地址。

- - -
### <a name="itemornull"></a>ItemOrNull
**說明：**  
ItemOrNull 函式會從多重值字串/屬性傳回一個項目。

**語法：**  
`var ItemOrNull(mvstr attribute, num index)`

* attribute：多重值的屬性
* index：多重值字串中項目的索引。

**備註：**  
ItemOrNull 函式可以與 Contains 函式搭配使用，因為後者會將索引傳回多重值屬性中的項目。

如果索引超出範圍，則會傳回 Null 值。

- - -
### <a name="join"></a>Join
**說明：**  
Join 函式會接受多重值的字串，並傳回單一值的字串，其中每個項目之間都插入指定的分隔符號。

**語法：**  
`str Join(mvstr attribute)`  
`str Join(mvstr attribute, str Delimiter)`

* attribute：包含要聯結之字串的多重值屬性。
* delimiter：任何字串，可用來分隔傳回字串中的子字串。 如果省略，即會使用空格字元 (" ")。 如果分隔符號是零長度字串 ("") 或 Nothing，就不會使用分隔符號來串連清單中的所有項目。

**備註**  
Join 和 Split 函式之間有同位。 Join 函式可接受字串陣列，並使用分隔符號字串來聯結它們，以傳回單一字串。 Split 函式會取得字串並以分隔符號來分隔，以傳回字串陣列。 不過，主要的差別是 Join 可以使用任何分隔符號字串來串連字串，Split 只能使用單一字元分隔符號來分隔字串。

**範例：**  
`Join([proxyAddresses],",")`  
可能傳回："SMTP:john.doe@contoso.com,smtp:jd@contoso.com"

- - -
### <a name="lcase"></a>LCase
**說明：**  
LCase 函式會將字串中的所有字元轉換為小寫。

**語法：**  
`str LCase(str value)`

**範例：**  
`LCase("TeSt")`  
傳回 "test"。

- - -
### <a name="left"></a>Left
**說明：**  
Left 函式會從字串左邊傳回指定的字元數。

**語法：**  
`str Left(str string, num NumChars)`

* string：要傳回字元的字串
* NumChars：數字，識別從 string 開頭 (左邊) 傳回的字元數

**備註：**  
包含 string 中前 numChars 個字元的字串：

* 如果 numChars = 0，會傳回空字串。
* 如果 numChars < 0，會傳回輸入字串。
* 如果 string 為 Null，會傳回空字串。

如果 string 包含的字元數比 numChars 中指定的數目少，即會傳回與 string 完全相同的字串 (也就是，包含參數 1 中的所有字元)。

**範例：**  
`Left("John Doe", 3)`  
傳回 "Joh"。

- - -
### <a name="len"></a>Len
**說明：**  
Len 函式會傳回字串中的字元數。

**語法：**  
`num Len(str value)`

**範例：**  
`Len("John Doe")`  
傳回 8

- - -
### <a name="ltrim"></a>LTrim
**說明：**  
LTrim 函式會從字串中移除開頭空白字元。

**語法：**  
`str LTrim(str value)`

**範例：**  
`LTrim(" Test ")`  
傳回 "Test"

- - -
### <a name="mid"></a>Mid
**說明：**  
Mid 函式會從字串中的指定位置傳回指定的字元數。

**語法：**  
`str Mid(str string, num start, num NumChars)`

* string：要傳回字元的字串
* start：數字，可識別 string 中要傳回字元的起始位置
* NumChars：數字，可識別要從 string 中的位置傳回的字元數

**備註：**  
從 string 中的位置 start 開始傳回 numChars 個字元。  
包含從 string 中的位置 start 起算的 numChars 個字元的字串：

* 如果 numChars = 0，會傳回空字串。
* 如果 numChars < 0，會傳回輸入字串。
* 如果 start > 字串的長度，會傳回輸入字串。
* 如果 start < = 0，會傳回輸入字串。
* 如果 string 為 Null，會傳回空字串。

如果 string 中從位置 start 起算所剩餘的字元數不足 numChar 個字元，即會盡可能地傳回許多字元。

**範例：**  
`Mid("John Doe", 3, 5)`  
傳回 "hn Do"。

`Mid("John Doe", 6, 999)`  
傳回 "Doe"

- - -
### <a name="now"></a>Now
**說明：**  
Now 函式會根據您電腦的系統日期和時間，傳回指定目前日期和時間的 DateTime。

**語法：**  
`dt Now()`

- - -
### <a name="numfromdate"></a>NumFromDate
**說明：**  
NumFromDate 函式會以 AD 的日期格式傳回日期。

**語法：**  
`num NumFromDate(dt value)`

**範例：**  
`NumFromDate(CDate("2012-01-01 23:00:00"))`  
傳回 129699324000000000

- - -
### <a name="padleft"></a>PadLeft
**說明：**  
PadLeft 函式會使用提供的填補字元，將字串左側填補到指定的長度。

**語法：**  
`str PadLeft(str string, num length, str padCharacter)`

* string：要填補的字串。
* length：表示 string 所需長度的整數。
* padCharacter：字串，由用來做為填補字元的單一字元所組成

**備註：**

* 如果 string 的長度小於 length，則會將 padCharacter 重複附加至 string 的開頭 (左邊)，直到它的長度等於 length 為止。
* PadCharacter 可以是空白字元，但不能是 Null 值。
* 如果 string 的長度等於或大於 length，就會以原樣傳回 string。
* 如果 string 的長度大於或等於 length，即會傳回與 string 完全相同的字串。
* 如果 string 的長度小於 length，則會傳回所需長度的新字串，包含使用 padCharacter 填補的 string。
* 如果 string 為 Null，函式即會傳回空字串。

**範例：**  
`PadLeft("User", 10, "0")`  
傳回 "000000User"。

- - -
### <a name="padright"></a>PadRight
**說明：**  
PadRight 函式會使用提供的填補字元，將字串右側填補到指定的長度。

**語法：**  
`str PadRight(str string, num length, str padCharacter)`

* string：要填補的字串。
* length：表示 string 所需長度的整數。
* padCharacter：字串，由用來做為填補字元的單一字元所組成

**備註：**

* 如果 string 的長度小於 length，則會將 padCharacter 重複附加至 string 的結尾 (右邊)，直到它的長度等於 length 為止。
* PadCharacter 可以是空白字元，但不能是 Null 值。
* 如果 string 的長度等於或大於 length，就會以原樣傳回 string。
* 如果 string 的長度大於或等於 length，即會傳回與 string 完全相同的字串。
* 如果 string 的長度小於 length，則會傳回所需長度的新字串，包含使用 padCharacter 填補的 string。
* 如果 string 為 Null，函式即會傳回空字串。

**範例：**  
`PadRight("User", 10, "0")`  
傳回 "User000000"。

- - -
### <a name="pcase"></a>PCase
**說明：**  
PCase 函式會將字串中每個空格分隔之單字的第一個字元轉換為大寫，並將其他所有字元轉換為小寫。

**語法：**  
`String PCase(string)`

**備註：**

* 此函式目前未提供可轉換全大寫文字 (例如縮略字) 的正確大小寫。

**範例：**  
`PCase("TEsT")`  
傳回 "test"。

`PCase(LCase("TEST"))`  
傳回 "Test"

- - -
### <a name="randomnum"></a>RandomNum
**說明：**  
RandomNum 函式會傳回指定區間內的隨機數字。

**語法：**  
`num RandomNum(num start, num end)`

* start：可識別要產生之隨機值下限的數字
* end：可識別要產生之隨機值上限的數字

**範例：**  
`Random(100,999)`  
可傳回 734。

- - -
### <a name="removeduplicates"></a>RemoveDuplicates
**說明：**  
RemoveDuplicates 函式會接受多重值的字串，並確定每個值都是唯一的。

**語法：**  
`mvstr RemoveDuplicates(mvstr attribute)`

**範例：**  
`RemoveDuplicates([proxyAddresses])`  
傳回處理過的 proxyAddress 屬性，其中已移除所有重複的值。

- - -
### <a name="replace"></a>Replace
**說明：**  
Replace 函式會將所有出現的字串取代為另一個字串。

**語法：**  
`str Replace(str string, str OldValue, str NewValue)`

* string：要取代其值的字串。
* OldValue：要搜尋並取代的字串。
* NewValue：要取代的字串。

**備註：**  
這個函式會辨識下列特殊的 Moniker：

* \n – 新行
* \r – 歸位字元
* \t – 定位字元

**範例：**  
`Replace([address],"\r\n",", ")`  
使用逗號和空格來取代 CRLF，並且可產生 "One Microsoft Way, Redmond, WA, USA"

- - -
### <a name="replacechars"></a>ReplaceChars
**說明：**  
ReplaceChars 函式會取代 ReplacePattern 字串中找到的所有出現的字元。

**語法：**  
`str ReplaceChars(str string, str ReplacePattern)`

* string：要取代字元的字串。
* ReplacePattern：字串，包含要取代之字元的字典。

格式為 {source1}:{target1},{source2}:{target2},{sourceN},{targetN}，其中 source 為要尋找的字元，而 target 則為要取代的字串。

**備註：**

* 此函式會取得已定義之來源的每個出現項目，並使用目標來取代它們。
* 來源必須是一個 (unicode) 字元。
* 來源不能是空字串或超過一個字元 (剖析錯誤)。
* 目標可以有多個字元，例如，ö:oe、β:ss。
* 目標可以空白，表示應移除此字元。
* 來源是區分大小寫且必須完全相符。
* , (逗號) 和 : (冒號) 是保留字元，無法使用這個函式來取代。
* 系統會忽略空格和 ReplacePattern 字串中的其他空白字元。

**範例：**  
`%ReplaceString% = ’:,Å:A,Ä:A,Ö:O,å:a,ä:a,ö,o`

`ReplaceChars("Räksmörgås",%ReplaceString%)`  
傳回 Raksmorgas

`ReplaceChars("O’Neil",%ReplaceString%)`  
傳回 "ONeil"，已將單一撇號定義為要移除。

- - -
### <a name="right"></a>Right
**說明：**  
Right 函式會從字串右邊 (結尾處) 傳回指定的字元數。

**語法：**  
`str Right(str string, num NumChars)`

* string：要傳回字元的字串
* NumChars：數字，可識別要從 string 結尾 (右邊) 傳回的字元數

**備註：**  
會從 string 的最後一個位置傳回 NumChars 個字元。

包含 string 中最後 numChars 個字元的字串：

* 如果 numChars = 0，會傳回空字串。
* 如果 numChars < 0，會傳回輸入字串。
* 如果 string 為 Null，會傳回空字串。

如果 string 包含的字元數比 numChars 中指定的數目少，即會傳回與 string 完全相同的字串。

**範例：**  
`Right("John Doe", 3)`  
傳回 "Doe"。

- - -
### <a name="rtrim"></a>RTrim
**說明：**  
RTrim 函式會從字串移除結尾空白字元。

**語法：**  
`str RTrim(str value)`

**範例：**  
`RTrim(" Test ")`  
傳回 "Test"。

- - -
### <a name="select"></a>選取
**說明：**  
在以指定函式為基礎的多重值屬性 (或運算式的輸出) 中處理所有值。

**語法：**  
`mvattr Select(variable item, mvattr attribute, func function)`  
`mvattr Select(variable item, exp expression, func function)`

* item：表示多重值屬性中的項目
* attribute：多重值屬性
* expression：傳回值集合的運算式
* condition：可以在屬性中處理項目的任何函式

**範例：**  
`Select($item,[otherPhone],Replace($item,"-",""))`  
傳回移除連字號 (-) 後，多重值屬性 otherPhone 中所有值。

- - -
### <a name="split"></a>Split
**說明：**  
Split 函式會接受以分隔符號分隔的字串，並使其成為多重值的字串。

**語法：**  
`mvstr Split(str value, str delimiter)`  
`mvstr Split(str value, str delimiter, num limit)`

* value：以分隔符號字元分隔的字串。
* delimiter：用來做為分隔符號的單一字元。
* limit：可以傳回的值數目上限。

**範例：**  
`Split("SMTP:john.doe@contoso.com,smtp:jd@contoso.com",",")`  
傳回多重值的字串，其中包含 2 個 proxyAddress 屬性可用的元素。

- - -
### <a name="stringfromguid"></a>StringFromGuid
**說明：**  
StringFromGuid 函式會接受二進位 GUID，並將其轉換為字串

**語法：**  
`str StringFromGuid(bin GUID)`

- - -
### <a name="stringfromsid"></a>StringFromSid
**說明：**  
StringFromSid 函式會將包含安全性識別碼的位元組陣列轉換為字串。

**語法：**  
`str StringFromSid(bin ObjectSID)`  

- - -
### <a name="switch"></a>Switch
**說明：**  
Switch 函式可用來根據評估的條件傳回單一值。

**語法：**  
`var Switch(exp expr1, var value1[, exp expr2, var value … [, exp expr, var valueN]])`

* expr：您想要評估的 Variant 運算式。
* value：當對應的運算式為 True 時要傳回的值。

**備註：**  
Switch 函式引數清單是由運算式和值的配對所組成。 運算式是以從左到右的方式進行評估，並會傳回與要評估為 True 的第一個運算式相關聯的值。 如果未正確配對組件，就會發生執行階段錯誤。

例如，如果 expr1 為 True，Switch 就會傳回 value1。 如果 expr-1 為 False，但 expr-2 為 True，Switch 就會傳回 value-2，依此類推。

在下列情況下，參數會傳回 Nothing︰

* 沒有任何運算式為 True。
* 第一個 True 運算式具有對應值 Null。

雖然 Switch 只會傳回其中一個運算式，但它會評估所有運算式。 基於這個理由，您應該留意非預期的副作用。 例如，如果任何運算式的評估會產生除以零的錯誤，就會發生錯誤。

Value 也可以是會傳回自訂字串的 Error 函式。

**範例：**  
`Switch([city] = "London", "English", [city] = "Rome", "Italian", [city] = "Paris", "French", True, Error("Unknown city"))`  
傳回一些主要城市中所使用的語言，否則會傳回錯誤。

- - -
### <a name="trim"></a>Trim
**說明：**  
Trim 函式會從字串移除開頭和結尾的空白字元。

**語法：**  
`str Trim(str value)`  

**範例：**  
`Trim(" Test ")`  
傳回 "test"。

`Trim([proxyAddresses])`  
會移除 proxyAddress 屬性中每個值的開頭和結尾空格。

- - -
### <a name="ucase"></a>UCase
**說明：**  
UCase 函式會將字串中的所有字元轉換為大寫。

**語法：**  
`str UCase(str string)`

**範例：**  
`UCase("TeSt")`  
傳回 "test"。

- - -
### <a name="where"></a>Where

**說明：**  
傳回以指定條件為基礎的多重值屬性 (或運算式的輸出) 的值子集。

**語法：**  
`mvattr Where(variable item, mvattr attribute, exp condition)`  
`mvattr Where(variable item, exp expression, exp condition)`  
* item：表示多重值屬性中的項目
* attribute：多重值屬性
* condition：可評估為 True 或 False 的任何運算式
* expression：傳回值集合的運算式

**範例：**  
`Where($item,[userCertificate],CertNotAfter($item)>Now())`  
傳回未過期的多重值屬性 userCertificate 中的憑證值。

- - -
### <a name="with"></a>With
**說明：**  
With 函式可以簡化複雜的運算式，使用變數代表在複雜運算式中會出現一或多次的子運算式。

**語法：**
`With(var variable, exp subExpression, exp complexExpression)`  
* variable：代表子運算式。
* subExpression：變數代表的子運算式。
* complexExpression：複雜的運算式。

**範例：**  
`With($unExpiredCerts,Where($item,[userCertificate],CertNotAfter($item)>Now()),IIF(Count($unExpiredCerts)>0,$unExpiredCerts,NULL))`  
在功能上等同於：  
`IIF (Count(Where($item,[userCertificate],CertNotAfter($item)>Now()))>0, Where($item,[userCertificate],CertNotAfter($item)>Now()),NULL)`  
只傳回 UserCertificate 屬性中未過期的憑證值。


- - -
### <a name="word"></a>Word
**說明：**  
Word 函式會根據描述要使用之分隔符號及要傳回之字數的參數，傳回字串內含的單字。

**語法：**  
`str Word(str string, num WordNumber, str delimiters)`

* string：要傳回單字的字串
* WordNumber：一個數字，用於識別可傳回的字數。
* delimiters：字串，表示應用來識別單字的分隔符號

**備註：**  
string 內以 delimiters 其中一個字元來分隔之字元的每個字串，都會被識別為單字：

* 如果 number < 1，會傳回空字串。
* 如果 string 為 Null，會傳回空字串。

如果 string 所含的字數少於 number 個字，或者 string 不包含任何 delimeters 所識別的單字，就會傳回空字串。

**範例：**  
`Word("The quick brown fox",3," ")`  
傳回 "brown"

`Word("This,string!has&many separators",3,",!&#")`  
會傳回 "has"

## <a name="additional-resources"></a>其他資源
* [了解宣告式佈建運算式](concept-azure-ad-connect-sync-declarative-provisioning-expressions.md)
* [Azure AD Connect 同步處理：自訂同步處理選項](how-to-connect-sync-whatis.md)
* [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)
