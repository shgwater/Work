---
title: SQL学习 
tags: mysql
grammar_cjkRuby: true
---
[toc!?direction=lr]

# sqlserver 动态计算
2019年02月28日 18时56分22秒


``` sql

--模拟数据
IF OBJECT_ID('tempdb..#t')>0 DROP TABLE #t
SELECT * INTO #t
FROM (
SELECT '1' id,2030 g,265 h, 830 k,'g*h+h*k' gs,0 tt
UNION ALL
SELECT '2' id,2030 g,0 h, 0 k,'g*4' gs,0 tt
UNION ALL
SELECT '3' id,2030 g,265 h, 0 k,'(g+h)*2' gs,0 tt
UNION ALL
SELECT '4' id,2030 g,265 h, 0 k,'(g+h)*2' gs,0 tt)t
--原始数据
SELECT * FROM #t



--按公式类别分类
IF OBJECT_ID('tempdb..#tt')>0 DROP TABLE #tt
SELECT ROW_NUMBER()OVER(ORDER BY gs)rowid 
,gs 
INTO #tt
FROM #t
GROUP BY gs


--变量
DECLARE @i int,@n INT
DECLARE @gs VARCHAR(50)
DECLARE @sql VARCHAR(MAX)
--按公式类别遍历
SELECT @i=MIN(rowid),@n=MAX(rowid) FROM #tt
WHILE(@i<=@n)
BEGIN
SELECT @gs=gs FROM #tt WHERE rowid=@i
SET @sql='update #t set tt='+@gs+' where gs='''+@gs+''''--生成脚本
EXEC(@sql)--执行脚本，可以print看效果
SET @i=@i+1
END
--处理后效果
SELECT * FROM #t

drop table #t
```
2019年03月15日 19时09分48秒
实际的动态计算
需要注意一下几点，@gs 变量的长度，一定要大于公式字段最长的长度。
第二个需要注意的点是，存放结果的字段一定要是够大的浮点型，如果是使用数字新建的话，可用 100000.0000 代替。
``` sql
	
	--模拟数据 
IF OBJECT_ID('tempdb..#t')>0 DROP TABLE #t 

SELECT 
a.store_no
,a.contract_no 
,a.contract_area
,a.real_area
,a.product_no  -- ' COMMENT '套餐编号[inco1]',
-- ,a.id -- ’设计空间实例id',
,a.design_case_no --   '设计实例编码',
,a.case_space_name --   '空间实例名',
,a.case_space_no --   '空间实例编码',
,a.space_no --   '空间编码',
,a.no -- '相同空间的编号(比如 WS1,WS2,WS3 后面的数字)',
,a.design_case_id -- '设计实例id',
,a.sort_val -- '排序',
,a.area BL_SCMJ-- '实测面积',
,a.girth BL_KJZC-- '周长',
,a.height BL_KJCG-- '层高',
,a.is_balcony -- '是否关联阳台:0:无,1:有',
,a.remain_area -- ’可用墙面积',
,a.state -- '1:已完成,2.未完成',
,a.relation_space_no --   '关联空间实例编码 ',
,a.added -- '1:户型外添加的空间 2:户型内的空间 ',
-- ,b.store_no                 
-- ,b.package_no               
-- ,b.cp_combo_name            
-- ,b.space_no                 
,b.work_item_no             
,b.work_item_name           
,b.dimension_no             
-- ,b.stand_choose             
-- ,b.dimension_name           
,b.dimension_val   
,c.bl_sgl bl_sgl_1
,case when c.bl_sgl = 'SCCG' then a.height*c.sgl_num when c.bl_sgl='SCZC' then a.girth*c.sgl_num when c.bl_sgl='SCQM' then a.remain_area*c.sgl_num when c.bl_sgl='SCMJ' then a.area*c.sgl_num else c.bl_sgl end BL_SGL
,c.sgl_num
,c.job_place
,c.job_place_no 
,d.goods_no
,d.goods_name
,d.category_no
,d.length JC_SKU_length
,d.width JC_SKU_width
,d.height goods_height
,d.base_unit
,d.normal_num -- 标配数量
,d.attrition_rate JC_SHL-- 损耗率
,d.function_content1 gs -- 施工量和用量转化公式    
, 10000000000.0000 tt
into #t
FROM
	Fact_Design_Case_Space a
LEFT JOIN dim_03_stand_package b ON a.store_no = b.store_no  -- 关联标准套餐表，出来标准套餐所需要的施工项
AND a.product_no = b.package_no
AND a.space_no = b.space_no
LEFT JOIN (select * from dim_03_stand_work_item where is_suit = 1) c -- 关联标准施工量，得到每个施工项的施工量。
ON b.work_item_no = c.work_item_no
LEFT JOIN Fact_Design_House_Space_Goods d -- 根据城市-施工项-选材维度确定商品
on a.store_no = d.store_no and b.work_item_no = d.work_item_no  and b.dimension_no  = d.dimension_no  
WHERE
	a.contract_no = 'DD20180602001060'

--原始数据 
SELECT * FROM #t 
 
 
 
 
--按公式类别分类 
IF OBJECT_ID('tempdb..#tt')>0 DROP TABLE #tt 
SELECT ROW_NUMBER()OVER(ORDER BY gs)rowid  
,gs  
INTO #tt 
FROM #t 
where gs is not null 
GROUP BY gs 
 select * from #tt
 
--变量 
DECLARE @i int,@n INT 
DECLARE @gs VARCHAR(100) 
DECLARE @sql VARCHAR(MAX) 
--按公式类别遍历 
SELECT @i=MIN(rowid),@n=MAX(rowid) FROM #tt 
WHILE(@i<=@n) 
BEGIN 
SELECT @gs=gs FROM #tt WHERE rowid=@i 
print @i
SET @sql='update #t set tt='+@gs+' where gs='''+@gs+''''--生成脚本 

print @sql

EXEC(@sql)--执行脚本，可以print看效果 
/*
*/
SET @i=@i+1 
END 

--处理后效果 
SELECT * FROM #t 
 
drop table #t
drop table #tt
-- 向上取整数 CEILING


select a.*,b.* from 
(select a.*,CEILINg(a.tt) as use_num from #t a) a
left join Fact_Sale_price b
on a.store_no=b.store_no and a.goods_no = b.goods_no

```
# sqlserver decimal转为varchar
因为 decimal 格式的有小数位，cast 函数只能将他变成 10.0000 的形式，因此还需要替换掉的后面的。
``` sql
select replace(cast(101.00000000 as varchar(100)),'.00000000','') store_no 
```
# mysql 组内分组排序
``` sql



SELECT
	a.*, count(1) AS rank
FROM
	(select 1 as id,'aaa' as name ,1 as category_id union all
select 2 as id,'bbb' as name ,2 as category_id union all
select 3 as id,'ccc' as name ,1 as category_id union all
select 4 as id,'ddd' as name ,2 as category_id union all
select 5 as id,'eee' as name ,1 as category_id 
) a
LEFT JOIN (select 1 as id,'aaa' as name ,1 as category_id union all
select 2 as id,'bbb' as name ,2 as category_id union all
select 3 as id,'ccc' as name ,1 as category_id union all
select 4 as id,'ddd' as name ,2 as category_id union all
select 5 as id,'eee' as name ,1 as category_id 
) b ON a.category_id = b.category_id
AND a.id <= b.id
GROUP BY
	a.category_id,
	a.id
ORDER BY
	a.category_id,
	a.id DESC


SELECT
	*
FROM
	(select 1 as id,'aaa' as name ,1 as category_id union all
select 2 as id,'bbb' as name ,2 as category_id union all
select 3 as id,'ccc' as name ,1 as category_id union all
select 4 as id,'ddd' as name ,2 as category_id union all
select 5 as id,'eee' as name ,1 as category_id 
) a
LEFT JOIN (select 1 as id,'aaa' as name ,1 as category_id union all
select 2 as id,'bbb' as name ,2 as category_id union all
select 3 as id,'ccc' as name ,1 as category_id union all
select 4 as id,'ddd' as name ,2 as category_id union all
select 5 as id,'eee' as name ,1 as category_id 
) b ON a.category_id = b.category_id
AND a.id >= b.id
where a.category_id = 1
GROUP BY
	a.category_id,
	a.id

``` 
# mysql分组后取组内某列最小（大）的行
此问题是由于工作中之前在sqlserver中是使用的 分组后排序，然后取rank=1的，从而得到分组后的某列极值的。但是mysql中分组后排序的机制太过复杂（关联后count数量），特别是大数量的时候会极度的影响效率，再加上我最核心的需求实际上可以总结为**每个用户的创建时间最早的一行数据**，因此在之前的路走不通的情况下寻求其他的办法。
``` sql
-- 返回每个用户的创建时间最早的订单编号
-- step 1 首先根据时间排序

select * from fact_decorate_order order by concat(DATE(create_date),' ',TIME(create_time));

-- step 2 根据用户编号分组
-- 注意，此时mysql5.7 以上的话需要加limit，要不然会有问题。

select a.*,'1' rank from (
select * from fact_decorate_order order by concat(DATE(create_date),' ',TIME(create_time))
-- limit 100
) a
group by user_id 
```
## 思路更新

