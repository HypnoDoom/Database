# Oracle DB 数据库基础知识总结

### 1 数据库的对象

数据库中的对象包含：
- 表
- 视图
- 存储过程
- _同义词_
- 索引
- 触发器
- 约束

下面将对这些数据库对象进行解释分析。

#### 1.1 表(TABLE)

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

#### 1.2 视图(VIEW)
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

#### 1.3 存储过程(PROCEDURE)

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

#### 1.4 同义词(SYNONYM)

表、视图等所有数据库对象以及函数、类型都可使用同义词来进行引用

### 2 SQL基础总结

- SQL语言主要包含四类: DDL(Data Definition Language)--数据定义语言, DQL(Data Query Language)--数据查询语言, DCL(Data Control Language)--数据控制语言, DML(Data Manipulation Language)--数据操控语言。