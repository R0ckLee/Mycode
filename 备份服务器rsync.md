## 备份服务器
### 作用
	1.备份数据
	2.日志统一保存
	
### 搭建备份服务器：rsync服务

### 什么是rsync服务
	Rsync是一款开源的、快速的、多功能的，实现全量及增量的本地或远程数据同步备份的工具
	
### rsync原理
	客户端执行传输数据的命令
	客户端发送一个请求给服务端 告诉他我想传输数据了
	服务端有个门会问你是谁 客户端说我是认证用户rsync_backup
	服务端会去配置文件查看这个认证的用户 如果有就继续 没有就拒绝了
	服务端再检查密码是否正确
	传过来的文件是以服务端配置文件设置的rsync这个虚拟用户来存到配置文件中设置的备份目录中
	
	



### rsync软件使用方法:
1 V 4

* 可代替本地cp命令格式一致

	cp -rp /etc/hosts /tmp (-r递归，-p保持文件属性一致)
	
	rsync -rp /etc/hosts /tmp

* 可代替远程scp命令

	scp -rp /etc/hosts root@10.10.20.25:/backup/
	
	rsync -rp /etc/hosts root@10.102.0.25:/backup/
	
	远程目录备份：
	rsync -rp /backup root@10.10.20.25:/backup（不带 / 表示备份目录以及下面的文件）
	
	rsync -rp /backup/ root@10.10.20.25:/backup（带 / 表示只备份目录下面的文件）

	
	