最近在有需求取最早的订单编号，想了一下可能不需要现在这么复杂，只需要找出最大或者最小的ID，然后使用原表来关联（使用ID  in 也可以，但是数据量大的时候join效率更高）此id，再加个条件不为空，既能得到想要的最大（小）的值那行的信息。
``` sql

select user_id,orders_no from fact_decorate_order fdo
left join (
	select min(id) id from fact_decorate_order
	group by user_id 
) fdoId on fdo.id=fdoId.id
where fdoId.id is not null 
```

# mysql使用 in 子查询效率低下
mysql直接使用in (select user_id from table) 时效率会超级超级超级慢，具体原理暂且不论，可以在外面再套一层，这样的话查询速度就会变正常。
``` sql
select * 
from table_A
where user_id in (
	select user_id
	from (
		select user_id 
		from table_B
	) temp
)
```
# mysql 大量数据关联查询
大量数据（百万）表间关联的时候，查询速度会特别的慢，此时需要在关联字段上创建索引，会极大的提高查询效率。
但是有多个大表关联的时候，即便建立了索引查询效率依然很低，此时需要了解mysql中驱动表的概念。最外层的表，也就是left join 最左边的表就是驱动表，查询的效率与驱动表的行数有关，行数越低查询效率越高。
因此遇到那种百万级别的主表取进行关联的时候，可以分组进行关联。如下。

