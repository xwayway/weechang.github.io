---
title: 记一次SQL优化
date: 2018-09-27 14:31:52
categories: 日常记录
tags: [日常记录, MySql, Sql]
---

>最近收到项目的慢查询报警，刚好手上空下来了，就准备做一些优化。特开此贴记录一下此次的优化详情。

# 背景介绍

由于项目保密原因，因此很多字段采取保密写法，望各位看官见谅。

其实呢，整个表的数据量其实也不大，也就170W+。字段大概在65个左右，除了几个单号，几个内容这种稍大一点的字段外，其他的基本都是状态和关联id的字段。所以行大小也不大。

但是，上个月的时候，产品提出一个需要根据单号模糊查询的功能，就是这个功能，导致了慢查询。优化前，根据现有逻辑以及模糊查询的需求，大概的sql是这样的

<pre>

SELECT
	id,
	work_order_num AS workOrderNum,
	content,
	OTHERS_COLUMNS...
FROM
	work_order
WHERE
	work_order_num LIKE '%201808221644820%'
AND sender_id = 11768
AND status IN (2, 1, 0)
AND record_status IN (4, 2)
AND (locked = 0 OR locked = 2)
AND yn = 1
ORDER BY
	create_time DESC
LIMIT 0, 10

</pre>

# 数据分析

其中，上述Sql中，条件中的字段  work_order_num、sender_id 都是建了索引的，以下是慢查询统计情况

|   |   |
| ------ | ------ |
| total_count | 19 |
| total_time | 133.23 |
| avg_time | 7.01 |
| max_time | 7.6426 |
| avg_return_rows |	0.79 |
| max_return_rows |	1 |
| avg_examined_rows | 1695975.47 |
| max_examined_rows | 1734740 |

可以看见的是，慢查询基本都是全表扫描。为什么会这样呢，我具体执行了一下Sql发现，以上条件根本就查不出数据啊。然后修改条件发现，是工单号有问题。该工单号不存在，，，不存在啊。所以导致了全表扫描。

# 调优进行时

首先想的是去掉like 特别是like的左匹配，试了一下，去掉左匹配，速度能减少到50%。虽然有效，但是业务不允许这样做。

然后考虑的是用instr函数代替like，测试了一下，能够减少1S的时间，但是对于7S的查询来说，这TM有什么效果。还是不行。

最后想到了一次案例分享中的，先用子查询查询出符合条件的id，再用id作为筛选条件，去获取所有的列。最后修改后的sql如下

<pre>

SELECT
	id,
	work_order_num AS workOrderNum,
	content,
	OTHERS_COLUMNS...
FROM
	work_order
WHERE
	id IN (
		SELECT
			id
		FROM
			ws_work_order
		WHERE
			work_order_num LIKE '%201808221644820%'
		AND sender_id = 11768
		AND STATUS IN (2, 1, 0)
		AND record_status IN (4, 2)
		AND (locked = 0 OR locked = 2)
		AND yn = 1
	)
ORDER BY
	create_time DESC
LIMIT 0, 10

</pre>

运行一下试试，，，果然，时间减少到了 0.03xxS。看着就有点小激动啊。

# 原理剖析

原理呢，暂时忘了，先挖一个坑，等我想起了，再回来填坑。

