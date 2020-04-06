## 批量管理ansible
> ansible服务特点

> * 管理端不需要启动服务程序（no server）
> * 管理端不需要编写配置文件（/etc/ansible/ansible.cfg默认配置就够了）
> * 被管理端不需要安装软件（如果有特属需求不能关闭selinux的情况下 需要安装libselinux-python，及时开启selinux的情况下也可以使用ansible）
> * 被管理端不需要启动服务程序（no agent）
> * 服务程序管理的操作模块有很多（module）
> * 利用剧本编写实现自动化（playbook）

`ansible-doc 命令行查看文档手册`
`ansible-doc -l 列出所有模块`
`ansinle-doc -s 列出模块详细信息`
### ansible批量管理服务部署
#### 管理服务端
	1.安装部署软件
	  yum -y install ansible  --  依赖epel的yum源
	2.需要编写主机清单文件
	  vim /etc/ansible/hosts
	  添加要管理的主机地址（前提要管理的主机已经分发好ssh秘钥可以免密登录）
	3.测试时候可以管理多个主机
	  ansible all -a "hostname"
	  
#### ansible服务架构
	1.主机清单配置
	2.软件模块信息
	3.基于秘钥连接管理主机
	4.主机需要关闭selinux
	5.剧本功能 
	
#### ansible软件模块应用
> 模块语法格式 

	ansible 主机名称/主机组名称/主机地址信息/all -m(指定应用的模块信息) 模块名称 -a(指定执行的动作) "执行什么动作"
	
* 第一个模块

####	
	command - Executes a command on targets   
	简单用法:
	ansible host -m command -a "hostname" (command是默认模块可省略)
####
	扩展应用:
	1. chdir --  在执行命令前对目录进行切换
	   ansible host -m command -a "chdir=/tmp touch 1.txt"
	2. creates --  如果文件存在了，就不执行后面命令操作  相当于一个判断  
	   ansible host -m command -a "creates=/tmp/hosts touch 1.txt" (如果远端tmp目录下存在了hosts文件 则就不执行后面的touch操作)
	3. removes -- 如果文件存在了，就执行后面命令操作 与creates相反
	   ansible host -m command -a "removes=/tmp/hosts touch 1.txt (如果远端tmp目录下存在了hosts文件 则执行后面的touch操作)
	 
	 注意事项:
	 有些符号无法在命令中识别: "<",">","|",";","&"
	  
* 第二个模块

####
	shell(万能模块) 
	简单用法:
	ansible host -m shell -a "hostname" 

####
	扩展应用:
	1. chdir --  在执行命令前对目录进行切换
	   ansible host -m shell -a "chdir=/tmp touch 1.txt"
	2. creates --  如果文件存在了，就不执行后面命令操作  相当于一个判断  
	   ansible host -m shell -a "creates=/tmp/hosts touch 1.txt" (如果远端tmp目录下存在了hosts文件 则就不执行后面的touch操作)
	3. removes -- 如果文件存在了，就执行后面命令操作 与creates相反
	   ansible host -m shell -a "removes=/tmp/hosts touch 1.txt (如果远端tmp目录下存在了hosts文件 则执行后面的touch操作)
	 
	 注意事项:
	 可以识别特殊符号: "<",">","|",";","&"
	 基本上是linux中的命令shell模块都支持
	
* 第三个模块

####
	script
	功能和command模块类似
	
	与shell模块对比
	同样是在远端机器上执行一个脚本(脚本在管理端机器上)
	
	用shell模块:
	1.编写脚本
	2.将脚本发送到远程主机上
	3.添加脚本执行权限
	4.运行ansible命令执行脚本
	
	用script模块:
	1.编写脚本
	2.运行ansible命令执行脚本
	
	ansible host -m script -a "/root/test.sh"
	
	ps:专业的模块干专业的事，不要都依赖于shell模块
	
`总结 以上三个模块都是属于命令类型模块`


* 第四个模块（文件类型模块）

####
	copy - copies files to remote locations
	       将数据进行批量分发至远程主机
	
	基本用法:
	ansible host -m copy -a "src=/etc/hosts dest=/etc/ backup=yes"
	传送文件之前先备份源文件
	
	扩展用法:
	ansible host -m copy -a "src=/etc/hosts dest=/etc/ owner=test group=test mode=666" 
	在传输文件的时候刚改文件的属主、属组还有权限信息
	
	ansible host -m copy -a "content='123456' dest=/etc/test.txt"
	直接在远程主机上创建文件并追加一行内容
	
	ansible host -m copy -a "src=/tmp/1.txt dest=/root remote_src=yes"
	remote_src默认是no表示从管理端复制文件到远程端
	如果remote_src=yes表示从远程端复制文件到远端目录中	
	ps:copy模块复制目录信息
	ansible host -m copy -a "src=/test dest=/root/ backup=yes"
	src目录后面没有/ : 表示将目录本身以及目录下面的内容都进行远程传输
	ansible host -m copy -a "src=/test/ dest=/root/ backup=yes"
	src目录后面有/ : 表示只将目录下面的内容都进行远程传输


* 第五个模块

