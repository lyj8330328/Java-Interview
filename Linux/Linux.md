# 一、文件和目录

1.pwd 显示工作路径

2.ls命令

ls 查看目录中的文件 
ls -l 显示文件和目录的详细资料 
ls -a 列出全部文件，包含隐藏文件
ls -R 连同子目录的内容一起列出（递归列出），等于该目录下的所有文件都会显示出来  
ls *[0-9]* 显示包含数字的文件名和目录名

3.cp

（用于复制文件，copy之意，它还可以把多个文件一次性地复制到一个目录下）
-a ：将文件的特性一起复制
-p ：连同文件的属性一起复制，而非使用默认方式，与-a相似，常用于备份
-i ：若目标文件已经存在时，在覆盖时会先询问操作的进行
-r ：递归持续复制，用于目录的复制行为
-u ：目标文件与源文件有差异时才会复制

4.mv

（用于移动文件、目录或更名，move之意）
-f ：force强制的意思，如果目标文件已经存在，不会询问而直接覆盖
-i ：若目标文件已经存在，就会询问是否覆盖
-u ：若目标文件已经存在，且比目标文件新，才会更新

5.rm

（用于删除文件或目录，remove之意）
-f ：就是force的意思，忽略不存在的文件，不会出现警告消息
-i ：互动模式，在删除前会询问用户是否操作
-r ：递归删除，最常用于目录删除，它是一个非常危险的参数

# 二、查看文件内容

### 7. cat命令

（用于查看文本文件的内容，后接要查看的文件名，通常可用管道与more和less一起使用）
cat file1 从第一个字节开始正向查看文件的内容 
tac file1 从最后一行开始反向查看一个文件的内容 
cat -n file1 标示文件的行数 
more file1 查看一个长文件的内容 
head -n 2 file1 查看一个文件的前两行 
tail -n 2 file1 查看一个文件的最后两行 
tail -n +1000 file1  从1000行开始显示，显示1000行以后的
cat filename | head -n 3000 | tail -n +1000  显示1000行到3000行
cat filename | tail -n +3000 | head -n 1000  从第3000行开始，显示1000(即显示3000~3999行)

# 三、文件搜索

### 8. find命令（）

find / -name file1 从 '/' 开始进入根文件系统搜索文件和目录 
find / -user user1 搜索属于用户 'user1' 的文件和目录 
find /usr/bin -type f -atime +100 搜索在过去100天内未被使用过的执行文件 
find /usr/bin -type f -mtime -10 搜索在10天内被创建或者修改过的文件 
whereis halt 显示一个二进制文件、源码或man的位置 
which halt 显示一个二进制文件或可执行文件的完整路径

删除大于50M的文件：
find /var/mail/ -size +50M -exec rm {} ＼;

# 四、文件权限

### 9. chmod 命令

ls -lh 显示权限 
chmod ugo+rwx directory1 设置目录的所有人(u)、群组(g)以及其他人(o)以读（r，4 ）、写(w，2)和执行(x，1)的权限 
chmod go-rwx directory1  删除群组(g)与其他人(o)对目录的读写执行权限

### 10. chown 命令

（改变文件的所有者）
chown user1 file1 改变一个文件的所有人属性 
chown -R user1 directory1 改变一个目录的所有人属性并同时改变改目录下所有文件的属性 
chown user1:group1 file1 改变一个文件的所有人和群组属性

### 11. chgrp 命令

（改变文件所属用户组）
chgrp group1 file1 改变文件的群组

# 五、文本处理

### 12. grep 命令

