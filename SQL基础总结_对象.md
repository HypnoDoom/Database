# Oracle DB 数据库基础知识总结

## 1 数据库的对象

数据库中的对象包含：
- 表
- 视图
- 存储过程
- _同义词_
- 索引
- 触发器
- 约束

下面将对这些数据库对象进行解释分析。

### 1.1 表(TABLE)

在数据库系统中，表是用来存储数据的基本工具，是包含数据库中所有数据的数据库对象。
> Oracle数据库系统中表的存储过程：
> Oracle数据库按照【数据库——表空间——段——区——Oracle（逻辑）数据块】的等级对数据进行存储，其中Oracle逻辑数据块是存储数据库数据的基本单元，是逻辑 I/O 的最小单位。
> Oracle逻辑数据块的大小从2KB到32KB之间。DB_BLOCK_SIZE 参数指定了逻辑块的大小，用户经常需要在建立数据库系统之前为DB_BLOCK_SIZE指定一个合适的值（也是唯一需要用户指定的值，通常将数据库中最经常用到的块的大小作为DB_BLOCK_SIZE的值，一般为4KB或者是8KB），如果用户选择不指定DB_BLOCK_SIZE的大小，那么系统会默认设置一个足够用户使用的值。如果数据库块大小与操作系统块大小不同，在设置数据库块大小时还需确保数据库块大小是操作系统块大小的倍数。
> 特定数目的Oracle逻辑数据块组合成一个“区”，为特定逻辑结构分配的一组“区”形成一个“段”，而表空间通常代表一个“段集合”。创建数据库表时，也会创建用于保存数据的“段”，表中的行列值以“行片段”的形式存储在逻辑数据块中，“行片段”的称呼是因为整行数据在某些情况下不是在连续的空间下存储的，而是分散开来进行存储的，因此称作“行片段”。

使用SQL语言创建一个简单的表：
```SQL
--创建一个名为table_name的表，内含tableid与val两个字段
CREATE TABLE table_name (
    tableid INT NOT NULL,
    val VARCHAR(10) NULL);
```

### 1.2 视图(VIEW)
视图是从一个（或者多个）数据库表或者是视图中导出的虚拟的表，定义视图后或存放在数据字典中，不会存储在数据库表中。使用视图的优点有：

- 使代码变得简洁，节省篇幅
- 能够更好的管理个用户访问数据的权限（开放视图相关权限，关闭原始数据库表的权限）
- 保证数据的独立性

视图分为简单视图与复杂视图，复杂视图一般包含多表数据，有时还会包含聚合数据、函数等。

|                  | 简单视图 | 复杂视图 |
| ---------------- | -------- | -------- |
| 是否包含多表数据 |  不包含  |   包含   |
| 是否可以包含函数 |  不可以  |   可以   |
| 是否可含聚合数据 |  不可以  |   可以   |
| 是否能用DML语句  |    能    | 一般不能 |

- 虽然简单视图可以使用DML语句对数据库表中的数据进行修改，但是仍然不建议在任何视图中使用DML语句对数据进行增、删、改的操作。

使用SQL语言创建一个简单的视图：
```SQL
--从table_name的表中选取tableid为1的行创建视图
CREATE VIEW view_name AS 
    SELECT tableid, val FROM table_name
    WHERE tableid = 1;
```

可以直接引用视图的名称来查找视图内的数据：

```SQL
SELECT * FROM view_name;
```

### 1.3 存储过程(PROCEDURE)

存储过程是存储于大型数据库系统的【服务器】中，一组为了完成特定的功能的SQL语句集，经编译之后永久有效，用户可以通过在SQL语句中输入指定存储过程的名字并附带参数（如果存储过程需要参数）来调用、执行存储过程。存储过程是数据库中的一个重要对象。在数据量特别庞大的情况下利用存储过程能达到倍速的效率提升。
Oracle DB中具有许多预先编译好的存储过程，可以直接引用，例如输出一行内容：

```SQL
DBMS_OUTPUT.PUT_LINE('Hello World');
```

除此之外，用户也可以在数据库中自己定义存储过程：
```SQL
CREATE PROCEDURE my_procedure IS
    v_table_id table_name.tableid%TYPE;
    v_val table_name.val%TYPE;
BEGIN
    v_table_id := 100;
    v_val = 'John';
    INSERT INTO table_name(tableid, val) VALUES(v_table_id, v_val);
    --SQL支持在存储过程中嵌套调用其他的定义好的存储过程
    DBMS_OUTPUT.PUT_LINE(' Inserted '|| SQL%ROWCOUNT ||' row ');
END;
```

