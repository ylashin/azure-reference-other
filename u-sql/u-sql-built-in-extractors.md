---
title: "U-SQL Built-in Extractors | Microsoft Docs"
ms.custom: ""
ms.date: "03/28/2017"
ms.reviewer: ""
ms.service: "data-lake-analytics"
ms.suite: ""
ms.tgt_pltfrm: ""
ms.topic: "reference"
ms.assetid: c28b8199-bd14-4b6d-8d03-691bd4819be1
caps.latest.revision: 21
author: "MikeRys"
ms.author: "mrys"
manager: "ryanw"
---
# U-SQL Built-in Extractors
U-SQL provides a built-in extractor class called `Extractors` that provides the following three built-in extractors to generate a rowset from the [input file or files](input-files-u-sql.md):    
-   [Extractors.Text()](extractors-text.md) : Provides extraction from delimited text files of different encodings.    
-   [Extractors.Csv()](extractors-csv.md) : Provides extraction from comma-separated value (CSV) files of different encodings.    
-   [Extractors.Tsv()](extractors-tsv.md) : Provides extraction from tab-separated value (TSV) files of different encodings.  
  
Technically speaking these are factory methods that generate an instance of the `IExtractor` class and they can be used in the [USING](extract-expression-u-sql.md#us_cl) clause of the [`EXTRACT`](extract-expression-u-sql.md) expression. Since they create the extractor object, one does not need to call them with new.  
  
The `Csv()` and `Tsv()` extractors are special versions of the generic `Text()` extractor where the delimiter has been fixed to comma and tab respectively.  

If the [EXTRACT](extract-expression-u-sql.md) expression specifies a file set pattern, then the extractor [parameters](extractor-parameters-u-sql.md) will be applied to all the selected files equally. If different files require different parameter values, then different [EXTRACT](extract-expression-u-sql.md) expressions need to be used.  
  
### Built-in Extractor Processing Model    
The built-in extractors transforms a byte stream in parallel into a rowset that can be further processed with U-SQL statements. The following figure provides a logical view of the processing model (that in turn is based on the general [UDO Extractor](https://docs.microsoft.com/azure/data-lake-analytics/data-lake-analytics-u-sql-programmability-guide#user-defined-extractor)
 processing model).  
 
![ExtractorProcessingModel](media/extractorprocessingmodel.png)  
  
If the maximal input row length or the maximal output row length are being exceeded, errors are raised.  
  
### Built-in Type Conversions    
The extractors will convert the string values `val` in the stream to an instance of the specified type in the extractor schema after the processing of encoding and escaped values have occurred.  
  
Per default, an empty field is mapped to a zero-length string if target type is string and null otherwise. Most types follow the standard C# `<Type>.Parse(val)` behaviour without a specific culture behaviour and being defaulted to the cluster machine’s locale. The table below provides more details for each type.  
  
> [!TIP]
> Since the built-in extractors are implemented natively, the conversions may differ in small details from the U-SQL/C# conversion semantics. Some differences cannot be avoided, primarily around floating point values where minute differences may be present.  
  
The following data types (and their nullable variant) are supported by the built-in extractors. Any data type that is not listed and is not supported by the extractor (such as [SQL.MAP](complex-built-in-u-sql-types.md)  and [SQL.ARRAY](complex-built-in-u-sql-types.md)) either needs to be converted in a subsequent SELECT statement or a [user-defined extractor](https://docs.microsoft.com/azure/data-lake-analytics/data-lake-analytics-u-sql-programmability-guide#user-defined-extractor) has to be written.  

|Type|Conversion|  
|---|---|  
|`bool`|`bool.Parse(val)`.<br /> `val` can be either "true" or "false" or any casing thereof and maps to true or false respectively. |  
|`sbyte`|`sbyte.Parse(val)` with the following `NumberFormatInfo` and `NumberStyles`:<br />&emsp;&#9679;&emsp;Cultural locale is "en-us", which includes<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberFormatInfo.PositiveSign:  '+'`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberFormatInfo.NegativeSign:  '-'`<br />&emsp;&#9679;&emsp;`NumberStyle.Integer` that includes:<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowLeadingWhite`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowTrailingWhite`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowLeadingSign`<br />In particular, the following `NumberStyles` are **not** supported:<br />&emsp;&#9679;&emsp;`NumberStyles.AllowCurrencySymbol`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowTrailingSign`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowParentheses`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowThousands`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowExponent`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowDecimalPoint`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowHexSpecifier`<br />resulting in the following grammar for the supported numbers lexical representations:<hr />`[whitespace] [sign] [digits] [whitespace]`<hr />Note there is no whitespace allowed between the sign and the digits. If `val` satisfy the grammar and fits into the range of the type, it will successfully converted. Otherwise appropriate runtime errors will be raised.|
`byte`|`byte.Parse(val)` with the following `NumberFormatInfo` and `NumberStyles`:<br />&emsp;&#9679;&emsp;Cultural locale is "en-us", which includes<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberFormatInfo.PositiveSign: '+'`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberFormatInfo.NegativeSign: N/A`<br />&emsp;&#9679;&emsp;`NumberStyle.Integer` that includes:<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowLeadingWhite`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowTrailingWhite`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowLeadingSign`<br />In particular, the following `NumberStyles` are **not** supported:<br />&emsp;&#9679;&emsp;`NumberStyles.AllowCurrencySymbol`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowTrailingSign`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowParentheses`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowThousands`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowExponent`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowDecimalPoint`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowHexSpecifier`<br />resulting in the following grammar for the supported numbers lexical representations:<hr />`[whitespace] [sign] [digits] [whitespace]`<hr />Where sign is only '+'. Note there is no whitespace allowed between the sign and the digits. If `val` satisfy the grammar and fits into the range of the type, it will successfully converted. Otherwise apropriate runtime errors will be raised.|
|`short`|`short.Parse(val)` with the following `NumberFormatInfo` and `NumberStyles`:<br />&emsp;&#9679;&emsp;Cultural locale is "en-us", which includes<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberFormatInfo.PositiveSign: '+'`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberFormatInfo.NegativeSign: '-'`<br />&emsp;&#9679;&emsp;`NumberStyle.Integer` that includes:<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowLeadingWhite`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowTrailingWhite`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowLeadingSign`<br />In particular, the following `NumberStyles` are **not** supported:<br />&emsp;&#9679;&emsp;`NumberStyles.AllowCurrencySymbol`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowTrailingSign`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowParentheses`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowThousands`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowExponent`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowDecimalPoint`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowHexSpecifier`<br />resulting in the following grammar for the supported numbers lexical representations:<hr />`[whitespace] [sign] [digits] [whitespace]`<hr />Note there is no whitespace allowed between the sign and the digits. If `val` satisfy the grammar and fits into the range of the type, it will successfully converted. Otherwise apropriate runtime errors will be raised.|
|`ushort`|`ushort.Parse(val)` with the following `NumberFormatInfo` and `NumberStyles`:<br />&emsp;&#9679;&emsp;Cultural locale is "en-us", which includes<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberFormatInfo.PositiveSign: '+'`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberFormatInfo.NegativeSign: N/A`<br />&emsp;&#9679;&emsp;`NumberStyle.Integer` that includes:<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowLeadingWhite`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowTrailingWhite`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowLeadingSign`<br />In particular, the following `NumberStyles` are **not** supported:<br />&emsp;&#9679;&emsp;`NumberStyles.AllowCurrencySymbol`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowTrailingSign`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowParentheses`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowThousands`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowExponent`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowDecimalPoint`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowHexSpecifier`<br />resulting in the following grammar for the supported numbers lexical representations:<hr />`[whitespace] [sign] [digits] [whitespace]`<hr />Where sign is only '+'. Note there is no whitespace allowed between the sign and the digits. If `val` satisfy the grammar and fits into the range of the type, it will successfully converted. Otherwise apropriate runtime errors will be raised.|  
|`int`|`int.Parse(val)` with the following `NumberFormatInfo` and `NumberStyles`:<br />&emsp;&#9679;&emsp;Cultural locale is "en-us", which includes<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberFormatInfo.PositiveSign: '+'`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberFormatInfo.NegativeSign: '-'`<br />&emsp;&#9679;&emsp;`NumberStyle.Integer` that includes:<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowLeadingWhite`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowTrailingWhite`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowLeadingSign`<br />In particular, the following `NumberStyles` are **not** supported:<br />&emsp;&#9679;&emsp;`NumberStyles.AllowCurrencySymbol`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowTrailingSign`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowParentheses`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowThousands`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowExponent`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowDecimalPoint`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowHexSpecifier`<br />resulting in the following grammar for the supported numbers lexical representations:<hr />`[whitespace] [sign] [digits] [whitespace]`<hr />Note there is no whitespace allowed between the sign and the digits.  If `val` satisfy the grammar and fits into the range of the type, it will successfully converted. Otherwise apropriate runtime errors will be raised.|
|`uint`|`uint.Parse(val)` with the following `NumberFormatInfo` and `NumberStyles`:<br />&emsp;&#9679;&emsp;Cultural locale is "en-us", which includes<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberFormatInfo.PositiveSign: '+'`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberFormatInfo.NegativeSign: N/A`<br />&emsp;&#9679;&emsp;`NumberStyle.Integer` that includes:<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowLeadingWhite`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowTrailingWhite`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowLeadingSign`<br />In particular, the following `NumberStyles` are **not** supported:<br />&emsp;&#9679;&emsp;`NumberStyles.AllowCurrencySymbol`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowTrailingSign`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowParentheses`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowThousands`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowExponent`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowDecimalPoint`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowHexSpecifier`<br />resulting in the following grammar for the supported numbers lexical representations:<hr />`[whitespace] [sign] [digits] [whitespace]`<hr />Where sign is only '+'.  Note there is no whitespace allowed between the sign and the digits.  If `val` satisfy the grammar and fits into the range of the type, it will successfully converted. Otherwise apropriate runtime errors will be raised.|
|`long`|`long.Parse(val)` with the following `NumberFormatInfo` and `NumberStyles`:<br />&emsp;&#9679;&emsp;Cultural locale is "en-us", which includes<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberFormatInfo.PositiveSign: '+'`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberFormatInfo.NegativeSign: '-'`<br />&emsp;&#9679;&emsp;`NumberStyle.Integer` that includes:<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowLeadingWhite`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowTrailingWhite`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowLeadingSign`<br />In particular, the following `NumberStyles` are **not** supported:<br />&emsp;&#9679;&emsp;`NumberStyles.AllowCurrencySymbol`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowTrailingSign`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowParentheses`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowThousands`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowExponent`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowDecimalPoint`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowHexSpecifier`<br />resulting in the following grammar for the supported numbers lexical representations:<hr />`[whitespace] [sign] [digits] [whitespace]`<hr />Note there is no whitespace allowed between the sign and the digits.  If `val` satisfy the grammar and fits into the range of the type, it will successfully converted. Otherwise apropriate runtime errors will be raised.|
|`ulong`|`ulong.Parse(val)` with the following `NumberFormatInfo` and `NumberStyles`:<br />&emsp;&#9679;&emsp;Cultural locale is "en-us", which includes<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberFormatInfo.PositiveSign: '+'`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberFormatInfo.NegativeSign: N/A`<br />&emsp;&#9679;&emsp;`NumberStyle.Integer` that includes:<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowLeadingWhite`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowTrailingWhite`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowLeadingSign`<br />In particular, the following `NumberStyles` are **not** supported:<br />&emsp;&#9679;&emsp;`NumberStyles.AllowCurrencySymbol`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowTrailingSign`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowParentheses`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowThousands`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowExponent`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowDecimalPoint`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowHexSpecifier`<br />resulting in the following grammar for the supported numbers lexical representations:<hr />`[whitespace] [sign] [digits] [whitespace]`<hr />Where sign is only '+'.  Note there is no whitespace allowed between the sign and the digits.  If `val` satisfy the grammar and fits into the range of the type, it will successfully converted. Otherwise apropriate runtime errors will be raised.|
|`decimal`|`decimal.Parse(val)` with the following `NumberFormatInfo` and `NumberStyles`:<br />&emsp;&#9679;&emsp;Cultural locale is "en-us", which includes<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberFormatInfo.PositiveSign: '+'`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberFormatInfo.NegativeSign: '-'`<br />&emsp;&#9679;&emsp;`NumberStyle.Integer` that includes:<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowLeadingWhite`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowTrailingWhite`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowLeadingSign`<br />&emsp;&emsp;&emsp;&#9675;&emsp;`NumberStyles.AllowDecimalPoint`<br />In particular, the following `NumberStyles` are **not** supported:<br />&emsp;&#9679;&emsp;`NumberStyles.AllowCurrencySymbol`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowTrailingSign` (note: `decimal.Parse()` supports this per default!)<br />&emsp;&#9679;&emsp;`NumberStyles.AllowParentheses`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowThousands` (note: `decimal.Parse()` supports this per default!)<br />&emsp;&#9679;&emsp;`NumberStyles.AllowExponent`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowHexSpecifier`<br />resulting in the following grammar for the supported numbers lexical representations:<hr />`[whitespace] [sign] [digits] [[decimal-point] digits] [whitespace]`<hr />Note there is no whitespace allowed between the sign, decimal-point (if present) and the digits. If `val` satisfy the grammar and fits into the range of the type, it will successfully converted.  If `val` represents more than the decimal type’s 29 digits of precision, has a fractional part and is within the range of MaxValue and MinValue, the number is rounded, not truncated, to 29 digits using rounding to nearest. Otherwise apropriate runtime errors will be raised. |
|`float`|`float.Parse(val)` with the following `NumberFormatInfo` and `NumberStyles`:<br />&emsp;&#9679;&emsp;Cultural locale is "en-us"<br />&emsp;&#9679;&emsp;`NumberFormatInfo.PositiveInfinitySymbol: Infinity`<br />&emsp;&#9679;&emsp;`NumberFormatInfo.NegativeInfinitySymbol: -Infinity`<br />&emsp;&#9679;&emsp;`NumberFormatInfo.NaNSymbol: NaN`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowLeadingWhite`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowTrailingWhite`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowLeadingSign`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowDecimalPoint`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowExponent`<br />In particular, the following `NumberStyles` are **not** supported:<br />&emsp;&#9679;&emsp;`NumberStyles.AllowCurrencySymbol`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowTrailingSign`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowParentheses`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowThousands` (note: `double.Parse()` supports this)<br />resulting in the following grammar for the supported numbers lexical representations:<hr />`[sign] "Infinity"` &#124; `"NaN"` &#124; `[whitespace] [sign] [digits] [[decimal-point] digits] [ ("d"` &#124; `"D"` &#124; `"e"` &#124; `"E") [sign] digits] [whitespace]` <hr />If no digits appear before the decimal point character, at least one must appear after the decimal point character. If neither an exponent part nor a decimal point character appears, a decimal point character is assumed to follow the last digit in the string. Values that are too large or small will raise a runtime error, values who are smaller than the supported precision will get rounded to 0 (but not -0) as in C#. Currently -0 is mapped to 0.| 
|`double`|`double.Parse(val)` with the following `NumberFormatInfo` and `NumberStyles`:<br />&emsp;&#9679;&emsp;Cultural locale is "en-us"<br />&emsp;&#9679;&emsp;`NumberFormatInfo.PositiveInfinitySymbol: Infinity`<br />&emsp;&#9679;&emsp;`NumberFormatInfo.NegativeInfinitySymbol: -Infinity`<br />&emsp;&#9679;&emsp;`NumberFormatInfo.NaNSymbol: NaN`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowLeadingWhite`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowTrailingWhite`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowLeadingSign`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowDecimalPoint`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowExponent`<br />In particular, the following `NumberStyles` are **not** supported:<br />&emsp;&#9679;&emsp;`NumberStyles.AllowCurrencySymbol`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowTrailingSign`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowParentheses`<br />&emsp;&#9679;&emsp;`NumberStyles.AllowThousands` (note: `double.Parse()` supports this)<br />resulting in the following grammar for the supported numbers lexical representations:<hr />`[sign] "Infinity"` &#124; `"NaN"` &#124; `[whitespace] [sign] [digits] [[decimal-point] digits] [ ("d"` &#124; `"D"` &#124; `"e"` &#124; `"E") [sign] digits] [whitespace]` <hr />If no digits appear before the decimal point character, at least one must appear after the decimal point character. If neither an exponent part nor a decimal point character appears, a decimal point character is assumed to follow the last digit in the string. Values that are too large or small will raise a runtime error, values who are smaller than the supported precision will get rounded to 0 (but not -0) as in C#. Currently -0 is mapped to 0.|
|`string`|string values are read as UTF-16 encoded strings.|
|`char`|a character is read as a Unicode 16-bit codepoint (this means including single surrogate pair codepoints that are not valid Unicode characters). It supports the following lexical representations on input:<br /><br />&emsp;1.&emsp;    Base-10 Integer numbers. A number will map to the Unicode codepoint of that number. E.g., 32 will map to space.<br />&emsp;2.&emsp;Unicode codepoints of the range 0x0000 to 0xFFFF.<br />If the character is outside the valid range, an error will be raised.|
|`byte[]`|It will read hexadecimal numbers into a byte array. The lexical representation in the file has to correspond the following grammar:<hr />`byte_array = {byte}.`<br />`byte = hexcode hexcode.`<br />`hexcode = '0…9'` &#124; `'A…F'` &#124; `'a…f'.`<hr />If the number of hex codes is odd, then an error is raised.|
|`DateTime`|`DateTime.Parse(val)` with the following:<br /><br />This means a variety of formats are supported, including (note optionality and short forms are not called out):<br />&emsp;&#9679;&emsp;UTC format: YYYY-MM-DDThh:mm:ss.nnnn[TZ]<br />&emsp;&#9679;&emsp;Day, DD MMM YYYY hh:mm:ss [AM/PM] [verbal TZ]<br />&emsp;&#9679;&emsp;MM/DD/YYYY [hh:mm:ss [AM/PM]]<br />&emsp;&#9679;&emsp;MM.DD.YYYY [hh:mm:ss [AM/PM]]<br />The hour information can be given using a 12 or 24 hour clock. If a 12 hour clock is used, PM has to be specified to indicate time points in the second half of the day.<br />If parts of the date or time information is missing in the value, it will be defaulted as follows:<br />&emsp;&#9679;&emsp;If the month or day value is missing, it is defaulted to 1.<br />&emsp;&#9679;&emsp;If the year is missing, it is defaulted to the current year. In the non-UTC format, if the year is given with 2 digits, then the current century is assumed (e.g., 14 is transformed to 2014).<br />&emsp;&#9679;&emsp;If hours, minutes, seconds or subseconds are missing, they are defaulted to 0 (note that 0 hours is often represented as 12AM).<br />U-SQL’s built-in extractors normalize all date time values to UTC -07:00 and then drop timezone information if present.|
|`Guid`|The following lexical representations can be converted into a GUID:<hr />`'{' byte4 '-' byte2 '-' byte2 '-' byte2 '-' byte4 '}'` &#124;<br />&emsp;&emsp;`byte4 '-' byte2 '-' byte2 '-' byte2 '-' byte4` &#124;<br />&emsp;&emsp;`byte4 byte2 byte2 byte2 byte4.`<br />`byte4 = byte2 byte2.`<br />`byte2 = byte byte.`<br />`byte = hexcode hexcode.`<br />`hexcode = '0…9'` &#124; `'A…F'` &#124; `'a…f'`.<hr />This means the following three guids are all valid lexical representations:<br />`F9168C5E-CEB2-4faa-B6BF-329BF39FA1E4`<br />`{F9168C5E-CEB2-4faa-B6BF-329BF39FA1E4}`<br />`F9168C5ECEB24faaB6BF329BF39FA1E4`<br /><br />But the following are not:<br />`{F9168C5ECEB24faaB6BF329BF39FA1E4}`<br />`F9168C5E-CEB24faaB6BF329BF39FA1E4` | 
 
 ### See Also 
* [Extractor Parameters (U-SQL)](extractor-parameters-u-sql.md)
* [Extractors.Text()](extractors-text.md)  
* [Extractors.Csv()](extractors-csv.md)  
* [Extractors.Tsv()](extractors-tsv.md)  
* [EXTRACT Expression (U-SQL)](extract-expression-u-sql.md)
* [Input Files (U-SQL)](input-files-u-sql.md)  
* [U-SQL Programmability Guide: User-Defined Extractor](https://docs.microsoft.com/azure/data-lake-analytics/data-lake-analytics-u-sql-programmability-guide#user-defined-extractor)  
* [Extending U-SQL Expressions with User-Code](extending-u-sql-expressions-with-user-code.md)   
