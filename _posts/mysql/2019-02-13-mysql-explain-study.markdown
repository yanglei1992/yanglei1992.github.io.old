---
layout: post
title:  "mysql explain用法和结果的含义"
date:   2019-02-13
categories: mysql
tag: explain
---

* content
{:toc}

# 初步了解 #
&nbsp;&nbsp;&nbsp;&nbsp;explain能够查询mysql是如何使用索引来处理select语句以及表连接的，学习他可以帮助我们选择更加好的索引和写出更加优化的查询语句。

使用方法

- 在select前面加上explain就好了

mysql执行计划分析的好处：

- sql是如何进行索引的
- 联接查询的执行顺序
- 查询扫描的数据行数

mysql执行计划的限制:

- 无法展示存储过程，触发器，udf对查询的影响
- 无法使用explain对储存过程进行分析
- 早期版本的mysql只支持对select语句进行分析


# 查询结果的含义 #


<table>
    <tr>
        <th>查询结果字段</th> 
        <th>含义</th> 
	</tr>

    <tr>
        <td rowspan="3">id</td>
        <td>id列中的数据为一组数组，表示执行select语句的顺序</td>
    </tr>
    <tr>
        <td>id值相同时，执行顺序由上至下</td>
    </tr>
    <tr>
		<td>id值越大优先级越高，越先被执行</td>
    </tr>

    <tr>
        <td rowspan = "8">select_type</td>
        <td><b>SIMPLE</b>:简单select(不包含子查询或是union操作的查询)</td>
    </tr>
    <tr>
        <td><b>PRIMARY</b>:最外面的select(查询中如果包含任何子查询，那么最外层的查询则被标记为primary)</td>
    </tr>
    <tr>
        <td><b>UNION</b>:UNION中的第二个或后的的select语句</td>
    </tr>
    <tr>
        <td><b>DEPENDENT UNION</b>:UNION中的第二个或后面的select语句，取决于外面的查询</td>
    </tr>
    <tr>
        <td><b>UNION RESULT</b>:union查询的结果</td>
    </tr>
    <tr>
        <td><b>SUBQUERY</b>:子查询中的第一个select</td>
    </tr>
    <tr>
        <td><b>DEPENDENT SUBQUERY</b>:子查询中的第一个select，取决于外面的查询</td>
    </tr>
    <tr>
        <td><b>DERIVED</b>:导出表的select（from子句的子查询）</td>
    </tr>

    <tr>
        <td rowspan="3">table</td>
        <td>输出的行所引用的表</td>
    </tr>
    <tr>
        <td><b>< unionM,N></b>由ID为M，N查询union产生的结果集</td>
    </tr>
    <tr>
        <td><b>< derivedN>/< subqueryN></b>由ID为N的查询产生的结果</td>
    </tr>

    <tr>
        <td rowspan="12">type</td>
        <td>联接类型。下面给出各种联接类型，按照从最佳类型到最坏类型进行排序</td>
    </tr>
    <tr>
        <td><b>system</b>:表仅有一行（=系统表）。这是const联接类型的一个特例</td>
    </tr>
    <tr>
        <td><b>const</b>:表最多有一个匹配行，他将在查询开始时被读取。因为仅有一行，在这行的列值可被优化其剩余部分认为是常数。const表很快，因为他们只读取一次</td>
    </tr>
    <tr>
        <td><b>eq_ref</b>:对于每个来自于前面的表的行组合，从表中读取一行。这可能是最好的联接类型，出来const类型</td>
    </tr>
    <tr>
        <td><b>ref</b>:对于每个来自于前面的表进行组合，所有有匹配索引值的行将从这张表中读取</td>
    </tr>
    <tr>
        <td><b>ref_or_null</b>:该链接类型如同ref,但是添加了MySLQ专门搜索包含NULL值的行</td>
    </tr>
    <tr>
        <td><b>index_merge</b>:该联接类型表示使用了索引合并优化方法</td>
    </tr>
    <tr>
        <td><b>unique_subquery</b>:该类型替换了下面形式的IN子查询的ref:value in(select primary_key from single_table where some_expr) unique_subquery是一个索引查找函数，可以完全替换子查询，效率更高</td>
    </tr>
    <tr>
        <td><b>index_subquery</b>:该联接类型类似于unique_subquery。可以替换IN子查询，但只适合下列形式的子查询中的非唯一索引：value IN (SELECT key_colunm FROM single_table WHERE some_expr)</td>
    </tr>
    <tr>
        <td><b>range</b>:只检索给定范围的行，使用一个索引来选择行</td>
    </tr>
    <tr>
        <td><b>index</b>:该联接类型与ALL相同，除了只有索引书被扫描。这通常比ALL快，因为索引文件通常比数据文件小。</td>
    </tr>
    <tr>
        <td><b>ALL</b>:对于每个来自于先前的表的行组合，进行完成的全表扫面</td>
    </tr>

    <tr>
        <td rowspan="2">possible_keys</td>
        <td>指出MySQL能使用哪个索引来优化查询</td>
	</tr>
    <tr>
        <td>查询列所涉及的列上的索引都会被列出来，但不一定会被使用</td>
	</tr>

    <tr>
        <td rowspan="2">key</td>
        <td>查询优化器优化查询实际所使用的索引</td>
	</tr>
    <tr>
        <td>如果没有可用的索引，则显示null</td>
	</tr>

    <tr>
        <td rowspan="2">key_len</td>
        <td>表示该索引字段的最大的可能长度</td>
	</tr>
    <tr>
        <td>key_len的长度有字段定义计算而来，并非数据的实际长度。如果键值为NULL在，则长度为NULl</td>
	</tr>

    <tr>
        <td rowspan="1">ref</td>
        <td>显示使用哪个列或常数与key一起从表中选择行。</td>
	</tr>

    <tr>
        <td rowspan="1">rows</td>
        <td>显示使msyql认为它执行查询时必须检查的行数。多行之间的数据相乘可以估算要处理的行数。</td>
	</tr>

    <tr>
        <td rowspan="10">Extra</td>
        <td>该列包含MySQL解决查询的详细信息</td>
	</tr>
    <tr>
        <td><b>Distinct</b>:MySQL发现第1个匹配行后，停止为当前的行组合搜索更多的行</td>
	</tr>
    <tr>
        <td><b>Not exists</b>:MySQL能够对查询进行LEFT JOIN优化，发现1个匹配LEFT JOIN标准的行后，不再为前面的行组合在该表内检查更多的行</td>
	</tr>
    <tr>
        <td><b>renge checked for each record(index map:#)</b>:MySQL没有发现好的可以使用的索引，但发现如果来自前面的表的列值已知，可能部分索引可以使用。</td>
	</tr>
    <tr>
        <td><b>Using filesort</b>:MySQL需要额外的一次传递，以找出如何按排序顺序检索行。</td>
	</tr>
    <tr>
        <td><b>Using index</b>:从只使用索引树中的信息而不需要进一步搜索读取实际的行来检索表中列的信息。</td>
	</tr>
    <tr>
        <td><b>Using temporary</b>:为了解决查询，MySQL需要创建一个临时表来容纳结果。</td>
	</tr>
    <tr>
        <td><b>Using where</b>:WHERE子句用于限制哪一行匹配下一个表或发送到客户。</td>
	</tr>
    <tr>
        <td><b>Using sort_union(...),Using union(...),Using intersect(...)</b>:这些函数说明如何为index_merge联接类型合并索引扫描。</td>
	</tr>
    <tr>
        <td><b>Using index for group-by</b>:类似于访问表的Using index方式，Using index for group-by 表示MySQL发现了一个索引，可以用来查询GROUP BY后DISTINCT查询的所有列，而不要额外搜索硬盘访问实际的表</td>
	</tr>



</table>

# 参考资料 #

[https://www.cnblogs.com/yycc/p/7338894.html](https://www.cnblogs.com/yycc/p/7338894.html)

[http://blog.chinaunix.net/uid-540802-id-3419311.html](http://blog.chinaunix.net/uid-540802-id-3419311.html)

[https://www.cnblogs.com/nirao/p/9721964.html](https://www.cnblogs.com/nirao/p/9721964.html)