``` sql
		WHILE i < num DO

				insert into Fact_Decorate_Order_Std
					SELECT
					*
				FROM	(select * from Fact_Decorate_Order where id >= i and id < i+100000) a 
				LEFT JOIN T71 b ON a.user_id = b.user_id
				LEFT JOIN T72 c ON a.orders_no = c.fk_docr_orders_no 
				left join  T6 e on a.orders_no = e.orders_no -- 20190304 订单明细标准表增加调整的时间，之前只是用户穿刺表处理了时间
				left join dim_01_contract_tag f
				on a.orders_no = f.orders_no ;

        SET i = i + 100000 ;

    END WHILE ;
```
按照id每次抽一万条。

# 同结构多关联查询问题

  我们的订单数据结构目前是以订单为核心的，所有阶段的时间都记录在一条信息上，这样的话如果我想要取用某天实际的各阶段的数据的时候，就需要每个阶段都建立一个查询，分别查出每个阶段需要使用的数据，然后关联在一起。
  	实际使用过程中每次都需要关联表格效率太低，因此想做一张类似已经查询完的结果表的表放在那里，这样的话使用起来会比较方便。
	在实际操作中遇到了一点疑问，就是每个阶段的维度不一定是相同的，我们需要取全连接的所有数据，并且把空的维度进行替换，两张表的时候没有任何问题，但是随着阶段的增多，当增加到三张表的时候就会出现问题，如果 a full join b full join c ，那么c的关联条件怎么做?
	现阶段我能想到的解决办法就是  a先与b关联做出一个子查询然后和c关联。

