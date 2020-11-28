---
title: DAX优化20招
date: 2020-11-27 16:43:09
tags: Power BI
subtitle: Power BI优化必看
---

Power BI 性能问题通常是由于数据分析表达式（DAX）语言不够理想而导致的。低效率的DAX会减慢处理速度，阻塞高级容量，增加等待时间，并妨碍刷新和报告加载时间。



### 1. 在优化DAX之前清除DAX缓存

- 缓存由内部VertiPaq查询产生。
- 从DAX Studio中清除缓存。
- 重置缓存可让您衡量有效的性能提升。

### 2. 格式化代码

- 使用DAX Formatter。
- 格式化的代码更易于阅读和维护。

### 3. 不要将BLANK值更改为零或其他字符串

- 通常的做法是用零或其他字符串替换空格。
- 但是，Power BI*自动*过滤所有带有空白值的行。当从具有大量数据的表中查看结果时，这会限制结果集并防止性能下降。
- 如果更换了空白，则Power BI不会过滤不需要的行，从而对性能产生负面影响。

### 4. 始终使用DISTINCT（）和VALUES（）函数

- DISTINCT（）：不返回由于完整性冲突而添加的空白。仅当DISTINCT（）函数是原始数据的一部分时，才包含空格。
- VALUES（）：包括Power BI由于引用完整性违规而添加的任何空白。
- 如果Power BI发现参照完整性违规，则会在列中添加空白值。
- 对于直接查询，因为Power BI无法检查违规，所以Power BI在列中添加了空白值。
- DISTINCT（）和VALUES（）函数不同：
- 在整个报表中，保持DISTINCT（）和VALUES（）函数的用法一致。
- 如果没有空白值，Power BI建议使用VALUES（）函数。

### 5. 使用ISBLANK（）代替= Blank（）

- 使用内置函数ISBLANK（）来检查任何空白值，而不是使用比较运算符= Blank（）。

### 6. 使用= 0而不是检查ISBLANK（）|| = 0

- Power BI中的BLANK值与列数据类型的基值相关联
- 对于整数，BLANK值对应于零，对于字符串列，BLANK值对应于“（空字符串）”，对于日期字段，BLANK值对应于“ 1-1–1900”。
- ISBLANK（）|| = 0时执行两个检查：ISBLANK（）并与零进行比较。
- Use = 0，在内部执行两项检查。
- 要仅执行零检查，请使用IN运算符。

### 7. 使用SELECTEDVALUE（）代替HASONEVALUE（）

- 在应用切片器和过滤器后，通常使用HASONEVALUE（）检查一列中是否只有一个值。您还必须使用VALUES（ColumnName）DAX函数来检索该单个值。
- SELECTEDVALUE（）在内部执行上述步骤。如果有一个值，它将自动检索单个值；如果有多个可用值，它将自动返回空白。

### 8. 使用SELECTEDVALUE（）而不是VALUES（）

- 如果遇到多个值，VALUES（）函数将返回错误。通常，用户使用错误功能解决错误，这会对性能产生负面影响。
- 而不是使用VALUES（），请使用SELECTEDVALUE（）。SELECTEDVALUE（）函数更安全，如果遇到多个值，则返回空白。

### 9. 使用变量而不是在IF分支内重复测量

- 由于度量是连续计算的，因此[Total Rows]表达式将计算两次：首先用于条件检查，然后用于真实条件表达式。

- 不正确的DAX：

  ```
  `Ratio = IF([Total Rows] > 10, SUM(Revenue) /[Total Rows], 0)`
  ```

- 正确的DAX：
  
```
  VAR totalRows = [Total Rows]; Ratio = IF(totalRows > 10, SUM(Revenue) / totalRows,0)
```

- 您可以将结果度量值存储在变量中，而不必多次计算相同的表达式。

- 您可以在任何需要的地方使用变量引用。相同的变量过程适用于您调用相同度量的所有实例。

- 变量可以帮助您避免重复功能。

- 注意：请注意，变量实际上是常量。

### 10. 将（ab）/ b与变量一起使用，而不是a / b — 1或a / b * 100-100

- 通常使用a / b_1来计算比率并避免重复进行度量计算。
- 但是，通过使用变量并使用（ab）/ b计算比率，可以实现相同的性能。
- 这就是为什么您应该使用（ab）/ b的原因： `If both a and b are blank values, then (a-b)/b returns a blank value and Power BI will filter the values out. a/b — 1 would return -1 as the result because both a and b are integers.`
- 参考：sqlbi

### 11. 停止使用IFERROR（）和ISERROR（）