* 可代替删除命令rm

	rm -rf /tmp/*
	
	rsync -rp --delete /null/ root@10.10.20.25:/tmp/（null为空目录）
	
	
## rsync守护进程的方式备份
	
### rsync服务端配置详细步骤:
### 1.下载安装软件（centos7自带）
	   rpm -qa | grep rsync
	   yum install -y rsync
	
### 2.编写配置文件
	#指定管理备份目录的用户
	uid = rsync
	#指定管理备份目录的用户组
	gid = rsync
	#和安全相关的配置
	use chroot = no
	#监听端口
	port = 873
	#传输过程是否将虚拟用户伪装超级管理员授权
	fake super = yes
	#日志文件位置
	log file = /var/log/rsyncd.log
	#锁文件 用来保证最大的连接数超过的连接数就锁定
	lock file = /var/run/sync.lock
	#存放进程ID的文件位置，判断服务是否开启 方便快速关闭服务
	pid file = /var/run/rsyncd.pid
	#允许访问的客户机地址
	hosts allow = 10.10.20.0/23
	#禁止访问的客户机地址
	#hosts deny = 0.0.0.0/32
	#最大连接数 同时连接服务端的客户端数量
	max connections = 20
	#超时时间 客户端连接上之后没有数据传输后倒计时300s
	timeout = 300
	#传输过程忽略简单错误
	ignore errors
	#指定备份目录是否为只读
	read only = false
	#是否客户端可以远程查看模块信息
	list = false
	#授权认证账户
	auth users = rsync_backup
	#指定用户密码文件 用户名称:密码
	secrets file = /etc/rsync.password
	#共享模块名称23
	[backup]
	comment = "backup dir"
	#源目录的实际路径
	path = /backup	
	
### 3.创建rsync服务的虚拟用户
	useradd rsync -M -s /sbin/nologin

### 4.创建认证用户的密码文件
	echo "rsync_backup:密码" > /etc/rsync.password
	chmod 600 /etc/rsync.password (只有root用户有查看权限 )
	
### 5.创建模块下保存备份的目录并修改所属主和组
	mkdir /backup
	chown rsync.rsync /backup/
	
### 6.启动备份服务并设置开机自启
	systemctl start rsyncd
	systemctl enable rsyncd
	systemctl status rsyncd
	
	

### rsync守护进程语法 
	推和拉两种 在客户端进行
	客户端拉的操作: 恢复数据
	客户端推的操作: 备份数据
### 1.Push
	rsync src [USER@]HOST::DEST
	src：要推送备份数据的信息
	[USER@]：指定认证用户 （配置文件中设定的用户）
	HOST：指定远程主机的IP地址或主机名称
	::DEST：备份服务器的模块名称 （也是配置文件中指定的模块名字）[backup]

### rsync客户端配置:
	1.创建一个密码文件
	echo "密码" > /etc/rsync.password
	chmod 600 /etc/rsync.password
	
	2.免交互传输信息
	加上客户端指定的密码 --password-file=/etc/password
	
	
	
### rsync常用参数
	-v, --verbose 显示详细传输信息
	-a, --archive 归档参数 包含的参数有：rtopgDl
	-r, --recursive 递归参数
	-t, --times 保持文件属性的时间信息不变（修改时间）
	-o, --owner 保持文件的属主信息不变
	-g, --group 保持文件的属组信息不变  
	ps:如何让-o和-g参数生效，配置文件要修改 需要加-o和-guid和gid为root，fake super要注释
	-p, --perms 保持文件的权限信息不变
	-D, 	      保持设备文件信息不变
	-l, --link   保持软链接文件属性信息不变 （只是一个软连接 没有源文件数据）
	-L,		      保持文件数据信息不变 （备份源文件的数据）
	-P,			   显示传输进度
	--exclude=PATTERN      排除指定数据不被传输（单个文件）
	--exclude-from=file		排除指定数据不被传输（批量 指定一个文件）
	--delete			无差异同步传输（我有的你也有，我没有的你也不能有）
	
### 守护进程企业服务的应用
#### 多模块功能
	配置文件下配置多个模块供不通需求使用
	一个模块对应一个目录

#### 守护进程的排除功能
	例:排除b目录下的1.txt和c目录
	rsync -avz /backupfile --exclude=b/1.txt --exclude=c/ rsync_backup@host::backup --password-file=/etc/rsync.password
	例:排除多个目录下的不同文件 批量排除
	先编辑一个排除文件exclude.txt把要排除的多个文件的路径信息（一行一个）别忘了把exclude.txt这个文件本身也排除了 
	多个目录多个文件的时候可有用find命令找到要排除的文件路径复制到这个文件中
	rsync -avz /backupfile --exclude-from=/exclude.txt rsync_backup@host::backup --password-file=/etc/rsync.password
	
	
#### 守护进程创建模块下面的子目录 并且只支持创建一个子目录不支持创建多及目录
	rsync -avz /backupfile rsync_backup@host::backup/子目录/ --password-file=/etc/rsync.password
	
	
	
#### 守护进程的访问控制黑白名单
	配置文件中
	hosts allow 
	hosts deny
	第一种情况：
	只允许白名单没有禁止黑名单
	白名单允许 其余流量禁止
	第二种情况：
	没有允许白名单 只有禁止的黑名单
	黑名单禁止 其余流量允许
	第三种情况：
	既有白名单也有黑名单
	白名单允许 黑名单禁止 其余流量允许 白名单优先级大

#### 守护进程的列表功能
	
	配置文件中的 list false/true
	如果为true
	查看远程服务端的模块信息 rsync rsync_backup@host::
	一般不建议开启 出于安全考虑
	
	
	
## 备份项目案例
	项目需求：
	1）所有服务器的备份目录为/backup
	2) 备份各系统的配置文件、网站目录、日志文件等
	3）服务器上要打包保留7天的备份数据
	4）备份服务器上要保留没周一的数据备份，其他的只保留6个月的
	5）备份服务器上要按照备份服务器的内网ip为目录保留备份，备份文件名按照时间保存
	6）需要确保备份服务器上的数据完整性，在备份服务器上进行数据检查，把备份的成功及失败结果发送给管理员的邮箱
	
`PS：tar 打包文件的时候 “-h” 参数 是可以备份软链接的真实数据`
`find 命令 ！-name 可以取反`
`服务端和客户端备份目录的路径应该一直，否则后面md5sum验证数据一致性的时候会出问题`
	
### 客户端备份脚本
	#!/bin/bash
	Backup_dir = "/backup"
	# 获取内网ip地址/etc/hosts文件内需要有对应的主机名和ip的映射
	HOST = $(hostname -i) 
	# 创建备份目录以主机ip命名
	mkdir -p $Backup_dir/$HOST
	# 打包要备份的数据	
	cd /
	# 实际是零点进行备份要备份前一天的数据所以要 -1day
	tar zchf $Backup_dir/$HOST/backup_data_$(date +%F_week%w -d -1day) ./path/file ./... ./...可以接多个
	# 删除7天前的数据
	find $Backup_dir/ -type f -mtime +7 |xargs rm 2>/dev/null
	# 创建指纹文件 用来保存每一天最新备份文件的md5值
	find $Backup_dir/ -type f -mtime -1 ! -name "finger.txt" | xargs md5sum > $Backup_dir/$HOST/finger.txt
	# 推送备份数据到备份服务器
	rsync -avz $Backup_dir/ rsync_backup@host::backup --password-file=/etc/rsync.password
	
### 服务端脚本
	#!/bin/bash
	Backup_dir = "/backup"
	# 删除30天以上的备份
	find $Backup_dir/ -type f -mtime +180 ! -name "不删除文件的名字" | xargs rm 2>/dev/null
	# 检查备份数据的完整性
	find $Backup_dir/ -type f -name "finger.txt" | xargs md5sum -c > /tmp/check.txt
	# 检查结果发送邮件
	首先配置发邮件的配置文件
	vim /etc/mail.rc
	最后添加
	set from=liyan@yixia.com smtp=smtp.yixia.com
	set smtp-auth-user=liyan@yixia.com smtp-auth-password=密码 smtp-auth=login
	systemctl restart postfix.service
	mail -s "备份数据 - $(date +%F)" liyan@yixia.com < /tmp/check.txt
	
`sh -x 脚本 脚本详细步骤拍错`
`自动全网备份把服务网段和客户端的脚本放到定时任务里`


	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	