可以把文件描述符看作一个变量这个变量暂存输入输出，这个变量还指向了一个文件，文件用来保存变量的内容。

2>/dev/null
意思就是把错误输出到“黑洞”

/dev/null 2>&1
默认情况是1，也就是等同于1>/dev/null 2>&1。意思就是把标准输出重定向到“黑洞”，还把错误输出2重定向到标准输出1，也就是标准输出和错误输出都进了“黑洞”

2>&1 >/dev/null
意思就是把错误输出2重定向到标准出书1，也就是屏幕，标准输出进了“黑洞”，也就是标准输出进了黑洞，错误输出打印





查看当前使用什么shell可以使用命令：echo $0
skip to head:ctrl+a
skip to end:ctrl+e

rman crosscheck
vi 
	skip to last line:gg
	skip to firsrt line:shift+g
	:set ignorecase
	:set noignorecase
	:/字符串+空格+\c

1.linux下递归列出目录下的所有文件名（不包括目录）
  ls -lR |grep -v ^d|awk '{print $9}'
2.linux下递归列出目录下的所有文件名（不包括目录），并且去掉空行
  ls -lR |grep -v ^d|awk '{print $9}' |tr -s '\n'

du -m -d 1 /|sort -nr
find / -size +500M -print0|xargs -0 du -m|sort -nr
在当前目录下查找所有用户权限644的文件，并更改权限600
find . -perm 644 | xargs chmod 600

ls -ltr

wget -c 

git clone ssh://i520726@hdbgerrit.wdf.sap.corp:29418/hana /home/I520726/SAPDevelop/HDB/git/sys/src

查看端口情况
1、lsof -i:端口号
2、netstat -tunlp|grep 端口号

为了频繁的执行某些只有超级用户才能执行的权限，而不用每次输入密码，可以使用该命令
sudo -i 运行结果 PWD=/root
sudo su 运行结果 PWD=/home/用户名（当前用户主目录）