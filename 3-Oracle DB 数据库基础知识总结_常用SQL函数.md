## 3 常用SQL函数汇总

### 3.1 有关日期类型计算的函数

> Oracle DB中，日期的存储格式是RR格式存储的，格式为'DD-MON-YY'，例如'01-DEC-98'，代表1998年12月1日。
> 在每个世纪的前50年（例如2000-2049年是21世纪的前50年），年份的两位数字若在[00, 49]之间，则代表本世纪的第1-50年（2000年-2049年），若在[50, 99]之间则表示上世纪的后50年（1950年-1999年）；在每个世纪的后50年（例如20050-2099年是21世纪的后50年），年份的两位数字若在[00, 49]之间，则代表下个世纪的第1-50年（2100年-2149年），若在[50, 99]之间则表示本世纪的后50年（2050年-2099年）。
> Oracle DB允许日期与日期之间进行直接的加减运算（一般多用减法），得到两日期之和的天数（加法）或者两日期之间的天数（减法）。

#### 3.1.1 返回当前日期与时间的相关函数：

- CURRENT_DATE() 返回当前的日期，时间为00:00:00，等同于CURDATE()
- CURRENT_TIME() 返回当前的时间，Oracle DB不支持这个函数，等同于CURTIME()
- CURRENT_TIMESTAMP() 返回当前时间戳（格式为：年-月-日 时:分:秒），等同于NOW()

用法：

```SQL
--获取日期：
SELECT CURRENT_DATE() FROM dual;
SELECT CURDATE() FROM dual;
--获取时间（Oracle DB不支持此类函数）：
SELECT CURRENT_TIME() FROM dual;
SELECT CURTIME() FROM dual;
--获取时间戳：
SELECT CURRENT_TIMESTAMP() FROM dual;
SELECT NOW() FROM dual;
```

#### 3.1.2 返回日期具体部分（年月日，时分秒）的相关函数：

- 日期：YEAR(), MONTH(), DAY()
- 时间：HOUR(), MINUTE(), SECOND()

> 返回日期的具体部分所有函数均可以使用EXTRACT()函数代替，使用EXTRACT(p FROM dt)进行获取。
> 其中p代表想要获取的日期的部分(YEAR, MONTH, DAY, HOUR, MINUTE, SECOND)，dt代表日期数据。

```SQL
--获取日期的年月日
SELECT YEAR(CURDATE()) FROM dual;
SELECT MONTH(CURDATE()) FROM dual;
SELECT DAY(CURDATE()) FROM dual;
--获取时间的时分秒
SELECT HOUR(CURRENT_TIMESTAMP()) FROM dual;
SELECT MINUTE(CURRENT_TIMESTAMP()) FROM dual;
SELECT SECOND(CURRENT_TIMESTAMP()) FROM dual;
--使用EXTRACT()方法的示例
SELECT EXTRACT(YEAR FROM CURDATE()) FROM dual;
```

> 使用MONTHNAME()和DAYNAME()可以获得日期所在月份的英文名称，或者日期当天的星期的英文名称。

```SQL
SELECT MONTHNAME(CURDATE()) FROM dual;
SELECT DAYNAME(CURDATE()) FROM dual;
```

#### 3.1.3 日期的相关计算

在SQL语言的规则中，不能够直接用数字加减来进行日期类型的计算，需要用到INTERVAL关键字。例如：

- INTERVAL 1 MONTH 表示一个月；
- INTERVAL -1 DAY 表示前一天；
- INTERVAL 2 YEAR 表示两年；

```SQL
--显示当前时间一个月前的日期
SELECT CURDATE() - INTERVAL 1 MONTH FROM dual;
SELECT CURDATE() + INTERVAL -1 MONTH FROM dual;
/* 拓展：获取当前日期所在月的第一天（每月1日）:
 * 之所以需要在最后加一天，是因为当前日期减去当前日期的天数会得到当月0日，
 * 也就是上一个月的最后一天，此时需要加上一天以获取这个月的1日的日期。 */
SELECT CURDATE() - INTERVAL DAY(CURDATE()) DAY + INTERVAL 1 DAY FROM dual;
--返回该月的最后一天
SELECT LAST_DAY(CURDATE()) FROM dual;
```

> 虽然使用代数计算的方法就能够对日期进行加减运算，SQL仍提供了对日期进行加减运算的函数方法。
> DATE_ADD()和DATE_SUB()就是用来对日期类型数据进行加减运算的SQL方法。

```SQL
--获取近七天内的数据
--其中DATE_SUB(CURDATE(), INTERVAL 7 DAY)代表近7天的限制条件
SELECT * FROM table_name
WHERE table_date >= DATE_SUB(CURDATE(), INTERVAL 7 DAY);
```

此外，还有一些*Oracle DB独有*的时间计算的函数。

