---
title: "- (Subtract) (U-SQL) | Microsoft Docs"
ms.custom: ""
ms.date: "03/27/2017"
ms.reviewer: ""
ms.service: "data-lake-analytics"
ms.suite: ""
ms.tgt_pltfrm: ""
ms.topic: "reference"
ms.assetid: bd2b059b-96ea-4180-953b-620a20d22978
caps.latest.revision: 3
author: "MikeRys"
ms.author: "mrys"
manager: "ryanw"
---
# - (Subtract) (U-SQL)
Subtracts two [numbers](numeric-types-and-literals.md) (an arithmetic subtraction operator).  Can also calculate [date and time](temporal-types-and-literals.md) differences.

<table><th align="left">Syntax</th><tr><td><pre>
Subtract_Operator :=                                                                                     
     <a href="#expr">expression</a> '-' <a href="#expr">expression</a>.
</pre></td></tr></table>

  
### Semantics of Syntax Elements    
-   <a name="expr"></a>**`expression`**  
Is the expression to subtract. 

### Return Type
Returns the data type of the argument with the higher precedence.

### Examples
- The examples can be executed in Visual Studio with the [Azure Data Lake Tools plug-in](https://www.microsoft.com/download/details.aspx?id=49504).  
- The scripts can be executed [locally](https://docs.microsoft.com/azure/data-lake-analytics/data-lake-analytics-data-lake-tools-get-started#run-u-sql-locally).  An Azure subscription and Azure Data Lake Analytics account is not needed when executed locally.

**Subtract with Numeric Types**   
```sql
@data = 
    SELECT * FROM 
        ( VALUES
        (38)
        ) AS T(aNumber);

DECLARE @val int = 5;

@result =
    SELECT 38 - 5 AS Int1,
           aNumber - 5 AS Int2,
           aNumber - @val AS Int3,
           aNumber - (int?)null AS intNull
    FROM @data;

OUTPUT @result
TO "/ReferenceGuide/Operators/Arithmetic/Subtract1.txt"
USING Outputters.Csv();
```

**Subtract with Temporal Types**   
```sql
@data = 
    SELECT * FROM 
        ( VALUES
        (new DateTime(2016,05,31))
        ) AS T(aDate);

@result =
    SELECT aDate - new System.TimeSpan(1, 0, 0, 0) AS date1,
          (new TimeSpan(1, 0, 0, 0) - new TimeSpan(0, 1, 0, 0)).ToString() AS date2,
          (aDate - new DateTime(2016,05,1)).Days AS date3
    FROM @data;

OUTPUT @result
TO "/ReferenceGuide/Operators/Arithmetic/Subtract2.txt"
USING Outputters.Csv();
```

### See Also
* [Operators (U-SQL)](operators-u-sql.md)
* [Date & Time](csharp-functions-and-operators-u-sql.md#DateTime)
* [Simple Built-In U-SQL Types](simple-built-in-u-sql-types.md)


