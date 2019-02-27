---
layout: post
title:  "mysql select的执行顺序"
date:   2019-02-25
categories: mysql
tag: select
---

* content
{:toc}

# 执行顺序 #

为了学会优化sql,我们必先了解sql语句执行的顺序，从数据的根源减少数据的查询量

编写sql的顺序：

	SELECT DISTINCT
	    < select_list >
	FROM
	    < left_table > < join_type >
	JOIN < right_table > ON < join_condition >
	WHERE
	    < where_condition >
	GROUP BY
	    < group_by_list >
	HAVING
	    < having_condition >
	ORDER BY
	    < order_by_condition >
	LIMIT < limit_number >

sql执行的顺序:
	
	1. FROM < left_table >
	2. ON < join_condition >
	3. < join_type > JOIN < right_table >
	4. WHERE < where_condition >
	5. GROUP BY < group_by_list >
	6. HAVING < having_condition >
	7. SELECT
	8. DISTINCT < select_list >
	9. ORDER BY < order_by_condition >
	10. LIMIT < limit_number >

具体解释sql执行的每一个阶段

步骤 | 含义
---|---
FROM | 计算两个关联表的笛卡尔积，生成虚拟表VT1
ON | 基于VT1这一个虚拟表进行过滤，过滤出满足ON条件的列，生成虚拟表VT2
JOIN | 使用了外连接（LEFT,RIGHT,FULL），主表（保留表）中的不符合ON条件的列也会被加入到VT2中，作为外不行，生成虚拟表VT3
WHERE | 对VT3过程中生成的临时表进行过滤，满足where子句的列被插入到VT4表中
GROUP BY | 这个子句会把VT4中生成的表按照GROUP BY中的列进行分组。生成VT5
HAVING | 这个子句对VT5表中的不同的组进行过滤，只作用于分组后的数据，满足HAVING条件的子句被加入到VT6表中
SELECT | 这个子句对select子句中的元素进行处理，生VT7表
DISTINCT | 对VT7中的记录进行去重。产生虚拟表VT8
ORDER BY | 将虚拟表VT8中的记录按照<order_by_condition>进行排序操作，产生虚拟表VT9
LIMIT | 取出执行行的记录，产生虚拟表VT10，并将结果返回


> * where与on的区别（对主表的过滤应该放在WHERE,对于关联表，先条件查询后连接则用ON,先连接后条件查询则用WHERE）  
>  1. 如果有外部列，ON针对过滤的是关联表，主表（保留表）会返回所有的列
>  2. 如果没有添加到外部列，两者的效果是一样的
>
> * offset和rows的正负带来的影响（当偏移量很大时，效率会很低），可以如下处理  
>  1. 采用子查询的方式优化，在子查询里先从索引获取到最大的ID，然后倒叙排列，再取rows行结果集
>  2. 采用INNER JOIN优化，JOIN子句里也优先从索引获取ID列表，然后直接关联查询获得最终结果



# 参考资料 #

[https://blog.csdn.net/yadicoco49/article/details/77976785](https://blog.csdn.net/yadicoco49/article/details/77976785)

[https://www.cnblogs.com/annsshadow/p/5037667.html](https://www.cnblogs.com/annsshadow/p/5037667.html)