- 当应用FIND（）和SEARCH（）函数时，IFERROR（）和ISERROR（）函数在Excel中得到了广泛使用。它们是必需的，因为如果查询未获得所需的结果，则FIND（）和SEARCH（）返回错误。
- IFERROR（）和ISERROR（）函数强制Power BI引擎对每一行执行逐步执行，以检查错误。当前没有任何方法可以直接说明哪一行返回了错误。
- FIND（）和SEARCH（）DAX函数提供了查询可以传递的额外参数。如果不存在搜索字符串，则返回该参数。
- FIND（）和SEARCH（）DAX函数检查是否返回了多个值。它们还确保没有任何东西被零除。通过使用正确的DAX函数（例如DIVIDE（）和SELECTEDVALUE（）），可以避免使用FIND（）和SEARCH（）DAX函数。DIVIDE（）和SELECTEDVALUE（）函数在内部执行错误检查并返回预期结果。
- 您始终可以使用DAX表达式，使其永远不会返回错误。

### 12. 使用DIVIDE（）代替/

- /如果分母为零，则引发异常。
- DIVIDE（）函数在内部执行检查以验证分母是否为零。如果是，它将返回第三个（额外）参数中指定的值。
- 对于“无效分母”的情况，请在使用“ /”运算符时使用IF条件。
- 注意：如果确定分母值不为零，则最好使用“ /”运算符而不进行IF检查。DIVIDE（）函数将始终在内部执行IF检查。


### 13. 不要在SUMMARIZE（）中使用标量变量

- 传统上，SUMMARIZE（）函数用于对列进行分组并返回结果聚合。但是，SUMMARIZECOLUMNS（）函数是较新的且已优化。改用它。
- 仅将SUMMARIZE（）用于表的分组元素，而没有任何关联的度量或聚合。例如： `SUMMARIZE(Table, Column1, Column2)`

### 14. 使用KEEPFILTERS（）代替FILTER（T）

- FILTER函数会覆盖通过切片器应用的列上的任何现有过滤器集。
- KEEPFILTER函数不会覆盖现有的过滤器集。而是使用两者中存在的值的交集，从而保持当前上下文。当您想要在执行计算时维护切片器应用的任何过滤器或在报告级别上使用此过滤器。

### 15. 使用FILTER（all（ColumnName））代替FILTER（values（））或FILTER（T）

- bid

- sqlbi

- 若要计算独立于应用于列的任何过滤器的度量，请将All（ColumnName）函数与FILTER函数结合使用，而不要使用Table或VALUE（）。例如：

  ```
  CALCULATE([Total Sales], FILTER(ALL(Products[Color]), Color = ‘Red’))
  ```

- 如果不需要保留当前上下文，则将ALL与FILTER函数一起使用。
- 使用表达式而不是FILTER函数直接应用过滤器的行为与上述相同。此方法在内部使用过滤器中的ALL函数进行转换。例如： `CALCULATE([Total Sales], FILTER(ALL(Products[Color]), Color = ‘Red’))`
- 出于可伸缩性考虑，始终将过滤器应用于所需的列而不是应用于整个表总是更好。


### 16. 避免在度量表达式中使用AddColumns（）函数

- 默认情况下，度量是迭代计算的。
- 如果度量定义使用诸如AddColumns（）之类的迭代函数，则Power BI将创建嵌套的迭代，这会对报表性能产生负面影响。


### 17. 根据列值使用正确的数据类型

- 如果一列中只有两个不同的值，请检查是否可以将其转换为布尔数据类型（真/假）。
- 当您有大量的行时，这可以加快处理速度。

### 18. 使用COUNTROWS而不是COUNT：

- 使用COUNT函数对列值进行计数，或者我们可以使用COUNTROWS函数对表行进行计数。只要计数的列不包含空白，这两个函数将达到相同的结果。 `Sales Orders = COUNT(Sales [OrderDate]) Sales Orders = COUNTROWS(Sales)`
- 第二个度量定义更好的三个原因：
- 参考：DAX-CountRows
  - 它效率更高，并且性能会更好。
  - 它不考虑表的任何列中包含的空白。
  - 公式的意图更加清晰和自我描述。

### 19. 将SEARCH（）与最后一个参数一起使用

- 如果未找到搜索字符串，则SEARCH（）DAX函数接受最后一个参数作为查询必须返回的值。
- 您应该始终使用SEARCH（）函数，而不是与SEARCH（）一起使用Error函数。

### 20. ALL vs.ALLExcept

- 只要“豁免”列是数据透视表上的列，ALLEXCEPT（）的行为就与ALL（），VALUES（）完全一样。
- ALLEXCEPT（）不会在不在枢轴上的列上保留枢轴上下文。
- 使用VALUES（）时，使用ALL（）代替ALLEXCEPT（）。



**
**

**—— —— —— —— —— —— —— —— —— —— —— —— —— —— —— ——** **—**

*注：本文翻译自“https://maqsoftware.com/expertise/powerbi/dax-best-practices”*