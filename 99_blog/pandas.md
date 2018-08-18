# Pandas Vs SQL
自认为比较熟悉SQL, 刚开始使用Pandas的时候,总觉得好多地方不如SQL处理来的便捷.但是在熟悉Pandas后,发现Pandas往往也有很简单的解决办法, 部分地方恰好是Pandas的优势地方.下面列出一些曾经遇到的场景:

### 滑动窗口/平滑各种曲线
* 应用场景:
	
		当每个月份数据抖动比较大的时候,想把n个月的数据累积到一个月然后平均计算趋势.

* DB解决方案:
	
		没想到特别简单的方法.希望有心人给个提示

* Pandas解决方案:
```python
		cost.sort_values(['city', 'month']).groupby('city')['score'].rolling(3).mean()
```
* 备注

		很显然, Pandas要方便多了
* [Jupyter 示例]( https://github.com/Flyfoxs/ai/blob/master/group/rolling_row_number.ipynb)



## 收入分配情况
* 应用场景:
	
		一个人的支出会运用到多个领域,比如娱乐,教育,投资.现在需要计算每一条数据占当事人的支出百分比.

* DB解决方案:
		
		由于SQL写的会比较长,下面只介绍一下思路
		1)通过分组,计算当前每个人的总收入
		2)把第一步的结果集(子表)和原始表通过人来关联,然后计算百分比

* Pandas解决方案:

		cost['total'] = cost.groupby('person_id')['amount'].transform(sum)
		cost['percentage'] = cost.amount/cost.total


* 备注:

		哪种方案好,就不用多说了
* [Jupyter 示例]( https://github.com/Flyfoxs/ai/blob/master/group/transform.ipynb)

## 异常值的处理
* 应用场景:

		异常值的处理,比如如果是统计小学生的各种生长指标,部分学生身高没有,如果用全局平均值去替代,这是明显不合适的.可选方案,就是取性别和年龄的平均值去替换.	

* DB解决方案:
		
		和上面的收入分配是完全类似的,也就是求每个年龄,性别的平均值,然后Join回去,最后在使用nvl去替换空值

* Pandas解决方案:

		df.groupby('age','gender')['height'].transform(lambda x: x.fillna(x.mean()))

* 备注:
			
		哪种方案好,就不用多说了
* [Jupyter 示例]( https://github.com/Flyfoxs/ai/blob/master/group/transform.ipynb)

## 行列转换
* 应用场景:
		
		行列转换

* DB解决方案:
		
		行转列: 多次select之后union
		列转行: select id, max(case when type='gender' then value else null end) gender
			   from student 
			   group by id

* Pandas解决方案:

		行转列: student.stack()
		列转行: student.stack().unstack()

* 备注:
		
		当有多列时,DB的解决方案会看起来十分累赘
* [Jupyter 示例]( https://github.com/Flyfoxs/ai/blob/master/group/pivot_stack_melt.ipynb)


## 分组排序(row_number)
* 应用场景:

		找出学校每个班级成绩最好的一个人

* DB解决方案:
		
		select * from (SELECT class_id, student_id,, Row_Number() OVER (partition by class_id ORDER BY score desc) rank FROM student_score ) where rank=1

* Pandas解决方案:

		student_score.sort_value(['class_id', 'score'], False).groupby('class_id').nth(0)

* 备注:
		
		2种方案差不多, pandas稍稍好一点点.
		
* [Jupyter 示例]( https://github.com/Flyfoxs/ai/blob/master/group/rolling_row_number.ipynb )


## 删除多余记录
* 应用场景:

		比如你的数据库里面有一个学生成绩的多条历史数据,你只想得到最后的一条数据.

* DB解决方案:
	
	```	
	select * from student_score where id in (
		select max(id) from student_score group by student_id
	)	
	```	

* Pandas解决方案:
	
		student_score.drop_duplicate(subset='student_id',keep='last')

* 备注:
	
		2种方案差别不大
* [Jupyter 示例]( https://github.com/Flyfoxs/ai/blob/master/other/duplicate.ipynb)



## 环比
* 应用场景:

* DB解决方案:

```sql
		select c2.city, c2.month,  (c2.score - c1.score)/c1.score
		from cost c1, cost c2 where 
			c1.month = c2.month-1
			and c1.city = c2.city
```
* Pandas解决方案:
```	python
		cost['previous'] = cost.sort_values(['city', 'month']).groupby('city')['score'].shift(1)
		cost['percentage'] = (cost.score -  cost.previous)/cost.previous
```
* 备注:
	
		2种Solution差别不大,但是感觉熟悉后Pandas要更直接些
* [Jupyter 示例]( https://github.com/Flyfoxs/ai/blob/master/other/chain_relative_ratio_shift.ipynb )
		
		
		
## 行级别的汇总	
* 应用场景:

		比如一个城市的PM2.5每分钟一次采样, 每天24*60条记录保存为一行. 现在的应用如果是要计算每天的PM2.5最高的值是多少, 什么时候是最高值

* DB解决方案:

		最大值:select greatest(col1, col2, col3, ***) , day from records
		
		最大值对应的时间:没想到好办法

* Pandas解决方案:

		最大值:df['max'] = df.apply(lambda row: max(row), axis=1)
	
		最大值对应的时间: df['min_sn'] = df.apply(lambda row: row.idxmax() , axis=1)

* 备注:
	
		很显然Pandas功能上要合适些,并且当Column数量比较多时, Pandas的优势更明显
		
* [Jupyter 示例](https://github.com/Flyfoxs/ai/blob/master/other/axis_daterange.ipynb)
		
	

# 结论

不同的技术应用于不同的场景,这种对比一定程度是不公平的.比如SQL最擅长的Join操作,这里就没有涉及.这样对比的目的,只是为了更好的理解SQL 和 Pandas不同的特点
