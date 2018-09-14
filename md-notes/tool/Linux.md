## Linux常用命令

### Linux目录结构

```
bin(binaries)：存放二进制可执行文件
sbin(super user binaries)：存放二进制可执行文件，只有root才可以访问
etc(etcetera)：存放系统配置文件，profile文件是配置环境变量的
usr(unix shared resources)：用于存放共享的系统资源
home：存放用户文件的根目录
```

### 切换目录

```
cd /：切换到根目录
cd /usr：切换到根目录下的usr目录中
cd ../：切换到上级目录
cd ./a：切换到当前目录的a文件夹中，./可以省略
cd ~：切换到当前用户的home目录
cd -：切换到刚才所在的目录
pwd(print working directory)：显示当前在哪个文件夹里
```

### 操作文件夹的命令

```
mkdir 文件夹名称：创建文件夹
ls [-al]：查看文件夹   ll：是ls -l的缩写
find 搜索位置 -name "文件名称"：搜索文件夹
mv 文件夹名称 目标位置：剪切文件夹
mv 文件夹名称 新名称：重命名文件夹
cp -r 文件夹 目标位置：拷贝文件夹
rm -rf 文件夹：删除文件夹，f表示强制执行不提示
```

### 操作文件的命名

```
touch 文件名称：创建文件
cp 原文件 文件名：拷贝文件
mv 文件名 目标位置：移动文件
mv 旧文件名 新文件名：重命名文件
rm -f 文件名：删除文件，-f表示强制删除不提示
cat/more/less/tail 文件名称：查看文件
	cat：不能翻页
	more：只能往后翻，按Q退出
	less：可以前后翻，按Q退出
	tail：查看文件尾部内容
vi/vim 文件名称：编辑文件
	普通模式：默认进入该模式，按"i/a/o"进入编辑模式，按":"进入底行模式
	编辑模式：对文件内容进行编辑修改，退出该模式"esc"
	底行模式：进入该模式，输入"wq"：保存并退出，"!q"强制退出不保存
grep 表达式 文件 [--color]：搜索符合表达式的文件内容
```

### 压缩与解压缩命令

```
.tar：打包归档，没有压缩
.gz：压缩文件
.tar.gz：打包并压缩

压缩文件：
	tar -zcvf 压缩包名 要压缩的文件1 要压缩的文件2 ...
		z：调用压缩命令进行压缩
		c：要创建压缩包文件
		v：显示压缩过程
		f：指定压缩包文件名称
解压缩文件(重点)
	tar -xvf 压缩包名 -C 解压位置
		x：extract，提取文件
		C：解压到哪个位置，解压的文件夹必须存在，不指定C，表示解压到当前文件夹里
```

### 其他常用命令

```
ps -ef：查看进程
	e：显示所有程序
	f：显示进程详细信息
kill -9 pid：强制结束pid对应的进程
ps -ef|less：查找所有进程信息
ps -ef|grep "java" --color：查找所有进程，获取其中含有"java"字符串的内容
ifconfig：查看网络配置信息
ping：测试网络
netstat -anp：查看网络连接情况，可以查询占用某端口的进程pid
	a：显示所有网络连接
	n：显示ip地址
	p：显示程序的pid
关机命令和重启
	halt：关机，reboot：重启
结束占用8080端口的进程(重要)
	netstat -anp | grep ":8080" --color
	kill -9 pid
```

### 权限管理

```
drwxr-xr-x ...
	d：表示一个文件夹，"-"表示一个文件，"l"：表示一个链接
	第2-4位：文件拥有者对当前文件的权限，r-read，w-write，x-execute，-表示无权限
	第5-7位：同组用户对当前文件的权限
	第8-10位：其他用户对当前文件的权限
权限管理命令
	chmod 权限 文件
	 chmod u=rwx，g=rwx，o=rx test.txt
	 u-user，g-group，o-other
权限的增量设置模式
	+表示增加授权，-表示取消授权
	chmod u+r test.txt
```

### Linux软件安装命令

```
rpm命令
	rpm -ivh 安装包：安装
		i：install
		v：显示执行过程
		h：安装时列出标记
	rpm -qa：查询
		q：query
		a：查询所有软件
	rpm -e --nodeps 软件名
		e：erase，删除
		nodeps：忽略软件包之间的关联
yum命令
	yum install 软件名
	yum remove 软件名
```

### Linux中mysql操作

```
启动mysql服务：service mysql start
把mysql添加到系统服务：chkconfig --add mysql
设置mysql服务为开机自启动：chkconfig mysql on
修改密码：set password=password("新密码")
修改权限sql语句：grant all on "." to 'root'@'%' identified by 'root';
让权限变更立即生效：flush priviledge;
开发3306端口：/sbin/iptables -I INPUT -p tcp --dport 3306 -j ACCEPT
防火墙设置永久保存：/etc/rc.d/init.d/iptables save
```

