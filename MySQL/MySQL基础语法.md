## DDL数据定义

### 创建数据库

```sql
create database databaseName;
```

### 删除数据库

```sql
drop
databaseName; 
```

### 创建数据表

```sql
create table tableName
(
    colName1 type1 [not null] [primary key],
    colName2 type2 [not null], ...
)
```

### 删除数据表

```sql
drop table tableName 
```

### 添加列

```sql
alter table tableName
    add column colName type
```

### 删除列

```sql
alter table tableName
    drop colName
```

### 修改列类型

```sql
alter table tableName
    alter column colName type
```

### 修改列名

```sql
exec sp_rename 'tableName.column1', 'column2'
```

### 列添加默认值

```sql
alter table tableName
    add constraint varName default defaultValue for columnName
```

### 增加主键

```sql
alter table tableName
    add primary key (colName)
```

### 删除主键

```sql
alter table tableName
    drop primary key(colName)
```

### 创建索引

> - **normal：** 普通索引
> - **unique：** 唯一
> - **fulltext：** 全文索引，仅 **MyISAM** 类型表可用
> - **spatial：**  空间索引，同样仅 **MyISAM** 类型表可用

```sql
create
[unique] index indexName on tableName(colName….)
```

### 删除索引

```sql
drop index indexName
```

### 创建视图

```sql
create view viewName as
select cloName
from tableName
```

### 删除视图

```sql
drop view viewName
```

## DML数据操作

### 插入数据

```sql
insert into tableName(colName1, colName2[, colName...])
values (colValue1, colValue2[, colValue...])[,(colValue1,colValue2[,colValue...])]
```

### 拷贝其他表

```sql
insert into tableName1(colName1, colName2[, colName...])
select colName3 as colName1, colName4 as colName2[,colName as 插入表的字段名...]
from tableName2
```

### 删除数据

> 整表删除，无log日志

```sql
truncate table tableName
```

> 按照条件逐条删除，有日志

```sql
delete
from tableName
where 条件表达式...
```

### 修改数据

```sql
update tableName
set colName  = colValue[,
    colName2 = colvalue2...]
where 条件表达式...
```

## DQL数据查询

### 简单查询
> **=** 等值判断
> 
> **>=** 、 **>** 、 **<** 、 **< =** 、 **between value1 and value2** 、 **in(value1,value2...)** 范围判断
> 
> **like '%value%'** 字符串模糊匹配 `%` 匹配多个文字， `.` 匹配单个文字
```sql
select colName1, colName2[, colName...]
from tableName
where colName1 = "" AND colName2 between 1 and 100 AND colName3 in ("A","B","C")
```

### 分组查询

```sql
select *
from tableName
where colName = colValue
group by colName
```

### 排序查询

> 默认按order by后面的字段升序排列

```sql
select *
from tableName
where colName = colValue
order by colName asc|desc
```

### 分页查询

> **offset：** 起始的行数，不包含在查询结果内，不传默认从0开始
>
> **size：** 每页的数量

```sql
select *
from tableName
where colName = colValue
limit offset,size
```

### 查询后结果筛选

```sql
select *
from tableName
where colName = colValue
group by colName
having 筛选条件表达式
```

### 多表内连接查询

> 取连接表的交集

```sql
select *
from tableName1 t1
         inner join tableName2 t2 on t1.colName = t2.colName,
     其他关联条件
where (t1 | t2).colName = colValue
```

### 多表左连接查询

> 左连接默认优先满足左表，右表有没有都可以不影响

```sql
select *
from tableName1 t1
         left join tableName2 t2 on t1.colName = t2.colName,
     其他关联条件
where t1.colName = colValue
```

### 多表右连接查询

> 与左连接刚好相反，右连接默认优先满足右表，左表有没有都可以不影响

```sql
select *
from tableName1 t1
         right join tableName2 t2 on t1.colName = t2.colName
    其他关联条件
where (t1|t2).colName = colValue
```

### 全连接

> 将多个查询结果进行拼接 ，union 会去重复行，union all不会

```sql
select *
from tableName
where colName = colValue
union all
select *
from tableName
where colName = colValue
```

## DCL权限控制