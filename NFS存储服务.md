## NFS网络存储服务

### NFS的作用
* 实现数据共享存储
* 便于数据操作管理
* 节省购买服务器磁盘开销(只需要存储服务器磁盘大点就行)

### NFS部署流程
`rpc介绍`

	rpc服务 - 相当于租房的中介 先启动rpc 再启动nfs服务的瞬间 nfs会主动去rpc注册启动的端口
	当客户端访问的时候是去访问rpc服务 rpc给你响应 响应完客户端才能去访问nfs服务 才能把数据存储到nfs对应的存储目录中
	因为nfs服务启动之后是多个端口多个进程而且每次重启服务端口都是随机的

#### 服务端部署流程
	1.下载安装软件
	yum install -y nfs-utils rpcbind
	2.编写nfs的配置文件
	vim /etc/exports 空文件
	格式：01      02(03)
	01：设置数据的存储目录
	02：设置一个白名单(允许哪些主机连接到存储服务器进行数据存储)
	03：配置存储目录的权限信息以及存储目录一些功能
	例:默认配置
	/data       host(rw,sync)
	3.创建一个存储目录
	mkdir /data
	chown nfsnobady.nfsnobady /data（授权特定的用户 nfsnobady为安装nfs是就创建好的用户 否则会有权限错误）
	4.启动服务
	先启动 rpc服务（111端口）
	systemctl start rpcbind.service
	systemctl enable rpcbind.service
	再启动 nfs服务（直接 netstat -tnlp | grep nfs 是看不到端口号的 因为nfs把端口号告诉了rpc服务 需要用rpcinfo -p host/nfs主机/命令查看nfs的端口号）
	systemctl start nfs
	systemctl enable nfs
	
#### 客户端部署流程
	1.客户端要先安装nfs 否则挂载nfs系统的的时候会报错
	yum install -y nfs-utils
	2.实现远程挂在共享目录
	mount -t nfs host:/data  /mnt
	
	
### NFS工作原理
	服务端:
	1.启动rpc服务 开启111端口
	2.启动nfs服务 并自动像rpc进行端口注册(重启一次服务只会注册一次)
	ps:查看nfs服务进程端口与注册信息
	rpcinfo -p localhost （nfs主机）
	
	客户端:
	1.建立tcp网络连接
	2.客户端执行mount命令，进行远程挂载
	3.可以实现数据远程传输存储


### nfs服务端详细配置说明
	1.实现多网段主机进行挂载
	/data       host/24(rw,sync) host2/24(rw,sync)
	或
	/data       host/24(rw,sync)
	/data       host2/24(rw,sync)
	总结:共享目录的权限和哪些因素有关:
	1)和存储目录本身的权限有关（权限是不是755，属主是不是nfsnobody ）
	2)和配置文件中的权限配置有关 rw/ro 
	3)还有配置文件中的各种squash以及anonuid/anongid有关
	4)和客户端挂载命令参数有关 ro



### NFS配置文件参数权限
	rw    --    存储目录是否有读写权限
	ro    --    存储目录是否是只读权限
	sync    --    同步
	async    --    异步
	no_root_squash    --    不要将root用户身份进行转换
	root_squash    --    将root用户身份进行转换
	all_squash    --    将所有用户身份都进行转换
	no_all_squash    --    不将普普通用户身份进行转换
	
	
	企业如何配置NFS配置文件 各种squash参数
	保证网站存储服务器用户数据的安全性：
	no_all_squash    需要进行配置    共享目录权限为其他用户不是nfsnobady(确保客户端用户和服务端用户uid数值一致)
	root_squash      需要进行配置    将root用户映射成nfsnobady   存储目录权限是其他用户不是nfsnobady 
	以上也是默认配置
	查看nfs的默认配置:
	cat /var/lib/nfs/etab -- nfs服务默认配置记录信息(也是nfsnobady的家目录)
	解释上面root_squash：
	如何让root用户可以操作管理其他用户管理的存储目录
	让root用户通过root_squash 映射成和存储目录一样的普通用户 而不是nfsnobady
	anonuid=65534,anongid=65534 这是nfsnobady的id -- 可以指定映射用户信息
	修改映射用户:普通用户www的id是1002
	/data    host/24(rw,sync,anonuid=1002,anongid=1002)
	
	企业中如何编辑nfs配置文件
	01.通用方法:
	/data    host/24(rw,sync)
	02.特殊情况（让部分人员不能操作存储目录，只能查看）
	/data    host/24(ro,sync)
	03.修改默认的映射用户nfsnobady(anonuid为其他用户的id)
	/data    host/24(rw,sync,anonuid=1002,anongid=1002)
	
	
	