- 验证sql如下（sqlserver环境）
``` sql
select 'A' as apt
			,'甲' as name 
			,1 as num_1
into temp_1

insert into temp_1 values('B','乙',2);

select 'B' as apt
			,'乙' as name 
			,3 as num_2
into temp_2

insert into temp_2 values('C','丙',4);

select 'C' as apt
			,'丁' as name 
			,5 as num_3
into temp_3

insert into temp_3 values('D','戊',6);
insert into temp_3 values('A','甲',7);
insert into temp_3 values('C','丙',8);
-- 不能正常合并的方法
select *
from temp_1 full join temp_2 on temp_1.apt=temp_2.apt and temp_1.name=temp_2.name
full join temp_3 on temp_1.apt=temp_3.apt and temp_1.name=temp_3.name 

-- 现有方法
select case when temp_1.apt is null then temp_3.apt else temp_1.apt end apt
					,case when temp_1.name is null then temp_3.name else temp_1.name end name 
					,temp_1.num_1
					,temp_1.num_2 
					,temp_3.num_3
from 
(
		select case when temp_1.apt is null then temp_2.apt else temp_1.apt end apt
					,case when temp_1.name is null then temp_2.name else temp_1.name end name 
					,temp_1.num_1
					,temp_2.num_2
		from temp_1 full join temp_2 on temp_1.apt=temp_2.apt and temp_1.name=temp_2.name
) temp_1
full join temp_3 on temp_1.apt=temp_3.apt and temp_1.name=temp_3.name 



drop table if EXISTS temp_1;
drop table if EXISTS temp_2;
drop table if EXISTS temp_3;
``` 
# 相关子查询和非相关子查询（可实现累加）


子查询：嵌套在其它查询中的查询语句。（又称为内部查询）

主查询：包含其它子查询的查询称为主查询。（又称外部查询）

 

子查询分为两类：

相关子查询
非相关子查询
在主查询中，每查询一条记录，需要重新做一次子查询，这种称为相关子查询。

在主查询中，子查询只需要执行一次，子查询结果不再变化，供主查询使用，这种查询方式称为非相关子查询。


 ``` sql
 
  -- 相关子查询
SELECT sname
FROM student
WHERE sex = ‘女’ AND
EXISTS ( SELECT  *         //相关子查询
FROM sc
WHERE sc.sno = student.sno AND
sc.cno LIKE ‘ee%’);
 
 
 -- 非相关子查询  
 
 SELECT sname
FROM student
WHERE sex = ‘女’ AND
sno IN ( SELECT DISTINCT sno       //不相关子查询
FROM sc
WHERE cno LIKE ‘ee%’);


-- 实际用到的子查询 实现了累加的功能  实际是一个相关子查询
-- 在mysql中实现了累加
SELECT
	(
		SELECT
			sum(

				IF (
					payment_category IN (6, 7, 11, 13),
					payment_amount *- 1,
					payment_amount
				)
			)
		FROM
			decorate_order_pay b
		WHERE
			b.orders_no = a.orders_no
		AND b.id * 1 <= a.id * 1
	) sumnum,
	a.*
FROM
	decorate_order_pay a
WHERE
	orders_no = 'DD20181202000485'
 ```
sqlserver中实现分组累加要方便许多。有函数可以直接使用。

``` sql
select 
sum(payment_amount) over(partition by orders_no order by create_time)
 from fact_decorate_order_pay 
where orders_no ='DD20181202000485'

```



# sqlserver中游标的使用

现在有一个链接地址处理的问题，有多个链接中的特定字符需要替换成多个城市的缩写。
考虑使用嵌套循环的方式处理，但是sql中没有类似python中的列表，没办法临时存储，经过查询资料发现sql解决这个问题的办法是创建索引，使用索引作为临时存储的容器。

