## 常见索引类型

- **普通索引** ：仅加速查询
- **唯一索引** ：加速查询 + 列值唯一（可以有null）
- **主键索引** ：加速查询 + 列值唯一 + 表中只有一个（不可以有null）
- **联合索引** ：多列值组成一个索引，专门用于组合搜索

## 如何选择索引字段

尽管索引能提高查询性能，但不当地使用反而起不到作用。在加索引时需要注意以下几点：

1. **选择适合的列作为索引** ：经常作为查询条件（WHERE 子句）、排序条件（ORDER BY 子句）、分组条件（GROUP BY 子句）的列是建立索引的好选择。对于区分度低的字段不要建索引，例如性别、类型枚举等。频繁更新的字段同样不要作为主键或者索引，因为更新索引得成本会大幅提升。同样不建议用无序的值(例如身份证、UUID )作为索引，当主键具有不确定性，会造成叶子节点频繁分裂，出现磁盘存储的碎片化
2. **避免过多的索引** ：每个索引都需要占用额外的磁盘空间。更新表（INSERT、UPDATE、DELETE 操作）时，所有的索引都需要被更新。维护索引文件需要成本；还会导致页分裂，IO 次数增多
3. **利用前缀索引和索引列的顺序** ：对于字符串类型的列，可以考虑使用前缀索引来减少索引大小。在创建复合索引时，应该根据查询条件将最常用作过滤条件的列放在前面

## SQL索引分析

```sql
explain
    查询语句
```

```shell
mysql> explain select * from t_user;
+----+-------------+------------+-------+---------------+---------+---------+------+------+-------------+
| id | select_type | table      | type  | possible_keys | key     | key_len | ref  | rows | Extra       |
+----+-------------+------------+-------+---------------+---------+---------+------+------+-------------+
|  1 | PRIMARY     | t_user     | ALL   | NULL          | NULL    | NULL    | NULL |    9 | NULL        |
+----+-------------+------------+-------+---------------+---------+---------+------+------+-------------+
```

- **select_type**
    - *SIMPLE：* 简单查询
    - *PRIMARY：* 最外层查询
    - *SUBQUERY：* 映射为子查询
    - *DERIVED：* 子查询
    - *UNION：* 联合
    - *UNION RESULT：* 使用联合的结果

- **table** ：正在访问的表名

- **type** ：索引类型

  > `all` < `index` < `range` < `index_merge` < `ref_or_null` < `ref` < `eq_ref` < `system\const`

    - *ALL：* 数据表全表扫描

      ```sql
      select * from t_user
      ```

    - *INDEX：* 全索引扫描

      ```sql
      select id from t_user
      ```

    - *RANGE：* 对索引列进行范围查找

      ```sql
      # >、>=、<=、<、between a and b
      select *  from t_user where id < 100
      ```

    - *INDEX_MERGE：* 合并索引，使用多个单列索引搜索

      ```sql
      select *  from t_user where nickname = 'abc' or id in (1,10,100)
      ```

    - *REF：*  根据索引查找一个或多个值

      ```sql
      select *  from t_user where nickname = 'abc'
      ```

    - *EQ_REF：* 连接时使用 `primarykey` 或  `unique` 类型

      ```sql
       select tb2.id,tb1.nickname from tb2 left join tb1 on tb2.id = tb1.id;
      ```

    - *CONST：* 常量，表最多有一个匹配行,因为仅有一行,在这行的列值可被优化器剩余部分认为是常数,const表很快,因为它们只读取一次

      ```sql
      select id from t_user where id = 2
      ```

    - *SYSTEM：* 系统

- **possible_keys** ：可能使用的索引

- **key** ：真实使用的索引

- **key_len** ：MySQL中使用索引字节长度

- **rows** ：mysql估计为了找到所需的行而要读取的行数  *--只是预估值*

- **extra** ：该列包含MySQL解决查询的详细信息

## 造成索引失效的原因

```sql
-- 使用左like查询 'x%'可以走索引
select *
from t_user
where nickname like '%bc';

-- 使用函数
select *
from t_user
where reverse(nickname) = "cba";

-- 类型不一致，比如varchar条件写int
select *
from t_user
where nickname = 123;

-- !=，主键还好，其他索引会失效
select *
from t_user
where nickname != "abc";

-- >，主键或索引是整数类型走索引，其他不会
select *
from t_user
where nickname > 'abc';

-- order by，当查询字段非主键或非索引字段不走索引
select id, nickname, sex
from t_user
order by nickname desc;

-- 组合索引最左原则，假设(nickname,sex,addr)为组合索引，但遇到nickname是才会走组合索引，且后面必须按照建立组合索引是的顺序，可以没有但不能间隔、乱序
where nickname="" and sex=0 and addr=""	#走索引
where nickname=""						#走索引
where nickname="" and addr="" and sex=0	#不走索引
where sex=0 and addr=""  				#不走索引
```