```SQL
--计算两个日期之间间隔多少个月，下面的例子是2024年1月1日距离今天有多少个月
SELECT MONTHS_BETWEEN(SYSDATE, '01-JAN-24') FROM dual;
--返回某个日期过n个月后的日期，下面的例子是从今天向后推两个月的日期
SELECT ADD_MONTHS(SYSDATE, 2) FROM dual;
/* ROUND(四舍五入)和TRUNC(截取)在Oracle DB中同样有效
 * 参数：ROUND/TRUNC(date, <'month'/'year'>)
 * ROUND()意为过半向上取，TRUNC()一般取当前时间所在年/月的第一天 */
SELECT ROUND('16-MAR-24', 'month') FROM dual; --返回'01-APR-24'
SELECT TRUNC('16-AUG-24', 'year') FROM dual; --返回'01-JAN-24'
--返回给定日期之后首个满足条件的日期，下面的例子是找当前时间之后的首个周一
SELECT NEXT_DAY(SYSDATE, 'Monday') FROM dual;
```

### 3.2 有关数学计算的函数

#### 3.2.1 数字计算

绝对值相关函数：ABS(number)

> 取某个数字的绝对值，例如ABS(-1.3) = 1.3，ABS(3) = 3

```SQL
SELECT ABS(-1.3), ABS(3) FROM dual;
```

平方根相关函数：SQRT(number)

> 取某个正数的平方根

```SQL
SELECT SQRT(4), SQRT(6.25) FROM dual;
```

乘方、对数相关函数：POW(x, y), POWER(x, y), EXP(number), LOG(number)

> POW()和POWER()的作用相同，均为求x的y次方，EXP()是求自然对数的底数e的number次方
> 例如POW(2, 2) = 4, POWER(3, 3) = 27, EXP(2) = e的平方
> LOG()是求自然对数(以e为底数的对数)，例如LOG(EXP(3)) = 3

```SQL
SELECT POW(2, 2), POWER(3, 3) FROM dual;
SELECT EXP(2) FROM dual;
SELECT LOG(EXP(3)) FROM dual;
```

舍入函数：ROUND(x, y), TRUNC(x, y), TRUNC()是*Oracle DB独有*的函数
FLOOR(number), CEIL(number), CEILING(number)

> ROUND()函数是按照y的位置基准做四舍五入计算，而TRUNC()函数是按照y的位置基准做截取计算(意为直接舍掉某个数字后面的部分), y代表舍入的位置，y>=0代表保留y位小数，y<0代表从整数部分个位数开始，向左侧的第y位进行舍入。
> 例如ROUND(234.567, 1) = 234.6, ROUND(234.567, -1) = 200, TRUNC(234.567, 1) = 234.5, TRUNC(234.567, -2) = 200。

```SQL
SELECT ROUND(234.567, 1), ROUND(234.567, -2) FROM dual;
SELECT TRUNC(234.567, 1), TRUNC(234.567, -2) FROM dual;
```

> FLOOR()会返回一个不大于输入参数number的最大整数，CEIL()与CEILING()会返回一个不小于输入参数number的最小整数。
> 例如，FLOOR(-3.5) = -4, FLOOR(3.5) = 3, CEIL(-3.5) = -3, CEIL(3.5) = 4。

```SQL
SELECT FLOOR(-3.5), FLOOR(3.5) FROM dual;
SELECT CEIL(-3.5), CEILING(3.5) FROM dual;
```

#### 3.2.2 角度与弧度、三角函数的相关转化与计算

角度弧度计算的基础——圆周率：使用PI()来获得圆周率的值

```SQL
--在不规定保留几位小数的情况下，PI()默认返回3.141593
SELECT PI() FROM dual;
--使用RADIANS()将输入的角度参数转变为弧度，使用DEGREES()将输入的弧度参数转变为角度
SELECT RADIANS(90) FROM dual; --2/PI
SELECT DEGREES(PI()) FROM dual; --180度
```

SQL提供了正/反正弦、余弦、正切的函数计算，提供了余切的函数计算：

```SQL
SELECT SIN(PI() / 2), ASIN(1) FROM dual;
SELECT COS(PI() / 2), ACOS(1) FROM dual;
SELECT TAN(PI() / 6), ATAN(0.75) FROM dual;
--没有反余切
SELECT COT(PI() / 6) FROM dual;
```

### 3.3 字符串相关计算

#### 3.3.1 字符串格式转化

将数据转换成字符串：*(Oracle DB专属)*
日期转换为字符串格式：TO_CHAR(date, format)，将日期date转换成VARCHAR2的类型，可指定转换的类型format，例如'YYYY-MM-DD'，'DD-MON-RR'等。
转换为货币格式：TO_CHAR(number, format)，将数字转换成货币格式，货币格式format可以是'$9,999.99'等格式。

```SQL
--日期变字符串
SELECT TO_CHAR(SYSDATE, 'DD-MON-RR') FROM dual;
--数字变货币格式
SELECT TO_CHAR(1234.56, '$9,999.99') FROM dual;
```

字符串转换成日期：*(Oracle DB专属)*
TO_DATE(string, format)，日期可以是任何接受的日期格式，例如'YYYY-MM-DD'，'DD-MON-RR'等，字符串的内容需要与输入的日期格式参数一致。