### NFS服务的问题
	1.nfs服务重启，重新挂载后创建数据比较慢
	服务重启方式不对
	restart  重启服务    强制断开所有连接用户感受不好
	reload   平滑重启    强制断开没有数据传输的连接提升用户感受
	
### 客户端详细配置说明
	mount -t nfs host:/data  /mnt
	
	如何实现自动挂载
	01.利用 /etc/rc.local 增加执行权限
	echo "mount -t nfs host:/data  /mnt" >> /etc/rc.local
	02.利用/etc/fstab
	vim /etc/fstab
	host:/data    /mnt       nfs       defaults    0 0
	需要开启remote-fs.target这个服务实现自动挂载(centos6是autofs)
	centos7无法开机启动挂载因为根据系统启动顺序造成的
	同样服务的情况下centos7启动比centos6快因为7的服务是并行启动
	挂载nfs需要网络，但是centos7开机的时候是先加载/etc/fstab,但是这个时候还没有启动网络服务，所以导致挂载不上，有了remote-fs.target这个服务，他是在网络服务启动之后启动的，启动了remote-fs.target会重新加载一遍/etc/fstab从而达到自动挂载


### 客户端mount命令参数
	-a    自动加载/etc/fstab中所有设备文件
	-o    指定加载文件系统时的选项。有些选项也可在/etc/fstab中使用 -o后面包括下面的参数
	rw    --    实现挂载后挂载点可读可写（默认）
	ro    --    实现挂载后挂载点只读
	suid    --    在共享目录中可以让setuid权限生效（默认）可以设置chmod +s的权限 （例如自己创建个cat命令并设置s的权限就可以在普通用户的情况下 查看没有读权限的文件了）
	nosuid    --    在共享目录中可以让setuid权限失效 不可以设置chmod +s的权限  提供共享目录的安全性
	exec    --    共享目录中的执行文件可以直接执行
	noexec    --    共享目录中的执行文件无法直接执行    提供共享目录的安全性
	auto    --    可以实现自动挂载    mount -a 实现加载fstab文件自动挂载
	noauto    --    不可以实现自动挂载
	nouser    --    禁止普通用户可以下载挂载点
	user    --    允许普通用户卸载挂载点

### NFS服务挂载不上排查思路
	服务端进行排查:
	1.检查nfs进程信息是否注册
	  rpcinfo -p host
	  如果有问题服务启动顺序不对 应该先启用rpc，在启用nfs
	2.检查有没有可用的存储目录
	  cat /etc/exports 查看配置文件
	  或者
	  命令查看:showmount -e host
	  重启nfs服务
	3.在服务端本地进行挂载测试 时候能在存储目录中创建或删除数据

    客户端进行排查:
    1.检查nfs进程信息是否注册
	  rpcinfo -p host
	  如果有问题服务启动顺序不对 应该先启用rpc，在启用nfs
	2.检查有没有可用的存储目录
	  cat /etc/exports 查看配置文件
	  或者
	  命令查看:showmount -e host
	  重启nfs服务
	3.客户端和服务端网络是否可达
	 
	
	

### 客户端卸载

	umount -lf /mnt    --  强制卸载挂载点
	-l    不用退出挂载点目录
	-f    强制进行卸载操作






























	
	
	
	
















