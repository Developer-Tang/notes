## 常见索引类型

- **普通索引：** 仅加速查询
- **唯一索引：** 加速查询 + 列值唯一（可以有null）
- **主键索引：** 加速查询 + 列值唯一 + 表中只有一个（不可以有null）
- **联合索引：** 多列值组成一个索引，专门用于组合搜索

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
|  1 | PRIMARY     | t_user  | ALL   | NULL          | NULL    | NULL    | NULL |    9 | NULL        |
+----+-------------+------------+-------+---------------+---------+---------+------+------+-------------+
```

- **select_type**
    - *SIMPLE：* 简单查询
    - *PRIMARY：* 最外层查询
    - *SUBQUERY：* 映射为子查询
    - *DERIVED：* 子查询
    - *UNION：* 联合
    - *UNION RESULT：* 使用联合的结果

- **table：** 正在访问的表名

- **type**

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
      select *  from t_user where id <100
      # >
      # >= 
      # <=
      # <
      # in()
      # between a and b
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

    - *CONST：* 常量，表最多有一个匹配行,因为仅有一行,在这行的列值可被优化器剩余部分认为是常数,const表很快,因为它们只读取一次。

      ```sql
      select id from t_user where id = 2
      ```

    - *SYSTEM：* 系统

- **possible_keys：** 可能使用的索引

- **key：** 真实使用的索引

- **key_len：** MySQL中使用索引字节长度

- **rows：** mysql估计为了找到所需的行而要读取的行数  *--只是预估值*

- **extra：** 该列包含MySQL解决查询的详细信息

## 造成索引失效的原因

```sql
-- 使用like查询
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
where addr="" and sex=0				#不走索引
```