调用上面预先定义好的存储过程my_procedure：

```SQL
BEGIN
    my_procedure;
END;
/
SELECT * FROM table_name WHERE tableid = 100;
```

与存储过程相类似的概念还有函数，函数相较于存储过程限制较多：

|          |   函数   |  存储过程  |
| -------- | -------- | ---------- |
|  保留字  | FUNCTION | PROCEDURE |
|  返回值  | 必须有一个返回值 | 可以有0，1或多个 |
| 可用语句 | 只有SELECT...INTO... | 可使用更多语句 |
|   调用   | 可在查询语句调用 | 必须单独调用 |

下面是一个用户自定义函数以及调用方法的示例：

```SQL
CREATE FUNCTION check_id(v_input_id table_name.tableid%TYPE)
RETURN BOOLEAN IS
    v_table_id table_name.tableid%TYPE;
    v_val table_name.val%TYPE;
    v_msg VARCHAR(3) := 'John';
BEGIN
    SELECT tableid, val INTO v_table_id, v_val FROM table_name
    WHERE tableid = v_input_id;
    IF v_val = v_msg THEN
        RETURN TRUE;
    ELSE
        RETURN FALSE;
    END IF;
EXCEPTION
    --添加意外处理办法
    RETURN NULL;
END;
```

函数调用示例：

```SQL
BEGIN
    IF (check_id(100) IS NULL) THEN
        DBMS_OUTPUT.PUT_LINE('The function returned NULL due to exception');
    ELSIF (check_id(100)) THEN
        DBMS_OUTPUT.PUT_LINE('This ID matches John.');
    ELSE
        DBMS_OUTPUT.PUT_LINE('This ID doesn't match John.);
    END IF;
END;
```

### 1.4 同义词(SYNONYM)

表、视图等所有数据库对象以及函数、类型都可使用同义词来进行引用，其作用主要是简化命令、简化代码，使其变得更加易读，并且方便用户对数据库对象进行一系列操作。在Oracle DB中，可以创建私有/公有同义词，私有同义词只有具有使用权限的用户才可以使用，公有同义词所有用户均可以使用。

创建一个私有同义词：
```SQL
--为视图view_name创建一个私有同义词
CREATE SYNONYM id_view FOR view_name;
```
创建一个公有同义词：
```SQL
--为视图view_name创建一个公有同义词
CREATE PUBLIC SYNONYM id_view FOR view_name;
```

### 1.5 索引(INDEX)

索引是对数据库表中一列或多列的值进行排序的一种结构，使用索引可快速访问数据库表中的特定信息。为数据库表中添加索引能够：

- 更快速的查询到指定信息，减少多数查询对系统资源的消耗
- 让用户能够更快速的定位到所需数据
- 能够加快表与表之间的连接速度
- 添加唯一性索引，能够保证数据的唯一性与独立性

但是添加索引也有一定的不足：

- 需要花费更多时间来对数据库表进行维护（增、删、改需要额外对索引进行相应的操作，称为动态维护）
- 索引储存在数据库中，需要额外的物理空间消耗

因此对于经常进行增、删、修改数据的数据库表，则不建议添加索引以减少不必要的资源开销。

对于唯一索引，比较常见的几种有：

- 在创建数据库表时，添加主键约束时，系统会自动添加主键索引（主键索引是唯一索引的一种特例，主键PRIMARY KEY=非空性NOT NULL+唯一性UNIQUE）
- 在创建数据库表时，添加唯一性约束时，系统会自动添加唯一索引
- 建议优先使用添加约束条件的方式添加索引（约束部分内容见1.7）

添加自定义的索引示例：
```SQL
--添加索引
CREATE INDEX tab_id_val ON table_name(tableid, val);
--使用下面的SELECT语句时将会提升查询速度(不唯一)
SELECT * FROM table_name WHERE tableid = 100 AND val = 'John';
```

### 1.6 触发器(TRIGGER)

触发器是提供给程序员或者数据分析员的一种保证数据完整性的方法，触发器中的内容不是通过引用调用，也不是通过手工启动，而是通过用户在增、删、改等对数据库表进行修改操作的时候（之前或之后）执行。触发器可以包含简单的查询语句，也可以包含多种复杂的SQL语句，也可用于数据库所有者对用户的权限管理（例如非管理员身份不能修改删除数据，用户不能在非工作时段修改删除数据等）。

