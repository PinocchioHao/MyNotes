### **Linux Commands**

I use commands like **ls**, **ll**, and **cd** for directories; **cat**, **grep**, and **tail** to check logs; **chmod** for permissions; and **ps** or **kill** to manage processes.


查找java进程
`ps -ef | grep java`
杀死进程
`kill -9 pid`

windows shell 对应命令:
`netstat -ano | findstr 12345`
`taskkill /PID 12345 /F`

`nohup java -jar service-energy-reporting-0.0.1-SNAPSHOT.jar > energy-reporting.log 2>&1 &` 后台启动Java程序，不受ssh会话影响
`java -jar xxx.jar &` 后台运行




`sudo` 给后续操作都加上管理员权限

`free -h` 查看剩余空间

AWS EC2 用dnf来进行包管理 -- **Amazon Linux 2023**
``` shell
sudo dnf update -y  --更新 
sudo dnf install nginx -y

```


nginx
```shell
# 启动 nginx
sudo systemctl start nginx

# 查看状态，active(running)则表示正常启动
sudo systemctl status nginx

# 设置开机自启
sudo systemctl enable nginx

# 重启nginx
sudo systemctl restart nginx
```

前端文件注意放在share文件之类的nginx能够访问的位置，否则权限报错不能访问


默认主配置文件是：

```
/etc/nginx/nginx.conf
```

这个文件通常包含如下内容：

Nginx Config

include /etc/nginx/conf.d/*.conf;

include /etc/nginx/default.d/*.conf;

这表示：

- 所有在 `/etc/nginx/conf.d/` 目录下的 `.conf` 文件都会被加载
- 所有在 `/etc/nginx/default.d/` 目录下的 `.conf` 文件也会被加载

`conf.d/`目录下的`myapp.conf`
```shell
server {
	listen 80;
	server name -'
	root /usr/share/nginx/html/myapp;index index.html index.htm;
	location /{
		try_files $uri /index.html;
	}
}
```

**推荐做法：在 `conf.d` 中创建你自己的配置文件**








连接ec2

```bash
ssh -i My_ec2_key.pem ec2-user@13.237.197.44
3.107.28.186
```


`tree`命令可以输出文件结构目录，默认是文件夹结构，如果要输出所有文档可以`tree /f (后可加文件夹)`



##### 挂载共享文件（虚拟机）：
VirtualBox‘设置好宿主机共享文件夹，自动挂载，固定分配
`sudo mount -t vboxsf VirtualBoxShare(设置里的共享文件夹名称) ./Shared`



### Mysql踩坑
安装，设置安全组，AWS配置入站规则，以及MySQL要配置user允许哪个ip进行访问
`EC2 MySQL  密码  StrongPass!123`







### python项目运行踩坑
先安装python，进入项目根目录设置开发环境`python3 -m venv venv`  -- 但是注意AWS Linux 2023 默认是python3.9，而`xxx: str | None = None`这种语法只有python3.10之后才支持，所以要安装3.10及之后的版本使用`python3.11 -m venv venv`创建环境


1. python版本，3.9跟3.10语法支持差别 -- 这里我选择了妥协，改代码为3.9兼容
2. 在项目根目录执行`source venv/bin/activate`进入python环境，这样项目的依赖安装`pip install -r requirements.txt`就只是跟随项目，而不是被全局安装到本地环境中。以及执行`uvicorn`命令需要在虚拟环境`venv`中，最后使用`deactivate`可退出虚拟环境



`uvicorn app.main:app --reload`   --- 会导致拒绝连接！！！
`nohup uvicorn app.main:app --reload --host 0.0.0.0 --port 8000 > uvicorn.log 2>&1 &`  -- 用这个命令后台启动！！






### docker踩坑
AWS上默认没有docker compose
需要手动安装
```bash
sudo mkdir -p /usr/local/lib/docker/cli-plugins
sudo curl -SL https://github.com/docker/compose/releases/download/v2.27.0/docker-compose-linux-x86_64 -o /usr/local/lib/docker/cli-plugins/docker-compose
sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
docker compose version
```


启动命令：`sudo docker compose up -d`
停止命令：`sudo docker compose down`
重新构建镜像：`sudo docker compose build --no-cache` 










```js
{/* 2. 标题区域 (大幅调整：加大字号，拉大字距，增强对比) */}
                <div className="text-center mb-16 sm:mb-20">
                    <h1 className="font-general-bold text-6xl sm:text-8xl lg:text-9xl text-primary-dark dark:text-white mb-8 tracking-tighter leading-none">
                        Code with <br className="sm:hidden" /> purpose.
                        <br />
                        <span className="text-gray-300 dark:text-gray-600">
                            Built to last.
                        </span>
                    </h1>

                    <p className="font-general-regular text-lg sm:text-2xl text-gray-500 dark:text-gray-400 max-w-3xl mx-auto leading-relaxed">
                        Bringing architectural discipline and full-stack agility.<br className="hidden sm:block" />
                        Focused on <b>clean code</b>, <b>stability</b>, and <b>precision</b>.
                    </p>
                </div>
```