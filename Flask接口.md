### 记一次Flask接口
### '''
### 同步账号 并记录	
### '''
#### 此接口的作用是把访问这个接口会把人员的邮箱名存在文本中

	@app.route('/getstatus',methods=['post','get'])
	def get_oastatus():
	    mail = request.args.get('mail')
	    if '@xxx.com' in mail:
	        with open('mail_list.txt','a')as f:
	            f.write(mail+'\n')
	            with open('mail_list.txt','r')as line:
	                lines = line.readlines()
	                if len(lines):
	                    pass
	                return json.dumps({'code':200})
	    else:
	        return json.dumps({'code':204,'msg':'用户名有误'},ensure_ascii=False)
	return data
	if __name__ == "__main__":
	    app.debug = 1
	    app.run("0.0.0.0", 8000)

### UWSGI部署flask
	  1 [uwsgi]
	  2 socket = 127.0.0.1:9001
	  3 wsgi-file = /path/app3.py
	  4 callable = app
	  5 processes = 2
	  6 threads = 2
	  7 master = true
	  8 logto = /path/uwsgioa.log
	  9 daemonize = /path/uwsgioa.log
	  
### nginx中关联uwsgi模块
	 server {
         listen      8080;
         location / {
             uwsgi_pass 127.0.0.1:9001;
             include uwsgi_params;
         }
### 脚本读文件拿到邮箱名并进行授权
	# /usr/local/bin/python3
	 # -*- coding:utf-8 -*-
	 import requests
	 import json
	 import time
	 import sys
	 import re
	 
	 #  账号
	  session = requests.Session()
	  header = {'User-Agent': 'Mozilla/5.0 (compatible; WOW64; MSIE 10.0; Windows NT 6.2)',
	            'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8',
	  }
	  
	  # 登录请求
	def login():
	    login_url = '登陆url'

    login_data = {
        'login.timezone': 'GMT+8:00',
        'login_username': '用户名',
        'login_password': '密码', }
    response = session.post(login_url, data=login_data, headers=header)
    return response


	def get_id(name):
	    url='获得oaidurl{}'.format(name)
	    oaid = session.get(url,headers=header)

    id = re.findall(r'<a6id>(.*?)</a6id>',id.text,re.S)

    return  id[0]+','


	def config_save(oid,name):
	    url='保存的url'
	    data = {
			"param": "参数"
	    }
	    print(data)
	    save_config = session.post(url,data=data,headers=header)
	    print(save_config.text)
	  
	  
	  if __name__=='__main__':
	       #login()
	    try:
	        with open('/path/mail_list.txt','r')as f:
	            lines = f.readlines()
	            if len(lines):
	                login()
	                for name in lines:
	                    # oaname = lines[-1].strip()
	                    name = name.strip()
	                    oid = get_id(name.strip())
	                    print('账号已开通 - %s - %s'%(name,time.strftime('%Y-%m-%d %H:%M:%S')))
	                    config_save(oid,name)
	                with open('/path/mail_list.txt','w')as f:
	                    f.truncate()
	                print('文件已清空\n')
	            else:
	                print('文件为空 - %s\n'%time.strftime('%Y-%m-%d %H:%M:%S'))
	    except IndexError:
	        print('用户名无效\n')       

	  