``` sql

DECLARE @i int
DECLARE @area VARCHAR (10)
set @i = 0
SET @area = 'area' -- 创建游标
DECLARE area_cursor CURSOR FOR (		select 'bj' as area union		select 'sh' as area union		select 'sz' as area union		select 'szh' as area union		select 'cd' as area union		select 'tj' as area union		select 'ty' as area union		select 'gz' as area union		select 'gy' as area union		select 'wh' as area union		select 'jn' as area union		select 'xa' as area union		select 'lf' as area union		select 'zz' as area union		select 'nj' as area union		select 'nc' as area 
) --打开游标--
OPEN area_cursor 
--开始循环游标变量--
FETCH NEXT FROM	area_cursor INTO @area


-- 创建临时表
drop table if exists #stand_url_temp

create table #stand_url_temp(
entry_url nvarchar(256),
source_no nvarchar(10)
)

WHILE @@FETCH_STATUS = 0 --返回被 FETCH语句执行的最后游标的状态--
	BEGIN
		insert into #stand_url_temp
			select REPLACE(entry_url, '*',@area) ,source_no from (
					select 'http://*.ikongjian.com/' as entry_url,'790001' as source_no union
					select 'http://*.ikongjian.com/reservation/index%' as entry_url,'790001' as source_no union
					select 'http://m.ikongjian.com/*/reservation/index%' as entry_url,'790002' as source_no union
					select 'http://m.ikongjian.com/' as entry_url,'790002' as source_no union
					select 'http://m.ikongjian.com/*/' as entry_url,'790002' as source_no union
					select 'https://m.ikongjian.com/*/changeCity' as entry_url,'790002' as source_no union
					select 'https://m.ikongjian.com/*/liveOffice/%' as entry_url,'790002' as source_no union
					select 'http://*.ikongjian.com/liveOffice/%' as entry_url,'790001' as source_no union
					select 'http://*.ikongjian.com/construction/' as entry_url,'790001' as source_no union
					select 'http://*.ikongjian.com/hotHouse/index' as entry_url,'790001' as source_no union
					select 'http://*.ikongjian.com/cooperation/tequan' as entry_url,'790001' as source_no union
					select 'http://*.ikongjian.com/kujiale/p/index' as entry_url,'790001' as source_no union
					select 'http://*.ikongjian.com/news/%' as entry_url,'790001' as source_no union
					select 'http://*.ikongjian.com/case/%' as entry_url,'310101' as source_no union
					select 'http://www.ikongjian.com/zixun/%' as entry_url,'310101' as source_no union
					select 'http://www.ikongjian.com/wen/%' as entry_url,'310101' as source_no union
					select 'http://www.ikongjian.com/tu/%' as entry_url,'310101' as source_no union
					select 'https://m.ikongjian.com/zixun/%' as entry_url,'310102' as source_no union
					select 'https://m.ikongjian.com/wen/%' as entry_url,'310102' as source_no union
					select 'https://m.ikongjian.com/tu/%' as entry_url,'310102' as source_no union
					select 'https://m.ikongjian.com/*/case/%' as entry_url,'310102' as source_no union
					select 'http://m.ikongjian.com/*/ikj/%' as entry_url,'790003' as source_no union
					select 'https://*.ikongjian.com/ikj/%' as entry_url,'790003' as source_no union
					select 'https://m.ikongjian.com/*/activitys/%' as entry_url,'790003' as source_no union
					select 'https://*.ikongjian.com/ activitys/%' as entry_url,'790003' as source_no 
		) stand_url
			where entry_url like '%*%'

	FETCH NEXT	FROM		area_cursor INTO @area --转到下一个游标，没有会死循环
	END 

CLOSE area_cursor --关闭游标
DEALLOCATE area_cursor --释放游标

insert into  #stand_url_temp
select entry_url,source_no
from (
	select 'http://*.ikongjian.com/' as entry_url,'790001' as source_no union
	select 'http://*.ikongjian.com/reservation/index%' as entry_url,'790001' as source_no union
	select 'http://m.ikongjian.com/*/reservation/index%' as entry_url,'790002' as source_no union
	select 'http://m.ikongjian.com/' as entry_url,'790002' as source_no union
	select 'http://m.ikongjian.com/*/' as entry_url,'790002' as source_no union
	select 'https://m.ikongjian.com/*/changeCity' as entry_url,'790002' as source_no union
	select 'https://m.ikongjian.com/*/liveOffice/%' as entry_url,'790002' as source_no union
	select 'http://*.ikongjian.com/liveOffice/%' as entry_url,'790001' as source_no union
	select 'http://*.ikongjian.com/construction/' as entry_url,'790001' as source_no union
	select 'http://*.ikongjian.com/hotHouse/index' as entry_url,'790001' as source_no union
	select 'http://*.ikongjian.com/cooperation/tequan' as entry_url,'790001' as source_no union
	select 'http://*.ikongjian.com/kujiale/p/index' as entry_url,'790001' as source_no union
	select 'http://*.ikongjian.com/news/%' as entry_url,'790001' as source_no union
	select 'http://*.ikongjian.com/case/%' as entry_url,'310101' as source_no union
	select 'http://www.ikongjian.com/zixun/%' as entry_url,'310101' as source_no union
	select 'http://www.ikongjian.com/wen/%' as entry_url,'310101' as source_no union
	select 'http://www.ikongjian.com/tu/%' as entry_url,'310101' as source_no union
	select 'https://m.ikongjian.com/zixun/%' as entry_url,'310102' as source_no union
	select 'https://m.ikongjian.com/wen/%' as entry_url,'310102' as source_no union
	select 'https://m.ikongjian.com/tu/%' as entry_url,'310102' as source_no union
	select 'https://m.ikongjian.com/*/case/%' as entry_url,'310102' as source_no union
	select 'http://m.ikongjian.com/*/ikj/%' as entry_url,'790003' as source_no union
	select 'https://*.ikongjian.com/ikj/%' as entry_url,'790003' as source_no union
	select 'https://m.ikongjian.com/*/activitys/%' as entry_url,'790003' as source_no union
	select 'https://*.ikongjian.com/ activitys/%' as entry_url,'790003' as source_no 
) stand_url
where entry_url not like '%*%'


```
# sqlserver中取周初周末的日期

