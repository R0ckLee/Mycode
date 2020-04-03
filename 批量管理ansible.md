## 批量管理ansible

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
	ansible host -m shell -a "hostname" (command是默认模块可省略)

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
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
