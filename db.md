# 关于数据库的研究

环境：14.04.1-Ubuntu x86_64 GNU/Linux

## MySQL

1. install

    ```bash
    apt-get install mysql-server
    ```
			
    配置文件路径：/etc/mysql/my.cnf

    数据文件路径：/var/lib/mysql

    日志文件路径：/var/log/mysql

			
2. status
    ```bash
     service mysql status
    ```

3. login
    ```bash
     mysql -u root -p
    ```

4. remote login
    ```bash
     mysql -h ip -u root -p
    ```

5. 设置字符集（以utf8为例）：

　　　　-  查看当前的编码：*show variables like 'character%'*;

　　　　-　修改my.cnf，在[client]下添加default-character-set=utf8

　　　　-  在[server]下添加default-character-set=utf8，init_connect='SET NAMES utf8;'   ---好像没有这一项，加了之后 会导致MySQL 启动不了。

　　　　-  重启mysql  *service mysql restart  or   mysqld restart*

6. 设置bind-address，如果是127.0.0.1的话，只能在本机访问MySQL，无法远程访问。
        bind-address = 0.0.0.0

7. 修改mysql最大连接数
        max_connections=1024  设置太大的话 会导致 MySQL 启不来
		
8. 旧数据升级到utf8（旧数据以latin1为例）：

　　　　- 导出旧数据：*mysqldump --default-character-set=latin1 -hlocalhost -uroot -B dbname --tables old_table >old.sql*

　　　　- 转换编码(Linux和UNIX)：*iconv -t utf-8 -f gb2312 -c old.sql > new.sql*
         这里假定原表的数据为gb2312，也可以去掉-f，让iconv自动判断原来的字符集。

　　　  - 导入：修改new.sql，在插入或修改语句前加一句话："SET NAMES utf8;"，并修改所有的gb2312为utf8，保存。

　　　  ```bash　　　
        mysql -h localhost -u root -p dbname < new.sql
        ```
	
        如果报max_allowed_packet的错误，是因为文件太大，mysql默认的这个参数是1M，修改my.cnf中的值即可（需要重启mysql)。

9. 支持utf8的客户端：Mysql-Front,Navicat,PhpMyAdmin，Linux Shell（连接后执行SET NAMES utf8;后就可以读写utf8的数据了。5.4设置完毕后就不用再执行这句话了）

10. 备份和恢复

　　　　备份单个数据库：*mysqldump -uroot -p -B dbname > dbname.sql*

　　　　备份全部数据库：*mysqldump -uroot -p --all-databases > all.sql*

　　　　备份表： *mysqldump -uroot -p -B dbname --table tablename > tablename.sql*

　　　　恢复数据库：*mysql -uroot -p < name.sql*

　　　　恢复表：*mysql -uroot -p dbname < name.sql (必须指定数据库) *


11. 使用：
    查看所有的数据库：
	show databases;
	use smdb;
	show tables;


## MongoDB 2.4.9

1. install
    ```bash
     apt-get install mongodb
    ```	
    The MongoDB instance stores its data files in /var/lib/mongodb 
    and its log files in /var/log/mongodb by default, 
    and runs using the mongodb user account. 
    You can specify alternate log and data file directories in /etc/mongod.conf. 
    See systemLog.path and storage.dbPath for additional information.

    If you change the user that runs the MongoDB process, 
    you must modify the access control rights to the /var/lib/mongodb 
    and /var/log/mongodb directories to give this user access to these directories.
		
2.  restart mongodb
    ```bash
     service mongodb restart
    ```	

3.  manage：

        admin web console  http://ip:28017
	
        user connection port 27017
		
        mongodb shell by  input mongo
		
        and mongod


4.  设置bind_ip

       bind_ip: "0.0.0.0"
	   
5.  使用：

6.  卸载：
    service mongodb stop
	
    apt-get purge mongodb
	
    remove data and log
	
    sudo rm -r /var/log/mongodb
	
    sudo rm -r /var/lib/mongodb
	
7. 概念
   文档型的NoSql数据库
   
   对标 关系型数据库的 概念
   
   db（database）, collection(table), document(record)
   
   mongodb没有固定的schema。存储的document是类似于json的bson文档。比json支持更多的数据类型，如：Date, BinDate
   
   查看数据库的方法：
   mongo
   show dbs;
   use smdb;
   show collections;
   
