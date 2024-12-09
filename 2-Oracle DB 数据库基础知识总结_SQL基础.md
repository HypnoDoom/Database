# Oracle DB 数据库基础知识总结

## 2 SQL语言基础总结

结构化查询语言(SQL, Structured Query Language)是关系数据库的标准语言，也是一个通用的、功能极强的关系数据库语言。SQL是一种综合统一、高度非过程化、十分简洁易懂的语言。

> 非过程化：非关系数据库的数据操纵语言是“面向过程”的语言，使用过程化语言完成某项请求之前，必须制定其存取路径。而使用SQL语言完成相同的请求，无需指定存取路径，只需要告诉系统“做什么”，而无需告诉系统“怎么做”，存取路径的选择以及SQL的操作过程均由系统自动完成，极大地减轻了用户的编程负担，有利于提高数据独立性。

SQL语言主要包含四类: 

- DDL(Data Definition Language)--数据定义语言;
- DQL(Data Query Language)--数据查询语言;
- DML(Data Manipulation Language)--数据操控语言;
- DCL(Data Control Language)--数据控制语言。

下面将对每一类SQL语言进行分析。

### 2.1 数据定义语言——DDL

DDL是用于描述数据库中要存储的现实世界实体的语言。
DDL主要针对于数据库/模式、数据库表的创建、修改以及删除的操作。
对于模式的创建/删除：

```SQL
--创建数据库/创建模式
CREATE DATABASE new_DB;
CREATE SCHEMA new_DB;
--删除数据库/删除模式
DROP DATABASE new_DB;
DROP SCHEMA new_DB;
```

对于数据库表的创建、删除与修改(MySQL)：

```SQL
--创建数据库表
CREATE TABLE table_name (
    tableid INT,
    val VARCHAR(10));
--修改数据库表的信息，改变字段属性，添加约束
ALTER TABLE table_name
CHANGE COLUMN val val VARCHAR(20) NOT NULL;
ALTER TABLE table_name
ADD CONSTRAINT pk_tableid PRIMARY KEY(tableid);
--删除数据库表
DROP TABLE table_name;
```

> 如果一个表的某些字段是另外的表的参照，或者成为了其他数据库对象的依赖，那么将不能够删除这个数据库表，除非在DELETE语句后面添加CASCADE（不加CASCADE不能删除的原因是系统默认删除参数是RESTRICT，即指定存在关联或依赖时删除操作是无法操作的），这样所有存在与该表关联的内容，以及与该表存在依赖关系的内容都将一并删除。

```SQL
--强制删除具有依赖关系的数据库表
DELETE table_name CASCADE;
```

### 2.2 数据查询语言——DQL

#### 2.2.1 基本用法
DQL语句主要针对SELECT语句，SELECT具有查询并返回所有满足条件的数据的指定字段值。
SELECT语句的基本用法：

```SQL
--查询并返回table_name表格中的全部数据
SELECT * FROM table_name;
--查询并返回table_name表格中列名为tableid的全部数据
SELECT tableid FROM table_name;
```

> 使用SELECT DISTINCT命令可以得到某一个字段互不相同的结果，在某些情况下可以有效避免数据冗余的现象。

#### 2.2.2 条件查询(WHERE)

如果用户需要只看到自己需要的结果，那么就需要在查询语句中加入限制条件，我们称加入了限制条件的查询语句为条件查询。
通常为SELECT语句后面添加WHERE+布尔表达式的方式来添加限制条件。

```SQL
SELECT * FROM table_name
WHERE tableid = 100;
```

> WHERE后面的布尔表达式的形式可以是：
> 1.变量 =(或者>, <, >=, <=, !=或<>) 另一个变量 (例如:tableid = 100, tableid >= 30);
> 2.变量 BETWEEN a AND b (变量在a和b之间均为成立，a和b均为变量，例如:tableid BETWEEN 10 AND 50);
> 3.变量 IN (一组变量值) (变量在这一组变量值中存在即为成立，例如:tableid IN (10, 20, 30));
> 4.变量 IS NULL (变量为空值时成立，不能用于带有非空约束的变量，例如:val IS NULL)
> 5.变量 LIKE ... (用于字符串型变量，只要字符串满足右侧的样式即为成立，例如:val LIKE '张*', val LIKE 'J*hn'，*代表通配符)。

