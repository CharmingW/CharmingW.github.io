## 查找
```
SELECT 列名称 FROM 表名称
```
以及
```
SELECT * FROM 表名称
```

Persons 表

|Id|LastName|FirstName|Address|City|
|-|-|-|-|-|
|1|	Adams|	John|	Oxford Street|	London|
|2|	Bush	|George	|Fifth Avenue|	New York|
|3|	Carter	|Thomas|Changan Street|	Beijing|

获取名为 "LastName" 和 "FirstName" 的列的内容
```
SELECT LastName,FirstName FROM Persons
```

结果

|LastName|	FirstName|
|-|-|
|Adams	|John|
|Bush	|George|
|Carter	|Thomas|

从 "Persons" 表中选取所有的列
```
SELECT * FROM Persons
```

结果

Id	|LastName	|FirstName|	Address|	City
-|-|-|-|-
1|	Adams|	John	|Oxford Street|	London
2	|Bush	|George	|Fifth Avenue	|New York
3	|Carter|	Thomas	|Changan Street	|Beijing

## 更新
```
UPDATE 表名称 SET 列名称 = 新值 WHERE 列名称 = 某值
```
Persons 表

LastName	|FirstName	|Address|	City
-|-|-|-
Gates	Bill	|Xuanwumen |10	|Beijing
Wilson	 |	|Champs-Elysees|

更新某一行中的一个列，为 lastname 是 "Wilson" 的人添加 firstname
```
UPDATE Person SET FirstName = 'Fred' WHERE LastName = 'Wilson'
```

结果

LastName	|FirstName|	Address|	City
-|-|-|-
Gates	Bill	|Xuanwumen |10	|Beijing
Wilson|	Fred	|Champs-Elysees	|

更新某一行中的若干列，修改地址（address），并添加城市名称（city）
```
UPDATE Person SET Address = 'Zhongshan 23', City = 'Nanjing' WHERE LastName = 'Wilson'
```

结果

LastName	|FirstName|	Address|	City
-|-|-|-
Gates|	Bill	|Xuanwumen 10|	Beijing
Wilson	|Fred	|Zhongshan 23	|Nanjing

## 插入
```
INSERT INTO 表名称 VALUES (值1, 值2,....)
```
指定所要插入数据的列
```
INSERT INTO table_name (列1, 列2,...) VALUES (值1, 值2,....)
```
Persons 表

LastName|	FirstName|	Address|	City
-|-|-|-
Carter|	Thomas|	Changan Street|	Beijing

插入新的行
```
INSERT INTO Persons VALUES ('Gates', 'Bill', 'Xuanwumen 10', 'Beijing')
```
结果

LastName|	FirstName|	Address	|City
-|-|-|-
Carter|	Thomas|	Changan Street|	Beijing
Gates	|Bill|	Xuanwumen 10|	Beijing

在指定的列中插入数据

```
INSERT INTO Persons (LastName, Address) VALUES ('Wilson', 'Champs-Elysees')
```
结果

LastName|	FirstName|	Address|	City
-|-|-|-
Carter|	Thomas|	Changan Street|	Beijing
Gates|	Bill|	Xuanwumen 10|	Beijing
Wilson|	 |	Champs-Elysees	| |
## 删除
```
DELETE FROM 表名称 WHERE 列名称 = 值
```
Persons 表

LastName|	FirstName|	Address	|City
-|-|-|-
Carter|	Thomas|	Changan Street|	Beijing
Gates	|Bill|	Xuanwumen 10|	Beijing

删除FirstName为"Bill"的行

```
DELETE FROM Person WHERE FirstName = 'Bill'
```
结果
LastName|	FirstName|	Address|	City
-|-|-|-
Carter|	Thomas|	Changan Street|	Beijing

删除所有行

```
DELETE FROM table_name
```
或

```
DELETE * FROM table_name
```


## WHERE
```
SELECT 列名称 FROM 表名称 WHERE 列 运算符 值
```

下面的运算符可在 WHERE 子句中使用

|=	|等于|
|-|-|
|<>	|不等于|
|>	|大于|
|<	|小于|
|>=	|大于等于|
|<=	|小于等于|
|BETWEEN	|在某个范围内|
|LIKE	|搜索某种模式|
注释：在某些版本的 SQL 中，操作符 <> 可以写为 !=

Persons表

LastName|	FirstName|	Address	|City|	Year
-|-|-|-|-
Adams|	John|	Oxford Street|	London	|1970
Bush|	George	|Fifth Avenue|	New York|	1975
Carter|	Thomas|	Changan Street|	Beijing|	1980
Gates|	Bill|	Xuanwumen 10|	Beijing	|1985
选取居住在城市 "Beijing" 中的人

```
SELECT * FROM Persons WHERE City='Beijing'
```