8. 初级CURD:
  
    - insert：  
      单条：db.smcollection.insertOne({name: ""}), 批量：db.smcollection.insertMany([{name: "a"},{name: "b"}]) 新版本支持 
      也可以对insert() 做循环插入
  
    - find: 
       日常开发中，我们玩查询，玩的最多的是下面这两类：

       1： >, >=, <, <=, !=, =。  对应的mongo封装是 "$gt", "$gte", "$lt", "$lte", "$ne"

       2：And，OR，In，NotIn        mongodb都封装好了 这些操作，对应的是  "$or", "$in"，"$nin"
	 
	如：
	
	 > db.info.find({name: {$in:["moon","ru"]}});
         { "_id" : ObjectId("59ccdbba5363253e0ef0f42d"), "name" : "moon", "sex" : "male" }
         { "_id" : ObjectId("59ccdd17ce94b645f5b101f9"), "name" : "ru", "sex" : "female" }
		
	 > db.info.find({$or: [{name: "ru"}, {name: "moon"}]});
         { "_id" : ObjectId("59ccdbba5363253e0ef0f42d"), "name" : "moon", "sex" : "male" }
         { "_id" : ObjectId("59ccdd17ce94b645f5b101f9"), "name" : "ru", "sex" : "female" }
		 
       3： 特殊的匹配，正则表达式，威力强劲
	  
	  startwith m  endwith u  ，好像 如果后面的匹配到的话 会以后面的为准。
	    
	   > db.info.find({name: /^m/, name: /u$/});
           { "_id" : ObjectId("59ccdd17ce94b645f5b101f9"), "name" : "ru", "sex" : "female" }
	 
       4：$where
        
	   > db.info.find({$where: function(){return this.name=='moon'}});
           { "_id" : ObjectId("59ccdbba5363253e0ef0f42d"), "name" : "moon", "sex" : "male" }	
	 
    - update：
	     整体更新。局部更新，mongodb中提供了两个修改器： $inc 和 $set
		 
	     $inc也就是increase的缩写，每次修改会在原有的基础上自增$inc指定的值，如果“文档”中没有此key，则会创建key
             如：
		 
	     > db.info.update({name: "moon"},{$inc: {age: 30}});
             > db.info.find();
             { "_id" : ObjectId("59ccdbba5363253e0ef0f42d"), "name" : "moon", "age" : 60 }
		
	     $set修改器
		
	     > db.info.update({name: "moon"},{$set: {age: 20}});
             > db.info.find();
             { "_id" : ObjectId("59ccdbba5363253e0ef0f42d"), "name" : "moon", "age" : 20 }
	   
     - upsert：
            如果没有查到，就在数据库里面新增一条，将update的第三个参数设为true即可
		  
            如：
		  
            批量更新：
	      
	    在mongodb中如果匹配多条，默认的情况下只更新第一条，如果批量更新，那么在mongodb中实现是在update的第四个参数中设为true即可

     - remove：
	   remove中如果不带参数将删除所有数据，在mongodb中是一个不可撤回的操作
           如：
	   
	   > db.info.remove({name: "ru"})
           > db.info.find();
           { "_id" : ObjectId("59ccdcd9ce94b645f5b101f8"), "name" : "heather", "sex" : "female" }
           { "_id" : ObjectId("59ccdbba5363253e0ef0f42d"), "age" : 20, "id" : 2, "name" : "moon" }
		