（分析一行的信息，若当中有我们所需要的信息，就将该行显示出来，该命令通常与管道命令一起使用，用于对一些命令的输出进行筛选加工等等）
grep Aug /var/log/messages  在文件 '/var/log/messages'中查找关键词"Aug" 
grep ^Aug /var/log/messages 在文件 '/var/log/messages'中查找以"Aug"开始的词汇 
grep [0-9]  /var/log/messages 选择 '/var/log/messages' 文件中所有包含数字的行 
grep Aug -R /var/log/* 在目录 '/var/log' 及随后的目录中搜索字符串"Aug" 
sed 's/stringa1/stringa2/g' example.txt 将example.txt文件中的 "string1" 替换成 "string2" 
sed '/^$/d' example.txt 从example.txt文件中删除所有空白行

### 13. paste 命令

paste file1 file2 合并两个文件或两栏的内容 
paste -d '+' file1 file2 合并两个文件或两栏的内容，中间用"+"区分

### 14. sort 命令

sort file1 file2 排序两个文件的内容 
sort file1 file2 | uniq 取出两个文件的并集(重复的行只保留一份) 
sort file1 file2 | uniq -u 删除交集，留下其他的行 
sort file1 file2 | uniq -d 取出两个文件的交集(只留下同时存在于两个文件中的文件)

### 15. comm 命令

comm -1 file1 file2 比较两个文件的内容只删除 'file1' 所包含的内容 
comm -2 file1 file2 比较两个文件的内容只删除 'file2' 所包含的内容 
comm -3 file1 file2 比较两个文件的内容只删除两个文件共有的部分

# 六、打包压缩文件

### 16. tar 命令

（对文件进行打包，默认情况并不会压缩，如果指定了相应的参数，它还会调用相应的压缩程序（如gzip和bzip等）进行压缩和解压）
-c ：新建打包文件
-t ：查看打包文件的内容含有哪些文件名
-x ：解打包或解压缩的功能，可以搭配-C（大写）指定解压的目录，注意-c,-t,-x不能同时出现在同一条命令中
-j ：通过bzip2的支持进行压缩/解压缩
-z ：通过gzip的支持进行压缩/解压缩
-v ：在压缩/解压缩过程中，将正在处理的文件名显示出来
-f filename ：filename为要处理的文件
-C dir ：指定压缩/解压缩的目录dir
压缩：tar -jcv -f filename.tar.bz2 要被处理的文件或目录名称
查询：tar -jtv -f filename.tar.bz2
解压：tar -jxv -f filename.tar.bz2 -C 欲解压缩的目录

bunzip2 file1.bz2 解压一个叫做 'file1.bz2'的文件 
bzip2 file1 压缩一个叫做 'file1' 的文件 
gunzip file1.gz 解压一个叫做 'file1.gz'的文件 
gzip file1 压缩一个叫做 'file1'的文件 
gzip -9 file1 最大程度压缩 
rar a file1.rar test_file 创建一个叫做 'file1.rar' 的包 
rar a file1.rar file1 file2 dir1 同时压缩 'file1', 'file2' 以及目录 'dir1' 
rar x file1.rar 解压rar包

zip file1.zip file1 创建一个zip格式的压缩包 
unzip file1.zip 解压一个zip格式压缩包 
zip -r file1.zip file1 file2 dir1 将几个文件和目录同时压缩成一个zip格式的压缩包

# 七、进程相关

### jps命令

（显示当前系统的java进程情况，及其id号）
jps(Java Virtual Machine Process Status Tool)是JDK 1.5提供的一个显示当前所有java进程pid的命令，简单实用，非常适合在linux/unix平台上简单察看当前java进程的一些简单情况。

### ps命令

（用于将某个时间点的进程运行情况选取下来并输出，process之意）
-A ：所有的进程均显示出来
-a ：不与terminal有关的所有进程
-u ：有效用户的相关进程
-x ：一般与a参数一起使用，可列出较完整的信息
-l ：较长，较详细地将PID的信息列出

ps aux # 查看系统所有的进程数据
ps ax # 查看不与terminal有关的所有进程
ps -lA # 查看系统所有的进程数据
ps axjf # 查看连同一部分进程树状态

### kill命令

（用于向某个工作（%jobnumber）或者是某个PID（数字）传送一个信号，它通常与ps和jobs命令一起使用）

### killall命令

（向一个命令启动的进程发送一个信号）

### top命令

是Linux下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况，类似于Windows的任务管理器。

如何杀死进程：
（1）图形化界面的方式
（2）kill -9 pid  （-9表示强制关闭）
（3）killall -9 程序的名字
（4）pkill 程序的名字

查看进程端口号：
netstat -tunlp|grep 端口号

# 八、普通文件和目录文件的区别

### 8.1 文件的类型

Linux下面一切皆文件，配置是文件，设备是文件，目录也是特殊的文件，文件有如下几种：
d：目录文件的标识，
-：普通文件标识，
l：软连接文件，亦称符号链接文件；
b，块文件，是设备文件的一种（还有另一种），b是block的简写。
c，字符文件，也是设备文件的一种，c是character的文件。

### 8.2 普通文件和目录文件

普通文件：存储普通数据，一般就是字符串。
目录文件：存储了一张表，该表就是该目录文件下，所有文件名和inode的映射关系。

### 8.3 权限的区别

对于普通文件来说，rwx的意义是：
r：可以获得这个普通文件的名字和内容。
w：可以修改这个文件的内容和文件名。可以删除该文件。
x：该文件是否具有被执行的权限。

对于目录文件来说，rwx的意义是：
r：表示具有读取目录结构列表的权限，所以当你具有读取(r)一个目录的权限时，表示你可以查询该目录下的文件名。 就可以利用 ls 这个命令将该目录的内容列表显示出来， 必须这个目录有x的权限，才可以进入这个目录。
w：移动该目录结构列表的权限（建立新的文件与目录、删除已经存在的文件与目录、更名、移动位置）。
x：目录不可以被执行，目录的x代表的是用户能否进入该目录成为工作目录。