手动添加一个行级触发器，阻止用户在非工作时段对table_name表进行修改：
```SQL
CREATE TRIGGER new_trigger 
BEFORE INSERT OR UPDATE OR DELETE ON table_name
--这里可以添加FOR EACH ROW标注为行级触发器，也可以不添加
BEGIN
    IF (TO_CHAR(sysdate，‘DY’ IN (‘SAT’，‘SUN’)) 
    OR (TO_NUMBER(sysdate，‘HH24’) NOT BETWEEN 8 AND18) THEN
        IF INSERTING THEN 
            --RAISE_APPLICATION_ERROR()是预定义好的存储过程，用于THROW EXCEPTION
            RAISE_APPLICATION_ERROR(-20500，
            'You may only insert into the table during working hours.');
        ELSIF DELETING THEN
            RAISE_APPLICATION_ERROR(-20502，
            'You may only delete from the table during working hours.');
        ELSE THEN
            RAISE_APPLICATION_ERROR(-20504，
            'You may only update the table during working hours.');
        END IF;
    END IF;
END;
```

添加触发器之后，用户如果在非工作时段（非周一至周五的8点-18点）进行以下操作时，将会收到提示信息并且不会对数据库表进行修改：

```SQL
UPDATE table_name SET val = 'Sam' WHERE tableid = 100; --收到错误-20504
INSERT INTO table_name(tableid, val) VALUES (101, 'Sam'); --收到错误-20500
DELETE FROM table_name WHERE tableid = 100; --收到错误-20502
```

### 1.7 约束(CONSTRAINT)

常见的约束包含：

- 非空约束(NOT NULL)
- 默认值约束(DEFAULT)
- 唯一性约束(UNIQUE)
- 主键约束(PRIMARY KEY)
- 外键约束(FOREIGN KEY)
- 检查约束(CHECK)

非空约束：具有非空约束的字段值不能够为NULL，否则会报错；
创建数据库表时，为数据库表的某个字段添加非空约束：

```SQL
CREATE TABLE new_table (
    tableid INT NOT NULL,
    val VARCHAR(10));
```

默认值约束：在定义字段名时添加，用于保证添加数据时含有缺省输入值时，为这些字段设置一个默认的值；
为数据库表的某个字段添加默认值约束：

```SQL
CREATE TABLE new_table (
    tableid INT,
    val VARCHAR(10) DEFAULT NULL);
```

唯一性约束：约束输入/修改数据库表时该字段的每一个值唯一存在，试图添加字段中已有数值的对应数据将会报错；

> 添加唯一性约束的同时还会添加一个唯一性索引，用于加快数据检索速度。

为数据库表的某个字段添加唯一性约束：

```SQL
CREATE TABLE new_table (
    tableid INT,
    val VARCHAR(10),
    CONSTRAINT un_id UNIQUE(tableid));
```

主键约束：主键约束是能够区分表中每一行数据的标识符，主键约束在唯一性约束的基础上还添加了非空条件；

> 用户在添加主键约束的同时，系统还会自动添加一个主键索引，用于加快数据检索速度。

为数据库表的某个字段添加主键约束：

```SQL
CREATE TABLE new_table (
    tableid INT,
    val VARCHAR(10),
    CONSTRAINT pk_id PRIMARY KEY(tableid));
```

外键约束：在一个关系中，存在一个公共关键字是主键/唯一键，那么这个字段在从属关系中称为外键。外键约束能够保证多表之间进行连接时的数据完整性。

> 通常会在子表添加外键约束，外键约束参考的（字段所在的）表称作父表，外键约束要求子表中受外键约束的字段的值必须在参考的父表字段中出现过。外键约束要求参考的父表字段必须是主键或者是唯一键，否则无法为子表添加外键约束。

为数据库表的某个字段添加外键约束：

```SQL
CREATE TABLE new_table (
    tableid INT,
    val VARCHAR(10)
    --添加外键的前提是父表参考字段必须是主键或唯一键
    CONSTRAINT fk_id FOREIGN KEY(tableid) REFERENCES table_name(tableid)
    ON DELETE CASCADE
    ON UPDATE SET NULL);
```

检查约束：用户在添加、修改数据库表中的数据时，系统会检查添加检查约束的字段值是否满足条件，如果不满足条件将会报错。
为数据库表的某个字段添加检查约束：

```SQL
CREATE TABLE new_table (
    tableid INT,
    val VARCHAR(10),
    CONSTRAINT chk_id CHECK(tableid > 0));
```