9.  高级操作，聚合 和 游标

    - 聚合
      常见的聚合操作跟sql server一样，有：count，distinct，group，mapReduce。
	   
     1. count
	     
	如：
	
	> db.info.count();
          2
        > db.info.count({name: "moon"});
          1
		  
      2. distinct ，指定哪个key，可以显示那个key下 所有不重复的value
		
        如：
	
        > db.info.distinct("name");
          [ "heather", "moon" ]
		  
      3. group 
         在mongodb里面做group操作有点小复杂，其实group操作本质上形成了一种“k-v”模型，有了这种思维，我们来看看如何使用group。

         下面举的例子就是按照age进行group操作，value为对应age的姓名。下面对这些参数介绍一下：
          
	 key：  这个就是分组的key，我们这里是对年龄分组。

        initial: 每组都分享一个”初始化函数“，特别注意：是每一组，比如这个的age=20的value的list分享一个initial函数，age=22同样也分享一个initial函数。

        $reduce: 这个函数的第一个参数是当前的文档对象，第二个参数是上一次function操作的累计对象，第一次为initial中的{”perosn“：[]}。有多少个文档， $reduce就会调用多少次。
        
	如：
	> db.info.find();
         { "_id" : ObjectId("59ccdcd9ce94b645f5b101f8"), "name" : "heather", "sex" : "female" }
         { "_id" : ObjectId("59ccdbba5363253e0ef0f42d"), "age" : 20, "id" : 2, "name" : "moon" }
         { "_id" : ObjectId("59cf018853be1d881737f967"), "name" : "jac", "age" : 22 }
         { "_id" : ObjectId("59cf019553be1d881737f968"), "name" : "joe", "age" : 23 }
         { "_id" : ObjectId("59cf01a153be1d881737f969"), "name" : "ru", "age" : 22 }
         { "_id" : ObjectId("59cf01af53be1d881737f96a"), "name" : "ace", "age" : 30 }
        
	> db.info.group({
		      key:{age: true}, 
			  initial: {info: []}, 
			  $reduce: function(cur, prev){prev.info.push(cur.name)}})
         [
	        {
		       "age" : null,
		       "info" : [
			         "heather"
		       ]
	        },
	        {
		       "age" : 20,
		       "info" : [
			         "moon"
		       ]
	        },
	       {
		       "age" : 22,
		       "info" : [
			        "jac",
			        "ru"
		       ]
	        },
	        {
		        "age" : 23,
		        "info" : [
			        "joe"
		        ]
	        },
	        {
		        "age" : 30,
		        "info" : [
			        "ace"
		         ]
	        }
        ]
		
	有时可能有如下的要求：

        1：想过滤掉age>25一些人员。

        2：有时info数组里面的人员太多，我想加上一个count属性标明一下。

        针对上面的需求，在group里面还是很好办到的，因为group有这么两个可选参数: condition 和 finalize。

        condition:  这个就是过滤条件。

        finalize:这是个函数，每一组文档执行完后，多会触发此方法，那么在每组集合里面加上count也就是它的活了
		
		> db.info.group({
		      key:{age:true}, 
			  initial:{info:[]}, 
			  $reduce: function(cur, prev){prev.info.push(cur.name)}, 
			  finalize: function(out){out.count = out.info.length}, 
			  condition: {age: {$lt: 25}}})
        
		[
	       {
		      "age" : 20,
		      "info" : [
			        "moon"
		      ],
		      "count" : 1
	       },
	       {
		     "age" : 22,
		     "info" : [
			       "jac",
			       "ru"
		     ],
		     "count" : 2
	       },
	       {
		     "age" : 23,
		     "info" : [
			        "joe"
		     ],
		     "count" : 1
	       }
        ]
		
		4. mapReduce

        是聚合函数中最复杂的了，不过复杂也好，越复杂就越灵活。

        mapReduce其实是一种编程模型，用在分布式计算中，其中有一个“map”函数，一个”reduce“函数。

        map：

          这个称为映射函数，里面会调用emit(key,value)，集合会按照你指定的key进行映射分组。

        reduce：

         这个称为简化函数，会对map分组后的数据进行分组简化，注意：在reduce(key,value)中的key就是

        emit中的key，vlaue为emit分组后的emit(value)的集合，这里也就是很多{"count":1}的数组。

        mapReduce:

        这个就是最后执行的函数了，参数为map，reduce和一些可选参数
		
		> map =  function(){emit(this.name, {count: 1})}
          function (){emit(this.name, {count: 1})}
 
        > reduce = function(key, value){var result = {count: 0}; for (var i = 0; i < value.length; i++) {result.count += value[i].count;} return result;}
          function (key, value){var result = {count: 0}; for (var i = 0; i < value.length; i++) {result.count += value[i].count;} return result;}
 
        > db.info.mapReduce(map, reduce, {out: "collection"})
         {
	       "result" : "collection",
	       "timeMillis" : 21,
	       "counts" : {
		         "input" : 6,
		         "emit" : 6,
		         "reduce" : 0,
		         "output" : 6
	        },
	        "ok" : 1,
         }
		 
		 result: "存放的集合名“；

         input:传入文档的个数。

         emit：此函数被调用的次数。

         reduce：此函数被调用的次数。

         output:最后返回文档的个数。
	   
         > 
         > db.collection.find();
           { "_id" : "ace", "value" : { "count" : 1 } }
           { "_id" : "heather", "value" : { "count" : 1 } }
           { "_id" : "jac", "value" : { "count" : 1 } }
           { "_id" : "joe", "value" : { "count" : 1 } }
           { "_id" : "moon", "value" : { "count" : 1 } }
           { "_id" : "ru", "value" : { "count" : 1 } }
        > 

    二： 游标
	
	     mongodb里面的游标有点类似闭包中的延迟执行，比如：

         var list=db.person.find();

        针对这样的操作，list其实并没有获取到person中的文档，而是申明一个“查询结构”，等我们需要的时候通过

        for或者next()一次性加载过来，然后让游标逐行读取，当我们枚举完了之后，游标销毁，之后我们在通过list获取时，发现没有数据返回了。
		
		> var list = db.info.find();
        > list.forEach(function(x){print(x.name)});
          heather
          moon
          jac
          joe
          ru
          ace

		  