结果

LastName|	FirstName|	Address|	City|	Year
-|-|-|-|-
Carter|	Thomas|	Changan Street|	Beijing	|1980
Gates|	Bill|	Xuanwumen 10|	Beijing	|1985

请注意，我们在例子中的条件值周围使用的是单引号。  
SQL 使用单引号来环绕文本值（大部分数据库系统也接受双引号）。如果是数值，请不要使用引号。

文本
```
这是正确的：
SELECT * FROM Persons WHERE FirstName='Bush'

这是错误的：
SELECT * FROM Persons WHERE FirstName=Bush
```

数值
```
这是正确的：
SELECT * FROM Persons WHERE Year>1965

这是错误的：
SELECT * FROM Persons WHERE Year>'1965'
```

## AND & OR
AND 和 OR 运算符用于基于一个以上的条件对记录进行过滤。  
AND 和 OR 可在 WHERE 子语句中把两个或多个条件结合起来。  
如果第一个条件和第二个条件都成立，则 AND 运算符显示一条记录。  
如果第一个条件和第二个条件中只要有一个成立，则 OR 运算符显示一条记录。  

表

|LastName	|FirstName	|Address	|City|
|-|-|-|-|
|Adams	|John	|Oxford Street	|London|
|Bush	|George	|Fifth Avenue	|New York|
|Carter	|Thomas	|Changan Street	|Beijing|
|Carter	|William	|Xuanwumen 10	|Beijing|

使用 AND 来显示所有姓为 "Carter" 并且名为 "Thomas" 的人：
```
SELECT * FROM Persons WHERE FirstName='Thomas' AND LastName='Carter'
```

结果

|LastName	|FirstName	|Address	|City|
|-|-|-|-|
|Carter	|Thomas	|Changan |Street	Beijing|

使用 OR 来显示所有姓为 "Carter" 或者名为 "Thomas" 的人
```
SELECT * FROM Persons WHERE firstname='Thomas' OR lastname='Carter'
```

结果

LastName|	FirstName|	Address|	City
-|-|-|-
Carter	|Thomas|	Changan Street	|Beijing
Carter|	William	|Xuanwumen 10|	Beijing

结合 AND 和 OR 运算符
```
SELECT * FROM Persons WHERE (FirstName='Thomas' OR FirstName='William')
AND LastName='Carter'
```

结果

LastName|	FirstName|	Address|	City
-|-|-|-
Carter	|Thomas|	Changan Street	|Beijing
Carter|	William	|Xuanwumen 10|	Beijing

## ORDER BY
ORDER BY 语句用于对结果集进行排序。  
ORDER BY 语句用于根据指定的列对结果集进行排序。  
ORDER BY 语句默认按照升序对记录进行排序。  
如果您希望按照降序对记录进行排序，可以使用 DESC 关键字。

Orders 表

|Company	|OrderNumber|
|-|-|
|IBM	|3532|
|W3School	|2356|
|Apple|	4698|
|3School	|6953  |

以字母顺序显示公司名称
```
SELECT Company, OrderNumber FROM Orders ORDER BY Company
```

结果

|Company|	OrderNumber|
|-|-|
|Apple|	4698|
|IBM	|3532|
|W3School|	6953|
|W3School	|2356|

以字母顺序显示公司名称（Company），并以数字顺序显示顺序号（OrderNumber）
```
SELECT Company, OrderNumber FROM Orders ORDER BY Company, OrderNumber
```

结果

Company|	OrderNumber
-|-
Apple|	4698
IBM	|3532
W3School	|2356
W3School|	6953

以逆字母顺序显示公司名称
```
SELECT Company, OrderNumber FROM Orders ORDER BY Company DESC
```

结果

Company	|OrderNumber
-|-
W3School|	6953
W3School|	2356
IBM|	3532
Apple|	4698

以逆字母顺序显示公司名称，并以数字顺序显示顺序号
```
SELECT Company, OrderNumber FROM Orders ORDER BY Company DESC, OrderNumber ASC
```

结果

Company	|OrderNumber
-|-
W3School	|2356
W3School|	6953
IBM|	3532
Apple|	4698

## DISTINCT

```
SELECT DISTINCT 列名称 FROM 表名称
```

Company|	OrderNumber
-|-
IBM|	3532
W3School|	2356
Apple|	4698
W3School|	6953

如果要从 "Company" 列中选取所有的值，我们需要使用 SELECT 语句
```
SELECT Company FROM Orders
```


结果

Company|
-|
IBM|
W3School|
Apple|
W3School|

如需从 Company" 列中仅选取唯一不同的值，我们需要使用 SELECT DISTINCT 语句

```
SELECT DISTINCT Company FROM Orders
```

结果

Company|
-|
IBM|
W3School|
Apple|