``` sql

-- 首先将当前日期减一,因为sqlserver默认从周日开始一周，如果我们需要周一开始的话需要减一
-- 然后计算从计算机0开始到现在有多少周
-- 再然后从零时间加上面取出来的周。即为周初
-- 再然后6代表第七天的时间加上面取出来的周。即为周末
select DATEADD(wk, DATEDIFF(wk,0,DATEADD(dd, -1, '2019-11-08') ), 0)

-- 假如你要计算这个季度的第一天，这个例子告诉你该如何做。
　SELECT DATEADD(qq, DATEDIFF(qq,0,getdate()), 0)
```

# sqlserver 查看各表大小

``` sql
drop table if EXISTS  #Data
drop table if EXISTS  #DataNew

create table #Data(name varchar(100),row varchar(100),reserved varchar(100),data varchar(100),index_size varchar(100),unused varchar(100)) 
 
declare @name varchar(100) 
declare cur cursor  for 
    select name from sysobjects where xtype='u' order by name 
open cur 
fetch next from cur into @name 
while @@fetch_status=0 
begin 
    insert into #data 
    exec sp_spaceused   @name 
    print @name 
 
    fetch next from cur into @name 
end 
close cur 
deallocate cur 
 
create table #DataNew(name varchar(100),row int,reserved int,data int,index_size int,unused int) 
 
insert into #dataNew 
select name,convert(int,row) as row,convert(int,replace(reserved,'KB','')) as reserved,convert(int,replace(data,'KB','')) as data, 
convert(int,replace(index_size,'KB','')) as index_size,convert(int,replace(unused,'KB','')) as unused from #data  
 


select * from #dataNew order by data desc

drop table if EXISTS  #Data
drop table if EXISTS  #DataNew

```

# 数据库的思路