10.  索引操作

    mongodb中关于索引的基本操作，日常开发都避免不了要对程序进行性能优化，而程序的操作无非就是CURD，通常
    又会花费50%的时间在R上面，因为Read操作对用户来说是非常敏感的。

    索引查找 带来的性能提升 见 如下：
	
	一：性能分析函数（explain）
	既然要做分析，肯定要有分析的工具，mongodb中提供了一个关键字叫做“explain"， 类似于Oracle的执行计划。
	
	> db.info.find();
     { "_id" : ObjectId("59ccdcd9ce94b645f5b101f8"), "name" : "heather", "sex" : "female" }
     { "_id" : ObjectId("59ccdbba5363253e0ef0f42d"), "age" : 20, "id" : 2, "name" : "moon" }
     { "_id" : ObjectId("59cf018853be1d881737f967"), "name" : "jac", "age" : 22 }
     { "_id" : ObjectId("59cf019553be1d881737f968"), "name" : "joe", "age" : 23 }
     { "_id" : ObjectId("59cf01a153be1d881737f969"), "name" : "ru", "age" : 22 }
     { "_id" : ObjectId("59cf01af53be1d881737f96a"), "name" : "ace", "age" : 30 }
    > 
    > 
    > db.info.find({name: "moon"}).explain();
   {
	"cursor" : "BasicCursor",
	"isMultiKey" : false,
	"n" : 1,
	"nscannedObjects" : 6,
	"nscanned" : 6,
	"nscannedObjectsAllPlans" : 6,
	"nscannedAllPlans" : 6,
	"scanAndOrder" : false,
	"indexOnly" : false,
	"nYields" : 0,
	"nChunkSkips" : 0,
	"millis" : 0,
	"indexBounds" : {
		
	},
	"server" : "host-192-168-2-38:27017"
   }
   
   解释：
   几个关心的key

   cursor:        这里出现的是”BasicCursor",意思是查找采用的是“表扫描”，也就是顺序查找。

   nscanned:      这里是6，也就是说数据库浏览了6个文档，如果有上万个文档的话，会很恐怖。

   n:             这里是1，也就是最终返回了1个文档。

   millis:        这个就是我们最最最....关心的东西，总共耗时的毫秒数。
   
   
   二：建立索引（ensureIndex）

     那么该如何优化呢？使用 mongodb中的索引查找，看看能不能让我们的查询一飞冲天.....
	 
	> db.info.ensureIndex({name: 1});
    > db.info.find({name: "moon"}).explain();
    {
	"cursor" : "BtreeCursor name_1",
	"isMultiKey" : false,
	"n" : 1,
	"nscannedObjects" : 1,
	"nscanned" : 1,
	"nscannedObjectsAllPlans" : 1,
	"nscannedAllPlans" : 1,
	"scanAndOrder" : false,
	"indexOnly" : false,
	"nYields" : 0,
	"nChunkSkips" : 0,
	"millis" : 0,
	"indexBounds" : {
		"name" : [
			[
				"moon",
				"moon"
			]
		]
	},
	"server" : "host-192-168-2-38:27017"
   }
   
   这里使用了ensureIndex在name上建立了索引。”1“：表示按照name进行升序，”-1“：表示按照name进行降序。
   
   关键的key:

   cursor:       这里出现的是”BtreeCursor"，这么牛X，mongodb采用B树的结构来存放索引，索引名为后面的“name_1"。

   nscanned:     数据库只浏览了一个文档就OK了。

   n:            直接定位返回。
   
   三：唯一索引

    建立唯一索引，重复的键值就不能插入，在mongodb中的使用方法是：

        > db.info.ensureIndex({"name": 1}, {"unique": true});
        > db.info.insert({name: "moon", age: 60})
          E11000 duplicate key error index: moon.info.$name_1  dup key: { : "moon" }
		  
	查询索引：
	    > db.info.getIndexes();
       [
	    {
		   "v" : 1,
		   "key" : {
			"_id" : 1
		   },
		   "ns" : "moon.info",
		   "name" : "_id_"
	    },
	   {
		"v" : 1,
		"key" : {
			"name" : 1
		},
		"unique" : true,
		"ns" : "moon.info",
		"name" : "name_1"
	   }
       ]
		  
		  
   四：组合索引

     有时候查询不是单条件的，可能是多条件，比如查找age 是 20，名字叫‘jack’的人，那么我们可以建立“姓名”和"age“的联合索引来加速查询。
	 
	> db.info.ensureIndex({name: 1, age: 1});
    > db.info.getIndexes();
   [
	{
		"v" : 1,
		"key" : {
			"_id" : 1
		},
		"ns" : "moon.info",
		"name" : "_id_"
	},
	{
		"v" : 1,
		"key" : {
			"name" : 1
		},
		"unique" : true,
		"ns" : "moon.info",
		"name" : "name_1"
	},
	{
		"v" : 1,
		"key" : {
			"name" : 1,
			"age" : 1
		},
		"ns" : "moon.info",
		"name" : "name_1_age_1"
	}
  ]
  > db.info.find({name: "moon", age: 20}).explain();
  {
	"cursor" : "BtreeCursor name_1_age_1",
	"isMultiKey" : false,
	"n" : 0,
	"nscannedObjects" : 0,
	"nscanned" : 0,
	"nscannedObjectsAllPlans" : 0,
	"nscannedAllPlans" : 0,
	"scanAndOrder" : false,
	"indexOnly" : false,
	"nYields" : 0,
	"nChunkSkips" : 0,
	"millis" : 0,
	"indexBounds" : {
		"name" : [
			[
				"moon",
				"moon"
			]
		],
		"age" : [
			[
				20,
				20
			]
		]
	},
	"server" : "host-192-168-2-38:27017"
  }

   查询优化器的选择往往是最优的，因为我们做查询时，查询优化器会使用我们建立的这些索引来创建查询方案，

   如果某一个先执行完则其他查询方案被close掉，这种方案会被mongodb保存起来，当然如果非要用自己指定的查询方案，在mongodb中给我们提供了hint方法让我们可以暴力执行
  
   > db.info.find({name: "moon", age: 50}).hint({name: 1}).explain();
{
	"cursor" : "BtreeCursor name_1",
	"isMultiKey" : false,
	"n" : 1,
	"nscannedObjects" : 1,
	"nscanned" : 1,
	"nscannedObjectsAllPlans" : 1,
	"nscannedAllPlans" : 1,
	"scanAndOrder" : false,
	"indexOnly" : false,
	"nYields" : 0,
	"nChunkSkips" : 0,
	"millis" : 0,
	"indexBounds" : {
		"name" : [
			[
				"moon",
				"moon"
			]
		]
	},
	"server" : "host-192-168-2-38:27017"
}


  五： 删除索引

     可能随着业务需求的变化，原先建立的索引可能不符合要求了，索引会降低CUD这三

     种操作的性能，因为需要实时维护，使用 dropIndexes 的使用：
	 
	 > db.info.dropIndexes("name_1")

   
	
	
    
