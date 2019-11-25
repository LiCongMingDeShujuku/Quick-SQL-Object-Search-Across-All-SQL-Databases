![CLEVER DATA GIT REPO](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/0-clever-data-github.png "李聪明的数据库")

# 在所有数据库中搜索所有快速对象
#### Quick SQL Object Search Across All SQL Databases
**发布-日期: 2017年10月26日 (评论)**

## Contents

- [中文](#中文)
- [English](#English)
- [SQL Logic](#Logic)
- [Build Info](#Build-Info)
- [Author](#Author)
- [License](#License) 


## 中文
想在所有数据库中搜索sysobjects表？下面的逻辑（logic）会帮助你。当然，写一个历遍所有数据库的查询并不难，小菜一碟，但如果你需要的话，下面的逻辑（logic）还是有些东西可以为你所用，又何尝不是一件好事。可以帮你减少一些点击次数。
这个逻辑（logic）会在#logic_table中构建一个查询列表，然后将结果运行到#results表中。
在这个例子中，我寻找了一些浮动的旧对象，然后连结一个查询来撤消它们的权限。 如下所示，它们分别是RTblDMProps，mswebtask和dtproperties对象。


## English
Want to search the sysobjects tables across all databases? This little bit of logic should help. Sure; it’s not hard to write a query across all databases… pffft… kid stuff, but it’s nice to have something to get you going if you need it. Save you a few keystrokes. Ok; so what this does is builds a list of queries in a #logic_table, and then runs the results into a #results table.
In this example I’m looking for a few older objects floating around so I can concatenate a query to revoke permissions from them. As you can see below; they are the RTblDMProps, mswebtask, and dtproperties objects.


![#](images/Quick-SQL-Object-Search-Across-All-SQL-Databases-a.png?raw=true "#")


如你所见， 我已经确认对象存在，然后就可以编写一些逻辑（logic）来撤销权限。我在这个方框上可能已经得到了几百个数据库，很高兴可以得到一个整齐的小列表，包含临时表中的所有代码，这样我就可以根据需要调整或进行扩展。

As you can see; I’ve already confirmed the objects exist so I can write some logic to revoke the permissions. I’ve got maybe a couple hundred databases on this box so it’s nice to get a tidy little list with all the code in the temp tables so I can tweak, or expand on it if needed.



---
## Logic
```SQL
use master;
set nocount on
 
 
--basic search across all databases be sure to replace MyObject with the object you're searching for.
-- 历遍所有数据库的基本搜索，确保要将MyObject替换为你要查找的对象。
select
'use [' + upper([name]) + ']; select upper(db_name()), [name], [xtype] from [' + name + ']..sysobjects where [name] in (''MyObject'');'
from sys.databases
where [name] not in ('tempdb')
 --the remaining logic will produce a script on removing rights from the desired object(s).
-- 剩余的逻辑将生成一个从所需对象中删除权限的脚本。
-- 
if object_id('tempdb..#logic_table') is not null drop table #logic_table
create table #logic_table   ([logic] varchar(max))
insert into #logic_table    ([logic])
select 'use [' + upper([name]) + ']; select upper(db_name()), [name], [xtype] from [' + name + ']..sysobjects where [name] in (''MyObject1'', ''MyObject2'', ''MyObject3'');'
from  master.sys.databases where database_id <> 2
 
if object_id('tempdb..#results') is not null drop table #results
create table    #results ([lineid] int identity(1,1), [database] varchar(255), [name] varchar(255), [xtype] varchar(2))
declare @sql    varchar(max) 
set @sql    = ''
select  @sql    = @sql + [logic] from #logic_table
insert into #results ([database], [name], [xtype])
exec (@sql) 
select * from #results where [database] is not null
 
-- build revoke statement
-- 构建撤销声明
select 'use [' + [database] + ']; revoke select, insert, update, delete, references on ' + [name] + ' to public;'
from #results


```



[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

- **李聪明的数据库 Lee's Clever Data**
- **Mike的数据库宝典 Mikes Database Collection**
- **李聪明的数据库** "Lee Songming"

[![Gist](https://img.shields.io/badge/Gist-李聪明的数据库-<COLOR>.svg)](https://gist.github.com/congmingshuju)
[![Twitter](https://img.shields.io/badge/Twitter-mike的数据库宝典-<COLOR>.svg)](https://twitter.com/mikesdatawork?lang=en)
[![Wordpress](https://img.shields.io/badge/Wordpress-mike的数据库宝典-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

---
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Lee Songming](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/1-clever-data-github.png "李聪明的数据库")