关于增加上传日期的问题，可以从数据库表本身的可用功能的思路去想，不必修改现有的上传逻辑等。
给需要上传时间的表增加一列时间，给该列一个默认值为当前时间，当上传的时候，会自动添加上传时间。
sqlserver下语句如下。

 ``` sql
 alter table fact_eas_adjustment_upload_temp add AddDate datetime default GETDATE()
``` 
# mysql 实现累加
``` sql
		select 
			(select sum(if(payment_category in (6,7,11,13),payment_amount*-1,payment_amount)) from www.decorate_order_pay b where b.orders_no = a.orders_no and b.id*1<=a.id*1) CUSUM
			-- 2019年8月8日 shg 以下为 累加字段的标记字段，用作后续判断
			,if((select sum(if(payment_category in (6,7,11,13),payment_amount*-1,payment_amount)) from www.decorate_order_pay b where b.orders_no = a.orders_no and b.id*1<=a.id*1)>=500,1,0) CUSUM_sign
			-- 2019年11月19日 shg  之前的判断逻辑有误，会取到首次交钱不满500的作为首次退订至不满五百的时间。
			-- 新增交定数：期初金额=(回款明细当前余额 减 回款明细当前金额 ) < 500   & 回款明细当前余额 >=500
			-- 新减交定数：期初金额=(回款明细当前余额 减 回款明细当前金额 ) >= 500   & 回款明细当前余额 >500
			,case when (select sum(if(payment_category in (6,7,11,13),payment_amount*-1,payment_amount)) from www.decorate_order_pay b where b.orders_no = a.orders_no and b.id*1<=a.id*1) - if(payment_category in (6,7,11,13),payment_amount*-1,payment_amount) < 500 and
			 (select sum(if(payment_category in (6,7,11,13),payment_amount*-1,payment_amount)) from www.decorate_order_pay b where b.orders_no = a.orders_no and b.id*1<=a.id*1) >= 500 then 1 else 0 end  as new_payment_state
			,case when (select sum(if(payment_category in (6,7,11,13),payment_amount*-1,payment_amount)) from www.decorate_order_pay b where b.orders_no = a.orders_no and b.id*1<=a.id*1) - if(payment_category in (6,7,11,13),payment_amount*-1,payment_amount) >= 500 and
			 (select sum(if(payment_category in (6,7,11,13),payment_amount*-1,payment_amount)) from www.decorate_order_pay b where b.orders_no = a.orders_no and b.id*1<=a.id*1) < 500 then 1 else 0 end  as new_un_payment_state
			,a.*
		from www.decorate_order_pay  a
		-- where orders_no = 'DD20180518000839'
		order by orders_no,create_time 

```
# sql关联的附表的过滤

之前使用sql关联附表的时候，如果只需要附表中的部分信息的话，我会使用子查询先将部分信息筛选出来，然后用该子查询的查询结果再与主表关联。这样的话会对查询效率有些影响。
今天看到了其他人写的sql，她的关联条件中增加了过滤，即下代码框中的
**left join dim_area b on a.area_code = b.area_code and b.city = '西安'**
 没想到可以直接放到关联字段之后，好用。之前竟然都不知道。

``` sql
select a.orders_no,b.*  from fact_decorate_order_std  a
left join dim_area b on a.area_code = b.area_code and b.city = '西安'
where user_id = '1043087'

```
# sqlserver 临时表清理

使用存储过程方式来实现临时表的清理。

```
ALTER PROCEDURE [dbo].[PF_ETL_Clear_TempTable]
AS
BEGIN	

	DECLARE @name VARCHAR(100)
	DECLARE @sql VARCHAR(MAX) 

	DECLARE TableNameCursor CURSOR FOR 
	SELECT NAME FROM SYSOBJECTS WHERE XTYPE='U' and name like 'BI%' ORDER BY NAME
	
	-- 打开游标 
	OPEN TableNameCursor
	fetch next from TableNameCursor into @name 
	WHILE @@fetch_status = 0 
	BEGIN
	
		set @sql = 'drop table '+@name
		print @sql
		exec(@sql)
		fetch next from TableNameCursor into @name 

	END

	
	-- 关闭游标
	CLOSE TableNameCursor
	deallocate TableNameCursor 

	-- drop table BI_1_1_G1_120900029263CE
END
```


 