#### 2.2.3 分组查询与聚合函数(GROUP BY)

还可以通过SQL查询语句对表格中的数据进行汇总计算，这种计算称为聚合函数。聚合函数包括：

- SUM()--计算总和
- AVG()--计算平均值
- COUNT()--对结果的数量进行统计
- MAX()--从所有结果中找出最大值
- MIN()--从所有结果中找出最小值
- STDDEV()--对结果进行标准差计算
- VARIANCE()--对结果进行方差计算

> 除了COUNT()聚合函数会把空值也算进去之外，其他的聚合函数在计算时都会自动过滤掉空值。

使用聚合函数输出结果之前需要对原有数据进行分组，分组的原则由用户输入，原则上，所有其他没有进行聚合函数计算，并且需要通过SELECT显示的字段都应作为分组依据。比如下面的例子：

```SQL
/* 除了聚合函数之外SELECT中的其他所有字段都应当放到GROUP BY后面
 * 在下面代码示例中，dept和ename都应放到GROUP BY后面作为分组依据
 * 这一段代码能够显示出每个部门收入最高的员工
 */
SELECT dept, ename, MAX(salary) FROM emp
GROUP BY dept, ename;
```

> 聚合函数的结果可以作为限制条件的变量，但是需要用关键字HAVING而不是WHERE来创建限制条件，例如HAVING MAX(salary) > 10000。HAVING条件需要放在GROUP BY后面。

SELECT语句的执行顺序（从上到下依次执行）：

- FROM
- WHERE
- GROUP BY
- HAVING
- SELECT
- ORDER BY

#### 2.2.4 排序查询(ORDER BY)

用户还可以对查询的结果进行排序，例如对学生的考试成绩进行排序，就要使用ORDER BY关键字进行排序。

> ORDER BY语句默认排序方式是升序排列(ASC)，可通过在ORDER BY语句的最后添加DESC关键字的方式，将排序方式更改为降序排列。

```SQL
--将所有考试及格的学生按成绩从高到低排序
SELECT stuname, score FROM exam
WHERE score > 60
ORDER BY score DESC;
```

#### 2.2.5 嵌套查询

嵌套查询是将一个查询块（一个SELECT-FROM-WHERE语句称为一个查询块）嵌套在另一个查询块的WHERE或HAVING条件之后，成为了这个查询块的查询条件。嵌套查询可以让用户将多个简单的查询操作组合在一起，构成较为复杂的查询。

> 以“层层嵌套”的方式来构造程序正是SQL语言“结构化”的体现。

嵌套查询中的子查询分为不相关子查询与相关子查询。其中不相关子查询意为子查询的查询条件不依赖于父查询，与之相对，相关子查询意为子查询的查询条件依赖于父查询。下面是一个简单的嵌套查询例子：

```SQL
--不相关子查询举例
--查询和张三在同一个部门的学生id、姓名以及所在部门
SELECT stuid, stuname, dept FROM student
WHERE dept IN (
    SELECT dept FROM student
    WHERE stuname = '张三');
--相关子查询举例
--查询考试成绩不低于自己平均成绩的课程
SELECT stuid, course FROM exam a
WHERE score >= (
    SELECT AVG(score) FROM exam b
    WHERE a.stuid = b.stuid);
```

嵌套查询中，子查询有时会得到一列/多列值，而不是单一的值，与一列值/多列值进行比较时需要带上限制词ALL或者ANY。

- ANY：子查询返回的结果中存在满足布尔表达式的结果即为真，否则为假；
- ALL：子查询返回的全部结果均满足布尔表达式才为真，否则为假。
- 例如>ANY意为等式左侧大于子查询结果中的最小值即为真，而>ALL意为等式左侧大于子查询结果中的最大值才为真。

```SQL
--查询考试成绩比张三最好成绩还要高的同学的对应科目考试成绩
SELECT stuname, course, score FROM exam
WHERE score > ALL (
    SELECT score FROM exam
    WHERE stuname = '张三');
```

#### 2.2.6 多表连接查询

> 在大规模的数据库系统中，往往由若干个逻辑模式（关系表）组成。
> 数据库设计者不可能设计将所有数据都存放在一张数据库表中。

