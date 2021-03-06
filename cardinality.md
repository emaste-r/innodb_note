# Innodb 索引列的Cardinality
## 0x0 背景
```
并不是在所有的查询条件中出现的列都需要添加索引。
对于什么时候添加B+树索引，一般的经验是，在访问表中很少一部分时使用B+树索引才有意义。
我们倾向于：
拒绝低选择性 如性别，可能筛选出来50%的rows
崇尚高选择性 如用户名，几乎唯一。
```

## Ox1 什么是Cardinality
```
Cardinality值非常关键，表示索引中不重复记录数量的预估值。
同时需要注意的是，Cardinality是一个预估值，而不是一个准确值，基本上用户也不可能得到一个准确的值。
```
	
## Ox2 如何判断高选择性
```
show index 查看 Cardinality
Cardinality/n_rows_in_table越接近1，越高
```
		
## Ox3 Innodb更新Cardinality的时机
```
  策略1：表中1/16的数据已发生过变化
  策略2：stat_modified_counter＞2 000 000 000 （防止策略1重复更新一行，总更新行数没怎么变化）
```

## Ox4 计算Cardinality算法
```
1、获取B+树索引中叶子节点的数量，记为A
2、随机找到8个叶子结点（Leaf Page）每个页不同记录的个数,P1,P2...P8
3、(P1+P2+..P8) *A / 8
```

## Ox5 影响参数
### innodb_stats_sample_pages
```
设置采样页的数量，默认8
```
### innodb_stats_method
```
如何对待索引中出现NULL值记录
			nulls_equal 视为相等
				索引页：NULL、NULL、1、2、2、3、3、3，Cardinality=4
			nulls_unequal 视为不相等
				索引页：NULL、NULL、1、2、2、3、3、3，Cardinality=5
			nulls_ignored 视为忽略
				索引页：NULL、NULL、1、2、2、3、3、3，Cardinality=3
```
### innodb_stats_on_metadata
```
每次show tables; show index; 访问information_schema架构下的tables以及statistics的时候
是否需要重新计算索引的Cardinality值
```
### innodb_stats_persistent
```
是否要把Cardinality的值保存到磁盘上，重启db的时候减少重新计算
```

## 0x6 总结：
![image](https://github.com/emaste-r/innodb_note/blob/master/imgs/Innodb%20Cardinality.svg)
