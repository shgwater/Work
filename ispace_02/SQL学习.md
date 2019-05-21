---
title: SQL学习 
tags: 新建,模板,小书匠
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