####
	file - Sets attritubes of files
			设置文件的属性信息
 	基本用法:
 	ansible host -m file -a "dest=/etc/hosts owner=test group=test mode=666"
 	
 	扩展用法:
 	1.可以利用模块创建数据(文件、目录、链接)
 	state 参数 后面可以接
 	=absent          -- 缺席/删除远端数据
 	=directory       -- 远端创建一个目录信息
 	=file            -- 检查远端要创建的数据是否存在 如果返回红色则不存在 返回绿色存在
 	=hard				-- 远端创建一个硬链接文件
 	=link      		-- 远端创建一个软链接文件
 	=touch     		-- 远端创建一个文件信息 
 	
	1. 创建目录以及多级目录信息:
	ansible host -m file -a "dest=/root/test/test1/test2/ statr=directory"
	
	创建文件信息:
	ansible host -m file -a "dest=/root/test.txt state=touch"
	
	创建链接文件:
	ansible host -m file -a "src=/tmp/1.txt dest=/root/1_hard.txt state=hard"（src是在远端主机上的文件）
	ansible host -m file -a "src=/tmp/1.txt dest=/root/1_link.txt state=link"（src是在远端主机上的文件）
	
	2. 利用模块删除数据信息
	ansible host -m file -a "dest=/tmp/1.txt state=absent"（删除文件）
	ansible host -m file -a "dest=/tmp/test/ state=absent"（删除目录）
	
	自行研究模块:
	recuese -- set the specified file attributes (onle to directory)    针对指定目标目录递归修改文件属性
	=yes
	=no
	
* 第六个模块

####
	fethch - Fetch files from remote nodes
	         从远端主机拉取文件
	基本用法:
	ansible host -m fetch -a "src=/tmp/1.txt dest=/tmp"
	src为远端主机 dest为管理端 拉取过来的文件会以远程端的主机ip命名的一个文件夹形式

* 第七个模块

####
	yum - 安装软件
	基本用法:
	ansible host -m yum -a "name=htop state=installed"
	name - 指定软件名称
	state - 安装状态
	        
	state状态包括:
	installed、present、latest  -  为安装软件
	absent、removed            -  卸载软件
	
* 第八个模块

####
	service - 管理服务的运行状态 停止 开启 重启
	基本用法:ansible host -m service -a "name=xxx.service state=started enabled=yes"
	name - 指定管理的服务完整名称（systemctl status xxx.service查看完整名称）
	state - 指定服务状态
			 started
			 stopped
			 restarted
    enabled - 指定服务是否开机自启
 
* 第九个模块

####
	cron - 批量设置多个主机的定时任务
	 *   *  *   *  *  定时任务动作
    分  时  天  月  周
   
    基本用法:
    ansible host -m cron -a "minute=0 hour=2 job='/usr/sbin/ntpdate ntp1.aliyun.com > /dev/null 2>&1'"
   
    参数信息:
   
    minute: 设置分钟信息（0-59，*，*/2）
   
    hour:   设置小时信息（0-23，*，*/2）
   
    day:    设置日期信息（1-31，*，*/2）
   
    month:  设置月份信息（1-12，*，*/2）
   
    weekday: 设置周信息（0-6 for sunday-saturday，*）
   
    job: 用户定时任务需要干的事情
    
    扩展用法:
    1. 定时任务的描述信息:
    ansible host -m cron -a "name='time sync' minute=0 hour=2 job='/usr/sbin/ntpdate ntp1.aliyun.com > /dev/null 2>&1'"
    
    name - 添加定时任务的描述信息
    区分每条定时任务的作用 如果不添加 可能会有重复记录
    
    2. 删除指定定时任务
    	ansible host -m cron -a "name='time sync' state=absent"
    	PS:删除只能是ansible设置好的定时任务
    3. 批量注释定时任务
       ansible host -m crom -a "name='time sync' job='/usr/sbin/ntpdate ntp1.aliyun.com > /dev/null 2>&1' disabled=yes"
       PS:注释也只能注释ansible设置好的定时任务
       disabled=yes  -  注释
       disabled=no   -  取消注释

* 第十个模块

####
	mount - 批量进行挂载操作
	基本用法:
	ansible host -m mount -a "src=host:/data path=/mnt fstype=nfs state=mounted"
	
	src: 需要挂载的设备或文件信息
	path: 指定目标挂载点目录
	fstype: 指定挂载时的文件系统类型
	
	state
	
	=present/mounted  --  进行挂载
	present: 不会实现立即挂载，修改fstab文件，实现开机自动挂载
	mounted: 会实现立即挂载，并且修改fstab文件，实现开机自动挂载 *常用
	
	=absent/unmounted  -- 进行卸载
	absent: 会实现立即卸载，并且会删除fstab问价信息，禁止开机自动挂载
	unmounted: 会实现立即卸载，但是不会删除fstab文件信息 *常用
	
* 第十一个模块

####
	user - 实现批量创建用户
	基本用法:
	ansible host -m user -a "name=test"
	
	扩展用法:
	1.指定uid信息
	ansible host -m user -a "name=test uid=6666"
	
	2.指定用户组
	ansible host -m user -a "name=test group=test2"(主要组)
	ansible host -m user -a "name=test2 groups=test2"(附加到另外的组)

	3.批量创建虚拟用户
	ansible host -m user -a "name=rsync create_home=no shell=/sbin/nologin"
	
	4.给指定用户创建密码
	 ps:利用ansible程序user模块设置用户密码信息，需要将密码明文信息转换为密文信息进行设置
	 ansible host -m user -a 'name=test password=密文密码'
	 生成密码密码的时候注意:
	'name=test password=密文密码' 不能用双引号 否则有转义问题 或者密码中的有特殊字符用'\'转义
	 
	 生成密文密码信息方法:
	 方法一:
	 ansible all -i localhost, -m debug -a "msg={{ '123456' | password_hash('sha512', 'mysecretsalt') }}"
	 
	 方法二:
	 pip install passlib
	 python3 -c "from passlib.hash import sha512_crypt;import getpass;print(sha512_crypt.using(rounds=5000).hash('123456'.encode('utf-8')))"
	
    
    
   
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    

	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
