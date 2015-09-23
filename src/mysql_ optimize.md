##mysql的优化
1. **适当增加索引（个人觉得这是废话）**
2. **查看各种SQL执行的频率**
 
 > mysql> show status (like 'Com_select');
 
 Com_insert写入次数,Com_delete删除次数,connections(试图连接mysql服务的次数),uptime(mysql工作时间),slow_queries(慢查询次数)[具体参数]()

3. **定位执行效率较低的SQL语句**  
 通过慢查询日志，定位查询效率低下的SQL语句，然后分析语句进行优化
 > show variables like 'slow_query_log%';  
 
 Variable_name | Value
 --- | ---
 slow_query_log| OFF
 slow_query_log_file| /usr/local/var/mysql/chaitao-MacBook-Pro-slow.log
 通过修改以上值可以配置慢查询日志，当让也可以在my.cnf配置中进行配置
 > slow_query_log=on  
 > slow_query_log_file=log_path
 
4. **通过explain或desc分析SQL语句的执行计划，如要查看所访问的分区使用explain partitions**
 > mysql> desc select * from rbac_role where role_id = 1\G;  
*************************** 1. row ***************************  
           id: 1  
  select_type: SIMPLE  
        table: NULL  
         type: NULL  
possible_keys: NULL  
          key: NULL  
      key_len: NULL  
          ref: NULL  
         rows: NULL  
        Extra: Impossible WHERE noticed after reading const tables  
 
 > **id**包含一组数字,表示查询在执行中的select子句或者操作表的顺序.  
 (1)id相同,执行顺序由上至下  
 (2)子查询中,id的序号会递增,id越大优先级越高,越先被执行.  
 (3)id如果相同,可以认为是一个group,从上往下顺序执行;在所有的group中,id值越大,优先级越高,越先被执行  
 
 > **select_type**: 表示查询中每个SELECT子句的类型.   
  *SIMPLE*: 查询中不包含子查询或者UNION.  
  *PRIMARY*:查询中包含任何复杂的子部分,最外层查询被标记为PRIMARY.  
  *SUBQUERY*:在SELECT或者WHERE列表中包含了子查询,该子查询被标记为SUBQUERY.  
  *DERIVED*:在FROM列表中包含的子查询被标记为DERIVED.  
  *UNION*:若第二个SELECT出现在UNION之后,则被标记为UNION;若UNION包含在FROM子句的子查询中,则被标记为DERIVED.  
  *UNION RESULT*:从UNION表获取结果的SELECT被标记为UNION RESULT.
 
 > **type**:表示MySQL执行查询时是如何获取所需行的,又称”访问类型”(从上至下,执行效率右最差到最好)  
 *ALL*: Full Table Scan,MySQL扫描全表以找到匹配的行.  
 *index*: Full Index Scan,index与ALL的区别在于index只遍历索引树  
 *range*: 索引范围扫描,对索引的扫描开始于某一点,返回匹配值域的行.  
 *ref*: 非唯一性索引扫描,返回匹配某个值的所有.  
 *eq_ref*: 唯一性索引扫描,对于某个索引键,表中只有一条记录与之匹配,常见于主键或者唯一索引查询.  
 *const,system*: 当MySQL优化部分查询,并转换为一个常量时,使用这些类型访问.如将主键置与where列表中,MySQL就能将该查询替换为一个常量.需要注意的是system是const的特殊类型,当查询的表只有一行的情况下使用system.  
 *NULL*: MySQL在优化过程中分解语句,执行时不用访问表或者索引.
 
 > **possible_keys**:MySQL能够使用那个索引找到匹配的行,查询涉及到字段上若存在索引,则该索引将被列出,但是不一定被查询使用到.
 
 > **key**: MySQL在查询中实际使用的索引,若没有使用索引,将显示为NULL.若使用了覆盖索引,该索引仅出现在key列表中.
 
 > **key_len**:表示索引中使用的字节数,可以通过该列计算查询中使用索引的长度.(key_len 显示的值是索引字段的最大可能长度,并非使用长度,key_len是跟进表定义计算得到的,而不是检索表的实际内容得到的)
 
 > **ref**:表示上述表的连接匹配条件,既那些列或者常量用于查询索引列上的值.
 
 > **row**:表示MySQL根据表的统计信息及索引选用情况,估算找到匹配行所需要读取的行数.
 
 > **Extra**:包含不适合在其他列显示但是十分重要的额外消息.
 
5. **使用profile分析SQL，profile就是详细地列出SQL语句执行过程**  
 查看是否开启/支持profile  
 > mysql> show variables like 'profiling';  
 如果是关闭状态，则开启  
 > mysql> set profiling = on;
 查看所有查询执行的时间  
 > mysql> show profiles;  
 上述命令会展示执行sql的时间，如果查看某一个运行的详细信息则可以  
 > mysql> show profile for query (Query_id);
 
6. **常用SQL优化**
 * 加载大量数据时，关闭非唯一索引，取消唯一性检查，以及取消自动提交以提高插入速度
 
 > mysql> set unique_checks=0   
 > mysql> alter table stu disable keys   
 > mysql> set autocommit=0  
 > mysql> load load infile........  
 > mysql> alter table stu enable keys  
 > mysql> set unique_checks=1  
 > mysql> set autocommit =1  

 * insert语句优化，一次性插入多条记录
 * order by语句排序优化，优化思路就是尽可能的减少额外的排序(filesort)，通过索引直接返回有序数据
 * where条件和order by 字段使用相同的索引，并且order by的顺序和索引顺序相同，还有order by的字段都是降序或者升序。
 * SELECT查询时最好指定具体的字段名，SELECT * 会选择所有字段，会增加排序区的使用，降低SQL性能
 * group by语句优化 MySQL默认情况下对group by col1,col2..的字段进行排序，可以通过指定order by null来消除这种排序
 * or条件优化 用到OR的查询，如果要使用索引，那OR之间的每个条件必须是索引，并且要分别建立索引，不能使用联合索引。
 
 
 