当用户需要查找来自不同的数据库表中的数据时，就需要进行表连接。
表连接分为内连接、外连接，内连接的例子有等值连接：寻找两个表中相同的字段，若两行中相同字段具有相同的值，那么将这两行进行连接。等值连接把目标列中重复的字段去掉就是自然连接。外连接分为左连接和右连接，左（右）连接就是在等值连接的基础上额外添加左边（右边）表格没有连接上的行的内容，匹配失败的行的对应另一张表的全部字段内容均为空值NULL。除此之外，一张数据库表还可以与自己进行连接（一般是等值连接）。

一般来说，在SELECT语句中的FROM后面添加多个表格即可完成表连接操作，等值连接需要加入相同字段值相同的筛选条件，一个基本的多表连接查询例子：

```SQL
/* 假设students表中有stuid(主键)、stuname
 * department表中有stuid(主键)、dept
 * 可以通过筛选主码相同的连接结果，即可得到等值连接的结果
 * 从等值连接的结果中查找两张表格的所有数据 */
SELECT st.stuid, st.stuname, dp.dept
FROM students st, department dp
WHERE st.stuid = dp.stuid;
```

一个题目：有两个表A和B，均有key和value两个字段，写一个SQL语句，进行如下操作：如果B的key字段的值A中也有，就将B的value换成A中对应的value。

```SQL
UPDATE B, A
SET B.value = A.value
WHERE B.key = A.key;
```

### 2.3 数据操控语言——DML

DML主要是用于修改数据库表的信息，包括增、删、改的操作。

#### 2.3.1 数据添加(INSERT)

向数据库表中添加一条数据：

```SQL
/* 给数据库表的指定列添加一条数据
 * 所有的具有非空约束(NOT NULL)的字段必须有数据，除非定义关系时添加了默认值约束
 * 或者是主键定义了自增约束(AUTO INCREMENT) */
INSERT INTO table_name(tableid, val) VALUES (100, 'John');
/* 给数据库表中所有列按顺序添加一条数据
 * 字段必须按顺序添加，并且每个字段都要添加值，不能有遗漏，空值用NULL */
INSERT INTO table_name VALUES(101, 'Sam');
```

#### 2.3.2 数据修改(UPDATE)

修改数据库中一整列数据：

```SQL
--将table_name表中val字段的所有值都变成Sam
UPDATE table_name
SET val = 'Sam';
```

使用WHERE进行条件筛选，修改指定行的数据：

```SQL
--将tableid为100的行的val字段修改为Sam
UPDATE table_name
SET val = 'Sam'
WHERE tableid = 100;
```

#### 2.3.3 数据删除(DELETE)

删除一整张表的数据：

- 删除一整张表的数据不意味着数据库表（逻辑模式）也会一并删除，数据库表及其结构、约束、触发器等仍存在。
- 如果删除掉的数据中含有作为外键的参考字段，则会按照定义外键时的设置执行操作（阻止删除、级联删除、置为空值等）

```SQL
--删除table_name表中的全部数据，不影响表结构以及依赖其的对象（有外键除外）
DELETE FROM table_name;
--使用TRUNCATE也可立即删除整张表的数据，同时删除索引、触发器等内容，约束条件不变
--使用TRUNCATE可以更有效地删除表中数据，并且留下更少的日志数据
TRUNCATE TABLE table_name;
```

使用WHERE进行条件筛选，删除指定行的数据：

```SQL
--删除table_id为100的行
DELETE FROM table_name
WHERE tableid = 100;
```

### 2.4 数据控制语言——DCL

数据控制语言主要是为数据库管理员管理用户访问相关数据库对象的权限，主要集中于两个关键词：

- GRANT: 授予用户相关权利
- REVOKE: 回收用户相关权利

```SQL
--为guest授权在table_name的表上进行增删改查的权利
GRANT SELECT, INSERT, UPDATE, DELETE ON table_name TO guest;
--回收用户guest对table_name进行增删改查的权利
REVOKE SELECT, INSERT, UPDATE, DELETE ON table_name FROM guest;
```

> 使用GRANT/REVOKE，也可以对整个角色组，甚至是所有用户的权限进行修改。使用角色名字或者PUBLIC关键字代替用户名字，即可对整个角色组或所有角色进行权限管理，一般情况下很少直接对所有角色的权限进行更改。
