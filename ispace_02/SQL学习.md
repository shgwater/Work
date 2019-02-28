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