```SQL
--字符串转换成日期格式
SELECT TO_DATE('2024-12-09 13:25:38', 'YYYY-MM-DD HH24:MI:SS') FROM dual;
```

#### 3.3.2 处理空值相关函数

NVL与NVL2(NVL与NVL2为*Oracle DB专属函数*)
NVL(exp1, exp2)，代表一组判断：如果exp1的值为NULL则返回exp2的值，若exp1的值不为NULL则返回exp1的值。
NVL2(exp1, exp2, exp3)，代表一组判断：如果exp1的值不为NULL则返回exp2的值，若exp1的值为NULL则返回exp3的值。与NVL相比，NVL2能够返回与exp1不同的值（exp1不为NULL的情况下），更加灵活。
NVL与NVL2函数能够：

- 处理空值情况，返回非空值1；
- 可根据不同条件的满足情况返回不同的值或执行不同的操作，类似于简化版的CASE；
- 以及更多……

NULLIF(exp1, exp2)，代表一组判断：如果exp1与exp2的值相等则返回NULL，若不相等则返回exp1。
NULLIF函数能够：

- 处理除法算式除数为0的情况：可使用NULLIF函数，在除数为零的时候将结果设置为NULL，避免除零现象而产生的错误；
- 处理相等情况并控制返回值：可人为设置字段值等于特定值时返回NULL，实现了人为控制；
- 以及更多……

```SQL
SELECT NVL(table_col, 'Hello') FROM table_name;
SELECT NVL2(table_col, 'Hello', 'World') FROM table_name;
SELECT NULLIF(divider, 0) FROM table_name;
```

COALESCE函数：COALESCE()能够在众多给定的参数中按顺序找到第一个非空的值，还支持输入默认数值，在给定参数均为空的情况下将会返回默认值。
COALESCE函数能够：

- 将NULL值变更为非NULL值；
- 在众多非NULL值中设置优先级，可人为选择第一个非空值填入字段；
- 以及更多……

```SQL
--从col1、col2、col3中按顺序查询，返回查询到的第一个非空值填入对应字段
SELECT COALESCE(col1, col2, col3, 'Default') AS notnull FROM table_name;
```

DECODE函数：DECODE可对某个字段中具有特定值的数据更换为其他数据，其作用类似于CASE。
DECODE函数能够：

- 提供人为修改多个具有特定值的数据为其他数据的功能，简单实用
- 可以用一行代码在处理空值的同时还可以处理其他具有特定值的数据
- 以及更多……

```SQL
--将employee表中的性别一栏以全称显示出来，如果性别为空则显示'Unknown'未知性别
SELECT names, DECODE(sex, 'M', 'Male', 'F', 'Female', 'Unknown') AS gender
FROM employee;
```

#### 3.3.3 字符串变换相关函数

字符串中字母大小写变化的函数：

- UPPER()——每个字母都变成大写；
- LOWER()——每个字母都变成小写；
- INITCAP()——将每个单词的首字母变为大写，其余字母变为小写。

```SQL
SELECT UPPER('HELLO world') FROM dual; --HELLO WORLD
SELECT LOWER('HELLO world') FROM dual; --hello world
SELECT INITCAP('HELLO world') FROM dual; --Hello World
```

字符串的连接与子串：

- CONCAT函数可以用来连接多个子字符串为一个完整的字符串；
- SUBSTR可以在一整个字符串内取指定区间的子串。

SUBSTR()有两种不同的格式：

- SUBSTR(string, a, b)：意为从string字符串的第a个位置开始，截取长度为b的子串；
- SUBSTR(string, a)：意为从string字符串的第a个位置开始截取字符串，直到字符串末尾。

```SQL
SELECT CONCAT('Hello', ' World') FROM dual; --Hello World
SELECT SUBSTR('HELLOWORLD', 2, 3) FROM dual; --ELL
SELECT SUBSTR('HELLOWORLD', 6) FROM dual; --WORLD
```

字符串的填充：

- LPAD()：若给定长度参数大于给定字符串参数的长度，则在原有字符串的左侧重复填充指定内容直到字符串长度达到给定长度参数，若给定长度参数不大于给定字符串参数的长度，则从左至右截取与给定长度参数相等长度的子串；
- RPAD()：RPAD的作用与LPAD相似，区别仅仅在于RPAD会在字符串右侧填充指定内容直到字符串长度达到给定长度参数。

```SQL
SELECT LPAD('HELLOWORLD', 4, '?') FROM dual; --HELL
SELECT LPAD('HELLOWORLD', 12, '?') FROM dual; --??HELLOWORLD
SELECT RPAD('HELLOWORLD', 4, '?') FROM dual; --HELL
SELECT RPAD('HELLOWORLD', 14, '?') FROM dual; --HELLOWORLD????
```

#### 3.3.4 字符串分析相关函数

LENGTH()：计算字符串的长度
INSTR()：返回指定子串在字符串中首次出现的位置

```SQL
SELECT LENGTH('HELLOWORLD') FROM dual; --10
SELECT INSTR('HELLOWORLD', 'O') FROM dual; --5
```
